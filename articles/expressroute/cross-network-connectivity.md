---
title: Connectivité interréseau Azure
description: Cette page décrit un scénario d'application relatif à la connectivité interréseau et une solution basée sur des fonctionnalités réseau Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 04/03/2019
ms.author: duau
ms.openlocfilehash: 018afa1b2a31ebd44925a3fc79cbdc729b2d4695
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202495"
---
# <a name="cross-network-connectivity"></a>Connectivité interréseau

Fabrikam Inc. a une présence physique et un déploiement Azure importants dans la région USA Est. Fabrikam a une connectivité back-end entre ses déploiements locaux et Azure par le biais d’ExpressRoute. De même, Contoso Ltd. est présent et dispose d'un déploiement Azure dans la région USA Ouest. Contoso a une connectivité back-end entre ses déploiements locaux et Azure par le biais d’ExpressRoute.  

Fabrikam Inc. acquiert Contoso Ltd. Suite à la fusion, Fabrikam souhaite interconnecter les réseaux. La figure suivante illustre le scénario :

![Scénario d’application](./media/cross-network-connectivity/premergerscenario.png)

Les flèches pointillées situées au centre de la figure ci-dessus indiquent les interconnexions réseau souhaitées. Plus précisément, les interconnexions souhaitées se décomposent en trois types : 1) Interconnexion des réseaux virtuels de Fabrikam et Contoso, 2) Connexions interrégionales locales et connexions entre réseaux virtuels (c'est-à-dire la connexion du réseau local de Fabrikam au réseau virtuel de Contoso et la connexion du réseau local de Contoso au réseau virtuel de Fabrikam), et 3) Interconnexion des réseaux locaux de Fabrikam et Contoso. 

Le tableau suivant présente la table de routage du peering privé du circuit ExpressRoute de Contoso Ltd., avant la fusion.

![Table de routage ExpressRoute de Contoso avant la fusion](./media/cross-network-connectivity/contosoexr-rt-premerger.png)

Le tableau suivant présente les itinéraires effectifs d'une machine virtuelle de l'abonnement Contoso, avant la fusion. Selon le tableau, la machine virtuelle du réseau virtuel connaît l'espace d'adressage du réseau virtuel et le réseau local de Contoso, à l'exception de ceux par défaut.

![Itinéraires des machines virtuelles de Contoso avant la fusion](./media/cross-network-connectivity/contosovm-routes-premerger.png)

Le tableau suivant présente la table de routage du peering privé du circuit ExpressRoute de Fabrikam Inc, avant la fusion.

![Table de routage ExpressRoute de Fabrikam avant la fusion](./media/cross-network-connectivity/fabrikamexr-rt-premerger.png)

Le tableau suivant présente les itinéraires effectifs d'une machine virtuelle de l'abonnement Fabrikam, avant la fusion. Selon le tableau, la machine virtuelle du réseau virtuel connaît l'espace d'adressage du réseau virtuel et le réseau local de Fabrikam, à l'exception de ceux par défaut.

![Itinéraires des machines virtuelles de Fabrikam avant la fusion](./media/cross-network-connectivity/fabrikamvm-routes-premerger.png)

Dans cet article, nous allons passer chaque étape en revue et expliquer comment obtenir les interconnexions souhaitées à l'aide des fonctionnalités réseau Azure suivantes :

* [Peering de réseau virtuel][Virtual network peering] 
* [Connexion de réseau virtuel ExpressRoute][connection]
* [Global Reach][Global Reach] 

## <a name="cross-connecting-vnets"></a>Interconnexion des réseaux virtuels

Le peering de réseaux virtuels (VNet Peering) est celui qui offre les meilleurs résultats et les meilleures performances réseau lors de la connexion de deux réseaux virtuels. Le peering de réseaux virtuels prend en charge le peering de deux réseaux virtuels dans la même région Azure (communément appelé VNet Peering) ainsi que dans deux régions Azure différentes (communément appelé Global VNet Peering). 

Nous allons configurer le Global VNet Peering entre les réseaux virtuels des abonnements Azure de Contoso et Fabrikam. Pour plus d’informations sur l’appairage de deux réseaux virtuels, consultez l’article [Créer un appairage de réseaux virtuels][Configure VNet peering].

L'illustration suivante présente l'architecture réseau après la configuration du Global VNet Peering.

![Architecture après le VNet Peering](./media/cross-network-connectivity/vnet-peering.png )

Le tableau suivant présente les itinéraires connus pour accéder à la machine virtuelle de l'abonnement Contoso. Prêtez attention à la dernière entrée du tableau. Cette entrée est le résultat de l'interconnexion des réseaux virtuels.

![Itinéraires des machines virtuelles de Contoso après le VNet Peering](./media/cross-network-connectivity/contosovm-routes-peering.png)

Le tableau suivant présente les itinéraires connus pour accéder à la machine virtuelle de l'abonnement Fabrikam. Prêtez attention à la dernière entrée du tableau. Cette entrée est le résultat de l'interconnexion des réseaux virtuels.

![Itinéraires des machines virtuelles de Fabrikam après le VNet Peering](./media/cross-network-connectivity/fabrikamvm-routes-peering.png)

Le VNet Peering relie directement deux réseaux virtuels (absence de tronçon suivant pour l'entrée *VNetGlobalPeering* dans les deux tableaux ci-dessus)

## <a name="cross-connecting-vnets-to-the-on-premises-networks"></a>Interconnexion des réseaux virtuels avec les réseaux locaux

Nous pouvons connecter un circuit ExpressRoute à différents réseaux virtuels. Consultez [Limites du service et de l’abonnement][Subscription limits] pour connaître le nombre maximum de réseaux virtuels qui peuvent être connectés à un circuit ExpressRoute. 

Connectons le circuit ExpressRoute de Fabrikam au réseau virtuel de l'abonnement Contoso et, de la même manière, le circuit ExpressRoute de Contoso au réseau virtuel de l'abonnement Fabrikam afin de permettre l'interconnexion entre les réseaux virtuels et les réseaux locaux. Pour connecter un réseau virtuel à un circuit ExpressRoute d'un autre abonnement, vous devez créer et utiliser une autorisation.  Consultez l'article : [Connecter un réseau virtuel à un circuit ExpressRoute][Connect-ER-VNet].

L'illustration suivante présente l'architecture réseau après la configuration de l'interconnexion du circuit ExpressRoute avec les réseaux virtuels.

![Architecture après l'interconnexion des circuits ExpressRoute](./media/cross-network-connectivity/exr-x-connect.png)

Le tableau suivant présente la table de routage du peering privé du circuit ExpressRoute de Contoso Ltd., après l'interconnexion des réseaux virtuels avec les réseaux locaux via ExpressRoute. Vérifiez que la table de routage contient des itinéraires appartenant aux deux réseaux virtuels.

![Table de routage ExpressRoute de Contoso après l'interconnexion des circuits ExR et des réseaux virtuels](./media/cross-network-connectivity/contosoexr-rt-xconnect.png)

Le tableau suivant présente la table de routage du peering privé du circuit ExpressRoute de Fabrikam Inc., après l'interconnexion des réseaux virtuels avec les réseaux locaux via ExpressRoute. Vérifiez que la table de routage contient des itinéraires appartenant aux deux réseaux virtuels.

![Table de routage ExpressRoute de Fabrikam après l'interconnexion des circuits ExR et des réseaux virtuels](./media/cross-network-connectivity/fabrikamexr-rt-xconnect.png)

Le tableau suivant présente les itinéraires connus pour accéder à la machine virtuelle de l'abonnement Contoso. Prêtez attention aux entrées *Passerelle de réseau virtuel* du tableau. La machine virtuelle voit les itinéraires des deux réseaux locaux.

![Itinéraires des machines virtuelles de Contoso après l'interconnexion des circuits ExR et des réseaux virtuels](./media/cross-network-connectivity/contosovm-routes-xconnect.png)

Le tableau suivant présente les itinéraires connus pour accéder à la machine virtuelle de l'abonnement Fabrikam. Prêtez attention aux entrées *Passerelle de réseau virtuel* du tableau. La machine virtuelle voit les itinéraires des deux réseaux locaux.

![Itinéraires des machines virtuelles de Fabrikam après l'interconnexion des circuits ExR et des réseaux virtuels](./media/cross-network-connectivity/fabrikamvm-routes-xconnect.png)

>[!NOTE]
>Dans l'un ou l'autre des abonnements Fabrikam et/ou Contoso, vous pouvez également avoir des réseaux virtuels Spoke connectés au réseau virtuel Hub correspondant (la conception Hub et Spoke n'est pas illustrée dans les diagrammes d'architecture de cet article). Les interconnexions entre les passerelles de réseau virtuel Hub et ExpressRoute permettront également la communication entre les Hubs et Spokes Est et Ouest.
>

## <a name="cross-connecting-on-premises-networks"></a>Interconnexion des réseaux locaux

ExpressRoute Global Reach assure la connectivité entre les réseaux locaux connectés à différents circuits ExpressRoute. Configurons Global Reach entre les circuits ExpressRoute de Contoso et Fabrikam. Comme les circuits ExpressRoute se trouvent dans des abonnements différents, nous devons créer et utiliser une autorisation. Pour des instructions détaillées, consultez l’article [Configurer ExpressRoute Global Reach][Configure Global Reach].

L'illustration suivante présente l'architecture réseau après la configuration de Global Reach.

![Architecture après la configuration de Global Reach](./media/cross-network-connectivity/globalreach.png)

Le tableau suivant présente la table de routage du peering privé du circuit ExpressRoute de Contoso Ltd., après la configuration de Global Reach. Vérifiez que la table de routage contient des itinéraires appartenant aux deux réseaux locaux. 

![Table de routage ExpressRoute de Contoso après Global Reach](./media/cross-network-connectivity/contosoexr-rt-gr.png)

Le tableau suivant présente la table de routage du peering privé du circuit ExpressRoute de Fabrikam Inc., après la configuration de Global Reach. Vérifiez que la table de routage contient des itinéraires appartenant aux deux réseaux locaux.

![Table de routage ExpressRoute de Fabrikam après Global Reach]( ./media/cross-network-connectivity/fabrikamexr-rt-gr.png )

## <a name="next-steps"></a>Étapes suivantes

Pour toute question complémentaire sur les réseaux virtuels et le peering de réseau virtuel, consultez l’article [FAQ sur les réseaux virtuels][VNet-FAQ]. Pour toute question complémentaire sur ExpressRoute et la connectivité des réseaux virtuels, consultez l’article [FAQ sur ExpressRoute][ER-FAQ].

Global Reach est déployé pays/région par pays/région. Pour vérifier si Global Reach est disponible dans les pays/régions qui vous intéressent, consultez [ExpressRoute Global Reach][Global Reach].

<!--Link References-->
[Virtual network peering]: ../virtual-network/virtual-network-peering-overview.md
[connection]: ./expressroute-howto-linkvnet-portal-resource-manager.md
[Global Reach]: ./expressroute-global-reach.md
[Configure VNet peering]: ../virtual-network/create-peering-different-subscriptions.md
[Configure Global Reach]: ./expressroute-howto-set-global-reach.md
[Subscription limits]: ../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits
[Connect-ER-VNet]: ./expressroute-howto-linkvnet-portal-resource-manager.md
[ER-FAQ]: ./expressroute-faqs.md
[VNet-FAQ]: https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq