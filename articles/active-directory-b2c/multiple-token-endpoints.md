---
title: Migrer des API web OWIN vers b2clogin.com
titleSuffix: Azure AD B2C
description: Découvrez comment permettre à une API web .NET de prendre en charge les jetons émis par plusieurs émetteurs de jetons pendant que vous migrez vos applications vers b2clogin.com.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/31/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: c362ce256259606c85af0a7e13ccde1715bb012b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953931"
---
# <a name="migrate-an-owin-based-web-api-to-b2clogincom"></a>Migrer une API web OWIN vers b2clogin.com

Cet article décrit une technique permettant d’activer la prise en charge de plusieurs émetteurs de jetons dans des API web qui implémentent l’[interface OWIN (Open Web interface for .NET)](http://owin.org/). La prise en charge de plusieurs points de terminaison de jeton s’avère utile quand vous migrez des API Azure AD B2C (Azure Active Directory B2C) et leurs applications de *login.microsoftonline.com* vers *b2clogin.com*.

En ajoutant la prise en charge dans votre API en vue d’accepter les jetons émis par b2clogin.com et login.microsoftonline.com, vous pouvez migrer vos applications web de manière échelonnée avant de supprimer la prise en charge des jetons émis par login.microsoftonline.com à partir de l’API.

Les sections suivantes présentent un exemple d’activation de plusieurs émetteurs dans une API web qui utilise les composants intergiciels (middleware) [Microsoft OWIN][katana] (Katana). Bien que les exemples de code soient spécifiques à l’intergiciel Microsoft OWIN, la technique générale doit s’appliquer aux autres bibliothèques OWIN.

> [!NOTE]
> Cet article s’adresse aux clients Azure AD B2C ayant actuellement des API et des applications déployées qui font référence à `login.microsoftonline.com` et qui souhaitent migrer vers le point de terminaison `b2clogin.com` recommandé. Si vous configurez une nouvelle application, utilisez [b2clogin.com](b2clogin.md) comme indiqué.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de poursuivre les étapes de cet article, vous devez disposer des ressources Azure AD B2C suivantes :

* [Flux d’utilisateurs](tutorial-create-user-flows.md) ou [stratégies personnalisées](custom-policy-get-started.md) créées dans votre locataire

## <a name="get-token-issuer-endpoints"></a>Obtenir les points de terminaison d’émetteur de jeton

Vous devez d’abord obtenir les URI de point de terminaison d’émetteur de jeton pour chaque émetteur que vous souhaitez prendre en charge dans votre API. Pour accéder aux points de terminaison *b2clogin.com* et *login.microsoftonline.com* pris en charge par votre locataire Azure AD B2C, utilisez la procédure suivante dans le Portail Microsoft Azure.

Commencez par sélectionner l’un de vos flux d’utilisateurs existants :

1. Accédez à votre locataire Azure AD B2C dans le [Portail Microsoft Azure](https://portal.azure.com)
1. Sous **Stratégies**, sélectionnez **Flux utilisateur (stratégies)**
1. Sélectionnez une stratégie existante, par exemple *B2C_1_signupsignin1*, puis sélectionnez **Exécuter le flux d’utilisateur**
1. Sous l’en-tête **Exécuter le flux d’utilisateur** en haut de la page, sélectionnez le lien hypertexte pour accéder au point de terminaison de découverte OpenID Connect pour ce flux d’utilisateur.

    ![Lien hypertexte URI bien connu dans la page Exécuter maintenant du Portail Azure](media/multi-token-endpoints/portal-01-policy-link.png)

1. Dans la page qui s’ouvre dans votre navigateur, enregistrez la valeur `issuer`, par exemple :

    `https://your-b2c-tenant.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/`

1. Utilisez la liste déroulante **Sélectionner un domaine** pour sélectionner l’autre domaine, puis effectuez à nouveau les deux étapes précédentes et enregistrez sa valeur `issuer`.

Vous devriez maintenant avoir deux URI enregistrés qui ressemblent à ceci :

```
https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/
https://your-b2c-tenant.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/
```

### <a name="custom-policies"></a>Stratégies personnalisées

Si vous avez des stratégies personnalisées au lieu de flux d’utilisateurs, vous pouvez utiliser un processus similaire pour obtenir les URI d’émetteur.

1. Accédez à votre locataire Azure AD B2C
1. Sélectionnez **Infrastructure d’expérience d’identité**
1. Sélectionnez l’une de vos stratégies de partie de confiance, par exemple, *B2C_1A_signup_signin*
1. Utilisez la liste déroulante **Sélectionner un domaine** pour sélectionner un domaine, par exemple *yourtenant.b2clogin.com*
1. Sélectionnez le lien hypertexte affiché sous **Point de terminaison de détection OpenID Connect**
1. Enregistrez la valeur `issuer`
1. Effectuez les étapes 4 à 6 pour l’autre domaine, par exemple *login.microsoftonline.com*

## <a name="get-the-sample-code"></a>Obtention de l'exemple de code

Maintenant que vous avez les deux URI de point de terminaison de jeton, vous devez mettre à jour votre code pour spécifier que les deux points de terminaison sont des émetteurs valides. Pour explorer un exemple, téléchargez ou clonez l’exemple d’application, puis mettez à jour l’exemple pour qu’il prenne en charge les deux points de terminaison en tant qu’émetteurs valides.

Téléchargez l’archive : [active-directory-b2c-dotnet-webapp-and-webapi-master.zip][sample-archive]

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

## <a name="enable-multiple-issuers-in-web-api"></a>Activer plusieurs émetteurs dans l’API Web

Dans cette section, vous allez mettre à jour le code pour spécifier que les deux points de terminaison de l’émetteur du jeton sont valides.

1. Ouvrez la solution **B2C-WebAPI-DotNet.sln** dans Visual Studio
1. Dans le projet **TaskService**, ouvrez le fichier *TaskService\\App_Start\\**Startup.Auth.cs** _ dans votre éditeur.
1. Ajoutez la directive `using` suivante en haut du fichier :

    `using System.Collections.Generic;`
1. Ajoutez la propriété [`ValidIssuers`][validissuers] à la définition [`TokenValidationParameters`][tokenvalidationparameters] et spécifiez les deux URI que vous avez enregistrés dans la section précédente :

    ```csharp
    TokenValidationParameters tvps = new TokenValidationParameters
    {
        // Accept only those tokens where the audience of the token is equal to the client ID of this app
        ValidAudience = ClientId,
        AuthenticationType = Startup.DefaultPolicy,
        ValidIssuers = new List<string> {
            "https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/",
            "https://{your-b2c-tenant}.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/"
        }
    };
    ```

`TokenValidationParameters` est fourni par MSAL.net et est utilisé par l’intergiciel OWIN dans la section de code suivante dans _Startup.auth.cs*. Lorsque plusieurs émetteurs valides sont spécifiés, le pipeline d’application OWIN est conscient que les deux points de terminaison de jeton sont des émetteurs valides.

```csharp
app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions
{
    // This SecurityTokenProvider fetches the Azure AD B2C metadata &  from the OpenID Connect metadata endpoint
    AccessTokenFormat = new JwtFormat(tvps, new tCachingSecurityTokenProvider(String.Format(AadInstance, ultPolicy)))
});
```

Comme mentionné précédemment, d’autres bibliothèques OWIN fournissent généralement une fonctionnalité similaire pour la prise en charge de plusieurs émetteurs. Bien que les exemples fournis pour chaque bibliothèque n’entrent pas dans le cadre de cet article, vous pouvez utiliser une technique similaire pour la plupart des bibliothèques.

## <a name="switch-endpoints-in-web-app"></a>Basculer les points de terminaison dans l’application Web

Maintenant que les deux URI sont pris en charge par votre API Web, vous devez mettre à jour votre application Web afin qu’elle récupère les jetons à partir du point de terminaison b2clogin.com.

Par exemple, vous pouvez configurer l’exemple d’application Web pour utiliser le nouveau point de terminaison en modifiant la valeur `ida:AadInstance` dans le fichier *TaskWebApp\\**Web.config** _ du projet _* TaskWebApp**.

Modifiez la valeur `ida:AadInstance` dans le fichier *Web.config* de TaskWebApp afin qu’elle fasse référence à `{your-b2c-tenant-name}.b2clogin.com` à la place de `login.microsoftonline.com`.

Avant :

```xml
<!-- Old value -->
<add key="ida:AadInstance" value="https://login.microsoftonline.com/tfp/{0}/{1}" />
```

Après (remplacez `{your-b2c-tenant}` par le nom de votre locataire B2C) :

```xml
<!-- New value -->
<add key="ida:AadInstance" value="https://{your-b2c-tenant}.b2clogin.com/tfp/{0}/{1}" />
```

Lorsque les chaînes de point de terminaison sont construites lors de l’exécution de l’application web, les points de terminaison basés sur b2clogin.com sont utilisés lors de la demande de jetons.

## <a name="next-steps"></a>Étapes suivantes

Cet article a présenté une méthode de configuration d’une API web qui implémente l’intergiciel Microsoft OWIN (Katana) afin d’accepter des jetons de plusieurs points de terminaison d’émetteur. Comme vous pouvez le constater, il existe plusieurs autres chaînes dans les fichiers *Web.Config* des projets TaskService et TaskWebApp qui doivent être modifiés si vous souhaitez générer et exécuter ces projets sur votre propre locataire. Vous êtes invité à modifier les projets de manière appropriée si vous souhaitez les voir en action. Toutefois, une procédure pas à pas complète n’entre pas dans le cadre de cet article.

Pour plus d’informations sur les différents types de jetons de sécurité émis par Azure AD B2C, consultez [Vue d’ensemble des jetons dans Azure Active Directory B2C](tokens-overview.md).

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip
[sample-repo]: https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi

<!-- LINKS - Internal -->
[katana]: /aspnet/aspnet/overview/owin-and-katana/
[validissuers]: /dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters.validissuers
[tokenvalidationparameters]: /dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters