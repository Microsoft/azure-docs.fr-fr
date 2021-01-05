---
title: VMware Solution by CloudSimple - Connexions réseau Azure
description: En savoir plus sur la connexion de votre réseau virtuel Azure au réseau de votre région CloudSimple
author: sharaths-cs
ms.author: dikamath
ms.date: 04/10/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: cfd4d65b07cf255ac2b60d6bf8376723a997374e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025076"
---
# <a name="azure-network-connections-overview"></a>Vue d’ensemble des connexions réseau Azure

Lorsque vous créez un service CloudSimple dans une région ou des nœuds, vous pouvez :

* Faire une requête du circuit Azure ExpressRoute et le joindre au réseau CloudSimple de cette région.
* Connecter le réseau de votre région CloudSimple au réseau virtuel Azure ou à votre réseau local à l’aide d’Azure ExpressRoute.
* Fournir l’accès aux services s’exécutant dans votre abonnement Azure ou dans votre réseau local à partir de votre environnement de cloud privé.

La connexion ExpressRoute est une bande passante élevée à faible latence.

## <a name="benefits"></a>Avantages

Une connexion réseau Azure vous permet d’effectuer les tâches suivantes :

* Utiliser Azure comme cible de sauvegarde pour les machines virtuelles sur votre cloud privé.
* Déployer des serveurs KMS dans votre abonnement Azure pour chiffrer la banque de données vSAN de votre cloud privé.
* Utiliser des applications hybrides où la couche web de l’application s’exécute dans le cloud public pendant que les couches application et base de données s’exécutent dans votre cloud privé.

## <a name="azure-virtual-network-connection"></a>Connexion au réseau virtuel Azure

Les clouds privés peuvent être connectés à vos ressources Azure à l’aide d’ExpressRoute.  Vous pouvez utiliser la connexion ExpressRoute pour accéder aux ressources s’exécutant dans votre abonnement Azure à partir de votre cloud privé.  Cette connexion vous permet d’étendre le réseau de votre cloud privé à votre réseau virtuel Azure.  Les itinéraires du réseau CloudSimple seront échangés par votre réseau virtuel Azure via BGP.  Si le peering du réseau virtuel est configuré, tous les peering des réseaux virtuels sont accessibles à partir de votre réseau CloudSimple.

![Connexion Azure ExpressRoute au réseau virtuel](media/cloudsimple-azure-network-connection.png)

## <a name="expressroute-connection-to-on-premises-network"></a>Connexion ExpressRoute au réseau local

Vous pouvez connecter votre circuit Azure ExpressRoute existant à votre région CloudSimple. La fonctionnalité Global Reach d’ExpressRoute permet de connecter les deux circuits entre eux.  Une connexion est établie entre les circuits ExpressRoute locaux et CloudSimple.  Cette connexion vous permet d’étendre vos réseaux locaux au réseau de votre cloud privé. Les itinéraires de votre réseau CloudSimple seront échangés avec votre réseau virtuel Azure via BGP.

![Connexion ExpressRoute locale - Global Reach](media/cloudsimple-global-reach-connection.png)

## <a name="connection-to-on-premises-network-and-azure-virtual-network"></a>Connexion à un réseau local et à un réseau virtuel Azure

Les connexions à un réseau local et à un réseau virtuel Azure peuvent coexister à partir de votre réseau CloudSimple.  La connexion utilise le BGP pour échanger des itinéraires entre un réseau local, un réseau virtuel Azure et un réseau CloudSimple.  Lorsque vous connectez votre réseau CloudSimple à votre réseau virtuel Azure en présence d’une connexion Global Reach, les itinéraires de réseau virtuel Azure sont visibles sur votre réseau local.  L’échange d’itinéraire s’effectue dans Azure entre les routeurs de périphérie.

![Connexion ExpressRoute locale avec connexion réseau virtuel Azure](media/cloudsimple-global-reach-and-vnet-connection.png)

### <a name="important-considerations"></a>Points importants à prendre en compte

La connexion au réseau CloudSimple à partir d’un réseau local et r d’un réseau virtuel Azure permet d’échanger des itinéraires entre tous les réseaux.

* Le réseau virtuel Azure sera visible à partir du réseau local et du réseau CloudSimple.
* Si vous vous êtes connecté à votre réseau virtuel Azure à partir d’un réseau local, la connexion au réseau CloudSimple à l’aide de Global Reach autorise l’accès aux réseaux virtuels à partir du réseau CloudSimple.
* Les adresses de sous-réseau **ne doivent pas** se chevaucher entre les réseaux connectés.
* CloudSimple ne publiera **pas** l’itinéraire par défaut vers les connexions ExpressRoute
* Si votre routeur local publie l’itinéraire par défaut, le trafic du réseau CloudSimple et du réseau virtuel Azure utilisera l’itinéraire par défaut publié.  Par conséquent, il est impossible d’accéder aux machines virtuelles Azure à l’aide d’adresses IP publiques.

## <a name="next-steps"></a>Étapes suivantes

* [Connecter un réseau virtuel Azure à CloudSimple à l’aide d’ExpressRoute](virtual-network-connection.md)
* [Connecter un réseau local à CloudSimple à l’aide d’ExpressRoute](on-premises-connection.md)
