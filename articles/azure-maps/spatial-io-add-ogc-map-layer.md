---
title: Ajouter une couche Open Geospatial Consortium (OGC) | Microsoft Azure Maps
description: Découvrez comment superposer une couche OGC sur la carte et comment utiliser les différentes options de la classe OgcMapLayer.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 7d94a681fa987a4b23dbcda744c2f8516da4437a
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891663"
---
# <a name="add-a-map-layer-from-the-open-geospatial-consortium-ogc"></a>Ajouter une couche fournie par l’Open Geospatial Consortium (OGC)

La classe `atlas.layer.OgcMapLayer` peut superposer des images WMS (Web Map Service) et WMTS (Web Map Tile Service) sur la carte. WMS est un protocole standard développé par l’OGC pour servir les images cartographiques géoréférencées sur Internet. Le géoréférencement des images est le processus d’association d’une image à un emplacement géographique. WMTS est également un protocole standard développé par l’OGC. Il est conçu pour proposer des mosaïques prérendues et géoréférencées.

Les sections suivantes décrivent les fonctionnalités du service de carte web prises en charge par la classe `OgcMapLayer`.

**Web Map Service (WMS)**

- Versions prises en charge : `1.0.0`, `1.1.0`, `1.1.1` et `1.3.0`
- Le service doit prendre en charge le système de projection `EPSG:3857`, ou gérer les reprojections.
- GetFeatureInfo requiert que le service prenne en charge `EPSG:4326` ou gère les reprojections. 
- Opérations prises en charge :

    | Opération | Description |
    | :-- | :-- |
    | GetCapabilities | Récupère les métadonnées relatives au service avec les capacités prises en charge |
    | GetMap | Récupère une image de carte pour une région spécifiée |
    | GetFeatureInfo | Récupère `feature_info`, qui contient les données sous-jacentes relatives à la fonctionnalité |

**Web Map Tile Service (WMTS)**

- Versions prises en charge : `1.0.0`
- Les mosaïques doivent être carrées, de sorte que `TileWidth == TileHeight`.
- CRS pris en charge : `EPSG:3857` ou `GoogleMapsCompatible` 
- L’identificateur TileMatrix doit être une valeur entière qui correspond à un niveau de zoom sur la carte. Sur une carte Azure, le niveau de zoom est une valeur comprise entre `"0"` et `"22"`. Ainsi, `"0"` est pris en charge, contrairement à `"00"`.
- Opérations prises en charge :

    | Opération | Description |
    | :-- | :-- |
    | GetCapabilities | Récupère les opérations et fonctionnalités prises en charge |
    | GetTile | Récupère les images d’une mosaïque particulière |

## <a name="overlay-an-ogc-map-layer"></a>Superposer une couche OGC

L’`url` peut être l’URL de base du service ou une URL complète avec la requête visant à obtenir les capacités du service. Selon les détails fournis, le client WFS peut essayer plusieurs formats d’URL standard pour déterminer comment accéder initialement au service.

Le code suivant montre comment superposer une couche OGC sur la carte.

<br/>

<iframe height='700' scrolling='no' title='Exemple de couche OGC' src='//codepen.io/azuremaps/embed/xxGLZWB/?height=700&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/xxGLZWB/'>OGC Map layer example</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="ogc-map-layer-options"></a>Options de couche OGC

L’exemple ci-dessous illustre les différentes options de couche OGC. Vous pouvez cliquer sur le bouton CodePen dans le coin supérieur droit pour modifier l’extrait de code.

<br/>

<iframe height='700' scrolling='no' title='Options de couche OGC' src='//codepen.io/azuremaps/embed/abOyEVQ/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/abOyEVQ/'>OGC map layer options</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="ogc-web-map-service-explorer"></a>Explorateur Web Map Service OGC

L’outil suivant superpose les images des services WMS (Web Map Service) et WMTS (Web Map Tile Service) en tant que couches. Vous pouvez sélectionner les couches du service qui sont affichées sur la carte. Vous pouvez également afficher les légendes associées à ces couches.

<br/>

<iframe height='750' style='width: 100%;' scrolling='no' title='Explorateur Web Map Service OGC' src='//codepen.io/azuremaps/embed/YzXxYdX/?height=750&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/YzXxYdX/'>OGC Web Map Service explorer</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Vous pouvez également spécifier les paramètres de carte pour utiliser un service proxy. Le service proxy vous permet de charger des ressources hébergées sur des domaines où CORS n’est pas activé.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [OgcMapLayer](/javascript/api/azure-maps-spatial-io/atlas.layer.ogcmaplayer)

> [!div class="nextstepaction"]
> [OgcMapLayerOptions](/javascript/api/azure-maps-spatial-io/atlas.ogcmaplayeroptions)

Consultez les articles suivants, qui contiennent des exemples de code que vous pouvez ajouter à vos cartes :

> [!div class="nextstepaction"]
> [Se connecter à un service WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Exploiter les opérations de base](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Détails sur les formats de données pris en charge](spatial-io-supported-data-format-details.md)