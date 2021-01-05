---
title: Clause GROUP BY dans Azure Cosmos DB
description: Découvrez la clause GROUP BY pour Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: tisande
ms.openlocfilehash: d9cafc100ddd4b553577c447e82334e6ee7d1b6d
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545416"
---
# <a name="group-by-clause-in-azure-cosmos-db"></a>Clause GROUP BY dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La clause GROUP BY divise les résultats de la requête en fonction des valeurs d’une ou plusieurs propriétés spécifiées.

## <a name="syntax"></a>Syntaxe

```sql  
<group_by_clause> ::= GROUP BY <scalar_expression_list>

<scalar_expression_list> ::=
          <scalar_expression>
        | <scalar_expression_list>, <scalar_expression>
```  

## <a name="arguments"></a>Arguments

- `<scalar_expression_list>`

   Spécifie les expressions qui seront utilisées pour diviser les résultats de la requête.

- `<scalar_expression>`
  
   Toutes les expressions scalaires sont autorisées à l’exception des sous-requêtes scalaires et des agrégats scalaires. Chaque expression scalaire doit contenir au moins une référence de propriété. Il n’existe aucune limite quant au nombre d’expressions individuelles ou à la cardinalité de chaque expression.

## <a name="remarks"></a>Notes
  
  Quand une requête utilise une clause GROUP BY, la clause SELECT ne peut contenir que le sous-ensemble de propriétés et de fonctions système incluses dans la clause GROUP BY. Les [fonctions d’agrégation](sql-query-aggregate-functions.md) font exception. Elles peuvent apparaître dans la clause SELECT sans être incluses dans la clause GROUP BY. Par ailleurs, vous pouvez toujours inclure des valeurs littérales dans la clause SELECT.

  La clause GROUP BY doit être placée après les clauses SELECT, FROM et WHERE et avant la clause OFFSET LIMIT. Vous ne pouvez pas utiliser la clause GROUP BY avec une clause ORDER BY pour le moment, mais cela est prévu.

  La clause GROUP BY n’autorise pas les éléments suivants :
  
- Propriétés d’alias ou fonctions système d’alias (l’alias est toujours autorisé dans la clause SELECT)
- Sous-requêtes
- Fonctions système d’agrégation (uniquement autorisées dans la clause SELECT)

Les requêtes comprenant une fonction système d’agrégation et une sous-requête avec `GROUP BY` ne sont pas prises en charge. Par exemple, la requête suivante n’est pas prise en charge :

```sql
SELECT COUNT(UniqueLastNames)
FROM (
SELECT AVG(f.age)
FROM f
GROUP BY f.lastName
) AS UniqueLastNames
```

## <a name="examples"></a>Exemples

Ces exemples utilisent le jeu de données sur l’alimentation, disponible sur le site du [Playground de test de requêtes Azure Cosmos DB](https://www.documentdb.com/sql/demo).

Par exemple, voici une requête qui retourne le nombre total d’éléments dans chaque foodGroup :

```sql
SELECT TOP 4 COUNT(1) AS foodGroupCount, f.foodGroup
FROM Food f
GROUP BY f.foodGroup
```

Certains résultats sont présentés ci-après (le mot clé TOP est utilisé pour limiter les résultats) :

```json
[
    {
        "foodGroupCount": 183,
        "foodGroup": "Cereal Grains and Pasta"
    },
    {
        "foodGroupCount": 133,
        "foodGroup": "Nut and Seed Products"
    },
    {
        "foodGroupCount": 113,
        "foodGroup": "Meals, Entrees, and Side Dishes"
    },
    {
        "foodGroupCount": 64,
        "foodGroup": "Spices and Herbs"
    }
]
```

Cette requête comporte deux expressions utilisées pour diviser les résultats :

```sql
SELECT TOP 4 COUNT(1) AS foodGroupCount, f.foodGroup, f.version
FROM Food f
GROUP BY f.foodGroup, f.version
```

Certains résultats sont présentés ci-après :

```json
[
    {
        "foodGroupCount": 183,
        "foodGroup": "Cereal Grains and Pasta",
        "version": 1
    },
    {
        "foodGroupCount": 133,
        "foodGroup": "Nut and Seed Products",
        "version": 1
    },
    {
        "foodGroupCount": 113,
        "foodGroup": "Meals, Entrees, and Side Dishes",
        "version": 1
    },
    {
        "foodGroupCount": 64,
        "foodGroup": "Spices and Herbs",
        "version": 1
    }
]
```

Cette requête comporte une fonction système dans la clause GROUP BY :

```sql
SELECT TOP 4 COUNT(1) AS foodGroupCount, UPPER(f.foodGroup) AS upperFoodGroup
FROM Food f
GROUP BY UPPER(f.foodGroup)
```

Certains résultats sont présentés ci-après :

```json
[
    {
        "foodGroupCount": 183,
        "upperFoodGroup": "CEREAL GRAINS AND PASTA"
    },
    {
        "foodGroupCount": 133,
        "upperFoodGroup": "NUT AND SEED PRODUCTS"
    },
    {
        "foodGroupCount": 113,
        "upperFoodGroup": "MEALS, ENTREES, AND SIDE DISHES"
    },
    {
        "foodGroupCount": 64,
        "upperFoodGroup": "SPICES AND HERBS"
    }
]
```

Cette requête utilise des mots clés et des fonctions système dans l’expression de propriété de l’élément :

```sql
SELECT COUNT(1) AS foodGroupCount, ARRAY_CONTAINS(f.tags, {name: 'orange'}) AS containsOrangeTag,  f.version BETWEEN 0 AND 2 AS correctVersion
FROM Food f
GROUP BY ARRAY_CONTAINS(f.tags, {name: 'orange'}), f.version BETWEEN 0 AND 2
```

Les résultats sont :

```json
[
    {
        "foodGroupCount": 10,
        "containsOrangeTag": true,
        "correctVersion": true
    },
    {
        "foodGroupCount": 8608,
        "containsOrangeTag": false,
        "correctVersion": true
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Clause SELECT](sql-query-select.md)
- [Fonctions d’agrégation](sql-query-aggregate-functions.md)
