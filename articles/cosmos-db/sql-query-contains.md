---
title: 'Langage de requête Azure Cosmos DB : contains'
description: Découvrez la fonction système SQL CONTAINS dans Azure Cosmos DB, qui retourne une valeur booléenne indiquant si la première expression de chaîne contient la seconde
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 06/02/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 4b6835b22e5cfa4ca703b95d70e20112b8723def
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339170"
---
# <a name="contains-azure-cosmos-db"></a>CONTAINS (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Retourne une valeur booléenne indiquant si la première expression de chaîne contient la seconde.  
  
## <a name="syntax"></a>Syntaxe
  
```sql
CONTAINS(<str_expr1>, <str_expr2> [, <bool_expr>])  
```  
  
## <a name="arguments"></a>Arguments
  
*str_expr1*  
   Est l’expression de chaîne faisant l’objet de la recherche.  
  
*str_expr2*  
   Est l’expression de chaîne à rechercher.  

*bool_expr* est une valeur facultative permettant d’ignorer la casse. Quand la valeur est « true », CONTAINS effectue une recherche qui ne respecte pas la casse. Lorsqu’elle n’est pas spécifiée, la valeur est « false ».
  
## <a name="return-types"></a>Types de retour
  
  Retourne une expression booléenne.  
  
## <a name="examples"></a>Exemples
  
  L’exemple suivant vérifie si « abc » contient « ab » et « A ».  
  
```sql
SELECT CONTAINS("abc", "ab", false) AS c1, CONTAINS("abc", "A", false) AS c2, CONTAINS("abc", "A", true) AS c3
```  
  
 Voici le jeu de résultats obtenu.  
  
```json
[
    {
        "c1": true,
        "c2": false,
        "c3": true
    }
]
```  

## <a name="remarks"></a>Notes

Cette fonction système bénéficiera d’un [index de plage](index-policy.md#includeexclude-strategy).

La consommation de RU par Contains augmente en même temps que la cardinalité de la propriété dans la fonction système. En d’autres termes, si vous vérifiez si une valeur de propriété contient une chaîne donnée, la charge en RU de la requête dépend du nombre de valeurs possibles pour cette propriété.

Par exemple, considérez deux propriétés : ville et pays. La cardinalité de la propriété ville est 5 000 et celle de la propriété pays est 200. Voici deux exemples de requêtes :

```sql
    SELECT * FROM c WHERE CONTAINS(c.town, "Red", false)
```

```sql
    SELECT * FROM c WHERE CONTAINS(c.country, "States", false)
```

La première requête utilisera probablement plus de RU que la deuxième, car la cardinalité de la propriété ville est supérieure à celle de la propriété pays.

Si la taille de propriété dans Contains est supérieure à 1 Ko pour certains documents, le moteur de requête doit charger ces documents. Dans ce cas, le moteur de requête ne peut pas évaluer entièrement Contains avec un index. Le coût des unités de requête pour Contains est élevé si vous avez un grand nombre de documents dont les tailles de propriété sont supérieures à 1 Ko.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de chaîne Azure Cosmos DB](sql-query-string-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
