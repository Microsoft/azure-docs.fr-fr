---
title: 'Interopérabilité des fonctionnalités de connectivité de back-end Azure : Détails de configuration | Microsoft Docs'
description: Cet article décrit les détails de configuration de l’initialisation (tearDown) de test que vous pouvez utiliser pour analyser l’interopérabilité entre ExpressRoute, un réseau privé virtuel (VPN) de site à site et le peering de réseau virtuel dans Azure.
documentationcenter: na
services: networking
author: rambk
manager: tracsman
ms.service: virtual-network
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/18/2018
ms.author: rambala
ms.openlocfilehash: 7be326e0f01ed6a00244c0f5b9ed6a960b2b6e0b
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "86171854"
---
# <a name="interoperability-in-azure-back-end-connectivity-features-test-configuration-details"></a>Interopérabilité des fonctionnalités de connectivité de back-end Azure : détails de la configuration de test

Cet article décrit les détails de configuration de l’[initialisation (tearDown) de test][Setup]. L’initialisation (tearDown) de test vous permet d’analyser l’interopérabilité des services de mise en réseau au niveau du plan de contrôle et du plan de données.

## <a name="spoke-vnet-connectivity-by-using-vnet-peering"></a>Connectivité de réseau virtuel spoke via le peering de réseaux virtuels

La figure suivante montre les détails du peering de réseau virtuel Azure d’un réseau virtuel spoke. Pour savoir comment configurer le Peering de deux réseaux virtuels, consultez [Gérer le Peering de réseaux virtuels][VNet-Config]. Si vous voulez que le réseau virtuel spoke utilise les passerelles qui sont connectées au réseau virtuel hub, sélectionnez **Utiliser des passerelles distantes**.

[![1]][1]

La figure suivante montre les détails du peering du réseau virtuel hub. Si vous souhaitez que le réseau virtuel hub autorise le réseau virtuel spoke à utiliser les passerelles du hub, sélectionnez **Autoriser le transit de la passerelle**.

[![2]][2]

## <a name="branch-vnet-connectivity-by-using-a-site-to-site-vpn"></a>Connectivité de réseau virtuel branch à l’aide d’un VPN de site à site

Configurez la connectivité VPN de site à site entre les réseaux virtuels hub et branch à l’aide de passerelles VPN dans une passerelle VPN Azure. Par défaut, les passerelles VPN et les passerelles Azure ExpressRoute utilisent la valeur de numéro de système autonome (ASN) privée **65515**. Vous pouvez changer la valeur ASN dans la passerelle VPN. Dans l’initialisation (tearDown) de test, la valeur ASN de la passerelle VPN du réseau virtuel branch est remplacée par **65516** pour prendre en charge le routage eBGP entre les réseaux virtuels hub et branch.


[![3]][3]


## <a name="on-premises-location-1-connectivity-by-using-expressroute-and-a-site-to-site-vpn"></a>Connectivité de l’emplacement Location 1 local à l’aide d’ExpressRoute et d’un VPN de site à site

### <a name="expressroute-1-configuration-details"></a>Détails de la configuration d’ExpressRoute 1

La figure suivante montre la configuration du circuit ExpressRoute de région Azure 1 vers les routeurs de périphérie du client (CE) de l’emplacement Location 1 local :

[![4]][4]

La figure suivante montre la configuration de la connexion entre le circuit ExpressRoute 1 et le réseau virtuel hub :

[![5]][5]

La liste suivante montre la configuration de routeur CE principal pour la connectivité de peering privé ExpressRoute. (Les routeurs Cisco ASR1000 sont utilisés en tant que routeurs CE dans l’initialisation (tearDown) de test.) Quand les circuits ExpressRoute et le VPN de site à site sont configurés en parallèle pour connecter un réseau local à Azure, Azure choisit en priorité le circuit ExpressRoute par défaut. Pour éviter un routage asymétrique, le réseau local doit également donner la priorité à la connectivité ExpressRoute plutôt qu’à la connectivité VPN de site à site. La configuration suivante établit la hiérarchisation des priorités en utilisant l’attribut BGP **local-préférence** :

```config
interface TenGigabitEthernet0/0/0.300
 description Customer 30 private peering to Azure
 encapsulation dot1Q 30 second-dot1q 300
 ip vrf forwarding 30
 ip address 192.168.30.17 255.255.255.252
!
interface TenGigabitEthernet1/0/0.30
 description Customer 30 to south bound LAN switch
 encapsulation dot1Q 30
 ip vrf forwarding 30
 ip address 192.168.30.0 255.255.255.254
 ip ospf network point-to-point
!
router ospf 30 vrf 30
 router-id 10.2.30.253
 redistribute bgp 65021 subnets route-map BGP2OSPF
 network 192.168.30.0 0.0.0.1 area 0.0.0.0
default-information originate always
 default-metric 10
!
router bgp 65021
 !
 address-family ipv4 vrf 30
  network 10.2.30.0 mask 255.255.255.128
  neighbor 192.168.30.18 remote-as 12076
  neighbor 192.168.30.18 activate
  neighbor 192.168.30.18 next-hop-self
  neighbor 192.168.30.18 soft-reconfiguration inbound
  neighbor 192.168.30.18 route-map prefer-ER-over-VPN in
  neighbor 192.168.30.18 prefix-list Cust30_to_Private out
 exit-address-family
!
route-map prefer-ER-over-VPN permit 10
 set local-preference 200
!
ip prefix-list Cust30_to_Private seq 10 permit 10.2.30.0/25
!
```

### <a name="site-to-site-vpn-configuration-details"></a>Détails de la configuration VPN de site à site

La liste suivante montre la configuration du routeur CE principal pour la connectivité VPN de site à site :

```config
crypto ikev2 proposal Cust30-azure-proposal
 encryption aes-cbc-256 aes-cbc-128 3des
 integrity sha1
 group 2
!
crypto ikev2 policy Cust30-azure-policy
 match address local 66.198.12.106
 proposal Cust30-azure-proposal
!
crypto ikev2 keyring Cust30-azure-keyring
 peer azure
  address 52.168.162.84
  pre-shared-key local IamSecure123
  pre-shared-key remote IamSecure123
!
crypto ikev2 profile Cust30-azure-profile
 match identity remote address 52.168.162.84 255.255.255.255
 identity local address 66.198.12.106
 authentication local pre-share
 authentication remote pre-share
 keyring local Cust30-azure-keyring
!
crypto ipsec transform-set Cust30-azure-ipsec-proposal-set esp-aes 256 esp-sha-hmac
 mode tunnel
!
crypto ipsec profile Cust30-azure-ipsec-profile
 set transform-set Cust30-azure-ipsec-proposal-set
 set ikev2-profile Cust30-azure-profile
!
interface Loopback30
 ip address 66.198.12.106 255.255.255.255
!
interface Tunnel30
 ip vrf forwarding 30
 ip address 10.2.30.125 255.255.255.255
 tunnel source Loopback30
 tunnel mode ipsec ipv4
 tunnel destination 52.168.162.84
 tunnel protection ipsec profile Cust30-azure-ipsec-profile
!
router bgp 65021
 !
 address-family ipv4 vrf 30
  network 10.2.30.0 mask 255.255.255.128
  neighbor 10.10.30.254 remote-as 65515
  neighbor 10.10.30.254 ebgp-multihop 5
  neighbor 10.10.30.254 update-source Tunnel30
  neighbor 10.10.30.254 activate
  neighbor 10.10.30.254 soft-reconfiguration inbound
 exit-address-family
!
ip route vrf 30 10.10.30.254 255.255.255.255 Tunnel30
```

## <a name="on-premises-location-2-connectivity-by-using-expressroute"></a>Connectivité de l’emplacement Location 2 local à l’aide d’ExpressRoute

Un deuxième circuit ExpressRoute, très proche de l’emplacement Location 2 local, connecte l’emplacement Location 2 local au réseau virtuel hub. La figure suivante montre la deuxième configuration ExpressRoute :

[![6]][6]

La figure suivante montre la configuration de la connexion entre le deuxième circuit ExpressRoute et le réseau virtuel hub :

[![7]][7]

ExpressRoute 1 connecte le réseau virtuel hub et l’emplacement Location 1 local à un réseau virtuel distant dans une autre région Azure :

[![8]][8]

## <a name="expressroute-and-site-to-site-vpn-connectivity-in-tandem"></a>Connectivité ExpressRoute et VPN de site à site en tandem

###  <a name="site-to-site-vpn-over-expressroute"></a>VPN de site à site sur ExpressRoute

Vous pouvez configurer un VPN de site à site à l’aide du peering Microsoft ExpressRoute pour échanger des données de façon privée entre votre réseau local et vos réseaux virtuels Azure. Avec cette configuration, vous pouvez échanger des données en garantissant confidentialité, authenticité et intégrité. L’échange de données est également soumis à un système anti-relecture. Pour plus d’informations sur la configuration d’un VPN IPsec de site à site en mode tunnel via l’homologation Microsoft ExpressRoute, consultez l’article [Configurer un réseau VPN de site à site via le Peering Microsoft ExpressRoute][S2S-Over-ExR]. 

La principale limitation liée à la configuration d’un VPN de site à site qui utilise le peering Microsoft est le débit. Le débit sur le tunnel IPsec est limité par la capacité de la passerelle VPN. Le débit d’une passerelle VPN est inférieur au débit ExpressRoute. Dans ce scénario, le fait d’utiliser le tunnel IPsec pour un trafic très sécurisé et d’utiliser le peering privé pour toutes les autres catégories de trafic permet d’optimiser l’utilisation de la bande passante ExpressRoute.

### <a name="site-to-site-vpn-as-a-secure-failover-path-for-expressroute"></a>VPN de site à site en tant que chemin de basculement sécurisé pour ExpressRoute

ExpressRoute est utilisé comme une paire de circuits redondants pour garantir une haute disponibilité. Vous pouvez configurer une connectivité ExpressRoute géo-redondante dans différentes régions Azure. En outre, comme nous l’avons démontré dans notre initialisation (tearDown) de test, dans une région Azure, vous pouvez utiliser un VPN de site à site pour créer un chemin de basculement pour votre connectivité ExpressRoute. Quand les mêmes préfixes sont publiés sur ExpressRoute et un VPN de site à site, Azure choisit en priorité ExpressRoute. Pour éviter un routage asymétrique entre ExpressRoute et le VPN de site à site, la configuration réseau locale doit en faire autant en utilisant la connectivité ExpressRoute avant la connectivité VPN de site à site.

Pour plus d’informations sur la façon de configurer des connexions coexistantes pour ExpressRoute et un VPN de site à site, consultez [Coexistence de connexions ExpressRoute et de site à site][ExR-S2S-CoEx].

## <a name="extend-back-end-connectivity-to-spoke-vnets-and-branch-locations"></a>Étendre la connectivité de back-end à des réseaux virtuels spoke et à des emplacements branch

### <a name="spoke-vnet-connectivity-by-using-vnet-peering"></a>Connectivité de réseau virtuel spoke via le peering de réseaux virtuels

L’architecture des réseaux virtuels hub et spoke est largement utilisée. Le hub est un réseau virtuel dans Azure qui centralise la connectivité entre vos réseaux virtuels spoke et votre réseau local. Les spokes sont des réseaux virtuels appairés avec le hub et que vous pouvez utiliser pour isoler les charges de travail. Le trafic circule entre le centre de données local et le hub via une connexion ExpressRoute ou VPN. Pour plus d’informations sur l’architecture, consultez [Implémenter une topologie réseau hub-and-spoke dans Azure][Hub-n-Spoke].

Dans le peering de réseau virtuel dans une région, les réseaux virtuels spoke peuvent utiliser des passerelles de réseau virtuel hub (les passerelles VPN et ExpressRoute) pour communiquer avec des réseaux distants.

### <a name="branch-vnet-connectivity-by-using-site-to-site-vpn"></a>Connectivité de réseau virtuel branch à l’aide d’un VPN de site à site

Vous pouvez souhaiter que les réseaux virtuels branch, qui se trouvent dans différentes régions, et les réseaux locaux communiquent entre eux via un réseau virtuel hub. La solution Azure native pour cette configuration est la connectivité VPN de site à site à l’aide d’un VPN. Une autre solution consiste à utiliser une appliance virtuelle réseau (NVA) pour le routage dans le hub.

Pour plus d’informations, consultez [Qu’est-ce qu’une passerelle VPN ?][VPN] et [Déployer une appliance virtuelle réseau hautement disponible][Deploy-NVA].

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur l’[analyse du plan de contrôle][Control-Analysis] de l’initialisation (tearDown) de test et les vues de différents réseaux virtuels ou réseaux locaux virtuels (VLAN) dans la topologie.

Découvrez l’[analyse du plan de données][Data-Analysis] de l’initialisation (tearDown) de test et les affichages des fonctionnalités de supervision de réseau Azure.

Consultez le [FAQ ExpressRoute][ExR-FAQ] pour :
-   Connaître le nombre de circuits ExpressRoute que vous pouvez connecter à une passerelle ExpressRoute.
-   Connaître le nombre de passerelles ExpressRoute que vous pouvez connecter à un circuit ExpressRoute.
-   Découvrir les autres limites de mise à l’échelle d’ExpressRoute.


<!--Image References-->
[1]: ./media/backend-interoperability/SpokeVNet_peering.png "Peering du réseau virtuel spoke"
[2]: ./media/backend-interoperability/HubVNet-peering.png "Peering du réseau virtuel hub"
[3]: ./media/backend-interoperability/BranchVNet-VPNGW.png "Configuration de passerelle VPN d’un réseau virtuel branch"
[4]: ./media/backend-interoperability/ExR1.png "Configuration d’ExpressRoute 1"
[5]: ./media/backend-interoperability/ExR1-Hub-Connection.png "Configuration de la connexion d’ExpressRoute 1 à la passerelle ExR de réseau virtuel hub"
[6]: ./media/backend-interoperability/ExR2.png "Configuration d’ExpressRoute 2"
[7]: ./media/backend-interoperability/ExR2-Hub-Connection.png "Configuration de la connexion d’ExpressRoute 2 à la passerelle ExR de réseau virtuel hub"
[8]: ./media/backend-interoperability/ExR2-Remote-Connection.png "Configuration de la connexion d’ExpressRoute 2 à la passerelle ExR de réseau virtuel distant"

<!--Link References-->
[Setup]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-preface
[ExpressRoute]: https://docs.microsoft.com/azure/expressroute/expressroute-introduction
[VPN]: https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways
[VNet]: https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-portal
[Configuration]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-configuration
[Control-Analysis]:https://docs.microsoft.com/azure/networking/connectivty-interoperability-control-plane
[Data-Analysis]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-data-plane
[ExR-FAQ]: https://docs.microsoft.com/azure/expressroute/expressroute-faqs
[S2S-Over-ExR]: https://docs.microsoft.com/azure/expressroute/site-to-site-vpn-over-microsoft-peering
[ExR-S2S-CoEx]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-coexist-resource-manager
[Hub-n-Spoke]: https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke
[Deploy-NVA]: https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha
[VNet-Config]: https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-peering


