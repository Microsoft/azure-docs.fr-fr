---
title: RelyingParty - Azure Active Directory B2C | Microsoft Docs
description: Spécifiez l’élément RelyingParty d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9c50bd71f4e2e5bbe12518f5a5d1cd486af9723a
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509749"
---
# <a name="relyingparty"></a>RelyingParty

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

L’élément **RelyingParty** spécifie le parcours utilisateur à appliquer pour la demande actuelle à Azure Active Directory B2C (Azure AD B2C). Il spécifie également la liste des revendications dont l’application par partie de confiance a besoin dans le cadre du jeton émis. Une application par partie de confiance, telle qu’une application web, mobile ou de bureau, appelle le fichier de stratégie de partie de confiance. Le fichier de stratégie de partie de confiance exécute une tâche spécifique, telle qu’une connexion, une réinitialisation de mot de passe ou une modification de profil. Plusieurs applications peuvent utiliser la même stratégie de partie de confiance, et une application peut utiliser plusieurs stratégies. Toutes les applications par partie de confiance reçoivent le même jeton avec des revendications, et l’utilisateur suit le même parcours utilisateur.

L’exemple suivant montre un élément **RelyingParty** dans le fichier de stratégie *B2C_1A_signup_signin* :

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="your-tenant.onmicrosoft.com"
  PolicyId="B2C_1A_signup_signin"
  PublicPolicyUri="http://your-tenant.onmicrosoft.com/B2C_1A_signup_signin">

  <BasePolicy>
    <TenantId>your-tenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>

  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <UserJourneyBehaviors>
      <SingleSignOn Scope="Tenant" KeepAliveInDays="7"/>
      <SessionExpiryType>Rolling</SessionExpiryType>
      <SessionExpiryInSeconds>300</SessionExpiryInSeconds>
      <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="your-application-insights-key" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      <ContentDefinitionParameters>
        <Parameter Name="campaignId">{OAUTH-KV:campaignId}</Parameter>
      </ContentDefinitionParameters>
    </UserJourneyBehaviors>
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Description>The policy profile</Description>
      <Protocol Name="OpenIdConnect" />
      <Metadata>collection of key/value pairs of data</Metadata>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider" />
        <OutputClaim ClaimTypeReferenceId="loyaltyNumber" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
  ...
```

L’élément facultatif **RelyingParty** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| DefaultUserJourney | 1:1 | Parcours utilisateur par défaut pour l’application par partie de confiance. |
| Points de terminaison | 0:1 | Liste de points de terminaison. Pour plus d’informations, consultez [Point de terminaison UserInfo](userinfo-endpoint.md). |
| UserJourneyBehaviors | 0:1 | Étendue des comportements de parcours utilisateur. |
| TechnicalProfile | 1:1 | Profil technique pris en charge par l’application par partie de confiance. Le profil technique fournit un contrat conformément auquel l’application par partie de confiance contacte Azure AD B2C. |

## <a name="endpoints"></a>Points de terminaison

L’élément **Points de terminaison** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Point de terminaison | 1:1 | Référence à un point de terminaison.|

L’élément **Point de terminaison** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur unique du point de terminaison.|
| UserJourneyReferenceId | Oui | Identificateur du parcours utilisateur dans la stratégie. Pour plus d’informations, consultez [Parcours utilisateur](userjourneys.md)  | 

L’exemple suivant montre une partie de confiance avec un [point de terminaison UserInfo](userinfo-endpoint.md) :

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <Endpoints>
    <Endpoint Id="UserInfo" UserJourneyReferenceId="UserInfoJourney" />
  </Endpoints>
  ...
```

## <a name="defaultuserjourney"></a>DefaultUserJourney

L’élément `DefaultUserJourney` spécifie une référence à l’identificateur du parcours utilisateur qui est généralement défini dans la stratégie De base ou Extensions. Les exemples suivants montrent le parcours utilisateur d’inscription ou de connexion spécifié dans l’élément **RelyingParty** :

Stratégie *B2C_1A_signup_signin* :

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn">
  ...
```

*B2C_1A_TrustFrameWorkBase* ou *B2C_1A_TrustFrameworkExtensionPolicy* :

```xml
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
  ...
```

L’élément **DefaultUserJourney** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | Identificateur du parcours utilisateur dans la stratégie. Pour plus d’informations, consultez [Parcours utilisateur](userjourneys.md) |

## <a name="userjourneybehaviors"></a>UserJourneyBehaviors

L’élément **UserJourneyBehaviors** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| SingleSignOn | 0:1 | Étendue du comportement de session d’authentification unique d’un parcours utilisateur. |
| SessionExpiryType |0:1 | Comportement d’authentification de la session. Valeurs possibles : `Rolling` ou `Absolute`. La valeur `Rolling` (valeur par défaut) indique que l’utilisateur reste connecté tant qu’il est actif dans l’application. La valeur `Absolute` indique que l’utilisateur est obligé de se réauthentifier après le délai spécifié par la durée de vie de session d’application. |
| SessionExpiryInSeconds | 0:1 | Durée de vie du cookie de la session d’Azure AD B2C spécifiée en tant que valeur entière et stockée dans le navigateur de l’utilisateur après une authentification réussie. |
| JourneyInsights | 0:1 | Clé d’instrumentation Azure Application Insights à utiliser. |
| ContentDefinitionParameters | 0:1 | Liste des paires clé/valeur à ajouter à l’URI de charge de définition de contenu. |
|ScriptExecution| 0:1| Modes d’exécution [JavaScript](javascript-and-page-layout.md) pris en charge. Valeurs possibles : `Allow` ou `Disallow` (par défaut).

### <a name="singlesignon"></a>SingleSignOn

L’élément **SingleSignOn** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Étendue | Oui | Étendue du comportement d’authentification unique. Valeurs possibles : `Suppressed`, `Tenant`, `Application` ou `Policy`. La valeur `Suppressed` indique que le comportement est supprimé et que l’utilisateur est toujours invité à sélectionner un fournisseur d’identité.  La valeur `Tenant` indique que le comportement est appliqué à toutes les stratégies dans le locataire. Par exemple, un utilisateur suivant deux parcours de stratégie pour un locataire n’est pas invité à sélectionner un fournisseur d’identité. La valeur `Application` indique que le comportement est appliqué à toutes les stratégies pour l’application qui effectue la requête. Par exemple, un utilisateur suivant deux parcours de stratégie pour une application n’est pas invité à sélectionner un fournisseur d’identité. La valeur `Policy` indique que le comportement s’applique uniquement à une stratégie. Par exemple, un utilisateur suivant deux parcours de stratégie pour une infrastructure de confiance est invité à sélectionner un fournisseur d’identité en cas de basculement d’une stratégie à une autre. |
| KeepAliveInDays | Oui | Contrôle la durée pendant laquelle l’utilisateur reste connecté. Affecter la valeur 0 désactive la fonctionnalité KMSI. Pour plus d’informations, consultez [Maintenir la connexion](session-behavior.md?pivots=b2c-custom-policy#enable-keep-me-signed-in-kmsi). |
|EnforceIdTokenHintOnLogout| Non|  Force la transmission d’un jeton d’ID émis précédemment au point de terminaison de déconnexion en tant qu’indicateur de la session authentifiée active de l’utilisateur final avec le client. Valeurs possibles : `false` (par défaut) ou `true`. Pour plus d’informations, consultez [Connexion web avec OpenID Connect](openid-connect.md).  |


## <a name="journeyinsights"></a>JourneyInsights

L’élément **JourneyInsights** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| TelemetryEngine | Oui | La valeur doit être `ApplicationInsights`. |
| InstrumentationKey | Oui | Chaîne qui contient la clé d’instrumentation pour l’élément application insights. |
| DeveloperMode | Oui | Valeurs possibles : `true` ou `false`. Si `true`, Application Insights envoie la télémétrie par le biais du pipeline de traitement. Ce paramètre est adapté au développement, mais restreint à des volumes élevés. Les journaux d’activité détaillés sont conçus uniquement pour faciliter le développement de stratégies personnalisées. N’utilisez pas le mode de développement en production. Les journaux d’activité recueillent toutes les revendications envoyées par et aux fournisseurs d’identité au cours du développement. En cas d’utilisation en production, le développeur assume la responsabilité des informations d’identification personnelle (PII) recueillies dans le journal Application Insights dont il est propriétaire. Ces journaux d’activité détaillés sont recueillis uniquement quand cette valeur est `true`.|
| ClientEnabled | Oui | Valeurs possibles : `true` ou `false`. Si `true`, envoie le script côté client Application Insights pour le suivi des affichages de page et des erreurs côté client. |
| ServerEnabled | Oui | Valeurs possibles : `true` ou `false`. Si `true`, envoie le JSON UserJourneyRecorder existant en tant qu’événement personnalisé à Application Insights. |
| TelemetryVersion | Oui | La valeur doit être `1.0.0`. |

Pour plus d’informations, consultez [Collecte de journaux d’activité](troubleshoot-with-application-insights.md).

## <a name="contentdefinitionparameters"></a>ContentDefinitionParameters

Les stratégies personnalisées dans Azure AD B2C vous permettent d’envoyer un paramètre dans une chaîne de requête. En transmettant le paramètre à votre point de terminaison HTML, vous pouvez changer le contenu de la page de façon dynamique. Par exemple, vous pouvez changer l’image d’arrière-plan dans la page de connexion ou d’inscription Azure AD B2C en fonction d’un paramètre que vous transmettez depuis votre application web ou mobile. Azure AD B2C transmet les paramètres de chaîne de requête à votre fichier HTML dynamique, tel qu’un fichier aspx.

L’exemple suivant transmet un paramètre nommé `campaignId` avec la valeur `hawaii` dans la chaîne de requête :

`https://login.microsoft.com/contoso.onmicrosoft.com/oauth2/v2.0/authorize?pB2C_1A_signup_signin&client_id=a415078a-0402-4ce3-a9c6-ec1947fcfb3f&nonce=defaultNonce&redirect_uri=http%3A%2F%2Fjwt.io%2F&scope=openid&response_type=id_token&prompt=login&campaignId=hawaii`

L’élément **ContentDefinitionParameters** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| ContentDefinitionParameter | 0:n | Chaîne qui contient la paire clé/valeur qui est ajoutée à la chaîne de requête d’un URI de charge de définition de contenu. |

L’élément **ContentDefinitionParameter** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Nom | Oui | Nom de la paire clé/valeur. |

Pour plus d’informations, consultez [Configurer l’interface utilisateur avec du contenu dynamique à l’aide de stratégies personnalisées](customize-ui-with-html.md#configure-dynamic-custom-page-content-uri).

## <a name="technicalprofile"></a>TechnicalProfile

L’élément **TechnicalProfile** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | La valeur doit être `PolicyProfile`. |

L’élément **TechnicalProfile** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| DisplayName | 1:1 | Chaîne qui contient le nom du profil technique. |
| Description | 0:1 | Chaîne qui contient la description du profil technique. |
| Protocol | 1:1 | Protocole utilisé pour la fédération. |
| Métadonnées | 0:1 | Collection d’*éléments* de paires clé/valeur utilisée par le protocole pour communiquer avec le point de terminaison durant une transaction afin de configurer l’interaction entre la partie de confiance et les autres participants de la communauté. |
| OutputClaims | 1:1 | Liste de types de revendications qui sont pris comme sortie dans le profil technique. Chacun de ces éléments contient une référence à un **ClaimType** déjà défini dans la section **ClaimsSchema** ou dans une stratégie dont hérite ce fichier de stratégie. |
| SubjectNamingInfo | 1:1 | Nom du sujet utilisé dans les jetons. |

L’élément **Protocol** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Nom | Oui | Nom d’un protocole valide pris en charge par Azure AD B2C et utilisé dans le cadre du profil technique. Valeurs possibles : `OpenIdConnect` ou `SAML2`. La valeur `OpenIdConnect` représente la norme de protocole OpenID Connect 1.0 conformément à la spécification OpenID Foundation. `SAML2` représente la norme de protocole SAML 2.0 conformément à la spécification OASIS. |

### <a name="metadata"></a>Métadonnées

Quand le protocole est `SAML`, un élément metadata contient les éléments suivants.

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| IdpInitiatedProfileEnabled | Non | Indique si le flux initié par IDP est pris en charge. Valeurs possibles : `true` ou `false` (par défaut). | 
| XmlSignatureAlgorithm | Non | Méthode utilisée par Azure AD B2C pour signer la réponse SAML. Valeurs possibles : `Sha256`, `Sha384`, `Sha512` ou `Sha1`. Veillez à configurer l’algorithme de signature des deux côtés avec la même valeur. Utilisez uniquement l’algorithme pris en charge par votre certificat. Pour configurer l’assertion SAML, consultez les [métadonnées du profil technique de l’émetteur SAML](saml-issuer-technical-profile.md#metadata). |
| DataEncryptionMethod | Non | Indique la méthode utilisée par Azure AD B2C pour chiffrer les données, à l’aide de l’algorithme Advanced Encryption Standard (AES). Les métadonnées contrôlent la valeur de l’élément `<EncryptedData>` dans la réponse SAML. Valeurs possibles : `Aes256` (par défaut), `Aes192`, `Sha512`, ou ` Aes128`. |
| KeyEncryptionMethod| Non | Indique la méthode utilisée par Azure AD B2C pour chiffrer la copie de la clé qui a été utilisée pour chiffrer les données. Les métadonnées contrôlent la valeur de l’élément `<EncryptedKey>` dans la réponse SAML. Valeurs possibles : ` Rsa15` (par défaut), algorithme PKCS (Public Key Cryptography standard) RSA version 1.5 ; ` RsaOaep`, algorithme de chiffrement OAEP (Optimal Asymmetric Encryption Padding) RSA. |
| UseDetachedKeys | Non |  Valeurs possibles : `true` ou `false` (par défaut). Lorsque la valeur est définie sur `true`, Azure AD B2C modifie le format des assertions chiffrées. L’utilisation de clés détachées ajoute l’assertion chiffrée comme enfant d’EncrytedAssertion, par opposition à EncryptedData. |
| WantsSignedResponses| Non | Indique si Azure AD B2C signe la section `Response` de la réponse SAML. Valeurs possibles : `true` (par défaut) ou `false`.  |

### <a name="outputclaims"></a>OutputClaims

L’élément **OutputClaims** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| OutputClaim | 0:n | Nom d’un type de revendication attendu dans la liste prise en charge pour la stratégie à laquelle la partie de confiance s’abonne. Cette revendication sert de sortie pour le profil technique. |

L’élément **OutputClaim** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Oui | Référence à un **ClaimType** déjà défini dans la section **ClaimsSchema** du fichier de stratégie. |
| DefaultValue | Non | Valeur par défaut qui peut être utilisée si la valeur de revendication est vide. |
| PartnerClaimType | Non | Envoie la revendication sous un autre nom, tel que configuré dans la définition de ClaimType. |

### <a name="subjectnaminginfo"></a>SubjectNamingInfo

Avec l’élément **SubjectNameingInfo**, vous contrôlez la valeur du sujet du jeton :
- **Jeton JWT** : la revendication `sub`. Il s’agit d’un principal sur lequel portent les assertions d’informations du jeton, comme l’utilisateur d’une application. Cette valeur est immuable et ne peut pas être réattribuée ou réutilisée. Vous pouvez l’utiliser pour effectuer des vérifications d’autorisation en toute sécurité, par exemple quand le jeton est utilisé pour accéder à une ressource. Par défaut, la revendication de l’objet est remplie avec l’ID d’objet de l’utilisateur dans le répertoire. Pour plus d’informations, consultez [Configuration du jeton, de la session et de l’authentification unique](session-behavior.md).
- **Jeton SAML** : élément `<Subject><NameID>` qui identifie l’élément de sujet. Le format NameId peut être modifié.

L’élément **SubjectNamingInfo** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClaimType | Oui | Référence au **PartnerClaimType** d’une revendication de sortie. Les revendications de sortie doivent être définies dans la collection **OutputClaims** de la stratégie de partie de confiance. |
| Format | Non | Utilisé pour les parties de confiance SAML, afin de définir le format **NameId** retourné dans l’assertion SAML. |

L’exemple suivant montre comment définir une partie de confiance OpenID Connect. Les informations de nom de sujet sont configurées en tant qu’`objectId` :

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
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```
Le jeton JWT inclut la revendication `sub` avec l’objectId d’utilisateur :

```json
{
  ...
  "sub": "6fbbd70d-262b-4b50-804c-257ae1706ef2",
  ...
}
```

L’exemple suivant montre comment définir une partie de confiance SAML. Les informations relatives au nom du sujet sont configurées en tant que `objectId`, et le `format` NameId a été fourni :

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient"/>
  </TechnicalProfile>
</RelyingParty>
```
