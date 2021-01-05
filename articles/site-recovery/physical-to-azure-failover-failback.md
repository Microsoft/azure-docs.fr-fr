---
title: Configurer le basculement et la restauration automatique pour les serveurs physiques avec Site Recovery
description: Découvrez comment basculer des serveurs physiques vers Azure, et comment les restaurer automatiquement sur le site local pour la reprise d’activité avec Azure Site Recovery
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: article
ms.date: 12/17/2019
ms.author: raynew
ms.openlocfilehash: 2994f68e4159c7c4aa7d82bef7a5891deb5055a0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017421"
---
# <a name="fail-over-and-fail-back-physical-servers-replicated-to-azure"></a>Basculer et restaurer automatiquement des serveurs physiques répliqués vers Azure

Ce tutoriel explique comment basculer des serveurs physiques locaux qui répliquent vers Azure avec [Azure Site Recovery](site-recovery-overview.md). Une fois que vous avez procédé au basculement, vous restaurez automatiquement d’Azure vers votre site local quand il est disponible.

## <a name="before-you-start"></a>Avant de commencer

- [Découvrez](failover-failback-overview.md) le processus de basculement en cas de récupération d’urgence.
- Si vous souhaitez effectuer le basculement de plusieurs ordinateurs, [découvrez](recovery-plan-overview.md) comment rassembler des machines dans un plan de récupération.
- Avant de procéder à un basculement complet, effectuez un [test de récupération d'urgence](site-recovery-test-failover-to-azure.md) pour vous vérifier que tout fonctionne comme prévu.
- Suivez [ces instructions](site-recovery-failover.md#prepare-to-connect-after-failover) pour préparer la connexion aux machines virtuelles Azure après le basculement.



## <a name="run-a-failover"></a>Exécuter un basculement

### <a name="verify-server-properties"></a>Vérifier les propriétés du serveur

Vérifiez les propriétés du serveur et que le serveur est conforme aux [conditions requises Azure](vmware-physical-azure-support-matrix.md#replicated-machines) pour les machines virtuelles.

1. Dans **Éléments protégés**, cliquez sur **Éléments répliqués** et sélectionnez la machine.
2. Dans le volet **Élément répliqué**, vous voyez un récapitulatif des informations de la machine, son état d’intégrité et ses derniers points de récupération disponibles. Cliquez sur **Propriétés** pour obtenir plus de détails.
3. Dans **Calcul et réseau**, vous pouvez modifier le nom Azure, le groupe de ressources, la taille cible, le [groupe à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md) ainsi que les paramètres de disque managé
4. Vous pouvez afficher et modifier les paramètres réseau, notamment le réseau/sous-réseau dans lequel la machine virtuelle Azure se trouvera après le basculement et l’adresse IP à lui affecter.
5. Des informations sur les disques de données et de système d’exploitation de la machine s’affichent dans **Disques**.

### <a name="fail-over-to-azure"></a>Basculer vers Azure

1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine > **Basculement**.
2. Dans **Basculer**, sélectionnez un **point de récupération** vers lequel basculer. Vous pouvez utiliser l’une des options suivantes :
   - **Les dernières** : cette option traite d’abord toutes les données envoyées à Site Recovery. Elle fournit l’objectif de point de récupération (RPO) le plus faible, car la machine virtuelle Azure créée après le basculement a toutes les données qui ont été répliquées vers Site Recovery quand le basculement a été déclenché.
   - **Dernier point traité** : cette option bascule la machine vers le dernier point de récupération traité par Site Recovery. Cette option fournit un objectif de délai de récupération (RTO) faible, car aucun temps n’est consacré à traiter les données non traitées.
   - **Dernier point de cohérence des applications** : cette option bascule la machine vers le dernier point de récupération de cohérence des applications traité par Site Recovery.
   - **Personnalisé** : spécifiez un point de récupération.

3. Sélectionnez **Arrêter la machine avant de commencer le basculement** si vous souhaitez que Site Recovery tente d’arrêter la machine source avant de déclencher le basculement. Le basculement est effectué même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
4. Si vous avez préparé la connexion à la machine virtuelle Azure, connectez-vous pour la vérifier après le basculement.
5. Après vérification, **validez** le basculement. Cela supprime tous les points de récupération disponibles.

> [!WARNING]
> N’annulez pas un basculement en cours. Avant le début du basculement, la réplication de la machine s’arrête. Si vous annulez le basculement, celui-ci s’arrête, mais la machine n’est pas à nouveau répliquée.
> Pour les serveurs physiques, le basculement nécessite un traitement supplémentaire qui dure environ huit à dix minutes.

## <a name="automate-actions-during-failover"></a>Automatiser des actions pendant le basculement

Vous souhaiterez peut-être automatiser des actions pendant le basculement. Pour ce faire, vous pouvez utiliser des scripts ou des runbooks Azure Automation dans les plans de récupération.

- [Découvrez](site-recovery-create-recovery-plans.md) la création et la personnalisation des plans de récupération, notamment l’ajout de scripts.
- [Découvrez](site-recovery-runbook-automation.md) l’ajout de runbooks Azure Automation à des plans de récupération.

## <a name="configure-settings-after-failover"></a>Configurer des paramètres après un basculement

Après le basculement, vous devez [configurer les paramètres Azure](site-recovery-failover.md#prepare-in-azure-to-connect-after-failover) pour vous connecter aux machines virtuelles Azure répliquées. En outre, configurez les adresses IP [interne et publique](site-recovery-failover.md#set-up-ip-addressing).

## <a name="prepare-for-reprotection-and-failback"></a>Préparer en vue d’une reprotection et de la restauration automatique

Après le basculement vers Azure, vous reprotégez les machines virtuelles Azure en les répliquant sur le site local. Après qu’elles sont répliquées, vous pouvez les restaurer automatiquement localement en effectuant un basculement d’Azure vers votre site local.

1. Les serveurs physiques répliqués dans Azure à l’aide de Site Recovery peuvent être restaurés automatiquement en tant que machines virtuelles VMware. Vous avez besoin d’une infrastructure VMware locale pour la restauration automatique. Suivez les étapes décrites dans [cet article](vmware-azure-prepare-failback.md) pour préparer la reprotection et la restauration automatique, notamment la configuration d’un serveur de processus dans Azure et d’un serveur cible maître local, ainsi que la configuration d’un VPN de site à site ou d’une homologation privée ExpressRoute pour la restauration automatique.
2. Vérifiez que le serveur de configuration local est en cours d’exécution et qu’il est connecté à Azure. Pendant le basculement vers Azure, le site local risque de ne pas être accessible et le serveur de configuration peut être indisponible ou à l’arrêt. Lors de la restauration automatique, la machine virtuelle doit exister dans la base de données du serveur de configuration. Sinon, la restauration automatique échoue.
3. Supprimez les captures instantanées sur le serveur cible maître local. La reprotection ne fonctionnera pas en présence de captures instantanées.  Les captures instantanées sur la machine virtuelle sont automatiquement fusionnées lors d’une tâche de reprotection.
4. Si vous reprotégez des machines virtuelles rassemblées dans un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles, vérifiez qu’elles ont toutes le même système d’exploitation (Windows ou Linux) et que le serveur cible maître que vous déployez a le même type de système d’exploitation. Toutes les machines virtuelles d’un groupe de réplication doivent utiliser le même serveur cible maître.
5. Ouvrez [les ports requis](vmware-azure-prepare-failback.md#ports-for-reprotectionfailback) pour la restauration automatique.
6. Assurez-vous que le serveur vCenter est connecté avant la restauration automatique. Sinon, la déconnexion des disques et leur attachement à la machine virtuelle échouent.
7. Si un serveur vCenter gère les machines virtuelles vers lesquelles vous allez effectuer une restauration automatique, vérifiez que vous disposez des autorisations nécessaires. Si vous effectuez une découverte utilisateur vCenter en mode lecture seule et protégez des machines virtuelles, la protection réussit et le basculement fonctionne. Cependant, pendant la reprotection, le basculement échoue parce que les magasins de données ne peuvent pas être détectés et ne sont pas listés. Pour résoudre ce problème, vous pouvez mettre à jour les informations d’identification vCenter avec [des autorisations ou un compte appropriés](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery), puis faire une nouvelle tentative. 
8. Si vous avez utilisé un modèle pour créer vos machines virtuelles, vérifiez que chaque machine virtuelle possède son propre UUID pour les disques. Si l’UUID de la machine virtuelle locale est en conflit avec l’UUID du serveur cible maître (car les deux ont été créés à partir du même modèle), la reprotection échoue. Déployez à partir d’un autre modèle.
9. Si vous effectuez une restauration automatique vers un autre serveur vCenter, vérifiez que le nouveau serveur vCenter et le serveur cible maître sont détectés. Généralement, si cela n’est pas le cas, les magasins de données ne sont pas accessibles ou ne sont pas visibles dans **Reprotéger**.
10. Vérifiez les scénarios suivants, qui ne permettent pas de restauration automatique :
    - Si vous utilisez les éditions gratuites d’ESXi 5.5 ou de vSphere Hypervisor 6. Effectuez une mise à niveau vers une autre version.
    - Si vous avez un serveur physique Windows Server 2008 R2 SP1.
    - Machines virtuelles qui ont été migrées.
    - Une machine virtuelle qui a été déplacée vers un autre groupe de ressources.
    - Une machine virtuelle Azure de réplication qui a été supprimée.
    - Une machine virtuelle Azure de réplication qui n’est pas protégée (réplication sur le site local).
10. [Examinez en revue les types de restaurations automatiques](concepts-types-of-failback.md) que vous pouvez utiliser : récupération dans l’emplacement d’origine et récupération dans un autre emplacement.


## <a name="reprotect-azure-vms-to-an-alternate-location"></a>Reprotéger les machines virtuelles Azure à un autre emplacement

Cette procédure présuppose que la machine virtuelle locale n’est pas disponible.

1. Dans le coffre **Paramètres** > **Éléments répliqués**, cliquez avec le bouton droit de la souris sur la machine virtuelle qui a été basculée > **Reprotéger**.
2. Dans **Reprotéger**, vérifiez que **D’Azure à local** est sélectionné.
3. Spécifiez le serveur cible maître local et le serveur de processus.
4. Dans **Magasin de données**, sélectionnez le magasin de données où vous voulez récupérer les disques locaux.
       - Utilisez cette option si la machine virtuelle locale a été supprimée ou n’existe pas et que vous devez créer des disques.
       - Ce paramètre est ignoré si les disques existent déjà, mais vous devez spécifier une valeur.
5. Sélectionnez le lecteur de conservation du serveur cible maître. La stratégie de restauration automatique est sélectionnée automatiquement.
6. Cliquez sur **OK** pour commencer l’opération de reprotection. Un travail de réplication de la machine virtuelle Azure sur le site local commence. Vous pouvez en suivre la progression sous l’onglet **Tâches** .

> [!NOTE]
> Si vous voulez récupérer la machine virtuelle Azure sur une machine virtuelle locale existante, montez le magasin de données de la machine virtuelle locale avec un accès en lecture/écriture sur l’hôte ESXi du serveur cible maître.


## <a name="fail-back-from-azure"></a>Effectuer une restauration automatique à partir d’Azure

Effectuez le basculement en procédant comme suit :

1. Dans la page **Éléments répliqués**, cliquez avec le bouton droit sur la machine virtuelle > **Basculement non planifié**.
2. Dans **Confirmer le basculement**, vérifiez que le sens du basculement est depuis Azure.
3.Sélectionnez le point de récupération à utiliser pour le basculement.
    - Nous vous recommandons d’utiliser le **dernier** point de récupération. Le point de cohérence d’application se situe derrière le point le plus récent et entraîne une perte de données.
    - Le **dernier** point de récupération est cohérent en cas d’incident.
    - Quand le basculement est effectué, Site Recovery arrête les machines virtuelles Azure et démarre la machine virtuelle locale. Il faut prévoir un temps d’indisponibilité : choisissez donc un moment approprié.
4. Cliquez avec le bouton droit sur la machine, puis cliquez sur **Valider**. Ceci déclenche une tâche qui supprime les machines virtuelles Azure.
5. Vérifiez que les machines virtuelles Azure ont été arrêtées comme attendu.


## <a name="reprotect-on-premises-machines-to-azure"></a>Reprotéger les machines locales sur Azure

Les données doivent maintenant être de retour sur votre site local, mais elles ne sont pas répliquées vers Azure. Vous pouvez redémarrer la réplication vers Azure comme suit :

1. Dans le coffre > **Paramètres** >**Éléments répliqués**, sélectionnez les machines virtuelles qui ont été restaurées automatiquement, puis cliquez sur **Reprotéger**.
2. Sélectionnez le serveur de processus qui est utilisé pour envoyer les données répliquées vers Azure, puis cliquez sur **OK**.


## <a name="next-steps"></a>Étapes suivantes

Une fois la tâche de reprotection terminée, la machine virtuelle locale est répliquée vers Azure. Si nécessaire, vous pouvez [exécuter un autre basculement](site-recovery-failover.md) vers Azure.
