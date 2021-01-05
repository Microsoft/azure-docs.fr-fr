---
title: AVG dans le langage de requête Azure Cosmos DB
description: En savoir plus sur la fonction système Average (AVG) SQL dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 2cfbca798a7b404e4ee12b93396b2a5b08d7d5bb
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550572"
---
# <a name="avg-azure-cosmos-db"></a>AVG (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cette fonction d’agrégation retourne la moyenne des valeurs de l’expression.
  
## <a name="syntax"></a>Syntaxe
  
```sql
AVG(<numeric_expr>)  
```  
  
## <a name="arguments"></a>Arguments
  
*numeric_expr*  
   Est une expression numérique.  
  
## <a name="return-types"></a>Types de retour
  
Renvoie une expression numérique.  
  
## <a name="examples"></a>Exemples
  
L’exemple suivant retourne la valeur moyenne de `propertyA` :
  
```sql
SELECT AVG(c.propertyA)
FROM c
```  

## <a name="remarks"></a>Notes

Cette fonction système bénéficiera d’un [index de plage](index-policy.md#includeexclude-strategy). Si des arguments dans `AVG` sont de type chaîne, booléen ou null, l’ensemble de la fonction système d’agrégation retourne `undefined`. Si un argument a une valeur `undefined`, il n’a pas d’impact sur le calcul de `AVG`.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions mathématiques dans Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Fonctions système dans Azure Cosmos DB](sql-query-system-functions.md)
- [Fonctions d’agrégation dans Azure Cosmos DB](sql-query-aggregate-functions.md)