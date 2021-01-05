---
title: Questions fréquentes (FAQ) sur Azure Network Watcher | Microsoft Docs
description: Cet article répond aux questions fréquentes sur le service Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
manager: ''
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/10/2019
ms.author: damendo
ms.openlocfilehash: 959062d493d9eb47204be2488f216b70804b3605
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965763"
---
# <a name="frequently-asked-questions-faq-about-azure-network-watcher"></a>Questions fréquentes (FAQ) sur Azure Network Watcher
Le service [Azure Network Watcher](./network-watcher-monitoring-overview.md) offre une suite d’outils permettant de superviser, diagnostiquer, consulter des métriques, ainsi qu’activer et désactiver des journaux pour les ressources d’un réseau virtuel Azure. Cet article répond aux questions courantes sur le service.

## <a name="general"></a>Général

### <a name="what-is-network-watcher"></a>Qu’est-ce que Network Watcher ?
Network Watcher est conçu pour superviser et réparer l’intégrité réseau des composants IaaS (infrastructure en tant que service), notamment les machines virtuelles, les réseaux virtuels, les passerelles d’application, les équilibreurs de charge et d’autres ressources figurant dans un réseau virtuel Azure. Il ne s’agit pas d’une solution pour superviser l’infrastructure PaaS (plateforme en tant que service) ou pour obtenir une analytique web/mobile.

### <a name="what-tools-does-network-watcher-provide"></a>Quels sont les outils fournis par Network Watcher ?
Network Watcher propose trois principaux ensembles de fonctionnalités
* Surveillance
  * La [vue Topologie](./view-network-topology.md) vous montre les ressources figurant dans votre réseau virtuel et les relations entre ces ressources.
  * Le [moniteur de connexion](./connection-monitor.md) vous permet de surveiller la connectivité et la latence entre une machine virtuelle et une autre ressource réseau.
  * L’[Analyseur de performances réseau](../azure-monitor/insights/network-performance-monitor.md) vous permet de surveiller la connectivité et les latences dans les architectures réseau hybrides, les circuits Expressroute et les points de terminaison de service/application.  
* Diagnostics
  * La [vérification des flux IP](./network-watcher-ip-flow-verify-overview.md) vous permet de détecter les problèmes de filtrage du trafic au niveau d’une machine virtuelle.
  * Le [tronçon suivant](./network-watcher-next-hop-overview.md) vous aide à vérifier les itinéraires de trafic et à détecter les problèmes de routage.
  * La [résolution des problèmes de connexion](./network-watcher-connectivity-portal.md) permet une connexion ponctuelle et la vérification de la latence entre une machine virtuelle et une autre ressource réseau.
  * La [capture de paquets](./network-watcher-packet-capture-overview.md) vous permet de capturer tout le trafic sur une machine virtuelle dans votre réseau virtuel.
  * La [résolution des problèmes liés au VPN](./network-watcher-troubleshoot-overview.md) exécute plusieurs vérifications de diagnostic sur vos connexions et passerelles VPN pour aider à résoudre les problèmes.
* Journalisation
  * Les [journaux de flux NSG](./network-watcher-nsg-flow-logging-overview.md) vous permettent de journaliser tout le trafic dans vos [groupes de sécurité réseau (groupes NSG)](../virtual-network/network-security-groups-overview.md)
  * [Traffic Analytics ](./traffic-analytics.md) traite les données de vos journaux de flux NSG, ce qui vous permet de visualiser, d’interroger, d’analyser et de comprendre votre trafic réseau.


Pour plus d’informations, consultez la [page de présentation de Network Watcher](./network-watcher-monitoring-overview.md).


### <a name="how-does-network-watcher-pricing-work"></a>Comment fonctionne la tarification de Network Watcher ?
Visitez la page [Tarification](https://azure.microsoft.com/pricing/details/network-watcher/) pour obtenir le détail des composants Network Watcher et de leur tarification.

### <a name="which-regions-is-network-watcher-supportedavailable-in"></a>Dans quelles régions Network Watcher est-il pris en charge/disponible ?
Vous pouvez consulter la disponibilité régionale la plus récente sur la [page de disponibilité des services Azure](https://azure.microsoft.com/global-infrastructure/services/?products=network-watcher)

### <a name="which-permissions-are-needed-to-use-network-watcher"></a>Quelles sont les autorisations nécessaires pour utiliser Network Watcher ?
Consultez la liste des [Autorisations Azure RBAC requises pour utiliser Network Watcher](./required-rbac-permissions.md). Pour déployer des ressources, vous devez disposer d’autorisations de contributeur sur NetworkWatcherRG (voir ci-dessous).

### <a name="how-do-i-enable-network-watcher"></a>Comment faire pour activer Network Watcher ?
Le service Network Watcher est [activé automatiquement](https://azure.microsoft.com/updates/azure-network-watcher-will-be-enabled-by-default-for-subscriptions-containing-virtual-networks/) pour chaque abonnement.

### <a name="what-is-the-network-watcher-deployment-model"></a>Qu’est-ce que le modèle de déploiement Network Watcher ?
La ressource parente Network Watcher est déployée avec une instance unique sur chaque région. Format de nommage : NetworkWatcher_NomRégion. Exemple : NetworkWatcher_centralus est la ressource Network Watcher pour la région « USA Centre ».

### <a name="what-is-the-networkwatcherrg"></a>Qu’est-ce que NetworkWatcherRG ?
Les ressources Network Watcher se trouvent dans le groupe de ressources **NetworkWatcherRG** masqué, qui est créé automatiquement. Par exemple, la ressource Journaux de flux NSG est une ressource enfant de Network Watcher, qui est activée dans NetworkWatcherRG.

### <a name="why-do-i-need-to-install-the-network-watcher-extension"></a>Pourquoi dois-je installer l’extension Network Watcher ? 
L’extension Network Watcher est requise pour toute fonctionnalité qui doit générer ou intercepter le trafic à partir d’une machine virtuelle. 

### <a name="which-features-require-the-network-watcher-extension"></a>Quelles fonctionnalités nécessitent l’extension Network Watcher ?
Les fonctionnalités Capture de paquets, Résolution des problèmes de connexion et Moniteur de connexion nécessitent la présence de l’extension Network Watcher.

### <a name="what-are-resource-limits-on-network-watcher"></a>Quelles sont les limites des ressources sur Network Watcher ?
Consultez la page [Limites du service](../azure-resource-manager/management/azure-subscription-service-limits.md#network-watcher-limits) pour connaître toutes les limites.  

### <a name="why-is-only-one-instance-of-network-watcher-allowed-per-region"></a>Pourquoi une seule instance de Network Watcher est-elle autorisée par région ? 
Network Watcher n’a besoin d’être activé qu’une seule fois au niveau d’un abonnement pour que ses fonctionnalités soient utilisables ; il ne s’agit pas d’une limite de service.

### <a name="how-can-i-manage-the-network-watcher-resource"></a>Comment puis-je gérer la ressource Network Watcher ? 
La ressource Network Watcher représente le service back-end pour Network Watcher et est entièrement gérée par Azure. Les clients n’ont pas besoin de la gérer. Les opérations telles que le déplacement ne sont pas prises en charge sur la ressource. Toutefois, [la ressource peut être supprimée](./network-watcher-create.md#delete-a-network-watcher-in-the-portal). 

## <a name="service-availability-and-redundancy"></a>Disponibilité et redondance du service 

### <a name="is-the-network-watcher-service-zone-resilient"></a>Le service Network Watcher est-il résilient aux zones ? 
Oui. Le service Network Watcher est résilient aux zones par défaut. 

### <a name="how-do-i-configure-the-network-watcher-service-to-be-zone-resilient"></a>Comment configurer le service Network Watcher pour qu’il soit résilient aux zones ? 
Aucune configuration client n’est nécessaire pour activer la résilience des zones. La résilience des zones pour les ressources Network Watcher est disponible par défaut et gérée par le service lui-même. 

## <a name="nsg-flow-logs"></a>Journaux de flux NSG

### <a name="what-does-nsg-flow-logs-do"></a>Que font les journaux de flux NSG ?
Les ressources réseau Azure peuvent être combinées et gérées via les [groupes de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md). Les journaux de flux NSG vous permettent de consigner les informations de flux à 5 tuples concernant tout le trafic via vos groupes de sécurité réseau. Les journaux de flux bruts sont écrits dans un compte de stockage Azure à partir duquel ils peuvent être traités, analysés, interrogés ou exportés en fonction des besoins.

### <a name="how-do-i-use-nsg-flow-logs-with-a-storage-account-behind-a-firewall"></a>Comment utiliser des journaux de flux NSG sur un compte de stockage derrière un pare-feu ?

Pour utiliser un compte de stockage derrière un pare-feu, vous devez fournir une exception aux services Microsoft approuvés pour accéder à votre compte de stockage :

* Accédez au compte de stockage en tapant son nom dans la recherche globale sur le portail ou à partir de la [page Comptes de stockage](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Storage%2FStorageAccounts)
* Dans la section **PARAMÈTRES**, sélectionnez **Pare-feu et réseaux virtuels**.
* Dans « Autoriser l’accès depuis », sélectionnez **Réseaux sélectionnés**. Ensuite, sous **Exceptions**, cochez la case en regard de **« Autoriser les services Microsoft approuvés à accéder à ce compte de stockage »** 
* Si cette option est déjà sélectionnée, aucune modification n’est nécessaire.  
* Localisez votre groupe de sécurité réseau cible dans la [page de vue d’ensemble des journaux de flux NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) et activez les journaux de flux NSG avec le compte de stockage ci-dessus sélectionné.

Vous pouvez consulter les journaux de stockage au bout de quelques minutes. Vous devriez y voir qu’un horodateur a été mis à jour ou qu’un fichier JSON a été créé.

### <a name="how-do-i-use-nsg-flow-logs-with-a-storage-account-behind-a-service-endpoint"></a>Comment utiliser des journaux de flux NSG sur un compte de stockage derrière un point de terminaison de service ?

Les journaux de flux NSG sont compatibles avec les points de terminaison de service et ne nécessitent aucune configuration supplémentaire. Consultez le [tutoriel sur l’activation des points de terminaison de service](../virtual-network/tutorial-restrict-network-access-to-resources.md#enable-a-service-endpoint) dans votre réseau virtuel.


### <a name="what-is-the-difference-between-flow-logs-versions-1--2"></a>Quelle est la différence entre les versions 1 et 2 des journaux de flux ?
La version 2 des journaux de flux introduit le concept d’*état de flux* et stocke des informations sur les octets et les paquets transmis. [En savoir plus](./network-watcher-nsg-flow-logging-overview.md#log-format).

## <a name="next-steps"></a>Étapes suivantes
 - Consultez notre [page de présentation de la documentation](./index.yml) pour accéder à des tutoriels qui vous aideront à bien démarrer avec Network Watcher.