---
title: Concevoir des tables à l’aide de SQL Synapse
description: Présentation de la conception de tables dans SQL Synapse.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 33eb5977ecb373a0dba87c26cacea247f541be8f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452735"
---
# <a name="design-tables-using-synapse-sql-in-azure-synapse-analytics"></a>Concevoir des tables avec Synapse SQL dans Azure Synapse Analytics

Ce document inclut des concepts clés liés à l’élaboration de tables avec un pool SQL dédié et un pool SQL serverless.  

Le [pool SQL serverless](on-demand-workspace-overview.md) est un service de requête sur les données de votre lac de données. Il ne dispose pas d’espace de stockage local pour l’ingestion de données. Le [pool SQL dédié](best-practices-sql-pool.md) représente une collection de ressources analytiques en cours de provisionnement quand Synapse SQL est utilisé. La taille du pool SQL dédié est déterminée par les unités d’entreposage de données (DWU).

Le tableau suivant liste les rubriques pertinentes pour le pool SQL dédié et le pool SQL serverless :

| Rubrique                                                        | Pool SQL dédié | Pool SQL serverless |
| ------------------------------------------------------------ | ------------------ | ----------------------- |
| [Déterminer la catégorie des tables](#determine-table-category)        | Oui                | Non                      |
| [Noms de schémas](#schema-names)                                | Oui                | Oui                     |
| [Noms de tables](#table-names)                                  | Oui                | Non                      |
| [Persistance de la table](#table-persistence)                      | Oui                | Non                      |
| [Table normale](#regular-table)                              | Oui                | Non                      |
| [Table temporaire](#temporary-table)                          | Oui                | Oui                     |
| [Table externe](#external-table)                            | Oui                | Oui                     |
| [Types de données](#data-types)                                    | Oui                | Oui                     |
| [Tables distribuées](#distributed-tables)                    | Oui                | Non                      |
| [Tables distribuées par hachage](#hash-distributed-tables)          | Oui                | Non                      |
| [Tables répliquées](#replicated-tables)                      | Oui                | Non                      |
| [Tables de type tourniquet](#round-robin-tables)                    | Oui                | Non                      |
| [Méthodes courantes de distribution pour les tables](#common-distribution-methods-for-tables) | Oui                | Non                      |
| [Partitions](#partitions)                                    | Oui                | Oui                     |
| [Index Columnstore](#columnstore-indexes)                  | Oui                | Non                      |
| [Statistiques](#statistics)                                    | Oui                | Oui                     |
| [Clé primaire et clé unique](#primary-key-and-unique-key)    | Oui                | Non                      |
| [Commandes pour la création de tables](#commands-for-creating-tables) | Oui                | Non                      |
| [Alignement des données sources avec l’entrepôt de données](#align-source-data-with-the-data-warehouse) | Oui                | Non                      |
| [Fonctionnalités de tables non prises en charge](#unsupported-table-features)    | Oui                | Non                      |
| [Requêtes de taille de table](#table-size-queries)                    | Oui                | Non                      |

## <a name="determine-table-category"></a>Déterminer la catégorie des tables

Un [schéma en étoile](https://en.wikipedia.org/wiki/Star_schema) organise les données dans des tables de faits et de dimension. Certaines tables sont utilisées pour l’intégration ou la mise en lots des données avant leur transfert dans une table de faits ou de dimension. Quand vous concevez une table, déterminez si les données de la table sont contenues dans une table de faits, de dimension ou d’intégration. Ceci est important pour choisir une structure et une distribution appropriées pour la table.

- Les **tables de faits** contiennent des données quantitatives qui sont le plus souvent générées dans un système transactionnel avant d’être chargées dans l’entrepôt de données. Par exemple, une entreprise de vente au détail génère des transactions de ventes chaque jour et charge ensuite ces données dans une table de faits de l’entrepôt de données pour les analyser.

- Les **tables de dimension** contiennent des données d’attribut modifiables, mais qui changent peu en règle générale. Par exemple, le nom et l’adresse d’un client sont stockés dans une table de dimension et sont mis à jour uniquement si le profil du client change. Pour réduire la taille d’une table de faits volumineuse, il est inutile d’indiquer le nom et l’adresse du client dans chaque ligne d’une table de faits. Au lieu de cela, la table de faits et la table de dimension peuvent partager un ID client. Vous pouvez alors créer une requête de jointure entre les deux tables pour associer le profil d’un client et les transactions qui le concernent.

- Les **tables d’intégration** fournissent un emplacement pour l’intégration ou la mise en lots des données. Vous pouvez créer une table d’intégration en tant que table normale, table externe ou table temporaire. Vous pouvez, par exemple, charger des données dans une table de mise en lots, effectuer des transformations sur ces données, puis insérer les données dans une table de production.

## <a name="schema-names"></a>Noms de schémas

Les schémas sont un bon moyen de regrouper des objets utilisés de manière similaire. Le code suivant crée un [schéma défini par l’utilisateur](/sql/t-sql/statements/create-schema-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) appelé wwi.

```sql
CREATE SCHEMA wwi;
```

## <a name="table-names"></a>Noms de tables

Si vous migrez plusieurs bases de données d’une solution locale vers un pool SQL dédié, la bonne pratique consiste à migrer toutes les tables de faits, de dimension et d’intégration dans un seul schéma de pool SQL. Par exemple, stockez toutes les tables de l’entrepôt de données [WideWorldImportersDW](/sql/samples/wide-world-importers-dw-database-catalog?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) dans un seul schéma appelé wwi.

Pour voir l’organisation des tables dans le pool SQL dédié, vous pouvez utiliser les préfixes fact, dim et int dans les noms de table. La table ci-dessous répertorie quelques noms de schéma et de table pour WideWorldImportersDW.  

| Table WideWorldImportersDW  | Type de la table | Pool SQL dédié |
|:-----|:-----|:------|:-----|
| City | Dimension | wwi.DimCity |
| JSON | Fact | wwi.FactOrder |

## <a name="table-persistence"></a>Persistance de la table

Les tables stockent les données soit de manière permanente ou temporaire dans Stockage Azure, soit dans un magasin de données externe à l’entrepôt de données.

### <a name="regular-table"></a>Table normale

Une table normale stocke les données dans le stockage Azure comme partie intégrante de l’entrepôt de données. La table et les données sont persistantes, qu’une session soit ouverte ou non.  L’exemple ci-après crée une table normale contenant deux colonnes.

```sql
CREATE TABLE MyTable (col1 int, col2 int );  
```

### <a name="temporary-table"></a>Table temporaire

Une table temporaire existe uniquement pendant la durée de la session. Vous pouvez utiliser une table temporaire pour empêcher d’autres utilisateurs de voir les résultats temporaires. L’utilisation de tables temporaires réduit également le besoin de nettoyage.  Les tables temporaires utilisent un stockage local et, dans les pools SQL dédiés, peuvent offrir des performances plus rapides.  

Le pool SQL serverless prend en charge les tables temporaires. Toutefois, son utilisation est limitée car, s’il permet d’opérer une sélection dans une table temporaire, il ne permet pas de la joindre à des fichiers dans le stockage.

Pour plus d’informations, consultez [Tables temporaires](develop-tables-temporary.md).

### <a name="external-table"></a>Table externe

Les [tables externes](develop-tables-external-tables.md) pointent vers des données situées dans Azure Storage Blob ou Azure Data Lake Store.

Importez des données de tables externes dans les pools SQL dédiés à l’aide de l’instruction [CREATE TABLE AS SELECT](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Pour obtenir un didacticiel sur le chargement, consultez [Utiliser PolyBase pour charger des données du Stockage Blob Azure](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Pour un pool SQL serverless, vous pouvez utiliser [CETAS](develop-tables-cetas.md) afin d’enregistrer le résultat de la requête dans une table externe du stockage Azure.

## <a name="data-types"></a>Types de données

Un pool SQL dédié prend en charge les types de données les plus couramment utilisés. Pour obtenir la liste des types de données pris en charge, consultez les [types de données dans la référence CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest#DataTypes&preserve-view=true) dans l’instruction CREATE TABLE. Pour plus d’informations sur l’utilisation des types de données, consultez [Types de données](../sql/develop-tables-data-types.md).

## <a name="distributed-tables"></a>Tables distribuées

Une fonctionnalité essentielle du pool SQL dédié est la manière dont il peut stocker et utiliser des tables sur des [distributions](../sql-data-warehouse/massively-parallel-processing-mpp-architecture.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#distributions).  Le pool SQL dédié prend en charge trois méthodes pour la distribution de données :

- Tourniquet (par défaut)
- Hachage
- Répliquée

### <a name="hash-distributed-tables"></a>Tables distribuées par hachage

Une table distribuée par hachage distribue les lignes de la table en fonction de la valeur dans la colonne de distribution. Elle offre les meilleures performances pour les requêtes sur des tables volumineuses. Il existe plusieurs facteurs à prendre en compte lors du choix d’une colonne de distribution.

Pour plus d’informations, consultez le [Guide de conception pour les tables distribuées](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="replicated-tables"></a>Tables répliquées

Les tables répliquées effectuent une copie complète de la table disponible sur chaque nœud de calcul. Les requêtes sur des tables répliquées s’exécutent rapidement, car les jointures ces tables ne nécessitent pas de déplacement de données. Toutefois, la réplication nécessitant plus de stockage, elle ne convient pas pour des tables volumineuses.

Pour plus d’informations, consultez [Guide de conception pour les tables répliquées](../sql-data-warehouse/design-guidance-for-replicated-tables.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="round-robin-tables"></a>Tables par tourniquet

Une table par tourniquet distribue les lignes de la table uniformément sur toutes les distributions. Les lignes sont distribuées de façon aléatoire. Le chargement des données dans une table par tourniquet se fait rapidement.  Cependant, les requêtes peuvent nécessiter davantage de déplacements de données que les autres méthodes de distribution.

Pour plus d’informations, consultez le [Guide de conception pour les tables distribuées](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="common-distribution-methods-for-tables"></a>Méthodes courantes de distribution pour les tables

La catégorie de table détermine souvent l’option optimale pour la distribution de table.

| Catégorie de table | Option de distribution recommandée |
|:---------------|:--------------------|
| Fact           | Utilisez la distribution par hachage avec un index columnstore cluster. Les performances sont meilleures quand deux tables de hachage sont jointes sur la même colonne de distribution. |
| Dimension      | Utilisez la réplication pour les tables de petite taille. Si les tables sont trop volumineuses pour être stockées sur chaque nœud de calcul, utilisez la distribution par hachage. |
| Staging        | Utilisez la distribution par tourniquet (round robin) pour la table de mise en lots. Le chargement avec la fonctionnalité CTAS est rapide. Une fois que vous avez mis les données dans la table de mise en lots, utilisez l’instruction INSERT…SELECT pour déplacer les données vers les tables de production. |

## <a name="partitions"></a>Partitions

Dans les pools SQL dédiés, une table partitionnée stocke les lignes de table et exécute des opérations sur celles-ci en fonction de plages de données. Par exemple, une table peut être partitionnée par jour, mois ou année. Vous pouvez améliorer les performances des requêtes via l’élimination de partition, qui limite l’analyse d’une requête aux seules données contenues dans une partition.

Vous pouvez également tenir à jour les données à l’aide du basculement de partition. Comme les données dans un pool SQL dédié sont déjà distribuées, un partitionnement excessif risque de ralentir les requêtes. Pour plus d’informations, consultez [Partitionnement de tables](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).  

> [!TIP]
> Lorsque le basculement de partitions se fait vers des partitions de table qui ne sont pas vides, pensez à utiliser l’option TRUNCATE_TARGET dans votre instruction [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) si les données existantes sont le point d’être tronquées.

Le code ci-dessous convertit les données quotidiennes transformées en une partition SalesFact et remplace toutes les données existantes.

```sql
ALTER TABLE SalesFact_DailyFinalLoad SWITCH PARTITION 256 TO SalesFact PARTITION 256 WITH (TRUNCATE_TARGET = ON);  
```

Dans un pool SQL serverless, vous pouvez limiter les fichiers/dossiers (partitions) lus par votre requête. Un partitionnement par chemin d’accès est pris en charge à l’aide des fonctions filepath et fileinfo décrites dans [Interrogation de fichiers de stockage](develop-storage-files-overview.md). L’exemple suivant lit un dossier contenant des données de l’année 2017 :

```sql
SELECT
    nyc.filepath(1) AS [year],
    payment_type,
    SUM(fare_amount) AS fare_total
FROM  
    OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS nyc
WHERE
    nyc.filepath(1) = 2017
GROUP BY
    nyc.filepath(1),
    payment_type
ORDER BY
    nyc.filepath(1),
    payment_type
```

## <a name="columnstore-indexes"></a>Index columnstore

Par défaut, le pool SQL dédié stocke une table sous la forme d’un index columnstore cluster. Ce format de stockage de données permet une compression élevée des données et offre des performances optimales pour les requêtes sur des tables volumineuses.  L’index columnstore cluster est généralement le meilleur choix, mais dans certains cas, un index cluster ou un segment de mémoire est la structure de stockage la plus appropriée.  

> [!TIP]
> Une table de segments de mémoire peut être particulièrement utile pour le chargement des données temporaires, comme une table de mise en lots qui est transformée en table finale.

Pour obtenir la liste des fonctionnalités columnstore, consultez [Nouveautés pour les index columnstore](/sql/relational-databases/indexes/columnstore-indexes-what-s-new?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Pour améliorer les performances des index columnstore, consultez [Optimiser la qualité du rowgroup pour les index columnstore](../sql/data-load-columnstore-compression.md).

## <a name="statistics"></a>Statistiques

L’optimiseur de requête utilise des statistiques au niveau des colonnes quand il crée le plan d’exécution d’une requête. Pour améliorer les performances des requêtes, il est important d’avoir des statistiques sur des colonnes individuelles, en particulier les colonnes utilisées dans les jointures de requête. SQL Synapse prend en charge la création automatique de statistiques. 

La mise à jour statistique ne se fait pas automatiquement. Mettez à jour les statistiques après l’ajout ou la modification d’un nombre significatif de lignes. Par exemple, effectuez une mise à jour des statistiques après un chargement. Pour plus d’informations, consultez l’article sur les [statistiques dans SQL Synapse](develop-tables-statistics.md).

## <a name="primary-key-and-unique-key"></a>Clé primaire et clé unique

Pour un pool SQL dédié, la contrainte PRIMARY KEY est prise en charge seulement si NONCLUSTERED et NOT ENFORCED sont tous les deux utilisés.  La contrainte UNIQUE n’est prise en charge que si NOT ENFORCED est utilisé.  Pour plus d’informations, consultez l’article [Contraintes de table du pool SQL](../sql-data-warehouse/sql-data-warehouse-table-constraints.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="commands-for-creating-tables"></a>Commandes pour la création de tables

Pour un pool SQL dédié, vous pouvez créer une table comme nouvelle table vide. Vous pouvez aussi créer une table et la remplir avec les résultats d’une instruction select. Le tableau suivant répertorie les instructions T-SQL disponibles pour la création d’une table.

| Instruction T-SQL | Description |
|:----------------|:------------|
| [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Crée une table vide en définissant toutes les colonnes et options de la table. |
| [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Crée une table externe. La définition de la table est stockée dans le pool SQL dédié. Les données de la table sont stockées dans Stockage Blob Azure ou Azure Data Lake Store. |
| [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Crée une table et la remplit avec les résultats d’une instruction select. Les colonnes et les types de données de la table sont basés sur les résultats de l’instruction select. Pour importer des données, cette instruction peut sélectionner les données dans une table externe. |
| [CREATE EXTERNAL TABLE AS SELECT](/sql/t-sql/statements/create-external-table-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Crée une table externe en exportant les résultats d’une instruction select vers un emplacement externe.  L’emplacement est Stockage Blob Azure ou Azure Data Lake Store. |

## <a name="align-source-data-with-the-data-warehouse"></a>Aligner les données sources avec l’entrepôt de données

Les tables du pool SQL dédié sont remplies avec les données chargées à partir d’une autre source de données. Pour accomplir un chargement correct, le nombre et les types de données des colonnes dans les données sources doivent être alignés sur la définition de la table dans l’entrepôt de données.

> [!NOTE]
> L’alignement des données est parfois l’étape la plus difficile dans la conception des tables.

Si les données proviennent de plusieurs magasins, vous pouvez les porter dans l’entrepôt de données et les stocker dans une table d’intégration. Vous pouvez ensuite effectuer des opérations de transformation sur les données de la table d’intégration en tirant parti de la puissance du pool SQL dédié. Une fois que les données sont préparées, vous pouvez les insérer dans des tables de production.

## <a name="unsupported-table-features"></a>Fonctionnalités de table non prises en charge

Le pool SQL dédié prend en charge bon nombre des fonctionnalités de table proposées par d’autres bases de données.  La liste suivante répertorie certaines fonctionnalités de table qui ne sont pas prises en charge dans le pool SQL dédié.

- Clé étrangère, voir [Contraintes de table](/sql/t-sql/statements/alter-table-table-constraint-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Colonnes calculées](/sql/t-sql/statements/alter-table-computed-column-definition-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Vues indexées](/sql/relational-databases/views/create-indexed-views?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Séquence](/sql/t-sql/statements/create-sequence-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Colonnes éparses](/sql/relational-databases/tables/use-sparse-columns?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- Clés de substitution, implémenter avec [Identité](../sql-data-warehouse/sql-data-warehouse-tables-identity.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Synonymes](/sql/t-sql/statements/create-synonym-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Déclencheurs](/sql/t-sql/statements/create-trigger-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Index uniques](/sql/t-sql/statements/create-index-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Types définis par l’utilisateur](/sql/relational-databases/native-client/features/using-user-defined-types?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## <a name="table-size-queries"></a>Requêtes de taille de table

Dans un pool SQL dédié, un moyen simple d’identifier l’espace et les lignes consommés par une table dans chacune des 60 distributions consiste à utiliser [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

```sql
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

Gardez à l’esprit le fait que l’utilisation des commandes DBCC peut être très limitante.  Les vues de gestion dynamique (DMV) affichent des informations plus détaillées que les commandes DBCC. Commencez par créer l’affichage ci-dessous.

```sql
CREATE VIEW dbo.vTableSizes
AS
WITH base
AS
(
SELECT
 GETDATE()                                                             AS  [execution_time]
, DB_NAME()                                                            AS  [database_name]
, s.name                                                               AS  [schema_name]
, t.name                                                               AS  [table_name]
, QUOTENAME(s.name)+'.'+QUOTENAME(t.name)                              AS  [two_part_name]
, nt.[name]                                                            AS  [node_table_name]
, ROW_NUMBER() OVER(PARTITION BY nt.[name] ORDER BY (SELECT NULL))     AS  [node_table_name_seq]
, tp.[distribution_policy_desc]                                        AS  [distribution_policy_name]
, c.[name]                                                             AS  [distribution_column]
, nt.[distribution_id]                                                 AS  [distribution_id]
, i.[type]                                                             AS  [index_type]
, i.[type_desc]                                                        AS  [index_type_desc]
, nt.[pdw_node_id]                                                     AS  [pdw_node_id]
, pn.[type]                                                            AS  [pdw_node_type]
, pn.[name]                                                            AS  [pdw_node_name]
, di.name                                                              AS  [dist_name]
, di.position                                                          AS  [dist_position]
, nps.[partition_number]                                               AS  [partition_nmbr]
, nps.[reserved_page_count]                                            AS  [reserved_space_page_count]
, nps.[reserved_page_count] - nps.[used_page_count]                    AS  [unused_space_page_count]
, nps.[in_row_data_page_count]
    + nps.[row_overflow_used_page_count]
    + nps.[lob_used_page_count]                                        AS  [data_space_page_count]
, nps.[reserved_page_count]
 - (nps.[reserved_page_count] - nps.[used_page_count])
 - ([in_row_data_page_count]
         + [row_overflow_used_page_count]+[lob_used_page_count])       AS  [index_space_page_count]
, nps.[row_count]                                                      AS  [row_count]
from
    sys.schemas s
INNER JOIN sys.tables t
    ON s.[schema_id] = t.[schema_id]
INNER JOIN sys.indexes i
    ON  t.[object_id] = i.[object_id]
    AND i.[index_id] <= 1
INNER JOIN sys.pdw_table_distribution_properties tp
    ON t.[object_id] = tp.[object_id]
INNER JOIN sys.pdw_table_mappings tm
    ON t.[object_id] = tm.[object_id]
INNER JOIN sys.pdw_nodes_tables nt
    ON tm.[physical_name] = nt.[name]
INNER JOIN sys.dm_pdw_nodes pn
    ON  nt.[pdw_node_id] = pn.[pdw_node_id]
INNER JOIN sys.pdw_distributions di
    ON  nt.[distribution_id] = di.[distribution_id]
INNER JOIN sys.dm_pdw_nodes_db_partition_stats nps
    ON nt.[object_id] = nps.[object_id]
    AND nt.[pdw_node_id] = nps.[pdw_node_id]
    AND nt.[distribution_id] = nps.[distribution_id]
LEFT OUTER JOIN (select * from sys.pdw_column_distribution_properties where distribution_ordinal = 1) cdp
    ON t.[object_id] = cdp.[object_id]
LEFT OUTER JOIN sys.columns c
    ON cdp.[object_id] = c.[object_id]
    AND cdp.[column_id] = c.[column_id]
WHERE pn.[type] = 'COMPUTE'
)
, size
AS
(
SELECT
   [execution_time]
,  [database_name]
,  [schema_name]
,  [table_name]
,  [two_part_name]
,  [node_table_name]
,  [node_table_name_seq]
,  [distribution_policy_name]
,  [distribution_column]
,  [distribution_id]
,  [index_type]
,  [index_type_desc]
,  [pdw_node_id]
,  [pdw_node_type]
,  [pdw_node_name]
,  [dist_name]
,  [dist_position]
,  [partition_nmbr]
,  [reserved_space_page_count]
,  [unused_space_page_count]
,  [data_space_page_count]
,  [index_space_page_count]
,  [row_count]
,  ([reserved_space_page_count] * 8.0)                                 AS [reserved_space_KB]
,  ([reserved_space_page_count] * 8.0)/1000                            AS [reserved_space_MB]
,  ([reserved_space_page_count] * 8.0)/1000000                         AS [reserved_space_GB]
,  ([reserved_space_page_count] * 8.0)/1000000000                      AS [reserved_space_TB]
,  ([unused_space_page_count]   * 8.0)                                 AS [unused_space_KB]
,  ([unused_space_page_count]   * 8.0)/1000                            AS [unused_space_MB]
,  ([unused_space_page_count]   * 8.0)/1000000                         AS [unused_space_GB]
,  ([unused_space_page_count]   * 8.0)/1000000000                      AS [unused_space_TB]
,  ([data_space_page_count]     * 8.0)                                 AS [data_space_KB]
,  ([data_space_page_count]     * 8.0)/1000                            AS [data_space_MB]
,  ([data_space_page_count]     * 8.0)/1000000                         AS [data_space_GB]
,  ([data_space_page_count]     * 8.0)/1000000000                      AS [data_space_TB]
,  ([index_space_page_count]  * 8.0)                                   AS [index_space_KB]
,  ([index_space_page_count]  * 8.0)/1000                              AS [index_space_MB]
,  ([index_space_page_count]  * 8.0)/1000000                           AS [index_space_GB]
,  ([index_space_page_count]  * 8.0)/1000000000                        AS [index_space_TB]
FROM base
)
SELECT *
FROM size
;
```

### <a name="table-space-summary"></a>Résumé de l’espace de table

Cette requête renvoie les lignes et l’espace par table.  Le résumé de l’espace de table vous permet de voir quelles sont vos tables les plus volumineuses. Vous pouvez également voir si elles sont de type tourniquet (Round Robin), répliquées ou distribuées par hachage.  Pour les tables distribuées par hachage, la requête affiche la colonne de distribution.  

```sql
SELECT
     database_name
,    schema_name
,    table_name
,    distribution_policy_name
,      distribution_column
,    index_type_desc
,    COUNT(distinct partition_nmbr) as nbr_partitions
,    SUM(row_count)                 as table_row_count
,    SUM(reserved_space_GB)         as table_reserved_space_GB
,    SUM(data_space_GB)             as table_data_space_GB
,    SUM(index_space_GB)            as table_index_space_GB
,    SUM(unused_space_GB)           as table_unused_space_GB
FROM
    dbo.vTableSizes
GROUP BY
     database_name
,    schema_name
,    table_name
,    distribution_policy_name
,      distribution_column
,    index_type_desc
ORDER BY
    table_reserved_space_GB desc
;
```

### <a name="table-space-by-distribution-type"></a>Espace de table par type de distribution

```sql
SELECT
     distribution_policy_name
,    SUM(row_count)                as table_type_row_count
,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
,    SUM(data_space_GB)            as table_type_data_space_GB
,    SUM(index_space_GB)           as table_type_index_space_GB
,    SUM(unused_space_GB)          as table_type_unused_space_GB
FROM dbo.vTableSizes
GROUP BY distribution_policy_name
;
```

### <a name="table-space-by-index-type"></a>Espace de table par type d’index

```sql
SELECT
     index_type_desc
,    SUM(row_count)                as table_type_row_count
,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
,    SUM(data_space_GB)            as table_type_data_space_GB
,    SUM(index_space_GB)           as table_type_index_space_GB
,    SUM(unused_space_GB)          as table_type_unused_space_GB
FROM dbo.vTableSizes
GROUP BY index_type_desc
;
```

### <a name="distribution-space-summary"></a>Résumé de l’espace de distribution

```sql
SELECT
    distribution_id
,    SUM(row_count)                as total_node_distribution_row_count
,    SUM(reserved_space_MB)        as total_node_distribution_reserved_space_MB
,    SUM(data_space_MB)            as total_node_distribution_data_space_MB
,    SUM(index_space_MB)           as total_node_distribution_index_space_MB
,    SUM(unused_space_MB)          as total_node_distribution_unused_space_MB
FROM dbo.vTableSizes
GROUP BY     distribution_id
ORDER BY    distribution_id
;
```

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé les tables dans votre entrepôt de données, l’étape suivante va être de charger des données dans ces tables.  Pour suivre un tutoriel sur le chargement, consultez [Chargement de données dans le pool SQL dédié](../sql-data-warehouse/load-data-wideworldimportersdw.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#load-the-data-into-sql-pool).
