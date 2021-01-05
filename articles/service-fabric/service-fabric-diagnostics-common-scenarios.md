---
title: Diagnostiquer des scénarios courants avec Azure Service Fabric
description: Découvrez comment résoudre les problèmes liés aux scénarios de supervision et de diagnostic courants au sein des applications Azure Service Fabric.
ms.topic: article
ms.date: 02/25/2019
ms.openlocfilehash: f92bc02082d8bcd9d917f05c93d3da413f772cd0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257731"
---
# <a name="diagnose-common-scenarios-with-service-fabric"></a>Diagnostiquer des scénarios courants avec Service Fabric

Cet article décrit les scénarios courants que les utilisateurs ont rencontrés dans le domaine du monitoring et des diagnostics avec Service Fabric. Les scénarios présentés couvrent les 3 couches de Service Fabric : Application, Cluster et Infrastructure. Chaque solution utilise Application Insights et les journaux Azure Monitor, ainsi que des outils de monitoring Azure, pour chaque scénario. Les étapes de chaque solution offrent aux utilisateurs une introduction à l’utilisation d’Application Insights et de journaux Azure Monitor dans le contexte de Service Fabric.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites-and-recommendations"></a>Prérequis et recommandations

Les solutions de cet article utilisent les outils suivants. Nous vous recommandons de les installer et de les configurer :

* [Application Insights avec Service Fabric](service-fabric-tutorial-monitoring-aspnet.md)
* [Activer Diagnostics Azure sur votre cluster](service-fabric-diagnostics-event-aggregation-wad.md)
* [Configurer un espace de travail Log Analytics](service-fabric-diagnostics-oms-setup.md)
* [Agent Log Analytics pour suivre les compteurs de performances](service-fabric-diagnostics-oms-agent.md)

## <a name="how-can-i-see-unhandled-exceptions-in-my-application"></a>Comment voir les exceptions non gérées dans mon application ?

1. Accédez à votre ressource Application Insights avec laquelle est configurée votre application.
2. Cliquez sur *Rechercher* en haut à gauche. Cliquez ensuite sur Filtrer dans le panneau suivant.

    ![Vue d’ensemble d’AI](media/service-fabric-diagnostics-common-scenarios/ai-search-filter.png)

3. Vous voyez un grand nombre de types d’événements (traces, demandes, événements personnalisés). Choisissez « Exception » comme filtre.

    ![Liste de filtres Application Insights](media/service-fabric-diagnostics-common-scenarios/ai-filter-list.png)

    En cliquant sur une exception dans la liste, vous pouvez voir plus de détails, notamment le contexte du service si vous utilisez le kit SDK de Service Fabric Application Insights.

    ![Exception Application Insights](media/service-fabric-diagnostics-common-scenarios/ai-exception.png)

## <a name="how-do-i-view-which-http-calls-are-used-in-my-services"></a>Comment voir les appels HTTP qui ont été utilisés dans mes services ?

1. Dans la même ressource Application Insights, vous pouvez filtrer sur « demandes » au lieu des exceptions et voir ainsi toutes les demandes effectuées.
2. Si vous utilisez le kit SDK Service Fabric Application Insights, vous pouvez voir une représentation visuelle de vos services connectés les uns aux autres et le nombre de demandes ayant réussi et échoué. Sur la gauche, cliquez sur « Cartographie d’application ».

    ![Panneau de cartographie d’application IA](media/service-fabric-diagnostics-common-scenarios/app-map-blade.png)![Cartographie d’application IA](media/service-fabric-diagnostics-common-scenarios/app-map-new.png)

    Pour plus d’informations sur la cartographie d’application, consultez la [documentation sur la cartographie d’application](../azure-monitor/app/app-map.md).

## <a name="how-do-i-create-an-alert-when-a-node-goes-down"></a>Comment créer une alerte quand un nœud est en baisse ?

1. Les événements de nœud sont suivis par votre cluster Service Fabric. Accédez à la ressource de la solution Service Fabric Analytics nommée **ServiceFabric(NameofResourceGroup)** .
2. Cliquez sur le graphe en bas du panneau intitulé « Résumé ».

    ![Solution de journaux Azure Monitor](media/service-fabric-diagnostics-common-scenarios/oms-solution-azure-portal.png)

3. Ici, vous avez un grand nombre de graphes et de vignettes affichant diverses métriques. Cliquez sur l’un des graphes, ce qui vous dirigera vers la recherche dans les journaux. Ici, vous pouvez créer des requêtes sur des événements de cluster ou des compteurs de performances.
4. Entrez la requête suivante. Ces ID d’événement se trouvent dans la [référence des événements de nœud](service-fabric-diagnostics-event-generation-operational.md#application-events)

    ```kusto
    ServiceFabricOperationalEvent
    | where EventID >= 25622 and EventID <= 25626
    ```

5. Cliquez sur « Nouvelle règle d’alerte » en haut. À partir de maintenant, chaque fois qu’un événement arrive en fonction de cette requête, vous recevez une alerte dans le mode de communication de votre choix.

    ![Azure Monitor journalise une nouvelle alerte](media/service-fabric-diagnostics-common-scenarios/oms-create-alert.png)

## <a name="how-can-i-be-alerted-of-application-upgrade-rollbacks"></a>Comment être averti de restaurations de mise à niveau d’application ?

1. Dans la même fenêtre de recherche dans les journaux qu’avant, entrez la requête suivante pour les restaurations de mise à niveau. Ces ID d’événements se trouvent sous la [référence des événements d’application](service-fabric-diagnostics-event-generation-operational.md#application-events)

    ```kusto
    ServiceFabricOperationalEvent
    | where EventID == 29623 or EventID == 29624
    ```

2. Cliquez sur « Nouvelle règle d’alerte » en haut. À partir de maintenant, chaque fois qu’un événement arrive en fonction de cette requête, vous recevez une alerte.

## <a name="how-do-i-see-container-metrics"></a>Comment voir les métriques de conteneur ?

Dans la même vue où se trouvent tous les graphes, vous pouvez voir quelques vignettes pour les performances de vos conteneurs. Vous avez besoin de l’Agent Log Analytics et de la [solution de supervision de conteneurs](service-fabric-diagnostics-oms-containers.md) pour que ces vignettes soient renseignées.

![Métriques des conteneurs Log Analytics](media/service-fabric-diagnostics-common-scenarios/containermetrics.png)

>[!NOTE]
>Pour utiliser les données de télémétrie **à l’intérieur** de votre conteneur, vous devez ajouter le [package nuget Application Insights pour les conteneurs](https://github.com/Microsoft/ApplicationInsights-servicefabric#microsoftapplicationinsightsservicefabric--for-service-fabric-lift-and-shift-scenarios).

## <a name="how-can-i-monitor-performance-counters"></a>Comment suivre les compteurs de performances ?

1. Une fois que vous avez ajouté l’Agent Log Analytics à votre cluster, vous devez ajouter les compteurs de performances spécifiques que vous voulez suivre. Accédez à la page de l’espace de travail Log Analytics dans le portail. Dans la page de la solution, l’onglet de l’espace de travail se trouve dans le menu de gauche.

    ![Onglet Espace de travail Log Analytics](media/service-fabric-diagnostics-common-scenarios/workspacetab.png)

2. Une fois que vous êtes dans la page de l’espace de travail, cliquez sur « Paramètres avancés » dans le même menu de gauche.

    ![Paramètres avancés de Log Analytics](media/service-fabric-diagnostics-common-scenarios/advancedsettingsoms.png)

3. Cliquez sur Données > Compteurs de performances Windows (Données > Compteurs de performances Linux pour les machines Linux) pour démarrer la collecte des compteurs spécifiques de vos nœuds via l’Agent Log Analytics. Voici des exemples du format des compteurs à ajouter

   * `.NET CLR Memory(<ProcessNameHere>)\\# Total committed Bytes`
   * `Processor(_Total)\\% Processor Time`

     Dans le guide de démarrage rapide, VotingData et VotingWeb sont les noms des processus utilisés ; voici à quoi devrait ressembler le suivi de ces compteurs

   * `.NET CLR Memory(VotingData)\\# Total committed Bytes`
   * `.NET CLR Memory(VotingWeb)\\# Total committed Bytes`

     ![Compteurs de performances Log Analytics](media/service-fabric-diagnostics-common-scenarios/omsperfcounters.png)

4. Ils vous permettent de voir comment votre infrastructure gère vos charges de travail et de définir des alertes pertinentes en fonction de l’utilisation des ressources. Par exemple, vous souhaiterez probablement définir une alerte si l’utilisation totale du processeur est supérieure à 90 % ou inférieure à 5 %. Le nom de compteur que vous utiliseriez pour cela serait « % temps processeur ». Vous pourriez alors créer une règle d’alerte pour la requête suivante :

    ```kusto
    Perf | where CounterName == "% Processor Time" and InstanceName == "_Total" | where CounterValue >= 90 or CounterValue <= 5.
    ```

## <a name="how-do-i-track-performance-of-my-reliable-services-and-actors"></a>Comment suivre les performances de mes événements Reliable Services et Actors ?

Pour le suivi des performances des événements Reliable Services ou Actors dans vos applications, vous devez aussi collecter les compteurs Service Fabric Actor, Actor Method, Service et Service Method. Voici des exemples de compteurs de performances des services et des acteurs fiables à collecter

>[!NOTE]
>Pour le moment, l’agent Log Analytics ne peut pas collecter les compteurs de performance de Service Fabric. Cependant, ces derniers peuvent être collectés par d’[autres solutions de diagnostic](service-fabric-diagnostics-partners.md)

* `Service Fabric Service(*)\\Average milliseconds per request`
* `Service Fabric Service Method(*)\\Invocations/Sec`
* `Service Fabric Actor(*)\\Average milliseconds per request`
* `Service Fabric Actor Method(*)\\Invocations/Sec`

Consultez ces liens pour obtenir la liste complète des compteurs de performances sur Reliable [Services](service-fabric-reliable-serviceremoting-diagnostics.md) et [Actors](service-fabric-reliable-actors-diagnostics.md)

## <a name="next-steps"></a>Étapes suivantes

* [Rechercher des erreurs d’activation du package de code commun](./service-fabric-diagnostics-code-package-errors.md)
* [Configurer des alertes dans AI](../azure-monitor/platform/alerts-log.md) pour être averti des changements de performances ou d’utilisation
* La [détection intelligente dans Application Insights](../azure-monitor/app/proactive-diagnostics.md) effectue une analyse proactive de la télémétrie envoyée à AI pour vous avertir d’éventuels problèmes de performances
* Découvrez plus en détail la [création d’alertes](../azure-monitor/platform/alerts-overview.md) dans les journaux Azure Monitor pour faciliter la détection et les diagnostics.
* Pour les clusters locaux, les journaux Azure Monitor proposent une passerelle (proxy de transfert HTTP) qui peut être utilisée pour envoyer des données aux journaux Azure Monitor. Pour plus d’informations à ce sujet, consultez [Connexion d’ordinateurs aux journaux Azure Monitor sans accès Internet à l’aide de la passerelle Log Analytics](../azure-monitor/platform/gateway.md)
* Familiarisez-vous avec les fonctionnalités de [requête et de recherche dans les journaux](../azure-monitor/log-query/log-query-overview.md) proposées par Journaux Azure Monitor
* Pour obtenir une présentation plus détaillée des journaux Azure Monitor et de ce qu’ils proposent, lisez [Présentation des journaux Azure Monitor](../azure-monitor/overview.md).
