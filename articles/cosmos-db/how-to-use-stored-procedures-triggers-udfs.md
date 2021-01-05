---
title: Inscrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans les kits SDK Azure Cosmos DB
description: Découvrez comment inscrire et appeler des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur à l’aide des kits de développement logiciel (SDK) Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tisande
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: 022a45199cfc2d467b1d0d408e86cb5d621070d9
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339846"
---
# <a name="how-to-register-and-use-stored-procedures-triggers-and-user-defined-functions-in-azure-cosmos-db"></a>Comment inscrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

L’API SQL dans Azure Cosmos DB prend en charge l’inscription et l’appel de procédures stockées, déclencheurs et fonctions définies par l’utilisateur (UDF) écrites en JavaScript. Vous pouvez utiliser les SDK API SQL [.NET](sql-api-sdk-dotnet.md), [.NET Core](sql-api-sdk-dotnet-core.md), [Java](sql-api-sdk-java.md), [JavaScript](sql-api-sdk-node.md), [Node.js](sql-api-sdk-node.md) ou [Python](sql-api-sdk-python.md) pour inscrire et appeler les procédures stockées. Après avoir défini des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur, vous pouvez les charger et les afficher dans le [portail Azure](https://portal.azure.com/) à l’aide de l’Explorateur de données.

## <a name="how-to-run-stored-procedures"></a><a id="stored-procedures"></a> Comment exécuter des procédures stockées

Les procédures stockées sont écrites à l’aide de JavaScript. Elles peuvent créer, mettre à jour, lire, interroger et supprimer des éléments dans un conteneur Azure Cosmos. Pour plus d’informations concernant l’écriture des procédures stockées dans Azure Cosmos DB, consultez l’article [Comment écrire des procédures stockées dans Azure Cosmos DB](how-to-write-stored-procedures-triggers-udfs.md#stored-procedures).

Les exemples suivants montrent comment inscrire et appeler une procédure stockée à l’aide des kits de développement logiciel (SDK) Azure Cosmos DB. Reportez-vous à [Créer un Document](how-to-write-stored-procedures-triggers-udfs.md#create-an-item) puisque la source pour cette procédure stockée est enregistrée en tant que `spCreateToDoItem.js`.

> [!NOTE]
> Pour les conteneurs partitionnés, lorsque vous exécutez une procédure stockée, vous devez fournir une valeur de clé de partition dans les options de requête. Les procédures stockées se limitent toujours à une clé de partition. Les éléments qui ont une valeur de clé de partition différente ne seront pas visibles dans la procédure stockée. Cela s’applique également aux déclencheurs.

### <a name="stored-procedures---net-sdk-v2"></a>Procédures stockées - Kit de développement logiciel (SDK) .NET V2

L’exemple suivant montre comment inscrire une procédure stockée à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
string storedProcedureId = "spCreateToDoItems";
StoredProcedure newStoredProcedure = new StoredProcedure
   {
       Id = storedProcedureId,
       Body = File.ReadAllText($@"..\js\{storedProcedureId}.js")
   };
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
var response = await client.CreateStoredProcedureAsync(containerUri, newStoredProcedure);
StoredProcedure createdStoredProcedure = response.Resource;
```

L’exemple suivant montre comment appeler une procédure stockée à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
dynamic[] newItems = new dynamic[]
{
    new {
        category = "Personal",
        name = "Groceries",
        description = "Pick up strawberries",
        isComplete = false
    },
    new {
        category = "Personal",
        name = "Doctor",
        description = "Make appointment for check up",
        isComplete = false
    }
};

Uri uri = UriFactory.CreateStoredProcedureUri("myDatabase", "myContainer", "spCreateToDoItem");
RequestOptions options = new RequestOptions { PartitionKey = new PartitionKey("Personal") };
var result = await client.ExecuteStoredProcedureAsync<string>(uri, options, new[] { newItems });
```

### <a name="stored-procedures---net-sdk-v3"></a>Procédures stockées - Kit de développement logiciel (SDK) .NET V3

L’exemple suivant montre comment inscrire une procédure stockée à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
string storedProcedureId = "spCreateToDoItems";
StoredProcedureResponse storedProcedureResponse = await client.GetContainer("myDatabase", "myContainer").Scripts.CreateStoredProcedureAsync(new StoredProcedureProperties
{
    Id = storedProcedureId,
    Body = File.ReadAllText($@"..\js\{storedProcedureId}.js")
});
```

L’exemple suivant montre comment appeler une procédure stockée à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
dynamic[] newItems = new dynamic[]
{
    new {
        category = "Personal",
        name = "Groceries",
        description = "Pick up strawberries",
        isComplete = false
    },
    new {
        category = "Personal",
        name = "Doctor",
        description = "Make appointment for check up",
        isComplete = false
    }
};

var result = await client.GetContainer("database", "container").Scripts.ExecuteStoredProcedureAsync<string>("spCreateToDoItem", new PartitionKey("Personal"), new[] { newItems });
```

### <a name="stored-procedures---java-sdk"></a>Procédures stockées - Kit de développement logiciel (SDK) Java

L’exemple suivant montre comment inscrire une procédure stockée à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
StoredProcedure newStoredProcedure = new StoredProcedure(
    "{" +
        "  'id':'spCreateToDoItems'," +
        "  'body':" + new String(Files.readAllBytes(Paths.get("..\\js\\spCreateToDoItems.js"))) +
    "}");
//toBlocking() blocks the thread until the operation is complete and is used only for demo.  
StoredProcedure createdStoredProcedure = asyncClient.createStoredProcedure(containerLink, newStoredProcedure, null)
    .toBlocking().single().getResource();
```

L’exemple suivant montre comment appeler une procédure stockée à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String sprocLink = String.format("%s/sprocs/%s", containerLink, "spCreateToDoItems");
final CountDownLatch successfulCompletionLatch = new CountDownLatch(1);

List<ToDoItem> ToDoItems = new ArrayList<ToDoItem>();

class ToDoItem {
    public String category;
    public String name;
    public String description;
    public boolean isComplete;
}

ToDoItem newItem = new ToDoItem();
newItem.category = "Personal";
newItem.name = "Groceries";
newItem.description = "Pick up strawberries";
newItem.isComplete = false;

ToDoItems.add(newItem)

newItem.category = "Personal";
newItem.name = "Doctor";
newItem.description = "Make appointment for check up";
newItem.isComplete = false;

ToDoItems.add(newItem)

RequestOptions requestOptions = new RequestOptions();
requestOptions.setPartitionKey(new PartitionKey("Personal"));

Object[] storedProcedureArgs = new Object[] { ToDoItems };
asyncClient.executeStoredProcedure(sprocLink, requestOptions, storedProcedureArgs)
    .subscribe(storedProcedureResponse -> {
        String storedProcResultAsString = storedProcedureResponse.getResponseAsString();
        successfulCompletionLatch.countDown();
        System.out.println(storedProcedureResponse.getActivityId());
    }, error -> {
        successfulCompletionLatch.countDown();
        System.err.println("an error occurred while executing the stored procedure: actual cause: "
                + error.getMessage());
    });

successfulCompletionLatch.await();
```

### <a name="stored-procedures---javascript-sdk"></a>Procédures stockées - Kit de développement logiciel (SDK) JavaScript

L’exemple suivant montre comment inscrire une procédure stockée à l’aide du kit de développement logiciel (SDK) JavaScript

```javascript
const container = client.database("myDatabase").container("myContainer");
const sprocId = "spCreateToDoItems";
await container.scripts.storedProcedures.create({
    id: sprocId,
    body: require(`../js/${sprocId}`)
});
```

L’exemple suivant montre comment appeler une procédure stockée à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const newItem = [{
    category: "Personal",
    name: "Groceries",
    description: "Pick up strawberries",
    isComplete: false
}];
const container = client.database("myDatabase").container("myContainer");
const sprocId = "spCreateToDoItems";
const {body: result} = await container.scripts.storedProcedure(sprocId).execute(newItem, {partitionKey: newItem[0].category});
```

### <a name="stored-procedures---python-sdk"></a>Procédures stockées - Kit de développement logiciel (SDK) Python

L’exemple suivant montre comment inscrire une procédure stockée à l’aide du kit de développement logiciel (SDK) Python :

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/spCreateToDoItems.js') as file:
    file_contents = file.read()

sproc = {
    'id': 'spCreateToDoItem',
    'serverScript': file_contents,
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
created_sproc = container.scripts.create_stored_procedure(body=sproc) 
```

L’exemple suivant montre comment appeler une procédure stockée à l’aide du kit de développement logiciel (SDK) Python :

```python
import uuid

new_id= str(uuid.uuid4())

# Creating a document for a container with "id" as a partition key.

new_item =   {
      "id": new_id, 
      "category":"Personal",
      "name":"Groceries",
      "description":"Pick up strawberries",
      "isComplete":False
   }
result = container.scripts.execute_stored_procedure(sproc=created_sproc,params=[[new_item]], partition_key=new_id) 
```

## <a name="how-to-run-pre-triggers"></a><a id="pre-triggers"></a>Comment exécuter les pré-déclencheurs

Les exemples suivants montrent comment inscrire et appeler un pré-déclencheur à l’aide des kits de développement logiciel (SDK) Azure Cosmos DB. Reportez-vous à [Exemple de pré-déclencheur](how-to-write-stored-procedures-triggers-udfs.md#pre-triggers) puisque la source pour ce pré-déclencheur est enregistrée en tant que `trgPreValidateToDoItemTimestamp.js`.

Lors de l’exécution, les pré-déclencheurs sont transmis dans l’objet RequestOptions en spécifiant `PreTriggerInclude`, puis en passant le nom du déclencheur dans un objet de liste.

> [!NOTE]
> Même si le nom du déclencheur est transmis en tant que liste, vous ne pouvez pas exécuter plus d’un déclencheur par opération.

### <a name="pre-triggers---net-sdk-v2"></a>Pré-déclencheurs - Kit de développement logiciel (SDK) .NET V2

Le code suivant montre comment inscrire un pré-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
string triggerId = "trgPreValidateToDoItemTimestamp";
Trigger trigger = new Trigger
{
    Id =  triggerId,
    Body = File.ReadAllText($@"..\js\{triggerId}.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Pre
};
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateTriggerAsync(containerUri, trigger);
```

Le code suivant montre comment appeler un pré-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
RequestOptions requestOptions = new RequestOptions { PreTriggerInclude = new List<string> { "trgPreValidateToDoItemTimestamp" } };
await client.CreateDocumentAsync(containerUri, newItem, requestOptions);
```

### <a name="pre-triggers---net-sdk-v3"></a>Pré-déclencheurs - Kit de développement logiciel (SDK) .NET V3

Le code suivant montre comment inscrire un pré-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
await client.GetContainer("database", "container").Scripts.CreateTriggerAsync(new TriggerProperties
{
    Id = "trgPreValidateToDoItemTimestamp",
    Body = File.ReadAllText("@..\js\trgPreValidateToDoItemTimestamp.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Pre
});
```

Le code suivant montre comment appeler un pré-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

await client.GetContainer("database", "container").CreateItemAsync(newItem, null, new ItemRequestOptions { PreTriggers = new List<string> { "trgPreValidateToDoItemTimestamp" } });
```

### <a name="pre-triggers---java-sdk"></a>Pré-déclencheurs - Kit de développement logiciel (SDK) Java

Le code suivant montre comment inscrire un pré-déclencheur à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String triggerId = "trgPreValidateToDoItemTimestamp";
Trigger trigger = new Trigger();
trigger.setId(triggerId);
trigger.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", triggerId)));
trigger.setTriggerOperation(TriggerOperation.Create);
trigger.setTriggerType(TriggerType.Pre);
//toBlocking() blocks the thread until the operation is complete and is used only for demo. 
Trigger createdTrigger = asyncClient.createTrigger(containerLink, trigger, new RequestOptions()).toBlocking().single().getResource();
```

Le code suivant montre comment appeler un pré-déclencheur à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
    Document item = new Document("{ "
            + "\"category\": \"Personal\", "
            + "\"name\": \"Groceries\", "
            + "\"description\": \"Pick up strawberries\", "
            + "\"isComplete\": false, "
            + "}"
            );
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPreTriggerInclude(Arrays.asList("trgPreValidateToDoItemTimestamp"));
//toBlocking() blocks the thread until the operation is complete and is used only for demo. 
asyncClient.createDocument(containerLink, item, requestOptions, false).toBlocking();
```

### <a name="pre-triggers---javascript-sdk"></a>Pré-déclencheurs - Kit de développement logiciel (SDK) JavaScript

Le code suivant montre comment inscrire un pré-déclencheur à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPreValidateToDoItemTimestamp";
await container.triggers.create({
    id: triggerId,
    body: require(`../js/${triggerId}`),
    triggerOperation: "create",
    triggerType: "pre"
});
```

Le code suivant montre comment appeler un pré-déclencheur à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPreValidateToDoItemTimestamp";
await container.items.create({
    category: "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
}, {preTriggerInclude: [triggerId]});
```

### <a name="pre-triggers---python-sdk"></a>Pré-déclencheurs - Kit de développement logiciel (SDK) Python

Le code suivant montre comment inscrire un pré-déclencheur à l’aide du kit de développement logiciel (SDK) Python :

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/trgPreValidateToDoItemTimestamp.js') as file:
    file_contents = file.read()

trigger_definition = {
    'id': 'trgPreValidateToDoItemTimestamp',
    'serverScript': file_contents,
    'triggerType': documents.TriggerType.Pre,
    'triggerOperation': documents.TriggerOperation.All
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
trigger = container.scripts.create_trigger(trigger_definition)
```

Le code suivant montre comment appeler un pré-déclencheur à l’aide du kit de développement logiciel (SDK) Python :

```python
item = {'category': 'Personal', 'name': 'Groceries',
        'description': 'Pick up strawberries', 'isComplete': False}
container.create_item(item, {'pre_trigger_include': 'trgPreValidateToDoItemTimestamp'})
```

## <a name="how-to-run-post-triggers"></a><a id="post-triggers"></a>Comment exécuter les post-déclencheurs

Les exemples suivants montrent comment inscrire un post-déclencheur à l’aide des kits de développement logiciel (SDK) Azure Cosmos DB. Reportez-vous à [Exemple de post-déclencheur](how-to-write-stored-procedures-triggers-udfs.md#post-triggers) puisque la source pour ce post-déclencheur est enregistrée en tant que `trgPostUpdateMetadata.js`.

### <a name="post-triggers---net-sdk-v2"></a>Post-déclencheurs - Kit de développement logiciel (SDK) .NET V2

Le code suivant montre comment inscrire un post-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
string triggerId = "trgPostUpdateMetadata";
Trigger trigger = new Trigger
{
    Id = triggerId,
    Body = File.ReadAllText($@"..\js\{triggerId}.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Post
};
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateTriggerAsync(containerUri, trigger);
```

Le code suivant montre comment appeler un post-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
var newItem = { 
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};

RequestOptions options = new RequestOptions { PostTriggerInclude = new List<string> { "trgPostUpdateMetadata" } };
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.createDocumentAsync(containerUri, newItem, options);
```

### <a name="post-triggers---net-sdk-v3"></a>Post-déclencheurs - Kit de développement logiciel (SDK) .NET V3

Le code suivant montre comment inscrire un post-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
await client.GetContainer("database", "container").Scripts.CreateTriggerAsync(new TriggerProperties
{
    Id = "trgPostUpdateMetadata",
    Body = File.ReadAllText(@"..\js\trgPostUpdateMetadata.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Post
});
```

Le code suivant montre comment appeler un post-déclencheur à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
var newItem = { 
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};

await client.GetContainer("database", "container").CreateItemAsync(newItem, null, new ItemRequestOptions { PostTriggers = new List<string> { "trgPostUpdateMetadata" } });
```

### <a name="post-triggers---java-sdk"></a>Post-déclencheurs - Kit de développement logiciel (SDK) Java

Le code suivant montre comment inscrire un post-déclencheur à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String triggerId = "trgPostUpdateMetadata";
Trigger trigger = new Trigger();
trigger.setId(triggerId);
trigger.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", triggerId)))));
trigger.setTriggerOperation(TriggerOperation.Create);
trigger.setTriggerType(TriggerType.Post);
Trigger createdTrigger = asyncClient.createTrigger(containerLink, trigger, new RequestOptions()).toBlocking().single().getResource();
```

Le code suivant montre comment appeler un post-déclencheur à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
Document item = new Document(String.format("{ "
    + "\"name\": \"artist_profile_1023\", "
    + "\"artist\": \"The Band\", "
    + "\"albums\": [\"Hellujah\", \"Rotators\", \"Spinning Top\"]"
    + "}"
));
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPostTriggerInclude(Arrays.asList("trgPostUpdateMetadata"));
//toBlocking() blocks the thread until the operation is complete, and is used only for demo.
asyncClient.createDocument(containerLink, item, requestOptions, false).toBlocking();
```

### <a name="post-triggers---javascript-sdk"></a>Post-déclencheurs - Kit de développement logiciel (SDK) JavaScript

Le code suivant montre comment inscrire un post-déclencheur à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPostUpdateMetadata";
await container.triggers.create({
    id: triggerId,
    body: require(`../js/${triggerId}`),
    triggerOperation: "create",
    triggerType: "post"
});
```

Le code suivant montre comment appeler un post-déclencheur à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const item = {
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPostUpdateMetadata";
await container.items.create(item, {postTriggerInclude: [triggerId]});
```

### <a name="post-triggers---python-sdk"></a>Post-déclencheurs - Kit de développement logiciel (SDK) Python

Le code suivant montre comment inscrire un post-déclencheur à l’aide du kit de développement logiciel (SDK) Python :

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/trgPostValidateToDoItemTimestamp.js') as file:
    file_contents = file.read()

trigger_definition = {
    'id': 'trgPostValidateToDoItemTimestamp',
    'serverScript': file_contents,
    'triggerType': documents.TriggerType.Post,
    'triggerOperation': documents.TriggerOperation.All
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
trigger = container.scripts.create_trigger(trigger_definition)
```

Le code suivant montre comment appeler un post-déclencheur à l’aide du kit de développement logiciel (SDK) Python :

```python
item = {'category': 'Personal', 'name': 'Groceries',
        'description': 'Pick up strawberries', 'isComplete': False}
container.create_item(item, {'post_trigger_include': 'trgPreValidateToDoItemTimestamp'})
```

## <a name="how-to-work-with-user-defined-functions"></a><a id="udfs"></a> Comment utiliser des fonctions définies par l’utilisateur

Les exemples suivants montrent comment inscrire une fonction définie par l’utilisateur à l’aide des kits de développement logiciel (SDK) Azure Cosmos DB. Reportez-vous à [Exemple de fonction définie par l’utilisateur](how-to-write-stored-procedures-triggers-udfs.md#udfs) puisque la source pour ce post-déclencheur est enregistrée en tant que `udfTax.js`.

### <a name="user-defined-functions---net-sdk-v2"></a>Fonction définie par l’utilisateur - Kit de développement logiciel (SDK) .NET V2

Le code suivant montre comment inscrire une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
string udfId = "Tax";
var udfTax = new UserDefinedFunction
{
    Id = udfId,
    Body = File.ReadAllText($@"..\js\{udfId}.js")
};

Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateUserDefinedFunctionAsync(containerUri, udfTax);

```

Le code suivant montre comment appeler une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) .NET V2 :

```csharp
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
var results = client.CreateDocumentQuery<dynamic>(containerUri, "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000"));

foreach (var result in results)
{
    //iterate over results
}
```

### <a name="user-defined-functions---net-sdk-v3"></a>Fonction définie par l’utilisateur - Kit de développement logiciel (SDK) .NET V3

Le code suivant montre comment inscrire une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
await client.GetContainer("database", "container").Scripts.CreateUserDefinedFunctionAsync(new UserDefinedFunctionProperties
{
    Id = "Tax",
    Body = File.ReadAllText(@"..\js\Tax.js")
});
```

Le code suivant montre comment appeler une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) .NET V3 :

```csharp
var iterator = client.GetContainer("database", "container").GetItemQueryIterator<dynamic>("SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000");
while (iterator.HasMoreResults)
{
    var results = await iterator.ReadNextAsync();
    foreach (var result in results)
    {
        //iterate over results
    }
}
```

### <a name="user-defined-functions---java-sdk"></a>Fonctions définies par l’utilisateur - Kit de développement logiciel (SDK) Java

Le code suivant montre comment inscrire une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String udfId = "Tax";
UserDefinedFunction udf = new UserDefinedFunction();
udf.setId(udfId);
udf.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", udfId)))));
//toBlocking() blocks the thread until the operation is complete and is used only for demo.
UserDefinedFunction createdUDF = client.createUserDefinedFunction(containerLink, udf, new RequestOptions()).toBlocking().single().getResource();
```

Le code suivant montre comment appeler une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) Java :

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
Observable<FeedResponse<Document>> queryObservable = client.queryDocuments(containerLink, "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000", new FeedOptions());
final CountDownLatch completionLatch = new CountDownLatch(1);
queryObservable.subscribe(
        queryResultPage -> {
            System.out.println("Got a page of query result with " +
                    queryResultPage.getResults().size());
        },
        // terminal error signal
        e -> {
            e.printStackTrace();
            completionLatch.countDown();
        },

        // terminal completion signal
        () -> {
            completionLatch.countDown();
        });
completionLatch.await();
```

### <a name="user-defined-functions---javascript-sdk"></a>Fonctions définies par l’utilisateur - Kit de développement logiciel (SDK) JavaScript

Le code suivant montre comment inscrire une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const container = client.database("myDatabase").container("myContainer");
const udfId = "Tax";
await container.userDefinedFunctions.create({
    id: udfId,
    body: require(`../js/${udfId}`)
```

Le code suivant montre comment appeler une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) JavaScript :

```javascript
const container = client.database("myDatabase").container("myContainer");
const sql = "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000";
const {result} = await container.items.query(sql).toArray();
```

### <a name="user-defined-functions---python-sdk"></a>Fonctions définies par l’utilisateur - Kit de développement logiciel (SDK) Python

Le code suivant montre comment inscrire une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) Python :

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/udfTax.js') as file:
    file_contents = file.read()
udf_definition = {
    'id': 'Tax',
    'serverScript': file_contents,
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
udf = container.scripts.create_user_defined_function(udf_definition)
```

Le code suivant montre comment appeler une fonction définie par l’utilisateur à l’aide du kit de développement logiciel (SDK) Python :

```python
results = list(container.query_items(
    'query': 'SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000'))
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus de concepts et comment écrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB :

- [Working with Azure Cosmos DB stored procedures, triggers, and user-defined functions](stored-procedures-triggers-udfs.md) (Utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur Azure Cosmos DB)
- [Working with JavaScript language-integrated query API with Azure Cosmos DB](javascript-query-api.md) (Utiliser l’API de requête avec langage intégré JavaScript dans Azure Cosmos DB)
- [Comment écrire des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB](how-to-write-stored-procedures-triggers-udfs.md)
- [How to write stored procedures and triggers in Azure Cosmos DB by using the JavaScript query API](how-to-write-javascript-query-api.md) (Comment écrire des procédures stockées et des déclencheurs à l’aide de l’API de requête JavaScript dans Azure Cosmos DB)
