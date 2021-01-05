---
title: Personnaliser et suggérer des requêtes de recherche d’images - API Recherche d’images Bing
titleSuffix: Azure Cognitive Services
description: Découvrez comment personnaliser les requêtes de recherche que vous envoyez à l’API Recherche d’images Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: C2862E98-8BCC-423B-9C4A-AC79A287BE38
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: aahi
ms.openlocfilehash: 2566b2cf950df915f8ea843c34ea1fb6f8e7ea21
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96342003"
---
# <a name="customize-and-suggest-image-search-queries"></a>Personnaliser et suggérer des requêtes de recherche d’images

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Utilisez cet article pour apprendre à personnaliser les requêtes et suggérer des termes de recherche à envoyer à l’API Recherche d’images Bing.

## <a name="suggest-search-terms"></a>Suggérer des termes de recherche

Si votre application comprend une zone de recherche où les termes de recherche sont entrés, vous pouvez utiliser l’[API Suggestion automatique Bing](../../bing-autosuggest/get-suggested-search-terms.md) pour améliorer l’expérience. L’API peut afficher des suggestions de termes en temps réel. L’API suggère des chaînes de requête basées sur des termes de recherche partiels et Cognitive Services.

## <a name="pivot-the-query"></a>Ajouter un tableau croisé dynamique à la requête

Si Bing peut segmenter la requête de recherche d’origine, l’objet [Images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images) retourné contient `pivotSuggestions`. Les suggestions de tableau croisé dynamique peuvent être affichées comme des termes de recherche facultatifs à l’utilisateur. Par exemple, Bing peut segmenter la requête d’origine *Microsoft Surface* en *Microsoft* et *Surface* et suggérer des tableaux croisés dynamiques pour chacun. Ces suggestions peuvent être affichées comme des termes de requête facultatifs à l’utilisateur.

L’exemple suivant montre les suggestions de tableau croisé dynamique pour *Microsoft Surface* :  

```json
{
    "_type": "Images",
    "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=microsoft%20surface&FORM=OIIARP",
    "totalEstimatedMatches": 1000,
    "value": [...],
    "queryExpansions": [...],
    "pivotSuggestions": [{
        "pivot": "microsoft",
        "suggestions": [{
            "text": "Contoso Surface",
            "displayText": "Contoso",
            "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=OtterBox+Surface&FORM=IRQBPS",
            "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=Contoso...",
                    "searchLink": "https:\/\/api.cognitive.microsoft.com\/api...",
            "thumbnail": {
                "thumbnailUrl": "https:\/\/tse3.mm.bing.net\/th?q=Contoso+Surface..."
            }
        },
        {
            "text": "Adatum Surface",
            "displayText": "Adatum",
            "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=Adatum+Surface&FORM=IRQBPS",
            "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=...",
            "thumbnail": {
                "thumbnailUrl": "https:\/\/tse3.mm.bing.net\/th?q=Adatum+Surface&pid=Ap..."
            }
        },
        ...
        ]
    },
    {
        "pivot": "surface",
        "suggestions": [{
            "text": "Microsoft Surface4",
            "displayText": "Surface4",
            "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=Microsoft+Surface...",
            "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?...",
            "thumbnail": {
                "thumbnailUrl": "https:\/\/tse4.mm.bing.net\/th?q=Microsoft..."
            }
        },
        {
            "text": "Microsoft Tablet",
            "displayText": "Tablet",
            "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=Microsoft+Tablet&FORM=IRQBPS",
            "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?...",
            "thumbnail": {
                "thumbnailUrl": "https:\/\/tse3.mm.bing.net\/th?q=Microsoft+Tablet..."
            }
        },
        ...
    ],
    "nextOffsetAddCount": 0
}
```

Le champ `pivotSuggestions` contient la liste des segments (tableaux croisés dynamiques) composant la requête d’origine. Pour chaque tableau croisé dynamique, la réponse contient une liste d’objets [Query](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#query_obj) contenant les requêtes suggérées. Le champ `text` contient la requête suggérée. Le champ `displayText` contient le terme qui remplace le tableau croisé dynamique dans la requête d’origine. Il peut s’agir par exemple de la date de lancement de Surface.

Si la chaîne de requête du tableau croisé dynamique est vraiment ce que l’utilisateur recherche, utilisez les champs `text` et `thumbnail` pour présenter les requêtes du tableau croisé dynamique. Pour rendre la miniature et le texte interactifs, utilisez l’URL `webSearchUrl` ou l’URL `searchLink`. Utilisez `webSearchUrl` pour envoyer l’utilisateur vers les résultats de recherche Bing. Si vous fournissez votre propre page de résultats, utilisez `searchLink`.

<!-- Need a sanitized version of the image
The following shows an example of the pivot queries.

![Pivot suggestions](./media/cognitive-services-bing-images-api/bing-image-pivotsuggestion.GIF)
-->

## <a name="expand-the-query"></a>Développer la requête

Si Bing peut développer la requête pour affiner la recherche d’origine, l’objet [Images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images) contient le champ `queryExpansions`. Par exemple, si la requête était *Microsoft Surface*, les requêtes développées peuvent être :
- Microsoft Surface **Pro 3**.
- Microsoft Surface **RT**.
- Microsoft Surface **Phone**.
- Microsoft Surface **Hub**.

L’exemple suivant montre les requêtes développées pour *Microsoft Surface*.

```json
{
    "_type": "Images",
    "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=microsoft%20surface...",
    "totalEstimatedMatches": 1000,
    "value": [...],
    "queryExpansions":  [{
        "text": "Microsoft Surface Pro 3",
        "displayText": "Pro 3",
        "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=Microsoft+Surface+Pro+3...",
        "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=Microsoft...",
        "thumbnail": {
            "thumbnailUrl": "https:\/\/tse4.mm.bing.net\/th?q=Microsoft+Surface+Pro+3..."
        }
    },
    {
        "text": "Microsoft Surface RT",
        "displayText": "RT",
        "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=Microsoft+Surface+RT...",
        "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=...",
        "thumbnail": {
            "thumbnailUrl": "https:\/\/tse4.mm.bing.net\/th?q=Microsoft+Surface+RT..."
        }
    },
    {
        "text": "Microsoft Surface Phone",
        "displayText": "Phone",
        "webSearchUrl": "https:\/\/www.bing.com\/images\/search?q=Microsoft+Surface+Phone",
        "searchLink": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=...",
        "thumbnail": {
            "thumbnailUrl": "https:\/\/tse4.mm.bing.net\/th?q=Microsoft+Surface+Phone..."
        }
    }],
    "pivotSuggestions": [...],
    "nextOffsetAddCount": 0
}
```

Le champ `queryExpansions` contient une liste d’objets [Query](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#query_obj). Le champ `text` contient la requête développée. Le champ `displayText` contient le terme de développement. Si la chaîne de requête développée est vraiment ce que l’utilisateur recherche, utilisez les champs `text` et `thumbnail` pour présenter les requêtes de requête développée. Pour rendre la miniature et le texte interactifs, utilisez l’URL `webSearchUrl` ou l’URL `searchLink`. Utilisez `webSearchUrl` pour envoyer l’utilisateur vers les résultats de recherche Bing. Si vous fournissez votre propre page de résultats, utilisez `searchLink`.

<!-- Removing until we can replace with a sanitized image.
The following shows an example Bing implementation that uses expanded queries. If the user clicks the Microsoft Surface Pro 3 link, they're taken to the Bing search results page, which shows them images of the Pro 3.

![Query expansion suggestions](./media/cognitive-services-bing-images-api/bing-image-queryexpansion.GIF)
-->


## <a name="throttling-requests"></a>Demandes de limitation

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>Étapes suivantes

Si vous n’avez pas déjà essayé l’API Recherche d’images Bing, essayez un [démarrage rapide](../quickstarts/csharp.md). Si vous recherchez quelque chose de plus complexe, suivez le didacticiel de création d’une [application web à une seule page](../tutorial-bing-image-search-single-page-app.md).