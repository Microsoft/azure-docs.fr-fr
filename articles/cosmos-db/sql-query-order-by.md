---
title: Clause ORDER BY dans Azure Cosmos DB
description: Découvrez la clause SQL ORDER BY pour Azure Cosmos DB. Utilisez SQL comme langage de requête JSON Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 06/06/2020
ms.author: tisande
ms.openlocfilehash: 887dc13eb5e351688718d2a221e69499557b23e5
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93338293"
---
# <a name="order-by-clause-in-azure-cosmos-db"></a>Clause ORDER BY dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La clause `ORDER BY` facultative spécifie l’ordre de tri des résultats retournés par la requête.

## <a name="syntax"></a>Syntaxe
  
```sql  
ORDER BY <sort_specification>  
<sort_specification> ::= <sort_expression> [, <sort_expression>]  
<sort_expression> ::= {<scalar_expression> [ASC | DESC]} [ ,...n ]  
```  

## <a name="arguments"></a>Arguments
  
- `<sort_specification>`  
  
   Spécifie une propriété ou expression sur laquelle trier le jeu de résultats de requête. Une colonne de tri peut être spécifiée en tant qu’alias de nom ou de propriété.  
  
   Plusieurs propriétés peuvent être spécifiées. Les noms de propriété doivent être uniques. La séquence des propriétés de tri dans la clause `ORDER BY` détermine l’organisation du jeu de résultats trié. Autrement dit, le jeu de résultats est trié par la première propriété, puis cette liste triée est triée par la deuxième propriété, et ainsi de suite.  
  
   Les noms de propriétés référencés dans la clause `ORDER BY` doivent correspondre à une propriété dans la liste de sélection ou à une propriété définie dans la collection spécifiée dans la clause `FROM` sans ambiguïté.  
  
- `<sort_expression>`  
  
   Spécifie une ou plusieurs propriétés ou expressions sur lesquelles trier le jeu de résultats de requête.  
  
- `<scalar_expression>`  
  
   Consultez la section [Expressions scalaires](sql-query-scalar-expressions.md) pour plus d’informations.  
  
- `ASC | DESC`  
  
   Spécifie que les valeurs dans la colonne spécifiée doivent être triées par ordre croissant ou décroissant. `ASC` effectue le tri de la valeur la plus faible à la valeur la plus élevée. `DESC` effectue le tri de la valeur la plus élevée à la valeur la plus faible. `ASC` correspond à l’ordre de tri par défaut. Les valeurs NULL sont traitées comme les plus petites valeurs possibles.  
  
## <a name="remarks"></a>Notes  
  
   La clause `ORDER BY` nécessite que la stratégie d’indexation comprenne un index pour les champs de tri. Le runtime de requête Azure Cosmos DB prend en charge le tri par rapport à un nom de propriété et non par rapport à des propriétés calculées. Azure Cosmos DB prend en charge plusieurs propriétés `ORDER BY`. Pour exécuter une requête avec plusieurs propriétés ORDER BY, vous devez définir un [index composite](index-policy.md#composite-indexes) sur les champs de tri.

> [!Note]
> Si les propriétés en cours de tri peuvent ne pas être définies pour certains documents et que vous voulez les récupérer dans une requête ORDER BY, vous devez inclure explicitement ce chemin d’accès dans l’index. La stratégie d’indexation par défaut n’autorise pas la récupération des documents où la propriété de tri n’est pas définie. [Passez en revue les exemples de requêtes sur les documents contenant des champs manquants](#documents-with-missing-fields).

## <a name="examples"></a>Exemples

Par exemple, voici une requête qui récupère les familles dans l’ordre croissant de la ville de résidence :

```sql
    SELECT f.id, f.address.city
    FROM Families f
    ORDER BY f.address.city
```

Les résultats sont :

```json
    [
      {
        "id": "WakefieldFamily",
        "city": "NY"
      },
      {
        "id": "AndersenFamily",
        "city": "Seattle"
      }
    ]
```

La requête suivante récupère les `id` de famille dans l’ordre de la date de création de leur élément. L’élément `creationDate` est un nombre représentant l’ *heure d’époque* , ou le temps écoulé depuis le 1er janvier 1970 en secondes.

```sql
    SELECT f.id, f.creationDate
    FROM Families f
    ORDER BY f.creationDate DESC
```

Les résultats sont :

```json
    [
      {
        "id": "WakefieldFamily",
        "creationDate": 1431620462
      },
      {
        "id": "AndersenFamily",
        "creationDate": 1431620472
      }
    ]
```

De plus, vous pouvez trier par plusieurs propriétés. Une requête qui trie par plusieurs propriétés requiert un [index composite](index-policy.md#composite-indexes). Considérez la requête suivante :

```sql
    SELECT f.id, f.creationDate
    FROM Families f
    ORDER BY f.address.city ASC, f.creationDate DESC
```

Cette requête récupère les `id` de famille dans l’ordre croissant du nom de la ville. Si plusieurs éléments ont le même nom de ville, la requête trie par la `creationDate` dans l’ordre décroissant.

## <a name="documents-with-missing-fields"></a>Documents avec champs manquants

Les requêtes `ORDER BY` qui sont exécutées sur des conteneurs avec la stratégie d’indexation par défaut ne retournent pas de documents dont la propriété de tri n’est pas définie. Si vous souhaitez inclure des documents dont la propriété de tri n’est pas définie, vous devez inclure explicitement cette propriété dans la stratégie d’indexation.

Par exemple, voici un conteneur avec une stratégie d’indexation qui n’inclut pas explicitement de chemins d’accès autres que `"/*"` :

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": []
}
```

Si vous exécutez une requête qui inclut `lastName` dans la clause `Order By`, les résultats incluront uniquement les documents dont la propriété `lastName` est définie. Nous n’avons pas défini de chemin d’accès inclus explicite pour `lastName`, de sorte que tous les documents sans `lastName` n’apparaissent pas dans les résultats de la requête.

Voici une requête qui trie par `lastName` sur deux documents, dont l’un n’a pas de propriété `lastName` définie :

```sql
    SELECT f.id, f.lastName
    FROM Families f
    ORDER BY f.lastName
```

Les résultats incluent uniquement le document dont `lastName` est définie :

```json
    [
        {
            "id": "AndersenFamily",
            "lastName": "Andersen"
        }
    ]
```

Si nous mettons à jour la stratégie d’indexation du conteneur pour inclure explicitement un chemin d’accès pour `lastName`, nous inclurons des documents avec une propriété de tri non définie dans les résultats de la requête. Vous devez définir explicitement le chemin d’accès menant à cette valeur scalaire (et non au-delà de celle-ci). Vous devez utiliser le caractère `?` dans la définition de votre chemin d’accès dans la stratégie d’indexation pour vous assurer que vous indexez explicitement la propriété `lastName` et qu’il n’y a pas d’autres chemins d’accès imbriqués au-delà. Si votre requête `Order By` utilise un [index composite](index-policy.md#composite-indexes), les résultats incluront toujours les documents avec une propriété de tri non définie dans les résultats de la requête.

Voici un exemple de stratégie d’indexation qui vous permet d’afficher des documents avec une propriété `lastName` non définie dans les résultats de la requête :

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/lastName/?"
        },
        {
            "path": "/*"
        }
    ],
    "excludedPaths": []
}
```

Si vous réexécutez la même requête, les documents qui n’ont pas de `lastName` s’affichent en premier dans les résultats de la requête :

```sql
    SELECT f.id, f.lastName
    FROM Families f
    ORDER BY f.lastName
```

Les résultats sont :

```json
[
    {
        "id": "WakefieldFamily"
    },
    {
        "id": "AndersenFamily",
        "lastName": "Andersen"
    }
]
```

Si vous modifiez l’ordre de tri en `DESC`, les documents sans `lastName` apparaissent en dernier dans les résultats de la requête :

```sql
    SELECT f.id, f.lastName
    FROM Families f
    ORDER BY f.lastName DESC
```

Les résultats sont :

```json
[
    {
        "id": "AndersenFamily",
        "lastName": "Andersen"
    },
    {
        "id": "WakefieldFamily"
    }
]
```

> [!Note]
> Seul le kit SDK .NET version 3.4.0 ou ultérieure prend en charge ORDER BY avec des types mixtes. Par conséquent, si vous souhaitez faire un tri par combinaison de valeurs non définies et définies, vous devez utiliser cette version (ou une version ultérieure).

Vous ne pouvez pas contrôler l’ordre dans lequel les différents types apparaissent dans les résultats. Dans l’exemple ci-dessus, nous avons montré comment les valeurs non définies ont été triées avant les valeurs de chaîne. Si, par exemple, vous souhaitez plus de contrôle sur l’ordre de tri des valeurs non définies, vous pouvez assigner à toutes les propriétés non définies une valeur de chaîne « aaaaaaaaa » ou « zzzzzzzz » pour vous assurer qu’elles étaient soit la première, soit la dernière.

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Stratégie d’indexation dans Azure Cosmos DB](index-policy.md)
- [Clause OFFSET LIMIT](sql-query-offset-limit.md)
