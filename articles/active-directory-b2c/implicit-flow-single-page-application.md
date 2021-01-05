---
title: Connexion monopage avec un flux implicite
titleSuffix: Azure AD B2C
description: Découvrez comment ajouter une connexion monopage en utilisant un flux implicite OAuth 2.0 avec Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/19/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: fe31e1bf095d15cfdd7945288486cb866ace8246
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94840608"
---
# <a name="single-page-sign-in-using-the-oauth-20-implicit-flow-in-azure-active-directory-b2c"></a>Connexion sur page unique en utilisant un flux implicite OAuth 2.0 dans Azure Active Directory B2C

De nombreuses applications modernes disposent d’un front-end d’application monopage écrit principalement en JavaScript. Souvent, l’application est écrite à l’aide d’un framework comme React, Angular ou Vue.js. Les applications à page unique et d’autres applications JavaScript qui s’exécutent principalement dans un navigateur présentent certaines problématiques supplémentaires pour l’authentification :

- Les caractéristiques de sécurité de ces applications sont différentes de celles des applications web traditionnelles basées sur serveur.
- Beaucoup de serveurs d’autorisation et de fournisseurs d’identité ne prennent pas en charge les demandes de partage des ressources cross-origin (CORS).
- Chacune des redirections à partir de l’application du navigateur plein écran peut perturber l’expérience utilisateur.

La méthode recommandée pour la prise en charge d’applications monopages est [le Flux de code d’autorisation OAuth 2.0 (avec PKCE)](./authorization-code-flow.md).

Certaines infrastructures, telles que [MSAL.js 1.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core), ne prennent en charge que le flux d’octroi implicite. Dans ces cas, Azure Active Directory B2C (Azure AD B2C) prend en charge le flux d’octroi implicite de l’autorisation OAuth 2.0. Ce flux est décrit dans la [section 4.2 de la spécification OAuth 2.0](https://tools.ietf.org/html/rfc6749). Dans un flux implicite, l’application reçoit des jetons directement du point de terminaison d’autorisation Azure AD, sans aucun échange de serveur à serveur. L’intégralité de la logique d’authentification et de la gestion des sessions est effectuée sur le client JavaScript, à l’aide d’une redirection de page ou d’une fenêtre contextuelle.

Azure AD B2C étend le flux implicite OAuth 2.0 standard au-delà de la simple authentification et de la simple autorisation. Azure AD B2C introduit le [paramètre de stratégie](user-flow-overview.md). Avec le paramètre de stratégie, vous pouvez utiliser OAuth 2.0 pour ajouter des stratégies à votre application, telles que des flux d’utilisateur d’inscription, de connexion et de gestion des profils. Dans les exemples de requêtes HTTP de cet article, nous utilisons **{tenant}.onmicrosoft.com** comme exemple. Remplacez `{tenant}` par le nom de votre locataire si vous en avez un et que vous avez également créé un flux utilisateur.

Le flux de connexion implicite peut être décrit comme dans la figure suivante. Chaque étape est décrite en détail plus loin dans l’article.

![Diagramme de style couloir montrant le flux implicite OpenID Connect](./media/implicit-flow-single-page-application/convergence_scenarios_implicit.png)

## <a name="send-authentication-requests"></a>Envoi de demandes d’authentification

Lorsque votre application web a besoin d’authentifier l’utilisateur et d’exécuter un flux utilisateur, elle peut diriger l’utilisateur vers le point de terminaison `/authorize`. L’utilisateur prend des mesures en fonction du flux utilisateur.

Dans cette requête, le client désigne les autorisations qu'il doit obtenir de l'utilisateur dans le paramètre `scope` ainsi que le flux utilisateur à exécuter. Pour avoir une idée du fonctionnement de la requête, collez-la dans un navigateur et exécutez-la. Remplacez `{tenant}` par le nom de votre locataire Azure AD B2C. Remplacez `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` par l'ID de l'application que vous avez précédemment inscrite dans votre locataire. Remplacez `{policy}` par le nom d'une stratégie que vous avez créée dans votre locataire, par exemple `b2c_1_sign_in`.

```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=id_token+token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=fragment
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
```

| Paramètre | Obligatoire | Description |
| --------- | -------- | ----------- |
|{tenant}| Oui | Nom de votre locataire Azure AD B2C|
|{policy}| Oui| Flux utilisateur à exécuter. Spécifiez le nom d'un flux utilisateur créé dans votre locataire Azure AD B2C. Par exemple : `b2c_1_sign_in`, `b2c_1_sign_up` ou `b2c_1_edit_profile`. |
| client_id | Oui | ID d’application que le [portail Azure](https://portal.azure.com/) a affecté à votre application. |
| response_type | Oui | Doit inclure `id_token` pour la connexion à OpenID Connect. Il peut également inclure le type de réponse `token`. Si vous utilisez `token`, votre application peut recevoir immédiatement un jeton d’accès du point de terminaison d’autorisation sans avoir à effectuer une deuxième demande au point de terminaison d’autorisation.  Si vous utilisez le type de réponse `token`, le paramètre `scope` doit contenir une étendue indiquant la ressource pour laquelle le jeton doit être émis. |
| redirect_uri | Non  | L’URI de redirection de votre application, vers lequel votre application peut envoyer et recevoir des réponses d’authentification. Il doit correspondre exactement à l’un des URI de redirection que vous avez enregistrés dans le portail, sauf qu’il doit être encodé au format URL. |
| response_mode | Non | Spécifie la méthode à utiliser pour renvoyer le jeton résultant à votre application.  Pour les flux implicites, utilisez `fragment`. |
| scope | Oui | Une liste d’étendues séparées par des espaces. Une valeur d’étendue unique indique à Azure AD les deux autorisations qui sont demandées. L’étendue `openid` indique une autorisation pour connecter l’utilisateur et obtenir des données relatives à l’utilisateur sous la forme de jetons d’ID. L’étendue `offline_access` est facultative pour les applications Web. Elle indique que votre application a besoin d’un jeton d’actualisation pour un accès durable aux ressources. |
| state | Non  | Valeur incluse dans la demande qui est également retournée dans la réponse de jeton. Il peut s’agir d’une chaîne de n’importe quel contenu que vous voulez utiliser. Généralement, une valeur unique générée de manière aléatoire est utilisée, de façon à empêcher les attaques par falsification de requête intersites. L’état est également utilisé pour coder les informations sur l’état de l’utilisateur dans l’application avant la demande d’authentification, comme la page sur laquelle il était positionné. |
| nonce | Oui | Une valeur incluse dans la demande (générée par l’application) qui est incluse dans le jeton d’ID résultant en tant que revendication. L’application peut ensuite vérifier cette valeur pour atténuer les attaques par relecture de jetons. En général, la valeur est une chaîne unique aléatoire qui peut être utilisée pour identifier l’origine de la demande. |
| prompt | Non  | Type d’interaction utilisateur demandée. Actuellement, la seule valeur possible est `login`. Ce paramètre force l’utilisateur à entrer ses informations d’identification pour cette demande. L’authentification unique ne prend pas effet. |

À ce stade, il est demandé à l’utilisateur d’effectuer le flux de travail de la stratégie. L’utilisateur peut avoir à entrer son nom d’utilisateur et son mot de passe, à se connecter avec une identité sociale, à s’inscrire à l’annuaire ou à suivre n’importe quelle autre étape. Les actions de l’utilisateur dépendent de la façon dont le flux d’utilisateur est défini.

Une fois que l’utilisateur a terminé le flux d’utilisateur, Azure AD retourne une réponse à votre application avec la valeur que vous avez utilisée pour `redirect_uri`. Il utilise la méthode spécifiée dans le paramètre `response_mode`. La réponse est exactement la même pour chacun des scénarios d’actions utilisateur, indépendamment du flux d’utilisateur exécuté.

### <a name="successful-response"></a>Réponse correcte
Une réponse correcte qui utilise `response_mode=fragment` et `response_type=id_token+token` est semblable à ceci (des sauts de ligne ont été insérés pour une meilleure lisibilité) :

```http
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&token_type=Bearer
&expires_in=3599
&scope="90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
```

| Paramètre | Description |
| --------- | ----------- |
| access_token | Le jeton d'accès que l’application a demandé. |
| token_type | Valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le jeton porteur. |
| expires_in | Durée de validité du jeton d’accès (en secondes). |
| scope | Étendues de validité du jeton. Vous pouvez également utiliser des étendues pour mettre en cache des jetons pour une utilisation ultérieure. |
| id_token | Le jeton d'ID que l’application a demandé. Vous pouvez utiliser le jeton d'ID pour vérifier l’identité de l’utilisateur et démarrer une session avec lui. Pour plus d’informations sur les jetons d’ID et leur contenu, consultez les [Informations de référence sur les jetons Azure AD B2C](tokens-overview.md). |
| state | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |

### <a name="error-response"></a>Réponse d’erreur
Les réponses d’erreur peuvent également être envoyées à l’URI de redirection, pour que l’application puisse les traiter de façon appropriée :

```http
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| Paramètre | Description |
| --------- | ----------- |
| error | Code utilisé pour classer les types d’erreurs qui se produisent. |
| error_description | Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |
| state | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques.|

## <a name="validate-the-id-token"></a>Validation du jeton d’ID

La réception d’un jeton d’ID n’est pas suffisante pour authentifier l’utilisateur. Validez la signature du jeton d’ID et vérifiez les revendications du jeton selon les exigences de votre application. Azure AD B2C utilise les [jetons Web JSON (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) et le chiffrement de clés publiques pour signer les jetons et vérifier leur validité.

Il existe de nombreuses bibliothèques open source pour valider les jetons JWT en fonction du langage que vous préférez utiliser. Nous vous recommandons d’explorer ces bibliothèques open source plutôt que d’implémenter votre propre logique de validation. Vous pouvez utiliser les informations de cet article pour découvrir comment utiliser correctement ces bibliothèques.

Azure AD B2C a un point de terminaison des métadonnées OpenID Connect. Une application peut utiliser le point de terminaison pour extraire des informations sur Azure AD B2C lors de l’exécution. Ces informations incluent les points de terminaison, le contenu des jetons et les clés de signature de jetons. Il existe un document de métadonnées JSON pour chaque flux d’utilisateur dans votre locataire Azure AD B2C. Par exemple, le document de métadonnées pour le flux d’utilisateur b2c_1_sign_in dans le locataire fabrikamb2c.onmicrosoft.com se trouve ici :

```http
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/v2.0/.well-known/openid-configuration
```

Une des propriétés de ce document de configuration est `jwks_uri`. La valeur du même flux d’utilisateur serait :

```http
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/discovery/v2.0/keys
```

Pour déterminer le flux d’utilisateur utilisé pour signer un jeton d’ID (et l’emplacement d’où extraire les métadonnées), deux options sont possibles. D’abord, le nom du flux d’utilisateur est inclus dans la revendication `acr` de `id_token`. Pour plus d’informations sur la façon d’analyser les revendications à partir d’un jeton d’ID, consultez [Informations de référence sur les jetons Azure AD B2C](tokens-overview.md). L’autre option consiste à coder le flux d’utilisateur dans la valeur du paramètre `state` lors de l’émission de la demande, puis de décoder le paramètre `state` pour déterminer quel flux d’utilisateur a été utilisé. Les 2 méthodes sont valides.

Après avoir acquis le document de métadonnées auprès du point de terminaison de métadonnées OpenID Connect, vous pouvez utiliser les clés publiques RSA 256 (qui se trouvent sur ce point de terminaison) pour valider la signature du jeton d’ID. Il peut y avoir plusieurs clés répertoriées sur ce point de terminaison, chacune étant identifiée par un `kid`. L’en-tête de `id_token` contient également une revendication de `kid`. Il indique laquelle de ces clés a été utilisée pour signer le jeton d’ID. Pour plus d’informations, notamment sur la [validation des jetons](tokens-overview.md), consultez les [informations de référence sur les jetons d’Azure AD B2C](tokens-overview.md).
<!--TODO: Improve the information on this-->

Après la validation de la signature du jeton d’ID, plusieurs revendications nécessitent une vérification. Par exemple :

* Validez la revendication `nonce` afin d’empêcher les attaques par relecture de jetons. Sa valeur doit correspondre à ce que vous avez spécifié dans la requête de connexion.
* Validez la revendication `aud` pour vérifier que le jeton d’ID a été émis pour votre application. Sa valeur doit correspondre à l’ID d’application de votre application.
* Validez les revendications `iat` et `exp` pour vérifier que le jeton d’ID n’est pas expiré.

Plusieurs autres validations à effectuer sont décrites en détail dans les [spécifications principales d’OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html). En fonction de votre scénario, vous pouvez également valider des revendications supplémentaires. Voici quelques validations courantes :

* Vérifier que l’utilisateur ou l’organisation s’est inscrit pour l’application.
* Vérifier que l’utilisateur dispose de l’autorisation et des privilèges appropriés.
* Vérifier qu'un certain niveau d'authentification a été atteint, par exemple en utilisant Azure AD Multi-Factor Authentication.

Pour plus d’informations sur les revendications dans un jeton d’ID, consultez les [Informations de référence sur les jetons Azure AD B2C](tokens-overview.md).

Une fois que vous avez validé le jeton d’ID, vous pouvez démarrer une session avec l’utilisateur. Dans votre application, utilisez les revendications du jeton d’ID pour obtenir des informations sur l’utilisateur. Ces informations peuvent être utilisées pour l’affichage, les enregistrements, les autorisations, etc.

## <a name="get-access-tokens"></a>Obtenir des jetons d’accès
Si la seule chose que doit faire votre application web est exécuter des flux d’utilisateur, vous pouvez ignorer les quelques sections suivantes. Les informations des sections suivantes s’appliquent uniquement aux applications web qui doivent effectuer des appels authentifiés à une API web et qui sont protégées par Azure AD B2C.

Maintenant que vous avez connecté l’utilisateur à votre application à page unique, vous pouvez obtenir des jetons d’accès pour appeler des API web sécurisées par Azure AD. Même si vous avez déjà reçu un jeton en utilisant le type de réponse `token`, vous pouvez utiliser cette méthode pour acquérir des jetons pour des ressources supplémentaires sans avoir à demander à l’utilisateur de se reconnecter.

Dans le flux d’une application web standard, vous effectuez une demande au point de terminaison `/token`. Cependant, le point de terminaison ne prend pas en charge les requêtes CORS : il n’est donc pas possible d’effectuer des appels AJAX pour obtenir un jeton d’actualisation. Au lieu de cela, vous pouvez utiliser le flux implicite d’un élément IFrame HTML masqué pour obtenir de nouveaux jetons pour d’autres API web. Voici un exemple, avec des sauts de ligne pour une meilleure lisibilité :

```http
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
&response_mode=fragment
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&prompt=none
```

| Paramètre | Requis ? | Description |
| --- | --- | --- |
|{tenant}| Obligatoire | Nom de votre locataire Azure AD B2C|
{policy}| Obligatoire| Flux utilisateur à exécuter. Spécifiez le nom d'un flux utilisateur créé dans votre locataire Azure AD B2C. Par exemple : `b2c_1_sign_in`, `b2c_1_sign_up` ou `b2c_1_edit_profile`. |
| client_id |Obligatoire |ID d’application affecté à votre application dans le [portail Azure](https://portal.azure.com). |
| response_type |Obligatoire |Doit inclure `id_token` pour la connexion à OpenID Connect.  Il peut également inclure le type de réponse `token`. Si vous utilisez `token` ici, votre application peut recevoir immédiatement un jeton d’accès du point de terminaison d’autorisation, sans avoir à effectuer une deuxième demande au point de terminaison d’autorisation. Si vous utilisez le type de réponse `token`, le paramètre `scope` doit contenir une étendue indiquant la ressource pour laquelle le jeton doit être émis. |
| redirect_uri |Recommandé |L’URI de redirection de votre application, vers lequel votre application peut envoyer et recevoir des réponses d’authentification. Il doit correspondre exactement à un des URI de redirection inscrits dans le portail, sauf qu’il doit être codé dans une URL. |
| scope |Obligatoire |Une liste d’étendues séparées par des espaces.  Pour obtenir des jetons, incluez toutes les étendues dont vous avez besoin pour la ressource concernée. |
| response_mode |Recommandé |Spécifie la méthode utilisée pour envoyer le jeton résultant à votre application. Pour des flux implicites, utilisez `fragment`. Deux autres modes peuvent être spécifiés, `query` et `form_post`, mais ne fonctionnent pas dans le flux implicite. |
| state |Recommandé |Une valeur incluse dans la requête qui est également renvoyée dans la réponse de jeton.  Il peut s’agir d’une chaîne de n’importe quel contenu que vous voulez utiliser.  Généralement, une valeur unique générée de manière aléatoire est utilisée, de façon à empêcher les attaques par falsification de requête intersites.  L’état est également utilisé pour coder les informations sur l’état de l’utilisateur dans l’application avant la demande d’authentification. Par exemple, la page ou la vue où était l’utilisateur. |
| nonce |Obligatoire |Valeur incluse dans la demande, générée par l’application, qui est incluse dans le jeton d’ID résultant en tant que revendication.  L’application peut ensuite vérifier cette valeur pour atténuer les attaques par relecture de jetons. La valeur est généralement une chaîne unique aléatoire qui identifie l’origine de la demande. |
| prompt |Obligatoire |Pour actualiser et obtenir des jetons dans un IFrame masqué, utilisez `prompt=none` pour que l’IFrame ne se bloque pas sur la page de connexion et ne retourne pas immédiatement. |
| login_hint |Obligatoire |Pour actualiser et obtenir des jetons dans un IFrame masqué, vous devez inclure dans cet indicateur le nom d’utilisateur de l’utilisateur afin de faire la distinction entre différentes sessions que l’utilisateur pourrait avoir à un moment donné. Vous pouvez extraire le nom d’utilisateur à partir d’une précédente connexion à l’aide de la revendication `preferred_username` (l’étendue `profile` est nécessaire pour recevoir la revendication `preferred_username`). |
| domain_hint |Obligatoire |Peut être `consumers` ou `organizations`.  Pour actualiser et obtenir des jetons dans un IFrame masqué, incluez la valeur `domain_hint` dans la demande.  Extrayez la revendication `tid` du jeton d’ID d’une connexion précédente pour déterminer la valeur à utiliser (l’étendue `profile` est nécessaire pour recevoir la revendication `tid`). Si la valeur de la revendication `tid` est `9188040d-6c67-4c5b-b112-36a304b66dad`, utilisez `domain_hint=consumers`.  Sinon, utilisez `domain_hint=organizations`. |

Avec le paramètre `prompt=none`, cette demande réussit ou échoue immédiatement, et retourne à votre application.  Une réponse correcte est envoyée à votre application à l’URI de redirection indiqué, en utilisant la méthode spécifiée dans le paramètre `response_mode`.

### <a name="successful-response"></a>Réponse correcte
Une réponse de réussite utilisant `response_mode=fragment` se présente ainsi :

```http
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
&token_type=Bearer
&expires_in=3599
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
```

| Paramètre | Description |
| --- | --- |
| access_token |Le jeton que l’application a demandé. |
| token_type |Le type de jeton sera toujours Porteur. |
| state |Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |
| expires_in |Durée de validité du jeton d’accès (en secondes). |
| scope |Les étendues de validité du jeton d’accès. |

### <a name="error-response"></a>Réponse d’erreur
Les réponses d’erreur peuvent également être envoyées à l’URI de redirection, pour que l’application puisse les traiter de façon appropriée.  Pour `prompt=none`, une erreur attendue se présente ainsi :

```http
GET https://aadb2cplayground.azurewebsites.net/#
error=user_authentication_required
&error_description=the+request+could+not+be+completed+silently
```

| Paramètre | Description |
| --- | --- |
| error |Une chaîne de code d’erreur qui peut être utilisée pour classer les types d’erreurs qui se produisent. Vous pouvez également utiliser la chaîne pour réagir aux erreurs. |
| error_description |Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |

Si vous recevez cette erreur dans la requête iFrame, l’utilisateur doit se connecter de nouveau de manière interactive, ceci pour récupérer un nouveau jeton.

## <a name="refresh-tokens"></a>Jetons d’actualisation
Les jetons d’ID et les jetons d’accès expirent après une courte période de temps. Votre application doit actualiser ces jetons périodiquement.  Pour actualiser chaque type de jeton, exécutez la même demande d’IFrame masqué que dans l’exemple précédent, en utilisant le paramètre `prompt=none` pour contrôler les étapes d’Azure AD.  Pour recevoir une nouvelle valeur `id_token`, utilisez `response_type=id_token` et `scope=openid`, ainsi qu’un paramètre `nonce`.

## <a name="send-a-sign-out-request"></a>Envoi d’une demande de déconnexion
Lorsque vous souhaitez déconnecter l’utilisateur de l’application, redirigez l’utilisateur vers Azure AD pour effectuer cette déconnexion. Si vous ne le faites pas, l’utilisateur pourrait être en mesure de se réauthentifier à votre application sans avoir à saisir de nouveau ses informations d’identification, car il dispose d’une session d’authentification unique valide auprès d’Azure AD.

Vous pouvez simplement rediriger l’utilisateur vers le `end_session_endpoint` qui est répertorié dans le même document de métadonnées OpenID Connect décrit dans [Validation du jeton d’ID](#validate-the-id-token). Par exemple :

```http
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/logout?post_logout_redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
```

| Paramètre | Obligatoire | Description |
| --------- | -------- | ----------- |
| {tenant} | Oui | Nom de votre locataire Azure AD B2C |
| {policy} | Oui | Flux utilisateur que vous voulez utiliser pour déconnecter l’utilisateur de votre application. |
| post_logout_redirect_uri | Non  | URL vers laquelle l’utilisateur doit être redirigé après la déconnexion. Si elle n’est pas incluse, Azure AD B2C affiche un message générique à l’utilisateur. |
| state | Non  | Si un paramètre `state` est inclus dans la demande, la même valeur doit apparaître dans la réponse. L’application doit vérifier que les valeurs `state` de la demande et de la réponse sont identiques. |


> [!NOTE]
> La redirection de l’utilisateur vers le `end_session_endpoint` efface certains états de l’authentification unique de l’utilisateur auprès d’Azure AD B2C. Il ne déconnecte cependant pas l’utilisateur de la session du fournisseur d’identité sociale de l’utilisateur. Si l’utilisateur sélectionne le même fournisseur d’identité lors d’une connexion ultérieure, il sera réauthentifié sans avoir à entrer ses informations d’identification. Si un utilisateur veut se déconnecter de votre application Azure AD B2C, cela ne signifie pas nécessairement qu’il veut se déconnecter complètement, par exemple de son compte Facebook. Cependant, pour des comptes locaux, la session de l’utilisateur sera terminée correctement.
>

## <a name="next-steps"></a>Étapes suivantes

### <a name="code-sample-azure-ad-b2c-with-microsoft-authentication-library-for-javascript"></a>Exemple de code : Azure AD B2C avec la bibliothèque d’authentification Microsoft pour JavaScript

[Application monopage élaborée avec msal.js pour Azure AD B2C][github-msal-js-example] (GitHub)

La finalité de cet exemple sur GitHub est de vous aider à démarrer votre utilisation d’Azure AD B2C dans une application web simple s’appuyant sur [msal.js][github-msal-js] et utilisant l’authentification avec fenêtre contextuelle.

<!-- Links - EXTERNAL -->
[github-msal-js-example]: https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp
[github-msal-js]: https://github.com/AzureAD/microsoft-authentication-library-for-js
