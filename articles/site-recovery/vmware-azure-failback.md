---
title: Restaurer automatiquement des serveurs physiques/machines virtuelles VMware à partir d’Azure avec Azure Site Recovery
description: Découvrez comment effectuer une restauration automatique sur le site local après un basculement vers Azure, pendant la reprise d’activité de machines virtuelles VMware et serveurs physiques sur Azure.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.date: 01/15/2019
ms.topic: conceptual
ms.author: mayg
ms.openlocfilehash: aed015b67aa36e7678b31d7f2f047cb1e77c6a3d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004192"
---
# <a name="fail-back-vmware-vms-to-on-premises-site"></a>Restaurer automatiquement des machines virtuelles VMware sur un site local

Cet article explique comment restaurer automatiquement des machines virtuelles Azure sur un site local suite au [basculement](site-recovery-failover.md) de machines virtuelles locales vers Azure avec [Azure Site Recovery](site-recovery-overview.md). Après la restauration automatique en local, vous activez la réplication afin que les machines virtuelles locales démarrent la réplication vers Azure.

## <a name="before-you-start"></a>Avant de commencer

1. Découvrez la [restauration automatique VMware](failover-failback-overview.md#vmwarephysical-reprotectionfailback). 
2. Veillez à passer en revue et à effectuer les étapes pour [préparer la restauration automatique](vmware-azure-prepare-failback.md), et à ce que tous les composants requis soient déployés. Les composants incluent un serveur de processus dans Azure, un serveur cible maître local et une connexion de site à site VPN (ou un peering privé ExpressRoute) pour la restauration automatique.
3. Vérifiez que vous avez répondu aux [exigences](vmware-azure-reprotect.md#before-you-begin) pour la reprotection et la restauration automatique, et que vous avez [activé la reprotection](vmware-azure-reprotect.md#enable-reprotection) des machines virtuelles Azure, afin qu’elles soient répliquées à partir d’Azure vers le site local. Les machines virtuelles doivent être dans un état répliqué afin de pouvoir effectuer une restauration automatique.




## <a name="run-a-failover-to-fail-back"></a>Exécuter un basculement pour effectuer une restauration automatique

1. Assurez-vous que les machines virtuelles Azure sont reprotégées et répliquées sur le site local.
    - Une machine virtuelle a besoin d’au moins un point de récupération pour pouvoir être restaurée.
    - Si vous restaurez un plan de récupération, tous les ordinateurs du plan doivent avoir au moins un point de récupération.
2. Dans le coffre > **Éléments répliqués**, sélectionnez la machine virtuelle. Cliquez avec le bouton droit sur la machine virtuelle > **Basculement non planifié**.
3. Dans **Confirmer le basculement**, vérifiez la direction du basculement (depuis Azure).
4. Sélectionnez le point de récupération à utiliser pour le basculement.
    - Nous vous recommandons d’utiliser le **dernier** point de récupération. Le point de cohérence d’application se situe derrière le point le plus récent et entraîne une perte de données.
    - Le **dernier** point de récupération est cohérent en cas d’incident.
    - Avec le **dernier** point de récupération, une machine virtuelle bascule vers son dernier point dans le temps disponible. Si vous disposez d’un groupe de réplication pour assurer la cohérence de plusieurs machines virtuelles dans un plan de récupération, chaque machine virtuelle du groupe bascule vers son point indépendant le plus récent dans le temps.
    - Si vous utilisez un point de récupération de cohérence d’application, chaque machine virtuelle est restaurée sur son dernier point disponible. Si un plan de récupération a un groupe de réplication, chaque groupe est récupéré sur son point de récupération disponible commun.
5. Le basculement commence. Site Recovery arrête les machines virtuelles Azure.
6. Une fois le basculement terminé, vérifiez que tout fonctionne comme prévu. Vérifiez que les machines virtuelles Azure sont arrêtées. 
7. Une fois que vous avez tout vérifié, cliquez avec le bouton droit sur la machine virtuelle > **Valider** pour terminer le processus de basculement. La validation supprime la machine virtuelle Azure basculée. 

> [!NOTE]
> Pour les machines virtuelles Windows, Site Recovery désactive les outils VMware pendant le basculement. Lors de la restauration automatique de la machine virtuelle Windows, les outils VMware sont réactivés. 




## <a name="reprotect-from-on-premises-to-azure"></a>Procéder à une reprotection de l’environnement local vers Azure

Après la validation de la restauration automatique, les machines virtuelles Azure sont supprimées. La machine virtuelle est de nouveau sur le site local, mais elle n’est pas protégée. Pour redémarrer la réplication des machines virtuelles vers Azure, procédez comme suit :

1. Dans le coffre > **Éléments répliqués**, sélectionnez les machines virtuelles restaurées automatiquement, puis **Reprotéger**.
2. Spécifiez le serveur de processus qui est utilisé pour renvoyer des données à Azure.
3. Sélectionnez **OK** pour commencer le travail de reprotection.

> [!NOTE]
> Après le démarrage d’une machine virtuelle locale, l’agent prend jusqu’à 15 minutes pour se réinscrire auprès du serveur de configuration. Pendant ce temps, la reprotection échoue et renvoie un message d’erreur indiquant que l’agent n’est pas installé. Si cela se produit, patientez quelques minutes et relancez la reprotection.

## <a name="next-steps"></a>Étapes suivantes

Une fois la tâche de reprotection terminée, la machine virtuelle locale est répliquée vers Azure. Si nécessaire, vous pouvez [exécuter un autre basculement](site-recovery-failover.md) vers Azure.

