---
title: Définir un profil technique de facteur téléphone dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définissez un profil technique de facteur téléphone dans une stratégie personnalisée d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/12/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 322e4b78fbfb38f1822fb7a7cdcdbfcc0738b303
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91950395"
---
# <a name="define-a-phone-factor-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique de facteur téléphone dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge l’inscription et la vérification de numéros de téléphone. Ce profil technique :

- Fournit une interface utilisateur qui permet d’interagir avec l’utilisateur pour vérifier ou inscrire un numéro de téléphone.
- Prend en charge les appels téléphoniques et les SMS pour valider le numéro de téléphone.
- Prend en charge plusieurs numéros de téléphone. L’utilisateur peut sélectionner un des numéros de téléphone pour la vérification.  
- Retourne une revendication indiquant si l’utilisateur a fourni un nouveau numéro de téléphone. Vous pouvez utiliser cette revendication pour décider si le numéro de téléphone doit être rendu persistant dans le profil utilisateur Azure AD B2C.  
- Utilise la [définition de contenu](contentdefinitions.md) pour contrôler l’aspect.

## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `Proprietary`. L’attribut **handler** doit contenir le nom complet de l’assembly de gestionnaire de protocole qui est utilisé par Azure AD B2C pour le facteur téléphone : `Web.TPEngine.Providers.PhoneFactorProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`

L’exemple suivant montre un profil technique de facteur téléphone pour l’inscription et la validation :

```xml
<TechnicalProfile Id="PhoneFactor-InputOrVerify">
  <DisplayName>PhoneFactor</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.PhoneFactorProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

## <a name="input-claims-transformations"></a>Transformations de revendications d’entrée

L’élément InputClaimsTransformations peut contenir une collection de transformations de revendications d’entrée qui sont utilisées pour modifier les revendications d’entrée ou en générer de nouvelles. La transformation de revendications d’entrée suivante génère une revendication `UserId` utilisée ultérieurement dans la collection de revendications d’entrée.

```xml
<InputClaimsTransformations>
  <InputClaimsTransformation ReferenceId="CreateUserIdForMFA" />
</InputClaimsTransformations>
```

## <a name="input-claims"></a>Revendications d’entrée

L’élément InputClaims doit contenir les revendications suivantes. Vous pouvez également mapper le nom de votre revendication au nom défini dans le profil technique du facteur téléphone. 

|  Type de données| Obligatoire | Description |
| --------- | -------- | ----------- | 
| string| Oui | Identificateur unique de l’utilisateur. Le nom de la revendication ou PartnerClaimType doit être défini sur `UserId`. Cette revendication ne doit pas contenir d’informations d’identification personnelle.|
| string| Oui | Liste des types de revendication. Chaque revendication contient un numéro de téléphone. Si l’une des revendications d’entrée ne contient pas de numéro de téléphone, l’utilisateur est invité à inscrire et à vérifier un nouveau numéro de téléphone. Le numéro de téléphone validé est retourné en tant que revendication de sortie. Si l’une des revendications d’entrée contient un numéro de téléphone, l’utilisateur est invité à le vérifier. Si plusieurs revendications d’entrée contiennent un numéro de téléphone, l’utilisateur est invité à choisir et à vérifier l’un des numéros de téléphone. |

L’exemple suivant illustre l’utilisation de plusieurs numéros de téléphone. Pour plus d’informations, consultez [Exemple de stratégie](https://github.com/azure-ad-b2c/samples/tree/master/policies/mfa-add-secondarymfa).

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="userIdForMFA" PartnerClaimType="UserId" />
  <InputClaim ClaimTypeReferenceId="strongAuthenticationPhoneNumber" />
  <InputClaim ClaimTypeReferenceId="secondaryStrongAuthenticationPhoneNumber" />
</InputClaims>
```

## <a name="output-claims"></a>Revendications de sortie

L’élément OutputClaims contient une liste de revendications retournées par le profil technique du facteur téléphone.

|  Type de données| Obligatoire | Description |
|  -------- | ----------- |----------- |
| boolean | Oui | Indique si le nouveau numéro de téléphone a été entré par l’utilisateur. Le nom de la revendication ou PartnerClaimType doit être défini sur `newPhoneNumberEntered`.|
| string| Oui | Numéro de téléphone vérifié. Le nom de la revendication ou PartnerClaimType doit être défini sur `Verified.OfficePhone`.|

L’élément OutputClaimsTransformations peut contenir une collection d’éléments OutputClaimsTransformation qui sont utilisés pour modifier les revendications de sortie ou en générer de nouvelles.

## <a name="cryptographic-keys"></a>Clés de chiffrement

L’élément **CryptographicKeys** n’est pas utilisé.


## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ContentDefinitionReferenceId | Oui | Identificateur de la [définition de contenu](contentdefinitions.md) associée à ce profil technique. |
| ManualPhoneNumberEntryAllowed| Non | Indique si un utilisateur est autorisé ou non à entrer manuellement un numéro de téléphone. Valeurs possibles : `true` ou `false` (par défaut).|
| setting.authenticationMode | Non | Méthode permettant de valider le numéro de téléphone. Valeurs possibles : `sms`, `phone` ou `mixed` (par défaut).|
| setting.autodial| Non| Indique si le profil technique doit composer automatiquement le numéro ou envoyer automatiquement un SMS. Valeurs possibles : `true` ou `false` (par défaut). La numérotation automatique nécessite que les métadonnées `setting.authenticationMode` soient définies sur `sms` ou `phone`. La collection de revendications d’entrée doit avoir un numéro de téléphone unique. |

### <a name="ui-elements"></a>Éléments d’interface utilisateur

Les éléments d’interface utilisateur de la page d’authentification par facteur téléphone peuvent être [localisés](localization-string-ids.md#phone-factor-authentication-page-user-interface-elements).

## <a name="next-steps"></a>Étapes suivantes

- Vérifiez le pack de démarrage des [comptes locaux et de réseaux sociaux avec MFA](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccountsWithMfa).
