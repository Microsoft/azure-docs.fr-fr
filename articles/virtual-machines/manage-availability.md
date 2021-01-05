---
title: Gérer la disponibilité des machines virtuelles
description: Apprendre à utiliser plusieurs machines virtuelles afin de garantir la haute disponibilité de vos applications dans Azure
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cynthn
ms.openlocfilehash: 4dd15df0e745a5c6e3130233e693bbdb22655775
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500442"
---
# <a name="manage-the-availability-of-linux-virtual-machines"></a>Gestion de la disponibilité des machines virtuelles Linux

Découvrez comment configurer et gérer plusieurs machines virtuelles afin de garantir une haute disponibilité pour votre application Linux dans Azure. 


## <a name="understand-vm-reboots---maintenance-vs-downtime"></a>Comprendre les redémarrages des machines virtuelles : maintenance et temps d’arrêt
Il existe trois scénarios pouvant affecter une machine virtuelle dans Azure : maintenance matérielle non planifiée, temps d’arrêt imprévu et maintenance planifiée.

* Les **événements de maintenance matérielle non planifiée** surviennent lorsque la plateforme Azure prédit qu’une défaillance du matériel ou d’un composant de la plateforme, associé à une machine physique, est sur le point de se produire. Lorsque la plateforme prédit une défaillance, elle génère un événement de maintenance matérielle non prévue afin de réduire l’impact sur les machines virtuelles hébergées sur ce matériel. Azure utilise la technologie [Migration dynamique](./maintenance-and-updates.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json%252c%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json%253ftoc%253d%2fazure%2fvirtual-machines%2flinux%2ftoc.json) pour migrer les machines virtuelles depuis le matériel défectueux vers une machine physique intègre. La migration dynamique est une opération de préservation des machines virtuelles qui interrompt la machine virtuelle pendant une courte période. La mémoire, les fichiers ouverts et les connexions réseau sont conservés, mais les performances peuvent être réduites avant et/ou après l’événement. Dans les cas où la migration dynamique ne peut pas être utilisée, la machine virtuelle subit des temps d’arrêt imprévus, comme décrit ci-dessous.


* On parle de **temps d’arrêt inattendu** lorsque le matériel ou l’infrastructure physique de l’ordinateur virtuel échoue de manière inattendue. Cela comprend les défaillances du réseau local, du disque local ou au niveau du rack. Lorsqu’une défaillance de ce type est détectée, la plateforme Azure migre automatiquement (répare) votre machine virtuelle vers une machine physique saine dans le même centre de données. Lors de la procédure de réparation, les machines virtuelles subissent des temps d’arrêt (redémarrage) et, dans certains cas, une perte du lecteur temporaire. Le système d’exploitation attaché et les disques de données sont toujours conservés.

  Les machines virtuelles peuvent éventuellement subir un temps d’arrêt en cas de panne ou de catastrophe qui affectent l’ensemble d’un centre de données ou d’une région. Dans ce genre de scénarios, Azure offre des options de protection pour les [zones de disponibilité](../availability-zones/az-overview.md) et les [régions jumelées](regions.md#region-pairs).

* Les **événements de maintenance planifiés** sont des mises à jour périodiques effectuées par Microsoft sur la plateforme sous-jacente Azure afin d’améliorer la fiabilité, les performances et la sécurité de l’infrastructure de la plateforme hébergeant vos machines virtuelles. La plupart de ces mises à jour se déroulent sans aucune incidence sur les machines virtuelles ou les services cloud (voir [Maintenance ne nécessitant pas de redémarrage](maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot)). La plateforme Azure s’efforce d’utiliser la maintenance de conservation des machines virtuelles lorsque cela est possible. Toutefois, dans quelques rares cas, ces mises à jour nécessitent le redémarrage de votre machine virtuelle afin d’appliquer les mises à jour requises sur l’infrastructure sous-jacente. Dans ce cas, vous pouvez effectuer la maintenance planifiée Azure via l’opération de maintenance-redéploiement, en lançant la maintenance pour les machines virtuelles dans la période de temps adaptée. Pour plus d’informations, consultez [Maintenance planifiée pour les machines virtuelles](maintenance-and-updates.md).


Pour réduire l'effet des interruptions de service dues à un ou plusieurs de ces événements, nous vous recommandons les meilleures pratiques suivantes pour la haute disponibilité de vos machines virtuelles :

* Utiliser Zones de disponibilité pour se protéger contre les défaillances au niveau du centre de données
* Configuration de plusieurs machines virtuelles dans un groupe à haute disponibilité pour assurer la redondance
* Utilisation de disques managés pour les machines virtuelles dans le groupe à haute disponibilité
* Utiliser des événements planifiés pour répondre de façon proactive aux événements qui impactent les machines virtuelles
* Configuration de chaque couche application dans des groupes à haute disponibilité séparés
* Combiner un équilibreur de charge avec des zones de disponibilité ou des groupes à haute disponibilité
* Utiliser les zones de disponibilité pour se protéger contre les défaillances au niveau du centre de données

## <a name="use-availability-zones-to-protect-from-datacenter-level-failures"></a>Utiliser les zones de disponibilité pour se protéger contre les défaillances au niveau du centre de données

Les [zones de disponibilité](../availability-zones/az-overview.md) élargissent le niveau de contrôle dont vous disposez pour maintenir la disponibilité des applications et des données sur vos machines virtuelles. Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Pour garantir la résilience, il existe un minimum de trois zones distinctes dans toutes les régions activées. La séparation physique des Zones de disponibilité dans une région protège les applications et les données des défaillances dans le centre de données. Les services redondants interzone répliquent vos applications et données entre des Zones de disponibilité pour les protéger contre des points uniques de panne.

Au sein d'une région Azure, une zone de disponibilité combine un **domaine d'erreur** et un **domaine de mise à jour**. Par exemple, si vous créez trois ou plusieurs machines virtuelles dans trois zones d’une région Azure, vos machines virtuelles sont efficacement réparties sur trois domaines d’erreur et trois domaines de mise à jour. La plateforme Azure reconnaît cette répartition entre les domaines de mise à jour pour vous assurer que les machines virtuelles des différentes zones ne sont pas mises à jour en même temps.

Avec les Zones de disponibilité, Azure propose des contrats de niveau de service de durée de fonctionnement des machines virtuelles de pointe de 99,99 %. En concevant vos solutions pour répliquer des machines virtuelles dans des zones, vous pouvez protéger vos applications et vos données de la perte d'un centre de données. Si une zone est compromise, les applications et données répliquées sont instantanément rendues disponibles dans une autre zone.

![Zones de disponibilité](./media/virtual-machines-common-manage-availability/three-zones-per-region.png)

Apprenez-en davantage sur le déploiement d’une machine virtuelle [Windows](./windows/create-powershell-availability-zone.md) ou [Linux](./linux/create-cli-availability-zone.md) dans une zone de disponibilité.

## <a name="configure-multiple-virtual-machines-in-an-availability-set-for-redundancy"></a>Configuration de plusieurs machines virtuelles dans un groupe à haute disponibilité pour assurer la redondance
Les groupes à haute disponibilité constituent une autre configuration de centre de données destinée à assurer la redondance et la disponibilité des machines virtuelles. Cette configuration au sein d’un centre de données assure la disponibilité d’au moins une des machines virtuelles pendant un événement de maintenance planifié ou non, avec le niveau de 99,95 % stipulé dans le contrat de niveau de service (SLA) Azure. Pour plus d’informations, consultez le [SLA pour Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

> [!IMPORTANT]
> Une seule machine virtuelle à instance unique dans un groupe à haute disponibilité doit utiliser le SSD Premium ou le disque Ultra pour tous les disques de système d’exploitation et disques de données afin de bénéficier de la connectivité d’au moins 99,9 % du SLA pour les machines virtuelles. 
> 
> Une machine virtuelle à instance unique avec un SSD Standard aura un contrat SLA d’au moins 99,5 %, tandis qu’une machine virtuelle à instance unique avec un HDD Standard aura un contrat SLA d’au moins 95 %.  Voir [Contrat SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/)

Chaque machine virtuelle de votre groupe à haute disponibilité se voit attribuer un **domaine de mise à jour** et un **domaine d’erreur** par la plateforme Azure sous-jacente. Pour un groupe à haute disponibilité donné, cinq domaines de mise à jour non configurables par l’utilisateur sont affectés par défaut (les déploiements Resource Manager peuvent ensuite être étendus à 20 domaines de mise à jour) pour indiquer les groupes de machines virtuelles et les équipements physiques sous-jacents qui peuvent être redémarrés simultanément. Dans le cas où un seul groupe à haute disponibilité comprend plus de cinq machines virtuelles, la sixième machine est placée dans le même domaine de mise à jour que la première, la septième dans le même que la deuxième, etc. Le redémarrage des domaines de mise à jour peut ne pas suivre un ordre séquentiel au cours de la maintenance planifiée, mais un seul domaine de mise à jour peut être redémarré à la fois. Un domaine de mise à jour redémarré bénéficie de 30 minutes pour récupérer avant que la maintenance ne soit lancée sur un autre domaine de mise à jour.

Les domaines d’erreur définissent le groupe de machines virtuelles partageant une source d’alimentation et un commutateur réseau communs. Par défaut, les machines virtuelles configurées dans votre groupe à haute disponibilité sont réparties entre trois domaines d’erreur au maximum pour les déploiements Resource Manager (deux domaines d’erreur pour les déploiements Classic). Le fait de placer vos machines virtuelles dans un groupe à haute disponibilité ne protège pas vos applications des défaillances du système d’exploitation ou propres aux applications, mais limite l’effet des défaillances des équipements physiques, des pannes du serveur et des coupures d’électricité.

<!--Image reference-->
   ![Schéma conceptuel de la configuration du domaine de mise à jour et du domaine d’erreur](./media/virtual-machines-common-manage-availability/ud-fd-configuration.png)

## <a name="use-managed-disks-for-vms-in-an-availability-set"></a>Utilisation de disques managés pour les machines virtuelles dans le groupe à haute disponibilité
Si vous utilisez actuellement des machines virtuelles avec des disques non managés, nous vous recommandons fortement de passer à des disques managés pour [Linux](./linux/convert-unmanaged-to-managed-disks.md) et [Windows](./windows/convert-unmanaged-to-managed-disks.md).

Les [disques managés](./managed-disks-overview.md) augmentent la fiabilité des groupes à haute disponibilité en garantissant que les disques des machines virtuelles d’un groupe à haute disponibilité sont suffisamment isolés les uns des autres, ceci pour éviter les points de défaillance uniques. Ceci se fait en plaçant automatiquement les disques dans des domaines d’erreur de stockage différents (clusters de stockage) et en les alignant avec le domaine d’erreur de machine virtuelle. Si un domaine d’erreur de stockage est mis en échec en raison d’une défaillance matérielle ou logicielle, seule l’instance de machine virtuelle possédant des disques sur le domaine d’erreur de stockage est mise en échec.
![Domaines d’erreurs de disques managés](./media/virtual-machines-common-manage-availability/md-fd-updated.png)

> [!IMPORTANT]
> Le nombre de domaines d’erreur des groupes à haute disponibilité gérés varie selon la région : de deux à trois par région. Vous pouvez voir le domaine d’erreur de chaque région en exécutant les scripts suivants.

```azurepowershell-interactive
Get-AzComputeResourceSku | where{$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'}
```

```azurecli-interactive 
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location, MaximumFaultDomainCount:capabilities[0].value}' -o Table
```

> [!NOTE]
> Dans certains cas, deux machines virtuelles d’un même groupe à haute disponibilité peuvent partager un domaine d’erreur. Vous pouvez vérifier si un domaine d’erreur est partagé en accédant à votre groupe à haute disponibilité et en regardant la colonne **Domaine d’erreur**. Vous pouvez avoir obtenu le partage d’un domaine d’erreur si vous avez effectué cette série d’étapes lors du déploiement des machines virtuelles :
> 1. Déploiement de la première machine virtuelle
> 1. Arrêt ou libération de la première machine virtuelle
> 1. Déploiement de la deuxième machine virtuelle
>
> Dans ces circonstances, le disque du système d’exploitation de la deuxième machine virtuelle peut être créé dans le même domaine d’erreur que la première machine virtuelle, de sorte que les deux machines virtuelles se retrouvent dans le même domaine d’erreur. Pour éviter ce problème, il est recommandé de ne pas arrêter ni libérer les machines virtuelles entre deux déploiements.

Si vous prévoyez d’utiliser des machines virtuelles avec des disques non managés, suivez les meilleures pratiques ci-dessous pour les comptes de stockage sur lesquels les disques durs virtuels (VHD) d’ordinateurs virtuels sont stockés en tant [qu’objets blob de pages](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs#about-page-blobs).

1. **Conservez tous les disques (système d’exploitation et données) associés à une machine virtuelle dans le même compte de stockage.**
2. **Examinez les [limites](../storage/blobs/scalability-targets-premium-page-blobs.md) sur le nombre de disques non managés dans un compte de stockage Azure** avant d’ajouter plus de disques durs virtuels à un compte de stockage.
3. **Utilisez un compte de stockage distinct pour chaque machine virtuelle d’un groupe à haute disponibilité.** Ne partagez pas de comptes de stockage avec plusieurs machines virtuelles d’un même groupe à haute disponibilité. Il est acceptable pour les machines virtuelles de différents groupes à haute disponibilité de partager des comptes de stockage si les meilleures pratiques ci-dessus sont suivies ![FDs de disques non managés](./media/virtual-machines-common-manage-availability/umd-updated.png)

## <a name="use-scheduled-events-to-proactively-respond-to-vm-impacting-events"></a>Utiliser des événements planifiés pour répondre de façon proactive aux événements qui impactent les machines virtuelles

Quand vous vous abonnez à des [événements planifiés](./linux/scheduled-events.md), votre machine virtuelle reçoit une notification relative aux futurs événements de maintenance susceptibles de l’impacter. Quand des événements planifiés sont activés, votre machine virtuelle dispose d’un délai minimal avant l’exécution de l’activité de maintenance. Par exemple, les mises à jour du système d’exploitation hôte susceptibles d’impacter votre machine virtuelle sont mises en file d’attente en tant qu’événements indiquant l’impact ainsi que l’heure d’exécution de la maintenance si aucune action n’est entreprise. Les événements de planification sont également mis en file d’attente quand Azure détecte une défaillance matérielle imminente pouvant impacter votre machine virtuelle, ce qui vous permet de décider du moment de la réparation. Les clients peuvent utiliser l’événement pour effectuer des tâches préalables à la maintenance, par exemple une sauvegarde de l’état, un basculement vers le nœud secondaire, etc. Une fois que vous avez fini de mettre au point votre logique de gestion de l’événement de maintenance, vous pouvez approuver l’événement planifié en attente pour permettre à la plateforme de procéder à la maintenance.


## <a name="combine-a-load-balancer-with-availability-zones-or-sets"></a>Combiner un équilibreur de charge avec des zones de disponibilité ou des groupes à haute disponibilité
Combinez [Azure Load Balancer](../load-balancer/load-balancer-overview.md) avec une zone de disponibilité ou un groupe à haute disponibilité pour une meilleure résilience de votre application. L'équilibrage de charge Azure répartit le trafic entre plusieurs machines virtuelles. L'équilibrage de charge Azure est compris pour nos machine virtuelles de niveau Standard. Certains niveaux de machines virtuelles n’intègrent pas Azure Load Balancer. Pour plus d’informations sur l’équilibrage de charge des machines virtuelles, consultez **Équilibrage de charge des machines virtuelles** pour [Linux](linux/tutorial-load-balancer.md) ou [Windows](windows/tutorial-load-balancer.md).

Si l’équilibrage de charge n’est pas configuré pour équilibrer le trafic entre plusieurs machines virtuelles, tout événement de maintenance planifié affecte l’unique machine virtuelle en charge du trafic, entraînant ainsi une interruption de votre couche Application. Placer plusieurs machines virtuelles de la même couche dans le même équilibrage de charge et groupe à haute disponibilité permet de toujours avoir au moins une instance disponible pour le trafic.

Pour obtenir un didacticiel sur la façon d’équilibrer la charge entre les zones de disponibilité, consultez [Didacticiel : Équilibrer la charge des machines virtuelles sur les zones de disponibilité avec un équilibreur de charge Standard Load Balancer à l’aide du portail Azure](../load-balancer/tutorial-load-balancer-standard-public-zone-redundant-portal.md).


## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur l’équilibrage de charge de vos machines virtuelles, consultez la rubrique concernant [l’équilibrage de charge des machines virtuelles](../load-balancer/load-balancer-overview.md).
