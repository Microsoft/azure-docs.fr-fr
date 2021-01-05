---
title: Exclure des disques de machines virtuelles VMware de la récupération d’urgence sur Azure avec Azure Site Recovery
description: Comme exclure des disques de machines virtuelles VMware de la réplication sur Azure avec Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.date: 12/10/2019
ms.author: mayg
ms.topic: conceptual
ms.openlocfilehash: c4842172ff181b5cdbe7f6fecf69da8755ae43fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86129871"
---
# <a name="exclude-disks-from-vmware-vm-replication-to-azure"></a>Exclure des disques de la réplication de machines virtuelles VMware sur Azure

Cet article explique comment exclure des disques pendant la réplication de machines virtuelles VMware dans Azure pour la récupération d'urgence. Vous pouvez exclure des disques de la réplication pour plusieurs raisons :

- Assurez-vous que les données sans importance perdues sur le disque exclu ne sont pas répliquées.
- Optimiser la bande passante utilisée pour la réplication ou les ressources côté cible en excluant les disques que vous n’avez pas besoin de répliquer.
- Économisez des ressources de stockage et réseau en ne répliquant pas les données dont vous n’avez pas besoin.

Avant d’exclure des disques de la réplication :

- [Apprenez-en davantage](exclude-disks-replication.md) sur l’exclusion de disques.
- Passez en revue les [scénarios d’exclusion typiques](exclude-disks-replication.md#typical-scenarios) et les [exemples](exclude-disks-replication.md#example-1-exclude-the-sql-server-tempdb-disk) qui montrent comment l’exclusion d’un disque affecte la réplication, le basculement et la restauration automatique.

## <a name="before-you-start"></a>Avant de commencer

 Avant de commencer, prenez note que :

- **Réplication** : Par défaut, tous les disques d’une machine sont répliqués.
- **Type de disque** : Seuls les disques de base peuvent être exclus de la réplication. Vous ne pouvez pas exclure de système d’exploitation ni de disque dynamique.
- **Service Mobility** : Pour exclure un disque de la réplication, vous devez installer manuellement le service Mobility sur la machine avant d’activer la réplication. Vous ne pouvez pas utiliser l’installation push, car cette méthode n’installe le service Mobility sur une machine virtuelle que lorsque la réplication est activée.  
- **Ajouter/supprimer/exclure des disques** : Une fois la réplication activée, vous ne pouvez pas ajouter/supprimer/exclure de disques pour la réplication. Si vous voulez ajouter/supprimer ou exclure des disques, vous devez désactiver la protection de la machine et la réactiver ensuite.
- **Basculement** : Après le basculement, si les applications ayant basculé ont besoin de disques exclus pour fonctionner, vous devez créer ces disques manuellement. Vous pouvez également intégrer Azure Automation dans un plan de récupération afin de créer le disque pendant le basculement de la machine.
- **Restauration automatique Windows** : Lorsque vous effectuez une restauration automatique sur votre site local après le basculement, les disques Windows que vous créez manuellement dans Azure ne sont pas restaurés. Par exemple, si vous basculez trois disques et que vous en créez deux directement sur des machines virtuelles Azure, seuls les trois disques qui ont été basculés seront restaurés automatiquement.
- **Restauration automatique Linux** : Pour la restauration automatique de machines Linux, les disques que vous créez manuellement dans Azure sont restaurés automatiquement. Par exemple, si vous basculez trois disques et créez deux disques directement sur des machines virtuelles Azure, les cinq disques seront restaurés. Vous ne pouvez pas exclure de disques créés manuellement pendant la restauration automatique ou la reprotection des machines virtuelles.



## <a name="exclude-disks-from-replication"></a>Exclure les disques de la réplication

1. Lorsque vous [activez la réplication](./hyper-v-azure-tutorial.md) pour une machine virtuelle VMware, après avoir sélectionné les machines virtuelles que vous souhaitez répliquer, dans la page **Activer la réplication** >  **Propriétés** > **Configurer les propriétés**, consultez la colonne **Disques à répliquer**. Par défaut, tous les disques sont sélectionnés pour la réplication.
2. Si vous ne souhaitez pas répliquer un disque spécifique, dans **Disques à répliquer** désactivez la sélection pour tous les disques que vous souhaitez exclure. 

    ![Exclure les disques de la réplication](./media/vmware-azure-exclude-disk/enable-replication-exclude-disk1.png)



## <a name="next-steps"></a>Étapes suivantes
Une fois votre déploiement configuré et effectué, pour en savoir plus sur les différents types de basculement, [cliquez ici](failover-failback-overview.md) .
