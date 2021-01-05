---
title: Tutoriel - Exécuter une charge de travail parallèle à l’aide de l’API .NET
description: Didacticiel - Transcoder des fichiers multimédias en parallèle avec ffmpeg dans Azure Batch à l’aide de la bibliothèque cliente Batch .NET
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 09/29/2020
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: a990a5480a8a6462bb6ef9f84070b78768628fd0
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106527"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-net-api"></a>Tutoriel : Exécuter une charge de travail parallèle avec Azure Batch à l’aide de l’API .NET

Utilisez Azure Batch pour exécuter des programmes de traitement par lots de calcul haute performance (HPC) en parallèle, efficacement et à grande échelle dans Azure. Ce didacticiel vous permet de découvrir un exemple d’exécution C# d’une charge de travail parallèle utilisant Batch. Vous découvrez un workflow d’application Batch courant et comment interagir par programme avec les ressources de stockage et Batch. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Ajouter un package d’application à votre compte Batch
> * S’authentifier avec des comptes de stockage et Batch
> * Charger des fichiers d’entrée sur le stockage
> * Créer un pool de nœuds de calcul pour exécuter une application
> * Créer un travail et des tâches pour traiter les fichiers d’entrée
> * Surveiller l’exécution d’une tâche
> * Récupérer les fichiers de sortie

Dans ce didacticiel, vous convertissez des fichiers de multimédia MP4 en parallèle au format MP3 à l’aide de l’outil open-source [ffmpeg](https://ffmpeg.org/). 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* [Visual Studio 2017 ou version ultérieure](https://www.visualstudio.com/vs), ou [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1) pour Linux, macOS ou Windows.

* Un compte Batch et un compte Stockage Azure lié. Pour créer ces comptes, consultez les démarrages rapides Azure Batch à l’aide du [portail Azure](quick-create-portal.md) ou de l’[interface de ligne de commande Azure](quick-create-cli.md).

* [Version Windows 64 bits de ffmpeg 4.3.1](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08) (.zip). Téléchargez le fichier zip sur votre ordinateur local. Pour ce tutoriel, seul le fichier zip est nécessaire. Vous n’avez pas besoin de décompresser le fichier ou de l’installer localement.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="add-an-application-package"></a>Ajouter un package d’application

Utiliser le portail Azure pour ajouter ffmpeg à votre compte Batch en tant que [package d’application](batch-application-packages.md). Les packages d’application permettent de gérer les applications de tâche et leur déploiement sur les nœuds de calcul dans votre pool. 

1. Dans le portail Azure, cliquez sur **Plus de services** > **Comptes Batch**, puis sur le nom de votre compte Batch.
3. Cliquez sur **Applications** > **Ajouter**.
4. Entrez *ffmpeg* comme **ID d’application** et *4.3.1* comme version de package. Sélectionnez le fichier zip ffmpeg que vous avez téléchargé précédemment, puis cliquez sur **OK**. Le package d’application ffmpeg est ajouté à votre compte Batch.

![Ajouter un package d’application](./media/tutorial-parallel-dotnet/add-application.png)

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-and-run-the-sample"></a>Télécharger et exécuter l’exemple

### <a name="download-the-sample"></a>Télécharger l’exemple

[Téléchargez ou clonez l’exemple d’application](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) à partir de GitHub. Pour cloner le référentiel d’exemple d’application avec un client Git, utilisez la commande suivante :

```
git clone https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial.git
```

Naviguez vers le répertoire qui contient le fichier de la solution Visual Studio `BatchDotNetFfmpegTutorial.sln`.

Ouvrez le fichier de la solution dans Visual Studio et mettez à jour les chaînes d’informations d’identification dans `Program.cs` avec les valeurs obtenues pour vos comptes. Par exemple :

```csharp
// Batch account credentials
private const string BatchAccountName = "mybatchaccount";
private const string BatchAccountKey  = "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==";
private const string BatchAccountUrl  = "https://mybatchaccount.mybatchregion.batch.azure.com";

// Storage account credentials
private const string StorageAccountName = "mystorageaccount";
private const string StorageAccountKey  = "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ==";
```

[!INCLUDE [batch-credentials-include](../../includes/batch-credentials-include.md)]

Vérifiez également que la référence du package d’application ffmpeg de la solution correspond à l’Id et à la version du package ffmpeg chargée sur votre compte Batch.

```csharp
const string appPackageId = "ffmpeg";
const string appPackageVersion = "4.3.1";
```

### <a name="build-and-run-the-sample-project"></a>Créer et exécuter l’exemple de projet

Générez et exécutez l’application dans Visual Studio ou sur la ligne de commande avec les commandes `dotnet build` et `dotnet run`. Après avoir exécuté l’application, passez en revue le code pour savoir ce que fait chaque partie de l’application. Par exemple, dans Visual Studio :

* Cliquez avec le bouton droit dans l’Explorateur de solutions, puis sur **Générer la solution**. 

* Si vous y êtes invité, confirmez la restauration de tous les packages NuGet. Si vous devez télécharger des packages manquants, vérifiez que le [Gestionnaire de Package NuGet](https://docs.nuget.org/consume/installing-nuget) est installé.

Puis exécutez-le. Lorsque vous exécutez l’exemple d’application, la sortie de la console est identique à ce qui suit. Pendant l’exécution, l’étape `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` fait l’objet d’une pause correspondant au démarrage des nœuds de calcul du pool. 

```
Sample start: 11/19/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [WinFFmpegPool]...
Creating job [WinFFmpegJob]...
Adding 5 tasks to job [WinFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]...

Sample end: 11/19/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

Accédez à votre compte Batch dans le portail Azure pour surveiller le pool, les nœuds de calcul, les travaux et les tâches. Par exemple, pour voir une carte thermique des nœuds de calcul de votre pool, cliquez sur **Pools** > *WinFFmpegPool*.

Lorsque les tâches sont en cours d’exécution, la carte thermique est similaire à ce qui suit :

![Carte thermique des pools](./media/tutorial-parallel-dotnet/pool.png)

Le temps d’exécution standard est d’environ **10 minutes** lorsque l’application fonctionne dans sa configuration par défaut. La création d’un pool est l’opération la plus longue.

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>Vérifier le code

Dans les sections suivantes, nous examinons l’exemple d’application en nous servant des opérations qu’elle effectue pour traiter une charge de travail dans le service Batch. Référez-vous au fichier `Program.cs` dans la solution lorsque vous lisez le reste de cet article, car certaines lignes de code de cet exemple ne sont pas expliquées ici.

### <a name="authenticate-blob-and-batch-clients"></a>Authentifier les clients Blob et Batch

Pour interagir avec le compte de stockage lié, l’application utilise la bibliothèque cliente de stockage Azure pour .NET. Elle crée une référence au compte avec [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) avec authentification de la clé partagée. Ensuite, elle crée un [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient).

```csharp
// Construct the Storage account connection string
string storageConnectionString = String.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
                                StorageAccountName, StorageAccountKey);

// Retrieve the storage account
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);

CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

L’application crée un objet [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) pour créer et gérer des pools, des travaux et des tâches dans le service Batch. Le client Batch dans l’exemple utilise l’authentification de la clé partagée. Batch prend également en charge l’authentification via [Azure Active Directory](batch-aad-auth.md) pour authentifier des utilisateurs ou une application sans assistance.

```csharp
BatchSharedKeyCredentials sharedKeyCredentials = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(sharedKeyCredentials))
...
```

### <a name="upload-input-files"></a>Charger des fichiers d’entrée

L’application transmet l’objet `blobClient` à la méthode `CreateContainerIfNotExistAsync` pour créer un conteneur de stockage pour les fichiers d’entrée (format MP4) et un conteneur pour la sortie de la tâche.

```csharp
CreateContainerIfNotExistAsync(blobClient, inputContainerName);
CreateContainerIfNotExistAsync(blobClient, outputContainerName);
```

Ensuite, les fichiers sont chargés dans le conteneur d’entrée à partir du dossier `InputFiles` local. Les fichiers de stockage sont définis en tant qu’objets Batch [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile) que Batch peut télécharger ultérieurement sur les nœuds de calcul. 

Deux méthodes de `Program.cs` sont impliquées dans le chargement des fichiers :

* `UploadFilesToContainerAsync`: retourne une collection d’objets ResourceFile et appelle `UploadResourceFileToContainerAsync` en interne pour charger chaque fichier transmis dans le paramètre `inputFilePaths`.
* `UploadResourceFileToContainerAsync`: charge chaque fichier en tant qu’objet blob dans le conteneur d’entrée. Après le chargement du fichier, la méthode obtient une signature d’accès partagé (SAP) pour l’objet Blob et renvoie un objet ResourceFile pour la représenter.

```csharp
string inputPath = Path.Combine(Environment.CurrentDirectory, "InputFiles");

List<string> inputFilePaths = new List<string>(Directory.GetFileSystemEntries(inputPath, "*.mp4",
    SearchOption.TopDirectoryOnly));

List<ResourceFile> inputFiles = await UploadFilesToContainerAsync(
  blobClient,
  inputContainerName,
  inputFilePaths);
```

Pour plus d’informations sur le chargement de fichiers en tant qu’objets blob sur un compte de stockage avec .NET, consultez [Upload, download, and list blobs using .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md) (Charger, télécharger et répertorier des objets blob à l’aide de .NET).

### <a name="create-a-pool-of-compute-nodes"></a>Créer un pool de nœuds de calcul

Ensuite, l’exemple crée un pool de nœuds de traitement dans le compte Batch, avec un appel à `CreatePoolIfNotExistAsync`. Cette méthode définie utilise la méthode [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) pour définir le nombre de nœuds, la taille de la machine virtuelle et une configuration de pool. Ici, un objet [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) spécifie une référence [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) à une image Windows Server publiée dans la Place de marché Microsoft Azure. Azure Batch prend en charge une large plage de machine virtuelle dans la Place de marché Microsoft Azure, ainsi que des images de machines virtuelles personnalisées.

Le nombre de nœuds et la taille de machine virtuelle sont définis à l’aide de constantes définies. Azure Batch prend en charge les nœuds dédiés et les [nœuds de faible priorité](batch-low-pri-vms.md) que vous pouvez utiliser dans vos pools. Les nœuds dédiés sont réservés à votre pool. Les nœuds de faible priorité sont proposés à prix réduit à partir de la capacité de machine virtuelle excédentaire dans Azure. Les nœuds de faible priorité deviennent indisponibles si la capacité d’Azure est insuffisante. L’exemple par défaut crée un pool contenant seulement 5 nœuds de faible priorité taille *Standard_A1_v2*.

>[!Note]
>Veillez à vérifier vos quotas de nœud. Consultez [Quotas et limites du service Batch](batch-quota-limit.md#increase-a-quota) pour obtenir des instructions sur la création d’une demande de quota.

L’application ffmpeg est déployée sur les nœuds de calcul en ajoutant un [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) à la configuration du pool.

La méthode [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync) soumet le pool au service Batch.

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "MicrosoftWindowsServer",
    offer: "WindowsServer",
    sku: "2016-Datacenter-smalldisk",
    version: "latest");

VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.windows amd64");

pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: DedicatedNodeCount,
    targetLowPriorityComputeNodes: LowPriorityNodeCount,
    virtualMachineSize: PoolVMSize,
    virtualMachineConfiguration: virtualMachineConfiguration);

pool.ApplicationPackageReferences = new List<ApplicationPackageReference>
    {
    new ApplicationPackageReference {
    ApplicationId = appPackageId,
    Version = appPackageVersion}};

await pool.CommitAsync();  
```

### <a name="create-a-job"></a>Créer un travail

Un programme de traitement par lots spécifie un pool pour exécuter des tâches et des paramètres facultatifs tels qu’une priorité et un calendrier pour le travail. L’exemple crée un travail avec un appel à `CreateJobAsync`. Cette méthode définie utilise la méthode [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) pour créer un travail sur le pool.

La méthode [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudjob.commitasync) soumet le travail au service Batch. Dans un premier temps, le travail n’a aucune tâche.

```csharp
CloudJob job = batchClient.JobOperations.CreateJob();
job.Id = JobId;
job.PoolInformation = new PoolInformation { PoolId = PoolId };

await job.CommitAsync();
```

### <a name="create-tasks"></a>Créer des tâches

L’exemple crée des tâches dans le travail avec un appel à la méthode `AddTasksAsync`, ce qui crée une liste d’objets [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask). Chaque `CloudTask` exécute ffmpeg pour traiter un objet `ResourceFile` d’entrée à l’aide de la propriété [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline). ffmpeg a été précédemment installé sur chaque nœud lors de la création du pool. Ici, la ligne de commande exécute ffmpeg pour convertir chaque fichier (vidéo) MP4 en fichier MP3 (audio).

L’exemple crée un objet [OutputFile](/dotnet/api/microsoft.azure.batch.outputfile) pour le fichier MP3 après l’exécution de la ligne de commande. Les fichiers de sortie de chaque tâche (un, dans ce cas) sont chargés sur un conteneur dans le compte de stockage lié, à l’aide de la propriété [OutputFiles](/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles). Précédemment dans l’exemple de code, une URL de signature d’accès partagé (`outputContainerSasUrl`) a été obtenue pour fournir un accès en écriture au conteneur de sortie. Notez les conditions définies sur l’objet `outputFile`. Un fichier de sortie d’une tâche est chargé sur le conteneur seulement une fois la tâche terminée avec succès (`OutputFileUploadCondition.TaskSuccess`). Pour plus de détails sur l’implémentation, consultez l’[exemple de code complet](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) sur GitHub.

Ensuite, l’exemple ajoute des tâches au travail avec la méthode [AddTaskAsync](/dotnet/api/microsoft.azure.batch.joboperations.addtaskasync), qui les met en file d’attente afin de les exécuter sur les nœuds de calcul.

```csharp
 // Create a collection to hold the tasks added to the job.
List<CloudTask> tasks = new List<CloudTask>();

for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);

    // Define task command line to convert each input file.
    string appPath = String.Format("%AZ_BATCH_APP_PACKAGE_{0}#{1}%", appPackageId, appPackageVersion);
    string inputMediaFile = inputFiles[i].FilePath;
    string outputMediaFile = String.Format("{0}{1}",
        System.IO.Path.GetFileNameWithoutExtension(inputMediaFile),
        ".mp3");
    string taskCommandLine = String.Format("cmd /c {0}\\ffmpeg-4.3.1-2020-09-21-full_build\\bin\\ffmpeg.exe -i {1} {2}", appPath, inputMediaFile, outputMediaFile);

    // Create a cloud task (with the task ID and command line)
    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };

    // Task output file
    List<OutputFile> outputFileList = new List<OutputFile>();
    OutputFileBlobContainerDestination outputContainer = new OutputFileBlobContainerDestination(outputContainerSasUrl);
    OutputFile outputFile = new OutputFile(outputMediaFile,
       new OutputFileDestination(outputContainer),
       new OutputFileUploadOptions(OutputFileUploadCondition.TaskSuccess));
    outputFileList.Add(outputFile);
    task.OutputFiles = outputFileList;
    tasks.Add(task);
}

// Add tasks as a collection
await batchClient.JobOperations.AddTaskAsync(jobId, tasks);
return tasks
```

### <a name="monitor-tasks"></a>Surveiller les tâches

Lorsque Batch ajoute des tâches à un travail, le service les met automatiquement en file d’attente et planifie leur exécution sur les nœuds de calcul dans le pool associé. Selon les paramètres que vous spécifiez, Batch gère l’ensemble des opérations de mise en file d’attente, de planification, de ré-exécution et d’administration des tâches.

Il existe plusieurs approches pour l’exécution de la tâche d’analyse. Cet exemple définit une méthode `MonitorTasks` pour signaler uniquement l’achèvement d’une tâche ou son échec ou les états réussis. Le code `MonitorTasks` spécifie un [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) pour sélectionner efficacement uniquement un minimum d’informations sur les tâches. Ensuite, il crée un [TaskStateMonitor](/dotnet/api/microsoft.azure.batch.taskstatemonitor), qui fournit des utilitaires d’assistance pour l’analyse des états de la tâche. Dans `MonitorTasks`, l’exemple attend que toutes les tâches atteignent `TaskState.Completed` dans un délai imparti. Puis, il met fin au travail et signale toutes les tâches terminées, mais pouvant avoir subi une défaillance, telle qu’un code de sortie différent de zéro.

```csharp
TaskStateMonitor taskStateMonitor = batchClient.Utilities.CreateTaskStateMonitor();
try
{
    await taskStateMonitor.WhenAll(addedTasks, TaskState.Completed, timeout);
}
catch (TimeoutException)
{
    batchClient.JobOperations.TerminateJob(jobId);
    Console.WriteLine(incompleteMessage);
    return false;
}
batchClient.JobOperations.TerminateJob(jobId);
 Console.WriteLine(completeMessage);
...

```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir exécuté les tâches, l’application supprime automatiquement le conteneur de stockage d’entrée créé et vous donne la possibilité de supprimer le travail et le pool Azure Batch. Les classes [JobOperations](/dotnet/api/microsoft.azure.batch.batchclient.joboperations) et [PoolOperations](/dotnet/api/microsoft.azure.batch.batchclient.pooloperations) de BatchServiceClient disposent toutes deux de méthodes de suppression correspondantes, appelées si l’utilisateur confirme la suppression. Bien que vous ne soyez pas facturé pour les travaux et les tâches à proprement parler, les nœuds de calcul vous sont facturés. Par conséquent, nous vous conseillons d’affecter les pools uniquement en fonction des besoins. Lorsque vous supprimez le pool, toutes les sorties de tâche sur les nœuds sont supprimées. Toutefois, les fichiers de sortie restent dans le compte de stockage.

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le compte Batch et le compte de stockage. Pour ce faire, dans le portail Azure, sélectionnez le groupe de ressources pour le compte Batch, puis cliquez sur **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter un package d’application à votre compte Batch
> * S’authentifier avec des comptes de stockage et Batch
> * Charger des fichiers d’entrée sur le stockage
> * Créer un pool de nœuds de calcul pour exécuter une application
> * Créer un travail et des tâches pour traiter les fichiers d’entrée
> * Surveiller l’exécution d’une tâche
> * Récupérer les fichiers de sortie

Pour plus d’exemples d’utilisation de l’API .NET pour planifier et traiter les charges de travail Batch, consultez les exemples sur GitHub.

> [!div class="nextstepaction"]
> [Exemples C# Batch](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp)
