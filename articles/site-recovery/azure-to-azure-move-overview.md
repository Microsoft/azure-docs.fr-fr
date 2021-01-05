---
title: Déplacement de machines virtuelles Azure vers une autre région à l’aide d’Azure Site Recovery
description: Utilisation d’Azure Site Recovery pour déplacer des machines virtuelles Azure d’une région Azure vers une autre.
author: rajani-janaki-ram
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/28/2019
ms.author: rajanaki
ms.custom: MVC
ms.openlocfilehash: 61d596c4b3a65c54e1a70682adad5b7328c384f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90007364"
---
# <a name="moving-azure-vms-to-another-azure-region"></a>Déplacement de machines virtuelles Azure vers une autre région Azure

Cet article fournit une vue d’ensemble des raisons et des étapes relatives au déplacement de machines virtuelles Azure vers une autre région Azure à l’aide d’[Azure Site Recovery](site-recovery-overview.md). 


## <a name="reasons-to-move-azure-vms"></a>Raisons pouvant motiver un déplacement de machines virtuelles Azure

Vous pouvez déplacer des machines virtuelles pour les raisons suivantes :

- Vous avez déjà effectué un déploiement dans une région, et une nouvelle région plus proche des utilisateurs finaux de votre application ou service est maintenant prise en charge. Dans ce scénario, vous pourriez avoir envie de déplacer vos machines virtuelles en l’état vers cette nouvelle région afin de réduire la latence. Adoptez la même approche si vous voulez regrouper des abonnements ou si des règles de gouvernance/de l’organisation vous obligent à effectuer un tel déplacement
- Votre machine virtuelle a été déployée en tant que machine virtuelle à instance unique ou dans le cadre d’un groupe à haute disponibilité. Si vous souhaitez augmenter les contrats SLA de disponibilité, vous pouvez déplacer vos machines virtuelles dans une zone de disponibilité

## <a name="move-vms-with-resource-mover"></a>Déplacer des machines virtuelles avec Resource Mover

Vous pouvez désormais déplacer des machines virtuelles vers une autre région avec [Azure Resource Mover](../resource-mover/tutorial-move-region-virtual-machines.md). Azure Resource Mover est en préversion publique et fournit ce qui suit :
- Un hub unique pour déplacer des ressources entre les régions
- Une réduction de la complexité et du temps de déplacement. Tout ce dont vous avez besoin se trouve dans un même emplacement.
- Une expérience simple et cohérente pour le déplacement des différents types de ressources Azure.
- Un moyen simple d’identifier les dépendances des ressources que vous souhaitez déplacer. Cela vous permet de déplacer les ressources associées ensemble, pour que tout fonctionne comme prévu dans la région cible après le déplacement.
- Le nettoyage automatique des ressources dans la région source, si vous souhaitez les supprimer après le déplacement
- Tests. Vous pouvez essayer un déplacement, puis l’annuler si vous ne souhaitez pas effectuer un déplacement complet.



## <a name="move-vms-with-site-recovery"></a>Déplacer des machines virtuelles avec Site Recovery

Le déplacement de machines virtuelles avec Site Recovery implique les étapes suivantes :

1. Assurez-vous que la configuration requise est respectée.
2. Préparez les machines virtuelles sources.
3. Préparez la région cible.
4. Copiez les données vers la région cible. Utilisez la technologie de réplication Azure Site Recovery pour copier les données de la machine virtuelle source vers la région cible.
5. Testez la configuration. Une fois la réplication terminée, testez la configuration en effectuant un test de basculement vers un réseau hors production.
6. Effectuez le déplacement.
7. Supprimez les ressources dans la région source.

> [!NOTE]
> Les détails de ces étapes sont fournis dans les sections suivantes.
> [!IMPORTANT]
> Actuellement, Azure Site Recovery prend en charge le déplacement des machines virtuelles d’une région à une autre, mais pas le déplacement au sein d’une même région.

## <a name="typical-architectures-for-a-multi-tier-deployment"></a>Architectures classiques d’un déploiement multicouche

Cette section décrit les architectures de déploiement les plus courantes pour une application multicouche dans Azure. L’exemple est une application à trois couches avec une adresse IP publique. Chaque couche (web, application et base de données) comporte deux machines virtuelles et est connectée aux autres couches par le biais d’un équilibreur de charge Azure. La couche Base de données a une réplication SQL Server Always On entre les machines virtuelles à des fins de haute disponibilité.

* **Machines virtuelles à instance unique déployées sur diverses couches** : chaque machine virtuelle d’une couche est configurée en tant qu’instance unique et connectée par des équilibreurs de charge aux autres couches. Cette configuration est la plus simple à adopter.

     ![Sélection de l’option permettant de déplacer le déploiement d’une machine virtuelle à instance unique d’une couche à l’autre](media/move-vm-overview/regular-deployment.png)

* **Machines virtuelles de chaque couche déployées entre groupes à haute disponibilité** : chaque machine virtuelle d’une couche est configurée dans un groupe à haute disponibilité. Les [groupes à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md) garantissent que les machines virtuelles que vous déployez sur Azure sont distribuées sur plusieurs nœuds matériels isolés dans un cluster. Leur utilisation garantit qu’en cas de défaillance matérielle ou logicielle dans Azure, seule une partie de vos machines virtuelles est affectée tandis que votre solution globale reste disponible et opérationnelle.

     ![Déploiement de machines virtuelles entre groupes à haute disponibilité](media/move-vm-overview/avset.png)

* **Machines virtuelles de chaque couche déployées entre zones de disponibilité** : chaque machine virtuelle d’une couche est configurée dans des [zones de disponibilité](../availability-zones/az-overview.md). Une zone de disponibilité dans une région Azure est une combinaison d’un domaine d’erreur et d’un domaine de mise à jour. Par exemple, si vous créez trois ou plusieurs machines virtuelles dans trois zones d’une région Azure, vos machines virtuelles sont efficacement réparties sur trois domaines d’erreur et trois domaines de mise à jour. La plateforme Azure reconnaît cette répartition entre les domaines de mise à jour pour vous assurer que les machines virtuelles des différentes zones ne sont pas mises à jour en même temps.

     ![Déploiement de zone de disponibilité](media/move-vm-overview/zone.png)

## <a name="move-vms-as-is-to-a-target-region"></a>Déplacer des machines virtuelles en l’état vers une région cible

Selon les [architectures](#typical-architectures-for-a-multi-tier-deployment) mentionnées ci-dessus, voici ce à quoi vont ressembler les déploiements une fois que vous aurez effectué le déplacement en l’état vers la région cible.

* **Machines virtuelles à instance unique déployées sur diverses couches**
* **Machines virtuelles de chaque couche déployées entre groupes à haute disponibilité**
* **Machines virtuelles de chaque couche déployées entre zones de disponibilité**

## <a name="move-vms-to-increase-availability"></a>Déplacer des machines virtuelles pour augmenter la disponibilité

* **Machines virtuelles à instance unique déployées sur diverses couches**

     ![Déploiement de machines virtuelles à instance unique entre les couches](media/move-vm-overview/single-zone.png)

* **Machines virtuelles de chaque couche déployées entre groupes à haute disponibilité** : Vous pouvez configurer vos machines virtuelles dans un groupe à haute disponibilité dans des zones de disponibilité distinctes quand vous activez la réplication pour votre machine virtuelle à l’aide d’Azure Site Recovery. Le contrat de niveau de service (SLA) s’élèvera à 99,99 % de disponibilité une fois que vous aurez terminé l’opération de déplacement.

     ![Déploiement de machines virtuelles entre groupes à haute disponibilité et zones de disponibilité](media/move-vm-overview/aset-azone.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> 
> * [Déplacer des machines virtuelles Azure vers une autre région](azure-to-azure-tutorial-migrate.md)
> 
> * [Déplacer des machines virtuelles Azure vers des zones de disponibilité](move-azure-vms-avset-azone.md)

