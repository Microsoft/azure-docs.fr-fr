---
title: Restaurer un pool SQL dédié (anciennement SQL DW) supprimé
description: Guide pratique sur la restauration d'un pool SQL dédié supprimé dans Azure Synapse Analytics.
services: synapse-analytics
author: anumjs
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 08/29/2018
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 7264791654bf1b646338f0d429930b63f0cc3a06
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96449930"
---
# <a name="restore-a-deleted-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Restaurer un pool SQL dédié (anciennement SQL DW) supprimé dans Azure Synapse Analytics

Dans cet article, vous allez apprendre à restaurer un pool SQL dédié (anciennement SQL DW) à l'aide du portail Azure ou de PowerShell.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

**Vérifiez votre capacité de DTU.** Chaque pool SQL dédié (anciennement SQL DW) est hébergé par un [serveur SQL logique](../../azure-sql/database/logical-servers.md) (par exemple, myserver.database.windows.net) qui dispose d'un quota de DTU par défaut.  Vérifiez que le quota de DTU restant sur le serveur est suffisant pour la base de données en cours de restauration. Pour savoir comment calculer la capacité DTU nécessaire ou pour demander davantage de capacité DTU, consultez [Request a DTU quota change](sql-data-warehouse-get-started-create-support-ticket.md)(Demander une modification du quota DTU).

## <a name="restore-a-deleted-data-warehouse-through-powershell"></a>Restaurer un entrepôt de données supprimé via PowerShell

Pour restaurer un pool SQL dédié (anciennement SQL DW) supprimé, utilisez la cmdlet [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Si le serveur correspondant a également été supprimé, vous ne pouvez pas restaurer cet entrepôt de données.

1. Avant de commencer, veillez à [installer Azure PowerShell](/powershell/azure/?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
2. Ouvrez PowerShell.
3. Connectez-vous à votre compte Azure et répertoriez tous les abonnements associés à votre compte.
4. Sélectionnez l'abonnement qui contient le pool SQL dédié (anciennement SQL DW) supprimé à restaurer.
5. Récupérez l’entrepôt de données supprimé spécifique.
6. Restaurer le pool SQL dédié (anciennement SQL DW) supprimé
    1. Pour restaurer sur un autre serveur le pool SQL dédié (anciennement SQL DW) supprimé, veillez à spécifier le nom de ce serveur.  Ce serveur peut également se trouver dans un groupe de ressources et une région différents.
    1. Pour effectuer une restauration sur un autre abonnement, utilisez le bouton [Déplacer](../../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#use-the-portal) afin de déplacer le serveur vers un autre abonnement.
7. Vérifiez que l’entrepôt de données restauré est en ligne.
8. Une fois la restauration terminée, vous pouvez configurer votre entrepôt de données récupéré en suivant [Configurer votre base de données après récupération](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

```Powershell
$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
#$TargetResourceGroupName="<YourTargetResourceGroupName>" # uncomment to restore to a different server.
#$TargetServerName="<YourtargetServerNameWithoutURLSuffixSeeNote>"
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName

# Get the deleted database to restore
$DeletedDatabase = Get-AzSqlDeletedDatabaseBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Restore deleted database
$RestoredDatabase = Restore-AzSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Use the following command to restore deleted data warehouse to a different server
#$RestoredDatabase = Restore-AzSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID

# Verify the status of restored database
$RestoredDatabase.status
```

## <a name="restore-a-deleted-database-using-the-azure-portal"></a>Restaurer une base de données supprimée à l’aide du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez au serveur sur lequel votre entrepôt de données supprimé était hébergé.
3. Sélectionnez l’icône **Bases de données supprimées** dans la table des matières.

    ![Bases de données supprimées](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-01.png)

4. Sélectionnez la base de données Azure Synapse Analytics que vous souhaitez restaurer.

    ![Sélectionner Bases de données supprimées](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-11.png)

5. Spécifiez un nouveau **nom pour la base de données** et cliquez sur **OK**

    ![Spécifier un nom de base de données](./media/sql-data-warehouse-restore-deleted-dw/restoring-deleted-21.png)

## <a name="next-steps"></a>Étapes suivantes

- [Restaurer un pool SQL dédié (anciennement SQL DW) existant](sql-data-warehouse-restore-active-paused-dw.md)
- [Restauration à partir d'un pool SQL dédié (anciennement SQL DW) de géosauvegarde](sql-data-warehouse-restore-from-geo-backup.md)
