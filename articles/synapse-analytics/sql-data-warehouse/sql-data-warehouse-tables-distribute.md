---
title: Guide de conception de tables distribuées
description: Recommandations pour la conception de tables distribuées par hachage et par tourniquet (round robin) à l’aide d’un pool SQL dédié dans Azure Synapse Analytics.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: c452d51018ef3f204cd7281971c07fb6337d39bf
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96449711"
---
# <a name="guidance-for-designing-distributed-tables-using-dedicated-sql-pool-in-azure-synapse-analytics"></a>Conseils pour la conception de tables distribuées à l’aide d’un pool SQL dédié dans Azure Synapse Analytics

Recommandations pour la conception de tables distribuées par hachage et par tourniquet (round robin) dans des pools SQL dédiés.

Cet article suppose que les concepts de distribution et de déplacement de données dans un pool SQL dédié vous sont familiers.  Pour plus d'informations, consultez [Architecture Azure Synapse Analytics](massively-parallel-processing-mpp-architecture.md).

## <a name="what-is-a-distributed-table"></a>Qu’est-ce qu’une table distribuée ?

Une table distribuée apparaît sous la forme d’une table unique, mais les lignes sont en réalité stockées sur 60 distributions. Les lignes sont distribuées avec un algorithme de hachage ou de tourniquet.  

Les **tables distribuées par hachage** améliorent les performances des requêtes sur des tables de faits volumineuses et sont au cœur de cet article. Les **tables distribuées par tourniquet** sont utiles pour améliorer la vitesse de chargement. Ces choix de conception ont un impact significatif sur l’amélioration des performances des requêtes et de chargement.

Une autre option de stockage de table est de répliquer une petite table sur tous les nœuds de calcul. Pour plus d’informations, consultez [Guide de conception pour les tables répliquées](design-guidance-for-replicated-tables.md). Pour choisir rapidement parmi les trois options, consultez Tables distribuées dans la [vue d’ensemble des tables](sql-data-warehouse-tables-overview.md).

Dans le cadre de la conception d’une table, essayez d’en savoir autant que possible sur vos données et la façon dont elles sont interrogées.    Considérez par exemple les questions suivantes :

- Quelle est la taille de la table ?
- Quelle est la fréquence d’actualisation de la table ?
- Est-ce que je dispose de tables de faits et de dimension dans un pool SQL dédié ?

### <a name="hash-distributed"></a>Distribution par hachage

Une table distribuée par hachage distribue les lignes de la table sur les nœuds de calcul à l’aide d’une fonction de hachage déterministe pour affecter chaque ligne à une [distribution](massively-parallel-processing-mpp-architecture.md#distributions).

![Table distribuée](./media/sql-data-warehouse-tables-distribute/hash-distributed-table.png "Table distribuée")  

Comme les valeurs identiques sont toujours hachées sur la même distribution, SQL Analytics dispose d'une connaissance intégrée de l'emplacement des lignes. Dans un pool SQL dédié, ces informations sont utilisées pour réduire le déplacement des données pendant les requêtes, ce qui améliore les performances de ces dernières.

Les tables distribuées par hachage fonctionnent correctement pour des tables de faits volumineuses dans un schéma en étoile. Elles peuvent contenir un très grand nombre de lignes et réaliser néanmoins des performances élevées. Il existe bien entendu certaines considérations relatives à la conception qui vous aident à obtenir les performances que le système distribué doit fournir. Le choix d’une colonne de distribution appropriée est l’une de ces considérations qui est décrite dans cet article.

Envisagez d’utiliser une table distribuée par hachage quand :

- La taille de la table sur le disque est supérieure à 2 Go.
- La table est l’objet d’opérations d’insertion, de mise à jour et de suppression fréquentes.

### <a name="round-robin-distributed"></a>Distribution par tourniquet

Une table distribuée par tourniquet distribue les lignes de la table uniformément sur toutes les distributions. L’attribution de lignes aux distributions est aléatoire. Contrairement aux tables distribuées par hachage, il n’est pas garanti que les lignes avec des valeurs égales soient affectées à la même distribution.

Par conséquent, le système doit parfois appeler une opération de déplacement des données pour mieux organiser vos données avant de pouvoir résoudre une requête.  Cette étape supplémentaire peut ralentir vos requêtes. Par exemple, la jointure d’une table distribuée par tourniquet nécessite généralement un remaniement des lignes, ce qui entraîne une baisse des performances.

Vous pouvez envisager une distribution par tourniquet des données de votre table dans les cas suivants :

- lors de la mise en route sous forme de point de départ simple, puisqu’il s’agit de l’option par défaut ;
- s’il n’existe aucune clé de jointure évidente ;
- s’il n’existe aucune colonne adaptée à la distribution par hachage de la table ;
- si la table ne partage aucune clé de jointure avec d’autres tables ;
- si la jointure est moins importante que d’autres dans la requête ;
- lorsque la table est une table temporaire intermédiaire ;

Le tutoriel [Chargement des données des taxis new-yorkais](load-data-from-azure-blob-storage-using-polybase.md#load-the-data-into-your-data-warehouse) donne un exemple de chargement de données dans une table de mise en lots distribuée par tourniquet (round robin).

## <a name="choosing-a-distribution-column"></a>Choix d’une colonne de distribution

Une table distribuée par hachage possède une colonne de distribution qui est la clé de hachage. Par exemple, le code suivant crée une table distribuée par hachage avec ProductKey comme colonne de distribution.

```sql
CREATE TABLE [dbo].[FactInternetSales]
(   [ProductKey]            int          NOT NULL
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
,  DISTRIBUTION = HASH([ProductKey])
)
;
```

Les données stockées dans la colonne de distribution peuvent être mises à jour. Ces mises à jour peuvent entraîner une lecture aléatoire des données.

Le choix d’une colonne de distribution est une décision de conception importante, car les valeurs de cette colonne déterminent la façon dont les lignes sont distribuées. Le meilleur choix dépend de plusieurs facteurs et implique généralement des compromis. Une fois la colonne de distribution choisie, vous ne pouvez plus la modifier.  

Si vous ne choisissez pas la meilleure colonne la première fois, vous pouvez utiliser [CTAS (CREATE TABLE AS SELECT)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour recréer la table avec une colonne de distribution différente.

### <a name="choose-a-distribution-column-with-data-that-distributes-evenly"></a>Choisir une colonne de distribution avec des données distribuées de manière uniforme

Pour obtenir des performances optimales, toutes les distributions doivent avoir environ le même nombre de lignes. Quand une ou plusieurs distributions ont un nombre disproportionné de lignes, certaines distributions finissent leur part d’une requête parallèle avant les autres. Comme la requête ne peut pas aboutir tant que toutes les distributions n’ont pas terminé le traitement, chaque requête est seulement aussi rapide que la distribution la plus lente.

- L’asymétrie des données signifie que les données ne sont pas réparties uniformément entre les distributions
- Le décalage de traitement signifie que certaines distributions prennent plus de temps que d’autres lors de l’exécution de requêtes parallèles. Cela peut se produire quand les données sont asymétriques.
  
Pour équilibrer le traitement parallèle, sélectionnez une colonne de distribution qui :

- **Possède de nombreuses valeurs uniques.** La colonne peut avoir quelques valeurs en double. Toutefois, toutes les lignes ayant la même valeur sont affectées à la même distribution. Avec 60 distributions, la colonne doit avoir au moins 60 valeurs uniques.  Généralement, le nombre de valeurs uniques est beaucoup plus important.
- **N’a pas de valeurs NULL, ou a uniquement quelques valeurs NULL.** Pour prendre un exemple extrême, si toutes les valeurs dans la colonne sont NULL, toutes les lignes sont affectées à la même distribution. Par conséquent, le traitement des requêtes est limité à une distribution et ne bénéficie pas du traitement parallèle.
- **N’est pas une colonne de date**. Toutes les données pour la même date arrivent dans la même distribution. Si plusieurs utilisateurs filtrent tous sur la même date, seule 1 des 60 distributions effectue tout le travail de traitement.

### <a name="choose-a-distribution-column-that-minimizes-data-movement"></a>Choisir une colonne de distribution qui réduit le déplacement des données

Pour obtenir un résultat de requête correct, les requêtes peuvent déplacer des données d’un nœud de calcul à un autre. Le déplacement des données se produit généralement quand les requêtes possèdent des jointures et agrégations sur des tables distribuées. Le choix d’une colonne de distribution qui réduit le déplacement des données est l’une des stratégies les plus importantes pour optimiser les performances votre pool SQL dédié.

Pour réduire le déplacement des données, sélectionnez une colonne de distribution qui :

- Est utilisée dans les clauses `JOIN`, `GROUP BY`, `DISTINCT`, `OVER` et `HAVING`. Quand deux tables de faits volumineuses ont des jointures fréquentes, la distribution des deux tables sur l’une des colonnes de jointure permet d’améliorer les performances des requêtes.  Quand une table n’est pas utilisée dans les jointures, envisagez de la distribuer sur une colonne qui figure fréquemment dans la clause `GROUP BY`.
- N’est *pas* utilisée dans les clauses `WHERE`. Cela peut affiner la requête pour qu’elle ne soit pas exécutée sur toutes les distributions.
- N’est *pas* une colonne de date. Les clauses WHERE filtrent souvent par date.  Dans ce cas, l’ensemble du traitement peut être exécuté sur seulement quelques distributions.

### <a name="what-to-do-when-none-of-the-columns-are-a-good-distribution-column"></a>Que faire quand aucune des colonnes ne constitue une colonne de distribution appropriée

Si aucune de vos colonnes n’a suffisamment de valeurs distinctes pour une colonne de distribution, vous pouvez créer une colonne en tant que composite d’une ou plusieurs valeurs. Pour éviter le déplacement des données pendant l’exécution de requête, utilisez la colonne de distribution composite en tant que colonne de jointure dans les requêtes.

Une fois que vous avez conçu une table distribuée par hachage, l’étape suivante consiste à charger des données dans la table.  Pour obtenir des conseils de chargement, consultez [Vue d’ensemble du chargement](design-elt-data-loading.md).

## <a name="how-to-tell-if-your-distribution-column-is-a-good-choice"></a>Comment savoir si votre colonne de distribution est un choix judicieux

Une fois que les données sont chargées dans une table distribuée par hachage, vérifiez si les lignes sont réparties uniformément entre les 60 distributions. Les lignes par distribution peuvent varier jusqu’à 10 % sans un impact perceptible sur les performances.

### <a name="determine-if-the-table-has-data-skew"></a>Déterminer si la table présente une asymétrie des données

Un moyen rapide de rechercher une asymétrie des données consiste à utiliser [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest). Le code SQL suivant retourne le nombre de lignes de la table qui sont stockées dans chacune des 60 distributions. Pour obtenir des performances équilibrées, les lignes de votre table distribuée doivent être partagées uniformément entre toutes les distributions.

```sql
-- Find data skew for a distributed table
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

Pour identifier les tables avec une asymétrie des données supérieure à 10 % :

1. Créez la vue dbo.vTableSizes qui figure dans l’article [Vue d’ensemble des tables](sql-data-warehouse-tables-overview.md#table-size-queries).  
2. Exécutez la requête suivante :

```sql
select *
from dbo.vTableSizes
where two_part_name in
    (
    select two_part_name
    from dbo.vTableSizes
    where row_count > 0
    group by two_part_name
    having (max(row_count * 1.000) - min(row_count * 1.000))/max(row_count * 1.000) >= .10
    )
order by two_part_name, row_count
;
```

### <a name="check-query-plans-for-data-movement"></a>Vérifier le déplacement des données dans les plans de requête

Une colonne de distribution appropriée permet un déplacement des données minimal pour les jointures et agrégations. Cela affecte la façon dont les jointures doivent être écrites. Pour obtenir un déplacement des données minimal pour une jointure sur deux tables distribuées par hachage, l’une des colonnes de jointure doit être la colonne de distribution.  Lors de la jointure de deux tables distribuées par hachage sur une colonne de distribution du même type de données, la jointure ne nécessite aucun déplacement des données. Les jointures peuvent utiliser des colonnes supplémentaires sans entraîner de déplacement des données.

Pour éviter le déplacement des données au cours d’une jointure :

- Les tables impliquées dans la jointure doivent être distribuées par hachage sur **l’une** des colonnes participant à la jointure.
- Les types de données des colonnes de jointure doivent être identiques dans les deux tables.
- Les colonnes doivent être jointes avec un opérateur d’égalité.
- Le type de jointure ne peut pas être `CROSS JOIN`.

Pour savoir si les requêtes subissent un déplacement des données, vous pouvez examiner le plan de requête.  

## <a name="resolve-a-distribution-column-problem"></a>Résoudre un problème de colonne de distribution

Il n’est pas nécessaire de résoudre tous les cas d’asymétrie des données. La distribution de données consiste à trouver le juste équilibre entre la réduction de l’asymétrie des données et la réduction du déplacement des données. Il n’est pas toujours possible de réduire à la fois l’asymétrie des données et le déplacement des données. L’avantage d’un déplacement des données minimal peut parfois compenser l’impact d’une asymétrie des données.

Pour déterminer si vous devez résoudre un décalage des données dans une table, vous devez comprendre au mieux les volumes de données et les requêtes dans votre charge de travail. Vous pouvez utiliser les étapes décrites dans l’article [Surveillance des requêtes](sql-data-warehouse-manage-monitor.md) pour surveiller l’impact de l’asymétrie sur les performances des requêtes. En particulier, recherchez la durée de l’exécution des requêtes volumineuses sur les distributions individuelles.

Comme vous ne pouvez pas modifier la colonne de distribution sur une table existante, la manière classique de résoudre l’asymétrie des données consiste à recréer la table avec une colonne de distribution différente.  

### <a name="re-create-the-table-with-a-new-distribution-column"></a>Recréer la table avec une nouvelle colonne de distribution

Cet exemple utilise [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour recréer une table avec une colonne de distribution par hachage différente.

```sql
CREATE TABLE [dbo].[FactInternetSales_CustomerKey]
WITH (  CLUSTERED COLUMNSTORE INDEX
     ,  DISTRIBUTION =  HASH([CustomerKey])
     ,  PARTITION       ( [OrderDateKey] RANGE RIGHT FOR VALUES (   20000101, 20010101, 20020101, 20030101
                                                                ,   20040101, 20050101, 20060101, 20070101
                                                                ,   20080101, 20090101, 20100101, 20110101
                                                                ,   20120101, 20130101, 20140101, 20150101
                                                                ,   20160101, 20170101, 20180101, 20190101
                                                                ,   20200101, 20210101, 20220101, 20230101
                                                                ,   20240101, 20250101, 20260101, 20270101
                                                                ,   20280101, 20290101
                                                                )
                        )
    )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
OPTION  (LABEL  = 'CTAS : FactInternetSales_CustomerKey')
;

--Create statistics on new table
CREATE STATISTICS [ProductKey] ON [FactInternetSales_CustomerKey] ([ProductKey]);
CREATE STATISTICS [OrderDateKey] ON [FactInternetSales_CustomerKey] ([OrderDateKey]);
CREATE STATISTICS [CustomerKey] ON [FactInternetSales_CustomerKey] ([CustomerKey]);
CREATE STATISTICS [PromotionKey] ON [FactInternetSales_CustomerKey] ([PromotionKey]);
CREATE STATISTICS [SalesOrderNumber] ON [FactInternetSales_CustomerKey] ([SalesOrderNumber]);
CREATE STATISTICS [OrderQuantity] ON [FactInternetSales_CustomerKey] ([OrderQuantity]);
CREATE STATISTICS [UnitPrice] ON [FactInternetSales_CustomerKey] ([UnitPrice]);
CREATE STATISTICS [SalesAmount] ON [FactInternetSales_CustomerKey] ([SalesAmount]);

--Rename the tables
RENAME OBJECT [dbo].[FactInternetSales] TO [FactInternetSales_ProductKey];
RENAME OBJECT [dbo].[FactInternetSales_CustomerKey] TO [FactInternetSales];
```

## <a name="next-steps"></a>Étapes suivantes

Pour créer une table distribuée, utilisez l’une de ces instructions :

- [CREATE TABLE (pool SQL dédié)](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE TABLE AS SELECT (pool SQL dédié)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
