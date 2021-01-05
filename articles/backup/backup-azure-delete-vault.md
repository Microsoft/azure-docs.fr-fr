---
title: Supprimer un coffre Recovery Services Microsoft Azure
description: Dans cet article, découvrez comment supprimer les dépendances, puis supprimer un coffre Azure Backup Recovery Services.
ms.topic: conceptual
ms.date: 06/04/2020
ms.openlocfilehash: fd941db933d243b83c1c19c7ae0fdfc2d7869b8a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91293065"
---
# <a name="delete-an-azure-backup-recovery-services-vault"></a>Supprimer un coffre Azure Backup Recovery Services

Cet article explique comment supprimer un coffre Recovery Services [Sauvegarde Azure](backup-overview.md). Il indique comment supprimer des dépendances, puis un coffre.

## <a name="before-you-start"></a>Avant de commencer

Vous ne pouvez pas supprimer un coffre Recovery Services doté de l’une des dépendances suivantes :

- Vous ne pouvez pas supprimer un coffre qui contient des sources de données protégées (par exemple, des machines virtuelles IaaS, des bases de données SQL, des partages de fichiers Azure).
- Vous ne pouvez pas supprimer un coffre qui contient des données de sauvegarde. Une fois les données de sauvegarde supprimées, elles passent à l’état de suppression réversible.
- Vous ne pouvez pas supprimer un coffre qui contient des données de sauvegarde à l’état de suppression réversible.
- Vous ne pouvez pas supprimer un coffre qui contient des comptes de stockage inscrits.

Si vous essayez de supprimer le coffre sans enlever les dépendances, vous obtenez l’un des messages d’erreur suivants :

- Impossible de supprimer le coffre car il contient des ressources. Veuillez vérifier qu’aucun élément de sauvegarde, serveur protégé ou serveur de gestion des sauvegardes n’est associé à ce coffre. Annulez l’inscription des conteneurs suivants associés à ce coffre avant de procéder à la suppression.

- Le coffre Recovery Services ne peut pas être supprimé, car il contient des éléments de sauvegarde dans l’état de suppression réversible. Les éléments supprimés de manière réversible sont définitivement supprimés au bout de 14 jours de l’opération de suppression. Essayez de supprimer le coffre une fois que les éléments de sauvegarde ont été supprimés définitivement et qu’il ne contient plus d’élément dans l’état de suppression réversible. Pour plus d’informations, consultez [Suppression réversible pour la Sauvegarde Azure](./backup-azure-security-feature-cloud.md).

## <a name="proper-way-to-delete-a-vault"></a>Méthode appropriée pour supprimer un coffre

>[!WARNING]
>L’opération suivante est destructrice et ne peut pas être annulée. Toutes les données de sauvegarde et tous les éléments de sauvegarde associés au serveur protégé seront définitivement supprimés. Procédez avec prudence.

Pour supprimer correctement un coffre, vous devez suivre les étapes ci-après dans l’ordre indiqué :

- **Étape 1** : Désactivez la fonctionnalité de suppression réversible. Pour connaître les étapes de désactivation de la suppression réversible, [cliquez ici](./backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete).

- **Étape 2** : Une fois la suppression réversible désactivée, vérifiez s’il existe des éléments à l’état de suppression réversible. Si c’est le cas, vous devez *annuler leur suppression* et les *supprimer* de nouveau. [Suivez ces étapes](./backup-azure-security-feature-cloud.md#permanently-deleting-soft-deleted-backup-items) pour rechercher les éléments à l’état de suppression réversible et les supprimer définitivement.

- **Étape 3** : Vous devez vérifier s’il existe des éléments protégés aux trois emplacements ci-dessous :

  - **Éléments protégés dans le cloud** : Accédez au menu du tableau de bord du coffre, puis sélectionnez **Éléments de sauvegarde**. Tous les éléments listés à cet endroit doivent être supprimés avec **Arrêter la sauvegarde** ou **Supprimer les données de sauvegarde** ainsi que leurs données de sauvegarde.  [Suivez ces étapes](#delete-protected-items-in-the-cloud) pour supprimer ces éléments.
  - **Instance SQL Server** : Accédez au menu du tableau de bord du coffre, puis sélectionnez **Infrastructure de sauvegarde** > **Serveurs protégés**. Dans Serveurs protégés, sélectionnez le serveur dont vous souhaitez annuler l'inscription. Si vous souhaitez supprimer le coffre, vous devez annuler l’inscription de tous les serveurs. Cliquez sur le serveur protégé, puis sélectionnez **Désinscrire**.
  - **Serveurs protégés par MARS** : Accédez au menu du tableau de bord du coffre, puis sélectionnez **Infrastructure de sauvegarde** > **Serveurs protégés**. Si vous avez des serveurs protégés par MARS, tous les éléments listés à cet endroit doivent être supprimés ainsi que leurs données de sauvegarde. [Suivez ces étapes](#delete-protected-items-on-premises) pour supprimer les serveurs protégés par MARS.
  - **Serveurs d’administration MABS ou DPM** : Accédez au menu du tableau de bord du coffre, puis sélectionnez **Infrastructure de sauvegarde** > **Serveurs de gestion des sauvegardes**. Si vous avez un serveur DPM ou de sauvegarde Azure (MABS), tous les éléments listés à cet endroit doivent être supprimés ou désinscrits ainsi que leurs données de sauvegarde. [Suivez ces étapes](#delete-protected-items-on-premises) pour supprimer les serveurs d’administration.

- **Étape 4** : Vous devez vous assurer que tous les comptes de stockage inscrits sont supprimés. Accédez au menu du tableau de bord du coffre, puis sélectionnez **Infrastructure de sauvegarde** > **Comptes de stockage**. Si vous disposez de comptes de stockage listés à cet endroit, vous devez annuler leur inscription. Pour savoir comme désinscrire un compte, consultez [Annuler l’inscription d’un compte de stockage](manage-afs-backup.md#unregister-a-storage-account).

Une fois ces étapes terminées, vous pouvez poursuivre la [suppression du coffre](#delete-the-recovery-services-vault).

Si vous n’avez pas d’éléments protégés localement ou dans le cloud, mais que vous obtenez toujours l’erreur de suppression du coffre, effectuez les étapes décrites dans [Supprimer le coffre Recovery Services à l’aide d’Azure Resource Manager](#delete-the-recovery-services-vault-by-using-azure-resource-manager).

## <a name="delete-protected-items-in-the-cloud"></a>Supprimer des éléments protégés dans le cloud

Commencez par lire la section **[Avant de commencer](#before-you-start)** pour comprendre le processus de suppression des dépendances et du coffre.

Pour arrêter la protection et supprimer les données de sauvegarde, effectuez les étapes suivantes :

1. À partir du portail, accédez à **Coffre Recovery Services**, puis à **Éléments de sauvegarde**. Ensuite, choisissez les éléments protégés dans le cloud (par exemple, machines virtuelles Azure, Stockage Azure [service Azure Files] ou SQL Server sur les machines virtuelles Azure).

    ![Sélectionnez le type de sauvegarde.](./media/backup-azure-delete-vault/azure-storage-selected.png)

2. Cliquez avec le bouton droit pour sélectionner l’élément de sauvegarde. Selon que l’élément de sauvegarde est protégé ou non, le menu présente soit le volet **Arrêter la sauvegarde**, soit le volet **Supprimer les données de sauvegarde**.

    - Si le volet **Arrêter la sauvegarde** s’affiche, sélectionnez **Supprimer les données de sauvegarde** dans le menu déroulant. Entrez le nom de l’élément de sauvegarde (ce champ respecte la casse), puis sélectionnez une raison dans le menu déroulant. Entrez vos commentaires, le cas échéant. Ensuite, sélectionnez **Arrêter la sauvegarde**.

        ![Volet Arrêter la sauvegarde.](./media/backup-azure-delete-vault/stop-backup-item.png)

    - Si le volet **Supprimer les données de sauvegarde** s’affiche, entrez le nom de l’élément de sauvegarde (ce champ respecte la casse), puis sélectionnez une raison dans le menu déroulant. Entrez vos commentaires, le cas échéant. Ensuite, sélectionnez **Supprimer**.

         ![Volet Supprimer les données de sauvegarde.](./media/backup-azure-delete-vault/stop-backup-blade-delete-backup-data.png)

3. Vérifiez l’icône **Notification** : ![Icône Notification.](./media/backup-azure-delete-vault/messages.png) Une fois le processus terminé, le service affiche le message suivant : *Arrêt de la sauvegarde et suppression des données de sauvegarde pour «*  Élément de sauvegarde  *»* . *L’opération s’est terminée correctement.*
4. Sélectionnez **Actualiser** dans le menu **Éléments de sauvegarde** pour vous assurer que l’élément de sauvegarde a été supprimé.

      ![Page Supprimer des éléments de sauvegarde.](./media/backup-azure-delete-vault/empty-items-list.png)

## <a name="delete-protected-items-on-premises"></a>Supprimer des éléments protégés localement

Commencez par lire la section **[Avant de commencer](#before-you-start)** pour comprendre le processus de suppression des dépendances et du coffre.

1. Dans le menu du tableau de bord du coffre, sélectionnez **Infrastructure de sauvegarde**.
2. Selon votre scénario local, choisissez l’une des options suivantes :

      - Pour MARS, sélectionnez **Serveurs protégés**, puis **Agent Azure Backup**. Ensuite, sélectionnez le serveur à supprimer.

        ![Pour MARS, sélectionnez votre coffre pour en ouvrir le tableau de bord.](./media/backup-azure-delete-vault/identify-protected-servers.png)

      - Pour MABS ou DPM, sélectionnez **Serveurs de gestion de sauvegarde**. Ensuite, sélectionnez le serveur à supprimer.

          ![Pour MABS ou DPM, sélectionnez votre coffre pour en ouvrir le tableau de bord.](./media/backup-azure-delete-vault/delete-backup-management-servers.png)

3. Le volet **Supprimer** s’affiche avec un message d’avertissement.

     ![Volet Supprimer.](./media/backup-azure-delete-vault/delete-protected-server.png)

     Consultez le message d’avertissement et les instructions fournies dans la case à cocher de consentement.
    > [!NOTE]
    >
    >- Si le serveur protégé est synchronisé avec les services Azure et que les éléments de sauvegarde existent, la case à cocher de consentement indique le nombre d’éléments de sauvegarde dépendants et le lien pour les consulter.
    >- Si le serveur protégé n’est pas synchronisé avec les services Azure et qu’il existe des éléments de sauvegarde, la case à cocher de consentement présente uniquement le nombre d’éléments de sauvegarde.
    >- En l’absence d’éléments de sauvegarde, la case à cocher de consentement demande une suppression.

4. Cochez la case de consentement, puis sélectionnez **Supprimer**.

5. Vérifiez l’icône **Notification** ![suppression des données de sauvegarde](./media/backup-azure-delete-vault/messages.png). Une fois l’opération terminée, le service affiche le message suivant : *Arrêt de la sauvegarde et suppression des données de sauvegarde pour « Élément de sauvegarde ».* *L’opération s’est terminée correctement.*
6. Sélectionnez **Actualiser** dans le menu **Éléments de sauvegarde** pour vous assurer que l’élément de sauvegarde est supprimé.

>[!NOTE]
>Si vous supprimez un élément protégé local d’un portail qui contient des dépendances, vous recevez un avertissement indiquant « La suppression de l’inscription du serveur est une opération destructrice qui ne peut pas être annulée. Toutes les données de sauvegarde (points de récupération nécessaires pour restaurer les données) et les éléments de sauvegarde associés au serveur protégé seront définitivement supprimés. ».

Une fois ce processus terminé, vous pouvez supprimer les éléments de sauvegarde de la console de gestion :

- [Supprimer des éléments de sauvegarde de la console de gestion MARS](#delete-backup-items-from-the-mars-management-console)
- [Supprimer des éléments de sauvegarde de la console de gestion MABS ou DPM](#delete-backup-items-from-the-mabs-or-dpm-management-console)

### <a name="delete-backup-items-from-the-mars-management-console"></a>Supprimer des éléments de sauvegarde de la console de gestion MARS

>[!NOTE]
>Si vous avez supprimé ou perdu la machine source sans arrêter la sauvegarde, la prochaine sauvegarde planifiée échoue. L’ancien point de récupération expire conformément à la stratégie, mais le dernier point de récupération unique est conservé jusqu’à ce que vous arrêtiez la sauvegarde et supprimiez les données. Pour ce faire, vous pouvez suivre les étapes décrites dans [cette section](#delete-protected-items-on-premises).

1. Ouvrez la console de gestion de MARS, accédez au volet **Actions** et sélectionnez **Planifier la sauvegarde**.
2. Dans la page **Modifier ou arrêter une sauvegarde planifiée**, sélectionnez **Ne plus utiliser cette panification de sauvegarde et supprimer toutes les sauvegardes stockées**. Ensuite, sélectionnez **Suivant**.

    ![Modifiez ou arrêtez une sauvegarde planifiée.](./media/backup-azure-delete-vault/modify-schedule-backup.png)

3. Dans la page **Arrêter une sauvegarde planifiée**, sélectionnez **Terminer**.

    ![Arrêtez une sauvegarde planifiée.](./media/backup-azure-delete-vault/stop-schedule-backup.png)
4. Il vous est demandé d’entrer un code PIN (Personal Identification Number) de sécurité, que vous devez générer manuellement. Pour cela, commencez par vous connecter au portail Azure.
5. Accédez à **Coffre Recovery Services** > **Paramètres** > **Propriétés**.
6. Sous **Code PIN de sécurité**, sélectionnez **Générer**. Copiez ce code PIN. Il n’est valide que pendant cinq minutes.
7. Dans la console de gestion, collez le code PIN, puis sélectionnez **OK**.

    ![Générez un code PIN de sécurité.](./media/backup-azure-delete-vault/security-pin.png)

8. Dans la page **Modifier la progression de la sauvegarde**, le message suivant s’affiche : *Les données supprimées sont conservées pendant 14 jours. Passé ce délai, les données de sauvegarde sont définitivement supprimées.*  

    ![Supprimez l’infrastructure de sauvegarde.](./media/backup-azure-delete-vault/deleted-backup-data.png)

Une fois les éléments de sauvegarde locaux supprimés, effectuez les étapes suivantes à partir du portail.

### <a name="delete-backup-items-from-the-mabs-or-dpm-management-console"></a>Supprimer des éléments de sauvegarde de la console de gestion MABS ou DPM

>[!NOTE]
>Si vous avez supprimé ou perdu la machine source sans arrêter la sauvegarde, la prochaine sauvegarde planifiée échoue. L’ancien point de récupération expire conformément à la stratégie, mais le dernier point de récupération unique est conservé jusqu’à ce que vous arrêtiez la sauvegarde et supprimiez les données. Pour ce faire, vous pouvez suivre les étapes décrites dans [cette section](#delete-protected-items-on-premises).

Il existe deux méthodes pour supprimer des éléments de sauvegarde de la console de gestion MABS ou DPM.

#### <a name="method-1"></a>Méthode 1

Pour arrêter la protection et supprimer des données de sauvegarde, effectuez les étapes suivantes :

1. Ouvrez la console Administrateur DPM, puis sélectionnez **Protection** dans la barre de navigation.
2. Dans le volet d’informations, sélectionnez le membre du groupe de protection que vous souhaitez supprimer. Cliquez avec le bouton droit pour sélectionner l’option **Arrêter la protection des membres du groupe**.
3. Dans la boîte de dialogue **Arrêter la protection**, cochez la case **Supprimer les données protégées**, puis **Supprimer le stockage en ligne**. Ensuite, sélectionnez **Arrêter la protection**.

    ![Sélectionnez Supprimer les données protégées dans le volet Arrêter la protection.](./media/backup-azure-delete-vault/delete-storage-online.png)

    L’état du membre protégé devient *Réplica inactif disponible*.

4. Cliquez avec le bouton droit sur le groupe de protection inactif, puis sélectionnez **Supprimer la protection inactive**.

    ![Supprimez la protection inactive.](./media/backup-azure-delete-vault/remove-inactive-protection.png)

5. Dans la fenêtre **Supprimer la protection inactive**, cochez la case **Supprimer le stockage en ligne**, puis sélectionnez **OK**.

    ![Supprimez le stockage en ligne.](./media/backup-azure-delete-vault/remove-replica-on-disk-and-online.png)

#### <a name="method-2"></a>Méthode 2

Ouvrez la console de **gestion MABS** ou de **gestion DPM**. Sous **Sélectionner la méthode de protection des données**, décochez la case **Je voudrais une protection en ligne**.

  ![Sélectionnez la méthode de protection des données.](./media/backup-azure-delete-vault/data-protection-method.png)

Une fois les éléments de sauvegarde locaux supprimés, effectuez les étapes suivantes à partir du portail.

## <a name="delete-the-recovery-services-vault"></a>Supprimer le coffre Recovery Services

1. Une fois toutes les dépendances supprimées, faites défiler l’affichage jusqu’au volet **Bases** dans le menu du coffre.
2. Vérifiez qu’aucun élément de sauvegarde, serveur de gestion de sauvegarde ni élément répliqué n’est listé. Si des éléments apparaissent encore dans le coffre, reportez-vous à la section [Avant de commencer](#before-you-start).

3. Lorsqu’il n’y a plus d’éléments dans le coffre, sélectionnez **Supprimer** dans le tableau de bord du coffre.

    ![Sélectionnez Supprimer dans le tableau de bord du coffre.](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

4. Sélectionnez **Oui** pour confirmer la suppression du coffre. Le coffre est supprimé. Le portail revient au menu de service **Nouveau**.

## <a name="delete-the-recovery-services-vault-by-using-powershell"></a>Supprimer le coffre Recovery Services à l’aide de PowerShell

Commencez par lire la section **[Avant de commencer](#before-you-start)** pour comprendre le processus de suppression des dépendances et du coffre.

Pour arrêter la protection et supprimer les données de sauvegarde :

- Si vous utilisez SQL dans la sauvegarde de machines virtuelles Azure et que vous activez la protection automatique pour les instances SQL, désactivez d’abord la protection automatique.

    ```PowerShell
        Disable-AzRecoveryServicesBackupAutoProtection
           [-InputItem] <ProtectableItemBase>
           [-BackupManagementType] <BackupManagementType>
           [-WorkloadType] <WorkloadType>
           [-PassThru]
           [-VaultId <String>]
           [-DefaultProfile <IAzureContextContainer>]
           [-WhatIf]
           [-Confirm]
           [<CommonParameters>]
    ```

  [En savoir plus](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupautoprotection) sur la désactivation de la protection d’un élément protégé par la Sauvegarde Azure.

- Arrêtez la protection et supprimez les données de tous les éléments protégés par la sauvegarde dans le cloud (par exemple, une machine virtuelle laaS, un partage de fichiers Azure, etc.) :

    ```PowerShell
       Disable-AzRecoveryServicesBackupProtection
       [-Item] <ItemBase>
       [-RemoveRecoveryPoints]
       [-Force]
       [-VaultId <String>]
       [-DefaultProfile <IAzureContextContainer>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
    ```

    [En savoir plus](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection)  sur la désactivation de la protection d’un élément protégé par la Sauvegarde.

- Pour les fichiers et dossiers locaux protégés à l’aide de la sauvegarde Azure Backup Agent (MARS) sur Azure, utilisez la commande PowerShell suivante pour supprimer les données sauvegardées de chaque module PowerShell MARS :

    ```powershell
    Get-OBPolicy | Remove-OBPolicy -DeleteBackup -SecurityPIN <Security Pin>
    ```

    Par la suite, l’invite suivante s’affiche :

    *Sauvegarde Microsoft Azure  Voulez-vous vraiment supprimer cette stratégie de sauvegarde ? Les données de sauvegarde supprimées sont conservées pendant 14 jours. Passé ce délai, les données de sauvegarde sont supprimées définitivement. <br/> [O] Oui [T] Oui pour tout [N] Non [R] Non pour tout [I] Interrompre [ ?] Aide (la valeur par défaut est « O ») :*

- Pour les machines locales protégées par MABS (serveur de sauvegarde Microsoft Azure) ou DPM (System Center Data Protection Manager) sur Azure, utilisez la commande suivante pour supprimer les données sauvegardées dans Azure.

    ```powershell
    Get-OBPolicy | Remove-OBPolicy -DeleteBackup -SecurityPIN <Security Pin>
    ```

    Par la suite, l’invite suivante s’affiche :

   *Sauvegarde Microsoft Azure* Voulez-vous vraiment supprimer cette stratégie de sauvegarde ? Les données de sauvegarde supprimées sont conservées pendant 14 jours. Passé ce délai, les données de sauvegarde sont définitivement supprimées. <br/>
   [Y] Oui [A] Oui pour tout [N] non [L] Non pour tout [S] Suspendre [?] Aide (la valeur par défaut est « Y ») :

Après avoir supprimé les données sauvegardées, annulez l’inscription de tous les serveurs d’administration et conteneurs locaux.

- Pour les fichiers et dossiers locaux protégés à l’aide de la sauvegarde Azure Backup Agent (MARS) sur Azure :

    ```PowerShell
    Unregister-AzRecoveryServicesBackupContainer
              [-Container] <ContainerBase>
              [-PassThru]
              [-VaultId <String>]
              [-DefaultProfile <IAzureContextContainer>]
              [-WhatIf]
              [-Confirm]
              [<CommonParameters>]
    ```

    [En savoir plus](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) sur l’annulation de l’inscription d’un conteneur Windows Server ou d’un autre conteneur auprès du coffre.

- Pour les machines locales protégées à l’aide de MABS ou de DPM sur Azure :

    ```PowerShell
        Unregister-AzRecoveryServicesBackupManagementServer
          [-AzureRmBackupManagementServer] <BackupEngineBase>
          [-PassThru]
          [-VaultId <String>]
          [-DefaultProfile <IAzureContextContainer>]
          [-WhatIf]
          [-Confirm]
          [<CommonParameters>]
    ```

    [En savoir plus](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) sur l’annulation de l’inscription d’un conteneur de gestion de la Sauvegarde auprès du coffre.

Après avoir supprimé définitivement des données sauvegardées et annulé l’inscription de tous les conteneurs, supprimez le coffre.

Pour supprimer un coffre Recovery Services :

   ```PowerShell
       Remove-AzRecoveryServicesVault
      -Vault <ARSVault>
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
   ```

[Découvrez-en plus ](/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault) sur la suppression d’un coffre Recovery Services.

## <a name="delete-the-recovery-services-vault-by-using-cli"></a>Supprimer le coffre Recovery Services à l’aide de l’interface de ligne de commande (CLI)

Commencez par lire la section **[Avant de commencer](#before-you-start)** pour comprendre le processus de suppression des dépendances et du coffre.

> [!NOTE]
> Actuellement, l’interface de ligne de commande (CLI) de la Sauvegarde Azure prend en charge la gestion des sauvegardes de machines virtuelles Azure uniquement. Par conséquent, la commande suivante destinée à supprimer le coffre fonctionne seulement si le coffre contient des sauvegardes de machines virtuelles Azure. Vous ne pouvez pas supprimer un coffre à l’aide de l’interface CLI de sauvegarde Azure si le coffre contient un élément de sauvegarde d’un type autre que des machines virtuelles Azure.

Pour supprimer un coffre Recovery Services existant, procédez comme suit :

- Pour arrêter la protection et supprimer les données de sauvegarde

    ```azurecli
    az backup protection disable --container-name
                             --item-name
                             [--delete-backup-data {false, true}]
                             [--ids]
                             [--resource-group]
                             [--subscription]
                             [--vault-name]
                             [--yes]
    ```

    Pour plus d’informations, consultez cet  [article](/cli/azure/backup/protection#az-backup-protection-disable).

- Supprimez un coffre Recovery Services existant :

    ```azurecli
    az backup vault delete [--force]
                       [--ids]
                       [--name]
                       [--resource-group]
                       [--subscription]
                       [--yes]
    ```

    Pour plus d’informations, consultez cet  [article](/cli/azure/backup/vault).

## <a name="delete-the-recovery-services-vault-by-using-azure-resource-manager"></a>Supprimer le coffre Recovery Services à l’aide d’Azure Resource Manager

Cette option de suppression du coffre Recovery Services est recommandée uniquement si toutes les dépendances sont supprimées alors que vous obtenez toujours le *message d’erreur de suppression du coffre*. Essayez l’une ou l’ensemble des astuces suivantes :

- Dans le volet **Bases** du menu du coffre, vérifiez qu’il n’y a pas d’éléments de sauvegarde, de serveurs de gestion de sauvegarde ou d’éléments répliqués listés. S’il existe des éléments de sauvegarde, reportez-vous à la section [Avant de commencer](#before-you-start).
- Réessayez de [supprimer le coffre du portail](#delete-the-recovery-services-vault).
- Si toutes les dépendances sont supprimées et que vous recevez toujours le *message d’erreur de suppression du coffre*, utilisez l’outil ARMClient pour effectuer les étapes suivantes (après la remarque).

1. Accédez à [chocolatey.org](https://chocolatey.org/) pour télécharger et installer Chocolatey. Ensuite, installez ARMClient en exécutant la commande suivante :

   `choco install armclient --source=https://chocolatey.org/api/v2/`
2. Connectez-vous à votre compte Azure, puis exécutez la commande suivante :

    `ARMClient.exe login [environment name]`

3. Dans le portail Azure, rassemblez l’ID d’abonnement et le nom du groupe de ressources pour le coffre à supprimer.

Pour plus d’informations sur la commande ARMClient, consultez le fichier [ARMClient README](https://github.com/projectkudu/ARMClient/blob/master/README.md).

### <a name="use-the-azure-resource-manager-client-to-delete-a-recovery-services-vault"></a>Utiliser le client Azure Resource Manager pour supprimer un coffre Recovery Services

1. Exécutez la commande suivante à l’aide de votre ID d’abonnement, du nom du groupe de ressources et du nom du coffre. Si vous n’avez pas de dépendances, le coffre est supprimé au moment où vous exécutez la commande suivante :

   ```azurepowershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<Recovery Services vault name>?api-version=2015-03-15
   ```

2. Si le coffre n’est pas vide, vous recevez le message d’erreur suivant : *Impossible de supprimer le coffre car celui-ci contient des ressources.* Pour supprimer un élément ou conteneur protégé au sein d’un coffre, exécutez la commande suivante :

   ```azurepowershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<Recovery Services vault name>/registeredIdentities/<container name>?api-version=2016-06-01
   ```

3. Dans le portail Azure, vérifiez que le coffre est supprimé.

## <a name="next-steps"></a>Étapes suivantes

[Découvrez les coffres Recovery Services](backup-azure-recovery-services-vault-overview.md)
[Découvrez la supervision et la gestion des coffres Recovery Services](backup-azure-manage-windows-server.md)
