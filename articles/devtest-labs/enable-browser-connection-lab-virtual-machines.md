---
title: Activer la connexion du navigateur sur les machines virtuelles Azure DevTest Labs
description: DevTest Labs s’intègre désormais à Azure Bastion. En tant que propriétaire du labo, vous pouvez activer l’accès à toutes les machines virtuelles du labo via un navigateur.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 6d9d631c79c22f1f713cfc4ee7cdd766a4ad8f06
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341170"
---
# <a name="enable-browser-connection-on-azure-devtest-labs-virtual-machines"></a>Activer la connexion du navigateur sur les machines virtuelles Azure DevTest Labs 
DevTest Labs s’intègre à [Azure Bastion](../bastion/index.yml), ce qui vous permet de vous connecter à vos machines virtuelles via un navigateur. Vous devez tout d’abord activer la connexion du navigateur sur les machines virtuelles du labo.

En tant que propriétaire d’un labo, vous pouvez activer l’accès à toutes les machines virtuelles du labo via un navigateur. Vous n’avez pas besoin d’un client, d’un agent ou d’un logiciel supplémentaire. Azure Bastion fournit une connectivité RDP/SSH sécurisée et fluide à vos machines virtuelles directement dans le portail Azure via TLS. Lorsque vous vous connectez via Azure Bastion, vos machines virtuelles n’ont pas besoin d’une adresse IP publique. Pour plus d’informations, consultez la page [Présentation d’Azure Bastion](../bastion/bastion-overview.md).


Cet article vous indique comment activer la connexion du navigateur sur les machines virtuelles du labo.

## <a name="prerequisites"></a>Prérequis 
- Déployez un hôte bastion dans le réseau virtuel de votre labo existant **(OU)** connectez votre labo à un réseau virtuel configuré sur Bastion.
Pour découvrir comment déployer un hôte bastion dans un réseau virtuel, consultez la page [Créer un hôte bastion Azure](../bastion/tutorial-create-host-portal.md). Lorsque vous créez l’hôte Bastion, sélectionnez le réseau virtuel du labo. 
- L’utilisateur du labo doit disposer d’un rôle de **Lecteur** sur l’hôte Bastion et le réseau virtuel sur lequel Bastion a été configuré. 

## <a name="create-a-second-sub-net-in-the-bastion-virtual-network"></a>Créez un deuxième sous-réseau dans le réseau virtuel bastion
Tout d’abord, vous devez créer un deuxième sous-réseau dans le réseau virtuel bastion, car AzureBastionSubnet n’autorise pas la création de ressources autres que bastion. Créez un autre sous-réseau au sein du réseau virtuel bastion, comme indiqué dans l’image suivante :

![Deuxième sous-réseau dans le réseau virtuel Azure Bastion](./media/connect-virtual-machine-through-browser/second-subnet.png)

## <a name="enable-vm-creation-in-the-subnet"></a>Activez la création de machines virtuelles dans le sous-réseau
À présent, activez la création de machines virtuelles dans ce sous-réseau en procédant comme suit : 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu de navigation de gauche, sélectionnez **Tous les services**. 
1. Sélectionnez **DevTest Labs** dans la liste. 
1. Dans la liste de labos, sélectionnez *votre labo*. 

    > [!NOTE]
    > Azure Bastion est généralement disponible dans les régions suivantes : Usa West, USA Est, Europe Ouest, USA Centre Sud, Australie Est et Japon Est. Créez donc un labo dans l’une de ces régions s’il n’est pas dans l’une d’entre elles. 
    
1. Sélectionnez **Configuration et stratégies** dans la section **Paramètres** du menu à gauche. 
1. Sélectionner **Réseaux virtuels**.
1. Sélectionnez **Ajouter** depuis la barre d’outils. 
1. Sélectionnez le **réseau virtuel** sur lequel l’hôte bastion est déployé. 
1. Sélectionnez le sous-réseau pour les machines virtuelles, et non **AzureBastionSubnet**, l’autre que vous avez créé précédemment. Fermez la page et rouvrez-la si vous ne voyez pas le sous-réseau dans la liste au bas de l’écran. 

    ![Activez la création de machines virtuelles dans le sous-réseau](./media/connect-virtual-machine-through-browser/enable-vm-creation-subnet.png)
1. Sélectionnez l’option **Utiliser dans la création de machine virtuelle**. 
1. Sélectionnez **Enregistrer** dans la barre d’outils. 
1. Si vous avez un ancien réseau virtuel pour le labo, supprimez-le en sélectionnant * *…* et **Supprimer**. 

## <a name="enable-browser-connection"></a>Activer la connexion du navigateur 

Une fois que vous avez configuré un réseau virtuel Bastion dans le labo, vous pouvez, en tant que propriétaire du labo, activer la connexion du navigateur sur les machines virtuelles du labo.

Pour activer la connexion du navigateur sur les machines virtuelles du labo, procédez comme suit :

1. Accédez à *votre labo* dans le portail Azure.
1. Sélectionnez **Configuration et stratégies**.
1. Dans **Paramètres**, sélectionnez **Connexion du navigateur**. Si vous ne voyez pas cette option, fermez la page **Stratégies de configuration**, puis rouvrez-la. 

    ![Activer la connexion du navigateur](./media/enable-browser-connection-lab-virtual-machines/browser-connect.png)

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article suivant pour découvrir comment connecter à vos machines virtuelles à l’aide d’un navigateur : [Se connecter à votre machine virtuelle à l’aide d’un navigateur](connect-virtual-machine-through-browser.md)