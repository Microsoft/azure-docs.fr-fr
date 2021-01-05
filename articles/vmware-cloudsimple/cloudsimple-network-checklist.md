---
title: Azure VMware Solution by CloudSimple - Check-list pour le réseau
description: Check-list pour allouer le CIDR réseau sur Azure VMware Solution by CloudSimple
author: sharaths-cs
ms.author: dikamath
ms.date: 09/25/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: bfb170036293dc9f519259dc92737f30380aa84a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025008"
---
# <a name="networking-prerequisites-for-azure-vmware-solution-by-cloudsimple"></a>Prérequis du réseau pour Azure VMware Solution by CloudSimple

Azure VMware Solution by CloudSimple offre un environnement de cloud privé VMware accessible aux utilisateurs et aux applications à partir d’environnements locaux, d’appareils gérés par l’entreprise et de ressources Azure. La connectivité est fournie à l’aide de services réseau comme des réseaux privés virtuels et des connexions Azure ExpressRoute. L’activation de certains de ces services réseau nécessite la spécification de plages d’adresses réseau. 

Les tableaux de cet article décrivent l’ensemble des plages d’adresses et les services correspondants qui utilisent les adresses spécifiées. Certaines des adresses sont obligatoires et d’autres dépendent des services que vous voulez déployer. Ces espaces d’adressage ne doivent pas chevaucher vos sous-réseaux locaux, les sous-réseaux du réseau virtuel Azure ou les sous-réseaux des charges de travail CloudSimple planifiées.

## <a name="network-address-ranges-required-for-creating-a-private-cloud"></a>Plages d’adresses réseau nécessaires à la création d’un cloud privé

Pendant la création d’un service CloudSimple et d’un cloud privé, vous devez vous conformer aux plages CIDR (Classless Inter-Domain Routing) de réseau spécifiées, comme suit.

| Nom/utilisé pour     | Description                                                                                                                            | Plage d’adresses            |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------|--------------------------|
| CIDR de la passerelle      | Nécessaire pour les services Edge (passerelles VPN).  Ce CIDR est nécessaire lors de la création du service CloudSimple et doit provenir de l’espace RFC 1918. | /28                      |
| CIDR vSphere/vSAN | Nécessaire pour les réseaux de gestion VMware. Ce CIDR doit être spécifié lors de la création d’un cloud privé.                                    | /24 ou /23 ou /22 ou /21 |

## <a name="network-address-range-required-for-azure-network-connection-to-an-on-premises-network"></a>Plage d’adresses réseau nécessaire pour la connexion du réseau Azure à un réseau local

La connexion d’un [réseau local au réseau du cloud privé avec ExpressRoute](on-premises-connection.md) établit une connexion Global Reach.  La connexion utilise le protocole BGP (Border Gateway Protocol) pour échanger les routes entre votre réseau local, le réseau du cloud privé et vos réseaux Azure.

| Nom/utilisé pour             | Description                                                                                                                                                                             | Plage d’adresses |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| CIDR de l’appairage ExpressRoute | Nécessaire quand vous utilisez ExpressRoute Global Reach pour la connectivité locale. Ce CIDR doit être fourni quand une demande de connexion Global Reach est effectuée par le biais d’un ticket de support. | /29           |

## <a name="network-address-range-required-for-using-a-site-to-site-vpn-connection-to-an-on-premises-network"></a>Plage d’adresses réseau nécessaire pour l’utilisation d’une connexion VPN site à site à un réseau local

La connexion d’un [réseau local au réseau du cloud privé avec un VPN site à site](vpn-gateway.md) nécessite les adresses IP, le réseau local et les identificateurs suivants. 

| Adresse/plage d’adresses | Description                                                                                                                                                                                                                                                           |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Adresse IP du pair               | Adresse IP publique de la passerelle VPN locale. Nécessaire pour établir une connexion VPN site à site entre un centre de données local et la région du service CloudSimple. Cette adresse IP est nécessaire lors de la création d’une passerelle VPN site à site.                                         |
| Identificateur du pair       | Identificateur du pair de votre passerelle VPN locale. C’est généralement identique à l’**adresse IP du pair**.  Si un identificateur unique est spécifié sur votre passerelle VPN locale, l’identificateur doit être spécifié.  L’ID de pair est nécessaire lors de la création d’une passerelle VPN site à site.   |
| Réseaux locaux   | Préfixes locaux qui doivent accéder aux réseaux CloudSimple dans la région.  Incluez tous les préfixes d’un réseau local qui vont accéder au réseau CloudSimple, notamment le réseau client à partir duquel les utilisateurs vont accéder au réseau.                                         |

## <a name="network-address-range-required-for-using-point-to-site-vpn-connections"></a>Plage d’adresses réseau nécessaire pour l’utilisation de connexions VPN point à site

Une connexion VPN point à site permet d’accéder au réseau CloudSimple à partir d’un ordinateur client.  [Pour configurer un VPN point à site](vpn-gateway.md), vous devez spécifier la plage d’adresses réseau suivante.

| Adresse/plage d’adresses | Description                                                                                                                                                                                                                                                                                                  |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Sous-réseau client         | Les adresses DHCP sont fournies par le sous-réseau client quand vous vous connectez avec un VPN point à site. Ce sous-réseau est nécessaire lors de la création d’une passerelle VPN point à site sur un portail CloudSimple.  Le réseau est divisé en deux sous-réseaux : un pour la connexion UDP et l’autre pour les connexions TCP. |

## <a name="next-steps"></a>Étapes suivantes

* [Configuration d’un pare-feu local pour accéder à votre cloud privé](on-premises-firewall-configuration.md)
* [Démarrage rapide : Créer un service CloudSimple](quickstart-create-cloudsimple-service.md)
* [Démarrage rapide : Configurer un environnement de cloud privé](quickstart-create-private-cloud.md)
* Découvrir plus d’informations sur les [connexions réseau Azure](cloudsimple-azure-network-connection.md)
* Découvrir plus d’informations sur les [passerelles VPN](cloudsimple-vpn-gateways.md)
