---
title: 'Tutoriel : Recherche d’entités Bing dans une application web monopage'
titleSuffix: Azure Cognitive Services
description: Ce tutoriel montre comment utiliser l’API Recherche d’entités Bing dans une application web monopage.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: f725a4095103a7dcfc3dcdbdcefdc84d16501632
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94366531"
---
# <a name="tutorial-single-page-web-app"></a>Tutoriel : Application web à page unique

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020** , toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’entités Bing vous permet de rechercher sur le web des informations sur les *entités* et les *lieux.* Vous pouvez demander l’un ou l’autre type de résultat, ou les deux, dans une requête donnée. Les définitions des lieux et des entités sont fournies ci-dessous.

| Résultats | Description |
|-|-|
|Entités|Personnes, lieux et éléments connus que vous recherchez par nom|
|Lieux|Restaurants, hôtels et autres entreprises locales que vous recherchez par nom *ou* par type (restaurants italiens)|

Dans ce didacticiel, nous allons générer une application web à page unique qui utilise l’API Recherche d’entités Bing pour afficher les résultats de la recherche directement sur la page. L’application inclut des composants HTML, CSS et JavaScript.

L’API vous permet de hiérarchiser les résultats par emplacement. Dans une application mobile, vous pouvez demander à l’appareil son emplacement. Dans une application web, vous pouvez utiliser la fonction `getPosition()`. Mais cet appel fonctionne uniquement dans des contextes sécurisés et ne fournit pas toujours un emplacement précis. L’utilisateur peut également rechercher des entités près d’un autre emplacement que le sien.

Notre application appelle donc le service Bing Maps pour obtenir la latitude et la longitude à partir d’un emplacement saisi par l’utilisateur. L’utilisateur peut ensuite saisir le nom d’un repère (« Space Needle ») ou une adresse complète ou partielle (« New York City ») et l’API Bing Maps fournit les coordonnées.

<!-- Remove until we can replace with a sanitized version.
![[Single-page Bing Entity Search app]](media/entity-search-spa-demo.png)
-->

> [!NOTE]
> Les en-têtes HTTP et JSON en bas de la page révèlent la réponse JSON et les informations de requête HTTP lorsque vous cliquez dessus. Ces détails sont utiles lorsque vous explorez le service.

L’application du didacticiel illustre les actions suivantes :

> [!div class="checklist"]
> * Effectuer un appel d’API Recherche d’entités Bing dans JavaScript
> * Effectuer un appel d’API `locationQuery` Bing Maps dans JavaScript
> * Transmettre des options de recherche aux appels d’API
> * Afficher les résultats de la recherche
> * Gérer l’ID de client Bing et les clés d’abonnement d’API
> * Gérer les erreurs pouvant se produire

La page du didacticiel est entièrement autonome ; elle n’utilise pas d’infrastructures, de feuilles de style ni même de fichiers image externes. Elle a uniquement recours à des fonctionnalités de langage JavaScript largement prises en charge et fonctionne avec les versions actuelles des principaux navigateurs web.

Dans ce didacticiel, nous abordons seulement certaines parties du code source. Le code source complet est disponible [sur une page distincte](). Copiez et collez ce code dans un éditeur de texte, puis enregistrez-le en tant que `bing.html`.

> [!NOTE]
> Ce didacticiel est très similaire au [didacticiel sur l’application de recherche Web Bing à page unique](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md), mais traite uniquement des résultats de recherche d’entités.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous avez besoin de clés d’abonnement pour les API Recherche Bing et Bing Cartes. 

* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure :
  * <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="Créer une ressource Recherche Bing"  target="_blank">Créez une ressource Recherche Bing<span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
  * <a href="https://www.microsoft.com/maps/create-a-bing-maps-key.aspx"  title="Créer une ressource Vision par ordinateur"  target="_blank">Créez une ressource Bing Cartes<span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.

## <a name="app-components"></a>Composants de l’application

Comme n’importe quelle application web à page unique, l’application du didacticiel comprend trois parties :

> [!div class="checklist"]
> * HTML : définit la structure et le contenu de la page
> * CSS : définit l’apparence de la page
> * JavaScript : définit le comportement de la page

Ce didacticiel n’aborde pas la plupart du code HTML ou CSS en détail, car il est simple.

Le code HTML contient le formulaire de recherche dans lequel l’utilisateur saisit une requête et choisit les options de recherche. Le formulaire est connecté à JavaScript, qui effectue la recherche selon la balise `onsubmit` de l’attribut `<form>` :

```html
<form name="bing" onsubmit="return newBingEntitySearch(this)">
```

Le gestionnaire `onsubmit` renvoie `false`, ce qui empêche l’envoi du formulaire vers un serveur. Le code JavaScript effectue réellement le travail de collecte des informations nécessaires à partir du formulaire et exécute la recherche.

La recherche se déroule en deux phases. D’abord, si l’utilisateur a saisi une restriction d’emplacement, une requête Bing Maps est exécutée pour la convertir en coordonnées. Le rappel de cette requête déclenche alors la requête de recherche d’entités Bing.

Le code HTML contient également les divisions (balises `<div>` HTML) où les résultats de recherche s’affichent.

## <a name="managing-subscription-keys"></a>Gestion des clés d’abonnement

> [!NOTE]
> Cette application nécessite des clés d’abonnement pour l’API Recherche Bing et l’API Bing Maps.

Pour éviter d’avoir à inclure les clés d’abonnement des API Recherche Bing et Bing Maps dans le code, nous utilisons le stockage persistant du navigateur pour les stocker. Si l’une des clés n’a pas été stockée, nous la demandons et la stockons pour une utilisation ultérieure. Si la clé est par la suite rejetée par l’API, nous invalidons la clé stockée et en demandons une autre à l’utilisateur lors de sa recherche suivante.

Nous définissons les fonctions `storeValue` et `retrieveValue` qui utilisent l’objet `localStorage` (si le navigateur le prend en charge) ou un cookie. Notre fonction `getSubscriptionKey()` utilise ces fonctions pour stocker et récupérer la clé de l’utilisateur. Vous pouvez utiliser le point de terminaison global ci-dessous, ou le point de terminaison de [sous-domaine personnalisé](../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

```javascript
// cookie names for data we store
SEARCH_API_KEY_COOKIE = "bing-search-api-key";
MAPS_API_KEY_COOKIE   = "bing-maps-api-key";
CLIENT_ID_COOKIE      = "bing-search-client-id";

// API endpoints
SEARCH_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/entities";
MAPS_ENDPOINT   = "https://dev.virtualearth.net/REST/v1/Locations";

// ... omitted definitions of storeValue() and retrieveValue()

// get stored API subscription key, or prompt if it's not found
function getSubscriptionKey(cookie_name, key_length, api_name) {
    var key = retrieveValue(cookie_name);
    while (key.length !== key_length) {
        key = prompt("Enter " + api_name + " API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(cookie_name, key);
    return key;
}

function getMapsSubscriptionKey() {
    return getSubscriptionKey(MAPS_API_KEY_COOKIE, 64, "Bing Maps");
}

function getSearchSubscriptionKey() {
    return getSubscriptionKey(SEARCH_API_KEY_COOKIE, 32, "Bing Search");
}
```

La balise `<body>` HTML inclut un attribut `onload` qui appelle `getSearchSubscriptionKey()` et `getMapsSubscriptionKey()` une fois la page entièrement chargée. Ces appels permettent de demander immédiatement leurs clés aux utilisateurs s’ils ne les ont pas encore saisies.

```html
<body onload="document.forms.bing.query.focus(); getSearchSubscriptionKey(); getMapsSubscriptionKey();">
```

## <a name="selecting-search-options"></a>Sélection des options de recherche

![[Formulaire de recherche d’entités Bing]](media/entity-search-spa-form.png)

Le formulaire HTML comprend les contrôles suivants :

| Control | Description |
|-|-|
|`where`|Menu déroulant pour sélectionner le marché (lieu et langue) utilisé pour la recherche.|
|`query`|Champ de texte dans lequel saisir les termes de recherche.|
|`safe`|Case à cocher indiquant si l’option Filtre adulte est activée (limite les résultats « adultes »)|
|`what`|Menu permettant de choisir de rechercher des entités, des lieux ou les deux.|
|`mapquery`|Champ de texte dans lequel l’utilisateur peut saisir une adresse complète ou partielle, un repère, etc. pour que Recherche d’entités Bing renvoie des résultats plus pertinents.|

> [!NOTE]
> Les résultats des lieux sont actuellement disponibles uniquement aux États-Unis. Les menus `where` et `what` comportent un code pour appliquer cette restriction. Si vous choisissez un marché non américain alors que Lieux est sélectionné dans le menu `what`, `what` est remplacé par Tout. Si vous choisissez Lieux alors qu’un marché non américain est sélectionné dans le menu `where`, `where` est remplacé par États-Unis.

La fonction JavaScript `bingSearchOptions()` convertit ces champs en chaîne de requête partielle pour l’API Recherche Bing.

```javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.what.selectedIndex) options.push("responseFilter=" + form.what.value);
    return options.join("&");
}
```

Par exemple, la fonctionnalité Filtre adulte peut être `strict`, `moderate` ou `off`. `moderate` est la valeur par défaut. Cependant, notre formulaire utilise une case à cocher qui a seulement deux états. Le code JavaScript convertit ce paramètre en la valeur `strict` ou `off` (nous n’utilisons pas `moderate`).

Le champ `mapquery` n’est pas géré dans `bingSearchOptions()`, car il est utilisé pour la requête d’emplacement Bing Maps, pas pour la Recherche d’entités Bing.

## <a name="obtaining-a-location"></a>Obtention d’un emplacement

L’API Bing Maps offre une [`locationQuery`méthode](//msdn.microsoft.com/library/ff701711.aspx), qui nous permet de trouver la latitude et longitude de l’emplacement saisi par l’utilisateur. Ces coordonnées sont ensuite transmises à l’API Recherche d’entités Bing avec la requête de l’utilisateur. Les résultats de la recherche hiérarchisent les entités et les lieux à proximité de l’emplacement spécifié.

Nous ne pouvons pas accéder à l’API Bing Maps en utilisant une requête `XMLHttpRequest` ordinaire dans une application web, car le service ne prend pas en charge les requêtes cross-origin. Heureusement, il prend en charge JSONP (« P » pour « padded » qui signifie « rempli »). Une réponse JSONP est une réponse JSON ordinaire encapsulée dans un appel de fonction. La requête est effectuée en insérant une balise `<script>` dans le document. (Le chargement des scripts n’est pas soumis aux stratégies de sécurité de navigateur.)

La fonction `bingMapsLocate()` crée et insère la balise `<script>` pour la requête. Le segment `jsonp=bingMapsCallback` de la chaîne de requête spécifie le nom de la fonction qui soit être appelée avec la réponse.

```javascript
function bingMapsLocate(where) {

    where = where.trim();
    var url = MAPS_ENDPOINT + "?q=" + encodeURIComponent(where) + 
                "&jsonp=bingMapsCallback&maxResults=1&key=" + getMapsSubscriptionKey();

    var script = document.getElementById("bingMapsResult")
    if (script) script.parentElement.removeChild(script);

    // global variable holds reference to timer that will complete the search if the maps query fails
    timer = setTimeout(function() {
        timer = null;
        var form = document.forms.bing;
        bingEntitySearch(form.query.value, "", bingSearchOptions(form), getSearchSubscriptionKey());
    }, 5000);

    script = document.createElement("script");
    script.setAttribute("type", "text/javascript");
    script.setAttribute("id", "bingMapsResult");
    script.setAttribute("src", url);
    script.setAttribute("onerror", "BingMapsCallback(null)");
    document.body.appendChild(script);

    return false;
}
```

> [!NOTE]
> Si l’API Bing Maps ne répond pas, la fonction `bingMapsCallBack()` n’est jamais appelée. En règle générale, cela signifie que `bingEntitySearch()` n’est pas appelée et que les résultats de la recherche d’entités n’apparaissent pas. Pour éviter ce scénario, `bingMapsLocate()` définit également un minuteur afin d’appeler `bingEntitySearch()` après cinq secondes. La logique de la fonction de rappel permet d’éviter d’effectuer la recherche d’entité à deux reprises.

Lorsque la requête est terminée, la fonction `bingMapsCallback()` est appelée, comme demandé.

```javascript
function bingMapsCallback(response) {

    if (timer) {    // we beat the timer; stop it from firing
        clearTimeout(timer);
        timer = null;
    } else {        // the timer beat us; don't do anything
        return; 
    }

    var location = "";
    var name = "";
    var radius = 1000;

    if (response) {
        try {
            if (response.statusCode === 401) {
                invalidateMapsKey();
            } else if (response.statusCode === 200) {
                var resource = response.resourceSets[0].resources[0];
                var coords   = resource.point.coordinates;
                name         = resource.name;

                // the radius is the largest of the distances between the location and the corners
                // of its bounding box (in case it's not in the center) with a minimum of 1 km
                try {
                    var bbox    = resource.bbox;
                    radius  = Math.max(haversineDistance(bbox[0], bbox[1], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[1]),
                                       haversineDistance(bbox[0], bbox[3], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[3]), 1000);
                } catch(e) {  }
                var location = "lat:" + coords[0] + ";long:" + coords[1] + ";re:" + Math.round(radius);
            }
        }
        catch (e) { }   // response is unexpected. this isn't fatal, so just don't provide location
    }

    var form = document.forms.bing;
    if (name) form.mapquery.value = name;
    bingEntitySearch(form.query.value, location, bingSearchOptions(form), getSearchSubscriptionKey());

}
```

Outre la latitude et la longitude, la requête Recherche d’entités Bing requiert un *rayon* qui indique la précision des informations d’emplacement. Nous calculons le rayon à l’aide du *rectangle englobant* fourni dans la réponse Bing Maps. Le rectangle englobant entoure l’emplacement complet. Par exemple, si l’utilisateur saisit `NYC`, le résultat contient les coordonnées approximatives de New York City et un rectangle englobant qui entoure la ville. 

Nous calculons d’abord les distances entre les coordonnées principales et chaque coin du rectangle englobant à l’aide de la fonction `haversineDistance()` (non affichée). Nous utilisons la plus grande de ces quatre distances comme rayon. Le rayon minimal est un kilomètre. Cette valeur est également utilisée comme valeur par défaut si aucun rectangle englobant n’est fourni dans la réponse.

Après avoir obtenu les coordonnées et le rayon, nous appelons `bingEntitySearch()` pour exécuter la recherche.

## <a name="performing-the-search"></a>Exécution de la recherche

En fonction de la requête, de l’emplacement, de la chaîne d’options et de la clé de l’API, la fonction `BingEntitySearch()` exécute la requête de Recherche d'entités Bing.

```javascript
// perform a search given query, location, options string, and API keys
function bingEntitySearch(query, latlong, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "error");

    var request = new XMLHttpRequest();
    var queryurl = SEARCH_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

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

    if (latlong) request.setRequestHeader("X-Search-Location", latlong);

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

En cas de réussite de la requête HTTP, JavaScript appelle notre gestionnaire d’événements `load`, la fonction `handleBingResponse()`, pour gérer une requête HTTP GET réussie auprès de l’API. 

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
            if (jsobj._type === "SearchResponse") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    if (divHidden("pole") && divHidden("mainline") && divHidden("sidebar")) 
        showDiv("noresults", "No results.<p><small>Looking for restaurants or other local businesses? Those currently areen't supported outside the US.</small>");
    }

    // Any other HTTP status is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSearchKey();

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
> Une requête HTTP réussie ne signifie *pas* nécessairement que la recherche a réussi. Si une erreur se produit dans l’opération de recherche, l’API Recherche d’entités Bing renvoie un code d’état HTTP autre que 200 et inclut des informations sur l’erreur dans la réponse JSON. En outre, si la requête avait un débit limité, l’API renvoie une réponse vide.

Une grande partie du code dans les deux fonctions précédentes est dédiée à la gestion des erreurs. Des erreurs peuvent se produire aux étapes suivantes :

|Étape|Erreurs potentielles|Gérées par|
|-|-|-|
|Génération de l’objet de requête JavaScript|URL non valide|Bloc `try`/`catch`|
|Construction de la requête|Erreurs réseau, connexions abandonnées|Gestionnaires d’événements `error` et `abort`|
|Exécution de la recherche|Requête non valide, JSON non valide, limites de débit|Tests dans le gestionnaire d’événements `load`|

Les erreurs sont gérées en appelant `renderErrorMessage()` avec des détails connus de l’erreur. Si la réponse transmet tous les tests d’erreurs, nous appelons `renderSearchResults()` pour afficher les résultats de la recherche sur la page.

## <a name="displaying-search-results"></a>Affichage des résultats de la recherche

L’API Recherche d’entités Bing [exige d’afficher les résultats dans un ordre spécifié](../bing-web-search/use-display-requirements.md). Comme l’API peut retourner deux types de réponses différents, il n’est pas suffisant de procéder à une itération au sein de la collection de plus haut niveau `Entities` ou `Places` dans la réponse JSON et d’afficher les résultats. (Si vous ne souhaitez qu’un seul type de résultat, utilisez le paramètre de requête `responseFilter`.)

Nous utilisons plutôt la collection `rankingResponse` dans les résultats de la recherche pour classer l’affichage. Cet objet fait référence à des éléments dans les collections `Entitiess` et/ou `Places`.

`rankingResponse` peut contenir jusqu'à trois collections des résultats de recherche, désignés par `pole`, `mainline` et `sidebar`. 

`pole`, le cas échéant, est le résultat de recherche le plus pertinent et doit être affiché en premier. `mainline` fait référence à l’ensemble des résultats de la recherche. Les résultats principaux doivent apparaître immédiatement après `pole` (ou en premier, si `pole` n’est pas présent). 

Enfin, `sidebar` fait référence aux résultats de recherche auxiliaires. Ils peuvent s’afficher dans une barre latérale ou simplement après les résultats principaux. Pour notre application du didacticiel, nous avons choisi cette dernière méthode.

Chaque élément d’une collection `rankingResponse` fait référence à des éléments de résultats de recherche réels de deux manières différentes mais équivalentes.

| Élément | Description |
|-|-|
|`id`|L’`id` ressemble à une URL, mais ne doit pas être utilisé pour les liens. Le type `id` d’un résultat de classement correspond à l’`id` d’un élément résultat de recherche dans une collection de réponses, *ou* une collection de réponses entière (telle que `Entities`).
|`answerType`<br>`resultIndex`|`answerType` fait référence à la collection de réponses de niveau supérieur qui contient le résultat (par exemple, `Entities`). `resultIndex` fait référence à l’index du résultat dans cette collection. Si `resultIndex` est omis, le résultat de classement fait référence à toute la collection.

> [!NOTE]
> Pour plus d’informations sur cette partie de la réponse de recherche, consultez la rubrique que le [classement des résultats](rank-results.md).

Vous pouvez utiliser la méthode la plus pratique dans votre application pour localiser l’élément de résultat de recherche référencé. Dans le code du didacticiel, nous utilisons `answerType` et `resultIndex` pour localiser chaque résultat de recherche.

Enfin, il est temps d’examiner notre fonction `renderSearchResults()`. Cette fonction effectue une itération sur les trois collections `rankingResponse` qui représentent les trois sections des résultats de recherche. Pour chaque section, nous appelons `renderResultsItems()` pour afficher les résultats correspondants.

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // if spelling was corrected, update search field
    if (results.queryContext.alteredQuery) 
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // for each possible section, render the results from that section
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

## <a name="rendering-result-items"></a>Rendu des éléments de résultat

Notre code JavaScript inclut un objet, `searchItemRenderers`, qui contient des fonctions de *renderer* générant du code HTML pour chaque type de résultat de recherche.

```javascript
searchItemRenderers = { 
    entities: function(item) { ... },
    places: function(item) { ... }
}
```

Une fonction de renderer peut accepter les paramètres suivants :

| Paramètre | Description |
|-|-|
|`item`|L’objet JavaScript contenant les propriétés de l’élément, telles que son URL et sa description.|
|`index`|L’index de l’élément de résultat dans sa collection.|
|`count`|Le nombre d’éléments dans la collection de l’élément du résultat de la recherche.|

Les paramètres `index` et `count` peuvent être utilisés pour compter les résultats, pour générer un code HTML spécial pour le début ou la fin d’une collection, pour insérer des sauts de ligne après un certain nombre d’éléments et ainsi de suite. Si un renderer n’a pas besoin de cette fonctionnalité, il est inutile d’accepter ces deux paramètres. En fait, nous ne les utilisons pas dans les renderers de notre application du didacticiel.

Examinons de plus près le renderer `entities` :

```javascript
    entities: function(item) {
        var html = [];
        html.push("<p class='entity'>");
        if (item.image) {
            var img = item.image;
            if (img.hostPageUrl) html.push("<a href='" + img.hostPageUrl + "'>");
            html.push("<img src='" + img.thumbnailUrl +  "' title='" + img.name + "' height=" + img.height + " width= " + img.width + ">");
            if (img.hostPageUrl) html.push("</a>");
            if (img.provider) {
                var provider = img.provider[0];
                html.push("<small>Image from ");
                if (provider.url) html.push("<a href='" + provider.url + "'>");
                html.push(provider.name ? provider.name : getHost(provider.url));
                if (provider.url) html.push("</a>");
                html.push("</small>");
            }
        }
        html.push("<p>");
        if (item.entityPresentationInfo) {
            var pi = item.entityPresentationInfo;
            if (pi.entityTypeHints || pi.entityTypeDisplayHint) {
                html.push("<i>");
                if (pi.entityTypeDisplayHint) html.push(pi.entityTypeDisplayHint);
                else if (pi.entityTypeHints) html.push(pi.entityTypeHints.join("/"));
                html.push("</i> - ");
            }
        }
        html.push(item.description);
        if (item.webSearchUrl) html.push("&nbsp;<a href='" + item.webSearchUrl + "'>More</a>")
        if (item.contractualRules) {
            html.push("<p><small>");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++) {
                var rule = item.contractualRules[i];
                var link = [];
                if (rule.license) rule = rule.license;
                if (rule.url) link.push("<a href='" + rule.url + "'>");
                link.push(rule.name || rule.text || rule.targetPropertyName + " source");
                if (rule.url) link.push("</a>");
                rules.push(link.join(""));
            }
            html.push("License: " + rules.join(" - "));
            html.push("</small>");
        }
        return html.join("");
    }, // places renderer omitted
```

Notre fonction de renderer d’entité :

> [!div class="checklist"]
> * génère la balise HTML `<img>` pour afficher la miniature de l’image, le cas échéant. 
> * génère la balise HTML `<a>` qui établit un lien vers la page contenant l’image.
> * Génère la description qui affiche des informations sur l’image et le site sur lequel elle se trouve.
> * Incorpore la classification de l’entité à l’aide d’indicateurs d’affichage, le cas échéant.
> * Inclut un lien vers une recherche Bing pour obtenir plus d’informations sur l’entité.
> * Affiche les informations de licence ou d’attribution requises par les sources de données.

## <a name="persisting-client-id"></a>ID client persistant

Les réponses provenant des API Recherche Bing peuvent inclure un en-tête `X-MSEdge-ClientID`, qui doit être renvoyé à l’API avec les requêtes suivantes. Si plusieurs API Recherche Bing sont utilisées, le même ID de client doit être employé avec toutes, si possible.

Le fait de fournir l’en-tête `X-MSEdge-ClientID` permet aux API Bing d’associer toutes les recherches d’un utilisateur, ce qui présente deux avantages importants.

Tout d’abord, cela permet au moteur de recherche Bing d’appliquer un contexte passé aux recherches pour trouver les résultats répondant le mieux à l’utilisateur. Si un utilisateur a précédemment recherché des termes liés à la navigation, par exemple, une recherche ultérieure sur « quais » peut éventuellement renvoyer des informations sur les lieux où mettre un navire à quai.

Par ailleurs, Bing peut sélectionner au hasard des utilisateurs pour leur faire profiter des nouvelles fonctionnalités avant qu’elles ne deviennent disponibles au grand public. Le fait de fournir le même ID de client avec chaque requête garantit que les utilisateurs choisis pour voir une fonctionnalité y aient toujours accès. Sans l’ID client, l’utilisateur peut voir une fonctionnalité apparaître et disparaître, de manière aléatoire, dans les résultats de recherche.

Les stratégies de sécurité de navigateur (CORS) peuvent rendre l’en-tête `X-MSEdge-ClientID` indisponible pour JavaScript. Cette limitation se produit lorsque la réponse de recherche a une origine différente de la page d’où provient la requête. Dans un environnement de production, vous devez gérer cette stratégie en hébergeant un script côté serveur qui effectue l’appel d’API sur le même domaine que la page web. Étant donné que le script a la même origine que la page web, l’en-tête `X-MSEdge-ClientID` est ensuite disponible pour JavaScript.

> [!NOTE]
> Dans une application web de production, vous devez effectuer la requête côté serveur malgré tout. Dans le cas contraire, votre clé API Recherche Bing doit être incluse dans la page web, où elle est accessible à toute personne qui consulte la source. Vous êtes facturé pour toutes les utilisations associées à votre clé d’abonnement d’API, y compris les requêtes effectuées par des tiers non autorisés. Il est donc important de ne pas exposer votre clé.

À des fins de développement, vous pouvez effectuer la requête d’API Recherche Web Bing via un proxy CORS. La réponse provenant d’un tel proxy a un en-tête `Access-Control-Expose-Headers` qui autorise les en-têtes de réponse de listes et les rend accessibles à JavaScript.

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
> [Référence à l’API Recherche d’entités Bing](//docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference)

> [!div class="nextstepaction"]
> [Documentation sur l’API Bing Maps](//msdn.microsoft.com/library/dd877180.aspx)