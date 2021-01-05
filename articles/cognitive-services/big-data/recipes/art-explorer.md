---
title: 'Recette : Exploration intelligente d’images avec Cognitive Services pour le Big Data'
titleSuffix: Azure Cognitive Services
description: Cette recette montre comment créer une base de données d’art pouvant servir à faire des recherches à l’aide d’Azure Search et de MMLSpark.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: how-to
ms.date: 07/06/2020
ms.author: marhamil
ms.custom: devx-track-python
ms.openlocfilehash: 5a65ff28a38e42e05844063a330c0325f16b2247
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94363287"
---
# <a name="recipe-intelligent-art-exploration-with-the-cognitive-services-for-big-data"></a>Recette : Exploration intelligente d’images avec Cognitive Services pour le Big Data

Dans cet exemple, nous allons utiliser Cognitive Services pour le Big Data afin d’ajouter des annotations intelligentes à la collection en libre accès du Metropolitan Museum of Art (MET). Ceci nous permettra de créer un moteur de recherche intelligent à l’aide d’Azure Search, même sans annotations manuelles. 

## <a name="prerequisites"></a>Prérequis

* Vous devez disposer d’une clé d’abonnement pour Vision par ordinateur et Recherche cognitive. Suivez les instructions mentionnées dans [Créer un compte Cognitive Services](../../cognitive-services-apis-create-account.md) pour vous abonner à Vision par ordinateur et obtenir votre clé.
  > [!NOTE]
  > Pour obtenir des informations de tarification, consultez [Recherche cognitive Azure](https://azure.microsoft.com/services/search/#pricing).

## <a name="import-libraries"></a>Importer les bibliothèques

Exécutez la commande suivante pour importer les bibliothèques pour cette recette.

```python
import os, sys, time, json, requests
from pyspark.ml import Transformer, Estimator, Pipeline
from pyspark.ml.feature import SQLTransformer
from pyspark.sql.functions import lit, udf, col, split
```

## <a name="set-up-subscription-keys"></a>Configurer les clés d’abonnement

Exécutez la commande suivante pour configurer des variables pour les clés de service. Insérez vos clés d’abonnement pour Vision par ordinateur et Recherche cognitive.

```python
VISION_API_KEY = 'INSERT_COMPUTER_VISION_SUBSCRIPTION_KEY'
AZURE_SEARCH_KEY = 'INSERT_AZURE_COGNITIVE_SEARCH_SUBSCRIPTION_KEY'
search_service = "mmlspark-azure-search"
search_index = "test"
```

## <a name="read-the-data"></a>Lire les données

Exécutez la commande suivante pour charger les données à partir de la collection en libre accès du MET.

```python
data = spark.read\
  .format("csv")\
  .option("header", True)\
  .load("wasbs://publicwasb@mmlspark.blob.core.windows.net/metartworks_sample.csv")\
  .withColumn("searchAction", lit("upload"))\
  .withColumn("Neighbors", split(col("Neighbors"), ",").cast("array<string>"))\
  .withColumn("Tags", split(col("Tags"), ",").cast("array<string>"))\
  .limit(25)
```

<a name="AnalyzeImages"></a>

## <a name="analyze-the-images"></a>Analyser les images

Exécutez la commande suivante pour utiliser Vision par ordinateur sur la collection d’œuvres d’art en libre accès du MET. Vous obtiendrez alors les fonctionnalités visuelles des œuvres d’art.

```python
from mmlspark.cognitive import AnalyzeImage
from mmlspark.stages import SelectColumns

#define pipeline
describeImage = (AnalyzeImage()
  .setSubscriptionKey(VISION_API_KEY)
  .setLocation("eastus")
  .setImageUrlCol("PrimaryImageUrl")
  .setOutputCol("RawImageDescription")
  .setErrorCol("Errors")
  .setVisualFeatures(["Categories", "Tags", "Description", "Faces", "ImageType", "Color", "Adult"])
  .setConcurrency(5))

df2 = describeImage.transform(data)\
  .select("*", "RawImageDescription.*").drop("Errors", "RawImageDescription")
```

<a name="CreateSearchIndex"></a>

## <a name="create-the-search-index"></a>Créer l’index de recherche

Exécutez la commande suivante pour écrire les résultats dans la recherche Azure afin de créer un moteur de recherche des œuvres d’art avec des métadonnées enrichies à partir de Vision par ordinateur.

```python
from mmlspark.cognitive import *
df2.writeToAzureSearch(
  subscriptionKey=AZURE_SEARCH_KEY,
  actionCol="searchAction",
  serviceName=search_service,
  indexName=search_index,
  keyCol="ObjectID"
)
```

## <a name="query-the-search-index"></a>Interroger l’index de recherche

Exécutez la commande suivante pour interroger l’index recherche Azure.

```python
url = 'https://{}.search.windows.net/indexes/{}/docs/search?api-version=2019-05-06'.format(search_service, search_index)
requests.post(url, json={"search": "Glass"}, headers = {"api-key": AZURE_SEARCH_KEY}).json()
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser [Cognitive Services pour le Big Data pour détecter des anomalies](anomaly-detection.md).