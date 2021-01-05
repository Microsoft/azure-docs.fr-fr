---
title: Diagnostics des performances dans Hyperscale
description: Cet article explique comment résoudre les problèmes de performances Hyperscale dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: seo-lt-2019 sqldbrb=1
ms.topic: troubleshooting
author: denzilribeiro
ms.author: denzilr
ms.reviewer: sstein
ms.date: 10/18/2019
ms.openlocfilehash: ed31ff5d77b258d141a77fc174c2d5452adf7d01
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791713"
---
# <a name="sql-hyperscale-performance-troubleshooting-diagnostics"></a>Diagnostics de résolution des problèmes de performances Hyperscale SQL
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Pour résoudre les problèmes de performances dans une base de données Hyperscale, les [méthodologies d’optimisation des performances générales](monitor-tune-overview.md) sur le nœud de calcul Azure SQL Database sont le point de départ d’une investigation des performances. Cependant, compte tenu de l’[architecture distribuée](service-tier-hyperscale.md#distributed-functions-architecture) d’Hyperscale, des diagnostics supplémentaires ont été ajoutés pour faciliter les opérations. Cet article décrit les données de diagnostic propres à Hyperscale.

## <a name="log-rate-throttling-waits"></a>Attentes liées à la limitation du taux de journalisation

Chaque niveau de service Azure SQL Database présente des limites de taux de génération de journaux qui sont appliquées via la [gouvernance du taux de journalisation](resource-limits-logical-server.md#transaction-log-rate-governance). Dans Hyperscale, la limite de génération de journaux actuellement définie est de 100 Mo/s, quel que soit le niveau de service. Cependant, le taux de génération de journaux doit parfois être limité sur le réplica de calcul principal pour respecter les contrats SLA de récupération. Cette limitation se produit quand un [serveur de pages ou un autre réplica de calcul ](service-tier-hyperscale.md#distributed-functions-architecture) tarde à appliquer les nouveaux enregistrements de journal du service de journal.

Les types d’attente suivants (dans [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql/)) décrivent les raisons pour lesquelles le taux de journalisation peut être limité sur le réplica de calcul principal :

|Type d’attente    |Description                         |
|-------------          |------------------------------------|
|RBIO_RG_STORAGE        | Se produit quand le taux de génération de journal sur le nœud de calcul principal d’une base de données Hyperscale est limité en raison d’une consommation de journal retardée sur le ou les serveurs de pages.         |
|RBIO_RG_DESTAGE        | Se produit quand le taux de génération de journal sur le nœud de calcul d’une base de données Hyperscale est limité en raison d’une consommation de journal retardée par le stockage de journal à long terme.         |
|RBIO_RG_REPLICA        | Se produit quand le taux de génération de journal sur le nœud de calcul d’une base de données Hyperscale est limité en raison d’une consommation de journal retardée par le ou les réplicas secondaire accessibles en lecture.         |
|RBIO_RG_LOCALDESTAGE   | Se produit quand le taux de génération de journal sur le nœud de calcul d’une base de données Hyperscale est limité en raison d’une consommation de journal retardée par le service de journal.         |

## <a name="page-server-reads"></a>Lectures de serveur de pages

Les réplicas de calcul ne mettent pas en cache une copie complète de la base de données localement. Les données locales du réplica de calcul sont stockées dans le pool de mémoires tampons (en mémoire) et dans le cache local de l’extension du pool de mémoires tampons résilientes (RBPEX), qui est un cache partiel (non couvrant) des pages de données. Ce cache RBPEX local est dimensionné proportionnellement à la taille de calcul et représente trois fois la taille de mémoire du niveau de calcul. Le cache RBPEX est similaire au pool de mémoires tampons dans le sens où il contient les données les plus fréquemment sollicitées. En revanche, chaque serveur de pages dispose d’un cache RBPEX couvrant pour la partie de la base de données qu’il gère.

Quand une lecture est émise sur un réplica de calcul, si les données n’existent pas dans le pool de mémoires tampons ou dans le cache RBPEX local, un appel de fonction getPage (pageId, LSN) est émis et la page est récupérée à partir du serveur de pages correspondant. Les lectures à partir des serveurs de pages étant des lectures distantes, elles sont plus lentes que les lectures à partir du cache RBPEX local. Pour résoudre les problèmes de performances liés aux E/S, il importe de connaître le nombre d’E/S liées aux lectures distantes relativement lentes effectuées à partir d’un serveur de pages.

Plusieurs vues de gestion dynamique (DMV) et événements étendus contiennent des colonnes et des champs qui spécifient le nombre de lectures à distante à partir d’un serveur de pages, qui peuvent être comparées au nombre total de lectures. Le magasin des requêtes capture également les lectures distantes dans le cadre des statistiques de durée d’exécution des requêtes.

- Les colonnes où sont consignées les lectures de serveur de pages sont disponibles dans les vues DMV d’exécution et les vues catalogue, à savoir :

  - [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql/)
  - [sys.dm_exec_query_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql/)
  - [sys.dm_exec_procedure_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-procedure-stats-transact-sql/)
  - [sys.dm_exec_trigger_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-trigger-stats-transact-sql/)
  - [sys.query_store_runtime_stats](/sql/relational-databases/system-catalog-views/sys-query-store-runtime-stats-transact-sql/)
- Les lectures de serveur de pages sont ajoutées aux événements étendus suivants :
  - sql_statement_completed
  - sp_statement_completed
  - sql_batch_completed
  - rpc_completed
  - scan_stopped
  - query_store_begin_persist_runtime_stat
  - query-store_execution_runtime_info
- ActualPageServerReads/ActualPageServerReadAheads sont ajoutés au code XML du plan de requête pour les plans réels. Par exemple :

`<RunTimeCountersPerThread Thread="8" ActualRows="90466461" ActualRowsRead="90466461" Batches="0" ActualEndOfScans="1" ActualExecutions="1" ActualExecutionMode="Row" ActualElapsedms="133645" ActualCPUms="85105" ActualScans="1" ActualLogicalReads="6032256" ActualPhysicalReads="0" ActualPageServerReads="0" ActualReadAheads="6027814" ActualPageServerReadAheads="5687297" ActualLobLogicalReads="0" ActualLobPhysicalReads="0" ActualLobPageServerReads="0" ActualLobReadAheads="0" ActualLobPageServerReadAheads="0" />`

> [!NOTE]
> Pour voir ces attributs dans la fenêtre de propriétés du plan de requête, vous avez besoin de SSMS 18.3 ou version ultérieure.

## <a name="virtual-file-stats-and-io-accounting"></a>Statistiques sur les fichiers virtuels et comptabilisation des E/S

Dans Azure SQL Database, la fonction de gestion dynamique (DMF) [sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql/) est le principal moyen de superviser les E/S de SQL Database. Les caractéristiques des E/S dans Hyperscale sont différentes en raison de son [architecture distribuée](service-tier-hyperscale.md#distributed-functions-architecture). Dans cette section, nous nous concentrons sur les E/S (lectures et écritures) dans les fichiers de données tels qu’ils apparaissent dans cette fonction DMF. Dans Hyperscale, chaque fichier de données visible dans cette fonction DMF correspond à un serveur de pages distant. Le cache RBPEX mentionné ici est un cache local basé sur SSD qui est un cache non couvrant sur le réplica de calcul.

### <a name="local-rbpex-cache-usage"></a>Utilisation de cache RBPEX local

Il existe un cache RBPEX local sur le réplica de calcul du stockage SSD local. Par conséquent, les E/S sur ce cache sont plus rapides que les E/S sur les serveurs de pages distants. Actuellement, dans une base de données Hyperscale, [sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql/) contient une ligne spéciale qui fait état des E/S qui se sont produites dans le cache RBPEX local du réplica de calcul. Cette ligne a la valeur 0 pour les colonnes `database_id` et `file_id`. Par exemple, la requête ci-dessous retourne des statistiques d’utilisation RBPEX depuis le démarrage de la base de données.

`select * from sys.dm_io_virtual_file_stats(0,NULL);`

Le ratio de lectures effectuées dans le cache RBPEX par rapport aux lectures agrégées effectuées dans tous les autres fichiers de données donne le taux d’accès au cache RBPEX.

### <a name="data-reads"></a>Lectures de données

- Quand les lectures sont émises par le moteur de base de données SQL Server sur un réplica de calcul, elles peuvent être traitées par le cache RBPEX local, par des serveurs de pages distants ou par une combinaison des deux dans le cas où plusieurs pages sont lues.
- Quand le réplica de calcul lit certaines pages d’un fichier spécifique, par exemple file_id 1, si ces données résident uniquement dans le cache RBPEX local, toutes les E/S de cette lecture comptent pour file_id 0 (RBPEX). Si une partie de ces données se trouve dans le cache RBPEX local et qu’une autre partie se trouve sur un serveur de pages distant, les E/S comptent pour file_id 0 pour la partie traitée à partir du cache RBPEX, et la partie traitée à partir du serveur de pages distant compte pour file_id 1.
- Quand un réplica de calcul demande une page à un numéro [LSN](/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide/) particulier sur un serveur de pages, si celui-ci a tardé à atteindre le LSN demandé, la lecture sur le réplica de calcul attend que le serveur de pages l’ait atteinte avant que la page soit retournée au réplica de calcul. Pour toute lecture auprès d’un serveur de pages du réplica de calcul, vous constaterez la présence du type d’attente PAGEIOLATCH_* si cette E/S est attendue. Dans Hyperscale, ce temps d’attente est constitué à la fois du temps nécessaire pour atteindre la page demandée sur le serveur de pages au numéro LSN voulu et du temps nécessaire pour transférer la page du serveur de pages au réplica de calcul.
- Les lectures volumineuses que sont notamment les lectures anticipées se produisent souvent sous forme de [lectures par ventilation-regroupement](/sql/relational-databases/reading-pages/). Cela autorise des lectures pouvant atteindre 4 Mo de pages à la fois, ce que le moteur de base de données SQL Server considère comme une lecture unique. En revanche, quand les données lues se trouvent dans un cache RBPEX, ces lectures comptent pour plusieurs lectures individuelles de 8 Ko, car le pool de mémoires tampons et le cache RBPEX utilisent toujours des pages de 8 Ko. De ce fait, le nombre d’E/S en lecture détectées sur le cache RBPEX peut être supérieur au nombre réel d’E/S exécutées par le moteur.

### <a name="data-writes"></a>Écritures de données

- Le réplica de calcul principal n’écrit pas directement sur les serveurs de pages. Au lieu de cela, les enregistrements de journal du service de journal sont relus sur les serveurs de pages correspondants.
- Les écritures qui se produisent sur le réplica de calcul sont principalement des écritures dans le cache RBPEX local (file_id 0). Pour les écritures dans les fichiers logiques dont la taille est supérieure à 8 Ko, en d’autres termes celles qui résultent d’[opérations d’écriture avec regroupement](/sql/relational-databases/writing-pages/), chaque opération d’écriture est traduite en plusieurs écritures individuelles de 8 Ko dans le cache RBPEX, car le pool de mémoires tampons et le cache RBPEX utilisent toujours des pages de 8 Ko. De ce fait, le nombre d’E/S en écriture détectées sur le cache RBPEX peut être supérieur au nombre réel d’E/S exécutées par le moteur.
- Les fichiers non RBPEX, ou les fichiers de données autres que file_id 0 qui correspondent à des serveurs de pages, présentent aussi des écritures. Dans le niveau de service Hyperscale, ces écritures sont simulées, car les réplicas de calcul n’écrivent jamais directement sur des serveurs de pages. Les E/S par seconde en écriture et le débit sont comptabilisés à mesure qu’elles se produisent sur le réplica de calcul, mais la latence pour les fichiers de données autres que file_id 0 ne reflète pas la latence réelle des écritures de serveur de pages.

### <a name="log-writes"></a>Écritures de journal

- Sur le réplica de calcul principal, une écriture de journal est représentée dans le fichier file_id 2 de sys.dm_io_virtual_file_stats. Une écriture de journal sur le réplica de calcul principal est une écriture dans la zone d’atterrissage du journal.
- Les enregistrements de journal ne sont pas renforcés sur le réplica secondaire après une validation. Dans Hyperscale, le journal est appliqué par le service de journal aux réplicas secondaires de manière asynchrone. Étant donné que les écritures de journal ne se produisent pas réellement sur les réplicas secondaires, toute comptabilité des E/S de journal sur les réplicas secondaires est établie uniquement à des fins de suivi.

## <a name="data-io-in-resource-utilization-statistics"></a>E/S de données dans les statistiques d’utilisation des ressources

Dans une base de données non Hyperscale, les E/S en lecture et en écriture combinées sur les fichiers de données, par rapport à la limite d’E/S de données de la [gouvernance des ressources](./resource-limits-logical-server.md#resource-governance), sont rapportées dans les affichages [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) et [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database), au niveau de la colonne `avg_data_io_percent`. La même valeur est indiquée dans le portail Azure en tant que _pourcentage d’E/S de données_ .

Dans une base de données Hyperscale, cette colonne indique l’utilisation d’IOPS des données par rapport à la limite du stockage local sur le réplica de calcul uniquement, en particulier les E/S liées à RBPEX et `tempdb`. Une valeur de 100 % dans cette colonne indique que la gouvernance des ressources limite le stockage local en IOPS. Si cela est lié à un problème de performances, paramétrez la charge de travail pour générer moins d’E/S ou augmentez le service de base de données afin d’augmenter la _limite_ [Max Data IOPS](resource-limits-vcore-single-databases.md) (IOPS maxi de données) pour la gouvernance des ressources. Pour la gouvernance des ressources des lectures et écritures RBPEX, le système compte un nombre d’E/S individuelles de 8 Ko plutôt que des E/S plus volumineuses qui peuvent être émises par le moteur de base de données SQL Server.

Les E/S de données sur les serveurs de pages distants ne sont pas rapportées dans les affichages d’utilisation des ressources ou dans le portail, mais le sont dans la fonction de gestion dynamique [sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql/), comme indiqué précédemment.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour connaître les limites de ressources vCore pour une base de données Hyperscale unique, consultez [Limites vCore pour le niveau de service Hyperscale](resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5)
- Pour savoir comment optimiser les performances d’Azure SQL Database, consultez [Ajustement des performances de requêtes dans Azure SQL Database](performance-guidance.md)
- Pour savoir comment optimiser les performances en utilisant le magasin de requêtes, consultez [Surveillance des performances à l’aide du magasin de requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store/)
- Pour obtenir des scripts de surveillance DMV, consultez [Supervision des performances d’Azure SQL Database à l’aide de vues de gestion dynamique](monitoring-with-dmvs.md)