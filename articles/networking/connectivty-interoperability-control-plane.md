---
title: 'Interopérabilité dans Azure : Analyse du plan de contrôle'
description: Cet article fournit l’analyse du plan de contrôle de la configuration de test que vous pouvez utiliser pour analyser l’interopérabilité entre ExpressRoute, un réseau virtuel de site à site et le peering de réseau virtuel dans Azure.
documentationcenter: na
services: networking
author: rambk
manager: tracsman
ms.service: virtual-network
ms.topic: article
ms.workload: infrastructure-services
ms.date: 10/18/2018
ms.author: rambala
ms.openlocfilehash: 5e41bc86533815c394077bf5276d930fe958cd19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80518274"
---
# <a name="interoperability-in-azure--control-plane-analysis"></a>Interopérabilité dans Azure : Analyse du plan de contrôle

Cet article décrit l’analyse du plan de contrôle de [l’initialisation (tearDown) de test][Setup]. Vous pouvez également consulter la [configuration de l’initialisation (tearDown) de test][Configuration] et [l’analyse du plan de données][Data-Analysis] de l’initialisation (tearDown) de test.

L’analyse du plan de contrôle examine essentiellement les itinéraires qui sont échangés entre des réseaux au sein d’une topologie. L’analyse du plan de contrôle vous permet de comprendre comment des réseaux différents affichent la topologie.

## <a name="hub-and-spoke-vnet-perspective"></a>Perspective du réseau virtuel hub and spoke

La figure suivante illustre le réseau du point de vue d’un réseau virtuel hub (VNet) et d’un réseau virtuel spoke (en bleu). Cette figure montre également le numéro de système autonome (ASN) de différents réseaux et les itinéraires qui sont échangés entre réseaux différents : 

![1][1]

L’ASN de la passerelle Azure ExpressRoute du réseau virtuel est différent de l’ASN des routeurs Microsoft Enterprise Edge (MSEE). Une passerelle ExpressRoute utilise un ASN privé (valeur de **65515**) et les MSEE utilisent un ASN public (valeur de **12076**) dans le monde entier. Lorsque vous configurez le peering ExpressRoute, étant donné que le MSEE est l’homologue, vous utilisez l’ASN d’homologue **12076**. Du côté Azure, MSEE établit le peering eBGP la passerelle ExpressRoute. Le double peering eBGP que MSEE établit pour chaque peering ExpressRoute est transparent au niveau du plan de contrôle. Par conséquent, lorsque vous visualisez une table de routage ExpressRoute, l’ASN de la passerelle ExpressRoute du réseau virtuel s’affiche pour les préfixes du réseau virtuel. 

La figure suivante montre un exemple de table de routage ExpressRoute : 

![5][5]

Dans Azure, l’ASN n’est significatif que de la perspective du peering. Par défaut, l’ASN de la passerelle ExpressRoute et de la passerelle du réseau privé virtuel est **65515** dans la passerelle VPN Azure.

## <a name="on-premises-location-1-and-the-remote-vnet-perspective-via-expressroute-1"></a>Perspective du réseau local Location 1 et du réseau virtuel distant via ExpressRoute 1

Le réseau local Location 1 et du réseau virtuel distant sont connectés au réseau virtuel du hub via ExpressRoute 1. Ils partagent la même perspective de la topologie, comme le montre le schéma suivant :

![2][2]

## <a name="on-premises-location-1-and-the-branch-vnet-perspective-via-a-site-to-site-vpn"></a>Perspective du réseau local Location 1 et du réseau virtuel branche via le VPN de site à site

Le réseau local Location 1 et le réseau virtuel branche sont tous deux connectés à la passerelle VPN du réseau virtuel Hub via les connexions VPN de site à site. Ils partagent la même perspective de la topologie, comme le montre le schéma suivant :

![3][3]

## <a name="on-premises-location-2-perspective"></a>Perspective du réseau local Location 2

Le réseau local Location 2 est connecté au réseau local hub via le peering privé d’ExpressRoute 2 : 

![4][4]

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

Découvrez l’[analyse du plan de données][Data-Analysis] de l’initialisation (tearDown) de test et les affichages des fonctionnalités de supervision de réseau Azure.

Consultez le [FAQ ExpressRoute][ExR-FAQ] pour :
-   Connaître le nombre de circuits ExpressRoute que vous pouvez connecter à une passerelle ExpressRoute.
-   Connaître le nombre de passerelles ExpressRoute que vous pouvez connecter à un circuit ExpressRoute.
-   Découvrir les autres limites de mise à l’échelle d’ExpressRoute.


<!--Image References-->
[1]: ./media/backend-interoperability/HubView.png "Perspective du réseau virtuel Hub and Spoke de la topologie"
[2]: ./media/backend-interoperability/Loc1ExRView.png "Perspective du réseau virtuel Location 1 et distant de la topologie via ExpressRoute 1"
[3]: ./media/backend-interoperability/Loc1VPNView.png "Perspective du réseau virtuel Location 1 et de branche de la topologie via un VPN site à site"
[4]: ./media/backend-interoperability/Loc2View.png "Perspective du réseau Location 2 de la topologie"
[5]: ./media/backend-interoperability/ExR1-RouteTable.png "Table de route ExpressRoute 1"

<!--Link References-->
[Setup]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-preface
[Configuration]: https://docs.microsoft.com/azure/networking/connectivty-interoperability-configuration
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


