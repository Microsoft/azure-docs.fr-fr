---
title: Fonctions d’agrégation dans Azure Cosmos DB
description: Découvrez la syntaxe de la fonction d’agrégation SQL et les types de fonctions d’agrégation prises en charge par Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.openlocfilehash: c0d953c8d99582f63744d51b505852b5c44bc409
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550571"
---
# <a name="aggregate-functions-in-azure-cosmos-db"></a>Fonctions d’agrégation dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les fonctions d’agrégation effectuent un calcul sur un ensemble de valeurs dans la clause `SELECT` et retournent une valeur unique. Par exemple, la requête suivante retourne le nombre d’éléments présents dans un conteneur :

```sql
    SELECT COUNT(1)
    FROM c
```

## <a name="types-of-aggregate-functions"></a>Types de fonctions d’agrégation

L’API SQL prend en charge les fonctions d’agrégation suivantes. `SUM` et `AVG` opèrent sur des valeurs numériques, tandis que `COUNT`, `MIN` et `MAX` fonctionnent sur des nombres, des chaînes, des booléens et des valeurs Null.

| Fonction | Description |
|-------|-------------|
| [AVG](sql-query-aggregate-avg.md) | Renvoie la moyenne des valeurs de l’expression. |
| [COUNT](sql-query-aggregate-count.md) | Renvoie le nombre d’éléments que contient l’expression. |
| [MAX](sql-query-aggregate-max.md) | Retourne la valeur maximale de l'expression. |
| [MIN](sql-query-aggregate-min.md) | Retourne la valeur minimale de l'expression. |
| [SUM](sql-query-aggregate-sum.md) | Renvoie la somme de toutes les valeurs de l’expression. |


Vous pouvez également ne retourner que la valeur scalaire de l’agrégation à l’aide du mot clé VALUE. Par exemple, la requête suivante renvoie le nombre de valeurs sous forme de nombre unique :

```sql
    SELECT VALUE COUNT(1)
    FROM Families f
```

Les résultats sont :

```json
    [ 2 ]
```

Vous pouvez également combiner des agrégations avec des filtres. Par exemple, la requête suivante retourne le nombre d’éléments dont l’État de l’adresse est `WA`.

```sql
    SELECT VALUE COUNT(1)
    FROM Families f
    WHERE f.address.state = "WA"
```

Les résultats sont :

```json
    [ 1 ]
```

## <a name="remarks"></a>Remarques

Ces fonctions système d’agrégation bénéficieront d’un [index de plage](index-policy.md#includeexclude-strategy). Si vous comptez exécuter une fonction `AVG`, `COUNT`, `MAX`, `MIN` ou `SUM` sur une propriété, [indiquez le chemin approprié dans la stratégie d’indexation](index-policy.md#includeexclude-strategy).

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Cosmos DB](introduction.md)
- [Fonctions système](sql-query-system-functions.md)
- [Fonctions définies par l’utilisateur](sql-query-udfs.md)