---
title: Affichage de l’intégrité agrégée des entités Azure Service Fabric
description: Explique comment interroger, afficher et évaluer l’état d’intégrité agrégé des entités Azure Service Fabric, via des requêtes d’intégrité et des requêtes générales.
author: georgewallace
ms.topic: conceptual
ms.date: 2/28/2018
ms.author: gwallace
ms.custom: devx-track-csharp
ms.openlocfilehash: eeb2fd94e6b98bc9d89be22501406db9a8ba7773
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89013161"
---
# <a name="view-service-fabric-health-reports"></a>Affichage rapports d’intégrité de Service Fabric
Azure Service Fabric propose un [modèle d’intégrité](service-fabric-health-introduction.md) avec des entités d’intégrité sur lesquelles les composants système et les agents de surveillance peuvent signaler les conditions locales qu’ils surveillent. Le [magasin d’intégrité](service-fabric-health-introduction.md#health-store) agrège toutes les données d’intégrité pour déterminer si les entités sont saines.

Les rapports d’intégrité envoyés par les composants système alimentent automatiquement le cluster. Pour en savoir plus, consultez l’article [Utiliser les rapports d’intégrité du système pour la résolution des problèmes](service-fabric-understand-and-troubleshoot-with-system-health-reports.md).

Service Fabric offre de multiples moyens d’obtenir les données d’intégrité agrégée des entités :

* [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) ou d’autres outils de visualisation
* Requêtes d’intégrité (via PowerShell, API ou REST)
* Requêtes générales retournant une liste d’entités qui présentent l’intégrité comme l’une de leurs propriétés (via PowerShell, API ou REST).

Pour illustrer ces options, utilisons un cluster local doté de cinq nœuds et de l’[application fabric:/WordCount](https://github.com/Azure-Samples/service-fabric-wordcount/raw/master/WordCountV1.sfpkg). L’application **fabric:/WordCount** contient deux services par défaut, un service avec état de type `WordCountServiceType` et un service sans état du type `WordCountWebServiceType`. J’ai modifié le fichier `ApplicationManifest.xml` pour exiger sept réplicas cibles pour le service avec état et une partition. Comme il n’existe que cinq nœuds dans le cluster, les composants système signalent un avertissement sur la partition du service, car le nombre cible est supérieur.

```xml
<Service Name="WordCountService">
  <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
    <UniformInt64Partition PartitionCount="[WordCountService_PartitionCount]" LowKey="1" HighKey="26" />
  </StatefulService>
</Service>
```

## <a name="health-in-service-fabric-explorer"></a>Intégrité dans Service Fabric Explorer
Service Fabric Explorer procure un aperçu visuel du cluster. Dans l’image ci-dessous, vous pouvez constater que :

* L’application **fabric:/WordCount** apparaît en rouge (erreur), car un événement d’erreur a été signalé par **MyWatchdog** pour la propriété **Availability**.
* L’un de ses services, **fabric:/WordCount/WordCountService** apparaît en jaune (avertissement). Le service est configuré avec sept réplicas et le cluster a cinq nœuds : deux réplicas ne peuvent donc pas être placés. Même si elle n’est pas représentée ici, la partition de service est en jaune, conformément à un rapport système de `System.FM` indiquant que `Partition is below target replica or instance count`. La partition en jaune déclenche le service en jaune.
* Le cluster apparaît en rouge en raison de l’application rouge.

L’évaluation utilise les stratégies par défaut du manifeste de cluster et du manifeste de l’application. Il s’agit de stratégies strictes qui ne tolèrent aucun échec.

Vue du cluster avec Service Fabric Explorer :

![Vue du cluster avec Service Fabric Explorer.][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [!NOTE]
> En savoir plus sur [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).
>
>

## <a name="health-queries"></a>Requêtes d'intégrité
Service Fabric expose les requêtes d’intégrité pour chacun des [types d’entité](service-fabric-health-introduction.md#health-entities-and-hierarchy)pris en charge. Elles sont accessibles par l’intermédiaire de l’API, via les méthodes sur [FabricClient.HealthManager](/dotnet/api/system.fabric.fabricclient.healthmanager?view=azure-dotnet), des applets de commande PowerShell et de REST. Ces requêtes renvoient des informations complètes sur l’intégrité de l’entité : l’état d’intégrité agrégé, les événements d’intégrité de l’entité, les états d’intégrité enfants (le cas échéant), les évaluations de défaut d’intégrité (lorsque l’entité n’est pas intègre) et les statistiques d’intégrité enfants (au besoin).

> [!NOTE]
> Une entité d’intégrité est renvoyée si elle est intégralement renseignée dans le magasin d’intégrité. Elle doit être active (pas supprimée) et doit disposer d’un rapport système. Ses entités parentes dans la chaîne hiérarchique doivent également disposer de rapports système. Si l’une de ces conditions n’est pas satisfaite, les requêtes d’intégrité retournent [FabricException](/dotnet/api/system.fabric.fabricexception) avec [FabricErrorCode](/dotnet/api/system.fabric.fabricerrorcode) `FabricHealthEntityNotFound` indiquant la raison du non-renvoi de l’entité.
>
>

Les requêtes d’intégrité doivent transmettre l’identificateur de l’entité, qui dépend du type d’entité. Elles acceptent des paramètres facultatifs de stratégie de contrôle d’intégrité. Si aucune stratégie d’intégrité n’est indiquée, les [stratégies de contrôle d’intégrité](service-fabric-health-introduction.md#health-policies) du manifeste de cluster ou d’application sont utilisées pour l’évaluation. Si les manifestes ne contiennent pas de définition pour les stratégies de contrôle d’intégrité, ces stratégies par défaut sont utilisées pour l’évaluation. Les stratégies de contrôle d’intégrité par défaut ne tolèrent pas les échecs. Les requêtes acceptent également des filtres pour ne renvoyer le cas échéant que des enfants ou des événements partiels. Un autre filtre permet l’exclusion des statistiques enfants.

> [!NOTE]
> Les filtres de sortie sont appliqués côté serveur, ce qui réduit la taille de la réponse du message. Nous vous recommandons d’utiliser les filtres de sortie afin de limiter le volume de données renvoyées, plutôt que d’appliquer des filtres côté client.
>
>

Les données d’intégrité d’une entité incluent les éléments suivants :

* L’état agrégé d’intégrité de l’entité. Il est calculé par le magasin d’intégrité en fonction des rapports d’intégrité de l’entité, de l’état d’intégrité des enfants (le cas échéant) et des stratégies de contrôle d’intégrité. En savoir plus sur l’ [évaluation de l’intégrité de l’entité](service-fabric-health-introduction.md#health-evaluation).  
* Les événements d’intégrité sur l’entité.
* La collection des états d’intégrité de l’ensemble des enfants pour les entités pouvant en avoir. Les états d’intégrité contiennent les identificateurs d’entité et l’état d’intégrité agrégé. Pour obtenir les données d’intégrité complètes d’un enfant, sollicitez la requête d’intégrité du type de l’entité enfant et transmettez l’identificateur de l’enfant.
* Les évaluations de défaut d’intégrité qui indiquent le rapport qui a déclenché l’état de l’entité si celle-ci n’est pas saine. Les évaluations sont récursives, contenant les évaluations d’intégrité enfants qui ont déclenché l’état d’intégrité actuel. Par exemple, un agent de surveillance a signalé une erreur sur un réplica. Le contrôle d’intégrité de l’application présente une évaluation de défaut d’intégrité en raison d’un service non intègre ; celui-ci est défectueux en raison d’une partition comportant des erreurs ; celle-ci est défectueuse en raison d’un réplica incorrect ; celui-ci est défectueux en raison du rapport de défaut d’intégrité de l’agent de surveillance.
* Les statistiques d’intégrité pour tous les types enfants des entités qui ont des enfants. Par exemple, le contrôle d’intégrité du cluster indique le nombre total d’applications, de services, de partitions, de réplicas et d’entités déployées dans le cluster. Le contrôle d’intégrité du service indique le nombre de partitions et de réplicas sous le service spécifié.

## <a name="get-cluster-health"></a>Obtenir les données d’intégrité du cluster
Cette requête renvoie les données d’intégrité de l’entité du cluster et contient les états d’intégrité des applications et des nœuds (enfants du cluster). Entrée :

* [Facultatif] La stratégie d’intégrité du cluster utilisée pour évaluer les événements de nœuds et de cluster
* [Facultatif] Mappage de la stratégie de contrôle d’intégrité de l’application avec les stratégies de contrôle d’intégrité utilisées pour remplacer les stratégies du manifeste de l’application.
* [Facultatif] Filtres des événements, des nœuds et des applications qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements, nœuds et applications permettent d’évaluer l’intégrité d’une entité agrégée, quel que soit le filtre utilisé.
* [Facultatif] Filtre pour exclure les statistiques d’intégrité.
* [Facultatif] Filtre pour inclure les statistiques d’intégrité de fabric:/System dans les statistiques d’intégrité. Applicable uniquement lorsque les statistiques d’intégrité ne sont pas exclues. Par défaut, les statistiques d’intégrité comprennent uniquement les statistiques des applications utilisateur et pas celles de l’application système.

### <a name="api"></a>API
Pour obtenir les données d’intégrité du cluster, créez un élément `FabricClient` et appelez la méthode [GetClusterHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthasync) sur son élément **HealthManager**.

L’appel suivant permet de récupérer l’intégrité du cluster :

```csharp
ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();
```

Le code suivant obtient les données d’intégrité du cluster à l’aide d’une stratégie personnalisée de contrôle d’intégrité du cluster et de filtres dédiés aux nœuds et aux applications. Il spécifie que les statistiques d’intégrité incluent les statistiques fabric:/System. Il crée l’élément [ClusterHealthQueryDescription](/dotnet/api/system.fabric.description.clusterhealthquerydescription), qui contient les informations d’entrée.

```csharp
var policy = new ClusterHealthPolicy()
{
    MaxPercentUnhealthyNodes = 20
};
var nodesFilter = new NodeHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error | HealthStateFilter.Warning
};
var applicationsFilter = new ApplicationHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error
};
var healthStatisticsFilter = new ClusterHealthStatisticsFilter()
{
    ExcludeHealthStatistics = false,
    IncludeSystemApplicationHealthStatistics = true
};
var queryDescription = new ClusterHealthQueryDescription()
{
    HealthPolicy = policy,
    ApplicationsFilter = applicationsFilter,
    NodesFilter = nodesFilter,
    HealthStatisticsFilter = healthStatisticsFilter
};

ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité du cluster est [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

L’état du cluster indique cinq nœuds, l’application système et l’application fabric:/WordCount, configurés comme indiqué.

L’applet de commande suivante permet d’obtenir les données d’intégrité du cluster à l’aide des stratégies de contrôle d’intégrité par défaut. L’état d’intégrité agrégé est l’avertissement, car l’application fabric:/WordCount présente cet état. Notez que les évaluations de défaut d’intégrité fournissent des détails sur les conditions qui ont déclenché l’intégrité agrégée.

```xml
PS D:\ServiceFabric> Get-ServiceFabricClusterHealth


AggregatedHealthState   : Warning
UnhealthyEvaluations    : 
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Warning'.

                            Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                            Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                                    Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.


NodeHealthStates        : 
                          NodeName              : _Node_4
                          AggregatedHealthState : Ok

                          NodeName              : _Node_3
                          AggregatedHealthState : Ok

                          NodeName              : _Node_2
                          AggregatedHealthState : Ok

                          NodeName              : _Node_1
                          AggregatedHealthState : Ok

                          NodeName              : _Node_0
                          AggregatedHealthState : Ok

ApplicationHealthStates : 
                          ApplicationName       : fabric:/System
                          AggregatedHealthState : Ok

                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Warning

HealthEvents            : None
HealthStatistics        : 
                          Node                  : 5 Ok, 0 Warning, 0 Error
                          Replica               : 6 Ok, 0 Warning, 0 Error
                          Partition             : 1 Ok, 1 Warning, 0 Error
                          Service               : 1 Ok, 1 Warning, 0 Error
                          DeployedServicePackage : 6 Ok, 0 Warning, 0 Error
                          DeployedApplication   : 5 Ok, 0 Warning, 0 Error
                          Application           : 0 Ok, 1 Warning, 0 Error
```

L’applet de commande PowerShell suivante permet d’obtenir les données d’intégrité du cluster à l’aide d’une stratégie d’application personnalisée. Elle filtre les résultats afin d’identifier uniquement les applications et les nœuds indiquant une erreur ou un avertissement. En conséquence, aucun nœud n’est renvoyé puisqu’ils sont tous sains. Seule l’application fabric:/WordCount respecte le filtre d’applications. Comme la stratégie personnalisée exige de considérer les avertissements comme des erreurs pour l’application fabric:/WordCount, l’application présente l’état d’erreur tout comme le cluster.

```powershell
PS D:\ServiceFabric> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
$appHealthPolicy.ConsiderWarningAsError = $true
$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/WordCount"
$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error" -ExcludeHealthStatistics


AggregatedHealthState   : Error
UnhealthyEvaluations    : 
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Error'.

                            Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                            Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                    Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=true.


NodeHealthStates        : None
ApplicationHealthStates : 
                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Error

HealthEvents            : None
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité du cluster grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-cluster) ou [POST](/rest/api/servicefabric/get-the-health-of-a-cluster-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-node-health"></a>Obtenir les données d’intégrité du nœud
Cette requête renvoie les données d’intégrité d’une entité de nœud et contient les événements d’intégrité signalés sur le nœud. Entrée :

* [Obligatoire] Nom du nœud qui identifie le nœud.
* [Facultatif] Paramètres de la stratégie de contrôle d’intégrité du cluster utilisés pour l’évaluation de l’intégrité.
* [Facultatif] Filtres des événements qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements permettent d’évaluer l’intégrité agrégée de l’entité, quel que soit le filtre utilisé.

### <a name="api"></a>API
Pour obtenir les données d’intégrité du nœud via l’API, créez un élément `FabricClient` et appelez la méthode [GetNodeHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getnodehealthasync) sur son élément HealthManager.

Le code suivant permet d’obtenir les données d’intégrité du nœud dont le nom est spécifié :

```csharp
NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);
```

Le code suivant permet d’obtenir les données d’intégrité du nœud spécifié, et transmet un filtre d’événements et la stratégie personnalisée via [NodeHealthQueryDescription](/dotnet/api/system.fabric.description.nodehealthquerydescription):

```csharp
var queryDescription = new NodeHealthQueryDescription(nodeName)
{
    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
};

NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité du nœud est [Get-ServiceFabricNodeHealth](/powershell/module/servicefabric/get-servicefabricnodehealth). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .
L’applet de commande suivante permet d’obtenir les données d’intégrité du nœud à l’aide des stratégies de contrôle d’intégrité par défaut :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricNodeHealth _Node_1


NodeName              : _Node_1
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 3
                        SentAt                : 7/13/2017 4:39:23 PM
                        ReceivedAt            : 7/13/2017 4:40:47 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 4:40:47 PM, LastWarning = 1/1/0001 12:00:00 AM
```

L’applet de commande suivante permet d’obtenir les données d’intégrité de tous les nœuds du cluster :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize

NodeName AggregatedHealthState
-------- ---------------------
_Node_4                     Ok
_Node_3                     Ok
_Node_2                     Ok
_Node_1                     Ok
_Node_0                     Ok
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité du nœud grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-node) ou [POST](/rest/api/servicefabric/get-the-health-of-a-node-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-application-health"></a>Obtenir les données d’intégrité des applications
Renvoie les données d’intégrité d’une entité d’application. Elle contient les états d’intégrité de l’application déployée et des enfants du service. Entrée :

* [Obligatoire] Nom de l’application (URI) qui identifie l’application.
* [Facultatif] Stratégie de contrôle d’intégrité d’application remplaçant les stratégies du manifeste d’application.
* [Facultatif] Filtres des événements, des services et des applications déployées qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements, services et applications déployés permettent d’évaluer l’intégrité d’entité agrégée, quel que soit le filtre utilisé.
* [Facultatif] Filtre pour exclure les statistiques d’intégrité. S’il n’est pas spécifié, les statistiques d’intégrité comprennent le nombre de OK, d’avertissements et d’erreurs de tous les enfants de l’application : les services, partitions, réplicas, applications déployées et packages de service déployés.

### <a name="api"></a>API
Pour obtenir les données d’intégrité de l’application, créez un élément `FabricClient` et appelez la méthode [GetApplicationHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getapplicationhealthasync) sur son élément HealthManager.

Le code suivant permet d’obtenir les données d’intégrité de l’application spécifiée (URI) :

```csharp
ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);
```

Le code suivant permet d’obtenir les données d’intégrité de l’application spécifiée (URI), avec les filtres et les stratégies personnalisées spécifiés via le paramètre [ApplicationHealthQueryDescription](/dotnet/api/system.fabric.description.applicationhealthquerydescription).

```csharp
HealthStateFilter warningAndErrors = HealthStateFilter.Error | HealthStateFilter.Warning;
var serviceTypePolicy = new ServiceTypeHealthPolicy()
{
    MaxPercentUnhealthyPartitionsPerService = 0,
    MaxPercentUnhealthyReplicasPerPartition = 5,
    MaxPercentUnhealthyServices = 0,
};
var policy = new ApplicationHealthPolicy()
{
    ConsiderWarningAsError = false,
    DefaultServiceTypeHealthPolicy = serviceTypePolicy,
    MaxPercentUnhealthyDeployedApplications = 0,
};

var queryDescription = new ApplicationHealthQueryDescription(applicationName)
{
    HealthPolicy = policy,
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = warningAndErrors },
    ServicesFilter = new ServiceHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
    DeployedApplicationsFilter = new DeployedApplicationHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
};

ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité de l’application est [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth?view=azureservicefabricps). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

L’applet de commande suivante renvoie les données d’intégrité de l’application **fabric:/WordCount** :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth fabric:/WordCount


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Warning
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ServiceHealthStates             : 
                                  ServiceName           : fabric:/WordCount/WordCountWebService
                                  AggregatedHealthState : Ok

                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Warning

DeployedApplicationHealthStates : 
                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_4
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_3
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_0
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_2
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_1
                                  AggregatedHealthState : Ok

HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/13/2017 5:57:05 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics                : 
                                  Replica               : 6 Ok, 0 Warning, 0 Error
                                  Partition             : 1 Ok, 1 Warning, 0 Error
                                  Service               : 1 Ok, 1 Warning, 0 Error
                                  DeployedServicePackage : 6 Ok, 0 Warning, 0 Error
                                  DeployedApplication   : 5 Ok, 0 Warning, 0 Error
```

L’applet de commande PowerShell suivante transmet des stratégies personnalisées et filtre les enfants et les événements.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth -ApplicationName fabric:/WordCount -ConsiderWarningAsError $true -ServicesFilter Error -EventsFilter Error -DeployedApplicationsFilter Error -ExcludeHealthStatistics


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=true.

ServiceHealthStates             : 
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Error

DeployedApplicationHealthStates : None
HealthEvents                    : None
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité de l’application grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-an-application) ou [POST](/rest/api/servicefabric/get-the-health-of-an-application-by-using-an-application-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-service-health"></a>Obtenir l’état d’intégrité du service
Renvoie les données d’intégrité d’une entité de service. Elle contient les états d’intégrité des partitions. Entrée :

* [Obligatoire] Nom (URI) qui identifie le service.
* [Facultatif] Stratégie de contrôle d’intégrité d’application remplaçant la stratégie du manifeste d’application.
* [Facultatif] Filtres des événements et des partitions qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements et les partitions permettent d’évaluer l’intégrité d’entité agrégée, quel que soit le filtre utilisé.
* [Facultatif] Filtre pour exclure les statistiques d’intégrité. S’il n’est pas spécifié, les statistiques d’intégrité indiquent le nombre de OK, d’avertissements et d’erreurs de toutes les partitions et réplicas du service.

### <a name="api"></a>API
Pour obtenir les données d’intégrité de service via l’API, créez un élément `FabricClient` et appelez la méthode [GetServiceHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getservicehealthasync) sur son élément HeathManager.

L’exemple suivant permet d’obtenir les données d’intégrité d’un service présentant un nom (URI) spécifique :

```csharp
ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);
```

Le code suivant permet d’obtenir les données d’intégrité du service présentant le nom (URI) spécifique, en spécifiant des filtres et une stratégie personnalisée via le paramètre [ServiceHealthQueryDescription](/dotnet/api/system.fabric.description.servicehealthquerydescription):

```csharp
var queryDescription = new ServiceHealthQueryDescription(serviceName)
{
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
};

ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité du service est [Get-ServiceFabricServiceHealth](/powershell/module/servicefabric/get-servicefabricservicehealth). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

L’applet de commande suivante permet d’obtenir les données d’intégrité du service à l’aide des stratégies de contrôle d’intégrité par défaut :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricServiceHealth -ServiceName fabric:/WordCount/WordCountService


ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                        Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Warning'.

                            Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

PartitionHealthStates : 
                        PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
                        AggregatedHealthState : Warning

HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 15
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics      : 
                        Replica               : 5 Ok, 0 Warning, 0 Error
                        Partition             : 0 Ok, 1 Warning, 0 Error
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité du service grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-service) ou [POST](/rest/api/servicefabric/get-the-health-of-a-service-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-partition-health"></a>Obtenir l'intégrité de la partition
Renvoie les données d’intégrité d’une entité de partition. Elle contient les états d’intégrité des réplicas. Entrée :

* [Obligatoire] ID de partition (GUID) qui identifie la partition.
* [Facultatif] Stratégie de contrôle d’intégrité d’application remplaçant la stratégie du manifeste d’application.
* [Facultatif] Filtres des événements et des réplicas qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements et les réplicas permettent d’évaluer l’intégrité d’entité agrégée, quel que soit le filtre utilisé.
* [Facultatif] Filtre pour exclure les statistiques d’intégrité. S’il n’est pas spécifié, les statistiques d’intégrité précisent le nombre de réplicas affichant l’état OK, l’état d’avertissement et l’état d’erreur.

### <a name="api"></a>API
Pour obtenir les données d’intégrité de partition via l’API, créez un élément `FabricClient` et appelez la méthode [GetPartitionHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getpartitionhealthasync) sur son élément HealthManager. Pour spécifier des paramètres facultatifs, créez un élément [PartitionHealthQueryDescription](/dotnet/api/system.fabric.description.partitionhealthquerydescription).

```csharp
PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité de la partition est [Get-ServiceFabricPartitionHealth](/powershell/module/servicefabric/get-servicefabricpartitionhealth). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

L’applet de commande suivante permet d’obtenir les données d’intégrité de l’ensemble des partitions du service **fabric:/WordCount/WordCountService** et de filtrer les états d’intégrité des réplicas :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 72
                        SentAt                : 7/13/2017 5:57:29 PM
                        ReceivedAt            : 7/13/2017 5:57:48 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/P RD _Node_2 Up 131444422260002646
                          N/S RD _Node_4 Up 131444422293113678
                          N/S RD _Node_3 Up 131444422293113679
                          N/S RD _Node_1 Up 131444422293118720
                          N/S RD _Node_0 Up 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5.)

                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 7/13/2017 5:57:48 PM, LastError = 1/1/0001 12:00:00 AM

                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131444445174851664
                        SentAt                : 7/13/2017 6:35:17 PM
                        ReceivedAt            : 7/13/2017 6:35:18 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        Secondary replica could not be placed due to the following constraints and properties:  
                        TargetReplicaSetSize: 7
                        Placement Constraint: N/A
                        Parent Service: N/A

                        Constraint Elimination Sequence:
                        Existing Secondary Replicas eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        Existing Primary Replica eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        Nodes Eliminated By Constraints:

                        Existing Secondary Replicas -- Nodes with Partition's Existing Secondary Replicas/Instances:
                        --
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None

                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None


                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/13/2017 5:57:48 PM, LastOk = 1/1/0001 12:00:00 AM

HealthStatistics      : 
                        Replica               : 5 Ok, 0 Warning, 0 Error
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité des partitions grâce à une [requête GET](/rest/api/servicefabric/sfclient-api-getpartitionhealth) ou [POST](/rest/api/servicefabric/get-the-health-of-a-partition-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-replica-health"></a>Obtenir les données d’intégrité des réplicas
Représente l’intégrité d’un réplica de service avec état ou d’une instance de service sans état. Entrée :

* [Obligatoire] ID de partition (GUID) et ID de réplica qui identifient le réplica.
* [Facultatif] Paramètres de stratégie de contrôle d’intégrité d’application remplaçant les stratégies de manifeste d’application.
* [Facultatif] Filtres des événements qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements permettent d’évaluer l’intégrité agrégée de l’entité, quel que soit le filtre utilisé.

### <a name="api"></a>API
Pour obtenir les données d’intégrité du réplica via l’API, créez un élément `FabricClient` et appelez la méthode [GetReplicaHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getreplicahealthasync) sur son élément HealthManager. Pour spécifier des paramètres avancés, utilisez l’élément [ReplicaHealthQueryDescription](/dotnet/api/system.fabric.description.replicahealthquerydescription).

```csharp
ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité du réplica est [Get-ServiceFabricReplicaHealth](/powershell/module/servicefabric/get-servicefabricreplicahealth). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

L’applet de commande suivante permet d’obtenir les données d’intégrité du réplica principal de l’ensemble des partitions du service :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422260002646
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131444422263668344
                        SentAt                : 7/13/2017 5:57:06 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_2
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité du service grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-replica) ou [POST](/rest/api/servicefabric/get-the-health-of-a-replica-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-deployed-application-health"></a>Obtenir les données d’intégrité des applications déployées
Renvoie les données d’intégrité d’une application déployée sur une entité de nœud. Elle contient les états d’intégrité du package de services déployé. Entrée :

* [Obligatoire] Nom d’application (URI) et nom de nœud (chaîne) qui identifient l’application déployée.
* [Facultatif] Stratégie de contrôle d’intégrité d’application remplaçant les stratégies du manifeste d’application.
* [Facultatif] Filtres des événements et des packages de services déployés qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements et packages de services déployés permettent d’évaluer l’intégrité d’entité agrégée, quel que soit le filtre utilisé.
* [Facultatif] Filtre pour exclure les statistiques d’intégrité. S’il n’est pas spécifié, les statistiques d’intégrité affichent le nombre de packages de services déployés ayant l’état OK, l’état d’avertissement et l’état d’erreur.

### <a name="api"></a>API
Pour obtenir les données d’intégrité d’une application déployée sur un nœud via l’API, créez un élément `FabricClient` et appelez la méthode [GetDeployedApplicationHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync) sur son élément HealthManager. Pour spécifier des paramètres facultatifs, utilisez l’élément [DeployedApplicationHealthQueryDescription](/dotnet/api/system.fabric.description.deployedapplicationhealthquerydescription).

```csharp
DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    new DeployedApplicationHealthQueryDescription(applicationName, nodeName));
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité des applications déployées est [Get-ServiceFabricDeployedApplicationHealth](/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth?view=azureservicefabricps). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) . Pour connaître l’emplacement de déploiement d’une application, exécutez [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth?view=azureservicefabricps) et visualisez les enfants de l’application déployée.

L’applet de commande suivante permet d’obtenir les données d’intégrité de l’application **fabric:/WordCount** déployée sur **_Node_2**.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricDeployedApplicationHealth -ApplicationName fabric:/WordCount -NodeName _Node_0


ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_0
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_0
                                     AggregatedHealthState : Ok

                                     ServiceManifestName   : WordCountWebServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_0
                                     AggregatedHealthState : Ok

HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131444422261848308
                                     SentAt                : 7/13/2017 5:57:06 PM
                                     ReceivedAt            : 7/13/2017 5:57:17 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/13/2017 5:57:17 PM, LastWarning = 1/1/0001 12:00:00 AM

HealthStatistics                   : 
                                     DeployedServicePackage : 2 Ok, 0 Warning, 0 Error
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité de l’application déployée grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-deployed-application) ou [POST](/rest/api/servicefabric/get-the-health-of-a-deployed-application-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="get-deployed-service-package-health"></a>Obtenir les données d’intégrité d’un package de services déployé
Renvoie les données d’intégrité d’une entité de package de services déployé. Entrée :

* [Obligatoire] Nom d’application (URI), nom de nœud (chaîne) et nom de manifeste de service (chaîne) qui identifient le package de services déployé.
* [Facultatif] Stratégie de contrôle d’intégrité d’application remplaçant la stratégie du manifeste d’application.
* [Facultatif] Filtres des événements qui spécifient les entrées intéressantes qui doivent être retournées dans le résultat (par exemple, uniquement les erreurs ou à la fois les avertissements et les erreurs). Tous les événements permettent d’évaluer l’intégrité agrégée de l’entité, quel que soit le filtre utilisé.

### <a name="api"></a>API
Pour obtenir les données d’intégrité d’un package de services déployé par l’intermédiaire de l’API, créez un élément `FabricClient` et appelez la méthode [GetDeployedServicePackageHealthAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync) sur son élément HealthManager. Pour spécifier des paramètres facultatifs, utilisez l’élément [DeployedServicePackageHealthQueryDescription](/dotnet/api/system.fabric.description.deployedservicepackagehealthquerydescription).

```csharp
DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité du package de services déployé est [Get-ServiceFabricDeployedServicePackageHealth](/powershell/module/servicefabric/get-servicefabricdeployedservicepackagehealth). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) . Pour connaître l’emplacement de déploiement d’une application, exécutez l’applet [Get-ServiceFabricApplicationHealth](/powershell/module/servicefabric/get-servicefabricapplicationhealth?view=azureservicefabricps) et visualisez les applications déployées. Pour connaître les packages de services d’une application, consultez les enfants du package de services déployé dans la sortie de l’applet [Get-ServiceFabricDeployedApplicationHealth](/powershell/module/servicefabric/get-servicefabricdeployedapplicationhealth?view=azureservicefabricps) .

L’applet de commande suivante permet d’obtenir les données d’intégrité du package de services **WordCountServicePkg** de l’application **fabric:/WordCount** déployée sur **_Node_2**. L’entité présente des rapports **System.Hosting** indiquant la réussite de l’activation du package de services et du point d’entrée et la réussite de l’inscription du type de services.

```powershell
PS D:\ServiceFabric> Get-ServiceFabricDeployedApplication -ApplicationName fabric:/WordCount -NodeName _Node_2 | Get-ServiceFabricDeployedServicePackageHealth -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_2
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131444422267693359
                             SentAt                : 7/13/2017 5:57:06 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131444422267903345
                             SentAt                : 7/13/2017 5:57:06 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131444422272458374
                             SentAt                : 7/13/2017 5:57:07 PM
                             ReceivedAt            : 7/13/2017 5:57:18 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="rest"></a>REST
Vous pouvez obtenir les données d’intégrité du package de services déployé grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-service-package) ou [POST](/rest/api/servicefabric/get-the-health-of-a-service-package-by-using-a-health-policy) qui inclut les stratégies d’intégrité décrites dans le corps.

## <a name="health-chunk-queries"></a>Requêtes par segment d’intégrité
Les requêtes par segment d’intégrité peuvent renvoyer des enfants de cluster de plusieurs niveaux (de manière récursive) par filtre d’entrée. Il prend en charge les filtres avancés qui permettent une grande souplesse dans le choix des enfants à retourner. Les filtres peuvent spécifier des enfants par l’identificateur unique ou par d’autres identificateurs de groupe et/ou d’états d’intégrité. Par défaut, les enfants ne sont pas inclus, contrairement aux commandes d’intégrité qui intègrent toujours les enfants de premier niveau.

Les [requêtes d’intégrité](service-fabric-view-entities-aggregated-health.md#health-queries) renvoient uniquement les enfants de premier niveau de l’entité spécifiée par filtre requis. Pour obtenir les enfants des enfants, vous devez appeler des API d’intégrité supplémentaires pour chaque entité d’intérêt. De même, pour obtenir les données d’intégrité d’entités spécifiques, vous devez appeler une API d’intégrité pour chaque entité souhaitée. Le filtrage avancé des requêtes de segments vous permet de demander plusieurs éléments d’intérêt dans une requête, ce qui réduit de fait la taille et le nombre des messages.

La requête par segment d’intégrité présente l’avantage suivant : vous pouvez obtenir l’état d’intégrité d’un nombre plus important d’entités de cluster (potentiellement l’ensemble des clusters à partir de la racine requise) au cours d’un seul appel. Vous pouvez transmettre des requêtes d’intégrité complexes, comme :

* Retourner uniquement les applications ayant des erreurs, et pour ces applications inclure l’ensemble des services affichant un état d’avertissement ou d’erreur. Pour les services renvoyés, inclure l’ensemble des partitions.
* Retourner uniquement les données d’intégrité de quatre applications, désignées par leur nom.
* Renvoyer uniquement les données d’intégrité des applications présentant un type particulier.
* Renvoyer l’ensemble des entités déployées sur un nœud. L’ensemble des applications, des applications déployées sur le nœud spécifié et des packages de services déployés sur ce nœud sont renvoyés.
* Retourner l’ensemble des réplicas présentant l’état d’erreur. L’ensemble des applications, des services, des partitions et uniquement les réplicas présentant un état d’erreur sont retournés.
* Renvoyer l’ensemble des applications. Pour un service spécifié, inclure l’ensemble des partitions.

Actuellement, la requête par segment d’intégrité est exposée uniquement pour l’entité du cluster. Elle renvoie un segment d’intégrité de cluster, qui comprend :

* L’état d’intégrité agrégé du cluster.
* La liste de nœuds du segment d’intégrité, conformément aux filtres d’entrée.
* La liste d’applications du segment d’intégrité, conformément aux filtres d’entrée. Chaque segment d’intégrité d’application comporte une liste de segments contenant l’ensemble des services retenus par les filtres d’entrée et une autre liste de segments contenant l’ensemble des applications déployées retenues par les filtres. Il en va de même pour les enfants des services et des applications déployées. Ainsi, l’ensemble des entités du cluster peuvent être potentiellement renvoyées si demandées, suivant un ordre hiérarchique.

### <a name="cluster-health-chunk-query"></a>Requête par segment d’intégrité de cluster
Renvoie les données d’intégrité de l’entité du cluster et contient les segments d’intégrité hiérarchiques des enfants requis. Entrée :

* [Facultatif] La stratégie d’intégrité du cluster utilisée pour évaluer les événements de nœuds et de cluster
* [Facultatif] Mappage de la stratégie de contrôle d’intégrité de l’application avec les stratégies de contrôle d’intégrité utilisées pour remplacer les stratégies du manifeste de l’application.
* [Facultatif] Filtres pour les nœuds et applications qui spécifient les entrées d’intérêt qui doivent figurer dans les résultats. Les filtres sont spécifiques à une entité/un groupe d’entités, ou s’appliquent à l’ensemble des entités de ce niveau. La liste de filtres peut contenir un filtre général et/ou des filtres associés à des identificateurs spécifiques isolant des entités individuelles renvoyées par la requête. Si elle est vide, les enfants ne sont pas renvoyés par défaut.
  Pour en savoir plus sur les filtres, voir [NodeHealthStateFilter](/dotnet/api/system.fabric.health.nodehealthstatefilter) et [ApplicationHealthStateFilter](/dotnet/api/system.fabric.health.applicationhealthstatefilter). Les filtres d’application peuvent spécifier de manière récursive des filtres avancés pour les enfants.

Le résultat de segments inclut les enfants retenus par les filtres.

Actuellement, la requête par segment ne renvoie pas les évaluations ou les événements d’entité présentant un défaut d’intégrité. Ces informations supplémentaires peuvent être récupérées via la requête d’intégrité du cluster.

### <a name="api"></a>API
Pour obtenir un segment d’intégrité de cluster, créez un élément `FabricClient` et appelez la méthode [GetClusterHealthChunkAsync](/dotnet/api/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync) sur son élément **HealthManager**. Vous pouvez transférer l’élément [ClusterHealthQueryDescription](/dotnet/api/system.fabric.description.clusterhealthchunkquerydescription) pour décrire les stratégies d’intégrité et les filtres avancés.

Le code suivant permet d’obtenir un segment d’intégrité de cluster avec des filtres avancés.

```csharp
var queryDescription = new ClusterHealthChunkQueryDescription();
queryDescription.ApplicationFilters.Add(new ApplicationHealthStateFilter()
    {
        // Return applications only if they are in error
        HealthStateFilter = HealthStateFilter.Error
    });

// Return all replicas
var wordCountServiceReplicaFilter = new ReplicaHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };

// Return all replicas and all partitions
var wordCountServicePartitionFilter = new PartitionHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };
wordCountServicePartitionFilter.ReplicaFilters.Add(wordCountServiceReplicaFilter);

// For specific service, return all partitions and all replicas
var wordCountServiceFilter = new ServiceHealthStateFilter()
{
    ServiceNameFilter = new Uri("fabric:/WordCount/WordCountService"),
};
wordCountServiceFilter.PartitionFilters.Add(wordCountServicePartitionFilter);

// Application filter: for specific application, return no services except the ones of interest
var wordCountApplicationFilter = new ApplicationHealthStateFilter()
    {
        // Always return fabric:/WordCount application
        ApplicationNameFilter = new Uri("fabric:/WordCount"),
    };
wordCountApplicationFilter.ServiceFilters.Add(wordCountServiceFilter);

queryDescription.ApplicationFilters.Add(wordCountApplicationFilter);

var result = await fabricClient.HealthManager.GetClusterHealthChunkAsync(queryDescription);
```

### <a name="powershell"></a>PowerShell
L’applet de commande permettant d’obtenir les données d’intégrité du cluster est [Get-ServiceFabricClusterChunkHealth](/powershell/module/servicefabric/get-servicefabricclusterhealthchunk). Commencez par vous connecter au cluster à l’aide de l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) .

Le code suivant obtient des nœuds uniquement s’ils présentent un état d’erreur, sauf pour un nœud spécifique, qui doit toujours être renvoyé.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$nodeFilter1 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{HealthStateFilter=$errorFilter}
$nodeFilter2 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{NodeNameFilter="_Node_1";HealthStateFilter=$allFilter}
# Create node filter list that will be passed in the cmdlet
$nodeFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.NodeHealthStateFilter]
$nodeFilters.Add($nodeFilter1)
$nodeFilters.Add($nodeFilter2)

Get-ServiceFabricClusterHealthChunk -NodeFilters $nodeFilters


HealthState                  : Warning
NodeHealthStateChunks        : 
                               TotalCount            : 1

                               NodeName              : _Node_1
                               HealthState           : Ok

ApplicationHealthStateChunks : None
```

L’applet de commande suivante obtient le segment de cluster avec des filtres d’application.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

# All replicas
$replicaFilter = New-Object System.Fabric.Health.ReplicaHealthStateFilter -Property @{HealthStateFilter=$allFilter}

# All partitions
$partitionFilter = New-Object System.Fabric.Health.PartitionHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$partitionFilter.ReplicaFilters.Add($replicaFilter)

# For WordCountService, return all partitions and all replicas
$svcFilter1 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{ServiceNameFilter="fabric:/WordCount/WordCountService"}
$svcFilter1.PartitionFilters.Add($partitionFilter)

$svcFilter2 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{HealthStateFilter=$errorFilter}

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{ApplicationNameFilter="fabric:/WordCount"}
$appFilter.ServiceFilters.Add($svcFilter1)
$appFilter.ServiceFilters.Add($svcFilter2)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)

Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks : 
                               TotalCount            : 1

                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               ServiceHealthStateChunks : 
                                TotalCount            : 1

                                ServiceName           : fabric:/WordCount/WordCountService
                                HealthState           : Error
                                PartitionHealthStateChunks : 
                                    TotalCount            : 1

                                    PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
                                    HealthState           : Error
                                    ReplicaHealthStateChunks : 
                                        TotalCount            : 5

                                        ReplicaOrInstanceId   : 131444422293118720
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293118721
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293113678
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422293113679
                                        HealthState           : Ok

                                        ReplicaOrInstanceId   : 131444422260002646
                                        HealthState           : Error
```

L’applet de commande suivante renvoie toutes les entités déployées sur un nœud.

```xml
PS D:\ServiceFabric> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$dspFilter = New-Object System.Fabric.Health.DeployedServicePackageHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$daFilter =  New-Object System.Fabric.Health.DeployedApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter;NodeNameFilter="_Node_2"}
$daFilter.DeployedServicePackageFilters.Add($dspFilter)

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$appFilter.DeployedApplicationFilters.Add($daFilter)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)
Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks : 
                               TotalCount            : 2

                               ApplicationName       : fabric:/System
                               HealthState           : Ok
                               DeployedApplicationHealthStateChunks : 
                                TotalCount            : 1

                                NodeName              : _Node_2
                                HealthState           : Ok
                                DeployedServicePackageHealthStateChunks :
                                    TotalCount            : 1

                                    ServiceManifestName   : FAS
                                    ServicePackageActivationId : 
                                    HealthState           : Ok



                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               DeployedApplicationHealthStateChunks : 
                                TotalCount            : 1

                                NodeName              : _Node_2
                                HealthState           : Ok
                                DeployedServicePackageHealthStateChunks :
                                    TotalCount            : 1

                                    ServiceManifestName   : WordCountServicePkg
                                    ServicePackageActivationId : 
                                    HealthState           : Ok
```

### <a name="rest"></a>REST
Vous pouvez obtenir le segment d’intégrité du cluster grâce à une [requête GET](/rest/api/servicefabric/get-the-health-of-a-cluster-using-health-chunks) ou [POST](/rest/api/servicefabric/health-of-cluster) qui inclut les stratégies d’intégrité et les filtres avancés décrits dans le corps.

## <a name="general-queries"></a>Requêtes générales
Les requêtes générales renvoient la liste des entités Service Fabric d’un type spécifié. Elles sont exposées par l’intermédiaire de l’API (via les méthodes sur **FabricClient.QueryManager**), des applets de commande PowerShell et de REST. Ces requêtes agrègent les sous-requêtes de plusieurs composants. L’un d’eux est le [magasin d’intégrité](service-fabric-health-introduction.md#health-store), qui renseigne l’état d’intégrité agrégé pour chaque résultat de requête.  

> [!NOTE]
> Les requêtes générales renvoient l’état d’intégrité agrégé de l’entité et ne contiennent pas de données d’intégrité enrichies. Si une entité n’est pas saine, vous pouvez recourir à des requêtes d’intégrité afin d’obtenir l’ensemble des informations sur son intégrité, comme les événements, l’état d’intégrité des enfants et les évaluations de défaut d’intégrité.
>
>

Si les requêtes générales renvoient un état d’intégrité inconnu pour une entité, il se peut que le magasin d’intégrité ne dispose pas des données complètes sur l’entité. Il est également possible qu’une sous-requête adressée au magasin d’intégrité ait échoué (par exemple, en cas d’erreur de communication ou de limitation du magasin d’intégrité). Recourez à une requête d’intégrité sur l’entité. Si la sous-requête a rencontré des erreurs temporaires telles que des problèmes de réseau, cette requête de suivi peut aboutir. Elle peut également vous donner plus de détails du magasin d’intégrité sur la raison pour laquelle l’entité n’est pas exposée.

Les requêtes contenant le paramètre **HealthState** pour les entités sont les suivantes :

* Liste des nœuds : Retourne la liste des nœuds dans le cluster (paginée).
  * API : [FabricClient.QueryClient.GetNodeListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getnodelistasync)
  * PowerShell : Get-ServiceFabricNode
* Liste des applications : Retourne la liste des applications du cluster (paginée).
  * API : [FabricClient.QueryClient.GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync)
  * PowerShell : Get-ServiceFabricApplication
* Liste des services : Retourne la liste des services d’une application (paginée).
  * API : [FabricClient.QueryClient.GetServiceListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync)
  * PowerShell : Get-ServiceFabricService
* Liste des partitions : Retourne la liste des partitions d’un service (paginée).
  * API : [FabricClient.QueryClient.GetPartitionListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getpartitionlistasync)
  * PowerShell : Get-ServiceFabricPartition
* Liste des réplicas : Retourne la liste des réplicas d’une partition (paginée).
  * API : [FabricClient.QueryClient.GetReplicaListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getreplicalistasync)
  * PowerShell : Get-ServiceFabricReplica
* Liste des applications déployées : Retourne la liste des applications déployées sur un nœud.
  * API : [FabricClient.QueryClient.GetDeployedApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync)
  * PowerShell : Get-ServiceFabricDeployedApplication
* Liste des packages de service déployés : Retourne la liste des packages de service d’une application déployée.
  * API : [FabricClient.QueryClient.GetDeployedServicePackageListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync)
  * PowerShell : Get-ServiceFabricDeployedApplication

> [!NOTE]
> Certaines des requêtes renvoient des résultats paginés. Ces requêtes renvoient une liste dérivée de [PagedList\<T>](/dotnet/api/system.fabric.query.pagedlist-1). Si les résultats ne tiennent pas dans un message, une seule page est renvoyée. Un élément ContinuationToken effectue un suivi de l’emplacement de l’arrêt de l’énumération. Continuez à appeler la même requête et transmettez le jeton de continuation de la requête précédente pour obtenir les résultats suivants.

### <a name="examples"></a>Exemples
Le code suivant permet d’obtenir la liste des applications défectueuses du cluster :

```csharp
var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
  app => app.HealthState == HealthState.Error);
```

L’applet de commande suivante permet d’obtenir les détails de l’application fabric:/WordCount. Notez l’état d’intégrité d’avertissement.

```powershell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/WordCount

ApplicationName        : fabric:/WordCount
ApplicationTypeName    : WordCount
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Warning
ApplicationParameters  : { "WordCountWebService_InstanceCount" = "1";
                         "_WFDebugParams_" = "[{"ServiceManifestName":"WordCountWebServicePkg","CodePackageName":"Code","EntryPointType":"Main","Debug
                         ExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {74f7e5d5-71a9-47e2-a8cd-1878ec4734f1} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"WordCountServicePkg","CodeP
                         ackageName":"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {2ab462e6-e0d1-4fda-a844-972f561fe751} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }
```

L’applet de commande suivante permet d’obtenir les services présentant un état d’intégrité d’erreur :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Error"}


ServiceName            : fabric:/WordCount/WordCountService
ServiceKind            : Stateful
ServiceTypeName        : WordCountServiceType
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
HasPersistedState      : True
ServiceStatus          : Active
HealthState            : Error
```

## <a name="cluster-and-application-upgrades"></a>Mises à niveau d’applications et de clusters
Au cours d’une mise à niveau surveillée d’un cluster et d’une application, Service Fabric vérifie les données d’intégrité afin de garantir le maintien de l’état d’intégrité. Si une entité n’est pas saine au vu de l’évaluation effectuée à l’aide des stratégies de contrôle d’intégrité configurées, la mise à niveau applique des stratégies qui lui sont propres pour déterminer l’action ultérieure. La mise à niveau peut être interrompue pour permettre une intervention de l’utilisateur (par exemple la correction des conditions d’erreur ou la modification des stratégies), ou elle peut automatiquement restaurer la version correcte précédente.

Pendant la mise à niveau d’un *cluster* , vous pouvez obtenir son statut de mise à niveau. Ce statut inclut les évaluations de défaut d’intégrité, qui désignent les éléments qui ne sont pas sains dans le cluster. Si la mise à niveau est annulée en raison de problèmes d’intégrité, son statut se rappelle les derniers motifs de défaut d’intégrité. Ces informations peuvent aider les administrateurs à examiner la cause du problème après que la mise à niveau a été annulée ou arrêtée.

De la même manière, durant la mise à niveau d’une *application* , des évaluations de défaut d’intégrité sont contenues dans le statut de mise à niveau de l’application.

Le code suivant représente l’état de mise à niveau d’une application fabric:/WordCount modifiée. Un agent de surveillance a signalé une erreur sur l’un de ses réplicas. La mise à niveau est annulée en raison du non-respect des contrôles d’intégrité.

```powershell
PS C:\> Get-ServiceFabricApplicationUpgrade fabric:/WordCount

ApplicationName               : fabric:/WordCount
ApplicationTypeName           : WordCount
TargetApplicationTypeVersion  : 1.0.0.0
ApplicationParameters         : {}
StartTimestampUtc             : 4/21/2017 5:23:26 PM
FailureTimestampUtc           : 4/21/2017 5:23:37 PM
FailureReason                 : HealthCheck
UpgradeState                  : RollingBackInProgress
UpgradeDuration               : 00:00:23
CurrentUpgradeDomainDuration  : 00:00:00
CurrentUpgradeDomainProgress  : UD1

                                NodeName            : _Node_1
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_2
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_3
                                UpgradePhase        : PreUpgradeSafetyCheck
                                PendingSafetyChecks :
                                EnsurePartitionQuorum - PartitionId: 30db5be6-4e20-4698-8185-4bd7ca744020
NextUpgradeDomain             : UD2
UpgradeDomainsStatus          : { "UD1" = "Completed";
                                "UD2" = "Pending";
                                "UD3" = "Pending";
                                "UD4" = "Pending" }
UnhealthyEvaluations          :
                                Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                      Unhealthy partition: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b', AggregatedHealthState='Error'.

                                          Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                          Unhealthy replica: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b',
                                  ReplicaOrInstanceId='131031502346844058', AggregatedHealthState='Error'.

                                              Error event: SourceId='DiskWatcher', Property='Disk'.

UpgradeKind                   : Rolling
RollingUpgradeMode            : UnmonitoredAuto
ForceRestart                  : False
UpgradeReplicaSetCheckTimeout : 00:15:00
```

Découvrez plus d’informations sur la [mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md).

## <a name="use-health-evaluations-to-troubleshoot"></a>Résoudre les problèmes à l’aide des évaluations d’intégrité
Chaque fois qu’un problème est identifié dans le cluster ou dans une application, examinez l’état d’intégrité du cluster ou de l’application afin d’isoler le problème. Les évaluations de défaut d’intégrité fournissent des détails sur les raisons de l’état actuel de défaut d’intégrité. Si vous le souhaitez, vous pouvez descendre dans la hiérarchie jusqu’aux entités enfants non saines afin d’identifier l’origine du problème.

Par exemple, considérez une application comme non intègre du fait de l’existence d’un rapport d’erreurs sur l’un de ses réplicas. L’applet de commande Powershell suivante montre les évaluations non intègres :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricApplicationHealth fabric:/WordCount -EventsFilter None -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            : 
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                    Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                    Unhealthy partition: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', AggregatedHealthState='Error'.

                                        Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                        Unhealthy replica: PartitionId='af2e3e44-a8f8-45ac-9f31-4093eb897600', ReplicaOrInstanceId='131444422260002646', AggregatedHealthState='Error'.

                                            Error event: SourceId='MyWatchdog', Property='Memory'.

ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : None
```

Vous pouvez examiner le réplica pour obtenir plus d’informations :

```powershell
PS D:\ServiceFabric> Get-ServiceFabricReplicaHealth -ReplicaOrInstanceId 131444422260002646 -PartitionId af2e3e44-a8f8-45ac-9f31-4093eb897600


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422260002646
AggregatedHealthState : Error
UnhealthyEvaluations  : 
                        Error event: SourceId='MyWatchdog', Property='Memory'.

HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131444422263668344
                        SentAt                : 7/13/2017 5:57:06 PM
                        ReceivedAt            : 7/13/2017 5:57:18 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_2
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : MyWatchdog
                        Property              : Memory
                        HealthState           : Error
                        SequenceNumber        : 131444451657749403
                        SentAt                : 7/13/2017 6:46:05 PM
                        ReceivedAt            : 7/13/2017 6:46:05 PM
                        TTL                   : Infinite
                        Description           : 
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Warning->Error = 7/13/2017 6:46:05 PM, LastOk = 1/1/0001 12:00:00 AM
```

> [!NOTE]
> Les évaluations de défaut d’intégrité mettent en rapport le premier motif d’évaluation de l’entité avec l’état d’intégrité actuel. D’autres événements peuvent déclencher cet état, mais ils ne sont pas représentés dans les évaluations. Pour en savoir plus, vous devez examiner les entités d’intégrité afin de prendre connaissance de l’ensemble des rapports d’intégrité du cluster.
>
>

## <a name="next-steps"></a>Étapes suivantes
[Utiliser les rapports d’intégrité du système pour la résolution des problèmes](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Ajout de rapports d’intégrité Service Fabric personnalisés](service-fabric-report-health.md)

[Comment signaler et contrôler l’intégrité du service](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[Surveiller et diagnostiquer les services localement](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)
