---
title: 'Vue d’ensemble d’Azure ExpressRoute : connexion via une connexion privée'
description: La présentation technique d’ExpressRoute détaille le fonctionnement d’une connexion ExpressRoute pour étendre votre réseau local à Azure via une connexion privée.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: overview
ms.date: 10/05/2020
ms.author: duau
ms.openlocfilehash: 82562eae748753cd785851c5d91f5f152b4c9960
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206949"
---
# <a name="what-is-azure-expressroute"></a>Qu’est-ce qu’Azure ExpressRoute ?
ExpressRoute vous permet d’étendre vos réseaux locaux dans le cloud Microsoft via une connexion privée avec l’aide d’un fournisseur de connectivité. Avec ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, comme Microsoft Azure et Microsoft 365.

La connectivité peut provenir d’un réseau universel (IP VPN), d’un réseau Ethernet point à point ou d’une interconnexion virtuelle via un fournisseur de connectivité dans un centre de colocalisation. Les connexions ExpressRoute ne passent pas par l’Internet public. Elles offrent ainsi une meilleure fiabilité, des vitesses supérieures, des latences cohérentes et une plus grande sécurité que les connexions classiques sur Internet. Pour plus d’informations sur la connexion de votre réseau à Microsoft à l’aide d’ExpressRoute, consultez [Modèles de connectivité ExpressRoute](expressroute-connectivity-models.md).

![Présentation de la connexion à ExpressRoute](./media/expressroute-introduction/expressroute-connection-overview.png)

## <a name="key-benefits"></a>Principaux avantages

* Connectivité de couche 3 entre votre réseau local et le cloud de Microsoft via un fournisseur de connectivité. La connectivité peut provenir d'un réseau universel (IPVPN), d’une connexion Ethernet point à point, ou d’une interconnexion virtuelle via un échange Ethernet.
* Connectivité aux services de cloud de Microsoft dans toutes les régions de la zone géopolitique.
* Connectivité globale aux services de Microsoft dans toutes les régions grâce au module complémentaire ExpressRoute premium.
* Routage dynamique entre votre réseau et Microsoft via le protocole de routage dynamique standard (BGP).
* Redondance intégrée dans chaque emplacement de peering pour une plus grande fiabilité.
* [SLA](https://azure.microsoft.com/support/legal/sla/)de disponibilité de la connexion.
* Support de la qualité de service pour Skype Entreprise.

Pour plus d’informations, consultez le [Forum Aux Questions ExpressRoute](expressroute-faqs.md).

## <a name="features"></a>Fonctionnalités

### <a name="layer-3-connectivity"></a>Connectivité de couche 3
Microsoft utilise le protocole de routage dynamique standard (BGP) pour échanger des routages entre votre réseau local, vos instances dans Azure et les adresses publiques Microsoft. Nous établissons plusieurs sessions BGP avec votre réseau pour tester différents profils de trafic. Vous trouverez plus de détails à ce sujet dans l’article [Circuit ExpressRoute et domaines de routage](expressroute-circuit-peerings.md) .

### <a name="redundancy"></a>Redondance
Chaque circuit ExpressRoute se compose de deux connexions à deux routeurs Microsoft Enterprise Edge (MSEE) entre le fournisseur de connectivité et le périmètre de votre réseau au niveau d’un [site ExpressRoute](./expressroute-locations.md#expressroute-locations). Microsoft nécessite une double connexion BGP entre le fournisseur de connectivité et la périphérie de votre réseau, une pour chaque routeur MSEE. Vous pouvez choisir de ne pas déployer d’appareils redondants/de circuits Ethernet de votre côté. Cependant, les fournisseurs de connectivité utilisent des appareils redondants pour garantir que vos connexions sont transmises à Microsoft de manière redondante. Une configuration de connectivité de couche 3 redondante est nécessaire à la validité de notre [SLA](https://azure.microsoft.com/support/legal/sla/) .

### <a name="connectivity-to-microsoft-cloud-services"></a>Connectivité aux services de cloud Microsoft
Les connexions ExpressRoute permettent d’accéder aux services suivants :
* Services Microsoft Azure
* Services Microsoft 365

> [!NOTE]
> [!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]
> 

Pour consulter une liste détaillée des services pris en charge via ExpressRoute, visitez la page [Forum Aux Questions ExpressRoute](expressroute-faqs.md).

### <a name="connectivity-to-all-regions-within-a-geopolitical-region"></a>Connectivité à toutes les zones d’une région géopolitique
Vous pouvez vous connecter à Microsoft dans un de nos [emplacements de peering](expressroute-locations.md) et ainsi accéder à toutes les zones de la région géopolitique.

Par exemple, si vous vous connectez à Microsoft à Amsterdam via ExpressRoute. Vous aurez accès à tous les services cloud Microsoft hébergés dans les régions Europe Nord et Europe Ouest. Pour obtenir une vue d’ensemble des régions géopolitiques, des zones de cloud Microsoft associées et des emplacements de peering ExpressRoute correspondants, consultez l’article [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md).

### <a name="global-connectivity-with-expressroute-premium"></a>Connectivité globale avec ExpressRoute Premium
Vous pouvez activer [ExpressRoute Premium](expressroute-faqs.md) pour étendre la connectivité au-delà des limites géopolitiques. Par exemple, si vous êtes connecté à Microsoft à Amsterdam via ExpressRoute, vous aurez accès à tous les services cloud Microsoft hébergés dans toutes les régions du monde. Vous pouvez aussi accéder aux services déployés en Amérique du Sud ou en Australie de la même façon que vous accédez aux régions Europe Nord et Europe Ouest. Les clouds nationaux sont exclus.

### <a name="local-connectivity-with-expressroute-local"></a>Connectivité locale avec ExpressRoute Local
Vous pouvez transférer des données de façon économique en activant la [référence SKU Local](expressroute-faqs.md). Avec la référence SKU Local, vous pouvez placer vos données à un emplacement ExpressRoute proche de la région Azure de votre choix. Avec la référence SKU Local, le transfert de données est compris dans les frais liés au port ExpressRoute. 

### <a name="across-on-premises-connectivity-with-expressroute-global-reach"></a>Connectivité locale avec ExpressRoute Global Reach
Vous pouvez activer ExpressRoute Global Reach pour échanger des données entre vos sites locaux en connectant vos circuits ExpressRoute. Par exemple, si vous avez un centre de données privé en Californie connecté à un circuit ExpressRoute dans la Silicon Valley et un autre centre de données privé dans le Texas connecté à un circuit ExpressRoute à Dallas. Avec ExpressRoute Global Reach, vous pouvez connecter ensemble vos centres de données privés via ces deux circuits ExpressRoute. Le trafic entre vos centres de données transitera par réseau Microsoft.

Pour plus d’informations, consultez [ExpressRoute Global Reach](expressroute-global-reach.md).
### <a name="rich-connectivity-partner-ecosystem"></a>Riche écosystème de partenaires de connectivité
ExpressRoute offre un écosystème sans cesse croissant de fournisseurs de connectivité et de partenaires intégrateurs de systèmes. Vous pouvez consulter l’article [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md) pour obtenir les dernières informations.

### <a name="connectivity-to-national-clouds"></a>Connectivité aux clouds nationaux
Microsoft gère des environnements de cloud isolés dans des régions géopolitiques et des segments de clientèle spécifiques. Reportez-vous à la page [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md) pour obtenir la liste des clouds nationaux et des fournisseurs.

### <a name="expressroute-direct"></a>ExpressRoute Direct
Avec ExpressRoute Direct, les clients ont la possibilité de se connecter directement au réseau international de Microsoft à partir d’emplacements de peering qui sont distribués stratégiquement dans le monde entier. ExpressRoute Direct offre une double connectivité de 100 Gbit/s qui prend en charge la connectivité Active/Active à grande échelle.

Les principales fonctionnalités offertes par ExpressRoute Direct sont les suivantes (sans s’y limiter) :

* Ingestion de données massive dans des services comme le stockage et Cosmos DB
* Isolation physique pour les secteurs qui sont réglementés, pour lesquels la connectivité isolée et dédiée est obligatoire banque, secteur public et commerce de détail
* Contrôle précis de la distribution du circuit en fonction de l’unité commerciale

Pour plus d’informations, consultez [A propos d’ExpressRoute Direct](./expressroute-erdirect-about.md).

### <a name="bandwidth-options"></a>Options de bande passante
Vous pouvez acheter des circuits ExpressRoute pour un large éventail de bandes passantes. Les bandes passantes prises en charge sont listées ci-dessous. Veillez à vérifier auprès de votre fournisseur de connectivité la liste des bandes passantes prises en charge qu’il propose.

* 50 Mbits/s
* 100 Mbits/s
* 200 Mbits/s
* 500 Mbits/s
* 1 Gbit/s
* 2 Gbit/s
* 5 Gbit/s
* 10 Gbits/s

### <a name="dynamic-scaling-of-bandwidth"></a>Mise à l'échelle dynamique de la bande passante
Vous pouvez augmenter la bande passante du circuit ExpressRoute (dans la mesure du possible) sans ralentir vos connexions. Pour plus d’informations, consultez [Modification d’un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md#modify).

### <a name="flexible-billing-models"></a>Modèles de facturation flexibles
Vous pouvez choisir le modèle de facturation qui vous convient le mieux. Choisissez parmi les modèles de facturation listés ci-dessous. Pour plus d’informations, consultez le [Forum Aux Questions ExpressRoute](expressroute-faqs.md).

* **Données illimitées**. Ce service est facturé sur une base mensuelle, et tous les transferts de données entrants et sortants sont inclus gratuitement.
* **Données limitées**. Ce service est facturé sur une base mensuelle, et tous les transferts de données entrants sont inclus gratuitement. Chaque transfert de données sortant est facturé par Go de données transférées. Les taux de transfert de données varient selon la région.
* **Module complémentaire ExpressRoute premium**. Le module complémentaire ExpressRoute premium est un module qui s’ajoute au circuit ExpressRoute. Le module complémentaire ExpressRoute premium offre les fonctionnalités suivantes : 
  * Augmentation de 4 000 à 10 000 routages des limites de routage pour les peerings publics et privés Azure.
  * Connectivité globale des services. Un circuit ExpressRoute créé dans une région quelconque (à l’exclusion des clouds nationaux) a accès aux ressources de chacune des autres régions dans le monde. Par exemple, un réseau virtuel créé dans la région Europe Ouest est accessible via un circuit ExpressRoute configuré dans la Silicon Valley.
  * Augmentation de 10 à une limite supérieure du nombre de liens de réseaux virtuels par circuit ExpressRoute, selon la bande passante du circuit.

## <a name="faq"></a>Questions fréquentes (FAQ)
Pour les questions fréquemment posées sur ExpressRoute, consultez le [Forum aux questions ExpressRoute](expressroute-faqs.md).

## <a name="whats-new"></a><a name="new"></a>Nouveautés

Abonnez-vous au flux RSS et découvrez les dernières mises à jour des fonctionnalités ExpressRoute dans la page [Mises à jour Azure](https://azure.microsoft.com/updates/?category=networking&query=ExpressRoute).

## <a name="next-steps"></a>Étapes suivantes
* Assurez-vous que toutes les conditions préalables sont remplies. Consultez la page [Configuration requise pour ExpressRoute](expressroute-prerequisites.md).
* Consultez des informations supplémentaires sur les [modèles de connectivité ExpressRoute](expressroute-connectivity-models.md).
* Recherchez un fournisseur de services. Consultez [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md).