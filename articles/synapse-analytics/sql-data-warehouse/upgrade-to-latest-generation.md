---
title: Mise à niveau vers la dernière génération de pool SQL dédié (anciennement SQL DW)
description: Mettez à niveau un pool SQL dédié (anciennement SQL DW) Azure Synapse Analytics vers la dernière génération de l’architecture matérielle et de stockage Azure.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/19/2019
ms.author: martinle
ms.reviewer: jrasnick
ms.custom: seo-lt-2019
ms.openlocfilehash: b5a9d1781bd0498ac6ad74439b1572c52e3c345a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459257"
---
# <a name="optimize-performance-by-upgrading-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Optimiser les performances en mettant à niveau un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

Mettez à niveau votre pool SQL dédié (anciennement SQL DW) vers la dernière génération de l’architecture matérielle et de stockage Azure.

## <a name="why-upgrade"></a>Pourquoi procéder à une mise à niveau ?

Vous pouvez maintenant effectuer une mise à niveau de manière fluide vers le niveau Gen2 optimisé pour le calcul du pool SQL dédié (anciennement SQL DW) dans le portail Azure pour les [régions prises en charge](gen2-migration-schedule.md#automated-schedule-and-region-availability-table). Si votre région ne prend pas en charge la mise à niveau automatique, vous pouvez passer à une région prise en charge ou attendre que la mise à niveau automatique soit disponible dans votre région. Effectuez la mise à niveau dès maintenant pour bénéficier de la dernière génération de matériel et de l’architecture de stockage améliorée d’Azure, y compris de performances plus rapides, d’une plus grande évolutivité et d’un stockage en colonnes illimité.

> [!VIDEO https://www.youtube.com/embed/9B2F0gLoyss]

> [!IMPORTANT]
> Cette mise à niveau s’applique aux pools SQL dédiés (anciennement SQL DW) de niveau Gen1 optimisé pour le calcul dans les [régions prises en charge](gen2-migration-schedule.md#automated-schedule-and-region-availability-table).

## <a name="before-you-begin"></a>Avant de commencer

1. Vérifiez si votre [région](gen2-migration-schedule.md#automated-schedule-and-region-availability-table) est prise en charge pour la migration GEN1 vers GEN2. Prenez note des dates de migration automatique. Pour éviter tout conflit avec le processus automatisé, planifiez votre migration manuelle avant la date de début du processus automatisé.
2. Si vous vous trouvez dans une région qui n’est pas encore prise en charge, continuez à vérifier si votre région doit être ajoutée ou [effectuer la mise à niveau en utilisant la restauration](#upgrade-from-an-azure-geographical-region-using-restore-through-the-azure-portal) dans une région prise en charge.
3. Si votre région est prise en charge, [effectuez la mise à niveau via le portail Azure](#upgrade-in-a-supported-region-using-the-azure-portal).
4. **Sélectionnez le niveau de performance suggéré** pour un pool SQL dédié (anciennement SQL DW) en fonction de votre niveau de performance actuel sur le niveau Gen1 optimisé pour le calcul en utilisant le mappage ci-dessous :

   | Niveau Gen1 optimisé pour le calcul | Niveau Gen2 optimisé pour le calcul |
   | :-------------------------: | :-------------------------: |
   |            DW100            |           DW100c            |
   |            DW200            |           DW200c            |
   |            DW300            |           DW300c            |
   |            DW400            |           DW400c            |
   |            DW500            |           DW500c            |
   |            DW600            |           DW500c            |
   |           DW1000            |           DW1000c           |
   |           DW1200            |           DW1000c           |
   |           DW1500            |           DW1500c           |
   |           DW2000            |           DW2000c           |
   |           DW3000            |           DW3000c           |
   |           DW6000            |           DW6000c           |

> [!NOTE]
> Les niveaux de performance suggérés ne constituent pas une conversion directe. Par exemple, nous recommandons de passer du DW600 au DW500c.

## <a name="upgrade-in-a-supported-region-using-the-azure-portal"></a>Mise à niveau dans une région prise en charge à l’aide du Portail Azure

- La migration de Gen1 vers Gen2 via le portail Azure est définitive. Il est impossible de revenir à Gen1.
- Le pool SQL dédié (anciennement SQL DW) doit être en cours d’exécution pour migrer vers Gen2.

### <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Connectez-vous au [portail Azure](https://portal.azure.com/).
- Vérifiez que le pool SQL dédié (anciennement SQL DW) est en cours d’exécution, ce qui est obligatoire pour migrer vers Gen2.

### <a name="powershell-upgrade-commands"></a>Commandes de mise à niveau PowerShell

1. Si le pool SQL dédié (anciennement SQL DW) de niveau Gen1 optimisé pour le calcul à mettre à niveau est suspendu, [reprenez l’exécution du pool SQL dédié (anciennement SQL DW)](pause-and-resume-compute-portal.md).

2. Attendez-vous à quelques minutes de temps d’arrêt.

3. Identifiez les références de code à des niveaux de performances Gen1 optimisé pour le calcul et modifiez-les à leur niveau de performances Gen2 optimisé pour le calcul équivalent. Les deux exemples ci-dessous illustrent où vous devez mettre à jour les références de code avant la mise à niveau :

   Commande PowerShell Gen1 d’origine :

   ```powershell
   Set-AzSqlDatabase -ResourceGroupName "myResourceGroup" -DatabaseName "mySampleDataWarehouse" -ServerName "mynewserver-20171113" -RequestedServiceObjectiveName "DW300"
   ```

   Remplacée par :

   ```powershell
   Set-AzSqlDatabase -ResourceGroupName "myResourceGroup" -DatabaseName "mySampleDataWarehouse" -ServerName "mynewserver-20171113" -RequestedServiceObjectiveName "DW300c"
   ```

   > [!NOTE]
   > -RequestedServiceObjectiveName "DW300" est remplacé par - RequestedServiceObjectiveName "DW300 **c**"
   >

   Commande T-SQL Gen1 d’origine :

   ```SQL
   ALTER DATABASE mySampleDataWarehouse MODIFY (SERVICE_OBJECTIVE = 'DW300') ;
   ```

   Remplacée par :

   ```sql
   ALTER DATABASE mySampleDataWarehouse MODIFY (SERVICE_OBJECTIVE = 'DW300c') ;
   ```

   > [!NOTE]
   > SERVICE_OBJECTIVE = 'DW300' est remplacé par SERVICE_OBJECTIVE = 'DW300 **c**'

## <a name="start-the-upgrade"></a>Lancer la mise à niveau

1. Accédez à votre pool SQL dédié (anciennement SQL DW) Gen1 optimisé pour le calcul dans le portail Azure. Si le pool SQL dédié (anciennement SQL DW) de niveau Gen1 optimisé pour le calcul à mettre à niveau est suspendu, [reprenez l’exécution du pool SQL dédié](pause-and-resume-compute-portal.md).
2. Sous l’onglet Tâches, sélectionnez ensuite **Mettre à niveau vers Gen2** : ![Upgrade_1](./media/upgrade-to-latest-generation/upgrade-to-gen2-1.png)

   > [!NOTE]
   > Si vous ne voyez pas la carte **Mettre à niveau vers la 2e génération** sous l’onglet Tâches, votre type d’abonnement est limité dans la région actuelle.
   > [Envoyez un ticket de support](sql-data-warehouse-get-started-create-support-ticket.md) pour faire en sorte que votre abonnement soit approuvé.

3. Vérifiez que l’exécution de votre charge de travail est terminée et arrêtée avant de mettre à niveau. Vous subirez un temps d’arrêt de quelques minutes avant la remise en ligne de votre pool SQL dédié (anciennement SQL DW) en tant que pool SQL dédié (anciennement SQL DW) de niveau Gen2 optimisé pour le calcul. **Sélectionnez Upgrade**  (Mettre à niveau) :

   ![Upgrade_2](./media/upgrade-to-latest-generation/upgrade-to-gen2-2.png)

4. **Surveillez votre mise à niveau** en vérifiant l’état dans le portail Azure :

   ![Upgrade3](./media/upgrade-to-latest-generation/upgrade-to-gen2-3.png)

   La première étape du processus de mise à niveau passe par l’opération de mise à l’échelle (« Mise à niveau - Hors connexion ») pendant laquelle toutes les sessions seront supprimées et les connexions fermées.

   La deuxième étape du processus de mise à niveau est la migration des données (« Mise à niveau - En ligne »). La migration des données est un processus en arrière-plan progressif en ligne. Ce processus déplace lentement les données en colonnes de l’ancienne architecture de stockage vers la nouvelle architecture de stockage, en utilisant un cache de disque SSD local. Pendant ce temps, votre pool SQL dédié (anciennement SQL DW) sera en ligne à des fins d’interrogation et de chargement. Vos données pourront être interrogées, qu’elles aient été migrées ou non. La migration des données se produit à des taux variables selon la taille de vos données, de votre niveau de performance et du nombre de vos segments de columnstore.

5. **Recommandation facultative :** Une fois l’opération de mise à l’échelle est terminée, vous pouvez accélérer le processus de migration en arrière-plan. Vous pouvez forcer immédiatement le déplacement des données en exécutant la commande [Alter Index Rebuild](sql-data-warehouse-tables-index.md) sur toutes les tables columnstore primaires que vous interrogez sur une plus large classe de SLO et de ressources. Cette opération, par rapport au processus en arrière-plan progressif dont l’exécution peut prendre plusieurs heures en fonction du nombre et de la taille de vos tables, est en mode **hors connexion**. Toutefois, la migration des données sera beaucoup plus rapide et vous pourrez tirer pleinement parti de la nouvelle architecture de stockage améliorée avec des groupes de lignes de haute qualité.

> [!NOTE]
> Alter Index Rebuild est une opération hors connexion et les tables ne seront pas disponibles tant que la reconstruction ne sera pas terminée.

La requête suivante génère les commandes Alter Index Rebuild exigées pour accélérer la migration des données :

```sql
SELECT 'ALTER INDEX [' + idx.NAME + '] ON ['
       + Schema_name(tbl.schema_id) + '].['
       + Object_name(idx.object_id) + '] REBUILD ' + ( CASE
                                                         WHEN (
                                                     (SELECT Count(*)
                                                      FROM   sys.partitions
                                                             part2
                                                      WHERE  part2.index_id
                                                             = idx.index_id
                                                             AND
                                                     idx.object_id =
                                                     part2.object_id)
                                                     > 1 ) THEN
              ' PARTITION = '
              + Cast(part.partition_number AS NVARCHAR(256))
              ELSE ''
                                                       END ) + '; SELECT ''[' +
              idx.NAME + '] ON [' + Schema_name(tbl.schema_id) + '].[' +
              Object_name(idx.object_id) + '] ' + (
              CASE
                WHEN ( (SELECT Count(*)
                        FROM   sys.partitions
                               part2
                        WHERE
                     part2.index_id =
                     idx.index_id
                     AND idx.object_id
                         = part2.object_id) > 1 ) THEN
              ' PARTITION = '
              + Cast(part.partition_number AS NVARCHAR(256))
              + ' completed'';'
              ELSE ' completed'';'
                                                    END )
FROM   sys.indexes idx
       INNER JOIN sys.tables tbl
               ON idx.object_id = tbl.object_id
       LEFT OUTER JOIN sys.partitions part
                    ON idx.index_id = part.index_id
                       AND idx.object_id = part.object_id
WHERE  idx.type_desc = 'CLUSTERED COLUMNSTORE';
```

## <a name="upgrade-from-an-azure-geographical-region-using-restore-through-the-azure-portal"></a>Mettre à niveau à partir d’une région géographique Azure en utilisant la restauration via le portail Azure

## <a name="create-a-user-defined-restore-point-using-the-azure-portal"></a>Créer un point de restauration défini par l’utilisateur à l’aide du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Accédez au pool SQL dédié (anciennement SQL DW) pour lequel vous voulez créer un point de restauration.

3. En haut de la section Vue d’ensemble, sélectionnez **+Nouveau point de restauration**.

    ![Nouveau point de restauration](./media/upgrade-to-latest-generation/creating_restore_point_0.png)

4. Spécifiez un nom pour votre point de restauration.

    ![Nom du point de restauration](./media/upgrade-to-latest-generation/creating_restore_point_1.png)

## <a name="restore-an-active-or-paused-database-using-the-azure-portal"></a>Restaurer une base de données active ou interrompue à l’aide du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez au pool SQL dédié (anciennement SQL DW) à partir duquel vous voulez effectuer la restauration.
3. En haut de la section Vue d’ensemble, sélectionnez **Restaurer**.

    ![ Présentation de la restauration](./media/upgrade-to-latest-generation/restoring_0.png)

4. Sélectionnez **Points de restauration automatiques** ou **Points de restauration définis par l’utilisateur**. Pour l’option Points de restauration définis par l’utilisateur, **sélectionnez un point de restauration défini par l’utilisateur** ou **créez un point de restauration défini par l’utilisateur**. Pour le serveur, sélectionnez **Créer** et choisissez un serveur dans une région géographique prise en charge par Gen2.

    ![Points de restauration automatiques](./media/upgrade-to-latest-generation/restoring_1.png)

## <a name="restore-from-an-azure-geographical-region-using-powershell"></a>Restaurer à partir d’une région géographique Azure à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour récupérer une base de données, utilisez la cmdlet [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

> [!NOTE]
> Vous pouvez effectuer une géorestauration vers Gen2 ! Pour ce faire, spécifiez une valeur ServiceObjectiveName Gen2 (par exemple, DW1000 **c**) comme paramètre facultatif.

1. Ouvrez Windows PowerShell.
2. Connectez-vous à votre compte Azure et répertoriez tous les abonnements associés à votre compte.
3. Sélectionnez l’abonnement contenant la base de données à restaurer.
4. Obtenez la base de données à récupérer.
5. Créez la demande de récupération de la base de données, en indiquant un ServiceObjectiveName Gen2.
6. Vérifiez l’état de la base de données affectée par la géo-restauration.

```Powershell
Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName "<Subscription_name>"

# Get the database you want to recover
$GeoBackup = Get-AzSqlDatabaseGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

# Recover database
$GeoRestoredDatabase = Restore-AzSqlDatabase –FromGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourTargetServer>" -TargetDatabaseName "<NewDatabaseName>" –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName "<YourTargetServiceLevel>" -RequestedServiceObjectiveName "DW300c"

# Verify that the geo-restored database is online
$GeoRestoredDatabase.status
```

> [!NOTE]
> Pour configurer votre base de données une fois la restauration terminée, consultez la page [Configurer votre base de données après récupération](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

La base de données récupérée sera compatible avec le chiffrement transparent des données si la base de données source l’est aussi.

Si vous rencontrez des problèmes avec votre pool SQL dédié, créez une [demande de support](sql-data-warehouse-get-started-create-support-ticket.md) et indiquez « Mise à niveau vers Gen2 » comme cause possible.

## <a name="next-steps"></a>Étapes suivantes

Votre pool SQL dédié (anciennement SQL DW) mis à niveau est en ligne. Pour tirer parti de l’architecture améliorée, consultez [Classes de ressources pour la gestion des charges de travail](resource-classes-for-workload-management.md).
