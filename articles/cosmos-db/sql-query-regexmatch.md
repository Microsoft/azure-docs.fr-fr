---
title: RegexMatch dans langage de requête Azure Cosmos DB
description: En savoir plus sur la fonction système SQL RegexMatch dans Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 7f5b2831f45e902b312636e4133557a16ee7ec95
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341635"
---
# <a name="regexmatch-azure-cosmos-db"></a>REGEXMATCH (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Fournit des fonctionnalités d’expressions régulières. Les expressions régulières sont des notations concises et souples pour la recherche de modèles de texte. Azure Cosmos DB utilise les[PCRE (expressions régulières compatibles PERL)](http://www.pcre.org/). 

## <a name="syntax"></a>Syntaxe
  
```sql
RegexMatch(<str_expr1>, <str_expr2>, [, <str_expr3>])  
```  
  
## <a name="arguments"></a>Arguments
  
*str_expr1*  
   Est l’expression de chaîne faisant l’objet de la recherche.  
  
*str_expr2*  
   Est l’expression régulière.

*str_expr3*  
   Chaîne de modificateurs sélectionnés à utiliser avec l’expression régulière. Cette valeur de chaîne est facultative. Si vous souhaitez exécuter RegexMatch sans modificateurs, vous pouvez soit ajouter une chaîne vide, soit l’omettre entièrement. 

Vous pouvez en savoir plus sur la syntaxe [pour créer des expressions régulières dans les Perl](https://perldoc.perl.org/perlre). 

Azure Cosmos DB prend en charge les quatre modificateurs suivants :

| Modificateur | Description |
| ------ | ----------- |
| `m` | Traitez l’expression de chaîne à rechercher sur plusieurs lignes. Sans cette option, « ^ » et « $ » correspondent au début ou à la fin de la chaîne et non à chaque ligne individuelle. |
| `s` | Autorise « . » à correspondre à n’importe quel caractère, y compris un caractère de saut de ligne. | 
| `i` | Ignorer la casse lors de la recherche de critères spéciaux. |
| `x` | Ignorer tous les caractères d’espace blanc. |

## <a name="return-types"></a>Types de retour
  
  Retourne une expression booléenne. Retourne une valeur non définie si l’expression de chaîne à rechercher, l’expression régulière ou les modificateurs sélectionnés ne sont pas valides.
  
## <a name="examples"></a>Exemples
  
L’exemple RegexMatch simple suivant vérifie la chaîne « ABCD » pour la correspondance d’expression régulière à l’aide de quelques modificateurs différents.
  
```sql
SELECT RegexMatch ("abcd", "ABC", "") AS NoModifiers, 
RegexMatch ("abcd", "ABC", "i") AS CaseInsensitive, 
RegexMatch ("abcd", "ab.", "") AS WildcardCharacter,
RegexMatch ("abcd", "ab c", "x") AS IgnoreWhiteSpace, 
RegexMatch ("abcd", "aB c", "ix") AS CaseInsensitiveAndIgnoreWhiteSpace 
```  
  
 Voici le jeu de résultats obtenu.  
  
```json
[
    {
        "NoModifiers": false,
        "CaseInsensitive": true,
        "WildcardCharacter": true,
        "IgnoreWhiteSpace": true,
        "CaseInsensitiveAndIgnoreWhiteSpace": true
    }
]
```

Avec RegexMatch, vous pouvez utiliser des métacaractères pour effectuer des recherches de chaînes plus complexes qui seraient autrement possibles avec les fonctions système StartsWith, EndsWith, CONTAINS ou StringEquals. Voici quelques exemples supplémentaires que vous pouvez exécuter à l’aide du jeu de données nutritionnel disponible par le biais du [Playground de test de requêtes Azure Cosmos DB](https://www.documentdb.com/sql/demo). 

> [!NOTE] 
> Si vous devez utiliser un métacaractère dans une expression régulière et que vous ne souhaitez pas qu’il ait une signification particulière, vous devez sortir le métacaractère à l’aide de `\`.

**Vérifiez les éléments dont la description contient le mot « Salt » exactement une fois :**

```sql
SELECT * 
FROM c 
WHERE RegexMatch (c.description, "salt{1}","")
```

**Vérifiez les éléments dont la description contient un nombre compris entre 0 et 99 :**

```sql
SELECT * 
FROM c 
WHERE RegexMatch (c.description, "[0-99]","")
```

**Vérifiez les éléments dont la description contient quatre mots de lettres commençant par « S » ou « s » :**

```sql
SELECT * 
FROM c 
WHERE RegexMatch (c.description, " s... ","i")
```

## <a name="remarks"></a>Remarques

Cette fonction système tire parti d’un [index de plage](index-policy.md#includeexclude-strategy) si l’expression régulière peut être décomposée en fonctions de système StartsWith, EndsWith, CONTAINS ou StringEquals.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de chaîne Azure Cosmos DB](sql-query-string-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](introduction.md)
