---
title: Envoi de demandes à l’API Suggestion automatique Bing
titleSuffix: Azure Cognitive Services
description: L’API Suggestion automatique Bing retourne une liste de requêtes suggérées en fonction d’une chaîne de requête partielle dans la zone de recherche. Découvrez-en plus sur l’envoi de demandes.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-autosuggest
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: scottwhi
ms.openlocfilehash: dd845c0fb877afa76b84eb5c2d86392f763eccf7
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353389"
---
# <a name="sending-requests-to-the-bing-autosuggest-api"></a>Envoi de demandes à l’API Suggestion automatique Bing.

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Si votre application envoie des requêtes à l’une des API Recherche Bing, vous pouvez utiliser l’API Suggestion automatique Bing pour améliorer l’expérience de recherche de vos utilisateurs. L’API Suggestion automatique Bing retourne une liste de requêtes suggérées en fonction d’une chaîne de requête partielle dans la zone de recherche. À mesure que des caractères sont entrés dans la zone de recherche de votre application, vous pouvez afficher des suggestions dans une liste déroulante. Utilisez cet article pour savoir comment envoyer des requêtes de recherche cette API. 

## <a name="bing-autosuggest-api-endpoint"></a>Point de terminaison de l’API Suggestion automatique Bing

L'**API Suggestion automatique Bing** inclut un point de terminaison, qui renvoie une liste de requêtes suggérées à partir d’un terme de recherche partielle.

Pour obtenir des suggestions de requête à l’aide de l’API Bing, envoyez une requête `GET` au point de terminaison suivant. Utilisez les en-têtes et les paramètres d’URL pour définir d’autres spécifications.

**Point de terminaison :** Retourne les suggestions de recherche en tant que résultats JSON qui correspondent à l’entrée utilisateur définie par `?q=""`.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/Suggestions 
```

Pour plus d’informations sur les en-têtes, les paramètres, les codes de marché, les objets de réponse, les erreurs, etc., consultez la référence [API Suggestion automatique Bing v7](/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference).

Les API **Bing** prennent en charge les actions de recherche qui renvoient des résultats en fonction de leur type.  Tous les points de terminaison de recherche renvoient des résultats en tant qu’objets de réponse JSON.
Tous les points de terminaison prennent en charge les requêtes qui renvoient une langue et/ou un emplacement spécifique par longitude, latitude et rayon de recherche.

Pour plus d’informations sur les paramètres pris en charge par chaque point de terminaison, consultez les pages de référence pour chaque type.
Pour obtenir des exemples de requêtes de base à l’aide de l’API Suggestion automatique, consultez les [démarrages rapides de suggestion automatique](/azure/cognitive-services/Bing-Autosuggest).

## <a name="bing-autosuggest-api-requests"></a>Requêtes API Suggestion automatique Bing

> [!NOTE]
> * Les requêtes envoyées à l’API Suggestion automatique Bing doivent utiliser le protocole HTTPS.

Nous vous recommandons de générer toutes les requêtes à partir d’un serveur. Si vous diffusez la clé dans le cadre d’une application client, vous prenez le risque qu’un tiers malveillant puisse y accéder. Par ailleurs, en appelant l’API à partir d’un serveur, vous disposez d’un unique point de mise à niveau pour les prochaines versions.

La requête doit indiquer le paramètre [q](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query) qui contient le terme de recherche partiel de l’utilisateur. La requête peut également indiquer le paramètre [mkt](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#mkt) (facultatif), qui vous permet de choisir le marché d’où proviennent les résultats. Pour obtenir la liste complète des paramètres de requête facultatifs, consultez la page [Paramètres de requête](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query-parameters). Toutes les valeurs de paramètres de requête doivent être codées au format URL.

La requête doit indiquer l’en-tête [Ocp-Apim-Subscription-Key](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#subscriptionkey). Nous vous conseillons également d’indiquer les en-têtes suivants (qui sont facultatifs) :

- [User-Agent](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#useragent)
- [X-MSEdge-ClientID](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientid)
- [X-Search-ClientIP](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientip)
- [X-Search-Location](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#location)

Les en-têtes d’emplacement et d’adresse IP client sont importants, car ils permettent de renvoyer du contenu géolocalisé.

Pour obtenir la liste complète des en-têtes de requête et de réponse, consultez la page [En-têtes](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#headers).

> [!NOTE]
> Lorsque vous appelez l’API Suggestion automatique Bing en JavaScript, les fonctionnalités de sécurité intégrées de votre navigateur peuvent dans certains cas vous empêcher d’accéder aux valeurs de ces en-têtes.

Pour résoudre ce problème, vous pouvez envoyer la demande à l’API Suggestion automatique Bing par le biais d’un proxy CORS. La réponse émanant d’un proxy de ce type a un en-tête `Access-Control-Expose-Headers` qui filtre les en-têtes de réponse et les rend accessibles à du code JavaScript.

Il est facile d’installer un proxy CORS pour autoriser [l’application du didacticiel](../tutorials/autosuggest.md) à accéder aux en-têtes clients facultatifs. Tout d’abord, [installez Node.js](https://nodejs.org/en/download/) si ce n’est pas déjà fait. Entrez alors la commande suivante dans l’invite de commandes.

```console
npm install -g cors-proxy-server
```

Ensuite, remplacez le point de terminaison de l’API Suggestion automatique Bing dans le fichier HTML par :

```http
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/Suggestions
```

Enfin, lancez le proxy CORS avec la commande suivante :

```console
cors-proxy-server
```

Laissez la fenêtre de commande ouverte pendant que vous utilisez l’application du tutoriel ; si vous fermez la fenêtre, le proxy s’arrête. Dans la section des en-têtes HTTP (qui peut être développée) sous les résultats de la recherche, vous pouvez maintenant voir l’en-tête `X-MSEdge-ClientID` (entre autres) et vérifier qu’il est identique pour toutes les requêtes.

Les requêtes doivent inclure tous les paramètres et en-têtes suggérés. 

L’exemple suivant illustre une requête qui renvoie les chaînes de requête suggérées pour *sail* (voile).

> ```http
> GET https://api.cognitive.microsoft.com/bing/v7.0/suggestions?q=sail&mkt=en-us HTTP/1.1
> Ocp-Apim-Subscription-Key: 123456789ABCDE
> X-MSEdge-ClientIP: 999.999.999.999
> X-Search-Location: lat:47.60357;long:-122.3295;re:100
> X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
> Host: api.cognitive.microsoft.com
> ```

Si vous appelez l’une des API Bing pour la première fois, n’incluez pas l’en-tête ID client. Indiquez uniquement l’en-tête ID client si vous avez précédemment appelé une API Bing et que Bing a retourné un ID client pour la combinaison utilisateur/appareil.

Le groupe de suggestions web suivant est une réponse à la requête ci-dessus. Le groupe contient une liste de suggestions de requête de recherche, chaque suggestion contenant des champs `displayText`, `query` et `url`.

Le champ `displayText` correspond à la requête suggérée à utiliser dans la liste déroulante de la zone de recherche. Vous devez afficher toutes les suggestions incluses dans la réponse, dans l’ordre indiqué.  

Si l’utilisateur sélectionne une requête dans la liste déroulante, vous pouvez l’utiliser pour appeler l’une des [API Recherche Bing](../../bing-web-search/bing-api-comparison.md?bc=%2fen-us%2fazure%2fbread%2ftoc.json&toc=%2fen-us%2fazure%2fcognitive-services%2fbing-autosuggest%2ftoc.json) et afficher les résultats vous-même, ou bien renvoyer l’utilisateur vers la page de résultats de Bing à l’aide du champ `url`.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "Suggestions",
    "queryContext" : {
        "originalQuery" : "sail"
    },
    "suggestionGroups" : [{
        "name" : "Web",
        "searchSuggestions" : [{
            "url" : "https:\/\/www.bing.com\/search?q=sailing+lessons+seattle&FORM=USBAPI",
            "displayText" : "sailing lessons seattle",
            "query" : "sailing lessons seattle",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+moon+news&FORM=USBAPI",
            "displayText" : "sailor moon news",
            "query" : "sailor moon news",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+jack%27s+lincoln+city&FORM=USBAPI",
            "displayText" : "sailor jack's lincoln city",
            "query" : "sailor jack's lincoln city",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailing+anarchy&FORM=USBAPI",
            "displayText" : "sailing anarchy",
            "query" : "sailing anarchy",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale&FORM=USBAPI",
            "displayText" : "sailboats for sale",
            "query" : "sailboats for sale",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailstn.mylabsplus.com&FORM=USBAPI",
            "displayText" : "sailstn.mylabsplus.com",
            "query" : "sailstn.mylabsplus.com",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailusfood&FORM=USBAPI",
            "displayText" : "sailusfood",
            "query" : "sailusfood",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale+seattle&FORM=USBAPI",
            "displayText" : "sailboats for sale seattle",
            "query" : "sailboats for sale seattle",
            "searchKind" : "WebSearch"
        }]
    }]
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce que la Suggestion automatique Bing ?](../get-suggested-search-terms.md)
- [Informations de référence sur l’API Suggestion automatique Bing v7](/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference)
- [Obtention de suggestions de termes de recherche avec l’API Suggestion automatique Bing](get-suggestions.md)