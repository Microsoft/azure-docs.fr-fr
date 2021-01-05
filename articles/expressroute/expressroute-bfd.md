---
title: 'Azure ExpressRoute : Configurer BFD'
description: Cet article fournit des instructions sur la configuration de BFD (Bidirectional Forwarding Detection) via un appairage privé d’un circuit ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 12/14/2020
ms.author: duau
ms.openlocfilehash: 254f5909e7ed8db4dc18ade2677a3213b268cf41
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511261"
---
# <a name="configure-bfd-over-expressroute"></a>Configurer BFD sur ExpressRoute

ExpressRoute prend en charge la détection de transfert bidirectionnel (BFD, Bidirectional Forwarding Detection) sur le peering privé et le peering Microsoft. Lorsque vous activez BFD sur ExpressRoute, vous pouvez accélérer la détection d’échec de liaison entre les appareils Microsoft Enterprise Edge (MSEE) et les routeurs que votre circuit ExpressRoute configure (CE/PE). Vous pouvez configurer ExpressRoute sur vos appareils de routage en périphérie ou des appareils de routage en périphérie de partenaire (si vous avez utilisé un service de connexion de couche 3 managé). Ce document explique pourquoi BFD peut être nécessaire et comment activer BFD sur ExpressRoute.

## <a name="need-for-bfd"></a>Besoin de BFD

Le diagramme suivant présente les avantages de l’activation de BFD sur le circuit ExpressRoute : [![1]][1]

Vous pouvez activer le circuit ExpressRoute par des connexions de couche 2 ou par des connexions de couche 3 managées. Dans les deux cas, s’il existe plusieurs appareils de couche 2 dans le chemin de connexion ExpressRoute, la responsabilité de la détection des échecs des liaisons dans le chemin incombe à la session BGP superposée.

Sur les appareils MSEE, les durées de conservation et de suspension de BGP sont généralement configurées respectivement sur 60 et 180 secondes. Par conséquent, après un échec de liaison, jusqu’à trois minutes peuvent être nécessaires pour détecter un échec et basculer le trafic vers une autre connexion.

Vous pouvez contrôler les minuteurs du protocole BGP en configurant des durées inférieures de conservation et de suspension du protocole BGP sur votre appareil de peering en périphérie. Si les minuteurs du protocole BGP ne sont pas les mêmes entre les deux appareils de peering, la session BGP s’établit en utilisant la valeur de temps inférieure. La durée de conservation du protocole BGP peut être définie sur une valeur aussi basse que trois secondes, et la durée de suspension sur un valeur aussi basse que 10 secondes. Toutefois, la définition d’un minuteur de protocole BGP très agressif n’est pas recommandée, car le protocole est exigeant en processus.

Dans ce scénario, BFD peut être bénéfique. BFD assure la détection de la défaillance des liaisons en consommant peu de ressources et dans un intervalle de temps inférieur à la seconde. 


## <a name="enabling-bfd"></a>Activation de BFD

BFD est configuré par défaut sous toutes les interfaces de peering privé ExpressRoute nouvellement créées sur les appareils MSEE. Par conséquent, pour activer BFD, il vous suffit de configurer BFD sur vos appareils principaux et secondaires. La configuration de BFD est un processus en deux étapes. Vous configurez BFD sur l’interface, puis liez celle-ci à la session BGP.

Voici un exemple de configuration de CE/PE (avec Cisco IOS XE). 

```console
interface TenGigabitEthernet2/0/0.150
   description private peering to Azure
   encapsulation dot1Q 15 second-dot1q 150
   ip vrf forwarding 15
   ip address 192.168.15.17 255.255.255.252
   bfd interval 300 min_rx 300 multiplier 3


router bgp 65020
   address-family ipv4 vrf 15
      network 10.1.15.0 mask 255.255.255.128
      neighbor 192.168.15.18 remote-as 12076
      neighbor 192.168.15.18 fall-over bfd
      neighbor 192.168.15.18 activate
      neighbor 192.168.15.18 soft-reconfiguration inbound
   exit-address-family
```

>[!NOTE]
>Pour activer BFD sous un peering privé déjà existant, vous devez réinitialiser le peering. Consultez [Réinitialiser les peerings ExpressRoute][ResetPeering]
>

## <a name="bfd-timer-negotiation"></a>Négociation des minuteurs BFD

Entre les pairs BFD, le plus lent des deux pairs détermine la vitesse de transmission. Les intervalles de transmission/réception BFD des appareils MSEE sont définis sur 300 millisecondes. Dans certains scénarios, l'intervalle peut être défini sur une valeur supérieure de 750 millisecondes. En configurant une valeur plus élevée, vous pouvez forcer des intervalles plus longs, mais il n’est pas possible de les raccourcir.

>[!NOTE]
>Si vous avez configuré des circuits ExpressRoute géoredondants, utilisez une connectivité VPN IPSec de site à site en tant que solution de secours. L’activation de BFD permettrait un basculement plus rapide suite à une défaillance de connectivité ExpressRoute. 
>

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations ou d'aide, consultez les liens suivants :

- [Création et modification d’un circuit ExpressRoute][CreateCircuit]
- [Créer et modifier le routage pour un circuit ExpressRoute][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-bfd/BFD_Need.png "BFD accélère les temps de détection des défaillances des liaisons"

<!--Link References-->
[CreateCircuit]: ./expressroute-howto-circuit-portal-resource-manager.md
[CreatePeering]: ./expressroute-howto-routing-portal-resource-manager.md
[ResetPeering]: ./expressroute-howto-reset-peering.md