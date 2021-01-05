---
title: Modifier des tables distribuées - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Commandes SQL à utiliser pour créer et modifier des tables distribuées - Hyperscale (Citus) à l'aide du portail Azure
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 8/10/2020
ms.openlocfilehash: cf9f9ca5b8690a38c6e5aa6f519378c0a2e3a4f2
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026435"
---
# <a name="distribute-and-modify-tables"></a>Distribuer et modifier des tables

## <a name="distributing-tables"></a>Distribution de tables

Pour créer une table distribuée, vous devez d'abord définir le schéma de table. Pour ce faire, vous pouvez définir une table à l'aide de l'instruction [CREATE TABLE](http://www.postgresql.org/docs/current/static/sql-createtable.html), comme avec une table PostgreSQL standard.

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    actor jsonb,
    org jsonb,
    created_at timestamp
);
```

Vous pouvez ensuite utiliser la fonction create\_distributed\_table() pour spécifier la colonne de distribution de table et créer les partitions du Worker.

```sql
SELECT create_distributed_table('github_events', 'repo_id');
```

L'appel de fonction informe Hyperscale (Citus) que la table github\_events doit être distribuée sur la colonne repo\_id (en hachant la valeur de la colonne). La fonction crée également des partitions sur les nœuds Worker à l'aide des valeurs de configuration citus.shard\_count et citus.shard\_replication\_factor.

Elle crée un nombre citus.shard\_count total de partitions, où chaque partition possède une partie d'un espace de hachage et est répliquée en fonction de la valeur de configuration citus.shard\_replication\_factor par défaut. Les réplicas de partition créés sur le Worker disposent du même schéma de table, du même index et des mêmes définitions de contraintes que la table figurant sur le coordinateur. Une fois les réplicas créés, la fonction enregistre toutes les métadonnées distribuées sur le coordinateur.

Chaque partition créée reçoit un ID de partition unique, et tous ses réplicas possèdent le même ID de partition. Les partitions sont représentées sur le nœud Worker sous forme de tables PostgreSQL classiques nommées \'tablename\_shardid\', sachant que tablename correspond au nom de la table distribuée et shardid à l'ID unique attribué. Vous pouvez vous connecter aux instances Postgres du Worker pour afficher ou exécuter des commandes sur des partitions individuelles.

Vous êtes maintenant prêt à insérer des données dans la table distribuée et à exécuter des requêtes sur celle-ci. Pour en savoir plus sur la fonction UDF utilisée dans cette section, consultez la référence [Table et DDL de partition](reference-hyperscale-functions.md#table-and-shard-ddl).

### <a name="reference-tables"></a>Tables de référence

La méthode ci-dessus distribue les tables en les répartissant sur plusieurs partitions horizontales.  Une autre possibilité consiste à distribuer les tables sur une seule partition et à répliquer la partition sur chaque nœud Worker. Les tables ainsi distribuées sont appelées des *tables de référence.* Elles sont utilisées pour stocker les données qui doivent être fréquemment consultées par plusieurs nœuds au sein d'un cluster.

Principales candidates aux tables de référence :

-   Petites tables qui doivent se joindre à des tables distribuées plus volumineuses
-   Tables d'applications multilocataires qui n'ont pas de colonne ID de locataire ou qui ne sont associées à aucun locataire (ou, lors de la migration, même pour certaines tables associées à un locataire)
-   Tables qui nécessitent des contraintes uniques sur plusieurs colonnes et qui sont suffisamment petites

Supposons par exemple qu'un site de commerce électronique multilocataire ait besoin de calculer la taxe de vente pour les transactions effectuées dans l'une de ses boutiques. Les informations fiscales ne sont pas spécifiques à un locataire. Il est donc logique de les placer dans une table partagée. Une table de référence spécifique aux États-Unis pourrait se présenter comme suit :

```postgresql
-- a reference table

CREATE TABLE states (
  code char(2) PRIMARY KEY,
  full_name text NOT NULL,
  general_sales_tax numeric(4,3)
);

-- distribute it to all workers

SELECT create_reference_table('states');
```

Désormais, les requêtes telles que le calcul de la taxe correspondant à un panier d'achat peuvent se joindre à la table `states` sans surcharge réseau, et peuvent ajouter une clé étrangère au code d'état pour une meilleure validation.

En plus de distribuer une table en tant que partition répliquée unique, la fonction UDF `create_reference_table` la marque comme table de référence dans les tables de métadonnées Hyperscale (Citus). Hyperscale (Citus) effectue automatiquement des validations en deux phases ([2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)) pour les modifications apportées aux tables ainsi marquées, ce qui offre de solides garanties de cohérence.

Si vous disposez d'une table distribuée qui ne comporte qu'une seule partition, vous pouvez la mettre à niveau pour la transformer en table de référence reconnue :

```postgresql
SELECT upgrade_to_reference_table('table_name');
```

Pour obtenir un autre exemple d'utilisation de tables de référence, consultez le [tutoriel consacré aux bases de données multilocataires](tutorial-design-database-hyperscale-multi-tenant.md).

### <a name="distributing-coordinator-data"></a>Distribution des données du coordinateur

Si une base de données PostgreSQL existante est convertie en nœud coordinateur pour un cluster Hyperscale (Citus), les données de ses tables peuvent être distribuées efficacement et avec une interruption minimale de l'application.

La fonction `create_distributed_table` décrite précédemment fonctionne aussi bien sur les tables vides que sur les tables non vides, et pour ces dernières, elle distribue automatiquement les lignes des tables dans l'ensemble du cluster. Vous saurez si elle copie des données grâce à la présence du message \"AVIS : Copie des données à partir de la table locale\...\" Par exemple :

```postgresql
CREATE TABLE series AS SELECT i FROM generate_series(1,1000000) i;
SELECT create_distributed_table('series', 'i');
NOTICE:  Copying data from local table...
 create_distributed_table
 --------------------------

 (1 row)
```

Les écritures sur la table sont bloquées pendant la migration des données, et les écritures en attente sont traitées comme des requêtes distribuées une fois la fonction validée. (Si la fonction échoue, les requêtes redeviennent locales). Les lectures peuvent se poursuivre normalement, et elles deviendront des requêtes distribuées une fois la fonction validée.

Lors de la distribution des tables A et B, où A dispose d'une clé étrangère vers B, distribuez d'abord la table de destination de la clé B. Si cet ordre n'est pas respecté, une erreur sera générée :

```
ERROR:  cannot create foreign key constraint
DETAIL:  Referenced table must be a distributed table or a reference table.
```

S'il n'est pas possible de procéder à la distribution dans le bon ordre, supprimez les clés étrangères, distribuez les tables et recréez les clés étrangères.

Lors d'une migration de données à partir d'une base de données externe, par exemple d'Amazon RDS vers Hyperscale (Citus) Cloud, créez d'abord les tables distribuées Hyperscale (Citus) via `create_distributed_table`, puis copiez les données dans la table.
La copie dans des tables distribuées évite de manquer d'espace sur le nœud coordinateur.

## <a name="colocating-tables"></a>Colocation de tables

La colocation consiste à placer des informations connexes sur les mêmes machines. Elle permet des requêtes efficaces, tout en tirant parti de la scalabilité horizontale pour l'ensemble du jeu de données. Pour plus d'informations, consultez [Colocation](concepts-hyperscale-colocation.md).

Les tables sont colocalisées dans des groupes. Pour contrôler manuellement l'attribution des groupes de colocation d'une table, utilisez le paramètre facultatif `colocate_with` de `create_distributed_table`. Si vous ne vous souciez pas de la colocation d'une table, omettez ce paramètre. Par défaut, sa valeur est `'default'`. Celle-ci regroupe la table avec toute autre table de colocation par défaut dotée du même type de colonne de distribution, du même nombre de partitions et du même facteur de réplication. Si vous souhaitez arrêter ou mettre à jour cette colocation implicite, vous pouvez utiliser `update_distributed_table_colocation()`.

```postgresql
-- these tables are implicitly co-located by using the same
-- distribution column type and shard count with the default
-- co-location group

SELECT create_distributed_table('A', 'some_int_col');
SELECT create_distributed_table('B', 'other_int_col');
```

Lorsqu'une nouvelle table n'est liée à aucune autre de son groupe de colocation implicite potentiel, spécifiez `colocated_with => 'none'`.

```postgresql
-- not co-located with other tables

SELECT create_distributed_table('A', 'foo', colocate_with => 'none');
```

Le fractionnement de tables non liées en groupes de colocation qui leur sont propres améliorera les performances de [rééquilibrage des partitions](howto-hyperscale-scale-rebalance.md), car les partitions d'un même groupe doivent être déplacées ensemble.

Lorsque des tables sont effectivement liées (par exemple, lorsqu'elles sont jointes), il peut être judicieux de procéder à une colocation explicite de celles-ci. Les gains d'une colocation appropriée sont plus importants que les frais de rééquilibrage.

Pour procéder à une colocation explicite de plusieurs tables, distribuez-en une, puis placez les autres dans son groupe de colocation. Par exemple :

```postgresql
-- distribute stores
SELECT create_distributed_table('stores', 'store_id');

-- add to the same group as stores
SELECT create_distributed_table('orders', 'store_id', colocate_with => 'stores');
SELECT create_distributed_table('products', 'store_id', colocate_with => 'stores');
```

Les informations relatives aux groupes de colocation sont stockées dans la table [pg_dist_colocation](reference-hyperscale-metadata.md#colocation-group-table), tandis que [pg_dist_partition](reference-hyperscale-metadata.md#partition-table) révèle quelles tables sont attribuées à quels groupes.

## <a name="dropping-tables"></a>Suppression de tables

Vous pouvez utiliser la commande standard de PostgreSQL DROP TABLE pour supprimer vos tables distribuées. Comme pour les tables classiques, DROP TABLE supprime l'ensemble des index, règles, déclencheurs et contraintes associés à la table cible. En outre, elle supprime également les partitions sur les nœuds Worker et nettoie leurs métadonnées.

```sql
DROP TABLE github_events;
```

## <a name="modifying-tables"></a>Modification de tables

Hyperscale (Citus) propage automatiquement de nombreux types d'instructions DDL.
La modification d'une table distribuée sur le nœud coordinateur mettra également à jour les partitions sur les Workers. D'autres instructions DDL nécessitent une propagation manuelle, et certaines sont interdites, comme celles capables de modifier une colonne de distribution.
Toute tentative d'exécution d'un DDL qui n'est pas éligible à la propagation automatique génère une erreur et laisse les tables du nœud coordinateur inchangées.

Voici une référence des catégories d'instructions DDL qui se propagent.
La propagation automatique peut être activée ou désactivée à l'aide d'un [paramètre de configuration](reference-hyperscale-parameters.md#citusenable_ddl_propagation-boolean).

### <a name="addingmodifying-columns"></a>Ajout/modification de colonnes

Hyperscale (CITUS) propage automatiquement la plupart des commandes [ALTER TABLE](https://www.postgresql.org/docs/current/static/ddl-alter.html). L'ajout de colonnes et la modification de leurs valeurs par défaut fonctionnent comme dans une base de données PostgreSQL à une seule machine :

```postgresql
-- Adding a column

ALTER TABLE products ADD COLUMN description text;

-- Changing default value

ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;
```

Les modifications importantes apportées à une colonne existante, comme le fait de la renommer ou de changer son type de données, ne posent aucun problème non plus. Toutefois, le type de données de la [colonne de distribution](concepts-hyperscale-nodes.md#distribution-column) ne peut pas être modifié.
Cette colonne détermine le mode de distribution des données de la table à travers le cluster Hyperscale (Citus), et la modification de son type de données nécessiterait un déplacement des données.

Toute tentative en ce sens génère une erreur :

```postgres
-- assumining store_id is the distribution column
-- for products, and that it has type integer

ALTER TABLE products
ALTER COLUMN store_id TYPE text;

/*
ERROR:  XX000: cannot execute ALTER TABLE command involving partition column
LOCATION:  ErrorIfUnsupportedAlterTableStmt, multi_utility.c:2150
*/
```

### <a name="addingremoving-constraints"></a>Ajout/suppression de contraintes

L'utilisation d'Hyperscale (Citus) vous permet de continuer à bénéficier de la sécurité d'une base de données relationnelle, contraintes comprises (voir la [documentation](https://www.postgresql.org/docs/current/static/ddl-constraints.html) PostgreSQL).
En raison de la nature des systèmes distribués, Hyperscale (Citus) n'établit pas de références croisées entre les nœuds Worker pour les contraintes d'unicité ou l'intégrité référentielle.

Pour configurer une clé étrangère entre des tables distribuées colocalisées, incluez toujours la colonne de distribution dans la clé. L'inclusion de la colonne de distribution peut impliquer la création de la clé composée.

Des clés étrangères peuvent être créées dans les cas suivants :

-   entre deux tables locales (non distribuées),
-   entre deux tables de référence,
-   entre deux tables distribuées [colocalisées](concepts-hyperscale-colocation.md) lorsque la clé contient la colonne de distribution, ou
-   en tant que table distribuée référençant une [table de référence](concepts-hyperscale-nodes.md#type-2-reference-tables).

Les clés étrangères entre les tables de référence et les tables distribuées ne sont pas prises en charge.

> [!NOTE]
>
> Les clés primaires et les contraintes d'unicité doivent inclure la colonne de distribution. Leur ajout à une colonne autre qu'une colonne de distribution générera une erreur.

Cet exemple montre comment créer des clés primaires et étrangères sur des tables distribuées :

```postgresql
--
-- Adding a primary key
-- --------------------

-- We'll distribute these tables on the account_id. The ads and clicks
-- tables must use compound keys that include account_id.

ALTER TABLE accounts ADD PRIMARY KEY (id);
ALTER TABLE ads ADD PRIMARY KEY (account_id, id);
ALTER TABLE clicks ADD PRIMARY KEY (account_id, id);

-- Next distribute the tables

SELECT create_distributed_table('accounts', 'id');
SELECT create_distributed_table('ads',      'account_id');
SELECT create_distributed_table('clicks',   'account_id');

--
-- Adding foreign keys
-- -------------------

-- Note that this can happen before or after distribution, as long as
-- there exists a uniqueness constraint on the target column(s) which
-- can only be enforced before distribution.

ALTER TABLE ads ADD CONSTRAINT ads_account_fk
  FOREIGN KEY (account_id) REFERENCES accounts (id);
ALTER TABLE clicks ADD CONSTRAINT clicks_ad_fk
  FOREIGN KEY (account_id, ad_id) REFERENCES ads (account_id, id);
```

De même, incluez la colonne de distribution dans les contraintes d'unicité :

```postgresql
-- Suppose we want every ad to use a unique image. Notice we can
-- enforce it only per account when we distribute by account id.

ALTER TABLE ads ADD CONSTRAINT ads_unique_image
  UNIQUE (account_id, image_url);
```

Des contraintes non nulles peuvent être appliquées à n'importe quelle colonne (de distribution ou autre) car elles ne nécessitent aucune recherche entre les Workers.

```postgresql
ALTER TABLE ads ALTER COLUMN image_url SET NOT NULL;
```

### <a name="using-not-valid-constraints"></a>Utilisation de contraintes NON VALID

Dans certains cas, il peut être utile d'appliquer des contraintes pour les nouvelles lignes, tout en permettant aux lignes non conformes existantes de rester inchangées. Hyperscale (Citus) prend en charge cette fonctionnalité pour les contraintes CHECK et les clés étrangères, en utilisant la désignation de contrainte \"NOT VALID\" de PostgreSQL.

Prenons par exemple une application qui stocke les profils utilisateur dans une [table de référence](concepts-hyperscale-nodes.md#type-2-reference-tables).

```postgres
-- we're using the "text" column type here, but a real application
-- might use "citext" which is available in a postgres contrib module

CREATE TABLE users ( email text PRIMARY KEY );
SELECT create_reference_table('users');
```

Au fil du temps, imaginez que quelques non-adresses se retrouvent dans la table.

```postgres
INSERT INTO users VALUES
   ('foo@example.com'), ('hacker12@aol.com'), ('lol');
```

Nous aimerions valider les adresses, mais PostgreSQL ne nous permet généralement pas d'ajouter une contrainte CHECK qui échoue pour les lignes existantes. Toutefois, il *​​autorise* une contrainte marquée comme non valide :

```postgres
ALTER TABLE users
ADD CONSTRAINT syntactic_email
CHECK (email ~
   '^[a-zA-Z0-9.!#$%&''*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$'
) NOT VALID;
```

Les nouvelles lignes sont maintenant protégées.

```postgres
INSERT INTO users VALUES ('fake');

/*
ERROR:  new row for relation "users_102010" violates
        check constraint "syntactic_email_102010"
DETAIL:  Failing row contains (fake).
*/
```

Plus tard, pendant les heures creuses, un administrateur de base de données peut tenter de corriger les lignes incorrectes et de revalider la contrainte.

```postgres
-- later, attempt to validate all rows
ALTER TABLE users
VALIDATE CONSTRAINT syntactic_email;
```

La documentation PostgreSQL contient plus d'informations sur NOT VALID et VALIDATE CONSTRAINT dans la section [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html).

### <a name="addingremoving-indices"></a>Ajout/suppression d'index

Hyperscale (Citus) prend en charge l'ajout et la suppression d'[index](https://www.postgresql.org/docs/current/static/sql-createindex.html) :

```postgresql
-- Adding an index

CREATE INDEX clicked_at_idx ON clicks USING BRIN (clicked_at);

-- Removing an index

DROP INDEX clicked_at_idx;
```

L'ajout d'un index nécessite un verrou d'écriture, ce qui peut être indésirable dans un \"système d'enregistrement\" multilocataire. Pour réduire le temps d'arrêt des applications, créez plutôt l'index [simultanément](https://www.postgresql.org/docs/current/static/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY). Cette méthode nécessite plus de travail qu'une génération d'index standard et prend plus de temps. Mais dans la mesure où elle permet de poursuivre les opérations normales pendant la génération de l'index, cette méthode s'avère utile pour ajouter de nouveaux index dans un environnement de production.

```postgresql
-- Adding an index without locking table writes

CREATE INDEX CONCURRENTLY clicked_at_idx ON clicks USING BRIN (clicked_at);
```
