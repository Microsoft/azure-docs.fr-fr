---
title: Cognitive Services pour le Big Data - Exemples Python
description: Essayez les exemples Cognitive Services dans Python pour Azure Databricks afin d’exécuter votre pipeline MMLSpark pour le Big Data.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.topic: sample
ms.date: 07/06/2020
ms.author: marhamil
ms.openlocfilehash: 590ddef27315f37719da5b28c68b6c402371e986
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94363253"
---
# <a name="python-samples-for-cognitive-services-for-big-data"></a>Exemples Python destinés aux services Cognitive Services pour le Big Data

Les extraits de code suivants sont prêts à être exécutés pour vous permettre de commencer à utiliser Cognitive Services sur Spark avec Python.

Les exemples de cet article utilisent les services Cognitive Services suivants :

- Analyse de texte : obtenir le sentiment (ou l’humeur) d’un ensemble de phrases.
- Vision par ordinateur : obtenir les étiquettes (descriptions en un mot) associées à un ensemble d’images.
- Recherche d’images Bing : rechercher sur le web des images associées à une demande en langage naturel.
- Reconnaissance vocale : transcrire des fichiers audio pour extraire des transcriptions texte.
- Détecteur d’anomalies : détecter les anomalies dans des données de séries chronologiques.

## <a name="prerequisites"></a>Prérequis

1. Suivez les étapes décrites dans [Bien démarrer](getting-started.md) pour configurer votre environnement Azure Databricks et Cognitive Services. Ce tutoriel vous montre comment installer MMLSpark et comment créer votre cluster Spark dans Databricks.
1. Après avoir créé un notebook dans Azure Databricks, copiez le **code partagé** ci-dessous et collez-le dans une nouvelle cellule de votre notebook.
1. Choisissez un exemple de service, ci-dessous, puis copiez-le et collez-le dans une deuxième nouvelle cellule de votre notebook.
1. Remplacez l’un des espaces réservés de clé d’abonnement au service par votre propre clé.
1. Cliquez sur le bouton Exécuter (icône de triangle) en haut à gauche de la cellule, puis sélectionnez **Exécuter la cellule**.
1. Consultez les résultats dans un tableau affiché sous la cellule.

## <a name="shared-code"></a>Code partagé

Pour commencer, nous devons ajouter ce code au projet :

```python
from mmlspark.cognitive import *

# A general Cognitive Services key for Text Analytics and Computer Vision (or use separate keys that belong to each service)
service_key = "ADD_YOUR_SUBSCRIPION_KEY"
# A Bing Search v7 subscription key
bing_search_key = "ADD_YOUR_SUBSCRIPION_KEY"
# An Anomaly Dectector subscription key
anomaly_key = "ADD_YOUR_SUBSCRIPION_KEY"

# Validate the key
assert service_key != "ADD_YOUR_SUBSCRIPION_KEY"
```    

## <a name="text-analytics-sample"></a>Exemple d’Analyse de texte

Le service [Analyse de texte](../text-analytics/index.yml) fournit plusieurs algorithmes permettant d’extraire des insights intelligents à partir d’un texte. Par exemple, nous pouvons trouver le sentiment d’un texte d’entrée donné. Le service retourne un score compris entre 0.0 et 1.0, où un score faible indique un sentiment négatif et où un score élevé indique un sentiment positif.  Cet exemple utilise trois phrases simples et retourne le sentiment pour chacune d’elles.

```python
from pyspark.sql.functions import col

# Create a dataframe that's tied to it's column names
df = spark.createDataFrame([
  ("I am so happy today, its sunny!", "en-US"),
  ("I am frustrated by this rush hour traffic", "en-US"),
  ("The cognitive services on spark aint bad", "en-US"),
], ["text", "language"])

# Run the Text Analytics service with options
sentiment = (TextSentiment()
    .setTextCol("text")
    .setLocation("eastus")
    .setSubscriptionKey(service_key)
    .setOutputCol("sentiment")
    .setErrorCol("error")
    .setLanguageCol("language"))

# Show the results of your text query in a table format
display(sentiment.transform(df).select("text", col("sentiment")[0].getItem("sentiment").alias("sentiment")))
```

### <a name="expected-result"></a>Résultat attendu

| texte                                      | sentiment                                             |
|:------------------------------------------|:------------------------------------------------------|
| I am so happy today, its sunny! (Je suis trop content aujourd’hui, il fait beau !)           | positif                                              |
| I am frustrated by this rush hour traffic (Cette circulation aux heures de pointe m’agace) | négatif                                              |
| The cognitive services on spark aint bad (Les services Cognitive Services sur Spark ne sont pas mal)  | positif                                              |

## <a name="computer-vision-sample"></a>API Vision par ordinateur

Le service [Vision par ordinateur](../computer-vision/index.yml) analyse des images pour identifier une structure comme des visages, des objets et des descriptions en langage naturel. Dans cet exemple, nous ajoutons des étiquettes à une liste d’images. Les étiquettes sont des descriptions en un mot des choses figurant sur l’image, comme des objets, des personnes, un paysage et des actions reconnaissables.

```python

# Create a dataframe with the image URLs
df = spark.createDataFrame([
        ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg", ),
        ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/dog.jpg", ),
        ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/house.jpg", )
    ], ["image", ])

# Run the Computer Vision service. Analyze Image extracts infortmation from/about the images.
analysis = (AnalyzeImage()
    .setLocation("eastus")
    .setSubscriptionKey(service_key)
    .setVisualFeatures(["Categories","Color","Description","Faces","Objects","Tags"])
    .setOutputCol("analysis_results")
    .setImageUrlCol("image")
    .setErrorCol("error"))

# Show the results of what you wanted to pull out of the images.
display(analysis.transform(df).select("image", "analysis_results.description.tags"))
```

### <a name="expected-result"></a>Résultat attendu

| image | tags |
|:------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------|
| https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg | [’skate' 'personne' 'homme' 'extérieur' 'monter' 'sport' 'skateboard' 'jeune' 'planche' 'maillot' 'air' 'noir' 'parc' 'garçon' 'côté' 'saut' 'figure' 'rampe' 'action' 'vol']
| https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/dog.jpg | ['chien' 'extérieur' 'barrière' 'bois' 'petit' 'marron' 'bâtiment' 'assis' 'devant' 'banc' 'debout' 'table' 'randonnée' 'planche' 'plage' 'blanc' 'soutien' 'pont' 'piste']                
| https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/house.jpg | ['extérieur' 'pelouse' 'maison' 'bâtiment' 'ancien' 'foyer' 'devant' 'petit' 'église' 'pierre' 'grand' 'pâturage' 'cour' 'vert' 'assis' 'premier plan' 'mouton' 'brique' 'banc' 'rue' 'white' 'blanc' 'horloge' 'panneau' 'garé' 'champ' 'debout' 'jardin' 'eau' 'rouge' 'cheval' 'homme' 'grand' 'feu' 'groupe']


## <a name="bing-image-search-sample"></a>Exemple de Recherche d’images Bing

La [Recherche d’images Bing](../bing-image-search/overview.md) explore le web pour récupérer des images associées à une demande en langage naturel d’un utilisateur. Dans cet exemple, nous utilisons une demande texte qui recherche des images avec des guillemets. Elle retourne une liste d’URL d’images qui contiennent des photos liées à notre demande.

```python
from pyspark.ml import PipelineModel

# Number of images Bing will return per query
imgsPerBatch = 10
# A list of offsets, used to page into the search results
offsets = [(i*imgsPerBatch,) for i in range(100)]
# Since web content is our data, we create a dataframe with options on that data: offsets
bingParameters = spark.createDataFrame(offsets, ["offset"])

# Run the Bing Image Search service with our text query
bingSearch = (BingImageSearch()
    .setSubscriptionKey(bing_search_key)
    .setOffsetCol("offset")
    .setQuery("Martin Luther King Jr. quotes")
    .setCount(imgsPerBatch)
    .setOutputCol("images"))

# Transformer that extracts and flattens the richly structured output of Bing Image Search into a simple URL column
getUrls = BingImageSearch.getUrlTransformer("images", "url")

# This displays the full results returned, uncomment to use
# display(bingSearch.transform(bingParameters))

# Since we have two services, they are put into a pipeline
pipeline = PipelineModel(stages=[bingSearch, getUrls])

# Show the results of your search: image URLs
display(pipeline.transform(bingParameters))
```

### <a name="expected-result"></a>Résultat attendu

| url |
|:-------------------------------------------------------------------------------------------------------------------|
| https://iheartintelligence.com/wp-content/uploads/2019/01/powerful-quotes-martin-luther-king-jr.jpg      |
| http://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes-16.jpg             |
| http://www.sofreshandsogreen.com/wp-content/uploads/2012/01/martin-luther-king-jr-quote-sofreshandsogreendotcom.jpg |
| https://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes-18.jpg            |
| https://tsal-eszuskq0bptlfh8awbb.stackpathdns.com/wp-content/uploads/2018/01/MartinLutherKingQuotes.jpg  |


## <a name="speech-to-text-sample"></a>Exemple de Reconnaissance vocale
Le service [Reconnaissance vocale](../speech-service/index-speech-to-text.yml) convertit des flux ou des fichiers de contenu audio en texte. Dans cet exemple, nous transcrivons deux fichiers audio. Le premier fichier est facile à comprendre et le second est plus complexe.

```python

# Create a dataframe with our audio URLs, tied to the column called "url"
df = spark.createDataFrame([("https://mmlspark.blob.core.windows.net/datasets/Speech/audio2.wav",),
                           ("https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3",)
                           ], ["url"])

# Run the Speech-to-text service to translate the audio into text
speech_to_text = (SpeechToTextSDK()
    .setSubscriptionKey(service_key)
    .setLocation("eastus")
    .setOutputCol("text")
    .setAudioDataCol("url")
    .setLanguage("en-US")
    .setProfanity("Masked"))

# Show the results of the translation
display(speech_to_text.transform(df).select("url", "text.DisplayText"))
```

### <a name="expected-result"></a>Résultat attendu

| url | DisplayText |
|:------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio2.wav | Custom Speech propose des outils qui vous permettent d’inspecter visuellement la qualité de la reconnaissance d’un modèle en comparant les données audio au résultat de la reconnaissance correspondante provenant du portail Custom Speech. Vous pouvez lire un contenu audio chargé pour déterminer si le résultat proposé de la reconnaissance est correct. Cet outil vous permet d’inspecter rapidement la qualité du modèle de référence de reconnaissance vocale de Microsoft ou d’un modèle personnalisé entraîné sans qu’il soit nécessaire de transcrire des données audio. |
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3 | Ajoutez un contrôle visuel des pensées d’un monsieur.    |
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3 | Je m’entends. |
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3 | J’aime avoir la garantie par la radio que je peux également l’entendre. |


## <a name="anomaly-detector-sample"></a>Exemple de Détecteur d’anomalies

Le [Détecteur d’anomalies](../anomaly-detector/index.yml) est idéal pour détecter les irrégularités dans vos données de séries chronologiques. Dans cet exemple, nous utilisons le service pour rechercher des anomalies dans l’ensemble de la série chronologique.

```python
from pyspark.sql.functions import lit

# Create a dataframe with the point data that Anomaly Detector requires
df = spark.createDataFrame([
    ("1972-01-01T00:00:00Z", 826.0),
    ("1972-02-01T00:00:00Z", 799.0),
    ("1972-03-01T00:00:00Z", 890.0),
    ("1972-04-01T00:00:00Z", 900.0),
    ("1972-05-01T00:00:00Z", 766.0),
    ("1972-06-01T00:00:00Z", 805.0),
    ("1972-07-01T00:00:00Z", 821.0),
    ("1972-08-01T00:00:00Z", 20000.0),
    ("1972-09-01T00:00:00Z", 883.0),
    ("1972-10-01T00:00:00Z", 898.0),
    ("1972-11-01T00:00:00Z", 957.0),
    ("1972-12-01T00:00:00Z", 924.0),
    ("1973-01-01T00:00:00Z", 881.0),
    ("1973-02-01T00:00:00Z", 837.0),
    ("1973-03-01T00:00:00Z", 9000.0)
], ["timestamp", "value"]).withColumn("group", lit("series1"))

# Run the Anomaly Detector service to look for irregular data
anamoly_detector = (SimpleDetectAnomalies()
  .setSubscriptionKey(anomaly_key)
  .setLocation("eastus")
  .setTimestampCol("timestamp")
  .setValueCol("value")
  .setOutputCol("anomalies")
  .setGroupbyCol("group")
  .setGranularity("monthly"))

# Show the full results of the analysis with the anomalies marked as "True"
display(anamoly_detector.transform(df).select("timestamp", "value", "anomalies.isAnomaly"))
```

### <a name="expected-result"></a>Résultat attendu

| timestamp            |   value | isAnomaly   |
|:---------------------|--------:|:------------|
| 1972-01-01T00:00:00Z |     826 | False       |
| 1972-02-01T00:00:00Z |     799 | False       |
| 1972-03-01T00:00:00Z |     890 | False       |
| 1972-04-01T00:00:00Z |     900 | False       |
| 1972-05-01T00:00:00Z |     766 | False       |
| 1972-06-01T00:00:00Z |     805 | False       |
| 1972-07-01T00:00:00Z |     821 | False       |
| 1972-08-01T00:00:00Z |   20000 | True        |
| 1972-09-01T00:00:00Z |     883 | False       |
| 1972-10-01T00:00:00Z |     898 | False       |
| 1972-11-01T00:00:00Z |     957 | False       |
| 1972-12-01T00:00:00Z |     924 | False       |
| 1973-01-01T00:00:00Z |     881 | False       |
| 1973-02-01T00:00:00Z |     837 | False       |
| 1973-03-01T00:00:00Z |    9000 | True        |

## <a name="arbitrary-web-apis"></a>API web arbitraires

Avec HTTP sur Spark, tout service web peut être utilisé dans votre pipeline de Big Data. Dans cet exemple, nous utilisons l’[API Banque mondiale](http://api.worldbank.org/v2/country/) pour obtenir des informations sur différents pays du monde entier.

```python
from requests import Request
from mmlspark.io.http import HTTPTransformer, http_udf
from pyspark.sql.functions import udf, col

# Use any requests from the python requests library
def world_bank_request(country):
  return Request("GET", "http://api.worldbank.org/v2/country/{}?format=json".format(country))

# Create a dataframe with spcificies which countries we want data on
df = (spark.createDataFrame([("br",),("usa",)], ["country"])
  .withColumn("request", http_udf(world_bank_request)(col("country"))))

# Much faster for big data because of the concurrency :)
client = (HTTPTransformer()
      .setConcurrency(3)
      .setInputCol("request")
      .setOutputCol("response"))

# Get the body of the response
def get_response_body(resp):
  return resp.entity.content.decode()

# Show the details of the country data returned
display(client.transform(df).select("country", udf(get_response_body)(col("response")).alias("response")))
```

### <a name="expected-result"></a>Résultat attendu

| country   | réponse |
|:----------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| br | [{"page":1,"pages":1,"per_page":"50","total":1},[{"id":"BRA","iso2Code":"BR","name":"Brésil","region":{"id":"LCN","iso2code":"ZJ","value":"Amérique latine et Antilles "},"adminregion":{"id":"LAC","iso2code":"XJ","value":"Amérique latine et Antilles (à l’exception des revenus élevés)"},"incomeLevel":{"id":"UMC","iso2code":"XT","value":"Revenus moyens supérieurs"},"lendingType":{"id":"IBD","iso2code":"XF","value":"IBRD"},"capitalCity":"Brésil","longitude":"-47.9292","latitude":"-15.7801"}]] |
| usa  | [{"page":1,"pages":1,"per_page":"50","total":1},[{"id":"USA","iso2Code":"US","name":"États-Unis","region":{"id":"NAC","iso2code":"XU","value":"Amérique du Nord"},"adminregion":{"id":"","iso2code":"","value":""},"incomeLevel":{"id":"HIC","iso2code":"XD","value":"Revenus élevés"},"lendingType":{"id":"LNX","iso2code":"XX","value":"Non classé"},"capitalCity":"Washington D.C.","longitude":"-77.032","latitude":"38.8895"}]] |

## <a name="see-also"></a>Voir aussi

* [Recette : Détection d’anomalie](./recipes/anomaly-detection.md)
* [Recette : Art Explorer](./recipes/art-explorer.md)