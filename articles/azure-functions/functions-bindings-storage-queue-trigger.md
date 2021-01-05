---
title: Déclencheur Stockage File d’attente Azure pour Azure Functions
description: Apprenez à exécuter une fonction Azure lorsque des données Stockage File d’attente Azure changent.
author: craigshoemaker
ms.topic: reference
ms.date: 02/18/2020
ms.author: cshoe
ms.custom: devx-track-csharp, cc996988-fb4f-47, devx-track-python
ms.openlocfilehash: 95560801d4132735435e4d45e8a588476636ec38
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96001233"
---
# <a name="azure-queue-storage-trigger-for-azure-functions"></a>Déclencheur Stockage File d’attente Azure pour Azure Functions

Le déclencheur de stockage de file d’attente exécute une fonction lorsque des messages sont ajoutés à Stockage File d’attente Azure.

## <a name="encoding"></a>Encodage

Les fonctions attendent une chaîne codée en *base64*. Tout ajustement du type d’encodage (afin de préparer les données sous forme de chaîne codée en *base64*) doit être effectué dans le service appelant.

## <a name="example"></a>Exemple

Utilisez le déclencheur de file d’attente pour démarrer une fonction lorsqu’un nouvel élément est reçu sur une file d’attente. Le message de file d’attente est fourni comme entrée pour la fonction.

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre un code de [fonction C#](functions-dotnet-class-library.md) qui interroge la file d’attente `myqueue-items` et écrit un journal chaque fois qu’un élément de la file d’attente est traité.

```csharp
public static class QueueFunctions
{
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur de file d’attente dans un fichier *function.json*, ainsi que le code du [script C# (.csx)](functions-reference-csharp.md) qui utilise cette liaison. La fonction interroge la file d’attente `myqueue-items` et écrit un journal chaque fois qu’un élément de la file d’attente est traité.

Voici le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

Voici le code Script C# :

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.Extensions.Logging;
using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, 
    DateTimeOffset expirationTime, 
    DateTimeOffset insertionTime, 
    DateTimeOffset nextVisibleTime,
    string queueTrigger,
    string id,
    string popReceipt,
    int dequeueCount,
    ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem.AsString}\n" +
        $"queueTrigger={queueTrigger}\n" +
        $"expirationTime={expirationTime}\n" +
        $"insertionTime={insertionTime}\n" +
        $"nextVisibleTime={nextVisibleTime}\n" +
        $"id={id}\n" +
        $"popReceipt={popReceipt}\n" + 
        $"dequeueCount={dequeueCount}");
}
```

La section [utilisation](#usage) explique `myQueueItem`, qui est nommé par la propriété `name` dans function.json.  La [section sur les métadonnées de message](#message-metadata) détaille toutes les autres variables indiquées.

# <a name="java"></a>[Java](#tab/java)

L’exemple Java suivant montre une fonction de déclencheur de file d’attente de stockage, qui consigne le message déclenché placé dans la file d’attente `myqueuename`.

 ```java
 @FunctionName("queueprocessor")
 public void run(
    @QueueTrigger(name = "msg",
                   queueName = "myqueuename",
                   connection = "myconnvarname") String message,
     final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
 ```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur de file d’attente dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction interroge la file d’attente `myqueue-items` et écrit un journal chaque fois qu’un élément de la file d’attente est traité.

Voici le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

La section [configuration](#configuration) décrit ces propriétés.

> [!NOTE]
> Le paramètre de nom se reflète en tant que `context.bindings.<name>` dans le code JavaScript qui contient la charge utile de l’élément de file d’attente. Cette charge utile est également passée comme deuxième paramètre à la fonction.

Voici le code JavaScript :

```javascript
module.exports = async function (context, message) {
    context.log('Node.js queue trigger function processed work item', message);
    // OR access using context.bindings.<name>
    // context.log('Node.js queue trigger function processed work item', context.bindings.myQueueItem);
    context.log('expirationTime =', context.bindingData.expirationTime);
    context.log('insertionTime =', context.bindingData.insertionTime);
    context.log('nextVisibleTime =', context.bindingData.nextVisibleTime);
    context.log('id =', context.bindingData.id);
    context.log('popReceipt =', context.bindingData.popReceipt);
    context.log('dequeueCount =', context.bindingData.dequeueCount);
    context.done();
};
```

La section [utilisation](#usage) explique `myQueueItem`, qui est nommé par la propriété `name` dans function.json.  La [section sur les métadonnées de message](#message-metadata) détaille toutes les autres variables indiquées.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L’exemple suivant montre comment lire un message de file d’attente passé à une fonction via un déclencheur.

Un déclencheur de file d’attente de stockage est défini dans le fichier *function.json*, où `type` est défini sur `queueTrigger`.

```json
{
  "bindings": [
    {
      "name": "QueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

Le code du fichier dans *Run.ps1* déclare un paramètre `$QueueItem`, ce qui vous permet de lire le message de la file d’attente dans votre fonction.

```powershell
# Input bindings are passed in via param block.
param([string] $QueueItem, $TriggerMetadata)

# Write out the queue message and metadata to the information log.
Write-Host "PowerShell queue trigger function processed work item: $QueueItem"
Write-Host "Queue item expiration time: $($TriggerMetadata.ExpirationTime)"
Write-Host "Queue item insertion time: $($TriggerMetadata.InsertionTime)"
Write-Host "Queue item next visible time: $($TriggerMetadata.NextVisibleTime)"
Write-Host "ID: $($TriggerMetadata.Id)"
Write-Host "Pop receipt: $($TriggerMetadata.PopReceipt)"
Write-Host "Dequeue count: $($TriggerMetadata.DequeueCount)"
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre comment lire un message de file d’attente passé à une fonction via un déclencheur.

Un déclencheur de file d’attente de stockage est défini dans *function.json*, où *Type* est défini sur `queueTrigger`.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "msg",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages",
      "connection": "AzureStorageQueuesConnectionString"
    }
  ]
}
```

Le code dans *_\_init_\_.py* déclare un paramètre `func.QueueMessage` qui vous permet de lire le message de la file d’attente dans votre fonction.

```python
import logging
import json

import azure.functions as func

def main(msg: func.QueueMessage):
    logging.info('Python queue trigger function processed a queue item.')

    result = json.dumps({
        'id': msg.id,
        'body': msg.get_body().decode('utf-8'),
        'expiration_time': (msg.expiration_time.isoformat()
                            if msg.expiration_time else None),
        'insertion_time': (msg.insertion_time.isoformat()
                           if msg.insertion_time else None),
        'time_next_visible': (msg.time_next_visible.isoformat()
                              if msg.time_next_visible else None),
        'pop_receipt': msg.pop_receipt,
        'dequeue_count': msg.dequeue_count
    })

    logging.info(result)
```

 ---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez les attributs suivants pour configurer un déclencheur de file d’attente :

* [QueueTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues/QueueTriggerAttribute.cs)

  Le constructeur de l’attribut prend le nom de la file d’attente à surveiller, comme illustré dans l’exemple suivant :

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items")] string myQueueItem, 
      ILogger log)
  {
      ...
  }
  ```

  Vous pouvez définir la propriété `Connection` pour spécifier le paramètre d’application qui contient la chaîne de connexion du compte de stockage à utiliser, comme illustré dans l’exemple suivant :

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem, 
      ILogger log)
  {
      ....
  }
  ```

  Pour obtenir un exemple complet, consultez [exemple](#example).

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Fournit une autre manière de spécifier le compte de stockage à utiliser. Le constructeur prend le nom d’un paramètre d’application comportant une chaîne de connexion de stockage. L’attribut peut être appliqué au niveau du paramètre, de la méthode ou de la classe. L’exemple suivant montre le niveau de la classe et celui de la méthode :

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("QueueTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

Le compte de stockage à utiliser est déterminé dans l’ordre suivant :

* La propriété `Connection` de l’attribut `QueueTrigger`.
* L’attribut `StorageAccount` appliqué au même paramètre que l’attribut `QueueTrigger`.
* L’attribut `StorageAccount` appliqué à la fonction.
* L’attribut `StorageAccount` appliqué à la classe.
* Le paramètre d’application « AzureWebJobsStorage ».

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="java"></a>[Java](#tab/java)

L’annotation `QueueTrigger` vous donne accès à la file d’attente qui déclenche la fonction. L’exemple suivant rend le message de file d’attente disponible pour la fonction via le paramètre `message`.

```java
package com.function;
import com.microsoft.azure.functions.annotation.*;
import java.util.Queue;
import com.microsoft.azure.functions.*;

public class QueueTriggerDemo {
    @FunctionName("QueueTriggerDemo")
    public void run(
        @QueueTrigger(name = "message", queueName = "messages", connection = "MyStorageConnectionAppSetting") String message,
        final ExecutionContext context
    ) {
        context.getLogger().info("Queue message: " + message);
    }
}
```

| Propriété    | Description |
|-------------|-----------------------------|
|`name`       | Déclare le nom du paramètre dans la signature de la fonction. Quand la fonction est déclenchée, la valeur de ce paramètre a le contenu du message de la file d’attente. |
|`queueName`  | Déclare le nom de la file d’attente dans le compte de stockage. |
|`connection` | Pointe vers la chaîne de connexion du compte de stockage. |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les attributs ne sont pas pris en charge par PowerShell.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `QueueTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a| Cette propriété doit être définie sur `queueTrigger`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction**| n/a | Dans le fichier *function.json* uniquement. Cette propriété doit être définie sur `in`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. |
|**name** | n/a |Nom de la variable qui contient la charge utile de l’élément de file d’attente dans le code de fonction.  |
|**queueName** | **QueueName**| Nom de la file d’attente à interroger. |
|**connection** | **Connection** |Nom d’un paramètre d’application comportant la chaîne de connexion de stockage à utiliser pour cette liaison. Si le nom du paramètre d’application commence par « AzureWebJobs », vous ne pouvez spécifier que le reste du nom ici. Par exemple, si vous définissez `connection` sur « MyStorage », le runtime Functions recherche un paramètre d’application qui est nommé « MyStorage ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion de stockage par défaut dans le paramètre d’application nommé `AzureWebJobsStorage`.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

Accédez aux données du message en utilisant un paramètre de méthode comme `string paramName`. Vous pouvez lier aux types suivants :

* Objet : le runtime Functions désérialise une charge utile JSON dans une instance d’une classe arbitraire définie dans votre code.
* `string`
* `byte[]`
* [CloudQueueMessage]

Si vous essayez de lier à `CloudQueueMessage` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Accédez aux données du message en utilisant un paramètre de méthode comme `string paramName`. `paramName` est la valeur spécifiée dans la propriété `name` de *function.json*. Vous pouvez lier aux types suivants :

* Objet : le runtime Functions désérialise une charge utile JSON dans une instance d’une classe arbitraire définie dans votre code. 
* `string`
* `byte[]`
* [CloudQueueMessage]

Si vous essayez de lier à `CloudQueueMessage` et obtenez un message d’erreur, vérifiez que vous avez une référence à [la bonne version du SDK Stockage](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

# <a name="java"></a>[Java](#tab/java)

L’annotation [QueueTrigger](/java/api/com.microsoft.azure.functions.annotation.queuetrigger?view=azure-java-stable&preserve-view=true) vous donne accès au message de la file d’attente qui a déclenché la fonction.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La charge utile de l’élément de la file d’attente est disponible via `context.bindings.<NAME>`, où `<NAME>` correspond au nom défini dans *function.json*. Si la charge utile est JSON, la valeur est désérialisée en objet.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Accédez au message de la file d’attente via un paramètre de chaîne correspondant au nom désigné par le paramètre `name` de liaison du fichier *function.json*.

# <a name="python"></a>[Python](#tab/python)

Accédez au message de la file d’attente via le paramètre typé en [QueueMessage](/python/api/azure-functions/azure.functions.queuemessage?view=azure-python&preserve-view=true).

---

## <a name="message-metadata"></a>Métadonnées de message

Le déclencheur de file d’attente fournit plusieurs [propriétés de métadonnées](./functions-bindings-expressions-patterns.md#trigger-metadata). Ces propriétés peuvent être utilisées dans les expressions de liaison dans d’autres liaisons ou en tant que paramètres dans votre code. Les propriétés sont membres de la classe [CloudQueueMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage).

|Propriété|Type|Description|
|--------|----|-----------|
|`QueueTrigger`|`string`|Charge utile de file d’attente (s’il s’agit d’une chaîne valide). Si la charge utile du message de la file d’attente est une chaîne, `QueueTrigger` a la même valeur que la variable nommée par la propriété `name` dans *function.json*.|
|`DequeueCount`|`int`|Nombre de fois que ce message a été enlevé de la file d’attente.|
|`ExpirationTime`|`DateTimeOffset`|Heure à laquelle le message expire.|
|`Id`|`string`|ID de message de la file d’attente.|
|`InsertionTime`|`DateTimeOffset`|Heure à laquelle le message a été ajouté à la file d’attente.|
|`NextVisibleTime`|`DateTimeOffset`|Heure à laquelle le message sera de nouveau visible.|
|`PopReceipt`|`string`|Réception pop du message.|

## <a name="poison-messages"></a>Messages incohérents

En cas d’échec d’une fonction de déclenchement de file d’attente, Azure Functions réessaie la fonction jusqu’à cinq fois (première tentative comprise) pour un message de file d’attente donné. Si les cinq tentatives échouent, le runtime Functions ajoute un message à une file d’attente nommée *&lt;nom_file_d’attente_d’origine>-poison*. Vous pouvez écrire une fonction pour traiter les messages de la file d’attente de messages incohérents en les consignant dans un journal ou en envoyant une notification signalant qu’une attention manuelle est nécessaire.

Pour gérer manuellement les messages incohérents, vérifiez la propriété [dequeueCount](#message-metadata) dans le message de file d’attente.

## <a name="polling-algorithm"></a>Algorithme d’interrogation

Le déclencheur de file d’attente implémente un algorithme d’interruption exponentiel et aléatoire pour réduire l’effet de l’interrogation de file d’attente inactive sur les coûts de transactions de stockage.

L’algorithme utilise la logique suivante :

- Lorsqu’un message est trouvé, le runtime attend deux secondes, puis vérifie s’il existe un autre message.
- Lorsqu’aucun message n’est trouvé, il attend environ quatre secondes avant de réessayer.
- Après plusieurs échecs de tentatives d’obtention d’un message de file d’attente, le temps d’attente continue à augmenter jusqu’à ce qu’il atteigne le délai d’attente maximal par défaut (une minute).
- La durée d’attente maximale est configurable via la propriété `maxPollingInterval` dans le [fichier host.json](functions-host-json-v1.md#queues).

Pour le développement local, l’intervalle d’interrogation maximal par défaut est de deux secondes.

En ce qui concerne la facturation, le temps passé par le runtime à interroger est « gratuit » et n’est pas compté sur votre compte.

## <a name="concurrency"></a>Accès concurrentiel

Lorsque plusieurs messages de file d’attente attendent, le déclencheur de file d’attente récupère un lot de messages et appelle les instances de fonction simultanément pour les traiter. Par défaut, la taille de lot est de 16. Quand le nombre de messages en cours de traitement descend à 8, le runtime obtient un autre lot et commence à traiter ces messages. Par conséquent, le nombre maximal de messages traités simultanément par fonction sur une machine virtuelle est de 24. Cette limite s’applique séparément à chaque fonction déclenchée par une file d’attente sur chaque machine virtuelle. Si votre application de fonction est étendue à plusieurs machines virtuelles, chaque machine virtuelle attend des déclencheurs et essaie d’exécuter les fonctions. Par exemple, si une application de fonction est étendue à 3 machines virtuelles, le nombre maximal d’instances simultanées d’une fonction déclenché par une file d’attente est par défaut de 72.

La taille du lot et le seuil d’obtention d’un nouveau lot sont configurables dans le [fichier host.json](functions-host-json.md#queues). Si vous souhaitez minimiser l’exécution en parallèle des fonctions déclenchées par une file d’attente dans une application de fonction, vous pouvez définir la taille du lot sur 1. Ce paramètre évite les opérations simultanées uniquement pendant l’exécution de votre application de fonction sur une machine virtuelle unique. 

Le déclencheur de la file d’attente empêche automatiquement une fonction de traiter un message de file d’attente plusieurs fois de manière simultanée.

## <a name="hostjson-properties"></a>Propriétés host.json

Le fichier [host.json](functions-host-json.md#queues) contient les paramètres qui contrôlent le comportement du déclencheur de file d’attente. Consultez la section [Paramètres host.json](functions-bindings-storage-queue-output.md#hostjson-settings) pour plus d’informations concernant les paramètres disponibles.

## <a name="next-steps"></a>Étapes suivantes

- [Écrire des messages de stockage de file d’attente (liaison de sortie)](./functions-bindings-storage-queue-output.md)

<!-- LINKS -->

[CloudQueueMessage]: /dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage
