---
title: Sauvegarder des bases de données SQL Server sur Azure
description: Cet article explique comment sauvegarder SQL Server avec Azure, ainsi que la récupération de SQL Server.
ms.topic: conceptual
ms.date: 06/18/2019
ms.openlocfilehash: 510d9637031928e31abaa5f82a5bf58c6ef44719
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91316837"
---
# <a name="about-sql-server-backup-in-azure-vms"></a>À propos de la sauvegarde SQL Server sur des machines virtuelles Azure

La [sauvegarde Azure](backup-overview.md) offre une solution spécialisée basée sur les flux pour sauvegarder SQL Server quand il s’exécute sur des machines virtuelles Azure. Cette solution s’aligne sur les avantages de la sauvegarde Azure : sauvegarde sans infrastructure, conservation à long terme et gestion centralisée. Elle offre par ailleurs les avantages suivants, en particulier pour SQL Server :

1. Sauvegardes de charges de travail prenant en charge tous les types de sauvegarde : complète, différentielle et de fichier journal
2. RPO (objectif de point de récupération) de 15 minutes avec des sauvegardes de fichier journal fréquentes
3. Récupération jusqu’à une date et heure d’une seconde maximum
4. Sauvegarde et restauration individuelles au niveau de la base de données

Pour voir les scénarios de sauvegarde et de restauration que nous prenons en charge aujourd’hui, consultez la [matrice de prise en charge](sql-support-matrix.md#scenario-support).

## <a name="backup-process"></a>Processus de sauvegarde

Cette solution exploite les API natives de SQL pour effectuer des sauvegardes de vos bases de données SQL.

* Une fois que vous avez spécifié la machine virtuelle SQL Server que vous voulez protéger et dont vous voulez interroger des bases de données, le service Sauvegarde Azure installe une extension de sauvegarde de charge de travail sur la machine virtuelle nommée `AzureBackupWindowsWorkload`.
* Cette extension se compose d’un coordinateur et d’un plug-in SQL. Alors que le coordinateur est responsable du déclenchement des flux de travail pour diverses opérations, comme le configuration de la sauvegarde, la sauvegarde et la restauration, le plug-in est responsable du flux de données réel.
* Pour pouvoir découvrir les bases de données sur cette machine virtuelle, Sauvegarde Azure crée le compte `NT SERVICE\AzureWLBackupPluginSvc`. Ce compte est utilisé pour la sauvegarde et la restauration. Il doit disposer d’autorisations d’administrateur système SQL. Le compte `NT SERVICE\AzureWLBackupPluginSvc` est un [compte de service virtuel](/windows/security/identity-protection/access-control/service-accounts#virtual-accounts) et ne nécessite donc pas de gestion des mots de passe. Le service de sauvegarde Azure utilise le compte `NT AUTHORITY\SYSTEM` pour la découverte et la recherche des bases de données. Ce compte doit donc être une connexion publique sur SQL. Si vous n’avez pas créé la machine virtuelle SQL Server à partir de la Place de marché Azure, il se peut que vous receviez une erreur **UserErrorSQLNoSysadminMembership**. Si cela se produit, [suivez ces instructions](#set-vm-permissions).
* Lorsque vous déclenchez la configuration de la protection sur les bases de données sélectionnées, le service de sauvegarde configure le coordinateur avec les planifications de sauvegarde et d’autres détails de stratégie, ce que l’extension met en cache localement sur la machine virtuelle.
* À l’heure planifiée, le coordinateur communique avec le plug-in et démarre le streaming des données de sauvegarde à partir du serveur SQL avec l’infrastructure VDI.  
* Le plug-in envoie les données directement au coffre Recovery Services, ce qui élimine la nécessité d’un emplacement intermédiaire. Les données sont chiffrées et stockées par le service de Sauvegarde Azure dans des comptes de stockage.
* Une fois le transfert de données terminé, le coordinateur confirme la validation avec le service de sauvegarde.

  ![Architecture de sauvegarde SQL](./media/backup-azure-sql-database/azure-backup-sql-overview.png)

## <a name="before-you-start"></a>Avant de commencer

Avant de commencer, contrôlez les exigences suivantes :

1. Vérifier qu’une instance SQL Server s’exécute dans Azure. Vous pouvez [rapidement créer une instance SQL Server](../azure-sql/virtual-machines/windows/sql-vm-create-portal-quickstart.md) dans la Place de marché.
2. Consultez les sections [Considérations relatives aux fonctionnalités](sql-support-matrix.md#feature-considerations-and-limitations) et [Prise en charge de scénarios](sql-support-matrix.md#scenario-support).
3. [Consulter les questions les plus fréquentes](faq-backup-sql-server.md) sur ce scénario.

## <a name="set-vm-permissions"></a>Définir des autorisations de machine virtuelle

  Lorsque vous exécutez la découverte sur un serveur SQL Server, Sauvegarde Azure effectue les opérations suivantes :

* Ajoute l’extension AzureBackupWindowsWorkload.
* Crée un compte NT SERVICE\AzureWLBackupPluginSvc pour découvrir les bases de données sur la machine virtuelle. Ce compte est utilisé pour la sauvegarde et la restauration, et doit disposer d’autorisations d’administrateur système SQL.
* Découvre les bases de données en cours d’exécution sur une machine virtuelle ; Sauvegarde Azure utilise le compte NT AUTHORITY\SYSTEM. Ce compte doit être une connexion publique sur SQL.

Si vous n’avez pas créé la machine virtuelle SQL Server dans la Place de marché Azure, ou si vous êtes sur SQL 2008 ou 2008 R2, il se peut que vous receviez une erreur **UserErrorSQLNoSysadminMembership**.

Pour accorder des autorisations dans le cas de **SQL 2008** et **2008 R2** s’exécutant sur Windows 2008 R2, voir [ici](#give-sql-sysadmin-permissions-for-sql-2008-and-sql-2008-r2).

Pour toutes les autres versions, corrigez les autorisations en procédant comme suit :

  1. Utilisez un compte disposant d’autorisations d’administrateur système SQL Server pour vous connecter à SQL Server Management Studio (SSMS). À moins que vous ayez besoin d’autorisations spéciales, l’authentification Windows doit fonctionner.
  2. Sur le serveur SQL Server, ouvrez le dossier **Security/Logins**.

      ![Ouvrir le dossier Security/Logins pour afficher les comptes](./media/backup-azure-sql-database/security-login-list.png)

  3. Cliquez avec le bouton droit sur le dossier **Logins** et sélectionnez **Nouvelle connexion**. Dans **Nouvelle connexion**, sélectionnez **Rechercher**.

      ![Dans la boîte de dialogue Connexion - Nouveau, sélectionnez Rechercher](./media/backup-azure-sql-database/new-login-search.png)

  4. Le compte de service virtuel Windows **NT SERVICE\AzureWLBackupPluginSvc** a été créé pendant la phase d’inscription de la machine virtuelle et de découverte SQL. Entrez le nom du compte, comme indiqué dans **Entrez le nom de l’objet à sélectionner**. Sélectionnez **Vérifier les noms** pour résoudre le nom. Sélectionnez **OK**.

      ![Cliquer sur Vérifier les noms pour résoudre le nom de service inconnu](./media/backup-azure-sql-database/check-name.png)

  5. Dans **Rôles du serveur**, assurez-vous que le rôle **sysadmin** (administrateur système) est sélectionné. Sélectionnez **OK**. Les autorisations requises doivent désormais exister.

      ![S’assurer que le rôle serveur administrateur système est sélectionné](./media/backup-azure-sql-database/sysadmin-server-role.png)

  6. À présent, associez la base de données avec le coffre Recovery Services. Dans la liste **Serveurs protégés** du portail Azure, cliquez avec le bouton droit sur le serveur en erreur > **Redécouvrir les bases de données**.

      ![Vérifier que le serveur dispose des autorisations appropriées](./media/backup-azure-sql-database/check-erroneous-server.png)

  7. Vérifiez la progression dans la zone **Notifications**. Lorsque les bases de données sélectionnées ont été détectées, un message de réussite s’affiche.

      ![Message Déploiement réussi](./media/backup-azure-sql-database/notifications-db-discovered.png)

> [!NOTE]
> Si plusieurs instances de SQL Server sont installées sur votre serveur SQL Server, vous devez ajouter l’autorisation sysadmin du compte **NT Service\AzureWLBackupPluginSvc** à toutes les instances SQL.

### <a name="give-sql-sysadmin-permissions-for-sql-2008-and-sql-2008-r2"></a>Accorder des autorisations d’administrateur système SQL pour SQL 2008 et SQL 2008 R2

Ajouter des connexions **NT AUTHORITY\SYSTEM** et **NT Service\AzureWLBackupPluginSvc** à l’Instance SQL Server :

1. Accédez à l’Instance SQL Server dans l’Explorateur d’objets.
2. Accédez à Sécurité -> Connexions
3. Cliquez avec le bouton droit sur les connexions, puis sélectionnez *Nouvelle connexion*.

    ![Nouvelle connexion à l’aide de SSML](media/backup-azure-sql-database/sql-2k8-new-login-ssms.png)

4. Accédez à l’onglet Général et entrez **NT AUTHORITY\SYSTEM** en tant que nom de connexion.

    ![Nom de connexion pour SSMS](media/backup-azure-sql-database/sql-2k8-nt-authority-ssms.png)

5. Accédez à *Rôles de serveur*, puis choisissez les rôles *public* et *sysadmin*.

    ![Choix de rôles dans SSMS](media/backup-azure-sql-database/sql-2k8-server-roles-ssms.png)

6. Cliquez sur *État*. *Accordez* l’autorisation de se connecter au moteur de base de données et de se connecter en tant que *Activé*.

    ![Accorder des autorisations dans SSMS](media/backup-azure-sql-database/sql-2k8-grant-permission-ssms.png)

7. Sélectionnez OK.
8. Répétez la même séquence d’étapes (1 à 7 ci-dessus) pour ajouter la connexion NT Service\AzureWLBackupPluginSvc à l’instance SQL Server. Si la connexion existe, assurez-vous qu’elle a le rôle serveur sysadmin et que, sous État, l’autorisation lui est accordée de se connecter au moteur de base de données et de se connecter en tant que Activé.
9. Une fois l’autorisation octroyée, **redécouvrez les bases de données** dans le portail : Coffre **->** Infrastructure de sauvegarde **->** Charge de travail dans machine virtuelle Azure :

    ![Redécouvrir les bases de données dans le portail Azure](media/backup-azure-sql-database/sql-rediscover-dbs.png)

Vous pouvez aussi automatiser l’octroi d’autorisations en exécutant les commandes PowerShell suivantes en mode Administrateur. Le nom d’instance a la valeur MSSQLSERVER par défaut. Modifiez l’argument de nom d’instance dans le script si nécessaire :

```powershell
param(
    [Parameter(Mandatory=$false)]
    [string] $InstanceName = "MSSQLSERVER"
)
if ($InstanceName -eq "MSSQLSERVER")
{
    $fullInstance = $env:COMPUTERNAME   # In case it is the default SQL Server Instance
}
else
{
    $fullInstance = $env:COMPUTERNAME + "\" + $InstanceName   # In case of named instance
}
try
{
    sqlcmd.exe -S $fullInstance -Q "sp_addsrvrolemember 'NT Service\AzureWLBackupPluginSvc', 'sysadmin'" # Adds login with sysadmin permission if already not available
}
catch
{
    Write-Host "An error occurred:"
    Write-Host $_.Exception|format-list -force
}
try
{
    sqlcmd.exe -S $fullInstance -Q "sp_addsrvrolemember 'NT AUTHORITY\SYSTEM', 'sysadmin'" # Adds login with sysadmin permission if already not available
}
catch
{
    Write-Host "An error occurred:"
    Write-Host $_.Exception|format-list -force
}
```

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur](backup-sql-server-database-azure-vms.md) la sauvegarde de bases de données SQL Server.
* [En savoir plus sur](restore-sql-database-azure-vm.md) la restauration des bases de données SQL Server sauvegardées.
* [En savoir plus sur](manage-monitor-sql-database-backup.md) la gestion des bases de données SQL Server sauvegardées.
