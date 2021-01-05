---
title: Événements de l’outil de dessin | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à ajouter une barre d’outils de dessin à une carte à l’aide du kit SDK web Microsoft Azure Maps
author: rbrundritt
ms.author: richbrun
ms.date: 12/05/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: 6abe0ed88adbdf8263aa27d340fb2fff156d98e6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90089341"
---
# <a name="drawing-tool-events"></a>Événements des outils de dessin

Il est utile de réagir à certains événements quand l’utilisateur utilise les outils de dessin sur la carte. Ce tableau présente tous les événements pris en charge par la classe `DrawingManager`.

| Événement | Description |
|-------|-------------|
| `drawingchanged` | Se déclenche en cas d’ajout ou de modification d’une coordonnée dans une forme. | 
| `drawingchanging` | Se déclenche lorsqu’une coordonnée d’aperçu d’une forme est affichée. Par exemple, cet événement se déclenche plusieurs fois en cas de glisser-déposer d’une coordonnée. | 
| `drawingcomplete` | Se déclenche lorsqu’une forme vient d’être dessinée ou sort du mode d’édition. |
| `drawingmodechanged` | Se déclenche lorsque le mode de dessin change. Le nouveau mode de dessin est passé dans le gestionnaire d’événements. |
| `drawingstarted` | Se déclenche lorsque l’utilisateur commence à dessiner une forme ou en met une en mode d’édition.  |

Le code suivant illustre le fonctionnement des événements du module Outils de dessin. Dessinez des formes sur la carte et regardez les événements se déclencher.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Événements des outils de dessin" src="https://codepen.io/azuremaps/embed/dyPMRWo?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/dyPMRWo'>Événements des outils de dessin</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="examples"></a>Exemples

Examinons quelques scénarios courants qui utilisent les événements des outils de dessin.

### <a name="select-points-in-polygon-area"></a>Sélectionner des points dans une zone polygonale

Ce code montre comment superviser un événement de dessin de formes par un utilisateur. Pour cet exemple, le code supervise les formes de polygones, de rectangles et de cercles. Ensuite, il détermine les points de données de la carte qui se trouvent dans la zone dessinée. L’événement `drawingcomplete` est utilisé pour déclencher la logique de sélection. Dans la logique de sélection, le code parcourt tous les points de données sur la carte. Il vérifie s’il existe une intersection entre le point et la zone de la forme dessinée. Cet exemple utilise la bibliothèque open source [Turf.js](https://turfjs.org/) pour effectuer un calcul d’intersection spatiale.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Sélectionner des données dans la zone polygonale dessinée" src="https://codepen.io/azuremaps/embed/XWJdeja?height=500&theme-id=default&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/XWJdeja'>Sélectionner des données dans une zone polygonale dessinée</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

### <a name="draw-and-search-in-polygon-area"></a>Dessiner une zone polygonale et effectuer une recherche

Ce code recherche des points d’intérêt à l’intérieur de la zone d’une forme une fois que l’utilisateur a terminé de dessiner celle-ci. Vous pouvez modifier et exécuter le code en cliquant sur « Edit on CODEPEN » en haut à droite du cadre. L’événement `drawingcomplete` est utilisé pour déclencher la logique de recherche. Si l’utilisateur dessine un rectangle ou un polygone, une recherche est effectuée dans la forme géométrique. Si un cercle est dessiné, le rayon et la position du centre sont utilisés pour effectuer une recherche de point d’intérêt. L’événement `drawingmodechanged` est utilisé pour déterminer quand l’utilisateur bascule vers le mode de dessin, et cet événement efface la zone de dessin.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Dessiner une zone polygonale et effectuer une recherche" src="https://codepen.io/azuremaps/embed/eYmZGNv?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/eYmZGNv'>Dessiner et effectuer une recherche dans une zone polygonale dessinée</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

### <a name="create-a-measuring-tool"></a>Créer un outil de mesure

Le code ci-dessous montre comment utiliser les événements de dessin pour créer un outil de mesure. `drawingchanging` permet de superviser la forme en cours de dessin. Lorsque l’utilisateur déplace la souris, les dimensions de la forme sont calculées. L’événement `drawingcomplete` permet d’effectuer un calcul final sur la forme une fois dessinée. L’événement `drawingmodechanged` est utilisé pour déterminer quand l’utilisateur bascule en mode de dessin. De plus, l’événement `drawingmodechanged` efface la zone de dessin et les anciennes informations de mesure.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Outil de mesure" src="https://codepen.io/azuremaps/embed/RwNaZXe?height=500&theme-id=default&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/RwNaZXe'>Outil de mesure</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser les fonctionnalités supplémentaires du module Outils de dessin :

> [!div class="nextstepaction"]
> [Récupérer les données de la forme](map-get-shape-data.md)

> [!div class="nextstepaction"]
> [Types d’interaction et raccourcis clavier](drawing-tools-interactions-keyboard-shortcuts.md)

Découvrez le module Service :

> [!div class="nextstepaction"]
> [Module de services](how-to-use-services-module.md)

Découvrez d’autres exemples de code :

> [!div class="nextstepaction"]
> [Page d’exemples de code](https://aka.ms/AzureMapsSamples)
