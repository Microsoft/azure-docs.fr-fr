---
title: Exporter et importer en bloc des inscriptions Azure Notification Hubs | Microsoft Docs
description: Découvrez comment utiliser la fonctionnalité en bloc de Notification Hubs pour effectuer un grand nombre d’opérations à la fois sur un hub de notifications ou pour exporter l’ensemble des inscriptions.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/18/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: c0771864229c8a3918da076de48fb6e033d2cf5a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89018176"
---
# <a name="export-and-import-azure-notification-hubs-registrations-in-bulk"></a>Exporter et importer en bloc des inscriptions Azure Notification Hubs

Certains scénarios nécessitent de créer ou modifier un grand nombre d’inscriptions dans un hub de notifications. Les mises à jour des étiquettes après des calculs par lots ou la migration d’une implémentation push existante pour utiliser Azure Notification Hubs en sont deux exemples.

Cet article explique comment effectuer un grand nombre d’opérations sur un hub de notifications ou exporter l’ensemble des inscriptions, en bloc.

## <a name="high-level-flow"></a>Flux général

La prise en charge du traitement par lots vise à permettre l’exécution de travaux durables impliquant des millions d’inscriptions. Pour répondre à cette charge de travail, la prise en charge du traitement par lots utilise le stockage Azure pour stocker la sortie et les détails des tâches. Pour les opérations de mise à jour en bloc, l’utilisateur doit créer un fichier dans un conteneur d’objets blob, dans lequel sont listées les opérations de mise à jour des inscriptions. Quand il démarre le travail, l’utilisateur fournit une URL vers l’objet blob d’entrée ainsi qu’une URL vers un répertoire de sortie (également dans un conteneur d’objets blob). Une fois que le travail a démarré, l’utilisateur peut vérifier l’état en interrogeant l’emplacement de l’URL indiqué au démarrage du travail. Un travail donné peut uniquement effectuer des opérations d’un type spécifique (création, mise à jour ou suppression). Les opérations d’exportation sont effectuées de façon analogue.

## <a name="import"></a>Importer

### <a name="set-up"></a>Configurer

Cette section suppose la présence des entités suivantes :

- Un hub de notifications provisionné.
- Un conteneur d’objets blob de stockage Azure.
- Des références au [package NuGet Azure Storage](https://www.nuget.org/packages/windowsazure.storage/) et au [package NuGet Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

### <a name="create-input-file-and-store-it-in-a-blob"></a>Créer un fichier d’entrée et le stocker dans un objet blob

Un fichier d’entrée contient une liste d’inscriptions sérialisées en XML, à raison d’une par ligne. En utilisant le SDK Azure, l’exemple de code suivant montre comment sérialiser les inscriptions et les charger dans le conteneur d’objets blob :

```csharp
private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
{
    StringBuilder builder = new StringBuilder();
    foreach (var registrationDescription in descriptions)
    {
        builder.AppendLine(registrationDescription.Serialize());
    }

    var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
    using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
    {
        inputBlob.UploadFromStream(stream);
    }
}
```

> [!IMPORTANT]
> Le code ci-dessus sérialise les inscriptions en mémoire, puis charge le flux entier dans un objet blob. Si vous avez chargé un fichier plus gros que quelques mégaoctets, consultez l’aide sur les objets blob Azure pour savoir comment effectuer ces étapes, par exemple, les [objets blob de blocs](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs).

### <a name="create-url-tokens"></a>Créer des jetons d’URL

Après avoir chargé votre fichier d’entrée, générez les URL du fichier d’entrée et du répertoire de sortie à fournir à votre hub de notifications. Vous pouvez utiliser deux conteneurs d’objets blob différents pour l’entrée et la sortie.

```csharp
static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
    };

    string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + sasContainerToken);
}

static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Read
    };
    string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + "/" + filePath + sasToken);
}
```

### <a name="submit-the-job"></a>Envoi du travail

Quand vous avez les deux URL d’entrée et de sortie, vous êtes prêt à démarrer le traitement du travail par lots.

```csharp
NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
var createTask = client.SubmitNotificationHubJobAsync(
new NotificationHubJob {
        JobType = NotificationHubJobType.ImportCreateRegistrations,
        OutputContainerUri = outputContainerSasUri,
        ImportFileUri = inputFileSasUri
    }
);
createTask.Wait();

var job = createTask.Result;
long i = 10;
while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
{
    var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
    getJobTask.Wait();
    job = getJobTask.Result;
    Thread.Sleep(1000);
    i--;
}
```

En plus des URL d’entrée et de sortie, cet exemple crée un objet `NotificationHubJob` qui contient un objet `JobType` d’un des types suivants :

- `ImportCreateRegistrations`
- `ImportUpdateRegistrations`
- `ImportDeleteRegistrations`

Une fois l’appel terminé, le travail est poursuivi par le hub de notifications. Vous pouvez vérifier l’état du travail en appelant [GetNotificationHubJobAsync](/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.getnotificationhubjobasync?view=azure-dotnet).

À l’issue du travail, vous pouvez examiner les résultats dans les fichiers suivants situés dans votre répertoire de sortie :

- `/<hub>/<jobid>/Failed.txt`
- `/<hub>/<jobid>/Output.txt`

Ces fichiers contiennent la liste des opérations traitées par lots qui ont réussi ou échoué. Le fichier est au format `.cvs` : chaque ligne a le numéro de ligne du fichier d’entrée d’origine et contient la sortie de l’opération (généralement la description de l’inscription créée ou mise à jour).

### <a name="full-sample-code"></a>Exemple de code complet

L’exemple de code suivant importe des inscriptions vers un hub de notifications.

```csharp
using Microsoft.Azure.NotificationHubs;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Runtime.Serialization;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Xml;

namespace ConsoleApplication1
{
    class Program
    {
        private static string CONNECTION_STRING = "---";
        private static string HUB_NAME = "---";
        private static string INPUT_FILE_NAME = "CreateFile.txt";
        private static string STORAGE_ACCOUNT = "---";
        private static string STORAGE_PASSWORD = "---";
        private static StorageUri STORAGE_ENDPOINT = new StorageUri(new Uri("---"));

        static void Main(string[] args)
        {
            var descriptions = new[]
            {
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMkUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMjUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMhUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMdUxREQFBlVTTkMwMQ"),
            };

            // Write to blob store to create an input file
            var blobClient = new CloudBlobClient(STORAGE_ENDPOINT, new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(STORAGE_ACCOUNT, STORAGE_PASSWORD));
            var container = blobClient.GetContainerReference("testjobs");
            container.CreateIfNotExists();

            SerializeToBlob(container, descriptions);

            // TODO then create Sas
            var outputContainerSasUri = GetOutputDirectoryUrl(container);
            var inputFileSasUri = GetInputFileUrl(container, INPUT_FILE_NAME);


            // Import this file
            NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
            var createTask = client.SubmitNotificationHubJobAsync(
                new NotificationHubJob {
                    JobType = NotificationHubJobType.ImportCreateRegistrations,
                    OutputContainerUri = outputContainerSasUri,
                    ImportFileUri = inputFileSasUri
                }
            );
            createTask.Wait();

            var job = createTask.Result;
            long i = 10;
            while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
            {
                var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
                getJobTask.Wait();
                job = getJobTask.Result;
                Thread.Sleep(1000);
                i--;
            }
        }

        private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
        {
            StringBuilder builder = new StringBuilder();
            foreach (var registrationDescription in descriptions)
            {
                builder.AppendLine(registrationDescription.Serialize());
            }

            var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
            using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
            {
                inputBlob.UploadFromStream(stream);
            }
        }

        static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + sasContainerToken);
        }

        static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + "/" + filePath + sasToken);
        }

        static string GetJobPath(string namespaceName, string notificationHubPath, string jobId)
        {
            return string.Format(CultureInfo.InvariantCulture, @"{0}//{1}/{2}/", namespaceName, notificationHubPath, jobId);
        }
    }
}
```

## <a name="export"></a>Exporter

L’exportation des inscriptions est similaire à leur importation, avec toutefois les différences suivantes :

- Vous avez uniquement besoin de l’URL de sortie.
- Vous créez un travail NotificationHubJob de type ExportRegistrations.

### <a name="sample-code-snippet"></a>Exemple d’extrait de code

Voici un exemple d’extrait de code qui exporte des inscriptions en Java :

```java
// Submit an export job
NotificationHubJob job = new NotificationHubJob();
job.setJobType(NotificationHubJobType.ExportRegistrations);
job.setOutputContainerUri("container uri with SAS signature");
job = hub.submitNotificationHubJob(job);

// Wait until the job is done
while(true){
    Thread.sleep(1000);
    job = hub.getNotificationHubJob(job.getJobId());
    if(job.getJobStatus() == NotificationHubJobStatus.Completed)
        break;
}

```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les inscriptions, consultez les articles suivants :

- [Gestion des inscriptions](notification-hubs-push-notification-registration-management.md)
- [Balises pour les inscriptions](notification-hubs-tags-segment-push-message.md)
- [Inscriptions de modèles](notification-hubs-templates-cross-platform-push-messages.md)
