---
title: 'Tutoriel : Créer une application web monopage - API Recherche d’images Bing'
titleSuffix: Azure cognitive services
description: L’API Recherche d’images Bing vous permet de rechercher sur le web des images pertinentes de qualité. Utilisez ce tutoriel pour créer une application web monopage qui peut envoyer des requêtes de recherche à l’API et afficher les résultats dans la page web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: bcb82ffcf4c4ad861c6523f507c816c666a71772
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351382"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-image-search-api"></a>Tutoriel : Créer une application monopage avec l’API Recherche d’images Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’images Bing vous permet de rechercher sur le web des images pertinentes de qualité. Utilisez ce tutoriel pour créer une application web monopage qui peut envoyer des requêtes de recherche à l’API et afficher les résultats dans la page web. Ce tutoriel est similaire au [tutoriel correspondant](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md) pour l’API Recherche Web Bing.

L’application du didacticiel illustre les actions suivantes :

> [!div class="checklist"]
> * Effectuer un appel d’API Recherche d’images Bing dans JavaScript
> * Améliorer les résultats de la recherche à l’aide d’options de recherche
> * Afficher et parcourir les résultats de la recherche
> * Demander et gérer une clé d’abonnement d’API et l’ID de client Bing.

L’intégralité du code source pour ce tutoriel est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Image-Search).

## <a name="prerequisites"></a>Prérequis

* La dernière version de [Node.js](https://nodejs.org/).
* Le framework [Express.js](https://expressjs.com/) pour Node.js. Les instructions d’installation du code source sont disponibles dans le fichier Lisez-moi de l’exemple GitHub.

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="manage-and-store-user-subscription-keys"></a>Gérer et stocker des clés d’abonnement utilisateur

Cette application utilise le stockage persistant des navigateurs web pour stocker les clés d’abonnement d’API. Si aucune clé n’est stockée, la page web demande sa clé à l’utilisateur et la stocke pour l’utiliser ultérieurement. Si la clé est ensuite rejetée par l’API, l’application la supprime du stockage. Cet exemple utilise le point de terminaison global. Vous pouvez aussi utiliser le point de terminaison de [sous-domaine personnalisé](../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.


Définissez les fonctions `storeValue` et `retrieveValue` pour utiliser l’objet `localStorage` (si le navigateur le prend en charge) ou un cookie.

```javascript
// Cookie names for data being stored
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";
// The Bing Image Search API endpoint
BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/images/search";

try { //Try to use localStorage first
    localStorage.getItem;   

    window.retrieveValue = function (name) {
        return localStorage.getItem(name) || "";
    }
    window.storeValue = function(name, value) {
        localStorage.setItem(name, value);
    }
} catch (e) {
    //If the browser doesn't support localStorage, try a cookie
    window.retrieveValue = function (name) {
        var cookies = document.cookie.split(";");
        for (var i = 0; i < cookies.length; i++) {
            var keyvalue = cookies[i].split("=");
            if (keyvalue[0].trim() === name) return keyvalue[1];
        }
        return "";
    }
    window.storeValue = function (name, value) {
        var expiry = new Date();
        expiry.setFullYear(expiry.getFullYear() + 1);
        document.cookie = name + "=" + value.trim() + "; expires=" + expiry.toUTCString();
    }
}
```

La fonction `getSubscriptionKey()` tente de récupérer une clé déjà stockée à l’aide de `retrieveValue`. Si aucune clé n’est trouvée, elle demande sa clé à l’utilisateur, puis la stocke à l’aide de `storeValue`.

```javascript

// Get the stored API subscription key, or prompt if it's not found
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```

La balise `onsubmit` du code HTML `<form>` appelle la fonction `bingWebSearch` pour renvoyer les résultats de la recherche. `bingWebSearch` utilise `getSubscriptionKey` pour authentifier chaque requête. Comme indiqué dans la définition précédente, `getSubscriptionKey` demande une clé à l’utilisateur si la clé n’a pas été saisie. La clé est ensuite stockée en vue d’une utilisation continue par l’application.

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value,
bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="send-search-requests"></a>Envoyer des requêtes de recherche

Cette application utilise un `<form>` HTML pour envoyer initialement des requêtes de recherche utilisateur, à l’aide de l’attribut `onsubmit` pour appeler `newBingImageSearch()`.

```html
<form name="bing" onsubmit="return newBingImageSearch(this)">
```

Par défaut, le gestionnaire `onsubmit` retourne `false`, ce qui empêche l’envoi du formulaire.

## <a name="select-search-options"></a>Sélectionner des options de recherche

![[Formulaire de Recherche d’images Bing]](media/cognitive-services-bing-images-api/image-search-spa-form.png)

L’API Recherche d’images Bing offre plusieurs [paramètres de requête de filtre](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#filter-query-parameters) pour affiner et filtrer les résultats de la recherche. Le formulaire HTML de cette application utilise et affiche les options de paramètre suivantes :

| Option | Description |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `where`      | Menu déroulant pour sélectionner le marché (lieu et langue) utilisé pour la recherche.                                                                                             |
| `query`      | Champ de texte dans lequel saisir les termes de recherche.                                                                                                                                 |
| `aspect`     | Cases d’option pour choisir les proportions des images trouvées : carré approximatif, largeur ou hauteur.                                                                                     |
| `color`      |                                                                                                                                                                                    |
| `when`       | Menu déroulant pour éventuellement limiter la recherche à un jour, une semaine ou un mois.                                                                                          |
| `safe`       | Case à cocher indiquant s’il faut utiliser la fonctionnalité Filtre adulte de Bing afin de filtrer les résultats pour « adultes ».                                                                                      |
| `count`      | Champ masqué. Nombre de résultats de recherche à renvoyer pour chaque requête. À modifier pour afficher plus ou moins de résultats par page.                                                            |
| `offset`     | Champ masqué. Décalage du premier résultat de recherche dans la requête ; utilisé pour la pagination. Réinitialisé sur `0` lors d’une nouvelle requête.                                                           |
| `nextoffset` | Champ masqué. Lorsqu’un résultat de la recherche est reçu, ce champ est défini sur la valeur de `nextOffset` dans la réponse. L’utilisation de ce champ permet d’éviter des résultats qui se chevauchent sur des pages successives. |
| `stack`      | Champ masqué. Liste encodée JSON des décalages de pages précédentes de résultats de la recherche, pour revenir aux pages précédentes.                                                      |

La fonction `bingSearchOptions()` met en forme ces options dans une chaîne de requête partielle, qui peut être utilisée dans les requêtes d’API de l’application.  

```javascript
// Build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var aspect = "all";
    for (var i = 0; i < form.aspect.length; i++) {
        if (form.aspect[i].checked) {
            aspect = form.aspect[i].value;
            break;
        }
    }
    options.push("aspect=" + aspect);
    if (form.color.value) options.push("color=" + form.color.value);
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    return options.join("&");
}
```

## <a name="performing-the-request"></a>Exécution de la requête

En utilisant la requête de recherche, la chaîne d’options et la clé d’API, la fonction `BingImageSearch()` utilise un objet XMLHttpRequest pour effectuer la requête au point de terminaison Recherche d’images Bing.


```javascript
// perform a search given query, options string, and API key
function bingImageSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("results", "related", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // open the request
    try {
        request.open("GET", queryurl);
    }
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // add request headers
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // event handler for successful response
    request.addEventListener("load", handleBingResponse);

    // event handler for erorrs
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // event handler for aborted request
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // send the request
    request.send();
    return false;
}
```

Quand la requête HTTP aboutit, JavaScript appelle le gestionnaire d’événements « load » `handleBingResponse()` pour gérer une requête HTTP GET réussie.

```javascript
// handle Bing search request results
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and HTTP request
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " +
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "Images") {
                if (jsobj.nextOffset) document.forms.bing.nextoffset.value = jsobj.nextOffset;
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSubscriptionKey();

        // some error responses don't have a top-level errors object, so gin one up
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // display HTTP status code
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // add all fields from all error responses
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // also display Bing Trace ID if it isn't blocked by CORS
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // and display the error message
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> Les requêtes HTTP réussies peuvent contenir des informations relatives à des échecs de recherche. Si une erreur se produit pendant l’opération de recherche, l’API Recherche d’images Bing retourne un code d’état HTTP autre que 200 et des informations sur l’erreur dans la réponse JSON. De plus, si la requête avait un débit limité, l’API retourne une réponse vide.

## <a name="display-the-search-results"></a>Afficher les résultats de la recherche

Les résultats de la recherche sont affichés par la fonction `renderSearchResults()`, qui prend le code JSON retourné par le service Recherche d’images Bing et appelle une fonction de renderer appropriée sur n’importe quelle image retournée et les recherches associées.

```javascript
function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    showDiv("results", renderImageResults(results.value));
    if (results.relatedSearches)
        showDiv("sidebar", renderRelatedItems(results.relatedSearches));
}
```

Les résultats de recherche d’images sont contenus dans un objet `value` de niveau supérieur dans la réponse JSON. Ces résultats sont passés à `renderImageResults()`, qui itère au sein des résultats et convertit chaque élément en HTML.

```javascript
function renderImageResults(items) {
    var len = items.length;
    var html = [];
    if (!len) {
        showDiv("noresults", "No results.");
        hideDivs("paging1", "paging2");
        return "";
    }
    for (var i = 0; i < len; i++) {
        html.push(searchItemRenderers.images(items[i], i, len));
    }
    return html.join("\n\n");
}
```

L’API Recherche d’images Bing peut retourner quatre types de suggestions de recherche afin de guider les expériences de recherche des utilisateurs, chacun dans son propre objet de niveau supérieur :

| Suggestion         | Description                                                                                                                                                                                                         |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pivotSuggestions` | Requêtes qui remplacent un mot pivot de la recherche d’origine par un autre. Par exemple, si vous recherchez « fleurs rouges », « rouge » peut être un mot pivot et une suggestion peut être « fleurs jaunes ». |
| `queryExpansions`  | Requêtes qui affinent votre recherche d’origine en ajoutant d’autres termes. Par exemple, si vous recherchez « Microsoft Surface », une expansion de la requête peut être « Microsoft Surface Pro ».                                   |
| `relatedSearches`  | Requêtes qui ont également été entrées par d’autres utilisateurs qui ont entré la recherche d’origine. Par exemple, si vous recherchez « Mont Rainier », une recherche associée peut être « Mt. Sainte-Hélène ».                       |
| `similarTerms`     | Requêtes dont le sens est similaire à celui de la recherche d’origine. Par exemple, si vous recherchez « chatons », un terme similaire peut être « mignon ».                                                                   |

Cette application affiche uniquement les suggestions `relatedItems` et place les liens qui en résultent dans la barre latérale de la page.

## <a name="rendering-search-results"></a>Rendu des résultats de recherche

Dans cette application, l’objet `searchItemRenderers` contient des fonctions de renderer qui génèrent du code HTML pour chaque type de résultat de recherche.

```javascript
searchItemRenderers = {
    images: function(item, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```

Ces fonctions de renderer acceptent les paramètres suivants :

| Paramètre         | Description                                                                                              |
|---------|----------------------------------------------------------------------------------------------|
| `item`  | L’objet JavaScript contenant les propriétés de l’élément, telles que son URL et sa description. |
| `index` | L’index de l’élément de résultat dans sa collection.                                          |
| `count` | Le nombre d’éléments dans la collection de l’élément du résultat de la recherche.                                  |

Les paramètres `index` et `count` sont utilisés pour compter les résultats, générer du code HTML pour les collections et organiser le contenu. Plus précisément, il :

* Calcule la taille des miniatures (largeur variable, avec un minimum de 120 pixels, mais hauteur fixe de 90 pixels).
* Génère la balise HTML `<img>` pour afficher l’image miniature.
* Génère les balises HTML `<a>` qui établissent un lien vers l’image et la page la contenant.
* Génère la description qui affiche des informations sur l’image et le site sur lequel elle se trouve.

```javascript
    images: function (item, index, count) {
        var height = 120;
        var width = Math.max(Math.round(height * item.thumbnail.width / item.thumbnail.height), 120);
        var html = [];
        if (index === 0) html.push("<p class='images'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<p class='images' style='max-width: " + width + "px'>");
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + "'>");
        html.push("<br>");
        html.push("<nobr><a href='" + item.contentUrl + "'>Image</a> - ");
        html.push("<a href='" + item.hostPageUrl + "'>Page</a></nobr><br>");
        html.push(title.replace("\n", " (").replace(/([a-z0-9])\.([a-z0-9])/g, "$1.<wbr>$2") + ")</p>");
        return html.join("");
    }, // relatedSearches renderer omitted
```

Les valeurs `height` et `width` de l’image miniature sont utilisées à la fois dans la balise `<img>` et dans les champs `h` et `w` de l’URL de la miniature. Cela permet à Bing de retourner [une miniature](../bing-web-search/resize-and-crop-thumbnails.md) d’exactement cette taille.

## <a name="persisting-client-id"></a>ID client persistant

Les réponses provenant des API Recherche Bing peuvent inclure un en-tête `X-MSEdge-ClientID`, qui doit être renvoyé à l’API avec les requêtes suivantes. Si plusieurs API Recherche Bing sont utilisées, le même ID de client doit être employé avec toutes, si possible.

Le fait de fournir l’en-tête `X-MSEdge-ClientID` permet aux API Bing d’associer toutes les recherches d’un utilisateur.

Tout d’abord, cela permet au moteur de recherche Bing d’appliquer un contexte passé aux recherches pour trouver les résultats répondant le mieux à l’utilisateur. Si un utilisateur a précédemment recherché des termes liés à la navigation, par exemple, une recherche ultérieure sur « nœuds » peut éventuellement renvoyer des informations sur les nœuds de navigation.

Par ailleurs, Bing peut sélectionner au hasard des utilisateurs pour leur faire profiter des nouvelles fonctionnalités avant qu’elles ne deviennent disponibles au grand public. Le fait de fournir le même ID de client avec chaque requête garantit que les utilisateurs choisis pour voir une fonctionnalité y aient toujours accès. Sans l’ID client, l’utilisateur peut voir une fonctionnalité apparaître et disparaître, de manière aléatoire, dans les résultats de recherche.

Les stratégies de sécurité de navigateur (CORS) peuvent rendre l’en-tête `X-MSEdge-ClientID` indisponible pour JavaScript. Cette limitation se produit lorsque la réponse de recherche a une origine différente de la page d’où provient la requête. Dans un environnement de production, vous devez gérer cette stratégie en hébergeant un script côté serveur qui effectue l’appel d’API sur le même domaine que la page web. Étant donné que le script a la même origine que la page web, l’en-tête `X-MSEdge-ClientID` est ensuite disponible pour JavaScript.

> [!NOTE]
> Dans une application web de production, vous devez effectuer la requête côté serveur malgré tout. Dans le cas contraire, votre clé API Recherche Bing doit être incluse dans la page web, où elle est accessible à toute personne qui consulte la source. Vous êtes facturé pour toutes les utilisations associées à votre clé d’abonnement d’API, y compris les requêtes effectuées par des tiers non autorisés. Il est donc important de ne pas exposer votre clé.

À des fins de développement, vous pouvez effectuer la requête d’API Recherche Web Bing via un proxy CORS. La réponse provenant d’un proxy de ce type a un en-tête `Access-Control-Expose-Headers` qui autorise les en-têtes de réponse et les rend accessibles à JavaScript.

Il est facile d’installer un proxy CORS pour autoriser l’application du didacticiel à accéder à l’en-tête d’ID client. Tout d’abord, [installez Node.js](https://nodejs.org/en/download/) si ce n’est pas déjà fait. Exécutez alors la commande suivante dans une fenêtre de commande :

```console
npm install -g cors-proxy-server
```

Ensuite, remplacez le point de terminaison Recherche Web Bing dans le fichier HTML par : \
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search`

Enfin, lancez le proxy CORS avec la commande suivante :

```console
cors-proxy-server
```

Laissez la fenêtre de commande ouverte pendant que vous utilisez l’application du tutoriel ; si vous fermez la fenêtre, le proxy s’arrête. Dans la section des en-têtes HTTP (qui peut être développée) sous les résultats de la recherche, vous pouvez maintenant voir l’en-tête `X-MSEdge-ClientID` (entre autres) et vérifier qu’il est identique pour toutes les requêtes.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Extraire les détails de l’image avec l’API Recherche d’images Bing](tutorial-image-post.md)

## <a name="see-also"></a>Voir aussi

* [Informations de référence sur l’API Recherche d’images Bing](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)