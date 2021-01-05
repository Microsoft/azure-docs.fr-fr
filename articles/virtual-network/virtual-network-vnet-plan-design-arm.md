---
title: Planifier des réseaux virtuels Azure | Microsoft Docs
description: Découvrez comment planifier des réseaux virtuels selon vos besoins en isolation, connectivité et emplacements.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/08/2020
ms.author: kumud
ms.openlocfilehash: 29ea65e94e97b69e24c6935328cc01c2295adc5a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518634"
---
# <a name="plan-virtual-networks"></a>Planifier des réseaux virtuels

La création d’un réseau virtuel à titre d’essai est assez facile, mais vous risquez de déployer plusieurs réseaux virtuels au fil du temps pour prendre en charge les besoins en production de votre organisation. La planification vous permet de déployer des réseaux virtuels et de vous connecter aux ressources dont vous avez besoin plus efficacement. Les informations contenues dans cet article sont particulièrement utiles si vous êtes déjà familiarisé avec les réseaux virtuels et que vous disposez d’expérience dans leur utilisation. Si vous n’êtes pas familiarisé avec les réseaux virtuels, il vous est recommandé de lire [Virtual network overview](virtual-networks-overview.md) (Vue d’ensemble du réseau virtuel).

## <a name="naming"></a>Dénomination

Toutes les ressources Azure ont un nom. Le nom doit être unique au sein d’une étendue, qui peut varier pour chaque type de ressource. Par exemple, le nom d’un réseau virtuel doit être unique au sein d’un [groupe de ressources](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#resource-group), mais peut être dupliqué dans un [abonnement](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) ou une [région](https://azure.microsoft.com/regions/#services) Azure. La définition d’une convention d’affectation de noms que vous pouvez utiliser régulièrement lorsque vous nommez des ressources est utile lors de la gestion de plusieurs ressources réseau au fil du temps. Pour des suggestions, consultez [Conventions d’affectation de noms](../azure-resource-manager/management/resource-name-rules.md#microsoftnetwork).

## <a name="regions"></a>Régions

Toutes les ressources Azure sont créées dans une région Azure et un abonnement. Une ressource peut être connectée uniquement dans un réseau virtuel qui existe dans les mêmes région et abonnement que la ressource. Toutefois, vous pouvez connecter des réseaux virtuels qui existent dans différents abonnements et différentes régions. Pour plus d’informations, consultez [Connectivité](#connectivity). Lorsque vous choisissez les régions dans lesquelles déployer des ressources, prenez en compte l’emplacement physique des consommateurs des ressources :

- Les consommateurs de ressources souhaitent généralement la latence du réseau la plus faible possibles sur leurs ressources. Pour déterminer les latences relatives entre un emplacement spécifié et des régions Azure, consultez [View relative latencies](../network-watcher/view-relative-latencies.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Afficher les latences relatives).
- Avez-vous des exigences en matière de résidence, souveraineté, conformité ou résilience des données ? Dans c’est le cas, il est essentiel de choisir la région qui correspond à la configuration requise. Pour plus d’informations, consultez [Azure geographies](https://azure.microsoft.com/global-infrastructure/geographies/) (Zones géographiques Azure).
- Avez-vous besoin de résilience entre Zones de disponibilité Azure d’une même région Azure pour les ressources que vous déployez ? Vous pouvez déployer des ressources, comme des machines virtuelles, dans différentes zones de disponibilité au sein du même réseau virtuel. Toutefois, toutes les régions Azure ne prennent pas en charge les zones de disponibilité. Pour en savoir plus sur les zones de la disponibilité et les régions qui les prennent en charge, consultez [Zones de disponibilité](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="subscriptions"></a>Abonnements

Vous pouvez déployer autant de réseaux virtuels que nécessaire dans chaque abonnement, jusqu’à la [limite](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Par exemple, certaines organisations disposent d’abonnements distincts pour différents services. Pour plus d’informations et des remarques relatives aux abonnements, consultez [Subscription governance](/azure/cloud-adoption-framework/reference/migration-with-enterprise-scaffold#define-your-hierarchy) (Gouvernance de l’abonnement).

## <a name="segmentation"></a>Segmentation

Vous pouvez créer plusieurs réseaux virtuels par abonnement et par région. Vous pouvez créer plusieurs sous-réseaux au sein de chaque réseau virtuel. Les remarques qui suivent vous aident à déterminer le nombre de réseaux virtuels et sous-réseaux dont vous avez besoin :

### <a name="virtual-networks"></a>Réseaux virtuels

Un réseau virtuel est une partie virtuelle isolée du réseau public Azure. Chaque réseau virtuel est dédié à votre abonnement. Éléments à prendre en compte lorsque vous choisissez de créer un réseau virtuel ou plusieurs réseaux virtuels dans un abonnement :

- Avez-vous des exigences de sécurité d’organisation pour isoler le trafic dans des réseaux virtuels distincts ? Vous pouvez choisir de connecter des réseaux virtuels ou non. Si vous connectez des réseaux virtuels, vous pouvez implémenter une appliance virtuelle réseau, comme un pare-feu, pour contrôler le flux de trafic entre les réseaux virtuels. Pour plus d’informations, consultez [Sécurité](#security) et [Connectivité](#connectivity).
- Avez-vous des exigences d’organisation pour isoler les réseaux virtuels dans des [abonnements](#subscriptions) ou [régions](#regions) distincts ?
- Une [interface réseau](virtual-network-network-interface.md) permet à un machine virtuelle de communiquer avec d’autres ressources. Une ou plusieurs adresses IP privées sont affectées à chaque interface réseau. De combien d’interfaces réseau et d’[adresses IP privées](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses) avez-vous besoin dans un réseau virtuel ? Des [limites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits) s’appliquent quant au nombre d’interfaces réseau et d’adresses IP privées que vous pouvez avoir dans un réseau virtuel.
- Voulez-vous connecter le réseau virtuel à un autre réseau virtuel ou au réseau local ? Vous pouvez choisir de connecter certains réseaux virtuels entre eux ou des réseaux locaux, mais pas d’autres. Pour plus d’informations, consultez [Connectivité](#connectivity). Chaque réseau virtuel que vous connectez à un autre réseau virtuel, ou au réseau local, doit avoir un espace d’adresses unique. Chaque réseau virtuel comprend une ou plusieurs plages d’adresses publiques ou privées affectées à son espace d’adresses. Une plage d’adresses est spécifiée dans un format de routage de domaine Interne sans classe (CIDR), par exemple, 10.0.0.0/16. En savoir plus sur [plages d’adresses](manage-virtual-network.md#add-or-remove-an-address-range) pour les réseaux virtuels.
- Avez-vous des exigences d’administration d’organisation pour les ressources dans différents réseaux virtuels ? Si c’est le cas, vous pouvez séparer des ressources dans un réseau virtuel distinct afin de simplifier l’[affectation d’autorisations](#permissions) à des personnes de votre organisation ou pour assigner différentes stratégies à différents réseaux virtuels.
- Lorsque vous déployez des ressources de service Azure dans un réseau virtuel, elles créent leur propre réseau virtuel. Pour déterminer si un service Azure crée son propre réseau virtuel, consultez les informations de chaque [service Azure pouvant être déployé dans un réseau virtuel](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network).

### <a name="subnets"></a>Sous-réseaux

Un réseau virtuel peut être segmenté en un ou plusieurs sous-réseaux, jusqu’aux [limites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Éléments à prendre en compte lorsque vous choisissez de créer un sous-réseau ou plusieurs réseaux virtuels dans un abonnement :

- Chaque sous-réseau doit avoir une plage d’adresses unique, spécifiée au format CIDR, dans l’espace d’adresses du réseau virtuel. La plage d’adresses ne peut pas chevaucher d’autres sous-réseaux au sein du réseau virtuel.
- Si vous envisagez de déployer des ressources de service Azure dans un réseau virtuel, elles peuvent avoir besoin de créer leur propre sous-réseau. De l’espace non alloué doit être suffisant pour le faire. Pour déterminer si un service Azure crée son propre sous-réseau, consultez les informations de chaque [service Azure pouvant être déployé dans un réseau virtuel](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network). Par exemple, si vous connectez un réseau virtuel à un réseau local à l’aide d’une passerelle VPN Azure, le réseau virtuel doit avoir un sous-réseau dédié pour la passerelle. En savoir plus sur les [sous-réseaux de passerelle](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md?toc=%2fazure%2fvirtual-network%2ftoc.json#gwsub).
- Par défaut, Azure achemine le trafic réseau entre tous les sous-réseaux dans un réseau virtuel. Par exemple, vous pouvez remplacer le routage par défaut d’Azure pour empêcher le routage Azure entre des sous-réseaux ou pour acheminer le trafic entre des sous-réseaux via une appliance virtuelle réseau. Si vous avez besoin que le trafic entre des ressources d’un même réseau virtuel circulent via une appliance virtuelle réseau (NVA), déployez les ressources sur des sous-réseaux différents. En savoir plus la [sécurité](#security).
- Vous pouvez limiter l’accès aux ressources Azure, un compte de stockage Azure ou une base de données Azure SQL par exemple, à des sous-réseaux spécifiques avec un point de terminaison de service de réseau virtuel. Vous pouvez également refuser l’accès aux ressources à partir d’Internet. Vous pouvez créer plusieurs sous-réseaux et activer un point de terminaison de service pour certains sous-réseaux, mais pas pour d’autres. En savoir plus sur les [points de terminaison de service](virtual-network-service-endpoints-overview.md), et les ressources Azure pour lesquelles vous pouvez les activer.
- Vous pouvez associer zéro ou un groupe de sécurité réseau à chaque sous-réseau dans un réseau virtuel. Vous pouvez associer le même groupe de sécurité réseau, ou un autre, à chaque sous-réseau. Chaque groupe de sécurité réseau contient des règles, qui autorisent ou refusent le trafic vers et depuis des sources et des destinations. En savoir plus sur les [groupes de sécurité réseau](#traffic-filtering).

## <a name="security"></a>Sécurité

Vous pouvez filtrer le trafic réseau vers et depuis des ressources dans un réseau virtuel à l’aide de groupes de sécurité réseau et d’appliances virtuelles réseau. Vous pouvez contrôler la façon selon laquelle Azure achemine le trafic à partir des sous-réseaux. Vous pouvez également limiter qui dans votre organisation peut utiliser des ressources dans des réseaux virtuels.

### <a name="traffic-filtering"></a>Filtrage du trafic

- Vous pouvez filtrer le trafic réseau entre des ressources dans un réseau virtuel à l’aide d’un groupe de sécurité réseau, d’une NVA qui filtre le trafic réseau, ou des deux. Pour déployer une NVA, comme un pare-feu, pour filtrer le trafic réseau, consultez la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?subcategories=appliances&page=1). Lorsque vous utilisez une NVA, vous créez également des itinéraires personnalisés pour acheminer le trafic entre les sous-réseaux et la NVA. En savoir plus sur le [routage du trafic](#traffic-routing).
- Un groupe de sécurité réseau contient plusieurs règles de sécurité par défaut qui autorisent ou refusent le trafic vers ou depuis des ressources. Un groupe de sécurité réseau peut être associé à une interface réseau, au sous-réseau où se trouve l’interface réseau, ou aux deux. Pour simplifier la gestion des règles de sécurité, il vous est recommandé d’associer, chaque fois que cela est possible, un groupe de sécurité réseau à des sous-réseaux, plutôt qu’à des interfaces réseau du sous-réseau.
- Si différentes machines virtuelles au sein d’un sous-réseau ont besoin de règles de sécurité différentes, vous pouvez associer l’interface réseau de la machine virtuelle à un ou plusieurs groupes de sécurité d’application. Une règle de sécurité peut spécifier un groupe de sécurité d’application dans sa source, sa destination, ou les deux. Cette règle ne s’applique alors qu’aux interfaces réseau qui sont membres du groupe de sécurité d’application. En savoir plus sur les [groupes de sécurité réseau](security-overview.md) et les [groupes de sécurité d’application](security-overview.md#application-security-groups).
- Azure crée plusieurs règles de sécurité par défaut dans chaque groupe de sécurité réseau. Une règle par défaut autorise l’ensemble du trafic entre toutes les ressources dans un réseau virtuel. Pour remplacer ce comportement, utilisez des groupes de sécurité réseau, un routage personnaliser pour acheminer le trafic vers une NVA, ou les deux. Il vous est recommandé de vous familiariser avec toutes les [règles de sécurité par défaut](security-overview.md#default-security-rules) d’Azure et de comprendre comment les règles de groupe de sécurité réseau s’appliquent à une ressource.

Vous pouvez consulter des exemples de conception pour l’implémentation d’un réseau de périmètre (ou DMZ) entre Azure et Internet, à l’aide d’une [appliance virtuelle réseau](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz?toc=%2Fazure%2Fvirtual-network%2Ftoc.json).

### <a name="traffic-routing"></a>Routage du trafic

Azure crée plusieurs itinéraires par défaut pour le trafic sortant à partir d’un sous-réseau. Vous pouvez remplacer le routage par défaut d’Azure en créant une table de routage et en l’associant à un sous-réseau. Les raisons courantes du remplacement du routage par défaut d’Azure sont les suivantes :
- Car vous souhaitez que le trafic entre des sous-réseaux circule via une NVA. Pour en savoir plus sur la [configuration de tables de routage et forcer le trafic via une NVA](tutorial-create-route-table-portal.md).
- Car vous souhaitez forcer l’ensemble du trafic Internet via une NVA, ou en local, via une passerelle VPN Azure. Le fait de forcer le trafic Internet en local pour inspection et la journalisation sont souvent appelés le tunneling forcé. En savoir plus sur la configuration du [tunneling forcé](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2Fazure%2Fvirtual-network%2Ftoc.json).

Si vous avez besoin d’implémenter un routage personnalisé, il vous est recommandé de vous familiariser avec le [routage dans Azure](virtual-networks-udr-overview.md).

## <a name="connectivity"></a>Connectivité

Vous pouvez connecter un réseau virtuel à d’autres réseaux virtuels à l’aide du peering de réseau virtuel, ou à votre réseau local, à l’aide d’une passerelle VPN Azure.

### <a name="peering"></a>Peering

Lorsque vous utilisez le [peering de réseau virtuel](virtual-network-peering-overview.md), les réseaux virtuels peuvent se trouver dans la même région, ou dans différentes régions Azure prises en charge. Les réseaux virtuels peuvent se trouver dans le même abonnement ou dans des abonnements Azure différents (voire dans des abonnements appartenant à des locataires Azure Active Directory différents). Avant de créer un peering, il est recommandé de vous familiariser avec toutes les [exigences et contraintes du peering](virtual-network-manage-peering.md#requirements-and-constraints). La bande passante entre des ressources figurant dans des réseaux virtuels homologués dans la même région est la même que si ces ressources se trouvaient dans le même réseau virtuel.

### <a name="vpn-gateway"></a>passerelle VPN

Vous pouvez utiliser une [passerelle VPN](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure pour connecter un réseau virtuel à votre réseau local à l’aide un [VPN site à site](../vpn-gateway/vpn-gateway-tutorial-vpnconnection-powershell.md?toc=%2fazure%2fvirtual-network%2ftoc.json), ou à l’aide d’une connexion dédiée à Azure [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Vous pouvez combiner le peering avec une passerelle VPN pour créer des [réseaux Hub and Spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json), dans lesquels les réseaux virtuels Spoke se connectent à un réseau virtuel Hub, et où le Hub se connecte à un réseau local, par exemple.

### <a name="name-resolution"></a>Résolution de noms

Les ressources dans un réseau virtuel ne peuvent pas résoudre les noms de ressources dans un réseau virtuel homologué à l’aide du [DNS intégré](virtual-networks-name-resolution-for-vms-and-role-instances.md) d’Azure. Pour résoudre les noms dans un réseau virtuel homologué, [déployez votre propre serveur DNS](virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) ou utilisez des [domaines privés](../dns/private-dns-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure DNS. La résolution des noms entre des ressources dans un réseau virtuel et des réseaux locaux requiert également que vous déployiez votre propre serveur DNS.

## <a name="permissions"></a>Autorisations

Azure utilise le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour gérer l’accès aux ressources. Des autorisations sont affectées à une [étendue](../role-based-access-control/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#scope) dans la hiérarchie suivante : groupe d’administration, abonnement, groupe de ressources et ressource individuelle. Pour en savoir plus sur la hiérarchie, consultez [Organize your resources](../azure-resource-manager/management-groups-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Organiser vos ressources). Pour utiliser des réseaux virtuels Azure et toutes leurs fonctionnalités associées, comme le peering, les groupes de sécurité réseau, les points de terminaison de service et les tables de routage, vous pouvez assigner à des membres de votre organisation le rôle [Propriétaire](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#owner), [Contributeur](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#contributor) ou [Contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) intégré, puis assigner le rôle à l’étendue appropriée. Si vous souhaitez assigner des autorisations spécifiques pour un sous-ensemble de fonctionnalités de réseau virtuel, créez un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et assignez les autorisations spécifiques nécessaires pour les [réseaux virtuels](manage-virtual-network.md#permissions), [sous-réseaux et points de terminaison de service](virtual-network-manage-subnet.md#permissions), [interfaces réseau](virtual-network-network-interface.md#permissions), [peering](virtual-network-manage-peering.md#permissions), [groupes de sécurité réseau et d’application](manage-network-security-group.md#permissions) ou les [tables de routage](manage-route-table.md#permissions) au rôle.

## <a name="policy"></a>Policy

Azure Policy vous permet de créer, d’assigner et de gérer des définitions de stratégie. Les définitions de stratégie appliquent différentes règles sur vos ressources, qui restent donc conformes aux normes et aux contrats de niveau de service de l’organisation. Azure Policy exécute une évaluation de vos ressources, en analysant les ressources qui ne sont pas conformes avec les définitions de stratégie dont vous disposez. Par exemple, vous pouvez définir et appliquer une stratégie qui autorise la création de réseaux virtuels dans seulement une région ou un groupe de ressources spécifique. Une autre stratégie peut exiger qu’un groupe de sécurité réseau soit associé à chaque sous-réseau. Les stratégies sont alors évaluées lors de la création et de la mise à jour des ressources.

Les stratégies sont appliquées à la hiérarchie suivante : groupe d’administration, abonnement et groupe de ressources. Apprenez-en plus sur [Azure Policy](../governance/policy/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou déployez des [définitions Azure Policy](policy-samples.md) de réseau virtuel.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les tâches, paramètres et options d’un [réseau virtuel](manage-virtual-network.md), d’un [point de terminaison de service et de sous-réseau](virtual-network-manage-subnet.md), d’une [interface réseau](virtual-network-network-interface.md), d’un [peering](virtual-network-manage-peering.md), d’un [groupe de sécurité réseau et d’application](manage-network-security-group.md) ou d’une [table de routage](manage-route-table.md).
