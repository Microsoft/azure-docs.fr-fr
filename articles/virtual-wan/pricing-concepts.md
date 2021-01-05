---
title: À propos de la tarification Virtual WAN
titleSuffix: Azure Virtual WAN
description: Cet article décrit les questions courantes sur la tarification de Virtual WAN
services: virtual-wan
author: reyandap
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 12/08/2020
ms.author: reyandap
ms.custom: references_pricing
ms.openlocfilehash: 8ced9bb33be341d35904967092414676a6ffe3ac
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905534"
---
# <a name="about-virtual-wan-pricing"></a>À propos de la tarification Virtual WAN

Azure Virtual WAN propose plusieurs services de réseau et de sécurité réunis dans une infrastructure unifiée. Elle se base sur une architecture hub-and-spoke, où les hubs sont gérés par Microsoft avec différents services fournis dans le hub, tels que le VPN, ExpressRoute, le VPN utilisateur (point à site), le pare-feu, le routage, etc.

Chaque service dans Virtual WAN est facturé. Par conséquent, il n’est pas possible d’appliquer un prix unique à Virtual WAN. La [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/) propose un mécanisme permettant de déduire le coût, basé sur les services approvisionnés dans un réseau Virtual WAN. Cet article aborde les questions fréquemment posées sur la tarification des réseaux Virtual WAN.

>[!NOTE]
>Pour connaître les informations de tarification actuelles, consultez la page [Tarification de Virtual WAN](https://azure.microsoft.com/pricing/details/virtual-wan/).
>

## <a name="common-pricing-questions"></a><a name="questions"></a>Questions courantes sur la tarification

### <a name="what-is-a-scale-unit"></a><a name="scale-unit"></a>Qu’est-ce qu’une unité d’échelle ?

Une **unité d’échelle** est une unité de capacité agrégée de site à site (S2S), de point à site (P2S) et ExpressRoute (ER) dans un hub virtuel. Par exemple :

* **1 unité d’échelle VPN S2S** implique une capacité totale de la passerelle VPN de 500 Mbits/s (des instances doubles sont déployées pour des raisons de résilience) dans un hub virtuel selon un coût horaire de 0,361 $.
* **1 unité d’échelle ER** implique un total de 2 Gbit/s de passerelle ER dans un hub virtuel, pour un coût de 0,42 $/h.
* **5 unités d’échelle ER** impliquent un total de 10 Gbit/s de passerelle ER au sein d’un réseau virtuel de hub virtuel facturé à 0,42*5 $/h. Paliers ER de 0,25 $/h de 6 à 10 unités d’échelle.

### <a name="what-is-a-connection-unit"></a><a name="connection-unit">Qu’est-ce qu’une unité de connexion ?</a>

Une **unité de connexion** s’applique à tout point de terminaison local/non-Microsoft se connectant aux passerelles Azure. Pour un VPN de site à site, cette valeur implique des branches. Pour un VPN utilisateur (point à site), cette valeur implique des utilisateurs distants. Pour ExpressRoute, cette valeur implique des connexions de circuit ExpressRoute.<br>Par exemple :

* Une connexion de branche se connectant à Azure VPN dans un hub virtuel coûte de 0,05 $/h. Par conséquent, 100 connexions de branche se connectant à un hub virtuel Azure coûtent 0,05 $*100/h.

* Deux connexions de circuit ExpressRoute qui se connectent à un hub virtuel coûtent 0,05 $*2/h.

* Trois connexions utilisateur distantes se connectant à la passerelle P2S Azure Virtual Hub coûtent 0,03 $*3/h.

### <a name="how-are-data-transfer-charges-calculated"></a><a name="data-transfer"></a>Comment les frais de transfert de données sont-ils calculés ?

* Le trafic entrant dans Azure n’est pas facturé. Le trafic qui quitte Azure (via VPN, ExpressRoute ou les connexions VPN utilisateur de point à site) est soumis aux [frais de transfert de données Azure](https://azure.microsoft.com/pricing/details/bandwidth/) standard.

* Pour les frais de transfert de données entre un hub Virtual WAN et un hub Virtual WAN distant ou un réseau virtuel dans une région différente de celle du hub source, les frais de transfert de données s’appliquent au trafic quittant le hub. Exemple : Le trafic qui part d’un hub de la région USA Est est facturé 0,02 $/Go vers un hub USA Ouest. Il n’y a pas de frais pour le trafic entrant dans le hub USA Ouest. Tout le trafic de hub à hub est soumis à des frais inter-régions (intra/Inter-continentaux) [Frais de transfert de données Azure](https://azure.microsoft.com/pricing/details/bandwidth/). 

### <a name="what-is-the-difference-between-a-standard-hub-fee-and-a-standard-hub-processing-fee"></a><a name="fee"></a>Quelle est la différence entre un tarif hub standard et les frais de traitement de hub standard ?

Il existe deux types de réseaux Virtual WAN :

* Un **Virtual WAN de base**, dans lequel les utilisateurs peuvent déployer plusieurs hubs et bénéficier d’une connectivité VPN de site à site. Le Virtual WAN de base ne dispose pas de fonctionnalités avancées telles que les hubs entièrement maillés, la connectivité ExpressRoute, la connectivité VPN utilisateur/point à site, la connectivité de réseau virtuel à réseau virtuel et la connectivité de transit ExpressRoute ou le pare-feu Azure, etc. Aucun frais de base ou frais de traitement des données n’est facturé pour les hubs dans un Virtual WAN de base.

* Le **Virtual WAN standard** propose des fonctionnalités avancées telles que les hubs entièrement maillés, la connectivité ExpressRoute, la connectivité VPN utilisateur/point à site, la connectivité de réseau virtuel à réseau virtuel et la connectivité de transit ExpressRoute, le pare-feu Azure, etc. Tout le routage du hub virtuel est fourni par un routeur qui active plusieurs services dans un hub virtuel. Il y a des frais de base pour le hub, facturé à 0,25 $/h. Il y a également un coût pour le traitement des données dans le routeur de hub virtuel pour la connectivité de transit de réseau virtuel à réseau virtuel. Voir la figure suivante.

 Dans l’**exemple** ci-dessous, il y a deux réseaux virtuels, le réseau virtuel 1 et le réseau virtuel 2. Supposons que des données sont envoyées à 1 GBit/s à partir d’une machine virtuelle du réseau virtuel 1 vers une autre machine virtuelle dans le réseau virtuel 2. Les frais suivants s’appliquent :

* Frais de base du hub virtuel de 0,25 $/h

* Frais de traitement de données du hub virtuel de 0,02 $/Go pour 1 Gbit/s

**Exemple**

   :::image type="content" source="./media/pricing-concepts/figure-1.png" alt-text="Exemple":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).

* Pour connaître la tarification actuelle, consultez la page [Tarification de Virtual WAN](https://azure.microsoft.com/pricing/details/virtual-wan/).