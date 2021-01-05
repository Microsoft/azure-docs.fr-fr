---
title: Gérer l’indexation dans l’API pour MongoDB d’Azure Cosmos DB
description: Cet article présente une vue d’ensemble des fonctionnalités d’indexation d’Azure Cosmos DB à l’aide de l’API Azure Cosmos DB pour MongoDB
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 11/06/2020
author: timsander1
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: e920af85c511387e66bcafcb6a140844d25f204c
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369288"
---
# <a name="manage-indexing-in-azure-cosmos-dbs-api-for-mongodb"></a>Gérer l’indexation dans l’API pour MongoDB d’Azure Cosmos DB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

L’API d’Azure Cosmos DB pour MongoDB tire parti des fonctionnalités de base de gestion des index d’Azure Cosmos DB. Cet article décrit comment ajouter des index à l’aide de l’API d’Azure Cosmos DB pour MongoDB. Vous pouvez également lire une [vue d’ensemble de l’indexation dans Azure Cosmos DB](index-overview.md) qui est pertinente pour toutes les API.

## <a name="indexing-for-mongodb-server-version-36"></a>Indexation pour le serveur MongoDB version 3.6

L’API d’Azure Cosmos DB pour le serveur MongoDB version 3.6 indexe automatiquement le champ `_id`, qui ne peut pas être abandonné. Elle applique automatiquement l’unicité du champ `_id` par clé de partition. Dans l’API d’Azure Cosmos DB pour MongoDB, le partitionnement et l’indexation sont des concepts distincts. Vous n’êtes pas tenu d’indexer votre clé de partition. Toutefois, comme pour toute autre propriété dans votre document, si cette propriété est un filtre courant dans vos requêtes, nous vous recommandons d’indexer la clé de partition.

Pour indexer des champs supplémentaires, appliquez les commandes de gestion d’index MongoDB. Comme dans MongoDB, l’API d’Azure Cosmos DB pour MongoDB indexe automatiquement le champ `_id` uniquement. Cette stratégie d’indexation par défaut est différente de celle de l’API SQL Azure Cosmos DB, qui indexe tous les champs par défaut.

Pour appliquer un tri sur une requête, vous devez créer un index sur les champs utilisés dans l’opération de tri.

## <a name="index-types"></a>Types d’index

### <a name="single-field"></a>Champ unique

Vous pouvez créer des index sur n’importe quel champ unique. L’ordre de tri d’un index monochamp n’a pas d’importance. La commande suivante crée un index sur le champ `name` :

`db.coll.createIndex({name:1})`

Une requête utilise plusieurs index monochamps, le cas échéant. Vous pouvez créer jusqu’à 500 index monochamp par conteneur.

### <a name="compound-indexes-mongodb-server-version-36"></a>Index composés (serveur MongoDB version 3.6)

L’API d’Azure Cosmos DB pour MongoDB prend en charge les index composés pour les comptes qui utilisent le protocole Wire filaire 3.6. Vous pouvez ajouter jusqu’à huit champs dans un index composé. Contrairement à MongoDB, vous devez créer un index composé uniquement si votre requête doit effectuer un tri efficace sur plusieurs champs à la fois. Pour les requêtes avec plusieurs filtres qui n’ont pas besoin de tri, créez plusieurs index monochamp au lieu d’un seul index composé. 

> [!NOTE]
> Vous ne pouvez pas créer d’index composés sur des tableaux ou des propriétés imbriqués.

La commande suivante crée un index composé sur les champs `name` et `age` :

`db.coll.createIndex({name:1,age:1})`

Vous pouvez utiliser des index composés pour effectuer un tri efficace sur plusieurs champs à la fois, comme illustré dans l’exemple suivant :

`db.coll.find().sort({name:1,age:1})`

L’index composé précédent peut également être utilisé pour un tri efficace sur une requête avec l’ordre de tri inverse sur tous les champs. Voici un exemple :

`db.coll.find().sort({name:-1,age:-1})`

Toutefois, la séquence des chemins dans l’index composé doit correspondre exactement à la requête. Voici un exemple de requête nécessitant un index composé supplémentaire :

`db.coll.find().sort({age:1,name:1})`

> [!NOTE]
> Les index composés sont utilisés uniquement dans les requêtes qui trient des résultats. Pour les requêtes qui ont plusieurs filtres qui n’ont pas besoin d’être triés, créez plusieurs index de champs uniques.

### <a name="multikey-indexes"></a>Index multiclés

Azure Cosmos DB crée des index multiclés pour indexer le contenu stocké dans des tableaux. Si vous indexez un champ avec une valeur de tableau, Azure Cosmos DB indexe automatiquement chaque élément du tableau.

### <a name="geospatial-indexes"></a>Index géospatiaux

De nombreux opérateurs géospatiaux tirent parti des index géospatiaux. Actuellement, l’API d’Azure Cosmos DB pour MongoDB prend en charge les index `2dsphere`. L’API ne prend pas encore en charge les index `2d`.

Voici un exemple de création d’index géospatial sur le champ `location` :

`db.coll.createIndex({ location : "2dsphere" })`

### <a name="text-indexes"></a>Index de texte

L’API d’Azure Cosmos DB pour MongoDB ne prend pas en charge les index de texte actuellement. Pour les requêtes de recherche de texte sur les chaînes, vous devez utiliser l’intégration de la [recherche cognitive Azure](../search/search-howto-index-cosmosdb.md) à Azure Cosmos DB. 

## <a name="wildcard-indexes"></a>Index génériques

Vous pouvez utiliser des index génériques pour prendre en charge des requêtes sur des champs inconnus. Supposons que vous disposiez d’une collection contenant des données sur des familles.

Voici un exemple de document dans cette collection :

```json
  "children": [
     {
         "firstName": "Henriette Thaulow",
         "grade": "5"
     }
  ]
```

Voici un autre exemple, cette fois avec un ensemble de propriétés légèrement différent dans `children` :

```json
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
      },
      {
        "familyName": "Merriam",
        "givenName": "John",
      }
  ]
```

Dans cette collection, les documents peuvent avoir de nombreuses propriétés possibles. Si vous souhaitez indexer toutes les données dans le tableau `children`, vous avez deux options : créer des index distincts pour chaque propriété individuelle ou créer un seul index générique pour l’ensemble du tableau `children`.

### <a name="create-a-wildcard-index"></a>Créer un index générique

La commande suivante crée un index générique sur toutes les propriétés dans `children` :

`db.coll.createIndex({"children.$**" : 1})`

**Contrairement à MongoDB, les index génériques peuvent prendre en charge plusieurs champs dans les prédicats de requête**. Il n’y aura pas de différence dans les performances de requête si vous utilisez un seul index générique au lieu de créer un index distinct pour chaque propriété.

Vous pouvez créer les types d’index suivants à l’aide de la syntaxe des caractères génériques :

- Champ unique
- Géospatial

### <a name="indexing-all-properties"></a>Indexation de toutes les propriétés

Voici comment vous pouvez créer un index de caractères génériques sur tous les champs :

`db.coll.createIndex( { "$**" : 1 } )`

> [!NOTE]
> Si vous commencez à développer, nous vous recommandons **fortement** de commencer avec un index générique sur tous les champs. Cela peut simplifier le développement et faciliter l’optimisation des requêtes.

Les documents contenant de nombreux champs peuvent avoir un coût d’unités de requête (RU) élevé pour les écritures et les mises à jour. Par conséquent, si vous avez une charge de travail avec d’importantes opérations d’écriture, vous devez opter pour des chemins d’index individuels au lieu d’utiliser des index génériques.

### <a name="limitations"></a>Limites

Les index génériques ne prennent pas en charge les types ou propriétés d’index suivants :

- Composé
- TTL
- Unique

**Contrairement à MongoDB**, dans l’API d’Azure Cosmos DB pour MongoDB, vous **ne pouvez pas** utiliser des index génériques pour les opérations suivantes :

- Création d’un index générique incluant plusieurs champs spécifiques

`db.coll.createIndex(
    { "$**" : 1 },
    { "wildcardProjection " :
        {
           "children.givenName" : 1,
           "children.grade" : 1
        }
    }
)`

- Création d’un index générique excluant plusieurs champs spécifiques

`db.coll.createIndex(
    { "$**" : 1 },
    { "wildcardProjection" :
        {
           "children.givenName" : 0,
           "children.grade" : 0
        }
    }
)`

Vous pouvez également créer plusieurs index génériques.

## <a name="index-properties"></a>Propriétés d’index

Les opérations suivantes sont communes pour les comptes desservant la version 3.6 du protocole Wire et les comptes desservant les versions antérieures. Vous pouvez obtenir plus d’informations sur les [index pris en charge et les propriétés indexées](mongodb-feature-support-36.md#indexes-and-index-properties).

### <a name="unique-indexes"></a>Index uniques

[Les index uniques](unique-keys.md) sont utiles pour s’assurer que deux documents ou plus ne contiennent pas la même valeur pour les champs indexés.

> [!IMPORTANT]
> Les index uniques peuvent être créés seulement quand la collection est vide (qu’elle ne contient aucun document).

La commande suivante crée un index unique sur le champ `student_id` :

```shell
globaldb:PRIMARY> db.coll.createIndex( { "student_id" : 1 }, {unique:true} )
{
        "_t" : "CreateIndexesResponse",
        "ok" : 1,
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 4
}
```

Pour les collections partitionnées, la création d’un index unique nécessite la clé de partition. En d’autres termes, tous les index uniques dans une collection partitionnée sont des index composés où l’un des champs est la clé de partition.

Les commandes suivantes créent une collection partitionnée ```coll``` (la clé de partition est ```university```) avec un index unique sur les champs `student_id` et `university` :

```shell
globaldb:PRIMARY> db.runCommand({shardCollection: db.coll._fullName, key: { university: "hashed"}});
{
        "_t" : "ShardCollectionResponse",
        "ok" : 1,
        "collectionsharded" : "test.coll"
}
globaldb:PRIMARY> db.coll.createIndex( { "university" : 1, "student_id" : 1 }, {unique:true});
{
        "_t" : "CreateIndexesResponse",
        "ok" : 1,
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 3,
        "numIndexesAfter" : 4
}
```

Dans l’exemple précédent, l’omission de la clause ```"university":1``` renvoie une erreur avec le message suivant :

```"cannot create unique index over {student_id : 1.0} with shard key pattern { university : 1.0 }"```

### <a name="ttl-indexes"></a>Index TTL

Pour activer l’expiration des documents dans une collection particulière, vous devez créer un [index de durée de vie (TTL, Time-to-Live)](../cosmos-db/time-to-live.md). Un index TTL est un index sur le champ `_ts` avec une valeur `expireAfterSeconds`.

Exemple :

```JavaScript
globaldb:PRIMARY> db.coll.createIndex({"_ts":1}, {expireAfterSeconds: 10})
```

La commande précédente supprime tous les documents de la collection ```db.coll``` qui n’ont pas été modifiés au cours des 10 dernières secondes.

> [!NOTE]
> Le champ **_ts** est spécifique à Azure Cosmos DB et n’est pas accessible à partir des clients MongoDB. Il s’agit d’une propriété (système) réservée qui contient le timestamp de la dernière modification du document.

## <a name="track-index-progress"></a>Suivre la progression de l’index

La version 3.6 de l’API d’Azure Cosmos DB pour MongoDB prend en charge la commande `currentOp()`, laquelle permet de suivre la progression de l’index sur une instance de base de données. Cette commande retourne un document qui contient des informations sur les opérations en cours sur une instance de base de données. Vous utilisez la commande `currentOp` pour effectuer le suivi de toutes les opérations en cours dans MongoDB en mode natif. Dans l’API d’Azure Cosmos DB pour MongoDB, cette commande ne prend en charge que le suivi de l’opération d’index.

Voici quelques exemples qui montrent comment utiliser la commande `currentOp` pour suivre la progression de l’index :

* Obtenir la progression de l’index pour une collection :

   ```shell
   db.currentOp({"command.createIndexes": <collectionName>, "command.$db": <databaseName>})
   ```

* Obtenir la progression de l’index pour toutes les collections d’une base de données :

  ```shell
  db.currentOp({"command.$db": <databaseName>})
  ```

* Obtenir la progression de l’index pour toutes les bases de données et collections d’un compte Azure Cosmos :

  ```shell
  db.currentOp({"command.createIndexes": { $exists : true } })
  ```

### <a name="examples-of-index-progress-output"></a>Exemples de sortie de la progression d’index

Les détails de la progression de l’index affichent le pourcentage de progression pour l’opération d’index active. Voici un exemple qui montre le format du document de sortie pour les différentes étapes de la progression de l’index :

- Pour une progression en pourcentage de l’opération d’index sur une collection « foo » et une base de données « bar » est de 60 %, le document de sortie suivant est généré. Le champ `Inprog[0].progress.total` affiche 100 comme pourcentage d’achèvement cible.

   ```json
   {
        "inprog" : [
        {
                ………………...
                "command" : {
                        "createIndexes" : foo
                        "indexes" :[ ],
                        "$db" : bar
                },
                "msg" : "Index Build (background) Index Build (background): 60 %",
                "progress" : {
                        "done" : 60,
                        "total" : 100
                },
                …………..…..
        }
        ],
        "ok" : 1
   }
   ```

- Si une opération d’index vient de démarrer sur une collection « foo » et une base de données « bar », le document de sortie peut indiquer une progression de 0 pour cent tant qu’un niveau mesurable n’est pas atteint.

   ```json
   {
        "inprog" : [
        {
                ………………...
                "command" : {
                        "createIndexes" : foo
                        "indexes" :[ ],
                        "$db" : bar
                },
                "msg" : "Index Build (background) Index Build (background): 0 %",
                "progress" : {
                        "done" : 0,
                        "total" : 100
                },
                …………..…..
        }
        ],
       "ok" : 1
   }
   ```

- Une fois l’opération d’index en cours terminée, le document de sortie indique des opérations `inprog` vides.

   ```json
   {
      "inprog" : [],
      "ok" : 1
   }
   ```

## <a name="background-index-updates"></a>Mises à jour d’index en arrière-plan

Quelle que soit la valeur spécifiée pour la propriété d’index **En arrière-plan**, les mises à jour d’index sont toujours effectuées en arrière-plan. Étant donné que les mises à jour d’index consomment des unités de requête (RU) à une priorité inférieure à celle des autres opérations de base de données, les modifications d’index n’entraînent aucun temps d’arrêt pour les écritures, les mises à jour ou les suppressions.

Il n’y a aucun impact sur la disponibilité de lecture lors de l’ajout d’un nouvel index. Les requêtes utilisent uniquement les nouveaux index une fois la transformation d’index terminée. Pendant la transformation d’index, le moteur de requête continue d’utiliser les index existants, ce qui vous permet d’observer des performances de lecture similaires pendant la transformation d’indexation à ce que vous aviez observé avant de lancer la modification de l’indexation. Lors de l’ajout de nouveaux index, il n’y a pas non plus de risque de résultats de requête incomplets ou incohérents.

Lorsque vous supprimez des index et que vous exécutez immédiatement des requêtes, il y a des filtres sur les index supprimés, les résultats peuvent être incohérents et incomplets jusqu’à la fin de la transformation d’index. Si vous supprimez des index, le moteur de requête ne fournit pas de résultats cohérents ou complets lorsque les requêtes appliquent des filtres sur ces index récemment supprimés. La plupart des développeurs ne suppriment pas les index, puis essaient immédiatement de les interroger. Or, en pratique, cette situation est peu probable.

> [!NOTE]
> Vous pouvez [suivre la progression de l’index](#track-index-progress).

## <a name="reindex-command"></a>Commande ReIndex

La commande `reIndex` recrée tous les index d’une collection. Dans la plupart des cas, c’est inutile. Toutefois, dans de rares cas, les performances des requêtes peuvent s’améliorer après l’exécution de la commande `reIndex`.

Vous pouvez exécuter la commande `reIndex` à l’aide de la syntaxe suivante :

`db.runCommand({ reIndex: <collection> })`

Vous pouvez utiliser la syntaxe ci-dessous pour vérifier si vous devez exécuter la commande `reIndex` :

`db.runCommand({"customAction":"GetCollection",collection:<collection>, showIndexes:true})`

Exemple de sortie :

```
{
        "database" : "myDB",
        "collection" : "myCollection",
        "provisionedThroughput" : 400,
        "indexes" : [
                {
                        "v" : 1,
                        "key" : {
                                "_id" : 1
                        },
                        "name" : "_id_",
                        "ns" : "myDB.myCollection",
                        "requiresReIndex" : true
                },
                {
                        "v" : 1,
                        "key" : {
                                "b.$**" : 1
                        },
                        "name" : "b.$**_1",
                        "ns" : "myDB.myCollection",
                        "requiresReIndex" : true
                }
        ],
        "ok" : 1
}
```

Si `reIndex` est nécessaire, **requiresReIndex** prend la valeur True. Si `reIndex` n’est pas nécessaire, cette propriété est omise.

## <a name="migrate-collections-with-indexes"></a>Migrer des collections avec des index

Actuellement, vous pouvez uniquement créer des index uniques lorsque la collection ne contient aucun document. Les outils de migration MongoDB populaires tentent de créer des index uniques après l’importation des données. Pour contourner ce problème, vous pouvez créer manuellement les collections et les index uniques correspondants au lieu d’autoriser l’outil de migration à essayer. (Vous pouvez obtenir ce comportement pour ```mongorestore``` à l’aide de l’indicateur `--noIndexRestore` dans la ligne de commande.)

## <a name="indexing-for-mongodb-version-32"></a>Indexation pour MongoDB version 3.2

Les fonctionnalités d’indexation disponibles et les valeurs par défaut sont différentes pour les comptes Azure Cosmos compatibles avec la version 3.2 du protocole filaire MongoDB. Vous pouvez [vérifier la version de votre compte](mongodb-feature-support-36.md#protocol-support) et [mettre à niveau vers la version 3.6](mongodb-version-upgrade.md).

Si vous utilisez la version 3.2, cette section décrit les principales différences avec la version 3.6.

### <a name="dropping-default-indexes-version-32"></a>Suppression des index par défaut (version 3.2)

Contrairement à la version 3.6 de l’API d’Azure Cosmos DB pour MongoDB, la version 3.2 indexe chaque propriété par défaut. Vous pouvez utiliser la commande suivante pour supprimer ces index par défaut pour une collection (```coll```) :

```JavaScript
> db.coll.dropIndexes()
{ "_t" : "DropIndexesResponse", "ok" : 1, "nIndexesWas" : 3 }
```

Après la suppression des index par défaut, vous pouvez ajouter d’autres index comme vous le feriez dans la version 3.6.

### <a name="compound-indexes-version-32"></a>Index composés (version 3.2)

Les index composés comportent des références à plusieurs champs d’un document. Si vous souhaitez créer un index composé, [mettez à niveau vers la version 3.6](mongodb-version-upgrade.md).

### <a name="wildcard-indexes-version-32"></a>Index génériques (version 3.2)

Si vous souhaitez créer un index générique, [mettez à niveau vers la version 3.6](mongodb-version-upgrade.md).

## <a name="next-steps"></a>Étapes suivantes

* [Indexation dans Azure Cosmos DB](../cosmos-db/index-policy.md)
* [Faire expirer automatiquement des données avec la durée de vie dans Azure Cosmos DB](../cosmos-db/time-to-live.md)
* Pour en savoir plus sur la relation entre le partitionnement et l’indexation, consultez le guide pratique pour [interroger un conteneur Azure Cosmos](how-to-query-container.md).
