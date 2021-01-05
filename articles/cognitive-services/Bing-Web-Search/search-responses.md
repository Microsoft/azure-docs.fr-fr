---
title: Types et structure de la réponse de l’API Recherche Web Bing
titleSuffix: Azure Cognitive Services
description: Lorsque vous envoyez une demande de recherche à l’API Recherche Web Bing, celle-ci retourne un objet `SearchResponse` dans le corps de la réponse.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 06/25/2019
ms.author: aahi
ms.custom: seodec2018
ms.openlocfilehash: 3dda95312a0b9191ddc11de62959f308ee19fff4
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94380978"
---
# <a name="bing-web-search-api-response-structure-and-answer-types"></a>Types et structure de la réponse de l’API Recherche Web Bing  

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020** , toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Lorsque vous envoyez une demande de recherche à l’API Recherche Web Bing, celle-ci retourne un objet [`SearchResponse`](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#searchresponse) dans le corps de la réponse. L’objet inclut un champ pour chaque réponse que Bing considère comme pertinente vis-à-vis de la requête. Cet exemple montre comment se présente un objet de réponse lorsque Bing retourne toutes les réponses :

```json
{
    "_type": "SearchResponse",
    "queryContext": {...},
    "webPages": {...},
    "images": {...},
    "relatedSearches": {...},
    "videos": {...},
    "news": {...},
    "spellSuggestion": {...},
    "computation": {...},
    "timeZone": {...},
    "rankingResponse": {...}
}, ...
```

En général, l’API Recherche Web Bing retourne un sous-ensemble des réponses. Par exemple, si le terme de requête est *sailing dinghies* , la réponse peut inclure `webPages`, `images` et `rankingResponse`. À moins que vous n’utilisiez [responseFilter](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#responsefilter) pour filtrer les pages web, la réponse inclut toujours les réponses `webpages` et `rankingResponse`.

[!INCLUDE [cognitive-services-bing-url-note](../../../includes/cognitive-services-bing-url-note.md)]

## <a name="webpages-answer"></a>Réponse webPages

La réponse de type [webPages](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#webanswer) contient une liste de liens vers des pages web que l’API Recherche Web Bing considère comme pertinentes vis-à-vis de la requête. Chaque [page web](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#webpage) de la liste inclut le nom de la page, son URL, son URL d’affichage, une brève description du contenu et la date à laquelle Bing a trouvé le contenu.

```json
{
    "id": "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#WebPages.0",
    "name": "Dinghy sailing",
    "url": "https:\/\/www.bing.com\/cr?IG=3A43CA5...",
    "displayUrl": "https:\/\/en.contoso.com\/wiki\/Dinghy_sailing",
    "snippet": "Dinghy sailing is the activity of sailing small boats...",
    "dateLastCrawled": "2017-04-05T16:25:00"
}, ...
```

Utilisez `name` et `url` pour créer un lien hypertexte permettant à l’utilisateur d’accéder à la page web.

<!-- Remove until this can be replaced with a sanitized version.
The following shows an example of how you might display the webpage in a search results page.

![Rendered webpage example](./media/cognitive-services-bing-web-api/bing-rendered-webpage-example.PNG)
-->

## <a name="images-answer"></a>Réponse d’images

La réponse de type [images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images) contient une liste d’images que Bing considère comme pertinentes vis-à-vis de la requête. Chaque [image](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#image) de la liste inclut l’URL de l’image, sa taille, ses dimensions et son format d’encodage. L’objet Image comprend également l’URL et les dimensions d’une miniature de l’image.

```json
{
    "name": "Rich Passage Sailing Dinghy",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=3A43CA5CA64...",
    "thumbnailUrl": "https:\/\/tse1.mm.bing.net\/th?id=OIP....",
    "datePublished": "2011-10-29T11:26:00",
    "contentUrl": "http:\/\/upload.contoso.com\/sailing\/...",
    "hostPageUrl": "http:\/\/www.bing.com\/cr?IG=3A43CA5CA6464....",
    "contentSize": "79239 B",
    "encodingFormat": "jpeg",
    "hostPageDisplayUrl": "http:\/\/en.contoso.com\/wiki\/File...",
    "width": 526,
    "height": 688,
    "thumbnail": {
        "width": 229,
        "height": 300
    },
    "insightsSourcesSummary": {
        "shoppingSourcesCount": 0,
        "recipeSourcesCount": 0
    }
}, ...
```

Selon l’appareil de l’utilisateur, vous allez généralement afficher un sous-ensemble des miniatures et offrir à l’utilisateur la possibilité de [parcourir](./paging-search-results.md) les images restantes.

<!-- Remove until this can be replaced with a sanitized version.
![List of thumbnail images](./media/cognitive-services-bing-web-api/bing-web-image-thumbnails.PNG)
-->

Vous pouvez également développer la miniature quand l’utilisateur pointe sur celle-ci. Veillez à attribuer l’image si vous la développez. Vous pouvez, par exemple, extraire l’hôte de `hostPageDisplayUrl` et l’afficher sous l’image. Pour plus d’informations sur le redimensionnement de la miniature, consultez [Redimensionnement et rognage de miniatures](./resize-and-crop-thumbnails.md).

<!-- Remove until this can be replaced with a sanitized version.
![Expanded view of thumbnail image](./media/cognitive-services-bing-web-api/bing-web-image-thumbnail-expansion.PNG)
-->

Si l’utilisateur clique sur la miniature, utilisez `webSearchUrl` pour le faire accéder à la page de résultats de recherche Bing relative aux images, qui présente un montage des images.

Pour plus d’informations sur la réponse de type images et les images, consultez [API Recherche d’images](../bing-image-search/overview.md).

## <a name="related-searches-answer"></a>Réponse de type recherches connexes

La réponse [relatedSearches](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#searchresponse-relatedsearches) contient une liste des requêtes connexes les plus populaires effectuées par d’autres utilisateurs. Chaque [requête](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query_obj) de la liste inclut une chaîne de requête (`text`), une chaîne de requête avec des caractères de mise en surbrillance des correspondances (`displayText`) et l’URL (`webSearchUrl`) de la page de résultats de recherche Bing pour cette requête.

```json
{
    "text": "dinghy racing teams",
    "displayText": "dinghy racing teams",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=96C4CF214A0..."
}, ...
```

Utilisez la chaîne de requête `displayText` et l’URL `webSearchUrl` pour créer un lien hypertexte permettant à l’utilisateur d’accéder à la page de résultats de recherche Bing relative à la requête connexe. Vous pouvez également utiliser la chaîne de requête `text` dans votre propre requête à l’API Recherche Web Bing et afficher vous-même les résultats.

Pour plus d’informations sur la façon de gérer les marqueurs de mise en surbrillance dans `displayText`, consultez [Mise en surbrillance des correspondances](../bing-web-search/hit-highlighting.md).

L’exemple suivant illustre l’utilisation des requêtes connexes dans Bing.com.

![Exemple de recherche connexe sur Bing](./media/cognitive-services-bing-web-api/bing-web-rendered-relatedsearches.GIF)

## <a name="videos-answer"></a>Réponse de type vidéos

La réponse de type [vidéos](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videos) contient une liste de vidéos que Bing considère comme pertinentes vis-à-vis de la requête. Chaque [vidéo](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video) de la liste inclut l’URL de la vidéo, sa durée, ses dimensions et son format d’encodage. L’objet vidéo comprend également l’URL et les dimensions d’une miniature de la vidéo.

```json
{
    "name": "Sailing dinghy",
    "description": "Northwind Traders is a 12 foot gunter rigged...",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=1CAE739681D84...",
    "thumbnailUrl": "https:\/\/tse2.mm.bing.net\/th?id=OVP.wsKiL...",
    "datePublished": "2013-11-06T01:56:28",
    "publisher": [{
        "name": "Fabrikam"
    }],
    "contentUrl": "https:\/\/www.fabrikam.com\/watch?v=MrVBWZpJjX",
    "hostPageUrl": "https:\/\/www.bing.com\/cr?IG=1CAE739681D8400DB...",
    "encodingFormat": "mp4",
    "hostPageDisplayUrl": "https:\/\/www.fabrikam.com\/watch?v=MrBWZpJjXo",
    "width": 1280,
    "height": 720,
    "duration": "PT3M47S",
    "motionThumbnailUrl": "https:\/\/tse2.mm.bing.net\/th?id=OM.oa...",
    "embedHtml": "<iframe width=\"1280\" height=\"720\" src=\"http:\/\/www....><\/iframe>",
    "allowHttpsEmbed": true,
    "viewCount": 19089,
    "thumbnail": {
        "width": 300,
        "height": 168
    },
    "allowMobileEmbed": true,
    "isSuperfresh": false
}, ...
```

Selon l’appareil de l’utilisateur, vous allez généralement afficher un sous-ensemble des vidéos et offrir à l’utilisateur la possibilité d’afficher les vidéos restantes. Vous allez afficher une miniature de la vidéo avec sa longueur, une description (nom) et une attribution (éditeur).

<!-- Remove until this can be replaced with a sanitized version.
![List of video thumbnails](./media/cognitive-services-bing-web-api/bing-web-video-thumbnails.PNG)
-->

Vous pouvez utiliser `motionThumbnailUrl` pour lire une version miniature de la vidéo lorsque l’utilisateur pointe sur la miniature. Veillez à attribuer la vidéo miniature lorsque vous l’affichez.

<!-- Remove until this can be replaced with a sanitized version.
![Motion thumbnail of a video](./media/cognitive-services-bing-web-api/bing-web-video-motion-thumbnail.PNG)
-->

Les options d’affichage vidéo suivantes sont disponibles lorsque l’utilisateur clique sur la miniature :

- Utilisez `hostPageUrl` pour afficher la vidéo sur le site web hôte (par exemple, YouTube).
- Utilisez `webSearchUrl` pour afficher la vidéo dans le navigateur vidéo Bing.
- Utilisez `embedHtml` pour incorporer la vidéo à votre propre expérience.

Pour plus d’informations sur les vidéos et la réponse de type vidéos, consultez [API Recherche de vidéos](../bing-video-search/overview.md).

## <a name="news-answer"></a>Réponse de type actualités

La réponse de type [actualités](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#news) contient une liste d’articles d’actualités que Bing considère comme pertinents vis-à-vis de la requête. Chaque [article d’actualités](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) de la liste inclut le nom de l’article, une description et l’URL de l’article sur le site web de l’hôte. Si l’article contient une image, l’objet inclut une miniature de l’image.

```json
{
    "name": "WC Sailing Qualifies for America Trophy with...",
    "url": "http:\/\/www.bing.com\/cr?IG=3445EEF15DAF4FFFBF7...",
    "image": {
        "contentUrl": "http:\/\/www.contoso.com\/sports\/sail...",
        "thumbnail": {
            "contentUrl": "https:\/\/www.bing.com\/th?id=ON.1...",
            "width": 400,
            "height": 272
        }
    },
    "description": "The WC sailing team qualified for a...",
    "provider": [{
        "_type": "Organization",
        "name": "contoso.com"
    }],
    "datePublished": "2017-04-16T21:56:00"
}, ...
```

Selon l’appareil de l’utilisateur, vous allez afficher un sous-ensemble des articles d’actualités et offrir à l’utilisateur la possibilité d’afficher les articles restants. Utilisez `name` et `url` pour créer un lien hypertexte permettant à l’utilisateur d’accéder à l’article d’actualité sur le site de l’hôte. Si l’article inclut une image, rendez-la interactive en utilisant `url`. Veillez à utiliser `provider` pour attribuer l’article.

<!-- Remove until this can be replaced with a sanitized version.
The following shows an example of how you might display articles in a search results page.

![List of news articles](./media/cognitive-services-bing-web-api/bing-web-news-list.PNG)
-->

Pour plus d’informations sur la réponse de type actualités et les articles d’actualités, consultez [API Recherche d’actualités](../bing-news-search/search-the-web.md).

## <a name="computation-answer"></a>Réponse de type calcul

Si l’utilisateur entre une requête incluant une expression mathématique ou portant sur une conversion d’unités, la réponse peut contenir une réponse de type [calcul](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#computation). La réponse `computation` contient l’expression normalisée et son résultat.

Une requête de conversion d’unité est une requête qui convertit une unité en une autre. Par exemple, *À combien de pieds équivalent 10 mètres ?* ou *Combien de cuillères à soupe représente un quart de tasse ?*

L’exemple suivant montre comment se présente la réponse `computation` pour la requête *À combien de pieds équivalent 10 mètres ?*

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "10 meters",
    "value": "32.808399 feet"
}, ...
```

L’exemple suivant présente des requêtes mathématiques et les réponses `computation` correspondantes.

```
Query: (5+3)(10/2)+8
Encoded query: %285%2B3%29%2810%2F2%29%2B8
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "((5+3)*(10\/2))+8",
    "value": "48"
}
```

```
Query: sqrt(4^2+8^2)
Encoded query: sqrt%284^2%2B8^2%29
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
    "expression": "sqrt((4^2)+(8^2))",
    "value": "8.94427191"
}
```

```
Query: 30 6/8 - 18 8/16
Encoded query: 30%206%2F8%20-%2018%208%2F16
```

```json
"computation": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#WolframAlpha",
    "expression": "30 6\/8-18 8\/16",
    "value": "12.25"
}
```

```
Query: 8^2+11^2-2*8*11*cos(37)
Encoded query: 8^2%2B11^2-2*8*11*cos%2837%29
```

```json
"computation": {
        "id": "https:\/\/www.bing.com\/api\/v7\/#Computation",
        "expression": "(8^2)+(11^2)-(2*8*11*cos(37))",
        "value": "44.4401502"
}
```

Une expression mathématique peut contenir les symboles suivants :

|Symbole|Description|
|------------|-----------------|
|+|Addition|
|-|Soustraction|
|/|Division|
|*|Multiplication|
|^|Power|
|!|Factorielle|
|.|Decimal|
|()|Calcul prioritaire|
|[]|Fonction|

Une expression mathématique peut contenir les constantes suivantes :

|Symbole|Description|
|------------|-----------------|
|Pi|3,14159...|
|Degré|Degré|
|i|Nombre imaginaire|
|e|e, 2,71828...|
|GoldenRatio|Nombre d’or, 1,61803...|

Une expression mathématique peut contenir les fonctions suivantes :

|Symbole|Description|
|------------|-----------------|
|Trier|Racine carrée|
|Sin[x], Cos[x], Tan[x]<br />Csc[x], Sec[x], Cot[x]|Fonctions trigonométriques (avec arguments en radians)|
|ArcSin[x], ArcCos[x], ArcTan[x]<br />ArcCsc[x], ArcSec[x], ArcCot[x]|Fonctions trigonométriques inverses (donnant les résultats en radians)|
|Exp[x], E^x|Fonction exponentielle|
|Log[x]|Logarithme népérien|
|Sinh[x], Cosh[x], Tanh[x]<br />Csch[x], Sech[x], Coth[x]|Fonctions hyperboliques|
|ArcSinh[x], ArcCosh[x], ArcTanh[x]<br />ArcCsch[x], ArcSech[x], ArcCoth[x]|Fonctions hyperboliques inverses|

Les expressions mathématiques contenant des variables (par exemple, 4x+6=18, où x est la variable) ne sont pas prises en charge.

## <a name="timezone-answer"></a>Réponse de type fuseau horaire

Si l’utilisateur entre une requête portant sur une heure ou une date, la réponse peut contenir une réponse de type [fuseau horaire](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#timezone). Cette réponse prend en charge les requêtes implicites et explicites. Une requête implicite comme *Quelle heure est-il ?* retourne l’heure locale en fonction de l’emplacement de l’utilisateur. Une requête explicite comme *Quelle heure est-il à Seattle ?* retourne l’heure locale de Seattle, dans l’État de Washington.

La réponse `timeZone` fournit le nom de l’emplacement, la date et l’heure UTC actuelles à l’emplacement spécifié et le décalage UTC. Si l’emplacement couvre plusieurs fuseaux horaires, la réponse contient la date et l’heure UTC actuelles pour tous les fuseaux horaires concernés. Par exemple, l’État de Floride couvre deux fuseaux horaires. Par conséquent, la réponse contient la date et heure locales pour ces deux fuseaux horaires.  

Si la requête porte sur l’heure actuelle dans un État ou un pays/une région, Bing détermine la principale ville incluse dans les limites géographiques correspondantes et la retourne dans le champ `primaryCityTime`. Si ces limites couvrent plusieurs fuseaux horaires, les fuseaux horaires restants sont retournés dans le champ `otherCityTimes`.

L’exemple suivant illustre des requêtes retournant la réponse `timeZone`.

```
Query: What time is it?

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Redmond, Washington, United States",
        "time": "2015-10-27T08:38:12.1189231Z",
        "utcOffset": "UTC-7"
    }
}

Query: What time is it in the Pacific time zone?

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Pacific Time Zone",
        "time": "2015-10-23T12:33:19.0728146Z",
        "utcOffset": "UTC-7"
    }
}

Query: Time in Florida?

"timeZone": {
        "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
        "primaryCityTime": {
            "location": "Tallahassee, Florida, United States",
            "time": "2015-10-23T13:04:56.6774389Z",
            "utcOffset": "UTC-4"
        },
        "otherCityTimes": [{
            "location": "Pensacola",
            "time": "2015-10-23T12:04:56.6664294Z",
            "utcOffset": "UTC-5"
        }]
}

Query: What time is it in the U.S.

"timeZone": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#TimeZone",
    "primaryCityTime": {
        "location": "Washington, D.C., United States",
        "time": "2015-10-23T15:27:59.8892745Z",
        "utcOffset": "UTC-4"
    },
    "otherCityTimes": [{
        "location": "Honolulu",
        "time": "2015-10-23T09:27:59.8892745Z",
        "utcOffset": "UTC-10"
    },
    {
        "location": "Anchorage",
        "time": "2015-10-23T11:27:59.8892745Z",
        "utcOffset": "UTC-8"
    },
    {
        "location": "Phoenix",
        "time": "2015-10-23T12:27:59.8892745Z",
        "utcOffset": "UTC-7"
    },
    {
        "location": "Los Angeles",
        "time": "2015-10-23T12:27:59.8942788Z",
        "utcOffset": "UTC-7"
    },
    {
        "location": "Denver",
        "time": "2015-10-23T13:27:59.8812681Z",
        "utcOffset": "UTC-6"
    },
    {
        "location": "Chicago",
        "time": "2015-10-23T14:27:59.8892745Z",
        "utcOffset": "UTC-5"
    }]
}
```

## <a name="spellsuggestion-answer"></a>Réponse de type SpellSuggestion

Si Bing détermine que l’utilisateur rechercherait peut-être autre chose, la réponse inclut un objet [SpellSuggestions](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#spellsuggestions). Par exemple, si l’utilisateur recherche *carlos pen* , Bing peut déterminer qu’il recherche probablement Carlos Pena (en s’appuyant sur les recherches antérieures de *carlos pen* par d’autres utilisateurs). L’exemple suivant illustre une réponse de type suggestion orthographique.

```json
"spellSuggestions": {
    "id": "https:\/\/www.bing.com\/api\/v7\/#SpellSuggestions",
    "value": [{
        "text": "carlos pena",
        "displayText": "carlos pena"
    }]
}, ...
```

## <a name="response-headers"></a>En-têtes de réponse

Les réponses de l’API Recherche Web Bing peuvent contenir les en-têtes suivants :

| En-tête | Description |
|-|-|
|`X-MSEdge-ClientID`|L’ID unique que Bing a affecté à l’utilisateur|
|`BingAPIs-Market`|Le marché qui a été utilisé pour répondre à la requête|
|`BingAPIs-TraceId`|L’entrée du journal sur le serveur de l’API Bing pour cette requête (pour la prise en charge)|

Il est particulièrement important de conserver l’identifiant du client et de le renvoyer avec les requêtes suivantes. Lorsque vous effectuez cette opération, la recherche utilise un contexte passé pour classer les résultats de la recherche et propose également une expérience utilisateur cohérente.

Toutefois, lorsque vous appelez l’API Recherche Web Bing depuis JavaScript, les fonctionnalités de sécurité intégrées de votre navigateur (CORS) peuvent vous empêcher d’accéder aux valeurs de ces en-têtes.

Pour accéder aux en-têtes, vous pouvez effectuer la requête d’API Recherche Web Bing via un proxy CORS. La réponse provenant d’un proxy de ce type a un en-tête `Access-Control-Expose-Headers` qui filtre les en-têtes de réponse et les rend accessibles à JavaScript.

Il est facile d’installer un proxy CORS pour autoriser [l’application du didacticiel](tutorial-bing-web-search-single-page-app.md) à accéder aux en-têtes clients facultatifs. Tout d’abord, [installez Node.js](https://nodejs.org/en/download/) si ce n’est pas déjà fait. Entrez alors la commande suivante dans l’invite de commandes.

```console
npm install -g cors-proxy-server
```

Ensuite, remplacez le point de terminaison de l’API Recherche Web Bing dans le fichier HTML par :\
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search`

Enfin, lancez le proxy CORS avec la commande suivante :

```console
cors-proxy-server
```

Laissez la fenêtre de commande ouverte pendant que vous utilisez l’application du tutoriel ; si vous fermez la fenêtre, le proxy s’arrête. Dans la section des en-têtes HTTP (qui peut être développée) sous les résultats de la recherche, vous pouvez maintenant voir l’en-tête `X-MSEdge-ClientID` (entre autres) et vérifier qu’il est identique pour toutes les requêtes.

## <a name="response-headers-in-production"></a>En-têtes de réponse en production

L’approche du proxy CORS décrite dans la réponse précédente est appropriée pour le développement, le test et l’apprentissage.

Dans un environnement de production, vous devez héberger un script côté serveur dans le même domaine que la page web qui utilise l’API Recherche Web Bing. Ce script doit effectuer les appels d’API à la demande à partir de la page web JavaScript et transmettre tous les résultats, y compris les en-têtes, au client. Étant donné que les deux ressources (la page et le script) partagent une origine, le CORS n'est pas utilisé et les en-têtes spéciaux sont accessibles pour JavaScript dans la page web.

Cette approche protège également votre clé API de l’exposition au public, étant donné que seul le script côté serveur en a besoin. Le script peut utiliser une autre méthode pour s’assurer que la demande est autorisée.

L’exemple suivant montre comment Bing utilise la suggestion orthographique.

![Exemple de suggestion orthographique de Bing](./media/cognitive-services-bing-web-api/bing-web-spellingsuggestion.GIF)  

## <a name="next-steps"></a>Étapes suivantes  

* Passez en revue la documentation sur la [limitation de requêtes](throttling-requests.md).  

## <a name="see-also"></a>Voir aussi  

* [Référence pour l’API Recherche Web Bing](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference)