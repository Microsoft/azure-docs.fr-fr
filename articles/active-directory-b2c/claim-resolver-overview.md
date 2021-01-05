---
title: Programmes de résolution de revendication
titleSuffix: Azure AD B2C
description: Découvrez comment utiliser les programme de résolution de revendication dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/28/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 682b83d7016a89b27b5c936853abda1438f59c28
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508014"
---
# <a name="about-claim-resolvers-in-azure-active-directory-b2c-custom-policies"></a>À propos des résolveurs de revendication dans les stratégies personnalisées d’Azure Active Directory B2C

Les résolveurs de revendication dans les [stratégies personnalisées](custom-policy-overview.md) d’Azure Active Directory B2C (Azure AD B2C) fournissent des informations de contexte sur une demande d’autorisation, telles que le nom de la stratégie, l’ID de corrélation de la demande ou la langue de l’interface utilisateur, entre autres.

Pour utiliser un résolveur de revendication dans une revendication d’entrée ou de sortie, vous définissez une chaîne **ClaimType** sous l’élément [ClaimsSchema](claimsschema.md), puis vous définissez **DefaultValue** sur le résolveur de revendication dans l’élément de la revendication d’entrée ou de sortie. Azure AD B2C lit la valeur du résolveur de revendication, puis l’utilise dans le profil technique.

Dans l’exemple suivant, un type de revendication nommé `correlationId` est défini avec un **DataType** de `string`.

```xml
<ClaimType Id="correlationId">
  <DisplayName>correlationId</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Request correlation Id</UserHelpText>
</ClaimType>
```

Dans le profil technique, mappez le résolveur de revendication au type de revendication. Azure AD B2C renseigne la valeur du résolveur de revendication `{Context:CorrelationId}` dans la revendication `correlationId` et envoie la demande au profil technique.

```xml
<InputClaim ClaimTypeReferenceId="correlationId" DefaultValue="{Context:CorrelationId}" />
```

## <a name="claim-resolver-types"></a>Types de résolveur de revendication

Les sections suivantes répertorient les résolveurs de revendication disponibles.

### <a name="culture"></a>Culture

| Revendication | Description |  Exemple |
| ----- | ----------- | --------|
| {Culture:LanguageName} | Code ISO à deux lettres pour la langue. | en |
| {Culture:LCID}   | LCID du code de langue | 1033 |
| {Culture:RegionName} | Code ISO à deux lettres pour la région. | US |
| {Culture:RFC5646} | Code de langue RFC5646. | fr-FR |

### <a name="policy"></a>Stratégie

| Revendication | Description |  Exemple |
| ----- | ----------- | --------|
| {Policy:PolicyId} | Nom de la stratégie de partie de confiance. | B2C_1A_signup_signin |
| {Policy:RelyingPartyTenantId} | ID de locataire de la stratégie de partie de confiance. | votre-locataire.onmicrosoft.com |
| {Policy:TenantObjectId} | ID d’objet de locataire de la stratégie de partie de confiance. | 00000000-0000-0000-0000-000000000000 |
| {Policy:TrustFrameworkTenantId} | ID de locataire de l’infrastructure de confiance. | votre-locataire.onmicrosoft.com |

### <a name="openid-connect"></a>OpenID Connect

| Revendication | Description |  Exemple |
| ----- | ----------- | --------|
| {OIDC:AuthenticationContextReferences} |Paramètre de chaîne de requête `acr_values`. | N/A |
| {OIDC:ClientId} |Paramètre de chaîne de requête `client_id`. | 00000000-0000-0000-0000-000000000000 |
| {OIDC:DomainHint} |Paramètre de chaîne de requête `domain_hint`. | facebook.com |
| {OIDC:LoginHint} |  Paramètre de chaîne de requête `login_hint`. | someone@contoso.com |
| {OIDC:MaxAge} | Le `max_age`. | N/A |
| {OIDC:Nonce} |Paramètre de chaîne de requête `Nonce`. | defaultNonce |
| {OIDC:Password}| Mot de passe utilisateur du [flux des informations d’identification par mot de passe du propriétaire de ressource](ropc-custom.md).| password1| 
| {OIDC:Prompt} | Paramètre de chaîne de requête `prompt`. | login |
| {OIDC:RedirectUri} |Paramètre de chaîne de requête `redirect_uri`. | https://jwt.ms |
| {OIDC:Resource} |Paramètre de chaîne de requête `resource`. | N/A |
| {OIDC:Scope} |Paramètre de chaîne de requête `scope`. | openid |
| {OIDC:Username}| Nom d’utilisateur de l’utilisateur du [flux des informations d’identification par mot de passe du propriétaire de ressource](ropc-custom.md).| emily@contoso.com| 

### <a name="context"></a>Context

| Revendication | Description |  Exemple |
| ----- | ----------- | --------|
| {Context:BuildNumber} | Version de l’infrastructure d’expérience d’identité (numéro de build).  | 1.0.507.0 |
| {Context:CorrelationId} | L’ID de corrélation.  | 00000000-0000-0000-0000-000000000000 |
| {Context:DateTimeInUtc} |Date et heure UTC.  | 10/10/2018 12:00:00 |
| {Context:DeploymentMode} |Mode de déploiement de la stratégie.  | Production |
| {Context:IPAddress} | Adresse IP utilisateur. | 11.111.111.11 |
| {Context:KMSI} | Indique si la case [Maintenir la connexion](session-behavior.md?pivots=b2c-custom-policy#enable-keep-me-signed-in-kmsi) est cochée. |  true |

### <a name="claims"></a>Revendications 

| Revendication | Description |  Exemple |
| ----- | ----------- | --------|
| {Claim:claim type} | Identificateur d’un type de revendication déjà défini dans la section ClaimsSchema du fichier de stratégie ou d’un fichier de stratégie parent.  Par exemple : `{Claim:displayName}` ou `{Claim:objectId}`. | Valeur du type de revendication.|


### <a name="oauth2-key-value-parameters"></a>Paramètres clé-valeur OAuth2

Tous les noms de paramètre inclus dans le cadre d’une requête OIDC ou OAuth2 peuvent être mappés à une revendication dans le parcours utilisateur. Par exemple, la demande de l’application peut inclure un paramètre de chaîne de requête avec le nom `app_session`, `loyalty_number` ou toute chaîne de requête personnalisée.

| Revendication | Description |  Exemple |
| ----- | ----------------------- | --------|
| {OAUTH-KV:campaignId} | Paramètre de chaîne de requête. | Hawaii |
| {OAUTH-KV:app_session} | Paramètre de chaîne de requête. | A3C5R |
| {OAUTH-KV:loyalty_number} | Paramètre de chaîne de requête. | 1234 |
| {OAUTH-KV : n’importe quelle chaîne de requête personnalisée} | Paramètre de chaîne de requête. | N/A |

### <a name="oauth2"></a>OAuth2

| Revendication | Description |  Exemple |
| ----- | ----------------------- | --------|
| {oauth2:access_token} | Jeton d’accès. | N/A |
| {oauth2:refresh_token} | Le jeton d’actualisation. | N/A |


### <a name="saml"></a>SAML

| Revendication | Description |  Exemple |
| ----- | ----------- | --------|
| {SAML:AuthnContextClassReferences} | Valeur de l'élément `AuthnContextClassRef`, provenant de la demande SAML. | urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport |
| {SAML:NameIdPolicyFormat} | Attribut `Format`, provenant de l'élément `NameIDPolicy` de la demande SAML. | urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress |
| {SAML:Issuer} |  Valeur de l'élément `Issuer` SAML de la demande SAML.| `https://contoso.com` |
| {SAML:AllowCreate} | Valeur de l'attribut `AllowCreate`, provenant de l'élément `NameIDPolicy` de la demande SAML. | True |
| {SAML:ForceAuthn} | Valeur de l'attribut `ForceAuthN`, provenant de l'élément `AuthnRequest` de la demande SAML. | True |
| {SAML:ProviderName} | Valeur de l'attribut `ProviderName`, provenant de l'élément `AuthnRequest` de la demande SAML.| Contoso.com |
| {SAML:RelayState} | Paramètre de chaîne de requête `RelayState`.| 
| {SAML:Subject} | `Subject` de l’élément NameId de la demande SAML AuthN.| 

## <a name="using-claim-resolvers"></a>Utilisation de programmes de résolution de revendications

Vous pouvez utiliser des programmes de résolution de revendications avec les éléments suivants :

| Élément | Élément | Paramètres |
| ----- | ----------------------- | --------|
|Profil technique Application Insights |`InputClaim` | |
|Profil technique [Azure Active Directory](active-directory-technical-profile.md)| `InputClaim`, `OutputClaim`| 1, 2|
|Profil technique [OAuth2](oauth2-technical-profile.md)| `InputClaim`, `OutputClaim`| 1, 2|
|Profil technique [OpenID Connect](openid-connect-technical-profile.md)| `InputClaim`, `OutputClaim`| 1, 2|
|Profil technique de [transformation de revendications](claims-transformation-technical-profile.md)| `InputClaim`, `OutputClaim`| 1, 2|
|Profil technique du [fournisseur RESTful](restful-technical-profile.md)| `InputClaim`| 1, 2|
|Profil technique du [fournisseur d’identité SAML](saml-identity-provider-technical-profile.md)| `OutputClaim`| 1, 2|
|Profil technique [Autodéclaré](self-asserted-technical-profile.md)| `InputClaim`, `OutputClaim`| 1, 2|
|[ContentDefinition](contentdefinitions.md)| `LoadUri`| |
|[ContentDefinitionParameters](relyingparty.md#contentdefinitionparameters)| `Parameter` | |
|Profil technique [RelyingParty](relyingparty.md#technicalprofile)| `OutputClaim`| 2 |

Paramètres :
1. Les métadonnées `IncludeClaimResolvingInClaimsHandling` doivent être définies sur `true`.
1. L'attribut des revendications d'entrée ou de sortie `AlwaysUseDefaultValue` doit être défini sur `true`.

## <a name="claim-resolvers-samples"></a>Exemples de programmes de résolution de revendications

### <a name="restful-technical-profile"></a>Profil technique RESTful

Dans un profil technique [RESTful](restful-technical-profile.md), vous pouvez envoyer la langue de l’utilisateur, le nom de la stratégie, l’étendue et l’ID client. Selon ces revendications, l’API REST peut exécuter une logique métier personnalisée et éventuellement déclencher un message d’erreur localisé.

L'exemple suivant illustre un profil technique RESTful en lien avec ce scénario :

```xml
<TechnicalProfile Id="REST">
  <DisplayName>Validate user input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app.azurewebsites.net/api/identity</Item>
    <Item Key="AuthenticationType">None</Item>
    <Item Key="SendClaimsIn">Body</Item>
    <Item Key="IncludeClaimResolvingInClaimsHandling">true</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userLanguage" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
    <InputClaim ClaimTypeReferenceId="policyName" DefaultValue="{Policy:PolicyId}" AlwaysUseDefaultValue="true" />
    <InputClaim ClaimTypeReferenceId="scope" DefaultValue="{OIDC:Scope}" AlwaysUseDefaultValue="true" />
    <InputClaim ClaimTypeReferenceId="clientId" DefaultValue="{OIDC:ClientId}" AlwaysUseDefaultValue="true" />
  </InputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

### <a name="direct-sign-in"></a>Connexion directe

À l’aide de résolveurs de revendication, vous pouvez préremplir le nom de connexion ou diriger la connexion vers un fournisseur d’identité sociale spécifique, comme Facebook, LinkedIn ou un compte Microsoft. Pour plus d’informations, consultez [Configurer la connexion directe avec Azure Active Directory B2C](direct-signin.md).

### <a name="dynamic-ui-customization"></a>Personnalisation dynamique de l’interface utilisateur

Azure AD B2C vous permet de transmettre des paramètres de chaîne de requête à vos points de terminaison de définition de contenu HTML afin d'afficher dynamiquement le contenu de la page. Par exemple, cette fonctionnalité permet de changer l’image d’arrière-plan sur la page de connexion ou d’inscription Azure AD B2C en fonction d’un paramètre personnalisé que vous transmettez depuis votre application web ou mobile. Pour plus d’informations, consultez [Configurer dynamiquement l’interface utilisateur à l’aide de stratégies personnalisées dans Azure Active Directory B2C](customize-ui-with-html.md#configure-dynamic-custom-page-content-uri). Vous pouvez également localiser votre page HTML en fonction d’un paramètre de langue ou changer le contenu selon l’ID client.

L'exemple suivant transmet dans la chaîne de requête les paramètres **campaignId** (valeur `Hawaii`), **language** (valeur `en-US`), et **app** (représentant l'ID client) :

```xml
<UserJourneyBehaviors>
  <ContentDefinitionParameters>
    <Parameter Name="campaignId">{OAUTH-KV:campaignId}</Parameter>
    <Parameter Name="language">{Culture:RFC5646}</Parameter>
    <Parameter Name="app">{OIDC:ClientId}</Parameter>
  </ContentDefinitionParameters>
</UserJourneyBehaviors>
```

Ainsi, Azure AD B2C envoie les paramètres ci-dessus à la page de contenu HTML :

```
/selfAsserted.aspx?campaignId=hawaii&language=en-US&app=0239a9cc-309c-4d41-87f1-31288feb2e82
```

### <a name="content-definition"></a>Définition de contenu

Une définition de contenu ([ContentDefinition](contentdefinitions.md)`LoadUri`) vous permet d'envoyer des programmes de résolution de revendications pour extraire du contenu à partir de différents emplacements, conformément aux paramètres utilisés.

```xml
<ContentDefinition Id="api.signuporsignin">
  <LoadUri>https://contoso.blob.core.windows.net/{Culture:LanguageName}/myHTML/unified.html</LoadUri>
  ...
</ContentDefinition>
```

### <a name="application-insights-technical-profile"></a>Profil technique Application Insights

Avec Azure Application Insights et les résolveurs de revendication, vous pouvez obtenir des insights sur le comportement de l’utilisateur. Dans le profil technique Application Insights, vous envoyez des revendications d’entrée qui sont conservées dans Azure Application Insights. Pour plus d’informations, consultez [Suivre le comportement des utilisateurs dans les parcours Azure AD B2C à l’aide d’Application Insights](analytics-with-application-insights.md). L’exemple suivant envoie l’ID de stratégie, l’ID de corrélation, la langue et l’ID client à Azure Application Insights.

```xml
<TechnicalProfile Id="AzureInsights-Common">
  <DisplayName>Alternate Email</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.Insights.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="PolicyId" PartnerClaimType="{property:Policy}" DefaultValue="{Policy:PolicyId}" />
    <InputClaim ClaimTypeReferenceId="CorrelationId" PartnerClaimType="{property:CorrelationId}" DefaultValue="{Context:CorrelationId}" />
    <InputClaim ClaimTypeReferenceId="language" PartnerClaimType="{property:language}" DefaultValue="{Culture:RFC5646}" />
    <InputClaim ClaimTypeReferenceId="AppId" PartnerClaimType="{property:App}" DefaultValue="{OIDC:ClientId}" />
  </InputClaims>
</TechnicalProfile>
```

### <a name="relying-party-policy"></a>Stratégie de partie de confiance

Le profil technique d'une stratégie [Partie de confiance](relyingparty.md) vous permet d'envoyer l'ID du locataire ou l'ID de corrélation à l'application par partie de confiance dans le jeton JWT.

```xml
<RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
        <OutputClaim ClaimTypeReferenceId="correlationId" AlwaysUseDefaultValue="true" DefaultValue="{Context:CorrelationId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
```
