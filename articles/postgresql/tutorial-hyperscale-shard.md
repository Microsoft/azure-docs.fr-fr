---
title: 'Tutoriel : Partitionner des données sur des nœuds Worker – Hyperscale (Citus) – Azure Database pour PostgreSQL'
description: Ce tutoriel illustre comment créer des tables distribuées et visualiser leur distribution de données avec Hyperscale (Citus) Azure Database pour PostgreSQL.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 12/16/2020
ms.openlocfilehash: bc93c3643e329879e5118d1cfb61a356442df808
ms.sourcegitcommit: 86acfdc2020e44d121d498f0b1013c4c3903d3f3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97618390"
---
# <a name="tutorial-shard-data-on-worker-nodes-in-azure-database-for-postgresql--hyperscale-citus"></a>Tutoriel : Partitionner des données sur des nœuds Worker dans Azure Database pour PostgreSQL – Hyperscale (Citus)

Dans ce tutoriel, vous utilisez Azure Database pour PostgreSQL - Hyperscale (Citus) pour apprendre à :

> [!div class="checklist"]
> * Créer des tables distribuées par hachage
> * Voir où les partitions de table sont placées
> * Identifier une distribution non uniforme
> * Créer des contraintes sur des tables distribuées
> * Exécuter des requêtes sur des données distribuées

## <a name="prerequisites"></a>Prérequis

Ce tutoriel requiert un groupe de serveurs Hyperscale (Citus) en cours d’exécution avec deux nœuds Worker. Si vous n’avez pas de groupe de serveurs en cours d’exécution, suivez le tutoriel [Créer un groupe de serveurs](tutorial-hyperscale-server-group.md), puis revenez à celui-ci.

## <a name="hash-distributed-data"></a>Données distribuées par hachage

La distribution des lignes de la table sur plusieurs serveurs PostgreSQL est une technique clé pour les requêtes scalables dans Hyperscale (Citus). Ensemble, plusieurs nœuds peuvent contenir plus de données qu’une base de données traditionnelle et, dans de nombreux cas, ils peuvent utiliser les processeurs des Workers en parallèle pour exécuter des requêtes.

Dans la section des prérequis, nous avons créé un groupe de serveurs Hyperscale (Citus) avec deux nœuds Worker.

![coordinateur et deux Workers](tutorial-hyperscale-shard/nodes.png)

Les tables de métadonnées du nœud coordinateur effectuent le suivi des Workers et des données distribuées. Nous pouvons vérifier les Workers actifs dans la table [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table).

```sql
select nodeid from pg_dist_node where isactive;
```
```
 nodeid | nodename
--------+-----------
      1 | 10.0.0.21
      2 | 10.0.0.23
```

> [!NOTE]
> Les NodeNames sur Hyperscale (Citus) sont des adresses IP internes figurant dans un réseau virtuel, et les adresses réelles que vous voyez peuvent être différentes.

### <a name="rows-shards-and-placements"></a>Lignes, partitions et sélections électives

Pour utiliser les ressources de processeur et de stockage des nœuds Worker, nous devons distribuer les données de table dans le groupe de serveurs.  La distribution d’une table affecte chaque ligne à un groupe logique appelé *partition*. Nous allons créer une table et la distribuer :

```sql
-- create a table on the coordinator
create table users ( email text primary key, bday date not null );

-- distribute it into shards on workers
select create_distributed_table('users', 'email');
```

Hyperscale (Citus) attribue chaque ligne à une partition en fonction de la valeur de la *colonne de distribution*, que nous avons spécifiée ici comme `email`. Chaque ligne figurera dans exactement une partition et chaque partition peut contenir plusieurs lignes.

![table Users avec des lignes pointant vers des partitions](tutorial-hyperscale-shard/table.png)

Par défaut, `create_distributed_table()` crée 32 partitions, comme nous pouvons le voir en les comptant dans la table de métadonnées [pg_dist_shard](reference-hyperscale-metadata.md#shard-table) :

```sql
select logicalrelid, count(shardid)
  from pg_dist_shard
 group by logicalrelid;
```
```
 logicalrelid | count
--------------+-------
 users        |    32
```

Hyperscale (Citus) utilise la table `pg_dist_shard` pour assigner des lignes aux partitions, en fonction d’un hachage de la valeur dans la colonne de distribution. Les détails de hachage n’ont pas d’importance pour ce tutoriel. Il est important de noter que nous pouvons effectuer des requêtes pour voir quelles valeurs sont mappées aux ID de partitions :

```sql
-- Where would a row containing hi@test.com be stored?
-- (The value doesn't have to actually be present in users, the mapping
-- is a mathematical operation consulting pg_dist_shard.)
select get_shard_id_for_distribution_column('users', 'hi@test.com');
```
```
 get_shard_id_for_distribution_column
--------------------------------------
                               102008
```

Le mappage des lignes aux partitions est purement logique. Les partitions doivent être affectées à des nœuds Worker spécifiques pour être stockées, dans ce qu’Hyperscale (Citus) appelle la *sélection élective des partitions*.

![partitions assignées aux Workers](tutorial-hyperscale-shard/shard-placement.png)

Nous pouvons examiner les sélections électives des partitions dans [pg_dist_placement](reference-hyperscale-metadata.md#shard-placement-table).
La jointure de cela aux autres tables de métadonnées que nous avons vues montre où chaque partition se trouve.

```sql
-- limit the output to the first five placements

select
    shard.logicalrelid as table,
    placement.shardid as shard,
    node.nodename as host
from
    pg_dist_placement placement,
    pg_dist_node node,
    pg_dist_shard shard
where placement.groupid = node.groupid
  and shard.shardid = placement.shardid
order by shard
limit 5;
```
```
 table | shard  |    host
-------+--------+------------
 users | 102008 | 10.0.0.21
 users | 102009 | 10.0.0.23
 users | 102010 | 10.0.0.21
 users | 102011 | 10.0.0.23
 users | 102012 | 10.0.0.21
```

### <a name="data-skew"></a>Asymétrie des données

Un groupe de serveurs s’exécute plus efficacement lorsque vous placez des données uniformément sur les nœuds Worker, et lorsque vous placez les données associées sur les mêmes Workers. Dans cette section, nous allons nous concentrer sur la première partie, l’uniformité de la sélection élective.

Pour illustrer cela, nous allons créer des exemples de données pour notre table `users` :

```sql
-- load sample data
insert into users
select
    md5(random()::text) || '@test.com',
    date_trunc('day', now() - random()*'100 years'::interval)
from generate_series(1, 1000);
```

Pour voir les tailles des partitions, nous pouvons exécuter des [fonctions de taille de table](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-DBSIZE) sur les partitions.

```sql
-- sizes of the first five shards
select *
from
    run_command_on_shards('users', $cmd$
      select pg_size_pretty(pg_table_size('%1$s'));
    $cmd$)
order by shardid
limit 5;
```
```
 shardid | success | result
---------+---------+--------
  102008 | t       | 16 kB
  102009 | t       | 16 kB
  102010 | t       | 16 kB
  102011 | t       | 16 kB
  102012 | t       | 16 kB
```

Nous pouvons voir que les partitions sont de taille égale. Nous avons déjà vu que les sélections électives sont réparties uniformément entre les Workers, de sorte que nous pouvons déduire que les nœuds Worker contiennent à peu près le même nombre de lignes.

Les lignes de notre exemple `users` sont distribuées uniformément en raison des propriétés de la colonne de distribution, `email`.

1. Le nombre d’adresses e-mail était supérieur ou égal au nombre de partitions.
2. Le nombre de lignes par adresse e-mail était similaire (dans notre cas, exactement une ligne par adresse parce que nous avons déclaré email comme clé).

Un choix quelconque de table et de colonne de distribution pour lequel l’une ou l’autre des propriétés échoue aboutira à une taille de données inégale sur les Workers, autrement dit, à une *asymétrie des données*.

### <a name="add-constraints-to-distributed-data"></a>Ajouter des contraintes aux données distribuées

L’utilisation d’Hyperscale (Citus) vous permet de continuer de bénéficier de la sécurité d’une base de données relationnelle, y compris des [contraintes de base de données](https://www.postgresql.org/docs/current/ddl-constraints.html).
Toutefois, une limite s’applique. En raison de la nature des systèmes distribués, Hyperscale (Citus) n’établit pas de références croisées entre les nœuds Worker pour les contraintes d’unicité et l’intégrité référentielle.

Prenons notre exemple de table `users` avec une table associée.

```sql
-- books that users own
create table books (
    owner_email text references users (email),
    isbn text not null,
    title text not null
);

-- distribute it
select create_distributed_table('books', 'owner_email');
```

À des fins d’efficacité, nous distribuons `books` de la même façon que `users` : par l’adresse e-mail du propriétaire. La distribution par des valeurs de colonne similaires est appelée [colocation](concepts-hyperscale-colocation.md).

Nous n’avons eu aucun problème à distribuer les livres avec une clé étrangère aux utilisateurs, car la clé était sur une colonne de distribution. Toutefois, nous aurions eu du mal à définir `isbn` comme clé :

```sql
-- will not work
alter table books add constraint books_isbn unique (isbn);
```
```
ERROR:  cannot create constraint on "books"
DETAIL: Distributed relations cannot have UNIQUE, EXCLUDE, or
        PRIMARY KEY constraints that do not include the partition column
        (with an equality operator if EXCLUDE).
```

Dans une table distribuée, le mieux que nous pouvons faire est de définir le modulo unique des colonnes comme colonne de distribution :

```sql
-- a weaker constraint is allowed
alter table books add constraint books_isbn unique (owner_email, isbn);
```

La contrainte ci-dessus rend simplement isbn unique par utilisateur. Une autre option consiste à faire de books une [table de référence](howto-hyperscale-modify-distributed-tables.md#reference-tables) plutôt qu’une table distribuée, et à créer une table distribuée distincte qui associe les livres aux utilisateurs.

## <a name="query-distributed-tables"></a>Interroger les tables distribuées

Dans les sections précédentes, nous avons vu comment les lignes des tables distribuées sont placées dans des partitions sur les nœuds Worker. La plupart du temps, vous n’avez pas besoin de savoir comment ni où les données sont stockées dans un groupe de serveurs. Hyperscale (Citus) dispose d’un exécuteur de requêtes distribuées qui fractionne automatiquement les requêtes SQL régulières. Il les exécute en parallèle sur les nœuds Worker à proximité des données.

Par exemple, nous pouvons exécuter une requête pour rechercher l’âge moyen des utilisateurs, en traitant la table `users` distribuée comme s’il s’agissait d’une table normale sur le coordinateur.

```sql
select avg(current_date - bday) as avg_days_old from users;
```
```
    avg_days_old
--------------------
 17926.348000000000
```

![requête accédant aux partitions via le coordinateur](tutorial-hyperscale-shard/query-fragments.png)

En arrière-plan, l’exécuteur Hyperscale (Citus) crée une requête distincte pour chaque partition, les exécute sur les Workers et associe le résultat. Vous pouvez le voir en utilisant la commande PostgreSQL EXPLAIN :

```sql
explain select avg(current_date - bday) from users;
```
```
                                  QUERY PLAN
----------------------------------------------------------------------------------
 Aggregate  (cost=500.00..500.02 rows=1 width=32)
   ->  Custom Scan (Citus Adaptive)  (cost=0.00..0.00 rows=100000 width=16)
     Task Count: 32
     Tasks Shown: One of 32
     ->  Task
       Node: host=10.0.0.21 port=5432 dbname=citus
       ->  Aggregate  (cost=41.75..41.76 rows=1 width=16)
         ->  Seq Scan on users_102040 users  (cost=0.00..22.70 rows=1270 width=4)
```

La sortie montre un exemple de plan d’exécution pour un *fragment de requête* s’exécutant sur la partition 102040 (la table `users_102040` sur le Worker 10.0.0.21). Les autres fragments ne sont pas affichés, car ils sont similaires. Nous pouvons voir que le nœud Worker analyse les tables de partitions et applique l’agrégat. Le nœud coordinateur combine les agrégats pour fournir le résultat final.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, nous avons créé une table distribuée et étudié ses partitions et les sélections électives. Nous avons vu le défi que représente l’utilisation de contraintes d’unicité et de clé étrangère, et nous avons vu comment les requêtes distribuées fonctionnent à un niveau élevé.

* Obtenez plus d’informations sur les [types de table](concepts-hyperscale-nodes.md) Hyperscale (Citus).
* Obtenez davantage de conseils sur le [choix d’une colonne de distribution](concepts-hyperscale-choose-distribution-column.md).
* Découvrez les avantages de la [colocation des tables](concepts-hyperscale-colocation.md).
