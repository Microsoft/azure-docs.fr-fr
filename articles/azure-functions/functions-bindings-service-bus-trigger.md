---
title: Déclencheur Azure Service Bus pour Azure Functions
description: Apprenez à exécuter une fonction Azure lorsque des messages Azure Service Bus sont créés.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: cd0b73dd22e5e2cab720bb1a33e58e25e517b1f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90605013"
---
# <a name="azure-service-bus-trigger-for-azure-functions"></a>Déclencheur Azure Service Bus pour Azure Functions

Utilisez le déclencheur Service Bus pour répondre aux messages provenant d'une file d’attente ou d'une rubrique Service Bus. À partir de la version d’extension 3.1.0, vous pouvez déclencher une file d’attente ou une rubrique activée par une session.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-service-bus.md).

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui lit les [métadonnées du message](#message-metadata) et consigne un message de la file d’attente Service Bus :

```cs
[FunctionName("ServiceBusQueueTriggerCSharp")]                    
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
    string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    ILogger log)
{
    log.LogInformation($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
    log.LogInformation($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.LogInformation($"DeliveryCount={deliveryCount}");
    log.LogInformation($"MessageId={messageId}");
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur Service Bus dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction lit [les métadonnées du message](#message-metadata) et consigne un message de la file d’attente Service Bus.

Voici les données de liaison dans le fichier *function.json* :

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

Voici le code Script C# :

```cs
using System;

public static void Run(string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    TraceWriter log)
{
    log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");

    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"DeliveryCount={deliveryCount}");
    log.Info($"MessageId={messageId}");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur Service Bus dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction lit [les métadonnées du message](#message-metadata) et consigne un message de la file d’attente Service Bus. 

Voici les données de liaison dans le fichier *function.json* :

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

Voici le code de script JavaScript :

```javascript
module.exports = function(context, myQueueItem) {
    context.log('Node.js ServiceBus queue trigger function processed message', myQueueItem);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('DeliveryCount =', context.bindingData.deliveryCount);
    context.log('MessageId =', context.bindingData.messageId);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre comment lire un message de file d’attente Service Bus via un déclencheur.

Une liaison Service Bus est définie dans *function.json*, où le *type* est défini sur `serviceBusTrigger`.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "msg",
      "type": "serviceBusTrigger",
      "direction": "in",
      "queueName": "inputqueue",
      "connection": "AzureServiceBusConnectionString"
    }
  ]
}
```

Le code dans *_\_init_\_.py* déclare un paramètre `func.ServiceBusMessage`, ce qui vous permet de lire le message de la file d’attente dans votre fonction.

```python
import azure.functions as func

import logging
import json

def main(msg: func.ServiceBusMessage):
    logging.info('Python ServiceBus queue trigger processed message.')

    result = json.dumps({
        'message_id': msg.message_id,
        'body': msg.get_body().decode('utf-8'),
        'content_type': msg.content_type,
        'expiration_time': msg.expiration_time,
        'label': msg.label,
        'partition_key': msg.partition_key,
        'reply_to': msg.reply_to,
        'reply_to_session_id': msg.reply_to_session_id,
        'scheduled_enqueue_time': msg.scheduled_enqueue_time,
        'session_id': msg.session_id,
        'time_to_live': msg.time_to_live,
        'to': msg.to,
        'user_properties': msg.user_properties,
        'metadata' : msg.metadata
    })

    logging.info(result)
```

# <a name="java"></a>[Java](#tab/java)

La fonction Java suivante utilise l’annotation `@ServiceBusQueueTrigger` provenant de la [bibliothèque runtime de fonctions Java](/java/api/overview/azure/functions/runtime) afin de décrire la configuration d’un déclencheur de file d’attente Service Bus. La fonction récupère le message placé dans la file d’attente et l’ajoute dans les journaux.

```java
@FunctionName("sbprocessor")
 public void serviceBusProcess(
    @ServiceBusQueueTrigger(name = "msg",
                             queueName = "myqueuename",
                             connection = "myconnvarname") String message,
   final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
```

Les fonctions Java peuvent également être déclenchées lorsqu’un message est ajouté à une rubrique Service Bus. L’exemple suivant utilise l’annotation `@ServiceBusTopicTrigger` pour décrire la configuration du déclencheur.

```java
@FunctionName("sbtopicprocessor")
    public void run(
        @ServiceBusTopicTrigger(
            name = "message",
            topicName = "mytopicname",
            subscriptionName = "mysubscription",
            connection = "ServiceBusConnection"
        ) String message,
        final ExecutionContext context
    ) {
        context.getLogger().info(message);
    }
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez les attributs suivants pour configurer un déclencheur de Service Bus :

* [ServiceBusTriggerAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusTriggerAttribute.cs)

  Le constructeur de l’attribut prend le nom de la file d’attente ou la rubrique et l’abonnement. Dans Azure Functions version 1.x, vous pouvez également spécifier les droits d’accès de la connexion. Si vous ne spécifiez pas de droits d’accès, la valeur par défaut est `Manage`. Pour en savoir plus, consultez la section [Déclencheur - configuration](#configuration).

  Voici un exemple montrant l’attribut utilisé avec un paramètre de chaîne :

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue")] string myQueueItem, ILogger log)
  {
      ...
  }
  ```

  Étant donné que la propriété `Connection` n’est pas définie, Functions recherche un paramètre d’application nommé `AzureWebJobsServiceBus`, qui est le nom par défaut de la chaîne de connexion Service Bus. Vous pouvez également définir la propriété `Connection` pour spécifier le nom d’un paramètre d’application contenant la chaîne de connexion Service Bus à utiliser, comme illustré dans l’exemple suivant :

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
      string myQueueItem, ILogger log)
  {
      ...
  }
  ```

  Pour obtenir un exemple complet, voir [Exemple de déclencheur](#example).

* [ServiceBusAccountAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAccountAttribute.cs)

  Fournit une autre manière de spécifier le compte Service Bus à utiliser. Le constructeur prend le nom d’un paramètre d’application comportant une chaîne de connexion Service Bus. L’attribut peut être appliqué au niveau du paramètre, de la méthode ou de la classe. L’exemple suivant montre le niveau de la classe et celui de la méthode :

  ```csharp
  [ServiceBusAccount("ClassLevelServiceBusAppSetting")]
  public static class AzureFunctions
  {
      [ServiceBusAccount("MethodLevelServiceBusAppSetting")]
      [FunctionName("ServiceBusQueueTriggerCSharp")]
      public static void Run(
          [ServiceBusTrigger("myqueue", AccessRights.Manage)] 
          string myQueueItem, ILogger log)
  {
      ...
  }
  ```

Le compte Service Bus à utiliser est déterminé dans l’ordre suivant :

* La propriété `Connection` de l’attribut `ServiceBusTrigger`.
* L’attribut `ServiceBusAccount` appliqué au même paramètre que l’attribut `ServiceBusTrigger`.
* L’attribut `ServiceBusAccount` appliqué à la fonction.
* L’attribut `ServiceBusAccount` appliqué à la classe.
* Le paramètre d’application « AzureWebJobsServiceBus ».

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

L’annotation `ServiceBusQueueTrigger` vous permet de créer une fonction qui s’exécute lors de la création d’un message de file d’attente Service Bus. Les options de configuration disponibles sont le nom de la file d’attente et le nom de la chaîne de connexion.

L’annotation `ServiceBusTopicTrigger` vous permet de désigner une rubrique et un abonnement pour cibler les données qui déclenchent la fonction.

Pour plus de détails, voir l’[exemple](#example) de déclencheur.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `ServiceBusTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Doit être défini sur « serviceBusTrigger ». Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction** | n/a | Doit être défini sur « in ». Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. |
|**name** | n/a | Nom de la variable qui représente le message de la file d’attente ou de la rubrique dans le code de la fonction. |
|**queueName**|**QueueName**|Nom de la file d’attente à surveiller.  Défini uniquement en cas de surveillance d’une file d’attente, ne s’applique pas à une rubrique.
|**topicName**|**TopicName**|Nom de la rubrique à surveiller. Défini uniquement en cas de surveillance d’une rubrique, ne s’applique pas à une file d’attente.|
|**subscriptionName**|**SubscriptionName**|Nom de l’abonnement à surveiller. Défini uniquement en cas de surveillance d’une rubrique, ne s’applique pas à une file d’attente.|
|**connection**|**Connection**|Nom d’un paramètre d’application comportant la chaîne de connexion Service Bus à utiliser pour cette liaison. Si le nom du paramètre d’application commence par « AzureWebJobs », vous ne pouvez spécifier que le reste du nom. Par exemple, si vous définissez `connection` sur « MyServiceBus », le runtime Functions recherche un paramètre d’application qui est nommé « AzureWebJobsMyServiceBus ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion Service Bus par défaut dans le paramètre d’application nommé « AzureWebJobsServiceBus ».<br><br>Pour obtenir une chaîne de connexion, suivez les étapes indiquées à la section [Obtenir les informations d’identification de gestion](../service-bus-messaging/service-bus-quickstart-portal.md#get-the-connection-string). La chaîne de connexion doit être destinée à un espace de noms Service Bus, et non limitée à une file d’attente ou une rubrique spécifique. |
|**accessRights**|**y accéder**|Droits d’accès de la chaîne de connexion. Les valeurs disponibles sont `manage` et `listen`. La valeur par défaut est `manage`, ce qui indique que `connection` a l'autorisation **Gérer**. Si vous utilisez une chaîne de connexion qui n’a pas l'autorisation **Gérer**, définissez `accessRights` sur « écouter ». Sinon, le runtime Functions pourrait échouer à effectuer des opérations qui nécessitent des droits de gestion. Dans Azure Functions versions 2.x et ultérieures, cette propriété n’est pas disponible parce que la version la plus récente du Kit de développement logiciel (SDK) Service Bus ne prend pas en charge les opérations de gestion.|
|**isSessionsEnabled**|**IsSessionsEnabled**|`true` en cas de connexion à une file d’attente ou à un abonnement [prenant en charge la session](../service-bus-messaging/message-sessions.md). Sinon `false`, qui est la valeur par défaut.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Les types de paramètres suivants sont disponibles pour le message de la file d’attente ou de la rubrique :

* `string` -Si le message est un texte.
* `byte[]` - Utile pour les données binaires.
* Un type personnalisé - Si le message contient JSON, Azure Functions essaie de désérialiser les données JSON.
* `BrokeredMessage` - Vous donne le message désérialisé avec la méthode [BrokeredMessage.GetBody\<T>()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody?view=azure-dotnet#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1).
* [`MessageReceiver`](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver?view=azure-dotnet) - Utilisé pour recevoir et accuser réception des messages du conteneur de messages (requis lorsque [`autoComplete`](functions-bindings-service-bus-output.md#hostjson-settings) est défini sur `false`)

Ces types de paramètres concernent Azure Functions version 1.x. Pour les versions 2.x et ultérieures, utilisez [`Message`](/dotnet/api/microsoft.azure.servicebus.message) au lieu de `BrokeredMessage`.

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les types de paramètres suivants sont disponibles pour le message de la file d’attente ou de la rubrique :

* `string` -Si le message est un texte.
* `byte[]` - Utile pour les données binaires.
* Un type personnalisé - Si le message contient JSON, Azure Functions essaie de désérialiser les données JSON.
* `BrokeredMessage` - Vous donne le message désérialisé avec la méthode [BrokeredMessage.GetBody\<T>()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody?view=azure-dotnet#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1).

Ces paramètres concernent Azure Functions version 1.x ; pour 2.x et ultérieur, utilisez [`Message`](/dotnet/api/microsoft.azure.servicebus.message) au lieu de `BrokeredMessage`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Accédez au message de la file d’attente ou de la rubrique à l’aide de `context.bindings.<name from function.json>`. Le message Service Bus est passé à la fonction en tant que chaîne ou en tant qu’objet JSON.

# <a name="python"></a>[Python](#tab/python)

Le message de la file d’attente est disponible pour la fonction via un paramètre de type `func.ServiceBusMessage`. Le message Service Bus est passé à la fonction en tant que chaîne ou en tant qu’objet JSON.

# <a name="java"></a>[Java](#tab/java)

Le message de Service Bus entrant est disponible via un paramètre `ServiceBusQueueMessage` ou `ServiceBusTopicMessage`.

[Pour plus de détails, voir les exemples](#example).

---

## <a name="poison-messages"></a>Messages incohérents

La gestion des messages incohérents ne peut pas être contrôlée ou configurée dans Azure Functions. Service Bus gère lui-même les messages incohérents.

## <a name="peeklock-behavior"></a>Comportement de PeekLock

Le runtime Functions reçoit un message en [mode PeekLock](../service-bus-messaging/service-bus-performance-improvements.md#receive-mode). Il appelle l’élément `Complete` sur le message si la fonction se termine correctement. Si la fonction échoue, il appelle l’élément `Abandon`. Si la fonction s’exécute au-delà du délai imparti `PeekLock`, le verrou est automatiquement renouvelé tant que la fonction s’exécute. 

Le paramètre `maxAutoRenewDuration` peut être configuré dans *host.json*, qui mappe à [OnMessageOptions.MaxAutoRenewDuration](/dotnet/api/microsoft.azure.servicebus.messagehandleroptions.maxautorenewduration?view=azure-dotnet). La valeur maximale autorisée pour ce paramètre est de 5 minutes, conformément à la documentation de Service Bus, tandis que vous pouvez augmenter la limite de temps de Functions jusqu’à 10 minutes (5 minutes par défaut). Pour les fonctions de Service Bus, vous ne voudrez pas le faire, car vous dépasseriez la limite de renouvellement de Service Bus.

## <a name="message-metadata"></a>Métadonnées de message

Le déclencheur Service Bus fournit plusieurs [propriétés de métadonnées](./functions-bindings-expressions-patterns.md#trigger-metadata). Ces propriétés peuvent être utilisées dans les expressions de liaison dans d’autres liaisons ou en tant que paramètres dans votre code. Ces propriétés sont membres de la classe [Message](/dotnet/api/microsoft.azure.servicebus.message?view=azure-dotnet).

|Propriété|Type|Description|
|--------|----|-----------|
|`ContentType`|`string`|Identificateur de type de contenu utilisé par l’expéditeur et le récepteur pour une logique spécifique à l’application.|
|`CorrelationId`|`string`|L’ID de corrélation.|
|`DeadLetterSource`|`string`|La source de lettre morte.|
|`DeliveryCount`|`Int32`|Le nombre de remises.|
|`EnqueuedTimeUtc`|`DateTime`|Le temps de file d’attente en UTC.|
|`ExpiresAtUtc`|`DateTime`|Le délai d'expiration en UTC.|
|`Label`|`string`|L’étiquette spécifique de l’application.|
|`MessageId`|`string`|Valeur définie par l’utilisateur que Service Bus peut utiliser pour identifier les messages en double, si cette fonctionnalité est activée.|
|`MessageReceiver`|`MessageReceiver`|Récepteur du message Service Bus. Peut être utilisé pour abandonner, terminer ou mettre en file d’attente le message.|
|`MessageSession`|`MessageSession`|Récepteur de messages spécifiquement pour les files d’attente et les rubriques activées par la session.|
|`ReplyTo`|`string`|L’adresse de file d’attente de réponse.|
|`SequenceNumber`|`long`|Le numéro unique attribué à un message par Service Bus.|
|`To`|`string`|L’adresse de destination.|
|`UserProperties`|`IDictionary<string, object>`|Propriétés définies par l’expéditeur.|

Consultez les [exemples de code](#example) qui utilisent ces propriétés précédemment dans cet article.

## <a name="next-steps"></a>Étapes suivantes

- [Envoyer des messages Azure Service Bus à partir d’Azure Functions (liaison de sortie)](./functions-bindings-service-bus-output.md)
