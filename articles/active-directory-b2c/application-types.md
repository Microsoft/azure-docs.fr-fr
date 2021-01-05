---
title: Types d’applications pris en charge par Azure AD B2C
titleSuffix: Azure AD B2C
description: Découvrez les types d’applications que vous pouvez utiliser avec Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/24/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 07897823a3ba3b83e240e8e8dc005ea13b036fce
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94952044"
---
# <a name="application-types-that-can-be-used-in-active-directory-b2c"></a>Types d’applications pouvant être utilisés dans Active Directory B2C
 
Azure Active Directory B2C (Azure AD B2C) prend en charge l’authentification pour une large gamme d’architectures d’applications modernes. Toutes sont basées sur les protocoles standard [OAuth 2.0](protocols-overview.md) ou [OpenID Connect](protocols-overview.md). Cet article décrit les types d’applications que vous pouvez créer, indépendamment de votre langage et de votre plateforme préférés. Il vous permet également de comprendre les principaux scénarios avant de commencer à créer des applications.

Chaque application qui utilise Azure AD B2C doit être inscrite auprès de votre [locataire Azure AD B2C](tutorial-create-tenant.md) dans le [portail Azure](https://portal.azure.com/). Le processus d’inscription des applications collecte et attribue des valeurs, par exemple :

* Un **ID d’application** qui identifie de manière unique votre application
* Une **URL de réponse**  pouvant être utilisé pour rediriger les réponses vers votre application

Chaque requête envoyée à Azure AD B2C spécifie un **flux d’utilisateur** (une stratégie intégrée) ou une **stratégie personnalisée** qui contrôle le comportement d’Azure AD B2C. Ces types de stratégies vous permettent de créer un ensemble d’expériences utilisateur hautement personnalisable.

Le mode d’interaction de chaque application suit un modèle général similaire :

1. L’application dirige l’utilisateur vers le point de terminaison v2.0 pour exécuter une [stratégie](user-flow-overview.md).
2. L'utilisateur exécute la stratégie en fonction de la définition de celle-ci.
3. L’application reçoit un jeton de sécurité de la part du point de terminaison v2.0.
4. L’application utilise le jeton de sécurité pour accéder aux informations ou à la ressource protégées.
5. Le serveur de ressources valide le jeton de sécurité afin de garantir l’octroi de l’accès.
6. L’application actualise régulièrement le jeton de sécurité.

Ces étapes peuvent varier légèrement selon le type d’application que vous créez.

## <a name="web-applications"></a>Applications web

Pour les applications web (notamment .NET, PHP, Java, Ruby, Python et Node.js) qui sont hébergées sur un serveur et accessibles par le biais d’un navigateur, Azure AD B2C prend en charge [OpenID Connect](protocols-overview.md) pour toutes les expériences utilisateur. Dans l’implémentation Azure AD B2C d’OpenID Connect, votre application web déclenche les expériences utilisateur en envoyant des demandes d’authentification à Azure AD. Le résultat de la demande est un élément `id_token`. Ce jeton de sécurité représente l’identité de l’utilisateur. Il fournit également des informations sur l’utilisateur sous forme de revendications :

```json
// Partial raw id_token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded id_token
{
    "name": "John Smith",
    "email": "john.smith@gmail.com",
    "oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
    ...
}
```

Pour plus d’informations sur les différents types de jetons et de revendications disponibles pour une application, consultez la [documentation de référence sur les jetons Azure AD B2C](tokens-overview.md).

Dans les applications web, chaque exécution d’une [stratégie](user-flow-overview.md) suit cette procédure générale :

1. L’utilisateur accède à l’application web.
2. L’application web redirige l’utilisateur vers Azure AD B2C en indiquant la stratégie à exécuter.
3. L’utilisateur exécute la stratégie.
4. Azure AD B2C retourne un `id_token` au navigateur.
5. Le `id_token` est publié dans l’URI de redirection.
6. Le `id_token` est validé et un cookie de session est défini.
7. Une page sécurisée est retournée à l’utilisateur.

La validation de l’élément `id_token` à l’aide d’une clé de signature publique provenant d’Azure AD est suffisante pour vérifier l’identité de l’utilisateur. Ce processus définit également un cookie de session qui peut être utilisé pour identifier l’utilisateur sur les demandes de page suivantes.

Pour voir ce scénario à l’œuvre, exécutez l’un des exemples de code de connexion d’application web de la section [Prise en main](overview.md).

Outre la simplification des connexions, une application de serveur web peut également nécessiter l’accès à un service web backend. Dans ce cas, l’application web peut exécuter un [flux OpenID Connect](openid-connect.md) légèrement différent et acquérir des jetons à l’aide de codes d’autorisation et de jetons d’actualisation. Ce scénario est représenté dans la section [API Web](#web-apis)ci-après.

## <a name="single-page-applications"></a>Applications monopages
De nombreuses applications web modernes sont créées en tant qu’applications monopages (« SPA ») côté client. Les développeurs les écrivent à l’aide de JavaScript ou d’un framework d’application monopage, comme Angular, Vue et React. Ces applications s’exécutent sur un navigateur web et présentent des caractéristiques d’authentification différentes de celles des applications web classiques côté serveur.

Azure AD B2C offre **deux** options pour permettre aux applications monopages d’effectuer la connexion des utilisateurs et d’obtenir des jetons pour accéder aux services back-end ou aux API web :

### <a name="authorization-code-flow-with-pkce"></a>Flux de code d’autorisation (avec PKCE)
- [Flux de code d’autorisation OAuth 2.0 (avec PKCE)](./authorization-code-flow.md). Le flux de code d’autorisation permet à l’application d’échanger un code d’autorisation pour obtenir des jetons d’**ID** afin de représenter l’utilisateur authentifié et des jetons d’**accès** nécessaires pour appeler des API protégées. De plus, il retourne des jetons d’**actualisation** qui fournissent à votre application un accès à long terme à des ressources au nom d’utilisateurs sans nécessiter l’intervention de ces utilisateurs. 

Il s’agit de l’approche **recommandée**. Le fait d’avoir des jetons d’actualisation à durée de vie limitée permet à votre application de s’adapter aux [restrictions de confidentialité des cookies des navigateurs modernes](../active-directory/develop/reference-third-party-cookies-spas.md), tels que Safari ITP.

Pour tirer parti de ce flux, votre application peut utiliser une bibliothèque d’authentification qui le prend en charge, par exemple [MSAL.js 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser).

<!-- ![Single-page applications-auth](./media/tutorial-single-page-app/spa-app-auth.svg) -->
![Authentification des applications monopages](./media/tutorial-single-page-app/active-directory-oauth-code-spa.png)

### <a name="implicit-grant-flow"></a>Octroi de flux implicite
- [Flux implicite OAuth 2.0](implicit-flow-single-page-application.md). Certaines infrastructures, telles que [MSAL.js 1.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core), ne prennent en charge que le flux d’octroi implicite. Le flux d’octroi implicite permet à l’application d’obtenir des jetons d’**ID** et d’**accès**. Contrairement au flux de code d’autorisation, le flux d’octroi implicite ne retourne pas de **jeton d’actualisation**. 

Ce flux d’authentification n’inclut pas de scénarios d’application utilisant des frameworks JavaScript multiplateformes, tels qu’Electron et React-Native. Ces scénarios exigent davantage de capacités pour l’interaction avec les plateformes natives.

## <a name="web-apis"></a>API Web

Vous pouvez utiliser Azure AD B2C pour sécuriser les services web, comme l’API web RESTful de votre application. Les API web peuvent utiliser OAuth 2.0 pour sécuriser leurs données, en authentifiant les requêtes HTTP entrantes à l’aide de jetons. L’appelant d’une API web ajoute un jeton dans l’en-tête d’autorisation d’une requête HTTP :

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
``` 

L’API web peut ensuite utiliser le jeton pour vérifier l’identité de l’appelant de l’API et extraire les informations à son sujet, à partir des revendications encodées dans le jeton. Pour plus d’informations sur les différents types de jetons et de revendications disponibles pour une application, voir la [référence sur les jetons Azure AD B2C](tokens-overview.md).

Une API web peut recevoir des jetons de tous types de clients, notamment des applications web, des applications de bureau et mobiles, des applications monopages, des démons côté serveur, et même d’autres API web. Voici un exemple de flux complet d’une application web appelant une API web :

1. L’application web exécute une stratégie et l’utilisateur termine l’expérience utilisateur.
2. Azure AD B2C renvoie un `id_token` (OpenID Connect) et un code d’autorisation au navigateur.
3. Le navigateur publie le `id_token` et le code d’autorisation à l’URI de redirection.
4. Le serveur web valide le `id_token` et définit un cookie de session.
5. Le serveur web demande à Azure AD B2C un `access_token` en lui fournissant le code d’autorisation, l'ID du client d'application et les informations d’identification du client.
6. Le `access_token` et le `refresh_token` sont retournés au serveur web.
7. L’API web est appelée avec le `access_token` dans un en-tête d’autorisation.
8. L’API web valide le jeton.
9. Les données sécurisées sont retournées à l’application web.

Pour plus d’informations sur les codes d’autorisation, les jetons d’actualisation et les étapes d’obtention des jetons, voir l’article concernant le [protocole OAuth 2.0](authorization-code-flow.md).

Pour savoir comment sécuriser une API web avec Azure AD B2C, consultez les didacticiels d’API web dans notre section [Prise en main](overview.md).

## <a name="mobile-and-native-applications"></a>Applications mobiles et natives

Les applications installées sur des appareils, comme les applications de bureau et les applications mobiles, nécessitent souvent que vous accédiez aux services backend ou aux API web pour le compte de l’utilisateur. Vous pouvez ajouter des expériences personnalisées de gestion des identités à vos applications natives et appeler de manière sécurisée les services backend à l’aide d’Azure AD B2C et du [flux de code d’autorisation OAuth 2.0](authorization-code-flow.md).

Dans ce flux, l’application exécute des [stratégies](user-flow-overview.md) et reçoit un `authorization_code` d’Azure AD une fois que l’utilisateur a exécuté la stratégie. `authorization_code` représente l’autorisation de l’application à appeler les services backend pour le compte de l’utilisateur actuellement connecté. L’application peut ensuite échanger `authorization_code` en arrière-plan contre `access_token` et `refresh_token`.  L’application peut utiliser `access_token` pour s’authentifier auprès d’une API web backend dans les requêtes HTTP. Elle peut également utiliser l’élément `refresh_token` pour obtenir un nouvel élément `access_token` lorsque le précédent arrive à expiration.

## <a name="current-limitations"></a>Limites actuelles

### <a name="unsupported-application-types"></a>Types d'applications non pris en charge

#### <a name="daemonsserver-side-applications"></a>Démons et applications côté serveur

Les applications qui contiennent des processus de longue durée ou qui fonctionnent sans la présence d’un utilisateur doivent également disposer d’un moyen d’accès aux ressources sécurisées, comme les API web. Ces applications peuvent s’authentifier et obtenir des jetons à l’aide de l’identité d’application (plutôt qu’avec l’identité déléguée d’un utilisateur), et à l’aide du flux des informations d’identification du client OAuth 2.0. Le flux d’informations d’identification du client n’est pas le même que celui avec emprunt d’identité, et celui-ci ne doit pas être utilisé pour l’authentification de serveur à serveur.

Bien que le flux d’octroi d’informations d’identification du client OAuth 2.0 ne soit pas directement pris en charge par le service d’authentification Azure AD B2C, vous pouvez configurer le flux d’informations d’identification du client à l’aide d’Azure AD et de la plateforme d’identités Microsoft ou du point de terminaison de jeton pour une application dans votre locataire Azure AD B2C. Un locataire Azure AD B2C partage certaines fonctionnalités avec les locataires d’entreprise Azure AD.

Pour configurer le flux d’informations d’identification du client, consultez [Azure Active Directory v2.0 et le flux d’informations d’identification du client OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Une authentification réussie entraîne la réception d’un jeton mis en forme de telle sorte qu’il peut être utilisé par Azure AD, comme décrit dans la [documentation de référence sur les jetons Azure AD](../active-directory/develop/id-tokens.md).

Pour obtenir des instructions sur l’inscription d’une application de gestion, voir [Gérer Azure AD B2C avec Microsoft Graph](microsoft-graph-get-started.md).

#### <a name="web-api-chains-on-behalf-of-flow"></a>Chaînes d’API web (flux On-Behalf-Of)

De nombreuses architectures incluent une API web qui doit appeler une autre API web en aval, toutes deux sécurisées par Azure AD B2C. Ce scénario est courant dans les clients natifs qui disposent d’une API web principale et appellent un service en ligne Microsoft, comme l’API Microsoft Graph.

Ce scénario d’API web chaînée peut être pris en charge à l’aide de la concession des informations d’identification du porteur OAuth 2.0 Jwt, également appelé flux On-Behalf-Of.  Toutefois, le flux On-Behalf-Of n’est pas implémenté dans Azure AD B2C pour l’instant.

### <a name="faulted-apps"></a>Applications ayant généré une erreur

Ne modifiez pas les applications Azure AD B2C comme suit :

- Sur les autres portails de gestion des applications tels que le [Portail d’inscription des applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
- À l’aide de l’API Graph ou de PowerShell

Si vous modifiez l’application Azure AD B2C en dehors du portail Azure, elle devient défaillante et n'est plus utilisable avec Azure AD B2C. Supprimer l’application, puis recréez-la.

Pour supprimer l’application, accédez au [Portail d’inscription des applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade), puis supprimez l’application à cet emplacement. Pour que l’application soit visible, vous devez en être le propriétaire (et non simplement un administrateur du locataire).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les stratégies intégrées fournies par [Flux d’utilisateur dans Azure Active Directory B2C](user-flow-overview.md).