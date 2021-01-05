---
title: Fichier Include
description: Fichier Include
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 03/19/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 317a480c13c5c6e00653fd61878a379df3f65ac4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67176799"
---
### <a name="what-is-expressroute-global-reach"></a>Présentation d’ExpressRoute Global Reach

ExpressRoute Global Reach est un service Azure qui connecte vos réseaux locaux via le service ExpressRoute à travers du réseau global de Microsoft. Par exemple, si vous disposez d’un centre de données privé en Californie, connecté à ExpressRoute dans la Silicon Valley, et d’un autre centre de données privé au Texas, connecté à ExpressRoute à Dallas, ExpressRoute Global Reach vous permet de connecter vos différents centres de données privés par le biais de deux circuits ExpressRoute, et le trafic entre vos centres de données traversera le réseau principal de Microsoft.

### <a name="how-do-i-enable-or-disable-expressroute-global-reach"></a>Comment faire pour activer ou désactiver ExpressRoute Global Reach ?

Vous activez ExpressRoute Global Reach en connectant vos circuits ExpressRoute ensemble. Vous désactivez la fonctionnalité en déconnectant les circuits. Reportez-vous à la [configuration](../articles/expressroute/expressroute-howto-set-global-reach.md).

### <a name="do-i-need-expressroute-premium-for-expressroute-global-reach"></a>ExpressRoute Premium est-il nécessaire pour ExpressRoute Global Reach ?

Si vos circuits ExpressRoute se trouvent dans la même région géopolitique, vous n’avez pas besoin d’ExpressRoute Premium pour les connecter entre eux. Si deux circuits ExpressRoute sont dans des régions géopolitiques différentes, vous avez besoin d’ExpressRoute Premium pour les deux circuits, afin de pouvoir les connecter entre eux. 

### <a name="how-will-i-be-charged-for-expressroute-global-reach"></a>Comment est facturée la fonctionnalité ExpressRoute Global Reach ?

ExpressRoute permet d’établir une connectivité entre votre réseau local et des services de cloud computing Microsoft. ExpressRoute Global Reach permet d’établir une connectivité entre vos réseaux locaux via vos circuits ExpressRoute existants, en utilisant le réseau global de Microsoft. La fonctionnalité ExpressRoute Global Reach est facturée séparément du service ExpressRoute existant. Des frais de module complémentaire sont associés à l’activation de cette fonctionnalité sur chaque circuit ExpressRoute. Le trafic entre vos réseaux locaux activés par ExpressRoute Global Reach est facturé selon un tarif de sortie au niveau de la source et selon un tarif d’entrée au niveau de la destination. Les tarifs sont basés sur la zone dans laquelle se trouvent les circuits.

### <a name="where-is-expressroute-global-reach-supported"></a>Dans quelles régions la fonctionnalité ExpressRoute Global Reach est-elle prise en charge ?

ExpressRoute Global Reach est pris en charge dans [des pays/régions ou lieux sélectionnés](../articles/expressroute/expressroute-global-reach.md). Les circuits ExpressRoute doivent être créés sur les emplacements de Peering dans ces pays/régions ou lieux.

### <a name="i-have-more-than-two-on-premises-networks-each-connected-to-an-expressroute-circuit-can-i-enable-expressroute-global-reach-to-connect-all-of-my-on-premises-networks-together"></a>J’ai plus de deux réseaux locaux, chacun connecté à un circuit ExpressRoute. Puis-je activer ExpressRoute Global Reach pour connecter tous mes réseaux locaux ensemble ?

Oui, sous réserve que les circuits se trouvent dans les pays/régions pris en charge. Vous devez connecter deux circuits ExpressRoute à la fois. Pour créer un réseau entièrement maillé, vous devez énumérer toutes les paires de circuit et répéter la configuration. 

### <a name="can-i-enable-expressroute-global-reach-between-two-expressroute-circuits-at-the-same-peering-location"></a>Puis-je activer la fonctionnalité ExpressRoute Global Reach entre deux circuits ExpressRoute se trouvant au même emplacement de peering ?

Non. Les deux circuits doivent provenir d’emplacements de peering différents. Si un métro dans un pays/une région pris en charge a plusieurs emplacements de peering ExpressRoute, vous pouvez connecter ensemble les circuits ExpressRoute créés à différents emplacements de peering dans ce métro. 

### <a name="if-expressroute-global-reach-is-enabled-between-circuit-x-and-circuit-y-and-between-circuit-y-and-circuit-z-will-my-on-premises-networks-connected-to-circuit-x-and-circuit-z-talk-to-each-other-via-microsofts-network"></a>Si la fonctionnalité ExpressRoute Global Reach est activée entre le circuit X et le circuit Y, mes réseaux locaux connectés au circuit X et au circuit Z communiquent-ils entre eux via le réseau de Microsoft ?

Non. Pour activer la connectivité entre deux de vos réseaux locaux, vous devez connecter explicitement les circuits ExpressRoute correspondants. Dans l’exemple ci-dessus, vous devez connecter le circuit X et le circuit Z. 

### <a name="what-is-the-network-throughput-i-can-expect-between-my-on-premises-networks-after-i-enable-expressroute-global-reach"></a>À quel débit réseau puis-je m’attendre entre mes réseaux locaux après avoir activé la fonctionnalité ExpressRoute Global Reach ?

Le débit réseau entre vos réseaux locaux, activés par ExpressRoute Global Reach, est limité par le plus petit des deux circuits ExpressRoute. Le trafic local vers Azure et le trafic local à local partagent le même circuit et sont soumis au même plafond de bande passante. 

### <a name="with-expressroute-global-reach-what-are-the-limits-on-the-number-of-routes-i-can-advertise-and-the-number-of-routes-i-will-receive"></a>Avec ExpressRoute Global Reach, quelles sont les limites du nombre de routes que je peux publier et du nombre de routes que je recevrai ?

Le nombre de routes que vous pouvez publier à Microsoft sur le peering privé Azure reste 4 000 sur un circuit Standard ou 10 000 sur un circuit Premium. Le nombre de routes que vous recevrez de Microsoft sur le peering privé Azure sera la somme des routes de vos réseaux virtuels Azure et des routes de vos autres réseaux locaux connectés par le biais d’ExpressRoute Global Reach. Veillez à définir une limite de préfixe maximale appropriée sur votre routeur local. 

### <a name="what-is-the-sla-for-expressroute-global-reach"></a>Quel contrat SLA s’applique à ExpressRoute Global Reach ?

ExpressRoute Global Reach fournit le même [contrat SLA de disponibilité](https://azure.microsoft.com/support/legal/sla/expressroute/v1_3/) que le service ExpressRoute standard.
