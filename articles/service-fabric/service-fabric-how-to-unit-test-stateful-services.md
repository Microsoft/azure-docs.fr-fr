---
title: Développer des tests unitaires pour les services avec état
description: En savoir plus sur les tests unitaires dans Azure Service Fabric pour les services avec état et sur les considérations spéciales à prendre en compte lors du développement.
ms.topic: conceptual
ms.date: 09/04/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 287c0544daa3c44d91fd336b502c496b9b4bb266
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89011393"
---
# <a name="create-unit-tests-for-stateful-services"></a>Créer des tests unitaires pour les services avec état
Les tests unitaires sur les services avec état Service Fabric permettent d’identifier des erreurs courantes qui ne seraient pas forcément interceptées par les tests unitaires conventionnels propres à une application ou à un domaine. Lors du développement des tests unitaires pour les services avec état, vous devez tenir compte de certains points.

1. Chaque réplica exécute le code d’application, mais sous un contexte différent. Si le service utilise trois réplicas, le code de service s’exécute sur trois nœuds en parallèle sous un rôle/contexte différent.
2. L’état stocké dans le service avec état doit être cohérent entre tous les réplicas. Le gestionnaire d’état et les collections fiables fournissent par défaut cette cohérence. Toutefois, l’état en mémoire doit être géré par le code d’application.
3. Chaque réplica change de rôle à un moment donné pendant l’exécution sur le cluster. Un réplica secondaire devient un réplica principal si le nœud qui héberge le réplica principal est indisponible ou surchargé. Comme il s’agit d’un comportement naturel pour Service Fabric, les services doivent planifier de façon à s’exécuter tôt ou tard sous un autre rôle.

Cet article part du principe que vous avez lu [Tests unitaires des services avec état dans Service Fabric](service-fabric-concepts-unit-testing.md).

## <a name="the-servicefabricmocks-library"></a>La bibliothèque ServiceFabric.Mocks
À compter de la version 3.3.0, [ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) fournit une API pour la simulation à la fois de l’orchestration des réplicas et de la gestion de l’état. Elle sera utilisée dans les exemples.

[Nuget](https://www.nuget.org/packages/ServiceFabric.Mocks/)
[GitHub](https://github.com/loekd/ServiceFabric.Mocks)

*ServiceFabric.Mocks n’est pas la propriété de Microsoft et n’est pas gérée par Microsoft. Toutefois, il s’agit actuellement de la bibliothèque recommandée par Microsoft pour les tests unitaires des services avec état.*

## <a name="set-up-the-mock-orchestration-and-state"></a>Configurer l’orchestration et le gestionnaire d’état fictifs
Dans le cadre de la partie « disposition » du test, un jeu de réplicas et un gestionnaire d’état fictifs seront créés. Le jeu de réplicas sera ensuite propriétaire de la création d’une instance du service testé pour chaque réplica. Il sera également propriétaire de l’exécution des événements de cycle de vie comme `OnChangeRole` et `RunAsync`. Le gestionnaire d’état fictif garantit que les opérations effectuées sur le gestionnaire d’état sont exécutées et conservées comme le ferait le gestionnaire d’état réel.

1. Créez un délégué de fabrique de service qui instanciera le service testé. Il doit être similaire ou identique au rappel de fabrique de service que l’on trouve généralement dans `Program.cs` pour un service ou un acteur Service Fabric. Il doit respecter la signature suivante :
   ```csharp
   MyStatefulService CreateMyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica2 stateManager)
   ```
2. Créez une instance de la classe `MockReliableStateManager`. Elle simulera toutes les interactions avec le gestionnaire d’état.
3. Créez une instance de `MockStatefulServiceReplicaSet<TStatefulService>` où `TStatefulService` est le type du service testé. Cela nécessitera le délégué créé à l’étape 1 et le gestionnaire d’état instancié à l’étape 2.
4. Ajoutez des réplicas au jeu de réplicas. Spécifiez le rôle (par exemple Primary, ActiveSecondary, IdleSecondary) et l’ID du réplica.
   > Prenez note des ID de réplicas. Vous les utiliserez sans doute durant les parties « action » et « assertion » d’un test unitaire.

```csharp
//service factory to instruct how to create the service instance
var serviceFactory = (StatefulServiceContext context, IReliableStateManagerReplica2 stateManager) => new MyStatefulService(context, stateManager);
//instantiate a new mock state manager
var stateManager = new MockReliableStateManager();
//instantiate a new replica set with the service factory and state manager
var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
//add a new Primary replica with id 1
await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
//add a new ActiveSecondary replica with id 2
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
//add a second ActiveSecondary replica with id 3
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);
```

## <a name="execute-service-requests"></a>Exécuter les requêtes de service
Les requêtes de service peuvent être exécutées sur un réplica spécifique à l’aide des recherches et des propriétés utilitaires.
```csharp
const string stateName = "test";
var payload = new Payload(StatePayload);

//execute a request on the primary replica using
await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);

//execute a request against replica with id 2
await replicaSet[2].ServiceInstance.InsertAsync(stateName, payload);

//execute a request against one of the active secondary replicas
await replicaSet.FirstActiveSecondary.InsertAsync(stateName, payload);
```

## <a name="execute-a-service-move"></a>Exécuter un déplacement de service
Le jeu de réplicas fictif expose plusieurs méthodes utilitaires pour déclencher différents types de déplacements de service.
```csharp
//promote the first active secondary to primary
replicaSet.PromoteNewReplicaToPrimaryAsync();
//promote the secondary with replica id 4 to primary
replicaSet.PromoteNewReplicaToPrimaryAsync(4);

//promote the first idle secondary to an active secondary
PromoteIdleSecondaryToActiveSecondaryAsync();
//promote idle secondary with replica id 4 to active secondary
PromoteIdleSecondaryToActiveSecondaryAsync(4);

//add a new replica with randomly assigned replica id and promote it to primary
PromoteNewReplicaToPrimaryAsync()
//add a new replica with replica id 4 and promote it to primary
PromoteNewReplicaToPrimaryAsync(4)
```

## <a name="putting-it-all-together"></a>Exemple complet
Le test suivant illustre la configuration d’un jeu de réplicas à trois nœuds et la vérification du fait que les données sont disponibles à partir d’un réplica secondaire après un changement de rôle. Ce test pourrait par exemple intercepter un problème courant selon lequel des données ajoutées pendant `InsertAsync` auraient été enregistrées dans quelque chose en mémoire ou dans une collection fiable sans exécuter `CommitAsync`. Dans les deux cas, le réplica secondaire ne serait pas synchronisé avec le réplica principal. Cela provoquerait des réponses incohérentes après un déplacement de service.

```csharp
[TestMethod]
public async Task TestServiceState_InMemoryState_PromoteActiveSecondary()
{
    var stateManager = new MockReliableStateManager();
    var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
    await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);

    const string stateName = "test";
    var payload = new Payload(StatePayload);

    //insert data
    await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);
    //promote one of the secondaries to primary
    await replicaSet.PromoteActiveSecondaryToPrimaryAsync(2);
    //get data
    var payloads = (await replicaSet.Primary.ServiceInstance.GetPayloadsAsync()).ToList();

    //data should match what was inserted against the primary
    Assert.IsTrue(payloads.Count == 1);
    Assert.IsTrue(payloads[0].Content == payload.Content);

    //verify the data was saved against the reliable dictionary
    var dictionary = await StateManager.GetOrAddAsync<IReliableDictionary<string, Payload>>(MyStatefulService.StateManagerDictionaryKey);
    using(var tx = StateManager.CreateTransaction())
    {
        var payload = await dictionary.TryGetValue(stateName);
        Assert.IsTrue(payload.HasValue);
        Assert.IsTrue(payload.Value.Content == payload.Content);
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment tester les [communications de service à service](service-fabric-testability-scenarios-service-communication.md) et [simuler des échecs à l’aide d’un chaos contrôlé](service-fabric-controlled-chaos.md).
