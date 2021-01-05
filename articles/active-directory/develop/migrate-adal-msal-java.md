---
title: Guide de migration ADAL vers (MSAL4j) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment migrer votre application Java ADAL (bibliothèque d’authentification Active Directory) Azure vers Microsoft Authentication Library (MSAL).
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.tgt_pltfrm: Java
ms.workload: identity
ms.date: 11/04/2019
ms.author: sagonzal
ms.reviewer: nacanuma, twhitney
ms.custom: aaddev, devx-track-java
ms.openlocfilehash: 1cc94519525d5f87a14769b0109e425171265c6b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91258317"
---
# <a name="adal-to-msal-migration-guide-for-java"></a>Guide de migration ADAL vers MSAL pour Java

Cet article souligne les changements que vous devez apporter pour migrer une application qui utilise la bibliothèque d’authentification Active Directory (ADAL) Azure vers une application qui utilise Microsoft Authentication Library (MSAL).

Microsoft Authentication Library pour Java (MSAL4J) et la Bibliothèque d’authentification Azure AD pour Java (ADAL4J) sont tous les deux utilisés pour authentifier des entités Azure AD et demander des jetons auprès d’Azure AD. À ce jour, la plupart des développeurs utilisent la plateforme Azure AD pour développeurs (v1.0) afin d’authentifier les identités Azure AD (comptes professionnels et scolaires) en demandant des jetons avec la Bibliothèque d’authentification Active Directory (ADAL).

MSAL offre les avantages suivants :

- Étant donné qu’il utilise le nouveau point de terminaison de la plateforme d’identité Microsoft, vous pouvez authentifier un ensemble plus large d’identités Microsoft (identités Azure AD, comptes Microsoft, comptes de réseaux sociaux et locaux par le biais d’Azure AD B2C) ;
- vos utilisateurs bénéficieront de la meilleure expérience d’authentification unique ;
- votre application peut activer le consentement incrémentiel, et l’accès conditionnel est plus facile à prendre en charge.

MSAL pour Java est la bibliothèque d’authentification qu’il est recommandé d’utiliser avec la plateforme d’identité Microsoft. Aucune nouvelle fonctionnalité ne sera implémentée sur ADAL4J. Tous les efforts se focalisent sur l’amélioration de MSAL.

## <a name="differences"></a>Différences

Si vous travaillez déjà avec le point de terminaison Azure AD pour développeurs (v1.0) (et ADAL4J), nous vous suggérons de lire l’article sur les [différences du point de terminaison (v2.0) de la plateforme d’identités Microsoft](../azuread-dev/azure-ad-endpoint-comparison.md).

## <a name="scopes-not-resources"></a>Étendues au lieu de ressources

ADAL4J acquiert des jetons pour les ressources, tandis que MSAL pour Java en acquiert pour les étendues. Certaines classes MSAL pour Java nécessitent un paramètre d’étendue. Ce paramètre est une liste de chaînes qui déclarent les autorisations souhaitées et les ressources demandées. Découvrez des exemples d’étendues dans l’article [Étendues de Microsoft Graph](/graph/permissions-reference).

Vous pouvez ajouter le suffixe d’étendue `/.default` à la ressource pour facilité la migration de vos applications du point de terminaison v1.0 (ADAL) vers le point de terminaison de la Plateforme d’identités Microsoft (MSAL). Par exemple, pour la valeur de ressource `https://graph.microsoft.com`, la valeur d’étendue équivalente est `https://graph.microsoft.com/.default`.  Si la ressource ne se trouve pas dans la forme d’URL, mais dans un ID de ressource de la forme `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX`, vous pouvez toujours utiliser la valeur d’étendue `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX/.default`.

Pour plus d’informations sur les différents types d’étendues, consultez [Autorisations et consentement dans le point de terminaison de la Plateforme d’identités Microsoft](./v2-permissions-and-consent.md) et les articles [Étendues pour une API web acceptant des jetons v1.0](./msal-v1-app-scopes.md).

## <a name="core-classes"></a>Classes principales

Dans ADAL4J, la classe `AuthenticationContext` représente votre connexion auprès du service d’émission de jetons de sécurité (STS) ou du serveur d’autorisation, par le biais d’une autorité. En revanche, MSAL pour Java est conçue autour des applications clientes. La bibliothèque fournit deux classes distinctes : `PublicClientApplication` et `ConfidentialClientApplication` pour représenter les applications clientes.  Cette dernière, `ConfidentialClientApplication`, représente une application conçue pour garder en lieu sûr un secret tel que l’identificateur d’une application pour une application démon.

Le tableau suivant montre les correspondances entre les fonctions ADAL4J et les nouvelles fonctions MSAL pour Java :

| Méthode ADAL4J| Méthode MSAL4J|
|------|-------|
|acquireToken(String resource, ClientCredential credential, AuthenticationCallback callback) | acquireToken(ClientCredentialParameters)|
|acquireToken(String resource, ClientAssertion assertion, AuthenticationCallback callback)|acquireToken(ClientCredentialParameters)|
|acquireToken(String resource, AsymmetricKeyCredential credential, AuthenticationCallback callback)|acquireToken(ClientCredentialParameters)|
|acquireToken(String resource, String clientId, String username, String password, AuthenticationCallback callback)| acquireToken(UsernamePasswordParameters)|
|acquireToken(String resource, String clientId, String username, String password=null, AuthenticationCallback callback)|acquireToken(IntegratedWindowsAuthenticationParameters)|
|acquireToken(String resource, UserAssertion userAssertion, ClientCredential credential, AuthenticationCallback callback)| acquireToken(OnBehalfOfParameters)|
|acquireTokenByAuthorizationCode() | acquireToken(AuthorizationCodeParameters) |
| acquireDeviceCode() and acquireTokenByDeviceCode()| acquireToken(DeviceCodeParameters)|
|acquireTokenByRefreshToken()| acquireTokenSilently(SilentParameters)|

## <a name="iaccount-instead-of-iuser"></a>IAccount au lieu de IUser

ADAL4J manipulait des utilisateurs. Bien qu’un utilisateur représente une seule personne ou un agent de logiciel, il peut avoir plusieurs comptes dans le système d’identité Microsoft. Par exemple, un utilisateur peut avoir plusieurs comptes personnels Azure AD, Azure AD B2C ou Microsoft.

MSAL pour Java définit le concept de compte via l’interface `IAccount`. Il s’agit d’un changement cassant par rapport à ADAL4J, mais bénéfique, car il prend en compte le fait qu’un même utilisateur peut avoir différents comptes, notamment dans différents répertoires Azure AD. MSAL pour Java fournit de meilleures informations dans les scénarios d’invité, car des informations de compte d’accueil sont fournies.

## <a name="cache-persistence"></a>Persistance du cache

ADAL4J n’offrait pas de prise en charge pour le cache de jeton.
MSAL pour Java ajoute un [cache de jetons](msal-acquire-cache-tokens.md) pour simplifier la gestion de la durée de vie des jetons en actualisant automatiquement les jetons ayant expiré lorsque cela est possible et en évitant les invites inutiles à entrer des informations d’identification lorsque cela est possible.

## <a name="common-authority"></a>Autorité commune

Dans v1.0, si vous utilisez l’autorité `https://login.microsoftonline.com/common`, les utilisateurs peuvent se connecter avec n’importe quel compte Azure Active Directory (AAD) (pour n’importe quelle organisation).

Si vous utilisez l’autorité `https://login.microsoftonline.com/common` dans v2.0, les utilisateurs peuvent se connecter avec n’importe quelle organisation AAD ou même un compte personnel Microsoft (MSA). Dans MSAL pour Java, si vous voulez limiter la connexion à un compte AAD, vous devez utiliser l’autorité `https://login.microsoftonline.com/organizations` (même comportement qu’avec ADAL4J). Pour spécifier une autorité, définissez le paramètre `authority` dans la méthode [PublicClientApplication.Builder](https://javadoc.io/doc/com.microsoft.azure/msal4j/1.0.0/com/microsoft/aad/msal4j/PublicClientApplication.Builder.html) lorsque vous créez votre classe `PublicClientApplication`.

## <a name="v10-and-v20-tokens"></a>Jetons v1.0 et v2.0

Le point de terminaison v1.0 (utilisé par ADAL) émet uniquement des jetons v1.0.

Le point de terminaison v2.0 (utilisé par MSAL) peut émettre des jetons v1.0 et v2.0. Une propriété du manifeste d'application de l'API web permet aux développeurs de choisir quelle version du jeton est acceptée. Consultez `accessTokenAcceptedVersion` dans la documentation de référence du [manifeste de l’application](./reference-app-manifest.md).

Pour plus d’informations sur les jetons v1.0 et v2.0, consultez [Jetons d’accès Azure Active Directory](./access-tokens.md).

## <a name="adal-to-msal-migration"></a>Migration ADAL vers MSAL

Dans ADAL4J, les jetons d’actualisation étaient exposés, ce qui permettait aux développeurs de les mettre en cache. Ils utilisaient alors `AcquireTokenByRefreshToken()` pour activer des solutions telles que l’implémentation de services durables qui actualisent les tableaux de bord au nom de l’utilisateur lorsque celui-ci n’est plus connecté.

Pour des raisons de sécurité, MSAL pour Java n’expose pas les jetons d’actualisation. C’est MSAL qui gère l’actualisation des jetons pour vous.

MSAL pour Java possède une API qui vous permet de migrer des jetons d’actualisation acquis avec ADAL4J dans l’application cliente : [acquireToken(RefreshTokenParameters)](https://javadoc.io/static/com.microsoft.azure/msal4j/1.0.0/com/microsoft/aad/msal4j/PublicClientApplication.html#acquireToken-com.microsoft.aad.msal4j.RefreshTokenParameters-). Avec cette méthode, vous pouvez fournir le jeton d’actualisation utilisé précédemment, ainsi que toutes les étendues (ressources) souhaitées. Le jeton d’actualisation est échangé avec un nouveau jeton et mis en cache pour être utilisé par votre application.

Cet extrait de code montre du code de migration dans une application cliente confidentielle :

```java
String rt = GetCachedRefreshTokenForSignedInUser(); // Get refresh token from where you have them stored
Set<String> scopes = Collections.singleton("SCOPE_FOR_REFRESH_TOKEN");

RefreshTokenParameters parameters = RefreshTokenParameters.builder(scopes, rt).build();

PublicClientApplication app = PublicClientApplication.builder(CLIENT_ID) // ClientId for your application
                .authority(AUTHORITY)  //plug in your authority
                .build();

IAuthenticationResult result = app.acquireToken(parameters);
```

Le paramètre `IAuthenticationResult` renvoie un jeton d’accès et un jeton d’ID, tandis que votre nouveau jeton d’actualisation est stocké dans le cache.
L’application contient également désormais un IAccount :

```java
Set<IAccount> accounts =  app.getAccounts().join();
```

Pour utiliser les jetons qui se trouvent désormais dans le cache, appelez :

```java
SilentParameters parameters = SilentParameters.builder(scope, accounts.iterator().next()).build();
IAuthenticationResult result = app.acquireToken(parameters);
```
