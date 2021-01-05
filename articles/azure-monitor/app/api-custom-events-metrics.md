---
title: API Application Insights pour les événements et les métriques personnalisés | Microsoft Docs
description: Insérez quelques lignes de code dans votre application de périphérique ou de bureau, votre page web ou votre service pour suivre l'utilisation et diagnostiquer les problèmes.
ms.topic: conceptual
ms.date: 05/11/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: ae64888669fb9a3c053802ee4f7ad7db6316265d
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780499"
---
# <a name="application-insights-api-for-custom-events-and-metrics"></a>API Application Insights pour les événements et les mesures personnalisés

Insérez quelques lignes de code dans votre application pour découvrir ce qu’en font les utilisateurs ou pour faciliter le diagnostic des problèmes. Vous pouvez envoyer la télémétrie depuis des applications de périphérique et de bureau, des clients web et des serveurs web. Utilisez l’API des données de télémétrie principales [Azure Application Insights](./app-insights-overview.md) pour envoyer des événements et métriques personnalisés, ainsi que vos propres versions de la télémétrie standard. Cette API est la même que celle utilisée par les collecteurs de données standard d’Application Insights.

## <a name="api-summary"></a>Résumé des API

L’API principale est uniforme sur toutes les plateformes, à l’exception de quelques petites variantes telles que `GetMetric` (.NET uniquement).

| Méthode | Utilisé pour |
| --- | --- |
| [`TrackPageView`](#page-views) |Pages, écrans, panneaux ou formes. |
| [`TrackEvent`](#trackevent) |Actions de l’utilisateur et autres événements. Utilisé pour suivre le comportement de l’utilisateur ou pour analyser les performances. |
| [`GetMetric`](#getmetric) |Zéro métrique et métriques multidimensionnels, agrégation configurée de manière centralisée, C# uniquement. |
| [`TrackMetric`](#trackmetric) |Mesures de performances telles que la longueur des files d’attente non liées à des événements spécifiques. |
| [`TrackException`](#trackexception) |Exceptions de journal pour des diagnostics. Effectuez un suivi lorsqu’ils se produisent par rapport à d’autres événements et examinez les arborescences des appels de procédure. |
| [`TrackRequest`](#trackrequest) |Notez la fréquence et la durée des requêtes du serveur pour l’analyse des performances. |
| [`TrackTrace`](#tracktrace) |Messages du journal de diagnostic des ressources. Vous pouvez également capturer des journaux d’activité tiers. |
| [`TrackDependency`](#trackdependency) |La journalisation de la durée et de la fréquence des appels vers les composants externes dont dépend votre application. |

Vous pouvez [associer des propriétés et des mesures](#properties) à la plupart de ces appels de télémétrie.

## <a name="before-you-start"></a><a name="prep"></a>Avant de commencer

Si vous n’avez pas encore de référence sur le kit SDK Application Insights :

* Ajoutez le Kit de développement logiciel (SDK) Application Insights à votre projet :

  * [Projet ASP.NET](./asp-net.md)
  * [ASP.NET Core project](./asp-net-core.md)
  * [Projet Java](./java-get-started.md)
  * [Projet Node.js](./nodejs.md)
  * [JavaScript dans chaque page web](./javascript.md) 
* Ajoutez au code de votre périphérique ou de votre serveur web :

    *C#:* `using Microsoft.ApplicationInsights;`

    *Visual Basic :* `Imports Microsoft.ApplicationInsights`

    *Java :* `import com.microsoft.applicationinsights.TelemetryClient;`

    *Node.js :* `var applicationInsights = require("applicationinsights");`

## <a name="get-a-telemetryclient-instance"></a>Obtenir une instance de TelemetryClient

Obtenez une instance de `TelemetryClient` (sauf en JavaScript dans les pages web) :

Pour les applications [ASP.NET Core](asp-net-core.md#how-can-i-track-telemetry-thats-not-automatically-collected) et [non HTTP/Worker pour .NET/.NET Core](worker-service.md#how-can-i-track-telemetry-thats-not-automatically-collected), nous vous recommandons d’obtenir une instance de `TelemetryClient` à partir du conteneur d’injection de dépendances comme expliqué dans la documentation correspondante.

Si vous utilisez Azure Functions v2+ ou Azure WebJobs v3+, consultez ce document : https://docs.microsoft.com/azure/azure-functions/functions-monitoring#version-2x-and-higher

*C#*

```csharp
private TelemetryClient telemetry = new TelemetryClient();
```
Si vous constatez que cette méthode contient des messages obsolètes, consultez [microsoft/ApplicationInsights-dotnet#1152](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1152) pour plus d’informations.

*Visual Basic*

```vb
Private Dim telemetry As New TelemetryClient
```

*Java*

```java
private TelemetryClient telemetry = new TelemetryClient();
``` 

*Node.JS*

```javascript
var telemetry = applicationInsights.defaultClient;
```

TelemetryClient est thread-safe.

Dans le cas des projets ASP.NET et Java, les requêtes HTTP entrantes sont automatiquement capturées. Si vous le souhaitez, vous pouvez créer des instances supplémentaires de TelemetryClient pour l’autre module de votre application. Il est par exemple possible d’en avoir une dans une classe d’intergiciels pour signaler les événements de logique métier. Vous pouvez définir des propriétés comme UserId et DeviceId pour identifier l’ordinateur. Cette information est associée à tous les événements envoyés par l'instance.

*C#*

```csharp
TelemetryClient.Context.User.Id = "...";
TelemetryClient.Context.Device.Id = "...";
```

*Java*

```java
telemetry.getContext().getUser().setId("...");
telemetry.getContext().getDevice().setId("...");
```

Dans les projets Node.js, vous pouvez utiliser `new applicationInsights.TelemetryClient(instrumentationKey?)` pour créer une instance, mais cette procédure est réservée aux scénarios qui nécessitent une configuration isolée du singleton `defaultClient`.

## <a name="trackevent"></a>TrackEvent

Dans Application Insights, un *événement personnalisé* est un point de données que vous pouvez afficher dans [Metrics Explorer](../platform/metrics-charts.md) en tant que nombre agrégé et dans [Recherche de diagnostic](./diagnostic-search.md) en tant qu’occurrences individuelles. (Il n’est pas lié à des « événements » de type MVC ou autres.)

Insérez des appels `TrackEvent` dans votre code pour compter les différents événements. Par exemple, la fréquence à laquelle les utilisateurs choisissent une fonctionnalité particulière, la fréquence à laquelle ils atteignent des objectifs particuliers ou à laquelle ils commettent éventuellement des types d’erreurs particuliers.

Par exemple, dans une application de jeu, envoyez un événement chaque fois qu'un utilisateur gagne le jeu :

*JavaScript*

```javascript
appInsights.trackEvent({name:"WinGame"});
```

*C#*

```csharp
telemetry.TrackEvent("WinGame");
```

*Visual Basic*

```vb
telemetry.TrackEvent("WinGame")
```

*Java*

```java
telemetry.trackEvent("WinGame");
```

*Node.JS*

```javascript
telemetry.trackEvent({name: "WinGame"});
```

### <a name="custom-events-in-analytics"></a>Événements personnalisés dans l’analytique

La télémétrie est disponible dans la table `customEvents` dans [Application Insights - Analytique](../log-query/log-query-overview.md). Chaque ligne représente un appel à `trackEvent(..)` dans votre application.

Si un [échantillonnage](./sampling.md) est en cours, la propriété itemCount affiche une valeur supérieure à 1. Par exemple, itemCount==10 signifie que sur 10 appels à trackEvent(), le processus d’échantillonnage n’en a transmis qu’un seul. Pour obtenir un nombre correct d’événements personnalisés, vous devez donc utiliser un code similaire à `customEvents | summarize sum(itemCount)`.

## <a name="getmetric"></a>GetMetric

Pour découvrir comment utiliser efficacement l’appel GetMetric() afin de capturer des métriques pré-agrégées localement pour les applications .NET et .NET Core, consultez la documentation de [GetMetric](./get-metric.md).

## <a name="trackmetric"></a>TrackMetric

> [!NOTE]
> La méthode Microsoft.ApplicationInsights.TelemetryClient.TrackMetric n’est pas la méthode recommandée pour l’envoi de mesures. Les métriques doivent toujours être pré-agrégés au cours d’une période spécifique avant d’être envoyés. Utilisez l’une des surcharges GetMetric(..) afin d’obtenir un objet de métrique pour accéder aux fonctionnalités de pré-agrégation du Kit de développement logiciel (SDK). Si vous implémentez votre propre logique de pré-agrégation, vous pouvez utiliser la méthode TrackMetric() pour envoyer les agrégats résultants. Si votre application requiert l’envoi systématique d’un élément de télémétrie distinct sans agrégation au fil du temps, il s’agit probablement d’un cas d’usage de télémétrie d’événement ; consultez la méthode TelemetryClient.TrackEvent (Microsoft.Applicationlnsights.DataContracts.EventTelemetry).

Application Insight peut représenter des mesures qui ne sont pas associées à des événements particuliers. Par exemple, vous pouvez analyser la longueur d’une file d'attente à des intervalles réguliers. Avec les mesures, les mesures individuelles sont moins intéressantes que les variations et tendances, ainsi les graphiques statistiques sont utiles.

Pour pouvoir envoyer des métriques à Application Insights, vous pouvez utiliser l’API `TrackMetric(..)`. Il existe deux façons d’envoyer une métrique :

* Valeur unique. Chaque fois que vous effectuez une mesure dans votre application, vous envoyez la valeur correspondante à Application Insights. Par exemple, supposons que vous avez une mesure décrivant le nombre d’éléments dans un conteneur. Pendant une période donnée, vous placez d’abord trois éléments dans le conteneur puis vous en supprimez deux. En conséquence, vous appelez `TrackMetric` à deux reprises : d’abord en passant la valeur `3` puis la valeur `-2`. Application Insights stocke les deux valeurs pour votre compte.

* Agrégation. Quand vous travaillez avec des métriques, chaque mesure individuelle est rarement intéressante. Au lieu de cela, un récapitulatif de ce qui s’est passé au cours d’une période donnée est important. Un tel récapitulatif est appelé _agrégation_. Dans l’exemple ci-dessus, la somme totale des métriques pour cette période est `1` et le nombre de valeurs des métriques est `2`. Quand vous utilisez l’approche par agrégation, vous appelez `TrackMetric` une seule fois par période et vous envoyez les valeurs agrégées. C’est l’approche recommandée, car elle peut réduire considérablement le coût et les problèmes de performances en envoyant moins de points de données à Application Insights, tout en collectant néanmoins toutes les informations pertinentes.

### <a name="examples"></a>Exemples

#### <a name="single-values"></a>Valeurs uniques

Pour envoyer une seule valeur métrique :

*JavaScript*

 ```javascript
appInsights.trackMetric("queueLength", 42.0);
 ```

*C#*

```csharp
var sample = new MetricTelemetry();
sample.Name = "metric name";
sample.Value = 42.3;
telemetryClient.TrackMetric(sample);
```

*Java*

```java
telemetry.trackMetric("queueLength", 42.0);
```

*Node.JS*

 ```javascript
telemetry.trackMetric({name: "queueLength", value: 42.0});
 ```

### <a name="custom-metrics-in-analytics"></a>Métriques personnalisées dans Analytics

La télémétrie est disponible dans la table `customMetrics` dans [Application Insights - Analytique](../log-query/log-query-overview.md). Chaque ligne représente un appel à `trackMetric(..)` dans votre application.

* `valueSum` - Il s’agit de la somme des mesures. Pour obtenir la valeur moyenne, divisez par `valueCount`.
* `valueCount` : le nombre de mesures qui ont été agrégées dans cet appel de `trackMetric(..)`.

## <a name="page-views"></a>Affichages de page

Dans un périphérique ou une application de page web, la télémétrie d'affichage de page est envoyée par défaut lorsque chaque écran ou page est chargé. Mais vous pouvez modifier cela pour suivre les affichages de page à différents moments. Par exemple, dans une application qui affiche les onglets ou les panneaux, vous pouvez effectuer le suivi d'une « page » chaque fois que l'utilisateur ouvre un nouveau panneau.

Les données d’utilisateur et de session sont envoyées en tant que propriétés avec les affichages de page, de façon à ce que les graphiques d’utilisateur et de session soient actifs s’il existe une télémétrie de l’affichage de page.

### <a name="custom-page-views"></a>Affichages de pages personnalisées

*JavaScript*

```javascript
appInsights.trackPageView("tab1");
```

*C#*

```csharp
telemetry.TrackPageView("GameReviewPage");
```

*Visual Basic*

```vb
telemetry.TrackPageView("GameReviewPage")
```

*Java*

```java
telemetry.trackPageView("GameReviewPage");
```

Si vous avez plusieurs onglets dans différentes pages HTML, vous pouvez aussi spécifier l'URL :

```javascript
appInsights.trackPageView("tab1", "http://fabrikam.com/page1.htm");
```

### <a name="timing-page-views"></a>Affichages de la page de durée

Par défaut, les heures déclarées **Temps de chargement de l’affichage de la page** sont mesurées à partir du moment où le navigateur envoie la demande, jusqu’à ce que l’événement de chargement de la page du navigateur soit appelé.

Au lieu de cela, vous pouvez :

* Définir une durée explicite dans l’appel [trackPageView](https://github.com/microsoft/ApplicationInsights-JS/blob/17ef50442f73fd02a758fbd74134933d92607ecf/legacy/API.md#trackpageview) : `appInsights.trackPageView("tab1", null, null, null, durationInMilliseconds);`.
* Utiliser les appels de minutage d’affichage de la page `startTrackPage` et `stopTrackPage`.

*JavaScript*

```javascript
// To start timing a page:
appInsights.startTrackPage("Page1");

...

// To stop timing and log the page:
appInsights.stopTrackPage("Page1", url, properties, measurements);
```

Le nom que vous utilisez comme premier paramètre associe les appels de démarrage et d’arrêt. Le nom de la page actuelle est utilisé par défaut.

Les durées de chargement de la page résultantes affichées dans Metrics Explorer sont dérivées de l’intervalle entre les appels de démarrage et d’arrêt. C’est à vous de décider quel intervalle de temps vous voulez mesurer.

### <a name="page-telemetry-in-analytics"></a>Télémétrie des pages dans Analytique

Dans [Analytique](../log-query/log-query-overview.md), deux tables affichent les données des opérations du navigateur :

* La table `pageViews` contient des données sur l’URL et le titre de la page.
* La table `browserTimings` contient des données sur les performances du client, comme le temps nécessaire pour traiter les données entrantes.

Pour trouver le temps mis par le navigateur pour traiter différentes pages :

```kusto
browserTimings
| summarize avg(networkDuration), avg(processingDuration), avg(totalDuration) by name
```

Pour découvrir la popularité de différents navigateurs :

```kusto
pageViews
| summarize count() by client_Browser
```

Pour associer des vues de pages à des appels AJAX, joindre à des dépendances :

```kusto
pageViews
| join (dependencies) on operation_Id 
```

## <a name="trackrequest"></a>TrackRequest

Le kit de développement logiciel de serveur utilise TrackRequest pour consigner les requêtes HTTP.

Vous pouvez également l'appeler vous-même si vous souhaitez simuler des requêtes dans le cas où le module du service web n’est pas en cours d'exécution.

Toutefois, le moyen recommandé d’envoyer la télémétrie de la demande est là où la demande agit comme un <a href="#operation-context">contexte d’opération</a>.

## <a name="operation-context"></a>Contexte de l’opération

Vous pouvez associer des éléments de télémétrie en les associant à un contexte d’opération. Le module de suivi de requête standard effectue cette opération pour les exceptions et les autres événements envoyés lors du traitement d’une requête HTTP. Dans [Recherche](./diagnostic-search.md) et [Analytique](../log-query/log-query-overview.md), vous pouvez trouver facilement tous les événements associés à la requête avec son ID d’opération.

Consultez [Corrélation de télémétrie dans Application Insights](./correlation.md) pour plus d’informations sur la corrélation.

Lors du suivi manuel des données de télémétrie, le moyen le plus simple pour vous assurer de la corrélation des données de télémétrie en utilisant ce modèle :

*C#*

```csharp
// Establish an operation context and associated telemetry item:
using (var operation = telemetryClient.StartOperation<RequestTelemetry>("operationName"))
{
    // Telemetry sent in here will use the same operation ID.
    ...
    telemetryClient.TrackTrace(...); // or other Track* calls
    ...

    // Set properties of containing telemetry item--for example:
    operation.Telemetry.ResponseCode = "200";

    // Optional: explicitly send telemetry item:
    telemetryClient.StopOperation(operation);

} // When operation is disposed, telemetry item is sent.
```

Outre la définition d’un contexte d’opération, `StartOperation` crée un élément de télémétrie du type que vous spécifiez. Il envoie l’élément de télémétrie lorsque vous libérez l’opération, ou si vous appelez explicitement `StopOperation`. Si vous utilisez `RequestTelemetry` comme type de télémétrie, alors sa durée est définie sur l’intervalle compris entre le début et la fin.

Les éléments de télémétrie signalés dans une étendue de l’opération deviennent « enfants » de cette opération. Les contextes de l’opération ne peuvent pas être imbriqués.

Dans Recherche, le contexte d’opération est utilisé pour créer la liste **Éléments connexes** :

![Éléments connexes](./media/api-custom-events-metrics/21.png)

Consultez [Suivi des opérations personnalisées avec le kit SDK .NET d’Application Insights](./custom-operations-tracking.md) pour plus d’informations sur le suivi des opérations personnalisées.

### <a name="requests-in-analytics"></a>Requêtes dans Analytique

Dans [Application Insights - Analytique](../log-query/log-query-overview.md), les demandes s’affichent dans la table `requests`.

Si [l’échantillonnage](./sampling.md) est en cours, la propriété itemCount affiche une valeur supérieure à 1. Par exemple, itemCount==10 signifie que sur 10 appels à trackRequest(), le processus d’échantillonnage n’en a transmis qu’un seul. Pour obtenir un nombre correct de demandes et une durée moyenne segmentée par nom des demandes, utilisez un code similaire à celui-ci :

```kusto
requests
| summarize count = sum(itemCount), avgduration = avg(duration) by name
```

## <a name="trackexception"></a>TrackException

Envoi d’exceptions à Application Insights :

* Pour [les compter](../platform/metrics-charts.md), comme une indication de la fréquence d’un problème.
* Pour [inspecter les occurrences individuelles](./diagnostic-search.md).

Les rapports incluent des arborescences des appels de procédure.

*C#*

```csharp
try
{
    ...
}
catch (Exception ex)
{
    telemetry.TrackException(ex);
}
```

*Java*

```java
try {
    ...
} catch (Exception ex) {
    telemetry.trackException(ex);
}
```

*JavaScript*

```javascript
try
{
    ...
}
catch (ex)
{
    appInsights.trackException(ex);
}
```

*Node.JS*

```javascript
try
{
    ...
}
catch (ex)
{
    telemetry.trackException({exception: ex});
}
```

Les Kits de développement logiciel (SDK) interceptent de nombreuses exceptions automatiquement, ce qui vous évite ainsi d’avoir toujours à appeler TrackException explicitement.

* ASP.NET : [écriture d’un code pour intercepter les exceptions](./asp-net-exceptions.md).
* Java EE : [les exceptions sont interceptées automatiquement](./java-get-started.md#exceptions-and-request-failures).
* JavaScript : les exceptions sont interceptées automatiquement. Si vous souhaitez désactiver la collecte automatique, ajoutez une ligne dans l'extrait de code que vous insérez dans vos pages web :

```javascript
({
    instrumentationKey: "your key",
    disableExceptionTracking: true
})
```

### <a name="exceptions-in-analytics"></a>Exceptions dans Analytique

Dans [Application Insights - Analytique](../log-query/log-query-overview.md), les exceptions s’affichent dans la table `exceptions`.

Si un [échantillonnage](./sampling.md) est en cours, la propriété `itemCount` affiche une valeur supérieure à 1. Par exemple, itemCount==10 signifie que sur 10 appels à trackException(), le processus d’échantillonnage n’en a transmis qu’un seul. Pour obtenir un nombre correct d’exceptions segmentées par type d’exception, utilisez un code similaire à celui-ci :

```kusto
exceptions
| summarize sum(itemCount) by type
```

La plupart des informations importantes sur la pile sont déjà extraites dans des variables distinctes, mais vous pouvez extraire séparément la structure `details` pour en savoir plus. Comme cette structure est dynamique, vous devez effectuer une conversion de type (transtypage) du résultat vers le type attendu. Par exemple :

```kusto
exceptions
| extend method2 = tostring(details[0].parsedStack[1].method)
```

Pour associer des exceptions aux demandes qui s’y rapportent, utilisez une jointure :

```kusto
exceptions
| join (requests) on operation_Id
```

## <a name="tracktrace"></a>TrackTrace

Utilisez TrackTrace pour diagnostiquer des problèmes en envoyant une « piste de navigation » à Application Insights. Vous pouvez envoyer des blocs de données de diagnostic et les examiner dans la [Recherche de diagnostic](./diagnostic-search.md).

Dans .NET, les [Adaptateurs de journaux d’activité](./asp-net-trace-logs.md) utilisent cette API pour envoyer des journaux d’activité tiers au portail.

En Java, pour les [Enregistreurs d’événements standard, comme Log4J ou Logback](./java-trace-logs.md), utilisez les appenders Application Insights Log4j ou Logback afin d’envoyer des journaux d’activité tiers au portail.

*C#*

```csharp
telemetry.TrackTrace(message, SeverityLevel.Warning, properties);
```

*Java*

```java
telemetry.trackTrace(message, SeverityLevel.Warning, properties);
```

*Node.JS*

```javascript
telemetry.trackTrace({
    message: message,
    severity: applicationInsights.Contracts.SeverityLevel.Warning,
    properties: properties
});
```

*JavaScript côté client/navigateur*

```javascript
trackTrace(message: string, properties?: {[string]:string}, severityLevel?: SeverityLevel)
```

Enregistrer un événement de diagnostic comme l'utilisation ou la non-utilisation d'une méthode.

 Paramètre | Description
---|---
`message` | Données de diagnostic. Peut être plus longue qu'un nom.
`properties` | Mappage de chaîne à chaîne : Données supplémentaires utilisées pour [filtrer les exceptions](#properties) dans le portail. Vide par défaut.
`severityLevel` | Valeurs prises en charge : [SeverityLevel.ts](https://github.com/microsoft/ApplicationInsights-JS/blob/17ef50442f73fd02a758fbd74134933d92607ecf/shared/AppInsightsCommon/src/Interfaces/Contracts/Generated/SeverityLevel.ts)

Vous pouvez effectuer une recherche dans le contenu du message, mais (contrairement aux valeurs de propriété), vous ne pouvez pas les filtrer.

La limite de taille sur `message` est plus importante que la limite des propriétés.
l’un des avantages de TrackTrace est que vous pouvez insérer des données relativement longues dans le message. Par exemple, vous pourriez y encoder des données POST.  

Par ailleurs, vous pouvez ajouter un niveau de gravité à votre message. Comme pour les autres données de télémétrie, vous pouvez également ajouter des valeurs de propriété qui permettent de filtrer ou rechercher différents jeux de traces. Par exemple :

*C#*

```csharp
var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
telemetry.TrackTrace("Slow database response",
                SeverityLevel.Warning,
                new Dictionary<string,string> { {"database", db.ID} });
```

*Java*

```java
Map<String, Integer> properties = new HashMap<>();
properties.put("Database", db.ID);
telemetry.trackTrace("Slow Database response", SeverityLevel.Warning, properties);
```

Dans [Recherche](./diagnostic-search.md), vous pouvez filtrer facilement tous les messages d’un niveau de gravité particulier portant sur une certaine base de données.

### <a name="traces-in-analytics"></a>Traces dans Analytique

Dans [Application Insights - Analytique](../log-query/log-query-overview.md), les appels à TrackTrace s’affichent dans la table `traces`.

Si un [échantillonnage](./sampling.md) est en cours, la propriété itemCount affiche une valeur supérieure à 1. Par exemple, itemCount==10 signifie que sur 10 appels à `trackTrace()`, le processus d’échantillonnage n’en a transmis qu’un seul. Pour obtenir un nombre correct d’appels de trace, vous devez utiliser un code similaire à `traces | summarize sum(itemCount)`.

## <a name="trackdependency"></a>TrackDependency

Utilisez l’appel à TrackDependency pour suivre les temps de réponse et les taux de réussite des appels vers un bloc de code externe. Les résultats s'affichent dans les graphiques de dépendance sur le portail. L’extrait de code ci-dessous doit être ajouté chaque fois qu’un appel de dépendance est effectué.

> [!NOTE]
> Pour .NET et .NET Core, vous pouvez également utiliser la méthode `TelemetryClient.StartOperation` (extension) qui remplit les propriétés `DependencyTelemetry` nécessaires pour la corrélation et d’autres propriétés telles que l’heure de début et la durée. Vous n’avez donc pas besoin de créer un minuteur personnalisé comme dans les exemples ci-dessous. Pour plus d’informations, consultez la [section relative au suivi des dépendances sortantes](https://docs.microsoft.com/azure/azure-monitor/app/custom-operations-tracking#outgoing-dependencies-tracking) de cet article.

*C#*

```csharp
var success = false;
var startTime = DateTime.UtcNow;
var timer = System.Diagnostics.Stopwatch.StartNew();
try
{
    success = dependency.Call();
}
catch(Exception ex) 
{
    success = false;
    telemetry.TrackException(ex);
    throw new Exception("Operation went wrong", ex);
}
finally
{
    timer.Stop();
    telemetry.TrackDependency("DependencyType", "myDependency", "myCall", startTime, timer.Elapsed, success);
}
```

*Java*

```java
boolean success = false;
Instant startTime = Instant.now();
try {
    success = dependency.call();
}
finally {
    Instant endTime = Instant.now();
    Duration delta = Duration.between(startTime, endTime);
    RemoteDependencyTelemetry dependencyTelemetry = new RemoteDependencyTelemetry("My Dependency", "myCall", delta, success);
    dependencyTelemetry.setTimeStamp(startTime);
    telemetry.trackDependency(dependencyTelemetry);
}
```

*Node.JS*

```javascript
var success = false;
var startTime = new Date().getTime();
try
{
    success = dependency.Call();
}
finally
{
    var elapsed = new Date() - startTime;
    telemetry.trackDependency({
        dependencyTypeName: "myDependency",
        name: "myCall",
        duration: elapsed,
        success: success
    });
}
```

N’oubliez pas que les kits SDK de serveur incluent un [module de dépendance](./asp-net-dependencies.md) qui détecte certains appels de dépendance et en effectue le suivi automatiquement. C’est notamment le cas des bases de données et des API REST. Vous devez installer un agent sur votre serveur pour que le module fonctionne. 

En Java, [l’Agent Java](./java-agent.md) permet de suivre automatiquement certains appels de dépendance.

Vous utiliserez cet appel si vous souhaitez effectuer le suivi des appels qui ne sont pas interceptés par le système de suivi automatisé, ou si vous ne souhaitez pas installer l'agent.

Pour désactiver le module de suivi des dépendances standard en C#, modifiez [ApplicationInsights.config](./configuration-with-applicationinsights-config.md) et supprimez la référence à `DependencyCollector.DependencyTrackingTelemetryModule`. En Java, n’installez pas l’agent Java si vous ne souhaitez pas collecter automatiquement les dépendances standards.

### <a name="dependencies-in-analytics"></a>Dépendances dans Analytique

Dans [Application Insights - Analytique](../log-query/log-query-overview.md), les appels de trackDependency s’affichent dans la table `dependencies`.

Si un [échantillonnage](./sampling.md) est en cours, la propriété itemCount affiche une valeur supérieure à 1. Par exemple, itemCount==10 signifie que sur 10 appels à trackDependency(), le processus d’échantillonnage n’en a transmis qu’un seul. Pour obtenir un nombre correct de dépendances segmentées par composant cible, utilisez un code similaire à celui-ci :

```kusto
dependencies
| summarize sum(itemCount) by target
```

Pour associer des dépendances aux demandes qui s’y rapportent, utilisez une jointure :

```kusto
dependencies
| join (requests) on operation_Id
```

## <a name="flushing-data"></a>Vidage des données

Normalement, le Kit de développement logiciel (SDK) envoie des données à intervalles réguliers (généralement 30 secondes), ou chaque fois que la mémoire tampon est saturée (en général, lorsqu’elle inclut plus de 500 éléments). Toutefois, dans certains cas vous pouvez vider la mémoire tampon - par exemple, si vous utilisez le kit SDK dans une application qui s'arrête.

*C#*

 ```csharp
telemetry.Flush();
// Allow some time for flushing before shutdown.
System.Threading.Thread.Sleep(5000);
```

*Java*

```java
telemetry.flush();
//Allow some time for flushing before shutting down
Thread.sleep(5000);
```

*Node.JS*

```javascript
telemetry.flush();
```

La fonction est asynchrone pour le [canal de télémétrie du serveur](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/).

Dans l’idéal, il est recommandé d’utiliser la méthode flush() dans l’activité d’arrêt de l’application.

## <a name="authenticated-users"></a>Utilisateurs authentifiés

Dans une application web, les utilisateurs sont [identifiés par des cookies](./usage-segmentation.md#the-users-sessions-and-events-segmentation-tool) (par défaut). Un utilisateur peut être compté plusieurs fois s’il accède à votre application à partir d’un autre ordinateur ou navigateur, ou s’il supprime des cookies.

Mais si les utilisateurs se connectent à votre application, vous pouvez obtenir un nombre plus précis en définissant l’ID de l’utilisateur authentifié dans le code du navigateur :

*JavaScript*

```javascript
// Called when my app has identified the user.
function Authenticated(signInId) {
    var validatedId = signInId.replace(/[,;=| ]+/g, "_");
    appInsights.setAuthenticatedUserContext(validatedId);
    ...
}
```

Dans une application MVC Web ASP.NET, par exemple :

*Razor*

```cshtml
@if (Request.IsAuthenticated)
{
    <script>
        appInsights.setAuthenticatedUserContext("@User.Identity.Name
            .Replace("\\", "\\\\")"
            .replace(/[,;=| ]+/g, "_"));
    </script>
}
```

Il n’est pas nécessaire d’utiliser le nom de connexion réel de l’utilisateur. Il doit uniquement s’agir d’un ID unique pour cet utilisateur. Il ne doit pas inclure d'espaces ni l'un des caractères suivants : `,;=|`.

L’ID d’utilisateur est également défini dans un cookie de session et envoyé au serveur. Si le kit SDK de serveur est installé, l’ID d’utilisateur authentifié est envoyé dans le cadre des propriétés de contexte de télémétrie client et serveur. Vous pouvez ensuite filtrer et rechercher dessus.

Si votre application regroupe les utilisateurs par comptes, vous pouvez également fournir un identificateur pour ce compte (avec les mêmes restrictions de caractères).

```javascript
appInsights.setAuthenticatedUserContext(validatedId, accountId);
```

Dans [Metrics Explorer](../platform/metrics-charts.md), vous pouvez créer un graphique qui compte les **Utilisateurs authentifiés** et les **Comptes d’utilisateur**.

Vous pouvez également [rechercher](./diagnostic-search.md) les points de données client avec des comptes et des noms d’utilisateur spécifiques.

## <a name="filtering-searching-and-segmenting-your-data-by-using-properties"></a><a name="properties"></a>Filtrage, recherche et segmentation de vos données à l’aide des propriétés

Vous pouvez associer des propriétés et des mesures à vos événements (et également à des mesures, des affichages de page, des exceptions et d'autres données de télémétrie).

*propriétés* sont des valeurs de chaîne que vous pouvez utiliser pour filtrer votre télémétrie dans les rapports d'utilisation. Par exemple, si votre application fournit plusieurs jeux, vous pouvez attacher le nom du jeu à chaque événement pour vous permettre de savoir quels sont les jeux les plus populaires.

Il existe une limite de 8192 sur la longueur de chaîne. (Si vous souhaitez envoyer d’importants blocs de données, utilisez le paramètre de message de TrackTrace.)

*mesures* sont des valeurs numériques qui peuvent être représentées sous forme graphique. Par exemple, observez s'il existe une augmentation progressive des scores atteints par vos joueurs. Les graphes peuvent être segmentés par les propriétés envoyées avec l'événement pour vous permettre d’obtenir des graphes distincts ou empilés pour différents jeux.

Pour que les valeurs de mesure s’affichent correctement, elles doivent être supérieures ou égales à 0.

Il existe certaines [limites au nombre de propriétés, de valeurs de propriété et de mesures](#limits) que vous pouvez utiliser.

*JavaScript*

```javascript
appInsights.trackEvent
    ("WinGame",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );

appInsights.trackPageView
    ("page name", "http://fabrikam.com/pageurl.html",
        // String properties:
        {Game: currentGame.name, Difficulty: currentGame.difficulty},
        // Numeric metrics:
        {Score: currentGame.score, Opponents: currentGame.opponentCount}
        );
```

*C#*

```csharp
// Set up some properties and metrics:
var properties = new Dictionary <string, string>
    {{"game", currentGame.Name}, {"difficulty", currentGame.Difficulty}};
var metrics = new Dictionary <string, double>
    {{"Score", currentGame.Score}, {"Opponents", currentGame.OpponentCount}};

// Send the event:
telemetry.TrackEvent("WinGame", properties, metrics);
```

*Node.JS*

```javascript
// Set up some properties and metrics:
var properties = {"game": currentGame.Name, "difficulty": currentGame.Difficulty};
var metrics = {"Score": currentGame.Score, "Opponents": currentGame.OpponentCount};

// Send the event:
telemetry.trackEvent({name: "WinGame", properties: properties, measurements: metrics});
```

*Visual Basic*

```vb
' Set up some properties:
Dim properties = New Dictionary (Of String, String)
properties.Add("game", currentGame.Name)
properties.Add("difficulty", currentGame.Difficulty)

Dim metrics = New Dictionary (Of String, Double)
metrics.Add("Score", currentGame.Score)
metrics.Add("Opponents", currentGame.OpponentCount)

' Send the event:
telemetry.TrackEvent("WinGame", properties, metrics)
```

*Java*

```java
Map<String, String> properties = new HashMap<String, String>();
properties.put("game", currentGame.getName());
properties.put("difficulty", currentGame.getDifficulty());

Map<String, Double> metrics = new HashMap<String, Double>();
metrics.put("Score", currentGame.getScore());
metrics.put("Opponents", currentGame.getOpponentCount());

telemetry.trackEvent("WinGame", properties, metrics);
```

> [!NOTE]
> Veillez à ne pas journaliser des informations personnelles dans les propriétés.
>
>

### <a name="alternative-way-to-set-properties-and-metrics"></a>Autre façon de définir des propriétés et des mesures

Si cela est plus pratique, vous pouvez collecter les paramètres d'un événement dans un objet séparé :

```csharp
var event = new EventTelemetry();

event.Name = "WinGame";
event.Metrics["processingTime"] = stopwatch.Elapsed.TotalMilliseconds;
event.Properties["game"] = currentGame.Name;
event.Properties["difficulty"] = currentGame.Difficulty;
event.Metrics["Score"] = currentGame.Score;
event.Metrics["Opponents"] = currentGame.Opponents.Length;

telemetry.TrackEvent(event);
```

> [!WARNING]
> Ne réutilisez pas la même instance d’élément de télémétrie (`event` dans cet exemple) pour appeler Track*() plusieurs fois. Cela peut provoquer un envoi de données de télémétrie configurées de façon incorrecte.
>
>

### <a name="custom-measurements-and-properties-in-analytics"></a>Mesures et propriétés personnalisées dans Analytique

Dans [Analytique](../log-query/log-query-overview.md), les mesures et les propriétés personnalisées s’affichent dans les attributs `customMeasurements` et `customDimensions` de chaque enregistrement de télémétrie.

Par exemple, si vous avez ajouté une propriété nommée « game » à votre télémétrie des demandes, cette requête compte les occurrences des différentes valeurs de « game » et affiche la moyenne de la métrique personnalisée « score » :

```kusto
requests
| summarize sum(itemCount), avg(todouble(customMeasurements.score)) by tostring(customDimensions.game)
```

Notez que :

* Quand vous extrayez une valeur du JSON la customDimensions ou customMeasurements, elle est de type dynamique et par conséquent, vous devez la transtyper en `tostring` ou `todouble`.
* Pour tenir compte de la possibilité [d’échantillonnage](./sampling.md), vous devez utiliser `sum(itemCount)`, et non pas `count()`.

## <a name="timing-events"></a><a name="timed"></a> Événements de durée

Vous avez parfois besoin d’obtenir une représentation graphique de la durée nécessaire à la réalisation d’une action. Par exemple, vous souhaitez savoir de combien de temps les utilisateurs ont besoin pour évaluer leurs choix dans un jeu. Vous pouvez utiliser le paramètre de mesure pour cela.

*C#*

```csharp
var stopwatch = System.Diagnostics.Stopwatch.StartNew();

// ... perform the timed action ...

stopwatch.Stop();

var metrics = new Dictionary <string, double>
    {{"processingTime", stopwatch.Elapsed.TotalMilliseconds}};

// Set up some properties:
var properties = new Dictionary <string, string>
    {{"signalSource", currentSignalSource.Name}};

// Send the event:
telemetry.TrackEvent("SignalProcessed", properties, metrics);
```

*Java*

```java
long startTime = System.currentTimeMillis();

// Perform timed action

long endTime = System.currentTimeMillis();
Map<String, Double> metrics = new HashMap<>();
metrics.put("ProcessingTime", (double)endTime-startTime);

// Setup some properties
Map<String, String> properties = new HashMap<>();
properties.put("signalSource", currentSignalSource.getName());

// Send the event
telemetry.trackEvent("SignalProcessed", properties, metrics);
```

## <a name="default-properties-for-custom-telemetry"></a><a name="defaults"></a>Propriétés par défaut pour la télémétrie personnalisée

Si vous souhaitez définir des valeurs de propriété par défaut pour certains des événements personnalisés que vous écrivez, vous pouvez les définir dans une instance de TelemetryClient. Ils sont associés à chaque élément de télémétrie envoyée à partir de ce client.

*C#*

```csharp
using Microsoft.ApplicationInsights.DataContracts;

var gameTelemetry = new TelemetryClient();
gameTelemetry.Context.GlobalProperties["Game"] = currentGame.Name;
// Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame");
```

*Visual Basic*

```vb
Dim gameTelemetry = New TelemetryClient()
gameTelemetry.Context.GlobalProperties("Game") = currentGame.Name
' Now all telemetry will automatically be sent with the context property:
gameTelemetry.TrackEvent("WinGame")
```

*Java*

```java
import com.microsoft.applicationinsights.TelemetryClient;
import com.microsoft.applicationinsights.TelemetryContext;
...


TelemetryClient gameTelemetry = new TelemetryClient();
TelemetryContext context = gameTelemetry.getContext();
context.getProperties().put("Game", currentGame.Name);

gameTelemetry.TrackEvent("WinGame");
```

*Node.JS*

```javascript
var gameTelemetry = new applicationInsights.TelemetryClient();
gameTelemetry.commonProperties["Game"] = currentGame.Name;

gameTelemetry.TrackEvent({name: "WinGame"});
```

Les appels de télémétrie individuels peuvent remplacer les valeurs par défaut dans leurs dictionnaires de propriété.

*Pour les clients Web JavaScript*, utilisez des initialiseurs de télémétrie JavaScript.

*Pour ajouter des propriétés à toutes les données de télémétrie*, notamment les données des modules de collecte standard, [implémentez `ITelemetryInitializer`](./api-filtering-sampling.md#add-properties).

## <a name="sampling-filtering-and-processing-telemetry"></a>Échantillonnage, filtrage et traitement de la télémétrie

Vous pouvez écrire du code pour traiter votre télémétrie avant de l’envoyer à partir du Kit de développement logiciel (SDK). Le traitement inclut les données envoyées par les modules de télémétrie standard, telles que la collection de requêtes HTTP et la collection de dépendances.

[Ajoutez des propriétés](./api-filtering-sampling.md#add-properties) à la télémétrie en implémentant `ITelemetryInitializer`. Par exemple, vous pouvez ajouter des numéros de version ou des valeurs calculées à partir d'autres propriétés.

[Le filtrage](./api-filtering-sampling.md#filtering) peut modifier ou abandonner des données de télémétrie avant leur envoi depuis le SDK en implémentant `ITelemetryProcessor`. Vous contrôlez ce qui est envoyé ou rejeté, mais vous devez prendre en compte l’impact sur vos mesures. Suivant la façon dont vous ignorez les éléments, vous risquez de ne plus pouvoir naviguer entre des éléments connexes.

[L’échantillonnage](./api-filtering-sampling.md) est une solution intégrée pour réduire le volume des données envoyées à partir de votre application vers le portail. Cela n’affecte pas les mesures affichées. Et il n’affecte pas votre capacité à diagnostiquer les problèmes en navigant entre des éléments connexes, tels que les exceptions, les requêtes et les affichages de page.

[Plus d’informations](./api-filtering-sampling.md)

## <a name="disabling-telemetry"></a>Désactivation de la télémétrie

Pour *arrêter et démarrer dynamiquement* la collecte et la transmission de la télémétrie :

*C#*

```csharp
using  Microsoft.ApplicationInsights.Extensibility;

TelemetryConfiguration.Active.DisableTelemetry = true;
```

*Java*

```java
telemetry.getConfiguration().setTrackingDisabled(true);
```

Pour *désactiver les collecteurs standard sélectionnés* (par exemple, les compteurs de performances, les requêtes HTTP ou les dépendances), supprimez ou commentez les lignes correspondantes dans [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). Par exemple, vous pouvez faire cela si vous souhaitez envoyer vos propres données TrackRequest.

*Node.JS*

```javascript
telemetry.config.disableAppInsights = true;
```

Pour *désactiver les collecteurs standard sélectionnés* (par exemple, les compteurs de performances, les requêtes HTTP ou les dépendances) lors de l’initialisation, enchaînez plusieurs méthodes de configuration dans votre code d’initialisation SDK.

```javascript
applicationInsights.setup()
    .setAutoCollectRequests(false)
    .setAutoCollectPerformance(false)
    .setAutoCollectExceptions(false)
    .setAutoCollectDependencies(false)
    .setAutoCollectConsole(false)
    .start();
```

Pour désactiver ces collecteurs après l’initialisation, utilisez l’objet de configuration : `applicationInsights.Configuration.setAutoCollectRequests(false)`

## <a name="developer-mode"></a><a name="debug"></a>Mode Développeur :

Pendant le débogage, il est utile d'avoir votre télémétrie envoyée par le pipeline afin que vous puissiez voir immédiatement les résultats. Vous obtenez également des messages supplémentaires qui vous permettent de suivre tout problème relatif à la télémétrie. Désactivez-les lors de la production, car ils peuvent ralentir votre application.

*C#*

```csharp
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = true;
```

*Visual Basic*

```vb
TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = True
```

*Node.JS*

Pour Node.js, vous pouvez activer le mode développeur en activant la journalisation interne via `setInternalLogging` et en définissant le paramètre `maxBatchSize` sur 0, ce qui entraîne l’envoi de votre télémétrie dès qu’elle est collectée.

```js
applicationInsights.setup("ikey")
  .setInternalLogging(true, true)
  .start()
applicationInsights.defaultClient.config.maxBatchSize = 0;
```

## <a name="setting-the-instrumentation-key-for-selected-custom-telemetry"></a><a name="ikey"></a> Définition de la touche d’instrumentation pour la télémétrie personnalisée sélectionnée

*C#*

```csharp
var telemetry = new TelemetryClient();
telemetry.InstrumentationKey = "---my key---";
// ...
```

## <a name="dynamic-instrumentation-key"></a><a name="dynamic-ikey"></a> Clé d'instrumentation dynamique

Pour éviter de mélanger la télémétrie fournie par les environnements de développement, de test et de production, vous pouvez [créer des ressources Application Insights distinctes](./create-new-resource.md) et modifier leurs clés en fonction de l’environnement.

Au lieu de récupérer la clé d'instrumentation à partir du fichier de configuration, vous pouvez la définir dans votre code. Définissez la clé dans une méthode d'initialisation, par exemple global.aspx.cs dans un service ASP.NET :

*C#*

```csharp
protected void Application_Start()
{
    Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey =
        // - for example -
        WebConfigurationManager.Settings["ikey"];
    ...
}
```

*JavaScript*

```javascript
appInsights.config.instrumentationKey = myKey;
```

Dans les pages web, vous pouvez la définir depuis l'état du serveur web au lieu de la coder littéralement dans le script. Par exemple, dans une page web générée dans une application ASP.NET :

*JavaScript dans Razor*

```cshtml
<script type="text/javascript">
// Standard Application Insights webpage script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
    // Generate from server property:
    @Microsoft.ApplicationInsights.Extensibility.
        TelemetryConfiguration.Active.InstrumentationKey;
}) // ...
```

```java
    String instrumentationKey = "00000000-0000-0000-0000-000000000000";

    if (instrumentationKey != null)
    {
        TelemetryConfiguration.getActive().setInstrumentationKey(instrumentationKey);
    }
```

## <a name="telemetrycontext"></a>TelemetryContext

TelemetryClient a une propriété de contexte contenant les valeurs qui sont envoyées avec toutes les données de télémétrie. Elles sont normalement définies par les modules de télémétrie standard, mais vous pouvez également les définir vous-même. Par exemple :

```csharp
telemetry.Context.Operation.Name = "MyOperationName";
```

Si vous définissez une de ces valeurs vous-même, supprimez la ligne appropriée dans [ApplicationInsights.config](./configuration-with-applicationinsights-config.md), pour ne pas mélanger vos valeurs et les valeurs standard.

* **Composant** : l’application et sa version.
* **Appareil** : données concernant l’appareil sur lequel l’application s’exécute. (dans les applications web, il s’agit du serveur ou de l’appareil client depuis lequel la télémétrie est envoyée.)
* **InstrumentationKey** : la ressource Application Insights dans Azure dans laquelle la télémétrie s’affiche. Elle est généralement récupérée dans ApplicationInsights.config.
* **Emplacement** : emplacement géographique de l’appareil.
* **Opération** : dans les applications web, il s’agit de la requête HTTP actuelle. Dans d'autres types d'application, vous pouvez définir celle-ci sur les événements regroupés.
  * **ID** : une valeur générée qui met en relation différents événements de manière à ce que vous trouviez les « Éléments associés » quand vous inspectez un événement dans la Recherche de diagnostic.
  * **Name** : un identificateur, généralement l’URL de la requête HTTP.
  * **SyntheticSource** : si elle est non nulle ou vide, cette chaîne indique que la source de la requête a été identifiée en tant que robot ou test web. Par défaut, elle est exclue des calculs dans Metrics Explorer.
* **Propriétés** : ce sont les propriétés qui sont envoyées avec toutes les données de télémétrie. Elles peuvent être remplacées dans les appels Track* individuels.
* **Session**: la session utilisateur. L'ID est définie sur une valeur générée qui est modifiée lorsque l'utilisateur n'a pas été actif pendant un certain temps.
* **Utilisateur** : informations utilisateur.

## <a name="limits"></a>limites

[!INCLUDE [application-insights-limits](../../../includes/application-insights-limits.md)]

Pour éviter d'atteindre la limite de débit de données, utilisez [l’échantillonnage](./sampling.md).

Pour déterminer la durée de conservation des données, consultez [Rétention des données et confidentialité](./data-retention-privacy.md).

## <a name="reference-docs"></a>Documents de référence

* [Référence ASP.NET](/dotnet/api/overview/azure/insights?view=azure-dotnet)
* [Référence Java](/java/api/overview/azure/appinsights?view=azure-java-stable/)
* [Référence JavaScript](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md)

## <a name="sdk-code"></a>Code de Kit de développement logiciel (SDK)

* [Kit de développement logiciel (SDK) principal ASP.NET](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [ASP.NET](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [Packages Windows Server](https://github.com/Microsoft/ApplicationInsights-dotnet)
* [Kit SDK Java](https://github.com/Microsoft/ApplicationInsights-Java)
* [Kit de développement logiciel (SDK) Node.js](https://github.com/Microsoft/ApplicationInsights-Node.js)
* [Kit de développement logiciel (SDK) JavaScript](https://github.com/Microsoft/ApplicationInsights-JS)

## <a name="questions"></a>Questions

* *Quelles exceptions peuvent être lancées par les appels Track_() ?*

    Aucun. Vous n’avez pas besoin de les inclure dans des clauses try-catch. Si le Kit de développement logiciel (SDK) rencontre des problèmes, il enregistrera des messages dans la sortie de la console de débogage et, si les messages aboutissent, dans la recherche de diagnostic.
* *Existe-t-il une API REST pour obtenir des données à partir du portail ?*

    Oui, [l’API d’accès aux données](https://dev.applicationinsights.io/). Les autres méthodes d’extraction des données sont [l’exportation d’Analytics vers Power BI](./export-power-bi.md) et [l’exportation continue](./export-telemetry.md).

## <a name="next-steps"></a><a name="next"></a>Étapes suivantes

* [Recherche d’événements et de journaux d’activité](./diagnostic-search.md)
* [Dépannage](../faq.md)

