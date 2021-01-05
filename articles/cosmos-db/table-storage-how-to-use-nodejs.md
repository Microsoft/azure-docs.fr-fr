---
title: Utiliser le Stockage Table Azure ou l’API Table Azure Cosmos DB avec Node.js
description: Stockez des données structurées dans le cloud à l’aide du Stockage Table Azure ou de l’API Table Azure Cosmos DB avec Node.js.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: nodejs
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.custom: devx-track-js
ms.openlocfilehash: 2d40b70d49b1934c9dd2d911369245b1b2e4f2ff
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93079697"
---
# <a name="how-to-use-azure-table-storage-or-the-azure-cosmos-db-table-api-from-nodejs"></a>Guide pratique pour utiliser le Stockage Table Azure ou l’API Table d’Azure Cosmos DB avec Node.js
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Cet article explique comment créer des tables, stocker vos données et effectuer des opérations CRUD sur les données. Choisissez le service de Table Azure ou l’API Table Azure Cosmos DB. Les exemples sont écrits en Node.js.

## <a name="create-an-azure-service-account"></a>Créer un compte de service Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Créer un compte de stockage Azure**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Créer un compte d’API Table Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="configure-your-application-to-access-azure-storage-or-the-azure-cosmos-db-table-api"></a>Configurer votre application pour accéder au Stockage Azure ou à l’API Table Azure Cosmos DB

Pour utiliser le Stockage Azure ou Azure Cosmos DB, vous avez besoin du Kit de développement logiciel (SDK) Stockage Azure pour Node.js, qui inclut un ensemble de bibliothèques pratiques communiquant avec les services de stockage REST.

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Utilisation de Node Package Manager (NPM) pour installer le package

1. Utilisez une interface de ligne de commande telle que **PowerShell** (Windows), **Terminal** (Mac) ou **Bash** (Unix) pour accéder au dossier dans lequel vous avez créé votre application.
2. Tapez **npm install azure-storage** dans la fenêtre de commande. Le résultat de la commande ressemble à l’exemple suivant.

   ```bash
    azure-storage@0.5.0 node_modules\azure-storage
    +-- extend@1.2.1
    +-- xmlbuilder@0.4.3
    +-- mime@1.2.11
    +-- node-uuid@1.4.3
    +-- validator@3.22.2
    +-- underscore@1.4.4
    +-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
    +-- xml2js@0.2.7 (sax@0.5.2)
    +-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
   ```

3. Vous pouvez exécuter manuellement la commande **ls** pour vérifier que le dossier **node_modules** a été créé. Dans ce dossier, recherchez le dossier **azure-storage** , qui contient les bibliothèques dont vous avez besoin pour accéder au stockage.

### <a name="import-the-package"></a>Importation du package

Ajoutez le code suivant en haut du fichier **server.js** dans votre application :

```javascript
var azure = require('azure-storage');
```

## <a name="add-your-connection-string"></a>Ajouter votre chaîne de connexion

Vous pouvez vous connecter au compte de stockage Azure ou au compte de l’API Table Azure Cosmos DB. Obtenez la chaîne de connexion en fonction du type de compte que vous utilisez.

### <a name="add-an-azure-storage-connection"></a>Ajout d’une connexion au stockage Azure

Le module Azure lit les variables d’environnement AZURE_STORAGE_ACCOUNT et AZURE_STORAGE_ACCESS_KEY, ou AZURE_STORAGE_CONNECTION_STRING pour obtenir les informations nécessaires à la connexion à votre compte de stockage Azure. Si ces variables d’environnement ne sont pas définies, vous devez spécifier les informations de compte lors de l’appel de `TableService`. Par exemple, le code suivant crée un objet `TableService` :

```javascript
var tableSvc = azure.createTableService('myaccount', 'myaccesskey');
```

### <a name="add-an-azure-cosmos-db-connection"></a>Ajouter une connexion à Azure Cosmos DB

Pour ajouter une connexion Azure Cosmos DB, créez un objet `TableService` et spécifiez le nom, la clé primaire et le point de terminaison de votre compte. Vous pouvez copier ces valeurs depuis **Paramètres** > **Chaîne de connexion** dans le portail Azure pour votre compte Cosmos DB. Par exemple :

```javascript
var tableSvc = azure.createTableService('myaccount', 'myprimarykey', 'myendpoint');
```

## <a name="create-a-table"></a>Créer une table

Le code suivant crée un objet `TableService` et l’utilise pour créer une table.

```javascript
var tableSvc = azure.createTableService();
```

L’appel de `createTableIfNotExists` crée une table du nom spécifié si elle n’existe pas. Dans l'exemple suivant, la table 'mytable' est créée, si elle n'existe pas déjà :

```javascript
tableSvc.createTableIfNotExists('mytable', function(error, result, response){
  if(!error){
    // Table exists or created
  }
});
```

Le `result.created` est `true` si une table est créée et `false` si la table existe déjà. La `response` contient des informations sur la requête.

### <a name="apply-filters"></a>Appliquer des filtres

Vous pouvez appliquer un filtrage facultatif aux opérations effectuées en utilisant `TableService`. Il peut s’agir d’opérations de journalisation, de relance automatique, etc. Les filtres sont des objets qui implémentent une méthode avec la signature :

```javascript
function handle (requestOptions, next)
```

Après le prétraitement des options de la requête, la méthode doit appeler **next** en passant un rappel avec la signature suivante :

```javascript
function (returnObject, finalCallback, next)
```

Dans ce rappel, et après le traitement de `returnObject` (la réponse à la requête du serveur), le rappel doit appeler `next`, s’il existe, pour continuer à traiter d’autres filtres, ou appeler `finalCallback` pour terminer l’appel du service.

Deux filtres, `ExponentialRetryPolicyFilter` et `LinearRetryPolicyFilter`, implémentant une logique de nouvelle tentative sont inclus avec le kit de développement logiciel (SDK) Azure pour Node.js. Le code suivant permet de créer un objet `TableService` utilisant `ExponentialRetryPolicyFilter` :

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var tableSvc = azure.createTableService().withFilter(retryOperations);
```

## <a name="add-an-entity-to-a-table"></a>Ajout d'une entité à une table

Pour ajouter une entité, commencez par créer un objet qui définit les propriétés de l'entité. Toutes les entités doivent contenir une propriété **PartitionKey** et **RowKey** , qui sont des identificateurs uniques de l’entité.

* **PartitionKey**  : détermine la partition dans laquelle l’entité est stockée.
* **RowKey**  : identifie de façon unique l’entité dans la partition.

**PartitionKey** et **RowKey** doivent être des valeurs de chaîne. Pour plus d'informations, consultez la rubrique [Présentation du modèle de données du service de Table](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model).

Voici un exemple de définition d'une entité. La **dueDate** est définie en tant que type de `Edm.DateTime`. L'indication du type est facultative et s'ils ne sont pas spécifiés, les types sont inférés.

```javascript
var task = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'},
  description: {'_':'take out the trash'},
  dueDate: {'_':new Date(2015, 6, 20), '$':'Edm.DateTime'}
};
```

> [!NOTE]
> Il existe également un champ `Timestamp` pour chaque enregistrement, qu’Azure définit lors de l’insertion ou la mise à jour d’une entité.

Vous pouvez également utiliser le `entityGenerator` pour créer des entités. L’exemple suivant crée la même entité de tâche à l’aide du `entityGenerator`.

```javascript
var entGen = azure.TableUtilities.entityGenerator;
var task = {
  PartitionKey: entGen.String('hometasks'),
  RowKey: entGen.String('1'),
  description: entGen.String('take out the trash'),
  dueDate: entGen.DateTime(new Date(Date.UTC(2015, 6, 20))),
};
```

Pour ajouter une entité à votre table, transmettez l’objet entité à la méthode `insertEntity`.

```javascript
tableSvc.insertEntity('mytable',task, function (error, result, response) {
  if(!error){
    // Entity inserted
  }
});
```

Si l’opération aboutit, `result` contient l’élément [ETag](https://en.wikipedia.org/wiki/HTTP_ETag) de l’enregistrement inséré et `response` contient des informations sur l’opération.

Exemple de réponse :

```javascript
{ '.metadata': { etag: 'W/"datetime\'2015-02-25T01%3A22%3A22.5Z\'"' } }
```

> [!NOTE]
> Par défaut, la méthode `insertEntity` ne retourne pas l’entité insérée dans le cadre des informations de `response`. Si vous prévoyez d’exécuter d’autres opérations sur cette entité, ou si vous voulez mettre en cache les informations, il peut être utile de la faire renvoyer dans le cadre de `result`. Pour ce faire, vous pouvez activer `echoContent` comme suit :
>
> `tableSvc.insertEntity('mytable', task, {echoContent: true}, function (error, result, response) {...}`

## <a name="update-an-entity"></a>Mise à jour d'une entité

Plusieurs méthodes permettent de mettre à jour une entité existante :

* `replaceEntity` : met à jour une entité existante en la remplaçant.
* `mergeEntity` : met à jour une entité existante en fusionnant les nouvelles valeurs de propriétés avec l’entité existante.
* `insertOrReplaceEntity` : met à jour une entité existante en la remplaçant. En l'absence d'entité, une nouvelle entité est insérée.
* `insertOrMergeEntity` : met à jour une entité existante en fusionnant les nouvelles valeurs de propriétés dans l’entité existante. En l'absence d'entité, une nouvelle entité est insérée.

L’exemple suivant illustre la mise à jour d’une entité en utilisant `replaceEntity` :

```javascript
tableSvc.replaceEntity('mytable', updatedTask, function(error, result, response){
  if(!error) {
    // Entity updated
  }
});
```

> [!NOTE]
> Par défaut, la mise à jour d'une entité ne vérifie pas si les données en cours de mise à jour ont déjà été modifiées par un autre processus. Pour activer la prise en charge de mises à jour simultanées :
>
> 1. Obtenez l'ETag de l'objet mis à jour. Il est renvoyé dans la `response` d’une opération sur une entité et peut être extrait dans `response['.metadata'].etag`.
> 2. Lors d'une opération de mise à jour sur une entité, ajoutez les informations ETag précédemment extraites dans la nouvelle entité. Par exemple :
>
>       entity2['.metadata'].etag = currentEtag;
> 3. Effectuez l'opération de mise à jour. Si l’entité a été modifiée depuis que vous avez récupéré la valeur ETag, par exemple une autre instance de votre application, une `error` est renvoyée, indiquant que la condition de mise à jour spécifiée dans la requête n’est pas remplie.
>
>

Avec `replaceEntity` et `mergeEntity`, si l’entité à mettre à jour n’existe pas, l’opération de mise à jour échoue. Par conséquent, si vous souhaitez stocker une entité indépendamment de son existence ou non, utilisez `insertOrReplaceEntity` ou `insertOrMergeEntity`.

Le `result` des opérations de mise à jour réussies contient l’ **Etag** de l’entité mise à jour.

## <a name="work-with-groups-of-entities"></a>Utilisation des groupes d'entités

Il est parfois intéressant de soumettre un lot d'opérations simultanément pour assurer un traitement atomique par le serveur. Pour ce faire, utilisez la classe `TableBatch` pour créer un lot, puis la méthode `executeBatch` de `TableService` pour effectuer les opérations par lot.

 L'exemple suivant illustre la soumission par lot de deux entités :

```javascript
var task1 = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'},
  description: {'_':'Take out the trash'},
  dueDate: {'_':new Date(2015, 6, 20)}
};
var task2 = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '2'},
  description: {'_':'Wash the dishes'},
  dueDate: {'_':new Date(2015, 6, 20)}
};

var batch = new azure.TableBatch();

batch.insertEntity(task1, {echoContent: true});
batch.insertEntity(task2, {echoContent: true});

tableSvc.executeBatch('mytable', batch, function (error, result, response) {
  if(!error) {
    // Batch completed
  }
});
```

Pour les opérations de traitement par lot réussies, `result` contient les informations de chaque opération du lot.

### <a name="work-with-batched-operations"></a>Ultiliser des opérations de traitement par lot

Les opérations ajoutées à un traitement par lot peuvent être inspectées en affichant la propriété `operations`. Vous pouvez également utiliser les méthodes suivantes avec les opérations :

* **clear**  : permet de supprimer toutes les opérations d’un lot.
* **getOperations**  : permet d’obtenir une opération du lot.
* **hasOperations**  : permet de renvoyer true si le lot contient des opérations.
* **removeOperations**  : permet de supprimer une opération.
* **size**  : permet de renvoyer le nombre d’opérations du lot.

## <a name="retrieve-an-entity-by-key"></a>Récupération d'une entité par clé

Pour envoyer une entité spécifique d’après la valeur **PartitionKey** et **RowKey** , utilisez la méthode **retrieveEntity**.

```javascript
tableSvc.retrieveEntity('mytable', 'hometasks', '1', function(error, result, response){
  if(!error){
    // result contains the entity
  }
});
```

Après cette opération, `result` contient l’entité.

## <a name="query-a-set-of-entities"></a>Interrogation d’un ensemble d’entités

Pour interroger une table, utilisez l’objet **TableQuery** pour générer une expression de requête en utilisant les clauses suivantes :

* **select**  : les champs à renvoyer par la requête.
* **where**  : la clause where.

  * **and**  : une condition where `and`.
  * **or**  : une condition where `or`.
* **top**  : le nombre d’éléments à extraire.

L’exemple suivant crée une requête qui renvoie les cinq premiers éléments avec une PartitionKey « hometasks ».

```javascript
var query = new azure.TableQuery()
  .top(5)
  .where('PartitionKey eq ?', 'hometasks');
```

Comme **select** n'est pas utilisé, tous les champs sont renvoyés. Pour exécuter la requête dans une table, utilisez **queryEntities**. L'exemple suivant utilise cette requête pour renvoyer des entités de « mytable ».

```javascript
tableSvc.queryEntities('mytable',query, null, function(error, result, response) {
  if(!error) {
    // query was successful
  }
});
```

En cas de réussite, `result.entries` contient un tableau d’entités qui correspondent à la requête. Si la requête n’a pas pu renvoyer toutes les entités, `result.continuationToken` est non *null* et peut servir de troisième paramètre de **queryEntities** pour obtenir davantage de résultats. Pour la requête initiale, utilisez *null* comme troisième paramètre.

### <a name="query-a-subset-of-entity-properties"></a>Interrogation d'un sous-ensemble de propriétés d'entité

Vous pouvez utiliser une requête de table pour extraire uniquement quelques champs d'une entité.
Ceci permet de réduire la consommation de bande passante et peut améliorer les performances des requêtes, notamment pour les entités volumineuses. Utilisez la clause **select** et transmettez les noms des champs à renvoyer. Par exemple, la requête suivante renvoie uniquement les champs **description** et **dueDate**.

```javascript
var query = new azure.TableQuery()
  .select(['description', 'dueDate'])
  .top(5)
  .where('PartitionKey eq ?', 'hometasks');
```

## <a name="delete-an-entity"></a>Suppression d’une entité

Vous pouvez supprimer une entité en utilisant ses clés de partition et de ligne. Dans cet exemple, l’objet **task1** contient les valeurs **RowKey** et **PartitionKey** de l’entité à supprimer. L'objet est transmis à la méthode **deleteEntity** .

```javascript
var task = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'}
};

tableSvc.deleteEntity('mytable', task, function(error, response){
  if(!error) {
    // Entity deleted
  }
});
```

> [!NOTE]
> Vous avez intérêt à utiliser les ETag pour supprimer des éléments afin de vous assurer que les éléments n'ont pas été modifiés par un autre processus. Consultez [Mise à jour d’une entité](#update-an-entity) pour plus d’informations sur l’utilisation des ETags.
>
>

## <a name="delete-a-table"></a>Suppression d’une table

Le code suivant permet de supprimer une table d'un compte de stockage.

```javascript
tableSvc.deleteTable('mytable', function(error, response){
    if(!error){
        // Table deleted
    }
});
```

Si vous ne savez pas si la table existe, utilisez **deleteTableIfExists**.

## <a name="use-continuation-tokens"></a>Utiliser des jetons de liaison

Si vous interrogez des tables et que les résultats peuvent être volumineux, recherchez des jetons de liaison. Sans que vous en ayez vraiment conscience, de grandes quantités de données peuvent être disponibles pour votre requête si elle n’est pas en mesure de détecter la présence d’un jeton de liaison.

L’objet **résultats** renvoyé après l’interrogation des entités définit une propriété`continuationToken` si ce jeton est présent. Vous pouvez ensuite utiliser cette propriété pour exécuter une requête sur l’ensemble des entités de table et de partition.

Pendant l’interrogation, un paramètre `continuationToken` peut être fourni entre l’instance d’objet de requête et la fonction de rappel :

```javascript
var nextContinuationToken = null;
dc.table.queryEntities(tableName,
    query,
    nextContinuationToken,
    function (error, results) {
        if (error) throw error;

        // iterate through results.entries with results

        if (results.continuationToken) {
            nextContinuationToken = results.continuationToken;
        }

    });
```

L’objet `continuationToken` contient des propriétés telles que `nextPartitionKey`, `nextRowKey` et `targetLocation` que vous pouvez utiliser pour effectuer une itération dans tous les résultats.

Vous pouvez également utiliser `top` avec `continuationToken` pour définir la taille de la page.

## <a name="work-with-shared-access-signatures"></a>Utilisation des signatures d'accès partagé

Les signatures d’accès partagé (SAP) sont un moyen sécurisé de fournir un accès précis aux tables sans fournir le nom ni les clés de votre compte de stockage. Elles servent souvent à fournir un accès limité à vos données, par exemple pour autoriser une application mobile à interroger des enregistrements.

Une application approuvée, comme un service cloud, génère une SAP à l’aide de l’élément **generateSharedAccessSignature** du **TableService** , et la fournit à une application non approuvée ou semi-approuvée, comme une application mobile. La signature d'accès partagé est générée à l'aide d'une stratégie, qui décrit les dates de début et de fin de validité de la signature, et le niveau d'accès accordé au détenteur de la signature.

L'exemple suivant génère une nouvelle stratégie d'accès partagé qui autorise le détenteur de la signature d'accès partagé à interroger (« r ») la table et expire 100 minutes après son heure de création.

```javascript
var startDate = new Date();
var expiryDate = new Date(startDate);
expiryDate.setMinutes(startDate.getMinutes() + 100);
startDate.setMinutes(startDate.getMinutes() - 100);

var sharedAccessPolicy = {
  AccessPolicy: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
    Start: startDate,
    Expiry: expiryDate
  },
};

var tableSAS = tableSvc.generateSharedAccessSignature('mytable', sharedAccessPolicy);
var host = tableSvc.host;
```

Notez que vous devez également fournir les informations sur l'hôte, car elles sont obligatoires lorsque le détenteur de la signature d'accès partagé tente d'accéder à la table.

L'application cliente utilise les signatures d'accès partagé avec **TableServiceWithSAS** pour effectuer les opérations sur la table. L'exemple suivant se connecte à la table et exécute une requête. Consultez l’article [Accorder un accès limité aux ressources du Stockage Azure à l’aide de la signature d’accès partagé (SAS)](../storage/common/storage-sas-overview.md) pour connaître le format de la signature d’accès partagé (SAS) à une table.

```javascript
// Note in the following command, host is in the format: `https://<your_storage_account_name>.table.core.windows.net` and the tableSAS is in the format: `sv=2018-03-28&si=saspolicy&tn=mytable&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D`;

var sharedTableService = azure.createTableServiceWithSas(host, tableSAS);
var query = azure.TableQuery()
  .where('PartitionKey eq ?', 'hometasks');

sharedTableService.queryEntities(query, null, function(error, result, response) {
  if(!error) {
    // result contains the entities
  }
});
```

Comme la signature d'accès partagé a été générée seulement avec un accès en requête, une erreur est renvoyée si vous tentez d'insérer, de mettre à jour ou de supprimer des entités.

### <a name="access-control-lists"></a>Listes de contrôle d’accès

Vous pouvez également utiliser une liste de contrôle d'accès (ACL) pour définir la stratégie d'accès pour une signature d'accès partagé. Cela est utile si vous voulez autoriser plusieurs clients à accéder à la table tout en fournissant des stratégies d'accès différentes à chaque client.

Une liste de contrôle d'accès est implémentée à l'aide d'un tableau de stratégies d'accès, dans lequel un ID est associé à chaque stratégie. L’exemple suivant définit deux stratégies ; une pour « user1 » et une pour « user2 » :

```javascript
var sharedAccessPolicy = {
  user1: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
    Start: startDate,
    Expiry: expiryDate
  },
  user2: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};
```

L’exemple suivant obtient la liste de contrôle d’accès actuelle pour la table **hometasks** , puis ajoute les nouvelles stratégies à l’aide de **setTableAcl**. Cette approche permet :

```javascript
var extend = require('extend');
tableSvc.getTableAcl('hometasks', function(error, result, response) {
if(!error){
    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
    tableSvc.setTableAcl('hometasks', newSignedIdentifiers, function(error, result, response){
      if(!error){
        // ACL set
      }
    });
  }
});
```

Après avoir défini la liste de contrôle d'accès, vous pouvez créer une signature d'accès partagé basée sur l'ID pour une stratégie. L'exemple suivant crée une signature d'accès partagé pour « user2 » :

```javascript
tableSAS = tableSvc.generateSharedAccessSignature('hometasks', { Id: 'user2' });
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez les ressources ci-dessous.

* [Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.
* Référentiel du [Kit de développement logiciel (SDK) Azure Storage pour Node.js](https://github.com/Azure/azure-storage-node) sur GitHub.
* [Azure pour développeurs Node.js](/azure/developer/javascript/)
* [Créer une application web Node.js dans Azure](../app-service/quickstart-nodejs.md)
* [Création et déploiement d’une application Node.js dans un service cloud Azure](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (avec Windows PowerShell)