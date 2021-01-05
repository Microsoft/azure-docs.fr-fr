---
title: 'Langage de requête Azure Cosmos DB : EndsWith'
description: Découvrir la fonction système SQL ENDSWITH dans Azure Cosmos DB, qui retourne une valeur booléenne indiquant si la première expression de chaîne se termine par la seconde
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 06/02/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: c0cc93fee8aacc711a797925cb2e2808b73cafd1
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93338830"
---
# <a name="endswith-azure-cosmos-db"></a>ENDSWITH (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Retourne une valeur booléenne indiquant si la première expression de chaîne se termine par la seconde.  
  
## <a name="syntax"></a>Syntaxe
  
```sql
ENDSWITH(<str_expr1>, <str_expr2> [, <bool_expr>])
```  
  
## <a name="arguments"></a>Arguments
  
*str_expr1*  
   Est une expression de chaîne.  
  
*str_expr2*  
   Est une expression de chaîne à comparer avec la fin de *str_expr1*.

*bool_expr* est une valeur facultative permettant d’ignorer la casse. Quand la valeur est « true », ENDSWITH effectue une recherche qui ne respecte pas la casse. Quand elle n’est pas spécifiée, cette valeur est « false ».
  
## <a name="return-types"></a>Types de retour
  
  Retourne une expression booléenne.  
  
## <a name="examples"></a>Exemples
  
L’exemple suivant vérifie si la chaîne « abc » finit par « b » et « bC ».  
  
```sql
SELECT ENDSWITH("abc", "b", false) AS e1, ENDSWITH("abc", "bC", false) AS e2, ENDSWITH("abc", "bC", true) AS e3
```  
  
 Voici le jeu de résultats obtenu.  
  
```json
[
    {
        "e1": false,
        "e2": false,
        "e3": true
    }
]
```  

## <a name="remarks"></a>Notes

Cette fonction système bénéficiera d’un [index de plage](index-policy.md#includeexclude-strategy).

La consommation de RU par EndsWith augmente en même temps que la cardinalité de la propriété dans la fonction système. En d’autres termes, si vous vérifiez si une valeur de propriété se termine par une chaîne donnée, la charge en RU de la requête dépend du nombre de valeurs possibles pour cette propriété.

Par exemple, considérez deux propriétés : ville et pays. La cardinalité de la propriété ville est 5 000 et celle de la propriété pays est 200. Voici deux exemples de requêtes :

```sql
    SELECT * FROM c WHERE ENDSWITH(c.town, "York", false)
```

```sql
    SELECT * FROM c WHERE ENDSWITH(c.country, "States", false)
```

La première requête utilisera probablement plus de RU que la deuxième, car la cardinalité de la propriété ville est supérieure à celle de la propriété pays.

Si la taille de la propriété dans EndsWith est supérieure à 1 Ko pour certains documents, le moteur de requête doit charger ces documents. Dans ce cas, le moteur de requête ne peut pas évaluer correctement EndsWith avec un index. Le coût des unités de requête pour EndsWith est élevé si vous avez un grand nombre de documents dont les tailles de propriété sont supérieures à 1 Ko.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de chaîne Azure Cosmos DB](sql-query-string-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
