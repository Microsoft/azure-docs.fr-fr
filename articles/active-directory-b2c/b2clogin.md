---
title: Migrer les applications et les API vers b2clogin.com
titleSuffix: Azure AD B2C
description: Découvrez l’utilisation de b2clogin.com dans vos URL de redirection pour Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/27/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 20df5fc3a4d7c392be62df2b7778854d1e2e1cba
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109060"
---
# <a name="set-redirect-urls-to-b2clogincom-for-azure-active-directory-b2c"></a>Paramétrer les URL de redirection sur b2clogin.com pour Azure Active Directory B2C

Quand vous configurez un fournisseur d’identité pour l’inscription et la connexion à votre application Azure AD B2C (Azure Active Directory B2C) , vous devez spécifier une URL de redirection. Vous ne devez plus référencer *login.microsoftonline.com* dans vos applications et API pour authentifier les utilisateurs avec Azure AD B2C. Utilisez à la place *b2clogin.com* pour toutes les nouvelles applications, et migrez les applications existantes de *login.microsoftonline.com* vers *b2clogin.com*.

## <a name="deprecation-of-loginmicrosoftonlinecom"></a>Dépréciation de login.microsoftonline.com

**Mise à jour d’octobre 2020 :** Nous accordons une période de grâce pour les locataires qui ne sont pas en mesure de respecter la date de dépréciation initialement annoncée du 4 décembre 2020. La mise hors service de login.microsoftonline.com interviendra désormais au plus tôt le **14 janvier 2021**.

**Contexte** : Le 4 décembre 2019, nous avions initialement [annoncé](https://azure.microsoft.com/updates/b2c-deprecate-msol/) la mise hors service de la prise en charge de login.microsoftonline.com dans Azure AD B2C prévue pour le 4 décembre 2020. Cela a donné un (1) an aux locataires existants pour migrer vers b2clogin.com. Les nouveaux locataires créés après le 4 décembre 2019 n’accepteront pas les demandes de login.microsoftonline.com. Toutes les fonctionnalités restent les mêmes sur le point de terminaison b2clogin.com.

La dépréciation de login.microsoftonline.com n’a aucun impact sur les locataires Azure Active Directory. Seuls les locataires Azure Active Directory B2C sont concernés par cette modification.

## <a name="what-endpoints-does-this-apply-to"></a>Points de terminaison concernés
La transition vers b2clogin.com s’applique uniquement aux points de terminaison d’authentification qui utilisent des stratégies Azure AD B2C (flux utilisateur ou stratégies personnalisées) pour authentifier les utilisateurs. Ces points de terminaison ont un paramètre `<policy-name>` qui spécifie la stratégie qu’Azure AD B2C doit utiliser. [En savoir plus sur les stratégies Azure AD B2C](technical-overview.md#identity-experiences-user-flows-or-custom-policies). 

Ces points de terminaison peuvent ressembler à ce qui suit :
- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/<b>\<policy-name\></b>/oauth2/v2.0/authorize</code>

- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/<b>\<policy-name\></b>/oauth2/v2.0/token</code>

Sinon, `<policy-name>` peut être transmis en tant que paramètre de requête :
- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/oauth2/v2.0/authorize?<b>p=\<policy-name\></b></code>
- <code>https://login.microsoft.com/\<tenant-name\>.onmicrosoft.com/oauth2/v2.0/token?<b>p=\<policy-name\></b></code>

> [!IMPORTANT]
> Les points de terminaison qui utilisent le paramètre « policy », ainsi que les [URL de redirection du fournisseur d’identité](#change-identity-provider-redirect-urls), doivent être mis à jour.

Certains clients Azure AD B2C utilisent les capacités partagées des locataires d’entreprise Azure AD, comme le flux d’octroi d’informations d’identification du client OAuth 2.0. Ces fonctionnalités sont accessibles à l’aide des points de terminaison login.microsoftonline.com dAzure AD, *qui ne contiennent pas de paramètre de stratégie*. __Ces points de terminaison ne sont pas concernés__.

## <a name="benefits-of-b2clogincom"></a>Avantages de b2clogin.com

Quand vous utilisez *b2clogin.com* comme URL de redirection :

* L’espace utilisé dans l’en-tête de cookie par les services Microsoft est réduit.
* Vos URL de redirection n’ont plus besoin de contenir de référence à Microsoft.
* Le code côté client JavaScript est pris en charge (actuellement en [préversion](javascript-and-page-layout.md)) dans les pages personnalisées. En raison de restrictions de sécurité, le code JavaScript et les éléments de formulaire HTML sont supprimés des pages personnalisées si vous utilisez *login.microsoftonline.com*.

## <a name="overview-of-required-changes"></a>Vue d’ensemble des modifications nécessaires

Vous pouvez être amené à apporter plusieurs modifications avant de migrer vos applications vers *b2clogin.com* :

* Modifiez l’URL de redirection dans les applications de votre fournisseur d’identité pour faire référence à *b2clogin.com*.
* Mettez à jour vos applications Azure AD B2C pour qu’elles utilisent *b2clogin.com* dans leurs références de flux utilisateur et de point de terminaison de jeton. Cela peut inclure la mise à jour de votre utilisation d’une bibliothèque d’authentification, par exemple la bibliothèque d’authentification Microsoft (MSAL).
* Mettez à jour les **origines autorisées** que vous avez définies dans les paramètres CORS de [personnalisation de l’interface utilisateur](customize-ui-with-html.md).

Un ancien point de terminaison peut ressembler à ceci :
- <b><code>https://login.microsoft.com/</b>\<tenant-name\>.onmicrosoft.com/\<policy-name\>/oauth2/v2.0/authorize</code>

Un point de terminaison mis à jour correspondant se présente comme suit :
- <code><b>https://\<tenant-name\>.b2clogin.com/</b>\<tenant-name\>.onmicrosoft.com/\<policy-name\>/oauth2/v2.0/authorize</code>


## <a name="change-identity-provider-redirect-urls"></a>Modifier les URL de redirection des fournisseurs d’identité

Sur le site web de chaque fournisseur d’identité sur lequel vous avez créé une application, modifiez toutes les URL approuvées de sorte que les redirections s’effectuent vers `your-tenant-name.b2clogin.com` au lieu de *login.microsoftonline.com*.

Vous pouvez utiliser deux formats différents pour vos URL de redirection b2clogin.com. L’avantage du premier est que « Microsoft » ne figure pas dans l’URL, car il utilise l’ID de locataire (GUID) et non le nom de domaine votre locataire :

```
https://{your-tenant-name}.b2clogin.com/{your-tenant-id}/oauth2/authresp
```

Le deuxième utilise le nom de domaine de votre locataire sous la forme `your-tenant-name.onmicrosoft.com`. Par exemple :

```
https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp
```

Pour les deux formats :

* Remplacez `{your-tenant-name}` par le nom de votre locataire Azure AD B2C.
* Supprimez l’élément `/te` s’il figure dans l’URL.

## <a name="update-your-applications-and-apis"></a>Mettre à jour vos applications et API

Le code de vos applications et API Azure AD B2C peut faire référence à `login.microsoftonline.com` à plusieurs endroits. Par exemple, votre code peut contenir des références à des flux utilisateur et à des points de terminaison de jeton. Mettez à jour les éléments suivants pour qu’ils fassent plutôt référence à `your-tenant-name.b2clogin.com` :

* Point de terminaison d’autorisation
* Point de terminaison de jeton
* Émetteur de jeton

Par exemple, le point de terminaison d’autorité de la stratégie d’inscription/connexion de Contoso serait désormais :

```
https://contosob2c.b2clogin.com/00000000-0000-0000-0000-000000000000/B2C_1_signupsignin1
```

Pour plus d’informations sur la migration des applications web basées sur OWIN vers b2clogin.com, consultez la section [Migrer une API web basée sur OWIN vers b2clogin.com](multiple-token-endpoints.md).

Pour plus d’informations sur la migration des API Gestion des API Azure protégées par Azure AD B2C, consultez la section [Migrer vers b2clogin.com](secure-api-management.md#migrate-to-b2clogincom) de la partie [Sécuriser une API Gestion des API Azure avec Azure AD B2C](secure-api-management.md).

## <a name="microsoft-authentication-library-msal"></a>Bibliothèque d’authentification Microsoft (MSAL)

### <a name="msalnet-validateauthority-property"></a>Propriété MSAL.NET ValidateAuthority

Si vous utilisez [MSAL.NET][msal-dotnet] v2 ou une version antérieure, définissez la propriété **ValidateAuthority** sur `false` à l’instanciation du client pour autoriser les redirections vers *b2clogin.com*. La définition de ce paramètre sur `false` n’est pas obligatoire pour MSAL.NET v3 et les versions ultérieures.

```csharp
ConfidentialClientApplication client = new ConfidentialClientApplication(...); // Can also be PublicClientApplication
client.ValidateAuthority = false; // MSAL.NET v2 and earlier **ONLY**
```

### <a name="msal-for-javascript-validateauthority-property"></a>MSAL pour la propriété JavaScript validateAuthority

Si vous utilisez [MSAL pour JavaScript][msal-js] v1.2.2 ou une version antérieure, définissez la propriété **validateAuthority** sur `false`.

```JavaScript
// MSAL.js v1.2.2 and earlier
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: false // Required in MSAL.js v1.2.2 and earlier **ONLY**
  }
);
```

Si vous définissez `validateAuthority: true` dans MSAL.js 1.3.0+ (le comportement par défaut), vous devez également spécifier un émetteur de jeton valide avec `knownAuthorities` :

```JavaScript
// MSAL.js v1.3.0+
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: true, // Supported in MSAL.js v1.3.0+
    knownAuthorities: ['tenant-name.b2clogin.com'] // Required if validateAuthority: true
  }
);
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la migration des applications web basées sur OWIN vers b2clogin.com, consultez la section [Migrer une API web basée sur OWIN vers b2clogin.com](multiple-token-endpoints.md).

Pour plus d’informations sur la migration des API Gestion des API Azure protégées par Azure AD B2C, consultez la section [Migrer vers b2clogin.com](secure-api-management.md#migrate-to-b2clogincom) de la partie [Sécuriser une API Gestion des API Azure avec Azure AD B2C](secure-api-management.md).

<!-- LINKS - External -->
[msal-dotnet]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[msal-dotnet-b2c]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics
[msal-js]: https://github.com/AzureAD/microsoft-authentication-library-for-js
[msal-js-b2c]: ../active-directory/develop/msal-b2c-overview.md
