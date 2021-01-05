---
title: Fonctions SQL – Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Fonctions de l’API SQL d’Hyperscale (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: f324ef44d002f50bf27c08072e904c1d92b5512f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026231"
---
# <a name="functions-in-the-hyperscale-citus-sql-api"></a>Fonctions de l’API SQL d’Hyperscale (Citus)

Cette section contient des informations de référence pour les fonctions définies par l’utilisateur fournies par Hyperscale (Citus). Ces fonctions permettent de fournir des fonctionnalités distribuées supplémentaires à Hyperscale (Citus) autres que les commandes SQL standard.

> [!NOTE]
>
> Les groupes de serveurs Hyperscale (Citus) exécutant des versions antérieures de Citus Engine peuvent ne pas proposer toutes les fonctions listées ci-dessous.

## <a name="table-and-shard-ddl"></a>Table et DDL de partition

### <a name="create_distributed_table"></a>create\_distributed\_table

La fonction create\_distributed\_table() permet de définir une table distribuée et de créer ses partitions s’il s’agit d’une table distribuée par hachage. Cette fonction accepte un nom de table, la colonne de distribution et une méthode de distribution facultative, et insère les métadonnées appropriées pour marquer la table comme distribuée. La fonction prend par défaut la valeur de distribution « hash » si aucune méthode de distribution n’est spécifiée. Si la table est distribuée par hachage, la fonction crée également des partitions de travail en fonction des valeurs de configuration du nombre de partitions et du facteur de réplication de partition. Si la table contient des lignes, elles sont automatiquement distribuées aux nœuds Worker.

Cette fonction remplace l’utilisation de master\_create\_distributed\_table(), suivie de master\_create\_worker\_shards().

#### <a name="arguments"></a>Arguments

**table\_name:** Nom de la table qui doit être distribuée.

**distribution\_column:** Colonne sur laquelle la table doit être distribuée.

**distribution\_type:** (Facultatif) Méthode selon laquelle la table doit être distribuée. Les valeurs autorisées sont append ou hash, avec « hash » comme valeur par défaut.

**colocate\_with:** (Facultatif) Inclure la table actuelle dans le groupe de colocation d’une autre table. Par défaut, les tables sont colocalisées lorsqu’elles sont distribuées par colonnes du même type, ont le même nombre de partitions et ont le même facteur de réplication. Les valeurs possibles pour `colocate_with` sont `default`, `none` pour démarrer un nouveau groupe de colocation, ou le nom d’une autre table à mettre en colocation avec cette table.  (See [colocation de table](concepts-hyperscale-colocation.md).)

Gardez à l’esprit que la valeur par défaut de `colocate_with` effectue une colocation implicite. [La colocation](concepts-hyperscale-colocation.md) peut être une bonne chose lorsque les tables sont liées ou jointes.  Toutefois, lorsque deux tables ne sont pas liées, mais qu’elles utilisent le même type de données pour leurs colonnes de distribution, les mettre en colocation par inadvertance peut réduire les performances lors du [rééquilibrage de partition](howto-hyperscale-scale-rebalance.md).  Les partitions de table sont déplacées ensemble inutilement en \"cascade.\"

Si une nouvelle table distribuée n’est pas associée à d’autres tables, il est préférable de spécifier `colocate_with => 'none'`.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

Cet exemple indique à la base de données que la table github\_events doit être distribuée par hachage sur la colonne repo\_id.

```postgresql
SELECT create_distributed_table('github_events', 'repo_id');

-- alternatively, to be more explicit:
SELECT create_distributed_table('github_events', 'repo_id',
                                colocate_with => 'github_repo');
```

### <a name="create_reference_table"></a>create\_reference\_table

La fonction create\_reference\_table() est utilisée pour définir une petite référence ou une table de dimension. Cette fonction accepte un nom de table et crée une table distribuée avec une seule partition, répliquée sur chaque nœud Worker.

#### <a name="arguments"></a>Arguments

**table\_name:** Nom de la petite dimension ou de la table de référence qui doit être distribuée.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

Cet exemple indique à la base de données que la table nation doit être définie comme table de référence.

```postgresql
SELECT create_reference_table('nation');
```

### <a name="upgrade_to_reference_table"></a>upgrade\_to\_reference\_table

La fonction upgrade\_to\_reference\_table() de prend une table distribuée existante qui a un nombre partition de un et la met à niveau pour qu’elle soit une table de référence reconnue. Après l’appel de cette fonction, la table est comme si elle avait été créée avec [create_reference_table](#create_reference_table).

#### <a name="arguments"></a>Arguments

**table\_name:** Nom de la table distribuée (avec partition Count = 1) qui sera distribué comme table de référence.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

Cet exemple indique à la base de données que la table nation doit être définie comme table de référence.

```postgresql
SELECT upgrade_to_reference_table('nation');
```

### <a name="mark_tables_colocated"></a>mark\_tables\_colocated

La fonction mark\_tables\_colocated() prend une table distribuée (la source), et une liste d’autres tables (les cibles), et place les cibles dans le même groupe de colocation que la source. Si la source n’est pas encore dans un groupe, cette fonction en crée une et lui affecte la source et les cibles.

La colocation des tables doit être effectuée au moment de la distribution de table via le paramètre `colocate_with` de [create_distributed_table](#create_distributed_table), mais `mark_tables_colocated` peut le prendre en charge ultérieurement si nécessaire.

#### <a name="arguments"></a>Arguments

**source\_table\_name:** Nom de la table distribuée dont les cibles seront affectées au groupe de colocation.

**target\_table\_names:** Le tableau de noms des tables cibles distribuées ne doit pas être vide. Ces tables distribuées doivent correspondre à la table source dans :

> -   distribution (méthode)
> -   type de colonne de distribution
> -   type de réplication
> -   nombre de partitions

Si aucun des éléments ci-dessus ne s’applique, Hyperscale (Citus) génèrera une erreur. Par exemple, la tentative de colocation des tables `apples` et `oranges` dont les types de colonne de distribution diffèrent selon les valeurs suivantes :

```
ERROR:  XX000: cannot colocate tables apples and oranges
DETAIL:  Distribution column types don't match for apples and oranges.
```

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

Cet exemple place `products` et `line_items` dans le même groupe de colocation que `stores`. L’exemple part du principe que ces tables sont toutes distribuées sur une colonne avec le type correspondant, probablement un \"store id.\"

```postgresql
SELECT mark_tables_colocated('stores', ARRAY['products', 'line_items']);
```

### <a name="create_distributed_function"></a>create\_distributed\_function

Propage une fonction du nœud coordinateur aux Workers et la marque pour une exécution distribuée. Lorsqu’une fonction distribuée est appelée sur le coordinateur, Hyperscale (Citus) utilise la valeur de l’\"argument de distribution\" pour choisir un nœud Worker afin d’exécuter la fonction. L’exécution de la fonction sur les Workers augmente le parallélisme et peut rendre le code plus proche des données dans les partitions pour une latence plus faible.

Le chemin de recherche Postgres n’étant pas propagé du coordinateur aux threads de travail pendant l’exécution des fonctions distribuées, le code de la fonction distribuée doit qualifier entièrement les noms des objets de base de données. En outre, les notifications émises par les fonctions ne sont pas affichées à l’utilisateur.

#### <a name="arguments"></a>Arguments

**function\_name:** Nom de la fonction à distribuer. Le nom doit inclure les types de paramètres de la fonction entre parenthèses, car plusieurs fonctions peuvent avoir le même nom dans PostgreSQL. Par exemple, `'foo(int)'` est différent de `'foo(int, text)'`.

**distribution\_arg\_name:** (Facultatif) Nom de l’argument par lequel distribuer. Pour plus de commodité (ou si les arguments de fonction n’ont pas de nom), un espace réservé positionnel est autorisé, tel que `'$1'`. Si ce paramètre n’est pas spécifié, la fonction nommée par `function_name` est simplement créée sur les workers. Si les nœuds Worker sont ajoutés à l’avenir, la fonction sera automatiquement créée.

**colocate\_with:** (Facultatif) Lorsque la fonction distribuée lit ou écrit dans une table distribuée (ou, plus généralement, un groupe de colocation), veillez à nommer cette table à l’aide du paramètre `colocate_with` . Ensuite, chaque appel de la fonction s’exécute sur le nœud Worker contenant les partitions pertinentes.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

```postgresql
-- an example function which updates a hypothetical
-- event_responses table which itself is distributed by event_id
CREATE OR REPLACE FUNCTION
  register_for_event(p_event_id int, p_user_id int)
RETURNS void LANGUAGE plpgsql AS $fn$
BEGIN
  INSERT INTO event_responses VALUES ($1, $2, 'yes')
  ON CONFLICT (event_id, user_id)
  DO UPDATE SET response = EXCLUDED.response;
END;
$fn$;

-- distribute the function to workers, using the p_event_id argument
-- to determine which shard each invocation affects, and explicitly
-- colocating with event_responses which the function updates
SELECT create_distributed_function(
  'register_for_event(int, int)', 'p_event_id',
  colocate_with := 'event_responses'
);
```

## <a name="metadata--configuration-information"></a>Métadonnées/Informations de configuration

### <a name="master_get_table_metadata"></a>master\_get\_table\_metadata

La fonction master\_get\_table\_metadata() peut être utilisée pour retourner des métadonnées relatives à la distribution pour une table distribuée. Ces métadonnées incluent l’ID de relation, le type de stockage, la méthode de distribution, la colonne de distribution, le nombre de réplications, la taille de partition maximale et la stratégie de sélection élective de partition pour la table. En coulisses, cette fonction interroge les tables de métadonnées Hyperscale (Citus) pour obtenir les informations nécessaires et les concaténer dans un tuple avant de les renvoyer à l’utilisateur.

#### <a name="arguments"></a>Arguments

**table\_name:** Nom de la table distribuée pour laquelle vous souhaitez récupérer les métadonnées.

#### <a name="return-value"></a>Valeur de retour

Un tuple contenant les informations suivantes :

**logical\_relid:** Oid de la table distribuée. Elle fait référence à la colonne relfilenode de la table du catalogue système pg\_class.

**part\_storage\_type:** Type de stockage utilisé pour la table Peut être « t » (table standard), « f » (table étrangère) ou « c » (table en colonnes).

**part\_method:** Méthode de distribution utilisée pour la table. Peut être « a » (append) ou « h » (hash).

**part\_key:** Colonne de distribution pour la table.

**part\_replica\_count:** Nombre actuel de réplications de partition.

**part\_max\_size:** Taille maximale actuelle de partition en octets.

**part\_placement\_policy:** Stratégie de positionnement de partition utilisée pour placer les partitions de la table. Peut être 1 (local-node-first) ou 2 (round-robin).

#### <a name="example"></a>Exemple

L’exemple ci-dessous récupère et affiche les métadonnées de la table github\_events.

```postgresql
SELECT * from master_get_table_metadata('github_events');
 logical_relid | part_storage_type | part_method | part_key | part_replica_count | part_max_size | part_placement_policy 
---------------+-------------------+-------------+----------+--------------------+---------------+-----------------------
         24180 | t                 | h           | repo_id  |                  2 |    1073741824 |                     2
(1 row)
```

### <a name="get_shard_id_for_distribution_column"></a>get\_shard\_id\_for\_distribution\_column

Hyperscale (Citus) attribue chaque ligne d’une table distribuée à une partition en fonction de la valeur de la colonne de distribution de la ligne et de la méthode de distribution de la table. Dans la plupart des cas, le mappage précis est un détail de bas niveau que l’administrateur de base de données peut ignorer. Toutefois, il peut être utile de déterminer la partition d’une ligne, que ce soit pour des tâches de maintenance de base de données manuelles ou simplement par curiosite. La fonction `get_shard_id_for_distribution_column` fournit ces informations pour les tables de hachage distribuées et par plage, ainsi que les tables de référence. Elle ne fonctionne pas pour la distribution append.

#### <a name="arguments"></a>Arguments

**table\_name:** La table distribuée.

**distribution\_value:** Valeur de la colonne de distribution.

#### <a name="return-value"></a>Valeur de retour

L’ID de partition Hyperscale (Citus) s’associe à la valeur de la colonne de distribution pour la table donnée.

#### <a name="example"></a>Exemple

```postgresql
SELECT get_shard_id_for_distribution_column('my_table', 4);

 get_shard_id_for_distribution_column
--------------------------------------
                               540007
(1 row)
```

### <a name="column_to_column_name"></a>column\_to\_column\_name

Convertit la colonne `partkey` de `pg_dist_partition` en nom de colonne textuel. La conversion est utile pour déterminer la colonne de distribution d’une table distribuée.

Pour plus d’informations, consultez [choix d’une colonne de distribution](concepts-hyperscale-choose-distribution-column.md).

#### <a name="arguments"></a>Arguments

**table\_name:** La table distribuée.

**column\_var\_text:** Valeur de `partkey` dans la table `pg_dist_partition`.

#### <a name="return-value"></a>Valeur de retour

Nom de la colonne de distribution de `table_name`.

#### <a name="example"></a>Exemple

```postgresql
-- get distribution column name for products table

SELECT column_to_column_name(logicalrelid, partkey) AS dist_col_name
  FROM pg_dist_partition
 WHERE logicalrelid='products'::regclass;
```

Sortie :

```
┌───────────────┐
│ dist_col_name │
├───────────────┤
│ company_id    │
└───────────────┘
```

### <a name="citus_relation_size"></a>citus\_relation\_size

Obtenez l’espace disque utilisé par toutes les partitions de la table distribuée spécifiée.
L’espace disque inclut la taille de la \"branche principale,\" mais exclut la carte de visibilité et le mappage d’espace libre pour les partitions.

#### <a name="arguments"></a>Arguments

**logicalrelid :** le nom d’une table distribuée.

#### <a name="return-value"></a>Valeur de retour

Taille en octets comme bigint.

#### <a name="example"></a>Exemple

```postgresql
SELECT pg_size_pretty(citus_relation_size('github_events'));
```

```
pg_size_pretty
--------------
23 MB
```

### <a name="citus_table_size"></a>citus\_table\_size

Obtenez l’espace disque utilisé par toutes les partitions de la table distribuée spécifiée, à l’exclusion des index (mais TOAST, carte d’espace libre et carte de visibilité compris).

#### <a name="arguments"></a>Arguments

**logicalrelid :** le nom d’une table distribuée.

#### <a name="return-value"></a>Valeur de retour

Taille en octets comme bigint.

#### <a name="example"></a>Exemple

```postgresql
SELECT pg_size_pretty(citus_table_size('github_events'));
```

```
pg_size_pretty
--------------
37 MB
```

### <a name="citus_total_relation_size"></a>citus\_total\_relation\_size

Obtenez l’espace disque total utilisé par toutes les partitions de la table distribuée spécifiée, y compris tous les index et données TOAST.

#### <a name="arguments"></a>Arguments

**logicalrelid :** le nom d’une table distribuée.

#### <a name="return-value"></a>Valeur de retour

Taille en octets comme bigint.

#### <a name="example"></a>Exemple

```postgresql
SELECT pg_size_pretty(citus_total_relation_size('github_events'));
```

```
pg_size_pretty
--------------
73 MB
```

### <a name="citus_stat_statements_reset"></a>citus\_stat\_statements\_reset

Supprime toutes les lignes de [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table).
Cette fonction fonctionne indépendamment de `pg_stat_statements_reset()`. Pour réinitialiser toutes les statistiques, appelez les deux fonctions.

#### <a name="arguments"></a>Arguments

N/A

#### <a name="return-value"></a>Valeur de retour

None

## <a name="server-group-management-and-repair"></a>Gestion et réparation des groupes de serveurs

### <a name="master_copy_shard_placement"></a>master\_copy\_shard\_placement

Si la sélection élective d’une partition ne peut pas être mise à jour au cours d’une commande de modification ou d’une opération DDL, il est marqué comme étant inactif. La fonction master\_copy\_shard\_placement peut ensuite être appelée pour réparer une sélection élective de partition inactive à l’aide de données provenant d’une sélection élective saine.

Pour réparer une partition, la fonction supprime d’abord la sélection élective de partition défectueuse et la crée de nouveau à l’aide du schéma sur le coordinateur. Une fois la sélection élective de partition créée, la fonction copie les données à partir de l’emplacement sain et met à jour les métadonnées pour marquer la nouvelle sélection élective de partition comme saine. Cette fonction garantit que la partition est protégée contre toute modification simultanée pendant la réparation.

#### <a name="arguments"></a>Arguments

**shard\_id :** ID de la partition à réparer.

**source\_node\_name :** Nom DNS du nœud sur lequel la sélection élective de partition saine est présente (nœud \"source\" ).

**source\_node\_port :** Port sur le nœud worker source sur lequel le serveur de base de données est à l’écoute.

**target\_node\_name :** Nom DNS du nœud sur lequel la sélection élective de partition défectueuse est présente (nœud \"source\").

**target\_node\_port :** Port sur le nœud worker cible sur lequel le serveur de base de données est à l’écoute.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

L’exemple ci-dessous réparera une sélection élective de partition inactive de la partition 12345, qui est présent sur le serveur de base de données en cours d’exécution sur « bad\_host » sur le port 5432. Pour le réparer, il utilise les données d’un emplacement partition sain présent sur le serveur en cours d’exécution sur « good\_host » sur le port
5432.

```postgresql
SELECT master_copy_shard_placement(12345, 'good_host', 5432, 'bad_host', 5432);
```

### <a name="master_move_shard_placement"></a>master\_move\_shard\_placement

Cette fonction déplace une partition donnée (et les partitions en colocation avec) d’un nœud à un autre. Il est généralement utilisé indirectement pendant le rééquilibrage de partition plutôt que directement appelé par un administrateur de base de données.

Il existe deux façons de déplacer les données : le blocage ou le non-blocage. L’approche de blocage signifie qu’au cours du déplacement, toutes les modifications apportées aux partitions sont suspendues.
La seconde méthode, qui évite le blocage des écritures de partition, s’appuie sur la réplication logique Postgre 10.

Après une opération de déplacement réussie, les partitions dans le nœud source sont supprimées. Si le déplacement échoue, cette fonction génère une erreur et laisse les nœuds source et cible inchangés.

#### <a name="arguments"></a>Arguments

**shard\_id :** ID de la partition à déplacer.

**source\_node\_name :** Nom DNS du nœud sur lequel la sélection élective de partition saine est présente (nœud \"source\" ).

**source\_node\_port :** Port sur le nœud worker source sur lequel le serveur de base de données est à l’écoute.

**target\_node\_name :** Nom DNS du nœud sur lequel la sélection élective de partition défectueuse est présente (nœud \"source\").

**target\_node\_port :** Port sur le nœud worker cible sur lequel le serveur de base de données est à l’écoute.

**shard\_transfer\_mode :** (Facultatif) Spécifiez la méthode de réplication, si vous souhaitez utiliser la réplication logique PostgreSQL ou une commande COPY inter-worker. Les valeurs possibles sont les suivantes :

> -   `auto`: Exigez une identité de réplica si la réplication logique est possible, sinon utilisez le comportement hérité (par exemple, pour la réparation partition, PostgreSQL 9,6). Il s’agit de la valeur par défaut.
> -   `force_logical`: Utilisez la réplication logique même si la table n’a pas d’identité de réplica. Toute commande update/delete simultanée vers la table échouera lors de la réplication.
> -   `block_writes`: Utilisez COPY (blocking writes) pour les tables dépourvues d’une clé primaire ou d’une identité de réplica.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

```postgresql
SELECT master_move_shard_placement(12345, 'from_host', 5432, 'to_host', 5432);
```

### <a name="rebalance_table_shards"></a>rebalance\_table\_shards

La fonction rebalance\_table\_shards() déplace les partitions de la table donnée pour les distribuer uniformément entre les Workers. La fonction calcule d’abord la liste des déplacements qu’elle doit effectuer pour s’assurer que le groupe de serveurs est équilibré dans le seuil donné. Ensuite, il déplace les emplacements partition un par un du nœud source vers le nœud de destination et met à jour les métadonnées de partition correspondantes pour refléter le déplacement.

Un coût est attribué à chaque partition pour déterminer si les partitions sont \"réparties uniformément.\" Par défaut, chaque partition a le même coût (une valeur de 1). par conséquent, la distribution pour égaliser le coût entre les workers est identique à l’égalisation du nombre de partitions sur chaque worker. La stratégie de coût constant est appelée \"by\_shard\_count\" et est la stratégie de rééquilibrage par défaut.

La stratégie par défaut est appropriée dans les circonstances suivantes :

*  Les partitions sont à peu près de la même taille
*  Le partitions obtiennent à peu près la même quantité de trafic
*  Les nœuds worker sont de la même taille/type
*  Les partitions n’ont pas été épinglées à des workers particuliers

Si l’une de ces hypothèses ne tient pas, le rééquilibrage par défaut peut entraîner un mauvais plan. Dans ce cas, vous pouvez personnaliser la stratégie à l’aide du paramètre `rebalance_strategy` .

Il est recommandé d’appeler [get_rebalance_table_shards_plan](#get_rebalance_table_shards_plan) avant d’exécuter rebalance\_table\_shards, pour afficher et vérifier les actions à effectuer.

#### <a name="arguments"></a>Arguments

**table\_name:** (Facultatif) Nom de la table dont les partitions doivent être rééquilibrées. Si la valeur est NULL, rééquilibrer tous les groupes de colocation existants.

**threshold :** (Facultatif) Nombre à virgule flottante compris entre 0,0 et 1,0 qui indique le taux de différence maximal de l’utilisation des nœuds par rapport à l’utilisation moyenne. Par exemple, si vous spécifiez 0,1, le rééquilibreur de partition tentera d’équilibrer tous les nœuds pour qu’ils contiennent le même nombre de partitions ± 10%.
Plus précisément, le rééquilibreur de partition tente de converger l’utilisation de tous les nœuds worker vers le (1 - threshold) \* average\_utilization \..... (1
+ threshold) \* average\_utilization range.

**max\_shard\_moves :** (Facultatif) Nombre maximal de partitions à déplacer.

**excluded\_shard\_list :** (Facultatif) Identificateurs de partitions qui ne doivent pas être déplacées au cours de l’opération de rééquilibrage.

**shard\_transfer\_mode :** (Facultatif) Spécifiez la méthode de réplication, si vous souhaitez utiliser la réplication logique PostgreSQL ou une commande COPY inter-worker. Les valeurs possibles sont les suivantes :

> -   `auto`: Exigez une identité de réplica si la réplication logique est possible, sinon utilisez le comportement hérité (par exemple, pour la réparation partition, PostgreSQL 9,6). Il s’agit de la valeur par défaut.
> -   `force_logical`: Utilisez la réplication logique même si la table n’a pas d’identité de réplica. Toute commande update/delete simultanée vers la table échouera lors de la réplication.
> -   `block_writes`: Utilisez COPY (blocking writes) pour les tables dépourvues d’une clé primaire ou d’une identité de réplica.

**drain\_only :** (Facultatif) Si la valeur est true, les déplacez partitions hors des nœuds Worker qui ont `shouldhaveshards` défini sur false dans [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table); ne déplacer aucune autre partition.

**rebalance\_strategy :** (Facultatif) le nom d’une stratégie dans [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md#rebalancer-strategy-table).
Si cet argument est omis, la fonction choisit la stratégie par défaut, comme indiqué dans la table.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

L’exemple ci-dessous tente de rééquilibrer les partitions de la table github\_events dans le seuil par défaut.

```postgresql
SELECT rebalance_table_shards('github_events');
```

Cet exemple d’utilisation va tenter de rééquilibrer la table github\_events sans déplacer de partitions avec les ID 1 et 2.

```postgresql
SELECT rebalance_table_shards('github_events', excluded_shard_list:='{1,2}');
```

### <a name="get_rebalance_table_shards_plan"></a>get\_rebalance\_table\_shards\_plan

Générez les mouvements partition planifiés de [rebalance_table_shards](#rebalance_table_shards) sans les exécuter.
Bien que cela soit peu probable, get\_rebalance\_table\_shards\_plan peut générer un plan légèrement différent de celui d’un appel de rebalance\_table\_shards avec les mêmes arguments. Ils ne sont pas exécutés en même temps. par conséquent, les faits sur le groupe de serveurs \-, par exemple, l’espace disque \-, peuvent varier entre les appels.

#### <a name="arguments"></a>Arguments

Les mêmes arguments que rebalance\_table\_shards : relation, threshold, max\_shard\_moves, excluded\_shard\_list, et drain\_only. Consultez la documentation de cette fonction pour la signification des arguments.

#### <a name="return-value"></a>Valeur de retour

Tuples contenant les colonnes suivantes :

-   **table\_name** : Table dont les partitions se déplaceraient
-   **shardid** : La partition en question
-   **shard\_size** : Taille en octets
-   **sourcename** : Nom d’hôte du nœud source
-   **sourceport** : Port du nœud source
-   **targetname** : Nom d’hôte du nœud de destination
-   **targetport** : Port du nœud de destination

### <a name="get_rebalance_progress"></a>get\_rebalance\_progress

Une fois qu’un rééquilibrage de partition commence, la fonction `get_rebalance_progress()` répertorie la progression de chaque partition impliquée. Elle surveille les déplacements planifiés et exécutés par `rebalance_table_shards()`.

#### <a name="arguments"></a>Arguments

N/A

#### <a name="return-value"></a>Valeur de retour

Tuples contenant les colonnes suivantes :

-   **sessionid** : PID Postgres de l’analyse de rééquilibrage
-   **table\_name** : Table dont les partitions se déplacent
-   **shardid** : La partition en question
-   **shard\_size** : Taille en octets
-   **sourcename** : Nom d’hôte du nœud source
-   **sourceport** : Port du nœud source
-   **targetname** : Nom d’hôte du nœud de destination
-   **targetport** : Port du nœud de destination
-   **progress** : 0 = en attente de déplacement ; 1 = en déplacement ; 2 = terminé

#### <a name="example"></a>Exemple

```sql
SELECT * FROM get_rebalance_progress();
```

```
┌───────────┬────────────┬─────────┬────────────┬───────────────┬────────────┬───────────────┬────────────┬──────────┐
│ sessionid │ table_name │ shardid │ shard_size │  sourcename   │ sourceport │  targetname   │ targetport │ progress │
├───────────┼────────────┼─────────┼────────────┼───────────────┼────────────┼───────────────┼────────────┼──────────┤
│      7083 │ foo        │  102008 │    1204224 │ n1.foobar.com │       5432 │ n4.foobar.com │       5432 │        0 │
│      7083 │ foo        │  102009 │    1802240 │ n1.foobar.com │       5432 │ n4.foobar.com │       5432 │        0 │
│      7083 │ foo        │  102018 │     614400 │ n2.foobar.com │       5432 │ n4.foobar.com │       5432 │        1 │
│      7083 │ foo        │  102019 │       8192 │ n3.foobar.com │       5432 │ n4.foobar.com │       5432 │        2 │
└───────────┴────────────┴─────────┴────────────┴───────────────┴────────────┴───────────────┴────────────┴──────────┘
```

### <a name="citus_add_rebalance_strategy"></a>citus\_add\_rebalance\_strategy

Ajoutez une ligne à [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md?#rebalancer-strategy-table).

#### <a name="arguments"></a>Arguments

Pour plus d’informations sur ces arguments, consultez les valeurs de colonnes correspondantes dans `pg_dist_rebalance_strategy`.

**name :** identificateur pour la nouvelle stratégie

**shared\_cost\_function :** identifie la fonction utilisée pour déterminer le \"coût\" de chaque partition

**node\_capacity\_function :** identifie la fonction pour mesurer la capacité du nœud

**shard\_allowed\_on\_node\_function :** identifie la fonction qui détermine quelles partitions peuvent être placées sur quels nœuds

**default\_threshold :** un seuil à virgule flottante qui ajuste la précision avec laquelle le coût de partition cumulé doit être équilibré entre les nœuds

**minimum\_threshold :** (Facultatif) colonne de protection qui contient la valeur minimale autorisée pour l’argument de seuil de rebalance\_table\_shards(). Sa valeur par défaut est 0

#### <a name="return-value"></a>Valeur de retour

N/A

### <a name="citus_set_default_rebalance_strategy"></a>citus\_set\_default\_rebalance\_strategy

Mettez à jour la table [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md#rebalancer-strategy-table) en modifiant la stratégie nommée par son argument comme étant la valeur par défaut choisie lors du rééquilibrage de partitions.

#### <a name="arguments"></a>Arguments

**name :** le nom de la stratégie dans pg\_dist\_rebalance\_strategy

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

```postgresql
SELECT citus_set_default_rebalance_strategy('by_disk_size');
```

### <a name="citus_remote_connection_stats"></a>citus\_remote\_connection\_stats

La fonction citus\_remote\_connection\_stats() affiche le nombre de connexions actives à chaque nœud distant.

#### <a name="arguments"></a>Arguments

N/A

#### <a name="example"></a>Exemple

```postgresql
SELECT * from citus_remote_connection_stats();
```

```
    hostname    | port | database_name | connection_count_to_node
----------------+------+---------------+--------------------------
 citus_worker_1 | 5432 | postgres      |                        3
(1 row)
```

### <a name="master_drain_node"></a>master\_drain\_node

La fonction master\_drain\_node() déplace les partitions hors du nœud désigné et sur les autres nœuds qui ont `shouldhaveshards` défini sur true dans [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table). Appelez la fonction avant de supprimer un nœud du groupe de serveurs et de désactiver le serveur physique du nœud.

#### <a name="arguments"></a>Arguments

**nodename :** Nom d’hôte du nœud à drainer.

**nodeport :** Numéro de port du nœud à drainer.

**shard\_transfer\_mode :** (Facultatif) Spécifiez la méthode de réplication, si vous souhaitez utiliser la réplication logique PostgreSQL ou une commande COPY inter-worker. Les valeurs possibles sont les suivantes :

> -   `auto`: Exigez une identité de réplica si la réplication logique est possible, sinon utilisez le comportement hérité (par exemple, pour la réparation partition, PostgreSQL 9,6). Il s’agit de la valeur par défaut.
> -   `force_logical`: Utilisez la réplication logique même si la table n’a pas d’identité de réplica. Toute commande update/delete simultanée vers la table échouera lors de la réplication.
> -   `block_writes`: Utilisez COPY (blocking writes) pour les tables dépourvues d’une clé primaire ou d’une identité de réplica.

**rebalance\_strategy :** (Facultatif) le nom d’une stratégie dans [pg_dist_rebalance_strategy](reference-hyperscale-metadata.md#rebalancer-strategy-table).
Si cet argument est omis, la fonction choisit la stratégie par défaut, comme indiqué dans la table.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="example"></a>Exemple

Voici les étapes habituelles pour supprimer un seul nœud (par exemple, « 10.0.0.1 » sur un port PostgreSQL standard) :

1.  Drainez le nœud.

    ```postgresql
    SELECT * from master_drain_node('10.0.0.1', 5432);
    ```

2.  Patientez jusqu’à la fin de la commande

3.  Supprimez le nœud

Lors du drain de plusieurs nœuds, il est recommandé d’utiliser [rebalance_table_shards](#rebalance_table_shards) à la place. Cela permet à Hyperscale (Citus) de planifier à l’avance et de déplacer des partitions le nombre de fois minimum.

1.  Exécutez pour chaque nœud que vous souhaitez supprimer :

    ```postgresql
    SELECT * FROM master_set_node_property(node_hostname, node_port, 'shouldhaveshards', false);
    ```

2.  Drainez-les tous en même temps avec [rebalance_table_shards](#rebalance_table_shards)

    ```postgresql
    SELECT * FROM rebalance_table_shards(drain_only := true);
    ```

3.  Attendez la fin de l’équilibrage du drainage

4.  Supprimez les nœuds

### <a name="replicate_table_shards"></a>replicate\_table\_shards

La fonction replicate\_table\_shards() réplique les partitions sous-répliquées de la table donnée. La fonction calcule d’abord la liste des partitions sous-répliquées et les emplacements à partir desquels ils peuvent être extraits pour la réplication. La fonction copie ensuite sur ces partitions et met à jour les métadonnées de partition correspondantes pour refléter la copie.

#### <a name="arguments"></a>Arguments

**table\_name :** Nom de la table dont les partitions doivent être répliquées.

**shard\_replication\_factor :** (Facultatif) Facteur de réplication souhaité à atteindre pour chaque partition.

**max\_shard\_copies:** (Facultatif) Nombre maximum de partitions à copier pour atteindre le facteur de réplication souhaité.

**excluded\_shard\_list :** (Facultatif) Identificateurs de partitions qui ne doivent pas être copiées au cours de l’opération de réplication.

#### <a name="return-value"></a>Valeur de retour

N/A

#### <a name="examples"></a>Exemples

L’exemple ci-dessous tente de rééquilibrer les partitions de la table github\_events table to shard\_replication\_factor.

```postgresql
SELECT replicate_table_shards('github_events');
```

Cet exemple tente de ramener les partitions de la table github\_events au facteur de réplication souhaité, avec un maximum de 10 copies de partition. Le rééquilibreur copiera un maximum de 10 partitions dans sa tentative d’atteindre le facteur de réplication souhaité.

```postgresql
SELECT replicate_table_shards('github_events', max_shard_copies:=10);
```

### <a name="isolate_tenant_to_new_shard"></a>isolate\_tenant\_to\_new\_shard

Cette fonction crée une nouvelle partition pour stocker des lignes avec une valeur unique spécifique dans la colonne de distribution. Elles est particulièrement pratique pour le cas d’utilisation d’Hyperscale (Citus) multi-entité, où un grand locataire peut être placé seul sur sa propre partition et finalement son propre nœud physique.

#### <a name="arguments"></a>Arguments

**table\_name :** Nom de la table pour laquelle obtenir une nouvelle partition.

**tenant\_id :** Valeur de la colonne de distribution qui sera affectée à la nouvelle partition.

**cascade\_option :** (Facultatif) Lorsqu’il est défini sur \"CASCADE,\" isole également une partition de toutes les tables dans le [groupe de colocation](concepts-hyperscale-colocation.md) de la table active.

#### <a name="return-value"></a>Valeur de retour

**shard\_id :** La fonction retourne l’ID unique affecté à la partition nouvellement créée.

#### <a name="examples"></a>Exemples

Créez une nouvelle partition pour contenir les lineitems pour le locataire 135 :

```postgresql
SELECT isolate_tenant_to_new_shard('lineitem', 135);
```

```
┌─────────────────────────────┐
│ isolate_tenant_to_new_shard │
├─────────────────────────────┤
│                      102240 │
└─────────────────────────────┘
```

## <a name="next-steps"></a>Étapes suivantes

* La plupart des fonctions de cet article modifient les [tables de métadonnées](reference-hyperscale-metadata.md) système
* [Paramètres du serveur](reference-hyperscale-parameters.md) personnaliser le comportement de certaines fonctions
