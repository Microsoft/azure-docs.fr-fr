---
title: Métriques standard Azure Application Insights | Microsoft Docs
description: Cet article énumère les métriques Azure Application Insights contenant des agrégations et des dimensions prises en charge.
services: azure-monitor
ms.topic: reference
ms.date: 07/03/2019
ms.subservice: application-insights
ms.openlocfilehash: f59dce5a450cd745ad05ea3b53d85606706a8eaa
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97512064"
---
# <a name="application-insights-standard-metrics"></a>Métriques standard Application Insights

Comme les métriques standard sont pré-agrégées pendant la collecte, elles offrent de meilleures performances lors des requêtes. Elles constituent donc le meilleur choix pour les tableaux de bord et les alertes en temps réel.

## <a name="availability-metrics"></a>Métriques de disponibilité

Les métriques de la catégorie Disponibilité vous permettent d’afficher l’intégrité de votre application web telle qu'elle apparaît dans le monde entier. [Configurer les tests de disponibilité](../app/monitor-web-app-availability.md) pour commencer à utiliser les métriques de cette catégorie.

### <a name="availability-availabilityresultsavailabilitypercentage"></a>Disponibilité (availabilityResults/availabilityPercentage)
La métrique *Disponibilité* indique le pourcentage des tests web qui n'ont détecté aucun problème. La valeur la plus basse possible est 0, qui indique que tous les tests web ont échoué. La valeur 100 signifie que tous les tests en ligne ont satisfait aux critères de validation.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|---|---|---|
|Pourcentage|Average|`Run location`, `Test name`|


### <a name="availability-test-duration-availabilityresultsduration"></a>Durée du test de disponibilité (availabilityResults/duration)

La métrique *Durée du test de disponibilité* indique le temps nécessaire pour exécuter le test web. Pour les [tests web à plusieurs étapes](../app/availability-multistep.md), la métrique reflète le temps d'exécution total de toutes les étapes.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|---|---|---|
|Millisecondes|Moyenne, Min, Max|`Run location`, `Test name`, `Test result`

### <a name="availability-tests-availabilityresultscount"></a>Test de disponibilité (availabilityResults/count)

La métrique *Tests de disponibilité* reflète le nombre de tests web exécutés par Azure Monitor.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|---|---|---|
|Count|Count|`Run location`, `Test name`, `Test result`|


## <a name="browser-metrics"></a>Métriques du navigateur

Les métriques du navigateur sont collectées par le Kit de développement logiciel (SDK) JavaScript Application Insights à partir des navigateurs réels des utilisateurs finaux. Elles vous donnent un excellent aperçu de l'expérience de vos utilisateurs avec votre application web. Les métriques du navigateur ne sont généralement pas échantillonnées, ce qui signifie qu’elles offrent une plus grande précision des données d'utilisation par rapport aux métriques côté serveur qui peuvent être biaisées par l'échantillonnage.

> [!NOTE]
> Pour collecter les métriques du navigateur, votre application doit être instrumentée avec le [kit SDK JavaScript Application Insights](../app/javascript.md).

### <a name="browser-page-load-time-browsertimingstotalduration"></a>Temps de chargement de la page de navigateur (browserTimings/totalDuration)

|Unité de mesure|Agrégations prises en charge| Dimensions prises en charge|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|Aucun|

### <a name="client-processing-time-browsertimingprocessingduration"></a>Temps de traitement du client (browserTiming/processingDuration)

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|Aucun|

### <a name="page-load-network-connect-time-browsertimingsnetworkduration"></a>Temps de connexion au réseau pour le chargement de page (browserTimings/networkDuration)

|Unité de mesure|Agrégations prises en charge| Dimensions prises en charge|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|Aucun|

### <a name="receiving-response-time-browsertimingsreceiveduration"></a>Temps de réception de réponse (browserTimings/receiveDuration)

|Unité de mesure|Agrégations prises en charge| Dimensions prises en charge|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|Aucun|

### <a name="send-request-time-browsertimingssendduration"></a>Temps d’envoi de la requête (browserTimings/sendDuration)

|Unité de mesure|Agrégations prises en charge| Dimensions prises en charge|
|---|---|---|
|Millisecondes|Moyenne, Min, Max|Aucun|

## <a name="failure-metrics"></a>Métriques d’échec

Les métriques de la catégorie **Échecs** montrent les problèmes liés au traitement des requêtes, aux appels de dépendances et aux exceptions levées.

### <a name="browser-exceptions-exceptionsbrowser"></a>Exceptions du navigateur (exceptions/browser)

Cette métrique reflète le nombre d'exceptions levées à partir de votre code d'application exécuté dans le navigateur. Seules les exceptions suivies avec un appel à l’API Application Insights ```trackException()``` sont incluses dans la métrique.

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge|
|---|---|---|---|
| Count | Count | `Cloud role name` |

### <a name="dependency-call-failures-dependenciesfailed"></a>Échecs d'appel de dépendance (dependencies/failed)

Nombre d’appels de dépendance ayant échoué.

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
|Count|Count| `Cloud role instance`, `Cloud role name`, `Dependency performance`, `Dependency type`, `Is traffic synthetic`, `Result code`, `Rarget of dependency call`.


### <a name="exceptions-exceptionscount"></a>Exceptions (exceptions/count)

Chaque fois que vous consignez une exception dans Application Insights, un appel à la [méthode trackException()](../app/api-custom-events-metrics.md#trackexception) du SDK est effectué. La métrique Exceptions indique le nombre d'exceptions consignées.

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
|Count|Count|`Cloud role instance`, `Cloud role name`, `Device type`|

### <a name="failed-requests-requestsfailed"></a>Demandes ayant échoué (requests/failed)

Nombre de requêtes de serveur suivies qui ont été marquées comme ayant *échoué*. Par défaut, le SDK Application Insights marque automatiquement chaque requête de serveur qui renvoie le code de réponse HTTP 5xx ou 4xx comme une requête ayant échoué. Vous pouvez personnaliser cette logique en modifiant la propriété *success* (réussite) de l'élément de télémétrie de requête dans un [initialisateur de télémétrie personnalisé](../app/api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).


|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
|Count|Count|`Cloud role instance`, `Cloud role name`, `Is synthetic traffic`, `Request performance`, `Result code`|


### <a name="server-exceptions-exceptionsserver"></a>Exceptions du serveur (exceptions/server)

Cette métrique indique le nombre d'exceptions du serveur.

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
|Count|Count|`Cloud role instance`, `Cloud role name`|

## <a name="performance-counters"></a>Compteurs de performance

Utilisez les métriques de la catégorie **Compteurs de performance** pour accéder aux [compteurs de performance système collectés par Application Insights](../app/performance-counters.md).

### <a name="available-memory-performancecountersavailablememory"></a>Mémoire disponible (performanceCounters/availableMemory)

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
|Dépendant des données : Mégaoctets, Gigaoctets|Moyenne, Max, Min|`Cloud role instance`|


### <a name="exception-rate-performancecountersexceptionrate"></a>Taux d’exceptions (performanceCounters/exceptionRate)

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
| Count | Moyenne, Max, Min | `Cloud role instance` |


### <a name="http-request-execution-time-performancecountersrequestexecutiontime"></a>Durée d’exécution de la requête HTTP (performanceCounters/requestExecutionTime)

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
| Millisecondes | Moyenne, Max, Min | `Cloud role instance` |

### <a name="http-request-rate-performancecountersrequestspersecond"></a>Taux de requêtes HTTP (performanceCounters/requestsPerSecond)

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
| Demandes par seconde | Moyenne, Max, Min | `Cloud role instance` |

### <a name="http-requests-in-application-queue-performancecountersrequestsinqueue"></a>Requêtes HTTP dans la file d'attente de l'application (performanceCounters/requestsInQueue)

|Unité de mesure|Agrégations prises en charge | Dimensions prises en charge |
|---|---|---|---|
| Count | Moyenne, Max, Min | `Cloud role instance` |


### <a name="process-cpu-performancecountersprocesscpupercentage"></a>Processeur de processus (performanceCounters/processCpuPercentage)

La métrique indique la part de la capacité totale du processeur consommée par le processus qui héberge votre application surveillée.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Pourcentage|Moyenne, Max, Min| `Cloud role instance` |


### <a name="process-io-rate-performancecountersprocessiobytespersecond"></a>Taux d’E/S du processus (performanceCounters/processIOBytesPerSecond)

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Octets par seconde|Moyenne, Min, Max|`Cloud role instance` |



### <a name="process-private-bytes-performancecountersprocessprivatebytes"></a>Octets privés du processus (performanceCounters/processPrivateBytes)

Quantité de mémoire non partagée que le processus surveillé a allouée pour ses données.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Octets|Moyenne, Min, Max|`Cloud role instance` |

### <a name="processor-time-performancecountersprocessorcpupercentage"></a>Temps processeur (performanceCounters/processorCpuPercentage)

Consommation d’UC de *tous* les processus s'exécutant sur l'instance de serveur surveillée.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
|Pourcentage|Moyenne, Min, Max|`Cloud role instance` |

>[!NOTE]
> La métrique Temps processeur n'est pas disponible pour les applications hébergées dans Azure App Services. Utilisez la métrique [Utilisation de l'UC](#process-cpu-performancecountersprocesscpupercentage) pour suivre l'utilisation d’UC des applications web hébergées dans App Services.

## <a name="server-metrics"></a>Indicateurs de performances de serveur

### <a name="dependency-calls-dependenciescount"></a>Appels de dépendance (dependencies/count)

Cette métrique est liée au nombre d'appels de dépendance.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Count | Count | `Cloud role instance`, `Cloud role name`, `Dependency performance`, `Dependency type`, `Is traffic synthetic`, `Result code`, `Successful call`, `Target of a dependency call` |

### <a name="dependency-duration-dependenciesduration"></a>Durée de la dépendance (dependencies/duration)

Cette métrique fait référence à la durée des appels de dépendance.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Heure | Moyenne, Min, Max | `Cloud role instance`, `Cloud role name`, `Dependency performance`, `Dependency type`, `Is traffic synthetic`, `Result code`, `Successful call`, `Target of a dependency call` |


### <a name="server-request-rate-requestscount"></a>Taux de requêtes du serveur (requests/count)

Cette métrique reflète le nombre de requêtes de serveur entrantes reçues par votre application web.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Count | Average | `Cloud role instance`, `Cloud role name`, `Is traffic synthetic`, `Result performance` `Result code`, `Successful request` |

### <a name="server-requests-requestscount"></a>Requêtes serveur (requests/count)

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Count | Count | `Cloud role instance`, `Cloud role name`, `Is traffic synthetic`, `Result performance` `Result code`, `Successful request` |

### <a name="server-response-time-requestsduration"></a>Temps de réponse du serveur (requests/duration)

Cette métrique reflète le temps nécessaire aux serveurs pour traiter les demandes entrantes.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Heure | Moyenne, Min, Max | `Cloud role instance`, `Cloud role name`, `Is traffic synthetic`, `Result performance` `Result code`, `Successful request` |

## <a name="usage-metrics"></a>Métriques d'utilisation

### <a name="page-view-load-time-pageviewsduration"></a>Temps de chargement de la page consultée (pageViews/duration)

Cette métrique fait référence au temps nécessaire au chargement des événements PageView.

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Heure | Moyenne, Min, Max | `Cloud role name`, `Is traffic synthetic` |

### <a name="page-views-pageviewscount"></a>Affichages de pages (pageViews/count)

Nombre d'événements PageView consignés avec l'API Application Insights TrackPageView().

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Count | Count | `Cloud role name`, `Is traffic synthetic` |

### <a name="traces-tracescount"></a>Traces (traces/count)

Nombre d'instructions de suivi consignées avec l'appel à l’API Application Insights TrackTrace().

|Unité de mesure|Agrégations prises en charge|Dimensions prises en charge|
|---|---|---|
| Count | Count | `Cloud role instance`, `Cloud role name`,  `Is traffic synthetic`, `Severity level` |


## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [les métriques basées sur les journaux et les métriques pré-agrégées](https://docs.microsoft.com/azure/azure-monitor/app/pre-aggregated-metrics-log-metrics).
* [Définitions et requêtes de métriques basées sur le journal](https://docs.microsoft.com/azure/azure-monitor/platform/app-insights-metrics).