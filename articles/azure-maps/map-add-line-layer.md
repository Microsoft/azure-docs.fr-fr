---
title: Ajouter une couche de lignes à une carte | Microsoft Azure Maps
description: Découvrez comment ajouter des lignes aux cartes. Consultez les exemples qui utilisent le Kit de développement logiciel (SDK) web Azure Maps pour ajouter des couches de lignes aux cartes et pour personnaliser les lignes avec des symboles et des dégradés de couleurs.
author: rbrundritt
ms.author: richbrun
ms.date: 08/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 9cf2f4876d62572ace118562fe5760f2b4010fa5
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891157"
---
# <a name="add-a-line-layer-to-the-map"></a>Ajouter un calque de lignes à la carte

Vous pouvez utiliser un calque de lignes pour afficher des caractéristiques `LineString` et `MultiLineString` sous forme de chemins ou d’itinéraires sur la carte. Vous pouvez aussi utiliser un calque de lignes pour afficher le contour de caractéristiques `Polygon` et `MultiPolygon`. Une source de données est connectée à une couche de lignes afin de lui fournir des données à afficher. 

> [!TIP]
> Les couches de lignes par défaut affichent les coordonnées de polygones ainsi que de lignes dans une source de données. Pour limiter la couche afin qu’elle n’affiche que les fonctionnalités LineString, définissez la propriété `filter` de la couche sur `['==', ['geometry-type'], 'LineString']` ou `['any', ['==', ['geometry-type'], 'LineString'], ['==', ['geometry-type'], 'MultiLineString']]` si vous voulez également inclure les fonctionnalités MultiLineString.

Le code suivant montre comment créer une ligne. Ajoutez la ligne à une source de données, puis affichez-la avec une couche de lignes à l’aide de la classe [LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer).

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a line and add it to the data source.
dataSource.add(new atlas.data.LineString([[-73.972340, 40.743270], [-74.004420, 40.756800]]));
  
//Create a line layer to render the line to the map.
map.layers.add(new atlas.layer.LineLayer(dataSource, null, {
    strokeColor: 'blue',
    strokeWidth: 5
}));
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus.

<br/>

<iframe height='500' scrolling='no' title='Ajouter une ligne à une carte' src='//codepen.io/azuremaps/embed/qomaKv/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez le stylet <a href='https://codepen.io/azuremaps/pen/qomaKv/'>Add a line to a map</a> (Ajouter une ligne à une carte) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Les calques de lignes peuvent être stylisés à l’aide de [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions) et [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md).

## <a name="add-symbols-along-a-line"></a>Ajouter des symboles le long d’une ligne

Cet exemple montre comment ajouter des icônes en forme de flèches le long d’une ligne sur la carte. Lorsque vous utilisez une couche de symboles, affectez à l’option « placement » la valeur « line ». Cette option affiche les symboles le long de la ligne et fait pivoter les icônes (0 degré = droite).

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Afficher une flèche le long de la ligne" src="//codepen.io/azuremaps/embed/drBJwX/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/drBJwX/'>Show arrow along line</a> (Afficher une flèche le long de la ligne) proposée par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Le kit SDK web Azure Maps fournit plusieurs modèles d’image personnalisables que vous pouvez utiliser avec le calque de symboles. Pour plus d’informations, consultez le document [Guide pratique pour utiliser des modèles d’image](how-to-use-image-templates-web-sdk.md).

<a name="line-stroke-gradient"></a>

## <a name="add-a-stroke-gradient-to-a-line"></a>Ajouter un dégradé de traits à une ligne

Vous pouvez appliquer une couleur de trait à une ligne. Vous pouvez également remplir une ligne avec un dégradé de couleurs pour représenter la transition d’un segment de ligne au segment suivant. Par exemple, les dégradés de ligne peuvent être utilisés pour représenter des changements dans le temps et la distance, ou différentes températures sur une ligne d’objets connectés. Pour appliquer cette fonctionnalité à une ligne, la source de données doit avoir l’option `lineMetrics` définie sur true. Une expression de dégradé de couleurs peut alors être transmise à l’option `strokeColor` de la ligne. L’expression de dégradé de traits a pour référence l’expression de données `['line-progress']` qui expose les métriques de ligne calculées à l’expression.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Ligne avec dégradé de traits" src="//codepen.io/azuremaps/embed/wZwWJZ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/wZwWJZ/'>Line with Stroke Gradient</a> (Ligne avec dégradé de traits) par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-line-layer"></a>Personnaliser une couche de lignes

Plusieurs options de style sont disponibles pour les couches de lignes. Voici un outil qui va vous permettre de les essayer.

<br/>

<iframe height='700' scrolling='no' title='Options de la couche de lignes' src='//codepen.io/azuremaps/embed/GwLrgb/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/GwLrgb/'>Line Layer Options</a> (Options de la couche de lignes) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [LineLayer](/javascript/api/azure-maps-control/atlas.layer.linelayer) 

> [!div class="nextstepaction"]
> [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions)

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Créer une source de données](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Ajouter une fenêtre contextuelle](map-add-popup.md)

> [!div class="nextstepaction"]
> [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Guide pratique pour utiliser des modèles d’image](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de polygones](map-add-shape.md)