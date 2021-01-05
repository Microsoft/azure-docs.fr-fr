---
title: Meilleures pratiques de développement pour SQL Synapse
description: Recommandations et meilleures pratiques à connaître si vous développez pour SQL Synapse.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: a5e514602668c96d63562e45fb114cf9770a54a9
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321499"
---
# <a name="development-best-practices-for-synapse-sql"></a>Meilleures pratiques de développement pour SQL Synapse

Cet article fournit des conseils et décrit les bonnes pratiques à adopter quand vous développez votre solution d’entrepôt de données. 

## <a name="dedicated-sql-pool-development-best-practices"></a>Meilleures pratiques du développement de pools SQL dédiés

### <a name="reduce-cost-with-pause-and-scale"></a>Réduire les coûts avec les opérations de suspension et de mise à l’échelle

Pour plus d’informations sur la réduction des coûts à l’aide de la suspension et de la mise à l’échelle, consultez la section [Gérer le calcul](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="maintain-statistics"></a>Mettre à jour les statistiques

Veillez à mettre à jour vos statistiques quotidiennement ou après chaque chargement.  Lorsque vous créez et mettez des statistiques à jour, vous devez toujours faire un compromis entre les performances et les coûts. Si vous trouvez que la mise à jour de toutes vos statistiques prend trop de temps, sélectionnez les colonnes qui contiennent des statistiques ou qui nécessitent une mise à jour fréquente.  

Par exemple, vous pouvez mettre à jour des colonnes de date auxquelles de nouvelles valeurs peuvent être ajoutées quotidiennement. 

> [!NOTE]
> Vous tirerez le plus grand avantage de disposer de statistiques sur des colonnes impliquées dans des jointures, utilisées dans la clause WHERE et figurant dans GROUP BY.

Consultez également [Gestion des statistiques sur les tables](develop-tables-statistics.md), [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [UPDATE STATISTICS](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="hash-distribute-large-tables"></a>Hacher et distribuer de grandes tables

Par défaut, les tables sont distribuées par tourniquet (Round Robin). Cette fonctionnalité aide les utilisateurs à créer leurs tables sans avoir à déterminer comment celles-ci doivent être distribuées.  Des tables round robin peuvent être suffisamment performantes pour certaines charges de travail. Toutefois, dans la plupart des cas, la sélection d’une colonne de distribution sera bien plus performante.  

L’exemple le plus courant de meilleures performances observées avec une table distribuée par une colonne par rapport à une table Round Robin est lorsque deux grandes tables de faits sont jointes.  

Par exemple, si vous avez une table de commandes distribuée par order_id, et une table de transactions également distribuée par order_id, lorsque vous joignez votre table de commandes à votre table de transactions sur order_id, cette requête devient une requête directe. 

Cela signifie que nous éliminons les opérations de déplacement de données.  Moins d’étapes signifie une requête plus rapide.  Moins de déplacement des données permet également d’obtenir des requêtes plus rapides.

> [!TIP]
> Lors du chargement d’une table distribuée, assurez-vous que vos données entrantes ne sont pas triées sur la clé de distribution car cela ralentit vos charges.  

Consultez les liens suivants pour des explications plus détaillées sur la façon dont la sélection d’une colonne de distribution peut améliorer les performances, et pour apprendre à définir une table distribuée dans la clause WITH de l’instruction CREATE TABLES.

Voir aussi [Vue d’ensemble des tables](develop-tables-overview.md), [Distribution de tables](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), [Sélection d’une distribution de tables](https://blogs.msdn.microsoft.com/sqlcat/20../../choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service/), [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="do-not-over-partition"></a>Ne pas créer trop de partitions
Bien que le partitionnement des données peut être efficace pour mettre à jour vos données grâce au basculement de partitions ou à l’optimisation des analyses avec élimination des partitions, avoir un trop grand nombre de partitions peut ralentir vos requêtes.  Souvent une stratégie de partitionnement à granularité élevée qui peut fonctionner correctement sur SQL Server peut poser des problèmes sur un pool SQL dédié.  

> [!NOTE]
> Souvent une stratégie de partitionnement à granularité élevée qui peut fonctionner correctement sur SQL Server peut poser des problèmes sur un pool SQL dédié.  

Un trop grand nombre de partitions peut également réduire l’efficacité des index columnstore en cluster si chaque partition possède moins d’1 million de lignes. Le pool SQL dédié partitionne vos données pour vous en 60 bases de données. 

Ainsi, si vous créez une table contenant 100 partitions, vous obtenez 6000 bases de données.  Chaque charge de travail est différente ; par conséquent, le meilleur conseil serait d’expérimenter le partitionnement pour voir ce qui fonctionne le mieux pour votre charge de travail.  

L’une des options à prendre en compte est l’utilisation d’une granularité inférieure à celle qui a peut-être fonctionné pour vous dans SQL Server.  Par exemple, utilisez plutôt des partitions hebdomadaires ou mensuelles plutôt que des partitions quotidiennes.

Voir aussi [Partitionnement de table](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="minimize-transaction-sizes"></a>Minimiser la taille des transactions

Les instructions INSERT, UPDATE et DELETE s’exécutent dans une transaction, et en cas d’échec elles doivent être restaurées.  Pour minimiser le risque d’une restauration longue, réduisez si possible les tailles de transactions.  Pour ce faire, vous pouvez diviser les instructions INSERT, UPDATE et DELETE en plusieurs parties.  

Par exemple, si vous avez une instruction INSERT dont l’exécution devrait prendre une heure, vous pouvez la fractionner en quatre parties de 15 minutes.

> [!TIP]
> Exploitez des cas spéciaux de journalisation minimale, tels que CTAS, TRUNCATE, DROP TABLE ou INSERT, dans des tables vides, afin de réduire le risque de restauration.  

Un autre moyen d’éliminer les restaurations consiste à utiliser des opérations de métadonnées uniquement comme le basculement de partitions pour la gestion des données.  

Par exemple, plutôt que d’exécuter une instruction DELETE pour supprimer toutes les lignes d’une table où order_date était octobre 2001, vous pouvez partitionner vos données tous les mois et ensuite extraire la partition contenant les données vers une partition vide à partir d’une autre table (voir les exemples ALTER TABLE).  

Pour les tables non partitionnées, utilisez une instruction CTAS pour écrire les données que vous souhaitez conserver dans une table plutôt que l’instruction DELETE.  Si une instruction CTAS prend le même laps de temps, elle permet une opération beaucoup plus sûre car elle offre une journalisation des transactions minimale et peut être annulée rapidement si nécessaire.

Voir aussi [Transactions](develop-transactions.md), [Optimisation des transactions](../sql-data-warehouse/sql-data-warehouse-develop-best-practices-transactions.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), [Partitionnement de table](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), [TRUNCATE TABLE](/sql/t-sql/statements/truncate-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [Create table as select (CTAS)](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="use-the-smallest-possible-column-size"></a>Utiliser la plus petite taille de colonne possible

Lorsque vous définissez votre DDL, l’utilisation du plus petit type de données prenant en charge vos données améliore les performances de la requête. Cette action est particulièrement importante pour les colonnes CHAR et VARCHAR.  

Si la valeur la plus longue dans une colonne est de 25 caractères, définissez la colonne en tant que VARCHAR(25).  Évitez de définir toutes les colonnes de caractères sur une grande longueur par défaut.  En outre, définissez des colonnes VARCHAR lorsque cela suffit, au lieu d’utiliser NVARCHAR.

Voir aussi [Vue d’ensemble des tables](develop-tables-overview.md), [Types de données des tables](develop-tables-data-types.md) et [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="optimize-clustered-columnstore-tables"></a>Optimiser les tables columnstore en clusters

Les index columnstore en cluster sont l’une des méthodes les plus efficaces pour stocker vos données dans un pool SQL dédié.  Par défaut, les tables du pool SQL dédié sont créées en tant que ColumnStore en cluster.  

Pour obtenir les meilleures performances pour les requêtes sur les tables columnstore, la qualité du segment est importante.  Lorsque les lignes sont écrites dans les tables columnstore avec une mémoire insuffisante, la qualité du segment columnstore peut être affectée.  

La qualité du segment peut être mesurée par le nombre de lignes dans un groupe de lignes compressé.  Consultez la section [Causes de la qualité médiocre des index columnstore](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#causes-of-poor-columnstore-index-quality) dans l’article [Index de table](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) pour obtenir des instructions étape par étape sur la détection et l’amélioration de la qualité de segment pour les tables columnstore en cluster.  

Étant donné que la qualité des segments columnstore est importante, nous vous conseillons d’utiliser des ID d’utilisateurs qui se trouvent dans la classe de ressource de moyenne ou grande taille pour le chargement des données. L’utilisation d’[unités DWU (Data Warehouse Unit)](resource-consumption-models.md) inférieures signifie que vous souhaitez assigner une classe de ressource plus grande à votre utilisateur de chargement.

Étant donné que les tables columnstore ne transmettent généralement pas de données dans un segment columnstore compressé s’il existe moins de 1 million de lignes par table et si chaque table du pool SQL dédié est partitionnée en 60 tables, les tables columnstore ne tireront aucun avantage d’une requête, sauf si la table comporte plus de 60 millions de lignes.  

> [!TIP]
> Pour les tables contenant moins de 60 millions lignes, il se peut qu’un index columnstore ne soit pas la solution optimale.  

En outre, si vous partitionnez vos données, sachez que chaque partition doit avoir 1 million de lignes pour bénéficier d’un index columnstore en cluster.  Si une table possède 100 partitions, elle devra avoir au moins 6 milliards de lignes pour bénéficier d’une banque de colonnes en cluster (60 distributions *100 partitions* 1 million de lignes).  

Si votre table ne possède pas 6 milliards de lignes, réduisez le nombre de partitions ou envisagez d’utiliser une table de segments de mémoire.  Il peut également être intéressant d’expérimenter pour voir si de meilleures performances peuvent être obtenues en utilisant une table de segments de mémoire avec des index secondaires plutôt qu’avec une table columnstore.

Lorsque vous interrogez une table columnstore, les requêtes s’exécutent plus vite si vous sélectionnez uniquement les colonnes dont vous avez besoin.  

Consultez également [Index de table](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), [Guide des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et [Reconstruction des index columnstore](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#rebuilding-indexes-to-improve-segment-quality).

## <a name="serverless-sql-pool-development-best-practices"></a>Meilleures pratiques du développement de pools SQL serverless

### <a name="general-considerations"></a>Considérations d’ordre général

Un pool SQL serverless vous permet d’interroger des fichiers dans vos comptes de stockage Azure. Il ne dispose pas de capacités de stockage ou d’ingestion locales, ce qui signifie que tous les fichiers que la requête cible sont externes au pool SQL serverless. Par conséquent, tout ce qui est lié à la lecture de fichiers à partir du stockage peut avoir un impact sur les performances des requêtes.

### <a name="colocate-azure-storage-account-and-serverless-sql-pool"></a>Colocaliser un compte Stockage Azure et un pool SQL serverless

Pour réduire la latence, colocalisez votre compte de stockage Azure et le point de terminaison de votre pool SQL serverless. Les comptes de stockage et les points de terminaison approvisionnés lors de la création de l’espace de travail se trouvent dans la même région.

Pour optimiser les performances, si vous accédez à d’autres comptes de stockage avec le pool SQL serverless, assurez-vous qu’ils se trouvent dans la même région. Autrement, vous obtiendrez une latence accrue pour le transfert réseau des données de la région distante vers la région du point de terminaison.

### <a name="azure-storage-throttling"></a>Limitation de Stockage Azure

Plusieurs applications et services peuvent accéder à votre compte de stockage. Lorsque les IOPS ou le débit combinés générés par des applications, des services et la charge de travail du pool SQL serverless dépassent les limites du compte de stockage, une limitation du stockage se produit. Celle-ci a un impact négatif substantiel sur les performances des requêtes.

Une fois la limitation détectée, le pool SQL serverless gère ce scénario de manière intégrée. Le pool SQL serverless ralentit la fréquence des demandes adressées au stockage jusqu’à ce que la limitation soit résolue. 

Toutefois, pour optimiser l’exécution des requêtes, il est recommandé de ne pas solliciter le compte de stockage avec d’autres charges de travail lors de l’exécution de la requête.

### <a name="prepare-files-for-querying"></a>Préparer les fichiers pour l’interrogation

Si possible, vous pouvez préparer les fichiers pour améliorer les performances :

- Convertir CSV en Parquet – Parquet est un format en colonnes. Dans la mesure où il est compressé, les fichiers sont de plus petite taille que des fichiers CSV contenant les mêmes données, et le pool SQL serverless a besoin de moins de temps et de demandes de stockage pour le lire.
- Si une requête cible un seul fichier volumineux, il est avantageux de fractionner celui-ci en fichiers plus petits.
- Essayez de conserver une taille de fichier CSV inférieure à 10 Go.
- Il est préférable d’avoir des fichiers de taille identique pour un chemin d’accès OPENROWSET unique ou un emplacement de table externe.
- Partitionnez vos données en stockant des partitions dans différents dossiers ou noms de fichiers. Consultez [Utiliser les fonctions filename et filepath pour cibler des partitions spécifiques](#use-fileinfo-and-filepath-functions-to-target-specific-partitions).

### <a name="use-fileinfo-and-filepath-functions-to-target-specific-partitions"></a>Utiliser les fonctions filename et filepath pour cibler des partitions spécifiques

Les données sont souvent organisées en partitions. Vous pouvez demander au pool SQL serverless d’interroger des dossiers et fichiers particuliers. Cela contribue à réduire le nombre de fichiers et la quantité de données que la requête doit lire et traiter. 

Par conséquent, vous obtiendrez de meilleures performances. Pour plus d’informations, consultez les fonctions [filename](query-data-storage.md#filename-function) et [filepath](query-data-storage.md#filepath-function), ainsi que les exemples montrant comment [interroger des fichiers spécifiques](query-specific-files.md).

Si vos données dans le stockage ne sont pas partitionnées, envisagez de les partitionner afin de pouvoir utiliser ces fonctions pour optimiser les requêtes ciblant ces fichiers.

Lorsque vous [interrogez des tables externes partitionnées Apache Spark pour Azure Synapse](develop-storage-files-spark-tables.md) à partir d’un pool SQL serverless, la requête ne cible automatiquement que les fichiers nécessaires.

### <a name="use-cetas-to-enhance-query-performance-and-joins"></a>Utiliser CETAS pour améliorer les performances des requêtes et les jointures

[CETAS](develop-tables-cetas.md) est l’une des fonctionnalités les plus importantes disponibles dans le pool SQL serverless. CETAS est une opération parallèle qui crée des métadonnées de table externe et exporte le résultat de la requête SELECT vers un ensemble de fichiers dans votre compte de stockage.

Vous pouvez utiliser CETAS pour stocker dans un nouveau jeu de fichiers des parties de requêtes souvent utilisées, telles des tables de référence jointes. Par la suite, vous pouvez joindre une telle table externe au lieu de répéter des jointures communes dans plusieurs requêtes. 

Comme CETAS génère des fichiers Parquet, les statistiques sont automatiquement créées lorsque la première requête cible cette table externe, et les performances sont améliorées.

### <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’informations non fournies dans cet article, utilisez la fonction **Recherche de documents** sur le côté gauche de cette page pour effectuer une recherche dans tous les documents du pool SQL.  La [page de questions Microsoft Q&A sur Azure Synapse Analytics](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html) a été créée pour vous permettre de poser des questions à d’autres utilisateurs et au groupe de produits Azure Synapse Analytics. Nous suivons activement ce forum pour vous assurer que vos questions sont traitées par un autre utilisateur ou un membre de notre équipe.  

Si vous préférez poser vos questions sur Stack Overflow, nous avons également un [Forum Stack Overflow sur Azure Synapse Analytics](https://stackoverflow.com/questions/tagged/azure-sqldw).
 
