---
title: Métriques Azure Application Insights reposant sur un journal | Microsoft Docs
description: Cet article énumère les métriques Azure Application Insights contenant des agrégations et des dimensions prises en charge. Les détails sur les métriques reposant sur un journal incluent les instructions de requête Kusto sous-jacentes.
author: vgorbenko
services: azure-monitor
ms.topic: reference
ms.date: 07/03/2019
ms.author: vitalyg
ms.subservice: application-insights
ms.openlocfilehash: 5c61287475eb82241aa5c9e1d1649e8b20e3b28c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185944"
---
# <a name="application-insights-log-based-metrics"></a>Métriques reposant sur un journal d’Application Insights

Les métriques Application Insights reposant sur un journal vous permettent d'analyser l'intégrité de vos applications surveillées, de créer de puissants tableaux de bord, et de configurer des alertes. Il existe deux types de métriques :

* [Les métriques reposant sur le journal](../app/pre-aggregated-metrics-log-metrics.md#log-based-metrics) en arrière-plan sont converties en [requêtes Kusto](/azure/kusto/query/) à partir d'événements stockés.
* [Les métriques standard](../app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics) sont stockées sous forme de séries chronologiques pré-agrégées.

Comme les *métriques standard* sont pré-agrégées pendant la collecte, elles offrent de meilleures performances lors des requêtes. Elles constituent donc un meilleur choix pour les tableaux de bord et les alertes en temps réel. Les *métriques reposant sur un journal* contiennent plus de dimensions, ce qui en fait une meilleure option pour l'analyse des données et les diagnostics ad hoc. Utilisez le [sélecteur d'espace de noms](metrics-getting-started.md#create-your-first-metric-chart) pour basculer entre les métriques reposant sur un journal et les métriques standard dans [Metrics Explorer](metrics-getting-started.md).

## <a name="interpret-and-use-queries-from-this-article"></a>Interpréter et utiliser les requêtes de cet article

Cet article énumère les métriques contenant des agrégations et des dimensions prises en charge. Les détails sur les métriques reposant sur un journal incluent les instructions de requête Kusto sous-jacentes. Par commodité, chaque requête utilise des valeurs par défaut pour la granularité temporelle, le type de graphique et parfois la dimension de fractionnement, ce qui simplifie l'utilisation de la requête dans Log Analytics sans aucune modification nécessaire.

Lorsque vous tracez la même métrique dans [Metrics Explorer](metrics-getting-started.md), il n'y a aucune valeur par défaut : la requête est ajustée dynamiquement en fonction des paramètres de votre graphique :

- La **plage de temps** sélectionnée est convertie en une clause *where timestamp...* qui sélectionne uniquement les événements de la plage de temps sélectionnée. Par exemple, dans un graphique affichant les données des dernières 24 heures, la requête inclut *| where timestamp > ago(24 h)* .

- La **granularité temporelle** sélectionnée est placée dans la clause *summarize ... by bin(timestamp, [time grain])* finale.

- Toutes les dimensions **Filtre** sélectionnées sont converties en clauses *where* supplémentaires.

- La dimension **Split chart** (Fractionner le graphique) sélectionnée est convertie en une propriété de résumé supplémentaire. Par exemple, si vous fractionnez votre graphique par *emplacement* et que vous le tracez en utilisant une granularité temporelle de 5 minutes, la clause *summarize* est résumée par *... by bin(timestamp, 5 m), location*.

> [!NOTE]
> Si vous découvrez le langage de requête Kusto, commencez par copier et coller les instructions Kusto dans le volet de requête Log Analytics sans y apporter de modification. Cliquez sur **Exécuter** pour afficher le graphique de base. À mesure que vous assimilez la syntaxe du langage de requête, vous pouvez commencer à apporter de petites modifications et voir l'impact de votre changement. Explorer vos propres données est un excellent moyen de tirer pleinement parti de [Log Analytics](../log-query/log-analytics-tutorial.md)et [Azure Monitor](../overview.md).

## <a name="availability-metrics"></a>Métriques de disponibilité

Les métriques de la catégorie Disponibilité vous permettent d’afficher l’intégrité de votre application web telle qu'elle apparaît dans le monde entier. [Configurer les tests de disponibilité](../app/monitor-web-app-availability.md) pour commencer à utiliser les métriques de cette catégorie.

### <a name="availability-availabilityresultsavailabilitypercentage"></a>Disponibilité (availabilityResults/availabilityPercentage)
La métrique *Disponibilité* indique le pourcentage des tests web qui n'ont détecté aucun problème. La valeur la plus basse possible est 0, qui indique que tous les tests web ont échoué. La valeur 100 signifie que tous les tests en ligne ont satisfait aux critères de validation.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|---|---|---|
|Pourcentage|Average|Emplacement d'exécution, Nom du test|

```Kusto
availabilityResults 
| summarize sum(todouble(success == 1) * 100) / count() by bin(timestamp, 5m), location
| render timechart
```

### <a name="availability-test-duration-availabilityresultsduration"></a>Durée du test de disponibilité (availabilityResults/duration)

La métrique *Durée du test de disponibilité* indique le temps nécessaire pour exécuter le test web. Pour les [tests web à plusieurs étapes](../app/availability-multistep.md), la métrique reflète le temps d'exécution total de toutes les étapes.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|---|---|---|
|Millisecondes|Moyenne, Min, Max|Emplacement d'exécution, Nom du test, Résultat du test

```Kusto
availabilityResults
| where notempty(duration)
| extend availabilityResult_duration = iif(itemType == 'availabilityResult', duration, todouble(''))
| summarize sum(availabilityResult_duration)/sum(itemCount) by bin(timestamp, 5m), location
| render timechart
```

### <a name="availability-tests-availabilityresultscount"></a>Test de disponibilité (availabilityResults/count)

La métrique *Tests de disponibilité* reflète le nombre de tests web exécutés par Azure Monitor.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|---|---|---|
|Count|Count|Emplacement d'exécution, Nom du test, Résultat du test|

```Kusto
availabilityResults
| summarize sum(itemCount) by bin(timestamp, 5m)
| render timechart
```

## <a name="browser-metrics"></a>Métriques du navigateur

Les métriques du navigateur sont collectées par le Kit de développement logiciel (SDK) JavaScript Application Insights à partir des navigateurs réels des utilisateurs finaux. Elles vous donnent un excellent aperçu de l'expérience de vos utilisateurs avec votre application web. Les métriques du navigateur ne sont généralement pas échantillonnées, ce qui signifie qu’elles offrent une plus grande précision des données d'utilisation par rapport aux métriques côté serveur qui peuvent être biaisées par l'échantillonnage.

> [!NOTE]
> Pour collecter les métriques du navigateur, votre application doit être instrumentée avec le [kit SDK JavaScript Application Insights](../app/javascript.md).

### <a name="browser-page-load-time-browsertimingstotalduration"></a>Temps de chargement de la page de navigateur (browserTimings/totalDuration)

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|None|

```Kusto
browserTimings
| where notempty(totalDuration)
| extend _sum = totalDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="client-processing-time-browsertimingprocessingduration"></a>Temps de traitement du client (browserTiming/processingDuration)

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|None|

```Kusto
browserTimings
| where notempty(processingDuration)
| extend _sum = processingDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum)/sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="page-load-network-connect-time-browsertimingsnetworkduration"></a>Temps de connexion au réseau pour le chargement de page (browserTimings/networkDuration)

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|None|

```Kusto
browserTimings
| where notempty(networkDuration)
| extend _sum = networkDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="receiving-response-time-browsertimingsreceiveduration"></a>Temps de réception de réponse (browserTimings/receiveDuration)

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|None|

```Kusto
browserTimings
| where notempty(receiveDuration)
| extend _sum = receiveDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="send-request-time-browsertimingssendduration"></a>Temps d’envoi de la requête (browserTimings/sendDuration)

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|None|

```Kusto
browserTimings
| where notempty(sendDuration)
| extend _sum = sendDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

## <a name="failure-metrics"></a>Métriques d’échec

Les métriques de la catégorie **Échecs** montrent les problèmes liés au traitement des requêtes, aux appels de dépendances et aux exceptions levées.

### <a name="browser-exceptions-exceptionsbrowser"></a>Exceptions du navigateur (exceptions/browser)

Cette métrique reflète le nombre d'exceptions levées à partir de votre code d'application exécuté dans le navigateur. Seules les exceptions suivies avec un appel à l’API Application Insights ```trackException()``` sont incluses dans la métrique.

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|Notes|
|---|---|---|---|
|Count|Count|None|La version reposant sur un journal utilise l’agrégation **Sum**|

```Kusto
exceptions
| where notempty(client_Browser)
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="dependency-call-failures-dependenciesfailed"></a>Échecs d'appel de dépendance (dependencies/failed)

Nombre d’appels de dépendance ayant échoué.

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|Notes|
|---|---|---|---|
|Count|Count|None|La version reposant sur un journal utilise l’agrégation **Sum**|

```Kusto
dependencies
| where success == 'False'
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="exceptions-exceptionscount"></a>Exceptions (exceptions/count)

Chaque fois que vous consignez une exception dans Application Insights, un appel à la [méthode trackException()](../app/api-custom-events-metrics.md#trackexception) du SDK est effectué. La métrique Exceptions indique le nombre d'exceptions consignées.

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|Notes|
|---|---|---|---|
|Count|Count|Nom du rôle cloud, Instance de rôle cloud, Type d’appareil|La version reposant sur un journal utilise l’agrégation **Sum**|

```Kusto
exceptions
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="failed-requests-requestsfailed"></a>Demandes ayant échoué (requests/failed)

Nombre de requêtes de serveur suivies qui ont été marquées comme ayant *échoué*. Par défaut, le SDK Application Insights marque automatiquement chaque requête de serveur qui renvoie le code de réponse HTTP 5xx ou 4xx comme une requête ayant échoué. Vous pouvez personnaliser cette logique en modifiant la propriété *success* (réussite) de l'élément de télémétrie de requête dans un [initialisateur de télémétrie personnalisé](../app/api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|Notes|
|---|---|---|---|
|Count|Count|Instance de rôle cloud, Nom du rôle cloud, Trafic réel ou synthétique, Performances de la requête, Code de réponse|La version reposant sur un journal utilise l’agrégation **Sum**|

```Kusto
requests
| where success == 'False'
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="server-exceptions-exceptionsserver"></a>Exceptions du serveur (exceptions/server)

Cette métrique indique le nombre d'exceptions du serveur.

|Unité de mesure|Agrégations prises en charge|Dimensions pré-agrégées|Notes|
|---|---|---|---|
|Count|Count|Nom du rôle cloud, Instance de rôle cloud|La version reposant sur un journal utilise l’agrégation **Sum**|

```Kusto
exceptions
| where isempty(client_Browser)
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

## <a name="performance-counters"></a>Compteurs de performance

Utilisez les métriques de la catégorie **Compteurs de performance** pour accéder aux [compteurs de performance système collectés par Application Insights](../app/performance-counters.md).

### <a name="available-memory-performancecountersavailablememory"></a>Mémoire disponible (performanceCounters/availableMemory)

```Kusto
performanceCounters
| where ((category == "Memory" and counter == "Available Bytes") or name == "availableMemory")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="exception-rate-performancecountersexceptionrate"></a>Taux d’exceptions (performanceCounters/exceptionRate)

```Kusto
performanceCounters
| where ((category == ".NET CLR Exceptions" and counter == "# of Exceps Thrown / sec") or name == "exceptionRate")
| extend performanceCounter_value = iif(itemType == 'performanceCounter',value,todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-request-execution-time-performancecountersrequestexecutiontime"></a>Durée d’exécution de la requête HTTP (performanceCounters/requestExecutionTime)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Request Execution Time") or name == "requestExecutionTime")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-request-rate-performancecountersrequestspersecond"></a>Taux de requêtes HTTP (performanceCounters/requestsPerSecond)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Requests/Sec") or name == "requestsPerSecond")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-requests-in-application-queue-performancecountersrequestsinqueue"></a>Requêtes HTTP dans la file d'attente de l'application (performanceCounters/requestsInQueue)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Requests In Application Queue") or name == "requestsInQueue")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-cpu-performancecountersprocesscpupercentage"></a>Processeur de processus (performanceCounters/processCpuPercentage)

La métrique indique la part de la capacité totale du processeur consommée par le processus qui héberge votre application surveillée.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Pourcentage|Moyenne, Min, Max|Instance de rôle cloud

```Kusto
performanceCounters
| where ((category == "Process" and counter == "% Processor Time Normalized") or name == "processCpuPercentage")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-io-rate-performancecountersprocessiobytespersecond"></a>Taux d’E/S du processus (performanceCounters/processIOBytesPerSecond)

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Octets par seconde|Moyenne, Min, Max|Instance de rôle cloud

```Kusto
performanceCounters
| where ((category == "Process" and counter == "IO Data Bytes/sec") or name == "processIOBytesPerSecond")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-private-bytes-performancecountersprocessprivatebytes"></a>Octets privés du processus (performanceCounters/processPrivateBytes)

Quantité de mémoire non partagée que le processus surveillé a allouée pour ses données.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Octets|Moyenne, Min, Max|Instance de rôle cloud

```Kusto
performanceCounters
| where ((category == "Process" and counter == "Private Bytes") or name == "processPrivateBytes")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="processor-time-performancecountersprocessorcpupercentage"></a>Temps processeur (performanceCounters/processorCpuPercentage)

Consommation d’UC de *tous* les processus s'exécutant sur l'instance de serveur surveillée.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Pourcentage|Moyenne, Min, Max|Instance de rôle cloud

>[!NOTE]
> La métrique Temps processeur n'est pas disponible pour les applications hébergées dans Azure App Services. Utilisez la métrique [Utilisation de l'UC](#process-cpu-performancecountersprocesscpupercentage) pour suivre l'utilisation d’UC des applications web hébergées dans App Services.

```Kusto
performanceCounters
| where ((category == "Processor" and counter == "% Processor Time") or name == "processorCpuPercentage")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

## <a name="server-metrics"></a>Indicateurs de performances de serveur

### <a name="dependency-calls-dependenciescount"></a>Appels de dépendance (dependencies/count)

Cette métrique est liée au nombre d'appels de dépendance.

```Kusto
dependencies
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="dependency-duration-dependenciesduration"></a>Durée de la dépendance (dependencies/duration)

Cette métrique fait référence à la durée des appels de dépendance.

```Kusto
dependencies
| where notempty(duration)
| extend dependency_duration = iif(itemType == 'dependency',duration,todouble(''))
| extend _sum = dependency_duration
| extend _count = itemCount
| extend _sum = _sum*_count
| summarize sum(_sum)/sum(_count) by bin(timestamp, 1m)
| render timechart
```

### <a name="server-requests-requestscount"></a>Requêtes serveur (requests/count)

Cette métrique reflète le nombre de requêtes de serveur entrantes reçues par votre application web.

```Kusto
requests
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="server-response-time-requestsduration"></a>Temps de réponse du serveur (requests/duration)

Cette métrique reflète le temps nécessaire aux serveurs pour traiter les demandes entrantes.

```Kusto
requests
| where notempty(duration)
| extend request_duration = iif(itemType == 'request', duration, todouble(''))
| extend _sum = request_duration
| extend _count = itemCount
| extend _sum = _sum*_count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 1m)
| render timechart
```

## <a name="usage-metrics"></a>Mesures d’utilisation

### <a name="page-view-load-time-pageviewsduration"></a>Temps de chargement de la page consultée (pageViews/duration)

Cette métrique fait référence au temps nécessaire au chargement des événements PageView.

```Kusto
pageViews
| where notempty(duration)
| extend pageView_duration = iif(itemType == 'pageView', duration, todouble(''))
| extend _sum = pageView_duration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render barchart
```

### <a name="page-views-pageviewscount"></a>Affichages de pages (pageViews/count)

Nombre d'événements PageView consignés avec l'API Application Insights TrackPageView().

```Kusto
pageViews
| summarize sum(itemCount) by bin(timestamp, 1h)
| render barchart
```

### <a name="sessions-sessionscount"></a>Sessions (sessions/count)

Cette métrique fait référence au nombre d'ID de session distincts.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(session_Id)
| summarize dcount(session_Id) by bin(timestamp, 1h)
| render barchart
```

### <a name="traces-tracescount"></a>Traces (traces/count)

Nombre d'instructions de suivi consignées avec l'appel à l’API Application Insights TrackTrace().

```Kusto
traces
| summarize sum(itemCount) by bin(timestamp, 1h)
| render barchart
```

### <a name="users-userscount"></a>Utilisateurs (users/count)

Nombre d’utilisateurs distincts qui ont accédé à votre application. L'utilisation de l'échantillonnage et du filtrage par télémétrie peut avoir une incidence importante sur la précision de cette métrique.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(user_Id)
| summarize dcount(user_Id) by bin(timestamp, 1h)
| render barchart
```

### <a name="users-authenticated-usersauthenticated"></a>Utilisateurs, Authentifié (users/authenticated)

Nombre d’utilisateurs distincts qui se sont authentifiés auprès de votre application.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(user_AuthenticatedId)
| summarize dcount(user_AuthenticatedId) by bin(timestamp, 1h)
| render barchart
```