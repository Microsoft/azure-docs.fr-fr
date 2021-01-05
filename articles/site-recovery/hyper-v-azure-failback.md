---
title: Restauration automatique des machines virtuelles Hyper-V à partir d’Azure avec Azure Site Recovery
description: Découvrez comment restaurer automatiquement des machines virtuelles Hyper-V à partir d’Azure vers un site local avec Azure Site Recovery.
services: site-recovery
author: Rajeswari-Mamilla
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 09/12/2019
ms.author: ramamill
ms.openlocfilehash: a31a28728dd0521262bd0518cc49a385f4314302
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87416228"
---
# <a name="run-a-failback-for-hyper-v-vms"></a>Exécuter une restauration automatique pour les machines virtuelles Hyper-V

Cet article explique comment restaurer automatiquement des machines virtuelles Azure créées après le basculement de machines virtuelles Hyper-V à partir d’un site local vers Azure avec [Azure Site Recovery](site-recovery-overview.md).

- Vous restaurez automatiquement des machines virtuelles Hyper-V à partir d’Azure en exécutant un basculement planifié d’Azure vers le site local. Si le sens du basculement est d’Azure vers un site local, il est considéré comme une restauration automatique.
- Étant donné qu’Azure est un environnement hautement disponible et que les machines virtuelles sont toujours disponibles, la restauration automatique à partir d’Azure est une activité planifiée. Vous pouvez planifier un faible temps d’arrêt afin que les charges de travail puissent recommencer à s’exécuter localement. 
- La restauration automatique planifiée désactive les machines virtuelles dans Azure et télécharge les dernières modifications. Il ne doit y avoir aucune perte de données.

## <a name="before-you-start"></a>Avant de commencer

1. [Examinez en revue les types de restaurations automatiques](failover-failback-overview.md#hyper-v-reprotectionfailback) que vous pouvez utiliser : récupération dans l’emplacement d’origine et récupération dans un autre emplacement.
2. Assurez-vous que les machines virtuelles Azure utilisent un compte de stockage et non des disques managés. La restauration automatique de machines virtuelles Hyper-V répliquées à l’aide de la gestion de disques managés n’est pas prise en charge.
3. Vérifiez que l’hôte Hyper-V local (ou le serveur System Center VMM si vous utilisez Site Recovery) est en cours d’exécution et connecté à Azure. 
4. Assurez-vous que le basculement et la validation sont terminés pour les machines virtuelles. Vous n’avez pas besoin de configurer des composants Site Recovery spécifiques pour la restauration automatique de machines virtuelles Hyper-V à partir d’Azure.
5. Le temps nécessaire pour effectuer la synchronisation des données et démarrer la machine virtuelle locale dépend de plusieurs facteurs. Pour accélérer le téléchargement des données, vous pouvez configurer l’agent Microsoft Azure Recovery Services afin qu’il utilise davantage de threads pour effectuer le téléchargement en parallèle. [Plus d’informations](https://support.microsoft.com/help/3056159/how-to-manage-on-premises-to-azure-protection-network-bandwidth-usage)


## <a name="fail-back-to-the-original-location"></a>Restauration dans l’emplacement d’origine

Pour restaurer automatiquement des machines virtuelles Hyper-V dans Azure sur la machine virtuelle d’origine, exécutez un basculement planifié d’Azure vers le site local comme suit :

1. Dans le coffre > **Éléments répliqués**, sélectionnez la machine virtuelle. Cliquez avec le bouton droit sur la machine virtuelle > **Basculement planifié**. Si vous effectuez restaurez automatiquement un plan de récupération, sélectionnez le nom du plan, puis cliquez sur **Basculement** > **Basculement planifié**.
2. Dans **Confirmer le basculement planifié**, choisissez les emplacements source et cible. Notez le sens du basculement. Si le basculement à partir du site principal a fonctionné comme prévu, et si toutes les machines virtuelles se trouvent dans l’emplacement secondaire, ces informations sont communiquées à titre purement informatif.
3. Dans **Synchronisation des données**, sélectionnez une option :
    - **Synchroniser les données avant le basculement (synchroniser seulement les modifications d’ordre différentiel)**  : cette option minimise le temps d’arrêt des machines virtuelles, car elle les synchronise sans les arrêter.
        - **Phase 1** : Prend une capture instantanée de la machine virtuelle Azure et la copie sur l’hôte Hyper-V local. La machine continue de s’exécuter dans Microsoft Azure.
        - **Phase 2** : Arrête la machine virtuelle Azure de sorte que plus aucune modification ne se produise. L’ensemble final des modifications différentielles est transféré au serveur local et la machine virtuelle locale est démarrée.
    - **Synchroniser les données uniquement lors du basculement (téléchargement complet)** : cette option est plus rapide car nous supposons que la majeure partie du disque a changé et ne voulons pas perdre de temps à calculer des sommes de contrôle. Cette option n’effectue aucun calcul de somme de contrôle.
        - Elle effectue un téléchargement du disque. 
        - Nous vous recommandons d’utiliser cette option si vous exécutez Azure depuis un certain temps (au moins un mois) ou en cas de suppression de machine virtuelle locale.

4. Pour VMM uniquement, si la fonction de chiffrement des données est activée pour le cloud, dans **Clé de chiffrement**, sélectionnez le certificat émis lorsque vous avez activé le chiffrement des données pendant l’installation du fournisseur sur le serveur VMM.
5. Lancez le basculement. Vous pouvez suivre la progression du basculement sur l’onglet **Tâches** .
6. Si vous avez sélectionné l’option de synchronisation des données avant le basculement, une fois la synchronisation initiale des données terminée, quand vous êtes prêt à arrêter les machines virtuelles dans Azure, cliquez sur **Tâches** > nom de la tâche > **Terminer le basculement**. Cette opération effectue les actions suivantes :
    - Arrête l’ordinateur Azure.
    - Transfère les dernières modifications apportées à la machine virtuelle locale.
    - Démarre la machine virtuelle locale.
7. Vous pouvez maintenant vous connecter à la machine virtuelle locale pour vérifier qu’elle est disponible comme prévu.
8. Cette dernière présente un état de validation en attente. Cliquez sur **Valider** pour valider le basculement.
9. Pour terminer la restauration automatique, cliquez sur **Réplication inverse** pour démarrer à nouveau la réplication de la machine virtuelle locale vers Azure.



## <a name="fail-back-to-an-alternate-location"></a>Restauration dans un autre emplacement 

Restauration dans un autre emplacement comme suit :

1. Si vous configurez un nouveau matériel, installez une [version prise en charge de Windows](hyper-v-azure-support-matrix.md#replicated-vms)et le rôle Hyper-V sur l’ordinateur.
2. Créez un commutateur réseau virtuel portant le même nom que celui du serveur d’origine.
3. Dans **Éléments protégés** > **Groupe de protection** > \<ProtectionGroupName> -> \<VirtualMachineName>, sélectionnez la machine virtuelle que vous souhaitez restaurer de manière automatique, puis **Basculement planifié**.
4. Dans **Confirmer le basculement planifié**, sélectionnez **Créer une machine virtuelle locale si elle n’existe pas**.
5. Dans **Nom d’hôte**, sélectionnez le nouveau serveur hôte Hyper-V sur lequel vous souhaitez placer la machine virtuelle.
6. Dans **Synchronisation des données**, nous vous recommandons de sélectionner l’option de synchronisation des données avant le basculement. Cela permet de réduire le temps d’arrêt des machines virtuelles, car elles sont synchronisées sans être arrêtées. Elle effectue les opérations suivantes :
    - **Phase 1** : Prend une capture instantanée de la machine virtuelle Azure et la copie sur l’hôte Hyper-V local. La machine continue de s’exécuter dans Microsoft Azure.
    - **Phase 2** : Arrête la machine virtuelle Azure de sorte que plus aucune modification ne se produise. L’ensemble final des modifications est transféré au serveur local, et la machine virtuelle locale est démarrée.
    
7. Cochez la base pour commencer le basculement (restauration automatique).
8. Quand la synchronisation initiale se termine et que vous êtes prêt à arrêter la machine virtuelle Azure, cliquez sur **Travaux** > \<planned failover job> > **Terminer le basculement**. Cette action arrête la machine Microsoft Azure et transfère les dernières modifications apportées à la machine virtuelle locale, puis démarre celle-ci.
9. Vous pouvez vous connecter à la machine virtuelle locale pour vérifier que tout fonctionne comme prévu.
10. Cliquez sur **Valider** pour terminer le basculement. La validation supprime la machine virtuelle Azure et ses disques, et prépare la machine virtuelle locale en vue de la protéger à nouveau.
10. Cliquez sur **Réplication inverse** pour démarrer la réplication de la machine virtuelle locale sur Azure. Seules les modifications différentielles depuis la désactivation de la machine virtuelle dans Azure sont répliquées.

    > [!NOTE]
    > Si vous annulez l’opération de restauration automatique durant la synchronisation des données, la machine virtuelle locale est dans un état endommagé. En effet, la synchronisation des données copie les données les plus récentes de la machine virtuelle Azure vers les disques de données locaux et, tant que la synchronisation n’est pas terminée, l’état du disque de données risque de ne pas être cohérent. Si la machine virtuelle locale est démarrée après l’annulation de la synchronisation des données, l’opération risque d’échouer. Dans ce cas, réexécutez le basculement pour terminer la synchronisation des données.


## <a name="next-steps"></a>Étapes suivantes
Une fois la machine virtuelle locale répliquée sur Azure, vous pouvez [exécuter un autre basculement](site-recovery-failover.md) vers Azure si nécessaire.
