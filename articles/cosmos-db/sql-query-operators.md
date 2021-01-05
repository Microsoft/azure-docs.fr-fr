---
title: Opérateurs de requête SQL pour Azure Cosmos DB
description: Découvrez les opérateurs SQL, tels que les opérateurs d’égalité, de comparaison et logiques pris en charge par Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: c1409bd7f098c24efbb4196d78c6dffb6048119b
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93335435"
---
# <a name="operators-in-azure-cosmos-db"></a>Opérateurs dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article décrit les différents opérateurs pris en charge par Azure Cosmos DB.

## <a name="equality-and-comparison-operators"></a>Opérateurs d’égalité et de comparaison

Le tableau suivant répertorie les résultats des comparaisons d'égalité dans l’API SQL entre deux types JSON.

| **Opérateur** | **Undefined** | **Null** | **Booléen** | **Nombre** | **Chaîne** | **Object** | **Tableau** |
|---|---|---|---|---|---|---|---|
| **Undefined** | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini |
| **Null** | Indéfini | **OK** | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini |
| **Booléen** | Indéfini | Indéfini | **OK** | Indéfini | Indéfini | Indéfini | Indéfini |
| **Nombre** | Indéfini | Indéfini | Indéfini | **OK** | Indéfini | Indéfini | Indéfini |
| **Chaîne** | Indéfini | Indéfini | Indéfini | Indéfini | **OK** | Indéfini | Indéfini |
| **Object** | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini | **OK** | Indéfini |
| **Tableau** | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini | Indéfini | **OK** |

Pour les opérateurs de comparaison tels que `>`, `>=`, `!=`, `<` et `<=`, la comparaison entre types ou entre deux objets ou tableaux génère `Undefined`.  

Si le résultat de l’expression scalaire est `Undefined`, l’élément n’est pas inclus dans le résultat, car `Undefined` n’est pas égal à `true`.

Par exemple, la comparaison de la requête suivante entre un nombre et une valeur de chaîne génère `Undefined`. Par conséquent, le filtre n’inclut pas de résultats.

```sql
SELECT *
FROM c
WHERE 7 = 'a'
```

## <a name="logical-and-or-and-not-operators"></a>Opérateurs logiques (AND, OR et NOT)

Les opérateurs logiques interviennent sur des valeurs booléennes. Les tableaux suivants présentent les tables de vérité logiques de ces opérateurs :

**Opérateur OR**

Retourne la valeur `true` lorsque l'une ou l'autre des conditions est `true`.

|  | **True** | **False** | **Undefined** |
| --- | --- | --- | --- |
| **True** |True |True |True |
| **False** |True |False |Indéfini |
| **Undefined** |True |Indéfini |Indéfini |

**Opérateur OR**

Retourne `true` lorsque les deux expressions sont `true`.

|  | **True** | **False** | **Undefined** |
| --- | --- | --- | --- |
| **True** |True |False |Indéfini |
| **False** |False |False |False |
| **Undefined** |Indéfini |False |Indéfini |

**Opérateur NOT**

Inverse la valeur d'une expression booléenne.

|  | **NOT** |
| --- | --- |
| **True** |False |
| **False** |True |
| **Undefined** |Indéfini |

**Priorité des opérateurs**

Les opérateurs logiques `OR`, `AND` et `NOT` ont le niveau de priorité indiqué ci-dessous :

| **Opérateur** | **Priorité** |
| --- | --- |
| **NOT** |1 |
| **AND** |2 |
| **OR** |3 |

## <a name="-operator"></a>Opérateur *

L’opérateur spécial * projette l’élément entier en l’état. Une fois utilisé, il doit être le seul champ projeté. Une requête comme `SELECT * FROM Families f` est valide, contrairement à `SELECT VALUE * FROM Families f` et `SELECT *, f.id FROM Families f`.

## <a name="-and--operators"></a>? et ?? , opérateurs

Vous pouvez utiliser les opérateurs Ternary (?) et Coalesce (??) pour créer des expressions conditionnelles, comme dans des langages de programmation courants tels que C# et JavaScript.

Vous pouvez utiliser l’opérateur ? pour construire de nouvelles propriétés JSON à la volée. Par exemple, la requête suivante classifie les niveaux de classe dans `elementary` ou `other` :

```sql
     SELECT (c.grade < 5)? "elementary": "other" AS gradeLevel
     FROM Families.children[0] c
```

Vous pouvez également imbriquer des appels à l’opérateur ? , comme dans la requête suivante : 

```sql
    SELECT (c.grade < 5)? "elementary": ((c.grade < 9)? "junior": "high") AS gradeLevel
    FROM Families.children[0] c
```

Comme avec d’autres opérateurs de requête, l’opérateur ? exclut des éléments si les propriétés référencées sont manquantes ou que les types comparés sont différents.

Utilisez l’opérateur ?? pour vérifier efficacement la présence d’une propriété dans un élément quand l’interrogation porte sur des données semi-structurées ou de type mixte. Par exemple, la requête suivante retourne `lastName` s’il est présent, ou `surname` si `lastName` n’est pas présent.

```sql
    SELECT f.lastName ?? f.surname AS familyName
    FROM Families f
```

## <a name="next-steps"></a>Étapes suivantes

- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Mots clés](sql-query-keywords.md)
- [Clause SELECT](sql-query-select.md)
