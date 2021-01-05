---
title: Nœuds – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Découvrez les types de nœuds et tables au sein d’un groupe de serveurs dans Azure Database pour PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/28/2019
ms.openlocfilehash: b3eda2c8de8319552f32938f20ff98af0e0a49fc
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91314825"
---
# <a name="nodes-and-tables-in-azure-database-for-postgresql--hyperscale-citus"></a>Nœuds et tables dans Azure Database pour PostgreSQL - Hyperscale (Citus)

## <a name="nodes"></a>Nœuds

Le type d’hébergement Hyperscale (Citus) permet à Azure Database pour les serveurs PostgreSQL (appelés nœuds) de se coordonner entre eux dans une architecture dite « sans partage » (« shared nothing »). Collectivement, les nœuds dans un groupe de serveurs contiennent davantage de données et utilisent plus de cœurs d’UC que ce qui serait possible sur un seul serveur. L’architecture permet également à la base de données de procéder à la mise à l’échelle en ajoutant des nœuds supplémentaires au groupe de serveurs.

### <a name="coordinator-and-workers"></a>Coordinateur et travailleurs

Chaque groupe de serveurs dispose d’un nœud coordinateur et de plusieurs Workers. Les applications envoient leurs requêtes vers le nœud coordinateur, qui les relaie vers les Workers concernés et accumule leurs résultats. Les applications ne sont pas en mesure de se connecter directement aux Workers.

Hyperscale (CITUS) permet à l’administrateur de base de données de *distribuer* des tables, en stockant des différentes lignes sur différents nœuds Worker. Les tables distribuées sont la clé des performances d’Hyperscale (Citus). À défaut de distribuer les tables, celles-ci restent entièrement sur le nœud coordinateur et ne peut pas tirer parti du parallélisme entre ordinateurs.

Soit le coordinateur achemine chaque requête sur des tables distribuées vers un nœud Worker unique, soit il la met en parallèle sur plusieurs autres, en fonction de l’emplacement des données requises, sur un nœud unique ou sur plusieurs nœuds. Le coordinateur décide de la mesure à prendre en consultant les tables de métadonnées. Ces tables suivent les noms DNS et l’intégrité des nœuds Worker, ainsi que la distribution des données entre les nœuds.

## <a name="table-types"></a>Types de tables

Il existe trois types de tables dans un groupe de serveurs Hyperscale (Citus), et chacun est stocké différemment sur des nœuds et utilisé à des fins différentes.

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

- [Déterminer le type de votre application](concepts-hyperscale-app-type.md) pour préparer la modélisation des données
