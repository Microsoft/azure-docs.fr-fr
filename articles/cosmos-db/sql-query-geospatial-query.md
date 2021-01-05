---
title: Interrogation de données géospatiales avec Azure Cosmos DB
description: Interrogation de données spatiales avec Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: tisande
ms.openlocfilehash: bb9a0351b6f1de47f3687995c65060a23bdb2874
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336110"
---
# <a name="querying-geospatial-data-with-azure-cosmos-db"></a>Interrogation de données géospatiales avec Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment interroger des données géospatiales dans Azure Cosmos DB avec SQL et LINQ. Le stockage des données géospatiales et leur accès sont actuellement pris en charge par les comptes d’API SQL Azure Cosmos DB uniquement. Azure Cosmos DB prend en charge les fonctions intégrées Open Geospatial Consortium (OGC) suivantes pour les requêtes géospatiales. Pour plus d’informations sur l’ensemble complet de fonctions intégrées dans le langage SQL, consultez [Fonctions système de requête dans Azure Cosmos DB](sql-query-system-functions.md).

## <a name="spatial-sql-built-in-functions"></a>Fonctions spatiales SQL intégrées

Voici une liste de fonctions système géospatiales utiles pour l’interrogation dans Azure Cosmos DB :

|**Utilisation**|**Description**|
|---|---|
| ST_DISTANCE (spatial_expr, spatial_expr) | Retourne la distance entre les deux expressions Point, Polygone ou LineString de GeoJSON.|
|ST_WITHIN (spatial_expr, spatial_expr) | Renvoie une expression booléenne indiquant si le premier objet GeoJSON (Point, Polygone ou LineString) se trouve dans le second objet GeoJSON (Point, Polygone ou LineString).|
|ST_INTERSECTS (spatial_expr, spatial_expr)| Retourne une expression booléenne indiquant si les deux objets GeoJSON spécifiés (Point, Polygone ou LineString) se croisent.|
|ST_ISVALID| Renvoie une valeur booléenne indiquant si l’expression Point, Polygone ou LineString GeoJSON spécifiée est valide.|
| ST_ISVALIDDETAILED| Retourne une valeur JSON contenant une valeur booléenne si l’expression Point, Polygone ou LineString GeoJSON spécifiée est valide. Si elle est non valide, elle retourne le motif sous la forme d’une valeur de chaîne.|

Les fonctions spatiales peuvent être utilisées pour effectuer des requêtes de proximité par rapport aux données spatiales. Par exemple, voici une requête qui retourne tous les documents de famille se trouvant dans un rayon de 30 kilomètres de la localisation spécifiée à l’aide de la fonction intégrée `ST_DISTANCE`.

**Requête**

```sql
    SELECT f.id
    FROM Families f
    WHERE ST_DISTANCE(f.location, {"type": "Point", "coordinates":[31.9, -4.8]}) < 30000
```

**Résultats**

```json
    [{
      "id": "WakefieldFamily"
    }]
```

Si vous incluez l'indexation spatiale dans votre stratégie d'indexation, les « requêtes à distance » seront servies efficacement dans l'index. Pour plus d’informations sur l’indexation spatiale, consultez la section sur l’[indexation géospatiale](sql-query-geospatial-index.md). Si vous n’avez pas d’index spatial pour les chemins spécifiés, la requête effectue une analyse du conteneur.

`ST_WITHIN` peut être utilisé pour vérifier si un point se trouve dans un polygone. Généralement, les polygones sont utilisés pour représenter des limites comme les codes postaux, les frontières d’États ou les formations naturelles. Si vous incluez l'indexation spatiale dans votre stratégie d'indexation, les requêtes « within » seront servies efficacement dans l'index.

Les arguments de polygone dans `ST_WITHIN` ne peuvent contenir qu’un seul cercle. Cela signifie que les polygones ne doivent pas contenir de trous.

**Requête**

```sql
    SELECT *
    FROM Families f
    WHERE ST_WITHIN(f.location, {
        "type":"Polygon",
        "coordinates": [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]
    })
```

**Résultats**

```json
    [{
      "id": "WakefieldFamily",
    }]
```

> [!NOTE]
> Tout comme pour les types non correspondants dans une requête Azure Cosmos DB, si la valeur de l’emplacement spécifié dans un argument est incorrecte ou non valide, elle prend alors la valeur **indéfinie** et le document évalué est ignoré des résultats de requête. Si votre requête ne retourne aucun résultat, exécutez `ST_ISVALIDDETAILED` afin de déboguer l’absence de validité du type spatial.
>
>

Azure Cosmos DB prend également en charge les requêtes inversées. Vous pouvez par exemple indexer des polygones ou des lignes dans Azure Cosmos DB, puis interroger les zones qui contiennent un point spécifique. Ce modèle est généralement utilisé dans la logistique, par exemple pour identifier quand un camion entre dans une région spécifique ou la quitte.

**Requête**

```sql
    SELECT *
    FROM Areas a
    WHERE ST_WITHIN({"type": "Point", "coordinates":[31.9, -4.8]}, a.location)
```

**Résultats**

```json
    [{
      "id": "MyDesignatedLocation",
      "location": {
        "type":"Polygon",
        "coordinates": [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]
      }
    }]
```

`ST_ISVALID` et `ST_ISVALIDDETAILED` peuvent être utilisés pour vérifier si un objet spatial est valide. Par exemple, la requête suivante vérifie la validité d'un point avec une valeur de latitude hors limites (-132.8). `ST_ISVALID` retourne simplement une valeur booléenne et `ST_ISVALIDDETAILED` retourne la valeur booléenne et une chaîne contenant la raison pour laquelle il est non valide.

**Requête**

```sql
    SELECT ST_ISVALID({ "type": "Point", "coordinates": [31.9, -132.8] })
```

**Résultats**

```json
    [{
      "$1": false
    }]
```

Vous pouvez aussi utiliser ces fonctions pour valider des polygones. Par exemple, nous utilisons ici `ST_ISVALIDDETAILED` pour valider un polygone non fermé.

**Requête**

```sql
    SELECT ST_ISVALIDDETAILED({ "type": "Polygon", "coordinates": [[ 
        [ 31.8, -5 ], [ 31.8, -4.7 ], [ 32, -4.7 ], [ 32, -5 ] 
        ]]})
```

**Résultats**

```json
    [{
       "$1": { 
            "valid": false, 
            "reason": "The Polygon input is not valid because the start and end points of the ring number 1 are not the same. Each ring of a Polygon must have the same start and end points." 
          }
    }]
```

## <a name="linq-querying-in-the-net-sdk"></a>Interrogation LINQ dans le SDK .NET

Le SDK .NET SQL fournit également les méthodes de stub `Distance()` et `Within()` pour une utilisation dans des expressions LINQ. Le fournisseur SQL LINQ traduit ces appels de méthode en appels de fonction intégrés SQL équivalents (ST_DISTANCE et ST_WITHIN, respectivement).

Voici un exemple de requête LINQ qui recherche tous les documents du conteneur Azure Cosmos dont la valeur `location` se trouve dans un rayon de 30 kilomètres du point spécifié à l’aide de LINQ.

**Requête LINQ de distance**

```csharp
    foreach (UserProfile user in container.GetItemLinqQueryable<UserProfile>(allowSynchronousQueryExecution: true)
        .Where(u => u.ProfileType == "Public" && u.Location.Distance(new Point(32.33, -4.66)) < 30000))
    {
        Console.WriteLine("\t" + user);
    }
```

De même, voici une requête permettant de rechercher tous les documents dont la valeur `location` est dans la zone/le polygone spécifié.

**Requête LINQ within**

```csharp
    Polygon rectangularArea = new Polygon(
        new[]
        {
            new LinearRing(new [] {
                new Position(31.8, -5),
                new Position(32, -5),
                new Position(32, -4.7),
                new Position(31.8, -4.7),
                new Position(31.8, -5)
            })
        });

    foreach (UserProfile user in container.GetItemLinqQueryable<UserProfile>(allowSynchronousQueryExecution: true)
         .Where(a => a.Location.Within(rectangularArea)))
    {
        Console.WriteLine("\t" + user);
    }
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment démarrer avec la prise en charge géospatiale dans Azure Cosmos DB, vous pouvez :

* En savoir plus sur les [requêtes Azure Cosmos DB](sql-query-getting-started.md)
* En savoir plus sur l’[utilisation de données de localisation géospatiales et GeoJSON](sql-query-geospatial-intro.md) dans Azure Cosmos DB
* En savoir plus sur la façon d’[Indexer des données spatiales avec Azure Cosmos DB](sql-query-geospatial-index.md)
