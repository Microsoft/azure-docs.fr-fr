---
title: Définir un profil technique OpenID Connect dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définissez un profil technique OpenID Connect dans une stratégie personnalisée d'Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/01/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 8364e67e71143729e97c5253f0dfd7b30a1e5c2f
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559818"
---
# <a name="define-an-openid-connect-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique OpenID Connect dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge le fournisseur d’identité du protocole [OpenID Connect](https://openid.net/2015/04/17/openid-connect-certification-program/). OpenID Connect 1.0 définit une couche d’identité par dessus OAuth 2.0, et est à la pointe des protocoles d’authentification modernes. Un profil technique OpenID Connect vous permet d'opérer une fédération avec un fournisseur d'identité reposant sur OpenID Connect, comme Azure AD. Fédérer avec un fournisseur d’identité permet aux utilisateurs de se connecter avec leurs identités existantes de réseaux sociaux ou d’entreprise.

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `OpenIdConnect`. Par exemple, le protocole pour le profil technique **MSA-OIDC** est `OpenIdConnect` :

```xml
<TechnicalProfile Id="MSA-OIDC">
  <DisplayName>Microsoft Account</DisplayName>
  <Protocol Name="OpenIdConnect" />
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

L’élément **OutputClaims** contient une liste de revendications renvoyée par le fournisseur d'identité OpenID Connect. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans le fournisseur d'identité. Vous pouvez également inclure des revendications qui ne sont pas retournées par le fournisseur d’identité, pour autant que vous définissiez l’attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

L’exemple suivant montre les revendications retournées par le fournisseur d’identité Compte Microsoft :

- La revendication **sub** mappée à la revendication **issuerUserId**.
- La revendication **name** mappée à la revendication **displayName**.
- La revendication **e-mail** sans mappage de nom.

Le profil technique retourne également des revendications qui ne sont pas retournées par le fournisseur d’identité :

- La revendication **identityProvider** contenant le nom du fournisseur d’identité.
- La revendication **authenticationSource** avec la valeur par défaut **socialIdpAuthentication**.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="live.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
</OutputClaims>
```

## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| client_id | Oui | Identificateur d’application du fournisseur d’identité. |
| IdTokenAudience | Non | Audience du jeton id_token. Si la valeur est spécifiée, Azure AD B2C vérifie si la revendication `aud` dans un jeton retourné par le fournisseur d’identité est identique à celle indiquée dans les métadonnées IdTokenAudience.  |
| METADATA | Oui | URL qui pointe vers un document de configuration de fournisseur d’identité OpenID Connect, également appelé point de terminaison de configuration OpenID connu. L’URL peut contenir l’expression `{tenant}`, qui est remplacée par le nom du locataire.  |
| authorization_endpoint | Non | URL qui pointe vers un point de terminaison d’autorisation de configuration de fournisseur d’identité OpenID Connect. La valeur des métadonnées authorization_endpoint est prioritaire sur celle de `authorization_endpoint` spécifiée dans le point de terminaison de configuration OpenID connu. L’URL peut contenir l’expression `{tenant}`, qui est remplacée par le nom du locataire. |
| end_session_endpoint | Non | URL du point de terminaison de la session de fin. La valeur des métadonnées authorization_endpoint est prioritaire sur celle de `end_session_endpoint` spécifiée dans le point de terminaison de configuration OpenID connu. |
| émetteur | Non | Identificateur unique d’un fournisseur d’identité OpenID Connect. La valeur des métadonnées de l’émetteur est prioritaire sur celle de `issuer` spécifiée dans le point de terminaison de configuration OpenID connu.  Si la valeur est spécifiée, Azure AD B2C vérifie si la revendication `iss` dans un jeton retourné par le fournisseur d’identité est identique à celle indiquée dans les métadonnées de l’émetteur. |
| ProviderName | Non | Nom du fournisseur d'identité.  |
| response_types | Non | Type de réponse conformément à la spécification OpenID Connect Core 1.0. Valeurs possibles : `id_token`, `code` ou `token`. |
| response_mode | Non | Méthode que le fournisseur d’identité utilise pour renvoyer le résultat à Azure AD B2C. Valeurs possibles : `query`, `form_post` (par défaut) ou `fragment`. |
| scope | Non | Étendue de la requête définie conformément à la spécification OpenID Connect Core 1.0. Par exemple, `openid`, `profile` ou `email`. |
| HttpBinding | Non | Liaison HTTP attendue aux points de terminaison de jeton d’accès et de jeton de revendications. Valeurs possibles : `GET` ou `POST`.  |
| ValidTokenIssuerPrefixes | Non | Clé utilisable pour se connecter à chacun des locataires en cas d’utilisation d’un fournisseur d’identité mutualisé tel qu’Azure Active Directory. |
| UsePolicyInRedirectUri | Non | Indique s’il faut utiliser une stratégie lors de la construction de l’URI de redirection. Lorsque vous configurez votre application dans le fournisseur d’identité, vous devez spécifier l’URI de redirection. L’URI de redirection pointe vers Azure AD B2C, `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp`.  Si vous spécifiez `false`, vous devez ajouter un URI de redirection pour chaque stratégie que vous utilisez. Par exemple : `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/{policy-name}/oauth2/authresp`. |
| MarkAsFailureOnStatusCode5xx | Non | Indique si une demande adressée à un service externe doit être marquée comme un échec si le code d’état HTTP s’inscrit dans la plage 5xx. Par défaut, il s’agit de `false`. |
| DiscoverMetadataByTokenIssuer | Non | Indique si les métadonnées OIDC doivent être découvertes à l’aide de l’émetteur dans le jeton JSON Web Token. |
| IncludeClaimResolvingInClaimsHandling  | Non | Pour les revendications d’entrée et de sortie, spécifie si la [résolution des revendications](claim-resolver-overview.md) est incluse dans le profil technique. Valeurs possibles : `true` ou `false` (par défaut). Si vous souhaitez utiliser un programme de résolution des revendications dans le profil technique, définissez cette valeur sur `true`. |
|token_endpoint_auth_method| Non| Indique comment Azure AD B2C envoie l’en-tête d’authentification au point de terminaison du jeton. Valeurs possibles : `client_secret_post` (par défaut), `private_key_jwt` (préversion publique) et `client_secret_basic` (préversion publique). Pour plus d’informations, consultez [Section d’authentification du client OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication). |
|SingleLogoutEnabled| Non| Indique si, lors de la connexion, le profil technique tente de se déconnecter des fournisseurs d’identité fédérés. Pour plus d’informations, consultez [Déconnexion d’une session Azure AD B2C](session-behavior.md#sign-out).  Valeurs possibles : `true` (par défaut) ou `false`.|

```xml
<Metadata>
  <Item Key="ProviderName">https://login.live.com</Item>
  <Item Key="METADATA">https://login.live.com/.well-known/openid-configuration</Item>
  <Item Key="response_types">code</Item>
  <Item Key="response_mode">form_post</Item>
  <Item Key="scope">openid profile email</Item>
  <Item Key="HttpBinding">POST</Item>
  <Item Key="UsePolicyInRedirectUri">false</Item>
  <Item Key="client_id">Your Microsoft application client ID</Item>
</Metadata>
```

### <a name="ui-elements"></a>Éléments d’interface utilisateur
 
Les paramètres suivants peuvent être utilisés pour configurer le message d’erreur affiché en cas d’échec. Les métadonnées doivent être configurées dans le profil technique OpenID Connect. Les messages d’erreur peuvent être [localisés](localization-string-ids.md#sign-up-or-sign-in-error-messages).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| UserMessageIfClaimsPrincipalDoesNotExist | Non | Message à afficher à l’utilisateur si un compte avec le nom d’utilisateur fourni est introuvable dans l’annuaire. |
| UserMessageIfInvalidPassword | Non | Message à afficher à l’utilisateur si le mot de passe est incorrect. |
| UserMessageIfOldPasswordUsed| Non |  Message à afficher à l’utilisateur si le mot de passe utilisé est ancien.|

## <a name="cryptographic-keys"></a>Clés de chiffrement

L’élément **CryptographicKeys** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| client_secret | Oui | Clé secrète client de l’application du fournisseur d’identité. La clé de chiffrement est requise uniquement si les métadonnées **response_types** sont définies sur `code`. Dans ce cas, Azure AD B2C émet un autre appel pour échanger le code d’autorisation pour un jeton d’accès. Si les métadonnées sont définies sur `id_token`, vous pouvez omettre la clé de chiffrement.  |

## <a name="redirect-uri"></a>URI de redirection

Lorsque vous configurez l’URI de redirection de votre fournisseur d’identité, entrez `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp`. Veillez à remplacer `{your-tenant-name}` par le nom de votre locataire. L’URI de redirection doit être en minuscules.

Exemples :

- [Ajouter Compte Microsoft (MSA) comme fournisseur d’identité utilisant des stratégies personnalisées](identity-provider-microsoft-account.md)
- [Se connecter à l’aide de comptes Azure AD](identity-provider-azure-ad-single-tenant.md)
- [Autoriser la connexion d’utilisateurs à un fournisseur d’identité Azure AD mutualisé à l’aide de stratégies personnalisées](identity-provider-azure-ad-multi-tenant.md)
