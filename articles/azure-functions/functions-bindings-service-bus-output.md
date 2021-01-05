---
title: Liaisons de sortie Azure Service Bus pour Azure Functions
description: Apprenez à envoyer des messages Azure Service Bus à partir d’Azure Functions.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 2d0b66d2b4d89b512b34cb33a5607b471b7d1e84
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040938"
---
# <a name="azure-service-bus-output-binding-for-azure-functions"></a>Liaison de sortie Azure Service Bus pour Azure Functions

Utilisez la liaison de sortie Azure Service Bus pour envoyer des messages de file d’attente ou de rubrique.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-service-bus.md).

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui envoie un message de la file d’attente Service Bus :

```cs
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue", Connection = "ServiceBusConnection")]
public static string ServiceBusOutput([HttpTrigger] dynamic input, ILogger log)
{
    log.LogInformation($"C# function processed: {input.Text}");
    return input.Text;
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de sortie Service Bus dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction utilise un déclencheur de minuteur pour envoyer un message de file d’attente toutes les 15 secondes.

Voici les données de liaison dans le fichier *function.json*  :

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

Voici le code de script C# qui crée un message unique :

```cs
public static void Run(TimerInfo myTimer, ILogger log, out string outputSbQueue)
{
    string message = $"Service Bus queue message created at: {DateTime.Now}";
    log.LogInformation(message); 
    outputSbQueue = message;
}
```

Voici le code de script C# qui crée plusieurs messages :

```cs
public static async Task Run(TimerInfo myTimer, ILogger log, IAsyncCollector<string> outputSbQueue)
{
    string message = $"Service Bus queue messages created at: {DateTime.Now}";
    log.LogInformation(message); 
    await outputSbQueue.AddAsync("1 " + message);
    await outputSbQueue.AddAsync("2 " + message);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de sortie Service Bus dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction utilise un déclencheur de minuteur pour envoyer un message de file d’attente toutes les 15 secondes.

Voici les données de liaison dans le fichier *function.json*  :

```json
{
    "bindings": [
        {
            "schedule": "0/15 * * * * *",
            "name": "myTimer",
            "runsOnStartup": true,
            "type": "timerTrigger",
            "direction": "in"
        },
        {
            "name": "outputSbQueue",
            "type": "serviceBus",
            "queueName": "testqueue",
            "connection": "MyServiceBusConnection",
            "direction": "out"
        }
    ],
    "disabled": false
}
```

Voici le code de script JavaScript qui crée un message unique :

```javascript
module.exports = function (context, myTimer) {
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueue = message;
    context.done();
};
```

Voici le code de script JavaScript qui crée plusieurs messages :

```javascript
module.exports = function (context, myTimer) {
    var message = 'Service Bus queue message created at ' + timeStamp;
    context.log(message);   
    context.bindings.outputSbQueue = [];
    context.bindings.outputSbQueue.push("1 " + message);
    context.bindings.outputSbQueue.push("2 " + message);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre comment écrire dans une file d’attente Service Bus en Python.

Une définition de liaison Service Bus figure dans *function.json* , où *type* a la valeur `serviceBus`.

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
      "type": "serviceBus",
      "direction": "out",
      "connection": "AzureServiceBusConnectionString",
      "name": "msg",
      "queueName": "outqueue"
    }
  ]
}
```

Dans *_\_init_\_.py* , vous pouvez écrire un message dans la file d’attente en passant une valeur à la méthode `set`.

```python
import azure.functions as func

def main(req: func.HttpRequest, msg: func.Out[str]) -> func.HttpResponse:

    input_msg = req.params.get('message')

    msg.set(input_msg)

    return 'OK'
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant montre une fonction Java qui envoie un message à une file d’attente Service Bus `myqueue` lorsqu’elle est déclenchée par une requête HTTP.

```java
@FunctionName("httpToServiceBusQueue")
@ServiceBusQueueOutput(name = "message", queueName = "myqueue", connection = "AzureServiceBusConnection")
public String pushToQueue(
  @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
  final String message,
  @HttpOutput(name = "response") final OutputBinding<T> result ) {
      result.setValue(message + " has been sent.");
      return message;
 }
```

 Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `@QueueOutput` sur les paramètres de fonction dont la valeur serait écrite dans une file d’attente Service Bus.  Le type de paramètre doit être `OutputBinding<T>`, où T désigne n’importe quel type Java natif d’un POJO.

Les fonctions Java peuvent également écrire dans une rubrique Service Bus. L’exemple suivant utilise l’annotation `@ServiceBusTopicOutput` afin de décrire la configuration pour la liaison de sortie. 

```java
@FunctionName("sbtopicsend")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> request,
            @ServiceBusTopicOutput(name = "message", topicName = "mytopicname", subscriptionName = "mysubscription", connection = "ServiceBusConnection") OutputBinding<String> message,
            final ExecutionContext context) {
        
        String name = request.getBody().orElse("Azure Functions");

        message.setValue(name);
        return request.createResponseBuilder(HttpStatus.OK).body("Hello, " + name).build();
        
    }
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [ServiceBusAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAttribute.cs).

Le constructeur de l’attribut prend le nom de la file d’attente ou la rubrique et l’abonnement. Vous pouvez également spécifier les droits d’accès de la connexion. La détermination du paramètre des droits d’accès est expliquée dans la section [Sortie - configuration](#configuration). Voici un exemple qui illustre l’attribut appliqué à la valeur de retour de la fonction :

```csharp
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue")]
public static string Run([HttpTrigger] dynamic input, ILogger log)
{
    ...
}
```

Vous pouvez définir la propriété `Connection` pour spécifier le nom d’un paramètre d’application contenant la chaîne de connexion Service Bus à utiliser, comme illustré dans l’exemple suivant :

```csharp
[FunctionName("ServiceBusOutput")]
[return: ServiceBus("myqueue", Connection = "ServiceBusConnection")]
public static string Run([HttpTrigger] dynamic input, ILogger log)
{
    ...
}
```

Pour obtenir un exemple complet, voir [Exemple de sortie](#example).

Vous pouvez utiliser l’attribut `ServiceBusAccount` pour spécifier le compte Service Bus à utiliser au niveau de la classe, de la méthode ou du paramètre.  Pour plus d’informations, consultez [Déclencheur - attributs](functions-bindings-service-bus-trigger.md#attributes-and-annotations).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

Les annotations `ServiceBusQueueOutput` et `ServiceBusTopicOutput` sont disponibles pour écrire un message en tant que sortie de fonction. Le paramètre assorti de ces annotations doit être déclaré en tant que `OutputBinding<T>` où `T` est le type correspondant au type du message.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `ServiceBus`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Doit être défini sur « serviceBus ». Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction** | n/a | Doit être défini sur « out ». Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. |
|**name** | n/a | Nom de la variable qui représente le message de la file d’attente ou de la rubrique dans le code de la fonction. Défini sur « $return » pour faire référence à la valeur de retour de la fonction. |
|**queueName**|**QueueName**|Nom de la file d’attente.  Défini uniquement en cas d’envoi de messages de file d’attente, ne s’applique pas à une rubrique.
|**topicName**|**TopicName**|Nom de la rubrique. Défini uniquement en cas d’envoi de messages de rubrique, ne s’applique pas à une file d’attente.|
|**connection**|**Connection**|Nom d’un paramètre d’application comportant la chaîne de connexion Service Bus à utiliser pour cette liaison. Si le nom du paramètre d’application commence par « AzureWebJobs », vous ne pouvez spécifier que le reste du nom. Par exemple, si vous définissez `connection` sur « MyServiceBus », le runtime Functions recherche un paramètre d’application qui est nommé « AzureWebJobsMyServiceBus ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion Service Bus par défaut dans le paramètre d’application nommé « AzureWebJobsServiceBus ».<br><br>Pour obtenir une chaîne de connexion, suivez les étapes indiquées à la section [Obtenir les informations d’identification de gestion](../service-bus-messaging/service-bus-quickstart-portal.md#get-the-connection-string). La chaîne de connexion doit être destinée à un espace de noms Service Bus, et non limitée à une file d’attente ou une rubrique spécifique.|
|**accessRights** (v1 uniquement)|**y accéder**|Droits d’accès de la chaîne de connexion. Les valeurs disponibles sont `manage` et `listen`. La valeur par défaut est `manage`, ce qui indique que `connection` a l'autorisation **Gérer**. Si vous utilisez une chaîne de connexion qui n’a pas l'autorisation **Gérer** , définissez `accessRights` sur « écouter ». Sinon, le runtime Functions pourrait échouer à effectuer des opérations qui nécessitent des droits de gestion. Dans Azure Functions versions 2.x et ultérieures, cette propriété n’est pas disponible parce que la version la plus récente du Kit de développement logiciel (SDK) Service Bus ne prend pas en charge les opérations de gestion.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

Dans Azure Functions version 1.x, le runtime crée la file d’attente si elle n’existe pas et si vous avez défini `accessRights` sur `manage`. Dans Functions version 2.x et ultérieur, la file d’attente ou la rubrique doit déjà exister. Si vous spécifiez une file d’attente ou une rubrique qui n’existe pas, la fonction échoue. 

# <a name="c"></a>[C#](#tab/csharp)

Utilisez les types de paramètres suivants pour la liaison de sortie :

* `out T paramName` - `T` peut être n’importe quel type sérialisable au format JSON. Si la valeur du paramètre est null lorsque la fonction se termine, Functions crée le message avec un objet null.
* `out string` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `out byte[]` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `out BrokeredMessage` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message (pour Functions 1.x).
* `out Message` - Si la valeur du paramètre est null quand la fonction se termine, Functions ne crée pas de message (pour Functions 2.x et ultérieur)
* `ICollector<T>` ou `IAsyncCollector<T>` (pour les méthodes asynchrones) - Pour la création de plusieurs messages. Un message est créé quand vous appelez la méthode `Add` .

Quand vous utilisez des fonctions C# :

* Les fonctions asynchrones ont besoin d’une valeur renvoyée ou de `IAsyncCollector` au lieu d’un paramètre `out`.

* Pour accéder à l’ID de session, établissez une liaison à un type [`Message`](/dotnet/api/microsoft.azure.servicebus.message) et utilisez la propriété `sessionId`.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Utilisez les types de paramètres suivants pour la liaison de sortie :

* `out T paramName` - `T` peut être n’importe quel type sérialisable au format JSON. Si la valeur du paramètre est null lorsque la fonction se termine, Functions crée le message avec un objet null.
* `out string` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `out byte[]` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message.
* `out BrokeredMessage` - Si la valeur du paramètre est null lorsque la fonction se termine, Functions ne crée pas de message (pour Functions 1.x).
* `out Message` - Si la valeur du paramètre est null quand la fonction se termine, Functions ne crée pas de message (pour Functions 2.x et ultérieur)
* `ICollector<T>` ou `IAsyncCollector<T>` - Pour la création de plusieurs messages. Un message est créé quand vous appelez la méthode `Add` .

Quand vous utilisez des fonctions C# :

* Les fonctions asynchrones ont besoin d’une valeur renvoyée ou de `IAsyncCollector` au lieu d’un paramètre `out`.

* Pour accéder à l’ID de session, établissez une liaison à un type [`Message`](/dotnet/api/microsoft.azure.servicebus.message) et utilisez la propriété `sessionId`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Accédez à la file d’attente ou la rubrique à l’aide de `context.bindings.<name from function.json>`. Vous pouvez affecter une chaîne, un tableau d’octets ou un objet Javascript (désérialisé au format JSON) à `context.binding.<name>`.

# <a name="python"></a>[Python](#tab/python)

Utilisez le [Kit de développement logiciel (SDK) Azure Service Bus](../service-bus-messaging/index.yml) plutôt que la liaison de sortie intégrée.

# <a name="java"></a>[Java](#tab/java)

Utilisez le [Kit de développement logiciel (SDK) Azure Service Bus](../service-bus-messaging/index.yml) plutôt que la liaison de sortie intégrée.

---

## <a name="exceptions-and-return-codes"></a>Exceptions et codes de retour

| Liaison | Informations de référence |
|---|---|
| Service Bus | [Codes d’erreur de Service Bus](../service-bus-messaging/service-bus-messaging-exceptions.md) |
| Service Bus | [Limites de Service Bus](../service-bus-messaging/service-bus-quotas.md) |

<a name="host-json"></a>  

## <a name="hostjson-settings"></a>Paramètres host.json

Cette section décrit les paramètres de configuration globaux disponibles pour cette liaison dans les versions 2.x et ultérieures. L’exemple de fichier host.json ci-dessous contient seulement les paramètres pour cette liaison. Pour plus d’informations sur les paramètres de configuration globale, consultez [Informations de référence sur le fichier host.json pour la version d’Azure Functions](functions-host-json.md).

> [!NOTE]
> Pour obtenir une référence de host.json dans Functions 1.x, consultez [Informations de référence sur le fichier host.json pour Azure Functions 1.x](functions-host-json-v1.md).

```json
{
    "version": "2.0",
    "extensions": {
        "serviceBus": {
            "prefetchCount": 100,
            "messageHandlerOptions": {
                "autoComplete": true,
                "maxConcurrentCalls": 32,
                "maxAutoRenewDuration": "00:05:00"
            },
            "sessionHandlerOptions": {
                "autoComplete": false,
                "messageWaitTimeout": "00:00:30",
                "maxAutoRenewDuration": "00:55:00",
                "maxConcurrentSessions": 16
            }
        }
    }
}
```

Si vous avez défini `isSessionsEnabled` sur `true`, les options `sessionHandlerOptions` sont respectées.  Si vous avez défini `isSessionsEnabled` sur `false`, les options `messageHandlerOptions` sont respectées.

|Propriété  |Default | Description |
|---------|---------|---------|
|prefetchCount|0|Obtient ou définit le nombre de messages que le destinataire des messages peut demander simultanément.|
|maxAutoRenewDuration|00:05:00|Durée maximale pendant laquelle le verrouillage de message doit être renouvelé automatiquement.|
|autoComplete|true|Indique si le déclencheur doit terminer l’appel automatiquement après le traitement, ou si le code de la fonction termine manuellement l’appel.<br><br>La définition de `false` est prise en charge uniquement dans C# .<br><br>Si la valeur est `true`, le déclencheur termine automatiquement le message si l’exécution de la fonction se termine correctement et abandonne le message dans le cas contraire.<br><br>Lorsque la valeur est `false`, il vous incombe d’appeler des méthodes [MessageReceiver](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver?view=azure-dotnet) pour terminer, abandonner ou mettre au rebut le message. Si une exception est levée (et qu’aucune des méthodes `MessageReceiver` n’est appelée), le verrou reste. Une fois le verrou expiré, le message est de nouveau mis en file d’attente avec `DeliveryCount` incrémenté, et le verrou est automatiquement renouvelé.<br><br>Dans les fonctions non C#, les exceptions de la fonction entraînent l’appel par le runtime de `abandonAsync` en arrière-plan. Si aucune exception ne se produit, `completeAsync` est appelé en arrière-plan. |
|maxConcurrentCalls|16|Nombre maximal d’appels simultanés pour le rappel que la pompe de messages doit initier par instance mise à l’échelle. Par défaut, le runtime Functions traite plusieurs messages simultanément.|
|maxConcurrentSessions|2000|Nombre maximal de sessions qui peuvent être traitées simultanément par instance mise à l’échelle.|

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter une fonction durant la création d’un message de file d’attente ou de rubrique Service Bus (déclencheur)](./functions-bindings-service-bus-trigger.md)
