---
title: Flux de code d’autorisation - Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment créer des applications web à l’aide d’Azure AD B2C et du protocole d’authentification OpenID Connect.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: 10444974cf31b95fccd2d11aef20bfd57fab7939
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92275286"
---
# <a name="oauth-20-authorization-code-flow-in-azure-active-directory-b2c"></a>Flux de code d’autorisation OAuth 2.0 dans Azure Active Directory B2C

Vous pouvez utiliser l’octroi de code d’autorisation OAuth 2.0 dans les applications qui sont installées sur un appareil pour accéder à des ressources protégées, comme des API web. Grâce à l’implémentation Azure Active Directory B2C (Azure AD B2C) d’OAuth 2.0, vous pouvez ajouter l’inscription, la connexion et d’autres tâches de gestion des identités à vos applications monopages, à vos applications mobiles et à vos applications de bureau. Cet article est indépendant du langage. Il décrit comment envoyer et recevoir des messages HTTP sans utiliser de bibliothèque open source. Dans la mesure du possible, nous vous recommandons d’utiliser les bibliothèques d’authentification Microsoft (MSAL) prises en charge. Voir les [exemples d’applications qui utilisent une MSAL](code-samples.md).

Le flux de code d’autorisation OAuth 2.0 est décrit dans la [section 4.1 des spécifications OAuth 2.0](https://tools.ietf.org/html/rfc6749). Vous pouvez l’utiliser pour les activités d’authentification et d’autorisation avec la plupart des [types d’applications](application-types.md), notamment les applications web, les applications monopages et les applications installées de façon native. Vous pouvez utiliser le flux de code d’autorisation OAuth 2.0 pour acquérir de manière sécurisée des jetons d’accès et des jetons d’actualisation pour vos applications, en vue d’accéder à des ressources sécurisées par un [serveur d’autorisation](protocols-overview.md).  Le jeton d’actualisation permet au client d’acquérir de nouveaux jetons d’accès (et d’actualisation) après l’expiration du jeton d’accès, soit généralement au bout d’une heure.

Cet article est axé sur le flux de code d’autorisation OAuth 2.0 pour les **clients publics** . Un client public est une application cliente qui ne peut pas être approuvée pour maintenir de façon sécurisée l'intégrité d’un mot de passe secret. Cela comprend les applications monopages, les applications mobiles, les applications de bureau et quasiment toutes les applications qui ne s’exécutent pas sur un serveur.

> [!NOTE]
> Pour ajouter la gestion d’identité à une application web à l’aide d’Azure AD B2C, utilisez [OpenID Connect](openid-connect.md) plutôt qu’OAuth 2.0.

Azure AD B2C étend les flux OAuth 2.0 standard pour proposer plus qu’une simple authentification et une simple autorisation. Il introduit le [flux utilisateur](user-flow-overview.md). Avec les flux d’utilisateur, vous pouvez utiliser OAuth 2.0 pour ajouter des expériences utilisateur à votre application, comme l’inscription, la connexion et la gestion des profils. Les fournisseurs d’identité qui utilisent le protocole OAuth 2.0 incluent [Amazon](identity-provider-amazon.md), [Azure Active Directory](identity-provider-azure-ad-single-tenant.md), [Facebook](identity-provider-facebook.md), [GitHub](identity-provider-github.md), [ Google](identity-provider-google.md) et [LinkedIn](identity-provider-linkedin.md).

Pour tester les requêtes HTTP de cet article :

1. Remplacez `{tenant}` par le nom de votre locataire Azure AD B2C.
1. Remplacez `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` par l’ID d’application d’une application que vous avez précédemment inscrite dans votre locataire Azure AD B2C.
1. Remplacez `{policy}` par le nom d'une stratégie que vous avez créée dans votre locataire, par exemple `b2c_1_sign_in`.

## <a name="redirect-uri-setup-required-for-single-page-apps"></a>Configuration de l’URI de redirection requise pour les applications monopages

Le flux de code d’autorisation pour des applications monopages requiert une configuration supplémentaire.  Suivez les instructions permettant de [créer votre application monopage](tutorial-register-spa.md) pour marquer correctement votre URI de redirection comme étant activé pour CORS. Pour mettre à jour un URI de redirection afin d’activer CORS, vous pouvez cliquer sur l’invite de migration dans la section « Web » de l’onglet **Authentification** d’ **Inscriptions d’applications** . Vous pouvez également ouvrir l’ **éditeur de manifeste d’inscriptions d’applications** et définir le champ `type` pour votre URI de redirection sur `spa` dans la section `replyUrlsWithType`.

Le type de redirection `spa` est rétrocompatible avec le flux implicite. Les applications qui utilisent actuellement le flux implicite pour obtenir des jetons peuvent passer au type d’URI de redirection `spa` sans problème et continuer à utiliser le flux implicite.

## <a name="1-get-an-authorization-code"></a>1. Obtention d’un code d’autorisation
Le flux de code d'autorisation commence par le client dirigeant l'utilisateur vers le point de terminaison `/authorize` . Il s’agit de la partie interactive du flux, dans laquelle l’utilisateur agit réellement. Dans cette demande, le client indique dans le paramètre `scope` les autorisations qu’il doit obtenir auprès de l’utilisateur. Les trois exemples suivants (avec sauts de ligne pour faciliter la lecture) utilisent chacun un flux d’utilisateur différent.


```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=code
&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob
&response_mode=query
&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&code_challenge=YTFjNjI1OWYzMzA3MTI4ZDY2Njg5M2RkNmVjNDE5YmEyZGRhOGYyM2IzNjdmZWFhMTQ1ODg3NDcxY2Nl
&code_challenge_method=S256
```

| Paramètre | Requis ? | Description |
| --- | --- | --- |
|{tenant}| Obligatoire | Nom de votre locataire Azure AD B2C|
| {policy} | Obligatoire | Flux utilisateur à exécuter. Spécifiez le nom d'un flux utilisateur créé dans votre locataire Azure AD B2C. Par exemple : `b2c_1_sign_in`, `b2c_1_sign_up` ou `b2c_1_edit_profile`. |
| client_id |Obligatoire |ID d’application affecté à votre application dans le [portail Azure](https://portal.azure.com). |
| response_type |Obligatoire |Le type de réponse, qui doit inclure `code` pour le flux de code d’autorisation. |
| redirect_uri |Obligatoire |URI de redirection de votre application, où votre application envoie et reçoit des réponses d’authentification. Il doit correspondre exactement à l’un des URI de redirection que vous avez enregistrés dans le portail, sauf qu’il doit être encodé au format URL. |
| scope |Obligatoire |Une liste d’étendues séparées par des espaces. Une valeur d’étendue unique indique à Azure Active Directory (Azure AD) les deux autorisations qui sont demandées. Utiliser l’ID de client comme étendue indique que votre application a besoin d’un jeton d’accès qui peut être utilisé avec votre propre service ou API web, représenté par le même ID de client.  L’étendue `offline_access` indique que votre application a besoin d’un jeton d’actualisation pour un accès durable aux ressources. Vous pouvez également utiliser l’étendue `openid` pour demander un jeton d’ID à partir d’Azure AD B2C. |
| response_mode |Recommandé |Méthode à utiliser pour renvoyer le code d’autorisation résultant à votre application. Il peut s’agir de `query`, `form_post` ou `fragment`. |
| state |Recommandé |Une valeur incluse dans la requête peut être une chaîne de n’importe quel contenu que vous voulez utiliser. Généralement, une valeur unique générée de manière aléatoire est utilisée, pour empêcher les attaques par falsification de requête intersites. L’état est également utilisé pour coder les informations sur l’état de l’utilisateur dans l’application avant la demande d’authentification. Par exemple, la page dans laquelle l’utilisateur se trouvait ou le flux d’utilisateur en cours d’exécution. |
| prompt |Facultatif |Type d’interaction utilisateur requis. Actuellement, la seule valeur valide est `login`, ce qui oblige l’utilisateur à entrer ses informations d’identification sur cette demande. L’authentification unique ne prendra pas effet. |
| code_challenge  | recommandé/obligatoire | Utilisé pour sécuriser l’octroi du code d’autorisation par PKCE (Proof Key for Code Exchange). Obligatoire si `code_challenge_method` est inclus. Pour plus d'informations, consultez le [RFC PKCE](https://tools.ietf.org/html/rfc7636). Cette option est désormais recommandée pour tous les types d’applications (natives, monopages et confidentielles clients comme des applications web). | 
| `code_challenge_method` | recommandé/obligatoire | La méthode utilisée pour encoder le `code_verifier` pour le paramètre `code_challenge`. Ce *DEVRAIT* être `S256`, mais la spécification autorise l’utilisation de `plain` si, pour une raison quelconque, le client ne peut pas prendre en charge SHA256. <br/><br/>S’il est exclu, `code_challenge` est censé être dans un texte en clair si `code_challenge` est inclus. La plateforme d’identités Microsoft prend en charge `plain` et `S256`. Pour plus d'informations, consultez le [RFC PKCE](https://tools.ietf.org/html/rfc7636). Cela est obligatoire pour les [applications monopages utilisant le flux de code d’autorisation](tutorial-register-spa.md).|

À ce stade, il est demandé à l’utilisateur d’effectuer le workflow du flux utilisateur. Il est possible que l’utilisateur doive entrer son nom d’utilisateur et son mot de passe, se connecter avec une identité sociale, s’inscrire à l’annuaire, etc. Les actions de l’utilisateur dépendent de la façon dont le flux d’utilisateur est défini.

Une fois que l’utilisateur a terminé le flux d’utilisateur, Azure AD retourne une réponse à votre application avec la valeur que vous avez utilisée pour `redirect_uri`. Il utilise la méthode spécifiée dans le paramètre `response_mode`. La réponse est exactement la même pour chacun des scénarios d’actions utilisateur, indépendamment du flux d’utilisateur exécuté.

Une réponse correcte utilisant `response_mode=query` se présente ainsi :

```http
GET urn:ietf:wg:oauth:2.0:oob?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...        // the authorization_code, truncated
&state=arbitrary_data_you_can_receive_in_the_response                // the value provided in the request
```

| Paramètre | Description |
| --- | --- |
| code |Le code d’autorisation demandé par l’application. L’application peut utiliser ce code d’autorisation afin de demander un jeton d’accès pour une ressource cible. Les codes d’autorisation ont des durées de vie très courtes. Généralement, ils expirent au bout de 10 minutes. |
| state |Consultez la description complète dans le tableau de la section précédente. Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |

Les réponses d’erreur peuvent également être envoyées à l’URI de redirection, pour que l’application puisse les traiter de façon appropriée :

```http
GET urn:ietf:wg:oauth:2.0:oob?
error=access_denied
&error_description=The+user+has+cancelled+entering+self-asserted+information
&state=arbitrary_data_you_can_receive_in_the_response
```

| Paramètre | Description |
| --- | --- |
| error |Chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreurs se produisant. Vous pouvez également utiliser la chaîne pour réagir aux erreurs. |
| error_description |Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |
| state |Consultez la description complète dans le tableau précédent. Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |

## <a name="2-get-an-access-token"></a>2. Obtention d’un jeton d’accès
Un code d’autorisation étant acquis, vous pouvez maintenant échanger `code` contre un jeton sur la ressource prévue en envoyant une demande POST au point de terminaison `/token`. Dans Azure AD B2C, vous pouvez [demander des jetons d’accès pour les autres API](access-tokens.md#request-a-token) comme d’habitude en spécifiant leurs étendues dans la requête.

Vous pouvez également demander un jeton d’accès pour l’API web de back-end de votre application par convention visant à utiliser l’ID client de l’application comme étendue demandée (ce qui génère un jeton d’accès avec cet ID client comme « public ») :

```http
POST https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1

Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob&code_verifier=ThisIsntRandomButItNeedsToBe43CharactersLong 
```

| Paramètre | Requis ? | Description |
| --- | --- | --- |
|{tenant}| Obligatoire | Nom de votre locataire Azure AD B2C|
|{policy}| Obligatoire| Le flux utilisateur qui a été utilisé pour obtenir le code d’autorisation. Vous ne pouvez pas utiliser un autre flux utilisateur dans cette demande. |
| client_id |Obligatoire |ID d’application affecté à votre application dans le [portail Azure](https://portal.azure.com).|
| client_secret | Oui, dans Web Apps | Secret d'application qui a été généré dans le [portail Azure](https://portal.azure.com/). Les clés secrètes client sont utilisées dans ce flux pour les scénarios de type application web, dans lesquels le client peut les stocker de manière sécurisée. Dans les scénarios de type application native (client public), les clés secrètes client ne peuvent pas être stockées de façon sécurisée et ne sont donc pas utilisées dans cet appel. Si vous utilisez une clé secrète client, modifiez-la régulièrement. |
| grant_type |Obligatoire |Type d’autorisation. Pour le flux de code d’autorisation, le type d’autorisation doit être `authorization_code`. |
| scope |Recommandé |Une liste d’étendues séparées par des espaces. Une valeur d’étendue unique indique à Azure AD les deux autorisations qui sont demandées. Utiliser l’ID de client comme étendue indique que votre application a besoin d’un jeton d’accès qui peut être utilisé avec votre propre service ou API web, représenté par le même ID de client.  L’étendue `offline_access` indique que votre application a besoin d’un jeton d’actualisation pour un accès durable aux ressources.  Vous pouvez également utiliser l’étendue `openid` pour demander un jeton d’ID à partir d’Azure AD B2C. |
| code |Obligatoire |Code d’autorisation acquis dans le premier tronçon du flux. |
| redirect_uri |Obligatoire |URI de redirection de l’application où vous avez reçu le code d’autorisation. |
| code_verifier | recommandé | Le même code_verifier utilisé pour obtenir le authorization_code. Obligatoire si PKCE est utilisé dans la requête d’octroi du code d’autorisation. Pour plus d'informations, consultez le [RFC PKCE](https://tools.ietf.org/html/rfc7636). |

Un jeton de réponse correct se présente ainsi :

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
| --- | --- |
| not_before |Heure à laquelle le jeton est considéré comme valide, en heure epoch. |
| token_type |Valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le jeton porteur. |
| access_token |JSON Web Token (JWT) signé que vous avez demandé. |
| scope |Étendues de validité du jeton. Vous pouvez également utiliser des étendues pour mettre en cache des jetons pour une utilisation ultérieure. |
| expires_in |La durée de validité du jeton (en secondes). |
| refresh_token |Un jeton d’actualisation OAuth 2.0. L’application peut utiliser ce jeton pour acquérir des jetons supplémentaires après l’expiration du jeton actuel. Les jetons d’actualisation sont de longue durée. Vous pouvez les utiliser pour conserver l’accès aux ressources pendant de longues périodes. Pour plus d’informations, consultez la [référence sur les jetons Azure AD B2C](tokens-overview.md). |

Les réponses d’erreur ressemblent à ceci :

```json
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Paramètre | Description |
| --- | --- |
| error |Chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreurs se produisant. Vous pouvez également utiliser la chaîne pour réagir aux erreurs. |
| error_description |Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |

## <a name="3-use-the-token"></a>3. Utilisation du jeton
Un jeton d’accès étant acquis, vous pouvez maintenant l’utiliser dans les demandes effectuées à vos API web principales en l’incluant dans l’en-tête `Authorization` :

```http
GET /tasks
Host: mytaskwebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="4-refresh-the-token"></a>4. Actualisation du jeton
Les jetons d’accès et les jetons d’ID ont une courte durée de vie. Après leur expiration, vous devez les actualiser pour continuer à accéder aux ressources. Pour cela, envoyez une autre demande POST au point de terminaison `/token`. Cette fois, fournissez l’élément `refresh_token` au lieu de l’élément `code` :

```http
POST https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/token HTTP/1.1

Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6&scope=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access&refresh_token=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&redirect_uri=urn:ietf:wg:oauth:2.0:oob
```

| Paramètre | Requis ? | Description |
| --- | --- | --- |
|{tenant}| Obligatoire | Nom de votre locataire Azure AD B2C|
|{policy} |Obligatoire |Flux utilisateur utilisé pour obtenir le jeton d’actualisation d’origine. Vous ne pouvez pas utiliser un autre flux utilisateur dans cette demande. |
| client_id |Obligatoire |ID d’application affecté à votre application dans le [portail Azure](https://portal.azure.com). |
| client_secret | Oui, dans Web Apps | Secret d'application qui a été généré dans le [portail Azure](https://portal.azure.com/). Les clés secrètes client sont utilisées dans ce flux pour les scénarios de type application web, dans lesquels le client peut les stocker de manière sécurisée. Dans les scénarios de type application native (client public), les clés secrètes client ne peuvent pas être stockées de façon sécurisée et ne sont donc pas utilisées dans cet appel. Si vous utilisez une clé secrète client, modifiez-la régulièrement. |
| grant_type |Obligatoire |Type d’autorisation. Pour ce tronçon du flux de code d’autorisation, le type d’autorisation doit être `refresh_token`. |
| scope |Recommandé |Une liste d’étendues séparées par des espaces. Une valeur d’étendue unique indique à Azure AD les deux autorisations qui sont demandées. Utiliser l’ID de client comme étendue indique que votre application a besoin d’un jeton d’accès qui peut être utilisé avec votre propre service ou API web, représenté par le même ID de client.  L’étendue `offline_access` indique que votre application a besoin d’un jeton d’actualisation pour un accès de longue durée aux ressources.  Vous pouvez également utiliser l’étendue `openid` pour demander un jeton d’ID à partir d’Azure AD B2C. |
| redirect_uri |Facultatif |URI de redirection de l’application où vous avez reçu le code d’autorisation. |
| refresh_token |Obligatoire |Jeton d’actualisation d’origine que vous avez acquis dans le second tronçon du flux. |

Un jeton de réponse correct se présente ainsi :

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
| --- | --- |
| not_before |Heure à laquelle le jeton est considéré comme valide, en heure epoch. |
| token_type |Valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le jeton porteur. |
| access_token |Jeton JWT signé que vous avez demandé. |
| scope |Étendues de validité du jeton. Vous pouvez également utiliser les étendues pour mettre en cache des jetons pour une utilisation ultérieure. |
| expires_in |La durée de validité du jeton (en secondes). |
| refresh_token |Un jeton d’actualisation OAuth 2.0. L’application peut utiliser ce jeton pour acquérir des jetons supplémentaires après l’expiration du jeton actuel. Les jetons d’actualisation sont durables, et peuvent être utilisés pour conserver l’accès aux ressources pendant des périodes prolongées. Pour plus d’informations, consultez la [référence sur les jetons Azure AD B2C](tokens-overview.md). |

Les réponses d’erreur ressemblent à ceci :

```json
{
    "error": "access_denied",
    "error_description": "The user revoked access to the app.",
}
```

| Paramètre | Description |
| --- | --- |
| error |Chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreurs qui se produisent. Vous pouvez également utiliser la chaîne pour réagir aux erreurs. |
| error_description |Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |

## <a name="use-your-own-azure-ad-b2c-directory"></a>Utiliser votre propre annuaire Azure AD B2C
Pour essayer vous-même ces demandes, effectuez les étapes suivantes. Remplacez les exemples de valeurs que nous utilisons dans cet article par vos propres valeurs.

1. [Créez un annuaire Azure AD B2C](tutorial-create-tenant.md). Utilisez le nom de votre annuaire dans les demandes.
2. [Créez une application](tutorial-register-applications.md) pour obtenir un ID d’application et un URI de redirection. Incluez un client natif dans votre application.
3. [Créez vos flux d’utilisateurs](user-flow-overview.md) pour obtenir vos noms de flux utilisateur.
