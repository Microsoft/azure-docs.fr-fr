---
title: Création de branches dans un pipeline Azure Data Factory
description: Découvrez comment contrôler le flux de données dans Azure Data Factory à l’aide d’activités de création de branches et de chaînage.
services: data-factory
author: dcstwh
ms.author: weetok
manager: anandsub
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 9/27/2019
ms.openlocfilehash: ab7d17ee61d733483b6d3573e9bd69b1628c7940
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496940"
---
# <a name="branching-and-chaining-activities-in-a-data-factory-pipeline"></a>Activités de création de branches et chaînage dans un pipeline Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous allez créer un pipeline Data Factory qui présente certaines fonctionnalités de flux de contrôle. Ce pipeline effectue une copie d’un conteneur Stockage Blob Azure vers un autre conteneur dans le même compte de stockage. Si l’activité de copie réussit, le pipeline envoie les détails de l’opération de copie réussie dans un e-mail. Ces informations peuvent inclure la quantité de données écrites. Si l’activité de copie échoue, le pipeline envoie les détails de l’échec de la copie (par exemple le message d’erreur) dans un e-mail. Tout au long de ce didacticiel, vous allez apprendre à passer des paramètres.

Ce graphique fournit une vue d’ensemble du scénario :

![Le diagramme montre le Stockage Blob Azure qui est la cible d’une copie, laquelle, en cas de réussite, envoie un e-mail avec des détails ou, en cas d’échec, envoie un e-mail avec les détails de l’erreur.](media/tutorial-control-flow/overview.png)

Ce tutoriel explique comment effectuer les tâches suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données
> * Créer un service lié Stockage Azure
> * Créer un jeu de données d’objet blob Azure
> * Créer un pipeline qui contient une activité de copie et une activité web
> * Envoyer les sorties des activités aux activités ultérieures
> * Utiliser la transmission de paramètres et des variables système
> * Démarrer une exécution de pipeline
> * Surveiller les exécutions de pipeline et d’activité

Ce didacticiel utilise le kit .NET SDK. Vous pouvez utiliser d’autres mécanismes pour interagir avec Azure Data Factory. Pour obtenir des guides de démarrage rapide Data Factory, consultez [Démarrages en 5 minutes](./quickstart-create-data-factory-portal.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Compte Stockage Azure. Vous utilisez le stockage d’objets blob comme magasin de données source. Si vous ne possédez pas de compte de stockage Azure, voir [Création d’un compte de stockage](../storage/common/storage-account-create.md).
* Azure Storage Explorer. Pour installer cet outil, consultez [Explorateur Stockage Azure](https://storageexplorer.com/).
* Azure SQL Database. Vous utilisez la base de données comme magasin de données réceptrice. Si vous n’avez pas de base de données dans Azure SQL Database, consultez [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).
* Visual Studio. Cet article utilise Visual Studio 2019.
* Kit de développement logiciel Microsoft Azure SDK pour .NET. Téléchargez et installez [Azure SDK pour .NET](https://azure.microsoft.com/downloads/).

Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, consultez [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données et les calculs peuvent se trouver dans d’autres régions. Les magasins incluent Stockage Azure et Azure SQL Database. Les calculs incluent HDInsight, utilisé par Data Factory.

Créez une application comme décrit dans [Créer une application Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal). Affectez l’application au rôle **Contributeur** en suivant les instructions dans le même article. Vous aurez besoin de plusieurs valeurs pour les parties ultérieures de ce tutoriel, telles qu’**ID d’application (client)** et **ID de l’annuaire (locataire)** .

### <a name="create-a-blob-table"></a>Créer une table d’objets blob

1. Ouvrez un éditeur de texte. Copiez le texte suivant et enregistrez-le localement sous le nom *input.txt*.

   ```
   Ethel|Berg
   Tamika|Walsh
   ```

1. Ouvrez l’Explorateur de stockage Azure. Développez votre compte de stockage. Cliquez avec le bouton droit sur **Conteneurs d’objets blob** et sélectionnez **Créer un conteneur d’objets blob**.
1. Nommez le nouveau conteneur *adfv2branch* et sélectionnez **Charger** pour ajouter votre fichier *input.txt* au conteneur.

## <a name="create-visual-studio-project"></a>Créer un projet Visual Studio<a name="create-visual-studio-project"></a>

Créez une application console .NET C# :

1. Démarrez Visual Studio et sélectionnez **Créer un projet**.
1. Dans **Créer un projet**, sélectionnez **Application console (.NET Framework)** pour C#, puis **Suivant**.
1. Nommez le projet *ADFv2BranchTutorial*.
1. Sélectionnez **.NET version 4.5.2** ou ultérieure, puis **Créer**.

### <a name="install-nuget-packages"></a>Installer les packages NuGet

1. Cliquez sur **Outils** > **Gestionnaire de package NuGet** > **Console du Gestionnaire de package**.
1. Dans la **console du Gestionnaire de package**, exécutez les commandes suivantes pour installer les packages. Pour plus d’informations, consultez [Package NuGet Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/).

   ```powershell
   Install-Package Microsoft.Azure.Management.DataFactory
   Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```

### <a name="create-a-data-factory-client"></a>Créer un client de fabrique de données

1. Ouvrez *Program.cs* et ajoutez les instructions suivantes :

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using Microsoft.Rest;
   using Microsoft.Azure.Management.ResourceManager;
   using Microsoft.Azure.Management.DataFactory;
   using Microsoft.Azure.Management.DataFactory.Models;
   using Microsoft.IdentityModel.Clients.ActiveDirectory;
   ```

1. Ajoutez ces variables statiques à la classe `Program`. Remplacez les espaces réservés par vos propres valeurs.

   ```csharp
   // Set variables
   static string tenantID = "<tenant ID>";
   static string applicationId = "<application ID>";
   static string authenticationKey = "<Authentication key for your application>";
   static string subscriptionId = "<Azure subscription ID>";
   static string resourceGroup = "<Azure resource group name>";

   static string region = "East US";
   static string dataFactoryName = "<Data factory name>";

   // Specify the source Azure Blob information
   static string storageAccount = "<Azure Storage account name>";
   static string storageKey = "<Azure Storage account key>";
   // confirm that you have the input.txt file placed in th input folder of the adfv2branch container.
   static string inputBlobPath = "adfv2branch/input";
   static string inputBlobName = "input.txt";
   static string outputBlobPath = "adfv2branch/output";
   static string emailReceiver = "<specify email address of the receiver>";

   static string storageLinkedServiceName = "AzureStorageLinkedService";
   static string blobSourceDatasetName = "SourceStorageDataset";
   static string blobSinkDatasetName = "SinkStorageDataset";
   static string pipelineName = "Adfv2TutorialBranchCopy";

   static string copyBlobActivity = "CopyBlobtoBlob";
   static string sendFailEmailActivity = "SendFailEmailActivity";
   static string sendSuccessEmailActivity = "SendSuccessEmailActivity";
   ```

1. Ajoutez le code suivant à la méthode `Main` . Ce code crée une instance de la classe `DataFactoryManagementClient`. Cet objet vous permet ensuite de créer la fabrique de données, un service lié, des jeux de données ainsi qu’un pipeline. Vous pouvez aussi l’utiliser pour superviser les détails de l’exécution du pipeline.

   ```csharp
   // Authenticate and create a data factory management client
   var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
   ```

### <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Ajoutez une méthode `CreateOrUpdateDataFactory` à votre fichier *Program.cs* :

   ```csharp
   static Factory CreateOrUpdateDataFactory(DataFactoryManagementClient client)
   {
       Console.WriteLine("Creating data factory " + dataFactoryName + "...");
       Factory resource = new Factory
       {
           Location = region
       };
       Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));

       Factory response;
       {
           response = client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, resource);
       }

       while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState == "PendingCreation")
       {
           System.Threading.Thread.Sleep(1000);
       }
       return response;
   }
   ```

1. Ajoutez la ligne suivante à la méthode `Main` qui crée une fabrique de données :

   ```csharp
   Factory df = CreateOrUpdateDataFactory(client);
   ```

## <a name="create-an-azure-storage-linked-service"></a>Créer un service lié Stockage Azure

1. Ajoutez une méthode `StorageLinkedServiceDefinition` à votre fichier *Program.cs* :

   ```csharp
   static LinkedServiceResource StorageLinkedServiceDefinition(DataFactoryManagementClient client)
   {
      Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");
      AzureStorageLinkedService storageLinkedService = new AzureStorageLinkedService
      {
          ConnectionString = new SecureString("DefaultEndpointsProtocol=https;AccountName=" + storageAccount + ";AccountKey=" + storageKey)
      };
      Console.WriteLine(SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings));
      LinkedServiceResource linkedService = new LinkedServiceResource(storageLinkedService, name:storageLinkedServiceName);
      return linkedService;
   }
   ```

1. Ajoutez la ligne suivante à la méthode `Main` qui crée un service lié Stockage Azure :

   ```csharp
   client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
   ```

Pour plus d’informations sur les propriétés prises en charge, consultez [Propriétés du service lié](connector-azure-blob-storage.md#linked-service-properties).

## <a name="create-datasets"></a>Créez les jeux de données

Dans cette section, vous créez deux jeux de données : un pour la source et un pour le récepteur.

### <a name="create-a-dataset-for-a-source-azure-blob"></a>Créer un jeu de données pour un objet blob Azure source

Ajoutez une méthode qui crée un *jeu de données d’objet blob Azure*. Pour plus d’informations sur les propriétés prises en charge, consultez [Propriétés du jeu de données](connector-azure-blob-storage.md#dataset-properties).

Ajoutez une méthode `SourceBlobDatasetDefinition` à votre fichier *Program.cs* :

```csharp
static DatasetResource SourceBlobDatasetDefinition(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating dataset " + blobSourceDatasetName + "...");
    AzureBlobDataset blobDataset = new AzureBlobDataset
    {
        FolderPath = new Expression { Value = "@pipeline().parameters.sourceBlobContainer" },
        FileName = inputBlobName,
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        }
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
    DatasetResource dataset = new DatasetResource(blobDataset, name:blobSourceDatasetName);
    return dataset;
}
```

Vous définissez un jeu de données qui représente les données sources dans l’objet blob Azure. Ce jeu de données d’objet blob fait référence au service lié Stockage Azure pris en charge à l’étape précédente. Le jeu de données d’objet blob décrit l’emplacement de l’objet blob à partir duquel effectuer la copie : *FolderPath* et *FileName*.

Notez l’utilisation de paramètres pour *FolderPath*. `sourceBlobContainer` est le nom du paramètre, et l’expression est remplacée par les valeurs passées dans l’exécution du pipeline. La syntaxe pour définir des paramètres est `@pipeline().parameters.<parameterName>`

### <a name="create-a-dataset-for-a-sink-azure-blob"></a>Créer un jeu de données pour un objet blob Azure récepteur

1. Ajoutez une méthode `SourceBlobDatasetDefinition` à votre fichier *Program.cs* :

   ```csharp
   static DatasetResource SinkBlobDatasetDefinition(DataFactoryManagementClient client)
   {
       Console.WriteLine("Creating dataset " + blobSinkDatasetName + "...");
       AzureBlobDataset blobDataset = new AzureBlobDataset
       {
           FolderPath = new Expression { Value = "@pipeline().parameters.sinkBlobContainer" },
           LinkedServiceName = new LinkedServiceReference
           {
               ReferenceName = storageLinkedServiceName
           }
       };
       Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
       DatasetResource dataset = new DatasetResource(blobDataset, name: blobSinkDatasetName);
       return dataset;
   }
   ```

1. Ajoutez le code suivant à la méthode `Main` qui crée les jeux de données source et récepteur de l’objet blob Azure.

   ```csharp
   client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));

   client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));
   ```

## <a name="create-a-c-class-emailrequest"></a>Créez une classe C# : EmailRequest

Dans votre projet C#, créez une classe nommée `EmailRequest`. Cette classe définit les propriétés que le pipeline envoie dans le corps de la requête lors de l’envoi d’un e-mail. Dans ce didacticiel, le pipeline envoie quatre propriétés à l’adresse e-mail :

* Message. Corps de l’e-mail. Pour une copie réussie, cette propriété contient la quantité de données écrites. Pour une copie ayant échoué, cette propriété contient les détails de l’erreur.
* dataFactoryName. Nom de la fabrique de données.
* pipelineName. Nom du pipeline.
* receiver. Paramètre transmis. Cette propriété spécifie le destinataire de l’e-mail.

```csharp
    class EmailRequest
    {
        [Newtonsoft.Json.JsonProperty(PropertyName = "message")]
        public string message;

        [Newtonsoft.Json.JsonProperty(PropertyName = "dataFactoryName")]
        public string dataFactoryName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "pipelineName")]
        public string pipelineName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "receiver")]
        public string receiver;

        public EmailRequest(string input, string df, string pipeline, string receiverName)
        {
            message = input;
            dataFactoryName = df;
            pipelineName = pipeline;
            receiver = receiverName;
        }
    }
```

## <a name="create-email-workflow-endpoints"></a>Créer des points de terminaison de flux de travail d’e-mail

Pour déclencher l’envoi d’un e-mail, vous utilisez [Logic Apps](../logic-apps/logic-apps-overview.md) pour définir le flux de travail. Pour plus d’informations sur la création d’un workflow Logic Apps, consultez [Guide pratique pour créer une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

### <a name="success-email-workflow"></a>Flux de travail d’un e-mail d’avis de réussite

Dans le [portail Azure](https://portal.azure.com), créez un workflow Logic Apps nommé *CopySuccessEmail*. Définissez `When an HTTP request is received` comme déclencheur de workflow. Pour le déclencheur de votre requête, renseignez `Request Body JSON Schema` avec le texte JSON suivant :

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Votre workflow ressemble à l’exemple suivant :

![Flux de travail d’un e-mail d’avis de réussite](media/tutorial-control-flow/success-email-workflow-trigger.png)

Ce contenu JSON correspond à la classe `EmailRequest` que vous avez créée dans la section précédente.

Ajoutez une action `Office 365 Outlook – Send an email`. Pour l’action **Envoyer un e-mail**, personnalisez la mise en forme de l’e-mail à l’aide des propriétés transmises dans le schéma JSON du **corps** de la requête. Voici un exemple :

![Concepteur d’application logique - Action Envoyer un e-mail](media/tutorial-control-flow/customize-send-email-action.png)

Une fois que vous avez enregistré le workflow, copiez et enregistrez la valeur de l’**URL HTTP POST** à partir du déclencheur.

## <a name="fail-email-workflow"></a>Flux de travail d’un e-mail d’avis d’échec

Clonez **CopySuccessEmail** en tant qu’autre workflow Logic Apps nommé *CopyFailEmail*. Dans le déclencheur de la requête, `Request Body JSON schema` est identique. Modifiez la mise en forme de votre e-mail, notamment `Subject`, pour l’adapter à un avis d’échec. Voici un exemple :

![Concepteur d’application logique - Workflow d’un e-mail d’avis d’échec](media/tutorial-control-flow/fail-email-workflow.png)

Une fois que vous avez enregistré le workflow, copiez et enregistrez la valeur de l’**URL HTTP POST** à partir du déclencheur.

Vous devez maintenant avoir deux URL de workflow, comme dans les exemples suivants :

```csharp
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="create-a-pipeline"></a>Créer un pipeline

Revenez à votre projet dans Visual Studio. Nous allons maintenant ajouter le code qui crée un pipeline avec une activité de copie et une propriété `DependsOn`. Dans ce tutoriel, le pipeline contient une seule activité, une activité de copie, qui accepte le jeu de données blob en tant que source et un autre jeu de données blob en tant que récepteur. Si l’activité de copie réussit ou échoue, elle appelle différentes tâches de messagerie.

Dans ce pipeline, vous utilisez les fonctionnalités suivantes :

* Paramètres
* Activité web
* Dépendance des activités
* Utilisation de la sortie d’une activité en tant qu’entrée d’une autre activité

1. Ajoutez cette méthode à votre projet. Pour plus d’informations, lisez les sections suivantes.

    ```csharp
    static PipelineResource PipelineDefinition(DataFactoryManagementClient client)
            {
                Console.WriteLine("Creating pipeline " + pipelineName + "...");
                PipelineResource resource = new PipelineResource
                {
                    Parameters = new Dictionary<string, ParameterSpecification>
                    {
                        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                        { "receiver", new ParameterSpecification { Type = ParameterType.String } }

                    },
                    Activities = new List<Activity>
                    {
                        new CopyActivity
                        {
                            Name = copyBlobActivity,
                            Inputs = new List<DatasetReference>
                            {
                                new DatasetReference
                                {
                                    ReferenceName = blobSourceDatasetName
                                }
                            },
                            Outputs = new List<DatasetReference>
                            {
                                new DatasetReference
                                {
                                    ReferenceName = blobSinkDatasetName
                                }
                            },
                            Source = new BlobSource { },
                            Sink = new BlobSink { }
                        },
                        new WebActivity
                        {
                            Name = sendSuccessEmailActivity,
                            Method = WebActivityMethod.POST,
                            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/00000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000000",
                            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                            DependsOn = new List<ActivityDependency>
                            {
                                new ActivityDependency
                                {
                                    Activity = copyBlobActivity,
                                    DependencyConditions = new List<String> { "Succeeded" }
                                }
                            }
                        },
                        new WebActivity
                        {
                            Name = sendFailEmailActivity,
                            Method =WebActivityMethod.POST,
                            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000",
                            Body = new EmailRequest("@{activity('CopyBlobtoBlob').error.message}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                            DependsOn = new List<ActivityDependency>
                            {
                                new ActivityDependency
                                {
                                    Activity = copyBlobActivity,
                                    DependencyConditions = new List<String> { "Failed" }
                                }
                            }
                        }
                    }
                };
                Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));
                return resource;
            }
    ```

1. Ajoutez la ligne suivante à la méthode `Main` qui crée le pipeline :

   ```csharp
   client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));
   ```

### <a name="parameters"></a>Paramètres

La première section de notre code de pipeline définit les paramètres.

* `sourceBlobContainer`. Le jeu de données blob source consomme ce paramètre dans le pipeline.
* `sinkBlobContainer`. Le jeu de données blob récepteur consomme ce paramètre dans le pipeline.
* `receiver`. Les deux activités web dans le pipeline qui envoient des e-mails de réussite ou d’échec au récepteur utilisent ce paramètre.

```csharp
Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "receiver", new ParameterSpecification { Type = ParameterType.String } }
    },
```

### <a name="web-activity"></a>Activité web

L’activité web permet d’appeler n’importe quel point de terminaison REST. Pour plus d’informations sur l’activité, consultez [Activité web dans Azure Data Factory](control-flow-web-activity.md). Ce pipeline utilise une activité web pour appeler le workflow d’e-mail Logic Apps. Vous créez deux activités web : une qui appelle le workflow `CopySuccessEmail` et une autre qui appelle `CopyFailWorkFlow`.

```csharp
        new WebActivity
        {
            Name = sendCopyEmailActivity,
            Method = WebActivityMethod.POST,
            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/12345",
            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
            DependsOn = new List<ActivityDependency>
            {
                new ActivityDependency
                {
                    Activity = copyBlobActivity,
                    DependencyConditions = new List<String> { "Succeeded" }
                }
            }
        }
```

Dans la propriété `Url`, collez les points de terminaison d’**URL HTTP POST** à partir de vos workflows Logic Apps. Dans la propriété `Body`, transmettez une instance de la classe `EmailRequest`. La requête d’e-mail contient les propriétés suivantes :

* Message. Transmet la valeur de `@{activity('CopyBlobtoBlob').output.dataWritten`. Accède à une propriété de l’activité de copie précédente et transmet la valeur de `dataWritten`. Pour un échec, passez la sortie de l’erreur au lieu de `@{activity('CopyBlobtoBlob').error.message`.
* dataFactoryName. Transmet la valeur de `@{pipeline().DataFactory}`. Cette variable système vous permet d’accéder au nom de la fabrique de données correspondant. Pour obtenir la liste des variables système, consultez [Variables système](control-flow-system-variables.md).
* pipelineName. Transmet la valeur de `@{pipeline().Pipeline}`. Cette variable système vous permet d’accéder au nom du pipeline correspondant.
* receiver. Transmet la valeur de `"@pipeline().parameters.receiver"`. Accède aux paramètres de pipeline.

Ce code crée une dépendance de l’activité qui dépend de l’activité de copie précédente.

## <a name="create-a-pipeline-run"></a>Créer une exécution du pipeline

Ajoutez le code suivant à la méthode `Main` qui déclenche une exécution du pipeline.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="main-class"></a>Classe Main

Votre méthode `Main` finale doit ressembler à ceci.

```csharp
// Authenticate and create a data factory management client
var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };

Factory df = CreateOrUpdateDataFactory(client);

client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));

Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

Générez et exécutez votre programme pour déclencher une exécution du pipeline.

## <a name="monitor-a-pipeline-run"></a>Surveiller une exécution du pipeline

1. Ajoutez le code suivant à la méthode `Main` :

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

    Ce code vérifie continuellement l’état de l’exécution jusqu’à la fin de la copie des données.

1. Ajoutez le code suivant à la méthode `Main` qui récupère les détails de l’exécution de l’activité de copie, par exemple la taille des données lues/écrites :

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    List<ActivityRun> activityRuns = client.ActivityRuns.ListByPipelineRun(
    resourceGroup, dataFactoryName, runResponse.RunId, DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)).ToList();

    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(activityRuns.First().Output);
        //SaveToJson(SafeJsonConvert.SerializeObject(activityRuns.First().Output, client.SerializationSettings), "ActivityRunResult.json", folderForJsons);
    }
    else
        Console.WriteLine(activityRuns.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>Exécuter le code

Créez et démarrez l’application, puis vérifiez l’exécution du pipeline.

L’application affiche la progression de la création de la fabrique de données, du service lié, des jeux de données, du pipeline et de l’exécution du pipeline. Elle vérifie ensuite l’état de l’exécution du pipeline. Patientez jusqu’à l’affichage des détails de l’exécution de l’activité de copie avec la taille des données lues/écrites. Utilisez ensuite des outils comme l’Explorateur Stockage Azure pour vérifier que les objets blob sont copiés dans *outputBlobPath* à partir de *inputBlobPath* comme vous l’avez spécifié dans les variables.

La sortie doit ressembler à l’exemple suivant  :

```json
Creating data factory DFTutorialTest...
{
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "type": "AzureStorage",
  "typeProperties": {
    "connectionString": "DefaultEndpointsProtocol=https;AccountName=***;AccountKey=**_"
  }
}
Creating dataset SourceStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sourceBlobContainer"
    },
    "fileName": "input.txt"
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating dataset SinkStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sinkBlobContainer"
    }
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating pipeline Adfv2TutorialBranchCopy...
{
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "inputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SourceStorageDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SinkStorageDataset"
          }
        ],
        "name": "CopyBlobtoBlob"
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendSuccessEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Succeeded"
            ]
          }
        ]
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendFailEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Failed"
            ]
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      },
      "receiver": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 00000000-0000-0000-0000-0000000000000
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 20,
  "dataWritten": 20,
  "copyDuration": 4,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
}
{}

Press any key to exit...
```

## <a name="next-steps"></a>Étapes suivantes

Vous avez accompli les tâches suivantes dans le cadre de ce tutoriel :

> [!div class="checklist"]
> _ Créer une fabrique de données
> * Créer un service lié Stockage Azure
> * Créer un jeu de données d’objet blob Azure
> * Créer un pipeline qui contient une activité de copie et une activité web
> * Envoyer les sorties des activités aux activités ultérieures
> * Utiliser la transmission de paramètres et des variables système
> * Démarrer une exécution de pipeline
> * Surveiller les exécutions de pipeline et d’activité

Vous pouvez maintenant poursuivre avec la section Concepts pour obtenir plus d’informations sur Azure Data Factory.
> [!div class="nextstepaction"]
>[Pipelines et activités](concepts-pipelines-activities.md)