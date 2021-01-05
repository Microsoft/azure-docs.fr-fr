---
title: Unités DWU (Data Warehouse Unit) pour un pool SQL dédié (anciennement SQL DW)
description: Suggestions pour choisir le nombre idéal de Data Warehouse Units (DWU) en vue d’optimiser les coûts et les performances, et indications pour changer le nombre d’unités.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/22/2019
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 3d9d5d4009ad40eecee26271b726c6a3e9aeb8b6
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459186"
---
# <a name="data-warehouse-units-dwus-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Unités DWU (Data Warehouse Unit) pour un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

Suggestions pour choisir le nombre idéal de Data Warehouse Units (DWU) en vue d’optimiser les coûts et les performances, et indications pour changer le nombre d’unités.

## <a name="what-are-data-warehouse-units"></a>Que sont les Data Warehouse Units ?

Un [pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-overview-what-is.md) représente une collection de ressources analytiques qui sont en cours de provisionnement. Les ressources analytiques sont définies comme une combinaison d’UC, de mémoire et d’E/S.

Ces trois ressources sont regroupées dans des unités d’échelle de calcul appelées Data Warehouse Units (DWU). Une DWU représente une mesure abstraite et standardisée des ressources de calcul et de performances.

Une modification de votre niveau de service change le nombre de DWU disponibles sur le système, qui à son tour ajuste le niveau de performance et le coût de votre système.

Pour un meilleur niveau de performance, vous pouvez augmenter le nombre de DWU. Pour un niveau de performance inférieur, vous pouvez diminuer le nombre de DWU. Les coûts de stockage et de calcul sont facturés séparément. Ainsi, la modification des DWU n’a pas d’effet sur les coûts de stockage.

Les performances des DWU sont basées sur les métriques de charge de travail d’entrepôt de données suivantes :

- Vitesse à laquelle une requête de pool SQL dédié (anciennement SQL DW) standard peut analyser un grand nombre de lignes avant d’effectuer une agrégation complexe. C’est une opération très gourmande en E/S et en UC.
- Vitesse à laquelle un pool SQL dédié (anciennement SQL DW) peut ingérer des données provenant d’objets blob du Stockage Azure ou d’Azure Data Lake. C’est une opération très gourmande en réseau et en UC.
- Rapidité avec laquelle la commande T-SQL [`CREATE TABLE AS SELECT`](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) peut copier une table. Cette opération implique la lecture des données sur le système de stockage, leur distribution entre les nœuds de l’appliance et la réécriture dans le système de stockage. Cette opération est très gourmande en UC, E/S et réseau.

Augmentation du nombre de DWU :

- Modification linéaire des performances du système pour les analyses, les agrégations et les instructions CTAS
- Augmentation du nombre de processus de lecture et d’écriture pour les opérations de chargement PolyBase
- Augmentation du nombre maximal de requêtes simultanées et d’emplacements de concurrence

## <a name="service-level-objective"></a>Objectif de niveau de service

L’Objectif de niveau de service (SLO) est le paramètre d’extensibilité qui détermine le niveau de coût et de performance de votre entrepôt de données. Les niveaux de service pour Gen2 sont mesurés en unités cDWU (compute Data Warehouse Unit), par exemple DW2000C. Les niveaux de service Gen1 sont mesurés en unités DWU (Data Warehouse Unit), par exemple DW2000.

L’objectif de niveau de service (SLO) est le paramètre de scalabilité qui détermine le coût et le niveau de performance de votre pool SQL dédié (anciennement SQL DW). Les niveaux de service pour le pool SQL dédié pool SQL dédié (anciennement SQL DW) Gen2 sont mesurés en unités DWU (Data Warehouse Unit) ; par exemple, DW2000c.

> [!NOTE]
> Un pool SQL dédié (anciennement SQL DW) Gen2 a récemment ajouté des fonctionnalités de mise à l’échelle supplémentaires pour prendre en charge des niveaux de calcul inférieurs à 100 cDWU. Les entrepôts de données Gen1 existants qui ont besoin des niveaux de calcul inférieurs peuvent désormais être mis à niveau vers Gen2 dans les régions actuellement disponibles sans aucun coût supplémentaire.  Si votre région n'est pas encore prise en charge, vous pouvez procéder à une mise à niveau vers une région qui l'est. Pour plus d’informations, consultez [Mettre à niveau vers Gen2](../sql-data-warehouse/upgrade-to-latest-generation.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Dans T-SQL, le paramètre SERVICE_OBJECTIVE détermine le niveau de service et le niveau de performance de votre pool SQL dédié (anciennement SQL DW).

```sql
CREATE DATABASE mySQLDW
(Edition = 'Datawarehouse'
 ,SERVICE_OBJECTIVE = 'DW1000c'
)
;
```

## <a name="performance-tiers-and-data-warehouse-units"></a>Niveaux de performances et Data Warehouse Units

Chaque niveau de performances utilise une unité de mesure légèrement différente pour ses DWU. Cette différence est répercutée sur la facture, car l’unité d’échelle s’applique directement à la facturation.

- Les entrepôts de données Gen1 sont mesurés en unités DWU.
- Les entrepôts de données Gen2 sont mesurés en unités cDWU.

Les DWU et les cDWU prennent en charge la mise à l’échelle du calcul (augmentation ou réduction) et la suspension du calcul lorsque vous n’avez pas besoin d’utiliser l’entrepôt de données. Ces opérations se font toutes à la demande. Gen2 utilise un cache sur disque local sur les nœuds de calcul pour améliorer les performances. Lorsque vous mettez à l’échelle ou suspendez le système, le cache est invalidé, et une période de préchauffage du cache est nécessaire pour pouvoir bénéficier de performances optimales.  

Chaque serveur SQL (par exemple, myserver.database.windows.net) a un quota de [d’unités de transaction de base de données (DTU)](../../azure-sql/database/service-tiers-dtu.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) qui autorise un nombre spécifique d’unités d’entrepôt de données. Pour plus d’informations, consultez les [limites de capacité de gestion de la charge de travail](sql-data-warehouse-service-capacity-limits.md#workload-management).

## <a name="capacity-limits"></a>Limites de capacité

Chaque serveur SQL (par exemple, myserver.database.windows.net) a un quota de [d’unités de transaction de base de données (DTU)](../../sql-database/sql-database-what-is-a-dtu.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) qui autorise un nombre spécifique d’unités d’entrepôt de données. Pour plus d’informations, consultez les [limites de capacité de gestion de la charge de travail](../sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#workload-management).

## <a name="how-many-data-warehouse-units-do-i-need"></a>De combien de DWU ai-je besoin ?

Le nombre idéal de DWU dépend en grande partie de votre charge de travail et de la quantité de données que vous avez chargées dans le système.

Étapes pour rechercher la DWU la mieux adaptée à votre charge de travail :

1. Commencez par sélectionner une DWU plus petite.
2. Surveillez les performances de votre application pendant le test des charges de données dans le système, en observant notamment le nombre de DWU sélectionné.
3. Identifiez des besoins supplémentaires pour les périodes ponctuelles de pics d’activité. Les charges de travail présentant d'importantes variations d'activité doivent être fréquemment mises à l’échelle.

Un pool SQL dédié (anciennement SQL DW) est un système de scale-out qui peut provisionner des quantités importantes de calcul et lancer des requêtes sur une grande quantité de données.

Pour tester ses capacités de mise à l’échelle, surtout avec un nombre élevé de DWU, nous vous recommandons d’effectuer la mise à l’échelle de l’ensemble de données en vous assurant que vous disposez de suffisamment de données pour alimenter les UC. Pour le test de mise à l’échelle, nous recommandons d’utiliser au moins 1 To.

> [!NOTE]
>
> Si les travaux peuvent être fractionnés entre les nœuds de calcul, les performances des requêtes augmentent uniquement avec une parallélisation renforcée. Si vous trouvez que la mise à l’échelle n’altère pas les performances, vous devrez éventuellement modifier la conception de votre table et/ou vos requêtes. Pour des conseils de paramétrage des requêtes, consultez [Gérer les requêtes utilisateur](cheat-sheet.md).

## <a name="permissions"></a>Autorisations

La modification des DWU requiert les autorisations décrites dans [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

Les rôles intégrés Azure, comme Contributeur de base de données SQL et Contributeur de SQL Server, peuvent changer les paramètres des DWU.

## <a name="view-current-dwu-settings"></a>Afficher les paramètres d’unités DWU actuels

Pour afficher le paramètre DWU actuel :

1. Ouvrez l’Explorateur d’objets SQL Server dans Visual Studio.
2. Connectez-vous à la base de données associée au serveur SQL logique.
3. Sélectionnez dans la vue de gestion dynamique sys.database_service_objectives. Voici un exemple :

```sql
SELECT  db.name [Database]
,        ds.edition [Edition]
,        ds.service_objective [Service Objective]
FROM    sys.database_service_objectives   AS ds
JOIN    sys.databases                     AS db ON ds.database_id = db.database_id
;
```

## <a name="change-data-warehouse-units"></a>Modifier les DWU

### <a name="azure-portal"></a>Portail Azure

Pour modifier les DWU :

1. Ouvrez le [portail Azure](https://portal.azure.com), ouvrez votre base de données, puis cliquez sur **Mettre à l’échelle**.

2. Sous **Mettre à l’échelle**, déplacez le curseur vers la gauche ou vers la droite pour modifier le paramètre DWU.

3. Cliquez sur **Enregistrer**. Un message de confirmation s’affiche. Cliquez sur **Oui** pour confirmer ou sur **Non** pour annuler.

#### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour modifier les DWU, utilisez l’applet de commande PowerShell [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase). L'exemple suivant définit l'objectif de niveau de service sur DW1000 pour la base de données MySQLDW hébergée sur le serveur MyServer.

```Powershell
Set-AzSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer" -RequestedServiceObjectiveName "DW1000c"
```

Pour plus d’informations, consultez [Applets de commande PowerShell pour un pool SQL dédié (anciennement SQL DW)](../sql-data-warehouse/sql-data-warehouse-reference-powershell-cmdlets.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="t-sql"></a>T-SQL

Avec T-SQL, vous pouvez afficher les paramètres actuels de DWU, les modifier et consulter la progression.

Pour modifier les unités DWU :

1. Connectez-vous à la base de données MASTER associée à votre serveur.
2. Utilisez l’instruction TSQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). L’exemple suivant définit l'objectif de niveau de service sur DW1000c pour la base de données MySQLDW.

```Sql
ALTER DATABASE MySQLDW
MODIFY (SERVICE_OBJECTIVE = 'DW1000c')
;
```

### <a name="rest-apis"></a>API REST

Pour modifier les DWU, utilisez l’API REST [Créer ou mettre à jour une base de données](/rest/api/sql/databases/createorupdate). L’exemple suivant définit l'objectif de niveau de service sur DW1000c pour la base de données MySQLDW hébergée sur le serveur MyServer. Le serveur est un groupe de ressources Azure appelé ResourceGroup1.

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}?api-version=2014-04-01-preview HTTP/1.1
Content-Type: application/json; charset=UTF-8

{
    "properties": {
        "requestedServiceObjectiveName": DW1000
    }
}
```

Pour obtenir des exemples d’API REST supplémentaires, consultez [API REST pour un pool SQL dédié (anciennement SQL DW)](../sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="check-status-of-dwu-changes"></a>Vérifier l’état des modifications de DWU

Les modifications de DWU peuvent prendre plusieurs minutes. Si vous effectuez une mise à l’échelle automatique, envisagez d’implémenter une logique pour vous assurer que certaines opérations sont bien terminées avant de passer à une autre action.

Nous vous conseillons de vérifier l’état de la base de données en différents points de terminaison afin d’implémenter correctement l’automatisation. Si le portail vous informe de la fin d’une opération et de l’état actuel des bases de données, il ne vous permet pas de programmer la vérification de l’état.

Vous ne pouvez pas vérifier l’état de la base de données pour les opérations de montée en puissance avec le portail Azure.

Pour vérifier l’état des modifications de DWU :

1. Connectez-vous à la base de données MASTER associée à votre serveur.
2. Envoyez la requête suivante pour vérifier l’état de la base de données.

```sql
SELECT    *
FROM      sys.databases
;
```

1. Envoyez la requête suivante pour vérifier l’état de l’opération.

    ```sql
    SELECT    *
    FROM      sys.dm_operation_status
    WHERE     resource_type_desc = 'Database'
    AND       major_resource_id = 'MySQLDW'
    ;
    ```

Cette vue de gestion dynamique (DMV) retourne des informations concernant différentes opérations de gestion sur votre pool SQL dédié (anciennement SQL DW), comme l’opération et son état, qui est IN_PROGRESS ou COMPLETED.

## <a name="the-scaling-workflow"></a>Flux de travail de mise à l’échelle

Lorsque vous démarrez une opération de mise à l’échelle, le système arrête tout d’abord toutes les sessions ouvertes, ce qui annule toutes les transactions en cours pour garantir un état cohérent. La mise à l’échelle intervient uniquement une fois la restauration des transactions effectuée.  

- Pour une opération de montée en puissance, le système détache tous les nœuds de calcul, approvisionne le calcul supplémentaire, puis le réassocie à la couche de stockage.
- Pour une opération de descente en puissance, le système détache tous les nœuds de calcul, puis rattache uniquement les nœuds nécessaire à la couche de stockage.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la gestion des performances, consultez [Classes de ressources pour la gestion de la charge de travail](resource-classes-for-workload-management.md) et [Limites de mémoire et de concurrence](memory-concurrency-limits.md).
