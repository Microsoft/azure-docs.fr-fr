---
title: 'Tutoriel : Sauvegarder plusieurs machines virtuelles Azure'
description: Dans ce didacticiel explique comment créer un coffre Recovery Services, définir une stratégie de sauvegarde et à sauvegarder simultanément plusieurs machines virtuelles.
ms.date: 07/26/2020
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: a9517ffc1e37d50f7c0e57b9ed53fb8bcf55fd70
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89180573"
---
# <a name="use-azure-portal-to-back-up-multiple-virtual-machines"></a>Utiliser le portail Azure pour sauvegarder plusieurs machines virtuelles

Lorsque vous sauvegardez des données dans Azure, vous stockez celles-ci dans une ressource Azure que l’on appelle coffre Recovery Services. La ressource coffre Recovery Services est disponible dans le menu Paramètres de la plupart des services Azure. L’avantage de disposer du coffre Recovery Services intégré dans le menu Paramètres de la plupart des services Azure est que cela simplifie la sauvegarde de données. Toutefois, il est fastidieux de manipuler individuellement chaque base de données ou machine virtuelle disponible dans votre entreprise. Que se passe-t-il si vous souhaitez sauvegarder les données de toutes les machines virtuelles d’un service ou d’un emplacement ? Il est facile de sauvegarder plusieurs machines virtuelles en créant une stratégie de sauvegarde et en l’appliquant aux machines virtuelles de votre choix. Ce didacticiel explique comment :

> [!div class="checklist"]
>
> * Créer un coffre Recovery Services
> * Créer une stratégie de sauvegarde
> * Appliquer la stratégie de sauvegarde pour protéger plusieurs machines virtuelles
> * Déclencher un travail de sauvegarde à la demande pour les machines virtuelles protégées

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Le coffre Recovery Services contient les données de sauvegarde et la stratégie de sauvegarde appliquée aux machines virtuelles protégées. La sauvegarde des machines virtuelles est un processus local. Vous ne pouvez pas sauvegarder une machine virtuelle se trouvant dans un emplacement vers un coffre Recovery Services situé dans un autre emplacement. Donc, chaque emplacement Azure contenant des machines virtuelles à sauvegarder, doit comprendre au moins un coffre Recovery Services.

1. Dans le menu de gauche, sélectionnez **Tous les services**.

    ![Sélectionner Tous les services](./media/tutorial-backup-vm-at-scale/click-all-services.png)

1. Dans la boîte de dialogue **Tous les services**, entrez *Recovery Services*. Liste des filtres de ressources variant en fonction de votre entrée. Dans la liste des ressources, sélectionnez **Coffres Recovery Services**.

    ![Entrée et choisir Coffres Recovery Services](./media/tutorial-backup-vm-at-scale/all-services.png)

    La liste des coffres Recovery Services de l’abonnement s’affiche.

1. Dans le tableau de bord **Coffres Recovery Services**, cliquez sur **Ajouter**.

    ![Ajouter un coffre Recovery Services](./media/tutorial-backup-vm-at-scale/add-button-create-vault.png)

1. Dans le menu du coffre Recovery Services,

    * tapez *myRecoveryServicesVault* dans **Nom**.
    * L’ID d’abonnement actuel apparaît dans **Abonnement**. Si vous avez des abonnements supplémentaires, vous pouvez en choisir un autre pour le nouveau coffre.
    * Pour **Groupe de ressources**, sélectionnez **Utiliser existant** et choisissez *myResourceGroup*. Si *myResourceGroup* n’existe pas, sélectionnez **Créer un nouveau**, puis tapez *myResourceGroup*.
    * Dans le menu déroulant **Emplacement**, choisissez *Europe Ouest*.

    ![Valeurs du coffre Recovery Services](./media/tutorial-backup-vm-at-scale/review-and-create.png)

    Un coffre Recovery Services doit être situé dans le même emplacement que les machines virtuelles protégées. Si vous possédez des machines virtuelles dans plusieurs régions, créez un coffre Recovery Services dans chacune d’entre elles. Ce tutoriel crée un coffre Recovery Services *Europe Ouest* car c’est là que la machine virtuelle *myVM* a été créée avec le démarrage rapide.

1. Quand vous êtes prêt à créer le coffre Recovery Services, sélectionnez **Créer**.

    ![Créer le coffre Recovery Services](./media/tutorial-backup-vm-at-scale/click-create-button.png)

1. La création du coffre Recovery Services peut prendre un certain temps. Surveillez les notifications d’état dans la zone **Notifications** dans l’angle supérieur droit du portail. Une fois que le coffre est créé, il apparaît dans la liste des coffres Recovery Services. Si vous ne voyez pas votre coffre, sélectionnez **Actualiser**.

     ![Actualiser la liste des coffres de sauvegarde](./media/tutorial-backup-vm-at-scale/refresh-button.png)

Lorsque vous créez un coffre Recovery Services, par défaut, celui-ci a un stockage géo-redondant. Pour assurer la résilience des données, un stockage géo-redondant réplique celles-ci plusieurs fois dans deux régions Azure.

## <a name="set-backup-policy-to-protect-vms"></a>Définir une stratégie de sauvegarde pour protéger les machines virtuelles

Une fois le coffre Recovery Services créé, l’étape suivante consiste à configurer celui-ci pour le type de données, et à définir la stratégie de sauvegarde. La stratégie de sauvegarde planifie la fréquence et l’heure de la création des points de récupération. Elle inclut également la durée de rétention de ces derniers. Pour ce tutoriel, supposons que votre entreprise est un complexe sportif avec un hôtel, un stade, des restaurants et des concessions et que vous protégez les données figurant sur les machines virtuelles. Les étapes suivantes créent une stratégie de sauvegarde pour les données financières.

1. Dans la liste des coffres Recovery Services, sélectionnez le coffre **myRecoveryServicesVault** pour ouvrir son tableau de bord.

   ![Ouvrir le menu Scénario](./media/tutorial-backup-vm-at-scale/open-vault-from-list.png)

1. Dans le menu du tableau de bord du coffre, sélectionnez **Sauvegarder** pour ouvrir le menu Sauvegarder.

1. Dans le menu Objectif de sauvegarde, dans le menu déroulant **Où s’exécute votre charge de travail ?** , choisissez *Azure*. Dans le menu déroulant **Que voulez-vous sauvegarder ?** , choisissez *Machine virtuelle*, puis sélectionnez **Sauvegarder**.

    Ces actions préparent le coffre Recovery Services à interagir avec une machine virtuelle. Les coffres Recovery Services ont une stratégie par défaut qui crée un point de restauration quotidiennement, et conserve les points de restauration pendant 30 jours.

    ![Objectif de sauvegarde](./media/tutorial-backup-vm-at-scale/backup-goal.png)

1. Pour créer une stratégie, dans le menu Stratégie de sauvegarde, dans le menu déroulant **Choisir une stratégie de sauvegarde**, sélectionnez *Créer une nouvelle stratégie*.

    ![Créez une stratégie.](./media/tutorial-backup-vm-at-scale/create-new-policy.png)

1. Le volet **Stratégie de sauvegarde** s’ouvre. Renseignez les valeurs suivantes :
   * Pour **Nom de la stratégie** tapez *Finance*. Entrez les modifications suivantes pour la stratégie de sauvegarde :
   * Pour **Fréquence de sauvegarde**, définissez le fuseau horaire *Central*. Le complexe sportif étant situé au Texas, son propriétaire souhaite que le calcul du temps soit local. Laissez la fréquence de sauvegarde définie sur Chaque jour à 3:30.
   * Pour **Rétention du point de sauvegarde quotidien**, définissez une période de 90 jours.
   * Pour **Rétention du point de sauvegarde hebdomadaire**, choisissez le point de restauration *lundi* et la rétention de 52  semaines.
   * Pour **Rétention du point de sauvegarde mensuel**, choisissez le point de restauration Premier dimanche de chaque mois et la rétention de 36 mois.
   * Désactivez l’option **Rétention du point de sauvegarde annuel**. Le responsable financier ne souhaite pas conserver les données plus de 36 mois.
   * Sélectionnez **OK** pour créer la stratégie de sauvegarde.

     ![Paramètres de stratégie de sauvegarde](./media/tutorial-backup-vm-at-scale/set-new-policy.png)

     Après avoir créé la stratégie de sauvegarde, associez-la aux machines virtuelles.

1. Sous **Machines virtuelles**, sélectionnez **Ajouter**.

     ![Ajouter des machines virtuelles](./media/tutorial-backup-vm-at-scale/add-virtual-machines.png)

1. Le volet **Sélectionner les machines virtuelles** s’ouvre. Sélectionnez *myVM*, puis sélectionnez **OK** pour déployer la stratégie de sauvegarde sur les machines virtuelles.

    Tous les machines virtuelles situées au même emplacement qui ne sont pas encore associées à une stratégie de sauvegarde s’affichent. Les machines virtuelles *myVMH1* et *myVMR1* sont sélectionnées pour être associées la stratégie *Finance*.

    ![Choisir les machines virtuelles à protéger](./media/tutorial-backup-vm-at-scale/choose-vm-to-protect.png)

1. Après avoir choisi les machines virtuelles, sélectionnez **Activer la sauvegarde**.

    Une fois le déploiement terminé, vous recevez une notification pour vous en informer.

## <a name="initial-backup"></a>Sauvegarde initiale

Vous avez activé la sauvegarde pour les coffres Recovery Services, mais aucune sauvegarde initiale n’a été créée. Une bonne pratique pour la reprise d’activité consiste à déclencher la première sauvegarde de façon à ce que les données soient protégées.

Pour exécuter un travail de sauvegarde à la demande :

1. Sur le tableau de bord du coffre, sous **Éléments de sauvegarde**, sélectionnez **3** pour ouvrir le menu Éléments de sauvegarde.

    ![Éléments de sauvegarde](./media/tutorial-backup-vm-at-scale/tutorial-vm-back-up-now.png)

    Le menu **Éléments de sauvegarde** s’ouvre.

1. Dans le menu **Éléments de sauvegarde**, sélectionnez **Machine virtuelle Azure** pour ouvrir la liste des machines virtuelles associées au coffre.

    ![Liste des machines virtuelles](./media/tutorial-backup-vm-at-scale/three-virtual-machines.png)

1. La liste **Éléments de sauvegarde** s’affiche.

    ![Travail de sauvegarde déclenché](./media/tutorial-backup-vm-at-scale/initial-backup-context-menu.png)

1. Dans la liste **Éléments de sauvegarde**, sélectionnez le bouton de sélection **...** pour ouvrir le menu contextuel.

1. Dans le menu contextuel, sélectionnez **Sauvegarder maintenant**.

    ![Menu contextuel – Sélectionner Sauvegarder maintenant](./media/tutorial-backup-vm-at-scale/context-menu.png)

    Le menu Sauvegarder maintenant s’ouvre.

1. Dans le menu Sauvegarder maintenant, entrez le dernier jour de conservation du point de récupération, puis sélectionnez **OK**.

    ![Définition du dernier jour de conservation du point de récupération dans le panneau Sauvegarder maintenant](./media/tutorial-backup-vm-at-scale/backup-now-short.png)

    Les notifications de déploiement vous informent que la sauvegarde a été déclenchée et que vous pouvez surveiller la progression du travail sur la page Travaux de sauvegarde. Selon la taille de votre machine virtuelle, la création de la sauvegarde initiale peut prendre un certain temps.

    Une fois le travail de sauvegarde initiale terminé, vous pouvez voir son état dans le menu Travail de sauvegarde. Le travail de sauvegarde à la demande a créé le point de restauration initial pour *myVM*. Si vous souhaitez sauvegarder d’autres machines virtuelles, répétez ces étapes pour chacune d’elles.

    ![Vignette Travaux de sauvegarde](./media/tutorial-backup-vm-at-scale/initial-backup-complete.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous prévoyez de continuer avec les tutoriels suivants, ne nettoyez pas les ressources créées dans ce tutoriel. Sinon, effectuez les étapes suivantes pour supprimer toutes les ressources créées dans le cadre de ce tutoriel dans le portail Azure.

1. Sur le tableau de bord **myRecoveryServicesVault**, sous **Éléments de sauvegarde**, sélectionnez **3** pour ouvrir le menu Éléments de sauvegarde.

    ![Ouvrir le menu Éléments de sauvegarde](./media/tutorial-backup-vm-at-scale/tutorial-vm-back-up-now.png)

1. Dans le menu **Éléments de sauvegarde**, sélectionnez **Machine virtuelle Azure** pour ouvrir la liste des machines virtuelles associées au coffre.

    ![Liste des machines virtuelles](./media/tutorial-backup-vm-at-scale/three-virtual-machines.png)

    La liste **Éléments de sauvegarde** s’affiche.

1. Dans le menu **Éléments de sauvegarde**, sélectionnez le bouton de sélection (...) pour ouvrir le menu contextuel.

    ![Dans le menu Éléments de sauvegarde, ouvrez le menu contextuel.](./media/tutorial-backup-vm-at-scale/context-menu-to-delete-vm.png)

1. Dans le menu contextuel, sélectionnez **Arrêter la sauvegarde** pour ouvrir le menu Arrêter la sauvegarde.

    ![Menu Arrêter la sauvegarde](./media/tutorial-backup-vm-at-scale/context-menu-for-delete.png)

1. Dans le menu **Arrêter la sauvegarde**, sélectionnez le menu déroulant supérieur, puis choisissez **Supprimer les données de sauvegarde**.

1. Dans la boîte de dialogue **Tapez le nom de l’élément de sauvegarde** , tapez *myVM*.

1. Une fois l’élément de sauvegarde vérifié (une coche s’affiche), le bouton **Arrêter la sauvegarde** est activé. Sélectionnez **Arrêter la sauvegarde** pour arrêter la stratégie et supprimer les points de restauration.

    ![Sélectionner Arrêter la sauvegarde pour supprimer le coffre](./media/tutorial-backup-vm-at-scale/provide-reason-for-delete.png)

    >[!NOTE]
    >Les éléments supprimés sont conservés avec l’état de suppression réversible pendant 14 jours. Le coffre peut être supprimé après cette période uniquement. Pour plus d’informations, consultez [Supprimer un coffre Recovery Services Sauvegarde Azure](backup-azure-delete-vault.md).

1. Quand le coffre est vide, sélectionnez **Supprimer**.

    ![Sélectionner Supprimer](./media/tutorial-backup-vm-at-scale/deleting-the-vault.png)

    Après la suppression du coffre, vous revenez à la liste des coffres Recovery Services.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé le portail Azure pour effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Créer un coffre Recovery Services
> * Définir le coffre pour protéger les machines virtuelles
> * Créer une stratégie de sauvegarde et de rétention personnalisée
> * Attribuer la stratégie pour protéger plusieurs machines virtuelles
> * Déclencher une sauvegarde à la demande pour des machines virtuelles

Passez au didacticiel suivant pour restaurer une machine virtuelle Azure à partir d’un disque.

> [!div class="nextstepaction"]
> [Restaurer des machines virtuelles à l’aide d’Azure](./tutorial-restore-disk.md)
