---
title: Restaurer les bases de données SAP HANA sur des machines virtuelles Azure
description: Dans cet article, découvrez comment restaurer des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure. Vous pouvez également utiliser la restauration inter-régions pour restaurer vos bases de données dans une région secondaire.
ms.topic: conceptual
ms.date: 11/7/2019
ms.openlocfilehash: c502b7741acd343baefe5e2bf8b95cfc02e46688
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021671"
---
# <a name="restore-sap-hana-databases-on-azure-vms"></a>Restaurer les bases de données SAP HANA sur des machines virtuelles Azure

Cet article décrit comment restaurer des bases de données SAP HANA exécutées sur une machine virtuelle Azure, qui a été sauvegardée dans un coffre Recovery Services avec le service Sauvegarde Azure. Les restaurations peuvent être utilisées pour créer des copies des données dans le cadre de scénarios de développement ou de test ou pour revenir à un état antérieur.

Pour plus d’informations sur la sauvegarde de bases de données SAP HANA, consultez [Sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure](./backup-azure-sap-hana-database.md).

## <a name="restore-to-a-point-in-time-or-to-a-recovery-point"></a>Restaurer à un point dans le temps ou à un point de récupération

Sauvegarde Azure peut restaurer des bases de données SAP HANA s’exécutant sur des machines virtuelles Azure comme suit :

* Restaurer à une date ou une heure spécifique (à la seconde), en utilisant les sauvegardes de fichiers journaux. Sauvegarde Azure détermine automatiquement les sauvegardes différentielles complètes appropriées, et la chaîne des sauvegardes de fichiers journaux nécessaires pour restaurer en fonction de la date/heure sélectionnée.

* Restaurer en fonction d’une sauvegarde complète ou différentielle spécifique à un point de récupération spécifique.

## <a name="prerequisites"></a>Prérequis

Avant de restaurer une base de données, notez les points suivants :

* Vous pouvez restaurer la base de données uniquement sur une instance SAP HANA qui se trouve dans la même région.

* L’instance cible doit être inscrite auprès du même coffre que la source.

* Sauvegarde Azure ne peut pas identifier deux instances de SAP HANA différentes sur la même machine virtuelle. Par conséquent, il est impossible de restaurer des données d’une instance vers une autre sur la même machine virtuelle.

* Pour que l’instance de SAP HANA cible soit prête pour la restauration, vérifiez son état **Préparation de la sauvegarde** :

  1. Ouvrez le coffre dans lequel l’instance SAP HANA cible est inscrite.

  1. Dans le tableau de bord du coffre, sous **Démarrage**, choisissez **Sauvegarde**.

      ![Sauvegarde dans le tableau de bord du coffre](media/sap-hana-db-restore/getting-started-backup.png)

  1. Dans **Sauvegarde**, sous **Que souhaitez-vous sauvegarder ?** , choisissez **SAP HANA dans la machine virtuelle Azure**.

      ![Choisir SAP HANA dans les machines virtuelles Azure](media/sap-hana-db-restore/sap-hana-backup.png)

  1. Sous **Découvrir les bases de données dans les machines virtuelles**, sélectionnez **Afficher les détails**.

      ![Afficher les détails](media/sap-hana-db-restore/view-details.png)

  1. Passez en revue la **Préparation de la sauvegarde** de la machine virtuelle cible.

      ![Serveurs protégés](media/sap-hana-db-restore/protected-servers.png)

* Pour en savoir plus sur les types de restauration pris en charge par SAP HANA, reportez-vous à la note SAP HANA [1642148](https://launchpad.support.sap.com/#/notes/1642148).

## <a name="restore-a-database"></a>Restaurer une base de données

Pour effectuer la restauration, vous avez besoin des autorisations suivantes :

* Les autorisations **Opérateur de sauvegarde** pour le coffre dans lequel vous effectuez la restauration.
* L’accès **Contributeur (écriture)** à la machine virtuelle source qui est sauvegardée.
* L’accès **Contributeur (écriture)** à la machine virtuelle cible :
  * Si vous effectuez une restauration sur la même machine virtuelle, ce sera la machine virtuelle source.
  * Si vous effectuez une restauration à un autre emplacement, ce sera la nouvelle machine virtuelle cible.

1. Ouvrez le coffre dans lequel la base de données SAP HANA à restaurer est inscrite.

1. Dans le tableau de bord du coffre, sous **Éléments protégés**, choisissez **Éléments de sauvegarde**.

    ![Éléments de sauvegarde](media/sap-hana-db-restore/backup-items.png)

1. Dans **Éléments de sauvegarde**, sous **Type de gestion de sauvegarde**, sélectionnez **SAP HANA dans les machines virtuelles Azure**.

    ![Type de gestion de sauvegarde](media/sap-hana-db-restore/backup-management-type.png)

1. Sélectionnez la base de données à restaurer.

    ![Base de données à restaurer](media/sap-hana-db-restore/database-to-restore.png)

1. Passez en revue le menu de la base de données. Il fournit des informations sur la sauvegarde de base de données, notamment :

    * Les points de restauration les plus anciens et les plus récents

    * L’état de la sauvegarde de fichier journal pour les dernières 24 et 72 heures pour la base de données

    ![Menu de la base de données](media/sap-hana-db-restore/database-menu.png)

1. Sélectionnez **Restaurer la base de données**.

1. Sous **Configuration de la restauration**, indiquez où (ou comment) restaurer les données :

    * **Autre emplacement** : restaure la base de données vers un autre emplacement et conserve la base de données source d’origine.

    * **Remplacer la base de données** : Restaurez les données sur la même instance de SAP HANA que la source d’origine. Cette option remplace la base de données d’origine.

      ![Configuration de la restauration](media/sap-hana-db-restore/restore-configuration.png)

### <a name="restore-to-alternate-location"></a>Restaurer à un autre emplacement

1. Dans le menu **Configuration de la restauration**, sous **Où voulez-vous restaurer**, sélectionnez **Autre emplacement**.

    ![Restaurer à un autre emplacement](media/sap-hana-db-restore/restore-alternate-location.png)

1. Sélectionnez le nom de l’hôte SAP HANA et le nom de l’instance dans lesquels vous souhaitez restaurer la base de données.
1. Vérifiez si l’instance de SAP HANA cible est prête pour la restauration en passant en revue sa **Préparation de la sauvegarde**. Pour plus d’informations, reportez-vous à la [section des prérequis](#prerequisites).
1. Dans la boîte de dialogue **Nom de la base de données restaurée**, indiquez le nom de la base de données cible.

    > [!NOTE]
    > Les restaurations SDC (Single Database Container) doivent suivre ces [contrôles](backup-azure-sap-hana-database-troubleshoot.md#single-container-database-sdc-restore).

1. Le cas échéant, sélectionnez **Remplacer si une base de données du même nom existe déjà sur l’instance HANA sélectionnée**.
1. Sélectionnez **OK**.

    ![Configuration de la restauration : écran final](media/sap-hana-db-restore/restore-configuration-last.png)

1. Dans **Sélectionner un point de restauration**, sélectionnez **Journaux (point dans le temps)** pour [effectuer une restauration vers un point spécifique dans le temps](#restore-to-a-specific-point-in-time). Ou sélectionnez **Complète et différentielle** pour [effectuer une restauration à un point de récupération spécifique](#restore-to-a-specific-recovery-point).

### <a name="restore-and-overwrite"></a>Restaurer et remplacer

1. Dans le menu **Configuration de la restauration**, sous **Où voulez-vous restaurer**, sélectionnez **Remplacer la base de données** > **OK**.

    ![Remplacer la base de données](media/sap-hana-db-restore/overwrite-db.png)

1. Dans **Sélectionner un point de restauration**, sélectionnez **Journaux (point dans le temps)** pour [effectuer une restauration vers un point spécifique dans le temps](#restore-to-a-specific-point-in-time). Ou sélectionnez **Complète et différentielle** pour [effectuer une restauration à un point de récupération spécifique](#restore-to-a-specific-recovery-point).

### <a name="restore-as-files"></a>Restaurer sous forme de fichiers

Pour restaurer les données de sauvegarde sous forme de fichiers plutôt que sous forme de base de données, choisissez **Restaurer sous forme de fichiers**. Une fois les fichiers vidés sous un chemin d’accès spécifié, vous pouvez placer ces fichiers sur n’importe quel ordinateur SAP HANA sur lequel vous souhaitez les restaurer sous forme de base de données. Étant donné que vous pouvez déplacer ces fichiers sur n’importe quel ordinateur, vous pouvez désormais restaurer les données entre les abonnements et les régions.

1. Dans le menu **Configuration de la restauration**, sous **Où et comment voulez-vous restaurer**, sélectionnez **Restaurer sous forme de fichiers**.
1. Sélectionnez l’**hôte** ou le nom du serveur HANA où vous voulez restaurer les fichiers de sauvegarde.
1. Dans le champ **Chemin d’accès de destination sur le serveur**, entrez le chemin d’accès du dossier sur le serveur sélectionné à l’étape 2. Il s’agit de l’emplacement où le service doit vider tous les fichiers de sauvegarde nécessaires.

    Les fichiers qui sont vidés sont les suivants :

    * Fichiers de sauvegarde de base de données
    * Fichiers catalogue
    * Fichiers de métadonnées JSON (pour chaque fichier de sauvegarde impliqué)

    En règle générale, un chemin d’accès de partage réseau, ou le chemin d’un partage de fichiers Azure monté lorsqu’il est spécifié comme chemin de destination, permet un accès plus facile à ces fichiers par d’autres ordinateurs du même réseau ou avec le même partage de fichiers Azure monté sur ces fichiers.

    >[!NOTE]
    >Pour restaurer les fichiers de sauvegarde de base de données sur un partage de fichiers Azure monté sur la machine virtuelle inscrite cible, assurez-vous que le compte racine dispose des autorisations de lecture ou d’écriture sur le partage de fichiers Azure.

    ![Choisir un chemin d’accès de destination](media/sap-hana-db-restore/restore-as-files.png)

1. Sélectionnez le **point de restauration** auquel tous les fichiers et dossiers de sauvegarde seront restaurés.

    ![Sélectionner le point de restauration](media/sap-hana-db-restore/select-restore-point.png)

1. Tous les fichiers de sauvegarde associés au point de restauration sélectionné sont vidés dans le chemin de destination.
1. En fonction du type de point de restauration choisi (**Limite dans le temps** ou **Différentielle et complète**), vous verrez un ou plusieurs dossiers créés dans le chemin de destination. L’un des dossiers nommés `Data_<date and time of restore>` contient les sauvegardes complètes et différentielles, et l’autre dossier nommé `Log` contient les sauvegardes de fichiers journaux.
1. Déplacez ces fichiers restaurés sur le serveur SAP HANA sur lequel vous souhaitez les restaurer en tant que base de données.
1. Exécutez ensuite les opérations qui suivent :
    1. Définissez des autorisations sur le dossier/répertoire dans lequel les fichiers de sauvegarde sont stockés à l’aide de la commande suivante :

        ```bash
        chown -R <SID>adm:sapsys <directory>
        ```

    1. Exécutez l’ensemble de commandes suivant en tant que `<SID>adm`.

        ```bash
        su - <sid>adm
        ```

    1. Générez le fichier catalogue pour la restauration. Extrayez le **BackupId** du fichier de métadonnées JSON pour la sauvegarde complète et qui sera utilisé ultérieurement dans l’opération de restauration. Assurez-vous que les sauvegardes complètes et celles de fichiers journaux se trouvent dans des dossiers différents et supprimez les fichiers catalogue et les fichiers de métadonnées JSON dans ces dossiers.

        ```bash
        hdbbackupdiag --generate --dataDir <DataFileDir> --logDirs <LogFilesDir> -d <PathToPlaceCatalogFile>
        ```

        Dans la commande ci-dessus :

        * `<DataFileDir>` : le dossier qui contient les sauvegardes complètes
        * `<LogFilesDir>` : le dossier qui contient les sauvegardes de fichiers journaux
        * `<PathToPlaceCatalogFile>` : le dossier dans lequel le fichier catalogue généré doit être placé

    1. Restaurez à l’aide du fichier catalogue qui vient d’être généré via HANA Studio ou exécutez la requête de restauration HDBSQL avec ce catalogue nouvellement généré. Les requêtes HDBSQL sont répertoriées ci-dessous :

    * Pour restaurer à une limite dans le temps :

        Si vous créez une base de données restaurée, exécutez la commande HDBSQL pour créer une base de données `<DatabaseName>`, puis arrêtez la base de données pour la restauration. Toutefois, si vous restaurez uniquement une base de données existante, exécutez la commande HDBSQL pour arrêter la base de données.

        Exécutez ensuite la commande suivante pour restaurer la base de données :

        ```hdbsql
        RECOVER DATABASE FOR <DatabaseName> UNTIL TIMESTAMP '<TimeStamp>' CLEAR LOG USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING LOG PATH (' <LogFileDir>') USING DATA PATH ('<DataFileDir>') USING BACKUP_ID <BackupIdFromJsonFile> CHECK ACCESS USING FILE
        ```

        * `<DatabaseName>` : nom de la nouvelle base de données ou de la base de données existante que vous souhaitez restaurer
        * `<Timestamp>` : horodatage exact de la limite de restauration dans le temps
        * `<DatabaseName@HostName>` : nom de la base de données dont la sauvegarde est utilisée pour la restauration et l’**hôte**/le nom du serveur SAP HANA sur lequel réside cette base de données. L’option `USING SOURCE <DatabaseName@HostName>` spécifie que la sauvegarde de données (utilisée pour la restauration) est celle d’une base de données avec un SID ou un nom différent de celui de la machine SAP HANA cible. Il n’est donc pas nécessaire de spécifier les restaurations effectuées sur le même serveur HANA à partir duquel la sauvegarde est effectuée.
        * `<PathToGeneratedCatalogInStep3>` : chemin d’accès au fichier catalogue généré à l’**étape C**
        * `<DataFileDir>` : le dossier qui contient les sauvegardes complètes
        * `<LogFilesDir>` : le dossier qui contient les sauvegardes de fichiers journaux
        * `<BackupIdFromJsonFile>` : **BackupId** extrait à l’**étape C**

    * Pour restaurer une sauvegarde complète ou différentielle particulière :

        Si vous créez une base de données restaurée, exécutez la commande HDBSQL pour créer une base de données `<DatabaseName>`, puis arrêtez la base de données pour la restauration. Toutefois, si vous restaurez uniquement une base de données existante, exécutez la commande HDBSQL pour arrêter la base de données :

        ```hdbsql
        RECOVER DATA FOR <DatabaseName> USING BACKUP_ID <BackupIdFromJsonFile> USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING DATA PATH ('<DataFileDir>')  CLEAR LOG
        ```

        * `<DatabaseName>` : nom de la nouvelle base de données ou de la base de données existante que vous souhaitez restaurer
        * `<Timestamp>` : horodatage exact de la limite de restauration dans le temps
        * `<DatabaseName@HostName>` : nom de la base de données dont la sauvegarde est utilisée pour la restauration et l’**hôte**/le nom du serveur SAP HANA sur lequel réside cette base de données. L’option `USING SOURCE <DatabaseName@HostName>` spécifie que la sauvegarde de données (utilisée pour la restauration) est celle d’une base de données avec un SID ou un nom différent de celui de la machine SAP HANA cible. Il n’est donc pas nécessaire de spécifier les restaurations effectuées sur le même serveur HANA à partir duquel la sauvegarde est effectuée.
        * `<PathToGeneratedCatalogInStep3>` : chemin d’accès au fichier catalogue généré à l’**étape C**
        * `<DataFileDir>` : le dossier qui contient les sauvegardes complètes
        * `<LogFilesDir>` : le dossier qui contient les sauvegardes de fichiers journaux
        * `<BackupIdFromJsonFile>` : **BackupId** extrait à l’**étape C**

### <a name="restore-to-a-specific-point-in-time"></a>Restaurer à un point spécifique dans le temps

Si vous avez sélectionné **Journaux d’activité (point dans le temps)** comme type de restauration, effectuez les actions suivantes :

1. Sélectionnez un point de récupération dans le graphique du journal et sélectionnez **OK** pour choisir le point de restauration.

    ![Point de restauration](media/sap-hana-db-restore/restore-point.png)

1. Dans le menu **Restaurer**, sélectionnez **Restaurer** pour démarrer le travail de restauration.

    ![Sélectionner Restaurer](media/sap-hana-db-restore/restore-restore.png)

1. Suivez la progression de la restauration dans la zone **Notifications** ou suivez-la en sélectionnant **Restaurer les travaux** dans le menu de la base de données.

    ![Restauration correctement déclenchée](media/sap-hana-db-restore/restore-triggered.png)

### <a name="restore-to-a-specific-recovery-point"></a>Restaurer à un point de récupération spécifique

Si vous avez sélectionné **Complète et différentielle** comme type de restauration, effectuez les actions suivantes :

1. Sélectionnez un point de récupération dans la liste et sélectionnez **OK** pour choisir le point de restauration.

    ![Restaurer à un point de récupération spécifique](media/sap-hana-db-restore/specific-recovery-point.png)

1. Dans le menu **Restaurer**, sélectionnez **Restaurer** pour démarrer le travail de restauration.

    ![Démarrer le travail de restauration](media/sap-hana-db-restore/restore-specific.png)

1. Suivez la progression de la restauration dans la zone **Notifications** ou suivez-la en sélectionnant **Restaurer les travaux** dans le menu de la base de données.

    ![Progression de la restauration](media/sap-hana-db-restore/restore-progress.png)

    > [!NOTE]
    > Dans les restaurations MDC, une fois que la base de référence système est restaurée vers une instance cible, vous devez réexécuter le script de pré-inscription. C’est uniquement à ce moment-là que les restaurations ultérieures de la base de données locataire réussiront. Pour en savoir plus, reportez-vous à [Résolution des problèmes : restauration MDC](backup-azure-sap-hana-database-troubleshoot.md#multiple-container-database-mdc-restore).

## <a name="cross-region-restore"></a>Restauration interrégion

parmi les options de restauration, la restauration entre régions (CRR) vous permet de restaurer des bases de données SAP HANA hébergées sur des machines virtuelles Azure dans une région secondaire, qui est une région jumelée Azure.

Pour intégrer la fonctionnalité à la préversion, lisez la [section Avant de commencer](./backup-create-rs-vault.md#set-cross-region-restore).

Pour voir si la CRR est activée, suivez les instructions indiquées dans [Configurer la restauration interrégion](backup-create-rs-vault.md#configure-cross-region-restore)

### <a name="view-backup-items-in-secondary-region"></a>Afficher les éléments de sauvegarde dans la région secondaire

Si la CRR est activée, vous pouvez afficher les éléments de sauvegarde dans la région secondaire.

1. À partir du portail, accédez à **Coffre Recovery Services** > **Éléments de sauvegarde**.
1. Cliquez sur **Région secondaire** pour afficher les éléments de la région secondaire.

>[!NOTE]
>Seuls les types de gestion des sauvegardes prenant en charge la fonctionnalité CRR s’affichent dans la liste. Actuellement, seule la prise en charge de la restauration de données de région secondaire vers une région secondaire est autorisée.

![Éléments de sauvegarde dans la région secondaire](./media/sap-hana-db-restore/backup-items-secondary-region.png)

![Bases de données dans la région secondaire](./media/sap-hana-db-restore/databases-secondary-region.png)

### <a name="restore-in-secondary-region"></a>Restaurer dans la région secondaire

L’expérience utilisateur de restauration de la région secondaire est similaire à celle de la région primaire. Quand vous configurez votre restauration dans le panneau Configuration de la restauration, vous êtes invité à fournir uniquement les paramètres de région secondaire.

![Où et comment restaurer](./media/sap-hana-db-restore/restore-secondary-region.png)

>[!NOTE]
>Le réseau virtuel de la région secondaire doit être attribué de manière unique et ne peut être utilisé pour aucune autre machine virtuelle de ce groupe de ressources.

![Déclencher la notification de restauration en cours](./media/backup-azure-arm-restore-vms/restorenotifications.png)

>[!NOTE]
>
>* Une fois la restauration déclenchée et au cours de la phase de transfert des données, le travail de restauration ne peut pas être annulé.
>* Les rôles Azure nécessaires à la restauration dans la région secondaire sont les mêmes que ceux de la région primaire.

### <a name="monitoring-secondary-region-restore-jobs"></a>Surveillance des travaux de restauration de la région secondaire

1. À partir du portail, accédez à **Coffre Recovery Services** > **Travaux de sauvegarde**.
1. Cliquez sur **Région secondaire** pour afficher les éléments de la région secondaire.

    ![Travaux de sauvegarde filtrés](./media/sap-hana-db-restore/backup-jobs-secondary-region.png)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment](sap-hana-db-manage.md) gérer des bases de données SAP HANA sauvegardées à l’aide de Sauvegarde Azure.
