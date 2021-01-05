---
title: À propos de l’utilisation d’ExpressRoute avec Azure Site Recovery
description: Cet article décrit comment utiliser Azure ExpressRoute avec Azure Site Recovery pour la reprise d’activité et la migration.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/13/2019
ms.author: mayg
ms.openlocfilehash: 99fa8d4cf8f48d0fe72da36baef20c83add438c0
ms.sourcegitcommit: 46c5ffd69fa7bc71102737d1fab4338ca782b6f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94330255"
---
# <a name="azure-expressroute-with-azure-site-recovery"></a>Azure ExpressRoute avec Azure Site Recovery

Microsoft Azure ExpressRoute vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité. Avec ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, par exemple Microsoft Azure, Microsoft 365 et Dynamics 365.

Cet article décrit comment utiliser Azure ExpressRoute avec Azure Site Recovery pour la récupération d’urgence et la migration.

## <a name="expressroute-circuits"></a>Circuits ExpressRoute

Un circuit ExpressRoute représente une connexion logique entre votre infrastructure locale et des services de cloud computing Microsoft via un fournisseur de connexion. Vous pouvez commander plusieurs circuits ExpressRoute. Chaque circuit peut se trouver dans une région identique ou différente des autres, et peut être connecté à votre site via des fournisseurs de connectivité. Pour en savoir plus sur les circuits ExpressRoute, [voir ici](../expressroute/expressroute-circuit-peerings.md).

Un circuit ExpressRoute dispose de plusieurs domaines de routage qui lui sont associés. Pour en savoir plus et comparer les domaines de routage ExpressRoute, [voir ici](../expressroute/expressroute-circuit-peerings.md#peeringcompare).

## <a name="on-premises-to-azure-replication-with-expressroute"></a>Réplication de service local sur Azure avec ExpressRoute

Azure Site Recovery permet une récupération d’urgence et une migration sur Azure de [machines virtuelles Hyper-V](hyper-v-azure-architecture.md), de [machines virtuelles VMware](vmware-azure-architecture.md) et de [serveurs physiques](physical-azure-architecture.md) locaux. Pour tous les scénarios de réplication de service local sur Azure, les données sont envoyées à un compte de stockage Azure et stockées sur celui-ci. Lors d’une réplication, vous ne payez aucuns frais de machine virtuelle. Quand vous exécutez un basculement vers Azure, Site Recovery crée automatiquement des machines virtuelles Azure IaaS.

Site Recovery réplique des données vers un compte de stockage Azure ou des disques managés de réplica dans la région Azure cible via un point de terminaison public. Pour utiliser ExpressRoute afin d’effectuer une réplication de trafic Site Recovery, vous pouvez utiliser un [peering Microsoft](../expressroute/expressroute-circuit-peerings.md#microsoftpeering) ou un [peering public](../expressroute/about-public-peering.md) existant (déconseillé pour les créations). Le peering Microsoft est le domaine de routage recommandé pour la réplication. Notez que la réplication est prise en charge sur un Peering privé uniquement lorsque des [points de terminaison privés sont activés pour le coffre](hybrid-how-to-enable-replication-private-endpoints.md).

Vérifiez que les [exigences réseau](vmware-azure-configuration-server-requirements.md#network-requirements) pour le serveur de configuration sont également remplies. La connectivité à des URL spécifiques est requise par le serveur de configuration pour l’orchestration de la réplication Site Recovery. ExpressRoute ne peut pas être utilisé pour cette connectivité. 

Si vous utilisez un proxy en local et que vous souhaitez utiliser ExpressRoute pour le trafic de réplication, vous devez configurer la liste de contournement Proxy sur le serveur de configuration et les serveurs de traitement. Pour ce faire, procédez comme suit :

- Téléchargez l’outil PsExec [ici](/sysinternals/downloads/psexec) pour accéder au contexte de l’utilisateur système.
- Ouvrez Internet Explorer dans le contexte de l’utilisateur système en exécutant la ligne de commande suivante : psexec -s -i "%programfiles%\Internet Explorer\iexplore.exe"
- Ajouter des paramètres de proxy dans Internet Explorer
- Dans la liste de contournement, ajoutez l’URL de stockage Azure *. blob.core.windows.net

Seule la circulation du trafic de réplication peut alors s’écouler via ExpressRoute tandis que la communication peut passer par proxy.

Après que des machines virtuelles ou des serveurs ont basculé vers un réseau virtuel Azure, vous pouvez y accéder à l’aide d’un [peering privé](../expressroute/expressroute-circuit-peerings.md#privatepeering). 

Le scénario combiné est représenté dans le diagramme suivant : ![De local à Azure avec ExpressRoute](./media/concepts-expressroute-with-site-recovery/site-recovery-with-expressroute.png)

## <a name="azure-to-azure-replication-with-expressroute"></a>Réplication d’Azure sur Azure avec ExpressRoute

Azure Site Recovery permet une récupération d’urgence de [machines virtuelles Azure](azure-to-azure-architecture.md). Selon que vos machines virtuelles Azure utilisent ou non [Azure Managed Disks](../virtual-machines/managed-disks-overview.md), les données de réplication sont envoyées à un compte de stockage Azure ou à un disque managé de réplica dans la région Azure cible. Bien que les points de terminaison de réplication soient publics, le trafic de réplication pour la réplication de machines virtuelles Azure, par défaut, ne passe pas par Internet, quelle que soit la région Azure dans laquelle se trouve le réseau virtuel source. Vous pouvez remplacer l’itinéraire système par défaut d’Azure pour le préfixe d’adresse 0.0.0.0/0 par un [itinéraire personnalisé](../virtual-network/virtual-networks-udr-overview.md#custom-routes) et rediriger le trafic des machines virtuelles vers une appliance virtuelle réseau locale, mais cette configuration n’est pas recommandée pour la réplication Site Recovery. Si vous utilisez des itinéraires personnalisés, vous devez [créer un point de terminaison de service de réseau virtuel](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage) dans votre réseau virtuel pour « Stockage » afin que le trafic de réplication ne quitte pas la limite Azure.

Pour la récupération d’urgence de machine virtuelle Azure, par défaut, ExpressRoute n’est pas requis pour la réplication. Après le basculement des machines virtuelles vers la région Azure cible, vous pouvez y accéder via un [peering privé](../expressroute/expressroute-circuit-peerings.md#privatepeering). Notez que les tarifs de transfert de données s’appliquent quel que soit le mode de réplication des données entre les régions Azure.

Si vous utilisez déjà ExpressRoute pour vous connecter à partir de votre centre de données local aux machines virtuelles Azure dans la région source, vous pouvez planifier un rétablissement de la connectivité ExpressRoute dans la région cible du basculement. Vous pouvez utiliser le même circuit ExpressRoute pour vous connecter à la région cible via une nouvelle connexion de réseau virtuel, ou utiliser un circuit et une connexion ExpressRoute distincts pour la récupération d’urgence. Les différents scénarios possibles sont décrits [ici](azure-vm-disaster-recovery-with-expressroute.md#fail-over-azure-vms-when-using-expressroute).

Vous pouvez répliquer des machines virtuelles Azure vers n’importe quelle région Azure au sein du même cluster géographique, comme expliqué [ici](../site-recovery/azure-to-azure-support-matrix.md#region-support). Si la région Azure cible choisie ne figure pas dans la même région géopolitique que la source, il se peut que vous deviez activer ExpressRoute Premium. Pour plus d’informations, voir [Emplacements ExpressRoute](../expressroute/expressroute-locations.md) et [Tarification ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [circuits ExpressRoute](../expressroute/expressroute-circuit-peerings.md).
- En savoir plus sur les [domaines de routage ExpressRoute](../expressroute/expressroute-circuit-peerings.md#peeringcompare).
- En savoir plus sur les [emplacements ExpressRoute](../expressroute/expressroute-locations.md).
- En savoir plus sur la récupération d’urgence des [machines virtuelles Azure via ExpressRoute](azure-vm-disaster-recovery-with-expressroute.md).