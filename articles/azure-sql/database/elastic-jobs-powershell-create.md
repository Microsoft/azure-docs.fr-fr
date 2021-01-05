---
title: Créer un agent de travail élastique à l’aide de PowerShell (préversion)
description: Découvrez comment créer un agent de travail élastique à l’aide de PowerShell.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.devlang: ''
ms.topic: tutorial
author: johnpaulkee
ms.author: joke
ms.reviwer: sstein
ms.date: 10/21/2020
ms.openlocfilehash: 27cd35eba7320022ea9b137a7b8bb079a1226751
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92427293"
---
# <a name="create-an-elastic-job-agent-using-powershell-preview"></a>Créer un agent de travail élastique à l’aide de PowerShell (préversion)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Les [travaux élastiques (préversion)](job-automation-overview.md#elastic-database-jobs-preview) permettent l’exécution d’un ou plusieurs scripts Transact-SQL (T-SQL) en parallèle sur plusieurs bases de données.

Dans ce tutoriel, vous découvrez les étapes requises pour exécuter une requête sur plusieurs bases de données :

> [!div class="checklist"]
> * Créer un agent de travail élastique
> * Créer des informations d’identification de travail afin que les travaux puissent exécuter des scripts sur ses cibles
> * Définir les cibles (serveurs, pools élastiques, bases de données, cartes de partitions) sur lesquelles vous voulez exécuter le travail
> * Créer des informations d’identification limitées à une base de données dans les bases de données cibles afin que l’agent se connecte et exécute des travaux
> * Créer un travail
> * Ajouter des étapes de travail à un travail
> * Démarrer l’exécution d’un travail
> * Surveiller un travail

## <a name="prerequisites"></a>Prérequis

La version mise à niveau des tâches de base de données élastique comprend un nouvel ensemble d’applets de commande PowerShell à utiliser lors de la migration. Ces nouvelles applets de commande transfèrent l’ensemble des informations d’identification de tâche, des cibles (dont les bases de données, serveurs, collections personnalisées), des déclencheurs de tâche, des planifications de tâches, du contenu de la tâche ainsi que des tâches vers un nouvel agent de travail élastique.

### <a name="install-the-latest-elastic-jobs-cmdlets"></a>Installer les dernières applets de commande de tâches élastiques

Si vous n’avez pas encore d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

Installez le module **Az.Sql** pour obtenir les dernières applets de commande de travail élastique. Exécutez les commandes suivantes dans PowerShell avec un accès administrateur.

```powershell
# installs the latest PackageManagement and PowerShellGet packages
Find-Package PackageManagement | Install-Package -Force
Find-Package PowerShellGet | Install-Package -Force

# Restart your powershell session with administrative access

# Install and import the Az.Sql module, then confirm
Install-Module -Name Az.Sql
Import-Module Az.Sql

Get-Module Az.Sql
```

En plus du module **Az.Sql** , ce tutoriel nécessite le module PowerShell *SqlServer* . Pour plus d’informations, consultez [Installer le module SQL Server PowerShell](/sql/powershell/download-sql-server-ps-module).

## <a name="create-required-resources"></a>Créer les ressources nécessaires

La création d’un agent de travail élastique requiert une base de données (S0 ou une version ultérieure) pour une utilisation en tant que [Base de données des travaux](job-automation-overview.md#job-database).

Le script ci-dessous crée un groupe de ressources, un serveur et une base de données à utiliser comme base de données de travaux. Le deuxième script crée un deuxième serveur avec deux bases de données vides sur lesquelles exécuter les travaux.

Travaux élastiques n’ont aucune exigence d’affectation de noms, c’est la raison pour laquelle vous pouvez utiliser n’importe quelle convention d’affectation de noms, tant qu’elle est conforme aux [conditions requises pour Azure](/azure/architecture/best-practices/resource-naming).

```powershell
# sign in to Azure account
Connect-AzAccount

# create a resource group
Write-Output "Creating a resource group..."
$resourceGroupName = Read-Host "Please enter a resource group name"
$location = Read-Host "Please enter an Azure Region"
$rg = New-AzResourceGroup -Name $resourceGroupName -Location $location
$rg

# create a server
Write-Output "Creating a server..."
$agentServerName = Read-Host "Please enter an agent server name"
$agentServerName = $agentServerName + "-" + [guid]::NewGuid()
$adminLogin = Read-Host "Please enter the server admin name"
$adminPassword = Read-Host "Please enter the server admin password"
$adminPasswordSecure = ConvertTo-SecureString -String $AdminPassword -AsPlainText -Force
$adminCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $adminLogin, $adminPasswordSecure
$agentServer = New-AzSqlServer -ResourceGroupName $resourceGroupName -Location $location `
    -ServerName $agentServerName -ServerVersion "12.0" -SqlAdministratorCredentials ($adminCred)

# set server firewall rules to allow all Azure IPs
Write-Output "Creating a server firewall rule..."
$agentServer | New-AzSqlServerFirewallRule -AllowAllAzureIPs
$agentServer

# create the job database
Write-Output "Creating a blank database to be used as the Job Database..."
$jobDatabaseName = "JobDatabase"
$jobDatabase = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $agentServerName -DatabaseName $jobDatabaseName -RequestedServiceObjectiveName "S0"
$jobDatabase
```

```powershell
# create a target server and sample databases - uses the same credentials
Write-Output "Creating target server..."
$targetServerName = Read-Host "Please enter a target server name"
$targetServerName = $targetServerName + "-" + [guid]::NewGuid()
$targetServer = New-AzSqlServer -ResourceGroupName $resourceGroupName -Location $location `
    -ServerName $targetServerName -ServerVersion "12.0" -SqlAdministratorCredentials ($adminCred)

# set target server firewall rules to allow all Azure IPs
$targetServer | New-AzSqlServerFirewallRule -AllowAllAzureIPs
$targetServer | New-AzSqlServerFirewallRule -StartIpAddress 0.0.0.0 -EndIpAddress 255.255.255.255 -FirewallRuleName AllowAll
$targetServer

# create sample databases to execute jobs against
$db1 = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $targetServerName -DatabaseName "database1"
$db1
$db2 = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $targetServerName -DatabaseName "database2"
$db2
```

## <a name="use-elastic-jobs"></a>Utiliser des travaux élastiques

Pour utiliser les travaux élastiques, inscrivez la fonctionnalité dans votre abonnement Azure en exécutant la commande suivante. Exécutez cette commande une seule fois pour l’abonnement dans lequel vous souhaitez provisionner l’agent de travail élastique. Il n’est pas nécessaire d’inscrire les abonnements qui contiennent uniquement des bases de données et qui sont des cibles de travail.

```powershell
Register-AzProviderFeature -FeatureName sqldb-JobAccounts -ProviderNamespace Microsoft.Sql
```

### <a name="create-the-elastic-job-agent"></a>Créer l’agent de travail élastique

Un agent de travail élastique est une ressource Azure permettant de créer, exécuter et gérer des travaux. L’agent exécute les travaux selon un calendrier ou de manière ponctuelle.

Pour l’applet de commande **New-AzSqlElasticJobAgent** , une base de données dans Azure SQL Database doit déjà exister pour que les paramètres *ResourceGroupName* , *ServerName* et *DatabaseName* pointent tous vers des ressources existantes.

```powershell
Write-Output "Creating job agent..."
$agentName = Read-Host "Please enter a name for your new Elastic Job agent"
$jobAgent = $jobDatabase | New-AzSqlElasticJobAgent -Name $agentName
$jobAgent
```

### <a name="create-the-job-credentials"></a>Créer les informations d’identification de travail

Les travaux utilisent des informations d’identification de base de données pour se connecter aux bases de données cibles spécifiées par le groupe cible au moment de l’exécution et pour exécuter les scripts. Ces informations d’identification sont également utilisées pour se connecter à la base de données MASTER afin d’énumérer toutes les bases de données d’un serveur ou d’un pool élastique, lorsque l’une d’entre elles est utilisée en tant que type de membre du groupe cible.

Ces informations d’identification doivent être créées dans la base de données des travaux. Toutes les bases de données cibles doivent avoir une connexion disposant d’autorisations suffisantes pour que le travail s’exécute avec succès.

![Informations d’identification des travaux élastiques](./media/elastic-jobs-powershell-create/job-credentials.png)

Outre les informations d’identification de l’image, notez l’ajout des commandes **GRANT** dans le script suivant. Ces autorisations sont requises pour le script que nous avons choisi pour cet exemple de travail. Dans la mesure où l’exemple crée une table dans les bases de données ciblées, chaque base de données cible doit avoir les autorisations appropriées pour pouvoir s’exécuter.

Pour créer les informations d’identification de travail requises (dans la base de données des travaux), exécutez le script suivant :

```powershell
# in the master database (target server)
# create the master user login, master user, and job user login
$params = @{
  'database' = 'master'
  'serverInstance' =  $targetServer.ServerName + '.database.windows.net'
  'username' = $adminLogin
  'password' = $adminPassword
  'outputSqlErrors' = $true
  'query' = 'CREATE LOGIN masteruser WITH PASSWORD=''password!123'''
}
Invoke-SqlCmd @params
$params.query = "CREATE USER masteruser FROM LOGIN masteruser"
Invoke-SqlCmd @params
$params.query = 'CREATE LOGIN jobuser WITH PASSWORD=''password!123'''
Invoke-SqlCmd @params

# for each target database
# create the jobuser from jobuser login and check permission for script execution
$targetDatabases = @( $db1.DatabaseName, $Db2.DatabaseName )
$createJobUserScript =  "CREATE USER jobuser FROM LOGIN jobuser"
$grantAlterSchemaScript = "GRANT ALTER ON SCHEMA::dbo TO jobuser"
$grantCreateScript = "GRANT CREATE TABLE TO jobuser"

$targetDatabases | % {
  $params.database = $_
  $params.query = $createJobUserScript
  Invoke-SqlCmd @params
  $params.query = $grantAlterSchemaScript
  Invoke-SqlCmd @params
  $params.query = $grantCreateScript
  Invoke-SqlCmd @params
}

# create job credential in Job database for master user
Write-Output "Creating job credentials..."
$loginPasswordSecure = (ConvertTo-SecureString -String 'password!123' -AsPlainText -Force)

$masterCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "masteruser", $loginPasswordSecure
$masterCred = $jobAgent | New-AzSqlElasticJobCredential -Name "masteruser" -Credential $masterCred

$jobCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "jobuser", $loginPasswordSecure
$jobCred = $jobAgent | New-AzSqlElasticJobCredential -Name "jobuser" -Credential $jobCred
```

### <a name="define-the-target-databases-to-run-the-job-against"></a>Définir les bases de données cibles sur lesquelles exécuter le travail

Un [groupe cible](job-automation-overview.md#target-group) définit l’ensemble de base de données sur lequel une étape de travail s’exécute.

L’extrait de code suivant crée deux groupes cibles : *serverGroup* et *serverGroupExcludingDb2* . *serverGroup* cible toutes les bases de données existant sur le serveur au moment de l’exécution et *serverGroupExcludingDb2* toutes les bases de données sur le serveur, à l’exception de *targetDb2*  :

```powershell
Write-Output "Creating test target groups..."
# create ServerGroup target group
$serverGroup = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'ServerGroup'
$serverGroup | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName

# create ServerGroup with an exclusion of db2
$serverGroupExcludingDb2 = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'ServerGroupExcludingDb2'
$serverGroupExcludingDb2 | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName
$serverGroupExcludingDb2 | Add-AzSqlElasticJobTarget -ServerName $targetServerName -Database $db2.DatabaseName -Exclude
```

### <a name="create-a-job-and-steps"></a>Créer un travail et une tâche

Cet exemple définit un travail et deux étapes de travail pour le travail à exécuter. La première étape de travail ( *step1* ) crée une nouvelle table ( *Step1Table* ) dans chaque base de données du groupe cible *ServerGroup* . La deuxième étape de travail ( *step2* ) crée une table ( *Step2Table* ) dans chaque base de données à l’exception de *TargetDb2* , car le groupe cible défini précédemment indiquait de l’exclure.

```powershell
Write-Output "Creating a new job..."
$jobName = "Job1"
$job = $jobAgent | New-AzSqlElasticJob -Name $jobName -RunOnce
$job

Write-Output "Creating job steps..."
$sqlText1 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('Step1Table')) CREATE TABLE [dbo].[Step1Table]([TestId] [int] NOT NULL);"
$sqlText2 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('Step2Table')) CREATE TABLE [dbo].[Step2Table]([TestId] [int] NOT NULL);"

$job | Add-AzSqlElasticJobStep -Name "step1" -TargetGroupName $serverGroup.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText1
$job | Add-AzSqlElasticJobStep -Name "step2" -TargetGroupName $serverGroupExcludingDb2.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText2
```

### <a name="run-the-job"></a>Exécuter le travail

Pour démarrer le travail immédiatement, utilisez la commande suivante :

```powershell
Write-Output "Start a new execution of the job..."
$jobExecution = $job | Start-AzSqlElasticJob
$jobExecution
```

Une fois le travail réussi, vous devez voir deux nouvelles tables dans TargetDb1 et seulement une nouvelle table dans TargetDb2 :

   ![vérification des nouvelles tables dans SSMS](./media/elastic-jobs-powershell-create/job-execution-verification.png)

Vous pouvez aussi planifier le travail pour une exécution ultérieure. Pour planifier l’exécution ultérieure d’un travail, exécutez la commande suivante :

```powershell
# run every hour starting from now
$job | Set-AzSqlElasticJob -IntervalType Hour -IntervalCount 1 -StartTime (Get-Date) -Enable
```

### <a name="monitor-status-of-job-executions"></a>Surveiller l’état d’exécution des travaux

L’extrait de code suivant obtient les informations relatives au travail d’exécution :

```powershell
# get the latest 10 executions run
$jobAgent | Get-AzSqlElasticJobExecution -Count 10

# get the job step execution details
$jobExecution | Get-AzSqlElasticJobStepExecution

# get the job target execution details
$jobExecution | Get-AzSqlElasticJobTargetExecution -Count 2
```

Le tableau suivant répertorie les états d’exécution de travail possibles :

|State|Description|
|:---|:---|
|**Créé le** | L’exécution du travail vient d’être créée et n’est pas encore en cours d’exécution.|
|**InProgress** | L’exécution du travail est en cours.|
|**WaitingForRetry** | L’exécution du travail n’a pas été en mesure de terminer son action et est en attente de nouvelle tentative.|
|**Réussi** | L’exécution du travail a réussi.|
|**SucceededWithSkipped** | L’exécution de la tâche s’est terminée avec succès, mais certains de ses enfants ont été ignorés.|
|**Échec** | L’exécution du travail a échoué et a épuisé ses nouvelles tentatives.|
|**TimedOut** | L’exécution du travail a expiré.|
|**Canceled** | L’exécution du travail a été annulée.|
|**Ignoré** | L’exécution du travail a été ignorée, car une autre exécution de la même étape du travail était déjà en cours d’exécution sur la même cible.|
|**WaitingForChildJobExecutions** | L’exécution du travail attend que les exécutions de ses enfants se termine.|

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez les ressources Azure créées dans ce tutoriel en supprimant le groupe de ressources.

> [!TIP]
> Si vous envisagez de continuer à utiliser ces travaux, ne nettoyez pas les ressources créées dans cet article.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez exécuté un script Transact-SQL sur un ensemble de bases de données. Vous avez appris à effectuer les tâches suivantes :

> [!div class="checklist"]
> * Créer un agent de travail élastique
> * Créer des informations d’identification de travail afin que les travaux puissent exécuter des scripts sur ses cibles
> * Définir les cibles (serveurs, pools élastiques, bases de données, cartes de partitions) sur lesquelles vous voulez exécuter le travail
> * Créer des informations d’identification limitées à une base de données dans les bases de données cibles afin que l’agent se connecte et exécute des travaux
> * Créer un travail
> * Ajout d’une étape au travail
> * Démarrage d’une exécution du travail
> * Surveiller la tâche

> [!div class="nextstepaction"]
> [Gérer des travaux élastiques à l’aide de Transact-SQL](elastic-jobs-tsql-create-manage.md)
