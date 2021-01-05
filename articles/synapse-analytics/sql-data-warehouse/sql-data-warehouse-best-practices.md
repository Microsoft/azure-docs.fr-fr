---
title: Meilleures pratiques pour un pool SQL dédié (anciennement SQL DW)
description: Recommandations et meilleures pratiques pour le développement de solutions pour un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/04/2019
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: 9802e6553d553aae4f13194dc9951d1a17af6f66
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462879"
---
# <a name="best-practices-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Meilleures pratiques pour un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

Cet article rassemble les meilleures pratiques qui vous aideront à obtenir des performances optimales de votre déploiement de [pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-overview-what-is.md).  L’objectif de cet article est de vous fournir des conseils de base et de mettre l’accent sur les points importants.  

## <a name="reduce-cost-with-pause-and-scale"></a>Réduire les coûts avec les opérations de suspension et de mise à l’échelle

Pour plus d’informations sur la réduction des coûts à l’aide de la suspension et de la mise à l’échelle, consultez la section [Gérer le calcul](sql-data-warehouse-manage-compute-overview.md).

## <a name="maintain-statistics"></a>Mettre à jour les statistiques

Vous pouvez configurer un pool SQL dédié (anciennement SQL DW) pour détecter et créer automatiquement des statistiques sur des colonnes.  Les plans de requête créés par l’optimiseur sont aussi bons que les statistiques disponibles.  

Nous vous recommandons d’activer AUTO_CREATE_STATISTICS pour vos bases de données et de conserver les statistiques mises à jour quotidiennement ou après chaque charge pour vous assurer que les statistiques sur les colonnes utilisées dans vos requêtes sont toujours à jour.

Si vous trouvez que la mise à jour de toutes vos statistiques prend trop de temps, vous souhaiterez peut-être sélectionner les colonnes nécessitant des mises à jour fréquentes des statistiques. Par exemple, vous pouvez mettre à jour des colonnes de date, où de nouvelles valeurs peuvent être ajoutées de façon quotidienne.

> [!TIP]
> Vous bénéficierez de performances optimales en lançant des mises à jour des statistiques sur les colonnes impliquées dans les jointures, celles utilisées dans la clause WHERE et celles figurant dans GROUP BY.

Voir aussi [Gestion des statistiques sur les tables](sql-data-warehouse-tables-statistics.md), [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et [UPDATE STATISTICS](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="use-dmvs-to-monitor-and-optimize-your-queries"></a>Utiliser des DMV pour surveiller et optimiser vos requêtes

Un pool SQL dédié (anciennement SQL DW) offre plusieurs vues de gestion dynamique permettant de surveiller l’exécution des requêtes.  L’article [Surveiller votre charge de travail à l’aide de vues de gestion dynamique](sql-data-warehouse-manage-monitor.md) fournit des instructions pas à pas sur la façon d’examiner les détails d’une requête en cours d’exécution.  

Pour trouver rapidement des requêtes dans ces DMV, l’utilisation de l’option LABEL avec vos requêtes peut aider.

Voir aussi [Surveiller votre charge de travail à l’aide de vues de gestion dynamique](sql-data-warehouse-manage-monitor.md), [LABEL](sql-data-warehouse-develop-label.md), [OPTION](/sql/t-sql/queries/option-clause-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="tune-query-performance-with-new-product-enhancements"></a>Optimiser les performances des requêtes avec de nouvelles améliorations produit

- [Réglage des performances avec des vues matérialisées](performance-tuning-materialized-views.md)
- [Réglage des performances avec un index columnstore en cluster ordonné](performance-tuning-ordered-cci.md)
- [Optimisation des performances avec la mise en cache des jeux de résultats](performance-tuning-result-set-caching.md)

## <a name="group-insert-statements-into-batches"></a>Regrouper des instructions INSERT dans des lots

Un chargement unique dans une petite table avec une instruction INSERT ou même un rechargement périodique d’une recherche peuvent répondre à vos besoins grâce à une instruction comme `INSERT INTO MyLookup VALUES (1, 'Type 1')`.  

Toutefois, si vous avez besoin charger des milliers ou des millions de lignes sur une même journée, vous constaterez que les instructions INSERTS singleton sont inadaptées.  Au lieu de cela, développez vos processus afin qu’ils écrivent dans un fichier et un autre processus arrive régulièrement pour charger ce fichier.

Voir aussi [INSERT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="use-polybase-to-load-and-export-data-quickly"></a>Utiliser PolyBase pour charger et exporter rapidement des données

Un pool SQL dédié (anciennement SQL DW) prend en charge le chargement et l’exportation de données via plusieurs outils dont Azure Data Factory, PolyBase et BCP.  Pour les petits volumes de données où les performances ne sont pas essentielles, n’importe quel outil peut suffire à vos besoins.  Toutefois, lorsque vous chargez ou exportez de gros volumes de données ou si des performances de vitesse sont nécessaires, PolyBase représente le meilleur choix.  

PolyBase est conçu pour tirer parti de la nature distribuée du système et chargera et exportera les magnitudes des données plus rapidement que n’importe quel autre outil.  Les charges PolyBase peuvent être exécutées à l’aide de CTAS ou d’INSERT INTO.   

> [!TIP]
> L’utilisation de CTAS permet de minimiser la journalisation des transactions et constitue le moyen le plus rapide de charger vos données.

Azure Data Factory prend également en charge les charges PolyBase et peut atteindre les mêmes performances que CTAS.  PolyBase prend en charge une variété de formats de fichiers, y compris les fichiers Gzip.  
  
> [!NOTE]
> Pour maximiser le débit lors de l’utilisation de fichiers texte gzip, divisez les fichiers en 60 fichiers ou plus pour optimiser le parallélisme de votre charge.  Pour un débit total plus rapide, envisagez le chargement simultané des données.

Consultez aussi [Chargement de données](design-elt-data-loading.md), [Guide d’utilisation de PolyBase](guidance-for-loading-data.md), [Modèles et stratégies de chargement de pool SQL dédié](https://blogs.msdn.microsoft.com/sqlcat/20../../), [Téléchargement de données avec Azure Data Factory]( ../../data-factory/load-azure-sql-data-warehouse.md), [Déplacer des données à l’aide d’Azure Data Factory](../../data-factory/transform-data-using-machine-learning.md), [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et [Create table as select (CTAS)](sql-data-warehouse-develop-ctas.md).

## <a name="load-then-query-external-tables"></a>Charger, puis interroger les tables externes

Si Polybase, également appelé tables externes, peut être le moyen le plus rapide pour charger les données, il n’est pas optimal pour les requêtes. Les tables Polybase ne prennent actuellement en charge que les fichiers d’objets blob Azure et le stockage Azure Data Lake. Aucune ressource de calcul ne se charge de la sauvegarde de ces fichiers.  

Par conséquent, un pool SQL dédié ne peut pas décharger ce travail et doit lire le fichier entier en le chargeant dans tempdb pour lire les données.  Donc, si vous disposez de plusieurs requêtes qui vont interroger ces données, il est préférable de charger ces données une fois et que les requêtes utilisent la table locale.

Voir aussi [Guide d’utilisation de PolyBase](guidance-for-loading-data.md).

## <a name="hash-distribute-large-tables"></a>Hacher et distribuer de grandes tables

Par défaut, les tables sont distribuées par tourniquet (Round Robin).  Cela aide les utilisateurs à commencer la création de leurs tables sans avoir à déterminer comment les tables doivent être distribuées.  Les tables distribuées par tourniquet (Round Robin) peuvent offrir des performances suffisantes pour certaines charges de travail, mais souvent la sélection d’une colonne de distribution s’avérera plus efficace.  

L’exemple le plus courant de meilleures performances observées avec une table distribuée par une colonne par rapport à une table Round Robin est lorsque deux grandes tables de faits sont jointes.  

Par exemple, si vous avez une table de commandes, qui est distribuée par order_id, et une table de transactions, également distribuée par order_id, lorsque vous joignez votre table de commandes à votre table de transactions sur order_id, cette requête devient une requête directe, ce qui signifie que nous éliminons les opérations de déplacement de données.  Moins d’étapes signifie une requête plus rapide.  Moins de déplacement des données permet également d’obtenir des requêtes plus rapides.  

> [!TIP]
> Lors du chargement d’une table distribuée, assurez-vous que vos données entrantes ne sont pas triées sur la clé de distribution car cela ralentit vos charges.  

Consultez les liens ci-dessous pour des explications détaillées sur la façon dont la sélection d’une colonne de distribution peut améliorer les performances et pour apprendre à définir une table distribuée dans la clause WITH de l’instruction CREATE TABLE.

Voir aussi [Vue d’ensemble des tables](sql-data-warehouse-tables-overview.md), [Distribution de tables](sql-data-warehouse-tables-distribute.md), [Sélection d’une distribution de tables](https://blogs.msdn.microsoft.com/sqlcat/20../../choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service/), [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="do-not-over-partition"></a>Ne pas créer trop de partitions

Bien que le partitionnement des données peut être efficace pour mettre à jour vos données grâce au basculement de partitions ou à l’optimisation des analyses avec élimination des partitions, avoir un trop grand nombre de partitions peut ralentir vos requêtes.  Souvent, une stratégie de partitionnement à granularité élevée qui peut fonctionner correctement sur SQL Server peut poser des problèmes dans un pool SQL dédié (anciennement SQL DW).  

Un trop grand nombre de partitions peut également réduire l’efficacité des index columnstore en cluster si chaque partition possède moins d’1 million de lignes.  N’oubliez pas qu’en arrière-plan un pool SQL dédié partitionne vos données en 60 bases de données. Ainsi, si vous créez une table avec 100 partitions, cela produit en réalité 6 000 partitions.  

Chaque charge de travail est différente ; par conséquent, le meilleur conseil serait d’expérimenter le partitionnement pour voir ce qui fonctionne le mieux pour votre charge de travail.  Envisagez d’abaisser le niveau de granularité à un niveau inférieur à celui qui a fonctionné pour vous dans SQL Server.  Par exemple, utilisez plutôt des partitions hebdomadaires ou mensuelles plutôt que des partitions quotidiennes.

Voir aussi [Partitionnement de table](sql-data-warehouse-tables-partition.md).

## <a name="minimize-transaction-sizes"></a>Minimiser la taille des transactions

Les instructions INSERT, UPDATE et DELETE s’exécutent dans une transaction, et en cas d’échec elles doivent être restaurées.  Pour minimiser le risque d’une restauration longue, réduisez si possible les tailles de transactions.  Pour ce faire, vous pouvez diviser les instructions INSERT, UPDATE et DELETE en plusieurs parties.  

Par exemple, si vous disposez d’une instruction INSERT qui devrait prendre une heure, décomposez si possible l’insertion en quatre parties, qui seront chacune exécutées en 15 minutes.  Exploitez des cas spéciaux de journalisation minimale, tels que CTAS, TRUNCATE, DROP TABLE ou INSERT, dans des tables vides, afin de réduire le risque de restauration.  

Un autre moyen d’éliminer les restaurations consiste à utiliser des opérations de métadonnées uniquement comme le basculement de partitions pour la gestion des données.  Par exemple, plutôt que d’exécuter une instruction DELETE pour supprimer toutes les lignes d’une table où order_date était octobre 2001, vous pouvez partitionner vos données tous les mois et ensuite extraire la partition contenant les données vers une partition vide à partir d’une autre table (voir les exemples [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)).  

Pour les tables non partitionnées, utilisez une instruction CTAS pour écrire les données que vous souhaitez conserver dans une table plutôt que l’instruction DELETE.  Si une instruction CTAS prend le même laps de temps, elle permet une opération beaucoup plus sûre car elle offre une journalisation des transactions minimale et peut être annulée rapidement si nécessaire.

Voir aussi [Transactions](sql-data-warehouse-develop-transactions.md), [Optimisation des transactions](sql-data-warehouse-develop-best-practices-transactions.md), [Partitionnement de table](sql-data-warehouse-tables-partition.md), [TRUNCATE TABLE](/sql/t-sql/statements/truncate-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et [Create table as select (CTAS)](sql-data-warehouse-develop-ctas.md).

## <a name="reduce-query-result-sizes"></a>Réduire la taille des résultats de requête

Cette étape vous aide à éviter les problèmes côté client causés par des résultats de requête volumineux.  Vous pouvez modifier votre requête afin de réduire le nombre de lignes retournées. Certains outils de génération de requêtes vous permettent d’ajouter la syntaxe « N premiers » à chaque requête.  Vous pouvez également utiliser CREATE EXTERNAL TABLE AS SELECT (CETAS) pour exporter le résultat de la requête dans une table temporaire, puis utiliser l’exportation PolyBase pour le traitement de niveau inférieur.

## <a name="use-the-smallest-possible-column-size"></a>Utiliser la plus petite taille de colonne possible

Lorsque vous définissez votre DDL, l’utilisation du plus petit type de données prenant en charge vos données améliore les performances de la requête.  Cette approche est particulièrement importante pour les colonnes CHAR et VARCHAR.  

Si la valeur la plus longue dans une colonne est de 25 caractères, définissez la colonne en tant que VARCHAR(25).  Évitez de définir toutes les colonnes de caractères sur une grande longueur par défaut.  En outre, définissez des colonnes VARCHAR lorsque cela suffit, au lieu d’utiliser NVARCHAR.

Voir aussi [Vue d’ensemble des tables](sql-data-warehouse-tables-overview.md), [Types de données des tables](sql-data-warehouse-tables-data-types.md), [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="use-temporary-heap-tables-for-transient-data"></a>Utiliser des tables de segments de mémoire temporaires pour les données temporaires

Lorsque vous envoyez des données, vous trouverez peut-être que l’utilisation d’une table de segments de mémoire accélère le processus global.  Si vous chargez des données uniquement pour les mettre en préproduction avant d’exécuter d’autres transformations, il sera beaucoup plus rapide de charger la table dans une table de segments de mémoire que de charger des données dans une table columnstore cluster.  

En outre, le chargement des données dans une table temporaire sera également beaucoup plus rapide que le chargement d’une table dans un stockage permanent.  Les tables temporaires commencent par le signe « # » et sont accessibles uniquement par la session qui les a créées ; par conséquent, il se peut qu’elles ne fonctionnent que dans des scénarios limités.

Les tables de segments de mémoire sont définies dans la clause WITH d’une instruction CREATE TABLE.  Si vous utilisez une table temporaire, n’oubliez pas de créer des statistiques dans la table temporaire également.

Voir aussi [Tables temporaires](sql-data-warehouse-tables-temporary.md), [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="optimize-clustered-columnstore-tables"></a>Optimiser les tables columnstore en clusters

Les index columnstore en cluster sont l’une des méthodes les plus efficaces pour stocker vos données dans un pool SQL dédié.  Par défaut, les tables du pool SQL dédié sont créées en tant que ColumnStore en cluster.  Pour obtenir les meilleures performances pour les requêtes sur les tables columnstore, la qualité du segment est importante.  

Lorsque les lignes sont écrites dans les tables columnstore avec une mémoire insuffisante, la qualité du segment columnstore peut être affectée.  La qualité du segment peut être mesurée par le nombre de lignes dans un groupe de lignes compressé.  Consultez la section [Causes de la qualité médiocre des index columnstore](sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality) dans l’article [Index de table](sql-data-warehouse-tables-index.md) pour obtenir des instructions étape par étape sur la détection et l’amélioration de la qualité de segment pour les tables columnstore en cluster.  

Étant donné que la qualité des segments columnstore est importante, nous vous conseillons d’utiliser des ID d’utilisateurs qui se trouvent dans la classe de ressource de moyenne ou grande taille pour le chargement des données. L’utilisation d’[unités DWU (Data Warehouse Unit)](what-is-a-data-warehouse-unit-dwu-cdwu.md) inférieures signifie que vous souhaitez assigner une classe de ressource plus grande à votre utilisateur de chargement.

Étant donné que les tables columnstore ne transmettent généralement pas de données dans un segment columnstore compressé s’il existe moins de 1 million de lignes par table et si chaque table du pool SQL dédié est partitionnée en 60 tables, en règle générale, les tables columnstore ne tireront aucun profit d’une requête, sauf si la table comporte plus de 60 millions de lignes.  Pour une table comportant moins de 60 millions de lignes, il ne sera peut-être pas judicieux d’avoir un index columnstore.  Mais cela ne peut pas nuire non plus.  

En outre, si vous partitionnez vos données, vous souhaiterez peut-être estimer que chaque partition nécessitera 1 million de lignes pour bénéficier d’un index columnstore en cluster.  Si une table possède 100 partitions, elle devra avoir au moins 6 milliards de lignes pour bénéficier d’une banque de colonnes en cluster (60 distributions *100 partitions* 1 million de lignes).  

Si votre table ne possède pas six milliards de lignes dans cet exemple, réduisez le nombre de partitions ou envisagez plutôt d’utiliser une table de segment de mémoire.  Il peut être également intéressant de tester pour voir si de meilleures performances peuvent être obtenues avec une table de segment de mémoire ayant des index secondaires plutôt qu’avec une table columnstore.

> [!TIP]
> Lorsque vous interrogez une table columnstore, les requêtes s’exécutent plus vite si vous sélectionnez uniquement les colonnes dont vous avez besoin.  

Voir aussi [Index de table](sql-data-warehouse-tables-index.md), [Guide des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [Reconstruction des index columnstore](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality)

## <a name="use-larger-resource-class-to-improve-query-performance"></a>Utiliser une classe de ressource plus grande pour améliorer les performances des requêtes

Un pool SQL dédié utilise des groupes de ressources pour allouer de la mémoire aux requêtes.  Dès le départ, tous les utilisateurs sont affectés à la petite classe de ressource qui accorde 100 Mo de mémoire par distribution.  Dans la mesure où il existe toujours 60 distributions et que chaque distribution reçoit un minimum de 100 Mo au niveau du système, l’allocation de mémoire totale est de 6 000 Mo, ou juste en dessous de 6 Go.  

Certaines requêtes, telles que des grandes jointures ou des charges dans des tables columnstore en cluster, bénéficieront d’allocations de mémoire supérieures.  Certaines requêtes, comme les analyses pures, n’en tireront aucun bénéfice.  En revanche, l’utilisation de classes de ressource plus grandes réduit l’accès concurrentiel. Par conséquent, vous devez prendre cet impact en considération avant de déplacer tous les utilisateurs vers une grande classe de ressource.

Voir également [Classes de ressources pour la gestion des charges de travail](resource-classes-for-workload-management.md).

## <a name="use-smaller-resource-class-to-increase-concurrency"></a>Utiliser une classe ressource plus petite pour augmenter l’accès concurrentiel

Si vous constatez que les requêtes utilisateur semblent avoir un délai trop long, cela peut signifier que vos utilisateurs s’exécutent dans des classes de ressource plus grandes et consomment beaucoup d’emplacements de concurrence entraînant la mise en file d’attente des autres requêtes.  Pour voir si les requêtes des utilisateurs sont en attente, exécutez `SELECT * FROM sys.dm_pdw_waits` pour déterminer si des lignes sont renvoyées.

Voir également [Classes de ressources pour la gestion des charges de travail](resource-classes-for-workload-management.md), [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="other-resources"></a>Autres ressources

Consultez également notre article [Dépannage](sql-data-warehouse-troubleshoot.md) concernant les problèmes courants et leurs solutions.

Si vous ne trouvez pas ce que vous recherchez dans cet article, essayez d’utiliser la fonction de recherche de documents située sur le côté gauche de cette page pour rechercher tous les documents relatifs à Azure Synapse.  La [page de questions Microsoft Q&A sur Azure Synapse](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html) a été créée pour vous permettre de poser des questions à d’autres utilisateurs et au groupe de produits Azure Synapse. Nous suivons activement ce forum pour vous assurer que vos questions sont traitées par un autre utilisateur ou un membre de notre équipe.  

Si vous préférez poser vos questions sur Stack Overflow, nous avons également un [Forum Azure Synapse Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sqldw).

Enfin, utilisez la page des [commentaires relatifs à Azure Synapse](https://feedback.azure.com/forums/307516-sql-data-warehouse) afin de faire des demandes de fonctionnalités.  L’ajout de vos demandes ou la confirmation des autres demandes nous permet vraiment de hiérarchiser les fonctions.
