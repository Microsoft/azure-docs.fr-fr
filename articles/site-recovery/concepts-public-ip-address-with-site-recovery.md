---
title: Affecter des adresses IP publiques après le basculement avec Azure Site Recovery
description: Cet article explique comment configurer des adresses IP publiques avec Azure Site Recovery et Azure Traffic Manager pour la migration et la reprise d’activité
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: mayg
ms.openlocfilehash: 01c2f61dcf024e8c9dbbd5b2ee11a479b3c16305
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86130277"
---
# <a name="set-up-public-ip-addresses-after-failover"></a>Configurer des adresses IP publiques après un basculement

Les adresses IP publiques permettent aux ressources Internet de communiquer avec les ressources Azure (communication entrante). Les adresses IP publiques permettent également aux ressources Azure de communiquer avec Internet et les services Azure publics (communication sortante) quand une adresse IP est attribuée à la ressource.
- Communication entrante d’Internet vers la ressource, par exemple les machines virtuelles Azure (VM), les passerelles Azure Application Gateway, les équilibreurs de charge Azure Load Balancer, les passerelles VPN Azure, etc. Vous pouvez toujours communiquer avec des ressources telles qu’une machine virtuelle, à partir d’Internet si aucune adresse IP publique ne lui est attribuée à condition que la machine virtuelle fasse partie d’un pool principal à équilibreur de charge et qu’une adresse IP publique soit attribuée à l’équilibreur de charge.
- Connectivité sortante vers Internet à l’aide d’une adresse IP prédictible. Par exemple, une machine virtuelle peut communiquer en sortie vers Internet sans adresse IP publique attribuée, mais son adresse fait l’objet d’une traduction d’adresse réseau par Azure en adresse publique non prédictible par défaut. L’attribution d’une adresse IP publique à une ressource vous permet de savoir quelle adresse IP est utilisée pour la connexion sortante. Bien que prédictible, l’adresse peut changer en fonction de la méthode d’attribution choisie. Pour plus d’informations, consultez [Créer une adresse IP publique](../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address). Pour en savoir plus sur les connexions sortantes des ressources Azure, consultez [Comprendre les connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Dans Azure Resource Manager, une adresse IP publique est une ressource ayant ses propres propriétés. Voici quelques unes des ressources auxquelles vous pouvez associer une ressource d’adresse IP publique :

* Interfaces réseau de machine virtuelle
* Équilibreurs de charge accessibles sur Internet
* Passerelles VPN
* Passerelles d’application

Cet article décrit comment utiliser des adresses IP publiques avec Site Recovery.

## <a name="public-ip-address-assignment-using-recovery-plan"></a>Affectation d’adresses IP publiques à l’aide d’un plan de récupération

L’adresse IP publique de l’application de production **ne peut pas être conservée lors du basculement**. Les charges de travail montées dans le cadre du processus de basculement doivent être affectées à une ressource d’adresse IP publique Azure disponible dans la région cible. Cette étape peut être effectuée manuellement, ou automatiquement avec des plans de récupération. Un plan de récupération rassemble les machines dans des groupes de récupération. Il vous aide à définir un processus de récupération systématique. Vous pouvez utiliser un plan de récupération pour imposer un ordre et automatiser les actions nécessaires à chaque étape, à l’aide de runbooks Azure Automation pour le basculement vers Azure, ou de scripts.

La configuration est la suivante :
- Créez un [plan de récupération](../site-recovery/site-recovery-create-recovery-plans.md#create-a-recovery-plan) et regroupez les charges de travail nécessaires dans ce plan.
- Personnalisez le plan en ajoutant une étape pour attacher une adresse IP publique, à l’aide de scripts de [runbooks Azure Automation](../site-recovery/site-recovery-runbook-automation.md#customize-the-recovery-plan), à la machine virtuelle basculée.

 
## <a name="public-endpoint-switching-with-dns-level-routing"></a>Routage au niveau du DNS entre les points de terminaison publics

Azure Traffic Manager permet le routage au niveau du DNS entre les points de terminaison, et vous aide à [avoir un objectif de délai de récupération plus court](../site-recovery/concepts-traffic-manager-with-site-recovery.md#recovery-time-objective-rto-considerations) dans un scénario de reprise d’activité. 

En savoir plus sur les scénarios de basculement avec Traffic Manager :
1. [Basculement de l’environnement local vers Azure](../site-recovery/concepts-traffic-manager-with-site-recovery.md#on-premises-to-azure-failover) avec Traffic Manager 
2. [Basculement d’Azure vers Azure](../site-recovery/concepts-traffic-manager-with-site-recovery.md#azure-to-azure-failover) avec Traffic Manager 

La configuration est la suivante :
- Créez un [profil Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md).
- Avec la méthode de routage **Priorité**, créez deux points de terminaison : **Principal** pour l’environnement source et **Basculement** pour Azure. **Principal** reçoit la Priorité 1 et **Basculement** reçoit la Priorité 2.
- Le point de terminaison **Principal** peut être [Azure](../traffic-manager/traffic-manager-endpoint-types.md#azure-endpoints) ou [Externe](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints) selon que votre environnement source est dans ou en dehors d’Azure.
- Le point de terminaison **Basculement** est créé comme point de terminaison **Azure**. Utilisez une **adresse IP publique statique**, car il s’agira d’un point de terminaison externe pour Traffic Manager dans l’événement d’incident.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur [Traffic Manager avec Azure Site Recovery](../site-recovery/concepts-traffic-manager-with-site-recovery.md)
- En savoir plus sur les [méthodes de routage](../traffic-manager/traffic-manager-routing-methods.md)de Traffic Manager
- En savoir plus sur les [plans de récupération](site-recovery-create-recovery-plans.md) pour automatiser le basculement d’application
