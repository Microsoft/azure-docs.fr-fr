---
title: Meilleures pratiques pour les pools SQL dédiés
description: Recommandations et meilleures pratiques à connaître lorsque vous utilisez des pools SQL dédiés.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: 03a536e16a6ba12611ed704b404c1bd411f0c4c8
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322698"
---
# <a name="best-practices-for-dedicated-sql-pools-in-azure-synapse-analytics"></a>Meilleures pratiques pour les pools SQL dédiés dans Azure Synapse Analytics

Cet article énumère une série de meilleures pratiques destinées à optimiser les performances des pools SQL dédiés dans Azure Synapse Analytics. Vous trouverez ci-dessous des instructions de base et des aspects importants sur lesquels vous concentrer lorsque vous créez votre solution. Chaque section présente un concept et vous dirige ensuite vers des articles plus détaillés qui expliquent davantage le concept.

## <a name="dedicated-sql-pools-loading"></a>Chargement des pools SQL dédiés

Pour obtenir des conseils sur le chargement des pools SQL dédiés, consultez les [conseils relatifs au chargement de données](data-loading-best-practices.md).

## <a name="reduce-cost-with-pause-and-scale"></a>Réduire les coûts avec les opérations de suspension et de mise à l’échelle

Pour plus d’informations sur la réduction des coûts au travers de la suspension et de la mise à l’échelle, consultez [Gérer le calcul](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="maintain-statistics"></a>Mettre à jour les statistiques

Contrairement à SQL Server qui détecte et crée ou met à jour les statistiques dans les colonnes automatiquement, les pools SQL dédiés nécessitent une tenue à jour manuelle des statistiques. La tenue à jour des statistiques garantit que les plans de pool SQL sont optimisés.  Les plans créés par l’optimiseur sont aussi bons que les statistiques disponibles.

> [!TIP]
> La création de statistiques échantillonnées est un bon moyen de se familiariser avec la notion de statistiques.  

Il est également important de mettre à jour les statistiques car des modifications significatives affectent vos données.  Une méthode plus classique serait peut-être de mettre à jour vos statistiques tous les jours ou après chaque charge.  Lorsque vous créez et mettez des statistiques à jour, vous devez toujours faire un compromis entre les performances et les coûts.

Pour raccourcir le temps de maintenance des statistiques, vous devez sélectionner des colonnes contenant des statistiques ou nécessitant les mises à jour les plus fréquentes. Par exemple, vous pouvez mettre à jour des colonnes de date auxquelles de nouvelles valeurs peuvent être ajoutées quotidiennement. Concentrez-vous sur l’obtention de statistiques pour les colonnes utilisées dans des jointures, celles utilisées dans la clause WHERE et celles figurant dans GROUP BY.

Pour plus d’informations sur les statistiques, consultez les articles [Gestion des statistiques sur les tables](develop-tables-statistics.md), [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [UPDATE STATISTICS](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="group-insert-statements-into-batches"></a>Regrouper des instructions INSERT dans des lots

Une charge unique dans une petite table avec une instruction INSERT telle que `INSERT INTO MyLookup VALUES (1, 'Type 1')` peut être la meilleure approche en fonction de vos besoins. Toutefois, si vous avez besoin de charger des milliers ou millions de lignes sur une même journée, l’utilisation d’instructions INSERTS de singleton n’est peut-être pas optimale.

Une façon de résoudre ce problème consiste à développer un processus qui écrit dans un fichier, et un autre processus pour charger régulièrement ce fichier. Pour plus d’informations, consultez l’article [INSERT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="use-polybase-to-load-and-export-data-quickly"></a>Utiliser PolyBase pour charger et exporter rapidement des données

Le pool SQL prend en charge le chargement et l’exportation de données via plusieurs outils dont Azure Data Factory, PolyBase et BCP.  Pour les petits volumes de données où les performances ne sont pas essentielles, n’importe quel outil peut suffire à vos besoins.  

> [!NOTE]
> Polybase est le meilleur choix si vous chargez ou exportez des volumes importants de données, ou si vous avez besoin de performances plus rapides.

Les charges PolyBase peuvent être exécutées à l’aide de CTAS ou d’INSERT INTO. La fonctionnalité CTAS permet de minimiser la journalisation des transactions et est la méthode la plus rapide pour charger vos données. Azure Data Factory prend également en charge les charges PolyBase et peut atteindre des performances similaires à celles de CTAS. PolyBase prend en charge une variété de formats de fichiers, dont les fichiers Gzip.

Pour maximiser le débit lors de l’utilisation de fichiers texte Gzip, divisez les fichiers en 60 fichiers ou plus pour optimiser le parallélisme de votre charge. Pour un débit total plus rapide, envisagez le chargement simultané des données. Vous trouverez des informations supplémentaires sur les sujets relatifs à cette section dans les articles suivants :

- [Chargement des données](../sql-data-warehouse/design-elt-data-loading.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Guide d’utilisation de PolyBase](data-loading-best-practices.md)
- [Modèles et stratégies de chargement de pools SQL Azure](https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-loading-patterns-and-strategies/)
- [Téléchargement de données avec Azure Data Factory](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Déplacer les données avec Azure Data Factory](../../data-factory/transform-data-using-machine-learning.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Create table as select (CTAS)](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

## <a name="load-then-query-external-tables"></a>Charger, puis interroger les tables externes

Polybase n’est une solution optimale pour les requêtes. Les tables Polybase pour pools SQL dédiés ne prennent en charge que les fichiers blob Azure et le stockage Azure Data Lake. Ces fichiers ne sont adossés à aucune ressource de calcul. Par conséquent, les pools SQL dédiés ne peuvent pas délester ce travail et doivent lire le fichier entier en le chargeant dans tempdb afin de pouvoir lire les données.

Si vous disposez de plusieurs requêtes pour interroger ces données, il est préférable de charger ces dernières une fois et que les requêtes utilisent la table locale. D’autres conseils sur Polybase sont inclus dans le [Guide d’utilisation de PolyBase](data-loading-best-practices.md).

## <a name="hash-distribute-large-tables"></a>Hacher et distribuer de grandes tables

Par défaut, les tables sont distribuées par tourniquet (Round Robin).   Cela aide les utilisateurs à commencer à créer leurs tables sans devoir déterminer comment celles-ci doivent être distribuées. Des tables round robin peuvent être suffisamment performantes pour certaines charges de travail. Toutefois, dans la plupart des cas, une colonne de distribution offre de meilleures performances.  

L’exemple le plus courant de table distribuée par une colonne surpassant les performances d’une table round robin est lorsque deux grandes tables de faits sont jointes.  

Par exemple, si vous avez une table de commandes distribuée par order_id, et une table de transactions également distribuée par order_id, lorsque vous joignez votre table de commandes à votre table de transactions sur order_id, cette requête devient une requête directe. Les opérations de déplacement de données sont ensuite éliminées. Moins d’étapes signifie une requête plus rapide. Moins de déplacement des données permet également d’obtenir des requêtes plus rapides.

> [!NOTE]
> Lors du chargement d’une table distribuée, vos données entrantes ne doivent pas être triées sur la clé de distribution. Cela ralentit vos chargements.

Les liens d’articles ci-dessous vous permettent d’accéder à des informations supplémentaires sur l’amélioration des performances en sélectionnant une colonne de distribution. Vous y trouverez également des informations sur la définition d’une table distribuée dans la clause WITH de votre instruction CREATE TABLE :

- [Vue d’ensemble des tables](develop-tables-overview.md)
- [Distribution de table](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Sélection d’une distribution de tables](https://blogs.msdn.microsoft.com/sqlcat/20../../choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service/)
- [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## <a name="do-not-over-partition"></a>Ne pas créer trop de partitions

Bien que le partitionnement des données peut être efficace pour mettre à jour vos données grâce au basculement de partitions ou à l’optimisation des analyses avec élimination des partitions, avoir un trop grand nombre de partitions peut ralentir vos requêtes.  Souvent une stratégie de partitionnement à granularité élevée qui peut fonctionner correctement sur SQL Server peut poser des problèmes sur un pool SQL dédié.  

Un trop grand nombre de partitions peut réduire l’efficacité des index columnstore en cluster si chaque partition compte moins d’1 million de lignes. Les pools SQL dédiés partitionnent automatiquement vos données en 60 bases de données. Ainsi, si vous créez une table contenant 100 partitions, vous obtenez 6000 bases de données. Chaque charge de travail étant singulière, le meilleur conseil est d’expérimenter le partitionnement pour voir ce qui fonctionne le mieux pour votre charge de travail.  

L’une des options à prendre en compte est l’utilisation d’une granularité inférieure à celle que vous avez implémentée à l’aide de SQL Server. Par exemple, envisagez d’utiliser des partitions hebdomadaires ou mensuelles plutôt que quotidiennes.

Pour plus d’informations sur le partitionnement, consultez l’article [Partitionnement de tables](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="minimize-transaction-sizes"></a>Minimiser la taille des transactions

Les instructions INSERT, UPDATE et DELETE s’exécutent dans une transaction. En cas d’échec, elles doivent être restaurées. Pour minimiser le risque d’une restauration longue, réduisez autant que possible les tailles de transactions.  Vous pouvez réduire la taille des transactions en fractionnant les instructions INSERT, UPDATE et DELETE. Par exemple, si vous avez une instruction INSERT dont l’exécution devrait prendre une heure, vous pouvez la fractionner en quatre parties. Chaque exécution sera ensuite raccourcie à 15 minutes.  

> [!TIP]
> Tirez parti des cas spéciaux de journalisation minimale, tels que CTAS, TRUNCATE, DROP TABLE ou INSERT pour vider des tables afin de réduire le risque de restauration.  

Un autre moyen d’éliminer les restaurations consiste à utiliser des opérations de métadonnées uniquement comme le basculement de partitions pour la gestion des données.  Par exemple, au lieu d’exécuter une instruction DELETE pour supprimer toutes les lignes d’une table dans lesquelles la date order_date était en octobre 2001, vous pouvez partitionner vos données mensuellement. Vous pouvez ensuite quitter la partition avec des données pour une partition vide d’une autre table (voir les exemples ALTER TABLE).  

Pour les tables non partitionnées, utilisez une instruction CTAS pour écrire les données que vous souhaitez conserver dans une table plutôt que l’instruction DELETE.  Si une instruction CTAS prend le même temps, son exécution est beaucoup plus sûre, car elle effectue une journalisation des transactions minimale et peut être annulée rapidement si nécessaire.

Pour plus d’informations sur le contenu lié à cette section, consultez les articles suivants :

- [Create table as select (CTAS)](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Présentation des transactions](develop-transactions.md)
- [Optimizing transactions](../sql-data-warehouse/sql-data-warehouse-develop-best-practices-transactions.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Partitionnement de table](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [TRUNCATE TABLE](/sql/t-sql/statements/truncate-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## <a name="reduce-query-result-sizes"></a>Réduire la taille des résultats de requête

La réduction de la taille des résultats de requête vous aide à éviter des problèmes côté client occasionnés par des résultats de requête volumineux.  Vous pouvez modifier votre requête afin de réduire le nombre de lignes retournées. Certains outils de génération de requêtes vous permettent d’ajouter la syntaxe « N premiers » à chaque requête.  Vous pouvez également utiliser CREATE EXTERNAL TABLE AS SELECT (CETAS) pour exporter le résultat de la requête dans une table temporaire, puis utiliser l’exportation PolyBase pour le traitement de niveau inférieur.

## <a name="use-the-smallest-possible-column-size"></a>Utiliser la plus petite taille de colonne possible

Lorsque vous définissez votre DDL, le plus petit type de données prenant en charge vos données. Cela améliore les performances de requête.  Cette recommandation est particulièrement importante pour les colonnes CHAR et VARCHAR.  Si la valeur la plus longue dans une colonne est de 25 caractères, définissez la colonne en tant que VARCHAR(25).  Évitez de définir toutes les colonnes de caractères sur une grande longueur par défaut.  Par ailleurs, lorsque cela suffit, définissez des colonnes VARCHAR plutôt que NVARCHAR.

Pour plus d’informations sur les concepts essentiels liés aux informations ci-dessus, consultez les articles [Vue d’ensemble des tables](develop-tables-overview.md), [Types de données des table](develop-tables-data-types.md) et [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="use-temporary-heap-tables-for-transient-data"></a>Utiliser des tables de segments de mémoire temporaires pour les données temporaires

Lorsque vous posez temporairement des données sur des pools SQL dédiés, des tables de segments de mémoire accélèrent généralement le processus global.  Si vous chargez des données uniquement pour les ranger avant d’exécuter d’autres transformations, il est plus rapide de charger la table dans une table de segments de mémoire que de charger les données dans une table columnstore en cluster.  

Le chargement des données dans une table temporaire sera également beaucoup plus rapide que le chargement d’une table dans un stockage permanent.  Les tables temporaires commencent par « # » et ne sont accessibles qu’à la session qui les a créées. Elles ne peuvent donc fonctionner que dans des scénarios limités. Les tables de segments de mémoire sont définies dans la clause WITH d’une instruction CREATE TABLE.  Si vous utilisez une table temporaire, n’oubliez pas de créer des statistiques dans la table temporaire également.

Pour des conseils supplémentaires, consultez les articles [Tables temporaires](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="optimize-clustered-columnstore-tables"></a>Optimiser les tables columnstore en clusters

Les index columnstore en cluster sont l’une des méthodes les plus efficaces pour stocker vos données dans un pool SQL dédié.  Par défaut, les tables du pool SQL dédié sont créées en tant que ColumnStore en cluster.  Pour obtenir les meilleures performances pour les requêtes sur les tables columnstore, la qualité du segment est importante.  Lorsque les lignes sont écrites dans les tables columnstore avec une mémoire insuffisante, la qualité du segment columnstore peut être affectée.  

La qualité d’un segment peut se mesurer au nombre de lignes dans un groupe de lignes compressé. Consultez la section [Causes de la qualité médiocre des index columnstore](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#causes-of-poor-columnstore-index-quality) dans l’article [Index de table](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) pour obtenir des instructions étape par étape sur la détection et l’amélioration de la qualité de segment pour les tables columnstore en cluster.  

Étant donné que la qualité des segments columnstore est importante, nous vous conseillons d’utiliser des ID d’utilisateurs qui se trouvent dans la classe de ressource de moyenne ou grande taille pour le chargement des données. L’utilisation d’[unités DWU (Data Warehouse Unit)](resource-consumption-models.md) inférieures signifie que vous souhaitez assigner une classe de ressource plus grande à votre utilisateur de chargement.

En général, les tables columnstore n’envoient pas de données dans un segment columnstore compressé tant qu’il y a pas plus d’un million de lignes par table. Chaque table de pool SQL dédié est partitionnée en 60 tables. Par conséquent, des tables columnstore ne bénéficient pas d’une requête, sauf si elles comptent plus de 60 millions lignes.  

> [!TIP]
> Pour les tables contenant moins de 60 millions lignes, il se peut qu’un index columnstore ne soit pas la solution optimale.  

Si vous partitionnez vos données, chaque partition doit compter 1 million de lignes pour qu’un index columnstore en cluster soit avantageux.  Si une table comprend 100 partitions, elle devra compter au moins 6 milliards de lignes pour tirer avantage d’un index columnstore en cluster (60 distributions  *100 partitions*  1 million de lignes).  

Si votre table ne contient pas 6 milliards lignes, vous avez deux options principales. Réduire le nombre de partitions ou envisager d’utiliser une table de segments de mémoire à la place.  Il peut également être intéressant d’expérimenter pour voir si de meilleures performances peuvent être obtenues en utilisant une table de segments de mémoire avec des index secondaires plutôt qu’avec une table columnstore.

Lorsque vous interrogez une table columnstore, les requêtes s’exécutent plus vite si vous sélectionnez uniquement les colonnes dont vous avez besoin.  Pour plus d’informations sur les index de table et columnstore, consultez les articles [Index de table](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), [Guide des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [Reconstruction des index columnstore](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true#rebuilding-indexes-to-improve-segment-quality).

## <a name="use-larger-resource-class-to-improve-query-performance"></a>Utiliser une classe de ressource plus grande pour améliorer les performances des requêtes

Les pools SQL utilisent des groupes de ressources pour allouer de la mémoire aux requêtes. Initialement, tous les utilisateurs sont affectés à la petite classe de ressources qui accorde 100 Mo de mémoire par distribution.  Il y a toujours 60 distributions. Chaque distribution se voit attribuer un minimum de 100 Mo. L’allocation de mémoire totale pour l’ensemble du système est de 6 000 Mo ou juste inférieure à 6 Go.  

Certaines requêtes, telles que des grandes jointures ou des charges dans des tables columnstore en cluster, bénéficieront d’allocations de mémoire supérieures.  Certaines requêtes comme des analyses pures n’en tireront aucun bénéfice. L’utilisation de classes de ressources de plus grande taille a une incidence sur la concurrence. Gardez donc ces faits à l’esprit avant de déplacer tous vos utilisateurs vers une classe de ressources de grande taille.

Pour plus d’informations sur les classes de ressources, consultez l’article [Classes de ressources pour la gestion des charges de travail](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="use-smaller-resource-class-to-increase-concurrency"></a>Utiliser une classe de ressources de plus petite taille pour augmenter la concurrence

Si vous constatez un délai long dans les requêtes utilisateur, il se peut que vos utilisateurs peuvent exécutent les requêtes dans des classes de ressources de plus grande taille. Ce scénario favorise l’utilisation d’emplacements pour la concurrence, ce qui peut entraîner la mise en file d’attente d’autres requêtes.  Pour déterminer si des requêtes utilisateurs sont mises en file d’attente, exécutez `SELECT * FROM sys.dm_pdw_waits` pour voir si des lignes sont retournées.

Pour plus d’informations, consultez les articles [Classes de ressources pour la gestion des charges de travail](../sql-data-warehouse/resource-classes-for-workload-management.md) et [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="use-dmvs-to-monitor-and-optimize-your-queries"></a>Utiliser des DMV pour surveiller et optimiser vos requêtes

Les pools SQL disposent de plusieurs vues de gestion dynamique qui peuvent être utilisées pour surveiller l’exécution de requête.  L’article sur la surveillance ci-dessous fournit des instructions détaillées sur la façon de consulter les détails d’une requête en cours d’exécution.  Pour trouver rapidement des requêtes dans ces DMV, l’utilisation de l’option LABEL avec vos requêtes peut aider. Pour plus d’informations détaillées, consultez les articles figurant dans la liste ci-dessous :

- [Surveiller votre charge de travail à l'aide de vues de gestion dynamique](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

- [LABEL](develop-label.md)
- [OPTION](/sql/t-sql/queries/option-clause-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## <a name="next-steps"></a>Étapes suivantes

Consultez également notre article [Dépannage](../sql-data-warehouse/sql-data-warehouse-troubleshoot.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) concernant les problèmes courants et leurs solutions.

Si vous avez besoin d’informations non fournies dans cet article, la [page de questions Microsoft Q&A sur Azure Synapse](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html) vous permet de poser des questions à d’autres utilisateurs et au groupe du produit pool SQL.  

Nous suivons activement ce forum pour vous assurer que vos questions sont traitées par un autre utilisateur ou un membre de notre équipe.  Si vous préférez poser vos questions sur Stack Overflow, nous avons également un [Forum Stack Overflow Azure Synapse Stack](https://stackoverflow.com/questions/tagged/azure-sqldw).

Pour demander des fonctionnalités, utilisez la page [Commentaires sur le pool SQL Azure](https://feedback.azure.com/forums/307516-sql-data-warehouse).  L’ajout de demandes ou le vote pour d’autres demandes nous aide à nous concentrer sur les fonctionnalités les plus demandées.
