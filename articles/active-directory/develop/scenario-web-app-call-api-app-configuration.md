---
title: Configurer une application web qui appelle des API web | Azure
titleSuffix: Microsoft identity platform
description: Apprenez à configurer le code d'une application web qui appelle des API web
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: e055287f069c477318a54aedf3d9a2fe22343367
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509153"
---
# <a name="a-web-app-that-calls-web-apis-code-configuration"></a>Application web qui appelle des API web : Configuration de code

Comme illustré dans le scénario [Application web qui connecte des utilisateurs](scenario-web-app-sign-user-overview.md), l'application web utilise le flux du code d'autorisation [OAuth 2.0](v2-oauth2-auth-code-flow.md) pour connecter l'utilisateur. Ce flux comporte deux étapes :

1. Demande d’un code d’autorisation. Cette partie délègue à la plateforme d'identités Microsoft un dialogue privé avec l'utilisateur. Pendant ce dialogue, l'utilisateur se connecte et consent à utiliser des API web. Lorsque le dialogue privé se termine correctement, l'application web reçoit un code d'autorisation sur son URI de redirection.
1. Demande d’un jeton d’accès pour l’API en échange du code d’autorisation.

Les scénarios [Application web qui connecte des utilisateurs](scenario-web-app-sign-user-overview.md) ne couvraient que la première étape. Vous allez ici apprendre à modifier votre application web afin qu'elle appelle les API web en plus de connecter les utilisateurs.

## <a name="libraries-that-support-web-app-scenarios"></a>Bibliothèques qui prennent en charge les scénarios relatifs aux applications web

Les bibliothèques suivantes de Microsoft Authentication Library (MSAL) prennent en charge le flux de code d'autorisation pour les applications web :

| Bibliothèque MSAL | Description |
|--------------|-------------|
| ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | Prise en charge des plateformes .NET Framework et .NET Core. Les plateformes UWP (plateforme Windows universelle), Xamarin.iOS et Xamarin.Android ne sont pas prises en charge car elles sont utilisées pour créer des applications clientes publiques. <br/><br/>Pour les applications web ASP.NET Core et les API web, MSAL.NET est encapsulé dans une bibliothèque de niveau supérieur nommée [Microsoft.Identity.Web](https://aka.ms/ms-identity-web). |
| ![MSAL Python](media/sample-v2-code/logo_python.png) <br/> MSAL pour Python | Prise en charge des applications web Python. |
| ![MSAL Java](media/sample-v2-code/logo_java.png) <br/> MSAL pour Java | Prise en charge des applications web Java. |

Sélectionnez l'onglet correspondant à la plateforme qui vous intéresse :

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

## <a name="client-secrets-or-client-certificates"></a>Secrets clients ou certificats clients

Dans la mesure où votre application web appelle désormais une API web en aval, vous devez fournir un secret client ou un certificat client dans le fichier *appsettings.json*. Vous pouvez également ajouter une section spécifiant ce qui suit :

- URL de l’API web en aval ;
- étendues requises pour appeler l’API.

Dans l’exemple suivant, la section `GraphBeta` spécifie ces paramètres.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-app-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
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
    "ClientId": "[Client_id-of-web-app-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
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

*Microsoft.Identity.Web* offre plusieurs façons de décrire les certificats, que ce soit en définissant une configuration ou en écrivant du code. Pour plus d’informations, consultez [Microsoft.Identity.Web - Using certificates](https://github.com/AzureAD/microsoft-identity-web/wiki/Using-certificates) sur GitHub.

## <a name="startupcs"></a>Startup.cs

Votre application web doit acquérir un jeton pour l’API en aval. Vous le spécifiez en ajoutant la ligne `.EnableTokenAcquisitionToCallDownstreamApi()` après `.AddMicrosoftIdentityWebApi(Configuration)`. Cette ligne expose le service `ITokenAcquisition` que vous pouvez utiliser dans vos actions de contrôleur et de page. Toutefois, comme vous le verrez dans les deux options suivantes, il existe un moyen plus simple. Vous devez également choisir une implémentation de cache de jeton, par exemple `.AddInMemoryTokenCaches()`, dans *Startup.cs* :

   ```csharp
   using Microsoft.Identity.Web;

   public class Startup
   {
     // ...
     public void ConfigureServices(IServiceCollection services)
     {
     // ...
     services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
             .AddMicrosoftIdentityWebApp(Configuration, Configuration.GetSection("AzureAd"))
               .EnableTokenAcquisitionToCallDownstreamApi(new string[]{"user.read" })
               .AddInMemoryTokenCaches();
      // ...
     }
     // ...
   }
   ```

Les étendues transmises à `EnableTokenAcquisitionToCallDownstreamApi` sont facultatives et permettent à votre application web de demander les étendues et le consentement de l’utilisateur à ces étendues lors de la connexion. Si vous ne spécifiez pas les étendues, *Microsoft.Identity.Web* permettra une expérience de consentement incrémentielle.

Si vous ne souhaitez pas acquérir le jeton vous-même, *Microsoft.Identity.Web* fournit deux mécanismes pour appeler une API web à partir d’une application web. L’option que vous choisissez varie selon que vous souhaitez appeler Microsoft Graph ou une autre API.

### <a name="option-1-call-microsoft-graph"></a>Option 1 : Appeler Microsoft Graph

Si vous souhaitez appeler Microsoft Graph, *Microsoft.Identity.Web* vous permet d’utiliser directement le `GraphServiceClient` (exposé par le Kit de développement logiciel (SDK) Microsoft Graph) dans vos actions d’API. Pour exposer Microsoft Graph :

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
     services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
             .AddMicrosoftIdentityWebApp(Configuration, Configuration.GetSection("AzureAd"))
               .EnableTokenAcquisitionToCallDownstreamApi(new string[]{"user.read" })
                  .AddMicrosoftGraph(Configuration.GetSection("GraphBeta"))
               .AddInMemoryTokenCaches();
      // ...
     }
     // ...
   }
   ```

### <a name="option-2-call-a-downstream-web-api-other-than-microsoft-graph"></a>Option n°2 : Appeler une API web en aval autre que Microsoft Graph

Pour appeler une API web autre que Microsoft Graph, *Microsoft.Identity.Web* fournit `.AddDownstreamWebApi()`, qui demande des jetons et appelle l’API web en aval.

   ```csharp
   using Microsoft.Identity.Web;

   public class Startup
   {
     // ...
     public void ConfigureServices(IServiceCollection services)
     {
     // ...
     services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
             .AddMicrosoftIdentityWebApp(Configuration, "AzureAd")
               .EnableTokenAcquisitionToCallDownstreamApi(new string[]{"user.read" })
                  .AddDownstreamWebApi("MyApi", Configuration.GetSection("GraphBeta"))
               .AddInMemoryTokenCaches();
      // ...
     }
     // ...
   }
   ```

### <a name="summary"></a>Résumé

Comme avec des API web, vous pouvez choisir diverses implémentations de cache de jeton. Pour plus d’informations, consultez [Microsoft.Identity.Web - Token cache serialization](https://aka.ms/ms-id-web/token-cache-serialization) sur GitHub.

L’illustration suivante montre les différentes possibilités de *Microsoft.Identity.Web* et leur impact sur le fichier *Startup.cs* :

:::image type="content" source="media/scenarios/microsoft-identity-web-startup-cs.svg" alt-text="Diagramme de bloc montrant des options de configuration de service dans le point de départ CS pour appeler une API web et spécifier une implémentation de cache de jeton":::

> [!NOTE]
> Pour bien comprendre les exemples de code présentés ici, vous devez maîtriser les [notions de base d'ASP.NET Core](/aspnet/core/fundamentals), en particulier l'[injection de dépendances](/aspnet/core/fundamentals/dependency-injection) et le modèle [options](/aspnet/core/fundamentals/configuration/options).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Étant donné que la connexion utilisateur est déléguée à l’intergiciel OIDC (OpenID Connect), vous devez interagir avec le processus OIDC. Le mode d'interaction dépend de l'infrastructure que vous utilisez.

Pour ASP.NET, vous vous abonnez aux événements OIDC de l'intergiciel :

- Vous laissez ASP.NET Core demander un code d'autorisation au moyen de l'intergiciel Open ID Connect. ASP.NET ou ASP.NET Core permet à l'utilisateur de se connecter et de donner son consentement.
- Vous vous abonnez à l'application web pour recevoir le code d'autorisation. Cet abonnement s'effectue par le biais d'un délégué C#.
- Une fois le code d'authentification reçu, vous utilisez les bibliothèques MSAL pour l'échanger. Les jetons d'accès et d'actualisation obtenus sont stockés dans le cache associé. Le cache peut être utilisé dans d'autres parties de l'application, comme les contrôleurs, pour acquérir d'autres jetons en mode silencieux.

Les exemples de code présentés dans cet article et le suivant proviennent de l'[exemple d'application web ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect). Vous pouvez vous référer à cet exemple pour obtenir tous les détails d'implémentation.

# <a name="java"></a>[Java](#tab/java)

Les exemples de code présentés dans cet article et le suivant proviennent de l'exemple d'[application web Java qui appelle Microsoft Graph](https://github.com/Azure-Samples/ms-identity-java-webapp). Cet exemple d'application web utilise MSAL pour Java.
L'exemple permet actuellement à MSAL pour Java de produire l'URL du code d'autorisation et gère la navigation vers le point de terminaison d'autorisation de la plateforme d'identités Microsoft. Il est également possible d'utiliser la sécurité Sprint pour connecter l'utilisateur. Vous pouvez vous référer à cet exemple pour obtenir tous les détails d'implémentation.

# <a name="python"></a>[Python](#tab/python)

Les exemples de code présentés dans cet article et le suivant proviennent de l'exemple d'[application web Python qui appelle Microsoft Graph](https://github.com/Azure-Samples/ms-identity-python-webapp). Cet exemple d'application web utilise MSAL.Python.
L'exemple permet actuellement à MSAL.Python de produire l'URL du code d'autorisation et gère la navigation vers le point de terminaison d'autorisation de la plateforme d'identités Microsoft. Vous pouvez vous référer à cet exemple pour obtenir tous les détails d'implémentation.

---

## <a name="code-that-redeems-the-authorization-code"></a>Code qui échange le code d’autorisation

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Microsoft.Identity.Web simplifie votre code en définissant les paramètres corrects d’OpenID Connect, en s’abonnant à l’événement code reçu et en échangeant le code. Aucun code supplémentaire n’est nécessaire pour échanger le code d’autorisation. Pour plus d’informations sur le fonctionnement, consultez [Code source de Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/blob/c29f1a7950b940208440bebf0bcb524a7d6bee22/src/Microsoft.Identity.Web/WebAppExtensions/WebAppCallsWebApiAuthenticationBuilderExtensions.cs#L140).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Le mode de gestion d'ASP.NET est semblable à celui d'ASP.NET Core, à ceci près que la configuration d'OpenID Connect et l'abonnement à l'événement `OnAuthorizationCodeReceived` se produisent dans le fichier [App_Start\Startup.Auth.cs](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/App_Start/Startup.Auth.cs). De même, les concepts sont semblables à ceux d'ASP.NET Core, sauf que dans ASP.NET, vous devez spécifier le `RedirectUri` dans [Web.config#L15](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/master/WebApp/Web.config#L15). Cette configuration est un peu moins robuste que celle d'ASP.NET Core car il est nécessaire de la modifier lors du déploiement de l'application.

Voici le code de Startup.Auth.cs :

```csharp
public partial class Startup
{
  public void ConfigureAuth(IAppBuilder app)
  {
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

    app.UseCookieAuthentication(new CookieAuthenticationOptions());

    // Custom middleware initialization. This is activated when the code obtained from a code_grant is present in the query string (&code=<code>).
    app.UseOAuth2CodeRedeemer(
        new OAuth2CodeRedeemerOptions
        {
            ClientId = AuthenticationConfig.ClientId,
            ClientSecret = AuthenticationConfig.ClientSecret,
            RedirectUri = AuthenticationConfig.RedirectUri
        }
      );

  app.UseOpenIdConnectAuthentication(
      new OpenIdConnectAuthenticationOptions
      {
        // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0.
        Authority = AuthenticationConfig.Authority,
        ClientId = AuthenticationConfig.ClientId,
        RedirectUri = AuthenticationConfig.RedirectUri,
        PostLogoutRedirectUri = AuthenticationConfig.RedirectUri,
        Scope = AuthenticationConfig.BasicSignInScopes + " Mail.Read", // A basic set of permissions for user sign-in and profile access "openid profile offline_access"
        TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = false,
            // In a real application, you would use IssuerValidator for additional checks, such as making sure the user's organization has signed up for your app.
            //     IssuerValidator = (issuer, token, tvp) =>
            //     {
            //        //if(MyCustomTenantValidation(issuer))
            //        return issuer;
            //        //else
            //        //    throw new SecurityTokenInvalidIssuerException("Invalid issuer");
            //    },
            //NameClaimType = "name",
        },
        Notifications = new OpenIdConnectAuthenticationNotifications()
        {
            AuthorizationCodeReceived = OnAuthorizationCodeReceived,
            AuthenticationFailed = OnAuthenticationFailed,
        }
      });
  }

  private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification context)
  {
      // Upon successful sign-in, get the access token and cache it by using MSAL.
      IConfidentialClientApplication clientApp = MsalAppBuilder.BuildConfidentialClientApplication(new ClaimsPrincipal(context.AuthenticationTicket.Identity));
      AuthenticationResult result = await clientApp.AcquireTokenByAuthorizationCode(new[] { "Mail.Read" }, context.Code).ExecuteAsync();
  }

  private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> notification)
  {
      notification.HandleResponse();
      notification.Response.Redirect("/Error?message=" + notification.Exception.Message);
      return Task.FromResult(0);
  }
}
```

# <a name="java"></a>[Java](#tab/java)

Pour en savoir plus sur la façon dont l'exemple Java se procure le code d'autorisation, consultez [Application web qui connecte des utilisateurs : configuration de code](scenario-web-app-sign-user-app-configuration.md?tabs=java#initialization-code). Une fois que l'application a reçu le code, [AuthFilter.java#L51-L56](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java#L51-L56) :

1. Délègue à la méthode `AuthHelper.processAuthenticationCodeRedirect` dans [AuthHelper.java#L67-L97](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthHelper.java#L67-L97).
1. Appelle `getAuthResultByAuthCode`.

```Java
class AuthHelper {
  // Code omitted
  void processAuthenticationCodeRedirect(HttpServletRequest httpRequest, String currentUri, String fullUrl)
            throws Throwable {

  // Code omitted
  AuthenticationResponse authResponse = AuthenticationResponseParser.parse(new URI(fullUrl), params);

  // Code omitted
  IAuthenticationResult result = getAuthResultByAuthCode(
                    httpRequest,
                    oidcResponse.getAuthorizationCode(),
                    currentUri);

// Code omitted
  }
}
```

La méthode `getAuthResultByAuthCode` est définie dans [AuthHelper.java#L176](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthHelper.java#L176). Elle crée un `ConfidentialClientApplication` MSAL, puis appelle `acquireToken()` avec les paramètres `AuthorizationCodeParameters` créés à partir du code d'autorisation.

```Java
   private IAuthenticationResult getAuthResultByAuthCode(
            HttpServletRequest httpServletRequest,
            AuthorizationCode authorizationCode,
            String currentUri) throws Throwable {

        IAuthenticationResult result;
        ConfidentialClientApplication app;
        try {
            app = createClientApplication();

            String authCode = authorizationCode.getValue();
            AuthorizationCodeParameters parameters = AuthorizationCodeParameters.builder(
                    authCode,
                    new URI(currentUri)).
                    build();

            Future<IAuthenticationResult> future = app.acquireToken(parameters);

            result = future.get();
        } catch (ExecutionException e) {
            throw e.getCause();
        }

        if (result == null) {
            throw new ServiceUnavailableException("authentication result was null");
        }

        SessionManagementHelper.storeTokenCacheInSession(httpServletRequest, app.tokenCache().serialize());

        return result;
    }

    private ConfidentialClientApplication createClientApplication() throws MalformedURLException {
        return ConfidentialClientApplication.builder(clientId, ClientCredentialFactory.create(clientSecret)).
                authority(authority).
                build();
    }
```

# <a name="python"></a>[Python](#tab/python)

Le flux de code d'autorisation est demandé comme indiqué dans [Application Web qui connecte des utilisateurs : configuration de code](scenario-web-app-sign-user-app-configuration.md?tabs=python#initialization-code). La fonction `authorized` reçoit alors le code et Flask l'achemine à partir de l'URL `/getAToken`. Pour accéder au contexte complet de ce code, consultez [app.py#L30-L44](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L30-L44) :

```python
 @app.route("/getAToken")  # Its absolute URL must match your app's redirect_uri set in AAD.
def authorized():
    if request.args['state'] != session.get("state"):
        return redirect(url_for("login"))
    cache = _load_cache()
    result = _build_msal_app(cache).acquire_token_by_authorization_code(
        request.args['code'],
        scopes=app_config.SCOPE,  # Misspelled scope would cause an HTTP 400 error here.
        redirect_uri=url_for("authorized", _external=True))
    if "error" in result:
        return "Login failure: %s, %s" % (
            result["error"], result.get("error_description"))
    session["user"] = result.get("id_token_claims")
    _save_cache(cache)
    return redirect(url_for("index"))
```

---

Au lieu d'utiliser un secret client, l'application cliente confidentielle peut également prouver son identité à l'aide d'un certificat client ou d'une assertion de client.
L'utilisation d'assertions de client est un scénario avancé décrit en détail dans [Assertions de client](msal-net-client-assertions.md).

## <a name="token-cache"></a>Cache de jeton

> [!IMPORTANT]
> L'implémentation du cache de jetons des applications ou API web est différente de l'implémentation relative aux applications de bureau, qui est souvent [basée sur des fichiers](scenario-desktop-acquire-token.md#file-based-token-cache).
> En termes de sécurité et de performances, pour les applications et API web, vous devez impérativement disposer d'un cache de jetons par compte d'utilisateur. Vous devez sérialiser le cache de jetons pour chaque compte.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Le tutoriel ASP.NET Core utilise l’injection de dépendances pour vous laisser déterminer l’implémentation du cache de jetons dans le fichier Startup.cs de votre application. Microsoft.Identity.Web est fourni avec des sérialiseurs de cache de jetons prédéfinis décrits dans [Sérialisation du cache de jetons](msal-net-token-cache-serialization.md#token-cache-for-a-web-app-confidential-client-application). Une possibilité intéressante consiste à choisir des [caches en mémoire distribuée](/aspnet/core/performance/caching/distributed#distributed-memory-cache) ASP.NET Core :

```csharp
// Use a distributed token cache by adding:
    services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAd")
            .EnableTokenAcquisitionToCallDownstreamApi(
                initialScopes: new string[] { "user.read" })
            .AddDistributedTokenCaches();

// Then, choose your implementation.
// For instance, the distributed in-memory cache (not cleared when you stop the app):
services.AddDistributedMemoryCache();

// Or a Redis cache:
services.AddStackExchangeRedisCache(options =>
{
 options.Configuration = "localhost";
 options.InstanceName = "SampleInstance";
});

// Or even a SQL Server token cache:
services.AddDistributedSqlServerCache(options =>
{
 options.ConnectionString = _config["DistCache_ConnectionString"];
 options.SchemaName = "dbo";
 options.TableName = "TestCache";
});
```

Pour plus d’informations sur les fournisseurs de cache de jetons, consultez également l’article relatif à la [sérialisation du cache de jetons de Microsoft.Identity.Web](https://aka.ms/ms-id-web/token-cache-serialization), ainsi que la phase [ASP.NET Core Web app tutorials | Token caches](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-2-TokenCache) du didacticiel sur les applications web.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

L'implémentation du cache de jetons des applications ou API web est différente de l'implémentation relative aux applications de bureau, qui est souvent [basée sur des fichiers](scenario-desktop-acquire-token.md#file-based-token-cache).

L'implémentation des applications web peut utiliser la session ASP.NET ou la mémoire du serveur. Observez, par exemple, comment l'implémentation du cache est accrochée après la création de l'application MSAL.NET dans [MsalAppBuilder.cs#L39-L51](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Utils/MsalAppBuilder.cs#L39-L51) :

```csharp
public static class MsalAppBuilder
{
 // Omitted code
    public static IConfidentialClientApplication BuildConfidentialClientApplication(ClaimsPrincipal currentUser)
    {
      IConfidentialClientApplication clientapp = ConfidentialClientApplicationBuilder.Create(AuthenticationConfig.ClientId)
            .WithClientSecret(AuthenticationConfig.ClientSecret)
            .WithRedirectUri(AuthenticationConfig.RedirectUri)
            .WithAuthority(new Uri(AuthenticationConfig.Authority))
            .Build();

      // After the ConfidentialClientApplication is created, we overwrite its default UserTokenCache with our implementation.
      MSALPerUserMemoryTokenCache userTokenCache = new MSALPerUserMemoryTokenCache(clientapp.UserTokenCache, currentUser ?? ClaimsPrincipal.Current);

      return clientapp;
  }
```

# <a name="java"></a>[Java](#tab/java)

MSAL Java fournit des méthodes permettant de sérialiser et de désérialiser le cache de jetons. L'exemple Java gère la sérialisation à partir de la session, comme illustré par la méthode `getAuthResultBySilentFlow` dans [AuthHelper.java#L99-L122](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthHelper.java#L99-L122) :

```Java
IAuthenticationResult getAuthResultBySilentFlow(HttpServletRequest httpRequest, HttpServletResponse httpResponse)
      throws Throwable {

  IAuthenticationResult result =  SessionManagementHelper.getAuthSessionObject(httpRequest);

  IConfidentialClientApplication app = createClientApplication();

  Object tokenCache = httpRequest.getSession().getAttribute("token_cache");
  if (tokenCache != null) {
      app.tokenCache().deserialize(tokenCache.toString());
  }

  SilentParameters parameters = SilentParameters.builder(
          Collections.singleton("User.Read"),
          result.account()).build();

  CompletableFuture<IAuthenticationResult> future = app.acquireTokenSilently(parameters);
  IAuthenticationResult updatedResult = future.get();

  // Update session with latest token cache.
  SessionManagementHelper.storeTokenCacheInSession(httpRequest, app.tokenCache().serialize());

  return updatedResult;
}
```

Les détails de la classe `SessionManagementHelper` sont fournis dans l'[exemple MSAL pour Java](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/SessionManagementHelper.java).

# <a name="python"></a>[Python](#tab/python)

Dans l'exemple Python, un seul cache par compte est garanti en recréant une application cliente confidentielle pour chaque demande, puis en la sérialisant dans le cache de la session Flask :

```python
from flask import Flask, render_template, session, request, redirect, url_for
from flask_session import Session  # https://pythonhosted.org/Flask-Session
import msal
import app_config


app = Flask(__name__)
app.config.from_object(app_config)
Session(app)

# Code omitted here for simplicity

def _load_cache():
    cache = msal.SerializableTokenCache()
    if session.get("token_cache"):
        cache.deserialize(session["token_cache"])
    return cache

def _save_cache(cache):
    if cache.has_state_changed:
        session["token_cache"] = cache.serialize()

def _build_msal_app(cache=None):
    return msal.ConfidentialClientApplication(
        app_config.CLIENT_ID, authority=app_config.AUTHORITY,
        client_credential=app_config.CLIENT_SECRET, token_cache=cache)
```

---

## <a name="next-steps"></a>Étapes suivantes

À ce stade, lorsque l'utilisateur se connecte, un jeton est stocké dans le cache de jetons. Nous allons voir comment il est ensuite utilisé dans d'autres parties de l'application web.

[Supprimer les comptes du cache lors de la déconnexion globale](scenario-web-app-call-api-sign-in.md)
