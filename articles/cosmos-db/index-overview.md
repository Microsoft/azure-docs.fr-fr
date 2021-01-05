---
title: Indexation dans Azure Cosmos DB
description: Comprendre le fonctionnement de l’indexation dans Azure Cosmos DB et différents types d’index, tels que Range, Spatial et les index composites pris en charge.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/21/2020
ms.author: tisande
ms.openlocfilehash: 4211f13324b9fda0b0823b2d035eb03863cb686d
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339751"
---
# <a name="indexing-in-azure-cosmos-db---overview"></a>Vue d’ensemble de l’indexation dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB est une base de données indépendante des schémas qui vous permet d’itérer sur votre application sans avoir à vous soucier de la gestion des schémas ou des index. Par défaut, Azure Cosmos DB indexe automatiquement toutes les propriétés de tous les éléments de votre [conteneur](account-databases-containers-items.md#azure-cosmos-containers) sans avoir à définir de schéma ou configurer d’index secondaires.

L’objectif de cet article est d’expliquer comment Azure Cosmos DB indexe les données et comment il utilise les index pour améliorer les performances des requêtes. Il est recommandé de parcourir cette section avant de découvrir comment personnaliser les [stratégies d’indexation](index-policy.md).

## <a name="from-items-to-trees"></a>Des éléments aux arborescences

Chaque fois qu’un élément est stocké dans un conteneur, son contenu est projeté sous forme de document JSON, puis converti en une représentation arborescente. Cela signifie que chaque propriété de cet élément est représentée en tant que nœud dans une arborescence. Un pseudo nœud est créé en tant que parent pour toutes les propriétés de premier niveau de l’élément. Les nœuds terminaux contiennent des valeurs scalaires réelles effectuées par un élément.

Par exemple, tenez compte de cet élément :

```json
    {
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 },
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }
```

Il serait représenté par l’arborescence suivante :

:::image type="content" source="./media/index-overview/item-as-tree.png" alt-text="L’élément précédent représenté sous forme d’arborescence" border="false":::

Notez comment les tableaux sont encodés dans l’arborescence : chaque entrée d’un tableau reçoit un nœud intermédiaire étiqueté avec l’index de cette entrée dans le tableau (0, 1, et ainsi de suite).

## <a name="from-trees-to-property-paths"></a>Des arborescences aux chemins de propriété

Azure Cosmos DB transforme les éléments en arborescence car il permet de référencer les propriétés par leurs chemins d’accès dans ces arborescences. Pour obtenir le chemin d’accès d’une propriété, nous pouvons parcourir l’arborescence entre le nœud racine et cette propriété, et concaténer les étiquettes de chaque nœud traversé.

Voici les chemins d’accès de chaque propriété de l’élément exemple décrit ci-dessus :

- /locations/0/country: "Germany"
- /locations/0/city: "Berlin"
- /locations/1/country: "France"
- /locations/1/city: "Paris"
- /headquarters/country: "Belgium"
- /headquarters/employees: 250
- /exports/0/city: "Moscow"
- /exports/1/city: "Athens"

Lorsqu’un élément est écrit, Azure Cosmos DB indexe efficacement le chemin d’accès de chaque propriété et sa valeur correspondante.

## <a name="index-kinds"></a>Types d’index

Azure Cosmos DB prend actuellement en charge trois types d’index.

### <a name="range-index"></a>Index de plage

L’index de **plage** est basé sur une structure de type arborescence ordonnée. Le type d’index de plage est utilisé pour les types de requête suivants :

- Requêtes d’égalité :

    ```sql
   SELECT * FROM container c WHERE c.property = 'value'
   ```

   ```sql
   SELECT * FROM c WHERE c.property IN ("value1", "value2", "value3")
   ```

   Correspondance d’égalité sur un élément de tableau
   ```sql
    SELECT * FROM c WHERE ARRAY_CONTAINS(c.tags, "tag1")
    ```

- Requêtes de plage :

   ```sql
   SELECT * FROM container c WHERE c.property > 'value'
   ```
  (fonctionne pour `>`, `<`, `>=`, `<=`, `!=`)

- Vérification de la présence d’une propriété :

   ```sql
   SELECT * FROM c WHERE IS_DEFINED(c.property)
   ```

- Fonctions système String :

   ```sql
   SELECT * FROM c WHERE CONTAINS(c.property, "value")
   ```

   ```sql
   SELECT * FROM c WHERE STRINGEQUALS(c.property, "value")
   ```

- Requêtes `ORDER BY` :

   ```sql
   SELECT * FROM container c ORDER BY c.property
   ```

- Requêtes `JOIN` :

   ```sql
   SELECT child FROM container c JOIN child IN c.properties WHERE child = 'value'
   ```

Les index plage sont utilisables sur des valeurs scalaires (chaîne ou nombre).

### <a name="spatial-index"></a>Index spatial

Les index **spatiaux** permettent d’exécuter des requêtes efficaces sur des objets géospatiaux comme des points, des lignes, des polygones et des multipolygones. Ces requêtes utilisent des mots clés ST_DISTANCE, ST_WITHIN, ST_INTERSECTS. Voici quelques exemples qui utilisent le type d’index spatial :

- Requêtes de distance géospatiale :

   ```sql
   SELECT * FROM container c WHERE ST_DISTANCE(c.property, { "type": "Point", "coordinates": [0.0, 10.0] }) < 40
   ```

- Géospatial dans les requêtes :

   ```sql
   SELECT * FROM container c WHERE ST_WITHIN(c.property, {"type": "Point", "coordinates": [0.0, 10.0] })
   ```

- Requêtes d’intersection géospatiale :

   ```sql
   SELECT * FROM c WHERE ST_INTERSECTS(c.property, { 'type':'Polygon', 'coordinates': [[ [31.8, -5], [32, -5], [31.8, -5] ]]  })  
   ```

Les index spatiaux sont utilisables sur des objets [GeoJSON](./sql-query-geospatial-intro.md) correctement formatés . Les points, les LineStrings, les polygones et les multipolygones sont actuellement pris en charge.

### <a name="composite-indexes"></a>Index composites

Les index **composites** augmentent l’efficacité lorsque vous effectuez des opérations sur plusieurs champs. Le type d’index composite est utilisé pour les types de requête suivants :

- Requêtes `ORDER BY` sur plusieurs propriétés :

```sql
 SELECT * FROM container c ORDER BY c.property1, c.property2
```

- Requêtes avec un filtre et `ORDER BY`. Ces requêtes peuvent utiliser un index composite si la propriété de filtre est ajoutée à la clause `ORDER BY`.

```sql
 SELECT * FROM container c WHERE c.property1 = 'value' ORDER BY c.property1, c.property2
```

- Les requêtes avec un filtre sur deux ou plusieurs propriétés où au moins une propriété est un filtre d'égalité

```sql
 SELECT * FROM container c WHERE c.property1 = 'value' AND c.property2 > 'value'
```

Tant qu’un prédicat de filtre utilise un type d’index, le moteur de requête évalue celui-ci en premier avant d’analyser le reste. Par exemple, si vous avez une requête SQL comme `SELECT * FROM c WHERE c.firstName = "Andrew" and CONTAINS(c.lastName, "Liu")`

* La requête ci-dessus filtre d’abord les entrées où firstName = « Andrew » à l’aide de l’index. Elle transmet ensuite toutes les entrées firstName = « Andrew » via un pipeline pour évaluer le prédicat de filtre CONTAINS.

* Vous pouvez accélérer les requêtes et éviter les analyses de conteneur complètes lorsque vous utilisez des fonctions qui n’utilisent pas l’index (par exemple, CONTAINS) en ajoutant des prédicats de filtre supplémentaires qui utilisent l’index. L’ordre des clauses de filtre n’est pas important. Le moteur de requête détermine les prédicats les plus sélectifs et exécute la requête en conséquence.

## <a name="querying-with-indexes"></a>Interrogation avec des index

Les chemins d’accès extraits lors de l’indexation des données facilitent la recherche de l’index lors du traitement d’une requête. En faisant correspondre la clause `WHERE` d’une requête avec la liste des chemins d’accès indexés, il est possible d’identifier très rapidement les éléments qui correspondent au prédicat de la requête.

Considérez la requête suivante : `SELECT location FROM location IN company.locations WHERE location.country = 'France'`. Le prédicat de requête (filtrage sur les éléments, où une localisation affiche « France » comme pays/région) correspondrait au chemin mis en évidence en rouge ci-dessous :

:::image type="content" source="./media/index-overview/matching-path.png" alt-text="Mise en correspondance d’un chemin d’accès spécifique au sein d’une arborescence" border="false":::

> [!NOTE]
> Une clause `ORDER BY` qui commande par une seule propriété a *toujours* besoin d’un index plage et échouera si le chemin d’accès qu’elle référence n’en a pas. De même, une requête `ORDER BY` qui commande selon plusieurs propriétés nécessite *toujours* un index composite.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail l’indexation dans les articles suivants :

- [Stratégie d’indexation](index-policy.md)
- [Guide pratique pour gérer la stratégie d’indexation](how-to-manage-indexing-policy.md)