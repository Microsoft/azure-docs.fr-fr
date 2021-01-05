---
title: Azure VMware Solution by CloudSimple - Connectez le cloud privé au réseau Azure à l’aide d’ExpressRoute
description: Décrit comment connecter votre environnement de cloud privé de CloudSimple au réseau virtuel Azure à l’aide d’ExpressRoute
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 9bb68ec68f4de646239477ceeaac50a7a33989fc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77015233"
---
# <a name="connect-your-cloudsimple-private-cloud-environment-to-the-azure-virtual-network-using-expressroute"></a>Connecter votre environnement de cloud privé de CloudSimple au réseau virtuel Azure à l’aide d’ExpressRoute

Votre cloud privé CloudSimple peut être connecté à votre réseau virtuel Azure à l’aide d’Azure ExpressRoute.  Cette connexion à bande passante élevée et à faible latence vous permet d’accéder aux services qui s’exécutent dans votre abonnement Azure à partir de votre environnement cloud privé.

Une connexion de réseau virtuel vous permet d’effectuer les tâches suivantes :

* Utiliser Azure comme cible de sauvegarde pour les machines virtuelles sur votre cloud privé.
* Déployer des serveurs KMS dans votre abonnement Azure pour chiffrer la banque de données vSAN de votre cloud privé.
* Utiliser des applications hybrides où la couche web de l’application s’exécute dans le cloud public pendant que les couches application et base de données s’exécutent dans votre cloud privé.

![Connexion Azure ExpressRoute au réseau virtuel](media/cloudsimple-azure-network-connection.png)

## <a name="set-up-a-virtual-network-connection"></a>Configurer une connexion de réseau virtuel

Pour configurer la connexion du réseau virtuel à votre cloud privé, vous avez besoin de votre clé d’autorisation, de l’URI du circuit homologue et de l’accès à votre abonnement Azure. Ces informations sont disponibles sur la page Connexion au réseau virtuel du portail CloudSimple. Pour plus d’instructions, consultez [Obtenir des informations de peering pour la connexion entre le réseau virtuel Azure et CloudSimple](virtual-network-connection.md). Si vous avez du mal à obtenir ces informations, envoyez une <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">demande de support</a>.

> [!TIP]
> Si vous disposez déjà d’un réseau virtuel Azure, d’un sous-réseau de passerelle et d’une passerelle de réseau virtuel, vous pouvez passer à l’étape 4.

1. Créez un réseau virtuel sur votre abonnement Azure et vérifiez que l’espace d’adressage que vous sélectionnez est différent de l’espace d’adressage de votre cloud privé.  Si vous disposez déjà d’un réseau virtuel Azure, vous pouvez utiliser celui qui existe déjà.  Pour plus de détails, consultez [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).
2. Créez le sous-réseau de passerelle sur votre réseau virtuel Azure.  Si vous disposez déjà d’un sous-réseau de passerelle dans votre réseau virtuel Azure, vous pouvez utiliser celui qui existe déjà. Pour plus d’informations, consultez [Créer le sous-réseau de passerelle](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-gateway-subnet).
3. Créez la passerelle de réseau virtuel sur votre réseau virtuel.  Si vous disposez d’une passerelle de réseau virtuel existante, vous pouvez utiliser celle qui existe déjà. Pour plus d’informations, consultez [Créer la passerelle de réseau virtuel](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-virtual-network-gateway).
4. Créez la connexion entre votre réseau virtuel et votre cloud privé en échangeant la clé d’autorisation, comme décrit dans [Connecter un réseau virtuel à un circuit - abonnement différent](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#connect-a-vnet-to-a-circuit---different-subscription).

> [!WARNING]
> Si vous utilisez une passerelle de réseau virtuel existante et qu’elle dispose d’une connexion ExpressRoute au même emplacement que le circuit ExpressRoute CloudSimple, la connexion n’est pas établie.  Créez un nouveau réseau virtuel et suivez les étapes précédentes.

## <a name="test-the-virtual-network-connection"></a>Tester la connexion de réseau virtuel Azure

Une fois la connexion créée, vous pouvez vérifier l’état de la connexion en sélectionnant **Propriétés** sous **Paramètres**.  Le statut et l'état d’approvisionnement doivent indiquer **Réussite**.

![État de la connexion](media/azure-expressroute-connection.png)

Pour tester la connexion de réseau virtuel :

1. Créez une machine virtuelle dans votre abonnement Azure.
2. Recherchez l’adresse IP de votre vCenter de cloud privé (reportez-vous à votre e-mail de bienvenue).
3. Exécutez une commande ping sur votre vCenter cloud à partir de la machine virtuelle créée dans votre réseau virtuel Azure.
4. Exécutez une commande ping sur votre machine virtuelle Azure à partir d’une machine virtuelle en cours d’exécution dans votre vCenter de cloud privé.

Si vous rencontrez des problèmes lors de l’établissement de la connexion, envoyez une <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">demande de support</a>.
