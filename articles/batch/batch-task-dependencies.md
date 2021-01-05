---
title: Créer des dépendances de tâches pour exécuter des tâches
description: Créez des tâches qui dépendent de l’achèvement d’autres tâches pour le traitement de charges de travail de type MapReduce ou Big Data dans Azure Batch.
ms.topic: how-to
ms.date: 05/22/2017
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: 05b1bb289c215208be448d8ca7de144c82b313b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936977"
---
# <a name="create-task-dependencies-to-run-tasks-that-depend-on-other-tasks"></a>Créer des dépendances de tâches pour exécuter des tâches qui dépendent d’autres tâches

Vous pouvez définir des dépendances de tâche pour exécuter une tâche ou un ensemble de tâches uniquement après la fin d’une tâche parente. Voici quelques scénarios dont les dépendances de tâche sont utiles :

- des charges de travail MapReduce dans le cloud ;
- des travaux dont les tâches de traitement des données peuvent être exprimées sous la forme d’un graphe orienté acyclique (DAG) ;
- Processus de pré-rendu et de post-rendu dans lesquels chaque tâche doit s’achever avant que la tâche suivante puisse commencer.
- tout autre travail dont les tâches en aval dépendent de la sortie des tâches en amont.

Avec les dépendances de tâches Batch, vous pouvez créer des tâches dont l’exécution est planifiée sur des nœuds de calcul à condition qu’une ou plusieurs tâches parentes aient été exécutées. Par exemple, vous pouvez créer un travail qui restitue chaque image d’un film 3D avec des tâches parallèles distinctes. La dernière tâche (dite de fusion) fusionne les images restituées dans la vidéo complète uniquement après restitution de toutes les images.

Par défaut, les tâches dépendantes sont planifiées pour que leur exécution ait lieu uniquement après la fin de l’exécution de la tâche parente. Vous pouvez indiquer une action de dépendance afin de remplacer le comportement par défaut et exécuter des tâches en cas d’échec de la tâche parente. Pour plus d’informations, consultez la section [Actions de dépendance](#dependency-actions).  

Vous pouvez créer des tâches qui dépendent d’autres tâches dans une relation un-à-un ou un-à-plusieurs. Vous pouvez également créer une dépendance de plage dans laquelle une tâche dépend de l’exécution d’un groupe de tâches au sein d’une plage spécifiée d’ID de tâches. Vous pouvez combiner ces trois scénarios de base pour créer des relations plusieurs-à-plusieurs.

## <a name="task-dependencies-with-batch-net"></a>Dépendances de tâches avec Batch.NET

Cet article explique comment configurer les dépendances de tâches à l’aide de la bibliothèque [Batch .NET][net_msdn]. Nous allons tout d’abord vous montrer comment [activer la dépendance de tâches](#enable-task-dependencies) dans vos travaux, puis vous expliquer comment [configurer une tâche avec des dépendances](#create-dependent-tasks). Nous décrivons également comment spécifier une action de dépendance pour exécuter des tâches dépendantes en cas d’échec de la tâche parente. Pour finir, nous passerons en revue les [scénarios de dépendance](#dependency-scenarios) pris en charge par Batch.

## <a name="enable-task-dependencies"></a>Activation des dépendances de tâches

Pour utiliser les dépendances de tâches dans votre application Batch, vous devez d’abord configurer la tâche afin d’utiliser des dépendances de tâches. Dans Batch.NET, activez la dépendance de tâches sur votre [CloudJob][net_cloudjob] en définissant sa propriété [UsesTaskDependencies][net_usestaskdependencies] sur `true` :

```csharp
CloudJob unboundJob = batchClient.JobOperations.CreateJob( "job001",
    new PoolInformation { PoolId = "pool001" });

// IMPORTANT: This is REQUIRED for using task dependencies.
unboundJob.UsesTaskDependencies = true;
```

Dans l’extrait de code précédent, « batchClient » est une instance de la classe [BatchClient][net_batchclient].

## <a name="create-dependent-tasks"></a>Création de tâches dépendantes

Pour créer une tâche qui dépend de l’exécution d’une ou plusieurs tâches parentes, vous devez indiquer que la tâche « dépend » des autres tâches. Dans Batch.NET, configurez la propriété [CloudTask][net_cloudtask].[DependsOn][net_dependson] avec une instance de la classe [TaskDependencies][net_taskdependencies] :

```csharp
// Task 'Flowers' depends on completion of both 'Rain' and 'Sun'
// before it is run.
new CloudTask("Flowers", "cmd.exe /c echo Flowers")
{
    DependsOn = TaskDependencies.OnIds("Rain", "Sun")
},
```

Cet extrait de code crée une tâche dépendante avec l’ID de tâche « Flowers ». La tâche « Flowers » dépend des tâches « Rain » et « Sun ». La tâche « Flowers » est programmée pour s’exécuter sur un nœud de calcul uniquement après la réussite de l’exécution des tâches « Rain » et « Sun ».

> [!NOTE]
> Par défaut, une tâche est considérée comme réussie lorsqu’elle se trouve à l’état **terminé** et que son **code de sortie** est `0`. Dans Batch.NET, la valeur de propriété [CloudTask][net_cloudtask].[State][net_taskstate] doit être `Completed` et la valeur de propriété [TaskExecutionInformation][net_taskexecutioninformation].[ExitCode][net_exitcode] de CloudTask doit être de `0`. Pour savoir comment modifier cela, consultez la section [Actions de dépendance](#dependency-actions).

## <a name="dependency-scenarios"></a>scénarios de dépendance

Vous pouvez utiliser trois scénarios de dépendance de tâches de base dans Azure Batch : un-à-un, un-à-plusieurs et dépendance de plage d’ID de tâche. Ces scénarios peuvent être combinés pour créer un quatrième scénario : plusieurs-à-plusieurs.

| Scénario&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Exemple | Illustration |
|:---:| --- | --- |
|  [Un-à-un](#one-to-one) |*taskB* dépend de *taskA* <p/> *taskB* n’est pas planifié pour s’exécuter tant que l’exécution de *taskA* n’est pas terminée |![Schéma : dépendance de tâches un-à-un][1] |
|  [Un-à-plusieurs](#one-to-many) |*taskC* dépend de *taskA* et de *taskB* <p/> *taskC* n’est pas planifié pour s’exécuter tant que l’exécution de *taskA* et *taskB* n’est pas terminée |![Schéma : dépendance de tâches un-à-plusieurs][2] |
|  [Plage d’ID de tâche](#task-id-range) |*taskD* dépend d’une plage de tâches <p/> *taskD* n’est pas planifié pour s’exécuter tant que l’exécution des tâches associées aux ID *1* à *10* n’est pas terminée |![Schéma : Dépendance de plage d'ID de tâche][3] |

> [!TIP]
> Vous pouvez créer des relations **plusieurs-à-plusieurs** où, par exemple, les tâches C, D, E et F dépendent toutes des tâches A et B. Cela est utile, par exemple, dans les scénarios de prétraitement parallélisés où vos tâches en aval dépendent de la sortie de plusieurs tâches en amont.
> 
> Dans les exemples de cette section, une tâche dépendante s’exécute uniquement après l’achèvement des tâches parentes. Ce comportement est le comportement par défaut d’une tâche dépendante. Vous pouvez exécuter une tâche dépendante après l’échec d’une tâche parente en indiquant l’action de dépendance destinée à se substituer au comportement par défaut. Pour plus d’informations, consultez la section [Actions de dépendance](#dependency-actions).

### <a name="one-to-one"></a>Un à un

Dans une relation un-à-un, une tâche dépend de la bonne exécution d’une tâche parente. Pour créer la dépendance, fournissez un ID de tâche unique à la méthode statique [TaskDependencies][net_taskdependencies].[OnId][net_onid] lorsque vous renseignez la propriété [DependsOn][net_dependson] de [CloudTask][net_cloudtask].

```csharp
// Task 'taskA' doesn't depend on any other tasks
new CloudTask("taskA", "cmd.exe /c echo taskA"),

// Task 'taskB' depends on completion of task 'taskA'
new CloudTask("taskB", "cmd.exe /c echo taskB")
{
    DependsOn = TaskDependencies.OnId("taskA")
},
```

### <a name="one-to-many"></a>Un-à-plusieurs

Dans une relation un-à-plusieurs, une tâche dépend de la bonne exécution de plusieurs tâches parentes. Pour créer la dépendance, fournissez une collection d’ID de tâche à la méthode statique [TaskDependencies][net_taskdependencies].[OnId][net_onids] lorsque vous renseignez la propriété [DependsOn][net_dependson] de [CloudTask][net_cloudtask].

```csharp
// 'Rain' and 'Sun' don't depend on any other tasks
new CloudTask("Rain", "cmd.exe /c echo Rain"),
new CloudTask("Sun", "cmd.exe /c echo Sun"),

// Task 'Flowers' depends on completion of both 'Rain' and 'Sun'
// before it is run.
new CloudTask("Flowers", "cmd.exe /c echo Flowers")
{
    DependsOn = TaskDependencies.OnIds("Rain", "Sun")
},
``` 

### <a name="task-id-range"></a>Plage d’ID de tâche

En cas de dépendance à une plage de tâches parentes, la tâche dépend de la bonne exécution des tâches dont le numéro figure dans la plage.
Pour créer la dépendance, fournissez le premier et le dernier ID de tâche dans la plage à la méthode statique [TaskDependencies][net_taskdependencies].[OnIdRange][net_onidrange] lorsque vous renseignez la propriété [DependsOn][net_dependson] de [CloudTask][net_cloudtask].

> [!IMPORTANT]
> Lorsque des plages de numéros de tâche sont utilisées pour les dépendances, seules les tâches dont le numéro représente des valeurs entières sont sélectionnées par la plage. Ainsi, la plage `1..10` sélectionne les tâches `3` et `7`, mais pas `5flamingoes`. 
>
> Les zéros de gauche n'étant pas significatifs pour l'évaluation des dépendances aux plages, les tâches ayant pour identificateurs de chaîne `4`, `04` et `004` se trouvent toutes *dans* la plage et sont traitées comme correspondant à la tâche `4`, de sorte que la première à se terminer satisfait la dépendance.
>
> Chaque tâche de la plage doit satisfaire la dépendance soit en se terminant avec succès, soit en échouant avec une erreur associée à une action de dépendance définie sur **Satisfy**. Pour plus d’informations, consultez la section [Actions de dépendance](#dependency-actions).

```csharp
// Tasks 1, 2, and 3 don't depend on any other tasks. Because
// we will be using them for a task range dependency, we must
// specify string representations of integers as their ids.
new CloudTask("1", "cmd.exe /c echo 1"),
new CloudTask("2", "cmd.exe /c echo 2"),
new CloudTask("3", "cmd.exe /c echo 3"),

// Task 4 depends on a range of tasks, 1 through 3
new CloudTask("4", "cmd.exe /c echo 4")
{
    // To use a range of tasks, their ids must be integer values.
    // Note that we pass integers as parameters to TaskIdRange,
    // but their ids (above) are string representations of the ids.
    DependsOn = TaskDependencies.OnIdRange(1, 3)
},
```

## <a name="dependency-actions"></a>Actions de dépendance

Par défaut, une tâche dépendante ou un ensemble de tâches s’exécute uniquement après la fin d’une tâche parente. Dans certains scénarios, vous pouvez exécuter des tâches dépendantes même si la tâche parente échoue. Vous pouvez remplacer le comportement par défaut en spécifiant une action de dépendance. Une action de dépendance spécifie si une tâche dépendante peut être exécutée en fonction de la réussite ou de l’échec de la tâche parente. 

Par exemple, supposons qu’une tâche dépendante attend des données de l’achèvement de la tâche amont. Si la tâche en amont échoue, la tâche dépendante peut toujours être en mesure de s’exécuter en utilisant des données plus anciennes. Dans ce cas, une action de dépendance peut spécifier que la tâche dépendante peut être exécutée malgré l’échec de la tâche parente.

Une action de dépendance est basée sur une condition de sortie pour la tâche parente. Vous pouvez indiquer une action de dépendance pour toutes les conditions de sortie suivantes ; pour un environnement .NET, consultez la classe [ExitConditions][net_exitconditions] pour plus de détails :

- Lorsqu’une erreur de prétraitement se produit.
- Lorsqu’une erreur de chargement de fichier se produit. Si la tâche se termine avec un code de sortie qui a été spécifié via **exitCodes** ou **exitCodeRanges**, puis rencontre une erreur de chargement de fichier, l’action spécifiée par le code de sortie est prioritaire.
- Lorsque la tâche se termine avec un code de sortie défini par la propriété **ExitCodes**.
- Lorsque la tâche se termine avec un code de sortie dans une plage définie par la propriété **ExitCodeRanges**.
- Le cas par défaut : si la tâche se termine avec un code de sortie non défini par **ExitCodes** ou **ExitCodeRanges**, ou si la tâche se termine avec une erreur de prétraitement et si la propriété **PreProcessingError** n’est pas définie, ou si la tâche échoue avec une erreur de chargement de fichier et si la propriété **FileUploadError** n’est pas définie. 

Pour spécifier une action de dépendance dans .NET, définissez la propriété [ExitOptions][net_exitoptions].[DependencyAction][net_dependencyaction] pour la condition de sortie. La propriété **DependencyAction** accepte l’une des deux valeurs suivantes :

- Définir la propriété **DependencyAction** sur **Satisfy** indique que les tâches dépendantes sont autorisées à s’exécuter si la tâche parente se termine avec une erreur spécifiée.
- Définir la propriété **DependencyAction** sur **Block** indique que les tâches dépendantes ne sont pas autorisées à s’exécuter.

Le paramètre par défaut de la propriété **DependencyAction** est **Satisfy** pour le code de sortie 0 et **Block** pour toutes les autres conditions de sortie.

L’extrait de code suivant définit la propriété **DependencyAction** d’une tâche parente. Si la tâche parente se termine avec une erreur de prétraitement ou avec les codes d’erreur spécifiés, la tâche dépendante est bloquée. Si la tâche parente se termine avec une autre erreur non nulle, la tâche dépendante peut être exécutée.

```csharp
// Task A is the parent task.
new CloudTask("A", "cmd.exe /c echo A")
{
    // Specify exit conditions for task A and their dependency actions.
    ExitConditions = new ExitConditions
    {
        // If task A exits with a pre-processing error, block any downstream tasks (in this example, task B).
        PreProcessingError = new ExitOptions
        {
            DependencyAction = DependencyAction.Block
        },
        // If task A exits with the specified error codes, block any downstream tasks (in this example, task B).
        ExitCodes = new List<ExitCodeMapping>
        {
            new ExitCodeMapping(10, new ExitOptions() { DependencyAction = DependencyAction.Block }),
            new ExitCodeMapping(20, new ExitOptions() { DependencyAction = DependencyAction.Block })
        },
        // If task A succeeds or fails with any other error, any downstream tasks become eligible to run 
        // (in this example, task B).
        Default = new ExitOptions
        {
            DependencyAction = DependencyAction.Satisfy
        }
    }
},
// Task B depends on task A. Whether it becomes eligible to run depends on how task A exits.
new CloudTask("B", "cmd.exe /c echo B")
{
    DependsOn = TaskDependencies.OnId("A")
},
```

## <a name="code-sample"></a>Exemple de code

L’exemple de projet [TaskDependencies][github_taskdependencies] est l’un des [exemples de code Azure Batch][github_samples] disponibles sur GitHub. La solution Visual Studio montre :

- Comment activer la dépendance d’une tâche
- Comment créer des tâches qui dépendent d’autres tâches
- Comment exécuter ces tâches sur un pool de nœuds de calcul

## <a name="next-steps"></a>Étapes suivantes

- La fonctionnalité [packages d’application](batch-application-packages.md) de Batch est un moyen facile de déployer et contrôler les versions des applications exécutées par vos tâches sur des nœuds de calcul.
- Pour découvrir les méthodes de préparation des nœuds à l’exécution de tâches, consultez l’article [Installing applications and staging data on Batch compute nodes][forum_post] (Installation d’applications et de données intermédiaires sur les nœuds de calcul Batch) sur le forum Azure Batch. Rédigée par un membre de l’équipe Azure Batch, cette publication est une excellente introduction aux différentes façons de copier des applications, des données d’entrée de tâche et d’autres fichiers sur vos nœuds de calcul.

[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_taskdependencies]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: /dotnet/api/microsoft.azure.batch.batchclient
[net_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_cloudtask]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_dependson]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_exitcode]: /dotnet/api/microsoft.azure.batch.taskexecutioninformation
[net_exitconditions]: /dotnet/api/microsoft.azure.batch.exitconditions
[net_exitoptions]: /dotnet/api/microsoft.azure.batch.exitoptions
[net_dependencyaction]: /dotnet/api/microsoft.azure.batch.exitoptions
[net_msdn]: /dotnet/api/microsoft.azure.batch
[net_onid]: /dotnet/api/microsoft.azure.batch.taskdependencies
[net_onids]: /dotnet/api/microsoft.azure.batch.taskdependencies
[net_onidrange]: /dotnet/api/microsoft.azure.batch.taskdependencies
[net_taskexecutioninformation]: /dotnet/api/microsoft.azure.batch.taskexecutioninformation
[net_taskstate]: /dotnet/api/microsoft.azure.batch.common.taskstate
[net_usestaskdependencies]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_taskdependencies]: /dotnet/api/microsoft.azure.batch.taskdependencies

[1]: ./media/batch-task-dependency/01_one_to_one.png "Schéma : dépendance un-à-un"
[2]: ./media/batch-task-dependency/02_one_to_many.png "Schéma : dépendance un-à-plusieurs"
[3]: ./media/batch-task-dependency/03_task_id_range.png "Schéma : dépendance de plage d’ID de tâche"
