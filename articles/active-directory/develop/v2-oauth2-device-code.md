---
title: Flux du code d’appareil OAuth 2.0 | Azure
titleSuffix: Microsoft identity platform
description: Connectez des utilisateurs sans navigateur. Générez des flux d’authentification intégrés et sans navigateur à l’aide de l’octroi d’autorisation d’appareil.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 11/19/2019
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 8c757f3e067aeac5d8145ca47b2eac145daba574
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88272448"
---
# <a name="microsoft-identity-platform-and-the-oauth-20-device-authorization-grant-flow"></a>Plateforme d’identités Microsoft et flux d’octroi d’autorisation d’appareil OAuth 2.0

La plateforme d’identités Microsoft prend en charge l’[octroi d’autorisation d’appareil](https://tools.ietf.org/html/rfc8628), ce qui permet aux utilisateurs de se connecter à des appareils à entrée limitée comme une télévision connectée, un appareil IoT ou une imprimante.  Pour activer ce flux, l’appareil exige que l’utilisateur consulte une page web dans son navigateur sur un autre appareil pour se connecter.  Lorsque l’utilisateur est connecté, l’appareil peut obtenir des jetons d’accès et actualiser les jetons si nécessaire.

Cet article explique comment programmer directement par rapport au protocole dans votre application.  Dans la mesure du possible, nous vous recommandons d’utiliser les bibliothèques d’authentification Microsoft (MSAL) prises en charge au lieu d’[acquérir des jetons et d’appeler des API web sécurisées](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Jetez également un coup d’œil aux [exemples d’applications qui utilisent MSAL](sample-v2-code.md).

## <a name="protocol-diagram"></a>Schéma de protocole

Le flux de code d’appareil complet est similaire à l’illustration suivante. Nous décrirons en détail chacune des étapes plus loin dans cet article.

![Flux de code d’appareil](./media/v2-oauth2-device-code/v2-oauth-device-flow.svg)

## <a name="device-authorization-request"></a>Requête d’autorisation d’appareil

Le client doit d’abord rechercher sur le serveur d’authentification un appareil et un code utilisateur, utilisés pour lancer l’authentification. Le client collecte cette requête à partir du point de terminaison `/devicecode`. Dans cette requête, le client doit également inclure les autorisations dont il a besoin d’obtenir auprès de l’utilisateur. À partir du moment où cette requête est envoyée, l’utilisateur ne dispose que de 15 minutes pour se connecter (la valeur habituelle de `expires_in`). N’effectuez cette requête que lorsque l’utilisateur a indiqué qu’il est prêt à se connecter.

> [!TIP]
> Essayez d'exécuter cette requête dans Postman !
> [![Essayez d’exécuter cette requête dans Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```HTTP
// Line breaks are for legibility only.

POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/devicecode
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
scope=user.read%20openid%20profile

```

| Paramètre | Condition | Description |
| --- | --- | --- |
| `tenant` | Obligatoire | Peut être /common, /consumers ou /organizations.  Il peut également s’agir du locataire d’annuaire dont vous souhaitez demander une autorisation au format GUID ou de nom convivial.  |
| `client_id` | Obligatoire | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `scope` | Obligatoire | Liste séparée par des espaces d’ [étendues](v2-permissions-and-consent.md) pour lesquelles vous souhaitez que l’utilisateur donne son consentement.  |

### <a name="device-authorization-response"></a>Réponse d’autorisation d’appareil

Une réponse réussie est un objet JSON contenant les informations requises pour autoriser l’utilisateur à se connecter.

| Paramètre | Format | Description |
| ---              | --- | --- |
|`device_code`     | String | Chaîne longue utilisée pour vérifier la session entre le client et le serveur d’autorisation. Ce client est utilisé par le client pour demander le jeton d’accès au serveur d’autorisation. |
|`user_code`       | String | Chaîne courte présentée à l’utilisateur, utilisée pour identifier la session sur un appareil secondaire.|
|`verification_uri`| URI | URI auquel l’utilisateur doit accéder avec le `user_code` pour pouvoir se connecter. |
|`expires_in`      | int | Nombre de secondes avant l’expiration de `device_code` et `user_code`. |
|`interval`        | int | Nombre de secondes d’attente du client entre chaque requête d’interrogation. |
| `message`        | String | Chaîne lisible par l’homme contenant des instructions pour l’utilisateur. Elle peut être localisée en incluant un **paramètre de requête** dans la requête sous la forme `?mkt=xx-XX`, en remplissant le code de langue approprié. |

> [!NOTE]
> Le champ de réponse `verification_uri_complete` n’est ni inclus ni pris en charge pour l’instant.  Nous le mentionnons car, si vous lisez la [norme](https://tools.ietf.org/html/rfc8628), vous voyez que `verification_uri_complete` est répertorié comme une partie facultative de la norme de flux de code d’appareil.

## <a name="authenticating-the-user"></a>Authentification de l’utilisateur

Après avoir reçu `user_code` et `verification_uri`, le client les présente aux utilisateurs, en les invitant à se connecter à l’aide de leur téléphone mobile ou navigateur PC.

Si l’utilisateur s’authentifie avec un compte personnel (sur /common ou /consumers), il est invité à se connecter à nouveau pour transférer l’état d’authentification à l’appareil.  Il est également invité à donner son consentement pour s’assurer qu’il connaît les autorisations accordées.  Cela ne s’applique pas aux comptes professionnels ou scolaires utilisés pour l’authentification.

Tandis que l’utilisateur s’authentifie auprès de `verification_uri`, le client doit interroger le point de terminaison `/token` pour le jeton demandé à l’aide du `device_code`.

```HTTP
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

grant_type: urn:ietf:params:oauth:grant-type:device_code
client_id: 6731de76-14a6-49ae-97bc-6eba6914391e
device_code: GMMhmHCXhWEzkobqIHGG_EnNYYsAkukHspeYUk9E8...
```

| Paramètre | Obligatoire | Description|
| -------- | -------- | ---------- |
| `tenant`  | Obligatoire | Même locataire ou alias de locataire que celui utilisé dans la demande initiale. |
| `grant_type` | Obligatoire | Doit être `urn:ietf:params:oauth:grant-type:device_code`|
| `client_id`  | Obligatoire | Doit correspondre au `client_id` utilisé dans la requête initiale. |
| `device_code`| Obligatoire | `device_code` retourné dans la requête d’autorisation d’appareil.  |

### <a name="expected-errors"></a>Erreurs attendues

Étant donné que le flux de code d’appareil est un protocole d’interrogation, votre client doit s’attendre à recevoir des erreurs avant que l’utilisateur ait terminé l’authentification.

| Error | Description | Action du client |
| ------ | ----------- | -------------|
| `authorization_pending` | L’utilisateur n’a pas encore terminé l’authentification, mais n’a pas annulé le flux. | Répétez la requête après `interval` secondes minimum. |
| `authorization_declined` | L’utilisateur final a refusé la requête d’autorisation.| Arrêtez l’interrogation et revenez à un état non authentifié.  |
| `bad_verification_code`| Le `device_code` envoyé au point de terminaison `/token` n’a pas été reconnu. | Vérifiez que le client envoie le `device_code` approprié dans la requête. |
| `expired_token` | `expires_in` secondes minimum se sont écoulées, et l’authentification n’est plus possible avec ce `device_code`. | Arrêtez l’interrogation et revenez à un état non authentifié. |

### <a name="successful-authentication-response"></a>Réponse d’authentification réussie

Une réponse de jeton réussie se présente ainsi :

```json
{
    "token_type": "Bearer",
    "scope": "User.Read profile openid email",
    "expires_in": 3599,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD..."
}
```

| Paramètre | Format | Description |
| --------- | ------ | ----------- |
| `token_type` | String| Toujours Porteur. |
| `scope` | Chaînes séparées par un espace | Si un jeton d’accès est retourné, répertorie les étendues pour lesquelles le jeton d’accès est valide. |
| `expires_in`| int | Nombre de secondes avant lequel le jeton d’accès fourni est valide. |
| `access_token`| Chaîne opaque | Émise pour les [étendues](v2-permissions-and-consent.md) qui ont été demandées.  |
| `id_token`   | JWT | Émis si le paramètre `scope` d’origine inclut l’étendue `openid`.  |
| `refresh_token` | Chaîne opaque | Émise si le paramètre `scope` d’origine inclut `offline_access`.  |

Vous pouvez utiliser le jeton d’actualisation pour acquérir de nouveaux jetons d’accès et actualiser des jetons avec le même flux détaillé décrit dans la [documentation du flux de code OAuth](v2-oauth2-auth-code-flow.md#refresh-the-access-token).
