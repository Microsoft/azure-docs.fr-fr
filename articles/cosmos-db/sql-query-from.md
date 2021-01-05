---
title: Clause FROM dans Azure Cosmos DB
description: Découvrez la syntaxe SQL et l’exemple de clause FROM pour Azure Cosmos DB. Cet article contient également des exemples montrant comment définir l’étendue des résultats et obtenir des sous-éléments à l’aide de la clause FROM.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: tisande
ms.openlocfilehash: 3844a628a8e3773821736324bfdaab3f64ff9da5
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336246"
---
# <a name="from-clause-in-azure-cosmos-db"></a>Clause FROM dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La clause FROM (`FROM <from_specification>`) est facultative, sauf si la source est filtrée ou projetée plus loin dans la requête. Une requête comme `SELECT * FROM Families` énumère l’ensemble du conteneur `Families`. Vous pouvez également utiliser l’identificateur ROOT spécial pour le conteneur au lieu d’utiliser le nom du conteneur.

La clause `FROM` applique les règles suivantes par requête :

* Le conteneur peut être un alias, tel que `SELECT f.id FROM Families AS f` ou simplement `SELECT f.id FROM Families f`. Ici, `f` est l’alias de `Families`. AS est un mot clé facultatif pour [attribuer un alias](sql-query-working-with-json.md#aliasing) à l’identificateur.  

* Une fois l’alias attribué, vous ne pouvez plus lier le nom source d’origine. Par exemple, `SELECT Families.id FROM Families f` est syntaxiquement incorrect, car l’identificateur `Families`, qui s’est vu attribuer un alias, ne peut plus être résolu.  

* Toutes les propriétés référencées doivent être complètes, afin d’éviter toute liaison ambiguë en l’absence de conformité de schéma strict. Par exemple, `SELECT id FROM Families f` est syntaxiquement incorrect, car la propriété `id` n’est pas liée.

## <a name="syntax"></a>Syntaxe
  
```sql  
FROM <from_specification>  
  
<from_specification> ::=
        <from_source> {[ JOIN <from_source>][,...n]}  
  
<from_source> ::=
          <container_expression> [[AS] input_alias]  
        | input_alias IN <container_expression>  
  
<container_expression> ::=
        ROOT
     | container_name  
     | input_alias  
     | <container_expression> '.' property_name  
     | <container_expression> '[' "property_name" | array_index ']'  
```  
  
## <a name="arguments"></a>Arguments
  
- `<from_source>`  
  
  Spécifie une source de données, avec ou sans alias. Si aucun alias n’est spécifié, il est déduit à partir de `<container_expression>` à l’aide des règles suivantes :  
  
-  Si l’expression est un container_name, alors container_name sera être utilisé en tant qu’alias.  
  
-  Si l’expression est `<container_expression>`, puis property_name, alors property_name sera être utilisé en tant qu’alias. Si l’expression est un container_name, alors container_name sera être utilisé en tant qu’alias.  
  
- AS `input_alias`  
  
  Spécifie que `input_alias` est un ensemble de valeurs renvoyées par l’expression de conteneur sous-jacent.  
 
- `input_alias` IN  
  
  Spécifie que `input_alias` doit représenter l’ensemble des valeurs obtenues en effectuant une itération sur tous les éléments de tableau de chaque tableau retourné par l’expression de conteneur sous-jacent. Toute valeur retournée par l’expression de conteneur sous-jacent qui n’est pas un tableau est ignorée.  
  
- `<container_expression>`  
  
  Spécifie l’expression de conteneur à utiliser pour récupérer les documents.  
  
- `ROOT`  
  
  Spécifie qu’un document doit être récupéré à partir du conteneur par défaut, actuellement connecté.  
  
- `container_name`  
  
  Spécifie qu’un document doit être récupéré à partir du conteneur fourni. Le nom du conteneur doit correspondre au nom du conteneur actuellement connecté.  
  
- `input_alias`  
  
  Spécifie que le document doit être récupéré à partir de l’autre source définie par l’alias fourni.  
  
- `<container_expression> '.' property_name`  
  
  Spécifie qu’un document doit être récupéré en accédant à la propriété `property_name`.  
  
- `<container_expression> '[' "property_name" | array_index ']'`  
  
  Spécifie que le document doit être récupéré en accédant à la propriété `property_name` ou à l’élément de tableau array_index pour tous les documents récupérés par l’expression de conteneur spécifiée.  
  
## <a name="remarks"></a>Notes
  
Tous les alias fournis ou déduits dans la ou les `<from_source>(` doivent être uniques. La syntaxe `<container_expression>.`property_name est identique à `<container_expression>' ['"property_name"']'`. Toutefois, cette dernière syntaxe peut être utilisée si un nom de propriété contient un caractère hors identificateur.  
  
### <a name="handling-missing-properties-missing-array-elements-and-undefined-values"></a>Gestion des propriétés manquantes, des éléments de tableau manquants et des valeurs non définies
  
Si une expression de conteneur accède à des propriétés ou éléments de tableau et que la valeur n’existe pas, cette valeur sera ignorée et n’est plus traitée.  
  
### <a name="container-expression-context-scoping"></a>Étendue de contexte d’expression de conteneur  
  
Une expression de conteneur peut avoir une étendue de document ou de conteneur :  
  
- Une expression est étendue à un conteneur si la source sous-jacente de l’expression de conteneur est ROOT ou `container_name`. Une telle expression représente un ensemble de documents récupérés directement à partir du conteneur, et n’est pas dépendante du traitement d’autres expressions de conteneur.  
  
- Une expression est étendue à un document si la source sous-jacente de l’expression de conteneur est `input_alias`, introduit plus tôt dans la requête. Une telle expression représente un ensemble de documents obtenus en évaluant l’expression de conteneur dans l’étendue de chaque document appartenant au jeu associé au conteneur avec alias. Le jeu résultant sera une union de jeux obtenus en évaluant l’expression de conteneur pour chacun des documents du jeu sous-jacent.

## <a name="examples"></a>Exemples

### <a name="get-subitems-by-using-the-from-clause"></a>Obtenir des sous-éléments à l’aide de la clause FROM

La clause FROM peut réduire la source à un sous-ensemble plus petit. Pour l’énumération d’une seule sous-arborescence de chaque élément, le sous-dossier racine peut devenir la source, comme dans l’exemple suivant :

```sql
    SELECT *
    FROM Families.children
```

Les résultats sont :

```json
    [
      [
        {
            "firstName": "Henriette Thaulow",
            "gender": "female",
            "grade": 5,
            "pets": [
              {
                  "givenName": "Fluffy"
              }
            ]
        }
      ],
      [
       {
            "familyName": "Merriam",
            "givenName": "Jesse",
            "gender": "female",
            "grade": 1
        },
        {
            "familyName": "Miller",
            "givenName": "Lisa",
            "gender": "female",
            "grade": 8
        }
      ]
    ]
```

La requête précédente a utilisé un tableau comme source, mais vous pouvez également utiliser un objet comme source. La requête prend en compte toute valeur JSON définie valide dans la source en vue de son inclusion dans le résultat. L’exemple suivant exclut les éléments `Families` qui n’ont pas de valeur `address.state`.

```sql
    SELECT *
    FROM Families.address.state
```

Les résultats sont :

```json
    [
      "WA",
      "NY"
    ]
```

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Clause SELECT](sql-query-select.md)
- [Clause WHERE](sql-query-where.md)
