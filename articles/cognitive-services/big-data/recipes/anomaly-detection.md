---
title: 'Recette : Maintenance prédictive avec Cognitive Services pour le Big Data'
titleSuffix: Azure Cognitive Services
description: Ce guide de démarrage rapide montre comment effectuer une détection des anomalies distribuée avec Cognitive Services pour le Big Data
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: how-to
ms.date: 07/06/2020
ms.author: marhamil
ms.custom: devx-track-python
ms.openlocfilehash: d2995f39bc61ae5bb87abafd674f411271e57ca2
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94366279"
---
# <a name="recipe-predictive-maintenance-with-the-cognitive-services-for-big-data"></a>Recette : Maintenance prédictive avec Cognitive Services pour le Big Data

Cette recette montre comment vous pouvez utiliser Azure Synapse Analytics et Cognitive Services sur Apache Spark pour la maintenance prédictive des appareils IoT. Nous allons suivre l’exemple [CosmosDB et Synapse Link](https://github.com/Azure-Samples/cosmosdb-synapse-link-samples). Pour simplifier les choses, dans cette recette, nous allons lire les données directement à partir d’un fichier CSV au lieu d’obtenir des données diffusées en streaming via CosmosDB et Synapse Link. Nous vous encourageons vivement à examiner l’exemple Synapse Link.

## <a name="hypothetical-scenario"></a>Scénario hypothétique

Notre scénario hypothétique correspond à une centrale électrique, où les appareils IoT surveillent des [turbines à vapeur](https://en.wikipedia.org/wiki/Steam_turbine). La collection IoTSignals a des données Tours par minute (RPM) et Mégawatts (MW) pour chaque turbine. Les signaux provenant des turbines à vapeur sont analysés et des signaux anormaux sont détectés.

Il peut y avoir des valeurs hors norme dans les données de fréquence aléatoire. Dans ce cas, les valeurs RPM augmentent et la sortie MW diminue, pour la protection des circuits. L’idée est de voir les données varier au même moment, mais avec des signaux différents.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* [Espace de travail Azure Synapse](../../../synapse-analytics/quickstart-create-workspace.md) configuré avec un [pool Apache Spark serverless](../../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md)

## <a name="setup"></a>Programme d’installation

### <a name="create-an-anomaly-detector-resource"></a>Créer une ressource Détecteur d’anomalies

Les services Azure Cognitive Services sont représentés par des ressources Azure auxquelles vous vous abonnez. Créez une ressource pour Translator avec le [portail Azure](../../cognitive-services-apis-create-account.md) ou [Azure CLI](../../cognitive-services-apis-create-account-cli.md). Vous pouvez également :

- Visualiser une ressource existante dans le [portail Azure](https://portal.azure.com/).

Notez le point de terminaison et la clé de cette ressource. Vous en aurez besoin dans ce guide.

## <a name="enter-your-service-keys"></a>Entrer vos clés de service

Commençons par ajouter votre clé et votre emplacement.

```python
service_key = None # Paste your anomaly detector key here
location = None # Paste your anomaly detector location here

assert (service_key is not None)
assert (location is not None)
```

## <a name="read-data-into-a-dataframe"></a>Lire les données dans un DataFrame

À présent, nous allons lire le fichier IoTSignals dans un DataFrame. Ouvrez un nouveau notebook dans votre espace de travail Synapse et créez un DataFrame à partir du fichier.

```python
df_device_info = spark.read.csv("wasbs://publicwasb@mmlspark.blob.core.windows.net/iot/IoTSignals.csv", header=True, inferSchema=True)
```

### <a name="run-anomaly-detection-using-cognitive-services-on-spark"></a>Exécuter la détection d’anomalies à l’aide de Cognitive Services sur Spark

L’objectif est de trouver des instances où les signaux des appareils IoT comportent des valeurs anormales afin de détecter l’apparition d’un problème et d’effectuer une maintenance prédictive. Pour ce faire, nous allons utiliser le détecteur d’anomalies sur Spark :

```python
from pyspark.sql.functions import col, struct
from mmlspark.cognitive import SimpleDetectAnomalies
from mmlspark.core.spark import FluentAPI

detector = (SimpleDetectAnomalies()
    .setSubscriptionKey(service_key)
    .setLocation(location)
    .setOutputCol("anomalies")
    .setGroupbyCol("grouping")
    .setSensitivity(95)
    .setGranularity("secondly"))

df_anomaly = (df_signals
    .where(col("unitSymbol") == 'RPM')
    .withColumn("timestamp", col("dateTime").cast("string"))
    .withColumn("value", col("measureValue").cast("double"))
    .withColumn("grouping", struct("deviceId"))
    .mlTransform(detector)).cache()

df_anomaly.createOrReplaceTempView('df_anomaly')
```

Examinons les données :

```python
df_anomaly.select("timestamp","value","deviceId","anomalies.isAnomaly").show(3)
```

Cette cellule doit donner un résultat ressemblant à ceci :

| timestamp           |   value | deviceId   | isAnomaly   |
|:--------------------|--------:|:-----------|:------------|
| 2020-05-01 18:33:51 |    3174 | dev-7      | False       |
| 2020-05-01 18:33:52 |    2976 | dev-7      | False       |
| 2020-05-01 18:33:53 |    2714 | dev-7      | False       |


 ## <a name="visualize-anomalies-for-one-of-the-devices"></a>Visualiser les anomalies de l’un des appareils

IoTSignals.csv contient des signaux provenant de plusieurs appareils IoT. Nous allons nous concentrer sur un appareil spécifique et visualiser des sorties anormales à partir de cet appareil.

```python
df_anomaly_single_device = spark.sql("""
select
  timestamp,
  measureValue,
  anomalies.expectedValue,
  anomalies.expectedValue + anomalies.upperMargin as expectedUpperValue,
  anomalies.expectedValue - anomalies.lowerMargin as expectedLowerValue,
  case when anomalies.isAnomaly=true then 1 else 0 end as isAnomaly
from
  df_anomaly
where deviceid = 'dev-1' and timestamp < '2020-04-29'
order by timestamp
limit 200""")
```

Maintenant que nous avons créé un DataFrame qui représente les anomalies d’un appareil particulier, nous pouvons visualiser ces anomalies :

```python
import matplotlib.pyplot as plt
from pyspark.sql.functions import col

adf = df_anomaly_single_device.toPandas()
adf_subset = df_anomaly_single_device.where(col("isAnomaly") == 1).toPandas()

plt.figure(figsize=(23,8))
plt.plot(adf['timestamp'],adf['expectedUpperValue'], color='darkred', linestyle='solid', linewidth=0.25, label='UpperMargin')
plt.plot(adf['timestamp'],adf['expectedValue'], color='darkgreen', linestyle='solid', linewidth=2, label='Expected Value')
plt.plot(adf['timestamp'],adf['measureValue'], 'b', color='royalblue', linestyle='dotted', linewidth=2, label='Actual')
plt.plot(adf['timestamp'],adf['expectedLowerValue'],  color='black', linestyle='solid', linewidth=0.25, label='Lower Margin')
plt.plot(adf_subset['timestamp'],adf_subset['measureValue'], 'ro', label = 'Anomaly')
plt.legend()
plt.title('RPM Anomalies with Confidence Intervals')
plt.show()
```

Si tout se passe bien, votre sortie se présentera ainsi :

![Graphique du détecteur d’anomalies](../media/anomaly-output.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment effectuer une maintenance prédictive à grande échelle avec Azure Cognitive Services, Azure Synapse Analytics et Azure CosmosDB. Pour plus d’informations, consultez l’exemple complet sur [GitHub](https://github.com/Azure-Samples/cosmosdb-synapse-link-samples).