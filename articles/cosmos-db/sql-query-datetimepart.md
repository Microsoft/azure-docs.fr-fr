---
title: 'Langage de requête Azure Cosmos DB : DateTimePart'
description: Découvrez la fonction système SQL DateTimePart dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/14/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 97eaff405086190b60279ac7d5cf8bf441c8f840
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336365"
---
# <a name="datetimepart-azure-cosmos-db"></a>DateTimePart (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Retourne la valeur du DateTimePart spécifié entre la DateHeure spécifiée.
  
## <a name="syntax"></a>Syntaxe
  
```sql
DateTimePart (<DateTimePart> , <DateTime>)
```

## <a name="arguments"></a>Arguments
  
*DateTimePart*  
   Partie de la date pour laquelle DateTimePart retourne la valeur. Ce tableau répertorie tous les arguments DateTimePart valides :

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

*DateTime*  
   Valeur de la chaîne ISO 8601 (date et heure UTC) au format `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Types de retour

Retourne une valeur entière positive

## <a name="remarks"></a>Notes

DateTimePart renvoie `undefined` pour les raisons suivantes :

- La valeur DateTimePart spécifiée n'est pas valide
- La valeur DateHeure n’est pas une valeur DateHeure ISO 8601 valide

Cette fonction système n’utilisera pas l’index.

## <a name="examples"></a>Exemples

Voici un exemple qui retourne la valeur entière du mois :

```sql
SELECT DateTimePart("m", "2020-01-02T03:04:05.6789123Z") AS MonthValue
```

```json
[
    {
        "MonthValue": 1
    }
]
```

Voici un exemple qui retourne le nombre de microsecondes :

```sql
SELECT DateTimePart("mcs", "2020-01-02T03:04:05.6789123Z") AS MicrosecondsValue
```

```json
[
    {
        "MicrosecondsValue": 678912
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de date et heure Azure Cosmos DB](sql-query-date-time-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
