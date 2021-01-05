---
title: Restaurer l’état du système sur un serveur Windows Server
description: Explication étape par étape pour la restauration de l’état du système Windows Server à partir d’une sauvegarde dans Azure.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 4ef23d6ff16c263e310304cc240c2090751640b1
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97008467"
---
# <a name="restore-system-state-to-windows-server"></a>Restaurer l’état du système sur Windows Server

Cet article explique comment restaurer des sauvegardes de l’état du système Windows Server à partir d’un coffre Azure Recovery Services. Pour restaurer l’état du système, vous devez disposer d’une sauvegarde de l’état du système (créée en suivant les instructions fournies dans [Sauvegarder l’état du système](backup-azure-system-state.md#back-up-windows-server-system-state) et vérifier que vous avez installé la [version la plus récente de l’agent Microsoft Azure Recovery Services (MARS)](https://aka.ms/azurebackup_agent). La récupération des données d’état du système Windows Server à partir d’un coffre Azure Recovery Services est un processus en deux étapes :

1. Restaurer l’état du système sous forme de fichiers à partir de Sauvegarde Azure. Lors de la restauration de l’état du système sous forme de fichiers à partir de Sauvegarde Azure, vous pouvez :
   * Restaurer l’état du système sur le même serveur que celui où les sauvegardes ont été effectuées, ou
   * Restaurer le fichier d’état du système sur un autre serveur

2. Appliquez les fichiers d’état du système restaurés à un serveur Windows Server à l’aide de l’utilitaire Sauvegarde Windows Server.

## <a name="recover-system-state-files-to-the-same-server"></a>Récupérer les fichiers d’état du système sur le même serveur

Les étapes suivantes expliquent comment restaurer votre configuration Windows Server à un état antérieur. La restauration de votre configuration de serveur à un état stable connu peut être extrêmement utile. Les étapes suivantes restaurent l’état du système du serveur à partir d’un coffre Recovery Services.

1. Ouvrez le composant logiciel enfichable **Sauvegarde Microsoft Azure**. Si vous ne savez pas où il a été installé, recherchez **Sauvegarde Microsoft Azure** sur l’ordinateur ou le serveur.

    L’application de bureau devrait apparaître dans les résultats de recherche.

2. Sélectionnez **Récupérer des données** pour lancer l’Assistant.

    ![Récupérer des données](./media/backup-azure-restore-windows-server/recover.png)

3. Sur le volet **Prise en main**, pour restaurer les données sur le même serveur ou ordinateur, sélectionnez **Ce serveur (`<server name>`)** , puis **Suivant**.

    ![Choisissez l’option Ce serveur pour restaurer les données sur le même ordinateur.](./media/backup-azure-restore-system-state/samemachine.png)

4. Dans le volet **Sélectionner le mode de récupération**, choisissez **État du système**, puis sélectionnez **Suivant**.

    ![Parcourir les fichiers](./media/backup-azure-restore-system-state/recover-type-selection.png)

5. Sur le calendrier dans le volet **Sélectionner un volume et une date**, sélectionnez un point de récupération.

    Vous pouvez effectuer la restauration à partir du point de récupération de votre choix dans le temps. Les dates **en gras** indiquent la disponibilité d’au moins un point de récupération. Une fois que vous avez sélectionné une date, si plusieurs points de récupération sont disponibles, cliquez sur l’un d’entre eux dans le menu déroulant **Temps**.

    ![Volume et date](./media/backup-azure-restore-system-state/select-date.png)

6. Une fois que vous avez choisi le point de récupération à restaurer, sélectionnez **Suivant**.

    La Sauvegarde Azure monte le point de récupération local et l’utilise comme volume de récupération.

7. Dans le volet suivant, spécifiez la destination des fichiers d’état du système récupérés. Ensuite, sélectionnez **Parcourir** pour ouvrir l’Explorateur Windows et rechercher les fichiers et dossiers que vous souhaitez. L’option **Créer des copies pour avoir les deux versions** permet de créer des copies des différents fichiers dans une archive de fichiers d’état du système existante, au lieu de créer la copie de la totalité de l’archive d’état du système.

    ![Options de récupération](./media/backup-azure-restore-system-state/recover-as-files.png)

8. Vérifiez les détails de la récupération dans le volet **Confirmation** et sélectionnez **Récupérer**.

   ![Sélectionner Récupérer pour accuser réception de l’action de récupération](./media/backup-azure-restore-system-state/confirm-recovery.png)

9. Copiez le répertoire *WindowsImageBackup* dans la destination de récupération sur un volume non critique du serveur. En règle générale, le volume de système d’exploitation Windows est le volume critique.

10. Une fois la récupération réussie, suivez les étapes décrites dans la section [Appliquer l’état du système restauré sur un serveur Windows](#apply-restored-system-state-on-a-windows-server) pour terminer le processus de récupération de l’état du système.

## <a name="recover-system-state-files-to-an-alternate-server"></a>Récupérer les fichiers d’état du système sur un autre serveur

Si votre serveur Windows est endommagé ou inaccessible et que vous voulez le restaurer à un état stable en récupérant l’état du système Windows Server, vous pouvez restaurer l’état du système du serveur endommagé à partir d’un autre serveur. Pour restaurer l’état du système sur un autre serveur, effectuez les étapes suivantes.  

Les termes ci-après sont utilisés pour cette procédure :

* *Ordinateur source* : ordinateur d’origine à partir duquel la sauvegarde a été effectuée et qui est actuellement indisponible.
* *Ordinateur cible* : ordinateur sur lequel les données sont récupérées.
* *Exemple d’archivage* – coffre Recovery Services dans lequel *l’ordinateur source* et *l’ordinateur cible* sont enregistrés.

> [!NOTE]
> Les sauvegardes effectuées à partir d’un ordinateur ne peuvent pas être restaurées sur un ordinateur qui exécute une version antérieure du système d’exploitation. Par exemple, les sauvegardes effectuées à partir d’un ordinateur Windows Server 2016 ne peut pas être restaurées vers Windows Server 2012 R2. Toutefois, l’inverse est possible. Vous pouvez utiliser des sauvegardes de Windows Server 2012 R2 pour restaurer Windows Server 2016.
>

1. Ouvrez le composant logiciel enfichable **Sauvegarde Microsoft Azure** sur l’*Ordinateur cible*.
2. Vérifiez que *l’ordinateur cible* et *l’ordinateur source* sont inscrits auprès du même coffre Recovery Services.
3. Sélectionnez **Récupérer des données** pour lancer le workflow.
4. Sélectionnez **Un autre serveur**

    ![Un autre serveur](./media/backup-azure-restore-system-state/anotherserver.png)

5. Fournissez le fichier d’informations d’identification de coffre qui correspond à l’ *exemple d’archivage*. Si le fichier d’informations d’identification du coffre n’est pas valide (ou a expiré), téléchargez un nouveau fichier d’informations d’identification de coffre à partir de *l’Exemple de coffre* dans le Portail Azure. Une fois le fichier d’informations d’identification de coffre fourni, le coffre Recovery Services associé au fichier d’informations d’identification de coffre apparaît.

6. Dans le volet Sélectionner un serveur de sauvegarde, sélectionnez l’*Ordinateur source* dans la liste des ordinateurs affichés.
7. Dans le volet Sélectionner le mode de récupération, choisissez **État du système** et sélectionnez **Suivant**.

    ![Recherche](./media/backup-azure-restore-system-state/recover-type-selection.png)

8. Sur le calendrier dans le volet **Sélectionner un volume et une date**, sélectionnez un point de récupération. Vous pouvez effectuer la restauration à partir du point de récupération de votre choix dans le temps. Les dates **en gras** indiquent la disponibilité d’au moins un point de récupération. Une fois que vous avez sélectionné une date, si plusieurs points de récupération sont disponibles, cliquez sur l’un d’entre eux dans le menu déroulant **Temps**.

    ![Éléments de recherche](./media/backup-azure-restore-system-state/select-date.png)

9. Une fois que vous avez choisi le point de récupération à restaurer, sélectionnez **Suivant**.

10. Dans le volet **Sélectionner le mode de récupération de l’état du système**, spécifiez la destination où vous souhaitez récupérer les fichiers d’état du système, puis sélectionnez **Suivant**.

    ![Chiffrement](./media/backup-azure-restore-system-state/recover-as-files.png)

    L’option **Créer des copies pour avoir les deux versions** permet de créer des copies des différents fichiers dans une archive de fichiers d’état du système existante, au lieu de créer la copie de la totalité de l’archive d’état du système.

11. Vérifiez les détails de la récupération dans le volet Confirmation et sélectionnez **Récupérer**.

    ![Sélectionner le bouton Récupérer pour confirmer le processus de récupération](./media/backup-azure-restore-system-state/confirm-recovery.png)

12. Copiez le répertoire *WindowsImageBackup* sur un volume non critique du serveur (par exemple D:\). En règle générale, le volume de système d’exploitation Windows est le volume critique.

13. Pour terminer le processus de récupération, utilisez la section suivante pour [appliquer les fichiers d’état du système restaurés sur un serveur Windows](#apply-restored-system-state-on-a-windows-server).

## <a name="apply-restored-system-state-on-a-windows-server"></a>Appliquer l’état du système restauré sur un serveur Windows

Une fois que vous avez récupéré l’état du système sous forme de fichiers à l’aide de l’Agent Azure Recovery Services, utilisez l’utilitaire Sauvegarde Windows Server pour appliquer l’état du système récupéré à Windows Server. L’utilitaire Sauvegarde Windows Server est déjà disponible sur le serveur. Les étapes suivantes expliquent comment appliquer l’état du système récupéré.

1. Ouvrez le composant logiciel enfichable Sauvegarde de Windows Server. Si vous ne savez pas où il a été installé, recherchez **Sauvegarde Windows Server** sur l’ordinateur ou le serveur.

    L’application de bureau s’affiche dans les résultats de recherche. Si ce n’est pas le cas, ou si vous rencontrez des erreurs à l’ouverture de l’application, vous devez installer les **Fonctionnalités de la Sauvegarde de Windows Server** ainsi que les composants dépendants situés en dessous, disponibles dans l’**Assistant Ajout de fonctionnalités** du **Gestionnaire de serveur**.

1. Dans le composant logiciel enfichable, sélectionnez **Sauvegarde locale**.

    ![sélectionner Sauvegarde locale pour restaurer à partir de là](./media/backup-azure-restore-system-state/win-server-backup-local-backup.png)

1. Dans la console Sauvegarde locale, dans le volet **Actions**, sélectionnez **Récupérer** pour ouvrir l’Assistant Récupération.

1. Sélectionnez l’option **Un autre emplacement**, puis sélectionnez **Suivant**.

   ![choisir de récupérer sur un autre serveur](./media/backup-azure-restore-system-state/backup-stored-in-diff-location.png)

1. Quand vous spécifiez le type d’emplacement, sélectionnez **Dossier partagé distant** si votre sauvegarde de l’état du système a été récupérée sur un autre serveur. Si l’état du système a été récupéré localement, sélectionnez **Lecteurs locaux**.

    ![sélectionner s’il faut récupérer à partir d’un serveur local ou autre](./media/backup-azure-restore-system-state/ss-recovery-remote-shared-folder.png)

1. Entrez le chemin du répertoire *WindowsImageBackup* ou choisissez le lecteur local contenant ce répertoire (par exemple, D:\WindowsImageBackup), récupéré dans le cadre de la récupération des fichiers d’état du système à l’aide de l’Agent Azure Recovery Services, puis sélectionnez **Suivant**.

    ![chemin du fichier partagé](./media/backup-azure-restore-system-state/ss-recovery-remote-folder.png)

1. Sélectionnez la version de l’état du système que vous souhaitez restaurer, puis sélectionnez **Suivant**.

1. Dans le volet Sélectionner le type de récupération, sélectionnez **État du système**, puis **Suivant**.

1. Comme emplacement de récupération de l’état du système, sélectionnez **Emplacement d’origine**, puis **Suivant**.

    Si vous restaurez un contrôleur de domaine, l’option supplémentaire suivante s’affiche :

    ![Emplacement pour la récupération de l’état du système](./media/backup-azure-restore-system-state/location-for-system-state-recovery.png)

    >[!NOTE]
    >Sélectionnez uniquement « Effectuer une restauration faisant autorité des fichiers Active Directory » si vous avez l’intention explicite de faire une restauration faisant autorité de toutes les données Active Directory.

1. Passez en revue les détails de confirmation, vérifiez les paramètres de redémarrage, puis sélectionnez **Récupérer** pour appliquer les fichiers d’état du système restaurés.

    ![lancez la restauration des fichiers d’état du système](./media/backup-azure-restore-system-state/launch-ss-recovery.png)

    >[!NOTE]
    >Ne sélectionnez pas l’option **Redémarrer automatiquement le serveur** si vous effectuez la restauration en mode DSRM.

1. Une fois la restauration terminée, vous devez redémarrer le serveur en mode normal. Ouvrez une invite de commandes et tapez ce qui suit : `bcdedit /deletevalue safeboot`
1. Redémarrez le serveur.

## <a name="special-considerations-for-system-state-recovery-on-a-domain-controller"></a>Considérations spéciales pour la récupération de l’état du système sur un contrôleur de domaine

La sauvegarde de l’état du système inclut les données Active Directory. Effectuez les étapes suivantes pour restaurer les services de domaine Active Directory (AD DS) de leur état actuel vers un état antérieur. Ce type de restauration peut être effectué dans deux scénarios :

* Restauration de toutes les données Active Directory lorsqu’il ne reste aucun contrôleur de domaine opérationnel dans la forêt
* Restauration d’une partie des données Active Directory lorsque ces objets ont été supprimés ou endommagés

Cet article aborde uniquement le premier scénario, qui appelle une restauration ne faisant pas autorité d’AD DS et une restauration faisant autorité du dossier sysvol.  Si vous devez effectuer le deuxième scénario (où les contrôleurs de domaine sont toujours opérationnels, mais que vous devez restaurer des objets Active Directory particuliers), consultez [ces instructions](https://support.microsoft.com/help/840001/how-to-restore-deleted-user-accounts-and-their-group-memberships-in-ac).

1. Suivez ces étapes pour [récupérer les fichiers d’état du système sur un autre serveur](#recover-system-state-files-to-an-alternate-server).
1. Exécutez les commandes suivantes pour redémarrer votre serveur en *Mode de réparation des services d’annuaire*. À partir d’une invite de commandes avec élévation de privilèges :

    ```cmd
    Bcdedit /set safeboot dsrepair
    Shutdown /r /t 0
    ```

1. Pour récupérer Active Directory dans le cadre d’une restauration de l’état du système, vous pouvez choisir l’une des deux méthodes suivantes :

    * Suivez les instructions ci-dessus pour [appliquer l’état du système restauré sur un serveur Windows Server](#apply-restored-system-state-on-a-windows-server) avec l’utilitaire Sauvegarde Windows Server.

        >[!NOTE]
        >Si vous restaurez toutes les données Active Directory (et qu’il ne reste aucun contrôleur de domaine opérationnel dans la forêt), à l’étape 9 ci-dessus, veillez à sélectionner **Effectuer une restauration faisant autorité des fichiers Active Directory**.

    * Utilisez l’utilitaire [wbadmin](/windows-server/administration/windows-commands/wbadmin-start-systemstaterecovery) pour effectuer la restauration à partir de la ligne de commande.

        Vous aurez besoin de l’identificateur de version de la sauvegarde que vous souhaitez utiliser. Vous pouvez obtenir la liste des identificateurs de version en exécutant la commande suivante :

        ```cmd
        wbadmin get versions -backuptarget <servername\sharename>
        ```

        Vous utilisez ensuite cet identificateur de version pour exécuter la restauration.

        Par exemple, pour effectuer une [restauration ne faisant pas autorité d’AD DS et une restauration faisant autorité du dossier sysvol](/windows-server/identity/ad-ds/manage/ad-forest-recovery-nonauthoritative-restore) à l’aide de la sauvegarde du 30/04/2020 à 9:00 AM, qui est stockée sur la ressource partagée `\\servername\share` pour `server01`, tapez :

        ```cmd
        wbadmin start systemstaterecovery -version:04/30/2020-09:00 -backupTarget:\\servername\share -machine:server01 -authsysvol
        ```

1. Une fois la restauration terminée, vous devez redémarrer le serveur en mode normal. Ouvrez une invite de commandes et tapez ce qui suit : `bcdedit /deletevalue safeboot`
1. Redémarrez le serveur.

Pour plus d’informations, consultez [Sauvegarder et restaurer des contrôleurs de domaine Active Directory](active-directory-backup-restore.md).

## <a name="troubleshoot-failed-system-state-restore"></a>Résoudre les problèmes d’échec de restauration de l’état du système

Si le processus précédent d’application de l’état du système ne se termine pas correctement, utilisez l’Environnement de récupération Windows (WinRE) pour récupérer votre serveur Windows Server. Les étapes suivantes expliquent comment récupérer à l’aide de l’Environnement de récupération Windows (WinRE). Utilisez cette option uniquement si Windows Server ne démarre pas normalement après une restauration de l’état du système. Le processus suivant efface les données autres que les données système. Soyez prudent.

1. Démarrez votre ordinateur Windows Server dans l’Environnement de récupération Windows (WinRE).

2. Sélectionnez Dépanner parmi les trois options disponibles.

    ![Sélectionner Dépanner](./media/backup-azure-restore-system-state/winre-1.png)

3. Dans l’écran **Options avancées**, sélectionnez **Invite de commandes** et fournissez le nom d’utilisateur et le mot de passe de l’administrateur de serveur.

   ![Sélectionner Invite de commandes](./media/backup-azure-restore-system-state/winre-2.png)

4. Fournissez le nom d’utilisateur et le mot de passe de l’administrateur de serveur.

    ![Boîte de dialogue Saisie du mot de passe](./media/backup-azure-restore-system-state/winre-3.png)

5. Quand vous ouvrez l’invite de commandes en mode administrateur, exécutez la commande suivante pour obtenir les versions de sauvegarde de l’état du système.

    ```cmd
    Wbadmin get versions -backuptarget:<Volume where WindowsImageBackup folder is copied>:
    ```

    ![Obtenir les versions de sauvegarde de l’état du système](./media/backup-azure-restore-system-state/winre-4.png)

6. Exécutez la commande suivante pour obtenir tous les volumes disponibles dans la sauvegarde.

    ```cmd
    Wbadmin get items -version:<copy version from above step> -backuptarget:<Backup volume>
    ```

    ![Obtenir tous les volumes disponibles](./media/backup-azure-restore-system-state/winre-5.png)

7. La commande suivante récupère tous les volumes qui font partie de la sauvegarde de l’état du système. Notez que cette étape récupère uniquement les volumes critiques qui font partie de l’état du système. Toutes les données autres que les données système sont effacées.

    ```cmd
    Wbadmin start recovery -items:C: -itemtype:Volume -version:<Backupversion> -backuptarget:<backup target volume>
    ```

     ![Récupérer tous les volumes](./media/backup-azure-restore-system-state/winre-6.png)

## <a name="next-steps"></a>Étapes suivantes

* Maintenant que vous avez restauré vos fichiers et vos dossiers, vous pouvez [gérer vos sauvegardes](backup-azure-manage-windows-server.md).
