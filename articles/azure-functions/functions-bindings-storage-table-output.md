---
title: Liaisons de sortie de stockage de table Azure pour Azure Functions
description: Apprenez à utiliser les liaisons de sortie de stockage de table Azure dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: ec857db64529a27db7412c61f8f09c66f8a76363
ms.sourcegitcommit: 93329b2fcdb9b4091dbd632ee031801f74beb05b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92098142"
---
# <a name="azure-table-storage-output-bindings-for-azure-functions"></a>Liaisons de sortie de stockage de table Azure pour Azure Functions

Utilisez une liaison de sortie de stockage de table Azure pour écrire des entités dans une table d’un compte de Stockage Azure.

> [!NOTE]
> Cette liaison de sortie ne prend pas en charge la mise à jour d’entités existantes. Utilisez l’opération `TableOperation.Replace`[à partir du Kit de développement logiciel (SDK) Stockage Azure](../cosmos-db/tutorial-develop-table-dotnet.md#delete-an-entity) pour mettre à jour une entité existante.

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant illustre une [fonction C#](functions-dotnet-class-library.md) qui utilise un déclencheur HTTP pour écrire une ligne de table.

```csharp
public class TableStorage
{
    public class MyPoco
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public string Text { get; set; }
    }

    [FunctionName("TableOutput")]
    [return: Table("MyTable")]
    public static MyPoco TableOutput([HttpTrigger] dynamic input, ILogger log)
    {
        log.LogInformation($"C# http trigger function processed: {input.Text}");
        return new MyPoco { PartitionKey = "Http", RowKey = Guid.NewGuid().ToString(), Text = input.Text };
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de sortie de table dans un fichier *function.json* et un code [Script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction écrit plusieurs entités de table.

Voici le fichier *function.json*  :

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# :

```csharp
public static void Run(string input, ICollector<Person> tableBinding, ILogger log)
{
    for (int i = 1; i < 10; i++)
        {
            log.LogInformation($"Adding Person entity {i}");
            tableBinding.Add(
                new Person() { 
                    PartitionKey = "Test", 
                    RowKey = i.ToString(), 
                    Name = "Name" + i.ToString() }
                );
        }

}

public class Person
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public string Name { get; set; }
}

```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de sortie de table dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction écrit plusieurs entités de table.

Voici le fichier *function.json*  :

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "manualTrigger",
      "direction": "in"
    },
    {
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting",
      "name": "tableBinding",
      "type": "table",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function (context) {

    context.bindings.tableBinding = [];

    for (var i = 1; i < 10; i++) {
        context.bindings.tableBinding.push({
            PartitionKey: "Test",
            RowKey: i.toString(),
            Name: "Name " + i
        });
    }

    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre comment utiliser la liaison de sortie de stockage de table. La liaison `table` est configurée dans le fichier *function.json* en assignant des valeurs à `name`, `tableName`, `partitionKey`et `connection` :

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "message",
      "type": "table",
      "tableName": "messages",
      "partitionKey": "message",
      "connection": "AzureWebJobsStorage",
      "direction": "out"
    },
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
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

La fonction suivante génère un UUI unique pour la valeur `rowKey` et conserve le message dans le stockage de table.

```python
import logging
import uuid
import json

import azure.functions as func

def main(req: func.HttpRequest, message: func.Out[str]) -> func.HttpResponse:

    rowKey = str(uuid.uuid4())

    data = {
        "Name": "Output binding message",
        "PartitionKey": "message",
        "RowKey": rowKey
    }

    message.set(json.dumps(data))

    return func.HttpResponse(f"Message created with the rowKey: {rowKey}")
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant illustre une fonction Java utilisant un déclencheur HTTP pour écrire une seule ligne de table.

```java
public class Person {
    private String PartitionKey;
    private String RowKey;
    private String Name;

    public String getPartitionKey() {return this.PartitionKey;}
    public void setPartitionKey(String key) {this.PartitionKey = key; }
    public String getRowKey() {return this.RowKey;}
    public void setRowKey(String key) {this.RowKey = key; }
    public String getName() {return this.Name;}
    public void setName(String name) {this.Name = name; }
}

public class AddPerson {

    @FunctionName("addPerson")
    public HttpResponseMessage get(
            @HttpTrigger(name = "postPerson", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION, route="persons/{partitionKey}/{rowKey}") HttpRequestMessage<Optional<Person>> request,
            @BindingName("partitionKey") String partitionKey,
            @BindingName("rowKey") String rowKey,
            @TableOutput(name="person", partitionKey="{partitionKey}", rowKey = "{rowKey}", tableName="%MyTableName%", connection="MyConnectionString") OutputBinding<Person> person,
            final ExecutionContext context) {

        Person outPerson = new Person();
        outPerson.setPartitionKey(partitionKey);
        outPerson.setRowKey(rowKey);
        outPerson.setName(request.getBody().get().getName());

        person.setValue(outPerson);

        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(outPerson)
                        .build();
    }
}
```

L’exemple suivant illustre une fonction Java utilisant un déclencheur HTTP pour écrire plusieurs lignes de table.

```java
public class Person {
    private String PartitionKey;
    private String RowKey;
    private String Name;

    public String getPartitionKey() {return this.PartitionKey;}
    public void setPartitionKey(String key) {this.PartitionKey = key; }
    public String getRowKey() {return this.RowKey;}
    public void setRowKey(String key) {this.RowKey = key; }
    public String getName() {return this.Name;}
    public void setName(String name) {this.Name = name; }
}

public class AddPersons {

    @FunctionName("addPersons")
    public HttpResponseMessage get(
            @HttpTrigger(name = "postPersons", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION, route="persons/") HttpRequestMessage<Optional<Person[]>> request,
            @TableOutput(name="person", tableName="%MyTableName%", connection="MyConnectionString") OutputBinding<Person[]> persons,
            final ExecutionContext context) {

        persons.setValue(request.getBody().get());

        return request.createResponseBuilder(HttpStatus.OK)
                        .header("Content-Type", "application/json")
                        .body(request.getBody().get())
                        .build();
    }
}
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [TableAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables/TableAttribute.cs).

Le constructeur de l’attribut prend le nom de la table. L’attribut peut être utilisé sur un paramètre `out` ou sur la valeur de retour de la fonction, comme dans l’exemple suivant :

```csharp
[FunctionName("TableOutput")]
[return: Table("MyTable")]
public static MyPoco TableOutput(
    [HttpTrigger] dynamic input, 
    ILogger log)
{
    ...
}
```

Vous pouvez définir la propriété `Connection` pour spécifier le compte de stockage à utiliser, comme indiqué dans l’exemple suivant :

```csharp
[FunctionName("TableOutput")]
[return: Table("MyTable", Connection = "StorageConnectionAppSetting")]
public static MyPoco TableOutput(
    [HttpTrigger] dynamic input, 
    ILogger log)
{
    ...
}
```

Pour obtenir un exemple complet, consultez l'[Exemple C#](#example).

Vous pouvez utiliser l’attribut `StorageAccount` pour spécifier le compte de stockage au niveau de la classe, de la méthode ou du paramètre. Pour plus d’informations, consultez [Entrée - attributs](./functions-bindings-storage-table-input.md#attributes-and-annotations).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

Dans la [bibliothèque runtime de fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation [TableOutput](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/annotation/TableOutput.java/) sur les paramètres pour écrire des valeurs dans le stockage de table.

Voir l’[exemple pour plus de détails](#example).

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `Table`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Cette propriété doit être définie sur `table`. Cette propriété est définie automatiquement lorsque vous créez la liaison dans le portail Azure.|
|**direction** | n/a | Cette propriété doit être définie sur `out`. Cette propriété est définie automatiquement lorsque vous créez la liaison dans le portail Azure. |
|**name** | n/a | Nom de variable utilisé dans le code de la fonction qui représente la table ou l’entité. La valeur doit être `$return` pour faire référence à la valeur de retour de la fonction.| 
|**tableName** |**TableName** | Nom de la table.| 
|**partitionKey** |**PartitionKey** | Clé de partition de l’entité de table à écrire. Consultez la section [utilisation](#usage) pour obtenir des conseils sur l’utilisation de cette propriété.| 
|**rowKey** |**RowKey** | Clé de ligne de l’entité de table à écrire. Consultez la section [utilisation](#usage) pour obtenir des conseils sur l’utilisation de cette propriété.| 
|**connection** |**Connection** | Nom d’un paramètre d’application comportant la chaîne de connexion de stockage à utiliser pour cette liaison. Si le nom du paramètre d’application commence par « AzureWebJobs », vous ne pouvez spécifier que le reste du nom ici. Par exemple, si vous définissez `connection` sur « MonStockage », le runtime Functions recherche un paramètre d’application qui nommé « MonStockage ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion de stockage par défaut dans le paramètre d’application nommé `AzureWebJobsStorage`.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Accédez à l’entité de table de sortie à l’aide d’un paramètre de méthode `ICollector<T> paramName` ou `IAsyncCollector<T> paramName`, où `T` contient les propriétés `PartitionKey` et `RowKey`. Ces propriétés sont souvent accompagnées de l’implémentation de `ITableEntity` ou de l’héritage de `TableEntity`.

Une autre solution consiste à utiliser un paramètre de méthode `CloudTable` pour écrire dans la table en utilisant le Kit de développement logiciel (SDK) Stockage Azure. Si vous essayez de lier à `CloudTable` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Accédez à l’entité de table de sortie à l’aide d’un paramètre de méthode `ICollector<T> paramName` ou `IAsyncCollector<T> paramName`, où `T` contient les propriétés `PartitionKey` et `RowKey`. Ces propriétés sont souvent accompagnées de l’implémentation de `ITableEntity` ou de l’héritage de `TableEntity`. La valeur `paramName` est spécifiée dans la propriété `name` de *function.json* .

Une autre solution consiste à utiliser un paramètre de méthode `CloudTable` pour écrire dans la table en utilisant le Kit de développement logiciel (SDK) Stockage Azure. Si vous essayez de lier à `CloudTable` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](./functions-bindings-storage-table.md#azure-storage-sdk-version-in-functions-1x).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Accédez à l’événement de sortie à l’aide de `context.bindings.<name>` où `<name>` est la valeur spécifiée dans la propriété `name` de *function.json* .

# <a name="python"></a>[Python](#tab/python)

Il existe deux options pour générer en sortie un message de ligne de stockage de table à partir d’une fonction :

- **Valeur de retour**  : Définissez la propriété `name` dans *function.json* sur `$return`. Avec cette configuration, la valeur de retour de la fonction est conservée en tant que ligne de stockage de table.

- **Impératif**  : Passez une valeur à la méthode [set](/python/api/azure-functions/azure.functions.out?view=azure-python&preserve-view=true#set-val--t-----none) du paramètre déclaré en tant que type [Out](/python/api/azure-functions/azure.functions.out?view=azure-python&preserve-view=true). La valeur transmise à `set` est conservée en tant que message d’Event Hub.

# <a name="java"></a>[Java](#tab/java)

Il existe deux options pour produire en sortie une ligne de stockage de table à partir d’une fonction en utilisant l’annotation [TableStorageOutput](/java/api/com.microsoft.azure.functions.annotation.tableoutput?view=azure-java-stablet&preserve-view=true) :

- **Valeur de retour**  : En appliquant l’annotation à la fonction elle-même, la valeur de retour de la fonction est conservée sous la forme d’une ligne de stockage de table.

- **Impératif**  : Pour définir explicitement la valeur du message, appliquez l’annotation à un paramètre spécifique du type [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding), où `T` inclut les propriétés `PartitionKey` et `RowKey`. Ces propriétés sont souvent accompagnées de l’implémentation de `ITableEntity` ou de l’héritage de `TableEntity`.

---

## <a name="exceptions-and-return-codes"></a>Exceptions et codes de retour

| Liaison | Informations de référence |
|---|---|
| Table de charge de travail | [Codes d’erreur de table](/rest/api/storageservices/fileservices/table-service-error-codes) |
| Objet blob, Table, File d’attente | [Codes d’erreur de stockage](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Objet blob, Table, File d’attente | [Dépannage](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les déclencheurs et les liaisons Azure Functions](functions-triggers-bindings.md)
