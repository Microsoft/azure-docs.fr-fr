---
title: GetMetric dans Azure Monitor Application Insights
description: Découvrez comment utiliser efficacement l’appel GetMetric() afin de capturer des métriques pré-agrégées localement pour les applications .NET et .NET Core avec Azure Monitor Application Insights.
ms.service: azure-monitor
ms.subservice: application-insights
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 345d0d31528f7bdc40be4400e783ad5be45df72f
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91930558"
---
# <a name="custom-metric-collection-in-net-and-net-core"></a>Collecte de métriques personnalisées dans .NET et .NET Core

Les kits SDK .NET et .NET Core Azure Monitor Application Insights offrent deux méthodes de collecte des métriques personnalisées, `TrackMetric()` et `GetMetric()`. La principale différence entre ces deux méthodes est l’agrégation locale. Avec `TrackMetric()` il n’y a pas de pré-agrégation, tandis qu’avec `GetMetric()` il y en a une. L’approche recommandée consiste à utiliser l’agrégation. Par conséquent, `TrackMetric()` n’est plus la méthode recommandée pour recueillir des métriques personnalisées. Cet article explique pas à pas comment utiliser la méthode GetMetric(), ainsi que la logique qui sous-tend son fonctionnement.

## <a name="trackmetric-versus-getmetric"></a>TrackMetric et GetMetric

`TrackMetric()` envoie des données de télémétrie brutes indiquant une métrique. Il est inefficace d’envoyer un seul élément de télémétrie pour chaque valeur. `TrackMetric()` est également inefficace en termes de performances, car chaque `TrackMetric(item)` transite par le pipeline de SDK complet des initialiseurs et des processeurs de télémétrie. Contrairement à `TrackMetric()`, `GetMetric()` gère la pré-agrégation locale pour vous et envoie ensuite une métrique récapitulative agrégée à un intervalle fixe d’une minute. Ainsi, si vous avez besoin de superviser étroitement une métrique personnalisée au niveau de la seconde ou même de la milliseconde, vous pouvez le faire tout en n’encourant que le coût de stockage et de trafic réseau de la supervision à intervalle d’une minute. Cela réduit aussi considérablement le risque de limitation, car le nombre total d’éléments de télémétrie à envoyer pour une métrique agrégée est fortement réduit.

Dans Application Insights, les métriques personnalisées collectées par le biais de `TrackMetric()` et `GetMetric()` ne sont pas soumises à l’[échantillonnage](./sampling.md). L’échantillonnage de métriques importantes peut entraîner des scénarios dans lesquels les alertes que vous avez créées autour de ces métriques peuvent devenir peu fiables. En n’échantillonnant jamais vos métriques personnalisées, vous pouvez généralement être sûr que quand vos seuils d’alerte sont enfreints, une alerte est déclenchée.  Toutefois, sans échantillonnage des métriques personnalisées, il existe des problèmes potentiels.

Si vous avez besoin d’effectuer le suivi des tendances dans une métrique chaque seconde, ou à un intervalle encore plus détaillé, cela peut se traduire par :

- Une augmentation des coûts de stockage de données. Il existe un coût associé à la quantité de données que vous envoyez à Azure Monitor. (Plus vous envoyez de données, plus le coût global de supervision est élevé.)
- Une augmentation du trafic réseau/une baisse des performances. (Dans certains scénarios, cela peut avoir un coût en termes de performances d’application et un coût financier.)
- Un risque de limitation de l’ingestion. (Le service Application Monitor supprime (« limite ») des points de données quand votre application envoie un taux de télémétrie très élevé dans un court laps de temps.)

La limitation est particulièrement importante dans le sens où, comme l’échantillonnage, elle peut entraîner des non-signalement d’alertes puisque la condition de déclenchement d’une alerte peut se produire localement, puis être supprimée au point de terminaison d’ingestion en raison d’un trop grand nombre de données envoyées. C’est pourquoi, pour .NET et .NET Core, nous ne recommandons pas l’utilisation de `TrackMetric()`, sauf si vous avez implémenté votre propre logique d’agrégation locale. Si vous essayez d’effectuer le suivi de chaque occurrence d’un événement sur une période donnée, vous constaterez peut-être que [`TrackEvent()`](./api-custom-events-metrics.md#trackevent) est plus adaptée. Toutefois, n’oubliez pas que contrairement aux métriques personnalisées, les événements personnalisés sont soumis à l’échantillonnage. Bien entendu, vous pouvez toujours utiliser `TrackMetric()` même sans écrire votre propre pré-agrégation locale, mais dans ce cas vous devez être conscient des risques.

En résumé, l’approche recommandée consiste à utiliser `GetMetric()`, car elle effectue une pré-agrégation, elle accumule les valeurs de tous les appels Track() et envoie un résumé/une agrégation une fois par minute. Cela peut réduire considérablement le coût et les problèmes de performances en envoyant moins de points de données, tout en recueillant néanmoins toutes les informations pertinentes.

> [!NOTE]
> Seuls les kits SDK .NET et .NET Core ont une méthode GetMetric(). Si vous utilisez Java, vous pouvez utiliser des [métriques Micrometer](./micrometer-java.md) ou `TrackMetric()`. Pour Python, vous pouvez utiliser [OpenCensus.stats](./opencensus-python.md#metrics) pour envoyer des métriques personnalisées. Pour JavaScript et Node.js, vous utiliserez quand même `TrackMetric()`, mais gardez à l’esprit les avertissements décrits dans la section précédente.

## <a name="getting-started-with-getmetric"></a>Bien démarrer avec GetMetric

Pour nos exemples, nous allons utiliser une application de service Worker .NET Core 3.1 de base. Si vous souhaitez répliquer exactement l’environnement de test utilisé avec ces exemples, suivez les étapes 1 à 6 de l’[article sur la supervision de service Worker](./worker-service.md#net-core-30-worker-service-application) pour ajouter Application Insights à un modèle de projet de service Worker de base. Ces concepts s’appliquent à toute application générale dans laquelle le SDK peut être utilisé, notamment les applications web et les applications de console.

### <a name="sending-metrics"></a>Envoi de métriques

Remplacez le contenu de votre fichier `worker.cs` par ce qui suit :

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.ApplicationInsights;

namespace WorkerService3
{
    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private TelemetryClient _telemetryClient;

        public Worker(ILogger<Worker> logger, TelemetryClient tc)
        {
            _logger = logger;
            _telemetryClient = tc;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {   // The following line demonstrates usages of GetMetric API.
            // Here "computersSold", a custom metric name, is being tracked with a value of 42 every second.
            while (!stoppingToken.IsCancellationRequested)
            {
                _telemetryClient.GetMetric("computersSold").TrackValue(42);

                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(1000, stoppingToken);
            }
        }
    }
}
```

Si vous exécutez le code ci-dessus et observez la télémétrie envoyée par le biais de la fenêtre de sortie de Visual Studio ou d’un outil tel que Fiddler de Telerik, vous verrez que la boucle while s’exécute de manière répété sans que la télémétrie soit envoyée, puis qu’un seul élément de télémétrie est envoyé aux alentours de la marque de 60 secondes, qui dans le cadre de notre test ressemble à ceci :

```json
Application Insights Telemetry: {"name":"Microsoft.ApplicationInsights.Dev.00000000-0000-0000-0000-000000000000.Metric", "time":"2019-12-28T00:54:19.0000000Z",
"ikey":"00000000-0000-0000-0000-000000000000",
"tags":{"ai.application.ver":"1.0.0.0",
"ai.cloud.roleInstance":"Test-Computer-Name",
"ai.internal.sdkVersion":"m-agg2c:2.12.0-21496",
"ai.internal.nodeName":"Test-Computer-Name"},
"data":{"baseType":"MetricData",
"baseData":{"ver":2,"metrics":[{"name":"computersSold",
"kind":"Aggregation",
"value":1722,
"count":41,
"min":42,
"max":42,
"stdDev":0}],
"properties":{"_MS.AggregationIntervalMs":"42000",
"DeveloperMode":"true"}}}}
```

Cet élément de télémétrie unique représente une agrégation de 41 mesures de métriques distinctes. Étant donné que nous envoyions sans cesse la même valeur, nous avons un *écart type (stDev)* de 0 avec des valeurs *maximale (max)* et *minimale (min)* identiques. La propriété *value* représente la somme de toutes les valeurs individuelles qui ont été agrégées.

Si nous examinons notre ressource Application Insights dans l’expérience Journaux (analytique), cet élément de télémétrie se présentera comme suit :

![Vue de la requête Log Analytics](./media/get-metric/log-analytics.png)

> [!NOTE]
> L’élément de télémétrie brut ne contenait pas de champ ou de propriété de somme explicite. Une fois ingéré, nous en créons un pour vous. En l’occurrence, la propriété `value` et `valueSum` représentent la même chose.

Vous pouvez également accéder à vos données de télémétrie de métriques personnalisées dans la section [_Métriques_](../platform/metrics-charts.md) du portail, à la fois en tant que [métriques basées sur un journal et métriques personnalisées](pre-aggregated-metrics-log-metrics.md). (La capture d’écran ci-dessous montre un exemple de métriques basées sur un journal.) ![Vue de Metrics Explorer](./media/get-metric/metrics-explorer.png)

### <a name="caching-metric-reference-for-high-throughput-usage"></a>Mise en cache de référence de métrique pour une utilisation à débit élevé

Dans certains cas, les valeurs de métrique sont observées très fréquemment. Par exemple, un service à débit élevé qui traite 500 requêtes/seconde peut souhaiter émettre 20 métriques de télémétrie pour chaque requête. Cela signifie qu’il faut effectuer le suivi de 10 000 valeurs par seconde. Dans de tels scénarios à débit élevé, les utilisateurs peuvent devoir aider le SDK en évitant certaines recherches.

Par exemple, l’exemple ci-dessus a effectué une recherche pour un descripteur pour la métrique « ComputersSold », puis a suivi une valeur observée 42. Au lieu de cela, le descripteur peut être mis en cache pour plusieurs appels de suivi :

```csharp
//...

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // This is where the cache is stored to handle faster lookup
            Metric computersSold = _telemetryClient.GetMetric("ComputersSold");
            while (!stoppingToken.IsCancellationRequested)
            {

                computersSold.TrackValue(42);

                computersSold.TrackValue(142);

                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(50, stoppingToken);
            }
        }

```

Outre la mise en cache du descripteur de métrique, l’exemple ci-dessus a également réduit `Task.Delay` à 50 millisecondes, de sorte que la boucle s’exécute plus fréquemment, entraînant 772 appels à `TrackValue()`.

## <a name="multi-dimensional-metrics"></a>Métriques multidimensionnelles

Les exemples de la section précédente présentent des métriques à zéro dimension. Les métriques peuvent également être multidimensionnelles. Nous prenons actuellement en charge jusqu’à 10 dimensions.

 Voici un exemple qui illustre comment créer une métrique unidimensionnelle :

```csharp
//...

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // This is an example of a metric with a single dimension.
            // FormFactor is the name of the dimension.
            Metric computersSold= _telemetryClient.GetMetric("ComputersSold", "FormFactor");

            while (!stoppingToken.IsCancellationRequested)
            {
                // The number of arguments (dimension values)
                // must match the number of dimensions specified while GetMetric.
                // Laptop, Tablet, etc are values for the dimension "FormFactor"
                computersSold.TrackValue(42, "Laptop");
                computersSold.TrackValue(20, "Tablet");
                computersSold.TrackValue(126, "Desktop");


                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(50, stoppingToken);
            }
        }

```

L’exécution de ce code pendant au moins 60 secondes entraînera l’envoi de trois éléments de télémétrie distincts à Azure, chacun représentant l’agrégation de l’un des trois facteurs de forme. Comme auparavant, vous pouvez les examiner dans la vue Journaux (analytique) :

![Vue Log Analytics de métrique multidimensionnelle](./media/get-metric/log-analytics-multi-dimensional.png)

Ainsi que dans l’expérience Metrics Explorer :

![Mesures personnalisées](./media/get-metric/custom-metrics.png)

Toutefois, vous remarquerez que vous ne pouvez pas diviser la métrique par votre nouvelle dimension personnalisée, ni afficher votre dimension personnalisée avec la vue des métriques :

![Prise en charge de la division](./media/get-metric/splitting-support.png)

Par défaut, les métriques multidimensionnelles dans l’expérience Metrics Explorer ne sont pas activées dans les ressources Application Insights.

### <a name="enable-multi-dimensional-metrics"></a>Activer les métriques multidimensionnelles

Pour activer les métriques multidimensionnelles pour une ressource Application Insights, sélectionnez **Utilisation et estimation des coûts** > **Métriques personnalisées** > **Activer les alertes sur les dimensions des métriques personnalisées** > **OK**. Pour plus d’informations à ce sujet, cliquez [ici](pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).

Une fois que vous avez apporté cette modification et envoyé de nouvelles données de télémétrie multidimensionnelles, vous pouvez **appliquer la division**.

> [!NOTE]
> Seules les métriques envoyées après l’activation de la fonctionnalité dans le portail auront des dimensions stockées.

![Appliquer la division](./media/get-metric/apply-splitting.png)

Et afficher vos agrégations de métriques pour chaque dimension _FormFactor_ :

![Facteurs de forme](./media/get-metric/formfactor.png)

### <a name="how-to-use-metricidentifier-when-there-are-more-than-three-dimensions"></a>Comment utiliser MetricIdentifier quand il y a plus de trois dimensions ?

Dix dimensions sont actuellement prises en charge, mais la présence de plus de trois dimensions requiert l’utilisation de `MetricIdentifier` :

```csharp
// Add "using Microsoft.ApplicationInsights.Metrics;" to use MetricIdentifier
// MetricIdentifier id = new MetricIdentifier("[metricNamespace]","[metricId],"[dim1]","[dim2]","[dim3]","[dim4]","[dim5]");
MetricIdentifier id = new MetricIdentifier("CustomMetricNamespace","ComputerSold", "FormFactor", "GraphicsCard", "MemorySpeed", "BatteryCapacity", "StorageCapacity");
Metric computersSold  = _telemetryClient.GetMetric(id);
computersSold.TrackValue(110,"Laptop", "Nvidia", "DDR4", "39Wh", "1TB");
```

## <a name="custom-metric-configuration"></a>Configuration de métrique personnalisée

Si vous souhaitez modifier la configuration des métriques, vous devez le faire à l’endroit où la métrique est initialisée.

### <a name="special-dimension-names"></a>Noms de dimensions spéciales

Les métriques n’utilisent pas le contexte de télémétrie du `TelemetryClient` utilisé pour y accéder. Les noms de dimensions spéciales disponibles en tant que constantes dans la classe `MetricDimensionNames` constituent la meilleure solution de contournement pour cette limitation.

Les agrégations de métriques envoyées par la métrique « Special Operation Request Size » ci-dessous n’auront **pas** leur `Context.Operation.Name` défini sur « Special Operation », alors que `TrackMetric()` ou tout autre TrackXXX() aura `OperationName` défini correctement sur « Special Operation ».

``` csharp
        //...
        TelemetryClient specialClient;
        private static int GetCurrentRequestSize()
        {
            // Do stuff
            return 1100;
        }
        int requestSize = GetCurrentRequestSize()

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                //...
                specialClient.Context.Operation.Name = "Special Operation";
                specialClient.GetMetric("Special Operation Request Size").TrackValue(requestSize);
                //...
            }
                   
        }
```

Dans cette situation, utilisez les noms de dimensions spéciales listés dans la classe `MetricDimensionNames` afin de spécifier des valeurs `TelemetryContext`.

Par exemple, quand l’agrégation de métriques résultant de l’instruction suivante est envoyé au point de terminaison cloud Application Insights, son champ de données `Context.Operation.Name` est défini sur « Special Operation » :

```csharp
_telemetryClient.GetMetric("Request Size", MetricDimensionNames.TelemetryContext.Operation.Name).TrackValue(requestSize, "Special Operation");
```

Les valeurs de cette dimension spéciale sont copiées dans le `TelemetryContext` et ne seront pas utilisées comme dimension « normale ». Si vous souhaitez également conserver une dimension d’opération pour l’exploration normale des métriques, vous devez créer une dimension distincte à cet effet :

```csharp
_telemetryClient.GetMetric("Request Size", "Operation Name", MetricDimensionNames.TelemetryContext.Operation.Name).TrackValue(requestSize, "Special Operation", "Special Operation");
```

### <a name="dimension-and-time-series-capping"></a>Limitation des dimensions et des séries chronologiques

 Pour empêcher que le sous-système de télémétrie n’utilise accidentellement vos ressources, vous pouvez contrôler la quantité maximale de séries de données par métrique. Les limites par défaut sont : 1000 séries de données au total par métrique, et 100 valeurs différentes par dimension.

 Dans le contexte de la limitation des séries chronologiques et des dimensions, nous utilisons `Metric.TrackValue(..)` pour nous assurer que les limites sont respectées. Si les limites sont déjà atteintes, `Metric.TrackValue(..)` retourne « false » et la valeur n’est pas suivie. Dans le cas contraire, elle retourne la valeur « True ». Ceci est utile si les données d’une métrique proviennent d’une entrée d’utilisateur.

Le constructeur `MetricConfiguration` accepte certaines options relatives à la façon de gérer différentes séries au sein de la métrique et un objet d’une classe implémentant `IMetricSeriesConfiguration` qui spécifie le comportement d’agrégation pour chaque série de la métrique :

``` csharp
var metConfig = new MetricConfiguration(seriesCountLimit: 100, valuesPerDimensionLimit:2,
                new MetricSeriesConfigurationForMeasurement(restrictToUInt32Values: false));

Metric computersSold = _telemetryClient.GetMetric("ComputersSold", "Dimension1", "Dimension2", metConfig);

// Start tracking.
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value1");
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value2");

// The following call gives 3rd unique value for dimension2, which is above the limit of 2.
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value3");
// The above call does not track the metric, and returns false.
```

* `seriesCountLimit` est la quantité maximale de séries chronologiques de données qu’une métrique peut contenir. Une fois cette limite atteinte, les appels à `TrackValue()`.
* `valuesPerDimensionLimit` limite le nombre de valeurs distinctes par dimension d’une manière similaire.
* `restrictToUInt32Values` détermine si seules les valeurs entières non négatives doivent être suivies.

Voici un exemple qui illustre comment envoyer un message pour savoir si les limites sont dépassées :

```csharp
if (! computersSold.TrackValue(100, "Dim1Value1", "Dim2Value3"))
{
// Add "using Microsoft.ApplicationInsights.DataContract;" to use SeverityLevel.Error
_telemetryClient.TrackTrace("Metric value not tracked as value of one of the dimension exceeded the cap. Revisit the dimensions to ensure they are within the limits",
SeverityLevel.Error);
}
```

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus](./worker-service.md) sur la supervision des applications de service Worker.
* En savoir plus sur [les métriques basées sur les journaux et les métriques pré-agrégées](./pre-aggregated-metrics-log-metrics.md).
* [Metric Explorer](../platform/metrics-getting-started.md)
* Guide pratique pour activer Application Insights pour les [applications ASP.NET Core](asp-net-core.md)
* Guide pratique pour activer Application Insights pour les [applications ASP.NET](asp-net.md)
