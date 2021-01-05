---
title: Configurer des applications API web protégées | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment créer une API web protégée et configurer le code de votre application.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: bfbfb1ff5b6cb9c711d987608226c51822dfc935
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442954"
---
# <a name="protected-web-api-code-configuration"></a>API web protégée : Configuration de code

Pour configurer le code de votre API web protégée, vous devez comprendre ce qui suit :

- Ce qui définit les API comme étant protégées.
- Comment configurer un jeton du porteur.
- Comment valider le jeton.

## <a name="what-defines-aspnet-and-aspnet-core-apis-as-protected"></a>Qu’est-ce qui définit les API ASP.NET et ASP.NET Core comme étant protégées ?

Comme les applications web, les API web ASP.NET et ASP.NET Core sont protégées parce que leurs actions de contrôleur sont préfixées par l’attribut **[Authorize]** . Les actions de contrôleur peuvent être appelées uniquement si l’API est appelée avec une identité autorisée.

Considérez les questions suivantes :

- Seule une application peut appeler une API web. Comment l’API connaît-elle l’identité de l’application qui l’appelle ?
- Si l’application appelle l’API au nom d’un utilisateur, quelle est l’identité de celui-ci ?

## <a name="bearer-token"></a>Jeton du porteur

Le jeton du porteur qui est défini dans l’en-tête lorsque l’application est appelée contient des informations sur l’identité de l’application. Il contient également des informations sur l’utilisateur, sauf si l’application web accepte les appels de service à service à partir d’une application démon.

Voici un exemple de code C# qui présente un client appelant l’API après avoir acquis un jeton de Microsoft Authentication Library pour .NET (MSAL.NET) :

```csharp
var scopes = new[] {$"api://.../access_as_user"};
var result = await app.AcquireToken(scopes)
                      .ExecuteAsync();

httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the web API.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
```

> [!IMPORTANT]
> Une application cliente demande le jeton du porteur au point de terminaison de la plateforme d’identités Microsoft *pour l’API web*. L’API web est la seule application qui doive vérifier le jeton et afficher les revendications qu’il contient. Les applications clientes ne doivent jamais tenter d’inspecter les revendications contenues dans les jetons
>
> À l’avenir, l’API web pourrait exiger que le jeton soit chiffré. Cette exigence empêcherait l’accès des applications clientes capables d’afficher les jetons d’accès.

## <a name="jwtbearer-configuration"></a>Configuration de JwtBearer

Cette section décrit comment configurer un jeton du porteur.

### <a name="config-file"></a>Fichier de configuration

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    /*
      You need specify the TenantId only if you want to accept access tokens from a single tenant
     (line-of-business app).
      Otherwise, you can leave them set to common.
      This can be:
      - A GUID (Tenant ID = Directory ID)
      - 'common' (any organization and personal accounts)
      - 'organizations' (any organization)
      - 'consumers' (Microsoft personal accounts)
    */
    "TenantId": "common"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

#### <a name="case-where-you-used-a-custom-app-id-uri-for-your-web-api"></a>Cas où vous avez utilisé un URI ID d’application personnalisé pour votre API web

Si vous avez accepté l’URI ID d’application proposé par le portail d’inscription des applications, vous n’avez pas besoin de spécifier le public (consultez [URI et étendues de l’ID d’application](scenario-protected-web-api-app-registration.md#application-id-uri-and-scopes)). Dans le cas contraire, vous devez ajouter une propriété `Audience` dont la valeur est l’URI ID d’application de votre API web.

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common",
    "Audience": "custom App ID URI for your web API"
  },
  // more lines
}
```

### <a name="code-initialization"></a>Initialisation du code

Quand une application est appelée sur une action de contrôleur qui contient un attribut **[Authorize]**, ASP.NET et ASP.NET Core extraient le jeton d’accès à partir du jeton du porteur dans l’en-tête d’autorisation. Le jeton d’accès est ensuite transmis à l’intergiciel JwtBearer qui appelle Microsoft IdentityModel Extensions pour .NET.

#### <a name="microsoftidentityweb"></a>Microsoft.Identity.Web

Microsoft vous recommande d’utiliser le package NuGet [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web) lors du développement d’une API web avec ASP.NET Core.

_Microsoft.Identity.Web_ fournit le ciment entre ASP.NET Core, l’intergiciel d’authentification et la [bibliothèque d’authentification Microsoft (MSAL)](msal-overview.md) pour .NET. Il favorise une expérience développeur plus claire et plus robuste, et tire parti de la puissance de la plateforme d’identités Microsoft et d’Azure AD B2C.

#### <a name="using-microsoftidentityweb-templates"></a>Utilisation de modèles Microsoft.Identity.Web

Vous pouvez créer une API web à partir de zéro à l’aide des modèles de projet Microsoft.Identity.Web. Pour plus d’informations, consultez [Modèles de projet d’API web Microsoft.Identity.Web](https://aka.ms/ms-id-web/webapi-project-templates).

#### <a name="starting-from-an-existing-aspnet-core-31-application"></a>À partir d’une application ASP.NET Core 3.1 existante

Aujourd’hui, ASP.NET Core 3.1 utilise la bibliothèque Microsoft.AspNetCore.AzureAD.UI. Cet intergiciel est initialisé dans le fichier Startup.cs.

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
```

L’intergiciel (middleware) est ajouté à l’API web par cette instruction :

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
  services.AddAuthentication(AzureADDefaults.JwtBearerAuthenticationScheme)
          .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
}
```

 Actuellement, les modèles ASP.NET Core créent des API web Azure Active Directory (Azure AD) qui connectent des utilisateurs de votre organisation ou d’une autre organisation. Elles ne connectent pas d’utilisateurs à l’aide de comptes personnels. Toutefois, vous pouvez modifier les modèles permettant d’utiliser le point de terminaison de la plateforme d’identités Microsoft en utilisant [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web) pour remplacer le code dans *Startup.cs* :

```csharp
using Microsoft.Identity.Web;
```

```csharp
public void ConfigureServices(IServiceCollection services)
{
 // Adds Microsoft Identity platform (AAD v2.0) support to protect this API
 services.AddMicrosoftIdentityWebApiAuthentication(Configuration, "AzureAd");

 services.AddControllers();
}
```

vous pouvez également écrire ce qui suit (qui est équivalent)

```csharp
public void ConfigureServices(IServiceCollection services)
{
 // Adds Microsoft Identity platform (AAD v2.0) support to protect this API
 services.AddAuthentication(AzureADDefaults.JwtBearerAuthenticationScheme)
             .AddMicrosoftIdentityWebApi(Configuration, "AzureAd");

services.AddControllers();
}
```

> [!NOTE]
> Si vous utilisez Microsoft.Identity.Web et que vous ne définissez pas `Audience` dans *appsettings.json*, les éléments suivants sont utilisés :
> -  `$"{ClientId}"` si vous avez défini la [version acceptée du jeton d’accès](scenario-protected-web-api-app-registration.md#accepted-token-version) sur `2` ou pour des API web Azure AD B2C.
> - `$"api://{ClientId}` dans tous les autres cas (pour les [jetons d’accès](access-tokens.md) v1.0).
> Pour plus d’informations, consultez le [code source](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/Resource/RegisterValidAudience.cs#L70-L83) de Microsoft.Identity.Web.

L’extrait de code précédent est extrait du [didacticiel incrémentiel de l’API web ASP.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/63087e83326e6a332d05fee6e1586b66d840b08f/1.%20Desktop%20app%20calls%20Web%20API/TodoListService/Startup.cs#L23-L28). Les détails relatifs à **AddMicrosoftIdentityWebApiAuthentication** sont disponibles dans [Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/WebApiExtensions/WebApiServiceCollectionExtensions.cs#L27). Cette méthode appelle [AddMicrosoftWebAPI](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/WebApiExtensions/WebApiAuthenticationBuilderExtensions.cs#L58), qui lui-même indique à l’intergiciel comment valider le jeton. Pour plus d’informations, consultez son [code source](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/WebApiExtensions/WebApiAuthenticationBuilderExtensions.cs#L104-L122).

## <a name="token-validation"></a>Validation du jeton

Dans l’extrait de code précédent, l’intergiciel JwtBearer, tout comme le middleware OpenID Connect dans des applications web, valide le jeton en fonction de la valeur de `TokenValidationParameters`. Le jeton est déchiffré si nécessaire, les revendications extraites et la signature vérifiée. L’intergiciel valide ensuite le jeton en vérifiant les points suivants :

- Audience: Le jeton est destiné à l’API web.
- Sub : Il a été émis pour une application autorisée à appeler l’API web.
- Émetteur : Il a été émis par un service d’émission de jeton de sécurité (STS) approuvé.
- Expiration : Sa durée de vie s’inscrit dans la plage valide.
- Signature : Il n’a pas été falsifié.

Il peut également y avoir des validations spéciales. Par exemple, il est possible de valider le fait que les clés de signature, quand elles sont incorporées dans un jeton, sont approuvées et que le jeton n’est pas en cours de réexécution. Enfin, certains protocoles nécessitent des validations spécifiques.

### <a name="validators"></a>Validateurs

Les étapes de validation sont capturées dans des validateurs qui sont fournis par la bibliothèque open source [Microsoft IdentityModel Extensions for .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet). Les validateurs sont définis dans le fichier source de la bibliothèque [Microsoft.IdentityModel.Tokens/Validators.cs](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/master/src/Microsoft.IdentityModel.Tokens/Validators.cs).

Ce tableau décrit les validateurs :

| Validateur | Description |
|---------|---------|
| **ValidateAudience** | S’assure que le jeton est destiné à l’application qui le valide pour vous. |
| **ValidateIssuer** | S’assure que le jeton a été émis par un STS approuvé, ce qui signifie qu’il s’agit d’une personne à qui vous faites confiance. |
| **ValidateIssuerSigningKey** | S’assure que l’application qui valide le jeton approuve la clé utilisée pour signer le jeton Il existe un cas particulier où la clé est incorporée dans le jeton. Cependant, ce cas ne se produit généralement pas. |
| **ValidateLifetime** | S’assure que le jeton est encore ou déjà valide. Le validateur vérifie si la durée de vie du jeton est comprise dans la plage spécifiée par les revendications **notbefore** et **expires**. |
| **ValidateSignature** | S’assure que le jeton n’a pas été falsifié. |
| **ValidateTokenReplay** | S’assure que le jeton n’est pas en cours de réexécution Il existe un cas particulier pour certains protocoles d’utilisation unique. |

#### <a name="customizing-token-validation"></a>Personnalisation de la validation des jetons

Les validateurs sont associés aux propriétés de la classe **TokenValidationParameters**. Les propriétés sont initialisées à partir de la configuration ASP.NET et ASP.NET Core.

Dans la plupart des cas, vous n’avez pas besoin de modifier les paramètres. Les applications qui ne sont pas à locataire unique sont des exceptions. Ces applications web acceptent des utilisateurs de toute organisation ou de comptes Microsoft personnels. Dans ce cas, les émetteurs doivent être validés. Microsoft.Identity.Web s’occupe également de la validation de l’émetteur. Pour plus d’informations, consultez [AadIssuerValidator](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web/Resource/AadIssuerValidator.cs) de Microsoft.Identity.Web.

Dans ASP.NET Core, si vous souhaitez personnaliser les paramètres de validation du jeton, utilisez l’extrait de code suivant dans votre *Startup.cs* :

```c#
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApi(Configuration);
services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
{
  var existingOnTokenValidatedHandler = options.Events.OnTokenValidated;
  options.Events.OnTokenValidated = async context =>
  {
       await existingOnTokenValidatedHandler(context);
      // Your code to add extra configuration that will be executed after the current event implementation.
      options.TokenValidationParameters.ValidIssuers = new[] { /* list of valid issuers */ };
      options.TokenValidationParameters.ValidAudiences = new[] { /* list of valid audiences */};
  }
});
```

Pour le MVC d’ASP.NET, l’exemple de code suivant montre comment effectuer une validation de jeton personnalisée :

https://github.com/azure-samples/active-directory-dotnet-webapi-manual-jwt-validation

## <a name="token-validation-in-azure-functions"></a>Validation des jetons dans Azure Functions

Vous pouvez également valider les jetons d’accès entrants dans Azure Functions. Vous trouverez des exemples de cette validation dans les exemples de code suivants sur GitHub :

- .NET : [Azure-Samples/ms-identity-dotnet-webapi-azurefunctions](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)
- Node.js : [Azure-Samples/ms-identity-nodejs-webapi-azurefunctions](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions)
- Python : [Azure-Samples/ms-identity-python-webapi-azurefunctions)](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions)

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Vérifier les étendues et les rôles d’application dans votre code](scenario-protected-web-api-verification-scope-app-roles.md).
