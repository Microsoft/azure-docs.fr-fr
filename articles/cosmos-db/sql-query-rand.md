---
title: 'Langage de requête Azure Cosmos DB : RAND'
description: Découvrez la fonction système SQL RAND dans Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/16/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: fb3e310970fcc2146ee0d4b790a9744dcd566bad
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341652"
---
# <a name="rand-azure-cosmos-db"></a>RAND (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Retourne une valeur numérique générée de façon aléatoire à partir de [0, 1).
 
## <a name="syntax"></a>Syntaxe
  
```sql
RAND ()  
```  

## <a name="return-types"></a>Types de retour

  Renvoie une expression numérique.

## <a name="remarks"></a>Notes

  `RAND` est une fonction non déterministe. Les appels répétitifs de `RAND` ne retournent pas les mêmes résultats. Cette fonction système n’utilisera pas l’index.


## <a name="examples"></a>Exemples
  
  L’exemple suivant retourne une valeur numérique générée de façon aléatoire.
  
```sql
SELECT RAND() AS rand 
```  
  
 Voici le jeu de résultats obtenu.  
  
```json
[{"rand": 0.87860053195618093}]  
``` 

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions mathématiques Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
