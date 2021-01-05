---
title: Protocoles de consentement administrateur de plateforme d’identités Microsoft
description: Il s’agit d’une description de l’autorisation dans le point de terminaison de la plateforme d’identités Microsoft, ce qui comprend notamment les étendues, les autorisations et les consentements.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 12/3/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 7de97fd775853f64803ab62ac397e754d065e4df
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509323"
---
# <a name="admin-consent-on-the-microsoft-identity-platform"></a>Consentement administrateur sur la plateforme d’identités Microsoft

Certaines autorisations requièrent le consentement d’un administrateur pour pouvoir être accordées au sein d’un locataire.  Vous pouvez également utiliser le point de terminaison de consentement administrateur pour accorder des autorisations à un locataire entier.

## <a name="recommended-sign-the-user-into-your-app"></a>Recommandé : connecter l’utilisateur à votre application

En général, lorsque vous créez une application qui utilise le point de terminaison de consentement de l’administrateur, l’application doit disposer d’une page ou vue dans laquelle l’administrateur peut approuver ses autorisations. Cette page peut faire partie du flux d’inscription de l’application, des paramètres de l’application, ou ce peut être un flux de connexion dédié. Dans de nombreux cas, il est judicieux pour l’application d’afficher la vue de « connexion » uniquement après qu’un utilisateur se soit connecté avec un compte Microsoft professionnel ou scolaire.

Lorsque vous connectez l’utilisateur à votre application, vous pouvez identifier l’organisation à laquelle l’administrateur appartient, avant de lui demander d’approuver les autorisations nécessaires. Même si cela n’est pas strictement nécessaire, cela peut vous aider à créer une expérience plus intuitive pour les utilisateurs de l’organisation. Pour connecter l’utilisateur, suivez nos [tutoriels sur le protocole de la plateforme d’identités Microsoft](active-directory-v2-protocols.md).

## <a name="request-the-permissions-from-a-directory-admin"></a>Demander les autorisations à un administrateur d’annuaire

Lorsque vous êtes prêt à demander les autorisations à l’administrateur de votre organisation, vous pouvez rediriger l’utilisateur vers le *point de terminaison de consentement administrateur* de la plateforme d’identités Microsoft.

```HTTP
// Line breaks are for legibility only.
GET https://login.microsoftonline.com/{tenant}/v2.0/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
&scope=
https://graph.microsoft.com/calendars.read
https://graph.microsoft.com/mail.send
```

| Paramètre | Condition | Description |
| ---: | ---: | :---: |
| `tenant` | Obligatoire | Le client d’annuaire auquel vous souhaitez demander l’autorisation. Peut être fourni au format GUID ou sous forme de nom convivial OU référencé de manière générique avec `organizations` comme indiqué dans l’exemple. N’utilisez pas « common », car les comptes personnels ne peuvent pas fournir le consentement de l’administrateur, sauf dans le contexte d’un locataire. Pour garantir une meilleure compatibilité avec les comptes personnels qui gèrent les locataires, utilisez l’ID de locataire, dans la mesure du possible. |
| `client_id` | Obligatoire | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `redirect_uri` | Obligatoire |L'URI de redirection où vous souhaitez que la réponse soit envoyée pour être gérée par votre application. Il doit correspondre exactement à l’un des URI de redirection que vous avez inscrits dans le portail d’inscription des applications. |
| `state` | Recommandé | Une valeur incluse dans la requête, qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. Utilisez l’état pour encoder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou la vue sur laquelle ou laquelle il était positionné. |
|`scope` | Obligatoire | Définit l’ensemble des autorisations demandées par l’application. Il peut s’agir d’étendues statiques (utilisant /.default) ou dynamiques. Cela peut inclure les étendues OIDC (`openid`, `profile`, `email`). |


À ce stade, Azure AD nécessite qu’un administrateur client se connecte pour terminer la demande. L’administrateur est invité à approuver toutes les autorisations que vous avez demandées dans le paramètre `scope`.  Si vous avez utilisé une valeur (`/.default`) statique, celle-ci fonctionne comme le point de terminaison de consentement administrateur v 1.0 et demande un consentement pour toutes les étendues trouvées dans les autorisations requises pour l’application.

### <a name="successful-response"></a>Réponse correcte

Si l’administrateur approuve les autorisations pour votre application, la réponse correcte ressemble à ce qui suit :

```
http://localhost/myapp/permissions?admin_consent=True&tenant=fa00d692-e9c7-4460-a743-29f2956fd429&state=12345&scope=https%3a%2f%2fgraph.microsoft.com%2fCalendars.Read+https%3a%2f%2fgraph.microsoft.com%2fMail.Send
```

| Paramètre | Description |
| ---: | :---: |
| `tenant`| Client d’annuaire ayant accordé à votre application les autorisations demandées au format GUID.|
| `state` | Une valeur incluse dans la requête qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. La valeur d’état est utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné.|
| `scope` | Ensemble d’autorisations auquel l’accès a été accordé pour l’application.|
| `admin_consent` | Sera défini sur `True`.|

### <a name="error-response"></a>Réponse d’erreur

`http://localhost/myapp/permissions?error=consent_required&error_description=AADSTS65004%3a+The+resource+owner+or+authorization+server+denied+the+request.%0d%0aTrace+ID%3a+d320620c-3d56-42bc-bc45-4cdd85c41f00%0d%0aCorrelation+ID%3a+8478d534-5b2c-4325-8c2c-51395c342c89%0d%0aTimestamp%3a+2019-09-24+18%3a34%3a26Z&admin_consent=True&tenant=fa15d692-e9c7-4460-a743-29f2956fd429&state=12345`

Ajout aux paramètres affichés dans une réponse correcte. Les paramètres d’erreur s’affichent comme ci-dessous.

| Paramètre | Description |
|-------------------:|:-------------------------------------------------------------------------------------------------:|
| `error` | Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs.|
| `error_description` | Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur.|
| `tenant`| Client d’annuaire ayant accordé à votre application les autorisations demandées au format GUID.|
| `state` | Une valeur incluse dans la requête qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. La valeur d’état est utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné.|
| `admin_consent` | Aura la valeur `True` pour indiquer que cette réponse s’est produite sur un flux de consentement administrateur.|

## <a name="next-steps"></a>Étapes suivantes
- Consultez [comment convertir une application multi-locataire](howto-convert-app-to-be-multi-tenant.md)
- Découvrez comment [le consentement est pris en charge au niveau de la couche du protocole OAuth 2.0 pendant le flux d’octroi de code d’autorisation](v2-oauth2-auth-code-flow.md#request-an-authorization-code).
- Découvrez [comment une application mutualisée peut utiliser l’infrastructure de consentement](./howto-convert-app-to-be-multi-tenant.md) pour implémenter un consentement de type « utilisateur » et « admin », en prenant en charge des modèles d’applications mutualisées plus avancés.
- Compréhension des [expériences de consentement de l’application Azure AD](application-consent-experience.md)