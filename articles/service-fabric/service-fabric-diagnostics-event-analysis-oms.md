---
title: Analyse d’événements Azure Service Fabric avec les journaux Azure Monitor
description: Découvrez l’analyse et la visualisation d’événements à l’aide des journaux Azure Monitor pour la surveillance et le diagnostic de clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 02/21/2019
ms.author: srrengar
ms.openlocfilehash: f44426103b8f0fce275f33682edbc3b84a08344b
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92329557"
---
# <a name="event-analysis-and-visualization-with-azure-monitor-logs"></a>Analyse et visualisation d’événements avec les journaux Azure Monitor
 Les journaux Azure Monitor collectent et analysent les données de télémétrie des applications et services hébergés dans le cloud, et fournissent des outils d’analyse pour vous aider à maximiser leur disponibilité et leurs performances. Cet article explique comment exécuter des requêtes dans les journaux Azure Monitor pour obtenir des informations détaillées et résoudre les problèmes qui surviennent dans votre cluster. Les questions courantes suivantes sont traitées :

* Comment résoudre les problèmes d’intégrité ?
* Comment savoir quand un nœud tombe en panne ?
* Comment savoir si les services de mon application ont démarré ou se sont arrêtés ?

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="overview-of-the-log-analytics-workspace"></a>Vue d’ensemble de l’espace de travail Log Analytics

>[!NOTE] 
>Même si le stockage des diagnostics est activé par défaut au moment de la création du cluster, vous devez configurer l’espace de travail Log Analytics de façon à ce qu’il lise les données du stockage des diagnostics.

Les journaux Azure Monitor collectent des données à partir de ressources gérées, notamment un agent ou une table de stockage Azure, et les conservent dans un référentiel central. Les données peuvent ensuite être utilisées pour l’analyse, l’alerte et la visualisation, ou une nouvelle exportation. Les journaux Azure Monitor prennent en charge les événements, les données de performances ou toute autre donnée personnalisée. Consultez [étapes de configuration de l’extension de diagnostics pour regrouper des événements](service-fabric-diagnostics-event-aggregation-wad.md) et [étapes de création d’un espace de travail Log Analytics pour lire les événements dans le stockage](service-fabric-diagnostics-oms-setup.md) pour vous assurer que les données circulent dans les journaux Azure Monitor.

Après réception des données par les journaux Azure Monitor, Azure dispose de plusieurs *Solutions de supervision* , à savoir des solutions prépackagées ou des tableaux de bord opérationnels qui effectuent un monitoring des données entrantes suivant différents scénarios. Ces solutions incluent une solution *Service Fabric Analytics* et une solution *Conteneurs* , qui sont les deux solutions les plus appropriées pour les diagnostics et le suivi lors de l’utilisation des clusters Service Fabric. Cet article décrit comment utiliser la solution Service Fabric Analytics, qui est créée avec l’espace de travail.

## <a name="access-the-service-fabric-analytics-solution"></a>Accéder à la solution Service Fabric Analytics dans le portail

Dans le [portail Microsoft Azure](https://portal.azure.com), accédez au groupe de ressources dans lequel vous avez créé la solution Service Fabric Analytics.

Sélectionnez la ressource **ServiceFabric\<nameOfOMSWorkspace\>** .

Dans `Summary`, des vignettes de type graphe s’affichent pour chacune des solutions activées, y compris pour Service Fabric. Cliquez sur le graphique **Service Fabric** pour accéder à la solution Service Fabric Analytics.

![Solution Service Fabric](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_summary.PNG)

L’image ci-dessus montre la page d’accueil de la solution Service Fabric Analytics. Cette page d’accueil fournit une capture instantanée de ce qui se passe dans votre cluster.

![Capture d’écran montrant la page d’accueil de la solution Service Fabric Analytics.](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_solution.PNG)

 Si vous avez activé des diagnostics lors de la création du cluster, vous pouvez voir les événements pour le 

* [Événements de cluster Service Fabric](service-fabric-diagnostics-event-generation-operational.md)
* [Événements du modèle de programmation Reliable Actors](service-fabric-reliable-actors-diagnostics.md)
* [Événements du modèle de programmation Reliable Services](service-fabric-reliable-services-diagnostics.md)

>[!NOTE]
>En dehors des événements Service Fabric prêts à l’emploi, il est possible de collecter des événements système plus détaillés en [mettant à jour la configuration de l’extension de diagnostics](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations).

## <a name="view-service-fabric-events-including-actions-on-nodes"></a>Afficher les événements Service Fabric, notamment les actions sur les nœuds

Sur la page Service Fabric Analytics, cliquez sur le graphique de **Service Fabric Events** .

![Canal opérationnel de la solution Service Fabric](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_events_selection.png)

Cliquez sur **Liste** pour afficher les événements dans une liste. Vous verrez alors tous les événements système qui ont été collectés. Pour référence, il s’agit de la table **WADServiceFabricSystemEventsTable** dans le compte Stockage Azure. De même, les services fiables et les événements acteurs que vous voyez ensuite proviennent de ces tables respectives.
    
![Canal opérationnel de requêtes](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_events.png)

Vous pouvez également cliquer sur la loupe à gauche et utiliser le langage de requête Kusto pour trouver ce que vous cherchez. Par exemple, pour rechercher toutes les actions effectuées sur des nœuds du cluster, vous pouvez utiliser la requête suivante. Les ID d’événement ci-dessous se trouvent dans la [référence des événements du canal opérationnel](service-fabric-diagnostics-event-generation-operational.md).

```kusto
ServiceFabricOperationalEvent
| where EventId < 25627 and EventId > 25619 
```

Vous pouvez effectuer des requêtes sur beaucoup plus de champs, tels que les nœuds spécifiques (ordinateur) et le service système (TaskName).

## <a name="view-service-fabric-reliable-service-and-actor-events"></a>Afficher le service Service Fabric Reliable et des événements d’acteurs

Sur la page Service Fabric Analytics, cliquez sur le graphique de **Reliable Services** .

![Modèle Reliable Services de la solution Service Fabric](media/service-fabric-diagnostics-event-analysis-oms/oms_reliable_services_events_selection.png)

Cliquez sur **Liste** pour afficher les événements dans une liste. Ici, vous pouvez voir les événements des services fiables. Vous pouvez voir différents événements qui surviennent lorsque le service runasync est démarré et terminé, ce qui est généralement le cas lors de déploiements et de mises à niveau. 

![Reliable Services pour les requêtes](media/service-fabric-diagnostics-event-analysis-oms/oms_reliable_service_events.png)

Les événements d’acteurs fiables peuvent être affichés de manière similaire. Pour configurer des événements plus détaillés pour des acteurs fiables, vous devez modifier le `scheduledTransferKeywordFilter` dans le fichier config pour l’extension de diagnostic (voir ci-dessous). Vous trouverez plus d’informations à ce sujet dans la [référence des événements des acteurs fiables](service-fabric-reliable-actors-diagnostics.md#keywords).

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

Le langage de requête Kusto est puissant. Une autre requête précieuse que vous pouvez exécuter consiste à savoir quels nœuds génèrent la plupart des événements. La requête dans la capture d’écran ci-dessous montre des événements opérationnels Service Fabric agrégés avec le service et le nœud spécifiques.

![Événements de requête par nœud](media/service-fabric-diagnostics-event-analysis-oms/oms_kusto_query.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour activer la surveillance de l’infrastructure, par exemple, les compteurs de performances, rendez-vous sur [Ajout de l’agent Log Analytics](service-fabric-diagnostics-oms-agent.md). L’agent collecte des compteurs de performances et les ajoute à votre espace de travail existant.
* Pour les clusters locaux, les journaux Azure Monitor proposent une passerelle (proxy de transfert HTTP) qui peut être utilisée pour envoyer des données aux journaux Azure Monitor. Pour plus d’informations à ce sujet, consultez [Connexion d’ordinateurs aux journaux Azure Monitor sans accès Internet à l’aide de la passerelle Log Analytics](../azure-monitor/platform/gateway.md).
* Configurez l’[alerte automatisée](../azure-monitor/platform/alerts-overview.md) afin de faciliter la détection et les diagnostics.
* Familiarisez-vous avec les fonctionnalités de [requêtes et recherches dans les journaux](../azure-monitor/log-query/log-query-overview.md) proposées par les journaux Azure Monitor.
* Pour obtenir une présentation plus détaillée des journaux Azure Monitor et de ce qu’ils proposent, lisez [Présentation des journaux Azure Monitor](../azure-monitor/overview.md).
