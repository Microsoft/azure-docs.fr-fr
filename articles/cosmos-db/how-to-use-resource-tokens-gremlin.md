---
title: Utiliser des jetons de ressource Azure Cosmos DB avec le SDK Gremlin
description: Découvrez comment créer des jetons de ressource et les utiliser pour accéder à la base de données de graphe.
author: christopheranderson
ms.author: chrande
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 09/06/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 22c048b748806404ccfa580e660552a1744f3781
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93361691"
---
# <a name="use-azure-cosmos-db-resource-tokens-with-the-gremlin-sdk"></a>Utiliser des jetons de ressource Azure Cosmos DB avec le SDK Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article explique comment utiliser des [jetons de ressource Azure Cosmos DB](secure-access-to-data.md) pour accéder à la base de données de graphe via le SDK Gremlin.

## <a name="create-a-resource-token"></a>Créer un jeton de ressources

Le SDK Apache TinkerPop Gremlin ne dispose pas d’une API à utiliser pour créer des jetons de ressource. Le terme *jeton de ressource* est un concept d’Azure Cosmos DB. Pour créer des jetons de ressource, téléchargez le [SDK Azure Cosmos DB](sql-api-sdk-dotnet.md). Si votre application doit créer des jetons de ressource et les utiliser pour accéder à la base de données de graphe, elle a besoin de 2 SDK distincts.

La hiérarchie du modèle d’objets au-dessus des jetons de ressource est illustrée dans le plan suivant :

- **Compte Azure Cosmos DB**  : entité de plus haut niveau à laquelle est associé un DNS (par exemple `contoso.gremlin.cosmos.azure.com`).
  - **Base de données Azure Cosmos DB**
    - **Utilisateur**
      - **Permission**
        - **Jeton**  : propriété d’un objet d’autorisation qui indique les actions autorisées ou refusées.

Un jeton de ressource utilise le format suivant : `"type=resource&ver=1&sig=<base64 string>;<base64 string>;"`. Cette chaîne est opaque pour les clients et doit être utilisée telle quelle, sans modification ni interprétation.

```csharp
// Notice that document client is created against .NET SDK endpoint, rather than Gremlin.
DocumentClient client = new DocumentClient(
  new Uri("https://contoso.documents.azure.com:443/"), 
  "<primary key>", 
  new ConnectionPolicy 
  {
    EnableEndpointDiscovery = false, 
    ConnectionMode = ConnectionMode.Direct 
  });

  // Read specific permission to obtain a token.
  // The token isn't returned during the ReadPermissionReedAsync() call.
  // The call succeeds only if database id, user id, and permission id already exist. 
  // Note that <database id> is not a database name. It is a base64 string that represents the database identifier, for example "KalVAA==".
  // Similar comment applies to <user id> and <permission id>.
  Permission permission = await client.ReadPermissionAsync(UriFactory.CreatePermissionUri("<database id>", "<user id>", "<permission id>"));

  Console.WriteLine("Obtained token {0}", permission.Token);
}
```

## <a name="use-a-resource-token"></a>Utiliser un jeton de ressources
Vous pouvez utiliser des jetons de ressource directement comme propriété « password » quand vous construisez la classe GremlinServer.

```csharp
// The Gremlin application needs to be given a resource token. It can't discover the token on its own.
// You can obtain the token for a given permission by using the Azure Cosmos DB SDK, or you can pass it into the application as a command line argument or configuration value.
string resourceToken = GetResourceToken();

// Configure the Gremlin server to use a resource token rather than a primary key.
GremlinServer server = new GremlinServer(
  "contoso.gremlin.cosmosdb.azure.com",
  port: 443,
  enableSsl: true,
  username: "/dbs/<database name>/colls/<collection name>",

  // The format of the token is "type=resource&ver=1&sig=<base64 string>;<base64 string>;".
  password: resourceToken);

  using (GremlinClient gremlinClient = new GremlinClient(server, new GraphSON2Reader(), new GraphSON2Writer(), GremlinClient.GraphSON2MimeType))
  {
      await gremlinClient.SubmitAsync("g.V().limit(1)");
  }
```

La même approche fonctionne dans tous les SDK TinkerPop Gremlin.

```java
Cluster.Builder builder = Cluster.build();

AuthProperties authenticationProperties = new AuthProperties();
authenticationProperties.with(AuthProperties.Property.USERNAME,
    String.format("/dbs/%s/colls/%s", "<database name>", "<collection name>"));

// The format of the token is "type=resource&ver=1&sig=<base64 string>;<base64 string>;".
authenticationProperties.with(AuthProperties.Property.PASSWORD, resourceToken);

builder.authProperties(authenticationProperties);
```

## <a name="limit"></a>Limite

Avec un seul compte Gremlin, vous pouvez émettre un nombre illimité de jetons. Cependant, vous pouvez utiliser seulement jusqu’à 100 jetons simultanément au cours d’une heure. Si une application dépasse la limite de jetons par heure, une demande d’authentification est refusée et vous recevez le message d’erreur suivant : « La limite autorisée de 100 jetons de ressources pouvant être utilisés simultanément a été dépassée. » La fermeture des connexions actives utilisant des jetons spécifiques pour libérer des emplacements pour de nouveaux jetons ne fonctionne pas. Le moteur de base de données Azure Cosmos DB Gremlin effectue le suivi des jetons uniques au cours de l’heure précédant immédiatement la demande d’authentification.

## <a name="permission"></a>Autorisation

Une erreur courante que les applications rencontrent quand elles utilisent des jetons de ressource est « Autorisations insuffisantes fournies dans l’en-tête d’autorisation pour la demande correspondante. Réessayez avec un autre en-tête d’autorisation. » Cette erreur est retournée quand une traversée Gremlin tente d’écrire une arête ou un sommet, mais que le jeton de ressource accorde seulement des autorisations *Lecture*. Inspectez votre traversée pour voir si elle contient une des étapes suivantes : *.addV()* , *.addE()* , *.drop()* ou *.property()* .

## <a name="next-steps"></a>Étapes suivantes
* [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](role-based-access-control.md) dans Azure Cosmos DB
* [Découvrir comment sécuriser l’accès aux données](secure-access-to-data.md) dans Azure Cosmos DB
