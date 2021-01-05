---
title: Comment utiliser l’API Détecteur d’anomalies sur vos données de série chronologique
titleSuffix: Azure Cognitive Services
description: Apprenez à détecter les anomalies dans vos données sous forme de lot ou dans vos données diffusées.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 10/01/2019
ms.author: mbullwin
ms.openlocfilehash: b2cd5e32503953de874ab470ca3f9413d2b37d59
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92014645"
---
# <a name="how-to-use-the-anomaly-detector-api-on-your-time-series-data"></a>Procédure : Utiliser l’API Détecteur d’anomalies sur vos données de série chronologique  

[L’API Détecteur d’anomalies](https://westus2.dev.cognitive.microsoft.com/docs/services/AnomalyDetector/operations/post-timeseries-entire-detect) propose deux méthodes de détection des anomalies. Vous pouvez soit détecter les anomalies sous forme de lot tout au long de votre série chronologique, soit au moment où vos données sont générées en détectant l’état d’anomalie du dernier point de données. Le modèle de détection retourne les résultats d’anomalie, ainsi que la valeur attendue de chaque point de données et les limites supérieure et inférieure de la détection d’anomalie. Vous pouvez utiliser ces valeurs pour visualiser la plage des valeurs normales et les anomalies dans les données.

## <a name="anomaly-detection-modes"></a>Modes de détection d’anomalie 

L’API Détecteur d’anomalies fournit des modes de détection : batch et diffusion en continu.

> [!NOTE]
> Les URL de demande suivantes doivent être combinées avec le point de terminaison approprié pour votre abonnement. Par exemple : `https://<your-custom-subdomain>.api.cognitive.microsoft.com/anomalydetector/v1.0/timeseries/entire/detect`


### <a name="batch-detection"></a>Détection par lot

Pour détecter les anomalies tout au long d’un lot de points de données sur une période donnée, utilisez l’URI de demande suivant avec vos données de série chronologique : 

`/timeseries/entire/detect`. 

En envoyant vos données de série chronologique en même temps, l’API génère un modèle à l’aide de la série entière et en analyse chaque point de données.  

### <a name="streaming-detection"></a>Détection dans la diffusion en continu

Pour détecter en continu les anomalies dans les données diffusées, utilisez l’URI de demande suivant avec votre dernier point de données : 

`/timeseries/last/detect'`. 

En envoyant les nouveaux points de données dès que vous les générez, vous pouvez surveiller vos données en temps réel. Un modèle est généré avec les points de données que vous envoyez et l’API détermine si le dernier point dans la série chronologique est une anomalie.

## <a name="adjusting-lower-and-upper-anomaly-detection-boundaries"></a>Ajustement des limites inférieure et supérieure de la détection d’anomalie

Par défaut, les limites supérieure et inférieure de la détection d’anomalie sont calculées à l’aide de `expectedValue`, `upperMargin` et `lowerMargin`. Si vous voulez des limites différentes, nous vous recommandons d’appliquer un `marginScale` à `upperMargin` ou `lowerMargin`. Les limites sont calculées comme suit :

|Limite  |Calcul  |
|---------|---------|
|`upperBoundary` | `expectedValue + (100 - marginScale) * upperMargin`        |
|`lowerBoundary` | `expectedValue - (100 - marginScale) * lowerMargin`        |

Les exemples suivants montrent un résultat de l’API Détecteur d’anomalies avec des sensibilités différentes.

### <a name="example-with-sensitivity-at-99"></a>Exemple avec une sensibilité de 99

![Sensibilité par défaut](../media/sensitivity_99.png)

### <a name="example-with-sensitivity-at-95"></a>Exemple avec une sensibilité de 95

![Sensibilité 99](../media/sensitivity_95.png)

### <a name="example-with-sensitivity-at-85"></a>Exemple avec une sensibilité de 85

![Sensibilité 85](../media/sensitivity_85.png)

## <a name="next-steps"></a>Étapes suivantes

* [Présentation de l’API Détecteur d’anomalies](../overview.md)
* [Démarrage rapide : Détecter des anomalies dans vos données de séries chronologiques avec l’API Détecteur d’anomalies](../quickstarts/detect-data-anomalies-csharp.md)
