---
title: 'Tutoriel : Comment charger une image à l’aide de la l’API Recherche visuelle Bing'
titleSuffix: Azure Cognitive Services
description: Découvrez comment charger une image sur Bing, obtenir des insights à son sujet, afficher la réponse.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: tutorial
ms.date: 03/31/2020
ms.author: scottwhi
ms.custom: devx-track-js
ms.openlocfilehash: 96a4b13d11e40e24e78d3aed8dfebcc88b41c525
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486876"
---
# <a name="tutorial-upload-images-to-the-bing-visual-search-api"></a>Tutoriel : Charger des images vers l’API Recherche visuelle Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche visuelle Bing vous permet de rechercher sur le web des images similaires à celles que vous chargez. Utilisez ce tutoriel pour créer une application web qui peut envoyer une image à l’API et afficher les aperçus que cette dernière retourne dans la page web. Notez que cette application n’est pas conforme à toutes les [conditions d’utilisation et d’affichage de Bing](../bing-web-search/use-display-requirements.md) concernant l’utilisation de l’API.

Vous trouverez le code source complet de cet exemple avec une gestion des erreurs et des annotations supplémentaires sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchUploadImage.html).

L’application du didacticiel illustre les actions suivantes :

> [!div class="checklist"]
> * Charger une image dans l’API Recherche visuelle Bing
> * Afficher les résultats de la recherche d’images dans une application web
> * Explorer les différents aperçus fournis par l’API

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="create-and-structure-the-webpage"></a>Créer et structurer une page Web

Créez une page HTML qui envoie une image à l’API Recherche visuelle Bing, reçoit des insights et les affiche. Dans votre éditeur ou votre IDE favori, créez un fichier nommé « uploaddemo.html ». Ajoutez la structure HTML de base suivante au fichier :

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Visual Search Upload Demo</title>
    </head>

    <body>
    </body>
</html>
```

Divisez la page en créant une section pour la requête où l’utilisateur fournit toutes les informations nécessaires pour la requête, puis une section pour la réponse, où les insights sont affichés. Ajoutez les balises `<div>` suivantes à `<body>`. La balise `<hr>` sépare visuellement la section pour la requête de la section pour la réponse :

```html
<div id="requestSection"></div>
<hr />
<div id="responseSection"></div>
```

Ajoutez une balise `<script>` à la balise `<head>` qui contiendra le code JavaScript pour l’application :

```html
<script>
<\script>
```

## <a name="get-the-upload-file"></a>Obtenir le fichier de chargement

Pour permettre à l’utilisateur de sélectionner l’image à charger, la démonstration utilise la balise `<input>` avec l’attribut de type défini sur `file`. L’interface utilisateur doit clairement indiquer que la démonstration utilise Bing pour obtenir les résultats de recherche.

Ajoutez l’élément `<div>` suivant à `requestSection` `<div>`. L’entrée du fichier accepte un seul fichier de n’importe quel type d’image (par exemple, .jpg, .gif, .png). L’événement `onchange` spécifie le gestionnaire appelé lorsqu’un utilisateur sélectionne un fichier.

La balise `<output>` est utilisée pour afficher une miniature de l’image sélectionnée :

```html
<div>
    <p>Select image to get insights from Bing:
        <input type="file" accept="image/*" id="uploadImage" name="files[]" size=40 onchange="handleFileSelect('uploadImage')" />
    </p>

    <output id="thumbnail"></output>
</div>
```

## <a name="create-a-file-handler"></a>Créer un descripteur de fichier

Créez une fonction descripteur qui peut lire dans l’image que vous souhaitez charger. Pendant l’itération sur les fichiers dans l’objet `FileList`, le gestionnaire doit s’assurer que le fichier sélectionné est un fichier image, et que sa taille fait au maximum 1 Mo. Si l’image est plus grande, vous devez réduire sa taille avant de la charger. Enfin, le gestionnaire affiche une miniature de l’image :

```javascript
function handleFileSelect(selector) {

    var files = document.getElementById(selector).files; // A FileList object

    for (var i = 0, f; f = files[i]; i++) {

        // Ensure the file is an image file.
        if (!f.type.match('image.*')) {
            alert("Selected file must be an image file.");
            document.getElementById("uploadImage").value = null;
            continue;
        }

        // Image must be <= 1 MB and should be about 1500px.
        if (f.size > 1000000) {
            alert("Image must be less than 1 MB.");
            document.getElementById("uploadImage").value = null;
            continue;
        }

        var reader = new FileReader();

        // Capture the file information.
        reader.onload = (function(theFile) {
            return function(e) {
                var fileOutput = document.getElementById('thumbnail');

                if (fileOutput.childElementCount > 0) {
                    fileOutput.removeChild(fileOutput.lastChild);  // Remove the current pic, if it exists
                }

                // Render thumbnail.
                var span = document.createElement('span');
                span.innerHTML = ['<img class="thumb" src="', e.target.result,
                                    '" title="', escape(theFile.name), '"/>'].join('');
                fileOutput.insertBefore(span, null);
            };
        })(f);

        // Read in the image file as a data URL.
        reader.readAsDataURL(f);
    }
}
```

## <a name="add-and-store-a-subscription-key"></a>Utiliser et stocker une clé d’abonnement

L’application a besoin d’une clé d’abonnement pour appeler l’API Recherche visuelle Bing. Pour ce didacticiel, vous devez la fournir dans l’interface utilisateur. Ajoutez la balise `<input>` suivante (avec l’attribut « type » défini sur « text ») à `<body>`, juste en dessous de la balise `<output>` du fichier :

```html
    <div>
        <p>Subscription key: 
            <input type="text" id="key" name="subscription" size=40 maxlength="32" />
        </p>
    </div>
```

En disposant de l’image et de la clé d’abonnement, vous pouvez effectuer l’appel à Recherche visuelle Bing pour obtenir des informations sur l’image. Dans ce tutoriel, l’appel utilise le marché par défaut (`en-us`) et la valeur de recherche sécurisée (`moderate`).

Cette application présente une option pour modifier ces valeurs. Ajoutez l’élément `<div>` suivant sous l’élément `<div>` de la clé d’abonnement. L’application utilise une balise `<select>` pour fournir une liste déroulante des valeurs de marché et de recherche sécurisée. Les deux listes affichent la valeur par défaut.

```html
<div>
    <p><a href="#" onclick="expandCollapse(options.id)">Query options</a></p>

    <div id="options" style="display:none">
        <p style="margin-left: 20px">Market: 
            <select id="mkt">
                <option value="es-AR">Argentina (Spanish)</option>
                <option value="en-AU">Australia (English)</option>
                <option value="de-AT">Austria (German)</option>
                <option value="nl-BE">Belgium (Dutch)</option>
                <option value="fr-BE">Belgium (French)</option>
                <option value="pt-BR">Brazil (Portuguese)</option>
                <option value="en-CA">Canada (English)</option>
                <option value="fr-CA">Canada (French)</option>
                <option value="es-CL">Chile (Spanish)</option>
                <option value="da-DK">Denmark (Danish)</option>
                <option value="fi-FI">Finland (Finnish)</option>
                <option value="fr-FR">France (French)</option>
                <option value="de-DE">Germany (German)</option>
                <option value="zh-HK">Hong Kong SAR(Traditional Chinese)</option>
                <option value="en-IN">India (English)</option>
                <option value="en-ID">Indonesia (English)</option>
                <option value="it-IT">Italy (Italian)</option>
                <option value="ja-JP">Japan (Japanese)</option>
                <option value="ko-KR">Korea (Korean)</option>
                <option value="en-MY">Malaysia (English)</option>
                <option value="es-MX">Mexico (Spanish)</option>
                <option value="nl-NL">Netherlands (Dutch)</option>
                <option value="en-NZ">New Zealand (English)</option>
                <option value="no-NO">Norway (Norwegian)</option>
                <option value="zh-CN">People's Republic of China (Chinese)</option>
                <option value="pl-PL">Poland (Polish)</option>
                <option value="pt-PT">Portugal (Portuguese)</option>
                <option value="en-PH">Philippines (English)</option>
                <option value="ru-RU">Russia (Russian)</option>
                <option value="ar-SA">Saudi Arabia (Arabic)</option>
                <option value="en-ZA">South Africa (English)</option>
                <option value="es-ES">Spain (Spanish)</option>
                <option value="sv-SE">Sweden (Swedish)</option>
                <option value="fr-CH">Switzerland (French)</option>
                <option value="de-CH">Switzerland (German)</option>
                <option value="zh-TW">Taiwan (Traditional Chinese)</option>
                <option value="tr-TR">Turkey (Turkish)</option>
                <option value="en-GB">United Kingdom (English)</option>
                <option value="en-US" selected>United States (English)</option>
                <option value="es-US">United States (Spanish)</option>
            </select>
        </p>
        <p style="margin-left: 20px">Safe search: 
            <select id="safesearch">
                <option value="moderate" selected>Moderate</option>
                <option value="strict">Strict</option>
                <option value="off">off</option>
            </select>
        </p>
    </div>
</div>
```

## <a name="add-search-options-to-the-webpage"></a>Ajouter des options de recherche à la page web

L’application masque les listes dans un élément `<div>` réductible contrôlé par le lien Options de requête. Quand vous cliquez sur le lien Options de requête, l’élément `<div>` se développe : vous pouvez alors afficher et modifier les options de la requête. Si vous recliquez sur le lien Options de requête, l’élément `<div>` est réduit et masqué. L’extrait de code suivant montre le gestionnaire `onclick` du lien Options de requête. Le gestionnaire contrôle si l’élément `<div>` est développé ou réduit. Ajoutez ce gestionnaire à la section `<script>`. Le gestionnaire est utilisé par toutes les sections `<div>` réductibles de la démonstration.

```javascript
// Contains the toggle state of divs.
var divToggleMap = {};  // divToggleMap['foo'] = 0;  // 1 = show, 0 = hide


// Toggles between showing and hiding the specified div.
function expandCollapse(divToToggle) {
    var div = document.getElementById(divToToggle);

    if (divToggleMap[divToToggle] == 1) {   // if div is expanded
        div.style.display = "none";
        divToggleMap[divToToggle] = 0;
    }
    else {                                  // if div is collapsed
        div.style.display = "inline-block";
        divToggleMap[divToToggle] = 1;
    }
}
```

## <a name="call-the-onclick-handler"></a>Appeler le gestionnaire `onclick`

Ajoutez le bouton `"Get insights"` suivant sous l’élément `<div>` des options dans le corps. Il vous permet de lancer l’appel. Quand l’utilisateur clique sur le bouton, le curseur se transforme en curseur d’attente en rotation et le gestionnaire `onclick` est appelé.

```html
<p><input type="button" id="query" value="Get insights" onclick="document.body.style.cursor='wait'; handleQuery()" /></p>
```

Ajoutez le gestionnaire `onclick` du bouton, `handleQuery()`, à la balise `<script>`.

## <a name="handle-the-query"></a>Gérer la requête

Le gestionnaire `handleQuery()` vérifie que la clé d’abonnement est présente, qu’elle contient 32 caractères, et qu’une image est sélectionnée. Il efface également toutes les informations d’une requête précédente. Par la suite, il appelle la fonction `sendRequest()` pour effectuer l’appel.

```javascript
function handleQuery() {
    var subscriptionKey = document.getElementById('key').value;

    // Make sure user provided a subscription key and image.
    // For this demo, the user provides the key but typically you'd
    // get it from secured storage.
    if (subscriptionKey.length !== 32) {
        alert("Subscription key length is not valid. Enter a valid key.");
        document.getElementById('key').focus();
        return;
    }

    var imagePath = document.getElementById('uploadImage').value;

    if (imagePath.length === 0)
    {
        alert("Please select an image to upload.");
        document.getElementById('uploadImage').focus();
        return;
    }

    var responseDiv = document.getElementById('responseSection');

    // Clear out the response from the last query.
    while (responseDiv.childElementCount > 0) {
        responseDiv.removeChild(responseDiv.lastChild);
    }

    // Send the request to Bing to get insights about the image.
    var f = document.getElementById('uploadImage').files[0];
    sendRequest(f, subscriptionKey);
}
```

## <a name="send-the-search-request"></a>Envoyer la requête de recherche

La fonction `sendRequest()` met en forme l’URL du point de terminaison, définit l’en-tête `Ocp-Apim-Subscription-Key` sur la clé d’abonnement, ajoute le fichier binaire de l’image à charger, spécifie le gestionnaire de réponse et effectue l’appel :

```javascript
function sendRequest(file, key) {
    var market = document.getElementById('mkt').value;
    var safeSearch = document.getElementById('safesearch').value;
    var baseUri = `https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch?mkt=${market}&safesearch=${safeSearch}`;

    var form = new FormData();
    form.append("image", file);

    var request = new XMLHttpRequest();

    request.open("POST", baseUri);
    request.setRequestHeader('Ocp-Apim-Subscription-Key', key);
    request.addEventListener('load', handleResponse);
    request.send(form);
}
```

## <a name="get-and-handle-the-api-response"></a>Obtenir et gérer la réponse de l’API

La fonction `handleResponse()` gère la réponse de l’appel à Recherche visuelle Bing. Si l’appel réussit, elle analyse la réponse JSON dans les balises individuelles qui contiennent les informations. Ensuite, elle ajoute les résultats de recherche à la page. L’application crée ensuite un élément `<div>` réductible pour chaque balise afin de gérer la quantité de données affichée. Ajoutez ce gestionnaire à la section `<script>`.

```javascript
function handleResponse() {
    if(this.status !== 200){
        alert("Error calling Bing Visual Search. See console log for details.");
        console.log(this.responseText);
        return;
    }

    var tags = parseResponse(JSON.parse(this.responseText));
    var h4 = document.createElement('h4');
    h4.textContent = 'Bing internet search results';
    document.getElementById('responseSection').appendChild(h4);
    buildTagSections(tags);

    document.body.style.cursor = 'default'; // reset the wait cursor set by query insights button
}
```

### <a name="parse-the-response"></a>Analyser la réponse

La fonction `parseResponse` convertit la réponse JSON en un objet de dictionnaire en traitant tous les `json.tags` de manière itérative.

```javascript
function parseResponse(json) {
    var dict = {};

    for (var i =0; i < json.tags.length; i++) {
        var tag = json.tags[i];

        if (tag.displayName === '') {
            dict['Default'] = JSON.stringify(tag);
        }
        else {
            dict[tag.displayName] = JSON.stringify(tag);
        }
    }

    return(dict);
}
```

### <a name="build-a-tag-section"></a>Créer une section de balise

La fonction `buildTagSections()` effectue une itération dans les balises JSON analysées et appelle la fonction `buildDiv()` pour générer un élément `<div>` pour chaque balise. Chaque balise est affichée sous forme de lien. Lorsque l’utilisateur clique sur le lien, la balise se développe pour afficher les informations qui lui sont associées. Le fait de recliquer sur le lien entraîne la réduction de la section.

```javascript
function buildTagSections(tags) {
    for (var tag in tags) {
        if (tags.hasOwnProperty(tag)) {
            var tagSection = buildDiv(tags, tag);
            document.getElementById('responseSection').appendChild(tagSection);
        }
    }  
}

function buildDiv(tags, tag) {
    var tagSection = document.createElement('div');
    tagSection.setAttribute('class', 'subSection');

    var link = document.createElement('a');
    link.setAttribute('href', '#');
    link.setAttribute('style', 'float: left;')
    link.text = tag;
    tagSection.appendChild(link);

    var contentDiv = document.createElement('div');
    contentDiv.setAttribute('id', tag);
    contentDiv.setAttribute('style', 'clear: left;')
    contentDiv.setAttribute('class', 'section');
    tagSection.appendChild(contentDiv);

    link.setAttribute('onclick', `expandCollapse("${tag}")`);
    divToggleMap[tag] = 0;  // 1 = show, 0 = hide

    addDivContent(contentDiv, tag, tags[tag]);

    return tagSection;
}
```

## <a name="display-the-search-results-in-the-webpage"></a>Afficher les résultats de la recherche dans la page web

La fonction `buildDiv()` appelle la fonction `addDivContent` pour générer le contenu de l’élément `<div>` réductible de chaque balise.

Le contenu d’une balise inclut l’élément JSON de la réponse pour la balise. Initialement, seuls les 100 premiers caractères du JSON sont affichés, mais vous pouvez cliquer sur la chaîne JSON pour afficher l’intégralité du JSON. Si vous cliquez à nouveau, la chaîne JSON est réduite à 100 caractères.

Ensuite, ajoutez les types d’actions trouvés dans la balise. Pour chaque type d’action, appelez les fonctions appropriées pour ajouter ses insights :

```javascript
function addDivContent(div, tag, json) {

    // Adds the first 100 characters of the json that contains
    // the tag's data. The user can click the text to show the
    // full json. They can click it again to collapse the json.
    var para = document.createElement('p');
    para.textContent = String(json).substr(0, 100) + '...';
    para.setAttribute('title', 'click to expand');
    para.setAttribute('style', 'cursor: pointer;')
    para.setAttribute('data-json', json);
    para.addEventListener('click', function(e) {
        var json = e.target.getAttribute('data-json');

        if (e.target.textContent.length <= 103) {  // 100 + '...'
            e.target.textContent = json;
            para.setAttribute('title', 'click to collapse');
        }
        else {
            para.textContent = String(json).substr(0, 100) + '...';
            para.setAttribute('title', 'click to expand');
        }
    });
    div.appendChild(para); 

    var parsedJson = JSON.parse(json);

    // Loop through all the actions in the tag and display them.
    for (var j = 0; j < parsedJson.actions.length; j++) {
        var action = parsedJson.actions[j];

        var subSectionDiv = document.createElement('div');
        subSectionDiv.setAttribute('class', 'subSection');
        div.appendChild(subSectionDiv);

        var h4 = document.createElement('h4');
        h4.innerHTML = action.actionType;
        subSectionDiv.appendChild(h4);

        if (action.actionType === 'ImageResults') {
            addImageWithWebSearchUrl(subSectionDiv, parsedJson.image, action);
        }
        else if (action.actionType === 'DocumentLevelSuggestions') {
            addRelatedSearches(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'RelatedSearches') {
            addRelatedSearches(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'PagesIncluding') {
            addPagesIncluding(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'VisualSearch') {
            addRelatedImages(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'Recipes') {
            addRecipes(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'ShoppingSources') {
            addShopping(subSectionDiv, action.data.offers);
        }
        else if (action.actionType === 'ProductVisualSearch') {
            addProducts(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'TextResults') {
            addTextResult(subSectionDiv, action);
        }
        else if (action.actionType === 'Entity') {
            addEntity(subSectionDiv, action);
        }
    }
}
```

## <a name="display-insights-for-different-actions"></a>Afficher les informations concernant différentes actions

Les fonctions suivantes affichent des informations pour différentes actions. Elles fournissent une image interactive ou un lien interactif qui vous renvoie à une page Web avec plus d’informations sur l’image. Cette page est hébergée soit par Bing.com, soit sur le site Web d’origine de l’image. Toutes les données des aperçus ne s’affichent pas dans cette application. Pour voir tous les champs disponibles pour un insight, consultez les informations de référence sur [Images - Recherche visuelle](/rest/api/cognitiveservices/bingvisualsearch/images/visualsearch).

> [!NOTE]
> Une quantité minimale d’informations d’aperçu doit être affichée sur la page. Pour plus d’informations , consultez [Conditions d’utilisation et d’affichage de l’API Recherche Bing](../bing-web-search/use-display-requirements.md).

### <a name="relatedimages-insights"></a>Insights RelatedImages

La fonction `addRelatedImages()` crée un titre pour chacun des sites web qui hébergent l’image associée en procédant à des itérations dans la liste des actions `RelatedImages` et en ajoutant une balise `<img>` à l’élément `<div>` extérieur pour chacun :

```javascript
    function addRelatedImages(div, images) {
        var length = (images.length > 10) ? 10 : images.length;

        // Set the title to the website that hosts the image. The title displays
        // when the user hovers over the image.

        // Make the image clickable. If the user clicks the image, they're taken
        // to the image in Bing.com.

        for (var j = 0; j < length; j++) {
            var img = document.createElement('img');
            img.setAttribute('src', images[j].thumbnailUrl + '&w=120&h=120');
            img.setAttribute('style', 'margin: 20px 20px 0 0; cursor: pointer;');
            img.setAttribute('title', images[j].hostPageDisplayUrl);
            img.setAttribute('data-webSearchUrl', images[j].webSearchUrl)

            img.addEventListener('click', function(e) {
                var url = e.target.getAttribute('data-webSearchUrl');
                window.open(url, 'foo');
            })

            div.appendChild(img);
        }
    }
```

### <a name="pagesincluding-insights"></a>Insights PagesIncluding

La fonction `addPagesIncluding()` crée un lien pour chacun des sites web qui hébergent l’image chargée en procédant à des itérations dans la liste des actions `PagesIncluding` et en ajoutant une balise `<img>` à l’élément `<div>` extérieur pour chacun :

```javascript

    // Display links to the first 5 webpages that include the image.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addPagesIncluding(div, pages) {
        var length = (pages.length > 5) ? 5 : pages.length;

        for (var j = 0; j < length; j++) {
            var page = document.createElement('a');
            page.text = pages[j].name;
            page.setAttribute('href', pages[j].hostPageUrl);
            page.setAttribute('style', 'margin: 20px 20px 0 0');
            page.setAttribute('target', '_blank')
            div.appendChild(page);

            div.appendChild(document.createElement('br'));
        }
    }
```

### <a name="relatedsearches-insights"></a>Insights RelatedSearches

La fonction `addRelatedSearches()` crée un lien pour le site web qui héberge l’image en procédant à des itérations dans la liste des actions `RelatedSearches` et en ajoutant une balise `<img>` à l’élément `<div>` extérieur pour chacune :

```javascript

    // Display the first 10 related searches. Include a link with the image
    // that when clicked, takes the user to Bing.com and displays the 
    // related search results.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addRelatedSearches(div, relatedSearches) {
        var length = (relatedSearches.length > 10) ? 10 : relatedSearches.length;

        for (var j = 0; j < length; j++) {
            var childDiv = document.createElement('div');
            childDiv.setAttribute('class', 'stackLink');
            div.appendChild(childDiv);

            var img = document.createElement('img');
            img.setAttribute('src', relatedSearches[j].thumbnail.url + '&w=120&h=120');
            img.setAttribute('style', 'margin: 20px 20px 0 0;');
            childDiv.appendChild(img);

            var relatedSearch = document.createElement('a');
            relatedSearch.text = relatedSearches[j].displayText;
            relatedSearch.setAttribute('href', relatedSearches[j].webSearchUrl);
            relatedSearch.setAttribute('target', '_blank');
            childDiv.appendChild(relatedSearch);

        }
    }
```

### <a name="recipes-insights"></a>Insights Recipes

La fonction `addRecipes()` crée un lien pour chacune des recettes en procédant à des itérations dans la liste des actions `Recipes` et en ajoutant une balise `<img>` à l’élément `<div>` extérieur pour chacune :

```javascript
    // Display links to the first 10 recipes. Include the recipe's rating,
    // if available.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addRecipes(div, recipes) {
        var length = (recipes.length > 10) ? 10 : recipes.length;

        for (var j = 0; j < length; j++) {
            var para = document.createElement('p');

            var recipe = document.createElement('a');
            recipe.text = recipes[j].name;
            recipe.setAttribute('href', recipes[j].url);
            recipe.setAttribute('style', 'margin: 20px 20px 0 0');
            recipe.setAttribute('target', '_blank')
            para.appendChild(recipe);

            if (recipes[j].hasOwnProperty('aggregateRating')) {
                var span = document.createElement('span');
                span.textContent = 'rating: ' + recipes[j].aggregateRating.text;
                para.appendChild(span);
            }

            div.appendChild(para);
        }
    }
```

### <a name="shopping-insights"></a>Insights Shopping

La fonction `addShopping()` crée un lien pour tout résultat d’achat renvoyé en procédant à des itérations dans la liste des actions `RelatedImages` et en ajoutant une balise `<img>` à l’élément `<div>` extérieur pour chacun :

```javascript
    // Display links for the first 10 shopping offers.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addShopping(div, offers) {
        var length = (offers.length > 10) ? 10 : offers.length;

        for (var j = 0; j < length; j++) {
            var para = document.createElement('p');

            var offer = document.createElement('a');
            offer.text = offers[j].name;
            offer.setAttribute('href', offers[j].url);
            offer.setAttribute('style', 'margin: 20px 20px 0 0');
            offer.setAttribute('target', '_blank')
            para.appendChild(offer);

            var span = document.createElement('span');
            span.textContent = 'by ' + offers[j].seller.name + ' | ' + offers[j].price + ' ' + offers[j].priceCurrency;
            para.appendChild(span);

            div.appendChild(para);
        }
    }
```

### <a name="products-insights"></a>Insights Products

La fonction `addProducts()` crée un lien pour chaque résultat de produit retourné en effectuant des itérations dans la liste des actions `Products` et en ajoutant une balise `<img>` à l’élément `<div>` extérieur pour chacun :

```javascript

    // Display the first 10 related products. Display a clickable image of the
    // product that takes the user to Bing.com search results for the product.
    // If there are any offers associated with the product, provide links to the offers.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addProducts(div, products) {
        var length = (products.length > 10) ? 10 : products.length;

        for (var j = 0; j < length; j++) {
            var childDiv = document.createElement('div');
            childDiv.setAttribute('class', 'stackLink');
            div.appendChild(childDiv);

            var img = document.createElement('img');
            img.setAttribute('src', products[j].thumbnailUrl + '&w=120&h=120');
            img.setAttribute('title', products[j].name);
            img.setAttribute('style', 'margin: 20px 20px 0 0; cursor: pointer;');
            img.setAttribute('data-webSearchUrl', products[j].webSearchUrl)
            img.addEventListener('click', function(e) {
                var url = e.target.getAttribute('data-webSearchUrl');
                window.open(url, 'foo');
            })
            childDiv.appendChild(img);

            if (products[j].insightsMetadata.hasOwnProperty('aggregateOffer')) {
                if (products[j].insightsMetadata.aggregateOffer.offerCount > 0) {
                    var offers = products[j].insightsMetadata.aggregateOffer.offers;

                    // Show all the offers. Not all markets provide links to offers.
                    for (var i = 0; i < offers.length; i++) {  
                        var para = document.createElement('p');

                        var offer = document.createElement('a');
                        offer.text = offers[i].name;
                        offer.setAttribute('href', offers[i].url);
                        offer.setAttribute('style', 'margin: 20px 20px 0 0');
                        offer.setAttribute('target', '_blank')
                        para.appendChild(offer);

                        var span = document.createElement('span');
                        span.textContent = 'by ' + offers[i].seller.name + ' | ' + offers[i].price + ' ' + offers[i].priceCurrency;
                        para.appendChild(span);

                        childDiv.appendChild(para);
                    }
                }
                else {  // Otherwise, just show the lowest price that Bing found.
                    var offer = products[j].insightsMetadata.aggregateOffer;

                    var para = document.createElement('p');
                    para.textContent = `${offer.name} | ${offer.lowPrice} ${offer.priceCurrency}`; 

                    childDiv.appendChild(para);
                }
            }
        }
    }
```

### <a name="textresult-insights"></a>Insights TextResult

La fonction `addTextResult()` affichera le texte reconnu dans l’image :

```javascript

    function addTextResult(div, action) {
        var text = document.createElement('p');
        text.textContent = action.displayName;
        div.appendChild(text);
    }
```

La fonction `addEntity()` affiche un lien qui redirige l’utilisateur vers Bing.com où il peut obtenir plus d’informations sur le type d’entité dans l’image, si une entité a été détectée :

```javascript
    // If the image is of a person, the tag might include an entity
    // action type. Display a link that takes the user to Bing.com
    // where they can get details about the entity.
    function addEntity(div, action) {
        var entity = document.createElement('a');
        entity.text = action.displayName;
        entity.setAttribute('href', action.webSearchUrl);
        entity.setAttribute('style', 'margin: 20px 20px 0 0');
        entity.setAttribute('target', '_blank');
        div.appendChild(entity);
    }
```

La fonction `addImageWithWebSearchUrl()` affiche une image interactive pour l’élément `<div>`, qui redirige l’utilisateur vers les résultats de recherche sur Bing.com :

```javascript
    function addImageWithWebSearchUrl(div, image, action) {
        var img = document.createElement('img');
        img.setAttribute('src', image.thumbnailUrl + '&w=120&h=120');
        img.setAttribute('style', 'margin: 20px 20px 0 0; cursor: pointer;');
        img.setAttribute('data-webSearchUrl', action.webSearchUrl);
        img.addEventListener('click', function(e) {
            var url = e.target.getAttribute('data-webSearchUrl');
            window.open(url, 'foo');
        })
        div.appendChild(img);
    }

```

## <a name="add-a-css-style"></a>Ajouter un style CSS

Ajoutez la section `<style>` suivante à la balise `<head>` pour organiser la disposition de la page web :

```html
        <style>

            .thumb {
                height: 75px;
                border: 1px solid #000;
            }

            .stackLink {
                width:180px;
                min-height:210px;
                display:inline-block;
            }
            .stackLink a {
                float:left;
                clear:left;
            }

            .section {
                float:left;
                display:none;
            }

            .subSection {
                clear:left;
                float:left;
            }

        </style>
```

## <a name="next-steps"></a>Étapes suivantes

>[!div class="nextstepaction"]
> [Tutoriel : Rechercher des images similaires dans des recherches précédentes à l’aide du jeton ImageInsightsToken](./tutorial-visual-search-insights-token.md)