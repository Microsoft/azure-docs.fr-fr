---
title: Liaisons d’entrée de stockage de table Azure pour Azure Functions
description: Découvrez comment utiliser les liaisons d’entrée de stockage de table Azure dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 20dc6cde9cce6a9d57047940a38adb5cf004ae6a
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347674"
---
# <a name="azure-table-storage-input-bindings-for-azure-functions"></a>Liaisons d’entrée de stockage de table Azure pour Azure Functions

La liaison d’entrée de stockage de table Azure permet de lire une table dans un compte de stockage Azure.

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

### <a name="one-entity"></a>Une entité

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui lit une ligne de table unique. Pour chaque message envoyé à la file d'attente, la fonction sera déclenchée.

La valeur de clé de ligne "{queueTrigger}" indique que la clé de ligne provient de la chaîne de message de file d’attente.

```csharp
public class TableStorage
{
    public class MyPoco
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public string Text { get; set; }
    }

    [FunctionName("TableInput")]
    public static void TableInput(
        [QueueTrigger("table-items")] string input, 
        [Table("MyTable", "MyPartition", "{queueTrigger}")] MyPoco poco, 
        ILogger log)
    {
        log.LogInformation($"PK={poco.PartitionKey}, RK={poco.RowKey}, Text={poco.Text}");
    }
}
```

### <a name="cloudtable"></a>CloudTable

`CloudTable` est uniquement pris en charge dans les [fonctions v2 et versions ultérieures](functions-versions.md).

Utilisez un paramètre de méthode `CloudTable` pour lire la table à l’aide du kit de développement logiciel (SDK) stockage Azure. Voici un exemple d’une fonction qui interroge une table du journal Azure Functions :

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.Cosmos.Table;
using System;
using System.Threading.Tasks;

namespace FunctionAppCloudTable2
{
    public class LogEntity : TableEntity
    {
        public string OriginalName { get; set; }
    }
    public static class CloudTableDemo
    {
        [FunctionName("CloudTableDemo")]
        public static async Task Run(
            [TimerTrigger("0 */1 * * * *")] TimerInfo myTimer, 
            [Table("AzureWebJobsHostLogscommon")] CloudTable cloudTable,
            ILogger log)
        {
            log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

            TableQuery<LogEntity> rangeQuery = new TableQuery<LogEntity>().Where(
                TableQuery.CombineFilters(
                    TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, 
                        "FD2"),
                    TableOperators.And,
                    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, 
                        "t")));

            // Execute the query and loop through the results
            foreach (LogEntity entity in 
                await cloudTable.ExecuteQuerySegmentedAsync(rangeQuery, null))
            {
                log.LogInformation(
                    $"{entity.PartitionKey}\t{entity.RowKey}\t{entity.Timestamp}\t{entity.OriginalName}");
            }
        }
    }
}
```

Pour plus d’informations sur l’utilisation de CloudTable, consultez [Bien démarrer avec Stockage Table Azure](../cosmos-db/tutorial-develop-table-dotnet.md).

Si vous essayez de lier à `CloudTable` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

### <a name="iqueryable"></a>IQueryable

`IQueryable` est pris en charge uniquement dans le [runtime Functions v1](functions-versions.md).

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui lit plusieurs lignes de table où la classe `MyPoco` dérive de `TableEntity`.

```csharp
public class TableStorage
{
    public class MyPoco : TableEntity
    {
        public string Text { get; set; }
    }

    [FunctionName("TableInput")]
    public static void TableInput(
        [QueueTrigger("table-items")] string input, 
        [Table("MyTable", "MyPartition")] IQueryable<MyPoco> pocos, 
        ILogger log)
    {
        foreach (MyPoco poco in pocos)
        {
            log.LogInformation($"PK={poco.PartitionKey}, RK={poco.RowKey}, Text={poco.Text}");
        }
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

### <a name="one-entity"></a>Une entité

L’exemple suivant montre une liaison d’entrée de table dans un fichier *function.json* et un code [Script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction utilise un déclencheur de file d’attente pour lire une seule ligne du tableau. 

Le fichier *function.json* spécifie une propriété `partitionKey` et une propriété `rowKey`. La valeur `rowKey` "{queueTrigger}" indique que la clé de ligne provient de la chaîne de message de file d’attente.

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# :

```csharp
public static void Run(string myQueueItem, Person personEntity, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    log.LogInformation($"Name in Person entity: {personEntity.Name}");
}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}
```

### <a name="cloudtable"></a>CloudTable

`IQueryable` n’est pas pris en charge dans le runtime Functions pour les [versions 2.x et ultérieures](functions-versions.md). Une alternative consiste à utiliser un paramètre de méthode `CloudTable` pour lire la table en utilisant le kit SDK Stockage Azure. Voici un exemple d’une fonction qui interroge une table du journal Azure Functions :

```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */1 * * * *"
    },
    {
      "name": "cloudTable",
      "type": "table",
      "connection": "AzureWebJobsStorage",
      "tableName": "AzureWebJobsHostLogscommon",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;
using System;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

public static async Task Run(TimerInfo myTimer, CloudTable cloudTable, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

    TableQuery<LogEntity> rangeQuery = new TableQuery<LogEntity>().Where(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, 
            "FD2"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, 
            "a")));

    // Execute the query and loop through the results
    foreach (LogEntity entity in 
    await cloudTable.ExecuteQuerySegmentedAsync(rangeQuery, null))
    {
        log.LogInformation(
            $"{entity.PartitionKey}\t{entity.RowKey}\t{entity.Timestamp}\t{entity.OriginalName}");
    }
}

public class LogEntity : TableEntity
{
    public string OriginalName { get; set; }
}
```

Pour plus d’informations sur l’utilisation de CloudTable, consultez [Bien démarrer avec Stockage Table Azure](../cosmos-db/tutorial-develop-table-dotnet.md).

Si vous essayez de lier à `CloudTable` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

### <a name="iqueryable"></a>IQueryable

L’exemple suivant montre une liaison d’entrée de table dans un fichier *function.json* et un code [Script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction lit les entités d’une clé de partition qui est spécifiée dans un message de file d’attente.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "tableBinding",
      "type": "table",
      "connection": "MyStorageConnectionAppSetting",
      "tableName": "Person",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

La section [configuration](#configuration) décrit ces propriétés.

Le code Script C# ajoute une référence au kit SDK Stockage Azure afin que le type d’entité puisse dériver de `TableEntity` :

```csharp
#r "Microsoft.WindowsAzure.Storage"
using Microsoft.WindowsAzure.Storage.Table;
using Microsoft.Extensions.Logging;

public static void Run(string myQueueItem, IQueryable<Person> tableBinding, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    foreach (Person person in tableBinding.Where(p => p.PartitionKey == myQueueItem).ToList())
    {
        log.LogInformation($"Name: {person.Name}");
    }
}

public class Person : TableEntity
{
    public string Name { get; set; }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison d’entrée de table dans un fichier *function.json* et un [code JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction utilise un déclencheur de file d’attente pour lire une seule ligne du tableau. 

Le fichier *function.json* spécifie une propriété `partitionKey` et une propriété `rowKey`. La valeur `rowKey` "{queueTrigger}" indique que la clé de ligne provient de la chaîne de message de file d’attente.

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "personEntity",
      "type": "table",
      "tableName": "Person",
      "partitionKey": "Test",
      "rowKey": "{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);
    context.log('Person entity name: ' + context.bindings.personEntity.Name);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

Ligne de table unique 

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "messageJSON",
      "type": "table",
      "tableName": "messages",
      "partitionKey": "message",
      "rowKey": "{id}",
      "connection": "AzureWebJobsStorage",
      "direction": "in"
    },
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ],
      "route": "messages/{id}"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ],
  "disabled": false
}
```

```python
import json

import azure.functions as func

def main(req: func.HttpRequest, messageJSON) -> func.HttpResponse:

    message = json.loads(messageJSON)
    return func.HttpResponse(f"Table row: {messageJSON}")
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant illustre une fonction déclenchée par HTTP qui retourne une liste d’objets Person qui se trouvent dans une partition spécifiée du stockage de table. Dans l’exemple, la clé de partition est extraite de l’itinéraire http, tandis que le nom de table (tableName) et la connexion proviennent des paramètres de fonction. 

```java
public class Person {
    private String PartitionKey;
    private String RowKey;
    private String Name;

    public String getPartitionKey() { return this.PartitionKey; }
    public void setPartitionKey(String key) { this.PartitionKey = key; }
    public String getRowKey() { return this.RowKey; }
    public void setRowKey(String key) { this.RowKey = key; }
    public String getName() { return this.Name; }
    public void setName(String name) { this.Name = name; }
}

@FunctionName("getPersonsByPartitionKey")
public Person[] get(
        @HttpTrigger(name = "getPersons", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.FUNCTION, route="persons/{partitionKey}") HttpRequestMessage<Optional<String>> request,
        @BindingName("partitionKey") String partitionKey,
        @TableInput(name="persons", partitionKey="{partitionKey}", tableName="%MyTableName%", connection="MyConnectionString") Person[] persons,
        final ExecutionContext context) {

    context.getLogger().info("Got query for person related to persons with partition key: " + partitionKey);

    return persons;
}
```

L’annotation TableInput peut également extraire les liaisons du corps JSON de la requête, comme le montre l’exemple suivant.

```java
@FunctionName("GetPersonsByKeysFromRequest")
public HttpResponseMessage get(
        @HttpTrigger(name = "getPerson", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.FUNCTION, route="query") HttpRequestMessage<Optional<String>> request,
        @TableInput(name="persons", partitionKey="{partitionKey}", rowKey = "{rowKey}", tableName="%MyTableName%", connection="MyConnectionString") Person person,
        final ExecutionContext context) {

    if (person == null) {
        return request.createResponseBuilder(HttpStatus.NOT_FOUND)
                    .body("Person not found.")
                    .build();
    }

    return request.createResponseBuilder(HttpStatus.OK)
                    .header("Content-Type", "application/json")
                    .body(person)
                    .build();
}
```

Les exemples suivants utilisent le filtre pour interroger des personnes portant un nom spécifique dans une table Azure, et limiter le nombre de correspondances possibles à 10 résultats.

```java
@FunctionName("getPersonsByName")
public Person[] get(
        @HttpTrigger(name = "getPersons", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.FUNCTION, route="filter/{name}") HttpRequestMessage<Optional<String>> request,
        @BindingName("name") String name,
        @TableInput(name="persons", filter="Name eq '{name}'", take = "10", tableName="%MyTableName%", connection="MyConnectionString") Person[] persons,
        final ExecutionContext context) {

    context.getLogger().info("Got query for person related to persons with name: " + name);

    return persons;
}
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

 Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez les attributs suivants pour configurer un déclencheur d’entrée de table :

* [TableAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables/TableAttribute.cs)

  Le constructeur de l’attribut prend le nom de la table, la clé de partition et la clé de ligne. L’attribut peut être utilisé sur un paramètre `out` ou sur la valeur de retour de la fonction, comme dans l’exemple suivant :

  ```csharp
  [FunctionName("TableInput")]
  public static void Run(
      [QueueTrigger("table-items")] string input, 
      [Table("MyTable", "Http", "{queueTrigger}")] MyPoco poco, 
      ILogger log)
  {
      ...
  }
  ```

  Vous pouvez définir la propriété `Connection` pour spécifier le compte de stockage à utiliser, comme indiqué dans l’exemple suivant :

  ```csharp
  [FunctionName("TableInput")]
  public static void Run(
      [QueueTrigger("table-items")] string input, 
      [Table("MyTable", "Http", "{queueTrigger}", Connection = "StorageConnectionAppSetting")] MyPoco poco, 
      ILogger log)
  {
      ...
  }
  ```

  Pour obtenir un exemple complet, consultez l' C# exemple.

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Fournit une autre manière de spécifier le compte de stockage à utiliser. Le constructeur prend le nom d’un paramètre d’application comportant une chaîne de connexion de stockage. L’attribut peut être appliqué au niveau du paramètre, de la méthode ou de la classe. L’exemple suivant montre le niveau de la classe et celui de la méthode :

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("TableInput")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

Le compte de stockage à utiliser est déterminé dans l’ordre suivant :

* La propriété `Connection` de l’attribut `Table`.
* L’attribut `StorageAccount` appliqué au même paramètre que l’attribut `Table`.
* L’attribut `StorageAccount` appliqué à la fonction.
* L’attribut `StorageAccount` appliqué à la classe.
* Le compte de stockage par défaut pour l’application de fonction (paramètre d’application « AzureWebJobsStorage »).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@TableInput` sur les paramètres dont la valeur proviendrait du Stockage Table.  Vous pouvez utiliser cette annotation avec des types Java natifs, des objets POJO ou des valeurs Null à l’aide de `Optional<T>`.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `Table`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Cette propriété doit être définie sur `table`. Cette propriété est définie automatiquement lorsque vous créez la liaison dans le portail Azure.|
|**direction** | n/a | Cette propriété doit être définie sur `in`. Cette propriété est définie automatiquement lorsque vous créez la liaison dans le portail Azure. |
|**name** | n/a | Nom de la variable qui représente la table ou l’entité dans le code de la fonction. | 
|**tableName** | **TableName** | Nom de la table.| 
|**partitionKey** | **PartitionKey** |facultatif. Clé de partition de l’entité de table à lire. Consultez la section [utilisation](#usage) pour obtenir des conseils sur l’utilisation de cette propriété.| 
|**rowKey** |**RowKey** | facultatif. Clé de ligne de l’entité de table à lire. Consultez la section [utilisation](#usage) pour obtenir des conseils sur l’utilisation de cette propriété.| 
|**take** |**Take** | facultatif. Nombre maximal d’entités à lire en JavaScript. Consultez la section [utilisation](#usage) pour obtenir des conseils sur l’utilisation de cette propriété.| 
|**filter** |**Filter** | facultatif. Expression de filtre OData pour l’entrée de table dans JavaScript. Consultez la section [utilisation](#usage) pour obtenir des conseils sur l’utilisation de cette propriété.| 
|**connection** |**Connection** | Nom d’un paramètre d’application comportant la chaîne de connexion de stockage à utiliser pour cette liaison. Le paramètre peut être le nom d’un paramètre d’application préfixé « AzureWebJobs » ou un nom de chaîne de connexion. Par exemple, si le nom de votre paramètre est « AzureWebJobsMyStorage », vous pouvez spécifier « MyStorage » ici. Le runtime Functions recherche automatiquement un paramètre d’application nommé « AzureWebJobsMyStorage ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion de stockage par défaut dans le paramètre d’application nommé `AzureWebJobsStorage`.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

* **Lire une ligne dans**

  Définissez `partitionKey` et `rowKey`. Accédez aux données de la table à l’aide d’un paramètre de méthode `T <paramName>`. Dans Script C#, `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. `T` est généralement un type qui implémente `ITableEntity` ou est dérivé de `TableEntity`. Les propriétés `filter` et `take` ne sont pas utilisées dans ce scénario.

* **Lire une ou plusieurs lignes**

  Accédez aux données de la table à l’aide d’un paramètre de méthode `IQueryable<T> <paramName>`. Dans Script C#, `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. `T` doit être généralement un type qui implémente `ITableEntity` ou est dérivé de `TableEntity`. Vous pouvez utiliser les méthodes `IQueryable` pour effectuer le filtrage voulu. Les propriétés `partitionKey`, `rowKey`, `filter` et `take` ne sont pas utilisées dans ce scénario.  

  > [!NOTE]
  > `IQueryable` n’est pas pris en charge dans le [runtime Functions v2](functions-versions.md). Une alternative consiste à utiliser un [paramètre de méthode CloudTable paramName](https://stackoverflow.com/questions/48922485/binding-to-table-storage-in-v2-azure-functions-using-cloudtable) pour lire la table à l’aide du SDK Stockage Azure. Si vous essayez de lier à `CloudTable` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

* **Lire une ligne dans**

  Définissez `partitionKey` et `rowKey`. Accédez aux données de la table à l’aide d’un paramètre de méthode `T <paramName>`. Dans Script C#, `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. `T` est généralement un type qui implémente `ITableEntity` ou est dérivé de `TableEntity`. Les propriétés `filter` et `take` ne sont pas utilisées dans ce scénario.

* **Lire une ou plusieurs lignes**

  Accédez aux données de la table à l’aide d’un paramètre de méthode `IQueryable<T> <paramName>`. Dans Script C#, `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. `T` doit être généralement un type qui implémente `ITableEntity` ou est dérivé de `TableEntity`. Vous pouvez utiliser les méthodes `IQueryable` pour effectuer le filtrage voulu. Les propriétés `partitionKey`, `rowKey`, `filter` et `take` ne sont pas utilisées dans ce scénario.  

  > [!NOTE]
  > `IQueryable` n’est pas pris en charge dans le [runtime Functions v2](functions-versions.md). Une alternative consiste à utiliser un [paramètre de méthode CloudTable paramName](https://stackoverflow.com/questions/48922485/binding-to-table-storage-in-v2-azure-functions-using-cloudtable) pour lire la table à l’aide du SDK Stockage Azure. Si vous essayez de lier à `CloudTable` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Définissez les propriétés `filter` et `take`. Ne définissez pas `partitionKey` ni `rowKey`. Accédez à l’entité (ou les entités) de table d’entrée à l’aide de `context.bindings.<BINDING_NAME>`. Les objets désérialisés ont des propriétés `RowKey` et `PartitionKey`.

# <a name="python"></a>[Python](#tab/python)

Les données de table sont passées à la fonction sous la forme d’une chaîne JSON. Désérialisez le message en appelant `json.loads` comme dans l’[exemple](#example) d’entrée.

# <a name="java"></a>[Java](#tab/java)

L’attribut [TableInput](/java/api/com.microsoft.azure.functions.annotation.tableinput) vous donne accès à la ligne de table qui ayant déclenché la fonction.

---

## <a name="next-steps"></a>Étapes suivantes

* [Écrire des données de stockage de table à partir d’une fonction](./functions-bindings-storage-table-output.md)
