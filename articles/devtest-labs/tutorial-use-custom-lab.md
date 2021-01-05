---
title: Accéder à un laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Dans ce didacticiel, vous accédez au laboratoire créé à l’aide d’Azure DevTest Labs, revendiquez des machines virtuelles, vous les utilisez, puis vous cessez de les revendiquer.
ms.topic: tutorial
ms.date: 06/26/2020
ms.author: spelluru
ms.openlocfilehash: b4477e0b98ef534b8170ee297edf88ac6fa62dd7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85476442"
---
# <a name="tutorial-access-a-lab-in-azure-devtest-labs"></a>Didacticiel : accéder à un laboratoire dans Azure DevTest Labs
Dans ce didacticiel, vous utilisez le laboratoire créé dans le [didacticiel : créer un laboratoire dans Azure DevTest Labs](tutorial-create-custom-lab.md).

Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Revendiquer une machine virtuelle dans le laboratoire
> * Connexion à la machine virtuelle
> * Cesser de revendiquer la machine virtuelle

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="access-the-lab"></a>Accéder à l’atelier

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Toutes les ressources** dans le menu de gauche. 
3. Sélectionnez **DevTest Labs** comme type de ressource. 
4. Sélectionnez le laboratoire. 

    ![Sélectionner le laboratoire](./media/tutorial-use-custom-lab/search-for-select-custom-lab.png)

## <a name="claim-a-vm"></a>Revendiquer une machine virtuelle

1. Dans la liste des **machines virtuelles pouvant être réclamées**, sélectionnez **...** (points de suspension), puis sélectionnez **Revendiquer la machine**.

    ![Revendiquer une machine virtuelle](./media/tutorial-use-custom-lab/claim-virtual-machine.png)
1. Vérifiez que la présence de la machine virtuelle dans la liste **Mes machines virtuelles**.

    ![Ma machine virtuelle](./media/tutorial-use-custom-lab/my-virtual-machines.png)

## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

1. Sélectionnez votre machine virtuelle dans la liste. Vous voyez la **page de la machine virtuelle** pour votre machine virtuelle. Sélectionnez **Se connecter** sur la barre d’outils.

    ![Connexion à la machine virtuelle](./media/tutorial-use-custom-lab/connect-button.png)
2. Enregistrer le fichier **RDP** téléchargé sur votre disque dur et utilisez-le pour vous connecter à la machine virtuelle. Spécifiez le nom d’utilisateur et le mot de passe que vous avez indiqués en créant la machine virtuelle à la section précédente. 

    Pour vous connecter à une machine virtuelle Linux, l’accès par protocole SSH et/ou RDP doit être activé pour la machine virtuelle. Pour les étapes à suivre pour vous connecter à une machine virtuelle Linux via un protocole RDP, consultez [Installer et configurer le Bureau à distance pour se connecter à une machine virtuelle Linux dans Azure](../virtual-machines/linux/use-remote-desktop.md). 

    > [!NOTE]
    > Il existe d’autres façons d’accéder à la page de votre machine virtuelle. En voici quelques-unes : 
    > 
    > 1. Recherchez toutes les machines virtuelles de votre abonnement. Sélectionnez votre machine virtuelle dans la liste pour accéder à la page **Machine virtuelle**.
    > 2. Accédez à la page **Groupe de ressources** pour le groupe de ressources. Ensuite, sélectionnez votre machine virtuelle dans la liste des ressources du groupe de ressources pour accéder à la page **Machine virtuelle**. 
    >
    > N’utilisez pas le bouton **Se connecter** de la barre d’outils de la page **Machine virtuelle** à laquelle vous accédez à l’aide de ces options. Au lieu de cela, accédez à la page **Machine virtuelle** à partir de la page **DevTest Labs**, comme indiqué dans cet article, puis utilisez le bouton **Se connecter** dans la barre d’outils.


## <a name="unclaim-the-vm"></a>Cesser de revendiquer la machine virtuelle
Une fois que vous avez fini d’utiliser la machine virtuelle, cessez de la revendiquer en suivant ces étapes : 

1. Sur la page de la machine virtuelle, sélectionnez **Cesser de revendiquer** sur la barre d’outils. 

    ![Cesser de revendiquer une machine virtuelle](./media/tutorial-use-custom-lab/unclaim-vm-menu.png)
1. La machine virtuelle est arrêtée avant que la revendication ne cesse. Vous pouvez voir l’état de cette opération dans les notifications.  
3. Revenez à la page DevTest Lab en cliquant sur le nom de votre laboratoire, en haut dans le menu de navigation. 
    
    ![Revenir au laboratoire](./media/tutorial-use-custom-lab/breadcrumb-to-lab.png)
1. Vérifiez que vous voyez la machine virtuelle dans la liste des **machines virtuelles revendicables** en bas.

    
## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous a montré comment accéder à et utiliser un laboratoire créé à l’aide d’Azure DevTest Labs. Pour plus d’informations sur l’accès et l’utilisation de machines virtuelles dans un laboratoire personnalisé, consultez 

> [!div class="nextstepaction"]
> [Procédure : utiliser des machines virtuelles dans un laboratoire](devtest-lab-add-vm.md)

