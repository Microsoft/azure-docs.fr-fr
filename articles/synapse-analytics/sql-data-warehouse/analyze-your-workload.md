---
title: Analyser votre charge de travail pour un pool SQL dédié
description: Techniques d’analyse des priorités de requête pour un pool SQL dédié dans Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 14c3ad30bac7cec4c11822d825323bb9db2ba440
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454540"
---
# <a name="analyze-your-workload-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Analyser votre charge de travail pour un pool SQL dédié dans Azure Synapse Analytics

Techniques d’analyse des priorités de votre charge de travail de pool SQL dédié dans Azure Synapse Analytics. 

## <a name="resource-classes"></a>Classes de ressources

Un pool SQL dédié fournit des classes de ressources pour attribuer des ressources système aux requêtes.  Pour plus d’informations sur les classes de ressources, consultez [Classes de ressources et gestion des charges de travail](resource-classes-for-workload-management.md).  Les requêtes attendront si la classe de ressources affectée à une requête nécessite plus de ressources que de ressources actuellement disponibles.

## <a name="queued-query-detection-and-other-dmvs"></a>Détection des requêtes en file d’attente et autres vues de gestion dynamique

Vous pouvez utiliser la DMV `sys.dm_pdw_exec_requests` pour identifier les requêtes en attente dans une file d’attente de concurrence. Les requêtes en attente d’emplacement de concurrence ont le statut **suspendu**.

```sql
SELECT  r.[request_id]                           AS Request_ID
,       r.[status]                               AS Request_Status
,       r.[submit_time]                          AS Request_SubmitTime
,       r.[start_time]                           AS Request_StartTime
,       DATEDIFF(ms,[submit_time],[start_time])  AS Request_InitiateDuration_ms
,       r.resource_class                         AS Request_resource_class
FROM    sys.dm_pdw_exec_requests r
;
```

Vous pouvez afficher les rôles de gestion des charges de travail avec `sys.database_principals`.

```sql
SELECT  ro.[name]           AS [db_role_name]
FROM    sys.database_principals ro
WHERE   ro.[type_desc]      = 'DATABASE_ROLE'
AND     ro.[is_fixed_role]  = 0
;
```

La requête suivante montre le rôle auquel est affecté chaque utilisateur.

```sql
SELECT  r.name AS role_principal_name
,       m.name AS member_principal_name
FROM    sys.database_role_members rm
JOIN    sys.database_principals AS r            ON rm.role_principal_id      = r.principal_id
JOIN    sys.database_principals AS m            ON rm.member_principal_id    = m.principal_id
WHERE   r.name IN ('mediumrc','largerc','xlargerc')
;
```

Un pool SQL dédié a les types d’attente suivants :

* **LocalQueriesConcurrencyResourceType** : requêtes qui figurent à l’extérieur de l’infrastructure d’emplacements de concurrence. Les requêtes DMV et les fonctions système telles que `SELECT @@VERSION` sont des exemples de requête locale.
* **UserConcurrencyResourceType** : requêtes qui figurent à l’intérieur de l’infrastructure d’emplacements de concurrence. Les requêtes exécutées sur des tables d’utilisateurs finaux sont des exemples de requêtes qui doivent utiliser ce type de ressource.
* **DmsConcurrencyResourceType** : attentes résultant d’opérations de déplacement des données.
* **BackupConcurrencyResourceType** : cette attente indique qu’une base de données est en cours de sauvegarde. La valeur maximale de ce type de ressource est égale à 1. Si plusieurs sauvegardes ont été demandées en même temps, les autres sont placées en file d’attente. En général, nous recommandons une durée minimale de 10 minutes entre deux instantanés consécutifs.

Vous pouvez utiliser la DMV `sys.dm_pdw_waits` pour connaître les ressources attendues par une demande.

```sql
SELECT  w.[wait_id]
,       w.[session_id]
,       w.[type]                                           AS Wait_type
,       w.[object_type]
,       w.[object_name]
,       w.[request_id]
,       w.[request_time]
,       w.[acquire_time]
,       w.[state]
,       w.[priority]
,       SESSION_ID()                                       AS Current_session
,       s.[status]                                         AS Session_status
,       s.[login_name]
,       s.[query_count]
,       s.[client_id]
,       s.[sql_spid]
,       r.[command]                                        AS Request_command
,       r.[label]
,       r.[status]                                         AS Request_status
,       r.[submit_time]
,       r.[start_time]
,       r.[end_compile_time]
,       r.[end_time]
,       DATEDIFF(ms,r.[submit_time],r.[start_time])        AS Request_queue_time_ms
,       DATEDIFF(ms,r.[start_time],r.[end_compile_time])   AS Request_compile_time_ms
,       DATEDIFF(ms,r.[end_compile_time],r.[end_time])     AS Request_execution_time_ms
,       r.[total_elapsed_time]
FROM    sys.dm_pdw_waits w
JOIN    sys.dm_pdw_exec_sessions s  ON w.[session_id] = s.[session_id]
JOIN    sys.dm_pdw_exec_requests r  ON w.[request_id] = r.[request_id]
WHERE    w.[session_id] <> SESSION_ID()
;
```

La DMV `sys.dm_pdw_resource_waits` affiche les informations d’attente pour une requête donnée. Le temps d’attente d’une ressource mesure le temps d’attente avant que la ressource soit fournie. Le temps d’attente du signal désigne le temps nécessaire pour que les serveurs SQL sous-jacents planifient la requête dans l’UC.

```sql
SELECT  [session_id]
,       [type]
,       [object_type]
,       [object_name]
,       [request_id]
,       [request_time]
,       [acquire_time]
,       DATEDIFF(ms,[request_time],[acquire_time])  AS acquire_duration_ms
,       [concurrency_slots_used]                    AS concurrency_slots_reserved
,       [resource_class]
,       [wait_id]                                   AS queue_position
FROM    sys.dm_pdw_resource_waits
WHERE    [session_id] <> SESSION_ID()
;
```

Vous pouvez aussi utiliser la DMV `sys.dm_pdw_resource_waits` pour calculer la quantité d’emplacements de concurrence accordée.

```sql
SELECT  SUM([concurrency_slots_used]) as total_granted_slots
FROM    sys.[dm_pdw_resource_waits]
WHERE   [state]           = 'Granted'
AND     [resource_class] is not null
AND     [session_id]     <> session_id()
;
```

Vous pouvez utiliser la DMV `sys.dm_pdw_wait_stats` pour l’analyse des tendances historiques des attentes.

```sql
SELECT   w.[pdw_node_id]
,        w.[wait_name]
,        w.[max_wait_time]
,        w.[request_count]
,        w.[signal_time]
,        w.[completed_count]
,        w.[wait_time]
FROM    sys.dm_pdw_wait_stats w
;
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la gestion de la sécurité et des utilisateurs de base de données, consultez [Sécuriser un pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-overview-manage-security.md). Pour plus d’informations sur la façon dont des classes de ressources plus importantes peuvent améliorer la qualité des index columnstore en cluster, consultez [Reconstruire des index pour améliorer la qualité de segment](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality).
