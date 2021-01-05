---
title: Mettre à l’échelle avec plusieurs instances – Azure SignalR Service
description: Dans de nombreux scénarios de mise à l’échelle, le client doit souvent provisionner plusieurs instances et les configurer pour les utiliser ensemble, afin de créer un déploiement à grande échelle. Par exemple, le partitionnement nécessite la prise en charge de plusieurs instances.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 03/27/2019
ms.author: zhshang
ms.openlocfilehash: fd6ac8c4d4fc4c3fec4f549f8ef4f955e2b1c637
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89439212"
---
# <a name="how-to-scale-signalr-service-with-multiple-instances"></a>Comment mettre à l’échelle SignalR Service avec plusieurs instances ?
La dernière version du SDK de SignalR Service prend en charge plusieurs points de terminaison pour les instances de SignalR Service. Vous pouvez utiliser cette fonctionnalité pour la messagerie multirégion ou pour mettre à l’échelle des connexions simultanées.

## <a name="for-aspnet-core"></a>Pour ASP.NET Core

### <a name="how-to-add-multiple-endpoints-from-config"></a>Comment ajouter plusieurs points de terminaison à partir de la configuration ?

Effectuez la configuration avec la clé `Azure:SignalR:ConnectionString` ou `Azure:SignalR:ConnectionString:` pour la chaîne de connexion de SignalR Service.

Si la clé commence par `Azure:SignalR:ConnectionString:`, elle doit être au format `Azure:SignalR:ConnectionString:{Name}:{EndpointType}`, où `Name` et `EndpointType` sont des propriétés de l’objet `ServiceEndpoint` et sont accessibles à partir du code.

Vous pouvez ajouter plusieurs chaînes de connexion d’instance à l’aide des commandes `dotnet` suivantes :

```cmd
dotnet user-secrets set Azure:SignalR:ConnectionString:east-region-a <ConnectionString1>
dotnet user-secrets set Azure:SignalR:ConnectionString:east-region-b:primary <ConnectionString2>
dotnet user-secrets set Azure:SignalR:ConnectionString:backup:secondary <ConnectionString3>
```

### <a name="how-to-add-multiple-endpoints-from-code"></a>Comment ajouter plusieurs points de terminaison à partir du code ?

Une classe `ServicEndpoint` est introduite pour décrire les propriétés d’un point de terminaison Azure SignalR Service.
Vous pouvez configurer plusieurs points de terminaison d’instance quand vous utilisez le SDK de Service Azure SignalR au moyen du code suivant :
```cs
services.AddSignalR()
        .AddAzureSignalR(options => 
        {
            options.Endpoints = new ServiceEndpoint[]
            {
                // Note: this is just a demonstration of how to set options.Endpoints
                // Having ConnectionStrings explicitly set inside the code is not encouraged
                // You can fetch it from a safe place such as Azure KeyVault
                new ServiceEndpoint("<ConnectionString0>"),
                new ServiceEndpoint("<ConnectionString1>", type: EndpointType.Primary, name: "east-region-a"),
                new ServiceEndpoint("<ConnectionString2>", type: EndpointType.Primary, name: "east-region-b"),
                new ServiceEndpoint("<ConnectionString3>", type: EndpointType.Secondary, name: "backup"),
            };
        });
```

### <a name="how-to-customize-endpoint-router"></a>Comment personnaliser le routeur des points de terminaison ?

Par défaut, le SDK utilise le [DefaultEndpointRouter](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/EndpointRouters/DefaultEndpointRouter.cs) pour sélectionner les points de terminaison.

#### <a name="default-behavior"></a>Comportement par défaut 
1. Routage des demandes clientes

    Quand le client négocie (`/negotiate`) avec le serveur d’applications, par défaut, le SDK **sélectionne de manière aléatoire** un point de terminaison dans l’ensemble de points de terminaison de service disponibles.

2. Routage des messages de serveur

    Quand un message est envoyé à une connexion spécifique et que la connexion cible est routée vers le serveur actuel, le message parvient directement à ce point de terminaison connectée. Sinon, les messages sont diffusés à chaque point de terminaison Azure SignalR.

#### <a name="customize-routing-algorithm"></a>Personnaliser l’algorithme de routage
Vous pouvez créer votre propre routeur si vous avez suffisamment de connaissances pour identifier les points de terminaison vers lesquels les messages doivent être envoyés.

Dans l’exemple de routeur personnalisé ci-dessous, les groupes commençant par `east-` accèdent toujours au point de terminaison nommé `east` :

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override IEnumerable<ServiceEndpoint> GetEndpointsForGroup(string groupName, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the group broadcast behavior, if the group name starts with "east-", only send messages to endpoints inside east
        if (groupName.StartsWith("east-"))
        {
            return endpoints.Where(e => e.Name.StartsWith("east-"));
        }

        return base.GetEndpointsForGroup(groupName, endpoints);
    }
}
```

Dans l’autre exemple ci-dessous, le comportement de négociation par défaut est remplacé afin que les points de terminaison soient sélectionnés en fonction de l’emplacement du serveur d’applications.

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override ServiceEndpoint GetNegotiateEndpoint(HttpContext context, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the negotiate behavior to get the endpoint from query string
        var endpointName = context.Request.Query["endpoint"];
        if (endpointName.Count == 0)
        {
            context.Response.StatusCode = 400;
            var response = Encoding.UTF8.GetBytes("Invalid request");
            context.Response.Body.Write(response, 0, response.Length);
            return null;
        }

        return endpoints.FirstOrDefault(s => s.Name == endpointName && s.Online) // Get the endpoint with name matching the incoming request
               ?? base.GetNegotiateEndpoint(context, endpoints); // Or fallback to the default behavior to randomly select one from primary endpoints, or fallback to secondary when no primary ones are online
    }
}
```

N’oubliez pas d’inscrire le routeur sur le conteneur d’injection de dépendances comme suit :

```cs
services.AddSingleton(typeof(IEndpointRouter), typeof(CustomRouter));
services.AddSignalR()
        .AddAzureSignalR(
            options => 
            {
                options.Endpoints = new ServiceEndpoint[]
                {
                    new ServiceEndpoint(name: "east", connectionString: "<connectionString1>"),
                    new ServiceEndpoint(name: "west", connectionString: "<connectionString2>"),
                    new ServiceEndpoint("<connectionString3>")
                };
            });
```

## <a name="for-aspnet"></a>Pour ASP.NET

### <a name="how-to-add-multiple-endpoints-from-config"></a>Comment ajouter plusieurs points de terminaison à partir de la configuration ?

Effectuez la configuration avec la clé `Azure:SignalR:ConnectionString` ou `Azure:SignalR:ConnectionString:` pour la chaîne de connexion de SignalR Service.

Si la clé commence par `Azure:SignalR:ConnectionString:`, elle doit être au format `Azure:SignalR:ConnectionString:{Name}:{EndpointType}`, où `Name` et `EndpointType` sont des propriétés de l’objet `ServiceEndpoint` et sont accessibles à partir du code.

Vous pouvez ajouter plusieurs chaînes de connexion d’instance à `web.config` :

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="Azure:SignalR:ConnectionString" connectionString="<ConnectionString1>"/>
    <add name="Azure:SignalR:ConnectionString:en-us" connectionString="<ConnectionString2>"/>
    <add name="Azure:SignalR:ConnectionString:zh-cn:secondary" connectionString="<ConnectionString3>"/>
    <add name="Azure:SignalR:ConnectionString:Backup:secondary" connectionString="<ConnectionString4>"/>
  </connectionStrings>
  ...
</configuration>
```

### <a name="how-to-add-multiple-endpoints-from-code"></a>Comment ajouter plusieurs points de terminaison à partir du code ?

Une classe `ServicEndpoint` est introduite pour décrire les propriétés d’un point de terminaison Azure SignalR Service.
Vous pouvez configurer plusieurs points de terminaison d’instance quand vous utilisez le SDK de Service Azure SignalR au moyen du code suivant :

```cs
app.MapAzureSignalR(
    this.GetType().FullName, 
    options => {
            options.Endpoints = new ServiceEndpoint[]
            {
                // Note: this is just a demonstration of how to set options.Endpoints
                // Having ConnectionStrings explicitly set inside the code is not encouraged
                // You can fetch it from a safe place such as Azure KeyVault
                new ServiceEndpoint("<ConnectionString1>"),
                new ServiceEndpoint("<ConnectionString2>"),
                new ServiceEndpoint("<ConnectionString3>"),
            }
        });
```

### <a name="how-to-customize-router"></a>Comment personnaliser le routeur ?

La seule différence entre ASP.NET SignalR et ASP.NET Core SignalR est le type de contexte http pour `GetNegotiateEndpoint`. Pour ASP.NET SignalR, il s’agit du type [IOwinContext](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR.AspNet/EndpointRouters/DefaultEndpointRouter.cs#L19).

Voici l’exemple de négociation personnalisé pour SignalR ASP.NET :

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override ServiceEndpoint GetNegotiateEndpoint(IOwinContext context, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the negotiate behavior to get the endpoint from query string
        var endpointName = context.Request.Query["endpoint"];
        if (string.IsNullOrEmpty(endpointName))
        {
            context.Response.StatusCode = 400;
            context.Response.Write("Invalid request.");
            return null;
        }

        return endpoints.FirstOrDefault(s => s.Name == endpointName && s.Online) // Get the endpoint with name matching the incoming request
               ?? base.GetNegotiateEndpoint(context, endpoints); // Or fallback to the default behavior to randomly select one from primary endpoints, or fallback to secondary when no primary ones are online
    }
}
```

N’oubliez pas d’inscrire le routeur sur le conteneur d’injection de dépendances comme suit :

```cs
var hub = new HubConfiguration();
var router = new CustomRouter();
hub.Resolver.Register(typeof(IEndpointRouter), () => router);
app.MapAzureSignalR(GetType().FullName, hub, options => {
    options.Endpoints = new ServiceEndpoint[]
                {
                    new ServiceEndpoint(name: "east", connectionString: "<connectionString1>"),
                    new ServiceEndpoint(name: "west", connectionString: "<connectionString2>"),
                    new ServiceEndpoint("<connectionString3>")
                };
});
```

## <a name="configuration-in-cross-region-scenarios"></a>Configuration dans les scénarios inter-régions

L’objet `ServiceEndpoint` a une propriété `EndpointType` avec la valeur `primary` ou `secondary`.

Les points de terminaison `primary` sont des points de terminaison par défaut pour la réception du trafic client et sont considérés comme ayant des connexions réseau plus fiables ; les points de terminaison `secondary` sont considérés comme ayant des connexions réseau moins fiables et sont utilisés uniquement pour l’acheminement du trafic entre le serveur et le client, par exemple, pour la diffusion de messages, mais pas pour l’acheminement du trafic entre le client et le serveur.

Dans les cas impliquant plusieurs régions, le réseau peut être instable. Pour un même serveur d’applications situé dans la région *USA Est*, le point de terminaison SignalR Service situé dans la même région *USA Est* peut être configuré en tant que `primary` et les points de terminaison dans d’autres régions peuvent être marqués comme `secondary`. Dans cette configuration, les points de terminaison de service dans les autres régions peuvent **recevoir** les messages de ce serveur d’applications *USA Est*, mais aucun client **inter-régions** n’est routé vers ce serveur d’applications. L’architecture est illustrée dans le diagramme ci-après :

![Infrastructure multigéographique](./media/signalr-howto-scale-multi-instances/cross_geo_infra.png)

Quand un client tente de négocier (`/negotiate`) avec le serveur d’applications, avec le routeur par défaut, le SDK **sélectionne de manière aléatoire** un point de terminaison dans l’ensemble de points de terminaison `primary` disponibles. Si le point de terminaison principal n’est pas disponible, le SDK **effectue une sélection aléatoire** parmi tous les points de terminaison `secondary` disponibles. Le point de terminaison est marqué comme **disponible** quand la connexion entre le serveur et le point de terminaison de service est active.

Dans un scénario multirégion, quand un client tente de négocier (`/negotiate`) avec le serveur d’applications hébergé dans la région *USA Est*, par défaut, il retourne toujours le point de terminaison `primary` situé dans la même région. Si aucun point de terminaison *USA Est* n’est disponible, le client est redirigé vers des points de terminaison dans d’autres régions. La section Basculement ci-dessous décrit le scénario en détail.

![Négociation normale](./media/signalr-howto-scale-multi-instances/normal_negotiate.png)

## <a name="fail-over"></a>Basculement

Si aucun point de terminaison `primary` n’est disponible, le comportement de négociation du client (`/negotiate`) consiste à effectuer une sélection parmi les points de terminaison `secondary`. Ce mécanisme de basculement demande que chaque point de terminaison fasse office de point de terminaison `primary` sur au moins un serveur d’applications.

![Basculement](./media/signalr-howto-scale-multi-instances/failover_negotiate.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à configurer plusieurs instances dans la même application pour les scénarios de mise à l’échelle, de partitionnement et multirégions.

Les prises en charge de plusieurs points de terminaison peuvent également être utilisées dans les scénarios de haute disponibilité et reprise d’activité.

> [!div class="nextstepaction"]
> [Configurer SignalR Service pour la reprise d’activité et la haute disponibilité](./signalr-concept-disaster-recovery.md)
