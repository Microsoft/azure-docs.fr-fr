---
title: Types d’interaction et raccourcis clavier des outils de dessin sur la carte | Microsoft Azure Maps
description: Découvrez comment tracer et modifier des formes en utilisant la souris, l’écran tactile ou le clavier dans le SDK Microsoft Azure Maps
author: rbrundritt
ms.author: richbrun
ms.date: 12/05/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: b609a21348222150faf5fb828f2cc38ca6658f5d
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895815"
---
# <a name="interaction-types-and-keyboard-shortcuts-in-the-drawing-tools-module"></a>Types d'interaction et raccourcis clavier dans le module Outils de dessin

Cet article décrit les différentes méthodes disponibles pour tracer et modifier des formes à l’aide d’une souris, d’un écran tactile ou de raccourcis clavier.

Pour tracer des formes, le gestionnaire de dessins prend en charge trois modes d’interaction avec la carte.

* `click` - Les coordonnées sont ajoutées au moyen d’un clic (souris ou interaction tactile).
* `freehand ` - Les coordonnées sont ajoutées au moyen d’un glisser-déplacer sur la carte (souris ou interaction tactile).
* `hybrid` - Les coordonnées sont ajoutées au moyen d’un clic ou d’un glisser-déplacer (souris ou interaction tactile).

## <a name="how-to-draw-shapes"></a>Tracer des formes

 Avant de pouvoir tracer une forme, définissez l’option `drawingMode` du gestionnaire de dessins sur un paramètre de dessin pris en charge. Vous pouvez programmer ce paramètre, ou bien l’appeler en appuyant sur l’un des boutons de dessin de la barre d’outils. Le mode Dessin reste activé, même après la création d'une forme, ce qui facilite l'ajout d'autres formes du même type. Définissez programmatiquement le mode Dessin sur un état inactif. Vous pouvez aussi basculer vers un état inactif en cliquant sur le bouton Modes de dessin actuel de la barre d’outils.

Les sections suivantes présentent les différentes méthodes disponibles pour tracer des formes sur la carte.

### <a name="how-to-draw-a-point"></a>Tracer un point

Lorsque le gestionnaire de dessins est en mode de dessin `draw-point`, les actions suivantes peuvent être effectuées pour tracer des points sur la carte. Ces méthodes fonctionnent avec tous les modes d'interaction.

**Commencer à dessiner**
 - Cliquez sur le bouton gauche de la souris ou touchez la carte pour y ajouter un point. 
 - Si la souris se trouve sur la carte, appuyez sur la touche `F`. Un point sera alors ajouté aux coordonnées du pointeur de la souris. Cette méthode offre une plus grande précision pour ajouter un point à la carte. Aucune pression ne sera exercée sur le bouton gauche de la souris, ce qui risquerait de déplacer légèrement celle-ci.
 - Continuez à cliquer, toucher ou appuyer sur `F` pour ajouter d'autres points sur la carte.
 
**Terminer un dessin**
 - Cliquez sur un bouton de la barre d'outils Dessin. 
 - Définissez par programmation le mode Dessin. 
 - Appuyez sur la touche `C`.

**Annuler un dessin**
 - Appuyez sur la touche `Escape`.

### <a name="how-to-draw-a-line"></a>Tracer une ligne

Quand le gestionnaire de dessins est en mode `draw-line`, les actions suivantes peuvent être effectuées pour tracer des points sur la carte, en fonction du mode d’interaction.

**Commencer à dessiner**
 - Mode Clic
   * Cliquez sur le bouton gauche de la souris ou appuyez sur la carte pour y ajouter chacun des points d’une ligne. Les coordonnées correspondant à chaque clic ou interaction tactile sont ajoutées à la ligne. 
   * Si la souris se trouve sur la carte, appuyez sur la touche `F`. Un point sera alors ajouté aux coordonnées du pointeur de la souris. Cette méthode offre une plus grande précision pour ajouter un point à la carte. Aucune pression ne sera exercée sur le bouton gauche de la souris, ce qui risquerait de déplacer légèrement celle-ci.
   * Continuez à cliquer jusqu'à ce que tous les points souhaités aient été ajoutés à la ligne.
 - Mode Main levée
   * Appuyez sur le bouton gauche de la souris ou sur la carte et faites glisser la souris ou le point tactile. Les coordonnées sont ajoutées à la ligne à mesure que la souris ou le point tactile se déplace sur la carte. Dès que l'événement associé à la souris ou à l'interaction tactile est déclenché, le dessin est effectué. La fréquence à laquelle les coordonnées sont ajoutées est définie par l'option `freehandInterval` du gestionnaire de dessins.
 - Mode hybride
   * Passez à votre guise du mode Clic au mode Main levée (ou inversement) lors du tracé d’une ligne. Par exemple, cliquez sur quelques points, maintenez le bouton gauche enfoncé et faites glisser la souris pour ajouter une série de points, puis cliquez sur d'autres points. 

**Terminer un dessin**
 - Mode Hybride/Clic
   * Une fois le dernier point atteint, double-cliquez sur la carte. 
   * Cliquez sur un bouton de la barre d'outils Dessin. 
   * Définissez par programmation le mode Dessin. 
 - Mode Main levée
   * Relâchez le bouton de la souris ou le point tactile.
 - Appuyez sur la touche `C`.

**Annuler un dessin**
 - Appuyez sur la touche `Escape`.

### <a name="how-to-draw-a-polygon"></a>Tracer un polygone

Quand le gestionnaire de dessins est en mode `draw-polygon`, les actions suivantes peuvent être effectuées pour tracer des points sur la carte, en fonction du mode d’interaction.

**Commencer à dessiner**
 - Mode Clic
   * Cliquez sur le bouton gauche de la souris ou appuyez sur la carte pour y ajouter chacun des points d’un polygone. Les coordonnées correspondant à chaque clic ou interaction tactile sont ajoutées au polygone. 
   * Si la souris se trouve sur la carte, appuyez sur la touche `F`. Un point sera alors ajouté aux coordonnées du pointeur de la souris. Cette méthode offre une plus grande précision pour ajouter un point à la carte. Aucune pression ne sera exercée sur le bouton gauche de la souris, ce qui risquerait de déplacer légèrement celle-ci.
   * Continuez à cliquer jusqu'à ce que tous les points souhaités aient été ajoutés au polygone.
 - Mode Main levée
   * Appuyez sur le bouton gauche de la souris ou sur la carte et faites glisser la souris ou le point tactile. Les coordonnées sont ajoutées au polygone à mesure que la souris ou le point tactile se déplace sur la carte. Dès que l'événement associé à la souris ou à l'interaction tactile est déclenché, le dessin est effectué. La fréquence à laquelle les coordonnées sont ajoutées est définie par l'option `freehandInterval` du gestionnaire de dessins.
 - Mode hybride
   * Passez à votre guise du mode Clic au mode Main levée (ou inversement) lors du tracé d’un polygone. Par exemple, cliquez sur quelques points, maintenez le bouton gauche enfoncé et faites glisser la souris pour ajouter une série de points, puis cliquez sur d'autres points. 

**Terminer un dessin**
 - Mode Hybride/Clic
   * Une fois le dernier point atteint, double-cliquez sur la carte. 
   * Cliquez sur le premier point du polygone.
   * Cliquez sur un bouton de la barre d'outils Dessin. 
   * Définissez par programmation le mode Dessin. 
 - Mode Main levée
   * Relâchez le bouton de la souris ou le point tactile.
 - Appuyez sur la touche `C`.

**Annuler un dessin**
 - Appuyez sur la touche `Escape`.

### <a name="how-to-draw-a-rectangle"></a>Tracer un rectangle

Quand le gestionnaire de dessins est en mode `draw-rectangle`, les actions suivantes peuvent être effectuées pour tracer des points sur la carte, en fonction du mode d’interaction. La forme générée suivra la [spécification GeoJSON étendue relative aux rectangles](extend-geojson.md#rectangle).

**Commencer à dessiner**
 - Appuyez sur le bouton gauche de la souris ou sur la carte pour ajouter le premier angle du rectangle, puis faites glisser pour créer le rectangle. 

**Terminer un dessin**
 - Relâchez le bouton de la souris ou le point tactile.
 - Définissez par programmation le mode Dessin. 
 - Appuyez sur la touche `C`.

**Annuler un dessin**
 - Appuyez sur la touche `Escape`.

### <a name="how-to-draw-a-circle"></a>Tracer un cercle

Quand le gestionnaire de dessins est en mode `draw-circle`, les actions suivantes peuvent être effectuées pour tracer des points sur la carte, en fonction du mode d’interaction. La forme générée suivra la [spécification GeoJSON étendue relative aux cercles](extend-geojson.md#circle).

**Commencer à dessiner**
 - Appuyez sur le bouton gauche de la souris ou sur la carte pour ajouter le centre du cercle, puis faites glisser pour donner un rayon au cercle. 

**Terminer un dessin**
 - Relâchez le bouton de la souris ou le point tactile.
 - Définissez par programmation le mode Dessin. 
 - Appuyez sur la touche `C`.

**Annuler un dessin**
 - Appuyez sur la touche `Escape`.

## <a name="keyboard-shortcuts"></a>Raccourcis clavier

Les outils de dessin prennent en charge les raccourcis clavier. Ces raccourcis clavier fonctionnent quand le focus est placé sur la carte.

| Clé      | Action                            |
|----------|-----------------------------------|
| `C` | Permet de terminer tout dessin en cours et de désactiver le mode Dessin. Le focus se déplace sur l'élément de carte de niveau supérieur.  |
| `Escape` | Annule tout dessin en cours et désactive le mode Dessin. Le focus se déplace sur l'élément de carte de niveau supérieur.  |
| `F` | Ajoute des coordonnées à un point, une ligne ou un polygone si la souris se trouve sur la carte. Action équivalant à un clic sur la carte en mode Clic ou Hybride. Ce raccourci permet des dessins plus précis et plus rapides. Vous pouvez utiliser une main pour positionner la souris et l’autre pour appuyer sur le bouton sans la faire bouger sous l’effet de la pression. |

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur les classes du module Outils de dessin :

> [!div class="nextstepaction"]
> [Gestionnaire de dessins](/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)

> [!div class="nextstepaction"]
> [Barre d’outils de dessin](/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)