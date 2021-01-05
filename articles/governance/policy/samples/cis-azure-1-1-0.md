---
title: Détails de la Conformité réglementaire pour CIS Microsoft Azure Foundations Benchmark
description: Détails de l’initiative intégrée Conformité réglementaire de CIS Microsoft Azure Foundations Benchmark. Chaque contrôle est mis en correspondance avec une ou plusieurs définitions Azure Policy qui simplifient l’évaluation.
ms.date: 11/20/2020
ms.topic: sample
ms.custom: generated
ms.openlocfilehash: 2ed1a40ba7589d25864cd5bdd90526635f189d6d
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94991313"
---
# <a name="details-of-the-cis-microsoft-azure-foundations-benchmark-regulatory-compliance-built-in-initiative"></a>Détails de l’initiative intégrée Conformité réglementaire de CIS Microsoft Azure Foundations Benchmark

L’article suivant explique en détail comment la définition de l’initiative intégrée Conformité réglementaire Azure Policy est mappée à des **domaines de conformité** et des **contrôles** dans CIS Microsoft Azure Foundations Benchmark.
Pour plus d’informations sur cette norme de conformité, consultez [CIS Microsoft Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure/). Pour comprendre la _propriété_, consultez [Définition de stratégie Azure Policy](../concepts/definition-structure.md#type) et [Responsabilité partagée dans le cloud](../../../security/fundamentals/shared-responsibility.md).

Les correspondances suivantes sont relatives aux contrôles **CIS Microsoft Azure Foundations Benchmark**. Utilisez le volet de navigation de droite pour accéder directement à un **domaine de conformité** spécifique. De nombreux contrôles sont mis en œuvre avec la définition d’une initiative [Azure Policy](../overview.md). Pour examiner la définition d’initiative complète, ouvrez **Stratégie** dans le Portail Azure et sélectionnez la page **Définitions**.
Ensuite, recherchez et sélectionnez la définition de l’initiative intégrée Conformité réglementaire **CIS Microsoft Azure Foundations Benchmark 1.1.0**.

Cette initiative intégrée est déployée dans le cadre de l’[exemple de blueprint CIS Microsoft Azure Foundations Benchmark](../../blueprints/samples/cis-azure-1-1-0.md).

> [!IMPORTANT]
> Chaque contrôle ci-dessous est associé à une ou plusieurs définitions [Azure Policy](../overview.md).
> Ces stratégies peuvent vous aider à [évaluer la conformité](../how-to/get-compliance-data.md) avec le contrôle ; toutefois, il n’existe pas souvent de correspondance un-à-un ou parfaite entre un contrôle et une ou plusieurs stratégies. Ainsi, la **conformité** dans Azure Policy fait uniquement référence aux définitions de stratégie elles-mêmes ; cela ne garantit pas que vous êtes entièrement conforme à toutes les exigences d’un contrôle. En outre, la norme de conformité comprend des contrôles qui ne sont traités par aucune définition Azure Policy pour l’instant. Par conséquent, la conformité dans Azure Policy n’est qu’une vue partielle de l’état de conformité global. Les associations entre les domaines de conformité, les contrôles et les définitions Azure Policy pour cette norme de conformité peuvent changer au fil du temps. Pour afficher l’historique des changements, consultez l’[historique des validations GitHub](https://github.com/Azure/azure-policy/commits/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance/CISv1_1_0_audit.json).

## <a name="identity-and-access-management"></a>Gestion de l’identité et de l’accès

### <a name="ensure-that-multi-factor-authentication-is-enabled-for-all-privileged-users"></a>Vérifier que l’authentification multifacteur est activée pour tous les utilisateurs privilégiés

**ID** : **Propriété** CIS Azure 1.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |MFA (Multi-Factor Authentication) doit être activé pour tous les comptes de l'abonnement disposant de privilèges d'accès en écriture pour éviter une violation des comptes ou des ressources. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |
|[L’authentification multifacteur (MFA) doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |MFA (Multi-Factor Authentication) doit être activé pour tous les comptes de l'abonnement disposant d'autorisations de propriétaire afin d'éviter une violation des comptes ou des ressources. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="ensure-that-multi-factor-authentication-is-enabled-for-all-non-privileged-users"></a>Vérifier que l’authentification multifacteur est activée pour tous les utilisateurs non privilégiés

**ID** : **Propriété** CIS Azure 1.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’authentification multifacteur (MFA) doit être activée sur les comptes disposant d’autorisations de lecture sur votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |MFA (Multi-Factor Authentication) doit être activé pour tous les comptes de l'abonnement disposant de privilèges d'accès en lecture afin d'éviter une violation des comptes ou des ressources. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="ensure-that-there-are-no-guest-users"></a>Vérifier qu’il n’y a aucun utilisateur invité

**ID** : **Propriété** CIS Azure 1.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8456c1c-aa66-4dfb-861a-25d127b775c9) |Les comptes externes avec des autorisations de type propriétaire doivent être supprimés de votre abonnement pour empêcher un accès non contrôlé. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWithOwnerPermissions_Audit.json) |
|[Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5f76cf89-fbf2-47fd-a3f4-b891fa780b60) |Les comptes externes avec des privilèges d'accès en lecture doivent être supprimés de votre abonnement afin d'empêcher un accès non contrôlé. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsReadPermissions_Audit.json) |
|[Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c607a2e-c700-4744-8254-d77e7c9eb5e4) |Les comptes externes avec des privilèges d'accès en écriture doivent être supprimés de votre abonnement afin d'empêcher un accès non contrôlé. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWritePermissions_Audit.json) |

### <a name="ensure-that-no-custom-subscription-owner-roles-are-created"></a>Vérifier qu’aucun rôle de propriétaire d’abonnement personnalisé n’est créé

**ID** : **Propriété** CIS Azure 1.23 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les rôles de propriétaire d’abonnement personnalisé ne doivent pas exister](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |Cette stratégie garantit qu’il n’existe aucun rôle de propriétaire d’abonnement personnalisé. |Audit, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

## <a name="security-center"></a>Security Center

### <a name="ensure-that-automatic-provisioning-of-monitoring-agent-is-set-to-on"></a>Vérifier que l’option « Provisionnement automatique de l’agent de supervision » est définie sur « On »

**ID** : **Propriété** CIS Azure 2.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le provisionnement automatique de l’agent de supervision Log Analytics doit être activé sur votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |Activer le provisionnement automatique de l’agent de supervision Log Analytics pour collecter les données de sécurité |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |

### <a name="ensure-asc-default-policy-setting-monitor-system-updates-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Surveiller les mises à jour système » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les mises à jour système doivent être installées sur vos machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86b3d65f-7626-441e-b690-81a8b71cff60) |Les mises à jour système de sécurité manquantes sur vos serveurs seront supervisées par Azure Security Center en tant que recommandation |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingSystemUpdates_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-os-vulnerabilities-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Surveiller les vulnérabilités de système d’exploitation » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les vulnérabilités dans la configuration de la sécurité sur vos machines doivent être corrigées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe1e5fd5d-3e4c-4ce1-8661-7d1873ae6b15) |Les serveurs qui ne respectent pas la base de référence configurée seront supervisés par Azure Security Center en tant que recommandation |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OSVulnerabilities_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-endpoint-protection-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Surveiller Endpoint Protection » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.5 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Superviser les agents Endpoint Protection manquants dans Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Les serveurs sans agent Endpoint Protection installé seront supervisés par Azure Security Center en tant que recommandation |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-disk-encryption-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Surveiller le chiffrement des disques » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.6 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le chiffrement de disque doit être appliqué sur les machines virtuelles](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Les machines virtuelles sans chiffrement de disque activé seront supervisées par Azure Security Center en tant que recommandations. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-network-security-groups-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut ASC « Superviser les groupes de sécurité réseau » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.7 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les recommandations de renforcement de réseau adaptatif doivent être appliquées sur les machines virtuelles accessibles à partir d’Internet](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Azure Security Center analyse les tendances du trafic des machines virtuelles accessibles sur Internet et fournit des recommandations sur les règles de groupe de sécurité réseau à utiliser pour réduire la surface d’attaque potentielle |AuditIfNotExists, Désactivé |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |

### <a name="ensure-asc-default-policy-setting-enable-next-generation-firewallngfw-monitoring-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut ASC « Activer la supervision du pare-feu de nouvelle génération » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.9 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les machines virtuelles accessibles à partir d’Internet doivent être protégées avec des groupes de sécurité réseau](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Protégez vos machines virtuelles contre les menaces potentielles en limitant leur accès avec des groupes de sécurité réseau (NSG). Pour en savoir plus sur le contrôle du trafic avec des groupes de sécurité réseau, consultez [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[les sous-réseaux doivent être associés à un groupe de sécurité réseau](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe71308d3-144b-4262-b144-efdc3cc90517) |Protégez votre sous-réseau contre les menaces potentielles en y limitant l’accès avec un groupe de sécurité réseau (NSG). Les NSG contiennent une liste de règles de liste de contrôle d’accès (ACL) qui autorisent ou refusent le trafic réseau vers votre sous-réseau. |AuditIfNotExists, Désactivé |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnSubnets_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-vulnerability-assessment-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Superviser l’évaluation des vulnérabilités » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.10 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F760a85ff-6162-42b3-8d70-698e268f648c) |Surveille les vulnérabilités détectées par la solution d’évaluation des vulnérabilités et les machines virtuelles sans solution d’évaluation des vulnérabilités dans Azure Security Center en tant que recommandation. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VMVulnerabilities_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-jit-network-access-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Surveiller l’accès réseau JIT » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.12 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les ports de gestion des machines virtuelles doivent être protégés par un contrôle d’accès réseau juste-à-temps](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |L’éventuel accès juste-à-temps (JIT) au réseau sera supervisé par Azure Security Center en tant que recommandation |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-adaptive-application-whitelisting-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut ASC « Superviser la mise en liste verte des contrôles d’application adaptatifs » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.13 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les contrôles d’application adaptatifs pour définir les applications sécurisées doivent être activés sur vos machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a6b606-51aa-4496-8bb7-64b11cf66adc) |Activer les contrôles d’application pour définir la liste des applications reconnues fiables qui s’exécutent sur vos machines, et vous avertir quand d’autres applications s’exécutent. La sécurité de vos machines contre les logiciels malveillants s’en trouve renforcée. Pour simplifier le processus de configuration et de maintenance de vos règles, Security Center utilise le Machine Learning pour analyser les applications qui s’exécutent sur chaque machine et suggérer la liste des applications reconnues fiables. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveApplicationControls_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-sql-auditing-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut ASC « Superviser l’audit SQL » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.14 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’audit sur SQL Server doit être activé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |L’audit sur votre serveur SQL Server doit être activé pour suivre les activités de toutes les bases de données du serveur, à l’exception de Synapse, et les enregistrer dans un journal d’audit. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |

### <a name="ensure-asc-default-policy-setting-monitor-sql-encryption-is-not-disabled"></a>Vérifier que le paramètre de stratégie par défaut d’ASC « Surveiller le chiffrement SQL » n’est pas défini sur « Disabled »

**ID** : **Propriété** CIS Azure 2.15 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Transparent Data Encryption sur les bases de données SQL doit être activé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |Le chiffrement transparent des données doit être activé pour protéger les données au repos et respecter les conditions de conformité requises |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

### <a name="ensure-that-security-contact-emails-is-set"></a>Vérifier que l’option « E-mails de contact de sécurité » est définie

**ID** : **Propriété** CIS Azure 2.16 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une adresse e-mail de contact de sécurité doit être fournie pour votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f4f78b8-e367-4b10-a341-d9a4ad5cf1c7) |Entrez une adresse e-mail pour recevoir des notifications quand Azure Security Center détecte des ressources compromises |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Security_contact_email.json) |

### <a name="ensure-that-security-contact-phone-number-is-set"></a>Vérifier que l’option « Numéro de téléphone » de contact de sécurité est définie

**ID** : **Propriété** CIS Azure 2.17 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Un numéro de téléphone de contact de sécurité doit être fourni pour votre abonnement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4d66858-c922-44e3-9566-5cdb7a7be744) |Entrez un numéro de téléphone pour recevoir des notifications quand Azure Security Center détecte des ressources compromises |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Security_contact_phone_number.json) |

### <a name="ensure-that-send-email-notification-for-high-severity-alerts-is-set-to-on"></a>Vérifier que l’option « Envoyer une notification par e-mail pour les alertes à gravité élevée » est définie sur « On »

**ID** : **Propriété** CIS Azure 2.18 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La notification par e-mail pour les alertes à gravité élevée doit être activée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6e2593d9-add6-4083-9c9b-4b7d2188c899) |Activez l’envoi par e-mail des alertes de sécurité au contact de sécurité pour qu’il reçoive des e-mails d’alerte de sécurité de Microsoft. Cela permet de s’assurer que les personnes appropriées sont informées des problèmes de sécurité potentiels et peuvent réduire les risques |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Email_notification.json) |

### <a name="ensure-that-send-email-also-to-subscription-owners-is-set-to-on"></a>Vérifier que l’option « Envoyer également un e-mail aux propriétaires de l’abonnement » est définie sur « On »

**ID** : **Propriété** CIS Azure 2.19 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La notification par e-mail au propriétaire de l’abonnement pour les alertes à gravité élevée doit être activée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b15565f-aa9e-48ba-8619-45960f2c314d) |Activez l’envoi d’alertes de sécurité par e-mail au propriétaire de l’abonnement, afin qu’il reçoive des e-mails d’alerte de sécurité de Microsoft. Cela permet de s’assurer qu’il est informé de tous les problèmes de sécurité potentiels afin de réduire le risque dans le délai imparti |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Email_notification_to_subscription_owner.json) |

## <a name="storage-accounts"></a>Comptes de stockage

### <a name="ensure-that-secure-transfer-required-is-set-to-enabled"></a>Vérifier que l’option « Transfert sécurisé requis » est définie sur « Enabled »

**ID** : **Propriété** CIS Azure 3.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La sécurisation du transfert vers des comptes de stockage doit être activée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Auditer l’exigence de transfert sécurisé dans votre compte de stockage. L’option de sécurisation du transfert oblige votre compte de stockage à accepter uniquement des requêtes provenant de connexions sécurisées (HTTPS). L’utilisation de HTTPS garantit l’authentification entre le serveur et le service et protège les données en transit contre les attaques de la couche réseau (attaque de l’intercepteur ou « man-in-the-middle », écoute clandestine, détournement de session) |Audit, Refuser, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |

### <a name="ensure-default-network-access-rule-for-storage-accounts-is-set-to-deny"></a>Vérifier que la règle d’accès réseau par défaut pour les comptes de stockage est définie sur Refuser

**ID** : **Propriété** CIS Azure 3.7 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les comptes de stockage doivent limiter l’accès réseau](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |L’accès réseau aux comptes de stockage doit être limité. Configurez les règles du réseau de telle manière que seules les applications des réseaux autorisés puissent accéder au compte de stockage. Pour autoriser les connexions de clients Internet ou locaux spécifiques, l’accès au trafic peut être autorisé à partir de réseaux virtuels Azure spécifiques ou vers des plages d’adresses IP Internet publiques |Audit, Refuser, Désactivé |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |

### <a name="ensure-trusted-microsoft-services-is-enabled-for-storage-account-access"></a>Vérifier que l’option « Services Microsoft approuvés » est activée pour l’accès au compte de stockage

**ID** : **Propriété** CIS Azure 3.8 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les comptes de stockage doivent autoriser l’accès à partir des services Microsoft approuvés](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc9d007d0-c057-4772-b18c-01e546713bcd) |Certains services Microsoft qui interagissent avec les comptes de stockage fonctionnent à partir de réseaux qui ne peuvent pas obtenir l’accès par le biais des règles de réseau. Pour que ce type de service fonctionne comme prévu, autorisez l’ensemble des services Microsoft approuvés à contourner les règles de réseau. Ces services utilisent alors une authentification forte pour accéder au compte de stockage. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccess_TrustedMicrosoftServices_Audit.json) |

## <a name="database-services"></a>Services de base de données

### <a name="ensure-that-auditing-is-set-to-on"></a>Vérifier que l’option « Audit » est définie sur « On »

**ID** : **Propriété** CIS Azure 4.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’audit sur SQL Server doit être activé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |L’audit sur votre serveur SQL Server doit être activé pour suivre les activités de toutes les bases de données du serveur, à l’exception de Synapse, et les enregistrer dans un journal d’audit. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |

### <a name="ensure-that-auditactiongroups-in-auditing-policy-for-a-sql-server-is-set-properly"></a>Vérifier que « AuditActionGroups » dans la stratégie « Audit » pour un serveur SQL est correctement défini

**ID** : **Propriété** CIS Azure 4.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les paramètres d’audit SQL doivent avoir des groupes d’actions configurés pour capturer les activités critiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7ff426e2-515f-405a-91c8-4f2333442eb5) |La propriété AuditActionsAndGroups doit contenir au moins SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP, FAILED_DATABASE_AUTHENTICATION_GROUP, BATCH_COMPLETED_GROUP pour assurer la journalisation totale de l’audit |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_ActionsAndGroups_Audit.json) |

### <a name="ensure-that-auditing-retention-is-greater-than-90-days"></a>Vérifier que la période de rétention d’audit est supérieure à 90 jours

**ID** : **Propriété** CIS Azure 4.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les serveurs SQL doivent être configurés avec une période de conservation d’audit supérieure à 90 jours.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F89099bee-89e0-4b26-a5f4-165451757743) |Auditez les serveurs SQL configurés avec une période de conservation d’audit inférieure à 90 jours. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditingRetentionDays_Audit.json) |

### <a name="ensure-that-advanced-data-security-on-a-sql-server-is-set-to-on"></a>Vérifier que l’option « Advanced Data Security » sur un serveur SQL Server est définie sur « On »

**ID** : **Propriété** CIS Azure 4.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Advanced Data Security doit être activé sur SQL Managed Instance](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Auditez chaque instance managée SQL sans Advanced Data Security. |AuditIfNotExists, Désactivé |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[Advanced Data Security doit être activé sur vos serveurs SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Auditer les serveurs SQL sans Advanced Data Security |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |

### <a name="ensure-that-azure-active-directory-admin-is-configured"></a>Vérifier que l’administrateur Azure Active Directory est configuré

**ID** : **Propriété** CIS Azure 4.8 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1f314764-cb73-4fc9-b863-8eca98ac36e9) |Auditer l’approvisionnement d’un administrateur Azure Active Directory pour votre serveur SQL afin d’activer l’authentification Azure AD. L’authentification Azure AD permet une gestion simplifiée des autorisations et une gestion centralisée des utilisateurs de bases de données et d’autres services Microsoft |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SQL_DB_AuditServerADAdmins_Audit.json) |

### <a name="ensure-that-data-encryption-is-set-to-on-on-a-sql-database"></a>Vérifiez que l’option « Chiffrement des données » est définie sur « On » sur une base de données SQL

**ID** : **Propriété** CIS Azure 4.9 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Transparent Data Encryption sur les bases de données SQL doit être activé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |Le chiffrement transparent des données doit être activé pour protéger les données au repos et respecter les conditions de conformité requises |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

### <a name="ensure-sql-servers-tde-protector-is-encrypted-with-byok-use-your-own-key"></a>Vérifier que le protecteur TDE du serveur SQL est chiffré avec BYOK (Bring Your Own Key)

**ID** : **Propriété** CIS Azure 4.10 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le protecteur TDE SQL Managed Instance doit être chiffré avec votre propre clé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F048248b0-55cd-46da-b1ff-39efd52db260) |Transparent Data Encryption (TDE) avec prise en charge de votre propre clé offre les avantages suivants : transparence et contrôle améliorés sur le protecteur TDE, sécurité durcie avec un service externe HSM et promotion de la séparation des tâches. |AuditIfNotExists, Désactivé |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |
|[Le protecteur TDE du serveur SQL doit être chiffré avec votre propre clé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d134df8-db83-46fb-ad72-fe0c9428c8dd) |Transparent Data Encryption (TDE) avec prise en charge de votre propre clé offre les avantages suivants : transparence et contrôle améliorés sur le protecteur TDE, sécurité durcie avec un service externe HSM et promotion de la séparation des tâches. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |

### <a name="ensure-enforce-ssl-connection-is-set-to-enabled-for-mysql-database-server"></a>Vérifier que l’option « Appliquer une connexion SSL » est définie sur « ENABLED » pour le serveur de base de données MySQL

**ID** : **Propriété** CIS Azure 4.11 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’application de la connexion SSL doit être activée pour les serveurs de base de données MySQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |La base de données Azure pour MySQL prend en charge la connexion de votre serveur Azure Database pour MySQL aux applications clientes à l’aide de Secure Sockets Layer (SSL). L’application de connexions SSL entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application. Cette configuration garantit que le protocole SSL est toujours activé pour l’accès à votre serveur de base de données. |Audit, Désactivé |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |

### <a name="ensure-server-parameter-log_checkpoints-is-set-to-on-for-postgresql-database-server"></a>Vérifier que le paramètre de serveur « log_checkpoints » est défini sur « ON » pour le serveur de base de données PostgreSQL

**ID** : **Propriété** CIS Azure 4.12 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les points de contrôle de journal doivent être activés pour les serveurs de bases de données PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e43d) |Cette stratégie permet d’auditer toutes les bases de données PostgreSQL de votre environnement sans activer le paramètre log_checkpoints. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogCheckpoint_Audit.json) |

### <a name="ensure-enforce-ssl-connection-is-set-to-enabled-for-postgresql-database-server"></a>Vérifier que l’option « Appliquer une connexion SSL » est définie sur « ENABLED » pour le serveur de base de données PostgreSQL

**ID** : **Propriété** CIS Azure 4.13 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’application de la connexion SSL doit être activée pour les serveurs de base de données PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |Azure Database pour PostgreSQL prend en charge la connexion de votre serveur Azure Database pour PostgreSQL aux applications clientes via SSL (Secure Sockets Layer). L’application de connexions SSL entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application. Cette configuration garantit que le protocole SSL est toujours activé pour l’accès à votre serveur de base de données. |Audit, Désactivé |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |

### <a name="ensure-server-parameter-log_connections-is-set-to-on-for-postgresql-database-server"></a>Vérifier que le paramètre de serveur « log_connections » est défini sur « ON » pour le serveur de base de données PostgreSQL

**ID** : **Propriété** CIS Azure 4.14 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les connexions de journal doivent être activées pour les serveurs de bases de données PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e442) |Cette stratégie permet d’auditer toutes les bases de données PostgreSQL de votre environnement sans activer le paramètre log_connections. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogConnections_Audit.json) |

### <a name="ensure-server-parameter-log_disconnections-is-set-to-on-for-postgresql-database-server"></a>Vérifier que le paramètre de serveur « log_disconnections » est défini sur « ON » pour le serveur de base de données PostgreSQL

**ID** : **Propriété** CIS Azure 4.15 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les déconnexions doivent être journalisées pour les serveurs de bases de données PostgreSQL.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e446) |Cette stratégie permet d’auditer toutes les bases de données PostgreSQL de votre environnement sans activer le paramètre log_disconnections. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogDisconnections_Audit.json) |

### <a name="ensure-server-parameter-connection_throttling-is-set-to-on-for-postgresql-database-server"></a>Vérifier que le paramètre de serveur « connection_throttling » est défini sur « ON » pour le serveur de base de données PostgreSQL

**ID** : **Propriété** CIS Azure 4.17 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La limitation de connexion doit être activée pour les serveurs de base de données PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5345bb39-67dc-4960-a1bf-427e16b9a0bd) |Cette stratégie aide à auditer toutes les bases de données PostgreSQL de votre environnement sans activer la limitation de connexion. Ce paramètre active la limitation de connexion temporaire par adresse IP à la suite d’un trop grand nombre de connexions infructueuses avec des mots de passe non valides. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_ConnectionThrottling_Enabled_Audit.json) |

## <a name="logging-and-monitoring"></a>Journalisation et supervision

### <a name="ensure-that-a-log-profile-exists"></a>Vérifier l’existence d’un profil de journal

**ID** : **Propriété** CIS Azure 5.1.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les abonnements Azure doivent avoir un profil de journal pour le journal d’activité](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7796937f-307b-4598-941c-67d3a05ebfe7) |Cette stratégie garantit l’activation d’un profil de journal pour l’exportation des journaux d’activité. Elle vérifie si aucun profil de journal n’a été créé pour exporter les journaux vers un compte de stockage ou un hub d’événements. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/Logprofile_activityLogs_Audit.json) |

### <a name="ensure-that-activity-log-retention-is-set-365-days-or-greater"></a>Vérifier que la conservation du journal d’activité est définie sur 365 jours ou plus

**ID** : **Propriété** CIS Azure 5.1.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le journal d’activité doit être conservé pendant au moins un an](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb02aacc0-b073-424e-8298-42b22829ee0a) |Cette stratégie audite le journal d’activité si la conservation n’est pas définie sur 365 jours ou sur toujours (jours de conservation définis sur 0). |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLogRetention_365orGreater.json) |

### <a name="ensure-audit-profile-captures-all-the-activities"></a>Vérifier que le profil d’audit capture toutes les activités

**ID** : **Propriété** CIS Azure 5.1.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le profil de journal Azure Monitor doit collecter des journaux pour les catégories « écriture », « suppression » et « action »](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1a4e592a-6a6e-44a5-9814-e36264ca96e7) |Cette stratégie garantit qu’un profil de journal collecte les journaux pour les catégories « write », « delete » et « action » |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllCategories.json) |

### <a name="ensure-the-log-profile-captures-activity-logs-for-all-regions-including-global"></a>Vérifier que le profil de journal capture les journaux d’activité pour toutes les régions, notamment la région globale

**ID** : **Propriété** CIS Azure 5.1.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Monitor doit collecter les journaux d’activité dans toutes les régions](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F41388f1c-2db0-4c25-95b2-35d7f5ccbfa9) |Cette stratégie effectue l’audit du profil de journal Azure Monitor qui n’exporte pas d’activités à partir de toutes les régions Azure prises en charge, notamment la région globale. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_CaptureAllRegions.json) |

### <a name="ensure-the-storage-account-containing-the-container-with-activity-logs-is-encrypted-with-byok-use-your-own-key"></a>Vérifier que le compte de stockage comprenant le conteneur des journaux d’activité est chiffré avec BYOK (Bring Your Own Key)

**ID** : **Propriété** CIS Azure 5.1.6 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le compte de stockage disposant du conteneur des journaux d’activité doit être chiffré avec BYOK](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffbb99e8e-e444-4da0-9ff1-75c92f5a85b2) |Cette stratégie vérifie si le compte de stockage disposant du conteneur des journaux d’activité est chiffré avec BYOK. La stratégie fonctionne uniquement si le compte de stockage se trouve par défaut dans le même abonnement que les journaux d’activité. Vous trouverez des informations complémentaires sur le chiffrement Stockage Azure au repos ici [https://aka.ms/azurestoragebyok](https://aka.ms/azurestoragebyok).  |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_StorageAccountBYOK_Audit.json) |

### <a name="ensure-that-logging-for-azure-keyvault-is-enabled"></a>Vérifier que la journalisation d’Azure KeyVault est définie sur « Enabled »

**ID** : **Propriété** CIS Azure 5.1.7 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les journaux de diagnostic dans Key Vault doivent être activés](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf820ca0-f99e-4f3e-84fb-66e913812d21) |Auditer l’activation des journaux de diagnostic Permet de recréer les pistes d’activité à utiliser à des fins d’investigation en cas d’incident de sécurité ou de compromission du réseau |AuditIfNotExists, Désactivé |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_AuditDiagnosticLog_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-policy-assignment"></a>Vérifier l’existence de l’alerte de journal d’activité pour la création d’attribution de stratégie

**ID** : **Propriété** CIS Azure 5.2.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations de stratégie spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc5447c04-a4d7-4ba8-a263-c9ee321a6858) |Cette stratégie audite toute opération de stratégie spécifique sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_PolicyOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-network-security-group"></a>Vérifier l’existence d’une alerte de journal d’activité pour la création ou la mise à jour d’un groupe de sécurité réseau

**ID** : **Propriété** CIS Azure 5.1.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations d’administration spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Cette stratégie audite des opérations d’administration spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-network-security-group"></a>Vérifier l’existence d’une alerte de journal d’activité pour la suppression de groupe de sécurité réseau

**ID** : **Propriété** CIS Azure 5.2.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations d’administration spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Cette stratégie audite des opérations d’administration spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-network-security-group-rule"></a>Vérifier l’existence d’une alerte de journal d’activité pour la création ou la mise à jour d’une règle de groupe de sécurité réseau

**ID** : **Propriété** CIS Azure 5.2.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations d’administration spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Cette stratégie audite des opérations d’administration spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-the-delete-network-security-group-rule"></a>Vérifier l’existence d’une alerte de journal d’activité pour la suppression d’une règle de groupe de sécurité réseau

**ID** : **Propriété** CIS Azure 5.2.5 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations d’administration spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Cette stratégie audite des opérations d’administration spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-security-solution"></a>Vérifier l’existence d’une alerte de journal d’activité pour la création ou la mise à jour d’une solution de sécurité

**ID** : **Propriété** CIS Azure 5.2.6 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations de sécurité spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Cette stratégie audite des opérations de sécurité spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-security-solution"></a>Vérifier l’existence d’une alerte de journal d’activité pour la suppression d’une solution de sécurité

**ID** : **Propriété** CIS Azure 5.2.7 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations de sécurité spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Cette stratégie audite des opérations de sécurité spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-or-delete-sql-server-firewall-rule"></a>Vérifier l’existence d’une alerte de journal d’activité pour la création, la mise à jour ou la suppression d’une règle de pare-feu SQL Server

**ID** : **Propriété** CIS Azure 5.2.8 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations d’administration spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Cette stratégie audite des opérations d’administration spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-update-security-policy"></a>Vérifier l’existence d’une alerte de journal d’activité pour la mise à jour d’une stratégie de sécurité

**ID** : **Propriété** CIS Azure 5.2.9 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une alerte de journal d’activité doit exister pour des opérations de sécurité spécifiques](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Cette stratégie audite des opérations de sécurité spécifiques sans aucune alerte de journal d’activité configurée. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

## <a name="networking"></a>Mise en réseau

### <a name="ensure-that-rdp-access-is-restricted-from-the-internet"></a>Vérifier que l’accès RDP à partir d’Internet est limité

**ID** : **Propriété** CIS Azure 6.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’accès RDP à partir d’Internet doit être bloqué](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe372f825-a257-4fb8-9175-797a8a8627d6) |Cette stratégie audite toute règle de sécurité réseau autorisant l’accès RDP à partir d’Internet. |Audit, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroup_RDPAccess_Audit.json) |

### <a name="ensure-that-ssh-access-is-restricted-from-the-internet"></a>Vérifier que l’accès SSH à partir d’Internet est limité

**ID** : **Propriété** CIS Azure 6.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’accès SSH à partir d’Internet doit être bloqué](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fab) |Cette stratégie audite toute règle de sécurité réseau autorisant l’accès SSH à partir d’Internet. |Audit, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroup_SSHAccess_Audit.json) |

### <a name="ensure-that-network-watcher-is-enabled"></a>Vérifier que Network Watcher est défini sur « Enabled »

**ID** : **Propriété** CIS Azure 6.5 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Network Watcher doit être activé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6e2945c-0b7b-40f5-9233-7a5323b5cdc6) |Network Watcher est un service régional qui vous permet de surveiller et de diagnostiquer l’état au niveau d’un scénario réseau dans, vers et depuis Azure. La surveillance au niveau des scénarios vous permet de diagnostiquer les problèmes avec une vue de bout en bout du réseau. Les outils de visualisation et de diagnostic réseau disponibles avec Network Watcher vous aident à comprendre, diagnostiquer et obtenir des informations sur votre réseau dans Azure. |auditIfNotExists |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Enabled_Audit.json) |

## <a name="virtual-machines"></a>Virtual Machines

### <a name="ensure-that-os-disk-are-encrypted"></a>Vérifier que les disques du système d’exploitation sont chiffrés

**ID** : **Propriété** CIS Azure 7.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le chiffrement de disque doit être appliqué sur les machines virtuelles](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Les machines virtuelles sans chiffrement de disque activé seront supervisées par Azure Security Center en tant que recommandations. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |

### <a name="ensure-that-data-disks-are-encrypted"></a>Vérifier que les disques de données sont chiffrés

**ID** : **Propriété** CIS Azure 7.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le chiffrement de disque doit être appliqué sur les machines virtuelles](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Les machines virtuelles sans chiffrement de disque activé seront supervisées par Azure Security Center en tant que recommandations. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |

### <a name="ensure-that-unattached-disks-are-encrypted"></a>Vérifier que les disques non attachés sont chiffrés

**ID** : **Propriété** CIS Azure 7.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les disques non attachés doivent être chiffrés](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fb2) |Cette stratégie permet d’auditer tous les disques non attachés sans chiffrement activé. |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/UnattachedDisk_Encryption_Audit.json) |

### <a name="ensure-that-only-approved-extensions-are-installed"></a>Vérifier que seules les extensions approuvées sont installées

**ID** : **Propriété** CIS Azure 7.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Seules les extensions de machine virtuelle approuvées doivent être installées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc0e996f8-39cf-4af9-9f45-83fbde810432) |Cette stratégie régit les extensions de machine virtuelle qui ne sont pas approuvées. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VirtualMachines_ApprovedExtensions_Audit.json) |

### <a name="ensure-that-the-latest-os-patches-for-all-virtual-machines-are-applied"></a>Vérifier que les derniers correctifs de système d’exploitation sont appliqués pour toutes les machines virtuelles

**ID** : **Propriété** CIS Azure 7.5 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Les mises à jour système doivent être installées sur vos machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86b3d65f-7626-441e-b690-81a8b71cff60) |Les mises à jour système de sécurité manquantes sur vos serveurs seront supervisées par Azure Security Center en tant que recommandation |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingSystemUpdates_Audit.json) |

### <a name="ensure-that-the-endpoint-protection-for-all-virtual-machines-is-installed"></a>Vérifier que la protection de point de terminaison est installée pour toutes les machines virtuelles

**ID** : **Propriété** CIS Azure 7.6 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Superviser les agents Endpoint Protection manquants dans Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Les serveurs sans agent Endpoint Protection installé seront supervisés par Azure Security Center en tant que recommandation |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |

## <a name="other-security-considerations"></a>Autres considérations liées à la sécurité

### <a name="ensure-the-key-vault-is-recoverable"></a>Vérifier que le coffre de clés est récupérable

**ID** : **Propriété** CIS Azure 8.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La protection contre la suppression définitive doit être activée sur le coffre de clés](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |La suppression malveillante d’un coffre de clés peut entraîner une perte définitive des données. Un membre malveillant de votre organisation peut être en mesure d’accéder aux coffres de clés pour les supprimer et les vider. La protection contre la suppression définitive vous protège des attaques internes en appliquant une période de conservation obligatoire pour les coffres de clés supprimés de manière réversible. Personne au sein de votre organisation ni chez Microsoft ne pourra supprimer définitivement vos coffres de clés pendant la période de conservation de la suppression réversible. |Audit, Refuser, Désactivé |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |

### <a name="enable-role-based-access-control-rbac-within-azure-kubernetes-services"></a>Activer le contrôle d’accès en fonction du rôle (RBAC) dans Azure Kubernetes Services

**ID** : **Propriété** CIS Azure 8.5 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le contrôle d’accès en fonction du rôle (RBAC) doit être utilisé sur les services Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |Pour fournir un filtrage précis des actions que les utilisateurs peuvent effectuer, utilisez le contrôle d’accès en fonction du rôle (RBAC) pour gérer les autorisations dans les clusters Kubernetes Service et configurez les stratégies d’autorisation appropriées. |Audit, Désactivé |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

## <a name="appservice"></a>AppService

### <a name="ensure-app-service-authentication-is-set-on-azure-app-service"></a>Vérifier qu’App Service Authentication est défini sur Azure App Service

**ID** : **Propriété** CIS Azure 9.1 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’authentification doit être activée sur votre application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4ebc54a-46e1-481a-bee2-d4411e95d828) |L’authentification Azure App Service est une fonctionnalité qui peut empêcher les requêtes HTTP anonymes d’accéder à l’application API ou authentifier celles ayant des jetons avant qu’elles n’accèdent à l’application API. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_ApiApp_Audit.json) |
|[L’authentification doit être activée sur votre application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc75248c1-ea1d-4a9c-8fc9-29a6aabd5da8) |L’authentification Azure App Service est une fonctionnalité qui peut empêcher les requêtes HTTP anonymes d’accéder à l’application de fonction ou authentifier celles ayant des jetons avant qu’elles n’accèdent à l’application de fonction. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_functionapp_Audit.json) |
|[L’authentification doit être activée sur votre application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F95bccee9-a7f8-4bec-9ee9-62c3473701fc) |L’authentification Azure App Service est une fonctionnalité qui peut empêcher les requêtes HTTP anonymes d’accéder à l’application web ou authentifier celles ayant des jetons avant qu’elles n’accèdent à l’application web. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_WebApp_Audit.json) |

### <a name="ensure-web-app-redirects-all-http-traffic-to-https-in-azure-app-service"></a>Vérifier que l’application web redirige tout le trafic HTTP vers HTTPS dans Azure App Service

**ID** : **Propriété** CIS Azure 9.2 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’application web ne doit pas être accessible via HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |L'utilisation de HTTPS garantit l'authentification du serveur/service, et protège les données en transit contre les attaques par écoute clandestine de la couche réseau. |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="ensure-web-app-is-using-the-latest-version-of-tls-encryption"></a>Vérifier que l’application web utilise la dernière version du chiffrement TLS

**ID** : **Propriété** CIS Azure 9.3 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La dernière version de TLS doit être utilisée dans votre application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Mise à niveau vers la version la plus récente de TLS |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[La dernière version de TLS doit être utilisée dans votre application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Mise à niveau vers la version la plus récente de TLS |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[La dernière version de TLS doit être utilisée dans votre application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Mise à niveau vers la version la plus récente de TLS |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |

### <a name="ensure-the-web-app-has-client-certificates-incoming-client-certificates-set-to-on"></a>Vérifier que « Certificats clients (certificats clients entrants) » est activé pour l’application web

**ID** : **Propriété** CIS Azure 9.4 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Vérifier que « Certificats clients (certificats clients entrants) » est activé pour l’application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0c192fe8-9cbb-4516-85b3-0ade8bd03886) |Les certificats clients permettent à l’application de demander un certificat pour les demandes entrantes. Seuls les clients disposant d’un certificat valide peuvent accéder à l’application. |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_ClientCert.json) |
|[Vérifier que « Certificats clients (certificats clients entrants) » est activé pour l’application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feaebaea7-8013-4ceb-9d14-7eb32271373c) |Les certificats clients permettent à l’application de demander un certificat pour les demandes entrantes. Seuls les clients disposant d’un certificat valide peuvent accéder à l’application. |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_ClientCert.json) |
|[Vérifier que « Certificats clients (certificats clients entrants) » est activé pour l’application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5bb220d9-2698-4ee4-8404-b9c30c9df609) |Les certificats clients permettent à l’application de demander un certificat pour les demandes entrantes. Seuls les clients disposant d’un certificat valide peuvent accéder à l’application. |Audit, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_ClientCert.json) |

### <a name="ensure-that-register-with-azure-active-directory-is-enabled-on-app-service"></a>Vérifier que l’inscription auprès d’Azure Active Directory est activée pour App Service

**ID** : **Propriété** CIS Azure 9.5 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Une identité managée doit être utilisée dans votre application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4d441f8-f9d9-4a9e-9cef-e82117cb3eef) |Utiliser une identité managée pour renforcer la sécurité de l’authentification |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_ApiApp_Audit.json) |
|[Une identité managée doit être utilisée dans votre application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0da106f2-4ca3-48e8-bc85-c638fe6aea8f) |Utiliser une identité managée pour renforcer la sécurité de l’authentification |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_FunctionApp_Audit.json) |
|[Une identité managée doit être utilisée dans votre application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2b9ad585-36bc-4615-b300-fd4435808332) |Utiliser une identité managée pour renforcer la sécurité de l’authentification |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_WebApp_Audit.json) |

### <a name="ensure-that-php-version-is-the-latest-if-used-to-run-the-web-app"></a>Vérifier que la version de PHP est la plus récente si elle est utilisée pour exécuter l’application web

**ID** : **Propriété** CIS Azure 9.7 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Vérifier que la version de PHP est la plus récente, si elle est utilisée dans le cadre de l’application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1bc1795e-d44a-4d48-9b3b-6fff0fd5f9ba) |Régulièrement, de nouvelles versions sont publiées pour le logiciel PHP, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de PHP pour les applications API afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_PHP_Latest.json) |
|[Vérifier que la version de PHP est la plus récente, si elle est utilisée dans le cadre de l’application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7261b898-8a84-4db8-9e04-18527132abb3) |Régulièrement, de nouvelles versions sont publiées pour le logiciel PHP, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de PHP pour les applications web afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_PHP_Latest.json) |

### <a name="ensure-that-python-version-is-the-latest-if-used-to-run-the-web-app"></a>Vérifier que la version de Python est la plus récente si elle est utilisée pour exécuter l’application web

**ID** : **Propriété** CIS Azure 9.8 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Vérifier que la version de Python est la plus récente, si elle est utilisée dans le cadre de l’application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F74c3584d-afae-46f7-a20a-6f8adba71a16) |Régulièrement, de nouvelles versions sont publiées pour le logiciel Python, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de Python pour les applications API, afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_python_Latest.json) |
|[Vérifier que la version de Python est la plus récente, si elle est utilisée dans le cadre de l’application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7238174a-fd10-4ef0-817e-fc820a951d73) |Régulièrement, de nouvelles versions sont publiées pour le logiciel Python, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de Python pour les applications de fonction afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_python_Latest.json) |
|[Vérifier que la version de Python est la plus récente, si elle est utilisée dans le cadre de l’application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7008174a-fd10-4ef0-817e-fc820a951d73) |Régulièrement, de nouvelles versions sont publiées pour le logiciel Python, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de Python pour les applications web afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_python_Latest.json) |

### <a name="ensure-that-java-version-is-the-latest-if-used-to-run-the-web-app"></a>Vérifier que la version de Java est la plus récente si elle est utilisée pour exécuter l’application web

**ID** : **Propriété** CIS Azure 9.9 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Vérifier que la version de Java est la plus récente, si elle est utilisée dans le cadre de l’application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F88999f4c-376a-45c8-bcb3-4058f713cf39) |Régulièrement, de nouvelles versions sont publiées pour Java, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de Python pour les applications API, afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_java_Latest.json) |
|[Vérifier que la version de Java est la plus récente, si elle est utilisée dans le cadre de l’application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d0b6ea4-93e2-4578-bf2f-6bb17d22b4bc) |Régulièrement, de nouvelles versions sont publiées pour le logiciel Java, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. Nous vous recommandons d’utiliser la version la plus récente de Java pour les applications de fonction afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_java_Latest.json) |
|[Vérifier que la version de Java est la plus récente, si elle est utilisée dans le cadre de l’application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F496223c3-ad65-4ecd-878a-bae78737e9ed) |Régulièrement, de nouvelles versions sont publiées pour le logiciel Java, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. L’utilisation de la version la plus récente de Java pour les applications web est recommandée pour tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_java_Latest.json) |

### <a name="ensure-that-http-version-is-the-latest-if-used-to-run-the-web-app"></a>Vérifier que la version de HTTP est la plus récente si elle est utilisée pour exécuter l’application web

**ID** : **Propriété** CIS Azure 9.10 : Customer

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Vérifier que la version de HTTP est la plus récente, si elle est utilisée pour exécuter l’application API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F991310cd-e9f3-47bc-b7b6-f57b557d07db) |Régulièrement, de nouvelles versions sont publiées pour HTTP, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. L’utilisation de la version de HTTP la plus récente est recommandée pour les applications web afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_HTTP_Latest.json) |
|[Vérifier que la version de HTTP est la plus récente, si elle est utilisée pour exécuter l’application de fonction](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe2c1c086-2d84-4019-bff3-c44ccd95113c) |Régulièrement, de nouvelles versions sont publiées pour HTTP, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. L’utilisation de la version de HTTP la plus récente est recommandée pour les applications web afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_HTTP_Latest.json) |
|[Vérifier que la version de HTTP est la plus récente, si elle est utilisée pour exécuter l’application web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8c122334-9d20-4eb8-89ea-ac9a705b74ae) |Régulièrement, de nouvelles versions sont publiées pour HTTP, soit en raison de failles de sécurité, soit pour inclure des fonctionnalités supplémentaires. L’utilisation de la version de HTTP la plus récente est recommandée pour les applications web afin de tirer parti des correctifs de sécurité, le cas échéant, et/ou des nouvelles fonctionnalités. Actuellement, cette stratégie s’applique uniquement aux applications web Linux. |AuditIfNotExists, Désactivé |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_HTTP_Latest.json) |

> [!NOTE]
> La disponibilité des définitions Azure Policy peut varier dans Azure Government et dans d’autres clouds nationaux.

## <a name="next-steps"></a>Étapes suivantes

Autres articles sur Azure Policy :

- Présentation de la [Conformité réglementaire](../concepts/regulatory-compliance.md).
- Voir la [structure de la définition d’initiative](../concepts/initiative-definition-structure.md).
- Passez en revue d’autres exemples de la page [Exemples Azure Policy](./index.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
