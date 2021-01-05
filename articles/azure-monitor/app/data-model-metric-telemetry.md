---
title: Modèle de données pour la télémétrie des métriques - Azure Application Insights
description: Modèle de données Application Insights pour la télémétrie des mesures
ms.topic: conceptual
ms.date: 04/25/2017
ms.reviewer: sergkanz
ms.openlocfilehash: f8e787b8a94987e7d68e12a0e4e69f4c63e21e19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87320576"
---
# <a name="metric-telemetry-application-insights-data-model"></a>Télémétrie des mesure : modèle de données Application Insights

Deux types de télémétrie des mesures sont prises en charge par [Application Insights](./app-insights-overview.md) : la mesure unique et la mesure pré-agrégée. La mesure unique consiste simplement dans un nom et une valeur. La mesure pré-agrégée spécifie les valeurs minimale et maximale de la mesure dans l’intervalle d’agrégation, ainsi que son écart standard.

La télémétrie des mesures pré-agrégées suppose que cette période d’agrégation est d’une minute.

Plusieurs noms de mesure connus sont pris en charge par Application Insights. Ces métriques sont placées dans la table performanceCounters.

Système de représentation des mesures et compteurs de processus :

| **Nom .NET**             | **Nom sans plateforme spécifiée** | **Nom d’API REST** | **Description**
| ------------------------- | -------------------------- | ----------------- | ---------------- 
| `\Processor(_Total)\% Processor Time` | Travail en cours... | [processorCpuPercentage](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessorCpuPercentage) | nombre total de processeurs de l’ordinateur
| `\Memory\Available Bytes`                 | Travail en cours... | [memoryAvailableBytes](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FmemoryAvailableBytes) | Affiche la quantité de mémoire physique (en octets) disponible pour les processus en cours d’exécution sur l’ordinateur. Elle est calculée en additionnant la quantité d’espace sur les listes de mémoire mises à zéro, libres et en attente. La mémoire disponible est prête à être utilisée. La mémoire mise à zéro consiste en des pages de mémoire remplies de zéros pour empêcher la consultation des données d’un processus précédent par des processus ultérieurs. La mémoire en attente est une mémoire supprimée de la plage de travail d’un processus (sa mémoire physique) en cours de route sur le disque, mais pouvant toujours être rappelée. Voir [Objet mémoire](/previous-versions/ms804008(v=msdn.10))
| `\Process(??APP_WIN32_PROC??)\% Processor Time` | Travail en cours... | [processCpuPercentage](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessCpuPercentage) | processeur du processus hébergeant l’application
| `\Process(??APP_WIN32_PROC??)\Private Bytes`      | Travail en cours... | [processPrivateBytes](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessPrivateBytes) | mémoire utilisée par le processus hébergeant l’application
| `\Process(??APP_WIN32_PROC??)\IO Data Bytes/sec` | Travail en cours... | [processIOBytesPerSecond](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FprocessIOBytesPerSecond) | fréquence des opérations d’E/S exécutées par le processus hébergeant l’application
| `\ASP.NET Applications(??APP_W3SVC_PROC??)\Requests/Sec`             | Travail en cours... | [requestsPerSecond](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FrequestsPerSecond) | fréquence des requêtes traitées par application 
| `\.NET CLR Exceptions(??APP_CLR_PROC??)\# of Exceps Thrown / sec`    | Travail en cours... | [exceptionsPerSecond](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FexceptionsPerSecond) | fréquence des exceptions levées par application
| `\ASP.NET Applications(??APP_W3SVC_PROC??)\Request Execution Time`   | Travail en cours... | [requestExecutionTime](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FrequestExecutionTime) | durée d’exécution moyenne des requêtes
| `\ASP.NET Applications(??APP_W3SVC_PROC??)\Requests In Application Queue` | Travail en cours... | [requestsInQueue](https://dev.applicationinsights.io/apiexplorer/metrics?appId=DEMO_APP&apiKey=DEMO_KEY&metricId=performanceCounters%2FrequestsInQueue) | nombre de requêtes en attente de traitement dans une file d’attente

## <a name="name"></a>Name

Nom de la mesure que vous aimeriez voir dans le portail Application Insights et dans l’interface utilisateur. 

## <a name="value"></a>Valeur

Valeur unique pour la mesure. Somme des mesures individuelles de l’agrégation.

## <a name="count"></a>Count

Poids de mesure de la mesure agrégée. Ne doit pas être défini pour une mesure.

## <a name="min"></a>Min

Valeur minimale de la mesure agrégée. Ne doit pas être défini pour une mesure.

## <a name="max"></a>Max

Valeur maximale de la mesure agrégée. Ne doit pas être défini pour une mesure.

## <a name="standard-deviation"></a>Écart standard

Écart standard de la mesure agrégée. Ne doit pas être défini pour une mesure.

## <a name="custom-properties"></a>Propriétés personnalisées

Si la propriété personnalisée `CustomPerfCounter` d’une métrique est définie sur `true`, celle-ci représente le compteur de performances Windows. Ces métriques sont placées dans la table performanceCounters, pas dans customMetrics. De plus, le nom de cette métrique est analysé pour extraire la catégorie, le compteur et les noms d’instance.

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser [l’API Application Insights pour les événements et les mesures personnalisés](./api-custom-events-metrics.md#trackmetric).
- Pour connaître les types et les modèles de données Application Insights, consultez [Modèle de données](data-model.md).
- Découvrez quelles [plateformes](./platforms.md) sont prises en charge par Application Insights.

