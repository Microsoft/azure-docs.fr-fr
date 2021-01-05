---
title: Liaisons Mobile Apps pour Azure Functions
description: Découvrez comment utiliser des liaisons Azure Mobile Apps dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 11/21/2017
ms.author: cshoe
ms.openlocfilehash: 5ea58cc3d9f3615a74249b36f3f9ffb79caddda1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88212244"
---
# <a name="mobile-apps-bindings-for-azure-functions"></a>Liaisons Mobile Apps pour Azure Functions 

> [!NOTE]
> Les liaisons Azure Mobile Apps sont disponibles uniquement pour Azure Functions 1.x. Elles ne sont pas prises en charge dans Azure Functions 2.x et ultérieur.

Cet article explique comment utiliser des liaisons [Azure Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop) dans Azure Functions. Azure Functions prend en charge des liaisons d’entrée et de sortie pour Mobile Apps.

Les liaisons Mobile Apps vous permettent de lire et mettre à jour des tables de données dans des applications mobiles.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>Packages - Functions 1.x

Les liaisons Mobile Apps sont fournies dans le package NuGet [Microsoft.Azure.WebJobs.Extensions.MobileApps](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.MobileApps), version 1.x. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.MobileApps/).

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="input"></a>Entrée

La liaison d’entrée Mobile Apps charge un enregistrement à partir d’un point de terminaison de table mobile et le transmet à votre fonction. Dans des fonctions C# et F#, toutes les modifications apportées à l’enregistrement sont automatiquement renvoyées à la table une fois que la fonction s’est correctement terminée.

## <a name="input---example"></a>Entrée - exemple

Consultez l’exemple propre à un langage particulier :

* [Script C# (.csx)](#input---c-script-example)
* JavaScript

### <a name="input---c-script-example"></a>Entrée - exemple de script C#

L’exemple suivant montre une liaison d’entrée Mobile Apps dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction est déclenchée par un message de file d’attente qui a un identificateur d’enregistrement. La fonction lit l’enregistrement spécifié et modifie sa propriété `Text`.

Voici les données de liaison dans le fichier *function.json* :

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id": "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
La section [configuration](#input---configuration) décrit ces propriétés.

Voici le code Script C# :

```cs
#r "Newtonsoft.Json"    
using Newtonsoft.Json.Linq;

public static void Run(string myQueueItem, JObject record)
{
    if (record != null)
    {
        record["Text"] = "This has changed.";
    }    
}
```

### <a name="input---javascript"></a>Entrée - JavaScript

L’exemple suivant montre une liaison d’entrée Mobile Apps dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction est déclenchée par un message de file d’attente qui a un identificateur d’enregistrement. La fonction lit l’enregistrement spécifié et modifie sa propriété `Text`.

Voici les données de liaison dans le fichier *function.json* :

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "id": "{queueTrigger}",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "in"
    }
]
}
```
La section [configuration](#input---configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function (context, myQueueItem) {    
    context.log(context.bindings.record);
    context.done();
};
```

## <a name="input---attributes"></a>Entrée - attributs

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs).

Pour plus d’informations sur les propriétés d’attribut que vous pouvez configurer, consultez la [section de configuration suivante](#input---configuration).

## <a name="input---configuration"></a>Entrée - configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `MobileTable`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
| **type**| n/a | Doit être défini sur « mobileTable »|
| **direction**| n/a |Doit être défini sur « in »|
| **name**| n/a | Nom du paramètre d’entrée dans la signature de la fonction.|
|**tableName** |**TableName**|Nom de la table de données de l’application mobile|
| **id**| **Id** | Identificateur de l’enregistrement à récupérer. Peut être statique ou basé sur le déclencheur qui appelle la fonction. Par exemple, si vous utilisez un déclencheur de file d’attente pour votre fonction, `"id": "{queueTrigger}"` utilise la valeur de chaîne du message de file d’attente en tant qu’ID de l’enregistrement à récupérer.|
|**connection**|**Connection**|Nom d’un paramètre d’application qui a l’URL de l’application mobile. La fonction utilise cette URL pour construire les opérations REST requises par rapport à votre application mobile. Créez un paramètre d’application dans votre application de fonction, contenant l’URL de votre application mobile, puis spécifiez le nom du paramètre d’application dans la propriété `connection` de votre liaison d’entrée. L’URL est de type `http://<appname>.azurewebsites.net`.
|**apiKey**|**ApiKey**|Nom d’un paramètre d’application qui a la clé API de votre application mobile. Indiquez la clé API si vous [implémentez une clé API dans votre application mobile Node.js](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key) ou [implémentez une clé API dans votre application mobile .NET](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key). Pour indiquer la clé, créez un paramètre d’application dans votre application de fonction contenant la clé API, puis ajoutez la propriété `apiKey` dans votre liaison d’entrée avec le nom du paramètre d’application. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Ne partagez pas la clé API avec les clients de votre application mobile. Elle doit uniquement être distribuée de façon sécurisée aux clients côté service, comme Azure Functions. Azure Functions stocke vos informations de connexion et les clés API en tant que paramètres d’application, de sorte qu’elles ne soient pas vérifiées dans votre référentiel de contrôle de code source. Ceci protège vos informations sensibles.

## <a name="input---usage"></a>Entrée - utilisation

Dans les fonctions C#, quand l’enregistrement portant l’ID spécifié est trouvé, il est passé au paramètre [JObject](https://www.newtonsoft.com/json/help/html/t_newtonsoft_json_linq_jobject.htm) nommé. Si l’enregistrement est introuvable, la valeur du paramètre est `null`. 

Dans les fonctions JavaScript, l’enregistrement est passé à l’objet `context.bindings.<name>`. Si l’enregistrement est introuvable, la valeur du paramètre est `null`. 

Dans les fonctions C# et F#, toutes les modifications apportées à l’enregistrement d’entrée (paramètre d’entrée) sont automatiquement renvoyées à la table une fois que la fonction s’est correctement terminée. Vous ne pouvez pas modifier un enregistrement dans les fonctions JavaScript.

## <a name="output"></a>Output

Utilisez la liaison de sortie Mobile Apps pour écrire un nouvel enregistrement dans une table Mobile Apps.  

## <a name="output---example"></a>Sortie - exemple

Consultez l’exemple propre à un langage particulier :

* [C#](#output---c-example)
* [Script C# (.csx)](#output---c-script-example)
* [JavaScript](#output---javascript-example)

### <a name="output---c-example"></a>Sortie - exemple C#

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui est déclenchée par un message de file d’attente et crée un enregistrement dans une table d’application mobile.

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    return new { Text = $"I'm running in a C# function! {myQueueItem}" };
}
```

### <a name="output---c-script-example"></a>Sortie - exemple Script C#

L’exemple suivant montre une liaison de sortie Mobile Apps dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction est déclenchée par un message de file d’attente et crée un enregistrement avec une valeur codée en dur pour la propriété `Text`.

Voici les données de liaison dans le fichier *function.json* :

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "out"
    }
]
}
```

La section [configuration](#output---configuration) décrit ces propriétés.

Voici le code Script C# :

```cs
public static void Run(string myQueueItem, out object record)
{
    record = new {
        Text = $"I'm running in a C# function! {myQueueItem}"
    };
}
```

### <a name="output---javascript-example"></a>Sortie - exemple JavaScript

L’exemple suivant montre une liaison de sortie Mobile Apps dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction est déclenchée par un message de file d’attente et crée un enregistrement avec une valeur codée en dur pour la propriété `Text`.

Voici les données de liaison dans le fichier *function.json* :

```json
{
"bindings": [
    {
        "name": "myQueueItem",
        "queueName": "myqueue-items",
        "connection": "",
        "type": "queueTrigger",
        "direction": "in"
    },
    {
        "name": "record",
        "type": "mobileTable",
        "tableName": "MyTable",
        "connection": "My_MobileApp_Url",
        "apiKey": "My_MobileApp_Key",
        "direction": "out"
    }
],
"disabled": false
}
```

La section [configuration](#output---configuration) décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function (context, myQueueItem) {

    context.bindings.record = {
        text : "I'm running in a Node function! Data: '" + myQueueItem + "'"
    }   

    context.done();
};
```

## <a name="output---attributes"></a>Sortie - attributs

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [MobileTable](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.MobileApps/MobileTableAttribute.cs).

Pour plus d’informations sur les propriétés d’attribut que vous pouvez configurer, consultez [Sortie - configuration](#output---configuration). Voici un exemple d’attribut `MobileTable` dans une signature de méthode :

```csharp
[FunctionName("MobileAppsOutput")]        
[return: MobileTable(ApiKeySetting = "MyMobileAppKey", TableName = "MyTable", MobileAppUriSetting = "MyMobileAppUri")]
public static object Run(
    [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] string myQueueItem,
    TraceWriter log)
{
    ...
}
```

Pour obtenir un exemple complet, consultez [Sortie - exemple C#](#output---c-example).

## <a name="output---configuration"></a>Sortie - configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `MobileTable`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
| **type**| n/a | Doit être défini sur « mobileTable »|
| **direction**| n/a |Doit être défini sur « out »|
| **name**| n/a | Nom du paramètre de sortie dans la signature de la fonction.|
|**tableName** |**TableName**|Nom de la table de données de l’application mobile|
|**connection**|**MobileAppUriSetting**|Nom d’un paramètre d’application qui a l’URL de l’application mobile. La fonction utilise cette URL pour construire les opérations REST requises par rapport à votre application mobile. Créez un paramètre d’application dans votre application de fonction, contenant l’URL de votre application mobile, puis spécifiez le nom du paramètre d’application dans la propriété `connection` de votre liaison d’entrée. L’URL est de type `http://<appname>.azurewebsites.net`.
|**apiKey**|**ApiKeySetting**|Nom d’un paramètre d’application qui a la clé API de votre application mobile. Indiquez la clé API si vous [implémentez une clé API dans le backend de votre application mobile Node.js](https://github.com/Azure/azure-mobile-apps-node/tree/master/samples/api-key) ou si vous [implémentez une clé API dans le backend de votre application mobile .NET](https://github.com/Azure/azure-mobile-apps-net-server/wiki/Implementing-Application-Key). Pour indiquer la clé, créez un paramètre d’application dans votre application de fonction contenant la clé API, puis ajoutez la propriété `apiKey` dans votre liaison d’entrée avec le nom du paramètre d’application. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Ne partagez pas la clé API avec les clients de votre application mobile. Elle doit uniquement être distribuée de façon sécurisée aux clients côté service, comme Azure Functions. Azure Functions stocke vos informations de connexion et les clés API en tant que paramètres d’application, de sorte qu’elles ne soient pas vérifiées dans votre référentiel de contrôle de code source. Ceci protège vos informations sensibles.

## <a name="output---usage"></a>Sortie - utilisation

Dans les fonctions de script C#, utilisez un paramètre de sortie nommé de type `out object` pour accéder à l’enregistrement de sortie. Dans les bibliothèques de classes C#, l’attribut `MobileTable` est utilisable avec les types suivants :

* `ICollector<T>` ou `IAsyncCollector<T>`, où `T` est `JObject` ou tout type avec une propriété `public string Id`.
* `out JObject`
* `out T` ou `out T[]`, où `T` est tout type avec une propriété `public string Id`.

Dans les fonctions Node.js, utilisez `context.bindings.<name>` pour accéder à l’enregistrement de sortie.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les déclencheurs et les liaisons Azure Functions](functions-triggers-bindings.md)
