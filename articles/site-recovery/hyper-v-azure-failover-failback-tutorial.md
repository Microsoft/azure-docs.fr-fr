---
title: Configurer le basculement des machines virtuelles Hyper-V vers Azure dans Azure Site Recovery
description: Découvrez comment configurer le basculement des machines virtuelles Hyper-V vers Azure dans Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 12/16/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: b46a2ea12a697afde8223cc3595365c1286512c7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86132456"
---
# <a name="fail-over-hyper-v-vms-to-azure"></a>Effectuer le basculement de machines virtuelles Hyper-V vers Azure

Ce tutoriel explique comment effectuer le basculement des machines virtuelles Hyper-V vers Azure dans [Azure Site Recovery](site-recovery-overview.md). Une fois que vous avez procédé au basculement, vous restaurez automatiquement sur votre site local quand il est disponible. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vérifier la conformité des propriétés des machines virtuelles Hyper-V aux spécifications d’Azure
> * Effectuer le basculement de certaines machines virtuelles vers Azure


Ce didacticiel est le cinquième d’une série. Il suppose que vous avez déjà effectué les tâches des didacticiels précédents.    

1. [Préparer Azure](tutorial-prepare-azure.md)
2. [Préparer un serveur Hyper-V local](./hyper-v-prepare-on-premises-tutorial.md)
3. Configurer la récupération d’urgence pour les [machines virtuelles Hyper-V](./hyper-v-azure-tutorial.md) ou pour les [machines virtuelles Hyper-V gérées par les clouds System Center VMM](./hyper-v-vmm-azure-tutorial.md)
4. [Effectuer un test de récupération d’urgence](tutorial-dr-drill-azure.md)

[Découvrez](failover-failback-overview.md#types-of-failover) les différents types de basculements. Si vous souhaitez effectuer le basculement de plusieurs machines virtuelles dans le cadre d’un plan de récupération, consultez [cet article](site-recovery-failover.md).

## <a name="prepare-for-failover"></a>Préparer un basculement 
Assurez-vous qu’il n’existe aucun instantané sur la machine virtuelle et que la machine virtuelle locale est arrêtée pendant la restauration automatique. Ceci permet de garantir la cohérence des données pendant la réplication. N’activez pas la machine virtuelle locale pendant la restauration automatique. 

Le basculement et la restauration automatique comportent trois étapes :

1. **Basculement vers Azure** : basculement de machines virtuelles Hyper-V du site local vers Azure.
2. **Restauration automatique sur un site local** : basculement de machines virtuelles Azure vers votre site local, lorsqu’il est disponible. La synchronisation des données à partir d’Azure vers le site local débute et une fois l’opération terminée, les machines virtuelles se trouvent en local.  
3. **Inverser la réplication de machines virtuelles locales** : une fois le basculement en local effectué, inversez la réplication des machines virtuelles locales pour démarrer leur réplication vers Azure.

## <a name="verify-vm-properties"></a>Vérifier les propriétés de la machine virtuelle

Avant le basculement, vérifiez les propriétés de la machine virtuelle et que la machine virtuelle est conforme aux [conditions requises pour Azure](hyper-v-azure-support-matrix.md#replicated-vms).

Dans **Éléments protégés**, cliquez sur **Éléments répliqués** > Machine virtuelle.

1. Dans le volet **Élément répliqué**, vous voyez un récapitulatif des informations de la machine virtuelle, son état d’intégrité et ses derniers points de récupération disponibles. Cliquez sur **Propriétés** pour obtenir plus de détails.

1. Dans **Calcul et réseau**, vous pouvez modifier le nom Azure, le groupe de ressources, la taille cible, le [groupe à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md) et les paramètres de disque managé.

1. Vous pouvez afficher et modifier les paramètres réseau, notamment le réseau/sous-réseau dans lequel la machine virtuelle Azure se trouvera après le basculement et l’adresse IP à lui affecter.

1. Des informations sur les disques de données et du système d’exploitation de la machine virtuelle s’affichent dans **Disques**.

## <a name="fail-over-to-azure"></a>Basculer vers Azure

1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine virtuelle > **Basculer**.
2. Dans **Basculement** sélectionnez le point de récupération **le plus récent**. 
3. Sélectionnez **Arrêter la machine avant de commencer le basculement**. Site Recovery tente d’arrêter les machines virtuelles source avant de déclencher le basculement. Le basculement est effectué même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
4. Après avoir vérifié le basculement, cliquez sur **Valider**. Ceci supprime tous les points de récupération disponibles.

> [!WARNING]
> **N’annulez pas un basculement en cours** : si vous annulez un basculement en cours, le basculement s’arrête mais la machine virtuelle ne sera pas à nouveau répliquée.

## <a name="connect-to-failed-over-vm"></a>Se connecter à une machine virtuelle basculée

1. Si vous souhaitez vous connecter à des machines virtuelles Azure après le basculement en utilisant le protocole RDP (Remote Desktop Protocol) et Secure Shell (SSH), [vérifiez que les exigences ont été satisfaites](failover-failback-overview.md#connect-to-azure-after-failover).
2. Une fois le basculement effectué, accédez à la machine virtuelle et validez-la en vous [connectant](../virtual-machines/windows/connect-logon.md) à celle-ci.
3. Utilisez **Changer le point de récupération**, si vous souhaitez vous servir d’un autre point de récupération après le basculement. Une fois le basculement validé au cours de l’étape suivante, cette option n’est plus disponible.
4. Une fois la validation effectuée, sélectionnez **Valider** pour finaliser le point de récupération de la machine virtuelle après le basculement.
5. Une fois que vous avez effectué la validation, tous les autres points de récupération disponibles sont supprimés. Cette étape marque la fin du basculement.

>[!TIP]
> Si vous rencontrez des problèmes de connectivité après le basculement, suivez le [guide de résolution des problèmes](site-recovery-failover-to-azure-troubleshoot.md).


## <a name="next-steps"></a>Étapes suivantes

Après le basculement, reprotégez les machines virtuelles Azure en les répliquant localement. Ensuite, une fois les machines virtuelles reprotégées et répliquées sur le site local, effectuez une restauration automatique à partir d’Azure quand vous êtes prêt.
