---
title: Azure VMware Solution by CloudSimple – Configurer le chiffrage vSAN pour le cloud privé
description: Décrit comment configurer la fonctionnalité de chiffrement du logiciel vSAN afin que votre cloud privé CloudSimple puisse fonctionner avec un serveur de gestion de clés exécuté dans votre réseau virtuel Azure.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 638b60bd3612fa25350ecef0a738fea75c2f53d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77020639"
---
# <a name="configure-vsan-encryption-for-cloudsimple-private-cloud"></a>Configurer le chiffrement vSAN pour un cloud privé CloudSimple

Vous pouvez configurer la fonctionnalité de chiffrement du logiciel vSAN afin que votre cloud privé CloudSimple puisse fonctionner avec un serveur de gestion de clés exécuté dans votre réseau virtuel Azure.

VMware requiert l’utilisation d’un outil de serveur externe de gestion de clés (KMS) tiers conforme au protocole KMIP 1.1 pour le chiffrement vSAN. Vous pouvez tirer parti de tous les services KMS pris en charge certifiés par VMware et disponibles pour Azure.

Ce guide explique comment utiliser le service KMS HyTrust KeyControl exécuté dans un réseau virtuel Azure. Une approche similaire peut être utilisée pour toute autre solution KMS tierce certifiée pour vSAN.

Cette solution KMS requiert que vous effectuiez les opérations suivantes :

* Installer, configurer et gérer un outil KMS tiers certifié VMware dans votre réseau virtuel Azure.
* Fournir vos propres licences pour l’outil KMS.
* Configurer et gérer le chiffrement vSAN dans votre cloud privé à l’aide de l’outil KMS tiers exécuté dans votre réseau virtuel Azure.

## <a name="kms-deployment-scenario"></a>Scénario de déploiement de KMS

Le cluster de serveurs KMS s’exécute dans votre réseau virtuel Azure et l’adresse IP est joignable à partir du cloud privé vCenter via la connexion Azure ExpressRoute configurée.

![../media/cluster KMS dans le réseau virtuel Azure](media/vsan-kms-cluster.png)

## <a name="how-to-deploy-the-solution"></a>Procédure : déployer la solution

Le processus de déploiement comprend les étapes suivantes :

1. [Vérifier que les conditions préalables sont remplies](#verify-prerequisites-are-met)
2. [Portail CloudSimple : obtenir des informations de peering ExpressRoute](#cloudsimple-portal-obtain-expressroute-peering-information)
3. [Portail Azure : Connecter votre réseau virtuel au cloud privé](#azure-portal-connect-your-virtual-network-to-your-private-cloud)
4. [Portail Azure : déployer un cluster HyTrust KeyControl dans votre réseau virtuel](#azure-portal-deploy-a-hytrust-keycontrol-cluster-in-the-azure-resource-manager-in-your-virtual-network)
5. [HyTrust WebUI : configurer le serveur KMIP](#hytrust-webui-configure-the-kmip-server)
6. [Interface utilisateur de vCenter : configurer le chiffrement vSAN pour utiliser le cluster KMS dans votre réseau virtuel Azure](#vcenter-ui-configure-vsan-encryption-to-use-kms-cluster-in-your-azure-virtual-network)

### <a name="verify-prerequisites-are-met"></a>Vérifier que les conditions préalables sont remplies

Vérifier les éléments suivants avant le déploiement :

* Le fournisseur KMS, l’outil et la version sélectionnés figurent sur la liste de compatibilité vSAN.
* Le fournisseur sélectionné prend en charge une version de l’outil à exécuter dans Azure.
* La version Azure de l’outil KMS est conforme à KMIP 1.1.
* Une instance Azure Resource Manager et un réseau virtuel ont déjà été créés.
* Un cloud privé CloudSimple a déjà été créé.

### <a name="cloudsimple-portal-obtain-expressroute-peering-information"></a>Portail CloudSimple : obtenir des informations de peering ExpressRoute

Pour continuer l’installation, vous avez besoin de la clé d’autorisation et de l’URI du circuit pair pour ExpressRoute ainsi que de l’accès à votre abonnement Azure. Ces informations sont disponibles sur la page Connexion au réseau virtuel du portail CloudSimple. Pour recevoir des instructions, consultez [Configurer une connexion entre un réseau virtuel et le cloud privé](virtual-network-connection.md). Si vous avez du mal à obtenir ces informations, ouvrez une [demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

### <a name="azure-portal-connect-your-virtual-network-to-your-private-cloud"></a>Portail Azure : Connecter votre réseau virtuel à votre cloud privé

1. Créez une passerelle de réseau virtuel pour votre réseau virtuel en suivant les instructions données dans [Configurer une passerelle de réseau virtuel pour ExpressRoute à l’aide du portail Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).
2. Liez votre réseau virtuel au circuit CloudSimple ExpressRoute en suivant les instructions données dans [Connecter un réseau virtuel à un circuit ExpressRoute à l’aide du portail](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md).
3. Utilisez les informations de circuit CloudSimple ExpressRoute reçues dans votre e-mail de bienvenue de CloudSimple pour lier votre réseau virtuel au circuit ExpressRoute CloudSimple dans Azure.
4. Entrez la clé d’autorisation et l’URI du circuit pair, donnez un nom à la connexion, puis cliquez sur **OK**.

![Fournissez un URI de circuit pair CS lors de la création du réseau virtuel](media/vsan-azureportal01.png) 

### <a name="azure-portal-deploy-a-hytrust-keycontrol-cluster-in-the-azure-resource-manager-in-your-virtual-network"></a>Portail Azure : déployer un cluster HyTrust KeyControl dans votre réseau virtuel

Pour déployer un cluster HyTrust KeyControl dans Azure Resource Manager dans votre réseau virtuel, effectuez les tâches suivantes. Consultez la [documentation de HyTrust](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0) pour plus d'informations.

1. Créez un groupe de sécurité réseau Azure (nsg-hytrust) avec des règles de trafic entrant spécifiées en suivant les instructions de la documentation de HyTrust.
2. Générez une paire de clés SSH dans Azure.
3. Déployez le nœud KeyControl initial à partir de l’image dans la place de marché Azure.  Utilisez la clé publique de la paire de clés qui a été générée et sélectionnez **nsg-hytrust** comme groupe de sécurité réseau pour le nœud KeyControl.
4. Convertissez l’adresse IP privée de KeyControl en adresse IP statique.
5. SSH sur la machine virtuelle KeyControl à l’aide de son adresse IP publique et de la clé privée de la paire de clés mentionnée précédemment.
6. Lorsque vous y êtes invité dans l’interpréteur de commandes SSH Shell, sélectionnez `No` pour définir le nœud comme nœud KeyControl initial.
7. Ajoutez des nœuds KeyControl supplémentaires en répétant les étapes 3à 5 de cette procédure et en sélectionnant `Yes` lorsque vous êtes invité à ajouter à un cluster existant.

### <a name="hytrust-webui-configure-the-kmip-server"></a>HyTrust WebUI : configurer le serveur KMIP

Accédez à https://*public-ip*, où *public-IP* est l’adresse IP publique de la machine virtuelle du nœud KeyControl. Suivez ces étapes mentionnées dans la [documentation HyTrust ](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0).

1. [Configuration d’un serveur KMIP](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/configuring-kmip-server.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____2)
2. [Création d’un bundle de certificats pour le chiffrement VMware](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-user-for-vmcrypt.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____3)

### <a name="vcenter-ui-configure-vsan-encryption-to-use-kms-cluster-in-your-azure-virtual-network"></a>Interface utilisateur de vCenter : configurer le chiffrement vSAN pour utiliser le cluster KMS dans votre réseau virtuel Azure

Suivez les instructions de HyTrust pour [créer un cluster KMS dans vCenter](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-KMS-Cluster.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____4).

![Ajoutez les détails du cluster KMS dans vCenter](media/vsan-config01.png)

Dans vCenter, accédez à **Cluster > Configurer** et sélectionnez l'option **Général** pour vSAN. Activez le chiffrement et sélectionnez le cluster KMS précédemment ajouté à vCenter.

![Activez le chiffrement vSAN et configurez le cluster KMS dans vCenter](media/vsan-config02.png)

## <a name="references"></a>References

### <a name="azure"></a>Azure

[Configurer une passerelle de réseau virtuel pour ExpressRoute à l’aide du portail Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md)

[Connecter un réseau virtuel à un circuit ExpressRoute à l’aide du portail](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)

### <a name="hytrust"></a>HyTrust

[HyTrust DataControl et Microsoft Azure](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0)

[Configuration d’un serveur KMPI](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/configuring-kmip-server.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____2)

[Création d’un bundle de certificats pour le chiffrement VMware](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-user-for-vmcrypt.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____3)

[Création du cluster KMS dans vSphere](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-KMS-Cluster.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____4)
