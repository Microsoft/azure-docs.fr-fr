---
title: Ajouter une couche de carte thermique à une carte | Microsoft Azure Maps
description: Découvrez comment créer une carte thermique. Découvrez comment utiliser le Kit de développement logiciel (SDK) web Azure Maps pour ajouter une couche thermique à une carte. Découvrez comment personnaliser les couches thermiques.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: b15ee7091a68f7fcc79c71877c4af28b511b84de
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97680137"
---
# <a name="add-a-heat-map-layer"></a>Ajouter un calque de carte thermique

Les cartes thermiques, également appelées « cartes de densité de points », sont un type de visualisation de données. Elles servent à représenter la densité de données à l’aide d’une palette de couleurs et affichent les « zones réactives » des données sur une carte. Les cartes thermiques sont un excellent moyen de restituer des jeux de données dotés d’un grand nombre de points. 

Le rendu de dizaines de milliers de points sous forme de symboles peut couvrir la plus grande partie de la surface de la carte. Dans ce cas, il est probable que de nombreux symboles se chevauchent. Il est alors difficile d’avoir une meilleure compréhension des données. Toutefois, la visualisation de ce même jeu de données sous forme de carte thermique permet de voir plus facilement où la densité ainsi que la densité relative de chaque point de données.

Vous pouvez utiliser des cartes thermiques dans de nombreux scénarios, notamment :

- **Données de température** : Fournit des approximations de la température entre deux points de données.
- **Données pour les capteurs de bruit** : Affiche non seulement l’intensité du bruit là où se trouve le capteur, mais fournit également un insight sur la dissipation du bruit avec la distance. Le niveau de bruit sur un site n’est peut-être pas élevé. Si les zones de couverture du bruit de plusieurs capteurs se chevauchent, il est possible que la zone de chevauchement obtienne des niveaux de bruit plus élevés. Ainsi, la zone de chevauchement serait visible sur la carte thermique.
- **Trace GPS** : Indique la vitesse sous forme de carte à hauteur pondérée, où l’intensité de chaque point de données est basée sur la vitesse. Par exemple, cette fonctionnalité permet de voir où un véhicule roulait plus vite.

> [!TIP]
> Les couches par défaut de la carte thermique restituent les coordonnées de toutes les données géométriques d’une source de données. Pour limiter la couche afin qu’elle n’affiche que les fonctionnalités de géométrie de point, définissez la propriété `filter` de la couche sur `['==', ['geometry-type'], 'Point']`. Si vous souhaitez également inclure des fonctionnalités MultiPoint, réglez la propriété `filter` de la couche sur `['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]`.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Heat-Maps-and-Image-Overlays-in-Azure-Maps/player?format=ny]

## <a name="add-a-heat-map-layer"></a>Ajouter un calque de carte thermique

Pour restituer une source de données de points sous forme de carte thermique, transmettez cette source à une instance de la classe `HeatMapLayer` et ajoutez-la à la carte.

Dans le code suivant, chaque point de température a un rayon de 10 pixels, à tous les niveaux de zoom. Pour garantir une meilleure expérience utilisateur, la carte thermique se trouve sous de la couche d’étiquettes. Les étiquettes restent clairement visibles. Les données de cet exemple sont issues du [USGS Earthquake Hazards Program](https://earthquake.usgs.gov/). Elles concernent les tremblements de terre importants qui se sont produits au cours des 30 derniers jours.

```javascript
//Create a data source and add it to the map.
var datasource = new atlas.source.DataSource();
map.sources.add(datasource);

//Load a dataset of points, in this case earthquake data from the USGS.
datasource.importDataFromUrl('https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.geojson');

//Create a heat map and add it to the map.
map.layers.add(new atlas.layer.HeatMapLayer(datasource, null, {
  radius: 10,
  opacity: 0.8
}), 'labels');
```

Voici l’exemple complet du code précédent.

<br/>

<iframe height='500' scrolling='no' title='Couche de carte thermique simple' src='//codepen.io/azuremaps/embed/gQqdQB/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/gQqdQB/'>Simple Heat Map Layer</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-the-heat-map-layer"></a>Personnaliser la couche de carte thermique

Dans l’exemple précédent, la carte thermique a été personnalisée à l’aide des options de rayon et d’opacité. La couche de carte thermique offre plusieurs options de personnalisation, dont :

* `radius`: définit un rayon de pixels dans lequel restituer chaque point de données. Vous pouvez définir le rayon en tant que nombre fixe ou expression. Il est possible, avec une expression, d’adapter le rayon en fonction du niveau de zoom, pour représenter une zone spatiale cohérente sur la carte (par exemple, un rayon de 5 miles).
* `color`: spécifie la façon dont la carte thermique est colorisée. Un dégradé de couleur est une caractéristique courante des cartes thermiques. Vous pouvez obtenir l’effet grâce à une expression `interpolate`. Vous pouvez également utiliser une expression `step` pour coloriser la carte thermique décompose visuellement la densité en plages. Elle ressemble ainsi davantage à une carte de style de contours ou de radars. Ces palettes de couleurs permettent de définir une couleur pour toutes les valeurs comprises entre la valeur minimale et la valeur maximale de densité. 

  Vous spécifiez les valeurs de couleurs des cartes thermiques comme expressions de valeur `heatmap-density`. La couleur de la zone dans laquelle aucune donnée n’existe est définie au niveau de l’index 0 de l’expression « Interpolation » ou par la couleur par défaut d’une expression « en étapes ». Vous pouvez utiliser cette valeur pour définir une couleur d’arrière-plan. Souvent, cette valeur est définie sur une couleur noire transparente ou semi-transparente. 
   
  Voici des exemples d’expressions de couleurs :

  | Expression d’interpolation des couleurs | Expression d’échelle de couleurs | 
  |--------------------------------|--------------------------|
  | \[<br/>&nbsp;&nbsp;&nbsp;&nbsp;'interpolate',<br/>&nbsp;&nbsp;&nbsp;&nbsp;\['linear'\],<br/>&nbsp;&nbsp;&nbsp;&nbsp;\['heatmap-density'\],<br/>&nbsp;&nbsp;&nbsp;&nbsp;0, 'transparent',<br/>&nbsp;&nbsp;&nbsp;&nbsp;0.01, 'purple',<br/>&nbsp;&nbsp;&nbsp;&nbsp;0.5, '#fb00fb',<br/>&nbsp;&nbsp;&nbsp;&nbsp;1, '#00c3ff'<br/>\] | \[<br/>&nbsp;&nbsp;&nbsp;&nbsp;'step',<br/>&nbsp;&nbsp;&nbsp;&nbsp;\['heatmap-density'\],<br/>&nbsp;&nbsp;&nbsp;&nbsp;'transparent',<br/>&nbsp;&nbsp;&nbsp;&nbsp;0.01, 'navy',<br/>&nbsp;&nbsp;&nbsp;&nbsp;0.25, 'green',<br/>&nbsp;&nbsp;&nbsp;&nbsp;0.50, 'yellow',<br/>&nbsp;&nbsp;&nbsp;&nbsp;0.75, 'red'<br/>\] |   

- `opacity`: spécifie le niveau d’opacité ou de transparence de la couche de carte thermique.
- `intensity`: applique un multiplicateur à l’épaisseur de chaque point de données pour augmenter l’intensité globale de la carte thermique. Cela différencie l’épaisseur des points de données et facilite la visualisation.
- `weight`: par défaut, tous les points de données ont une épaisseur de 1, et ont donc tous la même épaisseur. L’option d’épaisseur fait office de multiplicateur, et vous pouvez la définir en tant que nombre ou expression. Si un nombre est défini comme épaisseur, cela équivaut à placer deux fois chaque point de données sur la carte. Par exemple, si l’épaisseur est de 2, alors la densité est doublée. Le fait de définir l’option d’épaisseur sur un nombre permet d’afficher la carte thermique comme vous le feriez à l’aide d’une option d’intensité. 

  Toutefois, si vous utilisez une expression, le poids de chaque point de données peut être basé sur les propriétés de chaque point de données. Par exemple, supposons que chaque point de données représente un tremblement de terre. La valeur de magnitude est une métrique importante de chaque point de données de tremblement de terre. Des tremblements de terre se produisent constamment, mais la plupart d’entre eux ont une magnitude faible et ne sont pas remarqués. Utilisez la valeur de magnitude dans une expression pour attribuer l’épaisseur à chaque point de données. Le fait d’utiliser la valeur de magnitude pour attribuer une épaisseur vous donne une meilleure représentation de la précision des tremblements de terre sur la carte thermique.
- `source` et `source-layer` : vous permettent de mettre à jour la source de données.

Voici un outil permettant de tester les options de couche de carte thermique.

<br/>

<iframe height='700' scrolling='no' title='Options de la couche de carte thermique' src='//codepen.io/azuremaps/embed/WYPaXr/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/WYPaXr/'>Heat Map Layer Options</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="consistent-zoomable-heat-map"></a>Carte thermique homogène pouvant faire l’objet d’un zoom

Par défaut, les rayons des points de données restitués dans la couche de carte thermique ont un rayon de pixels fixe pour tous les niveaux de zoom. Lorsque vous zoomez sur la carte, les données sont agrégées, et la couche de carte thermique change d’aspect. 

Utilisez une expression `zoom` pour mettre à l’échelle le rayon de chaque niveau de zoom de sorte que chaque point de données couvre la même zone physique de la carte. Cette expression fait paraître la couche de carte thermique plus statique et plus cohérente. Chaque niveau de zoom de la carte a deux fois plus de pixels verticalement et horizontalement que le niveau de zoom précédent. 

La mise à l’échelle du rayon de sorte qu’il soit multiplié par deux avec chaque niveau de zoom crée une carte thermique qui paraît cohérente sur tous les niveaux de zoom. Pour appliquer cette mise à l’échelle, utilisez `zoom` avec une expression `exponential interpolation` de base 2, avec le rayon de pixels défini pour le niveau de zoom minimal et un rayon mis à l’échelle pour le niveau de zoom maximal calculé en tant que `2 * Math.pow(2, minZoom - maxZoom)` comme indiqué dans l’exemple suivant. Effectuez un zoom sur la carte pour voir comment la carte thermique évolue avec le niveau de zoom.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Carte thermique homogène pouvant faire l’objet d’un zoom" src="//codepen.io/azuremaps/embed/OGyMZr/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez <a href='https://codepen.io/azuremaps/pen/OGyMZr/'>Consistent zoomable heat map</a> (Carte thermique pouvant faire l’objet d’un zoom) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Lorsque vous activez l’agrégation dans la source de données, les points qui sont proches les uns des autres sont regroupés pour former un point agrégé. Vous pouvez utiliser le nombre de points de chaque cluster comme expression d’épaisseur pour la carte thermique. Cela peut réduire de manière significative le nombre de points à restituer. Le nombre de points d’un cluster est stocké dans une propriété `point_count` de la fonctionnalité de point : 
> ```JavaScript
> var layer = new atlas.layer.HeatMapLayer(datasource, null, {
>    weight: ['get', 'point_count']
> });
> ```
> Si le rayon de clustering n’est constitué que de quelques pixels, la différence visuelle sera légère. Un plus grand rayon regroupe plus de points dans chaque cluster et améliore les performances de la carte thermique.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [HeatMapLayer](/javascript/api/azure-maps-control/atlas.htmlmarker)

> [!div class="nextstepaction"]
> [HeatMapLayerOptions](/javascript/api/azure-maps-control/atlas.heatmaplayeroptions)

Pour consulter plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Créer une source de données](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Utiliser des expressions de style basées sur les données](data-driven-style-expressions-web-sdk.md)