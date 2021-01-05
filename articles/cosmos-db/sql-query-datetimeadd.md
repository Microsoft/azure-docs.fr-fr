---
title: 'Langage de requête Azure Cosmos DB : DateTimeAdd'
description: Découvrez la fonction système SQL DateTimeAdd dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: dc81849a13fdaef748c5fd631ee66ea5e3eb67c4
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93335787"
---
# <a name="datetimeadd-azure-cosmos-db"></a>DateTimeAdd (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Renvoie la valeur de la chaîne DateTime résultant de l'ajout d'une valeur numérique spécifiée (sous forme d'entier signé) à une chaîne DateTime spécifiée  
  
## <a name="syntax"></a>Syntaxe
  
```sql
DateTimeAdd (<DateTimePart> , <numeric_expr> ,<DateTime>)
```

## <a name="arguments"></a>Arguments
  
*DateTimePart*  
   Partie de la date à laquelle DateTimeAdd ajoute un nombre entier. Ce tableau répertorie tous les arguments DateTimePart valides :

| DateTimePart | Abréviations        |
| ------------ | -------------------- |
| Year         | "year", "yyyy", "yy" |
| Month        | "month", "mm", "m"   |
| jour          | "day", "dd", "d"     |
| Heure         | "hour", "hh"         |
| Minute       | "minute", "mi", "n"  |
| Seconde       | "second", "ss", "s"  |
| Milliseconde  | "millisecond", "ms"  |
| Microseconde  | "microsecond", "mcs" |
| Nanoseconde   | "nanosecond", "ns"   |

*numeric_expr*  
   Valeur entière signée qui sera ajoutée à la valeur DateTimePart de la chaîne DateTime spécifiée

*DateTime*  
   Valeur de la chaîne ISO 8601 (date et heure UTC) au format `YYYY-MM-DDThh:mm:ss.fffffffZ` où :
  
  |Format|Description|
  |-|-|
  |YYYY|Année à quatre chiffres|
  |MM|Mois à deux chiffres (01 = janvier, etc.)|
  |DD|Jour du mois à deux chiffres (01 à 31)|
  |T|Indicateur de début des éléments de l’heure|
  |hh|Heure à deux chiffres (00 à 23)|
  |MM|Minutes à deux chiffres (00 à 59)|
  |ss|Secondes à deux chiffres (00 à 59)|
  |.fffffff|Fractions de seconde à 7 chiffres|
  |Z|Indicateur UTC (temps universel coordonné)||
  
  Pour plus d’informations sur le format ISO 8601, consultez [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601).

## <a name="return-types"></a>Types de retour

Renvoie une valeur de chaîne ISO 8601 (date et heure UTC) au format `YYYY-MM-DDThh:mm:ss.fffffffZ` où :
  
  |Format|Description|
  |-|-|
  |YYYY|Année à quatre chiffres|
  |MM|Mois à deux chiffres (01 = janvier, etc.)|
  |DD|Jour du mois à deux chiffres (01 à 31)|
  |T|Indicateur de début des éléments de l’heure|
  |hh|Heure à deux chiffres (00 à 23)|
  |MM|Minutes à deux chiffres (00 à 59)|
  |ss|Secondes à deux chiffres (00 à 59)|
  |.fffffff|Fractions de seconde à 7 chiffres|
  |Z|Indicateur UTC (temps universel coordonné)||

## <a name="remarks"></a>Notes

DateTimeAdd renvoie `undefined` pour les raisons suivantes :

- La valeur DateTimePart spécifiée n'est pas valide
- La valeur numeric_expr spécifiée n'est pas un nombre entier valide
- La valeur DateTime figurant dans l'argument ou le résultat n'est pas une valeur DateTime ISO 8601 valide.

## <a name="examples"></a>Exemples
  
L'exemple suivant ajoute 1 mois à la valeur DateTime : `2020-07-09T23:20:13.4575530Z`

```sql
SELECT DateTimeAdd("mm", 1, "2020-07-09T23:20:13.4575530Z") AS OneMonthLater
```

```json
[
    {
        "OneMonthLater": "2020-08-09T23:20:13.4575530Z"
    }
]
```  

L'exemple suivant soustrait 2 heures de la valeur DateTime : `2020-07-09T23:20:13.4575530Z`

```sql
SELECT DateTimeAdd("hh", -2, "2020-07-09T23:20:13.4575530Z") AS TwoHoursEarlier
```

```json
[
    {
        "TwoHoursEarlier": "2020-07-09T21:20:13.4575530Z"
    }
]
```  

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de date et heure Azure Cosmos DB](sql-query-date-time-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
