---
title: Liaison de sortie de Stockage File d’attente Azure pour Azure Functions
description: Apprenez à créer des messages de Stockage File d’attente Azure dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/18/2020
ms.author: cshoe
ms.custom: devx-track-csharp, cc996988-fb4f-47, devx-track-python
ms.openlocfilehash: 087073437fe9d6159422799c04ce095c0aae5eca
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96001250"
---
# <a name="azure-queue-storage-output-bindings-for-azure-functions"></a>Liaisons de sortie de Stockage File d’attente Azure pour Azure Functions

La solution Azure Functions peut créer des messages de Stockage File d’attente Azure en configurant une liaison de sortie.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](./functions-bindings-storage-queue.md).

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre un code [de fonction C#](functions-dotnet-class-library.md) qui crée un message de file d’attente pour chaque requête HTTP reçue.

```csharp
[StorageAccount("MyStorageConnectionAppSetting")]
public static class QueueFunctions
{
    [FunctionName("QueueOutput")]
    [return: Queue("myqueue-items")]
    public static string QueueOutput([HttpTrigger] dynamic input,  ILogger log)
    {
        log.LogInformation($"C# function processed: {input.Text}");
        return input.Text;
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur HTTP dans un fichier *function.json*, ainsi que le code du [script C# (.csx)](functions-reference-csharp.md) qui utilise cette liaison. La fonction crée un élément de file d’attente avec une charge utile d’objet **CustomQueueMessage** pour chaque requête HTTP reçue.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "$return",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# qui crée un message de file d’attente unique :

```cs
public class CustomQueueMessage
{
    public string PersonName { get; set; }
    public string Title { get; set; }
}

public static CustomQueueMessage Run(CustomQueueMessage input, ILogger log)
{
    return input;
}
```

Vous pouvez envoyer plusieurs messages à la fois en utilisant un paramètre `ICollector` ou `IAsyncCollector`. Voici le code Script C# qui envoie plusieurs messages, l’un avec les données de requête HTTP, et l’autre avec des valeurs codées en dur :

```cs
public static void Run(
    CustomQueueMessage input, 
    ICollector<CustomQueueMessage> myQueueItems, 
    ILogger log)
{
    myQueueItems.Add(input);
    myQueueItems.Add(new CustomQueueMessage { PersonName = "You", Title = "None" });
}
```

# <a name="java"></a>[Java](#tab/java)

 L’exemple ci-après montre une fonction Java qui crée un message de file d’attente en cas de déclenchement par une requête HTTP.

```java
@FunctionName("httpToQueue")
@QueueOutput(name = "item", queueName = "myqueue-items", connection = "MyStorageConnectionAppSetting")
 public String pushToQueue(
     @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
     final String message,
     @HttpOutput(name = "response") final OutputBinding<String> result) {
       result.setValue(message + " has been added.");
       return message;
 }
```

Dans la [bibliothèque runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@QueueOutput` sur les paramètres dont la valeur serait écrite dans Stockage File d’attente.  Le type de paramètre doit être `OutputBinding<T>`, où `T` est n’importe quel type Java natif d’un POJO.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur HTTP dans un fichier *function.json*, et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction crée un élément de file d’attente pour chaque requête HTTP reçue.

Voici le fichier *function.json* :

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "myQueueItem",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function (context, input) {
    context.bindings.myQueueItem = input.body;
    context.done();
};
```

Vous pouvez envoyer plusieurs messages à la fois en définissant un tableau de messages pour la liaison de sortie `myQueueItem`. Le code JavaScript suivant envoie deux messages de file d’attente avec des valeurs codées en dur pour chaque requête HTTP reçue.

```javascript
module.exports = function(context) {
    context.bindings.myQueueItem = ["message 1","message 2"];
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les exemples de code suivants montrent comment générer un message de file d’attente à partir d’une fonction déclenchée par le protocole HTTP. La section de configuration avec le `type` `queue` définit la liaison de sortie.

```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "Msg",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

Avec cette configuration de liaison, une fonction PowerShell peut créer un message de file d’attente à l’aide de `Push-OutputBinding`. Dans cet exemple, un message est créé à partir d’une chaîne de requête ou d’un paramètre de corps.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$message = $Request.Query.Message
Push-OutputBinding -Name Msg -Value $message
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = 200
    Body = "OK"
})
```

Pour envoyer plusieurs messages à la fois, définissez un tableau de messages et utilisez `Push-OutputBinding` pour envoyer des messages à la liaison de sortie de file d’attente.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$message = @("message1", "message2")
Push-OutputBinding -Name Msg -Value $message
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = 200
    Body = "OK"
})
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre comment sortir des valeurs uniques et multiples dans les files d’attente de stockage. La configuration requise pour *function.json* est la même dans les deux sens.

Une liaison de file d’attente de stockage est définie dans *function.json*, où *Type* est défini sur `queue`.

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
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "msg",
      "queueName": "outqueue",
      "connection": "AzureStorageQueuesConnectionString"
    }
  ]
}
```

Pour définir un message individuel sur la file d’attente, vous passez une valeur unique à la méthode `set`.

```python
import azure.functions as func

def main(req: func.HttpRequest, msg: func.Out[str]) -> func.HttpResponse:

    input_msg = req.params.get('message')

    msg.set(input_msg)

    return 'OK'
```

Pour créer plusieurs messages dans la file d’attente, déclarez un paramètre comme type de liste approprié et transmettez un tableau de valeurs (correspondant au type de liste) à la méthode `set`.

```python
import azure.functions as func
import typing

def main(req: func.HttpRequest, msg: func.Out[typing.List[str]]) -> func.HttpResponse:

    msg.set(['one', 'two'])

    return 'OK'
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [QueueAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues/QueueAttribute.cs).

L’attribut s’applique à un paramètre `out` ou à la valeur de retour de la fonction. Le constructeur de l’attribut prend le nom de la file d’attente, comme illustré dans l’exemple suivant :

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items")]
public static string Run([HttpTrigger] dynamic input,  ILogger log)
{
    ...
}
```

Vous pouvez définir la propriété `Connection` pour spécifier le compte de stockage à utiliser, comme indiqué dans l’exemple suivant :

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items", Connection = "StorageConnectionAppSetting")]
public static string Run([HttpTrigger] dynamic input,  ILogger log)
{
    ...
}
```

Pour obtenir un exemple complet, consultez [Exemple de sortie](#example).

Vous pouvez utiliser l’attribut `StorageAccount` pour spécifier le compte de stockage au niveau de la classe, de la méthode ou du paramètre. Pour plus d’informations, consultez Déclencheur - attributs.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="java"></a>[Java](#tab/java)

L’annotation `QueueOutput` vous permet d’écrire un message de sortie d’une fonction. L’exemple suivant montre une fonction déclenchée par HTTP qui crée un message de file d’attente.

```java
package com.function;
import java.util.*;
import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

public class HttpTriggerQueueOutput {
    @FunctionName("HttpTriggerQueueOutput")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION) HttpRequestMessage<Optional<String>> request,
            @QueueOutput(name = "message", queueName = "messages", connection = "MyStorageConnectionAppSetting") OutputBinding<String> message,
            final ExecutionContext context) {

        message.setValue(request.getQueryParameters().get("name"));
        return request.createResponseBuilder(HttpStatus.OK).body("Done").build();
    }
}
```

| Propriété    | Description |
|-------------|-----------------------------|
|`name`       | Déclare le nom du paramètre dans la signature de la fonction. Quand la fonction est déclenchée, la valeur de ce paramètre a le contenu du message de la file d’attente. |
|`queueName`  | Déclare le nom de la file d’attente dans le compte de stockage. |
|`connection` | Pointe vers la chaîne de connexion du compte de stockage. |

Le paramètre associé à l’annotation `QueueOutput` est tapé en tant qu’instance [OutputBinding\<T\>](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/OutputBinding.java).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les attributs ne sont pas pris en charge par PowerShell.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `Queue`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Cette propriété doit être définie sur `queue`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction** | n/a | Cette propriété doit être définie sur `out`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. |
|**name** | n/a | Nom de la variable qui représente la file d’attente dans le code de la fonction. La valeur doit être `$return` pour faire référence à la valeur de retour de la fonction.|
|**queueName** |**QueueName** | Nom de la file d'attente. |
|**connection** | **Connection** |Nom d’un paramètre d’application comportant la chaîne de connexion de stockage à utiliser pour cette liaison. Si le nom du paramètre d’application commence par « AzureWebJobs », vous ne pouvez spécifier que le reste du nom ici. Par exemple, si vous définissez `connection` sur « MyStorage », le runtime Functions recherche un paramètre d’application qui est nommé « MyStorage ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion de stockage par défaut dans le paramètre d’application nommé `AzureWebJobsStorage`.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Écrivez un message de file d’attente unique en utilisant un paramètre de méthode comme `out T paramName`. Vous pouvez utiliser le type de retour de la méthode au lieu d’un paramètre `out`, et `T` peut être un des types suivants :

* Un objet sérialisable au format JSON
* `string`
* `byte[]`
* [CloudQueueMessage] 

Si vous essayez de lier à `CloudQueueMessage` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

En C# et Script C#, écrivez plusieurs messages de file d’attente à l’aide d’un des types suivants : 

* `ICollector<T>` ou `IAsyncCollector<T>`
* [CloudQueue](/dotnet/api/microsoft.azure.storage.queue.cloudqueue)

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Écrivez un message de file d’attente unique en utilisant un paramètre de méthode comme `out T paramName`. `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. Vous pouvez utiliser le type de retour de la méthode au lieu d’un paramètre `out`, et `T` peut être un des types suivants :

* Un objet sérialisable au format JSON
* `string`
* `byte[]`
* [CloudQueueMessage] 

Si vous essayez de lier à `CloudQueueMessage` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

En C# et Script C#, écrivez plusieurs messages de file d’attente à l’aide d’un des types suivants : 

* `ICollector<T>` ou `IAsyncCollector<T>`
* [CloudQueue](/dotnet/api/microsoft.azure.storage.queue.cloudqueue)

# <a name="java"></a>[Java](#tab/java)

Il existe deux options pour produire en sortie un message File d’attente à partir d’une fonction en utilisant l’annotation [QueueOutput](/java/api/com.microsoft.azure.functions.annotation.queueoutput) :

- **Valeur de retour** : En appliquant l’annotation à la fonction elle-même, la valeur renvoyée de la fonction est conservée sous la forme d’un message File d’attente.

- **Impératif** : Pour définir explicitement la valeur du message, appliquez l’annotation à un paramètre spécifique du type [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding), où `T` est un POJO ou n’importe quel type Java natif. Avec cette configuration, le passage d’une valeur à la méthode `setValue` rend la valeur persistante en tant que message File d’attente.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’élément de la file d’attente en sortie est disponible via `context.bindings.<NAME>`, où `<NAME>` correspond au nom défini dans *function.json*. Vous pouvez utiliser une chaîne ou un objet sérialisable JSON pour la charge utile de l’élément de file d’attente.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

La sortie dans le message de file d’attente est disponible via `Push-OutputBinding`, où vous transmettez les arguments qui correspondent au nom désigné par le paramètre `name` de la liaison dans le fichier *function.json*.

# <a name="python"></a>[Python](#tab/python)

Il existe deux options pour produire en sortie un message File d’attente à partir d’une fonction :

- **Valeur de retour** : Définissez la propriété `name` dans *function.json* sur `$return`. Avec cette configuration, la valeur de retour de la fonction est conservée sous la forme d’un message de Stockage File d’attente.

- **Impératif** : Passez une valeur à la méthode [set](/python/api/azure-functions/azure.functions.out?view=azure-python&preserve-view=true#set-val--t-----none) du paramètre déclaré en tant que type [Out](/python/api/azure-functions/azure.functions.out?view=azure-python&preserve-view=true). La valeur passée à `set` est conservée en tant que message de Stockage File d’attente.

---

## <a name="exceptions-and-return-codes"></a>Exceptions et codes de retour

| Liaison |  Informations de référence |
|---|---|
| File d'attente | [Codes d’erreur de file d’attente](/rest/api/storageservices/queue-service-error-codes) |
| Objet blob, Table, File d’attente | [Codes d’erreur de stockage](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Objet blob, Table, File d’attente |  [Dépannage](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

<a name="host-json"></a>  

## <a name="hostjson-settings"></a>Paramètres host.json

Cette section décrit les paramètres de configuration globaux disponibles pour cette liaison dans les versions 2.x et ultérieures. L’exemple de fichier host.json ci-dessous contient seulement les paramètres des versions 2.x et ultérieures pour cette liaison. Pour plus d’informations sur les paramètres de configuration globaux dans les versions 2.x et ultérieures, consultez [Informations de référence sur le fichier host.json pour Azure Functions](functions-host-json.md).

> [!NOTE]
> Pour obtenir une référence de host.json dans Functions 1.x, consultez [Informations de référence sur le fichier host.json pour Azure Functions 1.x](functions-host-json-v1.md).

```json
{
    "version": "2.0",
    "extensions": {
        "queues": {
            "maxPollingInterval": "00:00:02",
            "visibilityTimeout" : "00:00:30",
            "batchSize": 16,
            "maxDequeueCount": 5,
            "newBatchThreshold": 8
        }
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------|
|maxPollingInterval|00:00:01|Intervalle maximal entre les interrogations de la file d’attente. L’intervalle minimum est de 00:00:00.100 (100 ms) et il est augmenté par incréments de 00:01:00 (1 minute).  Dans 1.x, le type de données est « milliseconds » (millisecondes) et, dans 2.x et ultérieur, il s’agit de « TimeSpan » (intervalle de temps).|
|visibilityTimeout|00:00:00|Intervalle de temps entre les nouvelles tentatives en cas d’échec du traitement d’un message. |
|batchSize|16|Le nombre de messages de file d’attente que le runtime Functions récupère simultanément et traite en parallèle. Quand le nombre de messages en cours de traitement descend à `newBatchThreshold`, le runtime obtient un autre lot et commence à traiter ces messages. Par conséquent, le nombre maximal de messages traités simultanément par fonction est `batchSize` plus `newBatchThreshold`. Cette limite s’applique séparément à chaque fonction déclenchée par une file d’attente. <br><br>Si vous souhaitez éviter les exécutions parallèles pour les messages reçus sur une file d’attente, vous pouvez définir `batchSize` sur 1. Toutefois, ce paramètre évite les opérations simultanées uniquement pendant l’exécution de votre application de fonction sur une machine virtuelle unique. Si l’application de fonction augmente la taille des instances sur plusieurs machines virtuelles, chaque machine virtuelle peut exécuter une instance de chaque fonction déclenchée par une file d’attente.<br><br>La valeur `batchSize` maximale est de 32. |
|maxDequeueCount|5|Nombre de tentatives de traitement d’un message avant de le placer dans la file d’attente de messages incohérents.|
|newBatchThreshold|batchSize/2|Quand le nombre de messages traités simultanément passe en dessous de cette valeur, le runtime récupère un autre lot.|

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction pour modifier des données de stockage de file d’attente (déclencheur)](./functions-bindings-storage-queue-trigger.md)

<!-- LINKS -->

[CloudQueueMessage]: /dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage
