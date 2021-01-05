---
title: Mots clés SQL pour Azure Cosmos DB
description: Découvrez les mots clés SQL pour Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: 35232f95bc18432db05775807d95f23ceab66aea
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93333781"
---
# <a name="keywords-in-azure-cosmos-db"></a>Mots clés dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article détaille les mots clés qui peuvent être utilisés dans les requêtes SQL Azure Cosmos DB.

## <a name="between"></a>BETWEEN

Vous pouvez utiliser le mot clé `BETWEEN` pour exprimer des requêtes sur des plages de chaînes ou de valeurs numériques. Par exemple, la requête suivante retourne tous les éléments dont la classe du premier enfant est comprise entre 1 et 5, inclus.

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE c.grade BETWEEN 1 AND 5
```

Vous pouvez également utiliser le mot clé `BETWEEN` dans la clause `SELECT`, comme dans l’exemple suivant.

```sql
    SELECT (c.grade BETWEEN 0 AND 10)
    FROM Families.children[0] c
```

Dans l’API SQL, contrairement à SQL ANSI, vous pouvez exprimer des requêtes de plage sur des propriétés de types mixtes. Par exemple, `grade` peut être un nombre comme `5` dans certains éléments et une chaîne comme `grade4` dans d’autres. Dans ces cas-là, comme dans JavaScript, la comparaison entre deux types différents a pour résultat `Undefined` ; l’élément est donc ignoré.

## <a name="distinct"></a>DISTINCT

Le mot clé `DISTINCT` élimine les doublons dans la projection de la requête.

Dans cet exemple, la requête projette des valeurs pour chaque nom de famille :

```sql
SELECT DISTINCT VALUE f.lastName
FROM Families f
```

Les résultats sont :

```json
[
    "Andersen"
]
```

Vous pouvez également projeter des objets uniques. Dans ce cas, le champ lastName n’existe pas dans un des deux documents ; ainsi, la requête retourne un objet vide.

```sql
SELECT DISTINCT f.lastName
FROM Families f
```

Les résultats sont :

```json
[
    {
        "lastName": "Andersen"
    },
    {}
]
```

`DISTINCT` peut également être utilisé dans la projection au sein d’une sous-requête :

```sql
SELECT f.id, ARRAY(SELECT DISTINCT VALUE c.givenName FROM c IN f.children) as ChildNames
FROM f
```

Cette requête projette un tableau qui contient le givenName de chaque enfant, avec suppression des doublons. Ce tableau se voit attribuer l’alias ChildNames et est projeté dans la requête externe.

Les résultats sont :

```json
[
    {
        "id": "AndersenFamily",
        "ChildNames": []
    },
    {
        "id": "WakefieldFamily",
        "ChildNames": [
            "Jesse",
            "Lisa"
        ]
    }
]
```

Les requêtes comprenant une fonction système d’agrégation et une sous-requête avec `DISTINCT` ne sont pas prises en charge. Par exemple, la requête suivante n’est pas prise en charge :

```sql
SELECT COUNT(1) FROM (SELECT DISTINCT f.lastName FROM f)
```

## <a name="in"></a>IN

Utilisez le mot clé IN pour vérifier si une valeur spécifiée correspond à une valeur dans une liste. Par exemple, la requête suivante retourne tous les éléments de famille dont le champ `id` a pour valeur `WakefieldFamily` ou `AndersenFamily`.

```sql
    SELECT *
    FROM Families
    WHERE Families.id IN ('AndersenFamily', 'WakefieldFamily')
```

L’exemple suivant retourne tous les éléments dont l’État (« state ») est l’une des valeurs spécifiées :

```sql
    SELECT *
    FROM Families
    WHERE Families.address.state IN ("NY", "WA", "CA", "PA", "OH", "OR", "MI", "WI", "MN", "FL")
```

L’API SQL prend en charge l’[itération sur les tableaux JSON](sql-query-object-array.md#Iteration), avec une nouvelle construction ajoutée par le biais du mot clé IN dans la source FROM.

Si vous incluez votre clé de partition dans le filtre `IN`, votre requête sera automatiquement filtrée sur les seules partitions pertinentes.

## <a name="top"></a>Haut de la page

Le mot clé TOP retourne les `N` premiers résultats de la requête dans un ordre non défini. En guise de meilleure pratique, utilisez TOP avec la clause `ORDER BY` pour limiter les résultats aux `N` premières valeurs ordonnées. La combinaison de ces deux clauses est le seul moyen d’indiquer de manière prévisible les lignes qui sont affectées par TOP.

Vous pouvez utiliser TOP avec une valeur constante, comme dans l’exemple suivant, ou avec une valeur variable à l’aide de requêtes paramétrables.

```sql
    SELECT TOP 1 *
    FROM Families f
```

Les résultats sont :

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "Seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Joins](sql-query-join.md)
- [Subqueries](sql-query-subquery.md)