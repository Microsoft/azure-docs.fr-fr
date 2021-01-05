---
title: Acquérir et mettre en cache des jetons à l’aide de la bibliothèque d’authentification Microsoft (MSAL) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez l’acquisition et la mise en cache des jetons avec MSAL.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/04/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 34cdaa42f3a41ae04c73c570bb4fede01a786af2
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107836"
---
# <a name="acquire-and-cache-tokens-using-the-microsoft-authentication-library-msal"></a>Acquérir et mettre en cache des jetons à l’aide de la bibliothèque d’authentification Microsoft (MSAL)

Les [jetons d’accès](access-tokens.md) permettent aux clients d’appeler de manière sécurisée des API web protégées par Azure. Il existe plusieurs façons d’acquérir un jeton en utilisant la bibliothèque d’authentification Microsoft (MSAL). Certaines nécessitent une interaction de l’utilisateur via un navigateur web, d’autres n’en demandent aucune. En règle générale, le moyen utilisé pour acquérir un jeton dépend de la nature de l’application : ce peut être une application cliente publique, comme une application de bureau ou mobile, mais aussi une application cliente confidentielle, telle qu’une application web, une API web ou une application de démon.

MSAL met en cache un jeton après son acquisition. Le code de votre application doit d’abord essayer d’obtenir un jeton silencieusement à partir du cache, avant de tenter de l’acquérir par d’autres moyens.

Vous pouvez également effacer le cache de jetons, en supprimant les comptes du cache. Le cookie de session qui se trouve dans le navigateur n’est quant à lui pas supprimé.

## <a name="scopes-when-acquiring-tokens"></a>Étendues lors de l’acquisition de jetons

Les [étendues](v2-permissions-and-consent.md) correspondent aux autorisations qu’une API web expose pour que les applications clientes puissent demander à y accéder. Les applications clientes demandent le consentement de l’utilisateur pour ces étendues lors des demandes d’authentification pour obtenir des jetons permettant d’accéder aux API web. MSAL vous permet d’obtenir des jetons pour accéder à des API Azure AD pour développeurs (v1.0) et de la plateforme d’identités Microsoft (v2.0). Le protocole v2.0 utilise des étendues au lieu de ressources dans les demandes. Pour plus d’informations, consultez la [comparaison entre v1.0 et v2.0](../azuread-dev/azure-ad-endpoint-comparison.md). Selon la configuration de la version de jeton acceptée par l’API web, le point de terminaison v2.0 retourne un jeton d’accès à MSAL.

Plusieurs méthodes d’acquisition de jetons de MSAL nécessitent un paramètre `scopes`. Le paramètre `scopes` représente une liste de chaînes qui déclarent les autorisations souhaitées et les ressources demandées. Les [autorisations Microsoft Graph](/graph/permissions-reference) sont des étendues bien connues.

Il est également possible d’accéder à des ressources v1.0 dans MSAL. Pour plus d’informations, consultez [Étendues pour une application v1.0](msal-v1-app-scopes.md).

### <a name="request-scopes-for-a-web-api"></a>Demander des étendues pour une API web

Quand votre application a besoin de demander un jeton d’accès avec des autorisations spécifiques pour une API de ressource, passez les étendues contenant l’URI de l’ID d’application de l’API au format `<app ID URI>/<scope>`.

Voici quelques exemples de valeurs d’étendue pour différentes ressources :

- API Microsoft Graph : `https://graph.microsoft.com/User.Read`
- API web personnalisée : `api://11111111-1111-1111-1111-111111111111/api.read`

Le format de la valeur d’étendue varie en fonction de la ressource (l’API) qui reçoit le jeton d’accès, et des valeurs de revendication `aud`qu’elle accepte.

Pour Microsoft Graph uniquement, l’étendue `user.read` mappe à `https://graph.microsoft.com/User.Read`, et les deux formats d’étendue peuvent être utilisés indifféremment.

Certaines API web, comme l’API Azure Resource Manager (https://management.core.windows.net/) , attendent une barre oblique (/) de fin dans la revendication d’audience (`aud` ) du jeton d’accès. Dans ce cas, transmettez l’étendue sous la forme `https://management.core.windows.net//user_impersonation`, en incluant la double barre oblique (//).

D’autres API peuvent exiger qu’*aucun schéma ou hôte* ne soit inclus dans la valeur d’étendue, et que seuls l’ID d’application (un GUID) et le nom de l’étendue soient attendus, par exemple :

`11111111-1111-1111-1111-111111111111/api.read`

> [!TIP]
> Si la ressource en aval n’est pas sous votre contrôle, vous devrez peut-être essayer différents formats de valeur d’étendue (par exemple, avec/sans schéma et hôte) si vous recevez `401` ou d’autres erreurs lors du passage du jeton d’accès à la ressource.

### <a name="request-dynamic-scopes-for-incremental-consent"></a>Demander des étendues dynamiques pour un consentement incrémentiel

À mesure que les fonctionnalités fournies par votre application, ou les exigences, changent, vous pouvez au besoin demander des autorisations supplémentaires en utilisant le paramètre d’étendue. De telles *étendues dynamiques* permettent à vos utilisateurs de fournir un consentement incrémentiel aux étendues.

Par exemple, vous pouvez connecter l’utilisateur, mais lui refuser au départ l’accès aux ressources. Plus tard, vous pouvez lui donner la possibilité de consulter son calendrier, en demandant l’étendue du calendrier dans la méthode d’acquisition de jeton et en obtenant le consentement de l’utilisateur pour ce faire. Par exemple, en demandant les étendues `https://graph.microsoft.com/User.Read` et `https://graph.microsoft.com/Calendar.Read`.

## <a name="acquiring-tokens-silently-from-the-cache"></a>Acquisition silencieuse de jetons (à partir du cache)

MSAL gère un cache de jetons (ou deux caches pour les applications clientes confidentielles) et met en cache un jeton après qu’il a été acquis. Dans de nombreux cas, une tentative d’obtention silencieuse d’un jeton acquiert un autre jeton avec d’autres étendues basées sur un jeton dans le cache. MSAL est également capable d’actualiser un jeton quand il arrive à expiration (étant donné que le cache de jetons contient également un jeton d’actualisation).

### <a name="recommended-call-pattern-for-public-client-applications"></a>Modèle d’appel recommandé pour les applications clientes publiques

Le code de l’application doit d’abord essayer d’obtenir un jeton silencieusement à partir du cache. Si l’appel de méthode retourne une erreur ou exception de type « Interface utilisateur requise », essayez d’acquérir un jeton par d’autres moyens.

Il existe deux flux, cependant, dans lesquels vous **ne devez pas** essayer d’acquérir silencieusement un jeton :

- Le [flux des informations d’identification du client](msal-authentication-flows.md#client-credentials), qui n’utilise pas le cache de jetons d’utilisateur, mais un cache de jetons d’application. Cette méthode se charge de vérifier le cache de jetons d’application avant d’envoyer une demande au service d’émission de jeton de sécurité (STS).
- Le [flux du code d’autorisation](msal-authentication-flows.md#authorization-code) dans les applications web, car il accepte un code que l’application a obtenu en connectant l’utilisateur et en lui demandant son consentement pour d’autres d’étendues. Dans la mesure où du code, et non un compte, est passé comme paramètre, la méthode ne peut pas examiner le cache avant d’accepter le code, ce qui lance un appel au service.

### <a name="recommended-call-pattern-in-web-apps-using-the-authorization-code-flow"></a>Modèle d’appel recommandé dans les applications web avec le flux du code d’autorisation

Pour les applications web qui utilisent le [flux du code d’autorisation OpenID Connect](v2-protocols-oidc.md), le modèle recommandé dans les contrôleurs consiste à :

- Instancier une application cliente confidentielle avec un cache de jetons avec sérialisation personnalisée.
- Acquérir le jeton en utilisant le flux du code d’autorisation

## <a name="acquiring-tokens"></a>Acquisition des jetons

En règle générale, la méthode d’acquisition d’un jeton varie selon qu’il s’agit d’une application cliente publique ou confidentielle.

### <a name="public-client-applications"></a>Applications clientes publiques

Pour les applications clientes publiques (application de bureau ou mobile) :

- Vous acquérez souvent les jetons de manière interactive, en obligeant l’utilisateur à se connecter par le biais d’une interface utilisateur ou d’une fenêtre contextuelle.
- Vous pouvez [obtenir un jeton silencieusement pour l’utilisateur connecté](msal-authentication-flows.md#integrated-windows-authentication) à l’aide de l’authentification Windows intégrée (IWA/Kerberos) si l’application de bureau est en cours d’exécution sur un ordinateur Windows joint à un domaine ou à Azure.
- Vous pouvez [obtenir un jeton avec un nom d’utilisateur et un mot de passe](msal-authentication-flows.md#usernamepassword) dans les applications clientes de bureau .NET Framework (déconseillé). Vous n’utilisez pas de nom d’utilisateur/mot de passe dans les applications clientes confidentielles.
- Vous pouvez acquérir un jeton par le biais du [flux de code d’appareil](msal-authentication-flows.md#device-code) dans les applications qui s’exécutent sur des appareils ne disposant pas de navigateur web. L’utilisateur reçoit une URL et un code. Il accède alors à un navigateur web sur un autre appareil, puis entre le code pour se connecter. Azure AD renvoie alors un jeton à l’appareil sans navigateur.

### <a name="confidential-client-applications"></a>Applications clientes confidentielles

Pour les applications clientes confidentielles (application web, API web ou application de démon, comme un service Windows) :

- Vous acquérez des jetons **pour l’application elle-même** et non pour un utilisateur, à l’aide du [flux des informations d’identification du client](msal-authentication-flows.md#client-credentials). Cette technique peut être utilisée pour des outils de synchronisation ou des outils qui traitent des utilisateurs en général, et non un utilisateur en particulier.
- Vous utilisez le [flux OBO (on-behalf-of)](msal-authentication-flows.md#on-behalf-of) pour qu’une API web appelle une API pour le compte de l’utilisateur. L’application est identifiée avec des informations d’identification du client, afin d’acquérir un jeton basé sur une assertion d’utilisateur (par exemple, SAML ou un jeton JWT). Ce flux est utilisé par les applications qui ont besoin d’accéder aux ressources d’un utilisateur particulier dans des appels de service à service.
- Vous acquérez des jetons à l’aide du [flux du code d’autorisation](msal-authentication-flows.md#authorization-code) dans les applications web une fois que l’utilisateur s’est connecté par le biais de l’URL de la demande d’autorisation. L’application OpenID Connect utilisent généralement ce mécanisme, ce qui permet à l’utilisateur de se connecter à l’aide d’Open ID Connect, puis d’accéder aux API web pour le compte de l’utilisateur.

## <a name="authentication-results"></a>Résultats d’authentification

Quand votre client demande un jeton d’accès, Azure AD retourne également un résultat d’authentification qui inclut des métadonnées sur le jeton d’accès. Ces informations incluent le délai d’expiration du jeton d’accès et les étendues dans lesquelles il est valide. Ces données permettent à votre application d’effectuer une mise en cache intelligente des jetons d’accès sans avoir à les analyser eux-mêmes. Le résultat d’authentification expose :

- Le [jeton d’accès](access-tokens.md) pour que l’API web accède aux ressources. Cette chaîne est généralement un jeton JWT encodé en Base64, mais le client ne doit jamais regarder à l’intérieur du jeton d’accès. La stabilité du format n’est pas garantie et ce dernier peut être chiffré pour la ressource. L’écriture humaine de code qui dépend du contenu des jetons d’accès sur le client constitue l’une des sources les plus courantes d’erreurs et de ruptures de logique client.
- Le [jeton d’ID](id-tokens.md) pour l’utilisateur (un jeton JWT).
- L’expiration du jeton, qui indique la date/heure auxquelles le jeton expire.
- L’ID de locataire contient le locataire dans lequel l’utilisateur a été trouvé. Pour les utilisateurs invités (scénarios d’Azure AD B2B), l’ID de locataire correspond au locataire invité, pas au locataire unique. Quand le jeton est remis au nom d’un utilisateur, le résultat d’authentification contient également des informations sur cet utilisateur. Pour les flux de clients confidentiels où les jetons sont demandés sans aucun utilisateur (pour l’application), ces informations sur l’utilisateur sont manquantes ou inconnues.
- Les étendues pour lesquelles le jeton a été émis.
- L’ID unique de l’utilisateur.

## <a name="advanced-accessing-the-users-cached-tokens-in-background-apps-and-services"></a>(Avancé) Accès aux jetons mis en cache de l’utilisateur dans les applications et services en arrière-plan

[!INCLUDE [advanced-token-caching](../../../includes/advanced-token-cache.md)]

## <a name="next-steps"></a>Étapes suivantes

Plusieurs des plateformes que MSAL prend en charge font l’objet d’informations supplémentaires relatives au cache de jeton dans la documentation de la bibliothèque de cette plateforme. Par exemple :
- [Obtenir un jeton à partir du cache de jeton à l’aide de MSAL.NET](msal-net-acquire-token-silently.md)
- [Authentification unique avec MSAL.js](msal-js-sso.md)
- [Sérialisation personnalisée du cache de jetons dans MSAL pour Python](msal-python-token-cache-serialization.md)
- [Sérialisation personnalisée du cache de jetons dans MSAL.NET pour Java](msal-java-token-cache-serialization.md)
