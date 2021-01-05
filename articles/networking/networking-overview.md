---
title: Vue d’ensemble des services de mise en réseau Azure
description: 'Découvrez les services de mise en réseau dans Azure : services de connectivité, services de protection des applications, services de distribution des applications et de surveillance réseau.'
services: networking
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/28/2020
ms.author: kumud
ms.openlocfilehash: f49a340a004a4aef37bcae9e3ae1c2b02ae030b9
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913079"
---
# <a name="azure-networking-services-overview"></a>Vue d’ensemble des services de mise en réseau Azure

Les services de mise en réseau dans Azure fournissent un éventail de fonctionnalités de mise en réseau qui peuvent être utilisées ensemble ou séparément. Cliquez sur une des fonctionnalités principales suivantes pour en savoir plus à leur sujet :
- [**Services de connectivité**](#connect) : Connectez les ressources Azure et les ressources locales à l’aide d’un de ces services de mise en réseau dans Azure – réseau virtuel (VNet), Virtual WAN, ExpressRoute, passerelle VPN, passerelle NAT de réseau virtuel, Azure DNS, Peering Service et Azure Bastion – ou d’une combinaison de ces derniers.
- [**Services de protection des applications**](#protect) : Protégez vos applications à l’aide d’un ou plusieurs de ces services de mise en réseau dans Azure – Private Link, protection DDoS, pare-feu, groupes de sécurité réseau, pare-feu d’applications Web et points de terminaison de réseau virtuel.
- [**Services de distribution d’applications**](#deliver) : Fournissez des applications dans le réseau Azure à l’aide d’un ou plusieurs de ces services de mise en réseau dans Azure – Content Delivery Network (CDN), Azure Front Door Service, Traffic Manager, Application Gateway, Internet Analyzer et Load Balancer.
- [**Surveillance du réseau**](#monitor) : Surveillez vos ressources réseau à l’aide d’un ou plusieurs de ces services de mise en réseau dans Azure – Network Watcher, ExpressRoute Monitor, Azure Monitor ou point d’accès terminal de réseau virtuel (TAP).

## <a name="connectivity-services"></a><a name="connect"></a>Services de connectivité
 
Cette section décrit les services qui assurent la connectivité entre ressources Azure, d’un réseau local à des ressources Azure et d’une branche à l’autre dans Azure : réseau virtuel, ExpressRoute, passerelle VPN, Virtual WAN, NAT Gateway de réseau virtuel, Azure DNS, Azure Peering Service et Azure Bastion.


### <a name="virtual-network"></a><a name="vnet"></a>Réseau virtuel

Le réseau virtuel Azure (VNet) est le bloc de construction fondamental pour votre réseau privé dans Azure. Vous pouvez utiliser des réseaux virtuels pour :
- **Communiquer entre les ressources Azure**  : vous pouvez déployer des machines virtuelles et plusieurs autres types de ressources Azure sur un réseau virtuel, comme des environnements Azure App Service, Azure Kubernetes Service et des groupes de machines virtuelles identiques Azure. Pour obtenir la liste complète des ressources Azure que vous pouvez déployer sur un réseau virtuel, consultez [Intégration des services de réseau virtuel](../virtual-network/virtual-network-for-azure-services.md).
- **Communiquer mutuellement**  : Vous pouvez connecter des réseaux virtuels entre eux, ce qui permet aux ressources de ces réseaux virtuels de communiquer entre eux à l’aide d’un peering. Les réseaux virtuels que vous connectez peuvent être situés dans des régions Azure identiques ou différentes. Pour en savoir plus, consultez [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).
- **Communiquer vers Internet**  : Par défaut, toutes les ressources d’un réseau virtuel peuvent communiquer en sortie vers Internet. Vous pouvez effectuer des communications entrantes vers une ressource en lui assignant une adresse IP publique ou un équilibreur de charge publique. Vous pouvez également utiliser des [adresses IP publique](../virtual-network/virtual-network-public-ip-address.md) ou [Load Balancer](../load-balancer/load-balancer-overview.md) pour gérer vos connexions sortantes.
- **Communiquer avec des réseaux locaux**  : Vous pouvez connecter vos ordinateurs et réseaux locaux à un réseau virtuel à l’aide la [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou d’ [ExpressRoute](../expressroute/expressroute-introduction.md).

Pour plus d’informations, consultez [Présentation du réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).

### <a name="expressroute"></a><a name="expressroute"></a>ExpressRoute
ExpressRoute vous permet d’étendre vos réseaux locaux dans le cloud Microsoft via une connexion privée assurée par un fournisseur de connectivité. Cette connexion est privée. Toutefois, le trafic ne passe pas par Internet. Avec ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, par exemple Microsoft Azure, Microsoft 365 et Dynamics 365.  Pour plus d’informations, consultez [Présentation d’ExpressRoute](../expressroute/expressroute-introduction.md).

:::image type="content" source="./media/networking-overview/expressroute-connection-overview.png" alt-text="Azure ExpressRoute" border="false":::

### <a name="vpn-gateway"></a><a name="vpngateway"></a>Passerelle VPN
La passerelle VPN vous aide à créer des connexions chiffrées intersites vers votre réseau virtuel à partir d’emplacements locaux, ou à créer des connexions chiffrées entre des réseaux virtuels. Différentes configurations sont disponibles pour les connexions de passerelle VPN, notamment de site à site, de point à site ou de réseau virtuel à réseau virtuel.
Le diagramme suivant illustre plusieurs connexions VPN de site à site au même réseau virtuel.

:::image type="content" source="./media/networking-overview/vpngateway-multisite-connection-diagram.png" alt-text="Connexions de passerelle VPN Azure de site à site":::

Pour plus d’informations sur les différents types de connexions VPN, consultez [Passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="virtual-wan"></a><a name="virtualwan"></a>Virtual WAN
Azure Virtual WAN est un service réseau qui offre une connectivité de branche optimisée et automatisée via Azure. Les régions Azure servent de hubs auxquels vous pouvez connecter vos branches. Vous pouvez exploiter la dorsale principale d’Azure pour également connecter des branches et profiter d’une connectivité de branche à réseau virtuel. Azure Virtual WAN regroupe plusieurs services de connectivité cloud Azure comme le VPN site à site, ExpressRoute et le VPN utilisateur point à site dans une même interface opérationnelle. La connectivité aux réseaux virtuels Azure est établie à l’aide de connexions de réseau virtuel. Pour plus d’informations, consultez [Présentation du réseau WAN virtuel](../virtual-wan/virtual-wan-about.md).

:::image type="content" source="./media/networking-overview/virtualwan1.png" alt-text="Diagramme WAN virtuel":::

### <a name="azure-dns"></a><a name="dns"></a>Azure DNS
Azure DNS est un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS à l’aide des mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure. Pour plus d’informations, consultez [Présentation d’Azure DNS](../dns/dns-overview.md).

### <a name="azure-bastion"></a><a name="bastion"></a>Azure Bastion
Le service Azure Bastion est un nouveau service PaaS complètement managé par la plateforme que vous provisionnez au sein de votre réseau virtuel. Il fournit une connectivité RDP/SSH sécurisée et fluide à vos machines virtuelles, directement dans le portail Azure via TLS. Lorsque vous vous connectez via Azure Bastion, vos machines virtuelles n’ont pas besoin d’une adresse IP publique. Pour plus d’informations, consultez [Présentation d’Azure Bastion](../bastion/bastion-overview.md).

:::image type="content" source="./media/networking-overview/architecture.png" alt-text="Architecture d’Azure Bastion":::

### <a name="virtual-network-nat-gateway"></a><a name="nat"></a>Passerelle NAT de réseau virtuel
Le service NAT (traduction d’adresses réseau) de Réseau virtuel simplifie la connectivité Internet sortante uniquement pour les réseaux virtuels. Quand il est configuré sur un sous-réseau, toute la connectivité sortante utilise vos adresses IP publiques statiques spécifiées. Une connectivité sortante est possible sans équilibreur de charge ni adresses IP publiques directement attachées aux machines virtuelles. Pour plus d’informations, consultez [Qu’est-ce qu’une passerelle NAT de réseau virtuel ?](../virtual-network/nat-overview.md).

:::image type="content" source="./media/networking-overview/flow-map.png" alt-text="Passerelle NAT de réseau virtuel":::

### <a name="azure-peering-service"></a><a name="azurepeeringservice"></a> Azure Peering Service
Azure Peering Service améliore la connectivité du client aux services cloud de Microsoft tels que Microsoft 365, Dynamics 365, les services SaaS, Azure ou les services Microsoft accessibles via le réseau Internet public. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Peering Service ?](../peering-service/about.md).

### <a name="azure-edge-zones"></a><a name="edge-zones"></a>Azure Edge Zones

Azure Edge Zone est une famille d’offres de Microsoft Azure qui permet de traiter les données à proximité de l’utilisateur. Vous pouvez déployer des machines virtuelles, des conteneurs et certains services Azure dans Edge Zone pour répondre aux exigences de faible latence et de débit élevé des applications. Pour plus d’informations, consultez [Présentation d’Azure Edge Zones](edge-zones-overview.md).

### <a name="azure-orbital"></a><a name="orbital"></a>Azure Orbital

Azure Orbital est une station terrienne complètement managée dans le Cloud en tant que service qui vous permet de communiquer avec votre engin spatial ou vos constellations de satellites, de transmettre des données en liaison descendante et ascendante, de traiter vos données dans le Cloud, de chaîner les services avec les services Azure dans des scénarios uniques et de générer des produits pour vos clients. Ce système est construit sur l’infrastructure globale Azure et le réseau mondial de fibres optiques à faible latence. Pour plus d’informations, consultez [Présentation d’Azure Orbital](azure-orbital-overview.md).

:::image type="content" source="./media/azure-orbital-overview/orbital-communications-use-flow.png" alt-text="Diagramme de communications Azure orbital":::

## <a name="application-protection-services"></a><a name="protect"></a>Services de protection des applications

Cette section décrit les services réseau Azure qui, séparément ou conjointement, permettent de protéger les ressources réseau et les applications : protection DDoS, Private Link, Pare-feu, Web Application Firewall, groupes de sécurité réseau et points de terminaison de réseaux virtuels.

### <a name="ddos-protection"></a><a name="ddosprotection"></a>DDoS Protection 
[Azure DDoS Protection](../virtual-network/manage-ddos-protection.md) fournit des mesures contre les menaces DDoS les plus sophistiquées. Le service fournit des fonctionnalités améliorées d’atténuation des attaques DDoS pour votre application et les ressources déployées dans vos réseaux virtuels. Par ailleurs, les clients qui utilisent Azure DDoS Protection ont accès à la prise en charge de la réponse rapide DDoS pour faire appel à des experts DDoS pendant une attaque active.

:::image type="content" source="./media/networking-overview/ddos-protection.png" alt-text="Protection DDOS":::

### <a name="azure-private-link"></a><a name="privatelink"></a>Azure Private Link
[Azure Private Link](../private-link/private-link-overview.md) vous permet d’accéder aux services Azure PaaS (par exemple Stockage Azure et SQL Database) ainsi qu’aux services de partenaires ou de clients hébergés par Azure sur un point de terminaison privé dans votre réseau virtuel.
Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft. L’exposition de votre service à l’Internet public n’est plus nécessaire. Vous pouvez créer votre propre service de liaison privée dans votre réseau virtuel et le distribuer à vos clients.

:::image type="content" source="./media/networking-overview/private-endpoint.png" alt-text="Vue d’ensemble du point de terminaison privé":::

### <a name="azure-firewall"></a><a name="firewall"></a>Pare-feu Azure
Pare-feu Azure est un service de sécurité réseau informatique géré qui protège vos ressources Réseau virtuel Azure. Avec le pare-feu Azure, vous pouvez créer, appliquer et consigner des stratégies de connectivité réseau et d’application de façon centralisée entre les abonnements et les réseaux virtuels. Pare-feu Azure utilise une adresse IP publique statique pour vos ressources de réseau virtuel, ce qui permet aux pare-feu situés à l’extérieur d’identifier le trafic provenant de votre réseau virtuel. 

Pour plus d’informations sur le pare-feu Azure, consultez la [documentation du pare-feu Azure](../firewall/overview.md).

:::image type="content" source="./media/networking-overview/firewall-threat.png" alt-text="Présentation du pare-feu":::

### <a name="web-application-firewall"></a><a name="waf"></a>Pare-feu d’applications web
Le [pare-feu d’applications web Azure](../web-application-firewall/overview.md) (WAF) offre une protection de vos applications web contre les codes malveillants exploitant une faille de sécurité et les vulnérabilités telles que les injections de code SQL et le script de site à site. Le pare-feu d'applications web Azure fournit une protection prête à l’emploi contre les 10 principales vulnérabilités établies par OWASP par le biais de règles managées. Les clients peuvent également configurer des règles personnalisées qu’ils gèrent pour offrir une protection supplémentaire basée sur la plage d’adresses IP sources et sur des attributs de requêtes tels que les en-têtes, les cookies, les champs de données de formulaires ou les paramètres de chaînes de requête.

Les clients peuvent choisir de déployer le [pare-feu d'applications web Azure avec Application Gateway](../application-gateway/waf-overview.md), qui offre une protection régionale aux entités dans un espace d’adressage public et privé. Ils peuvent également choisir de déployer le [pare-feu d'applications web Azure avec Front Door](../frontdoor/waf-overview.md), qui offre une protection à la périphérie du réseau vers les points de terminaison publics.

:::image type="content" source="./media/networking-overview/waf-overview.png" alt-text="Pare-feu d’applications web":::

### <a name="network-security-groups"></a><a name="nsg"></a>Groupes de sécurité réseau
Vous pouvez filtrer le trafic réseau depuis et vers les ressources Azure dans un réseau virtuel Azure avec un groupe de sécurité réseau. Pour plus d’informations, consultez [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).

### <a name="service-endpoints"></a><a name="serviceendpoints"></a>Points de terminaison de service
Les points de terminaison de service de réseau virtuel étendent votre espace d’adressage privé de réseau virtuel et l’identité de votre réseau virtuel aux services Azure, via une connexion directe. Les points de terminaison permettent de sécuriser vos ressources critiques du service Azure pour vos réseaux virtuels uniquement. Le trafic à partir de votre réseau virtuel vers le service Azure reste toujours sur le réseau principal de Microsoft Azure. Pour plus d’informations, consultez [Points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md).

:::image type="content" source="./media/networking-overview/vnet-service-endpoints-overview.png" alt-text="Points de terminaison de service de réseau virtuel":::

## <a name="application-delivery-services"></a><a name="deliver"></a>Services de livraison d’applications

Cette section décrit les services de mise en réseau dans Azure qui permettent de fournir des applications : Content Delivery Network, Azure Front Door Service, Traffic Manager, Load Balancer et Application Gateway.

### <a name="content-delivery-network"></a><a name="cdn"></a>Content Delivery Network
Le réseau de diffusion de contenu (CDN) Azure offre aux développeurs une solution globale pour la distribution rapide de contenu haut débit aux utilisateurs en mettant en cache leur contenu sur des nœuds physiques disposés stratégiquement dans le monde entier. Pour plus d’informations sur le CDN Azure, voir la page [Azure Content Delivery Network](../cdn/cdn-overview.md)

:::image type="content" source="./media/networking-overview/cdn-overview.png" alt-text="Azure CDN":::

### <a name="azure-front-door-service"></a><a name="frontdoor"></a>Azure Front Door Service
Azure Front Door Service vous permet de définir, de gérer et de superviser le routage global de votre trafic web en privilégiant l’optimisation des performances et le basculement instantané global à des fins de haute disponibilité. Avec Front Door, vous pouvez transformer vos applications grand public et professionnelles multirégions en applications, API et contenus modernes fiables, personnalisés et hautes performances bénéficiant avec Azure d’une audience mondiale. Pour plus d’informations, consultez [Azure Front Door](../frontdoor/front-door-overview.md).

:::image type="content" source="./media/networking-overview/front-door-visual-diagram.png" alt-text="Vue d’ensemble de Front Door Service":::

### <a name="traffic-manager"></a><a name="trafficmanager"></a>Traffic Manager

Azure Traffic Manager est un équilibreur de charge du trafic DNS qui vous permet de distribuer le trafic de manière optimale aux services dans toutes les régions Azure globales, tout en offrant réactivité et haute disponibilité. Traffic Manager fournit un éventail de méthodes de routage du trafic telles que la priorité, la pondération, les performances, la répartition géographique, la valeur multiple ou le sous-réseau pour distribuer le trafic. Pour plus d’informations sur les différentes méthodes de routage du trafic, consultez [Méthodes de routage de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md).

Le diagramme suivant montre le routage basé sur la priorité des points de terminaison avec Traffic Manager :

:::image type="content" source="./media/networking-overview/priority.png" alt-text="Méthode de routage du trafic « Priorité » d’Azure Traffic Manager":::

Pour plus d’informations sur Traffic Manager, consultez [Présentation d’Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md)

### <a name="load-balancer"></a><a name="loadbalancer"></a>Load Balancer
Azure Load Balancer fournit un équilibrage de charge de couche 4 hautement performant et à faible latence pour tous les protocoles UDP et TCP. Il gère les connexions entrantes et sortantes. Vous pouvez configurer des points de terminaison avec équilibrage de charge interne et public. Vous pouvez définir des règles de mappage des connexions entrantes aux destinations du pool principal, en utilisant des options d’analyse d’intégrité TCP et HTTP de façon à gérer la disponibilité du service. Pour en savoir plus sur l’équilibrage de charge, lisez l’article [Présentation de l’équilibreur de charge](../load-balancer/load-balancer-overview.md).

L’illustration suivante montre une application multiniveau sur Internet qui utilise à la fois l’équilibrage de charge interne et externe :

:::image type="content" source="./media/networking-overview/load-balancer.png" alt-text="Exemple avec Load Balancer":::

### <a name="application-gateway"></a><a name="applicationgateway"></a>Application Gateway
Azure Application Gateway est un équilibreur de charge du trafic web qui vous permet de gérer le trafic vers vos applications web. C’est un contrôleur de livraison d’applications (ADC) sous forme de service qui offre à vos applications plusieurs fonctionnalités d’équilibreur de charge de couche 7. Pour plus d’informations, consultez [Nouveautés d’Azure Application Gateway](../application-gateway/overview.md).

Le diagramme suivant montre le routage d’URL basé sur le chemin avec Application Gateway.

:::image type="content" source="./media/networking-overview/figure1-720.png" alt-text="Exemple avec Application Gateway":::

## <a name="network-monitoring-services"></a><a name="monitor"></a>Services de surveillance de réseau
Cette section décrit les services réseau Azure qui permettent de superviser les ressources réseau : Network Watcher, Azure Monitor pour réseaux, ExpressRoute Monitor, Azure Monitor et TAP de réseau virtuel.

### <a name="network-watcher"></a><a name="networkwatcher"></a>Network Watcher
Azure Network Watcher offre des outils permettant d’effectuer un monitoring et des diagnostics, d’afficher les métriques et d’activer et de désactiver les journaux d’activité pour les ressources se trouvant sur un réseau virtuel Azure. Pour plus d’informations, consultez [Présentation de Network Watcher](../network-watcher/network-watcher-monitoring-overview.md?toc=%2fazure%2fnetworking%2ftoc.json).

### <a name="azure-monitor-for-networks-preview"></a>Azure Monitor pour réseaux (préversion)
Azure Monitor pour réseaux offre une vue complète de l’intégrité et des métriques de toutes les ressources réseau déployées, sans aucune configuration nécessaire. Il permet également d’accéder à toutes les fonctionnalités de supervision du réseau, comme le [Moniteur de connexion](../network-watcher/connection-monitor-preview.md), la [journalisation de flux pour les groupes de sécurité réseau](../network-watcher/network-watcher-nsg-flow-logging-overview.md) et [Traffic Analytics](../network-watcher/traffic-analytics.md). Pour plus d’informations, consultez [Aperçu d’Azure Monitor pour réseaux](../azure-monitor/insights/network-insights-overview.md?toc=%2fazure%2fnetworking%2ftoc.json).

### <a name="expressroute-monitor"></a><a name="expressroutemonitor"></a>Moniteur ExpressRoute
Pour en savoir plus sur l’affichage des métriques de circuit ExpressRoute, les journaux de ressources et les alertes, consultez [Supervision, métriques et alertes ExpressRoute](../expressroute/expressroute-monitoring-metrics-alerts.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="azure-monitor"></a><a name="azuremonitor"></a>Azure Monitor
Azure Monitor optimise la disponibilité et les performances de vos applications en fournissant une solution complète pour collecter, analyser et agir sur les données de télémétrie de vos environnements cloud et locaux. Il vous aide à comprendre le fonctionnement de vos applications et identifie de façon proactive les problèmes qui les affectent et les ressources dont elles dépendent. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Monitor](../azure-monitor/overview.md?toc=%2fazure%2fnetworking%2ftoc.json).
### <a name="virtual-network-tap"></a><a name="vnettap"></a>TAP de réseau virtuel
Le TAP (point d’accès terminal) de réseau virtuel Azure vous permet de diffuser en continu votre trafic réseau de machine virtuelle vers un collecteur de paquets réseau ou un outil analytique. Le collecteur ou l’outil analytique est fourni par une [appliance virtuelle réseau](https://azure.microsoft.com/solutions/network-appliances/) partenaire.

L’image suivante illustre le fonctionnement d’un TAP de réseau virtuel :

:::image type="content" source="./media/networking-overview/virtual-network-tap-architecture.png" alt-text="Fonctionnement d’un TAP de réseau virtuel":::

Pour plus d’informations, consultez [Présentation du TAP de réseau virtuel](../virtual-network/virtual-network-tap-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Créez votre premier réseau virtuel et connectez-y plusieurs machines virtuelles en suivant la procédure décrite dans l’article [Création d’un réseau virtuel](../virtual-network/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
- Connectez votre ordinateur à un réseau virtuel en suivant la procédure décrite dans l’article [Configuration d’une connexion point à site](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
- Équilibrez la charge de trafic Internet sur les serveurs publics en effectuant les étapes décrites dans l’article [Créer un équilibrage de charge accessible sur Internet](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fnetworking%2ftoc.json).
