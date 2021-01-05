---
title: Informations de référence sur le contrôle d’affichage
titleSuffix: Azure AD B2C
description: Informations de référence pour les contrôles d’affichage Azure AD B2C. Utilisez des contrôles d’affichage pour personnaliser les parcours utilisateur définis dans vos stratégies personnalisées.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 441a77823c77305e567e9e1436715bc51ca48c11
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387052"
---
# <a name="display-controls"></a>Contrôles d’affichage

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Un **contrôle d’affichage** est un élément d’interface utilisateur qui présente une fonctionnalité particulière et interagit avec le service principal Azure Active Directory B2C (Azure AD B2C). Il permet à l’utilisateur d’exécuter des actions sur la page qui appellent un [profil technique de validation](validation-technical-profile.md) dans le back end. Les contrôles d’affichage sont affichés sur la page et sont référencés par un [profil technique auto-déclaré](self-asserted-technical-profile.md).

L’image suivante illustre une page d’inscription auto-déclarée avec deux contrôles d’affichage qui valident une adresse e-mail principale et une adresse e-mail secondaire.

![Exemple de contrôle d’affichage rendu](media/display-controls/display-control-email.png)

## <a name="prerequisites"></a>Conditions préalables requises

 Dans la section [Métadonnées](self-asserted-technical-profile.md#metadata) d’un [profil technique auto-déclaré](self-asserted-technical-profile.md), dans la propriété [ContentDefinition](contentdefinitions.md) référencée, `DataUri` doit être défini sur le contrat de pages version 2.0.0 ou ultérieure. Par exemple :

```xml
<ContentDefinition Id="api.selfasserted">
  <LoadUri>~/tenant/default/selfAsserted.cshtml</LoadUri>
  <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
  <DataUri>urn:com:microsoft:aad:b2c:elements:selfasserted:2.0.0</DataUri>
  ...
```

## <a name="defining-display-controls"></a>Définition de contrôles d’affichage

L’élément **DisplayControl** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur utilisé pour le contrôle d’affichage. Il peut être [référencé](#referencing-display-controls). |
| UserInterfaceControlType | Oui | Type du contrôle d’affichage. [VerificationControl](display-control-verification.md) est actuellement pris en charge |

L’élément **DisplayControl** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| InputClaims | 0:1 | Les **InputClaims** sont utilisés pour préremplir la valeur des revendications à collecter auprès de l’utilisateur. Pour plus d’informations, consultez l’élément [InputClaims](technicalprofiles.md#input-claims). |
| DisplayClaims | 0:1 | Les **DisplayClaims** sont utilisés pour représenter les revendications à collecter auprès de l’utilisateur. Pour plus d’informations, consultez l’élément [DisplayClaim](technicalprofiles.md#displayclaim).|
| OutputClaims | 0:1 | Les **OutputClaims** sont utilisés pour représenter les revendications à enregistrer temporairement pour ce **DisplayControl**. Pour plus d’informations, consultez l’élément [OutputClaims](technicalprofiles.md#output-claims).|
| Actions | 0:1 | Les **Actions** sont utilisées pour répertorier les profils techniques de validation à appeler pour les actions de l’utilisateur qui se produisent au niveau du serveur frontal. |

### <a name="input-claims"></a>Revendications d’entrée

Dans un contrôle d’affichage, vous pouvez utiliser des éléments **InputClaims** pour préremplir la valeur des revendications à collecter auprès de l’utilisateur sur la page. Des **InputClaimsTransformations** peuvent être définis dans le profil technique auto-déclaré qui fait référence à ce contrôle d’affichage.

L’exemple suivant préremplit l’adresse e-mail à vérifier avec l’adresse déjà présente.

```xml
<DisplayControl Id="emailControl" UserInterfaceControlType="VerificationControl">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="emailAddress" />
  </InputClaims>
  ...
```

### <a name="display-claims"></a>Revendications d’affichage

Chaque type de contrôle d’affichage nécessite un ensemble différent de revendications d’affichage, des [revendications de sortie](#output-claims) et des [actions](#display-control-actions) à effectuer.

Comme les **revendications d’affichage** définies dans un [profil technique auto-déclaré](self-asserted-technical-profile.md#display-claims), les revendications d’affichage représentent les revendications à collecter auprès de l’utilisateur dans le contrôle d’affichage. L’élément **ClaimType** référencé doit définir l’élément **UserInputType** sur n’importe quel type d’entrée d’utilisateur pris en charge par Azure AD B2C, tel que `TextBox` ou `DropdownSingleSelect`. Si une valeur de revendication d’affichage est requise par une **action**, définissez l’attribut **Requis** sur `true` pour forcer l’utilisateur à fournir une valeur pour cette revendication d’affichage spécifique.

Certaines revendications d’affichage sont requises pour certains types de contrôle d’affichage. Par exemple, **VerificationCode** est requis pour le contrôle d’affichage de type **VerificationControl**. Utilisez l’attribut **ControlClaimType** pour spécifier le DisplayClaim désigné pour la revendication requise. Par exemple :

```xml
<DisplayClaim ClaimTypeReferenceId="otpCode" ControlClaimType="VerificationCode" Required="true" />
```

### <a name="output-claims"></a>Revendications de sortie

Les **revendications de sortie** d’un contrôle d’affichage ne passent pas à l’étape d’orchestration suivante. Elles sont enregistrées temporairement et uniquement pour la session de contrôle d’affichage en cours. Ces revendications temporaires peuvent être partagées entre les différentes actions d’un même contrôle d’affichage.

Pour propager la sortie des revendications à l'étape d'orchestration suivante, utilisez les **OutputClaims** du profil technique auto-déclaré qui fait référence à ce contrôle d'affichage.

### <a name="display-control-actions"></a>Actions de contrôle d’affichage

Les **Actions** d’un contrôle d’affichage sont des procédures qui se produisent dans le back end Azure AD B2C lorsqu’un utilisateur effectue une action donnée côté client (navigateur). Par exemple, les validations à effectuer lorsque l’utilisateur sélectionne un bouton sur la page.

Une action définit une liste de **profils techniques de validation**. Ceux-ci sont utilisés pour valider tout ou partie des revendications d’affichage du contrôle d’affichage. Le profil technique de validation valide l’entrée d’utilisateur et peut retourner une erreur à l’utilisateur. Vous pouvez utiliser **ContinueOnError**, **ContinueOnSuccess** et **Preconditions** dans l’action de contrôle d’affichage de la même manière que dans les [profils techniques de validation](validation-technical-profile.md) d’un profil technique auto-déclaré.

#### <a name="actions"></a>Actions

L’élément **Actions** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Action | 1:n | Liste des actions à exécuter. |

#### <a name="action"></a>Action

L’élément **Action** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Type d’opération. Valeurs possibles : `SendCode` ou `VerifyCode`. La valeur `SendCode` envoie un code à l’utilisateur. Cette action peut contenir deux profils techniques de validation : un pour générer un code, et un autre pour l’envoyer. La valeur `VerifyCode` vérifie le code que l’utilisateur a tapé dans la zone de texte d’entrée. |

L’élément **Action** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| ValidationClaimsExchange | 1:1 | Identificateur des profils techniques utilisés pour valider la totalité ou certaines des revendications d’affichage du profil technique de référencement. Toutes les revendications d’entrée du profil technique référencé doivent apparaître dans les revendications d’affichage du profil technique de référencement. |

#### <a name="validationclaimsexchange"></a>ValidationClaimsExchange

L’élément **ValidationClaimsExchange** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | Un profil technique pour valider toutes ou certaines des revendications d’affichage du profil technique de référencement. |

L’élément **ValidationTechnicalProfile** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | L’identificateur d’un profil technique déjà défini dans la stratégie ou dans la stratégie parente. |
|ContinueOnError|Non| Indique si la validation des profils techniques de validation suivants doit se poursuivre si ce profil technique de validation génère une erreur. Valeurs possibles : `true` ou `false` (par défaut, arrête le traitement des profils de validation suivants et renvoie une erreur). |
|ContinueOnSuccess | Non | Indique si la validation des profils de validation suivants doit se poursuivre si ce profil technique de validation réussit. Valeurs possibles : `true` ou `false`. La valeur par défaut est `true`, ce qui signifie que le traitement des profils de validation suivants continue. |

L’élément **ValidationTechnicalProfile** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Preconditions | 0:1 | Une liste de conditions préalables qui doivent être satisfaites pour que le profil technique de validation puisse s’exécuter. |

L’élément **Precondition** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| `Type` | Oui | Le type de vérification ou de requête à exécuter pour la condition préalable. Valeurs possibles : `ClaimsExist` ou `ClaimEquals`. `ClaimsExist` spécifie que les actions doivent être exécutées si les revendications spécifiées existent dans le jeu de revendications actuel de l’utilisateur. `ClaimEquals` spécifie que les actions doivent être exécutées si la revendication spécifiée existe et que sa valeur est égale à la valeur spécifiée. |
| `ExecuteActionsIf` | Oui | Indique si les actions de la condition préalable doivent être effectuées si le test est true ou false. |

L’élément **Precondition** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Valeur | 1:n | Les données qui sont utilisées par la vérification. Si le type de cette vérification est `ClaimsExist`, ce champ spécifie un ClaimTypeReferenceId à rechercher. Si le type de la vérification est `ClaimEquals`, ce champ spécifie un ClaimTypeReferenceId à rechercher. Spécifie la valeur à vérifier dans un autre élément de valeur.|
| Action | 1:1 | L’action à entreprendre si la vérification de condition préalable dans une étape d’orchestration a la valeur true. La valeur de l’**Action** est définie sur `SkipThisValidationTechnicalProfile`, ce qui indique que le profil technique de validation associé ne doit pas être exécuté. |

L’exemple suivant envoie et vérifie l’adresse de messagerie à l’aide du [profil technique Azure AD SSPR](aad-sspr-technical-profile.md).

```xml
<DisplayControl Id="emailVerificationControl" UserInterfaceControlType="VerificationControl">
  <InputClaims></InputClaims>
  <DisplayClaims>
    <DisplayClaim ClaimTypeReferenceId="email" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="verificationCode" ControlClaimType="VerificationCode" Required="true" />
  </DisplayClaims>
  <OutputClaims></OutputClaims>
  <Actions>
    <Action Id="SendCode">
      <ValidationClaimsExchange>
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AadSspr-SendCode" />
      </ValidationClaimsExchange>
    </Action>
    <Action Id="VerifyCode">
      <ValidationClaimsExchange>
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AadSspr-VerifyCode" />
      </ValidationClaimsExchange>
    </Action>
  </Actions>
</DisplayControl>
```

L’exemple suivant envoie un code dans un courrier électronique ou un SMS en fonction de la sélection de l’utilisateur de la revendication **mfaType** avec des conditions préalables.

```xml
<Action Id="SendCode">
  <ValidationClaimsExchange>
    <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AzureMfa-SendSms">
      <Preconditions>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
          <Value>mfaType</Value>
          <Value>email</Value>
          <Action>SkipThisValidationTechnicalProfile</Action>
        </Precondition>
      </Preconditions>
    </ValidationClaimsExchangeTechnicalProfile>
    <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AadSspr-SendEmail">
      <Preconditions>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
          <Value>mfaType</Value>
          <Value>phone</Value>
          <Action>SkipThisValidationTechnicalProfile</Action>
        </Precondition>
      </Preconditions>
    </ValidationClaimsExchangeTechnicalProfile>
  </ValidationClaimsExchange>
</Action>
```

## <a name="referencing-display-controls"></a>Référencement de contrôles d’affichage

Les contrôles d’affichage sont référencés dans les [revendications d’affichage](self-asserted-technical-profile.md#display-claims) du [profil technique auto-déclaré](self-asserted-technical-profile.md).

Par exemple :

```xml
<TechnicalProfile Id="SelfAsserted-ProfileUpdate">
  ...
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim DisplayControlReferenceId="PhoneVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des exemples d’utilisation du contrôle d’affichage, consultez : 

- [Vérification des e-mails personnalisée avec Mailjet](custom-email-mailjet.md)
- [Vérification des e-mails personnalisée avec SendGrid](custom-email-sendgrid.md)
