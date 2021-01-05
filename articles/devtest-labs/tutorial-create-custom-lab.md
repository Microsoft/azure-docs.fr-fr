---
title: Créer un laboratoire à l’aide de Azure DevTest Labs | Microsoft Docs
description: Dans ce tutoriel, vous créez un laboratoire dans Azure DevTest Labs à l’aide du portail Azure. Un administrateur de laboratoire définit un laboratoire, crée dans celui-ci des machines virtuelles et configure des stratégies.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: 4b59edd4ab1204d6683cf95a02070d10b1abe061
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91324278"
---
# <a name="tutorial-set-up-a-lab-by-using-azure-devtest-labs"></a>Didacticiel : configurer un laboratoire à l’aide de Azure DevTest Labs
Dans ce didacticiel, vous créez un laboratoire en utilisant le portail Azure. Un administrateur de laboratoire définit un laboratoire dans une organisation, il crée des machines virtuelles dans le laboratoire et configure des stratégies. Les utilisateurs du laboratoire (les développeurs et les testeurs par exemple) revendiquent des machines virtuelles dans le laboratoire, se connectent à eux et les utilisent. 

Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Création d’un laboratoire
> * Ajouter des machines virtuelles (VM) dans le laboratoire
> * Ajouter un utilisateur au rôle Utilisateur de laboratoire

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-a-lab"></a>Création d’un laboratoire
Les étapes suivantes montrent comment utiliser le portail Azure pour créer un laboratoire dans Azure DevTest Labs. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu principal sur le côté gauche, sélectionnez **Créer une ressource** (en haut de la liste), pointez sur **Outils de développement**, puis cliquez sur **Lab Services (préversion)** . 

    ![Menu Nouveau DevTest Lab](./media/tutorial-create-custom-lab/new-custom-lab-menu.png)
1. Dans la fenêtre **Créer un DevTest Lab**, effectuez les actions suivantes : 
    1. Pour **Nom du laboratoire**, saisissez un nom pour le laboratoire. 
    2. Pour **Abonnement**, sélectionnez l’abonnement dans lequel vous souhaitez créer le laboratoire. 
    3. Pour **Groupe de ressources**, sélectionnez **Créer un nouveau** et entrez un nom pour le groupe de ressources. 
    4. Pour **Emplacement**, sélectionnez la région ou l’emplacement où créer le laboratoire. 
    5. Sélectionnez **Create** (Créer). 
    6. Sélectionnez **Épingler au tableau de bord**. Après avoir créé le laboratoire, il s’affiche dans le tableau de bord. 

        ![Création d’un laboratoire de DevTest Labs](./media/tutorial-create-custom-lab/create-custom-lab-blade.png)
2. Vérifiez que le labo est correctement créé en examinant les notifications. Sélectionnez **Accéder à la ressource**.  

    ![Notification](./media/tutorial-create-custom-lab/creation-notification.png)
3. Vérifiez que vous voyez la page **DevTest Lab** de votre laboratoire. 

    ![Page d’accueil de votre laboratoire](./media/tutorial-create-custom-lab/lab-home-page.png)

## <a name="add-a-vm-to-the-lab"></a>Ajouter une machine virtuelle au laboratoire

1. Sur la page **DevTest Lab**, sélectionnez **+ Ajouter** sur la barre d’outils. 

    ![Bouton Ajouter](./media/tutorial-create-custom-lab/add-vm-to-lab-button.png)
1. Sur la page **Choisir une base**, recherchez avec un mot-clé (par exemple : Windows, Ubuntu) puis sélectionnez l’une des images de base dans la liste. 
1. Sur la page **Machine virtuelle**, effectuez les actions suivantes : 
    1. Pour **Nom de la machine virtuelle**, saisissez le nom de la machine virtuelle. 
    2. Pour **Nom d’utilisateur**, entrez un nom pour l’utilisateur qui a accès à la machine virtuelle. 
    3. Dans le champ **Mot de passe**, entrez le mot de passe de l’utilisateur. 

        ![Capture d’écran montrant les paramètres de base de la page « Créer une ressource de laboratoire ».](./media/tutorial-create-custom-lab/new-virtual-machine.png)
1. Sélectionnez l’onglet **Paramètres avancés**.
    1. Pour **Rendre cette machine exigible**, sélectionnez **Oui**.
    2. Vérifiez que le **nombre d’instances** est défini sur **1**. Si vous le définissez sur **2**, 2 machines virtuelles sont créées avec les noms : `<base image name>00' and <base image name>01`. Par exemple : `win10vm00` et `win10vm01`.     
    3. Sélectionnez **Envoyer**. 

        ![Choisir une base](./media/tutorial-create-custom-lab/new-vm-advanced-settings.png)
    9. Vous voyez l’état de la machine virtuelle dans la liste des **machines virtuelles exigibles**. La création de la machine virtuelle peut prendre environ 25 minutes. La machine virtuelle est créée dans un groupe de ressources Azure distinct, dont le nom commence par le nom du groupe de ressources actuel qui possède le laboratoire. Par exemple, si le laboratoire est dans `labrg`, la machine virtuelle peut être créée dans le groupe de ressources `labrg3988722144002`. 

        ![État de la création de la machine virtuelle](./media/tutorial-create-custom-lab/vm-creation-status.png)
1. Une fois la machine virtuelle créée, elle apparaît dans la liste des **machines virtuelles exigibles** dans la liste. 

    > [!NOTE] 
    > Dans la page **Paramètres avancés**, vous pouvez configurer une adresse IP publique, privée ou partagée pour la machine virtuelle. Quand le paramètre d’**adresse IP partagée** est activé, Azure DevTest Labs active automatiquement RDP pour les machines virtuelles Windows et SSH pour les machines virtuelles Linux. Si vous créez des machines virtuelles avec des adresses **IP publiques**, RDP et SSH sont activés sans aucune modification à partir de DevTest Labs.  

## <a name="add-a-user-to-the-lab-user-role"></a>Ajouter un utilisateur au rôle Utilisateur de laboratoire

1. Sélectionnez **Configuration et stratégies** sur le panneau de gauche. 

    ![Configuration et stratégies](./media/tutorial-create-custom-lab/configuration-and-policies-menu.png)
1. Sélectionnez **Contrôle d’accès (IAM)** dans le menu, puis sélectionnez **+ Ajouter une attribution de rôle** dans la barre d’outils. 

    ![Bouton Ajouter une attribution de rôle](./media/tutorial-create-custom-lab/add-role-assignment-button.png)
1. Dans la page **Ajouter des autorisations**, effectuez les actions suivantes :
    1. Pour **Rôle**, sélectionnez **Utilisateurs de DevTest Labs**. 
    2. Sélectionnez l’**utilisateur** que vous souhaitez ajouter. 
    3. Sélectionnez **Enregistrer**.

        ![Ajouter un utilisateur](./media/tutorial-create-custom-lab/add-user.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources
Le didacticiel suivant montre comment un utilisateur de laboratoire peut revendiquer et se connecter à une machine virtuelle dans le laboratoire. Si vous ne souhaitez pas suivre ce didacticiel, et pour nettoyer les ressources créées dans le cadre de ce didacticiel, procédez comme suit : 

1. Dans le portail Azure, sélectionnez **Groupes de ressources** dans le menu. 

    ![Groupes de ressources](./media/tutorial-create-custom-lab/resource-groups.png)
1. Sélectionnez le groupe de ressources dans lequel vous avez créé le laboratoire. 
1. Sélectionnez **Supprimer le groupe de ressources** dans la barre d’outils. Supprimer un groupe de ressources supprime toutes les ressources dans le groupe, notamment le laboratoire. 

    ![Groupe de ressources de laboratoire](./media/tutorial-create-custom-lab/lab-resource-group.png)
1. Répétez ces étapes pour supprimer le groupe de ressources supplémentaire créé pour vous, avec le nom `<your resource group name><random numbers>`. Par exemple : `splab3988722144001`. Les machines virtuelles sont créées dans ce groupe de ressources plutôt que dans le groupe de ressources au sein duquel se trouve le laboratoire. 

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez créé un laboratoire avec une machine virtuelle et vous avez donné un accès utilisateur au laboratoire. Pour savoir comment accéder au laboratoire en tant qu’un utilisateur du laboratoire, passer au didacticiel suivant :

> [!div class="nextstepaction"]
> [Didacticiel : Accéder au laboratoire](tutorial-use-custom-lab.md)

