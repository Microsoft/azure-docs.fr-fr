---
title: Obtenir des images populaires avec l’API Recherche d’images Bing
titleSuffix: Azure Cognitive Services
description: Recherchez les images populaires du moment sur le web avec l’API Recherche d’images Bing.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: EAB92D35-5C0B-4A0A-8F49-02DF7FAD44B4
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: scottwhi
ms.custom: seodec2018
ms.openlocfilehash: cf7d1baf895d44730eb913b658ee4c7fe7eb7b11
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341612"
---
# <a name="get-trending-images-from-the-web"></a>Obtenir des images populaires à partir du web

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Pour obtenir des images qui sont populaires à l’heure actuelle, envoyez la requête GET suivante :  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/trending?mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

L’API Images populaires n’est actuellement disponible que sur les marchés suivants :  

- en-US (anglais, États-Unis)  
- en-CA (anglais, Canada)  
- en-AU (anglais, Australie)  
- zh-CN (chinois, Chine)

La réponse contient un objet [TrendingImages](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#trendingimages) qui classe les images par catégorie. Utilisez l’élément `title` de la catégorie pour grouper les images dans votre expérience utilisateur. Les catégories sont susceptibles de changer tous les jours.  

```json
{
    "_type" : "TrendingImages",  
    "categories" : [{  
        "title" : "Popular people searches",  
        "tiles" : [{  
            "query" : {  
                "text" : "Smith",  
                "displayText" : "Mr. Smith",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=smith&FORM=..."
            },  
            "image" : {  
                "id" : "C3C60AE779A054D5CD80D3CACF0F3",  
                "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OIP.M2532...",  
                "contentUrl" : "http:\/\/www.contoso.com.au\/assets\/Uploads\/smith-SH01.jpg",  
                "thumbnail" : {  
                    "width" : 288,  
                    "height" : 300  
                }  
            }  
        },  
        . . .  
        ]  
    },  
    . . .  
    {  
        "title" : "Popular Halloween searches",  
        "tiles" : [{  
            "query" : {  
                "text" : "Halloween costumes for adults",  
                "displayText" : "Halloween costumes for adults",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Halloween+costumes..."
            },  
            "image" : {  
                "id" : "0F3395F2983003F89DCEE711B55D7FA53E4",  
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OIP.Me429c...",  
                "contentUrl" : "http:\/\/images.domain.com\/products\/8179\/1-1\/adult-squirrel...",  
                "thumbnail" : {  
                    "width" : 336,  
                    "height" : 480  
                }  
            }  
        }]  
    }]  
}  
```  

Chaque vignette contient une image et des options permettant d’obtenir les images associées. Pour obtenir les images associées, vous pouvez utiliser la requête `text` pour appeler l’[API Recherche d’images](./overview.md) et afficher vous-même les images associées. Vous pouvez également utiliser l’URL dans `webSearchUrl` pour rediriger l’utilisateur sur la page des résultats de la recherche d’images Bing qui contient les images associées.

Si vous appelez l’API Recherche d’images pour obtenir les images associées, définissez le paramètre de requête [id](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#id) sur l’ID du champ `id`. En spécifiant l’ID, vous vous assurez que la réponse contient l’image (il s’agit de la première image de la réponse) et les images qui y sont associées. En outre, définissez le paramètre de requête [q](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference) sur le texte dans le champ `text` de l’objet `query`.

L’exemple suivant illustre l’utilisation de l’ID de l’image pour obtenir les images associées de M. Smith dans la précédente réponse de l’API Images populaires.

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=Smith&id=77FDE4A1C6529A23C7CF0EC073FAA64843E828F2&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```