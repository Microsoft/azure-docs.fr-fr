---
title: Demander un jeton d’accès Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment obtenir un jeton d’accès avec Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/26/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 937041bbb48f112e2c8ed7d222dc7c7ef7ea8d81
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92631391"
---
# <a name="request-an-access-token-in-azure-active-directory-b2c"></a>Demander un jeton d’accès dans Azure Active Directory B2C

Un *jeton d’accès* contient des revendications que vous pouvez utiliser dans Azure Active Directory B2C (Azure AD B2C) pour identifier les autorisations accordées à vos API. Lorsque vous appelez un serveur de ressources, un jeton d’accès doit être présent dans la requête HTTP. Un jeton d’accès est désigné **access_token** dans les réponses d’Azure AD B2C.

Cet article présente comment demander un jeton d’accès pour une application web et une API web. Pour plus d’informations sur les jetons dans Azure AD B2C, consultez [Vue d’ensemble des jetons dans Azure Active Directory B2C](tokens-overview.md).

> [!NOTE]
> **Les chaînes d’API web (pour le compte de) ne sont pas prises en charge par Azure AD B2C.** De nombreuses architectures incluent une API web qui doit appeler une autre API web en aval, toutes deux sécurisées par Azure AD B2C. Il s’agit d’un scénario courant pour les clients qui ont un back end d’API web, qui appelle à son tour un autre service. Ce scénario d’API web chaînée peut être pris en charge à l’aide de l’octroi des informations d’identification du porteur OAuth 2.0 JWT, également appelé flux On-Behalf-Of. Toutefois, le flux On-Behalf-Of n’est pas implémenté dans Azure AD B2C pour l’instant.

## <a name="prerequisites"></a>Prérequis

- [Créez un flux d'utilisateurs](tutorial-create-user-flows.md) pour permettre aux utilisateurs de s'inscrire et de se connecter à votre application.
- Si ce n’est pas déjà fait, [ajoutez une application d’API web à votre locataire Azure Active Directory B2C](add-web-api-application.md).

## <a name="scopes"></a>Étendues

Les étendues permettent de gérer les autorisations d’accès aux ressources protégées. Lorsqu’un jeton d’accès est demandé, l’application cliente doit spécifier les autorisations souhaitées dans le paramètre **d’étendue** de la requête. Par exemple, pour spécifier la **valeur d’étendue** de `read` pour l’API qui a l’ **URI ID d’application** de `https://contoso.onmicrosoft.com/api`, l’étendue serait `https://contoso.onmicrosoft.com/api/read`.

Elles sont utilisées par l’API web pour implémenter le contrôle d’accès basé sur les étendues. Par exemple, les utilisateurs de l’API web peuvent avoir un accès en lecture et en écriture, ou les utilisateurs de l’API web peuvent avoir l’accès en lecture uniquement. Pour acquérir plusieurs autorisations dans la même requête, vous pouvez ajouter plusieurs entrées séparées par des espaces dans le même paramètre d’ **étendue** de la requête.

L’exemple suivant présente des étendues décodées dans une URL :

```
scope=https://contoso.onmicrosoft.com/api/read openid offline_access
```

L’exemple suivant présente des étendues encodées dans une URL :

```
scope=https%3A%2F%2Fcontoso.onmicrosoft.com%2Fapi%2Fread%20openid%20offline_access
```

Si vous demandez plus d’étendues que ce qui est autorisé pour votre application cliente, l’appel réussit si au moins une autorisation est accordée. La revendication **scp** du jeton d’accès obtenue est remplie uniquement avec les autorisations qui ont été accordées. 

### <a name="openid-connect-scopes"></a>Étendues OpenId Connect

Le standard OpenID Connect spécifie plusieurs valeurs spéciales d’étendue. Les étendues suivantes représentent l'autorisation d'accès au profil de l'utilisateur :

- **openid** : cette étendue demande un jeton d’ID.
- **offline_access**  : cette étendue demande un jeton d’actualisation à l’aide du [flux de code d’authentification](authorization-code-flow.md).
- **00000000-0000-0000-0000-000000000000** : l’utilisation de l’ID de client comme étendue indique que votre application a besoin d’un jeton d’accès qui peut être utilisé avec votre propre service ou API web, représenté par le même ID de client.

Si le paramètre **response_type** dans une requête `/authorize` inclut `token`, le paramètre **scope** doit inclure au moins l’une des étendues de ressource (autre que `openid` et `offline_access`) qui sera accordée. Sinon, la demande `/authorize` échoue.

## <a name="request-a-token"></a>Demander un jeton

Pour demander un jeton, il vous faut un code d’autorisation. Voici un exemple de demande adressée au point de terminaison `/authorize` pour obtenir un code d’autorisation. Les domaines personnalisés ne sont pas pris en charge avec les jetons d’accès. Utilisez votre domaine tenant-name.onmicrosoft.com dans l’URL de la requête.

Dans l'exemple suivant, vous remplacez ces valeurs :

- `<tenant-name>` - Nom de votre locataire Azure AD B2C.
- `<policy-name>` - Nom de votre stratégie personnalisée ou de votre flux d'utilisateurs.
- `<application-ID>` - Identificateur de l’application web que vous avez inscrite pour prendre en charge le flux d’utilisateur.
- `<redirect-uri>` - **URI de redirection** que vous avez entré lorsque vous avez inscrit l'application cliente.

```http
GET https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/authorize?
client_id=<application-ID>
&nonce=anyRandomValue
&redirect_uri=https://jwt.ms
&scope=https://<tenant-name>.onmicrosoft.com/api/read
&response_type=code
```

La réponse avec le code d’autorisation ressemble à ce qui suit :

```
https://jwt.ms/?code=eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMC...
```

Après avoir reçu le code d’autorisation, vous pouvez l’utiliser pour demander un jeton d’accès :

```http
POST <tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/token HTTP/1.1
Host: <tenant-name>.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=<application-ID>
&scope=https://<tenant-name>.onmicrosoft.com/api/read
&code=eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMC...
&redirect_uri=https://jwt.ms
&client_secret=2hMG2-_:y12n10vwH...
```

Vous devriez voir quelque chose de similaire à la réponse suivante :

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrN...",
    "token_type": "Bearer",
    "not_before": 1549647431,
    "expires_in": 3600,
    "expires_on": 1549651031,
    "resource": "f2a76e08-93f2-4350-833c-965c02483b11",
    "profile_info": "eyJ2ZXIiOiIxLjAiLCJ0aWQiOiJjNjRhNGY3ZC0zMDkxLTRjNzMtYTcyMi1hM2YwNjk0Z..."
}
```

Lorsque vous utilisez https://jwt.ms pour examiner le jeton d’accès retourné, vous devriez voir quelque chose de similaire à l’exemple suivant :

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dl..."
}.{
  "iss": "https://contoso0926tenant.b2clogin.com/c64a4f7d-3091-4c73-a7.../v2.0/",
  "exp": 1549651031,
  "nbf": 1549647431,
  "aud": "f2a76e08-93f2-4350-833c-965...",
  "oid": "1558f87f-452b-4757-bcd1-883...",
  "sub": "1558f87f-452b-4757-bcd1-883...",
  "name": "David",
  "tfp": "B2C_1_signupsignin1",
  "nonce": "anyRandomValue",
  "scp": "read",
  "azp": "38307aee-303c-4fff-8087-d8d2...",
  "ver": "1.0",
  "iat": 1549647431
}.[Signature]
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [configuration des jetons dans Azure AD B2C](configure-tokens.md)
