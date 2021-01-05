---
title: 'Démarrage rapide : Activer l’Autogestion Azure pour machines virtuelles dans le portail Azure'
description: Découvrez comment activer rapidement l’Autogestion pour une machine virtuelle nouvelle ou existante dans le portail Azure.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: quickstart
ms.date: 09/04/2020
ms.author: jushiman
ms.openlocfilehash: 69f43b626bb50171ceaca1b7a8761bec040d1dd6
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92897226"
---
# <a name="quickstart-enable-azure-automanage-for-virtual-machines-in-the-azure-portal"></a>Démarrage rapide : Activer l’Autogestion Azure pour machines virtuelles dans le portail Azure

Utilisez l’Autogestion Azure dans le portail Azure afin d’activer la gestion automatique sur une machine virtuelle nouvelle ou existante.


## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) avant de commencer.

> [!NOTE]
> Les comptes associés à un essai gratuit n’ont pas accès aux machines virtuelles utilisées dans ce tutoriel. Veuillez passer à un abonnement avec paiement à l’utilisation.

> [!IMPORTANT]
> Vous devez disposer du rôle **Contributeur** sur le groupe de ressources contenant vos machines virtuelles pour activer Automanage à l’aide d’un compte Automanage existant. Si vous activez Automanage avec un nouveau compte Automanage, vous devez disposer des autorisations suivantes : Rôle de **Propriétaire** , ou rôle de **Contributeur** associé au rôle d’ **Administrateur de l’accès utilisateur** sur votre abonnement.


## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/).


## <a name="enable-automanage-for-vms-on-an-existing-vm"></a>Activer l’Autogestion pour machines virtuelles sur une machine virtuelle existante

1. Dans la barre de recherche, recherchez puis sélectionnez **Autogestion – Bonnes pratiques pour les machines virtuelles**.

2. Sélectionnez l’option **Activer sur la machine virtuelle existante**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\zero-vm-list-view.png" alt-text="Activation sur la machine virtuelle existante":::

3. Dans le panneau **Sélectionner les machines**  :
    1. Filtrez la liste des machines virtuelles selon votre **abonnement** et votre **groupe de ressources**.
    1. Cochez la case des machines virtuelles que vous souhaitez intégrer.
    1. Cliquez sur le bouton **Sélectionner**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\existing-vm-select-machine.png" alt-text="Sélectionnez une machine virtuelle existante dans la liste des machines virtuelles disponibles.":::

4. Sous **Profil de configuration** , cliquez sur **Parcourir et modifier les profils et les préférences**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\existing-vm-quick-create.png" alt-text="Parcourir et modifier les profils et les préférences":::

5. Dans le panneau **Sélectionner le profil de configuration + préférences**  :
    1. Sélectionnez un profil sur la gauche : *Dev/Test* à des fins de test, *Prod* pour la production.
    1. Cliquez sur le bouton **Sélectionner**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\browse-production-profile.png" alt-text="Parcourir le profil de configuration de production":::

6. Cliquez sur le bouton **Activer**.


## <a name="enable-automanage-for-vms-on-a-new-vm"></a>Activer l’Autogestion pour machines virtuelles sur une nouvelle machine virtuelle

Connectez-vous au portail Azure à partir d'[ici](https://aka.ms/automanageportalnextstep) pour créer une nouvelle machine virtuelle et activer Automanage.

1. Effectuez les étapes de création fournies dans [Démarrage rapide : Créer une machine virtuelle Windows dans le portail Azure](..\virtual-machines\windows\quick-create-portal.md).

2. Une fois votre machine virtuelle déployée, vous verrez la page d’état du déploiement au bas de laquelle se trouvent les **Étapes suivantes** qu’il est recommandé d’effectuer.

    :::image type="content" source="media\quick-create-virtual-machine-portal\create-next-steps.png" alt-text="Section Étapes suivantes, située au bas de la page de déploiement":::

3. Sous **Étapes suivantes** , sélectionnez **Activer l’Autogestion pour les machines virtuelles – Bonnes pratiques**.

4. Dans la page **Autogestion – Bonnes pratiques pour les machines virtuelles** , la machine virtuelle nouvellement créée sera automatiquement ajoutée à la section **Machines**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\create-new-enable-overview.png" alt-text="La machine virtuelle nouvellement créée s’affichera comme la machine sélectionnée":::

5. Sous **Profil de configuration** , cliquez sur **Parcourir et modifier les profils et les préférences**.

6. Dans le panneau **Sélectionner le profil de configuration + préférences**  :
    1. Sélectionnez un profil sur la gauche : *Dev/Test* à des fins de test, *Prod* pour la production.
    1. Cliquez sur le bouton **Sélectionner**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\browse-production-profile.png" alt-text="Parcourir le profil de configuration de production":::

7. Cliquez sur le bouton **Activer**.

## <a name="disable-automanage-for-vms"></a>Désactiver l’Autogestion pour machines virtuelles

Vous pouvez cesser rapidement d’utiliser l’Autogestion Azure pour machines virtuelles en désactivant la gestion automatique.

:::image type="content" source="media\automanage-virtual-machines\disable-step-1.png" alt-text="Désactivation de l’Autogestion sur une machine virtuelle":::

1. Accédez à la page **Autogestion – Bonnes pratiques pour les machines virtuelles** qui liste toutes vos machines virtuelles qui sont gérées automatiquement.
1. Cochez la case des machines virtuelles que vous souhaitez désactiver.
1. Cliquez sur le bouton **Désactiver la gestion automatique**.
1. Lisez attentivement le message dans la fenêtre contextuelle qui s’affiche avant d’accepter de **Désactiver**.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez créé un nouveau groupe de ressources pour essayer l’Autogestion Azure pour machines virtuelles, et que vous n’en avez plus besoin, vous pouvez le supprimer. La suppression du groupe supprime également la machine virtuelle, ainsi que toutes les autres ressources se trouvant dans le groupe.

L’Autogestion Azure crée des groupes de ressources par défaut dans lesquels stocker les ressources. Vérifiez les groupes de ressources qui suivent la convention de nommage « DefaultResourceGroupRegionName » ou « AzureBackupRGRegionName » afin de nettoyer toutes les ressources.

1. Sélectionnez le **groupe de ressources**.
1. Dans la page du groupe de ressources, sélectionnez **Supprimer**.
1. Lorsque vous y êtes invité, confirmez le nom du groupe de ressources, puis sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez activé l’Autogestion Azure pour machines virtuelles. 

Découvrez comment vous pouvez créer et appliquer des préférences personnalisées lors de l’activation de l’Autogestion sur votre machine virtuelle. 

> [!div class="nextstepaction"]
> [Autogestion Azure pour machines virtuelles - Profil de configuration personnalisé](virtual-machines-custom-preferences.md)
