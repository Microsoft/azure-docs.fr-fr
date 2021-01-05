---
title: Exécuter une procédure de reprise d’activité après sinistre sur Azure à l’aide d’Azure Site Recovery
description: Découvrez comment exécuter une procédure de reprise d’activité après sinistre d’un site local vers Azure à l’aide d’Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 11/12/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 5bd9f5316f8b8799633de8c0c84c61424c0e4f4a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84699615"
---
# <a name="run-a-disaster-recovery-drill-to-azure"></a>Effectuer un exercice de récupération d'urgence vers Azure

Cet article explique comment exécuter une simulation de reprise d’activité pour une machine locale sur Azure à l’aide du service [Azure Site Recovery](site-recovery-overview.md). Un exercice valide votre stratégie de réplication sans perte de données.


Il s’agit du quatrième tutoriel d’une série qui montre comment configurer la reprise d’activité sur Azure pour des machines locales.

Dans ce tutoriel, vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer un réseau isolé pour le test de basculement
> * Préparer la connexion à la machine virtuelle Azure après le basculement
> * Exécuter un test de basculement pour une seule machine

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario. Ils utilisent les options par défaut lorsque cela est possible et n’affichent pas tous les paramètres et chemins d’accès possibles. Pour en savoir plus sur les étapes de la simulation de reprise d’activité, vous pouvez [consulter cet article](site-recovery-test-failover-to-azure.md).

## <a name="before-you-start"></a>Avant de commencer

Effectuez les tutoriels précédents :

1. Vérifiez que vous avez [configuré Azure](tutorial-prepare-azure.md) pour permettre la reprise d’activité locale des machines virtuelles VMware, des machines virtuelles Hyper-V et des machines physiques sur Azure.
2. Préparez votre environnement [VMware](vmware-azure-tutorial-prepare-on-premises.md) ou [Hyper-V](hyper-v-prepare-on-premises-tutorial.md) local à la reprise d’activité. Si vous configurez la reprise d’activité pour les serveurs physiques, consultez la [matrice de prise en charge](vmware-physical-secondary-support-matrix.md).
3. Configurez la reprise d’activité pour les [machines virtuelles VMware](vmware-azure-tutorial.md), les [machines virtuelles Hyper-V](hyper-v-azure-tutorial.md) ou les [machines physiques](physical-azure-disaster-recovery.md).
 

## <a name="verify-vm-properties"></a>Vérifier les propriétés de la machine virtuelle

Avant d’exécuter un test de basculement, vérifiez les propriétés de la machine virtuelle, et assurez-vous que la [machine virtuelle Hyper-V](hyper-v-azure-support-matrix.md#replicated-vms) ou [VMware](vmware-physical-azure-support-matrix.md#replicated-machines) est conforme aux conditions requises pour Azure.

1. Dans **Éléments protégés**, cliquez sur **Éléments répliqués**, puis sur la machine virtuelle.
2. Dans le volet **Élément répliqué**, vous voyez un récapitulatif des informations de la machine virtuelle, son état d’intégrité et ses derniers points de récupération disponibles. Cliquez sur **Propriétés** pour obtenir plus de détails.
3. Dans **Calcul et réseau**, vous pouvez modifier le nom Azure, le groupe de ressources, la taille cible, le groupe à haute disponibilité et les paramètres de disque managé.
4. Vous pouvez afficher et modifier les paramètres réseau, notamment le réseau/sous-réseau dans lequel la machine virtuelle Azure se trouvera après le basculement et l’adresse IP à lui affecter.
5. Des informations sur les disques de données et du système d’exploitation de la machine virtuelle s’affichent dans **Disques**.

## <a name="create-a-network-for-test-failover"></a>Créer un réseau pour le test de basculement

Pour le test de basculement, nous vous recommandons de choisir un réseau isolé du réseau du site de récupération de production indiqué dans les paramètres **Calcul et réseau** de chaque machine virtuelle. Lorsque vous créez un réseau virtuel Azure, celui-ci est par défaut isolé des autres réseaux. Le réseau de test doit reproduire votre réseau de production :

- Le réseau de test doit avoir le même nombre de sous-réseaux que votre réseau de production. Les sous-réseaux doivent porter les mêmes noms.
- Le réseau de test doit utiliser la même plage d’adresses IP.
- Mettez à jour le DNS du réseau de test avec l’adresse IP spécifiée pour la machine virtuelle DNS dans les paramètres **Calcul et réseau**. Pour plus d’informations, consultez [Considérations relatives au test de basculement pour Active Directory](site-recovery-active-directory.md#test-failover-considerations).

## <a name="run-a-test-failover-for-a-single-vm"></a>Exécuter un test de basculement pour une seule machine virtuelle

Quand vous effectuez un test de basculement, voici ce qui se produit :

1. Une vérification des prérequis est effectuée pour garantir que toutes les conditions nécessaires pour le basculement sont en place.
2. Le basculement traite les données pour permettre la création d’une machine virtuelle Azure. Si vous avez sélectionné le dernier point de récupération, un point de récupération est créé à partir des données.
3. Une machine virtuelle Azure est créée en utilisant les données traitées à l’étape précédente.

Exécutez un test de basculement, en procédant comme suit :

1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur Machine virtuelle > **+Test de basculement**.
2. Sélectionnez le **dernier point de récupération traité** dans ce didacticiel. Cela bascule l’ordinateur virtuel vers la dernière version disponible à un moment donné. L’horodatage est affiché. Cette option, avec laquelle aucun temps n’est passé à traiter les données, offre un objectif de délai de récupération faible.
3. Dans **Tester le basculement**, sélectionnez le réseau Azure cible auquel les machines virtuelles Azure seront connectées après le basculement.
4. Cliquez sur **OK** pour commencer le basculement. Vous pouvez suivre la progression en cliquant sur la machine virtuelle pour ouvrir ses propriétés. Vous pouvez également cliquer sur le travail **Test de basculement** dans le nom du coffre > **Paramètres** > **Travaux** >
   **Travaux Site Recovery**.
5. Une fois le basculement terminé, la machine virtuelle Azure de réplication apparaît dans le portail Azure > **Machines virtuelles**. Vérifiez que la machine virtuelle est de la taille appropriée, qu’elle est connectée au réseau approprié et qu’elle est en cours d’exécution.
6. Vous devriez à présent pouvoir vous connecter à la machine virtuelle répliquée dans Azure.
7. Pour supprimer les machines virtuelles créées lors du test de basculement, cliquez sur **Nettoyer le test de basculement** sur la machine virtuelle. Cliquez sur **Notes** pour consigner et enregistrer d’éventuelles observations sur le test de basculement.

Dans certains scénarios, le basculement nécessite un traitement supplémentaire qui dure environ huit à dix minutes. Vous constaterez peut-être des délais de basculement plus longs pour les machines VMware Linux, les machines virtuelles VMware pour lesquelles le service DHCP n’est pas activé et les machines virtuelles VMware qui ne disposent pas des pilotes de démarrage suivants : storvsc, vmbus, storflt, intelide, atapi.

## <a name="connect-after-failover"></a>Se connecter après le basculement

Si vous souhaitez vous connecter aux machines virtuelles Azure via RDP/SSH après un basculement, [préparez la connexion](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover). Si vous rencontrez des problèmes de connectivité après le basculement, suivez le guide de [résolution des problèmes](site-recovery-failover-to-azure-troubleshoot.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Exécuter un basculement et une restauration automatique pour les machines virtuelles VMware](vmware-azure-tutorial-failover-failback.md)
> [Exécuter un basculement et une restauration automatique pour les machines virtuelles Hyper-V](hyper-v-azure-failover-failback-tutorial.md)
> [Exécuter un basculement et une restauration automatique pour les machines physiques](physical-to-azure-failover-failback.md)
