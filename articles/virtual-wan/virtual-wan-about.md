---
title: Présentation du WAN virtuel Azure | Microsoft Docs
description: Découvrez plus d’informations sur la connectivité branche à branche scalable et automatisée de Virtual WAN, les régions disponibles et les partenaires.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: overview
ms.date: 09/22/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to understand what Virtual WAN is and if it is the right choice for my Azure network.
ms.openlocfilehash: 641ed73f507d35d7af548d0164ef8e80979be217
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96530474"
---
# <a name="what-is-azure-virtual-wan"></a>Qu’est-ce que le WAN virtuel Azure ?

Azure Virtual WAN est un service de mise en réseau qui combine un grand nombre de fonctionnalités de mise en réseau, de sécurité et de routage pour fournir une interface opérationnelle unique. Ces fonctionnalités sont les suivantes : connectivité de branche (via une automatisation de la connectivité à partir d’appareils partenaires d’Azure Virtual WAN, tels que SD-WAN ou un CPE VPN), connectivité VPN site à site, connectivité VPN d’utilisateurs distants (point à site), connectivité privée (ExpressRoute), connectivité intra-cloud (connectivité transitive pour les réseaux virtuels), interconnectivité ExpressRoute de VPN, routage, Pare-feu Azure et chiffrement de connexion privée. Il n’est pas nécessaire d’avoir tous ces cas d’usage pour commencer à utiliser Virtual WAN. Vous pouvez simplement commencer avec un seul cas d’usage puis ajuster votre réseau au fur et à mesure de son évolution.

Virtual WAN repose sur une architecture hub-and-spoke avec des fonctionnalités de mise à l’échelle et de performances intégrées pour les branches (appareils VPN/SD-WAN), les utilisateurs (clients Azure VPN, openVPN ou IKEv2), les circuits ExpressRoute et les réseaux virtuels. Il offre une [architecture de réseau de transit global](virtual-wan-global-transit-network-architecture.md) où le « hub » du réseau hébergé sur le cloud permet une connectivité transitive entre les points de terminaison qui peut être répartie sur différents types de « rayons » (spoke).

Les régions Azure servent de hubs auxquels vous pouvez vous connecter. Tous les hubs sont connectés selon un maillage complet dans un réseau étendu virtuel standard, ce qui permet à l’utilisateur d’utiliser facilement le réseau principal Microsoft pour se connecter à n’importe quel spoke. Pour la connectivité entre membres spokes avec des appareils SD-WAN/VPN, les utilisateurs peuvent soit la configurer manuellement dans Azure Virtual WAN, soit utiliser la solution de partenaire CPE Virtual WAN (SD-WAN/VPN) pour configurer la connectivité à Azure. Nous disposons d’une liste de partenaires qui prennent en charge l’automatisation de la connectivité (c’est-à-dire la capacité d’exporter les informations relatives à l’appareil dans Azure, de télécharger la configuration Azure et d’établir la connectivité) avec Azure Virtual WAN. Pour plus d’informations sur les emplacements, consultez l’article [Partenaires et emplacements Virtual WAN](virtual-wan-locations-partners.md).

![Diagramme WAN virtuel](./media/virtual-wan-about/virtualwan1.png)

Cet article fournit un aperçu rapide de la connectivité réseau dans Azure Virtual WAN. Le WAN virtuel offre les avantages suivants :

* **Solutions de connectivité intégrée dans une topologie Hub-and-Spoke :** Automatisez la configuration et la connectivité site à site entre les sites locaux et un hub Azure.
* **Installation et configuration de spokes automatisés :** Connectez vos charges de travail et vos réseaux virtuels au hub Azure sans interruption.
* **Résolution de problèmes intuitive :** Vous pouvez voir le flux de bout en bout au sein d’Azure et utiliser ces informations pour prendre les mesures nécessaires.

## <a name="basic-and-standard-virtual-wans"></a><a name="basicstandard"></a>WAN virtuels De base et Standard

Il existe deux types de réseaux étendus (WAN) : De base et Standard. Le tableau suivant présente les configurations disponibles pour chaque type.

[!INCLUDE [Basic and Standard SKUs](../../includes/virtual-wan-standard-basic-include.md)]

Pour connaître la procédure de mise à niveau d'un WAN virtuel, consultez [Procéder à la mise à niveau d'un WAN virtuel De base vers le niveau Standard](upgrade-virtual-wan.md).

## <a name="architecture"></a><a name="architecture"></a>Architecture

Pour plus d’informations sur l’architecture du WAN virtuel et sur la migration vers un WAN virtuel, consultez les articles suivants :

* [Architecture du WAN virtuel](migrate-from-hub-spoke-topology.md)
* [Architecture du réseau de transit global](virtual-wan-global-transit-network-architecture.md)

## <a name="virtual-wan-resources"></a><a name="resources"></a>Ressources du WAN virtuel

Pour configurer un WAN virtuel de bout en bout, vous devez créer les ressources suivantes :

* **virtualWAN :** La ressource virtualWAN représente une superposition virtuelle de votre réseau Azure et est une collection de plusieurs ressources. Elle contient des liens vers tous les hubs virtuels que vous souhaitez faire figurer dans le WAN virtuel. Les ressources du WAN virtuel sont isolées les unes des autres et ne peuvent pas contenir de hub commun. Les hubs virtuels d'un WAN virtuel ne communiquent pas entre eux.

* **Hub :** Un hub virtuel est un réseau virtuel géré par Microsoft. Le hub contient différents points de terminaison de service pour activer la connectivité. À partir de votre réseau local (vpnsite), vous pouvez vous connecter à une passerelle VPN à l'intérieur du hub virtuel, connecter des circuits ExpressRoute à un hub virtuel, ou même connecter des utilisateurs mobiles à une passerelle point à site dans le hub virtuel. Le hub est le cœur de votre réseau au sein d’une région spécifique. Il ne peut exister qu’un seul hub par région Azure.

  Une passerelle de hub virtuel est différente d’une passerelle de réseau virtuel que vous utilisez pour les services ExpressRoute et Passerelle VPN. Par exemple, quand vous utilisez Virtual WAN, vous ne créez pas de connexion site à site directe entre votre site local et votre réseau virtuel. Au lieu de cela, vous établissez une connexion site à site avec le hub. Le trafic passe toujours par la passerelle du hub. Cela signifie que vos réseaux virtuels n’ont pas besoin d’avoir leur propre passerelle de réseau virtuel. Le WAN virtuel simplifie la mise à l’échelle de vos réseaux virtuels via le hub virtuel et la passerelle de hub virtuel.

* **Connexion entre hub et réseau virtuel :** La ressource Connexion entre hub et réseau virtuel permet de connecter facilement le hub à votre réseau virtuel.

* **Connexion de hub à hub :** les hubs sont tous connectés les uns aux autres dans un WAN virtuel. Cela signifie qu'une branche, un utilisateur ou un réseau virtuel connecté à un hub local peut communiquer avec une autre branche ou un autre réseau virtuel en utilisant l'intégralité de l'architecture maillée des hubs connectés. Vous pouvez également connecter des réseaux virtuels au sein d'un hub transitant par le hub virtuel, ainsi que des réseaux virtuels inter-hub, en utilisant l'infrastructure connectée hub à hub.

* **Table de routage de hub :**  Vous pouvez créer une route de hub virtuel et l’appliquer à la table de routage de hub virtuel. Vous pouvez appliquer plusieurs itinéraires à la table de routage du hub virtuel.

**Ressources Virtual WAN supplémentaires**

* **Site :** Cette ressource est utilisée pour les connexions site à site uniquement. La ressource de site est **vpnsite**. Elle représente votre périphérique VPN local et ses paramètres. En faisant appel à un partenaire de WAN virtuel, vous disposez d’une solution intégrée pour exporter automatiquement ces informations vers Azure.

## <a name="types-of-connectivity"></a><a name="connectivity"></a>Types de connectivité

Virtual WAN permet les types de connectivité suivants : VPN site à site, VPN utilisateur (point à site) et ExpressRoute.

### <a name="site-to-site-vpn-connections"></a><a name="s2s"></a>Connexion VPN site à site

Vous pouvez vous connecter à vos ressources Azure via une connexion IPsec/IKE (IKEv2) de site à site. Pour plus d’informations, consultez [Créer une connexion site à site à l’aide de Virtual WAN](virtual-wan-site-to-site-portal.md). 

Ce type de connexion nécessite un périphérique VPN ou un périphérique de partenaire Virtual WAN. Les partenaires Virtual WAN assurent l’automatisation de la connectivité, c’est-à-dire la capacité d’exporter les informations sur l’appareil dans Azure, de télécharger la configuration Azure et d’établir la connectivité avec le hub Azure Virtual WAN. Pour obtenir la liste des partenaires et des emplacements disponibles, consultez l’article [Partenaires et emplacements Virtual WAN](virtual-wan-locations-partners.md). Si votre fournisseur d’appareil VPN/SD-WAN n’est pas listé dans le lien mentionné, vous pouvez simplement utiliser l’instruction pas à pas [Créer une connexion site à site à l’aide de Virtual WAN](virtual-wan-site-to-site-portal.md) pour configurer la connexion.

### <a name="user-vpn-point-to-site-connections"></a><a name="uservpn"></a>Connexions VPN utilisateur (point à site)

Vous pouvez vous connecter à vos ressources Azure via une connexion IPsec/IKE (IKEv2) ou OpenVPN. Pour utiliser ce type de connexion, un client VPN doit être configuré sur l'ordinateur client. Pour plus d'informations, consultez [Créer une connexion point à site](virtual-wan-point-to-site-portal.md).

### <a name="expressroute-connections"></a><a name="er"></a>Connexions ExpressRoute
ExpressRoute vous permet de connecter votre réseau local à Azure via une connexion privée. Pour établir la connexion, consultez [Créer une connexion ExpressRoute à l’aide de Virtual WAN](virtual-wan-expressroute-portal.md).

### <a name="hub-to-vnet-connections"></a><a name="hub"></a>Connexions de hub à réseau virtuel

Vous pouvez connecter un réseau virtuel Azure à un hub virtuel. Pour plus d’informations, consultez [Connecter votre réseau virtuel à un hub](virtual-wan-site-to-site-portal.md#vnet).

### <a name="transit-connectivity"></a><a name="transit"></a>Connectivité de transit

#### <a name="transit-connectivity-between-vnets"></a><a name="transit-vnet"></a>Connectivité de transit entre réseaux virtuels

Virtual WAN autorise la connectivité de transit entre les réseaux virtuels. Les réseaux virtuels se connectent à un hub virtuel via une connexion de réseau virtuel. La présence d’un routeur dans chaque hub virtuel rend possible la connectivité de transit entre les réseaux virtuels dans un **réseau étendu virtuel Standard**. Ce routeur est instancié quand le hub virtuel est créé pour la première fois.

Le routeur peut avoir quatre états de routage : Provisionné, Provisionnement, Échec ou Aucun. Vous pouvez retrouver l’**état du routage** dans le portail Azure en accédant à la page Hub virtuel.

* Un état **Aucun** indique que le hub virtuel n’a pas provisionné le routeur. Cela peut se produire si le réseau étendu virtuel est de type *De base* ou que le hub virtuel a été déployé avant que le service ne soit mis à disposition.
* Un état **Échec** indique un échec pendant l’instanciation. Pour instancier ou réinitialiser le routeur, vous pouvez rechercher l’option **Réinitialiser le routeur** en accédant à la page de vue d’ensemble du hub virtuel dans le portail Azure.

Chaque routeur de hub virtuel prend en charge un débit agrégé pouvant atteindre 50 Gbits/s. La connectivité entre les connexions de réseau virtuel suppose une charge de travail totale de 2000 machines virtuelles sur tous les réseaux virtuels connectés à un même hub virtuel.

#### <a name="transit-connectivity-between-vpn-and-expressroute"></a><a name="transit-er"></a>Connectivité de transit entre le réseau privé virtuel et ExpressRoute

Virtual WAN permet la connectivité de transit entre le réseau privé virtuel et ExpressRoute. Cela implique que les sites connectés au VPN ou les utilisateurs distants puissent communiquer avec les sites connectés à ExpressRoute. Il existe également une hypothèse implicite selon laquelle l'**indicateur Branche à branche** est activé et le protocole BGP est pris en charge pour les connexions VPN et ExpressRoute. Cet indicateur se trouve dans les paramètres Azure Virtual WAN du portail Azure. La gestion des routes est assurée en totalité par le routeur de hub virtuel, qui permet également la connectivité de transit entre les réseaux virtuels.

### <a name="custom-routing"></a><a name="routing"></a>Routage personnalisé

Virtual WAN fournit des améliorations de routage avancées. Possibilité de définir des tables de routage personnalisées, d'optimiser le routage du réseau virtuel avec association et propagation d'itinéraires, de regrouper de façon logique les tables de routage avec des étiquettes et de simplifier de nombreux scénarios de routage d'appliance virtuelle réseau (NVA) ou de services partagés.

### <a name="global-vnet-peering"></a><a name="global"></a>Appairage de réseaux virtuels global

L’appairage de réseaux virtuels global fournit un mécanisme permettant de connecter deux réseaux virtuels dans différentes régions. Dans Virtual WAN, les connexions de réseau virtuel connectent les réseaux virtuels à des hubs virtuels. L’utilisateur n’a pas besoin de configurer explicitement un appairage de réseaux virtuels global. Les réseaux virtuels connectés à un hub virtuel dans une même région entraînent des frais d’appairage de réseaux virtuels. Les réseaux virtuels connectés à un hub virtuel dans une région différente entraînent des frais d’appairage de réseaux virtuels global.

### <a name="expressroute-traffic-encryption"></a><a name="encryption"></a>Chiffrement du trafic ExpressRoute

Azure Virtual WAN offre la possibilité de chiffrer votre trafic ExpressRoute. Cette technique assure un transit chiffré entre les réseaux locaux et les réseaux virtuels Azure par ExpressRoute, sans passer par l’Internet public ni utiliser des IP publiques. Pour plus d’informations, consultez [IPsec sur ExpressRoute pour Virtual WAN](vpn-over-expressroute.md).

## <a name="locations"></a><a name="locations"></a>Emplacements

Pour plus d’informations sur les emplacements, consultez l’article [Partenaires et emplacements Virtual WAN](virtual-wan-locations-partners.md).

## <a name="route-tables-for-basic-and-standard-virtual-wans"></a><a name="route"></a>Tables de routage pour les réseaux étendus virtuels De base et Standard

Les tables de routage disposent désormais de fonctionnalités d’association et de propagation. Une table de routage préexistante est une table de routage qui n’a pas ces fonctionnalités. Si le routage de hub comporte des itinéraires préexistants et que vous souhaitez utiliser les nouvelles fonctionnalités, tenez compte des éléments suivants :

* **Clients de réseau virtuel Standard avec des routes préexistantes dans le hub virtuel** : En présence d'itinéraires préexistants dans la section Routage du hub sur le portail Azure, vous devez d'abord les supprimer, puis essayer de créer de nouvelles tables de routage (disponible dans la section Tables de routage du hub sur le portail Azure). Il est vivement conseillé d’effectuer l’étape de suppression pour tous les hubs dans une instance Virtual WAN.

* **Clients de réseau virtuel De base avec des routes préexistantes dans le hub virtuel** : En présence d'itinéraires préexistants dans la section Routage du hub sur le portail Azure, vous devez d'abord les supprimer, puis **mettre à niveau** votre réseau étendu virtuel De base vers un réseau étendu virtuel Standard. Consultez [Mettre à niveau un réseau étendu virtuel De base vers le type Standard](upgrade-virtual-wan.md). Il est vivement conseillé d’effectuer l’étape de suppression pour tous les hubs dans une instance Virtual WAN.

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

[!INCLUDE [Virtual WAN FAQ](../../includes/virtual-wan-faq-include.md)]

## <a name="whats-new"></a><a name="new"></a>Nouveautés

Abonnez-vous au flux RSS et consultez les dernières mises à jour des fonctionnalités Virtual WAN sur la page [Mises à jour Azure](https://azure.microsoft.com/updates/?category=networking&query=VIRTUAL%20WAN).

## <a name="next-steps"></a>Étapes suivantes

[Créer une connexion site à site à l’aide de Virtual WAN](virtual-wan-site-to-site-portal.md)
