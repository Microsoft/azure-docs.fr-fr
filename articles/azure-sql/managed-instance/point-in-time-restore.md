---
title: Restauration à un point dans le temps
titleSuffix: Azure SQL Managed Instance
description: Restaurer une base de données d'Azure SQL Managed Instance à un point antérieur dans le temps
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, mathoma
ms.date: 08/25/2019
ms.openlocfilehash: 31be497d017cb60de6f46d7657889c9c1fabef4a
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92788347"
---
# <a name="restore-a-database-in-azure-sql-managed-instance-to-a-previous-point-in-time"></a>Restaurer une base de données d'Azure SQL Managed Instance à un point antérieur dans le temps
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Utilisez la limite de restauration dans le temps (PITR) pour créer une base de données en tant que copie d’une autre base de données à un moment donné dans le passé. Cet article explique comment effectuer une limite de restauration dans le temps d'une base de données Azure SQL Managed Instance.

La limite de restauration dans le temps est utile dans les scénarios de récupération, tels que les incidents provoqués par des erreurs, les données chargées de façon incorrecte ou la suppression de données cruciales. Vous pouvez également l’utiliser simplement à des fins de test ou d’audit. Les fichiers de sauvegarde sont conservés pendant une période comprise entre 7 et 35 jours, en fonction des paramètres de votre base de données.

La limite de restauration dans le temps peut restaurer une base de données :

- à partir d’une base de données existante.
- à partir d’une base de données supprimée.
- sur la même instance de SQL Managed Instance ou sur une autre. 

## <a name="limitations"></a>Limites

La limite de restauration dans le temps de SQL Managed Instance présente les limitations suivantes :

- Lorsque vous restaurez d'une instance de SQL Managed Instance sur une autre, les deux instances doivent se trouver dans le même abonnement et la même région. La restauration inter-régions et inter-abonnements ne sont actuellement pas prises en charge.
- La limite de restauration dans le temps d'une instance complète de SQL Managed Instance n'est pas possible. Cet article explique uniquement ce qui est possible : une limite de restauration dans le temps d'une base de données hébergée sur SQL Managed Instance.

> [!WARNING]
> Tenez compte de la taille de stockage de votre instance de SQL Managed Instance. Selon la taille des données à restaurer, vous risquez de manquer de stockage d’instance. S’il n’y a pas assez d’espace pour les données restaurées, utilisez une approche différente.

Le tableau suivant présente les scénarios de limite de restauration dans le temps pour SQL Managed Instance :

|           |Restaurer la base de données existante sur la même instance de SQL Managed Instance| Restaurer la base de données existante sur une autre instance de SQL Managed Instance|Restaurer la base de données déposée sur la même instance de SQL Managed Instance|Restaurer la base de données déposée sur une autre instance de SQL Managed Instance|
|:----------|:----------|:----------|:----------|:----------|
|**Azure portal**| Oui|Non |Oui|Non|
|**Azure CLI**|Oui |Oui |Non|Non|
|**PowerShell**| Oui|Oui |Oui|Oui|

## <a name="restore-an-existing-database"></a>Restaurer une base de données existante

Restaurez une base de données existante sur la même instance de SQL Managed Instance à l'aide du portail Azure, de PowerShell ou d'Azure CLI. Pour restaurer une base de données sur une autre instance de SQL Managed Instance, utilisez PowerShell ou Azure CLI en spécifiant les propriétés du groupe de ressources et de l'instance de SQL Managed Instance cibles. Si vous spécifiez pas ces paramètres, la base de données sera restaurée par défaut sur l'instance existante de SQL Managed Instance. Le portail Azure ne prend actuellement pas en charge la restauration vers une autre instance de SQL Managed Instance.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Accédez à votre instance de SQL Managed Instance et sélectionnez la base de données à restaurer.
3. Sélectionnez **Restaurer** dans la page de la base de données :

    ![Restaurer une base de données à l’aide du portail Azure](./media/point-in-time-restore/restore-database-to-mi.png)

4. Dans la page **Restaurer** , sélectionnez le point pour la date et l’heure auquel restaurer la base de données.
5. Sélectionnez **Confirmer** pour restaurer votre base de données. Cette action démarre le processus de restauration qui crée une nouvelle base de données et est rempli avec les données de la base de données d’origine au point spécifié dans le temps. Pour plus d’informations sur le processus de récupération, consultez [Heure de récupération](../database/recovery-using-backups.md#recovery-time).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si Azure PowerShell n’est pas encore installé, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

Pour restaurer la base de données à l’aide de PowerShell, spécifiez vos valeurs pour les paramètres de la commande suivante. Exécutez ensuite la commande :

```powershell-interactive
$subscriptionId = "<Subscription ID>"
$resourceGroupName = "<Resource group name>"
$managedInstanceName = "<SQL Managed Instance name>"
$databaseName = "<Source-database>"
$pointInTime = "2018-06-27T08:51:39.3882806Z"
$targetDatabase = "<Name of new database to be created>"

Get-AzSubscription -SubscriptionId $subscriptionId
Select-AzSubscription -SubscriptionId $subscriptionId

Restore-AzSqlInstanceDatabase -FromPointInTimeBackup `
                              -ResourceGroupName $resourceGroupName `
                              -InstanceName $managedInstanceName `
                              -Name $databaseName `
                              -PointInTime $pointInTime `
                              -TargetInstanceDatabaseName $targetDatabase `
```

Pour restaurer la base de données vers une autre instance de SQL Managed Instance, spécifiez également le nom du groupe de ressources et le nom de l'instance de SQL Managed Instance cibles :  

```powershell-interactive
$targetResourceGroupName = "<Resource group of target SQL Managed Instance>"
$targetInstanceName = "<Target SQL Managed Instance name>"

Restore-AzSqlInstanceDatabase -FromPointInTimeBackup `
                              -ResourceGroupName $resourceGroupName `
                              -InstanceName $managedInstanceName `
                              -Name $databaseName `
                              -PointInTime $pointInTime `
                              -TargetInstanceDatabaseName $targetDatabase `
                              -TargetResourceGroupName $targetResourceGroupName `
                              -TargetInstanceName $targetInstanceName 
```

Pour plus d’informations, consultez [.Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si Azure CLI n’est pas encore installé, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

Pour restaurer la base de données à l’aide d’Azure CLI, spécifiez vos valeurs pour les paramètres de la commande suivante. Exécutez ensuite la commande :

```azurecli-interactive
az sql midb restore -g mygroupname --mi myinstancename |
-n mymanageddbname --dest-name targetmidbname --time "2018-05-20T05:34:22"
```

Pour restaurer la base de données vers une autre instance de SQL Managed Instance, spécifiez également le nom du groupe de ressources et le nom de l'instance de SQL Managed Instance :  

```azurecli-interactive
az sql midb restore -g mygroupname --mi myinstancename -n mymanageddbname |
       --dest-name targetmidbname --time "2018-05-20T05:34:22" |
       --dest-resource-group mytargetinstancegroupname |
       --dest-mi mytargetinstancename
```

Pour obtenir une explication détaillée des paramètres disponibles, consultez la [documentation CLI consacrée à la restauration d'une base de données sur une instance de SQL Managed Instance](/cli/azure/sql/midb?view=azure-cli-latest#az-sql-midb-restore).

---

## <a name="restore-a-deleted-database"></a>La restauration d’une base de données supprimée

La restauration d’une base de données supprimée peut être effectuée en utilisant PowerShell ou le portail Azure. Pour restaurer une base de données supprimée sur la même instance, utilisez le portail Azure ou PowerShell. Pour restaurer une base de données supprimée sur une autre instance, utilisez PowerShell. 

### <a name="portal"></a>Portail 


Pour récupérer une base de données managée à partir du portail Azure, ouvrez la page de présentation de l'instance de SQL Managed Instance, puis sélectionnez **Bases de données supprimées** . Choisissez une base de données supprimée que vous souhaitez restaurer, puis tapez le nom de la nouvelle base de données qui sera créée avec les données restaurées à partir de la sauvegarde.

  ![Capture d’écran de la restauration d’une base de données d’instance Azure SQL supprimée](./media/point-in-time-restore/restore-deleted-sql-managed-instance-annotated.png)

../../sql-database

### <a name="powershell"></a>PowerShell

Pour restaurer une base de données sur la même instance, mettez à jour les valeurs des paramètres, puis exécutez la commande PowerShell suivante : 

```powershell-interactive
$subscriptionId = "<Subscription ID>"
Get-AzSubscription -SubscriptionId $subscriptionId
Select-AzSubscription -SubscriptionId $subscriptionId

$resourceGroupName = "<Resource group name>"
$managedInstanceName = "<SQL Managed Instance name>"
$deletedDatabaseName = "<Source database name>"
$targetDatabaseName = "<target database name>"

$deletedDatabase = Get-AzSqlDeletedInstanceDatabaseBackup -ResourceGroupName $resourceGroupName `
-InstanceName $managedInstanceName -DatabaseName $deletedDatabaseName

Restore-AzSqlinstanceDatabase -Name $deletedDatabase.Name `
   -InstanceName $deletedDatabase.ManagedInstanceName `
   -ResourceGroupName $deletedDatabase.ResourceGroupName `
   -DeletionDate $deletedDatabase.DeletionDate `
   -PointInTime UTCDateTime `
   -TargetInstanceDatabaseName $targetDatabaseName
```

Pour restaurer la base de données vers une autre instance de SQL Managed Instance, spécifiez également le nom du groupe de ressources et le nom de l'instance de SQL Managed Instance cibles :

```powershell-interactive
$targetResourceGroupName = "<Resource group of target SQL Managed Instance>"
$targetInstanceName = "<Target SQL Managed Instance name>"

Restore-AzSqlinstanceDatabase -Name $deletedDatabase.Name `
   -InstanceName $deletedDatabase.ManagedInstanceName `
   -ResourceGroupName $deletedDatabase.ResourceGroupName `
   -DeletionDate $deletedDatabase.DeletionDate `
   -PointInTime UTCDateTime `
   -TargetInstanceDatabaseName $targetDatabaseName `
   -TargetResourceGroupName $targetResourceGroupName `
   -TargetInstanceName $targetInstanceName 
```

## <a name="overwrite-an-existing-database"></a>Remplacer une base de données existante

Pour remplacer une base de données existante, vous devez :

1. Déposez la base de données existante que vous souhaitez remplacer.
2. Renommez la base de données restaurée dans le temps par le nom de la base de données qui a été déposée.

### <a name="drop-the-original-database"></a>Déposer la base de données d’origine

Vous pouvez déposer une base de données à l’aide du portail Azure, de PowerShell ou de Azure CLI.

Vous pouvez également déposer la base de données en vous connectant directement à l'instance de SQL Managed Instance, en lançant SQL Server Management Studio (SSMS) et en exécutant la commande Transact-SQL (T-SQL) suivante :

```sql
DROP DATABASE WorldWideImporters;
```

Utilisez l'une des méthodes suivantes pour vous connecter à votre base de données de l'instance de SQL Managed Instance :

- [SSMS/Azure Data Studio via une machine virtuelle Azure](./connect-vm-instance-configure.md)
- [Point à site](./point-to-site-p2s-configure.md)
- [Point de terminaison public](./public-endpoint-configure.md)

# <a name="portal"></a>[Portail](#tab/azure-portal)

Sur le portail Azure, sélectionnez la base de données à partir de l'instance de SQL Managed Instance et sélectionnez **Supprimer** .

   ![Supprimer une base de données à l’aide du portail Azure](./media/point-in-time-restore/delete-database-from-mi.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez la commande PowerShell suivante pour supprimer une base de données existante d'une instance de SQL Managed Instance :

```powershell
$resourceGroupName = "<Resource group name>"
$managedInstanceName = "<SQL Managed Instance name>"
$databaseName = "<Source database>"

Remove-AzSqlInstanceDatabase -Name $databaseName -InstanceName $managedInstanceName -ResourceGroupName $resourceGroupName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Utilisez la commande Azure CLI suivante pour supprimer une base de données existante d'une instance de SQL Managed Instance :

```azurecli-interactive
az sql midb delete -g mygroupname --mi myinstancename -n mymanageddbname
```

---

### <a name="alter-the-new-database-name-to-match-the-original-database-name"></a>Modifier le nouveau nom de base de données pour qu’il corresponde au nom de la base de données d’origine

Connectez-vous directement à l'instance de SQL Managed Instance et démarrez SQL Server Management Studio. Vous pouvez alors exécuter la requête Transact-SQL (T-SQL) suivante. La requête remplacera le nom de la base de données restaurée par celui de la base de données déposée que vous envisagez de remplacer.

```sql
ALTER DATABASE WorldWideImportersPITR MODIFY NAME = WorldWideImporters;
```

Utilisez l'une des méthodes suivantes pour vous connecter à votre base de données de l'instance de SQL Managed Instance :

- [Machine virtuelle Azure](./connect-vm-instance-configure.md)
- [Point à site](./point-to-site-p2s-configure.md)
- [Point de terminaison public](./public-endpoint-configure.md)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [sauvegardes automatisées](../database/automated-backups-overview.md).