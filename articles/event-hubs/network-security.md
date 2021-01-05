---
title: Sécurité réseau pour Azure Event Hubs
description: Cet article explique comment configurer l’accès à partir de points de terminaison privés
ms.topic: conceptual
ms.date: 10/20/2020
ms.openlocfilehash: 9503fc26c22d7dbff13c5754288f577b7bb3242f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010994"
---
# <a name="network-security-for-azure-event-hubs"></a>Sécurité du réseau pour Azure Event Hubs 
Cet article explique comment utiliser les fonctionnalités de sécurité suivantes avec Azure Event Hubs : 

- Balises de service
- Règles de pare-feu IP
- Points de terminaison de service réseau
- Instances Private Endpoint


## <a name="service-tags"></a>Balises de service
Une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné. Microsoft gère les préfixes d’adresses englobés par l’étiquette de service et met à jour automatiquement l’étiquette de service quand les adresses changent, ce qui réduit la complexité des mises à jour fréquentes relatives aux règles de sécurité réseau. Pour plus d’informations sur les étiquettes de service, consultez [Vue d’ensemble des balises de service](../virtual-network/service-tags-overview.md).

Vous pouvez utiliser des étiquettes de service pour définir des contrôles d’accès réseau sur des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md#security-rules) ou sur le [pare-feu Azure](../firewall/service-tags.md). Utilisez des étiquettes de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l'étiquette de service (par exemple, **EventHub**) dans le champ *source* ou *destination* approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant.

| Balise du service | Objectif | Peut-elle utiliser le trafic entrant ou sortant ? | Peut-elle être étendue à une zone régionale ? | Peut-elle être utilisée avec le Pare-feu Azure ? |
| --- | -------- |:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **EventHub** | Azure Event Hubs. | Règle de trafic sortant | Oui | Oui |


## <a name="ip-firewall"></a>Pare-feu IP 
Par défaut, les espaces de noms Event Hubs sont accessibles sur Internet tant que la demande s’accompagne d’une authentification et d’une autorisation valides. Avec le pare-feu IP, vous pouvez les limiter à un ensemble d’adresses IPv4 ou de plages d’adresses IPv4 dans la notation [CIDR (Classless InterDomain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Cette fonctionnalité est utile dans les scénarios où Azure Event Hubs ne doit être accessible qu’à partir de certains sites bien connus. Les règles de pare-feu permettent de configurer des règles pour accepter le trafic provenant d’adresses IPv4 spécifiques. Par exemple, si vous utilisez Event Hubs avec [Azure ExpressRoute](../expressroute/expressroute-faqs.md#supported-services), vous pouvez créer une **règle de pare-feu** pour autoriser uniquement le trafic provenant des adresses IP de votre infrastructure locale. 

Les règles de pare-feu IP sont appliquées au niveau de l’espace de noms Event Hubs. Par conséquent, les règles s’appliquent à toutes les connexions de clients utilisant un protocole pris en charge. Toute tentative de connexion à partir d’une adresse IP qui ne correspond pas à une règle IP autorisée dans l’espace de noms Event Hubs est rejetée comme étant non autorisée. La réponse ne mentionne pas la règle IP. Les règles de filtre IP sont appliquées dans l’ordre et la première règle qui correspond à l’adresse IP détermine l’action d’acceptation ou de rejet.

Pour plus d’informations, consultez le [Guide pratique pour configurer un pare-feu IP pour un Event Hub](event-hubs-ip-filtering.md)

## <a name="network-service-endpoints"></a>Points de terminaison de service réseau
L’intégration d’Event Hubs à des [points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md) permet de sécuriser l’accès aux fonctionnalités de messagerie à partir de charges de travail, notamment celles de machines virtuelles liées à des réseaux virtuels. Le chemin du trafic réseau est sécurisé aux deux extrémités.

Une fois configuré pour être lié à au moins un point de terminaison de service de sous-réseau de réseau virtuel, l’espace de noms Event Hubs respectif n’accepte que le trafic provenant de sous-réseaux autorisés dans les réseaux virtuels. Du point de vue du réseau virtuel, la liaison d’un espace de noms Event Hubs à un point de terminaison de service configure un tunnel de mise en réseau isolé allant du sous-réseau de réseau virtuel au service de messagerie. 

Il en résulte une relation privée et isolée entre les charges de travail liées au sous-réseau et l’espace de noms Event Hubs respectif, et ce malgré le fait que l’adresse réseau observable du point de terminaison du service de messagerie figure dans une plage d’adresses IP publique. Toutefois, il existe une exception à ce comportement. Par défaut, l’activation d’un point de terminaison de service active à la règle `denyall` dans le [pare-feu IP](event-hubs-ip-filtering.md) associé au réseau virtuel. Vous pouvez ajouter des adresses IP spécifiques dans le pare-feu IP pour permettre l’accès au point de terminaison public Event Hub. 

> [!IMPORTANT]
> Les réseaux virtuels sont pris en charge dans les niveaux **standard** et **dédié** d’Event Hubs. Il ne sont pas pris en charge dans le niveau **De base**.

### <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>Scénarios de sécurité avancés pris en charge par l’intégration à VNet 

En général, les solutions nécessitant une sécurité étroite et compartimentée et dans lesquelles les sous-réseaux de réseau virtuel assurent la segmentation entre les services compartimentés ont encore besoin de chemins de communication entre les services résidant dans ces compartiments.

Toute route IP immédiate entre les compartiments, notamment celles acheminant le trafic HTTPS sur TCP/IP, comporte un risque d’exploitation des vulnérabilités émanant de la couche réseau. Les services de messagerie fournissent des chemins de communication isolés, les messages étant même écrits sur disque à mesure qu’ils passent d’une partie à une autre. Les charges de travail dans deux réseaux virtuels distincts qui sont tous les deux liés à la même instance d’Event Hubs peuvent communiquer de manière efficace et fiable au moyen de messages, sans aucun impact sur l’intégrité des limites d’isolement de chaque réseau.
 
Vos solutions cloud sensibles en matière de sécurité accèdent donc aux fonctionnalités de messagerie asynchrones de pointe d’Azure qui allient fiabilité et scalabilité. Vos solutions peuvent désormais utiliser la messagerie pour créer des chemins de communication entre des compartiments de solution sécurisés, ce qui offre intrinsèquement une meilleure sécurité que n’importe quel mode de communication pair à pair (notamment HTTPS et d’autres protocoles de sockets sécurisés par TLS).

### <a name="bind-event-hubs-to-virtual-networks"></a>Lier des Event Hubs à des réseaux virtuels

Les **règles de réseau virtuel** sont une fonctionnalité de sécurité de pare-feu. Elles permettent de contrôler si votre espace de noms Azure Event Hubs doit accepter ou non les connexions d’un sous-réseau de réseau virtuel particulier.

La liaison d’un espace de noms Event Hubs à un réseau virtuel est un processus en deux étapes. Vous devez d’abord créer un **point de terminaison de service de réseau virtuel** sur un sous-réseau de réseau virtuel, puis l’activer pour **Microsoft.EventHub**, comme expliqué dans l’article [Vue d’ensemble des points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md). Après avoir ajouté le point de terminaison de service, liez-le à l’espace de noms Event Hubs au moyen d’une **règle de réseau virtuel**.

La règle de réseau virtuel est une association de l’espace de noms Event Hubs et d’un sous-réseau de réseau virtuel. Une fois la règle en place, toutes les charges de travail liées au sous-réseau sont autorisées à accéder à l’espace de noms Event Hubs. Event Hubs n’établit jamais de connexions sortantes, ne nécessite aucun accès et ne se voit donc jamais accorder l’accès à votre sous-réseau quand cette règle est activée.

Pour plus d’informations, consultez le [Guide pratique pour configurer des points de terminaison de service de réseau virtuel pour un Event Hub](event-hubs-service-endpoints.md)

## <a name="private-endpoints"></a>Instances Private Endpoint

Le [service Azure Private Link](../private-link/private-link-overview.md) vous permet d’accéder aux services Azure (par exemple, Azure Event Hubs, Stockage Azure et Azure Cosmos DB) ainsi qu’aux services de partenaire ou de client hébergés par Azure via un **point de terminaison privé** dans votre réseau virtuel.

Un point de terminaison privé est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez vous connecter à une instance d’une ressource Azure, ce qui vous donne le plus haut niveau de granularité en matière de contrôle d’accès.

> [!IMPORTANT]
> Cette fonctionnalité est prise en charge pour les niveaux **standard** et **dédié**. Il ne sont pas pris en charge dans le niveau **De base**.

Pour plus d’informations, consultez le [Guide pratique pour configurer des points de terminaison privés pour un Event Hub](private-link-service.md)


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Guide pratique pour configurer un pare-feu IP pour un Event Hub](event-hubs-ip-filtering.md)
- [Guide pratique pour configurer des points de terminaison de service de réseau virtuel pour un Event Hub](event-hubs-service-endpoints.md)
- [Guide pratique pour configurer des points de terminaison privés pour un Event Hub](private-link-service.md)