---
title: Compteurs d’événements dans Application Insights | Microsoft Docs
description: Supervisez les compteurs d’événements .NET/.NET Core système et personnalisés dans Application Insights.
ms.topic: conceptual
ms.date: 09/20/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: a9af36f3c81ee52b41a8eed875c1a286b95bf838
ms.sourcegitcommit: 23aa0cf152b8f04a294c3fca56f7ae3ba562d272
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91803641"
---
# <a name="eventcounters-introduction"></a>Présentation des compteurs d’événements

[`EventCounter`](/dotnet/core/diagnostics/event-counters) est le mécanisme .NET/.NET Core permettant de publier et d’utiliser des compteurs ou des statistiques. Les compteurs d’événements sont pris en charge sur toutes les plateformes de système d’exploitation : Windows, Linux et macOS. Vous pouvez les considérer comme l’équivalent multiplateforme des [compteurs de performances](/dotnet/api/system.diagnostics.performancecounter), uniquement pris en charge dans les systèmes Windows.

Alors que les utilisateurs peuvent publier n’importe quel `EventCounters` personnalisé pour répondre à leurs besoins, le runtime .NET Core 3.0 ou version ultérieure publie un ensemble de ces compteurs par défaut. Ce document décrit les étapes nécessaires à la collecte et à l’affichage des `EventCounters` (définis par le système ou par l’utilisateur) dans Azure Application Insights.

## <a name="using-application-insights-to-collect-eventcounters"></a>Utilisation d’Application Insights pour collecter des compteurs d’événements

Application Insights prend en charge la collecte de `EventCounters` avec son `EventCounterCollectionModule`, qui fait partie du package NuGet récemment publié [Microsoft.ApplicationInsights.EventCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventCounterCollector). `EventCounterCollectionModule` est automatiquement activé dès que vous utilisez [AspNetCore](asp-net-core.md) ou [WorkerService](worker-service.md). `EventCounterCollectionModule` collecte les compteurs selon une fréquence non configurable de 60 secondes. Aucune autorisation spéciale n’est nécessaire pour collecter des compteurs d’événements.

## <a name="default-counters-collected"></a>Compteurs par défaut collectés

À partir de la version 2.15.0 du [Kit de développement logiciel (SDK) AspNetCore](asp-net-core.md) et du [Kit de développement logiciel (SDK) WorkerService](worker-service.md), aucun compteur n’est collecté par défaut. Le module lui-même est activé, les utilisateurs peuvent donc simplement ajouter les compteurs souhaités pour les collecter.

Pour obtenir la liste des compteurs connus publiés par le Runtime .NET, consultez le document [Compteurs disponibles](/dotnet/core/diagnostics/event-counters#available-counters).

## <a name="customizing-counters-to-be-collected"></a>Personnalisation des compteurs à collecter

L’exemple suivant montre comment ajouter/supprimer des compteurs. Cette personnalisation est effectuée dans la méthode `ConfigureServices` de votre application une fois que la collecte de données de télémétrie Application Insights est activée en utilisant `AddApplicationInsightsTelemetry()` ou `AddApplicationInsightsWorkerService()`. Voici un exemple de code d’une application ASP.NET Core. Pour les autres types d’applications, reportez-vous à [ce](worker-service.md#configuring-or-removing-default-telemetrymodules) document.

```csharp
    using Microsoft.ApplicationInsights.Extensibility.EventCounterCollector;
    using Microsoft.Extensions.DependencyInjection;

    public void ConfigureServices(IServiceCollection services)
    {
        //... other code...

        // The following code shows how to configure the module to collect
        // additional counters.
        services.ConfigureTelemetryModule<EventCounterCollectionModule>(
            (module, o) =>
            {
                // This removes all default counters, if any.
                module.Counters.Clear();

                // This adds a user defined counter "MyCounter" from EventSource named "MyEventSource"
                module.Counters.Add(new EventCounterCollectionRequest("MyEventSource", "MyCounter"));

                // This adds the system counter "gen-0-size" from "System.Runtime"
                module.Counters.Add(new EventCounterCollectionRequest("System.Runtime", "gen-0-size"));
            }
        );
    }
```

## <a name="disabling-eventcounter-collection-module"></a>Désactivation du module de collecte EventCounter

`EventCounterCollectionModule` peut être désactivé avec `ApplicationInsightsServiceOptions`. Vous trouverez ci-dessous un exemple d’utilisation du kit de développement logiciel (SDK) ASP.NET Core.

```csharp
    using Microsoft.ApplicationInsights.AspNetCore.Extensions;
    using Microsoft.Extensions.DependencyInjection;

    public void ConfigureServices(IServiceCollection services)
    {
        //... other code...

        var applicationInsightsServiceOptions = new ApplicationInsightsServiceOptions();
        applicationInsightsServiceOptions.EnableEventCounterCollectionModule = false;
        services.AddApplicationInsightsTelemetry(applicationInsightsServiceOptions);
    }
```

Une approche similaire peut également être utilisée pour le kit de développement logiciel (SDK) WorkerService, mais l’espace de noms doit être modifié comme indiqué dans l’exemple ci-dessous.

```csharp
    using Microsoft.ApplicationInsights.WorkerService;
    using Microsoft.Extensions.DependencyInjection;

    var applicationInsightsServiceOptions = new ApplicationInsightsServiceOptions();
    applicationInsightsServiceOptions.EnableEventCounterCollectionModule = false;
    services.AddApplicationInsightsTelemetryWorkerService(applicationInsightsServiceOptions);
```

## <a name="event-counters-in-metric-explorer"></a>Compteurs d’événements dans Metric Explorer

Pour voir les métriques de compteur d’événements dans [Metric Explorer](../platform/metrics-charts.md), sélectionnez une ressource Application Insights, puis choisissez des métriques basées sur un journal comme espace de noms de métrique. Ainsi, les métriques de compteur d’événements s’affichent sous Catégorie personnalisée.

> [!div class="mx-imgBorder"]
> ![Compteurs d’événements signalés dans Application Insights Metrics Explorer](./media/event-counters/metrics-explorer-counter-list.png)

## <a name="event-counters-in-analytics"></a>Compteurs d’événements dans Analytics

Vous pouvez également rechercher et afficher des rapports de compteurs d’événements dans [Analytics](../log-query/log-query-overview.md), dans la table **customMetrics**.

Par exemple, exécutez la requête suivante pour voir quels compteurs sont collectés et disponibles pour la requête :

```Kusto
customMetrics | summarize avg(value) by name
```

> [!div class="mx-imgBorder"]
> ![Compteurs d’événements signalés dans Application Insights Analytics](./media/event-counters/analytics-event-counters.png)

Pour obtenir un graphique d’un compteur spécifique (par exemple, `ThreadPool Completed Work Item Count`) sur la période récente, exécutez la requête suivante.

```Kusto
customMetrics 
| where name contains "System.Runtime|ThreadPool Completed Work Item Count"
| where timestamp >= ago(1h)
| summarize  avg(value) by cloud_RoleInstance, bin(timestamp, 1m)
| render timechart
```
> [!div class="mx-imgBorder"]
> ![Conversation d’un compteur unique dans Application Insights](./media/event-counters/analytics-completeditems-counters.png)

Comme les autres données de télémétrie, **customMetrics** possède également une colonne `cloud_RoleInstance` qui indique l’identité de l’instance de serveur hôte sur lequel votre application est en cours d’exécution. La requête ci-dessus montre la valeur de compteur par instance et peut servir à comparer les performances des différentes instances de serveur.

## <a name="alerts"></a>Alertes
À l’instar d’autres métriques, vous pouvez [définir une alerte](../platform/alerts-log.md) pour vous avertir si un compteur d’événements dépasse une limite que vous spécifiez. Ouvrez le volet Alertes et cliquez sur Ajouter une alerte.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="can-i-see-eventcounters-in-live-metrics"></a>Puis-je voir les compteurs d’événements dans les métriques temps réel ?

Les métriques temps réel ne présentent pas les compteurs d’événements à partir du jour actuel. Utilisez Metrics Explorer ou Analytics pour voir les données de télémétrie.

### <a name="i-have-enabled-application-insights-from-azure-web-app-portal-but-i-cant-see-eventcounters"></a>J’ai activé Application Insights à partir du portail d’application web Azure. Mais je ne vois pas de compteurs d’événements.

 L’[extension Application Insights](./azure-web-apps.md) pour ASP.NET Core ne prend pas encore en charge cette fonctionnalité. Ce document sera mis à jour quand cette fonctionnalité sera prise en charge.

## <a name="next-steps"></a><a name="next"></a>Étapes suivantes

* [Suivi des dépendances](./asp-net-dependencies.md)

