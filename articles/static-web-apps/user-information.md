---
title: Accès aux informations utilisateur dans Azure Static Web Apps
description: Apprendre à lire les données utilisateur retournées par le fournisseur d’autorisation.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.custom: devx-track-js
ms.openlocfilehash: d5a1d810c357aa83b8069023b00d76352da124df
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844793"
---
# <a name="accessing-user-information-in-azure-static-web-apps-preview"></a>Accès aux informations utilisateur dans Azure Static Web Apps - Préversion

Azure Static Web Apps fournit des informations utilisateur relatives à l’authentification via un [point de terminaison à accès direct](#direct-access-endpoint) et à l’[API Functions](#api-functions).

De nombreuses interfaces utilisateur s’appuient largement sur les données d’authentification de l’utilisateur. Le point de terminaison à accès direct est une API qui expose les informations utilisateur sans avoir à implémenter de fonction personnalisée. Au-delà de sa simplicité, le point de terminaison à accès direct n’est pas soumis aux retards de démarrage à froid associés à l’architecture serverless.

## <a name="client-principal-data"></a>Données du principal du client.

L’objet de données du principal du client expose les informations utilisateur identifiables pour votre application. Les propriétés suivantes sont disponibles dans l’objet du principal du client :

| Propriété  | Description |
|-----------|---------|
| `identityProvider` | Nom du [fournisseur d’identité](authentication-authorization.md). |
| `userId` | Identificateur unique spécifique à Azure Static Web Apps pour l’utilisateur. <ul><li>La valeur est unique pour chaque application. Par exemple, le même utilisateur retourne une autre valeur `userId` sur une autre ressource Static Web Apps.<li>La valeur persiste pendant toute la durée de vie de l’utilisateur. Si vous supprimez et que vous rajoutez le même utilisateur à l’application, une nouvelle valeur `userId` est générée.</ul>|
| `userDetails` | Nom d’utilisateur ou adresse e-mail de l’utilisateur. Certains fournisseurs retournent l’[adresse e-mail](authentication-authorization.md) de l’utilisateur, tandis que d’autres envoient le [nom d’utilisateur](authentication-authorization.md). |
| `userRoles`     | Tableau des [rôles affectés à l’utilisateur](authentication-authorization.md). |

L’exemple suivant est un exemple d’objet de principal de client :

```json
{
  "identityProvider": "facebook",
  "userId": "d75b260a64504067bfc5b2905e3b8182",
  "userDetails": "user@example.com",
  "userRoles": [ "anonymous", "authenticated" ]
}
```

## <a name="direct-access-endpoint"></a>Point de terminaison à accès direct

Vous pouvez envoyer une demande `GET` à l’itinéraire `/.auth/me` et recevoir un accès direct aux données du principal du client. Lorsque l’état de votre vue s’appuie sur les données d’autorisation, utilisez cette approche pour bénéficier de performances optimales.

Pour les utilisateurs connectés, la réponse contient un objet JSON du principal du client. Les demandes provenant d’utilisateurs non authentifiés renvoient `null`.

À l’aide de l’API [fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch)<sup>1</sup>, vous pouvez accéder aux données du principal du client à l’aide de la syntaxe suivante.

```javascript
async function getUserInfo() {
  const response = await fetch("/.auth/me");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}

console.log(getUserInfo());
```

## <a name="api-functions"></a>Fonctions de l’API

Les fonctions d’API disponibles dans Static Web Apps via le serveur principal Azure Functions ont accès aux mêmes informations utilisateur qu’une application cliente. Si l’API reçoit des informations identifiables par l’utilisateur, elle ne vérifie pas si l’utilisateur est authentifié ou s’il correspond à un rôle requis. Les règles de contrôle d’accès sont définies dans le fichier [`routes.json`](routes.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les données du principal du client sont transmises aux fonctions API dans l’en-tête de demande `x-ms-client-principal`. Les données du principal du client sont envoyées en tant que chaîne codée en [base64](https://www.wikipedia.org/wiki/Base64)contenant un objet JSON sérialisé.

L’exemple de fonction suivant montre comment lire et retourner des informations utilisateur.

```javascript
module.exports = async function (context, req) {
  const header = req.headers["x-ms-client-principal"];
  const encoded = Buffer.from(header, "base64");
  const decoded = encoded.toString("ascii");

  context.res = {
    body: {
      clientPrincipal: JSON.parse(decoded)
    }
  };
};
```

En supposant que le nom de la fonction ci-dessus est `user`, vous pouvez utiliser l’API de navigateur [fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch)<sup>1</sup> pour accéder à la réponse de l’API à l’aide de la syntaxe suivante.

```javascript
async function getUser() {
  const response = await fetch("/api/user");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}

console.log(await getUser());
```

# <a name="c"></a>[C#](#tab/csharp)

Dans une fonction C#, les informations utilisateur sont disponibles dans l’en-tête `x-ms-client-principal` qui peut être désérialisé dans un objet `ClaimsPrincipal` ou dans votre propre type personnalisé. Le code suivant montre comment décompresser l’en-tête dans un type intermédiaire, `ClientPrincipal`, qui est ensuite converti en instance `ClaimsPrincipal`.

```csharp
  public static class StaticWebAppsAuth
  {
    private class ClientPrincipal
    {
        public string IdentityProvider { get; set; }
        public string UserId { get; set; }
        public string UserDetails { get; set; }
        public IEnumerable<string> UserRoles { get; set; }
    }

    public static ClaimsPrincipal Parse(HttpRequest req)
    {
        var principal = new ClientPrincipal();

        if (req.Headers.TryGetValue("x-ms-client-principal", out var header))
        {
            var data = header[0];
            var decoded = Convert.FromBase64String(data);
            var json = Encoding.ASCII.GetString(decoded);
            principal = JsonSerializer.Deserialize<ClientPrincipal>(json, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        }

        principal.UserRoles = principal.UserRoles?.Except(new string[] { "anonymous" }, StringComparer.CurrentCultureIgnoreCase);

        if (!principal.UserRoles?.Any() ?? true)
        {
            return new ClaimsPrincipal();
        }

        var identity = new ClaimsIdentity(principal.IdentityProvider);
        identity.AddClaim(new Claim(ClaimTypes.NameIdentifier, principal.UserId));
        identity.AddClaim(new Claim(ClaimTypes.Name, principal.UserDetails));
        identity.AddClaims(principal.UserRoles.Select(r => new Claim(ClaimTypes.Role, r)));

        return new ClaimsPrincipal(identity);
    }
  }
```

---

<sup>1</sup> L’API [fetch](https://caniuse.com/#feat=fetch) et l’opérateur [await](https://caniuse.com/#feat=mdn-javascript_operators_await) ne sont pas pris en charge dans Internet Explorer.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer les paramètres d’application](application-settings.md)
