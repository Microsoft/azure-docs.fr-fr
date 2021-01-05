---
title: Sécurité réseau pour Azure Relay
description: Cet article explique comment utiliser des règles de pare-feu IP et des points de terminaison privés avec Azure Relay.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 3aa3ffd119f65ec5181b0c382472cc4ef3c8bac4
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91263723"
---
# <a name="network-security-for-azure-relay"></a>Sécurité réseau pour Azure Relay 
Cet article explique comment utiliser les fonctionnalités de sécurité suivantes avec Azure Relay : 

- Règles de pare-feu IP (préversion)
- Points de terminaison privés (préversion)

> [!NOTE]
> Azure Relay ne prend pas en charge les points de terminaison de service réseau. 


## <a name="ip-firewall"></a>Pare-feu IP 
Par défaut, les espaces de noms Azure Relay sont accessibles sur Internet tant que la requête s’accompagne d’une authentification et d’une autorisation valides. Avec le pare-feu IP, vous pouvez les limiter à un ensemble d’adresses IPv4 ou de plages d’adresses IPv4 dans la notation [CIDR (Classless InterDomain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Cette fonctionnalité est utile dans les scénarios où Azure Relay ne doit être accessible qu’à partir de certains sites bien connus. Les règles de pare-feu permettent de configurer des règles pour accepter le trafic provenant d’adresses IPv4 spécifiques. Par exemple, si vous utilisez Azure Relay avec [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services), vous pouvez créer une **règle de pare-feu** pour autoriser uniquement le trafic provenant des adresses IP de votre infrastructure locale. 

Les règles de pare-feu IP sont appliquées au niveau de l’espace de noms Relay. Par conséquent, les règles s’appliquent à toutes les connexions de clients utilisant un protocole pris en charge. Toute tentative de connexion à partir d’une adresse IP qui ne correspond pas à une règle IP autorisée dans l’espace de noms Relay est rejetée comme étant non autorisée. La réponse ne mentionne pas la règle IP. Les règles de filtre IP sont appliquées dans l’ordre et la première règle qui correspond à l’adresse IP détermine l’action d’acceptation ou de rejet.

Pour plus d’informations, consultez [Guide pratique pour configurer un pare-feu IP pour un espace de noms Relay](ip-firewall-virtual-networks.md)

## <a name="private-endpoints"></a>Instances Private Endpoint

Le service **Azure Private Link** vous permet d’accéder aux services Azure (par exemple,Azure Relay, Azure Service Bus, Azure Event Hubs, Stockage Azure et Azure Cosmos DB) ainsi qu’aux services de partenaires/clients hébergés par Azure sur un point de terminaison privé de votre réseau virtuel. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Private Link (préversion) ?](../private-link/private-link-overview.md)

Un **point de terminaison privé** est une interface réseau qui permet à vos charges de travail exécutées dans un réseau virtuel de se connecter en privé et en toute sécurité à un service qui dispose d’une **ressource de liaison privée** (par exemple, un espace de noms Azure Relay). Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ni d’IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez fournir un niveau de granularité dans le contrôle d’accès en autorisant des connexions à des espaces de noms Azure Relay spécifiques.

> [!NOTE]
> Cette fonctionnalité est actuellement en **préversion**. 

Pour plus d’informations, consultez [Comment configurer les points de terminaison privés](private-link-service.md)


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Comment configurer le pare-feu IP](ip-firewall-virtual-networks.md)
- [Comment configurer les points de terminaison privés](private-link-service.md)
