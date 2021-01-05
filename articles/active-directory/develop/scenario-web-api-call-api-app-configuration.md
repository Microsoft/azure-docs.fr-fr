---
title: Configurer une API web qui appelle des API web | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment créer une API web qui appelle des API web (configuration du code de l’application)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/26/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 4a888c3ad771e4a7edbd7110ba584050fe68e810
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443787"
---
# <a name="a-web-api-that-calls-web-apis-code-configuration"></a>Une API web qui appelle des API web : Configuration de code

Une fois que vous avez inscrit votre API web, vous pouvez configurer le code pour l’application.

Le code que vous utilisez pour configurer votre API web afin qu’elle appelle des API web en aval s’appuie sur le code utilisé pour protéger une API web. Pour plus d’informations, consultez la section [API web protégée : Configuration d’application](scenario-protected-web-api-app-configuration.md).

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

## <a name="microsoftidentityweb"></a>Microsoft.Identity.Web

Microsoft vous recommande d’utiliser le package NuGet [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web) lors du développement d’une API protégée ASP.NET Core appelant des API web en aval. Voir [API web protégée : Configuration du code | Microsoft.Identity.Web](scenario-protected-web-api-app-configuration.md#microsoftidentityweb) pour une présentation rapide de cette bibliothèque dans le contexte d’une API web.

## <a name="client-secrets-or-client-certificates"></a>Secrets clients ou certificats clients

Dans la mesure où votre API web appelle désormais une API web en aval, vous devez fournir un secret client ou un certificat client dans le fichier *appsettings.json*. Vous pouvez également ajouter une section spécifiant ce qui suit :

- URL de l’API web en aval ;
- étendues requises pour appeler l’API.

Dans l’exemple suivant, la section `GraphBeta` spécifie ces paramètres.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common"

   // To call an API
   "ClientSecret": "[Copy the client secret added to the app from the Azure portal]",
   "ClientCertificates": [
  ]
 },
 "GraphBeta": {
    "BaseUrl": "https://graph.microsoft.com/beta",
    "Scopes": "user.read"
    }
}
```

À la place d’un secret client, vous pouvez fournir un certificat client. L’extrait de code suivant montre l’utilisation d’un certificat stocké dans Azure Key Vault.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common"

   // To call an API
   "ClientCertificates": [
      {
        "SourceType": "KeyVault",
        "KeyVaultUrl": "https://msidentitywebsamples.vault.azure.net",
        "KeyVaultCertificateName": "MicrosoftIdentitySamplesCert"
      }
   ]
  },
  "GraphBeta": {
    "BaseUrl": "https://graph.microsoft.com/beta",
    "Scopes": "user.read"
  }
}
```

Microsoft.Identity.Web propose plusieurs façons de décrire les certificats, que ce soit en définissant une configuration ou en écrivant du code. Pour plus d’informations, consultez [Microsoft.Identity.Web wiki - Using certificates](https://github.com/AzureAD/microsoft-identity-web/wiki/Using-certificates) sur GitHub.

## <a name="startupcs"></a>Startup.cs

Votre API web doit acquérir un jeton pour l’API en aval. Vous le spécifiez en ajoutant la ligne `.EnableTokenAcquisitionToCallDownstreamApi()` après `.AddMicrosoftIdentityWebApi(Configuration)`. Cette ligne expose le service `ITokenAcquisition` que vous pouvez utiliser dans vos actions de contrôleur et de pages. Toutefois, comme vous le verrez dans les deux points suivants, il y a encore plus simple. Vous devez également choisir une implémentation de cache de jeton, par exemple `.AddInMemoryTokenCaches()`, dans *Startup.cs* :

```csharp
using Microsoft.Identity.Web;

public class Startup
{
  // ...
  public void ConfigureServices(IServiceCollection services)
  {
  // ...
  services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
          .AddMicrosoftIdentityWebApi(Configuration, Configuration.GetSection("AzureAd"))
            .EnableTokenAcquisitionToCallDownstreamApi()
            .AddInMemoryTokenCaches();
   // ...
  }
  // ...
}
```

Si vous ne souhaitez pas acquérir le jeton vous-même, *Microsoft.Identity.Web* fournit deux mécanismes pour appeler une API web en aval à partir d’une autre API. L’option que vous choisissez varie selon que vous souhaitez appeler Microsoft Graph ou une autre API.

### <a name="option-1-call-microsoft-graph"></a>Option 1 : Appeler Microsoft Graph

Si vous souhaitez appeler Microsoft Graph, Microsoft.Identity.Web vous permet d’utiliser directement le `GraphServiceClient` (exposé par le Kit de développement logiciel (SDK) Microsoft Graph) dans vos actions d’API. Pour exposer Microsoft Graph :

1. Ajoutez le package NuGet [Microsoft.Identity.Web. MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) à votre projet.
1. Ajoutez `.AddMicrosoftGraph()` après `.EnableTokenAcquisitionToCallDownstreamApi()` dans le fichier *Startup.cs*. `.AddMicrosoftGraph()` a plusieurs remplacements. En utilisant le remplacement qui prend une section de configuration en tant que paramètre, le code devient :

```csharp
using Microsoft.Identity.Web;

public class Startup
{
  // ...
  public void ConfigureServices(IServiceCollection services)
  {
  // ...
  services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
          .AddMicrosoftIdentityWebApi(Configuration, Configuration.GetSection("AzureAd"))
            .EnableTokenAcquisitionToCallDownstreamApi()
               .AddMicrosoftGraph(Configuration.GetSection("GraphBeta"))
            .AddInMemoryTokenCaches();
   // ...
  }
  // ...
}
```

### <a name="option-2-call-a-downstream-web-api-other-than-microsoft-graph"></a>Option n°2 : Appeler une API web en aval autre que Microsoft Graph

Pour appeler une API en aval autre que Microsoft Graph, *Microsoft.Identity.Web* fournit `.AddDownstreamWebApi()`, qui demande des jetons et appelle l’API web en aval.

```csharp
using Microsoft.Identity.Web;

public class Startup
{
  // ...
  public void ConfigureServices(IServiceCollection services)
  {
  // ...
  services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
          .AddMicrosoftIdentityWebApi(Configuration, "AzureAd")
            .EnableTokenAcquisitionToCallDownstreamApi()
               .AddDownstreamWebApi("MyApi", Configuration.GetSection("GraphBeta"))
            .AddInMemoryTokenCaches();
   // ...
  }
  // ...
}
```

Comme pour les applications web, vous pouvez choisir diverses implémentations de cache de jetons. Pour plus d’informations, consultez [Microsoft identity web - Token cache serialization](https://aka.ms/ms-id-web/token-cache-serialization) sur GitHub.

L’illustration suivante montre les différentes possibilités de *Microsoft.Identity.Web* et leur impact sur le fichier *Startup.cs* :

:::image type="content" source="media/scenarios/microsoft-identity-web-startup-cs.svg" alt-text="Diagramme de bloc montrant des options de configuration de service dans le point de départ CS pour appeler une API web et spécifier une implémentation de cache de jeton":::

> [!NOTE]
> Pour bien comprendre les exemples de code présentés ici, vous devez maîtriser les [notions de base d'ASP.NET Core](/aspnet/core/fundamentals), en particulier l'[injection de dépendances](/aspnet/core/fundamentals/dependency-injection) et le modèle [options](/aspnet/core/fundamentals/configuration/options).

# <a name="java"></a>[Java](#tab/java)

Le flux On-behalf-of (OBO) permet d’obtenir un jeton pour appeler l’API web en aval. Dans ce flux, votre API web reçoit de l’application cliente un jeton du porteur avec des permissions déléguées par l’utilisateur, puis échange ce jeton contre un autre jeton d’accès pour appeler l’API web en aval.

Le code ci-dessous utilise le `SecurityContextHolder` de l’infrastructure de sécurité Spring dans l’API web pour récupérer le jeton du porteur validé. Il utilise ensuite la bibliothèque Java MSAL pour obtenir un jeton pour l’API en aval à l’aide de l’appel `acquireToken` avec `OnBehalfOfParameters`. MSAL met en cache le jeton afin que les appels subséquents à l’API puissent utiliser `acquireTokenSilently` pour récupérer ce jeton.

```Java
@Component
class MsalAuthHelper {

    @Value("${security.oauth2.client.authority}")
    private String authority;

    @Value("${security.oauth2.client.client-id}")
    private String clientId;

    @Value("${security.oauth2.client.client-secret}")
    private String secret;

    @Autowired
    CacheManager cacheManager;

    private String getAuthToken(){
        String res = null;
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if(authentication != null){
            res = ((OAuth2AuthenticationDetails) authentication.getDetails()).getTokenValue();
        }
        return res;
    }

    String getOboToken(String scope) throws MalformedURLException {
        String authToken = getAuthToken();

        ConfidentialClientApplication application =
                ConfidentialClientApplication.builder(clientId, ClientCredentialFactory.create(secret))
                        .authority(authority).build();

        String cacheKey = Hashing.sha256()
                .hashString(authToken, StandardCharsets.UTF_8).toString();

        String cachedTokens = cacheManager.getCache("tokens").get(cacheKey, String.class);
        if(cachedTokens != null){
            application.tokenCache().deserialize(cachedTokens);
        }

        IAuthenticationResult auth;
        SilentParameters silentParameters =
                SilentParameters.builder(Collections.singleton(scope))
                        .build();
        auth = application.acquireTokenSilently(silentParameters).join();

        if (auth == null){
            OnBehalfOfParameters parameters =
                    OnBehalfOfParameters.builder(Collections.singleton(scope),
                            new UserAssertion(authToken))
                            .build();

            auth = application.acquireToken(parameters).join();
        }

        cacheManager.getCache("tokens").put(cacheKey, application.tokenCache().serialize());

        return auth.accessToken();
    }
}
```

# <a name="python"></a>[Python](#tab/python)

Le flux On-behalf-of (OBO) permet d’obtenir un jeton pour appeler l’API web en aval. Dans ce flux, votre API web reçoit de l’application cliente un jeton du porteur avec des permissions déléguées par l’utilisateur, puis échange ce jeton contre un autre jeton d’accès pour appeler l’API web en aval.

Une API web Python doit utiliser un intergiciel pour valider le jeton du porteur reçu du client. L'API web peut ensuite obtenir le jeton d'accès de l'API située en aval à l'aide de la bibliothèque MSAL Python en appelant la méthode [`acquire_token_on_behalf_of`](https://msal-python.readthedocs.io/en/latest/?badge=latest#msal.ConfidentialClientApplication.acquire_token_on_behalf_of). Pour obtenir un exemple d’utilisation de cette API, consultez le [code de test pour microsoft-authentication-library-for-python sur GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.2.0/tests/test_e2e.py#L429-L472). Consultez également la présentation du [problème 53](https://github.com/AzureAD/microsoft-authentication-library-for-python/issues/53) dans ce même dépôt pour une approche qui contourne la nécessité d’une application de niveau intermédiaire.

---

Vous pouvez également voir un exemple d’implémentation de flux OBO dans [NodeJS et Azure Functions](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-onbehalfof-azurefunctions/blob/master/Function/MyHttpTrigger/index.js#L61).

## <a name="protocol"></a>Protocol

Pour plus d’informations sur le protocole OBO, consultez la section [Plateforme d’identités Microsoft et flux On-Behalf-Of OAuth 2.0](./v2-oauth2-on-behalf-of-flow.md).

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Acquérir un jeton pour l’application](scenario-web-api-call-api-acquire-token.md).
