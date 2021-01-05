---
title: Liaisons d’entrée SignalR Service Azure Functions
description: Découvrez comment retourner une URL de point de terminaison SignalR Service et un jeton d’accès dans Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 02/20/2020
ms.author: cshoe
ms.openlocfilehash: 326115a2a6cf29fcf211cdbd918edd0994fe45ed
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88212120"
---
# <a name="signalr-service-input-binding-for-azure-functions"></a>Liaisons d’entrée SignalR Service pour Azure Functions

Avant qu’un client puisse se connecter au service SignalR Azure, il doit récupérer l’URL du point de terminaison du service et un jeton d’accès valide. La liaison d’entrée *SignalRConnectionInfo* génère l’URL du point de terminaison du service SignalR et un jeton valide qui sont utilisés pour se connecter au service. Le jeton étant limité dans le temps et pouvant être utilisé pour authentifier un utilisateur spécifique sur une connexion, vous ne devez pas mettre le jeton en cache ou le partager entre plusieurs clients. Un déclencheur HTTP utilisant cette liaison peut être utilisé par les clients pour récupérer les informations de connexion.

Pour plus d’informations sur la façon dont cette liaison est utilisée pour créer une fonction « negotiate » qui peut être utilisée par un client SignalR, consultez l’[article sur le développement et la configuration d’Azure Functions](../azure-signalr/signalr-concept-serverless-development-config.md) dans la documentation sur les concepts de SignalR Service.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](functions-bindings-signalr-service.md).

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui acquiert des informations de connexion SignalR à l’aide de la liaison d’entrée et les renvoie via HTTP.

```cs
[FunctionName("negotiate")]
public static SignalRConnectionInfo Negotiate(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req,
    [SignalRConnectionInfo(HubName = "chat")]SignalRConnectionInfo connectionInfo)
{
    return connectionInfo;
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison d’entrée d’informations de connexion SignalR dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison pour retourner les informations de connexion.

Voici les données de liaison dans le fichier *function.json* :

Exemple de fichier function.json :

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Voici le code de script C# :

```cs
#r "Microsoft.Azure.WebJobs.Extensions.SignalRService"
using Microsoft.Azure.WebJobs.Extensions.SignalRService;

public static SignalRConnectionInfo Run(HttpRequest req, SignalRConnectionInfo connectionInfo)
{
    return connectionInfo;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison d’entrée d’informations de connexion SignalR dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison pour retourner les informations de connexion.

Voici les données de liaison dans le fichier *function.json* :

Exemple de fichier function.json :

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Voici le code JavaScript :

```javascript
module.exports = async function (context, req, connectionInfo) {
    context.res.body = connectionInfo;
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre une liaison d’entrée d’informations de connexion SignalR dans un fichier *function.json* et une [fonction Python](functions-reference-python.md) qui utilise la liaison pour retourner les informations de connexion.

Voici les données de liaison dans le fichier *function.json* :

Exemple de fichier function.json :

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Voici le code Python :

```python
def main(req: func.HttpRequest, connectionInfoJson: str) -> func.HttpResponse:
    return func.HttpResponse(
        connectionInfoJson,
        status_code=200,
        headers={
            'Content-type': 'application/json'
        }
    )
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant montre une [fonction Java](functions-reference-java.md) qui acquiert des informations de connexion SignalR à l’aide de la liaison d’entrée et les renvoie via HTTP.

```java
@FunctionName("negotiate")
public SignalRConnectionInfo negotiate(
        @HttpTrigger(
            name = "req",
            methods = { HttpMethod.POST },
            authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> req,
        @SignalRConnectionInfoInput(
            name = "connectionInfo",
            hubName = "chat") SignalRConnectionInfo connectionInfo) {
    return connectionInfo;
}
```

---

## <a name="authenticated-tokens"></a>Jetons authentifiés

Si la fonction est déclenchée par un client authentifié, vous pouvez ajouter une revendication d’ID d’utilisateur au jeton généré. Vous pouvez facilement ajouter l’authentification à une application de fonction à l’aide de [Authentification App Service](../app-service/overview-authentication-authorization.md).

L’authentification App Service définit les en-têtes HTTP nommés `x-ms-client-principal-id` et `x-ms-client-principal-name` qui contiennent l’identité principale du client et le nom de l’utilisateur authentifié, respectivement.

# <a name="c"></a>[C#](#tab/csharp)

Vous pouvez définir la propriété `UserId` de la liaison sur la valeur de l’un des en-têtes à l’aide d’une [expression de liaison](./functions-bindings-expressions-patterns.md) : `{headers.x-ms-client-principal-id}` ou `{headers.x-ms-client-principal-name}`.

```cs
[FunctionName("negotiate")]
public static SignalRConnectionInfo Negotiate(
    [HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req, 
    [SignalRConnectionInfo
        (HubName = "chat", UserId = "{headers.x-ms-client-principal-id}")]
        SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier claim set to the authenticated user
    return connectionInfo;
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Vous pouvez définir la propriété `userId` de la liaison sur la valeur de l’un des en-têtes à l’aide d’une [expression de liaison](./functions-bindings-expressions-patterns.md) : `{headers.x-ms-client-principal-id}` ou `{headers.x-ms-client-principal-name}`.

Exemple de fichier function.json :

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Voici le code de script C# :

```cs
#r "Microsoft.Azure.WebJobs.Extensions.SignalRService"
using Microsoft.Azure.WebJobs.Extensions.SignalRService;

public static SignalRConnectionInfo Run(HttpRequest req, SignalRConnectionInfo connectionInfo)
{
    // connectionInfo contains an access key token with a name identifier
    // claim set to the authenticated user
    return connectionInfo;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Vous pouvez définir la propriété `userId` de la liaison sur la valeur de l’un des en-têtes à l’aide d’une [expression de liaison](./functions-bindings-expressions-patterns.md) : `{headers.x-ms-client-principal-id}` ou `{headers.x-ms-client-principal-name}`.

Exemple de fichier function.json :

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Voici le code JavaScript :

```javascript
module.exports = async function (context, req, connectionInfo) {
    // connectionInfo contains an access key token with a name identifier
    // claim set to the authenticated user
    context.res.body = connectionInfo;
};
```

# <a name="python"></a>[Python](#tab/python)

Vous pouvez définir la propriété `userId` de la liaison sur la valeur de l’un des en-têtes à l’aide d’une [expression de liaison](./functions-bindings-expressions-patterns.md) : `{headers.x-ms-client-principal-id}` ou `{headers.x-ms-client-principal-name}`.

Exemple de fichier function.json :

```json
{
    "type": "signalRConnectionInfo",
    "name": "connectionInfo",
    "hubName": "chat",
    "userId": "{headers.x-ms-client-principal-id}",
    "connectionStringSetting": "<name of setting containing SignalR Service connection string>",
    "direction": "in"
}
```

Voici le code Python :

```python
def main(req: func.HttpRequest, connectionInfoJson: str) -> func.HttpResponse:
    # connectionInfo contains an access key token with a name identifier
    # claim set to the authenticated user
    return func.HttpResponse(
        connectionInfoJson,
        status_code=200,
        headers={
            'Content-type': 'application/json'
        }
    )
```

# <a name="java"></a>[Java](#tab/java)

Vous pouvez définir la propriété `userId` de la liaison sur la valeur de l’un des en-têtes à l’aide d’une [expression de liaison](./functions-bindings-expressions-patterns.md) : `{headers.x-ms-client-principal-id}` ou `{headers.x-ms-client-principal-name}`.

```java
@FunctionName("negotiate")
public SignalRConnectionInfo negotiate(
        @HttpTrigger(
            name = "req",
            methods = { HttpMethod.POST },
            authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> req,
        @SignalRConnectionInfoInput(
            name = "connectionInfo",
            hubName = "chat",
            userId = "{headers.x-ms-client-principal-id}") SignalRConnectionInfo connectionInfo) {
    return connectionInfo;
}
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Envoyer des messages SignalR Service (Liaison de sortie)](./functions-bindings-signalr-service-output.md) 
