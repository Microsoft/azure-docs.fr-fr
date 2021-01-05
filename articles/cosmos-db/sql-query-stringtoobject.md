---
title: 'Langage de requête Azure Cosmos DB : StringToObject'
description: Découvrez la fonction système SQL StringToObject dans Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 369c86c8fc2ef4d125c3f13db4eb3af88b7e726e
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93337844"
---
# <a name="stringtoobject-azure-cosmos-db"></a>StringToObject (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Retourne une expression convertie en objet. Si l’expression ne peut pas être convertie, le résultat est indéfini.  
  
## <a name="syntax"></a>Syntaxe
  
```sql
StringToObject(<str_expr>)  
```  
  
## <a name="arguments"></a>Arguments
  
*str_expr*  
   Est une expression de chaîne à analyser comme expression d’objet JSON. Notez que les valeurs de chaîne imbriquées doivent être écrites avec des guillemets doubles pour être valides. Pour plus d’informations sur le format JSON, consultez [json.org](https://json.org/)  
  
## <a name="return-types"></a>Types de retour
  
  Retourne une expression d’objet ou une valeur non définie.  
  
## <a name="examples"></a>Exemples
  
  L’exemple suivant montre comment `StringToObject` se comporte avec différents types. 
  
 Voici quelques exemples avec une entrée valide.

```sql
SELECT 
    StringToObject("{}") AS obj1, 
    StringToObject('{"A":[1,2,3]}') AS obj2,
    StringToObject('{"B":[{"b1":[5,6,7]},{"b2":8},{"b3":9}]}') AS obj3, 
    StringToObject("{\"C\":[{\"c1\":[5,6,7]},{\"c2\":8},{\"c3\":9}]}") AS obj4
``` 

Voici le jeu de résultats obtenu.

```json
[{"obj1": {}, 
  "obj2": {"A": [1,2,3]}, 
  "obj3": {"B":[{"b1":[5,6,7]},{"b2":8},{"b3":9}]},
  "obj4": {"C":[{"c1":[5,6,7]},{"c2":8},{"c3":9}]}}]
```

 Voici quelques exemples avec une entrée non valide.
Même s’ils sont valides dans une requête, leur analyse ne permet pas d’obtenir des objets valides. Les chaînes dans la chaîne d’objet doivent être placées dans une séquence d’échappement ("{\\"a\\":\\"str\\"}") ou les guillemets de début et de fin doivent être des guillemets simples ('{"a": "str"}').

Les guillemets simples autour des noms de propriété ne sont pas valides en JSON.

```sql
SELECT 
    StringToObject("{'a':[1,2,3]}")
```

Voici le jeu de résultats obtenu.

```json
[{}]
```  

Les noms de propriété non entourés de guillemets ne sont pas valides en JSON.

```sql
SELECT 
    StringToObject("{a:[1,2,3]}")
```

Voici le jeu de résultats obtenu.

```json
[{}]
``` 

Voici quelques exemples avec une entrée non valide.

 L’expression passée est analysée en tant qu’objet JSON ; ces entrées n’étant pas évaluées comme étant de type objet, une valeur non définie est retournée.

```sql
SELECT 
    StringToObject("}"),
    StringToObject("{"),
    StringToObject("1"),
    StringToObject(NaN), 
    StringToObject(false), 
    StringToObject(undefined)
``` 
 
 Voici le jeu de résultats obtenu.

```json
[{}]
```

## <a name="remarks"></a>Notes

Cette fonction système n’utilisera pas l’index.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de chaîne Azure Cosmos DB](sql-query-string-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
