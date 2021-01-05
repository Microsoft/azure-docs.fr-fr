---
title: 'Comment filtrer les résultats de la recherche : API Recherche Web Bing'
titleSuffix: Azure Cognitive Services
description: Vous pouvez filtrer les types de réponses que Bing inclut dans la réponse (par exemple, des images, des vidéos et des actualités) à l’aide du paramètre de requête responseFilter.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: 8B837DC2-70F1-41C7-9496-11EDFD1A888D
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 07/08/2019
ms.author: scottwhi
ms.openlocfilehash: 571314009b6f58e5c2ab6aac02cfebc82c53f42f
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351859"
---
# <a name="filtering-the-answers-that-the-search-response-includes"></a>Filtrage des correspondances contenues dans la réponse de recherche  

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Lorsque vous interrogez le web, Bing renvoie tout le contenu qu’il juge pertinent pour la recherche. Par exemple, si la requête de recherche est « sailing+dinghies », la réponse peut contenir les correspondances suivantes :

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43C...",
        "totalEstimatedMatches" : 262000,
        "value" : [...]
    },
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=sail...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43CA5CA6464E5D...",
        "isFamilyFriendly" : true,
        "value" : [...]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [...]
        }
    }
}    
```

## <a name="query-parameters"></a>Paramètres de requête

Pour filtrer les réponses retournées par Bing, utilisez les paramètres de requête ci-dessous lors de l’appel à l’API.  

### <a name="responsefilter"></a>ResponseFilter

Vous pouvez filtrer les types de réponses que Bing inclut dans la réponse (par exemple des images, des vidéos et des actualités) à l’aide du paramètre de requête [responseFilter](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#responsefilter), qui est une liste délimitée par des virgules de réponses. Une réponse figurera dans la réponse si Bing détecte que le contenu est pertinent. 

Pour exclure des réponses spécifiques du résultat, comme des images, ajoutez un caractère `-` avant le type de réponse. Par exemple :

```
&responseFilter=-images,-videos
```

Le code suivant montre comment utiliser `responseFilter` pour demander des images, des vidéos et des actualités sur les dériveurs. Quand vous encodez la chaîne de requête, les virgules sont remplacées par %2C.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&responseFilter=images%2Cvideos%2Cnews&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Le code suivant affiche la réponse à la requête précédente. Bing n’ayant pas trouvé de résultats pertinents en matière de vidéos et d’actualités, la réponse ne les inclut pas.

```json
{
    "_type" : "SearchResponse",
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=sail...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3AD78B183C56456C...",
        "isFamilyFriendly" : true,
        "value" : [...]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "Images",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images"
                }
            }]
        }
    }
}
```

Même si Bing n’a pas retourné de résultats en matière de vidéos et d’actualités dans la réponse précédente, cela ne signifie pas que ce contenu n’existe pas. Cela signifie simplement que la page ne contient pas ces résultats. Toutefois, si vous [parcourez](./paging-search-results.md) plus de résultats, il est probable qu’ils s’affichent dans les pages suivantes. En outre, si vous appelez directement les points de terminaison [d’API Recherche de vidéos](../bing-video-search/overview.md) et [d’API Recherche d’actualités](../bing-news-search/search-the-web.md), la réponse contient probablement les résultats.

Il est déconseillé d’utiliser `responseFilter` pour obtenir les résultats à partir d’une seule API. Si vous souhaitez obtenir le contenu d’une seule API Bing, appelez cette API directement. Par exemple, pour recevoir uniquement des images, envoyez une demande au point de terminaison d’API Recherche d’images, `https://api.cognitive.microsoft.com/bing/v7.0/images/search` ou à l’un des autres points de terminaison [Images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#endpoints). L’appel de l’API unique est important non seulement pour des raisons de performances, mais parce que les API spécifiques au contenu offrent des résultats plus riches. Par exemple, vous pouvez utiliser des filtres qui ne sont pas disponibles pour l’API Recherche web pour filtrer les résultats.  

### <a name="site"></a>Site

Pour obtenir des résultats de recherche à partir d’un domaine spécifique, ajoutez le paramètre de requête `site:` dans la chaîne de requête.  

```
https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us
```

> [!NOTE]
> En fonction de la requête, si vous utilisez l’opérateur de requête `site:`, il est possible que la réponse présente du contenu pour adultes, et ce quel que soit le paramètre [safeSearch](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#safesearch) défini. Utilisez `site:` uniquement si vous connaissez le contenu du site et si votre scénario prend en charge le contenu pour adultes.

### <a name="freshness"></a>Actualisation

Pour limiter les résultats de la réponse web vers des pages web que Bing a découvertes pendant une période spécifique, définissez le paramètre de requête [freshness](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#freshness) sur une des valeurs (non sensibles à la casse) suivantes :

* `Day` : retourne les pages web que Bing a découvertes dans les dernières 24 heures
* `Week` : retourne les pages web que Bing a découvertes dans les derniers 7 jours
* `Month` : retourne les pages web que Bing a découvertes dans les derniers 30 jours

Vous pouvez également définir ce paramètre sur une plage de dates personnalisée au format `YYYY-MM-DD..YYYY-MM-DD`. 

`https://<host>/bing/v7.0/search?q=ipad+updates&freshness=2019-02-01..2019-05-30`

Pour limiter les résultats à une date unique, définissez le paramètre freshness sur une date spécifique :

`https://<host>/bing/v7.0/search?q=ipad+updates&freshness=2019-02-04`

Les résultats peuvent inclure des pages web qui se situent en dehors de la période spécifiée si le nombre de pages web que Bing fait correspondre à vos critères de filtre est inférieur au nombre de pages web que vous avez demandé (ou le nombre par défaut que retourne Bing).

## <a name="limiting-the-number-of-answers-in-the-response"></a>Limitation du nombre de correspondances dans la réponse

Bing peut retourner plusieurs types de réponses dans la réponse JSON. Par exemple, si votre requête porte sur *sailing+dinghies*, Bing peut retourner `webpages`, `images`, `videos` et `relatedSearches`.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "relatedSearches" : {...},
    "videos" : {...},
    "rankingResponse" : {...}
}
```

Pour limiter le nombre de réponses retournées par Bing aux deux premières (pages web et images), affectez au paramètre de requête [answerCount](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#answercount) la valeur 2.

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&answerCount=2&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

La réponse inclut uniquement `webPages` et `images`.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "rankingResponse" : {...}
}
```

Si vous ajoutez le paramètre de requête `responseFilter` à la requête précédente avec pour valeur les pages web et actualités, la réponse contient uniquement des pages web, car les actualités ne sont pas classées.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "rankingResponse" : {...}
}
```

## <a name="promoting-answers-that-are-not-ranked"></a>Promotion des correspondances qui ne sont pas classées

Si les correspondances en tête du classement retournées par Bing pour une requête sont des pages web, images, vidéos et relatedSearches, la réponse inclut ces correspondances. Si vous affectez à [answerCount](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#answercount) la valeur deux (2), Bing retourne les deux correspondances en tête du classement : pages web et images. Si vous souhaitez que Bing ajoute des images et vidéos dans la réponse, spécifiez le paramètre de requête [promote](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#promote) et définissez-le sur les images et vidéos.

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&answerCount=2&promote=images%2Cvideos&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Voici la réponse à la requête ci-dessus. Bing retourne les deux premières correspondances, pages web et images, et promeut les vidéos dans la correspondance.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailiing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "videos" : {...},
    "rankingResponse" : {...}
}
```

Si vous définissez `promote` sur les actualités, la réponse n’inclut pas la correspondance actualités, car il ne s’agit pas d’une correspondance classée &mdash; vous pouvez promouvoir uniquement des correspondances classées.

Les correspondances que vous souhaitez promouvoir n’entrent pas dans la limite `answerCount`. Par exemple, si les correspondances classées sont des actualités, des images et des vidéos, et que vous affectez à `answerCount` la valeur 1 et définissez `promote` sur les actualités, la réponse contient des actualités et des images. Par ailleurs, si les correspondances classées sont des actualités, des images et des vidéos, la réponse contient des vidéos et des actualités.

Vous pouvez utiliser `promote` uniquement si vous spécifiez le paramètre de requête `answerCount`.