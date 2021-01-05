---
title: Commandes d’extension MongoDB pour gérer les données dans les API d’Azure Cosmos DB pour MongoDB
description: Cet article décrit comment utiliser les commandes d’extension MongoDB pour gérer les données stockées dans les API d’Azure Cosmos DB pour MongoDB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 05/28/2020
ms.author: chrande
ms.custom: devx-track-js
ms.openlocfilehash: 2fd2fa7620e57c58f72dad73c1012a19190e8fbc
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359644"
---
# <a name="use-mongodb-extension-commands-to-manage-data-stored-in-azure-cosmos-dbs-api-for-mongodb"></a>Utiliser les commandes d’extension MongoDB pour gérer les données stockées dans les API d’Azure Cosmos DB pour MongoDB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Le document suivant contient les commandes d’action personnalisées qui sont spécifiques à l’API d’Azure Cosmos DB pour MongoDB. Ces commandes peuvent être utilisées pour créer et obtenir des ressources de base de données spécifiques au [modèle de capacité Azure Cosmos DB](account-databases-containers-items.md).

En utilisant l’API d’Azure Cosmos DB pour MongoDB, vous pouvez profiter des nombreux avantages de Cosmos DB, comme la distribution mondiale, le partitionnement automatique, la haute disponibilité, les garanties de latence, le chiffrement automatique au repos, les sauvegardes, etc. tout en préservant vos investissements dans votre application MongoDB. Vous pouvez communiquer avec l'API Azure Cosmos DB pour MongoDB par le biais de n'importe quel [pilote du client MongoDB](https://docs.mongodb.org/ecosystem/drivers) open source. L’API d’Azure Cosmos DB pour MongoDB permet d’utiliser les pilotes clients existants en adhérant au [protocole Wire MongoDB](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol).

## <a name="mongodb-protocol-support"></a>Prise en charge des protocoles MongoDB

L’API d’ Azure Cosmos DB pour MongoDB est compatible avec la version 3.2 et 3.6 du serveur MongoDB. Voir les [fonctionnalités et la syntaxe prises en charge](mongodb-feature-support.md) pour plus de détails. 

Les commandes d’extension suivantes permettent de créer et de modifier des ressources spécifiques à Azure Cosmos DB par le biais de requêtes de base de données :

* [Créer une base de données](#create-database)
* [Mettre à jour la base de données](#update-database)
* [Obtenir une base de données](#get-database)
* [Créer une collection](#create-collection)
* [Mettre à jour la collection](#update-collection)
* [Obtenir une collection](#get-collection)

## <a name="create-database"></a><a id="create-database"></a> Créer une base de données

La commande d’extension de création de base de données crée une base de données MongoDB. Le nom de la base de données peut être utilisé à partir du contexte de base de données défini par la commande `use database`. Le tableau suivant décrit les paramètres inclus dans la commande :

|**Champ**|**Type** |**Description** |
|---------|---------|---------|
| `customAction`   |  `string`  |   Nom de la commande personnalisée, qui doit être "CreateDatabase".      |
| `offerThroughput` | `int`  | Débit approvisionné que vous définissez sur la base de données. Ce paramètre est facultatif. |
| `autoScaleSettings` | `Object` | Requis pour le [mode de mise à l’échelle automatique](provision-throughput-autoscale.md). Cet objet contient les paramètres associés au mode de capacité de mise à l’échelle automatique. Vous pouvez configurer la valeur `maxThroughput`, qui décrit le nombre d’unités de requête le plus élevé auquel la collection sera augmentée de manière dynamique. |

### <a name="output"></a>Output

Si la commande réussit, la réponse suivante est retournée :

```javascript
{ "ok" : 1 }
```

Consultez la [sortie par défaut](#default-output) de la commande personnalisée pour connaître les paramètres de la sortie.

### <a name="examples"></a>Exemples

#### <a name="create-a-database"></a>Création d'une base de données

Pour créer une base de données nommée `"test"` qui utilise toutes les valeurs par défaut, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "CreateDatabase"});
```

Cette commande permet de créer une base de données sans débit au niveau de la base de données. Cela signifie que les collections de cette base de données devront spécifier la quantité de débit que vous devez utiliser.

#### <a name="create-a-database-with-throughput"></a>Créer une base de données avec débit

Pour créer une base de données nommée `"test"` et spécifier un débit approvisionné [au niveau de la base de données](set-throughput.md#set-throughput-on-a-database) de 1 000 unités de requête, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "CreateDatabase", offerThroughput: 1000 });
```

Cela permet de créer une base de données et de lui définir un débit. Toutes les collections de cette base de données partageront le débit défini, à moins que les collections ne soient créées avec [un niveau de débit spécifique](set-throughput.md#set-throughput-on-a-database-and-a-container).

#### <a name="create-a-database-with-autoscale-throughput"></a>Créer une base de données avec un débit avec mise à l’échelle automatique

Pour créer une base de données nommée `"test"` et spécifier un débit maximal avec mise à l’échelle automatique de 20 000 RU/s [au niveau de la base de données](set-throughput.md#set-throughput-on-a-database), utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "CreateDatabase", autoScaleSettings: { maxThroughput: 20000 } });
```

## <a name="update-database"></a><a id="update-database"></a> Mettre à jour la base de données

La commande d’extension de mise à jour de base de données met à jour les propriétés associées à la base de données spécifiée. Le tableau suivant décrit les paramètres inclus dans la commande :

|**Champ**|**Type** |**Description** |
|---------|---------|---------|
| `customAction`    |    `string`     |   Nom de la commande personnalisée. Il doit être "UpdateDatabase".      |
|  `offerThroughput`   |  `int`       |     Nouveau débit approvisionné que vous souhaitez définir sur la base de données si la base de données utilise [un débit au niveau de la base de données](set-throughput.md#set-throughput-on-a-database).  |
| `autoScaleSettings` | `Object` | Requis pour le [mode de mise à l’échelle automatique](provision-throughput-autoscale.md). Cet objet contient les paramètres associés au mode de capacité de mise à l’échelle automatique. Vous pouvez configurer la valeur `maxThroughput`, qui décrit le nombre d’unités de requête le plus élevé auquel la base de données sera augmentée de manière dynamique. |

Cette commande utilise la base de données spécifiée dans le contexte de la session. Il s’agit de la base de données que vous avez utilisée dans la commande `use <database>`. Pour le moment, le nom de la base de données ne peut pas être modifié à l’aide de cette commande.

### <a name="output"></a>Output

Si la commande réussit, la réponse suivante est retournée :

```javascript
{ "ok" : 1 }
```

Consultez la [sortie par défaut](#default-output) de la commande personnalisée pour connaître les paramètres de la sortie.

### <a name="examples"></a>Exemples

#### <a name="update-the-provisioned-throughput-associated-with-a-database"></a>Mettre à jour le débit approvisionné associé à une base de données

Pour mettre à jour le débit approvisionné d’une base de données nommée `"test"` à 1 200 unités de requête, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "UpdateDatabase", offerThroughput: 1200 });
```

#### <a name="update-the-autoscale-throughput-associated-with-a-database"></a>Mettre à jour le débit avec mise à l’échelle automatique associé à une base de données

Pour mettre à jour le débit approvisionné d’une base de données nommée `"test"` à 20 000 unités de requête ou pour le transformer en un [niveau de débit avec mise à l’échelle automatique](provision-throughput-autoscale.md), utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "UpdateDatabase", autoScaleSettings: { maxThroughput: 20000 } });
```


## <a name="get-database"></a><a id="get-database"></a> Obtenir une base de données

La commande d’extension d’obtention de base de données retourne l’objet de base de données. Le nom de la base de données est tiré du contexte de base de données par rapport auquel la commande est exécutée.

```javascript
{
  customAction: "GetDatabase"
}
```

Le tableau suivant décrit les paramètres inclus dans la commande :


|**Champ**|**Type** |**Description** |
|---------|---------|---------|
|  `customAction`   |   `string`      |   Nom de la commande personnalisée. Il doit être "GetDatabase".|
        
### <a name="output"></a>Output

Si la commande réussit, la réponse contient un document avec les champs suivants :

|**Champ**|**Type** |**Description** |
|---------|---------|---------|
|  `ok`   |   `int`     |   État de la réponse. 1 == réussite. 0 == échec.      |
| `database`    |    `string`        |   Nom de la base de données.      |
|   `provisionedThroughput`  |    `int`      |    Débit approvisionné qui est défini sur la base de données si la base de données utilise [un débit manuel au niveau de la base de données](set-throughput.md#set-throughput-on-a-database).     |
| `autoScaleSettings` | `Object` | Cet objet contient les paramètres de capacité associés à la base de données si elle utilise le [mode de mise à l’échelle automatique](provision-throughput-autoscale.md). La valeur `maxThroughput` décrit le nombre d’unités de requête le plus élevé auquel la base de données sera augmentée de manière dynamique. |

Si la commande échoue, une réponse de commande personnalisée par défaut est retournée. Consultez la [sortie par défaut](#default-output) de la commande personnalisée pour connaître les paramètres de la sortie.

### <a name="examples"></a>Exemples

#### <a name="get-the-database"></a>Obtenir la base de données

Pour obtenir l’objet de base de données pour une base de données nommée `"test"`, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "GetDatabase"});
```

Si la base de données n’a pas de débit associé, la sortie est la suivante :

```javascript
{ "database" : "test", "ok" : 1 }
```

Si la base de données a un [débit manuel au niveau de la base de données](set-throughput.md#set-throughput-on-a-database) qui lui est associé, la sortie affiche les valeurs `provisionedThroughput` suivantes :

```javascript
{ "database" : "test", "provisionedThroughput" : 20000, "ok" : 1 }
```

Si la base de données a un [débit avec mise à l’échelle automatique au niveau de la base de données](provision-throughput-autoscale.md) qui lui est associé, la sortie indique le `provisionedThroughput`, qui décrit le nombre minimal de RU/s pour la base de données, et l’objet `autoScaleSettings`, y compris le `maxThroughput`, qui décrit le nombre maximal de RU/s pour la base de données.

```javascript
{
        "database" : "test",
        "provisionedThroughput" : 2000,
        "autoScaleSettings" : {
                "maxThroughput" : 20000
        },
        "ok" : 1
}
```

## <a name="create-collection"></a><a id="create-collection"></a> Créer une collection

La commande d’extension de création de collection crée une collection MongoDB. Le nom de la base de données est utilisé à partir du contexte de base de données défini par la commande `use database`. Le format de la commande CreateCollection est le suivant :

```javascript
{
  customAction: "CreateCollection",
  collection: "<Collection Name>",
  shardKey: "<Shard key path>",
  offerThroughput: (int), // Amount of throughput allocated to a specific collection

}
```

Le tableau suivant décrit les paramètres inclus dans la commande :

| **Champ** | **Type** | **Obligatoire** | **Description** |
|---------|---------|---------|---------|
| `customAction` | `string` | Obligatoire | Nom de la commande personnalisée. Doit être « CreateCollection ».|
| `collection` | `string` | Obligatoire | Nom de la collection. Aucun caractère spécial ni aucune espace ne sont autorisés.|
| `offerThroughput` | `int` | Facultatif | Débit approvisionné à définir sur la base de données. Si ce paramètre n'est pas fourni, la valeur par défaut est la valeur minimum, 400 RU/s. * Pour spécifier un débit supérieur à 10 000 RU/s, le paramètre `shardKey` est obligatoire.|
| `shardKey` | `string` | Obligatoire pour les collections avec un débit élevé | Chemin d'accès à la clé de partition pour la collection partitionnée. Ce paramètre est obligatoire si vous définissez plus de 10 000 RU/s dans `offerThroughput`.  S’il est spécifié, tous les documents insérés auront besoin de cette clé et de cette valeur. |
| `autoScaleSettings` | `Object` | Obligatoire pour le [mode de mise à l’échelle automatique](provision-throughput-autoscale.md) | Cet objet contient les paramètres associés au mode de capacité de mise à l’échelle automatique. Vous pouvez configurer la valeur `maxThroughput`, qui décrit le nombre d’unités de requête le plus élevé auquel la collection sera augmentée de manière dynamique. |

### <a name="output"></a>Output

Retourne une réponse de commande personnalisée par défaut. Consultez la [sortie par défaut](#default-output) de la commande personnalisée pour connaître les paramètres de la sortie.

### <a name="examples"></a>Exemples

#### <a name="create-a-collection-with-the-minimum-configuration"></a>Créer une collection avec la configuration minimale

Pour créer une collection avec le nom `"testCollection"` et les valeurs par défaut, utilisez la commande suivante : 

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection"});
```

Cela génère une nouvelle collection fixe et non partitionnée, avec 400 RU/s et un index sur le champ `_id` créé automatiquement. Ce type de configuration s’applique également lors de la création de collections à l’aide de la fonction `insert()`. Par exemple : 

```javascript
use test
db.newCollection.insert({});
```

#### <a name="create-a-unsharded-collection"></a>Créer une collection non partitionnée

Pour créer une collection non partitionnée nommée `"testCollection"` avec un débit approvisionné de 1 000 unités de requête, utilisez la commande suivante : 

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", offerThroughput: 1000});
``` 

Vous pouvez créer une collection avec un maximum de 10 000 RU/s en tant que `offerThroughput` sans avoir besoin de spécifier une clé de partition. Pour les collections avec un débit plus élevé, consultez la section suivante.

#### <a name="create-a-sharded-collection"></a>Créer une collection partitionnée

Pour créer une collection partitionnée nommée `"testCollection"` avec un débit approvisionné de 11 000 unités de requête et une propriété `shardkey` « a.b », utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", offerThroughput: 11000, shardKey: "a.b" });
```

Cette commande requiert désormais le paramètre `shardKey`, car plus de 10 000 RU/s sont spécifiées dans le `offerThroughput`.

#### <a name="create-an-unsharded-autoscale-collection"></a>Créer une collection avec mise à l’échelle automatique non partitionnée

Pour créer une collection non partitionnée nommée `'testCollection'` qui utilise la [capacité de débit avec mise à l’échelle automatique](provision-throughput-autoscale.md) définie sur 4 000 RU/s, utilisez la commande suivante :

```javascript
use test
db.runCommand({ 
    customAction: "CreateCollection", collection: "testCollection", 
    autoScaleSettings:{
      maxThroughput: 4000
    } 
});
```

Pour la valeur `autoScaleSettings.maxThroughput`, vous pouvez spécifier une plage comprise entre 4 000 RU/s et 10 000 RU/s sans clé partition. Pour un débit avec mise à l’échelle automatique plus élevé, vous devez spécifier le paramètre `shardKey`.

#### <a name="create-a-sharded-autoscale-collection"></a>Créer une collection avec mise à l’échelle automatique partitionnée

Pour créer une collection partitionnée nommée `'testCollection'` avec une clé de partition appelée `'a.b'` et qui utilise la [capacité de débit avec mise à l’échelle automatique](provision-throughput-autoscale.md) définie sur 20 000 RU/s, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", shardKey: "a.b", autoScaleSettings: { maxThroughput: 20000 }});
```

## <a name="update-collection"></a><a id="update-collection"></a> Mettre à jour la collection

La commande d’extension de mise à jour de collection met à jour les propriétés associées à la collection spécifiée.

```javascript
{
  customAction: "UpdateCollection",
  collection: "<Name of the collection that you want to update>",
  offerThroughput: (int) // New throughput that will be set to the collection
}
```

Le tableau suivant décrit les paramètres inclus dans la commande :

|**Champ**|**Type** |**Description** |
|---------|---------|---------|
|  `customAction`   |   `string`      |   Nom de la commande personnalisée. Il doit être "UpdateCollection".      |
|  `collection`   |   `string`      |   Nom de la collection.       |
| `offerThroughput` | `int` |   Débit approvisionné à définir sur la collection.|
| `autoScaleSettings` | `Object` | Requis pour le [mode de mise à l’échelle automatique](provision-throughput-autoscale.md). Cet objet contient les paramètres associés au mode de capacité de mise à l’échelle automatique. La valeur `maxThroughput` décrit le nombre d’unités de requête le plus élevé auquel la collection sera augmentée de manière dynamique. |

## <a name="output"></a>Output

Retourne une réponse de commande personnalisée par défaut. Consultez la [sortie par défaut](#default-output) de la commande personnalisée pour connaître les paramètres de la sortie.

### <a name="examples"></a>Exemples

#### <a name="update-the-provisioned-throughput-associated-with-a-collection"></a>Mettre à jour le débit approvisionné associé à une collection

Pour mettre à jour le débit approvisionné d’une collection nommée `"testCollection"` à 1 200 unités de requête, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "UpdateCollection", collection: "testCollection", offerThroughput: 1200 });
```

## <a name="get-collection"></a><a id="get-collection"></a> Obtenir une collection

La commande personnalisée d’obtention de collection retourne l’objet de collection.

```javascript
{
  customAction: "GetCollection",
  collection: "<Name of the collection>"
}
```

Le tableau suivant décrit les paramètres inclus dans la commande :


|**Champ**|**Type** |**Description** |
|---------|---------|---------|
| `customAction`    |   `string`      |   Nom de la commande personnalisée. Il doit être "GetCollection".      |
| `collection`    |    `string`     |    Nom de la collection.     |

### <a name="output"></a>Output

Si la commande réussit, la réponse contient un document avec les champs suivants.


|**Champ**|**Type** |**Description** |
|---------|---------|---------|
|  `ok`   |    `int`     |   État de la réponse. 1 == réussite. 0 == échec.      |
| `database`    |    `string`     |   Nom de la base de données.      |
| `collection`    |    `string`     |    Nom de la collection.     |
|  `shardKeyDefinition`   |   `document`      |  Document de spécification d’index utilisé comme clé de partition. Ce paramètre de réponse est facultatif.       |
|  `provisionedThroughput`   |   `int`      |    Débit approvisionné à définir sur la collection. Ce paramètre de réponse est facultatif.     |
| `autoScaleSettings` | `Object` | Cet objet contient les paramètres de capacité associés à la base de données si elle utilise le [mode de mise à l’échelle automatique](provision-throughput-autoscale.md). La valeur `maxThroughput` décrit le nombre d’unités de requête le plus élevé auquel la collection sera augmentée de manière dynamique. |

Si la commande échoue, une réponse de commande personnalisée par défaut est retournée. Consultez la [sortie par défaut](#default-output) de la commande personnalisée pour connaître les paramètres de la sortie.

### <a name="examples"></a>Exemples

#### <a name="get-the-collection"></a>Obtenir la collection

Pour obtenir l’objet de collection pour une collection nommée `"testCollection"`, utilisez la commande suivante :

```javascript
use test
db.runCommand({customAction: "GetCollection", collection: "testCollection"});
```

Si la collection est associée à une capacité de débit, elle inclut la valeur `provisionedThroughput` et la sortie est la suivante :

```javascript
{
        "database" : "test",
        "collection" : "testCollection",
        "provisionedThroughput" : 400,
        "ok" : 1
}
```

Si la collection est associée à un débit avec mise à l’échelle automatique, elle inclut l’objet `autoScaleSettings` avec le paramètre `maxThroughput`, qui définit le débit maximal auquel la collection augmentera de façon dynamique. En outre, elle inclura également la valeur `provisionedThroughput`, qui définit le débit minimal auquel cette collection sera réduite en l’absence de requête dans la collection : 

```javascript
{
        "database" : "test",
        "collection" : "testCollection",
        "provisionedThroughput" : 1000,
        "autoScaleSettings" : {
            "maxThroughput" : 10000
        },
        "ok" : 1
}
```

Si la collection partage un [débit au niveau de la base de données](set-throughput.md#set-throughput-on-a-database), soit en mode de mise à l’échelle automatique, soit en manuel, la sortie est la suivante :

```javascript
{ "database" : "test", "collection" : "testCollection", "ok" : 1 }
```

```javascript
{
        "database" : "test",
        "provisionedThroughput" : 2000,
        "autoScaleSettings" : {
            "maxThroughput" : 20000
        },
        "ok" : 1
}
```


## <a name="default-output-of-a-custom-command"></a><a id="default-output"></a> Sortie par défaut d’une commande personnalisée

Si elle n’est pas spécifiée, la réponse personnalisée contient un document avec les champs suivants :

|**Champ**|**Type** |**Description** |
|---------|---------|---------|
|  `ok`   |    `int`     |   État de la réponse. 1 == réussite. 0 == échec.      |
| `code`    |   `int`      |   Retourné uniquement lorsque la commande a échoué (ok == 0). Contient le code d’erreur MongoDB. Ce paramètre de réponse est facultatif.      |
|  `errMsg`   |  `string`      |    Retourné uniquement lorsque la commande a échoué (ok == 0). Contient un message d’erreur convivial. Ce paramètre de réponse est facultatif.      |

Par exemple :

```javascript
{ "ok" : 1 }
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez ensuite commencer à apprendre les concepts relatifs à Azure Cosmos DB suivants : 

* [Indexation dans Azure Cosmos DB](../cosmos-db/index-policy.md)
* [Faire expirer automatiquement des données avec la durée de vie dans Azure Cosmos DB](../cosmos-db/time-to-live.md)
