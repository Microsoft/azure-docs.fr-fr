---
title: Fonctions définies par l’utilisateur dans Azure Cosmos DB
description: Découvrez les fonctions définies par l’utilisateur dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: 98698264f0beb25a8b4f74861f1150ae889d7115
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546333"
---
# <a name="user-defined-functions-udfs-in-azure-cosmos-db"></a>Fonctions définies par l’utilisateur dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

L’API SQL prend en charge les fonctions définies par l’utilisateur. Les fonctions scalaires définies par l’utilisateur vous permettent de passer zéro ou plusieurs arguments et de retourner un résultat contenant un seul argument. L’API vérifie que chaque argument est une valeur JSON légale.  

## <a name="udf-use-cases"></a>Fonctions définies par l'utilisateur - Cas d'usage

L’API étend la syntaxe SQL pour prendre en charge la logique d’application personnalisée à l’aide de fonctions définies par l’utilisateur. Vous pouvez inscrire les fonctions définies par l’utilisateur auprès de l’API SQL et les référencer dans les requêtes SQL. Contrairement aux procédures stockées et aux déclencheurs, les fonctions définies par l'utilisateur sont en lecture seule.

Les fonctions définies par l'utilisateur vous permettent d'étendre le langage de requête d'Azure Cosmos DB. Les fonctions définies par l'utilisateur sont un excellent moyen d'exprimer une logique métier complexe dans la projection d'une requête.

Nous vous recommandons toutefois d'éviter les fonctions définies par l'utilisateur dans les cas suivants :

- Une [fonction système](sql-query-system-functions.md) équivalente existe déjà dans Azure Cosmos DB. Les fonctions système utilisent toujours moins d'unités de requête que la fonction définie par l'utilisateur équivalente.
- La fonction définie par l'utilisateur est le seul filtre de la clause `WHERE` de votre requête. Les fonctions définies par l'utilisateur n'utilisent pas l'index. Par conséquent, leur évaluation nécessitera le chargement de documents. La combinaison de prédicats de filtre supplémentaires qui utilisent l'index, en association avec une fonction définie par l'utilisateur, dans la clause `WHERE` réduit le nombre de documents traités par la fonction définie par l'utilisateur.

Si vous devez utiliser plusieurs fois la même fonction définie par l'utilisateur dans une requête, vous devez la référencer dans une [sous-requête](sql-query-subquery.md#evaluate-once-and-reference-many-times), ce qui vous permet d'utiliser une expression JOIN pour évaluer une fois la fonction définie par l'utilisateur tout en la référençant plusieurs fois.

## <a name="examples"></a>Exemples

L’exemple suivant inscrit une fonction définie par l’utilisateur sous un conteneur d’éléments dans la base de données Cosmos. L’exemple crée une fonction définie par l’utilisateur dont le nom est `REGEX_MATCH`. Elle accepte les deux valeurs de chaîne JSON `input` et `pattern`, et vérifie si la première correspond au modèle spécifié dans la seconde à l’aide de la fonction `string.match()` de JavaScript.

```javascript
       UserDefinedFunction regexMatchUdf = new UserDefinedFunction
       {
           Id = "REGEX_MATCH",
           Body = @"function (input, pattern) {
                      return input.match(pattern) !== null;
                   };",
       };

       UserDefinedFunction createdUdf = client.CreateUserDefinedFunctionAsync(
           UriFactory.CreateDocumentCollectionUri("myDatabase", "families"),
           regexMatchUdf).Result;  
```

Utilisez maintenant cette fonction définie par l’utilisateur dans une projection de requête. Vous devez qualifier les fonctions définies par l’utilisateur avec le préfixe sensible à la casse `udf.` quand vous les appelez à partir de requêtes.

```sql
    SELECT udf.REGEX_MATCH(Families.address.city, ".*eattle")
    FROM Families
```

Les résultats sont :

```json
    [
      {
        "$1": true
      },
      {
        "$1": false
      }
    ]
```

Vous pouvez utiliser la fonction définie par l’utilisateur qualifiée avec le préfixe `udf.` à l’intérieur d’un filtre, comme dans l’exemple suivant :

```sql
    SELECT Families.id, Families.address.city
    FROM Families
    WHERE udf.REGEX_MATCH(Families.address.city, ".*eattle")
```

Les résultats sont :

```json
    [{
        "id": "AndersenFamily",
        "city": "Seattle"
    }]
```

Par essence, les fonctions définies par l’utilisateur sont des expressions scalaires valides que vous pouvez utiliser dans des projections et des filtres.

Pour développer la puissance des fonctions définies par l’utilisateur, étudiez un autre exemple de logique conditionnelle :

```javascript
       UserDefinedFunction seaLevelUdf = new UserDefinedFunction()
       {
           Id = "SEALEVEL",
           Body = @"function(city) {
                   switch (city) {
                       case 'Seattle':
                           return 520;
                       case 'NY':
                           return 410;
                       case 'Chicago':
                           return 673;
                       default:
                           return -1;
                    }"
            };

            UserDefinedFunction createdUdf = await client.CreateUserDefinedFunctionAsync(
                UriFactory.CreateDocumentCollectionUri("myDatabase", "families"),
                seaLevelUdf);
```

L’exemple suivant teste la fonction définie par l’utilisateur :

```sql
    SELECT f.address.city, udf.SEALEVEL(f.address.city) AS seaLevel
    FROM Families f
```

Les résultats sont :

```json
     [
      {
        "city": "Seattle",
        "seaLevel": 520
      },
      {
        "city": "NY",
        "seaLevel": 410
      }
    ]
```

Si les propriétés auxquelles font référence les paramètres des fonctions définies par l’utilisateur ne sont pas disponibles dans la valeur JSON, les paramètres sont considérés comme non définis et l’appel des fonctions définies par l’utilisateur est ignoré. De même, si le résultat des fonctions définies par l’utilisateur est indéfini, il n’est pas inclus dans le résultat.

Comme le montrent les exemples précédents, les fonctions définies par l’utilisateur intègrent la puissance du langage JavaScript à l’API SQL. Les fonctions définies par l’utilisateur fournissent une interface programmable enrichie qui permet de concevoir une logique conditionnelle procédurale complexe à l’aide de fonctionnalités de runtime JavaScript intégrées. L’API SQL fournit les arguments aux fonctions définies par l’utilisateur pour chaque élément source à l’étape actuelle du traitement concernant la clause WHERE ou SELECT. Le résultat est intégré au pipeline d’exécution générale de manière fluide. En résumé, les fonctions définies par l’utilisateur sont des outils efficaces pour mener à bien des logiques métier complexes dans le cadre de requêtes.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Cosmos DB](introduction.md)
- [Fonctions système](sql-query-system-functions.md)
- [Agrégats](sql-query-aggregate-functions.md)