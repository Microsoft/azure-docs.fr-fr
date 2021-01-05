---
title: Qu’est-ce que l’API Recherche d’actualités Bing ?
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API Recherche d’actualités Bing pour rechercher sur le web des articles actuels à travers plusieurs catégories, notamment des articles et rubriques tendances.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: overview
ms.date: 12/18/2019
ms.author: scottwhi
ms.custom: seodec2018
ms.openlocfilehash: c6fad3a006d2f3da74638e0680d6ba65f736ab7b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351229"
---
# <a name="what-is-the-bing-news-search-api"></a>Qu’est-ce que l’API Recherche d’actualités Bing ?

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’actualités Bing facilite l’intégration des capacités de recherche d’actualités cognitive de Bing dans vos applications. L’API offre une expérience similaire à [Bing Actualités](https://www.bing.com/news), en vous permettant d’envoyer des requêtes de recherche et de recevoir des articles d’actualité pertinents.

N’oubliez pas que l’API Recherche d’actualités Bing retourne uniquement des actualités dans les résultats de la recherche. Pour rechercher d’autres types de contenus web, utilisez l’[API Recherche Web Bing](../bing-web-search/overview.md), l’[API Recherche de vidéos Bing](../bing-video-search/overview.md) et l’[API Recherche d’images Bing](../bing-image-search/overview.md).

## <a name="bing-news-search-api-features"></a>Fonctionnalités de l’API Recherche d’actualités Bing

L’API Recherche d’actualités Bing recherche et retourne naturellement des articles d’actualité pertinents, mais elle fournit aussi plusieurs fonctionnalités de récupération d’actualités intelligente et ciblée sur le web.

|Caractéristique  |Description  |
|---------|---------|
|[Suggestion et utilisation de termes de recherche](concepts/search-for-news.md#suggest-and-use-search-terms)     | Améliorez votre expérience de recherche en utilisant l’[API Suggestion automatique Bing](../bing-autosuggest/get-suggested-search-terms.md) pour afficher des suggestions de termes de recherche pendant la saisie de la requête.         |
|[Obtenir des actualités générales](concepts/search-for-news.md#get-general-news)     | Recherchez des actualités en envoyant une requête de recherche à l’API Recherche d’actualités Bing et en obtenant une liste d’articles d’actualité pertinents.           |
|[Principales actualités du jour](concepts/search-for-news.md#get-todays-top-news)      | Obtenez les principales actualités du jour, dans toutes les catégories.       |
|[Actualités par catégorie](concepts/search-for-news.md)     | Recherchez des actualités dans des catégories spécifiques.        | 
|[Gros titres de l’actualité](concepts/search-for-news.md)     | Recherchez les gros titres dans toutes les catégories.         |

## <a name="workflow"></a>Workflow

L’API Recherche d’actualités Bing étant un service web RESTful, elle peut être facilement appelée à partir de n’importe quel langage de programmation capable d’exécuter des requêtes HTTP et d’analyser des réponses JSON. Vous pouvez utiliser ce service soit par le biais de l’API REST, soit à l’aide du Kit de développement logiciel (SDK).

1. Créez un [compte d’API Cognitive Services](../cognitive-services-apis-create-account.md) disposant d’un accès aux API Recherche Bing. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte](https://azure.microsoft.com/free/cognitive-services/) gratuitement.
2. Envoyez une requête à l’API avec une requête de recherche valide.
3. Traitez la réponse de l’API en analysant le message JSON renvoyé.

## <a name="next-steps"></a>Étapes suivantes

Commencez par essayer la [démonstration interactive](https://azure.microsoft.com/services/cognitive-services/bing-news-search-api/) de l’API Recherche d’actualités Bing. Cette démonstration vous explique comment personnaliser rapidement une requête de recherche et trouver des actualités sur le web.

Pour démarrer rapidement avec votre première requête d’API, suivez un guide de démarrage rapide de l’[API REST](./csharp.md) ou de l’un des [kits SDK](./quickstarts/client-libraries.md?pivots=programming-language-csharp).

## <a name="see-also"></a>Voir aussi

* La section de référence concernant l’[API Recherche d’actualités Bing v7](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference) contient des définitions et des informations sur les points de terminaison, en-têtes, réponses d’API et paramètres de requête que vous pouvez utiliser pour obtenir des actualités dans les résultats de la recherche.
* L’article [Conditions d’utilisation et d’affichage de l’API Recherche Bing](../bing-web-search/use-display-requirements.md) spécifie les utilisations acceptables du contenu et des informations obtenus par le biais des API Recherche Bing.
* Visitez la [page hub de l’API Recherche Bing](../bing-web-search/overview.md) pour explorer les autres API disponibles.