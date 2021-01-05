---
title: Réseau virtuel Azure
description: Découvrez les concepts et fonctionnalités d’Azure Virtual Network, dont l’espace d’adressage, les sous-réseaux, les régions et les abonnements.
services: virtual-network
documentationcenter: na
author: KumudD
Customer intent: As someone with a basic network background that is new to Azure, I want to understand the capabilities of Azure Virtual Network, so that my Azure resources such as VMs, can securely communicate with each other, the internet, and my on-premises resources.
ms.service: virtual-network
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/03/2020
ms.author: kumud
ms.openlocfilehash: 11651b7a76ea0e61f78f3b2f305e39ed621df391
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109315"
---
# <a name="what-is-azure-virtual-network"></a>Présentation du réseau virtuel Azure

Le réseau virtuel Azure (VNet) est le bloc de construction fondamental pour votre réseau privé dans Azure. Le réseau virtuel permet à de nombreux types de ressources Azure, telles que les machines virtuelles (VM) Azure, de communiquer de manière sécurisée entre elles, avec Internet et avec les réseaux locaux. Un réseau virtuel est similaire à un réseau traditionnel que vous utiliseriez dans votre propre centre de données, mais avec les avantages supplémentaires de l’infrastructure Azure, tels que la mise à l’échelle, la disponibilité et l’isolation.

## <a name="why-use-an-azure-virtual-network"></a>Pourquoi utiliser un réseau virtuel Azure ?
Le réseau virtuel Azure permet aux ressources Azure de communiquer de manière sécurisée entre elles, avec Internet et sur des réseaux locaux. Les principaux scénarios qui vous permettent de constituer un réseau virtuel incluent la communication des ressources Azure avec Internet, la communication entre les ressources Azure, la communication avec les ressources locales, le filtrage du trafic réseau, le routage du trafic réseau et l’intégration aux services Azure.

### <a name="communicate-with-the-internet"></a>Communiquer avec Internet

Par défaut, toutes les ressources d’un réseau virtuel peuvent communiquer en sortie vers Internet. Vous pouvez effectuer des communications entrantes vers une ressource en lui assignant une adresse IP publique ou un équilibreur de charge publique. Vous pouvez également utiliser l’adresse IP publique ou l’équilibreur de charge public pour gérer vos connexions sortantes.  Pour en savoir plus sur les connexions sortantes dans Azure, consultez [Connexions sortantes](../load-balancer/load-balancer-outbound-connections.md)., [Adresses IP publiques](virtual-network-public-ip-address.md) et [Équilibreur de charge](../load-balancer/load-balancer-overview.md).

>[!NOTE]
>Lorsque vous utilisez un [équilibreur de charge standard](../load-balancer/load-balancer-standard-overview.md) interne, une connectivité sortante n’est pas disponible jusqu'à ce que vous définissiez le fonctionnement des [connexions sortantes](../load-balancer/load-balancer-outbound-connections.md) pour travailler avec une adresse IP publique au niveau de l’instance ou un équilibreur de charge public.

### <a name="communicate-between-azure-resources"></a>Communiquer entre les ressources Azure

Les ressources Azure communiquent en toute sécurité entre elles de l’une des manières suivantes :

- **Via un réseau virtuel** : vous pouvez déployer des machines virtuelles et plusieurs autres types de ressources Azure sur un réseau virtuel, comme des environnements Azure App Service, Azure Kubernetes Service et des groupes de machines virtuelles identiques Azure. Pour obtenir la liste complète des ressources Azure que vous pouvez déployer sur un réseau virtuel, consultez [Intégration des services de réseau virtuel](virtual-network-for-azure-services.md).
- **Via un point de terminaison de service de réseau virtuel** : étendez l’espace d’adressage privé de votre réseau virtuel et l’identité de votre réseau virtuel aux ressources du service Azure, tels que les comptes Stockage Azure et Azure SQL Database, par le biais d’une connexion directe. Les points de terminaison de service vous permettent de sécuriser vos ressources critiques du service Azure pour un réseau virtuel uniquement. Pour plus d’informations, consultez [Présentation des points de terminaison de service de réseau virtuel](virtual-network-service-endpoints-overview.md).
- **Via un peering de réseau virtuel** : Vous pouvez connecter des réseaux virtuels entre eux, ce qui permet aux ressources de ces réseaux virtuels de communiquer entre eux à l’aide d’un peering. Les réseaux virtuels que vous connectez peuvent être situés dans des régions Azure identiques ou différentes. Pour en savoir plus, consultez [Peering de réseaux virtuels](virtual-network-peering-overview.md).

### <a name="communicate-with-on-premises-resources"></a>Communiquer avec les ressources locales

Vous pouvez connecter vos ordinateurs et réseaux locaux à un réseau virtuel à l’aide de n’importe quelle combinaison des options suivantes :

- **Réseau privé virtuel (VPN) de point à site :** connexion établie entre un réseau virtuel et un ordinateur unique de votre réseau. Chaque ordinateur qui doit établir une connexion avec un réseau virtuel doit configurer ses connexions. Ce type de connexion est utile si vous n’êtes pas familiarisé avec Azure, ou pour les développeurs car elle nécessite peu voire pas de modifications de votre réseau existant. La communication entre votre ordinateur et un réseau virtuel passe par un tunnel chiffré via Internet. Pour plus d’informations, consultez [VPN de point à site](../vpn-gateway/point-to-site-about.md?toc=%2fazure%2fvirtual-network%2ftoc.json#).
- **VPN de site à site :** connexion établie entre votre appareil VPN local et une passerelle VPN Azure déployée dans un réseau virtuel. Ce type de connexion permet à n’importe quelle ressource locale de votre choix d’accéder à un réseau virtuel. La communication entre votre appareil VPN local et une passerelle VPN Azure passe par un tunnel chiffré via Internet. Pour plus d’informations, consultez [VPN de site à site](../vpn-gateway/design.md?toc=%2fazure%2fvirtual-network%2ftoc.json#s2smulti).
- **Azure ExpressRoute :** connexion établie entre votre réseau et Azure via un partenaire ExpressRoute. Cette connexion est privée. Toutefois, le trafic ne passe pas par Internet. Pour plus d’informations, consultez [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="filter-network-traffic"></a>Filtrer le trafic

Vous pouvez filtrer le trafic réseau entre les sous-réseaux à l’aide d’une des deux options suivantes :

- **Groupes de sécurité réseau :** les groupes de sécurité réseau et les groupes de sécurité d'application peuvent contenir plusieurs règles de sécurité entrantes et sortantes qui vous permettent de filtrer le trafic vers et en provenance des ressources par source et adresse IP de destination, port et protocole. Pour en savoir plus, consultez [Groupes de sécurité réseau](security-overview.md#network-security-groups) et [Groupes de sécurité d’application](security-overview.md#application-security-groups).
- **Appliances virtuelles réseau :** une appliance virtuelle réseau est une machine virtuelle exécutant une fonction réseau, telle qu’un pare-feu, l’optimisation du WAN ou une autre fonction réseau. Pour afficher la liste des appliances virtuelles réseau disponibles que vous pouvez déployer dans un réseau virtuel, consultez [Place de marché Microsoft Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances).

### <a name="route-network-traffic"></a>Router le trafic

Les itinéraires Azure se chargent de l’acheminement entre les sous-réseaux, les réseaux virtuels connectés, les réseaux locaux et Internet, par défaut. Vous pouvez implémenter une ou les deux options suivantes pour remplacer les itinéraires par défaut créés par Azure :

- **Tables de routage :** vous pouvez créer des tables de routage personnalisées avec des itinéraires qui contrôlent où le trafic est acheminé pour chaque sous-réseau. Découvrez-en plus sur les [tables de routage](virtual-networks-udr-overview.md#user-defined).
- **Itinéraires BGP :** si vous connectez votre réseau virtuel à votre réseau local via une connexion ExpressRoute ou la passerelle VPN Azure, vous pouvez propager vos itinéraires BGP locaux à vos réseaux virtuels. En savoir plus sur l’utilisation du protocole BGP avec [la passerelle VPN Azure](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [ExpressRoute](../expressroute/expressroute-routing.md?toc=%2fazure%2fvirtual-network%2ftoc.json#dynamic-route-exchange).

### <a name="virtual-network-integration-for-azure-services"></a>Intégration d’un réseau virtuel pour les services Azure

L’intégration des services Azure à un réseau virtuel Azure permet un accès privé au service à partir de machines virtuelles ou de ressources de calcul dans le réseau virtuel.
Vous pouvez intégrer des services Azure dans votre réseau virtuel, grâce aux options suivantes :
- Déploiement d’[instances dédiées du service](virtual-network-for-azure-services.md) dans un réseau virtuel. Les services sont alors accessibles de manière privée dans le réseau virtuel, et à partir des réseaux locaux.
- Utilisation de [Liaison privée](../private-link/private-link-overview.md) pour accéder en privé à une instance spécifique du service à partir de votre réseau virtuel et de réseaux locaux.
- Vous pouvez également accéder au service à l’aide de points de terminaison publics en étendant un réseau virtuel au service par le biais de [points de terminaison de service](virtual-network-service-endpoints-overview.md). Les points de terminaison de service permettent de sécuriser les ressources de service au sein du réseau virtuel.
 

## <a name="azure-vnet-limits"></a>Limites des réseaux virtuels Azure

Le nombre de ressources Azure que vous pouvez déployer est limité. La plupart des limites de mise en réseau Azure sont des valeurs maximales. Toutefois, vous pouvez [augmenter certaines limites de mise en réseau](../azure-portal/supportability/networking-quota-requests.md) comme spécifié dans la page consacrée aux [limites de réseau virtuel](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits). 

## <a name="pricing"></a>Tarifs

L’utilisation de réseau virtuel Azure est totalement gratuite. Des frais standards s’appliquent aux ressources telles que les machines virtuelles et d’autres produits. Pour plus d’informations, voir la [tarification des réseaux virtuels](https://azure.microsoft.com/pricing/details/virtual-network/) et la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) Azure.

## <a name="next-steps"></a>Étapes suivantes
 - Découvrez les [concepts et bonnes pratiques du réseau virtuel Azure](concepts-and-best-practices.md).
 - Pour commencer à utiliser un réseau virtuel, créez-en un, déployez-y plusieurs machines virtuelles et communiquez entre les machines virtuelles. Pour en savoir plus, consultez l’article de démarrage rapide [Créer un réseau virtuel](quick-create-portal.md).
