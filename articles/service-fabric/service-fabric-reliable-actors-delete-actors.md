---
title: Supprimer des acteurs Azure Service Fabric
description: Découvrez comment supprimer manuellement et entièrement des Reliable Actors et leur état dans une application Azure Service Fabric.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 16d4ab6a3c155f897cf9212fb1cd6c34d977b9ec
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574018"
---
# <a name="delete-reliable-actors-and-their-state"></a>Supprimer des Reliable Actors et leur état
Le Garbage Collection des acteurs désactivés nettoie uniquement l’objet acteur, mais il ne supprime pas les données stockées dans le Gestionnaire d’état d’un acteur. Lorsqu’un acteur est réactivé, ses données sont de nouveau rendues disponibles par le biais du Gestionnaire d’état. Dans les cas où les acteurs stockent des données dans le Gestionnaire d’état et sont désactivés mais jamais réactivés, il peut être nécessaire de nettoyer leurs données.

Le [Service d’acteur](service-fabric-reliable-actors-platform.md) fournit une fonction de suppression des acteurs à partir d’un appelant à distance :

```csharp
ActorId actorToDelete = new ActorId(id);

IActorService myActorServiceProxy = ActorServiceProxy.Create(
    new Uri("fabric:/MyApp/MyService"), actorToDelete);

await myActorServiceProxy.DeleteActorAsync(actorToDelete, cancellationToken)
```
```Java
ActorId actorToDelete = new ActorId(id);

ActorService myActorServiceProxy = ActorServiceProxy.create(
    new Uri("fabric:/MyApp/MyService"), actorToDelete);

myActorServiceProxy.deleteActorAsync(actorToDelete);
```

La suppression d’un acteur a les effets suivants selon que l’acteur est actuellement actif ou pas :

* **Acteur actif**
  * L’acteur est supprimé de la liste des acteurs actifs et est désactivé.
  * Son état est définitivement supprimé.
* **Acteur inactif**
  * Son état est définitivement supprimé.

Un acteur ne peut pas effectuer un appel de suppression sur lui-même à partir de l’une de ses méthodes d’acteur, car l’acteur ne peut pas être supprimé pendant qu’il est exécuté dans un contexte d’appel d’acteur, dans lequel le runtime a obtenu un verrou autour de l’appel d’acteur pour autoriser l’accès monothread.

Pour plus d’informations sur Reliable Actors, consultez les rubriques suivantes :
* [Minuteries et rappels d’acteur](service-fabric-reliable-actors-timers-reminders.md)
* [Événements d’acteurs](service-fabric-reliable-actors-events.md)
* [Réentrance des acteurs](service-fabric-reliable-actors-reentrancy.md)
* [Diagnostics et surveillance des performances d’acteur](service-fabric-reliable-actors-diagnostics.md)
* [Documentation de référence de l’API d’acteur](/previous-versions/azure/dn971626(v=azure.100))
* [Exemple de code C#](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Exemple de code Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-lifecycle/garbage-collection.png
