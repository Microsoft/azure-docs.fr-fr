---
title: ClaimsTransformations - Azure Active Directory B2C | Microsoft Docs
description: Définition de l’élément ClaimsTransformations dans le schéma Infrastructure d’expérience d’identité d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 37d9bd78a80ac52d2a790537bf47e33807720349
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85202957"
---
# <a name="claimstransformations"></a>ClaimsTransformations

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

L’élément **ClaimsTransformations** contient une liste des fonctions de transformation de revendication utilisables dans des parcours utilisateur dans le cadre d’une [stratégie personnalisée](custom-policy-overview.md). Une transformation de revendication convertit une revendication donnée en une autre. Dans la transformation de revendication, vous spécifiez la méthode de transformation, par exemple l’ajout d’un élément à une collection de chaînes ou le changement de la casse d’une chaîne.

Pour inclure la liste des fonctions de transformation de revendication pouvant être utilisées dans les parcours utilisateur, vous devez déclarer un élément XML ClaimsTransformations sous la section BuildingBlocks de la stratégie.

```xml
<ClaimsTransformations>
  <ClaimsTransformation Id="<identifier>" TransformationMethod="<method>">
    ...
  </ClaimsTransformation>
</ClaimsTransformations>
```

L’élément **ClaimsTransformation** contient les attributs suivants :

| Attribut |Obligatoire | Description |
| --------- |-------- | ----------- |
| Id |Oui | Identificateur servant à identifier de façon unique la transformation de revendication. L’identificateur est référencé à partir d’autres éléments XML dans la stratégie. |
| Méthode de transformation | Oui | Méthode de transformation à utiliser dans la transformation de revendication. Chaque transformation de revendication a ses propres valeurs. Pour obtenir la liste complète des valeurs disponibles, consultez la [référence des transformations de revendications](#claims-transformations-reference). |

## <a name="claimstransformation"></a>ClaimsTransformation

L’élément **ClaimsTransformation** contient les éléments suivants :

```xml
<ClaimsTransformation Id="<identifier>" TransformationMethod="<method>">
  <InputClaims>
    ...
  </InputClaims>
  <InputParameters>
    ...
  </InputParameters>
  <OutputClaims>
    ...
  </OutputClaims>
</ClaimsTransformation>
```


| Élément | Occurrences | Description |
| ------- | -------- | ----------- |
| InputClaims | 0:1 | Liste d’éléments **InputClaim** qui spécifient les types de revendications pris comme entrées de la transformation de revendication. Chacun de ces éléments contient une référence à un ClaimType déjà défini dans la section ClaimsSchema de la stratégie. |
| InputParameters | 0:1 | Liste d’éléments **InputParameter** qui sont fournis comme entrée de la transformation de revendication.
| OutputClaims | 0:1 | Liste d’éléments **OutputClaim** qui spécifient les types de revendications générés après l’appel de la ClaimsTransformation. Chacun de ces éléments contient une référence à un ClaimType déjà défini dans la section ClaimsSchema. |

### <a name="inputclaims"></a>InputClaims

L’élément **InputClaims** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| InputClaim | 1:n | Type de revendication d’entrée attendu. |

#### <a name="inputclaim"></a>InputClaim

L’élément **InputClaim** contient les attributs suivants :

| Attribut |Obligatoire | Description |
| --------- | ----------- | ----------- |
| ClaimTypeReferenceId |Oui | Référence à un ClaimType déjà défini dans la section ClaimsSchema de la stratégie. |
| TransformationClaimType |Oui | Identificateur servant à référencer un type de transformation de revendication. Chaque transformation de revendication a ses propres valeurs. Pour obtenir la liste complète des valeurs disponibles, consultez la [référence des transformations de revendications](#claims-transformations-reference). |

### <a name="inputparameters"></a>InputParameters

L’élément **InputParameters** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| InputParameter | 1:n | Paramètre d’entrée attendu. |

#### <a name="inputparameter"></a>InputParameter

| Attribut | Obligatoire |Description |
| --------- | ----------- |----------- |
| Id | Oui | Identificateur qui est une référence à un paramètre de la méthode de transformation de revendication. Chaque méthode de transformation de revendication a ses propres valeurs. Pour obtenir une liste complète des valeurs disponibles, consultez le tableau de transformations de revendications. |
| DataType | Oui | Type de données du paramètre, tel que String, Boolean, Int ou DateTime conformément à l’énumération DataType dans le schéma XML de stratégie personnalisée. Ce type est utilisé pour effectuer des opérations arithmétiques correctement. Chaque transformation de revendication a ses propres valeurs. Pour obtenir la liste complète des valeurs disponibles, consultez la [référence des transformations de revendications](#claims-transformations-reference). |
| Valeur | Oui | Valeur passée telle quelle à la transformation. Certaines valeurs sont arbitraires, et vous en sélectionnez certaines à partir de la méthode de transformation de revendication. |

### <a name="outputclaims"></a>OutputClaims

L’élément **OutputClaims** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| OutputClaim | 0:n | Type de revendication de sortie attendu. |

#### <a name="outputclaim"></a>OutputClaim

L’élément **OutputClaim** contient les attributs suivants :

| Attribut |Obligatoire | Description |
| --------- | ----------- |----------- |
| ClaimTypeReferenceId | Oui | Référence à un ClaimType déjà défini dans la section ClaimsSchema de la stratégie.
| TransformationClaimType | Oui | Identificateur servant à référencer un type de transformation de revendication. Chaque transformation de revendication a ses propres valeurs. Pour obtenir la liste complète des valeurs disponibles, consultez la [référence des transformations de revendications](#claims-transformations-reference). |

Si la revendication d’entrée et la revendication de sortie sont du même type (chaîne ou booléen), vous pouvez utiliser la même revendication d’entrée que la revendication de sortie. Dans ce cas, la transformation de revendication change la revendication d’entrée avec la valeur de sortie.

## <a name="example"></a>Exemple

Par exemple, vous pouvez stocker la dernière version de vos conditions d’utilisation du service que l’utilisateur a acceptée. Quand vous mettez à jour les conditions d’utilisation du service, vous pouvez demander à l’utilisateur d’accepter la nouvelle version. Dans l’exemple suivant, la transformation de revendication **HasTOSVersionChanged** compare la valeur de la revendication **TOSVersion** à la valeur de la revendication **LastTOSAcceptedVersion**, puis retourne la revendication **TOSVersionChanged** booléenne.

```xml
<BuildingBlocks>
  <ClaimsSchema>
    <ClaimType Id="TOSVersionChanged">
      <DisplayName>Indicates if the TOS version accepted by the end user is equal to the current version</DisplayName>
      <DataType>boolean</DataType>
    </ClaimType>
    <ClaimType Id="TOSVersion">
      <DisplayName>TOS version</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
    <ClaimType Id="LastTOSAcceptedVersion">
      <DisplayName>TOS version accepted by the end user</DisplayName>
      <DataType>string</DataType>
    </ClaimType>
  </ClaimsSchema>

  <ClaimsTransformations>
    <ClaimsTransformation Id="HasTOSVersionChanged" TransformationMethod="CompareClaims">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="TOSVersion" TransformationClaimType="inputClaim1" />
        <InputClaim ClaimTypeReferenceId="LastTOSAcceptedVersion" TransformationClaimType="inputClaim2" />
      </InputClaims>
      <InputParameters>
        <InputParameter Id="operator" DataType="string" Value="NOT EQUAL" />
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="TOSVersionChanged" TransformationClaimType="outputClaim" />
      </OutputClaims>
    </ClaimsTransformation>
  </ClaimsTransformations>
</BuildingBlocks>
```

## <a name="claims-transformations-reference"></a>Référence des transformations de revendications

Pour obtenir des exemples de transformations de revendications, consultez les pages de référence suivantes :

- [Booléen](boolean-transformations.md)
- [Date](date-transformations.md)
- [Integer](integer-transformations.md)
- [JSON](json-transformations.md)
- [Numéro de téléphone](phone-number-claims-transformations.md)
- [Généralités](general-transformations.md)
- [Compte social](social-transformations.md)
- [Chaîne](string-transformations.md)
- [StringCollection](stringcollection-transformations.md)

