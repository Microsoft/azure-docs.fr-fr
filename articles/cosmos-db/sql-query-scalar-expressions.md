---
title: Expressions scalaires dans les requêtes SQL Azure Cosmos DB
description: Découvrez la syntaxe SQL des expressions scalaires pour Azure Cosmos DB. Cet article explique également comment combiner des expressions scalaires en expressions complexes à l’aide d’opérateurs.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/17/2019
ms.author: mjbrown
ms.openlocfilehash: 34df2d2f4a2209cca93c3f7ac12dcd203bf4a089
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339629"
---
# <a name="scalar-expressions-in-azure-cosmos-db-sql-queries"></a>Expressions scalaires dans les requêtes SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La [clause SELECT](sql-query-select.md) prend en charge les expressions scalaires. Une expression scalaire est une combinaison de symboles et d’opérateurs qui peut être évaluée pour obtenir une valeur unique. Voici quelques exemples d’expressions scalaires : constantes, références de propriété, références d’élément de tableau, références d’alias ou appels de fonction. Les expressions scalaires peuvent être combinées dans des expressions complexes utilisant des opérateurs.

## <a name="syntax"></a>Syntaxe
  
```sql  
<scalar_expression> ::=  
       <constant>
     | input_alias
     | parameter_name  
     | <scalar_expression>.property_name  
     | <scalar_expression>'['"property_name"|array_index']'  
     | unary_operator <scalar_expression>  
     | <scalar_expression> binary_operator <scalar_expression>    
     | <scalar_expression> ? <scalar_expression> : <scalar_expression>  
     | <scalar_function_expression>  
     | <create_object_expression>
     | <create_array_expression>  
     | (<scalar_expression>)
  
<scalar_function_expression> ::=  
        'udf.' Udf_scalar_function([<scalar_expression>][,…n])  
        | builtin_scalar_function([<scalar_expression>][,…n])  
  
<create_object_expression> ::=  
   '{' [{property_name | "property_name"} : <scalar_expression>][,…n] '}'  
  
<create_array_expression> ::=  
   '[' [<scalar_expression>][,…n] ']'  
  
```

## <a name="arguments"></a>Arguments
  
- `<constant>`  
  
   Représente une valeur constante. Consultez la section [Constantes](sql-query-constants.md) pour plus d’informations.  
  
- `input_alias`  
  
   Représente une valeur définie par le `input_alias` introduit dans la clause `FROM`.  
  Cette valeur est assurée de ne pas être **Undefined** , car les valeurs **Undefined** dans l’entrée sont ignorées.  
  
- `<scalar_expression>.property_name`  
  
   Représente une valeur de la propriété d’un objet. Si la propriété n’existe pas ou est référencée sur une valeur, qui n’est pas un objet, l’expression évaluée présente la valeur **Undefined**.  
  
- `<scalar_expression>'['"property_name"|array_index']'`  
  
   Représente une valeur de la propriété portant le nom `property_name` ou l’élément de tableau avec l’index `array_index` d’un tableau. Si la propriété/l’index de tableau n’existe pas ou est référencée sur une valeur qui n’est pas un objet/tableau, alors l’expression évaluée présente la valeur Undefined.  
  
- `unary_operator <scalar_expression>`  
  
   Représente un opérateur appliqué à une valeur unique. Consultez la section [Opérateurs](sql-query-operators.md) pour plus d’informations.  
  
- `<scalar_expression> binary_operator <scalar_expression>`  
  
   Représente un opérateur appliqué à deux valeurs. Consultez la section [Opérateurs](sql-query-operators.md) pour plus d’informations.  
  
- `<scalar_function_expression>`  
  
   Représente une valeur définie par le résultat d’un appel de fonction.  
  
- `udf_scalar_function`  
  
   Nom de la fonction scalaire définie par l’utilisateur.  
  
- `builtin_scalar_function`  
  
   Nom de la fonction scalaire intégrée.  
  
- `<create_object_expression>`  
  
   Représente une valeur obtenue en créant un nouvel objet avec les propriétés spécifiées et leurs valeurs.  
  
- `<create_array_expression>`  
  
   Représente une valeur obtenue en créant un nouveau tableau avec les valeurs spécifiées en tant qu’éléments  
  
- `parameter_name`  
  
   Représente une valeur du nom de paramètre spécifié. Les noms de paramètre doivent avoir un \@ unique comme premier caractère.  
  
## <a name="remarks"></a>Notes
  
  Lors de l’appel à une fonction scalaire intégrée ou définie par l’utilisateur, tous les arguments doivent être définis. Si un des arguments n’est pas défini, la fonction n’est pas appelée et le résultat est Undefined.  
  
  Lorsque vous créez un objet, toute propriété à laquelle est affectée une valeur Undefined est ignorée et n’est pas incluse dans l’objet créé.  
  
  Lorsque vous créez un tableau, toute valeur d’élément à laquelle est affectée une valeur **Undefined** est ignorée et n’est pas incluse dans l’objet créé. Dans ce cas, l’élément défini suivant prend sa place de telle sorte que le tableau créé n’aura pas d’index ignorés.  

## <a name="examples"></a>Exemples

```sql
    SELECT ((2 + 11 % 7)-2)/3
```

Les résultats sont :

```json
    [{
      "$1": 1.33333
    }]
```

Dans la requête suivante, le résultat de l’expression scalaire est un booléen :

```sql
    SELECT f.address.city = f.address.state AS AreFromSameCityState
    FROM Families f
```

Les résultats sont :

```json
    [
      {
        "AreFromSameCityState": false
      },
      {
        "AreFromSameCityState": true
      }
    ]
```

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Cosmos DB](introduction.md)
- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Subqueries](sql-query-subquery.md)