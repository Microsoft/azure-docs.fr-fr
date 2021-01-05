---
title: Utiliser l’API du service de Table de Stockage Azure ou de l’API Table d’Azure Cosmos DB à partir de code PHP
description: Stockez des données structurées dans le cloud à l’aide du Stockage Table Azure ou de l’API Table Azure Cosmos DB avec PHP.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: php
ms.topic: sample
ms.date: 07/23/2020
ms.openlocfilehash: 9d059c899e4a64d4d2c1b880b2a1d0f89258f33b
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93079629"
---
# <a name="how-to-use-azure-storage-table-service-or-the-azure-cosmos-db-table-api-from-php"></a>Utilisation de l’API du service de Table de Stockage Azure ou de l’API Azure Cosmos DB Table à partir de code PHP
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Cet article vous montre comment créer des tables, stocker vos données et effectuer des opérations CRUD sur les données. Choisissez le service de Table Azure ou l’API Table Azure Cosmos DB. Les exemples sont écrits en PHP et utilisent la [bibliothèque de client PHP pour le service de Table de Stockage Azure][download]. Les tâches couvertes incluent **la création et la suppression d’une table** , ainsi que **l’insertion, la suppression et l’interrogation d’entités dans une table**. Pour plus d'informations sur le service de Table Azure, consultez la section [Étapes suivantes](#next-steps) .

## <a name="create-an-azure-service-account"></a>Créer un compte de service Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Créer un compte de stockage Azure**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Créer un compte d’API Table Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-php-application"></a>Création d'une application PHP

La création d’une application PHP pour accéder à l’API du service de Table de Stockage Azure ou à l’API Table d’Azure Cosmos DB présente une seule exigence : référencer des classes du Kit de développement logiciel (SDK) de Table de Stockage Azure pour PHP à partir de votre code. Vous pouvez utiliser tous les outils de développement pour créer votre application, y compris Bloc-notes.

Dans ce guide, vous allez utiliser les fonctionnalités du service de Table de Stockage Azure ou d’Azure Cosmos DB qui peuvent être appelées dans une application PHP en local, ou dans le code s’exécutant dans un rôle web, un rôle de travail ou un site web Azure.

## <a name="get-the-client-library"></a>Obtenir la bibliothèque de client

1. Créez un fichier nommé composer.json à la racine de votre projet et ajoutez-y le code suivant :
   ```json
   {
   "require": {
    "microsoft/azure-storage-table": "*"
   }
   }
   ```
2. Téléchargez [composer.phar](https://getcomposer.org/composer.phar) à la racine. 
3. Ouvrez une invite de commandes et exécutez la commande suivante à la racine du projet :
   ```
   php composer.phar install
   ```
   Vous pouvez également accéder à la [bibliothèque de client PHP du service de Table de Stockage Azure](https://github.com/Azure/azure-storage-php/tree/master/azure-storage-table) sur GitHub pour cloner le code source.

## <a name="add-required-references"></a>Ajouter les références requises

Pour utiliser les API du service de Table de Stockage Azure ou d’Azure Cosmos DB, vous devez :

* référencer le fichier de chargeur automatique à l’aide de l’instruction [require_once][require_once] ; et
* référencer toutes les classes utilisées.

L’exemple suivant montre comment inclure le fichier du chargeur automatique et référencer la classe **TableRestProxy**.

```php
require_once 'vendor/autoload.php';
use MicrosoftAzure\Storage\Table\TableRestProxy;
```

Dans les exemples ci-dessous, l'instruction `require_once` s'affiche toujours, mais seules les classes nécessaires aux besoins de l'exemple à exécuter sont référencées.

## <a name="add-your-connection-string"></a>Ajouter votre chaîne de connexion

Vous pouvez vous connecter au compte de stockage Azure ou au compte de l’API Table Azure Cosmos DB. Obtient la chaîne de connexion en fonction du type de compte que vous utilisez.

### <a name="add-a-storage-table-service-connection"></a>Ajouter une connexion au service de Table de Stockage Azure

Pour instancier un client de service de Table de Stockage Azure, vous devez disposer au préalable d’une chaîne de connexion valide. Le format de la chaîne de connexion au service de Table de Stockage Azure est le suivant :

```php
$connectionString = "DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]"
```

### <a name="add-a-storage-emulator-connection"></a>Ajouter une connexion à l’émulateur de stockage

Pour accéder au stockage de l’émulateur :

```php
UseDevelopmentStorage = true
```

### <a name="add-an-azure-cosmos-db-connection"></a>Ajouter une connexion à Azure Cosmos DB

Pour instancier un client de Table Azure Cosmos DB, vous devez disposer au préalable d’une chaîne de connexion valide. Le format de la chaîne de connexion à Azure Cosmos DB est le suivant :

```php
$connectionString = "DefaultEndpointsProtocol=[https];AccountName=[myaccount];AccountKey=[myaccountkey];TableEndpoint=[https://myendpoint/]";
```

Pour créer un client de service de Table de Stockage Azure ou un client Azure Cosmos DB, vous devez utiliser la classe **TableRestProxy**. Vous pouvez :

* lui transmettre directement la chaîne de connexion ; ou
* utiliser **CloudConfigurationManager (CCM)** pour vérifier plusieurs sources externes pour la chaîne de connexion :
  * Par défaut, il prend en charge une source externe : les variables d’environnement.
  * Vous pouvez ajouter de nouvelles sources en étendant la classe `ConnectionStringSource`.

Dans les exemples ci-dessous, la chaîne de connexion est passée directement.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;

$tableClient = TableRestProxy::createTableService($connectionString);
```

## <a name="create-a-table"></a>Créer une table

Vous pouvez créer une table avec un objet **TableRestProxy** via la méthode **createTable**. Au moment de créer une table, vous pouvez définir le délai d'expiration du service de Table. (Pour plus d’informations sur le délai d’expiration du service de Table, consultez [Définition de délais d’expiration pour les opérations du service de Table][table-service-timeouts].)

```php
require_once 'vendor\autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create Table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    // Create table.
    $tableClient->createTable("mytable");
}
catch(ServiceException $e){
    $code = $e->getCode();
    $error_message = $e->getMessage();
    // Handle exception based on error codes and messages.
    // Error codes and messages can be found here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
}
```

Pour plus d’informations sur les restrictions au niveau des noms de table, consultez [Présentation du modèle de données du service de Table][table-data-model].

## <a name="add-an-entity-to-a-table"></a>Ajout d'une entité à une table

Pour ajouter une entité à une table, créez un objet **Entity** et transmettez-le à **TableRestProxy->insertEntity**. Notez que lorsque vous créez une entité, vous devez spécifier une clé `PartitionKey` et `RowKey`. Il s’agit des identificateurs uniques d’une entité, dont les valeurs peuvent être interrogées bien plus rapidement que les autres propriétés d’entité. Le système utilise `PartitionKey` pour distribuer automatiquement les entités de la table sur plusieurs nœuds de stockage. Les entités partageant la même clé `PartitionKey` sont stockées sur le même nœud. (Les opérations réalisées sur plusieurs entités offrent de meilleures performances lorsque ces entités sont stockées sur un même nœud plutôt que sur différents nœuds.) La clé `RowKey` est l’ID unique d’une entité au sein d’une partition.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$entity = new Entity();
$entity->setPartitionKey("tasksSeattle");
$entity->setRowKey("1");
$entity->addProperty("Description", null, "Take out the trash.");
$entity->addProperty("DueDate",
                        EdmType::DATETIME,
                        new DateTime("2012-11-05T08:15:00-08:00"));
$entity->addProperty("Location", EdmType::STRING, "Home");

try{
    $tableClient->insertEntity("mytable", $entity);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
}
```

Pour plus d’informations sur les propriétés et les types de table, consultez [Présentation du modèle de données du service de Table][table-data-model].

La classe **TableRestProxy** offre deux autres méthodes pour insérer des entités : **insertOrMergeEntity** et **insertOrReplaceEntity**. Pour utiliser ces méthodes, créez un objet **Entity** et transmettez-le en tant que paramètre à l'une ou l'autre des méthodes. Chaque méthode insère l'entité si elle n'existe pas. Si l’entité existe déjà, **insertOrMergeEntity** met à jour la valeur des propriétés si celles-ci existent déjà et en ajoute de nouvelles dans le cas contraire, alors que **insertOrReplaceEntity** remplace entièrement une entité existante. L'exemple suivant montre comment utiliser **insertOrMergeEntity**. Si l’entité associée à la clé `PartitionKey` « tasksSeattle » et à la clé `RowKey` « 1 » n’existe pas déjà, elle est insérée. En revanche, si elle a été ajoutée précédemment (comme indiqué dans l’exemple ci-dessus), la propriété `DueDate` est mise à jour et la propriété `Status` est ajoutée. Les propriétés `Description` et `Location` sont également mises à jour, mais avec des valeurs qui de fait les laissent inchangées. Si ces deux dernières propriétés n'ont pas été ajoutées comme indiqué dans l'exemple, mais qu'elles existaient sur l'entité cible, leurs valeurs existantes restent inchangées.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

//Create new entity.
$entity = new Entity();

// PartitionKey and RowKey are required.
$entity->setPartitionKey("tasksSeattle");
$entity->setRowKey("1");

// If entity exists, existing properties are updated with new values and
// new properties are added. Missing properties are unchanged.
$entity->addProperty("Description", null, "Take out the trash.");
$entity->addProperty("DueDate", EdmType::DATETIME, new DateTime()); // Modified the DueDate field.
$entity->addProperty("Location", EdmType::STRING, "Home");
$entity->addProperty("Status", EdmType::STRING, "Complete"); // Added Status field.

try    {
    // Calling insertOrReplaceEntity, instead of insertOrMergeEntity as shown,
    // would simply replace the entity with PartitionKey "tasksSeattle" and RowKey "1".
    $tableClient->insertOrMergeEntity("mytable", $entity);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="retrieve-a-single-entity"></a>Extraction d'une seule entité

La méthode **TableRestProxy->getEntity** vous permet de récupérer une seule entité via une requête portant sur ses clés `PartitionKey` et `RowKey`. Dans l’exemple ci-dessous, la clé de partition `tasksSeattle` et la clé de ligne `1` sont transmises à la méthode **getEntity**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    $result = $tableClient->getEntity("mytable", "tasksSeattle", 1);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$entity = $result->getEntity();

echo $entity->getPartitionKey().":".$entity->getRowKey();
```

## <a name="retrieve-all-entities-in-a-partition"></a>Extraction de toutes les entités d'une partition

Les requêtes d’entité sont construites à l’aide de filtres (pour plus d’informations, consultez [Interrogation de tables et d’entités][filters]). Pour extraire toutes les entités d’une partition, utilisez le filtre « PartitionKey eq *nom_partition* ». L’exemple suivant montre comment récupérer toutes les entités de la partition `tasksSeattle` en passant un filtre à la méthode **queryEntities** .

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$filter = "PartitionKey eq 'tasksSeattle'";

try    {
    $result = $tableClient->queryEntities("mytable", $filter);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$entities = $result->getEntities();

foreach($entities as $entity){
    echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
}
```

## <a name="retrieve-a-subset-of-entities-in-a-partition"></a>Extraction d'un sous-ensemble d'entités dans une partition

Pour extraire un sous-ensemble d'entités dans une partition, il est possible d'utiliser le modèle de l'exemple précédent. Le sous-ensemble d’entités extrait varie en fonction du filtre utilisé (pour plus d’informations, consultez [Interrogation de tables et d’entités][filters]). L’exemple suivant montre comment utiliser un filtre pour extraire toutes les entités avec une valeur `Location` spécifique et une valeur `DueDate` antérieure à une date spécifiée.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$filter = "Location eq 'Office' and DueDate lt '2012-11-5'";

try    {
    $result = $tableClient->queryEntities("mytable", $filter);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$entities = $result->getEntities();

foreach($entities as $entity){
    echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
}
```

## <a name="retrieve-a-subset-of-entity-properties"></a>Extraction d'un sous-ensemble de propriétés d'entité

Une requête peut extraire un sous-ensemble de propriétés d'entité. Cette technique, nommée *projection* , réduit la consommation de bande passante et peut améliorer les performances des requêtes, notamment pour les entités volumineuses. Pour spécifier une propriété à récupérer, transmettez son nom à la méthode **Query->addSelectField**. Vous pouvez appeler cette méthode plusieurs fois pour ajouter des propriétés supplémentaires. Après avoir exécuté **TableRestProxy->queryEntities** , les entités renvoyées contiennent uniquement les propriétés sélectionnées. (si vous voulez renvoyer un sous-ensemble d'entités de table, utilisez un filtre comme indiqué dans les requêtes précédentes).

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\QueryEntitiesOptions;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$options = new QueryEntitiesOptions();
$options->addSelectField("Description");

try    {
    $result = $tableClient->queryEntities("mytable", $options);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

// All entities in the table are returned, regardless of whether
// they have the Description field.
// To limit the results returned, use a filter.
$entities = $result->getEntities();

foreach($entities as $entity){
    $description = $entity->getProperty("Description")->getValue();
    echo $description."<br />";
}
```

## <a name="update-an-entity"></a>Mise à jour d'une entité

Vous pouvez mettre à jour une entité existante en lui appliquant les méthodes **Entity->setProperty** et **Entity->addProperty** , puis en appelant **TableRestProxy->updateEntity**. Dans l'exemple suivant, une entité est extraite, une propriété modifiée, une autre propriété supprimée et une nouvelle propriété ajoutée. Notez que vous pouvez supprimer une propriété en lui attribuant la valeur **null**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$result = $tableClient->getEntity("mytable", "tasksSeattle", 1);

$entity = $result->getEntity();
$entity->setPropertyValue("DueDate", new DateTime()); //Modified DueDate.
$entity->setPropertyValue("Location", null); //Removed Location.
$entity->addProperty("Status", EdmType::STRING, "In progress"); //Added Status.

try    {
    $tableClient->updateEntity("mytable", $entity);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="delete-an-entity"></a>Suppression d’une entité

Pour supprimer une entité, passez le nom de la table ainsi que les clés `PartitionKey` et `RowKey` à la méthode **TableRestProxy->deleteEntity**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    // Delete entity.
    $tableClient->deleteEntity("mytable", "tasksSeattle", "2");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Pour les contrôles d’accès concurrentiel, vous pouvez définir la suppression de la propriété Etag d’une entité en employant la méthode **DeleteEntityOptions->setEtag** et en transmettant l’objet **DeleteEntityOptions** à **deleteEntity** en tant que quatrième paramètre.

## <a name="batch-table-operations"></a>Traitement par lots d'opérations de table

La méthode **TableRestProxy->batch** permet d’exécuter plusieurs opérations dans une même demande. Ce modèle implique d’ajouter des opérations à l’objet **BatchRequest** et de transmettre **ce** dernier à la méthode **TableRestProxy->batch**. Pour ajouter une opération à un objet **BatchRequest** , vous pouvez appeler l'une des méthodes suivantes à plusieurs reprises :

* **addInsertEntity** (permet d'ajouter une opération insertEntity)
* **addUpdateEntity** (permet d'ajouter une opération updateEntity)
* **addMergeEntity** (permet d'ajouter une opération mergeEntity)
* **addInsertOrReplaceEntity** (permet d'ajouter une opération insertOrReplaceEntity)
* **addInsertOrMergeEntity** (permet d'ajouter une opération insertOrMergeEntity)
* **addDeleteEntity** (permet d'ajouter une opération deleteEntity)

L’exemple suivant montre comment exécuter des opérations **insertEntity** et **deleteEntity** en une seule requête. 

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;
use MicrosoftAzure\Storage\Table\Models\BatchOperations;

// Configure a connection string for Storage Table service.
$connectionString = "DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]"

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

// Create list of batch operation.
$operations = new BatchOperations();

$entity1 = new Entity();
$entity1->setPartitionKey("tasksSeattle");
$entity1->setRowKey("2");
$entity1->addProperty("Description", null, "Clean roof gutters.");
$entity1->addProperty("DueDate",
                        EdmType::DATETIME,
                        new DateTime("2012-11-05T08:15:00-08:00"));
$entity1->addProperty("Location", EdmType::STRING, "Home");

// Add operation to list of batch operations.
$operations->addInsertEntity("mytable", $entity1);

// Add operation to list of batch operations.
$operations->addDeleteEntity("mytable", "tasksSeattle", "1");

try    {
    $tableClient->batch($operations);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Pour plus d’informations sur le traitement par lots d’opérations de table, consultez [Exécution de transactions de groupe d’entités][entity-group-transactions].

## <a name="delete-a-table"></a>Suppression d’une table

Enfin, pour supprimer une table, transmettez son nom à la méthode **TableRestProxy->deleteTable**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    // Delete table.
    $tableClient->deleteTable("mytable");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert les principes de base du service de Table de Stockage Azure et d’Azure Cosmos DB, consultez les liens suivants pour approfondir vos connaissances.

* [Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.

* [Centre de développement PHP](https://azure.microsoft.com/develop/php/).

[download]: https://packagist.org/packages/microsoft/azure-storage-table
[require_once]: https://php.net/require_once
[table-service-timeouts]: /rest/api/storageservices/setting-timeouts-for-table-service-operations

[table-data-model]: /rest/api/storageservices/Understanding-the-Table-Service-Data-Model
[filters]: /rest/api/storageservices/Querying-Tables-and-Entities
[entity-group-transactions]: /rest/api/storageservices/Performing-Entity-Group-Transactions
