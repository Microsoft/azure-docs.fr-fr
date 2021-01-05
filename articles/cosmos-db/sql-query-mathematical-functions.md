---
title: 'Langage de requête Azure Cosmos DB : fonctions mathématiques'
description: Découvrez les fonctions mathématiques dans Azure Cosmos DB, qui effectuent un calcul basé sur les valeurs d’entrée fournies comme arguments et renvoient une valeur numérique.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 63d349c8cfff52932d51ce7143aba33521c43890
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549185"
---
# <a name="mathematical-functions-azure-cosmos-db"></a>Fonctions mathématiques (Azure Cosmos DB)  
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Chaque fonction mathématique effectue un calcul, basé sur les valeurs d’entrée fournies comme arguments, et renvoie une valeur numérique.

Vous pouvez exécuter des requêtes similaires à l’exemple suivant :

```sql
    SELECT VALUE ABS(-4)
```

Le résultat est le suivant :

```json
    [4]
```

## <a name="functions"></a>Fonctions

Les fonctions suivantes sont des fonctions mathématiques intégrées prises en charge, qui effectuent un calcul, généralement basé sur des arguments d’entrée, et retournent une expression numérique :
 
* [ABS](sql-query-abs.md)
* [ACOS](sql-query-acos.md)
* [ASIN](sql-query-asin.md)
* [ATAN](sql-query-atan.md)
* [ATN2](sql-query-atn2.md)
* [CEILING](sql-query-ceiling.md)
* [COS](sql-query-cos.md)
* [COT](sql-query-cot.md)
* [DEGREES](sql-query-degrees.md)
* [EXP](sql-query-exp.md)
* [FLOOR](sql-query-floor.md)
* [LOG](sql-query-log.md)
* [LOG10](sql-query-log10.md)
* [PI](sql-query-pi.md)
* [POWER](sql-query-power.md)
* [RADIANS](sql-query-radians.md)
* [RAND](sql-query-rand.md)
* [ROUND](sql-query-round.md)
* [SIGN](sql-query-sign.md)
* [SIN](sql-query-sin.md)
* [SQRT](sql-query-sqrt.md)
* [SQUARE](sql-query-square.md)
* [TAN](sql-query-tan.md)
* [TRUNC](sql-query-trunc.md)

  
Toutes les fonctions mathématiques, à l’exception de RAND, sont des fonctions déterministes. Cela signifie qu’elles retournent les mêmes résultats chaque fois qu’elles sont appelées avec un ensemble spécifique de valeurs d’entrée.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
- [Fonctions définies par l’utilisateur](sql-query-udfs.md)
- [Agrégats](sql-query-aggregate-functions.md)
