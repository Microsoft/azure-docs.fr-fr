---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 03/05/2019
ms.author: cshoe
ms.openlocfilehash: 0cd514c852e13b83a679821ca2d940e4ed112bd8
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95560310"
---
Utilisez le déclencheur de fonction pour répondre à un événement envoyé à un flux d’événements d’un hub d’événements. Vous devez disposer de l’accès en lecture au hub d’événements sous-jacent pour configurer le déclencheur. Une fois la fonction déclenchée, le message passé à la fonction est du type chaîne.

## <a name="scaling"></a>Mise à l'échelle

Chaque instance d’une fonction déclenchée par un événement est sauvegardée par une seule instance [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor). Le déclencheur (alimenté par Event Hubs) garantit qu’une seule instance [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor) peut obtenir un bail sur une partition donnée.

Prenons, par exemple, le hub d’événements suivant :

* 10 partitions
* 1 000 événements répartis uniformément sur toutes les partitions, avec 100 messages dans chaque partition

Lorsque votre fonction est activée pour la première fois, il n’existe qu’une seule instance de cette fonction. Nous appellerons la première instance de fonction `Function_0`. La fonction `Function_0` comprend une seule instance de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor) qui détient un bail sur les dix partitions. Cette instance lit les événements des partitions 0 à 9. À partir de là, l’un des événements suivants se produit :

* **Les nouvelles instances de fonction ne sont pas nécessaires** : `Function_0` est capable de traiter l’ensemble des 1 000 événements avant que la logique de mise à l’échelle Functions ne prenne effet. Dans ce cas, l’intégralité des 1 000 messages sont traités par `Function_0`.

* **Une instance de fonction supplémentaire est ajoutée** : la logique de mise à l’échelle Functions détermine que si `Function_0` a plus de messages qu’elle ne peut en traiter, une instance d’application de fonction (`Function_1`) est ajoutée. Cette nouvelle fonction a également une instance associée de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor). Comme les Event Hubs sous-jacents détectent qu’une nouvelle instance d’hôte tente de lire des messages, ils équilibrent la charge des partitions entre les instances d’hôte. Par exemple, les partitions 0 à 4 peuvent être affectées à `Function_0`, et les partitions 5 à 9 à `Function_1`.

* **N instances de fonction supplémentaires sont ajoutées** : Si la logique de mise à l’échelle de Functions détermine que `Function_0` et `Function_1` ont plus de messages qu’elles ne peuvent en traiter, de nouvelles instances d’application de fonction `Functions_N` sont créées.  Des applications sont créées jusqu’au point où `N` est supérieur au nombre de partitions de hub d’événements. Dans notre exemple, Event Hubs équilibre la charge des partitions, en l’occurrence, sur les instances `Function_0`...`Functions_9`.

Au fur et à mesure de la mise à l’échelle, `N` instances est un nombre supérieur à celui des partitions du Event Hub. Ce modèle est utilisé pour garantir que les instances [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor) sont toujours disponibles pour obtenir des verrous sur les partitions, à mesure que celles-ci sont mises à disposition par les autres instances. Vous n’êtes facturé que pour les ressources utilisées lors de l’exécution de l’instance de la fonction. En d’autres mots, vous n’êtes pas facturé pour cet approvisionnement excessif.

Quand toutes les exécutions de fonction se terminent (avec ou sans erreurs), des points de contrôle sont ajoutés au compte de stockage associé. Une fois les points de contrôle correctement créés, les 1 000 messages ne sont plus jamais récupérés.

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant illustre un code [de fonction C#](../articles/azure-functions/functions-dotnet-class-library.md) qui consigne le corps du message du déclencheur de hub d’événements.

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, ILogger log)
{
    log.LogInformation($"C# function triggered to process a message: {myEventHubMessage}");
}
```

Pour accéder aux [métadonnées d’événement](#event-metadata) en code de fonction, effectuez une liaison avec l’objet [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) (nécessite une instruction using pour `Microsoft.Azure.EventHubs`). Vous pouvez également accéder aux mêmes propriétés à l’aide d’expressions de liaison dans la signature de la méthode.  L’exemple suivant montre deux façons d’obtenir les mêmes données :

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run(
    [EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData myEventHubMessage,
    DateTime enqueuedTimeUtc,
    Int64 sequenceNumber,
    string offset,
    ILogger log)
{
    log.LogInformation($"Event: {Encoding.UTF8.GetString(myEventHubMessage.Body)}");
    // Metadata accessed by binding to EventData
    log.LogInformation($"EnqueuedTimeUtc={myEventHubMessage.SystemProperties.EnqueuedTimeUtc}");
    log.LogInformation($"SequenceNumber={myEventHubMessage.SystemProperties.SequenceNumber}");
    log.LogInformation($"Offset={myEventHubMessage.SystemProperties.Offset}");
    // Metadata accessed by using binding expressions in method parameters
    log.LogInformation($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.LogInformation($"SequenceNumber={sequenceNumber}");
    log.LogInformation($"Offset={offset}");
}
```

Pour recevoir des événements en lot, transformez `string` ou `EventData` en tableau.  

> [!NOTE]
> Lors de la réception dans un lot, vous ne pouvez pas établir de liaison avec les paramètres de la méthode comme dans l’exemple ci-dessus avec `DateTime enqueuedTimeUtc` et vous devez les recevoir de chaque objet `EventData`.  

```cs
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData[] eventHubMessages, ILogger log)
{
    foreach (var message in eventHubMessages)
    {
        log.LogInformation($"C# function triggered to process a message: {Encoding.UTF8.GetString(message.Body)}");
        log.LogInformation($"EnqueuedTimeUtc={message.SystemProperties.EnqueuedTimeUtc}");
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant illustre une liaison de déclencheur de hub d’événements dans un fichier *function.json* et une [fonction de script C#](../articles/azure-functions/functions-reference-csharp.md) qui utilise la liaison. La fonction consigne le corps du message du déclencheur de hub d’événements.

Les exemples suivants illustrent les données de liaison Event Hubs dans le fichier *function.json*.

### <a name="version-2x-and-higher"></a>Versions 2.x et ultérieures

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

### <a name="version-1x"></a>Version 1.x

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Voici le code Script C# :

```cs
using System;

public static void Run(string myEventHubMessage, TraceWriter log)
{
    log.Info($"C# function triggered to process a message: {myEventHubMessage}");
}
```

Pour accéder aux [métadonnées d’événement](#event-metadata) en code de fonction, effectuez une liaison avec l’objet [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) (nécessite une instruction using pour `Microsoft.Azure.EventHubs`). Vous pouvez également accéder aux mêmes propriétés à l’aide d’expressions de liaison dans la signature de la méthode.  L’exemple suivant montre deux façons d’obtenir les mêmes données :

```cs
#r "Microsoft.Azure.EventHubs"

using System.Text;
using System;
using Microsoft.ServiceBus.Messaging;
using Microsoft.Azure.EventHubs;

public static void Run(EventData myEventHubMessage,
    DateTime enqueuedTimeUtc,
    Int64 sequenceNumber,
    string offset,
    TraceWriter log)
{
    log.Info($"Event: {Encoding.UTF8.GetString(myEventHubMessage.Body)}");
    log.Info($"EnqueuedTimeUtc={myEventHubMessage.SystemProperties.EnqueuedTimeUtc}");
    log.Info($"SequenceNumber={myEventHubMessage.SystemProperties.SequenceNumber}");
    log.Info($"Offset={myEventHubMessage.SystemProperties.Offset}");

    // Metadata accessed by using binding expressions
    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"SequenceNumber={sequenceNumber}");
    log.Info($"Offset={offset}");
}
```

Pour recevoir des événements en lot, transformez `string` ou `EventData` en tableau :

```cs
public static void Run(string[] eventHubMessages, TraceWriter log)
{
    foreach (var message in eventHubMessages)
    {
        log.Info($"C# function triggered to process a message: {message}");
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant illustre une liaison de déclencheur Event Hub dans un fichier *function.json* et une [fonction JavaScript](../articles/azure-functions/functions-reference-node.md) qui utilise la liaison. La fonction lit les [métadonnées d’événement](#event-metadata) et enregistre le message.

Les exemples suivants illustrent les données de liaison Event Hubs dans le fichier *function.json*.

### <a name="version-2x-and-higher"></a>Versions 2.x et ultérieures

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

### <a name="version-1x"></a>Version 1.x

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, myEventHubMessage) {
    context.log('Function triggered to process a message: ', myEventHubMessage);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('SequenceNumber =', context.bindingData.sequenceNumber);
    context.log('Offset =', context.bindingData.offset);

    context.done();
};
```

Pour recevoir les événements dans un lot, définissez `cardinality` sur `many` dans le fichier *function.json*, comme le montrent les exemples suivants.

### <a name="version-2x-and-higher"></a>Versions 2.x et ultérieures

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

### <a name="version-1x"></a>Version 1.x

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "path": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, eventHubMessages) {
    context.log(`JavaScript eventhub trigger function called for message array ${eventHubMessages}`);

    eventHubMessages.forEach((message, index) => {
        context.log(`Processed message ${message}`);
        context.log(`EnqueuedTimeUtc = ${context.bindingData.enqueuedTimeUtcArray[index]}`);
        context.log(`SequenceNumber = ${context.bindingData.sequenceNumberArray[index]}`);
        context.log(`Offset = ${context.bindingData.offsetArray[index]}`);
    });

    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant illustre une liaison de déclencheur Event Hub dans un fichier *function.json* et une [fonction Python](../articles/azure-functions/functions-reference-python.md) qui utilise la liaison. La fonction lit les [métadonnées d’événement](#event-metadata) et enregistre le message.

Les exemples suivants illustrent les données de liaison Event Hubs dans le fichier *function.json*.

```json
{
  "type": "eventHubTrigger",
  "name": "event",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Voici le code Python :

```python
import logging
import azure.functions as func


def main(event: func.EventHubEvent):
    logging.info('Function triggered to process a message: ', event.get_body())
    logging.info('  EnqueuedTimeUtc =', event.enqueued_time)
    logging.info('  SequenceNumber =', event.sequence_number)
    logging.info('  Offset =', event.offset)

    # Metadata
    for key in event.metadata:
        logging.info(f'Metadata: {key} = ', event.metadata[key])
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant illustre une liaison de déclencheur Event Hub qui journalise le corps du message du déclencheur Event Hub.

```java
@FunctionName("ehprocessor")
public void eventHubProcessor(
  @EventHubTrigger(name = "msg",
                  eventHubName = "myeventhubname",
                  connection = "myconnvarname") String message,
       final ExecutionContext context )
       {
          context.getLogger().info(message);
 }
```

 Dans la [bibliothèque du runtime des fonctions Java](/java/api/overview/azure/functions/runtime), utilisez l’annotation `EventHubTrigger` sur les paramètres dont la valeur proviendrait d’Event Hub. Les paramètres ayant ces annotations entraînent l’exécution de la fonction quand un événement se produit.  Vous pouvez utiliser cette annotation avec des types Java natifs, des objets POJO ou des valeurs Null à l’aide de `Optional<T>`.

 ---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](../articles/azure-functions/functions-dotnet-class-library.md), utilisez l’attribut [EventHubTriggerAttribute](https://github.com/Azure/azure-functions-eventhubs-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs/EventHubTriggerAttribute.cs).

Le constructeur de l’attribut prend le nom du hub d’événements, le nom du groupe de consommateurs et le nom d’un paramètre d’application qui contient la chaîne de connexion. Pour plus d’informations sur ces paramètres, consultez la [section de configuration du déclencheur](#configuration). Voici un exemple d’attribut `EventHubTriggerAttribute` :

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, ILogger log)
{
    ...
}
```

Pour obtenir un exemple complet, consultez [Déclencheur - exemple C#](#example).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

À partir de la [bibliothèque du runtime des fonctions](/java/api/overview/azure/functions/runtime) Java, utilisez l’annotation [EventHubTrigger](/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger) sur les paramètres dont la valeur proviendrait d’Event Hub. Les paramètres ayant ces annotations entraînent l’exécution de la fonction quand un événement se produit. Vous pouvez utiliser cette annotation avec des types Java natifs, des objets POJO ou des valeurs Null à l’aide de `Optional<T>`.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `EventHubTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Cette propriété doit être définie sur `eventHubTrigger`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction** | n/a | Cette propriété doit être définie sur `in`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. |
|**name** | n/a | Nom de la variable qui représente l’élément d’événement dans le code de la fonction. |
|**path** |**EventHubName** | Functions 1.x uniquement. Nom du hub d’événements. Lorsque le nom d’Event Hub est également présent dans la chaîne de connexion, sa valeur remplace cette propriété lors de l’exécution. |
|**eventHubName** |**EventHubName** | Functions 2.x et versions ultérieures. Nom du hub d’événements. Lorsque le nom d’Event Hub est également présent dans la chaîne de connexion, sa valeur remplace cette propriété lors de l’exécution. Peut être référencé via les [paramètres d’application](../articles/azure-functions/functions-bindings-expressions-patterns.md#binding-expressions---app-settings) `%eventHubName%` |
|**consumerGroup** |**ConsumerGroup** | Propriété facultative qui définit le [groupe de consommateurs](../articles/event-hubs/event-hubs-features.md#event-consumers) utilisé pour l’abonnement à des événements dans le hub. En cas d’omission, le groupe de consommateurs `$Default` est utilisé. |
|**cardinalité** | n/a | Utilisé pour toutes les autres langages non-C#. Définissez sur `many` afin d’activer le traitement par lot.  Si omis ou défini sur `one`, un message unique est transmis à la fonction.<br><br>En C#, cette propriété est affectée automatiquement chaque fois que le déclencheur a un tableau pour le type.|
|**connection** |**Connection** | Le nom d’un paramètre d’application qui contient la chaîne de connexion à l’espace de noms du hub d’événements. Copiez cette chaîne de connexion en cliquant sur le bouton **Informations de connexion** pour [l’espace de noms](../articles/event-hubs/event-hubs-create.md#create-an-event-hubs-namespace), et non pour le hub d’événements lui-même. Cette chaîne de connexion doit avoir au moins des droits de lecture pour activer le déclencheur.|

[!INCLUDE [app settings to local.settings.json](../articles/azure-functions/../../includes/functions-app-settings-local.md)]

## <a name="event-metadata"></a>Métadonnées d’événement

Le déclencheur Event Hubs fournit plusieurs [propriétés de métadonnées](../articles/azure-functions/./functions-bindings-expressions-patterns.md). Les propriétés de métadonnées peuvent être utilisées dans le cadre d’expressions de liaison dans d’autres liaisons ou en tant que paramètres dans votre code. Les propriétés proviennent de la classe [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata).

|Propriété|Type|Description|
|--------|----|-----------|
|`PartitionContext`|[PartitionContext](/dotnet/api/microsoft.servicebus.messaging.partitioncontext)|Instance `PartitionContext`.|
|`EnqueuedTimeUtc`|`DateTime`|Le temps de file d’attente en UTC.|
|`Offset`|`string`|Le décalage des données par rapport au flux de données de la partition Event Hub. Le décalage est une marque ou un identificateur pour un événement au sein du flux Event Hubs. L’identificateur est unique au sein d’une partition du flux Event Hubs.|
|`PartitionKey`|`string`|La partition vers laquelle les données d’événement doivent être envoyées.|
|`Properties`|`IDictionary<String,Object>`|Les propriétés d’utilisateur pour les données d’événements.|
|`SequenceNumber`|`Int64`|Le numéro de séquence logique de l’événement.|
|`SystemProperties`|`IDictionary<String,Object>`|Les propriétés système, y compris les données d’événement.|

Consultez les [exemples de code](#example) qui utilisent ces propriétés précédemment dans cet article.

## <a name="hostjson-properties"></a>Propriétés host.json
<a name="host-json"></a>

Le fichier [host.json](../articles/azure-functions/functions-host-json.md#eventhub) contient les paramètres qui contrôlent le comportement du déclencheur Event Hubs. La configuration diffère selon la version d’Azure Functions.

[!INCLUDE [functions-host-json-event-hubs](../articles/azure-functions/../../includes/functions-host-json-event-hubs.md)]