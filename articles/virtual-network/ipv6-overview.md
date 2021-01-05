---
title: Vue d’ensemble du protocole IPv6 pour réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Description des points de terminaison et des chemins de données IPv6 dans un réseau virtuel Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: c0d4c72bd60f35088773ec6dc5df80709c5ae98c
ms.sourcegitcommit: 67e8e1caa8427c1d78f6426c70bf8339a8b4e01d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91666821"
---
# <a name="what-is-ipv6-for-azure-virtual-network"></a>Qu’est-ce que le protocole IPv6 pour réseau virtuel Azure ?

Le protocole IPv6 pour réseau virtuel Azure vous permet d’héberger des applications dans Azure avec une connectivité IPv6 et IPv4 tant au sein d’un réseau virtuel que vers et depuis Internet. En raison de l’épuisement des adresses IPv4 publiques, les nouveaux réseaux pour la mobilité et l’Internet des objets (IoT) reposent souvent sur le protocole IPv6. Même des fournisseurs de services Internet et réseaux mobiles établis de longue date sont en cours d’évolution vers le protocole IPv6. Des services uniquement IPv4 peuvent se retrouver réellement désavantagés dans les marchés existants et émergents. La connectivité double pile IPv4/IPv6 permet aux services hébergés dans Azure de combler ce fossé technologique avec des services à double pile disponibles dans le monde entier, qui se connectent sans difficulté tant aux appareils et réseaux IPv4 existants qu’aux nouveaux appareils et réseaux IPv6.

La connectivité IPv6 d’origine d’Azure facilite la fourniture d’une connectivité Internet double pile (IPv4/IPv6) pour les applications hébergées dans Azure. Cela permet un déploiement simple des machines virtuelles avec une connectivité IPv6 à charge équilibrée pour les connexions tant entrantes que sortantes. Cette fonctionnalité est toujours disponible et des informations supplémentaires sont accessibles [ici](../load-balancer/load-balancer-ipv6-overview.md).
Le protocole IPv6 pour réseau virtuel Azure est beaucoup plus complet et permet le déploiement d’architectures de solution IPv6 complètes dans Azure.


Le diagramme suivant illustre un déploiement double pile (IPv4/IPv6) dans Azure :

![Diagramme de déploiement de réseau IPv6](./media/ipv6-support-overview/ipv6-sample-diagram.png)

## <a name="benefits"></a>Avantages

Avantages d’IPv6 pour un réseau virtuel Azure :

- Vous aide à étendre vos applications hébergées dans Azure aux marchés en pleine expansion des solutions mobiles et de l’Internet des objets.
- Les machines virtuelles double pile IPv4/IPv6 offrent une flexibilité de déploiement de service maximale. Une simple instance de service peut se connecter indifféremment à des clients Internet compatibles IPv4 et IPv6.
- S’appuie sur la connectivité IPv6 à Internet stable et établie de longue date des machines virtuelles Azure.
- Sécurisation par défaut dans la mesure où la connectivité IPv6 à Internet n’est établie que lorsque vous le demandez explicitement dans votre déploiement.

## <a name="capabilities"></a>Fonctionnalités

IPv6 pour les réseaux virtuels Azure inclut les fonctionnalités suivantes :

- Les clients Azure peuvent définir leur propre espace d’adressage de réseau virtuel IPv6 pour répondre aux besoins de leurs applications et clients, ou intégrer sans difficulté dans leur propre espace d’adressage IP local.
- Des réseaux virtuels double pile (IPv4 et IPv6) avec des sous-réseaux double pile permettent aux applications de se connecter à des ressources tant IPv4 que IPv6 au sein de leur réseau virtuel ou sur Internet.
    > [!IMPORTANT]
    > Les sous-réseaux pour IPv6 doivent avoir une taille égale à /64.  Cela garantit la compatibilité ultérieure si vous décidez d’activer le routage du sous-réseau vers un réseau local, car certains routeurs acceptent uniquement les itinéraires IPv6 /64.  
- Possibilité de protéger vos ressources avec des règles de protocole IPv6 pour les groupes de sécurité réseau.
    - Et les protections de déni de service distribué (DDoS) de la plateforme Azure sont étendues aux adresses IP publiques accessibles sur Internet
- Possibilité de personnaliser le routage du trafic IPv6 dans votre réseau virtuel avec des itinéraires définis par l’utilisateur, en particulier lorsque vous utilisez des appliances virtuelles réseau pour augmenter votre application.
- Les machines virtuelles Linux et Windows peuvent toutes utiliser le protocole IPv6 pour les réseaux virtuels Azure
- Prise en charge de l'[équilibreur de charge public IPv6 standard](virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md) pour créer des applications extensibles résilientes, à savoir :
    - Sonde d’intégrité IPv6 en option pour déterminer quelles instances de pool back-end sont intègres et donc en état de recevoir de nouveaux flux.
    - Règles de trafic sortant fournissant un contrôle déclaratif complet sur la connectivité sortante pour mettre à l’échelle et adapter cette fonctionnalité en fonction de vos besoins spécifiques.
    - Configurations front-end facultatives qui permettent à un même équilibreur de charge d’utiliser plusieurs adresses IP publiques IPv6. Les mêmes protocole et port frontal peuvent être réutilisés pour plusieurs adresses front-end.
    - Les ports IPv6 facultatifs peuvent être réutilisés sur des instances principales à l’aide de la fonctionnalité *IP flottante* des règles d’équilibrage de charge 
    - Remarque : L’équilibrage de charge n’effectue aucune traduction de protocole (pas de NAT64). 
    - Remarque : La charge d’IPv6 peut être équilibrée uniquement vers l’interface réseau principale (carte réseau) sur les machines virtuelles Azure. 
- [L'équilibreur de charge interne IPv6 Standard](ipv6-dual-stack-standard-internal-load-balancer-powershell.md) prend en charge la création d'applications multiniveau résilientes dans des réseaux virtuels Azure.   
- Prise en charge de l'équilibreur de charge public IPv6 De base pour assurer la compatibilité avec des déploiements hérités
- [Les adresses et plages d’adresses IP publiques IPv6 réservées](ipv6-public-ip-address-prefix.md) fournissent des adresses IPv6 stables et prévisibles qui facilitent le filtrage de vos applications hébergées par Azure pour votre entreprise et vos clients.
- Une adresse IP publique de niveau d’instance fournit une connectivité Internet IPv6 directement aux machines virtuelles individuelles.
- [Ajouter IPv6 à des déploiements uniquement IPv4 existants](ipv6-add-to-existing-vnet-powershell.md) : cette fonctionnalité vous permet d’ajouter facilement une connectivité IPv6 à des déploiements uniquement IPv4 existants sans avoir à recréer de déploiements.  Le trafic réseau IPv4 n’étant pas affecté au cours de ce processus, en fonction de votre application et du système d’exploitation, vous pouvez ajouter le protocole IPv6 même à des services en direct.    
- Accordez aux clients Internet un accès fluide à votre application à double pile à l’aide du protocole de leur choix avec la prise en charge d’Azure DNS pour les enregistrements (AAAA) IPv6. 
- Créez des applications double pile dont l’échelle s’adapte automatiquement à votre charge à l’aide de groupes de machines virtuelles identiques avec IPv6.
- Un [appairage de réseaux virtuels](virtual-network-peering-overview.md), tant au sein d’une région qu’à l’échelle globale, vous permet de connecter sans problème des réseaux virtuels double pile. Les points de terminaison IPv4 et IPv6 sur les machines virtuelles dans les réseaux appairés pourront communiquer entre eux. Vous pouvez même effectuer un appairage double pile avec des réseaux virtuels uniquement IPv4 lorsque vous opérez la transition de vos déploiements vers le mode double pile. 
- Le dépannage et les diagnostics du protocole IPv6 sont disponibles avec les métriques et alertes de l’équilibreur de charge, ainsi que des fonctionnalités de Network Watcher telles que la capture de paquets, les journaux de flux de groupe de sécurité réseau (NSG), la résolution des problèmes de connexion et la vérification de la connectivité.   

## <a name="scope"></a>Étendue
Le protocole IPv6 pour réseaux virtuels Azure VNET est un ensemble de fonctionnalités de base qui permet aux clients d’héberger des applications double pile (IPv4 + IPv6) dans Azure.  Nous avons l’intention d’ajouter la prise en charge du protocole IPv6 à d’autres fonctionnalités de mise en réseau Azure au fil du temps, voire d’offrir des versions double pile des services PaaS Azure. Entre-temps, tous les services Azure PaaS sont accessibles via les points de terminaison IPv4 sur des machines virtuelles double pile.   

## <a name="limitations"></a>Limites
La version actuelle du protocole IPv6 pour réseau virtuel Azure présente les limitations suivantes :
- IPv6 pour réseau virtuel Azure est disponible dans les régions US Government et toutes les régions commerciales Azure dans le monde avec toutes les méthodes de déploiement.  
- Les passerelles ExpressRoute PEUVENT être utilisées pour le trafic IPv4 uniquement dans un réseau virtuel où le protocole IPv6 est activé.  La prise en charge du trafic IPv6 est incluse dans notre feuille de route.   
- Les passerelles VPN ne peuvent PAS être utilisés dans un réseau virtuel où le protocole IPv6 est activé, que ce soit directement ou appairé avec « UseRemoteGateway ».
- La plateforme Azure (AKS, etc.) ne prend pas en charge la communication IPv6 pour les conteneurs.  
- La charge d’IPv6 peut être équilibrée uniquement vers l’interface réseau principale (carte réseau) sur les machines virtuelles Azure. L’équilibrage de charge pour le trafic IPv6 vers les cartes réseau secondaires n’est pas pris en charge.    
- Les machines virtuelles ou groupes de machines virtuelles identiques exclusivement IPv6 ne sont pas pris en charge, chaque carte réseau doit inclure au moins une configuration IP IPv4. 
- Lorsque vous ajoutez IPv6 à des déploiements IPv4 existants, les plages IPv6 ne peuvent pas être ajoutées à un réseau virtuel avec des liens de navigation vers les ressources qui existent déjà.  
- Le DNS direct pour IPv6 est actuellement pris en charge pour le DNS public Azure, mais le DNS inversé ne l’est pas encore.   

## <a name="pricing"></a>Tarifs

Les ressources et la bande passante du protocole IPv6 Azure sont facturées au même tarif que le protocole IPv4. Le protocole IPv6 n’entraîne pas de faire supplémentaires ou autres. Des informations sont disponibles sur la tarification des [adresses IP publiques](https://azure.microsoft.com/pricing/details/ip-addresses/), de la [bande passante réseau](https://azure.microsoft.com/pricing/details/bandwidth/) ou de l’[équilibreur de charge](https://azure.microsoft.com/pricing/details/load-balancer/).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [déployer une application double pile IPv6 dans Azure PowerShell](virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md).
- Découvrez comment [déployer une application double pile IPv6 à l’aide d’Azure CLI](virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-cli.md).
- Découvrez comment [déployer une application IPv6 double pile à l’aide de modèles de Gestionnaire des ressources (JSON)](ipv6-configure-standard-load-balancer-template-json.md)
