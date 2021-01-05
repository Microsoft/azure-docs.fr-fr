---
title: 'Tutoriel : Générer une application monopage pour la recherche de vidéos Bing'
titleSuffix: Azure Cognitive Services
description: Ce tutoriel explique comment utiliser l’API Recherche de vidéos Bing dans une application web monopage.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: tutorial
ms.date: 02/03/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: 76d7f4e31934480be57cd5ac133c1c7e6a9dd364
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349603"
---
# <a name="tutorial-single-page-video-search-app"></a>Tutoriel : Application monopage pour la recherche de vidéos

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
L’API Recherche de vidéos Bing vous permet de parcourir le web et d’obtenir des résultats de vidéos relatifs à une requête de recherche. Dans ce didacticiel, nous allons générer une application web à page unique qui utilise l’API Recherche de vidéos Bing pour afficher les résultats de la recherche sur la page. L’application inclut des composants HTML, CSS et JavaScript.

<!-- Remove until it can be replaced with a sanitized version.
![Single-page Bing Video Search app](./media/video-search-singlepage.png)
-->

> [!NOTE]
> Lorsque vous cliquez dessus, les en-têtes HTTP et JSON en bas de la page affichent la réponse JSON et les informations de requête HTTP. Ces détails peuvent être utiles lorsque vous explorez le service.

![JSON, résultats bruts HTTP](./media/json-http-raw-results.png)

L’application de ce didacticiel illustre les actions suivantes :
> [!div class="checklist"]
> * Effectuer un appel d’API Recherche de vidéos Bing dans JavaScript
> * Transmettre des options de recherche à l’API Recherche Bing
> * Afficher les résultats de la recherche de vidéos ou inclure éventuellement des pages web, des actualités ou des images
> * Rechercher sur des périodes de 24 heures, semaine dernière, mois ou tout le temps disponible
> * Parcourir les résultats de la recherche
> * Gérer l’ID de client Bing et la clé d’abonnement d’API
> * Gérer les erreurs qui peuvent se produire

La page du didacticiel est entièrement autonome. Elle n’utilise pas d’infrastructures, de feuilles de style ni de fichiers image externes. Elle a uniquement recours à des fonctionnalités de langage JavaScript largement prises en charge et fonctionne avec les versions actuelles des principaux navigateurs web.

Dans ce didacticiel, nous abordons certaines parties du code source. Le [code source]() complet est disponible. Pour exécuter l’exemple, copiez et collez le code source dans un éditeur de texte et enregistrez le fichier sous `bing.html`.

## <a name="app-components"></a>Composants de l’application
Comme n’importe quelle application web à page unique, cette application du didacticiel comprend trois parties :

> [!div class="checklist"]
> * HTML : définit la structure et le contenu de la page
> * CSS : définit l’apparence de la page
> * JavaScript : définit le comportement de la page

La plupart du code HTML et CSS étant classique, le didacticiel ne l’explique pas. Le code HTML contient le formulaire de recherche dans lequel l’utilisateur saisit une requête et choisit les options de recherche. Le formulaire est connecté à JavaScript qui effectue la recherche selon l’attribut `onsubmit` de la balise `<form>` :

```html
<form name="bing" onsubmit="return bingWebSearch(this)">
```
Le gestionnaire `onsubmit` renvoie `false`, ce qui empêche l’envoi du formulaire vers un serveur. Le code JavaScript effectue le travail de collecte des informations nécessaires à partir du formulaire et exécute la recherche.

Le code HTML contient également les divisions (balises `<div>` HTML) où les résultats de recherche s’affichent.

## <a name="managing-subscription-key"></a>Gestion de la clé d’abonnement

Pour éviter d’avoir à inclure la clé d’abonnement de l’API Recherche Bing dans le code, nous utilisons le stockage persistant du navigateur pour stocker la clé. Avant de stocker la clé, nous demandons la clé de l’utilisateur. Si la clé est ensuite rejetée par l’API, nous invalidons la clé stockée et en demandons une autre à l’utilisateur.

Nous définissons les fonctions `storeValue` et `retrieveValue` qui utilisent l’objet `localStorage` (tous les navigateurs ne le prennent pas en charge) ou un cookie. La fonction `getSubscriptionKey()` utilise ces fonctions pour stocker et récupérer la clé de l’utilisateur.

``` javascript
// Cookie names for data we store
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

// ... omitted definitions of store value and retrieve value
// Browsers differ in their support for persistent storage by 
// local HTML files. See the source code for browser-specific
// options.

// Get stored API subscription key, or prompt if it's not found.
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
La balise `onsubmit` du code HTML `<form>` appelle la fonction `bingWebSearch` pour renvoyer les résultats de la recherche. `bingWebSearch` utilise `getSubscriptionKey()` pour authentifier chaque requête. Comme indiqué dans la définition précédente, `getSubscriptionKey` demande une clé à l’utilisateur si la clé n’a pas été saisie. La clé est ensuite stockée en vue d’une utilisation continue par l’application.

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value, 
    bingSearchOptions(this), getSubscriptionKey())">
```
## <a name="selecting-search-options"></a>Sélection des options de recherche
L’illustration suivante montre la zone de texte de requête et les options qui définissent une recherche.

![Options de recherche d’actualités Bing](media/video-search-options.png)

Le formulaire HTML inclut des éléments avec les noms suivants :

|Élément|Description|
|-|-|
| `where` | Menu déroulant pour sélectionner le marché (lieu et langue) utilisé pour la recherche. |
| `query` | Champ de texte dans lequel saisir les termes de recherche. |
| `modules` | Cases à cocher pour promouvoir des modules particuliers de résultats, tous les résultats ou les vidéos associées. |
| `when` | Menu déroulant pour éventuellement limiter la recherche à un jour, une semaine ou un mois. |
| `safe` | Case à cocher indiquant s’il faut utiliser la fonctionnalité Filtre adulte de Bing afin de filtrer les résultats pour « adultes ». |
| `count` | Champ masqué. Nombre de résultats de recherche à renvoyer pour chaque requête. À modifier pour afficher plus ou moins de résultats par page. |
| `offset`|  Champ masqué. Décalage du premier résultat de recherche dans la requête ; utilisé pour la pagination. Réinitialisé sur `0` lors d’une nouvelle requête. |

> [!NOTE]
> L’API Recherche Web Bing offre d’autres paramètres de requête. Nous n’en utilisons ici que quelques-uns.

``` javascript
// build query options from the HTML form
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "moderate"));

    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++) 
        if (form.what[i].checked) what.push(form.what[i].value);
    if (what.length) {
        options.push("modules=" + what.join(","));
        options.push("answerCount=9");
    }
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    options.push("textDecorations=true");
    options.push("textFormat=HTML");
    return options.join("&");
}
```

Par exemple, le paramètre `SafeSearch` dans un appel d’API réel peut être `strict` ou `moderate`, `moderate` étant la valeur par défaut.

## <a name="performing-the-request"></a>Exécution de la requête
Selon la requête, la chaîne d’options et la clé d’API, la fonction `BingWebSearch` utilise un objet `XMLHttpRequest` pour effectuer la requête au point de terminaison Recherche Bing. Vous pouvez utiliser le point de terminaison global ci-dessous, ou le point de terminaison de [sous-domaine personnalisé](../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

```javascript
// Search on the query, using search options, authenticated by the key.
function bingWebSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_headers", "paging1", "paging2", "error");

    var endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/videos/search";
    var request = new XMLHttpRequest();
    var queryurl = endpoint + "?q=" + encodeURIComponent(query) + "&" + options;

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
    request.addEventListener("load", handleOnLoad);

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
Lorsque la requête HTTP aboutit, JavaScript appelle le gestionnaire d’événements `load`, `handleOnLoad()`, pour gérer une requête HTTP GET réussie auprès de l’API. 

```javascript
// handle Bing search request results
function handleOnLoad() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and headers
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " + 
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "Videos") {//"SearchResponse" && "rankingResponse" in jsobj) {
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
> Si une erreur se produit dans l’opération de recherche, l’API Recherche d’actualités Bing renvoie un code d’état HTTP autre que 200 et inclut des informations sur l’erreur dans la réponse JSON. En outre, si la requête avait un débit limité, l’API renvoie une réponse vide.
Une requête HTTP réussie ne signifie *pas* nécessairement que la recherche a réussi. 

Une grande partie du code dans les deux fonctions précédentes est dédiée à la gestion des erreurs. Des erreurs peuvent se produire aux étapes suivantes :

|Étape|Erreurs potentielles|Gérées par|
|-|-|-|
|Génération de l’objet de requête JavaScript|URL non valide|Bloc `try`/`catch`|
|Construction de la requête|Erreurs réseau, connexions abandonnées|Gestionnaires d’événements `error` et `abort`|
|Exécution de la recherche|Requête non valide, JSON non valide, limites de débit|Tests dans le gestionnaire d’événements `load`|

Les erreurs sont gérées en appelant `renderErrorMessage()` avec des détails connus de l’erreur. Si la réponse transmet tous les tests d’erreurs, nous appelons `renderSearchResults()` pour afficher les résultats de la recherche sur la page.

## <a name="displaying-search-results"></a>Affichage des résultats de la recherche
La principale fonction d’affichage des résultats de la recherche est `renderSearchResults()`. Cette fonction utilise le JSON renvoyé par le service Recherche d’actualités Bing et restitue les résultats d’actualités et les recherches associées, le cas échéant.

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    // Render the results to the mainline section
    for (section in { mainline: 0 }) {
         showDiv(section, renderResultsItems(section, results));
    }
}
```
Les résultats de la recherche sont renvoyés en tant qu’objet `value` de niveau supérieur dans la réponse JSON. Nous les transmettons à notre fonction `renderResultsItems()`, qui effectue une itération au sein de ces résultats et appelle une fonction pour afficher chaque élément en HTML. Le code HTML résultant est renvoyé à `renderSearchResults()`, où il est inséré dans la division `results` de la page.

```javascript
// render search results
    function renderResultsItems(section, results) {   

        var items = results.value;
        var html = [];
        for (var i = 0; i < items.length; i++) { 
            var item = items[i];
            // collection name has lowercase first letter
            var type = "videos";
            var render = searchItemRenderers[type];
            html.push(render(item, section));  
        }
    return html.join("\n\n");
}
```

L’API Recherche d’actualités Bing renvoie jusqu’à quatre types de résultats liés différents, chacun dans son propre objet de niveau supérieur. Il s'agit de :

|Relation|Description|
|-|-|
|`pivotSuggestions`|Requêtes qui remplacent un mot pivot de la recherche d’origine par un autre. Par exemple, si vous recherchez « fleurs rouges », « rouge » peut être un mot pivot et une suggestion peut être « fleurs jaunes ».|
|`queryExpansions`|Requêtes qui affinent votre recherche d’origine en ajoutant d’autres termes. Par exemple, si vous recherchez « Microsoft Surface », une expansion de la requête peut être « Microsoft Surface Pro ».|
|`relatedSearches`|Requêtes qui ont également été entrées par d’autres utilisateurs qui ont entré la recherche d’origine. Par exemple, si vous recherchez « Mont Rainier », une recherche associée peut être « Mt. Sainte-Hélène ».|
|`similarTerms`|Requêtes dont le sens est similaire à celui de la recherche d’origine. Par exemple, si vous recherchez « écoles », un terme similaire peut être « éducation ».|

Comme nous l’avons vu précédemment dans `renderSearchResults()`, nous affichons uniquement les suggestions `relatedItems` et plaçons les liens qui en résultent dans la barre latérale de la page.

## <a name="rendering-result-items"></a>Rendu des éléments de résultat

Le code JavaScript inclut un objet, `searchItemRenderers`, qui peut contenir des fonctions de *renderer* générant du code HTML pour chaque type de résultat de recherche. La page de recherche de vidéos utilise uniquement `videos`. Consultez les autres didacticiels pour les différents types de renderer.

```javascript
searchItemRenderers = {
    news: function(item) { ... },
    webPages: function (item) { ... }, 
    images: function(item, index, count) { ... },
    videos: function (item, section, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```
Une fonction de renderer peut accepter les paramètres suivants :

|Paramètre|Description|
|-|-|
|`item`| L’objet JavaScript contenant les propriétés de l’élément, telles que son URL et sa description.|
|`index`| L’index de l’élément de résultat dans sa collection.|
|`count`| Le nombre d’éléments dans la collection de l’élément du résultat de la recherche.|

Les paramètres `index` et `count` peuvent être utilisés pour compter les résultats, pour générer un code HTML spécial pour le début ou la fin d’une collection, pour insérer des sauts de ligne après un certain nombre d’éléments et ainsi de suite. Si un renderer n’a pas besoin de cette fonctionnalité, il est inutile d’accepter ces deux paramètres.

Le renderer `video` est indiqué dans l’extrait de code JavaScript suivant. À l’aide du point de terminaison Vidéos, tous les résultats sont de type `Videos`. Les `searchItemRenderers` sont présentés dans le segment de code suivant.

```javascript
// render functions for various types of search results
    searchItemRenderers = {

    videos: function (item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];

        html.push("<p class='images'>");
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escapeQuotes(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='" + item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        html.push("<br>");
        html.push("<nobr><a href='" + item.contentUrl + "'>Video page source</a> - ");
        html.push(title.replace("\n", " (").replace(/([a-z0-9])\.([a-z0-9])/g, "$1.<wbr>$2") + ")</p>");
        return html.join("");
    }
}
```

La fonction de renderer :
> [!div class="checklist"]
> * Crée une balise de paragraphe, l’attribue à la classe `images`, puis la place dans le tableau html.
> * Calcule la taille de l’image miniature (la largeur est fixée à 60 pixels, la hauteur est calculée proportionnellement).
> * Génère la balise HTML `<img>` pour afficher l’image miniature. 
> * Génère les balises HTML `<a>` qui établissent un lien vers l’image et la page la contenant.
> * Génère la description qui affiche des informations sur l’image et le site sur lequel elle se trouve.

La taille des miniatures est utilisée à la fois dans la balise `<img>` et dans les champs `h` et `w` des URL de miniature. Bing retourne [une miniature](../bing-web-search/resize-and-crop-thumbnails.md) de cette taille précise.

## <a name="persisting-client-id"></a>ID client persistant
Les réponses provenant des API Recherche Bing peuvent inclure un en-tête `X-MSEdge-ClientID`, qui doit être renvoyé à l’API avec les requêtes suivantes. Si plusieurs API Recherche Bing sont utilisées, le même ID de client doit être employé avec toutes, si possible.

Le fait de fournir l’en-tête `X-MSEdge-ClientID` permet aux API Bing d’associer toutes les recherches d’un utilisateur, ce qui présente deux avantages essentiels.

Tout d’abord, cela permet au moteur de recherche Bing d’appliquer un contexte passé aux recherches pour trouver les résultats répondant le mieux à l’utilisateur. Si un utilisateur a précédemment recherché des termes liés à la navigation, par exemple, une recherche ultérieure sur « nœuds » peut éventuellement renvoyer des informations sur les nœuds de navigation.

Par ailleurs, Bing peut sélectionner au hasard des utilisateurs pour leur faire profiter des nouvelles fonctionnalités avant qu’elles ne deviennent disponibles au grand public. Le fait de fournir le même ID de client avec chaque requête garantit que les utilisateurs qui voient une fonctionnalité y aient toujours accès. Sans l’ID client, l’utilisateur peut voir une fonctionnalité apparaître et disparaître, de manière aléatoire, dans les résultats de recherche.

Les stratégies de sécurité de navigateur (CORS) peuvent rendre l’en-tête `X-MSEdge-ClientID` indisponible pour JavaScript. Cette limitation se produit lorsque la réponse de recherche a une origine différente de la page d’où provient la requête. Dans un environnement de production, vous devez gérer cette stratégie en hébergeant un script côté serveur qui effectue l’appel d’API sur le même domaine que la page web. Étant donné que le script a la même origine que la page web, l’en-tête `X-MSEdge-ClientID` est ensuite disponible pour JavaScript.

> [!NOTE]
> Dans une application web de production, vous devez effectuer la requête côté serveur. Dans le cas contraire, votre clé API Recherche Bing doit être incluse dans la page web, où elle est accessible à toute personne qui consulte la source. Vous êtes facturé pour toutes les utilisations associées à votre clé d’abonnement d’API, y compris les requêtes effectuées par des tiers non autorisés. Il est donc important de ne pas exposer votre clé.

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
> [Informations de référence sur l’API Recherche de vidéos Bing](//docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference)