---
title: Gestion de sessions d’authentification unique avec des stratégies personnalisées
titleSuffix: Azure AD B2C
description: Découvrez comment gérer des sessions d’authentification unique (SSO) à l’aide de stratégies personnalisées dans Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/07/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 453042766c427b05ec1ee1090a0702f64065542d
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508048"
---
# <a name="single-sign-on-session-management-in-azure-active-directory-b2c"></a>Gestion de session d’authentification unique dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

La gestion de [session d’authentification unique](session-behavior.md) utilise la même sémantique que tout autre profil technique dans des stratégies personnalisées. Quand une étape d’orchestration est exécutée, le profil technique associé à l’étape est interrogé pour obtenir une référence `UseTechnicalProfileForSessionManagement`. S’il en existe une, le fournisseur de la session de l’authentification unique référencé est vérifié pour voir si l’utilisateur est un participant de la session. Dans l’affirmative, le fournisseur de session d’authentification unique est utilisé pour remplir à nouveau la session. De même, lors de l’exécution d’une étape d’orchestration est terminée, le fournisseur est utilisé pour stocker les informations de la session si un fournisseur de session d’authentification unique a été spécifié.

Azure AD B2C a défini un certain nombre de fournisseurs de session d’authentification unique qui peuvent être utilisés :

|Fournisseur de session  |Étendue  |
|---------|---------|
|[NoopSSOSessionProvider](#noopssosessionprovider)     |  None       |       
|[DefaultSSOSessionProvider](#defaultssosessionprovider)    | Gestionnaire de session interne Azure AD B2C.      |       
|[ExternalLoginSSOSessionProvider](#externalloginssosessionprovider)     | Entre Azure AD B2C et le fournisseur d’identité OAuth1, OAuth2 ou OpenId Connect.        |         |
|[OAuthSSOSessionProvider](#oauthssosessionprovider)     | Entre une application de partie de confiance OAuth2 ou OpenId Connect et Azure AD B2C.        |        
|[SamlSSOSessionProvider](#samlssosessionprovider)     | Entre Azure AD B2C et le fournisseur d’identité SAML. Et entre un fournisseur de services SAML (application de partie de confiance) et Azure AD B2C.  |        




Les classes de gestion de l’authentification unique sont spécifiées à l’aide de l’élément `<UseTechnicalProfileForSessionManagement ReferenceId="{ID}" />` d’un profil technique.

## <a name="input-claims"></a>Revendications d’entrée

L’élément `InputClaims` est vide ou absent.

## <a name="persisted-claims"></a>Revendications persistantes

Les revendications qui doivent être retournées à l’application ou utilisées par les conditions préalables dans les étapes suivantes doivent être stockées dans la session ou complétées par une lecture depuis le profil de l’utilisateur dans le répertoire. L’utilisation de revendications persistantes garantit que vos parcours d’authentification n’échouent pas en raison de revendications manquantes. Pour ajouter des revendications dans la session, utilisez l’élément `<PersistedClaims>` du profil technique. Lorsque le fournisseur est utilisé pour remplir à nouveau la session, les revendications persistantes sont ajoutées au jeu de revendications.

## <a name="output-claims"></a>Revendications de sortie

`<OutputClaims>` est utilisé pour récupérer des revendications de la session.

## <a name="session-providers"></a>Fournisseurs de session

### <a name="noopssosessionprovider"></a>NoopSSOSessionProvider

Comme son nom l’indique, ce fournisseur ne fait rien. Ce fournisseur peut être utilisé pour supprimer un comportement d’authentification unique pour un profil technique spécifique. Le profil technique `SM-Noop` suivant est inclus dans le [pack de démarrage de stratégie personnalisée](custom-policy-get-started.md#custom-policy-starter-pack).

```xml
<TechnicalProfile Id="SM-Noop">
  <DisplayName>Noop Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.NoopSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

### <a name="defaultssosessionprovider"></a>DefaultSSOSessionProvider

Ce fournisseur peut être utilisé pour le stockage de revendications dans une session. Ce fournisseur est en général référencé dans un profil technique utilisé pour la gestion de comptes locaux et fédérés. Le profil technique `SM-AAD` suivant est inclus dans le [pack de démarrage de stratégie personnalisée](custom-policy-get-started.md#custom-policy-starter-pack).

```xml
<TechnicalProfile Id="SM-AAD">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.DefaultSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="objectId" />
    <PersistedClaim ClaimTypeReferenceId="signInName" />
    <PersistedClaim ClaimTypeReferenceId="authenticationSource" />
    <PersistedClaim ClaimTypeReferenceId="identityProvider" />
    <PersistedClaim ClaimTypeReferenceId="newUser" />
    <PersistedClaim ClaimTypeReferenceId="executed-SelfAsserted-Input" />
  </PersistedClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectIdFromSession" DefaultValue="true"/>
  </OutputClaims>
</TechnicalProfile>
```


Le profil technique `SM-MFA` suivant est inclus dans le [pack de démarrage de stratégie personnalisée](custom-policy-get-started.md#custom-policy-starter-pack) `SocialAndLocalAccountsWithMfa`. Ce profil technique gère la session d’authentification multifacteur.

```xml
<TechnicalProfile Id="SM-MFA">
  <DisplayName>Session Mananagement Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.DefaultSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="Verified.strongAuthenticationPhoneNumber" />
  </PersistedClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isActiveMFASession" DefaultValue="true"/>
  </OutputClaims>
</TechnicalProfile>
```

### <a name="externalloginssosessionprovider"></a>ExternalLoginSSOSessionProvider

Ce fournisseur est utilisé pour supprimer l’écran « Choisir un fournisseur d’identité » et se déconnecter d’un fournisseur d’identité fédéré. Il est généralement référencé dans un profil technique configuré pour un fournisseur d’identité fédéré tel que Facebook ou Azure Active Directory. Le profil technique `SM-SocialLogin` suivant est inclus dans le [pack de démarrage de stratégie personnalisée](custom-policy-get-started.md#custom-policy-starter-pack).

```xml
<TechnicalProfile Id="SM-SocialLogin">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.ExternalLoginSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="AlternativeSecurityId" />
  </PersistedClaims>
</TechnicalProfile>
```

#### <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description|
| --- | --- | --- |
| AlwaysFetchClaimsFromProvider | Non | Non utilisé actuellement, peut être ignoré. |

### <a name="oauthssosessionprovider"></a>OAuthSSOSessionProvider

Ce fournisseur est utilisé pour gérer les sessions Azure AD B2C entre une partie de confiance OAuth2 ou OpenId Connect et Azure AD B2C.

```xml
<TechnicalProfile Id="SM-jwt-issuer">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.OAuthSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

### <a name="samlssosessionprovider"></a>SamlSSOSessionProvider

Ce fournisseur est utilisé pour gérer les sessions SAML d’Azure AD B2C entre une application basée sur les revendications et un fournisseur d’identité SAML fédéré. Lorsque vous utilisez le fournisseur d’authentification unique pour stocker une session de fournisseur d’identité SAML, `RegisterServiceProviders` doit avoir la valeur `false`. Le profil technique `SM-Saml-idp` suivant est utilisé par le [profil technique du fournisseur d’identité SAML](saml-identity-provider-technical-profile.md).

```xml
<TechnicalProfile Id="SM-Saml-idp">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="RegisterServiceProviders">false</Item>
  </Metadata>
</TechnicalProfile>
```

Lorsque vous utilisez le fournisseur pour stocker la session SAML B2C, `RegisterServiceProviders` doit avoir la valeur `true`. La fermeture de la session SAML requiert les valeurs `SessionIndex` et `NameID` pour aboutir.

Le profil technique `SM-Saml-issuer` suivant est utilisé par le [profil technique de l’émetteur SAML](saml-issuer-technical-profile.md)

```xml
<TechnicalProfile Id="SM-Saml-issuer">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
</TechnicalProfile>
```

#### <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description|
| --- | --- | --- |
| IncludeSessionIndex | Non | Non utilisé actuellement, peut être ignoré.|
| RegisterServiceProviders | Non | Indique que le fournisseur doit inscrire tous les fournisseurs de services SAML auxquels une assertion a été envoyée. Valeurs possibles : `true` (par défaut) ou `false`.|


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [configurer un comportement de session](session-behavior.md).
