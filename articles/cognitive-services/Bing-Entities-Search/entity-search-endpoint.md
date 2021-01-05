---
title: Point de terminaison de l’API Recherche d’entités Bing
titleSuffix: Azure Cognitive Services
description: L’API Recherche d’entités Bing possède un point de terminaison qui renvoie des entités du Web en fonction d’une requête. Ces résultats de recherche sont retournés au format JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: conceptual
ms.date: 02/01/2019
ms.author: aahi
ms.openlocfilehash: 7fc65b27c3c02d6102210ae277690795d763d91a
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338255"
---
# <a name="bing-entity-search-api-endpoint"></a>Point de terminaison de l’API Recherche d’entités Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).


L’API Recherche d’entités Bing possède un point de terminaison qui renvoie des entités du Web en fonction d’une requête. Ces résultats de recherche sont retournés au format JSON.

## <a name="get-entity-results-from-the-endpoint"></a>Obtenir des résultats d’entité du point de terminaison

Pour obtenir des résultats d’entités à l’aide de l’**API Bing**, envoyez une requête `GET` au point de terminaison suivant. Utilisez les [en-têtes](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#headers) et les [ paramètres de requête](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#query-parameters) pour personnaliser votre requête de recherche. Les requêtes de recherche peuvent être envoyées à l’aide du paramètre `?q=`.

```cURL
 GET https://api.cognitive.microsoft.com/bing/v7.0/entities
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Qu’est-ce que l’API Recherche d’entités Bing ?](overview.md)

## <a name="see-also"></a>Voir aussi 

Pour plus d’informations sur les en-têtes, les paramètres, les codes de marché, les objets de réponse, les erreurs, entre autres, voir l’article de référence sur l’[API Recherche d’entités Bing v7](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference).