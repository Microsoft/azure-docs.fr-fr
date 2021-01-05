---
title: Utilisation de JSON dans Azure Cosmos DB
description: Découvrez l’interrogation et la consultation de propriétés JSON imbriquées, ainsi que l’utilisation de caractères spéciaux dans Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/19/2020
ms.author: tisande
ms.openlocfilehash: 9a9300db1adc3ff238c44887012400702690b0e8
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93337827"
---
# <a name="working-with-json-in-azure-cosmos-db"></a>Utilisation de JSON dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans l’API SQL (Core) d’Azure Cosmos DB, les éléments sont stockés au format JSON. Le système de type et les expressions peuvent uniquement traiter des types JSON. Pour en savoir plus, consultez la [spécification JSON](https://www.json.org/).

Nous allons résumer certains aspects importants de l’utilisation de JSON :

- Les objets JSON commencent toujours par une accolade ouvrante `{` et se terminent par une accolade fermante `}`.
- Vous pouvez avoir des propriétés JSON [imbriquées](#nested-properties) les unes dans les autres.
- Les valeurs de propriétés JSON peuvent être des tableaux.
- Les noms de propriétés JSON respectent la casse.
- Le nom d’une propriété JSON peut être n’importe quelle valeur de chaîne (y compris des espaces ou des signes autres que des lettres).

## <a name="nested-properties"></a>Propriétés imbriquées

Vous pouvez accéder à un JSON imbriqué à l’aide d’un accesseur de point. Vous pouvez utiliser des propriétés JSON imbriquées dans vos requêtes de la même façon que vous pouvez en utiliser d’autres.

Voici un document contenant du code JSON imbriqué :

```JSON
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "address": {
      "state": "WA",
      "county": "King",
      "city": "Seattle"
      },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

Dans ce cas, les propriétés `state`, `country` et `city` sont toutes imbriquées dans la propriété `address`.

L’exemple suivant projette deux propriétés imbriquées : `f.address.state` et `f.address.city`.

```sql
    SELECT f.address.state, f.address.city
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Les résultats sont :

```json
    [{
      "state": "WA",
      "city": "Seattle"
    }]
```

## <a name="working-with-arrays"></a>Utilisation de tableaux

Outre les propriétés imbriquées, JSON prend en charge les tableaux.

Voici un exemple de document comportant un tableau :

```json
{
  "id": "WakefieldFamily",
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
      },
      {
        "familyName": "Miller",
         "givenName": "Lisa",
         "gender": "female",
         "grade": 8
      }
  ],
}
```

Lorsque vous utilisez des tableaux, vous pouvez accéder à un élément spécifique dans un tableau en référençant sa position :

```sql
SELECT *
FROM Families f
WHERE f.children[0].givenName = "Jesse"
```

Toutefois, le plus souvent, vous utiliserez une [sous-requête](sql-query-subquery.md) ou une [jointure réflexive](sql-query-join.md).

Par exemple, voici un document qui montre le solde quotidien du compte bancaire d’un client.

```json
{
  "id": "Contoso-Checking-Account-2020",
  "balance": [
      {
        "checkingAccount": 1000,
        "savingsAccount": 5000
      },
      {
        "checkingAccount": 100,
        "savingsAccount": 5000
      },
      {
        "checkingAccount": -10,
        "savingsAccount": 5000,
      },
      {
        "checkingAccount": 5000,
        "savingsAccount": 5000,
      }
         ...
  ]
}
```

Si vous souhaitiez exécuter une requête affichant tous les clients ayant un solde négatif à un moment donné, vous pourriez utiliser la commande [EXISTS](sql-query-subquery.md#exists-expression) avec une sous-requête :

```sql
SELECT c.id
FROM c
WHERE EXISTS(
    SELECT VALUE n
    FROM n IN c.balance
    WHERE n.checkingAccount < 0
)
```

## <a name="difference-between-null-and-undefined"></a>Différence entre une valeur Null et non définie

Si une propriété n’est pas définie dans un élément, sa valeur est `undefined`. Une propriété dont la valeur est `null` doit être définie explicitement et se voir attribuer une valeur `null`.

Considérons par exemple cet exemple d’élément :

```json
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "address": {
      "state": "WA",
      "county": "King",
      "city": "Seattle"
      },
  "creationDate": null
}
```

Dans cet exemple, la propriété `isRegistered` a la valeur `undefined`, car elle est omise de l’élément. La propriété `creationDate` a la valeur `null`.

Azure Cosmos DB prend en charge deux fonctions système de contrôle de type utiles pour les propriétés `null` et `undefined` :

* [IS_NULL](sql-query-is-null.md) : vérifie si une valeur de propriété est `null`
* [IS_DEFINED](sql-query-is-defined.md) : vérifie si une valeur de propriété est définie

Vous pouvez vous renseigner sur les [opérateurs pris en charge](sql-query-operators.md) et leur comportement pour les valeurs `null` et `undefined`.

## <a name="reserved-keywords-and-special-characters-in-json"></a>Mots clés réservés et caractères spéciaux dans JSON

Vous pouvez accéder à des propriétés en utilisant l’opérateur de propriété entre guillemets `[]`. Par exemple, `SELECT c.grade` and `SELECT c["grade"]` sont équivalentes. Cette syntaxe est utile pour placer dans une séquence d’échappement une propriété qui contient des espaces, des caractères spéciaux, ou qui a le même nom qu’un mot clé SQL ou un mot réservé.

Par exemple, voici un document avec une propriété nommée `order` et une propriété `price($)` qui contient des caractères spéciaux :

```json
{
  "id": "AndersenFamily",
  "order": {
         "orderId": "12345",
         "productId": "A17849",
         "price($)": 59.33
   },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

Si vous exécutez des requêtes qui incluent la propriété `order` ou `price($)`, vous recevez une erreur de syntaxe.

```sql
SELECT * FROM c where c.order.orderid = "12345"
```

```sql
SELECT * FROM c where c.order.price($) > 50
```

Le résultat est le suivant :

`
Syntax error, incorrect syntax near 'order'
`

Vous devez réécrire les mêmes requêtes comme indiqué ci-dessous :

```sql
SELECT * FROM c WHERE c["order"].orderId = "12345"
```

```sql
SELECT * FROM c WHERE c["order"]["price($)"] > 50
```

## <a name="json-expressions"></a>Expressions JSON

La projection prend également en charge les expressions JSON, comme le montre l’exemple suivant :

```sql
    SELECT { "state": f.address.state, "city": f.address.city, "name": f.id }
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Les résultats sont :

```json
    [{
      "$1": {
        "state": "WA",
        "city": "Seattle",
        "name": "AndersenFamily"
      }
    }]
```

Dans l’exemple précédent, la clause `SELECT` a besoin de créer un objet JSON, et étant donné que l’exemple ne fournit aucune clé, la clause utilise le nom de variable d’argument implicite `$1`. La requête suivante retourne deux variables d’argument implicites : `$1` et `$2`.

```sql
    SELECT { "state": f.address.state, "city": f.address.city },
           { "name": f.id }
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Les résultats sont :

```json
    [{
      "$1": {
        "state": "WA",
        "city": "Seattle"
      },
      "$2": {
        "name": "AndersenFamily"
      }
    }]
```

## <a name="aliasing"></a>Alias

Vous pouvez explicitement attribuer des alias à des valeurs dans les requêtes. Si une requête a deux propriétés portant le même nom, utilisez l’attribution d’alias pour renommer l’une ou l’autre des propriétés, afin d’éviter toute ambiguïté dans le résultat projeté.

### <a name="examples"></a>Exemples

Le mot clé `AS` utilisé pour l’attribution d’alias est facultatif, comme le montre l’exemple suivant, lors de la projection de la deuxième valeur en tant que `NameInfo` :

```sql
    SELECT
           { "state": f.address.state, "city": f.address.city } AS AddressInfo,
           { "name": f.id } NameInfo
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Les résultats sont :

```json
    [{
      "AddressInfo": {
        "state": "WA",
        "city": "Seattle"
      },
      "NameInfo": {
        "name": "AndersenFamily"
      }
    }]
```

### <a name="aliasing-with-reserved-keywords-or-special-characters"></a>Attribution d’alias avec des mots clés réservés ou des caractères spéciaux

Vous ne pouvez pas utiliser d’alias pour projeter une valeur en tant que nom de propriété avec une espace, un caractère spécial ou un mot réservé. Si vous souhaitez modifier la projection d’une valeur en, par exemple, avoir un nom de propriété avec une espace, vous pouvez utiliser une expression [JSON](#json-expressions).

Voici un exemple :

```sql
    SELECT
           {"JSON expression with a space": { "state": f.address.state, "city": f.address.city }},
           {"JSON expression with a special character!": { "name": f.id }}
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Clause SELECT](sql-query-select.md)
- [Clause WHERE](sql-query-where.md)
