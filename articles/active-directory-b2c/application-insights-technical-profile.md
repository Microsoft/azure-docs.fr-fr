---
title: Définir un profil technique Application Insights dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Définissez un profil technique Application Insights dans une stratégie personnalisée d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/20/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 77bb53e2605913fcee6999284acb04616efc53af
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85201410"
---
# <a name="define-an-application-insights-technical-profile-in-an-azure-ad-b2c-custom-policy"></a>Définir un profil technique Application Insights dans une stratégie personnalisée Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge l’envoi de données d’événement directement à [Application Insights](../azure-monitor/app/app-insights-overview.md) à l’aide de la clé d’instrumentation fournie à Azure AD B2C.  Avec un profil technique Application Insights, vous pouvez obtenir des journaux d’événements détaillés et personnalisés permettant à vos parcours utilisateur d’effectuer ce qui suit :

* Obtenir des informations détaillées sur le comportement des utilisateurs
* Résoudre les problèmes de vos propres stratégies en développement ou en production
* Mesurer les performances
* Créer des notifications à partir d’Application Insights


## <a name="protocol"></a>Protocol

L’attribut **Name** de l’élément **Protocol** doit être défini sur `Proprietary`. L’attribut **handler** doit contenir le nom complet de l’assembly de gestionnaire de protocole qui est utilisé par Azure AD B2C pour Application Insights : `Web.TPEngine.Providers.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`.

L’exemple suivant montre le profil technique Application Insights habituel. D’autres profils techniques Application Insights incluent AzureInsights-Common pour tirer parti de cette configuration.  

```xml
<TechnicalProfile Id="AzureInsights-Common">
  <DisplayName>Azure Insights Common</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.Insights.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

## <a name="input-claims"></a>Revendications d’entrée

L’élément **InputClaims** contient une liste des revendications à envoyer à Application Insights. Vous pouvez également mapper le nom de votre revendication à un nom que vous préférez afficher dans Application Insights. L’exemple suivant montre comment envoyer des télémétries à Application Insights. Les propriétés d’un événement sont ajoutées via la syntaxe `{property:NAME}`, où NAME représente la propriété ajoutée à l’événement. DefaultValue peut être une valeur statique ou une valeur résolue par l’un des [programmes de résolution des revendications](claim-resolver-overview.md) pris en charge.

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="PolicyId" PartnerClaimType="{property:Policy}" DefaultValue="{Policy:PolicyId}" />
  <InputClaim ClaimTypeReferenceId="CorrelationId" PartnerClaimType="{property:JourneyId}" DefaultValue="{Context:CorrelationId}" />
  <InputClaim ClaimTypeReferenceId="Culture" PartnerClaimType="{property:Culture}" DefaultValue="{Culture:RFC5646}" />
  <InputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="{property:objectId}"  />
</InputClaims>
```

L’élément **InputClaimsTransformations** peut contenir une collection d’éléments **InputClaimsTransformation** qui sont utilisés pour modifier les revendications d’entrée ou en générer de nouvelles avant leur envoi à Application Insights.

## <a name="persist-claims"></a>Conserver les revendications

L’élément PersistedClaims n’est pas utilisé.

## <a name="output-claims"></a>Revendications de sortie

Les éléments OutputClaims et OutputClaimsTransformations ne sont pas utilisés.

## <a name="cryptographic-keys"></a>Clés de chiffrement

L’élément CryptographicKeys n’est pas utilisé.


## <a name="metadata"></a>Métadonnées

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| InstrumentationKey| Oui | [Clé d’instrumentation](../azure-monitor/app/create-new-resource.md#copy-the-instrumentation-key) Application Insights qui sera utilisée pour la journalisation des événements. | 
| DeveloperMode| Non | Valeur booléenne indiquant si le mode développeur est activé. Valeurs possibles : `true` ou `false` (par défaut). Ces métadonnées contrôlent la façon dont les événements sont mis en mémoire tampon. Dans un environnement de développement avec un volume minimal d’événements, l’activation du mode développeur entraîne l’envoi immédiat des événements à Application Insights.|  
|DisableTelemetry |Non |Valeur booléenne indiquant si la télémétrie doit être activée ou non. Valeurs possibles : `true` ou `false` (par défaut).| 


## <a name="next-steps"></a>Étapes suivantes

- [Création d’une ressource Application Insights](../azure-monitor/app/create-new-resource.md)
- Découvrez comment [suivre le comportement des utilisateurs dans Azure Active Directory B2C à l’aide d’Application Insights](analytics-with-application-insights.md)
