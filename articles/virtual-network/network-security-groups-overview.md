---
title: Vue d’ensemble des groupes de sécurité réseau Azure
titlesuffix: Azure Virtual Network
description: Apprenez-en davantage sur les groupes de sécurité réseau. Les groupes de sécurité réseau vous permettent de filtrer le trafic réseau entre les ressources Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: kumud
ms.reviewer: kumud
ms.custom: contperf-fy21q1
ms.openlocfilehash: 4e23c6f25145724a5300c9e5cdcb55431fb0b4f2
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028765"
---
# <a name="network-security-groups"></a>Groupes de sécurité réseau
<a name="network-security-groups"></a>

Vous pouvez utiliser un groupe de sécurité réseau Azure pour filtrer le trafic réseau à destination et en provenance des ressources Azure dans un réseau virtuel Azure. Un groupe de sécurité réseau contient des [règles de sécurité](#security-rules) qui autorisent ou rejettent le trafic réseau entrant et sortant vers différents types de ressources Azure. Pour chaque règle, vous pouvez spécifier la source et la destination, le port et le protocole.

Cet article décrit les propriétés d’une règle de groupe de sécurité réseau, les [règles de sécurité par défaut](#default-security-rules) appliquées et les propriétés de règle que vous pouvez modifier pour créer une [règle de sécurité augmentée](#augmented-security-rules).

## <a name="security-rules"></a><a name="security-rules"></a> Règles de sécurité

Un groupe de sécurité réseau contient le nombre des règles souhaité (ou aucune), dans les [limites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits) de l’abonnement Azure. Chaque règle spécifie les propriétés suivantes :

|Propriété  |Explication  |
|---------|---------|
|Nom|Nom unique au sein du groupe de sécurité réseau.|
|Priority | Nombre compris entre 100 et 4096. Les règles sont traitées dans l’ordre croissant, car les nombres les plus faibles sont prioritaires. Une fois que le trafic correspond à une règle, le traitement s’arrête. Par conséquent, les règles avec des priorités plus faibles (des nombres plus élevés) et ayant les mêmes attributs que les règles de priorité supérieure ne sont pas traitées.|
|Source ou destination| Une adresse IP, un bloc de routage CIDR (par exemple, 10.0.0.0/24), une balise de service ou un groupe de sécurité d’application. Si vous spécifiez une adresse pour une ressource Azure, spécifiez l’adresse IP privée assignée à la ressource. Les groupes de sécurité réseau sont traités une fois qu’Azure a converti une adresse IP publique en adresse IP privée pour le trafic entrant, et avant qu’Azure ne convertisse une adresse IP privée en une adresse IP publique pour le trafic sortant. . En spécifiant une plage, une balise de service ou un groupe de sécurité d’application, vous pouvez créer moins des règles de sécurité. La possibilité de spécifier plusieurs adresses IP individuelles et plages (vous ne pouvez pas spécifier plusieurs balises de service ou groupes d’applications) dans une règle est désignée sous le nom de [règles de sécurité augmentée](#augmented-security-rules). Les règles de sécurité augmentée peuvent uniquement être créées dans des groupes de sécurité réseau créés par le biais du modèle de déploiement du Gestionnaire de ressources. Vous ne pouvez pas spécifier plusieurs adresses IP et plages d’adresses IP dans les groupes de sécurité réseau créés par le biais du modèle de déploiement classique.|
|Protocol     | TCP, UDP, ICMP ou n’importe lequel.|
|Sens| Indique si la règle s’applique au trafic entrant ou sortant.|
|Plage de ports     |Vous pouvez spécifier un port individuel ou une plage de ports. Par exemple, indiquez 80 ou 10000-10005. La spécification de plages vous permet de créer moins de règles de sécurité. Les règles de sécurité augmentée peuvent uniquement être créées dans des groupes de sécurité réseau créés par le biais du modèle de déploiement du Gestionnaire de ressources. Vous ne pouvez pas spécifier plusieurs ports ou plages de ports dans les groupes de sécurité réseau créés par le biais du modèle de déploiement classique.   |
|Action     | Autoriser ou refuser        |

Les règles de sécurité des groupes de sécurité réseau sont évaluées par priorité selon un tuple à 5 éléments (source, port source, destination, port de destination et protocole) pour autoriser ou refuser le trafic. Vous ne pouvez pas créer deux règles de sécurité avec les mêmes priorité et direction. Un enregistrement de flux est créé pour les connexions existantes. La communication est autorisée ou refusée en fonction de l’état de connexion de l’enregistrement de flux. L’enregistrement de flux permet d’obtenir un groupe de sécurité réseau avec état. Si vous spécifiez une règle de sécurité sortante vers n’importe quelle adresse sur le port 80, par exemple, il n’est pas nécessaire d’indiquer une règle de sécurité entrante pour la réponse au trafic sortant. Vous devez uniquement spécifier une règle de sécurité entrante si la communication est établie en externe. Le contraire est également vrai. Si le trafic entrant est autorisé sur un port, il n’est pas nécessaire de spécifier une règle de sécurité sortante pour répondre au trafic sur ce port.

Les connexions existantes ne peuvent pas être interrompues quand vous supprimez une règle de sécurité ayant activé le flux. Les flux de trafic sont interrompus quand les connexions sont arrêtées et qu’aucun trafic ne transite dans un sens ou dans l’autre pendant au moins quelques minutes.

Le nombre de règles de sécurité que vous pouvez créer dans un groupe de sécurité réseau est limité. Pour plus d’informations, consultez [limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

### <a name="default-security-rules"></a><a name="default-security-rules"></a> Règles de sécurité par défaut

Azure crée les règles par défaut suivantes dans chaque groupe de sécurité réseau que vous créez :

#### <a name="inbound"></a>Trafic entrant

##### <a name="allowvnetinbound"></a>AllowVNetInBound

|Priority|Source|Ports source|Destination|Ports de destination|Protocol|Accès|
|---|---|---|---|---|---|---|
|65 000|VirtualNetwork|0-65535|VirtualNetwork|0-65535|Quelconque|Allow|

##### <a name="allowazureloadbalancerinbound"></a>AllowAzureLoadBalancerInBound

|Priority|Source|Ports source|Destination|Ports de destination|Protocol|Accès|
|---|---|---|---|---|---|---|
|65 001|AzureLoadBalancer|0-65535|0.0.0.0/0|0-65535|Quelconque|Allow|

##### <a name="denyallinbound"></a>DenyAllInbound

|Priority|Source|Ports source|Destination|Ports de destination|Protocol|Accès|
|---|---|---|---|---|---|---|
|65 500|0.0.0.0/0|0-65535|0.0.0.0/0|0-65535|Quelconque|Deny|

#### <a name="outbound"></a>Règle de trafic sortant

##### <a name="allowvnetoutbound"></a>AllowVnetOutBound

|Priority|Source|Ports source| Destination | Ports de destination | Protocol | Accès |
|---|---|---|---|---|---|---|
| 65 000 | VirtualNetwork | 0-65535 | VirtualNetwork | 0-65535 | Quelconque | Allow |

##### <a name="allowinternetoutbound"></a>AllowInternetOutBound

|Priority|Source|Ports source| Destination | Ports de destination | Protocol | Accès |
|---|---|---|---|---|---|---|
| 65 001 | 0.0.0.0/0 | 0-65535 | Internet | 0-65535 | Quelconque | Allow |

##### <a name="denyalloutbound"></a>DenyAllOutBound

|Priority|Source|Ports source| Destination | Ports de destination | Protocol | Accès |
|---|---|---|---|---|---|---|
| 65 500 | 0.0.0.0/0 | 0-65535 | 0.0.0.0/0 | 0-65535 | Quelconque | Deny |

Dans les colonnes **Source** et **Destination**, *VirtualNetwork*, *AzureLoadBalancer* et *Internet* sont des [balises de service](service-tags-overview.md), non des adresses IP. Dans la colonne de protocole, **Any** (N’importe lequel) englobe TCP, UDP et ICMP. Lorsque vous créez une règle, vous pouvez spécifier TCP, UDP, ICMP ou Any (N’importe lequel). *0.0.0.0/0* dans les colonnes **Source** et **Destination** représente toutes les adresses. Les clients comme le portail Azure, Azure CLI ou PowerShell peuvent utiliser « * » ou « any » pour cette expression.
 
Vous ne pouvez pas supprimer les règles par défaut, mais vous pouvez les remplacer par des règles de priorité plus élevée.

### <a name="augmented-security-rules"></a><a name="augmented-security-rules"></a> Règles de sécurité augmentée

Les règles de sécurité augmentée simplifient la définition de la sécurité pour les réseaux virtuels, ce qui vous permet de définir des stratégies de sécurité réseau plus vastes et plus complexes, avec moins de règles. Vous pouvez combiner plusieurs ports ainsi que des adresses ou plages d’adresses IP explicites dans une seule règle de sécurité facilement compréhensible. Utiliser des règles de sécurité augmentée dans les champs de la source, de la destination et du port d’une règle. Pour simplifier la maintenance de votre définition de règle de sécurité, combinez des règles de sécurité augmentée avec des [balises de service](service-tags-overview.md) ou des [groupes de sécurité d’application](#application-security-groups). Le nombre d’adresses, les plages et les ports que vous pouvez spécifier dans une règle sont limités. Pour plus d’informations, consultez [limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

#### <a name="service-tags"></a>Balises de service

Une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné. Elle permet de minimiser la complexité des mises à jour fréquentes des règles de sécurité réseau.

Pour plus d’informations, consultez [Balises de Service Azure](service-tags-overview.md). Pour obtenir un exemple d’utilisation de la balise de service de stockage pour limiter l’accès réseau, consultez [Limiter l’accès réseau aux ressources PaaS](tutorial-restrict-network-access-to-resources.md).

#### <a name="application-security-groups"></a>Groupes de sécurité d’application

Les groupes de sécurité d’application permettent de configurer la sécurité réseau comme un prolongement naturel de la structure de l’application, et donc de regrouper les machines virtuelles et définir des stratégies de sécurité réseau basés sur ces groupes. Vous pouvez réutiliser votre stratégie de sécurité à grande échelle sans maintenance manuelle d’adresses IP explicites. Pour en savoir plus, consultez [Groupes de sécurité d’application](application-security-groups.md).

## <a name="azure-platform-considerations"></a>Considérations relatives à la plateforme Azure

- **Adresse IP virtuelle du nœud hôte** : Des services d’infrastructure de base tels que DHCP, DNS, IMDS et l’analyse de l’intégrité sont fournis par le biais des adresses IP d’hôte virtualisées 168.63.129.16 et 169.254.169.254. Ces adresses IP appartiennent à Microsoft et sont les seules adresses IP virtualisées utilisées à cet effet dans toutes les régions. Les règles de sécurité en vigueur et les itinéraires effectifs n’incluent pas ces règles de plateforme. Pour remplacer cette communication d’infrastructure de base, vous pouvez créer une règle de sécurité pour refuser le trafic en utilisant les [balises de service](service-tags-overview.md) suivantes sur vos règles de groupe de sécurité réseau : AzurePlatformDNS, AzurePlatformIMDS, AzurePlatformLKM. Découvrez comment [diagnostiquer le filtrage de trafic réseau](diagnose-network-traffic-filter-problem.md) et [diagnostiquer le routage réseau](diagnose-network-routing-problem.md).
- **Gestion des licences (Service de gestion des clés)** : Les images Windows en cours d’exécution sur les machines virtuelles doivent être acquises sous licence. Pour assurer la gestion des licences, une requête est envoyée aux serveurs hôtes du Service de gestion de clés qui gèrent les requêtes de ce type. La requête est effectuée en sortie par le biais du port 1688. Cette règle de plateforme est désactivée pour les déploiements utilisant la configuration [itinéraire par défaut 0.0.0.0/0](virtual-networks-udr-overview.md#default-route).
- **Machines virtuelles dans des pools d’équilibrage de charge** : Le port source et la plage d’adresses appliquées proviennent de l’ordinateur d’origine, pas de l’équilibreur de charge. La plage de ports et d’adresses de destination sont ceux de l’ordinateur de destination, et non de l’équilibreur de charge.
- **Instances de service Azure** : Les instances de plusieurs services Azure, tels que HDInsight, les environnements de service d’application et Virtual Machine Scale Sets, sont déployées dans des sous-réseaux du réseau virtuel. Pour obtenir la liste complète des services que vous pouvez déployer sur des réseaux virtuels, consultez [Réseau virtuel pour les services Azure](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network). Familiarisez-vous avec les exigences de port pour chaque service avant d’appliquer un groupe de sécurité réseau au sous-réseau dans lequel la ressource est déployée. Si vous refusez les ports requis par le service, ce dernier ne fonctionnera pas correctement.
- **Envoi d’e-mail sortant** : Microsoft vous recommande l’utilisation de services de relais authentifiés SMTP (généralement connectés via le port TCP 587, mais parfois via d’autres ports) pour envoyer un e-mail depuis des machines virtuelles Azure. Les services de relais SMTP se spécialisent dans la réputation des expéditeurs, afin de minimiser les risques de renvoi de messages de la part de fournisseurs de messagerie tiers. Ce type de services de relais SMTP incluent, sans s’y limiter, Exchange Online Protection et SendGrid. L’utilisation de services de relais SMTP n’est en aucun cas limitée dans Azure et ne tient pas compte de votre type d’abonnement. 

  Si vous avez créé votre abonnement Azure avant le 15 novembre 2017, vous pouvez, en plus d’utiliser des services de relais SMTP, envoyer des messages électroniques via le port TCP 25 directement. Si vous avez créé votre abonnement après le 15 novembre 2017, vous ne serez peut-être pas en mesure d’envoyer des messages électroniques via le port TCP 25 directement. Le comportement des communications sortantes via le port 25 dépend de votre type d’abonnement :

     - **Contrat Entreprise** : Les communications sortantes via le port 25 sont autorisées. Vous pouvez envoyer du courrier sortant directement depuis des machines virtuelles vers des fournisseurs de messagerie électronique externes, sans restrictions de la plateforme Azure. 
     - **Paiement à l’utilisation** : Les communications sortantes via le port 25 sont bloquées pour toutes les ressources. Si vous devez envoyer des courriers électroniques directement depuis une machine virtuelle vers des fournisseurs de messagerie électronique externes (sans utiliser des relais SMTP authentifiés), vous pouvez effectuer une requête pour retirer la restriction. Les demandes sont étudiées et acceptées par Microsoft. Elles ne sont accordées qu’après des vérifications antifraude. Pour effectuer une requête, ouvrez un cas d’assistance avec pour type de problème *Technique*, *Connectivité du réseau virtuel*, *Envoi de messages électroniques impossible (SMTP/Port 25)* . Dans votre cas d’assistance, indiquez les raisons pour lesquelles votre abonnement doit être en mesure d’envoyer des courriers électroniques directement aux fournisseurs, sans passer par un relais authentifié SMTP. Si votre abonnement est exempté, seules les machines virtuelles créées après la date d’exemption sont en mesure d’utiliser des communications sortantes via le port 25.
     - **MSDN, Pass Azure, Azure dans Open, Éducation, BizSpark et Essai gratuit** : Les communications sortantes via le port 25 sont bloquées pour toutes les ressources. Aucune demande pour retirer la restriction ne peut être faite. Elles ne sont pas autorisées. Si vous devez envoyer des courriers électroniques depuis votre machine virtuelle, vous devez utiliser un service de relais SMTP.
     - **Fournisseur de services cloud** : Les clients qui consomment des ressources Azure via un fournisseur de services cloud peuvent créer une demande de support auprès de celui-ci et demander qu’il crée une demande de déblocage en son nom, si un relais SMTP sécurisé ne peut pas être utilisé.

  Si Azure vous permet d’envoyer des courriers électroniques via le port 25, Microsoft ne peut vous garantir que les fournisseurs de messagerie électronique accepteront le message entrant en provenance de votre machine virtuelle. Si un fournisseur spécifique rejette les messages en provenance de votre machine virtuelle, contactez directement le fournisseur pour résoudre tout problème de livraison de messages ou de filtrage de spam, ou bien utilisez un service de relais SMTP authentifié.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les ressources Azure pouvant être déployées dans un réseau virtuel et auxquelles des groupes de sécurité réseau peuvent être associées, consultez [Intégration d’un réseau virtuel pour les services Azure](virtual-network-for-azure-services.md).
* Pour savoir comment le trafic est évalué avec les groupes de sécurité réseau, consultez [À propos des groupes de sécurité réseau](network-security-group-how-it-works.md).
* Si vous n’avez jamais créé un groupe de sécurité réseau, vous pouvez suivre un [didacticiel](tutorial-filter-network-traffic.md) rapide pour vous entraîner.
* Si vous êtes familier avec les groupes de sécurité réseau et que vous devez en gérer un, consultez [Gérer un groupe de sécurité réseau](manage-network-security-group.md). 
* Si vous rencontrez des problèmes de communication et que vous avez besoin résoudre les problèmes de groupes de sécurité réseau, consultez [Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle](diagnose-network-traffic-filter-problem.md). 
* Découvrez comment activer les [journaux de flux de groupe de sécurité réseau](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour analyser le trafic réseau vers et à partir des ressources auxquelles un groupe de sécurité réseau est associé.
