---
title: Détection d’anomalies dans Azure Stream Analytics
description: Cet article décrit comment utiliser Azure Stream Analytics et Azure Machine Learning pour détecter les anomalies.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/21/2019
ms.openlocfilehash: c57a3920dac3e18e248109fafdf61fdfa871c54d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023395"
---
# <a name="anomaly-detection-in-azure-stream-analytics"></a>Détection d’anomalies dans Azure Stream Analytics

Disponible à la fois dans le cloud et Azure IoT Edge, Azure Stream Analytics offre des fonctionnalités de détection des anomalies basées sur un Machine Learning intégré qui peuvent permettre de surveiller les deux anomalies qui se produisent le plus souvent : temporaires et persistantes. Les fonctions **AnomalyDetection_SpikeAndDip** et **AnomalyDetection_ChangePoint** permettent de détecter des anomalies directement dans votre travail Stream Analytics.

Les modèles Machine Learning supposent une série chronologique équitablement échantillonnée. Si la série chronologique n’est pas uniforme, vous pouvez insérer une étape d’agrégation avec une fenêtre bascule avant l’appel de la détection des anomalies.

Actuellement, les opérations Machine Learning ne prennent pas en charge les tendances de saisonnalité ni les corrélations multivariées.

## <a name="anomaly-detection-using-machine-learning-in-azure-stream-analytics"></a>Détection des anomalies à laide de Machine Learning dans Azure Stream Analytics

La vidéo suivante montre comment détecter une anomalie en temps réel à l’aide des fonctions Machine Learning dans Azure Stream Analytics. 

> [!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Real-Time-ML-Based-Anomaly-Detection-In-Azure-Stream-Analytics/player]

## <a name="model-behavior"></a>Comportement du modèle

En règle générale, la précision du modèle s’améliore avec l’augmentation des données dans la fenêtre glissante. Les données de la fenêtre glissante spécifiée sont traitées comme si elles faisaient partie de la plage normale de valeurs pour cette période. Le modèle ne prend en considération que l’historique des événements de la fenêtre glissante pour vérifier si l’événement actuel est anormal. À mesure que la fenêtre glissante se déplace, les anciennes valeurs sont supprimées de la formation du modèle.

Les fonctions établissent une certaine normale selon ce qu’elles ont déjà vu jusqu’à présent. Les valeurs hors norme sont identifiées par la comparaison par rapport à la normale établie au sein du niveau de confiance. La taille de la fenêtre doit être basée sur le nombre minimal d’événements requis pour former le modèle à un comportement normal afin qu’il puisse reconnaître une anomalie lorsqu’elle se produit.

Le temps de réponse du modèle augmente avec la taille de l’historique, car il doit effectuer une comparaison avec un nombre plus élevé d’événements passés. Il est recommandé de n’inclure que le nombre d’événements nécessaires pour optimiser les performances.

Des lacunes dans la série chronologique peuvent être le résultat du modèle qui ne reçoit pas d’événements à certains moments au fil du temps. Cette situation est gérée par Stream Analytics à l’aide de la logique d'imputation. La taille de l’historique, ainsi que la durée, pour la même fenêtre glissante sont utilisées pour calculer la vitesse moyenne à laquelle les événements sont attendus.

Un générateur d’anomalies disponible [ici](https://aka.ms/asaanomalygenerator) peut être utilisé pour alimenter un Iot Hub en données avec différents modèles d’anomalies. Une tâche ASA peut être configurée avec ces fonctions de détection d’anomalies à des fins de lecture à partir de cet Iot Hub et de détection des anomalies.

## <a name="spike-and-dip"></a>Pics et baisses

Les anomalies temporaires d’un flux d’événements d’une série chronologique sont appelées « pics et baisses ». Les pics et baisses peuvent être surveillés à l’aide de l’opérateur basé sur Machine Learning [AnomalyDetection_SpikeAndDip](/stream-analytics-query/anomalydetection-spikeanddip-azure-stream-analytics
).

![Exemple d’anomalie de pic et de baisse](./media/stream-analytics-machine-learning-anomaly-detection/anomaly-detection-spike-dip.png)

Dans la même fenêtre glissante, si un deuxième pic est plus petit que le premier, le score calculé pour le plus petit pic n’est probablement pas suffisamment important par rapport au score du premier pic au niveau de confiance spécifié. Vous pouvez essayer de réduire le paramètre de niveau de confiance du modèle pour détecter ces anomalies. Toutefois, si vous commencez à recevoir trop d’alertes, vous pouvez utiliser un intervalle de confiance plus élevé.

L’exemple de requête suivant suppose une vitesse d’entrée uniforme d’un événement par seconde dans une fenêtre glissante de 2 minutes avec un historique de 120 événements. La dernière instruction SELECT extrait et génère le score et le statut d’anomalie avec un niveau de confiance de 95 %.

```SQL
WITH AnomalyDetectionStep AS
(
    SELECT
        EVENTENQUEUEDUTCTIME AS time,
        CAST(temperature AS float) AS temp,
        AnomalyDetection_SpikeAndDip(CAST(temperature AS float), 95, 120, 'spikesanddips')
            OVER(LIMIT DURATION(second, 120)) AS SpikeAndDipScores
    FROM input
)
SELECT
    time,
    temp,
    CAST(GetRecordPropertyValue(SpikeAndDipScores, 'Score') AS float) AS
    SpikeAndDipScore,
    CAST(GetRecordPropertyValue(SpikeAndDipScores, 'IsAnomaly') AS bigint) AS
    IsSpikeAndDipAnomaly
INTO output
FROM AnomalyDetectionStep
```

## <a name="change-point"></a>Point de changement

Les anomalies persistantes d’un flux d’événements d’une série chronologique sont des modifications de la distribution des valeurs du flux d’événements, telles que des modifications du niveau et des tendances. Dans Stream Analytics, ces anomalies sont détectées à l’aide de l’opérateur basé sur Machine Learning [AnomalyDetection_ChangePoint](/stream-analytics-query/anomalydetection-changepoint-azure-stream-analytics).

Les modifications persistantes durent beaucoup plus longtemps que les pics et les baisses, et peuvent indiquer un ou des événements catastrophiques. Les modifications persistantes ne sont généralement pas visibles à l’œil nu, mais peuvent être détectées avec l’opérateur **AnomalyDetection_ChangePoint**.

L’image suivante est un exemple de modification de niveau :

![Exemple d’anomalie de modification de niveau](./media/stream-analytics-machine-learning-anomaly-detection/anomaly-detection-level-change.png)

L’image suivante est un exemple de modification de tendance :

![Exemple d’anomalie de modification de tendance](./media/stream-analytics-machine-learning-anomaly-detection/anomaly-detection-trend-change.png)

L’exemple de requête suivant suppose une vitesse d’entrée uniforme d’un événement par seconde dans une fenêtre glissante de 20 minutes avec une taille d’historique de 1 200 événements. La dernière instruction SELECT extrait et génère le score et le statut d’anomalie avec un niveau de confiance de 80 %.

```SQL
WITH AnomalyDetectionStep AS
(
    SELECT
        EVENTENQUEUEDUTCTIME AS time,
        CAST(temperature AS float) AS temp,
        AnomalyDetection_ChangePoint(CAST(temperature AS float), 80, 1200) 
        OVER(LIMIT DURATION(minute, 20)) AS ChangePointScores
    FROM input
)
SELECT
    time,
    temp,
    CAST(GetRecordPropertyValue(ChangePointScores, 'Score') AS float) AS
    ChangePointScore,
    CAST(GetRecordPropertyValue(ChangePointScores, 'IsAnomaly') AS bigint) AS
    IsChangePointAnomaly
INTO output
FROM AnomalyDetectionStep

```

## <a name="performance-characteristics"></a>Caractéristiques du niveau de performance

Le niveau de performance de ces modèles dépend de la taille de l’historique, de la durée de la fenêtre, de la charge d'événements, ainsi que de l'utilisation du partitionnement au niveau de la fonction. Cette section traite de ces configurations et propose des exemples illustrant comment prendre en charge des taux d’ingestion d'événements de 1 Ko, 5 Ko et 10 Ko par seconde.

* **Taille de l’historique** - Ces modèles fonctionnent de manière linéaire avec la **taille de l’historique**. Plus l’historique est volumineux, plus les modèles mettent du temps à évaluer un nouvel événement. En effet, les modèles comparent le nouvel événement avec les différents événements passés dans la mémoire tampon de l’historique.
* **Durée de la fenêtre** - La **durée de la fenêtre** doit refléter le temps nécessaire à la réception du nombre d'événements défini par la taille de l’historique. Sans ce nombre d’événements dans la fenêtre, Azure Stream Analytics impute les valeurs manquantes. Par conséquent, la consommation du processeur est une fonction de la taille de l’historique.
* **Charge d'événements** - Plus la **charge d’événements** est importante, plus les modèles sont sollicités, ce qui a une incidence sur la consommation du processeur. Le travail peut être monté en charge moyennant un parallélisme massif, en supposant qu'il soit judicieux, en terme de logique métier, d'utiliser plus de partitions d’entrée.
* **Partitionnement au niveau de la fonction** - **Le partitionnement au niveau de la fonction** s’effectue avec ```PARTITION BY``` au sein de l’appel de fonction de détection des anomalies. Ce type de partitionnement ajoute une charge, car l'état doit être conservé pour plusieurs modèles en même temps. Le partitionnement au niveau de la fonction est utilisé dans des scénarios tels que le partitionnement au niveau de l'appareil.

### <a name="relationship"></a>Relation
La taille de l’historique, la durée de la fenêtre et la charge totale d’événements sont associées comme suit :

windowDuration (en ms) = 1000 * historySize / (total d'événements d'entrée par seconde / nombre de partitions d’entrée)

Lors du partitionnement de la fonction par deviceId, ajoutez « PARTITION par deviceId » à l’appel de la fonction de détection des anomalies.

### <a name="observations"></a>Observations
Le tableau suivant présente les observations de débit pour un seul nœud (6 SU) pour le cas non partitionné :

| Taille de l’historique (événements) | Durée de la fenêtre (ms) | Nombre total d’événements d’entrée par seconde |
| --------------------- | -------------------- | -------------------------- |
| 60 | 55 | 2 200 |
| 600 | 728 | 1 650 |
| 6 000 / 750 | 10 910 | 1 100 |

Le tableau suivant présente les observations de débit pour un seul nœud (6 SU) pour le cas partitionné :

| Taille de l’historique (événements) | Durée de la fenêtre (ms) | Nombre total d’événements d’entrée par seconde | Nombre d’appareils |
| --------------------- | -------------------- | -------------------------- | ------------ |
| 60 | 1 091 | 1 100 | 10 |
| 600 | 10 910 | 1 100 | 10 |
| 6 000 / 750 | 218 182 | <550 | 10 |
| 60 | 21 819 | 550 | 100 |
| 600 | 218 182 | 550 | 100 |
| 6 000 / 750 | 2 181 819 | <550 | 100 |

Un exemple de code permettant d'exécuter les configurations non partitionnées ci-dessus est disponible dans le [référentiel Streaming At Scale](https://github.com/Azure-Samples/streaming-at-scale/blob/f3e66fa9d8c344df77a222812f89a99b7c27ef22/eventhubs-streamanalytics-eventhubs/anomalydetection/create-solution.sh) des exemples Azure. Le code crée un travail Stream Analytics sans partitionnement au niveau de la fonction, utilisant Event Hub comme entrée et sortie. La charge d’entrée est générée à l’aide de clients de test. Chaque événement d’entrée correspond à un document json de 1 Ko. Les événements simulent un appareil IoT envoyant des données JSON (jusqu'à 1 Ko). La taille de l’historique, la durée de la fenêtre et la charge totale d'événements varient sur deux partitions d'entrée.

> [!Note]
> Pour une estimation plus précise, personnalisez les échantillons en fonction de votre scénario.

### <a name="identifying-bottlenecks"></a>Identification des goulots d’étranglement
Utilisez le volet Métriques de votre travail Azure Stream Analytics pour identifier les goulots d’étranglement de votre pipeline. Examinez les **événements d’entrée/sortie** pour le débit, ainsi que le [ « Délai en filigrane »](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/) ou les **Événements en backlog**, pour voir si le travail suit la vitesse d’entrée. Pour les métriques Event Hub, recherchez les **Demandes limitées** et ajustez les Unités de seuil en conséquence. Pour les métriques de Cosmos DB, examinez la valeur **Nombre maximal de RU/s consommées par groupe de clés de partition** sous Débit pour vous assurer que les groupes de clés de partition sont consommés de manière uniforme. Pour Azure SQL DB, surveillez **E/S journal** et **UC**.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)