---
title: Exemples de transformation de revendications de chaînes pour les stratégies personnalisées
titleSuffix: Azure AD B2C
description: Exemples de transformations de revendications de chaînes pour le schéma Identity Experience Framework (IEF) d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 11/03/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 4e74c33a18baff3e1cb39328ce265f16975ef1b5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994840"
---
# <a name="string-claims-transformations"></a>Transformations de revendications de chaînes

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Cet article fournit des exemples pour l’utilisation de transformations de revendications de chaîne du schéma Identity Experience Framework dans Azure Active Directory B2C (Azure AD B2C). Pour plus d’informations, voir [ClaimsTransformations](claimstransformations.md).

## <a name="assertstringclaimsareequal"></a>AssertStringClaimsAreEqual

Compare deux revendications et lève une exception si elles ne sont pas égales en fonction des éléments de comparaison inputClaim1, inputClaim2 et stringComparison spécifiés.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Type de la première revendication qui doit être comparée. |
| InputClaim | inputClaim2 | string | Type de la deuxième revendication qui doit être comparée. |
| InputParameter | stringComparison | string | comparaison de chaînes, une des valeurs suivantes : Ordinal, OrdinalIgnoreCase. |

La transformation de revendication **AssertStringClaimsAreEqual** est toujours exécutée à partir d’un [profil technique de validation](validation-technical-profile.md) appelé par un [profil technique autodéclaré](self-asserted-technical-profile.md) ou un [DisplayConrtol](display-controls.md). Les métadonnées `UserMessageIfClaimsTransformationStringsAreNotEqual` d’un profil technique autodéclaré contrôlent le message d’erreur présenté à l’utilisateur. Les messages d’erreur peuvent être [localisés](localization-string-ids.md#claims-transformations-error-messages).


![Exécution de AssertStringClaimsAreEqual](./media/string-transformations/assert-execution.png)

Vous pouvez utiliser cette transformation de revendication pour vérifier que deux ClaimTypes ont la même valeur. Si ce n’est pas le cas, un message d’erreur est levé. L’exemple suivant vérifie que le ClaimType **strongAuthenticationEmailAddress** est égal au ClaimType **email**. Si ce n’est pas le cas, un message d’erreur est levé.

```xml
<ClaimsTransformation Id="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" TransformationMethod="AssertStringClaimsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="strongAuthenticationEmailAddress" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
  </InputParameters>
</ClaimsTransformation>
```


Le profil technique de validation **login-NonInteractive** appelle la transformation de revendication **AssertEmailAndStrongAuthenticationEmailAddressAreEqual**.
```xml
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

Le profil technique autodéclaré appelle le profil technique de validation **login-NonInteractive**.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="UserMessageIfClaimsTransformationStringsAreNotEqual">Custom error message the email addresses you provided are not the same.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **inputClaim1** : someone@contoso.com
  - **inputClaim2** : someone@outlook.com
- Paramètres d’entrée :
  - **stringComparison** : ordinalIgnoreCase
- Résultat : Erreur levée

## <a name="changecase"></a>ChangeCase

Modifie la casse de la revendication fournie (minuscules ou majuscules) en fonction de l’opérateur.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Le ClaimType à changer. |
| InputParameter | toCase | string | L’une des valeurs suivantes : `LOWER` ou `UPPER`. |
| OutputClaim | outputClaim | string | ClaimType généré après que cette transformation de revendication a été appelée. |

Utilisez cette transformation de revendication pour mettre un ClaimType en majuscules ou en minuscules.

```xml
<ClaimsTransformation Id="ChangeToLower" TransformationMethod="ChangeCase">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim1" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="toCase" DataType="string" Value="LOWER" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **email** : SomeOne@contoso.com
- Paramètres d’entrée :
    - **toCase** : LOWER
- Revendications de sortie :
  - **email** : someone@contoso.com

## <a name="createstringclaim"></a>CreateStringClaim

Crée une revendication de chaîne à partir du paramètre d’entrée fourni dans la transformation.

| Élément | TransformationClaimType | Type de données | Notes |
|----- | ----------------------- | --------- | ----- |
| InputParameter | value | string | Chaîne à définir. Ce paramètre d’entrée prend en charge les [expressions de transformation de revendications de chaînes](string-transformations.md#string-claim-transformations-expressions). |
| OutputClaim | createdClaim | string | ClaimType généré après que cette transformation de revendication a été appelée, avec la valeur spécifiée dans le paramètre d’entrée. |

Utilisez cette transformation de revendication pour définir une chaîne de valeur ClaimType.

```xml
<ClaimsTransformation Id="CreateTermsOfService" TransformationMethod="CreateStringClaim">
  <InputParameters>
    <InputParameter Id="value" DataType="string" Value="Contoso terms of service..." />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="TOS" TransformationClaimType="createdClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Paramètre d’entrée :
    - **value** : Contoso terms of service...
- Revendications de sortie :
    - **createdClaim** : le ClaimType TOS contient la valeur « Contoso terms of service... ».

## <a name="compareclaims"></a>CompareClaims

Détermine si une revendication de chaîne est égale à une autre. Le résultat est un nouveau ClaimType booléen avec la valeur `true` ou `false`.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Premier type de revendication à comparer. |
| InputClaim | inputClaim2 | string | Deuxième type de revendication à comparer. |
| InputParameter | operator | string | Valeurs possibles : `EQUAL` ou `NOT EQUAL`. |
| InputParameter | ignoreCase | boolean | Spécifie si cette comparaison doit ignorer la casse des chaînes comparées. |
| OutputClaim | outputClaim | boolean | ClaimType généré après que cette transformation de revendication a été appelée. |

Utilisez cette transformation de revendication pour vérifier si une revendication est égale à une autre. Par exemple, la transformation de revendication suivante vérifie si la valeur de la revendication **email** est égale à la revendication **Verified.Email**.

```xml
<ClaimsTransformation Id="CheckEmail" TransformationMethod="CompareClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="Email" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="Verified.Email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="operator" DataType="string" Value="NOT EQUAL" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="SameEmailAddress" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **inputClaim1** : someone@contoso.com
  - **inputClaim2** : someone@outlook.com
- Paramètres d’entrée :
    - **operator** :  NON ÉGAL À
    - **ignoreCase** : true
- Revendications de sortie :
    - **outputClaim** : true

## <a name="compareclaimtovalue"></a>CompareClaimToValue

Détermine si une valeur de revendication est égale à la valeur du paramètre d’entrée.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | string | Type de la revendication à comparer. |
| InputParameter | operator | string | Valeurs possibles : `EQUAL` ou `NOT EQUAL`. |
| InputParameter | compareTo | string | comparaison de chaînes, une des valeurs suivantes : Ordinal, OrdinalIgnoreCase. |
| InputParameter | ignoreCase | boolean | Spécifie si cette comparaison doit ignorer la casse des chaînes comparées. |
| OutputClaim | outputClaim | boolean | ClaimType généré après que cette transformation de revendication a été appelée. |

Vous pouvez utiliser cette transformation de revendication pour vérifier si une revendication est égale à une valeur que vous avez spécifiée. Par exemple, la transformation de revendication suivante vérifie si la valeur de la revendication **termsOfUseConsentVersion** est égale à `v1`.

```xml
<ClaimsTransformation Id="IsTermsOfUseConsentRequiredForVersion" TransformationMethod="CompareClaimToValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim1" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="compareTo" DataType="string" Value="V1" />
    <InputParameter Id="operator" DataType="string" Value="not equal" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple
- Revendications d’entrée :
    - **inputClaim1** : v1
- Paramètres d’entrée :
    - **compareTo** : V1
    - **operator** : EQUAL
    - **ignoreCase** : true
- Revendications de sortie :
    - **outputClaim** : true

## <a name="createrandomstring"></a>CreateRandomString

Crée une chaîne aléatoire à l’aide du générateur de nombres aléatoires. Si le générateur de nombres aléatoires est de type `integer`, vous pouvez éventuellement spécifier un paramètre de départ et un nombre maximal. Un paramètre de format de chaîne facultatif permet de mettre en forme la sortie, et un paramètre base64 facultatif spécifie si la sortie est outputClaim (string) randomGeneratorType [guid, entier] encodée en base64.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputParameter | randomGeneratorType | string | Spécifie la valeur aléatoire à générer, `GUID` (ID global unique) ou `INTEGER` (nombre). |
| InputParameter | stringFormat | string | [Facultatif] Mettre en forme la valeur aléatoire. |
| InputParameter | base64 | boolean | [Facultatif] Convertir la valeur aléatoire en base64. Si la mise en forme de la chaîne est appliquée, la valeur après la mise en forme de la chaîne est encodée en base64. |
| InputParameter | maximumNumber | int | [Facultatif] Pour randomGeneratorType `INTEGER` uniquement. Spécifiez le nombre maximal. |
| InputParameter | seed  | int | [Facultatif] Pour randomGeneratorType `INTEGER` uniquement. Spécifiez la valeur de départ pour la valeur aléatoire. Remarque : la même valeur de départ génère la même séquence de nombres aléatoires. |
| OutputClaim | outputClaim | string | ClaimType généré après que cette transformation de revendication a été appelée. Valeur aléatoire. |

L’exemple suivant génère un ID unique global. Cette transformation de revendication permet de créer l’UPN (nom d’utilisateur principal) aléatoire.

```xml
<ClaimsTransformation Id="CreateRandomUPNUserName" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="GUID" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Exemple

- Paramètres d’entrée :
    - **randomGeneratorType** : GUID
- Revendications de sortie :
    - **outputClaim** : bc8bedd2-aaa3-411e-bdee-2f1810b73dfc

L’exemple suivant génère une valeur entière aléatoire comprise entre 0 et 1000. La valeur est mise en forme au format OTP_ {valeur aléatoire}.

```xml
<ClaimsTransformation Id="SetRandomNumber" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="INTEGER" />
    <InputParameter Id="maximumNumber" DataType="int" Value="1000" />
    <InputParameter Id="stringFormat" DataType="string" Value="OTP_{0}" />
    <InputParameter Id="base64" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="randomNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Paramètres d’entrée :
    - **randomGeneratorType** : INTEGER
    - **maximumNumber** : 1 000
    - **stringFormat** : OTP_{0}
    - **base64** : false
- Revendications de sortie :
    - **outputClaim** : OTP_853


## <a name="formatstringclaim"></a>FormatStringClaim

Met en forme une revendication en fonction de la chaîne de format fournie. Cette transformation utilise la méthode C# `String.Format`.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |string |ClaimType qui agit en tant que paramètre {0} de format de chaîne. |
| InputParameter | stringFormat | string | Format de chaîne, y compris le paramètre {0}. Ce paramètre d’entrée prend en charge les [expressions de transformation de revendications de chaînes](string-transformations.md#string-claim-transformations-expressions).  |
| OutputClaim | outputClaim | string | ClaimType généré après que cette transformation de revendication a été appelée. |

Utilisez cette transformation de revendication pour mettre en forme une chaîne avec un paramètre {0}. L’exemple suivant crée un **userPrincipalName**. Tous les profils techniques de fournisseurs d’identité sociale, tels que `Facebook-OAUTH` appellent **CreateUserPrincipalName** pour générer un **userPrincipalName**.

```xml
<ClaimsTransformation Id="CreateUserPrincipalName" TransformationMethod="FormatStringClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="cpim_{0}@{RelyingPartyTenantId}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim** : 5164db16-3eee-4629-bfda-dcc3326790e9
- Paramètres d’entrée :
    - **stringFormat** :  cpim_{0}@{RelyingPartyTenantId}
- Revendications de sortie :
  - **outputClaim** : cpim_5164db16-3eee-4629-bfda-dcc3326790e9@b2cdemo.onmicrosoft.com

## <a name="formatstringmultipleclaims"></a>FormatStringMultipleClaims

Met en forme deux revendications en fonction de la chaîne de format fournie. Cette transformation utilise la méthode C# `String.Format`.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |string | ClaimType qui agit en tant que paramètre {0} de format de chaîne. |
| InputClaim | inputClaim | string | ClaimType qui agit en tant que paramètre {1} de format de chaîne. |
| InputParameter | stringFormat | string | Format de chaîne, y compris les paramètres {0} et {1}. Ce paramètre d’entrée prend en charge les [expressions de transformation de revendications de chaînes](string-transformations.md#string-claim-transformations-expressions).   |
| OutputClaim | outputClaim | string | ClaimType généré après que cette transformation de revendication a été appelée. |

Utilisez cette transformation de revendication pour mettre en forme une chaîne avec deux paramètres, {0} et {1}. L’exemple suivant crée un **displayName** au format spécifié :

```xml
<ClaimsTransformation Id="CreateDisplayNameFromFirstNameAndLastName" TransformationMethod="FormatStringMultipleClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="surName" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="{0} {1}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="displayName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim1** : Joe
    - **inputClaim2** : Fernando
- Paramètres d’entrée :
    - **stringFormat** : {0} {1}
- Revendications de sortie :
    - **outputClaim** : Joe Fernando

## <a name="getlocalizedstringstransformation"></a>GetLocalizedStringsTransformation

Copie les chaînes localisées dans des revendications.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| OutputClaim | Nom de la chaîne localisée | string | Liste des types de revendications qui sont générées après l’appel de cette transformation de revendications. |

Pour utiliser la transformation de revendications GetLocalizedStringsTransformation :

1. Définissez une [chaîne de localisation](localization.md) et associez-la à un [profil technique autodéclaré](self-asserted-technical-profile.md).
1. La valeur `ElementType` de l’élément `LocalizedString` doit être définie sur `GetLocalizedStringsTransformationClaimType`.
1. `StringId` est un identificateur unique que vous définissez en vue de l’utiliser plus tard dans votre transformation de revendications.
1. Dans la transformation de revendications, spécifiez la liste des revendications à définir avec la chaîne localisée. `ClaimTypeReferenceId` est une référence à un ClaimType déjà défini dans la section ClaimsSchema dans la stratégie. `TransformationClaimType` est le nom de la chaîne localisée, tel qu’il est défini dans la valeur `StringId` de l’élément `LocalizedString`.
1. Dans un [profil technique autodéclaré](self-asserted-technical-profile.md), ou une transformation de revendications d’entrée ou de sortie d’un [contrôle d’affichage](display-controls.md), faites référence à votre transformation de revendications.

![GetLocalizedStringsTransformation](./media/string-transformations/get-localized-strings-transformation.png)

L’exemple suivant recherche l’objet, le corps, le message, le code et la signature de votre e-mail à partir des chaînes localisées. Ces revendications seront ensuite utilisées par un modèle personnalisé de vérification de l’e-mail.

Définissez les chaînes localisées pour l’anglais (par défaut) et l’espagnol.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
   </SupportedLanguages>

  <LocalizedResources Id="api.localaccountsignup.en">
    <LocalizedStrings>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for verifying your account!</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
     </LocalizedStrings>
   </LocalizedResources>
   <LocalizedResources Id="api.localaccountsignup.es">
     <LocalizedStrings>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Código de verificación del correo electrónico de la cuenta de Contoso</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Gracias por comprobar la cuenta de </LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Su código es</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Atentamente</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
</Localization>
```

La transformation de revendications définit la valeur du type de revendication *subject* à la valeur `StringId` pour *email_subject*.

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

### <a name="example"></a>Exemple

- Revendications de sortie :
  - **subject** : Code de vérification de l’e-mail du compte Contoso
  - **message** : Merci, votre compte a été vérifié.
  - **codeIntro** : Votre code est
  - **signature** : Cordialement,


## <a name="getmappedvaluefromlocalizedcollection"></a>GetMappedValueFromLocalizedCollection

Recherche un élément dans une collection de revendications **Restriction**.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | mapFromClaim | string | Revendication qui contient le texte à rechercher dans les revendications **restrictionValueClaim** avec la collection **Restriction**.  |
| OutputClaim | restrictionValueClaim | string | Revendication qui contient la collection **Restriction**. Une fois la transformation de revendications appelée, la valeur de cette revendication contient la valeur de l’élément sélectionné. |

L’exemple suivant recherche la description de message d’erreur en fonction de la clé de l’erreur. La revendication **responseMsg** contient une collection de messages d’erreur à présenter à l’utilisateur final ou à envoyer à la partie de confiance.

```xml
<ClaimType Id="responseMsg">
  <DisplayName>Error message: </DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
  <Restriction>
    <Enumeration Text="B2C_V1_90001" Value="You cannot sign in because you are a minor" />
    <Enumeration Text="B2C_V1_90002" Value="This action can only be performed by gold members" />
    <Enumeration Text="B2C_V1_90003" Value="You have not been enabled for this operation" />
  </Restriction>
</ClaimType>
```
La transformation de revendication recherche le texte de l’élément et retourne sa valeur. Si la restriction est localisée à l’aide de `<LocalizedCollection>`, la transformation de revendication retourne la valeur localisée.

```xml
<ClaimsTransformation Id="GetResponseMsgMappedToResponseCode" TransformationMethod="GetMappedValueFromLocalizedCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="responseCode" TransformationClaimType="mapFromClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" TransformationClaimType="restrictionValueClaim" />        
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **mapFromClaim** : B2C_V1_90001
- Revendications de sortie :
    - **restrictionValueClaim** : Vous ne pouvez pas vous connecter parce que vous en êtes mineur.

## <a name="lookupvalue"></a>LookupValue

Recherche une valeur de revendication dans une liste de valeurs en fonction de la valeur d’une autre revendication.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputParameterId | string | Revendication qui contient la valeur de recherche |
| InputParameter | |string | Collection d’inputParameters. |
| InputParameter | errorOnFailedLookup | boolean | Contrôle si une erreur est retournée en l’absence de correspondance. |
| OutputClaim | inputParameterId | string | ClaimType généré après que cette transformation de revendication a été appelée. Valeur de l’`Id` correspondant. |

L’exemple suivant recherche le nom de domaine dans l’une des collections inputParameters. La transformation de revendication recherche le nom de domaine dans l’identificateur et retourne sa valeur (un ID d’application).

```xml
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputParameterId** : test.com
- Paramètres d’entrée :
    - **contoso.com** : 13c15f79-8fb1-4e29-a6c9-be0d36ff19f1
    - **microsoft.com** : 0213308f-17cb-4398-b97e-01da7bd4804e
    - **test.com** : c7026f88-4299-4cdb-965d-3f166464b8a9
    - **errorOnFailedLookup** : false
- Revendications de sortie :
    - **outputClaim**:    c7026f88-4299-4cdb-965d-3f166464b8a9

Lorsque le paramètre d’entrée `errorOnFailedLookup` est défini sur `true`, la transformation de revendications **LookupValue** est toujours exécutée à partir d’un [profil technique de validation](validation-technical-profile.md) appelé par un [profil technique autodéclaré](self-asserted-technical-profile.md) ou un [DisplayConrtol](display-controls.md). Les métadonnées `LookupNotFound` d’un profil technique autodéclaré contrôlent le message d’erreur présenté à l’utilisateur.

![Exécution de AssertStringClaimsAreEqual](./media/string-transformations/assert-execution.png)

L’exemple suivant recherche le nom de domaine dans l’une des collections inputParameters. La transformation de revendication recherche le nom de domaine dans l’identificateur et retourne sa valeur (un ID d’application) ou génère un message d’erreur.

```xml
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputParameterId** : live.com
- Paramètres d’entrée :
    - **contoso.com** : 13c15f79-8fb1-4e29-a6c9-be0d36ff19f1
    - **microsoft.com** : 0213308f-17cb-4398-b97e-01da7bd4804e
    - **test.com** : c7026f88-4299-4cdb-965d-3f166464b8a9
    - **errorOnFailedLookup** : true
- Erreur :
    - Aucune correspondance trouvée pour la valeur de revendication d’entrée dans la liste d’ID de paramètre d’entrée et errorOnFailedLookup a la valeur true.


## <a name="nullclaim"></a>NullClaim

Nettoie la valeur d’une revendication donnée.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| OutputClaim | claim_to_null | string | La valeur de la revendication est définie sur NULL. |

Utilisez cette transformation de revendication pour supprimer les données inutiles du jeu de propriétés de revendications afin que le cookie de session soit plus petit. L’exemple suivant supprime la valeur du type de revendication `TermsOfService`.

```xml
<ClaimsTransformation Id="SetTOSToNull" TransformationMethod="NullClaim">
  <OutputClaims>
  <OutputClaim ClaimTypeReferenceId="TermsOfService" TransformationClaimType="claim_to_null" />
  </OutputClaims>
</ClaimsTransformation>
```

- Revendications d’entrée :
    - **outputClaim** : Welcome to Contoso App. Si vous continuez à parcourir et à utiliser ce site web, vous acceptez de vous conformer aux conditions générales suivantes...
- Revendications de sortie :
    - **outputClaim** : NULL

## <a name="parsedomain"></a>ParseDomain

Obtient la partie domaine d’une adresse e-mail.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | emailAddress | string | ClaimType qui contient l’adresse e-mail. |
| OutputClaim | domaine | string | ClaimType généré après que cette transformation de revendication a été appelée (le domaine). |

Utilisez cette transformation de revendication pour analyser le nom de domaine de l’utilisateur après le symbole @. La transformation de revendication suivante montre comment analyser le nom de domaine d’une revendication **e-mail**.

```xml
<ClaimsTransformation Id="SetDomainName" TransformationMethod="ParseDomain">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="emailAddress" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="domain" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **emailAddress** : joe@outlook.com
- Revendications de sortie :
    - **domain** : outlook.com

## <a name="setclaimsifregexmatch"></a>SetClaimsIfRegexMatch

Vérifie qu’une revendication de chaîne `claimToMatch` et un paramètre d’entrée `matchTo` sont égaux, et définit les revendications de sortie avec la valeur présente dans le paramètre d’entrée `outputClaimIfMatched`, ainsi que la revendication de sortie de résultat de comparaison, qui est définie sur `true` ou `false` en fonction du résultat de la comparaison.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| inputClaim | claimToMatch | string | Type de revendication à comparer. |
| InputParameter | matchTo | string | Expression régulière à mettre en correspondance. |
| InputParameter | outputClaimIfMatched | string | Valeur à définir si les chaînes sont égales. |
| InputParameter | extractGroups | boolean | [Facultatif] Spécifie si la correspondance d’expression régulière doit extraire les valeurs des groupes. Valeurs possibles : `true` ou `false` (par défaut). | 
| OutputClaim | outputClaim | string | Si l’expression régulière correspond, cette revendication de sortie contient la valeur du paramètre d’entrée `outputClaimIfMatched`. Si aucune correspondance n’est trouvée, la valeur sera Null. |
| OutputClaim | regexCompareResultClaim | boolean | Type de revendication de la sortie de résultat de correspondance d’expression régulière, qui doit être défini sur `true` ou `false` en fonction du résultat de la correspondance. |
| OutputClaim| Nom de la revendication| string | Si le paramètre d’entrée extractGroups a la valeur true, liste des types de revendications qui sont générées après l’appel de cette transformation de revendications. Le nom du claimType doit correspondre au nom du groupe d’expressions régulières. | 

### <a name="example-1"></a>Exemple 1

Vérifie si le numéro de téléphone fourni est valide, selon le modèle d’expression régulière de numéro de téléphone.

```xml
<ClaimsTransformation Id="SetIsPhoneRegex" TransformationMethod="SetClaimsIfRegexMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phone" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="^[0-9]{4,16}$" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="isPhone" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="validationResult" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isPhoneBoolean" TransformationClaimType="regexCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

- Revendications d’entrée :
    - **claimToMatch** : "64854114520"
- Paramètres d’entrée :
    - **matchTo** : "^[0-9]{4,16}$"
    - **outputClaimIfMatched** : "isPhone"
- Revendications de sortie :
    - **outputClaim** : "isPhone"
    - **regexCompareResultClaim** : true

### <a name="example-2"></a>Exemple 2

Vérifie si l’adresse e-mail fournie est valide, puis retourne l’alias de messagerie.

```xml
<ClaimsTransformation Id="GetAliasFromEmail" TransformationMethod="SetClaimsIfRegexMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="(?&lt;mailAlias&gt;.*)@(.*)$" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="isEmail" />
    <InputParameter Id="extractGroups" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="validationResult" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isEmailString" TransformationClaimType="regexCompareResultClaim" />
    <OutputClaim ClaimTypeReferenceId="mailAlias" />
  </OutputClaims>
</ClaimsTransformation>
```

- Revendications d’entrée :
    - **claimToMatch** : « emily@contoso.com »
- Paramètres d’entrée :
    - **matchTo** : `(?&lt;mailAlias&gt;.*)@(.*)$`
    - **outputClaimIfMatched** : « isEmail »
    - **extractGroups** : true
- Revendications de sortie :
    - **outputClaim** : « isEmail »
    - **regexCompareResultClaim** : true
    - **mailAlias** : emily
    
## <a name="setclaimsifstringsareequal"></a>SetClaimsIfStringsAreEqual

Vérifie qu’une revendication de chaîne et un paramètre d’entrée `matchTo` sont égaux, et définit les revendications de sortie avec la valeur présente dans les paramètres d’entrée `stringMatchMsg` et `stringMatchMsgCode`, ainsi que la revendication de sortie de résultat de comparaison, qui est définie sur `true` ou `false` en fonction du résultat de la comparaison.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Type de revendication à comparer. |
| InputParameter | matchTo | string | Chaîne à comparer à `inputClaim`. |
| InputParameter | stringComparison | string | Valeurs possibles : `Ordinal` ou `OrdinalIgnoreCase`. |
| InputParameter | stringMatchMsg | string | Première valeur à définir si les chaînes sont égales. |
| InputParameter | stringMatchMsgCode | string | Deuxième valeur à définir si les chaînes sont égales. |
| OutputClaim | outputClaim1 | string | Si les chaînes sont égales, cette revendication de sortie contient la valeur du paramètre d’entrée `stringMatchMsg`. |
| OutputClaim | outputClaim2 | string | Si les chaînes sont égales, cette revendication de sortie contient la valeur du paramètre d’entrée `stringMatchMsgCode`. |
| OutputClaim | stringCompareResultClaim | boolean | Type de revendication de la sortie de résultat de comparaison, qui doit être défini sur `true` ou `false` en fonction du résultat de la comparaison. |

Vous pouvez utiliser cette transformation de revendication pour vérifier si une revendication est égale à la valeur que vous avez spécifiée. Par exemple, la transformation de revendication suivante vérifie si la valeur de la revendication **termsOfUseConsentVersion** est égale à `v1`. Si c’est le cas, elle remplace la valeur par `v2`.

```xml
<ClaimsTransformation Id="CheckTheTOS" TransformationMethod="SetClaimsIfStringsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="v1" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="stringMatchMsg" DataType="string" Value="B2C_V1_90005" />
    <InputParameter Id="stringMatchMsgCode" DataType="string" Value="The TOS is upgraded to v2" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="outputClaim1" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeCode" TransformationClaimType="outputClaim2" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeResult" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim** : v1
- Paramètres d’entrée :
    - **matchTo** : V1
    - **stringComparison** : ordinalIgnoreCase
    - **stringMatchMsg** :  B2C_V1_90005
    - **stringMatchMsgCode** :  The TOS is upgraded to v2
- Revendications de sortie :
    - **outputClaim1** : B2C_V1_90005
    - **outputClaim2** : The TOS is upgraded to v2
    - **stringCompareResultClaim** : true

## <a name="setclaimsifstringsmatch"></a>SetClaimsIfStringsMatch

Vérifie qu’une revendication de chaîne et un paramètre d’entrée `matchTo` sont égaux, et définit les revendications de sortie avec la valeur présente dans le paramètre d’entrée `outputClaimIfMatched`, ainsi que la revendication de sortie de résultat de comparaison, qui est définie sur `true` ou `false` en fonction du résultat de la comparaison.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | claimToMatch | string | Type de revendication à comparer. |
| InputParameter | matchTo | string | Chaîne à comparer à inputClaim. |
| InputParameter | stringComparison | string | Valeurs possibles : `Ordinal` ou `OrdinalIgnoreCase`. |
| InputParameter | outputClaimIfMatched | string | Valeur à définir si les chaînes sont égales. |
| OutputClaim | outputClaim | string | Si les chaînes sont égales, cette revendication de sortie contient la valeur du paramètre d’entrée `outputClaimIfMatched`. Ou null, si les chaînes ne sont pas mises en correspondance. |
| OutputClaim | stringCompareResultClaim | boolean | Type de revendication de la sortie de résultat de comparaison, qui doit être défini sur `true` ou `false` en fonction du résultat de la comparaison. |

Par exemple, la transformation de revendication suivante vérifie si la valeur de la revendication **ageGroup** est égale à `Minor`. Si c’est le cas, elle retourne la valeur `B2C_V1_90001`.

```xml
<ClaimsTransformation Id="SetIsMinor" TransformationMethod="SetClaimsIfStringsMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="ageGroup" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="Minor" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="B2C_V1_90001" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isMinor" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isMinorResponseCode" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **claimToMatch** : Secondaire
- Paramètres d’entrée :
    - **matchTo** : Secondaire
    - **stringComparison** : ordinalIgnoreCase
    - **outputClaimIfMatched** :  B2C_V1_90001
- Revendications de sortie :
    - **isMinorResponseCode** : B2C_V1_90001
    - **isMinor** : true


## <a name="stringcontains"></a>StringContains

Détermine si une sous-chaîne spécifiée apparaît dans la revendication d’entrée. Le résultat est un nouveau ClaimType booléen avec la valeur `true` ou `false`. `true` si le paramètre de valeur se produit dans cette chaîne ; sinon, `false`.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Type de revendication dans lequel effectuer la recherche. |
|InputParameter|contains|string|Valeur à rechercher.|
|InputParameter|ignoreCase|string|Spécifie si cette comparaison doit ignorer la casse de la chaîne comparée.|
| OutputClaim | outputClaim | string | ClaimType généré après l’appel de cette ClaimsTransformation. Indicateur booléen si la sous-chaîne apparaît dans la revendication d’entrée. |

Utilisez cette transformation de revendication pour vérifier si un type de revendication de chaîne contient une sous-chaîne. L’exemple suivant vérifie si le type de revendication de chaîne `roles` contient la valeur **admin**.

```xml
<ClaimsTransformation Id="CheckIsAdmin" TransformationMethod="StringContains">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="roles" TransformationClaimType="inputClaim"/>
  </InputClaims>
  <InputParameters>
    <InputParameter  Id="contains" DataType="string" Value="admin"/>
    <InputParameter  Id="ignoreCase" DataType="string" Value="true"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isAdmin" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim** : "Admin, Approver, Editor"
- Paramètres d’entrée :
    - **contains** : "admin,"
    - **ignoreCase** : true
- Revendications de sortie :
    - **outputClaim** : true

## <a name="stringsubstring"></a>StringSubstring

Extrait des parties d’un type de revendication de chaîne, en commençant au caractère à la position spécifiée et retourne le nombre spécifié de caractères.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Type de revendication qui contient la chaîne. |
| InputParameter | index_début | int | Position du caractère de départ de base zéro d’une sous-chaîne de cette instance. |
| InputParameter | length | int | Nombre de caractères dans la sous-chaîne. |
| OutputClaim | outputClaim | boolean | Chaîne équivalente à la sous-chaîne length qui commence au niveau de startIndex dans cette instance, ou Empty si startIndex est égal à la longueur de cette instance et que la longueur est égale à zéro. |

Par exemple, obtenir le préfixe international du numéro de téléphone.


```xml
<ClaimsTransformation Id="GetPhonePrefix" TransformationMethod="StringSubstring">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="inputClaim" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="startIndex" DataType="int" Value="0" />
  <InputParameter Id="length" DataType="int" Value="2" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phonePrefix" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim** : "+1644114520"
- Paramètres d’entrée :
    - **startIndex** : 0
    - **length** :  2
- Revendications de sortie :
    - **outputClaim** : "+1"

## <a name="stringreplace"></a>StringReplace

Recherche une valeur spécifiée dans une chaîne de type de revendication et retourne une nouvelle chaîne de type de revendication dans laquelle toutes les occurrences d’une chaîne spécifiée dans la chaîne actuelle sont remplacées par une autre chaîne spécifiée.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Type de revendication qui contient la chaîne. |
| InputParameter | oldValue | string | Chaîne dans laquelle effectuer la recherche. |
| InputParameter | newValue | string | Chaîne permettant de remplacer toutes les occurrences de `oldValue`. |
| OutputClaim | outputClaim | boolean | Chaîne équivalente à la chaîne actuelle, sauf que toutes les instances de oldValue sont remplacées par newValue. Si oldValue est introuvable dans l’instance actuelle, la méthode retourne l’instance actuelle sans modification. |

Par exemple, normalisez un numéro de téléphone en supprimant les caractères `-`


```xml
<ClaimsTransformation Id="NormalizePhoneNumber" TransformationMethod="StringReplace">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="inputClaim" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="oldValue" DataType="string" Value="-" />
  <InputParameter Id="newValue" DataType="string" Value="" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim** : "+164-411-452-054"
- Paramètres d’entrée :
    - **oldValue** : "-"
    - **newValue** : ""
- Revendications de sortie :
    - **outputClaim** : "+164411452054"

## <a name="stringjoin"></a>StringJoin

Concatène les éléments d’un type de revendication de collection de chaînes spécifié, en utilisant le séparateur spécifié entre chaque élément ou membre.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | stringCollection | Collection qui contient les chaînes à concaténer. |
| InputParameter | delimiter | string | Chaîne à utiliser comme séparateur, telle que la virgule `,`. |
| OutputClaim | outputClaim | string | Chaîne composée des membres de la collection de chaînes `inputClaim`, délimitée par le paramètre d’entrée `delimiter`. |

L’exemple suivant prend une collection de chaînes de rôles d’utilisateur et la convertit en chaîne délimitée par des virgules. Vous pouvez utiliser cette méthode pour stocker une collection de chaînes dans un compte d’utilisateur Azure AD. Plus tard, lorsque vous lirez le compte à partir du répertoire, utilisez le paramètre `StringSplit` pour reconvertir la chaîne délimitée par des virgules en collection de chaînes.

```xml
<ClaimsTransformation Id="ConvertRolesStringCollectionToCommaDelimiterString" TransformationMethod="StringJoin">
  <InputClaims>
   <InputClaim ClaimTypeReferenceId="roles" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter DataType="string" Id="delimiter" Value="," />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="rolesCommaDelimiterConverted" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **inputClaim** : [ "Admin", "Author", "Reader" ]
- Paramètres d’entrée :
  - **delimiter** : ","
- Revendications de sortie :
  - **outputClaim** : "Admin,Author,Reader"


## <a name="stringsplit"></a>StringSplit

Retourne un tableau de chaînes qui contient les sous-chaînes de cette instance, séparées par les éléments d’une chaîne spécifiée.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | string | Type de revendication de chaîne qui contient les sous-chaînes à fractionner. |
| InputParameter | delimiter | string | Chaîne à utiliser comme séparateur, telle que la virgule `,`. |
| OutputClaim | outputClaim | stringCollection | Collection de chaînes dont les éléments contiennent les sous-chaînes de cette chaîne, qui sont délimitées par le paramètre d’entrée `delimiter`. |

L’exemple suivant prend une chaîne de rôles d’utilisateur délimitée par des virgules et la convertit en collection de chaînes.

```xml
<ClaimsTransformation Id="ConvertRolesToStringCollection" TransformationMethod="StringSplit">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="rolesCommaDelimiter" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
  <InputParameter DataType="string" Id="delimiter" Value="," />
    </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="roles" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **inputClaim** : "Admin,Author,Reader"
- Paramètres d’entrée :
  - **delimiter** : ","
- Revendications de sortie :
  - **outputClaim** : [ "Admin", "Author", "Reader" ]

## <a name="string-claim-transformations-expressions"></a>Expressions de transformations de revendications de chaînes
Les expressions de transformations de revendications dans les stratégies personnalisées Azure AD B2C fournissent des informations contextuelles sur l’ID du locataire et l’ID du profil technique.

  | Expression | Description | Exemple |
 | ----- | ----------- | --------|
 | `{TechnicalProfileId}` | Nom de l’ID du profil technique. | Facebook-OAUTH |
 | `{RelyingPartyTenantId}` | ID de locataire de la stratégie de partie de confiance. | votre-locataire.onmicrosoft.com |
 | `{TrustFrameworkTenantId}` | ID de locataire de l’infrastructure de confiance. | votre-locataire.onmicrosoft.com |
