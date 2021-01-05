---
title: Utiliser Azure Application Gateway avec des machines virtuelles VMware
description: Décrit comment utiliser Azure Application Gateway pour gérer le trafic web entrant pour les serveurs web qui s’exécutent sur des machines virtuelles VMware dans l’environnement de cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: b7dce0181987b7e61b243a7eb0e13b7ed687eb08
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82185691"
---
# <a name="use-azure-application-gateway-with-vmware-virtual-machines-in-the-cloudsimple-private-cloud-environment"></a>Utiliser Azure Application Gateway avec des machines virtuelles VMware dans l’environnement de cloud privé CloudSimple

Vous pouvez utiliser Azure Application Gateway pour gérer le trafic web entrant pour les serveurs web qui s’exécutent sur des machines virtuelles VMware dans l’environnement de cloud privé CloudSimple.

En tirant parti d’Azure Application Gateway dans un déploiement hybride public-privé, vous pouvez gérer le trafic web vers vos applications, fournir un serveur frontal sécurisé et décharger le traitement TLS vers les services exécutés dans un environnement VMware. Azure Application Gateway achemine le trafic web entrant vers les instances de pool principal résidant dans des environnements VMware en fonction des règles configurées et des sondes d’intégrité.

Cette solution Azure Application Gateway nécessite de répondre aux conditions suivantes :

* J’ai un abonnement Azure.
* Créer et configurer un réseau virtuel Azure et un sous-réseau au sein du réseau virtuel.
* Créez et configurez des règles NSG et homologuez votre réseau virtuel avec votre cloud privé CloudSimple à l’aide d’ExpressRoute.
* Créez & configurez votre cloud privé.
* Créez et configurez votre passerelle Azure Application Gateway.

## <a name="azure-application-gateway-deployment-scenario"></a>Scénario de déploiement d’Azure Application Gateway

Dans ce scénario, Azure Application Gateway s’exécute dans votre réseau virtuel Azure. Le réseau virtuel est connecté à votre cloud privé via un circuit ExpressRoute. Tous les sous-réseaux du cloud privé sont accessibles par IP à partir des sous-réseaux du réseau virtuel.

![Azure Load Balancer dans un réseau virtuel Azure](media/load-balancer-use-case.png)

## <a name="how-to-deploy-the-solution"></a>Procédure : déployer la solution

Le processus de déploiement comprend les tâches suivantes :

1. [Vérifier que les conditions préalables sont remplies](#1-verify-prerequisites)
2. [Connecter votre connexion virtuelle Azure au cloud privé](#2-connect-your-azure-virtual-network-to-your-private-cloud)
3. [Déployer une passerelle d’application Azure](#3-deploy-an-azure-application-gateway)
4. [Créer et configurer un pool de machines virtuelles de serveur web dans votre cloud privé](#4-create-and-configure-a-web-server-vm-pool-in-your-private-cloud)

## <a name="1-verify-prerequisites"></a>1. Vérifier la configuration requise

Vérifiez que les conditions préalables suivantes sont remplies :

* Une instance Azure Resource Manager et un réseau virtuel ont déjà été créés.
* Un sous-réseau dédié (pour Application Gateway) a déjà été créé au sein de votre réseau virtuel Azure.
* Un cloud privé CloudSimple a déjà été créé.
* Il n’existe aucun conflit d’adresse IP entre les sous-réseaux IP dans le réseau virtuel et les sous-réseaux dans le cloud privé.

## <a name="2-connect-your-azure-virtual-network-to-your-private-cloud"></a>2. Connecter votre réseau virtuel Azure à votre cloud privé

Pour connecter votre réseau virtuel Azure à votre Cloud privé, suivez ce processus.

1. [Dans le portail CloudSimple, copiez les informations de peering ExpressRoute](virtual-network-connection.md).

2. [Configurez une passerelle de réseau virtuel pour votre réseau virtuel Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).

3. [Liez votre réseau virtuel au circuit ExpressRoute CloudSimple](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#connect-a-vnet-to-a-circuit---different-subscription).

4. [Utilisez les informations de peering que vous avez copiées pour lier votre réseau virtuel au circuit ExpressRoute](virtual-network-connection.md).

## <a name="3-deploy-an-azure-application-gateway"></a>3. Déployer une passerelle d’application Azure

Les instructions détaillées pour cela sont disponibles dans [Créer une passerelle d’application avec des règles d’acheminement par chemin d’accès à l’aide du portail Azure](../application-gateway/create-url-route-portal.md). Voici un résumé de la procédure :

1. Créez un réseau virtuel dans votre abonnement et votre groupe de ressources.
2. Créez un sous-réseau (à utiliser comme sous-réseau dédié) au sein de votre réseau virtuel.
3. Créez une instance Application Gateway standard (activez éventuellement WAF) :  Sur la page d'accueil du portail Azure, cliquez sur **Ressource** > **Réseau** > **Application Gateway** dans le coin supérieur gauche de la page. Sélectionnez la référence et la taille standard et fournissez l’abonnement Azure, le groupe de ressources et les informations relatives à l’emplacement. Si nécessaire, créez une nouvelle adresse IP publique pour cette passerelle d’application et fournissez des détails sur le réseau virtuel et le sous-réseau dédié pour la passerelle d’application.
4. Ajoutez un pool principal avec des machines virtuelles et ajoutez-le à votre passerelle d’application.

## <a name="4-create-and-configure-a-web-server-vm-pool-in-your-private-cloud"></a>4. Créer et configurer un pool de machines virtuelles de serveur web dans votre cloud privé

Dans vCenter, créez des machines virtuelles avec le système d’exploitation et le serveur web de votre choix (par exemple, Windows/IIS ou Linux/Apache). Choisissez un sous-réseau/réseau local virtuel qui est désigné pour la couche web dans votre cloud privé. Vérifiez qu’au moins une carte réseau virtuelle des machines virtuelles du serveur web se trouve sur le sous-réseau de couche web.
