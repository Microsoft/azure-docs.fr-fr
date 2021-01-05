---
title: DateTimeDiff dans le langage de requête Azure Cosmos DB
description: Découvrez la fonction système SQL DateTimeDiff dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 09b801c48bf2998a2d8926009cae76287c1ac9b6
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342264"
---
# <a name="datetimediff-azure-cosmos-db"></a>DateTimeDiff (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
Retourne le nombre (valeur entière signée) des limites DateTimePart spécifiées, franchies entre les valeurs *StartDate* et *EndDate* spécifiées.
  
## <a name="syntax"></a>Syntaxe
  
```sql
DateTimeDiff (<DateTimePart> , <StartDate> , <EndDate>)
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

*StartDate*  
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

*EndDate*  
   Valeur de la chaîne ISO 8601 (date et heure UTC) au format `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Types de retour

Retourne une valeur d'entier signé.

## <a name="remarks"></a>Notes

DateTimeDiff renvoie `undefined` pour les raisons suivantes :

- La valeur DateTimePart spécifiée n'est pas valide
- StartDate ou EndDate n’est pas une valeur DateTime ISO 8601 valide

DateTimeDiff retourne toujours une valeur entière signée et est une mesure du nombre de limites DateTimePart franchies, et non de l’intervalle de temps.

## <a name="examples"></a>Exemples
  
L’exemple suivant calcule le nombre de limites de jours franchies entre `2020-01-01T01:02:03.1234527Z` et `2020-01-03T01:02:03.1234567Z`.

```sql
SELECT DateTimeDiff("day", "2020-01-01T01:02:03.1234527Z", "2020-01-03T01:02:03.1234567Z") AS DifferenceInDays
```

```json
[
    {
        "DifferenceInDays": 2
    }
]
```  

L’exemple suivant calcule le nombre de limites d’années franchies entre `2028-01-01T01:02:03.1234527Z` et `2020-01-03T01:02:03.1234567Z`.

```sql
SELECT DateTimeDiff("yyyy", "2028-01-01T01:02:03.1234527Z", "2020-01-03T01:02:03.1234567Z") AS DifferenceInYears
```

```json
[
    {
        "DifferenceInYears": -8
    }
]
```

L’exemple suivant calcule le nombre de limites d’heures franchies entre `2020-01-01T01:00:00.1234527Z` et `2020-01-01T01:59:59.1234567Z`. Même si ces valeurs DateTime sont supérieures à 0,99 heures, `DateTimeDiff` retourne 0, car aucune limite d’heure n’a été franchie.

```sql
SELECT DateTimeDiff("hh", "2020-01-01T01:00:00.1234527Z", "2020-01-01T01:59:59.1234567Z") AS DifferenceInHours
```

```json
[
    {
        "DifferenceInHours": 0
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de date et heure Azure Cosmos DB](sql-query-date-time-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
