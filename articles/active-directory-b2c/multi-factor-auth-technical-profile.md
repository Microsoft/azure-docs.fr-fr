---
title: Profils techniques Azure AD MFA dans les stratégies personnalisées
titleSuffix: Azure AD B2C
description: Informations de référence sur les stratégies personnalisées pour les profils techniques Azure AD Multi-Factor Authentication (MFA) dans Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/26/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: e81ac35555e6653cecb602e5af2f19aa3e2f05e9
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94840591"
---
# <a name="define-an-azure-ad-mfa-technical-profile-in-an-azure-ad-b2c-custom-policy"></a>Définir un profil technique Azure AD MFA dans une stratégie personnalisée Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) permet de vérifier un numéro de téléphone à l'aide d'Azure AD Multi-Factor Authentication (MFA). Utilisez ce profil technique pour générer et envoyer un code à un numéro de téléphone, puis vérifier le code. Le profil technique Azure AD MFA peut également renvoyer un message d'erreur.  Le profil technique de validation valide les données fournies par l’utilisateur avant la poursuite du parcours de celui-ci. Avec le profil technique de validation, un message d’erreur apparaît sur une page autodéclarée.

Ce profil technique :

- Ne fournit pas d’interface permettant d’interagir avec l’utilisateur. Au lieu de cela, l’interface utilisateur est appelée à partir d’un profil technique [autodéclaré](self-asserted-technical-profile.md) ou d’un [contrôle d’affichage](display-controls.md) en tant que [profil technique de validation](validation-technical-profile.md).
- Utilise le service Azure AD MFA pour générer et envoyer un code à un numéro de téléphone, puis vérifie le code.  
- Valide un numéro de téléphone via des SMS.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `Proprietary`. L’attribut **handler** doit contenir le nom complet de l’assembly de gestionnaire de protocole utilisé par Azure AD B2C :

```
Web.TPEngine.Providers.AzureMfaProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```

L'exemple suivant présente un profil technique Azure AD MFA :

```xml
<TechnicalProfile Id="AzureMfa-SendSms">
    <DisplayName>Send Sms</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureMfaProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    ...
```

## <a name="send-sms"></a>Envoyer un SMS

Le premier mode de ce profil technique consiste à générer un code et à l’envoyer. Les options suivantes peuvent être configurées pour ce mode.

### <a name="input-claims"></a>Revendications d’entrée

L'élément **InputClaims** contient la liste des revendications à envoyer à Azure AD MFA. Vous pouvez également faire correspondre le nom de votre revendication au nom défini dans le profil technique MFA.

| ClaimReferenceId | Obligatoire | Description |
| --------- | -------- | ----------- |
| userPrincipalName | Oui | Identificateur de l’utilisateur propriétaire du numéro de téléphone. |
| phoneNumber | Oui | Numéro de téléphone auquel le code SMS sera envoyé. |
| companyName | Non |Nom de la société dans le SMS. S’il n’est pas indiqué, c’est le nom de votre application qui est utilisé. |
| locale | Non | Paramètres régionaux du SMS. S’ils ne sont pas indiqués, ce sont les paramètres régionaux du navigateur de l’utilisateur qui sont utilisés. |

L'élément **InputClaimsTransformations** peut contenir une collection d'éléments **InputClaimsTransformation** utilisés pour modifier les revendications d'entrée ou en générer de nouvelles avant l'envoi au service Azure AD MFA.

### <a name="output-claims"></a>Revendications de sortie

Comme le fournisseur de protocole Azure AD MFA ne renvoie pas d'élément **OutputClaims**, il n'est pas nécessaire de spécifier des revendications de sortie. Vous pouvez toutefois inclure des revendications qui ne sont pas renvoyées par le fournisseur d'identité Azure AD MFA, à condition de définir l'attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

### <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Opération | Oui | Doit être **OneWaySMS**.  |

#### <a name="ui-elements"></a>Éléments d’interface utilisateur

Les métadonnées suivantes peuvent être utilisées pour configurer les messages d’erreur affichés en cas d’échec de l’envoi de SMS. Les métadonnées doivent être configurées dans le profil technique [autodéclaré](self-asserted-technical-profile.md). Les messages d’erreur peuvent être [localisés](localization-string-ids.md#azure-ad-mfa-error-messages).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| UserMessageIfCouldntSendSms | Non | Message d’erreur utilisateur si le numéro de téléphone fourni n’accepte pas les SMS. |
| UserMessageIfInvalidFormat | Non | Message d’erreur utilisateur si le numéro de téléphone fourni n’est pas un numéro de téléphone valide. |
| UserMessageIfServerError | Non | Message d’erreur utilisateur si le serveur a rencontré une erreur interne. |
| UserMessageIfThrottled| Non | Message d’erreur utilisateur si une requête a été limitée.|

### <a name="example-send-an-sms"></a>Exemple : envoyer un SMS

L'exemple suivant présente un profil technique Azure AD MFA utilisé pour envoyer un code par SMS.

```xml
<TechnicalProfile Id="AzureMfa-SendSms">
  <DisplayName>Send Sms</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureMfaProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="Operation">OneWaySMS</Item>
  </Metadata>
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="CombinePhoneAndCountryCode" />
    <InputClaimsTransformation ReferenceId="ConvertStringToPhoneNumber" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userPrincipalName" />
    <InputClaim ClaimTypeReferenceId="fullPhoneNumber" PartnerClaimType="phoneNumber" />
  </InputClaims>
</TechnicalProfile>
```

## <a name="verify-code"></a>Vérifier le code

Le deuxième mode de ce profil technique consiste à vérifier un code. Les options suivantes peuvent être configurées pour ce mode.

### <a name="input-claims"></a>Revendications d’entrée

L'élément **InputClaims** contient la liste des revendications à envoyer à Azure AD MFA. Vous pouvez également faire correspondre le nom de votre revendication au nom défini dans le profil technique MFA.

| ClaimReferenceId | Obligatoire | Description |
| --------- | -------- | ----------- | ----------- |
| phoneNumber| Oui | Même numéro de téléphone qui a servi à envoyer un code. Il est également utilisé pour rechercher une session de vérification par téléphone. |
| verificationCode  | Oui | Code de vérification fourni par l’utilisateur à vérifier. |

L'élément **InputClaimsTransformations** peut contenir une collection d'éléments **InputClaimsTransformation** utilisés pour modifier les revendications d'entrée ou en générer de nouvelles avant l'appel du service Azure AD MFA.

### <a name="output-claims"></a>Revendications de sortie

Comme le fournisseur de protocole Azure AD MFA ne renvoie pas d'élément **OutputClaims**, il n'est pas nécessaire de spécifier des revendications de sortie. Vous pouvez toutefois inclure des revendications qui ne sont pas renvoyées par le fournisseur d'identité Azure AD MFA, à condition de définir l'attribut `DefaultValue`.

L’élément **OutputClaimsTransformations** peut contenir une collection d’éléments **OutputClaimsTransformation** qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

### <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Opération | Oui | Doit être **Verify**. |

#### <a name="ui-elements"></a>Éléments d’interface utilisateur

Les métadonnées suivantes peuvent être utilisées pour configurer les messages d’erreur affichés en cas d’échec de la vérification du code. Les métadonnées doivent être configurées dans le profil technique [autodéclaré](self-asserted-technical-profile.md). Les messages d’erreur peuvent être [localisés](localization-string-ids.md#azure-ad-mfa-error-messages).

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| UserMessageIfMaxAllowedCodeRetryReached| Non | Message d’erreur utilisateur si l’utilisateur a effectué trop de tentatives de code de vérification. |
| UserMessageIfServerError | Non | Message d’erreur utilisateur si le serveur a rencontré une erreur interne. |
| UserMessageIfThrottled| Non | Message d’erreur utilisateur si la requête est limitée.|
| UserMessageIfWrongCodeEntered| Non| Message d’erreur utilisateur si le code entré pour la vérification est incorrect.|

### <a name="example-verify-a-code"></a>Exemple : vérifier un code

L'exemple suivant présente un profil technique Azure AD MFA utilisé pour vérifier le code.

```xml
<TechnicalProfile Id="AzureMfa-VerifySms">
    <DisplayName>Verify Sms</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureMfaProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    <Metadata>
        <Item Key="Operation">Verify</Item>
    </Metadata>
    <InputClaims>
        <InputClaim ClaimTypeReferenceId="phoneNumber" PartnerClaimType="phoneNumber" />
        <InputClaim ClaimTypeReferenceId="verificationCode" />
    </InputClaims>
</TechnicalProfile>
```
