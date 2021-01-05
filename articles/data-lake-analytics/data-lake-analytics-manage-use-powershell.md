---
title: Gestion d'Azure Data Lake Analytics à l'aide d'Azure PowerShell
description: Cet article décrit comment utiliser Azure PowerShell pour gérer des comptes Azure Data Lake Analytics, des sources de données, des utilisateurs et des travaux.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/29/2018
ms.openlocfilehash: afa21e6aae769e69e8bc83b9fa0d4f9b76396f7e
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220310"
---
# <a name="manage-azure-data-lake-analytics-using-azure-powershell"></a>Gestion d'Azure Data Lake Analytics à l'aide d'Azure PowerShell

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Cet article décrit comment gérer des comptes Azure Data Lake Analytics, des sources de données, des utilisateurs et des travaux à l’aide d’Azure PowerShell.

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour utiliser PowerShell avec Data Lake Analytics, collectez les informations suivantes :

* **ID d’abonnement** : ID de l’abonnement Azure qui contient votre compte Data Lake Analytics.
* **Groupe de ressources** : nom du groupe de ressources Azure qui contient votre compte Data Lake Analytics.
* **Nom du compte Data Lake Analytics** : nom de votre compte Data Lake Analytics.
* **Nom du compte Data Lake Store par défaut**  : chaque compte Data Lake Analytics possède un compte Data Lake Store par défaut.
* **Emplacement** : emplacement de votre compte Data Lake Analytics, comme « USA Est 2 » ou autres emplacements pris en charge.

Dans ce didacticiel, les extraits de code PowerShell utilisent ces variables pour stocker ces informations.

```powershell
$subId = "<SubscriptionId>"
$rg = "<ResourceGroupName>"
$adla = "<DataLakeAnalyticsAccountName>"
$adls = "<DataLakeStoreAccountName>"
$location = "<Location>"
```

## <a name="log-in-to-azure"></a>Connexion à Azure

### <a name="log-in-using-interactive-user-authentication"></a>Connexion à l’aide de l’authentification utilisateur interactif

Connexion à l’aide d’un ID d’abonnement ou par nom d’abonnement

```powershell
# Using subscription id
Connect-AzAccount -SubscriptionId $subId

# Using subscription name
Connect-AzAccount -SubscriptionName $subname
```

## <a name="saving-authentication-context"></a>Enregistrement du contexte d’authentification

La cmdlet `Connect-AzAccount` demande toujours les informations d’identification. Vous pouvez éviter cela à l’aide des cmdlets suivantes :

```powershell
# Save login session information
Save-AzAccounts -Path D:\profile.json  

# Load login session information
Select-AzAccounts -Path D:\profile.json
```

### <a name="log-in-using-a-service-principal-identity-spi"></a>Connexion à l’aide d’une identité de principal du service (SPI)

```powershell
$tenantid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"  
$spi_appname = "appname"
$spi_appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
$spi_secret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

$pscredential = New-Object System.Management.Automation.PSCredential ($spi_appid, (ConvertTo-SecureString $spi_secret -AsPlainText -Force))
Login-AzAccount -ServicePrincipal -TenantId $tenantid -Credential $pscredential -Subscription $subid
```

## <a name="manage-accounts"></a>Gérer les comptes

### <a name="list-accounts"></a>Énumérer les comptes

```powershell
# List Data Lake Analytics accounts within the current subscription.
Get-AdlAnalyticsAccount

# List Data Lake Analytics accounts within a specific resource group.
Get-AdlAnalyticsAccount -ResourceGroupName $rg
```

### <a name="create-an-account"></a>Créer un compte

Chaque compte Data Lake Analytics nécessite un compte Data Lake Store par défaut pour stocker les journaux d’activité. Vous pouvez réutiliser un compte ou en créer un.

```powershell
# Create a data lake store if needed, or you can re-use an existing one
New-AdlStore -ResourceGroupName $rg -Name $adls -Location $location
New-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla -Location $location -DefaultDataLake $adls
```

### <a name="get-account-information"></a>Obtenir les informations de compte

Obtenez les détails relatifs à un compte.

```powershell
Get-AdlAnalyticsAccount -Name $adla
```

### <a name="check-if-an-account-exists"></a>Vérifier si un compte existe

```powershell
Test-AdlAnalyticsAccount -Name $adla
```

## <a name="manage-data-sources"></a>Gérer les sources de données

Azure Data Lake Analytics prend actuellement en charge les sources de données suivantes :

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Stockage Azure](../storage/common/storage-introduction.md)

Chaque compte Data Lake Analytics possède un compte Data Lake Store par défaut. Le compte Data Lake Store par défaut est utilisé pour stocker les métadonnées du travail et les journaux d’audit du travail.

### <a name="find-the-default-data-lake-store-account"></a>Rechercher le compte Data Lake Store par défaut

```powershell
$adla_acct = Get-AdlAnalyticsAccount -Name $adla
$dataLakeStoreName = $adla_acct.DefaultDataLakeAccount
```

Vous trouverez le compte Data Lake Store par défaut en filtrant la liste des sources de données par le `IsDefault` propriété :

```powershell
Get-AdlAnalyticsDataSource -Account $adla  | ? { $_.IsDefault }
```

### <a name="add-a-data-source"></a>Ajouter une source de données

```powershell

# Add an additional Storage (Blob) account.
$AzureStorageAccountName = "<AzureStorageAccountName>"
$AzureStorageAccountKey = "<AzureStorageAccountKey>"
Add-AdlAnalyticsDataSource -Account $adla -Blob $AzureStorageAccountName -AccessKey $AzureStorageAccountKey

# Add an additional Data Lake Store account.
$AzureDataLakeStoreName = "<AzureDataLakeStoreAccountName"
Add-AdlAnalyticsDataSource -Account $adla -DataLakeStore $AzureDataLakeStoreName
```

### <a name="list-data-sources"></a>Lister les sources de données

```powershell
# List all the data sources
Get-AdlAnalyticsDataSource -Account $adla

# List attached Data Lake Store accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "DataLakeStore"

# List attached Storage accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "Blob"
```

## <a name="submit-u-sql-jobs"></a>Envoyer des travaux U-SQL

### <a name="submit-a-string-as-a-u-sql-job"></a>Envoyer une chaîne en tant que travail U-SQL

```powershell
$script = @"
@a  =
    SELECT * FROM
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"@

$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath

Submit-AdlJob -AccountName $adla -Script $script -Name "Demo"
```

### <a name="submit-a-file-as-a-u-sql-job"></a>Envoyer un fichier en tant que travail U-SQL

```powershell
$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath
Submit-AdlJob -AccountName $adla –ScriptPath $scriptpath -Name "Demo"
```

### <a name="list-jobs"></a>Répertorier les travaux

La sortie comprend les travaux en cours et ceux qui ont été terminés récemment.

```powershell
Get-AdlJob -Account $adla
```

### <a name="list-the-top-n-jobs"></a>Répertorier les N premiers travaux

Par défaut, la liste des travaux est triée en fonction de l’heure d’envoi. Par conséquent, les travaux envoyés le plus récemment s’affichent en premier. Par défaut, le compte ADLA garde en mémoire les travaux pendant 180 jours, mais l’applet de commande AdlJob-Ge ne retourne par défaut que les 500 premiers travaux. Utilisez le paramètre -Top pour lister un nombre spécifique de travaux.

```powershell
$jobs = Get-AdlJob -Account $adla -Top 10
```

### <a name="list-jobs-by-job-state"></a>Répertorier les travaux par état du travail

Utilisation du `-State` paramètre. Vous pouvez combiner chacune de ces valeurs :

* `Accepted`
* `Compiling`
* `Ended`
* `New`
* `Paused`
* `Queued`
* `Running`
* `Scheduling`
* `Start`

```powershell
# List the running jobs
Get-AdlJob -Account $adla -State Running

# List the jobs that have completed
Get-AdlJob -Account $adla -State Ended

# List the jobs that have not started yet
Get-AdlJob -Account $adla -State Accepted,Compiling,New,Paused,Scheduling,Start
```

### <a name="list-jobs-by-job-result"></a>Répertorier les travaux par résultat du travail

Utilisez le `-Result` paramètre pour détecter si les travaux terminés se sont achevés correctement. Il possède ces valeurs :

* Annulé
* Échec
* None
* Opération réussie

``` powershell
# List Successful jobs.
Get-AdlJob -Account $adla -State Ended -Result Succeeded

# List Failed jobs.
Get-AdlJob -Account $adla -State Ended -Result Failed
```

### <a name="list-jobs-by-job-submitter"></a>Répertorier les travaux par expéditeur du travail

Le `-Submitter` paramètre vous permet d’identifier qui a envoyé une tâche.

```powershell
Get-AdlJob -Account $adla -Submitter "joe@contoso.com"
```

### <a name="list-jobs-by-submission-time"></a>Répertorier les travaux par heure d’envoi

Le paramètre `-SubmittedAfter` est utile lorsque vous filtrez dans un intervalle de temps.

```powershell
# List  jobs submitted in the last day.
$d = [DateTime]::Now.AddDays(-1)
Get-AdlJob -Account $adla -SubmittedAfter $d

# List  jobs submitted in the last seven day.
$d = [DateTime]::Now.AddDays(-7)
Get-AdlJob -Account $adla -SubmittedAfter $d
```

### <a name="get-job-status"></a>Obtenir l’état de la tâche

Affichez l’état d’un travail.

```powershell
Get-AdlJob -AccountName $adla -JobId $job.JobId
```

### <a name="cancel-a-job"></a>Annulation d’une tâche

```powershell
Stop-AdlJob -Account $adla -JobID $jobID
```

### <a name="wait-for-a-job-to-finish"></a>Attendre la fin d’une tâche

Au lieu de répéter `Get-AdlAnalyticsJob` jusqu’à ce qu’un travail se termine, vous pouvez utiliser la cmdlet `Wait-AdlJob` pour attendre la fin du travail.

```powershell
Wait-AdlJob -Account $adla -JobId $job.JobId
```

## <a name="analyzing-job-history"></a>Analyse de l’historique des travaux

Utiliser Azure PowerShell pour analyser l’historique des travaux qui ont été exécutés dans Data Lake analytics est une technique puissante. Vous pouvez l’utiliser pour obtenir des informations sur l’utilisation et le coût. Pour en savoir plus, consultez [l’exemple de dépôt de l’analyse de l’historique des travaux](https://github.com/Azure-Samples/data-lake-analytics-powershell-job-history-analysis).  

## <a name="list-job-pipelines-and-recurrences"></a>Répertorier les pipelines et les récurrences des tâches

Utilisez l’ `Get-AdlJobPipeline` applet de commande pour afficher les tâches déjà soumises sur les informations de pipeline.

```powershell
$pipelines = Get-AdlJobPipeline -Account $adla
$pipeline = Get-AdlJobPipeline -Account $adla -PipelineId "<pipeline ID>"
```

Utilisez l’ `Get-AdlJobRecurrence` applet de commande pour afficher les informations sur la périodicité des tâches déjà soumises.

```powershell
$recurrences = Get-AdlJobRecurrence -Account $adla

$recurrence = Get-AdlJobRecurrence -Account $adla -RecurrenceId "<recurrence ID>"
```

## <a name="manage-compute-policies"></a>Gérer les stratégies de calcul

### <a name="list-existing-compute-policies"></a>Lister les stratégies de calcul existantes

L’ `Get-AdlAnalyticsComputePolicy` applet de commande récupère des informations sur les stratégies de calcul pour un compte Data Lake Analytics.

```powershell
$policies = Get-AdlAnalyticsComputePolicy -Account $adla
```

### <a name="create-a-compute-policy"></a>Créer une stratégie de calcul

L’ `New-AdlAnalyticsComputePolicy` applet de commande crée une nouvelle stratégie de calcul pour un compte Data Lake Analytics. Cet exemple définit les AU maximales disponibles pour l’utilisateur spécifié à 50 et la priorité minimale de la tâche à 250.

```powershell
$userObjectId = (Get-AzAdUser -SearchString "garymcdaniel@contoso.com").Id

New-AdlAnalyticsComputePolicy -Account $adla -Name "GaryMcDaniel" -ObjectId $objectId -ObjectType User -MaxDegreeOfParallelismPerJob 50 -MinPriorityPerJob 250
```

## <a name="manage-files"></a>Gérer les fichiers

### <a name="check-for-the-existence-of-a-file"></a>Vérifier l’existence d’un fichier

```powershell
Test-AdlStoreItem -Account $adls -Path "/data.csv"
```

### <a name="uploading-and-downloading"></a>Chargement et téléchargement

Chargez un fichier.

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\data.tsv" -Destination "/data_copy.csv"
```

Chargez un dossier entier de manière récursive.

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\myData\" -Destination "/myData/" -Recurse
```

Téléchargez un fichier.

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/data.csv" -Destination "c:\data.csv"
```

Téléchargez un dossier entier de manière récursive.

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/" -Destination "c:\myData\" -Recurse
```

> [!NOTE]
> Si le processus de chargement ou de téléchargement est interrompu, vous pouvez tenter de reprendre le processus en exécutant à nouveau la cmdlet avec l’indicateur ``-Resume``.

## <a name="manage-the-u-sql-catalog"></a>Gérer le catalogue U-SQL

Le catalogue U-SQL est utilisé pour structurer les données et le code afin que les scripts U-SQL puissent les partager. Le catalogue permet les meilleures performances possibles avec les données comprises dans Azure Data Lake. Pour plus d'informations, consultez [Utilisation du catalogue U-SQL](./data-lake-analytics-u-sql-get-started.md).

### <a name="list-items-in-the-u-sql-catalog"></a>Lister les éléments dans le catalogue U-SQL

```powershell
# List U-SQL databases
Get-AdlCatalogItem -Account $adla -ItemType Database

# List tables within a database
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database"

# List tables within a schema.
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database.schema"
```

### <a name="list-all-the-assemblies-the-u-sql-catalog"></a>Répertorier tous les assemblys du catalogue U-SQL

```powershell
$dbs = Get-AdlCatalogItem -Account $adla -ItemType Database

foreach ($db in $dbs)
{
    $asms = Get-AdlCatalogItem -Account $adla -ItemType Assembly -Path $db.Name

    foreach ($asm in $asms)
    {
        $asmname = "[" + $db.Name + "].[" + $asm.Name + "]"
        Write-Host $asmname
    }
}
```

### <a name="get-details-about-a-catalog-item"></a>Obtenir des détails sur un élément de catalogue

```powershell
# Get details of a table
Get-AdlCatalogItem  -Account $adla -ItemType Table -Path "master.dbo.mytable"

# Test existence of a U-SQL database.
Test-AdlCatalogItem  -Account $adla -ItemType Database -Path "master"
```

### <a name="store-credentials-in-the-catalog"></a>Stocker les informations d’identification dans le catalogue

Dans une base de données U-SQL, créez un objet informations d’identification pour une base de données hébergée dans Azure. Actuellement, les informations d’identification U-SQL sont le seul type d’élément de catalogue que vous pouvez créer via PowerShell.

```powershell
$dbName = "master"
$credentialName = "ContosoDbCreds"
$dbUri = "https://contoso.database.windows.net:8080"

New-AdlCatalogCredential -AccountName $adla `
          -DatabaseName $db `
          -CredentialName $credentialName `
          -Credential (Get-Credential) `
          -Uri $dbUri
```

## <a name="manage-firewall-rules"></a>Gérer les règles de pare-feu

### <a name="list-firewall-rules"></a>Répertorier les règles de pare-feu

```powershell
Get-AdlAnalyticsFirewallRule -Account $adla
```

### <a name="add-a-firewall-rule"></a>Ajouter une règle de pare-feu

```powershell
$ruleName = "Allow access from on-prem server"
$startIpAddress = "<start IP address>"
$endIpAddress = "<end IP address>"

Add-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="modify-a-firewall-rule"></a>Modifier une règle de pare-feu

```powershell
Set-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="remove-a-firewall-rule"></a>Supprimer une règle de pare-feu

```powershell
Remove-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName
```

### <a name="allow-azure-ip-addresses"></a>Autoriser les adresses IP Azure

```powershell
Set-AdlAnalyticsAccount -Name $adla -AllowAzureIpState Enabled
```

```powershell
Set-AdlAnalyticsAccount -Name $adla -FirewallState Enabled
Set-AdlAnalyticsAccount -Name $adla -FirewallState Disabled
```

## <a name="working-with-azure"></a>Utilisation d’Azure

### <a name="get-error-details"></a>Voir les détails de l’erreur

```powershell
Resolve-AzError -Last
```

### <a name="verify-if-you-are-running-as-an-administrator-on-your-windows-machine"></a>Vérifier que vous exécutez en tant qu’administrateur sur votre machine Windows

```powershell
function Test-Administrator  
{  
    $user = [Security.Principal.WindowsIdentity]::GetCurrent();
    $p = New-Object Security.Principal.WindowsPrincipal $user
    $p.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)  
}
```

### <a name="find-a-tenantid"></a>Trouver un TenantID

À partir du nom d’un abonnement :

```powershell
function Get-TenantIdFromSubscriptionName( [string] $subname )
{
    $sub = (Get-AzSubscription -SubscriptionName $subname)
    $sub.TenantId
}

Get-TenantIdFromSubscriptionName "ADLTrainingMS"
```

À partir de l’ID d’un abonnement :

```powershell
function Get-TenantIdFromSubscriptionId( [string] $subid )
{
    $sub = (Get-AzSubscription -SubscriptionId $subid)
    $sub.TenantId
}

$subid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Get-TenantIdFromSubscriptionId $subid
```

À partir d’une adresse de domaine, comme « contoso.com »

```powershell
function Get-TenantIdFromDomain( $domain )
{
    $url = "https://login.windows.net/" + $domain + "/.well-known/openid-configuration"
    return (Invoke-WebRequest $url|ConvertFrom-Json).token_endpoint.Split('/')[3]
}

$domain = "contoso.com"
Get-TenantIdFromDomain $domain
```

### <a name="list-all-your-subscriptions-and-tenant-ids"></a>Répertorier tous vos abonnements et leurs ID locataire

```powershell
$subs = Get-AzSubscription
foreach ($sub in $subs)
{
    Write-Host $sub.Name "("  $sub.Id ")"
    Write-Host "`tTenant Id" $sub.TenantId
}
```

## <a name="create-a-data-lake-analytics-account-using-a-template"></a>Créer un compte Data Lake Analytics à l’aide d’un modèle

Vous pouvez également utiliser un modèle de groupe de ressources Azure à l’aide de l’exemple suivant : [Créer un compte Data Lake Analytics à l’aide d’un modèle](https://github.com/Azure-Samples/data-lake-analytics-create-account-with-arm-template).

## <a name="next-steps"></a>Étapes suivantes
* [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* Bien démarrer avec Data Lake Analytics à l’aide du [portail Azure](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [Azure CLI](data-lake-analytics-get-started-cli.md)
* Gérer Azure Data Lake Analytics à l’aide du [portail Azure](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md)