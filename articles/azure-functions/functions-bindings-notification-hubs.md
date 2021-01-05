---
title: Liaisons Notification Hubs pour Azure Functions
description: Découvrez comment utiliser les liaisons Azure Notification Hubs dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 11/21/2017
ms.author: cshoe
ms.openlocfilehash: c4198a1b73f76d61e39324befc85b55bd260e363
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88212227"
---
# <a name="notification-hubs-output-binding-for-azure-functions"></a>Liaison de sortie Notification Hubs pour Azure Functions

Cet article explique comment envoyer des notifications Push à l’aide de liaisons [Azure Notification Hubs](../notification-hubs/notification-hubs-push-notification-overview.md) dans Azure Functions. Azure Functions prend en charge les liaisons de sortie pour Notification Hubs.

Azure Notification Hubs doit être configuré pour l’infrastructure Platform Notification System (PNS) que vous souhaitez utiliser. Pour savoir comment obtenir des notifications Push dans votre application cliente provenant de Notification Hubs, consultez [Prise en main de Notification Hubs](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md) et sélectionnez votre plateforme de client cible dans la liste déroulante en haut de la page.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!IMPORTANT]
> Google a décidé de [déprécier Google Cloud Messaging (GCM) en faveur de Firebase Cloud Messaging (FCM)](https://developers.google.com/cloud-messaging/faq). Cette liaison de sortie ne prend pas en charge FCM. Pour envoyer des notifications à l’aide de FCM, utilisez l’[API Firebase](https://firebase.google.com/docs/cloud-messaging/server#choosing-a-server-option) directement dans votre fonction ou utilisez des [notifications de modèle](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md).

## <a name="packages---functions-1x"></a>Packages - Functions 1.x

Les liaisons Notification Hubs sont fournies dans le package NuGet [Microsoft.Azure.WebJobs.Extensions.NotificationHubs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.NotificationHubs), version 1.x. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/tree/v2.x/src/WebJobs.Extensions.NotificationHubs).

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x-and-higher"></a>Packages – Functions 2.x et versions ultérieures

Cette liaison n’est pas disponible dans Functions 2.x et ultérieur.

## <a name="example---template"></a>Exemple - modèle

Les notifications que vous envoyez peuvent être des notifications natives ou des [notifications de modèle](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md). Les notifications natives ciblent une plateforme cliente spécifique telle que configurée dans la propriété `platform` de la liaison de sortie. Un modèle de notification peut servir à plusieurs plateformes cibles.   

Consultez l’exemple propre à un langage particulier :

* [Script C# - paramètre de sortie](#c-script-template-example---out-parameter)
* [Script C# - asynchrone](#c-script-template-example---asynchronous)
* [Script C# - JSON](#c-script-template-example---json)
* [Script C# - types de bibliothèque](#c-script-template-example---library-types)
* [F#](#f-template-example)
* [JavaScript](#javascript-template-example)

### <a name="c-script-template-example---out-parameter"></a>Exemple de modèle de script C# - paramètre de sortie

Cet exemple envoie une notification pour une [inscription de modèle](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) contenant un emplacement réservé `message` dans le modèle.

```cs
using System;
using System.Threading.Tasks;
using System.Collections.Generic;

public static void Run(string myQueueItem,  out IDictionary<string, string> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    notification = GetTemplateProperties(myQueueItem);
}

private static IDictionary<string, string> GetTemplateProperties(string message)
{
    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
    templateProperties["message"] = message;
    return templateProperties;
}
```

### <a name="c-script-template-example---asynchronous"></a>Exemple de modèle de script C# - asynchrone

Si vous utilisez du code asynchrone, les paramètres de sortie ne sont pas autorisés. Dans ce cas, utilisez `IAsyncCollector` pour renvoyer votre notification modèle. Le code suivant est un exemple de code asynchrone ci-dessus. 

```cs
using System;
using System.Threading.Tasks;
using System.Collections.Generic;

public static async Task Run(string myQueueItem, IAsyncCollector<IDictionary<string,string>> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    log.Info($"Sending Template Notification to Notification Hub");
    await notification.AddAsync(GetTemplateProperties(myQueueItem));    
}

private static IDictionary<string, string> GetTemplateProperties(string message)
{
    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
    templateProperties["user"] = "A new user wants to be added : " + message;
    return templateProperties;
}
```

### <a name="c-script-template-example---json"></a>Exemple de modèle de script C# - JSON

Cet exemple envoie une notification pour une [inscription de modèle](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) contenant un emplacement réservé `message` dans le modèle à l’aide d’une chaîne JSON valide.

```cs
using System;

public static void Run(string myQueueItem,  out string notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
    notification = "{\"message\":\"Hello from C#. Processed a queue item!\"}";
}
```

### <a name="c-script-template-example---library-types"></a>Exemple de modèle de script C# - types de bibliothèque

Cet exemple indique comment utiliser le type défini dans la [bibliothèque Microsoft Azure Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/). 

```cs
#r "Microsoft.Azure.NotificationHubs"

using System;
using System.Threading.Tasks;
using Microsoft.Azure.NotificationHubs;

public static void Run(string myQueueItem,  out Notification notification, TraceWriter log)
{
   log.Info($"C# Queue trigger function processed: {myQueueItem}");
   notification = GetTemplateNotification(myQueueItem);
}

private static TemplateNotification GetTemplateNotification(string message)
{
    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
    templateProperties["message"] = message;
    return new TemplateNotification(templateProperties);
}
```

### <a name="f-template-example"></a>Exemple de modèle F#

Cet exemple envoie une notification pour une [inscription de modèles](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) contenant `location` et `message`.

```fsharp
let Run(myTimer: TimerInfo, notification: byref<IDictionary<string, string>>) =
    notification = dict [("location", "Redmond"); ("message", "Hello from F#!")]
```

### <a name="javascript-template-example"></a>Exemple de modèle JavaScript

Cet exemple envoie une notification pour une [inscription de modèles](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) contenant `location` et `message`.

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if (myTimer.IsPastDue)
    {
        context.log('Node.js is running late!');
    }
    context.log('Node.js timer trigger function ran!', timeStamp);  
    context.bindings.notification = {
        location: "Redmond",
        message: "Hello from Node!"
    };
    context.done();
};
```

## <a name="example---apns-native"></a>Exemple - APNS natif

Cet exemple de script C# montre comment envoyer une notification APNS native. 

```cs
#r "Microsoft.Azure.NotificationHubs"
#r "Newtonsoft.Json"

using System;
using Microsoft.Azure.NotificationHubs;
using Newtonsoft.Json;

public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a new user to be processed in the form of a JSON string with 
    // a "name" value.
    //
    // The JSON format for a native APNS notification is ...
    // { "aps": { "alert": "notification message" }}  

    log.LogInformation($"Sending APNS notification of a new user");    
    dynamic user = JsonConvert.DeserializeObject(myQueueItem);    
    string apnsNotificationPayload = "{\"aps\": {\"alert\": \"A new user wants to be added (" + 
                                        user.name + ")\" }}";
    log.LogInformation($"{apnsNotificationPayload}");
    await notification.AddAsync(new AppleNotification(apnsNotificationPayload));        
}
```

## <a name="example---wns-native"></a>Exemple - WNS natif

Cet exemple de script C# indique comment utiliser les types définis dans la [bibliothèque Microsoft Azure Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) pour envoyer une notification toast WNS native. 

```cs
#r "Microsoft.Azure.NotificationHubs"
#r "Newtonsoft.Json"

using System;
using Microsoft.Azure.NotificationHubs;
using Newtonsoft.Json;

public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a new user to be processed in the form of a JSON string with 
    // a "name" value.
    //
    // The XML format for a native WNS toast notification is ...
    // <?xml version="1.0" encoding="utf-8"?>
    // <toast>
    //      <visual>
    //     <binding template="ToastText01">
    //       <text id="1">notification message</text>
    //     </binding>
    //   </visual>
    // </toast>

    log.Info($"Sending WNS toast notification of a new user");    
    dynamic user = JsonConvert.DeserializeObject(myQueueItem);    
    string wnsNotificationPayload = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                                    "<toast><visual><binding template=\"ToastText01\">" +
                                        "<text id=\"1\">" + 
                                            "A new user wants to be added (" + user.name + ")" + 
                                        "</text>" +
                                    "</binding></visual></toast>";

    log.Info($"{wnsNotificationPayload}");
    await notification.AddAsync(new WindowsNotification(wnsNotificationPayload));        
}
```

## <a name="attributes"></a>Attributs

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [NotificationHub](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.NotificationHubs/NotificationHubAttribute.cs).

Les paramètres de constructeur de l’attribut et les propriétés sont décrites dans la section [Configuration](#configuration).

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `NotificationHub` :

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** |n/a| Cette propriété doit être définie sur `notificationHub`. |
|**direction** |n/a| Cette propriété doit être définie sur `out`. | 
|**name** |n/a| Nom de variable utilisé dans le code de fonction pour le message du hub de notification. |
|**tagExpression** |**tagExpression** | Expressions de balise vous permettant de demander que les notifications soient remises à un ensemble d’appareils qui se sont inscrits pour la réception de notifications correspondant à l’expression de balise.  Pour plus d’informations, voir [Routage et expressions de balise](../notification-hubs/notification-hubs-tags-segment-push-message.md). |
|**hubName** | **HubName** | Nom de la ressource de hub de notification dans le portail Azure. |
|**connection** | **ConnectionStringSetting** | Nom d’un paramètre d’application contenant une chaîne de connexion Notification Hubs.  Vous devez définir la chaîne de connexion sur la valeur *DefaultFullSharedAccessSignature* de votre Hub de notification. Consultez [Configuration de la chaîne de connexion](#connection-string-setup) plus loin dans cet article.|
|**platform** | **Plateforme** | La propriété de la plateforme indique la plateforme cliente ciblée par votre notification. Par défaut, si la propriété de la plateforme est omise dans la liaison de sortie, les notifications de modèle peuvent être utilisées pour cibler n’importe quelle plateforme configurée sur Azure Notification Hub. Pour en savoir plus sur l’utilisation de modèles en général pour envoyer entre des notifications entre plusieurs plateformes avec un Azure Notification Hub, consultez la rubrique [Modèles](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md). Une fois définie, **platform** doit avoir l’une des valeurs suivantes : <ul><li><code>apns</code>&mdash;Apple Push Notification Service (APNS). Pour en savoir plus sur la configuration du hub de notification pour Apple Push Notification Service et la réception de notifications dans une application cliente, consultez [Envoi de notifications Push vers iOS avec Azure Notification Hubs](../notification-hubs/xamarin-notification-hubs-ios-push-notification-apns-get-started.md).</li><li><code>adm</code>&mdash;[Amazon Device Messaging (ADM)](https://developer.amazon.com/device-messaging). Pour en savoir plus sur la configuration du hub de notification pour ADM et la réception de notifications dans une application Kindle, consultez [Prise en main de Notification Hubs pour les applications Kindle](../notification-hubs/notification-hubs-android-push-notification-google-fcm-get-started.md).</li><li><code>wns</code>&mdash;[Services de notification Push Windows (WNS)](/windows/uwp/design/shell/tiles-and-notifications/windows-push-notification-services--wns--overview) ciblant les plateformes Windows. Windows Phone 8.1 et les versions ultérieures sont est également prises en charge par WNS. Pour plus d’informations, consultez [Prise en main de Notification Hubs pour les applications de plateforme Windows universelle](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).</li><li><code>mpns</code>&mdash;[Services de notifications Push Microsoft (MPNS)](/previous-versions/windows/apps/ff402558(v=vs.105)). Cette plateforme prend en charge Windows Phone 8 et les plateformes antérieures de Windows Phone. Pour plus d’informations, consultez [Envoi de notifications Push avec Azure Notification Hubs sur Windows Phone](../notification-hubs/notification-hubs-windows-mobile-push-notifications-mpns.md).</li></ul> |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

### <a name="functionjson-file-example"></a>Exemple de fichier function.json

Voici un exemple de liaison Notification Hubs dans un fichier *function.json*.

```json
{
  "bindings": [
    {
      "type": "notificationHub",
      "direction": "out",
      "name": "notification",
      "tagExpression": "",
      "hubName": "my-notification-hub",
      "connection": "MyHubConnectionString",
      "platform": "apns"
    }
  ],
  "disabled": false
}
```

### <a name="connection-string-setup"></a>Configuration de la chaîne de connexion

Pour utiliser une liaison de sortie de Hub de notification, vous devez configurer la chaîne de connexion pour le hub. Vous pouvez sélectionner un hub de notification existant ou en créer un nouveau sous l’onglet *Intégrer* du portail Azure. Vous pouvez également configurer la chaîne de connexion manuellement. 

Pour configurer la chaîne de connexion sur un hub de notification existant :

1. Accédez à votre hub de notification dans le [portail Azure](https://portal.azure.com), choisissez **Stratégies d’accès**, puis sélectionnez le bouton Copier en regard de la stratégie **DefaultFullSharedAccessSignature**. Vous copiez ainsi la chaîne de connexion de la stratégie *DefaultFullSharedAccessSignature* dans votre hub de notification. Cette chaîne de connexion permet à votre fonction d’envoyer des messages de notification au hub.
    ![Copier la chaîne de connexion du hub de notification](./media/functions-bindings-notification-hubs/get-notification-hub-connection.png)
1. Accédez à votre application de fonction dans le portail Azure, choisissez **Paramètres de l’application**, ajoutez une clé telle que **MyHubConnectionString**, collez la stratégie *DefaultFullSharedAccessSignature* copiée dans votre hub de notification en tant que valeur, puis cliquez sur **Enregistrer**.

Le nom de ce paramètre d’application passe dans le paramètre de connexion de liaison de sortie dans le fichier *function.json* ou l’attribut .NET. Consultez la [section Configuration](#configuration) plus haut dans cet article.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="exceptions-and-return-codes"></a>Exceptions et codes de retour

| Liaison | Informations de référence |
|---|---|
| Hub de notification | [Guide des opérations](/rest/api/notificationhubs/) |

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les déclencheurs et les liaisons Azure Functions](functions-triggers-bindings.md)
