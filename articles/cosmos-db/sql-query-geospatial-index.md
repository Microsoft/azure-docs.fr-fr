---
title: Indexer des données géospatiales avec Azure Cosmos DB
description: Indexer des données spatiales avec Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: tisande
ms.openlocfilehash: 47eedf1ddbb155180d364c42ec179b3e01279e44
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336212"
---
# <a name="index-geospatial-data-with-azure-cosmos-db"></a>Indexer des données géospatiales avec Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Nous avons conçu le moteur de base de données d’Azure Cosmos DB pour être véritablement indépendant du schéma et assurer une prise en charge de JSON de première qualité. Le moteur de base de données optimisé en écriture d’Azure Cosmos DB comprend les données spatiales représentées dans la norme GeoJSON en mode natif.

En bref, la géométrie est projetée à partir des coordonnées géodésiques sur un plan 2D, puis divisée progressivement en cellules à l'aide un **quadtree**. Ces cellules sont mappées en 1D selon l'emplacement de la cellule dans une **courbe de remplissage d'espace de Hilbert** qui permet de préserver la localité des points. En outre, quand les données d’emplacement sont indexées, elles passent par un processus connu sous le nom de **pavage** , c’est-à-dire que toutes les cellules qui se croisent à un emplacement sont identifiées et stockées en tant que clés dans l’index Azure Cosmos DB. Au moment de la requête, des arguments comme les points et les polygones sont également fractionnés pour extraire les plages d’ID de cellule appropriées, puis utilisés pour récupérer des données à partir de l’index.

Si vous spécifiez une stratégie d’indexation comportant un index spatial pour `/*` (tous les chemins), toutes les données trouvées dans le conteneur sont indexées de façon à optimiser l’efficacité des requêtes spatiales.

> [!NOTE]
> Azure Cosmos DB prend en charge l’indexation des types Point, LineString, Polygon et MultiPolygon. Si vous indexez l’un d’eux, nous indexerons automatiquement tous les autres. Par exemple, si vous indexez des types Polygon, nous indexerons également les types Point, LineString et MultiPolygon. L’indexation d’un nouveau type spatial n’a pas d’incidence sur la charge des unités de requête en écriture ni la taille de l’index, sauf si vous disposez de données GeoJSON valides de ce type.

## <a name="modifying-geospatial-configuration"></a>Modification de la configuration géospatiale

Dans le conteneur, la **Configuration géospatiale** spécifie le mode d’indexation des données spatiales. Spécifiez une **Configuration géospatiale** par conteneur : géographie ou géométrie.

Vous pouvez basculer entre les types spatiaux **géographie** et **géométrie** dans le portail Azure. Avant de basculer vers le type spatial géométrie, il est important de créer une [stratégie d’indexation de géométrie spatiale valide avec un cadre englobant](#geometry-data-indexing-examples).

Voici comment définir la **configuration géospatiale** dans l’ **Explorateur de données** au sein du portail Azure :

:::image type="content" source="./media/sql-query-geospatial-index/geospatial-configuration.png" alt-text="Définition de la configuration géospatiale":::

Vous pouvez également modifier la `geospatialConfig` dans le kit de développement logiciel (SDK) .NET pour ajuster la **Configuration géospatiale**  :

Si `geospatialConfig` n’est pas spécifiée, le type de données géographie est utilisé par défaut. Lorsque vous modifiez `geospatialConfig`, toutes les données géospatiales présentes dans le conteneur sont réindexées.

Voici un exemple de changement de type de données géospatiales pour `geometry`. La propriété `geospatialConfig` est définie et un **boundingBox** (cadre englobant) est ajouté pour passer au type géométrie :

```csharp
    //Retrieve the container's details
    ContainerResponse containerResponse = await client.GetContainer("db", "spatial").ReadContainerAsync();
    //Set GeospatialConfig to Geometry
    GeospatialConfig geospatialConfig = new GeospatialConfig(GeospatialType.Geometry);
    containerResponse.Resource.GeospatialConfig = geospatialConfig;
    // Add a spatial index including the required boundingBox
    SpatialPath spatialPath = new SpatialPath
            {  
                Path = "/locations/*",
                BoundingBox = new BoundingBoxProperties(){
                    Xmin = 0,
                    Ymin = 0,
                    Xmax = 10,
                    Ymax = 10
                }
            };
    spatialPath.SpatialTypes.Add(SpatialType.Point);
    spatialPath.SpatialTypes.Add(SpatialType.LineString);
    spatialPath.SpatialTypes.Add(SpatialType.Polygon);
    spatialPath.SpatialTypes.Add(SpatialType.MultiPolygon);

    containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(spatialPath);

    // Update container with changes
    await client.GetContainer("db", "spatial").ReplaceContainerAsync(containerResponse.Resource);
```

## <a name="geography-data-indexing-examples"></a>Exemples d’indexation des données Geography

L’extrait de code JSON suivant montre une stratégie d’indexation avec l’indexation spatiale activée pour le type de données **Geography**. Il est valide pour les données spatiales ayant le type de données Geography et indexe toute instance Point, Polygon, MultiPolygon ou LineString GeoJSON trouvée dans les documents pour la requête spatiale. Si vous modifiez la stratégie d’indexation à l’aide du Portail Azure, vous pouvez spécifier le JSON suivant pour la stratégie d’indexation afin d’activer l’indexation spatiale pour votre conteneur :

**JSON de stratégie d’indexation de conteneur avec indexation spatiale géographique**

```json
{
    "automatic": true,
    "indexingMode": "Consistent",
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/*",
            "types": [
                "Point",
                "Polygon",
                "MultiPolygon",
                "LineString"
            ]
        }
    ],
    "excludedPaths": []
}
```

> [!NOTE]
> Si la valeur GeoJSON d'emplacement dans le document est incorrecte ou non valide, elle n’est pas indexée pour les requêtes spatiales. Vous pouvez valider les valeurs d'emplacement à l'aide de ST_ISVALID et ST_ISVALIDDETAILED.

Vous pouvez également [modifier la stratégie d’indexation](how-to-manage-indexing-policy.md) à l’aide d’Azure CLI, de PowerShell ou de n’importe quel Kit de développement logiciel (SDK).

## <a name="geometry-data-indexing-examples"></a>Exemples d’indexation des données géométrie

Avec le type de données **géométrie** comme avec géographie, il est nécessaire de préciser les chemins et les types à indexer. Vous devez également spécifier un `boundingBox` dans la stratégie d’indexation afin d’indiquer la zone à indexer pour ce chemin spécifique. Chaque chemin géospatial doit avoir son propre `boundingBox`.

Le cadre englobant présente les propriétés suivantes :

- **xmin** : coordonnée x indexée minimale ;
- **ymin** : coordonnée y indexée minimale ;
- **xmax** : coordonnée x indexée maximale ;
- **ymax** : coordonnée y indexée maximale.

Un cadre englobant est nécessaire, car les données géométriques occupent un plan qui peut être infini. Les index spatiaux, eux, ont besoin d’un espace fini. Pour le type de données **géographie** , la Terre fait office de limite ; il n’est donc pas nécessaire de définir un cadre englobant.

Créez un cadre englobant contenant la totalité (ou la plupart) de vos données. Seules les opérations calculées sur les objets qui se trouvent entièrement à l’intérieur du cadre englobant pourront avoir recours à l’index spatial. Le fait de définir un cadre englobant plus grand que nécessaire aura un impact négatif sur les performances des requêtes.

Voici un exemple de stratégie d’indexation qui indexe des données **géométrie** avec **geospatialConfig** défini sur `geometry` :

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/\"_etag\"/?"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/locations/*",
            "types": [
                "Point",
                "LineString",
                "Polygon",
                "MultiPolygon"
            ],
            "boundingBox": {
                "xmin": -10,
                "ymin": -20,
                "xmax": 10,
                "ymax": 20
            }
        }
    ]
}
```

La stratégie d’indexation ci-dessus a un **boundingBox** de (-10, 10) pour les coordonnées x et (-20, 20) pour les coordonnées y. Le conteneur présentant la stratégie d’indexation ci-dessus indexera tous les Point, Polygon, MultiPolygon et LineString qui se trouvent entièrement dans cette région.

> [!NOTE]
> Si vous tentez d’ajouter une stratégie d’indexation comportant un **boundingBox** à un conteneur du type de données `geography`, l’opération échouera. Vous devez définir **geospatialConfig** sur `geometry` pour le conteneur avant d’ajouter un **boundingBox**. Vous pouvez ajouter des données et modifier le reste de votre stratégie d’indexation (notamment les chemins et les types) avant ou après avoir sélectionné le type de données géospatiales du conteneur.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment démarrer avec la prise en charge géospatiale dans Azure Cosmos DB, vous pouvez :

* En savoir plus sur les [requêtes Azure Cosmos DB](sql-query-getting-started.md)
* En savoir plus sur l’[Interrogation de données spatiales avec Azure Cosmos DB](sql-query-geospatial-query.md)
* En savoir plus sur les [données de localisation géospatiales et GeoJSON dans Azure Cosmos DB](sql-query-geospatial-intro.md)
