---
title: Clause WHERE dans Azure Cosmos DB
description: Découvrez la clause SQL WHERE pour Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/06/2020
ms.author: tisande
ms.openlocfilehash: 5620a9fb95fb52a487095afd75d5f30c82a8bce1
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341465"
---
# <a name="where-clause-in-azure-cosmos-db"></a>Clause WHERE dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La clause WHERE facultative (`WHERE <filter_condition>`) spécifie la ou les conditions que les éléments JSON sources doivent remplir pour que la requête les inclue dans les résultats. Un élément JSON doit évaluer les conditions spécifiées comme étant égales à `true` pour être pris en considération pour le résultat. La couche d’index utilise la clause WHERE pour identifier le plus petit sous-ensemble d’éléments sources pouvant appartenir au résultat.
  
## <a name="syntax"></a>Syntaxe
  
```sql  
WHERE <filter_condition>  
<filter_condition> ::= <scalar_expression>  
  
```  
  
## <a name="arguments"></a>Arguments

- `<filter_condition>`  
  
   Spécifie la condition à remplir pour les documents à retourner.  
  
- `<scalar_expression>`  
  
   Expression représentant la valeur à calculer. Consultez la [Expressions scalaires](sql-query-scalar-expressions.md) pour plus d’informations.  
  
## <a name="remarks"></a>Notes
  
  Pour que le document soit retourné, une expression spécifiée en tant que filtre de condition doit correspondre à la valeur true. Seule la valeur booléenne `true` satisfait la condition. Les autres valeurs : undefined, null, false, nombre, tableau ou objet ne satisfont pas la condition.

  Si vous incluez votre clé de partition dans la clause `WHERE` dans le cadre d’un filtre d’égalité, votre requête sera automatiquement filtrée sur les seules partitions pertinentes.

## <a name="examples"></a>Exemples

La requête suivante demande les éléments qui contiennent une propriété `id` dont la valeur est `AndersenFamily`. Elle exclut tout élément qui n’a pas une propriété `id` ou dont la valeur ne correspond pas à `AndersenFamily`.

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

### <a name="scalar-expressions-in-the-where-clause"></a>Expressions scalaires dans la clause WHERE

L'exemple précédent illustrait une simple requête d'égalité. L’API SQL prend également en charge diverses [expressions scalaires](sql-query-scalar-expressions.md). Les plus répandues sont les expressions binaires et unaires. Les références de propriété de l'objet JSON source sont également des expressions valides.

Vous pouvez utiliser les opérateurs binaires pris en charge suivants :  

|**Type d’opérateur**  | **Valeurs** |
|---------|---------|
|Arithmétique | +,-,*,/,% |
|Au niveau du bit    | \|, &, ^, <<, >>, >>> (décalage vers la droite avec remplissage de zéros) |
|Logical    | AND, OR, NOT      |
|Comparaison | =, !=, &lt;, &gt;, &lt;=, &gt;=, <> |
|String     |  \|\| (concaténer) |

Les requêtes suivantes utilisent des opérateurs binaires :

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE c.grade % 2 = 1     -- matching grades == 5, 1

    SELECT *
    FROM Families.children[0] c
    WHERE c.grade ^ 4 = 1    -- matching grades == 5

    SELECT *
    FROM Families.children[0] c
    WHERE c.grade >= 5    -- matching grades == 5
```

Vous pouvez également utiliser les opérateurs unaires +,-, ~ et NOT dans les requêtes, comme le montrent les exemples suivants :

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE NOT(c.grade = 5)  -- matching grades == 1

    SELECT *
    FROM Families.children[0] c
    WHERE (-c.grade = -5)  -- matching grades == 5
```

Vous pouvez également utiliser des références de propriété dans les requêtes. Par exemple, `SELECT * FROM Families f WHERE f.isRegistered` retourne l’élément JSON contenant la propriété `isRegistered` dont la valeur est égale à `true`. Toute autre valeur, telle que `false`, `null`, `Undefined`, `<number>`, `<string>`, `<object>` ou `<array>`, exclut l’élément du résultat. En outre, vous pouvez utiliser la fonction de vérification de type `IS_DEFINED` pour effectuer des requêtes en fonction de la présence ou de l’absence d’une propriété JSON donnée. Par exemple, `SELECT * FROM Families f WHERE NOT IS_DEFINED(f.isRegistered)` retourne un élément JSON qui n’a pas de valeur pour `isRegistered`.

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Mot clé IN](sql-query-keywords.md#in)
- [Clause FROM](sql-query-from.md)
