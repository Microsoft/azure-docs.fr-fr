---
title: Plateforme d’identités Microsoft et flux On-Behalf-Of OAuth 2.0 | Azure
titleSuffix: Microsoft identity platform
description: Cet article explique comment utiliser des messages HTTP pour implémenter l’authentification de service à service en utilisant le flux Pour le compte de OAuth 2.0.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/7/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 018d67b3e4e730cd46eb524a8927b3a6d68d74e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88958658"
---
# <a name="microsoft-identity-platform-and-oauth-20-on-behalf-of-flow"></a>Plateforme d’identités Microsoft et flux On-Behalf-Of OAuth 2.0


Le flux On-Behalf-Of (OBO) OAuth 2.0 répond au cas d’usage dans le cadre duquel une application appelle un service/une API web qui, à son tour, doit appeler un autre service/une autre API web. L’idée est de propager l’identité et les autorisations de l’utilisateur délégué via la chaîne de la demande. Pour que le service de niveau intermédiaire puisse faire des demandes authentifiées au service en aval, il doit sécuriser un jeton d’accès de la plateforme d’identité Microsoft pour le compte de l’utilisateur.

Cet article explique comment programmer directement par rapport au protocole dans votre application.  Dans la mesure du possible, nous vous recommandons d’utiliser les bibliothèques d’authentification Microsoft (MSAL) prises en charge au lieu d’[acquérir des jetons et d’appeler des API web sécurisées](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Jetez également un coup d’œil aux [exemples d’applications qui utilisent MSAL](sample-v2-code.md).

> [!NOTE]
> Depuis mai 2018, il n’est pas possible d’utiliser un jeton `id_token` dérivé du flux implicite pour le flux OBO. Les applications à une seule page doivent passer un jeton d’**accès** à un client confidentiel de niveau intermédiaire pour effectuer des flux OBO à la place. Pour plus d’informations sur les clients pouvant effectuer des appels OBO, consultez [Limitations](#client-limitations).

## <a name="protocol-diagram"></a>Schéma de protocole

Supposons que l’utilisateur ait été authentifié dans une application à l’aide du [flux d’octroi de code d’autorisation OAuth 2.0](v2-oauth2-auth-code-flow.md) ou d’un autre flux de connexion. À ce stade, l’application a un jeton d’accès *pour l’API A* (jeton A) avec les revendications et le consentement de l’utilisateur pour accéder à l’API web de niveau intermédiaire (API A). L’API A doit maintenant faire une demande authentifiée à l’API web en aval (API B).

Les étapes qui suivent constituent le flux OBO et sont décrites à l’aide du diagramme suivant.

![Montre le flux On-Behalf-Of OAuth 2.0](./media/v2-oauth2-on-behalf-of-flow/protocols-oauth-on-behalf-of-flow.png)

1. L’application cliente adresse une demande à l’API A avec le jeton A (avec une revendication d’API A `aud`).
1. L’API A s’authentifie auprès du point de terminaison d’émission de jeton de la plateforme d’identités Microsoft et demande un jeton pour accéder à l’API B.
1. Le point de terminaison d’émission de jeton de la plateforme d’identités Microsoft valide les informations d’identification de l’API A avec le jeton A et envoie le jeton d’accès pour l’API B (jeton B) à l’API A.
1. Le jeton B est défini par l’API A dans l’en-tête d’autorisation de la requête adressée à l’API B.
1. Les données de la ressource sécurisée sont retournées par l’API B à l’API A, puis au client.

> [!NOTE]
> Dans ce scénario, le service de niveau intermédiaire n’a aucune interaction utilisateur pour obtenir le consentement de l’utilisateur pour accéder à l’API en aval. Par conséquent, l’option d’accorder l’accès à l’API en aval est présentée au préalable lors de l’étape de consentement pendant l’authentification. Pour savoir comment effectuer cette configuration pour votre application, consultez [Obtention du consentement pour l’application de niveau intermédiaire](#gaining-consent-for-the-middle-tier-application).

## <a name="middle-tier-access-token-request"></a>Demande de jeton d’accès de niveau intermédiaire

Pour demander un jeton d’accès, adressez une requête HTTP POST au point de terminaison du jeton de la plateforme d’identités Microsoft spécifique au locataire, avec les paramètres suivants.

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token
```

Deux cas de figure se présentent, selon que l’application cliente choisit d’être sécurisée par un secret partagé ou un certificat.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Premier cas : Requête de jeton d’accès avec un secret partagé

Lorsque l’application utilise un secret partagé, la demande de jeton d’accès de service à service contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| `grant_type` | Obligatoire | Type de la demande de jeton. Pour une demande à l’aide d’un JWT, la valeur doit être `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Obligatoire | L’ID (client) d’application attribué à votre application par la page [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `client_secret` | Obligatoire | La clé secrète client que vous avez générée pour votre application sur la page Inscriptions d’applications du portail Azure. |
| `assertion` | Obligatoire | Jeton d’accès qui a été envoyé à l’API de niveau intermédiaire.  Ce jeton doit comporter une revendication d’audience (`aud`) de l’application qui effectue cette requête OBO (l’application indiquée par le champ `client-id`). Les applications ne peuvent pas accepter un jeton pour une autre application. (Par exemple, si un client envoie à une API un jeton pour MS Graph, l’API ne peut pas accepter ce jeton avec OBO.  Au lieu de cela, il doit refuser le jeton.)  |
| `scope` | Obligatoire | Liste des étendues (séparées par des espaces) pour la demande de jeton. Pour plus d’informations, consultez [Étendues](v2-permissions-and-consent.md). |
| `requested_token_use` | Obligatoire | Spécifie comment la demande doit être traitée. Dans le flux OBO, la valeur doit être définie sur `on_behalf_of`. |

#### <a name="example"></a>Exemple

La requête HTTP POST suivante demande un jeton d’accès et un jeton d’actualisation avec l’étendue `user.read` pour l’API web https://graph.microsoft.com.

```HTTP
//line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&client_id=2846f71b-a7a4-4987-bab3-760035b2f389
&client_secret=BYyVnAt56JpLwUcyo47XODd
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiIyO{a lot of characters here}
&scope=https://graph.microsoft.com/user.read+offline_access
&requested_token_use=on_behalf_of
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Deuxième cas : Requête de jeton d’accès avec un certificat

Une demande de jeton d’accès de service à service avec un certificat contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| `grant_type` | Obligatoire | Type de la demande de jeton. Pour une demande à l’aide d’un JWT, la valeur doit être `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Obligatoire |  L’ID (client) d’application attribué à votre application par la page [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `client_assertion_type` | Obligatoire | La valeur doit être `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| `client_assertion` | Obligatoire | Assertion (jeton Web JSON) dont vous avez besoin pour créer et signer avec le certificat inscrit comme informations d’identification pour votre application. Pour découvrir comment inscrire votre certificat et le format de l’assertion, consultez [Informations d’identification de certificat](active-directory-certificate-credentials.md). |
| `assertion` | Obligatoire |  Jeton d’accès qui a été envoyé à l’API de niveau intermédiaire.  Ce jeton doit comporter une revendication d’audience (`aud`) de l’application qui effectue cette requête OBO (l’application indiquée par le champ `client-id`). Les applications ne peuvent pas accepter un jeton pour une autre application. (Par exemple, si un client envoie à une API un jeton pour MS Graph, l’API ne peut pas accepter ce jeton avec OBO.  Au lieu de cela, il doit refuser le jeton.)  |
| `requested_token_use` | Obligatoire | Spécifie comment la demande doit être traitée. Dans le flux OBO, la valeur doit être définie sur `on_behalf_of`. |
| `scope` | Obligatoire | Liste des étendues (séparées par des espaces) pour la demande de jeton. Pour plus d’informations, consultez [Étendues](v2-permissions-and-consent.md).|

Notez que les paramètres sont presque les mêmes que dans le cas de la demande par secret partagé, sauf que le paramètre `client_secret` est remplacé par deux paramètres : `client_assertion_type` et `client_assertion`.

#### <a name="example"></a>Exemple

La requête HTTP POST suivante demande un jeton d’accès avec l’étendue `user.read` pour l’API web https://graph.microsoft.com avec un certificat.

```HTTP
// line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiO{a lot of characters here}
&requested_token_use=on_behalf_of
&scope=https://graph.microsoft.com/user.read+offline_access
```

## <a name="middle-tier-access-token-response"></a>Réponse à une demande de jeton d’accès de niveau intermédiaire

Une réponse correspondant à une réussite est une réponse JSON OAuth 2.0 avec les paramètres suivants.

| Paramètre | Description |
| --- | --- |
| `token_type` | Indique la valeur du type de jeton. Le seul type pris en charge par la plateforme d’identités Microsoft est `Bearer`. Pour plus d’informations sur les jetons du porteur, consultez [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt). |
| `scope` | Étendue de l’accès accordé dans le jeton. |
| `expires_in` | Durée de validité, en secondes, du jeton d’accès. |
| `access_token` | Le jeton d’accès demandé. Le service web appelant peut utiliser ce jeton pour s’authentifier auprès du service destinataire. |
| `refresh_token` | Jeton d’actualisation pour le jeton d’accès demandé. Le service appelant peut utiliser ce jeton pour demander un autre jeton d’accès après l’expiration du jeton d’accès actuel. Le jeton d’actualisation est uniquement fourni si l’étendue `offline_access` a été demandée. |

### <a name="success-response-example"></a>Exemple de réponse correspondant à une réussite

L’exemple suivant illustre une réponse affirmative à une demande de jeton d’accès pour l’API web https://graph.microsoft.com.

```json
{
  "token_type": "Bearer",
  "scope": "https://graph.microsoft.com/user.read",
  "expires_in": 3269,
  "ext_expires_in": 0,
  "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6IkFRQUJBQUFBQUFCbmZpRy1tQTZOVGFlN0NkV1c3UWZkQ0NDYy0tY0hGa18wZE50MVEtc2loVzRMd2RwQVZISGpnTVdQZ0tQeVJIaGlDbUN2NkdyMEpmYmRfY1RmMUFxU21TcFJkVXVydVJqX3Nqd0JoN211eHlBQSIsImFsZyI6IlJTMjU2IiwieDV0IjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIiwia2lkIjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIn0.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaWF0IjoxNDkzOTMwMzA1LCJuYmYiOjE0OTM5MzAzMDUsImV4cCI6MTQ5MzkzMzg3NSwiYWNyIjoiMCIsImFpbyI6IkFTUUEyLzhEQUFBQU9KYnFFWlRNTnEyZFcxYXpKN1RZMDlYeDdOT29EMkJEUlRWMXJ3b2ZRc1k9IiwiYW1yIjpbInB3ZCJdLCJhcHBfZGlzcGxheW5hbWUiOiJUb2RvRG90bmV0T2JvIiwiYXBwaWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJhcHBpZGFjciI6IjEiLCJmYW1pbHlfbmFtZSI6IkNhbnVtYWxsYSIsImdpdmVuX25hbWUiOiJOYXZ5YSIsImlwYWRkciI6IjE2Ny4yMjAuMC4xOTkiLCJuYW1lIjoiTmF2eWEgQ2FudW1hbGxhIiwib2lkIjoiZDVlOTc5YzctM2QyZC00MmFmLThmMzAtNzI3ZGQ0YzJkMzgzIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTIxMjc1MjExODQtMTYwNDAxMjkyMC0xODg3OTI3NTI3LTI2MTE4NDg0IiwicGxhdGYiOiIxNCIsInB1aWQiOiIxMDAzM0ZGRkEwNkQxN0M5Iiwic2NwIjoiVXNlci5SZWFkIiwic3ViIjoibWtMMHBiLXlpMXQ1ckRGd2JTZ1JvTWxrZE52b3UzSjNWNm84UFE3alVCRSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInVuaXF1ZV9uYW1lIjoibmFjYW51bWFAbWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hY2FudW1hQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJWR1ItdmtEZlBFQ2M1dWFDaENRSkFBIiwidmVyIjoiMS4wIn0.cubh1L2VtruiiwF8ut1m9uNBmnUJeYx4x0G30F7CqSpzHj1Sv5DCgNZXyUz3pEiz77G8IfOF0_U5A_02k-xzwdYvtJUYGH3bFISzdqymiEGmdfCIRKl9KMeoo2llGv0ScCniIhr2U1yxTIkIpp092xcdaDt-2_2q_ql1Ha_HtjvTV1f9XR3t7_Id9bR5BqwVX5zPO7JMYDVhUZRx08eqZcC-F3wi0xd_5ND_mavMuxe2wrpF-EZviO3yg0QVRr59tE3AoWl8lSGpVc97vvRCnp4WVRk26jJhYXFPsdk4yWqOKZqzr3IFGyD08WizD_vPSrXcCPbZP3XWaoTUKZSNJg",
  "refresh_token": "OAQABAAAAAABnfiG-mA6NTae7CdWW7QfdAALzDWjw6qSn4GUDfxWzJDZ6lk9qRw4An{a lot of characters here}"
}
```

> [!NOTE]
> Le jeton d’accès ci-dessus est un jeton au format v1.0 pour Microsoft Graph. En effet, le format dépend de la **ressource** consultée et n’est pas lié aux points de terminaison utilisés pour le demander. Comme Microsoft Graph est configuré pour accepter des jetons v1.0, la plateforme d’identités Microsoft génère des jetons d’accès v1.0 quand un client demande des jetons pour Microsoft Graph. D’autres applications peuvent indiquer qu’elles veulent des jetons au format v2.0, au format v1.0 ou même dans un format propriétaire ou chiffré.  Les points de terminaison v1.0 et v2.0 peuvent émettre des jetons aux deux formats : ainsi, la ressource reçoit toujours le bon format, quels que soient la façon dont le jeton a été demandé par le client et l’endroit. 
>
> Seules les applications doivent examiner les jetons d’accès. Les clients **ne doivent pas** les inspecter. Si vous inspectez les jetons d’accès pour d’autres applications dans votre code, l’application subira une interruption inattendue lorsqu’elle modifiera le format de ses jetons ou commencera à les chiffrer. 

### <a name="error-response-example"></a>Exemple de réponse d’erreur

Une réponse d’erreur est retournée par le point de terminaison du jeton lors de la tentative d’acquisition d’un jeton d’accès pour l’API en aval si une stratégie d’accès conditionnel comme l’[authentification multifacteur](../authentication/concept-mfa-howitworks.md) est définie sur cette API. Le service de niveau intermédiaire doit faire apparaître cette erreur à l’application cliente afin que celle-ci puisse fournir une interaction utilisateur pour satisfaire la stratégie d’accès conditionnel.

```json
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>Utiliser le jeton d’accès pour accéder à la ressource sécurisée

Le service de niveau intermédiaire peut maintenant utiliser le jeton obtenu ci-dessus pour faire des demandes authentifiées à l’API web en aval, en définissant le jeton dans l’en-tête `Authorization`.

### <a name="example"></a>Exemple

```HTTP
GET /v1.0/me HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>Assertions SAML obtenues avec un flux OBO OAuth 2.0

Certains services web basés sur OAuth doivent accéder à d’autres API de service web qui acceptent les instructions d’assertion SAML dans des flux non interactifs. Azure Active Directory peut fournir une instruction d’assertion SAML en réponse à un flux On-Behalf-Of qui utilise un service web basé sur SAML comme ressource cible.

>[!NOTE]
>Il s’agit d’une extension non standard pour le flux On-Behalf-Of OAuth 2.0 qui permet à une application OAuth2 d’accéder aux points de terminaison d’API du service web qui consomment des jetons SAML.

> [!TIP]
> Quand vous appelez un service web protégé par SAML à partir d’une application web front-end, vous pouvez simplement appeler l’API et lancer un flux d’authentification interactif normal avec la session existante de l’utilisateur. Vous devez seulement utiliser un flux OBO quand un appel de service à service nécessite un jeton SAML pour fournir le contexte de l’utilisateur.

## <a name="gaining-consent-for-the-middle-tier-application"></a>Obtention du consentement pour l’application de niveau intermédiaire

Selon l’architecture ou l’utilisation de votre application, vous pouvez envisager différentes stratégies pour garantir la réussite du flux OBO. Dans tous les cas, l’objectif ultime est de garantir qu’un consentement formel a été donné pour que l’application cliente puisse appeler l’application de niveau intermédiaire, et que l’application de niveau intermédiaire soit autorisée à appeler la ressource back-end.

> [!NOTE]
> Auparavant, le système de compte Microsoft (comptes personnels) ne prenait pas en charge le champ « Application cliente connue » et n’affichait pas non plus le consentement combiné.  Ces deux fonctionnalités ont été ajoutées et toutes les applications de la plateforme d’identités Microsoft peuvent désormais utiliser l’approche d’application cliente connue pour obtenir le consentement nécessaire aux appels OBO.

### <a name="default-and-combined-consent"></a>Étendue /.default et consentement combiné

L’application de niveau intermédiaire ajoute le client à la liste des applications clientes connues dans son manifeste et le client peut alors déclencher un flux de consentement combiné pour lui-même et l’application de niveau intermédiaire. Sur le point de terminaison de la plateforme d’identités Microsoft, [l’étendue `/.default`](v2-permissions-and-consent.md#the-default-scope) est utilisée. Lors du déclenchement d’un écran de consentement à l’aide des applications clientes connues et de `/.default`, l’écran de consentement affiche les autorisations **à la fois** pour le client et l’API de niveau intermédiaire, et demande également toutes les autorisations exigées par l’API de niveau intermédiaire. L’utilisateur fournit le consentement pour les deux applications et le flux OBO fonctionne ensuite.

### <a name="pre-authorized-applications"></a>Applications préalablement autorisées

Une ressource peut indiquer qu’une application donnée a toujours l’autorisation de recevoir certaines étendues. Cela est particulièrement utile pour rendre les connexions entre un client front-end et une ressource back-end plus fluides. Une ressource peut déclarer plusieurs applications préalablement autorisées : n’importe quelle application de ce type peut demander ces autorisations dans un flux OBO et les recevoir sans que l’utilisateur ne fournisse un consentement.

### <a name="admin-consent"></a>Consentement de l’administrateur

Un administrateur de locataire peut garantir que les applications ont l’autorisation d’appeler leurs API requises en fournissant le consentement de l’administrateur pour l’application de niveau intermédiaire. Pour ce faire, l’administrateur peut trouver l’application de niveau intermédiaire dans son locataire, ouvrir la page des autorisations nécessaires et choisir d’accorder l’autorisation pour l’application. Pour en savoir plus sur le consentement de l’administrateur, consultez la [documentation sur le consentement et les autorisations](v2-permissions-and-consent.md).

### <a name="use-of-a-single-application"></a>Utilisation d’une application unique

Dans certains scénarios, vous pouvez avoir uniquement une seule association d’un client de niveau intermédiaire et d’un client front-end. Dans ce scénario, il peut s’avérer plus facile d’en faire une seule application, annulant ainsi complètement la nécessité d’une application de niveau intermédiaire. Pour l’authentification entre le front-end et l’API web, vous pouvez utiliser des cookies, un jeton id_token ou un jeton d’accès demandé pour l’application elle-même. Demandez ensuite le consentement dans cette application unique à la ressource back-end.

## <a name="client-limitations"></a>Limitations du client

Si un client utilise le flux implicite pour obtenir un id_token, et que ce client a également des caractères génériques dans une URL de réponse, id_token ne peut pas être utilisé pour un flux OBO.  Toutefois, les jetons d’accès obtenus via le flux d’octroi implicite peuvent toujours être échangés par un client confidentiel même si le client d’origine a une URL de réponse générique inscrite.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus d’informations sur le protocole OAuth 2.0 et une autre méthode pour effectuer l’authentification de service à service à l’aide des informations d’identification du client.

* [Octroi d’informations d’identification du client OAuth 2.0 sur la plateforme d’identités Microsoft](v2-oauth2-client-creds-grant-flow.md)
* [Flux de code OAuth 2.0 sur la plateforme d’identité Microsoft](v2-oauth2-auth-code-flow.md)
* [Utilisation de l’étendue `/.default`](v2-permissions-and-consent.md#the-default-scope)
