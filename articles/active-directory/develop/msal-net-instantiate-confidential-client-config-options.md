---
title: Instancier une application cliente confidentielle (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment instancier une application cliente confidentielle avec des options de configuration à l’aide de Microsoft Authentication Library pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 140385a26dbfd754438d632ce327c3c4a3ea1b61
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88166176"
---
# <a name="instantiate-a-confidential-client-application-with-configuration-options-using-msalnet"></a>Instancier une application cliente confidentielle avec des options de configuration à l’aide de MSAL.NET

Cet article décrit comment instancier une [application cliente confidentielle](msal-client-applications.md) à l’aide de Microsoft Authentication Library pour .NET (MSAL.NET).  L’application est instanciée avec les options de configuration définies dans un fichier de paramètres.

Avant d’initialiser une application, vous devez d’abord [l’inscrire](quickstart-register-app.md) afin que votre application puisse être intégrée à la plateforme d’identités Microsoft. Après l’inscription, vous aurez peut-être besoin des informations suivantes (que vous trouverez sur le Portail Azure) :

- L’ID client (une chaîne représentant un GUID)
- L’URL du fournisseur d’identité (l’instance) et l’audience de connexion pour votre application. Ces deux paramètres représentent collectivement l’autorité.
- L’ID locataire si vous écrivez une application métier uniquement pour votre organisation (également nommée application à locataire unique).
- Le secret d’application (chaîne secrète client) ou le certificat (de type X509Certificate2) s’il s’agit d’une application cliente confidentielle.
- Pour les applications web et parfois pour les applications clientes publiques (en particulier lorsque votre application doit utiliser un répartiteur), vous devez également avoir défini l’URI de redirection auquel le fournisseur d’identité recontactera votre application avec les jetons de sécurité.

## <a name="configure-the-application-from-the-config-file"></a>Configurer l’application à partir du fichier config
Le nom des propriétés des options dans MSAL.NET correspond au nom des propriétés des `AzureADOptions` dans ASP.NET Core. Par conséquent, vous n’avez pas besoin d’écrire de code de collage.

Une configuration d’application ASP.NET Core est décrite dans un fichier *appsettings.json* :

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "[Enter the domain of your tenant, e.g. contoso.onmicrosoft.com]",
    "TenantId": "[Enter 'common', or 'organizations' or the Tenant Id (Obtained from the Azure portal. Select 'Endpoints' from the 'App registrations' blade and use the GUID in any of the URLs), e.g. da41245a5-11b3-996c-00a8-4d99re19f292]",
    "ClientId": "[Enter the Client Id (Application ID obtained from the Azure portal), e.g. ba74781c2-53c2-442a-97c2-3d60re42f403]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc",

    "ClientSecret": "[Copy the client secret added to the app from the Azure portal]"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

À partir de MSAL.NET v3.x, vous pouvez configurer votre application cliente confidentielle à partir du fichier config.

Dans la classe où vous souhaitez configurer et instancier votre application, vous devez déclarer un objet `ConfidentialClientApplicationOptions`.  Liez la configuration lue dans la source (y compris le fichier appconfig.json) à l’instance des options d’application. Pour cela, utilisez la méthode `IConfigurationRoot.Bind()` à partir du [package NuGet Microsoft.Extensions.Configuration.Binder](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder) :

```csharp
using Microsoft.Identity.Client;

private ConfidentialClientApplicationOptions _applicationOptions;
_applicationOptions = new ConfidentialClientApplicationOptions();
configuration.Bind("AzureAD", _applicationOptions);
```

Ceci permet de lier le contenu de la section « AzureAD » du fichier *appsettings.json* aux propriétés correspondantes de l’objet `ConfidentialClientApplicationOptions`.  Ensuite, générez un objet `ConfidentialClientApplication` :

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
        .Build();
```

## <a name="add-runtime-configuration"></a>Ajouter la configuration du runtime
Dans une application cliente confidentielle, vous disposez généralement d’un cache par utilisateur. Par conséquent, vous devrez obtenir le cache associé à l’utilisateur et informer le générateur d’applications que vous souhaitez l’utiliser. Dans la même façon, vous pouvez avoir un URI de redirection calculé de façon dynamique. Dans ce cas, utilisez le code suivant :

```csharp
IConfidentialClientApplication app;
var request = httpContext.Request;
var currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, _azureAdOptions.CallbackPath ?? string.Empty);
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
       .WithRedirectUri(currentUri)
       .Build();
TokenCache userTokenCache = _tokenCacheProvider.SerializeCache(app.UserTokenCache,httpContext, claimsPrincipal);
```

