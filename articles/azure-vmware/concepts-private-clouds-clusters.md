---
title: Concepts – Clusters et clouds privés
description: Découvrez les principales fonctionnalités des centres de données à définition logicielle Azure VMware Solution et des clusters vSphere.
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: 7f8508abed3446c58a683454bc49ec7dbb49d051
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825098"
---
#  <a name="azure-vmware-solution-private-cloud-and-cluster-concepts"></a>Concepts de cloud privé et de cluster Azure VMware Solution

Azure VMware Solution fournit des clouds privés basés sur VMware dans Azure. Les clouds privés contiennent des clusters créés avec des hôtes Azure nus dédiés. Ils sont déployés et gérés via le portail Azure, l’interface CLI ou PowerShell.  Les clusters approvisionnés dans des clouds privés incluent les logiciels VMware vSphere, vCenter, vSAN et NSX. Les déploiements de logiciels et composants matériels de cloud privé Azure VMware Solution sont entièrement intégrés et automatisés dans Azure.

Il existe une relation logique entre les abonnements Azure, les clouds privés Azure VMware Solution, les clusters vSAN et les hôtes. Le diagramme montre un abonnement Azure avec deux clouds privés représentant l’environnement de développement et de production.  Chacun de ces clouds privés comprend deux clusters. 

Cet article décrit tous ces concepts.

![Image de deux clouds privés dans un abonnement client](./media/hosts-clusters-private-clouds-final.png)

>[!NOTE]
>En raison des besoins potentiels moindres d’un environnement de développement, utilisez des clusters plus petits avec des hôtes de capacité inférieure. 

## <a name="private-clouds"></a>Clouds privés

Les clouds privés contiennent des clusters vSAN créés avec des hôtes Azure nus dédiés. Chaque cloud privé peut avoir plusieurs clusters gérés par les mêmes serveur vCenter et gestionnaire NSX-T. Vous pouvez déployer et gérer des clouds privés dans le portail, l’interface CLI ou PowerShell. 

Comme avec d’autres ressources, les clouds privés sont installés et gérés à partir d’un abonnement Azure. Le nombre de clouds privés au sein d’un abonnement est évolutif. Au départ, il existe une limite d’un cloud privé par abonnement.

## <a name="clusters"></a>Clusters
Pour chaque cloud privé créé, il existe un cluster vSAN par défaut. Vous pouvez ajouter, supprimer et mettre à l’échelle des clusters à l’aide du portail Azure ou via l’API.  Tous les clusters ont une taille par défaut de trois hôtes et peuvent se mettre à l’échelle pour inclure jusqu’à 16 hôtes.  Les hôtes utilisés dans un cluster doivent être du même type.

Les clusters d’évaluation sont disponibles pour évaluation et limités à trois hôtes. Il existe un cluster d’évaluation unique par cloud privé. Vous pouvez mettre à l’échelle un cluster d’évaluation à l’aide d’un seul hôte au cours de la période d’évaluation.

Vous utilisez vSphere et NSX-T Manager pour gérer la plupart des autres aspects de la configuration ou de l’exploitation du cluster. Tout le stockage local de chaque hôte dans un cluster est contrôlé par le logiciel vSAN.

## <a name="hosts"></a>Hôtes

Les clusters de cloud privé Azure VMware Solution utilisent des hôtes d’infrastructure nus hyper-convergés. Le tableau suivant indique les capacités de mémoire RAM, de processeur et de disque de l’ordinateur hôte. 

| Type d’hôte              |             UC             |   RAM (Go)   |  Niveau de cache du vSAN NVMe (To, RAW)  |  Niveau de capacité du vSAN SSD (To, RAW)  |
| :---                   |            :---:            |    :---:     |               :---:              |                :---:               |
| Haut de gamme (HE)          |  Deux processeurs Intel 18 cœurs cadencés à 2,3 GHz  |     576      |                3.2               |                15,20               |

Les hôtes utilisés pour créer ou mettre à l’échelle des clusters proviennent d’un pool isolé d’hôtes. Ces hôtes ont passé des tests matériels et toutes leurs données ont été effacées de manière sécurisée. 

## <a name="vmware-software-versions"></a>Versions des logiciels VMware

[!INCLUDE [vmware-software-versions](includes/vmware-software-versions.md)]


## <a name="host-maintenance-and-lifecycle-management"></a>Maintenance de l’hôte et gestion du cycle de vie

La maintenance et la gestion du cycle de vie de l’hôte n’ont aucune incidence sur la capacité ou les performances des clusters du cloud privé.  Les mises à niveau de microprogramme et la réparation ou le remplacement de matériel sont des exemples de maintenance automatisée de l’hôte.

Microsoft est responsable de la gestion du cycle de vie d’appliances NSX-T, telles que NSX-T Manager et NSX-T Edge. Microsoft est également responsable de l’amorçage de la configuration réseau, par exemple, la création de la passerelle de niveau 0 et l’activation du routage Nord-Sud. Vous êtes responsable de la configuration du SDN NSX-T. Par exemple, des segments réseau, des règles de pare-feu distribuées, des passerelles de niveau 1 et des équilibreurs de charge.

> [!IMPORTANT]
> Ne modifiez pas la configuration de NSX-T Edge ou de la passerelle de niveau 0, car cela peut entraîner une perte de service.

## <a name="backup-and-restoration"></a>Sauvegarde et restauration

Les configurations de vCenter et de NSX-T sont effectuées sur une planification de sauvegarde horaire.  Les sauvegardes sont conservées pendant trois jours. Si vous devez effectuer une restauration à partir d’une sauvegarde, ouvrez une [demande de support](https://rc.portal.azure.com/#create/Microsoft.Support) dans le portail Azure pour demander une restauration.

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante consiste à découvrir les [concepts de mise en réseau et d’interconnexion](concepts-networking.md).

<!-- LINKS - internal -->

<!-- LINKS - external-->
[VCSA versions]: https://kb.vmware.com/s/article/2143838
[ESXi versions]: https://kb.vmware.com/s/article/2143832
[vSAN versions]: https://kb.vmware.com/s/article/2150753

