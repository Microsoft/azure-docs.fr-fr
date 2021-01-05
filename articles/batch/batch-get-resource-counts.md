---
title: Compter les états des tâches et des nœuds
description: Comptez l’état des tâches et des nœuds de calcul Azure Batch pour faciliter la gestion et la surveillance des solutions Batch.
ms.date: 06/18/2020
ms.topic: how-to
ms.custom: seodec18
ms.openlocfilehash: 90f741b9ec5e17da4fd0cc95ef921e116b0c27dc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85960586"
---
# <a name="monitor-batch-solutions-by-counting-tasks-and-nodes-by-state"></a>Surveiller les solutions Batch en comptant les tâches et les nœuds par état

Pour surveiller et gérer des solutions Azure Batch à grande échelle, vous devrez peut-être déterminer le nombre de ressources dans les différents états. Azure Batch fournit des opérations efficaces pour obtenir ces nombres pour les tâches et les nœuds de calcul Batch. Vous pouvez utiliser ces opérations à la place des requêtes de liste potentiellement longues pour retourner des informations détaillées sur les grandes collections de tâches ou de nœuds.

- [Obtenir le nombre de tâches](/rest/api/batchservice/job/gettaskcounts) obtient un comptage agrégé des tâches actives, en cours d’exécution et terminées dans un travail, ainsi que des tâches ayant réussi ou échoué. 

  En comptant les tâches dans chaque état, vous pouvez plus facilement afficher la progression du travail à un utilisateur, ou détecter des retards ou échecs inattendus susceptibles d’affecter le travail. Obtenir le nombre de tâches est disponible à compter de l’API version 2017-06-01.5.1 et des kits SDK et outils associés du service Batch.

- [Répertorier le nombre de nœuds des pools](/rest/api/batchservice/account/listpoolnodecounts) obtient le nombre de nœuds de calcul dédiés et basse priorité dans chaque pool qui se trouvent dans différents états : création, inactif, hors connexion, anticipé, en cours de redémarrage, réinitialisation, démarrage, etc.

  En comptant les nœuds dans chaque état, vous pouvez déterminer quand vous avez des ressources de calcul suffisantes pour exécuter vos travaux, et identifier les problèmes potentiels avec vos pools. Répertorier les nombres de nœuds des pools est disponible à compter de l’API version 2018-03-01.6.1 et des kits SDK et outils associés du service Batch.

Notez que dans certains cas, les nombres retournés par ces opérations peuvent ne pas être à jour. Si vous devez vous assurer qu’un nombre est exact, utilisez une requête de liste pour compter ces ressources. Les requêtes de liste vous permettent également d’obtenir des informations sur d’autres ressources Batch, comme les applications. Pour plus d’informations sur l’application de filtres aux requêtes de liste, consultez [Créer des requêtes pour répertorier les ressources Batch efficacement](batch-efficient-list-queries.md).

## <a name="task-state-counts"></a>Nombre d’états des tâches

L’opération Obtenir le nombre de tâches compte les tâches selon les états suivants :

- **Actif** : tâche qui est en file d’attente et en mesure de s’exécuter, mais qui n’est actuellement pas affectée à un nœud de calcul. Une tâche est également `active` si elle est [dépendante d’une tâche parente](batch-task-dependencies.md) qui n’est pas encore terminée. 
- **En cours d’exécution** : tâche qui a été affectée à un nœud de calcul, mais qui n’est pas encore terminée. Une tâche est comptée comme `running` quand son état est `preparing` ou `running`, comme indiqué par l’opération [Obtenir des informations sur une tâche](/rest/api/batchservice/task/get).
- **Terminé** - tâche qui n’est plus éligible pour s’exécuter, car elle s’est terminée correctement, ou bien elle a échoué et a épuisé son nombre maximal de nouvelles tentatives. 
- **Réussite** - tâche dont le résultat de l’exécution est `success`. Le service Batch détermine si une tâche a réussi ou échoué en vérifiant la propriété `TaskExecutionResult` de la propriété [executionInfo](/rest/api/batchservice/task/get).
- **Échec** - une tâche dont le résultat de l’exécution est `failure`.

L’exemple de code .NET suivant montre comment récupérer le nombre de tâches par état :

```csharp
var taskCounts = await batchClient.JobOperations.GetJobTaskCountsAsync("job-1");

Console.WriteLine("Task count in active state: {0}", taskCounts.Active);
Console.WriteLine("Task count in preparing or running state: {0}", taskCounts.Running);
Console.WriteLine("Task count in completed state: {0}", taskCounts.Completed);
Console.WriteLine("Succeeded task count: {0}", taskCounts.Succeeded);
Console.WriteLine("Failed task count: {0}", taskCounts.Failed);
```

Pour obtenir le nombre de tâches pour un travail, vous pouvez utiliser un modèle semblable pour REST et d’autres langages pris en charge. 

> [!NOTE]
> Les versions d’API du service Batch avant 2018-08-01.7.0 retournent aussi une propriété `validationStatus` dans la réponse de l’opération Obtenir le nombre de tâches. Cette propriété indique si Batch a vérifié la cohérence entre le nombre d’états et les états signalés dans l’API Liste de tâches. La valeur `validated` indique uniquement que Batch a vérifié la cohérence du travail au moins une fois. La valeur de la propriété `validationStatus` n’indique pas si le nombre retourné par l’opération Get Task Counts est actuellement à jour.

## <a name="node-state-counts"></a>Nombre d’états des nœuds

L’opération Répertorier le nombre de nœuds des pools compte les nœuds selon les états suivants dans chaque pool. Des comptages agrégés distincts sont fournis pour les nœuds dédiés et pour les nœuds basse priorité dans chaque pool.

- **Création** : machine virtuelle allouée par Azure qui n’a pas encore commencé à rejoindre un pool.
- **Inactif** : nœud de calcul disponible qui n’exécute actuellement pas une tâche.
- **LeavingPool** : nœud qui quitte le pool parce que l’utilisateur l’a explicitement supprimé ou parce que le pool est redimensionné ou mis automatiquement à une plus basse échelle.
- **Hors connexion** : nœud que le service Batch ne peut pas utiliser pour planifier de nouvelles tâches.
- **Anticipé** : nœud basse priorité qui a été supprimé du pool, car Azure a récupéré la machine virtuelle. Un nœud `preempted` peut être réinitialisé quand la fonctionnalité de machine virtuelle basse priorité de remplacement est disponible.
- **En cours de redémarrage** : nœud qui redémarre.
- **Réimageage** : nœud sur lequel le système d’exploitation est en cours de réinstallation.
- **En cours d’exécution** : nœud qui exécute une ou plusieurs tâches (autre que la tâche de démarrage).
- **En cours de démarrage** : nœud sur lequel le service Batch démarre. 
- **StartTaskFailed** : nœud sur lequel la [tâche de démarrage](/rest/api/batchservice/pool/add#starttask) a échoué et a épuisé toutes les nouvelles tentatives, et sur lequel `waitForSuccess` est défini sur la tâche de démarrage. Le nœud n’est pas utilisable pour exécuter des tâches.
- **Inconnu** : nœud qui a perdu le contact avec le service Batch et dont l’état n’est pas connu.
- **Inutilisable** : nœud qui ne peut pas être utilisé pour l’exécution de tâches en raison d’erreurs.
- **WaitingForStartTask** : nœud sur lequel la tâche de démarrage a démarré, mais sur lequel `waitForSuccess` est défini et où la tâche de démarrage n’est pas terminée.

L’extrait de code C# suivant montre comment répertorier le nombre de nœuds pour tous les pools du compte actif :

```csharp
foreach (var nodeCounts in batchClient.PoolOperations.ListPoolNodeCounts())
{
    Console.WriteLine("Pool Id: {0}", nodeCounts.PoolId);

    Console.WriteLine("Total dedicated node count: {0}", nodeCounts.Dedicated.Total);

    // Get dedicated node counts in Idle and Offline states; you can get additional states.
    Console.WriteLine("Dedicated node count in Idle state: {0}", nodeCounts.Dedicated.Idle);
    Console.WriteLine("Dedicated node count in Offline state: {0}", nodeCounts.Dedicated.Offline);

    Console.WriteLine("Total low priority node count: {0}", nodeCounts.LowPriority.Total);

    // Get low-priority node counts in Running and Preempted states; you can get additional states.
    Console.WriteLine("Low-priority node count in Running state: {0}", nodeCounts.LowPriority.Running);
    Console.WriteLine("Low-priority node count in Preempted state: {0}", nodeCounts.LowPriority.Preempted);
}
```

L’extrait de code C# suivant montre comment répertorier le nombre de nœuds pour un pool donné dans le compte actuel.

```csharp
foreach (var nodeCounts in batchClient.PoolOperations.ListPoolNodeCounts(new ODATADetailLevel(filterClause: "poolId eq 'testpool'")))
{
    Console.WriteLine("Pool Id: {0}", nodeCounts.PoolId);

    Console.WriteLine("Total dedicated node count: {0}", nodeCounts.Dedicated.Total);

    // Get dedicated node counts in Idle and Offline states; you can get additional states.
    Console.WriteLine("Dedicated node count in Idle state: {0}", nodeCounts.Dedicated.Idle);
    Console.WriteLine("Dedicated node count in Offline state: {0}", nodeCounts.Dedicated.Offline);

    Console.WriteLine("Total low priority node count: {0}", nodeCounts.LowPriority.Total);

    // Get low-priority node counts in Running and Preempted states; you can get additional states.
    Console.WriteLine("Low-priority node count in Running state: {0}", nodeCounts.LowPriority.Running);
    Console.WriteLine("Low-priority node count in Preempted state: {0}", nodeCounts.LowPriority.Preempted);
}
```

Pour obtenir le nombre de nœuds pour des pools, vous pouvez utiliser un modèle similaire pour REST et pour les autres langages pris en charge.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md), telles que les pools, les nœuds, les travaux et les tâches.
- Pour en savoir plus sur l’application de filtres aux requêtes qui répertorient les ressources Batch, consultez [Créer des requêtes pour répertorier les ressources Batch efficacement](batch-efficient-list-queries.md).
