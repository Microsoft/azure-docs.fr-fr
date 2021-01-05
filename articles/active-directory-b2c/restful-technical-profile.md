---
title: Définir un profil technique RESTful dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définir un profil technique RESTful dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 891991fa938ad3dcfacae6d02e40efd6d6e9689e
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97386848"
---
# <a name="define-a-restful-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique RESTful dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) assure la prise en charge de l’intégration de votre propre service RESTful. Azure Active Directory B2C envoie des données au service RESTful dans une collection de revendications d’entrée et reçoit des données en retour dans une collection de revendications de sortie. Pour plus d’informations, consultez [Intégrer les échanges de revendications de l’API REST dans votre stratégie personnalisée Azure AD B2C](custom-policy-rest-api-intro.md).  

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `Proprietary`. L’attribut **handler** doit contenir le nom qualifié complet d’Assembly de gestionnaire de protocole utilisé par Azure AD B2C : `Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`.

L’exemple suivant montre un profil technique RESTful :

```xml
<TechnicalProfile Id="REST-UserMembershipValidator">
  <DisplayName>Validate user input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...
```

## <a name="input-claims"></a>Revendications d’entrée

L’élément **InputClaims** contient une liste de revendications à envoyer à l’API REST. Vous pouvez également mapper le nom de votre revendication au nom défini dans l’API REST. L’exemple suivant montre le mappage entre votre stratégie et l’API REST. La revendication **givenName** est envoyée à l’API REST en tant que **firstName**, tandis que la revendication **surname** est envoyée en tant que **lastName**. La revendication **email** est définie telle quelle.

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="email" />
  <InputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="firstName" />
  <InputClaim ClaimTypeReferenceId="surname" PartnerClaimType="lastName" />
</InputClaims>
```

L’élément **InputClaimsTransformations** peut contenir une collection d’éléments **InputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles avant l’envoi à l’API REST.

## <a name="send-a-json-payload"></a>Envoyer une charge utile JSON

Le profil technique de l’API REST vous permet d’envoyer une charge utile JSON complexe à un point de terminaison.

Pour envoyer une charge utile JSON complexe :

1. Créez votre charge utile JSON avec la transformation de revendications [GenerateJson](json-transformations.md).
1. Dans le profil technique API REST :
    1. Ajoutez une transformation de revendications d’entrée avec une référence à la transformation de revendications `GenerateJson`.
    1. Affectez à l’option de métadonnées `SendClaimsIn` la valeur `body`
    1. Donnez à l’option de métadonnées `ClaimUsedForRequestPayload` le nom de la revendication contenant la charge utile JSON.
    1. Dans la revendication d’entrée, ajoutez une référence à la revendication d’entrée contenant la charge utile JSON.

L’exemple suivant `TechnicalProfile` envoie un e-mail de vérification à l’aide d’un service de messagerie tiers (dans ce cas, SendGrid).

```xml
<TechnicalProfile Id="SendGrid">
  <DisplayName>Use SendGrid's email API to send the code the the user</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://api.sendgrid.com/v3/mail/send</Item>
    <Item Key="AuthenticationType">Bearer</Item>
    <Item Key="SendClaimsIn">Body</Item>
    <Item Key="ClaimUsedForRequestPayload">sendGridReqBody</Item>
    <Item Key="DefaultUserMessageIfRequestFailed">Cannot process your request right now, please try again later.</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_SendGridApiKey" />
  </CryptographicKeys>
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="GenerateSendGridRequestBody" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="sendGridReqBody" />
  </InputClaims>
</TechnicalProfile>
```

## <a name="output-claims"></a>Revendications de sortie

L’élément **OutputClaims** contient une liste de revendications retournée par l’API REST. Il se peut que vous deviez mapper le nom de la revendication définie dans votre stratégie au nom défini dans l’API REST. Vous pouvez également inclure des revendications qui ne sont pas retournées par le fournisseur d’identité de l’API REST, pour autant que vous définissiez l’attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

L’exemple suivant montre la revendication retournée par l’API REST :

- Revendication **MembershipId** mappée au nom de la revendication **loyaltyNumber**.

Le profil technique retourne également des revendications, qui ne sont pas retournées par le fournisseur d’identité :

- Revendication **loyaltyNumberIsNew** qui a comme la valeur par défaut `true`.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" PartnerClaimType="MembershipId" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumberIsNew" DefaultValue="true" />
</OutputClaims>
```

## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ServiceUrl | Oui | URL du point de terminaison de l’API REST. |
| AuthenticationType | Oui | Type de l’authentification effectuée par le fournisseur de revendications RESTful. Valeurs possibles : `None`, `Basic`, `Bearer`, `ClientCertificate` ou `ApiKeyHeader`. <br /><ul><li>La valeur `None` indique que l’API REST est anonyme. </li><li>La valeur `Basic` indique que l’API REST est sécurisée avec une authentification HTTP de base. Seuls des utilisateurs vérifiés, notamment Azure AD B2C, peuvent accéder à votre API. </li><li>La valeur `ClientCertificate` (recommandée) indique que l’API REST restreint l’accès à l’aide d’une authentification de certificat client. Seuls des services disposant des certificats appropriés, par exemple Azure AD B2C, peuvent accéder à votre API. </li><li>La valeur `Bearer` indique que l’API REST restreint l’accès à l’aide d’un jeton du porteur OAuth2 du client. </li><li>La valeur `ApiKeyHeader` indique que l’API REST est sécurisée par l’en-tête HTTP de la clé API, par exemple *x-functions-key*. </li></ul> |
| AllowInsecureAuthInProduction| Non| Indique si `AuthenticationType` peut être défini sur `none` dans l’environnement de production (`DeploymentMode` de la stratégie [TrustFrameworkPolicy](trustframeworkpolicy.md) est défini sur `Production` ou n’est pas spécifié). Valeurs possibles : true ou false (par défaut). |
| SendClaimsIn | Non | Spécifie la façon dont les revendications d’entrée sont envoyées au fournisseur de revendications RESTful. Valeurs possibles : `Body` (par défaut), `Form`, `Header`, `Url` ou `QueryString`. La valeur `Body` est la revendication d’entrée envoyée dans le corps de la demande au format JSON. Le valeur `Form` est la revendication d’entrée envoyée dans le corps de la demande, dans un format de valeurs de clé séparées par des perluètes (&). La valeur `Header` est la revendication d’entrée envoyée dans l’en-tête de la demande. La valeur `Url` correspond à la revendication d’entrée qui est envoyée dans l’URL, par exemple, https://{claim1}.example.com/{claim2}/{claim3}?{claim4}={claim5}. La valeur `QueryString` est la revendication d’entrée envoyée dans la chaîne de requête de la demande. Les verbes HTTP appelés par chacune sont les suivants :<br /><ul><li>`Body`: POST</li><li>`Form`: POST</li><li>`Header`: GET</li><li>`Url`: GET</li><li>`QueryString`: GET</li></ul> |
| ClaimsFormat | Non | Non utilisé actuellement, peut être ignoré. |
| ClaimUsedForRequestPayload| Non | Nom d’une revendication de chaîne qui contient la charge utile à envoyer à l’API REST. |
| DebugMode | Non | Exécute le profil technique en mode débogage. Valeurs possibles : `true` ou `false` (par défaut). En mode débogage, l’API REST peut retourner plus d’informations. Consultez la section [Retour de message d’erreur](#returning-validation-error-message). |
| IncludeClaimResolvingInClaimsHandling  | Non | Pour les revendications d’entrée et de sortie, spécifie si la [résolution des revendications](claim-resolver-overview.md) est incluse dans le profil technique. Valeurs possibles : `true` ou `false` (par défaut). Si vous souhaitez utiliser un programme de résolution des revendications dans le profil technique, définissez cette valeur sur `true`. |
| ResolveJsonPathsInJsonTokens  | Non | Indique si le profil technique résout les chemins d’accès JSON. Valeurs possibles : `true` ou `false` (par défaut). Utilisez ces métadonnées pour lire des données issues d’un élément JSON imbriqué. Dans un élément [OutputClaim](technicalprofiles.md#output-claims), définissez `PartnerClaimType` sur l’élément de chemin d’accès JSON que vous souhaitez générer. Par exemple : `firstName.localized` ou `data.0.to.0.email`.|
| UseClaimAsBearerToken| Non| Nom de la revendication qui contient le jeton du porteur.|

## <a name="error-handling"></a>Gestion des erreurs

Les métadonnées suivantes peuvent être utilisées pour configurer les messages d’erreur affichés en cas d’échec de l’API REST. Les messages d’erreur peuvent être [localisés](localization-string-ids.md#restful-service-error-messages).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| DefaultUserMessageIfRequestFailed | Non | Message d’erreur personnalisé par défaut pour toutes les exceptions de l’API REST.|
| UserMessageIfCircuitOpen | Non | Message d’erreur lorsque l’API REST n’est pas accessible. S’il n’est pas spécifié, la valeur DefaultUserMessageIfRequestFailed est retournée. |
| UserMessageIfDnsResolutionFailed | Non | Message d’erreur pour l’exception de résolution DNS. S’il n’est pas spécifié, la valeur DefaultUserMessageIfRequestFailed est retournée. | 
| UserMessageIfRequestTimeout | Non | Message d’erreur lorsque la connexion a expiré. S’il n’est pas spécifié, la valeur DefaultUserMessageIfRequestFailed est retournée. | 

## <a name="cryptographic-keys"></a>Clés de chiffrement

Si le type d’authentification est défini sur `None`, l’élément **CryptographicKeys** n’est pas utilisé.

```xml
<TechnicalProfile Id="REST-API-SignUp">
  <DisplayName>Validate user's input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/signup</Item>
    <Item Key="AuthenticationType">None</Item>
    <Item Key="SendClaimsIn">Body</Item>
  </Metadata>
</TechnicalProfile>
```

Si le type d’authentification est défini sur `Basic`, l’élément **CryptographicKeys** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| BasicAuthenticationUsername | Oui | Nom d’utilisateur utilisé pour l’authentification. |
| BasicAuthenticationPassword | Oui | Mot de passe utilisé pour l’authentification. |

L’exemple suivant montre un profil technique avec une authentification de base :

```xml
<TechnicalProfile Id="REST-API-SignUp">
  <DisplayName>Validate user's input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/signup</Item>
    <Item Key="AuthenticationType">Basic</Item>
    <Item Key="SendClaimsIn">Body</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_B2cRestClientId" />
    <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_B2cRestClientSecret" />
  </CryptographicKeys>
</TechnicalProfile>
```

Si le type d’authentification est défini sur `ClientCertificate`, l’élément **CryptographicKeys** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ClientCertificate | Oui | Certificat X509 (jeu de clés RSA) à utiliser pour l’authentification. |

```xml
<TechnicalProfile Id="REST-API-SignUp">
  <DisplayName>Validate user's input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/signup</Item>
    <Item Key="AuthenticationType">ClientCertificate</Item>
    <Item Key="SendClaimsIn">Body</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="ClientCertificate" StorageReferenceId="B2C_1A_B2cRestClientCertificate" />
  </CryptographicKeys>
</TechnicalProfile>
```

Si le type d’authentification est défini sur `Bearer`, l’élément **CryptographicKeys** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| BearerAuthenticationToken | Non | Jeton du porteur OAuth 2.0. |

```xml
<TechnicalProfile Id="REST-API-SignUp">
  <DisplayName>Validate user's input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/signup</Item>
    <Item Key="AuthenticationType">Bearer</Item>
    <Item Key="SendClaimsIn">Body</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="BearerAuthenticationToken" StorageReferenceId="B2C_1A_B2cRestClientAccessToken" />
  </CryptographicKeys>
</TechnicalProfile>
```

Si le type d’authentification est défini sur `ApiKeyHeader`, l’élément **CryptographicKeys** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Nom de l’en-tête HTTP, tel que `x-functions-key` ou `x-api-key`. | Oui | Clé utilisée pour l’authentification. |

```xml
<TechnicalProfile Id="REST-API-SignUp">
  <DisplayName>Validate user's input data and return loyaltyNumber claim</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/signup</Item>
    <Item Key="AuthenticationType">ApiKeyHeader</Item>
    <Item Key="SendClaimsIn">Body</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="x-functions-key" StorageReferenceId="B2C_1A_RestApiKey" />
  </CryptographicKeys>
</TechnicalProfile>
```

## <a name="returning-validation-error-message"></a>Renvoi du message d'erreur de validation

Il se peut que votre API REST doive retourner un message d’erreur tel que « Utilisateur introuvable dans le système CRM ». Lorsqu’une erreur se produit, l’API REST doit retourner un message d’erreur HTTP 4xx, par exemple le code d’état de réponse 400 (requête incorrecte) ou 409 (conflit). Le corps de la réponse contient le message d’erreur au format JSON :

```json
{
  "version": "1.0.0",
  "status": 409,
  "code": "API12345",
  "requestId": "50f0bd91-2ff4-4b8f-828f-00f170519ddb",
  "userMessage": "Message for the user",
  "developerMessage": "Verbose description of problem and how to fix it.",
  "moreInfo": "https://restapi/error/API12345/moreinfo"
}
```

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| version | Oui | Votre version d’API REST. Par exemple : 1.0.1 |
| status | Oui | Doit être 409 |
| code | Non | Code d’erreur provenant du fournisseur de point de terminaison RESTful, affiché quand `DebugMode` est activé. |
| requestId | Non | Identificateur de demande provenant du fournisseur de point de terminaison RESTful, affiché quand `DebugMode` est activé. |
| userMessage | Oui | Message d’erreur affiché à l’utilisateur. |
| developerMessage | Non | Description détaillée du problème et de la manière de le corriger, affiché quand `DebugMode` est activé. |
| moreInfo | Non | URI pointant vers des informations supplémentaires, affiché quand `DebugMode` est activé. |


L’exemple suivant montre une classe C# qui retourne un message d’erreur :

```csharp
public class ResponseContent
{
  public string version { get; set; }
  public int status { get; set; }
  public string code { get; set; }
  public string userMessage { get; set; }
  public string developerMessage { get; set; }
  public string requestId { get; set; }
  public string moreInfo { get; set; }
}
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour obtenir des exemples d’utilisation d’un profil technique RESTful :

- [Intégrer les échanges de revendications de l’API REST dans votre stratégie personnalisée Azure AD B2C](custom-policy-rest-api-intro.md)
- [Procédure pas à pas : Intégrer les échanges de revendications de l’API REST dans votre parcours utilisateur Azure AD B2C comme validation d’une entrée de l’utilisateur](custom-policy-rest-api-claims-validation.md)
- [Procédure pas à pas : Ajouter des échanges de revendications de l’API REST aux stratégies personnalisées dans Azure Active Directory B2C](custom-policy-rest-api-claims-validation.md)
- [Sécuriser vos services d’API REST](secure-rest-api.md)

