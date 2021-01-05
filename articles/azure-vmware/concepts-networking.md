---
title: Concepts – Interconnexion de réseaux
description: Découvrez des aspects clés et des cas d’utilisation de mise en réseau et d’interconnexion dans Azure VMware Solution.
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: f8e9ed143d53afe2f7a24c832c69390c6ffcb36b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91575756"
---
# <a name="azure-vmware-solution-networking-and-interconnectivity-concepts"></a>Concepts de réseau et d’interconnexion d’Azure VMware Solution

[!INCLUDE [avs-networking-description](includes/azure-vmware-solution-networking-description.md)]

Une approche utile de l’interconnexion consiste à prendre en compte les deux types d’implémentations de cloud privé Azure VMware Solution :

1. [**L’interconnexion Azure de base**](#azure-virtual-network-interconnectivity) vous permet de gérer et d’utiliser votre cloud privé avec un seul réseau virtuel dans Azure. Cette implémentation est idéale pour les évaluations ou implémentations d’Azure VMware Solution qui ne nécessitent pas d’accès à partir d’environnements locaux.

1. [**L’interconnexion complète entre un environnement local et un cloud privé**](#on-premises-interconnectivity) étend l’implémentation de base d’Azure pour inclure l’interconnexion entre des environnements locaux et des clouds privés Azure VMware Solution.
 
Cet article aborde quelques concepts clés qui établissent la mise en réseau et l’interconnexion, notamment les exigences et les limitations. Il fournit également des informations supplémentaires sur les deux types d’implémentations d’interconnexion de cloud privé Azure VMware Solution. Cet article vous fournit les informations dont vous avez besoin pour configurer votre réseau afin qu’il fonctionne correctement avec Azure VMware Solution.

## <a name="azure-vmware-solution-private-cloud-use-cases"></a>Cas d’usage des clouds privés Azure VMware Solution

Les cas d’usage des clouds privés Azure VMware Solution incluent :
- nouvelle charge de travail de machine virtuelle VMware dans le cloud ;
- Transfert de la charge de travail de machine virtuelle en rafale vers le cloud (localement vers Azure VMware Solution uniquement)
- Migration de la charge de travail de machine virtuelle vers le cloud (localement vers Azure VMware Solution uniquement)
- Reprise d’activité après sinistre (d’Azure VMware Solution vers Azure VMware Solution ou localement vers Azure VMware Solution)
- Consommation des services Azure

> [!TIP]
> Tous les cas d’usage du service Azure VMware Solution sont possibles avec la connexion de l’environnement local au cloud privé.

## <a name="azure-virtual-network-interconnectivity"></a>Interconnexion de réseau virtuel Azure

Dans l’implémentation de réseau virtuel à cloud privé, vous pouvez gérer votre cloud privé Azure VMware Solution, utiliser des charges de travail dans votre cloud privé et accéder à des services Azure via la connexion ExpressRoute. 

Le diagramme ci-dessous illustre l’interconnexion de réseaux de base établie au moment du déploiement d’un cloud privé. Celui-ci montre la mise en réseau logique basée sur ExpressRoute entre un réseau virtuel dans Azure et un cloud privé. L’interconnexion correspond à trois des principaux cas d’utilisation :
* Accès entrant à vCenter Server et à NSX-T Manager qui est accessible à partir des machines virtuelles de votre abonnement Azure et non à partir de vos systèmes locaux. 
* Accès sortant de machines virtuelles à des services Azure. 
* Accès entrant et utilisation de charges de travail exécutant un cloud privé.

:::image type="content" source="media/concepts/adjacency-overview-drawing-single.png" alt-text="Connexion de base de réseau virtuel à cloud privé" border="false":::

## <a name="on-premises-interconnectivity"></a>Interconnexion locale

Dans l’implémentation de réseau virtuel et environnement local à cloud privé complet, vous pouvez accéder à vos clouds privés Azure VMware Solution à partir d’environnements locaux. Cette implémentation est une extension de l’implémentation de base décrite dans la section précédente. À l’instar de l’implémentation de base, un circuit ExpressRoute est requis, mais, dans ce cas-ci, il est utilisé pour se connecter à votre cloud privé dans Azure à partir d’environnements locaux. 

Le diagramme ci-dessous illustre l’interconnexion d’un environnement local à un cloud privé, ce qui permet les cas d’usage suivants :
* Croisement à chauf/à froid – vCenter-vMotion
* Accès pour la gestion de cloud privé d’un emplacement local à Azure VMware Solution

:::image type="content" source="media/concepts/adjacency-overview-drawing-double.png" alt-text="Connexion de base de réseau virtuel à cloud privé" border="false":::

Pour une interconnexion complète à votre cloud privé, activez ExpressRoute Global Reach, puis demandez une clé d’autorisation et un ID de Peering privé pour Global Reach dans le portail Azure. La clé d’autorisation et l’ID de Peering sont utilisés pour établir un service Global Reach entre un circuit ExpressRoute faisant partie de votre abonnement et le circuit ExpressRoute destiné à votre nouveau cloud privé. Une fois liés, les deux circuits ExpressRoute acheminent le trafic réseau entre vos environnements locaux et votre cloud privé.  Pour connaître les procédures de demande et d’utilisation de la clé d’autorisation et de l’ID de Peering, consultez le [didacticiel relatif à la création d’un Peering ExpressRoute Global Reach vers un cloud privé](tutorial-expressroute-global-reach-private-cloud.md).



## <a name="next-steps"></a>Étapes suivantes 
Découvrez [les concepts de stockage de cloud privé](concepts-storage.md).


<!-- LINKS - external -->
[enable Global Reach]: ../expressroute/expressroute-howto-set-global-reach.md

<!-- LINKS - internal -->

