---
title: Guide pratique pour gérer les changements de cookie SameSite dans le navigateur Chrome | Azure
titleSuffix: Microsoft identity platform
description: Apprenez à gérer les changements de cookie SameSite dans le navigateur Chrome.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 01/27/2020
ms.author: jmprieur
ms.reviewer: kkrishna
ms.custom: aaddev
ms.openlocfilehash: 30c4f054259aa7c3f2a9fdfaeeadd64f26dd9bea
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94444909"
---
# <a name="handle-samesite-cookie-changes-in-chrome-browser"></a>Gérer les changements de cookie SameSite dans le navigateur Chrome

## <a name="what-is-samesite"></a>Qu’est-ce que SameSite ?

`SameSite` est une propriété qui peut être définie dans les cookies HTTP pour empêcher les attaques de falsification de requête intersites (CSRF) dans les applications web :

- Lorsque `SameSite` est défini sur **Lax** , le cookie est envoyé dans les requêtes au sein du même site et dans les requêtes GET d’autres sites. Il n’est pas envoyé dans les requêtes GET qui sont interdomaines.
- La valeur **Strict** garantit que le cookie est envoyé dans des requêtes uniquement au sein du même site.

Par défaut, la valeur `SameSite` n’est PAS définie dans les navigateurs et c’est la raison pour laquelle il n’existe aucune restriction sur les cookies envoyés dans les requêtes. Une application doit s’abonner à la protection CSRF en paramétrant **Lax** ou **Strict** selon leurs exigences.

## <a name="samesite-changes-and-impact-on-authentication"></a>Modifications SameSite et impact sur l’authentification

De récentes [mises à jour des normes sur SameSite](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00) proposent de protéger les applications en définissant le comportement par défaut de `SameSite` sur l’absence de valeur définie sur Lax. Cette atténuation signifie que les cookies seront restreints sur les requêtes HTTP, à l’exception des requêtes GET provenant d’autres sites. En outre, une valeur **None** est introduite pour supprimer les restrictions concernant les cookies envoyés. Ces mises à jour seront bientôt publiées dans une prochaine version du navigateur Chrome.

Lorsque les applications web s’authentifient auprès de la plateforme Microsoft Identity à l’aide du mode de réponse « form_post », le serveur de connexion répond à l’application à l’aide d’une requête HTTP POST pour envoyer les jetons ou le code d’authentification. Étant donné que cette requête est une requête interdomaine (de `login.microsoftonline.com` à votre domaine, par exemple `https://contoso.com/auth`), les cookies qui ont été définis par votre application sont désormais soumis aux nouvelles règles de Chrome. Les cookies qui doivent être utilisés dans les scénarios intersites sont des cookies qui contiennent les valeurs *état* et *nonce* , qui sont également envoyées dans la requête de connexion. D’autres cookies sont déposés par Azure AD pour maintenir la session.

Si vous ne mettez pas à jour vos applications web, ce nouveau comportement entraînera des échecs d’authentification.

## <a name="mitigation-and-samples"></a>Atténuation et exemples

Pour surmonter les échecs d’authentification, les applications web qui s’authentifient auprès de la plateforme d’identités Microsoft peuvent définir la propriété `SameSite` sur `None` pour les cookies utilisés dans les scénarios interdomaines lorsqu’ils s’exécutent sur le navigateur Chrome.
D’autres navigateurs (voir [ici](https://www.chromium.org/updates/same-site/incompatible-clients) pour obtenir une liste complète) suivent le comportement précédent de `SameSite` et n’incluront pas les cookies si `SameSite=None` est défini.
C’est pourquoi, pour prendre en charge l’authentification sur plusieurs navigateurs, les applications web devront définir la valeur `SameSite` sur `None` uniquement sur Chrome et laisser la valeur vide sur les autres navigateurs.

Cette approche est illustrée dans nos exemples de code ci-dessous.

# <a name="net"></a>[.NET](#tab/dotnet)

Le tableau ci-dessous présente les demandes de tirage (pull request) qui ont contournées les modifications apportées à SameSite dans nos exemples ASP.NET et ASP.NET Core.

| Exemple | Demande de tirage (pull request) |
| ------ | ------------ |
|  [Didacticiel incrémentiel sur les applications web ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)  |  [Correctif pour cookie SameSite n° 261](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/pull/261)  |
|  [Exemple d’application web ASP.NET MVC](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect)  |  [Correctif pour cookie SameSite n° 35](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/pull/35)  |
|  [active-directory-dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2)  |  [Correctif pour cookie SameSite n° 28](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2/pull/28)  |

Pour plus d’informations sur la gestion des cookies SameSite dans ASP.NET et ASP.NET Core, consultez également :

- [Utiliser des cookies SameSite dans ASP.NET Core](/aspnet/core/security/samesite).
- [Blog ASP.NET concernant SameSite](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/)

# <a name="python"></a>[Python](#tab/python)

| Exemple |
| ------ |
|  [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp)  |

# <a name="java"></a>[Java](#tab/java)

| Exemple | Demande de tirage (pull request) |
| ------ | ------------ |
|  [ms-identity-java-webapp](https://github.com/Azure-Samples/ms-identity-java-webapp)  | [Correctif pour cookie SameSite n° 24](https://github.com/Azure-Samples/ms-identity-java-webapp/pull/24)
|  [ms-identity-java-webapi](https://github.com/Azure-Samples/ms-identity-java-webapi)  | [Correctif pour cookie SameSite n° 4](https://github.com/Azure-Samples/ms-identity-java-webapi/pull/4)

---

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur SameSite et le scénario des applications web :

- [FAQ de Google Chrome sur SameSite](https://www.chromium.org/updates/same-site/faq)

- [Page SameSite de Chromium](https://www.chromium.org/updates/same-site)

- [Scénario : application web qui connecte les utilisateurs](scenario-web-app-sign-user-overview.md)