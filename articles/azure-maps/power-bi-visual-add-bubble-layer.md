---
title: Ajouter un calque de bulles au visuel Power BI Azure Maps | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à utiliser le calque de bulles dans le visuel Microsoft Azure Maps pour Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: ff0d2c8e90995222af79d5be63ca3e9da24b3c45
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896223"
---
# <a name="add-a-bubble-layer"></a>Ajouter un calque de bulles

Le **calque de bulles**  affiche les données d’emplacement sous forme de cercles à l’échelle sur la carte.

> [!div class="mx-imgBorder"]
> ![Carte affichant des données de point à l’aide du calque de bulles](media/power-bi-visual/bubble-layer-with-legend-color.png)

Initialement, toutes les bulles ont la même couleur de remplissage. Si un champ est transmis au compartiment **Légende** du volet **Champs** , les bulles sont colorées en fonction de leur catégorisation. Le contour des bulles est blanc par défaut, mais vous pouvez lui donner une nouvelle couleur ou activer l’option de contour à contraste élevé. L’option **bordure à contraste élevé** attribue dynamiquement une couleur de contour qui est une variante à contraste élevé de la couleur de remplissage. Cela permet de s’assurer que les bulles sont clairement visibles quel que soit le style de la carte. Vous trouverez ci-dessous les principaux paramètres du volet **Format** disponibles dans la section **Calque de bulles** .

| Paramètre               | Description    |
|-----------------------|----------------|
| Taille                  | Taille de chaque bulle. Cette option est masquée quand un champ est transmis dans le compartiment **Taille** du volet **Champs** . Des options supplémentaires s’affichent comme exposé dans la rubrique [Mise à l'échelle de la taille des bulles](#bubble-size-scaling) plus loin dans cet article. |
| Couleur de remplissage            | Couleur de chaque bulle. Cette option est masquée lorsqu’un champ est transmis dans le compartiment **Légende** du volet **Champs** , et une section **Couleurs des données** distincte apparaît dans le volet **Format** . |
| Transparence du remplissage     | Transparence de chaque bulle. |
| Contour à contraste élevé | Fait en sorte que la couleur du contour contraste avec la couleur de remplissage pour une meilleure accessibilité à l’aide d’une variante à contraste élevé de la couleur de remplissage. |
| Couleur du contour         | Couleur de contour de la bulle. Cette option est masquée lorsque l’option **Contour à contraste élevé** est activée. |
| Transparence du contour  | Transparence du contour. |
| Largeur du contour         | Largeur du contour en pixels. |
| Flou                  | Quantité de flou appliquée au contour. La valeur 1 floute les bulles de telle sorte que seul le point central n’a pas de transparence. La valeur 0 applique un effet de flou. |
| Alignement lors de l’inclinaison       | Spécifie l’alignement des bulles lorsque la carte est inclinée. <br/><br/>&nbsp;&nbsp;&nbsp;&nbsp;• Fenêtre d’affichage : les bulles apparaissent sur leur bord sur la carte par rapport à la fenêtre d’affichage. (par défaut)<br/>&nbsp;&nbsp;&nbsp;&nbsp;• Map : les bulles sont rendues à plat sur la surface de la carte. |
| Échelle de zoom            | Valeur de mise à l'échelle des bulles par rapport au niveau de zoom. Une échelle de zoom de un signifie qu’il n’y a pas de mise à l’échelle. Si les valeurs sont élevées, les bulles sont plus petites en cas de zoom arrière et plus grandes en cas de zoom avant. Cela permet de réduire l’encombrement sur la carte en cas de zoom arrière, tout en garantissant que les points ressortent davantage en cas de zoom avant. La valeur 1 n’applique pas de mise à l’échelle. |
| Zoom minimal              | Niveau de zoom minimal auquel les vignettes sont disponibles. |
| Zoom maximal              | Niveau de zoom maximal auquel les vignettes sont disponibles. |
| Position du calque        | Spécifie la position du calque par rapport aux autres calques de la carte. |

## <a name="bubble-size-scaling"></a>Mise à l’échelle de la taille des bulles

Si un champ est transmis au compartiment **Taille** du volet **Champs** , les bulles sont mises à l’échelle par rapport à la valeur de mesure de chaque point de données. L’option **Taille** dans la section **Calque de bulle** du volet **Format** disparaît lorsqu’un champ est transmis au compartiment **Taille** , car les rayons des bulles sont mis à l’échelle entre une valeur minimale et une valeur maximale. Les options suivantes s’affichent dans la section **Calque de bulles** du volet **Format** quand un compartiment **Taille** a un champ spécifié.

| Paramètre             | Description  |
|---------------------|--------------|
| Taille minimale            | Taille de bulle minimale lors de la mise à l’échelle des données.|
| Taille maximale            | Taille de bulle maximale lors de la mise à l’échelle des données.|
| Méthode de mise à l'échelle de la taille | Algorithme de mise à l’échelle utilisé pour déterminer la taille de bulle relative.<br/><br/>&nbsp;&nbsp;&nbsp;&nbsp;• Linéaire : plage de données d’entrée mappées de façon linéaire à la taille minimale et maximale. (par défaut)<br/>&nbsp;&nbsp;&nbsp;&nbsp;• Log : plage de données d’entrée mappées de façon logarithmique à la taille minimale et maximale.<br/>&nbsp;&nbsp;&nbsp;&nbsp;• Bezier cubique : spécifiez les valeurs X1, Y1, X2 et Y2 d’une courbe de Bézier cubique pour créer une méthode de mise à l'échelle personnalisée. |

Lorsque la **méthode de mise à l’échelle de la taille** est définie sur **Log** , les options suivantes sont disponibles.

| Paramètre   | Description      |
|-----------|------------------|
| Échelle logarithmique | Échelle logarithmique à appliquer lors du calcul de la taille des bulles. |

Lorsque la **méthode de mise à l’échelle de la taille** est définie sur **Bezier cubique** , les options suivantes sont disponibles pour personnaliser la courbe de mise à l'échelle.

| Paramètre | Description                           |
|---------|---------------------------------------|
| X1      | Paramètre X1 d’une courbe de Bézier cubique. |
| Y1      | Paramètre X2 d’une courbe de Bézier cubique. |
| X2      | Paramètre Y1 d’une courbe de Bézier cubique. |
| Y2      | Paramètre Y2 d’une courbe de Bézier cubique. |

> [!TIP]
> [https://cubic-bezier.com/](https://cubic-bezier.com/) dispose d’un outil pratique pour créer les paramètres des courbes de Bézier cubiques.

## <a name="next-steps"></a>Étapes suivantes

Modifier la façon dont vos données sont affichées sur la carte :

> [!div class="nextstepaction"]
> [Ajouter un calque de graphique à barres](power-bi-visual-add-bar-chart-layer.md)

Ajouter davantage de contexte à la carte :

> [!div class="nextstepaction"]
> [Ajouter un calque de référence](power-bi-visual-add-reference-layer.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de mosaïques](power-bi-visual-add-tile-layer.md)

> [!div class="nextstepaction"]
> [Afficher le trafic en temps réel](power-bi-visual-show-real-time-traffic.md)

Personnaliser le visuel :

> [!div class="nextstepaction"]
> [Astuces et conseils pour la mise en forme des couleurs dans Power BI](/power-bi/visuals/service-tips-and-tricks-for-color-formatting)

> [!div class="nextstepaction"]
> [Personnaliser les titres, les légendes et les arrière-plans des visualisations](/power-bi/visuals/power-bi-visualization-customize-title-background-and-legend)