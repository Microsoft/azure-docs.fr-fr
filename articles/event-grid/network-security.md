---
title: Sécurité du réseau pour les ressources Azure Event Grid
description: Cet article explique comment utiliser des étiquettes de service pour la sortie, des règles de pare-feu IP pour l’entrée et des points de terminaison privés pour les entrées à l’aide d’Azure Event Grid.
author: VidyaKukke
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: vkukke
ms.openlocfilehash: 10c9b165041f0a4a1f09511f17bef3629353c3b2
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94917526"
---
# <a name="network-security-for-azure-event-grid-resources"></a>Sécurité du réseau pour les ressources Azure Event Grid
Cet article explique comment utiliser les fonctionnalités de sécurité suivantes avec Azure Event Grid : 

- Étiquettes de service en sortie
- Règles de pare-feu IP en entrée
- Points de terminaison privés pour l’entrée


## <a name="service-tags"></a>Balises de service
Une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné. Microsoft gère les préfixes d’adresses englobés par l’étiquette de service et met à jour automatiquement l’étiquette de service quand les adresses changent, ce qui réduit la complexité des mises à jour fréquentes relatives aux règles de sécurité réseau. Pour plus d’informations sur les étiquettes de service, consultez [Vue d’ensemble des balises de service](../virtual-network/service-tags-overview.md).

Vous pouvez utiliser des étiquettes de service pour définir des contrôles d’accès réseau sur des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md#security-rules) ou sur le [pare-feu Azure](../firewall/service-tags.md). Utilisez des étiquettes de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l'étiquette de service (par exemple, **AzureEventGrid**) dans le champ *source* ou *destination* approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant.

| Balise du service | Objectif | Peut-elle utiliser le trafic entrant ou sortant ? | Peut-elle être étendue à une zone régionale ? | Peut-elle être utilisée avec le Pare-feu Azure ? |
| --- | -------- |:---:|:---:|:---:|
| AzureEventGrid | Azure Event Grid. | Les deux | Non | Non |


## <a name="ip-firewall"></a>Pare-feu IP 
Azure Event Grid prend en charge les contrôles d’accès basés sur IP pour la publication sur des rubriques et des domaines. Les contrôles basés sur IP vous permettent de limiter les serveurs de publication à une rubrique ou un domaine sur un ensemble d’ordinateurs et de services cloud approuvés. Cette fonctionnalité complète les [mécanismes d’authentification](security-authentication.md) pris en charge par Event Grid.

Par défaut, la rubrique et le domaine sont accessibles sur Internet tant que la demande s’accompagne d’une authentification et d’une autorisation valides. Avec le pare-feu IP, vous pouvez les limiter à un ensemble d’adresses IP ou de plages d’adresses IP dans la notation [CIDR (Classless InterDomain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). Les serveurs de publication provenant de toute autre adresse IP seront rejetés et recevront une réponse 403 (interdit).

Pour obtenir des instructions pas à pas sur la configuration du pare-feu IP pour les rubriques et les domaines, consultez [Configurer le pare-feu IP](configure-firewall.md).

## <a name="private-endpoints"></a>Instances Private Endpoint
Vous pouvez utiliser des [points de terminaison privés](../private-link/private-endpoint-overview.md) pour autoriser l’entrée sécurisée d’événements directement à partir de votre réseau virtuel vers vos rubriques et domaines via une [liaison privée](../private-link/private-link-overview.md), sans passer par le réseau Internet public. Un point de terminaison privé est une interface réseau spéciale pour un service Azure dans votre réseau virtuel. Lorsque vous créez un point de terminaison privé pour votre rubrique ou domaine, il offre une connectivité sécurisée entre les clients sur votre réseau virtuel et votre ressource Event Grid. Une adresse IP est attribuée au point de terminaison privé à partir de la plage d’adresses IP de votre réseau virtuel. La connexion entre le point de terminaison privé et le service Event Grid utilise une liaison privée.

![Diagramme de l'architecture](./media/network-security/architecture-diagram.png)

L’utilisation de points de terminaison privés pour ressource Event Grid vous permet d’effectuer les opérations suivantes :

- Sécuriser l’accès à votre rubrique ou domaine à partir d’un réseau virtuel via un réseau principal Microsoft, par opposition au réseau Internet public.
- Se connecter en toute sécurité à partir de réseaux locaux qui se connectent au réseau virtuel à l'aide d'un VPN ou d'itinéraires ExpressRoute avec le peering privé.

Quand vous créez un point de terminaison privé pour une rubrique ou un domaine dans votre réseau virtuel, une demande de consentement est envoyée pour approbation au propriétaire de la ressource. Si l’utilisateur qui demande la création du point de terminaison privé est également propriétaire de la ressource, cette demande de consentement est automatiquement approuvée. Sinon, la connexion affiche l’état **En attente** jusqu’à ce qu’elle soit approuvée. Les applications du réseau virtuel peuvent se connecter en toute transparence au service Event Grid sur le point de terminaison privé, à l’aide des mêmes chaînes de connexion et mécanismes d’autorisation qu’ils utiliseraient dans tous les cas. Les propriétaires de la ressource peuvent gérer les demandes de consentement et les points de terminaison privés, via l’onglet **Points de terminaison privés** de la ressource dans le portail Azure.

### <a name="connect-to-private-endpoints"></a>Se connecter à des points de terminaison privés
Les serveurs de publication sur un réseau virtuel utilisant le point de terminaison privé doivent utiliser la même chaîne de connexion pour la rubrique ou le domaine que les clients se connectant au point de terminaison public. La résolution DNS achemine automatiquement les connexions du réseau virtuel vers la rubrique ou le domaine via une liaison privée. Event Grid crée une [zone DNS privée](../dns/private-dns-overview.md) attachée au réseau virtuel avec la mise à jour nécessaire pour les points de terminaison privés, par défaut. Toutefois, si vous utilisez votre propre serveur DNS, vous devrez peut-être apporter des modifications supplémentaires à votre configuration DNS.

### <a name="dns-changes-for-private-endpoints"></a>Modifications DNS pour les points de terminaison privés
Quand vous créez un point de terminaison privé, l’enregistrement DNS CNAME pour la ressource est mis à jour avec un alias dans un sous-domaine avec le préfixe `privatelink`. Par défaut, une zone DNS privée correspondant au sous-domaine de la liaison privée est créée. 

Lorsque vous résolvez l’URL du point de terminaison de la rubrique ou du domaine à l’extérieur du réseau virtuel avec le point de terminaison privé, elle correspond au point de terminaison public du service. Les enregistrements de ressources DNS pour 'topicA', lorsqu’ils sont résolus depuis l’**extérieur du réseau virtuel** hébergeant le point de terminaison privé, sont les suivants :

| Nom                                          | Type      | Valeur                                         |
| --------------------------------------------- | ----------| --------------------------------------------- |  
| `topicA.westus.eventgrid.azure.net`             | CNAME     | `topicA.westus.privatelink.eventgrid.azure.net` |
| `topicA.westus.privatelink.eventgrid.azure.net` | CNAME     | \<Azure traffic manager profile\>

Vous pouvez refuser ou contrôler l’accès pour un client en dehors du réseau virtuel via le point de terminaison public à l’aide du [pare-feu ID](#ip-firewall). 

En cas de résolution à partir du réseau virtuel hébergeant le point de terminaison privé, l’URL du point de terminaison de la rubrique ou du domaine correspond à l’adresse IP du point de terminaison privé. Les enregistrements de ressources DNS pour la rubrique 'topicA', lorsqu’ils sont résolus par un client **depuis le réseau virtuel** hébergeant le point de terminaison privé, sont les suivants :

| Nom                                          | Type      | Valeur                                         |
| --------------------------------------------- | ----------| --------------------------------------------- |  
| `topicA.westus.eventgrid.azure.net`             | CNAME     | `topicA.westus.privatelink.eventgrid.azure.net` |
| `topicA.westus.privatelink.eventgrid.azure.net` | Un         | 10.0.0.5

Cette approche permet d’accéder à la rubrique ou au domaine avec la même chaîne de connexion pour les clients sur le réseau virtuel hébergeant les points de terminaison privés, ainsi que des clients en dehors du réseau virtuel.

Si vous utilisez un serveur DNS personnalisé sur votre réseau, les clients peuvent résoudre le nom de domaine complet du point de terminaison de la rubrique ou du domaine vers l’adresse IP du point de terminaison privé. Configurez votre serveur DNS pour déléguer votre sous-domaine de liaison privée à la zone DNS privée du réseau virtuel, ou configurer les enregistrements A pour `topicOrDomainName.regionName.privatelink.eventgrid.azure.net` avec l’adresse IP du point de terminaison privé.

Le nom recommandé pour la zone DNS est `privatelink.eventgrid.azure.net`.

### <a name="private-endpoints-and-publishing"></a>Points de terminaison privés et publication

Le tableau suivant décrit les différents états de connexion au point de terminaison privé et les effets sur la publication :

| État de la connexion   |  Publication réussie (oui/non) |
| ------------------ | -------------------------------|
| Approved           | Oui                            |
| Rejeté           | Non                             |
| Pending            | Non                             |
| Déconnecté       | Non                             |

Pour réussir une publication, l’état de la connexion au point de terminaison privé doit être **approuvé**. Si une connexion est rejetée, elle ne peut pas être approuvée à l’aide du portail Azure. La seule possibilité consiste à supprimer la connexion et à en créer une nouvelle à la place.

## <a name="pricing-and-quotas"></a>Tarifs et quotas
La fonctionnalité **Points de terminaison privés** est disponible dans les niveaux De base et Premium d’Event Grid. Event Grid permet de créer jusqu’à 64 connexions de points de terminaison privés par rubrique ou domaine. 

La fonctionnalité **Pare-feu IP** est disponible dans les niveaux De base et Premium d’Event Grid. Nous autorisons la création d’un maximum de 16 règles de pare-feu IP par rubrique ou domaine.

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez configurer le pare-feu IP de votre ressource Event Grid pour limiter l’accès au réseau Internet public à un ensemble spécifique d’adresses IP ou de plages d’adresses IP. Pour obtenir des instructions pas à pas, consultez [Configurer le pare-feu IP](configure-firewall.md).

Vous pouvez configurer des points de terminaison privés pour limiter l’accès exclusivement aux réseaux virtuels sélectionnés. Pour obtenir des instructions pas à pas, consultez [Configurer des points de terminaison privés](configure-private-endpoints.md).

Pour résoudre les problèmes de connectivité réseau, consultez [Résoudre les problèmes de connectivité réseau](troubleshoot-network-connectivity.md)