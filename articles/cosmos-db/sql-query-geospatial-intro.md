---
title: Données de localisation géospatiales et GeoJSON dans Azure Cosmos DB
description: Découvrez comment créer des objets spatiaux avec Azure Cosmos DB et l’API SQL.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: a2e4e14927932575c9da42392329eea279f922fe
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336161"
---
# <a name="geospatial-and-geojson-location-data-in-azure-cosmos-db"></a>Données de localisation géospatiales et GeoJSON dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article est une introduction aux fonctionnalités géospatiales dans Azure Cosmos DB. Le stockage des données géospatiales et leur accès sont actuellement pris en charge par les comptes d’API SQL Azure Cosmos DB uniquement. Après avoir lu notre documentation sur l’indexation géospatiale, vous serez en mesure de répondre aux questions suivantes :

* Comment puis-je stocker des données spatiales dans Azure Cosmos DB ?
* Comment puis-je interroger des données géospaciales dans Azure Cosmos DB dans SQL et LINQ ?
* Comment puis-je activer ou désactiver l’indexation spatiale dans Azure Cosmos DB ?

## <a name="spatial-data-use-cases"></a>Cas d’usage de données spatiales

Les données géospatiales impliquent souvent des requêtes de proximité, comme « rechercher tous les cafés près de mon emplacement actuel ». Les cas d’usage courants sont les suivants :

* Analyses de géolocalisation, qui conduisent à des initiatives marketing spécifiques.
* Personnalisation en fonction de l’emplacement, pour plusieurs secteurs tels que la vente au détail et les soins de santé.
* Amélioration de la logistique, pour l’optimisation du transport.
* Analyse des risques, en particulier pour les compagnies d’assurance et sociétés de financement.
* Conscience situationnelle, pour les alertes et les notifications.

## <a name="introduction-to-spatial-data"></a>Présentation des données spatiales

Les données spatiales décrivent la position et la forme des objets dans l'espace. Dans la plupart des applications, ils correspondent aux objets sur terre et aux données géospatiales. Les données spatiales peuvent servir à représenter l'emplacement d'une personne, d'un point d'intérêt ou de la limite d'une ville ou un lac.

L’API SQL Azure Cosmos DB prend en charge deux types de données spatiales : les données de type **géométrique** et les données de type **géographique**.

- Le type **géométrique** représente des données dans un système de coordonnées euclidien (plat)
- Le type **geography** représente des données dans un système de coordonnées de monde sphérique.

## <a name="supported-data-types"></a>Types de données pris en charge

Azure Cosmos DB prend en charge l’indexation et l’interrogation des données de point géospatiales représentées à l’aide de la [spécification GeoJSON](https://tools.ietf.org/html/rfc7946). Les structures de données GeoJSON sont toujours des objets JSON valides, afin de pouvoir les stocker et les interroger à l’aide d’Azure Cosmos DB, sans bibliothèques ou outils spécialisés.

Azure Cosmos DB prend en charge les types de données spatiales suivants :

- Point
- LineString
- Polygone
- MultiPolygon

### <a name="points"></a>Points

Un **point** désigne une position unique dans l'espace. Dans les données géospatiales, un point représente l’emplacement exact, qui peut être une adresse postale d’une épicerie, un kiosque, une voiture ou une ville.  Un point est représenté dans GeoJSON (et Azure Cosmos DB) à l’aide de sa paire de coordonnées ou de longitude et latitude.

Voici un exemple JSON pour un point :

**Points dans Azure Cosmos DB**

```json
{
    "type":"Point",
    "coordinates":[ 31.9, -4.8 ]
}
```

Les types de données spatiales peuvent être incorporés dans un document Azure Cosmos DB, comme illustré dans cet exemple de profil utilisateur contenant des données de localisation :

**Utilisation de profil avec emplacement stocké dans Azure Cosmos DB**

```json
{
    "id":"cosmosdb-profile",
    "screen_name":"@CosmosDB",
    "city":"Redmond",
    "topics":[ "global", "distributed" ],
    "location":{
        "type":"Point",
        "coordinates":[ 31.9, -4.8 ]
    }
}
```

### <a name="points-in-a-geometry-coordinate-system"></a>Points dans un système de coordonnées géométriques

Pour le type de données **géométrique** , la spécification GeoJSON définit d’abord l’axe horizontal puis l’axe vertical.

### <a name="points-in-a-geography-coordinate-system"></a>Points dans un système de coordonnées géographiques

Pour le type de données **geography** , la spécification GeoJSON spécifie d’abord la longitude, puis la latitude. Comme dans d'autres applications de mappage, la longitude et la latitude sont des angles et sont exprimées en degrés. Les valeurs de longitude sont mesurées à partir du premier méridien et sont comprises entre -180 degrés et 180 degrés. Les valeurs de latitude sont mesurées à partir de l’Équateur et sont comprises entre -90 degrés et 90 degrés.

Azure Cosmos DB interprète les coordonnées représentées par le système de référence WGS-84. Voir ci-dessous pour plus d’informations sur les systèmes de coordonnées de référence.

### <a name="linestrings"></a>LineStrings

**LineStrings** représentent une série de deux ou plusieurs points dans l'espace et les segments de ligne qui les connectent. Dans les données géographiques, les LineStrings sont généralement utilisées pour représenter les autoroutes ou les cours d’eau.

**LineStrings dans GeoJSON**

```json
    "type":"LineString",
    "coordinates":[ [
        [ 31.8, -5 ],
        [ 31.8, -4.7 ]
    ] ]
```

### <a name="polygons"></a>Polygones

Un **polygone** est une limite de points reliés qui constitue une LineString fermée. Les polygones sont couramment utilisés pour représenter des formations naturelles comme des lacs ou des juridictions politiques comme les villes et les États. Voici un exemple de polygone dans Azure Cosmos DB :

**Polygones dans GeoJSON**

```json
{
    "type":"Polygon",
    "coordinates":[ [
        [ 31.8, -5 ],
        [ 31.8, -4.7 ],
        [ 32, -4.7 ],
        [ 32, -5 ],
        [ 31.8, -5 ]
    ] ]
}
```

> [!NOTE]
> La spécification GeoJSON impose que, pour les polygones valides, la dernière paire de coordonnées fournie soit identique à la première, pour créer une forme fermée.
>
> Les points dans un polygone doivent être spécifiés dans le sens antihoraire. Un polygone spécifié dans le sens horaire représente l’inverse de la région qu’il contient.
>
>

### <a name="multipolygons"></a>Multipolygones

Un **multipolygone** est un tableau de zéro polygone ou plus. Les **multipolygones** ne peuvent pas avoir de zone en commun et leurs côtés ne peuvent pas se chevaucher. Ils peuvent se toucher en un ou plusieurs points.

**Multipolygones dans GeoJSON**

```json
{
    "type":"MultiPolygon",
    "coordinates":[[[
        [52.0, 12.0],
        [53.0, 12.0],
        [53.0, 13.0],
        [52.0, 13.0],
        [52.0, 12.0]
        ]],
        [[
        [50.0, 0.0],
        [51.0, 0.0],
        [51.0, 5.0],
        [50.0, 5.0],
        [50.0, 0.0]
        ]]]
}
```

## <a name="coordinate-reference-systems"></a>Coordination des systèmes de référence

Étant donné que la forme de la terre est irrégulière, les coordonnées des données géospatiales de type geography sont représentées dans de nombreux systèmes de coordonnées de référence (CRS), chacun ayant ses propres images de référence et unités de mesure. Par exemple, le « National Grid of Britain » est un système de référence précis pour le Royaume-Uni, mais en dehors.

Le CRS moderne le plus populaire est le World Geodetic System [WGS-84](https://earth-info.nga.mil/GandG/update/index.php). Les périphériques GPS et de nombreux services de mappage, notamment les API Bing Maps et Google Maps, utilisent le WGS-84. Azure Cosmos DB prend en charge l’indexation et l’interrogation de données géospatiales de type geography uniquement à l’aide du CRS WGS-84.

## <a name="creating-documents-with-spatial-data"></a>Création de documents avec les données spatiales
Lorsque vous créez des documents qui contiennent des valeurs GeoJSON, ils sont automatiquement indexés avec un index spatial conformément à la stratégie d'indexation du conteneur. Si vous travaillez avec un Kit de développement logiciel (SDK) Azure Cosmos DB dans un langage saisi dynamiquement comme Python ou Node.js, vous devez créer un GeoJSON valide.

**Création d’un document avec les données géographiques dans Node.js**

```javascript
var userProfileDocument = {
    "id":"cosmosdb",
    "location":{
        "type":"Point",
        "coordinates":[ -122.12, 47.66 ]
    }
};

client.createDocument(`dbs/${databaseName}/colls/${collectionName}`, userProfileDocument, (err, created) => {
    // additional code within the callback
});
```

Si vous travaillez avec les API SQL, vous pouvez utiliser les classes `Point`, `LineString`, `Polygon` et `MultiPolygon` dans l’espace de noms `Microsoft.Azure.Cosmos.Spatial` pour incorporer des informations de localisation dans des objets de votre application. Ces classes permettent de simplifier la sérialisation et la désérialisation de données spatiales dans GeoJSON.

**Création d’un document avec les données géographiques dans .NET**

```csharp
using Microsoft.Azure.Cosmos.Spatial;

public class UserProfile
{
    [JsonProperty("id")]
    public string id { get; set; }

    [JsonProperty("location")]
    public Point Location { get; set; }

    // More properties
}

await container.CreateItemAsync( new UserProfile
    {
        id = "cosmosdb",
        Location = new Point (-122.12, 47.66)
    });
```

Si vous n'avez pas les informations de latitude et de longitude, mais disposez des adresses physiques ou du nom d'emplacement comme la ville ou le pays/la région, vous pouvez rechercher les coordonnées réelles à l'aide d'un service de géocodage comme Bing Maps REST Services. En savoir plus sur le géocodage de Bing Maps [ici](/bingmaps/rest-services/).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment démarrer avec la prise en charge géospatiale dans Azure Cosmos DB, vous pouvez :

* En savoir plus sur les [requêtes Azure Cosmos DB](sql-query-getting-started.md)
* En savoir plus sur l’[Interrogation de données spatiales avec Azure Cosmos DB](sql-query-geospatial-query.md)
* En savoir plus sur la façon d’[Indexer des données spatiales avec Azure Cosmos DB](sql-query-geospatial-index.md)