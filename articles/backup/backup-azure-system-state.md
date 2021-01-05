---
title: Sauvegarder l’état du système Windows vers Azure
description: Découvrez comment sauvegarder l’état du système des ordinateurs Windows Server vers Azure.
ms.topic: conceptual
ms.date: 05/23/2018
ms.openlocfilehash: 1b3573d757d2f7b1ffec9ae718aa791488960f3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91332885"
---
# <a name="back-up-windows-system-state-to-azure"></a>Sauvegarder l’état du système Windows vers Azure

Cet article explique comment sauvegarder l’état du système Windows Server vers Azure. Il est destiné à vous présenter les notions de base.

Si vous souhaitez en savoir plus sur Sauvegarde Azure, lisez cette [présentation](backup-overview.md).

Si vous ne disposez pas d’un abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) pour accéder à n’importe quel service Azure.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="set-storage-redundancy-for-the-vault"></a>Définir la redondance du stockage pour un coffre

Lorsque vous créez un coffre Recovery Services, vérifiez que la redondance du stockage est configurée comme vous le souhaitez.

1. Dans le volet **Coffres Recovery Services**, sélectionnez le nouveau coffre.

    ![Sélectionnez le nouveau coffre dans la liste des coffres Recovery Services.](./media/backup-try-azure-backup-in-10-mins/rs-vault-list.png)

    Lorsque vous sélectionnez le coffre, le volet **Coffre Recovery Services** se réduit et le volet Paramètres (*qui porte le nom du coffre en haut*) ainsi que le volet des détails du coffre s’ouvrent.

    ![Afficher la configuration de stockage du nouveau coffre](./media/backup-try-azure-backup-in-10-mins/set-storage-configuration-2.png)
2. Dans le volet Paramètres du nouveau coffre, utilisez le curseur vertical pour faire défiler l’écran jusqu’à la section Gestion, puis sélectionnez **Infrastructure de sauvegarde**.
    Le volet Infrastructure de sauvegarde s'ouvre.
3. Dans le volet Infrastructure de sauvegarde, sélectionnez **Configuration de la sauvegarde** pour ouvrir le volet **Configuration de la sauvegarde**.

    ![Définir la configuration de stockage du nouveau coffre](./media/backup-try-azure-backup-in-10-mins/set-storage-configuration.png)
4. Choisissez l’option de réplication de stockage à appliquer à votre archivage.

    ![Options de configuration du stockage](./media/backup-try-azure-backup-in-10-mins/choose-storage-configuration-for-vault.png)

    Par défaut, votre archivage utilise un stockage géo-redondant. Si vous utilisez Azure comme principal point de terminaison du stockage de sauvegarde, laissez cette option **inchangée**. Sinon, choisissez l’option **Localement redondant**, qui réduit les coûts de stockage Azure. Pour en savoir plus sur les options de stockage [géo-redondant](../storage/common/storage-redundancy.md#geo-redundant-storage), [localement redondant](../storage/common/storage-redundancy.md#locally-redundant-storage) et [redondant interzone](../storage/common/storage-redundancy.md#zone-redundant-storage), consultez la [présentation de la redondance du stockage](../storage/common/storage-redundancy.md).

Une fois votre coffre créé, vous devez le configurer pour la sauvegarde de l’état du système Windows.

## <a name="configure-the-vault"></a>configuration du coffre

1. Dans le volet du coffre Recovery Services que vous venez de créer, accédez à la section Prise en main, puis sélectionnez **Sauvegarde** ; dans le volet **Prise en main de la sauvegarde**, sélectionnez **Objectif de la sauvegarde**.

    ![Ouvrir les paramètres de sauvegarde](./media/backup-try-azure-backup-in-10-mins/open-backup-settings.png)

    Le volet **Objectif de la sauvegarde** s’ouvre.

    ![Ouvrir le volet Objectif de la sauvegarde](./media/backup-try-azure-backup-in-10-mins/backup-goal-blade.png)

2. Dans la liste déroulante **Où s’exécute votre charge de travail ?** , sélectionnez **Local**.

    En effet, vous devez choisir l’option **Local**, car votre ordinateur Windows Server ou Windows est une machine physique, qui ne se trouve donc pas dans Azure.

3. Dans le menu **Que voulez-vous sauvegarder ?** , sélectionnez **État du système**, puis **OK**.

    ![Configuration des fichiers et dossiers](./media/backup-azure-system-state/backup-goal-system-state.png)

    Lorsque vous sélectionnez **OK**, une coche apparaît à côté de la zone **Objectif de la sauvegarde** et le volet **Préparer l’infrastructure** s’ouvre.

    ![Objectif de sauvegarde configuré, début de préparation de l’infrastructure](./media/backup-try-azure-backup-in-10-mins/backup-goal-configed.png)

4. Dans le volet **Préparer l’infrastructure**, sélectionnez **Télécharger l’agent pour Windows Server ou pour le client Windows**.

    ![Préparer l’infrastructure](./media/backup-try-azure-backup-in-10-mins/choose-agent-for-server-client.png)

    Si vous utilisez Windows Server Essentials, puis choisissez de télécharger l’agent associé à ce produit, un menu contextuel vous invite à exécuter ou enregistrer le fichier MARSAgentInstaller.exe.

    ![Boîte de dialogue MARSAgentInstaller](./media/backup-try-azure-backup-in-10-mins/mars-installer-run-save.png)

5. Dans le menu contextuel de téléchargement, sélectionnez **Enregistrer**.

    Par défaut, le fichier **MARSagentinstaller.exe** est enregistré dans le dossier Téléchargements. Une fois l’exécution du programme d’installation terminée, une fenêtre contextuelle s’affiche, vous demandant si vous voulez lancer ce programme ou ouvrir le dossier.

    ![L’installation de MARS est terminée](./media/backup-try-azure-backup-in-10-mins/mars-installer-complete.png)

    Vous n’avez pas besoin d’installer l’agent à cette étape ; Vous pouvez installer l’agent après avoir téléchargé les informations d’identification du coffre.

6. Dans le volet **Préparer l’infrastructure**, sélectionnez **Télécharger**.

    ![Télécharger les informations d’identification du coffre](./media/backup-try-azure-backup-in-10-mins/download-vault-credentials.png)

    Les informations d’identification du coffre sont téléchargées dans le dossier **Téléchargements**. Une fois cette opération terminée, une fenêtre contextuelle s’affiche, vous demandant si vous voulez ouvrir ou enregistrer ces informations. Sélectionnez **Enregistrer**. Si vous sélectionnez **Ouvrir** par erreur, attendez que la boîte de dialogue qui s’affiche annonce l’échec de la tentative d’ouverture des informations d’identification du coffre. Vous ne pouvez pas ouvrir les informations d’identification du coffre. Passez à l’étape suivante. Les informations d’identification du coffre se trouvent dans le dossier **Téléchargements**.

    ![Fin du téléchargement des informations d’identification du coffre](./media/backup-try-azure-backup-in-10-mins/vault-credentials-downloaded.png)
   > [!NOTE]
   > Les informations d’identification du coffre doivent être enregistrées uniquement à un emplacement proche du serveur Windows sur lequel vous souhaitez utiliser l’agent.
   >

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="install-and-register-the-agent"></a>Installer et inscrire l’agent

> [!NOTE]
> La sauvegarde via le Portail Azure n’est pas disponible pour le moment. Utilisez l’agent Microsoft Azure Recovery Services pour sauvegarder l’état du système Windows Server.
>

1. Recherchez et double-cliquez sur **MARSagentinstaller.exe** dans le dossier Téléchargements (ou tout autre emplacement d’enregistrement).

    Le programme d’installation affiche différents messages au fur et à mesure qu’il extrait, installe et enregistre l’agent Recovery Services.

    ![Informations d’identification de programme d’installation de l’agent Recovery Services exécuté](./media/backup-try-azure-backup-in-10-mins/mars-installer-registration.png)

2. Exécutez l’Assistant Installation de l’Agent Microsoft Azure Recovery Services. Pour terminer l’Assistant, vous devez :

   * Choisir un emplacement pour le dossier d’installation et de cache.
   * Fournir les informations relatives au serveur proxy, si vous en utilisez un pour vous connecter à Internet.
   * Fournir votre nom d’utilisateur et votre mot de passe si vous utilisez un proxy authentifié.
   * Fournir les informations d’identification de l’archivage téléchargées.
   * Enregistrer la phrase secrète de chiffrement dans un emplacement sécurisé.

     > [!NOTE]
     > En cas de perte ou d’oubli de la phrase secrète, Microsoft ne sera pas en mesure de vous aider à récupérer les données de sauvegarde. Enregistrez le fichier dans un emplacement sécurisé. Il est obligatoire pour restaurer une sauvegarde.
     >
     >

L’agent est désormais installé et votre ordinateur est inscrit dans le coffre. Vous êtes prêt à configurer et à planifier votre sauvegarde.

## <a name="back-up-windows-server-system-state"></a>Sauvegarder l’état du système Windows Server

La sauvegarde initiale comprend deux tâches :

* Planifier la sauvegarde
* Sauvegarder l’état du système pour la première fois

Pour effectuer la sauvegarde initiale, utilisez l’agent Microsoft Azure Recovery Services.

> [!NOTE]
> Vous pouvez sauvegarder l’état du système sur Windows Server, de la version 2008 R2 à la version 2016. La sauvegarde de l’état du système n’est pas prise en charge sur les références SKU du client. L’état du système n’apparaît pas comme option pour les clients Windows ni pour les machines Windows Server 2008 SP2.
>
>

### <a name="to-schedule-the-backup-job"></a>Pour planifier un travail de sauvegarde

1. Ouvrez l’agent Microsoft Azure Recovery Services. Vous pouvez le trouver en recherchant **Sauvegarde Microsoft Azure** sur votre ordinateur.

    ![Lancer l’agent Azure Recovery Services](./media/backup-try-azure-backup-in-10-mins/snap-in-search.png)

2. Dans l’agent Recovery Services, sélectionnez **Planifier la sauvegarde**.

    ![Planifier une sauvegarde de Windows Server](./media/backup-try-azure-backup-in-10-mins/schedule-first-backup.png)

3. Sur la page **Mise en route** de l’Assistant Planifier la sauvegarde, sélectionnez **Suivant**.

4. Sur la page **Sélectionner les éléments à sauvegarder**, sélectionnez **Ajouter des éléments**.

5. Sélectionnez **État du système**, puis **OK**.

6. Sélectionnez **Suivant**.

7. Sélectionnez la fréquence de sauvegarde requise et la stratégie de rétention pour les sauvegardes de l’état du système dans les pages suivantes.

8. Dans la page Confirmation, passez en revue les informations, puis sélectionnez **Terminer**.

9. Une fois que l’Assistant a créé la planification de sauvegarde, sélectionnez **Fermer**.

### <a name="to-back-up-windows-server-system-state-for-the-first-time"></a>Pour sauvegarder l’état du système Windows Server pour la première fois

1. Vérifiez qu’aucune mise à jour de Windows Server nécessitant un redémarrage n’est en attente.

2. Dans l’agent Recovery Services, sélectionnez **Sauvegarder maintenant** pour effectuer l’amorçage initial sur le réseau.

    ![Option Sauvegarder maintenant de Windows Server](./media/backup-try-azure-backup-in-10-mins/backup-now.png)

3. Sélectionnez **État du système** dans l’écran **Sélectionner l’élément de sauvegarde** qui s’affiche, puis sélectionnez **Suivant**.

4. Sur la page Confirmation, vérifiez les paramètres utilisés par l’Assistant Sauvegarder maintenant pour sauvegarder les données de l’ordinateur, Sélectionnez ensuite **Sauvegarder**.

5. Sélectionnez **Fermer** pour fermer l’Assistant. Si vous fermez l’Assistant avant la fin du processus de sauvegarde, celui-ci continuera de s’exécuter en arrière-plan.
    > [!NOTE]
    > L’agent MARS déclenche `SFC /verifyonly` dans le cadre des vérifications préalables à chaque sauvegarde de l’état du système. Cela permet de garantir que les fichiers sauvegardés dans le cadre de l’état du système disposent des versions appropriées correspondant à la version de Windows. Découvrez-en plus sur le vérificateur des fichiers système (SFC, System File Checker), consultez [cet article](/windows-server/administration/windows-commands/sfc).
    >

Une fois la sauvegarde initiale terminée, le statut **Tâche terminée** apparaît dans la console Backup.

  ![RI terminé](./media/backup-try-azure-backup-in-10-mins/ircomplete.png)

## <a name="questions"></a>Des questions ?

Si vous avez des questions, [envoyez-nous des commentaires](https://feedback.azure.com/forums/258995-azure-backup).

## <a name="next-steps"></a>Étapes suivantes

* Approfondissez vos connaissances sur la [sauvegarde de machines Windows](backup-windows-with-mars-agent.md).
* L’état du système Windows Server étant sauvegardé, vous pouvez [gérer vos archivages et vos serveurs](backup-azure-manage-windows-server.md).
* Si vous avez besoin de restaurer une sauvegarde, utilisez cet article pour [restaurer des fichiers sur un ordinateur Windows](backup-azure-restore-windows-server.md).
