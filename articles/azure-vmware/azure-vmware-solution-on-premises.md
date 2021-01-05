---
title: Connecter Azure VMware Solution à votre environnement local
description: Apprenez à connecter Azure VMware Solution à votre environnement local.
ms.topic: tutorial
ms.date: 10/02/2020
ms.openlocfilehash: 2a0cb641df00f3e580e87e38aff382d8e8101fc7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578764"
---
# <a name="connect-azure-vmware-solution-to-your-on-premises-environment"></a>Connecter Azure VMware Solution à votre environnement local

Dans cet article, vous allez continuer à utiliser les [informations recueillies lors de la planification](production-ready-deployment-steps.md) pour connecter Azure VMware Solution à votre environnement local.

Avant de commencer, sachez qu'il existe deux prérequis pour connecter Azure VMware Solution à votre environnement local :

- Un circuit ExpressRoute entre votre environnement local et Azure
- Un bloc d'adresses réseau /29 qui ne se chevauchent pas pour le peering d'ExpressRoute Global Reach, que vous avez défini dans le cadre de la [phase de planification](production-ready-deployment-steps.md)

>[!NOTE]
> Vous pouvez vous connecter par le biais d'un VPN, mais cette opération n'est pas couverte par ce document de démarrage rapide.

## <a name="establish-an-expressroute-global-reach-connection"></a>Établir une connexion ExpressRoute Global Reach

Pour établir une connectivité locale avec votre cloud privé Azure VMware Solution à l'aide d'ExpressRoute Global Reach, suivez le tutoriel [​​Peering d'environnements locaux avec un cloud privé](tutorial-expressroute-global-reach-private-cloud.md).



## <a name="verify-on-premises-network-connectivity"></a>Vérifier la connectivité réseau locale

Votre **routeur de périphérie local** doit maintenant indiquer où ExpressRoute connecte les segments de réseau NSX-T et les segments de gestion d'Azure VMware Solution.

>[!NOTE]
>Chaque environnement est différent, et certains doivent autoriser ces itinéraires à se propager à nouveau sur le réseau local.  

Certains environnements sont dotés d'un pare-feu qui protège le circuit ExpressRoute.  En l'absence de pare-feu et de fonctionnalité de nettoyage d'itinéraire, vous pouvez effectuer un test ping de votre serveur vCenter Azure VMware Solution ou d'une [machine virtuelle](deploy-azure-vmware-solution.md#add-a-vm-on-the-nsx-t-network-segment) située sur le segment NSX-T à partir de votre environnement local.

En outre, vous pouvez accéder aux ressources de votre environnement local à partir de la machine virtuelle située sur le segment NSX-T.

## <a name="next-steps"></a>Étapes suivantes

Passez à la section suivante pour déployer et configurer VMware HCX

> [!div class="nextstepaction"]
> [Déployer et configurer VMware HCX](tutorial-deploy-vmware-hcx.md)