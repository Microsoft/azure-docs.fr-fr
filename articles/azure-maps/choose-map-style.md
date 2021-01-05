---
title: Modifier le style du contrôle de carte web Azure Maps
description: Découvrez comment modifier le style et les options d’une carte. Découvrez comment ajouter un contrôle sélecteur de styles à un mappage dans Azure Maps afin que les utilisateurs puissent basculer entre différents styles.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 556e265cc0d1aae33823185ec98d23f191ed1694
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97680018"
---
# <a name="change-the-style-of-the-map"></a>Changer le style de la carte

Le contrôle de carte prend en charge différentes [options de style](/javascript/api/azure-maps-control/atlas.styleoptions) de carte et plusieurs [styles de carte de base](supported-map-styles.md). Tous les styles peuvent être définis lors de l’initialisation du contrôle de carte. Vous pouvez également définir des styles à l’aide de la fonction `setStyle` du contrôle de carte. Cet article explique comment utiliser ces options de style pour personnaliser l’apparence de la carte. En outre, vous apprendrez à implémenter le contrôle du sélecteur de styles dans votre carte. Le contrôle du sélecteur de styles permet à l’utilisateur de basculer entre différents styles de base.

## <a name="set-map-style-options"></a>Définir les options de style de carte

Les options de style peuvent être définies lors de l’initialisation du contrôle web. Vous pouvez également mettre à jour les options de style en appelant la fonction `setStyle` du contrôle de carte. Pour afficher toutes les options de style disponibles, consultez [Options de style](/javascript/api/azure-maps-control/atlas.styleoptions).

```javascript
//Set the style options when creating the map.
var map = new atlas.Map('map', {
    renderWorldCopies: false,
    showBuildingModels: false,
    showLogo: true,
    showFeedbackLink: true,
    style: 'road'

    //Additional map options.
};

//Update the style options at anytime using `setStyle` function.
map.setStyle({
    renderWorldCopies: true,
    showBuildingModels: true,
    showLogo: false,
    showFeedbackLink: false
});
```

L’outil suivant explique la manière dont les différentes options de style modifient l'affichage de la carte. Pour afficher les bâtiments 3D, effectuez un zoom avant à proximité d'une agglomération.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Options de style de carte" src="https://codepen.io/azuremaps/embed/eYNMjPb?height=700&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait <a href='https://codepen.io/azuremaps/pen/eYNMjPb'>Options de style de carte</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="set-a-base-map-style"></a>Définir un style de carte de base

Vous pouvez également initialiser le contrôle de carte avec l’un des [styles de carte de base](supported-map-styles.md) disponibles dans le Kit de développement logiciel (SDK) web. Vous pouvez ensuite utiliser la fonction `setStyle` pour mettre à jour le style de base avec un style de carte différent.

### <a name="set-a-base-map-style-on-initialization"></a>Définir un style de carte de base lors de l’initialisation

Les styles de base du contrôle de carte peuvent être définis lors de l’initialisation. Dans le code suivant, l’option `style` du contrôle de carte est définie sur le [style de carte de base `grayscale_dark`](supported-map-styles.md#grayscale_dark).  

```javascript
var map = new atlas.Map('map', {
    style: 'grayscale_dark',

    //Additional map options
);
```

<br/>

<iframe height='500' scrolling='no' title='Définition du style lors du chargement de carte' src='//codepen.io/azuremaps/embed/WKOQRq/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Voir la page <a href='https://codepen.io/azuremaps/pen/WKOQRq/'>Définition du style lors du chargement de carte</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="update-the-base-map-style"></a>Mettre à jour le style de carte de base

Le style de la carte de base peut être mis à jour à l’aide de la fonction `setStyle` et en définissant l’option `style` pour changer le style de la carte de base ou pour ajouter des options de style supplémentaires.

```javascript
map.setStyle({ style: 'satellite' });
```

Dans le code suivant, après le chargement d’une instance de carte, le style de carte est mis à jour de `grayscale_dark` en `satellite` à l’aide de la fonction [setStyle](/javascript/api/azure-maps-control/atlas.map#setstyle-styleoptions-).

<br/>

<iframe height='500' scrolling='no' title='Mise à jour du style' src='//codepen.io/azuremaps/embed/yqXYzY/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Voir la page <a href='https://codepen.io/azuremaps/pen/yqXYzY/'>Mise à jour du style</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-the-style-picker-control"></a>Ajouter le contrôle du sélecteur de styles

Le contrôle du sélecteur de styles fournit un bouton facile à utiliser doté d’un panneau volant que l’utilisateur final peut se servir pour passer d’un style de base à un autre.

Le sélecteur de styles propose deux options de disposition distinctes : `icon` et `list`. En outre, le sélecteur de styles vous permet de choisir deux options `style` différentes pour le contrôle : `light` et `dark`. Dans cet exemple, le sélecteur de styles utilise la disposition `icon` et affiche une liste sélective de styles de carte de base sous forme d’icônes. Le contrôle du sélecteur de styles comprend l’ensemble de styles de base suivant : `["road", "grayscale_light", "grayscale_dark", "night", "road_shaded_relief"]`. Pour plus d’informations sur les options de contrôle du sélecteur de styles, consultez [Options de contrôle de style](/javascript/api/azure-maps-control/atlas.stylecontroloptions).

L’image ci-dessous montre le contrôle du sélecteur de styles avec la disposition `icon`.

:::image type="content" source="./media/choose-map-style/style-picker-icon-layout.png" alt-text="Disposition des icônes du sélecteur de styles":::

L’image ci-dessous montre le contrôle du sélecteur de styles avec la disposition `list`.

:::image type="content" source="./media/choose-map-style/style-picker-list-layout.png" alt-text="Disposition de la liste du sélecteur de styles":::

> [!IMPORTANT]
> Par défaut, le contrôle du sélecteur de styles répertorie tous les styles disponibles sous le niveau tarifaire S0 d’Azure Maps. Si vous voulez réduire le nombre de styles figurant dans cette liste, passez dans l’option `mapStyle` du sélecteur de styles un tableau des styles que vous voulez voir apparaître dans la liste. Si vous utilisez S1 et que vous voulez afficher tous les styles disponibles, affectez à l’option `mapStyles` du sélecteur de styles la valeur `"all"`.

Le code suivant montre comment remplacer la liste de styles de base `mapStyles` par défaut. Dans cet exemple, nous définissons l’option `mapStyles` pour répertorier les styles de base que le contrôle du sélecteur de styles doit afficher.

<br/>

<iframe height='500' scrolling='no' title='Ajout du sélecteur de styles' src='//codepen.io/azuremaps/embed/OwgyvG/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Voir la page <a href='https://codepen.io/azuremaps/pen/OwgyvG/'>Ajout du sélecteur de styles</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [StyleOptions](/javascript/api/azure-maps-control/atlas.styleoptions)

> [!div class="nextstepaction"]
> [StyleControl](/javascript/api/azure-maps-control/atlas.control.stylecontrol)

> [!div class="nextstepaction"]
> [StyleControlOptions](/javascript/api/azure-maps-control/atlas.stylecontroloptions)

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Ajouter des contrôles de carte](map-add-controls.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](map-add-pin.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de bulles](map-add-bubble-layer.md)
