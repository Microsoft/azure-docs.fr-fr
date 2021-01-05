---
title: 'Azure SQL Managed Instance : Conservation de sauvegardes à long terme (PowerShell)'
description: Découvrez comment stocker et restaurer des sauvegardes automatisées sur des conteneurs de Stockage Blob Azure distincts pour une instance SQL Managed Instance avec PowerShell.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 04/29/2020
ms.openlocfilehash: ec193eab02d937e9d93b8632fa171fec8227d6c2
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94987488"
---
# <a name="manage-azure-sql-managed-instance-long-term-backup-retention-powershell"></a>Gérer la conservation des sauvegardes à long terme Azure SQL Managed Instance (PowerShell)
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Dans Azure SQL Managed Instance, vous pouvez configurer une stratégie de [conservation des sauvegardes à long terme](../database/long-term-retention-overview.md#sql-managed-instance-support) (LTR) grâce à une fonctionnalité en préversion publique limitée. Cela vous permet de conserver automatiquement les sauvegardes de base de données dans des conteneurs Stockage Blob Azure distincts pendant 10 ans maximum. Vous pouvez ensuite récupérer une base de données à l’aide de ces sauvegardes avec PowerShell.

   > [!IMPORTANT]
   > La conservation LTR des instances gérées, actuellement en préversion limitée, est disponible pour les abonnements EA et CSP selon les cas. Pour demander une inscription, créez un [ticket de support Azure](https://azure.microsoft.com/support/create-ticket/). Pour le type de problème, sélectionnez « Problème technique », pour le service, choisissez « SQL Database Managed Instance » et pour le type de problème, sélectionnez **Sauvegarde, restauration et continuité d’activité/conservation de sauvegarde à long terme**. Dans votre demande, indiquez que vous souhaitez être inscrit à la préversion publique limitée de LTR pour Managed Instance.

Les sections suivantes vous montrent comment utiliser PowerShell pour configurer la rétention des sauvegardes à long terme, afficher des sauvegardes dans le stockage SQL Azure et restaurer à partir d’une sauvegarde dans le stockage SQL Azure.

## <a name="azure-roles-to-manage-long-term-retention"></a>Rôles Azure pour gérer la conservation à long terme

Pour **Get-AzSqlInstanceDatabaseLongTermRetentionBackup** et **Restore-AzSqlInstanceDatabase**, vous devez avoir l’un des rôles suivants :

- Rôle Propriétaire de l’abonnement
- Rôle Collaborateur Managed Instance
- Rôle personnalisé avec les autorisations suivantes :
  - `Microsoft.Sql/locations/longTermRetentionManagedInstanceBackups/read`
  - `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionManagedInstanceBackups/read`
  - `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionDatabases/longTermRetentionManagedInstanceBackups/read`

Pour **Remove-AzSqlInstanceDatabaseLongTermRetentionBackup**, vous devez avoir l’un des rôles suivants :

- Rôle Propriétaire de l’abonnement
- Rôle personnalisé avec l’autorisation suivante :
  - `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionDatabases/longTermRetentionManagedInstanceBackups/delete`

> [!NOTE]
> Le rôle Collaborateur Managed Instance n’a pas l’autorisation de supprimer les sauvegardes LTR.

Les autorisations Azure RBAC peuvent être accordées dans l’étendue de l’*abonnement* ou du *groupe de ressources*. Toutefois, pour accéder aux sauvegardes LTR appartenant à une instance supprimée, il faut accorder l’autorisation dans l’étendue de *l’abonnement* de cette instance.

- `Microsoft.Sql/locations/longTermRetentionManagedInstances/longTermRetentionDatabases/longTermRetentionManagedInstanceBackups/delete`

## <a name="create-an-ltr-policy"></a>Créer une stratégie de rétention à long terme

```powershell
# get the Managed Instance
$subId = "<subscriptionId>"
$instanceName = "<instanceName>"
$resourceGroup = "<resourceGroupName>"
$dbName = "<databaseName>"

Connect-AzAccount
Select-AzSubscription -SubscriptionId $subId

$instance = Get-AzSqlInstance -Name $instanceName -ResourceGroupName $resourceGroup

# create LTR policy with WeeklyRetention = 12 weeks. MonthlyRetention and YearlyRetention = 0 by default.
Set-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
   -DatabaseName $dbName -ResourceGroupName $resourceGroup -WeeklyRetention P12W

# create LTR policy with WeeklyRetention = 12 weeks, YearlyRetention = 5 years and WeekOfYear = 16 (week of April 15). MonthlyRetention = 0 by default.
Set-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
    -DatabaseName $dbName -ResourceGroupName $resourceGroup -WeeklyRetention P12W -YearlyRetention P5Y -WeekOfYear 16
```

## <a name="view-ltr-policies"></a>Afficher des stratégies de rétention à long terme

Cet exemple montre comment lister les stratégies LTR au sein d’une instance.

```powershell
# gets the current version of LTR policy for the database
$ltrPolicies = Get-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
    -DatabaseName $dbName -ResourceGroupName $resourceGroup
```

## <a name="clear-an-ltr-policy"></a>Effacer une stratégie de rétention à long terme

Cet exemple montre comment effacer une stratégie de rétention à long terme d’une base de données

```powershell
Set-AzSqlInstanceDatabaseBackupLongTermRetentionPolicy -InstanceName $instanceName `
   -DatabaseName $dbName -ResourceGroupName $resourceGroup -RemovePolicy
```

## <a name="view-ltr-backups"></a>Afficher des sauvegardes de rétention à long terme

Cet exemple montre comment lister les sauvegardes LTR au sein d’une instance.

```powershell
# get the list of all LTR backups in a specific Azure region
# backups are grouped by the logical database id, within each group they are ordered by the timestamp, the earliest backup first
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location

# get the list of LTR backups from the Azure region under the given managed instance
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -InstanceName $instanceName

# get the LTR backups for a specific database from the Azure region under the given managed instance
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -InstanceName $instanceName -DatabaseName $dbName

# list LTR backups only from live databases (you have option to choose All/Live/Deleted)
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -DatabaseState Live

# only list the latest LTR backup for each database
$ltrBackups = Get-AzSqlInstanceDatabaseLongTermRetentionBackup -Location $instance.Location -InstanceName $instanceName -OnlyLatestPerDatabase
```

## <a name="delete-ltr-backups"></a>Supprimer des sauvegardes de rétention à long terme

Cet exemple montre comment supprimer une sauvegarde de rétention à long terme de la liste des sauvegardes.

```powershell
# remove the earliest backup
$ltrBackup = $ltrBackups[0]
Remove-AzSqlInstanceDatabaseLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId
```

> [!IMPORTANT]
> La suppression de sauvegardes de rétention à long terme n’est pas réversible. Pour supprimer une sauvegarde LTR une fois l’instance supprimée, vous devez disposer d’une autorisation étendue à l’abonnement. Vous pouvez configurer des notifications sur chaque suppression dans Azure Monitor en filtrant sur l’opération « Supprime une sauvegarde de conservation à long terme ». Le journal d’activité contient des informations sur la personne qui a effectué la requête et quand. Consultez [Créer des alertes de journal d’activité](../../azure-monitor/platform/alerts-activity-log.md) pour obtenir des instructions détaillées.

## <a name="restore-from-ltr-backups"></a>Restaurer à partir de sauvegardes de rétention à long terme

Cet exemple montre comment restaurer à partir d’une sauvegarde de rétention à long terme. Notez que cette interface n’a pas changé, mais que le paramètre d’ID de ressource requiert désormais l’ID de ressource de sauvegarde de rétention à long terme.

```powershell
# restore a specific LTR backup as an P1 database on the instance $instanceName of the resource group $resourceGroup
Restore-AzSqlInstanceDatabase -FromLongTermRetentionBackup -ResourceId $ltrBackup.ResourceId `
   -TargetInstanceName $instanceName -TargetResourceGroupName $resourceGroup -TargetInstanceDatabaseName $dbName
```

> [!IMPORTANT]
> Pour effectuer une restauration à partir d’une sauvegarde LTR après la suppression de l’instance, vous devez disposer d’autorisations étendues à l’abonnement de l’instance, cet abonnement devant être actif. Vous devez également omettre le paramètre facultatif -ResourceGroupName.

> [!NOTE]
> À ce stade, vous pouvez vous connecter à la base de données restaurée à l’aide de SQL Server Management Studio pour exécuter les tâches nécessaires, notamment pour extraire un bit de données de la base de données restaurée à copier dans la base de données existante ou pour supprimer la base de données existante et renommer la base de données restaurée avec le nom de la base de données existante. Consultez [Restauration dans le temps](../database/recovery-using-backups.md#point-in-time-restore).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les sauvegardes automatiques générées par le service, consultez la page [Sauvegardes automatiques](../database/automated-backups-overview.md).
- Pour plus d’informations sur la rétention des sauvegardes à long terme, consultez l’article décrivant la [rétention des sauvegardes à long terme](../database/long-term-retention-overview.md).
