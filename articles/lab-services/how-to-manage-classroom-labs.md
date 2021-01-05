---
title: Gestion des labos dans Azure Lab Services | Microsoft Docs
description: Découvrez comment créer et configurer un labo de classe, afficher tous les labos, partager le lien d’inscription avec un utilisateur de labo et supprimer un labo.
ms.topic: how-to
ms.date: 06/26/2020
ms.openlocfilehash: db7a2f58f99252a7e5076dd86c37b65bbe8ea37a
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434207"
---
# <a name="manage-labs-in-azure-lab-services"></a>Gestion des labos dans Azure Lab Services 
Cet article décrit comment créer et supprimer un laboratoire de classe. Il montre également comment afficher tous les labos d’un compte de labo. 

## <a name="prerequisites"></a>Prérequis
Pour configurer un laboratoire de classe dans un compte de laboratoire, vous devez être membre du rôle **Créateur de laboratoire** dans le compte de laboratoire. Le compte utilisé pour créer un compte de laboratoire est ajouté automatiquement à ce rôle. Un propriétaire de laboratoire peut ajouter d’autres utilisateurs au rôle Créateur de laboratoire à l’aide des étapes fournies dans l’article suivant : [Ajouter un utilisateur au rôle Créateur de laboratoire](tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role).

## <a name="create-a-classroom-lab"></a>Créer un laboratoire de classe

1. Accédez au [site web Azure Lab Services](https://labs.azure.com). Internet Explorer 11 n’est pas encore pris en charge. 
1. Sélectionnez **Se connecter** et entrez vos informations d’identification. Sélectionnez ou entrez un **ID utilisateur** qui est un membre du rôle **Créateur de laboratoire** dans le compte de laboratoire, puis entrez un mot de passe. Azure Lab Services prend en charge les comptes professionnels et les comptes Microsoft. 
1. Sélectionnez **Nouveau labo**. 
    
    ![Créer un laboratoire de classe](./media/tutorial-setup-classroom-lab/new-lab-button.png)
1. Dans la fenêtre **New Lab** (Nouveau laboratoire), effectuez les actions suivantes : 
    1. Spécifiez un **nom** pour votre laboratoire. 
    1. Sélectionnez la **taille des machines virtuelles**  dont vous avez besoin pour la classe. Pour obtenir la liste des tailles disponibles, consultez la section [Tailles de machine virtuelle](#vm-sizes). 
    1. Sélectionnez **l’image de machine virtuelle** que vous souhaitez utiliser pour le laboratoire de classe. Si vous sélectionnez une image Linux, vous voyez une option pour **activer la connexion Bureau à distance**. Pour plus d’informations, consultez [Activer la connexion Bureau à distance pour Linux](how-to-enable-remote-desktop-linux.md).

        Si vous vous êtes connecté avec les informations d’identification du propriétaire du compte Lab, vous verrez une option permettant d’activer d’autres images pour le lab. Pour plus d’informations, consultez [Activer des images au moment de la création d’un lab](specify-marketplace-images.md#enable-images-at-the-time-of-lab-creation).
    1. Vérifiez le **tarif horaire total** affiché sur la page. 
    1. Sélectionnez **Enregistrer**.

        ![Capture d’écran montrant la fenêtre « Nouveau Labo ».](./media/tutorial-setup-classroom-lab/new-lab-window.png)

        > [!NOTE]
        > Vous voyez une option permettant de sélectionner un emplacement pour votre lab si le compte de laboratoire a été configuré pour l’option [Autoriser le créateur du lab à choisir l’emplacement du lab](allow-lab-creator-pick-lab-location.md). 
4. Dans la page **Informations d’identification de machine virtuelle**, spécifiez les informations d’identification par défaut pour toutes les machines virtuelles du labo.
    1. Spécifiez le **nom d’utilisateur** pour toutes les VM du laboratoire.
    2. Spécifiez le **mot de passe** de l’utilisateur. 

        > [!IMPORTANT]
        > Notez le nom d'utilisateur et le mot de passe que vous créez. Ils ne s’afficheront plus.
    3. Désactivez l’option **Use same password for all virtual machines** (Utiliser le même mot de passe pour toutes les machines virtuelles) si vous souhaitez que les étudiants définissent leurs propres mots de passe. Cette étape est **facultative**. 

        Un formateur peut choisir d’utiliser le même mot de passe pour toutes les machines virtuelles du laboratoire ou d’autoriser les étudiants à définir des mots de passe pour leurs machines virtuelles. Par défaut, ce paramètre est activé pour toutes les images Windows et Linux, à l’exception d’Ubuntu. Lorsque vous sélectionnez une machine virtuelle **Ubuntu**, ce paramètre est désactivé. Les étudiants sont ainsi invités à définir un mot de passe lorsqu’ils se connectent à la machine virtuelle pour la première fois.  

        ![Fenêtre Nouveau labo](./media/tutorial-setup-classroom-lab/virtual-machine-credentials.png)
    4. Ensuite, sélectionnez **Suivant** dans la page **Informations d’identification de la machine virtuelle**. 
5. Dans la page **Stratégies du lab**, procédez comme suit :
    1. Entrez le nombre d’heures allouées à chaque utilisateur (**quota pour chaque utilisateur**) en dehors des heures planifiées pour le labo. 
    2. Pour l’option **Arrêt automatique des machines virtuelles**, indiquez si vous souhaitez que la machine virtuelle soit automatiquement arrêtée lorsque l’utilisateur se déconnecte. Vous pouvez également spécifier la durée pendant laquelle la machine virtuelle doit attendre que l’utilisateur se reconnecte avant de s’arrêter automatiquement. Pour plus d’informations, consultez [Activer l’arrêt automatique des machines virtuelles lors de la déconnexion](how-to-enable-shutdown-disconnect.md).
    3. Ensuite, sélectionnez **Terminer**. 

        ![Quota pour chaque utilisateur](./media/tutorial-setup-classroom-lab/quota-for-each-user.png)
    
5. Vous devez voir l’écran suivant qui indique l’état de la création du modèle de machine virtuelle. La création du modèle dans le laboratoire prend jusqu'à 20 minutes. 

    ![État de la création du modèle de machine virtuelle](./media/tutorial-setup-classroom-lab/create-template-vm-progress.png)
8. Dans la page **Modèle**, effectuez les étapes suivantes : Ces étapes sont **facultatives** pour le tutoriel.

    1. Connectez-vous au modèle de machine virtuelle en sélectionnant **Se connecter**. S’il s’agit d’un modèle de machine virtuelle Linux, choisissez si vous voulez vous connecter à l’aide de SSH ou d’un bureau à distance GUI.  Une configuration supplémentaire est requise pour utiliser un bureau à distance GUI. Pour plus d’informations, consultez [Activer un bureau à distance graphique pour des machines virtuelles Linux](how-to-use-remote-desktop-linux-student.md).
    1. Sélectionnez **Réinitialiser le mot de passe** pour réinitialiser le mot de passe pour la machine virtuelle. 
    1. Installez et configurez des logiciels sur votre modèle de machine virtuelle. 
    1. **Arrêtez** la machine virtuelle.  
    1. Entrez une **description** pour le modèle
9.  Dans la page **Modèle**, sélectionnez **Publier** dans la barre d’outils. 

    ![Bouton Publier le modèle](./media/tutorial-setup-classroom-lab/template-page-publish-button.png)

    > [!WARNING]
    > Une fois que vous publiez, vous ne pouvez pas annuler la publication. 
10. Dans la page **Publier le modèle**, entrez le nombre de machines virtuelles que vous souhaitez créer dans le labo, puis sélectionnez **Publier**. 

    ![Publier le modèle - nombre de machines virtuelles](./media/tutorial-setup-classroom-lab/publish-template-number-vms.png)
11. Vous pouvez voir l’**état de publication** du modèle sur la page. Ce processus peut prendre jusqu’à une heure. 

    ![Publier un modèle - progression](./media/tutorial-setup-classroom-lab/publish-template-progress.png)
12. Passez à la page **Pool de machines virtuelles** en sélectionnant Machines virtuelles dans le menu de gauche ou en sélectionnant la vignette Machines virtuelles. Vérifiez que des machines virtuelles se trouvent à l’état **Non affectée**. Ces machines virtuelles ne sont pas encore affectées aux étudiants. Elles doivent être à l’état **Arrêtée**. Vous pouvez démarrer la machine virtuelle d’un étudiant, vous y connecter, l’arrêter et la supprimer dans cette page. Vous pouvez démarrer les machines virtuelles dans cette page ou laisser les étudiants le faire. 

    ![Machines virtuelles à l’état Arrêtée](./media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)

    Vous effectuerez les tâches suivantes dans cette page, mais pas dans le cadre de ce tutoriel. Elles sont fournies ici à titre indicatif uniquement : 
    
    1. Pour changer la capacité du labo (nombre de machines virtuelles dans le labo), sélectionnez **Capacité du labo** dans la barre d’outils.
    2. Pour démarrer toutes les machines virtuelles en même temps, sélectionnez **Démarrer tout** dans la barre d’outils. 
    3. Pour démarrer une machine virtuelle spécifique, sélectionnez la flèche vers le bas sous **État**, puis sélectionnez **Démarrer**. Vous pouvez également démarrer une machine virtuelle en la sélectionnant dans la première colonne, puis en sélectionnant **Démarrer** dans la barre d’outils.                

### <a name="vm-sizes"></a>Tailles de machine virtuelle  

| Taille | Cœurs | RAM | Description | 
| ---- | ----- | --- | ----------- | 
| Petite | 2 | 3,5 Go | Cette taille est idéale pour la ligne de commande, l’ouverture d’un navigateur web, les serveurs web à faible trafic et les bases de données de petite et moyenne taille. |
| Moyenne | 4 | 7 Go | Cette taille est idéale pour les bases de données relationnelles, le caching en mémoire et l’analyse | 
| Moyenne (virtualisation imbriquée) | 4 | 16 Go | Cette taille est idéale pour les bases de données relationnelles, le caching en mémoire et l’analyse. Cette taille prend également en charge la virtualisation imbriquée. <p>Cette taille peut être utilisée dans des scénarios où chaque étudiant a besoin de plusieurs machines virtuelles. Les formateurs peuvent utiliser la virtualisation imbriquée pour configurer quelques machines virtuelles imbriquées de petite taille dans la machine virtuelle. </p> |
| GPU de petite taille (calcul) | 6 | 56 Go | <p>Cette taille est optimisée pour les applications nécessitant beaucoup de ressources système et de ressources réseau, notamment l'intelligence artificielle et les applications d'apprentissage approfondi (Deep Learning).</p><p>Azure Lab Services installe et configure automatiquement les pilotes GPU nécessaires quand vous créez un laboratoire avec des images GPU. </p> | 
| GPU de petite taille (visualisation) | 6 | 56 Go | Cette taille est optimisée pour la visualisation à distance, la diffusion en continu, les jeux et l’encodage avec des infrastructures comme OpenGL ou DirectX. | 
| grand | 8 | 16 Go | Cette taille est idéale pour les applications nécessitant des UC plus rapides, de meilleures performances du disque local, des bases de données volumineuses et des caches mémoire volumineux. |
| Grande (virtualisation imbriquée) | 8 | 32 Go | Cette taille est idéale pour les applications nécessitant des UC plus rapides, de meilleures performances du disque local, des bases de données volumineuses et des caches mémoire volumineux. Cette taille prend également en charge la virtualisation imbriquée. |  
| GPU de taille moyenne (visualisation) | 12 | 112 Go | Cette taille est optimisée pour la visualisation à distance, la diffusion en continu, les jeux et l’encodage avec des infrastructures comme OpenGL ou DirectX. | 

> [!NOTE]
> Il se peut que certaines de ces tailles de machine virtuelle ne s’affichent pas dans la liste lors de la création d’un laboratoire de classe. La liste est remplie en fonction de la capacité actuelle de l’emplacement du laboratoire. Si le créateur du compte lab [permet aux créateurs de laboratoire de choisir un emplacement pour le laboratoire](allow-lab-creator-pick-lab-location.md), vous pouvez essayer de choisir un autre emplacement pour le laboratoire et voir si la taille de la machine virtuelle est disponible. 

## <a name="view-all-labs"></a>Afficher tous les laboratoires

1. Accédez au [portail Azure Lab Services](https://labs.azure.com).
1. Sélectionnez **Connexion**. Sélectionnez ou entrez un **ID utilisateur** qui est un membre du rôle **Créateur de laboratoire** dans le compte de laboratoire, puis entrez un mot de passe. Azure Lab Services prend en charge les comptes professionnels et les comptes Microsoft. 

    [!INCLUDE [Select a tenant](./includes/multi-tenant-support.md)]
1. Vérifiez que tous les laboratoires se trouvent dans le compte de laboratoire sélectionné. Sur la vignette du labo, vous voyez le nombre de machines virtuelles dans le labo et le quota pour chaque utilisateur (en dehors des heures planifiées).

    ![Tous les laboratoires](./media/how-to-manage-classroom-labs/all-labs.png)
1. Utilisez la liste déroulante du haut pour sélectionner un autre compte de laboratoire. Vous voyez les laboratoires du compte de laboratoire sélectionné. 

## <a name="delete-a-classroom-lab"></a>Supprimer un laboratoire de classe

1. Sur la vignette du laboratoire, sélectionnez les points de suspension (...) situés dans l’angle, puis sélectionnez **Supprimer**. 

    ![Bouton Supprimer](./media/how-to-manage-classroom-labs/delete-button.png)
1. Dans la boîte de dialogue **Supprimer le labo**, sélectionnez **Supprimer** pour poursuivre la suppression. 

## <a name="switch-to-another-classroom-lab"></a>Basculer vers un autre laboratoire de classe

Pour passer d'un laboratoire de classe à un autre, sélectionnez la liste déroulante des laboratoires du compte lab en haut de la page.

![Sélectionnez le laboratoire dans la liste déroulante du haut de la page](./media/how-to-manage-classroom-labs/switch-lab.png)

Vous pouvez également créer un labo à l’aide de **Nouveau labo** dans cette liste déroulante. 

> [!NOTE]
> Vous pouvez également utiliser le module PowerShell Az.LabServices (préversion) pour gérer les laboratoires. Pour plus d’informations, consultez la [page d’accueil Az.LabServices sur GitHub](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Modules/Library).

Pour basculer vers un autre compte Lab, sélectionnez la liste déroulante en regard du compte lab et sélectionnez l’autre compte lab. 

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [En tant que propriétaire de labo, configurer et publier des modèles](how-to-create-manage-template.md)
- [En tant que propriétaire de labo, configurer et contrôler l’utilisation d’un labo](how-to-configure-student-usage.md)
- [En tant qu’utilisateur de labo, accéder aux labos](how-to-use-classroom-lab.md)

