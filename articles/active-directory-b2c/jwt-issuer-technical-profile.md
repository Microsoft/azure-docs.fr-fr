---
title: Définir un profil technique pour un émetteur JWT dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définissez un profil technique pour un émetteur JSON Web Token (JWT) dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/12/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: b3ad9c5d19d5d24154a8a63bfc412d6bbfdc1d8b
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91949222"
---
# <a name="define-a-technical-profile-for-a-jwt-token-issuer-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique pour un émetteur de jeton JSON Web Token dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) émet plusieurs types de jetons de sécurité lors du traitement de chaque flux d’authentification. Un profil technique pour un émetteur de jeton JSON Web Token émet un jeton JSON Web Token qui est retourné à l’application par partie de confiance. Ce profil technique est généralement la dernière étape d’orchestration dans le parcours utilisateur.

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `OpenIdConnect`. Définissez l’élément **OutputTokenFormat** sur `JWT`.

L’exemple suivant montre un profil technique pour `JwtIssuer` :

```xml
<TechnicalProfile Id="JwtIssuer">
  <DisplayName>JWT Issuer</DisplayName>
  <Protocol Name="None" />
  <OutputTokenFormat>JWT</OutputTokenFormat>
  <Metadata>
    <Item Key="client_id">{service:te}</Item>
    <Item Key="issuer_refresh_token_user_identity_claim_type">objectId</Item>
    <Item Key="SendTokenResponseBodyWithJsonNumbers">true</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
    <Key Id="issuer_refresh_token_key" StorageReferenceId="B2C_1A_TokenEncryptionKeyContainer" />
  </CryptographicKeys>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-jwt-issuer" />
</TechnicalProfile>
```

## <a name="input-output-and-persist-claims"></a>Revendications d’entrée, de sortie et de conservation

Les éléments **InputClaims**, **OutputClaims** et **PersistClaims** sont vides ou absents. Les éléments **InutputClaimsTransformations** et **OutputClaimsTransformations** sont également absents.

## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| issuer_refresh_token_user_identity_claim_type | Oui | Revendication à utiliser comme revendication d’identité d’utilisateur dans les codes d’autorisation OAuth2 et jetons d’actualisation. Par défaut, vous devriez la définir sur `objectId`, sauf si vous spécifiez un autre type de revendication SubjectNamingInfo. |
| SendTokenResponseBodyWithJsonNumbers | Non  | Toujours défini sur `true`. Pour un format hérité où des valeurs numériques sont fournies sous forme de chaînes au lieu de nombres JSON, défini sur `false`. Cet attribut est nécessaire pour les clients qui ont pris une dépendance sur une implémentation antérieure qui a retourné des propriétés telles que des chaînes. |
| token_lifetime_secs | Non  | Durées de vie des jetons d’accès. Durée de vie du jeton du porteur OAuth 2.0 utilisé pour accéder à une ressource protégée. La valeur par défaut est 3 600 secondes (1 heure). La valeur minimale (inclusive) est 300 secondes (5 minutes). La valeur maximale (inclusive) est 86 400 secondes (24 heures). |
| id_token_lifetime_secs | Non  | Durées de vie des jetons d’ID. La valeur par défaut est 3 600 secondes (1 heure). La valeur minimale (inclusive) est 300 secondes (5 minutes). La valeur maximale (inclusive) est 86 400 secondes (24 heures). |
| refresh_token_lifetime_secs | Non  | Durées de vie des jetons d’actualisation. Période maximale avant laquelle un jeton d’actualisation peut être utilisé pour acquérir un nouveau jeton d’accès si l’étendue offline_access a été accordée à votre application. La valeur par défaut est 1 209 600 secondes (14 jours). La valeur minimale (inclusive) est 86 400 secondes (24 heures). La valeur maximale (inclusive) est 7 776 000 secondes (90 jours). |
| rolling_refresh_token_lifetime_secs | Non  | Durée de vie de la fenêtre glissante du jeton d'actualisation. Une fois cette période écoulée, l’utilisateur est obligé de s’authentifier de nouveau, quelle que soit la période de validité du dernier jeton d’actualisation acquis par l’application. Si vous ne souhaitez pas appliquer une durée de vie de fenêtre glissante, définissez la valeur d’allow_infinite_rolling_refresh_token sur `true`. La valeur par défaut est 7 776 000 secondes (90 jours). La valeur minimale (inclusive) est 86 400 secondes (24 heures). La valeur maximale (inclusive) est 31 536 000 secondes (365 jours). |
| allow_infinite_rolling_refresh_token | Non  | Si la valeur est `true`, la durée de vie de la fenêtre glissante jeton d’actualisation n’expire jamais. |
| IssuanceClaimPattern | Non  | Contrôle la revendication d’émetteur (iss). Une des valeurs suivantes :<ul><li>AuthorityAndTenantGuid : La revendication iss inclut votre nom de domaine, tel que `login.microsoftonline` ou `tenant-name.b2clogin.com`, et votre identificateur de locataire https:\//login.microsoftonline.com/00000000-0000-0000-0000-000000000000/v2.0/</li><li>AuthorityWithTfp : la revendication iss inclut votre nom de domaine, tel que `login.microsoftonline` ou `tenant-name.b2clogin.com`, votre identificateur de locataire et votre nom de stratégie de partie de confiance. https:\//login.microsoftonline.com/tfp/00000000-0000-0000-0000-000000000000/b2c_1a_tp_sign-up-or-sign-in/v2.0/</li></ul> Valeur par défaut : AuthorityAndTenantGuid |
| AuthenticationContextReferenceClaimPattern | Non  | Contrôle la valeur de revendication `acr`.<ul><li>None : Azure AD B2C ne génère pas la revendication acr</li><li>PolicyId : la revendication `acr` contient le nom de la stratégie</li></ul>Les options permettant de définir cette valeur sont TFP (Trust Framework Policy) et ACR (Authentication Context Reference). Il est recommandé de définir cette valeur sur TFP. Pour définir la valeur, assurez-vous que `<Item>` avec `Key="AuthenticationContextReferenceClaimPattern"` existe et que la valeur est `None`. Dans votre stratégie de partie de confiance, ajoutez `<OutputClaims>` item et ajoutez cet élément `<OutputClaim ClaimTypeReferenceId="trustFrameworkPolicy" Required="true" DefaultValue="{policy}" />`. Vérifiez également que votre stratégie contient le type de revendication `<ClaimType Id="trustFrameworkPolicy">   <DisplayName>trustFrameworkPolicy</DisplayName>     <DataType>string</DataType> </ClaimType>` |
|RefreshTokenUserJourneyId| Non  | Identificateur d’un parcours utilisateur qui doit être exécuté pendant la requête POST d’[actualisation d’un jeton d’accès](authorization-code-flow.md#4-refresh-the-token) sur le point de terminaison `/token`. |

## <a name="cryptographic-keys"></a>Clés de chiffrement

L’élément CryptographicKeys contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| issuer_secret | Oui | Certificat X509 (jeu de clés RSA) à utiliser pour signer le jeton JSON Web Token. Il s’agit de la clé `B2C_1A_TokenSigningKeyContainer` que vous avez configurée à l’étape [Bien démarrer avec les stratégies personnalisées](custom-policy-get-started.md). |
| issuer_refresh_token_key | Oui | Certificat X509 (jeu de clés RSA) à utiliser pour déchiffrer le jeton d'actualisation. Vous avez configuré la clé `B2C_1A_TokenEncryptionKeyContainer` à l’étape [Bien démarrer avec les stratégies personnalisées](custom-policy-get-started.md) |

## <a name="session-management"></a>Gestion des sessions

Pour configurer les sessions Azure AD B2C entre Azure AD B2C et une application par partie de confiance, dans l’attribut de l’élément `UseTechnicalProfileForSessionManagement`, ajoutez une référence à une session SSO [OAuthSSOSessionProvider](custom-policy-reference-sso.md#oauthssosessionprovider).














