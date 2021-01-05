---
title: Afficher le trafic sur une carte | Microsoft Azure Maps
description: Découvrez comment ajouter des données de trafic aux cartes. Découvrez les données de flux et l’utilisation du Kit de développement logiciel (SDK) web Azure Maps pour ajouter des données d’incident et de flux aux cartes.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-js
ms.openlocfilehash: ec7459a356221fb7b599cbbc02f1cb825920b5b3
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890664"
---
# <a name="show-traffic-on-the-map"></a>Afficher le trafic sur la carte

Deux types de données de trafic sont disponibles dans Azure Maps :

- Données d’incident : elles se composent de données de type point et ligne pour des éléments tels que les travaux, les fermetures de route et les accidents.
- Données de circulation : elles fournissent des métriques concernant la circulation sur les routes. Les données de circulation sont souvent utilisées pour colorer les routes. Les couleurs dépendent du volume de circulation qui entraîne un ralentissement par rapport à la limite de vitesse ou à une autre métrique. Les données de circulation dans Azure Maps ont trois métriques différentes de mesure :
    - `relative` : est relatif à la vitesse de circulation libre de la route.
    - `absolute` : est la vitesse absolue de tous les véhicules sur la route.
    - `relative-delay` : affiche les zones qui sont plus lentes que le délai moyen attendu.

Le code suivant montre comment afficher les données de trafic sur la carte.

```javascript
//Show traffic on the map using the traffic options.
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```

Vous trouverez ci-dessous l’exemple de code d’exécution complet de la fonctionnalité ci-dessus.

<br/>

<iframe height='500' scrolling='no' title='Afficher le trafic sur une carte' src='//codepen.io/azuremaps/embed/WMLRPw/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez la page <a href='https://codepen.io/azuremaps/pen/WMLRPw/'>Show traffic on a map</a> (Afficher le trafic sur une carte) d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="traffic-overlay-options"></a>Options de superposition du trafic

L’outil suivant vous permet de basculer entre les différents paramètres de superposition du trafic pour voir comment le rendu change. 

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Options de superposition du trafic" src="//codepen.io/azuremaps/embed/RwbPqRY/?height=700&theme-id=0&default-tab=result" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/RwbPqRY/'>Traffic overlay options</a> (Options de superposition du trafic) Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="add-traffic-controls"></a>Ajouter des contrôles de trafic

Deux contrôles de trafic différents peuvent être ajoutés à la carte. Le premier contrôle, `TrafficControl`, ajoute un bouton bascule qui peut être utilisé pour activer et désactiver le trafic. Les options de ce contrôle vous permettent de spécifier les paramètres de trafic à utiliser lors de l’affichage du trafic. Par défaut, ce contrôle affiche la circulation relative et les données d’incident. Toutefois, vous pouvez le changer pour voir la circulation absolue sans les incidents si vous le souhaitez. Le deuxième contrôle, `TrafficLegendControl`, ajoute une légende de circulation à la carte qui aide l’utilisateur à comprendre ce que signifie le code couleur de la route. Ce contrôle apparaît sur la carte seulement si les données de circulation sont affichées sur la carte ; autrement, il est toujours masqué.

Le code suivant montre comment ajouter les contrôles de trafic à la carte.

```JavaScript
//Att the traffic control toogle button to the top right corner of the map.
map.controls.add(new atlas.control.TrafficControl(), { position: 'top-right' });

//Att the traffic legend control to the bottom left corner of the map.
map.controls.add(new atlas.control.TrafficLegendControl(), { position: 'bottom-left' });
```

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Contrôles de trafic" src="https://codepen.io/azuremaps/embed/ZEWaeLJ?height500&theme-id=0&default-tab=js,result&embed-version=2&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/ZEWaeLJ'>Traffic controls</a> Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [TrafficOptions](/javascript/api/azure-maps-control/atlas.trafficoptions)

Améliorez l’expérience de vos utilisateurs :

> [!div class="nextstepaction"]
> [Interaction avec la carte - Événements de souris](map-events.md)

> [!div class="nextstepaction"]
> [Building an accessible map](map-accessibility.md) (Création d’une carte accessible)

> [!div class="nextstepaction"]
> [Page d’exemples de code](https://aka.ms/AzureMapsSamples)