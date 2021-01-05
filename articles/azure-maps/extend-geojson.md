---
title: Géométries GeoJSON étendues | Microsoft Azure Maps
description: Découvrez comment Azure Maps étend la spécification GeoJSON pour inclure des formes géométriques supplémentaires. Affichez des exemples qui définissent des cercles et des rectangles à utiliser dans les mappages.
author: sataneja
ms.author: sataneja
ms.date: 05/17/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: 87a5dec2582796c6703bfb37b76e26e53a1192c0
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895781"
---
# <a name="extended-geojson-geometries"></a>Géométries GeoJSON étendues

Azure Maps fournit une liste de puissantes API permettant de rechercher à l’intérieur et le long des caractéristiques géographiques. Ces API adhèrent à la [spécification GeoJSON][1] standard de la représentation des caractéristiques géographiques.  

La [spécification GeoJSON][1] ne prend en charge que les géométries suivantes :

* GeometryCollection
* LineString
* MultiLineString
* MultiPoint
* MultiPolygon
* Point
* Polygone

Certaines API Azure Maps acceptent des géométries qui ne font pas partie de la [spécification GeoJSON][1]. Par exemple, l’API [Search Inside Geometry](/rest/api/maps/search/postsearchinsidegeometry) accepte des cercles et des polygones.

Cet article fournit une explication détaillée sur la façon dont Azure Maps étend la [spécification GeoJSON][1] pour représenter certaines géométries.

## <a name="circle"></a>Circle

La géométrie `Circle` n’est pas prise en charge par la [spécification GeoJSON][1]. Nous utilisons un objet `GeoJSON Point Feature` pour représenter un cercle.

Une géométrie `Circle` représentée à l’aide de l’objet `GeoJSON Feature` __doit__ contenir les coordonnées et les propriétés suivantes :

- Center

    Le centre du cercle est représenté par un objet `GeoJSON Point`.

- Radius

    Le `radius` du cercle est représenté à l’aide des propriétés de `GeoJSON Feature`. La valeur du rayon est exprimée en _mètres_ et doit être du type `double`.

- Subtype

    La géométrie circle doit également contenir la propriété `subType`. Cette propriété doit faire partie des propriétés de la `GeoJSON Feature`, et sa valeur doit être _Circle_ .

#### <a name="example"></a>Exemple

Voici comment vous représenterez un cercle à l’aide d’un objet `GeoJSON Feature`. Nous allons centrer le cercle à la latitude : 47,639754 et à la longitude :-122,126986, et lui attribuez un rayon égal à 100 mètres :

```json            
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.126986, 47.639754]
    },
    "properties": {
        "subType": "Circle",
        "radius": 100
    }
}          
```

## <a name="rectangle"></a>Rectangle

La géométrie `Rectangle` n’est pas prise en charge par la [spécification GeoJSON][1]. Nous utilisons un objet `GeoJSON Polygon Feature` pour représenter un rectangle. L’extension de rectangle est principalement utilisée par le module des outils de dessin du SDK web.

Une géométrie `Rectangle` représentée à l’aide de l’objet `GeoJSON Polygon Feature` __doit__ contenir les coordonnées et les propriétés suivantes :

- Angles

    Les angles du rectangle sont représentés à l’aide des coordonnées d’un objet `GeoJSON Polygon`. Il doit y avoir cinq coordonnées, une pour chaque sommet et une cinquième identique à la première coordonnée pour fermer l’anneau polygonal. Il est supposé que ces coordonnées s’alignent et que le développeur peut les faire pivoter comme il le souhaite.

- Subtype

    La géométrie rectangle doit également contenir la propriété `subType`. Cette propriété doit faire partie des propriétés de `GeoJSON Feature`, et sa valeur doit être _Rectangle_ .

### <a name="example"></a>Exemple

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Polygon",
        "coordinates": [[[5,25],[14,25],[14,29],[5,29],[5,25]]]
    },
    "properties": {
        "subType": "Rectangle"
    }
}

```
## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les données GeoJSON dans Azure Maps :

> [!div class="nextstepaction"]
> [Format GeoJSON de limite géographique](geofence-geojson.md)

Consultez le glossaire des termes techniques courants associés à Azure Maps et aux applications de géolocalisation :

> [!div class="nextstepaction"]
> [Glossaire Azure Maps](glossary.md)

[1]: https://tools.ietf.org/html/rfc7946