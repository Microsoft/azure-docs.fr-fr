---
title: Clustering de données de point sur une carte | Microsoft Azure Maps
description: Découvrez comment mettre en cluster des données de point sur des cartes. Découvrez comment utiliser le Kit de développement logiciel (SDK) web Azure Maps pour mettre en cluster des données, réagir aux événements de souris en cluster et afficher les agrégats de cluster.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: codepen, devx-track-js
ms.openlocfilehash: e80465cf8d43918e6ed6da8ebb3b96f3f197e887
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97679952"
---
# <a name="clustering-point-data"></a>Clustering de données de point

Lors de la visualisation de nombreux points de données sur la carte, les points peuvent se chevaucher. Le chevauchement peut rendre la carte illisible et difficile à utiliser. Le clustering de point de données est le processus permettant de combiner des données de point proches les unes des autres et de les représenter sur une carte en tant qu’un point de données en cluster unique. Lorsque l’utilisateur effectue un zoom avant sur la carte, les clusters se décomposent pour afficher les points de données individuels qui les composent. Quand vous travaillez avec un grand nombre de points de données, utilisez les processus de clustering pour améliorer l’expérience de vos utilisateurs.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Clustering-point-data-in-Azure-Maps/player?format=ny]

## <a name="enabling-clustering-on-a-data-source"></a>Activer le clustering sur une source de données

Activez le clustering dans la classe `DataSource` en définissant l’option `cluster` sur true. Définissez `ClusterRadius` pour qu’il sélectionne des points à proximité et les combine en un cluster. La valeur de `ClusterRadius` est exprimée en pixels. Utilisez `clusterMaxZoom` pour spécifier un niveau de zoom auquel désactiver la logique de clustering. Voici un exemple montrant comment activer le clustering dans une source de données.

```javascript
//Create a data source and enable clustering.
var datasource = new atlas.source.DataSource(null, {
    //Tell the data source to cluster point data.
    cluster: true,

    //The radius in pixels to cluster points together.
    clusterRadius: 45,

    //The maximum zoom level in which clustering occurs.
    //If you zoom in more than this, all points are rendered as symbols.
    clusterMaxZoom: 15
});
```

> [!TIP]
> Si deux points de données sont proches l’un de l’autre sur le sol, il est possible que le cluster ne se décompose jamais, quelle que soit l’importance du zoom avant effectué par l’utilisateur. Pour résoudre ce problème, vous pouvez définir l’option `clusterMaxZoom` de façon à désactiver la logique de clustering et à simplement afficher tous les éléments.

Voici des méthodes supplémentaires fournies par la classe `DataSource` pour le clustering :

| Méthode | Type de retour | Description |
|--------|-------------|-------------|
| getClusterChildren(clusterId: number) | Promise&lt;Array&lt;Feature&lt;Geometry, any&gt; \| Shape&gt;&gt; | Récupère les enfants du cluster donné sur le niveau de zoom suivant. Ces enfants peuvent être une combinaison de formes et sous-clusters. Les sous-clusters seront des fonctionnalités dont les propriétés correspondent à ClusteredProperties. |
| getClusterExpansionZoom(clusterId: number) | Promise&lt;number&gt; | Calcule un niveau de zoom à partir duquel le cluster commence à se développer ou à se décomposer. |
| getClusterLeaves(clusterId: number, limit: number, offset: number) | Promise&lt;Array&lt;Feature&lt;Geometry, any&gt; \| Shape&gt;&gt; | Récupère tous les points dans un cluster. Définissez le paramètre `limit` de manière à renvoyer un sous-ensemble des points et utilisez `offset` pour parcourir les points. |

## <a name="display-clusters-using-a-bubble-layer"></a>Afficher les clusters à l’aide d’une couche de bulles

Un calque de bulles est un bon moyen de rendre des points de données regroupés en cluster. Utilisez des expressions pour mettre à l’échelle le rayon et changer la couleur en fonction du nombre de points du cluster. Si vous affichez des clusters sous la forme d’un calque de bulles, vous devez également utiliser un calque distinct pour rendre les points de données non regroupés dans le cluster.

Pour afficher la taille du cluster en haut de la bulle, utilisez un calque de symboles avec du texte, et n’utilisez pas une icône.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Clustering en couche de bulles de base" src="//codepen.io/azuremaps/embed/qvzRZY/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le rendu <a href='https://codepen.io/azuremaps/pen/qvzRZY/'>Clustering en couche de bulles de base</a> Pen par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="display-clusters-using-a-symbol-layer"></a>Afficher les clusters à l’aide d’une couche de symboles

Lors de la visualisation des points de données, le calque de symboles masque automatiquement les symboles qui se chevauchent pour garantir une interface utilisateur plus propre. Ce comportement par défaut peut être indésirable si vous voulez montrer la densité des points de données sur la carte. Ces paramètres peuvent cependant être modifiés. Pour afficher tous les symboles, définissez l’option `allowOverlap` de la propriété `iconOptions` des calques de symboles sur `true`. 

Utilisez le clustering pour montrer la densité des points de données tout en conservant une interface utilisateur propre. L’exemple ci-dessous montre comment ajouter des symboles personnalisés, et comment représenter des clusters et des points de données individuels en utilisant le calque de symboles.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Couche de symboles en cluster" src="//codepen.io/azuremaps/embed/Wmqpzz/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le rendu <a href='https://codepen.io/azuremaps/pen/Wmqpzz/'>Couche de symboles en cluster</a> Pen par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="clustering-and-the-heat-maps-layer"></a>Clustering et couche de cartes thermiques

Les cartes thermiques sont un excellent moyen pour afficher la densité des données sur la carte. Cette méthode de visualisation peut gérer par elle-même un grand nombre de points de données. Si les points de données sont en cluster et que la taille du cluster est utilisée comme pondération de la carte thermique, la carte thermique peut traiter encore plus de données. Pour cela, définissez l’option `weight` du calque de la carte thermique sur `['get', 'point_count']`. Quand le rayon du cluster est faible, la carte thermique ressemblera fortement à une carte thermique utilisant des points de données qui ne sont pas mis en cluster, tout en étant beaucoup plus performante. Cependant, plus le rayon du cluster est petit, plus la carte thermique est précise, mais avec moins de bénéfices en termes de performances.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Carte thermique pondérée en fonction du cluster" src="//codepen.io/azuremaps/embed/VRJrgO/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le rendu <a href='https://codepen.io/azuremaps/pen/VRJrgO/'>Carte thermique pondérée en fonction du cluster</a> Pen par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="mouse-events-on-clustered-data-points"></a>Événements de souris sur des points de données en cluster

Quand des événements de souris se produisent sur un calque contenant des points de données en cluster, le point de données en cluster derniers est retourné à l’événement en tant qu’objet de fonctionnalité de point GeoJSON. Cette fonctionnalité de point possède les propriétés suivantes :

| Nom de la propriété             | Type    | Description   |
|---------------------------|---------|---------------|
| `cluster`                 | boolean | Indique si la fonctionnalité représente un cluster. |
| `cluster_id`              | string  | Un ID unique pour le cluster qui peut être utilisé avec les méthodes `getClusterExpansionZoom`, `getClusterChildren` et `getClusterLeaves` de DataSource. |
| `point_count`             | nombre  | Le nombre de points que contient le cluster.  |
| `point_count_abbreviated` | string  | Une chaîne qui abrège la valeur de `point_count` si elle est trop longue. (par exemple, 4 000 devient 4K)  |

Cet exemple prend un calque de bulles qui affiche des points de cluster et ajoute un événement de clic. Quand l’événement de clic est déclenché, le code calcule et effectue un zoom de la carte au niveau de zoom suivant, à partir duquel le cluster se décompose. Cette fonctionnalité est implémentée avec la méthode `getClusterExpansionZoom` de la classe `DataSource` et la propriété `cluster_id` du point de données en cluster sur lequel l’utilisateur a cliqué.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Méthode getClusterExpansionZoom du cluster" src="//codepen.io/azuremaps/embed/moZWeV/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le rendu <a href='https://codepen.io/azuremaps/pen/moZWeV/'>Méthode getClusterExpansionZoom du cluster</a> Pen par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="display-cluster-area"></a>Afficher la zone du cluster 

Les données de point représentées par un cluster sont réparties sur une zone. Dans cet exemple, quand la souris passe sur un cluster, deux comportements principaux se produisent. D’abord, les points de données individuels contenus dans le cluster sont utilisés pour calculer une enveloppe convexe. Ensuite, l’enveloppe convexe est affichée sur la carte pour montrer une zone.  Une enveloppe convexe est un polygone qui entoure un ensemble de points comme un élastique et qui peut être calculé à l’aide de la méthode `atlas.math.getConvexHull`. Tous les points contenus dans un cluster peuvent être récupérés à partir de la source de données à l’aide de la méthode `getClusterLeaves`.

<br/>

 <iframe height="500" style="width: 100%;" scrolling="no" title="Enveloppe convexe de la zone du cluster" src="//codepen.io/azuremaps/embed/QoXqWJ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le rendu <a href='https://codepen.io/azuremaps/pen/QoXqWJ/'>Enveloppe convexe de la zone du cluster</a> Pen par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="aggregating-data-in-clusters"></a>Agrégation de données dans des clusters

Souvent, les clusters sont représentés à l’aide d’un symbole avec le nombre de points qui se trouvent dans le cluster. Cependant, il est parfois souhaitable de pouvoir personnaliser le style des clusters avec des métriques supplémentaires. Avec les agrégats de cluster, les propriétés personnalisées peuvent être créées et remplies en utilisant un calcul [d’expression d’agrégation](data-driven-style-expressions-web-sdk.md#aggregate-expression).  Les agrégats de cluster peuvent être définis dans l'option `clusterProperties` de `DataSource`.

L’exemple suivant utilise une expression d’agrégation. Le code calcule un nombre basé sur la propriété du type d’entité de chaque point de données dans un cluster. Quand un utilisateur clique sur un cluster, une fenêtre contextuelle s’affiche avec des informations supplémentaires sur le cluster.

<iframe height="500" style="width: 100%;" scrolling="no" title="Agrégats de cluster" src="//codepen.io/azuremaps/embed/jgYyRL/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez le <a href='https://codepen.io/azuremaps/pen/jgYyRL/'>Agrégats de cluster</a> Pen par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Classe DataSource](/javascript/api/azure-maps-control/atlas.source.datasource)

> [!div class="nextstepaction"]
> [Objet DataSourceOptions](/javascript/api/azure-maps-control/atlas.datasourceoptions)

> [!div class="nextstepaction"]
> [Espace de noms atlas.math](/javascript/api/azure-maps-control/atlas.math)

Consultez les exemples de code pour ajouter la fonctionnalité à votre application :

> [!div class="nextstepaction"]
> [Ajouter une couche de bulles](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](map-add-pin.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de carte thermique](map-add-heat-map-layer.md)