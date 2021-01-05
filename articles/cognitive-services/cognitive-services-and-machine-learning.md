---
title: Cognitive Services et machine learning
titleSuffix: Azure Cognitive Services
description: Découvrez où Azure Cognitive Services s’intègre avec d’autres produits Azure pour le machine learning.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/22/2019
ms.openlocfilehash: 4ab32cd37d732cd9f177363da547edd22bd5af77
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349263"
---
# <a name="cognitive-services-and-machine-learning"></a>Cognitive Services et machine learning

Cognitive Services fournit des fonctionnalités de Machine Learning pour résoudre des problèmes généraux tels que l’analyse de texte pour inférer des sentiments, ou l’analyse d’images pour reconnaître des objets ou visages. Pour utiliser ces services, vous n’avez pas besoin de connaissances spéciales en matière de Machine Learning ou de science des données. 

[Cognitive services](./what-are-cognitive-services.md) est un groupe de services dont chacun prend en charge des fonctionnalités de prédiction généralisées différentes. Les services sont divisés en différentes catégories pour vous aider à trouver le service approprié. 

|Catégorie de service|Objectif|
|--|--|
|[Décision](https://azure.microsoft.com/services/cognitive-services/directory/decision/)|Créez des applications qui fournissent des recommandations pour une prise de décision éclairée et efficace.|
|[Langage](https://azure.microsoft.com/services/cognitive-services/directory/lang/)|Autorisez vos applications à traiter le langage naturel avec des scripts préconstruits, à évaluer les sentiments et à reconnaître les attentes des utilisateurs.|
|[action](https://azure.microsoft.com/services/cognitive-services/directory/search/)|Ajoutez des API Recherche Bing à vos applications et exploitez la possibilité de passer au crible des milliards de pages web, images, vidéos et actualités avec un seul appel d’API.|
|[Speech](https://azure.microsoft.com/services/cognitive-services/directory/speech/)|Convertissez de la parole en texte et du texte en parole naturelle. Traduisez d’une langue à une autre, et activez la vérification et la reconnaissance de l’orateur.|
|[Vision](https://azure.microsoft.com/services/cognitive-services/directory/vision/)|Identifiez, légendez, indexez et modérez le contenu de vos photos, vidéos et entrées manuscrites.|
||||

Utilisez Cognitive Services dans les cas suivants :

* Vous pouvez utiliser une solution généralisée.
* Vous accédez à la solution à partir d’une API REST de programmation ou d’un Kit de développement logiciel (SDK). 

Utilisez une autre solution de Machine Learning dans le cas suivant :

* Vous devez choisir l’algorithme et effectuer l’apprentissage sur des données très spécifiques.

## <a name="what-is-machine-learning"></a>L’apprentissage automatique - De quoi s’agit-il ?

Le Machine Learning consiste à associer des données et un algorithme pour répondre à un besoin spécifique. Une fois les données et l’algorithme formés, vous obtenez un modèle réutilisable avec d’autres données. Le modèle formé fournit des insights basés sur les nouvelles données. 

Le processus de création d’un système de Machine Learning nécessite une certain connaissance en matière de Machine Learning ou de science des données.

Le Machine Learning est fourni au travers des [produits et services Azure Machine Learning (AML)](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning?context=azure%2fmachine-learning%2fstudio%2fcontext%2fml-context).

## <a name="what-is-a-cognitive-service"></a>Qu’est-ce qu’un service cognitif ?

Un service cognitif fournit tout ou partie des composants d’une solution de Machine Learning : données, algorithme et modèle formé. Ces services requièrent une connaissance générale de vos données mais ne nécessitent aucune expérience en Machine Learning ou en science des données. Ces services fournissent des API REST et des kits de développement logiciel (SDK) basés sur le langage. Par conséquent, pour utiliser les services, vous devez avoir connaissance du langage de programmation.

## <a name="how-are-cognitive-services-and-azure-machine-learning-aml-similar"></a>En quoi Cognitive Services et Azure Machine Learning (AML) sont-ils similaires ?

Les deux solutions ont pour objectif final d’utiliser l’intelligence artificielle (IA) pour améliorer les opérations métier, même si les modes opératoires dans leurs offres respectives diffèrent. 

En règle générale, les audiences sont différentes :

* Les solutions Cognitive Services sont destinées aux développeurs sans expérience du Machine Learning.
* Azure Machine Learning est conçu pour les scientifiques des données. 

## <a name="how-is-a-cognitive-service-different-from-machine-learning"></a>En quoi un service cognitif se distingue-t-il du Machine Learning ?

Un service cognitif fournit un modèle formé pour vous. Celui-ci associe des données et un algorithme, disponibles à partir d’une ou plusieurs API REST ou d’un Kit de développement logiciel (SDK). Vous pouvez implémenter ce service en quelques minutes, selon votre scénario.  Un service cognitif fournit des réponses à des problèmes généraux, telles que des expressions clés dans du texte ou l’identification d’élément dans des images. 

Le Machine Learning est un processus dont l’implémentation nécessite généralement plus de temps. Ce temps est consacré à la collecte, au nettoyage et à la transformation des données, à la sélection de l’algorithme, ainsi qu’à la formation puis au déploiement du modèle afin d’obtenir le même niveau de fonctionnalité qu’un service cognitif. Le Machine Learning permet de fournir des réponses à des problèmes hautement spécialisés et/ou spécifiques. Les problèmes de Machine Learning nécessitent une connaissance de l’objet et des données propres au problème en question, ainsi qu’un savoir-faire en matière de science des données.

## <a name="what-kind-of-data-do-you-have"></a>De quel type de données disposez-vous ?

Cognitive Services, en tant que groupe de services, peut ne nécessiter aucune des données personnalisées ou nécessiter tout ou partie de celles-ci pour le modèle formé. 

### <a name="no-additional-training-data-required"></a>Aucune donnée d’entraînement supplémentaire requise

Les services qui fournissent un modèle entièrement entraîné peuvent être traités comme une _boîte opaque_. Vous n’avez pas besoin de savoir comment ils fonctionnent ou quelles données ont été utilisées pour les former. Vous apportez vos données à un modèle entièrement formé pour obtenir une prédiction. 

### <a name="some-or-all-training-data-required"></a>Tout ou partie des données d’entraînement requises

Certains services vous permettent d’apporter vos propres données, puis d’effectuer l’apprentissage d’un modèle. Cela vous permet d’étendre le modèle à l’aide des données et de l’algorithme du service avec vos propres données. La sortie correspond à vos besoins. Lorsque vous apportez vos propres données, il se peut que vous deviez baliser celles-ci d’une manière spécifique pour le service. Par exemple, si vous effectuez l’apprentissage d’un modèle pour identifier des fleurs, vous pouvez fournir un catalogue d’images de fleurs avec l’emplacement de la fleur dans chaque image pour former le modèle. 

Un service peut vous _permettre_ de fournir des données pour améliorer ses propres données. Un service peut _exiger_ que vous fournissiez des données. 

### <a name="real-time-or-near-real-time-data-required"></a>Données en temps réel ou en quasi temps réel requises

Un service peut nécessiter des données en temps réel ou en quasi temps réel pour créer un modèle efficace. Ces services traitent des quantités significatives de données de modèle. 

## <a name="service-requirements-for-the-data-model"></a>Exigences de service pour le modèle de données

Les données suivantes catégorisent chaque service selon le type de données qu’il autorise ou requiert.

|Service cognitif|Aucune donnée d’entraînement requise|Vous fournissez tout ou partie des données d’entraînement|Collecte de données en temps réel ou en quasi temps réel|
|--|--|--|--|
|[Détecteur d’anomalies](./Anomaly-Detector/overview.md)|x|x|x|
|Bing Search |x|||
|[Vision par ordinateur](./computer-vision/overview.md)|x|||
|[Content Moderator](./Content-Moderator/overview.md)|x||x|
|[Custom Vision](./custom-vision-service/overview.md)||x||
|[Visage](./Face/Overview.md)|x|x||
|[Form Recognizer](./form-recognizer/overview.md)||x||
|[Lecteur immersif](./immersive-reader/overview.md)|x|||
|[Ink Recognizer](./Ink-recognizer/overview.md)|x|x||
|[Language Understanding (LUIS)](./LUIS/what-is-luis.md)||x||
|[Personalizer](./personalizer/what-is-personalizer.md)|x*|x*|x|
|[QnA Maker](./QnAMaker/Overview/overview.md)||x||
|[Module de reconnaissance de l’orateur](./speech-service/speaker-recognition-overview.md)||x||
|[Synthèse vocale (TTS) Speech](speech-service/text-to-speech.md)|x|x||
|[Reconnaissance vocale (STT) Speech](speech-service/speech-to-text.md)|x|x||
|[Traduction vocale](speech-service/speech-translation.md)|x|||
|[Analyse de texte](./text-analytics/overview.md)|x|||
|[Translator](./translator/translator-info-overview.md)|x|||
|[Translator – Custom Translator](./translator/custom-translator/overview.md)||x||

*Personalizer n’a besoin que des données d’entraînement collectées par le service (pendant qu’il fonctionne en temps réel) pour évaluer votre stratégie et vos données. Personalizer n’a pas besoin de jeux de données historiques volumineux pour une formation initiale ou par lot. 

## <a name="where-can-you-use-cognitive-services"></a>Où utiliser Cognitive Services ?
 
Les services sont utilisés dans toute application pouvant effectuer des appels d’API REST ou de Kit de développement logiciel (SDK). Des exemples d’applications sont les sites web, les bots, la réalité virtuelle ou mixte, ou les applications de bureau et mobiles. 

## <a name="how-is-azure-cognitive-search-related-to-cognitive-services"></a>Quel est le lien entre Recherche cognitive Azure et Cognitive Services ?

[Recherche cognitive Azure](../search/search-what-is-azure-search.md) est un service de recherche cloud distinct qui utilise facultativement Cognitive Services pour ajouter un traitement des images et du langage naturel à l’indexation des charges de travail. Cognitive services est exposé dans Recherche cognitive Azure par le biais de [compétences intégrées](../search/cognitive-search-predefined-skills.md) qui encapsulent des API individuelles. Vous pouvez utiliser une ressource gratuite pour les procédures pas à pas, mais prévoyez de créer et de joindre une [ressource facturable](../search/cognitive-search-attach-cognitive-services.md) pour des volumes plus importants.

## <a name="how-can-you-use-cognitive-services"></a>Comment utiliser Cognitive Services ?

Chaque service fournit des informations sur vos données. Vous pouvez combiner des services pour chaîner des solutions telles que la conversion de parole (audio) en texte, la traduction de texte dans de nombreuses langues, puis l’utilisation des langues traduites pour obtenir des réponses d’une base de connaissances. Si les Cognitive Services peuvent être utilisés pour créer des solutions intelligentes en toute autonomie, ils peuvent également être combinés avec des projets de Machine Learning traditionnels pour compléter des modèles ou accélérer le processus de développement. 

Cognitive Services qui fournissent des modèles exportés pour d’autres outils de Machine Learning :

|Service cognitif|Informations sur le modèle|
|--|--|
|[Custom Vision](./custom-vision-service/overview.md)|[Exportation](./Custom-Vision-Service/export-model-python.md) pour Tensorflow pour Android, CoreML pour iOS 11, ONNX pour Windows ML|

## <a name="learn-more"></a>En savoir plus

* [Guide d'architecture - Quels sont les produits de machine learning proposés par Microsoft ?](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)
* [Machine learning - Présentation du deep learning par rapport au machine learning](../machine-learning/concept-deep-learning-vs-machine-learning.md)

## <a name="next-steps"></a>Étapes suivantes

* Créez votre compte Cognitive Services dans le [portail Azure](cognitive-services-apis-create-account.md) ou avec [Azure CLI](./cognitive-services-apis-create-account-cli.md).
* Découvrez comment vous [authentifier](authentication.md) sur un service cognitif.
* Utilisez la [journalisation des diagnostics](diagnostic-logging.md) pour l’identification et le débogage des problèmes. 
* Déployez un service cognitif dans un [conteneur](cognitive-services-container-support.md) Docker.
* Restez à jour avec les [mises à jour de service](https://azure.microsoft.com/updates/?product=cognitive-services).