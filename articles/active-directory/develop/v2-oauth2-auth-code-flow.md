---
title: Plateforme d’identité Microsoft et flux de code d’autorisation OAuth 2.0 | Azure
titleSuffix: Microsoft identity platform
description: Créez des applications web à l’aide de l’implémentation de la plateforme d’identités Microsoft du protocole d’authentification OAuth 2.0.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/14/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 6648cfb717ade4b842e8ff470a46bf744b630363
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88612314"
---
# <a name="microsoft-identity-platform-and-oauth-20-authorization-code-flow"></a>Plateforme d’identités Microsoft et flux de code d’autorisation OAuth

L'octroi d'un code d'autorisation OAuth 2.0 peut servir dans les applications qui sont installées sur un périphérique pour accéder à des ressources protégées, comme des API Web. Avec la mise en œuvre de la plateforme d’identités Microsoft d’OAuth 2.0, vous pouvez ajouter une connexion et un accès API à vos applications mobiles et de bureau.

Cet article explique comment programmer directement par rapport au protocole dans votre application en utilisant n’importe quel langage.  Dans la mesure du possible, nous vous recommandons d’utiliser les bibliothèques d’authentification Microsoft (MSAL) prises en charge au lieu d’[acquérir des jetons et d’appeler des API web sécurisées](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Jetez également un coup d’œil aux [exemples d’applications qui utilisent MSAL](sample-v2-code.md).

Le flux de code d’autorisation OAuth 2.0 est décrit dans la [section 4.1 des spécifications OAuth 2.0](https://tools.ietf.org/html/rfc6749). Il est utilisé pour exécuter des activités d’authentification et d’autorisation dans la majorité des types d’applications, notamment les [applications monopages](v2-app-types.md#single-page-apps-javascript), les [applications web](v2-app-types.md#web-apps) et les [applications installées de façon native](v2-app-types.md#mobile-and-native-apps). Le flux permet aux applications d’acquérir en toute sécurité des jetons d’accès (access_tokens) utilisables pour accéder à des ressources sécurisées par le point de terminaison de la plateforme d’identité Microsoft, ainsi que des jetons d’actualisation pour obtenir des jetons d’accès supplémentaires, ainsi que des jetons d’identification pour l’utilisateur connecté.

## <a name="protocol-diagram"></a>Schéma de protocole

À un niveau élevé, le flux d’authentification complet pour une application ressemble à ceci :

![Flux de code d’authentification OAuth](./media/v2-oauth2-auth-code-flow/convergence-scenarios-native.svg)

## <a name="redirect-uri-setup-required-for-single-page-apps"></a>Configuration de l’URI de redirection requise pour les applications monopages

Le flux de code d’autorisation pour des applications monopages requiert une configuration supplémentaire.  Suivez les instructions permettant de [créer votre application monopage](scenario-spa-app-registration.md#redirect-uri-msaljs-20-with-auth-code-flow) pour marquer correctement votre URI de redirection comme étant activé pour CORS. Afin de mettre à jour un URI de redirection existant pour activer CORS, ouvrez l’éditeur de manifeste et définissez le champ `type` de votre URI de redirection sur `spa` dans la section `replyUrlsWithType`. Vous pouvez également cliquer sur l’URI de redirection dans la section « Web » de l’onglet Authentification, puis sélectionner les URI vers lesquels vous souhaitez migrer en utilisant le flux de code d’autorisation.

Le type de redirection `spa` est rétrocompatible avec le flux implicite. Les applications qui utilisent actuellement le flux implicite pour obtenir des jetons peuvent passer au type d’URI de redirection `spa` sans problème et continuer à utiliser le flux implicite.

Si vous tentez d’utiliser le flux de code d’autorisation et voyez le message d’erreur suivant :

`access to XMLHttpRequest at 'https://login.microsoftonline.com/common/v2.0/oauth2/token' from origin 'yourApp.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

Vous devez consulter votre inscription d’application et mettre à jour l’URI de redirection de votre application pour le type `spa`.

## <a name="request-an-authorization-code"></a>Demander un code d’autorisation

Le flux de code d'autorisation commence par le client dirigeant l'utilisateur vers le point de terminaison `/authorize` . Dans cette demande, le client demande les autorisations `openid`, `offline_access` et `https://graph.microsoft.com/mail.read ` de l’utilisateur.  Certaines autorisations sont limitées par l'administrateur, par exemple l'écriture de données dans le répertoire d'une organisation à l'aide de `Directory.ReadWrite.All`. Si votre application requiert l’accès à l’une de ces autorisations d’un utilisateur de l’organisation, ce dernier recevra un message d’erreur indiquant qu’il n’est pas autorisé à donner son consentement pour les permissions de votre application. Pour demander l'accès aux étendues limitées par l'administrateur, vous devez vous adresser directement à un administrateur d'entreprise.  Pour plus d'informations, consultez [Autorisations limitées par l'administrateur](v2-permissions-and-consent.md#admin-restricted-permissions).

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&state=12345
&code_challenge=YTFjNjI1OWYzMzA3MTI4ZDY2Njg5M2RkNmVjNDE5YmEyZGRhOGYyM2IzNjdmZWFhMTQ1ODg3NDcxY2Nl
&code_challenge_method=S256
```

> [!TIP]
> Cliquez sur le lien ci-dessous pour exécuter cette requête ! Une fois que vous êtes connecté, votre navigateur doit être redirigé vers `https://localhost/myapp/` avec une valeur `code` dans la barre d’adresse.
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=query&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fmail.read&state=12345" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| Paramètre    | Obligatoire ou facultatif | Description |
|--------------|-------------|--------------|
| `tenant`    | Obligatoire    | La valeur `{tenant}` dans le chemin d’accès de la requête peut être utilisée pour contrôler les utilisateurs qui peuvent se connecter à l’application. Les valeurs autorisées sont `common`, `organizations`, `consumers` et les identificateurs du client. Pour plus d’informations, consultez les [principes de base du protocole](active-directory-v2-protocols.md#endpoints).  |
| `client_id`   | Obligatoire    | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908).  |
| `response_type` | Obligatoire    | Doit inclure `code` pour le flux de code d’autorisation.       |
| `redirect_uri`  | Obligatoire | L’URI de redirection de votre application, vers lequel votre application peut envoyer et recevoir des réponses d’authentification. Il doit correspondre exactement à l’un des URI de redirection enregistrés dans le portail, auquel s’ajoute le codage dans une URL. Pour les applications natives et mobiles, vous devez utiliser la valeur par défaut `https://login.microsoftonline.com/common/oauth2/nativeclient`.   |
| `scope`  | Obligatoire    | Liste séparée par des espaces d’ [étendues](v2-permissions-and-consent.md) pour lesquelles vous souhaitez que l’utilisateur donne son consentement.  Pour le tronçon `/authorize` de la requête, cela peut couvrir plusieurs ressources, ce qui permet à votre application d’obtenir le consentement pour les multiples API que vous souhaitez appeler. |
| `response_mode`   | recommandé | Spécifie la méthode à utiliser pour envoyer le jeton résultant à votre application. Il peut s'agir d'une des méthodes suivantes :<br/><br/>- `query`<br/>- `fragment`<br/>- `form_post`<br/><br/>`query` fournit le code en tant que paramètre d’une chaîne de requête sur votre URI de redirection. Si vous demandez un jeton ID à l’aide du flux implicite, vous ne pouvez pas utiliser `query` comme indiqué dans les [spécifications OpenID](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations). Si vous ne demandez que le code, vous pouvez utiliser `query`, `fragment` ou `form_post`. `form_post` exécute une requête POST contenant le code pour votre URI de redirection. Pour plus d’informations, voir [Protocole OpenID Connect](../azuread-dev/v1-protocols-openid-connect-code.md).  |
| `state`                 | recommandé | Une valeur incluse dans la requête, qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. Une valeur unique générée de manière aléatoire est généralement utilisée pour [empêcher les falsifications de requête intersite](https://tools.ietf.org/html/rfc6749#section-10.12). La valeur peut également encoder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou la vue sur laquelle il était. |
| `prompt`  | facultatif    | Indique le type d’interaction utilisateur requis. Les seules valeurs valides pour l’instant sont `login`, `none` et `consent`.<br/><br/>- `prompt=login` oblige l'utilisateur à saisir ses informations d'identification lors de cette requête, annulant de fait l'authentification unique.<br/>Avec - `prompt=none`, c’est le comportement inverse. Cette valeur vous garantit qu’aucune invite interactive d’aucune sorte n’est présentée à l’utilisateur. Si la demande ne peut pas être exécutée en mode silencieux au moyen d’une authentification unique, le point de terminaison de la plateforme d’identités Microsoft renvoie une erreur `interaction_required`.<br/>- `prompt=consent` déclenche l'affichage de la boîte de dialogue de consentement OAuth après la connexion de l'utilisateur, afin de lui demander d'octroyer des autorisations d'accès à l'application.<br/>- `prompt=select_account` interrompt l’authentification unique en fournissant une expérience de sélection de compte répertoriant tous les comptes dans la session ou tout compte mémorisé, ou une option pour choisir d’utiliser un autre compte.<br/> |
| `login_hint`  | facultatif    | Peut être utilisé pour remplir au préalable le champ réservé au nom d’utilisateur/à l’adresse électronique de la page de connexion de l’utilisateur si vous connaissez déjà son nom d’utilisateur. Les applications utilisent souvent ce paramètre au cours de la réauthentification, après avoir extrait le nom d’utilisateur à partir d’une connexion précédente à l’aide de la revendication `preferred_username`.   |
| `domain_hint`  | facultatif    | S’il est inclus, ce paramètre ignore le processus de découverte par e-mail auquel l’utilisateur doit se soumettre sur la page de connexion, ce qui améliore légèrement l’expérience utilisateur, par exemple, en le dirigeant vers son fournisseur d’identité fédéré. Les applications utilisent souvent ce paramètre au cours de la réauthentification, en extrayant la revendication `tid` à partir d’une connexion précédente. Si la revendication `tid` est définie sur la valeur `9188040d-6c67-4c5b-b112-36a304b66dad`, vous devez utiliser `domain_hint=consumers`. Sinon, utilisez `domain_hint=organizations`.  |
| `code_challenge`  | recommandé/obligatoire | Utilisé pour sécuriser l’octroi du code d’autorisation par PKCE (Proof Key for Code Exchange). Obligatoire si `code_challenge_method` est inclus. Pour plus d'informations, consultez le [RFC PKCE](https://tools.ietf.org/html/rfc7636). Cette option est désormais recommandée pour tous les types d’applications (natives, monopages et confidentielles clients comme des applications web). |
| `code_challenge_method` | recommandé/obligatoire | La méthode utilisée pour encoder le `code_verifier` pour le paramètre `code_challenge`. Ce *DEVRAIT* être `S256`, mais la spécification autorise l’utilisation de `plain` si, pour une raison quelconque, le client ne peut pas prendre en charge SHA256. <br/><br/>S’il est exclu, `code_challenge` est censé être dans un texte en clair si `code_challenge` est inclus. La plateforme d’identités Microsoft prend en charge `plain` et `S256`. Pour plus d'informations, consultez le [RFC PKCE](https://tools.ietf.org/html/rfc7636). Cela est obligatoire pour les [applications monopages utilisant le flux de code d’autorisation](reference-third-party-cookies-spas.md).|


À ce stade, l’utilisateur est invité à saisir ses informations d’identification et à exécuter l’authentification. Le point de terminaison de la plateforme d’identités Microsoft s’assure également que l’utilisateur a accepté les autorisations indiquées dans le paramètre de requête `scope`. Si l’utilisateur n’a pas accepté l’une ou plusieurs de ces autorisations, le point de terminaison lui demande de corriger ce manquement. Les détails sur les [autorisations, les consentements et les applications mutualisées sont disponibles ici](v2-permissions-and-consent.md).

Une fois que l’utilisateur a procédé à l’authentification et accordé son consentement, le point de terminaison de la plateforme d’identités Microsoft renvoie une réponse à votre application à l’URI `redirect_uri` indiqué à l’aide de la méthode spécifiée dans le paramètre `response_mode`.

#### <a name="successful-response"></a>Réponse correcte

Une réponse correcte utilisant `response_mode=query` se présente ainsi :

```HTTP
GET https://login.microsoftonline.com/common/oauth2/nativeclient?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=12345
```

| Paramètre | Description  |
|-----------|--------------|
| `code` | Le code d’autorisation demandé par l’application. L’application peut utiliser ce code d’autorisation pour demander un jeton d’accès pour la ressource cible. Les codes d’autorisation présentent une durée de vie courte. Généralement, ils expirent au bout de 10 minutes. |
| `state` | Si un paramètre d’état est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs d’état de la demande et de la réponse sont identiques. |

Vous pouvez également recevoir un jeton d’accès et un jeton d’ID si vous en demandez un et avez activé l’octroi implicite dans votre inscription d’application.  Cette opération est parfois appelée « flux hybride » et utilisée par des infrastructures telles qu’ASP.NET.

#### <a name="error-response"></a>Réponse d’erreur

Les réponses d’erreur peuvent également être envoyées à l’élément `redirect_uri` , de manière à ce que l’application puisse les traiter de manière appropriée :

```HTTP
GET https://login.microsoftonline.com/common/oauth2/nativeclient?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| Paramètre | Description  |
|----------|------------------|
| `error`  | Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur d’authentification. |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>Codes d’erreur pour les erreurs de point de terminaison d’autorisation

Le tableau suivant décrit les différents codes d’erreur qui peuvent être retournés dans le paramètre `error` de la réponse d’erreur.

| Code d'erreur  | Description    | Action du client   |
|-------------|----------------|-----------------|
| `invalid_request` | Erreur de protocole, tel qu’un paramètre obligatoire manquant. | Corrigez l’erreur, puis envoyez à nouveau la demande. Il s’agit d’une erreur de développement généralement détectée lors des tests initiaux. |
| `unauthorized_client` | L’application cliente n’est pas autorisée à demander un code d’autorisation. | Cette erreur se produit généralement lorsque l’application cliente n’est pas inscrite dans Azure AD ou n’est pas ajoutée au client Azure AD de l’utilisateur. L’application peut proposer à l’utilisateur des instructions pour installer l’application et l’ajouter à Azure AD. |
| `access_denied`  | Le propriétaire de la ressource n’a pas accordé son consentement.  | L’application cliente peut avertir l’utilisateur qu’elle ne peut pas continuer sans son consentement. |
| `unsupported_response_type` | Le serveur d’autorisation ne prend pas en charge le type de réponse dans la demande. | Corrigez l’erreur, puis envoyez à nouveau la demande. Il s’agit d’une erreur de développement généralement détectée lors des tests initiaux.  |
| `server_error`  | Le serveur a rencontré une erreur inattendue.| Relancez la requête. Ces erreurs peuvent résulter de conditions temporaires. L’application cliente peut expliquer à l’utilisateur que sa réponse est reportée en raison d’une erreur temporaire. |
| `temporarily_unavailable`   | Le serveur est temporairement trop occupé pour traiter la demande. | Relancez la requête. L’application cliente peut expliquer à l’utilisateur que sa réponse est reportée en raison d’une condition temporaire. |
| `invalid_resource`  | La ressource cible n’est pas valide car elle n’existe pas, Azure AD ne la trouve pas ou elle n’est pas configurée correctement. | Cette erreur indique que la ressource, si elle existe, n’a pas été configurée dans l’abonné. L’application peut proposer à l’utilisateur des instructions pour installer l’application et l’ajouter à Azure AD. |
| `login_required` | Trop d’utilisateurs ou aucun utilisateur trouvé | Le client a demandé l’authentification en mode silencieux (`prompt=none`), mais un utilisateur unique est introuvable. Cela peut signifier que plusieurs utilisateurs sont actifs dans la session, ou qu’il n’y a aucun utilisateur. Cela prend en compte le client choisi (par exemple, si vous disposez de 2 comptes Azure AD actifs et d’un compte Microsoft, et que `consumers` est choisi, l’authentification en mode silencieux fonctionne). |
| `interaction_required` | La demande nécessite une interaction utilisateur. | Une étape d’authentification ou un consentement supplémentaire est nécessaire. Relancez la requête sans `prompt=none`. |

## <a name="request-an-access-token"></a>Demander un jeton d’accès

Maintenant que vous avez acquis un authorization_code et que l'utilisateur vous a octroyé une autorisation, vous pouvez échanger `code` contre un `access_token` sur la ressource souhaitée. Pour ce faire, envoyez une requête `POST` au point de terminaison  `/token` :

```HTTP
// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&grant_type=authorization_code
&code_verifier=ThisIsntRandomButItNeedsToBe43CharactersLong 
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps. This secret needs to be URL-Encoded.
```

> [!TIP]
> Essayez d'exécuter cette requête dans Postman ! N’oubliez pas de remplacer le `code`[![Essayez d’exécuter cette requête dans Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

| Paramètre  | Obligatoire ou facultatif | Description     |
|------------|-------------------|----------------|
| `tenant`   | Obligatoire   | La valeur `{tenant}` dans le chemin d’accès de la requête peut être utilisée pour contrôler les utilisateurs qui peuvent se connecter à l’application. Les valeurs autorisées sont `common`, `organizations`, `consumers` et les identificateurs du client. Pour plus d’informations, consultez les [principes de base du protocole](active-directory-v2-protocols.md#endpoints).  |
| `client_id` | Obligatoire  | L’ID (client) d’application attribué à votre application par la page [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `grant_type` | Obligatoire   | Doit être `authorization_code` pour le flux de code d'autorisation.   |
| `scope`      | facultatif   | Liste d’étendues séparées par des espaces. Les étendues doivent toutes être issues d’une seule ressource, ainsi que les étendues OIDC (`profile`, `openid`, `email`). Pour obtenir une explication plus détaillée des étendues, consultez les [autorisations, consentements et étendues](v2-permissions-and-consent.md). Il s’agit d’une extension Microsoft au flux de code d’autorisation, conçue pour permettre aux applications de déclarer la ressource pour laquelle elles souhaitent obtenir le jeton pendant l’échange de jetons.|
| `code`          | Obligatoire  | Le code d’autorisation acquis dans le premier tronçon du flux. |
| `redirect_uri`  | Obligatoire  | Valeur redirect_uri qui a déjà été utilisée pour obtenir le paramètre authorization_code. |
| `client_secret` | obligatoire pour les applications web confidentielles | Le secret d’application que vous avez créé dans le portail d’inscription des applications pour votre application. Vous ne devez pas l’utiliser dans une application native ou monopage, car les clés secrètes client ne peuvent pas être stockées de manière fiable sur des appareils ou des pages web. Il est requis pour les applications web et les API web, qui présentent la capacité de stocker de manière sûre les clés secrètes client sur le côté serveur.  Le secret du client doit être codé en URL avant d’être envoyé. Pour plus d’informations sur l’encodage d’URI, consultez la [spécification de syntaxe générique URI](https://tools.ietf.org/html/rfc3986#page-12). |
| `code_verifier` | recommandé  | Le même code_verifier utilisé pour obtenir le authorization_code. Obligatoire si PKCE est utilisé dans la requête d’octroi du code d’autorisation. Pour plus d'informations, consultez le [RFC PKCE](https://tools.ietf.org/html/rfc7636). |

### <a name="successful-response"></a>Réponse correcte

Une réponse de jeton réussie se présente ainsi :

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```

| Paramètre     | Description   |
|---------------|------------------------------|
| `access_token`  | Le jeton d’accès demandé. L’application peut utiliser ce jeton pour procéder à l’authentification sur la ressource sécurisée, par exemple une API Web.  |
| `token_type`    | Indique la valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le jeton porteur. |
| `expires_in`    | La durée de validité (en secondes) du jeton d’accès. |
| `scope`         | L’étendue de validité du jeton d’accès. Facultatif : il s’agit d’un jeton non standard et, s’il est omis, le jeton sera destiné aux étendues demandées sur la branche initiale du flux. |
| `refresh_token` | Un jeton d’actualisation OAuth 2.0. L’application peut utiliser ce jeton pour acquérir des jetons d’accès supplémentaires après l’expiration du jeton d’accès actuel. Les jetons d’actualisation sont durables, et peuvent être utilisés pour conserver l’accès aux ressources pendant des périodes prolongées. Pour plus d’informations sur l’actualisation d’un jeton d’accès, reportez-vous à la [section ci-dessous](#refresh-the-access-token). <br> **Remarque :** Fourni uniquement si l’étendue `offline_access` a été demandée. |
| `id_token`      | Un JSON Web Token (JWT). L’application peut décoder les segments de ce jeton, afin de demander des informations relatives à l’utilisateur qui s’est connecté. L’application peut mettre en cache les valeurs et les afficher, et les clients confidentiels peuvent s’en servir pour obtenir une autorisation. Pour en savoir plus sur id_tokens, consultez [`id_token reference`](id-tokens.md). <br> **Remarque :** Fourni uniquement si l’étendue `openid` a été demandée. |

### <a name="error-response"></a>Réponse d’erreur

Les réponses d’erreur se présentent comme suit :

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Paramètre         | Description    |
|-------------------|----------------|
| `error`       | Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur d’authentification. |
| `error_codes` | Liste des codes d’erreur STS spécifiques pouvant être utiles dans les tests de diagnostic.  |
| `timestamp`   | Heure à laquelle l’erreur s’est produite. |
| `trace_id`    | Identifiant unique de la demande pouvant être utile dans les tests de diagnostic. |
| `correlation_id` | Identifiant unique de la demande pouvant être utile dans les tests de diagnostic sur les divers composants. |

### <a name="error-codes-for-token-endpoint-errors"></a>Codes d’erreur pour les erreurs de point de terminaison de jeton

| Code d'erreur         | Description        | Action du client    |
|--------------------|--------------------|------------------|
| `invalid_request`  | Erreur de protocole, tel qu’un paramètre obligatoire manquant. | Corriger la demande ou l’inscription d’application et renvoyer la demande   |
| `invalid_grant`    | Le code d’autorisation ou le vérificateur du code PCKE n’est pas valide ou a expiré. | Essayez une nouvelle requête au point de terminaison `/authorize` et vérifiez que le paramètre code_verifier est correct.  |
| `unauthorized_client` | Le client authentifié n’est pas autorisé à utiliser ce type d’octroi d’autorisation. | Cela se produit généralement lorsque l’application cliente n’est pas inscrite dans Azure AD ou n’est pas ajoutée au client Azure AD de l’utilisateur. L’application peut proposer à l’utilisateur des instructions pour installer l’application et l’ajouter à Azure AD. |
| `invalid_client` | Échec d’authentification du client.  | Les informations d’identification du client ne sont pas valides. Pour résoudre le problème, l’administrateur de l’application met à jour les informations d’identification.   |
| `unsupported_grant_type` | Le serveur d’autorisation ne prend pas en charge le type d’octroi d’autorisation. | Modifiez le type d’octroi dans la demande. Ce type d’erreur doit se produire uniquement lors du développement et doit être détecté lors du test initial. |
| `invalid_resource` | La ressource cible n’est pas valide car elle n’existe pas, Azure AD ne la trouve pas ou elle n’est pas configurée correctement. | Cela indique que la ressource, si elle existe, n’a pas été configurée dans le client. L’application peut proposer à l’utilisateur des instructions pour installer l’application et l’ajouter à Azure AD.  |
| `interaction_required` | Non standard, car la spécification OIDC l’appelle uniquement sur le point de terminaison `/authorize`. La requête nécessite une interaction avec l’utilisateur. Par exemple, une étape d’authentification supplémentaire est nécessaire. | Relancez la requête `/authorize` avec les mêmes étendues. |
| `temporarily_unavailable` | Le serveur est temporairement trop occupé pour traiter la demande. | Relancez la requête après un petit délai. L’application cliente peut expliquer à l’utilisateur que sa réponse est reportée en raison d’une condition temporaire. |
|`consent_required` | La requête nécessite le consentement de l’utilisateur. Cette erreur n’est pas standard, car elle est généralement retournée uniquement sur le point de terminaison `/authorize` conformément aux spécifications OIDC. Retournée lorsqu’un paramètre `scope` a été utilisé sur le flux d’échange de code et que l’application cliente n’a pas l’autorisation d’en faire la demande.  | Le client doit renvoyer l’utilisateur au point de terminaison `/authorize` avec l’étendue correcte afin de déclencher le consentement. |
|`invalid_scope` | L’étendue demandée par l’application n’est pas valide.  | Mettez à jour la valeur du paramètre d’étendue dans la demande d’authentification en la remplaçant par une valeur valide. |

> [!NOTE]
> Les applications monopages peuvent recevoir un message d’erreur `invalid_request` indiquant qu’un échange de jetons cross-origin n’est autorisé que pour le type de client « application monopage ».  Cela indique que l’URI de redirection utilisé pour demander le jeton n’a pas été marqué comme URI de redirection de `spa`.  Pour voir comment activer ce flux, examinez les [étapes d’inscription d’application](#redirect-uri-setup-required-for-single-page-apps).

## <a name="use-the-access-token"></a>Utiliser le jeton d’accès

Maintenant que vous avez acquis un jeton `access_token`, vous pouvez l’utiliser dans des requêtes adressées à des API web en l’incluant dans l’en-tête `Authorization` :

> [!TIP]
> Exécutez cette requête dans Postman ! Commencez par remplacez le `Authorization`[![Essayez d’exécuter cette requête dans Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```HTTP
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-access-token"></a>Actualiser le jeton d’accès

Les jetons d’accès présentent une durée de vie courte. Après leur expiration, vous devez les actualiser afin de pouvoir continuer à accéder aux ressources. Pour ce faire, envoyez une nouvelle requête `POST` au point de terminaison `/token`, en fournissant l’élément `refresh_token` au lieu de l’élément `code`.  Les jetons d’actualisation sont valides pour toutes les autorisations pour lesquelles votre client a déjà reçu un consentement. Par conséquent, un jeton d’actualisation émis pour une requête de `scope=mail.read` peut être utilisé pour demander un nouveau jeton d’accès pour `scope=api://contoso.com/api/UseResource`.

Les jetons d’actualisation pour les applications web et natives n’ont pas de durée de vie spécifiée. En règle générale, leur durée de vie est relativement longue. Toutefois, dans certains cas, les jetons d’actualisation expirent, sont révoqués ou ne disposent pas de privilèges suffisants pour l’action souhaitée. Votre application doit envisager et gérer correctement les [erreurs retournées par le point de terminaison d’émission de jeton](#error-codes-for-token-endpoint-errors). Toutefois, les applications monopages obtiennent un jeton d’une durée de vie de 24 heures, ce qui nécessite une nouvelle authentification quotidienne.  Cela peut être effectué en mode silencieux dans un iframe lorsque les cookies tiers sont activés, mais doit être fait dans un cadre de niveau supérieur (navigation en page complète ou fenêtre contextuelle) dans des navigateurs sans cookies tiers, tels que Safari.

Bien que les jetons d’actualisation ne sont pas révoqués lorsqu’il est utilisé pour acquérir de nouveaux jetons d’accès, vous êtes censé ignorer l’ancien jeton d’actualisation. La [spécification OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-6) indique ce qui suit : « Le serveur d’autorisation PEUT émettre un nouveau jeton d’actualisation, auquel cas, le client DOIT ignorer l’ancien jeton d’actualisation et le remplacer par le nouveau. Le serveur d’autorisation PEUT révoquer l’ancien jeton d’actualisation après en avoir émis un nouveau pour le client. »

>[!IMPORTANT]
> Pour les jetons d’actualisation envoyés à un URI de redirection inscrit en tant que `spa`, le jeton d’actualisation expire au bout de 24 heures. Les jetons d’actualisation supplémentaires acquis à l’aide du jeton d’actualisation initial répercutant ce délai d’expiration, les applications doivent être prêtes à réexécuter le flux de code d’autorisation en utilisant une authentification interactive pour obtenir un nouveau jeton d’actualisation toutes les 24 heures. Les utilisateurs n’ont pas besoin d’entrer leurs informations d’identification et ne voient généralement même pas l’interface utilisateur, mais juste un rechargement votre application. Le navigateur doit cependant accéder à la page de connexion dans un cadre de niveau supérieur pour voir la session de connexion.  Cela est dû au [fonctionnalités de confidentialité dans les navigateurs qui bloquent les cookies tiers](reference-third-party-cookies-spas.md).

```HTTP

// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&grant_type=refresh_token
&client_secret=JqQX2PNo9bpM0uEihUPzyrh      // NOTE: Only required for web apps. This secret needs to be URL-Encoded
```

> [!TIP]
> Essayez d'exécuter cette requête dans Postman ! N’oubliez pas de remplacer le `refresh_token`[![Essayez d’exécuter cette requête dans Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)
>

| Paramètre     | Type           | Description        |
|---------------|----------------|--------------------|
| `tenant`        | Obligatoire     | La valeur `{tenant}` dans le chemin d’accès de la requête peut être utilisée pour contrôler les utilisateurs qui peuvent se connecter à l’application. Les valeurs autorisées sont `common`, `organizations`, `consumers` et les identificateurs du client. Pour plus d’informations, consultez les [principes de base du protocole](active-directory-v2-protocols.md#endpoints).   |
| `client_id`     | Obligatoire    | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `grant_type`    | Obligatoire    | Doit inclure `refresh_token` pour ce tronçon du flux de code d'autorisation. |
| `scope`         | Obligatoire    | Une liste d’étendues séparées par des espaces. Les étendues demandées dans ce tronçon doivent être équivalentes aux étendues demandées dans le tronçon de requête authorization_code d’origine, ou correspondre à un sous-ensemble de ces dernières. Si les étendues spécifiées dans cette requête couvrent plusieurs serveurs de ressources, le point de terminaison de la plateforme d’identités Microsoft renvoie alors un jeton pour la ressource spécifiée dans la première étendue. Pour obtenir une explication plus détaillée des étendues, consultez les [autorisations, consentements et étendues](v2-permissions-and-consent.md). |
| `refresh_token` | Obligatoire    | Le jeton d’actualisation que vous avez acquis dans le second tronçon du flux. |
| `client_secret` | requis pour les applications Web | Le secret d’application que vous avez créé dans le portail d’inscription des applications pour votre application. Il ne doit pas être utilisé dans une application native, car les clés secrètes client ne peuvent pas être stockées de manière sûre sur les appareils. Il est requis pour les applications web et les API web, qui présentent la capacité de stocker de manière sûre les clés secrètes client sur le côté serveur. Ce secret doit se présenter sous la forme d’un code URL. Pour plus d’informations, consultez la [spécification de syntaxe générique URI](https://tools.ietf.org/html/rfc3986#page-12). |

#### <a name="successful-response"></a>Réponse correcte

Une réponse de jeton réussie se présente ainsi :

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```

| Paramètre     | Description         |
|---------------|-------------------------------------------------------------|
| `access_token`  | Le jeton d’accès demandé. L’application peut utiliser ce jeton pour procéder à l’authentification sur la ressource sécurisée, par exemple une API Web. |
| `token_type`    | Indique la valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le jeton porteur. |
| `expires_in`    | La durée de validité (en secondes) du jeton d’accès.   |
| `scope`         | L’étendue de validité du jeton d’accès.    |
| `refresh_token` | Un nouveau jeton d’actualisation OAuth 2.0. Vous devez remplacer l’ancien jeton d’actualisation par ce nouveau jeton d’actualisation nouvellement acquis, afin de vous assurer que vos jetons d’actualisation demeurent valident le plus longtemps possible. <br> **Remarque :** Fourni uniquement si l’étendue `offline_access` a été demandée.|
| `id_token`      | Un jeton Web JSON non signé (JWT). L’application peut décoder les segments de ce jeton, afin de demander des informations relatives à l’utilisateur qui s’est connecté. L’application peut mettre en cache les valeurs et les afficher, mais ne peut aucunement les utiliser pour les limites d’autorisation ou de sécurité. Pour en savoir plus sur id_tokens, consultez [`id_token reference`](id-tokens.md). <br> **Remarque :** Fourni uniquement si l’étendue `openid` a été demandée. |

#### <a name="error-response"></a>Réponse d’erreur

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Paramètre         | Description                                                                                        |
|-------------------|----------------------------------------------------------------------------------------------------|
| `error`           | Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur d’authentification.           |
| `error_codes` |Liste des codes d’erreur STS spécifiques pouvant être utiles dans les tests de diagnostic. |
| `timestamp` | Heure à laquelle l’erreur s’est produite. |
| `trace_id` | Identifiant unique de la demande pouvant être utile dans les tests de diagnostic. |
| `correlation_id` | Identifiant unique de la demande pouvant être utile dans les tests de diagnostic sur les divers composants. |

Pour obtenir une description des codes d’erreur et connaître l’action client recommandée, consultez [Codes d’erreur pour les erreurs de point de terminaison de jeton](#error-codes-for-token-endpoint-errors).
