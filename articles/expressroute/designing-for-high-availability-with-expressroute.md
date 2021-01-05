---
title: 'Azure ExpressRoute : Conception pour la haute disponibilité'
description: Cette page fournit des conseils sur l’architecture pour la haute disponibilité lors de l’utilisation d’Azure ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 06/28/2019
ms.author: duau
ms.openlocfilehash: 3602c3944e8731263fbb55f024c276783950329f
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92202359"
---
# <a name="designing-for-high-availability-with-expressroute"></a>Conception pour une haute disponibilité avec ExpressRoute

ExpressRoute est conçu pour la haute disponibilité afin de fournir à l’opérateur une connectivité de réseau privé de qualité aux ressources Microsoft. En d’autres termes, il n’existe aucun point de défaillance unique dans le chemin d’accès ExpressRoute au sein du réseau de Microsoft. Pour optimiser la disponibilité, le client et le segment de fournisseur de service de votre circuit ExpressRoute doivent également être conçus pour la haute disponibilité. Dans cet article, nous allons tout d’abord passer en revue les considérations relatives à l’architecture réseau pour la conception d’une connectivité réseau robuste à l’aide d’ExpressRoute. Nous analyserons ensuite les fonctionnalités qui vous aident à améliorer la haute disponibilité de votre circuit ExpressRoute.

>[!NOTE]
>Les concepts décrits dans cet article s’appliquent tout autant lorsqu’un circuit ExpressRoute est créé sous Virtual WAN ou à l’extérieur de celui-ci.
>

## <a name="architecture-considerations"></a>Considérations relatives à l’architecture

Le schéma suivant montre la méthode de connexion recommandée à l’aide d’un circuit ExpressRoute afin d’optimiser la disponibilité d’un circuit ExpressRoute.

 [![1]][1]

Pour une haute disponibilité, il est primordial de maintenir la redondance du circuit ExpressRoute dans le réseau de bout en bout. En d’autres termes, vous devez conserver la redondance au sein de votre réseau local et ne devez pas la compromettre au sein de votre réseau de fournisseur de service. Pour conserver la redondance à son minimum, il faut éviter des points de défaillance réseau uniques. Vous pouvez davantage améliorer la haute disponibilité en disposant d’une alimentation et d’un refroidissement redondants pour vos appareils réseau.

### <a name="first-mile-physical-layer-design-considerations"></a>Considérations relatives à la conception de couche physique pour le premier mile

 Si vous arrêtez les connexions primaires et secondaires d’un circuit ExpressRoute sur le même Customer Premises Equipment (CPE), vous compromettez la haute disponibilité au sein de votre réseau local. De plus, si vous configurez les connexions primaires et secondaires via le même port qu’un CPE (en arrêtant les deux connexions sous différentes interfaces ou en les fusionnant au sein d’un réseau partenaire), vous forcez le partenaire à compromettre la haute disponibilité sur leur segment réseau également. C’est ce qu’illustre le diagramme suivant.

[![2]][2]

En revanche, si vous arrêtez les connexions primaires et secondaires d’un circuit ExpressRoute dans différents emplacements géographiques, il se peut que vous compromettiez les performances réseau de la connectivité. Si le trafic est activement équilibré entre les connexions primaires et secondaires arrêtées dans différents emplacements géographiques, une différence potentielle considérable dans la latence du réseau entre les deux chemins entraînerait des sous-performances réseau. 

Pour des considérations relatives à la conception de la géoredondance, consultez [Conception pour une reprise d’activité après sinistre][DR].

### <a name="active-active-connections"></a>Connexions entre deux passerelles actives

Le réseau Microsoft est configuré pour opérer les connexions primaires et secondaires des circuits ExpressRoute en mode actif/actif. Toutefois, via vos annonces d’itinéraire, vous pouvez forcer les connexions redondantes d’un circuit ExpressRoute à opérer en mode actif/passif. L’annonce d’itinéraires plus spécifiques et l’ajout du chemin du système autonome BGP sont des techniques courantes utilisées pour préférer un chemin à un autre.

Pour améliorer la haute disponibilité, il est recommandé d’opérer les deux connexions d’un circuit ExpressRoute en mode actif/actif. Si vous autorisez les connexions à opérer en mode actif/actif, le réseau Microsoft équilibre les charges du trafic sur les connexions selon le flux.

L’exécution des connexions primaires et secondaires d’un circuit ExpressRoute en mode actif/passif risque de causer l’échec des deux connexions, après un autre échec dans le chemin actif. Les causes courantes d’échec sur le basculement sont le manque de gestion active de la connexion passive et les itinéraires obsolètes d’annonces de connexions passives.

Aussi, l’exécution des connexions primaires et secondaires d’un circuit ExpressRoute en mode actif/actif entraîne l’échec et le reroutage d’environ la moitié des flux, après l’échec de la connexion ExpressRoute. Ainsi, le mode actif/actif améliorera grandement le MTTR (temps moyen de récupération).

### <a name="nat-for-microsoft-peering"></a>NAT pour le peering Microsoft 

Le peering Microsoft est conçue pour la communication entre les points de terminaison publics. Souvent, les points de terminaison privés locaux deviennent des NAT avec une IP publique sur le réseau du client ou du partenaire avant qu’ils ne communiquent via le peering Microsoft. En supposant que vous utilisez tous deux les connexions primaires et secondaires en mode actif/actif, où et comment vous utilisez le NAT affecte votre temps de récupération après la défaillance d’une des connexions ExpressRoute. Deux options NAT sont illustrées dans la figure suivante :

[![3]][3]

Dans l’option 1, le NAT est appliqué après avoir fractionné le trafic entre les connexions primaires et secondaires du circuit ExpressRoute. Pour répondre aux exigences avec état du NAT, des pools NAT indépendants sont utilisés entre les appareils principaux et secondaires, de sorte que le trafic renvoyé arrive dans le même appareil edge par lequel sort le flux.

Dans l’option 2, un pool NAT commun est utilisé avant de fractionner le trafic entre les connexions primaires et secondaires du circuit ExpressRoute. Il est important de bien comprendre que le pool NAT commun avant fractionnement du trafic n’induit pas l’introduction d’un point de défaillance unique et donc le risque de compromettre la haute disponibilité.

Avec l’option 1, après un échec de connexion ExpressRoute, la capacité à atteindre le pool NAT correspondant est rompue. Par conséquent, tous les flux rompus doivent être rétablis par TCP ou par couche d’application après l’expiration du délai de la fenêtre correspondante. Si l’un des pools NAT est utilisé pour le serveur frontal d’un serveur local et si la connectivité correspondante cesse de fonctionner, les serveurs sur site ne sont pas accessibles à partir d’Azure tant que la connectivité n’est pas résolue.

Tandis qu’avec l’option 2, le NAT est accessible même après l’échec de la connexion primaire ou secondaire. Ainsi, la couche réseau elle-même peut rediriger les paquets et permettre un temps de récupération plus court après l’échec. 

> [!NOTE]
> Si vous utilisez l’option NAT 1 (des pools NAT indépendants pour les connexions ExpressRoute primaires et secondaires), et que vous mappez un port d’une adresse IP d’un des pools NAT à un serveur local, le serveur n’est plus accessible via le circuit ExpressRoute en cas d’échec de la connexion correspondante.
> 

## <a name="fine-tuning-features-for-private-peering"></a>Réglage des fonctionnalités du peering privé

Dans cette section, nous allons passer en revue des fonctionnalités facultatives (selon votre déploiement Azure et votre susceptibilité au temps de récupération moyen) qui visent à améliorer la haute disponibilité de votre circuit ExpressRoute. Plus précisément, nous allons voir un déploiement de passerelles de réseau virtuel ExpressRoute tenant compte de la zone, et la détection d’envoi bidirectionnel (BFD).

### <a name="availability-zone-aware-expressroute-virtual-network-gateways"></a>Passerelles de réseau virtuel ExpressRoute tenant compte de la zone de disponibilité

Une zone de disponibilité dans une région Azure est une combinaison d’un domaine d’erreur et d’un domaine de mise à jour. Si vous optez pour le déploiement Azure IaaS redondant interzone, il se peut que vous vouliez configurer des passerelles de réseau virtuel redondantes interzone qui termine le peering privé ExpressRoute. Pour en apprendre plus, consultez [À propos des passerelles de réseau virtuel redondantes interzone dans les zones de disponibilité Azure][zone redundant vgw]. Pour configurer une passerelle de réseau virtuel redondante interzone, consultez [Créer une passerelle de réseau virtuel redondante interzone dans les zones de disponibilité Azure][conf zone redundant vgw].

### <a name="improving-failure-detection-time"></a>Amélioration du temps de détection des défaillances

ExpressRoute prend en charge la détection d’envoi bidirectionnel via le peering privé. La détection d’envoi bidirectionnel réduit le temps de détection des défaillances sur le réseau de couche 2 entre Microsoft Enterprise Edge (MSEE) et leurs voisins BGP en local de 3 minutes environ à moins d’une seconde. Un temps de détection des défaillances plus rapide aide à accélérer la récupération. Pour en savoir plus, consultez [Configurer la détection d’envoi bidirectionnel sur ExpressRoute][BFD].

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons abordé la conception pour la haute disponibilité d’une connectivité de circuit ExpressRoute. Un point de peering du circuit ExpressRoute est épinglé à un emplacement géographique et par conséquent, peut être affecté par une défaillance grave affectant l’emplacement complet. 

Pour des considérations relatives à la conception de connectivité de réseau géoredondante de la dorsale principale de Microsoft qui peut résister à des défaillances catastrophiques, impactant une région entière, consultez [Conception d’une récupération après sinistre avec le peering privé ExpressRoute][DR].

<!--Image References-->
[1]: ./media/designing-for-high-availability-with-expressroute/exr-reco.png "Méthode recommandée pour se connecter via ExpressRoute"
[2]: ./media/designing-for-high-availability-with-expressroute/suboptimal-lastmile-connectivity.png "Connectivité inefficace du dernier mile"
[3]: ./media/designing-for-high-availability-with-expressroute/nat-options.png "Options NAT"


<!--Link References-->
[zone redundant vgw]: ../vpn-gateway/about-zone-redundant-vnet-gateways.md
[conf zone redundant vgw]: ../vpn-gateway/create-zone-redundant-vnet-gateway.md
[Configure Global Reach]: ./expressroute-howto-set-global-reach.md
[BFD]: ./expressroute-bfd.md
[DR]: ./designing-for-disaster-recovery-with-expressroute-privatepeering.md