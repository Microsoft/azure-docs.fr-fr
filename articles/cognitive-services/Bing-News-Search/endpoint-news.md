---
title: Points de terminaison Recherche d’actualités Bing
titleSuffix: Azure Cognitive Services
description: Cet article fournit un résumé des points de terminaison, actualités, principales actualités du jour et sujets d’actualités tendance de l’API Recherche d’actualités.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: conceptual
ms.date: 1/10/2019
ms.author: aahi
ms.openlocfilehash: 9c095a46b3a0526b23645c9cbb5e99eb8eda9067
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94366398"
---
# <a name="bing-news-search-api-endpoints"></a>Points de terminaison de l'API Recherche d'actualités Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020** , toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

**L’API Recherche d’actualités** renvoie des articles, des pages web, des images, des vidéos et des [entités](../bing-entities-search/overview.md) en lien avec des actualités. Les entités contiennent un résumé d’informations sur une personne, un lieu ou un sujet.

## <a name="endpoints"></a>Points de terminaison

Pour obtenir des résultats de recherche d'actualités à l'aide de l'API Recherche d'actualités Bing, envoyez une requête `GET` à l'un des points de terminaison suivants. Les en-têtes et les paramètres d’URL définissent d’autres spécifications.

### <a name="news-items-by-search-query"></a>Articles en lien avec des actualités par requête de recherche

```
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search
```

Renvoie des articles en lien avec des actualités basés sur une requête de recherche. Si la requête de recherche est vide, l'API renvoie les meilleurs articles issus de différentes catégories. Envoyez une requête en encodant votre terme de recherche sous forme d'URL et en l'ajoutant au paramètre `q=""`. Pour plus d’informations sur la disponibilité, consultez [Supported countries/regions and markets](language-support.md#supported-markets-for-news-search-endpoint) (Pays/régions et marchés pris en charge).

### <a name="top-news-items-by-category"></a>Meilleurs articles en lien avec des actualités par catégorie

```
GET https://api.cognitive.microsoft.com/bing/v7.0/news  
```

Renvoie les meilleurs articles en lien avec des actualités par catégorie. Vous pouvez demander spécifiquement les meilleurs articles en lien avec l’économie, le sport ou le divertissement à l’aide de `category=business`, `category=sports` ou `category=entertainment`.  Le paramètre `category` peut uniquement être utilisé avec l’URL `/news`. Certaines conditions s’appliquent pour spécifier des catégories ; reportez-vous à `category` dans la documentation sur le [paramètre de requête](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#query-parameters). Envoyez une requête en encodant votre terme de recherche sous forme d'URL et en l'ajoutant au paramètre `q=""`. Pour plus d’informations sur la disponibilité, consultez [Supported countries/regions and markets](language-support.md#supported-markets-for-news-endpoint) (Pays/régions et marchés pris en charge).

### <a name="trending-news-topics"></a>Sujets d'actualité tendance 

```
GET https://api.cognitive.microsoft.com/bing/v7.0/news/trendingtopics
```

Renvoie des sujets en lien avec des actualités qui sont actuellement populaires sur les réseaux sociaux. Lorsque l’option `/trendingtopics` est incluse, la recherche Bing ignore plusieurs autres paramètres, tels que `freshness` et `?q=""`. Pour plus d’informations sur la disponibilité, consultez [Supported countries/regions and markets](language-support.md#supported-markets-for-news-trending-endpoint) (Pays/régions et marchés pris en charge).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les en-têtes, les paramètres, les codes de marché, les objets de réponse, les erreurs, etc., consultez [News Search API v7 reference](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference) (Référence sur l’API Recherche d’actualités Bing v7).

Pour plus d’informations sur les paramètres pris en charge par chaque point de terminaison, consultez les pages de référence pour chaque type.
Pour obtenir des exemples de requêtes de base à l’aide de l’API Recherche d’actualités, consultez les [démarrages rapides sur l’API Recherche d'actualités Bing](/azure/cognitive-services/bing-news-search).