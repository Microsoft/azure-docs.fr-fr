---
title: Ajouter une barre d’outils de dessin à une carte | Microsoft Azure Maps
description: Comment ajouter une barre d’outils de dessin à une carte à l’aide du kit de développement logiciel (SDK) web Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/04/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: b00628ec5a9f41b027bf90b93421f3aa1404e97a
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896393"
---
# <a name="add-a-drawing-tools-toolbar-to-a-map"></a>Ajouter une barre d’outils de dessin à une carte

Cet article explique comment utiliser le module Outils de dessin et afficher la barre d’outils de dessin sur la carte. Le contrôle [DrawingToolbar](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar) ajoute la barre d’outils de dessin sur la carte. Vous allez apprendre à créer des cartes avec un seul et tous les outils de dessin et à personnaliser le rendu des formes de dessin dans le gestionnaire de dessins.

## <a name="add-drawing-toolbar"></a>Ajouter une barre d’outils de dessin

Le code suivant crée une instance du gestionnaire de dessins et affiche la barre d’outils sur la carte.

```javascript
//Create an instance of the drawing manager and display the drawing toolbar.
drawingManager = new atlas.drawing.DrawingManager(map, {
        toolbar: new atlas.control.DrawingToolbar({
            position: 'top-right',
            style: 'dark'
        })
    });
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus :

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Ajouter une barre d’outils de dessin" src="//codepen.io/azuremaps/embed/ZEzLeRg/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez le stylet <a href='https://codepen.io/azuremaps/pen/ZEzLeRg/'>Add drawing toolbar</a> (Ajouter une barre d’outils de dessin) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="limit-displayed-toolbar-options"></a>Limiter les options de barre d’outils affichées

Le code suivant crée une instance du gestionnaire de dessins avec seulement un outil de dessin de polygones et affiche la barre d’outils sur la carte. 

```javascript
//Create an instance of the drawing manager and display the drawing toolbar with polygon drawing tool.
drawingManager = new atlas.drawing.DrawingManager(map, {
        toolbar: new atlas.control.DrawingToolbar({
            position: 'top-right',
            style: 'light',
            buttons: ["draw-polygon"]
        })
    });
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus :

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Ajouter un outil de dessin de polygone" src="//codepen.io/azuremaps/embed/OJLWWMy/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez le stylet <a href='https://codepen.io/azuremaps/pen/OJLWWMy/'>Add a polygon drawing tool</a> (Ajouter un outil de dessin de polygone) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="change-drawing-rendering-style"></a>Modifier le style de rendu de dessin

Le style des formes qui sont dessinées peut être personnalisé en extrayant les couches sous-jacentes du gestionnaire de dessins à l’aide de la fonction `drawingManager.getLayers()`, puis en définissant les options sur les couches individuelles. Les poignées de glissement qui s’affichent pour les coordonnées lors de la modification d’une forme sont des marqueurs HTML. Le style des poignées de glissement peut être personnalisé en passant les options de marqueur HTML aux options `dragHandleStyle` et `secondaryDragHandleStyle` du gestionnaire de dessins.  

Le code suivant obtient les couches de rendu à partir du gestionnaire de dessins et modifie leurs options pour modifier le style de rendu du dessin. Dans ce cas, les points sont affichés avec une icône marker bleue. Les lignes sont rouges et d’une largeur de 4 pixels. Les polygones ont une couleur de remplissage verte et un contour orange. Il modifie ensuite les styles des poignées de glissement en icônes carrées. 

```javascript
//Get rendering layers of drawing manager.
var layers = drawingManager.getLayers();

//Change the icon rendered for points.
layers.pointLayer.setOptions({
    iconOptions: {
        image: 'marker-blue'
    }
});

//Change the color and width of lines.
layers.lineLayer.setOptions({
    strokeColor: 'red',
    strokeWidth: 4
});

//Change fill color of polygons.
layers.polygonLayer.setOptions({
    fillColor: 'green'
});

//Change the color of polygon outlines.
layers.polygonOutlineLayer.setOptions({
    strokeColor: 'orange'
});

//Update the style of the drag handles that appear when editting.
drawingManager.setOptions({
    //Primary drag handle that represents coordinates in the shape.
    dragHandleStyle: {
        anchor: 'center',
        htmlContent: '<svg width="15" height="15" viewBox="0 0 15 15" xmlns="http://www.w3.org/2000/svg" style="cursor:pointer"><rect x="0" y="0" width="15" height="15" style="stroke:black;fill:white;stroke-width:4px;"/></svg>',
        draggable: true
    },

    //Secondary drag hanle that represents mid-point coordinates that users can grab to add new cooridnates in the middle of segments.
    secondaryDragHandleStyle: {
        anchor: 'center',
        htmlContent: '<svg width="10" height="10" viewBox="0 0 10 10" xmlns="http://www.w3.org/2000/svg" style="cursor:pointer"><rect x="0" y="0" width="10" height="10" style="stroke:white;fill:black;stroke-width:4px;"/></svg>',
        draggable: true
    }
});  
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus :

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Modifier le style de rendu de dessin" src="//codepen.io/azuremaps/embed/OJLWpyj/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page <a href='https://codepen.io/azuremaps/pen/OJLWpyj/'>Modifier le style de rendu de dessin</a> par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser les fonctionnalités supplémentaires du module Outils de dessin :

> [!div class="nextstepaction"]
> [Obtenir des données de forme](map-get-shape-data.md)

> [!div class="nextstepaction"]
> [Réagir aux événements de dessin](drawing-tools-events.md)

> [!div class="nextstepaction"]
> [Types d’interaction et raccourcis clavier](drawing-tools-interactions-keyboard-shortcuts.md)

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [Barre d’outils de dessin](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)

> [!div class="nextstepaction"]
> [Gestionnaire de dessins](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)