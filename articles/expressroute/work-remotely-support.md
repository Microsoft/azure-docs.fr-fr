---
title: Utilisation d’Azure ExpressRoute pour prendre en charge les utilisateurs distants
description: Cette page décrit comment tirer parti d’Azure ExpressRoute pour permettre le travail à distance dans le contexte de l’épidémie de COVID-19.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/22/2020
ms.author: duau
ms.openlocfilehash: d51f47b73fe572ce81d3e7b54f902f94fcd11b5b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89567671"
---
# <a name="using-azure-expressroute-to-create-hybrid-connectivity-to-support-remote-users"></a>Utilisation d’Azure ExpressRoute pour créer une connectivité hybride pour prendre en charge les utilisateurs distants

Cet article explique comment utiliser ExpressRoute, Azure, Microsoft Network et l’écosystème de partenaires Azure pour travailler à distance.

## <a name="connecting-to-azure-services-with-expressroute"></a>Connexion aux services Azure avec ExpressRoute

>[!NOTE]
>Cet article décrit comment vous pouvez tirer parti d’ExpressRoute, d’Azure, des réseaux Microsoft et de l’écosystème de partenaires Azure pour travailler à distance et atténuer les problèmes de réseau auxquels vous faites face dans le contexte de l’épidémie de COVID-19.
>

[ExpressRoute](expressroute-introduction.md) est un service Azure qui permet une connectivité privée entre les centres de données Microsoft et une infrastructure locale ou une installation de colocalisation. Les connexions ExpressRoute ne sont pas établies par le biais de l'Internet public. Elles offrent une connectivité sûre, de la fiabilité et de la rapidité, ainsi que des latences inférieures et cohérentes par rapport aux connexions classiques sur Internet.

## <a name="useful-links"></a>Liens utiles

* [Comment augmenter la bande passante pour le circuit ExpressRoute existant](expressroute-howto-circuit-portal-resource-manager.md#modify)
* [Surveillance, indicateurs de performance et alertes ExpressRoute](expressroute-monitoring-metrics-alerts.md#expressroute-gateway-connections-in-bitsseconds)
* [Optimisation de l’itinéraire sur ExpressRoute](expressroute-optimize-routing.md)
* [Azure ExpressRoute pour Microsoft 365](/microsoft-365/enterprise/azure-expressroute)
* [Considérations relatives au routage asymétrique](expressroute-asymmetric-routing.md)
* [Comment ouvrir une demande de support dans le portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

## <a name="troubleshoot"></a>Dépanner

* [Vérification de la connectivité ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* Obtention de la table ARP dans le [Gestionnaire des ressources](expressroute-troubleshooting-arp-resource-manager.md) et [Classique](expressroute-troubleshooting-arp-classic.md)
* [Réinitialiser un circuit ayant échoué](reset-circuit.md)
* [Résoudre les problèmes liés aux performances réseau](expressroute-troubleshooting-network-performance.md)

## <a name="next-steps"></a>Étapes suivantes

* [Informations supplémentaires sur les modèles de connectivité ExpressRoute](expressroute-connectivity-models.md)
* Découvrez en détail les connexions ExpressRoute et les domaines de routage. Consultez la page [Circuits ExpressRoute et domaines de routage](expressroute-circuit-peerings.md)
* Recherchez un fournisseur de services. Consultez [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md)
* Assurez-vous que toutes les conditions préalables sont remplies. Consultez la page [Configuration requise pour ExpressRoute](expressroute-prerequisites.md).
* [Création et modification d’un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
* [Créer et modifier le Peering pour un circuit ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Connexion d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
