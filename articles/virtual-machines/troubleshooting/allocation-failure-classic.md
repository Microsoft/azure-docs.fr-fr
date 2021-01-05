---
title: Résolution des problèmes d’allocation de machines virtuelles Azure dans le modèle de déploiement Classic | Microsoft Docs
description: Résoudre les problèmes d’allocation pendant la création, le redémarrage ou le redimensionnement d’une machine virtuelle classique dans Azure
services: azure-service-management
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-resource-manager,azure-service-management
ms.assetid: bb939e23-77fc-4948-96f7-5037761c30e8
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 11/01/2018
ms.author: genli
ms.openlocfilehash: 20e64e5225987a8045e406a0e8fcae098c580c61
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77913376"
---
# <a name="troubleshooting-steps-specific-to-allocation-failure-scenarios-in-the-classic-deployment-model"></a>Étapes de résolution des problèmes d’allocation dans le modèle de déploiement Classic

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Voici des scénarios courants d’allocation qui entraînent l’épinglage d’une demande d’allocation. Nous aborderons chaque scénario ultérieurement dans cet article.

- Redimensionner une machine virtuelle ou ajouter des machines virtuelles ou des instances de rôle à un service cloud existant
- Redémarrer des machines virtuelles partiellement arrêtées (désallouées)
- Redémarrer des machines virtuelles complètement arrêtées (désallouées)
- Déploiements en environnement de préproduction et de production (PaaS uniquement)
- Groupe d’affinités (proximité de la machine virtuelle ou du service)
- Réseau virtuel basé sur les groupes d’affinités

Si vous recevez une erreur d’allocation, regardez si l’un des scénarios décrits s’applique à votre erreur. Servez-vous de l’erreur d’allocation retournée par la plateforme Azure pour identifier le scénario correspondant. Si votre demande est épinglée, supprimez certaines contraintes d’épinglage pour élargir votre demande à plusieurs clusters et augmenter ainsi les chances de succès de l’allocation.
En règle générale, si l’erreur n’indique pas que la taille de la machine virtuelle demandée n'est pas prise en charge, vous pourrez réessayer ultérieurement. Cela signifie, en effet, que suffisamment de ressources ont été libérées dans le cluster pour répondre à votre demande. Si le problème est lié à la non-prise en charge de la taille de machine virtuelle demandée, essayez une autre taille de machine virtuelle. Dans le cas contraire, la seule option consiste à supprimer la contrainte d’épinglage.

Deux scénarios d’échec courants sont liés aux groupes d’affinités. Le groupe d’affinités était utilisé pour assurer une proximité étroite aux instances de machines virtuelles et de services, ou encore pour permettre la création d’un réseau virtuel. Avec l’introduction des réseaux virtuels régionaux, les groupes d’affinités ne sont plus nécessaires pour la création d’un réseau virtuel. La diminution du temps de réponse du réseau dans l’infrastructure Azure a modifié la recommandation relative à l’utilisation de groupes d’affinités pour la proximité des machines virtuelles et des services.

Le diagramme suivant présente la taxonomie des scénarios d’allocation (épinglés). 

![Taxonomie d’une allocation épinglée](./media/virtual-machines-common-allocation-failure/Allocation3.png)

## <a name="resize-a-vm-or-add-vms-or-role-instances-to-an-existing-cloud-service"></a>Redimensionner une machine virtuelle ou ajouter des machines virtuelles ou des instances de rôle à un service cloud existant
**Error**

Upgrade_VMSizeNotSupported ou GeneralError

**Cause de l’épinglage au cluster**

Une demande de redimensionnement d’une machine virtuelle ou d’ajout d’une machine virtuelle ou d’une instance de rôle à un service cloud existant doit être exécutée sur le cluster d’origine qui héberge le service cloud existant. La création d’un service cloud permet à la plateforme Azure de trouver un autre cluster disposant de ressources ou prenant en charge la taille de machine virtuelle que vous avez demandée.

**Solution de contournement**

Si l’erreur est Upgrade_VMSizeNotSupported*, essayez une autre taille de machine virtuelle. Si vous ne pouvez pas utiliser une autre taille de machine virtuelle, mais qu’il est possible d’utiliser une autre adresse IP virtuelle (VIP), créez un service cloud pour héberger la nouvelle machine virtuelle puis ajoutez le nouveau service cloud au réseau virtuel régional où sont exécutées les machines virtuelles existantes. Si votre service cloud n’utilise pas de réseau virtuel régional, vous pouvez toujours en créer un pour le nouveau service cloud, puis connecter votre [réseau virtuel à celui créé](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/). Pour plus d’informations, consultez [Réseaux virtuels régionaux](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/).

Si l’erreur est GeneralError*, le type de ressource (par exemple, une taille de machine virtuelle particulière) est probablement pris en charge par le cluster, mais ce dernier ne peut pas libérer de ressources pour le moment. Comme dans le scénario ci-dessus, ajoutez de la ressource de calcul en créant un service cloud (notez que le nouveau service cloud doit utiliser une adresse IP virtuelle différente) et utilisez un réseau virtuel régional pour connecter vos services cloud.

## <a name="restart-partially-stopped-deallocated-vms"></a>Redémarrer des machines virtuelles partiellement arrêtées (désallouées)
**Error**

GeneralError*

**Cause de l’épinglage au cluster**

Une désallocation partielle signifie que vous avez arrêté (désalloué) une ou plusieurs machines virtuelles, mais pas toutes, dans un service cloud. Quand vous arrêtez (désallouez) une machine virtuelle, les ressources associées sont libérées. Le redémarrage de cette machine virtuelle constitue donc une nouvelle demande d’allocation. Redémarrer des machines virtuelles dans un service cloud partiellement désalloué équivaut à ajouter des machines virtuelles à un service cloud existant. La demande d’allocation doit être exécutée sur le cluster d’origine qui héberge le service cloud existant. La création d’un service cloud différent permet à la plateforme Azure de trouver un autre cluster disposant de ressources ou prenant en charge la taille de machine virtuelle que vous avez demandée.

**Solution de contournement**

S’il est possible d’utiliser une autre adresse IP virtuelle, supprimez les machines virtuelles arrêtées (désallouées) tout en conservant les disques associés, puis ajoutez les machines virtuelles par le biais d’un autre service cloud. Utilisez le réseau virtuel régional pour connecter vos services cloud :

* Si votre service cloud existant utilise un réseau virtuel régional, ajoutez simplement le nouveau service cloud au même réseau virtuel.
* Si votre service cloud n’utilise pas de réseau virtuel régional, créez-en un pour le nouveau service cloud, puis connectez votre [réseau virtuel existant à celui créé](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/). Pour plus d’informations, consultez [Réseaux virtuels régionaux](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/).

## <a name="restart-fully-stopped-deallocated-vms"></a>Redémarrer des machines virtuelles complètement arrêtées (désallouées)
**Error**

GeneralError*

**Cause de l’épinglage au cluster**

Une désallocation complète signifie que vous avez arrêté (désalloué) toutes les machines virtuelles d’un service cloud. Les demandes d’allocation pour redémarrer ces machines virtuelles doivent être exécutées sur le cluster d’origine qui héberge le service cloud. La création d’un service cloud permet à la plateforme Azure de trouver un autre cluster disposant de ressources ou prenant en charge la taille de machine virtuelle que vous avez demandée.

**Solution de contournement**

S’il est possible d’utiliser une autre adresse IP virtuelle, supprimez les machines virtuelles d’origine qui ont été arrêtées (désallouées) tout en conservant les disques associés, puis supprimez le service cloud correspondant (les ressources de calcul associées ont déjà été libérées quand vous avez arrêté (désalloué) les machines virtuelles). Créez un service cloud pour ajouter les machines virtuelles.

## <a name="stagingproduction-deployments-platform-as-a-service-only"></a>Déploiements en environnement intermédiaire ou de production (PaaS uniquement)
**Error**

New_General* ou New_VMSizeNotSupported*

**Cause de l’épinglage au cluster**

Les déploiements d’un service cloud en environnement intermédiaire et en environnement de production sont hébergés dans le même cluster. Lorsque vous ajoutez le deuxième déploiement, la demande d’allocation correspondante sera exécutée au niveau du cluster qui héberge le premier déploiement.

**Solution de contournement**

Supprimez le premier déploiement et le service cloud d’origine, puis redéployez le service cloud. Cette action peut avoir pour effet de réaffecter le premier déploiement dans un cluster qui dispose de suffisamment de ressources pour les deux déploiements ou dans un cluster qui prend en charge les tailles de machine virtuelle demandées.

## <a name="affinity-group-vmservice-proximity"></a>Groupe d’affinités (proximité entre la machine virtuelle et le service)
**Error**

New_General* ou New_VMSizeNotSupported*

**Cause de l’épinglage au cluster**

Les ressources de calcul affectées à un groupe d’affinités sont liées à un seul cluster. Les nouvelles demandes de ressources de calcul portant sur ce groupe d’affinités sont exécutées dans le même cluster hébergeant les ressources existantes. Cela est vrai que les ressources soient créées par le biais d’un nouveau service cloud ou d’un service cloud existant.

**Solution de contournement**

Si un groupe d’affinités n’est pas nécessaire, n’utilisez pas de groupe d’affinités ou regroupez vos ressources de calcul dans plusieurs groupes d’affinités.

## <a name="affinity-group-based-virtual-network"></a>Réseau virtuel par groupe d’affinités
**Error**

New_General* ou New_VMSizeNotSupported*

**Cause de l’épinglage au cluster**

Avant l’introduction des réseaux virtuels régionaux, vous avez dû associer un réseau virtuel à un groupe d’affinités. Les ressources de calcul placées dans un groupe d’affinités sont donc liées aux mêmes contraintes que celles décrites dans le « scénario d’allocation : groupe d’affinités (proximité entre la machine virtuelle et le service) » ci-dessus. Les ressources de calcul sont liées à un cluster.

**Solution de contournement**

Si vous n’avez pas besoin d’un groupe d’affinités, créez un réseau virtuel régional pour les nouvelles ressources que vous souhaitez ajouter, puis [connectez votre réseau virtuel à celui créé](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/). Pour plus d’informations, consultez [Réseaux virtuels régionaux](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/).

Vous pouvez également [migrer votre réseau virtuel basé sur le groupe d’affinités vers un réseau virtuel régional](https://azure.microsoft.com/blog/2014/11/26/migrating-existing-services-to-regional-scope/), puis ajouter à nouveau les ressources souhaitées.


