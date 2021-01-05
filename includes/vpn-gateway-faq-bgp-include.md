---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 09/17/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 649c5805c600b6282be6d05fefb59cecaf249f4f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92526098"
---
### <a name="is-bgp-supported-on-all-azure-vpn-gateway-skus"></a>Le protocole BGP est-il pris en charge sur toutes les références de passerelle VPN Azure ?
Le protocole BGP est pris en charge sur toutes les références SKU de passerelle VPN Azure, à l’exception de la référence SKU De base.

### <a name="can-i-use-bgp-with-azure-policy-vpn-gateways"></a>Est-ce que je peux utiliser BGP avec les passerelles VPN Azure Policy ?
Non, BGP est pris en charge uniquement sur les passerelles VPN basées sur des routes.

### <a name="what-asns-autonomous-system-numbers-can-i-use"></a>Est-ce que je peux utiliser des numéros ASN (Autonomous System Numbers) ?
Vous pouvez utiliser vos propres ASN publics ou privés pour vos réseaux locaux et vos réseaux virtuels Azure. Vous ne pouvez pas utiliser les plages réservées par Azure ou l’IANA.

Les ASN suivants sont réservés par Azure ou l’IANA :
* Numéros ASN réservés par Azure :
  * ASN publics : 8074, 8075, 12076
  * ASN privés : 65515, 65517, 65518, 65519, 65520
* ASN [réservés par l’IANA](http://www.iana.org/assignments/iana-as-numbers-special-registry/iana-as-numbers-special-registry.xhtml) :
   * 23456, 64496-64511, 65535-65551 et 429496729.

Vous ne pouvez pas spécifier ces ASN pour vos périphériques VPN locaux quand vous les connectez à des passerelles VPN Azure.

### <a name="can-i-use-32-bit-4-byte-asns"></a>Est-ce que je peux utiliser des ASN 32 bits (4 octets) ?
Oui, la passerelle VPN prend désormais en charge les ASN 32 bits (4 octets). Pour configurer en utilisant un ASN au format décimal, utilisez PowerShell, Azure CLI ou le kit SDK Azure.

### <a name="what-private-asns-can-i-use"></a>Quels sont les ASN privés que je peux utiliser ?
Les plages utilisables d’ASN privés sont les suivantes :

* 64512-65514 et 65521-65534

Ces ASN ne sont pas réservés par IANA ou Azure et peuvent donc être affectés à votre passerelle VPN Azure.

### <a name="what-address-does-vpn-gateway-use-for-bgp-peer-ip"></a>Quelle adresse la passerelle VPN utilise pour l’IP de pair BGP ?

Par défaut, la passerelle VPN alloue une seule adresse IP de la plage *GatewaySubnet* pour les passerelles VPN de type actif/passif, ou deux adresses IP pour les passerelles VPN actif-actif. Ces adresses sont allouées automatiquement lorsque vous créez la passerelle VPN. Vous pouvez obtenir l’adresse IP BGP qui a été effectivement allouée en utilisant PowerShell ou en la localisant dans le portail Azure. Dans PowerShell, utilisez **Get-AzVirtualNetworkGateway** et recherchez la propriété **bgpPeeringAddress** . Dans le portail Azure, dans la page **Configuration de la passerelle** , recherchez la propriété **Configurer l’ASN BGP** .

Si vos routeurs VPN locaux utilisent des adresses IP **APIPA** (169.254. x. x) comme adresses IP BGP, vous devez spécifier une autre **adresse IP BGP Azure APIPA** sur votre passerelle VPN Azure. La passerelle VPN Azure sélectionne l’adresse APIPA à utiliser avec le pair BGP APIPA local spécifié dans la passerelle de réseau local ou l’adresse IP privée pour un pair BGP local non-APIPA. Pour plus d’informations, consultez [Configurer BGP](../articles/vpn-gateway/bgp-howto.md).

### <a name="what-are-the-requirements-for-the-bgp-peer-ip-addresses-on-my-vpn-device"></a>Quelles sont les conditions requises concernant les adresses IP de pair BGP sur mon périphérique VPN ?
Votre adresse de pair BGP locale ne doit pas être identique à l’adresse IP publique de votre périphérique VPN ni issue de l’espace d’adressage VNet de la passerelle VPN. Utilisez une adresse IP différente sur le périphérique VPN de votre adresse IP BGP homologue. Il peut s’agir d’une adresse affectée à l’interface de bouclage sur l’appareil (soit une adresse IP normale, soit une adresse APIPA). Si votre appareil utilise une adresse APIPA pour BGP, vous devez spécifier une adresse IP BGP APIPA sur votre passerelle VPN Azure, comme décrit dans [Configurer BGP](../articles/vpn-gateway/bgp-howto.md). Spécifiez cette adresse sur la passerelle de réseau local correspondante, représentant l’emplacement.

### <a name="what-should-i-specify-as-my-address-prefixes-for-the-local-network-gateway-when-i-use-bgp"></a>Que dois-je spécifier comme préfixes d’adresse pour la passerelle de réseau local lorsque j’utilise BGP ?

> [!IMPORTANT]
>
>Il y a un changement par rapport à la spécification précédemment documentée. Si vous utilisez BGP pour une connexion, laissez le champ **Espace d’adressage** vide pour la ressource de passerelle de réseau local correspondante. La passerelle VPN Azure ajoute en interne une route hôte à l’IP de pair BGP local sur le tunnel IPsec. N’ajoutez pas la route /32 dans le champ **Espace d’adressage** . C’est redondant et si vous utilisez une adresse APIPA comme IP BGP du périphérique VPN local, vous ne pouvez pas l’ajouter à ce champ. Si vous ajoutez d’autres préfixes dans le champ Espace d’adressage, ceux-ci sont ajoutés comme **routes statiques** sur la passerelle VPN Azure, en plus des routes apprises via BGP.
>

### <a name="can-i-use-the-same-asn-for-both-on-premises-vpn-networks-and-azure-virtual-networks"></a>Est-ce que je peux utiliser le même ASN pour les réseaux VPN locaux et les réseaux virtuels Azure ?
Non, vous devez attribuer des ASN différents à vos réseaux locaux et vos réseaux virtuels Azure si vous les interconnectez avec le protocole BGP. L’ASN 65515 est attribué aux passerelles VPN Azure par défaut, et ce, que le protocole BGP soit activé ou non pour la connectivité entre les sites locaux. Vous pouvez remplacer cette valeur par défaut en attribuant un ASN différent quand vous créez la passerelle VPN, ou changer l’ASN après avoir créé la passerelle. Vous devez affecter vos ASN locaux aux passerelles de réseau local Azure correspondantes.

### <a name="what-address-prefixes-will-azure-vpn-gateways-advertise-to-me"></a>Quels préfixes d’adresse les passerelles VPN Azure publieront-elles pour moi ?
Les passerelles publient les routes suivantes sur vos périphériques BGP locaux :

* Vos préfixes d’adresse de réseau virtuel.
* Les préfixes d’adresse de chaque passerelle de réseau local connectée à la passerelle VPN Azure.
* Les routes apprises des autres sessions de peering BGP connectées à la passerelle VPN Azure, à l’exception de la route par défaut ou des routes qui se chevauchent avec un préfixe de réseau virtuel.

### <a name="how-many-prefixes-can-i-advertise-to-azure-vpn-gateway"></a>Combien de préfixes est-ce que je peux publier sur la passerelle VPN Azure ?
La passerelle VPN Azure prend en charge jusqu’à 4 000 préfixes. La session BGP s’arrête si le nombre de préfixes dépasse la limite.

### <a name="can-i-advertise-default-route-00000-to-azure-vpn-gateways"></a>Puis-je publier l’itinéraire par défaut (0.0.0.0/0) vers les passerelles VPN Azure ?
Oui. Notez que cela force tout le trafic de sortie de réseau virtuel vers votre site local. Cela empêche également les machines virtuelles du réseau virtuel d’accepter la communication publique directement depuis Internet, par exemple RDP ou SSH depuis Internet vers les machines virtuelles.

### <a name="can-i-advertise-the-exact-prefixes-as-my-virtual-network-prefixes"></a>Est-ce que je peux publier les mêmes préfixes que ceux de mon adresse de réseau virtuel ?

Non, la publication des mêmes préfixes que ceux de votre adresse de réseau virtuel sera bloquée ou filtrée par Azure. Toutefois, vous pouvez publier un préfixe qui soit un sur-ensemble de ce qe vous avez dans votre réseau virtuel. 

Par exemple, votre réseau virtuel peut utiliser l’espace d’adresse 10.0.0.0/16 et vous pouvez publier 10.0.0.0/8. Par contre, vous ne pouvez pas publier 10.0.0.0/16 ou 10.0.0.0/24.

### <a name="can-i-use-bgp-with-my-connections-between-virtual-networks"></a>Est-ce que je peux utiliser BGP avec mes connexions entre des réseaux virtuels ?
Oui, vous pouvez utiliser BGP pour les connexions entre sites locaux et entre réseaux virtuels.

### <a name="can-i-mix-bgp-with-non-bgp-connections-for-my-azure-vpn-gateways"></a>Puis-je combiner des connexions BGP et non-BGP pour mes passerelles VPN Azure ?
Oui. Vous pouvez combiner des connexions BGP et non-BGP pour la même passerelle VPN Azure.

### <a name="does-azure-vpn-gateway-support-bgp-transit-routing"></a>La passerelle VPN Azure prend-elle en charge le routage de transit BGP ?
Oui, le routage de transit BGP est pris en charge. Cependant, les passerelles VPN Azure ne publient pas les routes par défaut sur les autres pairs BGP. Pour activer le routage de transit via plusieurs passerelles VPN Azure, vous devez activer BGP sur toutes les connexions intermédiaires entre réseaux virtuels. Pour plus d’informations, consultez [À propose de BGP](../articles/vpn-gateway/vpn-gateway-bgp-overview.md).

### <a name="can-i-have-more-than-one-tunnel-between-an-azure-vpn-gateway-and-my-on-premises-network"></a>Est-ce que je peux créer plusieurs tunnels entre une passerelle VPN Azure et mon réseau local ?
Oui, vous pouvez établir plusieurs tunnels VPN de site à site (S2S) entre une passerelle VPN Azure et votre réseau local. Notez que tous ces tunnels sont comptabilisés dans le nombre total de tunnels pour vos passerelles VPN Azure et que vous devez activer BGP sur les deux tunnels.

Par exemple, si vous établissez deux tunnels redondants entre votre passerelle VPN Azure et l’un de vos réseaux locaux, 2 tunnels sont utilisés sur le quota total de votre passerelle VPN Azure.

### <a name="can-i-have-multiple-tunnels-between-two-azure-virtual-networks-with-bgp"></a>Est-ce que je peux avoir plusieurs tunnels entre deux réseaux virtuels Azure avec BGP ?
Oui, mais au moins une des passerelles de réseau virtuel doit être dans une configuration active-active.

### <a name="can-i-use-bgp-for-s2s-vpn-in-an-azure-expressroute-and-s2s-vpn-coexistence-configuration"></a>Est-ce que je peux utiliser BGP pour un VPN S2S dans une configuration de coexistence Azure ExpressRoute et S2S ?
Oui. 

### <a name="what-should-i-add-to-my-on-premises-vpn-device-for-the-bgp-peering-session"></a>Que dois-je ajouter à mon périphérique VPN local pour la session de peering BGP ?
Ajoutez une route hôte de l’adresse IP de pair BGP Azure sur votre périphérique VPN. Cette route pointe vers le tunnel VPN S2S IPsec. Par exemple, si l’IP de pair VPN Azure est 10.12.255.30, ajoutez une route hôte pour 10.12.255.30 avec une interface de tronçon suivant de l’interface de tunnel IPsec correspondante sur votre périphérique VPN.

### <a name="does-the-virtual-network-gateway-support-bfd-for-s2s-connections-with-bgp"></a>La passerelle de réseau virtuel prend-elle en charge BFD pour les connexions S2S avec BGP ?
Non. La détection de transfert bidirectionnel (BFD) est un protocole que vous pouvez utiliser avec BGP pour détecter les temps d’arrêt voisins plus vite que si vous utilisez des connexions actives (keepalives) BGP standard. BFD utilise des minuteurs secondaires conçus pour fonctionner dans des environnements LAN, mais pas avec les connexions WAN ou sur l’Internet public.

Pour les connexions sur l’Internet public, le fait d’avoir des paquets en retard ou même abandonnés n’est pas rare, donc introduire ces minuteurs agressifs peut ajouter une instabilité. Cette instabilité risque d’entraîner le blocage des routes par BGP. Comme alternative, vous pouvez configurer votre appareil local avec des minuteurs inférieurs à l’intervalle de conservation de 60 secondes par défaut et le minuteur de suspension de 180 secondes. Vous obtiendrez un temps de convergence plus rapide.
