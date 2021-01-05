---
title: Recherche de vidéos populaires sur le web à l’aide de l’API Recherche de vidéos Bing
titleSuffix: Azure Cognitive Services
description: Apprenez à utiliser l’API Recherche de vidéos Bing pour rechercher des vidéos populaires sur le web.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: conceptual
ms.date: 01/31/2019
ms.author: scottwhi
ms.openlocfilehash: cfeebcda201df592f6c396dcc780208a36d1e989
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353797"
---
# <a name="get-trending-videos-with-the-bing-video-search-api"></a>Obtenir des vidéos tendance avec l’API Recherche de vidéos Bing 

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche de vidéos Bing vous permet de trouver les vidéos tendance du jour sur le Web et dans différentes catégories. 

## <a name="get-request"></a>Requête GET

Pour obtenir des vidéos qui sont populaires à l’heure actuelle à l’aide de l’API Recherche de vidéos Bing, envoyez la requête GET suivante :  
  
```cURL
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/trending?mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

## <a name="market-support"></a>Support des marchés

Les marchés suivants prennent en charge les vidéos populaires.  
 
-   en-AU (anglais, Australie)  
-   en-CA (anglais, Canada)  
-   en-GB (anglais, Grande-Bretagne)  
-   en-ID (anglais, Indonésie)  
-   en-IE (anglais, Irlande)  
-   en-IN (anglais, Inde)  
-   en-NZ (anglais, Nouvelle-Zélande)  
-   en-PH (anglais, Philippines)  
-   en-SG (anglais, Singapour)  
-   en-US (anglais, États-Unis)  
-   en-WW (anglais, code agrégé mondial)  
-   en-ZA (anglais, Afrique du Sud)  
-   zh-CN (chinois, Chine)

## <a name="example-json-response"></a>Exemple de réponse JSON  

L’exemple suivant montre une réponse d’API qui contient des vidéos les plus populaires, qui sont répertoriées par catégorie et sous-catégorie. La réponse contient également les vidéos bannières, qui correspondent aux vidéos populaires les plus regardées et peuvent émaner d’une ou de plusieurs catégories.  

```json
{  
    "_type" : "TrendingVideos",  
    "bannerTiles" : [
        {  
            "query" : {  
                "text" : "Hello - Smith",  
                "displayText" : "Hello - Smith",  
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3E8F5..."
            },  
            "image" : {  
                "description" : "Image from: contosowallpapers.com",  
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=RsA%2fdPlTmx4zS...",  
                "headLine" : "\"Hello\" is a song by...",  
                "contentUrl" : "http:\/\/www.contosowallpapers.com\/wp-content..."  
            }  
        },  
        . . .  
    ],  
    "categories" : [
        {  
            "title" : "Music videos",  
            "subcategories" : [
                {  
                    "tiles" : [
                        {  
                            "query" : {  
                                "text" : "Song Title - Artist Name",  
                                "displayText" : "Song Title - Artist Name",  
                                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3E8F5..."
                            },  
                            "image" : {  
                                "description" : "Image from: contoso.com",  
                                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=...",  
                                "contentUrl" : "http:\/\/images6.contoso.com\/image..."  
                            }  
                        },  
                        . . .  
                    ],
                    "title" : "Top "  
                },
                {
                    "tiles" : [...],
                    "title" : "Trending"
                },
                . . .
            ],  
        },
        {
            "title" : "Viral videos",
            "subcategories" : [
                {
                    "tiles" : [...],
                    "title" : "Trending"
                },
                . . .
            ],  
        },
        . . .  
    ]  
}  
  
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Obtenir des insights sur les vidéos](video-insights.md)