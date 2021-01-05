---
title: 'Langage de requête Azure Cosmos DB : GetCurrentTicks'
description: Découvrez la fonction système SQL GetCurrentTicks dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/14/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 610c545bf25822d27e0a641a1b2631f899502420
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340156"
---
# <a name="getcurrentticks-azure-cosmos-db"></a>GetCurrentTicks (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Retourne le nombre de cycles de 100 nanosecondes qui se sont écoulés depuis le jeudi 1er janvier 1970 à 00:00:00.
  
## <a name="syntax"></a>Syntaxe
  
```sql
GetCurrentTicks ()
```

## <a name="return-types"></a>Types de retour

Retourne une valeur numérique signée indiquant le nombre actuel de cycles de 100 nanosecondes qui se sont écoulés depuis l’époque UNIX. En d’autres termes, GetCurrentTicks retourne le nombre de cycles de 100 nanosecondes qui se sont écoulés depuis le jeudi 1er janvier 1970 à 00:00:00.

## <a name="remarks"></a>Notes

GetCurrentTicks() est une fonction non déterministe. Le résultat retourné est au format UTC (temps universel coordonné).

Cette fonction système n’utilisera pas l’index.

## <a name="examples"></a>Exemples

Voici un exemple qui retourne l’heure actuelle, exprimée en nombre de cycles :

```sql
SELECT GetCurrentTicks() AS CurrentTimeInTicks
```

```json
[
    {
        "CurrentTimeInTicks": 15973607943002652
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de date et heure Azure Cosmos DB](sql-query-date-time-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
