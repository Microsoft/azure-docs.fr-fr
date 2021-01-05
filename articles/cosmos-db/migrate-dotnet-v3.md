---
title: Migration d’une application de façon à utiliser le SDK .NET Azure Cosmos DB 3.0 (com.azure.cosmos)
description: Apprenez à mettre à niveau votre application .NET existante du Kit de développement logiciel (SDK) v2 vers le SDK .NET v3 plus récent (package com.azure.cosmos) pour l’API Core (SQL).
author: stefArroyo
ms.author: esarroyo
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/23/2020
ms.openlocfilehash: 514a6c6daccfe63865ae0b2b9f5bf29c5cbedc29
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334019"
---
# <a name="migrate-your-application-to-use-the-azure-cosmos-db-net-sdk-v3"></a>Migration d’une application de façon à utiliser le SDK .NET Azure Cosmos DB v3
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!IMPORTANT]
> Pour en savoir plus sur le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v3, consultez les [notes de publication](sql-api-sdk-dotnet-standard.md), le [référentiel GitHub .NET](https://github.com/Azure/azure-cosmos-dotnet-v3), les [conseils de performances](performance-tips-dotnet-sdk-v3-sql.md) du Kit de développement logiciel (SDK) .NET v3 et le [guide de résolution des problèmes](troubleshoot-dot-net-sdk.md).
>

Cet article souligne quelques-unes des considérations relatives à la mise à niveau de votre application .NET existante vers le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v3 plus récent pour l’API Core (SQL). Le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v3 correspond à l’espace de noms Microsoft.Azure.Cosmos. Vous pouvez utiliser les informations fournies dans ce document si vous migrez votre application à partir de l’un des Kits de développement logiciel (SDK) .NET Azure Cosmos DB suivants :

* Kit de développement logiciel (SDK) .NET Framework Azure Cosmos DB v2 pour l’API SQL
* Kit SDK .NET Core Azure Cosmos DB v2 pour l’API SQL

Les instructions de cet article vous aideront également à migrer les bibliothèques externes suivantes, qui font désormais partie du Kit de développement logiciel (SDK) .NET Azure Cosmos DB v3 pour l’API Core (SQL) :

* Bibliothèque du processeur de flux de modification .NET 2.0
* Bibliothèque d’exécuteur en bloc .NET 1.1 ou ultérieure

## <a name="whats-new-in-the-net-v3-sdk"></a>Nouveautés dans le Kit de développement logiciel (SDK) .NET v3

Le Kit de développement logiciel (SDK) v3 contient de nombreuses améliorations en matière de convivialité et de performances, notamment :

* Nom intuitif du modèle de programmation
* .NET Standard 2.0**
* Amélioration des performances grâce à la prise en charge de l’API de flux
* Hiérarchie fluide qui remplace la nécessité d’une fabrique d’URI
* Prise en charge intégrée des bibliothèques du processeur de flux de modification
* Prise en charge intégrée des opérations en bloc
* Simulations d’API pour faciliter les tests unitaires
* Prise en charge des lots transactionnels et de Blazor
* Sérialiseurs enfichables
* Mise à l’échelle des conteneurs non partitionnés et mise à l’échelle automatique des conteneurs

** Le Kit de développement logiciel (SDK) cible le format .NET Standard 2.0 qui unifie les Kits de développement logiciel (SDK) .NET Framework Azure Cosmos DB et .NET Core existants en un seul SDK .NET. Vous pouvez utiliser le Kit de développement logiciel (SDK) .NET sur n’importe quelle plateforme qui implémente .NET Standard 2.0, y compris vos applications .NET Framework 4.6.1 et ultérieures et vos applications .NET Core 2.0 et ultérieures.

La majeure partie de la mise en réseau, de la logique de nouvelle tentative et des niveaux inférieurs du Kit de développement logiciel (SDK) restent largement inchangée.

**Le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v3 est désormais open source.** Nous recevons toutes les demandes de tirage (pull request) et nous enregistrerons les problèmes et suivrons les réactions sur[GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/). Nous travaillerons à l’adoption de toute fonctionnalité susceptible d’améliorer l’expérience utilisateur.

## <a name="why-migrate-to-the-net-v3-sdk"></a>Pourquoi migrer vers le Kit de développement logiciel (SDK) .NET v3

Outre les nombreuses améliorations en matière de convivialité et de performances, les nouvelles fonctionnalités investies dans le dernier Kit de développement logiciel (SDK) ne seront pas reportées sur les versions antérieures.

Bien qu’il ne soit pas prévu dans l’immédiat de [mettre hors service le support des Kits de développement logiciel (SDK) 2.0](sql-api-sdk-dotnet.md), les Kits de développement logiciel seront remplacés par des versions plus récentes à l’avenir et le SDK passera en mode de maintenance. Pour une expérience de développement optimale, nous vous recommandons de toujours démarrer avec la version prise en charge la plus récente du Kit de développement logiciel (SDK).

## <a name="major-name-changes-from-v2-sdk-to-v3-sdk"></a>Changements majeurs de nom du SDK v2 au SDK v3

Les changements de noms suivants ont été appliqués dans l’ensemble du Kit de développement logiciel (SDK) .NET 3.0 pour s’aligner sur les conventions d’affectation de noms de l’API Core (SQL) :

* `DocumentClient` est renommé `CosmosClient`
* `Collection` est renommé `Container`
* `Document` est renommé `Item`

Tous les objets de ressource sont renommés avec des propriétés supplémentaires, qui incluent le nom de la ressource pour plus de clarté.

Voici quelques-uns des principaux changements apportés aux noms de classe :

| SDK .NET v2 | SDK .NET v3 |
|-------------|-------------|
|`Microsoft.Azure.Documents.Client.DocumentClient`|`Microsoft.Azure.CosmosClient`|
|`Microsoft.Azure.Documents.Client.ConnectionPolicy`|`Microsoft.Azure.Cosmos.CosmosClientOptions`|
|`Microsoft.Azure.Documents.Client.DocumentClientException` |`Microsoft.Azure.Cosmos.CosmosException`|
|`Microsoft.Azure.Documents.Client.Database`|`Microsoft.Azure.Cosmos.DatabaseProperties`|
|`Microsoft.Azure.Documents.Client.DocumentCollection`|`Microsoft.Azure.Cosmos.ContainerProperties`|
|`Microsoft.Azure.Documents.Client.RequestOptions`|`Microsoft.Azure.Cosmos.ItemRequestOptions`|
|`Microsoft.Azure.Documents.Client.FeedOptions`|`Microsoft.Azure.Cosmos.QueryRequestOptions`|
|`Microsoft.Azure.Documents.Client.StoredProcedure`|`Microsoft.Azure.Cosmos.StoredProcedureProperties`|
|`Microsoft.Azure.Documents.Client.Trigger`|`Microsoft.Azure.Cosmos.TriggerProperties`|

### <a name="classes-replaced-on-net-v3-sdk"></a>Classes remplacées dans le Kit de développement logiciel (SDK) .NET v3

Les classes suivantes ont été remplacées dans le Kit de développement logiciel (SDK) 3.0 :

* `Microsoft.Azure.Documents.UriFactory`

* `Microsoft.Azure.Documents.Document`

* `Microsoft.Azure.Documents.Resource`

La classe Microsoft.Azure.Documents.UriFactory a été remplacée par la conception fluide. La conception fluide crée des URL en interne et permet de faire circuler un objet `Container` unique à la place des objets `DocumentClient`, `DatabaseName` et `DocumentCollection`.

### <a name="changes-to-item-id-generation"></a>Modifications apportées à la génération d’ID d’élément

L’ID d’élément n’est plus renseigné automatiquement dans le Kit de développement logiciel (SDK) .NET v3. Par conséquent, l’ID de l’élément doit inclure spécifiquement un ID généré. Consultez l’exemple suivant :

```csharp
[JsonProperty(PropertyName = "id")]
public Guid Id { get; set; }
```

### <a name="changed-default-behavior-for-connection-mode"></a>Modification du comportement par défaut pour le mode de connexion

Le Kit de développement logiciel (SDK) v3 est désormais défini par défaut sur les modes de connexion Direct + TCP par rapport au SDK v2 précédent, qui était défini par défaut sur les modes de connexion Passerelle + HTTPS. Cette modification permet d’améliorer les performances et la scalabilité.

### <a name="changes-to-feedoptions-queryrequestoptions-in-v30-sdk"></a>Modifications apportées à FeedOptions (QueryRequestOptions dans le SDK v3.0)

La classe `FeedOptions` dans le Kit de développement logiciel (SDK) v2 a été renommée `QueryRequestOptions` dans le Kit de développement logiciel (SDK) v3 et, dans la classe, plusieurs propriétés ont subi des modifications de nom et/ou de valeur par défaut ou ont été supprimées complètement.  

`FeedOptions.MaxDegreeOfParallelism` a été renommée `QueryRequestOptions.MaxConcurrency`, la valeur par défaut et le comportement associé restent les mêmes, et les opérations exécutées côté client pendant l’exécution d’une requête parallèle seront exécutées en série sans parallélisme.

`FeedOptions.EnableCrossPartitionQuery` a été supprimée et le comportement par défaut dans le Kit de développement logiciel (SDK) 3.0 est que les requêtes entre les partitions seront exécutées sans qu’il soit nécessaire d’activer la propriété spécifiquement.

`FeedOptions.PopulateQueryMetrics` est activée par défaut avec les résultats qui sont présents dans la propriété Diagnostics de la réponse.

`FeedOptions.RequestContinuation` a maintenant été promue aux méthodes de requête elles-mêmes.

Les propriétés suivantes ont été supprimées :

* `FeedOptions.DisableRUPerMinuteUsage`

* `FeedOptions.EnableCrossPartitionQuery`

* `FeedOptions.JsonSerializerSettings`

* `FeedOptions.PartitionKeyRangeId`

* `FeedOptions.PopulateQueryMetrics`

### <a name="constructing-a-client"></a>Construction d’un client

Le Kit de développement logiciel (SDK) .NET v3 fournit une classe `CosmosClientBuilder` fluide qui remplace la nécessité d’une fabrique d’URI pour le SDK v2.

L’exemple suivant crée une propriété `CosmosClientBuilder` avec un ConsistencyLevel fort et une liste d’emplacements par défaut :

```csharp
CosmosClientBuilder cosmosClientBuilder = new CosmosClientBuilder(
    accountEndpoint: "https://testcosmos.documents.azure.com:443/",
    authKeyOrResourceToken: "SuperSecretKey")
.WithConsistencyLevel(ConsistencyLevel.Strong)
.WithApplicationRegion(Regions.EastUS);
CosmosClient client = cosmosClientBuilder.Build();
```

### <a name="using-the-change-feed-processor-apis-directly-from-the-v3-sdk"></a>Utilisation des API du processeur de flux de modification directement à partir du SDK v3

Le Kit de développement logiciel (SDK) v3 offre une prise en charge intégrée des API du processeur de flux de modification, ce qui vous permet d’utiliser le même SDK pour générer votre application et modifier l’implémentation du processeur de flux. Auparavant, vous deviez utiliser une bibliothèque du processeur de flux de modification distincte.

Pour plus d’informations, consultez [comment migrer de la bibliothèque du processeur de flux de modification vers le Kit de développement logiciel (SDK) .NET Azure Cosmos DB v3](how-to-migrate-from-change-feed-library.md).

### <a name="using-the-bulk-executor-library-directly-from-the-v3-sdk"></a>Utilisation de la bibliothèque d’exécuteur en bloc directement à partir du Kit de développement logiciel (SDK) v3

Le Kit de développement logiciel (SDK) v3 offre une prise en charge intégrée de la bibliothèque d’exécuteur en bloc, ce qui vous permet d’utiliser le même Kit de développement logiciel (SDK) pour créer votre application et effectuer des opérations en bloc. Auparavant, vous deviez utiliser une bibliothèque d’exécuteur en bloc distincte.

Pour plus d’informations, consultez [comment migrer de la bibliothèque d’exécuteur en bloc vers la prise en charge en bloc dans le SDK .NET Azure Cosmos DB v3](how-to-migrate-from-bulk-executor-library.md).

## <a name="code-snippet-comparisons"></a>Comparaison d’extraits de code

L’extrait de code suivant montre les différences dans la façon dont les ressources sont créées entre les Kits de développement logiciel (SDK) .NET v2 et v3.

## <a name="database-operations"></a>Opérations de base de données

### <a name="create-a-database"></a>Création d'une base de données

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
// Create database with no shared provisioned throughput
DatabaseResponse databaseResponse = await client.CreateDatabaseIfNotExistsAsync(DatabaseName);
Database database = databaseResponse;
DatabaseProperties databaseProperties = databaseResponse;

// Create a database with a shared manual provisioned throughput
string databaseIdManual = new string(DatabaseName + "_SharedManualThroughput");
database = await client.CreateDatabaseIfNotExistsAsync(databaseIdManual, ThroughputProperties.CreateManualThroughput(400));

// Create a database with shared autoscale provisioned throughput
string databaseIdAutoscale = new string(DatabaseName + "_SharedAutoscaleThroughput");
database = await client.CreateDatabaseIfNotExistsAsync(databaseIdAutoscale, ThroughputProperties.CreateAutoscaleThroughput(4000));
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
// Create database
ResourceResponse<Database> databaseResponse = await client.CreateDatabaseIfNotExistsAsync(new Database { Id = DatabaseName });
Database database = databaseResponse;

// Create a database with shared standard provisioned throughput
database = await client.CreateDatabaseIfNotExistsAsync(new Database{ Id = databaseIdStandard }, new RequestOptions { OfferThroughput = 400 });

// Creating a database with shared autoscale provisioned throughput from v2 SDK is not supported use v3 SDK
```
---

### <a name="read-a-database-by-id"></a>Lire une base de données par identifiant

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
// Read a database
Console.WriteLine($"{Environment.NewLine} Read database resource: {DatabaseName}");
database = client.GetDatabase(DatabaseName);
Console.WriteLine($"{Environment.NewLine} database { database.Id.ToString()}");

// Read all databases
string findQueryText = "SELECT * FROM c";
using (FeedIterator<DatabaseProperties> feedIterator = client.GetDatabaseQueryIterator<DatabaseProperties>(findQueryText))
{
    while (feedIterator.HasMoreResults)
    {
        FeedResponse<DatabaseProperties> databaseResponses = await feedIterator.ReadNextAsync();
        foreach (DatabaseProperties _database in databaseResponses)
        {
            Console.WriteLine($"{ Environment.NewLine} database {_database.Id.ToString()}");
        }
    }
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
// Read a database
database = await client.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(DatabaseName));
Console.WriteLine("\n database {0}", database.Id.ToString());

// Read all databases
Console.WriteLine("\n1.1 Reading all databases resources");
foreach (Database _database in await client.ReadDatabaseFeedAsync())
{
    Console.WriteLine("\n database {0} \n {1}", _database.Id.ToString(), _database.ToString());
}
```
---

### <a name="delete-a-database"></a>Supprimer une base de données

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
// Delete a database
await client.GetDatabase(DatabaseName).DeleteAsync();
Console.WriteLine($"{ Environment.NewLine} database {DatabaseName} deleted.");

// Delete all databases in an account
string deleteQueryText = "SELECT * FROM c";
using (FeedIterator<DatabaseProperties> feedIterator = client.GetDatabaseQueryIterator<DatabaseProperties>(deleteQueryText))
{
    while (feedIterator.HasMoreResults)
    {
        FeedResponse<DatabaseProperties> databaseResponses = await feedIterator.ReadNextAsync();
        foreach (DatabaseProperties _database in databaseResponses)
        {
            await client.GetDatabase(_database.Id).DeleteAsync();
            Console.WriteLine($"{ Environment.NewLine} database {_database.Id} deleted");
        }
    }
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
// Delete a database
database = await client.DeleteDatabaseAsync(UriFactory.CreateDatabaseUri(DatabaseName));
Console.WriteLine(" database {0} deleted.", DatabaseName);

// Delete all databases in an account
foreach (Database _database in await client.ReadDatabaseFeedAsync())
{
    await client.DeleteDatabaseAsync(UriFactory.CreateDatabaseUri(_database.Id));
    Console.WriteLine("\n database {0} deleted", _database.Id);
}
```
---

## <a name="container-operations"></a>Opérations sur les conteneurs

### <a name="create-a-container-autoscale--time-to-live-with-expiration"></a>Créer un conteneur (mise à l’échelle automatique + durée de vie avec expiration)

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task CreateManualThroughputContainer(Database database)
{
    // Set throughput to the minimum value of 400 RU/s manually configured throughput
    string containerIdManual = ContainerName + "_Manual";
    ContainerResponse container = await database.CreateContainerIfNotExistsAsync(
        id: containerIdManual,
        partitionKeyPath: partitionKeyPath,
        throughput: 400);
}

// Create container with autoscale
private static async Task CreateAutoscaleThroughputContainer(Database database)
{
    string autoscaleContainerId = ContainerName + "_Autoscale";
    ContainerProperties containerProperties = new ContainerProperties(autoscaleContainerId, partitionKeyPath);

    Container container = await database.CreateContainerIfNotExistsAsync(
        containerProperties: containerProperties,
        throughputProperties: ThroughputProperties.CreateAutoscaleThroughput(autoscaleMaxThroughput: 4000);
}

// Create a container with TTL Expiration
private static async Task CreateContainerWithTtlExpiration(Database database)
{
    string containerIdManualwithTTL = ContainerName + "_ManualTTL";

    ContainerProperties properties = new ContainerProperties
        (id: containerIdManualwithTTL,
        partitionKeyPath: partitionKeyPath);

    properties.DefaultTimeToLive = (int)TimeSpan.FromDays(1).TotalSeconds; //expire in 1 day

    ContainerResponse containerResponse = await database.CreateContainerIfNotExistsAsync(containerProperties: properties);
    ContainerProperties returnedProperties = containerResponse;
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
// Create a collection
private static async Task CreateManualThroughputContainer(DocumentClient client)
{
    string containerIdManual = ContainerName + "_Manual";

    // Set throughput to the minimum value of 400 RU/s manually configured throughput

    DocumentCollection collectionDefinition = new DocumentCollection();
    collectionDefinition.Id = containerIdManual;
    collectionDefinition.PartitionKey.Paths.Add(partitionKeyPath);

    DocumentCollection partitionedCollection = await client.CreateDocumentCollectionIfNotExistsAsync(
        UriFactory.CreateDatabaseUri(DatabaseName),
        collectionDefinition,
        new RequestOptions { OfferThroughput = 400 });
}

private static async Task CreateAutoscaleThroughputContainer(DocumentClient client)
{
        // .NET v2 SDK does not support the creation of provisioned autoscale throughput containers
}

 private static async Task CreateContainerWithTtlExpiration(DocumentClient client)
{
    string containerIdManualwithTTL = ContainerName + "_ManualTTL";

    DocumentCollection collectionDefinition = new DocumentCollection();
    collectionDefinition.Id = containerIdManualwithTTL;
    collectionDefinition.DefaultTimeToLive = (int)TimeSpan.FromDays(1).TotalSeconds; //expire in 1 day
    collectionDefinition.PartitionKey.Paths.Add(partitionKeyPath);

    DocumentCollection partitionedCollection = await client.CreateDocumentCollectionIfNotExistsAsync(
        UriFactory.CreateDatabaseUri(DatabaseName),
        collectionDefinition,
        new RequestOptions { OfferThroughput = 400 });

}
```
---

### <a name="read-container-properties"></a>Lire les propriétés du conteneur

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task ReadContainerProperties(Database database)
{
    string containerIdManual = ContainerName + "_Manual";
    Container container = database.GetContainer(containerIdManual);
    ContainerProperties containerProperties = await container.ReadContainerAsync();
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
private static async Task ReadContainerProperties(DocumentClient client)
{
    string containerIdManual = ContainerName + "_Manual";
    DocumentCollection collection = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseName, containerIdManual));
}
```
---

### <a name="delete-a-container"></a>Supprimer un conteneur

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task DeleteContainers(Database database)
{
    string containerIdManual = ContainerName + "_Manual";

    // Delete a container
    await database.GetContainer(containerIdManual).DeleteContainerAsync();

    // Delete all CosmosContainer resources for a database
    using (FeedIterator<ContainerProperties> feedIterator = database.GetContainerQueryIterator<ContainerProperties>())
    {
        while (feedIterator.HasMoreResults)
        {
            foreach (ContainerProperties _container in await feedIterator.ReadNextAsync())
            {
                await database.GetContainer(_container.Id).DeleteContainerAsync();
                Console.WriteLine($"{Environment.NewLine}  deleted container {_container.Id}");
            }
        }
    }
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
private static async Task DeleteContainers(DocumentClient client)
{
    // Delete a collection
    string containerIdManual = ContainerName + "_Manual";
    await client.DeleteDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseName, containerIdManual));

    // Delete all containers for a database
    foreach (var collection in await client.ReadDocumentCollectionFeedAsync(UriFactory.CreateDatabaseUri(DatabaseName)))
    {
        await client.DeleteDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseName, collection.Id));
    }
}
```
---

## <a name="item-and-query-operations"></a>Opérations sur les éléments et les requêtes

### <a name="create-an-item"></a>Créer un élément

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task CreateItemAsync(Container container)
{
    // Create a SalesOrder POCO object
    SalesOrder salesOrder1 = GetSalesOrderSample("Account1", "SalesOrder1");
    ItemResponse<SalesOrder> response = await container.CreateItemAsync(salesOrder1,
        new PartitionKey(salesOrder1.AccountNumber));
}

private static async Task RunBasicOperationsOnDynamicObjects(Container container)
{
    // Dynamic Object
    dynamic salesOrder = new
    {
        id = "SalesOrder5",
        AccountNumber = "Account1",
        PurchaseOrderNumber = "PO18009186470",
        OrderDate = DateTime.UtcNow,
        Total = 5.95,
    };
    Console.WriteLine("\nCreating item");
    ItemResponse<dynamic> response = await container.CreateItemAsync<dynamic>(
        salesOrder, new PartitionKey(salesOrder.AccountNumber));
    dynamic createdSalesOrder = response.Resource;
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
private static async Task CreateItemAsync(DocumentClient client)
{
    // Create a SalesOrder POCO object
    SalesOrder salesOrder1 = GetSalesOrderSample("Account1", "SalesOrder1");
    await client.CreateDocumentAsync(
        UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
        salesOrder1,
        new RequestOptions { PartitionKey = new PartitionKey("Account1")});
}

private static async Task RunBasicOperationsOnDynamicObjects(DocumentClient client)
{
    // Create a dynamic object
    dynamic salesOrder = new
    {
        id= "SalesOrder5",
        AccountNumber = "Account1",
        PurchaseOrderNumber = "PO18009186470",
        OrderDate = DateTime.UtcNow,
        Total = 5.95,
    };
    ResourceResponse<Document> response = await client.CreateDocumentAsync(
        UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
        salesOrder,
        new RequestOptions { PartitionKey = new PartitionKey(salesOrder.AccountNumber)});

    dynamic createdSalesOrder = response.Resource;
    }
```
---

### <a name="read-all-the-items-in-a-container"></a>Lire tous les éléments figurant dans un conteneur

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task ReadAllItems(Container container)
{
    // Read all items in a container
    List<SalesOrder> allSalesForAccount1 = new List<SalesOrder>();

    using (FeedIterator<SalesOrder> resultSet = container.GetItemQueryIterator<SalesOrder>(
        queryDefinition: null,
        requestOptions: new QueryRequestOptions()
        {
            PartitionKey = new PartitionKey("Account1"),
            MaxItemCount = 5
        }))
    {
        while (resultSet.HasMoreResults)
        {
            FeedResponse<SalesOrder> response = await resultSet.ReadNextAsync();
            SalesOrder salesOrder = response.First();
            Console.WriteLine($"\n1.3.1 Account Number: {salesOrder.AccountNumber}; Id: {salesOrder.Id}");
            allSalesForAccount1.AddRange(response);
        }
    }
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
private static async Task ReadAllItems(DocumentClient client)
{
    // Read all items in a collection
    List<SalesOrder> allSalesForAccount1 = new List<SalesOrder>();

    string continuationToken = null;
    do
    {
        var feed = await client.ReadDocumentFeedAsync(
            UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
            new FeedOptions { MaxItemCount = 5, RequestContinuation = continuationToken });
        continuationToken = feed.ResponseContinuation;
        foreach (Document document in feed)
        {
            SalesOrder salesOrder = (SalesOrder)(dynamic)document;
            Console.WriteLine($"\n1.3.1 Account Number: {salesOrder.AccountNumber}; Id: {salesOrder.Id}");
            allSalesForAccount1.Add(salesOrder);

        }
    } while (continuationToken != null);
}
```
---

### <a name="query-items"></a>Éléments de requête

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task QueryItems(Container container)
{
    // Query for items by a property other than Id
    QueryDefinition queryDefinition = new QueryDefinition(
        "select * from sales s where s.AccountNumber = @AccountInput")
        .WithParameter("@AccountInput", "Account1");

    List<SalesOrder> allSalesForAccount1 = new List<SalesOrder>();
    using (FeedIterator<SalesOrder> resultSet = container.GetItemQueryIterator<SalesOrder>(
        queryDefinition,
        requestOptions: new QueryRequestOptions()
        {
            PartitionKey = new PartitionKey("Account1"),
            MaxItemCount = 1
        }))
    {
        while (resultSet.HasMoreResults)
        {
            FeedResponse<SalesOrder> response = await resultSet.ReadNextAsync();
            SalesOrder sale = response.First();
            Console.WriteLine($"\n Account Number: {sale.AccountNumber}; Id: {sale.Id};");
            allSalesForAccount1.AddRange(response);
        }
    }
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
private static async Task QueryItems(DocumentClient client)
{
    // Query for items by a property other than Id
    SqlQuerySpec querySpec = new SqlQuerySpec()
    {
        QueryText = "select * from sales s where s.AccountNumber = @AccountInput",
        Parameters = new SqlParameterCollection()
            {
                new SqlParameter("@AccountInput", "Account1")
            }
    };
    var query = client.CreateDocumentQuery<SalesOrder>(
        UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
        querySpec,
        new FeedOptions {EnableCrossPartitionQuery = true});

    var allSalesForAccount1 = query.ToList();

    Console.WriteLine($"\n1.4.2 Query found {allSalesForAccount1.Count} items.");
}
```
---

### <a name="delete-an-item"></a>Supprimer un élément

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET v3](#tab/dotnet-v3)

```csharp
private static async Task DeleteItemAsync(Container container)
{
    ItemResponse<SalesOrder> response = await container.DeleteItemAsync<SalesOrder>(
        partitionKey: new PartitionKey("Account1"), id: "SalesOrder3");
}
```

# <a name="net-sdk-v2"></a>[SDK .NET v2](#tab/dotnet-v2)

```csharp
private static async Task DeleteItemAsync(DocumentClient client)
{
    ResourceResponse<Document> response = await client.DeleteDocumentAsync(
        UriFactory.CreateDocumentUri(DatabaseName, ContainerName, "SalesOrder3"),
        new RequestOptions { PartitionKey = new PartitionKey("Account1") });
}
```
---

## <a name="next-steps"></a>Étapes suivantes

* [Créer une application console](sql-api-get-started.md) pour gérer les données de l’API SQL Azure Cosmos DB avec le Kit de développement logiciel (SDK) v3
* En savoir plus sur ce que le [Kit de développement logiciel (SDK) v3 vous permet de faire](sql-api-dotnet-v3sdk-samples.md)
