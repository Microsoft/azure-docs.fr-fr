---
title: Utiliser des options de l’instruction GROUP BY dans SQL Synapse
description: SQL Synapse permet de développer des solutions en implémentant différentes options de l’instruction GROUP BY.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: d160040620fcaa60a8adc955efcc54ac3ca2609e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321433"
---
# <a name="group-by-options-in-synapse-sql"></a>Options de l’instruction GROUP BY dans SQL Synapse

SQL Synapse permet de développer des solutions en implémentant différentes options de l’instruction GROUP BY. 

## <a name="what-group-by-does"></a>Action de GROUP BY

La clause [GROUP BY](/sql/t-sql/queries/select-group-by-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) T-SQL agrège des données en un ensemble de lignes récapitulatives.

Un pool SQL serverless prend en charge l’ensemble des options de l’instruction GROUP BY. Le pool SQL dédié prend en charge un nombre limité d’options de l’instruction GROUP BY.

## <a name="group-by-options-supported-in-dedicated-sql-pool"></a>Options de l’instruction GROUP BY prises en charge dans le pool SQL dédié

Le pool SQL dédié ne prend pas en charge certaines options de l’instruction GROUP BY. Ces options comportent des solutions de contournement, qui sont les suivantes :

* GROUP BY avec ROLLUP
* GROUPING SETS
* GROUP BY avec CUBE

### <a name="rollup-and-grouping-sets-options"></a>Options ROLLUP et GROUPING SETS

L’option la plus simple consiste à utiliser l’instruction UNION ALL pour effectuer le cumul plutôt que de s’appuyer sur la syntaxe explicite. Le résultat est exactement le même.

L’exemple suivant utilise l’instruction GROUP BY avec l’option ROLLUP :

```sql
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount)             AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t       ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY ROLLUP (
                        [SalesTerritoryCountry]
                ,       [SalesTerritoryRegion]
                )
;
```

À l’aide de ROLLUP, l’exemple précédent requiert les agrégations suivantes :

* Pays et région
* Country
* Total général

Pour remplacer ROLLUP et renvoyer les mêmes résultats, vous pouvez utiliser UNION ALL et spécifier explicitement les agrégations requises :

```sql
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY
       [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
UNION ALL
SELECT [SalesTerritoryCountry]
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY
       [SalesTerritoryCountry]
UNION ALL
SELECT NULL
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey;
```

Pour remplacer GROUPING SETS, le principe de l’exemple s’applique. Il vous suffit de créer des sections UNION ALL pour les niveaux d’agrégation que vous souhaitez afficher.

### <a name="cube-options"></a>Options CUBE

Il est possible de créer une instruction GROUP BY WITH CUBE à l’aide de l’approche UNION ALL. Il existe cependant un problème : le code peut rapidement devenir fastidieux et difficile à gérer. Pour atténuer ce problème, vous pouvez adopter cette approche plus avancée.

La première étape consiste à définir le « cube » qui définit tous les niveaux d’agrégation que nous souhaitons créer. Prenez note de la CROSS JOIN des deux tables dérivées, car elle génère tous les niveaux. Le reste du code est uniquement destiné au formatage.

```sql
CREATE TABLE #Cube
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
AS
WITH GrpCube AS
(SELECT    CAST(ISNULL(Country,'NULL')+','+ISNULL(Region,'NULL') AS NVARCHAR(50)) as 'Cols'
,          CAST(ISNULL(Country+',','')+ISNULL(Region,'') AS NVARCHAR(50))  as 'GroupBy'
,          ROW_NUMBER() OVER (ORDER BY Country) as 'Seq'
FROM       ( SELECT 'SalesTerritoryCountry' as Country
             UNION ALL
             SELECT NULL
           ) c
CROSS JOIN ( SELECT 'SalesTerritoryRegion' as Region
             UNION ALL
             SELECT NULL
           ) r
)
SELECT Cols
,      CASE WHEN SUBSTRING(GroupBy,LEN(GroupBy),1) = ','
            THEN SUBSTRING(GroupBy,1,LEN(GroupBy)-1)
            ELSE GroupBy
       END AS GroupBy  --Remove Trailing Comma
,Seq
FROM GrpCube;
```

L’image suivante illustre les résultats de l’instruction [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) :

![Regrouper par cube](./media/develop-group-by-options/develop-group-by-cube.png)

La deuxième étape consiste à spécifier une table cible pour stocker les résultats intermédiaires :

```sql
DECLARE
 @SQL NVARCHAR(4000)
,@Columns NVARCHAR(4000)
,@GroupBy NVARCHAR(4000)
,@i INT = 1
,@nbr INT = 0
;
CREATE TABLE #Results
(
 [SalesTerritoryCountry] NVARCHAR(50)
,[SalesTerritoryRegion]  NVARCHAR(50)
,[TotalSalesAmount]      MONEY
)
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
;
```

Quant à la troisième étape, elle consiste à exécuter une boucle sur le cube de colonnes effectuant l’agrégation. La requête est exécutée une fois pour chaque ligne de la table temporaire #Cube. Les résultats sont stockés dans la table temporaire #Results :

```sql
SET @nbr =(SELECT MAX(Seq) FROM #Cube);

WHILE @i<=@nbr
BEGIN
    SET @Columns = (SELECT Cols    FROM #Cube where seq = @i);
    SET @GroupBy = (SELECT GroupBy FROM #Cube where seq = @i);

    SET @SQL ='INSERT INTO #Results
              SELECT '+@Columns+'
              ,      SUM(SalesAmount) AS TotalSalesAmount
              FROM  dbo.factInternetSales s
              JOIN  dbo.DimSalesTerritory t  
              ON s.SalesTerritoryKey = t.SalesTerritoryKey
              '+CASE WHEN @GroupBy <>''
                     THEN 'GROUP BY '+@GroupBy ELSE '' END

    EXEC sp_executesql @SQL;
    SET @i +=1;
END
```

Enfin, vous pouvez renvoyer les résultats en lisant la table temporaire #Results :

```sql
SELECT *
FROM #Results
ORDER BY 1,2,3
;
```

Si nous fractionnons le code en sections et générons une construction en boucle, le code devient plus facile à gérer et à entretenir.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des conseils supplémentaires, consultez la [vue d’ensemble du développement](develop-overview.md).
