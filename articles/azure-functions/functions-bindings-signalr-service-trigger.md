---
title: Liaison de déclencheur SignalR Service pour Azure Functions
description: Apprenez à envoyer des messages SignalR Service à partir d’Azure Functions.
author: chenyl
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 05/11/2020
ms.author: chenyl
ms.openlocfilehash: e2651afbcdc3bae71bb531aa0e821f83264c295d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88212594"
---
# <a name="signalr-service-trigger-binding-for-azure-functions"></a>Liaison de déclencheur SignalR Service pour Azure Functions

Utilisez la liaison de déclencheur *SignalR* pour répondre aux messages envoyés par Azure SignalR Service. Quand la fonction est déclenchée, les messages passés à la fonction sont analysés en tant qu’objets JSON.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-signalr-service.md).

## <a name="example"></a>Exemple

L’exemple suivant montre une fonction qui reçoit un message par la liaison de déclencheur et journalise le message.

# <a name="c"></a>[C#](#tab/csharp)

La liaison de déclencheur SignalR Service pour C# a deux modèles de programmation : un modèle basé sur la classe et un modèle traditionnel. Le modèle basé sur la classe peut fournir une expérience de programmation cohérente côté serveur SignalR. Le modèle traditionnel offre une plus grande souplesse et présente des similitudes avec d’autres liaisons de fonction.

### <a name="with-class-based-model"></a>Avec le modèle basé sur la classe

Pour plus d’informations, consultez [Modèle basé sur la classe](../azure-signalr/signalr-concept-serverless-development-config.md#class-based-model).

```cs
public class SignalRTestHub : ServerlessHub
{
    [FunctionName("SignalRTest")]
    public async Task SendMessage([SignalRTrigger]InvocationContext invocationContext, string message, ILogger logger)
    {
        logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
    }
}
```

### <a name="with-traditional-model"></a>Avec le modèle traditionnel

Le modèle traditionnel obéit à la convention d’Azure Function développée par C#. Si vous n’êtes pas familiarisé avec ce langage, vous pouvez vous reporter à ces [documents](./functions-dotnet-class-library.md).

```cs
[FunctionName("SignalRTest")]
public static async Task Run([SignalRTrigger("SignalRTest", "messages", "SendMessage", parameterNames: new string[] {"message"})]InvocationContext invocationContext, string message, ILogger logger)
{
    logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
}
```

#### <a name="use-attribute-signalrparameter-to-simplify-parameternames"></a>Utiliser l’attribut `[SignalRParameter]` pour simplifier `ParameterNames`

À la place de `ParameterNames`, qui est un peu lourd à manier, utilisez `SignalRParameter` qui permet d’obtenir le même résultat.

```cs
[FunctionName("SignalRTest")]
public static async Task Run([SignalRTrigger("SignalRTest", "messages", "SendMessage")]InvocationContext invocationContext, [SignalRParameter]string message, ILogger logger)
{
    logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Voici les données de liaison dans le fichier *function.json* :

Exemple de fichier function.json :

```json
{
    "type": "signalRTrigger",
    "name": "invocation",
    "hubName": "SignalRTest",
    "category": "messages",
    "event": "SendMessage",
    "parameterNames": [
        "message"
    ],
    "direction": "in"
}
```

Voici le code de script C# :

```cs
#r "Microsoft.Azure.WebJobs.Extensions.SignalRService"
using System;
using Microsoft.Azure.WebJobs.Extensions.SignalRService;
using Microsoft.Extensions.Logging;

public static void Run(InvocationContext invocation, string message, ILogger logger)
{
    logger.LogInformation($"Receive {message} from {invocationContext.ConnectionId}.");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Voici les données de liaison dans le fichier *function.json* :

Exemple de fichier function.json :

```json
{
    "type": "signalRTrigger",
    "name": "invocation",
    "hubName": "SignalRTest",
    "category": "messages",
    "event": "SendMessage",
    "parameterNames": [
        "message"
    ],
    "direction": "in"
}
```

Voici le code JavaScript :

```javascript
module.exports = function (context, invocation) {
    context.log(`Receive ${context.bindingData.message} from ${invocation.ConnectionId}.`)
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

Voici les données de liaison dans le fichier *function.json* :

Exemple de fichier function.json :

```json
{
    "type": "signalRTrigger",
    "name": "invocation",
    "hubName": "SignalRTest",
    "category": "messages",
    "event": "SendMessage",
    "parameterNames": [
        "message"
    ],
    "direction": "in"
}
```

Voici le code Python :

```python
import logging
import json
import azure.functions as func

def main(invocation) -> None:
    invocation_json = json.loads(invocation)
    logging.info("Receive {0} from {1}".format(invocation_json['Arguments'][0], invocation_json['ConnectionId']))
```

---

## <a name="configuration"></a>Configuration

### <a name="signalrtrigger"></a>SignalRTrigger

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `SignalRTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type**| n/a | Cette propriété doit être définie sur `SignalRTrigger`.|
|**direction**| n/a | Cette propriété doit être définie sur `in`.|
|**name**| n/a | Nom de variable utilisé dans le code de fonction pour le contexte d’appel du déclencheur. |
|**hubName**|**HubName**| Cette valeur doit être définie sur le nom du hub SignalR pour que la fonction soit déclenchée.|
|**category**|**Catégorie**| Cette valeur doit être définie sur la catégorie des messages pour que la fonction soit déclenchée. La catégorie peut avoir l’une des valeurs suivantes : <ul><li>**connections** : pour les événements *connected* et *disconnected*</li><li>**messages** : pour tous les autres événements, hormis ceux de la catégorie *connections*</li></ul> |
|**event**|**Event**| Cette valeur doit être définie sur l’événement des messages pour que la fonction soit déclenchée. Pour la catégorie *messages*, l’événement est *target* dans les [messages d’appel](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocation-message-encoding) que les clients envoient. Pour la catégorie *connections*, seuls les événements *connected* et *disconnected* sont utilisés. |
|**parameterNames**|**ParameterNames**| (Facultatif) Liste des noms liés aux paramètres. |
|**connectionStringSetting**|**ConnectionStringSetting**| Nom du paramètre d’application contenant la chaîne de connexion du service SignalR (« AzureSignalRConnectionString » par défaut) |

## <a name="payload"></a>Payload

Le type d’entrée du déclencheur est déclaré comme étant `InvocationContext` ou un type personnalisé. Si vous choisissez `InvocationContext`, vous obtenez un accès complet au contenu de la requête. Pour un type personnalisé, le runtime tente d’analyser le corps de la requête JSON pour définir les propriétés de l’objet.

### <a name="invocationcontext"></a>InvocationContext

InvocationContext comporte tout le contenu du message envoyé par SignalR Service.

|Propriété dans InvocationContext | Description|
|------------------------------|------------|
|Arguments| Disponible pour la catégorie *messages*. Contient des *arguments* dans le [message d’appel](https://github.com/dotnet/aspnetcore/blob/master/src/SignalR/docs/specs/HubProtocol.md#invocation-message-encoding)|
|Error| Disponible pour l’événement *disconnected*. Elle peut être vide si la connexion s’est fermée sans erreur ou si elle contient les messages d’erreur.|
|Hub| Nom du hub auquel le message appartient.|
|Category| Catégorie du message.|
|Événement| Événement du message.|
|ConnectionId| ID de connexion du client qui envoie le message.|
|UserId| Identité de l’utilisateur du client qui envoie le message.|
|headers| en-têtes de la requête.|
|Requête| Requête de la demande lorsque les clients se connectent au service.|
|Revendications| Revendications du client.|

## <a name="using-parameternames"></a>Utilisation de `ParameterNames`

La propriété `ParameterNames` dans `SignalRTrigger` vous permet de lier les arguments des messages d’appel aux paramètres des fonctions. C’est un moyen plus pratique d’accéder aux arguments de `InvocationContext`.

Imaginons que vous ayez un client JavaScript SignalR qui tente d’appeler la méthode `broadcast` dans Azure Function avec deux arguments.

```javascript
await connection.invoke("broadcast", message1, message2);
```

Vous pouvez accéder à ces deux arguments à partir du paramètre et leur affecter un type de paramètre en utilisant `ParameterNames`.

### <a name="remarks"></a>Notes

L’ordre de la liaison de paramètre est important. Si vous utilisez `ParameterNames`, l’ordre dans `ParameterNames` correspond à l’ordre des arguments que vous appelez dans le client. Si vous utilisez l’attribut `[SignalRParameter]` dans C#, l’ordre des arguments dans les méthodes Azure Function correspond à l’ordre des arguments dans les clients.

`ParameterNames` et l’attribut `[SignalRParameter]` **ne peuvent pas** être utilisés simultanément ; sinon, une exception sera générée.

## <a name="send-messages-to-signalr-service-trigger-binding"></a>Envoyer des messages à la liaison de déclencheur SignalR Service

Azure Function génère une URL pour la liaison de déclencheur SignalR Service. Le format de l’URL est le suivant :

```http
https://<APP_NAME>.azurewebsites.net/runtime/webhooks/signalr?code=<API_KEY>
```

La clé API (`API_KEY`) est générée par Azure Function. Vous pouvez obtenir la valeur `API_KEY` dans le portail Azure quand vous utilisez la liaison de déclencheur SignalR Service.
:::image type="content" source="media/functions-bindings-signalr-service/signalr-keys.png" alt-text="Clé API":::

Vous devez définir cette URL dans `UrlTemplate` dans les paramètres en amont de SignalR Service.

## <a name="next-steps"></a>Étapes suivantes

* [Développement et configuration Azure Functions avec Azure SignalR Service](../azure-signalr/signalr-concept-serverless-development-config.md)
* [Exemple de liaison de déclencheur SignalR Service](https://github.com/Azure/azure-functions-signalrservice-extension/tree/dev/samples/bidirectional-chat)
