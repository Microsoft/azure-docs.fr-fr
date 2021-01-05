---
title: Navigateurs pris en charge par le SDK web | Microsoft Azure Maps
description: Découvrez comment vérifier si le Kit de développement logiciel (SDK) web Azure Maps prend en charge un navigateur. Affichez la liste des navigateurs pris en charge. Découvrez comment utiliser les services cartographiques avec les navigateurs hérités.
author: rbrundritt
ms.author: richbrun
ms.date: 03/25/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: 43bf70d66c42bc0ecd5e26e4cc724456bd4bf84e
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896920"
---
# <a name="web-sdk-supported-browsers"></a>Navigateurs pris en charge pour le kit SDK web

Le SDK web Azure Maps fournit une fonction d’assistance appelée [atlas.isSupported](/javascript/api/azure-maps-control/atlas#issupported-boolean-). Cette fonction détecte si un navigateur web a l’ensemble minimal de fonctionnalités WebGL requises pour prendre en charge le chargement et le rendu du contrôle de carte. L’exemple suivant montre comment utiliser la fonction :

```JavaScript
if (!atlas.isSupported()) {
    alert('Your browser is not supported by Azure Maps');
} else if (!atlas.isSupported(true)) {
    alert('Your browser is supported by Azure Maps, but may have major performance caveats.');
} else {
    // Your browser is supported. Add your map code here.
}
```

## <a name="desktop"></a>Bureau

Le SDK web Azure Maps prend en charge les navigateurs de bureau suivants :

- Microsoft Edge (versions actuelle et précédentes)
- Google Chrome (versions actuelle et précédentes)
- Mozilla Firefox (versions actuelle et précédentes)
- Apple Safari (Mac OS X) (versions actuelle et précédentes)

Consultez aussi la section [Navigateurs hérités cibles](#Target-Legacy-Browsers) plus loin dans cet article.

## <a name="mobile"></a>Mobile

Le SDK web Azure Maps prend en charge les navigateurs mobiles suivants :

- Android
  - Version actuelle de Chrome sur Android 6.0 et versions ultérieures
  - Chrome WebView sur Android 6.0 et versions ultérieures
- iOS
  - Safari Mobile sur la version principale actuelle et précédente d’iOS
  - UIWebView et WKWebView sur la version principale actuelle et précédente d’iOS
  - Version actuelle de Chrome pour iOS

> [!TIP]
> Si vous incorporez une carte à l’intérieur d’une application mobile à l’aide d’un contrôle WebView, vous pouvez utiliser le [package npm du SDK web Azure Maps](https://www.npmjs.com/package/azure-maps-control) au lieu de référencer la version du SDK qui est hébergé sur Azure CDN. Cette approche réduit le temps de chargement, car le SDK se trouve déjà sur l’appareil de l’utilisateur et n’a pas besoin d’être téléchargé au moment de l’exécution.

## <a name="nodejs"></a>Node.js

Les modules de SDK web suivants sont également pris en charge dans Node.js :

- Module Services ([documentation](how-to-use-services-module.md) | [module npm](https://www.npmjs.com/package/azure-maps-rest))

## <a name="target-legacy-browsers"></a><a name="Target-Legacy-Browsers"></a>Navigateurs hérités cibles

Vous pouvez cibler des navigateurs plus anciens qui ne prennent pas en charge WebGL ou qui en ont seulement une prise en charge limitée. Dans ces cas-là, nous vous recommandons d’utiliser les services Azure Maps avec un contrôle de carte open source tel que [Leaflet](https://leafletjs.com/). Voici un exemple :

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Azure Maps + Leaflet" src="//codepen.io/azuremaps/embed/GeLgyx/?height=500&theme-id=0&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consultez la page <a href='https://codepen.io/azuremaps/pen/GeLgyx/'>Azure Maps + Leaflet</a> par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le SDK web Azure Maps :

[Contrôle de carte](how-to-use-map-control.md)

[Module de services](how-to-use-services-module.md)