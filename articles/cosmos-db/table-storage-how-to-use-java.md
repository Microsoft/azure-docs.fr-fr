---
title: Utiliser le Stockage Table Azure ou l’API Table Azure Cosmos DB avec Java
description: Stockez des données structurées dans le cloud à l’aide du Stockage Table Azure ou de l’API Table Azure Cosmos DB avec Java.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: Java
ms.topic: sample
ms.date: 12/10/2020
author: ThomasWeiss
ms.author: thweiss
ms.custom: devx-track-java
ms.openlocfilehash: a5da5e1717f897d2236fd73f0fff525e157f7a0e
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093687"
---
# <a name="how-to-use-azure-table-storage-or-azure-cosmos-db-table-api-from-java"></a>Guide pratique pour utiliser le Stockage Table Azure ou l’API Table d’Azure Cosmos DB avec Java

[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Cet article vous montre comment créer des tables, stocker vos données et effectuer des opérations CRUD sur les données. Choisissez le service de Table Azure ou l’API Table Azure Cosmos DB. Les exemples sont écrits en Java et utilisent le [SDK Stockage Azure v8 pour Java][Azure Storage SDK for Java]. Les scénarios abordés sont les suivants : **création**, **suppression** et **affichage d’une liste de tables**, **insertion**, **interrogation**, **modification** et **suppression** des entités d’une table. Pour plus d'informations sur les tables, consultez la section [Étapes suivantes](#next-steps) .

> [!IMPORTANT]
> La version [v8][Azure Storage SDK for Java] est la dernière version du SDK Stockage Azure qui prend en charge Stockage Table. Une nouvelle version du SDK Stockage Table pour Java sera bientôt disponible.

> [!NOTE]
> un Kit de développement logiciel (SDK) est disponible pour les développeurs qui utilisent Azure Storage sur des appareils Android. Pour plus d'informations, consultez la page [Kit de développement logiciel (SDK) Azure Storage pour Android][Azure Storage SDK for Android].
>

## <a name="create-an-azure-service-account"></a>Créer un compte de service Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Créer un compte de stockage Azure**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Création d’un compte Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-java-application"></a>Création d’une application Java

Dans ce guide, vous allez utiliser des fonctionnalités de stockage que vous pouvez exécuter dans une application Java en local, ou dans le code s’exécutant dans un rôle Web ou un rôle de travail dans Azure.

Pour utiliser les exemples de cet article, installez le Kit de développement Java (JDK), puis créez un compte de stockage Azure ou un compte Azure Cosmos DB dans votre abonnement Azure. Vérifiez ensuite que votre système de développement répond à la configuration minimale requise et aux dépendances listées dans le dépôt [SDK Stockage Azure pour Java][Azure Storage SDK for Java] sur GitHub. Si tel est le cas, vous pouvez suivre les instructions pour télécharger et installer les bibliothèques Stockage Azure pour Java sur votre système à partir de ce référentiel. Une fois ces tâches effectuées, vous pouvez créer une application Java utilisant les exemples de cet article.

## <a name="configure-your-application-to-access-table-storage"></a>Configuration de votre application pour accéder au stockage de table

Ajoutez les instructions import suivantes au début du fichier Java dans lequel vous voulez utiliser des API de stockage Azure ou l’API Table d’Azure Cosmos DB pour accéder aux tables :

```java
// Include the following imports to use table APIs
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.table.*;
import com.microsoft.azure.storage.table.TableQuery.*;
```

## <a name="add-your-connection-string"></a>Ajouter votre chaîne de connexion

Vous pouvez vous connecter au compte de stockage Azure ou au compte de l’API Table Azure Cosmos DB. Obtenez la chaîne de connexion en fonction du type de compte que vous utilisez.

### <a name="add-an-azure-storage-connection-string"></a>Ajouter une chaîne de connexion de stockage Azure

Un client de stockage Azure utilise une chaîne de connexion de stockage pour stocker des points de terminaison et des informations d’identification permettant d’accéder aux services de gestion des données. Lors de l’exécution dans une application cliente, vous devez spécifier la chaîne de connexion au stockage au format suivant, en indiquant le nom de votre compte de stockage et sa clé d’accès primaire, correspondant aux valeurs **AccountName** et **AccountKey**, sur le [portail Azure](https://portal.azure.com). 

Cet exemple vous montre comment déclarer un champ statique pour qu’il contienne une chaîne de connexion :

```java
// Define the connection-string with your values.
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key";
```

### <a name="add-an-azure-cosmos-db-table-api-connection-string"></a>Ajouter une chaîne de connexion d’API Table d’Azure Cosmos DB

Un compte Azure Cosmos DB utilise une chaîne de connexion pour stocker le point de terminaison de table et vos informations d’identification. Lors de l’exécution dans une application cliente, vous devez spécifier la chaîne de connexion Azure Cosmos DB au format suivant, en indiquant le nom de votre compte Azure Cosmos DB et sa clé d’accès primaire, correspondant aux valeurs **AccountName** et **AccountKey**, sur le [portail Azure](https://portal.azure.com). 

Cet exemple vous montre comment déclarer un champ statique pour qu’il contienne la chaîne de connexion Azure Cosmos DB :

```java
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=https;" + 
    "AccountName=your_cosmosdb_account;" + 
    "AccountKey=your_account_key;" + 
    "TableEndpoint=https://your_endpoint;" ;
```

Dans une application exécutée au sein d’un rôle dans Azure, vous pouvez stocker cette chaîne dans le fichier de configuration de service *ServiceConfiguration.cscfg*. Vous pouvez y accéder en appelant la méthode **RoleEnvironment.getConfigurationSettings**. Voici un exemple de code vous permettant d'extraire la chaîne de connexion à partir d'un élément **Setting** nommé *StorageConnectionString* dans le fichier de configuration de service :

```java
// Retrieve storage account from connection-string.
String storageConnectionString =
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");
```

Vous pouvez également stocker la chaîne de connexion dans le fichier config.properties du projet :

```java
StorageConnectionString = DefaultEndpointsProtocol=https;AccountName=your_account;AccountKey=your_account_key;TableEndpoint=https://your_table_endpoint/
```

Les exemples ci-dessous partent du principe que vous avez utilisé l’une de ces méthodes pour obtenir la chaîne de connexion de stockage.

## <a name="create-a-table"></a>Créer une table

Un objet `CloudTableClient`vous permet d’obtenir les objets de référence pour les tables et entités. Le code suivant crée un objet `CloudTableClient` et l’utilise pour créer un objet `CloudTable` représentant une table nommée « people ». 

> [!NOTE]
> Il existe d’autres façons de créer des objets `CloudStorageAccount`. Pour plus d’informations, consultez `CloudStorageAccount` dans la [Référence du Kit de développement logiciel (SDK) du client Azure Storage].
>

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create the table if it doesn't exist.
    String tableName = "people";
    CloudTable cloudTable = tableClient.getTableReference(tableName);
    cloudTable.createIfNotExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="list-the-tables"></a>Établir une liste des tables

Pour obtenir une liste de tables, appelez la méthode **CloudTableClient.listTables()** afin de récupérer une liste de noms de tables pouvant être itérée.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Loop through the collection of table names.
    for (String table : tableClient.listTables())
    {
        // Output each table name.
        System.out.println(table);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="add-an-entity-to-a-table"></a>Ajout d'une entité à une table

Les entités mappent à des objets Java en utilisant une implémentation de classe personnalisée `TableEntity`. Pour plus de commodité, la classe `TableServiceEntity` implémente `TableEntity` et utilise la réflexion pour mapper les propriétés aux méthodes getter et setter nommées pour les propriétés. Pour ajouter une entité à une table, commencez par créer une classe définissant les propriétés de votre entité. Le code suivant définit une classe d’entité utilisant le prénom du client en tant que clé de ligne et son nom de famille en tant que clé de partition. Ensemble, les clés de partition et de ligne d’une entité identifient l’entité de façon unique dans la table. Les requêtes d'entités dont les clés de partition sont identiques sont plus rapides que celles d'entités dont les clés de partition sont différentes.

```java
public class CustomerEntity extends TableServiceEntity {
    public CustomerEntity(String lastName, String firstName) {
        this.partitionKey = lastName;
        this.rowKey = firstName;
    }

    public CustomerEntity() { }

    String email;
    String phoneNumber;

    public String getEmail() {
        return this.email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPhoneNumber() {
        return this.phoneNumber;
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
    }
}
```

Les opérations de table impliquant des entités ont besoin d’un objet `TableOperation`. Cet objet définit l’opération à effectuer sur une entité, qui peut être exécutée avec un objet `CloudTable`. Le code suivant crée une instance de la classe `CustomerEntity` avec des données client à stocker. Le code appelle ensuite l’opération `TableOperation`.insertOrReplace** pour créer un objet `TableOperation` afin d’insérer une entité dans une table, puis y associe la nouvelle `CustomerEntity`. Enfin, le code appelle la méthode `execute` sur l’objet `CloudTable`, en spécifiant la table « people » et le nouvel élément `TableOperation`, qui envoie ensuite une demande au service de stockage pour insérer la nouvelle entité client dans la table « people », ou la remplacer si elle existe déjà.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.setEmail("Walter@contoso.com");
    customer1.setPhoneNumber("425-555-0101");

    // Create an operation to add the new customer to the people table.
    TableOperation insertCustomer1 = TableOperation.insertOrReplace(customer1);

    // Submit the operation to the table service.
    cloudTable.execute(insertCustomer1);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="insert-a-batch-of-entities"></a>Insertion d'un lot d'entités

Vous pouvez insérer un lot d'entités dans le service de Table en une seule opération d'écriture. Le code suivant crée un objet `TableBatchOperation`, puis y ajoute trois opérations d’insertion. Chaque opération d’insertion est ajoutée en créant un objet d’entité, en définissant ses valeurs, puis en appelant la méthode `insert` sur l’objet `TableBatchOperation` pour associer la nouvelle entité à une nouvelle opération d’insertion. Le code appelle ensuite la méthode `execute` sur l’objet `CloudTable`, en spécifiant la table « people » et l’objet `TableBatchOperation` qui envoie le lot d’opérations de table au service de stockage en une seule demande.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Define a batch operation.
    TableBatchOperation batchOperation = new TableBatchOperation();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a customer entity to add to the table.
    CustomerEntity customer = new CustomerEntity("Smith", "Jeff");
    customer.setEmail("Jeff@contoso.com");
    customer.setPhoneNumber("425-555-0104");
    batchOperation.insertOrReplace(customer);

    // Create another customer entity to add to the table.
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.setEmail("Ben@contoso.com");
    customer2.setPhoneNumber("425-555-0102");
    batchOperation.insertOrReplace(customer2);

    // Create a third customer entity to add to the table.
    CustomerEntity customer3 = new CustomerEntity("Smith", "Denise");
    customer3.setEmail("Denise@contoso.com");
    customer3.setPhoneNumber("425-555-0103");
    batchOperation.insertOrReplace(customer3);

    // Execute the batch of operations on the "people" table.
    cloudTable.execute(batchOperation);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

Quelques remarques sur les opérations par lots :

* Vous pouvez effectuer jusqu'à 100 opérations d'insertion, de suppression, de fusion, de remplacement, d'insertion ou fusion et d'insertion ou de remplacement dans n'importe quelle combinaison en un seul lot.
* Une opération par lot peut comporter une opération d’extraction, s’il s’agit de la seule opération du lot.
* Toutes les entités d'une opération par lot doivent avoir la même clé de partition.
* Une opération par lot est limitée à une charge utile de données de 4 Mo.

## <a name="retrieve-all-entities-in-a-partition"></a>Extraction de toutes les entités d'une partition

Pour interroger une table afin d’obtenir les entités d’une partition, utilisez une requête `TableQuery`. Appelez `TableQuery.from` pour créer une requête sur une table donnée qui renvoie un type de résultat spécifique. Le code suivant indique un filtre pour les entités où 'Smith' est la clé de partition. `TableQuery.generateFilterCondition` est une méthode d’assistance pour créer des filtres pour les requêtes. Appelez `where` sur la référence retournée par la méthode `TableQuery.from` pour appliquer le filtre à la requête. Lorsque la requête est exécutée avec un appel de `execute` sur l’objet `CloudTable`, elle retourne un `Iterator` avec le type de résultat `CustomerEntity` spécifié. Vous pouvez ensuite utiliser l’`Iterator` renvoyé dans une boucle « ForEach » pour traiter les résultats. Ce code imprime les champs de chaque entité dans les résultats de requête vers la console.

```java
try
{
    // Define constants for filters.
    final String PARTITION_KEY = "PartitionKey";
    final String ROW_KEY = "RowKey";
    final String TIMESTAMP = "Timestamp";

    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a filter condition where the partition key is "Smith".
    String partitionFilter = TableQuery.generateFilterCondition(
        PARTITION_KEY,
        QueryComparisons.EQUAL,
        "Smith");

    // Specify a partition query, using "Smith" as the partition key filter.
    TableQuery<CustomerEntity> partitionQuery =
        TableQuery.from(CustomerEntity.class)
        .where(partitionFilter);

    // Loop through the results, displaying information about the entity.
    for (CustomerEntity entity : cloudTable.execute(partitionQuery)) {
        System.out.println(entity.getPartitionKey() +
            " " + entity.getRowKey() +
            "\t" + entity.getEmail() +
            "\t" + entity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="retrieve-a-range-of-entities-in-a-partition"></a>Extraction d’un ensemble d’entités dans une partition

Si vous ne voulez pas interroger toutes les entités d'une partition, vous pouvez définir une plage en utilisant les opérateurs de comparaison dans un filtre. Le code suivant combine deux filtres pour obtenir toutes les entités dans la partition « Smith » où la clé de ligne (prénom) commence par une lettre allant jusqu'à « E », puis imprime les résultats de la requête. Si vous utilisez les entités ajoutées à la table de la section de ce guide consacrée à l'insertion de lot, seulement deux entités sont renvoyées (Ben et Denise Smith). Jeff Smith n'est pas inclus.

```java
try
{
    // Define constants for filters.
    final String PARTITION_KEY = "PartitionKey";
    final String ROW_KEY = "RowKey";
    final String TIMESTAMP = "Timestamp";

    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a filter condition where the partition key is "Smith".
    String partitionFilter = TableQuery.generateFilterCondition(
        PARTITION_KEY,
        QueryComparisons.EQUAL,
        "Smith");

    // Create a filter condition where the row key is less than the letter "E".
    String rowFilter = TableQuery.generateFilterCondition(
        ROW_KEY,
        QueryComparisons.LESS_THAN,
        "E");

    // Combine the two conditions into a filter expression.
    String combinedFilter = TableQuery.combineFilters(partitionFilter,
        Operators.AND, rowFilter);

    // Specify a range query, using "Smith" as the partition key,
    // with the row key being up to the letter "E".
    TableQuery<CustomerEntity> rangeQuery =
        TableQuery.from(CustomerEntity.class)
        .where(combinedFilter);

    // Loop through the results, displaying information about the entity
    for (CustomerEntity entity : cloudTable.execute(rangeQuery)) {
        System.out.println(entity.getPartitionKey() +
            " " + entity.getRowKey() +
            "\t" + entity.getEmail() +
            "\t" + entity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="retrieve-a-single-entity"></a>Extraction d'une seule entité

Vous pouvez écrire une requête pour extraire une seule entité. Le code suivant appelle `TableOperation.retrieve` avec des clés de partition et de ligne pour spécifier le client « Jeff Smith », ce qui évite de créer une requête `Table Query` et d’utiliser des filtres pour parvenir au même résultat. Lors de son exécution, l'opération d'extraction renvoie une seule entité, plutôt que plusieurs. La méthode `getResultAsType` convertit le résultat dans le type de la cible d’assignation, un objet `CustomerEntity`. Si ce type n’est pas compatible avec celui spécifié pour la requête, une exception est levée. La valeur null est renvoyée si aucune entité n'a de correspondance exacte avec la clé de partition et de ligne. La méthode la plus rapide pour extraire une seule entité dans le service de Table consiste à spécifier une clé de partition et une clé de ligne.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff"
    TableOperation retrieveSmithJeff =
        TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Submit the operation to the table service and get the specific entity.
    CustomerEntity specificEntity =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Output the entity.
    if (specificEntity != null)
    {
        System.out.println(specificEntity.getPartitionKey() +
            " " + specificEntity.getRowKey() +
            "\t" + specificEntity.getEmail() +
            "\t" + specificEntity.getPhoneNumber());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="modify-an-entity"></a>Modifier une entité

Pour modifier une entité, extrayez-la dans le service de Table, apportez les modifications souhaitées à l'objet d'entité, puis enregistrez les modifications sur le service de Table via une opération de remplacement ou de fusion. Le code suivant modifie le numéro de téléphone d'un client existant. Plutôt que d’appeler **TableOperation.insert** comme nous l’avons fait pour l’opération d’insertion, ce code appelle **TableOperation.replace**. La méthode **CloudTable.execute** appelle le service de table et l'entité est remplacée, sauf si une autre application l'a changée après son extraction par cette application. Lorsque c'est le cas, une exception est levée et l'entité doit être extraite, modifiée, puis de nouveau enregistrée. Ce modèle de nouvelle tentative d'accès concurrentiel optimiste est courant dans un système de stockage distribué.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    TableOperation retrieveSmithJeff =
        TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Submit the operation to the table service and get the specific entity.
    CustomerEntity specificEntity =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Specify a new phone number.
    specificEntity.setPhoneNumber("425-555-0105");

    // Create an operation to replace the entity.
    TableOperation replaceEntity = TableOperation.replace(specificEntity);

    // Submit the operation to the table service.
    cloudTable.execute(replaceEntity);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="query-a-subset-of-entity-properties"></a>Interrogation d'un sous-ensemble de propriétés d'entité

Vous pouvez utiliser une requête de table pour extraire uniquement quelques propriétés d’une entité. Cette technique, nommée « projection », réduit la consommation de bande passante et peut améliorer les performances des requêtes, notamment pour les entités volumineuses. La requête contenue dans le code suivant utilise la méthode `select` pour retourner uniquement les adresses e-mail des entités dans la table. Les résultats sont projetés dans une collection `String` avec l’aide d’un `Entity Resolver` qui effectue la conversion de type sur les entités retournées par le serveur. Pour en savoir plus sur la projection, reportez-vous à [Tables Azure : présentation d'Upsert et Query Projection][Tables Azure : présentation d'Upsert et Query Projection]. La projection n’étant pas prise en charge sur l’émulateur de stockage local, ce code s’exécute uniquement lors de l’utilisation d’un compte sur le service de table.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Define a projection query that retrieves only the Email property
    TableQuery<CustomerEntity> projectionQuery =
        TableQuery.from(CustomerEntity.class)
        .select(new String[] {"Email"});

    // Define an Entity resolver to project the entity to the Email value.
    EntityResolver<String> emailResolver = new EntityResolver<String>() {
        @Override
        public String resolve(String PartitionKey, String RowKey, Date timeStamp, HashMap<String, EntityProperty> properties, String etag) {
            return properties.get("Email").getValueAsString();
        }
    };

    // Loop through the results, displaying the Email values.
    for (String projectedString :
        cloudTable.execute(projectionQuery, emailResolver)) {
            System.out.println(projectedString);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="insert-or-replace-an-entity"></a>Insérer ou remplacer une entité

Il arrive souvent de vouloir ajouter une entité à une table sans savoir si elle existe dans la table. Une opération d’insertion ou de remplacement permet d’envoyer une seule requête pour insérer l’entité si elle n’existe pas ou la remplacer si elle existe. À partir des exemples précédents, le code suivant insère ou remplace l'entité « Walter Harp ». Après la création d'une entité, ce code appelle la méthode **TableOperation.insertOrReplace** . Ce code appelle ensuite la commande **execute** sur l’objet **CloudTable** avec la table et l’opération de table insert ou replace comme paramètre. Pour mettre à jour seulement une partie de l'entité, il est possible d'utiliser la méthode **TableOperation.insertOrMerge** à la place. L’opération d’insertion ou de remplacement n’étant pas prise en charge sur l’émulateur de stockage local, ce code s’exécute uniquement lors de l’utilisation d’un compte sur le service de table. Pour plus d'informations sur les opérations d'insertion ou de remplacement et d'insertion ou de fusion, consultez la rubrique [Tables Azure : présentation d'Upsert et Query Projection][Tables Azure : présentation d'Upsert et Query Projection].

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer5 = new CustomerEntity("Harp", "Walter");
    customer5.setEmail("Walter@contoso.com");
    customer5.setPhoneNumber("425-555-0106");

    // Create an operation to add the new customer to the people table.
    TableOperation insertCustomer5 = TableOperation.insertOrReplace(customer5);

    // Submit the operation to the table service.
    cloudTable.execute(insertCustomer5);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="delete-an-entity"></a>Suppression d’une entité

Il est facile de supprimer une entité après l’avoir récupérée. Une fois l’entité récupérée, appelez `TableOperation.delete` avec l’entité à supprimer. Appelez ensuite `execute` sur l’objet `CloudTable`. Le code suivant extrait et supprime une entité de client.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Create a cloud table object for the table.
    CloudTable cloudTable = tableClient.getTableReference("people");

    // Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
    TableOperation retrieveSmithJeff = TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    CustomerEntity entitySmithJeff =
        cloudTable.execute(retrieveSmithJeff).getResultAsType();

    // Create an operation to delete the entity.
    TableOperation deleteSmithJeff = TableOperation.delete(entitySmithJeff);

    // Submit the delete operation to the table service.
    cloudTable.execute(deleteSmithJeff);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

## <a name="delete-a-table"></a>Suppression d’une table

Pour finir, le code suivant supprime une table d'un compte de stockage. Vous devez patienter environ 40 secondes avant de pouvoir recréer une table après l’avoir supprimée. 

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the table client.
    CloudTableClient tableClient = storageAccount.createCloudTableClient();

    // Delete the table and all its data if it exists.
    CloudTable cloudTable = tableClient.getTableReference("people");
    cloudTable.deleteIfExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

[!INCLUDE [storage-check-out-samples-java](../../includes/storage-check-out-samples-java.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Getting Started with Azure Table Service in Java (Prise en main du service de Table Azure en Java)](https://github.com/Azure-Samples/storage-table-java-getting-started)
* [Microsoft Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome et gratuite de Microsoft qui vous permet d’exploiter visuellement les données de Stockage Azure sur Windows, macOS et Linux.
* [Kit de développement logiciel (SDK) Azure Storage pour Java][Azure Storage SDK for Java]
* [Référence du Kit de développement logiciel (SDK) du client Azure Storage][Azure Storage Client SDK Reference]
* [API REST d’Azure Storage][Azure Storage REST API]
* [Blog de l'équipe Azure Storage][Azure Storage Team Blog]

Pour plus d’informations, consultez [Azure pour les développeurs Java](/java/azure).

[Azure SDK for Java]: https://go.microsoft.com/fwlink/?LinkID=525671
[Azure Storage SDK for Java]: https://github.com/Azure/azure-storage-java/tree/v8.6.5
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Référence du Kit de développement logiciel (SDK) du client Azure Storage]: https://azure.github.io/azure-storage-java/
[Azure Storage REST API]: /rest/api/storageservices/
[Azure Storage Team Blog]: https://blogs.msdn.microsoft.com/windowsazurestorage/