---
title: Ajouter une couche de symboles à une carte | Microsoft Azure Maps
description: En savoir plus sur l’ajout de symboles personnalisés, tels que du texte ou des icônes, aux cartes. Voir comment utiliser les sources de données et les couches de symboles dans le Kit de développement logiciel (SDK) web Azure Maps à cette fin.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: c5434406af1f912c1c89123baa344dd3f9c7ff91
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891052"
---
# <a name="add-a-symbol-layer-to-a-map"></a>Ajouter une couche de symboles à une carte

Connectez un symbole à une source de données et utilisez-le pour afficher une icône ou un texte à un point donné. 

Les couches de symboles sont affichées à l’aide de WebGL. Utilisez une couche de symboles pour afficher de grandes collections de points sur la carte. Par rapport au marqueur HTML, le calque de symboles affiche un grand nombre de données de point sur la carte de façon plus performante. Toutefois, la couche de symboles ne prend pas en charge les éléments CSS et HTML traditionnels pour le style.  

> [!TIP]
> Les couches de symboles par défaut affichent les coordonnées de toutes les données géométriques d’une source de données. Pour limiter le calque afin qu’il n’affiche que les caractéristiques de géométrie de point, définissez la propriété `filter` de la couche sur `['==', ['geometry-type'], 'Point']` ou `['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]`. Si vous le voulez, vous pouvez également inclure des caractéristiques MultiPoint.

Le gestionnaire de sprites d’images de cartes charge les images personnalisées que la couche de symboles utilise. Il prend en charge les formats d’image suivants :

- JPEG
- PNG
- SVG
- BMP
- GIF (sans animation)

## <a name="add-a-symbol-layer"></a>Ajouter un calque de symboles

Avant de pouvoir ajouter une couche de symboles à la carte, vous devez effectuer quelques étapes. Commencez par créer une source de données et l’ajouter à la carte. Créez une couche de symboles. Ensuite, transmettez la source de données à la couche de symboles pour récupérer les données de la source de données. Enfin, ajoutez des données à la source de données, afin qu’il y ait quelque chose à afficher. 

Le code ci-dessous montre ce qui doit être ajouté à la carte après son chargement. Cet exemple restitue un point unique sur la carte à l’aide d’une couche de symboles. 

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a symbol layer to render icons and/or text at points on the map.
var layer = new atlas.layer.SymbolLayer(dataSource);

//Add the layer to the map.
map.layers.add(layer);

//Create a point and add it to the data source.
dataSource.add(new atlas.data.Point([0, 0]));
```

Quatre types de données de point peuvent être ajoutés à la carte :

- Géométrie des points GeoJSON - cet objet contient uniquement une coordonnée d’un point et rien d’autre. La classe d’assistance `atlas.data.Point` peut être utilisée pour créer facilement ces objets.
- Géométrie MultiPoint GeoJSON - cet objet contient les coordonnées de plusieurs points et rien d’autre. La classe d’assistance `atlas.data.MultiPoint` peut être utilisée pour créer facilement ces objets.
- Fonctionnalité GeoJSON - cet objet se compose d’une géométrie GeoJSON et d’un ensemble de propriétés qui contiennent des métadonnées associées à la géométrie. La classe d’assistance `atlas.data.Feature` peut être utilisée pour créer facilement ces objets.
- La classe `atlas.Shape` est similaire à la caractéristique GeoJSON. Toutes deux consistent en une géométrie GeoJSON et un ensemble de propriétés contenant des métadonnées associées à la géométrie. En cas d’ajout d’un objet GeoJSON à une source de données, celui-ci peut être facilement affiché dans une couche. Toutefois, si la propriété des coordonnées de cet objet GeoJSON est mise à jour, la source de données et la carte ne changent pas. Cela est dû au fait qu’il n’existe aucun mécanisme dans l’objet JSON pour déclencher une mise à jour. La classe Shape fournit des fonctions permettant de mettre à jour les données qu’elle contient. Lors de l’apport d’une modification, la source de données et la carte sont automatiquement notifiées et mises à jour. 

L’exemple de code suivant crée une géométrie de points GeoJSON et la transmet dans la classe `atlas.Shape` pour faciliter sa mise à jour. Le centre de la carte est initialement utilisé pour afficher un symbole. Un événement de clic est ajouté à la carte, qui a pour effet, lorsqu’il se déclenche, que les coordonnées de la souris sont utilisées avec la fonction `setCoordinates` des formes. Les coordonnées de la souris sont enregistrées lors du déclenchement de l’événement de clic. Ensuite, la fonction `setCoordinates` met à jour l’emplacement du symbole sur la carte.

<br/>

<iframe height='500' scrolling='no' title='Switch pin location' src='//codepen.io/azuremaps/embed/ZqJjRP/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/ZqJjRP/'>Switch pin location</a> (Changer l’emplacement du repère) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Par défaut, les couches de symboles optimisent l’affichage des symboles en masquant ceux qui se chevauchent. Quand vous effectuez un zoom avant, les symboles masqués deviennent visibles. Pour désactiver cette fonctionnalité et afficher tous les symboles en permanence, définissez la propriété `allowOverlap` des options `iconOptions` sur `true`.

## <a name="add-a-custom-icon-to-a-symbol-layer"></a>Ajouter une icône personnalisée à une couche de symboles

Les couches de symboles sont affichées à l’aide de WebGL. Toutes les ressources, telles que les images d’icône, doivent donc être chargées dans le contexte de WebGL. Cet exemple montre comment ajouter une icône personnalisée aux ressources de carte. Cette icône est ensuite utilisée pour afficher des données de point avec un symbole personnalisé sur la carte. La propriété `textField` de la couche de symboles nécessite une expression. Dans ce cas, nous souhaitons afficher la propriété de température. La température étant un nombre, elle doit être convertie en chaîne. De plus, nous voulons y ajouter la mention « °F ». Pour effectuer cette concaténation, nous pouvons utiliser une expression : `['concat', ['to-string', ['get', 'temperature']], '°F']`. 

<br/>

<iframe height='500' scrolling='no' title='Icône de l’image de symbole personnalisée' src='//codepen.io/azuremaps/embed/WYWRWZ/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/WYWRWZ/'>Custom Symbol Image Icon</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Le kit SDK web Azure Maps fournit plusieurs modèles d’image personnalisables que vous pouvez utiliser avec le calque de symboles. Pour plus d’informations, consultez le document [Guide pratique pour utiliser des modèles d’image](how-to-use-image-templates-web-sdk.md).

## <a name="customize-a-symbol-layer"></a>Personnaliser une couche de symboles 

Il y a de nombreuses options de style disponibles pour la couche de symboles. Voici un outil permettant de les tester.

<br/>

<iframe height='700' scrolling='no' title='Options de la couche de symboles' src='//codepen.io/azuremaps/embed/PxVXje/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/PxVXje/'>Symbol Layer Options</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Si vous souhaitez afficher uniquement le texte avec une couche de symboles, vous pouvez masquer l’icône en définissant la propriété `image` des options d’icône sur `'none'`.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [SymbolLayer](/javascript/api/azure-maps-control/atlas.layer.symbollayer)

> [!div class="nextstepaction"]
> [SymbolLayerOptions](/javascript/api/azure-maps-control/atlas.symbollayeroptions)

> [!div class="nextstepaction"]
> [IconOptions](/javascript/api/azure-maps-control/atlas.iconoptions)

> [!div class="nextstepaction"]
> [TextOptions](/javascript/api/azure-maps-control/atlas.textoptions)

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
> [Ajouter une couche de lignes](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de polygones](map-add-shape.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de bulles](map-add-bubble-layer.md)

> [!div class="nextstepaction"]
> [Ajouter des marqueurs HTML](map-add-bubble-layer.md)