---
title: ReliableConcurrentQueue dans Azure Service Fabric
description: ReliableConcurrentQueue est une file d’attente à débit élevé qui permet des mises en file d’attente et retraits de file d’attente parallèles.
ms.topic: conceptual
ms.date: 5/1/2017
ms.openlocfilehash: 423ef3d1898176d7c25c596ad186a9c000108aa4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997118"
---
# <a name="introduction-to-reliableconcurrentqueue-in-azure-service-fabric"></a>Présentation de ReliableConcurrentQueue dans Azure Service Fabric
Une file d’attente simultanée fiable est une file d’attente asynchrone, transactionnelle et répliquée, qui permet d’effectuer des opérations de mise en file d’attente et de retrait de file d’attente avec un niveau élevé de simultanéité. Elle est conçue pour offrir un débit élevé et une latence faible en assouplissant la séquence stricte de premier entré, premier sorti fournie par une [file d’attente fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliablequeue-1?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliablequeue_1), et fournit à la place un classement selon le principe de l’effort optimal.

## <a name="apis"></a>API

|File d’attente simultanée                |File d’attente simultanée fiable                                         |
|--------------------------------|------------------------------------------------------------------|
| void Enqueue(T item)           | Task EnqueueAsync(ITransaction tx, T item)                       |
| bool TryDequeue(out T result)  | Task< ConditionalValue < T > > TryDequeueAsync(ITransaction tx)  |
| int Count()                    | long Count()                                                     |

## <a name="comparison-with-reliable-queue"></a>Comparaison avec un [File d’attente fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliablequeue-1?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliablequeue_1)

Une file d’attente simultanée fiable est proposée en guise d’alternative à une [file d’attente fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliablequeue-1?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliablequeue_1). Elle doit être utilisée dans les cas où la séquence stricte de premier entré, premier sorti n’est pas requise, celle-ci nécessitant un compromis sur le plan de la simultanéité.  Une [File d’attente fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliablequeue-1?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliablequeue_1) utilise des verrous pour appliquer la séquence de premier entré, premier sorti, avec au plus une transaction autorisée pour la mise en file d’attente et une transaction autorisée pour le retrait de file d’attente à la fois. En comparaison, une file d’attente simultanée fiable assouplit la contrainte de séquence et permet qu’un nombre quelconque de transactions simultanées entrelacent leurs opérations de mise en file d’attente et de retrait de file d’attente. Un classement selon le principe de l’effort optimal est fourni, mais l’ordre relatif de deux valeurs dans une file d’attente simultanée fiable ne peut jamais être garanti.

Une file d’attente simultanée fiable offre un débit supérieur et une latence moindre qu’une [file d’attente fiable](/dotnet/api/microsoft.servicefabric.data.collections.ireliablequeue-1?view=azure-dotnet#microsoft_servicefabric_data_collections_ireliablequeue_1) chaque fois que plusieurs transactions simultanées effectuent des mises en file d’attente ou des retraits de file d’attente.

Un exemple de cas d’utilisation de ReliableConcurrentQueue est le scénario de la [File d’attente des messages](https://en.wikipedia.org/wiki/Message_queue). Dans ce scénario, un ou plusieurs producteurs de messages créent et ajoutent des éléments à la file d’attente, et un ou plusieurs consommateurs de messages extraient des messages de la file d’attente et les traitent. Plusieurs producteurs et consommateurs peuvent travailler de façon indépendante, en utilisant des transactions simultanées pour traiter la file d’attente.

## <a name="usage-guidelines"></a>Instructions d’utilisation
* La file d’attente attend que la période de rétention des éléments qu’elle contient soit courte. Autrement dit, ces éléments ne sont pas supposés rester longtemps dans la file d’attente.
* La file d’attente ne garantit pas le respect strict de la séquence de premier entré, premier sorti.
* La file d’attente ne lit pas ses propres écritures. Si un élément est mis en file d’attente dans le cadre d’une transaction, il n’est pas visible pour un retrait de file d’attente dans le cadre de la même transaction.
* Les retraits de file d’attente ne sont pas isolés les uns des autres. Si un élément *A* est retiré d’une file d’attente dans le cadre d’une transaction *txnA*, même si la transaction *txnA* n’est pas validée, l’élément *A* n’est pas visible pour une transaction simultanée *txnB*.  Si la transaction *txnA* est abandonnée, l’élément *A* devient immédiatement visible pour la transaction *txnB*.
* Le comportement *TryPeekAsync* peut être implémenté à l’aide de *TryDequeueAsync* puis abandonner la transaction. Ce comportement est illustré dans la section Modèles de programmation.
* Count est non transactionnel. Il peut être utilisé pour obtenir une idée du nombre d’éléments présents dans la file d’attente, mais représente un point dans le temps non fiable.
* Un traitement coûteux sur les éléments retirés de la file d’attente ne doit pas être effectué pendant que la transaction est active, afin d’éviter des transactions à long terme qui peuvent avoir une incidence sur les performances du système.

## <a name="code-snippets"></a>Extraits de code
Examinons quelques extraits de code et leurs sorties attendues. La gestion des exceptions est ignorée dans cette section.

### <a name="instantiation"></a>Instanciation
La création d’une instance d’une file d’attente simultanée fiable est similaire à toute autre collection fiable.

```csharp
IReliableConcurrentQueue<int> queue = await this.StateManager.GetOrAddAsync<IReliableConcurrentQueue<int>>("myQueue");
```

### <a name="enqueueasync"></a>EnqueueAsync
Voici quelques extraits de code pour l’utilisation d’EnqueueAsync, suivis de leurs sorties attendues.

- *Cas 1 : tâche de mise en file d’attente unique*

```
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.EnqueueAsync(txn, 10, cancellationToken);
    await this.Queue.EnqueueAsync(txn, 20, cancellationToken);

    await txn.CommitAsync();
}
```

Supposons que la tâche s’est achevée correctement et qu’aucune transaction simultanée ne modifiait la file d’attente. L’utilisateur peut s’attendre à ce que la file d’attente contienne les éléments dans n’importe lequel des ordres suivants :

> 10, 20
> 
> 20, 10


- *Cas 2 : tâche de mise en file d’attente parallèle*

```
// Parallel Task 1
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.EnqueueAsync(txn, 10, cancellationToken);
    await this.Queue.EnqueueAsync(txn, 20, cancellationToken);

    await txn.CommitAsync();
}

// Parallel Task 2
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.EnqueueAsync(txn, 30, cancellationToken);
    await this.Queue.EnqueueAsync(txn, 40, cancellationToken);

    await txn.CommitAsync();
}
```

Supposons que les tâches aient été accomplies avec succès, qu’elles aient été exécutées en parallèle et qu’aucune autre transaction simultanée modifiant la file d’attente n’ait eu lieu. Il n’est pas possible d’inférer l’ordre des éléments dans la file d’attente. Pour cet extrait de code, les éléments peuvent apparaître dans les 4! ordres possibles.  La file d’attente essaie de conserver les éléments dans l’ordre (de mise en file d’attente) d’origine, mais peut être obligée à les réorganiser en raison d’opérations simultanées ou d’erreurs.


### <a name="dequeueasync"></a>DequeueAsync
Voici quelques extraits de code pour l’utilisation de TryDequeueAsync, suivis des résultats attendus. Supposons que la file d’attente contient déjà les éléments suivants :
> 10, 20, 30, 40, 50, 60

- *Cas 1 : simple tâche de retrait de file d’attente*

```
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.TryDequeueAsync(txn, cancellationToken);
    await this.Queue.TryDequeueAsync(txn, cancellationToken);
    await this.Queue.TryDequeueAsync(txn, cancellationToken);

    await txn.CommitAsync();
}
```

Supposons que la tâche s’est achevée correctement et qu’aucune transaction simultanée ne modifiait la file d’attente. Dans la mesure où aucune inférence ne peut être effectuée concernant l’ordre des éléments dans la file d’attente, trois éléments quelconques peuvent être retirés de celle-ci, dans n’importe quel ordre. La file d’attente essaie de conserver les éléments dans l’ordre (de mise en file d’attente) d’origine, mais peut être obligée à les réorganiser en raison d’opérations simultanées ou d’erreurs.  

- *Cas 2 : tâche de retrait de file d’attente parallèle*

```
// Parallel Task 1
List<int> dequeue1;
using (var txn = this.StateManager.CreateTransaction())
{
    dequeue1.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;
    dequeue1.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;

    await txn.CommitAsync();
}

// Parallel Task 2
List<int> dequeue2;
using (var txn = this.StateManager.CreateTransaction())
{
    dequeue2.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;
    dequeue2.Add(await this.Queue.TryDequeueAsync(txn, cancellationToken)).val;

    await txn.CommitAsync();
}
```

Supposons que les tâches aient été accomplies avec succès, qu’elles aient été exécutées en parallèle et qu’aucune autre transaction simultanée modifiant la file d’attente n’ait eu lieu. Dans la mesure où aucune inférence ne peut être effectuée concernant l’ordre des éléments dans la file d’attente, les listes *dequeue1* et *dequeue2* contiennent chacune deux éléments quelconques, dans n’importe quel ordre.

Un même élément n’apparaît *pas* dans les deux listes. Par conséquent, si la liste dequeue1 comprend *10* et *30*, la liste dequeue2 contient *20* et *40*.

- *Cas 3 : ordre de retrait de file d’attente avec abandon de transaction*

L’abandon d’une transaction avec des retraits de file d’attente a pour effet de remettre les éléments en tête de la file d’attente. L’ordre dans lequel les éléments sont remis en tête de la file d’attente n’est pas garanti. Examinons le code suivant :

```
using (var txn = this.StateManager.CreateTransaction())
{
    await this.Queue.TryDequeueAsync(txn, cancellationToken);
    await this.Queue.TryDequeueAsync(txn, cancellationToken);

    // Abort the transaction
    await txn.AbortAsync();
}
```
Supposons que les éléments ont été retirés de la file d’attente dans l’ordre suivant :
> 10, 20

Lors de l’abandon de la transaction, les éléments seraient rajoutés en tête de la file d’attente dans l’un des ordres suivants :
> 10, 20
> 
> 20, 10

Il en va de même dans tous les cas où la transaction n’a pas été *validée*.

## <a name="programming-patterns"></a>Modèles de programmation
Dans cette section, nous examinons quelques modèles de programmation qui peuvent s’avérer utiles lors de l’utilisation de ReliableConcurrentQueue.

### <a name="batch-dequeues"></a>Retraits de file d’attente par lot
Un modèle de programmation recommandé est que la tâche cliente traite en lots les retraits de file d’attente au lieu d’effectuer un retrait à la fois. L’utilisateur peut choisir de limiter les délais entre chaque lot ou la taille des lots. L’extrait de code suivant illustre ce modèle de programmation. Notez que, dans cet exemple, le traitement est effectué une fois la transaction validée. Dès lors, si une erreur se produit lors du traitement, les éléments non traités sont perdus sans avoir été traités.  Le traitement peut également être effectué dans l’étendue de la transaction, mais il peut avoir une incidence négative sur les performances et nécessiter de gérer les éléments déjà traités.

```
int batchSize = 5;
long delayMs = 100;

while(!cancellationToken.IsCancellationRequested)
{
    // Buffer for dequeued items
    List<int> processItems = new List<int>();

    using (var txn = this.StateManager.CreateTransaction())
    {
        ConditionalValue<int> ret;

        for(int i = 0; i < batchSize; ++i)
        {
            ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);

            if (ret.HasValue)
            {
                // If an item was dequeued, add to the buffer for processing
                processItems.Add(ret.Value);
            }
            else
            {
                // else break the for loop
                break;
            }
        }

        await txn.CommitAsync();
    }

    // Process the dequeues
    for (int i = 0; i < processItems.Count; ++i)
    {
        Console.WriteLine("Value : " + processItems[i]);
    }

    int delayFactor = batchSize - processItems.Count;
    await Task.Delay(TimeSpan.FromMilliseconds(delayMs * delayFactor), cancellationToken);
}
```

### <a name="best-effort-notification-based-processing"></a>Traitement basé sur une notification d’effort optimal
Un autre modèle de programmation intéressant utilise l’API Count. Ici, nous pouvons implémenter un traitement basé sur une notification d’effort optimal pour la file d’attente. Le dénombrement (Count) des éléments contenus dans la file d’attente permet de limiter une tâche de mise en file d’attente ou de retrait de file d’attente.  Notez que, comme dans l’exemple précédent, le traitement se produisant en dehors de la transaction, les éléments non traités peuvent être perdus si une erreur se produit en cours de traitement.

```
int threshold = 5;
long delayMs = 1000;

while(!cancellationToken.IsCancellationRequested)
{
    while (this.Queue.Count < threshold)
    {
        cancellationToken.ThrowIfCancellationRequested();

        // If the queue does not have the threshold number of items, delay the task and check again
        await Task.Delay(TimeSpan.FromMilliseconds(delayMs), cancellationToken);
    }

    // If there are approximately threshold number of items, try and process the queue

    // Buffer for dequeued items
    List<int> processItems = new List<int>();

    using (var txn = this.StateManager.CreateTransaction())
    {
        ConditionalValue<int> ret;

        do
        {
            ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);

            if (ret.HasValue)
            {
                // If an item was dequeued, add to the buffer for processing
                processItems.Add(ret.Value);
            }
        } while (processItems.Count < threshold && ret.HasValue);

        await txn.CommitAsync();
    }

    // Process the dequeues
    for (int i = 0; i < processItems.Count; ++i)
    {
        Console.WriteLine("Value : " + processItems[i]);
    }
}
```

### <a name="best-effort-drain"></a>Drainage d’effort optimal
Un drainage de la file d’attente ne peut pas être garanti en raison de la nature simultanée de la structure de données.  Il se peut que, même si aucune opération d’utilisateur sur la file d’attente n’est en cours, un appel particulier de TryDequeueAsync ne retourne pas un élément précédemment mis en file d’attente et validé.  Il est garanti que l’élément en file d’attente deviendra *finalement* visible pour retrait de la file d’attente. Toutefois, à défaut d’un mécanisme de communication hors bande, un client indépendant ne peut pas savoir que la file d’attente a atteint un état stable, même si tous les producteurs ont été arrêtés et si aucune nouvelle opération de mise en file d’attente n’est autorisée. Par conséquent, l’opération de drainage est implémentée selon le principe de l’effort optimal, comme ci-dessous.

L’utilisateur doit arrêter toutes les tâches de producteur et de client, et attendre que toutes les transactions en cours soient validées ou abandonnées, avant de tenter de drainer la file d’attente.  Si l’utilisateur connaît le nombre attendu d’éléments dans la file d’attente, il peut définir une notification qui signale que tous les éléments ont été retirés de la file d’attente.

```
int numItemsDequeued;
int batchSize = 5;

ConditionalValue ret;

do
{
    List<int> processItems = new List<int>();

    using (var txn = this.StateManager.CreateTransaction())
    {
        do
        {
            ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);

            if(ret.HasValue)
            {
                // Buffer the dequeues
                processItems.Add(ret.Value);
            }
        } while (ret.HasValue && processItems.Count < batchSize);

        await txn.CommitAsync();
    }

    // Process the dequeues
    for (int i = 0; i < processItems.Count; ++i)
    {
        Console.WriteLine("Value : " + processItems[i]);
    }
} while (ret.HasValue);
```

### <a name="peek"></a>Aperçu
ReliableConcurrentQueue ne fournit pas l’API *TryPeekAsync*. Les utilisateurs peuvent obtenir la sémantique d’aperçu en utilisant *TryDequeueAsync*, puis en abandonnant la transaction. Dans cet exemple, les retraits de file d’attente sont traités uniquement si la valeur est supérieure à *10*.

```
using (var txn = this.StateManager.CreateTransaction())
{
    ConditionalValue ret = await this.Queue.TryDequeueAsync(txn, cancellationToken);
    bool valueProcessed = false;

    if (ret.HasValue)
    {
        if (ret.Value > 10)
        {
            // Process the item
            Console.WriteLine("Value : " + ret.Value);
            valueProcessed = true;
        }
    }

    if (valueProcessed)
    {
        await txn.CommitAsync();    
    }
    else
    {
        await txn.AbortAsync();
    }
}
```

## <a name="must-read"></a>À lire
* [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start.md)
* [Utilisation des collections fiables](service-fabric-work-with-reliable-collections.md)
* [Notifications Reliable Services](service-fabric-reliable-services-notifications.md)
* [Sauvegarde et restauration de Reliable Services (récupération d’urgence)](service-fabric-reliable-services-backup-restore.md)
* [Configuration du Gestionnaire d’état fiable](service-fabric-reliable-services-configuration.md)
* [Prise en main des services API Web de Service Fabric](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Utilisation avancée du modèle de programmation de Reliable Services](./service-fabric-reliable-services-lifecycle.md)
* [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
