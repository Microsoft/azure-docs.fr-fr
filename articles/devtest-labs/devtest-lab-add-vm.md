---
title: Ajouter une machine virtuelle à un laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment ajouter le portail Azure pour ajouter une machine virtuelle à un labo dans Azure DevTest Labs. Vous pouvez choisir une base qui est soit une image personnalisée, soit une formule.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 07efcf93b963009494021b90e6c730c4615301ef
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91297621"
---
# <a name="add-a-vm-to-a-lab-in-azure-devtest-labs"></a>Ajouter une machine virtuelle à un laboratoire dans Azure DevTest Labs
Si vous avez déjà [créé votre première machine virtuelle](tutorial-create-custom-lab.md#add-a-vm-to-the-lab), vous l’avez probablement fait à partir d’une [image de la Place de marché](devtest-lab-configure-marketplace-images.md) préchargée. Maintenant, si vous souhaitez ajouter d’autres machines virtuelles à votre laboratoire, vous pouvez également choisir une *base* qui est soit une [image personnalisée](devtest-lab-create-template.md) soit une [formule](devtest-lab-manage-formulas.md). Ce didacticiel vous guide tout au long de l’utilisation du Portail Azure pour ajouter une machine virtuelle à un laboratoire dans DevTest Labs.

Cet article vous montre également comment gérer les artefacts d’une machine virtuelle dans votre laboratoire.

## <a name="steps-to-add-a-vm-to-a-lab-in-azure-devtest-labs"></a>Procédure d’ajout d’une machine virtuelle à un laboratoire dans Azure DevTest Labs
1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la section **DEVOPS**. Si vous sélectionnez * (étoile) à côté de **DevTest Labs** dans la section **DEVOPS**, cette action ajoute **DevTest Labs** au menu de navigation de gauche afin que vous puissiez y accéder facilement la prochaine fois. Vous pouvez ensuite sélectionner **DevTest Labs** dans le menu de navigation de gauche.

    ![Tous les services - Sélectionner DevTest Labs](./media/devtest-lab-create-lab/all-services-select.png)
1. Dans la liste des laboratoires, sélectionnez le laboratoire dans lequel vous souhaitez créer la machine virtuelle.
2. Dans la page **Vue d’ensemble** du laboratoire, sélectionnez **+ Ajouter**.

    ![Ajout du bouton de machine virtuelle](./media/devtest-lab-add-vm/devtestlab-home-blade-add-vm.png)
1. Dans la page **Choisir une base**, sélectionnez une image de la Place de marché pour la machine virtuelle.
1. Sur l’onglet **Paramètres de base** de la page **Machine virtuelle**, effectuez les actions suivantes :
    1. Entrez un nom pour la machine virtuelle dans la zone de texte **Nom de la machine virtuelle**. La zone de texte est préremplie avec un nom unique généré automatiquement. Le nom correspond au nom d’utilisateur de votre adresse e-mail, suivi d’un numéro unique à trois chiffres. Cette fonctionnalité vous évite d’avoir à penser à un nom d’ordinateur et à le taper chaque fois que vous créez une machine. Vous pouvez remplacer cette valeur renseignée automatiquement par un nom de votre choix si vous le souhaitez. Pour remplacer le nom de machine virtuelle renseigné automatiquement, entrez un nom dans la zone de texte **Nom de la machine virtuelle**.
    2. Entrez un **nom d’utilisateur** qui obtient les privilèges d’administrateur sur la machine virtuelle. Le **nom d’utilisateur** de la machine est prérenseigné avec un nom unique généré automatiquement. Le nom correspond au nom d’utilisateur de votre adresse e-mail. Cette fonctionnalité vous évite d’avoir à penser à un nom d’utilisateur chaque fois que vous créez une machine. Là encore, vous pouvez remplacer cette valeur renseignée automatiquement par un nom d’utilisateur de votre choix si vous le souhaitez. Pour remplacer le nom d’utilisateur renseigné automatiquement, entrez une valeur dans la zone de texte **Nom d’utilisateur**. Cet utilisateur obtient des privilèges d’**administrateur** sur la machine virtuelle.
    3. Si vous créez la première machine virtuelle dans le laboratoire, entrez un **mot de passe** pour l’utilisateur. Pour enregistrer ce mot de passe comme mot de passe par défaut dans le coffre de clés Azure associé au laboratoire, sélectionnez **Enregistrer comme mot de passe par défaut**. Le mot de passe par défaut est enregistré dans le coffre de clés sous le nom : **VmPassword**. Quand vous essaierez de créer d’autres machines virtuelles dans le laboratoire, **VmPassword** sera sélectionné automatiquement comme **mot de passe**. Pour remplacer la valeur, décochez la case **Utiliser un secret enregistré**, puis entrez un mot de passe.

        ![Capture d’écran montrant la page « Créer une ressource de lab » avec l’option « Paramètres de base » sélectionnée et les valeurs « Paramètres utilisateur » entrées.](./media/tutorial-create-custom-lab/new-virtual-machine.png)

        Vous pouvez également enregistrer d’abord les secrets dans le coffre de clés, puis les utiliser lors de la création d’une machine virtuelle dans le laboratoire. Pour plus d’informations, consultez [Stocker des secrets dans un coffre de clés](devtest-lab-store-secrets-in-key-vault.md). Pour utiliser le mot de passe stocké dans le coffre de clés, sélectionnez **Utiliser un secret enregistré** et spécifiez une valeur de clé correspondant à votre secret (mot de passe).
    4. Dans la section **Autres options**, sélectionnez **Modifier la taille**. Sélectionnez l’un des éléments prédéfinis qui spécifient les cœurs du processeur, la taille de la RAM et la taille du disque dur de la machine virtuelle à créer.
    5. Sélectionnez **Ajouter ou supprimer des artefacts**. Sélectionnez et configurez les artefacts que vous souhaitez ajouter à l’image de base.
    **Remarque :** Si vous n’êtes pas familier avec DevTest Labs ou avec la configuration d’artefacts, reportez-vous à la section [Ajout d’un artefact existant à une machine virtuelle](./devtest-lab-add-vm.md#add-an-existing-artifact-to-a-vm), puis reprenez la procédure à ce stade.
2. Basculez vers l’onglet **Paramètres avancés** en haut et effectuez les actions suivantes :
    1. Pour changer le réseau virtuel où se trouve la machine virtuelle, sélectionnez **Changer le réseau virtuel**.
    2. Pour changer de sous-réseau, sélectionnez **Changer de sous-réseau**.
    3. Spécifiez si l’adresse IP de la machine virtuelle est **publique, privée ou partagée**.
    4. Pour supprimer automatiquement la machine virtuelle, spécifiez **la date et l’heure d’expiration**.
    5. Pour rendre la machine virtuelle revendicable par un utilisateur de laboratoire, sélectionnez **Oui** pour l’option **Make this machine claimable** (Rendre cette machine revendicable).
    6. Spécifiez le nombre d’**instances de machine virtuelle** que vous souhaitez rendre disponibles aux utilisateurs de votre laboratoire.

        ![Choisir une base](./media/tutorial-create-custom-lab/new-vm-advanced-settings.png)
1. Sélectionnez **Créer** pour ajouter la machine virtuelle spécifiée au laboratoire.

   La page du laboratoire affiche l’état de la création de la machine virtuelle, tout d’abord sous la forme **Création en cours**, puis sous la forme **En cours d’exécution** après le démarrage de la machine virtuelle.

    ![État de la création de la machine virtuelle](./media/tutorial-create-custom-lab/vm-creation-status.png)

## <a name="add-an-existing-artifact-to-a-vm"></a>Ajout d’un artefact existant à une machine virtuelle
Au cours de la création d’une machine virtuelle, vous pouvez ajouter des artefacts existants. Chaque laboratoire comprend des artefacts provenant du dépôt d’artefacts DevTest Labs public, ainsi que les artefacts que vous avez créés et ajoutés à votre propre dépôt d’artefacts.

* Les *artefacts* Azure DevTest Labs vous permettent de spécifier les *actions* qui sont effectuées lorsque la machine virtuelle est approvisionnée (exécution de scripts Windows PowerShell, exécution de commandes Bash, installation de logiciel, etc.).
* Les *paramètres* de l’artefact vous permettent de personnaliser l’artefact pour votre scénario particulier

Pour découvrir comment créer des artefacts, consultez l’article [Découvrez comment créer vos propres artefacts pour les utiliser avec DevTest Labs](devtest-lab-artifact-author.md).

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
1. Dans la liste des laboratoires, sélectionnez le laboratoire contenant la machine virtuelle avec laquelle vous souhaitez travailler.
1. Sélectionnez **Mes machines virtuelles**.
1. Sélectionnez la machine virtuelle qui vous intéresse.
1. Sélectionnez **Gérer les artefacts**.
1. Sélectionnez **Appliquer des artefacts**.
1. Dans le volet **Appliquer des artefacts**, sélectionnez l’artefact que vous souhaitez ajouter à la machine virtuelle.
1. Dans le volet **Ajouter un artefact**, entrez les valeurs de paramètre requises et tous les paramètres facultatifs dont vous avez besoin.
1. Sélectionnez **Ajouter** pour ajouter l’artefact et revenir au volet **Appliquer des artefacts**.
1. Continuez à ajouter des artefacts en fonction des besoins de votre machine virtuelle.
1. Une fois que vous avez ajouté vos artefacts, vous pouvez [modifier l’ordre dans lequel les artefacts sont exécutés](#change-the-order-in-which-artifacts-are-run). Vous pouvez également revenir à [Afficher ou modifier un artefact](#view-or-modify-an-artifact).
1. Lorsque vous avez terminé d’ajouter des artefacts, sélectionnez **Appliquer**.

## <a name="change-the-order-in-which-artifacts-are-run"></a>Modification de l'ordre dans lequel les artefacts sont exécutés
Par défaut, les actions des artefacts sont exécutées dans l'ordre dans lequel ceux-ci sont ajoutés à la machine virtuelle.
Les étapes suivantes montrent comment modifier l’ordre dans lequel les artefacts sont exécutés.

1. En haut du volet **Appliquer des artefacts**, sélectionnez le lien indiquant le nombre d’artefacts ajoutés à la machine virtuelle.

    ![Nombre d’artefacts ajoutés à la machine virtuelle](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. Dans le volet **Artefacts sélectionnés**, effectuez un glisser-déplacer sur les artefacts dans l’ordre souhaité. **Remarque :** si vous avez des difficultés à faire glisser l’artefact, vérifiez que vous effectuez cette opération à partir du côté gauche de l’artefact.
1. Cliquez sur **OK** quand vous avez terminé.

## <a name="view-or-modify-an-artifact"></a>Afficher ou modifier un artefact
Les étapes suivantes montrent comment afficher ou modifier les paramètres d’un artefact :

1. En haut du volet **Appliquer des artefacts**, sélectionnez le lien indiquant le nombre d’artefacts ajoutés à la machine virtuelle.

    ![Nombre d’artefacts ajoutés à la machine virtuelle](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. Dans le volet **Artefacts sélectionnés**, sélectionnez l’artefact que vous voulez visualiser ou modifier.
1. Dans le volet **Ajouter un artefact**, apportez les modifications nécessaires, puis cliquez sur **OK** pour fermer le volet **Ajouter un artefact**.
1. Cliquez sur **OK** pour fermer le volet **Artefacts sélectionnés**.

## <a name="save-azure-resource-manager-template"></a>Enregistrer un modèle Azure Resource Manager
Un modèle Azure Resource Manager constitue un moyen déclaratif de définir un déploiement qui peut être répété.
Les étapes suivantes expliquent comment enregistrer le modèle Azure Resource Manager pour la machine virtuelle en cours de création.
Une fois enregistré, vous pouvez utiliser le modèle Azure Resource Manager pour [déployer de nouvelles machines virtuelles avec Azure PowerShell](../azure-resource-manager/templates/overview.md).

1. Dans le volet **Machine virtuelle**, sélectionnez **Afficher le modèle Azure Resource Manager**.
2. Dans le volet **Afficher le modèle Azure Resource Manager**, sélectionnez le texte du modèle.
3. Copiez le texte sélectionné dans le Presse-papiers.
4. Cliquez sur **OK** pour fermer le volet **Afficher le modèle Azure Resource Manager**.
5. Ouvrez un éditeur de texte.
6. Collez le texte du modèle depuis le Presse-papiers.
7. Enregistrez le fichier pour une utilisation ultérieure.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
* Une fois la machine virtuelle créée, vous pouvez vous y connecter en sélectionnant **Connexion** dans le volet de la machine virtuelle.
* Découvrez comment [créer des artefacts personnalisés pour vos machines virtuelles DevTest Labs](devtest-lab-artifact-author.md).
* Explorez la [Galerie de modèles de démarrage rapide d’Azure Resource Manager DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
