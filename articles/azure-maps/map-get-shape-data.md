---
title: Obtenir des données de formes sur une carte | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à obtenir des données de formes dessinées sur une carte à l’aide du SDK web Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/04/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: ddb8009e544ede82d1c56d112950ff247a87380c
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890698"
---
# <a name="get-shape-data"></a>Obtenir des données de forme

Cet article explique comment obtenir des données de formes dessinées sur la carte. Nous utilisons la fonction **drawingManager.getSource()** à l’intérieur du [gestionnaire de dessins](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager#getsource--). Il existe différents scénarios lorsque vous souhaitez extraire les données GeoJSON d’une forme dessinée et les utiliser ailleurs.  


## <a name="get-data-from-drawn-shape"></a>Obtenir des données à partir d’une forme dessinée

La fonction suivante obtient les données source de la forme dessinée et les affiche à l’écran. 

```javascript
function getDrawnShapes() {
    var source = drawingManager.getSource();

    document.getElementById('CodeOutput').value = JSON.stringify(source.toJson(), null, '    ');
}
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet, dans lequel vous pouvez dessiner une forme pour tester les fonctionnalités :

<br/>

<iframe height="686" title="Obtenir des données de forme" src="//codepen.io/azuremaps/embed/xxKgBVz/?height=265&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true" style='width: 100%;'>Consultez le stylet <a href='https://codepen.io/azuremaps/pen/xxKgBVz/'>Get shape data</a> (Obtenir des données de forme) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser les fonctionnalités supplémentaires du module Outils de dessin :

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