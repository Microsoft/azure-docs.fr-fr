---
title: Azure VMware Solution by CloudSimple - Client Access vSphere
description: Explique comment accéder au vCenter de votre cloud privé.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/30/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 18d9463bc512257034860e1188372879524924f7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77022662"
---
# <a name="access-your-private-cloud-vcenter-portal"></a>Accéder au portail vCenter de votre cloud privé

Vous pouvez ouvrir le portail vCenter de votre cloud privé à partir du portai Azure ou du portail CloudSimple.  Le portail vCenter vous permet de gérer l’infrastructure VMware sur votre cloud privé.

## <a name="before-you-begin"></a>Avant de commencer

Pour accéder au portail vCenter, une connexion réseau doit être établie et la résolution des noms DNS doit être activée.  Vous pouvez établir une connexion réseau vers votre cloud privé à l’aide de l’une des options ci-dessous.

* [Connecter un réseau local à CloudSimple à l’aide d’ExpressRoute](on-premises-connection.md)
* [Configurer une connexion VPN vers votre cloud privé CloudSimple](set-up-vpn.md)

Pour configurer la résolution des noms DNS des composants de l’infrastructure VMware de votre cloud privé, consultez [Configurer la résolution des noms DNS pour accéder au vCenter du cloud privé à partir de stations de travail locales](on-premises-dns-setup.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="access-vcenter-from-azure-portal"></a>Accéder à vCenter à partir du portail Azure

Vous pouvez ouvrir le portail vCenter de votre cloud privé à partir du portail Azure.

1. Sélectionnez **Tous les services**.

2. Recherchez **Services CloudSimple**.

3. Sélectionnez le service CloudSimple du cloud privé auquel vous voulez vous connecter.

4. Dans la page **Overview** (Vue d’ensemble), cliquez sur **View VMware Private Clouds** (Afficher les clouds privés VMware).

    ![Vue d’ensemble du service CloudSimple](media/cloudsimple-service-overview.png)

5. Sélectionnez le cloud privé dans la liste, puis cliquez sur **Launch vSphere Client** (Lancer le client vSphere).

    ![Lancer le client vSphere](media/cloudsimple-service-launch-vsphere-client.png)

## <a name="access-vcenter-from-cloudsimple-portal"></a>Accéder à vCenter à partir du portail CloudSimple

Vous pouvez ouvrir le portail vCenter de votre cloud privé à partir du portail CloudSimple.

1. Accédez au [portail CloudSimple](access-cloudsimple-portal.md).

2. Dans **Ressources**, sélectionnez le cloud privé auquel vous souhaitez accéder, puis cliquez sur **Launch vSphere Client**.

    ![Lancer le client vSphere - Ressources](media/cloudsimple-portal-resources-launch-vcenter.png)

3. Vous pouvez également ouvrir le portail vCenter à partir de l’écran récapitulatif de votre cloud privé.

    ![Lancer le client vSphere - Récapitulatif](media/cloudsimple-resources-summary-launch-vcenter.png)

## <a name="next-steps"></a>Étapes suivantes

* [Créer et gérer des réseaux locaux virtuels ou des sous-réseaux pour vos clouds privés](create-vlan-subnet.md)
* [Modèle d’autorisation de cloud privé CloudSimple de VMware vCenter](learn-private-cloud-permissions.md)