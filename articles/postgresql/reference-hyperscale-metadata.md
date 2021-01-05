---
title: Tables système – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Métadonnées pour l’exécution de requêtes distribuées
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: 74403365fe48584fa5d1db0e349c9dfc3772d874
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652848"
---
# <a name="system-tables-and-views"></a>Tables système et vues

Hyperscale (Citus) crée et gère des tables spéciales qui contiennent des informations sur les données distribuées dans le groupe de serveurs. Le nœud coordinateur consulte ces tables lors de la planification de l’exécution de requêtes sur les nœuds Worker.

## <a name="coordinator-metadata"></a>Métadonnées du coordinateur

Hyperscale (Citus) divise chaque table distribuée en plusieurs partitions logiques en fonction de la colonne de distribution. Le coordinateur gère ensuite les tables de métadonnées pour effectuer le suivi des statistiques et des informations sur l’intégrité et l’emplacement de ces partitions.

Dans cette section, nous décrivons chacune de ces tables de métadonnées et leur schéma.
Vous pouvez afficher et interroger ces tables à l’aide de SQL après vous être connecté au nœud coordinateur.

> [!NOTE]
>
> Les groupes de serveurs Hyperscale (Citus) exécutant des versions antérieures du CItus Engine peuvent ne pas proposer toutes les tables listées ci-dessous.

### <a name="partition-table"></a>Table de partition

La table de partition pg\_dist\_ stocke les métadonnées sur les tables de la base de données qui sont distribuées. Pour chaque table distribuée, elle stocke également des informations sur la méthode de distribution et des informations détaillées sur la colonne de distribution.

| Nom         | Type     | Description                                                                                                                                                                                                                                           |
|--------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| logicalrelid | regclass | Table distribuée à laquelle cette ligne correspond. Cette valeur fait référence à la colonne relfilenode de la table du catalogue système pg_class.                                                                                                                   |
| partmethod   | char     | Méthode utilisée pour le partitionnement/la distribution. Les valeurs de cette colonne correspondant aux différentes méthodes de distribution sont Append : « a », Hash : « h », table de référence : « n »                                                                          |
| partkey      | text     | Informations détaillées sur la colonne de distribution, y compris le numéro de colonne, le type et d’autres informations pertinentes.                                                                                                                                      |
| colocationid | entier  | Groupe de colocation auquel cette table appartient. Les tables dans le même groupe autorisent les jointures colocalisées et les cumuls distribués entre autres optimisations. Cette valeur fait référence à la colonne colocationid de la table pg_dist_colocation.                      |
| repmodel     | char     | Méthode utilisée pour la réplication des données. Les valeurs de cette colonne correspondant aux différentes méthodes de réplication sont les suivantes : Réplication basée sur une instruction Citus : « c », réplication de streaming PostgreSQL : « s », validation en deux phases (pour les tables de référence) : « t » |

```
SELECT * from pg_dist_partition;
 logicalrelid  | partmethod |                                                        partkey                                                         | colocationid | repmodel 
---------------+------------+------------------------------------------------------------------------------------------------------------------------+--------------+----------
 github_events | h          | {VAR :varno 1 :varattno 4 :vartype 20 :vartypmod -1 :varcollid 0 :varlevelsup 0 :varnoold 1 :varoattno 4 :location -1} |            2 | c
 (1 row)
```

### <a name="shard-table"></a>Table partition

La table partition\_dist\_stocke les métadonnées relatives aux partitions individuelles d’une table. Pg_dist_shard contient des informations sur la table distribuée à laquelle appartiennent les partitions et des statistiques sur la colonne de distribution pour partitions.
Pour des tables distribuées append, ces statistiques correspondent aux valeurs min/max de la colonne de distribution. Pour des tables distribuées hash, il s’agit de plages de jetons de hachage affectées à cette partition. Ces statistiques sont utilisées pour nettoyer les partitions non liés pendant les requêtes SELECT.

| Nom          | Type     | Description                                                                                                                                                                                  |
|---------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| logicalrelid  | regclass | Table distribuée à laquelle cette ligne correspond. Cette valeur fait référence à la colonne relfilenode de la table du catalogue système pg_class.                                                          |
| shardid       | bigint   | Identificateur général unique affecté à la partition.                                                                                                                                           |
| shardstorage  | char     | Type de stockage utilisé pour cette partition. Les différents types de stockage sont décrits dans le tableau ci-dessous.                                                                                               |
| shardminvalue | text     | Pour les tables distribuées append, valeur minimale de la colonne de distribution dans cette partition (valeur incluse). Pour les tables distribuées hash, il s’agit de la valeur minimale de jetons de hachage affectées à cette partition (valeur incluse). |
| shardmaxvalue | text     | Pour les tables distribuées append, valeur maximale de la colonne de distribution dans cette partition (valeur incluse). Pour les tables distribuées hash, il s’agit de la valeur maximale de jetons de hachage affectées à cette partition (valeur incluse). |

```
SELECT * from pg_dist_shard;
 logicalrelid  | shardid | shardstorage | shardminvalue | shardmaxvalue 
---------------+---------+--------------+---------------+---------------
 github_events |  102026 | t            | 268435456     | 402653183
 github_events |  102027 | t            | 402653184     | 536870911
 github_events |  102028 | t            | 536870912     | 671088639
 github_events |  102029 | t            | 671088640     | 805306367
 (4 rows)
```

#### <a name="shard-storage-types"></a>Type de stockage Azure

La colonne shardstorage dans pg\_dist\_shard indique le type de stockage utilisé pour la partition. Vous trouverez ci-dessous un bref aperçu des différents types de stockage de partition et de leur représentation.

| Type de stockage | Valeur shardstorage | Description                                                                        |
|--------------|--------------------|------------------------------------------------------------------------------------|
| TABLE        | « t »                | Indique que shard stocke les données appartenant à une table distribuée normale.         |
| COLUMNAR     | « c »                | Indique que shard stocke les données en colonnes. (Utilisé par les tables de cstore_fdw distribuées) |
| FOREIGN      | « f »                | Indique que shard stocke des données étrangères. (Utilisé par les tables de file_fdw distribuées)    |

### <a name="shard-placement-table"></a>Table de sélection élective de partition

La table pg\_dist\_placement suit l’emplacement des réplicas de partition sur les nœuds Worker. Chaque réplica d’une partition affectée à un nœud spécifique est appelé une sélection élective de partition. Cette table stocke des informations relatives à l’intégrité et l’emplacement de chaque sélection élective de partition.

| Nom        | Type   | Description                                                                                                                               |
|-------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------|
| shardid     | bigint | Identificateur de partition associé à ce sélection élective. Cette valeur fait référence à la colonne shardid de la table du catalogue pg_dist_shard.             |
| shardstate  | int    | Décrit l’état de cette sélection élective. Les différents états de partition sont décrits dans la section ci-dessous.                                         |
| shardlength | bigint | Pour des tables distribuées append, taille en octets de la sélection élective de partition sur le nœud Worker. Pour les tables de hachage distribuées, zéro.            |
| placementid | bigint | Identificateur unique généré automatiquement pour chaque sélection élective individuelle.                                                                           |
| groupid     | int    | Distingue un groupe d’un serveur principal et zéro, un ou plusieurs serveurs secondaires lors de l’utilisation du modèle de réplication de streaming. |

```
SELECT * from pg_dist_placement;
  shardid | shardstate | shardlength | placementid | groupid
 ---------+------------+-------------+-------------+---------
   102008 |          1 |           0 |           1 |       1
   102008 |          1 |           0 |           2 |       2
   102009 |          1 |           0 |           3 |       2
   102009 |          1 |           0 |           4 |       3
   102010 |          1 |           0 |           5 |       3
   102010 |          1 |           0 |           6 |       4
   102011 |          1 |           0 |           7 |       4
```

#### <a name="shard-placement-states"></a>États de sélection élective de partition

Hyperscale (Citus) gère l’intégrité des partitions par sélection élective. Si une sélection élective place le système dans un état incohérent, Citus le marque automatiquement comme étant indisponible. L’état de sélection élective est enregistré dans la table pg_dist_shard_placement, dans la colonne shardstate. Voici un bref aperçu des différents états de sélection élective de partition :

| Nom de l’état | Valeur shardstate | Description                                                                                                                                                                                                                                                                                                                                                                                                                         |
|------------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FINALIZED  | 1                | L’état dans lequel les nouvelles partitions sont créées. Les sélections électives de partition dans cet état sont considérées à jour et utilisées dans la planification et l’exécution des requêtes.                                                                                                                                                                                                                                                                                 |
| INACTIVE   | 3                | Les sélections électives de partition dans cet état sont considérés comme inactifs en raison d’une désynchronisation avec d’autres réplicas de la même partition. L’état peut se produire lorsqu’une opération d’ajout, de modification (INSERT, UPDATE, DELETE) ou de DDL échoue pour cet emplacement. Le planificateur de requêtes ignorera les sélections électives dans cet état au cours de la planification et de l’exécution. Les utilisateurs peuvent synchroniser les données de ces partitions avec un réplica finalisé en tant qu’activité en arrière-plan. |
| TO_DELETE  | 4                | Si Citus tente de supprimer une sélection élective de partition en réponse à un appel de master_apply_delete_command et échoue, le placement est déplacé vers cet état. Les utilisateurs peuvent ensuite supprimer ces partitions en tant qu’activité d’arrière-plan suivante.                                                                                                                                                                                                              |

### <a name="worker-node-table"></a>Table de nœud Worker

La table pg\_dist\_node contient des informations sur les nœuds Worker du cluster.

| Nom             | Type    | Description                                                                                                                                                                                |
|------------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| nodeId           | int     | Identificateur généré automatiquement pour un nœud individuel.                                                                                                                                          |
| groupid          | int     | Identificateur utilisé pour distinguer un groupe d’un serveur principal et zéro, un ou plusieurs serveurs secondaires lors de l’utilisation du modèle de réplication de streaming. Par défaut, il est identique au nodeid.         |
| nodename         | text    | Nom d’hôte ou adresse IP du nœud Worker PostgreSQL.                                                                                                                                     |
| nodeport         | int     | Numéro de port sur lequel le nœud worker PostgreSQL écoute.                                                                                                                              |
| noderack         | text    | (Facultatif) Informations de sélection élective du rack pour le nœud Worker.                                                                                                                                 |
| hasmetadata      | boolean | Réservé à un usage interne.                                                                                                                                                                 |
| isactive         | boolean | Indique si le nœud est actif et accepte les sélections électives de partition.                                                                                                                                     |
| noderole         | text    | Si le nœud est un nœud principal ou secondaire                                                                                                                                                 |
| nodecluster      | text    | Nom du cluster contenant ce nœud                                                                                                                                               |
| shouldhaveshards | boolean | Si la valeur est false, les partitions sont déplacées hors du nœud (drainés) lors du rééquilibrage, et les partitions de nouvelles tables distribuées sont placées sur le nœud, sauf s’ils sont colocalisés avec des partitions déjà présentes. |

```
SELECT * from pg_dist_node;
 nodeid | groupid | nodename  | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | shouldhaveshards
--------+---------+-----------+----------+----------+-------------+----------+----------+-------------+------------------
      1 |       1 | localhost |    12345 | default  | f           | t        | primary  | default     | t
      2 |       2 | localhost |    12346 | default  | f           | t        | primary  | default     | t
      3 |       3 | localhost |    12347 | default  | f           | t        | primary  | default     | t
(3 rows)
```

### <a name="distributed-object-table"></a>Table d’objet distribuée

La table citus.pg\_dist\_object contient une liste d’objets tels que les types et les fonctions qui ont été créés sur le nœud coordinateur et propagés aux nœuds Worker. Lorsqu’un administrateur ajoute de nouveaux nœuds Worker au cluster, hyperscale (Citus) crée automatiquement des copies des objets distribués sur les nouveaux nœuds (dans le bon ordre pour répondre aux dépendances des objets).

| Nom                        | Type    | Description                                          |
|-----------------------------|---------|------------------------------------------------------|
| classid                     | oid     | Classe de l’objet distribué                      |
| objid                       | oid     | ID d’objet de l’objet distribué                  |
| objsubid                    | entier | Sous-ID d’objet de l’objet distribué, par exemple, attnum |
| type                        | text    | Partie de l’adresse stable utilisée lors des mises à niveau de pg   |
| object_names                | text[]  | Partie de l’adresse stable utilisée lors des mises à niveau de pg   |
| object_args                 | text[]  | Partie de l’adresse stable utilisée lors des mises à niveau de pg   |
| distribution_argument_index | entier | Valide uniquement pour les fonctions/procédures distribuées      |
| colocationid                | entier | Valide uniquement pour les fonctions/procédures distribuées      |

Les \"adresses stables\" identifient de manière unique les objets indépendamment d’un serveur spécifique. Hyperscale (Citus) effectue le suivi des objets au cours d’une mise à niveau PostgreSQL à l’aide d’adresses stables créées avec la fonction [pg\_identify\_object\_as\_adress()](https://www.postgresql.org/docs/current/functions-info.html#FUNCTIONS-INFO-OBJECT-TABLE).

\'Voici un exemple de comment `create_distributed_function()` ajoute des entrées à la table `citus.pg_dist_object` :

```psql
CREATE TYPE stoplight AS enum ('green', 'yellow', 'red');

CREATE OR REPLACE FUNCTION intersection()
RETURNS stoplight AS $$
DECLARE
        color stoplight;
BEGIN
        SELECT *
          FROM unnest(enum_range(NULL::stoplight)) INTO color
         ORDER BY random() LIMIT 1;
        RETURN color;
END;
$$ LANGUAGE plpgsql VOLATILE;

SELECT create_distributed_function('intersection()');

-- will have two rows, one for the TYPE and one for the FUNCTION
TABLE citus.pg_dist_object;
```

```
-[ RECORD 1 ]---------------+------
classid                     | 1247
objid                       | 16780
objsubid                    | 0
type                        |
object_names                |
object_args                 |
distribution_argument_index |
colocationid                |
-[ RECORD 2 ]---------------+------
classid                     | 1255
objid                       | 16788
objsubid                    | 0
type                        |
object_names                |
object_args                 |
distribution_argument_index |
colocationid                |
```

### <a name="colocation-group-table"></a>Table de groupe de colocation

La table pg\_dist\_colocation contient des informations sur les tables\' partitions devant être placées ensemble ou [colocalisée](concepts-hyperscale-colocation.md).
Lorsque deux tables se trouvent dans le même groupe de colocation, Hyperscale (Citus) garantit que les partitions avec les mêmes valeurs de partition seront placées sur les mêmes nœuds Worker.
La colocation active les optimisations de jointure, certains rollups distribués et la prise en charge de clé étrangère. La colocation de partition est déduite lorsque le nombre de partition, les facteurs de réplication et les types de colonne de partition correspondent tous entre deux tables. Toutefois, un groupe de colocation personnalisé peut être spécifié lors de la création d’une table distribuée, si vous le souhaitez.

| Nom                   | Type | Description                                                                   |
|------------------------|------|-------------------------------------------------------------------------------|
| colocationid           | int  | Identificateur unique du groupe de colocation auquel cette ligne correspond.          |
| shardcount             | int  | Nombre de partitions pour toutes les tables de ce groupe de colocation                          |
| replicationfactor      | int  | Facteur de réplication pour toutes les tables de ce groupe de colocation.                  |
| distributioncolumntype | oid  | Type de la colonne de distribution pour toutes les tables de ce groupe de colocation. |

```
SELECT * from pg_dist_colocation;
  colocationid | shardcount | replicationfactor | distributioncolumntype 
 --------------+------------+-------------------+------------------------
             2 |         32 |                 2 |                     20
  (1 row)
```

### <a name="rebalancer-strategy-table"></a>Table de stratégie de rééquilibrage

Ce tableau définit les stratégies que [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards) peut utiliser pour déterminer où déplacer les partitions.

| Nom                           | Type    | Description                                                                                                                                       |
|--------------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| default_strategy               | boolean | Indique si rebalance_table_shards doit choisir cette stratégie par défaut. Utiliser citus_set_default_rebalance_strategy pour mettre à jour cette colonne             |
| shard_cost_function            | regproc | Identificateur d’une fonction cost, qui doit prendre un shardid en tant que bigint et retourner sa notion de cost comme type réel                                |
| node_capacity_function         | regproc | Identificateur d’une fonction capacity, qui doit prendre un nodeid en tant que int et retourner sa notion de node capacity (capacité de nœud) comme type réel                          |
| shard_allowed_on_node_function | regproc | Identificateur pour une fonction qui avec shardid bigint, et nodeidarg int, donnés, retourne la valeur booléenne pour déterminer si Citus peut stocker la partition sur le nœud |
| default_threshold              | float4  | Seuil pour estimer qu’un nœud est trop plein ou trop vide, qui détermine quand le rebalance_table_shards doit essayer de déplacer les partitions                    |
| minimum_threshold              | float4  | Protection permettant d’empêcher de définir l’argument du seuil rebalance_table_shards() à un niveau trop bas                                                  |

Une installation Hyperscale (Citus) est fournie avec ces stratégies dans le tableau suivant :

```postgresql
SELECT * FROM pg_dist_rebalance_strategy;
```

```
-[ RECORD 1 ]-------------------+-----------------------------------
Name                            | by_shard_count
default_strategy                | true
shard_cost_function             | citus_shard_cost_1
node_capacity_function          | citus_node_capacity_1
shard_allowed_on_node_function  | citus_shard_allowed_on_node_true
default_threshold               | 0
minimum_threshold               | 0
-[ RECORD 2 ]-------------------+-----------------------------------
Name                            | by_disk_size
default_strategy                | false
shard_cost_function             | citus_shard_cost_by_disk_size
node_capacity_function          | citus_node_capacity_1
shard_allowed_on_node_function  | citus_shard_allowed_on_node_true
default_threshold               | 0.1
minimum_threshold               | 0.01
```

La stratégie par défaut, `by_shard_count`, affecte à chaque partition le même coût. Son effet est d’égaliser le nombre de partition entre les nœuds. L’autre stratégie prédéfinie, `by_disk_size`, attribue un coût à chaque partition correspondant à sa taille de disque en octets plus celle des partitions colocalisées avec. La taille du disque est calculée à l’aide de `pg_total_relation_size`, ce qui signifie qu’elle comprend des index. Cette stratégie tente d’obtenir le même espace disque sur chaque nœud. Notez le seuil de 0,1 qui empêche les mouvements de partition inutiles causés par des différences non significatives dans l’espace disque.

#### <a name="creating-custom-rebalancer-strategies"></a>Création de stratégies de rééquilibrage personnalisées

Voici des exemples de fonctions qui peuvent être utilisées dans les nouvelles stratégies de rééquilibrage partition et enregistrées dans le [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md?#rebalancer-strategy-table) avec la fonction [citus_add_rebalance_strategy](reference-hyperscale-functions.md#citus_add_rebalance_strategy).

-   Définition d’une exception de capacité de nœud par modèle de nom d’hôte :

    ```postgresql
    CREATE FUNCTION v2_node_double_capacity(nodeidarg int)
        RETURNS boolean AS $$
        SELECT
            (CASE WHEN nodename LIKE '%.v2.worker.citusdata.com' THEN 2 ELSE 1 END)
        FROM pg_dist_node where nodeid = nodeidarg
        $$ LANGUAGE sql;
    ```

-   Rééquilibrage en fonction du nombre de requêtes qui accèdent à un partition, tel qu’il est mesuré par le [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table):

    ```postgresql
    -- example of shard_cost_function

    CREATE FUNCTION cost_of_shard_by_number_of_queries(shardid bigint)
        RETURNS real AS $$
        SELECT coalesce(sum(calls)::real, 0.001) as shard_total_queries
        FROM citus_stat_statements
        WHERE partition_key is not null
            AND get_shard_id_for_distribution_column('tab', partition_key) = shardid;
    $$ LANGUAGE sql;
    ```

-   Isolation d’un partition spécifique (10000) sur un nœud (adresse \'10.0.0.1\') :

    ```postgresql
    -- example of shard_allowed_on_node_function

    CREATE FUNCTION isolate_shard_10000_on_10_0_0_1(shardid bigint, nodeidarg int)
        RETURNS boolean AS $$
        SELECT
            (CASE WHEN nodename = '10.0.0.1' THEN shardid = 10000 ELSE shardid != 10000 END)
        FROM pg_dist_node where nodeid = nodeidarg
        $$ LANGUAGE sql;

    -- The next two definitions are recommended in combination with the above function.
    -- This way the average utilization of nodes is not impacted by the isolated shard.
    CREATE FUNCTION no_capacity_for_10_0_0_1(nodeidarg int)
        RETURNS real AS $$
        SELECT
            (CASE WHEN nodename = '10.0.0.1' THEN 0 ELSE 1 END)::real
        FROM pg_dist_node where nodeid = nodeidarg
        $$ LANGUAGE sql;
    CREATE FUNCTION no_cost_for_10000(shardid bigint)
        RETURNS real AS $$
        SELECT
            (CASE WHEN shardid = 10000 THEN 0 ELSE 1 END)::real
        $$ LANGUAGE sql;
    ```

### <a name="query-statistics-table"></a>Table des statistiques sur les requêtes

Hyperscale (Citus) fournit `citus_stat_statements` pour les statistiques sur la façon dont les requêtes sont exécutées et pour qui. \'Elle est analogue à (et peut être joint avec) la vue [pg\_stat\_statements](https://www.postgresql.org/docs/current/static/pgstatstatements.html) dans PostgreSQL, qui effectue le suivi des statistiques sur la vitesse des requêtes.

Cette vue peut suivre des requêtes à des locataires d’origine dans une application multi-entité, ce qui permet de décider quand procéder à l’isolation des locataires.

| Nom          | Type   | Description                                                                      |
|---------------|--------|----------------------------------------------------------------------------------|
| queryid       | bigint | identificateur (correct pour les jointures de pg_stat_statements)                                   |
| userid        | oid    | utilisateur qui a exécuté la requête                                                           |
| dbid          | oid    | instance de base de données du coordinateur                                                 |
| query         | text   | chaîne de requête anonyme                                                          |
| exécuteur      | text   | Citus executor utilisé : adaptatif, en temps réel, suivi des tâches, routeur ou insert-select |
| partition_key | text   | valeur de la colonne de distribution dans les requêtes exécutées par le routeur, sinon NULL               |
| calls         | bigint | nombre de fois où la requête a été exécutée                                                |

```sql
-- create and populate distributed table
create table foo ( id int );
select create_distributed_table('foo', 'id');
insert into foo select generate_series(1,100);

-- enable stats
-- pg_stat_statements must be in shared_preload libraries
create extension pg_stat_statements;

select count(*) from foo;
select * from foo where id = 42;

select * from citus_stat_statements;
```

Résultats :

```
-[ RECORD 1 ]-+----------------------------------------------
queryid       | -909556869173432820
userid        | 10
dbid          | 13340
query         | insert into foo select generate_series($1,$2)
executor      | insert-select
partition_key |
calls         | 1
-[ RECORD 2 ]-+----------------------------------------------
queryid       | 3919808845681956665
userid        | 10
dbid          | 13340
query         | select count(*) from foo;
executor      | adaptive
partition_key |
calls         | 1
-[ RECORD 3 ]-+----------------------------------------------
queryid       | 5351346905785208738
userid        | 10
dbid          | 13340
query         | select * from foo where id = $1
executor      | adaptive
partition_key | 42
calls         | 1
```

Avertissements :

-   \'Les données de statistiques ne sont pas répliquées et ne vont pas survivre à des pannes de base de données ou à un basculement
-   Effectue le suivi d’un nombre limité de requêtes, défini par le `pg_stat_statements.max` GUC (par défaut 5000)
-   Pour tronquer la table, utilisez la fonction `citus_stat_statements_reset()`

### <a name="distributed-query-activity"></a>Activité des requêtes distribuées

Hyperscale (Citus) fournit des affichages spéciaux pour surveiller les requêtes et les verrous dans le cluster, y compris les requêtes spécifiques à une partition utilisées en interne pour générer les résultats des requêtes distribuées.

-   **ciuts\_dist\_stat\_activity** : affiche les requêtes distribuées qui s’exécutent sur tous les nœuds. Sur-ensemble de `pg_stat_activity`, utilisable partout où se trouve ce dernier.
-   **citus\_worker\_stat\_activity** : affiche les requêtes sur les Workers, y compris les requêtes de fragment sur les partitions individuels.
-   **citus\_lock\_waits** : Requêtes bloquées dans le cluster.

Les deux premières vues incluent toutes les colonnes de [pg\_stat\_activity](https://www.postgresql.org/docs/current/static/monitoring-stats.html#PG-STAT-ACTIVITY-VIEW) plus l’hôte/port hôte du Worker qui a initié la requête et l’hôte/port du nœud coordinateur du cluster.

Par exemple, envisagez de compter les lignes dans une table distribuée :

```postgresql
-- run from worker on localhost:9701

SELECT count(*) FROM users_table;
```

Nous pouvons voir que la requête s’affiche dans `citus_dist_stat_activity` :

```postgresql
SELECT * FROM citus_dist_stat_activity;

-[ RECORD 1 ]----------+----------------------------------
query_hostname         | localhost
query_hostport         | 9701
master_query_host_name | localhost
master_query_host_port | 9701
transaction_number     | 1
transaction_stamp      | 2018-10-05 13:27:20.691907+03
datid                  | 12630
datname                | postgres
pid                    | 23723
usesysid               | 10
usename                | citus
application\_name      | psql
client\_addr           | 
client\_hostname       | 
client\_port           | -1
backend\_start         | 2018-10-05 13:27:14.419905+03
xact\_start            | 2018-10-05 13:27:16.362887+03
query\_start           | 2018-10-05 13:27:20.682452+03
state\_change          | 2018-10-05 13:27:20.896546+03
wait\_event_type       | Client
wait\_event            | ClientRead
state                  | idle in transaction
backend\_xid           | 
backend\_xmin          | 
query                  | SELECT count(*) FROM users_table;
backend\_type          | client backend
```

Cette requête nécessite des informations de toutes les partitions. Certaines informations se trouvent dans la partition `users_table_102038`, qui est stockée dans `localhost:9700`. Nous pouvons voir une requête qui accède à la partition en examinant l’affichage `citus_worker_stat_activity` :

```postgresql
SELECT * FROM citus_worker_stat_activity;

-[ RECORD 1 ]----------+-----------------------------------------------------------------------------------------
query_hostname         | localhost
query_hostport         | 9700
master_query_host_name | localhost
master_query_host_port | 9701
transaction_number     | 1
transaction_stamp      | 2018-10-05 13:27:20.691907+03
datid                  | 12630
datname                | postgres
pid                    | 23781
usesysid               | 10
usename                | citus
application\_name      | citus
client\_addr           | ::1
client\_hostname       | 
client\_port           | 51773
backend\_start         | 2018-10-05 13:27:20.75839+03
xact\_start            | 2018-10-05 13:27:20.84112+03
query\_start           | 2018-10-05 13:27:20.867446+03
state\_change          | 2018-10-05 13:27:20.869889+03
wait\_event_type       | Client
wait\_event            | ClientRead
state                  | idle in transaction
backend\_xid           | 
backend\_xmin          | 
query                  | COPY (SELECT count(*) AS count FROM users_table_102038 users_table WHERE true) TO STDOUT
backend\_type          | client backend
```

Le champ `query` affiche les données copiées en dehors de la partition à compter.

> [!NOTE]
> Si une requête de routeur (par exemple, un locataire unique dans une application multi-entité, `SELECT
> * FROM_table_WHERE_tenant_id = X’) est exécutée sans bloc de transaction, alors les colonnes de master\_query\_host\_name et master\_query\_host\_port seront NULL dans citus\_worker\_stat\_activity.

Pour voir comment `citus_lock_waits` fonctionne, nous pouvons générer une situation de verrouillage manuellement. Tout d\'abord, nous allons configurer une table de test à partir du coordinateur :

```postgresql
CREATE TABLE numbers AS
  SELECT i, 0 AS j FROM generate_series(1,10) AS i;
SELECT create_distributed_table('numbers', 'i');
```

Ensuite, à l’aide de deux sessions sur le coordinateur, nous pouvons exécuter cette séquence d’instructions :

```postgresql
-- session 1                           -- session 2
-------------------------------------  -------------------------------------
BEGIN;
UPDATE numbers SET j = 2 WHERE i = 1;
                                       BEGIN;
                                       UPDATE numbers SET j = 3 WHERE i = 1;
                                       -- (this blocks)
```

L’affichage `citus_lock_waits` montre la situation.

```postgresql
SELECT * FROM citus_lock_waits;

-[ RECORD 1 ]-------------------------+----------------------------------------
waiting_pid                           | 88624
blocking_pid                          | 88615
blocked_statement                     | UPDATE numbers SET j = 3 WHERE i = 1;
current_statement_in_blocking_process | UPDATE numbers SET j = 2 WHERE i = 1;
waiting_node_id                       | 0
blocking_node_id                      | 0
waiting_node_name                     | coordinator_host
blocking_node_name                    | coordinator_host
waiting_node_port                     | 5432
blocking_node_port                    | 5432
```

Dans cet exemple, les requêtes proviennent du coordinateur, mais l’affichage peut également répertorier les verrous entre les requêtes provenant de Worker (exécutées avec Hyperscale (Citus) MX par exemple).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment certaines [fonctions Hyperscale (Citus)](reference-hyperscale-functions.md) modifient les tables système
* Passez en revue les concepts de [nœuds et de tables](concepts-hyperscale-nodes.md)
