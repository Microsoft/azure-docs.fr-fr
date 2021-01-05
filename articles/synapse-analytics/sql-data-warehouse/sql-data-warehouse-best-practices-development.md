---
title: Bonnes pratiques pour le développement
description: Recommandations et meilleures pratiques que vous devez connaître pour développer des solutions pour le pool SQL Synapse.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 09/04/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: f8db19b209ddefebd4a297f18fb9b178fcf40f21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89457904"
---
# <a name="development-best-practices-for-synapse-sql-pool"></a>Meilleures pratiques de développement pour le pool SQL Synapse

Cet article fournit des conseils et décrit les meilleures pratiques à adopter quand vous développez votre solution de pool SQL.

## <a name="tune-query-performance-with-new-product-enhancements"></a>Optimiser les performances des requêtes avec de nouvelles améliorations produit

- [Réglage des performances avec des vues matérialisées](performance-tuning-materialized-views.md)
- [Réglage des performances avec un index columnstore en cluster ordonné](performance-tuning-ordered-cci.md)
- [Optimisation des performances avec la mise en cache des jeux de résultats](performance-tuning-result-set-caching.md)

## <a name="reduce-cost-with-pause-and-scale"></a>Réduire les coûts avec les opérations de suspension et de mise à l’échelle

Pour plus d’informations sur la réduction des coûts à l’aide de la suspension et de la mise à l’échelle, consultez l’article [Gérer le calcul](sql-data-warehouse-manage-compute-overview.md).

## <a name="maintain-statistics"></a>Mettre à jour les statistiques

Le pool SQL peut être configuré pour détecter et créer automatiquement des statistiques sur les colonnes.  Les plans de requête créés par l’optimiseur sont aussi bons que les statistiques disponibles.  

Nous vous recommandons d’activer AUTO_CREATE_STATISTICS pour vos bases de données et de conserver les statistiques mises à jour quotidiennement ou après chaque charge pour vous assurer que les statistiques sur les colonnes utilisées dans vos requêtes sont toujours à jour.

Si vous trouvez que la mise à jour de toutes vos statistiques prend trop de temps, vous souhaiterez peut-être sélectionner les colonnes nécessitant des mises à jour fréquentes des statistiques. Par exemple, vous pouvez mettre à jour des colonnes de date, où de nouvelles valeurs peuvent être ajoutées de façon quotidienne.

> [!TIP]
> Vous bénéficierez de performances optimales en lançant des mises à jour des statistiques sur les colonnes impliquées dans les jointures, celles utilisées dans la clause WHERE et celles figurant dans GROUP BY.

Voir aussi [Gestion des statistiques sur les tables](sql-data-warehouse-tables-statistics.md), [CREATE STATISTICS](sql-data-warehouse-tables-statistics.md) et [UPDATE STATISTICS](sql-data-warehouse-tables-statistics.md#update-statistics).

## <a name="hash-distribute-large-tables"></a>Hacher et distribuer de grandes tables

Par défaut, les tables sont distribuées par tourniquet (Round Robin).  Cette conception aide les utilisateurs à commencer la création de tables sans avoir à déterminer comment les tables doivent être distribuées.  

Les tables distribuées par tourniquet (Round Robin) peuvent offrir des performances suffisantes pour certaines charges de travail, mais souvent la sélection d’une colonne de distribution s’avérera plus efficace.  L’exemple le plus courant de meilleures performances observées avec une table distribuée par une colonne par rapport à une table Round Robin est lorsque deux grandes tables de faits sont jointes.  

Par exemple, si vous avez une table de commandes, qui est distribuée par order_id, et une table de transactions, également distribuée par order_id, lorsque vous joignez votre table de commandes à votre table de transactions sur order_id, cette requête devient une requête directe, ce qui signifie que nous éliminons les opérations de déplacement de données.  Moins d’étapes signifie une requête plus rapide.  Moins de déplacement des données permet également d’obtenir des requêtes plus rapides.  

Lors du chargement d’une table distribuée, assurez-vous que vos données entrantes ne sont pas triées sur la clé de distribution car cela ralentit vos charges.  Les articles suivants fournissent des détails supplémentaires sur l’amélioration des performances en sélectionnant une colonne de distribution et sur la façon de définir une table distribuée dans la clause WITH de l’instruction CREATE TABLES.

Voir aussi [Vue d’ensemble des tables](sql-data-warehouse-tables-overview.md), [Distribution de tables](sql-data-warehouse-tables-distribute.md), [Sélection d’une distribution de tables](https://blogs.msdn.microsoft.com/sqlcat/20../../choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service/), [CREATE TABLE](sql-data-warehouse-tables-overview.md) et [CREATE TABLE AS SELECT](sql-data-warehouse-develop-ctas.md).

## <a name="do-not-over-partition"></a>Ne pas créer trop de partitions

Bien que le partitionnement des données peut être efficace pour mettre à jour vos données grâce au basculement de partitions ou à l’optimisation des analyses avec élimination des partitions, avoir un trop grand nombre de partitions peut ralentir vos requêtes.  

Souvent, une stratégie de partitionnement à granularité élevée qui peut fonctionner correctement sur SQL Server peut poser des problèmes sur un pool SQL.  Un trop grand nombre de partitions peut également réduire l’efficacité des index columnstore en cluster si chaque partition possède moins d’1 million de lignes.  

N’oubliez pas que, en arrière-plan, le pool SQL partitionne vos données en 60 bases de données, donc si vous créez une table avec 100 partitions, cela produit en réalité 6 000 partitions.  Chaque charge de travail est différente ; par conséquent, le meilleur conseil serait d’expérimenter le partitionnement pour voir ce qui fonctionne le mieux pour votre charge de travail.  

> [!TIP]
> Envisagez d’utiliser un niveau de granularité inférieur à celui qui a pu être utilisé dans SQL Server.  Par exemple, utilisez plutôt des partitions hebdomadaires ou mensuelles plutôt que des partitions quotidiennes.

Voir aussi [Partitionnement de table](sql-data-warehouse-tables-partition.md).

## <a name="minimize-transaction-sizes"></a>Minimiser la taille des transactions

Les instructions INSERT, UPDATE et DELETE s’exécutent dans une transaction, et en cas d’échec elles doivent être restaurées.  Pour minimiser le risque d’une restauration longue, réduisez si possible les tailles de transactions.  Pour ce faire, vous pouvez diviser les instructions INSERT, UPDATE et DELETE en plusieurs parties.  

Par exemple, si vous disposez d’une instruction INSERT qui devrait prendre une heure, décomposez si possible INSERT en quatre parties, qui seront chacune exécutées en 15 minutes.  Exploitez des cas spéciaux de journalisation minimale, tels que CTAS, TRUNCATE, DROP TABLE ou INSERT, dans des tables vides, afin de réduire le risque de restauration.  

Un autre moyen d’éliminer les restaurations consiste à utiliser des opérations de métadonnées uniquement comme le basculement de partitions pour la gestion des données.  Par exemple, plutôt que d’exécuter une instruction DELETE pour supprimer toutes les lignes d’une table où order_date était octobre 2001, vous pouvez partitionner vos données tous les mois et ensuite extraire la partition contenant les données vers une partition vide à partir d’une autre table (voir les exemples ALTER TABLE).  

Pour les tables non partitionnées, utilisez une instruction CTAS pour écrire les données que vous souhaitez conserver dans une table plutôt que l’instruction DELETE.  Si une instruction CTAS prend le même laps de temps, elle permet une opération beaucoup plus sûre car elle offre une journalisation des transactions minimale et peut être annulée rapidement si nécessaire.

Voir aussi [Transactions](sql-data-warehouse-develop-transactions.md), [Optimisation des transactions](sql-data-warehouse-develop-best-practices-transactions.md), [Partitionnement de table](sql-data-warehouse-tables-partition.md), [TRUNCATE TABLE](/sql/t-sql/statements/truncate-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et [Create table as select (CTAS)](sql-data-warehouse-develop-ctas.md).

## <a name="use-the-smallest-possible-column-size"></a>Utiliser la plus petite taille de colonne possible

Lorsque vous définissez votre DDL, l’utilisation du plus petit type de données prenant en charge vos données améliore les performances de la requête.  Cette approche est particulièrement importante pour les colonnes CHAR et VARCHAR.  

Si la valeur la plus longue dans une colonne est de 25 caractères, définissez la colonne en tant que VARCHAR(25).  Évitez de définir toutes les colonnes de caractères sur une grande longueur par défaut.  En outre, définissez des colonnes VARCHAR lorsque cela suffit, au lieu d’utiliser NVARCHAR.

Voir aussi [Vue d’ensemble des tables](sql-data-warehouse-tables-overview.md), [Types de données des tables](sql-data-warehouse-tables-data-types.md) et [CREATE TABLE](sql-data-warehouse-tables-overview.md).

## <a name="optimize-clustered-columnstore-tables"></a>Optimiser les tables columnstore en clusters

Les index columnstore en cluster sont l’une des méthodes les plus efficaces pour stocker vos données dans le pool SQL.  Par défaut, les tables dans le pool SQL sont créées en tant que ColumnStore en cluster.  

> [!NOTE]
> Pour obtenir les meilleures performances pour les requêtes sur les tables columnstore, la qualité du segment est importante.  

Lorsque les lignes sont écrites dans les tables columnstore avec une mémoire insuffisante, la qualité du segment columnstore peut être affectée.  La qualité du segment peut être mesurée par le nombre de lignes dans un groupe de lignes compressé.  

Consultez la section [Causes de la qualité médiocre des index columnstore](sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality) dans l’article [Index de table](sql-data-warehouse-tables-index.md) pour obtenir des instructions étape par étape sur la détection et l’amélioration de la qualité de segment pour les tables columnstore en cluster.  

Étant donné que la qualité des segments columnstore est importante, nous vous conseillons d’utiliser des ID d’utilisateurs qui se trouvent dans la classe de ressource de moyenne ou grande taille pour le chargement des données. L’utilisation d’[unités DWU (Data Warehouse Unit)](what-is-a-data-warehouse-unit-dwu-cdwu.md) inférieures signifie que vous souhaitez assigner une classe de ressource plus grande à votre utilisateur de chargement.

Étant donné que les tables columnstore ne transmettent généralement pas de données dans un segment columnstore compressé s’il existe moins de 1 million de lignes par table et si chaque table du pool SQL est partitionnée en 60 tables, en règle générale, les tables columnstore ne tireront aucun avantage d’une requête, sauf si la table comporte plus de 60 millions de lignes.  

Pour une table comportant moins de 60 millions de lignes, il n’est pas judicieux d’avoir un index columnstore.  Mais cela ne peut pas nuire non plus.  

En outre, si vous partitionnez vos données, vous souhaiterez peut-être estimer que chaque partition nécessitera 1 million de lignes pour bénéficier d’un index columnstore en cluster.  Si une table possède 100 partitions, elle devra avoir au moins 6 milliards de lignes pour bénéficier d’une banque de colonnes en cluster (60 distributions *100 partitions* 1 million de lignes).  

Si votre table ne possède pas six milliards de lignes dans cet exemple, réduisez le nombre de partitions ou envisagez plutôt d’utiliser une table de segment de mémoire.  Il peut être également intéressant de tester pour voir si de meilleures performances peuvent être obtenues avec une table de segment de mémoire ayant des index secondaires plutôt qu’avec une table columnstore.

> [!TIP]
> Lorsque vous interrogez une table columnstore, les requêtes s’exécutent plus vite si vous sélectionnez uniquement les colonnes dont vous avez besoin.  

Voir aussi [Index de table](sql-data-warehouse-tables-index.md), [Guide des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et [Reconstruction des index columnstore](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality).

## <a name="next-steps"></a>Étapes suivantes

Si vous ne trouvez pas ce que vous recherchez dans cet article, essayez d’utiliser la fonction de recherche de documents située sur le côté gauche de cette page pour rechercher tous les documents relatifs à Azure Synapse.  

La [page de questions Microsoft Q&A sur Azure Synapse](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html) a été créée pour vous permettre de poser des questions à d’autres utilisateurs et au groupe de produits Azure Synapse.  Nous suivons activement ce forum pour vous assurer que vos questions sont traitées par un autre utilisateur ou un membre de notre équipe.  

Si vous préférez poser vos questions sur Stack Overflow, nous avons également un [Forum Stack Overflow sur Azure Synapse Analytics](https://stackoverflow.com/questions/tagged/azure-sqldw).

Utilisez la page des [commentaires relatifs à Azure Synapse](https://feedback.azure.com/forums/307516-sql-data-warehouse) pour effectuer des demandes de fonctionnalités.  L’ajout de vos demandes ou la confirmation des autres demandes nous permet vraiment de hiérarchiser les fonctions.
