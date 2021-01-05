---
title: Circuits Azure ExpressRoute et peering
description: Cette page fournit une vue d’ensemble des circuits ExpressRoute et des domaines de routage/peering.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: duau
ms.openlocfilehash: 87fed1d2ac4f5fa85c01d7af10bec10c1412744f
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202342"
---
# <a name="expressroute-circuits-and-peering"></a>Circuits ExpressRoute et peering

Les circuits ExpressRoute vous permettent de connecter votre infrastructure locale à Microsoft via un fournisseur de connectivité. Cet article vous aide à comprendre les circuits ExpressRoute et les domaines de routage/le peering. La figure suivante affiche une représentation logique de la connectivité entre votre WAN et Microsoft.

![Diagramme montrant la façon dont les circuits ExpressRoute connectent votre infrastructure locale à Microsoft via un fournisseur de connectivité.](./media/expressroute-circuit-peerings/expressroute-basic.png)

> [!IMPORTANT]
> Le peering public Azure a été déconseillé, et il n’est pas disponible pour les nouveaux circuits ExpressRoute. Les nouveaux circuits prennent en charge le peering Microsoft et le peering privé.  
>

## <a name="expressroute-circuits"></a><a name="circuits"></a>Circuits ExpressRoute

Un circuit ExpressRoute représente une connexion logique entre votre infrastructure locale et des services de cloud computing Microsoft via un fournisseur de connexion. Vous pouvez commander plusieurs circuits ExpressRoute. Chaque circuit peut se trouver dans une région identique ou différente des autres, et peut être connecté à votre site via des fournisseurs de connectivité.

Les circuits ExpressRoute ne sont mappés à aucune entité physique. Un circuit est identifié par un GUID standard appelé clé de service (s-key). La clé de service est la seule information échangée entre Microsoft, le fournisseur de connectivité, et vous. La clé de service n'est pas secrète pour des raisons de sécurité. Il existe un mappage 1:1 entre un circuit ExpressRoute et la clé de service.

Les nouveaux circuits ExpressRoute peuvent inclure deux peerings indépendants : Le peering privé et le peering Microsoft. En revanche, les circuits ExpressRoute existants peuvent contenir trois peerings : public Azure, privé Azure et Microsoft. Chaque peering est une paire de sessions BGP indépendantes, chacune configurée de manière redondante pour offrir une disponibilité optimale. Il existe un mappage 1:N (1 <= N <= 3) entre un circuit ExpressRoute et des domaines de routage. Un circuit ExpressRoute peut avoir un, deux ou trois peerings activés par circuit ExpressRoute.

Chaque circuit offre une bande passante fixe (50 Mbps, 100 Mbps, 200 Mbps, 500 Mbps, 1 Gbps, 10 Gbps) et est mappé à un fournisseur de connectivité et un emplacement de peering. La bande passante sélectionnée est partagée par tous les peerings de circuit.

### <a name="quotas-limits-and-limitations"></a><a name="quotas"></a>Quotas, limites et limitations

Des limites et quotas par défaut s'appliquent à chaque circuit ExpressRoute. Consultez la page [Limites, quotas et contraintes applicables à l’abonnement et au service Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) pour obtenir des informations actualisées sur les quotas.

## <a name="expressroute-peering"></a><a name="routingdomains"></a>Peering ExpressRoute

Un circuit ExpressRoute a plusieurs domaines de routage/peerings qui lui sont associés : public Azure, privé Azure et Microsoft. Chaque peering est configuré de manière identique sur une paire de routeurs (en configuration actif-actif ou de partage de la charge) pour la haute disponibilité. Les services Azure sont classés en *Public Azure* et *Privé Azure* pour représenter les schémas d’adressage IP.

![Diagramme montrant la façon dont les Peerings publics Azure, privés Azure et Microsoft sont configurés dans un circuit ExpressRoute.](./media/expressroute-circuit-peerings/expressroute-peerings.png)

### <a name="azure-private-peering"></a><a name="privatepeering"></a>Peering privé Azure

Les services de calcul Azure, à savoir les machines virtuelles (IaaS) et les services cloud (PaaS) déployés au sein d’un réseau virtuel peuvent être connectés via le domaine de peering privé. Celui-ci est considéré comme une extension de confiance de votre réseau de base dans Microsoft Azure. Vous pouvez configurer une connectivité bidirectionnelle entre votre réseau de base et les réseaux virtuels Azure. Ce peering vous permet de vous connecter aux machines virtuelles et services cloud directement sur leurs adresses IP privées.  

Vous pouvez connecter plusieurs réseaux virtuels au domaine de peering privé. Pour plus d’informations sur les limitations, consultez le [Forum Aux Questions](expressroute-faqs.md) . Vous pouvez consulter la page [Limites, quotas et contraintes applicables à l’abonnement et au service Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) pour obtenir des informations actualisées sur les limites.  Reportez-vous à la page [Routage](expressroute-routing.md) pour plus d'informations sur la configuration du routage.

### <a name="microsoft-peering"></a><a name="microsoftpeering"></a>Peering Microsoft

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

La connectivité aux services en ligne Microsoft (Microsoft 365 et les services PaaS Azure) s’effectue via un Peering Microsoft. Nous activons la connectivité bidirectionnelle entre votre réseau étendu et les services cloud Microsoft via le domaine de routage de peering Microsoft. Vous devez vous connecter aux services cloud Microsoft uniquement via des adresses IP publiques qui sont détenues par vous ou votre fournisseur de connectivité. Vous devez également respecter toutes les règles définies. Pour plus d’informations, consultez la page [Composants requis ExpressRoute](expressroute-prerequisites.md).

Pour plus d’informations sur les services pris en charge, les coûts et les détails de configuration, consultez le [Forum Aux Questions](expressroute-faqs.md) . Pour plus d’informations sur la liste des fournisseurs de connectivité offrant une prise en charge du peering Microsoft, consultez la page [Emplacements ExpressRoute](expressroute-locations.md)

## <a name="peering-comparison"></a><a name="peeringcompare"></a>Comparaison de peerings

Le tableau suivant compare les trois types de peering :

[!INCLUDE [peering comparison](../../includes/expressroute-peering-comparison.md)]

Vous pouvez activer un ou plusieurs domaines de routage dans le cadre de votre circuit ExpressRoute. Vous pouvez choisir de placer tous les domaines de routage sur le même VPN si vous souhaitez les combiner dans un domaine de routage unique. Vous pouvez également les placer dans différents domaines de routage comme indiqué dans le schéma. Nous vous recommandons de connecter le peering privé directement à votre réseau principal, et les peerings public et Microsoft à votre zone DMZ.

Chaque peering requiert des sessions BGP distinctes (une paire pour chaque type de peering). Les paires de session BGP fournissent un lien hautement disponible. Si vous vous connectez via des fournisseurs de connectivité de couche 2, il vous incombe de configurer et de gérer le routage. Pour en savoir plus, passez en revue les [workflows](expressroute-workflows.md) d’ExpressRoute.

## <a name="expressroute-health"></a><a name="health"></a>Intégrité ExpressRoute

Les circuits ExpressRoute peuvent faire l’objet d’une surveillance en matière de disponibilité, de connectivité aux réseaux virtuels et d’utilisation de la bande passante à l’aide de [Network Performance Monitor](../networking/network-monitoring-overview.md) (NPM).

NPM supervise l’intégrité du peering privé Azure et du peering Microsoft. Pour plus d’informations, consultez notre [billet](https://azure.microsoft.com/blog/monitoring-of-azure-expressroute-in-preview/).

## <a name="next-steps"></a>Étapes suivantes

* Recherchez un fournisseur de services. Consultez la rubrique [Emplacements et fournisseurs de services ExpressRoute](expressroute-locations.md).
* Assurez-vous que toutes les conditions préalables sont remplies. Consultez la page [Configuration requise pour ExpressRoute](expressroute-prerequisites.md).
* Configurez votre connexion ExpressRoute.
  * [Créer et gérer des circuits ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
  * [Configurer le routage (peering) pour les circuits ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)