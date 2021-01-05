---
title: Clause SELECT dans Azure Cosmos DB
description: Découvrez la clause SQL SELECT pour Azure Cosmos DB. Utilisez SQL comme langage de requête JSON Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: tisande
ms.openlocfilehash: 072e17b1c0ea312b4adfa1687e447fd2cadde233
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93335436"
---
# <a name="select-clause-in-azure-cosmos-db"></a>Clause SELECT dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Chaque requête se compose d’une clause `SELECT` et de clauses [FROM](sql-query-from.md) et [WHERE](sql-query-where.md) facultatives, conformes aux normes ANSI-SQL. En règle générale, la source dans la clause `FROM` est énumérée, tandis que la clause `WHERE` applique un filtre sur la source pour récupérer un sous-ensemble d’éléments JSON. Ensuite, la clause `SELECT` projette les valeurs JSON demandées dans la liste sélectionnée.

## <a name="syntax"></a>Syntaxe

```sql
SELECT <select_specification>  

<select_specification> ::=
      '*'
      | [DISTINCT] <object_property_list>
      | [DISTINCT] VALUE <scalar_expression> [[ AS ] value_alias]  
  
<object_property_list> ::=
{ <scalar_expression> [ [ AS ] property_alias ] } [ ,...n ]  
```  
  
## <a name="arguments"></a>Arguments
  
- `<select_specification>`  

  Propriétés ou valeur à sélectionner pour le jeu de résultats.  
  
- `'*'`  

  Spécifie que la valeur doit être récupérée sans apporter de modifications. Plus spécifiquement, si la valeur traitée est un objet, toutes les propriétés sont récupérées.  
  
- `<object_property_list>`  
  
  Spécifie la liste des propriétés à récupérer. Chaque valeur retournée sera un objet avec les propriétés spécifiées.  
  
- `VALUE`  

  Spécifie que la valeur JSON doit être récupérée au lieu de l’objet JSON complet. Cela, contrairement à `<property_list>`, n’encapsule pas la valeur projetée dans un objet.  

- `DISTINCT`
  
  Spécifie que les doublons de propriétés projetées doivent être supprimés.  

- `<scalar_expression>`  

  Expression représentant la valeur à calculer. Consultez la section [Expressions scalaires](sql-query-scalar-expressions.md) pour plus d’informations.  

## <a name="remarks"></a>Notes

La syntaxe `SELECT *` est valide uniquement si la clause FROM a déclaré exactement un alias. `SELECT *` fournit une projection d’identité, ce qui peut être utile si aucune projection n’est nécessaire. SELECT * est valide uniquement si la clause FROM est spécifiée et n’a introduit qu’une seule source d’entrée.  
  
`SELECT <select_list>` et `SELECT *` sont ce qu’on appelle du « sucre syntaxique » et que vous pouvez également les exprimer à l’aide d’instructions SELECT simples, comme indiqué ci-dessous.  
  
1. `SELECT * FROM ... AS from_alias ...`  
  
   équivaut à :  
  
   `SELECT from_alias FROM ... AS from_alias ...`  
  
2. `SELECT <expr1> AS p1, <expr2> AS p2,..., <exprN> AS pN [other clauses...]`  
  
   équivaut à :  
  
   `SELECT VALUE { p1: <expr1>, p2: <expr2>, ..., pN: <exprN> }[other clauses...]`  
  
## <a name="examples"></a>Exemples

L’exemple d’instruction SELECT suivante retourne `address` de `Families` dont `id` correspond à `AndersenFamily` :

```sql
    SELECT f.address
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Les résultats sont :

```json
    [{
      "address": {
        "state": "WA",
        "county": "King",
        "city": "Seattle"
      }
    }]
```

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Clause WHERE](sql-query-where.md)
