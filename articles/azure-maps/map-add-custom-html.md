---
title: Ajouter un marqueur HTML à une carte | Microsoft Azure Maps
description: Découvrez comment ajouter des marqueurs HTML aux cartes. Découvrez comment utiliser le Kit de développement logiciel (SDK) web Azure Maps pour personnaliser des marqueurs et ajouter des fenêtres contextuelles et des événements de souris à un marqueur.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 1c4367e2a649f4e239e2dab374afc4fb867e517b
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891191"
---
# <a name="add-html-markers-to-the-map"></a>Ajouter des marqueurs HTML à une carte

Cet article vous montre comment ajouter du code HTML personnalisé (comme un fichier image) à une carte, sous la forme d’un marqueur HTML.

> [!NOTE]
> Les marqueurs HTML ne se connectent pas aux sources de données. Au lieu de cela, des informations de position sont ajoutées directement au marqueur, qui est à son tour ajouté à la propriété `markers` de la carte, [HtmlMarkerManager](/javascript/api/azure-maps-control/atlas.htmlmarkermanager).

> [!IMPORTANT]
> Contrairement à la plupart des couches des contrôles web Azure Maps qui utilisent WebGL pour le rendu, les marqueurs HTML utilisent des éléments DOM traditionnels. Par conséquent, plus le nombre de marqueurs HTML ajoutés à une page est important, plus il y a d’éléments DOM. Les performances peuvent se dégrader si vous ajoutez plusieurs centaines de marqueurs HTML. Pour les jeux de données volumineux, envisagez le clustering de vos données ou l’utilisation d’une couche de symboles ou de bulles.

## <a name="add-an-html-marker"></a>Ajouter un marqueur HTML

La classe [HtmlMarker](/javascript/api/azure-maps-control/atlas.htmlmarker) a un style par défaut. Vous pouvez personnaliser le marqueur à l’aide des options de couleur et de texte. Le style par défaut de la classe Marqueur HTML est un modèle SVG qui comprend un espace réservé `{color}` et `{text}`. Définissez les propriétés de couleur et de texte dans les options de marqueur HTML pour une personnalisation rapide. 

Le code suivant crée un marqueur HTML et affecte à la propriété de couleur la valeur « DodgerBlue » et à la propriété de texte la valeur « 10 ». Une fenêtre contextuelle est attachée au marqueur et l’événement `click` est utilisé pour activer ou désactiver la visibilité de la fenêtre contextuelle.

```javascript
//Create an HTML marker and add it to the map.
var marker = new atlas.HtmlMarker({
    color: 'DodgerBlue',
    text: '10',
    position: [0, 0],
    popup: new atlas.Popup({
        content: '<div style="padding:10px">Hello World</div>',
        pixelOffset: [0, -30]
    })
});

map.markers.add(marker);

//Add a click event to toggle the popup.
map.events.add('click',marker, () => {
    marker.togglePopup();
});
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus.

<br/>

<iframe height='500' scrolling='no' title='Ajouter un marqueur HTML à une carte' src='//codepen.io/azuremaps/embed/MVoeVw/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/MVoeVw/'>Add an HTML Marker to a map</a> (Ajouter un marqueur HTML à une carte) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="create-svg-templated-html-marker"></a>Créer un marqueur HTML basé sur un modèle SVG

Le `htmlContent` par défaut d’un marqueur HTML est un modèle SVG contenant les espaces réservés `{color}` et `{text}`. Vous pouvez créer des chaînes SVG personnalisées et ajouter ces mêmes espaces réservés dans votre modèle SVG, de sorte que la définition des options `color` et `text` du marqueur mette à jour ces espaces réservés dans votre modèle SVG.

<br/>

<iframe height='500' scrolling='no' title='Marqueur HTML avec modèle SVG personnalisé' src='//codepen.io/azuremaps/embed/LXqMWx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/LXqMWx/'>HTML Marker with Custom SVG Template</a> (Marqueur HTML avec modèle SVG personnalisé) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Le kit de développement logiciel (SDK) web Azure Maps fournit plusieurs modèles d’images SVG utilisables avec des marqueurs HTML. Pour plus d’informations, consultez le document [Guide pratique pour utiliser des modèles d’image](how-to-use-image-templates-web-sdk.md).

## <a name="add-a-css-styled-html-marker"></a>Ajouter un marqueur HTML avec style CSS

L’un des avantages des marqueurs HTML est que vous pouvez utiliser de nombreuses personnalisations intéressantes à l’aide de feuilles de style en cascade. Dans cet exemple, le contenu de HtmlMarker se compose de code HTML et de feuilles de style en cascade qui créent une épingle animée qui « pulse » une fois en place.

<br/>

<iframe height='500' scrolling='no' title='HTML DataSource' src='//codepen.io/azuremaps/embed/qJVgMx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/qJVgMx/'>HTML DataSource</a> (Source de données HTML) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="draggable-html-markers"></a>Marqueurs HTML déplaçables

Cet exemple montre comment rendre un marqueur HTML déplaçable. Les marqueurs HTML prennent en charge les événements `drag`, `dragstart` et `dragend`.

<br/>

<iframe height='500' scrolling='no' title='Marqueur HTML déplaçable' src='//codepen.io/azuremaps/embed/wQZoEV/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/wQZoEV/'>Draggable HTML Marker</a> (Marqueur HTML déplaçable) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-mouse-events-to-html-markers"></a>Ajouter des événements de souris aux marqueurs HTML

Ces exemples montrent comment ajouter des événements de souris et de glisser à un marqueur HTML.

<br/>

<iframe height='500' scrolling='no' title='Ajout d’événements de souris aux marqueurs HTML' src='//codepen.io/azuremaps/embed/RqOKRz/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/RqOKRz/'>Adding Mouse Events to HTML Markers</a> (Ajout d’événements de souris aux marqueurs HTML) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [HtmlMarker](/javascript/api/azure-maps-control/atlas.htmlmarker)

> [!div class="nextstepaction"]
> [HtmlMarkerOptions](/javascript/api/azure-maps-control/atlas.htmlmarkeroptions)

> [!div class="nextstepaction"]
> [HtmlMarkerManager](/javascript/api/azure-maps-control/atlas.htmlmarkermanager)

Pour consulter plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Guide pratique pour utiliser des modèles d’image](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de bulles](./map-add-bubble-layer.md)