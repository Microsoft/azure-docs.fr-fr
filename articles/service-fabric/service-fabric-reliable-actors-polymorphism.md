---
title: Polymorphisme dans l’infrastructure Reliable Actors
description: Concevez des hiérarchies d’interfaces et de types .NET dans l’infrastructure Reliable Actors afin de réutiliser des fonctionnalités et des définitions d'API.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 568e306979e862c325264a4e12a64d95e6d13c5f
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575990"
---
# <a name="polymorphism-in-the-reliable-actors-framework"></a>Polymorphisme dans l’infrastructure Reliable Actors
L’infrastructure Reliable Actors vous permet de générer des acteurs en utilisant bon nombre des mêmes techniques que celles employées dans la conception orientée objet. L’une de ces techniques est le polymorphisme, qui permet à des types et à des interfaces d’hériter de parents plus généralisés. L’héritage dans l’infrastructure Reliable Actors suit généralement le modèle .NET avec quelques contraintes supplémentaires. Dans le cas de Java/Linux, il suit le modèle Java.

## <a name="interfaces"></a>Interfaces
L’infrastructure Reliable Actors exige que vous définissiez au moins une interface que votre type d’acteur doit implémenter. Cette interface est utilisée pour générer une classe proxy, utilisable par des clients pour communiquer avec vos acteurs. Les interfaces peuvent hériter d’autres interfaces dès lors que toutes les interfaces implémentées par un type d’acteur et tous ses parents dérivent au final d’IActor (C#) ou d’Actor (Java). IActor(C#) et Actor(Java) sont les interfaces de base définies par la plateforme pour les acteurs dans les infrastructures .NET et Java, respectivement. Par conséquent, voici à quoi peut ressembler un exemple de polymorphisme classique qui utilise des formes :

![Hiérarchie d'interfaces pour les acteurs de forme][shapes-interface-hierarchy]

## <a name="types"></a>Types
Vous pouvez également créer une hiérarchie des types d’acteur, qui sont dérivés de la classe de base Actor fournie par la plateforme. Dans le cas de formes, vous pouvez avoir un type `Shape` (C#) ou `ShapeImpl` (Java) de base :

```csharp
public abstract class Shape : Actor, IShape
{
    public abstract Task<int> GetVerticeCount();

    public abstract Task<double> GetAreaAsync();
}
```
```Java
public abstract class ShapeImpl extends FabricActor implements Shape
{
    public abstract CompletableFuture<int> getVerticeCount();

    public abstract CompletableFuture<double> getAreaAsync();
}
```

Les sous-types de `Shape` (C#) ou `ShapeImpl` (Java) peuvent remplacer des méthodes de la base.

```csharp
[ActorService(Name = "Circle")]
[StatePersistence(StatePersistence.Persisted)]
public class Circle : Shape, ICircle
{
    public override Task<int> GetVerticeCount()
    {
        return Task.FromResult(0);
    }

    public override async Task<double> GetAreaAsync()
    {
        CircleState state = await this.StateManager.GetStateAsync<CircleState>("circle");

        return Math.PI *
            state.Radius *
            state.Radius;
    }
}
```
```Java
@ActorServiceAttribute(name = "Circle")
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class Circle extends ShapeImpl implements Circle
{
    @Override
    public CompletableFuture<Integer> getVerticeCount()
    {
        return CompletableFuture.completedFuture(0);
    }

    @Override
    public CompletableFuture<Double> getAreaAsync()
    {
        return (this.stateManager().getStateAsync<CircleState>("circle").thenApply(state->{
          return Math.PI * state.radius * state.radius;
        }));
    }
}
```

Notez l’attribut `ActorService` sur le type d'intervenant. Cet attribut indique à l’infrastructure Reliable Actor qu’elle doit automatiquement créer un service pour héberger des acteurs de ce type. Dans certains cas, vous pouvez souhaiter créer un type de base destiné exclusivement à la fonctionnalité de partage avec des sous-types, qui ne sera jamais utilisé pour instancier des acteurs concrets. Dans ce cas, vous devez utiliser le mot clé `abstract` pour indiquer que vous ne créerez jamais d’acteur basé sur ce type.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez [comment l’infrastructure Reliable Actors s’appuie sur la plateforme Service Fabric](service-fabric-reliable-actors-platform.md) pour garantir la fiabilité, l’extensibilité et la cohérence.
* En savoir plus sur le [cycle de vie des acteurs](service-fabric-reliable-actors-lifecycle.md)

<!-- Image references -->

[shapes-interface-hierarchy]: ./media/service-fabric-reliable-actors-polymorphism/Shapes-Interface-Hierarchy.png
