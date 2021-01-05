---
title: " Créer une application web d’une seule page – Recherche visuelle Bing"
titleSuffix: Azure Cognitive Services
description: Découvrez comment intégrer l’API Recherche visuelle Bing dans une application web d’une seule page.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: tutorial
ms.date: 03/27/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: b1ca88cd654b7bf373bee60b1e9b7a2e7a129fa2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499014"
---
# <a name="tutorial-create-a-visual-search-single-page-web-app"></a>Tutoriel : Créer une application Recherche visuelle d’une seule page

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche visuelle Bing retourne des insights pour une image. Vous pouvez charger une image ou fournir une URL vers une image. Les insights sont des images visuellement similaires, des sources d’achat, des pages web qui incluent l’image, etc. Les insights retournés par l’API Recherche visuelle Bing sont similaires à ceux qui apparaissent sur Bing.com/images.

Ce didacticiel explique comment étendre une application web d’une seule page pour l’API Recherche d’images Bing. Pour afficher ce tutoriel ou obtenir le code source utilisé ici, consultez [Tutoriel : Créer une application monopage avec l’API Recherche d’images Bing](../Bing-Image-Search/tutorial-bing-image-search-single-page-app.md).

Le code source complet pour cette application (après son extension pour utiliser l’API Recherche visuelle Bing) est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchApp.html).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="call-the-bing-visual-search-api-and-handle-the-response"></a>Appeler l’API Recherche visuelle Bing et gérer la réponse

Modifiez le didacticiel Recherche d’images Bing et ajoutez le code suivant à la fin de l’élément `<script>` (et devant la balise `</script>` de fermeture). Le code suivant gère une réponse de recherche visuelle à partir de l’API, effectue une itération dans les résultats, et affiche ceux-ci :

``` javascript
function handleVisualSearchResponse(){
    if(this.status !== 200){
        console.log(this.responseText);
        return;
    }
    let json = this.responseText;
    let response = JSON.parse(json);
    for (let i =0; i < response.tags.length; i++) {
        let tag = response.tags[i];
        if (tag.displayName === '') {
            for (let j = 0; j < tag.actions.length; j++) {
                let action = tag.actions[j];
                if (action.actionType === 'VisualSearch') {
                    let html = '';
                    for (let k = 0; k < action.data.value.length; k++) {
                        let item = action.data.value[k];
                        let height = 120;
                        let width = Math.max(Math.round(height * item.thumbnail.width / item.thumbnail.height), 120);
                        html += "<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width + "' height=" + height + " width=" + width + "'>";
                    }
                    showDiv("insights", html);
                    window.scrollTo({top: document.getElementById("insights").getBoundingClientRect().top, behavior: "smooth"});
                }
            }
        }
    }
}
```

Le code suivant envoie une demande de recherche à l’API, en utilisant un détecteur d’événements pour appeler `handleVisualSearchResponse()` :

```javascript
function bingVisualSearch(insightsToken){
    let visualSearchBaseURL = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch',
        boundary = 'boundary_ABC123DEF456',
        startBoundary = '--' + boundary,
        endBoundary = '--' + boundary + '--',
        newLine = "\r\n",
        bodyHeader = 'Content-Disposition: form-data; name="knowledgeRequest"' + newLine + newLine;

    let postBody = {
        imageInfo: {
            imageInsightsToken: insightsToken
        }
    }
    let requestBody = startBoundary + newLine;
    requestBody += bodyHeader;
    requestBody += JSON.stringify(postBody) + newLine + newLine;
    requestBody += endBoundary + newLine;

    let request = new XMLHttpRequest();

    try {
        request.open("POST", visualSearchBaseURL);
    } 
    catch (e) {
        renderErrorMessage("Error performing visual search: " + e.message);
    }
    request.setRequestHeader("Ocp-Apim-Subscription-Key", getSubscriptionKey());
    request.setRequestHeader("Content-Type", "multipart/form-data; boundary=" + boundary);
    request.addEventListener("load", handleVisualSearchResponse);
    request.send(requestBody);
}
```

## <a name="capture-insights-token"></a>Capturer le jeton d’insights

Ajoutez le code suivant à l’objet `searchItemsRenderer`. Ce code ajoute un lien **Rechercher semblables** qui appelle la fonction `bingVisualSearch` quand vous cliquez dessus. La fonction reçoit le `imageInsightsToken` en tant qu’argument.

``` javascript
html.push("<a href='javascript:bingVisualSearch(\"" + item.imageInsightsToken + "\");'>find similar</a><br>");
```

## <a name="display-similar-images"></a>Afficher des images similaires

Ajoutez le code HTML suivant à la ligne 601. Ce code de balisage ajoute un élément utilisé pour afficher les résultats de l’appel de l’API Recherche visuelle Bing :

``` html
<div id="insights">
    <h3><a href="#" onclick="return toggleDisplay('_insights')">Similar</a></h3>
    <div id="_insights" style="display: none"></div>
</div>
```

Avec l’ensemble du nouveau code JavaScript et des nouveaux éléments HTML en place, les résultats de la recherche sont affichés avec un lien **Rechercher semblables**. Cliquez sur le lien pour remplir la section **Semblable** avec des images similaires à celle que vous avez choisie. Vous devrez peut-être développer la section **Semblable** pour afficher les images.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Rogner une image avec le SDK Recherche visuelle Bing pour C#](tutorial-visual-search-crop-area-results.md)