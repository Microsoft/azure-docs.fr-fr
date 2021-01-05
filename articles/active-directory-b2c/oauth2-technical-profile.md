---
title: Définir un profil technique OAuth2 dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définir un profil technique OAuth2 dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 78ce6466521c7903187798d902056948c659653c
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509850"
---
# <a name="define-an-oauth2-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique OAuth2 dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge le fournisseur d’identité du protocole OAuth2. OAuth2 est le principal protocole pour l’autorisation et l’authentification déléguée. Pour plus d’informations, voir la spécification [RFC 6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749). Un profil technique OAuth2 vous permet d’opérer une fédération avec un fournisseur d’identité basée sur OAuth2, tel que Facebook. Fédérer avec un fournisseur d’identité permet aux utilisateurs de se connecter avec leurs identités existantes de réseaux sociaux ou d’entreprise.

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `OAuth2`. Par exemple, le protocole pour profil technique **Facebook-OAUTH** est `OAuth2` :

```xml
<TechnicalProfile Id="Facebook-OAUTH">
  <DisplayName>Facebook</DisplayName>
  <Protocol Name="OAuth2" />
  ...
```

## <a name="input-claims"></a>Revendications d’entrée

Les éléments **InputClaims** et **InputClaimsTransformations** ne sont pas obligatoires. Vous pouvez cependant envoyer des paramètres supplémentaires à votre fournisseur d’identité. L’exemple suivant ajoute le paramètre de chaîne de requête **domain_hint** avec la valeur `contoso.com` à la demande d’autorisation.

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="domain_hint" DefaultValue="contoso.com" />
</InputClaims>
```

## <a name="output-claims"></a>Revendications de sortie

L’élément **OutputClaims** contient une liste de revendications retournée par le fournisseur d'identité OAuth2. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans le fournisseur d'identité. Vous pouvez également inclure des revendications qui ne sont pas retournées par le fournisseur d’identité, pour autant que vous définissiez l’attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

L’exemple suivant montre les revendications retournées par le fournisseur d’identité Facebook :

- Revendication **first_name** mappée à la revendication **givenName**.
- Revendication **last_name** mappée à la revendication **surname**.
- Revendication **displayName** sans mappage de nom.
- La revendication **email** sans mappage de nom.

Le profil technique retourne également des revendications qui ne sont pas retournées par le fournisseur d’identité :

- La revendication **identityProvider** contenant le nom du fournisseur d’identité.
- La revendication **authenticationSource** avec la valeur par défaut **socialIdpAuthentication**.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="id" />
  <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
  <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="facebook.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
</OutputClaims>
```

## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| client_id | Oui | Identificateur d’application du fournisseur d’identité. |
| IdTokenAudience | Non | Audience du jeton id_token. Si la valeur est spécifiée, Azure AD B2C vérifie si le jeton figure dans une revendication retournée par le fournisseur d’identité, et est identique à celui spécifié. |
| authorization_endpoint | Oui | URL du point de terminaison d’autorisation conformément à la norme RFC 6749. |
| AccessTokenEndpoint | Oui | URL du point de terminaison de jeton conformément à la norme RFC 6749. |
| ClaimsEndpoint | Oui | URL du point de terminaison d’informations utilisateur conformément à la norme RFC 6749. |
| end_session_endpoint | Oui | URL du point de terminaison de la session de fin, conformément à la norme RFC 6749. |
| AccessTokenResponseFormat | Non | Format de l’appel de point de terminaison du jeton d’accès. Par exemple, Facebook nécessite une méthode HTTP GET, mais la réponse de jeton d’accès est au format JSON. |
| AdditionalRequestQueryParameters | Non | Paramètres de requête de demande supplémentaire. Par exemple, vous pouvez envoyer des paramètres supplémentaires à votre fournisseur d’identité. Vous pouvez inclure plusieurs paramètres en utilisant un séparateur virgule. |
| ClaimsEndpointAccessTokenName | Non | Nom du paramètre de chaîne de requête du jeton accès. Les points de terminaison de revendications de certains fournisseurs d’identité prennent en charge les requêtes HTTP GET. Dans ce cas, le jeton du porteur est envoyé à l’aide d’un paramètre de chaîne de requête au lieu de l’en-tête d’autorisation. |
| ClaimsEndpointFormatName | Non | Nom du paramètre de chaîne de requête de format. Par exemple, vous pouvez définir le nom en tant que `format` dans ce point de terminaison de revendications LinkedIn `https://api.linkedin.com/v1/people/~?format=json`. |
| ClaimsEndpointFormat | Non | Valeur du paramètre de chaîne de requête de format. Par exemple, vous pouvez définir la valeur en tant que `json` dans ce point de terminaison de revendications LinkedIn `https://api.linkedin.com/v1/people/~?format=json`. |
| ProviderName | Non | Nom du fournisseur d'identité. |
| response_mode | Non | Méthode que le fournisseur d’identité utilise pour renvoyer le résultat à Azure AD B2C. Valeurs possibles : `query`, `form_post` (par défaut) ou `fragment`. |
| scope | Non | Étendue de la requête définie conformément à la spécification de fournisseur d’identité OAuth2. Par exemple, `openid`, `profile` ou `email`. |
| HttpBinding | Non | Liaison HTTP attendue aux points de terminaison de jeton d’accès et de jeton de revendications. Valeurs possibles : `GET` ou `POST`.  |
| ResponseErrorCodeParamName | Non | Nom du paramètre contenant le message d’erreur retourné sur HTTP 200 (OK). |
| ExtraParamsInAccessTokenEndpointResponse | Non | Contient les paramètres supplémentaires qui peuvent être retournés dans la réponse d’**AccessTokenEndpoint** par certains fournisseurs d’identité. Par exemple, la réponse d’**AccessTokenEndpoint** contient un paramètre supplémentaire tel que `openid`, qui est obligatoire, en plus du jeton d’accès dans une chaîne de requête de demande **ClaimsEndpoint**. S’il y a plusieurs noms de paramètre, ils doivent être échappés et séparés par le délimiteur virgule « , ». |
| ExtraParamsInClaimsEndpointRequest | Non | Contient les paramètres supplémentaires qui peuvent être retournés dans la demande **ClaimsEndpoint** par certains fournisseurs d’identité. S’il y a plusieurs noms de paramètre, ils doivent être échappés et séparés par le délimiteur virgule « , ». |
| IncludeClaimResolvingInClaimsHandling  | Non | Pour les revendications d’entrée et de sortie, spécifie si la [résolution des revendications](claim-resolver-overview.md) est incluse dans le profil technique. Valeurs possibles : `true` ou `false` (par défaut). Si vous souhaitez utiliser un programme de résolution des revendications dans le profil technique, définissez cette valeur sur `true`. |
| ResolveJsonPathsInJsonTokens  | Non | Indique si le profil technique résout les chemins d’accès JSON. Valeurs possibles : `true` ou `false` (par défaut). Utilisez ces métadonnées pour lire des données issues d’un élément JSON imbriqué. Dans un élément [OutputClaim](technicalprofiles.md#output-claims), définissez `PartnerClaimType` sur l’élément de chemin d’accès JSON que vous souhaitez générer. Par exemple : `firstName.localized` ou `data.0.to.0.email`.|
|token_endpoint_auth_method| Non| Indique comment Azure AD B2C envoie l’en-tête d’authentification au point de terminaison du jeton. Valeurs possibles : `client_secret_post` (par défaut) et `client_secret_basic` (préversion publique). Pour plus d’informations, consultez [Section d’authentification du client OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication). |
|SingleLogoutEnabled| Non| Indique si, lors de la connexion, le profil technique tente de se déconnecter des fournisseurs d’identité fédérés. Pour plus d’informations, consultez [Déconnexion d’une session Azure AD B2C](session-behavior.md#sign-out).  Valeurs possibles : `true` (par défaut) ou `false`.|

## <a name="cryptographic-keys"></a>Clés de chiffrement

L’élément **CryptographicKeys** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| client_secret | Oui | Clé secrète client de l’application du fournisseur d’identité. La clé de chiffrement est requise uniquement si les métadonnées **response_types** sont définies sur `code`. Dans ce cas, Azure AD B2C émet un autre appel pour échanger le code d’autorisation pour un jeton d’accès. Si les métadonnées sont définies sur `id_token`, vous pouvez omettre la clé de chiffrement. |

## <a name="redirect-uri"></a>URI de redirection

Lorsque vous configurez l’URI de redirection de votre fournisseur d’identité, entrez `https://{tenant-name}.b2clogin.com/{tenant-name}.onmicrosoft.com/oauth2/authresp`. Veillez à remplacer `{tenant-name}` par le nom de votre locataire (par exemple, contosob2c). L’URI de redirection doit être en minuscules.

Exemples :

- [Ajouter Google+ en tant que fournisseur d’identités OAuth2 en utilisant des stratégies personnalisées](identity-provider-google.md)
