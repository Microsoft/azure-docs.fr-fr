---
title: Types d’application pour la plateforme d’identité Microsoft | Azure
description: Types d’applications et de scénarios pris en charge par le point de terminaison de la plateforme d’identité Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 11/13/2020
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev, fasttrack-edit, contperf-fy21q2
ms.openlocfilehash: fd1fc59fd1ade6036c57f15415afccfc693f7bff
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029751"
---
# <a name="application-types-for-microsoft-identity-platform"></a>Types d’application pour la plateforme d’identité Microsoft

Le point de terminaison de la plateforme d’identité Microsoft prend en charge l’authentification pour plusieurs architectures d’application modernes, toutes basées sur des protocoles industriels standard [OAuth 2.0 ou OpenID Connect](active-directory-v2-protocols.md). Cet article décrit les types d’application que vous pouvez générer à l’aide de la plateforme d’identité Microsoft, quelle que soit votre plateforme ou votre langage préférés. Ces informations sont conçues pour vous aider à comprendre les scénarios de haut niveau avant de commencer à manipuler le code dans les [scénarios d’application](authentication-flows-app-scenarios.md#application-scenarios).

## <a name="the-basics"></a>Concepts de base

Vous devez inscrire chaque application utilisant la plateforme d’identité Microsoft dans le portail Azure [Inscriptions d’applications](https://go.microsoft.com/fwlink/?linkid=2083908). Le processus d’inscription des applications collecte les valeurs suivantes et les affecte à votre application :

* un **ID d’application** qui identifie de manière unique votre application ;
* un **URI de redirection** que vous pouvez utiliser pour renvoyer les réponses à votre application ;
* quelques autres valeurs spécifiques au scénario, tels que les types de compte pris en charge.

Pour en savoir plus, découvrez comment [inscrire une application](quickstart-register-app.md).

Une fois inscrite, l’application communique avec la plateforme d’identité Microsoft en transmettant les requêtes au point de terminaison. Nous fournissons les infrastructures et les bibliothèques open source qui gèrent les détails de ces requêtes. Vous avez également la possibilité d’implémenter la logique d’authentification vous-même en créant des requêtes à ces points de terminaison :

```HTTP
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
https://login.microsoftonline.com/common/oauth2/v2.0/token
```

## <a name="single-page-apps-javascript"></a>Applications à page unique (Javascript)

De nombreuses applications modernes ont un frontal d’application monopage écrit principalement en JavaScript, souvent avec une infrastructure telle que Angular, React ou Vue. Le point de terminaison de la plateforme d’identité Microsoft prend en charge ces applications avec le protocole [OpenID Connect](v2-protocols-oidc.md) pour l’authentification et soit le [flux d’octroi implicite OAuth 2.0](v2-oauth2-implicit-grant-flow.md), soit le système plus récent avec [code d’autorisation OAuth 2.0 + flux PKCE](v2-oauth2-auth-code-flow.md) pour l’autorisation (voir ci-dessous).

Le diagramme ci-dessous illustre l’octroi du code d’autorisation OAuth 2.0 (avec des détails sur le PKCE omis), dans lequel l’application reçoit un code du point de terminaison de la plateforme d’identité Microsoft `authorize` et l’utilise pour les jetons et les jetons d’actualisation à l’aide de requêtes web intersites. Le jeton d’actualisation expire toutes les 24 heures, et l’application doit demander un autre code. En plus du jeton d’accès, une `id_token` qui représente l’utilisateur connecté à l’application cliente est généralement également demandée par le biais du même flux et/ou d’une demande de connexion OpenID distincte (non illustrée ici).

:::image type="content" source="media/v2-oauth-auth-code-spa/active-directory-oauth-code-spa.svg" alt-text="Diagramme montrant le flot du code d’autorisation OAuth 2 entre une application à page unique et le point de terminaison du service d’émission de jeton de sécurité." border="false":::

Pour voir ce scénario en action, consultez le [didacticiel : Connecter les utilisateurs et appeler l’API Microsoft Graph à partir d’une application monopage JavaScript à l’aide d’un flux de code d’autorisation](tutorial-v2-javascript-auth-code.md).

### <a name="authorization-code-flow-vs-implicit-flow"></a>Flux de code d’autorisation et flux implicite

Pour l’essentiel de l’historique d’OAuth 2.0, le [flux implicite](v2-oauth2-implicit-grant-flow.md) était la méthode recommandée pour générer des applications monopages. Avec la suppression des [cookies tiers](reference-third-party-cookies-spas.md) et une [plus grande attention](https://tools.ietf.org/html/draft-ietf-oauth-security-topics-14) portée sur les problèmes de sécurité liés au flux implicite, nous avons adopté vers le code d’autorisation pour les applications monopages.

Pour garantir la compatibilité de votre application dans Safari et dans d’autres navigateurs prenant en charge la confidentialité, nous recommandons d’utiliser le flux de code d’autorisation plutôt que le flux implicite.

## <a name="web-apps"></a>les applications web

Pour les applications web (.NET, PHP, Java, Ruby, Python, Node, etc.) auxquelles l’utilisateur accède par le biais d’un navigateur, vous pouvez utiliser [OpenID Connect](active-directory-v2-protocols.md) pour la connexion de l’utilisateur. Dans OpenID Connect, l’application web reçoit un jeton d’ID. Un jeton d’ID est un jeton de sécurité qui vérifie l’identité de l’utilisateur et fournit des informations le concernant sous la forme de revendications :

```JSON
// Partial raw ID token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded ID token
{
    "name": "John Smith",
    "email": "john.smith@gmail.com",
    "oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
    ...
}
```

Pour en savoir plus sur les différents types de jetons utilisés dans le point de terminaison de la plateforme d’identité Microsoft, consultez les articles de référence au [jeton d’accès](access-tokens.md) et à [id_token reference](id-tokens.md).

Dans les applications de serveur web, le flux d’authentification de connexion respecte cette procédure de niveau supérieur :

![Affiche le flux d’authentification de l’application web](./media/v2-app-types/convergence-scenarios-webapp.svg)

Vous pouvez vérifier l’identité de l’utilisateur en validant le jeton d’ID avec une clé de signature publique reçue du point de terminaison de la plateforme d’identité Microsoft. Un cookie de session qui peut être utilisé pour identifier l’utilisateur sur les requêtes de page suivantes est défini.

Pour voir ce scénario en action, essayez les exemples de code du [scénario d'application web qui connecte des utilisateurs](scenario-web-app-sign-user-overview.md).

En plus de la connexion simple, une application de serveur web peut également nécessiter l’accès à un autre service Web, comme une API REST. Dans ce cas, l’application de serveur web s’engager dans un flux OpenID Connect et OAuth 2.0 à l’aide du [flux de code d’autorisation OAuth 2.0](v2-oauth2-auth-code-flow.md). Pour en savoir plus sur ce scénario, découvrez comment [la bien démarrer avec les applications web et des API web](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIDConnect-DotNet).

## <a name="web-apis"></a>API Web

Vous pouvez utiliser le point de terminaison de la plateforme d’identité Microsoft pour sécuriser des services web, comme l’API web RESTful de votre application. Les API web peuvent être implémentées dans de nombreuses plateformes et langages. Elles peuvent également être implémentées à l’aide de déclencheurs HTTP dans Azure Functions. En lieu et place des jetons d’ID et des cookies de session, une API web utilise les jetons d’accès OAuth 2.0 pour sécuriser les données et authentifier les requêtes entrantes. L’appelant d’une API web ajoute un jeton d’accès dans l’en-tête d’autorisation d’une requête HTTP de la manière suivante :

```HTTP
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

L’API web utilise le jeton d’accès pour vérifier l’identité de l’appelant de l’API et extraire des informations le concernant à partir de revendications encodées dans le jeton d’accès. Pour en savoir plus sur les différents types de jetons utilisés dans le point de terminaison de la plateforme d’identité Microsoft, consultez les articles de référence au [jeton d’accès](access-tokens.md) et à [id_token](id-tokens.md).

Une API web peut octroyer aux utilisateurs la possibilité d’accepter/de refuser des fonctionnalités ou données spécifiques en exposant des autorisations (également appelées [étendues](v2-permissions-and-consent.md)). Pour qu’une application appelante puisse acquérir l’autorisation à une étendue, l’utilisateur doit accepter l’étendue au cours d’un flux. La plateforme d’identité demande l’autorisation à l’utilisateur, puis enregistre ces autorisations dans l’ensemble des jetons d’accès reçus par l’API web. L’API web valide les jetons d’accès qu’elle reçoit à chaque appel et procède à des vérifications d’autorisation.

Une API web peut recevoir des jetons d’accès de tous types d’applications, notamment des applications de serveur web, des applications de bureau et mobiles, des applications de page unique, des démons côté serveur, et même d’autres API web. Le flux de haut niveau d'une API web se présente comme suit :

![Affiche le flux d’authentification d’API web](./media/v2-app-types/convergence-scenarios-webapi.svg)

Pour savoir comment sécuriser une API web avec des jetons d’accès OAuth2, consultez les exemples de code d’API web du [scénario d’API web protégée](scenario-protected-web-api-overview.md).

Dans de nombreux cas, les API web doivent également envoyer des demandes à d’autres API web en aval, sécurisées par la plateforme d’identité Microsoft. Pour ce faire, elles peuvent utiliser le flux **Au nom de** d’Azure AD, qui permet à l’API web d’échanger un jeton d’accès entrant contre un autre jeton d’accès, à utiliser pour les requêtes sortantes. Pour en savoir plus, voir [Plateforme d’identité Microsoft et flux On-Behalf-Of OAuth 2.0](v2-oauth2-on-behalf-of-flow.md).

## <a name="mobile-and-native-apps"></a>Applications mobiles et natives

Les applications installées sur un appareil, comme les applications de bureau et les applications mobiles nécessitent bien souvent un accès à des services principaux ou à des API web, qui stockent les données et exécutent des fonctions pour le compte d’un utilisateur. Ces applications peuvent ajouter des fonctionnalités de connexion et d’autorisation à des services principaux à l’aide du [flux de code d’autorisation OAuth 2.0](v2-oauth2-auth-code-flow.md).

Dans ce flux, l’application reçoit un code d’autorisation à partir du point de terminaison de la plateforme d’identité Microsoft lorsque l’utilisateur se connecte. Le code d'autorisation représente l'autorisation de l'application d'appeler les services principaux pour le compte de l'utilisateur connecté. L’application peut ensuite échanger le code d’autorisation en arrière-plan contre un jeton d’accès et un jeton d’actualisation OAuth 2.0. L’application peut utiliser le jeton d’accès pour s’authentifier sur des API web dans des requêtes HTTP et solliciter le jeton d’actualisation afin de récupérer de nouveaux jetons d’accès une fois les anciens expirés.

![Affiche le flux d’authentification des applications natives](./media/v2-app-types/convergence-scenarios-native.svg)

> [!NOTE]
> Si l’application utilise l’application System WebView par défaut, consultez les informations concernant la fonctionnalité « Confirmer ma connexion » et le code d’erreur AADSTS50199 dans [Codes d’erreur d’authentification et d’autorisation Azure AD](reference-aadsts-error-codes.md).

## <a name="daemons-and-server-side-apps"></a>Applications démons et côté serveur

Les applications qui contiennent des processus de longue durée ou qui fonctionnent sans interaction d’un utilisateur doivent également disposer d’un moyen d’accès aux ressources sécurisées, comme les API web. Ces applications peuvent s'authentifier et récupérer des jetons à l'aide de l'identité d'application plutôt qu'avec l'identité déléguée d'un utilisateur avec le flux des informations d'identification du client OAuth 2.0. Vous pouvez prouver l’identité de l’application à l’aide d’une clé secrète client ou d’un certificat. Pour plus d’informations, consultez [Application console démon .NET Core utilisant la plateforme d’identité Microsoft](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2).

Dans ce flux, l’application interagit directement avec le point de terminaison `/token` pour obtenir un accès :

![Affiche le flux d’authentification des applications démons](./media/v2-app-types/convergence-scenarios-daemon.svg)

Pour créer une application démon, consultez la [documentation sur les informations d’identification des clients](v2-oauth2-client-creds-grant-flow.md) ou consultez un [exemple d’application .NET](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2).

## <a name="next-steps"></a>Étapes suivantes

À présent que vous êtes familiarisé avec les types d’applications que la plateforme d’identité Microsoft prend en charge, apprenez-en davantage sur [OAuth 2.0 et OpenID Connect](active-directory-v2-protocols.md) afin de comprendre les composants de protocole utilisés par les différents scénarios.
