---
title: Exécuter des tâches simultanément pour optimiser l’utilisation des nœuds de calcul Batch
description: Améliorer l’efficacité et réduire les coûts en utilisant moins de nœuds de calcul et en exécutant des tâches en parallèle sur chaque nœud dans un pool Azure Batch
ms.topic: how-to
ms.date: 10/08/2020
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: 8bc9f03f05d52df6e400be5c57033ab2a38fa8eb
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92102963"
---
# <a name="run-tasks-concurrently-to-maximize-usage-of-batch-compute-nodes"></a>Exécuter des tâches simultanément pour optimiser l’utilisation des nœuds de calcul Batch

En exécutant simultanément plusieurs tâches sur chaque nœud dans votre pool, vous pouvez optimiser l’utilisation des ressources sur un plus petit nombre de nœuds de calcul.

Bien que certains scénarios fonctionnent mieux avec toutes les ressources d’un nœud dédiées à une seule tâche, certaines charges de travail peuvent avoir des temps de travail plus courts et réduire les coûts lorsque plusieurs tâches partagent ces ressources :

- **Réduisez les transferts de données** lorsque les tâches sont en mesure de partager des données. Vous pouvez considérablement réduire les frais de transfert de données en copiant les données partagées vers un plus petit nombre de nœuds et en exécutant les tâches en parallèle sur chaque nœud. Cela s'applique surtout si les données à copier sur chaque nœud doivent être transférées entre des régions géographiques.
- **Optimisation de l’utilisation de la mémoire** pour les tâches nécessitent une grande quantité de mémoire, mais seulement pendant de courtes périodes et à des moments variables au cours de l’exécution. Vous pouvez employer des nœuds de calcul moins nombreux mais de plus grande taille, avec plus de mémoire pour gérer efficacement ces pics. De cette façon, ces nœuds ont plusieurs tâches exécutées en parallèle sur chaque nœud, mais chaque tâche bénéficie de la mémoire abondante des nœuds à des moments différents.
- **Atténuez les limites au nombre de nœuds** lorsque la communication entre les nœuds est requise au sein d’un pool. Actuellement, les pools configurés pour la communication entre les nœuds sont limités à 50 nœuds de calcul. Si chaque nœud dans un pool de ce type est capable d’exécuter des tâches en parallèle, un plus grand nombre de tâches peuvent être exécutées simultanément.
- **Repliquez un cluster de calcul local** , comme lorsque vous déplacez un environnement de calcul vers Azure pour la première fois. Si cette configuration exécute actuellement plusieurs tâches par nœud de calcul, vous pouvez augmenter le nombre maximal de tâches de nœud pour refléter plus précisément cette configuration.

## <a name="example-scenario"></a>Exemple de scénario

Par exemple, imaginez une application de tâche avec des exigences de processeur et de mémoire signifiant que des nœuds [Standard\_D1](../cloud-services/cloud-services-sizes-specs.md) sont suffisants. Cependant, pour terminer le travail dans le délai imparti, 1 000 nœuds de ce type sont nécessaires.

Au lieu d’utiliser les nœuds Standard\_D1 avec 1 cœur de processeur, vous pouvez utiliser des nœuds [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) avec 16 cœurs chacun, et activer l’exécution de tâches parallèles. Vous pouvez donc utiliser *16 fois moins de nœuds* : à la place des 1 000 nœuds, seuls 63 sont requis. Si des fichiers d’application volumineux ou des données de référence sont requis pour chaque nœud, l’efficacité et la durée du travail sont encore améliorées, car les données ne sont copiées que sur 63 nœuds.

## <a name="enable-parallel-task-execution"></a>Activer l’exécution des tâches parallèles

Vous configurez les nœuds de calcul pour l’exécution des tâches parallèles au niveau du pool. Avec la bibliothèque Batch .NET, définissez la propriété [CloudPool.TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool) lorsque vous créez un pool. Si vous utilisez l’API REST Batch, définissez l’élément [taskSlotsPerNode](/rest/api/batchservice/pool/add) dans le corps de la demande lors de la création du pool.

> [!NOTE]
> Vous ne pouvez définir l’élément `taskSlotsPerNode` et la propriété [TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool) qu’au moment de la création du pool. Ils ne peuvent pas être modifiés après qu’un pool a déjà été créé.

Azure Batch vous permet de définir un nombre d’emplacements de tâches par nœud jusqu’à quatre fois supérieur au nombre de nœuds. Par exemple, si le pool est configuré avec des nœuds de grande taille (quatre cœurs), alors la valeur `taskSlotsPerNode` peut être définie sur 16. Toutefois, quel que soit le nombre de cœurs du nœud, vous ne pouvez pas dépasser les 256 emplacements de tâches par nœud. Pour plus d’informations sur le nombre de cœurs pour chacune des tailles de nœud, consultez [Tailles de services Cloud](../cloud-services/cloud-services-sizes-specs.md). Pour plus d’informations sur les limites du service, consultez [Quotas et les limites pour le service Azure Batch](batch-quota-limit.md).

> [!TIP]
> Veillez à prendre en compte la valeur `taskSlotsPerNode` lors de la construction d’une [formule de mise à l’échelle](/rest/api/batchservice/pool/enableautoscale) pour votre pool. Par exemple, une formule qui évalue `$RunningTasks` pourrait être considérablement affectée par une augmentation des tâches par nœud. Consultez [Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch](batch-automatic-scaling.md) pour plus d’informations.

## <a name="specify-task-distribution"></a>Spécifier la distribution des tâches

Lorsque vous activez les tâches simultanées, il est important de spécifier comment vous souhaitez que les tâches soient réparties entre les nœuds du pool.

La propriété [CloudPool.TaskSchedulingPolicy](/dotnet/api/microsoft.azure.batch.cloudpool) vous permet de spécifier que les tâches doivent être affectées uniformément entre tous les nœuds du pool (« propagation »). Vous pouvez également spécifier qu'autant de tâches que possible doivent être attribuées à chaque nœud avant que les tâches ne soient attribuées à un autre nœud du pool (« compression »).

Pour illustrer cette fonctionnalité, examinons le pool de nœuds [Standard\_D14](../cloud-services/cloud-services-sizes-specs.md) (dans l’exemple ci-dessus) configuré avec une propriété [CloudPool.TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool) de valeur 16. Si la propriété [CloudPool.TaskSchedulingPolicy](/dotnet/api/microsoft.azure.batch.cloudpool) est configurée avec une propriété [ComputeNodeFillType](/dotnet/api/microsoft.azure.batch.common.computenodefilltype) de type *Pack* , l’utilisation des 16 cœurs de chaque nœud est optimisée et un [pool de mise à l’échelle automatique](batch-automatic-scaling.md) est autorisé pour supprimer les nœuds inutilisés (nœuds sans aucune tâche affectée) du pool. Ceci limite l'utilisation des ressources et permet d'économiser de l'argent.

## <a name="define-variable-slots-per-task"></a>Définir des emplacements variables par tâche

Une tâche peut être définie avec la propriété [CloudTask.RequiredSlots](/dotnet/api/microsoft.azure.batch.cloudtask.requiredslots) pour spécifier le nombre d’emplacements nécessaires pour une exécution sur un nœud de calcul. La valeur par défaut est de 1. Vous pouvez définir des emplacements de tâche variables si vos tâches ont des pondérations différentes concernant l’utilisation des ressources sur le nœud de calcul. Cela permet à chaque nœud de calcul d’avoir un nombre raisonnable de tâches simultanées en cours d’exécution sans trop de ressources système, comme le processeur ou la mémoire.

Par exemple, pour un pool avec la propriété `taskSlotsPerNode = 8`, vous pouvez soumettre des tâches gourmandes en ressources processeur nécessitant plusieurs cœurs avec `requiredSlots = 8`, et d’autres tâches avec `requiredSlots = 1`. Lorsque cette charge de travail mixte est planifiée, les tâches gourmandes en ressources processeur s’exécutent exclusivement sur le nœud de calcul, tandis que d’autres tâches peuvent s’exécuter simultanément (jusqu’à huit tâches à la fois) sur d’autres nœuds. Cela vous permet d’équilibrer votre charge de travail entre les nœuds de calcul et d’améliorer l’efficacité de l’utilisation des ressources.

> [!TIP]
> Lors de l’utilisation d’emplacements de tâches variables, il est possible que des tâches volumineuses nécessitant davantage d’emplacements ne puissent temporairement pas être planifiées en raison d’un nombre insuffisant d’emplacements disponibles sur un nœud de calcul, même si des emplacements sont encore inactifs sur certains nœuds. Vous pouvez augmenter la priorité de travail pour ces tâches afin d’augmenter leurs chances de rivaliser pour les emplacements disponibles sur les nœuds.
>
> Le service Batch émet également [TaskScheduleFailEvent](batch-task-schedule-fail-event.md) lorsqu’il ne parvient pas à planifier l’exécution d’une tâche, tout en renouvelant la tentative de planification jusqu’à ce que les emplacements exigés soient disponibles. Vous pouvez être à l’écoute de cet événement pour détecter les problèmes potentiels de planification de tâches, et réaliser une atténuation en conséquence.

> [!NOTE]
> Ne spécifiez pas une valeur `requiredSlots` de la tâche supérieure à la valeur `taskSlotsPerNode` du pool. En conséquence, la tâche ne pourra jamais s’exécuter. Actuellement, le service Batch n’effectue pas cette validation de conflit lorsque vous soumettez des tâches, car le travail peut ne pas avoir de pool lié au moment de l’envoi, ou être remplacé par un pool différent par désactivation/réactivation.

## <a name="batch-net-example"></a>Exemple .NET Batch

Les extraits de code d’API [.NET Batch](/dotnet/api/microsoft.azure.batch) suivants montrent comment créer un pool avec plusieurs emplacements de tâches par nœud, et soumettre une tâche avec les emplacements exigés.

### <a name="create-a-pool-with-multiple-task-slots-per-node"></a>Créer un pool avec plusieurs emplacements de tâche par nœud

Cet extrait de code illustre une demande de création d’un pool contenant quatre nœuds avec un maximum de quatre emplacements de tâches par nœud. Une stratégie de planification de tâche est également spécifiée ; elle remplira chaque nœud de tâches avant d'attribuer des tâches à un autre nœud du pool.

Pour plus d’informations sur l’ajout de pools à l’aide de l’API Batch .NET, consultez [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations).

```csharp
CloudPool pool =
    batchClient.PoolOperations.CreatePool(
        poolId: "mypool",
        targetDedicatedComputeNodes: 4
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

pool.TaskSlotsPerNode = 4;
pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
pool.Commit();
```

### <a name="create-a-task-with-required-slots"></a>Créer une tâche avec les emplacements exigés

Cet extrait de code crée une tâche avec une valeur `requiredSlots` différente de sa valeur par défaut. Cette tâche s’exécute uniquement lorsque le nombre d’emplacements disponibles sur un nœud de calcul est suffisant.

```csharp
CloudTask task = new CloudTask(taskId, taskCommandLine)
{
    RequiredSlots = 2
};
```

### <a name="list-compute-nodes-with-counts-for-running-tasks-and-slots"></a>Lister les nœuds de calcul avec les nombres de tâches en cours d’exécution et d’emplacements

Cet extrait de code liste tous les nœuds de calcul dans le pool, puis affiche les nombres de tâches en cours d’exécution et d’emplacements de tâches par nœud.

```csharp
ODATADetailLevel nodeDetail = new ODATADetailLevel(selectClause: "id,runningTasksCount,runningTaskSlotsCount");
IPagedEnumerable<ComputeNode> nodes = batchClient.PoolOperations.ListComputeNodes(poolId, nodeDetail);

await nodes.ForEachAsync(node =>
{
    Console.WriteLine(node.Id + " :");
    Console.WriteLine($"RunningTasks = {node.RunningTasksCount}, RunningTaskSlots = {node.RunningTaskSlotsCount}");

}).ConfigureAwait(continueOnCapturedContext: false);
```

### <a name="list-task-counts-for-the-job"></a>Lister les nombres de tâches pour le travail

Cet extrait de code obtient le nombre de tâches pour le travail, qui comprend le nombre de tâches et le nombre d’emplacements de tâches par état de tâche.

```csharp
TaskCountsResult result = await batchClient.JobOperations.GetJobTaskCountsAsync(jobId);

Console.WriteLine("\t\tActive\tRunning\tCompleted");
Console.WriteLine($"TaskCounts:\t{result.TaskCounts.Active}\t{result.TaskCounts.Running}\t{result.TaskCounts.Completed}");
Console.WriteLine($"TaskSlotCounts:\t{result.TaskSlotCounts.Active}\t{result.TaskSlotCounts.Running}\t{result.TaskSlotCounts.Completed}");
```

## <a name="batch-rest-example"></a>Exemple REST Batch

Les extraits de code d’API [REST Batch](/rest/api/batchservice/) suivants montrent comment créer un pool avec plusieurs emplacements de tâches par nœud, et soumettre une tâche avec les emplacements exigés.

### <a name="create-a-pool-with-multiple-task-slots-per-node"></a>Créer un pool avec plusieurs emplacements de tâche par nœud

Cet extrait de code illustre une demande de création d’un pool contenant deux grands nœuds avec un maximum de quatre tâches par nœud.

Pour plus d’informations sur l’ajout de pools à l’aide de l’API REST, consultez la page [Ajout d’un pool à un compte](/rest/api/batchservice/pool/add).

```json
{
  "odata.metadata":"https://myaccount.myregion.batch.azure.com/$metadata#pools/@Element",
  "id":"mypool",
  "vmSize":"large",
  "cloudServiceConfiguration": {
    "osFamily":"4",
    "targetOSVersion":"*",
  },
  "targetDedicatedComputeNodes":2,
  "taskSlotsPerNode":4,
  "enableInterNodeCommunication":true,
}
```

### <a name="create-a-task-with-required-slots"></a>Créer une tâche avec les emplacements exigés

Cet extrait de code montre une demande d’ajout d’une tâche avec une valeur `requiredSlots` différente de sa valeur par défaut. Cette tâche s’exécute uniquement lorsque le nombre d’emplacements disponibles sur le nœud de calcul est suffisant.

```json
{
  "id": "taskId",
  "commandLine": "bash -c 'echo hello'",
  "userIdentity": {
    "autoUser": {
      "scope": "task",
      "elevationLevel": "nonadmin"
    }
  },
  "requiredSLots": 2
}
```

## <a name="code-sample-on-github"></a>Exemple de code sur GitHub

Le projet [ParallelTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks) sur GitHub illustre l’utilisation de la propriété [CloudPool.TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool).

Cette application de console en C# utilise la bibliothèque [Batch .NET](/dotnet/api/microsoft.azure.batch) pour créer un pool avec un ou plusieurs nœuds de calcul. Elle exécute un nombre configurable de tâches sur ces nœuds pour simuler la charge variable. La sortie de l’application spécifie quels nœuds ont exécuté chaque tâche. L'application fournit également un résumé des paramètres du travail et sa durée.

À titre d’exemple, la partie Résumé de la sortie de deux exécutions différentes de l’exemple ParallelTasks d’application apparaît ci-dessous. Les durées de travail indiquées ici n’incluent pas l’heure de création du pool, puisque chaque tâche a été envoyées à un pool créé précédemment dont les nœuds de calcul étaient dans l’état *Inactif* au moment de l’envoi.

La première exécution de l'exemple d'application montre qu'avec un nœud unique dans le pool et le paramètre par défaut d'une tâche par nœud, la durée du travail dépasse 30 minutes.

```
Nodes: 1
Node size: large
Task slots per node: 1
Max slots per task: 1
Tasks: 32
Duration: 00:30:01.4638023
```

La deuxième exécution de l'exemple montre une diminution significative de la durée du travail. Cela est dû au fait que le pool a été configuré avec quatre tâches par nœud, ce qui permet l’exécution de tâches parallèles pour terminer le travail en un quart du temps, environ.

```
Nodes: 1
Node size: large
Task slots per node: 4
Max slots per task: 1
Tasks: 32
Duration: 00:08:48.2423500
```

## <a name="next-steps"></a>Étapes suivantes

- Essayez la carte thermique [Batch Explorer](https://azure.github.io/BatchExplorer/). Batch Explorer est un outil client autonome gratuit, doté de nombreuses fonctionnalités aidant à créer, déboguer et surveiller les applications Azure Batch. Lorsque vous exécutez l’exemple d’application [ParallelTasks](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks), la fonctionnalité carte thermique Batch Explorer permet de visualiser l’exécution de tâches parallèles sur chaque nœud.
- Explorez les [exemples Azure Batch sur GitHub](https://github.com/Azure/azure-batch-samples).
- En savoir plus sur les [dépendances des tâches Batch](batch-task-dependencies.md).
