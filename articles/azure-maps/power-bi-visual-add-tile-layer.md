---
title: Ajouter un calque de vignettes au visuel Power BI Azure Maps | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à utiliser le calque de vignettes dans le visuel Microsoft Azure Maps pour Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 6ab99f32932f39d5ad140b7a16d16ceae30fff54
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896206"
---
# <a name="add-a-tile-layer"></a>Ajouter un calque de vignettes

La fonctionnalité de calque de vignettes, comme la fonctionnalité de calque de référence, permet de superposer des données supplémentaires sur la carte afin de fournir davantage de contexte. Les calques de vignettes vous permettent de superposer des images à des vignettes de carte de base Azure Maps. Il s’agit d’un excellent moyen de superposer des jeux de données volumineux ou complexes, comme des images provenant de drones ou des millions de lignes de données.

> [!div class="mx-imgBorder"]
> ![Carte affichant un calque de bulles au-dessus d’un calque de vignettes montrant les données météorologiques infrarouges actuelles provenant d’Azure Maps](media/power-bi-visual/radar-tile-layer-with-bubbles.png)

Un calque de vignettes charge des vignettes à partir d’un serveur. Ces images peuvent être prérendues ou rendues dynamiquement. Les images prérendues sont stockées comme toute autre image sur un serveur à l’aide d’une convention d’affectation de noms compréhensible par la couche de mosaïques. Les images rendues dynamiquement utilisent un service pour charger les images presque en temps réel. Les calques de vignettes sont un excellent moyen de visualiser des jeux de données volumineux sur une carte. Non seulement elle peut être générée à partir d’une image, mais les données vectorielles peuvent également être affichées sous la forme d’une couche de mosaïques.

Le cadre englobant et la plage de zoom indiquant où un service de vignettes est disponible peuvent être passés en tant que paramètres pour limiter les emplacements où les vignettes sont demandées. Il s’agit d’une amélioration des performances pour le visuel et le service de vignettes. Vous trouverez ci-dessous une vue d’ensemble de tous les paramètres disponibles dans le volet **Format** et dans la section **Calque de vignettes** .

| Paramètre        | Description   |
|----------------|---------------|
| Url            | URL mise en forme pointant vers un service de vignettes.  |
| Taille de la vignette      | Valeur entière qui spécifie la largeur et la hauteur des vignettes.   |
| Limite nord    | Latitude nord du cadre englobant où les vignettes sont disponibles. |
| Limite sud    | Latitude sud du cadre englobant où les vignettes sont disponibles. |
| Limite est     | Longitude est du cadre englobant où les vignettes sont disponibles.  |
| Limite ouest     | Longitude ouest du cadre englobant où les vignettes sont disponibles.   |
| Transparence   | Transparence du calque de vignettes.   |
| Est TMS         | Tile Map Services, spécification qui inverse l’axe des ordonnées Y du calque de vignettes. |
| Zoom minimal       | Niveau de zoom minimal auquel les vignettes sont disponibles. |
| Zoom maximal       | Niveau de zoom maximal auquel les vignettes sont disponibles.  |
| Position du calque | Spécifie la position du calque par rapport aux autres calques de la carte. |

## <a name="tile-url-formatting"></a>Mise en forme de l’URL de la vignette

Trois conventions de nommage du service de vignettes sont prises en charge par le visuel Azure Maps :

-   **X, Y et notation de zoom**  : X correspond à la position de colonne et Y à la position de ligne de la vignette dans la grille de vignettes, et la valeur de la notation de zoom est basée sur le niveau de zoom.
-   **Notation Quadkey**  : combine les informations x, y et de zoom en une valeur de chaîne unique. Cette valeur de chaîne devient un identificateur unique pour une seule mosaïque.
-   **Cadre englobant**  : spécifiez une image au format de coordonnées du cadre englobant : `{west},{south},{east},{north}`. Ce format est couramment utilisé par les [services de mappage web (WMS)](https://www.opengeospatial.org/standards/wms).

URL de la vignette : URL HTTPS vers un modèle d’URL de vignette qui utilise les paramètres suivants :

-   `{x}` - position X de la mosaïque. Nécessite également `{y}` et `{z}`.
-   `{y}` - position Y de la mosaïque. Nécessite également `{x}` et `{z}`.
-   `{z}` - niveau de zoom de la mosaïque. Nécessite également `{x}` et `{y}`.
-   `{quadkey}` : identificateur `quadkey` de la vignette basé sur la convention de nommage du système de vignettes de Bing Maps.
-   `{bbox-epsg-3857}` : chaîne de cadre englobant au format `{west},{south},{east},{north}` du système SRID EPSG 3857.

En guise d’exemple, voici une URL de vignette mise en forme pour le [service de vignettes de radar météo](/rest/api/maps/renderv2/getmaptilepreview) dans Azure Maps. Notez que `[subscription-key]` est un espace réservé pour votre clé d’abonnement Azure Maps.

> `https://atlas.microsoft.com/map/tile?zoom={z}&x={x}&y={y}&tilesetId=microsoft.weather.radar.main&api-version=2.0&subscription-key=[subscription-key]`

Pour plus d’informations sur le système de mosaïques Azure Maps, consultez [Niveaux de zoom et grille mosaïque](zoom-levels-and-tile-grid.md).

## <a name="next-steps"></a>Étapes suivantes

Ajoutez davantage de contexte à la carte :

> [!div class="nextstepaction"]
> [Afficher le trafic en temps réel](power-bi-visual-show-real-time-traffic.md)