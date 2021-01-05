---
title: Exclure des disques de machines virtuelles Hyper-V de la récupération d’urgence sur Azure avec Azure Site Recovery
description: Comme exclure des disques de machines virtuelles Hyper-V de la réplication sur Azure avec Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.topic: conceptual
ms.author: mayg
ms.date: 11/12/2019
ms.openlocfilehash: 373cffe96119af4a2fc0d74e2090e6cc24dcaf8f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86131254"
---
# <a name="exclude-disks-from-replication"></a>Exclure les disques de la réplication

Cet article explique comment exclure des disques pendant la réplication de machines virtuelles Hyper-V dans Azure. Vous pouvez exclure des disques de la réplication pour plusieurs raisons :

- Assurez-vous que les données sans importance perdues sur le disque exclu ne sont pas répliquées.
- Optimiser la bande passante utilisée pour la réplication ou les ressources côté cible en excluant les disques que vous n’avez pas besoin de répliquer.
- Économisez des ressources de stockage et réseau en ne répliquant pas les données dont vous n’avez pas besoin.

Avant d’exclure des disques de la réplication :

- [Apprenez-en davantage](exclude-disks-replication.md) sur l’exclusion de disques.
- Passez en revue les [scénarios d’exclusion typiques](exclude-disks-replication.md#typical-scenarios) et les [exemples](exclude-disks-replication.md#example-1-exclude-the-sql-server-tempdb-disk) qui montrent comment l’exclusion d’un disque affecte la réplication, le basculement et la restauration automatique.

## <a name="before-you-start"></a>Avant de commencer

Avant de commencer, prenez note que :

- **Réplication** : Par défaut, tous les disques d’une machine sont répliqués.
- **Type de disque** :
    - Les disques de base peuvent être exclus de la réplication.
    - Vous ne pouvez pas exclure les disques de système d’exploitation.
    - Nous vous recommandons de ne pas exclure de disques dynamiques. Site Recovery ne peut pas identifier le disque VHD qui est de type de base ou dynamique dans la machine virtuelle invitée.  Si vous n’excluez pas tous les disques de volume dynamique dépendants, le disque dynamique protégé devient un disque défectueux sur une machine virtuelle de basculement, et les données de ce disque ne sont pas accessibles.
- **Ajouter/supprimer/exclure des disques** : Une fois la réplication activée, vous ne pouvez pas ajouter/supprimer/exclure de disques pour la réplication. Si vous voulez ajouter/supprimer ou exclure un disque, vous devez désactiver la protection de la machine virtuelle, puis la réactiver.
- **Basculement** : Après le basculement, si les applications ayant basculé ont besoin d’exclure des disques pour fonctionner, vous devez créer ces disques manuellement. Vous pouvez également intégrer Azure Automation dans un plan de récupération afin de créer le disque pendant le basculement de la machine.
- **Restauration automatique** : Lorsque vous effectuez une restauration automatique sur votre site local après le basculement, les disques que vous créez manuellement dans Azure ne sont pas restaurés. Par exemple, si vous basculez trois disques et que vous en créez deux directement sur une machine virtuelle Azure, seuls trois disques qui ont été basculés sont restaurés automatiquement. Vous ne pouvez pas inclure de disques créés manuellement dans le processus de restauration automatique ou de réplication inverse de machines virtuelles.

## <a name="exclude-disks"></a>Exclure des disques

1. Pour exclure des disques lorsque vous [activez la réplication](./hyper-v-azure-tutorial.md) pour une machine virtuelle Hyper-V, après avoir sélectionné les machines virtuelles que vous souhaitez répliquer, dans la page **Activer la réplication** >  **Propriétés** > **Configurer les propriétés**, consultez la colonne **Disques à répliquer**. Par défaut, tous les disques sont sélectionnés pour la réplication.
2. Si vous ne souhaitez pas répliquer un disque spécifique, dans **Disques à répliquer** désactivez la sélection pour tous les disques que vous souhaitez exclure. 

    ![Exclure les disques de la réplication](./media/hyper-v-exclude-disk/enable-replication6-with-exclude-disk.png)


## <a name="next-steps"></a>Étapes suivantes
Une fois votre déploiement configuré et effectué, pour en savoir plus sur les différents types de basculement, [cliquez ici](failover-failback-overview.md) .
