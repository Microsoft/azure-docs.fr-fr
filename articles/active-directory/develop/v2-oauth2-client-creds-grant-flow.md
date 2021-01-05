---
title: Flux des informations d’identification du client OAuth 2.0 et plateforme d’identités Microsoft | Azure
description: Créez des applications web à l’aide de l’implémentation de la plateforme d’identités Microsoft du protocole d’authentification OAuth 2.0.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 10/2/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 89a4c62044e3be849650de703d2daa9ca3e2a975
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932581"
---
# <a name="microsoft-identity-platform-and-the-oauth-20-client-credentials-flow"></a>Plateforme d’identités Microsoft et flux d’informations d’identification du client OAuth 2.0

Vous pouvez utiliser [l’octroi des informations d’identification du client OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-4.4) spécifié dans RFC 6749, parfois appelé *OAuth à deux branches* pour accéder à des ressources hébergées sur le web à l’aide de l’identité d’une application. Ce type d’octroi est couramment utilisé pour les interactions de serveur à serveur qui doivent s’exécuter en arrière-plan sans l’interaction immédiate d’un utilisateur. Ces types d’application sont souvent appelés *démons* (daemons) ou *comptes de service*.

Cet article explique comment programmer directement par rapport au protocole dans votre application. Dans la mesure du possible, nous vous recommandons d’utiliser les bibliothèques d’authentification Microsoft (MSAL) prises en charge au lieu d’[acquérir des jetons et d’appeler des API web sécurisées](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Jetez également un coup d’œil aux [exemples d’applications qui utilisent MSAL](sample-v2-code.md).

Le flux d’octroi des informations d’identification du client OAuth 2.0 permet à un service web (client confidentiel) d’utiliser ses propres informations d’identification pour s’authentifier lorsqu’il appelle un autre service web, au lieu d’emprunter l’identité d’un utilisateur. Dans ce scénario, le client est généralement un service web de niveau intermédiaire, un service démon ou un site web. Pour augmenter le niveau d’assurance, la plateforme d’identités Microsoft autorise également le service d’appel à utiliser un certificat (au lieu d’un secret partagé) comme une information d’identification.

Dans le flux des informations d’identification du client, les autorisations sont accordées directement à l’application elle-même par l’administrateur. Lorsque l’application présente un jeton à une ressource, la ressource impose que l’application elle-même, et non pas l'utilisateur (puisqu’il n’est pas impliqué, ait l’autorisation d’effectuer une action.  Cet article décrit les étapes nécessaires pour [autoriser une application à appeler une API](#application-permissions), ainsi que [comment récupérer les jetons nécessaires pour appeler cette API](#get-a-token).

## <a name="protocol-diagram"></a>Schéma de protocole

Le flux d’informations d’identification client complet est similaire à l’illustration suivante. Nous décrirons en détail chacune des étapes plus loin dans cet article.

![Diagramme montrant le flux d’informations d’identification du client](./media/v2-oauth2-client-creds-grant-flow/convergence-scenarios-client-creds.svg)

## <a name="get-direct-authorization"></a>Obtenir l’autorisation directe

Une application reçoit généralement l’autorisation directe d’accéder à une ressource de deux manières :

* [Via une liste de contrôle d’accès (ACL) sur la ressource](#access-control-lists)
* [Via l’attribution de l’autorisation d’application dans Azure AD](#application-permissions)

Ces deux méthodes sont les plus courantes dans Azure AD et sont recommandées pour les clients et les ressources qui exécutent le flux des informations d’identification du client. Une ressource peut également choisir d’autoriser ses clients d’autres manières. Chaque serveur de ressources peut choisir la méthode la plus logique pour son application.

### <a name="access-control-lists"></a>Listes de contrôle d'accès

Un fournisseur de ressources peut appliquer une vérification d’autorisation basée sur une liste d’ID d’application (client) qu’il connaît et octroyer un niveau d’accès spécifique. Lorsque la ressource reçoit un jeton à partir du point de terminaison de la plateforme d’identités Microsoft, elle peut décoder le jeton et extraire l’ID d’application du client à partir des revendications `appid` et `iss`. Elle compare ensuite l’application par rapport à une liste de contrôle d'accès qu’elle gère. La granularité et la méthode ACL peuvent varier considérablement entre les ressources.

Un cas d’utilisation typique consiste à utiliser une liste ACL afin d'exécuter des tests pour une application web ou une API Web. L’API Web peut accorder uniquement un sous-ensemble d'autorisations complètes à un client spécifique. Pour exécuter des tests de bout en bout sur l’API, créez un client test qui acquiert des jetons à partir du point de terminaison de la plateforme d’identités Microsoft, puis envoie ceux-ci à l’API. L’API vérifie ensuite l’ID d’application du client de test dans la liste de contrôle d'accès pour lui accorder un accès complet à toutes les fonctionnalités de l’API. Si vous utilisez ce type de liste de contrôle d'accès, veillez à valider non seulement la valeur `appid` de l'appelant, mais également que la valeur `iss` du jeton est approuvée.

Ce type d’autorisation est courant pour les démons et les comptes de service qui doivent accéder à des données qui appartiennent à des utilisateurs avec des comptes Microsoft personnels. Pour les données appartenant à des organisations, nous vous recommandons d’acquérir l’autorisation requise via les autorisations de l’application.

#### <a name="controlling-tokens-without-the-roles-claim"></a>Contrôle des jetons sans la revendication `roles`

Pour activer ce modèle d’autorisation par listes ACL, Azure AD n’impose pas que les applications soient autorisées à obtenir des jetons pour une autre application. Les jetons de type application uniquement peuvent ainsi être émis sans revendication `roles`. Les applications qui exposent des API doivent implémenter des vérifications d’autorisation afin d’accepter les jetons.

Si vous souhaitez empêcher les applications d’obtenir des jetons d’accès de type application uniquement sans rôle pour votre application, [veillez à ce que les exigences d’affectation d’utilisateurs soient activées pour votre application](../manage-apps/assign-user-or-group-access-portal.md#configure-an-application-to-require-user-assignment). Ainsi, les utilisateurs et les applications dépourvus de rôles ne pourront pas récupérer de jeton pour cette application. 

### <a name="application-permissions"></a>Autorisations de l’application

Au lieu d’utiliser des listes ACL, vous pouvez utiliser des API pour exposer un ensemble d’**autorisations d’application**. Une autorisation de l’application est accordée à une application par un administrateur d’une organisation et peut uniquement être utilisée pour accéder aux données appartenant à cette organisation et ses employés. Par exemple, Microsoft Graph expose plusieurs autorisations d’application pour effectuer les opérations suivantes :

* Lire les messages dans toutes les boîtes aux lettres
* Lire et écrire des messages dans toutes les boîtes aux lettres
* Envoyer des messages en tant que n’importe quel utilisateur
* Lire les données du répertoire

Pour utiliser des autorisations d’application avec votre propre API (par opposition à Microsoft Graph), vous devez d’abord [exposer l’API](quickstart-configure-app-expose-web-apis.md) en définissant des étendues dans l’inscription de l’application de l’API dans le Portail Azure. Ensuite, [configurer l’accès à l’API](quickstart-configure-app-access-web-apis.md) en sélectionnant ces autorisations dans l’inscription de l’application de votre application cliente. Si vous n’avez pas exposé d’étendues dans l’inscription de l’application de votre API, vous ne pouvez pas spécifier d’autorisations d’application pour cette API dans l’inscription de l’application de votre application cliente dans le Portail Azure.

Lors de l’authentification en tant qu’application (par opposition à l’authentification avec un utilisateur), vous ne pouvez pas utiliser *d’autorisations déléguées*, étendues accordées par un utilisateur. Vous devez utiliser des autorisations d’application, également appelées « rôles », accordées par un administrateur pour l’application ou par le biais d’une pré-autorisation par l’API web.

Pour en savoir plus sur les autorisations d’application, consultez [Autorisation et Consentement](v2-permissions-and-consent.md#permission-types).

#### <a name="recommended-sign-the-user-into-your-app"></a>Recommandé : connecter l’utilisateur à votre application

En général, lorsque vous créez une application qui utilise des autorisations d’application, l’application doit disposer d’une page/vue qui permet à l’administrateur d’approuver les autorisations de l’application. Cette page peut faire partie du flux d’inscription de l’application, des paramètres de l’application ou d’un flux de connexion dédié. Dans de nombreux cas, il est judicieux pour l’application d’afficher la vue de « connexion » uniquement après qu’un utilisateur se soit connecté avec un compte Microsoft professionnel ou scolaire.

Si vous connectez l’utilisateur à votre application vous pouvez identifier l’organisation à laquelle l’utilisateur appartient avant de lui demander d’approuver les autorisations d’application. Bien que cela ne soit pas strictement nécessaire, cela peut vous aider à créer une expérience plus intuitive pour vos utilisateurs. Pour connecter l’utilisateur, suivez nos [tutoriels sur le protocole de la plateforme d’identités Microsoft](active-directory-v2-protocols.md).

#### <a name="request-the-permissions-from-a-directory-admin"></a>Demander les autorisations à un administrateur d’annuaire

Lorsque vous êtes prêt à demander les autorisations à l’administrateur de l’organisation, vous pouvez rediriger l’utilisateur vers le *point de terminaison de consentement administrateur* de la plateforme d’identités Microsoft.

> [!TIP]
> Essayez d'exécuter cette requête dans Postman ! Utilisez votre propre ID d’application pour de meilleurs résultats. L’application du tutoriel ne vous demande pas d’autorisations utiles. [![Essayez d’exécuter cette requête dans Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

Conseil pro : Essayez de coller la requête suivante dans un navigateur.

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| Paramètre | Condition | Description |
| --- | --- | --- |
| `tenant` | Obligatoire | Le client d’annuaire auquel vous souhaitez demander l’autorisation. Peut être au format GUID ou sous forme de nom convivial. Si vous ne savez pas à quel client appartient l’utilisateur et si vous souhaitez lui permettre de se connecter avec n’importe quel client, utilisez `common`. |
| `client_id` | Obligatoire | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `redirect_uri` | Obligatoire | L'URI de redirection où vous souhaitez que la réponse soit envoyée pour être gérée par votre application. Il doit correspondre exactement à l’un des URI de redirection enregistrés dans le portail, auquel s’ajoute le codage dans une URL, et peut avoir des segments de chemin d’accès supplémentaires. |
| `state` | Recommandé | Une valeur incluse dans la requête qui est également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. La valeur d’état est utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné. |

À ce stade, Azure AD impose que seul un administrateur de locataire peut se connecter pour terminer la demande. L’administrateur est invité à approuver toutes les autorisations directes d’application demandées pour votre application dans le portail d’inscription des applications.

##### <a name="successful-response"></a>Réponse correcte

Si l’administrateur approuve les autorisations pour votre application, la réponse correcte sera :

```HTTP
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| Paramètre | Description |
| --- | --- |
| `tenant` | Le client d’annuaire ayant accordé à votre application les autorisations demandées, au format GUID. |
| `state` | Une valeur incluse dans la requête, qui est également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. La valeur d’état est utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné. |
| `admin_consent` | Définissez sur **True**. |

##### <a name="error-response"></a>Réponse d’erreur

Si l’administrateur n’approuve pas les autorisations pour votre application, la réponse d’échec ressemble à ce qui suit :

```HTTP
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| Paramètre | Description |
| --- | --- |
| `error` | Une chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreur et pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur. |

Une fois que vous avez reçu une réponse correcte du point de terminaison de mise en service de l’application, votre application a acquis les autorisations directes d’application qu’elle avait demandées. Vous pouvez désormais demander un jeton pour la ressource que vous souhaitez.

## <a name="get-a-token"></a>Obtention d’un jeton

Une fois que vous avez acquis l’autorisation nécessaire pour votre application, passez à l’acquisition des jetons d’accès pour les API. Pour obtenir un jeton à l’aide de l’octroi des informations d’identification du client, envoyez une demande POST au point de terminaison de la plateforme d’identités Microsoft `/token` :

> [!TIP]
> Essayez d'exécuter cette requête dans Postman ! Utilisez votre propre ID d’application pour de meilleurs résultats. L’application du tutoriel ne vous demande pas d’autorisations utiles. [![Essayez d’exécuter cette requête dans Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Premier cas : Requête de jeton d’accès avec un secret partagé

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

```Bash
# Replace {tenant} with your tenant!
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token'
```

| Paramètre | Condition | Description |
| --- | --- | --- |
| `tenant` | Obligatoire | Le locataire de l’annuaire sur lequel les plans d’application opèrent, au format GUID ou nom de domaine. |
| `client_id` | Obligatoire | Copiez l’ID d’application affecté à votre application. Ces informations sont disponibles dans le portail où vous avez inscrit votre application. |
| `scope` | Obligatoire | La valeur transmise pour le paramètre `scope` dans cette demande doit être l’identificateur de ressource (URI ID d’application) de la ressource souhaitée, avec le suffixe `.default`. Dans l’exemple Microsoft Graph, la valeur est `https://graph.microsoft.com/.default`. <br/>Cette valeur indique au point de terminaison de la plateforme d’identités Microsoft que parmi toutes les autorisations directes d’application que vous avez configurées pour votre application, le point de terminaison doit émettre un jeton pour celles associées à la ressource que vous souhaitez utiliser. Pour en savoir plus sur l’étendue `/.default`, consultez la [documentation sur le consentement](v2-permissions-and-consent.md#the-default-scope). |
| `client_secret` | Obligatoire | La clé secrète client que vous avez générée pour votre application dans le portail d’inscription des applications. Le secret du client doit être codé en URL avant d’être envoyé. |
| `grant_type` | Obligatoire | Cette propriété doit être définie sur `client_credentials`. |

### <a name="second-case-access-token-request-with-a-certificate"></a>Deuxième cas : Requête de jeton d’accès avec un certificat

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

| Paramètre | Condition | Description |
| --- | --- | --- |
| `tenant` | Obligatoire | Le locataire de l’annuaire sur lequel les plans d’application opèrent, au format GUID ou nom de domaine. |
| `client_id` | Obligatoire |L'ID d'application (client) affecté à votre application. |
| `scope` | Obligatoire | La valeur transmise pour le paramètre `scope` dans cette demande doit être l’identificateur de ressource (URI ID d’application) de la ressource souhaitée, avec le suffixe `.default`. Dans l’exemple Microsoft Graph, la valeur est `https://graph.microsoft.com/.default`. <br/>Cette valeur indique au point de terminaison de la plateforme d’identités Microsoft que parmi toutes les autorisations directes d’application que vous avez configurées pour votre application, il doit émettre un jeton pour celles associées à la ressource que vous souhaitez utiliser. Pour en savoir plus sur l’étendue `/.default`, consultez la [documentation sur le consentement](v2-permissions-and-consent.md#the-default-scope). |
| `client_assertion_type` | Obligatoire | Elle doit avoir la valeur `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| `client_assertion` | Obligatoire | Assertion (jeton Web JSON) dont vous avez besoin pour créer et signer avec le certificat inscrit comme informations d’identification pour votre application. Pour découvrir comment inscrire votre certificat et le format de l’assertion, consultez la rubrique traitant des [informations d’identification des certificats](active-directory-certificate-credentials.md).|
| `grant_type` | Obligatoire | Cette propriété doit être définie sur `client_credentials`. |

Notez que les paramètres sont presque les mêmes que dans le cas de la demande par secret partagé, sauf que le paramètre client_secret est remplacé par deux paramètres : client_assertion_type et client_assertion.

### <a name="successful-response"></a>Réponse correcte

Une réponse correcte se présente ainsi :

```json
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
}
```

| Paramètre | Description |
| --- | --- |
| `access_token` | Le jeton d’accès demandé. L’application peut utiliser ce jeton pour procéder à l’authentification sur la ressource sécurisée, par exemple une API Web. |
| `token_type` | Indique la valeur du type de jeton. Le seul type pris en charge par la plateforme d’identités Microsoft est `bearer`. |
| `expires_in` | Durée de validité du jeton d’accès (en secondes). |

### <a name="error-response"></a>Réponse d’erreur

Une réponse d’erreur ressemble à ceci :

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/.default is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Paramètre | Description |
| --- | --- |
| `error` | Une chaîne de code d’erreur que vous pouvez utiliser pour classer les types d’erreur se produisant et pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut vous aider à identifier la cause principale d’une erreur d’authentification. |
| `error_codes` | Liste des codes d’erreur STS spécifiques pouvant être utiles dans les tests de diagnostic. |
| `timestamp` | Heure à laquelle l'erreur s'est produite. |
| `trace_id` | Identifiant unique de la demande pour faciliter les tests de diagnostic. |
| `correlation_id` | Identifiant unique de la demande pour faciliter les tests de diagnostic sur les différents composants. |

## <a name="use-a-token"></a>Utilisation d’un jeton

Maintenant que vous avez acquis un jeton, utilisez-le pour effectuer des demandes auprès de la ressource. Lorsque le jeton expire, répétez la demande auprès du point de terminaison `/token` pour acquérir un nouveau jeton d’accès.

```HTTP
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

```bash
# Pro tip: Try the following command! (Replace the token with your own.)

curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbG...." 'https://graph.microsoft.com/v1.0/me/messages'
```

## <a name="code-samples-and-other-documentation"></a>Exemples de code et autres documents

Lisez la [documentation de présentation des informations d’identification du client](https://aka.ms/msal-net-client-credentials) dans la bibliothèque d’authentification Microsoft.

| Exemple | Plateforme |Description |
|--------|----------|------------|
|[active-directory-dotnetcore-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) | Console .NET Core 2.1 | Application.NET Core simple qui affiche les utilisateurs d’un locataire interrogeant Microsoft Graph en utilisant l’identité de l’application plutôt que le compte d’un utilisateur. L’exemple illustre également la variation en utilisant des certificats pour l’authentification. |
|[active-directory-dotnet-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)|ASP.NET MVC | Application web qui synchronise les données Microsoft Graph en utilisant l’identité de l’application plutôt que le compte d’un utilisateur. |
