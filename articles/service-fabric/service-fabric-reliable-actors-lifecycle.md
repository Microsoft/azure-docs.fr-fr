---
title: Vue d’ensemble du cycle de vie des acteurs Azure Service Fabric
description: Explique le cycle de vie Service Fabric Reliable Actor, le Garbage Collection et la suppression manuelle des acteurs et de leur état
ms.topic: conceptual
ms.date: 10/06/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 88db4bb2376cbc418d6954e274a18a6c18a280d1
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576041"
---
# <a name="actor-lifecycle-automatic-garbage-collection-and-manual-delete"></a>Cycle de vie des acteurs, Garbage Collection automatique et suppression manuelle
Un acteur est activé la première fois qu’un appel est effectué à l’une de ses méthodes. Un acteur est désactivé (fait l’objet d’un Garbage Collection par le runtime Actors) s’il n’est pas utilisé pendant une durée configurable. Un acteur et son état peuvent également être supprimés manuellement, à tout moment.

## <a name="actor-activation"></a>Activation de l’acteur
Quand un acteur est activé, les événements suivants se produisent :

* Quand un appel est émis vers un acteur qui n'est pas actif, un autre acteur est créé.
* L’état de l’acteur est chargé s’il maintient l’état.
* La méthode `OnActivateAsync` (C#) ou `onActivateAsync` (Java), qui peut être remplacée dans l’implémentation de l’acteur, est appelée.
* L’acteur est désormais considéré comme actif.

## <a name="actor-deactivation"></a>Désactivation de l’acteur
Quand un acteur est désactivé, les événements suivants se produisent :

* Quand un acteur n'est pas utilisé pendant un certain temps, il est supprimé de la table d'acteurs actifs.
* La méthode `OnDeactivateAsync` (C#) ou `onDeactivateAsync` (Java), qui peut être remplacée dans l’implémentation de l’acteur, est appelée. Cette opération efface toutes les minuteries applicables à l'acteur. Les opérations de l’acteur, comme la modification de l’état, ne doivent pas être appelées avec cette méthode.

> [!TIP]
> Le runtime Fabric Actors émet des [événements liés à l’activation et la désactivation des acteurs](service-fabric-reliable-actors-diagnostics.md#list-of-events-and-performance-counters). Ces événements sont utiles dans les diagnostics et la surveillance des performances.
>
>

### <a name="actor-garbage-collection"></a>Garbage Collection des acteurs
Quand un acteur est désactivé, les références à l’objet acteur sont libérées et celui-ci peut faire l’objet d’un garbage collection normal par le récupérateur de mémoire CLR (common language runtime) ou de la machine virtuelle Java (JVM). L’opération Garbage Collection nettoie uniquement l’objet acteur. Elle ne supprime **pas** l’état stocké dans le Gestionnaire d’état de l’acteur. La prochaine fois que l’acteur est activé, un nouvel objet acteur est créé et son état est restauré.

Que signifie « être utilisé » dans le cadre de la désactivation et du Garbage Collection ?

* Réception d'un appel
* `IRemindable.ReceiveReminderAsync` (applicable uniquement si l'acteur utilise des rappels)

> [!NOTE]
> Si l’acteur utilise des minuteries et que son rappel de minuterie est appelé, cela ne signifie **pas** qu’il est « utilisé ».
>
>

Avant d’aborder les détails de la désactivation, il est important de définir les termes suivants :

* *Intervalle d'analyse*. Il s’agit de l’intervalle pendant lequel le runtime Actors recherche dans sa table d’acteurs actifs les acteurs qui peuvent faire l’objet d’une désactivation ou d’un Garbage Collection. La valeur par défaut est 1 minute.
* *Délai d'inactivité*. Il s’agit de la durée pendant laquelle un acteur reste inutilisé (inactif) avant de faire l’objet d’une désactivation ou d’un Garbage Collection. La valeur par défaut est 60 minutes.

En général, vous n'avez pas besoin de modifier les valeurs par défaut. Toutefois, si nécessaire, ces intervalles peuvent être modifiés via `ActorServiceSettings` lorsque vous enregistrez votre [Service d’acteur](service-fabric-reliable-actors-platform.md):

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        ActorRuntime.RegisterActorAsync<MyActor>((context, actorType) =>
                new ActorService(context, actorType,
                    settings:
                        new ActorServiceSettings()
                        {
                            ActorGarbageCollectionSettings =
                                new ActorGarbageCollectionSettings(10, 2)
                        }))
            .GetAwaiter()
            .GetResult();
    }
}
```

```Java
public class Program
{
    public static void main(String[] args)
    {
        ActorRuntime.registerActorAsync(
                MyActor.class,
                (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
                timeout);
    }
}
```
Pour chaque acteur actif, le runtime Actors effectue le suivi de la durée pendant laquelle il a été inactif (c’est-à-dire non utilisé). Le runtime Actors vérifie chacun des acteurs toutes les `ScanIntervalInSeconds` pour voir s’il peut faire l’objet d’un Garbage Collection, et le marque s’il est inactif depuis `IdleTimeoutInSeconds`.

Chaque fois qu'un acteur est utilisé, son délai d'inactivité est réinitialisé à 0. Ensuite, l'acteur peut uniquement faire l'objet d'un Garbage Collection s'il reste encore inactif pendant `IdleTimeoutInSeconds`. N’oubliez pas qu’un acteur est considéré utilisé si une méthode d’interface d’acteur ou un rappel de rappel d’acteur est exécuté. Un acteur n'est **pas** considéré utilisé si son rappel de minuterie est exécuté.

Le diagramme suivant illustre le cycle de vie d’un seul acteur pour illustrer ces concepts.

![Exemple de temps d'inactivité][1]

L'exemple montre l'impact des appels de méthode d'acteur, les rappels et les minuteries sur la durée de vie de cet acteur. Voici les points importants de l'exemple :

* ScanInterval et IdleTimeout sont définis sur 5 et 10, respectivement. (Les unités n'importent pas ici, dans la mesure où notre objectif se borne à illustrer le concept.)
* La recherche d'acteurs ayant fait l'objet d'un Garbage Collection s'effectue à T=0,5,10,15,20,25, comme défini par la valeur 5 de l'intervalle d'analyse.
* Une minuterie périodique se déclenche à T=4,8,12,16,20,24 et son rappel s'exécute. Il n'affecte pas la durée d'inactivité de l'acteur.
* Un appel de méthode d'acteur à T=7 réinitialise la durée d'inactivité à 0 et retarde le Garbage Collection de l'acteur.
* Un rappel de rappel d'acteur s'exécute à T=14 et retarde davantage le Garbage Collection de l'acteur.
* Lors de l'analyse de Garbage Collection à T=25, la durée d'inactivité de l'acteur dépasse la valeur 10 du paramètre Délai d'inactivité et l'acteur fait l'objet d'un Garbage Collection.

Un acteur ne peut jamais faire l’objet d’un Garbage Collection quand il exécute l’une de ses méthodes, quelle que soit la durée d’exécution de cette méthode. Comme mentionné précédemment, l'exécution des méthodes d'interface d'acteur et des rappels de rappel empêche le Garbage Collection en réinitialisant la durée d'inactivité de l'acteur à 0. L'exécution des rappels de minuterie ne réinitialise pas la durée d'inactivité à 0. Toutefois, le Garbage Collection de l'acteur est différé jusqu'à ce que le rappel de minuterie ait terminé son exécution.

## <a name="manually-deleting-actors-and-their-state"></a>Suppression manuelle d’acteurs et de leur état
Le Garbage Collection des acteurs désactivés nettoie uniquement l’objet acteur, mais il ne supprime pas les données stockées dans le Gestionnaire d’état d’un acteur. Lorsqu’un acteur est réactivé, ses données sont de nouveau rendues disponibles par le biais du Gestionnaire d’état. Dans les cas où les acteurs stockent des données dans le Gestionnaire d’état et sont désactivés mais jamais réactivés, il peut être nécessaire de nettoyer leurs données.  Pour obtenir des exemples montrant comment supprimer des acteurs, lire [Supprimer des acteurs et leur état](service-fabric-reliable-actors-delete-actors.md).

## <a name="next-steps"></a>Étapes suivantes
* [Minuteries et rappels d’acteur](service-fabric-reliable-actors-timers-reminders.md)
* [Événements d’acteurs](service-fabric-reliable-actors-events.md)
* [Réentrance des acteurs](service-fabric-reliable-actors-reentrancy.md)
* [Diagnostics et surveillance des performances d’acteur](service-fabric-reliable-actors-diagnostics.md)
* [Documentation de référence de l’API d’acteur](/previous-versions/azure/dn971626(v=azure.100))
* [Exemple de code C#](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Exemple de code Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-lifecycle/garbage-collection.png
