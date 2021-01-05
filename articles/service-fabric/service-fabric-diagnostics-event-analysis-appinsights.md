---
title: Analyse d’événement Azure Service Fabric avec Application Insights
description: Découvrez l’analyse et la visualisation d’événements à l’aide d’Application Insights pour la surveillance et le diagnostic de clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 11/21/2018
ms.author: srrengar
ms.openlocfilehash: e35206b5fa9466cda064c09f060f45b437fafd20
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92329574"
---
# <a name="event-analysis-and-visualization-with-application-insights"></a>Analyse et visualisation d’événements avec Application Insights

Dans le cadre d’Azure Monitor, Application Insights est une plateforme extensible pour superviser et diagnostiquer les applications. Elle inclut un puissant outil d’analyse et de requête, un tableau de bord et des visualisations personnalisables, ainsi que d’autres options incluant l’alerte automatisée. L’intégration d’Application Insights avec Service Fabric comprend des outils pour Visual Studio et le portail Azure, ainsi que des métriques spécifiques à Service Fabric, ce qui fournit une expérience de journalisation complète et prête à l’emploi. Même si de nombreux journaux d’activité sont automatiquement créés et collectés pour vous quand vous utilisez Application Insights, nous vous recommandons d’ajouter une journalisation personnalisée à vos applications pour créer une expérience de diagnostics plus riche.

Cet article répond aux questions courantes suivantes :

* Comment savoir ce qui se passe à l’intérieur de mon application et de mes services, et comment recueillir des données de télémétrie ?
* Comment résoudre les problèmes liés à mon application, et notamment aux services qui communiquent les uns avec les autres ?
* Comment obtenir des métriques sur les performances de mes services, comme le temps de chargement des pages ou les requêtes HTTP ?

Le rôle de cet article est de vous indiquer comment obtenir des informations détaillées et résoudre les problèmes à partir d’Application Insights (AI). Si vous souhaitez savoir comment installer et configurer Application Insights avec Service Fabric, consultez ce [didacticiel](service-fabric-tutorial-monitoring-aspnet.md).

## <a name="monitoring-in-application-insights"></a>Supervision dans Application Insights

Application Insights bénéficie d’une expérience riche et prête à l’emploi pour une utilisation avec Service Fabric. Dans la page de vue d’ensemble, Application Insights fournit des informations clés concernant votre service, telles que le temps de réponse et le nombre de requêtes traitées. Vous pouvez obtenir la liste des requêtes récentes dans votre application en cliquant sur le bouton « Rechercher » dans la partie supérieure de l’écran. En outre, la page qui s’affiche vous permet de visualiser les requêtes ayant échoué et de diagnostiquer les erreurs éventuelles.

![Vue d'ensemble Application Insights](media/service-fabric-diagnostics-event-analysis-appinsights/ai-overview.png)

Dans le panneau droit de la page illustrée ci-dessus, la liste comporte deux principaux types d’entrées : les requêtes et les événements. Les requêtes sont des appels adressés à l’API de l’application par le biais de requêtes HTTP dans ce cas précis, tandis que les événements sont des événements personnalisés, qui font office de données de télémétrie que vous pouvez ajouter à un endroit quelconque de votre code. Vous pouvez obtenir plus d’informations sur l’instrumentation de vos applications en consultant l’article [API Application Insights pour les événements et les mesures personnalisés](../azure-monitor/app/api-custom-events-metrics.md). En cliquant sur une requête, vous pouvez afficher de plus amples informations (cf. image suivante), notamment des données propres à Service Fabric, qui sont collectées dans le package NuGet Application Insights Service Fabric. Ces informations peuvent vous aider à résoudre les problèmes et à connaître l’état de votre application, et peuvent toutes faire l’objet de recherches dans Application Insights.

![Capture d’écran montrant des détails supplémentaires, notamment des données propres à Service Fabric, qui sont collectées dans le package NuGet Application Insights Service Fabric.](media/service-fabric-diagnostics-event-analysis-appinsights/ai-request-details.png)

Le service Application Insights intègre une vue désignée pour l’interrogation de toutes les données qu’il collecte. Sélectionnez « Metrics Explorer » en haut de la page Vue d’ensemble pour accéder au portail Application Insights. Ce portail vous permet d’exécuter des requêtes portant sur les événements personnalisés mentionnés précédemment, les exceptions, les compteurs de performances et d’autres métriques à l’aide du langage de requête Kusto. L’exemple ci-après présente toutes les requêtes exécutées au cours de la dernière heure.

![Détails de la requête Application Insights](media/service-fabric-diagnostics-event-analysis-appinsights/ai-metrics-explorer.png)

Pour découvrir en détail les fonctionnalités du portail Application Insights, reportez-vous à la [documentation du portail Application Insights](../azure-monitor/app/overview-dashboard.md).

### <a name="configuring-application-insights-with-eventflow"></a>Configuration d'Application Insights avec EventFlow

Si vous utilisez EventFlow pour agréger des événements, veillez à importer le package NuGet `Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights`. Le code ci-après est requis dans la section *outputs* du fichier *eventFlowConfig.json* :

```json
"outputs": [
    {
        "type": "ApplicationInsights",
        "instrumentationKey": "***ADD INSTRUMENTATION KEY HERE**_"
    }
]
```

Veillez à apporter les modifications obligatoires dans vos filtres, ainsi qu’à inclure toutes les autres entrées (avec leurs packages NuGet respectifs).

## <a name="application-insights-sdk"></a>Kit de développement logiciel (SDK) Application Insights

Il est recommandé d’utiliser EventFlow et WAD comme solutions d’agrégation, car elles permettent d’adopter une approche plus modulaire des diagnostics et de la surveillance. Par exemple, si vous voulez modifier vos sorties d’EventFlow, cette opération n’exige aucune modification de votre instrumentation proprement dite, mais implique une simple modification de votre fichier config. Si vous décidez d’investir dans Application Insights et qu’il est peu probable que vous changiez de plateforme, envisagez d’utiliser le nouveau kit SDK d’Application Insights pour agréger les événements et les envoyer à Application Insights. Cela signifie que vous n’aurez plus à configurer EventFlow pour envoyer vos données à Application Insights, mais simplement à installer le package NuGet Service Fabric d’Application Insights. Des informations supplémentaires sur le package sont disponibles [ici](https://github.com/Microsoft/ApplicationInsights-ServiceFabric).

La rubrique [Prise en charge d’Application Insights pour les microservices et les conteneurs](https://azure.microsoft.com/blog/app-insights-microservices/) vous présente quelques-unes des nouvelles fonctionnalités en cours de développement (encore actuellement en version bêta), qui vous permettent de bénéficier d’options de surveillance prêtes à l’emploi enrichies avec Application Insights. Ces fonctionnalités incluent le suivi de dépendance (utilisé dans la création d’une AppMap de l’ensemble de vos services et applications dans un cluster et la communication entre eux), ainsi qu’une meilleure corrélation des suivis provenant de vos services (ce qui facilite l’identification d’un problème dans le flux de travail d’une application ou d’un service).

Si vous effectuez un développement dans .NET, que vous utiliserez probablement certains des modèles de programmation de Service Fabric et que vous souhaitez utiliser Application Insights comme plateforme pour la visualisation et l’analyse des données d’événement et de journal, nous vous recommandons d’utiliser l’itinéraire du SDK Application Insights comme flux de travail de surveillance et de diagnostic. Lisez la [documentation Application Insights](../azure-monitor/azure-monitor-app-hub.yml) et la [documentation des journaux de suivi](../azure-monitor/app/asp-net-trace-logs.md) pour commencer à utiliser Application Insights afin de collecter et d’afficher vos journaux.

## <a name="navigating-the-application-insights-resource-in-azure-portal"></a>Accès à la ressource Application Insights dans le portail Azure

Une fois que vous avez configuré Application Insights comme sortie pour vos événements et journaux d’activité, les informations doivent commencer à s’afficher dans votre ressource Application Insights au bout de quelques minutes. Accédez à la ressource Application Insights, qui vous permettra d’accéder au tableau de bord des ressources Application Insights. Sélectionnez _ *Rechercher* * dans la barre des tâches Application Insights pour voir les dernières traces reçues et pour pouvoir les filtrer.

*Metrics Explorer* est un outil utile pour la création de tableaux de bord personnalisés basés sur les indicateurs de performance que vos applications, vos services et votre cluster peuvent signaler. Consultez [Exploration des indicateurs de performance dans Application Insights](../azure-monitor/platform/metrics-charts.md) pour configurer vous-mêmes quelques graphiques basés sur les données que vous collectez.

Si vous cliquez sur **Analyse** , vous accédez au portail Application Insights Analytics, où vous pouvez interroger des événements et des traces avec une plus grande étendue et plus de choix. Vous pouvez en apprendre plus à ce sujet dans [Analytics dans Application Insights](../azure-monitor/log-query/log-query-overview.md).

## <a name="next-steps"></a>Étapes suivantes

* [Configurer des alertes dans AI](../azure-monitor/platform/alerts-log.md) pour être averti des changements de performances ou d’utilisation
* La [détection intelligente dans Application Insights](../azure-monitor/app/proactive-diagnostics.md) effectue une analyse proactive de la télémétrie envoyée à Application Insights pour vous avertir d’éventuels problèmes de performances
