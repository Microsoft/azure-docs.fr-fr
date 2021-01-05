---
title: Modèles d’image dans le SDK web Azure Maps | Microsoft Azure Maps
description: Découvrez comment ajouter des icônes d’image et des polygones remplis de motifs à des mappages à l’aide du kit de développement logiciel (SDK) Web Azure Maps. Affichez les modèles d’image et de motif de remplissage disponibles.
author: rbrundritt
ms.author: richbrun
ms.date: 8/6/2019
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: cpendleton
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 5f455a1132a0f63e1ba3eb5d2a57a1f9bfa9a867
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895679"
---
# <a name="how-to-use-image-templates"></a>Guide pratique pour utiliser des modèles d’image

Les images peuvent être utilisées avec des marqueurs HTML et différentes couches dans le SDK web Azure Maps :

 - Les couches de symboles peuvent afficher des points sur la carte avec une icône d’image. Les symboles peuvent également être rendus le long d’un tracé de lignes.
 - Les couches de polygones peuvent être rendues avec une image de motif de remplissage. 
 - Les marqueurs HTML peuvent afficher des points à l’aide d’images et d’autres éléments HTML.

Pour garantir de bonnes performances avec les couches, chargez les images dans la ressource de sprite d’image de carte avant le rendu. Par défaut, l’interface [IconOptions](/javascript/api/azure-maps-control/atlas.iconoptions), de SymbolLayer, précharge dans le sprite d’image de carte quelques images de marqueur dans un certain nombre de couleurs. Ces images de marqueur, entre autres éléments, sont disponibles sous forme de modèles SVG. Elles peuvent être utilisées pour créer des images avec des échelles personnalisées ou être utilisées comme couleur principale et secondaire du client. Au total, 42 modèles d’images sont fournis : 27 icônes de symboles et 15 motifs de remplissage de polygones.

Il est possible d’ajouter des modèles d’image aux ressources de sprite d’image de carte à l’aide de la fonction `map.imageSprite.createFromTemplate`. Cette fonction permet de passer jusqu’à cinq paramètres :

```javascript
createFromTemplate(id: string, templateName: string, color?: string, secondaryColor?: string, scale?: number): Promise<void>
```

L’`id` est un identificateur unique que vous créez. L’`id` est attribué à l’image quand elle est ajoutée au sprite d’image de cartes. Utilisez cet identificateur dans les couches pour spécifier la ressource d’image à restituer. `templateName` spécifie le modèle d’image à utiliser. L’option `color` définit la couleur principale de l’image et les options `secondaryColor` définissent la couleur secondaire de l’image. L’option `scale` met à l’échelle le modèle d’image avant de l’appliquer au sprite d’image. Quand l’image est appliquée au sprite d’image, elle est convertie en image PNG. Pour garantir un rendu clair, il est préférable d’appliquer un scale-up au modèle d’image avant de l’ajouter au sprite plutôt que dans une couche.

Cette fonction charge l’image de façon asynchrone dans le sprite d’image. Ainsi, elle retourne une promesse que vous pouvez attendre la fin de l’exécution de cette fonction.

Le code suivant montre comment créer une image à partir de l’un des modèles intégrés, et comment l’utiliser avec une couche de symboles.

```javascript
map.imageSprite.createFromTemplate('myTemplatedIcon', 'marker-flat', 'teal', '#fff').then(function () {

    //Add a symbol layer that uses the custom created icon.
    map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
        iconOptions: {
            image: 'myTemplatedIcon'
        }
    }));
});
```

## <a name="use-an-image-template-with-a-symbol-layer"></a>Utiliser un modèle d’image avec une couche de symboles

Une fois qu’un modèle d’image a été chargé dans le sprite d’image de carte, il peut être restitué sous la forme d’un symbole dans une couche de symboles en référençant l’ID de ressource d’image dans l’option `image` de `iconOptions`.

L’exemple suivant restitue une couche de symboles à l’aide du modèle d’image `marker-flat` avec le bleu-vert comme couleur principale et le blanc comme couleur secondaire. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Couche de symboles avec un modèle d’icône intégré" src="//codepen.io/azuremaps/embed/VoQMPp/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/VoQMPp/'>Couche de symboles avec un modèle d’icône intégré</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="use-an-image-template-along-a-lines-path"></a>Utiliser un modèle d’image le long d’un tracé de lignes

Une fois qu’un modèle d’image a été chargé dans le sprite d’image de carte, il peut être restitué le long du tracé d’une ligne en ajoutant un LineString à une source de données et en utilisant une couche de symboles avec une option `lineSpacing`, puis en référençant l’ID de la ressource d’image dans l’option `image` de `iconOptions`. 

L’exemple suivant restitue une ligne rose sur la carte et utilise une couche de symboles à l’aide du modèle d’image `car` avec le bleu toile comme couleur principale et le blanc comme couleur secondaire. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Couche de lignes avec un modèle d’icône intégré" src="//codepen.io/azuremaps/embed/KOQvJe/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/KOQvJe/'>Couche de lignes avec un modèle d’icône intégré</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Si le modèle d’image pointe vers le haut, définissez l’option icône `rotation` de la couche de symboles sur 90 si vous voulez qu’il pointe dans la même direction que la ligne.

## <a name="use-an-image-template-with-a-polygon-layer"></a>Utiliser un modèle d’image avec une couche de polygones

Une fois qu’un modèle d’image a été chargé dans le sprite d’image de carte, il peut être restitué sous la forme d’un modèle de remplissage dans une couche de polygones en référençant l’ID de ressource d’image dans l’option `fillPattern` de la couche.

L’exemple suivant restitue une couche de polygones à l’aide du modèle d’image `dot` avec le rouge comme couleur principale et le transparent comme couleur secondaire.  

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Remplir un polygone avec un modèle d’icône intégré" src="//codepen.io/azuremaps/embed/WVMEmz/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/WVMEmz/'>Remplir un polygone avec un modèle d’icône intégré</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> La définition de la couleur secondaire des motifs de remplissage permet de mieux voir la carte sous-jacente tout en fournissant néanmoins le modèle principal. 

## <a name="use-an-image-template-with-an-html-marker"></a>Utiliser un modèle d’image avec un marqueur HTML

Un modèle d’image peut être récupéré à l’aide de la fonction `altas.getImageTemplate` et utilisé comme contenu d’un marqueur HTML. Le modèle peut être passé à l’option `htmlContent` du marqueur, puis personnalisé à l’aide des options `color`, `secondaryColor` et `text`.

L’exemple suivant utilise le modèle `marker-arrow` avec le rouge comme couleur principale, le rose comme couleur secondaire et la valeur de texte «00».

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Marqueur HTML avec un modèle d’icône intégré" src="//codepen.io/azuremaps/embed/EqQvzq/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/EqQvzq/'>Marqueur HTML avec un modèle d’icône intégré</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>


> [!TIP]
> Les modèles d’image peuvent aussi être utilisés en dehors de la carte. La fonction getImageTemplate retourne une chaîne SVG qui comprend des espaces réservés : `{color}`, `{secondaryColor}`, `{scale}`, `{text}`. Remplacez les valeurs de ces espaces réservés pour créer une chaîne SVG valide. Vous pouvez ajouter la chaîne SVG directement au modèle DOM HTML ou la convertir en URI de données et l’insérer dans une étiquette d’image. Par exemple :
> ```JavaScript
> //Retrieve an SVG template and replace the placeholder values.
> var svg = atlas.getImageTemplate('marker').replace(/{color}/, 'red').replace(/{secondaryColor}/, 'white').replace(/{text}/, '').replace(/{scale}/, 1);
>
> //Convert to data URI for use in image tags.
> var dataUri = 'data:image/svg+xml;base64,' + btoa(svg);
> ```

## <a name="create-custom-reusable-templates"></a>Créer des modèles réutilisables personnalisés

Si votre application utilise la même icône avec des icônes différentes ou si vous créez un module qui ajoute des modèles d’image supplémentaires, vous pouvez facilement ajouter et récupérer ces icônes à partir du SDK web Azure Maps. Utilisez les fonctions statiques suivantes sur l’espace de noms `atlas`.

| Nom | Type de retour | Description | 
|-|-|-|
| `addImageTemplate(templateName: string, template: string, override: boolean)` | | Ajoute un modèle d’image SVG personnalisé à l’espace de noms atlas. |
|  `getImageTemplate(templateName: string, scale?: number)`| string | Récupère un modèle SVG par son nom. |
| `getAllImageTemplateNames()` | string[] |  Récupère un modèle SVG par son nom. |

Les modèles d’images SVG prennent en charge les valeurs d’espace réservé suivantes :

| Espace réservé | Description |
|-|-|
| `{color}` | La couleur principale. | 
| `{secondaryColor}` | La couleur secondaire. | 
| `{scale}` | L’image SVG est convertie en image PNG quand elle est ajoutée au sprite d’image de carte. Cet espace réservé peut être utilisé pour mettre à l’échelle un modèle avant qu’il ne soit converti afin de garantir qu’il s’affiche clairement. | 
| `{text}` | L’emplacement de rendu du texte quand il est utilisé avec un marqueur HTML. |

L’exemple suivant montre comment prendre un modèle SVG et l’ajouter au SDK web Azure Maps en tant que modèle d’icône réutilisable. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Ajouter un modèle d’icône personnalisé à l’espace de noms atlas" src="//codepen.io/azuremaps/embed/NQyvEX/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/NQyvEX/'>Ajouter un modèle d’icône personnalisé à l’espace de noms atlas</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="list-of-image-templates"></a>Liste des modèles d’image

Ce tableau liste tous les modèles d’image actuellement disponibles dans le SDK web Azure Maps. Le nom de modèle se trouve au-dessus de chaque image. Par défaut, la couleur principale est le bleu et la couleur secondaire est le blanc. Pour rendre la couleur secondaire plus visible sur un fond blanc, la couleur secondaire est définie sur noir pour les images suivantes.

**Modèles d’icône de symbole**

:::row:::
   :::column span="":::
      marker
   :::column-end:::
   :::column span="":::
      marker-thick
   :::column-end:::
   :::column span="":::
      marker-circle
   :::column-end:::
   :::column span="":::
      marker-flat
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône marker](./media/image-templates/marker.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-thick](./media/image-templates/marker-thick.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-circle](./media/image-templates/marker-circle.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-flat](./media/image-templates/marker-flat.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      marker-square
   :::column-end:::
   :::column span="":::
      marker-square-cluster
   :::column-end:::
   :::column span="":::
      marker-arrow
   :::column-end:::
   :::column span="":::
      marker-ball-pin
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône marker-square](./media/image-templates/marker-square.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-square-cluster](./media/image-templates/marker-square-cluster.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-arrow](./media/image-templates/marker-arrow.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-ball-pin](./media/image-templates/marker-ball-pin.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      marker-square-rounded
   :::column-end:::
   :::column span="":::
      marker-square-rounded-cluster
   :::column-end:::
   :::column span="":::
      indicateur
   :::column-end:::
   :::column span="":::
      flag-triangle
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône marker-square-rounded](./media/image-templates/marker-square-rounded.png)
   :::column-end:::
   :::column span="":::
      ![icône marker-square-rounded-cluster](./media/image-templates/marker-square-rounded-cluster.png)
   :::column-end:::
   :::column span="":::
      ![icône flag](./media/image-templates/flag.png)
   :::column-end:::
   :::column span="":::
      ![icône flag-triangle](./media/image-templates/flag-triangle.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      triangle
   :::column-end:::
   :::column span="":::
      triangle-thick
   :::column-end:::
   :::column span="":::
      triangle-arrow-up
   :::column-end:::
   :::column span="":::
      triangle-arrow-left
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône triangle](./media/image-templates/triangle.png)
   :::column-end:::
   :::column span="":::
      ![icône triangle-thick](./media/image-templates/triangle-thick.png)
   :::column-end:::
   :::column span="":::
      ![icône triangle-arrow-up](./media/image-templates/triangle-arrow-up.png)
   :::column-end:::
   :::column span="":::
      ![icône triangle-arrow-left](./media/image-templates/triangle-arrow-left.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      hexagon
   :::column-end:::
   :::column span="":::
      hexagon-thick
   :::column-end:::
   :::column span="":::
      hexagon-rounded
   :::column-end:::
   :::column span="":::
      hexagon-rounded-thick
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône hexagon](./media/image-templates/hexagon.png)
   :::column-end:::
   :::column span="":::
      ![icône hexagon-thick](./media/image-templates/hexagon-thick.png)
   :::column-end:::
   :::column span="":::
      ![icône hexagon-rounded](./media/image-templates/hexagon-rounded.png)
   :::column-end:::
   :::column span="":::
      ![icône hexagon-rounded-thick](./media/image-templates/hexagon-rounded-thick.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      pin
   :::column-end:::
   :::column span="":::
      pin-round
   :::column-end:::
   :::column span="":::
      rounded-square
   :::column-end:::
   :::column span="":::
      rounded-square-thick
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône pin](./media/image-templates/pin.png)
   :::column-end:::
   :::column span="":::
      ![icône pin-round](./media/image-templates/pin-round.png)
   :::column-end:::
   :::column span="":::
      ![icône rounded-square](./media/image-templates/rounded-square.png)
   :::column-end:::
   :::column span="":::
      ![icône rounded-square-thick](./media/image-templates/rounded-square-thick.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      arrow-up
   :::column-end:::
   :::column span="":::
      arrow-up-thin
   :::column-end:::
   :::column span="":::
      voiture
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône arrow-up](./media/image-templates/arrow-up.png)
   :::column-end:::
   :::column span="":::
      ![icône arrow-up-thin](./media/image-templates/arrow-up-thin.png)
   :::column-end:::
   :::column span="":::
      ![icône car](./media/image-templates/car.png)
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::


**Modèles de motif de remplissage de polygones**

:::row:::
   :::column span="":::
      checker
   :::column-end:::
   :::column span="":::
      checker-rotated
   :::column-end:::
   :::column span="":::
      circles
   :::column-end:::
   :::column span="":::
      circles-spaced
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône checker](./media/image-templates/checker.png)
   :::column-end:::
   :::column span="":::
      ![icône checker-rotated](./media/image-templates/checker-rotated.png)
   :::column-end:::
   :::column span="":::
      ![icône circles](./media/image-templates/circles.png)
   :::column-end:::
   :::column span="":::
      ![icône circles-spaced](./media/image-templates/circles-spaced.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      diagonal-lines-up
   :::column-end:::
   :::column span="":::
      diagonal-lines-down
   :::column-end:::
   :::column span="":::
      diagonal-stripes-up
   :::column-end:::
   :::column span="":::
      diagonal-stripes-down
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône diagonal-lines-up](./media/image-templates/diagonal-lines-up.png)
   :::column-end:::
   :::column span="":::
      ![icône diagonal-lines-down](./media/image-templates/diagonal-lines-down.png)
   :::column-end:::
   :::column span="":::
      ![icône diagonal-stripes-up](./media/image-templates/diagonal-stripes-up.png)
   :::column-end:::
   :::column span="":::
      ![icône diagonal-stripes-down](./media/image-templates/diagonal-stripes-down.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      grid-lines
   :::column-end:::
   :::column span="":::
      rotated-grid-lines
   :::column-end:::
   :::column span="":::
      rotated-grid-stripes
   :::column-end:::
   :::column span="":::
      x-fill
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône grid-lines](./media/image-templates/grid-lines.png)
   :::column-end:::
   :::column span="":::
      ![icône rotated-grid-lines](./media/image-templates/rotated-grid-lines.png)
   :::column-end:::
   :::column span="":::
      ![icône rotated-grid-stripes](./media/image-templates/rotated-grid-stripes.png)
   :::column-end:::
   :::column span="":::
      ![icône x-fill](./media/image-templates/x-fill.png)
   :::column-end:::
:::row-end:::
<br>

:::row:::
   :::column span="":::
      zig-zag
   :::column-end:::
   :::column span="":::
      zig-zag-vertical
   :::column-end:::
   :::column span="":::
      dots
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      ![icône zig-zag](./media/image-templates/zig-zag.png)
   :::column-end:::
   :::column span="":::
      ![icône zig-zag-vertical](./media/image-templates/zig-zag-vertical.png)
   :::column-end:::
   :::column span="":::
      ![icône dots](./media/image-templates/dots.png)
   :::column-end:::
   :::column span="":::
      &nbsp;
   :::column-end:::
:::row-end:::
<br>

**Icônes d’images préchargées**

La carte précharge un ensemble d’icônes dans le sprite d’image de la carte en utilisant les modèles `marker`, `pin` et `pin-round`. Le nom de ces icônes et leur valeur de couleur sont listés dans le tableau ci-dessous.

| nom d’icône | couleur | couleur secondaire |
|-----------|-------|----------------|
| `marker-black` | `#231f20` | `#ffffff` |
| `marker-blue` | `#1a73aa` | `#ffffff` |
| `marker-darkblue` | `#003963` | `#ffffff` |
| `marker-red` | `#ef4c4c` | `#ffffff` |
| `marker-yellow` | `#f2c851` | `#ffffff` |
| `pin-blue` | `#2072b8` | `#ffffff` |
| `pin-darkblue` | `#003963` | `#ffffff` |
| `pin-red` | `#ef4c4c` | `#ffffff` |
| `pin-round-blue` | `#2072b8` | `#ffffff` |
| `pin-round-darkblue` | `#003963` | `#ffffff` |
| `pin-round-red` | `#ef4c4c` | `#ffffff` |


## <a name="try-it-now-tool"></a>Outil Essayer maintenant

Avec l’outil suivant, vous pouvez restituer les différents modèles d’image intégrés de différentes façons et personnaliser l’échelle et les couleurs principales et secondaires.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Options des modèles d’icône" src="//codepen.io/azuremaps/embed/NQyaaO/?height=500&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page Pen <a href='https://codepen.io/azuremaps/pen/NQyaaO/'>Options des modèles d’icône</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur le site <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [ImageSpriteManager](/javascript/api/azure-maps-control/atlas.imagespritemanager)

> [!div class="nextstepaction"]
> [Espace de noms atlas](/javascript/api/azure-maps-control/atlas#functions
)

Consultez les articles suivants pour obtenir d’autres exemples de code dans lesquels des modèles d’image peuvent être utilisés :

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](map-add-pin.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de lignes](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de polygones](map-add-shape.md)

> [!div class="nextstepaction"]
> [Ajouter des marqueurs HTML](map-add-bubble-layer.md)