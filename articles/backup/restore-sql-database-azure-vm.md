---
title: Restaurer des bases de données SQL Server sur une machine virtuelle Azure
description: Cet article explique comment restaurer des bases de données SQL Server exécutées sur une machine virtuelle Azure et sauvegardées avec Sauvegarde Azure. Vous pouvez également utiliser la restauration inter-régions pour restaurer vos bases de données dans une région secondaire.
ms.topic: conceptual
ms.date: 05/22/2019
ms.openlocfilehash: bbafd179f4b2f4e91a4bf19da41ffc14e4775e5c
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92172178"
---
# <a name="restore-sql-server-databases-on-azure-vms"></a>Restaurer des bases de données SQL Server sur des machines virtuelles Azure

Cet article décrit comment restaurer une base de données SQL Server exécutée sur une machine virtuelle Azure qui a été sauvegardée dans un coffre Azure Backup Recovery Services avec le service [Sauvegarde Azure](backup-overview.md).

Cet article explique comment restaurer des bases de données SQL Server. Pour plus d’informations, consultez [Sauvegarder des bases de données SQL Server sur des machines virtuelles Azure](backup-azure-sql-database.md).

## <a name="restore-to-a-time-or-a-recovery-point"></a>Restaurer à un point dans le temps ou un point de récupération

Sauvegarde Azure peut restaurer des bases de données SQL Server s’exécutant sur des machines virtuelles Azure comme suit :

- Restaurer à une date ou une heure spécifique (à la seconde), en utilisant les sauvegardes de fichiers journaux. Sauvegarde Azure détermine automatiquement la sauvegarde différentielle complète appropriée, et la chaîne des sauvegardes de fichiers journaux nécessaires pour restaurer en fonction de la date/heure sélectionnée.
- Restaurez une sauvegarde complète ou différentielle spécifique sur un point de récupération spécifique.

## <a name="prerequisites"></a>Prérequis

Avant de restaurer une base de données, notez les points suivants :

- Vous pouvez restaurer la base de données vers une instance SQL Server dans la même région Azure.
- Le serveur de destination doit être inscrit auprès du même coffre que la source.
- Pour restaurer une base de données chiffrée avec TDE sur un autre serveur SQL Server, vous devez d’abord [restaurer le certificat sur le serveur de destination](/sql/relational-databases/security/encryption/move-a-tde-protected-database-to-another-sql-server).
- Les bases de données compatibles avec la [capture des changements de données (CDC)](/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server) doivent être restaurées en utilisant l’option [Restaurer en tant que fichiers](#restore-as-files).
- Avant de restaurer la base de données « master », démarrez l’instance SQL Server en mode mono-utilisateur, avec l’option de démarrage **-m AzureWorkloadBackup**.
  - La valeur de **-m** est le nom du client.
  - Seul le nom du client spécifié peut ouvrir la connexion.
- Pour toutes les bases de données système (model, master, msdb), arrêtez le service SQL Server Agent avant de déclencher la restauration.
- Fermez toutes les applications qui tentent de voler une connexion à l’une de ces bases de données.
- Si plusieurs instances sont en cours d’exécution sur un serveur, toutes les instances doivent fonctionner. Dans le cas contraire, le serveur n’apparaîtra pas dans la liste des serveurs de destination pour vous permettre d’y restaurer la base de données.

## <a name="restore-a-database"></a>Restaurer une base de données

Pour effectuer la restauration, vous avez besoin des autorisations suivantes :

- Les autorisations **Opérateur de sauvegarde** pour le coffre dans lequel vous effectuez la restauration.
- L’accès **Contributeur (écriture)** à la machine virtuelle source qui est sauvegardée.
- L’accès **Contributeur (écriture)** à la machine virtuelle cible :
  - Si vous effectuez une restauration sur la même machine virtuelle, ce sera la machine virtuelle source.
  - Si vous effectuez une restauration à un autre emplacement, ce sera la nouvelle machine virtuelle cible.

Restaurez comme ceci :

1. Ouvrez le coffre dans lequel la machine virtuelle SQL Server est inscrite.
2. Dans le tableau de bord du coffre, sous **Utilisation**, sélectionnez **Éléments de sauvegarde**.
3. Dans **Éléments de sauvegarde**, sous **Type de gestion des sauvegardes**, sélectionnez **SQL dans une machine virtuelle Azure**.

    ![Sélectionner SQL in Azure VM](./media/backup-azure-sql-database/sql-restore-backup-items.png)

4. Sélectionnez la base de données à restaurer.

    ![Sélectionner la base de données à restaurer](./media/backup-azure-sql-database/sql-restore-sql-in-vm.png)

5. Passez en revue le menu de la base de données. Il fournit des informations sur la sauvegarde de la base de données, notamment :

    - Le point de restauration le plus ancien et le point de restauration le plus récent.
    - L’état de la sauvegarde de fichier journal pour les dernières 24 heures, concernant les bases de données en mode de récupération Complet et utilisant les journaux de transactions, si elles sont configurées pour les sauvegardes du journal des transactions.

6. Sélectionnez **Restaurer**.

    ![Sélectionnez Restaurer](./media/backup-azure-sql-database/restore-db.png)

7. Dans **Configuration de la restauration**, spécifiez où (et comment) vous voulez restaurer les données :
   - **Autre emplacement** : restaure la base de données vers un autre emplacement et conserve la base de données source d’origine.
   - **Remplacer la base de données** : restaure les données dans la même instance SQL Server que la source d’origine. Cette option remplace la base de données d’origine.

        > [!IMPORTANT]
        > Si la base de données sélectionnée fait partie d’un groupe de disponibilité Always On, SQL Server n’autorise pas le remplacement de la base de données. Seul **Autre emplacement** est disponible.
        >
   - **Restaurer sous forme de fichiers** : au lieu de restaurer sous forme de base de données, restaure les fichiers de sauvegarde qui peuvent être récupérés sous forme de base de données ultérieurement sur tout ordinateur sur lequel les fichiers sont présents à l’aide de SQL Server Management Studio.
     ![Menu Configuration de la restauration](./media/backup-azure-sql-database/restore-configuration.png)

### <a name="restore-to-an-alternate-location"></a>Restaurer à un autre emplacement

1. Dans le menu **Configuration de la restauration**, sous **Où voulez-vous restaurer**, sélectionnez **Autre emplacement**.
1. Sélectionnez le nom du serveur et l’instance SQL Server où vous voulez restaurer la base de données.
1. Dans la boîte de dialogue **Nom de la base de données restaurée**, indiquez le nom de la base de données cible.
1. Le cas échéant, sélectionnez **Remplacer si une base de données du même nom existe déjà sur l’instance SQL sélectionnée**.
1. Sélectionnez **Point de restauration**, puis sélectionnez [restaurer à un point spécifique dans le temps](#restore-to-a-specific-point-in-time) ou [restaurer à un point de récupération spécifique](#restore-to-a-specific-restore-point).

    ![Sélectionner un point de restauration](./media/backup-azure-sql-database/select-restore-point.png)

    ![Restaurer à un point dans le temps](./media/backup-azure-sql-database/restore-to-point-in-time.png)

1. Dans le menu **Configuration avancée** :

    - Si vous voulez que la base de données reste non opérationnelle après la restauration, activez **Restaurer avec NORECOVERY**.
    - Si vous voulez changer l’emplacement de restauration sur le serveur de destination, entrez de nouveaux chemins cibles.

        ![Entrer des chemins cibles](./media/backup-azure-sql-database/target-paths.png)

1. Sélectionnez **OK** pour déclencher la restauration. Suivez la progression de la restauration dans la zone **Notifications**, ou suivez-la sous la vue **Travaux de sauvegarde** dans le coffre.

    > [!NOTE]
    > La restauration à un point dans le temps est disponible seulement pour les sauvegardes de fichiers journaux pour les bases de données dont le mode de récupération est Complet et utilisant les journaux de transactions.

### <a name="restore-and-overwrite"></a>Restaurer et remplacer

1. Dans le menu **Configuration de la restauration**, sous **Où voulez-vous restaurer**, sélectionnez **Remplacer la base de données** > **OK**.

    ![Sélectionner Remplacer la base de données](./media/backup-azure-sql-database/restore-configuration-overwrite-db.png)

2. Dans **Sélectionner un point de restauration**, sélectionnez **Journaux (point dans le temps)** pour [effectuer une restauration vers un point spécifique dans le temps](#restore-to-a-specific-point-in-time). Ou sélectionnez **Complète et différentielle** pour effectuer une restauration vers un [point de récupération spécifique](#restore-to-a-specific-restore-point).

    > [!NOTE]
    > La restauration à un point dans le temps est disponible seulement pour les sauvegardes de fichiers journaux pour les bases de données dont le mode de récupération est Complet et utilisant les journaux de transactions.

### <a name="restore-as-files"></a>Restaurer sous forme de fichiers

Pour restaurer les données de sauvegarde sous forme de fichiers .bak plutôt que sous forme de base de données, choisissez **Restaurer sous forme de fichiers**. Une fois les fichiers vidés dans un chemin d’accès spécifié, vous pouvez placer ces fichiers sur n’importe quel ordinateur sur lequel vous souhaitez les restaurer sous forme de base de données. Étant donné que vous pouvez déplacer ces fichiers vers n’importe quel ordinateur, vous pouvez désormais restaurer les données entre les abonnements et les régions.

1. Sous **Où et comment restaurer**, sélectionnez **Restaurer sous forme de fichiers**.
1. Sélectionnez le nom du serveur SQL Server où vous voulez restaurer les fichiers de sauvegarde.
1. Dans le **Chemin d’accès de destination sur le serveur**, entrez le chemin d’accès du dossier sur le serveur sélectionné à l’étape 2. Il s’agit de l’emplacement où le service doit vider tous les fichiers de sauvegarde nécessaires. En règle générale, un chemin d’accès de partage réseau, ou le chemin d’un partage de fichiers Azure monté lorsqu’il est spécifié comme chemin de destination, permet un accès plus facile à ces fichiers par d’autres ordinateurs du même réseau ou avec le même partage de fichiers Azure monté sur ces fichiers.<BR>

    >Pour restaurer les fichiers de sauvegarde de base de données sur un partage de fichiers Azure monté sur la machine virtuelle inscrite cible, assurez-vous que NT AUTHORITY\SYSTEM a accès au partage de fichiers. Vous pouvez effectuer les étapes ci-dessous pour accorder les autorisations de lecture/écriture au compte AFS monté sur la machine virtuelle :
    >
    >- Exécuter `PsExec -s cmd` pour entrer dans l’interpréteur de commandes NT AUTHORITY\SYSTEM
    >   - Exécutez `cmdkey /add:<storageacct>.file.core.windows.net /user:AZURE\<storageacct> /pass:<storagekey>`
    >   - Vérifier l’accès avec `dir \\<storageacct>.file.core.windows.net\<filesharename>`
    >- Lancer une restauration en tant que fichiers à partir du coffre de sauvegarde vers `\\<storageacct>.file.core.windows.net\<filesharename>` comme chemin d’accès<BR>
    Vous pouvez télécharger PsExec à partir de la page [Sysinternals](/sysinternals/downloads/psexec).

1. Sélectionnez **OK**.

    ![Sélectionner Restaurer sous forme de fichiers](./media/backup-azure-sql-database/restore-as-files.png)

1. Sélectionnez **Point de restauration**, puis sélectionnez [restaurer à un point spécifique dans le temps](#restore-to-a-specific-point-in-time) ou [restaurer à un point de récupération spécifique](#restore-to-a-specific-restore-point).

1. Tous les fichiers de sauvegarde associés au point de récupération sélectionné sont vidés dans le chemin de destination. Vous pouvez restaurer les fichiers sous forme de base de données sur n’importe quel ordinateur sur lequel ils sont présents à l’aide de SQL Server Management Studio.

    ![Fichiers de sauvegarde restaurés dans le chemin de destination](./media/backup-azure-sql-database/sql-backup-files.png)

### <a name="restore-to-a-specific-point-in-time"></a>Restaurer à un point spécifique dans le temps

Si vous avez sélectionné **Journaux d’activité (point dans le temps)** comme type de restauration, effectuez les actions suivantes :

1. Sous **Date/heure de la restauration**, ouvrez le calendrier. Dans le calendrier, les dates marquant des points de récupération sont indiquées en gras, et la date du jour est mise en surbrillance.
1. Sélectionnez une date avec des points de récupération. Vous ne pouvez pas sélectionner des dates qui n’ont aucun point de récupération.

    ![Ouvrir le calendrier](./media/backup-azure-sql-database/recovery-point-logs-calendar.png)

1. Une fois que vous avez sélectionné une date, le graphique chronologique affiche les points de récupération disponibles dans une plage continue.
1. Spécifiez une heure pour la récupération avec le graphique chronologique, ou sélectionnez une heure. Sélectionnez ensuite **OK**.

### <a name="restore-to-a-specific-restore-point"></a>Restaurer à un point de restauration spécifique

Si vous avez sélectionné **Complète et différentielle** comme type de restauration, effectuez les actions suivantes :

1. Sélectionnez un point de récupération dans la liste, puis cliquez sur **OK** pour terminer la procédure de sélection du point de restauration.

    ![Choisir un point de récupération complète](./media/backup-azure-sql-database/choose-full-recovery-point.png)

    >[!NOTE]
    > Par défaut, les points de récupération des 30 derniers jours sont affichés. Vous pouvez afficher les points de récupération datant de plus de 30 jours en sélectionnant **Filtrer**, puis une plage personnalisée.

### <a name="restore-databases-with-large-number-of-files"></a>Restaurer des bases de données comprenant un grand nombre de fichiers

Si la taille totale de la chaîne des fichiers dans une base de données dépasse une [limite spécifique](backup-sql-server-azure-troubleshoot.md#size-limit-for-files), Sauvegarde Azure stocke la liste des fichiers de base de données dans un composant pit différent, de sorte que vous ne pouvez pas définir le chemin de restauration cible pendant l’opération de restauration. Au lieu de cela, les fichiers seront restaurés vers le chemin d’accès SQL par défaut.

  ![Restaurer une base de données avec des fichiers volumineux](./media/backup-azure-sql-database/restore-large-files.jpg)

## <a name="cross-region-restore"></a>Restauration interrégion

parmi les options de restauration, la restauration entre régions (CRR) vous permet de restaurer des bases de données SQL hébergées sur des machines virtuelles Azure dans une région secondaire, qui est une région jumelée Azure.

Pour intégrer la fonctionnalité à la préversion, lisez la [section Avant de commencer](./backup-create-rs-vault.md#set-cross-region-restore).

Pour voir si la CRR est activée, suivez les instructions indiquées dans [Configurer la restauration interrégion](backup-create-rs-vault.md#configure-cross-region-restore)

### <a name="view-backup-items-in-secondary-region"></a>Afficher les éléments de sauvegarde dans la région secondaire

Si la CRR est activée, vous pouvez afficher les éléments de sauvegarde dans la région secondaire.

1. À partir du portail, accédez à **Coffre Recovery Services** > **Éléments de sauvegarde**.
1. Cliquez sur **Région secondaire** pour afficher les éléments de la région secondaire.

>[!NOTE]
>Seuls les types de gestion des sauvegardes prenant en charge la fonctionnalité CRR s’affichent dans la liste. Actuellement, seule la prise en charge de la restauration de données de région secondaire vers une région secondaire est autorisée.

![Éléments de sauvegarde dans la région secondaire](./media/backup-azure-sql-database/backup-items-secondary-region.png)

![Bases de données dans la région secondaire](./media/backup-azure-sql-database/databases-secondary-region.png)

### <a name="restore-in-secondary-region"></a>Restaurer dans la région secondaire

L’expérience utilisateur de restauration de la région secondaire est similaire à celle de la région primaire. Quand vous configurez votre restauration dans le panneau Configuration de la restauration, vous êtes invité à fournir uniquement les paramètres de région secondaire.

![Où et comment restaurer](./media/backup-azure-sql-database/restore-secondary-region.png)

>[!NOTE]
>Le réseau virtuel de la région secondaire doit être attribué de manière unique et ne peut être utilisé pour aucune autre machine virtuelle de ce groupe de ressources.

![Déclencher la notification de restauration en cours](./media/backup-azure-arm-restore-vms/restorenotifications.png)

>[!NOTE]
>
>- Une fois la restauration déclenchée et au cours de la phase de transfert des données, le travail de restauration ne peut pas être annulé.
>- Les rôles Azure nécessaires à la restauration dans la région secondaire sont les mêmes que ceux de la région primaire.

### <a name="monitoring-secondary-region-restore-jobs"></a>Surveillance des travaux de restauration de la région secondaire

1. À partir du portail, accédez à **Coffre Recovery Services** > **Travaux de sauvegarde**.
1. Cliquez sur **Région secondaire** pour afficher les éléments de la région secondaire.

    ![Travaux de sauvegarde filtrés](./media/backup-azure-sql-database/backup-jobs-secondary-region.png)

## <a name="next-steps"></a>Étapes suivantes

[Gérez et surveillez](manage-monitor-sql-database-backup.md) des bases de données SQL Server sauvegardées par Sauvegarde Azure.