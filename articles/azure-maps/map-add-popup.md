---
title: Ajouter une fenêtre contextuelle à un point sur une carte | Microsoft Azure Maps
description: En savoir plus sur les fenêtres contextuelles, les modèles de fenêtres contextuelles et les événements contextuels dans Azure Maps. Découvrez comment ajouter une fenêtre contextuelle à un point sur une carte et comment réutiliser et personnaliser des fenêtres contextuelles.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: 903574a8c8696f10d9903a685bab8d12b3e6395f
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891034"
---
# <a name="add-a-popup-to-the-map"></a>Ajouter une fenêtre contextuelle à la carte

Cet article vous explique comment ajouter une fenêtre contextuelle à un point sur une carte.

## <a name="understand-the-code"></a>Comprendre le code

Le code suivant ajoute une caractéristique de point, qui a des propriétés `name` et `description`, à la carte à l’aide d’un calque de symboles. Une instance de la [classe Popup](/javascript/api/azure-maps-control/atlas.popup) est créée, mais elle n’est pas affichée. Les événements de souris sont ajoutés au calque de symboles pour déclencher l’ouverture et la fermeture de la fenêtre contextuelle. Quand vous survolez le symbole de marqueur, la propriété `position` de la fenêtre contextuelle est mise à jour avec la position du marqueur et l’option `content` est mise à jour avec du code HTML qui encapsule les propriétés `name` et `description` de la caractéristique de point survolée. La fenêtre contextuelle est ensuite affichée sur la carte à l’aide de sa fonction `open`.

```javascript
//Define an HTML template for a custom popup content laypout.
var popupTemplate = '<div class="customInfobox"><div class="name">{name}</div>{description}</div>';

//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

dataSource.add(new atlas.data.Feature(new atlas.data.Point([-122.1333, 47.63]), {
  name: 'Microsoft Building 41', 
  description: '15571 NE 31st St, Redmond, WA 98052'
}));

//Create a layer to render point data.
var symbolLayer = new atlas.layer.SymbolLayer(dataSource);

//Add the polygon and line the symbol layer to the map.
map.layers.add(symbolLayer);

//Create a popup but leave it closed so we can update it and display it later.
popup = new atlas.Popup({
  pixelOffset: [0, -18],
  closeButton: false
});

//Add a hover event to the symbol layer.
map.events.add('mouseover', symbolLayer, function (e) {
  //Make sure that the point exists.
  if (e.shapes && e.shapes.length > 0) {
    var content, coordinate;
    var properties = e.shapes[0].getProperties();
    content = popupTemplate.replace(/{name}/g, properties.name).replace(/{description}/g, properties.description);
    coordinate = e.shapes[0].getCoordinates();

    popup.setOptions({
      //Update the content of the popup.
      content: content,

      //Update the popup's position with the symbol's coordinate.
      position: coordinate

    });
    //Open the popup.
    popup.open(map);
  }
});

map.events.add('mouseleave', symbolLayer, function (){
  popup.close();
});
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus.

<br/>

<iframe height='500' scrolling='no' title='Ajouter une fenêtre contextuelle à l’aide d’Azure Maps' src='//codepen.io/azuremaps/embed/MPRPvz/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Affichez l’objet Pen <a href='https://codepen.io/azuremaps/pen/MPRPvz/'>Ajouter une fenêtre contextuelle à l’aide d’Azure Maps</a> par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="reusing-a-popup-with-multiple-points"></a>Réutilisation d’une fenêtre contextuelle avec plusieurs points

Dans certains cas, la meilleure approche consiste à créer une fenêtre contextuelle et à la réutiliser. Par exemple, vous pouvez avoir un grand nombre de points et souhaiter afficher une seule fenêtre contextuelle à la fois. En réutilisant la fenêtre contextuelle, le nombre d’éléments DOM créés par l’application est nettement réduit, ce qui peut offrir un meilleur niveau de performance. L’exemple suivant crée des caractéristiques à trois points. Si vous cliquez sur l’un d’eux, une fenêtre contextuelle s’affiche avec le contenu de ce point.

<br/>

<iframe height='500' scrolling='no' title='Réutilisation d’une fenêtre contextuelle avec plusieurs épingles' src='//codepen.io/azuremaps/embed/rQbjvK/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/rQbjvK/'>Reusing Popup with Multiple Pin</a> (Réutilisation d’une fenêtre contextuelle avec plusieurs épingles) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customizing-a-popup"></a>Personnalisation d’une fenêtre contextuelle

Par défaut, la fenêtre contextuelle a un arrière-plan blanc, une flèche de pointeur au bas et un bouton de fermeture dans le coin supérieur droit. L’exemple suivant change la couleur d’arrière-plan en noir à l’aide de l’option `fillColor` de la fenêtre contextuelle. Le bouton Fermer est supprimé en affectant la valeur false à l’option `CloseButton`. Le contenu HTML de la fenêtre contextuelle utilise un remplissage de 10 pixels par rapport aux bords de la fenêtre contextuelle. Le texte de couleur blanche s’affiche correctement sur l’arrière-plan noir.  

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Fenêtre contextuelle personnalisée" src="//codepen.io/azuremaps/embed/ymKgdg/?height=500&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page <a href='https://codepen.io/azuremaps/pen/ymKgdg/'>Customized Popup</a> (Fenêtre contextuelle personnalisée) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-popup-templates-to-the-map"></a>Ajouter des modèles de fenêtre contextuelle à la carte

Les modèles de fenêtre contextuelle facilitent la création de dispositions axées sur les données dans les fenêtres contextuelles. Les sections ci-dessous montrent comment différents modèles de fenêtre contextuelle sont utilisés pour générer du contenu mis en forme à l’aide des propriétés de fonctionnalités.

> [!NOTE]
> Par défaut, tout le contenu affiché dans le modèle de fenêtre contextuelle est placé dans un bac à sable (sandbox) à l’intérieur d’un IFrame en tant que fonctionnalité de sécurité. Toutefois, des limitations s’appliquent :
>
> - Les scripts, les formulaires, le verrou de pointeur et la fonctionnalité de navigation supérieure sont désactivés. Les liens sont autorisés à s’ouvrir dans un nouvel onglet en cas de clic. 
> - Les navigateurs plus anciens qui ne prennent pas en charge le paramètre `srcdoc` sur Iframes sont limités au rendu d’une petite quantité de contenu.
> 
> Si vous avez confiance dans les données chargées dans les fenêtres contextuelles et souhaitez potentiellement que ces scripts chargés dans les fenêtres contextuelles puissent accéder à votre application, vous pouvez désactiver ce comportement en définissant l’option `sandboxContent` des modèles de fenêtre contextuelle sur la valeur False. 

### <a name="string-template"></a>Modèle String

Le modèle String remplace les espaces réservés par les valeurs des propriétés de la fonctionnalité. Il n’est pas nécessaire d’attribuer des valeurs de type String aux propriétés de la fonctionnalité. Par exemple, `value1` contient un entier. Ces valeurs sont ensuite transmises à la propriété content de `popupTemplate`. 

L’option `numberFormat` spécifie le format du nombre à afficher. Si `numberFormat` n’est pas spécifié, le code utilise le format de date du modèle de fenêtre contextuelle. L’option `numberFormat` met en forme les nombres à l’aide de la fonction [Number.toLocaleString](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString). Pour mettre en forme les nombres importants, envisagez d’utiliser l’option `numberFormat` avec les fonctions de [NumberFormat.format](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat/format). Par exemple, l’extrait de code ci-dessous utilise `maximumFractionDigits` pour limiter le nombre de chiffres de fraction à deux.

> [!Note]
> Le modèle String ne peut afficher les images que d’une seule façon. Pour commencer, le modèle String doit contenir une balise d’image. La valeur transmise à la balise d’image doit être une URL qui mène à une image. Ensuite, le paramètre `isImage` de l’option `HyperLinkFormatOptions` du modèle String doit être défini sur true. L’option `isImage` indique que le lien hypertexte se rapporte à une image et qu’il sera chargé dans une balise d’image. L’image s’ouvrira une fois que l’utilisateur aura cliqué sur le lien hypertexte.

```javascript
var templateOptions = {
  content: 'This template uses a string template with placeholders.<br/><br/> - Value 1 = {value1}<br/> - Value 2 = {value2/subValue}<br/> - Array value [2] = {arrayValue/2}',
  numberFormat: {
    maximumFractionDigits: 2
  }
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 1 - String template',
    value1: 1.2345678,
    value2: {
        subValue: 'Pizza'
    },
    arrayValue: [3, 4, 5, 6]
});

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="propertyinfo-template"></a>Modèle PropertyInfo

Le modèle PropertyInfo affiche les propriétés disponibles de la fonctionnalité. L’option `label` indique le texte à afficher pour l’utilisateur. Si `label` n’est pas spécifié, le lien hypertexte s’affiche. Par ailleurs, si le lien hypertexte est une image, la valeur affectée à la balise « alt » s’affiche. `dateFormat` indique le format de la date. Si aucun format de date n’est spécifié, la date s’affiche sous forme de chaîne. L’option `hyperlinkFormat` affiche les liens cliquables. De la même manière, l’option `email` peut être utilisée pour afficher les adresses e-mail cliquables.

Avant de présenter les propriétés à l’utilisateur final, le modèle PropertyInfo vérifie de manière récursive que les propriétés sont bel et bien définies pour cette fonctionnalité. Par ailleurs, il n’affiche pas les propriétés de style et title. Par exemple, les propriétés `color`, `size`, `anchor`, `strokeOpacity` et `visibility` ne sont pas affichées. Ainsi, une fois la vérification du chemin des propriétés terminée en arrière-plan, le modèle PropertyInfo affiche le contenu sous forme de tableau.

```javascript
var templateOptions = {
  content: [
    {
        propertyPath: 'createDate',
        label: 'Created Date'
    },
    {
        propertyPath: 'dateNumber',
        label: 'Formatted date from number',
        dateFormat: {
          weekday: 'long',
          year: 'numeric',
          month: 'long',
          day: 'numeric',
          timeZone: 'UTC',
          timeZoneName: 'short'
        }
    },
    {
        propertyPath: 'url',
        label: 'Code samples',
        hideLabel: true,
        hyperlinkFormat: {
          lable: 'Go to code samples!',
          target: '_blank'
        }
    },
    {
        propertyPath: 'email',
        label: 'Email us',
        hideLabel: true,
        hyperlinkFormat: {
          target: '_blank',
          scheme: 'mailto:'
        }
    }
  ]
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 2 - PropertyInfo',
    createDate: new Date(),
    dateNumber: 1569880860542,
    url: 'https://aka.ms/AzureMapsSamples',
    email: 'info@microsoft.com'
}),

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="multiple-content-templates"></a>Modèles multicontenus

Une fonctionnalité peut aussi afficher du contenu en combinant le modèle String et le modèle PropertyInfo. Dans ce cas, le modèle String affiche les valeurs des espaces réservés sur un arrière-plan blanc.  Parallèlement, le modèle PropertyInfo affiche une image en pleine largeur à l’intérieur d’un tableau. Dans cet exemple, les propriétés sont similaires à celles que nous avons expliquées dans les exemples précédents.

```javascript
var templateOptions = {
  content: [
    'This template has two pieces of content; a string template with placeholders and a array of property info which renders a full width image.<br/><br/> - Value 1 = {value1}<br/> - Value 2 = {value2/subValue}<br/> - Array value [2] = {arrayValue/2}',
    [{
      propertyPath: 'imageLink',
      label: 'Image',
      hideImageLabel: true,
      hyperlinkFormat: {
        isImage: true
      }
    }]
  ],
  numberFormat: {
    maximumFractionDigits: 2
  }
};

var feature = new atlas.data.Feature(new atlas.data.Point([0, 0]), {
    title: 'Template 3 - Multiple content template',
    value1: 1.2345678,
    value2: {
    subValue: 'Pizza'
    },
    arrayValue: [3, 4, 5, 6],
    imageLink: 'https://azuremapscodesamples.azurewebsites.net/common/images/Pike_Market.jpg'
});

var popup = new atlas.Popup({
  content: atlas.PopupTemplate.applyTemplate(feature.properties, templateOptions),
  position: feature.geometry.coordinates
});
```

### <a name="points-without-a-defined-template"></a>Points sans modèle défini

Quand le modèle de fenêtre contextuelle (Popup) n’est pas défini comme étant un modèle String, un modèle PropertyInfo ou une combinaison des deux, il utilise les paramètres par défaut. Quand `title` et `description` sont les seules propriétés affectées, le modèle de fenêtre contextuelle présente un arrière-plan blanc et un bouton de fermeture dans l’angle supérieur droit. De plus, sur les écrans de petit et moyen format, il affiche une flèche au bas. Les paramètres par défaut s’affichent à l’intérieur d’un tableau pour toutes les propriétés autres que `title` et `description`. Même en revenant aux paramètres par défaut, le modèle de fenêtre contextuelle peut toujours être manipulé par programmation. Par exemple, les utilisateurs peuvent désactiver la détection des liens hypertexte, mais les paramètres par défaut s’appliqueront toujours aux autres propriétés.

Cliquez sur les points sur la carte dans CodePen. Il existe sur la carte un point pour chacun des modèles de fenêtre contextuelle suivants : modèle String, modèle PropertyInfo et modèle multicontenu. Il y a aussi trois points qui montrent comment les modèles s’affichent avec les paramètres par défaut.

<br/>

<iframe height='500' scrolling='no' title='PopupTemplates' src='//codepen.io/azuremaps/embed/dyovrzL/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez le stylet <a href='https://codepen.io/azuremaps/pen/dyovrzL/'>PopupTemplates</a> par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="reuse-popup-template"></a>Réutiliser le modèle de fenêtre contextuelle

Vous pouvez réutiliser les modèles de fenêtre contextuelle comme vous réutiliseriez une fenêtre contextuelle. Cette approche s’avère utile quand vous voulez seulement afficher un modèle de fenêtre contextuelle à la fois, pour plusieurs points. En réutilisant le modèle de fenêtre contextuelle, le nombre d’éléments DOM créés par l’application est réduit, ce qui améliore le niveau de performance de votre application. L’exemple suivant utilise le même modèle de fenêtre contextuelle pour les trois points. Si vous cliquez sur l’un d’eux, une fenêtre contextuelle s’affiche avec le contenu de ce point.

<br/>

<iframe height='500' scrolling='no' title='ReusePopupTemplate' src='//codepen.io/azuremaps/embed/WNvjxGw/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez le stylet <a href='https://codepen.io/azuremaps/pen/WNvjxGw/'>ReusePopupTemplates</a> par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="popup-events"></a>Événements de fenêtres contextuelles

Les fenêtres contextuelles peuvent être ouvertes, fermées et déplacées. La classe popup fournit des événements pour aider les développeurs à réagir à ces événements. L’exemple suivant met en évidence les événements qui se déclenchent lorsque l’utilisateur ouvre, ferme ou fait glisser la fenêtre contextuelle. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Événements de fenêtres contextuelles" src="//codepen.io/azuremaps/embed/BXrpvB/?height=500&theme-id=0&default-tab=result" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez la page <a href='https://codepen.io/azuremaps/pen/BXrpvB/'>Popup events</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Popup](/javascript/api/azure-maps-control/atlas.popup)

> [!div class="nextstepaction"]
> [PopupOptions](/javascript/api/azure-maps-control/atlas.popupoptions)

> [!div class="nextstepaction"]
> [PopupTemplate](/javascript/api/azure-maps-control/atlas.popuptemplate)

Pour voir des exemples de codes complets, consultez les articles suivants qui sont très intéressants :

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Ajouter un marqueur HTML](./map-add-custom-html.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de lignes](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de polygones](map-add-shape.md)