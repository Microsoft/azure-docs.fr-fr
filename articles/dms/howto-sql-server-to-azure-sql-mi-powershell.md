---
title: 'PowerShell : Migrer SQL Server vers SQL Managed Instance'
titleSuffix: Azure Database Migration Service
description: Apprenez à migrer de SQL Server vers Azure SQL Managed Instance à l’aide d’Azure PowerShell et du service Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019,fasttrack-edit, devx-track-azurepowershell
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: aa97fa5e92d4afe11cf8af5f7cfd3458a3c12014
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302722"
---
# <a name="migrate-sql-server-to-sql-managed-instance-with-powershell--azure-database-migration-service"></a>Migrer SQL Server vers SQL Managed Instance avec PowerShell et Azure Database Migration Service

Dans cet article, vous allez migrer la base de données **AdventureWorks2016** restaurée vers une instance locale de SQL Server 2005 ou ultérieure vers une instance Azure SQL Managed Instance à l’aide de Microsoft Azure PowerShell. Vous pouvez migrer des bases de données à partir d’une instance SQL Server vers une instance Azure SQL Managed Instance à l’aide du module `Az.DataMigration` dans Microsoft Azure PowerShell.

Dans cet article, vous apprendrez comment :
> [!div class="checklist"]
>
> * Créez un groupe de ressources.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration dans une instance Azure Database Migration Service.
> * Exécuter la migration.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Cet article contient des détails sur la manière d’effectuer des migrations en ligne et hors connexion.

## <a name="prerequisites"></a>Prérequis

Pour effectuer cette procédure, vous avez besoin de :

* [SQL Server 2016 ou version ultérieure](https://www.microsoft.com/sql-server/sql-server-downloads) (toute édition).
* Une copie locale de la base de données **AdventureWorks2016**, qui est disponible au téléchargement [ici](/sql/samples/adventureworks-install-configure?view=sql-server-2017).
* Activer le protocole TCP/IP, qui est désactivé par défaut avec l’installation de SQL Server Express. Activer le protocole TCP/IP en suivant l’article [Activer ou désactiver un protocole réseau de serveur](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
* Pour configurer votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Un abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Instance SQL Managed Instance. Créez une instance SQL Managed Instance en suivant les indications de l'article [Créer une instance SQL Managed Instance](../azure-sql/managed-instance/instance-create-quickstart.md).
* Téléchargez et installez l’[Assistant Migration de données](https://www.microsoft.com/download/details.aspx?id=53595) v3.3 ou version ultérieure.
* Un réseau virtuel Microsoft Azure créé à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité de site à site à vos serveurs sources locaux pour le service Azure Database Migration Service via [ExpressRoute](../expressroute/expressroute-introduction.md) ou une passerelle [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* L’évaluation de la base de données locale et de la migration de schéma à l’aide de l’Assistant Migration de données Microsoft, comme indiqué dans l’article [Évaluation de la migration de SQL Server](/sql/dma/dma-assesssqlonprem).
* Téléchargez et installez le module `Az.DataMigration` (version 0.7.2 ou ultérieure) à partir de PowerShell Gallery en utilisant la [cmdlet PowerShell Install-Module](/powershell/module/powershellget/Install-Module?view=powershell-5.1).
* Pour vous assurer que les informations d’identification utilisées pour se connecter à une instance SQL Server source disposent de l’autorisation [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Pour vous assurer que les informations d’identification utilisées pour se connecter à une instance Azure Managed Instance cible disposent de l’autorisation CONTROL DATABASE sur les bases de données SQL Managed Instance cibles.

    > [!IMPORTANT]
    > Pour les migrations en ligne, vous devez déjà avoir configuré vos informations d’identification Azure Active Directory. Pour plus d’informations, consultez l’article [Utiliser le portail pour créer une application et un principal du service Azure AD pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Se connecter à un abonnement Microsoft Azure

Connectez-vous à votre abonnement Azure à l’aide de PowerShell. Pour plus d’informations, consultez l’article [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup).

L’exemple suivant permet de créer un groupe de ressources nommé *myResourceGroup* dans la région *USA Est*.

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>Créer une instance Azure Database Migration Service

Vous pouvez créer une instance Azure Database Migration Service à l’aide de l’applet de commande `New-AzDataMigrationService`.
Cette cmdlet attend les paramètres requis suivants :

* *Nom du groupe de ressources Azure*. Vous pouvez utiliser la commande [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup) pour créer un groupe de ressources Azure indiqué précédemment et indiquez son nom en tant que paramètre.
* *Nom du service*. Chaîne qui correspond au nom de service unique à donner à l’instance Azure Database Migration Service.
* *Emplacement*. Spécifie l’emplacement du service. Indiquez un emplacement de centre de données Azure, tel que USA Ouest ou Asie Sud-Est.
* *Référence SKU*. Ce paramètre correspond au nom de référence SKU DMS. Les noms de référence SKU actuellement pris en charge sont *Basic_1vCore*, *Basic_2vCores* et *GeneralPurpose_4vCores*.
* *Identificateur de sous-réseau virtuel*. Vous pouvez utiliser la cmdlet [`New-AzVirtualNetworkSubnetConfig`](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) pour créer un sous-réseau.

L’exemple suivant crée un service nommé *MyDMS* dans le groupe de ressources *MyDMSResourceGroup*, qui se trouve dans la région *USA Est* à l’aide d’un réseau virtuel appelé *MyVNET* et d’un sous-réseau appelé *MySubnet*.

> [!IMPORTANT]
> L’extrait de code ci-dessous est destiné à une migration hors connexion, qui ne nécessite pas une instance d’Azure Database Migration Service basée sur une référence SKU Premium. Pour une migration en ligne, la valeur du paramètre SKU doit inclure une référence SKU Premium.

```powershell
$vNet = Get-AzVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois l’instance Azure Database Migration Service créée, générez un projet de migration. Un projet Azure Database Migration Service requiert des informations de connexion pour les instances source et cible, ainsi que la liste des bases de données que vous souhaitez migrer dans le cadre de ce projet.

### <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>Créer un objet d’informations de connexion de base de données pour les connexions source et cible

Vous pouvez créer un objet d’informations de connexion de base de données à l’aide de la cmdlet `New-AzDmsConnInfo`, qui attend les paramètres suivants :

* *ServerType*. Type de connexion de base de données demandé, par exemple, MySQL, Oracle ou MySQL. Utilisez SQL pour SQL Server et Azure SQL.
* *DataSource*. Nom ou adresse IP d’une instance SQL Server ou d’une instance de base de données Azure SQL.
* *AuthType*. Type d’authentification pour la connexion, qui peut être SqlAuthentication ou WindowsAuthentication.
* *TrustServerCertificate*. Ce paramètre définit une valeur qui indique si le canal est chiffré tout en ignorant l’analyse de la chaîne de certificat pour valider l’approbation. La valeur peut être `$true` ou `$false`.

L’exemple suivant crée un objet d’informations de connexion pour une instance SQL Server source, appelée *MySourceSQLServer* avec l’authentification SQL :

```powershell
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

L’exemple suivant illustre la création de l’objet d’informations de connexion pour une instance Azure SQL Managed Instance appelée « targetmanagedinstance » :

```powershell
$targetResourceId = (Get-AzSqlInstance -Name "targetmanagedinstance").Id
$targetConnInfo = New-AzDmsConnInfo -ServerType SQLMI -MiResourceId $targetResourceId
```

### <a name="provide-databases-for-the-migration-project"></a>Fournir des bases de données pour le projet de migration

Créez une liste d’objets `AzDataMigrationDatabaseInfo` spécifiant les bases de données dans le cadre du projet de service Azure Database Migration, qui peut être fourni en tant que paramètre de création du projet. Vous pouvez utiliser la cmdlet `New-AzDataMigrationDatabaseInfo` pour créer `AzDataMigrationDatabaseInfo`.

L’exemple suivant crée le projet `AzDataMigrationDatabaseInfo` pour la base de données **AdventureWorks2016** et l’ajoute à la liste qui doit être fournie en tant que paramètre de création du projet.

```powershell
$dbInfo1 = New-AzDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>Créer un objet de projet

Enfin, vous pouvez créer un projet de service Azure Database Migration appelé *MyDMSProject*, situé dans la région *USA Est*, à l’aide du paramètre `New-AzDataMigrationProject`, et en ajoutant les connexions source et cible créées précédemment et la liste des bases de données à migrer.

```powershell
$project = New-AzDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLMI `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>Créer et démarrer une tâche de migration

Ensuite, créez et démarre une tâche Azure Database Migration Service. Cette tâche nécessite des informations de connexion pour les systèmes source et cible, ainsi que la liste des tables de base de données à migrer, en plus des informations déjà fournies avec le projet créé, en tant que condition préalable.

### <a name="create-credential-parameters-for-source-and-target"></a>Créer des paramètres d’informations d’identification pour la source et la cible

Créez des informations d’identification de sécurité de la connexion en tant qu’objet [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0).

L’exemple suivant illustre la création d’objets *PSCredential* pour les connexions source et cible, en fournissant des mots de passe en tant que variables de chaîne *$sourcePassword* et *$targetPassword*.

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-a-backup-fileshare-object"></a>Créer un objet de partage de fichiers de sauvegarde

Passez maintenant à la création d’un objet de partage de fichiers de sauvegarde représentant le partage réseau SMB local qu’Azure Database Migration Service peut utiliser pour les sauvegardes de bases de données source à l’aide de la cmdlet `New-AzDmsFileShare`.

```powershell
$backupPassword = ConvertTo-SecureString -String $password -AsPlainText -Force
$backupCred = New-Object System.Management.Automation.PSCredential ($backupUserName, $backupPassword)

$backupFileSharePath="\\10.0.0.76\SharedBackup"
$backupFileShare = New-AzDmsFileShare -Path $backupFileSharePath -Credential $backupCred
```

### <a name="create-selected-database-object"></a>Créer l’objet de base de données sélectionné

L’étape suivante consiste à sélectionner les bases de données source et cible à l’aide de la cmdlet `New-AzDmsSelectedDB`.

L’exemple suivant concerne la migration d’une base de données unique à partir de SQL Server vers une instance Azure SQL Managed Instance :

```powershell
$selectedDbs = @()
$selectedDbs += New-AzDmsSelectedDB -MigrateSqlServerSqlDbMi `
  -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -BackupFileShare $backupFileShare `
```

Si une instance SQL Server entière requiert un lift-and-shift dans une instance Azure SQL Managed Instance, une boucle est fournie ci-dessous pour prendre toutes les bases de données de la source. Dans l’exemple suivant, pour $Server, $SourceUserName et $SourcePassword fournissent les détails du serveur SQL Server source.

```powershell
$Query = "(select name as Database_Name from master.sys.databases where Database_id>4)";
$Databases= (Invoke-Sqlcmd -ServerInstance "$Server" -Username $SourceUserName
-Password $SourcePassword -database master -Query $Query)
$selectedDbs=@()
foreach($DataBase in $Databases.Database_Name)
    {
      $SourceDB=$DataBase
      $TargetDB=$DataBase
      
$selectedDbs += New-AzureRmDmsSelectedDB -MigrateSqlServerSqlDbMi `
                                              -Name $SourceDB `
                                              -TargetDatabaseName $TargetDB `
                                              -BackupFileShare $backupFileShare
      }
```

### <a name="sas-uri-for-azure-storage-container"></a>URI SAP pour un conteneur Stockage Azure

Créez une variable contenant l’URI SAP qui donne à Azure Database Migration Service l’accès au conteneur du compte de stockage dans lequel le service charge les fichiers de sauvegarde.

```powershell
$blobSasUri="https://mystorage.blob.core.windows.net/test?st=2018-07-13T18%3A10%3A33Z&se=2019-07-14T18%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=qKlSA512EVtest3xYjvUg139tYSDrasbftY%3D"
```

> [!NOTE]
> Azure Database Migration Service ne prend pas en charge l’utilisation d’un jetons SAS au niveau du compte. Vous devez utiliser un URI SAS pour le conteneur de compte de stockage. [Découvrez comment obtenir l’URI SAS du conteneur d’objets blob](../vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container).

### <a name="additional-configuration-requirements"></a>Exigences de configuration supplémentaires

Il existe quelques exigences supplémentaires à prendre en compte, mais elles diffèrent selon que vous effectuez une migration en ligne ou hors connexion.

#### <a name="offline-migrations"></a>Migrations hors connexion

Pour les migrations hors connexion uniquement, effectuez les tâches de configuration supplémentaires suivantes.

* **Sélectionnez des connexions**. Dressez une liste des connexions à migrer comme indiqué dans l’exemple suivant :

    ```powershell
    $selectedLogins = @("user1", "user2")
    ```

    > [!IMPORTANT]
    > Actuellement, Azure Database Migration Service prend uniquement en charge la migration des connexions SQL.

* **Sélectionnez des travaux d’agents**. Créez la liste des travaux d’agents à migrer comme indiqué dans l’exemple suivant :

    ```powershell
    $selectedAgentJobs = @("agentJob1", "agentJob2")
    ```

    > [!IMPORTANT]
    > Azure Database Migration Service prend actuellement en charge les travaux avec les étapes de travail du sous-système T-SQL.

#### <a name="online-migrations"></a>Migrations en ligne

Pour les migrations en ligne uniquement, effectuez les tâches de configuration supplémentaires suivantes.

* **Configurer l’application Azure Active Directory**

    ```powershell
    $pwd = "Azure App Key"
    $appId = "Azure App Id"
    $AppPasswd = ConvertTo-SecureString $pwd -AsPlainText -Force
    $app = New-AzDmsAdApp -ApplicationId $appId -AppKey $AppPasswd
    ```

* **Configurer la ressource de stockage**

    ```powershell
    $storageResourceId = "Storage Resource Id"
    ```

### <a name="create-and-start-the-migration-task"></a>Créer et démarrer la tâche de migration

Utilisez la cmdlet `New-AzDataMigrationTask` pour créer et démarrer une tâche de migration.

#### <a name="specify-parameters"></a>Spécifier des paramètres

##### <a name="common-parameters"></a>Paramètres communs

Si vous effectuez une migration hors connexion ou en ligne, la cmdlet `New-AzDataMigrationTask` attend les paramètres suivants :

* *TaskType*. Type de tâche de migration à créer pour une migration de SQL Server vers Azure SQL Managed Instance. Le type *MigrateSqlServerSqlDbMi* est attendu. 
* *Resource Group Name*. Nom du groupe de ressources Azure dans lequel créer la tâche.
* *ServiceName*. Instance Azure Database Migration Service dans laquelle créer la tâche.
* *ProjectName*. Nom du projet Azure Database Migration Service dans lequel créer la tâche. 
* *TaskName*. Nom de la tâche à créer. 
* *SourceConnection*. Objet AzDmsConnInfo représentant la connexion SQL Server source.
* *TargetConnection*. Objet AzDmsConnInfo représentant la connexion Azure SQL Managed Instance cible.
* *SourceCred*. Objet [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0) pour la connexion au serveur source.
* *TargetCred*. Objet [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0) pour la connexion au serveur cible.
* *SelectedDatabase*. Objet AzDataMigrationSelectedDB représentant le mappage entre les bases de données source et cible.
* *BackupFileShare*. Objet FileShare représentant le partage réseau local qu’Azure Database Migration Service peut utiliser pour les sauvegardes de bases de données source.
* *BackupBlobSasUri*. L’URI SAP qui donne à Azure Database Migration Service l’accès au conteneur du compte de stockage dans lequel le service charge les fichiers de sauvegarde. Découvrez comment obtenir l’URI SAP du conteneur d’objets blob.
* *SelectedLogins*. Liste des connexions sélectionnées pour la migration.
* *SelectedAgentJobs*. Liste des travaux d’agents sélectionnés pour la migration.

##### <a name="additional-parameters"></a>Paramètres supplémentaires

La cmdlet `New-AzDataMigrationTask` attend également des paramètres uniques au type de migration (hors connexion ou en ligne) que vous effectuez.

* **Migrations hors connexion**. Pour les migrations hors connexion, la cmdlet `New-AzDataMigrationTask` attend également les paramètres suivants :

  * *SelectedLogins*. Liste des connexions sélectionnées pour la migration.
  * *SelectedAgentJobs*. Liste des travaux d’agents sélectionnés pour la migration.

* **Migrations en ligne**. Pour les migrations en ligne, la cmdlet `New-AzDataMigrationTask` attend également les paramètres suivants.

* *AzureActiveDirectoryApp*. Application Active Directory.
* *StorageResourceID*. ID de ressource du compte de stockage.

#### <a name="create-and-start-an-offline-migration-task"></a>Créer et démarrer une tâche de migration hors connexion

L’exemple suivant crée et démarre une tâche de migration hors connexion nommée **myDMSTask** :

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMi `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -BackupFileShare $backupFileShare `
  -BackupBlobSasUri $blobSasUri `
  -SelectedLogins $selectedLogins `
  -SelectedAgentJobs $selectedJobs `
```

#### <a name="create-and-start-an-online-migration-task"></a>Créer et démarrer une tâche de migration en ligne

L’exemple suivant crée et démarre une tâche de migration en ligne nommée **myDMSTask** :

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMiSync `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -BackupFileShare $backupFileShare `
  -SelectedDatabase $selectedDbs `
  -AzureActiveDirectoryApp $app `
  -StorageResourceId $storageResourceId
```

## <a name="monitor-the-migration"></a>Surveiller la migration

Pour surveiller la migration, effectuez les tâches suivantes.

1. Consolidez tous les détails de migration dans une variable appelée $CheckTask.

    Pour combiner les détails de migration, tels que les propriétés, l’état et les informations de la base de données associées à la migration, utilisez l’extrait de code suivant :

    ```powershell
    $CheckTask= Get-AzDataMigrationTask     -ResourceGroupName myResourceGroup `
                                            -ServiceName $service.Name `
                                        -ProjectName $project.Name `
                                            -Name myDMSTask `
                                            -ResultType DatabaseLevelOutput `
                        -Expand 
    Write-Host ‘$CheckTask.ProjectTask.Properties.Output’
    ```

2. Utilisez la variable `$CheckTask` pour obtenir l’état actuel de la tâche de migration.

    Pour utiliser la variable `$CheckTask` pour obtenir l’état actuel de la tâche de migration, vous pouvez surveiller la tâche de migration en cours d’exécution en interrogeant la propriété d’état de la tâche, comme indiqué dans l’exemple suivant :

    ```powershell
    if (($CheckTask.ProjectTask.Properties.State -eq "Running") -or ($CheckTask.ProjectTask.Properties.State -eq "Queued"))
    {
      Write-Host "migration task running"
    }
    else if($CheckTask.ProjectTask.Properties.State -eq "Succeeded")
    { 
      Write-Host "Migration task is completed Successfully"
    }
    else if($CheckTask.ProjectTask.Properties.State -eq "Failed" -or $CheckTask.ProjectTask.Properties.State -eq "FailedInputValidation" -or $CheckTask.ProjectTask.Properties.State -eq "Faulted")
    { 
      Write-Host "Migration Task Failed"
    }
    ```

## <a name="performing-the-cutover-online-migrations-only"></a>Exécution du basculement (migrations en ligne uniquement)

Avec une migration en ligne, une sauvegarde et une restauration complètes des bases de données sont réalisées, et puis la tâche poursuit la restauration des journaux des transactions stockés dans le BackupFileShare.

Lorsque la base de données d’une instance Azure SQL Managed Instance est mise à jour avec les données les plus récentes et synchronisée avec la base de données source, vous pouvez effectuer une migration à basculement.

L’exemple suivant termine le basculement\la migration. Les utilisateurs appeler cette commande à leur convenance.

```powershell
$command = Invoke-AzDmsCommand -CommandType CompleteSqlMiSync `
                               -ResourceGroupName myResourceGroup `
                               -ServiceName $service.Name `
                               -ProjectName $project.Name `
                               -TaskName myDMSTask `
                               -DatabaseName "Source DB Name"
```

## <a name="deleting-the-instance-of-azure-database-migration-service"></a>Supprimer l’instance Azure Database Migration Service

Au terme de la migration, vous pourrez supprimer l’instance Azure Database Migration Service :

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations sur les scénarios de migration supplémentaires (paires source/cible), consultez le [Guide de migration de base de données](https://datamigration.microsoft.com/) Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).