---
title: Exploration et analyse de texte avec l’API Analyse de texte - Azure Cognitive Services
titleSuffix: Azure Cognitive Services
description: Découvrez l’exploration de texte avec l’API Analyse de texte. Utilisez-la pour l’analyse de sentiments, la détection de langue et d’autres formes de traitement en langage naturel.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: overview
ms.date: 11/17/2020
ms.author: aahi
keywords: exploration de texte, analyse de sentiments, analyse de texte
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: 6cef9dc65a72134e0aa70db5f89f4934263c48b4
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563226"
---
# <a name="what-is-the-text-analytics-api"></a>Qu’est-ce que l’API Analyse de texte ?

L’API Analyse de texte est un service cloud qui fournit des fonctionnalités de traitement en langage naturel pour l’exploration de texte et l’analyse de texte, notamment l’analyse des sentiments, l’exploration des opinions, l’extraction de phrases clés, la détection de la langue et la reconnaissance des entités nommées.

L’API fait partie d’[Azure Cognitive Services](../index.yml), une collection d’algorithmes de machine learning et d’intelligence artificielle dans le cloud pour vos projets de développement. Vous pouvez utiliser ces fonctionnalités avec l’[API REST](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-V2-1/) ou la [bibliothèque de client](quickstarts/client-libraries-rest-api.md).

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Whats-New-in-Text-Analytics-Opinion-Mining-and-Async-API/player]

## <a name="sentiment-analysis"></a>analyse de sentiments

Utilisez l’[analyse des sentiments](how-tos/text-analytics-how-to-sentiment-analysis.md) et découvrez ce que les gens pensent de votre marque ou de votre thématique en explorant le texte pour trouver des indices sur les sentiments positifs ou négatifs. 

La fonctionnalité fournit des étiquettes de sentiment (comme « négatif », « neutre » et « positif ») en fonction du score de confiance le plus élevé trouvé par le service au niveau du document et de la phrase. Cette fonctionnalité retourne également des scores de confiance compris entre 0 et 1 pour chaque document, ainsi que les phrases qu’il contient pour les sentiments positif, neutre et négatif. Vous pouvez également exécuter le service localement [à l’aide d’un conteneur](how-tos/text-analytics-how-to-install-containers.md).

À partir de la préversion 3.1, l’exploration des opinions est une fonctionnalité d’Analyse des sentiments. Également connu sous le nom d’Analyse des sentiments basée sur l’aspect dans le registre du traitement en langage naturel, cette fonctionnalité fournit des informations plus granulaires sur les opinions liées aux aspects (tels que les attributs de produits ou de services) dans le texte.

## <a name="key-phrase-extraction"></a>Extraction d’expressions clés

Utilisez l’[extraction de phrases clés](how-tos/text-analytics-how-to-keyword-extraction.md) pour identifier rapidement les principaux concepts dans un texte. Par exemple, dans le texte « The food was delicious and there were wonderful staff » (La nourriture était délicieuse et le personnel adorable), l’extraction de phrases clés retourne les principaux points de discussion : « food » (nourriture) et « wonderful staff » (personnel adorable).

## <a name="language-detection"></a>Détection de la langue

La détection de la langue peut [détecter la langue dans laquelle un texte d’entrée est rédigé](how-tos/text-analytics-how-to-language-detection.md) et générer un code de langue unique pour chaque document envoyé dans la requête, ceci parmi une grande variété de langues, de variantes, de dialectes, et de quelques langues régionales ou de culture. Le code de langue est associé à un score de confiance.

## <a name="named-entity-recognition"></a>Reconnaissance d’entité nommée

La reconnaissance d’entités nommées (NER) peut [identifier et classer les entités](how-tos/text-analytics-how-to-entity-linking.md) dans votre texte en tant que personnes, lieux, organisations, quantités ; les entités bien connues sont également reconnues et liées à des informations supplémentaires sur le web.

## <a name="deploy-on-premises-using-docker-containers"></a>Déployer localement en utilisant des conteneurs Docker

[Utilisez des conteneurs Analyse de texte](how-tos/text-analytics-how-to-install-containers.md) pour déployer localement des fonctionnalités d’API. Ces conteneurs Docker vous donnent la possibilité de rapprocher le service plus près de vos données, ce qui peut être souhaitable pour des raisons de conformité, de sécurité ou opérationnelles. Analyse de texte offre les conteneurs suivants :

* analyse des sentiments
* Extraction de phrases clés (préversion)
* Détection de langue (préversion)
* Analyse de texte pour la santé (préversion)

## <a name="asynchronous-operations"></a>Opérations asynchrones

Le point de terminaison `/analyze` vous permet d’utiliser des fonctionnalités sélectionnées de l’API Analyse de texte [de façon asynchrone](how-tos/text-analytics-how-to-call-api.md), comme la reconnaissance d’entités nommées et l’extraction de phrases clés.

## <a name="typical-workflow"></a>Flux de travail classique

Le flux de travail est simple : vous envoyez des données pour analyse et traitez les résultats dans votre code. Les analyseurs sont utilisés tels quels, sans configuration ou personnalisation supplémentaires.

1. [Créez une ressource Azure](how-tos/text-analytics-how-to-call-api.md) pour Text Analytics. Ensuite, [récupérez la clé](how-tos/text-analytics-how-to-call-api.md) générée pour authentifier vos demandes.

2. [Formulez une demande](how-tos/text-analytics-how-to-call-api.md#json-schema) contenant vos données sous forme de texte brut non structuré en JSON.

3. Publiez la demande sur le point de terminaison établi pendant l’inscription, en ajoutant la ressource souhaitée : analyse des sentiments, extraction de phrases clés, détection de la langue ou reconnaissance des entités nommées.

4. Diffusez ou stockez la réponse localement. En fonction de la demande, les résultats sont un score de sentiment, une collection de phrases clés extraites ou un code de langue.

La sortie est renvoyée en tant que simple document JSON, avec des résultats pour chaque document texte que vous avez publié, en fonction de l’ID. Vous pouvez ensuite analyser, visualiser ou catégoriser les résultats en informations exploitables.

Les données ne sont pas stockées dans votre compte. Les opérations effectuées par l’API Text Analytics sont sans état, ce qui signifie que le texte que vous fournissez est traité et que les résultats sont renvoyés immédiatement.

## <a name="text-analytics-for-multiple-programming-experience-levels"></a>Text Analytics pour plusieurs niveaux d’expérience en programmation

Vous pouvez commencer à utiliser l’API Text Analytics dans votre processus, même si vous n’avez pas beaucoup d’expérience en programmation. Utilisez ces tutoriels pour savoir comment recourir à l’API pour analyser un texte de différentes façons en fonction de votre niveau d’expérience. 

* Expérience de programmation minimale requise :
    * [Extraire des informations dans Excel avec Analyse de texte et Power Automate](tutorials/extract-excel-information.md)
    * [Utiliser l’API Text Analytics et MS Flow pour identifier les sentiments des commentaires dans un groupe Yammer](/Yammer/integrate-yammer-with-other-apps/sentiment-analysis-flow-azure?bc=%2f%2fazure%2fbread%2ftoc.json&toc=%2f%2fazure%2fcognitive-services%2ftext-analytics%2ftoc.json)
    * [Intégrer Power BI à l’API Text Analytics pour analyser les commentaires des clients](tutorials/tutorial-power-bi-key-phrases.md)
* Expérience de programmation recommandée :
    * [Analyse des sentiments sur des données de streaming à l’aide d’Azure Databricks](/azure/databricks/scenarios/databricks-sentiment-analysis-cognitive-services?bc=%2f%2fazure%2fbread%2ftoc.json&toc=%2f%2fazure%2fcognitive-services%2ftext-analytics%2ftoc.json)
    * [Créer une application Flask pour la traduction de texte, l’analyse de sentiments et la synthèse vocale](../translator/tutorial-build-flask-app-translation-synthesis.md?bc=%2f%2fazure%2fbread%2ftoc.json&toc=%2f%2fazure%2fcognitive-services%2ftext-analytics%2ftoc.json)


<a name="supported-languages"></a>

## <a name="supported-languages"></a>Langues prises en charge

Cette section a été déplacée vers un article distinct pour une meilleure détectabilité. Pour accéder à ce contenu, voir [Langues prises en charge dans l’API Text Analytics](./language-support.md).

<a name="data-limits"></a>

## <a name="data-limits"></a>Limites de données

Tous les points de terminaison de l’API Analyse de texte acceptent des données en texte brut. Pour plus d’informations, consultez l’article [Limites de données](concepts/data-limits.md).

## <a name="unicode-encoding"></a>Codage Unicode

L’API Text Analytics utilise un codage Unicode pour la représentation textuelle et les calculs de nombre de caractères. Les demandes peuvent être soumises en code UTF-8 ou UTF-16, sans différence mesurable de nombre de caractères. Des points de code Unicode sont utilisés en guise d’heuristique pour la longueur de caractères, et sont considérés comme équivalents en ce qui concerne les limites de données pour Text Analytics. Si vous utilisez [`StringInfo.LengthInTextElements`](/dotnet/api/system.globalization.stringinfo.lengthintextelements) pour obtenir le nombre de caractères, vous recourez à la même méthode que nous pour mesurer la taille des données.

## <a name="next-steps"></a>Étapes suivantes

+ [Créez une ressource Azure](../cognitive-services-apis-create-account.md) pour Text Analytics afin d’obtenir une clé et un point de terminaison pour vos applications.

+ Utilisez le [démarrage rapide](quickstarts/client-libraries-rest-api.md) pour commencer à envoyer des appels d’API. Découvrez comment envoyer du texte, choisir une analyse et afficher les résultats avec un minimum de code.

+ Pour plus d’informations sur les nouvelles mises en production et fonctionnalités, voir [Nouveautés de l’API Text Analytics](whats-new.md).

+ Allez un peu plus loin avec ce [tutoriel d’analyse de sentiments](/azure/databricks/scenarios/databricks-sentiment-analysis-cognitive-services) à l’aide d’Azure Databricks.

+ Découvrez notre liste de billets de blog et de vidéos montrant comment utiliser l’API Text Analytics avec d’autres outils et technologies dans notre page [Contenu externe et de la communauté](text-analytics-resource-external-community.md).
