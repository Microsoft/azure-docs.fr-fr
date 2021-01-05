---
title: Superviser un cluster Service Fabric dans Azure
description: Dans ce tutoriel, vous allez découvrir comment superviser un cluster en consultant les événements Service Fabric, en interrogeant les APIs EventStore, en supervisant des compteurs de performances et en consultant des rapports d’intégrité.
author: srrengar
ms.topic: tutorial
ms.date: 07/22/2019
ms.author: srrengar
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: f9ad0f443b1647499f7085693f34f4da9ec85398
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92331989"
---
# <a name="tutorial-monitor-a-service-fabric-cluster-in-azure"></a>Tutoriel : Superviser un cluster Service Fabric dans Azure

Le monitoring et les diagnostics sont essentiels au développement, au test et au déploiement de charges de travail dans tout environnement cloud. Ce tutoriel est le deuxième d’une série. Il vous montre comment superviser et diagnostiquer un cluster Service Fabric à l’aide d’événements, de compteurs de performances et de rapports d’intégrité.   Pour plus d’informations, consultez la vue d’ensemble sur la [supervision des clusters](service-fabric-diagnostics-overview.md#platform-cluster-monitoring) et la [supervision de l’infrastructure](service-fabric-diagnostics-overview.md#infrastructure-performance-monitoring).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Afficher les événements Service Fabric
> * Interroger les API EventStore pour rechercher des événements de cluster
> * Superviser l’infrastructure/recueillir des compteurs de performances
> * Afficher des rapports d’intégrité de cluster

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé sur Azure à l’aide d’un modèle
> * Superviser un cluster
> * [Mettre à l’échelle un cluster](service-fabric-tutorial-scale-cluster.md)
> * [Mettre à niveau le runtime d’un cluster](service-fabric-tutorial-upgrade-cluster.md)
> * [Supprimer un cluster](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Installez [Azure PowerShell](/powershell/azure/install-az-ps) ou [Azure CLI](/cli/azure/install-azure-cli).
* Créer un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé 
* Configurer la [collecte des diagnostics](service-fabric-tutorial-create-vnet-and-windows-cluster.md#configurediagnostics_anchor) pour le cluster
* Activer le [service EventStore](service-fabric-tutorial-create-vnet-and-windows-cluster.md#configureeventstore_anchor) dans le cluster
* Configurer les [journaux Azure Monitor et l’agent Log Analytics](service-fabric-tutorial-create-vnet-and-windows-cluster.md#configureloganalytics_anchor) pour le cluster

## <a name="view-service-fabric-events-using-azure-monitor-logs"></a>Afficher les événements Service Fabric à l’aide des journaux Azure Monitor

Les journaux Azure Monitor collectent et analysent les données de télémétrie des applications et services hébergés dans le cloud, et fournissent des outils d’analyse pour vous aider à maximiser leur disponibilité et leurs performances. Vous pouvez exécuter des requêtes dans les journaux Azure Monitor pour obtenir des informations détaillées et résoudre les problèmes qui se produisent dans votre cluster.

Pour accéder à la solution Service Fabric Analytics, accédez au [portail Azure](https://portal.azure.com) et sélectionnez le groupe de ressources dans lequel vous avez créé la solution Service Fabric Analytics.

Sélectionnez la ressource **ServiceFabric(mysfomsworkspace)** .

Dans **Vue d’ensemble** , des vignettes de type graphe s’affichent pour chacune des solutions activées, y compris pour Service Fabric. Cliquez sur le graphique **Service Fabric** pour accéder à la solution Service Fabric Analytics.

![Capture d’écran montrant le graphe Service Fabric.](media/service-fabric-tutorial-monitor-cluster/oms-service-fabric-summary.png)

L’image ci-dessus montre la page d’accueil de la solution Service Fabric Analytics. Cette page d’accueil fournit une capture instantanée de ce qui se passe dans votre cluster.

![Solution Service Fabric](media/service-fabric-tutorial-monitor-cluster/oms-service-fabric-solution.png)

 Si vous avez activé des diagnostics lors de la création du cluster, vous pouvez voir les événements pour le 

* [Événements de cluster Service Fabric](service-fabric-diagnostics-event-generation-operational.md)
* [Événements du modèle de programmation Reliable Actors](service-fabric-reliable-actors-diagnostics.md)
* [Événements du modèle de programmation Reliable Services](service-fabric-reliable-services-diagnostics.md)

>[!NOTE]
>En dehors des événements Service Fabric prêts à l’emploi, il est possible de collecter des événements système plus détaillés en [mettant à jour la configuration de l’extension de diagnostics](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations).

### <a name="view-service-fabric-events-including-actions-on-nodes"></a>Afficher les événements Service Fabric, notamment les actions sur les nœuds

Dans la page Service Fabric Analytics, cliquez sur le graphe des **Événements de cluster** .  Les journaux pour tous les événements système qui ont été collectés s’affichent. Pour référence, il s’agit de la table **WADServiceFabricSystemEventsTable** dans le compte Stockage Azure. De même, les services fiables et les événements acteurs que vous voyez ensuite proviennent de ces tables respectives.
    
![Canal opérationnel de requêtes](media/service-fabric-tutorial-monitor-cluster/oms-service-fabric-events.png)

La requête utilise le langage de requête Kusto, que vous pouvez modifier pour affiner ce que vous cherchez. Par exemple, pour rechercher toutes les actions effectuées sur des nœuds du cluster, vous pouvez utiliser la requête suivante. Les ID d’événement ci-dessous se trouvent dans la [référence des événements du canal opérationnel](service-fabric-diagnostics-event-generation-operational.md).

```kusto
ServiceFabricOperationalEvent
| where EventId < 25627 and EventId > 25619 
```

Le langage de requête Kusto est puissant. Voici quelques autres requêtes utiles.

Créer une table de recherche *ServiceFabricEvent* en tant que fonction définie par l’utilisateur en enregistrant la requête en tant que fonction avec l’alias ServiceFabricEvent :

```kusto
let ServiceFabricEvent = datatable(EventId: int, EventName: string)
[
    ...
    18603, 'NodeUpOperational',
    18604, 'NodeDownOperational',
    ...
];
ServiceFabricEvent
```

Réexécuter les événements opérationnels enregistrés au cours de la dernière heure :
```kusto
ServiceFabricOperationalEvent
| where TimeGenerated > ago(1h)
| join kind=leftouter ServiceFabricEvent on EventId
| project EventId, EventName, TaskName, Computer, ApplicationName, EventMessage, TimeGenerated
| sort by TimeGenerated
```

Réexécuter les événements opérationnels avec EventId == 18604 et EventName == 'NodeDownOperational' :
```kusto
ServiceFabricOperationalEvent
| where EventId == 18604
| project EventId, EventName = 'NodeDownOperational', TaskName, Computer, EventMessage, TimeGenerated
| sort by TimeGenerated 
```

Réexécuter les événements opérationnels avec EventId == 18604 et EventName == 'NodeUpOperational' :
```kusto
ServiceFabricOperationalEvent
| where EventId == 18603
| project EventId, EventName = 'NodeUpOperational', TaskName, Computer, EventMessage, TimeGenerated
| sort by TimeGenerated 
``` 

Retourner les rapports d’intégrité avec HealthState == 3 (Erreur) et extraire les propriétés supplémentaires à partir du champ EventMessage :

```kusto
ServiceFabricOperationalEvent
| join kind=leftouter ServiceFabricEvent on EventId
| extend HealthStateId = extract(@"HealthState=(\S+) ", 1, EventMessage, typeof(int))
| where TaskName == 'HM' and HealthStateId == 3
| extend SourceId = extract(@"SourceId=(\S+) ", 1, EventMessage, typeof(string)),
         Property = extract(@"Property=(\S+) ", 1, EventMessage, typeof(string)),
         HealthState = case(HealthStateId == 0, 'Invalid', HealthStateId == 1, 'Ok', HealthStateId == 2, 'Warning', HealthStateId == 3, 'Error', 'Unknown'),
         TTL = extract(@"TTL=(\S+) ", 1, EventMessage, typeof(string)),
         SequenceNumber = extract(@"SequenceNumber=(\S+) ", 1, EventMessage, typeof(string)),
         Description = extract(@"Description='([\S\s, ^']+)' ", 1, EventMessage, typeof(string)),
         RemoveWhenExpired = extract(@"RemoveWhenExpired=(\S+) ", 1, EventMessage, typeof(bool)),
         SourceUTCTimestamp = extract(@"SourceUTCTimestamp=(\S+)", 1, EventMessage, typeof(datetime)),
         ApplicationName = extract(@"ApplicationName=(\S+) ", 1, EventMessage, typeof(string)),
         ServiceManifest = extract(@"ServiceManifest=(\S+) ", 1, EventMessage, typeof(string)),
         InstanceId = extract(@"InstanceId=(\S+) ", 1, EventMessage, typeof(string)),
         ServicePackageActivationId = extract(@"ServicePackageActivationId=(\S+) ", 1, EventMessage, typeof(string)),
         NodeName = extract(@"NodeName=(\S+) ", 1, EventMessage, typeof(string)),
         Partition = extract(@"Partition=(\S+) ", 1, EventMessage, typeof(string)),
         StatelessInstance = extract(@"StatelessInstance=(\S+) ", 1, EventMessage, typeof(string)),
         StatefulReplica = extract(@"StatefulReplica=(\S+) ", 1, EventMessage, typeof(string))
```

Retourner un graphique de temps des événements avec EventId != 17523 :

```kusto
ServiceFabricOperationalEvent
| join kind=leftouter ServiceFabricEvent on EventId
| where EventId != 17523
| summarize Count = count() by Timestamp = bin(TimeGenerated, 1h), strcat(tostring(EventId), " - ", case(EventName != "", EventName, "Unknown"))
| render timechart 
```

Obtenir les événements opérationnels Service Fabric agrégés avec le service et le nœud spécifiques :

```kusto
ServiceFabricOperationalEvent
| where ApplicationName  != "" and ServiceName != ""
| summarize AggregatedValue = count() by ApplicationName, ServiceName, Computer 
```

Afficher le nombre d’événements Service Fabric par ID d’événement / EventName à l’aide d’une requête inter-ressources :

```kusto
app('PlunkoServiceFabricCluster').traces
| where customDimensions.ProviderName == 'Microsoft-ServiceFabric'
| extend EventId = toint(customDimensions.EventId), TaskName = tostring(customDimensions.TaskName)
| where EventId != 17523
| join kind=leftouter ServiceFabricEvent on EventId
| extend EventName = case(EventName != '', EventName, 'Undocumented')
| summarize ["Event Count"]= count() by bin(timestamp, 30m), EventName = strcat(tostring(EventId), " - ", EventName)
| render timechart
```

### <a name="view-service-fabric-application-events"></a>Afficher les événements d’application Service Fabric

Vous pouvez afficher les événements pour les services fiables et les applications d’acteurs fiables déployés sur le cluster.  Dans la page Service Fabric Analytics, cliquez sur le graphe de **Événements d’application** .

Exécutez la requête suivante pour afficher les événements de vos applications de services fiables :
```kusto
ServiceFabricReliableServiceEvent
| sort by TimeGenerated desc
```

Vous pouvez voir différents événements qui surviennent lorsque le service `runasync` est démarré et terminé, ce qui est généralement le cas lors de déploiements et de mises à niveau.

![Modèle Reliable Services de la solution Service Fabric](media/service-fabric-tutorial-monitor-cluster/oms-reliable-services-events-selection.png)

Vous pouvez également trouver les événements pour le service fiable avec ServiceName == "fabric:/Watchdog/WatchdogService" :

```kusto
ServiceFabricReliableServiceEvent
| where ServiceName == "fabric:/Watchdog/WatchdogService"
| project TimeGenerated, EventMessage
| order by TimeGenerated desc  
```
 
Les événements d’acteurs fiables peuvent être affichés de manière similaire :

```kusto
ServiceFabricReliableActorEvent
| sort by TimeGenerated desc
```
Pour configurer des événements plus détaillés pour des acteurs fiables, vous pouvez modifier le `scheduledTransferKeywordFilter` dans le fichier config pour l’extension de diagnostic dans le modèle de cluster. Vous trouverez plus d’informations à ce sujet dans la [référence des événements des acteurs fiables](service-fabric-reliable-actors-diagnostics.md#keywords).

```json
"EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
```

## <a name="view-performance-counters-with-azure-monitor-logs"></a>Afficher les compteurs de performances avec les journaux Azure Monitor
Pour afficher les compteurs de performances, ouvrez le [portail Azure](https://portal.azure.com) et accédez au groupe de ressources dans lequel vous avez créé la solution Service Fabric Analytics. 

Sélectionnez la ressource **ServiceFabric(mysfomsworkspace)** , puis **espace de travail Log Analytics** , puis **Paramètres avancés** .

Cliquez sur **Données** , puis sur **Compteurs de performances Windows** . Une liste des compteurs par défaut que vous pouvez choisir d’activer s’affiche, et vous pouvez également définir l’intervalle de collecte. En outre, vous pouvez ajouter [d’autres compteurs de performances](service-fabric-diagnostics-event-generation-perf.md) à collecter. Le format approprié est référencé dans cet [article](/windows/desktop/PerfCtrs/specifying-a-counter-path). Cliquez sur **Enregistrer** , puis sur **OK** .

Fermez le panneau Paramètres avancés et sélectionnez **Récapitulatif de l’espace de travail** sous le titre **Général** . Pour chacune des solutions activées il existe une vignette graphique, y compris pour Service Fabric. Cliquez sur le graphique **Service Fabric** pour accéder à la solution Service Fabric Analytics.

Il existe des vignettes graphiques pour le canal opérationnel et les événements de services fiables. La représentation graphique du flux de données correspondant aux compteurs que vous avez sélectionnés s’affiche dans **Métriques de nœud** . 

Sélectionnez un graphe de **métrique de conteneur** pour afficher des détails supplémentaires. Vous pouvez également interroger les données du compteur de performances de la même façon que les événements de cluster et appliquer des filtres sur les nœuds, le nom du compteur de performances et les valeurs à l’aide du langage de requête Kusto.

## <a name="query-the-eventstore-service"></a>Interroger le service EventStore
Le [service EventStore](service-fabric-diagnostics-eventstore.md) offre un moyen de comprendre l’état de votre cluster ou de vos charges de travail à un moment donné dans le temps. EventStore est un service Service Fabric avec état qui conserve les événements du cluster. Les événements sont exposés par le biais de [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md), de REST et des API. EventStore interroge le cluster directement pour obtenir des données de diagnostics sur une entité dans votre cluster. Pour voir une liste complète des événements disponibles dans l’EventStore, consultez [Événements Service Fabric](service-fabric-diagnostics-event-generation-operational.md).

Vous pouvez interroger les API EventStore par programmation à l’aide de la [bibliothèque de client Service Fabric](/dotnet/api/overview/azure/service-fabric?view=azure-dotnet#client-library).

Voici un exemple de requête pour tous les événements de cluster entre 2018-04-03T18:00:00Z et 2018-04-04T18:00:00Z, par le biais de la fonction GetClusterEventListAsync.

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


## <a name="monitor-cluster-health"></a>Superviser l’intégrité des clusters
Service Fabric propose un [modèle d’intégrité](service-fabric-health-introduction.md) avec des entités d’intégrité sur lesquelles les composants système et les agents de supervision peuvent signaler les conditions locales qu’ils supervisent. Le [magasin d’intégrité](service-fabric-health-introduction.md#health-store) agrège toutes les données d’intégrité pour déterminer si les entités sont saines.

Les rapports d’intégrité envoyés par les composants système alimentent automatiquement le cluster. Pour en savoir plus, consultez l’article [Utiliser les rapports d’intégrité du système pour la résolution des problèmes](service-fabric-understand-and-troubleshoot-with-system-health-reports.md).

Service Fabric expose les requêtes d’intégrité pour chacun des [types d’entité](service-fabric-health-introduction.md#health-entities-and-hierarchy)pris en charge. Elles sont accessibles par l’intermédiaire de l’API, via les méthodes sur [FabricClient.HealthManager](/dotnet/api/system.fabric.fabricclient.healthmanager?view=azure-dotnet), des applets de commande PowerShell et de REST. Ces requêtes renvoient des informations complètes sur l’intégrité de l’entité : l’état d’intégrité agrégé, les événements d’intégrité de l’entité, les états d’intégrité enfants (le cas échéant), les évaluations de défaut d’intégrité (lorsque l’entité n’est pas intègre) et les statistiques d’intégrité enfants (au besoin).

### <a name="get-cluster-health"></a>Obtenir les données d’intégrité du cluster
L’applet de commande [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth) retourne les données d’intégrité de l’entité du cluster et contient les états d’intégrité des applications et des nœuds (enfants du cluster).  Commencez par vous connecter au cluster à l’aide de l’[applet de commande Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps).

L’état du cluster indique 11 nœuds, l’application système et l’application fabric:/Voting, configurés comme indiqué.

L’exemple suivant obtient les données d’intégrité du cluster à l’aide des stratégies de contrôle d’intégrité par défaut. Les 11 nœuds sont intègres, mais l’état d’intégrité agrégé du cluster est Erreur, car l’état de l’application fabric:/Voting est Erreur. Notez que les évaluations de défaut d’intégrité fournissent des détails sur les conditions qui ont déclenché l’intégrité agrégée.

```powershell
Get-ServiceFabricClusterHealth

AggregatedHealthState   : Error
UnhealthyEvaluations    : 
                          100% (1/1) applications are unhealthy. The evaluation tolerates 0% unhealthy applications.
                          
                          Application 'fabric:/Voting' is in Error.
                          
                            33% (1/3) deployed applications are unhealthy. The evaluation tolerates 0% unhealthy deployed applications.
                          
                            Deployed application on node '_nt2vm_3' is in Error.
                          
                                50% (1/2) deployed service packages are unhealthy.
                          
                                Service package for manifest 'VotingWebPkg' and service package activation ID '8723eb73-9b83-406b-9de3-172142ba15f3' is in Error.
                          
                                    'System.Hosting' reported Error for property 'CodePackageActivation:Code:SetupEntryPoint:131959376195593305'.
                                    There was an error during CodePackage activation.The service host terminated with exit code:1
                          
                          
NodeHealthStates        : 
                          NodeName              : _nt2vm_3
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt1vm_4
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt2vm_2
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt1vm_3
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt2vm_1
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt1vm_2
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt2vm_0
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt1vm_1
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt1vm_0
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt3vm_0
                          AggregatedHealthState : Ok
                          
                          NodeName              : _nt2vm_4
                          AggregatedHealthState : Ok
                          
ApplicationHealthStates : 
                          ApplicationName       : fabric:/System
                          AggregatedHealthState : Ok
                          
                          ApplicationName       : fabric:/Voting
                          AggregatedHealthState : Error
                          
HealthEvents            : None
HealthStatistics        : 
                          Node                  : 11 Ok, 0 Warning, 0 Error
                          Replica               : 4 Ok, 0 Warning, 0 Error
                          Partition             : 2 Ok, 0 Warning, 0 Error
                          Service               : 2 Ok, 0 Warning, 0 Error
                          DeployedServicePackage : 3 Ok, 1 Warning, 1 Error
                          DeployedApplication   : 1 Ok, 1 Warning, 1 Error
                          Application           : 0 Ok, 0 Warning, 1 Error
```

L’exemple PowerShell suivant obtient les données d’intégrité du cluster à l’aide d’une stratégie d’application personnalisée. Elle filtre les résultats afin d’identifier uniquement les applications et les nœuds indiquant une erreur ou un avertissement. Dans cet exemple, aucun nœud n’est retourné puisqu’ils sont tous sains. Seule l’application fabric:/Voting respecte le filtre d’applications. Comme la stratégie personnalisée exige de considérer les avertissements comme des erreurs pour l’application fabric:/Voting, l’application présente l’état d’erreur tout comme le cluster.

```powershell
$appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
$appHealthPolicy.ConsiderWarningAsError = $true
$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/Voting"
$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error" -ExcludeHealthStatistics

AggregatedHealthState   : Error
UnhealthyEvaluations    : 
                          100% (1/1) applications are unhealthy. The evaluation tolerates 0% unhealthy applications.
                          
                          Application 'fabric:/Voting' is in Error.
                          
                            100% (5/5) deployed applications are unhealthy. The evaluation tolerates 0% unhealthy deployed applications.
                          
                            Deployed application on node '_nt2vm_3' is in Error.
                          
                                50% (1/2) deployed service packages are unhealthy.
                          
                                Service package for manifest 'VotingWebPkg' and service package activation ID '8723eb73-9b83-406b-9de3-172142ba15f3' is in Error.
                          
                                    'System.Hosting' reported Error for property 'CodePackageActivation:Code:SetupEntryPoint:131959376195593305'.
                                    There was an error during CodePackage activation.The service host terminated with exit code:1
                          
                            Deployed application on node '_nt2vm_2' is in Error.
                          
                                50% (1/2) deployed service packages are unhealthy.
                          
                                Service package for manifest 'VotingWebPkg' and service package activation ID '2466f2f9-d5fd-410c-a6a4-5b1e00630cca' is in Error.
                          
                                    'System.Hosting' reported Error for property 'CodePackageActivation:Code:SetupEntryPoint:131959376486201388'.
                                    There was an error during CodePackage activation.The service host terminated with exit code:1
                          
                            Deployed application on node '_nt2vm_4' is in Error.
                          
                                100% (1/1) deployed service packages are unhealthy.
                          
                                Service package for manifest 'VotingWebPkg' and service package activation ID '5faa5201-eede-400a-865f-07f7f886aa32' is in Error.
                          
                                    'System.Hosting' reported Warning for property 'CodePackageActivation:Code:SetupEntryPoint:131959376207396204'. The evaluation treats 
                          Warning as Error.
                                    There was an error during CodePackage activation.The service host terminated with exit code:1
                          
                            Deployed application on node '_nt2vm_0' is in Error.
                          
                                100% (1/1) deployed service packages are unhealthy.
                          
                                Service package for manifest 'VotingWebPkg' and service package activation ID '204f1783-f774-4f3a-b371-d9983afaf059' is in Error.
                          
                                    'System.Hosting' reported Error for property 'CodePackageActivation:Code:SetupEntryPoint:131959375885791093'.
                                    There was an error during CodePackage activation.The service host terminated with exit code:1
                          
                            Deployed application on node '_nt3vm_0' is in Error.
                          
                                50% (1/2) deployed service packages are unhealthy.
                          
                                Service package for manifest 'VotingWebPkg' and service package activation ID '2533ae95-2d2a-4f8b-beef-41e13e4c0081' is in Error.
                          
                                    'System.Hosting' reported Error for property 'CodePackageActivation:Code:SetupEntryPoint:131959376108346272'.
                                    There was an error during CodePackage activation.The service host terminated with exit code:1                         
                          
NodeHealthStates        : None
ApplicationHealthStates : 
                          ApplicationName       : fabric:/Voting
                          AggregatedHealthState : Error
                          
HealthEvents            : None
```

### <a name="get-node-health"></a>Obtenir les données d’intégrité du nœud
L’applet de commande [Get-ServiceFabricNodeHealth](/powershell/module/servicefabric/get-servicefabricnodehealth) retourne les données d’intégrité d’une entité de nœud et contient les événements d’intégrité signalés sur le nœud. Commencez par vous connecter au cluster à l’aide de l’[applet de commande Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps). L’exemple suivant obtient l’intégrité d’un nœud spécifique à l’aide des stratégies de contrôle d’intégrité par défaut :

```powershell
Get-ServiceFabricNodeHealth _nt1vm_3
```

L’exemple suivant obtient les données d’intégrité de tous les nœuds du cluster :
```powershell
Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize
```

### <a name="get-system-service-health"></a>Obtenir l’intégrité du service système 

Obtenir l’intégrité agrégée des services système :

```powershell
Get-ServiceFabricService -ApplicationName fabric:/System | Get-ServiceFabricServiceHealth | select ServiceName, AggregatedHealthState | ft -AutoSize
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Afficher les événements Service Fabric
> * Interroger les API EventStore pour rechercher des événements de cluster
> * Superviser l’infrastructure/recueillir des compteurs de performances
> * Afficher des rapports d’intégrité de cluster

Maintenant, passez au tutoriel suivant pour savoir comment mettre à l’échelle un cluster.
> [!div class="nextstepaction"]
> [Mise à l'échelle d'un cluster](service-fabric-tutorial-scale-cluster.md)

[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[template]: https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
