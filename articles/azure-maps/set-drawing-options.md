---
title: Module Outils de dessin | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à définir des données d’options de dessin à l’aide du kit SDK Web Microsoft Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 01/29/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 95a04d763fa5982181cc1c797bce969d9857ae4b
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890630"
---
# <a name="use-the-drawing-tools-module"></a>Utiliser le module Outils de dessin

Le SDK web Azure Maps fournit un *module Outils de dessin* . Ce module facilite le dessin et la modification des formes sur la carte à l’aide d’un périphérique d’entrée tel qu’un écran tactile ou une souris. La classe de base de ce module est le [gestionnaire de dessins](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager#setoptions-drawingmanageroptions-). Le gestionnaire de dessins fournit toutes les fonctionnalités nécessaires pour dessiner et modifier des formes sur la carte. Il peut être utilisé directement et est intégré à une interface utilisateur de barre d’outils personnalisée. Vous pouvez également utiliser la classe de [barre d’outils de dessin](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar) intégrée. 

## <a name="loading-the-drawing-tools-module-in-a-webpage"></a>Chargement du module Outils de dessin dans une page web

1. Créez un fichier HTML et [implémentez la carte comme d’habitude](./how-to-use-map-control.md).
2. Chargez le module Outils de dessin d’Azure Maps. Vous pouvez le faire de deux façons :
    - Utiliser la version d’Azure Content Delivery Network du module des services Azure Maps hébergée globalement. Ajoutez une référence au code JavaScript et à la feuille de style CSS dans l’élément `<head>` du fichier :

        ```html
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/drawing/0/atlas-drawing.min.js"></script>
        ```

    - Ou vous pouvez charger le module Outils de dessin pour le code source du Kit de développement logiciel (SDK) web d’Azure Maps localement à l’aide du package npm [azure-maps-drawing-tools](https://www.npmjs.com/package/azure-maps-drawing-tools), puis l’héberger avec votre application. Ce package inclut aussi des définitions de TypeScript. Utilisez la commande suivante :
    
        > **npm install azure-maps-drawing-tools**
    
        Ensuite, ajoutez une référence au code JavaScript et à la feuille de style CSS dans l’élément `<head>` du fichier :

         ```html
        <link rel="stylesheet" href="node_modules/azure-maps-drawing-tools/dist/atlas-drawing.min.css" type="text/css" />
        <script src="node_modules/azure-maps-drawing-tools/dist/atlas-drawing.min.js"></script>
         ```

## <a name="use-the-drawing-manager-directly"></a>Utiliser le gestionnaire de dessins directement

Une fois le module Outils de dessin chargé dans votre application, vous pouvez activer les fonctionnalités de dessin et d’édition à l’aide du [gestionnaire de dessins](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager#setoptions-drawingmanageroptions-). Vous pouvez spécifier des options pour le gestionnaire de dessins lors de l’instanciation ou utiliser la fonction `drawingManager.setOptions()`.

### <a name="set-the-drawing-mode"></a>Définir le mode dessin

Le code suivant crée une instance du gestionnaire de dessins et définit l’option de mode **dessin** . 

```javascript
//Create an instance of the drawing manager and set drawing mode.
drawingManager = new atlas.drawing.DrawingManager(map,{
    mode: "draw-polygon"
});
```

Le code ci-dessous est un exemple complet d’exécution de la procédure de définition d’un mode dessin du gestionnaire de dessins. Cliquez sur la carte pour commencer à dessiner un polygone.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Dessiner un polygone" src="//codepen.io/azuremaps/embed/YzKVKRa/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le stylet <a href='https://codepen.io/azuremaps/pen/YzKVKRa/'>Draw a polygon</a> (Dessiner un polygone) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


### <a name="set-the-interaction-type"></a>Définir le type d’interaction

Pour tracer des formes, le gestionnaire de dessins prend en charge trois modes d’interaction avec la carte.

* `click` - Les coordonnées sont ajoutées au moyen d’un clic (souris ou interaction tactile).
* `freehand ` - Les coordonnées sont ajoutées au moyen d’un glisser-déplacer sur la carte (souris ou interaction tactile). 
* `hybrid` - Les coordonnées sont ajoutées au moyen d’un clic ou d’un glisser-déplacer (souris ou interaction tactile).

Le code suivant active le mode dessin de polygone et définit sur `freehand` le type d’interaction de dessin auquel le gestionnaire de dessins doit adhérer. 

```javascript
//Create an instance of the drawing manager and set drawing mode.
drawingManager = new atlas.drawing.DrawingManager(map,{
    mode: "draw-polygon",
    interactionType: "freehand"
});
```

 Cet exemple de code implémente la fonctionnalité de dessin d’un polygone sur la carte. Maintenez le bouton gauche de la souris enfoncé et faites glisser la souris librement.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Dessin libre" src="//codepen.io/azuremaps/embed/ZEzKoaj/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le stylet <a href='https://codepen.io/azuremaps/pen/ZEzKoaj/'>Free-hand drawing</a> (Dessin libre) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


### <a name="customizing-drawing-options"></a>Personnalisation des options de dessin

Les exemples précédents ont montré comment personnaliser les options de dessin lors de l’instanciation du gestionnaire de dessins. Vous pouvez également définir les options du gestionnaire de dessins à l’aide de la fonction `drawingManager.setOptions()`. Vous trouverez ci-dessous un outil permettant de tester la personnalisation de toutes les options du gestionnaire de dessins à l’aide de la fonction setOptions.

<br/>

<iframe height="685" title="Personnaliser le gestionnaire de dessins" src="//codepen.io/azuremaps/embed/LYPyrxR/?height=600&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true" style='width: 100%;'>Consultez le stylet <a href='https://codepen.io/azuremaps/pen/LYPyrxR/'>Get shape data</a> (Obtenir des données de forme) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser les fonctionnalités supplémentaires du module Outils de dessin :

> [!div class="nextstepaction"]
> [Ajouter une barre d’outils de dessin](map-add-drawing-toolbar.md)

> [!div class="nextstepaction"]
> [Récupérer les données de la forme](map-get-shape-data.md)

> [!div class="nextstepaction"]
> [Réagir aux événements de dessin](drawing-tools-events.md)

> [!div class="nextstepaction"]
> [Types d’interaction et raccourcis clavier](drawing-tools-interactions-keyboard-shortcuts.md)

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [Gestionnaire de dessins](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)

> [!div class="nextstepaction"]
> [Barre d’outils de dessin](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)