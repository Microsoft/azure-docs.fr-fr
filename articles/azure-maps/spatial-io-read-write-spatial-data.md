---
title: Lire et écrire des données spatiales | Microsoft Azure Maps
description: Découvrez comment lire et écrire des données à l’aide du module d’E/S spatiales, fourni par le Kit de développement logiciel (SDK) web Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/01/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 133674e6764e12742f5b238946e943d9b5011cd2
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891327"
---
# <a name="read-and-write-spatial-data"></a>Lire et écrire des données spatiales

Le tableau ci-dessous répertorie les formats de fichier spatial pris en charge pour la lecture et l’écriture d’opérations avec le module d’E/S spatiales.

| Format de données       | Lire | Write |
|-------------------|------|-------|
| GeoJSON           | ✓  |  ✓  |
| GeoRSS            | ✓  |  ✓  |
| GML               | ✓  |  ✓  |
| GPX               | ✓  |  ✓  |
| KML               | ✓  |  ✓  |
| KMZ               | ✓  |  ✓  |
| Spatial CSV       | ✓  |  ✓  |
| Well-Known Text   | ✓  |  ✓  |

Les sections suivantes décrivent tous les différents outils de lecture et d’écriture de données spatiales à l’aide du module d’E/S spatiales.

## <a name="read-spatial-data"></a>Lire des données spatiales

La fonction `atlas.io.read` est la fonction principale utilisée pour lire des formats courants de données spatiales tels que des fichiers KML, GPX, GeoRSS, GeoJSON et CSV avec des données spatiales. Cette fonction peut également lire les versions compressées de ces formats, sous la forme d’un fichier zip ou d’un fichier KMZ. Le format de fichier KMZ est une version compressée de KML qui peut également inclure des ressources telles que des images. La fonction de lecture peut également prendre une URL qui pointe vers un fichier dans l’un de ces formats. Les URL doivent être hébergées sur un point de terminaison CORS, sinon un service proxy doit être fourni dans les options de lecture. Le service proxy est utilisé pour charger des ressources sur des domaines qui ne sont pas compatibles avec CORS. La fonction de lecture retourne une promesse d’ajouter les icônes d’image à la carte et traite les données de façon asynchrone afin de réduire l’impact sur le thread d’interface utilisateur.

Lors de la lecture d’un fichier compressé, qu’il s’agisse d’un fichier zip ou KMZ, celui-ci est décompressé et analysé pour le premier fichier valide. Par exemple, doc.kml ou un fichier avec une autre extension valide, telle que : .kml, .xml, .geojson, .json, .csv, .tsv ou .txt. Ensuite, les images référencées dans les fichiers KML et GeoRSS sont préchargées pour s’assurer qu’elles sont accessibles. Les données d’image inaccessibles peuvent charger une autre image de secours ou seront supprimées des styles. Les images extraites des fichiers KMZ seront converties en URI de données.

Le résultat de la fonction de lecture est un objet `SpatialDataSet`. Cet objet étend la classe FeatureCollection GeoJSON. Il peut facilement être passé dans une `DataSource` tel quel pour afficher ses fonctionnalités sur une carte. Le `SpatialDataSet` contient non seulement des informations sur les fonctionnalités, mais il peut également inclure des superpositions de sol KML, des mesures de traitement et d’autres détails comme indiqué dans le tableau suivant.

| Nom de la propriété | Type | Description | 
|---------------|------|-------------|
| `bbox` | `BoundingBox` | Cadre englobant de toutes les données dans le jeu de données. |
| `features` | `Feature[]` | Fonctionnalités GeoJSON dans le jeu de données. |
| `groundOverlays` | `(atlas.layer.ImageLayer | atlas.layers.OgcMapLayer)[]` | Tableau de superpositions de sol KML. |
| `icons` | Record&lt;string, string&gt; | Ensemble d’URL d’icône. Clé = nom de l’icône, valeur = URL. |
| properties | n'importe laquelle | Informations de propriété fournies au niveau du document d’un jeu de données spatiales. |
| `stats` | `SpatialDataSetStats` | Statistiques relatives au contenu et au temps de traitement d’un jeu de données spatiales. |
| `type` | `'FeatureCollection'` | Valeur de type GeoJSON en lecture seule. |

## <a name="examples-of-reading-spatial-data"></a>Exemples de lecture des données spatiales

Le code suivant montre comment lire un jeu de données spatiales et le restituer sur la carte à l’aide de la classe `SimpleDataLayer`. Le code utilise un fichier GPX pointé par une URL.

<br/>

<iframe height='500' scrolling='no' title='Charger des données spatiales simples' src='//codepen.io/azuremaps/embed/yLNXrZx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/yLNXrZx/'>Load Spatial Data Simple</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

La démonstration de code suivante montre comment lire et charger un fichier KML ou KMZ sur la carte. Un fichier KML peut contenir des superpositions de sol, qui se présentent sous la forme `ImageLyaer` ou `OgcMapLayer`. Ces superpositions doivent être ajoutées à la carte séparément des fonctionnalités. En outre, si le jeu de données a des icônes personnalisées, ces icônes doivent être chargées dans les ressources de cartes avant le chargement des fonctionnalités.

<br/>

<iframe height='500' scrolling='no' title='Charger un fichier KML sur la carte' src='//codepen.io/azuremaps/embed/XWbgwxX/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/XWbgwxX/'>Load KML Onto Map</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Vous pouvez éventuellement fournir un service proxy pour accéder aux ressources interdomaines pour lesquelles CORS n’est pas activé. La fonction de lecture essaiera d’abord d’accéder aux fichiers d’un autre domaine à l’aide de CORS. Après l’échec de la première tentative d’accès à une ressource d’un autre domaine à l’aide de CORS, elle ne demandera des fichiers supplémentaires que si un service proxy a été fourni. La fonction de lecture ajoute l’URL du fichier à la fin de l’URL de proxy fournie. Cet extrait de code indique comment passer un service proxy à la fonction de lecture :

```javascript
//Read a file from a URL or pass in a raw data as a string.
atlas.io.read('https://nonCorsDomain.example.com/mySuperCoolData.xml', {
    //Provide a proxy service
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
}).then(async r => {
    if (r) {
        // Some code goes here . . .
    }
});

```

La démonstration ci-dessous montre comment lire un fichier délimité et l’afficher sur la carte. Dans ce cas, le code utilise un fichier CSV qui contient des colonnes de données spatiales.

<br/>

<iframe height='500' scrolling='no' title='Ajouter un fichier délimité' src='//codepen.io/azuremaps/embed/ExjXBEb/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/ExjXBEb/'>Add a Delimited File</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="write-spatial-data"></a>Écrire des données spatiales

Il existe deux fonctions d’écriture principales dans le module d’E/S spatiales. La fonction `atlas.io.write` génère une chaîne, tandis que la fonction `atlas.io.writeCompressed` génère un fichier zip compressé. Le fichier zip compressé contient un fichier texte contenant les données spatiales. Ces deux fonctions retournent une promesse d’ajouter les données au fichier. Elles peuvent aussi écrire les données suivantes : `SpatialDataSet`, `DataSource`, `ImageLayer`, `OgcMapLayer`, la collection de fonctionnalités, la fonctionnalité, la géométrie ou un tableau de n’importe quelle combinaison de ces types de données. Lors de l’écriture à l’aide de l’une de ces fonctions, vous pouvez spécifier le format de fichier souhaité. Si le format de fichier n’est pas spécifié, les données sont écrites au format KML.

L’outil ci-dessous illustre la majorité des options d’écriture qui peuvent être utilisées avec la fonction `atlas.io.write`.

<br/>

<iframe height='700' scrolling='no' title='Options d’écriture de données spatiales' src='//codepen.io/azuremaps/embed/YzXxXPG/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/YzXxXPG/'>Spatial data write options</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="example-of-writing-spatial-data"></a>Exemple d’écriture de données spatiales

L’exemple suivant vous permet de glisser-déplacer, puis de charger des fichiers spatiaux sur la carte. Vous pouvez exporter des données GeoJSON de la carte et les écrire dans l’un des formats de données spatiales pris en charge sous la forme d’une chaîne ou d’un fichier compressé.

<br/>

<iframe height='700' scrolling='no' title='Glisser-déplacer des fichiers spatiaux sur la carte' src='//codepen.io/azuremaps/embed/zYGdGoO/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/zYGdGoO/'>Drag and drop spatial files onto map</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Vous pouvez éventuellement fournir un service proxy pour accéder aux ressources interdomaines pour lesquelles CORS n’est pas activé. Cet extrait de code montre que vous pouvez incorporer un service proxy :

```javascript
atlas.io.read(data, {
    //Provide a proxy service
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
}).then(
    //Success
    function(r) {
        //some code goes here ...
    }
);
```

## <a name="read-and-write-well-known-text-wkt"></a>Lire et écrire au format WKT (Well-Known Text)

[WKT](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (Well-Known Text) est une norme de l’Open Geospatial Consortium (OGC) utilisée pour représenter les géométries spatiales sous forme de texte. De nombreux systèmes géospatiaux prennent en charge le format WKT, comme Azure SQL et Azure PostgreSQL à l’aide du plug-in PostGIS. Comme la plupart des normes OGC, les coordonnées sont formatées en « longitude latitude » pour s’aligner sur la convention « x y ». À titre d’exemple, un point de longitude -110 et de latitude 45 peut être écrit `POINT(-110 45)` à l’aide du format WKT.

Le format Well-known text peut être lu à l’aide de la fonction `atlas.io.ogc.WKT.read` et écrit à l’aide de la fonction `atlas.io.ogc.WKT.write`.

## <a name="examples-of-reading-and-writing-well-known-text-wkt"></a>Exemples de lecture et d’écriture de Well-Known Text (WKT)

Le code suivant montre comment lire la chaîne Well-known text `POINT(-122.34009 47.60995)` et l’afficher sur la carte à l’aide d’une couche de bulles.

<br/>

<iframe height='500' scrolling='no' title='Lire au format Well-Known Text' src='//codepen.io/azuremaps/embed/XWbabLd/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/XWbabLd/'>Read Well-Known Text</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

Le code suivant illustre la lecture et l’écriture de Well-known text dans les deux sens.

<br/>

<iframe height='700' scrolling='no' title='Lire et écrire au format Well-Known Text' src='//codepen.io/azuremaps/embed/JjdyYav/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez l’extrait de code <a href='https://codepen.io/azuremaps/pen/JjdyYav/'>Read and write Well-Known Text</a> d’Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="read-and-write-gml"></a>Lire et écrire un fichier GML

GML est une spécification de fichier XML spatial qui est souvent utilisée comme extension d’autres spécifications XML. Les données GeoJSON peuvent être écrites au format XML avec des balises GML à l’aide de la fonction `atlas.io.core.GmlWriter.write`. Le fichier XML qui contient GML peut être lu à l’aide de la fonction `atlas.io.core.GmlReader.read`. La fonction de lecture a deux options :

- L’option `isAxisOrderLonLat` : l’ordre des coordonnées « latitude, longitude » ou « longitude, latitude » peut varier selon les jeux de données et n’est pas toujours bien défini. Par défaut, le lecteur GML lit les données de coordonnée au format « latitude, longitude », mais si vous attribuez la valeur true à cette option, la valeur est « longitude, latitude ».
- L’option `propertyTypes` : cette option est une table de choix de valeurs de clé où la clé correspond au nom d’une propriété dans le jeu de données. La valeur est le type d’objet vers lequel caster la valeur lors de l’analyse. Les valeurs de type prises en charge sont les suivantes : `string`, `number`, `boolean` et `date`. Si une propriété ne figure pas dans la table de choix ou si le type n’est pas défini, la propriété est analysée en tant que chaîne.

La fonction `atlas.io.read` utilise par défaut la fonction `atlas.io.core.GmlReader.read` lorsqu’elle détecte que les données d’entrée sont au format XML, mais que les données ne correspondent pas à l’un des autres formats XML spatiaux pris en charge.

Le fonction `GmlReader` analyse les coordonnées qui ont l’un des SRID suivants :

- EPSG: 4326 (recommandé)
- EPSG:4269, EPSG:4283, EPSG:4258, EPSG:4308, EPSG:4230, EPSG:4272, EPSG:4271, EPSG:4267, EPSG:4608, EPSG:4674 éventuellement avec une petite marge d’erreur.
- EPSG:3857, EPSG:102100, EPSG:3785, EPSG:900913, EPSG:102113, EPSG:41001, EPSG:54004

## <a name="more-resources"></a>Plus de ressources

En savoir plus sur les classes et les méthodes utilisées dans cet article :

[Fonctions statiques atlas.io](/javascript/api/azure-maps-spatial-io/atlas.io)

[SpatialDataSet](/javascript/api/azure-maps-spatial-io/atlas.spatialdataset)

[SpatialDataSetStats](/javascript/api/azure-maps-spatial-io/atlas.spatialdatasetstats)

[GmlReader](/javascript/api/azure-maps-spatial-io/atlas.io.core.gmlreader)

[GmlWriter](/javascript/api/azure-maps-spatial-io/atlas.io.core.gmlwriter)

[Fonctions atlas.io.ogc.WKT](/javascript/api/azure-maps-spatial-io/atlas.io.ogc.wkt)

[Se connecter à un service WFS](spatial-io-connect-wfs-service.md)

[Exploiter les opérations de base](spatial-io-core-operations.md)

[Détails sur les formats de données pris en charge](spatial-io-supported-data-format-details.md)


## <a name="next-steps"></a>Étapes suivantes

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

[Ajouter une couche OGC](spatial-io-add-ogc-map-layer.md)