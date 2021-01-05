---
title: Gérer un modèle de laboratoire de classe dans Azure Lab Services | Microsoft Docs
description: Découvrez comment créer et gérer un modèle de laboratoire de classe dans Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: be84a05fd8b278f5176cd768191d10a20fc719e6
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434275"
---
# <a name="create-and-manage-a-classroom-template-in-azure-lab-services"></a>Créer et gérer un modèle de laboratoire de classe dans Azure Lab Services
Le modèle de laboratoire est une image de machine virtuelle de base, à partir de laquelle toutes les machines virtuelles des utilisateurs sont créées. Configurez la machine virtuelle du modèle de façon qu’elle propose exactement ce que vous souhaitez fournir aux utilisateurs du laboratoire. Vous pouvez indiquer le nom et la description du modèle que voient les utilisateurs du laboratoire. Ensuite, publiez le modèle pour rendre les instances du modèle de machine virtuelle accessibles aux utilisateurs de votre laboratoire. Quand vous publiez un modèle, Azure Lab Services crée les machines virtuelles dans le laboratoire à l’aide du modèle. Le nombre de machines virtuelles créées dans ce processus est identique au nombre maximal d’utilisateurs autorisés dans le laboratoire, que vous pouvez définir dans la politique d’utilisation du laboratoire. Toutes les machines virtuelles ont la même configuration que le modèle.

Cet article explique comment créer et gérer un modèle de machine virtuelle dans un laboratoire de classe Azure Lab Services. 

> [!NOTE]
> Lorsque vous créez un laboratoire, le modèle de machine virtuelle est créé, mais il n’est pas démarré. Vous pouvez le démarrer, vous y connecter et installer les logiciels requis pour le laboratoire, puis le publier. Lorsque vous publiez le modèle de machine virtuelle, celui-ci s’arrête automatiquement si vous ne l’avez pas arrêté vous-même. 
> 
> Étant donné que les modèles de machines virtuelles engendrent des **coûts** lors de leur exécution, prenez soin de vérifier que le modèle de machine virtuelle est arrêté lorsque vous n’avez pas besoin de l’exécuter. 


## <a name="set-or-update-template-title-and-description"></a>Définir ou mettre à jour un titre et une description pour le modèle
Effectuez les étapes suivantes pour définir un titre et une description pour la première fois, et pour les mettre à jour par la suite. 

1. Dans la page **Modèle**, entrez le nouveau **titre** pour le laboratoire.  
2. Entrez la nouvelle **description** pour le modèle. Lorsque vous déplacez le focus hors de la zone de texte, elle est automatiquement enregistrée. 

    ![Nom et description du modèle](./media/how-to-create-manage-template/template-name-description.png)

## <a name="update-a-template-vm"></a>Mettre à jour un modèle de machine virtuelle
Suivez les étapes ci-dessous pour mettre à jour un modèle de machine virtuelle.  

1. Attendez le démarrage du modèle de machine virtuelle, sélectionnez le bouton **Se connecter au modèle** dans la barre d’outils pour vous connecter au modèle, et suivez les instructions. S’il s’agit d’une machine Windows, vous verrez une option permettant de télécharger le fichier RDP. 
1. Une fois que vous êtes connecté au modèle et que vous y apportez des modifications, il n’a plus la même configuration que les machines virtuelles dernièrement publiées pour vos utilisateurs. Les modifications de modèle sont reflétées sur les machines virtuelles de vos utilisateurs seulement une fois que vous republiez.

    ![Se connecter au modèle de machine virtuelle](./media/how-to-create-manage-template/connect-template-vm.png)
    
1. Installez les logiciels nécessaires pour que les étudiants puissent travailler en laboratoire (par exemple Visual Studio, l’Explorateur Stockage Azure, et ainsi de suite). 
1. Déconnectez-vous (fermez votre session Bureau à distance) du modèle de machine virtuelle. 
1. **Arrêtez** le modèle de machine virtuelle en sélectionnant **Arrêter le modèle**. 
1. Suivez les étapes de la section suivante pour **publier** le modèle de machine virtuelle mis à jour. 

## <a name="publish-the-template-vm"></a>Publier le modèle de machine virtuelle  
Au cours de cette étape, vous allez publier le modèle de machine virtuelle. Quand vous publiez le modèle de machine virtuelle, Azure Lab Services crée les machines virtuelles dans le labo à l’aide du modèle. Toutes les machines virtuelles ont la même configuration que le modèle.


1. Dans la page **Modèle**, sélectionnez **Publier** dans la barre d’outils. 

    ![Bouton Publier le modèle](./media/tutorial-setup-classroom-lab/template-page-publish-button.png)

    > [!WARNING]
    > Une fois que vous publiez, vous ne pouvez pas annuler la publication. 
2. Dans la page **Publier le modèle**, entrez le nombre de machines virtuelles que vous souhaitez créer dans le labo, puis sélectionnez **Publier**. 

    ![Publier le modèle - nombre de machines virtuelles](./media/tutorial-setup-classroom-lab/publish-template-number-vms.png)
3. Vous pouvez voir l’**état de publication** du modèle sur la page. Ce processus peut prendre jusqu’à une heure. 

    ![Publier un modèle - progression](./media/tutorial-setup-classroom-lab/publish-template-progress.png)
4. Attendez la fin de la publication, puis passez à la page **Pool de machines virtuelles** en sélectionnant **Machines virtuelles** dans le menu de gauche ou en sélectionnant la vignette **Machines virtuelles**. Vérifiez que des machines virtuelles se trouvent à l’état **Non affectée**. Ces machines virtuelles ne sont pas encore affectées aux étudiants. Elles doivent être à l’état **Arrêtée**. Vous pouvez démarrer la machine virtuelle d’un étudiant, vous y connecter, l’arrêter et la supprimer dans cette page. Vous pouvez démarrer les machines virtuelles dans cette page ou laisser les étudiants le faire. 

    ![Machines virtuelles à l’état Arrêtée](./media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)
## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [En tant qu’administrateur, créer et gérer des comptes de labo](how-to-manage-lab-accounts.md)
- [En tant que propriétaire de labo, créer et gérer des labos](how-to-manage-classroom-labs.md)
- [En tant que propriétaire de labo, configurer et contrôler l’utilisation d’un labo](how-to-configure-student-usage.md)
- [En tant qu’utilisateur de labo, accéder aux labos](how-to-use-classroom-lab.md)
