---
title: Authentification de service à service Azure AD à l’aide d’OAuth2.0 | Microsoft Docs
description: Cet article explique comment utiliser des messages HTTP pour mettre en œuvre l’authentification de service à service à l’aide du flux d’octroi des informations d’identification du client OAuth2.0.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 02/08/2017
ms.author: ryanwi
ms.reviewer: nacanuma
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 977dfea28c5c0dc3f34ada0c138556d70c979e04
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85551708"
---
# <a name="service-to-service-calls-using-client-credentials-shared-secret-or-certificate"></a>Appels de service à service à l’aide des informations d’identification du client (secret partagé ou certificat)

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Le flux d’octroi des informations d’identification du client OAuth2.0 permet à un service web (*client confidentiel*) d’utiliser ses propres informations d’identification pour s’authentifier lorsqu’il appelle un autre service web, au lieu d’emprunter l’identité d’un utilisateur. Dans ce scénario, le client est généralement un service web de niveau intermédiaire, un service démon ou un site web. Pour augmenter le niveau d’assurance, Azure AD autorise également le service d’appel à utiliser un certificat (au lieu d’un secret partagé) comme une information d’identification.

## <a name="client-credentials-grant-flow-diagram"></a>Diagramme représentant le flux d’octroi des informations d’identification du client
Le diagramme suivant explique comment fonctionne le flux d’octroi des informations d’identification du client dans Azure Active Directory (Azure AD).

![Flux d’octroi des informations d’identification du client OAuth2.0](./media/v1-oauth2-client-creds-grant-flow/active-directory-protocols-oauth-client-credentials-grant-flow.jpg)

1. L’application cliente s’authentifie auprès du point de terminaison d’émission de jeton Azure AD et demande un jeton d’accès.
2. Le point de terminaison d’émission de jeton Azure AD émet le jeton d’accès.
3. Le jeton d’accès est utilisé pour l’authentification auprès de la ressource sécurisée.
4. Les données de la ressource sécurisée sont retournées à l’application cliente.

## <a name="register-the-services-in-azure-ad"></a>Inscription des services dans Azure AD
Inscrivez à la fois le service appelant et le service de destination dans Azure Active Directory (Azure AD). Pour obtenir des instructions détaillées, consultez [Intégration d’applications dans Azure Active Directory](../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

## <a name="request-an-access-token"></a>Demander un jeton d’accès
Pour demander un jeton d’accès, envoyez une requête HTTP POST au point de terminaison Azure AD propre au client.

```
https://login.microsoftonline.com/<tenant id>/oauth2/token
```

## <a name="service-to-service-access-token-request"></a>Demande de jeton d’accès de service à service
Deux cas de figure se présentent, selon que l’application cliente choisit d’être sécurisée par un secret partagé ou un certificat.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Premier cas : Requête de jeton d’accès avec un secret partagé
Lorsque l’application utilise un secret partagé, la demande de jeton d’accès de service à service contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| grant_type |Obligatoire |Spécifie le type d’autorisation demandée. Dans un flux d’octroi des informations d’identification du client, la valeur doit être **client_credentials**. |
| client_id |Obligatoire |Spécifie l’ID de client Azure AD du service web appelant. Pour rechercher l’ID de client de l’application appelante, dans le [portail Azure](https://portal.azure.com), cliquez successivement sur **Azure Active Directory**, **Inscriptions des applications** et sur l’application. Le paramètre client_id est l’*ID de l’application* |
| client_secret |Obligatoire |Entrez une clé enregistrée pour le service web appelant ou l’application démon dans Azure AD. Pour créer une clé, dans le portail Azure, cliquez successivement sur **Azure Active Directory**, **Inscriptions des applications**, l’application, **Paramètres**, **Clés** et ajoutez une clé.  Encodez en URL ce secret lorsque vous le fournissez. |
| resource |Obligatoire |Entrez l’URI ID d’application du service web de destination. Pour rechercher l’URI de l’ID d’application, dans le portail Azure, cliquez successivement sur **Azure Active Directory**, **Inscriptions des applications**, l’application du service, puis sur **Paramètres** et **Propriétés**. |

#### <a name="example"></a>Exemple
La requête HTTP POST suivante demande un [jeton d’accès](../develop/access-tokens.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) pour le service web `https://service.contoso.com/`. `client_id` identifie le service web qui demande le jeton d’accès.

```
POST /contoso.com/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=625bc9f6-3bf6-4b6d-94ba-e97cf07a22de&client_secret=qkDwDJlDfig2IpeuUZYKH1Wb8q1V0ju6sILxQQqhJ+s=&resource=https%3A%2F%2Fservice.contoso.com%2F
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Deuxième cas : Requête de jeton d’accès avec un certificat
Une demande de jeton d’accès de service à service avec un certificat contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| grant_type |Obligatoire |Spécifie le type de réponse demandé. Dans un flux d’octroi des informations d’identification du client, la valeur doit être **client_credentials**. |
| client_id |Obligatoire |Spécifie l’ID de client Azure AD du service web appelant. Pour rechercher l’ID de client de l’application appelante, dans le [portail Azure](https://portal.azure.com), cliquez successivement sur **Azure Active Directory**, **Inscriptions des applications** et sur l’application. Le paramètre client_id est l’*ID de l’application* |
| client_assertion_type |Obligatoire |La valeur doit être `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| client_assertion |Obligatoire | Assertion (JSON Web Token) dont vous avez besoin pour créer et signer avec le certificat inscrit comme informations d’identification pour votre application. Pour découvrir comment inscrire votre certificat et le format de l’assertion, consultez la rubrique traitant des [informations d’identification des certificats](../develop/active-directory-certificate-credentials.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).|
| resource | Obligatoire |Entrez l’URI ID d’application du service web de destination. Pour rechercher l’URI de l’ID d’application, dans le portail Azure, cliquez successivement sur **Azure Active Directory**, **Inscriptions des applications**, l’application du service, puis sur **Paramètres** et **Propriétés**. |

Notez que les paramètres sont presque les mêmes que dans le cas de la demande par secret partagé, sauf que le paramètre client_secret est remplacé par deux paramètres : client_assertion_type et client_assertion.

#### <a name="example"></a>Exemple
La requête HTTP POST suivante demande un jeton d’accès pour le service web `https://service.contoso.com/` avec un certificat. `client_id` identifie le service web qui demande le jeton d’accès.

```
POST /<tenant_id>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

resource=https%3A%2F%contoso.onmicrosoft.com%2Ffc7664b4-cdd6-43e1-9365-c2e1c4e1b3bf&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg&grant_type=client_credentials
```

### <a name="service-to-service-access-token-response"></a>Réponse de jeton d’accès de service à service

Une réponse affirmative contient une réponse JSON OAuth 2.0 avec les paramètres suivants :

| Paramètre | Description |
| --- | --- |
| access_token |Le jeton d’accès demandé. Le service web appelant peut utiliser ce jeton pour s’authentifier auprès du service web de destination. |
| token_type |Indique la valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le **jeton porteur**. Pour plus d’informations sur les jetons du porteur, consultez le [Framework d’autorisation OAuth 2.0 : Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt). |
| expires_in |La durée de validité (en secondes) du jeton d’accès. |
| expires_on |L’heure d’expiration du jeton d’accès. La date est représentée en nombre de secondes à partir du 1er janvier 1970 (1970-01-01T0:0:0Z) UTC jusqu’au moment de l’expiration. Cette valeur est utilisée pour déterminer la durée de vie des jetons en cache. |
| not_before |Heure à partir de laquelle le jeton d’accès devient utilisable. La date est exprimée en nombre de secondes entre 1970-01-01T0:0:0Z UTC et le début de la validité du jeton.|
| resource |L’URI ID d’application du service web de destination. |

#### <a name="example-of-response"></a>Exemple de réponse
L’exemple suivant illustre une réponse affirmative à une demande de jeton d’accès à un service web.

```
{
"access_token":"eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw",
"token_type":"Bearer",
"expires_in":"3599",
"expires_on":"1388452167",
"resource":"https://service.contoso.com/"
}
```
## <a name="use-the-access-token-to-access-the-secured-resource"></a>Utiliser le jeton d’accès pour accéder à la ressource sécurisée

Le service peut utiliser le jeton d’accès obtenu pour faire des requêtes authentifiées à l’API web en aval, en définissant le jeton dans `Authorization` l’en-tête.

### <a name="example"></a>Exemple

```
GET /me?api-version=2013-11-08 HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="see-also"></a>Voir aussi
* [OAuth 2.0 dans Azure AD](v1-protocols-oauth-code.md)
* [Exemple en C# de l’appel de service à service avec un secret partagé](https://github.com/Azure-Samples/active-directory-dotnet-daemon) et [Exemple en C# de l’appel de service à service avec un certificat](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential)
