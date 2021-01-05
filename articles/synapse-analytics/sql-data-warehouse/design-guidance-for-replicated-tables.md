---
title: Conseils de conception pour les tables répliquées
description: Suggestions sur la conception de tables répliquées dans un pool Synapse SQL
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/19/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 0cf40990d59aff984226244f520e6f8f937713fd
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96456488"
---
# <a name="design-guidance-for-using-replicated-tables-in-synapse-sql-pool"></a>Conseils de conception pour l'utilisation de tables répliquées dans un pool Synapse SQL

Cet article fournit des suggestions sur la conception de tables répliquées dans votre schéma de pool Synapse SQL. Utilisez ces recommandations pour améliorer les performances des requêtes en réduisant le déplacement de données et la complexité des requêtes.

> [!VIDEO https://www.youtube.com/embed/1VS_F37GI9U]

## <a name="prerequisites"></a>Prérequis

Cet article suppose que les concepts de distribution et de déplacement de données dans un pool SQL vous sont familiers.    Pour plus d’informations, consultez l’article sur [l’architecture](massively-parallel-processing-mpp-architecture.md).

Dans le cadre de la conception d’une table, essayez d’en savoir autant que possible sur vos données et la façon dont elles sont interrogées.    Considérez par exemple les questions suivantes :

- Quelle est la taille de la table ?
- Quelle est la fréquence d’actualisation de la table ?
- Est-ce que je dispose de tables de faits et de dimension dans un pool SQL ?

## <a name="what-is-a-replicated-table"></a>Qu’est-ce qu’une table répliquée ?

Une table répliquée possède une copie complète de la table accessible sur chaque nœud de calcul. La réplication d’une table évite le transfert de données entre des nœuds de calcul avant une jointure ou une agrégation. Étant donné que la table possède plusieurs copies, le fonctionnement des tables répliquées est optimal lorsque la taille de la table est inférieure à 2 Go compressés.  2 Go n’est pas une limite inconditionnelle.  Si les données sont statiques et ne changent pas, vous pouvez répliquer des tables plus volumineuses.

Le diagramme suivant illustre une table répliquée accessible sur chaque nœud de calcul. Dans un pool SQL, la table répliquée est entièrement copiée dans une base de données de distribution sur chaque nœud de calcul.

![Table répliquée](./media/design-guidance-for-replicated-tables/replicated-table.png "Table répliquée")  

Les tables répliquées fonctionnent bien dans des tables de dimension dans un schéma en étoile. Les tables de dimension sont généralement jointes à des tables de faits, qui sont réparties différemment des tables de dimension.  Les dimensions sont généralement une taille qui permet de stocker et de gérer plusieurs copies. Les dimensions stockent des données descriptives qui se modifient lentement, comme le nom et l’adresse du client, ainsi que les détails sur le produit. La lenteur de la nature changeante des données entraîne moins de maintenance pour la table répliquée.

Envisagez d’utiliser une table répliquée dans les cas suivants :

- La taille de la table sur le disque est inférieure à 2 Go, quel que soit le nombre de lignes. Pour connaître la taille d’une table, vous pouvez utiliser la commande [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) : `DBCC PDW_SHOWSPACEUSED('ReplTableCandidate')`.
- La table est utilisée dans des jointures qui requièrent normalement un déplacement des données. Quand vous joignez des tables qui ne sont pas distribuées sur la même colonne, telles qu’une table distribuée par hachage jointe à une table à distribution par tourniquet (round robin), un déplacement des données est nécessaire pour l’exécution de la requête.  Si l’une des tables est petite, pensez à utiliser une table répliquée. Nous vous recommandons d’utiliser des tables répliquées au lieu de tables de distribution par tourniquet dans la plupart des cas. Pour consulter les opérations de déplacement des données dans les plans de requêtes, utilisez [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  L’opération BroadcastMoveOperation est l’opération de déplacement de données qui peut généralement être supprimée à l’aide d’une table répliquée.  

Les tables répliquées ne produisent sans doute pas les meilleurs résultats dans les cas suivants :

- La table est l’objet d’opérations d’insertion, de mise à jour et de suppression fréquentes.  Les opérations DLM (langage de manipulation de données) nécessitent une regénération de la table répliquée.  La reconstruction fréquente peut diminuer les performances.
- Le pool SQL est fréquemment mis à l’échelle. La mise à l’échelle d’un pool SQL change le nombre de nœuds de calcul, ce qui entraîne une reconstruction de la table répliquée.
- La table comporte un grand nombre de colonnes, mais les opérations de données n’accèdent généralement qu’à un nombre restreint de colonnes. Dans ce scénario, au lieu de répliquer la table entière, il peut s’avérer plus efficace de distribuer la table et de créer ensuite un index sur les colonnes fréquemment sollicitées. Lorsqu'une requête exige un déplacement des données, le pool SQL déplace uniquement les données des colonnes demandées.

## <a name="use-replicated-tables-with-simple-query-predicates"></a>Utiliser des tables répliquées avec des prédicats de requête simples

Avant de décider de distribuer ou de répliquer une table, pensez aux types de requêtes que vous projetez d’exécuter sur la table. Lorsque possible,

- Utilisez des tables répliquées pour les requêtes avec des prédicats de requête simples, comme l’égalité ou l’inégalité.
- Utilisez des tables distribuées pour les requêtes avec des prédicats de requête complexes, comme LIKE ou NOT LIKE.

Les requêtes sollicitant beaucoup le processeur fonctionnent de manière optimale lorsque le travail est réparti entre tous les nœuds de calcul. Par exemple, les requêtes qui exécutent des calculs sur chaque ligne d’une table fonctionnent mieux sur les tables distribués que sur les tables répliquées. Étant donné qu’une table répliquée est entièrement stockée sur chaque nœud de calcul, une requête sollicitant beaucoup le processeur sur une table répliquée s’exécute sur la table entière sur chaque nœud de calcul. Le calcul supplémentaire peut ralentir les performances des requêtes.

Par exemple, cette requête comporte un prédicat complexe.  Elle s’exécute plus rapidement lorsque les données se trouvent dans une table distribuée au lieu d’une table répliquée. Dans cet exemple, les données peuvent être distribuées par tourniquet.

```sql

SELECT EnglishProductName
FROM DimProduct
WHERE EnglishDescription LIKE '%frame%comfortable%'

```

## <a name="convert-existing-round-robin-tables-to-replicated-tables"></a>Convertir les tables de distribution par tourniquet (round-robin) en tables répliquées

Si vous avez déjà des tables de distribution par tourniquet, nous vous recommandons de les convertir en tables répliquées si elles sont conformes aux critères décrits dans cet article. Les tables répliquées ont de meilleures performances que les tables de distribution par tourniquet, car elles éliminent le déplacement des données.  Une table de distribution par tourniquet requiert toujours le déplacement des données pour les jointures.

Cet exemple utilise [CTAS](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour modifier la table DimSalesTerritory en une table répliquée. Cet exemple fonctionne que la table DimSalesTerritory soit distribuée par hachage ou par tourniquet.

```sql
CREATE TABLE [dbo].[DimSalesTerritory_REPLICATE]
WITH
  (
    CLUSTERED COLUMNSTORE INDEX,  
    DISTRIBUTION = REPLICATE  
  )  
AS SELECT * FROM [dbo].[DimSalesTerritory]
OPTION  (LABEL  = 'CTAS : DimSalesTerritory_REPLICATE')

-- Switch table names
RENAME OBJECT [dbo].[DimSalesTerritory] to [DimSalesTerritory_old];
RENAME OBJECT [dbo].[DimSalesTerritory_REPLICATE] TO [DimSalesTerritory];

DROP TABLE [dbo].[DimSalesTerritory_old];
```

### <a name="query-performance-example-for-round-robin-versus-replicated"></a>Exemple de performances de requête pour une table de distribution par tourniquet et une table répliquée

Une table répliquée ne nécessite pas le déplacement des données pour les jointures, car la table entière est déjà présente sur chaque nœud de calcul. Si les tables de dimension sont des tables distribuées par tourniquet, une jointure copie entièrement la table de dimension sur chaque nœud de calcul. Pour déplacer les données, le plan de requête contient une opération appelée BroadcastMoveOperation. Ce type d’opération de déplacement des données ralentit les performances des requêtes. Il n’est pas utilisé par les tables répliquées. Pour afficher les étapes relatives au plan de requête, utilisez la vue catalogue système [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  

Par exemple, dans la requête suivante sur le schéma AdventureWorks, la table `FactInternetSales` est distribuée par hachage. Les tables `DimDate` et `DimSalesTerritory` sont des tables de dimension plus petites. Cette requête retourne le total des ventes en Amérique du Nord pour l’année fiscale 2004 :

```sql
SELECT [TotalSalesAmount] = SUM(SalesAmount)
FROM dbo.FactInternetSales s
INNER JOIN dbo.DimDate d
  ON d.DateKey = s.OrderDateKey
INNER JOIN dbo.DimSalesTerritory t
  ON t.SalesTerritoryKey = s.SalesTerritoryKey
WHERE d.FiscalYear = 2004
  AND t.SalesTerritoryGroup = 'North America'
```

Nous avons recréé les tables `DimDate` et `DimSalesTerritory` en tant que tables de distribution par tourniquet. Par conséquent, la requête affichait le plan de requête suivant, avec plusieurs opérations de déplacement pour la diffusion :

![Plan de requête de type tourniquet (round robin)](./media/design-guidance-for-replicated-tables/round-robin-tables-query-plan.jpg)

Nous avons recréé les tables `DimDate` et `DimSalesTerritory` en tant que tables répliquées et réexécuté la requête. Le plan de requête qui en résulte est beaucoup plus court et ne contient pas de mouvements de diffusion.

![Plan de requête répliqué](./media/design-guidance-for-replicated-tables/replicated-tables-query-plan.jpg)

## <a name="performance-considerations-for-modifying-replicated-tables"></a>Considérations relatives aux performances pour la modification des tables répliquées

Le pool SQL implémente une table répliquée en conservant une version principale de la table. Il copie la version principale dans la première base de données de distribution sur chaque nœud de calcul. En cas de modification, la version de référence est d’abord mise à jour, puis les tables sur chaque nœud de calcul sont reconstruites. Une reconstruction d’une table répliquée implique la copie de la table sur chaque nœud de calcul et la construction des index.  Par exemple, une table répliquée sur un DW2000c a 5 copies des données.  Une copie principale et une copie complète sur chaque nœud de calcul.  Toutes les données sont stockées dans des bases de données de distribution. Le pool SQL utilise ce modèle pour prendre en charge des instructions de modification de données plus rapides et des opérations de mise à l'échelle flexibles.

Les reconstructions asynchrones sont déclenchées par la première requête du tableau répliqué après :

- Des données sont chargées ou modifiées
- L'instance SQL Synapse est mise à l'échelle à un autre niveau
- La définition de la table est mise à jour

Les reconstructions ne sont pas requises après les événements suivants :

- Opération de suspension
- Opération de reprise

La reconstruction ne se produit pas immédiatement après la modification des données. Au lieu de cela, la reconstruction est déclenchée la première fois qu’une requête est sélectionnée à partir de la table.  La requête qui a déclenché la reconstruction lit immédiatement à partir de la version principale de la table pendant que les données sont copiées de façon asynchrone sur chaque nœud de calcul. Tant que la copie des données n’est pas terminée, les requêtes suivantes continuent d’utiliser la version principale de la table.  Si la table répliquée fait l’objet d’une activité qui entraîne une reconstruction, la copie des données est invalidée et l’instruction select suivante engendre une nouvelle copie des données.

### <a name="use-indexes-conservatively"></a>Utilisation restrictive des index

Les pratiques d’indexation standard s’appliquent aux tables répliquées. Le pool SQL reconstruit chaque index de table répliquée dans le cadre de la reconstruction. Utilisez les index uniquement lorsque le gain de performances compense le coût de reconstruction des index.

### <a name="batch-data-load"></a>Chargement de données par lots

Lors du chargement de données dans des tables répliquées, essayez de réduire les reconstructions en regroupant les chargements par lots. Effectuez tous les chargements par lots avant d’exécuter des instructions select.

Par exemple, ce modèle de chargement charge les données à partir de quatre sources et appelle quatre reconstructions.

- Charger à partir de la source 1.
- Instruction select qui déclenche la reconstruction 1.
- Charger à partir de la source 2.
- Instruction select qui déclenche la reconstruction 2.
- Charger à partir de la source 3.
- Instruction select qui déclenche la reconstruction 3.
- Charger à partir de la source 4.
- Instruction select qui déclenche la reconstruction 4.

Par exemple, ce modèle de chargement charge les données à partir de quatre sources mais n’appelle qu’une seule reconstruction.

- Charger à partir de la source 1.
- Charger à partir de la source 2.
- Charger à partir de la source 3.
- Charger à partir de la source 4.
- Instruction select qui déclenche la reconstruction.

### <a name="rebuild-a-replicated-table-after-a-batch-load"></a>Reconstruire une table répliquée après un chargement par lots

Pour garantir des temps d’exécution de requête cohérents, envisagez d’imposer la construction des tables répliquées après un chargement par lots. Sinon, la première requête recourt toujours à un déplacement.

Cette requête utilise le DMV [sys.pdw_replicated_table_cache_state](/sql/relational-databases/system-catalog-views/sys-pdw-replicated-table-cache-state-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour répertorier les tables répliquées qui ont été modifiées mais pas reconstruites.

```sql
SELECT [ReplicatedTable] = t.[name]
  FROM sys.tables t  
  JOIN sys.pdw_replicated_table_cache_state c  
    ON c.object_id = t.object_id
  JOIN sys.pdw_table_distribution_properties p
    ON p.object_id = t.object_id
  WHERE c.[state] = 'NotReady'
    AND p.[distribution_policy_desc] = 'REPLICATE'
```

Pour déclencher une reconstruction, exécutez l’instruction suivante sur chaque table dans la sortie précédente.

```sql
SELECT TOP 1 * FROM [ReplicatedTable]
```

## <a name="next-steps"></a>Étapes suivantes

Pour créer une table répliquée, utilisez l’une de ces instructions :

- [CREATE TABLE (pool SQL)](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE TABLE AS SELECT (pool SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

Pour une vue d’ensemble des tables distribuées, consultez [Tables distribuées](sql-data-warehouse-tables-distribute.md).
