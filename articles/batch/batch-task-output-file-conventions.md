---
title: Conserver les données de sortie dans Stockage Azure avec la bibliothèque File Conventions .NET
description: Apprenez à utiliser la bibliothèque File Conventions d’Azure Batch pour conserver les sorties de travaux et de tâches dans Stockage Azure et les afficher dans le portail Azure.
ms.topic: how-to
ms.date: 11/14/2018
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: 1a45eed421dd8d734fcef0dd452df1d4a65fd053
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936960"
---
# <a name="persist-job-and-task-data-to-azure-storage-with-the-batch-file-conventions-library-for-net"></a>Conserver le résultat d’un travail et d’une tâche dans Azure Storage avec la bibliothèque File Conventions Batch pour .NET

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

La [bibliothèque File Conventions pour .NET d’Azure Batch][nuget_package] est un moyen de conserver des données de tâches. La bibliothèque File Conventions simplifie les processus de stockage et de récupération des données de sortie de tâches dans Azure Storage. Cette bibliothèque est destinée à être utilisée dans le code de tâche et client &mdash; dans le code de tâche pour la conservation des fichiers et dans le code client pour les répertorier et les récupérer. Le code de tâche peut également utiliser la bibliothèque pour récupérer la sortie des tâches en amont, comme dans un cas de [dépendances de tâche](batch-task-dependencies.md).

Pour récupérer des fichiers de sortie avec la bibliothèque File Conventions, vous pouvez localiser les fichiers d’un travail donné ou d’une tâche donnée en les répertoriant par ID et objectif. Vous n’avez pas besoin de connaître les noms ou les emplacements des fichiers. Vous pouvez par exemple utiliser la bibliothèque File Conventions pour répertorier tous les fichiers intermédiaires d’une tâche donnée, ou obtenir un aperçu de fichier d’un travail donné.

> [!TIP]
> À partir de la version 2017-05-01, l’API du service Batch prend en charge la conservation des données de sortie vers Azure Storage pour les tâches, y compris celles du gestionnaire de travaux, qui s’exécutent sur des pools sous la configuration de la machine virtuelle. L’API du service Batch fournit un moyen simple pour conserver les sorties depuis le code, qui crée une tâche et constitue une alternative à la bibliothèque File Conventions. Vous pouvez modifier les applications du client Batch pour conserver les sorties sans mettre à jour l’application que votre tâche exécute. Pour en savoir plus, consultez l’article [Conserver les données de tâche dans Azure Storage avec l’API de service Batch](batch-task-output-files.md).

## <a name="when-do-i-use-the-file-conventions-library-to-persist-task-output"></a>Quand utiliser la bibliothèque File Conventions pour conserver les sorties de tâche ?

Azure Batch offre plusieurs manières de conserver les sorties de tâche. File Conventions remplit parfaitement son rôle dans les cas suivants :

- Vous pouvez facilement modifier le code de l’application que votre tâche exécute afin de conserver les fichiers à l’aide de la bibliothèque File Conventions.
- Vous voulez diffuser les données vers Azure Storage alors que la tâche est en cours d’exécution.
- Vous voulez conserver les données de pools créés avec la configuration de service cloud ou la configuration de machine virtuelle.
- Votre application cliente ou d’autres tâches du travail doivent localiser et télécharger les fichiers de sortie des tâches par ID ou usage.
- Vous voulez consulter les sorties de tâche dans le portail Azure.

Si votre scénario diffère de ceux répertoriés ci-dessus, vous devrez peut-être envisager une approche différente. Pour en savoir plus sur les autres options de conservation des sorties de tâche, consultez l’article [Conserver les sorties de travail et de tâche terminées dans Azure Storage](batch-task-output.md).

## <a name="what-is-the-batch-file-conventions-standard"></a>Qu’est-ce qu’un standard de nommage des fichiers Batch dans File Conventions ?

Le [standard de nommage des fichiers Batch dans File Conventions](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files) fournit un schéma d’affectation de noms pour les conteneurs de destination et chemins d’accès d’objets blob dans lesquels vos fichiers de sortie sont rédigés. Les fichiers conservés dans Azure Storage qui adhèrent au standard de nommage de File Conventions sont automatiquement disponibles à la consultation dans le portail Azure. Le portail connait les conventions d’affectation de noms et peut donc afficher les fichiers qui y adhèrent.

La bibliothèque File Conventions pour .NET nomme automatiquement vos conteneurs de stockage et les fichiers de sortie de tâche en respectant le standard de nommage de File Conventions. La bibliothèque File Conventions fournit également des méthodes de requête de fichiers de sortie dans Azure Storage selon l’ID ou usage de la tâche ou du travail.

Si vous développez avec un autre langage que .NET, vous pouvez implémenter le standard de nommage de File Convention dans votre application. Pour en savoir plus, consultez l’article [Implémenter le standard Batch File Conventions](batch-task-output.md#implement-the-batch-file-conventions-standard).

## <a name="link-an-azure-storage-account-to-your-batch-account"></a>Lier un compte Azure Storage à votre compte Batch

Pour conserver les données de sortie dans Azure Storage à l’aide de la bibliothèque File Conventions, vous devez d’abord lier un compte Azure Storage à votre compte Batch. Si ce n’est déjà fait, liez un compte Storage à votre compte Batch à l’aide du [portail Azure](https://portal.azure.com) :

1. Accédez à votre compte  Batch dans le portail Azure.
1. Sous **Paramètres**, sélectionnez **Compte Storage**.
1. Si vous n’avez pas déjà associé de compte Storage à votre compte Batch, cliquez sur **Compte Storage (aucun)** .
1. Sélectionnez un compte Storage pour votre abonnement dans la liste. Pour de meilleures performances, utilisez un compte Azure Storage qui se trouve dans la même région que le compte Batch où les tâches sont exécutées.

## <a name="persist-output-data"></a>Conserver les données de sortie

Pour conserver les données de sortie de travail et de tâche avec la bibliothèque File Conventions, créez un conteneur dans Azure Storage, puis enregistrez la sortie dans le conteneur. Utilisez la [bibliothèque du client Azure Storage pour .NET](https://www.nuget.org/packages/WindowsAzure.Storage) dans le code de tâche pour charger la sortie de tâche dans le conteneur.

Pour en savoir plus sur les conteneurs et les objets blob dans Azure Storage, consultez l’article [Prise en main du stockage d’objets blob Azure à l’aide de .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).

> [!WARNING]
> Toutes les sorties de travail et de tâche conservées avec la bibliothèque File Conventions sont stockées dans le même conteneur. Si de nombreuses tâches tentent de conserver des fichiers en même temps, des limitations de stockage Azure peuvent être appliquées. Pour plus d’informations sur les limitations, consultez [Liste de contrôle des performances et de la scalabilité pour le stockage Blob](../storage/blobs/storage-performance-checklist.md).

### <a name="create-storage-container"></a>Créer un conteneur de stockage

Pour conserver les sorties de tâche dans Azure Storage, créez d’abord un conteneur avec la commande [CloudJob][net_cloudjob].[PrepareOutputStorageAsync][net_prepareoutputasync]. Cette méthode d’extension nécessite un objet [CloudStorageAccount][net_cloudstorageaccount] comme paramètre. Elle crée un conteneur nommé selon le standard File Conventions, pour que son contenu puisse être découvert par le portail Azure et par les méthodes de récupération présentées plus loin dans l’article.

En général, vous placez ce code pour créer un conteneur dans votre application cliente &mdash;, qui crée vos pools, travaux et tâches.

```csharp
CloudJob job = batchClient.JobOperations.CreateJob(
    "myJob",
    new PoolInformation { PoolId = "myPool" });

// Create reference to the linked Azure Storage account
CloudStorageAccount linkedStorageAccount =
    new CloudStorageAccount(myCredentials, true);

// Create the blob storage container for the outputs
await job.PrepareOutputStorageAsync(linkedStorageAccount);
```

### <a name="store-task-outputs"></a>Stocker les sorties des tâches

Maintenant que vous avez préparé un conteneur dans Azure Storage, les tâches peuvent y enregistrer leur sortie à l’aide de la classe [TaskOutputStorage][net_taskoutputstorage] figurant dans la bibliothèque File Conventions.

Dans votre code de tâche, commencez par créer un objet [TaskOutputStorage][net_taskoutputstorage], puis, lorsque la tâche a terminé son travail, appelez la méthode [TaskOutputStorage][net_taskoutputstorage].[SaveAsync][net_saveasync] pour enregistrer sa sortie dans le Stockage Azure.

```csharp
CloudStorageAccount linkedStorageAccount = new CloudStorageAccount(myCredentials);
string jobId = Environment.GetEnvironmentVariable("AZ_BATCH_JOB_ID");
string taskId = Environment.GetEnvironmentVariable("AZ_BATCH_TASK_ID");

TaskOutputStorage taskOutputStorage = new TaskOutputStorage(
    linkedStorageAccount, jobId, taskId);

/* Code to process data and produce output file(s) */

await taskOutputStorage.SaveAsync(TaskOutputKind.TaskOutput, "frame_full_res.jpg");
await taskOutputStorage.SaveAsync(TaskOutputKind.TaskPreview, "frame_low_res.jpg");
```

Le paramètre `kind` de la méthode [TaskOutputStorage](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage).[SaveAsync](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync#overloads) permet d’organiser les fichiers conservés. Il existe quatre types prédéfinis [TaskOutputKind][net_taskoutputkind] : `TaskOutput` , `TaskPreview`, `TaskLog`, et `TaskIntermediate.`. Vous pouvez également définir des catégories de sortie personnalisées.

Ces types de sortie vous permettent de spécifier le type de sortie à répertorier lorsque vous interrogez ultérieurement Batch pour connaître les sorties conservées d’une tâche donnée. En d’autres termes, lorsque vous répertoriez les sorties d’une tâche, vous pouvez filtrer la liste sur l’un des types de sortie. Par exemple, « Donnez-moi un *aperçu* de la tâche *109* ». Vous trouverez plus d'informations sur les listes de sorties et leur récupération à la section Récupérer la sortie plus loin dans l'article.

> [!TIP]
> Le type de sortie détermine également où un fichier particulier s’affiche dans le portail Azure : Les fichiers classés *TaskOutput* apparaissent dans **Task output files** (Fichiers de sortie de tâche), et les fichiers *TaskLog* s’affichent dans **Journaux d’activité de la tâche**.

### <a name="store-job-outputs"></a>Stocker les sorties des travaux

Outre le stockage des sorties des tâches, vous pouvez stocker les sorties associées à un travail entier. Par exemple, dans la tâche de fusion d’un travail de rendu de film, vous pouvez conserver le film intégralement rendu sous forme de sortie de travail. Lorsque votre travail est terminé, votre application cliente peut répertorier et récupérer les sorties du travail, sans devoir interroger les tâches individuelles.

Stockez la sortie du travail en appelant la méthode [JobOutputStorage][net_joboutputstorage].[SaveAsync][net_joboutputstorage_saveasync] et spécifiez le paramètre [JobOutputKind][net_joboutputkind] et le nom de fichier :

```csharp
CloudJob job = new JobOutputStorage(acct, jobId);
JobOutputStorage jobOutputStorage = job.OutputStorage(linkedStorageAccount);

await jobOutputStorage.SaveAsync(JobOutputKind.JobOutput, "mymovie.mp4");
await jobOutputStorage.SaveAsync(JobOutputKind.JobPreview, "mymovie_preview.mp4");
```

Comme avec le type **TaskOutputKind** pour les sorties des tâches, vous pouvez utiliser le type [JobOutputKind][net_joboutputkind] pour catégoriser les fichiers conservés d’un travail. Ce paramètre vous permet d’interroger (répertorier) ultérieurement un type spécifique de sortie. Le type **JobOutputKind** inclut les catégories de sortie et d’aperçu, et prend en charge la création de catégories personnalisées.

### <a name="store-task-logs"></a>Stocker les journaux d’activité de tâches

En plus de conserver un fichier dans un espace de stockage durable à l’issue d’une tâche ou d’un travail, vous pouvez avoir besoin de conserver les fichiers mis à jour pendant l’exécution d’une tâche, par exemple les fichiers journaux &mdash; ou `stdout.txt` et `stderr.txt`. À cet effet, la bibliothèque Azure Batch File Conventions fournit la méthode [TaskOutputStorage][net_taskoutputstorage].[SaveTrackedAsync][net_savetrackedasync]. Avec [SaveTrackedAsync][net_savetrackedasync], vous pouvez effectuer le suivi des mises à jour apportées à un fichier sur le nœud (à un intervalle que vous définissez) et conserver ces mises à jour dans le Stockage Azure.

Dans l’extrait de code suivant, nous utilisons [SaveTrackedAsync][net_savetrackedasync] pour mettre à jour `stdout.txt` dans le Stockage Azure toutes les 15 secondes pendant l’exécution de la tâche :

```csharp
TimeSpan stdoutFlushDelay = TimeSpan.FromSeconds(3);
string logFilePath = Path.Combine(
    Environment.GetEnvironmentVariable("AZ_BATCH_TASK_DIR"), "stdout.txt");

// The primary task logic is wrapped in a using statement that sends updates to
// the stdout.txt blob in Storage every 15 seconds while the task code runs.
using (ITrackedSaveOperation stdout =
        await taskStorage.SaveTrackedAsync(
        TaskOutputKind.TaskLog,
        logFilePath,
        "stdout.txt",
        TimeSpan.FromSeconds(15)))
{
    /* Code to process data and produce output file(s) */

    // We are tracking the disk file to save our standard output, but the
    // node agent may take up to 3 seconds to flush the stdout stream to
    // disk. So give the file a moment to catch up.
     await Task.Delay(stdoutFlushDelay);
}
```

La section commentée `Code to process data and produce output file(s)` est un espace réservé pour le code que votre tâche exécuterait normalement. Par exemple, vous pouvez insérer un code qui télécharge des données à partir d’Azure Storage et effectue sur celles-ci des transformations ou des calculs. La partie importante de cet extrait de code montre comment encapsuler un tel code dans un bloc `using` pour mettre régulièrement à jour un fichier avec [SaveTrackedAsync][net_savetrackedasync].

L’agent de nœud est un programme qui s’exécute sur chaque nœud dans le pool et fournit l’interface de commande et de contrôle entre le nœud et le service Batch. L’appel `Task.Delay` est nécessaire à la fin de ce bloc `using` pour garantir que l’agent de nœud a le temps de vider le contenu du standard dans le fichier stdout.txt du nœud. Sans ce délai, il est possible de manquer les dernières secondes de la sortie. Ce délai n’est pas forcément requis pour tous les fichiers.

> [!NOTE]
> Lorsque vous activez le suivi des fichiers avec **SaveTrackedAsync**, seuls les *ajouts* apportés au fichier suivi sont conservés dans Azure Storage. Utilisez cette méthode uniquement pour le suivi des fichiers journaux sans rotation ou des autres fichiers qui sont rédigés avec des opérations d’ajout à la fin du fichier.

## <a name="retrieve-output-data"></a>Récupérer les données de sortie

Lorsque vous récupérez votre sortie conservée à l’aide de la bibliothèque Azure Batch File Conventions, vous procédez d’une façon centrée sur les travaux et les tâches. Vous pouvez demander la sortie d’une tâche ou d’un travail donné sans avoir à connaître son chemin d’accès dans Azure Storage ni même son nom de fichier. À la place, vous pouvez effectuer des requêtes de fichiers de sortie par ID de tâche ou travail.

L’extrait de code suivant itère les tâches d’un travail, imprime des informations sur les fichiers de sortie de tâche, puis télécharge les fichiers à partir d’Azure Storage.

```csharp
foreach (CloudTask task in myJob.ListTasks())
{
    foreach (OutputFileReference output in
        task.OutputStorage(storageAccount).ListOutputs(
            TaskOutputKind.TaskOutput))
    {
        Console.WriteLine($"output file: {output.FilePath}");

        output.DownloadToFileAsync(
            $"{jobId}-{output.FilePath}",
            System.IO.FileMode.Create).Wait();
    }
}
```

## <a name="view-output-files-in-the-azure-portal"></a>Afficher les fichiers de sortie dans le portail Azure

Le portail Azure affiche les fichiers de sortie de tâches et les journaux d’activité qui sont conservés dans un compte Azure Storage lié à l’aide du [standard de nommage des fichiers Batch dans File Conventions](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files). Vous pouvez implémenter ces conventions vous-même dans le langage de votre choix ou utiliser la bibliothèque File Conventions dans vos applications .NET.

Pour activer l’affichage de vos fichiers de sortie dans le portail, vous devez respecter les exigences suivantes :

1. Liez un compte Stockage Azure à votre compte Batch.
1. Respectez les conventions d’affectation de noms prédéfinies pour les conteneurs de stockage et les fichiers lors de la conservation des sorties. Vous trouverez la définition de ces conventions dans le fichier [LISEZMOI][github_file_conventions_readme] de la bibliothèque File Conventions. Si vous utilisez la bibliothèque [Azure Batch File Conventions][nuget_package] pour conserver vos sorties, vos fichiers sont conservés selon le standard de nommage de File Conventions.

Pour afficher les fichiers de sortie de tâches et les journaux d’activité dans le portail Azure, accédez à la tâche dont la sortie vous intéresse, puis cliquez sur **Fichiers de sortie enregistrés** ou **Journaux d’activité enregistrés**. Cette image affiche l’écran **Saved output files (Fichiers de sortie enregistrés)** pour la tâche pourvue de l’ID « 007 » :

![Panneau des sorties des tâches dans le Portail Azure][2]

## <a name="code-sample"></a>Exemple de code

L’exemple de projet [PersistOutputs][github_persistoutputs] est l’un des [exemples de code Azure Batch][github_samples] disponibles sur GitHub. Cette solution Visual Studio montre comment utiliser la bibliothèque Azure Batch File Conventions pour conserver une sortie de tâche dans l’espace de stockage durable. Pour exécuter l’exemple, procédez comme suit :

1. Ouvrez le projet dans **Visual Studio 2019**.
2. Ajoutez vos **informations d’identification de compte** Batch et Stockage à **AccountSettings.settings** dans le projet Microsoft.Azure.Batch.Samples.Common.
3. **Générez** la solution sans l’exécuter. Restaurez les packages NuGet si vous y êtes invité.
4. Utilisez le portail Azure pour charger un [package d’application](batch-application-packages.md) pour **PersistOutputsTask**. Insérez le fichier `PersistOutputsTask.exe` et ses assemblys dépendants dans le package .zip, puis définissez l’ID de l’application sur PersistOutputsTask et la version du package d’application sur 1.0.
5. **Démarrez** (exécutez) le projet **PersistOutputs**.
6. Quand vous êtes invité à choisir la technologie de persistance à utiliser pour l’exécution de l’exemple, entrez **1** pour exécuter l’exemple à l’aide de la bibliothèque File Conventions pour conserver les sorties de tâche. 

## <a name="next-steps"></a>Étapes suivantes

### <a name="get-the-batch-file-conventions-library-for-net"></a>Obtenir la bibliothèque Batch File Conventions pour .NET

La bibliothèque Batch File Conventions pour .NET est disponible sur [NuGet][nuget_package]. La bibliothèque développe les classes [CloudJob][net_cloudjob] et [CloudTask][net_cloudtask] avec de nouvelles méthodes. Consultez également la [documentation de référence](/dotnet/api/microsoft.azure.batch.conventions.files) pour la bibliothèque File Conventions.

Le [code source][github_file_conventions] de la bibliothèque File Conventions est disponible sur GitHub dans le Kit de développement logiciel (SDK) Microsoft Azure pour le référentiel .NET. 

### <a name="explore-other-approaches-for-persisting-output-data"></a>Explorer d’autres approches de conservation des données de sortie

- Consultez l’article [Conserver les sorties de travaux et tâches terminés dans Azure Storage](batch-task-output.md) pour une vue d’ensemble de la conservation des données de tâche et de travail.
- Consultez [Conserver les données de tâche dans le Stockage Azure à l’aide de l’API du service Batch](batch-task-output-files.md) pour savoir comment utiliser l’API du service Batch pour conserver des données de sortie.

[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_file_conventions]: https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files
[github_file_conventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files/README.md
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: /dotnet/api/microsoft.azure.batch.batchclient
[net_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_cloudstorageaccount]: /java/api/com.microsoft.azure.storage.cloudstorageaccount
[net_cloudtask]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_fileconventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files/README.md
[net_joboutputkind]: /dotnet/api/microsoft.azure.batch.conventions.files.joboutputkind
[net_joboutputstorage]: /dotnet/api/microsoft.azure.batch.conventions.files.joboutputstorage
[net_joboutputstorage_saveasync]: /dotnet/api/microsoft.azure.batch.conventions.files.joboutputstorage.saveasync
[net_msdn]: /dotnet/api/microsoft.azure.batch
[net_prepareoutputasync]: /dotnet/api/microsoft.azure.batch.conventions.files.cloudjobextensions.prepareoutputstorageasync
[net_saveasync]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync
[net_savetrackedasync]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.savetrackedasync
[net_taskoutputkind]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputkind
[net_taskoutputstorage]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage
[nuget_manager]: https://docs.nuget.org/consume/installing-nuget
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: https://storageexplorer.com/

[1]: ./media/batch-task-output/task-output-01.png "Sélecteurs Saved output files (Fichiers de sortie enregistrés) et Fichiers enregistrés dans le portail"
[2]: ./media/batch-task-output/task-output-02.png "Panneau des sorties des tâches dans le Portail Azure"
