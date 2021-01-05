---
title: Améliorer les performances de l’index columnstore
description: Réduisez les besoins de mémoire ou augmentez la mémoire disponible afin d’optimiser le nombre de lignes qu’un index columnstore compresse dans chaque rowgroup.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: e4656531f1424c3414ab00afbbe241a532738400
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462269"
---
# <a name="maximize-rowgroup-quality-for-columnstore-index-performance"></a>Optimiser la qualité du rowgroup pour améliorer le niveau de performance de l’index columnstore

La qualité du rowgroup est déterminée par le nombre de lignes d’un rowgroup. L’augmentation de la mémoire disponible peut optimiser le nombre de lignes qu’un index columnstore compresse dans chaque rowgroup.  Utilisez ces méthodes pour améliorer les taux de compression et les performances de requête pour les index columnstore.

## <a name="why-the-rowgroup-size-matters"></a>Importance de la taille de rowgroup

Dans la mesure où un index columnstore analyse une table en examinant les segments de colonne des rowgroups, l’optimisation du nombre de lignes dans chaque rowgroup améliore les performances de requête. Quand les rowgroups comportent un grand nombre de lignes, la compression des données s’améliore, ce qui signifie qu’il y a moins de données à lire à partir du disque.

Pour plus d’informations sur les rowgroups, voir [Description des index columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="target-size-for-rowgroups"></a>Taille cible des rowgroups

Pour optimiser les performances de requête, l’objectif est de maximiser le nombre de lignes par rowgroup dans un index columnstore. Un rowgroup peut compter au maximum 1 048 576 lignes. Vous ne devez pas nécessairement avoir le nombre maximal de lignes par rowgroup. Les index columnstore produisent de bonnes performances quand les rowgroups comprennent au moins 100 000 lignes.

## <a name="rowgroups-can-get-trimmed-during-compression"></a>Des rowgroups peuvent être découpés en cours de compression

Pendant un chargement en masse ou une reconstruction d’index columnstore, la mémoire disponible est parfois insuffisante pour compresser toutes les lignes désignées de chaque rowgroup. Lorsque la mémoire est sollicitée, les index columnstore découpent les rowgroups pour permettre la réussite de la compression dans le columnstore.

Quand la mémoire est insuffisante pour compresser au moins 10 000 lignes dans chaque rowgroup, une erreur est générée.

Pour plus d’informations sur le chargement en masse, voir [Chargement de données d’index columnstore](/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest#Bulk&preserve-view=true ).

## <a name="how-to-monitor-rowgroup-quality"></a>Comment surveiller la qualité du rowgroup

La vue de gestion dynamique sys.dm_pdw_nodes_db_column_store_row_group_physical_stats ([sys.dm_db_column_store_row_group_physical_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-column-store-row-group-physical-stats-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) contient la définition de vue correspondant à SQL DB) expose des informations utiles telles que le nombre de lignes dans les rowgroups et la raison du découpage, le cas échéant. Vous pouvez créer la vue suivante pour interroger facilement cette vue de gestion dynamique et obtenir ainsi des informations sur le découpage du rowgroup.

```sql
create view dbo.vCS_rg_physical_stats
as
with cte
as
(
select   tb.[name]                    AS [logical_table_name]
,        rg.[row_group_id]            AS [row_group_id]
,        rg.[state]                   AS [state]
,        rg.[state_desc]              AS [state_desc]
,        rg.[total_rows]              AS [total_rows]
,        rg.[trim_reason_desc]        AS trim_reason_desc
,        mp.[physical_name]           AS physical_name
FROM    sys.[schemas] sm
JOIN    sys.[tables] tb               ON  sm.[schema_id]          = tb.[schema_id]
JOIN    sys.[pdw_table_mappings] mp   ON  tb.[object_id]          = mp.[object_id]
JOIN    sys.[pdw_nodes_tables] nt     ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[dm_pdw_nodes_db_column_store_row_group_physical_stats] rg      ON  rg.[object_id]     = nt.[object_id]
                                                                            AND rg.[pdw_node_id]   = nt.[pdw_node_id]
                                        AND rg.[distribution_id]    = nt.[distribution_id]
)
select *
from cte;
```

trim_reason_desc indique si le rowgroup a été découpé (trim_reason_desc = NO_TRIM signifie qu’aucun découpage n’a été effectué et que la qualité du rowgroup est optimale). Les raisons suivantes indiquent un découpage prématuré du rowgroup :

- CHARGEMENT EN BLOC : cette raison de découpage est utilisée si le lot entrant de lignes pour la charge est inférieur à 1 million de lignes. Le moteur créera des rowgroups compressés si plus de 100 000 lignes sont insérées (par opposition à l’insertion dans le deltastore), mais affiche BULKLOAD comme raison du découpage. Dans ce scénario, vous pouvez augmenter votre charge de lot pour inclure davantage de lignes. En outre, réévaluez votre schéma de partitionnement pour vous assurer qu’il n’est pas trop granulaire car les rowgroups ne peuvent pas s’étendre sur plusieurs limites de partition.
- MEMORY_LIMITATION : pour créer des groupes de 1 million de lignes, le moteur requiert une certaine quantité de mémoire de travail. Lorsque la mémoire disponible de la session de chargement est inférieure à la mémoire de travail requise, les rowgroups sont prématurément découpés. Les sections suivantes expliquent comment estimer la mémoire requise et comment allouer davantage de mémoire.
- DICTIONARY_SIZE : cette raison indique qu’un rowgroup a été découpé, car il contenait au moins une colonne de chaîne avec des chaînes de cardinalité large et/ou haute. La taille du dictionnaire est limitée à 16 Mo de mémoire, et une fois cette limite atteinte, le rowgroup est compressé. Si cette situation se produit, vous pouvez isoler la colonne problématique dans un tableau distinct.

## <a name="how-to-estimate-memory-requirements"></a>Estimation des besoins en mémoire

La mémoire maximale nécessaire à la compression d’un rowgroup est d’environ :

- 72 Mo +
- \#lignes \* \#colonnes \* 8 octets +
- \#lignes \* \#colonnes de chaîne courte \* 32 octets +
- \#colonnes de chaîne longue \* 16 Mo pour le dictionnaire de compression

> [!NOTE]
> Où les colonnes de chaîne courte utilisent des données de type chaîne <= 32 octets, et les colonnes de chaîne longueur utilisent des données de type chaîne > 32 octets.

Les chaînes longues sont compressés avec une méthode de compression conçue pour la compression de texte. Cette méthode de compression utilise un *dictionnaire* pour stocker les modèles de texte. La taille maximale d’un dictionnaire est de 16 Mo. Il n’y qu’un seul dictionnaire pour chaque colonne de chaîne longue dans le rowgroup.

Pour obtenir une présentation détaillée des besoins en mémoire de columnstore, visionnez la vidéo [Mise à l’échelle de SQL Synapse : configuration et instructions](https://channel9.msdn.com/Events/Ignite/2016/BRK3291).

## <a name="ways-to-reduce-memory-requirements"></a>Réduction des besoins en mémoire

Pour réduire les besoins en mémoire pour la compression de rowgroups dans des index columnstore, utilisez les techniques suivantes.

### <a name="use-fewer-columns"></a>Utiliser moins de colonnes

Si possible, concevez la table avec moins de colonnes. Quand un rowgroup est compressé dans le columnstore, l’index columnstore compresse chaque segment de colonne séparément. Par conséquent, les besoins en mémoire pour compresser un rowgroup augmentent avec le nombre de colonnes.

### <a name="use-fewer-string-columns"></a>Utiliser moins de colonnes de chaîne

Les colonnes de données de type chaîne nécessitent davantage de mémoire que les données de type nombre et date. Pour réduire les besoins en mémoire, envisagez de supprimer des colonnes de type chaîne des tables de faits et de les placer dans des tables de dimension inférieure.

Besoins en mémoire supplémentaires pour la compression de chaîne :

- Les données de type chaîne comprenant jusqu’à 32 caractères peuvent nécessiter 32 octets supplémentaires par valeur.
- Les données de type chaîne comprenant plus de 32 caractères sont compressées à l’aide de méthodes de dictionnaire.  Chaque colonne dans le rowgroup peut nécessiter jusqu’à 16 Mo supplémentaires pour créer le dictionnaire.

### <a name="avoid-over-partitioning"></a>Éviter un partitionnement excessif

Les index columnstore créent un ou plusieurs rowgroups par partition. Pour l’entreposage des données dans Azure Synapse Analytics, le nombre de partitions augmente rapidement, car les données sont distribuées et chaque distribution est partitionnée. Si la table compte un trop grand nombre de partitions, il n’y a peut-être pas suffisamment de lignes pour remplir les rowgroups. Le manque de lignes ne crée pas de sollicitation de la mémoire lors de la compression, mais a pour effet que les rowgroups ne produisent pas des performances de requête columnstore optimales.

Une autre raison d’éviter un partitionnement excessif est que le chargement des lignes dans un index columnstore sur une table partitionnée entraîne une surcharge de mémoire. Lors d’un chargement, de nombreuses partitions pourraient recevoir les lignes entrantes qui sont conservées en mémoire jusqu’à ce que chaque partition compte suffisamment de lignes pour être compressée. Un trop grand nombre de partitions entraîne une sollicitation supplémentaire de la mémoire.

### <a name="simplify-the-load-query"></a>Simplifier la requête de chargement

La base de données partage l’allocation de mémoire pour une requête entre tous les opérateurs figurant dans la requête. Quand une requête de chargement comprend des tris et jointures complexes, la mémoire disponible pour la compression est réduite.

Concevez la requête de chargement pour vous concentrer uniquement sur le chargement de la requête. Si vous devez transformer les données, exécutez-les séparément de la requête de chargement. Par exemple, organisez les données dans une table de segments de mémoire, exécutez les transformations, puis chargez la table de mise en lots dans l’index columnstore. 

### <a name="adjust-maxdop"></a>Ajuster MAXDOP

Chaque distribution compresse les rowgroups dans le columnstore en parallèle lorsque plusieurs cœurs de processeur sont disponibles dans chaque distribution. Le parallélisme requiert des ressources de mémoire supplémentaires, qui peuvent entraîner une sollicitation de la mémoire et un découpage de rowgroup.

Pour réduire la sollicitation de la mémoire, vous pouvez utiliser l’indicateur de requête MAXDOP pour forcer l’exécution de l’opération de chargement en mode série au sein de chaque distribution.

```sql
CREATE TABLE MyFactSalesQuota
WITH (DISTRIBUTION = ROUND_ROBIN)
AS SELECT * FROM FactSalesQuota
OPTION (MAXDOP 1);
```

## <a name="ways-to-allocate-more-memory"></a>Méthodes pour allouer davantage de mémoire

La taille de DWU et la classe de ressources utilisateur déterminent ensemble la quantité de mémoire disponible pour une requête utilisateur. Pour augmenter l’allocation de mémoire pour une requête de chargement, vous pouvez augmenter soit le nombre de DWU, soit la classe de ressources.

- Pour augmenter le nombre de DWU, voir [Comment mettre les performances à l’échelle ?](../sql-data-warehouse/quickstart-scale-compute-portal.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
- Pour modifier la classe de ressources pour une requête, voir [Exemple de modification d’une classe de ressources utilisateur](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#change-a-users-resource-class).

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir d’autres façons d’améliorer les performances dans SQL Synapse, consultez [Vue d’ensemble des performances](../overview-cheat-sheet.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

 
