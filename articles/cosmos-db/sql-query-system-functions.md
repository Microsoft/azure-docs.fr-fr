---
title: 'Langage de requête Azure Cosmos DB : fonctions système'
description: Découvrez les fonctions système SQL intégrées et définies par l’utilisateur dans Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 10/15/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 71dc5b84deda04c02b6d1722b11abf6631e9aa03
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546367"
---
# <a name="system-functions-azure-cosmos-db"></a>Fonctions système (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Cosmos DB fournit de nombreuses fonctions SQL intégrées. Les catégories de fonctions intégrées sont répertoriées ci-dessous.  
  
|Groupe de fonctions|Description|Opérations|  
|--------------|-----------------|-----------------| 
|[Fonctions de tableau](sql-query-array-functions.md)|Les fonctions de tableau effectuent une opération sur une valeur d’entrée de tableau et retournent une valeur numérique, une valeur booléenne ou une valeur de tableau. | [ARRAY_CONCAT](sql-query-array-concat.md), [ARRAY_CONTAINS](sql-query-array-contains.md), [ARRAY_LENGTH](sql-query-array-length.md), [ARRAY_SLICE](sql-query-array-slice.md) |
|[Fonctions de date et heure](sql-query-date-time-functions.md)|Les fonctions de date et heure vous permettent d’obtenir les date et heure UTC actuelles sous deux formes : un horodatage numérique dont la valeur est l’époque Unix, en millisecondes, ou une chaîne conforme au format ISO 8601. | [GetCurrentDateTime](sql-query-getcurrentdatetime.md), [GetCurrentTimestamp](sql-query-getcurrenttimestamp.md) |
|[Fonctions mathématiques](sql-query-mathematical-functions.md)|Chaque fonction mathématique effectue un calcul, généralement basé sur les valeurs d'entrée fournies comme arguments, et retourne une valeur numérique. | [ABS](sql-query-abs.md), [ACOS](sql-query-acos.md), [ASIN](sql-query-asin.md), [ATAN](sql-query-atan.md), [ATN2](sql-query-atn2.md), [CEILING](sql-query-ceiling.md), [COS](sql-query-cos.md), [COT](sql-query-cot.md), [DEGREES](sql-query-degrees.md), [EXP](sql-query-exp.md), [FLOOR](sql-query-floor.md), [LOG](sql-query-log.md), [LOG10](sql-query-log10.md), [PI](sql-query-pi.md), [POWER](sql-query-power.md), [RADIANS](sql-query-radians.md), [RAND](sql-query-rand.md), [ROUND](sql-query-round.md), [SIGN](sql-query-sign.md), [SIN](sql-query-sin.md), [SQRT](sql-query-sqrt.md), [SQUARE](sql-query-square.md), [TAN](sql-query-tan.md), [TRUNC](sql-query-trunc.md) |
|[Fonctions spatiales](sql-query-spatial-functions.md)|Les fonctions spatiales effectuent une opération sur une valeur d’entrée de chaîne et retournent une valeur d’entrée d’objet spatial, une valeur numérique ou une valeur booléenne. | [ST_DISTANCE](sql-query-st-distance.md), [ST_INTERSECTS](sql-query-st-intersects.md), [ST_ISVALID](sql-query-st-isvalid.md), [ST_ISVALIDDETAILED](sql-query-st-isvaliddetailed.md), [ST_WITHIN](sql-query-st-within.md) |
|[Fonctions de chaîne](sql-query-string-functions.md)|Les fonctions de chaîne effectuent une opération sur une valeur d’entrée de chaîne et retournent une valeur de type chaîne, une valeur numérique ou une valeur booléenne. | [CONCAT](sql-query-concat.md), [CONTAINS](sql-query-contains.md), [ENDSWITH](sql-query-endswith.md), [INDEX_OF](sql-query-index-of.md), [LEFT](sql-query-left.md), [LENGTH](sql-query-length.md), [LOWER](sql-query-lower.md), [LTRIM](sql-query-ltrim.md), [REGEXMATCH](sql-query-regexmatch.md)[REPLACE](sql-query-replace.md), [REPLICATE](sql-query-replicate.md), [REVERSE](sql-query-reverse.md), [RIGHT](sql-query-right.md), [RTRIM](sql-query-rtrim.md), [STARTSWITH](sql-query-startswith.md), [StringToArray](sql-query-stringtoarray.md), [StringToBoolean](sql-query-stringtoboolean.md), [StringToNull](sql-query-stringtonull.md), [StringToNumber](sql-query-stringtonumber.md), [StringToObject](sql-query-stringtoobject.md), [SUBSTRING](sql-query-substring.md), [ToString](sql-query-tostring.md), [TRIM](sql-query-trim.md), [UPPER](sql-query-upper.md) |
|[Fonctions de vérification du type](sql-query-type-checking-functions.md)|Les fonctions de vérification du type vous permettent de vérifier le type d'une expression donnée dans les requêtes SQL. | [IS_ARRAY](sql-query-is-array.md), [IS_BOOL](sql-query-is-bool.md), [IS_DEFINED](sql-query-is-defined.md), [IS_NULL](sql-query-is-null.md), [IS_NUMBER](sql-query-is-number.md), [IS_OBJECT](sql-query-is-object.md), [IS_PRIMITIVE](sql-query-is-primitive.md), [IS_STRING](sql-query-is-string.md) |

## <a name="built-in-versus-user-defined-functions-udfs"></a>Fonctions intégrées et fonctions définies par l’utilisateur

Si vous utilisez actuellement une fonction définie par l’utilisateur pour laquelle une fonction intégrée est désormais disponible, la fonction intégrée correspondante s’exécute plus rapidement et est plus performante.

## <a name="built-in-versus-ansi-sql-functions"></a>Fonctions intégrées et fonctions ANSI SQL

La principale différence entre les fonctions Cosmos DB et les fonctions ANSI SQL est que les fonctions Cosmos DB sont conçues pour s’exécuter avec des données sans schéma et des données avec un schéma mixte. Par exemple, si une propriété est manquante ou a une valeur non numérique, telle que `unknown`, l’élément est ignoré et aucune erreur n’est retournée.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Cosmos DB](introduction.md)
- [Fonctions de tableau](sql-query-array-functions.md)
- [Fonctions de date et heure](sql-query-date-time-functions.md)
- [Fonctions mathématiques](sql-query-mathematical-functions.md)
- [Fonctions spatiales](sql-query-spatial-functions.md)
- [Fonctions de chaîne](sql-query-string-functions.md)
- [Fonctions de vérification du type](sql-query-type-checking-functions.md)
- [Fonctions définies par l’utilisateur](sql-query-udfs.md)
- [Agrégats](sql-query-aggregate-functions.md)
