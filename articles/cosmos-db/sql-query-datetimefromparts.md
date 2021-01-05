---
title: 'Langage de requête Azure Cosmos DB : DateTimeFromParts'
description: Découvrez la fonction système SQL DateTimeFromParts dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: da992715b5cf624592db72cacefd35c9513ac43f
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93335770"
---
# <a name="datetimefromparts-azure-cosmos-db"></a>DateTimeFromParts (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Retourne une valeur de type DateTime de chaîne construite à partir des valeurs d’entrée.
  
## <a name="syntax"></a>Syntaxe
  
```sql
DateTimeFromParts(<numberYear>, <numberMonth>, <numberDay> [, numberHour]  [, numberMinute]  [, numberSecond] [, numberOfFractionsOfSecond])
```

## <a name="arguments"></a>Arguments
  
*numberYear* valeur entière pour l’année au format `YYYY`

*numberMonth*  
   Valeur entière pour le mois au format `MM`

*numberDay*  
   Valeur entière pour le jour au format `DD`

*numberHour* (facultatif) Valeur entière pour l’heure au format `hh`

*numberMinute* (facultatif) Valeur entière pour la minute au format `mm`

*numberSecond* (facultatif) Valeur entière pour la seconde au format `ss`

*numberOfFractionsOfSecond* (facultatif) Valeur entière pour une fraction de seconde au format `.fffffff`

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
  
 Pour plus d’informations sur le format ISO 8601, consultez [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601).

## <a name="remarks"></a>Notes

Si les entiers spécifiés créent une valeur DateTime non valide, DateTimeFromParts retourne `undefined`.

Si un argument facultatif n’est pas spécifié, sa valeur est 0.

## <a name="examples"></a>Exemples

Voici un exemple qui comprend uniquement les arguments requis pour construire une valeur DateTime :

```sql
SELECT DateTimeFromParts(2020, 9, 4) AS DateTime
```

```json
[
    {
        "DateTime": "2020-09-04T00:00:00.0000000Z"
    }
]
```

Voici un autre exemple qui utilise également des arguments facultatifs pour construire une valeur DateTime :

```sql
SELECT DateTimeFromParts(2020, 9, 4, 10, 52) AS DateTime
```

```json
[
    {
        "DateTime": "2020-09-04T10:52:00.0000000Z"
    }
]
```

Voici un autre exemple qui utilise également tous les arguments facultatifs pour construire une valeur DateTime :

```sql
SELECT DateTimeFromParts(2020, 9, 4, 10, 52, 12, 3456789) AS DateTime
```

```json
[
    {
        "DateTime": "2020-09-04T10:52:12.3456789Z"
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de date et heure Azure Cosmos DB](sql-query-date-time-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
