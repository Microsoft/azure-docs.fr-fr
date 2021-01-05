---
title: TrustFrameworkPolicy - Azure Active Directory B2C | Microsoft Docs
description: Spécifiez l’élément TrustFrameworkPolicy d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 01/31/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 354c6f9710b7cbd70e0631bc973b2482ea8d8bb3
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97386882"
---
# <a name="trustframeworkpolicy"></a>TrustFrameworkPolicy

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Une stratégie personnalisée est représentée par un ou plusieurs fichiers au format XML qui se font mutuellement référence dans une chaîne hiérarchique. Les éléments XML définissent des éléments de la stratégie tels que les schéma de revendications, transformations de revendications, définitions de contenu, fournisseurs de revendications, profils techniques, parcours utilisateur et étapes d’orchestration. Chaque fichier de stratégie est défini dans l’élément **TrustFrameworkPolicy** de niveau supérieur d’un fichier de stratégie.

```xml
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="mytenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
  ...
```


L’élément **TrustFrameworkPolicy** contient les attributs suivants :

| Attribut | Obligatoire | Description |
|---------- | -------- | ----------- |
| PolicySchemaVersion | Oui | Version du schéma à utiliser pour exécuter la stratégie. La valeur doit être `0.3.0.0`. |
| TenantObjectId | Non | Identificateur d’objet unique du locataire Azure Active Directory B2C (Azure AD B2C). |
| TenantId | Oui | Identificateur unique du locataire auquel appartient cette stratégie. |
| PolicyId | Oui | Identificateur unique de la stratégie. Cet identificateur doit avoir le préfixe *B2C_1A_* . |
| PublicPolicyUri | Oui | URI de la stratégie, qui est une combinaison de l’ID de locataire et de l’ID de stratégie. |
| DeploymentMode | Non | Valeurs possibles : `Production` ou `Development`. `Production` est la valeur par défaut. Cette propriété permet de déboguer votre stratégie. Pour plus d’informations, voir [Collecte de journaux d’activité](troubleshoot-with-application-insights.md). |
| UserJourneyRecorderEndpoint | Non | Point de terminaison utilisé quand **DeploymentMode** est défini sur `Development`. La valeur doit être `urn:journeyrecorder:applicationinsights`. Pour plus d’informations, voir [Collecte de journaux d’activité](troubleshoot-with-application-insights.md). |


L’exemple suivant montre comment spécifier l’élément **TrustFrameworkPolicy** :

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="https://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="mytenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkBase"
   PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

L’élément **TrustFrameworkPolicy** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| BasePolicy| 0:1| Identificateur d’une stratégie de base. |
| [BuildingBlocks](buildingblocks.md) | 0:1 | Blocs de construction de votre stratégie. |
| [ClaimsProviders](claimsproviders.md) | 0:1 | Collection de fournisseurs de revendications. |
| [UserJourneys](userjourneys.md) | 0:1 | Collection de parcours utilisateur. |
| [RelyingParty](relyingparty.md) | 0:1 | Définition d’une stratégie de partie de confiance. |

Pour hériter d’une autre stratégie, un élément **BasePolicy** doit être déclaré sous l’élément **TrustFrameworkPolicy** du fichier de stratégie. L’élément **BasePolicy** est une référence à la stratégie de base dont cette stratégie est dérivée.

L’élément **BasePolicy** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | --------|
| TenantId | 1:1 | Identificateur de votre locataire Azure AD B2C. |
| PolicyId | 1:1 | Identificateur de la stratégie parente. |


L’exemple suivant montre comment spécifier une stratégie de base. Cette stratégie **B2C_1A_TrustFrameworkExtensions** est dérivée de la stratégie **B2C_1A_TrustFrameworkBase**.

``` XML
<TrustFrameworkPolicy
   xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
   xmlns:xsd="https://www.w3.org/2001/XMLSchema"
   xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
   PolicySchemaVersion="0.3.0.0"
   TenantId="mytenant.onmicrosoft.com"
   PolicyId="B2C_1A_TrustFrameworkExtensions"
   PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkExtensions">

  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkBase</PolicyId>
  </BasePolicy>
  ...
</TrustFrameworkPolicy>
```

