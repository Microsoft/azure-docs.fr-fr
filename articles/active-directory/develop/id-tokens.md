---
title: Jetons d’ID de la plateforme d’identités Microsoft | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment utiliser les id_tokens émis par Azure AD 1.0 et les points de terminaison de la plateforme d’identités Microsoft (v2.0).
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev, identityplatformtop40
ms:custom: fasttrack-edit
ms.openlocfilehash: eec24ca19af8c41a842b1db3db4b9e16f748f029
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97653246"
---
# <a name="microsoft-identity-platform-id-tokens"></a>Jetons d’ID de la plateforme d’identités Microsoft

Les jetons `id_tokens` sont envoyés à l’application cliente dans le cadre d’un flux [OpenID Connect](v2-protocols-oidc.md) (OIDC). Ils peuvent être envoyés avec ou à la place d’un jeton d’accès et sont utilisés par le client pour authentifier l’utilisateur.

## <a name="using-the-id_token"></a>Utilisation du jeton id_token

Les jetons d’ID doivent servir à confirmer qu’un utilisateur est bien celui qu’il prétend être et à obtenir des informations supplémentaires le concernant. Ils ne doivent pas être utilisés pour accorder une autorisation à la place d’un [jeton d’accès](access-tokens.md). Les revendications obtenues peuvent être utilisées pour l’expérience utilisateur au sein de votre application, en tant que [clés dans une base de données](#using-claims-to-reliably-identify-a-user-subject-and-object-id) et pour fournir l’accès à l’application cliente.  

## <a name="claims-in-an-id_token"></a>Revendications dans un jeton id_token

Les `id_tokens` sont des [JWT](https://tools.ietf.org/html/rfc7519) (JSON Web Tokens), ce qui signifie qu’ils se composent d’un en-tête, d’une charge utile et d’une signature. Vous pouvez utiliser l’en-tête et la signature pour vérifier l’authenticité du jeton, tandis que la charge utile contient les informations concernant l’utilisateur qui sont demandées par votre client. Sauf indication contraire, toutes les revendications JWT répertoriées ici apparaissent dans les jetons v1.0 et v2.0.

### <a name="v10"></a>v1.0

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyIsImtpZCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyJ9.eyJhdWQiOiJiMTRhNzUwNS05NmU5LTQ5MjctOTFlOC0wNjAxZDBmYzljYWEiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkvIiwiaWF0IjoxNTM2Mjc1MTI0LCJuYmYiOjE1MzYyNzUxMjQsImV4cCI6MTUzNjI3OTAyNCwiYWlvIjoiQVhRQWkvOElBQUFBcXhzdUIrUjREMnJGUXFPRVRPNFlkWGJMRDlrWjh4ZlhhZGVBTTBRMk5rTlQ1aXpmZzN1d2JXU1hodVNTajZVVDVoeTJENldxQXBCNWpLQTZaZ1o5ay9TVTI3dVY5Y2V0WGZMT3RwTnR0Z2s1RGNCdGsrTExzdHovSmcrZ1lSbXY5YlVVNFhscGhUYzZDODZKbWoxRkN3PT0iLCJhbXIiOlsicnNhIl0sImVtYWlsIjoiYWJlbGlAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiTGluY29sbiIsImdpdmVuX25hbWUiOiJBYmUiLCJpZHAiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaXBhZGRyIjoiMTMxLjEwNy4yMjIuMjIiLCJuYW1lIjoiYWJlbGkiLCJub25jZSI6IjEyMzUyMyIsIm9pZCI6IjA1ODMzYjZiLWFhMWQtNDJkNC05ZWMwLTFiMmJiOTE5NDQzOCIsInJoIjoiSSIsInN1YiI6IjVfSjlyU3NzOC1qdnRfSWN1NnVlUk5MOHhYYjhMRjRGc2dfS29vQzJSSlEiLCJ0aWQiOiJmYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkiLCJ1bmlxdWVfbmFtZSI6IkFiZUxpQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJMeGVfNDZHcVRrT3BHU2ZUbG40RUFBIiwidmVyIjoiMS4wIn0=.UJQrCA6qn2bXq57qzGX_-D3HcPHqBMOKDPx4su1yKRLNErVD8xkxJLNLVRdASHqEcpyDctbdHccu6DPpkq5f0ibcaQFhejQNcABidJCTz0Bb2AbdUCTqAzdt9pdgQvMBnVH1xk3SCM6d4BbT4BkLLj10ZLasX7vRknaSjE_C5DI7Fg4WrZPwOhII1dB0HEZ_qpNaYXEiy-o94UJ94zCr07GgrqMsfYQqFR7kn-mn68AjvLcgwSfZvyR_yIK75S_K37vC3QryQ7cNoafDe9upql_6pB2ybMVlgWPs_DmbJ8g0om-sPlwyn74Cc1tW3ze-Xptw_2uVdPgWyqfuWAfq6Q
```

Afficher cet exemple de jeton v1.0 dans [jwt.ms](https://jwt.ms/#id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyIsImtpZCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyJ9.eyJhdWQiOiJiMTRhNzUwNS05NmU5LTQ5MjctOTFlOC0wNjAxZDBmYzljYWEiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkvIiwiaWF0IjoxNTM2Mjc1MTI0LCJuYmYiOjE1MzYyNzUxMjQsImV4cCI6MTUzNjI3OTAyNCwiYWlvIjoiQVhRQWkvOElBQUFBcXhzdUIrUjREMnJGUXFPRVRPNFlkWGJMRDlrWjh4ZlhhZGVBTTBRMk5rTlQ1aXpmZzN1d2JXU1hodVNTajZVVDVoeTJENldxQXBCNWpLQTZaZ1o5ay9TVTI3dVY5Y2V0WGZMT3RwTnR0Z2s1RGNCdGsrTExzdHovSmcrZ1lSbXY5YlVVNFhscGhUYzZDODZKbWoxRkN3PT0iLCJhbXIiOlsicnNhIl0sImVtYWlsIjoiYWJlbGlAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiTGluY29sbiIsImdpdmVuX25hbWUiOiJBYmUiLCJpZHAiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaXBhZGRyIjoiMTMxLjEwNy4yMjIuMjIiLCJuYW1lIjoiYWJlbGkiLCJub25jZSI6IjEyMzUyMyIsIm9pZCI6IjA1ODMzYjZiLWFhMWQtNDJkNC05ZWMwLTFiMmJiOTE5NDQzOCIsInJoIjoiSSIsInN1YiI6IjVfSjlyU3NzOC1qdnRfSWN1NnVlUk5MOHhYYjhMRjRGc2dfS29vQzJSSlEiLCJ0aWQiOiJmYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkiLCJ1bmlxdWVfbmFtZSI6IkFiZUxpQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJMeGVfNDZHcVRrT3BHU2ZUbG40RUFBIiwidmVyIjoiMS4wIn0=.UJQrCA6qn2bXq57qzGX_-D3HcPHqBMOKDPx4su1yKRLNErVD8xkxJLNLVRdASHqEcpyDctbdHccu6DPpkq5f0ibcaQFhejQNcABidJCTz0Bb2AbdUCTqAzdt9pdgQvMBnVH1xk3SCM6d4BbT4BkLLj10ZLasX7vRknaSjE_C5DI7Fg4WrZPwOhII1dB0HEZ_qpNaYXEiy-o94UJ94zCr07GgrqMsfYQqFR7kn-mn68AjvLcgwSfZvyR_yIK75S_K37vC3QryQ7cNoafDe9upql_6pB2ybMVlgWPs_DmbJ8g0om-sPlwyn74Cc1tW3ze-Xptw_2uVdPgWyqfuWAfq6Q).

### <a name="v20"></a>v2.0

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjFMVE16YWtpaGlSbGFfOHoyQkVKVlhlV01xbyJ9.eyJ2ZXIiOiIyLjAiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vOTEyMjA0MGQtNmM2Ny00YzViLWIxMTItMzZhMzA0YjY2ZGFkL3YyLjAiLCJzdWIiOiJBQUFBQUFBQUFBQUFBQUFBQUFBQUFJa3pxRlZyU2FTYUZIeTc4MmJidGFRIiwiYXVkIjoiNmNiMDQwMTgtYTNmNS00NmE3LWI5OTUtOTQwYzc4ZjVhZWYzIiwiZXhwIjoxNTM2MzYxNDExLCJpYXQiOjE1MzYyNzQ3MTEsIm5iZiI6MTUzNjI3NDcxMSwibmFtZSI6IkFiZSBMaW5jb2xuIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiQWJlTGlAbWljcm9zb2Z0LmNvbSIsIm9pZCI6IjAwMDAwMDAwLTAwMDAtMDAwMC02NmYzLTMzMzJlY2E3ZWE4MSIsInRpZCI6IjkxMjIwNDBkLTZjNjctNGM1Yi1iMTEyLTM2YTMwNGI2NmRhZCIsIm5vbmNlIjoiMTIzNTIzIiwiYWlvIjoiRGYyVVZYTDFpeCFsTUNXTVNPSkJjRmF0emNHZnZGR2hqS3Y4cTVnMHg3MzJkUjVNQjVCaXN2R1FPN1lXQnlqZDhpUURMcSFlR2JJRGFreXA1bW5PcmNkcUhlWVNubHRlcFFtUnA2QUlaOGpZIn0.1AFWW-Ck5nROwSlltm7GzZvDwUkqvhSQpm55TQsmVo9Y59cLhRXpvB8n-55HCr9Z6G_31_UbeUkoz612I2j_Sm9FFShSDDjoaLQr54CreGIJvjtmS3EkK9a7SJBbcpL1MpUtlfygow39tFjY7EVNW9plWUvRrTgVk7lYLprvfzw-CIqw3gHC-T7IK_m_xkr08INERBtaecwhTeN4chPC4W3jdmw_lIxzC48YoQ0dB1L9-ImX98Egypfrlbm0IBL5spFzL6JDZIRRJOu8vecJvj1mq-IUhGt0MacxX8jdxYLP-KUu2d9MbNKpCKJuZ7p8gwTL5B7NlUdh_dmSviPWrw
```

Afficher cet exemple de jeton v2.0 dans [jwt.ms](https://jwt.ms/#id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjFMVE16YWtpaGlSbGFfOHoyQkVKVlhlV01xbyJ9.eyJ2ZXIiOiIyLjAiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vOTEyMjA0MGQtNmM2Ny00YzViLWIxMTItMzZhMzA0YjY2ZGFkL3YyLjAiLCJzdWIiOiJBQUFBQUFBQUFBQUFBQUFBQUFBQUFJa3pxRlZyU2FTYUZIeTc4MmJidGFRIiwiYXVkIjoiNmNiMDQwMTgtYTNmNS00NmE3LWI5OTUtOTQwYzc4ZjVhZWYzIiwiZXhwIjoxNTM2MzYxNDExLCJpYXQiOjE1MzYyNzQ3MTEsIm5iZiI6MTUzNjI3NDcxMSwibmFtZSI6IkFiZSBMaW5jb2xuIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiQWJlTGlAbWljcm9zb2Z0LmNvbSIsIm9pZCI6IjAwMDAwMDAwLTAwMDAtMDAwMC02NmYzLTMzMzJlY2E3ZWE4MSIsInRpZCI6IjkxMjIwNDBkLTZjNjctNGM1Yi1iMTEyLTM2YTMwNGI2NmRhZCIsIm5vbmNlIjoiMTIzNTIzIiwiYWlvIjoiRGYyVVZYTDFpeCFsTUNXTVNPSkJjRmF0emNHZnZGR2hqS3Y4cTVnMHg3MzJkUjVNQjVCaXN2R1FPN1lXQnlqZDhpUURMcSFlR2JJRGFreXA1bW5PcmNkcUhlWVNubHRlcFFtUnA2QUlaOGpZIn0.1AFWW-Ck5nROwSlltm7GzZvDwUkqvhSQpm55TQsmVo9Y59cLhRXpvB8n-55HCr9Z6G_31_UbeUkoz612I2j_Sm9FFShSDDjoaLQr54CreGIJvjtmS3EkK9a7SJBbcpL1MpUtlfygow39tFjY7EVNW9plWUvRrTgVk7lYLprvfzw-CIqw3gHC-T7IK_m_xkr08INERBtaecwhTeN4chPC4W3jdmw_lIxzC48YoQ0dB1L9-ImX98Egypfrlbm0IBL5spFzL6JDZIRRJOu8vecJvj1mq-IUhGt0MacxX8jdxYLP-KUu2d9MbNKpCKJuZ7p8gwTL5B7NlUdh_dmSviPWrw).

### <a name="header-claims"></a>Revendications de l’en-tête

|Revendication | Format | Description |
|-----|--------|-------------|
|`typ` | Chaîne : toujours « JWT » | Indique que le jeton est un jeton JWT.|
|`alg` | String | Indique l’algorithme utilisé pour signer le jeton. Exemple : "RS256" |
|`kid` | String | Empreinte de la clé publique utilisée pour signer ce jeton. Émise dans les jetons `id_tokens` v1.0 et v2.0. |
|`x5t` | String | Identique (en utilisation et en valeur) à `kid`. Il s'agit toutefois d'une ancienne revendication émise uniquement dans les jetons `id_tokens` v1.0 à des fins de compatibilité. |

### <a name="payload-claims"></a>Revendications de la charge utile

Cette liste affiche les revendications JWT présentes par défaut dans la plupart des jetons id_token (sauf indication contraire).  Toutefois, votre application peut utiliser les [revendications facultatives](active-directory-optional-claims.md) pour demander des revendications JWT supplémentaires dans le jeton id_token.  Cela peut concerner la revendication `groups` comme des informations sur le nom de l’utilisateur.

|Revendication | Format | Description |
|-----|--------|-------------|
|`aud` |  Chaîne, URI ID d’application | Identifie le destinataire du jeton. Dans les jetons `id_tokens`, l'audience est l'ID attribué à votre application dans le portail Azure. Votre application doit valider cette valeur et rejeter le jeton si la valeur ne correspond pas. |
|`iss` |  Chaîne, URI STS | Identifie le service d’émission de jeton de sécurité (STS) qui construit et retourne le jeton, ainsi que le client Azure AD dans lequel l’utilisateur a été authentifié. Si le jeton a été émis par le terminal v2.0, l’URI se termine par `/v2.0`.  La partie GUID qui indique que l’utilisateur est un utilisateur consommateur d’un compte Microsoft est `9188040d-6c67-4c5b-b112-36a304b66dad`. Votre application doit utiliser la partie GUID de la revendication pour restreindre l’ensemble des clients qui peuvent se connecter à l’application, le cas échéant. |
|`iat` |  int, horodatage UNIX | « Issued At » (Délivré le) indique quand l'authentification de ce jeton a eu lieu.  |
|`idp`|Chaîne, généralement un URI STS | Enregistre le fournisseur d’identité qui a authentifié le sujet du jeton. Cette valeur est identique à la valeur de la revendication de l’émetteur sauf si le compte d’utilisateur n’est pas dans le même locataire que l’émetteur (invités par exemple). Si la revendication n’est pas présente, cela signifie que la valeur `iss` peut être utilisée à la place.  Pour les comptes personnels utilisés dans un contexte organisationnel (par exemple, un compte personnel invité dans un locataire Azure AD), la revendication `idp` peut être « live.com » ou un URI STS contenant le locataire de compte Microsoft `9188040d-6c67-4c5b-b112-36a304b66dad`. |
|`nbf` |  int, horodatage UNIX | La revendication « nbf » (pas avant) indique le délai avant lequel le JWT ne doit PAS être accepté pour être traité.|
|`exp` |  int, horodatage UNIX | La revendication « exp » (délai d'expiration) indique le délai d'expiration à partir duquel le JWT ne doit PAS être accepté pour être traité.  Il est important de remarquer qu’une ressource peut également rejeter le jeton avant ce délai, par exemple lorsqu’un changement d’authentification est requis ou qu’une révocation de jeton a été détectée. |
| `c_hash`| String |Le hachage de code n’est inclus dans un jeton d’ID que si ce dernier est émis avec un code d’autorisation OAuth 2.0. Il peut servir à valider l’authenticité d’un code d’autorisation. Pour plus d’informations sur l’exécution de cette validation, consultez la [spécification OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) . |
|`at_hash`| String |Le hachage de jeton d’accès n’est inclus dans un jeton d’ID que si ce dernier est émis du point de terminaison `/authorize` avec un jeton d’accès OAuth 2.0. Il peut servir à valider l’authenticité d’un jeton d’accès. Pour plus d’informations sur l’exécution de cette validation, consultez la [spécification OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#HybridIDToken) . Cela n’est pas retourné sur les jetons d’ID du point de terminaison `/token`. |
|`aio` | Chaîne opaque | Revendication interne utilisée par Azure AD pour enregistrer des données afin de réutiliser les jetons. Cette valeur doit être ignorée.|
|`preferred_username` | String | Nom d’utilisateur principal qui représente l’utilisateur. Il peut s’agir d’une adresse e-mail, d’un numéro de téléphone ou d’un nom d’utilisateur générique sans format spécifié. Sa valeur est mutable et peut changer au fil du temps. Dans la mesure où elle est mutable, cette valeur ne doit pas utilisée pour prendre des décisions d’autorisation. L’étendue `profile` est requise afin de recevoir cette revendication.|
|`email` | String | La revendication `email` est présente par défaut pour les comptes invités qui disposent d’une adresse de messagerie.  Votre application peut demander la revendication de courrier électronique pour les utilisateurs gérés (provenant du même locataire que la ressource) à l’aide de la `email` [revendication facultative](active-directory-optional-claims.md).  Sur le point de terminaison v2.0, votre application peut également demander l’étendue OpenID Connect `email` : vous n’avez pas besoin de demander la revendication facultative et l’étendue pour obtenir la revendication.  La revendication de courrier électronique prend uniquement en charge la messagerie adressable provenant des informations du profil de l’utilisateur. |
|`name` | String | La revendication `name` fournit une valeur explicite qui identifie le sujet du jeton. Il n’est pas certain que cette valeur soit unique. Elle est mutable et conçue pour être utilisée uniquement à des fins d’affichage. L’étendue `profile` est requise afin de recevoir cette revendication. |
|`nonce`| String | La valeur à usage unique correspond au paramètre inclus dans la requête /authorize d’origine au point de distribution d’émission. Si ce n’est pas le cas, votre application doit rejeter le jeton. |
|`oid` | Chaîne, GUID | Identificateur immuable pour un objet dans le système d’identité Microsoft, dans cet exemple, un compte d’utilisateur. Cet ID identifie de manière unique l’utilisateur entre les applications ; deux applications différentes se connectant au même utilisateur auront la même valeur dans la revendication `oid`. Microsoft Graph renverra cet ID en tant que propriété `id` pour un compte d’utilisateur donné. `oid` permettant à plusieurs applications de faire correspondre des utilisateurs, l’étendue `profile` est requise afin de recevoir cette revendication. Notez que si un utilisateur existe dans plusieurs locataires, l’utilisateur contient un ID d’objet différent dans chaque locataire. Ils sont considérés comme des comptes différents, même si l’utilisateur se connecte à chaque compte avec les mêmes informations d’identification. La revendication `oid` est un GUID qui ne peut pas être réutilisé. |
|`roles`| Tableau de chaînes | Ensemble des rôles attribués à l’utilisateur qui se connecte. |
|`rh` | Chaîne opaque |Revendication interne utilisée par Azure pour revalider des jetons. Cette valeur doit être ignorée. |
|`sub` | String | Principal sur lequel portent les assertions d’informations du jeton, comme l’utilisateur d’une application. Cette valeur est immuable et ne peut pas être réattribuée ou réutilisée. L’objet est un identificateur par paire ; il est spécifique à un ID d’application donné. Si un utilisateur se connecte à deux applications différentes à l’aide de deux ID clients différents, ces applications reçoivent deux valeurs différentes pour la revendication de l’objet. Ceci peut être souhaitable ou non en fonction de vos exigences en matière d’architecture et de confidentialité. |
|`tid` | Chaîne, GUID | GUID représentant le client Azure AD d’où provient l’utilisateur. Pour les comptes professionnels et scolaires, le GUID correspond à l’ID de client immuable de l’organisation à laquelle appartient l’utilisateur. Pour les comptes personnels, la valeur est `9188040d-6c67-4c5b-b112-36a304b66dad`. L’étendue `profile` est requise afin de recevoir cette revendication. |
|`unique_name` | String | Fournit une valeur contrôlable de visu qui identifie le sujet du jeton. Cette valeur est unique à un moment donné mais, comme les e-mails et autres identificateurs peuvent être réutilisés, elle peut réapparaître sur d’autres comptes et doit donc être utilisée uniquement à des fins d’affichage. Émise uniquement dans les jetons `id_tokens` v1.0. |
|`uti` | Chaîne opaque | Revendication interne utilisée par Azure pour revalider des jetons. Cette valeur doit être ignorée. |
|`ver` | Chaîne, 1.0 ou 2.0 | Indique la version du jeton id_token. |
|`hasgroups`|Boolean|Le cas échéant, toujours true, ce qui indique que l’utilisateur appartient à au moins un groupe. Utilisé à la place de la revendication des groupes pour les JWT dans les flux d’octroi implicites si la revendication des groupes complets étend le fragment URI au-delà des limites de longueur d’URL (actuellement, 6 groupes ou plus). Indique que le client doit utiliser l’API Microsoft Graph pour déterminer les groupes de l’utilisateur (`https://graph.microsoft.com/v1.0/users/{userID}/getMemberObjects`).|
|`groups:src1`|Objet JSON | Pour les requêtes de jetons dont la longueur n’est pas limitée (voir `hasgroups` ci-dessus) mais qui sont toujours trop volumineuses pour le jeton, un lien vers la liste des groupes complets pour l’utilisateur sera inclus. Pour les jetons JWT en tant que revendication distribuée, pour SAML en tant que nouvelle revendication à la place de la revendication `groups`. <br><br>**Exemple de valeur JWT** : <br> `"groups":"src1"` <br> `"_claim_sources`: `"src1" : { "endpoint" : "https://graph.microsoft.com/v1.0/users/{userID}/getMemberObjects" }`<br><br> Pour plus d’informations, consultez [Revendication de dépassement des groupes](#groups-overage-claim).|

> [!NOTE]
> Les id_token v1.0 et v2.0 présentent des différences sur le plan de la quantité d’informations qu’ils contiennent, comme l’illustrent les exemples ci-dessus. La version est basée sur le point de terminaison à partir duquel le jeton a été demandé. Si les applications existantes utilisent probablement le point de terminaison Azure AD, les nouvelles applications doivent utiliser le point de terminaison de « Plateforme d’identité Microsoft » v2.0.
>
> - v1.0 : Points de terminaison Azure AD : `https://login.microsoftonline.com/common/oauth2/authorize`
> - V2.0 : Points de terminaison de Plateforme d’identité Microsoft : `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`

### <a name="using-claims-to-reliably-identify-a-user-subject-and-object-id"></a>Utilisation de revendications pour identifier de manière fiable un utilisateur (Objet et ID d’objet)

Lorsque vous identifiez un utilisateur (par exemple, en le recherchant dans une base de données ou en déterminant les autorisations qui lui sont attribuées), il est essentiel d’utiliser des informations qui resteront constantes et uniques au fil du temps. Les applications héritées utilisent parfois des champs tels que l’adresse e-mail, un numéro de téléphone ou l’UPN.  Toutes ces éléments peuvent changer au fil du temps et peuvent également être réutilisés au fil du temps, par exemple, quand un employé modifie son nom ou reçoit une adresse e-mail correspondant à celle d’un employé précédent qui n’est plus présent. Par conséquent, il est **critique** que votre application n’utilise pas de données lisibles à l’œil pour identifier un utilisateur. Quelqu’un pourrait les lire et vouloir les modifier. Utilisez plutôt les revendications fournies par la norme OIDC, ou les revendications d’extension fournies par Microsoft (revendications `sub` et `oid`).

Pour stocker correctement les informations par utilisateur, utilisez `sub` ou `oid` seul (qui, comme des GUID sont uniques), avec `tid` utilisé pour le routage ou partitionnement si nécessaire.  Si vous avez besoin de partager des données entre les services, `oid`+`tid` est préférable, car toutes les applications reçoivent les mêmes revendications `oid` et `tid` pour un utilisateur donné.  La revendication `sub` dans la plateforme d’identité Microsoft est « par paire ». Elle est unique en fonction d’une combinaison de destinataire, de locataire et d’utilisateur du jeton.  Ainsi, deux applications qui demandent des jetons d’ID pour un utilisateur donné reçoivent des revendications `sub` différentes, mais les mêmes revendications `oid` pour cet utilisateur.

>[!NOTE]
> N’utilisez pas la revendication `idp` pour stocker des informations sur un utilisateur dans une tentative de mettre en corrélation des utilisateurs parmi les locataires.  Elle ne fonctionnera pas, car, par conception, les revendications `oid` et `sub` pour un utilisateur changent entre locataires pour s’assurer que des applications ne puissent pas suivre des utilisateurs parmi les locataires.  
>
> Les scénarios d’invité, où un utilisateur est hébergé dans un locataire et s’authentifie dans un autre, doivent traiter l’utilisateur comme s’il s’agissait d’un tout nouvel utilisateur du service.  Vos documents et privilèges dans le locataire Contoso ne doivent pas s’appliquer dans le locataire Fabrikam. Cela est important pour empêcher des fuites accidentelles de données entre locataires.

### <a name="groups-overage-claim"></a>Revendication de dépassement des groupes
Pour s’assurer que la taille du jeton ne dépasse pas les limites de taille d’en-tête HTTP, Azure AD limite le nombre d’ID d’objets inclus dans la revendication `groups`. Si un utilisateur est membre d’un nombre de groupes supérieur à la limite de dépassement (150 pour les jetons SAML, 200 pour les jetons JWT), Azure AD n’émet pas la revendication des groupes dans le jeton. Au lieu de cela, il inclut une revendication de dépassement dans le jeton qui indique à l’application d’interroger l’API Microsoft Graph pour récupérer l’appartenance de groupe de l’utilisateur.

```json
{
  ...
  "_claim_names": {
   "groups": "src1"
    },
    {
  "_claim_sources": {
    "src1": {
        "endpoint":"[Url to get this user's group membership from]"
        }
       }
     }
  ...
}
```

## <a name="validating-an-id_token"></a>Valider un jeton id_token

Le processus de validation d’un `id_token` est similaire à la première étape de [validation d’un jeton d’accès](access-tokens.md#validating-tokens). Votre client doit confirmer que le bon émetteur a renvoyé le jeton et que celui-ci n’a pas été falsifié. Étant donné que les jetons `id_tokens` sont toujours des jetons JWT, de nombreuses bibliothèques servent à valider ces jetons. Nous vous recommandons d’en utiliser une plutôt que de le faire vous-même.  Notez que seuls des clients confidentiels (avec un secret) doivent valider des jetons d’ID.  Les applications publiques (dont le code s’exécute entièrement sur un appareil ou un réseau que vous ne contrôlez pas ; par exemple, le navigateur ou le réseau domestiques d’un utilisateur) ne bénéficient pas de la validation du jeton d’ID, car un utilisateur malveillant peut intercepter et modifier les clés utilisées pour la validation du jeton.

Pour valider manuellement le jeton, consultez les étapes détaillées dans la section [Validation d’un jeton d’accès](access-tokens.md#validating-tokens). Après avoir validé la signature du jeton, les revendications JWT suivantes doivent être validées dans le jeton id_token (cette opération peut également être effectuée par votre bibliothèque de validation de jetons) :

* Horodatages : les horodatages `iat`, `nbf` et `exp` doivent tous correspondre à un moment avant ou après l’heure actuelle, selon ce qui convient.
* Audience : la revendication `aud` doit correspondre à l’ID de votre application.
* Valeur à usage unique : la revendication `nonce` dans la charge utile doit correspondre au paramètre de valeur à usage unique transmis au point de terminaison /authorize lors de la demande initiale.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [jetons d’accès](access-tokens.md)
* Personnalisez les revendications JWT du jeton id_token à l’aide des [revendications facultatives](active-directory-optional-claims.md).
