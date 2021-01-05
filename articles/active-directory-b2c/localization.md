---
title: Localisation - Azure Active Directory B2C
description: Spécifiez l’élément Localization d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 88244ec3ba4bbebe7d6096fa3ac49bd4f1b8f661
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108618"
---
# <a name="localization-element"></a>Élément de localisation

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

L’élément **Localization** vous permet de prendre en charge plusieurs paramètres régionaux ou langues dans la stratégie pour les parcours utilisateur. La prise en charge de la localisation dans les stratégies vous permet d’effectuer les actions suivantes :

- Configurer la liste explicite des langues prises en charge dans une stratégie et choisir une langue par défaut.
- Fournir des collections et des chaînes spécifiques d’une langue.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
  </SupportedLanguages>
  <LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedResources Id="api.localaccountsignup.es">
  ...
```

L’élément **Localization** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| activé | Non | Valeurs possibles : `true` ou `false`. |

L’élément **Localization** contient les éléments XML suivants

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| SupportedLanguages | 1:n | Liste des langues prises en charge. |
| LocalizedResources | 0:n | Liste des ressources localisées. |

## <a name="supportedlanguages"></a>SupportedLanguages

L’élément **SupportedLanguages** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| DefaultLanguage | Oui | Langue à utiliser par défaut pour les ressources localisées. |
| MergeBehavior | Non | Énumération de valeurs fusionnées avec un élément ClaimType quelconque présent dans une stratégie parente avec le même identificateur. Utilisez cet attribut quand vous remplacez une revendication spécifiée dans la stratégie de base. Valeurs possibles : `Append`, `Prepend` ou `ReplaceAll`. La valeur `Append` spécifie que la collection de données présente doit être ajoutée à la fin de la collection spécifiée dans la stratégie parente. La valeur `Prepend` spécifie que la collection de données présente doit être ajoutée devant la collection spécifiée dans la stratégie parente. La valeur `ReplaceAll` spécifie que la collecte de données définie dans la stratégie parente doit être ignorée, et que les données définies dans la stratégie actuelle doivent être utilisées à la place. |

### <a name="supportedlanguages"></a>SupportedLanguages

L’élément **SupportedLanguages** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| SupportedLanguage | 1:n | Affiche le contenu conforme à une balise de langue en vertu de la spécification RFC 5646 - Tags pour l’identification les langues. |

## <a name="localizedresources"></a>LocalizedResources

L’élément **LocalizedResources** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur servant à identifier de façon unique les ressources localisées. |

L’élément **LocalizedResources** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| LocalizedCollections | 0:n | Définit des collections entières dans des cultures différentes. Une collection peut contenir un nombre différent d’éléments, et des chaînes différentes pour diverses cultures. Les exemples de collections incluent les énumérations qui apparaissent dans les types de revendications. Par exemple, une liste de pays/régions s’affiche à l’utilisateur dans une liste déroulante. |
| LocalizedStrings | 0:n | Définit toutes les chaînes, à l’exception de celles qui apparaissent dans des collections, dans diverses cultures. |

### <a name="localizedcollections"></a>LocalizedCollections

Le **LocalizedCollections** élément contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| LocalizedCollection | 1:n | Liste des langues prises en charge. |

#### <a name="localizedcollection"></a>LocalizedCollection

L’élément **LocalizedCollections** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ElementType | Oui | Fait référence à un élément ClaimType ou à un élément d’interface utilisateur dans le fichier de stratégie. |
| ElementId | Oui | Chaîne contenant une référence à un type de revendication déjà défini dans la section ClaimsSchema, qui est utilisée si **ElementType** est défini sur ClaimType. |
| TargetCollection | Oui | Collection cible. |

L’élément **LocalizedCollections** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Élément | 0:n | Définit une option que l’utilisateur peut sélectionner pour une revendication dans l’interface utilisateur, telle qu’une valeur dans une liste déroulante. |

L’élément **Item** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Texte | Oui | Chaîne d’affichage convivial à présenter à l’utilisateur dans l’interface utilisateur pour cette option. |
| Valeur | Oui | Valeur de revendication chaîne associée à la sélection de cette option. |
| SelectByDefault | Non | Indique si cette option doit être sélectionnée par défaut dans l’interface utilisateur. Valeurs possibles : True ou False. |

L’exemple suivant illustre l’utilisation de l’élément **LocalizedCollections**. Il contient deux éléments **LocalizedCollection**, l’un pour l’anglais et un autre pour l’espagnol. Tous deux définissent la collection **Restriction** de la revendication `Gender` avec une liste d’éléments pour l’anglais et l’espagnol.

```xml
<LocalizedResources Id="api.selfasserted.en">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Female" Value="F" />
      <Item Text="Male" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>

<LocalizedResources Id="api.selfasserted.es">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Femenino" Value="F" />
      <Item Text="Masculino" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>
```

### <a name="localizedstrings"></a>LocalizedStrings

L’élément **LocalizedStrings** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| LocalizedString | 1:n | Chaîne localisée. |

L’élément **LocalizedString** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ElementType | Oui | Valeurs possibles : [ClaimsProvider](#claimsprovider), [ClaimType](#claimtype), [ErrorMessage](#errormessage), [GetLocalizedStringsTransformationClaimType](#getlocalizedstringstransformationclaimtype), [Predicate](#predicate), [InputValidation](#inputvalidation) ou [UxElement](#uxelement).   | 
| ElementId | Oui | Si **ElementType** a la valeur `ClaimType`, `Predicate`, ou `InputValidation`, cet élément contient une référence à un type de revendication déjà défini dans la section ClaimsSchema. |
| StringId | Oui | Si **ElementType** a la valeur `ClaimType`, cet élément contient une référence à un attribut d’un type de revendication. Valeurs possibles : `DisplayName`, `AdminHelpText` ou `PatternHelpText`. La valeur `DisplayName` est utilisée pour définir le nom d’affichage de la revendication. La valeur `AdminHelpText` est utilisée pour définir le nom du texte d’aide de l’utilisateur de la revendication. La valeur `PatternHelpText` est utilisée pour définir le texte d’aide du modèle de revendication. Si **ElementType** a la valeur `UxElement`, cet élément contient une référence à un attribut d’un élément d’interface utilisateur. Si **ElementType** a la valeur `ErrorMessage`, cet élément spécifie l’identificateur d’un message d’erreur. Pour obtenir la liste complète des `UxElement` identificateurs, voir [ID de chaîne de localisation](localization-string-ids.md).|

## <a name="elementtype"></a>ElementType

Référence ElementType à un type de revendication, une transformation de revendication ou un élément d’interface utilisateur dans la stratégie à localiser.

| Élément à localiser | ElementType | ElementId |StringId |
| --------- | -------- | ----------- |----------- |
| Nom du fournisseur d’identité |`ClaimsProvider`| | ID de l’élément ClaimsExchange|
| Attributs du type de revendication|`ClaimType`|Nom du type de revendication| Attribut de la revendication à localiser. Valeurs possibles : `AdminHelpText`, `DisplayName`, `PatternHelpText` et `UserHelpText`.|
|Message d’erreur|`ErrorMessage`||ID du message d’erreur |
|Copie les chaînes localisées dans des revendications|`GetLocalizedStringsTra nsformationClaimType`||Nom de la colonne de sortie|
|Message utilisateur du prédicat|`Predicate`|Nom du prédicat| L’attribut du prédicat à localiser. Valeurs possibles : `HelpText`.|
|Message utilisateur du groupe de prédicats|`InputValidation`|ID de l’élément PredicateValidation.|ID de l’élément PredicateGroup. Le groupe de prédicats doit être un enfant de l’élément de validation du prédicat tel que défini dans l’ElementId.|
|Options de l’interface utilisateur |`UxElement` | | ID de l’élément d’interface utilisateur à localiser.|
|[Display Control](display-controls.md) |`DisplayControl` |ID du contrôle d’affichage. | ID de l’élément d’interface utilisateur à localiser.|

## <a name="examples"></a>Exemples

### <a name="claimsprovider"></a>ClaimsProvider

La valeur ClaimsProvider est utilisée pour localiser l’un des noms d’affichage des fournisseurs de revendications. 

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="LinkedInExchange" TechnicalProfileReferenceId="LinkedIn-OAUTH" />
  </ClaimsExchanges>
</OrchestrationStep>

```

L’exemple suivant montre comment localiser le nom d’affichage des fournisseurs de revendications.

```xml
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">Facebook</LocalizedString>
<LocalizedString ElementType="ClaimsProvider" StringId="GoogleExchange">Google</LocalizedString>
<LocalizedString ElementType="ClaimsProvider" StringId="LinkedInExchange">LinkedIn</LocalizedString>
```

### <a name="claimtype"></a>ClaimType

La valeur ClaimType est utilisée pour localiser l’un des attributs de revendication. 

```xml
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

L’exemple suivant montre comment localiser les attributs DisplayName, UserHelpText et PatternHelpText du type de revendication e-mail.

```xml
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email</LocalizedString>
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Please enter your email</LocalizedString>
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address</LocalizedString>
```

### <a name="errormessage"></a>ErrorMessage

La valeur ErrorMessage est utilisée pour localiser l’un des messages d’erreur système. 

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalAlreadyExists">You are already registered, please press the back button and sign in instead.</Item>
  </Metadata>
  ...
</TechnicalProfile>
```

L’exemple suivant montre comment localiser le message d’erreur UserMessageIfClaimsPrincipalAlreadyExists.


```xml
<LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">The account you are trying to create already exists, please sign-in.</LocalizedString>
```

### <a name="getlocalizedstringstransformationclaimtype"></a>GetLocalizedStringsTransformationClaimType

La valeur GetLocalizedStringsTransformationClaimType est utilisée pour copier les chaînes localisées dans des revendications. Pour plus d’informations, consultez [Transformation de revendications GetLocalizedStringsTransformation](string-transformations.md#getlocalizedstringstransformation)


```xml
<ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
    <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
    <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
    <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
   </OutputClaims>
</ClaimsTransformation>
```

L’exemple suivant montre comment localiser les revendications de sortie de la transformation de revendications GetLocalizedStringsTransformation.

```xml
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for verifying your account!</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
```

### <a name="predicate"></a>Predicate

La valeur Predicate est utilisée pour localiser l’un des messages d’erreur [Predicate](predicates.md). 

```xml
<Predicates>
  <Predicate Id="LengthRange" Method="IsLengthRange"  HelpText="The password must be between 6 and 64 characters.">
    <Parameters>
      <Parameter Id="Minimum">6</Parameter>
      <Parameter Id="Maximum">64</Parameter>
    </Parameters>
  </Predicate>
  <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">a-z</Parameter>
    </Parameters>
  </Predicate>
  <Predicate Id="Uppercase" Method="IncludesCharacters" HelpText="an uppercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">A-Z</Parameter>
    </Parameters>
  </Predicate>
</Predicates>
```

L’exemple suivant montre comment localiser des prédicats dans le texte d’aide.

```xml
<LocalizedString ElementType="Predicate" ElementId="LengthRange" StringId="HelpText">The password must be between 6 and 64 characters.</LocalizedString>
<LocalizedString ElementType="Predicate" ElementId="Lowercase" StringId="HelpText">a lowercase letter</LocalizedString>
<LocalizedString ElementType="Predicate" ElementId="Uppercase" StringId="HelpText">an uppercase letter</LocalizedString>
```

### <a name="inputvalidation"></a>InputValidation

La valeur InputValidation est utilisée pour localiser l’un des messages d’erreur du groupe [PredicateValidation](predicates.md). 

```xml
<PredicateValidations>
  <PredicateValidation Id="CustomPassword">
    <PredicateGroups>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences MatchAtLeast="1">
          <PredicateReference Id="LengthRange" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="CharacterClasses">
        <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
        <PredicateReferences MatchAtLeast="3">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

L’exemple suivant montre comment localiser un texte d’aide du groupe de validation des prédicats.

```xml
<LocalizedString ElementType="InputValidation" ElementId="CustomPassword" StringId="CharacterClasses">The password must have at least 3 of the following:</LocalizedString>
```

### <a name="uxelement"></a>UxElement

La valeur UxElement est utilisée pour localiser l’un des éléments d’interface utilisateur. L’exemple suivant montre comment localiser les boutons Continuer et Annuler.

```xml
<LocalizedString ElementType="UxElement" StringId="button_continue">Create new account</LocalizedString>
<LocalizedString ElementType="UxElement" StringId="button_cancel">Cancel</LocalizedString>
```

### <a name="displaycontrol"></a>DisplayControl

La valeur DisplayControl est utilisée pour localiser l’un des éléments de l’interface utilisateur [DisplayControl](display-controls.md). L’exemple suivant montre comment localiser les boutons Envoyer et Vérifier. 

```xml
<LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_code">Send verification code</LocalizedString>
<LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_verify_code">Verify code</LocalizedString>
```

Dans la section Métadonnées d’un profil technique auto-déclaré, dans la propriété ContentDefinition référencée, DataUri doit être défini sur la [mise en page version](page-layout.md) 2.1.0 ou ultérieure. Par exemple :

```xml
<ContentDefinition Id="api.selfasserted">
  <DataUri>urn:com:microsoft:aad:b2c:elements:selfasserted:2.1.0</DataUri>
  ...
```

## <a name="next-steps"></a>Étapes suivantes

Pour voir des exemples de localisation, consultez les articles suivants :

- [Personnalisation de la langue avec la stratégie personnalisée dans Azure Active Directory B2C](language-customization.md)
- [Personnalisation de la langue avec les flux d’utilisateurs dans Azure Active Directory B2C](language-customization.md)
