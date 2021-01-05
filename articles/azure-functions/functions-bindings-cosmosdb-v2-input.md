---
title: Liaison d'entrée Azure Cosmos DB pour Functions 2.x et versions ultérieures
description: Apprenez à utiliser la liaison d'entrée Azure Cosmos DB dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/24/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: dec41a5e05d22891aae9d16280ebb6b0c8da3f20
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185111"
---
# <a name="azure-cosmos-db-input-binding-for-azure-functions-2x-and-higher"></a>Liaison d'entrée Azure Cosmos DB pour Azure Functions 2.x et versions ultérieures

La liaison d’entrée Azure Cosmos DB utilise l’API SQL pour récupérer un ou plusieurs documents Azure Cosmos DB et les transmet au paramètre d’entrée de la fonction. L’ID du document ou les paramètres de requête peuvent être déterminés en fonction du déclencheur qui appelle la fonction.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](./functions-bindings-cosmosdb-v2.md).

> [!NOTE]
> Si la collection est [partitionnée](../cosmos-db/partitioning-overview.md#logical-partitions), les opérations de recherche doivent également spécifier la valeur de clé de partition.
>

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

Cette section contient les exemples suivants :

* [Déclencheur de file d’attente, rechercher l’ID à partir de JSON](#queue-trigger-look-up-id-from-json-c)
* [Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête](#http-trigger-look-up-id-from-query-string-c)
* [Déclencheur HTTP, ID de recherche à partir des données de routage](#http-trigger-look-up-id-from-route-data-c)
* [Déclencheur HTTP, ID de recherche à partir des données de routage, utilisation de SqlQuery](#http-trigger-look-up-id-from-route-data-using-sqlquery-c)
* [Déclencheur HTTP, obtenir plusieurs documents, utilisation de SqlQuery](#http-trigger-get-multiple-docs-using-sqlquery-c)
* [Déclencheur HTTP, obtenir plusieurs documents, utilisation de DocumentClient](#http-trigger-get-multiple-docs-using-documentclient-c)

Les exemples font référence à un type `ToDoItem` simple :

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        [JsonProperty("id")]
        public string Id { get; set; }
        
        [JsonProperty("partitionKey")]
        public string PartitionKey { get; set; }
        
        public string Description { get; set; }
    }
}
```

<a id="queue-trigger-look-up-id-from-json-c"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Déclencheur de file d’attente, rechercher l’ID à partir de JSON 

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui récupère un document unique. La fonction est déclenchée par un message de file d’attente qui contient un objet JSON. Le déclencheur de file d’attente analyse le JSON dans un objet de type `ToDoItemLookup` qui contient l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItemLookup
    {
        public string ToDoItemId { get; set; }

        public string ToDoItemPartitionKeyValue { get; set; }
    }
}
```

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromJSON
    {
        [FunctionName("DocByIdFromJSON")]
        public static void Run(
            [QueueTrigger("todoqueueforlookup")] ToDoItemLookup toDoItemLookup,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{ToDoItemId}",
                PartitionKey = "{ToDoItemPartitionKeyValue}")]ToDoItem toDoItem,
            ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed Id={toDoItemLookup?.ToDoItemId} Key={toDoItemLookup?.ToDoItemPartitionKeyValue}");

            if (toDoItem == null)
            {
                log.LogInformation($"ToDo item not found");
            }
            else
            {
                log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
            }
        }
    }
}
```

<a id="http-trigger-look-up-id-from-query-string-c"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise une chaîne de requête pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

>[!NOTE]
>Le paramètre de chaîne de requête HTTP respecte la casse.
>

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromQueryString
    {
        [FunctionName("DocByIdFromQueryString")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{Query.id}",
                PartitionKey = "{Query.partitionKey}")] ToDoItem toDoItem,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.LogInformation($"ToDo item not found");
            }
            else
            {
                log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
            }
            return new OkResult();
        }
    }
}
```

<a id="http-trigger-look-up-id-from-route-data-c"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise des données de routage pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromRouteData
    {
        [FunctionName("DocByIdFromRouteData")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems/{partitionKey}/{id}")]HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                Id = "{id}",
                PartitionKey = "{partitionKey}")] ToDoItem toDoItem,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            if (toDoItem == null)
            {
                log.LogInformation($"ToDo item not found");
            }
            else
            {
                log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
            }
            return new OkResult();
        }
    }
}
```

<a id="http-trigger-look-up-id-from-route-data-using-sqlquery-c"></a>

### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire, utilisation de SqlQuery

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui récupère un document unique. Cette fonction est déclenchée par une requête HTTP qui utilise des données de routage pour spécifier l’ID à rechercher. Cet ID est utilisé pour récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

L’exemple montre comment utiliser une expression de liaison dans le paramètre `SqlQuery`. Vous pouvez valider des données de routage sur le paramètre `SqlQuery` comme indiqué, mais actuellement [vous ne pouvez pas valider des valeurs de chaînes de requête](https://github.com/Azure/azure-functions-host/issues/2554#issuecomment-392084583).

> [!NOTE]
> Si vous devez interroger uniquement à l’aide de l’ID, il est recommandé d’utiliser une recherche, comme dans les [exemples précédents](#http-trigger-look-up-id-from-query-string-c), car cela consomme moins d’[unités de requête](../cosmos-db/request-units.md). Les opérations de lecture à point (GET) sont [plus efficaces](../cosmos-db/optimize-cost-reads-writes.md) que les requêtes par ID.
>

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocByIdFromRouteDataUsingSqlQuery
    {
        [FunctionName("DocByIdFromRouteDataUsingSqlQuery")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = "todoitems2/{id}")]HttpRequest req,
            [CosmosDB("ToDoItems", "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "select * from ToDoItems r where r.id = {id}")]
                IEnumerable<ToDoItem> toDoItems,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.LogInformation(toDoItem.Description);
            }
            return new OkResult();
        }
    }
}
```

<a id="http-trigger-get-multiple-docs-using-sqlquery-c"></a>

### <a name="http-trigger-get-multiple-docs-using-sqlquery"></a>Déclencheur HTTP, obtenir plusieurs documents, utilisation de SqlQuery

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui récupère une liste de documents. Cette fonction est déclenchée par une requête HTTP. La requête est spécifiée dans la propriété d’attribut `SqlQuery`.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class DocsBySqlQuery
    {
        [FunctionName("DocsBySqlQuery")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
                SqlQuery = "SELECT top 2 * FROM c order by c._ts desc")]
                IEnumerable<ToDoItem> toDoItems,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            foreach (ToDoItem toDoItem in toDoItems)
            {
                log.LogInformation(toDoItem.Description);
            }
            return new OkResult();
        }
    }
}

```

<a id="http-trigger-get-multiple-docs-using-documentclient-c"></a>

### <a name="http-trigger-get-multiple-docs-using-documentclient"></a>Déclencheur HTTP, obtenir plusieurs documents, utilisation de DocumentClient

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui récupère une liste de documents. Cette fonction est déclenchée par une requête HTTP. Le code utilise une instance `DocumentClient` fournie par la liaisonAzure Cosmos DB pour lire une liste de documents. L’instance `DocumentClient` peut également être utilisée pour les opérations d’écriture.

> [!NOTE]
> Vous pouvez également utiliser l’interface [IDocumentClient](/dotnet/api/microsoft.azure.documents.idocumentclient?view=azure-dotnet) pour faciliter le test.

```cs
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using System;
using System.Linq;
using System.Threading.Tasks;

namespace CosmosDBSamplesV2
{
    public static class DocsByUsingDocumentClient
    {
        [FunctionName("DocsByUsingDocumentClient")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post",
                Route = null)]HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoItems",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            var searchterm = req.Query["searchterm"];
            if (string.IsNullOrWhiteSpace(searchterm))
            {
                return (ActionResult)new NotFoundResult();
            }

            Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");

            log.LogInformation($"Searching for: {searchterm}");

            IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
                .Where(p => p.Description.Contains(searchterm))
                .AsDocumentQuery();

            while (query.HasMoreResults)
            {
                foreach (ToDoItem result in await query.ExecuteNextAsync())
                {
                    log.LogInformation(result.Description);
                }
            }
            return new OkResult();
        }
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Cette section contient les exemples suivants :

* [Déclencheur de file d’attente, rechercher l’ID à partir de la chaîne](#queue-trigger-look-up-id-from-string-c-script)
* [Déclencheur de file d’attente, obtenir plusieurs documents, utilisation de SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-c-script)
* [Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête](#http-trigger-look-up-id-from-query-string-c-script)
* [Déclencheur HTTP, ID de recherche à partir des données de routage](#http-trigger-look-up-id-from-route-data-c-script)
* [Déclencheur HTTP, obtenir plusieurs documents, utilisation de SqlQuery](#http-trigger-get-multiple-docs-using-sqlquery-c-script)
* [Déclencheur HTTP, obtenir plusieurs documents, utilisation de DocumentClient](#http-trigger-get-multiple-docs-using-documentclient-c-script)

Les exemples de déclencheur HTTP font référence à un type `ToDoItem` simple :

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

<a id="queue-trigger-look-up-id-from-string-c-script"></a>

### <a name="queue-trigger-look-up-id-from-string"></a>Déclencheur de file d’attente, rechercher l’ID à partir de la chaîne

L’exemple suivant montre une liaison d’entrée Cosmos DB dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction lit un document unique et met à jour la valeur texte du document.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "inputDocument",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger}",
    "partitionKey": "{partition key value}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "in"
}
```
La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# :

```cs
    using System;

    // Change input document contents using Azure Cosmos DB input binding
    public static void Run(string myQueueItem, dynamic inputDocument)
    {
      inputDocument.text = "This has changed.";
    }
```

<a id="queue-trigger-get-multiple-docs-using-sqlquery-c-script"></a>

### <a name="queue-trigger-get-multiple-docs-using-sqlquery"></a>Déclencheur de file d’attente, obtenir plusieurs documents, utilisation de SqlQuery

L’exemple suivant montre une liaison d’entrée Azure Cosmos DB dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction récupère plusieurs documents spécifiés par une requête SQL, à l’aide d’un déclencheur de file d’attente pour personnaliser les paramètres de requête.

Le déclencheur de file d’attente fournit un paramètre `departmentId`. Un message de file d’attente de `{ "departmentId" : "Finance" }` retourne tous les enregistrements du service financier.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# :

```csharp
    public static void Run(QueuePayload myQueueItem, IEnumerable<dynamic> documents)
    {
        foreach (var doc in documents)
        {
            // operate on each document
        }
    }

    public class QueuePayload
    {
        public string departmentId { get; set; }
    }
```

<a id="http-trigger-look-up-id-from-query-string-c-script"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête

L’exemple suivant illustre une [fonction de script C#](functions-reference-csharp.md) qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise une chaîne de requête pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}",
      "PartitionKey" : "{Query.partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Voici le code Script C# :

```cs
using System.Net;
using Microsoft.Extensions.Logging;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
         log.LogInformation($"ToDo item not found");
    }
    else
    {
        log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-look-up-id-from-route-data-c-script"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire

L’exemple suivant illustre une [fonction de script C#](functions-reference-csharp.md) qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise des données de routage pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{partitionKeyValue}/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}",
      "PartitionKey": "{partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Voici le code Script C# :

```cs
using System.Net;
using Microsoft.Extensions.Logging;

public static HttpResponseMessage Run(HttpRequestMessage req, ToDoItem toDoItem, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    if (toDoItem == null)
    {
         log.LogInformation($"ToDo item not found");
    }
    else
    {
        log.LogInformation($"Found ToDo item, Description={toDoItem.Description}");
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-get-multiple-docs-using-sqlquery-c-script"></a>

### <a name="http-trigger-get-multiple-docs-using-sqlquery"></a>Déclencheur HTTP, obtenir plusieurs documents, utilisation de SqlQuery

L’exemple suivant illustre une [fonction de script C#](functions-reference-csharp.md) qui récupère une liste de documents. Cette fonction est déclenchée par une requête HTTP. La requête est spécifiée dans la propriété d’attribut `SqlQuery`.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "sqlQuery": "SELECT top 2 * FROM c order by c._ts desc"
    }
  ],
  "disabled": false
}
```

Voici le code Script C# :

```cs
using System.Net;
using Microsoft.Extensions.Logging;

public static HttpResponseMessage Run(HttpRequestMessage req, IEnumerable<ToDoItem> toDoItems, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    foreach (ToDoItem toDoItem in toDoItems)
    {
        log.LogInformation(toDoItem.Description);
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

<a id="http-trigger-get-multiple-docs-using-documentclient-c-script"></a>

### <a name="http-trigger-get-multiple-docs-using-documentclient"></a>Déclencheur HTTP, obtenir plusieurs documents, utilisation de DocumentClient

L’exemple suivant illustre une [fonction de script C#](functions-reference-csharp.md) qui récupère une liste de documents. Cette fonction est déclenchée par une requête HTTP. Le code utilise une instance `DocumentClient` fournie par la liaisonAzure Cosmos DB pour lire une liste de documents. L’instance `DocumentClient` peut également être utilisée pour les opérations d’écriture.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "client",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "inout"
    }
  ],
  "disabled": false
}
```

Voici le code Script C# :

```cs
#r "Microsoft.Azure.Documents.Client"

using System.Net;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;
using Microsoft.Extensions.Logging;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, DocumentClient client, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoItems", "Items");
    string searchterm = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "searchterm", true) == 0)
        .Value;

    if (searchterm == null)
    {
        return req.CreateResponse(HttpStatusCode.NotFound);
    }

    log.LogInformation($"Searching for word: {searchterm} using Uri: {collectionUri.ToString()}");
    IDocumentQuery<ToDoItem> query = client.CreateDocumentQuery<ToDoItem>(collectionUri)
        .Where(p => p.Description.Contains(searchterm))
        .AsDocumentQuery();

    while (query.HasMoreResults)
    {
        foreach (ToDoItem result in await query.ExecuteNextAsync())
        {
            log.LogInformation(result.Description);
        }
    }
    return req.CreateResponse(HttpStatusCode.OK);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Cette section contient les exemples suivants qui lisent un document unique en spécifiant une valeur d’ID à partir de diverses sources :

* [Déclencheur de file d’attente, rechercher l’ID à partir de JSON](#queue-trigger-look-up-id-from-json-javascript)
* [Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête](#http-trigger-look-up-id-from-query-string-javascript)
* [Déclencheur HTTP, ID de recherche à partir des données de routage](#http-trigger-look-up-id-from-route-data-javascript)
* [Déclencheur de file d’attente, obtenir plusieurs documents, utilisation de SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-javascript)

<a id="queue-trigger-look-up-id-from-json-javascript"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Déclencheur de file d’attente, rechercher l’ID à partir de JSON

L’exemple suivant montre une liaison d’entrée Cosmos DB dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction lit un document unique et met à jour la valeur texte du document.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "inputDocumentIn",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "in"
},
{
    "name": "inputDocumentOut",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
    // Change input document contents using Azure Cosmos DB input binding, using context.bindings.inputDocumentOut
    module.exports = function (context) {
    context.bindings.inputDocumentOut = context.bindings.inputDocumentIn;
    context.bindings.inputDocumentOut.text = "This was updated!";
    context.done();
    };
```

<a id="http-trigger-look-up-id-from-query-string-javascript"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête

L’exemple suivant illustre une [fonction JavaScript](functions-reference-node.md) qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise une chaîne de requête pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}",
      "PartitionKey": "{Query.partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

<a id="http-trigger-look-up-id-from-route-data-javascript"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire

L’exemple suivant illustre une [fonction JavaScript](functions-reference-node.md) qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise des données de routage pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{partitionKeyValue}/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "toDoItem",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}",
      "PartitionKey": "{partitionKeyValue}"
    }
  ],
  "disabled": false
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, req, toDoItem) {
    context.log('JavaScript queue trigger function processed work item');
    if (!toDoItem)
    {
        context.log("ToDo item not found");
    }
    else
    {
        context.log("Found ToDo item, Description=" + toDoItem.Description);
    }

    context.done();
};
```

<a id="queue-trigger-get-multiple-docs-using-sqlquery-javascript"></a>

### <a name="queue-trigger-get-multiple-docs-using-sqlquery"></a>Déclencheur de file d’attente, obtenir plusieurs documents, utilisation de SqlQuery

L’exemple suivant montre une liaison d’entrée Azure Cosmos DB dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction récupère plusieurs documents spécifiés par une requête SQL, à l’aide d’un déclencheur de file d’attente pour personnaliser les paramètres de requête.

Le déclencheur de file d’attente fournit un paramètre `departmentId`. Un message de file d’attente de `{ "departmentId" : "Finance" }` retourne tous les enregistrements du service financier.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
    module.exports = function (context, input) {
        var documents = context.bindings.documents;
        for (var i = 0; i < documents.length; i++) {
            var document = documents[i];
            // operate on each document
        }
        context.done();
    };
```

# <a name="python"></a>[Python](#tab/python)

Cette section contient les exemples suivants qui lisent un document unique en spécifiant une valeur d’ID à partir de diverses sources :

* [Déclencheur de file d’attente, rechercher l’ID à partir de JSON](#queue-trigger-look-up-id-from-json-python)
* [Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête](#http-trigger-look-up-id-from-query-string-python)
* [Déclencheur HTTP, ID de recherche à partir des données de routage](#http-trigger-look-up-id-from-route-data-python)
* [Déclencheur de file d’attente, obtenir plusieurs documents, utilisation de SqlQuery](#queue-trigger-get-multiple-docs-using-sqlquery-python)

<a id="queue-trigger-look-up-id-from-json-python"></a>

### <a name="queue-trigger-look-up-id-from-json"></a>Déclencheur de file d’attente, rechercher l’ID à partir de JSON

L’exemple suivant montre une liaison d’entrée Cosmos DB dans un fichier *function.json* et une [fonction Python](functions-reference-python.md) qui utilise la liaison. La fonction lit un document unique et met à jour la valeur texte du document.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "id" : "{queueTrigger_payload_property}",
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "in"
},
{
    "name": "$return",
    "type": "cosmosDB",
    "databaseName": "MyDatabase",
    "collectionName": "MyCollection",
    "createIfNotExists": false,
    "partitionKey": "{queueTrigger_payload_property}",
    "connectionStringSetting": "MyAccount_COSMOSDB",
    "direction": "out"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Python :

```python
import azure.functions as func


def main(queuemsg: func.QueueMessage, documents: func.DocumentList) -> func.Document:
    if documents:
        document = documents[0]
        document['text'] = 'This was updated!'
        return document
```

<a id="http-trigger-look-up-id-from-query-string-python"></a>

### <a name="http-trigger-look-up-id-from-query-string"></a>Déclencheur HTTP, rechercher l’ID à partir de la chaîne de requête

L’exemple suivant illustre une [fonction Python](functions-reference-python.md) qui récupère un document. La fonction est déclenchée par une requête HTTP qui utilise une chaîne de requête pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "todoitems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connectionStringSetting": "CosmosDBConnection",
      "direction": "in",
      "Id": "{Query.id}",
      "PartitionKey": "{Query.partitionKeyValue}"
    }
  ],
  "scriptFile": "__init__.py"
}
```

Voici le code Python :

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest, todoitems: func.DocumentList) -> str:
    if not todoitems:
        logging.warning("ToDo item not found")
    else:
        logging.info("Found ToDo item, Description=%s",
                     todoitems[0]['description'])

    return 'OK'
```

<a id="http-trigger-look-up-id-from-route-data-python"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire

L’exemple suivant illustre une [fonction Python](functions-reference-python.md) qui récupère un document. La fonction est déclenchée par une requête HTTP qui utilise des données de routage pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document `ToDoItem` à partir de la base de données et de la collection spécifiées.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": [
        "get",
        "post"
      ],
      "route":"todoitems/{partitionKeyValue}/{id}"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "type": "cosmosDB",
      "name": "todoitems",
      "databaseName": "ToDoItems",
      "collectionName": "Items",
      "connection": "CosmosDBConnection",
      "direction": "in",
      "Id": "{id}",
      "PartitionKey": "{partitionKeyValue}"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

Voici le code Python :

```python
import logging
import azure.functions as func


def main(req: func.HttpRequest, todoitems: func.DocumentList) -> str:
    if not todoitems:
        logging.warning("ToDo item not found")
    else:
        logging.info("Found ToDo item, Description=%s",
                     todoitems[0]['description'])
    return 'OK'
```

<a id="queue-trigger-get-multiple-docs-using-sqlquery-python"></a>

### <a name="queue-trigger-get-multiple-docs-using-sqlquery"></a>Déclencheur de file d’attente, obtenir plusieurs documents, utilisation de SqlQuery

L’exemple suivant montre une liaison d’entrée Azure Cosmos DB dans un fichier *function.json* et une [fonction Python](functions-reference-python.md) qui utilise la liaison. La fonction récupère plusieurs documents spécifiés par une requête SQL, à l’aide d’un déclencheur de file d’attente pour personnaliser les paramètres de requête.

Le déclencheur de file d’attente fournit un paramètre `departmentId`. Un message de file d’attente de `{ "departmentId" : "Finance" }` retourne tous les enregistrements du service financier.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "documents",
    "type": "cosmosDB",
    "direction": "in",
    "databaseName": "MyDb",
    "collectionName": "MyCollection",
    "sqlQuery": "SELECT * from c where c.departmentId = {departmentId}",
    "connectionStringSetting": "CosmosDBConnection"
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Python :

```python
import azure.functions as func

def main(queuemsg: func.QueueMessage, documents: func.DocumentList):
    for document in documents:
        # operate on each document
```

# <a name="java"></a>[Java](#tab/java)

Cette section contient les exemples suivants :

* [Déclencheur HTTP, rechercher l’ID dans la chaîne de requête - paramètre String](#http-trigger-look-up-id-from-query-string---string-parameter-java)
* [Déclencheur HTTP, rechercher l’ID dans la chaîne de requête - paramètre POJO](#http-trigger-look-up-id-from-query-string---pojo-parameter-java)
* [Déclencheur HTTP, ID de recherche à partir des données de routage](#http-trigger-look-up-id-from-route-data-java)
* [Déclencheur HTTP, ID de recherche à partir des données de routage, utilisation de SqlQuery](#http-trigger-look-up-id-from-route-data-using-sqlquery-java)
* [Déclencheur HTTP, obtenir plusieurs documents à partir des données de routage, utilisation de SqlQuery](#http-trigger-get-multiple-docs-from-route-data-using-sqlquery-java)

Les exemples font référence à un type `ToDoItem` simple :

```java
public class ToDoItem {

  private String id;
  private String description;

  public String getId() {
    return id;
  }

  public String getDescription() {
    return description;
  }

  @Override
  public String toString() {
    return "ToDoItem={id=" + id + ",description=" + description + "}";
  }
}
```

<a id="http-trigger-look-up-id-from-query-string---string-parameter-java"></a>

### <a name="http-trigger-look-up-id-from-query-string---string-parameter"></a>Déclencheur HTTP, recherche d’ID dans la chaîne de requête : paramètre String

L’exemple suivant illustre une fonction Java qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise une chaîne de requête pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document à partir de la base de données et de la collection spécifiées sous la forme d’une chaîne.

```java
public class DocByIdFromQueryString {

    @FunctionName("DocByIdFromQueryString")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              id = "{Query.id}",
              partitionKey = "{Query.partitionKeyValue}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            Optional<String> item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("String from the database is " + (item.isPresent() ? item.get() : null));

        // Convert and display
        if (!item.isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            // return JSON from Cosmos. Alternatively, we can parse the JSON string
            // and return an enriched JSON object.
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item.get())
                          .build();
        }
    }
}
 ```

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@CosmosDBInput` sur les paramètres de fonction dont la valeur proviendrait de Cosmos DB.  Vous pouvez utiliser cette annotation avec des types Java natifs, des objets POJO ou des valeurs Null à l’aide de `Optional<T>`.

<a id="http-trigger-look-up-id-from-query-string---pojo-parameter-java"></a>

### <a name="http-trigger-look-up-id-from-query-string---pojo-parameter"></a>Déclencheur HTTP, recherche d’ID dans la chaîne de requête : paramètre POJO

L’exemple suivant illustre une fonction Java qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise une chaîne de requête pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document à partir de la base de données et de la collection spécifiées. Le document est ensuite converti en une instance du POJO ```ToDoItem``` précédemment créé et passé en tant qu’argument à la fonction.

```java
public class DocByIdFromQueryStringPojo {

    @FunctionName("DocByIdFromQueryStringPojo")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS)
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              id = "{Query.id}",
              partitionKey = "{Query.partitionKeyValue}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            ToDoItem item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("Item from the database is " + item);

        // Convert and display
        if (item == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item)
                          .build();
        }
    }
}
 ```

<a id="http-trigger-look-up-id-from-route-data-java"></a>

### <a name="http-trigger-look-up-id-from-route-data"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire

L’exemple suivant illustre une fonction Java qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise un paramètre de routage pour spécifier l’ID et la valeur de clé de partition à rechercher. Cet ID et cette valeur de clé de partition permettent de récupérer un document à partir de la base de données et de la collection spécifiées, en le retournant comme ```Optional<String>```.

```java
public class DocByIdFromRoute {

    @FunctionName("DocByIdFromRoute")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "todoitems/{partitionKeyValue}/{id}")
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              id = "{id}",
              partitionKey = "{partitionKeyValue}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            Optional<String> item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("String from the database is " + (item.isPresent() ? item.get() : null));

        // Convert and display
        if (!item.isPresent()) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            // return JSON from Cosmos. Alternatively, we can parse the JSON string
            // and return an enriched JSON object.
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item.get())
                          .build();
        }
    }
}
 ```

 <a id="http-trigger-look-up-id-from-route-data-using-sqlquery-java"></a>

### <a name="http-trigger-look-up-id-from-route-data-using-sqlquery"></a>Déclencheur HTTP, rechercher l’ID à partir des données d’itinéraire, utilisation de SqlQuery

L’exemple suivant illustre une fonction Java qui récupère un document unique. La fonction est déclenchée par une requête HTTP qui utilise un paramètre de routage afin de spécifier l’ID à rechercher. Cet ID est utilisé pour récupérer un document à partir de la base de données et de la collection spécifiées, en convertissant le jeu de résultats en ```ToDoItem[]```, étant donné que de nombreux documents peuvent être retournés, en fonction des critères de la requête.

> [!NOTE]
> Si vous devez interroger uniquement à l’aide de l’ID, il est recommandé d’utiliser une recherche, comme dans les [exemples précédents](#http-trigger-look-up-id-from-query-string---pojo-parameter-java), car cela consomme moins d’[unités de requête](../cosmos-db/request-units.md). Les opérations de lecture à point (GET) sont [plus efficaces](../cosmos-db/optimize-cost-reads-writes.md) que les requêtes par ID.
>

```java
public class DocByIdFromRouteSqlQuery {

    @FunctionName("DocByIdFromRouteSqlQuery")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET, HttpMethod.POST},
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "todoitems2/{id}")
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              sqlQuery = "select * from Items r where r.id = {id}",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            ToDoItem[] item,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("Items from the database are " + item);

        // Convert and display
        if (item == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("Document not found.")
                          .build();
        }
        else {
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(item)
                          .build();
        }
    }
}
 ```

 <a id="http-trigger-get-multiple-docs-from-route-data-using-sqlquery-java"></a>

### <a name="http-trigger-get-multiple-docs-from-route-data-using-sqlquery"></a>Déclencheur HTTP, obtention de plusieurs documents à partir de données d’itinéraire, utilisation de SqlQuery

L’exemple suivant illustre une fonction Java qui récupère plusieurs documents. La fonction est déclenchée par une requête HTTP qui utilise un paramètre de routage ```desc``` afin de spécifier la chaîne à rechercher dans le champ ```description```. Le terme recherché est utilisé pour récupérer une collection de documents à partir de la base de données et de la collection spécifiées, en convertissant le jeu de résultats en ```ToDoItem[]``` et en le passant en tant qu’argument à la fonction.

```java
public class DocsFromRouteSqlQuery {

    @FunctionName("DocsFromRouteSqlQuery")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req",
              methods = {HttpMethod.GET},
              authLevel = AuthorizationLevel.ANONYMOUS,
              route = "todoitems3/{desc}")
            HttpRequestMessage<Optional<String>> request,
            @CosmosDBInput(name = "database",
              databaseName = "ToDoList",
              collectionName = "Items",
              sqlQuery = "select * from Items r where contains(r.description, {desc})",
              connectionStringSetting = "Cosmos_DB_Connection_String")
            ToDoItem[] items,
            final ExecutionContext context) {

        // Item list
        context.getLogger().info("Parameters are: " + request.getQueryParameters());
        context.getLogger().info("Number of items from the database is " + (items == null ? 0 : items.length));

        // Convert and display
        if (items == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST)
                          .body("No documents found.")
                          .build();
        }
        else {
            return request.createResponseBuilder(HttpStatus.OK)
                          .header("Content-Type", "application/json")
                          .body(items)
                          .build();
        }
    }
}
 ```

 ---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [CosmosDB](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/CosmosDBAttribute.cs).

Le constructeur de l’attribut accepte le nom de la base de données et le nom de la collection. Pour plus d’informations sur ces paramètres et d’autres propriétés que vous pouvez configurer, consultez [la section de configuration suivante](#configuration).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@CosmosDBOutput` sur des paramètres qui écrivent dans Cosmos DB. Le type de paramètre d’annotation doit être `OutputBinding<T>`, où `T` désigne un type Java natif ou un POJO.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `CosmosDB`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type**     | n/a | Cette propriété doit être définie sur `cosmosDB`.        |
|**direction**     | n/a | Cette propriété doit être définie sur `in`.         |
|**name**     | n/a | Nom du paramètre de liaison qui représente le document dans la fonction.  |
|**databaseName** |**DatabaseName** |Base de données contenant le document.        |
|**collectionName** |**CollectionName** | Nom de la collection qui contient le document. |
|**id**    | **Id** | ID du document à récupérer. Cette propriété prend en charge les [expressions de liaison](./functions-bindings-expressions-patterns.md). Ne définissez pas à la fois les propriétés `id` et **sqlQuery**. Si vous ne définissez aucune des deux, l’ensemble de la collection est récupéré. |
|**sqlQuery**  |**SqlQuery**  | Requête SQL Azure Cosmos DB utilisée pour récupérer plusieurs documents. La propriété prend en charge les liaisons d’exécution, comme dans cet exemple : `SELECT * FROM c where c.departmentId = {departmentId}`. Ne définissez pas à la fois les propriétés `id` et `sqlQuery`. Si vous ne définissez aucune des deux, l’ensemble de la collection est récupéré.|
|**connectionStringSetting**     |**ConnectionStringSetting**|Nom du paramètre d’application contenant votre chaîne de connexion Azure Cosmos DB. |
|**partitionKey**|**PartitionKey**|Spécifie la valeur de la clé de partition pour la recherche. Peut inclure des paramètres de liaison. Requis pour les recherches dans les collections [partitionnées](../cosmos-db/partitioning-overview.md#logical-partitions).|
|**preferredLocations**| **PreferredLocations**| (Facultatif) Définit les endroits par défaut (régions) des comptes de base de données géorépliqués dans le service Azure Cosmos DB. Les valeurs doivent être séparées par des virgules. Par exemple, « USA Est, USA Centre Sud, Europe Nord ». |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Lorsque la fonction se termine correctement, toutes les modifications apportées au document d’entrée par le biais des paramètres d’entrée nommés sont automatiquement conservées.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Lorsque la fonction se termine correctement, toutes les modifications apportées au document d’entrée par le biais des paramètres d’entrée nommés sont automatiquement conservées.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les mises à jour ne sont pas effectuées automatiquement une fois la fonction terminée. Utilisez plutôt `context.bindings.<documentName>In` et `context.bindings.<documentName>Out` pour effectuer les mises à jour. Consultez l’exemple JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les données sont mises à la disposition de la fonction via un paramètre `DocumentList`. Les modifications apportées au document ne sont pas automatiquement conservées.

# <a name="java"></a>[Java](#tab/java)

À partir de la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), l’annotation [@CosmosDBInput](/java/api/com.microsoft.azure.functions.annotation.cosmosdbinput) expose les données de Cosmos DB à la fonction. Vous pouvez utiliser cette annotation avec des types Java natifs, des objets POJO ou des valeurs Null à l’aide de `Optional<T>`.

---

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction lors de la création ou de la modification d'un document Azure Cosmos DB (Déclencheur)](./functions-bindings-cosmosdb-v2-trigger.md)
- [Enregistrer les modifications apportées à un document Azure Cosmos DB (liaison de sortie)](./functions-bindings-cosmosdb-v2-output.md)