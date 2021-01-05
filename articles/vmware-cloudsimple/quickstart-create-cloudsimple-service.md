---
title: 'Démarrage rapide : Créer le service VMware de CloudSimple'
titleSuffix: Azure VMware Solution by CloudSimple
description: En savoir plus sur la création du service CloudSimple, l’achat et la réservation de nœuds
author: sharaths-cs
ms.author: dikamath
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 14df0f131aaef8a4c24e2d1eb242a9b440e7c7b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86507588"
---
# <a name="quickstart---create-azure-vmware-solution-by-cloudsimple-service"></a>Démarrage rapide - Créer le service Azure VMware Solution by CloudSimple

Pour commencer, créez Azure VMware Solution by CloudSimple dans le portail Azure.

## <a name="vmware-solution-by-cloudsimple---service-overview"></a>VMware Solution by CloudSimple - Vue d’ensemble du service

Le service CloudSimple vous permet d’utiliser Azure VMware Solution by CloudSimple.  En créant ce service, vous pouvez approvisionner, réserver des nœuds et créer des clouds privés.  Vous ajoutez le service de CloudSimple à chaque région Azure où ce service est disponible.  Le service définit le réseau de périmètre d’Azure VMware Solution by CloudSimple.  Ce réseau de périmètre est destiné aux services comprenant une connectivité VPN, ExpressRoute et Internet vers vos clouds privés.

Pour ajouter le service CloudSimple, vous devez créer un sous-réseau de passerelle. Le sous-réseau de passerelle est utilisé lors de la création du réseau de périmètre et requiert un bloc CIDR /28. L’espace d’adressage du sous-réseau de passerelle doit être unique. Il ne peut pas chevaucher un de vos espaces d'adressage de réseau local ni l'espace d'adressage du réseau virtuel Azure.

## <a name="before-you-begin"></a>Avant de commencer

Allouez un bloc /28 CIDR pour le sous-réseau de passerelle.  Un sous-réseau de passerelle est requis par le service CloudSimple et est propre à la région dans laquelle il est créé. Le sous-réseau de passerelle est utilisé pour les services de réseau de périmètre Azure VMware Solution by CloudSimple et nécessite un bloc CIDR /28. L’espace d’adressage du sous-réseau de passerelle doit être unique. Il ne doit chevaucher aucun réseau qui communique avec l’environnement CloudSimple.  Les réseaux qui communiquent avec CloudSimple incluent les réseaux locaux et les réseaux virtuels Azure.

Passez en revue les [Conditions préalables à la mise en réseau](cloudsimple-network-checklist.md). 

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-the-service"></a>Créer le service

1. Sélectionnez **Tous les services**.
2. Recherchez le **service CloudSimple**.

    ![Rechercher le service CloudSimple](media/create-cloudsimple-service-search.png)

3. Sélectionnez **Services CloudSimple**.
4. Cliquez sur **Ajouter** pour créer un service.

    ![Ajouter le service CloudSimple](media/create-cloudsimple-service-add.png)

5. Sélectionnez l’abonnement dans lequel vous souhaitez créer le service CloudSimple.
6. Sélectionnez le groupe de ressources du service. Pour ajouter un nouveau groupe de ressources, cliquez sur **Créer**.
7. Entrez un nom pour identifier le service.
8. Entrez la notation CIDR de la passerelle du service. Spécifiez un sous-réseau /28 qui ne chevauche aucun de vos sous-réseaux locaux, sous-réseaux Azure ou sous-réseaux CloudSimple planifiés. Une fois le service créé, vous ne pouvez plus modifier le CIDR.

    ![Créer le service CloudSimple](media/create-cloudsimple-service.png)

9. Cliquez sur **OK**.

Le service est créé et ajouté à la liste des services.

## <a name="provision-nodes"></a>Nœuds d'approvisionnement

Pour configurer la capacité avec paiement à l'utilisation d'un environnement de cloud privé CloudSimple, commencez par approvisionner des nœuds dans le portail Azure.

1. Sélectionnez **Tous les services**.
2. Recherchez **Nœuds CloudSimple**.

    ![Rechercher des nœuds CloudSimple](media/create-cloudsimple-node-search.png)

3. Sélectionnez **Nœuds CloudSimple**.
4. Cliquez sur **Ajouter** pour créer les nœuds.

    ![Ajouter des nœuds CloudSimple](media/create-cloudsimple-node-add.png)

5. Sélectionnez l’abonnement dans lequel vous souhaitez approvisionner des nœuds CloudSimple.
6. Sélectionnez le groupe de ressources pour vos nœuds. Pour ajouter un nouveau groupe de ressources, cliquez sur **Créer**.
7. Entrez le préfixe pour identifier les nœuds.
8. Sélectionnez l’emplacement des ressources de nœud.
9. Sélectionnez l’emplacement dédié où héberger les ressources de nœud.
10. Sélectionnez le [type de nœud](cloudsimple-node.md).
11. Sélectionnez le nombre de nœuds à approvisionner.
12. Sélectionnez **Vérifier + créer**.
13. Passez en revue les paramètres. Pour modifier les paramètres, cliquez sur **Précédent**.
14. Sélectionnez **Create** (Créer).

## <a name="next-steps"></a>Étapes suivantes

* [Créer un cloud privé et configurer l’environnement](quickstart-create-private-cloud.md)
* En savoir plus sur le [service CloudSimple](./cloudsimple-service.md)
