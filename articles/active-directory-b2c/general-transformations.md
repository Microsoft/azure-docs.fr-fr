---
title: Exemples de transformation de revendications générales pour les stratégies personnalisées
titleSuffix: Azure AD B2C
description: Exemples de transformations de revendications générales pour le schéma Identity Experience Framework (IEF) d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/03/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 52831a1907d5ca8d13b0477c909d0d0358873973
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85202218"
---
# <a name="general-claims-transformations"></a>Transformations de revendications générales

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Cet article fournit des exemples pour l’utilisation de transformations de revendications générales du schéma Identity Experience Framework dans Azure Active Directory B2C (Azure AD B2C). Pour plus d’informations, voir [ClaimsTransformations](claimstransformations.md).

## <a name="copyclaim"></a>CopyClaim

Copie la valeur d’une revendication dans une autre. Les deux revendications doivent être du même type.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim | chaîne, int | Type de revendication à copier. |
| OutputClaim | outputClaim | chaîne, int | ClaimType généré après l’appel de cette ClaimsTransformation. |

Utilisez cette transformation de revendications pour copier la valeur d’une revendication de chaîne ou numérique dans une autre revendication. L’exemple suivant copie la valeur de revendication externalEmail dans la revendication d’e-mail.

```xml
<ClaimsTransformation Id="CopyEmailAddress" TransformationMethod="CopyClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="externalEmail" TransformationClaimType="inputClaim"/>
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
    - **inputClaim** : bob@contoso.com
- Revendications de sortie :
    - **outputClaim** : bob@contoso.com

## <a name="doesclaimexist"></a>DoesClaimExist

Vérifie si l’**inputClaim** existe, et définit **outputClaim** sur true ou false en conséquence.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim |Quelconque | Revendication d’entrée dont l’existence doit être vérifiée. |
| OutputClaim | outputClaim | boolean | ClaimType généré après l’appel de cette ClaimsTransformation. |

Utilisez cette transformation de revendication pour vérifier si une revendication existe ou si elle contient une valeur quelconque. La valeur de retour est une valeur booléenne qui indique si la revendication existe. L’exemple suivant vérifie si l’adresse e-mail existe.

```xml
<ClaimsTransformation Id="CheckIfEmailPresent" TransformationMethod="DoesClaimExist">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isEmailPresent" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **inputClaim** : someone@contoso.com
- Revendications de sortie :
  - **outputClaim** : true

## <a name="hash"></a>Hachage

Hache le texte brut fourni à l’aide de la valeur salt et d’un secret. L’algorithme de hachage utilisé est SHA-256.

| Élément | TransformationClaimType | Type de données | Notes |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | plaintext | string | Revendication d’entrée à chiffrer. |
| InputClaim | salt | string | Paramètre salt. Vous pouvez créer une valeur aléatoire à l’aide de la transformation des revendication `CreateRandomString`. |
| InputParameter | randomizerSecret | string | Pointe vers une **clé de stratégie** Azure AD B2C existante. Pour créer une clé de stratégie : Dans votre locataire Azure AD B2C, sous **Gérer**, sélectionnez **Identity Experience Framework**. Sélectionnez **Clés de stratégie** pour afficher les clés qui sont disponibles dans votre locataire. Sélectionnez **Ajouter**. Pour **Options**, sélectionnez **Manuel**. Fournissez un nom (il est possible que le préfixe *B2C_1A_* soit ajouté automatiquement). Dans la zone de texte **Secret**, entrez un secret à utiliser, tel que 1234567890. Pour **Utilisation de la clé**, sélectionnez **Signature**. Sélectionnez **Create** (Créer). |
| OutputClaim | Hachage | string | ClaimType généré après que cette transformation de revendication a été appelée. Revendication configurée dans l’inputClaim `plaintext`. |

```xml
<ClaimsTransformation Id="HashPasswordWithEmail" TransformationMethod="Hash">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="password" TransformationClaimType="plaintext" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="salt" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="randomizerSecret" DataType="string" Value="B2C_1A_AccountTransformSecret" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="hashedPassword" TransformationClaimType="hash" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Exemple

- Revendications d’entrée :
  - **plaintext** : MyPass@word1
  - **salt** : 487624568
  - **randomizerSecret** : B2C_1A_AccountTransformSecret
- Revendications de sortie :
  - **outputClaim** : CdMNb/KTEfsWzh9MR1kQGRZCKjuxGMWhA5YQNihzV6U=
