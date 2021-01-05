---
title: Fonctionnalités et syntaxe prises en charge de l’API Azure Cosmos DB pour MongoDB (version 3.2)
description: Découvrez les fonctionnalités et la syntaxe prises en charge de l’API Azure Cosmos DB pour MongoDB (version 3.2).
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 10/16/2019
author: sivethe
ms.author: sivethe
ms.openlocfilehash: 60e806be85a36c2e8a64d731c8794d806a4fcae4
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93096524"
---
# <a name="azure-cosmos-dbs-api-for-mongodb-32-version-supported-features-and-syntax"></a>API Azure Cosmos DB pour MongoDB (version 3.2) : fonctionnalités et syntaxe prises en charge
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. Vous pouvez communiquer avec l’API Azure Cosmos DB pour MongoDB par le biais de n’importe quel [pilote](https://docs.mongodb.org/ecosystem/drivers) du client open source MongoDB. L’API Azure Cosmos DB pour MongoDB permet d’utiliser les pilotes clients existants en adhérant au [protocole Wire](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol) MongoDB.

À l’aide de l’API Azure Cosmos DB pour MongoDB, vous pouvez profiter des avantages de MongoDB que vous connaissez déjà, ainsi que de toutes les fonctionnalités d’entreprise fournies par Cosmos DB : [distribution globale](distribute-data-globally.md), [partitionnement automatique](partitioning-overview.md), garanties de disponibilité et latence, indexation automatique de tous les champs, chiffrement au repos, sauvegardes et bien plus encore.

> [!NOTE]
> Cet article concerne l’API Azure Cosmos DB pour MongoDB 3.2. Pour la version MongoDB 3.6, consultez [Fonctionnalités et syntaxe prises en charge de MongoDB 3.6](mongodb-feature-support-36.md).

## <a name="protocol-support"></a>Prise en charge du protocole

Tous les nouveaux comptes de l’API Azure Cosmos DB pour MongoDB sont compatibles avec la version **3.6** du serveur MongoDB. Cet article porte sur MongoDB version 3.2. Les opérateurs pris en charge, ainsi que les limitations ou exceptions sont répertoriés ci-dessous. Les pilotes clients comprenant ces protocoles doivent pouvoir se connecter à l’API Azure Cosmos DB pour MongoDB. 

L’API Azure Cosmos DB pour MongoDB offre également une expérience de mise à niveau transparente pour les comptes éligibles. Pour en savoir plus, consultez le [Guide de mise à niveau de la version MongoDB](mongodb-version-upgrade.md).

## <a name="query-language-support"></a>Prise en charge du langage de requêtes

L’API Azure Cosmos DB pour MongoDB permet la prise en charge complète des constructions de langage de requête MongoDB. Vous trouverez ci-dessous la liste détaillée des opérations prises en charge actuellement, les opérateurs, les étapes, les commandes et les options.

## <a name="database-commands"></a>Commandes de base de données

L’API Azure Cosmos DB pour MongoDB prend en charge les commandes de base de données suivantes :

### <a name="query-and-write-operation-commands"></a>Commandes d’opérations de requête et d’écriture

- supprimer
- trouver
- findAndModify
- getLastError
- getMore
- insert
- update

### <a name="authentication-commands"></a>Commandes d’authentification

- logout
- authentifier
- getnonce

### <a name="administration-commands"></a>Commandes d’administration

- dropDatabase
- listCollections
- drop
- create
- filemd5
- createIndexes
- listIndexes
- dropIndexes
- ConnectionStatus
- reIndex

### <a name="diagnostics-commands"></a>Commandes de diagnostic

- buildInfo
- collStats
- dbStats
- hostInfo
- listDatabases
- whatsmyuri

<a name="aggregation-pipeline"></a>

## <a name="aggregation-pipelinea"></a>Pipeline d’agrégation</a>

Cosmos DB prend en charge le pipeline d’agrégation pour MongoDB 3.2 en préversion publique. Consultez le [blog Azure](https://azure.microsoft.com/blog/azure-cosmosdb-extends-support-for-mongodb-aggregation-pipeline-unique-indexes-and-more/) pour obtenir des instructions sur l’obtention de la préversion publique.

### <a name="aggregation-commands"></a>Commandes d’agrégation

- aggregate
- count
- distinct

### <a name="aggregation-stages"></a>Étapes d’agrégation

- $project
- $match
- $limit
- $skip
- $unwind
- $group
- $sample
- $sort
- $lookup
- $out
- $count
- $addFields

### <a name="aggregation-expressions"></a>Expressions d’agrégation

#### <a name="boolean-expressions"></a>Expressions booléennes

- $and
- $or
- $not

#### <a name="set-expressions"></a>Expressions SET

- $setEquals
- $setIntersection
- $setUnion
- $setDifference
- $setIsSubset
- $anyElementTrue
- $allElementsTrue

#### <a name="comparison-expressions"></a>Expressions de comparaison

- $cmp
- $eq
- $gt
- $gte
- $lt
- $lte
- $ne

#### <a name="arithmetic-expressions"></a>Expressions arithmétiques

- $abs
- $add
- $ceil
- $divide
- $exp
- $floor
- $ln
- $log
- $log10
- $mod
- $multiply
- $pow
- $sqrt
- $subtract
- $trunc

#### <a name="string-expressions"></a>Expressions de chaîne

- $concat
- $indexOfBytes
- $indexOfCP
- $split
- $strLenBytes
- $strLenCP
- $strcasecmp
- $substr
- $substrBytes
- $substrCP
- $toLower
- $toUpper

#### <a name="array-expressions"></a>Expressions de tableau

- $arrayElemAt
- $concatArrays
- $filter
- $indexOfArray
- $isArray
- $range
- $reverseArray
- $size
- $slice
- $in

#### <a name="date-expressions"></a>Expressions de date

- $dayOfYear
- $dayOfMonth
- $dayOfWeek
- $year
- $month
- $week
- $hour
- $minute
- $second
- $millisecond
- $isoDayOfWeek
- $isoWeek

#### <a name="conditional-expressions"></a>Expressions conditionnelles

- $cond
- $ifNull

## <a name="aggregation-accumulators"></a>Accumulateurs d’agrégation

- $sum
- $avg
- $first
- $last
- $max
- $min
- $push
- $addToSet

## <a name="operators"></a>Opérateurs

Les opérateurs suivants sont pris en charge, voici des exemples correspondant à leur utilisation. L’exemple de document suivant est utilisé pour les requêtes ci-dessous :

```json
{
  "Volcano Name": "Rainier",
  "Country": "United States",
  "Region": "US-Washington",
  "Location": {
    "type": "Point",
    "coordinates": [
      -121.758,
      46.87
    ]
  },
  "Elevation": 4392,
  "Type": "Stratovolcano",
  "Status": "Dendrochronology",
  "Last Known Eruption": "Last known eruption from 1800-1899, inclusive"
}
```

Opérateur | Exemple |
--- | --- |
$eq | `{ "Volcano Name": { $eq: "Rainier" } }` |  | -
$gt | `{ "Elevation": { $gt: 4000 } }` |  | -
$gte | `{ "Elevation": { $gte: 4392 } }` |  | -
$lt | `{ "Elevation": { $lt: 5000 } }` |  | -
$lte | `{ "Elevation": { $lte: 5000 } }` | | -
$ne | `{ "Elevation": { $ne: 1 } }` |  | -
$in | `{ "Volcano Name": { $in: ["St. Helens", "Rainier", "Glacier Peak"] } }` |  | -
$nin | `{ "Volcano Name": { $nin: ["Lassen Peak", "Hood", "Baker"] } }` | | -
$or | `{ $or: [ { Elevation: { $lt: 4000 } }, { "Volcano Name": "Rainier" } ] }` |  | -
$and | `{ $and: [ { Elevation: { $gt: 4000 } }, { "Volcano Name": "Rainier" } ] }` |  | -
$not | `{ "Elevation": { $not: { $gt: 5000 } } }`|  | -
$nor | `{ $nor: [ { "Elevation": { $lt: 4000 } }, { "Volcano Name": "Baker" } ] }` |  | -
$exists | `{ "Status": { $exists: true } }`|  | -
$type | `{ "Status": { $type: "string" } }`|  | -
$mod | `{ "Elevation": { $mod: [ 4, 0 ] } }` |  | -
$regex | `{ "Volcano Name": { $regex: "^Rain"} }`|  | -

### <a name="notes"></a>Notes

Dans les requêtes $regex, les expressions ancrées à gauche autorisent la recherche d’index. Toutefois, l’utilisation des modificateurs « i » (non sensible à la casse) et « m » (multiligne), provoquent l’analyse de la collection pour toutes les expressions.
Lorsqu’il est nécessaire d’inclure « $ » ou « | », il est préférable de créer deux requêtes regex (ou plus).
Par exemple, étant donné la requête d’origine suivante : ```find({x:{$regex: /^abc$/})```, elle doit être modifiée comme suit : ```find({x:{$regex: /^abc/, x:{$regex:/^abc$/}})```.
La première partie utilise l’index pour limiter la recherche aux documents commençant par ^abc et la deuxième partie correspond aux entrées exactes.
L’opérateur à barre « | » agit comme une fonction « OR », la requête ```find({x:{$regex: /^abc|^def/})``` correspond aux documents dont le champ « x » comporte une valeur commençant par « abc » ou « def ». Pour utiliser l’index, il est recommandé de diviser la requête en deux requêtes différentes jointes par l’opérateur $or : ```find( {$or : [{x: $regex: /^abc/}, {$regex: /^def/}] })```.

### <a name="update-operators"></a>Opérateurs de mise à jour

#### <a name="field-update-operators"></a>Opérateurs de mise à jour de champ

- $inc
- $mul
- $rename
- $setOnInsert
- $set
- $unset
- $min
- $max
- $currentDate

#### <a name="array-update-operators"></a>Opérateurs de mise à jour de tableau

- $addToSet
- $pop
- $pullAll
- $pull (Remarque : l’opérateur $pull avec une condition n’est pas pris en charge)
- $pushAll
- $push
- $each
- $slice
- $sort
- $position

#### <a name="bitwise-update-operator"></a>Opérateur de mise à jour au niveau du bit

- $bit

### <a name="geospatial-operators"></a>Opérateurs géospatiaux

Opérateur | Exemple | Prise en charge |
--- | --- | --- |
$geoWithin | ```{ "Location.coordinates": { $geoWithin: { $centerSphere: [ [ -121, 46 ], 5 ] } } }``` | Oui |
$geoIntersects |  ```{ "Location.coordinates": { $geoIntersects: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Oui |
$near | ```{ "Location.coordinates": { $near: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Oui |
$nearSphere | ```{ "Location.coordinates": { $nearSphere : [ -121, 46  ], $maxDistance: 0.50 } }``` | Oui |
$geometry | ```{ "Location.coordinates": { $geoWithin: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Oui |
$minDistance | ```{ "Location.coordinates": { $nearSphere : { $geometry: {type: "Point", coordinates: [ -121, 46 ]}, $minDistance: 1000, $maxDistance: 1000000 } } }``` | Oui |
$maxDistance | ```{ "Location.coordinates": { $nearSphere : [ -121, 46  ], $maxDistance: 0.50 } }``` | Oui |
$center | ```{ "Location.coordinates": { $geoWithin: { $center: [ [-121, 46], 1 ] } } }``` | Oui |
$centerSphere | ```{ "Location.coordinates": { $geoWithin: { $centerSphere: [ [ -121, 46 ], 5 ] } } }``` | Oui |
$box | ```{ "Location.coordinates": { $geoWithin: { $box:  [ [ 0, 0 ], [ -122, 47 ] ] } } }``` | Oui |
$polygon | ```{ "Location.coordinates": { $near: { $geometry: { type: "Polygon", coordinates: [ [ [ -121.9, 46.7 ], [ -121.5, 46.7 ], [ -121.5, 46.9 ], [ -121.9, 46.9 ], [ -121.9, 46.7 ] ] ] } } } }``` | Oui |

## <a name="sort-operations"></a>Trier les opérations

Lorsque vous utilisez l’opération `findOneAndUpdate`, les opérations de tri sur un champ unique sont prises en charge, mais les opérations à effectuer sur plusieurs champs ne le sont pas.

## <a name="additional-operators"></a>Opérateurs supplémentaires

Opérateur | Exemple | Notes
--- | --- | --- |
$all | ```{ "Location.coordinates": { $all: [-121.758, 46.87] } }``` |
$elemMatch | ```{ "Location.coordinates": { $elemMatch: {  $lt: 0 } } }``` |
$size | ```{ "Location.coordinates": { $size: 2 } }``` |
$comment |  ```{ "Location.coordinates": { $elemMatch: {  $lt: 0 } }, $comment: "Negative values"}``` |
$text |  | Non pris en charge. Utilisez plutôt $regex.

## <a name="unsupported-operators"></a>Opérateurs non pris en charge

Les opérateurs ```$where``` et ```$eval``` ne sont pas pris en charge par Azure Cosmos DB.

### <a name="methods"></a>Méthodes

Les méthodes suivantes sont prises en charge :

#### <a name="cursor-methods"></a>Méthodes Cursor

Méthode | Exemple | Notes
--- | --- | --- |
Cursor.Sort() | ```cursor.sort({ "Elevation": -1 })``` | Les documents sans clé de tri ne sont pas retournées

## <a name="unique-indexes"></a>Index uniques

Cosmos DB indexe tous les champs dans les documents qui sont écrits dans la base de données par défaut. Les index uniques garantissent qu’un champ spécifique ne présente pas de valeurs en double dans tous les documents d’une collection. Cette approche est semblable à la façon dont l’unicité est conservée sur la clé `_id` par défaut. Vous pouvez créer des index personnalisés dans Cosmos DB à l’aide de la commande createIndex, y compris la contrainte « unique ».

Les index uniques sont disponibles pour tous les comptes Cosmos à l’aide de l’API Azure Cosmos DB pour MongoDB.

## <a name="time-to-live-ttl"></a>Durée de vie (TTL)

Cosmos DB prend en charge une durée de vie (TTL) en fonction du timestamp du document. La TTL peut être activée pour les collections via le [Portail Azure](https://portal.azure.com).

## <a name="user-and-role-management"></a>Gestion des rôles et des utilisateurs

Cosmos DB ne prend pas encore en charge les utilisateurs et les rôles. Cosmos DB prend cependant en charge le contrôle d'accès en fonction du rôle Azure (Azure RBAC) et les mots de passe/clés en lecture-écriture et en lecture seule, qui peuvent être obtenus par le biais du [portail Azure](https://portal.azure.com) (page de la chaîne de connexion).

## <a name="replication"></a>Réplication

Cosmos DB prend en charge la réplication automatique et native des couches inférieures. Cette logique est prolongée pour obtenir également la réplication globale et à faible latence. Cosmos DB ne prend pas en charge les commandes de réplication manuelle.

## <a name="write-concern"></a>Élément Write Concern

Certaines applications utilisent un élément [Write Concern](https://docs.mongodb.com/manual/reference/write-concern/), qui indique le nombre de réponses nécessaires au cours d’une opération d’écriture. En raison de la façon dont Cosmos DB gère la réplication en arrière-plan, toutes les écritures atteignent automatiquement le quorum par défaut. Tout élément Write Concern spécifié par le code client est ignoré. Pour en savoir plus, consultez [Niveaux de cohérence des données analysables dans Azure Cosmos DB](consistency-levels.md).

## <a name="sharding"></a>Partitionnement

Azure Cosmos DB prend en charge le partitionnement automatique côté serveur. Il gère automatiquement la création, le positionnement et l’équilibrage de partitions. Azure Cosmos DB ne prend pas en charge les commandes de partitionnement manuelles, ce qui signifie que vous n’avez pas besoin d’appeler de commandes, comme shardCollection, addShard, balancerStart, moveChunk, etc. Vous devez uniquement spécifier la clé de partition au moment de créer les conteneurs ou d’interroger les données.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser Studio 3T](mongodb-mongochef.md) avec l’API Azure Cosmos DB pour MongoDB.
- Découvrez comment [utiliser Robo 3T](mongodb-robomongo.md) avec l’API Azure Cosmos DB pour MongoDB.
- Explorez les [exemples](mongodb-samples.md) MongoDB avec l’API Azure Cosmos DB pour MongoDB.

<sup>Remarque : Cet article décrit une fonctionnalité d’Azure Cosmos DB qui assure la compatibilité des protocoles de transmission avec les bases de données MongoDB. Microsoft n’exécute pas les bases de données MongoDB pour fournir ce service. Azure Cosmos DB n’est pas affilié à MongoDB, Inc.</sup>
