---
title: Considérations relatives à la mise en réseau - HSM Azure dédié| Microsoft Docs
description: Vue d’ensemble des considérations relatives à la mise en réseau applicables aux déploiements de HSM Azure dédié
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: mbaldwin
ms.openlocfilehash: 3764b261b491c660da16d7989be20742fead1fbf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91359152"
---
# <a name="azure-dedicated-hsm-networking"></a>Mise en réseau du module Azure HSM dédié

Le module Azure HSM dédié requiert un environnement de mise en réseau hautement sécurisé. Cette condition est vraie, que ce soit du cloud Azure vers l’environnement informatique du client (local), en utilisant des applications distribuées, ou dans des scénarios à haute disponibilité. La mise en réseau Azure offre cette possibilité, et quatre zones distinctes doivent être prises en compte.

- Création d’appareils HSM à l’intérieur de votre réseau virtuel (VNet) dans Azure
- Connexion locale à des ressources dans le cloud pour la configuration et la gestion d’appareils HSM
- Création et connexion de réseaux virtuels pour l’interconnexion entre des ressources d’application et des appareils HSM
- Connexion de réseaux virtuels entre des régions pour l’intercommunication et les scénarios à haute disponibilité

## <a name="virtual-network-for-your-dedicated-hsms"></a>Réseau virtuel pour vos modules HSM dédiés

Les modules HSM dédiés sont intégrés à un réseau virtuel et placés dans le propre réseau privé de vos clients dans Azure. Cela permet l’accès aux appareils à partir de machines virtuelles ou de ressources de calcul dans le réseau virtuel.  
Pour plus d’informations sur l’intégration des services Azure au réseau virtuel et sur les fonctionnalités fournies, consultez la documentation [Réseau virtuel pour les services Azure](../virtual-network/virtual-network-for-azure-services.md).

### <a name="virtual-networks"></a>Réseaux virtuels

Avant de configurer un appareil HSM dédié, les clients devront d’abord créer un réseau virtuel dans Azure ou utiliser un réseau virtuel déjà présent dans l’abonnement des clients. Le réseau virtuel définit le périmètre de sécurité de l’appareil HSM dédié. Pour plus d’informations sur la création de réseaux virtuels, voir la [documentation du réseau virtuel](../virtual-network/virtual-networks-overview.md).

### <a name="subnets"></a>Sous-réseaux

Les sous-réseaux segmentent le réseau virtuel en espaces d’adressage distincts utilisables par les ressources Azure que vous placez dedans. Les modules HSM dédiés sont déployés dans un sous-réseau du réseau virtuel. Chaque appareil HSM dédié déployé dans le sous-réseau du client reçoit une adresse IP privée provenant de ce sous-réseau. Le sous-réseau dans lequel l’appareil HSM est déployé doit être explicitement délégué au service : Microsoft.HardwareSecurityModules/dedicatedHSMs. Cette opération octroie au service HSM certaines autorisations pour le déploiement dans le sous-réseau. La délégation à des modules HSM dédiés impose certaines restrictions de stratégie sur le sous-réseau. Les groupes de sécurité réseau (NSG) et les itinéraires définis par l’utilisateur (UDR) ne sont actuellement pas pris en charge sur les sous-réseaux délégués. Par conséquent, lorsqu’un sous-réseau est délégué à des modules HSM dédiés, il peut uniquement servir à déployer des ressources HSM. Le déploiement de toute autre ressource client dans le sous-réseau échouera.


### <a name="expressroute-gateway"></a>Passerelle ExpressRoute

L’architecture actuelle exige la configuration d’une passerelle ER dans le sous-réseau des clients, où un appareil HSM doit être placé pour permettre l’intégration de cet appareil HSM dans Azure. Cette passerelle ER ne peut pas être utilisée pour connecter des emplacements locaux aux appareils HSM clients dans Azure.

## <a name="connecting-your-on-premises-it-to-azure"></a>Connexion de votre environnement informatique local à Azure

La création de ressources dans le cloud exige généralement une connexion privée aux ressources informatiques locales. Dans le cas d’un module HSM dédié, il s’agira principalement de la configuration des appareils HSM par le logiciel client HSM ainsi que des activités telles que les sauvegardes et les extractions de journaux d’activité à partir de modules HSM à des fins d’analyse. Il est important ici de tenir compte de la nature de la connexion et des options qui en découlent.  L’option la plus souple est un VPN de site à site car plusieurs ressources locales nécessiteront probablement une communication sécurisée à des ressources (y compris les modules HSM) dans le cloud Azure. Pour cette opération, l’organisation du client devra disposer d’un appareil VPN pour établir la connexion. Une connexion VPN point à site peut être utilisée s’il n’existe qu’un seul point de terminaison local, par exemple une station de travail d’administration unique.
Pour plus d’informations sur les options de connectivité, consultez [Options de planification de la passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#planningtable).

> [!NOTE]
> À ce stade, ExpressRoute ne peut pas être utilisé pour la connexion à des ressources locales. Il convient également de noter que la passerelle ExpressRoute utilisée comme décrit ci-dessus n’est pas adaptée aux connexions à une infrastructure locale.

### <a name="point-to-site-vpn"></a>VPN de point à site

Un réseau privé virtuel point à site est la forme la plus simple d’une connexion sécurisée à un seul point de terminaison local. Cela peut être utile si vous prévoyez d’utiliser une station de travail d’administration unique pour les modules HSM dédiés basé sur Azure.

### <a name="site-to-site-vpn"></a>VPN de site à site

Un réseau privé virtuel de site à site garantit une communication sécurisée entre des modules HSM dédiés basés sur Azure et votre environnement informatique local. Avec cette configuration, vous disposez d’une solution de sauvegarde pour le module HSM sur site et d’une connexion entre ces deux éléments afin de lancer la sauvegarde.

## <a name="connecting-virtual-networks"></a>Connexion de réseaux virtuels

Une architecture de déploiement classique pour un module HSM dédié démarre avec un seul réseau virtuel et un sous-réseau correspondant dans lequel les appareils HSM sont créés et configurés. Cette même région peut également englober d’autres réseaux virtuels et sous-réseaux pour les composants d’application susceptibles d’utiliser le module HSM dédié. Pour permettre la communication entre ces réseaux, nous utilisons le peering de réseaux virtuels.

### <a name="virtual-network-peering"></a>Peering de réseau virtuel

Lorsque plusieurs réseaux virtuels d’une région doivent accéder aux ressources de chacun d’eux, le peering de réseaux virtuels peut servir à créer des canaux de communication sécurisés entre ces réseaux.  Le peering de réseaux virtuels garantit non seulement des communications sécurisées, mais également des connexions à faible latence et large bande passante entre les ressources dans Azure.

![Peering de réseau](media/networking/peering.png)

## <a name="connecting-across-azure-regions"></a>Connexion entre des régions Azure

Les appareils HSM ont la possibilité, par le biais de bibliothèques logicielles, de rediriger le trafic vers un autre module HSM. Cette redirection du trafic est utile en cas de défaillance d’un appareil ou de l’impossibilité d’y accéder. Ces risques de défaillance au niveau régional peuvent être limités en déployant des modules HSM dans d’autres régions et en établissant une communications entre des réseaux virtuels dans différentes régions.

### <a name="cross-region-ha-using-vpn-gateway"></a>Haute disponibilité inter-région à l’aide d’une passerelle VPN

Pour les applications distribuées globalement ou les scénarios de basculement régional à haute disponibilité, il est nécessaire de connecter des réseaux virtuels dans différentes régions. Un module Azure HSM dédié assure une haute disponibilité en utilisant une passerelle VPN qui fournit un tunnel sécurisé entre les deux réseaux virtuels. Pour plus d’informations sur les connexions de réseau virtuel à réseau virtuel à l’aide d’une passerelle VPN, consultez l’article [Présentation d’une passerelle VPN](../vpn-gateway/design.md#V2V).

> [!NOTE]
> Pour le moment, le peering de réseaux virtuels global n’est pas disponible dans des scénarios de connectivité inter-région incluant des modules HSM dédiés et vous devez utiliser à la place une passerelle VPN. 

![Diagramme représentant deux régions reliées par deux passerelles VPN. Chaque région contient des réseaux virtuels appairés.](media/networking/global-vnet.png)

## <a name="next-steps"></a>Étapes suivantes

- [Forum Aux Questions (FAQ)](faq.md)
- [Prise en charge](supportability.md)
- [Haute disponibilité](high-availability.md)
- [Sécurité physique](physical-security.md)
- [Surveillance](monitoring.md)
- [Architecture de déploiement](deployment-architecture.md)