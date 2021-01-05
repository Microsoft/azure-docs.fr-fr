---
title: Sécurité réseau pour Azure Service Bus
description: Cet article décrit les fonctionnalités de sécurité réseau, comme les balises de service, les règles de pare-feu IP, les points de terminaison de service et les points de terminaison privés.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: db0dd89d1f902699c27b724609505ba681757454
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92310459"
---
# <a name="network-security-for-azure-service-bus"></a>Sécurité réseau pour Azure Service Bus 
Cet article explique comment utiliser les fonctionnalités de sécurité suivantes avec Azure Service Bus : 

- Balises de service
- Règles de pare-feu IP
- Points de terminaison de service réseau
- Instances Private Endpoint


## <a name="service-tags"></a>Balises de service
Une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné. Microsoft gère les préfixes d’adresses englobés par l’étiquette de service et met à jour automatiquement l’étiquette de service quand les adresses changent, ce qui réduit la complexité des mises à jour fréquentes relatives aux règles de sécurité réseau. Pour plus d’informations sur les étiquettes de service, consultez [Vue d’ensemble des balises de service](../virtual-network/service-tags-overview.md).

Vous pouvez utiliser des étiquettes de service pour définir des contrôles d’accès réseau sur des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md#security-rules) ou sur le [pare-feu Azure](../firewall/service-tags.md). Utilisez des étiquettes de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple, **ServiceBus**) dans le champ *source* ou de *destination* approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant.

| Balise du service | Objectif | Peut-elle utiliser le trafic entrant ou sortant ? | Peut-elle être étendue à une zone régionale ? | Peut-elle être utilisée avec le Pare-feu Azure ? |
| --- | -------- |:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **ServiceBus** | Trafic Azure Service Bus qui utilise le niveau de service Premium. | Règle de trafic sortant | Oui | Oui |


> [!NOTE]
> Vous pouvez utiliser des balises de service uniquement pour les espaces de noms **Premium**. Si vous utilisez un espace de noms **standard**, utilisez l’adresse IP que vous voyez lorsque vous exécutez la commande suivante : `nslookup <host name for the namespace>`. Par exemple : `nslookup contosons.servicebus.windows.net`. 

## <a name="ip-firewall"></a>Pare-feu IP 
Par défaut, les espaces de noms Service Bus sont accessibles à partir d’Internet tant que la demande s’accompagne d’une authentification et d’une autorisation valides. Avec le pare-feu IP, vous pouvez la limiter à un ensemble d’adresses IPv4 ou de plages d’adresses IPv4 dans la notation [CIDR (Classless Inter-Domain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Cette fonctionnalité est utile dans les scénarios où Azure Service Bus ne doit être accessible qu’à partir de certains sites bien connus. Les règles de pare-feu permettent de configurer des règles pour accepter le trafic provenant d’adresses IPv4 spécifiques. Par exemple, si vous utilisez Service Bus avec [Azure Express Route][express-route], vous pouvez créer une **règle de pare-feu** pour autoriser uniquement le trafic provenant d’adresses IP de votre infrastructure locale ou d’adresses d’une passerelle NAT d’entreprise. 

Les règles de pare-feu IP sont appliquées au niveau de l’espace de noms Service Bus. Par conséquent, les règles s’appliquent à toutes les connexions de clients utilisant un protocole pris en charge. Toute tentative de connexion à partir d’une adresse IP qui ne correspond pas à une règle IP autorisée dans l’espace de noms Service Bus est rejetée comme étant non autorisée. La réponse ne mentionne pas la règle IP. Les règles de filtre IP sont appliquées dans l’ordre et la première règle qui correspond à l’adresse IP détermine l’action d’acceptation ou de rejet.

Pour plus d’informations, consultez [Guide pratique pour configurer un pare-feu IP pour un espace de noms Service Bus](service-bus-ip-filtering.md).

## <a name="network-service-endpoints"></a>Points de terminaison de service réseau
L’intégration de Service Bus à des [points de terminaison de service de réseau virtuel (VNet)](service-bus-service-endpoints.md) permet de sécuriser l’accès aux fonctionnalités de messagerie à partir de charges de travail, notamment celles de machines virtuelles liées à des réseaux virtuels. Le chemin du trafic réseau est sécurisé aux deux extrémités.

Une fois configuré pour être lié à au moins un point de terminaison de service de sous-réseau de réseau virtuel, l’espace de noms Service Bus respectif n’accepte que le trafic provenant de réseaux virtuels autorisés. Du point de vue du réseau virtuel, la liaison d’un espace de noms Service Bus à un point de terminaison de service configure un tunnel de mise en réseau isolé allant du sous-réseau de réseau virtuel au service de messagerie.

Il en résulte une relation privée et isolée entre les charges de travail liées au sous-réseau et l’espace de noms Service Bus respectif, et ce malgré le fait que l’adresse réseau observable du point de terminaison du service de messagerie figure dans une plage d’adresses IP publique.

> [!IMPORTANT]
> Les réseaux virtuels sont pris en charge uniquement dans les espaces de noms du [niveau Premium](service-bus-premium-messaging.md) de Service Bus.
> 
> Quand vous utilisez des points de terminaison de service de réseau virtuel avec Service Bus, vous ne devez pas activer ces points de terminaison dans les applications qui combinent des espaces de noms Service Bus de niveau Standard et Premium. Étant donné que le niveau Standard ne prend pas en charge les réseaux virtuels, le point de terminaison est limité aux espaces de noms du niveau Premium uniquement.

### <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>Scénarios de sécurité avancés pris en charge par l’intégration à VNet 

En général, les solutions nécessitant une sécurité étroite et compartimentée et dans lesquelles les sous-réseaux de réseau virtuel assurent la segmentation entre les services compartimentés ont toujours besoin de chemins de communication entre les services résidant dans ces compartiments.

Toute route IP immédiate entre les compartiments, notamment celles acheminant le trafic HTTPS sur TCP/IP, comporte un risque d’exploitation des vulnérabilités émanant de la couche réseau. Les services de messagerie fournissent des chemins de communication complètement isolés, les messages étant même écrits sur disque à mesure qu’ils passent d’une partie à une autre. Les charges de travail dans deux réseaux virtuels distincts qui sont tous les deux liés à la même instance de Service Bus peuvent communiquer de manière efficace et fiable au moyen de messages, sans aucun impact sur l’intégrité des limites d’isolement de chaque réseau.
 
Vos solutions cloud sensibles en matière de sécurité accèdent donc aux fonctionnalités de messagerie asynchrones de pointe d’Azure qui allient fiabilité et scalabilité. Vos solutions peuvent désormais utiliser la messagerie pour créer des chemins de communication entre des compartiments de solution sécurisés, ce qui offre intrinsèquement une meilleure sécurité que n’importe quel mode de communication pair à pair (notamment HTTPS et d’autres protocoles de sockets sécurisés par TLS).

### <a name="bind-service-bus-to-virtual-networks"></a>Lier Service Bus à des réseaux virtuels

Les *règles de réseau virtuel* sont une fonctionnalité de sécurité de pare-feu. Elles permettent de contrôler si votre serveur Azure Service Bus doit accepter ou non les connexions d’un sous-réseau de réseau virtuel particulier.

La liaison d’un espace de noms Service Bus à un réseau virtuel est un processus en deux étapes. Vous devez d’abord créer un **point de terminaison de service de réseau virtuel** sur un sous-réseau de réseau virtuel et l’activer pour **Microsoft.ServiceBus**, comme expliqué dans la [vue d’ensemble des points de terminaison de service](service-bus-service-endpoints.md). Après avoir ajouté le point de terminaison de service, liez-le à l’espace de noms Service Bus au moyen d’une **règle de réseau virtuel**.

La règle de réseau virtuel est une association de l’espace de noms Service Bus et d’un sous-réseau de réseau virtuel. Une fois la règle en place, toutes les charges de travail liées au sous-réseau sont autorisées à accéder à l’espace de noms Service Bus. Service Bus n’établit jamais de connexions sortantes, ne nécessite aucun accès et ne se voit donc jamais accorder l’accès à votre sous-réseau quand cette règle est activée.

Pour plus d’informations, consultez [Guide pratique pour configurer des points de terminaison de service de réseau virtuel pour un espace de noms Service Bus](service-bus-service-endpoints.md).

## <a name="private-endpoints"></a>Instances Private Endpoint

Le service Azure Private Link vous permet d’accéder aux services Azure (par exemple, Azure Service Bus, le Stockage Azure et Azure Cosmos DB) ainsi qu’aux services de partenaires/clients hébergés par Azure sur un **point de terminaison privé** de votre réseau virtuel.

Un point de terminaison privé est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez vous connecter à une instance d’une ressource Azure, ce qui vous donne le plus haut niveau de granularité en matière de contrôle d’accès.

Pour plus d’informations, consultez [Qu’est-ce qu’Azure Private Link ?](../private-link/private-link-overview.md)

> [!NOTE]
> Cette fonctionnalité est prise en charge avec le niveau **Premium** d’Azure Service Bus. Pour plus d’informations sur le niveau Premium, consultez l’article [Couches de messagerie Service Bus Premium et Standard](service-bus-premium-messaging.md).


Pour plus d’informations, consultez [Guide pratique pour configurer des points de terminaison privés pour un espace de noms Service Bus](private-link-service.md).


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Guide pratique pour configurer un pare-feu IP pour un espace de noms Service Bus](service-bus-ip-filtering.md)
- [Guide pratique pour configurer des points de terminaison de service de réseau virtuel pour un espace de noms Service Bus](service-bus-service-endpoints.md)
- [Guide pratique pour configurer des points de terminaison privés pour un espace de noms Service Bus](private-link-service.md)