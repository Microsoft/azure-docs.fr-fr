---
title: Configurer une application web qui connecte des utilisateurs - Plateforme d’identités Microsoft | Azure
description: Apprendre à générer une application web qui connecte des utilisateurs (configuration de code)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: dad7b0563fd1ca0dbf60403bc6172e7616e278b2
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443651"
---
# <a name="web-app-that-signs-in-users-code-configuration"></a>Application web qui connecte les utilisateurs : Configuration de code

Découvrez comment configurer le code de votre application web pour connecter les utilisateurs.

## <a name="libraries-for-protecting-web-apps"></a>Bibliothèques pour la protection des applications web

<!-- This section can be in an include for web app and web APIs -->
Les bibliothèques utilisées pour protéger une application web (et une API web) sont les suivantes :

| Plateforme | Bibliothèque | Description |
|----------|---------|-------------|
| ![.NET](media/sample-v2-code/logo_NET.png) | [Extensions des modèles d’identité pour .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki) | Directement utilisé par ASP.NET et ASP.NET Core, Microsoft Identity Model Extensions pour .NET propose un ensemble de DLL s’exécutant sur .NET Framework et .NET Core. À partir d’une application web ASP.NET ou ASP.NET Core, vous pouvez contrôler la validation de jeton à l’aide de la classe **TokenValidationParameters** (en particulier dans certains scénarios de partenaires). En pratique, la complexité est encapsulée dans la bibliothèque [Microsoft.Identity.Web](https://aka.ms/ms-identity-web) |
| ![Java](media/sample-v2-code/small_logo_java.png) | [MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki) | Prise en charge des applications web Java |
| ![Python](media/sample-v2-code/small_logo_python.png) | [MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki) | Prise en charge des applications web Python |

Sélectionnez l’onglet correspondant à la plateforme qui vous intéresse :

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Les extraits de code indiqués dans cet article et les suivants proviennent du [tutoriel incrémentiel sur l’application web ASP.NET Core, chapitre 1](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-1-MyOrg).

Vous pouvez vous y référer pour obtenir tous les détails d’implémentation.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Les extraits de code indiqués dans cet article et les suivants proviennent de l’[exemple d’application web ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect).

Vous pouvez vous référer à cet exemple pour obtenir tous les détails d’implémentation.

# <a name="java"></a>[Java](#tab/java)

Les extraits de code de cet article et les suivants proviennent de [l’exemple d’application web Java qui appelle Microsoft Graph](https://github.com/Azure-Samples/ms-identity-java-webapp).

Vous pouvez vous référer à cet exemple pour obtenir tous les détails d’implémentation.

# <a name="python"></a>[Python](#tab/python)

Les extraits de code indiqués dans cet article et les suivants proviennent de l’[exemple MSAL.Python qui appelle Microsoft Graph](https://github.com/Azure-Samples/ms-identity-python-webapp).

Vous pouvez vous référer à cet exemple pour obtenir tous les détails d’implémentation.

---

## <a name="configuration-files"></a>Fichiers de configuration

Les applications web qui connectent les utilisateurs à l’aide de la plateforme d’identités Microsoft sont généralement configurées via des fichiers config. Les paramètres que vous devez renseigner sont les suivants :

- L’instance de cloud `Instance` si vous souhaitez que votre application s’exécute dans les clouds nationaux, par exemple
- Le public dans l’ID de locataire (`TenantId`)
- L’ID client `ClientId` pour votre application, tel qu’il a été copié à partir du portail Azure

Parfois, les applications peuvent être paramétrées par `Authority`, qui est la concaténation de `Instance` et de `TenantId`.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Dans ASP.NET Core, ces paramètres se trouvent dans le fichier [appsettings.json](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/bc564d68179c36546770bf4d6264ce72009bc65a/1-WebApp-OIDC/1-1-MyOrg/appsettings.json#L2-L8), section AzureAd.

```Json
{
  "AzureAd": {
    // Azure cloud instance among:
    // - "https://login.microsoftonline.com/" for Azure public cloud
    // - "https://login.microsoftonline.us/" for Azure US government
    // - "https://login.microsoftonline.de/" for Azure AD Germany
    // - "https://login.chinacloudapi.cn/" for Azure AD China operated by 21Vianet
    "Instance": "https://login.microsoftonline.com/",

    // Azure AD audience among:
    // - "TenantId" as a GUID obtained from the Azure portal to sign in users in your organization
    // - "organizations" to sign in users in any work or school account
    // - "common" to sign in users with any work or school account or Microsoft personal account
    // - "consumers" to sign in users with a Microsoft personal account only
    "TenantId": "[Enter the tenantId here]",

    // Client ID (application ID) obtained from the Azure portal
    "ClientId": "[Enter the Client Id]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath": "/signout-oidc"
  }
}
```

Dans ASP.NET Core, il existe un autre fichier ([properties\launchSettings.json](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/bc564d68179c36546770bf4d6264ce72009bc65a/1-WebApp-OIDC/1-1-MyOrg/Properties/launchSettings.json#L6-L7)) qui contient l’URL (`applicationUrl`) et le port TLS/SSL (`sslPort`) de votre application, ainsi que divers profils.

```Json
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:3110/",
      "sslPort": 44321
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "webApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:3110/"
    }
  }
}
```

Dans le portail Azure, les URI de réponse à inscrire dans la page **Authentification** de votre application doivent correspondre à ces URL. Pour les deux fichiers de configuration précédents, il s’agirait de `https://localhost:44321/signin-oidc`. Cela est dû au fait que `applicationUrl` est `http://localhost:3110`, mais `sslPort` est spécifié (44321). `CallbackPath` est `/signin-oidc`, comme défini dans `appsettings.json`.

De la même façon, l’URI de déconnexion devrait être défini sur `https://localhost:44321/signout-oidc`.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Dans ASP.NET, l’application est configurée par le biais du fichier [Web.config](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Web.config#L12-L15), lignes 12 à 15.

```XML
<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, visit
  https://go.microsoft.com/fwlink/?LinkId=301880
  -->
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:ClientId" value="[Enter your client ID, as obtained from the app registration portal]" />
    <add key="ida:ClientSecret" value="[Enter your client secret, as obtained from the app registration portal]" />
    <add key="ida:AADInstance" value="https://login.microsoftonline.com/{0}{1}" />
    <add key="ida:RedirectUri" value="https://localhost:44326/" />
    <add key="vs:EnableBrowserLink" value="false" />
  </appSettings>
```

Dans le portail Azure, les URI de réponse à inscrire dans la page **Authentification** de votre application doivent correspondre à ces URL. Autrement dit, ils doivent être `https://localhost:44326/`.

# <a name="java"></a>[Java](#tab/java)

Dans Java, la configuration se trouve dans le fichier [application.properties](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/resources/application.properties) sous `src/main/resources`.

```Java
aad.clientId=Enter_the_Application_Id_here
aad.authority=https://login.microsoftonline.com/Enter_the_Tenant_Info_Here/
aad.secretKey=Enter_the_Client_Secret_Here
aad.redirectUriSignin=http://localhost:8080/msal4jsample/secure/aad
aad.redirectUriGraph=http://localhost:8080/msal4jsample/graph/me
```

Dans le portail Azure, les URI de réponse à inscrire dans la page **Authentification** de votre application doivent correspondre aux instances de `redirectUri` que l’application définit. Autrement dit, ils doivent être `http://localhost:8080/msal4jsample/secure/aad` et `http://localhost:8080/msal4jsample/graph/me`.

# <a name="python"></a>[Python](#tab/python)

Voici le fichier de configuration Python dans [app_config.py](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/0.1.0/app_config.py) :

```Python
CLIENT_SECRET = "Enter_the_Client_Secret_Here"
AUTHORITY = "https://login.microsoftonline.com/common""
CLIENT_ID = "Enter_the_Application_Id_here"
ENDPOINT = 'https://graph.microsoft.com/v1.0/users'
SCOPE = ["User.ReadBasic.All"]
SESSION_TYPE = "filesystem"  # So the token cache will be stored in a server-side session
```

> [!NOTE]
> Ce guide de démarrage rapide propose de stocker le secret client dans le fichier de configuration pour que ce soit plus simple. Dans votre application de production, vous allez utiliser d’autres moyens pour stocker votre secret, comme un coffre de clés ou une variable d’environnement, comme décrit dans la [documentation de Flask](https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables).
>
> ```python
> CLIENT_SECRET = os.getenv("CLIENT_SECRET")
> if not CLIENT_SECRET:
>     raise ValueError("Need to define CLIENT_SECRET environment variable")
> ```

---

## <a name="initialization-code"></a>Code d’initialisation

Le code d’initialisation est différent selon la plateforme. Pour ASP.NET Core et ASP.NET, la connexion des utilisateurs est déléguée à l’intergiciel (middleware) OpenID Connect. Le modèle ASP.NET ou ASP.NET Core génère des applications web pour le point de terminaison Azure Active Directory (Azure AD) v1.0. Un peu de configuration est nécessaire pour les adapter au point de terminaison de la plateforme des identités Microsoft (v 2.0). Dans le cas de Java, elle est gérée par Spring avec la coopération de l’application.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Dans les applications web ASP.NET Core (et les API web), l’application est protégée, car vous disposez d’un attribut `[Authorize]` sur les contrôleurs ou les actions des contrôleurs. Cet attribut vérifie que l’utilisateur est authentifié. Le code qui initialise l’application se trouve dans le fichier *Startup.cs*.

Pour ajouter l’authentification auprès la plateforme d’identités Microsoft (anciennement Azure AD v2.0), vous devez ajouter le code suivant. Les commentaires dans le code doivent être explicites.

> [!NOTE]
> Si vous souhaitez démarrer directement avec les nouveaux modèles ASP.NET Core pour la plateforme d’identités Microsoft, qui tirent profit de Microsoft.Identity.Web, vous pouvez télécharger un package NuGet en préversion contenant des modèles de projet pour .NET Core 3.1 et .NET 5.0. Une fois l’installation effectuée, vous pouvez instancier directement les applications web ASP.NET Core (MVC ou Blazor). Pour plus d’informations, consultez [Modèles de projet d’application web Microsoft.Identity.Web](https://aka.ms/ms-id-web/webapp-project-templates). Il s’agit de l’approche la plus simple, car elle permet l’exécution de toutes les étapes ci-dessous à votre place.
>
> Si vous préférez démarrer votre projet avec le projet web ASP.NET Core par défaut présent dans Visual Studio ou à l’aide de `dotnet new mvc --auth SingleAuth` ou `dotnet new webapp --auth SingleAuth`, vous verrez du code semblable à celui-ci :
>
>```c#
>  services.AddAuthentication(AzureADDefaults.AuthenticationScheme)
>          .AddAzureAD(options => Configuration.Bind("AzureAd", options));
> ```
>
> Ce code utilise le package NuGet hérité **Microsoft.AspNetCore.Authentication.AzureAD. UI** qui est utilisé pour créer une application Azure AD v1.0. Cet article explique comment créer une application de la Plateforme d’identités Microsoft (Azure AD v2.0) qui remplace ce code.
>

1. Ajoutez les packages NuGet [Microsoft.Identity.web](https://www.nuget.org/packages/Microsoft.Identity.Web) et [Microsoft.Identity.web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI) à votre projet. Supprimez le package Microsoft.AspNetCore.Authentication.AzureAD.UI NuGet, s’il est présent.

2. Mettez à jour le code dans `ConfigureServices` afin qu’il utilise les méthodes `AddMicrosoftIdentityWebAppAuthentication` et `AddMicrosoftIdentityUI`.

   ```c#
   public class Startup
   {
    ...
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
     services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAd");

     services.AddRazorPages().AddMvcOptions(options =>
     {
      var policy = new AuthorizationPolicyBuilder()
                    .RequireAuthenticatedUser()
                    .Build();
      options.Filters.Add(new AuthorizeFilter(policy));
     }).AddMicrosoftIdentityUI();
    ```

3. Dans la méthode `Configure` dans *Startup.cs*, activez l’authentification avec un appel à `app.UseAuthentication();`.

   ```c#
   // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
    // more code here
    app.UseAuthentication();
    app.UseAuthorization();
    // more code here
   }
   ```

Dans le code ci-dessus :
- La méthode d’extension `AddMicrosoftIdentityWebAppAuthentication` est définie dans **Microsoft.Identity.web**. Elle effectue les actions suivantes :
  - Ajoute le service d’authentification.
  - Configure les options pour lire le fichier de configuration (ici à partir de la section « AzureAD »).
  - Configure les options d’OpenID Connect afin que l’autorité soit le point de terminaison de la Plateforme d’identités Microsoft.
  - Valide l’émetteur du jeton.
  - Garantit que les revendications correspondant au nom sont mappées à partir de la revendication `preferred_username` dans le jeton d’ID.

- En plus de l’objet de configuration, vous pouvez spécifier le nom de la section de configuration lors de l’appel de `AddMicrosoftIdentityWebAppAuthentication`. Par défaut, il s’agit de `AzureAd`.

- `AddMicrosoftIdentityWebAppAuthentication` a d’autres paramètres pour des scénarios avancés. Par exemple, le suivi des événements de l’intergiciel OpenId Connect peut vous aider à dépanner votre application web si l’authentification ne fonctionne pas. La définition du paramètre facultatif `subscribeToOpenIdConnectMiddlewareDiagnosticsEvents` sur `true` vous montrera comment les informations sont traitées par le jeu d’intergiciels d’ASP.NET Core lors de leur progression de la réponse HTTP à l’identité de l’utilisateur dans `HttpContext.User`.

- La méthode d’extension `AddMicrosoftIdentityUI` est définie dans **Microsoft.Identity.web.UI**. Il fournit un contrôleur par défaut pour gérer la connexion et la déconnexion.

Vous trouverez plus de détails sur la façon dont Microsoft.Identity.Web vous permet de créer des applications web dans <https://aka.ms/ms-id-web/webapp>

> [!WARNING]
> Microsoft.Identity.Web ne prend pas en charge le scénario des **comptes d’utilisateurs individuels** (stockage de comptes d’utilisateurs dans l’application) quand Azure AD est utilisé en tant que fournisseur de connexion externe. Pour plus d’informations, consultez : [AzureAD/microsoft-identity-web#133](https://github.com/AzureAD/microsoft-identity-web/issues/133)

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Le code lié à l’authentification dans l’application web ASP.NET et les API web se trouve dans le fichier [App_Start/Startup.Auth.cs](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/App_Start/Startup.Auth.cs#L17-L61).

```csharp
 public void ConfigureAuth(IAppBuilder app)
 {
  app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

  app.UseCookieAuthentication(new CookieAuthenticationOptions());

  app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
     // `Authority` represents the identity platform endpoint - https://login.microsoftonline.com/common/v2.0.
     // `Scope` describes the initial permissions that your app will need.
     //  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/.
     ClientId = clientId,
     Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0"),
     RedirectUri = redirectUri,
     Scope = "openid profile",
     PostLogoutRedirectUri = redirectUri,
    });
 }
```

# <a name="java"></a>[Java](#tab/java)

L’exemple Java utilise le framework Spring. L’application est protégée parce que vous implémentez un filtre, qui intercepte chaque réponse HTTP. Dans le guide de démarrage rapide pour les applications web Java, ce filtre est `AuthFilter` dans `src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java`.

Le filtre traite le flux code d’autorisation OAuth 2.0 et vérifie si l’utilisateur est authentifié (méthode `isAuthenticated()`). Si l’utilisateur n’est pas authentifié, il calcule l’URL des points de terminaison d’autorisation Azure AD et redirige le navigateur vers cet URI.

Lorsque la réponse arrive, contenant le code d’autorisation, il acquiert le jeton à l’aide de MSAL Java. Lorsqu’il reçoit enfin le jeton du point de terminaison de jeton (sur l’URI de redirection), l’utilisateur est connecté.

Pour plus d’informations, consultez la méthode `doFilter()` dans [AuthFilter.java](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/master/msal-java-webapp-sample/src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java).

> [!NOTE]
> Le code de `doFilter()` est écrit dans un ordre légèrement différent, mais le flux est celui décrit.

Pour plus d’informations sur le flux du code d’autorisation déclenché par cette méthode, consultez [Plateforme d’identités Microsoft et flux du code d’autorisation OAuth 2.0](v2-oauth2-auth-code-flow.md).

# <a name="python"></a>[Python](#tab/python)

L’exemple Python utilise Flask. L’initialisation de Flask et de MSAL.Python s’effectue dans [app.py#L1-L28](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L1-L28).

```Python
import uuid
import requests
from flask import Flask, render_template, session, request, redirect, url_for
from flask_session import Session  # https://pythonhosted.org/Flask-Session
import msal
import app_config


app = Flask(__name__)
app.config.from_object(app_config)
Session(app)
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans l’article suivant, vous apprenez à déclencher la connexion et la déconnexion.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Passez à l’article suivant de ce scénario, [Connexion et déconnexion](./scenario-web-app-sign-user-sign-in.md?tabs=aspnetcore).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Passez à l’article suivant de ce scénario, [Connexion et déconnexion](./scenario-web-app-sign-user-sign-in.md?tabs=aspnet).

# <a name="java"></a>[Java](#tab/java)

Passez à l’article suivant de ce scénario, [Connexion et déconnexion](./scenario-web-app-sign-user-sign-in.md?tabs=java).

# <a name="python"></a>[Python](#tab/python)

Passez à l’article suivant de ce scénario, [Connexion et déconnexion](./scenario-web-app-sign-user-sign-in.md?tabs=python).

---
