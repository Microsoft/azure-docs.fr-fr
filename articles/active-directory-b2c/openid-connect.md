---
title: Connexion web avec OpenID Connect – Azure Active Directory B2C
description: Créez des applications web à l’aide du protocole d’authentification OpenID Connect dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: 48c60878a6a58b2f4629768b81af894a741dab1c
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509799"
---
# <a name="web-sign-in-with-openid-connect-in-azure-active-directory-b2c"></a>Connexion web avec OpenID Connect dans Azure Active Directory B2C

OpenID Connect est un protocole d’authentification basé sur OAuth 2.0, qui peut être utilisé pour connecter de façon sécurisée des utilisateurs à des applications web. En utilisant l’implémentation d’OpenID Connect d’Azure Active Directory B2C (Azure AD B2C), vous pouvez sous-traiter l’inscription, la connexion et d’autres tâches de gestion des identités de vos applications web à Azure Active Directory. Ce guide explique comment procéder, indépendamment du langage. Il explique comment envoyer et recevoir des messages HTTP sans utiliser l’une de nos bibliothèques open source.

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) étend le protocole *d’autorisation* OAuth 2.0 pour l’utiliser en tant que protocole *d’authentification*. Ce protocole d’authentification vous permet d’effectuer une authentification unique. Il introduit le concept de *jeton d’ID*, qui permet au client de vérifier l’identité de l’utilisateur et d’obtenir des informations de base sur son profil.

Comme il étend OAuth 2.0, il permet aux applications d’acquérir de façon sécurisée des *jetons d’accès*. Vous pouvez utiliser des jetons d’accès pour accéder aux ressources qui sont sécurisées par un [serveur d’autorisation](protocols-overview.md). OpenID Connect est recommandé si vous créez une application web hébergée sur un serveur et accessible via un navigateur. Pour plus d’informations sur les jetons, consultez [Vue d’ensemble des jetons dans Azure Active Directory B2C](tokens-overview.md)

Azure AD B2C étend le protocole OpenID Connect standard pour proposer plus qu’une simple authentification et une simple autorisation. Il introduit le [paramètre de flux utilisateur](user-flow-overview.md), grâce auquel vous pouvez utiliser OpenID Connect pour ajouter des expériences utilisateur à votre application, telles que l’inscription, la connexion et la gestion des profils.

## <a name="send-authentication-requests"></a>Envoi de demandes d’authentification

Lorsque votre application web a besoin d’authentifier l’utilisateur et d’exécuter un flux utilisateur, elle peut diriger l’utilisateur vers le point de terminaison `/authorize`. L’utilisateur prend des mesures en fonction du flux utilisateur.

Dans cette demande, le client indique les autorisations qu’il a besoin d’acquérir de l’utilisateur dans le paramètre `scope` et spécifie le flux utilisateur à exécuter. Pour avoir une idée du fonctionnement de la requête, collez-la dans un navigateur et exécutez-la. Remplacez `{tenant}` par le nom de votre locataire. Remplacez `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` par l'ID de l'application que vous avez précédemment inscrite dans votre locataire. De même, remplacez le nom de la stratégie (`{policy}`) par le nom de stratégie présent dans votre locataire, par exemple `b2c_1_sign_in`.

```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code+id_token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=form_post
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
```

| Paramètre | Obligatoire | Description |
| --------- | -------- | ----------- |
| {tenant} | Oui | Nom de votre locataire Azure AD B2C |
| {policy} | Oui | Flux utilisateur à exécuter. Spécifiez le nom d'un flux utilisateur créé dans votre locataire Azure AD B2C. Par exemple : `b2c_1_sign_in`, `b2c_1_sign_up` ou `b2c_1_edit_profile`. |
| client_id | Oui | ID d’application que le [portail Azure](https://portal.azure.com/) a affecté à votre application. |
| nonce | Oui | Valeur incluse dans la demande (générée par l’application) qui est incluse dans le jeton d’ID résultant en tant que revendication. L’application peut ensuite vérifier cette valeur afin de contrer les attaques par relecture de jetons. La valeur est généralement une valeur unique aléatoire qui peut être utilisée pour identifier l’origine de la demande. |
| response_type | Oui | Doit inclure un jeton d’ID pour OpenID Connect. Si votre application web a également besoin de jetons pour appeler une API web, vous pouvez utiliser `code+id_token`. |
| scope | Oui | Une liste d’étendues séparées par des espaces. L’étendue `openid` indique une autorisation pour connecter l’utilisateur et obtenir des données relatives à l’utilisateur sous la forme de jetons d’ID. L’étendue `offline_access` est facultative pour les applications web. Elle indique que votre application a besoin d’un *jeton d’actualisation* pour un accès étendu aux ressources. |
| prompt | Non  | Type d’interaction utilisateur demandée. La seule valeur valide pour l’instant est `login`, qui oblige l’utilisateur à saisir ses informations d’identification sur cette demande. |
| redirect_uri | Non  | Paramètre `redirect_uri` de votre application, où les réponses d’authentification peuvent être envoyées et reçues par votre application. Il doit correspondre exactement à un des paramètres `redirect_uri` que vous avez inscrits dans le portail Azure, si ce n’est qu’il doit être codé dans une URL. |
| response_mode | Non  | Méthode utilisée pour renvoyer le code d’autorisation résultant à votre application. Il peut s’agir de `query`, `form_post` ou `fragment`.  Le mode de réponse `form_post` est recommandé pour une sécurité optimale. |
| state | Non  | Valeur incluse dans la demande qui est également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. Une valeur unique générée de manière aléatoire est généralement utilisée pour empêcher les falsifications de requête intersite. La valeur d’état est également utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la demande d’authentification, comme la page sur laquelle il était positionné. |

À ce stade, il est demandé à l’utilisateur de terminer le flux de travail. L’utilisateur peut avoir à entrer son nom d’utilisateur et son mot de passe, à se connecter avec une identité sociale ou à s’inscrire à l’annuaire. Il peut y avoir un nombre quelconque d’étapes supplémentaires en fonction de la façon dont le flux utilisateur est défini.

Une fois que l’utilisateur a terminé le flux utilisateur, une réponse est renvoyée à votre application dans le paramètre `redirect_uri` indiqué, à l’aide de la méthode spécifiée dans le paramètre `response_mode`. La réponse est la même pour chacun des cas ci-dessus, indépendamment du flux utilisateur.

Une réponse réussie utilisant `response_mode=fragment` se présenterait ainsi :

```http
GET https://aadb2cplayground.azurewebsites.net/#
id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=arbitrary_data_you_can_receive_in_the_response
```

| Paramètre | Description |
| --------- | ----------- |
| id_token | Jeton d'ID que l’application a demandé. Vous pouvez utiliser le jeton d'ID pour vérifier l’identité de l’utilisateur et démarrer une session avec lui. |
| code | Code d’autorisation que l’application a demandé, si vous avez utilisé `response_type=code+id_token`. L’application peut utiliser ce code d’autorisation pour demander un jeton d’accès pour une ressource cible. Les codes d’autorisation expirent généralement au bout d’environ 10 minutes. |
| state | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |

Les réponses d’erreur peuvent également être envoyées au paramètre `redirect_uri` pour que l’application puisse les traiter de façon appropriée :

```http
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| Paramètre | Description |
| --------- | ----------- |
| error | Code pouvant être utilisé pour classer les types d’erreurs qui se produisent. |
| error_description | Message d’erreur spécifique qui peut aider à identifier la cause racine d’une erreur d’authentification. |
| state | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |

## <a name="validate-the-id-token"></a>Validation du jeton d’ID

La réception d’un jeton d’ID à elle seule n’est pas suffisante pour authentifier l’utilisateur. Validez la signature du jeton d’ID et vérifiez les revendications figurant dans le jeton par rapport aux exigences de votre application. Azure AD B2C utilise les [jetons Web JSON (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) et le chiffrement de clés publiques pour signer les jetons et vérifier leur validité. Il existe de nombreuses bibliothèques open source pour valider les jetons JWT en fonction de votre langage préféré. Nous vous recommandons d’explorer ces options plutôt que d’implémenter votre propre logique de validation.

Azure AD B2C présente un point de terminaison de métadonnées OpenID Connect, qui permet à une application d’obtenir des informations sur Azure AD B2C au moment de l’exécution. Ces informations incluent les points de terminaison, le contenu des jetons et les clés de signature de jetons. Il existe un document de métadonnées JSON pour chaque flux utilisateur dans votre locataire B2C. Par exemple, le document de métadonnées pour le flux utilisateur `b2c_1_sign_in` dans `fabrikamb2c.onmicrosoft.com` se trouve à l’emplacement suivant :

```http
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/v2.0/.well-known/openid-configuration
```

Une des propriétés de ce document de configuration est `jwks_uri`, dont la valeur pour le même flux utilisateur serait :

```http
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/discovery/v2.0/keys
```

Pour déterminer quel flux utilisateur a été utilisé pour la signature d’un jeton d’ID (et d’où obtenir les métadonnées), deux options sont possibles. Tout d’abord, le nom du flux utilisateur est inclus dans la revendication `acr` du jeton d’ID. L’autre option consiste à coder le flux utilisateur dans la valeur du paramètre `state` lors de l’émission de la requête, puis à la décoder pour déterminer le flux utilisateur qui a été utilisé. Les 2 méthodes sont valides.

Après avoir acquis le document de métadonnées à partir du point de terminaison de métadonnées OpenID Connect, vous pouvez utiliser les clés publiques RSA 256 pour valider la signature du jeton d’ID. Il peut y avoir plusieurs clés répertoriées sur ce point de terminaison, chacune étant identifiée par une revendication `kid`. L’en-tête de ce jeton d’ID contient également une revendication `kid`, qui indique quelle clé a été utilisée pour signer le jeton d’ID.

Pour vérifier les jetons Azure AD B2C, vous devez générer la clé publique à l’aide de l’exposant (e) et du modulo (n). Vous devez déterminer comment procéder en conséquence dans votre langage de programmation respectif. La documentation officielle sur la génération de la clé publique en utilisant le protocole RSA est disponible ici : https://tools.ietf.org/html/rfc3447#section-3.1

Après avoir validé la signature du jeton d’ID, vous devez vérifier plusieurs revendications. Exemple :

- Validez la revendication `nonce` afin d’empêcher les attaques par relecture de jetons. Sa valeur doit correspondre à ce que vous avez spécifié dans la requête de connexion.
- Validez la revendication `aud` pour vérifier que le jeton d’ID a été émis pour votre application. Sa valeur doit correspondre à l’ID d’application de votre application.
- Validez les revendications `iat` et `exp` pour vérifier que le jeton d’ID n’a pas expiré.

Vous devez également effectuer plusieurs autres validations. Ces validations sont décrites en détail dans les [Spécifications principales d’OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html). En fonction de votre scénario, vous pouvez également valider des revendications supplémentaires. Voici quelques validations courantes :

- Vérifier que l’utilisateur/l’organisation s’est inscrit pour l’application.
- Vérifier que l’utilisateur dispose de l’autorisation/des privilèges appropriés.
- Vérifier qu'un certain niveau d'authentification a été atteint, par exemple en utilisant Azure AD Multi-Factor Authentication.

Une fois que vous avez validé le jeton d’ID, vous pouvez commencer une session avec l’utilisateur. Vous pouvez utiliser les revendications figurant dans le jeton d’ID pour obtenir des informations sur l’utilisateur dans votre application. Les utilisations de ces informations sont notamment l’affichage, les enregistrements et les autorisations.

## <a name="get-a-token"></a>Obtention d’un jeton

Si votre application web doit seulement exécuter des flux utilisateur, vous pouvez ignorer les quelques sections suivantes. Ces sections s’appliquent seulement aux applications web qui doivent effectuer des appels authentifiés à une API web et qui sont également protégées par Azure AD B2C.

Vous pouvez échanger le code d’autorisation que vous avez acquis (en utilisant `response_type=code+id_token`) contre un jeton sur la ressource souhaitée en envoyant une demande `POST` au point de terminaison `/token`. Dans Azure AD B2C, vous pouvez [demander des jetons d’accès pour les autres API](access-tokens.md#request-a-token) comme d’habitude en spécifiant leurs étendues dans la requête.

Vous pouvez également demander un jeton d’accès pour l’API web de back-end de votre application par convention visant à utiliser l’ID client de l’application comme étendue demandée (ce qui génère un jeton d’accès avec cet ID client comme « public ») :

```http
POST {tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1
Host: {tenant}.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob
```

| Paramètre | Obligatoire | Description |
| --------- | -------- | ----------- |
| {tenant} | Oui | Nom de votre locataire Azure AD B2C |
| {policy} | Oui | Le flux utilisateur qui a été utilisé pour obtenir le code d’autorisation. Vous ne pouvez pas utiliser un autre flux utilisateur dans cette demande. Ajoutez ce paramètre à la chaîne de requête, et non pas au corps POST. |
| client_id | Oui | ID d’application que le [portail Azure](https://portal.azure.com/) a affecté à votre application. |
| client_secret | Oui, dans Web Apps | Secret d'application qui a été généré dans le [portail Azure](https://portal.azure.com/). Les clés secrètes client sont utilisées dans ce flux pour les scénarios de type application web, dans lesquels le client peut les stocker de manière sécurisée. Dans les scénarios de type application native (client public), il n’est pas possible de stocker les clés secrètes client de manière sécurisée. Elles ne sont donc pas utilisées sur ce flux. Si vous utilisez une clé secrète client, modifiez-la régulièrement. |
| code | Oui | Code d’autorisation que vous avez acquis au début du flux utilisateur. |
| grant_type | Oui | Le type d’octroi, qui doit être `authorization_code` pour le flux de code d’autorisation. |
| redirect_uri | Oui | Le paramètre `redirect_uri` de l’application où vous avez reçu le code d’autorisation. |
| scope | Non  | Une liste d’étendues séparées par des espaces. L’étendue `openid` indique une autorisation pour connecter l’utilisateur et obtenir des données relatives à l’utilisateur sous la forme de paramètres id_token. Elle peut être utilisée afin d’obtenir des jetons pour l’API web de back-end de votre application, qui est représentée par le même ID d’application que le client. L’étendue `offline_access` indique que votre application a besoin d’un jeton d’actualisation pour l’accès étendu aux ressources. |

Un jeton de réponse de réussite se présente ainsi :

```json
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```

| Paramètre | Description |
| --------- | ----------- |
| not_before | Heure à laquelle le jeton est considéré comme valide, en heure epoch. |
| token_type | Valeur du type de jeton. `Bearer` est le seul type pris en charge. |
| access_token | Le jeton JWT signé que vous avez demandé. |
| scope | Étendues pour lesquelles le jeton est valide. |
| expires_in | Durée de validité du jeton d’accès (en secondes). |
| refresh_token | Un jeton d’actualisation OAuth 2.0. L’application peut utiliser ce jeton pour acquérir des jetons supplémentaires après l’expiration du jeton actuel. Les jetons d’actualisation peuvent être utilisés pour conserver l’accès aux ressources pendant des périodes prolongées. L’étendue `offline_access` doit avoir été utilisée dans les demandes d’autorisation et de jeton pour recevoir un jeton d’actualisation. |

Les réponses d’erreur se présentent comme ceci :

```json
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Paramètre | Description |
| --------- | ----------- |
| error | Code pouvant être utilisé pour classer les types d’erreurs qui se produisent. |
| error_description | Message qui peut aider à identifier la cause racine d’une erreur d’authentification. |

## <a name="use-the-token"></a>Utilisation du jeton

Un jeton d’accès étant acquis, vous pouvez maintenant l’utiliser dans les demandes effectuées à vos API web principales en l’incluant dans l’en-tête `Authorization` :

```http
GET /tasks
Host: mytaskwebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-token"></a>Actualisation du jeton

Les jetons d’ID expirent après un court délai. Actualisez les jetons après leur expiration pour continuer à accéder aux ressources. Vous pouvez actualiser un jeton en envoyant une nouvelle demande `POST` au point de terminaison `/token`. Cette fois-ci, spécifiez le paramètre `refresh_token` au lieu du paramètre `code` :

```http
POST {tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1
Host: {tenant}.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=openid offline_access&refresh_token=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob
```

| Paramètre | Obligatoire | Description |
| --------- | -------- | ----------- |
| {tenant} | Oui | Nom de votre locataire Azure AD B2C |
| {policy} | Oui | Flux utilisateur utilisé pour obtenir le jeton d’actualisation d’origine. Vous ne pouvez pas utiliser un autre flux utilisateur dans cette demande. Ajoutez ce paramètre à la chaîne de requête, et non pas au corps POST. |
| client_id | Oui | ID d’application que le [portail Azure](https://portal.azure.com/) a affecté à votre application. |
| client_secret | Oui, dans Web Apps | Secret d'application qui a été généré dans le [portail Azure](https://portal.azure.com/). Les clés secrètes client sont utilisées dans ce flux pour les scénarios de type application web, dans lesquels le client peut les stocker de manière sécurisée. Dans les scénarios de type application native (client public), les clés secrètes client ne peuvent pas être stockées en toute sécurité. Elle ne sont donc pas utilisées sur cet appel. Si vous utilisez une clé secrète client, modifiez-la régulièrement. |
| grant_type | Oui | Type d’octroi, qui doit être `refresh_token` pour cette partie du flux de code d’autorisation. |
| refresh_token | Oui | Jeton d’actualisation d’origine qui a été acquis dans la seconde partie du flux. L’étendue `offline_access` doit être utilisée dans les demandes d’autorisation et de jeton pour recevoir un jeton d’actualisation. |
| redirect_uri | Non  | Le paramètre `redirect_uri` de l’application où vous avez reçu le code d’autorisation. |
| scope | Non  | Une liste d’étendues séparées par des espaces. L’étendue `openid` indique une autorisation pour connecter l’utilisateur et obtenir des données relatives à l’utilisateur sous la forme de jetons d’ID. Elle peut être utilisée afin d’envoyer des jetons à l’API web de back-end de votre application, qui est représentée par le même ID d’application que le client. L’étendue `offline_access` indique que votre application a besoin d’un jeton d’actualisation pour l’accès étendu aux ressources. |

Un jeton de réponse de réussite se présente ainsi :

```json
{
    "not_before": "1442340812",
    "token_type": "Bearer",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "scope": "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
    "expires_in": "3600",
    "refresh_token": "AAQfQmvuDy8WtUv-sd0TBwWVQs1rC-Lfxa_NDkLqpg50Cxp5Dxj0VPF1mx2Z...",
}
```

| Paramètre | Description |
| --------- | ----------- |
| not_before | Heure à laquelle le jeton est considéré comme valide, en heure epoch. |
| token_type | Valeur du type de jeton. `Bearer` est le seul type pris en charge. |
| access_token | Jeton JWT signé qui a été demandé. |
| scope | Étendue pour laquelle le jeton est valide. |
| expires_in | Durée de validité du jeton d’accès (en secondes). |
| refresh_token | Un jeton d’actualisation OAuth 2.0. L’application peut utiliser ce jeton pour acquérir des jetons supplémentaires après l’expiration du jeton actuel. Les jetons d’actualisation peuvent être utilisés pour conserver l’accès aux ressources pendant des périodes prolongées. |

Les réponses d’erreur se présentent comme ceci :

```json
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Paramètre | Description |
| --------- | ----------- |
| error | Code pouvant être utilisé pour classer les types d’erreurs qui se produisent. |
| error_description | Message qui peut aider à identifier la cause racine d’une erreur d’authentification. |

## <a name="send-a-sign-out-request"></a>Envoi d’une demande de déconnexion

Quand vous souhaitez déconnecter l’utilisateur de l’application, il ne suffit pas de supprimer les cookies de l’application ou d’arrêter la session de l’utilisateur. Redirigez l’utilisateur vers Azure AD B2C pour le déconnecter. Si vous n’y parvenez pas, l’utilisateur peut être en mesure de se réauthentifier auprès de votre application, sans entrer à nouveau ses informations d’identification. Pour plus d’informations, consultez [Session Azure AD B2C](session-behavior.md).

Pour déconnecter l’utilisateur, redirigez-le vers le point de terminaison `end_session` répertorié dans le document de métadonnées OpenID Connect décrit précédemment :

```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/logout?post_logout_redirect_uri=https%3A%2F%2Fjwt.ms%2F
```

| Paramètre | Obligatoire | Description |
| --------- | -------- | ----------- |
| {tenant} | Oui | Nom de votre locataire Azure AD B2C |
| {policy} | Oui | Flux utilisateur que vous voulez utiliser pour déconnecter l’utilisateur de votre application. |
| id_token_hint| Non  | Jeton d’ID émis précédemment à transmettre au point de terminaison de déconnexion en tant qu’indicateur de la session authentifiée active de l’utilisateur final avec le client. `id_token_hint` vérifie que `post_logout_redirect_uri` est une URL de réponse inscrite dans les paramètres de votre application Azure AD B2C. Pour en savoir plus, consultez [Sécuriser la redirection de déconnexion](#secure-your-logout-redirect). |
| client_id | Non* | ID d’application que le [portail Azure](https://portal.azure.com/) a affecté à votre application.<br><br>\**Cela est nécessaire lors de l’utilisation de la configuration de l’authentification unique de l’isolation `Application` et _Demander le jeton d’ID_ dans la demande de déconnexion est défini sur `No`.* |
| post_logout_redirect_uri | Non  | URL vers laquelle l’utilisateur doit être redirigé après la déconnexion. Si elle n’est pas incluse, Azure AD B2C affiche un message générique à l’utilisateur. À moins de fournir un `id_token_hint`, vous ne devez pas inscrire cette URL en tant qu’URL de réponse dans les paramètres de votre application Azure AD B2C. |
| state | Non  | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |

### <a name="secure-your-logout-redirect"></a>Sécuriser la redirection de déconnexion

Après la déconnexion, l’utilisateur est redirigé vers l’URI spécifié dans le paramètre `post_logout_redirect_uri`, quelles que soient les URL de réponse qui ont été spécifiées pour l’application. Cependant, si un `id_token_hint` valide est transmis et que l’option **Exiger un jeton d’ID dans les demandes de déconnexion** est activée, Azure AD B2C vérifie que la valeur de `post_logout_redirect_uri` correspond à l’un des URI de redirection configurés de l’application avant d’effectuer la redirection. Si aucune URL de réponse correspondante n’a été configurée pour l’application, un message d’erreur s’affiche et l’utilisateur n’est pas redirigé.

Pour définir le jeton d’ID requis dans les demandes de déconnexion, consultez [Configurer le comportement de session dans Azure Active Directory B2C](session-behavior.md#secure-your-logout-redirect).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur une [session Azure AD B2C](session-behavior.md).
