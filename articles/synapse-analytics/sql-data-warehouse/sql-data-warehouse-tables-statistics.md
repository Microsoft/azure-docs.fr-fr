---
title: Création et mise à jour des statistiques sur des tables
description: Recommandations et exemples de création et de mise à jour des statistiques d’optimisation des requêtes sur les tables dans le pool SQL dédié.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 05/09/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: e7fc89dcc0e7938ea2958d5c804abe82e20f186d
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447935"
---
# <a name="table-statistics-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Statistiques des tables pour le pool SQL dédié dans Azure Synapse Analytics

Dans cet article, vous trouverez des recommandations et des exemples de création et de mise à jour des statistiques d’optimisation des requêtes sur les tables dans le pool SQL dédié.

## <a name="why-use-statistics"></a>Pourquoi utiliser des statistiques ?

Plus le pool SQL dédié connaît vos données, plus vite il peut exécuter des requêtes dessus. Après le chargement des données dans le pool SQL dédié, collecter des statistiques sur ces données constitue l’une des actions les plus importantes à effectuer pour optimiser les requêtes.

L’optimiseur de requête du pool SQL dédié s’appuie sur les coûts. Il compare le coût de différents plans de requête, puis choisit le plan avec le coût le plus bas. Dans la plupart des cas, il choisit le plan qui s’exécute le plus rapidement.

Par exemple, si l’optimiseur estime que la date de filtrage de votre requête va renvoyer une ligne, il choisira un certain plan. S’il estime que la date sélectionnée va renvoyer un million de lignes, il choisira un autre plan.

## <a name="automatic-creation-of-statistic"></a>Création automatique de statistiques

Lorsque l’option AUTO_CREATE_STATISTICS de la base de données est activée, le pool SQL dédié analyse les requêtes utilisateur entrantes à la recherche de statistiques manquantes.

Si des statistiques manquent, l’optimiseur de requête crée des statistiques sur des colonnes individuelles dans le prédicat de requête ou la condition de jointure afin d’améliorer les estimations de cardinalité pour le plan de requête.

> [!NOTE]
> La création automatique de statistiques est activée par défaut.

Vous pouvez vérifier si l’option AUTO_CREATE_STATISTICS est configurée pour votre pool SQL dédié en exécutant la commande suivante :

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

Si la propriété AUTO_CREATE_STATISTICS n’est pas configurée pour votre pool SQL dédié, nous vous recommandons de l’activer en exécutant la commande suivante :

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

Les instructions suivantes déclenchent la création automatique de statistiques :

- SELECT
- INSERT-SELECT
- CTAS
- UPDATE
- Suppression
- EXPLAIN quand elles contiennent une jointure ou que la présence d’un prédicat est détectée

> [!NOTE]
> La création automatique de statistiques ne porte pas sur les tables temporaires ou externes.

La création automatique de statistiques étant effectuée de façon synchrone, les performances des requêtes risquent de subir une légère détérioration si des statistiques manquent dans vos colonnes. La durée de création de statistiques pour une seule colonne dépend de la taille de la table.

Pour éviter toute détérioration notable des performances, vous devez vérifier que les statistiques ont été créées en exécutant la charge de travail du test d’évaluation avant de profiler le système.

> [!NOTE]
> La création de statistiques est journalisée dans [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) sous un contexte utilisateur différent.

Quand des statistiques automatiques sont créées, elles prennent la forme suivante : _WA_Sys_<ID de colonne de 8 chiffres en notation hexadécimale>_<ID de table de 8 chiffres en notation hexadécimale>. Vous pouvez visualiser les statistiques qui ont déjà été créées en exécutant la commande [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) :

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

table_name spécifie le nom de la table contenant les statistiques à afficher. La table ne peut pas être une table externe. La cible est le nom de l’index, des statistiques ou de la colonne cibles pour lesquelles afficher des informations statistiques.

## <a name="update-statistics"></a>Mettre à jour les statistiques

Une meilleure pratique consiste à mettre à jour les statistiques sur les colonnes de date à chaque fois qu’une date est ajoutée. Chaque fois que de nouvelles lignes sont chargées dans le pool SQL dédié, de nouvelles dates de chargement ou de transaction sont ajoutées. Ces ajouts affectent la distribution des données et rendent les statistiques obsolètes.

Vous n’aurez peut-être jamais à mettre à jour les statistiques d’une colonne de pays/région dans une table des clients, car la distribution des valeurs change rarement. Si l’on part du principe que la distribution des données est constante d’un client à l’autre, l’ajout de nouvelles lignes dans une table ne devrait pas affecter cette distribution.

Toutefois, si votre pool SQL dédié ne contient qu’un seul pays/une seule région et que vous importez des données d’un nouveau pays/d’une nouvelle région (en conséquence de quoi sont stockées des données de plusieurs pays/régions), vous devez mettre à jour les statistiques de la colonne relative au pays/à la région.

Voici certaines recommandations pour la mise à jour des statistiques :

|||
|-|-|
| **Fréquence des mises à jour des statistiques**  | Classique : Quotidien </br> Après le chargement ou la transformation de données |
| **Échantillonnage** |  Pour moins d’un milliard de lignes, utilisez le taux d’échantillonnage par défaut (20 %). </br> Pour plus d’un milliard de lignes, utilisez le taux d’échantillonnage de 2 %. |

L’une des premières questions que vous devez vous poser quand vous dépannez une requête est la suivante : **« Les statistiques sont-elles à jour ? »**

Or, vous ne pouvez pas répondre à cette question en vous appuyant sur l’âge des données. Un objet de statistiques à jour peut être ancien si aucune modification notable n’affecte les données sous-jacentes. Quand le nombre de lignes ou la distribution des valeurs change de manière substantielle dans une colonne, *alors* il est temps de mettre à jour les statistiques. 

Il n’existe aucune vue de gestion dynamique pour déterminer si les données de la table ont changé depuis la dernière mise à jour des statistiques.  Les deux requêtes suivantes peuvent vous aider à déterminer si vos statistiques sont obsolètes.

**Requête 1 :**  Déterminez la différence entre le nombre de lignes des statistiques (**stats_row_count**) et le nombre réel de lignes (**actual_row_count**). 

```sql
select 
objIdsWithStats.[object_id], 
actualRowCounts.[schema], 
actualRowCounts.logical_table_name, 
statsRowCounts.stats_row_count, 
actualRowCounts.actual_row_count,
row_count_difference = CASE
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN actualRowCounts.actual_row_count - statsRowCounts.stats_row_count
    ELSE statsRowCounts.stats_row_count - actualRowCounts.actual_row_count
END,
percent_deviation_from_actual = CASE
    WHEN actualRowCounts.actual_row_count = 0 THEN statsRowCounts.stats_row_count
    WHEN statsRowCounts.stats_row_count = 0 THEN actualRowCounts.actual_row_count
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (actualRowCounts.actual_row_count - statsRowCounts.stats_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
    ELSE CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (statsRowCounts.stats_row_count - actualRowCounts.actual_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
END
from
(
    select distinct object_id from sys.stats where stats_id > 1
) objIdsWithStats
left join
(
    select object_id, sum(rows) as stats_row_count from sys.partitions group by object_id
) statsRowCounts
on objIdsWithStats.object_id = statsRowCounts.object_id 
left join
(
    SELECT sm.name [schema] ,
    tb.name logical_table_name ,
    tb.object_id object_id ,
    SUM(rg.row_count) actual_row_count
    FROM sys.schemas sm
    INNER JOIN sys.tables tb ON sm.schema_id = tb.schema_id
    INNER JOIN sys.pdw_table_mappings mp ON tb.object_id = mp.object_id
    INNER JOIN sys.pdw_nodes_tables nt ON nt.name = mp.physical_name
    INNER JOIN sys.dm_pdw_nodes_db_partition_stats rg
    ON rg.object_id = nt.object_id
    AND rg.pdw_node_id = nt.pdw_node_id
    AND rg.distribution_id = nt.distribution_id
    WHERE 1 = 1
    GROUP BY sm.name, tb.name, tb.object_id
) actualRowCounts
on objIdsWithStats.object_id = actualRowCounts.object_id

```

**Requête 2 :** Déterminez l’ancienneté de vos statistiques en vérifiant à quand remonte la dernière mise à jour de vos statistiques sur chaque table. 

> [!NOTE]
> Si la distribution des valeurs d’une colonne a subi une modification significative, vous devez mettre à jour les statistiques, quelle que soit la date de la dernière mise à jour.

```sql
SELECT
    sm.[name] AS [schema_name],
    tb.[name] AS [table_name],
    co.[name] AS [stats_column_name],
    st.[name] AS [stats_name],
    STATS_DATE(st.[object_id],st.[stats_id]) AS [stats_last_updated_date]
FROM
    sys.objects ob
    JOIN sys.stats st
        ON  ob.[object_id] = st.[object_id]
    JOIN sys.stats_columns sc
        ON  st.[stats_id] = sc.[stats_id]
        AND st.[object_id] = sc.[object_id]
    JOIN sys.columns co
        ON  sc.[column_id] = co.[column_id]
        AND sc.[object_id] = co.[object_id]
    JOIN sys.types  ty
        ON  co.[user_type_id] = ty.[user_type_id]
    JOIN sys.tables tb
        ON  co.[object_id] = tb.[object_id]
    JOIN sys.schemas sm
        ON  tb.[schema_id] = sm.[schema_id]
WHERE
    st.[user_created] = 1;
```

Par exemple, les statistiques des **colonnes de date** d’un pool SQL dédié doivent être mises à jour fréquemment. Chaque fois que de nouvelles lignes sont chargées dans le pool SQL dédié, de nouvelles dates de chargement ou de transaction sont ajoutées. Ces ajouts affectent la distribution des données et rendent les statistiques obsolètes.

À l’inverse, les statistiques d’une colonne indiquant le sexe d’un client dans une table n’auront peut-être jamais besoin d’être mises à jour. Si l’on part du principe que la distribution des données est constante d’un client à l’autre, l’ajout de nouvelles lignes dans une table ne devrait pas affecter cette distribution.

Si votre pool SQL dédié ne contient qu’un seul sexe alors qu’une nouvelle exigence impose le recours à plusieurs sexes, vous devez mettre à jour les statistiques de la colonne relative au sexe.

Pour plus d’informations, consultez les conseils généraux sur les [statistiques](/sql/relational-databases/statistics/statistics?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="implementing-statistics-management"></a>Implémentation de fonctions de gestion des statistiques

Il est souvent judicieux d’étendre le processus de chargement des données afin de vérifier que les statistiques sont mises à jour à la fin du chargement, et ce pour éviter ou minimiser le blocage ou la contention des ressources entre les requêtes simultanées.  

Le chargement des données se produit lorsque la taille et/ou la distribution des valeurs sont souvent modifiées dans les tables. Le chargement des données est un emplacement logique permettant d’implémenter certains processus de gestion.

Les principes généraux suivants sont fournis afin de vous aider à mettre à jour vos statistiques :

- Assurez-vous que chaque table chargée présente au moins un objet de statistiques mis à jour. Cela met à jour les informations sur la taille des tables (nombre de lignes et de pages) dans le cadre du processus de mise à jour des statistiques.
- Concentrez-vous sur les colonnes participant aux clauses JOIN, GROUP BY, ORDER BY et DISTINCT.
- Envisagez de mettre plus souvent à jour les colonnes de clé croissante, comme celles des dates de transactions, car ces valeurs ne seront pas incluses dans l’histogramme des statistiques.
- Envisagez de mettre moins souvent à jour les colonnes de distribution statiques.
- N’oubliez pas que chaque objet de statistiques est mis à jour dans l’ordre. L’implémentation de l’élément `UPDATE STATISTICS <TABLE_NAME>` ne suffit pas toujours, notamment quand les tables sont volumineuses et incluent un grand nombre d’objets de statistiques.

Pour plus d’informations, consultez [Évaluation de la cardinalité](/sql/relational-databases/performance/cardinality-estimation-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="examples-create-statistics"></a>Exemples : Create statistics

Ces exemples indiquent comment utiliser différentes options pour créer des statistiques. Les options à utiliser pour chaque colonne dépendent des caractéristiques de vos données et de l’utilisation de la colonne dans les requêtes.

### <a name="create-single-column-statistics-with-default-options"></a>Créer des statistiques sur une colonne en utilisant les options par défaut

Pour créer des statistiques sur une colonne, indiquez le nom de l’objet de statistiques ainsi que celui de la colonne.

Cette syntaxe a recours à toutes les options par défaut. Par défaut, **20 %** de la table est échantillonné lors de la création de statistiques.

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]);
```

Par exemple :

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1);
```

### <a name="create-single-column-statistics-by-examining-every-row"></a>Créer des statistiques sur plusieurs colonnes en examinant chaque ligne

Le taux d’échantillonnage par défaut est de 20 %, ce qui est suffisant pour la plupart des situations. Toutefois, vous pouvez l’ajuster en fonction de vos besoins.

Pour échantillonner la table entière, utilisez la syntaxe suivante :

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]) WITH FULLSCAN;
```

Par exemple :

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH FULLSCAN;
```

### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Créer des statistiques sur une colonne en spécifiant la taille de l’échantillon

Vous pouvez également spécifier cette taille sous la forme d’un pourcentage :

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH SAMPLE = 50 PERCENT;
```

### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Créer des statistiques sur une colonne sur certaines lignes uniquement

Vous pouvez également créer des statistiques sur une partie des lignes de votre table. On parle alors de « statistiques filtrées ».

Par exemple, vous pouvez utiliser les statistiques filtrées quand vous prévoyez d’interroger une partition spécifique dans une table partitionnée volumineuse. En créant des statistiques sur les valeurs des partitions uniquement, vous améliorez la précision des statistiques et, par conséquent, les performances des requêtes.

Dans cet exemple, des statistiques sont créées sur une plage de valeurs. Les valeurs peuvent facilement être définies de manière à correspondre à la plage de valeurs d’une partition.

```sql
CREATE STATISTICS stats_col1 ON table1(col1) WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Pour que l’optimiseur de requête envisage d’utiliser les statistiques filtrées lorsqu’il choisit le plan de requête distribuée, il faut que cette requête soit suffisamment petite pour pouvoir s’insérer dans la définition de l’objet de statistiques. Selon l’exemple précédent, la clause WHERE de la requête doit indiquer des valeurs incluses entre 2000101 et 20001231 dans la colonne col1.

### <a name="create-single-column-statistics-with-all-the-options"></a>Créer des statistiques sur une colonne en utilisant toutes les options

Vous pouvez aussi combiner les options. L’exemple suivant permet de créer un objet de statistiques filtrées avec une taille d’échantillon personnalisée :

```sql
CREATE STATISTICS stats_col1 ON table1 (col1) WHERE col1 > '2000101' AND col1 < '20001231' WITH SAMPLE = 50 PERCENT;
```

Pour accéder à la référence complète, consultez [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

### <a name="create-multi-column-statistics"></a>Créer des statistiques sur plusieurs colonnes

Pour créer un objet de statistiques sur plusieurs colonnes, utilisez les exemples précédents en spécifiant davantage de colonnes.

> [!NOTE]
> L’histogramme, qui est utilisé pour estimer le nombre de lignes dans le résultat d’une requête, est uniquement disponible pour la première colonne répertoriée dans la définition d’objet de statistiques.

Dans cet exemple, l’histogramme concerne l’élément *product\_category*. Les statistiques portant sur différentes colonnes sont calculées sur la base des éléments *product\_category* et *product\_sub_category* :

```sql
CREATE STATISTICS stats_2cols ON table1 (product_category, product_sub_category) WHERE product_category > '2000101' AND product_category < '20001231' WITH SAMPLE = 50 PERCENT;
```

Dans la mesure où il existe une corrélation entre les éléments *product\_category* et *product\_sub\_category*, un objet de statistiques sur plusieurs colonnes peut être utile quand le système accède à ces colonnes en même temps.

### <a name="create-statistics-on-all-columns-in-a-table"></a>Créer des statistiques sur toutes les colonnes d’une table

Pour créer des statistiques, vous pouvez par exemple émettre des commandes CREATE STATISTICS après avoir créé la table :

```sql
CREATE TABLE dbo.table1
(
   col1 int
,  col2 int
,  col3 int
)
WITH
  (
    CLUSTERED COLUMNSTORE INDEX
  )
;

CREATE STATISTICS stats_col1 on dbo.table1 (col1);
CREATE STATISTICS stats_col2 on dbo.table2 (col2);
CREATE STATISTICS stats_col3 on dbo.table3 (col3);
```

### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-sql-pool"></a>Utiliser une procédure stockée pour créer des statistiques sur toutes les colonnes d’un pool SQL

Le pool SQL dédié ne possède pas de procédure stockée système équivalente à sp_create_stats dans SQL Server. Cette procédure stockée crée un objet de statistiques sur une colonne portant sur chaque colonne du pool SQL non pourvu de statistiques.

L’exemple suivant vous aidera à commencer à concevoir votre pool SQL. N’hésitez pas à l’adapter à vos besoins.

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default 2 Fullscan 3 Sample
,   @sample_pct     tinyint
)
AS

IF @create_type IS NULL
BEGIN
    SET @create_type = 1;
END;

IF @create_type NOT IN (1,2,3)
BEGIN
    THROW 151000,'Invalid value for @stats_type parameter. Valid range 1 (default), 2 (fullscan) or 3 (sample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN;
    DROP TABLE #stats_ddl;
END;

CREATE TABLE #stats_ddl
WITH    (   DISTRIBUTION    = HASH([seq_nmbr])
        ,   LOCATION        = USER_DB
        )
AS
WITH T
AS
(
SELECT      t.[name]                        AS [table_name]
,           s.[name]                        AS [table_schema_name]
,           c.[name]                        AS [column_name]
,           c.[column_id]                   AS [column_id]
,           t.[object_id]                   AS [object_id]
,           ROW_NUMBER()
            OVER(ORDER BY (SELECT NULL))    AS [seq_nmbr]
FROM        sys.[tables] t
JOIN        sys.[schemas] s         ON  t.[schema_id]       = s.[schema_id]
JOIN        sys.[columns] c         ON  t.[object_id]       = c.[object_id]
LEFT JOIN   sys.[stats_columns] l   ON  l.[object_id]       = c.[object_id]
                                    AND l.[column_id]       = c.[column_id]
                                    AND l.[stats_column_id] = 1
LEFT JOIN    sys.[external_tables] e    ON    e.[object_id]        = t.[object_id]
WHERE       l.[object_id] IS NULL
AND            e.[object_id] IS NULL -- not an external table
)
SELECT  [table_schema_name]
,       [table_name]
,       [column_name]
,       [column_id]
,       [object_id]
,       [seq_nmbr]
,       CASE @create_type
        WHEN 1
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+')' AS VARCHAR(8000))
        WHEN 2
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH FULLSCAN' AS VARCHAR(8000))
        WHEN 3
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH SAMPLE '+CONVERT(varchar(4),@sample_pct)+' PERCENT' AS VARCHAR(8000))
        END AS create_stat_ddl
FROM T
;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''
;

WHILE @i <= @t
BEGIN
    SET @s=(SELECT create_stat_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

Pour créer des statistiques sur toutes les colonnes de la table avec les valeurs par défaut, exécutez la procédure stockée.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 1, NULL;
```

Pour créer des statistiques sur toutes les colonnes de la table avec une analyse complète (fullscan), appelez cette procédure.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

Pour créer des échantillons de statistiques sur toutes les colonnes de la table, entrez 3 et le pourcentage de l’échantillon. Cette procédure utilise un taux d’échantillonnage de 20 %.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

## <a name="examples-update-statistics"></a>Exemples : Mettre à jour les statistiques

Pour effectuer cette opération, vous avez différentes possibilités :

- Mettez à jour un objet de statistiques. Spécifiez le nom de l’objet de statistiques que vous souhaitez mettre à jour.
- Mettez à jour tous les objets de statistiques sur une table. Spécifiez le nom de la table, et non un objet de statistiques spécifique.

### <a name="update-one-specific-statistics-object"></a>Mettre à jour un objet de statistiques spécifique

Pour réaliser cette opération, utilisez la syntaxe suivante :

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Par exemple :

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

En mettant à jour des objets de statistiques spécifiques, vous pouvez réduire les ressources et le temps requis pour gérer les statistiques. La sélection des objets de statistiques les plus appropriés pour une mise à jour prend du temps.

### <a name="update-all-statistics-on-a-table"></a>Mettre à jour toutes les statistiques d'une table

Voici une méthode simple pour mettre à jour tous les objets de statistiques dans une table :

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Par exemple :

```sql
UPDATE STATISTICS dbo.table1;
```

L’instruction UPDATE STATISTICS est facile à utiliser. N’oubliez pas que cette action met à jour *toutes* les statistiques dans la table et, par conséquent, peut effectuer davantage de tâches que nécessaire. Si les performances ne sont pas un problème, il s’agit de la méthode la plus simple et la plus exhaustive pour garantir que les statistiques sont à jour.

> [!NOTE]
> Lors de la mise à jour de toutes les statistiques d’une table, le pool SQL dédié procède à une analyse afin d’échantillonner la table pour chaque objet de statistiques. Si la table est volumineuse et comprend un grand nombre de colonnes et de statistiques, il peut s’avérer plus efficace de mettre à jour les statistiques individuellement, en fonction des besoins.

Pour obtenir une implémentation d’une procédure `UPDATE STATISTICS`, consultez [Tables temporaires](sql-data-warehouse-tables-temporary.md). La méthode d’implémentation est légèrement différente de celle de la procédure `CREATE STATISTICS` précédente, mais le résultat est le même.

Pour accéder à la syntaxe complète, consultez [Mise à jour des statistiques](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="statistics-metadata"></a>Métadonnées de statistiques

Vous pouvez utiliser plusieurs fonctions et vues système pour rechercher des informations sur des statistiques. Par exemple, vous pouvez voir si un objet de statistiques peut être obsolète à l’aide de la fonction stats-date (qui permet de connaître la date de création ou de dernière mise à jour des statistiques).

### <a name="catalog-views-for-statistics"></a>Vues de catalogue des statistiques

Ces vues système fournissent des informations sur les statistiques :

| Affichage catalogue | Description |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque colonne. |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque objet de la base de données. |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque schéma de la base de données. |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque objet de statistiques. |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque colonne de l’objet de statistiques. Paramètre lié à l’élément « sys.columns ». |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque table (y compris les tables externes). |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Une ligne pour chaque type de données. |

### <a name="system-functions-for-statistics"></a>Fonctions système relatives aux statistiques

Ces fonctions système sont utiles lorsque vous gérez des statistiques :

| Fonction système | Description |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Date de la dernière mise à jour de l’objet de statistiques. |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) |Informations détaillées et récapitulatives sur la distribution des valeurs, telles que l’objet de statistiques la comprend. |

### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Combiner des fonctions et des colonnes de statistiques en une seule vue

Cette vue regroupe les colonnes portant sur les statistiques et les résultats de la fonction STATS_DATE().

```sql
CREATE VIEW dbo.vstats_columns
AS
SELECT
        sm.[name]                           AS [schema_name]
,       tb.[name]                           AS [table_name]
,       st.[name]                           AS [stats_name]
,       st.[filter_definition]              AS [stats_filter_definition]
,       st.[has_filter]                     AS [stats_is_filtered]
,       STATS_DATE(st.[object_id],st.[stats_id])
                                            AS [stats_last_updated_date]
,       co.[name]                           AS [stats_column_name]
,       ty.[name]                           AS [column_type]
,       co.[max_length]                     AS [column_max_length]
,       co.[precision]                      AS [column_precision]
,       co.[scale]                          AS [column_scale]
,       co.[is_nullable]                    AS [column_is_nullable]
,       co.[collation_name]                 AS [column_collation_name]
,       QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS two_part_name
,       QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS three_part_name
FROM    sys.objects                         AS ob
JOIN    sys.stats           AS st ON    ob.[object_id]      = st.[object_id]
JOIN    sys.stats_columns   AS sc ON    st.[stats_id]       = sc.[stats_id]
                            AND         st.[object_id]      = sc.[object_id]
JOIN    sys.columns         AS co ON    sc.[column_id]      = co.[column_id]
                            AND         sc.[object_id]      = co.[object_id]
JOIN    sys.types           AS ty ON    co.[user_type_id]   = ty.[user_type_id]
JOIN    sys.tables          AS tb ON  co.[object_id]        = tb.[object_id]
JOIN    sys.schemas         AS sm ON  tb.[schema_id]        = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

## <a name="dbcc-show_statistics-examples"></a>Exemples portant sur la fonction DBCC SHOW_STATISTICS()

La fonction DBCC SHOW_STATISTICS() présente les données contenues dans un objet de statistiques. Ces données sont affichées en trois parties :

- En-tête
- Vecteur de densité
- Histogramme

Métadonnées de l’en-tête sur les statistiques. L’histogramme affiche la distribution des valeurs dans la première colonne de l’objet de statistiques. Le vecteur de densité mesure la corrélation entre les colonnes.

> [!NOTE]
> Le pool SQL dédié calcule les estimations de cardinalité avec certaines données de l’objet de statistiques.

### <a name="show-header-density-and-histogram"></a>Afficher l’en-tête, la densité et l’histogramme

Cet exemple simple illustre les trois parties d’un objet de statistiques :

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Par exemple :

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>Afficher une ou plusieurs parties de la fonction DBCC SHOW_STATISTICS()

Si vous êtes uniquement intéressé par l’affichage de certaines parties, utilisez la clause `WITH` et spécifiez les parties que vous voulez voir :

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>) WITH stat_header, histogram, density_vector
```

Par exemple :

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1) WITH histogram, density_vector
```

## <a name="dbcc-show_statistics-differences"></a>Différences liées à la fonction DBCC SHOW_STATISTICS()

La fonction DBCC SHOW_STATISTICS() est implémentée de manière plus stricte dans le pool SQL dédié que dans SQL Server :

- Les fonctionnalités non documentées ne sont pas prises en charge.
- Impossible d’utiliser le paramètre Stats_stream.
- Impossible de joindre les résultats de sous-ensembles spécifiques de données de statistiques, Par exemple, STAT_HEADER JOIN DENSITY_VECTOR.
- L’élément NO_INFOMSGS ne peut pas être défini pour la suppression des messages.
- Vous ne pouvez pas placer de crochets autour des noms de statistiques.
- Vous ne pouvez pas utiliser les noms de colonnes pour identifier les objets de statistiques.
- L’erreur personnalisée 2767 n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

Pour améliorer davantage les performances des requêtes, consultez [Surveiller votre charge de travail](sql-data-warehouse-manage-monitor.md).
