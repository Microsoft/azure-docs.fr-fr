---
title: Cognitive Services pour le Big Data
description: Découvrez comment tirer parti d’Azure Cognitive Services sur les jeux de données volumineux à l’aide de Python, Java et Scala. Avec Cognitive Services pour le Big Data, vous pouvez incorporer des modèles toujours plus intelligents et plus performants dans les calculs Apache Spark&trade; et SQL.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.topic: overview
ms.date: 07/09/2020
ms.author: marhamil
ms.openlocfilehash: aaade03edbbb109656fb7371a063cdc2512c5a20
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461629"
---
# <a name="azure-cognitive-services-for-big-data"></a>Azure Cognitive Services pour le Big Data

![Azure Cognitive Services pour le Big Data](media/cognitive-services-big-data-overview.svg)

Azure Cognitive Services pour le Big Data permet aux utilisateurs de canaliser des téraoctets de données à travers Cognitive Services à l’aide d’[Apache Spark&trade;](/dotnet/spark/what-is-spark). Avec Cognitive Services pour le Big Data, il est facile de créer des applications intelligentes à grande échelle avec n’importe quel magasin de données.

Avec Cognitive Services pour le Big Data, vous pouvez incorporer des modèles toujours plus intelligents et plus performants dans les calculs Apache Spark&trade; et SQL. Ces outils libèrent les développeurs des tâches réseau de bas niveau, qui peuvent ainsi se concentrer sur la création d’applications intelligentes et distribuées.

## <a name="features-and-benefits"></a>Fonctionnalités et avantages

Cognitive Services pour le Big Data peut utiliser des services de n’importe quelle région du monde, ainsi que des [services cognitifs conteneurisés](../cognitive-services-container-support.md). Les conteneurs prennent en charge des déploiements à connectivité faible ou inexistante avec des réponses à très faible latence. Les services cognitifs conteneurisés peuvent être exécutés localement, directement sur les nœuds worker de votre cluster Spark, ou sur un orchestrateur externe comme Kubernetes.

## <a name="supported-services"></a>Services pris en charge

[Cognitive Services](../index.yml), accessible par le biais des API et des SDK, aide les développeurs à créer des applications intelligentes sans avoir de compétences en IA ou en science des données. Avec Cognitive Services, vous pouvez faire en sorte que vos applications puissent voir, entendre, parler, comprendre et raisonner. Pour utiliser Cognitive Services, votre application doit envoyer des données au service sur le réseau. Une fois ces données reçues, le service envoie une réponse intelligente en retour. Les services suivants sont disponibles pour les charges de travail de Big Data :

### <a name="vision"></a>Vision

|Nom du service|Description du service|
|:-----------|:------------------|
|[Vision par ordinateur](../computer-vision/index.yml "Vision par ordinateur")| Le service Vision par ordinateur vous donne accès à des algorithmes avancés pour le traitement d’images et le renvoi d’informations. |
|[Visage](../face/index.yml "Face")| Le service Visage donne accès à des algorithmes d’analyse du visage permettant la détection et la reconnaissance d’attributs faciaux. |

### <a name="speech"></a>Speech

|Nom du service|Description du service|
|:-----------|:------------------|
|[Service Speech](../speech-service/index.yml "Service Speech")|Le service Speech donne accès à des fonctionnalités telles que la reconnaissance vocale, la synthèse vocale, la traduction vocale, et la vérification et l’identification de l’orateur.|

### <a name="decision"></a>Décision

|Nom du service|Description du service|
|:-----------|:------------------|
|[Détecteur d’anomalies](../anomaly-detector/index.yml "Détecteur d’anomalies") | Le service Détecteur d’anomalies (préversion) vous permet de superviser et de détecter des anomalies dans vos données de série chronologique.|

### <a name="language"></a>Langage

|Nom du service|Description du service|
|:-----------|:------------------|
|[Analyse de texte](../text-analytics/index.yml "Analyse de texte")| Le service Analyse de texte effectue un traitement en langage naturel sur du texte brut pour l’analyse des sentiments, l’extraction de phrases clés et la détection de langue.|

### <a name="search"></a>Recherche

|Nom du service|Description du service|
|:-----------|:------------------|
|[Recherche d’images Bing](/azure/cognitive-services/bing-image-search "Recherche d’images Bing")|Le service Recherche d’images Bing affiche des images jugées pertinentes en lien avec la requête d’un utilisateur.|

## <a name="supported-programming-languages-for-cognitive-services-for-big-data"></a>Langages de programmation pris en charge pour Cognitive Services pour le Big Data

Cognitive Services pour le Big Data repose sur Apache Spark. Apache Spark est une bibliothèque de calcul distribuée qui prend en charge Java, Scala, Python, R et bien d’autres langages. Les langages suivants sont actuellement pris en charge.

### <a name="python"></a>Python

Nous fournissons une API PySpark dans l’espace de noms `mmlspark.cognitive` de [Microsoft ML pour Apache Spark](https://aka.ms/spark). Pour plus d’informations, consultez l’[API pour développeurs Python](https://mmlspark.blob.core.windows.net/docs/1.0.0-rc1/pyspark/mmlspark.cognitive.html). Pour obtenir des exemples d’utilisation, consultez les [Exemples Python](samples-python.md).

### <a name="scala-and-java"></a>Scala et Java

Nous fournissons une API Spark basée sur Scala et Java dans l’espace de noms `com.microsoft.ml.spark.cognitive` de [Microsoft ML pour Apache Spark](https://aka.ms/spark). Pour plus d’informations, consultez l’[API pour développeurs Scala](https://mmlspark.blob.core.windows.net/docs/1.0.0-rc1/scala/index.html#package). Pour obtenir des exemples d’utilisation, consultez les [Exemples Scala](samples-scala.md).

## <a name="supported-platforms-and-connectors"></a>Plateformes et connecteurs pris en charge

Cognitive Services pour le Big Data nécessite Apache Spark. Plusieurs plateformes Apache Spark prennent en charge Cognitive Services pour le Big Data.

### <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks) est une plateforme d’analytique basée sur Apache Spark et optimisée pour la plateforme de services cloud Microsoft Azure. Elle fournit une configuration en un clic, des workflows simplifiés et un espace de travail interactif permettant aux scientifiques des données, aux ingénieurs Données et aux analystes métier de collaborer.

### <a name="azure-synapse-analytics"></a>Azure Synapse Analytics

[Azure Synapse Analytics](/azure/databricks/data/data-sources/azure/synapse-analytics) est un entrepôt de données d’entreprise qui utilise un traitement parallèle massif. Avec Synapse Analytics, vous pouvez rapidement exécuter des requêtes complexes sur plusieurs pétaoctets de données. Azure Synapse Analytics fournit des pools Spark managés pour exécuter des travaux Spark avec une interface Jupyter Notebook intuitive.

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

[Azure Kubernetes service (AKS)](../../aks/index.yml) orchestre les conteneurs Docker et les applications distribuées à grande échelle. AKS est une offre Kubernetes managée qui simplifie l’utilisation de Kubernetes dans Azure. Kubernetes peut permettre un contrôle précis de l’échelle, de la latence et du réseau Cognitive Services. Toutefois, nous vous recommandons d’utiliser Azure Databricks ou Azure Synapse Analytics si vous ne connaissez pas Apache Spark.

### <a name="data-connectors"></a>Connecteurs de données

Une fois que vous avez un cluster Spark, l’étape suivante consiste à vous connecter à vos données. Apache Spark dispose d’une vaste collection de connecteurs de base de données. Ces connecteurs permettent aux applications de travailler avec des jeux de données volumineux, quel que soit l’endroit où ils sont stockés. Pour plus d’informations sur les bases de données et les connecteurs pris en charge, consultez la [liste des sources de données prises en charge pour Azure Databricks](/azure/databricks/data/data-sources/).

## <a name="concepts"></a>Concepts

### <a name="spark"></a>Spark

[Apache Spark&trade;](http://spark.apache.org/) est un moteur d’analytique unifié pour le traitement des données à grande échelle. Son infrastructure de traitement parallèle améliore les performances des applications analytiques et du Big Data. Spark peut fonctionner à la fois en tant que système de traitement par lot et de flux, sans changement du code d’application principal.

La base de Spark est le DataFrame, une collection tabulaire de données distribuées parmi les nœuds worker Apache Spark. Un DataFrame Spark est semblable à une table dans une base de données relationnelle ou à une trame de données en R/Python, mais avec une mise à l’échelle illimitée. Les DataFrames peuvent être construits à partir de nombreuses sources, telles que des fichiers de données structurées, des tables dans Hive ou des bases de données externes. Une fois que vos données se trouvent dans un DataFrame Spark, vous pouvez :

   - Effectuer des calculs de style SQL comme la jointure ou le filtrage de tables.
   - Appliquer des fonctions à des jeux de données volumineux à l’aide du parallélisme de style MapReduce.
   - Appliquer le Machine Learning distribué à l’aide de Microsoft Machine Learning pour Apache Spark.
   - Utiliser Cognitive Services pour le Big Data afin d’enrichir vos données avec des services intelligents prêts à l’emploi.

### <a name="microsoft-machine-learning-for-apache-spark-mmlspark"></a>Microsoft Machine Learning pour Apache Spark (MMLSpark)

[Microsoft Machine Learning pour Apache Spark](https://mmlspark.blob.core.windows.net/website/index.html#install) (MMLSpark) est une bibliothèque de Machine Learning distribuée, open source et basée sur Apache Spark. Cognitive Services pour le Big Data est inclus dans ce package. De plus, MMLSpark contient plusieurs autres outils ML pour Apache Spark, tels que LightGBM, Vowpal Wabbit, OpenCV, LIME et plus encore. Avec MMLSpark, vous pouvez générer des modèles prédictifs et analytiques puissants à partir de n’importe quelle source de données Spark.

### <a name="http-on-spark"></a>HTTP sur Spark

Cognitive Services pour le Big Data est un exemple de la façon dont nous pouvons intégrer des services web intelligents au Big Data. Les services web pilotent de nombreuses applications dans le monde entier, et la plupart des services communiquent par le biais du protocole HTTP (Hypertext Transfer Protocol). Pour travailler avec des services web *arbitraires* à grande échelle, nous fournissons HTTP sur Spark. Avec HTTP sur Spark, vous pouvez passer des téraoctets de données par le biais de n’importe quel service web. En coulisses, nous utilisons cette technologie pour piloter Cognitive Services pour le Big Data.

## <a name="developer-samples"></a>Exemples de développement

- [Recette : Maintenance prédictive](recipes/anomaly-detection.md)
- [Recette : Exploration intelligente d’images](recipes/art-explorer.md)

## <a name="blog-posts"></a>Billets de blog :

- [En savoir plus sur le fonctionnement de Cognitive Services sur Apache Spark&trade;](https://azure.microsoft.com/blog/dear-spark-developers-welcome-to-azure-cognitive-services/)
- [Protection des léopards des neiges avec le Deep Learning et Vision par ordinateur sur Spark](http://www.datawizard.io/2017/06/27/saving-snow-leopards-with-deep-learning-and-computer-vision-on-spark/)
- [Podcast Microsoft Research : MMLSpark, AI for Good avec Mark Hamilton](https://blubrry.com/microsoftresearch/49485070/092-mmlspark-empowering-ai-for-good-with-mark-hamilton/)
- [Livre blanc universitaire : Large Scale Intelligent Microservices](https://arxiv.org/abs/2009.08044)

## <a name="webinars-and-videos"></a>Webinaires et vidéos

- [Azure Cognitive Services sur Spark : Clusters avec services intelligents incorporés](https://databricks.com/session/the-azure-cognitive-services-on-spark-clusters-with-embedded-intelligent-services)
- [Discours d’ouverture Spark Summit : AI for Good scalable](https://databricks.com/session_eu19/scalable-ai-for-good)
- [Cognitive Services pour le Big Data dans Cosmos DB](https://medius.studios.ms/Embed/Video-nc/B19-BRK3004?latestplayer=true&l=2571.208093)
- [Coup de projecteur sur les microservices intelligents à grande échelle](https://www.youtube.com/watch?v=BtuhmdIy9Fk&t=6s)

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer avec Cognitive Services pour le Big Data](getting-started.md)
- [Exemples Python simples](samples-python.md)
- [Exemples Scala simples](samples-scala.md)