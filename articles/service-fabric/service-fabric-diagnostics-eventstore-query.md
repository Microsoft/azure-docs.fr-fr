---
title: Interroger des événements de cluster à l’aide des API EventStore
description: Découvrez comment utiliser les API EventStore d’Azure Service Fabric pour rechercher des événements de plateforme
author: srrengar
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: 286c59938413a912c005181fca12f6698a7bc55e
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93042887"
---
# <a name="query-eventstore-apis-for-cluster-events"></a>Interroger les API EventStore pour rechercher des événements de cluster

Cet article explique comment interroger les API EventStore disponibles dans Service Fabric version 6.2 et ultérieures. Si vous souhaitez en savoir plus sur le service EventStore, consultez la [vue d’ensemble du service EventStore](service-fabric-diagnostics-eventstore.md). Pour le moment, le service EventStore peut uniquement accéder aux données des 7 derniers jours (il est basé sur la stratégie de conservation des données des diagnostics de votre cluster).

>[!NOTE]
>Les API EventStore sont en disponibilité générale (GA) depuis Service Fabric version 6.4 seulement pour les clusters Windows exécutés sur Azure.

Les API EventStore sont accessibles directement via un point de terminaison REST, ou par programmation. En fonction de la requête, plusieurs paramètres sont requis pour collecter les données appropriées. Ces paramètres incluent généralement :
* `api-version` : la version des API EventStore que vous utilisez
* `StartTimeUtc` : définit le début de la période qui vous intéresse
* `EndTimeUtc` : fin de la période

En plus de ces paramètres, des paramètres facultatifs sont également disponibles comme :
* `timeout` : remplacer le délai de 60 secondes par défaut pour effectuer l’opération de requête
* `eventstypesfilter` : vous permet de filtrer des types d’événements spécifiques
* `ExcludeAnalysisEvents` : ne pas renvoyer d’événements « Analyse ». Par défaut, les requêtes EventStore sont retournées avec les événements « analyse » si possible. Les événements Analyse sont des événements de canal opérationnel plus complets, car ils contiennent du contexte ou des informations supplémentaires par rapport à un événement Service Fabric standard et fournissent plus de détails.
* `SkipCorrelationLookup` : ne pas rechercher les événements en corrélation potentiels dans le cluster. Par défaut, EventStore va tenter de mettre en corrélation les événements au sein d’un cluster, et de lier vos événements lorsque cela est possible. 

Chaque entité d’un cluster peut être interrogée pour y rechercher des événements. Vous pouvez également interroger toutes les entités du type pour y rechercher des événements. Par exemple, vous pouvez rechercher des événements dans un nœud spécifique ou tous les nœuds de votre cluster. Actuellement, les entités dans lesquelles vous pouvez rechercher des événements sont les suivantes (nous indiquons également comment la requête doit être structurée) :
* Cluster : `/EventsStore/Cluster/Events`
* Nœuds : `/EventsStore/Nodes/Events`
* Nœud : `/EventsStore/Nodes/<NodeName>/$/Events`
* Applications : `/EventsStore/Applications/Events`
* Application : `/EventsStore/Applications/<AppName>/$/Events`
* Services : `/EventsStore/Services/Events`
* Service : `/EventsStore/Services/<ServiceName>/$/Events`
* Partitions : `/EventsStore/Partitions/Events`
* Partition : `/EventsStore/Partitions/<PartitionID>/$/Events`
* Réplicas : `/EventsStore/Partitions/<PartitionID>/$/Replicas/Events`
* Réplica : `/EventsStore/Partitions/<PartitionID>/$/Replicas/<ReplicaID>/$/Events`

>[!NOTE]
>Lorsque vous référencez un nom d’application ou de service, la requête n’a pas besoin d’inclure le préfixe « fabric:/ ». En outre, si le nom de votre application ou service contient le signe « / », utilisez plutôt « ~ » pour que la requête puisse continuer à fonctionner. Par exemple, si votre application s’affiche sous la forme « fabric:/App1/FrontendApp », les requêtes spécifiques à votre application doivent être structurées comme suit : `/EventsStore/Applications/App1~FrontendApp/$/Events`.
>En outre, les rapports d’intégrité des services du jour s’affichent sous l’application correspondante, afin que vous recherchiez les événements `DeployedServiceHealthReportCreated` dans l’entité de l’application appropriée. 

## <a name="query-the-eventstore-via-rest-api-endpoints"></a>Interroger l’EventStore via des points de terminaison API REST

Vous pouvez interroger l’EventStore directement via un point de terminaison REST, en effectuant `GET` requêtes à : `<your cluster address>/EventsStore/<entity>/Events/`.

Par exemple, pour pouvoir rechercher tous les événements de cluster entre `2018-04-03T18:00:00Z` et `2018-04-04T18:00:00Z`, formulez votre requête comme suit :

```
Method: GET 
URL: http://mycluster:19080/EventsStore/Cluster/Events?api-version=6.4&StartTimeUtc=2018-04-03T18:00:00Z&EndTimeUtc=2018-04-04T18:00:00Z
```

Cela pourrait retourner zéro événement ou la liste des événements retournés en json :

```json
Response: 200
Body:
[
  {
    "Kind": "ClusterUpgradeStart",
    "CurrentClusterVersion": "0.0.0.0:",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeType": "Rolling",
    "RollingUpgradeMode": "UnmonitoredAuto",
    "FailureAction": "Manual",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:18:59.4313064Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeDomainComplete",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeState": "RollingForward",
    "UpgradeDomains": "(0 1 2)",
    "UpgradeDomainElapsedTimeInMs": "78.5288",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:19:59.5729953Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeDomainComplete",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeState": "RollingForward",
    "UpgradeDomains": "(3 4)",
    "UpgradeDomainElapsedTimeInMs": "0",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:20:59.6271949Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeComplete",
    "TargetClusterVersion": "6.2:1.0",
    "OverallUpgradeElapsedTimeInMs": "120196.5212",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:20:59.8134457Z",
    "HasCorrelatedEvents": false
  }
]
```

Ici, nous pouvons voir qu’entre `2018-04-03T18:00:00Z` et `2018-04-04T18:00:00Z`, ce cluster avait terminé sa première mise à niveau lorsqu’il a été créé pour la première fois, de `"CurrentClusterVersion": "0.0.0.0:"` à `"TargetClusterVersion": "6.2:1.0"`, dans `"OverallUpgradeElapsedTimeInMs": "120196.5212"`.

## <a name="query-the-eventstore-programmatically"></a>Interroger l’EventStore par programmation

Vous pouvez également interroger l’EventStore par programmation, via la [bibliothèque de client Service Fabric](/dotnet/api/overview/azure/service-fabric?view=azure-dotnet#client-library).

Une fois que vous avez configuré votre client Service Fabric, vous pouvez rechercher des événements en accédant à l’EventStore de la façon suivante : `sfhttpClient.EventStore.<request>`

Voici un exemple de requête pour tous les événements du cluster entre `2018-04-03T18:00:00Z` et `2018-04-04T18:00:00Z`, via la fonction `GetClusterEventListAsync`.

```csharp
var sfhttpClient = ServiceFabricClientFactory.Create(clusterUrl, settings);

var clstrEvents = sfhttpClient.EventsStore.GetClusterEventListAsync(
    "2018-04-03T18:00:00Z",
    "2018-04-04T18:00:00Z")
    .GetAwaiter()
    .GetResult()
    .ToList();
```

Voici un autre exemple qui interroge sur l’intégrité du cluster et tous les événements de nœud en septembre 2018 et les affiche.

```csharp
  const int timeoutSecs = 60;
  var clusterUrl = new Uri(@"http://localhost:19080"); // This example is for a Local cluster
  var sfhttpClient = ServiceFabricClientFactory.Create(clusterUrl);

  var clusterHealth = sfhttpClient.Cluster.GetClusterHealthAsync().GetAwaiter().GetResult();
  Console.WriteLine("Cluster Health: {0}", clusterHealth.AggregatedHealthState.Value.ToString());

  
  Console.WriteLine("Querying for node events...");
  var nodesEvents = sfhttpClient.EventsStore.GetNodesEventListAsync(
      "2018-09-01T00:00:00Z",
      "2018-09-30T23:59:59Z",
      timeoutSecs,
      "NodeDown,NodeUp")
      .GetAwaiter()
      .GetResult()
      .ToList();
  Console.WriteLine("Result Count: {0}", nodesEvents.Count());

  foreach (var nodeEvent in nodesEvents)
  {
      Console.Write("Node event happened at {0}, Node name: {1} ", nodeEvent.TimeStamp, nodeEvent.NodeName);
      if (nodeEvent is NodeDownEvent)
      {
          var nodeDownEvent = nodeEvent as NodeDownEvent;
          Console.WriteLine("(Node is down, and it was last up at {0})", nodeDownEvent.LastNodeUpAt);
      }
      else if (nodeEvent is NodeUpEvent)
      {
          var nodeUpEvent = nodeEvent as NodeUpEvent;
          Console.WriteLine("(Node is up, and it was last down at {0})", nodeUpEvent.LastNodeDownAt);
      }
  }
```

## <a name="sample-scenarios-and-queries"></a>Exemples de scénarios et de requêtes

Voici quelques exemples expliquant comment appeler les API REST de l’EventStore pour comprendre l’état de votre cluster.

*Mises à niveau du cluster :*

Pour savoir quand votre cluster a été mis à niveau, avec succès ou pas, pour la dernière fois la semaine dernière, vous pouvez interroger les API pour connaître les mises à niveau récemment effectuées en recherchant les événements « ClusterUpgradeComplete » dans EventStore : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=ClusterUpgradeCompleted`

*Problèmes de mise à niveau du cluster :*

De la même façon, si des problèmes sont survenus lors d’une récente mise à niveau du cluster, vous pouvez rechercher tous les événements de l’entité du cluster. Vous verrez différents événements, y compris l’initiation des mises à niveau et les UD pour lesquels la mise à niveau s’est correctement effectuée. Vous verrez également les événements correspondant au moment où la restauration a démarré et les événements d’intégrité correspondants. Voici la requête à utiliser pour cela : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*Modifications de l’état du nœud :*

Pour voir les modifications de l’état de votre nœud au cours des derniers jours (quand les nœuds ont augmenté ou baissé, quand ils ont été activés ou désactivés (par la plateforme, le service de chaos ou à la suite d’une saisie utilisateur)), utilisez la requête suivante : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Nodes/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*Événements liés aux applications :*

Vous pouvez également suivre vos derniers déploiements et mises à niveau d’applications. Utilisez la requête suivante pour voir tous les événements d’applications dans votre cluster : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*Intégrité des données historiques d’une application :*

Outre les événements du cycle de vie des applications, vous avez également la possibilité de voir les données historiques concernant l’intégrité d’une application spécifique. Pour cela, vous devez spécifier le nom de l’application pour laquelle vous souhaitez collecter les données. Utilisez cette requête pour obtenir tous les événements d’intégrité d’une application : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/myApp/$/Events?api-version=6.4&starttimeutc=2018-03-24T17:01:51Z&endtimeutc=2018-03-29T17:02:51Z&EventsTypesFilter=ApplicationNewHealthReport`. Si vous souhaitez inclure des événements d’intégrité qui sont susceptibles d’être expirés (d’avoir dépassé leur durée de vie), ajoutez `,ApplicationHealthReportExpired` à la fin de la requête, pour filtrer sur deux types d’événements.

*Intégrité des données historiques de tous les services de « myApp » :*

Actuellement, les événements du rapport d’intégrité des services apparaissent comme des événements `DeployedServicePackageNewHealthReport` sous l’application correspondante. Pour voir comment vos services se sont comportés pour « App1 », utilisez la requête suivante : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/myapp/$/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=DeployedServicePackageNewHealthReport`

*Reconfiguration de la partition :*

Pour voir tous les mouvements de partition qui se sont produits dans votre cluster, recherchez l’événement `PartitionReconfigured`. Cela peut vous aider à identifier quelles charges de travail ont été exécutées sur les différents nœuds à des instants donnés, lorsque vous diagnostiquez des problèmes dans votre cluster. Voici un exemple de requête pour cela : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Partitions/Events?api-version=6.4&starttimeutc=2018-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=PartitionReconfigured`

*Service de chaos :*

Un événement indiquant quand le service de chaos a été démarré ou arrêté est exposé au niveau du cluster. Pour voir votre utilisation récente du service de chaos, utilisez la requête suivante : `https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.4&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=ChaosStarted,ChaosStopped`
