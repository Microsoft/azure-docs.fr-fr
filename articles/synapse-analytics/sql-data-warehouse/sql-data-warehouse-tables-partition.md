---
title: Partitionnement de tables
description: Recommandations et exemples relatifs à l’utilisation de partitions de tables dans un pool SQL dédié
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/18/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: f65c1d6fda09d7762a59fb5a932a72ad706a767a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448020"
---
# <a name="partitioning-tables-in-dedicated-sql-pool"></a>Partitionnement de tables dans le pool SQL dédié

Recommandations et exemples relatifs à l’utilisation de partitions de tables dans un pool SQL dédié.

## <a name="what-are-table-partitions"></a>Qu’est-ce que les partitions de table ?

Les partitions de table permettent de diviser vos données en groupes de données plus petits. Dans la plupart des cas, les partitions de table sont créées sur une colonne de dates. Le partitionnement est pris en charge sur tous les types de tables de pools SQL dédié, notamment un columnstore en cluster, un index cluster et un segment de mémoire. Le partitionnement est également pris en charge sur tous les types de distribution, notamment le hachage ou le tourniquet (round robin) distribué.  

Le partitionnement peut bénéficier de la maintenance des données et des performances des requêtes. Le fait qu’il bénéficie à ces deux éléments ou à un seul dépend de la façon dont les données sont chargées et du fait que la même colonne puisse être utilisée ou non dans les deux cas, étant donné que le partitionnement peut uniquement être effectué sur une colonne.

### <a name="benefits-to-loads"></a>Avantages pour les charges

Le principal avantage du partitionnement dans un pool SQL dédié est d’améliorer l’efficacité et les performances du chargement de données en utilisant la suppression, le basculement et la fusion de partitions. Dans la plupart des cas, les données sont partitionnées sur une colonne de dates étroitement liée à l’ordre selon lequel les données sont chargées dans le pool SQL. L’utilisation de partitions à des fins de maintenance des données présente l’avantage d’éviter la journalisation des transactions. Bien que l’insertion, la mise à jour ou la suppression de données constituent l’approche la plus simple, avec un peu de réflexion et d’efforts, l’utilisation du partitionnement pendant votre processus de chargement peut considérablement améliorer les performances.

Le basculement de partitions peut servir à supprimer ou à remplacer rapidement une section d’une table.  Par exemple, une table de faits des ventes peut contenir seulement des données pour les 36 derniers mois. À la fin de chaque mois, le mois de données de ventes le plus ancien est supprimé de la table.  Ces données ont pu être supprimées à l’aide d’une instruction delete pour supprimer les données pour le mois plus ancien. 

Toutefois, la suppression d’une grande quantité de données ligne par ligne avec une instruction DELETE peut prendre beaucoup de temps, ainsi que créer le risque de transactions volumineuses qui pourraient nécessiter un temps de restauration considérable en cas de problème. Une approche plus optimale consiste à supprimer la partition de données la plus ancienne. Alors que la suppression des lignes individuelle peut prendre des heures, la suppression d’une partition complète ne prend que secondes.

### <a name="benefits-to-queries"></a>Avantages pour les requêtes

Le partitionnement peut également servir à améliorer les performances des requêtes. Une requête qui applique un filtre de données partitionnées peut limiter l’analyse aux seules partitions éligibles. Cette méthode de filtrage peut éviter l’analyse de table complète et n’analyser qu’un sous-ensemble de données moins conséquent. Avec l’introduction des index columnstore en cluster, les avantages des performances de l’élimination de prédicats sont moins notables, mais dans certains cas, il peut exister un avantage pour les requêtes. 

Par exemple, si la table de faits Sales (Ventes) est partagée en 36 mois grâce au champ de date des ventes, les requêtes filtrées par date de vente peuvent ignorer la recherche dans les partitions qui ne correspondent pas au filtre.

## <a name="sizing-partitions"></a>Dimensionnement des partitions

Tandis que le partitionnement peut être utilisé pour améliorer les performances de certains scénarios, la création d’une table avec **trop** de partitions peut nuire aux performances dans certaines circonstances.  Ces inquiétudes sont particulièrement avérées pour les tables columnstore en cluster. 

Pour que le partitionnement soit utile, il est important de savoir quand utiliser le partitionnement et le nombre de partitions à créer. Il n’existe aucune règle absolue concernant le nombre de partitions ; cela dépend de vos données et du nombre de partitions que vous chargez simultanément. En règle générale, un schéma de partition réussi n’a qu’entre dix et cent partitions, pas mille.

Lorsque vous créez des partitions sur des tables **columnstore en cluster**, il est important de prendre en compte le nombre de lignes dans chaque partition. Pour une compression et des performances des tables columnstore en cluster optimales, un minimum de 1 million de lignes par partition et par distribution est nécessaire. Avant la création des partitions, le pool SQL dédié divise déjà chaque table en 60 bases de données distribuées. 

Tout partitionnement ajouté à une table est en plus des distributions créées en arrière-plan. Dans cet exemple, si la table de faits de ventes contient 36 partitions mensuelles, et étant donné qu’un pool SQL dédié comporte 60 distributions, la table de faits de ventes doit contenir 60 millions de lignes par mois, ou 2,1 milliards de lignes lorsque tous les mois sont remplis. Si une table contient moins de lignes que le nombre minimum recommandé par partition, envisagez d’utiliser moins de partitions pour augmenter le nombre de lignes par partition. 

Pour plus d’informations, consultez l’article [Indexation](sql-data-warehouse-tables-index.md) qui comporte des requêtes pouvant accéder à la qualité des index columnstore en cluster.

## <a name="syntax-differences-from-sql-server"></a>Différences de syntaxe par rapport à SQL Server

Le pool SQL dédié présente un mode de définition des partitions qui est plus simple que SQL Server. Les fonctions et les schémas de partitionnement ne sont pas utilisés dans le pool SQL dédié, car ils se trouvent dans SQL Server. À la place, il vous suffit d’identifier la colonne partitionnée et les points de limite. 

Bien que la syntaxe de partitionnement puisse être légèrement différente de SQL Server, les concepts de base sont les mêmes. SQL Server et le pool SQL dédié prennent en charge une colonne de partition par table, qui peut être une partition par spécification de plages de valeurs. Pour en savoir plus sur le partitionnement, consultez [Tables et index partitionnés](/sql/relational-databases/partitions/partitioned-tables-and-indexes?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

L’exemple suivant utilise l’instruction [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour partitionner la table FactInternetSales sur la colonne OrderDateKey :

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
    [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

## <a name="migrating-partitioning-from-sql-server"></a>Migration du partitionnement dans SQL Server

Pour migrer les définitions de partitions SQL Server vers le pool SQL dédié, il vous suffit d’effectuer les opérations suivantes :

- Éliminez le [schéma de partition](/sql/t-sql/statements/create-partition-scheme-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) SQL Server.
- Ajoutez la définition de la [fonction de partition](/sql/t-sql/statements/create-partition-function-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) à votre instruction CREATE TABLE.

Si vous migrez une table partitionnée à partir d’une instance SQL Server, le SQL ci-dessous peut vous aider à déterminer le nombre de lignes se trouvant dans chaque partition. N’oubliez pas que si la même granularité de partitionnement est utilisée sur le pool SQL dédié, le nombre de lignes par partition diminue d’un facteur de 60.  

```sql
-- Partition information for a SQL Server Database
SELECT      s.[name]                        AS      [schema_name]
,           t.[name]                        AS      [table_name]
,           i.[name]                        AS      [index_name]
,           p.[partition_number]            AS      [partition_number]
,           SUM(a.[used_pages]*8.0)         AS      [partition_size_kb]
,           SUM(a.[used_pages]*8.0)/1024    AS      [partition_size_mb]
,           SUM(a.[used_pages]*8.0)/1048576 AS      [partition_size_gb]
,           p.[rows]                        AS      [partition_row_count]
,           rv.[value]                      AS      [partition_boundary_value]
,           p.[data_compression_desc]       AS      [partition_compression_desc]
FROM        sys.schemas s
JOIN        sys.tables t                    ON      t.[schema_id]         = s.[schema_id]
JOIN        sys.partitions p                ON      p.[object_id]         = t.[object_id]
JOIN        sys.allocation_units a          ON      a.[container_id]      = p.[partition_id]
JOIN        sys.indexes i                   ON      i.[object_id]         = p.[object_id]
                                            AND     i.[index_id]          = p.[index_id]
JOIN        sys.data_spaces ds              ON      ds.[data_space_id]    = i.[data_space_id]
LEFT JOIN   sys.partition_schemes ps        ON      ps.[data_space_id]    = ds.[data_space_id]
LEFT JOIN   sys.partition_functions pf      ON      pf.[function_id]      = ps.[function_id]
LEFT JOIN   sys.partition_range_values rv   ON      rv.[function_id]      = pf.[function_id]
                                            AND     rv.[boundary_id]      = p.[partition_number]
WHERE       p.[index_id] <=1
GROUP BY    s.[name]
,           t.[name]
,           i.[name]
,           p.[partition_number]
,           p.[rows]
,           rv.[value]
,           p.[data_compression_desc]
;
```

## <a name="partition-switching"></a>Basculement de partitions

Le pool SQL dédié prend en charge le fractionnement, la fusion et le basculement de partition. Chacune de ces fonctions est exécutée à l’aide de l’instruction [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Pour faire basculer une partition d’une table à une autre, vous devez vous assurer que les partitions s’alignent sur leurs limites respectives et que les définitions de tables correspondent. Comme aucune contrainte de validation n’est disponible pour appliquer la plage de valeurs dans une table, la table source doit contenir les mêmes limites de partition que la table cible. Si les limites de partition sont différentes, le basculement de la partition échoue, car les métadonnées de celle-ci ne sont pas synchronisées.

### <a name="how-to-split-a-partition-that-contains-data"></a>Fractionnement d’une partition contenant des données

Pour fractionner une partition qui contient déjà des données, la méthode la plus efficace consiste à utiliser une instruction `CTAS` . Si la table partitionnée est une table columnstore en cluster, la partition de table doit être vide pour pouvoir être fractionnée.

L’exemple suivant crée une table columnstore partitionnée. Il insère une ligne dans chaque partition :

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101
                    )
                )
)
;

INSERT INTO dbo.FactInternetSales
VALUES (1,19990101,1,1,1,1,1,1);
INSERT INTO dbo.FactInternetSales
VALUES (1,20000101,1,1,1,1,1,1);
```

La requête suivante recherche le nombre de lignes à l’aide de la vue catalogue `sys.partitions` :

```sql
SELECT  QUOTENAME(s.[name])+'.'+QUOTENAME(t.[name]) as Table_name
,       i.[name] as Index_name
,       p.partition_number as Partition_nmbr
,       p.[rows] as Row_count
,       p.[data_compression_desc] as Data_Compression_desc
FROM    sys.partitions p
JOIN    sys.tables     t    ON    p.[object_id]   = t.[object_id]
JOIN    sys.schemas    s    ON    t.[schema_id]   = s.[schema_id]
JOIN    sys.indexes    i    ON    p.[object_id]   = i.[object_Id]
                            AND   p.[index_Id]    = i.[index_Id]
WHERE t.[name] = 'FactInternetSales'
;
```

La commande de fractionnement suivante génère un message d’erreur :

```sql
ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Le message 35346, au niveau 15, état 1, ligne 44, indique que la clause SPLIT de l’instruction ALTER PARTITION a échoué, car la partition n’est pas vide. Seules les partitions vides peuvent être fractionnées quand il existe un index columnstore sur la table. Vous pouvez envisager de désactiver l’index columnstore avant de lancer l’instruction ALTER PARTITION, puis de reconstruire l’index columnstore une fois l’opération ALTER PARTITION terminée.

Cependant, vous pouvez utiliser la commande `CTAS` pour créer une table afin de stocker les données.

```sql
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    FactInternetSales
WHERE   1=2
;
```

Comme les limites de la partition sont alignées, le basculement est autorisé. La table source présente donc une partition vide, que vous pouvez par la suite fractionner.

```sql
ALTER TABLE FactInternetSales SWITCH PARTITION 2 TO  FactInternetSales_20000101 PARTITION 2;

ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Il ne reste plus qu’à aligner les données en fonction des limites de la nouvelle partition, via la commande `CTAS`, et à faire basculer les données une nouvelle fois vers la table principale.

```sql
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales_20000101]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

ALTER TABLE dbo.FactInternetSales_20000101_20010101 SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2;
```

Une fois le déplacement des données effectué, il est judicieux d’actualiser les statistiques sur la table cible. La mise à jour des statistiques permet de vérifier qu’elles reflètent de manière précise la nouvelle distribution des données sur leurs partitions respectives.

```sql
UPDATE STATISTICS [dbo].[FactInternetSales];
```

### <a name="load-new-data-into-partitions-that-contain-data-in-one-step"></a>Charger de nouvelles données dans les partitions qui contiennent des données en une seule étape

Le chargement des données dans des partitions avec basculement de partition est un moyen pratique d’organiser de nouvelles données dans une table qui n’est pas visible aux utilisateurs.  Sur les systèmes occupés, il peut être difficile de traiter la contention de verrouillage associée au basculement de partition.  

Pour effacer les données existantes dans une partition, le paramètre `ALTER TABLE` était nécessaire pour extraire les données.  Un autre paramètre `ALTER TABLE` était ensuite requis pour insérer les nouvelles données.  

Dans le pool SQL dédié, l’option `TRUNCATE_TARGET` est prise en charge dans la commande `ALTER TABLE`.  Avec `TRUNCATE_TARGET`, la commande `ALTER TABLE` remplace les données existantes de la partition par de nouvelles données.  Voici un exemple qui illustre l’utilisation de `CTAS` pour créer une table avec les données existantes, l’insertion de nouvelles données, puis le basculement de toutes les données dans la table cible remplaçant ainsi les données existantes.

```sql
CREATE TABLE [dbo].[FactInternetSales_NewSales]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

INSERT INTO dbo.FactInternetSales_NewSales
VALUES (1,20000101,2,2,2,2,2,2);

ALTER TABLE dbo.FactInternetSales_NewSales SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2 WITH (TRUNCATE_TARGET = ON);  
```

### <a name="table-partitioning-source-control"></a>Contrôle de code source dans le cadre du partitionnement d’une table

Pour éviter la **détérioration** de la définition de votre table dans le système de contrôle du code source, vous pouvez envisager l’approche suivante :

1. Créez la table en tant que table partitionnée, mais sans ajouter de valeurs de partition.

    ```sql
    CREATE TABLE [dbo].[FactInternetSales]
    (
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
    )
    WITH
    (   CLUSTERED COLUMNSTORE INDEX
    ,   DISTRIBUTION = HASH([ProductKey])
    ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES () )
    )
    ;
    ```

1. `SPLIT` lors du processus de déploiement :

    ```sql
     -- Create a table containing the partition boundaries

    CREATE TABLE #partitions
    WITH
    (
        LOCATION = USER_DB
    ,   DISTRIBUTION = HASH(ptn_no)
    )
    AS
    SELECT  ptn_no
    ,       ROW_NUMBER() OVER (ORDER BY (ptn_no)) as seq_no
    FROM    (
        SELECT CAST(20000101 AS INT) ptn_no
        UNION ALL
        SELECT CAST(20010101 AS INT)
        UNION ALL
        SELECT CAST(20020101 AS INT)
        UNION ALL
        SELECT CAST(20030101 AS INT)
        UNION ALL
        SELECT CAST(20040101 AS INT)
    ) a
    ;

     -- Iterate over the partition boundaries and split the table

    DECLARE @c INT = (SELECT COUNT(*) FROM #partitions)
    ,       @i INT = 1                                 --iterator for while loop
    ,       @q NVARCHAR(4000)                          --query
    ,       @p NVARCHAR(20)     = N''                  --partition_number
    ,       @s NVARCHAR(128)    = N'dbo'               --schema
    ,       @t NVARCHAR(128)    = N'FactInternetSales' --table
    ;

    WHILE @i <= @c
    BEGIN
        SET @p = (SELECT ptn_no FROM #partitions WHERE seq_no = @i);
        SET @q = (SELECT N'ALTER TABLE '+@s+N'.'+@t+N' SPLIT RANGE ('+@p+N');');

        -- PRINT @q;
        EXECUTE sp_executesql @q;
        SET @i+=1;
    END

     -- Code clean-up

    DROP TABLE #partitions;
    ```

Avec cette approche, le code dans le contrôle de code source reste statique. Quant aux valeurs limites du partitionnement, elles peuvent rester dynamiques et évoluer avec le pool SQL au fil du temps.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le développement des tables, consultez les articles sur [Vue d’ensemble de la table](sql-data-warehouse-tables-overview.md).
