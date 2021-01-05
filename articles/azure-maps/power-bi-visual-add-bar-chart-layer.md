---
title: Ajouter une couche de graphique à barres au visuel Power BI Azure Maps | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à utiliser la couche de graphique à barres dans le visuel Microsoft Azure Maps pour Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 136676b46df6e32f98dca99fccba19d4aa369dbe
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896274"
---
# <a name="add-a-bar-chart-layer"></a>Ajouter une couche de graphique à barres

La **couche de graphique à barres** se révèle utile pour visualiser des données d’emplacement sous forme de barres ou de cylindres 3D sur la carte. À l’instar de la couche de bulles, le graphique à barres permet de facilement visualiser deux métriques en même temps à l’aide de la couleur et de la hauteur relative. En termes de hauteur des barres, une mesure doit être ajoutée au compartiment **Taille** du volet **Champs** . Si aucune mesure n’est fournie, des barres sans hauteur apparaissent sous forme de carrés ou de cercles plats en fonction de l’option **Forme de la barre** .

> [!div class="mx-imgBorder"]
> ![Carte affichant des données de point à l’aide de la couche de graphique à barres](media/power-bi-visual/bar-chart-layer-styled.png)

Les utilisateurs peuvent incliner et faire pivoter la carte pour afficher vos données selon différentes perspectives. La carte peut être inclinée à l’aide d’une des méthodes suivantes.

-   Activez l’option **Contrôles de navigation** dans les **paramètres de la carte** du volet **Format** . Cela permet d’ajouter un bouton pour incliner la carte.
-   Appuyez sur le bouton droit de la souris et faites glisser la souris vers le haut ou vers le bas.
-   À l’aide d’un écran tactile, touchez la carte avec deux doigts et faites-les glisser ensemble vers le haut ou vers le bas.
-   Lorsque le focus est placé sur la carte, maintenez la touche **Maj** enfoncée et appuyez sur les touches de direction **Haut** ou **Bas** .

La carte peut pivoter en utilisant l'une des méthodes suivantes.

-   Activez l’option **Contrôles de navigation** dans les **paramètres de la carte** du volet **Format** . Cela permet d’ajouter un bouton pour faire pivoter la carte.
-   Appuyez sur le bouton droit de la souris et faites glisser la souris vers la gauche ou vers la droite.
-   À l’aide d’un écran tactile, touchez la carte avec deux doigts et effectuez une rotation.
-   Lorsque le focus est placé sur la carte, maintenez la touche **Maj** enfoncée et appuyez sur les touches de direction **Gauche** ou **Droite** .

Vous trouverez ci-dessous tous les paramètres du **Format** disponibles dans la section **Couche de graphique à barres** .

| Paramètre              | Description      |
|----------------------|------------------|
| Forme de la barre            | Forme de la barre 3D.<br/><br/>&nbsp;&nbsp;&nbsp;&nbsp;• Zone : barres affichées sous forme de zones rectangulaires.<br/>&nbsp;&nbsp;&nbsp;&nbsp;• Cylindre : barres affichées sous forme de cylindres. |
| Hauteur               | Hauteur de chaque barre. Si un champ est transmis dans le compartiment **Taille** du volet **Champs** , les barres sont mises à l’échelle par rapport à cette valeur de hauteur. |
| Mettre à l'échelle la hauteur pendant le zoom | Spécifie si la hauteur des barres doit être mise à l'échelle par rapport au niveau de zoom. |
| Largeur                | Largeur de chaque barre.  |
| Mettre à l’échelle la largeur pendant le zoom  | Spécifie si la largeur des barres doit être mise à l'échelle par rapport au niveau de zoom.  |
| Couleur de remplissage           | Couleur de chaque barre. Cette option est masquée lorsqu’un champ est transmis dans le compartiment **Légende** du volet **Champs** , et une section **Couleurs des données** distincte apparaît dans le volet **Format** . |
| Transparence         | Transparence de chaque barre. |
| Zoom minimal             | Niveau de zoom minimal auquel les vignettes sont disponibles. |
| Zoom maximal             | Niveau de zoom maximal auquel les vignettes sont disponibles. |
| Position du calque       | Spécifie la position du calque par rapport aux autres calques de la carte. |

> [!NOTE]
> Si les barres présentent une petite valeur de largeur et que l’option **Mettre à l’échelle la largeur pendant le zoom** est désactivée, elles peuvent disparaître en cas de zoom arrière important, car leur largeur affichée est inférieure à un pixel en termes de taille. Cependant, quand l’option **Mettre à l’échelle la largeur pendant le zoom** est activée, des calculs supplémentaires sont effectués lorsque le niveau de zoom évolue, ce qui peut avoir un impact sur les performances des grands jeux de données.

## <a name="next-steps"></a>Étapes suivantes

Ajouter davantage de contexte à la carte :

> [!div class="nextstepaction"]
> [Ajouter une couche de référence](power-bi-visual-add-reference-layer.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de mosaïques](power-bi-visual-add-tile-layer.md)

> [!div class="nextstepaction"]
> [Afficher le trafic en temps réel](power-bi-visual-show-real-time-traffic.md)

Personnaliser le visuel :

> [!div class="nextstepaction"]
> [Astuces et conseils pour la mise en forme des couleurs dans Power BI](/power-bi/visuals/service-tips-and-tricks-for-color-formatting)

> [!div class="nextstepaction"]
> [Personnaliser les titres, les légendes et les arrière-plans des visualisations](/power-bi/visuals/power-bi-visualization-customize-title-background-and-legend)