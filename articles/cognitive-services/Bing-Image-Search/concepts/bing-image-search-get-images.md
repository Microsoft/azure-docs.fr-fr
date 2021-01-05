---
title: 'Obtention d’images à partir du web : API Recherche d’images Bing'
titleSuffix: Azure Cognitive Services
description: Utilisez l’API Recherche d’images Bing pour rechercher et télécharger des images pertinentes à partir du web.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: AB1B9898-C94A-4B59-91A1-8623C94BA3D4
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: aahi
ms.openlocfilehash: 988a1332d03bf2c9563ab0576f7a20ee6b0615aa
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96342054"
---
# <a name="get-images-from-the-web-with-the-bing-image-search-api"></a>Obtenez des images à partir du web avec l’API Recherche d’images Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Lorsque vous utilisez l’API REST Recherche d’images Bing, vous pouvez obtenir des images web qui sont liées au terme de votre recherche en envoyant la requête GET suivante :

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Utilisez le paramètre de requête [q](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#query) pour votre terme de recherche codé URL. Par exemple, si vous saisissez *sailing dinghies*, définissez `q` avec la valeur `sailing+dinghies` ou `sailing%20dinghies`.

> [!IMPORTANT]
> * Toutes les requêtes doivent provenir d’un serveur, et non d’un client.
> * Si vous appelez l’une des API de recherche Bing pour la première fois, n’incluez pas l’en-tête ID client. Indiquez uniquement l’ID client si vous avez précédemment appelé une API Bing qui a renvoyé un ID client pour la combinaison utilisateur/appareil.

## <a name="get-images-from-a-specific-web-domain"></a>Obtenir des images à partir d’un domaine web spécifique

Pour obtenir des images à partir d’un domaine spécifique, utilisez l’opérateur de requête [site:](/previous-versions/bing/search/ff795613(v=msdn.10)).

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us HTTP/1.1
```

> [!NOTE]
> Les réponses aux requêtes utilisant l’opérateur `site:` peuvent inclure du contenu pour adultes, quel que soit le paramètre [safeSearch](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#safesearch). N’utilisez `site:` que si vous connaissez le contenu du domaine.

## <a name="filter-images"></a>Filtrer les images

 Par défaut, l’API Recherche d’images renvoie toutes les images ayant un rapport avec la requête. Si vous souhaitez filtrer les images que Bing renvoie (par exemple, pour ne recevoir que des images avec un arrière-plan transparent ou d’une taille spécifique), utilisez les paramètres de requête suivants :

* [aspect](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#aspect) : filtre les images selon leurs proportions (par exemple, standard ou grand écran).
* [color](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#color) : filtre les images selon leur couleur (couleur dominante ou noir et blanc).
* [freshness](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#freshness) : filtre les images selon leur ancienneté (par exemple, celles découvertes par Bing la semaine dernière).
* [height](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#height), [width](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#width) : filtre les images selon leur largeur et leur hauteur.
* [imageContent](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imagecontent) : filtre les images selon leur contenu (par exemple, celles qui montrent uniquement le visage d’une personne).
* [imageType](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imagetype) : filtre les images selon leur type (images clipart, images GIF animées, arrière-plans transparents, etc.).
* [license](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#license) : filtre les images selon le type de contrat de licence associé au site.
* [size](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#size) : filtre les images selon leur taille (par exemple, petites images jusqu’à 200 x 200 pixels).

Pour obtenir des images à partir d’un domaine spécifique, utilisez l’opérateur de requête [site:](/previous-versions/bing/search/ff795613(v=msdn.10)).

L’exemple suivant montre comment obtenir les petites images découvertes par Bing la semaine dernière sur le site ContosoSailing.com.  

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&size=small&freshness=week&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

## <a name="bing-image-search-response-format"></a>Format de réponse de Recherche d’images Bing

Le message de réponse de Bing contient une réponse [Images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images) qui inclut une liste d’images que Cognitives Services a déterminé comme étant pertinentes pour la requête. Chaque objet [Image](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#image) de la liste inclut les informations suivantes sur l’image : l’URL, sa taille, ses dimensions, son format d’encodage, une URL vers une miniature de l’image et les dimensions de la miniature.

> [!NOTE]
> * Les images doivent être affichées dans l’ordre dans lequel elles apparaissent dans la réponse.
> * Étant donné que les paramètres et formats d’URL sont susceptibles de changer sans préavis, utilisez toutes les URL telles quelles. Vous ne devez pas créer de dépendances par rapport au format d’URL ou aux paramètres sauf indication contraire.

```json
{
    "name": "Rich Passage Sailing Dinghy",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=73118C8B4E3...",
    "thumbnailUrl": "https:\/\/tse1.mm.bing.net\/th?id=OIP.GNarK7m...",
    "datePublished": "2011-10-29T11:26:00",
    "contentUrl": "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3...",
    "hostPageUrl": "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3687...",
    "contentSize": "79239 B",
    "encodingFormat": "jpeg",
    "hostPageDisplayUrl": "en.contoso.org\/wiki\/File:Rich_Passage...",
    "width": 526,
    "height": 688,
    "thumbnail": {
        "width": 229,
        "height": 300
    },
    "imageInsightsToken": "ccid_GNarK7ma*mid_CCF85447ADA6...",
    "insightsSourcesSummary": {
        "shoppingSourcesCount": 0,
        "recipeSourcesCount": 0
    },
    "imageId": "CCF85447ADA6FFF9E96E7DF0B796F7A86E34593",
    "accentColor": "376094"
},
```

Quand vous appelez l’API Recherche d’images Bing, Bing retourne une liste de résultats. La liste est un sous-ensemble du nombre total de résultats en rapport avec la requête. Le champ `totalEstimatedMatches` de la réponse contient une estimation du nombre d’images disponibles pour visualisation. Pour plus d’informations sur la manière de parcourir les images restantes, consultez [Parcourir les images](../../bing-web-search/paging-search-results.md).

## <a name="next-steps"></a>Étapes suivantes

Si vous n’avez pas déjà essayé l’API Recherche d’images Bing, essayez un [démarrage rapide](../quickstarts/csharp.md). Si vous recherchez quelque chose de plus complexe, suivez le didacticiel de création d’une [application web à une seule page](../tutorial-bing-image-search-single-page-app.md).