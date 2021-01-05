---
title: Se connecter à un service WFS (Web Feature Service) | Microsoft Azure Maps
description: Découvrez comment vous connecter à un service WFS, puis interroger celui-ci à l’aide du Kit de développement logiciel (SDK) web Azure Maps et du module d’E/S spatiales.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 75a7da41e4f0d5356ca135eb3cb4cbe5f29e8116
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891446"
---
# <a name="connect-to-a-wfs-service"></a>Se connecter à un service WFS

Un service WFS (Web Feature Service) est un service web qui interroge des données spatiales ayant une API normalisée, telle que définie par l’OGC (Open Geospatial Consortium). La classe `WfsClient` dans le module d’E/S spatiales permet aux développeurs de se connecter à un service WFS et d’interroger des données du service.

Les caractéristiques suivantes sont prises en charge par la classe `WfsClient` :

- Versions prises en charge : `1.0.0`, `1.1.0` et `2.0.0`
- Opérateurs de filtre pris en charge : comparaisons binaires, logique, math, valeur et `bbox`.
- Les demandes sont effectuées uniquement à l’aide de `HTTP GET`.
- Opérations prises en charge :

    | Opération | Description |
    | :-- | :-- |
    | GetCapabilities | Génère un document de métadonnées avec des paramètres et des opérations WFS valides |
    | GetFeature | Retourne une sélection de caractéristiques à partir d’une source de données |
    | DescribeFeatureType | Retourne les types de caractéristiques prises en charge |

## <a name="using-the-wfs-client"></a>Utilisation du client WFS

La classe `atlas.io.ogc.WfsClient` dans le module d’E/S spatiales permet d’interroger facilement un service WFS et de convertir les réponses en objets GeoJSON. Un tel objet GeoJSON est ensuite utilisable à d’autres fins de cartographie.

Le code suivant interroge un service WFS et restitue les caractéristiques retournées sur la carte.

<br/>

<iframe height='700' scrolling='no' title='Exemple de WFS simple' src='//codepen.io/azuremaps/embed/MWwvVYY/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’<a href='https://codepen.io/azuremaps/pen/MWwvVYY/'>exemple de WFS simple</a> Pen d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="supported-filters"></a>Filtres pris en charge

La spécification de la norme WFS utilise des filtres OGC. Les filtres ci-dessous sont pris en charge par le client WFS, en supposant que le service appelé prend également en charge ces filtres. Des chaînes de filtre personnalisées peuvent être passées dans la classe `CustomFilter`.

**Opérateurs logiques**

- `And`
- `Or`
- `Not`

**Opérateurs de valeur**

- `GmlObjectId`
- `ResourceId`

**Opérateurs mathématiques**

- `Add`
- `Sub`
- `Mul`
- `Div`

**Opérateurs de comparaison**

- `PropertyIsEqualTo`
- `PropertyIsNotEqualTo`
- `PropertyIsLessThan`
- `PropertyIsGreaterThan`
- `PropertyIsLessThanOrEqualTo`
- `PropertyIsGreaterThanOrEqualTo`
- `PropertyIsLike`
- `PropertyIsNull`
- `PropertyIsNil`
- `PropertyIsBetween`

Le code suivant illustre l’utilisation de différents filtres avec le client WFS.

<br/>

<iframe height='500' scrolling='no' title= 'Exemples de filtres WFS' src='//codepen.io/azuremaps/embed/NWqvYrV/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez les <a href='https://codepen.io/azuremaps/pen/NWqvYrV/'>exemples de filtres WFS</a> Pen d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="wfs-service-explorer"></a>Explorateur de service WFS

Le code suivant utilise le client WFS pour explorer les services WFS. Sélectionnez une couche de type de propriété dans le service et consultez la légende associée.

<br/>

<iframe height='700' style='width: 100%;' scrolling='no' title= 'Explorateur de service WFS' src='//codepen.io/azuremaps/embed/bGdrvmG/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’<a href='https://codepen.io/azuremaps/pen/bGdrvmG/'>Explorateur de service WFS</a> Pen d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Pour accéder à des services WFS hébergés sur des points de terminaison non-CORS, un service proxy CORS peut être passé dans l’option `proxyService` du client WFS, comme indiqué ci-dessous. 

```JavaScript
//Create the WFS client to access the service and use the proxy service settings
client = new atlas.io.ogc.WfsClient({
    url: url,
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
});
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [WfsClient](/JavaScript/api/azure-maps-spatial-io/atlas.io.ogc.wfsclient)

> [!div class="nextstepaction"]
> [WfsServiceOptions](/JavaScript/api/azure-maps-spatial-io/atlas.wfsserviceoptions)

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Exploiter les opérations de base](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Détails sur les formats de données pris en charge](spatial-io-supported-data-format-details.md)