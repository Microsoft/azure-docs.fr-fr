---
title: À propos de la conception de la passerelle VPN Azure
description: Découvrez comment concevoir une topologie de passerelle VPN pour vous connecter à des réseaux virtuels Azure.
services: vpn-gateway
author: cherylmc
Customer intent: As someone with a basic network background, but is new to Azure, I want to understand the capabilities of Azure VPN Gateway so that I can securely connect to my Azure virtual networks.
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 61732f66aef58f5a9edcb9e095782e19e8aaffdd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91397213"
---
# <a name="vpn-gateway-design"></a>Conception de la passerelle VPN

Il est important de savoir qu’il existe différentes configurations disponibles pour les connexions aux passerelles VPN. Vous devez déterminer la configuration qui correspond le mieux à vos besoins. Dans les sections ci-dessous, vous pouvez afficher des informations de conception et des diagrammes de topologie sur les connexions de passerelle VPN suivantes. Utilisez les graphiques et les descriptions pour sélectionner la topologie de connexion répondant à vos besoins. Le graphique présente les principales topologies de base, mais il est possible de créer des configurations plus complexes à l’aide des diagrammes.

## <a name="site-to-site-and-multi-site-ipsecike-vpn-tunnel"></a><a name="s2smulti"></a>Site à site et multi-sites (tunnel VPN IPsec/IKE)

### <a name="site-to-site"></a><a name="S2S"></a>Site à site

Une connexion par passerelle VPN site à site (S2S) est une connexion via un tunnel VPN IPsec/IKE (IKEv1 ou IKEv2). Les connexions S2S peuvent être utilisées pour les configurations hybrides et entre différents locaux. Une connexion site à site nécessite un appareil VPN local auquel est assignée une adresse IP publique. Pour plus d’informations sur la sélection d’un périphérique VPN, consultez [VPN Gateway FAQ - VPN devices](vpn-gateway-vpn-faq.md#s2s) (FAQ sur la passerelle VPN - Périphériques VPN).

![Exemple de connexion site à site de passerelle VPN Azure](./media/design/vpngateway-site-to-site-connection-diagram.png)

La passerelle VPN peut être configurée en mode actif/en attente à l’aide d’une adresse IP publique ou en mode actif/actif à l’aide de deux adresses IP publiques. En mode actif/de secours, un tunnel IPsec est actif et l’autre est en veille. Dans cette configuration, le trafic transite par le tunnel actif et, si un problème se produit avec ce tunnel, le trafic bascule vers le tunnel de secours. La configuration d’une passerelle VPN en mode actif/actif est *recommandée* dans laquelle les deux tunnels IPsec sont simultanément actifs, les données transitant par les deux tunnels en même temps. L’un des avantages supplémentaires du mode actif/actif est que les clients bénéficient de débits plus élevés.

### <a name="multi-site"></a><a name="Multi"></a>Multi-sites

Ce type de connexion est une variante de la connexion site à site. Vous créez plusieurs connexions VPN à partir de votre passerelle de réseau virtuel, généralement en vous connectant à plusieurs sites locaux. Lorsque vous travaillez avec plusieurs connexions, vous devez utiliser un type de VPN basé sur l’itinéraire (équivalent d’une passerelle dynamique pour les réseaux virtuels classiques). Chaque réseau virtuel ne pouvant disposer que d’une seule passerelle de réseau virtuel, toutes les connexions passant par la passerelle partagent la bande passante disponible. Ce type de connexion est souvent appelé connexion « multi-sites ».

![Exemple de connexion multisites de passerelle VPN Azure](./media/design/vpngateway-multisite-connection-diagram.png)

### <a name="deployment-models-and-methods-for-site-to-site-and-multi-site"></a>Modèles et méthodes de déploiement pour les connexions site à site et multi-sites

[!INCLUDE [site-to-site and multi-site table](../../includes/vpn-gateway-table-site-to-site-include.md)]

## <a name="point-to-site-vpn"></a><a name="P2S"></a>VPN de point à site

Une connexion par passerelle VPN point à site (P2S) vous permet de créer une connexion sécurisée à votre réseau virtuel à partir d’un ordinateur de client individuel. Une connexion P2S est établie en étant démarrée à partir de l’ordinateur client. Cette solution est utile pour les télétravailleurs souhaitant se connecter à un réseau virtuel à partir d’un emplacement distant, comme depuis leur domicile ou pendant une conférence. De même, l’utilisation d’un VPN P2S est une solution utile qui constitue une alternative au VPN Site à Site (S2S) lorsqu’un nombre restreint de clients doit se connecter à un réseau virtuel.

Contrairement aux connexions S2S, les connexions P2S ne nécessitent pas d’adresse IP publique locale ni de périphérique VPN. Les connexions P2S peuvent être utilisées avec des connexions S2S via la même passerelle VPN, dans la mesure où toutes les exigences de configuration des deux types de connexion sont compatibles. Pour plus d’informations sur les connexions point à site, consultez [À propos des VPN point à site](point-to-site-about.md).

![Exemple de connexion de point à site de passerelle VPN Azure](./media/design/point-to-site.png)

### <a name="deployment-models-and-methods-for-p2s"></a>Méthodes et modèles de déploiement pour les connexions P2S

[!INCLUDE [vpn-gateway-table-site-to-site](../../includes/vpn-gateway-table-point-to-site-include.md)]

## <a name="vnet-to-vnet-connections-ipsecike-vpn-tunnel"></a><a name="V2V"></a>Connexions de réseau virtuel à réseau virtuel (tunnel VPN IPsec/IKE)

La connexion entre deux réseaux virtuels est semblable à la connexion d’un réseau virtuel à un emplacement de site local. Les deux types de connectivité font appel à une passerelle VPN pour offrir un tunnel sécurisé utilisant Ipsec/IKE. Vous pouvez même combiner une communication de réseau virtuel à réseau virtuel avec des configurations de connexion multi-sites. Vous établissez ainsi des topologies réseau qui combinent une connectivité entre différents locaux et une connectivité entre différents réseaux virtuels.

Les réseaux virtuels que vous connectez peuvent être situés :

* dans la même région ou dans des régions différentes
* dans le même abonnement ou dans des abonnements différents 
* dans le même modèle de déploiement ou dans des modèles de déploiement différents

![Exemple de connexion de réseau virtuel à réseau virtuel de passerelle VPN Azure](./media/design/vpngateway-vnet-to-vnet-connection-diagram.png)

### <a name="connections-between-deployment-models"></a>Connexions entre modèles de déploiement

Azure propose actuellement deux modèles de déploiement : le modèle classique et le modèle Resource Manager. Si vous utilisez Azure depuis un certain temps, vous avez probablement des machines virtuelles et des rôles d’instance Azure exécutés dans un réseau virtuel classique. Il est possible que vos nouvelles machines virtuelles et instances de rôle s’exécutent dans un réseau virtuel créé dans Resource Manager. Vous pouvez créer une connexion entre les réseaux virtuels pour permettre aux ressources dans un réseau virtuel de communiquer directement avec les ressources d’un autre réseau virtuel.

### <a name="vnet-peering"></a>Peering de réseaux virtuels

Vous pouvez utiliser le peering de réseau virtuel pour créer votre connexion, tant que votre réseau virtuel répond à certaines exigences. Le peering de réseau virtuel n’utilise pas de passerelle de réseau virtuel. Pour plus d’informations, consultez l’article [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

### <a name="deployment-models-and-methods-for-vnet-to-vnet"></a>Modèles et méthodes de déploiement pour les connexions de réseau virtuel à réseau virtuel

[!INCLUDE [vpn-gateway-table-vnet-to-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

## <a name="expressroute-private-connection"></a><a name="ExpressRoute"></a>ExpressRoute (connexion privée)

ExpressRoute vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité. Grâce à ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, comme Microsoft Azure, Microsoft 365 et CRM Online. La connectivité peut provenir d’un réseau universel (IP VPN), d’un réseau Ethernet point à point ou d’une interconnexion virtuelle via un fournisseur de connectivité dans un centre de colocalisation.

Les connexions ExpressRoute ne sont pas établies par le biais de l'Internet public. Elles offrent ainsi de meilleurs niveaux de fiabilité, de rapidité, de latence et de sécurité que les connexions classiques sur Internet.

Une connexion ExpressRoute utilise une passerelle de réseau virtuel dans le cadre de sa configuration requise. Dans une connexion ExpressRoute, la passerelle de réseau virtuel est configurée avec le type de passerelle « ExpressRoute » plutôt que « Vpn ». Alors que le trafic qui transite sur un circuit ExpressRoute n’est pas chiffré par défaut, il est possible de créer une solution qui vous permet d’envoyer le trafic chiffré sur un circuit ExpressRoute. Pour plus d’informations sur ExpressRoute, consultez [Présentation technique d’ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="site-to-site-and-expressroute-coexisting-connections"></a><a name="coexisting"></a>Coexistence de connexions ExpressRoute et de site à site

ExpressRoute est une connexion directe et privée aux services Microsoft, notamment à Azure, à partir de votre WAN, qui ne passe pas par l’Internet public. Le trafic VPN de site à site transite via l’Internet public tout en étant chiffré. La possibilité de configurer des connexions VPN de site à site et ExpressRoute pour le même réseau virtuel présente plusieurs avantages.

Vous pouvez configurer un VPN de site à site comme un chemin d’accès de basculement sécurisé pour ExpressRoute, ou utiliser des VPN de site à site pour vous connecter à des sites qui ne font pas partie de votre réseau, mais qui sont connectés via ExpressRoute. Notez que cette configuration nécessite deux passerelles de réseau virtuel pour un même réseau virtuel, une de type « Vpn », et l’autre de type « ExpressRoute ».

![Exemple de connexions coexistantes ExpressRoute et passerelle VPN](./media/design/expressroute-vpngateway-coexisting-connections-diagram.png)

### <a name="deployment-models-and-methods-for-s2s-and-expressroute-coexist"></a>Coexistence des méthodes et modèles de déploiement pour les connexions S2S et ExpressRoute

[!INCLUDE [vpn-gateway-table-coexist](../../includes/vpn-gateway-table-coexist-include.md)]

## <a name="highly-available-connections"></a><a name="highly-available"></a>Connexions hautement disponibles

Pour la planification et la conception des connexions à haut niveau de disponibilité, consultez [Connexions hautement disponibles](vpn-gateway-highlyavailable.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations, consultez la [FAQ sur la passerelle VPN](vpn-gateway-vpn-faq.md).

* Découvrez-en plus sur les [paramètres de configuration de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md).

* Pour des considérations sur le protocole BGP de la passerelle VPN, consultez [À propos du protocole BGP](vpn-gateway-bgp-overview.md).

* Consultez les [Limites du service et de l’abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

* Découvrez certaines des autres [fonctionnalités de réseau](../networking/networking-overview.md) clés d’Azure.
