---
title: Détails sur les formats de données pris en charge | Microsoft Azure Maps
description: Découvrez comment les données spatiales délimitées sont analysées dans le module d’E/S spatiales.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 7227813f607ca18ee50f503a30b290414f333e21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91310167"
---
# <a name="supported-data-format-details"></a>Détails sur les formats de données pris en charge

Cet article fournit des détails sur la prise en charge de la lecture et de l’écriture pour toutes les balises XML et tous les types de géométrie Well Known Text. Il détaille également la manière dont les données spatiales délimitées sont analysées dans le module d’E/S spatiales.

## <a name="supported-xml-namespaces"></a>Espaces de noms XML pris en charge

Le module d’E/S spatiales prend en charge les balises XML des espaces de noms suivants.

| Préfixe d’espace de noms | URI d’espace de noms   | Notes                                                                    |
|:------------------|:-----------------|:----------------------------------------|
| `atom`           | `http://www.w3.org/2005/Atom`   |                                         |
| `geo`            | `http://www.w3.org/2003/01/geo/wgs84_pos#`  | Prise en charge de la lecture seule dans les fichiers GeoRSS.           |
| `georss`         | `http://www.georss.org/georss`  |                                                |
| `geourl`         | `http://geourl.org/rss/module/` | Prise en charge de la lecture seule dans les fichiers GeoRSS.                       |
| `gml`            | `http://www.opengis.net/gml`    |                                                        |
| `gpx`            | `http://www.topografix.com/GPX/1/1` |                                                   |
| `gpxx`           | `http://www.garmin.com/xmlschemas/GpxExtensions/v3` | Prise en charge de la lecture seule dans les fichiers GPX. Analyse et utilise DisplayColor. Toutes les autres propriétés ajoutées pour former les métadonnées. |
| `gpx_style`      | `http://www.topografix.com/GPX/gpx_style/0/2`      | Pris en charge dans les fichiers GPX. Utilise la couleur de ligne. |
| `gx`             | `http://www.google.com/kml/ext/2.2` |                                                      |
| `kml`            | `http://www.opengis.net/kml/2.2`    |                                                      |
| `rss`            |                                 | Lecture seule. GeoRSS écrit au format Atom.              |

## <a name="supported-xml-elements"></a>Éléments XML pris en charge

Le module d’E/S spatiales prend en charge les éléments XML suivants. Les balises XML qui ne sont pas prises en charge sont converties en objet JSON. Ensuite, chaque balise sera ajoutée en tant que propriété dans le champ `properties` de la forme ou de la couche parente.

### <a name="kml-elements"></a>Éléments KML

Le module d’E/S spatiales prend en charge les éléments KML suivants.

| Nom de l’élément         | Lire    | Write   | Notes                                                                                                                      |
|----------------------|---------|---------|----------------------------------------------------------------------------------------------------------------------------|
| `address`            | partielles | Oui     | L’objet est analysé, mais n’est pas utilisé pour le positionnement de la forme.                                                                    |
| `AddressDetails`     | partielles | non      | L’objet est analysé, mais n’est pas utilisé pour le positionnement de la forme.                                                                    |
| `atom:author`        | Oui     | Oui     |                                                                                                                            |
| `atom:link`          | Oui     | Oui     |                                                                                                                            |
| `atom:name`          | Oui     | Oui     |                                                                                                                            |
| `BalloonStyle`       | partielles | partielles | `displayMode` n’est pas pris en charge. Converti en `PopupTemplate`. Pour écrire, ajoutez une propriété `popupTemplate` en tant que propriété de la fonctionnalité pour laquelle vous souhaitez écrire. |
| `begin`              | Oui     | Oui     |                                                                                                                            |
| `color`              | Oui     | Oui     | Inclut `#AABBGGRR` et `#BBGGRR`. Analysé dans une chaîne de couleur CSS                                                           |
| `colorMode`          | Oui     | non      |                                                                                                                            |
| `coordinates`        | Oui     | Oui     |                                                                                                                            |
| `Data`               | Oui     | Oui     |                                                                                                                            |
| `description`        | Oui     | Oui     |                                                                                                                            |
| `displayName`        | Oui     | Oui     |                                                                                                                            |
| `Document`           | Oui     | Oui     |                                                                                                                            |
| `drawOrder`          | partielles | non      | Lecture pour les superpositions de sol et utilisées pour les trier. 
| `east`               | Oui     | Oui     |                                                                                                                            |
| `end`                | Oui     | Oui     |                                                                                                                            |
| `ExtendedData`       | Oui     | Oui     | Prend en charge les éléments `Data`, `SimpleData` ou `Schema` non typés, ainsi que les remplacements d’entité du formulaire `$[dataName]`.                      |
| `extrude`            | partielles | partielles | Uniquement pris en charge pour les polygones. Les polygéométries qui comportent des polygones de différentes hauteurs seront décomposées en fonctionnalités individuelles. Les styles de ligne ne sont pas pris en charge. Les polygones dont l’altitude est égale à 0 sont rendus sous forme de polygone plat. Lors de la lecture, l’altitude de la première coordonnée dans l’anneau extérieur est ajoutée en tant que propriété de hauteur du polygone. Ensuite, l’altitude de la première coordonnée sera utilisée pour restituer le polygone sur la carte. |
| `fill`               | Oui     | Oui     |                                                                                                                            |
| `Folder`             | Oui     | Oui     |                                                                                                                            |
| `GroundOverlay`      | Oui     | Oui     | `color` n’est pas pris en charge                                                                                                   |
| `heading`            | partielles | non      | Analysé mais non restitué par `SimpleDataLayer`. Écrit uniquement si les données sont stockées dans la propriété de la forme.                 |
| `hotSpot`            | Oui     | partielles | Écrit uniquement si les données sont stockées dans la propriété de la forme. Les unités sont envoyées en tant que « pixels » uniquement.                         |
| `href`               | Oui     | Oui     |                                                                                                                            |
| `Icon`               | partielles | partielles | Analysé mais non restitué par `SimpleDataLayer`. Écrit uniquement la propriété Icon de la forme si elle contient des données URI. Seul `href` est pris en charge. |
| `IconStyle`          | partielles | partielles | Les valeurs `icon`, `heading`, `colorMode` et `hotspots` sont analysées, mais elles ne sont pas rendues par `SimpleDataLayer`         |
| `innerBoundaryIs`    | Oui     | Oui     |                                                                                                                            |
| `kml`                | Oui     | Oui     |                                                                                                                            |
| `LabelStyle`         | non      | non      |                                                                                                                            |
| `LatLonBox`          | Oui     | Oui     |                                                                                                                            |
| `gx:LatLonQuad`      | Oui     | Oui     |                                                                                                                            |
| `LinearRing`         | Oui     | Oui     |                                                                                                                            |
| `LineString`         | Oui     | Oui     |                                                                                                                            |
| `LineStyle`          | Oui     | Oui     | `colorMode` n’est pas pris en charge.                                                                                         |
| `Link`               | Oui     | non      | Seule la propriété `href` est prise en charge pour les liens réseau.                                                                   |
| `MultiGeometry`      | partielles | partielles | Peut être décomposée en fonctionnalités individuelles lors de la lecture.                                                                     |
| `name`               | Oui     | Oui     |                                                                                                                            |
| `NetworkLink`        | Oui     | non      | Les liens doivent se trouver dans le même domaine que le document.                                                                  |
| `NetworkLinkControl` | non      | non      |                                                                                                                            |
| `north`              | Oui     | Oui     |                                                                                                                            |
| `open`               | Oui     | Oui     |                                                                                                                            |
| `outerBoundaryIs`    | Oui     | Oui     |                                                                                                                            |
| `outline`            | Oui     | Oui     |                                                                                                                            |
| `overlayXY`          | non      | non      |                                                                                                                            |
| `Pair`               | partielles | non      | Seul le style `normal` dans un `StyleMap` est pris en charge. `highlight` n’est pas pris en charge.                                   |
| `phoneNumber`        | Oui     | Oui     |                                                                                                                            |
| `PhotoOverlay`       | non      | non      |                                                                                                                            |
| `Placemark`          | Oui     | Oui     |                                                                                                                            |
| `Point`              | Oui     | Oui     |                                                                                                                            |
| `Polygon`            | Oui     | Oui     |                                                                                                                            |
| `PolyStyle`          | Oui     | Oui     |                                                                                                                            |
| `Region`             | partielles | partielles | `LatLongBox` est pris en charge au niveau du document.                                                                      |
| `rotation`           | non      | non      |                                                                                                                            |
| `rotationXY`         | non      | non      |                                                                                                                            |
| `scale`              | non      | non      |                                                                                                                            |
| `Schema`             | Oui     | Oui     |                                                                                                                            |
| `SchemaData`         | Oui     | Oui     |                                                                                                                            |
| `schemaUrl`          | partielles | Oui     | Ne prend pas en charge le chargement de styles à partir de documents externes qui ne sont pas inclus dans un KMZ.                             |
| `ScreenOverlay`      | non      | non      |                                                                                                                            |
| `screenXY`           | non      | non      |                                                                                                                            |
| `SimpleData`         | Oui     | Oui     |                                                                                                                            |
| `SimpleField`        | Oui     | Oui     |                                                                                                                            |
| `size`               | non      | non      |                                                                                                                            |
| `Snippet`            | partielles | partielles | L’attribut `maxLines` est ignoré.                                                                                  |
| `south`              | Oui     | Oui     |                                                                                                                            |
| `Style`              | Oui     | Oui     |                                                                                                                            |
| `StyleMap`           | partielles | non      | Seul le style normal dans un `StyleMap` est pris en charge.                                                                        |
| `styleUrl`           | partielles | Oui     | Les URL de style externe ne sont pas prises en charge.                                                                         |
| `text`               | Oui     | Oui     | Le remplacement de `$[geDirections]` n’est pas pris en charge                                                                          |
| `textColor`          | Oui     | Oui     |                                                                                                                            |
| `TimeSpan`           | Oui     | Oui     |                                                                                                                            |
| `TimeStamp`          | Oui     | Oui     |                                                                                                                            |
| `value`              | Oui     | Oui     |                                                                                                                            |
| `viewRefreshMode`    | partielles | non      |  Si vous pointez vers un service WMS, seul `onStop` est pris en charge pour les superpositions de sol. Ajoute `BBOX=[bboxWest],[bboxSouth],[bboxEast],[bboxNorth]` à l’URL et se met à jour au fur et à mesure du déplacement de la carte.  |
| `visibility`         | Oui     | Oui     |                                                                                                                            |
| `west`               | Oui     | Oui     |                                                                                                                            |
| `when`               | Oui     | Oui     |                                                                                                                            |
| `width`              | Oui     | Oui     |                                                                                                                            |

### <a name="georss-elements"></a>Éléments GeoRSS

Le module d’E/S spatiales prend en charge les éléments GeoRSS suivants.

| Nom de l’élément             | Lire    | Write | Notes                                                                                          |
|--------------------------|---------|-------|------------------------------------------------------------------------------------------------|
| `atom:author`            | Oui     | Oui   |                                                                                                |
| `atom:category`          | Oui     | Oui   |                                                                                                |
| `atom:content`           | Oui     | Oui   |                                                                                                |
| `atom:contributor`       | Oui     | Oui   |                                                                                                |
| `atom:email`             | Oui     | Oui   |                                                                                                |
| `atom:entry`             | Oui     | Oui   |                                                                                                |
| `atom:feed`              | Oui     | Oui   |                                                                                                |
| `atom:icon`              | Oui     | Oui   |                                                                                                |
| `atom:id`                | Oui     | Oui   |                                                                                                |
| `atom:link`              | Oui     | Oui   |                                                                                                |
| `atom:logo`              | Oui     | Oui   |                                                                                                |
| `atom:name`              | Oui     | Oui   |                                                                                                |
| `atom:published`         | Oui     | Oui   |                                                                                                |
| `atom:rights`            | Oui     | Oui   |                                                                                                |
| `atom:source`            | Oui     | Oui   |                                                                                                |
| `atom:subtitle`          | Oui     | Oui   |                                                                                                |
| `atom:summary`           | Oui     | Oui   |                                                                                                |
| `atom:title`             | Oui     | Oui   |                                                                                                |
| `atom:updated`           | Oui     | Oui   |                                                                                                |
| `atom:uri`               | Oui     | Oui   |                                                                                                |
| `geo:lat`                | Oui     | non    | Écrit sous la forme d’un `georss:point`.                                                                   |
| `geo:lon`                | Oui     | non    | Écrit sous la forme d’un `georss:point`.                                                                   |
| `geo:long`               | Oui     | non    | Écrit sous la forme d’un `georss:point`.                                                                   |
| `georss:box`             | Oui     | non    | Lu comme un polygone et doté d’une propriété `subType` de « Rectangle ».                                |
| `georss:circle`          | Oui     | Oui   |                                                                                                |
| `georss:elev`            | Oui     | Oui   |                                                                                                |
| `georss:featurename`     | Oui     | Oui   |                                                                                                |
| `georss:featuretypetag`  | Oui     | Oui   |                                                                                                |
| `georss:floor`           | Oui     | Oui   |                                                                                                |
| `georss:line`            | Oui     | Oui   |                                                                                                |
| `georss:point`           | Oui     | Oui   |                                                                                                |
| `georss:polygon`         | Oui     | Oui   |                                                                                                |
| `georss:radius`          | Oui     | Oui   |                                                                                                |
| `georss:relationshiptag` | Oui     | Oui   |                                                                                                |
| `georss:where`           | Oui     | Oui   |                                                                                                |
| `geourl:latitude`        | Oui     | non    | Écrit sous la forme d’un `georss:point`.                                                                   |
| `geourl:longitude`       | Oui     | non    | Écrit sous la forme d’un `georss:point`.                                                                   |
| `position`               | Oui     | non    | Certains flux XML enveloppent GML avec une balise de position au lieu de l’envelopper avec une balise `georss:where`. Lira cette balise, mais écrira à l’aide d’une balise `georss:where`. |
| `rss`                    | Oui     | non    | GeoRSS écrit au format ATOM.                                                                 |
| `rss:author`             | Oui     | partielles | Écrit sous la forme d’un `atom:author`.                                                                 |
| `rss:category`           | Oui     | partielles | Écrit sous la forme d’un `atom:category`.                                                               |
| `rss:channel`            | Oui     | non    |                                                                                                |
| `rss:cloud`              | Oui     | non    |                                                                                                |
| `rss:comments`           | Oui     | non    |                                                                                                |
| `rss:copyright`          | Oui     | partielles | Écrit comme un `atom:rights` si la forme n’avait pas déjà une propriété `rights` `properties`.       |
| `rss:description`        | Oui     | partielles | Écrit comme un `atom:content` si la forme n’avait pas déjà une propriété `content` `properties`.      |
| `rss:docs`               | Oui     | non    |                                                                                                |
| `rss:enclosure`          | Oui     | non    |                                                                                                |
| `rss:generator`          | Oui     | non    |                                                                                                |
| `rss:guid`               | Oui     | partielles | Écrit comme un `atom:id` si la forme n’avait pas déjà une propriété `id` `properties`.         |
| `rss:image`              | Oui     | partielles | Écrit comme un `atom:logo` si la forme n’avait pas déjà une propriété `logo` `properties`.      |
| `rss:item`               | Oui     | partielles | Écrit sous la forme d’un `atom:entry`.                                                                  |
| `rss:language`           | Oui     | non    |                                                                                                |
| `rss:lastBuildDate`      | Oui     | partielles | Écrit comme un `atom:updated` si la forme n’avait pas déjà une propriété `updated` `properties`.     |
| `rss:link`               | Oui     | partielles | Écrit sous la forme d’un `atom:link`.                                                                   |
| `rss:managingEditor`     | Oui     | partielles | Écrit sous la forme d’un `atom:contributor`.                                                            |
| `rss:pubDate`            | Oui     | partielles | Écrit comme un `atom:published` si la forme n’avait pas déjà une propriété `published` `properties`.  |
| `rss:rating`             | Oui     | non    |                                                                                                |
| `rss:skipDays`           | Oui     | non    |                                                                                                |
| `rss:skipHours`          | Oui     | non    |                                                                                                |
| `rss:source`             | Oui     | partielles | Écrit sous la forme d’un `atom:source` contenant un `atom:link`.                                       |
| `rss:textInput`          | Oui     | non    |                                                                                                |
| `rss:title`              | Oui     | partielles | Écrit sous la forme d’un `atom:title`.                                                                  |
| `rss:ttl`                | Oui     | non    |                                                                                                |
| `rss:webMaster`          | Oui     | non    |                                                                                                |

### <a name="gml-elements"></a>Éléments GML

Le module d’E/S spatiales prend en charge les éléments GML suivants. 

| Nom de l’élément            | Lire | Write | Notes                                                                                  |
|-------------------------|------|-------|----------------------------------------------------------------------------------------|
| `gml:coordinates`       | Oui  | non    | Écrit sous la forme d’un `gml:posList`.                                                              |
| `gml:curveMember`       | Oui  | non    |                                                                                        |
| `gml:curveMembers`      | Oui  | non    |                                                                                        |
| `gml:Box`               | Oui  | non    | Écrit sous la forme d’un `gml:Envelope`.                                                             |
| `gml:description`       | Oui  | Oui   |                                                                                        |
| `gml:Envelope`          | Oui  | Oui   |                                                                                        |
| `gml:exterior`          | Oui  | Oui   |                                                                                        |
| `gml:Feature`           | Oui  | non    | Écrit comme une forme.                                                                    |
| `gml:FeatureCollection` | Oui  | non    | Écrit sous la forme d’une collection de géométries.                                                      |
| `gml:featureMember`     | Oui  | non    | Écrit sous la forme d’une collection de géométries.                                                      |
| `gml:geometry`          | Oui  | non    | Écrit comme une forme.                                                                    |
| `gml:geometryMember`    | Oui  | Oui   |                                                                                        |
| `gml:geometryMembers`   | Oui  | Oui   |                                                                                        |
| `gml:identifier`        | Oui  | Oui   |                                                                                        |
| `gml:innerBoundaryIs`   | Oui  | non    | Écrit à l’aide de `gml.interior`.                                                          |
| `gml:interior`          | Oui  | Oui   |                                                                                        |
| `gml:LinearRing`        | Oui  | Oui   |                                                                                        |
| `gml:LineString`        | Oui  | Oui   |                                                                                        |
| `gml:lineStringMember`  | Oui  | Oui   |                                                                                        |
| `gml:lineStringMembers` | Oui  | non    |                                                                                        |
| `gml:MultiCurve`        | Oui  | non    | Lit uniquement les membres de `gml:LineString`. Écrit sous la forme d’un `gml.MultiLineString`.                  |
| `gml:MultiGeometry`     | partielles  | partielles   | Lecture uniquement en tant que FeatureCollection.                                              |
| `gml:MultiLineString`   | Oui  | Oui   |                                                                                        |
| `gml:MultiPoint`        | Oui  | Oui   |                                                                                        |
| `gml:MultiPolygon`      | Oui  | Oui   |                                                                                        |
| `gml:MultiSurface`      | Oui  | non    | Lit uniquement les membres de `gml:Polygon`. Écrit sous la forme d’un `gml.MultiPolygon`.                        |
| `gml:name`              | Oui  | Oui   |                                                                                        |
| `gml:outerBoundaryIs`   | Oui  | non    | Écrit à l’aide de `gml.exterior`.                                                          |
| `gml:Point`             | Oui  | Oui   |                                                                                        |
| `gml:pointMember`       | Oui  | Oui   |                                                                                        |
| `gml:pointMembers`      | Oui  | non    |                                                                                        |
| `gml:Polygon`           | Oui  | Oui   |                                                                                        |
| `gml:polygonMember`     | Oui  | Oui   |                                                                                        |
| `gml:polygonMembers`    | Oui  | non    |                                                                                        |
| `gml:pos`               | Oui  | Oui   |                                                                                        |
| `gml:posList`           | Oui  | Oui   |                                                                                        |
| `gml:surfaceMember`     | Oui  | Oui   |                                                                                        |

#### <a name="additional-notes"></a>Remarques supplémentaires

- Une géométrie qui peut être enfouie dans les éléments enfants est recherchée dans les éléments membres. Cette opération de recherche est nécessaire, car de nombreux formats XML qui s’étendent à partir de GML peuvent ne pas placer une géométrie en tant qu’enfant direct d’un élément membre.
- `srsName` est partiellement pris en charge pour les coordonnées WGS84 et les codes suivants :[EPSG:4326](https://epsg.io/4326) et web Mercator ([EPSG:3857](https://epsg.io/3857) ou l’un de ses autres codes). Tout autre système de coordonnées est analysé en tant que WGS84 en l’état.
- À moins d’être spécifié lors de la lecture d’un flux XML, l’ordre de l’axe est déterminé en fonction des indications dans le flux XML. Une préférence est donnée pour l’ordre de l’axe « Latitude, longitude ».
- À moins qu’un espace de noms GML personnalisé ne soit spécifié pour les propriétés lors de l’écriture dans un fichier GML, les informations de propriété supplémentaires ne seront pas ajoutées.

### <a name="gpx-elements"></a>Éléments GPX

Le module d’E/S spatiales prend en charge les éléments GPX suivants.

| Nom de l’élément             | Lire    | Write   | Notes                                                                                       |
|--------------------------|---------|---------|---------------------------------------------------------------------------------------------|
| `gpx:ageofdgpsdata`      | Oui     | Oui     |                                                                                             |
| `gpx:author`             | Oui     | Oui     |                                                                                             |
| `gpx:bounds`             | Oui     | Oui     | Converti en LocationRect lors de la lecture.                                                    |
| `gpx:cmt`                | Oui     | Oui     |                                                                                             |
| `gpx:copyright`          | Oui     | Oui     |                                                                                             |
| `gpx:desc`               | Oui     | Oui     | Copié dans une propriété Description lors de la lecture pour s’aligner avec d’autres formats XML.               |
| `gpx:dgpsid`             | Oui     | Oui     |                                                                                             |
| `gpx:ele`                | Oui     | Oui     |                                                                                             |
| `gpx:extensions`         | partielles | partielles | Lors de la lecture, les informations de style sont extraites. Toutes les autres extensions seront aplaties en objet JSON simple. Seules les informations de style de la forme sont écrites. |
| `gpx:geoidheight`        | Oui     | Oui     |                                                                                             |
| `gpx:gpx`                | Oui     | Oui     |                                                                                             |
| `gpx:hdop`               | Oui     | Oui     |                                                                                             |
| `gpx:link`               | Oui     | Oui     |                                                                                             |
| `gpx:magvar`             | Oui     | Oui     |                                                                                             |
| `gpx:metadata`           | Oui     | Oui     |                                                                                             |
| `gpx:name`               | Oui     | Oui     |                                                                                             |
| `gpx:pdop`               | Oui     | Oui     |                                                                                             |
| `gpx:rte`                | Oui     | Oui     |                                                                                             |
| `gpx:rtept`              | Oui     | Oui     |                                                                                             |
| `gpx:sat`                | Oui     | Oui     |                                                                                             |
| `gpx:src`                | Oui     | Oui     |                                                                                             |
| `gpx:sym`                | Oui     | Oui     | La valeur est capturée, mais elle n’est pas utilisée pour modifier l’icône de punaise.                               |
| `gpx:text`               | Oui     | Oui     |                                                                                             |
| `gpx:time`               | Oui     | Oui     |                                                                                             |
| `gpx:trk`                | Oui     | Oui     |                                                                                             |
| `gpx:trkpt`              | Oui     | Oui     |                                                                                             |
| `gpx:trkseg`             | Oui     | Oui     |                                                                                             |
| `gpx:type`               | Oui     | Oui     |                                                                                             |
| `gpx:vdop`               | Oui     | Oui     |                                                                                             |
| `gpx:wpt`                | Oui     | Oui     |                                                                                             |
| `gpx_style:color`        | Oui     | Oui     |                                                                                             |
| `gpx_style:line`         | partielles | partielles | `color`, `opacity`, `width` et `lineCap` sont pris en charge.                                           |
| `gpx_style:opacity`      | Oui     | Oui     |                                                                                             |
| `gpx_style:width`        | Oui     | Oui     |                                                                                             |
| `gpxx:DisplayColor`      | Oui     | non      | Utilisé pour spécifier la couleur d’une forme. Lors de l’écriture, la couleur `gpx_style:line` est utilisée à la place.  |
| `gpxx:RouteExtension`    | partielles | non      | Toutes les propriétés sont lues dans `properties`. Seul `DisplayColor` est utilisé.                     |
| `gpxx:TrackExtension`    | partielles | non      | Toutes les propriétés sont lues dans `properties`. Seul `DisplayColor` est utilisé.                     |
| `gpxx:WaypointExtension` | partielles | non      | Toutes les propriétés sont lues dans `properties`. Seul `DisplayColor` est utilisé.                     |
| `gpx:keywords`           | Oui     | Oui     |                                                                                             |
| `gpx:fix`                | Oui     | Oui     |                                                                                             |

#### <a name="additional-notes"></a>Remarques supplémentaires

Lors de l’écriture ;

- Les multipoints sont divisés en points de cheminement individuels.
- Les polygones et les multipolygones sont écrits sous forme de chemins. 
  
## <a name="supported-well-known-text-geometry-types"></a>Types de géométrie Well Known Text pris en charge

| Type de géométrie | Lire | Write |
|--------------|:----:|:-----:|
| POINT | x | x |
| POINT Z | x | x | 
| POINT M | x | x<sup>[2]</sup> |
| POINT ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| LINESTRING | x | x |
| LINESTRING Z | x | x | 
| LINESTRING M | x | x<sup>[2]</sup> |
| LINESTRING ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| POLYGON | x | x |
| POLYGON Z | x | x |
| POLYGON M | x | x<sup>[2]</sup> |
| POLYGON ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| MULTIPOINT | x | x |
| MULTIPOINT Z | x | x | 
| MULTIPOINT M | x | x<sup>[2]</sup> |
| POMULTIPOINTINT ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| MULTILINESTRING | x | x |
| MULTILINESTRING Z | x | x | 
| MULTILINESTRING M | x | x<sup>[2]</sup> |
| MULTILINESTRING ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| MULTIPOLYGON | x | x |
| MULTIPOLYGON Z | x | x | 
| MULTIPOLYGON M | x | x<sup>[2]</sup> |
| MULTIPOLYGON ZM | x<sup>[1]</sup><sup>[2]</sup> | | 
| GEOMETRYCOLLECTION | x | x |
| GEOMETRYCOLLECTION Z | x | x | 
| GEOMETRYCOLLECTION M | x | x<sup>[2]</sup> | 
| GEOMETRYCOLLECTION ZM | x<sup>[1]</sup><sup>[2]</sup> | x | 

\[1\] Seul le paramètre Z est capturé et ajouté en tant que troisième valeur dans la valeur Position.

\[2\] Le paramètre M n’est pas capturé.

## <a name="delimited-spatial-data-support"></a>Prise en charge des données spatiales délimitées

Les données spatiales délimitées, telles que les fichiers de valeurs séparées par des virgules (CSV), ont souvent des colonnes qui contiennent des données spatiales. Par exemple, il peut y avoir des colonnes qui contiennent des informations de latitude et de longitude. Dans un format Well Known Text, il peut y avoir une colonne qui contient des données géométriques spatiales.

### <a name="spatial-data-column-detection"></a>Détection de colonne de données spatiales

Lors de la lecture d’un fichier délimité qui contient des données spatiales, l’en-tête est analysé pour déterminer quelles colonnes contiennent des champs d’emplacement. Si l’en-tête contient des informations de type, il sera utilisé pour caster les valeurs de cellule vers le type approprié. Si aucun en-tête n’est spécifié, la première ligne est analysée et utilisée pour générer un en-tête. Lors de l’analyse de la première ligne, un contrôle est exécuté pour faire correspondre les noms de colonnes avec les noms suivants sans respecter la casse. L’ordre des noms correspond à la priorité, au cas où plusieurs noms existent dans un fichier.

#### <a name="latitude"></a>Latitude

- `latitude`
- `lat`
- `latdd`
- `lat_dd`
- `latitude83`
- `latdecdeg`
- `y`
- `ycenter`
- `point-y`

#### <a name="longitude"></a>Longitude

- `longitude`
- `lon`
- `lng`
- `long`
- `longdd`
- `long_dd`
- `longitude83`
- `longdecdeg`
- `x`
- `xcenter`
- `point-x`

#### <a name="elevation"></a>Elevation

- `elevation`
- `elv`
- `altitude`
- `alt`
- `z`

#### <a name="geography"></a>Geography

La première ligne de données sera analysée pour rechercher les chaînes qui sont au format Well Known Text. 

### <a name="delimited-data-column-types"></a>Types de colonnes des données délimitées

Lors de l’analyse de la ligne d’en-tête, toutes les informations de type qui se trouvent dans le nom de la colonne sont extraites et utilisées pour caster les cellules de cette colonne. Voici un exemple de nom de colonne qui a une valeur de type : « ColumnName (typeName) ». Les noms de types suivants ne respectant pas la casse sont pris en charge :

#### <a name="numbers"></a>Nombres

- edm.int64
- int
- long
- edm.double
- float
- double
- nombre

#### <a name="booleans"></a>Valeurs booléennes

- edm.boolean
- bool
- boolean

#### <a name="dates"></a>Dates

- edm.datetime
- Date
- DATETIME

#### <a name="geography"></a>Geography

- edm.geography
- Geography

#### <a name="strings"></a>Chaînes

- edm.string
- varchar
- text
- string

Si aucune information de type ne peut être extraite de l’en-tête et que l’option de typage dynamique est activée lors de la lecture, chaque cellule est analysée individuellement pour déterminer le type de données le mieux adapté au cast.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

[Lire et écrire des données spatiales](spatial-io-read-write-spatial-data.md)
