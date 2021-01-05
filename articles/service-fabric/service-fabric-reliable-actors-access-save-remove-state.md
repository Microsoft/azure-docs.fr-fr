---
title: Gérer l’état d’Azure Service Fabric
description: Découvrez l’accès à l’état ainsi que l’enregistrement et la suppression de celui-ci pour un service Azure Service Fabric Reliable Actor et les éléments à prendre en compte lors de la conception d’une application.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 1e796232719342883efc2aa5dd377d586f3039ad
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571315"
---
# <a name="access-save-and-remove-reliable-actors-state"></a>Utiliser, enregistrer et supprimer l’état de Reliable Actors
[Reliable Actors](service-fabric-reliable-actors-introduction.md) désignent des objets monothread capables d’encapsuler la logique et l’état, et de maintenir l’état de manière fiable. Chaque instance d’acteur possède son propre [gestionnaire d’état](service-fabric-reliable-actors-state-management.md), c’est-à-dire une structure de données de type dictionnaire qui stocke les paires clé/valeur de manière fiable. Le gestionnaire d’état est un wrapper autour d’un fournisseur d’état. Vous pouvez l’utiliser pour stocker des données, quel que soit le [réglage de persistance](service-fabric-reliable-actors-state-management.md#state-persistence-and-replication) utilisé.

Les clés de gestionnaire d’état doivent être des chaînes. Les valeurs sont génériques et peuvent être de n’importe quel type, y compris de types personnalisés. Les valeurs stockées dans le gestionnaire d’état doivent être sérialisables en contrat de données, car elles peuvent être transmises sur le réseau vers d’autres nœuds pendant la réplication et peuvent être écrites sur le disque, en fonction du paramètre de persistance d’état d’un acteur.

Pour la gestion des états, le gestionnaire d’état expose des méthodes de dictionnaire courantes similaires à celles disponibles dans Reliable Dictionary.

Pour plus d’informations, consultez les [meilleures pratiques relative à la gestion de l’état d’acteur](service-fabric-reliable-actors-state-management.md#best-practices).

## <a name="access-state"></a>Accès à l’état
L’état est accessible via le gestionnaire d’état par l’intermédiaire d’une clé. Les méthodes du gestionnaire d’état sont toutes asynchrones, car elles peuvent nécessiter des E/S disque lorsque les acteurs sont à l’état persistant. Lors du premier accès, les objets d’état sont mis en mémoire cache. Les opérations d’accès répétées permettent d’accéder aux objets directement à partir de la mémoire et sont retournées de façon synchrone sans entraîner d’E/S disque ou de surcharge asynchrone en cas de changement de contexte. Un objet d’état est supprimé du cache dans les cas suivants :

* Une méthode d’acteur lève une exception non gérée après avoir récupéré un objet à partir du gestionnaire d’état.
* Un acteur est réactivé soit après avoir été désactivé, soit en raison d’un échec.
* Si le fournisseur d’état écrit l’état sur le disque. Ce comportement dépend de l’implémentation du fournisseur d’état. Le fournisseur d’état par défaut pour le paramètre `Persisted` présente ce comportement.

Vous pouvez récupérer l’état à l’aide d’une opération *Get* standard qui lève l’exception `KeyNotFoundException`(C#) ou `NoSuchElementException`(Java) s’il n’existe aucune entrée pour la clé :

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<int> GetCountAsync()
    {
        return this.StateManager.GetStateAsync<int>("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().getStateAsync("MyState");
    }
}
```

Vous pouvez également récupérer l’état à l’aide d’une opération *TryGet* qui ne lève pas d’exception s’il n’existe aucune entrée pour la clé :

```csharp
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task<int> GetCountAsync()
    {
        ConditionalValue<int> result = await this.StateManager.TryGetStateAsync<int>("MyState");
        if (result.HasValue)
        {
            return result.Value;
        }

        return 0;
    }
}
```
```Java
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().<Integer>tryGetStateAsync("MyState").thenApply(result -> {
            if (result.hasValue()) {
                return result.getValue();
            } else {
                return 0;
            });
    }
}
```

## <a name="save-state"></a>Enregistrer l’état
Les méthodes de récupération du gestionnaire d’état renvoient une référence à un objet dans la mémoire locale. La modification de cet objet dans la mémoire locale uniquement ne permet pas de l’enregistrer durablement. Lorsqu’un objet est récupéré à partir du gestionnaire d’état puis modifié, il doit être réinséré dans le gestionnaire d’état afin d’être enregistré de façon durable.

Vous pouvez insérer l’état en utilisant une méthode *Set* inconditionnelle, ce qui équivaut à la syntaxe `dictionary["key"] = value` :

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task SetCountAsync(int value)
    {
        return this.StateManager.SetStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture setCountAsync(int value)
    {
        return this.stateManager().setStateAsync("MyState", value);
    }
}
```

Vous pouvez ajouter l’état en utilisant une méthode *Add*. Cette méthode lève une exception `InvalidOperationException`(C#) ou `IllegalStateException`(Java) lorsqu’elle tente d’ajouter une clé qui existe.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task AddCountAsync(int value)
    {
        return this.StateManager.AddStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().addOrUpdateStateAsync("MyState", value, (key, old_value) -> old_value + value);
    }
}
```

Vous pouvez également ajouter l’état en utilisant une méthode *TryAdd*. Cette méthode ne lève pas d’exception lorsqu’elle tente d’ajouter une clé qui existe déjà.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task AddCountAsync(int value)
    {
        bool result = await this.StateManager.TryAddStateAsync<int>("MyState", value);

        if (result)
        {
            // Added successfully!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().tryAddStateAsync("MyState", value).thenApply((result)->{
            if(result)
            {
                // Added successfully!
            }
        });
    }
}
```

À la fin d’une méthode d’acteur, le gestionnaire d’état enregistre automatiquement toutes les valeurs qui ont été ajoutées ou modifiées par une opération insert ou update. Une opération « save » peut inclure la conservation sur disque et la réplication, selon les paramètres utilisés. Les valeurs qui n’ont pas été modifiées ne sont pas conservées ou répliquées. Si aucune valeur n’a été modifiée, l’opération n’aura aucun effet. En cas d’échec de l’enregistrement, l’état modifié est ignoré et l’état d’origine est rechargé.

Vous pouvez également enregistrer l’état manuellement en appelant la méthode `SaveStateAsync` sur la base d’acteur :

```csharp
async Task IMyActor.SetCountAsync(int count)
{
    await this.StateManager.AddOrUpdateStateAsync("count", count, (key, value) => count > value ? count : value);

    await this.SaveStateAsync();
}
```
```Java
interface MyActor {
    CompletableFuture setCountAsync(int count)
    {
        this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value).thenApply();

        this.stateManager().saveStateAsync().thenApply();
    }
}
```

## <a name="remove-state"></a>Supprimer l’état
Vous pouvez supprimer définitivement l’état du gestionnaire d’état d’un acteur en appelant la méthode *Remove*. Cette méthode lève une exception `KeyNotFoundException`(C#) ou `NoSuchElementException`(Java) lorsqu’elle tente de supprimer une clé qui n’existe pas.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task RemoveCountAsync()
    {
        return this.StateManager.RemoveStateAsync("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().removeStateAsync("MyState");
    }
}
```

Vous pouvez également supprimer définitivement l’état en utilisant la méthode *TryRemove*. Cette méthode ne lève pas d’exception lorsqu’elle tente de supprimer une clé qui n’existe pas.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task RemoveCountAsync()
    {
        bool result = await this.StateManager.TryRemoveStateAsync("MyState");

        if (result)
        {
            // State removed!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().tryRemoveStateAsync("MyState").thenApply((result)->{
            if(result)
            {
                // State removed!
            }
        });
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

L’état stocké dans Reliable Actors doit être sérialisé avant d’être écrit sur le disque et répliqué pour une haute disponibilité. En savoir plus sur [Sérialisation du type d’acteur](service-fabric-reliable-actors-notes-on-actor-type-serialization.md).

Ensuite, en savoir plus sur l’[analyse des performances et des diagnostics des acteurs](service-fabric-reliable-actors-diagnostics.md).
