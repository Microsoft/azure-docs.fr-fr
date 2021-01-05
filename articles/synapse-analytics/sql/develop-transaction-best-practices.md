---
title: Optimiser les transactions pour un pool SQL dédié
description: Découvrez comment optimiser les performances de votre code transactionnel dans un pool SQL dédié.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: 908f047a22491d50337f51c0a6dce7f2db8a2ebc
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97026810"
---
# <a name="optimize-transactions-with-dedicated-sql-pool-in-azure-synapse-analytics"></a>Optimiser des transactions avec un pool SQL dédié dans Azure Synapse Analytics 

Découvrez comment optimiser les performances de votre code transactionnel dans le pool SQL dédié tout en minimisant les risques de restaurations de longue durée.

## <a name="transactions-and-logging"></a>Transactions et journalisation

Les transactions sont une composante importante d’un moteur de base de données relationnelle. Le pool SQL dédié utilise des transactions durant la modification des données. Ces transactions peuvent être explicites ou implicites. Les instructions uniques INSERT, UPDATE et DELETE sont toutes des exemples de transactions implicites. Les transactions explicites utilisent BEGIN TRAN, COMMIT TRAN ou ROLLBACK TRAN. Les transactions explicites sont généralement utilisées quand plusieurs instructions de modification doivent être liées dans une même unité atomique.

Le pool SQL dédié valide les modifications apportées à la base de données à l’aide de journaux des transactions. Chaque distribution présente son propre fichier journal. Les écritures des fichiers journaux de transactions sont automatiques. Aucune configuration n’est requise. Notez toutefois que ce processus, qui garantit l’écriture, introduit par ailleurs une surcharge sur le système. Pour réduire des effets, vous pouvez écrire du code efficace sur le plan transactionnel. Ce type de code se classe principalement en deux catégories.

* Utilisez autant que possible des constructions de journalisation minimale.
* Traitez les données en les regroupant par lots définis, afin d’éviter les transactions isolées de longue durée.
* Adoptez un modèle de basculement de partitions en cas de modifications importantes apportées à une partition donnée.

## <a name="minimal-vs-full-logging"></a>Journalisation minimale et journalisation complète

Contrairement aux opérations entièrement journalisées, qui utilisent le fichier journal de transactions pour effectuer le suivi de chaque modification de ligne, les journalisations minimales assurent le suivi des allocations d’étendue et des modifications de métadonnées uniquement. Par conséquent, la journalisation minimale implique la consignation exclusive des informations nécessaires à la restauration de la transaction en cas de défaillance ou pour une requête explicite (ROLLBACK TRAN). Dans la mesure où la journalisation minimale implique le suivi d’un volume de données moins important, cette opération est plus performante qu’une journalisation complète de taille similaire. En outre, un volume moins important d’écritures étant transmis vers le fichier journal de transactions, la quantité de données de journal générées est elle aussi réduite, et les performances E/S s’en trouvent accrues.

Les limites de sécurité des transactions s’appliquent uniquement aux opérations faisant l’objet d’une journalisation complète.

> [!NOTE]
> Les opérations faisant l’objet d’une journalisation minimale peuvent prendre part à des transactions explicites. Comme toutes les modifications des structures d’allocations font l’objet d’un suivi, il est possible de restaurer les journalisations minimales.

## <a name="minimally-logged-operations"></a>Journalisations minimales

Les opérations suivantes peuvent faire l’objet d’une journalisation minimale :

* CREATE TABLE AS SELECT ([CTAS](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json))
* INSERT..SELECT
* CREATE INDEX
* ALTER INDEX REBUILD
* DROP INDEX
* TRUNCATE TABLE
* DROP TABLE
* ALTER TABLE SWITCH PARTITION

<!--
- MERGE
- UPDATE on LOB Types .WRITE
- SELECT..INTO
-->

> [!NOTE]
> Les opérations de déplacement de données internes (telles que BROADCAST et SHUFFLE) ne sont pas affectées par la limite de sécurité des transactions.

## <a name="minimal-logging-with-bulk-load"></a>Journalisation minimale avec chargement en bloc

CTAS et INSERT...SELECT sont des opérations de chargement en bloc. Toutefois, ces deux éléments sont affectés par la définition de table cible et dépendent du scénario de chargement. Le tableau suivant décrit les situations de journalisations minimales ou complètes des opérations en bloc :  

| Index primaire | Scénario de chargement | Mode de journalisation |
| --- | --- | --- |
| Segment de mémoire (heap) |Quelconque |**Minimal** |
| Index cluster |Table cible vide |**Minimal** |
| Index cluster |Les lignes chargées ne se chevauchent pas avec les pages existantes dans la cible. |**Minimal** |
| Index cluster |Les lignes chargées se chevauchent avec les pages existantes dans la cible. |Complète |
| Index columstore en cluster |Taille de lot >= 102 400 par distribution alignée sur la partition |**Minimal** |
| Index columstore en cluster |Taille de lot < 102 400 par distribution alignée sur la partition |Complète |

Il est important de noter que toute opération d’écriture effectuée dans le cadre d’une mise à jour d’index secondaires ou non cluster correspond à une journalisation complète.

> [!IMPORTANT]
> Le pool SQL dédié contient 60 distributions. Par conséquent, en partant du principe que l’ensemble des lignes sont distribuées équitablement et placées dans une partition unique, votre lot devra comporter 6 144 000 lots ou plus pour faire l’objet d’une journalisation minimale lors d’une écriture vers un index columstore en cluster. Si la table est partitionnée et que les lignes insérées dépassent les limites de la partition, il vous faudra 6 144 000 lignes par limite de partition, pour une distribution uniforme des données. Dans chaque distribution, l’ensemble des partitions doivent isolément dépasser le seuil de 102 400 lignes pour que l’insertion fasse l’objet d’une journalisation minimale dans la distribution.

Le chargement de données dans une table non vide avec un index cluster comporte bien souvent une combinaison de lignes ayant fait l’objet d’une journalisation minimale et complète. Un index cluster est un arbre équilibré (arbre b) de pages. Si la page cible de l’écriture comporte déjà des lignes d’une autre transaction, ces opérations d’écriture feront l’objet d’une journalisation complète. Toutefois, si la page est vide, l’opération d’écriture fera l’objet d’une journalisation minimale.

## <a name="optimize-deletes"></a>Optimiser les suppressions

DELETE est une opération entièrement journalisée.  Si vous avez besoin de supprimer un volume important de données dans une table ou une partition, il est souvent plus judicieux d’appliquer une opération `SELECT` aux données que vous souhaitez conserver, qui peut être exécutée en tant qu’opération de journalisation minimale.  Pour sélectionner les données, créez une nouvelle table avec [CTAS](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).  Une fois la table créée, utilisez [RENAME](/sql/t-sql/statements/rename-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) pour permuter l’ancienne table et la table nouvellement créée.

```sql
-- Delete all sales transactions for Promotions except PromotionKey 2.

--Step 01. Create a new table select only the records we want to kep (PromotionKey 2)
CREATE TABLE [dbo].[FactInternetSales_d]
WITH
(    CLUSTERED COLUMNSTORE INDEX
,    DISTRIBUTION = HASH([ProductKey])
,     PARTITION     (    [OrderDateKey] RANGE RIGHT
                                    FOR VALUES    (    20000101, 20010101, 20020101, 20030101, 20040101, 20050101
                                                ,    20060101, 20070101, 20080101, 20090101, 20100101, 20110101
                                                ,    20120101, 20130101, 20140101, 20150101, 20160101, 20170101
                                                ,    20180101, 20190101, 20200101, 20210101, 20220101, 20230101
                                                ,    20240101, 20250101, 20260101, 20270101, 20280101, 20290101
                                                )
)
AS
SELECT     *
FROM     [dbo].[FactInternetSales]
WHERE    [PromotionKey] = 2
OPTION (LABEL = 'CTAS : Delete')
;

--Step 02. Rename the Tables to replace the
RENAME OBJECT [dbo].[FactInternetSales]   TO [FactInternetSales_old];
RENAME OBJECT [dbo].[FactInternetSales_d] TO [FactInternetSales];
```

## <a name="optimize-updates"></a>Optimiser les mises à jour

UPDATE est une opération entièrement journalisée.  Si vous devez mettre à jour un nombre important de lignes d’une table ou d’une partition, il peut souvent s’avérer plus efficace de recourir à une opération avec une journalisation minimale, comme [CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse).

Dans l’exemple suivant, une mise à jour complète de table a été convertie en CTAS, ce qui rend possible la mise en place d’une journalisation minimale.

Dans ce cas, nous ajoutons a posteriori une valeur de remise aux ventes dans la table :

```sql
--Step 01. Create a new table containing the "Update".
CREATE TABLE [dbo].[FactInternetSales_u]
WITH
(    CLUSTERED INDEX
,    DISTRIBUTION = HASH([ProductKey])
,     PARTITION     (    [OrderDateKey] RANGE RIGHT
                                    FOR VALUES    (    20000101, 20010101, 20020101, 20030101, 20040101, 20050101
                                                ,    20060101, 20070101, 20080101, 20090101, 20100101, 20110101
                                                ,    20120101, 20130101, 20140101, 20150101, 20160101, 20170101
                                                ,    20180101, 20190101, 20200101, 20210101, 20220101, 20230101
                                                ,    20240101, 20250101, 20260101, 20270101, 20280101, 20290101
                                                )
                )
)
AS
SELECT
    [ProductKey]  
,    [OrderDateKey]
,    [DueDateKey]  
,    [ShipDateKey]
,    [CustomerKey]
,    [PromotionKey]
,    [CurrencyKey]
,    [SalesTerritoryKey]
,    [SalesOrderNumber]
,    [SalesOrderLineNumber]
,    [RevisionNumber]
,    [OrderQuantity]
,    [UnitPrice]
,    [ExtendedAmount]
,    [UnitPriceDiscountPct]
,    ISNULL(CAST(5 as float),0) AS [DiscountAmount]
,    [ProductStandardCost]
,    [TotalProductCost]
,    ISNULL(CAST(CASE WHEN [SalesAmount] <=5 THEN 0
         ELSE [SalesAmount] - 5
         END AS MONEY),0) AS [SalesAmount]
,    [TaxAmt]
,    [Freight]
,    [CarrierTrackingNumber]
,    [CustomerPONumber]
FROM    [dbo].[FactInternetSales]
OPTION (LABEL = 'CTAS : Update')
;

--Step 02. Rename the tables
RENAME OBJECT [dbo].[FactInternetSales]   TO [FactInternetSales_old];
RENAME OBJECT [dbo].[FactInternetSales_u] TO [FactInternetSales];

--Step 03. Drop the old table
DROP TABLE [dbo].[FactInternetSales_old]
```

> [!NOTE]
> Les fonctionnalités de gestion de la charge de travail du pool SQL dédié peuvent faciliter la recréation des tables de grande taille. Pour plus d’informations, consultez l’article [Classes de ressources pour la gestion des charges de travail](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="optimize-with-partition-switching"></a>Optimisation avec basculement de partitions

Si vous devez procéder à des modifications à grande échelle au sein d’une [partition de table](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), il est plus judicieux d’adopter un modèle de basculement de partitions. Si la modification de données est considérable et relative à plusieurs partitions, vous obtiendrez un résultat identique en effectuant une itération sur les partitions.

Les étapes constitutives d’un basculement de partitions sont les suivantes :

1. Créer une partition vide
2. Effectuez la mise à jour en tant que CTAS
3. Extraire les données existantes vers la table cible
4. Basculer les nouvelles données
5. Nettoyer les données

Toutefois, pour faciliter l’identification des partitions à basculer, créez la procédure d’assistance suivante.  

```sql
CREATE PROCEDURE dbo.partition_data_get
    @schema_name           NVARCHAR(128)
,    @table_name               NVARCHAR(128)
,    @boundary_value           INT
AS
IF OBJECT_ID('tempdb..#ptn_data') IS NOT NULL
BEGIN
    DROP TABLE #ptn_data
END
CREATE TABLE #ptn_data
WITH    (    DISTRIBUTION = ROUND_ROBIN
        ,    HEAP
        )
AS
WITH CTE
AS
(
SELECT     s.name                            AS [schema_name]
,        t.name                            AS [table_name]
,         p.partition_number                AS [ptn_nmbr]
,        p.[rows]                        AS [ptn_rows]
,        CAST(r.[value] AS INT)            AS [boundary_value]
FROM        sys.schemas                    AS s
JOIN        sys.tables                    AS t    ON  s.[schema_id]        = t.[schema_id]
JOIN        sys.indexes                    AS i    ON     t.[object_id]        = i.[object_id]
JOIN        sys.partitions                AS p    ON     i.[object_id]        = p.[object_id]
                                                AND i.[index_id]        = p.[index_id]
JOIN        sys.partition_schemes        AS h    ON     i.[data_space_id]    = h.[data_space_id]
JOIN        sys.partition_functions        AS f    ON     h.[function_id]        = f.[function_id]
LEFT JOIN    sys.partition_range_values    AS r     ON     f.[function_id]        = r.[function_id]
                                                AND r.[boundary_id]        = p.[partition_number]
WHERE i.[index_id] <= 1
)
SELECT    *
FROM    CTE
WHERE    [schema_name]        = @schema_name
AND        [table_name]        = @table_name
AND        [boundary_value]    = @boundary_value
OPTION (LABEL = 'dbo.partition_data_get : CTAS : #ptn_data')
;
GO
```

Cette procédure optimise la réutilisation du code et permet de conserver un exemple de basculement de partitions plus compact.

Le code ci-dessous illustre les étapes mentionnées précédemment pour l’exécution d’une opération de basculement complet de partitions.

```sql
--Create a partitioned aligned empty table to switch out the data
IF OBJECT_ID('[dbo].[FactInternetSales_out]') IS NOT NULL
BEGIN
    DROP TABLE [dbo].[FactInternetSales_out]
END

CREATE TABLE [dbo].[FactInternetSales_out]
WITH
(    DISTRIBUTION = HASH([ProductKey])
,    CLUSTERED COLUMNSTORE INDEX
,     PARTITION     (    [OrderDateKey] RANGE RIGHT
                                    FOR VALUES    (    20020101, 20030101
                                                )
                )
)
AS
SELECT *
FROM    [dbo].[FactInternetSales]
WHERE 1=2
OPTION (LABEL = 'CTAS : Partition Switch IN : UPDATE')
;

--Create a partitioned aligned table and update the data in the select portion of the CTAS
IF OBJECT_ID('[dbo].[FactInternetSales_in]') IS NOT NULL
BEGIN
    DROP TABLE [dbo].[FactInternetSales_in]
END

CREATE TABLE [dbo].[FactInternetSales_in]
WITH
(    DISTRIBUTION = HASH([ProductKey])
,    CLUSTERED COLUMNSTORE INDEX
,     PARTITION     (    [OrderDateKey] RANGE RIGHT
                                    FOR VALUES    (    20020101, 20030101
                                                )
                )
)
AS
SELECT
    [ProductKey]  
,    [OrderDateKey]
,    [DueDateKey]  
,    [ShipDateKey]
,    [CustomerKey]
,    [PromotionKey]
,    [CurrencyKey]
,    [SalesTerritoryKey]
,    [SalesOrderNumber]
,    [SalesOrderLineNumber]
,    [RevisionNumber]
,    [OrderQuantity]
,    [UnitPrice]
,    [ExtendedAmount]
,    [UnitPriceDiscountPct]
,    ISNULL(CAST(5 as float),0) AS [DiscountAmount]
,    [ProductStandardCost]
,    [TotalProductCost]
,    ISNULL(CAST(CASE WHEN [SalesAmount] <=5 THEN 0
         ELSE [SalesAmount] - 5
         END AS MONEY),0) AS [SalesAmount]
,    [TaxAmt]
,    [Freight]
,    [CarrierTrackingNumber]
,    [CustomerPONumber]
FROM    [dbo].[FactInternetSales]
WHERE    OrderDateKey BETWEEN 20020101 AND 20021231
OPTION (LABEL = 'CTAS : Partition Switch IN : UPDATE')
;

--Use the helper procedure to identify the partitions
--The source table
EXEC dbo.partition_data_get 'dbo','FactInternetSales',20030101
DECLARE @ptn_nmbr_src INT = (SELECT ptn_nmbr FROM #ptn_data)
SELECT @ptn_nmbr_src

--The "in" table
EXEC dbo.partition_data_get 'dbo','FactInternetSales_in',20030101
DECLARE @ptn_nmbr_in INT = (SELECT ptn_nmbr FROM #ptn_data)
SELECT @ptn_nmbr_in

--The "out" table
EXEC dbo.partition_data_get 'dbo','FactInternetSales_out',20030101
DECLARE @ptn_nmbr_out INT = (SELECT ptn_nmbr FROM #ptn_data)
SELECT @ptn_nmbr_out

--Switch the partitions over
DECLARE @SQL NVARCHAR(4000) = '
ALTER TABLE [dbo].[FactInternetSales]    SWITCH PARTITION '+CAST(@ptn_nmbr_src AS VARCHAR(20))    +' TO [dbo].[FactInternetSales_out] PARTITION '    +CAST(@ptn_nmbr_out AS VARCHAR(20))+';
ALTER TABLE [dbo].[FactInternetSales_in] SWITCH PARTITION '+CAST(@ptn_nmbr_in AS VARCHAR(20))    +' TO [dbo].[FactInternetSales] PARTITION '        +CAST(@ptn_nmbr_src AS VARCHAR(20))+';'
EXEC sp_executesql @SQL

--Perform the clean-up
TRUNCATE TABLE dbo.FactInternetSales_out;
TRUNCATE TABLE dbo.FactInternetSales_in;

DROP TABLE dbo.FactInternetSales_out
DROP TABLE dbo.FactInternetSales_in
DROP TABLE #ptn_data
```

## <a name="minimize-logging-with-small-batches"></a>Minimiser la journalisation avec des lots de petite taille

Pour les opérations de grande envergure de modifications de données, il peut s’avérer judicieux de diviser l’activité en segments ou lots afin de répartir la charge de travail.

Le code ci-dessous est un exemple d’utilisation. La taille du lot a été définie sur un nombre insignifiant, ceci pour mettre en évidence la technique. Dans la réalité, la taille du lot serait bien plus importante.

```sql
SET NO_COUNT ON;
IF OBJECT_ID('tempdb..#t') IS NOT NULL
BEGIN
    DROP TABLE #t;
    PRINT '#t dropped';
END

CREATE TABLE #t
WITH    (    DISTRIBUTION = ROUND_ROBIN
        ,    HEAP
        )
AS
SELECT    ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS seq_nmbr
,        SalesOrderNumber
,        SalesOrderLineNumber
FROM    dbo.FactInternetSales
WHERE    [OrderDateKey] BETWEEN 20010101 and 20011231
;

DECLARE    @seq_start        INT = 1
,        @batch_iterator    INT = 1
,        @batch_size        INT = 50
,        @max_seq_nmbr    INT = (SELECT MAX(seq_nmbr) FROM dbo.#t)
;

DECLARE    @batch_count    INT = (SELECT CEILING((@max_seq_nmbr*1.0)/@batch_size))
,        @seq_end        INT = @batch_size
;

SELECT COUNT(*)
FROM    dbo.FactInternetSales f

PRINT 'MAX_seq_nmbr '+CAST(@max_seq_nmbr AS VARCHAR(20))
PRINT 'MAX_Batch_count '+CAST(@batch_count AS VARCHAR(20))

WHILE    @batch_iterator <= @batch_count
BEGIN
    DELETE
    FROM    dbo.FactInternetSales
    WHERE EXISTS
    (
            SELECT    1
            FROM    #t t
            WHERE    seq_nmbr BETWEEN  @seq_start AND @seq_end
            AND        FactInternetSales.SalesOrderNumber        = t.SalesOrderNumber
            AND        FactInternetSales.SalesOrderLineNumber    = t.SalesOrderLineNumber
    )
    ;

    SET @seq_start = @seq_end
    SET @seq_end = (@seq_start+@batch_size);
    SET @batch_iterator +=1;
END
```

## <a name="pause-and-scaling-guidance"></a>Conseils sur la suspension et la mise à l’échelle

Azure Synapse Analytics vous permet de [suspendre, reprendre et mettre à l’échelle](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) votre pool SQL dédié à la demande. 

Quand vous suspendez ou mettez à l’échelle votre pool SQL dédié, il est important de comprendre que l’ensemble des transactions en cours sont immédiatement arrêtées, ce qui entraîne la restauration de toute transaction ouverte. 

Si votre charge de travail a émis une modification de données incomplète et de longue durée avant l’opération de suspension ou de mise à l’échelle, cette tâche devra être annulée. Cette annulation peut avoir une incidence sur le temps nécessaire à la suspension ou à la mise à l’échelle de votre pool SQL dédié. 

> [!IMPORTANT]
> `UPDATE` et `DELETE` correspondant toutes deux à des journalisations complètes, ces opérations d’annulation et de rétablissement peuvent nécessiter un délai considérablement plus important que des journalisations minimales de taille équivalente.

La configuration idéale consiste à laisser les transactions de modification de données en cours se terminer avant la suspension ou la mise à l’échelle du pool SQL dédié. Toutefois, ce scénario n’est pas toujours pratique. Pour pallier le risque d’une longue restauration, envisagez l’une des options suivantes :

* Réécrire les opérations de longue durée à l’aide de [CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse)
* Décomposer l’opération en segments, en traitant un sous-ensemble des lignes

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les niveaux d’isolation et les limites des transactions, consultez [Transactions dans un pool SQL dédié](develop-transactions.md).  Pour une vue d’ensemble d’autres meilleures pratiques, consultez [Meilleures pratiques pour les pools SQL](best-practices-sql-pool.md).
