---
title: Obtenir des informations sur les images - API Recherche d’images Bing
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API Recherche d’images Bing pour obtenir plus d’informations sur une image.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: 0BCD936E-D4C0-472D-AE40-F4B2AB6912D5
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: scottwhi
ms.openlocfilehash: 9dffc16a1d1171343fa10fce10fddba2a12c2167
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348243"
---
# <a name="get-image-insights-with-the-bing-image-search-api"></a>Obtenir des informations sur les images avec l’API Recherche d’images Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

> [!IMPORTANT]
> Plutôt que d’utiliser le point de terminaison /images/détails pour obtenir des informations sur l’image, vous devez utiliser la [recherche visuelle](../bing-visual-search/overview.md) car elle fournit des informations plus complètes.


Chaque image inclut un jeton d’informations que vous pouvez utiliser pour obtenir des informations sur l’image. Par exemple, vous pouvez obtenir une collection d’images associées, des pages web qui incluent l’image ou une liste de prestataires auprès desquels vous pouvez acheter le produit indiqué dans l’image.  

Pour obtenir des informations sur une image, capturez le jeton [imageInsightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#image-imageinsightstoken) de l’image dans la réponse.

```json
"value" : [{
        . . .
        "name":"sailing dinghy.jpg",
        "imageInsightsToken" : "mid_D6426898706EC7..."
        "insightsSourcesSummary" : {
            "shoppingSourcesCount" : 9,
            "recipeSourcesCount" : 0
        },
        . . .
}],
```

Appelez ensuite le point de terminaison de détails de l’image et définissez le paramètre de requête [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) sur le jeton dans `imageInsightsToken`.  

Pour spécifier les informations que vous souhaitez obtenir, définissez le paramètre de requête `modules`. Pour obtenir toutes les informations, définissez `modules` sur `All`. Pour n’obtenir que les informations de légende et de collection, définissez `modules` sur `Caption%2CCollection`. Pour obtenir la liste complète des informations possibles, consultez [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested). Toutes les informations ne sont pas disponibles pour toutes les images. La réponse inclut toutes les informations que vous avez demandées, si disponibles.

L’exemple suivant demande toutes les informations disponibles pour l’image précédente.

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&insightsToken=mid_D6426898706EC7...&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

## <a name="getting-insights-of-a-known-image"></a>Obtention d’informations d’une image connue

Si vous disposez de l’URL vers une image sur laquelle vous souhaitez obtenir des informations, utilisez le paramètre de requête [imgUrl](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imgurl) au lieu du paramètre [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) pour spécifier l’image. Si vous disposez du fichier image, vous pouvez également insérer le binaire de l’image dans le corps d’une requête POST. Si vous utilisez une requête POST, l’en-tête `Content-Type` doit être défini sur `multipart/data-form`. Quelle que soit l’option, la taille de l’image ne peut pas dépasser 1 Mo.  

Si vous disposez d’une URL vers l’image, l’exemple suivant montre comment demander des informations sur une image.

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&imgUrl=https%3A%2F%2Fwww.mydomain.com%2Fimages%2Fsunflower.png&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

## <a name="getting-all-image-insights"></a>Obtention de toutes informations sur l’image  

Pour demander toutes les informations sur une image, définissez le paramètre de requête [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) sur `All`. Pour obtenir les recherches associées, la requête doit inclure la chaîne de requête de l’utilisateur. Cet exemple montre comment utiliser [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) pour spécifier l’image.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&insightsToken=mid_68364D764J...&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

L’objet de niveau supérieur est un objet [ImageInsightsResponse](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imageinsightsresponse) au lieu d’objet un [Images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images).  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "bcid_3297E6A54E4787A5F51C49D9BA342B9A*ccid_Fe2Hx...",
    "bestRepresentativeQuery" : {
        "text" : "Sailing Dinghy",
        "displayText" : "Sailing Dinghy",
        "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Sailing+Dinghy...",
    },
    "pagesIncluding" : {
        "value" : [
            {
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=...",
                "name" : "Powerboating Dublin, Dinghy Sailing Courses...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP....",
                "datePublished" : "2017-01-20T00:41:00.0000000Z",
                "contentUrl" : "http:\/\/www.contoso.ie\/content...",
                "hostPageUrl" : "http:\/\/www.contoso.ie\/powerboating...",
                "contentSize" : "59063 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "www.contoso.ie\/powerboating...",
                "width" : 800,
                "height" : 600,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 225
                },
                "imageInsightsToken" : "ccid_pHjQIA0x*mid_17F61B1316A39C92214...",
                "imageId" : "17F61B1316A39C922143FFDE9DFB5B0FB41171",
                "accentColor" : "0997C2"
            },
            . . .
        ]
    },
    "relatedSearches" : {
        "value" : [
            {
                "text" : "Sailing Fun",
                "displayText" : "Sailing Fun",
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Sailing...",
                "thumbnail" : {
                    "url" : "https:\/\/tse1.mm.bing.net\/th?q=Sailing+Fun..."
                }
            },
            . . .
        ]
    },
    "visuallySimilarImages" : {
        "value" : [
            {
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=...",
                "name" : "Weekend On the Water",
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=OIP...",
                "datePublished" : "2010-09-05T12:00:00.0000000Z",
                "contentUrl" : "http:\/\/1.bp.contoso.com\/_dc_6...",
                "hostPageUrl" : "http:\/\/contoso.com\/2010...",
                "contentSize" : "203806 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "contoso.com\/2010...",
                "width" : 1600,
                "height" : 1249,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 234
                },
                "imageInsightsToken" : "ccid_Jg1Kwuc4*mid_5B7DA43976D3A422...",
                "imageId" : "5B7DA43976D3A422BA679A3AB019BB52C08DBC",
                "accentColor" : "0B2543"
            },
            . . .
        ]
    },
    "imageTags" : {
        "value" : [
            {
                "name" : "sail boat"
            },
            . . .
        ]
    }
}
```

## <a name="recognizing-entities-in-an-image"></a>Reconnaissance des entités dans une image  

La fonctionnalité de reconnaissance d’entité identifie les entités dans une image, les personnes seulement pour le moment. Pour identifier les entités dans une image, définissez le paramètre de requête [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) sur `RecognizedEntities`.  

> [!NOTE]
> Vous ne pouvez pas spécifier ce module avec d’autres modules. Si vous spécifiez ce module avec d’autres modules, la réponse n’inclut pas les entités reconnues.  

L’exemple suivant montre comment spécifier l’image à l’aide du paramètre [imgUrl](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imgurl). N’oubliez pas de coder au format URL les paramètres de requête.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=faith+hill&insightsToken=mid_68364D764J...&modules=RecognizedEntities&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```  

Voici la réponse à la requête précédente. Étant donné que l’image contient deux personnes, la réponse identifie une région pour chaque personne. Dans ce cas, les personnes ont été reconnues dans les groupes CelebrityAnnotations et CelebRecognitionAnnotations. Bing répertorie les personnes de chaque groupe en fonction de la probabilité qu’elles correspondent à la personne dans l’image d’origine. La liste est présentée dans l’ordre décroissant de confiance. Le groupe CelebRecognitionAnnotations fournit le plus haut niveau de confiance d’exactitude de la correspondance.  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "recognizedEntityGroups" : {
        "value" : [
            {
                "recognizedEntityRegions" : [...],
                "name" : "CelebRecognitionAnnotations"
            },
            {
                "recognizedEntityRegions" : [...],
                "name" : "CelebrityAnnotations"
            }
        ]
    }
}
```

Le champ `region` identifie la zone de l’image où Bing a reconnu l’entité. Pour les personnes, la région représente le visage de la personne.  

Les valeurs du rectangle sont relatives à la largeur et la hauteur de l’image d’origine et sont comprises dans la plage de 0,0 à 1,0. Par exemple, si l’image est de 300 x 200, que le coin supérieur gauche de la région est au point (10, 20) et que le coin inférieur droit est au point (290, 150), le rectangle normalisé est :  

-   Gauche : 10 / 300 = 0,03333...  
-   Haut :  20 / 200 = 0,1  
-   Droite : 290 / 300 = 0,9667...  
-   Bas : 150 / 200 = 0,75  

Vous pouvez utiliser la région retournée par Bing dans les appels d’informations suivantes. Par exemple, pour obtenir des images visuellement similaires de l’entité reconnue. Pour plus d’informations, consultez Rognage d’images à utiliser avec les modules de reconnaissance Visuellement similaires et Entités. Voici le mappage entre les champs de la région et les paramètres de requête que vous utiliseriez pour rogner des images.  

-   Gauche correspond à [cal](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cal)  
-   Haut correspond à [cat](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cat)  
-   Droite correspond à [car](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#car)  
-   Bas correspond à [cab](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cab)  

## <a name="finding-visually-similar-images"></a>Recherche d’images visuellement similaires  

Pour rechercher des images qui sont visuellement similaires à l’image d’origine, définissez le paramètre de requête [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) sur SimilarImages.  

La requête suivante montre comment obtenir des images visuellement similaires. La requête utilise le paramètre de requête [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) pour identifier l’image d’origine. Pour améliorer la pertinence, vous devez inclure la chaîne de requête de l’utilisateur.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?insightsToken=mid_68364D764J...&modules=SimilarImages&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```


Voici la réponse à la requête précédente.  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "visuallySimilarImages" : {
        "value" : [
            {
                "name" : "typical Hawaiian Sunset! :) | Scenes of Hawaii",
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=detailv2...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP.Mda2a86...",
                 . . .
            }
        ]
    }
```

## <a name="cropping-images-to-use-with-visually-similar-and-entity-recognition-modules"></a>Rognage d’images à utiliser avec les modules de reconnaissance Visuellement similaires et Entités  

Pour spécifier la région de l’image utilisée par Bing pour déterminer si des images sont visuellement similaires ou pour effectuer une reconnaissance d’entité, utilisez les paramètres de requête [cal](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cal), [cat](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cat), [cab](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cab) et [car](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#car). Par défaut, Bing utilise l’image entière.  

Les paramètres spécifient les coins supérieur, gauche, droit et inférieur de la région utilisée par Bing pour la comparaison. Spécifiez les valeurs sous forme de fractions de la largeur et de la hauteur de l’image d’origine. Les valeurs fractionnaires commencent à (0.0, 0.0) dans le coin supérieur gauche et se terminent par (1.0, 1.0) dans le coin inférieur droit. Par exemple, pour spécifier que le coin supérieur gauche démarre à un quart vers le bas à partir du haut et à un quart à partir de la gauche, définissez `cal` sur 0,25 et `cat` sur 0,25.  

La séquence d’appels suivante montre l’effet de la définition de la zone de rognage. Le premier appel n’inclut pas de rognage et Bing reconnaît deux personnes côte à côte au milieu de l’image.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?modules=RecognizedEntities&imgurl=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M0cbee6fadb43f35b2344e53da7a23ec1o0%26pid%3DApi&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```  

La réponse présente deux entités reconnues.  

```json
{  
    "_type" : "ImageInsights",  
    "recognizedEntityGroups" : {
        "value": [  
            . . .  
            {  
                "recognizedEntityRegions" : [{  
                    "region" : {  
                        "left" : 0.5066667,  
                        "top" : 0.1955556,  
                        "right" : 0.75,  
                        "bottom" : 0.52  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Charlene Whitney",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                },  
                {  
                    "region" : {  
                        "left" : 0.25,  
                        "top" : 0.2488889,  
                        "right" : 0.4466667,  
                        "bottom" : 0.5111111  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Marcus Appel",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                }],  
            "name" : "CelebRecognitionAnnotations"
        }]
    }  
}  
```  

Le deuxième appel rogne l’image verticalement vers le bas au milieu et Bing n’a reconnu qu’une seule personne à droite de l’image.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?cal=0.5&cat=0.0&car=1.0&cab=1.0&modules=RecognizedEntities&imgurl=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M0cbee6fadb43f35b2344e53da7a23ec1o0%26pid%3DApi&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

La réponse présente une entité reconnue.  

```json  
{  
    "_type" : "ImageInsights",  
    "recognizedEntityGroups" : {
        "value" : [  
            . . .  
            {  
                "recognizedEntityRegions" : [{  
                    "region" : {  
                        "left" : 0.5066667,  
                        "top" : 0.1955556,  
                        "right" : 0.75,  
                        "bottom" : 0.52  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Charlene Whitney",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                }],  
                "name" : "CelebRecognitionAnnotations"  
            }
        ]
    }
}  
```  

## <a name="finding-visually-similar-products"></a>Recherche de produits visuellement similaires  

Pour rechercher des images contenant des produits qui sont visuellement similaires aux produits trouvés dans l’image d’origine, affectez la valeur SimilarProducts au paramètre de requête [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested).  

La requête suivante montre comment obtenir des images de produits visuellement similaires. La requête utilise le paramètre de requête [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) pour identifier l’image d’origine qui a été retournée dans une requête précédente. Pour améliorer la pertinence, vous devez inclure la chaîne de requête de l’utilisateur.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=anne+klein+dresses&modules=SimilarProducts&insightsToken=ccid_WOeyfoSp*mid_4B0A357&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

Voici la réponse à la requête précédente. La réponse contient une image d’un produit similaire et indique combien de prestataires proposent le produit en ligne, s’il existe des classements de produit, et le prix le plus bas trouvé (consultez le champ `aggregateOffer`).  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "visuallySimilarProducts" : {
        "value" : [
            {
                "name" : "Sequin One-Shoulder Twist-Drape Dress",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=de...",  
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=OIP.M85bdee...",  
                . . .
            },
            . . .
        ]
    }
}
```

Pour obtenir la liste des prestataires qui proposent le produit en ligne (consultez le champ [offerCount](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)), rappelez l’API et définissez `modules` sur ShoppingSources. Définissez ensuite le paramètre de requête `insightsToken` sur le jeton trouvé dans le fichier de résumé du produit.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?modules=ShoppingSources&insightsToken=ccid_hb3uRvUk*mid_BF5C252A47F2C765...&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

Voici la réponse à la requête précédente.  

```json  
{  
    "_type" : "ImageInsights",  
    "shoppingSources" : {  
        "offers" : [{  
            "url" : "http:\/\/www.contoso.com\/dp\/B00O...",  
            "seller" : {  
                "name" : "Contoso",  
                "image" : {  
                    "url" : "https:\/\/tse3.mm.bing.net\/th?id=A10d50fe..."  
                }  
            },  
            "price" : 126.87,  
            "priceCurrency" : "USD",  
            "availability" : "InStock"  
        },  
        {  
            "url" : "http:\/\/www.adatum.com\/product\/heritage...\/",  
            "seller" : {  
                "name" : "fabrikam.com"  
            },  
            "price" : 495,  
            "priceCurrency" : "USD",  
            "availability" : "InStock"  
        }]  
    }  
}  
```