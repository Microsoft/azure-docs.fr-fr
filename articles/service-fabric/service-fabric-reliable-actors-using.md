---
title: Implémenter des fonctionnalités dans Azure Service Fabric Actors
description: Décrit comment écrire votre propre service d’intervenant qui implémente les fonctionnalités de niveau du service de la même façon que lorsque vous héritez d’un élément StatefulService.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: d39ec93e0ad03d6c860bae9d0790e860c95457a5
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575565"
---
# <a name="implement-service-level-features-in-your-actor-service"></a>Implémenter des fonctionnalités de niveau de service dans votre service d’intervenant

Comme décrit dans les [couches de service](service-fabric-reliable-actors-platform.md#service-layering), le service d’acteur lui-même est un service fiable. Vous pouvez écrire votre propre service dérivé de `ActorService`. Vous pouvez également implémenter des fonctionnalités de niveau de service de la même façon que lorsque vous héritez d’un service avec état, par exemple :

- Sauvegarde et restauration du service.
- Fonctionnalité partagée par tous les acteurs. Par exemple, un disjoncteur.
- Appels de procédure à distance sur le service d’acteur proprement dit et sur chaque acteur.

## <a name="use-the-actor-service"></a>Utiliser le service d’intervenant

Les instances d’acteur ont accès au service d’acteur dans lequel elles s’exécutent. Via le service d’acteur, les instances d’acteur peuvent obtenir par programmation le contexte de service. Le contexte de service comprend l’ID de partition, le nom du service, le nom de l’application et d’autres informations spécifiques de la plateforme Azure Service Fabric.

```csharp
Task MyActorMethod()
{
    Guid partitionId = this.ActorService.Context.PartitionId;
    string serviceTypeName = this.ActorService.Context.ServiceTypeName;
    Uri serviceInstanceName = this.ActorService.Context.ServiceName;
    string applicationInstanceName = this.ActorService.Context.CodePackageActivationContext.ApplicationName;
}
```
```Java
CompletableFuture<?> MyActorMethod()
{
    UUID partitionId = this.getActorService().getServiceContext().getPartitionId();
    String serviceTypeName = this.getActorService().getServiceContext().getServiceTypeName();
    URI serviceInstanceName = this.getActorService().getServiceContext().getServiceName();
    String applicationInstanceName = this.getActorService().getServiceContext().getCodePackageActivationContext().getApplicationName();
}
```

Comme tous les services Reliable Services, le service d’acteur doit être inscrit avec un type de service dans le runtime de Service Fabric. Pour que le service d’intervenant exécute vos instances d’intervenant, votre type d’intervenant doit également être inscrit auprès du service d’intervenant. La méthode d’inscription `ActorRuntime` effectue ce travail pour les acteurs. Dans le cas le plus simple, vous pouvez inscrire votre type d’intervenant, et le service d’intervenant utilise les paramètres par défaut.

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>().GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```

Vous pouvez également utiliser une expression lambda fournie par la méthode d’inscription pour construire vous-même le service d’acteur. Vous pouvez ensuite configurer le service d’intervenant et construire explicitement vos instances d’intervenant. Vous pouvez injecter des dépendances à votre intervenant via son constructeur.

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new ActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```
```Java
static class Program
{
    private static void Main()
    {
      ActorRuntime.registerActorAsync(
              MyActor.class,
              (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
              timeout);

        Thread.sleep(Long.MAX_VALUE);
    }
}
```

## <a name="actor-service-methods"></a>Méthodes du service d’acteur

Le service d’intervenant implémente `IActorService` (C#) `ActorService` (Java), qui implémentent à leur tour `IService` (C#) ou `Service` (Java). Il s’agit de l’interface qu’utilise la communication à distance de Reliable Services, qui autorise des appels de procédure distante sur des méthodes de service. Elle contient des méthodes de niveau de service qui peuvent être appelées à distance via la communication à distance des services. Vous pouvez l’utiliser pour [énumérer](service-fabric-reliable-actors-enumerate.md) et [supprimer](service-fabric-reliable-actors-delete-actors.md) des intervenants.


## <a name="custom-actor-service"></a>Service d’acteur personnalisé

En utilisant la fonction lambda de l’inscription d’acteur, vous pouvez enregistrer votre propre service d’acteur personnalisé qui dérive de `ActorService` (c#) et `FabricActorService` (Java). Vous pouvez implémenter vos propres fonctionnalités de niveau de service en écrivant une classe de service qui hérite de `ActorService` (C#) ou `FabricActorService` (Java). Un service d’intervenant personnalisé hérite de toutes les fonctionnalités de runtime d’intervenant de `ActorService` (C#) ou `FabricActorService` (Java). Vous pouvez l’utiliser pour implémenter vos propres méthodes de service.

```csharp
class MyActorService : ActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<ActorBase> newActor)
        : base(context, typeInfo, newActor)
    { }
}
```
```Java
class MyActorService extends FabricActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, BiFunction<FabricActorService, ActorId, ActorBase> newActor)
    {
         super(context, typeInfo, newActor);
    }
}
```

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new MyActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
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
        Thread.sleep(Long.MAX_VALUE);
    }
}
```

## <a name="implement-actor-backup-and-restore"></a>Implémenter une sauvegarde et un restauration d’intervenant

Un service d’acteur personnalisé peut exposer une méthode pour sauvegarder des données d’acteur en tirant parti de l’écouteur de communication à distance déjà présent dans `ActorService`. Pour obtenir un exemple, consultez [Sauvegarder et restaurer des acteurs](service-fabric-reliable-actors-backup-and-restore.md).

## <a name="actor-that-uses-a-remoting-v2-interface-compatible-stack"></a>Intervenant qui utilise une pile V2 de communication à distance (compatible avec l’interface)

La pile de communication à distance V2 (compatible avec l’interface, appelée V2_1) a toutes les fonctionnalités de la pile de communication à distance V2. Son interface est compatible avec la pile de communication à distance V1, mais elle ne présente pas de compatibilité descendante avec les piles V1 et V2. Pour procéder à une mise à niveau de la pile V1 vers la pile V2_1 sans que cela ait d’incidence sur la disponibilité du service, suivez les étapes décrites dans la section suivante.

Les modifications suivantes sont requises pour utiliser la pile de communication à distance V2_1 :

1. Ajoutez l’attribut d’assembly suivant sur les interfaces d’intervenant.
  
   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
   ```

2. Créez et mettez à niveau des projets de service d’intervenant et de client d’intervenant pour commencer à utiliser la pile V2.

### <a name="actor-service-upgrade-to-remoting-v2-interface-compatible-stack-without-affecting-service-availability"></a>Mise à niveau du service d’intervenant vers la pile de communication à distance V2 (compatible avec l’interface) sans incidence sur la disponibilité du service

Cette modification est une mise à niveau en deux étapes. Suivez les étapes dans l’ordre indiqué.

1. Ajoutez l’attribut d’assembly suivant sur les interfaces d’intervenant. Cet attribut démarre deux écouteurs pour le service d’intervenant, V1 (existant) et V2_1. Mettez à niveau le service d’intervenant avec cette modification.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V1|RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
   ```

2. Mettez à niveau les clients d’intervenant après avoir terminé la mise à niveau précédente.
   Cette étape permet de s’assurer que le proxy d’intervenant utilise la pile de communication à distance V2_1.

3. Cette étape est facultative. Modifiez l’attribut précédent pour supprimer l’écouteur V1.

    ```csharp
    [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```

## <a name="actor-that-uses-the-remoting-v2-stack"></a>Intervenant qui utilise la pile de communication à distance V2

Avec le package NuGet version 2.8, les utilisateurs peuvent désormais utiliser la pile de communication à distance V2, qui est plus performante et offre des fonctionnalités telles que la sérialisation personnalisée. Le pile de communication à distance V2 ne présente pas de compatibilité descendante avec la pile de communication à distance existante (désormais appelée pile de communication à distance V1).

Les modifications suivantes sont requises pour utiliser la pile de communication à distance V2.

1. Ajoutez l’attribut d’assembly suivant sur les interfaces d’intervenant.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
   ```

2. Créez et mettez à niveau les projets de service d’intervenant et de client d’intervenant pour commencer à utiliser la pile V2.

### <a name="upgrade-the-actor-service-to-the-remoting-v2-stack-without-affecting-service-availability"></a>Mettre à niveau le service d’acteur vers la pile de communication à distance V2 sans que cela ait d’incidence sur la disponibilité du service

Cette modification est une mise à niveau en deux étapes. Suivez les étapes dans l’ordre indiqué.

1. Ajoutez l’attribut d’assembly suivant sur les interfaces d’intervenant. Cet attribut démarre deux écouteurs pour le service d’acteur, V1 (existant) et V2. Mettez à niveau le service d’intervenant avec cette modification.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V1|RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
   ```

2. Mettez à niveau les clients d’intervenant après avoir terminé la mise à niveau précédente.
   Cette étape permet de s’assurer que le proxy d’acteur utilise la pile de communication à distance V2.

3. Cette étape est facultative. Modifiez l’attribut précédent pour supprimer l’écouteur V1.

    ```csharp
    [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Gestion des états d’acteur](service-fabric-reliable-actors-state-management.md)
* [Cycle de vie des acteurs et Garbage Collection](service-fabric-reliable-actors-lifecycle.md)
* [Documentation de référence de l’API Actors](/previous-versions/azure/dn971626(v=azure.100))
* [Exemple de code .NET](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Exemple de code Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
