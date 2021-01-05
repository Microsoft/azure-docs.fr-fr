---
title: Plateforme d’identités Microsoft et protocole OpenID Connect | Azure
titleSuffix: Microsoft identity platform
description: Créez des applications web à l’aide de l’implémentation du protocole d’authentification OpenID Connect issue de la plateforme d’identités Microsoft.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/22/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 71e930898f1f86622357f9e02da69be7bf2f8088
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91256583"
---
# <a name="microsoft-identity-platform-and-openid-connect-protocol"></a>Plateforme d’identités Microsoft et protocole OpenID Connect

OpenID Connect (OIDC) est un protocole d’authentification basé sur OAuth 2.0, que vous pouvez utiliser pour connecter de façon sécurisée un utilisateur à une application. En utilisant l’implémentation d’OpenID Connect provenant du point de terminaison de la plateforme d’identités Microsoft, vous pouvez ajouter l’accès à la connexion et aux API à vos applications. Cet article ne tient pas compte du langage. Il explique comment envoyer et recevoir des messages HTTP sans utiliser l’une des [bibliothèques open source de Microsoft](reference-v2-libraries.md).

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) étend le protocole *d’autorisation* OAuth 2.0 pour l’utiliser en tant que protocole *d’authentification*, ce qui vous permet de procéder à une authentification unique à l’aide d’OAuth. OpenID Connect introduit le concept de *jeton d'ID*, qui est un jeton de sécurité permettant au client de vérifier l’identité de l’utilisateur. Il permet également les informations de base de profil de l'utilisateur. Il présente également le [point de terminaison de UserInfo](userinfo.md), une API qui renvoie des informations sur l’utilisateur. 


## <a name="protocol-diagram-sign-in"></a>Schéma de protocole : Connexion

Le flux de connexion le plus simple se compose des étapes présentées dans le diagramme suivant. Chaque étape est décrite en détail dans cet article.

![Protocole OpenID Connect : Connexion](./media/v2-protocols-oidc/convergence-scenarios-webapp.svg)

## <a name="fetch-the-openid-connect-metadata-document"></a>Récupérez le document de métadonnées OpenID Connect

OpenID Connect décrit un document de métadonnées [(RFC)](https://openid.net/specs/openid-connect-discovery-1_0.html) qui contient la plupart des informations nécessaires à une application pour effectuer la connexion. Cela inclut des informations telles que les URL à utiliser et l’emplacement des clés de signature publiques du service. Pour accéder à ce document, ajoutez le chemin d’accès au document de détection à l’adresse URL de l’autorité :

Chemin d’accès au document de détection : `/.well-known/openid-configuration`

Autorité : `https://login.microsoftonline.com/{tenant}/v2.0`

`{tenant}` peut prendre une des quatre valeurs :

| Valeur | Description |
| --- | --- |
| `common` |Les utilisateurs avec un compte Microsoft personnel et un compte professionnel ou scolaire Azure AD peuvent se connecter à l’application. |
| `organizations` |Seuls les utilisateurs avec des comptes professionnels ou scolaires Azure AD peuvent se connecter à l’application. |
| `consumers` |Seuls les utilisateurs avec un compte personnel Microsoft peuvent se connecter à l’application. |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` ou `contoso.onmicrosoft.com` | Seuls les utilisateurs d’un locataire Azure AD spécifique (qu’ils soient membres de l’annuaire avec un compte professionnel ou scolaire ou qu’ils soient invités dans l’annuaire avec un compte Microsoft personnel) peuvent se connecter à l’application. Le nom de domaine convivial du client Azure AD ou l’identificateur GUID du client peut être utilisé. Vous pouvez également utiliser le client consommateur `9188040d-6c67-4c5b-b112-36a304b66dad` à la place du client `consumers`.  |

L’autorité diffère d’un cloud national à l’autre, par exemple `https://login.microsoftonline.de` pour l’instance Azure AD Germany. Si vous n’utilisez pas le cloud public, passez en revue les [points de terminaison cloud nationaux](authentication-national-cloud.md#azure-ad-authentication-endpoints) pour trouver celui qui vous convient le mieux. Assurez-vous que le locataire et `/v2.0/` sont présents dans votre requête, afin que vous puissiez utiliser la version 2.0 du point de terminaison.

> [!TIP]
> Essayez ! Cliquez sur [https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration](https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration) pour voir la configuration de `common`.

### <a name="sample-request"></a>Exemple de requête

Pour appeler le point de terminaison UserInfo pour l’autorité commune sur le cloud public, utilisez ce qui suit :

```http
GET /common/v2.0/.well-known/openid-configuration
Host: login.microsoftonline.com
```

### <a name="sample-response"></a>Exemple de réponse

Les métadonnées représentent un simple document JavaScript Objet Notation (JSON). Consultez l’extrait suivant pour obtenir un exemple. Son contenu est décrit en détail dans les [spécifications d’OpenID Connect](https://openid.net/specs/openid-connect-discovery-1_0.html#rfc.section.4.2).

```json
{
  "authorization_endpoint": "https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize",
  "token_endpoint": "https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token",
  "token_endpoint_auth_methods_supported": [
    "client_secret_post",
    "private_key_jwt"
  ],
  "jwks_uri": "https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys",
  "userinfo_endpoint": "https://graph.microsoft.com/oidc/userinfo",
  "subject_types_supported": [
      "pairwise"
  ],
  ...

}
```

Si votre application dispose de clés de signature personnalisées après avoir utilisé la fonctionnalité [claims-mapping](active-directory-claims-mapping.md), vous devez ajouter un paramètre de requête `appid` contenant l’ID de l’application afin d’obtenir un `jwks_uri` qui redirige vers l’information de clé de signature de votre application. Par exemple : `https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration?appid=6731de76-14a6-49ae-97bc-6eba6914391e` contient un `jwks_uri` de `https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys?appid=6731de76-14a6-49ae-97bc-6eba6914391e`.

En règle générale, vous utilisez ce document de métadonnées pour configurer une bibliothèque OpenID Connect ou un Kit de développement logiciel (SDK). La bibliothèque utilise les métadonnées pour faire son travail. Toutefois, si vous n’utilisez pas une bibliothèque OpenID Connect précréée, vous pouvez suivre les étapes décrites dans le reste de cet article pour vous connecter à une application web à l’aide du point de terminaison de la plateforme d’identités Microsoft.

## <a name="send-the-sign-in-request"></a>Envoyer la requête de connexion

Lorsque votre application web a besoin d’authentifier l’utilisateur, elle peut le diriger vers le point de terminaison `/authorize`. Cette requête est similaire au premier tronçon du [flux de code d’autorisation OAuth 2.0](v2-oauth2-auth-code-flow.md), avec toutefois ces distinctions importantes :

* La requête doit inclure l’étendue `openid` dans le paramètre `scope`.
* Le paramètre `response_type` doit inclure `id_token`.
* La demande doit inclure le paramètre `nonce` .

> [!IMPORTANT]
> Pour réussir à demander un jeton d’ID au point de terminaison /authorization, l’inscription d’application dans le [portail d’inscription](https://portal.azure.com) doit disposer de l’octroi implicite de l’élément id_tokens dans l’onglet Authentification (celui-ci définit l’indicateur `oauth2AllowIdTokenImplicitFlow` dans le [manifeste d’application](reference-app-manifest.md) sur `true`). S’il n’est pas activé, une erreur `unsupported_response` est retournée : « La valeur fournie pour le paramètre d’entrée ’response_type’ n’est pas autorisée pour ce client. La valeur attendue est ’code’ »

Par exemple :

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=form_post
&scope=openid
&state=12345
&nonce=678910
```

| Paramètre | Condition | Description |
| --- | --- | --- |
| `tenant` | Obligatoire | Vous pouvez utiliser la valeur `{tenant}` dans le chemin d’accès de la requête pour contrôler les utilisateurs qui peuvent se connecter à l’application. Les valeurs autorisées sont `common`, `organizations`, `consumers` et les identificateurs du client. Pour plus d’informations, consultez les [principes de base du protocole](active-directory-v2-protocols.md#endpoints). |
| `client_id` | Obligatoire | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `response_type` | Obligatoire | Doit inclure `id_token` pour la connexion à OpenID Connect. Il peut également inclure d’autres valeurs `response_type`, par exemple `code`. |
| `redirect_uri` | Recommandé | L’URI de redirection de votre application, vers lequel votre application peut envoyer et recevoir des réponses d’authentification. Il doit correspondre exactement à l’un des URI de redirection enregistrés dans le portail, auquel s’ajoute le codage dans une URL. Si vous ne disposez pas d’un URI de redirection, le point de terminaison sélectionnera un élément redirect_uri inscrit au hasard vers lequel il redirigera l’utilisateur. |
| `scope` | Obligatoire | Une liste d’étendues séparées par des espaces. Pour OpenID Connect, vous devez inclure l’étendue `openid`, qui correspond à l’autorisation de connexion dans l’interface utilisateur de consentement. Vous pouvez inclure d’autres étendues dans cette requête pour solliciter le consentement. |
| `nonce` | Obligatoire | Une valeur incluse dans la requête, générée par l’application, qui sera intégrée dans la valeur id_token résultant en tant que revendication. L’application peut vérifier cette valeur afin de contrer les attaques par relecture de jetons. La valeur est généralement une valeur unique, aléatoire pouvant être utilisé pour identifier l’origine de la requête. |
| `response_mode` | Recommandé | Spécifie la méthode à utiliser pour envoyer le code d’autorisation résultant à votre application. Peut être `form_post` ou `fragment`. Pour les applications web, nous vous recommandons d’utiliser `response_mode=form_post` pour garantir le transfert le plus sécurisé des jetons à votre application. |
| `state` | Recommandé | Une valeur incluse dans la requête qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. Une valeur unique générée de manière aléatoire est généralement utilisée pour [empêcher les attaques par falsification de requête intersites](https://tools.ietf.org/html/rfc6749#section-10.12). La valeur d’état est également utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné. |
| `prompt` | Facultatif | Indique le type d’interaction utilisateur requis. Les seules valeurs valides pour l’instant sont `login`, `none` et `consent`. La revendication `prompt=login` oblige l'utilisateur à saisir ses informations d'identification lors de cette requête, annulant de fait l'authentification unique. La revendication `prompt=none` a l'effet inverse. Cette valeur garantit qu’aucune invite interactive n’est présentée à l’utilisateur. Si la demande ne peut pas être exécutée en mode silencieux au moyen d’une authentification unique, le point de terminaison de la plateforme d’identités Microsoft renvoie une erreur. La revendication `prompt=consent` déclenche l'affichage de la boîte de dialogue de consentement OAuth une fois que l’utilisateur se connecte. La boîte de dialogue invite l’utilisateur à accorder des autorisations à l’application. |
| `login_hint` | Facultatif | Vous pouvez utiliser ce paramètre pour remplir au préalable le champ réservé au nom d’utilisateur et à l’adresse électronique de la page de connexion de l’utilisateur si vous connaissez déjà son nom d’utilisateur. Les applications utilisent souvent ce paramètre au cours de la réauthentification, après avoir extrait le nom d’utilisateur à partir d’une connexion précédente à l’aide de la revendication `preferred_username`. |
| `domain_hint` | Facultatif | Le domaine de l’utilisateur dans un répertoire fédéré.  Ce paramètre ignore le processus de découverte par e-mail auquel l’utilisateur doit se soumettre sur la page de connexion, ce qui améliore légèrement l’expérience utilisateur. Pour les clients fédérés par le biais d’un répertoire local comme AD FS, cela aboutit généralement à une connexion fluide en raison d’une ouverture de session existante. |

À ce stade, l’utilisateur est invité à saisir ses informations d’identification et à exécuter l’authentification. Le point de terminaison de la plateforme d’identités Microsoft vérifie que l’utilisateur a accepté les autorisations indiquées dans le paramètre de requête `scope`. Si l’utilisateur n’a pas accepté l’une ou plusieurs de ces autorisations, le point de terminaison de la plateforme d’identités Microsoft lui demande de corriger ce manquement. Vous pouvez en savoir plus sur les [autorisations, consentements et applications mutualisées](v2-permissions-and-consent.md).

Une fois que l’utilisateur a procédé à l’authentification et accordé son consentement, le point de terminaison de la plateforme d’identités Microsoft renvoie une réponse à votre application à l’URI de redirection indiqué, à l’aide de la méthode spécifiée dans le paramètre `response_mode`.

### <a name="successful-response"></a>Réponse correcte

Une réponse correcte lorsque vous utilisez `response_mode=form_post` ressemble à ceci :

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&state=12345
```

| Paramètre | Description |
| --- | --- |
| `id_token` | Le jeton d'ID que l’application a demandé. Vous pouvez utiliser le paramètre `id_token` pour vérifier l’identité de l’utilisateur et démarrer une session avec lui. Pour plus d’informations sur les jetons d’ID et leur contenu, consultez les [références `id_tokens`](id-tokens.md). |
| `state` | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs d’état de la demande et de la réponse sont identiques. |

### <a name="error-response"></a>Réponse d’erreur

Les réponses d’erreur peuvent également être envoyées à l'URI de redirection, de manière que l’application puisse les traiter. Une réponse d’erreur ressemble à ceci :

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Paramètre | Description |
| --- | --- |
| `error` | Une chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreur se produisant et pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |

### <a name="error-codes-for-authorization-endpoint-errors"></a>Codes d’erreur pour les erreurs de point de terminaison d’autorisation

Le tableau suivant décrit les codes d’erreur qui peuvent être retournés dans le paramètre `error` de la réponse d’erreur :

| Code d'erreur | Description | Action du client |
| --- | --- | --- |
| `invalid_request` | Erreur de protocole, tel qu’un paramètre obligatoire manquant. |Corrigez l’erreur, puis envoyez à nouveau la demande. Il s’agit d’une erreur de développement généralement détectée lors des tests initiaux. |
| `unauthorized_client` | L’application cliente ne peut pas demander un code d’autorisation. |Cela se produit généralement lorsque l’application cliente n’est pas inscrite dans Azure AD ou n’est pas ajoutée au client Azure AD de l’utilisateur. L’application peut proposer à l’utilisateur des instructions pour installer l’application et l’ajouter à Azure AD. |
| `access_denied` | Le propriétaire de la ressource n’a pas accordé son consentement. |L’application cliente peut avertir l’utilisateur qu’elle ne peut pas continuer sans son consentement. |
| `unsupported_response_type` |Le serveur d’autorisation ne prend pas en charge le type de réponse dans la demande. |Corrigez l’erreur, puis envoyez à nouveau la demande. Il s’agit d’une erreur de développement généralement détectée lors des tests initiaux. |
| `server_error` | Le serveur a rencontré une erreur inattendue. |Relancez la requête. Ces erreurs peuvent résulter de conditions temporaires. L’application cliente peut expliquer à l’utilisateur que sa réponse est reportée en raison d’une erreur temporaire. |
| `temporarily_unavailable` | Le serveur est temporairement trop occupé pour traiter la demande. |Relancez la requête. L’application cliente peut expliquer à l’utilisateur que sa réponse est reportée en raison d’une condition temporaire. |
| `invalid_resource` | La ressource cible n’est pas valide car elle n’existe pas, Azure AD ne la trouve pas ou elle n’est pas configurée correctement. |Cela indique que la ressource, si elle existe, n’a pas été configurée dans le locataire. L’application peut proposer à l’utilisateur des instructions pour installer l’application et l’ajouter à Azure AD. |

## <a name="validate-the-id-token"></a>Validation du jeton d’ID

La seule réception du jeton id_token ne suffit pas toujours à authentifier l’utilisateur. Vous devrez peut-être aussi valider la signature du jeton id_token et la conformité des revendications du jeton par rapport aux exigences de votre application. Comme toutes les plateformes OIDC, le point de terminaison de la plateforme d’identités Microsoft utilise les [jetons web JSON (JWT)](https://tools.ietf.org/html/rfc7519) et le chiffrement de clés publiques pour signer les jetons d’ID et vérifier leur validité.

Les applications ne bénéficient pas toutes de la vérification du jeton d’ID ; pour les applications natives et à page unique, par exemple, il est rarement avantageux de valider ce jeton.  Une personne disposant d’un accès physique à l’appareil (ou au navigateur) peut contourner la validation de nombreuses façons, de la modification du trafic web vers l’appareil à la fourniture de jetons et de clés factices, en passant par le débogage pur et simple de l’application, afin d’ignorer la logique de validation.  En revanche, les applications web et les API utilisant un jeton d’ID pour l’autorisation doivent valider le jeton d’ID avec soin, car ils régulent l’accès aux données.

Une fois que vous avez validé la signature du jeton id_token, il vous faudra vérifier quelques revendications. Pour plus d’informations, consultez la page de [référence `id_token`](id-tokens.md), notamment les sections [Validation des jetons](id-tokens.md#validating-an-id_token) et [Informations importantes sur la substitution des clés de signature](active-directory-signing-key-rollover.md). Nous vous recommandons d’utiliser une bibliothèque pour analyser et valider les jetons. Il en existe au moins une pour la plupart des langages et plateformes.

En fonction de votre scénario, vous pouvez également valider des revendications supplémentaires. Voici quelques validations courantes :

* S’assurer que l’utilisateur/l’organisation s’est inscrit pour l’application.
* S’assurer que l’utilisateur dispose de l’autorisation/des privilèges appropriés.
* S’assurer de l’utilisation d’une force certaine d’authentification, comme une [authentification multifacteur](../authentication/concept-mfa-howitworks.md).

Une fois que vous avez validé le jeton id_token, vous pouvez démarrer une session avec l’utilisateur et utiliser les revendications du jeton id_token pour récupérer les informations sur l’utilisateur dans votre application. Ces informations peuvent être utilisées pour l’affichage, les enregistrements, la personnalisation, etc.

## <a name="protocol-diagram-access-token-acquisition"></a>Schéma de protocole : Acquisition de jeton d'accès

Beaucoup d’applications web nécessitent une connexion de l’utilisateur, puis un accès au service web pour le compte de cet utilisateur à l’aide d’OAuth. Ce scénario utilise OpenID Connect pour l’authentification de l’utilisateur tout en récupérant un code d’autorisation vous permettant d'obtenir des jetons d’accès si vous utilisez le flux de code d’autorisation OAuth.

Le flux complet de connexion OpenID Connect et d’acquisition des jetons ressemble à l'exemple du diagramme suivant. Nous décrirons en détail chaque étape dans les sections suivantes de cet article.

![Protocole OpenID Connect : Acquisition de jeton](./media/v2-protocols-oidc/convergence-scenarios-webapp-webapi.svg)

## <a name="get-an-access-token-to-call-userinfo"></a>Obtenir un jeton d’accès pour appeler UserInfo

Pour obtenir un jeton pour le point de terminaison OIDC UserInfo, modifiez la requête de connexion :

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e        // Your registered Application ID
&response_type=id_token%20token                       // this will return both an id_token and an access token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F       // Your registered redirect URI, URL encoded
&response_mode=form_post                              // 'form_post' or 'fragment'
&scope=openid+profile+email                           // `openid` is required.  `profile` and `email` provide additional information in the UserInfo endpoint the same way they do in an ID token. 
&state=12345                                          // Any value, provided by your app
&nonce=678910                                         // Any value, provided by your app
```

Vous pouvez également utiliser le [flux de code d’autorisation](v2-oauth2-auth-code-flow.md), le [flux de code de l’appareil](v2-oauth2-device-code.md), ou un [jeton d’actualisation](v2-oauth2-auth-code-flow.md#refresh-the-access-token) à la place de `response_type=token` pour obtenir un jeton pour votre application.

> [!TIP]
> Cliquez sur le lien suivant pour exécuter cette requête. Une fois que vous êtes connecté, votre navigateur est redirigé vers `https://localhost/myapp/`, avec un jeton d’ID et un jeton dans la barre d’adresse. Notez que cette requête utilise `response_mode=fragment` à des fins de démonstration uniquement ; pour une webapp, nous vous recommandons d’utiliser `form_post` pour une plus grande sécurité, le cas échéant. 
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token%20token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=fragment&scope=openid+profile+email&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

### <a name="successful-token-response"></a>Réponse de jeton correcte

Une réponse correcte utilisant `response_mode=form_post` se présente ainsi :

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
 access_token=eyJ0eXAiOiJKV1QiLCJub25jZSI6I....
 &token_type=Bearer
 &expires_in=3598
 &scope=email+openid+profile
 &id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI....
 &state=12345
```

Les paramètres de la réponse signifient la même chose quel que soit le flux utilisé pour les acquérir.

| Paramètre | Description |
| --- | --- |
| `access_token` | Jeton qui sera utilisé pour appeler le point de terminaison UserInfo.|
| `token_type` | Toujours « Porteur ». |
| `expires_in`| Temps écoulé jusqu’à l’expiration du jeton d’accès, en secondes. |
| `scope` | Autorisations d’accès accordées pour le jeton d’accès.  Notez que, comme le point de terminaison UserInfo est hébergé sur Microsoft Graph, des étendues Graph supplémentaires peuvent être répertoriées ici (par exemple, user.read) si elles ont été précédemment accordées à l’application.  Cela est dû au fait qu’un jeton associé à une ressource donnée comprend toujours toutes les autorisations actuellement accordées au client.  |
| `id_token` | Le jeton d'ID que l’application a demandé. Vous pouvez utiliser le jeton d'ID pour vérifier l’identité de l’utilisateur et démarrer une session avec lui. Pour plus de détails sur les jetons d'ID et leur contenu, consultez la page de [référence `id_tokens`](id-tokens.md). |
| `state` | Si un paramètre d’état est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs d’état de la demande et de la réponse sont identiques. |

### <a name="error-response"></a>Réponse d’erreur

Les réponses d’erreur peuvent également être envoyées à l'URI de redirection, de manière que l’application puisse les traiter de manière appropriée. Une réponse d’erreur ressemble à ceci :

```HTTP
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Paramètre | Description |
| --- | --- |
| `error` | Une chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreur se produisant et pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |

Pour obtenir une description des codes d’erreur éventuels et connaître les réponses client recommandées associées, consultez [Codes d’erreur pour les erreurs de point de terminaison d’autorisation](#error-codes-for-authorization-endpoint-errors).

Une fois que vous avez obtenu un code d'autorisation et un jeton d'ID, vous pouvez connecter l’utilisateur et obtenir des jetons d’accès pour son compte. Pour connecter l’utilisateur, vous devez valider le jeton d'ID [exactement comme décrit](id-tokens.md#validating-an-id_token). Pour obtenir des jetons d’accès, suivez la procédure décrite dans la [documentation du flux de code OAuth](v2-oauth2-auth-code-flow.md#request-an-access-token).

### <a name="calling-the-userinfo-endpoint"></a>Appel du point de terminaison UserInfo

Consultez la [documentation relative à UserInfo](userinfo.md#calling-the-api) pour savoir comment appeler le point de terminaison UserInfo avec ce jeton.

## <a name="send-a-sign-out-request"></a>Envoi d’une demande de déconnexion

Si vous souhaitez déconnecter l’utilisateur de votre application, vous ne pouvez pas vous contenter de supprimer les cookies de votre application ou d’arrêter la session de l’utilisateur. Vous devez également rediriger l’utilisateur vers le point de terminaison de la plateforme d’identités Microsoft pour suivre la procédure de déconnexion. Si vous ne le faites pas, l’utilisateur pourra se réauthentifier auprès de votre application sans avoir à saisir de nouveau ses informations d’identification, puisqu’il disposera d’une session d’authentification unique valide avec le point de terminaison de la plateforme d’identités Microsoft.

Vous pouvez rediriger l’utilisateur vers le `end_session_endpoint` répertorié dans le document de métadonnées OpenID Connect :

```HTTP
GET https://login.microsoftonline.com/common/oauth2/v2.0/logout?
post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
```

| Paramètre | Condition | Description |
| ----------------------- | ------------------------------- | ------------ |
| `post_logout_redirect_uri` | Recommandé | URL vers laquelle l’utilisateur est redirigé après sa déconnexion. Si le paramètre n’est pas inclus, un message générique généré par le point de terminaison de la plateforme d’identités Microsoft s’affiche sur l’écran de l’utilisateur. Cette URL doit correspondre exactement à l’un des URI de redirection inscrits pour votre application dans le portail d’inscription des applications. |

## <a name="single-sign-out"></a>Déconnexion unique

Lorsque vous redirigez l’utilisateur vers le point `end_session_endpoint`, le point de terminaison de la plateforme d’identités Microsoft efface la session de l’utilisateur dans le navigateur. Toutefois, l’utilisateur peut rester connecté à d’autres applications qui utilisent des comptes Microsoft pour s’authentifier. Pour permettre à ces applications de déconnecter simultanément l’utilisateur, le point de terminaison de la plateforme d’identités Microsoft envoie une requête HTTP GET au lien `LogoutUrl` inscrit de toutes les applications auxquelles l’utilisateur est actuellement connecté. Les applications doivent répondre à cette requête en effaçant toute session qui identifie l’utilisateur et en renvoyant une réponse `200`. Si vous souhaitez prendre en charge la déconnexion unique dans votre application, vous devez implémenter ce paramètre `LogoutUrl` dans le code de votre application. Vous pouvez définir le paramètre `LogoutUrl` à partir du portail d’inscription des applications.

## <a name="next-steps"></a>Étapes suivantes

* Vérifier la [documentation relative à UserInfo](userinfo.md)
* Découvrez comment [personnaliser les valeurs d’un jeton](active-directory-claims-mapping.md) en tirant parti des données de vos systèmes locaux. 
* Découvrez comment [inclure des revendications standard supplémentaires dans des jetons](active-directory-optional-claims.md).  
