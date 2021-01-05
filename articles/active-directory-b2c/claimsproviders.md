---
title: ClaimsProviders - Azure Active Directory B2C | Microsoft Docs
description: Spécifiez l’élément ClaimsProvider d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 01/29/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 1d1928de8c9731b54966e566d4dddf9c01073d41
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85201257"
---
# <a name="claimsproviders"></a>ClaimsProviders

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Un fournisseur de revendications contient un ensemble de [profils techniques](technicalprofiles.md). Chaque fournisseur de revendications doit avoir un ou plusieurs profils techniques déterminant les points de terminaison et les protocoles nécessaires pour communiquer avec lui. Un fournisseur de revendications peut avoir plusieurs profils techniques. Par exemple, plusieurs profils techniques peuvent être définis, car le fournisseur de revendications prend en charge plusieurs protocoles, divers points de terminaison avec des fonctionnalités distinctes ou publie des revendications différentes à divers niveaux d’assurance. Il peut être acceptable de publier des revendications sensibles dans un parcours utilisateur, mais pas dans un autre.

```xml
<ClaimsProviders>
  <ClaimsProvider>
    <Domain>Domain name</Domain>
    <DisplayName>Display name</DisplayName>
    <TechnicalProfiles>
      </TechnicalProfile>
        ...
      </TechnicalProfile>
        ...
    </TechnicalProfiles>
  </ClaimsProvider>
  ...
</ClaimsProviders>
```

L’élément **ClaimsProviders** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| ClaimsProvider | 1:n | Un fournisseur de revendications agréé qui peut être utilisé dans plusieurs parcours utilisateur. |

## <a name="claimsprovider"></a>ClaimsProvider

L’élément **ClaimsProvider** contient les éléments enfant suivants :

| Élément | Occurrences | Description |
| ------- | ---------- | ----------- |
| Domain | 0:1 | Une chaîne qui contient le nom de domaine pour le fournisseur de revendications. Par exemple, si votre fournisseur de revendications inclut le profil technique de Facebook, le nom de domaine est Facebook.com. Ce nom de domaine est utilisé pour tous les profils techniques définis dans le fournisseur de revendications, sauf substitution par le profil technique. Le nom de domaine peut également être référencé dans un **domain_hint**. Pour plus d’informations, consultez la section **Rediriger la connexion vers un fournisseur social** dans [Configurer une connexion directe à l’aide d’Azure Active Directory B2C](direct-signin.md). |
| DisplayName | 1:1 | Chaîne qui contient le nom du fournisseur de revendications. |
| [TechnicalProfiles](technicalprofiles.md) | 0:1 | Un ensemble de profils techniques pris en charge par le fournisseur de revendications |

**ClaimsProvider** organise le rapport entre vos profils techniques et le fournisseur de revendications. L’exemple suivant montre le fournisseur de revendications Azure Active Directory avec les profils techniques Azure Active Directory :

```xml
<ClaimsProvider>
  <DisplayName>Azure Active Directory</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="AAD-Common">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId-NoError">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
      ...
    </TechnicalProfile>
      ...
    <TechnicalProfile Id="AAD-UserWritePasswordUsingObjectId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWriteProfileUsingObjectId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingObjectId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWritePhoneNumberUsingObjectId">
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

L’exemple suivant montre le fournisseur de revendications Facebook avec le profil technique **Facebook-OAUTH**.

```xml
<ClaimsProvider>
  <Domain>facebook.com</Domain>
  <DisplayName>Facebook</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Facebook-OAUTH">
      <DisplayName>Facebook</DisplayName>
      <Protocol Name="OAuth2" />
        ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```
