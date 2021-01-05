---
title: Ajouter une couche de données simple | Microsoft Azure Maps
description: Découvrez comment ajouter une couche de données simple à l’aide du module d’E/S spatiales, fourni par le Kit de développement logiciel (SDK) web Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/29/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 90f3cb0ae44be176d3ae248988d098039c140c3e
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896155"
---
# <a name="add-a-simple-data-layer"></a>Ajouter une couche de données simple

Le module d’E/S spatiales fournit une classe `SimpleDataLayer`. Cette classe facilite le rendu des fonctionnalités stylisées sur la carte. Elle peut même afficher des jeux de données qui ont des propriétés de style et des jeux de données qui contiennent des types de géométrie mixtes. La couche de données simple permet d’obtenir cette fonctionnalité en enveloppant plusieurs couches de rendu et en utilisant des expressions de style. Les expressions de style recherchent les propriétés de style communes des fonctionnalités à l’intérieur de ces couches enveloppées. La fonction `atlas.io.read` et la fonction `atlas.io.write` utilisent ces propriétés pour lire et écrire des styles dans un format de fichier pris en charge. Après avoir ajouté les propriétés à un format de fichier pris en charge, le fichier peut être utilisé à différentes fins. Par exemple, le fichier peut être utilisé pour afficher les fonctionnalités stylisées sur la carte.

Outre les fonctionnalités de style, `SimpleDataLayer` fournit une fonctionnalité contextuelle intégrée avec un modèle de fenêtre contextuelle. La fenêtre contextuelle s’affiche lorsqu’un utilisateur clique sur une fonctionnalité. La fonctionnalité contextuelle par défaut peut être désactivée, si vous le souhaitez. Cette couche prend également en charge les données en cluster. Lorsqu’un utilisateur clique sur un cluster, la carte effectue un zoom avant sur le cluster et le développe en points individuels et sous-clusters.

La classe `SimpleDataLayer` est destinée à être utilisée sur des jeux de données volumineux avec de nombreux types de géométrie et de nombreux styles appliqués aux fonctionnalités. En cas d’utilisation, cette classe ajoute une surcharge de six couches contenant des expressions de style. Dans certains cas, il est plus efficace d’utiliser les couches de rendu principales. Par exemple, utilisez une couche de base pour restituer quelques types de géométrie et quelques styles sur une fonctionnalité.

## <a name="use-a-simple-data-layer"></a>Utiliser une couche de données simple

La classe `SimpleDataLayer` est utilisée comme les autres couches de rendu. Le code ci-dessous montre comment utiliser une couche de données simple dans une carte :

```javascript
//Create a data source and add it to the map.
var datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Add a simple data layer for rendering data.
var layer = new atlas.layer.SimpleDataLayer(datasource);
map.layers.add(layer);
```

Ajoutez des fonctionnalités à la source de données. Ensuite, la couche de données simple déterminera la meilleure façon d’afficher les fonctionnalités. Les styles des fonctionnalités individuelles peuvent être définis en tant que propriétés sur la fonctionnalité. Le code suivant montre une fonctionnalité de point GeoJSON avec une propriété `color` définie sur `red`. 

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [0, 0]
    },
    "properties": {
        "color": "red"
    }
}
```

Le code suivant restitue la fonctionnalité de point ci-dessus à l’aide de la couche de données simple. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Utiliser la couche de données simple" src="//codepen.io/azuremaps/embed/zYGzpQV/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true"> Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/zYGzpQV/'>Use the simple data layer</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Le véritable pouvoir de la couche de données simple quand :

- il existe différents types de fonctionnalités dans une source de données ;
- plusieurs propriétés de style sont définies individuellement sur les fonctionnalités du jeu de données ; ou
- vous ne savez pas exactement ce que contient le jeu de données.

Par exemple, lors de l’analyse de flux de données XML, vous pouvez ne pas connaître les styles et les types de géométrie exacts des fonctionnalités. L’exemple suivant illustre la puissance de la couche de données simple en restituant les fonctionnalités d’un fichier KML. Il illustre également les différentes options fournies par la classe de la couche de données simple.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Options de la couche de données simple" src="//codepen.io/azuremaps/embed/gOpRXgy/?height=700&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true"> Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/gOpRXgy/'>Simple data layer options</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


> [!NOTE]
> Cette couche de données simple utilise la classe [modèle de fenêtre contextuelle](map-add-popup.md#add-popup-templates-to-the-map) pour afficher des bulles KML ou des propriétés de fonctionnalité sous forme de tableau. Par défaut, tout le contenu affiché dans la fenêtre contextuelle est placé dans un bac à sable (sandbox) à l’intérieur d’un IFrame en tant que fonctionnalité de sécurité. Toutefois, des limitations s’appliquent :
>
> - Les scripts, les formulaires, le verrou de pointeur et la fonctionnalité de navigation supérieure sont désactivés. Les liens sont autorisés à s’ouvrir dans un nouvel onglet en cas de clic. 
> - Les navigateurs plus anciens qui ne prennent pas en charge le paramètre `srcdoc` sur Iframes sont limités au rendu d’une petite quantité de contenu.
> 
> Si vous avez confiance dans les données chargées dans les fenêtres contextuelles et souhaitez potentiellement que ces scripts chargés dans les fenêtres contextuelles puissent accéder à votre application, vous pouvez désactiver ce comportement en définissant l’option `sandboxContent` des modèles de fenêtre contextuelle sur la valeur False. 

## <a name="default-supported-style-properties"></a>Propriétés de style prises en charge par défaut

Comme indiqué précédemment, la couche de données simple enveloppe plusieurs des couches de rendu principales : bulle, symbole, ligne, polygone et polygone extrudé. Elle utilise ensuite des expressions pour rechercher des propriétés de style valides sur des fonctionnalités individuelles.

Les propriétés de style Azure Maps et GitHub sont les deux principaux ensembles de noms de propriétés pris en charge. La plupart des noms de propriété des différentes options de couche Azure Maps sont pris en charge en tant que propriétés de style des fonctionnalités dans la couche de données simple. Des expressions ont été ajoutées à certaines options de couche pour prendre en charge les noms de propriété de style couramment utilisés par GitHub. Ces noms de propriétés sont définis par le [support cartographique GeoJSON de GitHub](https://help.github.com/en/github/managing-files-in-a-repository/mapping-geojson-files-on-github), et ils sont utilisés pour styliser les fichiers GeoJSON qui sont stockés et rendus au sein de la plateforme. Toutes les propriétés de style de GitHub sont prises en charge dans la couche de données simple, à l’exception des propriétés de style `marker-symbol`.

Si le lecteur rencontre une propriété de style moins courante, il la convertit dans la propriété de style Azure Maps la plus proche. En outre, les expressions de style par défaut peuvent être remplacées à l’aide de la fonction `getLayers` de la couche de données simple et de la mise à jour des options sur l’une des couches.

La section suivante fournit des détails sur les propriétés de style par défaut qui sont prises en charge par la couche de données simple. L’ordre du nom de propriété pris en charge représente également la priorité de la propriété. Si deux propriétés de style sont définies pour la même option de couche, la première dans la liste a une priorité plus élevée. Les couleurs peuvent être n’importe quelle valeur de couleur CSS3 ; HEX, RGB, RVBA, TSL, HSLA ou la valeur de couleur nommée.

### <a name="bubble-layer-style-properties"></a>Propriétés de style de la couche de bulles

Si une fonctionnalité est un `Point` ou un `MultiPoint`, et que la fonctionnalité n’a pas de propriété `image` qui serait utilisée comme icône personnalisée pour afficher le point comme un symbole, la fonctionnalité sera rendue avec une `BubbleLayer`.

| Option de couche | Noms de propriété pris en charge | Valeur par défaut |
|--------------|----------------------------|---------------|
| `color` | `color`, `marker-color` | `'#1A73AA'` |
| `radius` | `size`<sup>1</sup>, `marker-size`<sup>2</sup>, `scale`<sup>1</sup> | `8` |
| `strokeColor` | `strokeColor`, `stroke` | `'#FFFFFF'` |

\[1\] Les valeurs `size` et `scale` sont considérées comme des valeurs scalaires et sont multipliées par `8`.

\[2\] Si l’option `marker-size` de GitHub est spécifiée, les valeurs suivantes seront utilisées pour le rayon.

| Taille de marqueur | Radius |
|-------------|--------|
| `small`     | `6`    |
| `medium`    | `8`    |
| `large`     | `12`   |

Les clusters sont également rendus à l’aide de la couche de bulles. Par défaut, le rayon d’un cluster est défini sur `16`. La couleur du cluster varie selon le nombre de points dans le cluster, comme indiqué ci-dessous :

| Nombre de points | Couleur    |
|-------------|----------|
| &gt;= 100   | `red`    |
| &gt;= 10    | `yellow` |
| &lt;= 10     | `green`  |

### <a name="symbol-style-properties"></a>Propriétés de style de symbole

Si une fonctionnalité est un `Point` ou un `MultiPoint`, et que la fonctionnalité n’a pas de propriété `image` qui serait utilisée comme icône personnalisée pour afficher le point comme un symbole, la fonctionnalité sera rendue avec une `SymbolLayer`.

| Option de couche | Noms de propriété pris en charge | Valeur par défaut |
|--------------|----------------------------|---------------|
| `image` | `image` | ``none`` |
| `size` | `size`, `marker-size`<sup>1</sup> | `1` |
| `rotation` | `rotation` | `0` |
| `offset` | `offset` | `[0, 0]` |
| `anchor` | `anchor` | `'bottom'` |

\[1\] Si l’option `marker-size` de GitHub est spécifiée, les valeurs suivantes seront utilisées pour l’option de taille de l’icône.

| Taille de marqueur | Taille du symbole |
|-------------|-------------|
| `small`     | `0.5`       |
| `medium`    | `1`         |
| `large`     | `2`         |

Si la fonctionnalité Point est un cluster, la propriété `point_count_abbreviated` est rendue sous la forme d’une étiquette de texte. Aucune image ne sera affichée.

### <a name="line-style-properties"></a>Propriétés de style de ligne

Si la fonctionnalité est un `LineString`, `MultiLineString`, `Polygon` ou `MultiPolygon`, la fonctionnalité sera rendue avec une `LineLayer`.

| Option de couche | Noms de propriété pris en charge | Valeur par défaut |
|--------------|----------------------------|---------------|
| `strokeColor` | `strokeColor`, `stroke` | `'#1E90FF'` |
| `strokeWidth` | `strokeWidth`, `stroke-width`, `stroke-thickness` | `3` |
| `strokeOpacity` | `strokeOpacity`, `stroke-opacity` | `1` |

### <a name="polygon-style-properties"></a>Propriétés de style de polygone

Si la fonctionnalité est un `Polygon` ou un `MultiPolygon`, et si la fonctionnalité n’a pas de propriété `height` ou si la propriété `height` est égale à zéro, la fonctionnalité sera rendue avec une `PolygonLayer`.

| Option de couche | Noms de propriété pris en charge | Valeur par défaut |
|--------------|----------------------------|---------------|
| `fillColor` | `fillColor`, `fill` | `'#1E90FF'` |
| `fillOpacity` | `fillOpacity`, '`fill-opacity` | `0.5` |

### <a name="extruded-polygon-style-properties"></a>Propriétés de style de polygone extrudé

Si la fonctionnalité est un `Polygon` ou un `MultiPolygon` et qu’elle a une propriété `height` dont la valeur est supérieure à 0, la fonctionnalité sera rendue avec une `PolygonExtrusionLayer`.

| Option de couche | Noms de propriété pris en charge | Valeur par défaut |
|--------------|----------------------------|---------------|
| `base` | `base` | `0` |
| `fillColor` | `fillColor`, `fill` | `'#1E90FF'` |
| `height` | `height` | `0` |

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [SimpleDataLayer](/javascript/api/azure-maps-spatial-io/atlas.layer.simpledatalayer)

> [!div class="nextstepaction"]
> [SimpleDataLayerOptions](/javascript/api/azure-maps-spatial-io/atlas.simpledatalayeroptions)

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Lire et écrire des données spatiales](spatial-io-read-write-spatial-data.md)

> [!div class="nextstepaction"]
> [Ajouter une couche OGC](spatial-io-add-ogc-map-layer.md)

> [!div class="nextstepaction"]
> [Se connecter à un service WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Exploiter les opérations de base](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Détails sur les formats de données pris en charge](spatial-io-supported-data-format-details.md)