---
title: Liaison de sortie Azure Cosmos DB pour Functions 2.x et versions ultérieures
description: Découvrez comment utiliser la liaison de sortie Azure Cosmos DB dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/24/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 454ac9a377800bd11a53250569c3e7b65bac713a
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558791"
---
# <a name="azure-cosmos-db-output-binding-for-azure-functions-2x-and-higher"></a>Liaison de sortie Azure Cosmos DB pour Azure Functions 2.x et versions ultérieures

La liaison de sortie Azure Cosmos DB vous permet d’écrire un nouveau document dans une base de données Azure Cosmos DB en utilisant l’API SQL.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](./functions-bindings-cosmosdb-v2.md).

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

Cette section contient les exemples suivants :

* [Déclencheur de file d’attente, écrire un document](#queue-trigger-write-one-doc-c)
* [Déclencheur de file d’attente, écrire des documents à l’aide d’IAsyncCollector](#queue-trigger-write-docs-using-iasynccollector-c)

Les exemples font référence à un type `ToDoItem` simple :

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

<a id="queue-trigger-write-one-doc-c"></a>

### <a name="queue-trigger-write-one-doc"></a>Déclencheur de la file d’attente, écriture d’un document

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui ajoute un document à une base de données, à l’aide des données fournies dans le message de Stockage File d’attente.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using System;

namespace CosmosDBSamplesV2
{
    public static class WriteOneDoc
    {
        [FunctionName("WriteOneDoc")]
        public static void Run(
            [QueueTrigger("todoqueueforwrite")] string queueMessage,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]out dynamic document,
            ILogger log)
        {
            document = new { Description = queueMessage, id = Guid.NewGuid() };

            log.LogInformation($"C# Queue trigger function inserted one row");
            log.LogInformation($"Description={queueMessage}");
        }
    }
}
```

<a id="queue-trigger-write-docs-using-iasynccollector-c"></a>

### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Déclencheur de la file d’attente, écriture de documents à l’aide d’IAsyncCollector

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui ajoute une collection de documents à une base de données, à l’aide de données fournies dans un JSON de message de file d’attente.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class WriteDocsIAsyncCollector
    {
        [FunctionName("WriteDocsIAsyncCollector")]
        public static async Task Run(
            [QueueTrigger("todoqueueforwritemulti")] ToDoItem[] toDoItemsIn,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]
                IAsyncCollector<ToDoItem> toDoItemsOut,
            ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

            foreach (ToDoItem toDoItem in toDoItemsIn)
            {
                log.LogInformation($"Description={toDoItem.Description}");
                await toDoItemsOut.AddAsync(toDoItem);
            }
        }
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Cette section contient les exemples suivants :

* [Déclencheur de file d’attente, écrire un document](#queue-trigger-write-one-doc-c-script)
* [Déclencheur de file d’attente, écrire des documents à l’aide d’IAsyncCollector](#queue-trigger-write-docs-using-iasynccollector-c-script)


<a id="queue-trigger-write-one-doc-c-script"></a>

### <a name="queue-trigger-write-one-doc"></a>Déclencheur de la file d’attente, écriture d’un document

L’exemple suivant montre une liaison de sortie Azure Cosmos DB dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction utilise une liaison d’entrée de file d’attente pour une file d’attente qui reçoit le code JSON au format suivant :

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La fonction crée des documents Azure Cosmos DB au format suivant pour chaque enregistrement :

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# :

```cs
    #r "Newtonsoft.Json"

    using Microsoft.Azure.WebJobs.Host;
    using Newtonsoft.Json.Linq;
    using Microsoft.Extensions.Logging;

    public static void Run(string myQueueItem, out object employeeDocument, ILogger log)
    {
      log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");

      dynamic employee = JObject.Parse(myQueueItem);

      employeeDocument = new {
        id = employee.name + "-" + employee.employeeId,
        name = employee.name,
        employeeId = employee.employeeId,
        address = employee.address
      };
    }
```

<a id="queue-trigger-write-docs-using-iasynccollector-c-script"></a>

### <a name="queue-trigger-write-docs-using-iasynccollector"></a>Déclencheur de la file d’attente, écriture de documents à l’aide d’IAsyncCollector

Pour créer plusieurs documents, vous pouvez vous lier à `ICollector<T>` ou `IAsyncCollector<T>` où `T` est un des types pris en charge.

Cet exemple fait référence à un type simple `ToDoItem` :

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

Voici le fichier function.json :

```json
{
  "bindings": [
    {
      "name": "toDoItemsIn",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "todoqueueforwritemulti",
      "connectionStringSetting": "AzureWebJobsStorage"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItemsOut",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Voici le code Script C# :

```cs
using System;
using Microsoft.Extensions.Logging;

public static async Task Run(ToDoItem[] toDoItemsIn, IAsyncCollector<ToDoItem> toDoItemsOut, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed {toDoItemsIn?.Length} items");

    foreach (ToDoItem toDoItem in toDoItemsIn)
    {
        log.LogInformation($"Description={toDoItem.Description}");
        await toDoItemsOut.AddAsync(toDoItem);
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de sortie Azure Cosmos DB dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction utilise une liaison d’entrée de file d’attente pour une file d’attente qui reçoit le code JSON au format suivant :

```json
{
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

La fonction crée des documents Azure Cosmos DB au format suivant pour chaque enregistrement :

```json
{
    "id": "John Henry-123456",
    "name": "John Henry",
    "employeeId": "123456",
    "address": "A town nearby"
}
```

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "employeeDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": true,
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
    module.exports = function (context) {

      context.bindings.employeeDocument = JSON.stringify({
        id: context.bindings.myQueueItem.name + "-" + context.bindings.myQueueItem.employeeId,
        name: context.bindings.myQueueItem.name,
        employeeId: context.bindings.myQueueItem.employeeId,
        address: context.bindings.myQueueItem.address
      });

      context.done();
    };
```

Pour BULK INSERT, commencez par créer les objets, puis exécutez la fonction de stringification. Voici le code JavaScript :

```javascript
    module.exports = function (context) {
    
        context.bindings.employeeDocument = JSON.stringify([
        {
            "id": "John Henry-123456",
            "name": "John Henry",
            "employeeId": "123456",
            "address": "A town nearby"
        },
        {
            "id": "John Doe-123457",
            "name": "John Doe",
            "employeeId": "123457",
            "address": "A town far away"
        }]);
    
      context.done();
    };
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre comment écrire un document dans une base de données Azure CosmosDB en tant que sortie d’une fonction.

Une définition de liaison est définie dans *function.json* , où *Type* possède la valeur `cosmosDB`.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "cosmosDB",
      "direction": "out",
      "name": "doc",
      "databaseName": "demodb",
      "collectionName": "data",
      "createIfNotExists": "true",
      "connectionStringSetting": "AzureCosmosDBConnectionString"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

Pour écrire dans la base de données, transmettez un objet de document à la méthode `set` du paramètre Base de données.

```python
import azure.functions as func

def main(req: func.HttpRequest, doc: func.Out[func.Document]) -> func.HttpResponse:

    request_body = req.get_body()

    doc.set(func.Document.from_json(request_body))

    return 'OK'
```

# <a name="java"></a>[Java](#tab/java)

* [Déclencheur de file d’attente, enregistrer le message dans la base de données via la valeur de retour](#queue-trigger-save-message-to-database-via-return-value-java)
* [Déclencheur HTTP, enregistrer un document dans la base de données via la valeur de retour](#http-trigger-save-one-document-to-database-via-return-value-java)
* [Déclencheur HTTP, enregistrer un document dans la base de données via OutputBinding](#http-trigger-save-one-document-to-database-via-outputbinding-java)
* [Déclencheur HTTP, enregistrer plusieurs documents dans la base de données via OutputBinding](#http-trigger-save-multiple-documents-to-database-via-outputbinding-java)


<a id="queue-trigger-save-message-to-database-via-return-value-java"></a>

### <a name="queue-trigger-save-message-to-database-via-return-value"></a>Déclencheur de file d’attente, enregistrer un message dans un base de données via une valeur de retour

L’exemple suivant montre une fonction Java qui ajoute un document à une base de données, à l’aide des données fournies dans un message de Stockage File d’attente.

```java
@FunctionName("getItem")
@CosmosDBOutput(name = "database",
  databaseName = "ToDoList",
  collectionName = "Items",
  connectionStringSetting = "AzureCosmosDBConnection")
public String cosmosDbQueryById(
    @QueueTrigger(name = "msg",
      queueName = "myqueue-items",
      connection = "AzureWebJobsStorage")
    String message,
    final ExecutionContext context)  {
     return "{ id: \"" + System.currentTimeMillis() + "\", Description: " + message + " }";
   }
```
<a id="http-trigger-save-one-document-to-database-via-return-value-java"></a>

#### <a name="http-trigger-save-one-document-to-database-via-return-value"></a>Déclencheur HTTP, enregistrer un document dans une base de données via une valeur de retour

L’exemple suivant illustre une fonction Java dont la signature est annotée avec ```@CosmosDBOutput``` et a une valeur de retour de type ```String```. Le document JSON retourné par la fonction sera automatiquement écrit dans la collection CosmosDB correspondante.

```java
    @FunctionName("WriteOneDoc")
    @CosmosDBOutput(name = "database",
      databaseName = "ToDoList",
      collectionName = "Items",
      connectionStringSetting = "Cosmos_DB_Connection_String")
    public String run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());

        // Parse query parameter
        String query = request.getQueryParameters().get("desc");
        String name = request.getBody().orElse(query);

        // Generate random ID
        final int id = Math.abs(new Random().nextInt());

        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " +
                                    "\"description\": \"" + name + "\"}";

        context.getLogger().info("Document to be saved: " + jsonDocument);

        return jsonDocument;
    }
```

<a id="http-trigger-save-one-document-to-database-via-outputbinding-java"></a>

### <a name="http-trigger-save-one-document-to-database-via-outputbinding"></a>Déclencheur HTTP, enregistrer un document dans une base de données via OutputBinding

L’exemple suivant illustre une fonction Java qui écrit un document dans CosmosDB via un paramètre de sortie ```OutputBinding<T>```. Dans cet exemple, le paramètre ```outputItem``` doit être annoté avec ```@CosmosDBOutput```, non la signature de la fonction. L’utilisation de ```OutputBinding<T>``` permet à votre fonction de tirer parti de la liaison pour écrire le document dans CosmosDB tout en permettant aussi de retourner une valeur différente à l’appelant de la fonction, telle qu’un document JSON ou XML.

```java
    @FunctionName("WriteOneDocOutputBinding")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBOutput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            OutputBinding<String> outputItem,
            final ExecutionContext context) {

        // Parse query parameter
        String query = request.getQueryParameters().get("desc");
        String name = request.getBody().orElse(query);

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());

        // Generate random ID
        final int id = Math.abs(new Random().nextInt());

        // Generate document
        final String jsonDocument = "{\"id\":\"" + id + "\", " +
                                    "\"description\": \"" + name + "\"}";

        context.getLogger().info("Document to be saved: " + jsonDocument);

        // Set outputItem's value to the JSON document to be saved
        outputItem.setValue(jsonDocument);

        // return a different document to the browser or calling client.
        return request.createResponseBuilder(HttpStatus.OK)
                      .body("Document created successfully.")
                      .build();
    }
```

<a id="http-trigger-save-multiple-documents-to-database-via-outputbinding-java"></a>

### <a name="http-trigger-save-multiple-documents-to-database-via-outputbinding"></a>Déclencheur HTTP, enregistrer plusieurs documents dans une base de données via OutputBinding

L’exemple suivant illustre une fonction Java qui écrit plusieurs documents dans CosmosDB via un paramètre de sortie ```OutputBinding<T>```. Dans cet exemple, le paramètre ```outputItem``` est annoté avec ```@CosmosDBOutput```, non la signature de la fonction. Le paramètre de sortie, ```outputItem```, possède une liste d’objets ```ToDoItem``` comme type de paramètre de modèle. L’utilisation de ```OutputBinding<T>``` permet à votre fonction de tirer parti de la liaison pour écrire les documents dans CosmosDB tout en permettant aussi de retourner une valeur différente à l’appelant de la fonction, telle qu’un document JSON ou XML.

```java
    @FunctionName("WriteMultipleDocsOutputBinding")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBOutput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            OutputBinding<List<ToDoItem>> outputItem,
            final ExecutionContext context) {

        // Parse query parameter
        String query = request.getQueryParameters().get("desc");
        String name = request.getBody().orElse(query);

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());

        // Generate documents
        List<ToDoItem> items = new ArrayList<>();

        for (int i = 0; i < 5; i ++) {
          // Generate random ID
          final int id = Math.abs(new Random().nextInt());

          // Create ToDoItem
          ToDoItem item = new ToDoItem(String.valueOf(id), name);

          items.add(item);
        }

        // Set outputItem's value to the list of POJOs to be saved
        outputItem.setValue(items);
        context.getLogger().info("Document to be saved: " + items);

        // return a different document to the browser or calling client.
        return request.createResponseBuilder(HttpStatus.OK)
                      .body("Documents created successfully.")
                      .build();
    }
```

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@CosmosDBOutput` sur les paramètres qui seraient écrits sur Cosmos DB.  Le type de paramètre d’annotation doit être ```OutputBinding<T>```, où T désigne un type Java natif ou un POJO.

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [CosmosDB](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/dev/test/WebJobs.Extensions.CosmosDB.Tests).

Le constructeur de l’attribut accepte le nom de la base de données et le nom de la collection. Pour plus d’informations sur ces paramètres et d’autres propriétés que vous pouvez configurer, consultez [Sortie - configuration](#configuration). Voici un exemple d’attribut `CosmosDB` dans une signature de méthode :

```csharp
    [FunctionName("QueueToDocDB")]
    public static void Run(
        [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
        [CosmosDB("ToDoList", "Items", Id = "id", ConnectionStringSetting = "myCosmosDB")] out dynamic document)
    {
        ...
    }
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

L’annotation `CosmosDBOutput` est disponible pour écrire des données dans Cosmos DB. Vous pouvez appliquer l’annotation à la fonction ou à un paramètre de fonction. En cas d’utilisation sur la méthode de fonction, la valeur de retour de la fonction est ce qui est écrit dans Cosmos DB. Si vous utilisez l’annotation avec un paramètre, le type du paramètre doit être déclaré en tant que `OutputBinding<T>` où `T` un type Java natif ou un POJO.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `CosmosDB`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type**     | n/a | Cette propriété doit être définie sur `cosmosDB`.        |
|**direction**     | n/a | Cette propriété doit être définie sur `out`.         |
|**name**     | n/a | Nom du paramètre de liaison qui représente le document dans la fonction.  |
|**databaseName** | **DatabaseName**|Base de données contenant la collection dans laquelle le document est créé.     |
|**collectionName** |**CollectionName**  | Nom de la collection dans laquelle le document est créé. |
|**createIfNotExists**  |**CreateIfNotExists**    | Valeur booléenne indiquant si la collection doit être créée si elle n’existe pas déjà. La valeur par défaut est *false* car les nouvelles collections sont créées avec un débit réservé, ce qui a des conséquences sur la tarification. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/cosmos-db/).  |
|**partitionKey**|**PartitionKey** |Lorsque `CreateIfNotExists` a la valeur true, définit le chemin de la clé de partition pour la collection créée.|
|**collectionThroughput**|**CollectionThroughput**| Lorsque `CreateIfNotExists` a la valeur true, définit le [débit](../cosmos-db/set-throughput.md) de la collection créée.|
|**connectionStringSetting**    |**ConnectionStringSetting** |Nom du paramètre d’application contenant votre chaîne de connexion Azure Cosmos DB.        |
|**preferredLocations**| **PreferredLocations**| (Facultatif) Définit les endroits par défaut (régions) des comptes de base de données géorépliqués dans le service Azure Cosmos DB. Les valeurs doivent être séparées par des virgules. Par exemple, « USA Est, USA Centre Sud, Europe Nord ». |
|**useMultipleWriteLocations**| **UseMultipleWriteLocations**| (Facultatif) Lorsqu’il est défini sur `true` avec `PreferredLocations`, il peut tirer parti des [écritures multirégions](../cosmos-db/how-to-manage-database-account.md#configure-multiple-write-regions) dans le service Azure Cosmos DB. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

Par défaut, lorsque vous écrivez dans le paramètre de sortie de votre fonction, un document est créé dans votre base de données. Ce document comporte un GUID généré automatiquement en tant qu’ID du document. Vous pouvez spécifier l’ID du document de sortie en spécifiant la propriété `id` dans l’objet JSON qui est passé au paramètre de sortie.

> [!Note]
> Lorsque vous spécifier l’ID d’un document existant, il est remplacé par le nouveau document de sortie.

## <a name="exceptions-and-return-codes"></a>Exceptions et codes de retour

| Liaison | Informations de référence |
|---|---|
| CosmosDB | [Codes d’erreur CosmosDB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) |

<a name="host-json"></a>

## <a name="hostjson-settings"></a>Paramètres host.json

Cette section décrit les paramètres de configuration globale disponibles pour cette liaison dans la version 2.x. Pour plus d’informations sur les paramètres de configuration globale dans la version 2.x, voir [Informations de référence sur le fichier host.json pour Azure Functions version 2.x](functions-host-json.md).

```json
{
    "version": "2.0",
    "extensions": {
        "cosmosDB": {
            "connectionMode": "Gateway",
            "protocol": "Https",
            "leaseOptions": {
                "leasePrefix": "prefix1"
            }
        }
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------|
|GatewayMode|Passerelle|Le mode de connexion utilisé par la fonction lors de la connexion au service Azure Cosmos DB. Les options sont `Direct` et `Gateway`.|
|Protocol|Https|Le protocole de connexion utilisé par la fonction lors de la connexion au service Azure Cosmos DB.  Voir [l’explication des deux modes](../cosmos-db/performance-tips.md#networking).|
|leasePrefix|n/a|Préfixe de bail à utiliser dans toutes les fonctions d’une application.|

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction lors de la création ou de la modification d’un document Azure Cosmos DB (Déclencheur)](./functions-bindings-cosmosdb-v2-trigger.md)
- [Lire un document Azure Cosmos DB (liaison d’entrée)](./functions-bindings-cosmosdb-v2-input.md)
