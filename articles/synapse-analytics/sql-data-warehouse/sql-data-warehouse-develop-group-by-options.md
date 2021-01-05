---
title: Utiliser les options Regrouper par
description: Conseils pour l’implémentation des options de regroupement pour des pools SQL dédiés dans Azure Synapse Analytics.
services: synapse-analytics
author: MSTehrani
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: emtehran
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: cdac3b1e502a203c272db75eb4345446b0f7c0c8
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462760"
---
# <a name="group-by-options-for-dedicated-sql-pools-in-azure-synapse-analytics"></a>Options de regroupement pour des pools SQL dédiés dans Azure Synapse Analytics

Dans cet article, vous trouverez des conseils pour l’implémentation des options de regroupement dans des pools SQL dédiés.

## <a name="what-does-group-by-do"></a>Que fait GROUP BY ?

La clause [GROUP BY](/sql/t-sql/queries/select-group-by-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) T-SQL agrège des données en un ensemble de lignes récapitulatives. Le pool SQL dédié ne prend pas en charge certaines options de l’instruction GROUP BY. Ces options comportent des solutions de contournement, qui sont les suivantes :

* GROUP BY avec ROLLUP
* GROUPING SETS
* GROUP BY avec CUBE

## <a name="rollup-and-grouping-sets-options"></a>Options ROLLUP et GROUPING SETS

L’option la plus simple consiste à utiliser UNION ALL pour effectuer le cumul plutôt que de se fier à la syntaxe explicite. Le résultat est exactement le même.

L’exemple suivant à l’aide de l’instruction GROUP BY avec l’option ROLLUP :

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

## <a name="cube-options"></a>Options CUBE

Il est possible de créer une instruction GROUP BY WITH CUBE à l’aide de l’approche UNION ALL. Il existe cependant un problème : le code peut rapidement devenir fastidieux et difficile à gérer. Pour atténuer ce problème, vous pouvez utiliser cette approche plus avancée.

À l’aide de l’exemple précédent, la première étape consiste à définir le « cube » qui définit tous les niveaux d’agrégation que nous souhaitons créer.

Prenez note de la CROSS JOIN des deux tables dérivées, car cela génère tous les niveaux pour nous. Le reste du code est uniquement placé à des fins de formatage :

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

L’illustration suivante montre les résultats de CTAS :

![Regrouper par cube](./media/sql-data-warehouse-develop-group-by-options/sql-data-warehouse-develop-group-by-cube.png)

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

Quant à la troisième étape, elle consiste à effectuer une boucle sur notre cube de colonnes effectuant l’agrégation. La requête est exécutée une fois pour chaque ligne de la table temporaire #Cube. Les résultats sont stockés dans la table temporaire #Results :

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

Enfin, vous pouvez renvoyer les résultats en lisant les données de la table temporaire #Results :

```sql
SELECT *
FROM #Results
ORDER BY 1,2,3
;
```

Si nous fractionnons le code en sections et générons une construction en boucle, le code devient plus facile à gérer et à entretenir.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des conseils supplémentaires, consultez la [vue d’ensemble du développement](sql-data-warehouse-overview-develop.md).
