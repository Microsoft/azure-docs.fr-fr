---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/20/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 13169635a235816b79306f73a826dea944549d07
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94988666"
---
## <a name="azure-security-benchmark"></a>Benchmark de sécurité Azure

Le [benchmark de sécurité Azure](../../../../articles/security/benchmarks/overview.md) fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Pour voir comment ce service est entièrement mappé au benchmark de sécurité Azure, consultez les [fichiers de mappage du benchmark de sécurité Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Pour passer en revue la façon dont les composants intégrés Azure Policy disponibles pour tous les services Azure sont mappés à ce standard de conformité, consultez [Conformité réglementaire Azure Policy - Benchmark de sécurité Azure](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protection des données |4.6 |Utiliser Azure RBAC pour contrôler l’accès aux ressources |[Auditer l’utilisation de règles personnalisées RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |

## <a name="cis-microsoft-azure-foundations-benchmark"></a>CIS Microsoft Azure Foundations Benchmark

Pour passer en revue la façon dont les composants intégrés Azure Policy disponibles pour tous les services Azure sont mappés à ce standard de conformité, consultez [Conformité réglementaire Azure Policy - CIS Microsoft Azure Foundations Benchmark 1.1.0](../../../../articles/governance/policy/samples/cis-azure-1-1-0.md).
Pour plus d’informations sur cette norme de conformité, consultez [CIS Microsoft Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure/).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Gestion de l’identité et de l’accès |1.23 |Vérifier qu’aucun rôle de propriétaire d’abonnement personnalisé n’est créé |[Les rôles de propriétaire d’abonnement personnalisé ne doivent pas exister](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

## <a name="hipaa-hitrust-92"></a>HIPAA HITRUST 9.2

Pour voir comment les composants intégrés Azure Policy disponibles pour tous les services Azure correspondent à ce standard de conformité, consultez [Conformité réglementaire Azure Policy – HIPAA HITRUST 9.2](../../../../articles/governance/policy/samples/hipaa-hitrust-9-2.md).
Pour plus d’informations sur cette norme de conformité, consultez [HIPAA HITRUST 9.2](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Gestion des privilèges |1148.01c2System.78 - 01.c |L’organisation restreint l’accès aux fonctions privilégiées et à toutes les informations relatives à la sécurité. |[Auditer l’utilisation de règles personnalisées RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |
|Répartition des tâches |1230.09c2Organizational.1 - 09.c |Aucune personne ne peut accéder, modifier ou utiliser des systèmes d’information sans autorisation ou détection. |[Auditer l’utilisation de règles personnalisées RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |
|Répartition des tâches |1276.09c2Organizational.2 - 09.c |Les activités d’audit de sécurité sont indépendantes. |[Les rôles de propriétaire d’abonnement personnalisé ne doivent pas exister](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |
|Répartition des tâches |1278.09c2Organizational.56 - 09.c |L’organisation identifie les tâches qui doivent être séparées et définit les autorisations d’accès au système d’information pour la prise en charge de la séparation des tâches ; et les tâches incompatibles sont réparties entre plusieurs utilisateurs pour réduire le risque de fraude ou d’utilisation incorrecte. |[Les rôles de propriétaire d’abonnement personnalisé ne doivent pas exister](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

## <a name="nist-sp-800-53-r4"></a>NIST SP 800-53 R4

Pour voir comment les composants intégrés Azure Policy disponibles pour tous les services Azure correspondent à ce standard de conformité, consultez [Conformité réglementaire Azure Policy – NIST SP 800-53 R4](../../../../articles/governance/policy/samples/nist-sp-800-53-r4.md).
Pour plus d’informations sur cette norme de conformité, consultez [NIST SP 800-53 R4](https://nvd.nist.gov/800-53).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Contrôle d’accès |AC-2 (7) |Gestion des comptes \| Schémas basés sur des rôles |[Auditer l’utilisation de règles personnalisées RBAC](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |

