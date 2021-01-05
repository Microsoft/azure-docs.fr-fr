---
title: Créer et modifier des stratégies de sécurité Azure Policy à l’aide de l’API REST
description: Découvrez comment gérer des stratégies Azure Policy à l’aide d’une API REST.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 11/04/2019
ms.author: memildin
ms.openlocfilehash: 6fcfed585aece4fd57c085e0c9f6fdcd6cbeae74
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91448351"
---
# <a name="configure-a-security-policy-in-azure-policy-using-the-rest-api"></a>Configurer une stratégie de sécurité dans Azure Policy à l’aide de l’API REST

Dans le cadre de l’intégration native à Azure Policy, Azure Security Center vous permet de tirer parti de l’API REST d’Azure Policy pour créer des affectations de stratégie. Les instructions suivantes vous guident tout au long de la création d’affectations de stratégies, ainsi que de la personnalisation d’affectations existantes. 

Concepts importants utilisés dans Azure Policy : 

- Une **définition de stratégie** est une règle 

- Une **initiative** est une collection de définitions de stratégie (règles) 

- Une **affectation** est l’application d’une initiative ou d’une stratégie à une étendue spécifique (groupe d’administration, abonnement, etc.) 

Security Center dispose d’une initiative intégrée qui inclut toutes ses stratégies de sécurité. Pour évaluer les stratégies de Security Center sur vos ressources Azure, vous devez créer une affectation sur le groupe d’administration ou un abonnement que vous voulez évaluer.

L’initiative intégrée a toutes les stratégies de Security Center activées par défaut. Vous pouvez choisir de désactiver certaines stratégies de l’initiative intégrée. Par exemple, pour appliquer toutes les stratégies de Security Center à l’exception de **Pare-feu d’application web**, changez la valeur du paramètre d’effet de la stratégie à **Désactivé**. 

## <a name="api-examples"></a>Exemples d'API

Dans les exemples suivants, remplacez les variables suivantes :

- **{scope}** Entrez le nom du groupe d’administration ou de l’abonnement auquel vous appliquez la stratégie.
- **{policyAssignmentName}** Entrez le [nom de l’affectation de stratégie appropriée](#policy-names).
- **{name}** Entrez votre nom ou le nom de l’administrateur qui a approuvé le changement de stratégie.

Cet exemple vous montre comment affecter l’initiative Security Center intégrée sur un abonnement ou un groupe d’administration :
 
 ```
    PUT  
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 

    Request Body (JSON) 

    { 

      "properties":{ 

    "displayName":"Enable Monitoring in Azure Security Center", 

    "metadata":{ 

    "assignedBy":"{Name}" 

    }, 

    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 

    "parameters":{}, 

    } 

    } 
 ```

Cet exemple vous montre comment affecter l’initiative Security Center intégrée sur un abonnement, avec les stratégies suivantes désactivées : 

- Mises à jour système (“systemUpdatesMonitoringEffect”) 

- Configurations de sécurité (« systemConfigurationsMonitoringEffect ») 

- Protection du point de terminaison (« endpointProtectionMonitoringEffect ») 

 ```
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
    
    Request Body (JSON) 
    
    { 
    
      "properties":{ 
    
    "displayName":"Enable Monitoring in Azure Security Center", 
    
    "metadata":{ 
    
    "assignedBy":"{Name}" 
    
    }, 
    
    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 
    
    "parameters":{ 
    
    "systemUpdatesMonitoringEffect":{"value":"Disabled"}, 
    
    "systemConfigurationsMonitoringEffect":{"value":"Disabled"}, 
    
    "endpointProtectionMonitoringEffect":{"value":"Disabled"}, 
    
    }, 
    
     } 
    
    } 
 ```
Cet exemple vous montre comment supprimer une affectation :
 ```
    DELETE   
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
 ```

## <a name="policy-names-reference"></a>Informations de référence sur les noms de stratégies <a name="policy-names"></a>

|Nom de stratégie dans Security Center|Nom de stratégie affiché dans Azure Policy |Nom du paramètre d’effet de la stratégie|
|----|----|----|
|Chiffrement SQL |Superviser les bases de données SQL non chiffrées dans Azure Security Center |sqlEncryptionMonitoringEffect| 
|Audit SQL |Superviser les bases de données SQL non auditées dans Azure Security Center |sqlAuditingMonitoringEffect|
|Mises à jour système |Superviser les mises à jour système manquantes dans Azure Security Center |systemUpdatesMonitoringEffect|
|Chiffrement du stockage |Auditer le chiffrement d’objet blob manquant pour les comptes de stockage |storageEncryptionMonitoringEffect|
|Accès réseau JIT |Superviser l’éventuel accès juste-à-temps (JIT) au réseau dans Azure Security Center |jitNetworkAccessMonitoringEffect |
|Contrôles d’application adaptative |Superviser la configuration éventuelle de listes vertes d’applications dans Azure Security Center |adaptiveApplicationControlsMonitoringEffect|
|Groupes de sécurité réseau |Superviser l’accès réseau permissif dans Azure Security Center |networkSecurityGroupsMonitoringEffect| 
|Configurations de sécurité |Superviser les vulnérabilités du système d’exploitation dans Azure Security Center |systemConfigurationsMonitoringEffect| 
|Protection du point de terminaison |Superviser les agents Endpoint Protection manquants dans Azure Security Center |endpointProtectionMonitoringEffect |
|Chiffrement de disque |Superviser les disques de machine virtuelle non chiffrés dans Azure Security Center |diskEncryptionMonitoringEffect|
|Évaluation des vulnérabilités |Superviser les vulnérabilités de machine virtuelle dans Azure Security Center |vulnerabilityAssessmentMonitoringEffect|
|Pare-feu d’application web |Superviser les applications web non protégées dans Azure Security Center |webApplicationFirewallMonitoringEffect |
|Pare-feu de nouvelle génération |Superviser les points de terminaison réseau non protégés dans Azure Security Center| |


## <a name="next-steps"></a>Étapes suivantes

Pour d’autres informations connexes, consultez les articles suivants : 

- [Stratégies de sécurité personnalisées](custom-security-policies.md)
- [Vue d’ensemble des stratégies de sécurité](tutorial-security-policy.md)