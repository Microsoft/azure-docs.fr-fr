---
title: Liaisons de sortie Azure RabbitMQ pour Azure Functions
description: Apprenez à envoyer des messages RabbitMQ à partir d’Azure Functions.
author: cachai2
ms.assetid: ''
ms.topic: reference
ms.date: 12/13/2020
ms.author: cachai
ms.custom: ''
ms.openlocfilehash: 212bfcee09cd63b6ff09faaba4d99e4b4c583fe8
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505661"
---
# <a name="rabbitmq-output-binding-for-azure-functions-overview"></a>Vue d’ensemble des liaisons de sortie RabbitMQ pour Azure Functions

> [!NOTE]
> Les liaisons RabbitMQ ne sont entièrement prises en charge que sur des plans **Windows Premium**. Les plans Consommation et Linux ne sont actuellement pas pris en charge.

Utilisez la liaison de sortie RabbitMQ pour envoyer des messages à une file d’attente RabbitMQ.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-rabbitmq-output.md).

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant présente une [fonction C#](functions-dotnet-class-library.md) qui envoie un message RabbitMQ quand elle est déclenchée par un TimerTrigger toutes les 5 minutes à l’aide de la valeur de retour de la méthode en tant que sortie :

```cs
[FunctionName("RabbitMQOutput")]
[return: RabbitMQ("outputQueue", ConnectionStringSetting = "ConnectionStringSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

L’exemple suivant montre comment utiliser l’interface IAsyncCollector pour envoyer des messages.

```cs
[FunctionName("RabbitMQOutput")]
public static async Task Run(
[RabbitMQTrigger("sourceQueue", ConnectionStringSetting = "TriggerConnectionString")] string rabbitMQEvent,
[RabbitMQ("destinationQueue", ConnectionStringSetting = "OutputConnectionString")]IAsyncCollector<string> outputEvents,
ILogger log)
{
    // processing:
    var myProcessedEvent = DoSomething(rabbitMQEvent);
    
     // send the message
    await outputEvents.AddAsync(JsonConvert.SerializeObject(myProcessedEvent));
}
```

L’exemple suivant montre comment envoyer les messages en tant qu’objets CLR traditionnels.

```cs
public class TestClass
{
    public string x { get; set; }
}

[FunctionName("RabbitMQOutput")]
public static async Task Run(
[RabbitMQTrigger("sourceQueue", ConnectionStringSetting = "TriggerConnectionString")] TestClass rabbitMQEvent,
[RabbitMQ("destinationQueue", ConnectionStringSetting = "OutputConnectionString")]IAsyncCollector<TestClass> outputPocObj,
ILogger log)
{
    // send the message
    await outputPocObj.Add(rabbitMQEvent);
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de sortie RabbitMQ dans un fichier *function.json*, et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction lit le message d’un déclencheur HTTP et l’envoie à la file d’attente RabbitMQ.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "bindings": [
        {
            "type": "httpTrigger",
            "direction": "in",
            "authLevel": "function",
            "name": "input",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "type": "rabbitMQ",
            "name": "outputMessage",
            "queueName": "outputQueue",
            "connectionStringSetting": "connectionStringAppSetting",
            "direction": "out"
        }
    ]
}
```

Voici le code Script C# :

```csx
using System;
using Microsoft.Extensions.Logging;

public static void Run(string input, out string outputMessage, ILogger log)
{
    log.LogInformation(input);
    outputMessage = input;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de sortie RabbitMQ dans un fichier *function.json*, et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction lit le message d’un déclencheur HTTP et l’envoie à la file d’attente RabbitMQ.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "bindings": [
        {
            "type": "httpTrigger",
            "direction": "in",
            "authLevel": "function",
            "name": "input",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "type": "rabbitMQ",
            "name": "outputMessage",
            "queueName": "outputQueue",
            "connectionStringSetting": "connectionStringAppSetting",
            "direction": "out"
        }
    ]
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, input) {
    context.bindings.myQueueItem = input.body;
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre une liaison de sortie RabbitMQ dans un fichier *function.json*, et une fonction Python qui utilise la liaison. La fonction lit le message d’un déclencheur HTTP et l’envoie à la file d’attente RabbitMQ.

Voici les données de liaison dans le fichier *function.json* :

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
            "type": "rabbitMQ",
            "name": "outputMessage",
            "queueName": "outputQueue",
            "connectionStringSetting": "connectionStringAppSetting",
            "direction": "out"
        }
    ]
}
```

Dans *_\_init_\_.py*, vous pouvez écrire un message dans la file d’attente en passant une valeur à la méthode `set`.

```python
import azure.functions as func

def main(req: func.HttpRequest, msg: func.Out[str]) -> func.HttpResponse:
    input_msg = req.params.get('message')
    msg.set(input_msg)
    return 'OK'
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant montre une fonction Java qui envoie un message à une file d’attente RabbitMQ quand elle est déclenchée par un TimerTrigger toutes les 5 minutes.

```java
@FunctionName("RabbitMQOutputExample")
public void run(
@TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
@RabbitMQOutput(connectionStringSetting = "rabbitMQ", queueName = "hello") OutputBinding<String> output,
final ExecutionContext context) {
    output.setValue("Some string");
}
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans des [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [RabbitMQAttribute](https://github.com/Azure/azure-functions-rabbitmq-extension/blob/dev/src/RabbitMQAttribute.cs).

Voici un attribut `RabbitMQAttribute` dans une signature de méthode :

```csharp
[FunctionName("RabbitMQOutput")]
public static async Task Run(
[RabbitMQTrigger("SourceQueue", ConnectionStringSetting = "TriggerConnectionString")] string rabbitMQEvent,
[RabbitMQ("DestinationQueue", ConnectionStringSetting = "OutputConnectionString")]IAsyncCollector<string> outputEvents,
ILogger log)
{
    ...
}
```

Vous trouverez un exemple complet sur la page [Exemple C#](#example).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

L’annotation `RabbitMQOutput` vous permet de créer une fonction qui s’exécute lors de l’envoi d’un message RabbitMQ. Les options de configuration disponibles sont le nom de la file d’attente et le nom de la chaîne de connexion. Pour plus d’informations sur les paramètres, consultez les [annotations Java RabbitMQOutput](https://github.com/Azure/azure-functions-rabbitmq-extension/blob/dev/binding-library/java/src/main/java/com/microsoft/azure/functions/rabbitmq/annotation/RabbitMQOutput.java).

Pour plus d’informations, consultez l’[exemple](#example) de liaison de sortie.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `RabbitMQ`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Doit être défini sur « RabbitMQ ».|
|**direction** | n/a | Doit être défini sur « out ». |
|**name** | n/a | Nom de la variable qui représente la file d’attente dans le code de la fonction. |
|**queueName**|**QueueName**| Nom de la file d’attente à laquelle envoyer les messages. |
|**hostName**|**HostName**|(facultatif si vous utilisez ConnectStringSetting) <br>Nom d’hôte de la file d’attente (par exemple : 10.26.45.210)|
|**userNameSetting**|**UserNameSetting**|(facultatif si vous utilisez ConnectionStringSetting) <br>Nom pour accéder à la file d’attente |
|**passwordSetting**|**PasswordSetting**|(facultatif si vous utilisez ConnectionStringSetting) <br>Mot de passe pour accéder à la file d’attente|
|**connectionStringSetting**|**ConnectionStringSetting**|Nom du paramètre d’application qui contient la chaîne de connexion de la file d’attente de messages RabbitMQ. Notez que si vous spécifiez la chaîne de connexion directement et non par le biais d’un paramètre d’application dans local.settings.json, le déclencheur ne fonctionnera pas. (exemple : dans *function.json* : connectionStringSetting: "rabbitMQConnection" <br> Dans *local.settings.json* : "rabbitMQConnection" : "< ActualConnectionstring >")|
|**port**|**Port**|Obtient ou définit les ports utilisés. La valeur par défaut est 0.|

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Utilisez les types de paramètres suivants pour la liaison de sortie :

* `byte[]` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `string` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `POCO` - Si la valeur du paramètre n’est pas formatée en tant qu’objet C#, une erreur est reçue. Vous trouverez un exemple complet sur la page [Exemple C#](#example).

Quand vous utilisez des fonctions C# :

* Les fonctions asynchrones ont besoin d’une valeur renvoyée ou de `IAsyncCollector` au lieu d’un paramètre `out`.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Utilisez les types de paramètres suivants pour la liaison de sortie :

* `byte[]` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `string` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `POCO` - Si la valeur du paramètre n’est pas formatée en tant qu’objet C#, une erreur est reçue.

Quand vous utilisez des fonctions de script C# :

* Les fonctions asynchrones ont besoin d’une valeur renvoyée ou de `IAsyncCollector` au lieu d’un paramètre `out`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le message RabbitMQ est envoyé via une chaîne.

# <a name="python"></a>[Python](#tab/python)

Le message RabbitMQ est envoyé via une chaîne.

# <a name="java"></a>[Java](#tab/java)

Utilisez les types de paramètres suivants pour la liaison de sortie :

* `byte[]` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `string` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `POJO` - Si la valeur du paramètre n’est pas formatée en tant qu’objet Java, une erreur est reçue.

---

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction lors de la création d’un message RabbitMQ (déclencheur)](./functions-bindings-rabbitmq-trigger.md)
