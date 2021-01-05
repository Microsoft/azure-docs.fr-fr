---
title: Superviser la charge de travail de votre pool SQL dédié à l’aide de vues de gestion dynamique
description: Découvrez comment superviser la charge de travail de votre pool SQL dédié Azure Synapse Analytics et l’exécution de requêtes à l’aide de vues de gestion dynamique.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/24/2020
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: synapse-analytics
ms.openlocfilehash: 1992c3d525fc1f5a098e1969887a752233d47990
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453797"
---
# <a name="monitor-your-azure-synapse-analytics-dedicated-sql-pool-workload-using-dmvs"></a>Superviser la charge de travail de votre pool SQL dédié Azure Synapse Analytics à l’aide de vues de gestion dynamique

Cet article décrit comment utiliser les vues de gestion dynamique (DMV) pour superviser votre charge de travail notamment en analysant l’exécution des requêtes dans le pool SQL.

## <a name="permissions"></a>Autorisations

Pour interroger les vues de gestion dynamique (DMV) de cet article, vous avez besoin de l’autorisation **VIEW DATABASE STATE** ou **CONTROL**. L’autorisation généralement accordée est **VIEW DATABASE STATE**, car elle est beaucoup plus restrictive.

```sql
GRANT VIEW DATABASE STATE TO myuser;
```

## <a name="monitor-connections"></a>Suivi des connexions

Toutes les connexions à votre entrepôt de données sont journalisées dans [sys.dm_pdw_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-sessions-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Cette DMV contient les 10 000 dernières connexions.  L’ID de session (session_id) est la clé principale. Elle est affectée de façon séquentielle pour chaque nouvelle connexion.

```sql
-- Other Active Connections
SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed' and session_id <> session_id();
```

## <a name="monitor-query-execution"></a>Surveillance de l’exécution des rêquetes

Toutes les requêtes exécutées sur le pool SQL sont journalisées dans [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Cette DMV contient les 10 000 dernières requêtes exécutées.  L’ID de requête (request_id) identifie de manière unique chaque requête. Il s’agit de la clé principale pour cette DMV.  L’ID de requête (request_id) est affecté de façon séquentielle pour chaque nouvelle requête et a le préfixe QID, qui signifie ID de requête.  En interrogeant cette DMV pour un ID de session (session_id) donné, vous obtenez toutes les requêtes pour une connexion donnée.

> [!NOTE]
> Les procédures stockées utilisent plusieurs ID de requête.  Les ID de requête sont affectés dans un ordre séquentiel.

Voici les étapes à suivre pour analyser les heures et les plans d’exécution d’une requête spécifique.

### <a name="step-1-identify-the-query-you-wish-to-investigate"></a>ÉTAPE 1 : identifier la requête que vous souhaitez examiner

```sql
-- Monitor active queries
SELECT *
FROM sys.dm_pdw_exec_requests
WHERE status not in ('Completed','Failed','Cancelled')
  AND session_id <> session_id()
ORDER BY submit_time DESC;

-- Find top 10 queries longest running queries
SELECT TOP 10 *
FROM sys.dm_pdw_exec_requests
ORDER BY total_elapsed_time DESC;

```

Dans les résultats de requête précédents, **notez l’ID de la requête** que vous souhaitez examiner.

Les requêtes dont l’état est **Interrompu** peuvent être mises en file d’attente en raison d’un grand nombre de requêtes actives en cours d’exécution. Ces requêtes apparaissent également dans la requête [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) de type UserConcurrencyResourceType. Pour plus d’informations sur les limites de concurrence, consultez [Limites de mémoire et de concurrence](memory-concurrency-limits.md) ou [Classes de ressources pour la gestion des charges de travail](resource-classes-for-workload-management.md). Les requêtes peuvent également attendre d’autres raisons, par exemple des verrouillages d’objets.  Si votre requête est en attente d’une ressource, consultez la rubrique [Examen des requêtes en attente de ressources](#monitor-waiting-queries) plus loin dans cet article.

Pour simplifier la recherche d’une requête dans la table [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), utilisez [LABEL](/sql/t-sql/queries/option-clause-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour attribuer un commentaire à votre requête qui peut être recherché dans l’affichage sys.dm_pdw_exec_requests.

```sql
-- Query with Label
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query')
;

-- Find a query with the Label 'My Query'
-- Use brackets when querying the label column, as it it a key word
SELECT  *
FROM    sys.dm_pdw_exec_requests
WHERE   [label] = 'My Query';
```

### <a name="step-2-investigate-the-query-plan"></a>ÉTAPE 2 : examiner le plan de requête

Utilisez l’ID de requête pour récupérer le plan SQL distribué (DSQL) de la requête dans [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

```sql
-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.

SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID####'
ORDER BY step_index;
```

Lorsqu’un plan DSQL prend plus de temps que prévu, la cause peut être un plan complexe avec de nombreuses étapes DSQL ou une seule étape chronophage.  Si le plan comprend de nombreuses étapes avec plusieurs opérations de déplacement, envisagez d’optimiser vos distributions de table pour réduire le déplacement des données. L’article [Distribution de table](sql-data-warehouse-tables-distribute.md) explique pourquoi les données doivent être déplacées pour résoudre une requête. L’article explique également certaines stratégies de distribution pour réduire le déplacement des données.

Pour examiner les détails d’une seule étape, vérifiez la colonne *operation_type* de l’étape de la requête longue et notez **l’index de l’étape** :

* Passez à l’étape 3 pour les **opérations SQL** : OnOperation, RemoteOperation, ReturnOperation.
* Passez à l’étape 4 pour les **opérations de déplacement des données** : ShuffleMoveOperation, BroadcastMoveOperation, TrimMoveOperation, PartitionMoveOperation, MoveOperation, CopyOperation.

### <a name="step-3-investigate-sql-on-the-distributed-databases"></a>ÉTAPE 3 : examiner SQL dans les bases de données distribuées

Utilisez l’ID de requête et l’index de l’étape pour récupérer des détails dans [sys.dm_pdw_sql_requests](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), qui contient des informations sur l’exécution de l’étape de la requête sur toutes les bases de données distribuées.

```sql
-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID####' AND step_index = 2;
```

Quand l’étape de la requête est en cours d’exécution, [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) peut être utilisé pour récupérer le plan estimé de SQL Server dans le cache du plan SQL Server pour l’étape en cours d’exécution dans une distribution particulière.

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL pool or control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);
```

### <a name="step-4-investigate-data-movement-on-the-distributed-databases"></a>ÉTAPE 4 : examiner le déplacement des données dans les bases de données distribuées

Utilisez l’ID de requête et l’index de l’étape pour récupérer des informations sur une étape de déplacement des données en cours d’exécution sur chaque distribution dans [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

```sql
-- Find information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID####' AND step_index = 2;
```

* Vérifiez la colonne *total_elapsed_time* pour voir si une distribution particulière prend un temps significativement plus important que les autres pour le déplacement des données.
* Pour la distribution longue, consultez la colonne *rows_processed* pour voir si le nombre de lignes déplacées dans le cadre de cette distribution est nettement plus élevé que les autres. Dans ce cas, cette recherche peut indiquer un décalage des données sous-jacentes. L’une des causes de l’asymétrie des données est la distribution sur une colonne avec de nombreuses valeurs NULL (dont les lignes sont toutes placées dans la même distribution). Réduisez les requêtes lentes en évitant la distribution sur ces types de colonnes ou en filtrant votre requête pour éliminer les valeurs NULL lorsque cela est possible. 

Si la requête est en cours d’exécution, vous pouvez utiliser [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour récupérer le plan estimé de SQL Server auprès du cache de plans SQL Serveur pour l’étape SQL en cours d’exécution dans une distribution particulière.

```sql
-- Find the SQL Server estimated plan for a query running on a specific SQL pool Compute or control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(55, 238);
```

<a name="waiting"></a>

## <a name="monitor-waiting-queries"></a>Surveillance des requêtes en attente

Si vous constatez que votre requête ne progresse pas car elle attend une ressource, voici une requête qui vous indique toutes les ressources attendues par une requête.

```sql
-- Find queries
-- Replace request_id with value from Step 1.

SELECT waits.session_id,
      waits.request_id,  
      requests.command,
      requests.status,
      requests.start_time,  
      waits.type,
      waits.state,
      waits.object_type,
      waits.object_name
FROM   sys.dm_pdw_waits waits
   JOIN  sys.dm_pdw_exec_requests requests
   ON waits.request_id=requests.request_id
WHERE waits.request_id = 'QID####'
ORDER BY waits.object_name, waits.object_type, waits.state;
```

Si la requête attend activement des ressources provenant d'une autre requête, l'état affichera **AcquireResources**.  Si la requête possède toutes les ressources requises, l'état sera **Granted**.

## <a name="monitor-tempdb"></a>Surveiller tempdb

Tempdb est utilisé pour stocker des résultats intermédiaires pendant l’exécution des requêtes. Une utilisation intensive de la base de données tempdb peut ralentir les performances des requêtes. Pour chaque DW100c configuré, 399 Go d’espace tempdb sont alloués (soit 3,99 To d’espace tempdb total pour DW1000c).  Voici des conseils à suivre pour superviser l’utilisation de tempdb et la réduire dans vos requêtes.

### <a name="monitoring-tempdb-with-views"></a>Supervision de tempdb avec des vues

Pour superviser l’utilisation de tempdb, commencez par installer la vue [microsoft.vw_sql_requests](https://github.com/Microsoft/sql-data-warehouse-samples/blob/master/solutions/monitoring/scripts/views/microsoft.vw_sql_requests.sql) à partir du dépôt [Microsoft Toolkit for SQL pool](https://github.com/Microsoft/sql-data-warehouse-samples/tree/master/solutions/monitoring). Vous pouvez alors exécuter la requête suivante pour voir l’utilisation de tempdb par nœud pour toutes les requêtes exécutées :

```sql
-- Monitor tempdb
SELECT
    sr.request_id,
    ssu.session_id,
    ssu.pdw_node_id,
    sr.command,
    sr.total_elapsed_time,
    es.login_name AS 'LoginName',
    DB_NAME(ssu.database_id) AS 'DatabaseName',
    (es.memory_usage * 8) AS 'MemoryUsage (in KB)',
    (ssu.user_objects_alloc_page_count * 8) AS 'Space Allocated For User Objects (in KB)',
    (ssu.user_objects_dealloc_page_count * 8) AS 'Space Deallocated For User Objects (in KB)',
    (ssu.internal_objects_alloc_page_count * 8) AS 'Space Allocated For Internal Objects (in KB)',
    (ssu.internal_objects_dealloc_page_count * 8) AS 'Space Deallocated For Internal Objects (in KB)',
    CASE es.is_user_process
    WHEN 1 THEN 'User Session'
    WHEN 0 THEN 'System Session'
    END AS 'SessionType',
    es.row_count AS 'RowCount'
FROM sys.dm_pdw_nodes_db_session_space_usage AS ssu
    INNER JOIN sys.dm_pdw_nodes_exec_sessions AS es ON ssu.session_id = es.session_id AND ssu.pdw_node_id = es.pdw_node_id
    INNER JOIN sys.dm_pdw_nodes_exec_connections AS er ON ssu.session_id = er.session_id AND ssu.pdw_node_id = er.pdw_node_id
    INNER JOIN microsoft.vw_sql_requests AS sr ON ssu.session_id = sr.spid AND ssu.pdw_node_id = sr.pdw_node_id
WHERE DB_NAME(ssu.database_id) = 'tempdb'
    AND es.session_id <> @@SPID
    AND es.login_name <> 'sa'
ORDER BY sr.request_id;
```

Si vous avez une requête qui consomme une grande quantité de mémoire ou si vous avez reçu un message d’erreur à propos de l’allocation de tempdb, le problème peut être dû à une instruction [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) ou [INSERT SELECT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) très lourde en cours d’exécution qui échoue pendant l’opération de déplacement des données finale. Vous pouvez généralement identifier ce problème dans une opération ShuffleMove dans le plan de requête distribuée juste avant la dernière instruction INSERT SELECT.  Utilisez [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour surveiller les opérations ShuffleMove.

Pour atténuer ce problème, vous pouvez diviser l’instruction CTAS ou INSERT SELECT en plusieurs instructions de chargement de sorte que le volume des données ne dépasse pas la limite de tempdb définie à 1 To par nœud. Vous pouvez également mettre à l’échelle votre cluster vers une taille plus grande qui répartit la taille de tempdb sur plus de nœuds en la réduisant sur chaque nœud individuel.

En plus des instructions CTAS et INSERT SELECT, les requêtes volumineuses et complexes s’exécutant avec une mémoire insuffisante peuvent déborder dans tempdb et faire échouer les requêtes.  Envisagez une exécution avec une plus grande [classe de ressources](resource-classes-for-workload-management.md) pour éviter tout débordement dans tempdb.

## <a name="monitor-memory"></a>Surveiller la mémoire

La mémoire peut être la cause racine du ralentissement des performances et des problèmes de mémoire insuffisante. Envisagez la mise à l’échelle de votre entrepôt de données si l’utilisation de la mémoire SQL Server atteint ses limites lors de l’exécution de requêtes.

La requête suivante renvoie des informations sur l’utilisation de la mémoire SQL Server et la saturation de la mémoire par nœud :

```sql
-- Memory consumption
SELECT
  pc1.cntr_value as Curr_Mem_KB,
  pc1.cntr_value/1024.0 as Curr_Mem_MB,
  (pc1.cntr_value/1048576.0) as Curr_Mem_GB,
  pc2.cntr_value as Max_Mem_KB,
  pc2.cntr_value/1024.0 as Max_Mem_MB,
  (pc2.cntr_value/1048576.0) as Max_Mem_GB,
  pc1.cntr_value * 100.0/pc2.cntr_value AS Memory_Utilization_Percentage,
  pc1.pdw_node_id
FROM
-- pc1: current memory
sys.dm_pdw_nodes_os_performance_counters AS pc1
-- pc2: total memory allowed for this SQL instance
JOIN sys.dm_pdw_nodes_os_performance_counters AS pc2
ON pc1.object_name = pc2.object_name AND pc1.pdw_node_id = pc2.pdw_node_id
WHERE
pc1.counter_name = 'Total Server Memory (KB)'
AND pc2.counter_name = 'Target Server Memory (KB)'
```

## <a name="monitor-transaction-log-size"></a>Surveiller la taille du journal des transactions

La requête suivante renvoie la taille du journal des transactions sur chaque distribution. Si l’un des fichiers journaux atteint 160 Go, vous devez envisager la mise à l’échelle de votre instance ou de limiter la taille de votre transaction.

```sql
-- Transaction log size
SELECT
  instance_name as distribution_db,
  cntr_value*1.0/1048576 as log_file_size_used_GB,
  pdw_node_id
FROM sys.dm_pdw_nodes_os_performance_counters
WHERE
instance_name like 'Distribution_%'
AND counter_name = 'Log File(s) Used Size (KB)'
```

## <a name="monitor-transaction-log-rollback"></a>Surveiller la restauration du journal des transactions

Si vos requêtes échouent ou que leur traitement prend beaucoup de temps, vous pouvez vérifier et surveiller si certaines transactions sont en cours de restauration.

```sql
-- Monitor rollback
SELECT
    SUM(CASE WHEN t.database_transaction_next_undo_lsn IS NOT NULL THEN 1 ELSE 0 END),
    t.pdw_node_id,
    nod.[type]
FROM sys.dm_pdw_nodes_tran_database_transactions t
JOIN sys.dm_pdw_nodes nod ON t.pdw_node_id = nod.pdw_node_id
GROUP BY t.pdw_node_id, nod.[type]
```

## <a name="monitor-polybase-load"></a>Surveiller la charge PolyBase

La requête suivante fournit une estimation approximative de la progression de votre charge. La requête affiche uniquement les fichiers en cours de traitement.

```sql

-- To track bytes and files
SELECT
    r.command,
    s.request_id,
    r.status,
    count(distinct input_name) as nbr_files,
    sum(s.bytes_processed)/1024/1024/1024 as gb_processed
FROM
    sys.dm_pdw_exec_requests r
    inner join sys.dm_pdw_dms_external_work s
        on r.request_id = s.request_id
GROUP BY
    r.command,
    s.request_id,
    r.status
ORDER BY
    nbr_files desc,
    gb_processed desc;
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les vues de gestion dynamique, consultez [Vues système](../sql/reference-tsql-system-views.md).
