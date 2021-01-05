---
title: Données distribuées - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Découvrez en quoi consistent les tables distribuées, les tables de référence, les tables locales et les partitions dans Azure Database pour PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 7757fdb4953640597a805c3d74a9e1ef08ef2c07
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86114497"
---
# <a name="distributed-data-in-azure-database-for-postgresql--hyperscale-citus"></a>Données distribuées dans Azure Database pour PostgreSQL – Hyperscale (Citus)

Cet article décrit les trois types de tables dans Azure Database pour PostgreSQL – Hyperscale (Citus).
Il montre comment les tables distribuées sont stockées sous forme de partitions et la manière dont les partitions sont placées sur les nœuds.

## <a name="table-types"></a>Types de tables

Il existe trois types de tables dans un groupe de serveurs Hyperscale (Citus), et chacun est utilisé à des fins différentes.

### <a name="type-1-distributed-tables"></a>Type 1 : Tables distribuées

Le premier type (le plus courant) est celui des tables distribuées. Elles ressemblent à des tables normales pour les instructions SQL, mais sont partitionnées horizontalement dans les nœuds worker. Cela signifie que les lignes de la table sont stockées sur des nœuds différents dans les tables de fragments appelées partitions.

Hyperscale (Citus) exécute non seulement des instructions SQL mais au DDL dans un cluster.
La modification du schéma d’une table distribuée a pour effet en cascade de mettre à jour toutes les partitions de table des Workers.

#### <a name="distribution-column"></a>Colonne de distribution

Hyperscale (Citus) utilise le partitionnement algorithmique pour affecter des lignes à des partitions. L’affectation est effectuée de façon déterministe en fonction de la valeur d’une colonne de table appelée colonne de distribution. L’administrateur de cluster doit désigner cette colonne lors de la distribution d’une table.
Il est important pour les performances et les fonctionnalités de faire le bon choix.

### <a name="type-2-reference-tables"></a>Type 2 : Tables de référence

Une table de référence est un type de table distribuée dont l’intégralité du contenu est concentré dans une seule partition. La partition est répliquée sur chaque Worker. Les requêtes sur un Worker peuvent accéder localement aux informations de référence, sans surcharger le réseau en demandant des lignes d’un autre nœud. Les tables de référence n’ont pas de colonne de distribution, car il n’est pas nécessaire de distinguer des partitions différentes par ligne.

Les tables de référence sont généralement petites et servent à stocker des données qui s’appliquent aux requêtes en cours d’exécution sur n’importe quel nœud Worker. C’est, par exemple, le cas des valeurs énumérées telles que les états de commande ou les catégories de produits.

### <a name="type-3-local-tables"></a>Type 3 : Tables locales

Lorsque vous utilisez Hyperscale (Citus), le nœud coordinateur auquel vous vous connectez est une base de données PostgreSQL normale. Vous pouvez créer des tables ordinaires sur le coordinateur et choisir de ne pas les partitionner.

Les petites tables d’administration qui ne font pas partie de requêtes de jointure font d’excellentes tables locales. C’est, par exemple, le cas d’une table d’utilisateurs pour l’authentification et la connexion à l’application.

## <a name="shards"></a>Partitions

La section précédente décrivait la façon dont les tables distribuées sont stockées en tant que partitions sur les nœuds worker. Cette section fournit d’autres détails techniques.

La table de métadonnées `pg_dist_shard` sur le coordinateur contient une ligne pour chaque partition de chaque table distribuée dans le système. La ligne correspond à un ID de partition avec une plage d’entiers dans un espace de hachage (shardminvalue, shardmaxvalue).

```sql
SELECT * from pg_dist_shard;
 logicalrelid  | shardid | shardstorage | shardminvalue | shardmaxvalue
---------------+---------+--------------+---------------+---------------
 github_events |  102026 | t            | 268435456     | 402653183
 github_events |  102027 | t            | 402653184     | 536870911
 github_events |  102028 | t            | 536870912     | 671088639
 github_events |  102029 | t            | 671088640     | 805306367
 (4 rows)
```

Si le nœud coordinateur souhaite déterminer quelle partition héberge une ligne de `github_events`, il hache la valeur de la colonne de distribution dans la ligne. Ensuite, le nœud vérifie quelle plage de la partition contient la valeur hachée. Les plages sont définies de sorte que l’image de la fonction de hachage soit leur union disjointe.

### <a name="shard-placements"></a>Placements de partition

Supposons que la partition 102027 est associée à la ligne en question. La ligne est lue ou écrite dans une table appelée `github_events_102027` dans un des Workers. Quel worker ? Cela est entièrement déterminé par les tables de métadonnées. Le mappage de partition au Worker est appelé positionnement de partition.

Le nœud coordinateur réécrit les requêtes en fragments qui font référence à des tables spécifiques, telles que `github_events_102027`, et exécute ces fragments sur les Workers appropriés. Voici un exemple d’une requête exécutée en arrière-plan pour rechercher le nœud contenant l’ID de partition 102027.

```sql
SELECT
    shardid,
    node.nodename,
    node.nodeport
FROM pg_dist_placement placement
JOIN pg_dist_node node
  ON placement.groupid = node.groupid
 AND node.noderole = 'primary'::noderole
WHERE shardid = 102027;
```

```output
┌─────────┬───────────┬──────────┐
│ shardid │ nodename  │ nodeport │
├─────────┼───────────┼──────────┤
│  102027 │ localhost │     5433 │
└─────────┴───────────┴──────────┘
```

## <a name="next-steps"></a>Étapes suivantes

- Apprendre à [choisir une colonne de distribution](concepts-hyperscale-choose-distribution-column.md) pour les tables distribuées.
