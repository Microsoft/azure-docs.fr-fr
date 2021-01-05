---
title: Développer et configurer une application Azure Functions – Azure SignalR
description: Informations détaillées sur la façon de développer et de configurer des applications serverless en temps réel à l’aide d’Azure Functions et Azure SignalR Service
author: anthonychu
ms.service: signalr
ms.topic: conceptual
ms.date: 03/01/2019
ms.author: antchu
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: acb85a04b8a1ca491058702510079a36b93fc657
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151044"
---
# <a name="azure-functions-development-and-configuration-with-azure-signalr-service"></a>Développement et configuration Azure Functions avec Azure SignalR Service

Les applications Azure Functions peuvent exploiter les [liaisons Azure SignalR Service](../azure-functions/functions-bindings-signalr-service.md) pour ajouter des fonctionnalités en temps réel. Les applications clientes utilisent des SDK clients disponibles dans plusieurs langages pour se connecter à Azure SignalR Service et recevoir des messages en temps réel.

Cet article décrit les concepts du développement et de la configuration d’une application de fonction Azure qui est intégrée à SignalR Service.

## <a name="signalr-service-configuration"></a>Configuration de SignalR Service

Azure SignalR Service peut être configuré dans des modes différents. Quand il est utilisé avec Azure Functions, le service doit être configuré en mode *Serverless*.

Dans le portail Azure, recherchez la page *Paramètres* de votre ressource SignalR Service. Définissez le *Mode de service* sur *Serverless*.

![Mode SignalR Service](media/signalr-concept-azure-functions/signalr-service-mode.png)

## <a name="azure-functions-development"></a>Développement Azure Functions

Une application serverless en temps réel intégrée à Azure Functions et Azure SignalR Service nécessite généralement deux fonctions Azure :

* Une fonction « negotiate » que le client appelle pour obtenir un jeton d’accès SignalR Service valide et une URL de point de terminaison du service
* Une ou plusieurs fonctions qui gèrent les messages de SignalR Service et envoient des messages ou gèrent l’appartenance à un groupe

### <a name="negotiate-function"></a>Fonction negotiate

Une application cliente nécessite un jeton d’accès valide pour se connecter à Azure SignalR Service. Un jeton d’accès peut être anonyme ou authentifié avec un ID d’utilisateur donné. Les applications SignalR Service serverless nécessitent un point de terminaison HTTP nommé « negotiate » pour obtenir un jeton et d’autres informations de connexion, telles que l’URL de point de terminaison de SignalR Service.

Utilisez une fonction Azure déclenchée via HTTP et la liaison d’entrée *SignalRConnectionInfo* pour générer l’objet d’informations de connexion. La fonction doit avoir une route HTTP qui se termine par `/negotiate`.

Avec le [modèle basé sur la classe](#class-based-model) en C#, vous n’avez pas besoin de liaison d’entrée *SignalRConnectionInfo* et pouvez ajouter des revendications personnalisées beaucoup plus facilement. Consultez [Négocier l’expérience dans un modèle basé sur la classe](#negotiate-experience-in-class-based-model).

Pour plus d’informations sur la création de la fonction negotiate, consultez les [informations de référence sur la liaison d’entrée *SignalRConnectionInfo*](../azure-functions/functions-bindings-signalr-service-input.md).

Pour en savoir plus sur la création d’un jeton authentifié, reportez-vous à [Utilisation de l’authentification App Service](#using-app-service-authentication).

### <a name="handle-messages-sent-from-signalr-service"></a>Gérer les messages envoyés par SignalR Service

Utilisez la liaison de déclencheur *SignalR* pour gérer les messages envoyés par SignalR Service. Le déclenchement peut s’effectuer quand des clients envoient des messages ou quand ils sont connectés ou déconnectés.

Pour plus d'informations, consultez les [informations de référence sur la liaison de *déclencheur SignalR*](../azure-functions/functions-bindings-signalr-service-trigger.md).

Vous devez également configurer le point de terminaison de votre fonction en amont afin que le service déclenche la fonction là où il y a un message du client. Pour plus d'informations sur la configuration en amont, consultez ce [document](concept-upstream.md).

### <a name="sending-messages-and-managing-group-membership"></a>Envoi de messages et gestion de l’appartenance au groupe

Utilisez la liaison de sortie *SignalR* pour envoyer des messages aux clients connectés à Azure SignalR Service. Vous pouvez diffuser des messages à tous les clients, ou vous pouvez les envoyer à un sous-ensemble de clients qui sont authentifiés avec un ID utilisateur spécifique ou qui ont été ajoutés à un groupe spécifique.

Les utilisateurs peuvent être ajoutés à un ou plusieurs groupes. Vous pouvez également utiliser la liaison de sortie *SignalR* pour ajouter des utilisateurs à des groupes ou en en supprimer.

Pour plus d’informations, consultez les [informations de référence sur la liaison de sortie *SignalR*](../azure-functions/functions-bindings-signalr-service-output.md).

### <a name="signalr-hubs"></a>Hubs SignalR

SignalR a un concept de « hubs ». L’étendue de chaque connexion cliente et de chaque message envoyé à partir d’Azure Functions correspond à un hub spécifique. Vous pouvez utiliser des hubs pour organiser vos connexions et messages en espaces de noms logiques.

## <a name="class-based-model"></a>Modèle basé sur la classe

Le modèle basé sur la classe est dédié à C#. Il fournit une expérience de programmation cohérente côté serveur SignalR. Il présente les caractéristiques suivantes.

* Moins de tâches de configuration : le nom de la classe est utilisé comme `HubName`, le nom de la méthode est utilisé comme `Event`, et le `Category` est déterminé automatiquement en fonction du nom de la méthode.
* Liaison de paramètre automatique : ni `ParameterNames` ni l’attribut `[SignalRParameter]` ne sont nécessaires. Les paramètres sont liés automatiquement aux arguments de la méthode de fonction Azure dans l’ordre.
* Expérience de sortie et de négociation pratique.

Les codes suivants illustrent ces fonctionnalités :

```cs
public class SignalRTestHub : ServerlessHub
{
    [FunctionName("negotiate")]
    public SignalRConnectionInfo Negotiate([HttpTrigger(AuthorizationLevel.Anonymous)]HttpRequest req)
    {
        return Negotiate(req.Headers["x-ms-signalr-user-id"], GetClaims(req.Headers["Authorization"]));
    }

    [FunctionName(nameof(OnConnected))]
    public async Task OnConnected([SignalRTrigger]InvocationContext invocationContext, ILogger logger)
    {
        await Clients.All.SendAsync(NewConnectionTarget, new NewConnection(invocationContext.ConnectionId));
        logger.LogInformation($"{invocationContext.ConnectionId} has connected");
    }

    [FunctionName(nameof(Broadcast))]
    public async Task Broadcast([SignalRTrigger]InvocationContext invocationContext, string message, ILogger logger)
    {
        await Clients.All.SendAsync(NewMessageTarget, new NewMessage(invocationContext, message));
        logger.LogInformation($"{invocationContext.ConnectionId} broadcast {message}");
    }

    [FunctionName(nameof(OnDisconnected))]
    public void OnDisconnected([SignalRTrigger]InvocationContext invocationContext)
    {
    }
}
```

Toutes les fonctions qui souhaitent tirer parti du modèle basé sur la classe doivent être la méthode de classe qui hérite de **ServerlessHub**. Le nom de classe `SignalRTestHub` dans l’exemple est le nom du hub.

### <a name="define-hub-method"></a>Définir la méthode de hub

Toutes les méthodes de hub **doivent** avoir un argument `InvocationContext` décoré par l'attribut `[SignalRTrigger]` et utiliser un constructeur sans paramètre. Le **nom de la méthode** est alors traité comme **événement** de paramètre.

Par défaut, `category=messages` sauf le nom de la méthode est l’un des noms suivants :

* **OnConnected** : traité comme `category=connections, event=connected`.
* **OnDisconnected** : traité comme `category=connections, event=disconnected`.

### <a name="parameter-binding-experience"></a>Expérience de liaison de paramètre

Dans un modèle basé sur la classe, `[SignalRParameter]` n’est pas nécessaire, car tous les arguments sont marqués comme `[SignalRParameter]` par défaut sauf dans les cas suivants :

* L’argument est décoré par un attribut de liaison
* Le type de l’argument est `ILogger` ou `CancellationToken`
* L’argument est décoré par l’attribut `[SignalRIgnore]`

### <a name="negotiate-experience-in-class-based-model"></a>Négocier l’expérience dans un modèle basé sur la classe

Au lieu d’utiliser la liaison d’entrée SignalR `[SignalR]`, la négociation dans un modèle basé sur la classe peut être plus flexible. La classe de base `ServerlessHub` a une méthode

```cs
SignalRConnectionInfo Negotiate(string userId = null, IList<Claim> claims = null, TimeSpan? lifeTime = null)
```

Cette fonctionnalité personnalise `userId` ou `claims` lors de l’exécution de la fonction.

## <a name="use-signalrfilterattribute"></a>Utilisez `SignalRFilterAttribute`.

L’utilisateur peut hériter et implémenter la classe abstraite `SignalRFilterAttribute`. Si des exceptions sont levées dans `FilterAsync`, une erreur `403 Forbidden` est retournée aux clients.

L’exemple suivant montre comment implémenter un filtre client qui autorise uniquement `admin` à appeler `broadcast`.

```cs
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true, Inherited = true)]
internal class FunctionAuthorizeAttribute: SignalRFilterAttribute
{
    private const string AdminKey = "admin";

    public override Task FilterAsync(InvocationContext invocationContext, CancellationToken cancellationToken)
    {
        if (invocationContext.Claims.TryGetValue(AdminKey, out var value) &&
            bool.TryParse(value, out var isAdmin) &&
            isAdmin)
        {
            return Task.CompletedTask;
        }

        throw new Exception($"{invocationContext.ConnectionId} doesn't have admin role");
    }
}
```

Tirez parti de l’attribut pour autoriser la fonction.

```cs
[FunctionAuthorize]
[FunctionName(nameof(Broadcast))]
public async Task Broadcast([SignalRTrigger]InvocationContext invocationContext, string message, ILogger logger)
{
}
```

## <a name="client-development"></a>Développement client

Les applications clientes SignalR peuvent tirer parti du SDK client SignalR dans un des langages pour se connecter facilement à Azure SignalR Service et recevoir des messages de ce dernier.

### <a name="configuring-a-client-connection"></a>Configuration d’une connexion cliente

Pour se connecter à SignalR Service, un client doit mener à bien une négociation de connexion qui se compose des étapes suivantes :

1. Adresser une demande au point de terminaison HTTP *negotiate* décrit plus haut pour obtenir des informations de connexion valides
1. Se connecter à SignalR Service à l’aide de l’URL du point de terminaison de service et du jeton d’accès obtenus auprès du point de terminaison *negotiate*

Les SDK clients SignalR contiennent déjà la logique nécessaire pour effectuer la procédure de négociation. Passez l’URL du point de terminaison negotiate, moins le segment `negotiate`, au `HubConnectionBuilder` du SDK. Voici un exemple en JavaScript :

```javascript
const connection = new signalR.HubConnectionBuilder()
  .withUrl('https://my-signalr-function-app.azurewebsites.net/api')
  .build()
```

Par convention, le SDK ajoute automatiquement `/negotiate` à l’URL et l’utilise pour commencer la négociation.

> [!NOTE]
> Si vous utilisez le SDK JavaScript/TypeScript dans un navigateur, vous devez [activer le partage de ressources entre origines multiples (CORS, Cross-Origin Resource Sharing)](#enabling-cors) sur votre application de fonction.

Pour plus d’informations sur la façon d’utiliser le SDK client SignalR, consultez la documentation de votre langage :

* [.NET Standard](/aspnet/core/signalr/dotnet-client)
* [JavaScript](/aspnet/core/signalr/javascript-client)
* [Java](/aspnet/core/signalr/java-client)

### <a name="sending-messages-from-a-client-to-the-service"></a>Envoi de messages à partir d’un client au service

Si votre ressource SignalR bénéficie d'une configuration [en amont](concept-upstream.md), vous pouvez envoyer des messages entre le client et votre instance d'Azure Functions à l'aide de n'importe quel client SignalR. Voici un exemple en JavaScript :

```javascript
connection.send('method1', 'arg1', 'arg2');
```

## <a name="azure-functions-configuration"></a>Configuration Azure Functions

Les applications de fonction Azure qui s’intègrent à Azure SignalR Service peuvent être déployées comme toute application de fonction Azure classique, à l’aide de techniques telles que le [déploiement en continu](../azure-functions/functions-continuous-deployment.md), le [déploiement Zip](../azure-functions/deployment-zip-push.md) et l’[exécution à partir du package](../azure-functions/run-functions-from-deployment-package.md).

Toutefois, il existe quelques considérations spéciales pour les applications qui utilisent les liaisons SignalR Service. Si le client s’exécute dans un navigateur, CORS doit être activé. En outre, si l’application requiert une authentification, vous pouvez intégrer le point de terminaison negotiate à l’authentification App Service.

### <a name="enabling-cors"></a>Activation de CORS

Le client JavaScript/TypeScript adresse des requêtes HTTP à la fonction negotiate pour lancer la négociation de la connexion. Quand l’application cliente est hébergée sur un autre domaine que l’application de fonction Azure, CORS doit être activé sur l’application de fonction, sinon le navigateur bloque les demandes.

#### <a name="localhost"></a>Localhost

Quand vous exécutez l’application de fonction sur votre ordinateur local, vous pouvez ajouter une section `Host` à *local.settings.json* pour activer CORS. Dans la section `Host`, ajoutez deux propriétés :

* `CORS` : entrez l’URL de base qui est l’origine de l’application cliente
* `CORSCredentials` : définissez cette propriété sur `true` pour autoriser les demandes « withCredentials »

Exemple :

```json
{
  "IsEncrypted": false,
  "Values": {
    // values
  },
  "Host": {
    "CORS": "http://localhost:8080",
    "CORSCredentials": true
  }
}
```

#### <a name="cloud---azure-functions-cors"></a>Cloud - CORS Azure Functions

Pour activer CORS sur une application de fonction Azure, accédez à l’écran de configuration de CORS sous l’onglet *Fonctionnalités de la plateforme* de votre application de fonction dans le portail Azure.

> [!NOTE]
> La configuration CORS n’est pas encore disponible dans le plan de consommation Linux Azure Functions. Utilisez [Gestion des API Azure](#cloud---azure-api-management) pour activer CORS.

CORS avec Access-Control-Allow-Credentials doit être activé pour que le client SignalR appelle la fonction negotiate. Cochez la case pour l’activer.

Dans la section *Origines autorisées*, ajoutez une entrée contenant l’URL de base d’origine de votre application web.

![Configuration de CORS](media/signalr-concept-serverless-development-config/cors-settings.png)

#### <a name="cloud---azure-api-management"></a>Cloud - Gestion des API Azure

La gestion des API Azure fournit une passerelle API qui ajoute des capacités aux services back-end existants. Vous pouvez l’utiliser pour ajouter CORS à votre application de fonction. Elle offre un niveau de consommation avec un tarif de paiement par action et un octroi gratuit mensuel.

Pour plus d’informations sur la façon d’[importer une application Azure Function](../api-management/import-function-app-as-api.md), reportez-vous à la documentation Gestion des API. Une fois l’importation effectuée, vous pouvez ajouter une stratégie de trafic entrant pour activer CORS avec une prise en charge d’Access-Control-Allow-Credentials.

```xml
<cors allow-credentials="true">
  <allowed-origins>
    <origin>https://azure-samples.github.io</origin>
  </allowed-origins>
  <allowed-methods>
    <method>GET</method>
    <method>POST</method>
  </allowed-methods>
  <allowed-headers>
    <header>*</header>
  </allowed-headers>
  <expose-headers>
    <header>*</header>
  </expose-headers>
</cors>
```

Configurez vos clients SignalR pour qu’ils utilisent l’URL de gestion des API.

### <a name="using-app-service-authentication"></a>Utilisation de l’authentification App Service

Azure Functions offre une authentification intégrée, prenant en charge les fournisseurs les plus courants tels que Facebook, Twitter, Microsoft Account, Google et Azure Active Directory. Cette fonctionnalité peut être intégrée à la liaison *SignalRConnectionInfo* pour créer des connexions à Azure SignalR Service qui ont été authentifiées avec un ID utilisateur. À l’aide de la liaison de sortie *SignalR*, votre application peut envoyer des messages qui sont destinés à cet ID d’utilisateur.

Dans le portail Azure, sous l’onglet *Fonctionnalités de la plateforme* de votre application de fonction, ouvrez la fenêtre de paramètres *Authentification/autorisation*. Consultez la documentation relative à l’[authentification App Service](../app-service/overview-authentication-authorization.md) pour configurer l’authentification à l’aide du fournisseur d’identité de votre choix.

Une fois la configuration effectuée, les requêtes HTTP authentifiées incluent des en-têtes `x-ms-client-principal-name` et `x-ms-client-principal-id` contenant respectivement le nom d’utilisateur et l’ID utilisateur de l’identité authentifiée.

Vous pouvez utiliser ces en-têtes dans la configuration de la liaison *SignalRConnectionInfo* pour créer des connexions authentifiées. Voici un exemple de fonction negotiate en C# qui utilise l’en-tête `x-ms-client-principal-id`.

```csharp
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

Vous pouvez ensuite envoyer des messages à cet utilisateur en définissant la propriété `UserId` d’un message SignalR.

```csharp
[FunctionName("SendMessage")]
public static Task SendMessage(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")]object message,
    [SignalR(HubName = "chat")]IAsyncCollector<SignalRMessage> signalRMessages)
{
    return signalRMessages.AddAsync(
        new SignalRMessage
        {
            // the message will only be sent to these user IDs
            UserId = "userId1",
            Target = "newMessage",
            Arguments = new [] { message }
        });
}
```

Pour plus d’informations sur les autres langages, consultez les informations de référence sur les [liaisons Azure SignalR Service](../azure-functions/functions-bindings-signalr-service.md) pour Azure Functions.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à développer et configurer des applications SignalR Service serverless à l’aide d’Azure Functions. Essayez de créer une application vous-même à l’aide d’un des guides de démarrage rapide ou tutoriels proposés dans la [page de vue d’ensemble de SignalR Service](index.yml).