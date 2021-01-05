---
title: 'PowerShell : Migrer SQL Server vers SQL Database'
titleSuffix: Azure Database Migration Service
description: Apprenez à migrer une base de données de SQL Server vers Azure SQL Database à l’aide d’Azure PowerShell et Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: 87505557653e70aab7f1392aeea8dbdf505327e0
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962754"
---
# <a name="migrate-a-sql-server-database-to-azure-sql-database-using-azure-powershell"></a>Migrer une base de données SQL Server vers Azure SQL Database à l’aide d’Azure PowerShell

Dans cet article, vous allez migrer la base de données **Adventureworks2012** restaurée sur une instance locale de SQL Server 2016 ou ultérieure ou sur Azure SQL Database à l’aide de Microsoft Azure PowerShell. Vous pouvez migrer des bases de données d’une instance SQL Server vers Azure SQL Database à l’aide du module `Az.DataMigration` dans Microsoft Azure PowerShell.

Dans cet article, vous apprendrez comment :
> [!div class="checklist"]
>
> * Créez un groupe de ressources.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration dans une instance Azure Database Migration Service.
> * Exécuter la migration.

## <a name="prerequisites"></a>Prérequis

Pour effectuer cette procédure, vous avez besoin de :

* [SQL Server 2016 ou version ultérieure](https://www.microsoft.com/sql-server/sql-server-downloads) (toute édition)
* Activer le protocole TCP/IP, qui est désactivé par défaut avec l’installation de SQL Server Express. Activer le protocole TCP/IP en suivant l’article [Activer ou désactiver un protocole réseau de serveur](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
* Pour configurer votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Une instance Azure SQL Database. Vous pouvez créer une instance Azure SQL Database en suivant les indications de l’article [Création d’une base de données dans Azure SQL Database à l’aide du portail Azure](../azure-sql/database/single-database-create-quickstart.md).
* [Assistant Migration des données](https://www.microsoft.com/download/details.aspx?id=53595) version 3.3 ou ultérieure.
* Un réseau virtuel Microsoft Azure créé à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité de site à site à vos serveurs sources locaux pour le service Azure Database Migration Service via [ExpressRoute](../expressroute/expressroute-introduction.md) ou une passerelle [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Avoir terminé l’évaluation de votre base de données locale et de la migration de schéma à l’aide de l’Assistant Migration de données, comme indiqué dans l’article [Évaluer votre migration vers SQL Server](/sql/dma/dma-assesssqlonprem).
* Télécharger et installer le module Az.DataMigration à partir de PowerShell Gallery en utilisant la [cmdlet PowerShell Install-Module](/powershell/module/powershellget/Install-Module?view=powershell-5.1). Veillez à ouvrir la fenêtre de commande PowerShell avec l’option Exécuter en tant qu’administrateur.
* Vous assurer que les informations d’identification utilisées pour se connecter à une instance SQL Server source disposent de l’autorisation [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Vous assurer que les informations d’identification utilisées pour se connecter à une instance Azure SQL DB cible disposent de l’autorisation CONTROL DATABASE sur les bases de données Azure SQL Database cibles.
* Un abonnement Azure. Si vous n’en avez pas, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="log-in-to-your-microsoft-azure-subscription"></a>Connexion à un abonnement Microsoft Azure

Suivez les instructions de l’article [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps) pour vous connecter à votre abonnement Azure à l’aide de PowerShell.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Créez un groupe de ressources pour pouvoir créer une machine virtuelle.

Créez un groupe de ressources à l’aide de la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

L’exemple suivant permet de créer un groupe de ressources nommé *myResourceGroup* dans la région *Est des États-Unis*.

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>Créer une instance Azure Database Migration Service

Vous pouvez créer une instance Azure Database Migration Service à l’aide de l’applet de commande `New-AzDataMigrationService`. Cette cmdlet attend les paramètres requis suivants :

* *Nom du groupe de ressources Azure*. Vous pouvez utiliser la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) pour créer le groupe de ressources Azure comme indiqué précédemment, et fournir son nom en tant que paramètre.
* *Nom du service*. Chaîne qui correspond au nom de service unique à donner à l’instance Azure Database Migration Service. 
* *Emplacement*. Spécifie l’emplacement du service. Indiquez un emplacement de centre de données Azure, tel que USA Ouest ou Asie Sud-Est.
* *Référence SKU*. Ce paramètre correspond au nom de référence SKU DMS. Le nom de référence SKU actuellement pris en charge est *GeneralPurpose_4vCores*.
* *Identificateur de sous-réseau virtuel*. Vous pouvez utiliser la cmdlet [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) pour créez un sous-réseau. 

L’exemple suivant crée un service nommé *MyDMS* dans le groupe de ressources *MyDMSResourceGroup*, qui se trouve dans la région *USA Est* à l’aide d’un réseau virtuel appelé *MyVNET* et d’un sous-réseau appelé *MySubnet*.

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

Vous pouvez créer un objet d’informations de connexion de base de données à l’aide de la cmdlet `New-AzDmsConnInfo`. Cette cmdlet attend les paramètres suivants :

* *ServerType*. Type de connexion de base de données demandé, par exemple, MySQL, Oracle ou MySQL. Utilisez SQL pour SQL Server et Azure SQL.
* *DataSource*. Nom ou adresse IP d’une instance SQL Server ou d’Azure SQL Database.
* *AuthType*. Type d’authentification pour la connexion, qui peut être SqlAuthentication ou WindowsAuthentication.
* Le paramètre *TrustServerCertificate* définit une valeur qui indique si le canal est chiffré tout en ignorant l’analyse de la chaîne de certificat pour valider l’approbation. La valeur doit être « true » ou « false ».

L’exemple suivant crée l’objet d’informations de connexion pour l’instance SQL Server source, appelée MySourceSQLServer avec l’authentification SQL :

```powershell
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

L’exemple suivant montre la création des informations de connexion pour un serveur appelé SQLAzureTarget à l’aide de l’authentification SQL :

```powershell
$targetConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource "sqlazuretarget.database.windows.net" `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$false
```

### <a name="provide-databases-for-the-migration-project"></a>Fournir des bases de données pour le projet de migration

Créez une liste d’objets `AzDataMigrationDatabaseInfo` spécifiant les bases de données dans le cadre du projet Azure Database Migration qui peut être fourni en tant que paramètre de création du projet. La cmdlet `New-AzDataMigrationDatabaseInfo` peut être utilisée pour créer AzDataMigrationDatabaseInfo. 

L’exemple suivant crée le projet `AzDataMigrationDatabaseInfo` pour la base de données **AdventureWorks2016** et l’ajoute à la liste qui doit être fournie en tant que paramètre de création du projet.

```powershell
$dbInfo1 = New-AzDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks2016
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>Créer un objet de projet

Enfin, vous pouvez créer le projet Azure Database Migration appelé *MyDMSProject*, situé dans la région *USA Est*, à l’aide du paramètre `New-AzDataMigrationProject`, et en ajoutant les connexions source et cible créées précédemment et la liste des bases de données à migrer.

```powershell
$project = New-AzDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLDB `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>Créer et démarrer une tâche de migration

Enfin, créez et démarrez une tâche Azure Database Migration. La tâche Azure Database Migration nécessite des informations de connexion pour les systèmes source et cible, ainsi que la liste des tables de base de données à migrer, en plus des informations déjà fournies avec le projet créé, en tant que condition préalable.

### <a name="create-credential-parameters-for-source-and-target"></a>Créer des paramètres d’informations d’identification pour la source et la cible

Vous pouvez créer des informations d’identification de sécurité de la connexion en tant qu’objet [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0).

L’exemple suivant illustre la création d’objets *PSCredential* pour les connexions source et cible, en fournissant des mots de passe en tant que variables de chaîne *$sourcePassword* et *$targetPassword*.

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-a-table-map-and-select-source-and-target-parameters-for-migration"></a>Créer un mappage de table et sélectionner des paramètres source et cible pour la migration

Le mappage des tables depuis des tables source et cible à migrer est un autre paramètre nécessaire à la migration. Créez un dictionnaire des tables qui assure le mappage entre les tables source et cible pour la migration. L’exemple suivant illustre le mappage entre les tables source et cible du schéma des ressources humaines pour la base de données AdventureWorks 2016.

```powershell
$tableMap = New-Object 'system.collections.generic.dictionary[string,string]'
$tableMap.Add("HumanResources.Department", "HumanResources.Department")
$tableMap.Add("HumanResources.Employee","HumanResources.Employee")
$tableMap.Add("HumanResources.EmployeeDepartmentHistory","HumanResources.EmployeeDepartmentHistory")
$tableMap.Add("HumanResources.EmployeePayHistory","HumanResources.EmployeePayHistory")
$tableMap.Add("HumanResources.JobCandidate","HumanResources.JobCandidate")
$tableMap.Add("HumanResources.Shift","HumanResources.Shift")
```

L’étape suivante consiste à sélectionner les bases de données source et cible et à fournir un mappage de table pour effectuer la migration en tant que paramètre, à l’aide de la cmdlet `New-AzDmsSelectedDB`, comme indiqué dans l’exemple suivant :

```powershell
$selectedDbs = New-AzDmsSelectedDB -MigrateSqlServerSqlDb -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -TableMap $tableMap
```

### <a name="create-the-migration-task-and-start-it"></a>Créer et démarrer la tâche de migration

Utilisez la cmdlet `New-AzDataMigrationTask` pour créer et démarrer une tâche de migration. Cette cmdlet attend les paramètres suivants :

* *TaskType*. Type de tâche de migration à créer pour une migration de SQL Server vers Azure SQL Database. Le type *MigrateSqlServerSqlDb* est attendu. 
* *Resource Group Name*. Nom du groupe de ressources Azure dans lequel créer la tâche.
* *ServiceName*. Instance Azure Database Migration Service dans laquelle créer la tâche.
* *ProjectName*. Nom du projet Azure Database Migration Service dans lequel créer la tâche. 
* *TaskName*. Nom de la tâche à créer. 
* *SourceConnection*. Objet AzDmsConnInfo représentant la connexion SQL Server source.
* *TargetConnection*. Objet AzureRmDmsConnInfo représentant la connexion Azure SQL Database cible.
* *SourceCred*. Objet [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0) pour la connexion au serveur source.
* *TargetCred*. Objet [PSCredential](/dotnet/api/system.management.automation.pscredential?view=powershellsdk-1.1.0) pour la connexion au serveur cible.
* *SelectedDatabase*. Objet AzDataMigrationSelectedDB représentant le mappage entre les bases de données source et cible.
* *SchemaValidation*. (facultatif, paramètre de commutateur) Effectue une comparaison des informations de schéma de la source et de la cible après la migration.
* *DataIntegrityValidation*. (facultatif, paramètre de commutateur) Effectue une validation de l’intégrité des données basée sur une somme de contrôle entre la source et la cible après la migration.
* *QueryAnalysisValidation*. (facultatif, paramètre de commutateur) Effectue une analyse rapide et intelligente des requêtes en extrayant des requêtes de la base de données source et les exécute dans la cible après la migration.

L’exemple suivant crée et démarre une tâche de migration nommée myDMSTask :

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDb `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
```

L’exemple suivant crée et démarre la même tâche de migration que ci-dessus, mais effectue également les trois validations suivantes :

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDb `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -SchemaValidation `
  -DataIntegrityValidation `
  -QueryAnalysisValidation `
```

## <a name="monitor-the-migration"></a>Surveiller la migration

Vous pouvez surveiller la tâche de migration en cours d’exécution en interrogeant la propriété d’état de la tâche, comme indiqué dans l’exemple suivant :

```powershell
if (($mytask.ProjectTask.Properties.State -eq "Running") -or ($mytask.ProjectTask.Properties.State -eq "Queued"))
{
  write-host "migration task running"
}
```

## <a name="deleting-the-dms-instance"></a>Suppression de l'instance de DMS

Au terme de la migration, vous pourrez supprimer l'instance d'Azure DMS :

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="next-step"></a>Étape suivante

* Lisez l’aide à la migration du [Guide de migration de bases de données](https://datamigration.microsoft.com/) de Microsoft.