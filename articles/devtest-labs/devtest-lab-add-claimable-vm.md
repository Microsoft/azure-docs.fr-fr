---
title: Créer et gérer des machines virtuelles revendiquables dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment utiliser le portail Azure pour ajouter une machine virtuelle revendicable dans Azure DevTest Labs, et voir les processus à suivre pour revendiquer/cesser de revendiquer une machine virtuelle.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1fe949d7104fe6437087e77c403cc0d921ebd025
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88270976"
---
# <a name="create-and-manage-claimable-vms-in-azure-devtest-labs"></a>Créer et gérer des machines virtuelles revendicables dans un laboratoire dans Azure DevTest Labs
Vous ajoutez une machine virtuelle exigible à un laboratoire comme vous [ajouteriez une machine virtuelle standard](devtest-lab-add-vm.md), à partir d’une *base* qui est une [image personnalisée](devtest-lab-create-template.md), une [formule](devtest-lab-manage-formulas.md) ou une [image de la plateforme Place de marché](devtest-lab-configure-marketplace-images.md). Ce didacticiel vous guide tout au long de l’utilisation du Portail Azure pour ajouter une machine virtuelle revendicable à un laboratoire dans DevTest Labs, et vous présente la procédure qu’un utilisateur suit pour revendiquer ou cesser de revendiquer la machine virtuelle.

## <a name="steps-to-add-a-claimable-vm-to-a-lab-in-azure-devtest-labs"></a>Procédure d’ajout d’une machine virtuelle exigible à un laboratoire dans Azure DevTest Labs
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
3. Sélectionnez **Créer** pour ajouter la machine virtuelle spécifiée au laboratoire.

   La page du laboratoire affiche l’état de la création de la machine virtuelle, tout d’abord sous la forme **Création en cours**, puis sous la forme **En cours d’exécution** après le démarrage de la machine virtuelle.

> [!NOTE]
> Si vous déployez des machines virtuelles via des  [Modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md), vous pouvez créer des machines virtuelles exigibles qui peuvent être revendiquées en définissant la propriété **allowClaim** sur true dans la section Propriétés.


## <a name="using-a-claimable-vm"></a>Utilisation d’une machine virtuelle exigible

Un utilisateur peut revendiquer toute machine virtuelle dans la liste des « Machines virtuelles exigibles » en procédant comme suit :

* Dans la liste des « machines virtuelles revendicables », en bas du volet « Vue d’ensemble » du laboratoire, cliquez avec le bouton droit sur une des machines virtuelles, puis choisissez **Revendiquer la machine**.

  ![Demandez une machine virtuelle exigible spécifique.](./media/devtest-lab-add-vm/devtestlab-claim-VM.png)


* En haut du volet « Vue d’ensemble », choisissez **Revendiquer n’importe laquelle**. Une machine virtuelle aléatoire est attribuée dans la liste des machines virtuelles qui peuvent être revendiquées.

  ![Demandez n’importe quelle machine virtuelle exigible.](./media/devtest-lab-add-vm/devtestlab-claim-any.png)


Après qu'un utilisateur revendique une machine virtuelle, DevTest Labs démarre la machine et la déplace vers le haut dans la liste des utilisateurs du laboratoire « Mes machines virtuelles ». Cela signifie que l'utilisateur du laboratoire aura désormais des privilèges de propriétaire sur cette machine. Le temps requis pour cette étape peut varier selon le temps de démarrage et toute autre action personnalisée effectuée pendant l'événement de revendication. Une fois revendiquée, la machine n'est plus disponible dans le pool revendicable.  

## <a name="unclaim-a-vm"></a>Cesser de revendiquer une machine virtuelle

Quand un utilisateur a fini d’utiliser une machine virtuelle revendiquée qu’il souhaite rendre disponible pour une autre personne, il peut renvoyer la machine virtuelle revendiquée à la liste des machines virtuelles revendiquées en procédant de l’une des manières suivantes :

- Dans la liste « Mes machines virtuelles », cliquez avec le bouton droit sur l’une des machines virtuelles (ou sélectionnez ses points de suspension (...)), puis choisissez **Cesser de revendiquer**.

  ![Cessez de revendiquer une machine virtuelle figurant dans la liste des machines virtuelles.](./media/devtest-lab-add-vm/devtestlab-unclaim-VM2.png)

- Dans la liste « Mes machines virtuelles », sélectionnez une machine virtuelle pour ouvrir son volet de gestion, puis sélectionnez **Cesser de revendiquer** dans la barre de menus supérieure.

  ![Cessez de revendiquer un machine virtuelle dans le volet de gestion de celle-ci.](./media/devtest-lab-add-vm/devtestlab-unclaim-VM.png)

Lorsqu'un utilisateur cesse de revendiquer une machine virtuelle, il n'a plus de droits de propriété pour cette machine virtuelle de laboratoire spécifique, et celle-ci peut être revendiquée par tout autre utilisateur du laboratoire dans l'état où elle a été replacée dans le pool. 

### <a name="transferring-the-data-disk"></a>Transfert du disque de données
Si un utilisateur cesse de revendiquer une machine virtuelle revendicable disposant d’un disque de données attaché, celui-ci reste avec la machine virtuelle. Ensuite, quand un autre utilisateur revendique cette machine virtuelle, il revendique également le disque de données.

C’est ce qu’on appelle le « transfert du disque de données ». Le disque de données devient alors disponible dans la liste **Mes disques de données** du nouvel utilisateur qui peut ainsi le gérer.

![Cessez de revendiquer des disques de données.](./media/devtest-lab-add-vm/devtestlab-unclaim-datadisks.png)



## <a name="next-steps"></a>Étapes suivantes
* Une fois la machine virtuelle crée, vous pouvez vous y connecter en sélectionnant **Se connecter** dans son volet de gestion.
* Explorez la [Galerie de modèles de démarrage rapide d’Azure Resource Manager DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
