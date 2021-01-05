---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: 11ad3bdcaa40c479c9358fd623edf0e6fdafa0d6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002071"
---
Utilisez la liaison de sortie Event Hubs pour écrire des événements dans un flux d’événements du hub d’événements. Vous devez disposer de l’autorisation d’envoi à un hub d’événements pour y écrire les événements.

Vérifiez que les références de package nécessaires sont en place avant d’essayer d’implémenter une liaison de sortie.

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant illustre une [fonction C#](../articles/azure-functions/functions-dotnet-class-library.md) qui écrit un message dans un hub d’événements, en utilisant la valeur retournée par la méthode comme sortie :

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

L’exemple suivant montre comment utiliser l’interface `IAsyncCollector` pour envoyer un lot de messages. Ce scénario est courant lorsque vous traitez des messages provenant d’un hub d’événements et que vous envoyez le résultat à un autre hub d’événements.

```csharp
[FunctionName("EH2EH")]
public static async Task Run(
    [EventHubTrigger("source", Connection = "EventHubConnectionAppSetting")] EventData[] events,
    [EventHub("dest", Connection = "EventHubConnectionAppSetting")]IAsyncCollector<string> outputEvents,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        // do some processing:
        var myProcessedEvent = DoSomething(eventData);

        // then send the message
        await outputEvents.AddAsync(JsonConvert.SerializeObject(myProcessedEvent));
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant illustre une liaison de déclencheur de hub d’événements dans un fichier *function.json* et une [fonction de script C#](../articles/azure-functions/functions-reference-csharp.md) qui utilise la liaison. La fonction écrit un message dans un hub d’événements.

Les exemples suivants illustrent les données de liaison Event Hubs dans le fichier *function.json*. Le premier exemple concerne Functions 2.x et versions ultérieures, tandis que le second concerne Functions 1.x. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Voici le code Script C# qui crée un message :

```cs
using System;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out string outputEventHubMessage, ILogger log)
{
    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
    log.LogInformation(msg);   
    outputEventHubMessage = msg;
}
```

Voici le code de script C# qui crée plusieurs messages :

```cs
public static void Run(TimerInfo myTimer, ICollector<string> outputEventHubMessage, ILogger log)
{
    string message = $"Message created at: {DateTime.Now}";
    log.LogInformation(message);
    outputEventHubMessage.Add("1 " + message);
    outputEventHubMessage.Add("2 " + message);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant illustre une liaison de déclencheur Event Hub dans un fichier *function.json* et une [fonction JavaScript](../articles/azure-functions/functions-reference-node.md) qui utilise la liaison. La fonction écrit un message dans un hub d’événements.

Les exemples suivants illustrent les données de liaison Event Hubs dans le fichier *function.json*. Le premier exemple concerne Functions 2.x et versions ultérieures, tandis que le second concerne Functions 1.x. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Voici le code JavaScript qui envoie un message unique :

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    context.log('Message created at: ', timeStamp);   
    context.bindings.outputEventHubMessage = "Message created at: " + timeStamp;
    context.done();
};
```

Voici le code JavaScript qui envoie plusieurs messages :

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();
    var message = 'Message created at: ' + timeStamp;

    context.bindings.outputEventHubMessage = [];

    context.bindings.outputEventHubMessage.push("1 " + message);
    context.bindings.outputEventHubMessage.push("2 " + message);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant illustre une liaison de déclencheur Event Hub dans un fichier *function.json* et une [fonction Python](../articles/azure-functions/functions-reference-python.md) qui utilise la liaison. La fonction écrit un message dans un hub d’événements.

Les exemples suivants illustrent les données de liaison Event Hubs dans le fichier *function.json*.

```json
{
    "type": "eventHub",
    "name": "$return",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Voici le code Python qui envoie un message unique :

```python
import datetime
import logging
import azure.functions as func


def main(timer: func.TimerRequest) -> str:
    timestamp = datetime.datetime.utcnow()
    logging.info('Message created at: %s', timestamp)
    return 'Message created at: {}'.format(timestamp)
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant illustre une fonction Java qui écrit un message contenant l’heure actuelle à l’attention d’un Event Hub.

```java
@FunctionName("sendTime")
@EventHubOutput(name = "event", eventHubName = "samples-workitems", connection = "AzureEventHubConnection")
public String sendTime(
   @TimerTrigger(name = "sendTimeTrigger", schedule = "0 */5 * * * *") String timerInfo)  {
     return LocalDateTime.now().toString();
 }
```

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@EventHubOutput` sur les paramètres dont la valeur serait publiée à l’attention d’Event Hub.  Le type de paramètre doit être `OutputBinding<T>`, où T désigne un POJO ou n’importe quel type Java natif.

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](../articles/azure-functions/functions-dotnet-class-library.md), utilisez l’attribut [EventHubAttribute](https://github.com/Azure/azure-functions-eventhubs-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs/EventHubAttribute.cs).

Le constructeur de l’attribut prend le nom du hub d’événements, le hub d’événements et le nom d’un paramètre d’application qui contient la chaîne de connexion. Pour plus d’informations sur ces paramètres, consultez [Sortie - configuration](#configuration). Voici un exemple d’attribut `EventHub` :

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Pour obtenir un exemple complet, consultez [Sortie - exemple C#](#example).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput) sur les paramètres dont la valeur serait publiée à l’attention d’Event Hub. Le type de paramètre doit être `OutputBinding<T>`, où `T` désigne un POJO ou n’importe quel type Java natif.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `EventHub`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Doit être défini sur eventHub. |
|**direction** | n/a | Doit être défini sur « out ». Ce paramètre est défini automatiquement lorsque vous créez la liaison dans le portail Azure. |
|**name** | n/a | Nom de variable utilisé dans le code de la fonction qui représente l’événement. |
|**path** |**EventHubName** | Functions 1.x uniquement. Nom du hub d’événements. Lorsque le nom d’Event Hub est également présent dans la chaîne de connexion, sa valeur remplace cette propriété lors de l’exécution. |
|**eventHubName** |**EventHubName** | Functions 2.x et versions ultérieures. Nom du hub d’événements. Lorsque le nom d’Event Hub est également présent dans la chaîne de connexion, sa valeur remplace cette propriété lors de l’exécution. |
|**connection** |**Connection** | Le nom d’un paramètre d’application qui contient la chaîne de connexion à l’espace de noms du hub d’événements. Copiez cette chaîne de connexion en cliquant sur le bouton **Informations de connexion** pour *l’espace de noms*, et non pour le hub d’événements lui-même. Cette chaîne de connexion doit disposer d’autorisations d’envoi pour envoyer le message au flux d’événements.|

[!INCLUDE [app settings to local.settings.json](../articles/azure-functions/../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Envoyez des messages en utilisant un paramètre de méthode comme `out string paramName`. Dans Script C#, `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. Pour écrire plusieurs messages, vous pouvez utiliser `ICollector<string>` ou `IAsyncCollector<string>` à la place de `out string`.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Envoyez des messages en utilisant un paramètre de méthode comme `out string paramName`. Dans Script C#, `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. Pour écrire plusieurs messages, vous pouvez utiliser `ICollector<string>` ou `IAsyncCollector<string>` à la place de `out string`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Accédez à l’événement de sortie à l’aide de `context.bindings.<name>` où `<name>` est la valeur spécifiée dans la propriété `name` de *function.json*.

# <a name="python"></a>[Python](#tab/python)

Il existe deux options pour produire en sortie un message Event Hub à partir d’une fonction :

- **Valeur de retour** : Définissez la propriété `name` dans *function.json* sur `$return`. Avec cette configuration, la valeur renvoyée de la fonction est conservée sous la forme d’un message Event Hub.

- **Impératif** : Passez une valeur à la méthode [set](/python/api/azure-functions/azure.functions.out?view=azure-python#set-val--t-----none) du paramètre déclaré en tant que type [Out](/python/api/azure-functions/azure.functions.out?view=azure-python). La valeur transmise à `set` est conservée en tant que message d’Event Hub.

# <a name="java"></a>[Java](#tab/java)

Il existe deux options pour produire en sortie un message Event hub à partir d’une fonction en utilisant l’annotation [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput) :

- **Valeur de retour** : En appliquant l’annotation à la fonction elle-même, la valeur de retour de la fonction est conservée sous la forme d’un message Event Hub.

- **Impératif** : Pour définir explicitement la valeur du message, appliquez l’annotation à un paramètre spécifique du type [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.OutputBinding), où `T` est un POJO ou n’importe quel type Java natif. Avec cette configuration, le passage d’une valeur à la méthode `setValue` rend la valeur persistante en tant que message Event Hub.

---

## <a name="exceptions-and-return-codes"></a>Exceptions et codes de retour

| Liaison | Informations de référence |
|---|---|
| Event Hub | [Guide des opérations](/rest/api/eventhub/publisher-policy-operations) |