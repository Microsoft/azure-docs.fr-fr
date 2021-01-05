---
title: 'Tutoriel : Créer une application web monopage - API Recherche Web Bing'
titleSuffix: Azure Cognitive Services
description: Cette application monopage montre comment l’API Recherche Web Bing peut être utilisée pour récupérer, analyser et afficher des résultats de recherche pertinents dans une application monopage.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: ff54d605fd81fa640314d99359f1aabacf7a469e
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350379"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-web-search-api"></a>Tutoriel : Créer une application web monopage à l’aide de l’API Recherche Web Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Cette application monopage montre comment récupérer, analyser et afficher des résultats de recherche à partir de l’API Recherche Web Bing. Le tutoriel utilise du code HTML et CSS réutilisable, et se concentre sur le code JavaScript. Les fichiers HTML, CSS et JS sont disponibles sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Web-Search) avec des instructions de démarrage rapide.

Cet exemple d’application peut :

> [!div class="checklist"]
> * Appeler l’API Recherche Web Bing avec des options de recherche
> * Afficher des résultats de recherche web, d’images, d’actualités et de vidéos
> * Paginer les résultats
> * Gérer les clés d’abonnement
> * des erreurs

Pour utiliser cette application, vous devrez disposer d’un [compte Azure Cognitive Services](../cognitive-services-apis-create-account.md) avec des API Recherche Bing.

## <a name="prerequisites"></a>Prérequis

Voici quelques éléments dont vous aurez besoin pour exécuter l’application :

* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="Créer une ressource Recherche Bing"  target="_blank">créez une ressource Recherche Bing<span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.

* Node.js 8 ou version ultérieure

La première étape consiste à cloner le dépôt avec le code source de l’exemple d’application.

```console
git clone https://github.com/Azure-Samples/cognitive-services-REST-api-samples.git
```

Exécutez ensuite `npm install`. Pour ce tutoriel, Express.js est la seule dépendance.

```console
cd <path-to-repo>/cognitive-services-REST-api-samples/Tutorials/Bing-Web-Search
npm install
```

## <a name="app-components"></a>Composants de l’application

L’exemple d’application que nous générons se compose de quatre parties :

* `bing-web-search.js` : notre application Express.js. Elle gère le routage et la logique des requêtes/réponses.
* `public/index.html` : la structure de notre application. Elle définit la façon dont les données sont présentées à l’utilisateur.
* `public/css/styles.css` : définit les styles de page, comme les polices, les couleurs et la taille du texte.
* `public/js/scripts.js` : contient la logique pour effectuer des requêtes auprès de l’API Recherche Web Bing, gérer les clés d’abonnement, gérer et analyser les réponses, et afficher les résultats.

Ce tutoriel se concentre sur `scripts.js` et la logique exigée pour appeler l’API Recherche Web Bing et gérer la réponse.

## <a name="html-form"></a>Formulaire HTML

L’élément `index.html` comprend un formulaire qui permet aux utilisateurs de rechercher et de sélectionner des options de recherche. L’attribut `onsubmit` se déclenche quand le formulaire est envoyé, en appelant la méthode `bingWebSearch()` définie dans `scripts.js`. Il tient compte de trois arguments :

* Requête de recherche
* Options sélectionnées
* Clé d’abonnement

```html
<form name="bing" onsubmit="return bingWebSearch(this.query.value,
    bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="query-options"></a>Options de requête

Le formulaire HTML inclut des options qui mappent à des paramètres de requête dans [l’API Recherche Web Bing v7](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query-parameters). Le tableau suivant montre comment les utilisateurs peuvent filtrer les résultats de recherche à l’aide de l’exemple d’application :

| Paramètre | Description |
|-----------|-------------|
| `query` | Champ de texte pour entrer une chaîne de requête. |
| `where` | Menu déroulant pour sélectionner le marché (lieu et langue). |
| `what` | Cases à cocher pour promouvoir des types de résultats spécifiques. La promotion d’images, par exemple, donne la priorité aux images dans les résultats de la recherche. |
| `when` | Menu déroulant qui permet à l’utilisateur de limiter les résultats de la recherche à aujourd’hui, à la semaine en cours ou au mois en cours. |
| `safe` | Case à cocher pour activer la fonctionnalité Filtre adulte Bing, qui exclut par filtre le contenu pour adultes. |
| `count` | Champ masqué. Nombre de résultats de recherche à renvoyer pour chaque requête. Changez cette valeur pour afficher plus ou moins de résultats par page. |
| `offset` | Champ masqué. Décalage du premier résultat de recherche dans la requête, qui est utilisé pour la pagination. Il est réinitialisé sur `0` à chaque nouvelle requête. |

> [!NOTE]
> L’API Recherche Web Bing offre des paramètres de requête supplémentaires pour affiner les résultats de la recherche. Le présent exemple en utilise seulement quelque uns. Pour obtenir la liste complète des paramètres disponibles, consultez les [informations de référence sur l’API Recherche Web Bing v7](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query-parameters).

La fonction `bingSearchOptions()` convertit ces options pour correspondre au format exigé par l’API Recherche Bing.

```javascript
// Build query options from selections in the HTML form.
function bingSearchOptions(form) {

    var options = [];
    // Where option.
    options.push("mkt=" + form.where.value);
    // SafeSearch option.
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "moderate"));
    // Freshness option.
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++)
        if (form.what[i].checked) what.push(form.what[i].value);
    // Promote option.
    if (what.length) {
        options.push("promote=" + what.join(","));
        options.push("answerCount=9");
    }
    // Count option.
    options.push("count=" + form.count.value);
    // Offset option.
    options.push("offset=" + form.offset.value);
    // Hardcoded text decoration option.
    options.push("textDecorations=true");
    // Hardcoded text format option.
    options.push("textFormat=HTML");
    return options.join("&");
}
```

`SafeSearch` peut être défini sur `strict`, `moderate` ou `off`, avec `moderate` qui est le paramètre par défaut pour la Recherche Web Bing. Ce formulaire utilise une case à cocher avec deux états possibles : `strict` ou `moderate`.

Si l’une des cases **Promouvoir** est cochée, le paramètre `answerCount` est ajouté à la requête. `answerCount` est requis lorsque vous utilisez le paramètre `promote`. Dans cet extrait de code, la valeur est définie sur `9` pour retourner tous les types de résultats disponibles.
> [!NOTE]
> La promotion d’un type de résultat ne *garantit* pas qu’il figurera dans les résultats de la recherche. Par contre, la promotion donne la priorité à ce type de résultats par rapport à son classement habituel. Pour limiter la recherche à des types particuliers de résultats, utilisez le paramètre de requête `responseFilter` ou appelez un point de terminaison plus spécifique, tels que Recherche d’images Bing ou Recherche d'actualités Bing.

Les paramètres de requête `textDecoration` et `textFormat` sont codés en dur dans le script, et entraînent l’affichage en gras du terme de recherche dans les résultats. Ces paramètres ne sont pas obligatoires.

## <a name="manage-subscription-keys"></a>Gérer les clés d’abonnement

Pour éviter le codage en dur de la clé d’abonnement de l’API Recherche Bing, cet exemple d’application utilise le stockage persistant d’un navigateur pour stocker la clé d’abonnement. Si aucune clé d’abonnement n’est stockée, l’utilisateur est invité à en entrer une. Si la clé d’abonnement est rejetée par l’API, l’utilisateur est invité à entrer à nouveau une clé d’abonnement.

La fonction `getSubscriptionKey()` utilise les fonctions `storeValue` et `retrieveValue` pour stocker et récupérer la clé d’abonnement d’un utilisateur. Ces fonctions utilisent l’objet `localStorage`, s’il est pris en charge, ou des cookies.

```javascript
// Cookie names for stored data.
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/search";

// See source code for storeValue and retrieveValue definitions.

// Get stored subscription key, or prompt if it isn't found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // Always set the cookie in order to update the expiration date.
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```

Comme nous l’avons vu précédemment, quand le formulaire est envoyé, `onsubmit` se déclenche, ce qui appelle `bingWebSearch`. Cette fonction initialise et envoie la requête. `getSubscriptionKey` est appelé à chaque envoi pour authentifier la requête.

## <a name="call-bing-web-search"></a>Appeler la Recherche Web Bing

Selon la requête, la chaîne d’options et la clé d’abonnement, la fonction `BingWebSearch` crée un objet `XMLHttpRequest` pour appeler le point de terminaison Recherche Web Bing.

```javascript
// Perform a search constructed from the query, options, and subscription key.
function bingWebSearch(query, options, key) {
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // Initialize the request.
    try {
        request.open("GET", queryurl);
    }
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // Add request headers.
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // Event handler for successful response.
    request.addEventListener("load", handleBingResponse);

    // Event handler for errors.
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // Event handler for an aborted request.
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // Send the request.
    request.send();
    return false;
}
```

Suite à une requête réussie, le gestionnaire d’événements `load` se déclenche et appelle la fonction `handleBingResponse`. `handleBingResponse` analyse l’objet de résultat, affiche les résultats et contient la logique d’erreur concernant les requêtes ayant échoué.

```javascript
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // Try to parse results object.
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
        return;
    }

    // Show raw JSON and the HTTP request.
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " +
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // If the HTTP response is 200 OK, try to render the results.
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "SearchResponse" && "rankingResponse" in jsobj) {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is considered an error.
    else {
        // 401 is unauthorized; force a re-prompt for the user's subscription
        // key on the next request.
        if (this.status === 401) invalidateSubscriptionKey();

        // Some error responses don't have a top-level errors object, if absent
        // create one.
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // Display the HTTP status code.
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // Add all fields from all error responses.
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // Display Bing Trace ID if it isn't blocked by CORS.
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // Display the error message.
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> Une requête HTTP réussie ne signifie *pas* que la recherche a réussi. Si une erreur se produit dans l’opération de recherche, l’API Recherche Web Bing renvoie un code d’état HTTP autre que 200 et inclut des informations sur l’erreur dans la réponse JSON. Si la requête avait un débit limité, l’API retourne une réponse vide.

Une grande partie du code dans les deux fonctions précédentes est dédiée à la gestion des erreurs. Des erreurs peuvent se produire aux étapes suivantes :

| Étape | Erreurs potentielles | Gérées par |
|-------|--------------------|------------|
| Génération de l’objet de requête | URL non valide | Bloc `try` / `catch` |
| Construction de la requête | Erreurs réseau, connexions abandonnées | Gestionnaires d’événements `error` et `abort` |
| Exécution de la recherche | Requête non valide, JSON non valide, limites de débit | Tests dans le gestionnaire d’événements `load` |

Les erreurs sont gérées en appelant `renderErrorMessage()`. Si la réponse réussit tous les tests d’erreur, `renderSearchResults()` est appelé pour afficher les résultats de la recherche.

## <a name="display-search-results"></a>Afficher les résultats de la recherche

Il existe des [conditions d’utilisation et d’affichage](./use-display-requirements.md) pour les résultats retournés par l’API Recherche Web Bing. Dans la mesure où une réponse peut inclure différents types de résultats, une itération dans la collection de niveau supérieur `WebPages` ne suffit pas. Au lieu de cela, l’exemple d’application utilise `RankingResponse` pour classer les résultats conformément aux spécifications.

> [!NOTE]
> Si vous voulez seulement un type de résultat unique, utilisez le paramètre de requête `responseFilter`, ou utilisez l’un des autres points de terminaison Recherche Bing, comme Recherche d’images Bing.

Chaque réponse a un objet `RankingResponse` qui peut inclure jusqu’à trois collections : `pole`, `mainline` et `sidebar`. `pole`, le cas échéant, est le résultat de recherche le plus pertinent et doit être affiché en premier. `mainline` contient la plupart des résultats de recherche et est affiché immédiatement après `pole`. `sidebar` inclut des résultats de recherche auxiliaires. Si possible, ces résultats doivent s’afficher dans la barre latérale. Si les limites de l’écran rendent la barre latérale peu pratique, ces résultats doivent apparaître après les résultats `mainline`.

Chaque `RankingResponse` inclut un tableau `RankingItem` qui spécifie la façon dont les résultats doivent être classés. Notre exemple d’application utilise les paramètres `answerType` et `resultIndex` pour identifier le résultat.

> [!NOTE]
> Il existe d’autres façons d’identifier et de classer les résultats. Pour plus d’informations, consultez [Utilisation du classement pour afficher les résultats](rank-results.md).

Examinons le code :

```javascript
// Render the search results from the JSON response.
function renderSearchResults(results) {

    // If spelling was corrected, update the search field.
    if (results.queryContext.alteredQuery)
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // Add Prev / Next links with result count.
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    // Render the results for each section.
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

La fonction `renderResultsItems()` effectue une itération dans les éléments de chaque collection `RankingResponse`, mappe chaque résultat de classement à un résultat de recherche à l’aide des valeurs `answerType` et `resultIndex`, et appelle la fonction de rendu appropriée pour générer le code HTML. Si `resultIndex` n’est pas spécifié pour un élément, `renderResultsItems()` effectue une itération dans tous les résultats de ce type et appelle la fonction de rendu pour chaque élément. Le code HTML obtenu est inséré dans l’élément `<div>` approprié dans `index.html`.

```javascript
// Render search results from the RankingResponse object per rank response and
// use and display requirements.
function renderResultsItems(section, results) {

    var items = results.rankingResponse[section].items;
    var html = [];
    for (var i = 0; i < items.length; i++) {
        var item = items[i];
        // Collection name has lowercase first letter while answerType has uppercase
        // e.g. `WebPages` RankingResult type is in the `webPages` top-level collection.
        var type = item.answerType[0].toLowerCase() + item.answerType.slice(1);
        if (type in results && type in searchItemRenderers) {
            var render = searchItemRenderers[type];
            // This ranking item refers to ONE result of the specified type.
            if ("resultIndex" in item) {
                html.push(render(results[type].value[item.resultIndex], section));
            // This ranking item refers to ALL results of the specified type.
            } else {
                var len = results[type].value.length;
                for (var j = 0; j < len; j++) {
                    html.push(render(results[type].value[j], section, j, len));
                }
            }
        }
    }
    return html.join("\n\n");
}
```

## <a name="review-renderer-functions"></a>Examiner les fonctions de renderer

Dans notre exemple d’application, l’objet `searchItemRenderers` inclut des fonctions qui génèrent du code HTML pour chaque type de résultat de recherche.

```javascript
// Render functions for each result type.
searchItemRenderers = {
    webPages: function(item) { ... },
    news: function(item) { ... },
    images: function(item, section, index, count) { ... },
    videos: function(item, section, index, count) { ... },
    relatedSearches: function(item, section, index, count) { ... }
}
```

> [!IMPORTANT]
> L’exemple d’application possède des renderers pour les pages web, les actualités, les images, les vidéos et les recherches connexes. Votre application aura besoin de renderers pour tous les types de résultats qu’elle peut recevoir, comme des calculs, des suggestions orthographiques, des entités, des fuseaux horaires et des définitions.

Certaines des fonctions de rendu acceptent uniquement le paramètre `item`. D’autres acceptent des paramètres supplémentaires, qui peuvent être utilisés pour restituer les éléments différemment en fonction du contexte. Un renderer qui n’utilise pas ces informations n’a pas besoin d’accepter ces paramètres.

Les arguments de contexte sont les suivants :

| Paramètre  | Description |
|------------|-------------|
| `section` | Section de résultats (`pole`, `mainline` ou `sidebar`) dans lequel l’élément figure. |
| `index`<br>`count` | Disponible lorsque l’élément `RankingResponse` spécifie que tous les résultats d’une collection donnée doivent être affichés ; `undefined` dans le cas contraire. L’index de l’élément dans sa collection et le nombre total d’éléments de cette collection. Vous pouvez utiliser ces informations pour compter les résultats, pour générer un code HTML différent pour le premier ou le dernier résultat, etc. |

Dans l’exemple d’application, les deux renderers `images` et `relatedSearches` utilisent les arguments de contexte pour personnaliser le code HTML généré. Examinons de plus près le renderer `images` :

```javascript
searchItemRenderers = {
    // Render image result with thumbnail.
    images: function(item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];
        if (section === "sidebar") {
            if (index) html.push("<br>");
        } else {
            if (!index) html.push("<p class='images'>");
        }
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        return html.join("");
    },
    // Other renderers are omitted from this sample...
}
```

Le renderer d’image :

* calcule la taille des miniatures (largeur variable, mais hauteur fixe de 60 pixels) ;
* insère le code HTML qui précède le résultat de l’image en fonction du contexte ;
* génère la balise HTML `<a>` qui établit un lien vers la page contenant l’image.
* Génère la balise HTML `<img>` pour afficher l’image miniature.

Le renderer d’image utilise les variables `section` et `index` pour afficher les résultats différemment selon l’endroit où ils apparaissent. Un saut de ligne (balise `<br>`) est inséré entre les résultats d’images dans la barre latérale, afin de les présenter sous forme de colonne. Dans d’autres sections, le premier résultat d’image `(index === 0)` est précédé de la balise `<p>`.

La taille des miniatures est utilisée à la fois dans la balise `<img>` et dans les champs `h` et `w` des URL de miniature. Les attributs `title` et `alt` (description textuelle de l’image) sont construits à partir de nom de l’image et du nom d’hôte dans l’URL.

Voici un exemple de la façon dont les images sont affichées dans l’exemple d’application :

![[Résultats d’images Bing]](media/cognitive-services-bing-web-api/web-search-spa-images.png)

## <a name="persist-the-client-id"></a>Rendre l’ID client permanent

Les réponses provenant des API Recherche Bing peuvent inclure un en-tête `X-MSEdge-ClientID` qui doit être renvoyé à l’API avec chaque requête successive. Si plusieurs API Recherche Bing sont utilisées par votre application, vérifiez que le même ID client est envoyé avec chaque requête entre les services.

Le fait de fournir l’en-tête `X-MSEdge-ClientID` permet aux API Bing d’associer des recherches d’un utilisateur. Tout d’abord, cela permet au moteur de recherche Bing d’appliquer un contexte passé aux recherches pour trouver les résultats répondant le mieux à la requête. Si un utilisateur a précédemment recherché des termes liés à la navigation, par exemple, une recherche ultérieure sur « nœuds » peut éventuellement renvoyer des informations sur les nœuds de navigation. Par ailleurs, Bing peut sélectionner au hasard des utilisateurs pour leur faire profiter des nouvelles fonctionnalités avant qu’elles ne deviennent disponibles au grand public. Le fait de fournir le même ID client avec chaque requête garantit que les utilisateurs choisis pour voir une fonctionnalité y auront toujours accès. Sans l’ID client, l’utilisateur peut voir une fonctionnalité apparaître et disparaître, de manière aléatoire, dans les résultats de recherche.

Les stratégies de sécurité de navigateur, comme le partage des ressources cross-origin (CORS), peuvent empêcher l’exemple d’application d’accéder à l’en-tête `X-MSEdge-ClientID`. Cette limitation se produit lorsque la réponse de recherche a une origine différente de la page d’où provient la requête. Dans un environnement de production, vous devez gérer cette stratégie en hébergeant un script côté serveur qui effectue l’appel d’API sur le même domaine que la page web. Étant donné que le script a la même origine que la page web, l’en-tête `X-MSEdge-ClientID` est ensuite disponible pour JavaScript.

> [!NOTE]
> Dans une application web de production, vous devez effectuer la requête côté serveur malgré tout. Dans le cas contraire, votre clé d’abonnement d’API Recherche Bing doit être incluse dans la page web, où elle est accessible à toute personne qui consulte la source. Vous êtes facturé pour toutes les utilisations associées à votre clé d’abonnement d’API, y compris les requêtes effectuées par des tiers non autorisés. Il est donc important de ne pas exposer votre clé.

À des fins de développement, vous pouvez effectuer la requête par le biais d’un proxy CORS. La réponse émanant de ce type de proxy a un en-tête `Access-Control-Expose-Headers` qui filtre les en-têtes de réponse et les rend disponibles pour JavaScript.

Il est facile d’installer un proxy CORS pour autoriser notre exemple d’application à accéder à l’en-tête d’ID client. Exécutez cette commande :

```console
npm install -g cors-proxy-server
```

Ensuite, remplacez le point de terminaison Recherche Web Bing dans `script.js` par :

```javascript
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search
```

Démarrer le proxy CORS avec la commande suivante :

```console
cors-proxy-server
```

Laissez la fenêtre de commande ouverte pendant que vous utilisez l’exemple d’application ; si vous fermez cette fenêtre, le proxy s’arrête. Dans la section En-têtes HTTP qui peut être développée, située sous les résultats de recherche, l’en-tête `X-MSEdge-ClientID` doit être visible. Vérifiez qu’il est identique pour chaque requête.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Référence pour l’API Recherche Web Bing v7](//docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)