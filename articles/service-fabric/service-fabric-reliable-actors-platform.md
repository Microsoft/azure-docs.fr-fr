---
title: Reliable Actors dans Service Fabric
description: Décrit comment les Reliable Actors sont superposés en couches sur des Reliable Services et utilisent les fonctionnalités de la plateforme Service Fabric.
ms.topic: conceptual
ms.date: 3/9/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: c3bd67e0f71e20a8b5a74b4b38e5b423a14dc08d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575616"
---
# <a name="how-reliable-actors-use-the-service-fabric-platform"></a>Comment le service Reliable Actors utilise la plateforme Service Fabric
Cet article décrit le fonctionnement du service Reliable Actors sur la plateforme Azure Service Fabric. La solution Reliable Actors s’exécute dans une infrastructure hébergée dans une implémentation d’un service fiable avec état nommé *service d’acteur*. Le service d’acteur contient tous les composants nécessaires pour gérer le cycle de vie et la distribution des messages destinés à vos acteurs :

* Le runtime de l’acteur gère le cycle de vie et le nettoyage de la mémoire, et applique une accès monothread.
* Un écouteur de communication à distance du service d’acteur accepte les appels d’accès à distance adressés aux acteurs, et les transmet à un répartiteur qui les route vers l’instance d’acteur appropriée.
* Le fournisseur d’état de l’acteur encapsule des fournisseurs d’état (par exemple, le fournisseur d’état Collections fiables), et fournit un adaptateur pour la gestion de l’état de l’acteur.

Ces composants forment ensemble l’infrastructure d’acteur fiable.

## <a name="service-layering"></a>Couches de service
Étant donné que le service d’acteur est un service fiable, l’ensemble des concepts des Reliable Services relatifs au [modèle d’application](service-fabric-application-model.md), au cycle de vie, à [l’empaquetage](service-fabric-package-apps.md), au [déploiement](service-fabric-deploy-remove-applications.md), à la mise à niveau et à la mise à l’échelle s’appliquent de la même manière aux services d’acteur.

![Superposition de service d’acteur][1]

Le diagramme précédent montre la relation entre les infrastructures d’application de Service Fabric et le code utilisateur. Les éléments en bleu représentent l’infrastructure d’application de Reliable Services, ceux en orange l’infrastructure de Reliable Actors, et ceux en vert le code utilisateur.

Dans Reliable Services, votre service hérite de la classe `StatefulService`. Cette classe est dérivée de `StatefulServiceBase` (ou `StatelessService` pour les services sans état). Dans les Reliable Actors, vous utilisez le service d’acteur. Le service d’acteur est une implémentation différente de la classe `StatefulServiceBase` qui implémente le modèle d’acteur dans lequel vos acteurs s’exécutent. Le service d’acteur proprement dit est simplement une implémentation de `StatefulServiceBase`. Vous pouvez donc écrire votre propre service qui dérive de `ActorService` et implémenter des fonctionnalités au niveau du service de la même façon que si vous héritiez de `StatefulService`, par exemple :

* Sauvegarde et restauration du service.
* Fonctionnalité partagée par tous les acteurs. Par exemple, un disjoncteur.
* Appels de procédure à distance sur le service d’acteur proprement dit et sur chaque acteur.

Pour plus d’informations, voir [Implémentation des fonctionnalités de niveau de service dans votre service d’acteur](service-fabric-reliable-actors-using.md).

## <a name="application-model"></a>Modèle d'application
Les services d’acteur étant des Reliable Services, le modèle d’application est le même. Toutefois, les outils de génération d’infrastructure d’acteur produisent certains fichiers de modèle d’application pour vous.

### <a name="service-manifest"></a>Manifeste de service
Les outils de génération d’infrastructure d’acteur génèrent automatiquement le contenu du fichier ServiceManifest.xml de votre service d’acteur. Ce fichier inclut :

* Le type de service d’acteur. Le nom du type est généré en fonction du nom du projet d’acteur. En fonction de l’attribut de persistance sur votre acteur, l’indicateur HasPersistedState est également défini en conséquence.
* le package de code ;
* le package de configuration ;
* les ressources et points de terminaison.

### <a name="application-manifest"></a>Manifeste d’application
Les outils de génération d’infrastructure d’acteur créent automatiquement une définition de service par défaut pour votre service d’acteur. Les outils de génération définissent les propriétés de service par défaut :

* Le nombre de jeux de réplicas est déterminé par l’attribut de persistance sur votre acteur. À chaque modification de l’attribut de persistance sur votre acteur, le nombre de jeux de réplicas dans la définition de service par défaut est réinitialisé en conséquence.
* La plage et le schéma de partition ont une valeur Int64 uniforme avec la plage complète de clés Int64.

## <a name="service-fabric-partition-concepts-for-actors"></a>Concepts de partition Service Fabric pour les acteurs
Les services d’acteur sont des services partitionnés avec état. Chaque partition d’un service d’acteur contient un ensemble d’acteurs. Les partitions de service sont automatiquement distribuées sur plusieurs nœuds dans Service Fabric. Par conséquent, les instances d’acteur sont distribuées.

![Partitionnement et distribution d’acteur][5]

Vous pouvez créer des Reliable Services avec différents schémas de partition et plages de clés de partition. Le service d’acteur utilise le schéma de partitionnement Int64 avec la plage complète de clés Int64 pour mapper des acteurs à des partitions.

### <a name="actor-id"></a>ID d’acteur
Chaque acteur créé dans le service possède un ID unique associé, représenté par la classe `ActorId` . `ActorId` est une valeur d’ID opaque qui peut être utilisée pour une distribution uniforme des acteurs sur les partitions de service en générant des identifiants aléatoires :

```csharp
ActorProxy.Create<IMyActor>(ActorId.CreateRandom());
```
```Java
ActorProxyBase.create<MyActor>(MyActor.class, ActorId.newId());
```


Chaque `ActorId` est haché en Int64. C’est pourquoi le service d’acteur doit utiliser un schéma de partitionnement Int64 avec la plage complète de clés Int64. Toutefois, vous pouvez utiliser des valeurs d’ID personnalisées pour un `ActorID`, dont des GUID / UUID, des chaînes et des valeurs Int64s.

```csharp
ActorProxy.Create<IMyActor>(new ActorId(Guid.NewGuid()));
ActorProxy.Create<IMyActor>(new ActorId("myActorId"));
ActorProxy.Create<IMyActor>(new ActorId(1234));
```
```Java
ActorProxyBase.create(MyActor.class, new ActorId(UUID.randomUUID()));
ActorProxyBase.create(MyActor.class, new ActorId("myActorId"));
ActorProxyBase.create(MyActor.class, new ActorId(1234));
```

Lorsque vous utilisez des chaînes et des GUID / UUID, les valeurs sont hachées en Int64. Toutefois, lorsque vous fournissez explicitement une valeur Int64 à un `ActorId`, la valeur Int64 mappe directement à une partition sans hachage supplémentaire. Vous pouvez utiliser cette technique pour contrôler la partition dans laquelle les acteurs sont placés.


## <a name="next-steps"></a>Étapes suivantes
* [Gestion des états d’acteur](service-fabric-reliable-actors-state-management.md)
* [Cycle de vie des acteurs et Garbage Collection](service-fabric-reliable-actors-lifecycle.md)
* [Documentation de référence de l’API Actors](/dotnet/api/microsoft.servicefabric.actors?view=azure-dotnet)
* [Exemple de code .NET](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Exemple de code Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
