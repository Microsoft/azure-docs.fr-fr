---
title: Cognitive Services pour le Big Data - Exemples Scala
description: Utilisez Cognitive Services pour Azure Databricks afin d’exécuter votre pipeline MMLSpark pour le Big Data.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.topic: sample
ms.date: 07/06/2020
ms.author: marhamil
ms.openlocfilehash: c47aa803774343b39efeabe3452f1b256cc64c0d
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94363270"
---
# <a name="quick-examples"></a>Exemples rapides

Les extraits de code suivants sont prêts à être exécutés et vous permettent de vous familiariser avec l’utilisation de Cognitive Services sur Spark. Les exemples ci-dessous sont disponibles dans Scala.

Les exemples utilisent les services Cognitive Services suivants :

- Analyse de texte : obtenir le sentiment (ou l’humeur) d’un ensemble de phrases.
- Vision par ordinateur : obtenir les étiquettes (descriptions en un mot) associées à un ensemble d’images.
- Recherche d’images Bing : rechercher sur le web des images associées à une demande en langage naturel.
- Reconnaissance vocale : transcrire les fichiers audio pour extraire des transcriptions texte.
- Détecteur d’anomalies : détecter les anomalies dans des données de séries chronologiques.

## <a name="prerequisites"></a>Prérequis

1. Suivez les étapes décrites dans [Bien démarrer](getting-started.md) pour configurer votre environnement Azure Databricks et Cognitive Services. Ce tutoriel explique comment installer MMLSpark et comment créer votre cluster Spark dans Databricks.
1. Après avoir créé un notebook dans Azure Databricks, copiez le **code partagé** ci-dessous et collez-le dans une nouvelle cellule de votre notebook.
1. Choisissez un exemple de service, ci-dessous, puis copiez-le et collez-le dans une deuxième nouvelle cellule de votre notebook.
1. Remplacez l’un des espaces réservés de clé d’abonnement au service par votre propre clé.
1. Cliquez sur le bouton Exécuter (icône de triangle) en haut à gauche de la cellule, puis sélectionnez **Exécuter la cellule**.
1. Consultez les résultats dans un tableau affiché sous la cellule.

## <a name="shared-code"></a>Code partagé
Pour bien démarrer, ajoutez ce code à votre projet :

```scala
import com.microsoft.ml.spark.cognitive._
import spark.implicits._ 

val serviceKey = "ADD-YOUR-SUBSCRIPTION-KEY"
val location = "eastus"
```

## <a name="text-analytics"></a>Analyse de texte

Le service [Analyse de texte](../text-analytics/index.yml) fournit plusieurs algorithmes permettant d’extraire des insights intelligents à partir d’un texte. Par exemple, nous pouvons trouver le sentiment d’un texte d’entrée donné. Le service retourne un score compris entre `0.0` et `1.0`, où un score faible indique un sentiment négatif et où un score élevé indique un sentiment positif.  L’exemple ci-dessous utilise trois phrases simples et retourne le score de sentiment pour chacune d’elles.

```scala
import org.apache.spark.sql.functions.col

val df = Seq(
  ("I am so happy today, its sunny!", "en-US"),
  ("I am frustrated by this rush hour traffic", "en-US"),
  ("The cognitive services on spark aint bad", "en-US")
).toDF("text", "language")

val sentiment = new TextSentiment()
    .setTextCol("text")
    .setLocation(location)
    .setSubscriptionKey(serviceKey)
    .setOutputCol("sentiment")
    .setErrorCol("error")
    .setLanguageCol("language")

display(sentiment.transform(df).select(col("text"), col("sentiment")(0).getItem("score").alias("sentiment")))
```

### <a name="expected-result"></a>Résultat attendu

| texte                                      | sentiment                                             |
|:------------------------------------------|:------------------------------------------------------|
| I am so happy today, its sunny! (Je suis trop content aujourd’hui, il fait beau !)           | 0.9789592027664185                                    |
| I am frustrated by this rush hour traffic (Cette circulation aux heures de pointe m’agace) | 0.023795604705810547                                  |
| The cognitive services on spark aint bad (Les services Cognitive Services sur Spark ne sont pas mal)  | 0.8888956308364868                                    |

## <a name="computer-vision"></a>Vision par ordinateur

Le service [Vision par ordinateur](../computer-vision/index.yml) analyse des images pour identifier une structure comme des visages, des objets et des descriptions en langage naturel.
Dans cet exemple, nous ajoutons des étiquettes à une liste d’images. Les étiquettes sont des descriptions en un mot des choses figurant sur l’image, comme des objets, des personnes, un paysage et des actions reconnaissables.

```scala
// Create a dataframe with the image URLs
val df = Seq(
    ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg"),
    ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/dog.jpg"),
    ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/house.jpg")
).toDF("image")

// Run the Computer Vision service. Analyze Image extracts infortmation from/about the images.
val analysis = new AnalyzeImage()
    .setLocation(location)
    .setSubscriptionKey(serviceKey)
    .setVisualFeatures(Seq("Categories","Color","Description","Faces","Objects","Tags"))
    .setOutputCol("results")
    .setImageUrlCol("image")
    .setErrorCol("error"))

// Show the results of what you wanted to pull out of the images.
display(analysis.transform(df).select(col("image"), col("results").getItem("tags").getItem("name")).alias("results")))

// Uncomment for full results with all visual feature requests
//display(analysis.transform(df).select(col("image"), col("results")))
``` 

### <a name="expected-result"></a>Résultat attendu

| image | tags |
|:------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------|
| https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg | [’skate' 'personne' 'homme' 'extérieur' 'monter' 'sport' 'skateboard' 'jeune' 'planche' 'maillot' 'air' 'noir' 'parc' 'garçon' 'côté' 'saut' 'figure' 'rampe' 'action' 'vol']
| https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/dog.jpg | ['chien' 'extérieur' 'barrière' 'bois' 'petit' 'marron' 'bâtiment' 'assis' 'devant' 'banc' 'debout' 'table' 'randonnée' 'planche' 'plage' 'blanc' 'soutien' 'pont' 'piste']                
| https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/house.jpg | ['extérieur' 'pelouse' 'maison' 'bâtiment' 'ancien' 'foyer' 'devant' 'petit' 'église' 'pierre' 'grand' 'pâturage' 'cour' 'vert' 'assis' 'premier plan' 'mouton' 'brique' 'banc' 'rue' 'white' 'blanc' 'horloge' 'panneau' 'garé' 'champ' 'debout' 'jardin' 'eau' 'rouge' 'cheval' 'homme' 'grand' 'feu' 'groupe'] 

## <a name="bing-image-search"></a>Recherche d’images Bing

La [Recherche d’images Bing](../bing-image-search/overview.md) explore le web pour récupérer des images associées à une requête en langage naturel d’un utilisateur. Dans cet exemple, nous utilisons une requête de texte qui recherche des images avec des guillemets. Elle retourne une liste d’URL d’images qui contiennent des photos liées à notre requête.


```scala
import org.apache.spark.ml.Pipeline

// Number of images Bing will return per query
val imgsPerBatch = 10

// A list of offsets, used to page into the search results
val df = (0 until 100).map(o => Tuple1(o*imgsPerBatch)).toSeq.toDF("offset")

// Run the Bing Image Search service with our text query
val bingSearch = new BingImageSearch()
    .setSubscriptionKey(bingSearchKey)
    .setOffsetCol("offset")
    .setQuery("Martin Luther King Jr. quotes")
    .setCount(imgsPerBatch)
    .setOutputCol("images")

// Transformer that extracts and flattens the richly structured output of Bing Image Search into a simple URL column
val getUrls = BingImageSearch.getUrlTransformer("images", "url")

// This displays the full results returned, uncomment to use
// display(bingSearch.transform(bingParameters))

// Since we have two services, they are put into a pipeline
val pipeline = new Pipeline().setStages(Array(bingSearch, getUrls))

// Show the results of your search: image URLs
display(pipeline.fit(df).transform(df))
```

### <a name="expected-result"></a>Résultat attendu

| url |
|:-------------------------------------------------------------------------------------------------------------------|
| https://iheartintelligence.com/wp-content/uploads/2019/01/powerful-quotes-martin-luther-king-jr.jpg      |
| http://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes-16.jpg             |
| http://www.sofreshandsogreen.com/wp-content/uploads/2012/01/martin-luther-king-jr-quote-sofreshandsogreendotcom.jpg |
| https://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes-18.jpg            |
| https://tsal-eszuskq0bptlfh8awbb.stackpathdns.com/wp-content/uploads/2018/01/MartinLutherKingQuotes.jpg  |

## <a name="speech-to-text"></a>Reconnaissance vocale

Le service [Reconnaissance vocale](../speech-service/index-speech-to-text.yml) convertit des flux ou des fichiers de contenu audio en texte. Dans cet exemple, nous transcrivons deux fichiers audio. Le premier fichier est facile à comprendre et le second est plus complexe.

```scala
import org.apache.spark.sql.functions.col

// Create a dataframe with audio URLs, tied to the column called "url"
val df = Seq(("https://mmlspark.blob.core.windows.net/datasets/Speech/audio2.wav"),
             ("https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3")).toDF("url")

// Run the Speech-to-text service to translate the audio into text
val speechToText = new SpeechToTextSDK()
    .setSubscriptionKey(serviceKey)
    .setLocation("eastus")
    .setOutputCol("text")
    .setAudioDataCol("url")
    .setLanguage("en-US")
    .setProfanity("Masked")

// Show the results of the translation
display(speechToText.transform(df).select(col("url"), col("text").getItem("DisplayText")))
```

### <a name="expected-result"></a>Résultat attendu

| url | DisplayText |
|:------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio2.wav | Custom Speech propose des outils qui vous permettent d’inspecter visuellement la qualité de la reconnaissance d’un modèle en comparant les données audio au résultat de la reconnaissance correspondante provenant du portail Custom Speech. Vous pouvez lire un contenu audio chargé pour déterminer si le résultat proposé de la reconnaissance est correct. Cet outil vous permet d’inspecter rapidement la qualité du modèle de base de référence de reconnaissance vocale de Microsoft ou d’un modèle personnalisé entraîné sans qu’il soit nécessaire de transcrire des données audio. |
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3 | Ajoutez un contrôle visuel des pensées d’un monsieur.    |
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3 | Je m’entends. |
| https://mmlspark.blob.core.windows.net/datasets/Speech/audio3.mp3 | J’aime avoir la garantie par la radio que je peux également l’entendre. |

## <a name="anomaly-detector"></a>Le détecteur d’anomalies

Le [Détecteur d’anomalies](../anomaly-detector/index.yml) est idéal pour détecter les irrégularités dans vos données de séries chronologiques. Dans cet exemple, nous utilisons le service pour rechercher des anomalies dans l’ensemble de la série chronologique.

```scala
import org.apache.spark.sql.functions.{col, lit}

val anomalyKey = "84a2c303cc7e49f6a44d692c27fb9967"

val df = Seq(
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
  ).toDF("timestamp", "value").withColumn("group", lit("series1"))

// Run the Anomaly Detector service to look for irregular data
val anamolyDetector = new SimpleDetectAnomalies()
    .setSubscriptionKey(anomalyKey)
    .setLocation("eastus")
    .setTimestampCol("timestamp")
    .setValueCol("value")
    .setOutputCol("anomalies")
    .setGroupbyCol("group")
    .setGranularity("monthly")

// Show the full results of the analysis with the anomalies marked as "True"
display(anamolyDetector.transform(df).select("timestamp", "value", "anomalies.isAnomaly"))
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