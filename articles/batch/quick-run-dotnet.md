---
title: 'Démarrage rapide : exécution de votre premier travail Microsoft Azure Batch avec l’API .NET'
description: Dans ce guide de démarrage rapide, vous exécutez rapidement un exemple de travail Azure Batch et ses tâches à partir d’une application C# en utilisant la bibliothèque cliente .NET de Batch.
ms.topic: quickstart
ms.date: 08/17/2020
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 9087261865b07d32afab65012e04ea478cb60611
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88921645"
---
# <a name="quickstart-run-your-first-azure-batch-job-with-the-net-api"></a>Démarrage rapide : exécution de votre premier travail Microsoft Azure Batch avec l’API .NET

Démarrez avec Azure Batch en exécutant un travail d’une application C# basée sur l’API .NET Azure Batch. L’application télécharge plusieurs fichiers de données d’entrée vers Stockage Azure, puis crée un pool de nœuds de calcul Azure Batch (machines virtuelles). Ensuite, elle crée un exemple de travail qui exécute des tâches pour traiter chaque fichier d’entrée sur le pool à l’aide d’une commande de base.

À l’issue de ce démarrage rapide, vous maîtriserez les concepts clés du service Batch et serez prêt à essayer Azure Batch avec des charges de travail plus réalistes à plus grande échelle.

![Diagramme montrant une vue d’ensemble du flux de travail de l’application Azure Batch.](./media/quick-run-dotnet/sampleapp.png)

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- Un compte Batch et un compte Stockage Azure lié. Pour créer ces comptes, consultez les démarrages rapides Azure Batch à l’aide du [portail Azure](quick-create-portal.md) ou de l’[interface de ligne de commande Azure](quick-create-cli.md).

- [Visual Studio 2017 ou version ultérieure](https://www.visualstudio.com/vs), ou [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1) pour Linux, macOS ou Windows. 

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-the-sample"></a>Télécharger l’exemple

[Téléchargez ou clonez l’exemple d’application](https://github.com/Azure-Samples/batch-dotnet-quickstart) à partir de GitHub. Pour cloner le référentiel d’exemple d’application avec un client Git, utilisez la commande suivante :

```
git clone https://github.com/Azure-Samples/batch-dotnet-quickstart.git
```

Naviguez vers le répertoire qui contient le fichier de la solution Visual Studio `BatchDotNetQuickstart.sln`.

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

## <a name="build-and-run-the-app"></a>Générer et exécuter l’application

Pour afficher le flux de travail Batch en action, générer et exécuter l’application dans Visual Studio ou sur la ligne de commande avec les commandes `dotnet build` et `dotnet run`. Après avoir exécuté l’application, passez en revue le code pour savoir ce que fait chaque partie de l’application. Par exemple, dans Visual Studio :

- Cliquez avec le bouton droit dans l’Explorateur de solutions, puis cliquez sur **Générer la solution**. 

- Si vous y êtes invité, confirmez la restauration de tous les packages NuGet. Si vous devez télécharger des packages manquants, vérifiez que le [Gestionnaire de Package NuGet](https://docs.nuget.org/consume/installing-nuget) est installé.

Lorsque vous exécutez l’exemple d’application, la sortie de la console est identique à ce qui suit. Pendant l’exécution, l’étape `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` fait l’objet d’une pause correspondant au démarrage des nœuds de calcul du pool. Les tâches sont mises en file d’attente pour s’exécuter dès que le premier nœud de calcul est en cours d’exécution. Accédez à votre compte Batch dans le [portail Azure](https://portal.azure.com) pour surveiller le pool, les nœuds de calcul, les travaux et les tâches.

```
Sample start: 11/16/2018 4:02:54 PM

Container [input] created.
Uploading file taskdata0.txt to container [input]...
Uploading file taskdata1.txt to container [input]...
Uploading file taskdata2.txt to container [input]...
Creating pool [DotNetQuickstartPool]...
Creating job [DotNetQuickstartJob]...
Adding 3 tasks to job [DotNetQuickstartJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
```

Une fois que les tâches sont terminées, vous devez obtenir un résultat similaire à ce qui suit pour chaque tâche :

```
Printing task output.
Task: Task0
Node: tvm-2850684224_3-20171205t000401z
Standard out:
Batch processing began with mainframe computers and punch cards. Today it still plays a central role in business, engineering, science, and other pursuits that require running lots of automated tasks....
stderr:
...
```

Le temps d’exécution standard de l’application est d’environ 5 minutes lorsque l’application fonctionne dans sa configuration par défaut. La configuration du pool initial prend le plus de temps. Pour réexécuter le travail, supprimez le travail de l’exécution précédente et ne supprimez pas le pool. Sur un pool préconfiguré, le travail se termine en quelques secondes.

## <a name="review-the-code"></a>Vérifier le code

L’application .NET dans ce démarrage rapide effectue les opérations suivantes :

- Chargement de trois petits fichiers texte dans un conteneur blob sur votre compte de stockage Azure. Ces fichiers sont des entrées pour le traitement par Azure Batch.
- Création d’un pool de nœuds de calcul exécutant Windows Server.
- Création d’un travail et de trois tâches à exécuter sur les nœuds. Chaque tâche traite l’un des fichiers d’entrée à l’aide d’une ligne de commande Windows. 
- Affichage de fichiers retournés par les tâches.

Consultez le fichier `Program.cs` et les sections suivantes pour plus de détails.

### <a name="preliminaries"></a>Étapes préalables

Pour interagir avec un compte de stockage, l’application utilise la bibliothèque cliente de Stockage Azure pour .NET. Elle crée une référence au compte avec [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount), et à partir de là elle crée un [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient).

```csharp
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

L’application utilise la référence `blobClient` pour créer un conteneur dans le compte de stockage ainsi que pour charger des fichiers de données vers le conteneur. Les fichiers de stockage sont définis en tant qu’objets Batch [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile) que Batch peut télécharger ultérieurement sur les nœuds de calcul.

```csharp
List<string> inputFilePaths = new List<string>
{
    "taskdata0.txt",
    "taskdata1.txt",
    "taskdata2.txt"
};

List<ResourceFile> inputFiles = new List<ResourceFile>();

foreach (string filePath in inputFilePaths)
{
    inputFiles.Add(UploadFileToContainer(blobClient, inputContainerName, filePath));
}
```

L’application crée un objet [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) pour créer et gérer des pools, des travaux et des tâches dans le service Batch. Le client Batch dans l’exemple utilise l’authentification de la clé partagée. Batch prend également en charge l’authentification Azure Active Directory.

```csharp
BatchSharedKeyCredentials cred = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(cred))
...
```

### <a name="create-a-pool-of-compute-nodes"></a>Créer un pool de nœuds de calcul

Pour créer un pool Batch, l’application utilise la méthode [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) pour définir le nombre de nœuds, la taille de machine virtuelle et une configuration de pool. Ici, un objet [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) spécifie une référence [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) à une image Windows Server publiée dans la Place de marché Microsoft Azure. Azure Batch prend en charge une large plage d’images Linux et Windows Server dans la Place de marché Microsoft Azure, ainsi que des images de machines virtuelles personnalisées.

Le nombre de nœuds (`PoolNodeCount`) et la taille de machine virtuelle (`PoolVMSize`) sont des constantes définies. L’exemple par défaut crée un pool de 2 nœuds de taille *Standard_A1_v2*. La taille suggérée offre un bon compromis entre performances et coûts pour cet exemple rapide.

La méthode [Commit](/dotnet/api/microsoft.azure.batch.cloudpool.commit) soumet le pool au service Batch.

```csharp

private static VirtualMachineConfiguration CreateVirtualMachineConfiguration(ImageReference imageReference)
{
    return new VirtualMachineConfiguration(
        imageReference: imageReference,
        nodeAgentSkuId: "batch.node.windows amd64");
}

private static ImageReference CreateImageReference()
{
    return new ImageReference(
        publisher: "MicrosoftWindowsServer",
        offer: "WindowsServer",
        sku: "2016-datacenter-smalldisk",
        version: "latest");
}

private static void CreateBatchPool(BatchClient batchClient, VirtualMachineConfiguration vmConfiguration)
{
    try
    {
        CloudPool pool = batchClient.PoolOperations.CreatePool(
            poolId: PoolId,
            targetDedicatedComputeNodes: PoolNodeCount,
            virtualMachineSize: PoolVMSize,
            virtualMachineConfiguration: vmConfiguration);

        pool.Commit();
    }
...

```

### <a name="create-a-batch-job"></a>Création d’un travail Batch

Un travail Batch est un regroupement logique d’une ou de plusieurs tâches. Un travail inclut les paramètres communs aux tâches, tels que la priorité et le pool pour exécuter des tâches. L’application utilise la méthode [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) pour créer un travail sur votre pool.

La méthode [Commit](/dotnet/api/microsoft.azure.batch.cloudjob.commit) soumet le travail au service Batch. Dans un premier temps, le travail n’a aucune tâche.

```csharp
try
{
    CloudJob job = batchClient.JobOperations.CreateJob();
    job.Id = JobId;
    job.PoolInformation = new PoolInformation { PoolId = PoolId };

    job.Commit();
}
...
```

### <a name="create-tasks"></a>Créer des tâches

L’application crée une liste d’objets [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask). Chaque tâche traite un objet d’entrée `ResourceFile` à l’aide d’une propriété [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline). Dans l’exemple, la ligne de commande exécute la commande Windows `type` afin d’afficher le fichier d’entrée. Cette commande est un exemple simple à des fins de démonstration. Lorsque vous utilisez Azure Batch, la ligne de commande se trouve là où vous spécifiez votre application ou votre script. Azure Batch fournit plusieurs façons de déployer des applications et des scripts sur des nœuds de calcul.

Ensuite, l’application ajoute des tâches au travail avec la méthode [AddTask](/dotnet/api/microsoft.azure.batch.joboperations.addtask), qui les met en file d’attente afin de les exécuter sur les nœuds de calcul.

```csharp
for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);
    string inputFilename = inputFiles[i].FilePath;
    string taskCommandLine = String.Format("cmd /c type {0}", inputFilename);

    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };
    tasks.Add(task);
}

batchClient.JobOperations.AddTask(JobId, tasks);
```

### <a name="view-task-output"></a>Afficher la sortie des tâches

L’application crée un [TaskStateMonitor](/dotnet/api/microsoft.azure.batch.taskstatemonitor) pour surveiller les tâches et s’assurer qu’elles se terminent. L’application utilise ensuite la propriété [CloudTask.ComputeNodeInformation](/dotnet/api/microsoft.azure.batch.cloudtask.computenodeinformation) pour afficher le fichier `stdout.txt` généré par chaque tâche terminée. Lorsque la tâche s’exécute avec succès, la sortie de la commande de tâche est écrite dans `stdout.txt` :

```csharp
foreach (CloudTask task in completedtasks)
{
    string nodeId = String.Format(task.ComputeNodeInformation.ComputeNodeId);
    Console.WriteLine("Task: {0}", task.Id);
    Console.WriteLine("Node: {0}", nodeId);
    Console.WriteLine("Standard out:");
    Console.WriteLine(task.GetNodeFile(Constants.StandardOutFileName).ReadAsString());
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

L’application supprime automatiquement le conteneur de stockage créé et vous donne la possibilité de supprimer le travail et le pool Azure Batch. Vous êtes facturé pour le pool pendant que les nœuds sont en cours d’exécution, même si aucun travail n’est planifié. Lorsque vous n’avez plus besoin du pool, supprimez-le. Lorsque vous supprimez le pool, toutes les sorties de tâche sur les nœuds sont supprimées.

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le compte Batch et le compte de stockage. Pour ce faire, dans le portail Azure, sélectionnez le groupe de ressources pour le compte Batch, puis cliquez sur **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez exécuté une petite application générée à l’aide de l’API .NET Batch pour créer un pool et un travail Batch. Le travail a exécuté des tâches d’exemple et téléchargé des sorties créées sur les nœuds. Maintenant que vous maîtriserez les concepts clés du service Batch, vous êtres prêt à essayer Azure Batch avec des charges de travail plus réalistes à plus grande échelle. Pour en savoir plus sur Azure Batch et parcourir une charge de travail parallèle avec une application réelle, accédez au didacticiel .NET Batch.

> [!div class="nextstepaction"]
> [Traitement d’une charge de travail parallèle avec .NET](tutorial-parallel-dotnet.md)
