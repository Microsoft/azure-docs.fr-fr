---
title: Profils techniques Azure AD SSPR dans les stratégies personnalisées
titleSuffix: Azure AD B2C
description: Informations de référence sur les stratégies personnalisées pour les profils techniques Azure AD SSPR dans Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/23/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 3e6fcf956639d827a8654c5ee80e7cab8cadf930
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85383595"
---
# <a name="define-an-azure-ad-sspr-technical-profile-in-an-azure-ad-b2c-custom-policy"></a>Définir un profil technique Azure AD SSPR dans une stratégie personnalisée Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure AD B2C (Azure Active Directory B2C) prend en charge la vérification d’une adresse e-mail dans le cadre de la réinitialisation de mot de passe en libre-service (SSPR). Utilisez le profil technique Azure AD SSPR pour générer et envoyer un code à une adresse e-mail, puis vérifier le code. Le profil technique Azure AD SSPR peut également retourner un message d’erreur. Le profil technique de validation valide les données fournies par l’utilisateur avant la poursuite du parcours de celui-ci. Avec le profil technique de validation, un message d’erreur apparaît sur une page autodéclarée.

Ce profil technique :

- Ne fournit pas d’interface permettant d’interagir avec l’utilisateur. Au lieu de cela, l’interface utilisateur est appelée à partir d’un profil technique [autodéclaré](self-asserted-technical-profile.md) ou d’un [contrôle d’affichage](display-controls.md) en tant que [profil technique de validation](validation-technical-profile.md).
- Utilise le service Azure AD SSPR pour générer et envoyer un code à une adresse e-mail, puis vérifie le code.  
- Valide une adresse e-mail au moyen d’un code de vérification.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `Proprietary`. L’attribut **handler** doit contenir le nom complet de l’assembly de gestionnaire de protocole utilisé par Azure AD B2C :

```
Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```

L’exemple suivant montre un profil technique Azure AD SSPR :

```XML
<TechnicalProfile Id="AadSspr-SendCode">
  <DisplayName>Send Code</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    ...
```

## <a name="send-email"></a>Envoi de courrier électronique

Le premier mode de ce profil technique consiste à générer un code et à l’envoyer. Les options suivantes peuvent être configurées pour ce mode.

### <a name="input-claims"></a>Revendications d’entrée

L’élément **InputClaims** contient la liste des revendications à envoyer à Azure AD SSPR. Vous pouvez également faire correspondre le nom de votre revendication au nom défini dans le profil technique SSPR.

| ClaimReferenceId | Obligatoire | Description |
| --------- | -------- | ----------- |
| emailAddress | Oui | Identificateur de l’utilisateur propriétaire de l’adresse e-mail. La propriété `PartnerClaimType` de la revendication d’entrée doit être définie sur `emailAddress`. |


L’élément **InputClaimsTransformations** peut contenir une collection d’éléments **InputClaimsTransformation** utilisés pour modifier les revendications d’entrée ou en générer de nouvelles avant l’envoi au service Azure AD SSPR.

### <a name="output-claims"></a>Revendications de sortie

Comme le fournisseur de protocole Azure AD SSPR ne retourne pas d’élément **OutputClaims**, il n’est pas nécessaire de spécifier des revendications de sortie. Vous pouvez toutefois inclure des revendications non retournées par le fournisseur d’identité Azure AD SSPR, à condition de définir l’attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

### <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Opération | Oui | Doit être **SendCode**.  |

#### <a name="ui-elements"></a>Éléments d’interface utilisateur

Les métadonnées suivantes peuvent être utilisées pour configurer les messages d’erreur affichés en cas d’échec de l’envoi de SMS. Les métadonnées doivent être configurées dans le profil technique [autodéclaré](self-asserted-technical-profile.md). Les messages d’erreur peuvent être [localisés](localization-string-ids.md#azure-ad-sspr).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| UserMessageIfInternalError | Non | Message d’erreur utilisateur si le serveur a rencontré une erreur interne. |
| UserMessageIfThrottled| Non | Message d’erreur utilisateur si une requête a été limitée.|


### <a name="example-send-an-email"></a>Exemple : envoyer un e-mail

L’exemple suivant montre l’utilisation d’un profil technique Azure AD SSPR pour envoyer un code par e-mail.

```XML
<TechnicalProfile Id="AadSspr-SendCode">
  <DisplayName>Send Code</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="Operation">SendCode</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress"/>
  </InputClaims>
</TechnicalProfile>
```

## <a name="verify-code"></a>Vérifier le code

Le deuxième mode de ce profil technique consiste à vérifier un code. Les options suivantes peuvent être configurées pour ce mode.

### <a name="input-claims"></a>Revendications d’entrée

L’élément **InputClaims** contient la liste des revendications à envoyer à Azure AD SSPR. Vous pouvez également faire correspondre le nom de votre revendication au nom défini dans le profil technique SSPR.

| ClaimReferenceId | Obligatoire | Description |
| --------- | -------- | ----------- | ----------- |
| emailAddress| Oui | Même adresse e-mail que celle ayant servi à envoyer un code. Il est également utilisé pour rechercher une session de vérification par e-mail. La propriété `PartnerClaimType` de la revendication d’entrée doit être définie sur `emailAddress`.|
| verificationCode  | Oui | Code de vérification fourni par l’utilisateur à vérifier. La propriété `PartnerClaimType` de la revendication d’entrée doit être définie sur `verificationCode`. |

L’élément **InputClaimsTransformations** peut contenir une collection d’éléments **InputClaimsTransformation** utilisés pour modifier les revendications d’entrée ou en générer de nouvelles avant l’appel du service Azure AD SSPR.

### <a name="output-claims"></a>Revendications de sortie

Comme le fournisseur de protocole Azure AD SSPR ne retourne pas d’élément **OutputClaims**, il n’est pas nécessaire de spécifier des revendications de sortie. Vous pouvez toutefois inclure des revendications non retournées par le fournisseur d’identité Azure AD SSPR, à condition de définir l’attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

### <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Opération | Oui | Doit être **VerifyCode** |

#### <a name="ui-elements"></a>Éléments d’interface utilisateur

Les métadonnées suivantes peuvent être utilisées pour configurer les messages d’erreur affichés en cas d’échec de la vérification du code. Les métadonnées doivent être configurées dans le profil technique [autodéclaré](self-asserted-technical-profile.md). Les messages d’erreur peuvent être [localisés](localization-string-ids.md#azure-ad-sspr).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
|UserMessageIfChallengeExpired | Message à afficher à l'utilisateur si la session de vérification du code a expiré. Soit le code a expiré, soit le code n’a jamais été généré pour un identificateur donné.|
|UserMessageIfInternalError | Message d’erreur utilisateur si le serveur a rencontré une erreur interne.|
|UserMessageIfThrottled | Message d’erreur utilisateur si une requête a été limitée.|
|UserMessageIfVerificationFailedNoRetry | Message à afficher à l’utilisateur s’il a fourni un code non valide et que l’utilisateur n’est pas autorisé à fournir le code correct.|
|UserMessageIfVerificationFailedRetryAllowed | Message à afficher à l’utilisateur s’il a fourni un code non valide et que l’utilisateur est autorisé à fournir le code correct.|

### <a name="example-verify-a-code"></a>Exemple : vérifier un code

L’exemple suivant montre l’utilisation d’un profil technique Azure AD SSPR pour vérifier le code.

```XML
<TechnicalProfile Id="AadSspr-VerifyCode">
  <DisplayName>Verify Code</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="Operation">VerifyCode</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="verificationCode" PartnerClaimType="verificationCode" />
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress"/>
  </InputClaims>
</TechnicalProfile>
```