---
title: Créer des tâches pour préparer et exécuter des travaux sur des nœuds de calcul
description: Utilisez des tâches de préparation au niveau du travail afin de minimiser le transfert de données vers les nœuds de calcul Azure Batch, et utilisez des tâches de validation pour le nettoyage des nœuds une fois le travail achevé.
ms.topic: how-to
ms.date: 02/17/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 5b1084cfdd5995b7983badcdce71460f7bdec3d5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88919452"
---
# <a name="run-job-preparation-and-job-release-tasks-on-batch-compute-nodes"></a>Exécuter des tâches de préparation et de validation du travail sur les nœuds de calcul Batch

 Un travail Azure Batch nécessite souvent une certaine forme de préparation avant l’exécution de ses tâches, ainsi qu’une maintenance ultérieure une fois les tâches terminées. Vous pouvez avoir besoin de télécharger les données d’entrée de tâche communes dans vos nœuds de calcul, ou de charger les données de sortie de tâche dans le stockage Azure une fois le travail terminé. Vous pouvez effectuer ces opérations à l’aide des tâches de **préparation du travail** et de **validation du travail**.

## <a name="what-are-job-preparation-and-release-tasks"></a>En quoi consistent les tâches de préparation et de validation du travail ?
Avant l’exécution des tâches d’un travail, la tâche de préparation du travail s’exécute sur tous les nœuds de calcul destinés à exécuter au moins l’une des tâches. Lorsqu'un travail est terminé, la tâche de validation du travail s'exécute sur chaque nœud dans le pool ayant exécuté au moins une tâche. Comme dans le cas des tâches Batch standard, vous pouvez spécifier une ligne de commande à appeler lors de l’exécution d’une tâche de préparation ou de validation du travail.

Les tâches de préparation et de validation du travail offrent des fonctionnalités de tâche Batch courantes, telles que téléchargement de fichiers ([fichiers de ressources][net_job_prep_resourcefiles]), exécution avec élévation de privilèges, variables d’environnement personnalisées, durée d’exécution maximale, nombre de tentatives et période de rétention des fichiers.

Dans les sections ci-après, vous découvrirez comment utiliser les classes [JobPreparationTask][net_job_prep] et [JobReleaseTask][net_job_release] disponibles dans la bibliothèque [Batch .NET][api_net].

> [!TIP]
> Les tâches de préparation et de validation du travail sont particulièrement utiles dans les environnements de « pool partagé », dans lesquels un pool de nœuds de calcul persiste entre les exécutions d’un travail et est utilisé par de nombreux travaux.
> 
> 

## <a name="when-to-use-job-preparation-and-release-tasks"></a>Utilisation des tâches de préparation et de validation du travail
Les tâches de préparation et de validation du travail sont parfaitement adaptées aux opérations suivantes :

**Télécharger les données de tâche communes**

Les travaux Batch nécessitent souvent un ensemble commun de données comme entrée pour les tâches du travail. Par exemple, dans les calculs quotidiens de l’analyse des risques, les données de marché sont propres à un travail, mais communes à toutes les tâches de ce travail. Ces données de marché, dont la taille atteint souvent plusieurs gigaoctets, ne doivent être téléchargées qu’une seule fois dans chaque nœud de calcul pour être utilisables par toutes les tâches qui s’exécutent sur un nœud. Utilisez une **tâche de préparation du travail** pour télécharger ces données sur chaque nœud avant l’exécution des autres tâches du travail.

**Supprimer la sortie des travaux et des tâches**

Dans un environnement de « pool partagé » dans lequel les nœuds de calcul d’un pool ne sont pas désactivés entre les travaux, il peut être nécessaire de supprimer les données du travail entre les exécutions afin d’économiser de l’espace disque sur les nœuds ou de respecter les stratégies de sécurité de votre organisation. Utilisez une **tâche de validation du travail** pour supprimer les données téléchargées par une tâche de préparation du travail ou générées pendant l’exécution d’une tâche.

**Rétention des journaux**

Vous voulez peut-être conserver une copie des fichiers journaux générés par les tâches ou peut-être les fichiers de vidage sur incident qui peuvent être générés par les applications ayant échoué. Dans ces cas, utilisez une **tâche de mise en production du travail** pour compresser et télécharger ces données vers un compte de [Stockage Azure][azure_storage].

> [!TIP]
> Une autre façon de conserver les journaux d’activité et les autres données de sortie des travaux et des tâches consiste à utiliser la bibliothèque de [conventions de fichier Azure Batch](batch-task-output.md) .
>
>

## <a name="job-preparation-task"></a>tâche de préparation du travail


Avant l’exécution des tâches d’un travail, Batch exécute la tâche de préparation du travail sur chaque nœud de calcul sur lequel l’exécution d’une tâche est planifiée. Par défaut, Batch attend la fin de la tâche de préparation du travail avant d’exécuter les tâches planifiées pour s’exécuter sur le nœud. Toutefois, vous pouvez configurer le service pour qu'il n'attende pas. Si le nœud redémarre, la tâche de préparation du travail est réexécutée. Vous pouvez aussi désactiver ce comportement. Si vous avez configuré un travail avec une tâche de préparation du travail et une tâche du gestionnaire de travaux, la tâche de préparation du travail est exécutée avant la tâche du gestionnaire de travaux, tout comme avec toutes les autres tâches. La tâche de préparation du travail est toujours exécutée en premier.

La tâche de préparation du travail est uniquement exécutée sur les nœuds sur lesquels l’exécution d’une tâche est planifiée. Ceci empêche l'exécution d'une tâche de préparation inutile dans le cas où une tâche n'est pas attribuée à un nœud. Cette situation peut survenir lorsque le nombre de tâches pour un travail est inférieur au nombre de nœuds dans un pool. Elle s’applique également si [l’exécution de tâches simultanées](batch-parallel-node-tasks.md) est activée. Dans ce cas, certains nœuds restent inactifs si le nombre de tâches est inférieur au nombre total de tâches simultanées possibles. Lorsque vous n’exécutez pas la tâche de préparation du travail sur des nœuds inactifs, vous pouvez réduire vos frais de transfert de données.

> [!NOTE]
> [JobPreparationTask][net_job_prep_cloudjob] diffère de [CloudPool.StartTask][pool_starttask] dans la mesure où JobPreparationTask s’exécute au début de chaque travail, tandis que StartTask s’exécute uniquement lorsqu’un nœud de calcul rejoint un pool ou redémarre.
>


>## <a name="job-release-task"></a>tâche de validation du travail

Lorsqu'un travail est marqué comme terminé, la tâche de validation du travail s'exécute sur chaque nœud dans le pool ayant exécuté au moins une tâche. Vous marquez un travail comme terminé en émettant une requête de fin. Le service Batch définit ensuite l’état du travail sur *arrêt*, met fin à toutes les tâches actives ou en cours d’exécution associées au travail, puis exécute la tâche de validation du travail. Le travail passe ensuite à l'état *terminé* .

> [!NOTE]
> La suppression du travail exécute également la tâche de validation du travail. Toutefois, si un travail a déjà été arrêté, la tâche de validation n’est pas exécutée une seconde fois si ce travail est supprimé par la suite.

Les tâches de validation du travail peuvent s’exécuter pendant un maximum de 15 minutes avant d’être terminées par le service Batch. Pour plus d’informations, voir la [Documentation de référence sur l’API REST](/rest/api/batchservice/job/add#jobreleasetask).
> 
> 

## <a name="job-prep-and-release-tasks-with-batch-net"></a>Tâches de préparation et de validation du travail avec Batch.NET
Pour utiliser une tâche de préparation du travail, affectez un objet [JobPreparationTask][net_job_prep] à la propriété [CloudJob.JobPreparationTask][net_job_prep_cloudjob] de votre travail. De même, initialisez la propriété [JobReleaseTask][net_job_release] et affectez-la à la propriété [CloudJob.JobReleaseTask][net_job_prep_cloudjob] de votre travail pour définir la tâche de validation du travail.

Dans cet extrait de code, `myBatchClient` est une instance de [BatchClient][net_batch_client], et `myPool` est un pool existant dans le compte Batch.

```csharp
// Create the CloudJob for CloudPool "myPool"
CloudJob myJob =
    myBatchClient.JobOperations.CreateJob(
        "JobPrepReleaseSampleJob",
        new PoolInformation() { PoolId = "myPool" });

// Specify the command lines for the job preparation and release tasks
string jobPrepCmdLine =
    "cmd /c echo %AZ_BATCH_NODE_ID% > %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
string jobReleaseCmdLine =
    "cmd /c del %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";

// Assign the job preparation task to the job
myJob.JobPreparationTask =
    new JobPreparationTask { CommandLine = jobPrepCmdLine };

// Assign the job release task to the job
myJob.JobReleaseTask =
    new JobReleaseTask { CommandLine = jobReleaseCmdLine };

await myJob.CommitAsync();
```

Comme mentionné ci-dessus, la tâche de validation est exécutée lorsqu’un travail est arrêté ou supprimé. Pour arrêter un travail, utilisez [JobOperations.TerminateJobAsync][net_job_terminate]. Pour supprimer un travail, utilisez [JobOperations.DeleteJobAsync][net_job_delete]. Généralement, vous arrêtez ou supprimez un travail lorsque les tâches de ce dernier sont terminées ou qu’un délai d’expiration que vous avez défini a été atteint.

```csharp
// Terminate the job to mark it as Completed; this will initiate the
// Job Release Task on any node that executed job tasks. Note that the
// Job Release Task is also executed when a job is deleted, thus you
// need not call Terminate if you typically delete jobs after task completion.
await myBatchClient.JobOperations.TerminateJobAsync("JobPrepReleaseSampleJob");
```

## <a name="code-sample-on-github"></a>Exemple de code sur GitHub
Pour découvrir les tâches de préparation et de validation du travail en action, consultez l’exemple de projet [JobPrepRelease][job_prep_release_sample] sur GitHub. Cette application de console effectue les opérations suivantes :

1. Crée un pool avec deux nœuds.
2. Crée un travail avec des tâches de préparation du travail, de validation et standard.
3. Exécute la tâche de préparation du travail qui écrit d'abord l'ID de nœud dans un fichier texte dans le répertoire « partagé » d'un nœud.
4. Exécute une tâche sur chaque nœud qui écrit son ID de tâche dans le même fichier texte.
5. Lorsque toutes les tâches sont terminées (ou que le délai d'attente est atteint), imprime le contenu du fichier texte de chaque nœud dans la console.
6. Lorsque le travail est terminé, exécute la tâche de validation du travail pour supprimer le fichier du nœud.
7. Imprime les codes de sortie des tâches de préparation et de validation du travail pour chaque nœud sur lequel elles sont exécutées.
8. Interrompt l'exécution pour permettre la confirmation de la suppression du pool et/ou du travail.

Le résultat de l'exemple d'application ressemble à ce qui suit :

```
Attempting to create pool: JobPrepReleaseSamplePool
Created pool JobPrepReleaseSamplePool with 2 nodes
Checking for existing job JobPrepReleaseSampleJob...
Job JobPrepReleaseSampleJob not found, creating...
Submitting tasks and awaiting completion...
All tasks completed.

Contents of shared\job_prep_and_release.txt on tvm-2434664350_1-20160623t173951z:
-------------------------------------------
tvm-2434664350_1-20160623t173951z tasks:
  task001
  task004
  task005
  task006

Contents of shared\job_prep_and_release.txt on tvm-2434664350_2-20160623t173951z:
-------------------------------------------
tvm-2434664350_2-20160623t173951z tasks:
  task008
  task002
  task003
  task007

Waiting for job JobPrepReleaseSampleJob to reach state Completed
...

tvm-2434664350_1-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

tvm-2434664350_2-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

Delete job? [yes] no
yes
Delete pool? [yes] no
yes

Sample complete, hit ENTER to exit...
```

> [!NOTE]
> En raison de la variabilité des heures de création et de démarrage des nœuds dans un nouveau pool (certains nœuds sont prêts pour les tâches avant d’autres), vous risquez d’obtenir un résultat différent. En particulier, étant donné que les tâches s’exécutent rapidement, l’un des nœuds du pool peut exécuter l’ensemble des tâches du travail. Si cela se produit, vous remarquerez que les tâches de préparation et de validation du travail n’existent pas pour le nœud qui n’a exécuté aucune tâche.
> 
> 

### <a name="inspect-job-preparation-and-release-tasks-in-the-azure-portal"></a>Inspection des tâches de préparation et de validation du travail dans le Portail Azure
Lorsque vous exécutez l’exemple d’application, vous pouvez utiliser le [Portail Azure][portal] pour visualiser les propriétés du travail et ses tâches, ou même télécharger le fichier texte partagé modifié par les tâches du travail.

La capture d’écran ci-après illustre le **panneau Tâches de préparation** du Portail Azure après une exécution de l’exemple d’application. Accédez aux propriétés *JobPrepReleaseSampleJob* une fois les tâches terminées (mais avant la suppression de votre travail et du pool), puis cliquez sur **Tâches de préparation** ou sur **Tâches de fin** pour en visualiser les propriétés.

![Propriétés de préparation du travail dans le portail Azure][1]

## <a name="next-steps"></a>Étapes suivantes
### <a name="application-packages"></a>packages d’application
Outre la tâche de préparation du travail, vous pouvez également utiliser la fonctionnalité [packages d’application](batch-application-packages.md) de Batch pour préparer des nœuds de calcul à l’exécution de tâches. Cette fonctionnalité est particulièrement utile pour déployer des applications qui ne nécessitent pas de programme d’installation, des applications qui contiennent de nombreux fichiers (plus de 100) ou des applications qui requièrent un contrôle de version strict.

### <a name="installing-applications-and-staging-data"></a>Installation d’applications et de données intermédiaires
Le billet MSDN ci-après fournit une vue d’ensemble de différentes méthodes de préparation de vos nœuds à l’exécution des tâches :

[Installation d’applications et de données intermédiaires sur les nœuds de calcul Batch][forum_post]

Rédigé par l’un des membres de l’équipe Azure Batch, ce billet décrit plusieurs techniques que vous pouvez utiliser pour déployer des applications et des données sur les nœuds de calcul.

[api_net]: /dotnet/api/microsoft.azure.batch
[api_net_listjobs]: /dotnet/api/microsoft.azure.batch.joboperations
[api_rest]: /rest/api/batchservice/
[azure_storage]: https://azure.microsoft.com/services/storage/
[portal]: https://portal.azure.com
[job_prep_release_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease
[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[net_batch_client]: /dotnet/api/microsoft.azure.batch.batchclient
[net_cloudjob]:https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_prep]: /dotnet/api/microsoft.azure.batch.jobpreparationtask
[net_job_prep_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_job_prep_resourcefiles]: /dotnet/api/microsoft.azure.batch.jobpreparationtask
[net_job_delete]: /previous-versions/azure/mt281411(v=azure.100)
[net_job_terminate]: /previous-versions/azure/mt188985(v=azure.100)
[net_job_release]: /dotnet/api/microsoft.azure.batch.jobreleasetask
[net_job_release_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[pool_starttask]: /dotnet/api/microsoft.azure.batch.cloudpool

[net_list_certs]: /dotnet/api/microsoft.azure.batch.certificateoperations
[net_list_compute_nodes]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_job_schedules]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_jobprep_status]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_jobs]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_nodefiles]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_pools]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_schedule_jobs]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_task_files]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_list_tasks]: /dotnet/api/microsoft.azure.batch.joboperations

[1]: ./media/batch-job-prep-release/portal-jobprep-01.png
