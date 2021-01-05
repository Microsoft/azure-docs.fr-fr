---
title: Superviser Batch avec Azure Application Insights
description: Découvrez comment instrumenter une application .NET Azure Batch avec la bibliothèque Azure Application Insights.
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 04/05/2018
ms.openlocfilehash: d06e2b61725f05d025acd8a2995ea041f138ae4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88933560"
---
# <a name="monitor-and-debug-an-azure-batch-net-application-with-application-insights"></a>Surveiller et déboguer une application .NET Azure Batch avec Application Insights

[Application Insights](../azure-monitor/app/app-insights-overview.md) est un moyen élégant et puissant permettant aux développeurs de surveiller et de déboguer les applications déployées sur les services Azure. Utilisez Application Insights pour surveiller les compteurs de performances et les exceptions, ainsi que pour instrumenter votre code avec un suivi et des métriques personnalisés. L’intégration d’Application Insights dans votre application Azure Batch vous permet d’obtenir des informations détaillées sur les comportements et d’étudier les problèmes en quasi temps réel.

Cet article explique comment ajouter et configurer la bibliothèque Application Insights dans votre solution .NET Azure Batch et instrumenter le code de votre application. Il montre également comment surveiller votre application à l’aide du portail Azure et créer des tableaux de bord personnalisés. Pour la prise en charge d’Application Insights dans d’autres langages, consultez [Analyse développeur : langages, plateformes et intégrations](../azure-monitor/app/platforms.md).

Un exemple de solution C# avec du code pour accompagner cet article est disponible sur [GitHub](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ApplicationInsights). Cet exemple ajoute le code d’instrumentation Application Insights dans l’exemple [TopNWords](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords). Si vous n’êtes pas familier avec cet exemple, essayez d’abord de générer et d’exécuter TopNWords. Cela vous aidera à comprendre le flux de traitement Batch d’un ensemble de blobs d’entrée, en parallèle sur plusieurs nœuds de calcul. 

> [!TIP]
> Vous pouvez également configurer votre solution Batch pour afficher des données Application Insights, comme des compteurs de performances de machines virtuelles dans Batch Explorer. [Batch Explorer](https://github.com/Azure/BatchExplorer) est un outil client autonome gratuit, doté de nombreuses fonctionnalités aidant à créer, déboguer et surveiller les applications Azure Batch. Téléchargez un [package d’installation](https://azure.github.io/BatchExplorer/) pour Mac, Linux ou Windows. Consultez le [référentiel batch-insights](https://github.com/Azure/batch-insights) pour obtenir les étapes à suivre pour activer les données Application Insights dans Batch Explorer. 
>

## <a name="prerequisites"></a>Prérequis
* [Visual Studio 2017 ou version ultérieure](https://www.visualstudio.com/vs)

* [Compte Batch et compte de stockage lié](batch-account-create-portal.md)

* [Ressource Application Insights](../azure-monitor/app/create-new-resource.md )
  
   * Utilisez le portail Azure pour créer une *ressource* Application Insights. Sélectionnez le **Type d’application** *Général*.

   * Copiez la [clé d’instrumentation](../azure-monitor/app/create-new-resource.md #copy-the-instrumentation-key) à partir du portail. Vous en aurez besoin plus loin dans cet article.
  
  > [!NOTE]
  > Les données stockées dans Application Insights peuvent faire l’objet d’une [facturation](https://azure.microsoft.com/pricing/details/application-insights/). Cela inclut les données de diagnostic et de surveillance traitées dans cet article.
  > 

## <a name="add-application-insights-to-your-project"></a>Ajout de Application Insights à votre projet

Votre projet requiert le package NuGet **Microsoft.ApplicationInsights.WindowsServer** et ses dépendances. Ajoutez-les ou restaurez-les dans le projet de votre application. Pour installer le package, utilisez la commande `Install-Package` ou le Gestionnaire de package NuGet.

```powershell
Install-Package Microsoft.ApplicationInsights.WindowsServer
```
Référencez Application Insights à partir de votre application .NET à l’aide de l’espace de noms **Microsoft.ApplicationInsights**.

## <a name="instrument-your-code"></a>instrumentalisation de votre code

Pour instrumenter votre code, votre solution doit créer un [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient) Application Insights. Dans l’exemple, le TelemetryClient charge sa configuration à partir du fichier [ApplicationInsights.config](../azure-monitor/app/configuration-with-applicationinsights-config.md). Veillez à mettre à jour ApplicationInsights.config dans les projets suivants avec votre clé d’instrumentation Application Insights : Microsoft.Azure.Batch.Samples.TelemetryStartTask et TopNWordsSample.

```xml
<InstrumentationKey>YOUR-IKEY-GOES-HERE</InstrumentationKey>
```
Ajoutez également la clé d’instrumentation dans le fichier TopNWords.cs.

L’exemple dans TopNWords.cs utilise les [appels d’instrumentation](../azure-monitor/app/api-custom-events-metrics.md) suivants de l’API Application Insights :
* `TrackMetric()` : surveille la durée moyenne que prend un nœud de calcul pour télécharger le fichier texte requis.
* `TrackTrace()` : ajoute des appels de débogage dans votre code.
* `TrackEvent()` : surveille les événements intéressants à capturer.

Cet exemple omet volontairement la gestion des exceptions. En revanche, Application Insights signale automatiquement les exceptions non prises en charge, ce qui améliore considérablement l’expérience de débogage. 

L’extrait de code suivant illustre l’utilisation de ces méthodes.

```csharp
public void CountWords(string blobName, int numTopN, string storageAccountName, string storageAccountKey)
{
    // simulate exception for some set of tasks
    Random rand = new Random();
    if (rand.Next(0, 10) % 10 == 0)
    {
        blobName += ".badUrl";
    }

    // log the url we are downloading the file from
    insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Download file from: {1}", this.taskId, blobName), SeverityLevel.Verbose));

    // open the cloud blob that contains the book
    var storageCred = new StorageCredentials(storageAccountName, storageAccountKey);
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(blobName), storageCred);
    using (Stream memoryStream = new MemoryStream())
    {
        // calculate blob download time
        DateTime start = DateTime.Now;
        blob.DownloadToStream(memoryStream);
        TimeSpan downloadTime = DateTime.Now.Subtract(start);

        // track how long the blob takes to download on this node
        // this will help debug timing issues or identify poorly performing nodes
        insightsClient.TrackMetric("Blob download in seconds", downloadTime.TotalSeconds, this.CommonProperties);

        memoryStream.Position = 0; //Reset the stream
        var sr = new StreamReader(memoryStream);
        var myStr = sr.ReadToEnd();
        string[] words = myStr.Split(' ');

        // log how many words were found in the text file
        insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Found {1} words", this.taskId, words.Length), SeverityLevel.Verbose));
        var topNWords =
            words.
                Where(word => word.Length > 0).
                GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
                OrderByDescending(x => x.Value).
                Take(numTopN).
                ToList();
        foreach (var pair in topNWords)
        {
            Console.WriteLine("{0} {1}", pair.Key, pair.Value);
        }

        // emit an event to track the completion of the task
        insightsClient.TrackEvent("Done counting words");
    }
}
```

### <a name="azure-batch-telemetry-initializer-helper"></a>Assistance de l’initialiseur de télémétrie Azure Batch
Lors du signalement des données de télémétrie d’un serveur particulier ou d’une instance donnée, Application Insights utilise le rôle de la machine virtuelle Azure et le nom de la machine virtuelle comme valeurs par défaut. Dans le cadre d’Azure Batch, l’exemple montre comment utiliser le nom du pool et le nom du nœud de calcul à la place. Utilisez un [initialiseur de télémétrie](../azure-monitor/app/api-filtering-sampling.md#add-properties) pour remplacer les valeurs par défaut. 

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;
using System;
using System.Threading;

namespace Microsoft.Azure.Batch.Samples.TelemetryInitializer
{
    public class AzureBatchNodeTelemetryInitializer : ITelemetryInitializer
    {
        // Azure Batch environment variables
        private const string PoolIdEnvironmentVariable = "AZ_BATCH_POOL_ID";
        private const string NodeIdEnvironmentVariable = "AZ_BATCH_NODE_ID";

        private string roleInstanceName;
        private string roleName;

        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
            {
                // override the role name with the Azure Batch Pool name
                string name = LazyInitializer.EnsureInitialized(ref this.roleName, this.GetPoolName);
                telemetry.Context.Cloud.RoleName = name;
            }

            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // override the role instance with the Azure Batch Compute Node name
                string name = LazyInitializer.EnsureInitialized(ref this.roleInstanceName, this.GetNodeName);
                telemetry.Context.Cloud.RoleInstance = name;
            }
        }

        private string GetPoolName()
        {
            return Environment.GetEnvironmentVariable(PoolIdEnvironmentVariable) ?? string.Empty;
        }

        private string GetNodeName()
        {
            return Environment.GetEnvironmentVariable(NodeIdEnvironmentVariable) ?? string.Empty;
        }
    }
}
```

Pour activer l’initialiseur de télémétrie, le fichier ApplicationInsights.config du projet TopNWordsSample inclut les éléments suivants :

```xml
<TelemetryInitializers>
    <Add Type="Microsoft.Azure.Batch.Samples.TelemetryInitializer.AzureBatchNodeTelemetryInitializer, Microsoft.Azure.Batch.Samples.TelemetryInitializer"/>
</TelemetryInitializers>
``` 

## <a name="update-the-job-and-tasks-to-include-application-insights-binaries"></a>Mettre à jour le projet et les tâches pour inclure les fichiers binaires d’Application Insights

Pour qu’Application Insights s’exécute correctement sur vos nœuds de calcul, assurez-vous que les fichiers binaires sont placés à leur emplacement correct. Ajoutez les fichiers binaires requis dans la collection de fichiers de ressources de votre tâche afin qu’ils soient téléchargés lors de l’exécution de votre tâche. Les extraits de code suivants sont similaires au code dans Job.cs.

Commencez par créer une liste statique des fichiers d’Application Insights à télécharger.

```csharp
private static readonly List<string> AIFilesToUpload = new List<string>()
{
    // Application Insights config and assemblies
    "ApplicationInsights.config",
    "Microsoft.ApplicationInsights.dll",
    "Microsoft.AI.Agent.Intercept.dll",
    "Microsoft.AI.DependencyCollector.dll",
    "Microsoft.AI.PerfCounterCollector.dll",
    "Microsoft.AI.ServerTelemetryChannel.dll",
    "Microsoft.AI.WindowsServer.dll",
    
    // custom telemetry initializer assemblies
    "Microsoft.Azure.Batch.Samples.TelemetryInitializer.dll",
 };
...
```

Ensuite, créez les fichiers intermédiaires qui sont utilisés par la tâche.
```csharp
...
// create file staging objects that represent the executable and its dependent assembly to run as the task.
// These files are copied to every node before the corresponding task is scheduled to run on that node.
FileToStage topNWordExe = new FileToStage(TopNWordsExeName, stagingStorageAccount);
FileToStage storageDll = new FileToStage(StorageClientDllName, stagingStorageAccount);

// Upload Application Insights assemblies
List<FileToStage> aiStagedFiles = new List<FileToStage>();
foreach (string aiFile in AIFilesToUpload)
{
    aiStagedFiles.Add(new FileToStage(aiFile, stagingStorageAccount));
}
...
```

La méthode `FileToStage` est une fonction d’assistance dans l’exemple de code, qui vous permet de charger un fichier facilement à partir du disque local dans un blob Stockage Azure. Chaque fichier est ensuite téléchargé sur un nœud de calcul et référencé par une tâche.

Enfin, ajoutez les tâches au travail et incluez les fichiers binaires d’Application Insights nécessaires.
```csharp
...
// initialize a collection to hold the tasks that will be submitted in their entirety
List<CloudTask> tasksToRun = new List<CloudTask>(topNWordsConfiguration.NumberOfTasks);
for (int i = 1; i <= topNWordsConfiguration.NumberOfTasks; i++)
{
    CloudTask task = new CloudTask("task_no_" + i, String.Format("{0} --Task {1} {2} {3} {4}",
        TopNWordsExeName,
        string.Format("https://{0}.blob.core.windows.net/{1}",
            accountSettings.StorageAccountName,
            documents[i]),
        topNWordsConfiguration.TopWordCount,
        accountSettings.StorageAccountName,
        accountSettings.StorageAccountKey));

    //This is the list of files to stage to a container -- for each job, one container is created and 
    //files all resolve to Azure Blobs by their name (so two tasks with the same named file will create just 1 blob in
    //the container).
    task.FilesToStage = new List<IFileStagingProvider>
                        {
                            // required application binaries
                            topNWordExe,
                            storageDll,
                        };
    foreach (FileToStage stagedFile in aiStagedFiles)
   {
        task.FilesToStage.Add(stagedFile);
   }    
    task.RunElevated = false;
    tasksToRun.Add(task);
}
```

## <a name="view-data-in-the-azure-portal"></a>Afficher les données dans le portail Azure

Maintenant que vous avez configuré le travail et les tâches pour utiliser Application Insights, exécutez l’exemple de travail dans votre pool. Accédez au portail Azure et ouvrez la ressource Application Insights que vous avez approvisionnée. Une fois le pool approvisionné, vous devez démarrer pour voir le flux de données et leur journalisation. Le reste de cet article n’aborde que certaines fonctionnalités d’Application Insights, mais n’hésitez pas à explorer toutes les autres.

### <a name="view-live-stream-data"></a>Afficher les données du flux temps réel

Pour afficher les journaux d’activité de suivi dans votre ressource Applications Insights, cliquez sur **Flux temps réel**. La capture d’écran suivante montre comment afficher les données temps réel provenant des nœuds de calcul dans le pool, comme l’utilisation du processeur par nœud de calcul.

![Flux temps réel des données des nœuds de calcul](./media/monitor-application-insights/applicationinsightslivestream.png)

### <a name="view-trace-logs"></a>Afficher les journaux d’activité de suivi

Pour afficher les journaux d’activité de suivi dans votre ressource Applications Insights, cliquez sur **Rechercher**. Cette vue affiche une liste des données de diagnostic capturées par Application Insights, avec les traces, les événements et les exceptions. 

La capture d’écran suivante montre comment une trace d’une tâche est journalisée puis interrogée à des fins de débogage.

![Image des journaux d’activité de suivi](./media/monitor-application-insights/tracelogsfortask.png)

### <a name="view-unhandled-exceptions"></a>Afficher les exceptions non prises en charge

Les captures d’écran suivantes montrent comment Application Insights journalise les exceptions générées par votre application. Dans ce cas, quelques secondes après la génération de l’exception par l’application, vous pouvez analyser une exception et diagnostiquer le problème.

![Exceptions non prises en charge](./media/monitor-application-insights/exception.png)

### <a name="measure-blob-download-time"></a>Mesurer le temps de téléchargement des blobs

Les métriques personnalisées sont également un outil précieux dans le portail. Par exemple, vous pouvez afficher la durée moyenne nécessaire à chaque nœud de calcul pour télécharger le fichier texte requis qu’il traitait.

Pour créer un exemple de graphique :
1. Dans votre ressource Application Insights, cliquez sur **Metrics Explorer** > **Ajouter un graphique**.
2. Cliquez sur **Modifier** sur le graphique ajouté.
2. Mettez à jour les détails du graphique comme suit :
   * Dans **Type de graphique**, sélectionnez **Grille**.
   * Dans **Agrégation**, sélectionnez **Moyenne**.
   * Dans **Group by (Grouper par)** , sélectionnez **NodeId (ID de nœud)** .
   * Dans **Métriques**, sélectionnez **Personnalisé** > **Blob download in seconds (Téléchargement de blocs en secondes)** .
   * Ajustez la **palette de couleurs** de l’affichage. 

![Temps de téléchargement des blobs par nœuds](./media/monitor-application-insights/blobdownloadtime.png)


## <a name="monitor-compute-nodes-continuously"></a>Surveiller les nœuds de calcul en permanence

Vous avez peut-être remarqué que toutes les métriques, notamment les compteurs de performance, ne sont journalisées que lorsque les tâches sont en cours d’exécution. Ce comportement est utile car il limite le volume de données journalisées par Application Insights. Toutefois, vous souhaiterez parfois surveiller les nœuds de calcul en continu. Par exemple, ils peuvent exécuter des travaux d’arrière-plan qui ne sont pas planifiés par le service Batch. Dans ce cas, configurez un processus de surveillance qui s’exécute pendant toute la durée de vie du nœud de calcul. 

Pour obtenir ce comportement, vous pouvez notamment générer un processus qui charge la bibliothèque Application Insights et s’exécute en arrière-plan. Dans l’exemple, la tâche de démarrage charge les fichiers binaires sur l’ordinateur et conserve un processus en exécution permanente. Configurez le fichier de configuration d’Application Insights pour que ce processus émette les données supplémentaires qui vous intéressent, comme les compteurs de performances.

```csharp
...
 // Batch start task telemetry runner
private const string BatchStartTaskFolderName = "StartTask";
private const string BatchStartTaskTelemetryRunnerName = "Microsoft.Azure.Batch.Samples.TelemetryStartTask.exe";
private const string BatchStartTaskTelemetryRunnerAIConfig = "ApplicationInsights.config";
...
CloudPool pool = client.PoolOperations.CreatePool(
    topNWordsConfiguration.PoolId,
    targetDedicated: topNWordsConfiguration.PoolNodeCount,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));
...

// Create a start task which will run a dummy exe in background that simply emits performance
// counter data as defined in the relevant ApplicationInsights.config.
// Note that the waitForSuccess on the start task was not set so the Compute Node will be
// available immediately after this command is run.
pool.StartTask = new StartTask()
{
    CommandLine = string.Format("cmd /c {0}", BatchStartTaskTelemetryRunnerName),
    ResourceFiles = resourceFiles
};
...
```

> [!TIP]
> Pour augmenter la facilité de gestion de votre solution, vous pouvez regrouper l’assembly dans un [package d’application](./batch-application-packages.md). Ensuite, pour déployer le package d’application automatiquement sur vos pools, ajoutez une référence de package d’application dans la configuration du pool.
>

## <a name="throttle-and-sample-data"></a>Limiter et échantillonner les données 

En raison de l’envergure des applications Azure Batch exécutées en production, vous pouvez limiter la quantité de données collectées par Application Insights pour gérer les coûts. Pour plus d’informations sur certains de ces mécanismes, consultez [Échantillonnage dans Application Insights](../azure-monitor/app/sampling.md).


## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur [Application Insights](../azure-monitor/app/app-insights-overview.md).

* Pour la prise en charge d’Application Insights dans d’autres langages, consultez [Analyse développeur : langages, plateformes et intégrations](../azure-monitor/app/platforms.md).


