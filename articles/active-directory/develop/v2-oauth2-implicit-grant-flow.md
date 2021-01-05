---
title: Flux d’octroi implicite OAuth 2.0 - Plateforme d’identités Microsoft | Azure
description: Sécurisez les applications à page unique avec le flux implicite de la plateforme d’identités Microsoft.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 11/30/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 4b5465cc5c1c3447af5303a5c0bfe82874705362
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511188"
---
# <a name="microsoft-identity-platform-and-implicit-grant-flow"></a>Plateforme d’identités Microsoft et flux d’octroi implicite

La plateforme d’identités Microsoft prend en charge le flux d’octroi implicite OAuth 2.0, comme décrit dans la [spécification OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-4.2). La définition de l’octroi implicite stipule que les jetons (jetons d’ID ou jetons d’accès) sont retournés directement à partir du point de terminaison /authorize au lieu du point de terminaison /token. Cette méthode est souvent utilisée dans le cadre du [flux du code d’autorisation](v2-oauth2-auth-code-flow.md), dans ce que l’on appelle le « flux hybride », qui récupère le jeton d’ID sur la demande /authorize avec un code d’autorisation.

[!INCLUDE [suggest-msal-from-protocols](includes/suggest-msal-from-protocols.md)]

## <a name="prefer-the-auth-code-flow"></a>Privilégier le flux du code d’authentification

Comme il est prévu de [supprimer les cookies tiers des navigateurs](reference-third-party-cookies-spas.md), le **flux d’octroi implicite n'est plus une méthode d'authentification adéquate**.  Les [fonctionnalités d’authentification unique](#getting-access-tokens-silently-in-the-background) silencieuse du flux implicite ne fonctionnent pas sans cookies tiers, ce qui entraîne le blocage des applications qui tentent d'obtenir un nouveau jeton. Nous recommandons vivement que toutes les nouvelles applications utilisent le [flux du code d'autorisation](v2-oauth2-auth-code-flow.md) qui prend désormais en charge les applications monopages au lieu du flux implicite, et que les applications [monopages existantes commencent également leur migration vers le flux du code d'autorisation](migrate-spa-implicit-to-auth-code.md).

## <a name="suitable-scenarios-for-the-oauth2-implicit-grant"></a>Scénarios adaptés à l’octroi implicite OAuth2

L’octroi implicite n'est fiable que pour la partie initiale et interactive de votre flux de connexion, où l'absence de [cookies tiers](reference-third-party-cookies-spas.md) ne peut avoir d'incidence sur votre application. Cette limitation signifie que vous devez l’utiliser exclusivement dans le cadre du flux hybride, où votre application demande un code ainsi qu’un jeton auprès du point de terminaison d’autorisation. Cela garantit que votre application reçoit un code qui peut être échangé contre un jeton d’actualisation, et que la session de connexion de votre application reste valide dans le temps.

## <a name="protocol-diagram"></a>Schéma de protocole

Le diagramme suivant montre à quoi ressemble le flux implicite de connexion complet. Les sections qui suivent décrivent chaque étape plus en détail.

![Diagramme montrant le flux de connexion implicite](./media/v2-oauth2-implicit-grant-flow/convergence-scenarios-implicit.svg)

## <a name="send-the-sign-in-request"></a>Envoyer la requête de connexion

Pour connecter une première fois l’utilisateur à votre application, vous pouvez envoyer une demande d’authentification [OpenID Connect](v2-protocols-oidc.md) et obtenir un jeton `id_token` à partir du point de terminaison de la plateforme d’identités Microsoft.

> [!IMPORTANT]
> Pour demander un jeton d’ID ou d’accès avec succès, le flux d’octroi implicite correspondant à l’inscription d’application sur la page [Inscriptions d’applications du Portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908) doit être activé. Pour ce faire, sélectionnez **Jetons d’ID** et/ou **Jetons d’accès** sous la section **Octroi implicite**. S’il n’est pas activé, une erreur `unsupported_response` est retournée : **La valeur fournie pour le paramètre d’entrée 'response_type' n’est pas autorisée pour ce client. La valeur attendue est ’code’**

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&scope=openid
&response_mode=fragment
&state=12345
&nonce=678910
```

> [!TIP]
> Pour tester la connexion à l’aide du flux implicite, cliquez sur <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=openid&response_mode=fragment&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a> Une fois que vous êtes connecté, votre navigateur doit être redirigé vers `https://localhost/myapp/` avec une valeur `id_token` dans la barre d’adresse.
>

| Paramètre | Type | Description |
| --- | --- | --- |
| `tenant` | Obligatoire |La valeur `{tenant}` dans le chemin d’accès de la requête peut être utilisée pour contrôler les utilisateurs qui peuvent se connecter à l’application. Les valeurs autorisées sont `common`, `organizations`, `consumers` et les identificateurs du client. Pour plus d’informations, consultez les [principes de base du protocole](active-directory-v2-protocols.md#endpoints). |
| `client_id` | Obligatoire | L’ID (client) d’application attribué à votre application par la page [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `response_type` | Obligatoire |Doit inclure `id_token` pour la connexion à OpenID Connect. Il peut également inclure l’élément response_type `token`. L’utilisation de l’élément `token` permet ici à votre application de recevoir immédiatement un jeton d’accès à partir du point de terminaison d’autorisation sans avoir à exécuter une deuxième requête sur le point de terminaison d’autorisation. Si vous utilisez l’élément response_type `token`, le paramètre `scope` doit contenir une étendue indiquant la ressource pour laquelle le jeton doit être émis (par exemple, user.read sur Microsoft Graph). Il peut également contenir la ressource `code` au lieu de `token` pour fournir un code d’autorisation à utiliser dans le [flux du code d’autorisation](v2-oauth2-auth-code-flow.md). Cette réponse id_token+code est parfois appelée « flux hybride ».  |
| `redirect_uri` | recommandé |L’URI de redirection de votre application, vers lequel votre application peut envoyer et recevoir des réponses d’authentification. Il doit correspondre exactement à l’un des URI de redirection enregistrés dans le portail, auquel s’ajoute le codage dans une URL. |
| `scope` | Obligatoire |Une liste d’[étendues](v2-permissions-and-consent.md) séparées par des espaces. Pour OpenID Connect (id_tokens), vous devez inclure l’étendue `openid`, qui correspond à l’autorisation de connexion dans l’interface utilisateur de consentement. Vous pouvez également inclure les étendues `email` et `profile` permettant d’accéder à des données utilisateur supplémentaires. Vous pouvez aussi inclure d’autres étendues dans cette requête pour solliciter le consentement sur diverses ressources, si un jeton d’accès est demandé. |
| `response_mode` | facultatif |Spécifie la méthode à utiliser pour envoyer le jeton résultant à votre application. Par défaut demande simplement un jeton d’accès, mais fragmente si la requête inclut un jeton id_token. |
| `state` | recommandé |Une valeur incluse dans la requête, qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. Une valeur unique générée de manière aléatoire est généralement utilisée pour [empêcher les falsifications de requête intersite](https://tools.ietf.org/html/rfc6749#section-10.12). La valeur d’état est également utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné. |
| `nonce` | Obligatoire |Valeur incluse dans la requête, générée par l’application, qui sera incluse dans l’id_token résultant en tant que revendication. L’application peut ensuite vérifier cette valeur pour atténuer les attaques par relecture de jetons. La valeur est généralement une chaîne unique et aléatoire qui peut être utilisée pour identifier l’origine de la requête. Nécessaire uniquement lorsqu’un jeton id_token est demandé. |
| `prompt` | facultatif |Indique le type d’interaction utilisateur requis. Les seules valeurs valides pour l’instant sont « login », « none », « select_account » et « consent ». `prompt=login` oblige l'utilisateur à saisir ses informations d'identification lors de cette requête, annulant de fait l'authentification unique. Avec `prompt=none`, c’est le comportement inverse. Cette valeur vous garantit qu’aucune invite interactive d’aucune sorte n’est présentée à l’utilisateur. Si la demande ne peut pas être exécutée en mode silencieux au moyen d’une authentification unique, le point de terminaison de la plateforme d’identités Microsoft renvoie une erreur. `prompt=select_account` envoie l’utilisateur vers un sélecteur de compte dans lequel tous les comptes mémorisés dans la session seront affichés. `prompt=consent` déclenche l’affichage de la boîte de dialogue de consentement OAuth après la connexion de l’utilisateur, afin de lui demander d’octroyer des autorisations à l’application. |
| `login_hint`  |facultatif |Peut être utilisé pour préremplir le champ Nom d’utilisateur/Adresse e-mail de la page de connexion pour l’utilisateur, si vous connaissez son nom d’utilisateur à l’avance. Les applications utilisent souvent ce paramètre au cours de la réauthentification, après avoir extrait le nom d’utilisateur à partir d’une connexion précédente à l’aide de la revendication `preferred_username`.|
| `domain_hint` | facultatif |S’il est inclus, ce paramètre ignore le processus de découverte par e-mail auquel l’utilisateur doit se soumettre sur la page de connexion, ce qui améliore légèrement l’expérience utilisateur. Ce paramètre est couramment utilisé pour les applications métier qui fonctionnent dans un locataire unique, où l’utilisateur est transféré vers le fournisseur de fédération pour ce locataire.  Notez que cette méthode empêche les invités de se connecter à cette application et limite l’utilisation des informations d’identification cloud comme FIDO.  |

À ce stade, l’utilisateur est invité à saisir ses informations d’identification et à exécuter l’authentification. Le point de terminaison de la plateforme d’identités Microsoft s’assure également que l’utilisateur a accepté les autorisations indiquées dans le paramètre de requête `scope`. Si l’utilisateur n’a accepté **aucune** de ces autorisations, il lui demande de corriger ce manquement. Pour plus d’informations, consultez [Autorisations, consentement et applications multi-locataires](v2-permissions-and-consent.md).

Une fois que l’utilisateur a procédé à l’authentification et accordé son consentement, le point de terminaison de la plateforme d’identités Microsoft renvoie une réponse à votre application à l’URI `redirect_uri` indiqué à l’aide de la méthode spécifiée dans le paramètre `response_mode`.

#### <a name="successful-response"></a>Réponse correcte

Une réponse correcte utilisant `response_mode=fragment` et `response_type=id_token+code` ressemble à ceci (des sauts de ligne sont ici insérés pour une meilleure lisibilité) :

```HTTP
GET https://localhost/myapp/#
code=0.AgAAktYV-sfpYESnQynylW_UKZmH-C9y_G1A
&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=12345
```

| Paramètre | Description |
| --- | --- |
| `code` | Inclus si `response_type` inclut `code`. Il s’agit d’un code d’autorisation pouvant être utilisé dans [flux du code d’autorisation](v2-oauth2-auth-code-flow.md).  |
| `access_token` |Inclus si `response_type` inclut `token`. Le jeton d'accès que l’application a demandé. Le jeton d’accès ne doit pas être décodé ou inspecté ; il doit être traité comme une chaîne opaque. |
| `token_type` |Inclus si `response_type` inclut `token`. Sera toujours `Bearer`. |
| `expires_in`|Inclus si `response_type` inclut `token`. Indique, en secondes, la durée de validité du jeton pour la mise en cache. |
| `scope` |Inclus si `response_type` inclut `token`. Indique la ou les étendues pour lesquelles le jeton access_token sera valide. Peut ne pas inclure toutes les étendues demandées, si elles n’étaient pas applicables à l’utilisateur (dans le cas des étendues spécifiques à Azure AD demandées au moment de la connexion avec un compte personnel). |
| `id_token` | Un jeton Web JSON signé (JWT). L’application peut décoder les segments de ce jeton, afin de demander des informations relatives à l’utilisateur qui s’est connecté. L’application peut mettre en cache les valeurs et les afficher, mais elle ne peut pas les utiliser pour les limites d’autorisation ou de sécurité. Pour en savoir plus sur id_tokens, consultez [`id_token reference`](id-tokens.md). <br> **Remarque :** Fourni uniquement si l’étendue `openid` a été demandée et si `response_type` incluait `id_tokens`. |
| `state` |Si un paramètre d’état est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs d’état de la demande et de la réponse sont identiques. |

#### <a name="error-response"></a>Réponse d’erreur

Les réponses d’erreur peuvent également être envoyées à l’élément `redirect_uri` , de manière à ce que l’application puisse les traiter de manière appropriée :

```HTTP
GET https://localhost/myapp/#
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| Paramètre | Description |
| --- | --- |
| `error` |Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs. |
| `error_description` |Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur d’authentification. |

## <a name="getting-access-tokens-silently-in-the-background"></a>Obtention des jetons d’accès en mode silencieux en arrière-plan

> [!Important]
> Il est peu probable que cette partie du flux implicite fonctionne pour votre application, car elle est utilisée dans différents navigateurs en raison de la [suppression des cookies tiers par défaut](reference-third-party-cookies-spas.md).  Même si cela fonctionne toujours dans les navigateurs basés sur Chromium qui ne sont pas en mode Incognito, les développeurs devraient reconsidérer l’utilisation de cette partie du flux. Dans les navigateurs qui ne prennent pas en charge les cookies tiers, vous recevrez un message d’erreur indiquant qu’aucun utilisateur n’est connecté, car les cookies de session de la page de connexion ont été supprimés par le navigateur. 

Maintenant que vous avez connecté l’utilisateur dans votre application à page unique, vous pouvez obtenir silencieusement des jetons d’accès destinés à appeler des API web sécurisées à l’aide de la plateforme d’identités Microsoft, comme [Microsoft Graph](https://developer.microsoft.com/graph). Même si vous avez déjà reçu un jeton utilisant l’élément response_type `token`, vous pouvez utiliser cette méthode pour acquérir des jetons vers des ressources supplémentaires sans avoir à demander à l’utilisateur de se reconnecter.

Dans le flux normal OpenID Connect/OAuth, il vous faut transmettre une demande au point de terminaison `/token` de la plateforme d’identités Microsoft. Vous pouvez effectuer la demande dans un iFrame masqué afin d’obtenir de nouveaux jetons pour d’autres API web :

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fuser.read
&response_mode=fragment
&state=12345
&nonce=678910
&prompt=none
&login_hint=myuser@mycompany.com
```

Pour plus d’informations sur les paramètres de requête dans l’URL, consultez [Envoyer la requête de connexion](#send-the-sign-in-request).

> [!TIP]
> Essayez de copier et coller la requête ci-dessous dans un onglet de navigateur ! (N'oubliez pas de remplacer les valeurs `login_hint` par la valeur correcte pour votre utilisateur)
>
>`https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=https%3A%2F%2Fgraph.microsoft.com%2Fuser.read&response_mode=fragment&state=12345&nonce=678910&prompt=none&login_hint={your-username}`
>
> Notez que cette opération fonctionne même dans les navigateurs sans prise en charge des cookies tiers, puisque vous entrez la requête directement dans une barre de navigateur au lieu de l’ouvrir dans un iFrame. 

Grâce au paramètre `prompt=none` , cette requête va immédiatement réussir ou échouer, avant de revenir vers votre application. La réponse sera envoyée à votre application, à l’`redirect_uri` indiqué, à l’aide de la méthode spécifiée dans le paramètre `response_mode`.

#### <a name="successful-response"></a>Réponse correcte

Une réponse correcte utilisant `response_mode=fragment` se présente ainsi :

```HTTP
GET https://localhost/myapp/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=12345
&token_type=Bearer
&expires_in=3599
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fdirectory.read
```

| Paramètre | Description |
| --- | --- |
| `access_token` |Inclus si `response_type` inclut `token`. Jeton d’accès demandé par l’application, dans ce cas Microsoft Graph. Le jeton d’accès ne doit pas être décodé ou inspecté ; il doit être traité comme une chaîne opaque. |
| `token_type` | Sera toujours `Bearer`. |
| `expires_in` | Indique, en secondes, la durée de validité du jeton pour la mise en cache. |
| `scope` | Indique la ou les étendues pour lesquelles le jeton access_token sera valide. Peut ne pas inclure toutes les étendues demandées, si elles n’étaient pas applicables à l’utilisateur (dans le cas des étendues spécifiques à Azure AD demandées au moment de la connexion avec un compte personnel). |
| `id_token` | Un jeton Web JSON signé (JWT). Inclus si `response_type` inclut `id_token`. L’application peut décoder les segments de ce jeton, afin de demander des informations relatives à l’utilisateur qui s’est connecté. L’application peut mettre en cache les valeurs et les afficher, mais elle ne peut pas les utiliser pour les limites d’autorisation ou de sécurité. Pour en savoir plus sur id_tokens, consultez le document de [`id_token`référence](id-tokens.md). <br> **Remarque :** Fourni uniquement si l’étendue `openid` a été demandée. |
| `state` |Si un paramètre d’état est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs d’état de la demande et de la réponse sont identiques. |

#### <a name="error-response"></a>Réponse d’erreur

Les réponses d’erreur peuvent également être envoyées à l’élément `redirect_uri` , de manière à ce que l’application puisse les traiter de manière appropriée. Dans le cas de `prompt=none`, une erreur se présentera généralement ainsi :

```HTTP
GET https://localhost/myapp/#
error=user_authentication_required
&error_description=the+request+could+not+be+completed+silently
```

| Paramètre | Description |
| --- | --- |
| `error` |Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs. |
| `error_description` |Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur d’authentification. |

Si vous recevez cette erreur dans la requête iFrame, l’utilisateur doit se connecter de nouveau de manière interactive, ceci pour récupérer un nouveau jeton. Vous êtes invité à gérer ce cas de la manière la plus appropriée pour votre application.

## <a name="refreshing-tokens"></a>Actualisation des jetons

L’octroi implicite ne fournit pas de jetons d’actualisation. Les `id_token` et les `access_token` expirant après une courte période, votre application doit être préparée à les actualiser de manière régulière. Pour actualiser chaque type de jeton, vous pouvez exécuter la demande iFrame masquée ci-dessus à l’aide du paramètre `prompt=none` afin de contrôler le comportement de la plateforme d’identités. Si vous souhaitez recevoir un nouvel élément `id_token`, veillez à utiliser `id_token` dans le `response_type` et le `scope=openid`, ainsi qu’un paramètre `nonce`.

Dans les navigateurs qui ne prennent pas en charge les cookies tiers, cela génère une erreur indiquant qu’aucun utilisateur n’est connecté. 

## <a name="send-a-sign-out-request"></a>Envoi d’une demande de déconnexion

L’élément `end_session_endpoint` OpenID Connect permet à votre application d’envoyer une demande au point de terminaison de la plateforme d’identités Microsoft pour mettre fin à la session d’un utilisateur et effacer les cookies définis par le point de terminaison de la plateforme d’identités Microsoft. Pour déconnecter complètement un utilisateur d’une application web, votre application doit mettre fin à sa propre session avec l’utilisateur (généralement en vidant un cache de jeton ou en supprimant les cookies), puis rediriger le navigateur vers :

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/logout?post_logout_redirect_uri=https://localhost/myapp/
```

| Paramètre | Type | Description |
| --- | --- | --- |
| `tenant` |Obligatoire |La valeur `{tenant}` dans le chemin d’accès de la requête peut être utilisée pour contrôler les utilisateurs qui peuvent se connecter à l’application. Les valeurs autorisées sont `common`, `organizations`, `consumers` et les identificateurs du client. Pour plus d’informations, consultez les [principes de base du protocole](active-directory-v2-protocols.md#endpoints). |
| `post_logout_redirect_uri` | recommandé | URL vers laquelle l’utilisateur doit être redirigé après la déconnexion. Cette valeur doit correspondre à l’un des URI de redirection inscrits pour l’application. Si elle n’est pas incluse, le point de terminaison de la plateforme d’identités Microsoft affiche un message générique. |

## <a name="next-steps"></a>Étapes suivantes

* Consultez les [exemples JS MSAL](sample-v2-code.md) pour commencer à coder.
* Vous pouvez utiliser le [flux du code d’autorisation](v2-oauth2-auth-code-flow.md) comme une alternative plus récente et plus efficace à l’octroi implicite. 
