---
title: Utiliser les classements pour afficher les résultats de recherche obtenus - API Recherche Web Bing
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser le classement pour afficher les résultats de recherche obtenus avec l’API Recherche Web Bing.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: BBF87972-B6C3-4910-BB52-DE90893F6C71
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 03/17/2019
ms.author: scottwhi
ms.openlocfilehash: 20501d0993cc4566a79d6e916d801911606bea35
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94380447"
---
# <a name="how-to-use-ranking-to-display-bing-web-search-api-results"></a>Comment utiliser le classement pour afficher les résultats obtenus avec l’API Recherche Web Bing  

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020** , toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Chaque réponse de recherche inclut une réponse [RankingResponse](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#rankingresponse) qui indique comment afficher les résultats de la recherche. La réponse de ce classement regroupe les résultats en fonction du contenu des liens premium et du contenu des encadrés pour une page de résultats de recherche type. Si vous n’affichez pas les résultats dans ce format type, vous devez fournir le contenu des liens premium dont la visibilité est supérieure à celle du contenu des encadrés.  

Au sein de chaque groupe (liens premium et encadrés), le tableau [Éléments](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#rankinggroup-items) identifie l’ordre dans lequel doit apparaître le contenu. Chaque élément fournit deux façons d’identifier le résultat au sein d’une réponse.  

-   `answerType` et `resultIndex` : le champ `answerType` identifie la réponse (par exemple, une page web ou des actualités) et `resultIndex` identifie un résultat au sein de la réponse (par exemple, un article). L'index est de base zéro.  

-   `value` : le champ `value` contient un identifiant qui correspond à l’identifiant de la réponse ou d’un résultat au sein de la réponse. La réponse ou les résultats contiennent l’identifiant, mais pas les deux.  

Il est plus simple d’utiliser l’identifiant. En effet, vous devez faire correspondre l’ID du classement et l’ID d’une réponse (ou l’un de ses résultats). Si un objet de réponse inclut un champ `id`, affichez tous les résultats de la réponse ensemble. Par exemple, si l’objet `News` inclut le champ `id`, affichez tous les articles d’actualité ensemble. Si l’objet `News` n’inclut pas le champ `id`, alors chaque article comprend un champ `id` et la réponse de classement mélange les articles d’actualité avec les résultats d’autres réponses.  

Par contre, l’utilisation de `answerType` et `resultIndex` se révèle un peu plus compliquée. Tout d’abord, vous utilisez `answerType` pour identifier la réponse qui contient les résultats à afficher. Ensuite, vous utilisez `resultIndex` pour parvenir à une indexation dans les résultats de cette réponse afin d’en afficher le résultat. (La valeur `answerType` correspond au nom du champ dans l’objet [SearchResponse](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#searchresponse).) Si vous êtes censé afficher tous les résultats de la réponse ensemble, l’élément de réponse de classement n’inclut pas le champ `resultIndex`.  

## <a name="ranking-response-example"></a>Exemple de réponse de classement

Le code suivant vous fournit un exemple [RankingResponse](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#rankingresponse). Étant donné que la réponse web n’inclut aucun champ `id`, vous affichez toutes les pages web individuellement selon le classement (chaque page web inclut un champ `id`). Et parce que les images, les vidéos et les réponses aux recherches associées n’incluent pas le champ `id`, vous affichez les résultats de chacune de ces réponses ensemble, selon le classement.

```json
{  
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214...",
        "totalEstimatedMatches" : 835000,
        "value" : [
            {
                "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
                "name" : "Motor Sports - Live at the race track ...",
                "url" : "http:\/\/www.bing.com\/cr?IG=96C4CF214A0A43...",
                "displayUrl" : "www.contoso.com\/usa\/eventsandracing\/motorsport",
                "snippet" : "Here you will find detailed information about racing...",
                "deepLinks" : [{
                    "name" : "Customer Racing",
                    "url" : "http:\/\/www.bing.com\/cr?IG=96C4CF214A0A43...",
                    "snippet" : "Customer racing news; General news..."
            },
            . . .  
        ]  
    }],  
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214A...",
        "isFamilyFriendly" : true,
        "value" : [
            {
                "name" : "2016 Supercar Wallpapers",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP...",
                "datePublished" : "2017-03-25T11:14:00",
                "contentUrl" : "http:\/\/www.contoso.com\/wall...",
                "hostPageUrl" : "http:\/\/www.bing.com\/cr?IG=96C4CF214...",
                "contentSize" : "373283 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "http:\/\/www.contoso.com\/lmp-...",
                "width" : 1920,
                "height" : 1080,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 168
                },
                "insightsSourcesSummary" : {
                    "shoppingSourcesCount" : 0,
                    "recipeSourcesCount" : 0
                }
            },
            . . .  
        ]  
    },  
    "relatedSearches" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches",
        "value" : [
            {
                "text" : "vintage racing teams",
                "displayText" : "vintage racing teams",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF2..."
            },
            . . .  
        ]  
    },  
    "videos" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Videos",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/videos...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214A...",
        "isFamilyFriendly" : true,
        "value" : [
            {
                "name" : "Why We Race",
                "description" : "A new era begins in motorsports this weekend...",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF2...",
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OVP.Vo1...",
                "datePublished" : "2014-01-25T16:31:48",
                "publisher" : [
                    {
                        "name" : "Fabrikam"
                    }
                ],
                "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=oL...",
                "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=96C4CF214...",
                "encodingFormat" : "mp4",
                "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?v=oLAZgD...",
                "width" : 480,
                "height" : 360,
                "duration" : "PT2M42S",
                "motionThumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OM...",
                "embedHtml" : "<iframe width=\"1280\" height=\"720\" src=\"http:\/\/www.you...<\/iframe>",
                "allowHttpsEmbed" : true,
                "viewCount" : 47325,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 168
                },
                "allowMobileEmbed" : true,
                "isSuperfresh" : false
            },
            . . .  
        ]  
    },  
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "WebPages",
                "resultIndex" : 0,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0"
                }
            },
            {
                "answerType" : "Images",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 1,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.1"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 2,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.2"
                }
            },
            {
                "answerType" : "Videos",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Videos"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 3,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.3"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 4,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.4"
                }
            },
            {
                "answerType" : "WebPages",
                "resultIndex" : 5,
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.5"
                }
            }]
        },
        "sidebar" : {
            "items" : [{
                "answerType" : "RelatedSearches",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#RelatedSearches"
                }
            }]
        }
    }
}  
```  

En fonction de cette réponse de classement, le lien premium affiche les résultats de recherche suivants :  

-   Le résultat de la première page web
-   Toutes les images  
-   Les résultats de la deuxième et de la troisième page web  
-   Toutes les vidéos  
-   Les résultats de la quatrième, de la cinquième et de la sixième page web  

L’encadré quant à lui affiche les résultats de recherche suivants :  

-   Toutes les recherches associées  


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la promotion des résultats hors classement, consultez la section [Promouvoir les réponses qui ne sont pas classées](./filter-answers.md#promoting-answers-that-are-not-ranked).

Pour plus d’informations sur la restriction du nombre de réponses classées dans la réponse, consultez la section [Restreindre le nombre de réponses](./filter-answers.md#limiting-the-number-of-answers-in-the-response).

Pour obtenir un exemple C# qui utilise le classement pour afficher les résultats, consultez la section [Didacticiel de classement C#](./csharp-ranking-tutorial.md).