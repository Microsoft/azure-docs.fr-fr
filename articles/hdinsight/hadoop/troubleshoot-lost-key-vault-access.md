---
title: Les clusters Azure HDInsight avec chiffrement de disque perdent l’accès à Key Vault
description: Décrit les étapes de détection de problèmes et les résolutions possibles de problèmes d’accès à Key Vault rencontrés lors d’interactions avec des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.openlocfilehash: bc8162f3a7ca8744a94aba039996275b5f13c727
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533408"
---
# <a name="scenario-azure-hdinsight-clusters-with-disk-encryption-lose-key-vault-access"></a>Scénario : Les clusters Azure HDInsight avec chiffrement de disque perdent l’accès à Key Vault

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

L’alerte du Resource Health Center (RHC), `The HDInsight cluster is unable to access the key for BYOK encryption at rest`, s’affiche pour les clusters Bring Your Own Key (BYOK), lorsque les nœuds de cluster ont perdu l’accès aux clients Key Vault (KV). Des alertes similaires peuvent également s’afficher sur l’interface utilisateur d’Apache Ambari.

## <a name="cause"></a>Cause

L’alerte garantit que KV est accessible à partir des nœuds de cluster, garantissant ainsi la connexion réseau, l’intégrité de KV et la stratégie d’accès de l’identité managée affectée par l’utilisateur. Cette alerte est uniquement un avertissement de l’arrêt imminent du répartiteur lors des redémarrages ultérieurs du nœud ; le cluster continue de fonctionner jusqu’au redémarrage des nœuds.

Accédez à l’interface utilisateur d’Apache Ambari pour obtenir plus d’informations sur l’alerte à partir de la rubrique **État de Key Vault pour le chiffrement de disque** . Cette alerte aura des détails sur la raison pour laquelle la vérification a échoué.

## <a name="resolution"></a>Résolution

### <a name="kvaad-outage"></a>Panne KV/AAD

Pour plus d’informations, consultez [Disponibilité et redondance d’Azure Key Vault](../../key-vault/general/disaster-recovery-guidance.md) et la page d’état d’Azure https://status.azure.com/

### <a name="kv-accidental-deletion"></a>Suppression accidentelle de KV

* Restaurez la clé supprimée sur KV pour la récupérer automatiquement. Pour plus d’informations, consultez [Récupérer une clé supprimée](/rest/api/keyvault/recoverdeletedkey).
* Contactez l’équipe KV pour une récupération en cas de suppressions accidentelles.

### <a name="kv-access-policy-changed"></a>Modification de la stratégie d’accès de KV

Restaurez les stratégies d’accès pour l’identité managée affectée par l’utilisateur qui est attribuée au cluster HDI pour accéder à KV.

### <a name="key-permitted-operations"></a>Opérations de clé autorisées

Pour chaque clé de KV, vous pouvez choisir l’ensemble des opérations autorisées. Assurez-vous d’avoir activé les opérations visant à envelopper et à désenvelopper la clé BYOK

### <a name="expired-key"></a>Clé expirée

Si la date d’expiration est passée et que la clé n’est pas renouvelée, restaurez la clé à partir du HSM de sauvegarde ou contactez l’équipe KV pour désactiver la date d’expiration.

### <a name="kv-firewall-blocking-access"></a>Pare-feu KV bloquant l’accès

Corrigez les paramètres du pare-feu KV pour permettre aux nœuds de cluster BYOK d’accéder à KV.

### <a name="nsg-rules-on-virtual-network-blocking-access"></a>Règles NSG sur le réseau virtuel bloquant l’accès

Vérifiez les règles NSG associées au réseau virtuel attaché au cluster.

## <a name="mitigation-and-prevention-steps"></a>Étapes d’atténuation et de prévention

### <a name="kv-accidental-deletion"></a>Suppression accidentelle de KV

* Configurez Key Vault en [définissant des verrous de ressources](../../azure-resource-manager/management/lock-resources.md).
* Sauvegardez les clés dans le module de sécurité matériel.

### <a name="key-deletion"></a>Suppression des clés

Le cluster doit être supprimé avant la suppression de la clé.

### <a name="kv-access-policy-changed"></a>Modification de la stratégie d’accès de KV

Auditez et testez régulièrement les stratégies d’accès.

### <a name="expired-key"></a>Clé expirée

* Sauvegardez les clés dans votre HSM.
* Utilisez une clé sans définir d’expiration.
* Si l’expiration doit être définie, renouvelez les clés avant la date d’expiration.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).