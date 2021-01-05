---
title: Gestion des ressources dans les pools élastiques denses
description: Gérez les ressources de calcul dans des pools élastiques Azure SQL Database englobant de nombreuses bases de données.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: dimitri-furman
ms.author: dfurman
ms.reviewer: sstein
ms.date: 09/16/2020
ms.openlocfilehash: 1398745a485b57c3a739c25dbca77d7b21f19758
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792155"
---
# <a name="resource-management-in-dense-elastic-pools"></a>Gestion des ressources dans les pools élastiques denses
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Les [pools élastiques](./elastic-pool-overview.md) Azure SQL Database sont une solution économique pour gérer de nombreuses bases de données avec diverses utilisations des ressources. Toutes les bases de données d’un pool élastique partagent la même allocation de ressources, telles que l’UC, la mémoire, les threads de travail, l’espace de stockage et tempdb, en partant du principe que **seul un sous-ensemble de bases de données dans le pool utilisera des ressources de calcul à un moment donné** . Cette hypothèse permet aux pools élastiques d’être rentables. Au lieu de payer pour toutes les ressources dont chaque base de données individuelle peut éventuellement avoir besoin, les clients paient un ensemble de ressources bien plus petit, partagé entre toutes les bases de données du pool.

## <a name="resource-governance"></a>Gouvernance des ressources

Le partage de ressources nécessite que le système contrôle minutieusement l’utilisation des ressources pour réduire l’effet de « voisinage bruyant », où une base de données avec une consommation de ressources élevée perturbe les autres bases de données dans le même pool élastique. En même temps, le système doit fournir suffisamment de ressources pour que des fonctionnalités telles que la haute disponibilité et la récupération d’urgence (HADR), la sauvegarde et la restauration, la surveillance, le Magasin des requêtes, le réglage automatique et autres fonctionnent de manière fiable.

Azure SQL Database atteint ces objectifs à l'aide de plusieurs mécanismes de gouvernance des ressources, notamment les [objets de travail](/windows/win32/procthread/job-objects) Windows pour la gouvernance des ressources au niveau du processus, les [Outils de gestion de ressources pour serveur de fichiers (FSRM)](/windows-server/storage/fsrm/fsrm-overview) Windows pour la gestion des quotas de stockage, ainsi qu'une version modifiée et étendue de SQL Server [Resource Governor](/sql/relational-databases/resource-governor/resource-governor) pour implémenter la gouvernance des ressources dans SQL Database.

Le principal objectif de conception des pools élastiques est d’être rentable. C’est la raison pour laquelle le système autorise intentionnellement les clients à créer des pools _denses_ , c’est-à-dire des pools dont le nombre de bases de données approche ou atteint le maximum autorisé, mais avec une allocation modérée des ressources de calcul. Pour la même raison, le système ne réserve pas toutes les ressources potentiellement nécessaires pour ses processus internes, mais autorise le partage des ressources entre les processus internes et les charges de travail utilisateur.

Cette approche permet aux clients d’utiliser des pools élastiques denses afin d’obtenir des performances adéquates et de réaliser des économies substantielles. Toutefois, si la charge de travail sur de nombreuses bases de données d’un pool dense est suffisamment intense, la contention des ressources devient importante. La contention des ressources réduit le niveau de performance des charges de travail utilisateur et peut avoir un impact négatif sur les processus internes.

> [!IMPORTANT]
> Dans des pools denses comptant de nombreuses bases de données actives, il n’est parfois pas possible d’augmenter le nombre de bases de données composant le pool jusqu’aux valeurs maximales documentées pour les pools élastiques [DTU](resource-limits-dtu-elastic-pools.md) et [vCore](resource-limits-vcore-elastic-pools.md).
>
> Le nombre de bases de données pouvant être placées dans des pools denses sans occasionner de problèmes de contention de ressources et de performances dépend du nombre de bases de données actives simultanément et de la consommation de ressources par les charges de travail utilisateur dans chaque base de données. Ce nombre peut changer au fil du temps à mesure que changent les charges de travail utilisateur.

Quand une contention de ressources se produit dans un pool très dense, les clients peuvent choisir une ou plusieurs des actions suivantes pour les atténuer :

- Réglez la charge de travail de requête de façon à réduire la consommation de ressources ou à répartir celle-ci sur plusieurs bases de données au fil du temps.
- Réduisez la densité du pool en déplaçant certaines bases de données vers un autre pool ou en les rendant autonomes.
- Montez le pool en puissance pour obtenir davantage de ressources.

Pour obtenir des suggestions sur la façon d’implémenter les deux dernières actions, consultez [Recommandations opérationnelles](#operational-recommendations) plus loin dans cet article. La réduction de la contention des ressources profite à la fois aux charges de travail utilisateur et aux processus internes et permet au système de conserver le niveau de service attendu de manière fiable.

## <a name="monitoring-resource-consumption"></a>Surveillance de la consommation des ressources

Pour éviter une détérioration des performances en raison d’une contention des ressources, les clients qui utilisent des pools élastiques denses doivent surveiller de façon proactive la consommation des ressources et prendre des mesures en temps opportun si l’augmentation de la contention des ressources commence à perturber les charges de travail. La surveillance continue est importante, car l’utilisation des ressources dans un pool change au fil du temps, en raison des modifications apportées à la charge de travail utilisateur, des modifications des volumes et de la distribution de données, des modifications de la densité du pool et des modifications du service Azure SQL Database.

Azure SQL Database fournit plusieurs mesures pertinentes pour ce type de surveillance. Le dépassement de la valeur moyenne recommandée pour chaque métrique indique une contention des ressources dans le pool, qui doit être résolu à l’aide de l’une des actions mentionnées précédemment.

|Nom de métrique|Description|Valeur moyenne recommandée|
|----------|--------------------------------|------------|
|`avg_instance_cpu_percent`|Utilisation de l'UC du processus SQL associée à un pool élastique, telle que mesurée par le système d'exploitation sous-jacent. Disponible dans l’affichage [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) de chaque base de données et dans l’affichage [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) de la base de données `master`. Cette métrique est également émise pour Azure Monitor, où elle est [nommée](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) `sqlserver_process_core_percent`, et peut être affichée dans le portail Azure. Cette valeur est la même pour toutes les bases de données du même pool élastique.|En dessous de 70 %. De courts pics occasionnels allant jusqu’à 90 % peuvent être acceptables.|
|`max_worker_percent`|Utilisation du [thread de travail]( https://docs.microsoft.com/sql/relational-databases/thread-and-task-architecture-guide). Fournie pour chaque base de données du pool, ainsi que pour le pool lui-même. Il existe différentes limites sur le nombre de threads de travail au niveau de la base de données et au niveau du pool. Il est donc recommandé de surveiller cette métrique aux deux niveaux. Disponible dans l’affichage [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) de chaque base de données et dans l’affichage [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) de la base de données `master`. Cette métrique est également émise pour Azure Monitor, où elle est [nommée](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) `workers_percent`, et peut être affichée dans le portail Azure.|En dessous de 80 %. Des pics allant jusqu’à 100 % entraînent l’échec des tentatives de connexion et des requêtes.|
|`avg_data_io_percent`|Utilisation de l’IOPS pour les E/S physiques en lecture et écriture. Fournie pour chaque base de données du pool, ainsi que pour le pool lui-même. Il existe différentes limites sur le nombre d’IOPS au niveau de la base de données et au niveau du pool. Il est donc recommandé de surveiller cette métrique aux deux niveaux. Disponible dans l’affichage [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) de chaque base de données et dans l’affichage [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) de la base de données `master`. Cette métrique est également émise pour Azure Monitor, où elle est [nommée](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) `physical_data_read_percent`, et peut être affichée dans le portail Azure.|En dessous de 80 %. De courts pics occasionnels allant jusqu’à 100 % peuvent être acceptables.|
|`avg_log_write_percent`|Utilisation du débit pour les E/S en écriture du journal des transactions. Fournie pour chaque base de données du pool, ainsi que pour le pool lui-même. Il existe différentes limites sur le débit des journaux au niveau de la base de données et au niveau du pool. Il est donc recommandé de surveiller cette métrique aux deux niveaux. Disponible dans l’affichage [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) de chaque base de données et dans l’affichage [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) de la base de données `master`. Cette métrique est également émise pour Azure Monitor, où elle est [nommée](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) `log_write_percent`, et peut être affichée dans le portail Azure. Lorsque cette mesure est proche de 100 %, toutes les modifications de base de données (instructions INSERT, UPDATE, DELETE, MERGE, SELECT… INTO, BULK INSERT, etc.) seront plus lentes.|En dessous de 90 %. De courts pics occasionnels allant jusqu’à 100 % peuvent être acceptables.|
|`oom_per_second`|Taux d’erreurs de mémoire insuffisante (OOM) dans un pool élastique, qui est un indicateur de sollicitation de la mémoire. Disponible dans l’affichage [sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database). Consultez [Exemples](#examples) pour obtenir un exemple de requête afin de calculer cette mesure.|0|
|`avg_storage_percent`|Espace de stockage total utilisé par les données dans toutes les bases de données au sein d’un pool élastique. N’inclut pas l’espace vide dans les fichiers de base de données. Disponible dans l’affichage [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) de la base de données `master`. Cette métrique est également émise pour Azure Monitor, où elle est [nommée](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) `storage_percent`, et peut être affichée dans le portail Azure.|En dessous de 80 %. Peut approcher 100 % pour les pools sans croissance des données.|
|`avg_allocated_storage_percent`|Espace de stockage total utilisé par les fichiers de base données stockés dans toutes les bases de données au sein d’un pool élastique. Inclut un espace vide dans les fichiers de base de données. Disponible dans l’affichage [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) de la base de données `master`. Cette métrique est également émise pour Azure Monitor, où elle est [nommée](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) `allocated_data_storage_percent`, et peut être affichée dans le portail Azure.|En dessous de 90 %. Peut approcher 100 % pour les pools sans croissance des données.|
|`tempdb_log_used_percent`|Utilisation de l’espace du journal des transactions dans la base de données `tempdb`. Même si les objets temporaires créés dans une base de données ne sont pas visibles dans d’autres bases de données du même pool élastique, `tempdb` est une ressource partagée pour toutes les bases de données du même pool. Une transaction durable ou orpheline dans `tempdb` démarrée à partir d’une base de données du pool peut consommer une grande partie du journal des transactions et provoquer des échecs pour les requêtes dans d’autres bases de données du même pool. Dérivé des affichages [sys.dm_db_log_space_usage](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-log-space-usage-transact-sql)et [sys.database_files](/sql/relational-databases/system-catalog-views/sys-database-files-transact-sql) Cette métrique est également émise pour Azure Monitor et peut être affichée dans le Portail Azure. Consultez [Exemples](#examples) pour obtenir un exemple de requête afin de retourner la valeur actuelle de cette mesure.|En dessous de 50 %. Des pics occasionnels allant jusqu’à 80 % sont acceptables.|
|||

En plus de ces métriques, Azure SQL Database fournit un affichage qui retourne les limites réelles de la gouvernance des ressources, ainsi que des affichages supplémentaires qui retournent des statistiques d’utilisation des ressources au niveau de la liste de ressources partagées et au niveau du groupe de charge de travail.

|Nom de l’affichage|Description|  
|-----------------|--------------------------------|  
|[sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database)|Retourne les paramètres réels de configuration et de capacité utilisés par les mécanismes de gouvernance des ressources dans la base de données ou le pool élastique actuel.|
|[sys.dm_resource_governor_resource_pools](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-transact-sql)|Retourne des informations sur l’état actuel, la configuration actuelle et les statistiques cumulées des listes de ressources partagées.|
|[sys.dm_resource_governor_workload_groups](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-transact-sql)|Retourne les statistiques cumulées et la configuration actuelle du groupe de charge de travail. Cet affichage peut être joint avec ys.dm_resource_governor_resource_pools sur la colonne `pool_id` pour obtenir des informations sur la liste de ressources partagées.|
|[sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database)|Retourne les statistiques d’utilisation de la liste de ressources partagées pour les 32 dernières minutes. Chaque ligne représente un intervalle de 20 secondes. Les colonnes `delta_` retournent la modification de chaque statistique au cours de l’intervalle.|
|[sys.dm_resource_governor_workload_groups_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-history-ex-azure-sql-database)|Retourne les statistiques d’utilisation du groupe de charge de travail pour les 32 dernières minutes. Chaque ligne représente un intervalle de 20 secondes. Les colonnes `delta_` retournent la modification de chaque statistique au cours de l’intervalle.|
|||

Ces affichages peuvent être utilisés pour surveiller l’utilisation des ressources et résoudre la contention des ressources en temps quasi-réel. La charge de travail utilisateur sur les réplicas primaires et secondaires accessibles en lecture, y compris les géoréplications, est classée dans la liste de ressources partagées `SloSharedPool1` et dans le groupe de charges de travail `UserPrimaryGroup.DBId[N]`, où `N` correspond à la valeur de l’ID de base de données.

Outre la surveillance de l’utilisation actuelle des ressources, les clients qui utilisent des pools denses peuvent gérer les données d’utilisation des ressources historiques dans un magasin de données distinct. Ces données peuvent être utilisées dans une analyse prédictive pour gérer de manière proactive l’utilisation des ressources en fonction des tendances historiques et saisonnières.

## <a name="operational-recommendations"></a>Recommandations opérationnelles

**Laissez une marge de manœuvre suffisante aux ressources** . En cas de détérioration des performances et de contention des ressources, l’atténuation peut impliquer le déplacement de certaines bases de données hors du pool élastique concerné ou la montée en puissance du pool, comme indiqué plus haut. Toutefois, ces actions nécessitent des ressources de calcul supplémentaires pour s’exécuter. En particulier, pour les pools Premium et Critique pour l’entreprise, ces actions nécessitent le transfert de toutes les données pour les bases de données déplacées, ou pour toutes les bases de données du pool élastique si le pool est monté en puissance. Le transfert de données est une opération durable et gourmande en ressources. Si le pool est déjà soumis à une forte sollicitation des ressources, l’opération d’atténuation proprement dite détériorera encore davantage les performances. Dans les cas extrêmes, il peut être impossible de résoudre la contention de ressources via la montée en puissance du pool ou le déplacement de bases de données, car les ressources requises ne sont pas disponibles. Dans ce cas, la réduction temporaire de la charge de travail des requêtes sur le pool élastique concerné peut être la seule solution.

Les clients qui utilisent des pools denses doivent surveiller étroitement les tendances d’utilisation des ressources comme décrit précédemment et prendre des mesures d’atténuation tant que les métriques restent dans les plages recommandées et qu’il existe toujours suffisamment de ressources dans le pool élastique.

L’utilisation des ressources dépend de plusieurs facteurs qui changent au fil du temps pour chaque base de données et chaque pool élastique. L’obtention d’un rapport prix/performances optimal dans les pools denses requiert une surveillance et un rééquilibrage continus, c’est-à-dire le déplacement des bases de données des pools très utilisés vers des pools qui le sont moins et la création de pools le cas échéant pour répondre à une charge de travail accrue.

**Ne déplacez pas les bases de données « chaudes »** . Si la contention de ressources au niveau du pool est principalement due à un petit nombre de bases de données fortement utilisées, il peut être tentant de déplacer ces bases de données vers un pool moins utilisé ou de les rendre autonomes. Toutefois, il est déconseillé d’effectuer cette opération alors qu’une base de données reste fortement utilisée, car l’opération de déplacement détériorera davantage les performances, à la fois pour la base de données en cours de déplacement et pour l’ensemble du pool. Au lieu de cela, attendez plutôt que le taux d’utilisation diminue ou déplacez les bases de données moins utilisées pour alléger la sollicitation des ressources au niveau du pool. Toutefois, le déplacement de bases de données ayant une utilisation très faible ne fournit aucun avantage dans ce cas, car il ne réduit pas de façon significative l’utilisation des ressources au niveau du pool.

**Créez des bases de données dans un pool de « quarantaine »** . Dans les scénarios où des bases de données sont créées fréquemment, telles que les applications utilisant le modèle de locataire par base de données, il existe un risque qu’une nouvelle base de données placée dans un pool élastique existant consomme de manière inattendue des ressources importantes et perturbe les autres bases de données et les processus internes du pool. Pour atténuer ce risque, créez un pool de « quarantaine » distinct avec une allocation généreuse de ressources. Utilisez ce pool pour les nouvelles bases de données ayant des modèles de consommation de ressources encore inconnus. Lorsqu’une base de données est restée dans ce pool pendant un cycle d’activité, par exemple une semaine ou un mois, et que sa consommation de ressources est connue, elle peut être déplacée vers un pool ayant une capacité suffisante pour prendre en charge cette utilisation supplémentaire des ressources.

**Surveiller l’espace utilisé et alloué** . Quand l’espace du pool alloué (taille totale de tous les fichiers de base de données stockés pour toutes les bases de données dans un pool) atteint la taille de pool maximale, des erreurs d’espace insuffisant peuvent se produire. Si l’espace alloué est en hausse et en passe d’atteindre la taille de pool maximale, les options d’atténuation sont les suivantes :
- déplacer des bases de données hors du pool pour réduire l’espace total alloué ;
- [réduire](file-space-manage.md) la taille des fichiers de base de données pour diminuer l’espace alloué vide dans les fichiers ;
- effectuer un scale-up du pool vers un objectif de service avec une taille de pool maximale supérieure.

Si l’espace du pool utilisé (taille totale des données de toutes les bases de données d’un pool, à l’exclusion de l’espace vide dans les fichiers) est en hausse et en passe d’atteindre la taille de pool maximale, les options d’atténuation sont les suivantes :
- déplacer des bases de données hors du pool pour réduire l’espace total utilisé ;
- déplacer (archiver) des données hors de la base de données ou supprimer les données qui ne sont plus nécessaires ;
- implémenter la [compression des données](/sql/relational-databases/data-compression/data-compression) ;
- effectuer un scale-up du pool vers un objectif de service avec une taille de pool maximale supérieure.

**Évitez les serveurs trop denses** . Azure SQL Database [prend en charge](./resource-limits-logical-server.md) un maximum de 5 000 bases de données par serveur. Les clients qui utilisent des pools élastiques ayant des milliers de bases de données peuvent envisager de placer plusieurs pools élastiques sur un seul serveur, avec le nombre total de bases de données allant jusqu’à la limite prise en charge. Toutefois, les serveurs englobant plusieurs milliers de bases de données constituent des défis opérationnels. Les opérations qui requièrent l’énumération de toutes les bases de données sur un serveur, par exemple l’affichage de bases de données dans le portail, seront plus lentes. Les erreurs opérationnelles, telles que la modification incorrecte des connexions au niveau du serveur ou des règles de pare-feu, perturbent un plus grand nombre de bases de données. La suppression accidentelle du serveur nécessite une assistance de la part du Support Microsoft afin de récupérer les bases de données sur le serveur supprimé, ce qui entraîne une interruption prolongée pour toutes les bases de données concernées.

Nous vous recommandons de limiter les bases de données par serveur à un nombre inférieur à la valeur maximale prise en charge. Dans de nombreux scénarios, l’utilisation de 1 000 à 2 000 bases de données par serveur est optimale. Afin de réduire les risques de suppression accidentelle d'un serveur, nous vous recommandons de placer un [verrou de suppression](../../azure-resource-manager/management/lock-resources.md) sur le serveur ou son groupe de ressources.

Dans le passé, certains scénarios impliquant le déplacement de bases de données dans, hors ou entre des pools élastiques d'un même serveur étaient plus rapides que le déplacement de bases de données entre des serveurs. Actuellement, tous les déplacements de base de données s'exécutent à la même vitesse, quels que soient le serveur source et le serveur de destination.

## <a name="examples"></a>Exemples

### <a name="monitoring-memory-utilization"></a>Surveillance de l’utilisation de la mémoire

Cette requête calcule la métrique `oom_per_second` pour chaque liste de ressources partagées, au cours des dernières 32 minutes. Cette requête peut être exécutée dans n’importe quelle base de données d’un pool élastique.

```sql
SELECT pool_id,
       name AS resource_pool_name,
       IIF(name LIKE 'SloSharedPool%' OR name LIKE 'UserPool%', 'user', 'system') AS resource_pool_type,
       SUM(CAST(delta_out_of_memory_count AS decimal))/(SUM(duration_ms)/1000.) AS oom_per_second
FROM sys.dm_resource_governor_resource_pools_history_ex
GROUP BY pool_id, name
ORDER BY pool_id;
```

### <a name="monitoring-tempdb-log-space-utilization"></a>Surveillance de l’utilisation de l’espace du journal `tempdb`

Cette requête retourne la valeur actuelle de la métrique `tempdb_log_used_percent`, en présentant l’utilisation relative du journal des transactions `tempdb` par rapport à sa taille maximale autorisée. Cette requête peut être exécutée dans n’importe quelle base de données d’un pool élastique.

```sql
SELECT (lsu.used_log_space_in_bytes / df.log_max_size_bytes) * 100 AS tempdb_log_space_used_percent
FROM tempdb.sys.dm_db_log_space_usage AS lsu
CROSS JOIN (
           SELECT SUM(CAST(max_size AS bigint)) * 8 * 1024. AS log_max_size_bytes
           FROM tempdb.sys.database_files
           WHERE type_desc = N'LOG'
           ) AS df
;
```

## <a name="next-steps"></a>Étapes suivantes

- Une introduction aux pools élastiques est disponible dans le document [Les pools élastiques vous aident à gérer et à mettre à l'échelle plusieurs bases de données dans Azure SQL Database](./elastic-pool-overview.md).
- Pour plus d’informations sur le réglage des charges de travail de requête afin de réduire l’utilisation des ressources, consultez [Surveillance et réglage]( https://docs.microsoft.com/azure/sql-database/sql-database-monitoring-tuning-index) et [Surveillance et réglage des performances](./monitor-tune-overview.md).