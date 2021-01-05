---
title: Créer et mettre à jour des statistiques à l’aide de ressources Azure Synapse SQL
description: Recommandations et exemples pour la création et la mise à jour de statistiques d’optimisation des requêtes dans SQL Synapse.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/19/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: 52e3ea3e07a81495f64f70f72686154a02a654af
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451795"
---
# <a name="statistics-in-synapse-sql"></a>Statistiques dans SQL Synapse

Cet article propose des recommandations et des exemples de création et de mise à jour des statistiques d’optimisation des requêtes à l’aide des ressources Synapse SQL : un pool SQL dédié et un pool SQL serverless.

## <a name="statistics-in-dedicated-sql-pool"></a>Statistiques dans un pool SQL dédié

### <a name="why-use-statistics"></a>Pourquoi utiliser des statistiques ?

Plus le pool SQL dédié connaît vos données, plus il peut exécuter des requêtes rapidement. Après le chargement des données dans un pool SQL dédié, la collecte de statistiques sur vos données est l’une des actions les plus importantes pour optimiser vos requêtes.  

L’optimiseur de requête du pool SQL dédié est un optimiseur basé sur les coûts. Il compare le coût de différents plans de requête, puis choisit le plan avec le coût le plus bas. Dans la plupart des cas, il choisit le plan qui s’exécute le plus rapidement.

Par exemple, si l’optimiseur estime que la date de filtrage de votre requête va renvoyer une ligne, il choisira un seul plan. S’il estime que la date sélectionnée va renvoyer un million de lignes, il choisira un autre plan.

### <a name="automatic-creation-of-statistics"></a>Création automatique de statistiques

Le moteur du pool SQL dédié analyse les requêtes utilisateur entrantes à la recherche de statistiques manquantes lorsque l’option AUTO_CREATE_STATISTICS de la base de données est définie sur `ON`.  Si des statistiques manquent, l’optimiseur de requête crée des statistiques sur des colonnes individuelles dans le prédicat de requête ou la condition de jointure. 

Cette fonction permet d’améliorer les estimations de cardinalité du plan de requête.

> [!IMPORTANT]
> La création automatique de statistiques est activée par défaut.

Vous pouvez vérifier si AUTO_CREATE_STATISTICS est configurée pour votre entrepôt de données en exécutant la commande suivante :

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

Si la propriété AUTO_CREATE_STATISTICS n’est pas activée pour votre entrepôt de données, nous vous recommandons de l’activer en exécutant la commande suivante :

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

Les instructions suivantes déclenchent la création automatique de statistiques :

- SELECT
- INSERT-SELECT
- CTAS
- UPDATE
- Suppression
- EXPLAIN quand elles contiennent une jointure ou que la présence d’un prédicat est détectée

> [!NOTE]
> La création automatique de statistiques ne porte pas sur les tables temporaires ou externes.

La création automatique de statistiques s'effectue de façon synchrone. Dès lors, les performances des requêtes risquent de subir une légère détérioration si des statistiques manquent dans vos colonnes. La durée de création de statistiques pour une seule colonne dépend de la taille de la table.

Pour éviter toute détérioration notable des performances, vous devez vérifier que les statistiques ont été créées en exécutant la charge de travail du test d’évaluation avant de profiler le système.

> [!NOTE]
> La création de statistiques est journalisée dans [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) sous un contexte utilisateur distinct.

Lorsque des statistiques automatiques sont créées, elles prennent la forme suivante : _WA_Sys_<ID de colonne de 8 chiffres en notation hexadécimale>_<ID de table de 8 chiffres en notation hexadécimale>. Vous pouvez visualiser les statistiques déjà créées en exécutant la commande [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) :

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

table_name spécifie le nom de la table contenant les statistiques à afficher, qui ne peut pas être une table externe. La cible est le nom de l’index, des statistiques ou de la colonne cibles pour lesquelles afficher des informations statistiques.

### <a name="update-statistics"></a>Mettre à jour les statistiques

Une meilleure pratique consiste à mettre à jour les statistiques sur les colonnes de date à chaque fois qu’une date est ajoutée. Chaque fois que de nouvelles lignes sont chargées dans l’entrepôt, de nouvelles dates de transaction et de chargement sont également ajoutées. Ces ajouts affectent la distribution des données et rendent les statistiques obsolètes.

Vous n’aurez peut-être jamais à mettre à jour les statistiques d’une colonne de pays ou de région dans une table des clients, car la distribution des valeurs change rarement. Si l’on part du principe que la distribution des données est constante d’un client à l’autre, l’ajout de nouvelles lignes dans une table ne devrait pas affecter cette distribution.

Toutefois, lorsque votre entrepôt de données indique un seul pays et si vous importez des données d’un nouveau pays ou d'une nouvelle région, il est nécessaire de mettre à jour les statistiques de la colonne de pays ou de région.

Voici certaines recommandations pour la mise à jour des statistiques :

|||
|-|-|
| **Fréquence des mises à jour des statistiques**  | Classique : Quotidien </br> Après le chargement ou la transformation de données |
| **Échantillonnage** |  Pour moins d’un milliard de lignes, utilisez le taux d’échantillonnage par défaut (20 %). </br> Pour plus d’un milliard de lignes, utilisez le taux d’échantillonnage de 2 %. |

### <a name="determine-last-statistics-update"></a>Déterminer la dernière mise à jour des statistiques

L’une des premières questions que vous devez vous poser quand vous dépannez une requête est la suivante : **« Les statistiques sont-elles à jour ? »**

Or, vous ne pouvez pas répondre à cette question en vous appuyant sur l’âge des données. Un objet de statistiques à jour peut être ancien si aucune modification notable n’affecte les données sous-jacentes. Lorsque le nombre de lignes ou la distribution des valeurs change de manière substantielle dans une colonne, il convient *alors* de mettre à jour les statistiques.

Aucune vue de gestion dynamique n'est disponible pour déterminer si les données de la table ont changé depuis la dernière mise à jour des statistiques. Le fait de connaître l’ancienneté de vos statistiques peut vous donner un petit aperçu. 

Vous pouvez utiliser la requête suivante pour déterminer la date de la dernière mise à jour des statistiques sur chaque table.

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

Par exemple, les statistiques des **colonnes de date** d’un entrepôt de données doivent souvent être mises à jour. Chaque fois que de nouvelles lignes sont chargées dans l’entrepôt, de nouvelles dates de transaction et de chargement sont également ajoutées. Ces ajouts affectent la distribution des données et rendent les statistiques obsolètes.

Les statistiques d’une colonne indiquant le sexe d’un client dans une table n’auront peut-être jamais besoin d’être mises à jour. Si l’on part du principe que la distribution des données est constante d’un client à l’autre, l’ajout de nouvelles lignes dans une table ne devrait pas affecter cette distribution.

Cela étant, si votre entrepôt de données ne fait mention que d’un seul sexe et qu’une nouvelle exigence nécessite le recours à plusieurs sexes, vous devez mettre à jour les statistiques de la colonne relative au sexe. 

Pour plus d’informations, consultez l’article [Statistiques](/sql/relational-databases/statistics/statistics).

### <a name="implement-statistics-management"></a>Implémenter la gestion des statistiques

Il est souvent judicieux d’étendre le processus de chargement des données afin de vérifier que les statistiques sont mises à jour à la fin du chargement. Le chargement des données se produit lorsque la taille ou la distribution des valeurs, voire les deux, sont souvent modifiées dans les tables. Dès lors, il est logique que le processus de chargement implémente certains processus de gestion.

Les principes généraux suivants sont fournis afin de vous aider à mettre à jour vos statistiques pendant le processus de chargement :

- Assurez-vous que chaque table chargée présente au moins un objet de statistiques mis à jour. Cela met à jour les informations sur la taille des tables (nombre de lignes et de pages) dans le cadre de la mise à jour des statistiques.
- Concentrez-vous sur les colonnes participant aux clauses JOIN, GROUP BY, ORDER BY et DISTINCT.
- Envisagez de mettre plus souvent à jour les colonnes de clé croissante, comme celles des dates de transactions, car ces valeurs ne seront pas incluses dans l’histogramme des statistiques.
- Envisagez de mettre moins souvent à jour les colonnes de distribution statiques.
- N’oubliez pas que chaque objet de statistiques est mis à jour dans l’ordre. L’implémentation de l’élément `UPDATE STATISTICS <TABLE_NAME>` ne suffit pas toujours, notamment quand les tables sont volumineuses et incluent un grand nombre d’objets de statistiques.

Pour plus d’informations, consultez [Évaluation de la cardinalité](/sql/relational-databases/performance/cardinality-estimation-sql-server).

### <a name="examples-create-statistics"></a>Exemples : Create statistics

Ces exemples indiquent comment utiliser différentes options pour créer des statistiques. Les options à utiliser pour chaque colonne dépendent des caractéristiques de vos données et de l’utilisation de la colonne dans les requêtes.

#### <a name="create-single-column-statistics-with-default-options"></a>Créer des statistiques sur une colonne en utilisant les options par défaut

Pour créer des statistiques sur une colonne, indiquez le nom de l’objet de statistiques ainsi que celui de la colonne.
Cette syntaxe a recours à toutes les options par défaut. Par défaut, le pool SQL dédié échantillonne **20 %** de la table quand il crée des statistiques.

```sql
CREATE STATISTICS [statistics_name]
    ON [schema_name].[table_name]([column_name]);
```

Par exemple :

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1);
```

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Créer des statistiques sur plusieurs colonnes en examinant chaque ligne

Le taux d’échantillonnage par défaut est de 20 %, ce qui est suffisant pour la plupart des situations. Toutefois, vous pouvez l’ajuster en fonction de vos besoins. Pour échantillonner la table entière, utilisez la syntaxe suivante :

```sql
CREATE STATISTICS [statistics_name]
    ON [schema_name].[table_name]([column_name])
    WITH FULLSCAN;
```

Par exemple :

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1)
    WITH FULLSCAN;
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Créer des statistiques sur une colonne en spécifiant la taille de l’échantillon

Une autre option consiste à spécifier la taille de l’échantillon sous forme de pourcentage :

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1)
    WITH SAMPLE = 50 PERCENT;
```

#### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Créer des statistiques sur une colonne sur certaines lignes uniquement

Vous pouvez également créer des statistiques sur une partie des lignes de votre table. On parle alors de statistiques filtrées.

Par exemple, vous pouvez utiliser les statistiques filtrées quand vous prévoyez d’interroger une partition spécifique dans une table partitionnée volumineuse. En créant des statistiques sur les valeurs des partitions uniquement, vous améliorez la précision des statistiques. Vous constaterez également une amélioration des performances des requêtes.

Dans cet exemple, des statistiques sont créées sur une plage de valeurs. Les valeurs peuvent facilement être définies de manière à correspondre à la plage de valeurs d’une partition.

```sql
CREATE STATISTICS stats_col1
    ON table1(col1)
    WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Pour que l’optimiseur de requête envisage d’utiliser les statistiques filtrées lorsqu’il choisit le plan de requête distribuée, il faut que cette requête soit suffisamment petite pour pouvoir s’insérer dans la définition de l’objet de statistiques. Selon l’exemple précédent, la clause WHERE de la requête doit indiquer des valeurs incluses entre 2000101 et 20001231 dans la colonne col1.

#### <a name="create-single-column-statistics-with-all-the-options"></a>Créer des statistiques sur une colonne en utilisant toutes les options

Vous pouvez aussi combiner les options. L’exemple suivant permet de créer un objet de statistiques filtrées avec une taille d’échantillon personnalisée :

```sql
CREATE STATISTICS stats_col1
    ON table1 (col1)
    WHERE col1 > '2000101' AND col1 < '20001231'
    WITH SAMPLE = 50 PERCENT;
```

Pour accéder à la référence complète, consultez [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

#### <a name="create-multi-column-statistics"></a>Créer des statistiques sur plusieurs colonnes

Pour créer un objet de statistiques sur plusieurs colonnes, utilisez les exemples précédents en spécifiant davantage de colonnes.

> [!NOTE]
> L’histogramme, qui est utilisé pour estimer le nombre de lignes dans le résultat d’une requête, est uniquement disponible pour la première colonne répertoriée dans la définition d’objet de statistiques.

Dans cet exemple, l’histogramme concerne l’élément *product\_category*. Les statistiques portant sur différentes colonnes sont calculées sur la base des éléments *product\_category* et *product\_sub_category* :

```sql
CREATE STATISTICS stats_2cols
    ON table1 (product_category, product_sub_category)
    WHERE product_category > '2000101' AND product_category < '20001231'
    WITH SAMPLE = 50 PERCENT;
```

En présence d'une corrélation entre les éléments *product\_category* et *product\_sub\_category*, un objet de statistiques sur plusieurs colonnes peut être utile quand le système accède à ces colonnes en même temps.

#### <a name="create-statistics-on-all-columns-in-a-table"></a>Créer des statistiques sur toutes les colonnes d’une table

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

#### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-database"></a>Utiliser une procédure stockée pour créer des statistiques sur toutes les colonnes d’une base de données

Le pool SQL n’inclut pas de procédure stockée par le système équivalente à sp_create_stats dans SQL Server. Cette procédure stockée crée un objet de statistiques sur une colonne portant sur chaque colonne de la base de données non pourvue de statistiques.

L’exemple suivant vous aidera à commencer à concevoir votre base de données. N’hésitez pas à l’adapter à vos besoins :

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default, 2 Fullscan, 3 Sample
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

Pour créer des statistiques sur toutes les colonnes de la table avec une analyse complète (fullscan), appelez cette procédure :

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

Pour créer des échantillons de statistiques sur toutes les colonnes de la table, entrez 3 et le pourcentage de l’échantillon. La procédure ci-dessous utilise un taux d’échantillonnage de 20 %.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

### <a name="examples-update-statistics"></a>Exemples : Mettre à jour les statistiques

Pour effectuer cette opération, vous avez différentes possibilités :

- Mettez à jour un objet de statistiques. Spécifiez le nom de l’objet de statistiques que vous souhaitez mettre à jour.
- Mettez à jour tous les objets de statistiques sur une table. Spécifiez le nom de la table, et non un objet de statistiques spécifique.

#### <a name="update-one-specific-statistics-object"></a>Mettre à jour un objet de statistiques spécifique

Pour réaliser cette opération, utilisez la syntaxe suivante :

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Par exemple :

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

En mettant à jour des objets de statistiques spécifiques, vous pouvez réduire les ressources et le temps requis pour gérer les statistiques. La sélection des objets de statistiques les plus appropriés pour une mise à jour implique une certaine réflexion.

#### <a name="update-all-statistics-on-a-table"></a>Mettre à jour toutes les statistiques d'une table

Voici une méthode simple pour mettre à jour tous les objets de statistiques dans une table :

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Par exemple :

```sql
UPDATE STATISTICS dbo.table1;
```

L’instruction UPDATE STATISTICS est facile à utiliser. N’oubliez pas que cette action met à jour *toutes* les statistiques dans la table et, ce qui implique davantage de tâches que nécessaire. 

Si les performances ne constituent pas un problème, il s’agit de la méthode la plus simple et la plus exhaustive pour garantir que les statistiques sont à jour.

> [!NOTE]
> Lors de la mise à jour de toutes les statistiques d’une table, le pool SQL dédié procède à une analyse pour échantillonner la table à la recherche de chaque objet de statistiques. Si la table est volumineuse et comprend un grand nombre de colonnes et de statistiques, il peut s’avérer plus efficace de mettre à jour les statistiques individuellement, en fonction des besoins.

Pour obtenir une implémentation d’une procédure `UPDATE STATISTICS`, consultez [Tables temporaires](develop-tables-temporary.md). La méthode d’implémentation est légèrement différente de celle de la procédure `CREATE STATISTICS` précédente, mais le résultat est le même.
Pour accéder à la syntaxe complète, consultez [Mettre à jour les statistiques](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="statistics-metadata"></a>Métadonnées de statistiques

Vous pouvez utiliser plusieurs fonctions et vues système pour rechercher des informations sur des statistiques. Par exemple, vous pouvez voir si un objet de statistiques peut être obsolète à l’aide de la fonction STATS_DATE(). STATS_DATE () vous permet de savoir quand les statistiques ont été créées ou mises à jour pour la dernière fois.

#### <a name="catalog-views-for-statistics"></a>Vues de catalogue des statistiques

Ces vues système fournissent des informations sur les statistiques :

| Affichage catalogue | Description |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque colonne. |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque objet de la base de données. |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque schéma de la base de données. |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque objet de statistiques. |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque colonne de l’objet de statistiques. Paramètre lié à l’élément « sys.columns ». |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque table (y compris les tables externes). |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Une ligne pour chaque type de données. |

#### <a name="system-functions-for-statistics"></a>Fonctions système relatives aux statistiques

Ces fonctions système sont utiles lorsque vous gérez des statistiques :

| Fonction système | Description |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Date de la dernière mise à jour de l’objet de statistiques. |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Informations détaillées et récapitulatives sur la distribution des valeurs, telles que l’objet de statistiques la comprend. |

#### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Combiner des fonctions et des colonnes de statistiques en une seule vue

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
JOIN    sys.tables          AS tb ON    co.[object_id]      = tb.[object_id]
JOIN    sys.schemas         AS sm ON    tb.[schema_id]      = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

### <a name="dbcc-show_statistics-examples"></a>Exemples portant sur la fonction DBCC SHOW_STATISTICS()

La fonction DBCC SHOW_STATISTICS() présente les données contenues dans un objet de statistiques. Ces données sont affichées en trois parties :

- En-tête
- Vecteur de densité
- Histogramme

L'en-tête correspond aux métadonnées sur les statistiques. L’histogramme affiche la distribution des valeurs dans la première colonne de l’objet de statistiques. 

Le vecteur de densité mesure la corrélation entre les colonnes. Le pool SQL dédié calcule les évaluations de cardinalité avec certaines données dans l’objet de statistiques.

#### <a name="show-header-density-and-histogram"></a>Afficher l’en-tête, la densité et l’histogramme

Cet exemple simple illustre les trois parties d’un objet de statistiques :

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Par exemple :

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

#### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>Afficher une ou plusieurs parties de la fonction DBCC SHOW_STATISTICS()

Si vous êtes uniquement intéressé par l’affichage de certaines parties, utilisez la clause `WITH` et spécifiez les parties que vous voulez voir :

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
    WITH stat_header, histogram, density_vector
```

Par exemple :

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1)
    WITH histogram, density_vector
```

### <a name="dbcc-show_statistics-differences"></a>Différences liées à la fonction DBCC SHOW_STATISTICS()

`DBCC SHOW_STATISTICS()` est implémentée de manière plus stricte dans le pool SQL dédié que dans SQL Server :

- Les fonctionnalités non documentées ne sont pas prises en charge.
- Impossible d’utiliser Stats_stream.
- Impossible de joindre les résultats de sous-ensembles spécifiques de données de statistiques. Par exemple, STAT_HEADER JOIN DENSITY_VECTOR.
- L’élément NO_INFOMSGS ne peut pas être défini pour la suppression des messages.
- Vous ne pouvez pas placer de crochets autour des noms de statistiques.
- Vous ne pouvez pas utiliser les noms de colonnes pour identifier les objets de statistiques.
- L’erreur personnalisée 2767 n’est pas prise en charge.


## <a name="statistics-in-serverless-sql-pool"></a>Statistiques dans un pool SQL serverless

Les statistiques sont créées par colonne donnée pour un jeu de données particulier (chemin de stockage).

> [!NOTE]
> Les statistiques ne peuvent pas être créées pour les colonnes LOB.

### <a name="why-use-statistics"></a>Pourquoi utiliser des statistiques ?

Mieux le pool SQL serverless connaît vos données, plus vite il peut exécuter des requêtes dessus. Collecter des statistiques sur vos données est l’une des actions les plus importantes pour optimiser vos requêtes. 

L’optimiseur de requête du pool SQL serverless est un optimiseur basé sur les coûts. Il compare le coût de différents plans de requête, puis choisit le plan avec le coût le plus bas. Dans la plupart des cas, il choisit le plan qui s’exécute le plus rapidement. 

Par exemple, si l’optimiseur estime que la date de filtrage de votre requête va renvoyer une ligne, il choisira un certain plan. S’il estime que la date sélectionnée va renvoyer un million de lignes, il choisira un autre plan.

### <a name="automatic-creation-of-statistics"></a>Création automatique de statistiques

Le pool SQL serverless analyse les requêtes utilisateur entrantes pour repérer les statistiques manquantes. Si des statistiques manquent, l’optimiseur de requête crée des statistiques sur des colonnes individuelles dans le prédicat de requête ou la condition de jointure afin d’améliorer les estimations de cardinalité pour le plan de requête.

L'instruction SELECT déclenche la création automatique de statistiques.

> [!NOTE]
> La création automatique de statistiques est activée pour les fichiers Parquet. Pour les fichiers CSV, vous devez créer des statistiques manuellement jusqu’à ce que la création automatique des statistiques de fichiers CSV soit prise en charge.

La création automatique de statistiques étant effectuée de façon synchrone, les performances des requêtes risquent de subir une légère détérioration si des statistiques manquent dans vos colonnes. La durée de création de statistiques pour une seule colonne dépend de la taille des fichiers ciblés.

### <a name="manual-creation-of-statistics"></a>Création manuelle de statistiques

Le pool SQL serverless vous permet de créer des statistiques manuellement. Pour les fichiers CSV, vous devez créer des statistiques manuellement, la création automatique de statistiques n'étant pas activée pour ces fichiers. 

Consultez les exemples suivants pour obtenir des instructions sur la création manuelle de statistiques.

### <a name="update-statistics"></a>Mettre à jour les statistiques

Les modifications apportées aux données des fichiers, de même que la suppression et l’ajout de fichiers entraînent des modifications de distribution des données et rendent les statistiques obsolètes. Dès lors, les statistiques doivent être mises à jour.

Le pool SQL serverless recrée automatiquement les statistiques si les données changent considérablement. Chaque fois que des statistiques sont créées automatiquement, l’état du jeu de données est enregistré : chemins d’accès aux fichiers, tailles, dates de la dernière modification.

Lorsque les statistiques sont obsolètes, de nouvelles statistiques sont créées. L’algorithme parcourt les données et les compare à l’état actuel du jeu de données. Si la taille des modifications est supérieure au seuil spécifique, les statistiques précédentes sont supprimées et recrées sur le nouveau jeu de données.

Les statistiques manuelles ne sont jamais déclarées obsolètes.

> [!NOTE]
> La recréation automatique des statistiques est activée pour les fichiers Parquet. Pour les fichiers CSV, vous devez supprimer et créer manuellement les statistiques jusqu’à ce que la création automatique des statistiques de fichiers CSV soit prise en charge. Consultez les exemples ci-dessous pour savoir comment supprimer et créer des statistiques.

L’une des premières questions que vous devez vous poser quand vous dépannez une requête est la suivante : **« Les statistiques sont-elles à jour ? »**

Lorsque le nombre de lignes ou la distribution des valeurs change de manière substantielle dans une colonne, il convient *alors* de mettre à jour les statistiques.

> [!NOTE]
> Si la distribution des valeurs d’une colonne a subi une modification significative, vous devez mettre à jour les statistiques, quelle que soit la date de la dernière mise à jour.

### <a name="implement-statistics-management"></a>Implémenter la gestion des statistiques

Vous pouvez étendre votre pipeline de données pour permettre la mise à jour des statistiques lorsque les données sont considérablement modifiées en cas d'ajout, de suppression ou de modification de fichiers.

Les principes généraux suivants sont fournis afin de vous aider à mettre à jour vos statistiques :

- Assurez-vous que jeu de données présente au moins un objet de statistiques mis à jour. Cela met à jour les informations sur la taille (nombre de lignes et de pages) dans le cadre du processus de mise à jour des statistiques.
- Concentrez-vous sur les colonnes participant aux clauses WHERE, JOIN, GROUP BY, ORDER BY et DISTINCT.
- Mettez à jour les colonnes de « clé croissante », comme celles des dates de transactions, car ces valeurs ne seront pas incluses dans l’histogramme des statistiques.
- Mettez moins souvent à jour les colonnes de distribution statiques.

Pour plus d’informations, consultez [Évaluation de la cardinalité](/sql/relational-databases/performance/cardinality-estimation-sql-server).

### <a name="examples-create-statistics-for-column-in-openrowset-path"></a>Exemples : Créer des statistiques pour les colonnes dans le chemin OPENROWSET

Ces exemples montrent comment utiliser différentes options pour créer des statistiques. Les options à utiliser pour chaque colonne dépendent des caractéristiques de vos données et de l’utilisation de la colonne dans les requêtes.

> [!NOTE]
> À ce stade, vous pouvez créer des statistiques à une seule colonne uniquement.
>
> Les autorisations suivantes sont requises pour exécuter sp_create_openrowset_statistics et sp_drop_openrowset_statistics : ADMINISTER BULK OPERATIONS ou ADMINISTER DATABASE BULK OPERATIONS.

La procédure stockée suivante est utilisée pour créer des statistiques :

```sql
sys.sp_create_openrowset_statistics [ @stmt = ] N'statement_text'
```

Arguments : [ @stmt =] N’statement_text' - Spécifie une instruction Transact-SQL qui renvoie les valeurs de colonne à utiliser pour les statistiques. Vous pouvez utiliser TABLESAMPLE pour spécifier des exemples de données à utiliser. Si TABLESAMPLE n’est pas spécifié, FULLSCAN est utilisé.

```syntaxsql
<tablesample_clause> ::= TABLESAMPLE ( sample_number PERCENT )
```

> [!NOTE]
> L’échantillonnage CSV ne fonctionne pas pour le moment. Seul FULLSCAN est pris en charge pour CSV.

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Créer des statistiques sur plusieurs colonnes en examinant chaque ligne

Pour créer des statistiques sur une colonne, fournissez une requête qui renvoie la colonne pour laquelle vous avez besoin de statistiques.

Par défaut, en l'absence de spécification, le pool SQL serverless utilise 100 % des données fournies dans le jeu de données lors de la création des statistiques.

Par exemple, pour créer des statistiques avec les options par défaut (FULLSCAN) pour une colonne year du jeu de données basé sur le fichier population.csv :

```sql
/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_openrowset_statistics N'SELECT year
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/csv/population/population.csv'',
        FORMAT = ''CSV'',
        FIELDTERMINATOR ='','',
        ROWTERMINATOR = ''\n''
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
'
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Créer des statistiques sur une colonne en spécifiant la taille de l’échantillon

Vous pouvez spécifier cette taille sous forme de pourcentage :

```sql
/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_openrowset_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
```

### <a name="examples-update-statistics"></a>Exemples : Mettre à jour les statistiques

Pour mettre à jour les statistiques, vous devez supprimer et créer des statistiques. La procédure stockée suivante est utilisée pour supprimer des statistiques :

```sql
sys.sp_drop_openrowset_statistics [ @stmt = ] N'statement_text'
```

> [!NOTE]
> Les autorisations suivantes sont requises pour exécuter sp_create_openrowset_statistics et sp_drop_openrowset_statistics : ADMINISTER BULK OPERATIONS ou ADMINISTER DATABASE BULK OPERATIONS.

Arguments : [ @stmt =] N’statement_text' - Spécifie la même instruction Transact-SQL que celle utilisée lors de la création des statistiques.

Pour mettre à jour les statistiques de la colonne year du jeu de données basé sur le fichier population.csv, vous devez supprimer et créer des statistiques :

```sql
EXEC sys.sp_drop_openrowset_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
GO

/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_openrowset_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
```

### <a name="examples-create-statistics-for-external-table-column"></a>Exemples : Créer des statistiques pour la colonne de table externe

Ces exemples montrent comment utiliser différentes options pour créer des statistiques. Les options à utiliser pour chaque colonne dépendent des caractéristiques de vos données et de l’utilisation de la colonne dans les requêtes.

> [!NOTE]
> À ce stade, vous pouvez créer des statistiques à une seule colonne uniquement.

Pour créer des statistiques sur une colonne, indiquez le nom de l’objet de statistiques ainsi que celui de la colonne.

```sql
CREATE STATISTICS statistics_name
ON { external_table } ( column )
    WITH
        { FULLSCAN
          | [ SAMPLE number PERCENT ] }
        , { NORECOMPUTE }
```

Arguments : external_table - Spécifie la table externe dans laquelle les statistiques doivent être créées.

FULLSCAN calcule les statistiques en analysant toutes les lignes. FULLSCAN et SAMPLE 100 PERCENT ont les mêmes résultats. FULLSCAN ne peut pas être utilisé avec l'option SAMPLE.

SAMPLE number PERCENT - Spécifie le pourcentage ou nombre de lignes approximatif dans la table ou vue indexée devant être utilisé par l'optimiseur de requête lors de la création des statistiques. Ce nombre peut être compris entre 0 et 100.

SAMPLE ne peut pas être utilisé avec l'option FULLSCAN.

> [!NOTE]
> L’échantillonnage CSV ne fonctionne pas pour le moment. Seul FULLSCAN est pris en charge pour CSV.

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Créer des statistiques sur plusieurs colonnes en examinant chaque ligne

```sql
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH FULLSCAN, NORECOMPUTE
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Créer des statistiques sur une colonne en spécifiant la taille de l’échantillon

```sql
-- following sample creates statistics with sampling 20%
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH SAMPLE 5 percent, NORECOMPUTE
```

### <a name="examples-update-statistics"></a>Exemples : Mettre à jour les statistiques

Pour mettre à jour les statistiques, vous devez supprimer et créer des statistiques. Commencez par supprimer des statistiques :

```sql
DROP STATISTICS census_external_table.sState
```

Puis créez des statistiques :

```sql
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH FULLSCAN, NORECOMPUTE
```

### <a name="statistics-metadata"></a>Métadonnées de statistiques

Vous pouvez utiliser plusieurs fonctions et vues système pour rechercher des informations sur des statistiques. Par exemple, vous pouvez voir si un objet de statistiques peut être obsolète à l’aide de la fonction STATS_DATE(). STATS_DATE () vous permet de savoir quand les statistiques ont été créées ou mises à jour pour la dernière fois.

> [!NOTE]
> Les métadonnées de statistiques sont disponibles uniquement pour les colonnes d’une table externe. Les métadonnées de statistiques ne sont pas disponibles pour les colonnes OPENROWSET.

#### <a name="catalog-views-for-statistics"></a>Vues de catalogue des statistiques

Ces vues système fournissent des informations sur les statistiques :

| Affichage catalogue                                                 | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque colonne.                                     |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque objet de la base de données.                     |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque schéma de la base de données.                     |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque objet de statistiques.                          |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque colonne de l’objet de statistiques. Paramètre lié à l’élément « sys.columns ». |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque table (y compris les tables externes).           |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Une ligne pour chaque type de données.                                  |

#### <a name="system-functions-for-statistics"></a>Fonctions système relatives aux statistiques

Ces fonctions système sont utiles lorsque vous gérez des statistiques :

| Fonction système                                              | Description                                  |
| :----------------------------------------------------------- | :------------------------------------------- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Date de la dernière mise à jour de l’objet de statistiques. |

#### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Combiner des fonctions et des colonnes de statistiques en une seule vue

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
JOIN    sys.tables          AS tb ON    co.[object_id]      = tb.[object_id]
JOIN    sys.schemas         AS sm ON    tb.[schema_id]      = sm.[schema_id]
WHERE   st.[user_created] = 1
;
```

## <a name="next-steps"></a>Étapes suivantes

Afin d’améliorer davantage les performances des requêtes pour le pool SQL dédié, consultez [Surveiller votre charge de travail](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) et [Bonnes pratiques pour les pools SQL dédiés](best-practices-sql-pool.md#maintain-statistics).

Afin d’améliorer davantage les performances des requêtes pour un pool SQL serverless, consultez [Bonnes pratiques pour les pools SQL serverless](best-practices-sql-on-demand.md)
