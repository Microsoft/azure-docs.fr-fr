---
title: Acquérir des jetons pour appeler une API web (application démon) - Plateforme d’identités Microsoft | Azure
description: Apprenez à générer une application démon qui appelle des API web (acquisition de jetons)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: c13b6ed991403e65c4c4d71c964f1f7f4d1ffe7b
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443311"
---
# <a name="daemon-app-that-calls-web-apis---acquire-a-token"></a>Application démon conçue pour appeler des API web - acquisition d'un jeton

Une fois l’application cliente confidentielle construite, vous pouvez acquérir un jeton pour celle-ci en appelant `AcquireTokenForClient`, en transmettant l’étendue, voire en forçant une actualisation du jeton.

## <a name="scopes-to-request"></a>Étendues à demander

L'étendue à demander pour un flux d'informations d'identification client est le nom de la ressource suivi de `/.default`. Cette notation indique à Azure Active Directory (Azure AD) d’utiliser les *autorisations de niveau application* déclarées de manière statique lors de l’inscription de l’application. En outre, ces autorisations d’API doivent être accordées par un administrateur client.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
ResourceId = "someAppIDURI";
var scopes = new [] {  ResourceId+"/.default"};
```

# <a name="python"></a>[Python](#tab/python)

Dans MSAL Python, le fichier de configuration ressemble à l’extrait de code suivant :

```Json
{
    "scope": ["https://graph.microsoft.com/.default"],
}
```

# <a name="java"></a>[Java](#tab/java)

```Java
final static String GRAPH_DEFAULT_SCOPE = "https://graph.microsoft.com/.default";
```

---

### <a name="azure-ad-v10-resources"></a>Ressources Azure AD (v1.0)

L’étendue utilisée pour les informations d’identification client doit toujours être l’ID de ressource suivi de `/.default`.

> [!IMPORTANT]
> Quand MSAL demande un jeton d’accès pour une ressource acceptant un jeton d’accès de version 1.0, Azure AD analyse l’audience souhaitée d’après l’étendue demandée, en prenant tout ce qui précède la dernière barre oblique et en l’utilisant comme identificateur de la ressource.
> Par conséquent, si, comme Azure SQL Database (**https :\//database.windows.net**), la ressource attend une audience se terminant par une barre oblique (pour Azure SQL Database, `https://database.windows.net/`), vous devez demander une étendue de `https://database.windows.net//.default` (notez la double barre oblique). Voir aussi le problème MSAL.NET [#747 : la barre oblique de fin est omise dans l’URL de la ressource, ce qui a entraîné un échec d’authentification sql ](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/747).

## <a name="acquiretokenforclient-api"></a>API AcquireTokenForClient

Pour acquérir un jeton pour l’application, vous devez utiliser `AcquireTokenForClient` ou l’équivalent, en fonction de la plateforme.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
using Microsoft.Identity.Client;

// With client credentials flows, the scope is always of the shape "resource/.default" because the
// application permissions need to be set statically (in the portal or by PowerShell), and then granted by
// a tenant administrator.
string[] scopes = new string[] { "https://graph.microsoft.com/.default" };

AuthenticationResult result = null;
try
{
 result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
    // The application doesn't have sufficient permissions.
    // - Did you declare enough app permissions during app creation?
    // - Did the tenant admin grant permissions to the application?
}
catch (MsalServiceException ex) when (ex.Message.Contains("AADSTS70011"))
{
    // Invalid scope. The scope has to be in the form "https://resourceurl/.default"
    // Mitigation: Change the scope to be as expected.
}
```

# <a name="python"></a>[Python](#tab/python)

```Python
# The pattern to acquire a token looks like this.
result = None

# First, the code looks up a token from the cache.
# Because we're looking for a token for the current app, not for a user,
# use None for the account parameter.
result = app.acquire_token_silent(config["scope"], account=None)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    result = app.acquire_token_for_client(scopes=config["scope"])

if "access_token" in result:
    # Call a protected API with the access token.
    print(result["token_type"])
else:
    print(result.get("error"))
    print(result.get("error_description"))
    print(result.get("correlation_id"))  # You might need this when reporting a bug.
```

# <a name="java"></a>[Java](#tab/java)

Ce code est extrait des [exemples de développement MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/confidential-client/).

```Java
private static IAuthenticationResult acquireToken() throws Exception {

     // Load token cache from file and initialize token cache aspect. The token cache will have
     // dummy data, so the acquireTokenSilently call will fail.
     TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

     IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);
     ConfidentialClientApplication cca =
             ConfidentialClientApplication
                     .builder(CLIENT_ID, credential)
                     .authority(AUTHORITY)
                     .setTokenCacheAccessAspect(tokenCacheAspect)
                     .build();

     IAuthenticationResult result;
     try {
         SilentParameters silentParameters =
                 SilentParameters
                         .builder(SCOPE)
                         .build();

         // try to acquire token silently. This call will fail since the token cache does not
         // have a token for the application you are requesting an access token for
         result = cca.acquireTokenSilently(silentParameters).join();
     } catch (Exception ex) {
         if (ex.getCause() instanceof MsalException) {

             ClientCredentialParameters parameters =
                     ClientCredentialParameters
                             .builder(SCOPE)
                             .build();

             // Try to acquire a token. If successful, you should see
             // the token information printed out to console
             result = cca.acquireToken(parameters).join();
         } else {
             // Handle other exceptions accordingly
             throw ex;
         }
     }
     return result;
 }
```

---

### <a name="protocol"></a>Protocol

Si vous n’avez pas encore de bibliothèque pour la langue de votre choix, vous pouvez utiliser directement le protocole :

#### <a name="first-case-access-the-token-request-by-using-a-shared-secret"></a>Premier cas : accéder à la demande de jeton à l’aide d’un secret partagé

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity.
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

#### <a name="second-case-access-the-token-request-by-using-a-certificate"></a>Deuxième cas : accéder à la demande de jeton à l’aide d’un certificat

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity.
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

Pour plus d'informations, consultez la documentation du protocole : [Plateforme d’identités Microsoft et flux d’informations d’identification du client OAuth 2.0](v2-oauth2-client-creds-grant-flow.md).

## <a name="application-token-cache"></a>Cache de jetons d'application

Dans MSAL.NET, `AcquireTokenForClient` utilise le cache de jetons de l’application (toutes les autres méthodes AcquireToken *XX* utilisent le cache de jetons de l’utilisateur). N’appelez pas `AcquireTokenSilent` avant d’appeler `AcquireTokenForClient`, car `AcquireTokenSilent` utilise le cache de jetons de l’*utilisateur*. `AcquireTokenForClient` vérifie le cache de jetons d'*application* et le met à jour.

## <a name="troubleshooting"></a>Dépannage

### <a name="did-you-use-the-resourcedefault-scope"></a>Avez-vous utilisé l'étendue resource/.default ?

Si vous recevez un message d'erreur indiquant que vous avez utilisé une étendue non valide, cela signifie probablement que vous n'avez pas utilisé l'étendue `resource/.default`.

### <a name="did-you-forget-to-provide-admin-consent-daemon-apps-need-it"></a>Avez-vous oublié de fournir le consentement administrateur ? Les applications démon en ont besoin !

Si vous rencontrez l’erreur **Privilèges insuffisants pour effectuer l’opération** lors de l’appel de l’API, cela signifie que l’administrateur client doit accorder des autorisations à l’application. Consultez l'étape 6 de la section Inscrire l'application cliente ci-dessus.
En général, une erreur similaire à celle-ci s’affiche :

```json
Failed to call the web API: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<guid>",
      "date": "<date>"
    }
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

# <a name="net"></a>[.NET](#tab/dotnet)

Passez à l’article suivant de ce scénario, [Appeler une API web](./scenario-daemon-call-api.md?tabs=dotnet).

# <a name="python"></a>[Python](#tab/python)

Passez à l’article suivant de ce scénario, [Appeler une API web](./scenario-daemon-call-api.md?tabs=python).

# <a name="java"></a>[Java](#tab/java)

Passez à l’article suivant de ce scénario, [Appeler une API web](./scenario-daemon-call-api.md?tabs=java).

---