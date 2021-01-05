---
title: Flux d’authentification MSAL | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les flux d’authentification et les octrois utilisés par Microsoft Authentication Library (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/08/2020
ms.author: marsma
ms.reviewer: saeeda
ms.openlocfilehash: 4a902ed53e92cd073d81626e80bdb3c8629ad072
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89437868"
---
# <a name="authentication-flows"></a>Flux d’authentification

Microsoft Authentication Library (MSAL) prend en charge plusieurs flux d’authentification à utiliser dans différents scénarios d’application.

| Flux | Description | Utilisé dans |
|--|--|--|
| [Code d’autorisation](#authorization-code) | Peut servir dans les applications qui sont installées sur un périphérique pour accéder à des ressources protégées, comme des API web. Vous permet d’ajouter un accès de connexion et d’API à vos applications mobiles et de bureau. | [Applications de bureau](scenario-desktop-overview.md), [applications mobiles](scenario-mobile-overview.md), [applications web](scenario-web-app-call-api-overview.md) |
| [Informations d’identification du client](#client-credentials) | Vous permet d’accéder aux ressources hébergées sur le web à l’aide de l’identité d’une application. Couramment utilisé pour les interactions de serveur à serveur qui doivent s’exécuter en arrière-plan sans l’interaction immédiate d’un utilisateur. | [Applications démon](scenario-daemon-overview.md) |
| [Code d’appareil](#device-code) | Permet aux utilisateurs de se connecter à des appareils à entrée limitée comme une télévision connectée, un appareil IoT ou une imprimante. | [Applications de bureau/mobiles](scenario-desktop-acquire-token.md#command-line-tool-without-a-web-browser) |
| [Octroi implicite](#implicit-grant) | Permet à l’application d’obtenir des jetons sans échange d’informations d’identification avec le serveur principal. L’application peut alors connecter l’utilisateur, maintenir la session et obtenir des jetons pour d’autres API web, le tout dans le code JavaScript client. | [Applications monopages (SPA)](scenario-spa-overview.md) |
| [On-behalf-of](#on-behalf-of) | Une application appelle un service ou une API web, qui à son tour doit appeler un autre service ou une autre API web. L’idée est de propager l’identité et les autorisations de l’utilisateur délégué via la chaîne de la demande. | [API web](scenario-web-api-call-api-overview.md) |
| [Nom d’utilisateur/mot de passe](#usernamepassword) | Permet à une application de connecter l’utilisateur en gérant directement son mot de passe. Ce flux n’est pas recommandé. | [Applications de bureau/mobiles](scenario-desktop-acquire-token.md#username-and-password) |
| [Authentification Windows intégrée](#integrated-windows-authentication) | Permet aux applications sur des ordinateurs joints à un domaine ou à Azure Active Directory (Azure AD) d’obtenir un jeton silencieusement (sans interaction de l’utilisateur avec l’interface utilisateur). | [Applications de bureau/mobiles](scenario-desktop-acquire-token.md#integrated-windows-authentication) |

## <a name="how-each-flow-emits-tokens-and-codes"></a>Comment chaque flux émet des jetons et des codes

Selon la façon dont votre application cliente est créée, elle peut utiliser un ou plusieurs des flux d’authentification pris en charge par la plateforme d’identité Microsoft. Ces flux peuvent produire divers types de jetons ainsi que des codes d’autorisation, et ils nécessitent des jetons différents pour les faire fonctionner.

| Flux                                                                               | Nécessite            | id_token | access token | jeton d'actualisation | code d’autorisation |
|------------------------------------------------------------------------------------|:-------------------:|:--------:|:------------:|:-------------:|:------------------:|
| [Flux du code d’autorisation](v2-oauth2-auth-code-flow.md)                             |                     | x        | x            | x             | x                  |
| [Informations d’identification du client](v2-oauth2-client-creds-grant-flow.md)                         |                     |          | x (application uniquement) |               |                    |
| [Flux de code d’appareil](v2-oauth2-device-code.md)                                       |                     | x        | x            | x             |                    |
| [Flux implicite](v2-oauth2-implicit-grant-flow.md)                                  |                     | x        | x            |               |                    |
| [Flux On-Behalf-Of](v2-oauth2-on-behalf-of-flow.md)                                | access token        | x        | x            | x             |                    |
| [Nom d'utilisateur/mot de passe](v2-oauth-ropc.md) (ROPC)                                       | nom d’utilisateur et mot de passe | x        | x            | x             |                    |
| [Circuit OIDC hybride](v2-protocols-oidc.md#protocol-diagram-access-token-acquisition) |                     | x        |              |               | x                  |
| [Échange de jetons d’actualisation](v2-oauth2-auth-code-flow.md#refresh-the-access-token)   | jeton d'actualisation       | x        | x            | x             |                    |

### <a name="interactive-and-non-interactive-authentication"></a>Authentification interactive et non interactive

Plusieurs de ces flux prennent en charge l’acquisition interactive et non interactive de jetons.

  - **Interactive** signifie que l’utilisateur peut être invité à entrer des données. Par exemple, demander à l’utilisateur de se connecter, d’effectuer l’authentification multifacteur (MFA) ou de donner des consentements supplémentaires à des ressources.
  - L’authentification **non interactive** ou *silencieuse* tente d’acquérir un jeton de manière à ce que le serveur de connexion *ne puisse pas* inviter l’utilisateur à fournir des informations supplémentaires.

Votre application basée sur MSAL doit d’abord tenter d’acquérir un jeton *en mode silencieux*, puis de manière interactive uniquement en cas d’échec de la méthode non interactive. Pour plus d’informations sur ce modèle, consultez [Acquérir et mettre en cache des jetons à l’aide de Microsoft Authentication Library (MSAL)](msal-acquire-cache-tokens.md).

## <a name="authorization-code"></a>Code d’autorisation.

L'[octroi d'un code d'autorisation OAuth 2](v2-oauth2-auth-code-flow.md) peut servir dans les applications qui sont installées sur un périphérique pour accéder à des ressources protégées, comme des API Web. Cela vous permet d’ajouter un accès de connexion et d’API à vos applications mobiles et de bureau.

Lorsque les utilisateurs se connectent aux applications web (sites web), l’application web reçoit un code d’autorisation. Ce code d’autorisation est utilisé pour acquérir un jeton servant à appeler les API web.

![Diagramme de flux de code d’autorisation](media/msal-authentication-flows/authorization-code.png)

Dans le diagramme ci-dessus, l’application :

1. Demande un code d’autorisation, qui est échangé contre un jeton d’accès.
2. Utilise le jeton d’accès pour appeler une API web.

### <a name="considerations"></a>Considérations

- Vous ne pouvez utiliser le code d’autorisation qu’une seule fois pour récupérer un jeton. N’essayez pas d’obtenir un jeton plusieurs fois avec le même code d’autorisation car cela est explicitement interdit par la spécification du protocole standard. Si vous échangez le code plusieurs fois, intentionnellement ou car vous ne savez pas qu’une infrastructure le fait également pour vous, vous obtiendrez l’erreur suivante :

    `AADSTS70002: Error validating credentials. AADSTS54005: OAuth2 Authorization code was already redeemed, please retry with a new valid code or use an existing refresh token.`

## <a name="client-credentials"></a>Informations d'identification du client

Le [flux d’informations d’identification du client OAuth 2](v2-oauth2-client-creds-grant-flow.md) vous permet d’accéder aux ressources hébergées sur le web à l’aide de l’identité d’une application. Ce type d’octroi est couramment utilisé pour les interactions de serveur à serveur qui doivent s’exécuter en arrière-plan sans l’interaction immédiate d’un utilisateur. Ces types d’application sont souvent appelés démons (daemons) ou comptes de service.

Le flux d’octroi des informations d’identification du client permet à un service web (un client confidentiel) d’utiliser ses propres informations d’identification pour s’authentifier lorsqu’il appelle un autre service web, au lieu d’emprunter l’identité d’un utilisateur. Dans ce scénario, le client est généralement un service web de niveau intermédiaire, un service démon ou un site web. Pour augmenter le niveau d’assurance, la plateforme d’identités Microsoft autorise également le service d’appel à utiliser un certificat (au lieu d’un secret partagé) comme une information d’identification.

> [!NOTE]
> Le flux client confidentiel n’est pas disponible sur les plateformes mobiles comme UWP, Xamarin.iOS et Xamarin.Android, car elles prennent uniquement en charge les applications clientes publiques. Les applications clientes publiques ne savent pas comment prouver l’identité de l’application au fournisseur d’identité. Une connexion sécurisée peut être obtenue sur les serveurs principaux des applications ou API web en déployant un certificat.

### <a name="application-secrets"></a>Secrets de l’application

![Diagramme de client confidentiel avec mot de passe](media/msal-authentication-flows/confidential-client-password.png)

Dans le diagramme ci-dessus, l’application :

1. Acquiert un jeton à l’aide d’un secret d’application ou d’informations d’identification avec mot de passe.
2. Utilise le jeton pour effectuer des demandes sur la ressource.

### <a name="certificates"></a>Certificats

![Diagramme de client confidentiel avec certificat](media/msal-authentication-flows/confidential-client-certificate.png)

Dans le diagramme ci-dessus, l’application :

1. Acquiert un jeton à l’aide d’informations d’identification de certificat.
2. Utilise le jeton pour effectuer des demandes sur la ressource.

Ces informations d’identification client doivent être :

- Inscrites auprès d’Azure AD.
- Transmises lors de la construction de l’objet de l’application cliente confidentielle dans votre code.

## <a name="device-code"></a>Code d’appareil

Le [flux de code d’appareil OAuth 2](v2-oauth2-device-code.md) permet aux utilisateurs de se connecter à des appareils à entrée limitée, par exemple des télévisions connectées, des appareils IoT et des imprimantes. L’authentification interactive avec Azure AD nécessite un navigateur web. Lorsque l’appareil ou le système d’exploitation ne propose pas de navigateur web, le flux de code d’appareil permet à l’utilisateur d’utiliser un autre appareil (par exemple un ordinateur ou un téléphone mobile) pour se connecter de manière interactive.

En utilisant le flux de code d’appareil, l’application obtient les jetons via un processus en deux étapes, conçu pour ces appareils et systèmes d’exploitation. De telles applications incluent par exemple celles qui s’exécutent sur des appareils IoT et des outils d’interface de ligne de commande (CLI).

![Diagramme de flux de code d’appareil](media/msal-authentication-flows/device-code.png)

Dans le diagramme ci-dessus :

1. Chaque fois que l’authentification de l’utilisateur est nécessaire, l’application fournit un code et invite l’utilisateur à utiliser un autre appareil comme un smartphone connecté à Internet pour accéder à une URL (par exemple, `https://microsoft.com/devicelogin`). L’utilisateur est ensuite invité à entrer le code ; son expérience se poursuit avec une authentification normale, présentant des invites de consentement et une [authentification multifacteur](../authentication/concept-mfa-howitworks.md) si nécessaire.
1. Après une authentification réussie, l’application de ligne de commande reçoit les jetons requis via un canal arrière et les utilise pour effectuer les appels d’API web dont il a besoin.

### <a name="constraints"></a>Contraintes

- Le flux de code d’appareil est disponible uniquement sur les applications clientes publiques.
- L’autorité transmise lors de la construction de l’application cliente publique doit être l’une des suivantes :
  - Avec locataire, sous la forme `https://login.microsoftonline.com/{tenant}/,` où `{tenant}` est le GUID représentant l’ID de locataire ou un nom de domaine associé au locataire.
  - Pour les comptes professionnels et scolaires sous la forme `https://login.microsoftonline.com/organizations/`.

## <a name="implicit-grant"></a>Octroi implicite.

Le [flux d’octroi implicite OAuth 2](v2-oauth2-implicit-grant-flow.md) permet à l’application d’obtenir des jetons de la plateforme d’identités Microsoft sans échanger les informations d’identification du serveur principal. Grâce à ce flux, l’application peut alors connecter l’utilisateur, maintenir une session et obtenir des jetons pour d’autres API web, le tout dans le code JavaScript client.

![Diagramme d’un flux d’octroi implicite](media/msal-authentication-flows/implicit-grant.svg)

De nombreuses applications web modernes sont créées en tant qu’applications monopages (SPA) côté client et écrites avec JavaScript ou une infrastructure de type SPA comme Angular, Vue.js et React.js. Ces applications s’exécutent dans un navigateur web et présentent des caractéristiques d’authentification différentes de celles des applications web côté serveur traditionnelles. La plateforme d’identité Microsoft permet aux applications monopages de connecter des utilisateurs et d’obtenir des jetons pour accéder aux services principaux ou aux API web à l’aide du flux d’octroi implicite. Le flux implicite permet à l’application d’obtenir des jetons d’ID pour représenter l’utilisateur authentifié, mais également des jetons d’accès qui sont nécessaires pour appeler des API protégées.

Ce flux d’authentification n’inclut pas les scénarios d’application qui utilisent des infrastructures JavaScript inter-plateformes comme Electron ou React-Native, car ils nécessitent des fonctionnalités plus poussées pour l’interaction avec les plateformes natives.

Les jetons émis via le mode de flux implicite comportent une **limite de longueur** car ils sont retournés au navigateur par URL (où `response_mode` est `query` ou `fragment`). Certains navigateurs limitent la longueur de l’URL dans la barre de navigateur et génèrent une erreur si l’URL est trop longue. Par conséquent, ces jetons de flux implicite ne contiennent pas de revendications `groups` ou `wids`.

## <a name="on-behalf-of"></a>On-behalf-of

Le [flux d’authentification On-Behalf-Of OAuth 2](v2-oauth2-on-behalf-of-flow.md) est utilisé lorsqu’une application appelle un service ou une API web, qui à son tour doit appeler un autre service ou une autre API web. L’idée est de propager l’identité et les autorisations de l’utilisateur délégué via la chaîne de la demande. Pour que le service de niveau intermédiaire puisse faire des demandes authentifiées au service en aval, il doit sécuriser un jeton d’accès de la plateforme d’identité Microsoft *pour le compte* de l’utilisateur.

![Diagramme de flux on-behalf-of](media/msal-authentication-flows/on-behalf-of.png)

Dans le diagramme ci-dessus :

1. L’application acquiert un jeton d’accès pour l’API web.
2. Un client (application web, de bureau, mobile ou monopage) appelle une API web protégée, en ajoutant le jeton d’accès comme porteur dans l’en-tête d’authentification de la requête HTTP. L’API web authentifie l’utilisateur.
3. Lorsque le client appelle l’API web, cette dernière demande un autre jeton pour le compte de l’utilisateur.
4. L’API web protégée utilise ce jeton pour appeler une APIweb en aval pour le compte de l’utilisateur. L’API web pourra également ultérieurement demander des jetons pour les autres API en aval (mais toujours pour le compte du même utilisateur).

## <a name="usernamepassword"></a>Nom d'utilisateur/mot de passe

L’[octroi des informations d’identification de mot de passe du propriétaire des ressources OAuth 2](v2-oauth-ropc.md) (ROPC) permet à une application de connecter l’utilisateur en gérant directement son mot de passe. Dans votre application de bureau, vous pouvez utiliser le flux de nom d’utilisateur/mot de passe pour acquérir un jeton silencieusement. Aucune interface utilisateur n’est requise lorsque vous utilisez l’application.

![Diagramme de flux de nom d’utilisateur/mot de passe](media/msal-authentication-flows/username-password.png)

Dans le diagramme ci-dessus, l’application :

1. Acquiert un jeton en envoyant le nom d’utilisateur et le mot de passe au fournisseur d’identité.
2. Appelle une API web en utilisant le jeton.

> [!WARNING]
> Ce flux n’est pas recommandé. Il requiert un niveau élevé de confiance et expose considérablement les informations d'identification. Utilisez ce flux *uniquement* si aucun autre flux plus sécurisé ne peut être utilisé. Pour plus d’informations, consultez l’article sur [la solution aux problèmes croissants associés aux mots de passe](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/).

Le flux par défaut pour l’acquisition d’un jeton silencieusement sur les ordinateurs Windows joints à un domaine est [l’authentification Windows intégrée](#integrated-windows-authentication). Dans d’autres cas, utilisez le [flux de code d’appareil](#device-code).

Même si le flux nom d'utilisateur/mot de passe peut parfois s’avérer utile dans certains scénarios comme DevOps, essayez de l’éviter si vous souhaitez utiliser le nom d'utilisateur/mot de passe dans des scénarios interactifs dans lesquels vous fournissez votre propre interface utilisateur.

Avec le flux de nom d’utilisateur/mot de passe :

- Les utilisateurs qui doivent utiliser une authentification multifacteur ne pourront pas se connecter car aucune interaction n’est disponible.
- Les utilisateurs ne pourront pas utiliser l’authentification unique.

### <a name="constraints"></a>Contraintes

En dehors des [contraintes inhérentes à l’authentification Windows intégrée](#integrated-windows-authentication), les contraintes suivantes s’appliquent :

- Le flux de nom d’utilisateur/mot de passe n’est pas compatible avec l’accès conditionnel et l’authentification multifacteur. Par conséquent, si votre application s’exécute dans un locataire Azure AD pour lequel l’administrateur de locataires exige une authentification multifacteur, vous ne pouvez pas utiliser ce flux. De nombreuses organisations font cela.
- ROPC fonctionne uniquement pour les comptes professionnels et scolaires. Vous ne pouvez pas l’utiliser pour les comptes Microsoft (MSA).
- Ce flux est disponible sur les applications de bureau .NET et .NET Core, mais pas sur la plateforme Windows universelle.
- Dans Azure AD B2C, le flux ROPC fonctionne uniquement pour les comptes locaux. Pour plus d’informations sur ROPC dans MSAL.NET et Azure AD B2C, consultez [Utilisation de ROPC avec Azure AD B2C](msal-net-aad-b2c-considerations.md#resource-owner-password-credentials-ropc).

## <a name="integrated-windows-authentication"></a>Authentification Windows intégrée

MSAL prend en charge l’authentification Windows intégrée (IWA) pour les applications de bureau et mobiles qui s’exécutent sur un ordinateur Windows joint à un domaine ou à Azure AD. Avec l’authentification Windows intégrée, ces applications peuvent acquérir un jeton silencieusement sans interaction de l’utilisateur avec l’interface utilisateur.

![Diagramme dl’authentification Windows intégrée](media/msal-authentication-flows/integrated-windows-authentication.png)

Dans le diagramme ci-dessus, l’application :

1. Acquiert un jeton à l’aide de l’authentification Windows intégrée.
2. Utilise le jeton pour effectuer des demandes sur la ressource.

### <a name="constraints"></a>Contraintes

L’authentification Windows intégrée prend *uniquement* en charge les utilisateurs fédérés, c’est-à-dire les utilisateurs créés dans Active Directory et reposant sur Azure AD. Les utilisateurs créés directement dans Azure AD, sans sauvegarde Active Directory (utilisateurs managés) ne peuvent pas utiliser ce flux d’authentification. Cette restriction ne concerne pas le [flux de nom d’utilisateur/mot de passe](#usernamepassword).

L’authentification Windows intégrée est conçue pour les applications compatibles avec .NET Framework, .NET Core et la plateforme Windows universelle.

IWA ne permet pas de contourner l’authentification multifacteur. Si l’authentification multifacteur est configurée, l’authentification Windows intégrée peut échouer si une demande d’authentification multifacteur est requise. L’authentification multifacteur nécessite une interaction utilisateur.

Vous ne pouvez pas contrôler les demandes d’authentification à deux facteurs côté fournisseur d’identité. L’administrateur de locataires le peut. En règle générale, l’authentification à deux facteurs est obligatoire lorsque vous vous connectez à partir d’un autre pays/une autre région, quand vous n’êtes pas connecté via un VPN à un réseau d’entreprise, et parfois même lorsque vous êtes connecté via un VPN. Azure AD utilise l’intelligence artificielle pour apprendre en continu à déterminer si l’authentification à deux facteurs est requise. En cas d’échec d’IWA, vous devez revenir à une [invite utilisateur interactive](#interactive-and-non-interactive-authentication).

L’autorité transmise lors de la construction de l’application cliente publique doit être l’une des suivantes :

- Avec locataire, sous la forme `https://login.microsoftonline.com/{tenant}/,` où `{tenant}` est le GUID représentant l’ID de locataire ou un nom de domaine associé au locataire.
- Pour tous les comptes professionnels et scolaires (`https://login.microsoftonline.com/organizations/`). Les comptes personnels Microsoft (MSA) ne sont pas pris en charge : vous ne pouvez pas utiliser les locataires `/common` ou `/consumers`.

Étant donné que l’authentification Windows intégrée est un flux silencieux, l’une des conditions suivantes doit être remplie :

- L’utilisateur de votre application doit avoir précédemment donné son consentement pour utiliser l’application.
- L’administrateur de locataires doit avoir précédemment consenti à ce que tous les utilisateurs dans la locataire utilisent l’application.

Cela signifie que l’une des conditions suivantes est remplie :

- En tant que développeur, vous avez choisi l’option **Accorder** dans le portail Azure pour vous-même.
- Un administrateur de locataires a sélectionné **Accorder/révoquer le consentement administrateur pour {domaine du locataire}** dans l’onglet **Autorisations de l’API** lors de l’inscription de l’application dans le portail Azure (consultez [Ajouter des autorisations pour accéder à votre API web](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api)).
- Vous avez fourni un moyen aux utilisateurs de donner leur consentement pour l’utilisation de l’application ; consultez [Demande de consentement d’utilisateur individuel](v2-permissions-and-consent.md#requesting-individual-user-consent).
- Vous avez fourni un moyen à l’administrateur de locataires de donner son consentement pour l’utilisation de l’application ; consultez [Consentement administrateur](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

Le flux d’authentification Windows intégrée est activé pour les applications de bureau .NET, .NET Core et la plateforme universelle Windows. Sur .NET Core, vous devez fournir le nom d’utilisateur à IWA, car .NET Core ne peut pas obtenir les noms d’utilisateur du système d’exploitation.

Pour plus d’informations sur le consentement, consultez [Autorisations et consentement dans v2.0](v2-permissions-and-consent.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez examiné les flux d’authentification pris en charge par Microsoft Authentication Library (MSAL), découvrez comment acquérir et mettre en cache les jetons utilisés dans ces flux :

[Acquérir et mettre en cache des jetons à l’aide de Microsoft Authentication Library (MSAL)](msal-acquire-cache-tokens.md)
