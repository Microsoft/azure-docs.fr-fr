---
title: Paramètres de serveur – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Paramètres de l’API SQL d’Hyperscale (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: reference
ms.date: 08/10/2020
ms.openlocfilehash: 07f966c7b0be542f848f1a0a4eaf2b5549735b4b
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91336239"
---
# <a name="server-parameters"></a>Paramètres de serveur

Différents paramètres de serveur déterminent le comportement d'Hyperscale (Citus), qu'il s'agisse de paramètres PostgreSQL standard ou de paramètres spécifiques à Hyperscale (Citus).
Ces paramètres peuvent être définis sur le portail Azure pour un groupe de serveurs Hyperscale (Citus). Dans la catégorie **Paramètres**, choisissez **Paramètres du nœud Worker** ou **Paramètres du nœud coordinateur**. Ces pages vous permettent de définir des paramètres pour tous les nœuds Worker, ou uniquement pour le nœud coordinateur.

## <a name="hyperscale-citus-parameters"></a>Paramètres Hyperscale (Citus)

> [!NOTE]
>
> Les groupes de serveurs Hyperscale (Citus) exécutant des versions antérieures de Citus Engine peuvent ne pas proposer tous les paramètres listés ci-dessous.

### <a name="general-configuration"></a>Configuration générale

#### <a name="citususe_secondary_nodes-enum"></a>citus.use\_secondary\_nodes (enum)

Définit la stratégie à utiliser lors du choix des nœuds pour les requêtes SELECT. S’il est défini sur « always », le planificateur interroge uniquement les nœuds marqués comme noderole « secondaires » dans [pg_dist_node](reference-hyperscale-metadata.md#worker-node-table).

Les valeurs prises en charge pour cette énumération sont les suivantes :

-   **never :** (par défaut) Toutes les lectures se produisent sur les nœuds principaux.
-   **always :** Les lectures sont exécutées sur les nœuds secondaires à la place et les commandes insert/update sont désactivées.

#### <a name="cituscluster_name-text"></a>citus.cluster\_name (texte)

Informe le responsable du nœud coordinateur du cluster qu’il coordonne. Une fois le cluster\_name défini, le planificateur interroge uniquement les nœuds worker dans ce cluster.

#### <a name="citusenable_version_checks-boolean"></a>citus.enable\_version\_checks (booléen)

Mettre à niveau la version d’Hyperscale (Citus) nécessite un redémarrage serveur (pour récupérer la nouvelle bibliothèque partagée), suivie de la commande ALTER EXTENSION UPDATE.  L’échec de l’exécution des deux étapes peut entraîner des erreurs ou des blocages.
Hyperscale (Citus) valide donc la version du code et celle de l’extension, et renvoie une erreur si elles ne correspondent pas.

Cette valeur est définie par défaut sur true et est effective sur le coordinateur. Dans de rares cas, les processus de mise à niveau complexes peuvent nécessiter l’affectation de la valeur false à ce paramètre, ce qui désactive la vérification.

#### <a name="cituslog_distributed_deadlock_detection-boolean"></a>citus.log\_distributed\_deadlock\_detection (boléen)

Indique s’il faut enregistrer le traitement lié à la détection de blocage distribué dans le journal du serveur. La valeur par défaut est « False ».

#### <a name="citusdistributed_deadlock_detection_factor-floating-point"></a>citus.distributed\_deadlock\_detection\_factor (virgule flottante)

Définit le délai d’attente avant de vérifier les blocages distribués. En particulier, le délai d’attente sera cette valeur multipliée par le paramètre [deadlock\_timeout](https://www.postgresql.org/docs/current/static/runtime-config-locks.html) de PostgreSQL. La valeur par défaut est `2`. Une valeur de `-1` désactive la détection de blocage distribué.

#### <a name="citusnode_connection_timeout-integer"></a>citus.node\_connection\_timeout (integer)

Le GUC `citus.node_connection_timeout` définit la durée maximale (en millisecondes) d’attente de l’établissement de la connexion. Hyperscale (Citus) génère une erreur si le délai d’attente s’écoule avant qu’au moins une connexion Worker soit établie. Ce GUC affecte les connexions entre le coordinateur et les workers, ainsi qu’entre les workers.

-   Default : cinq secondes
-   Minimum : 10 millisecondes
-   Maximum : une heure

```postgresql
-- set to 30 seconds
ALTER DATABASE foo
SET citus.node_connection_timeout = 30000;
```

### <a name="query-statistics"></a>Statistiques sur les requêtes

#### <a name="citusstat_statements_purge_interval-integer"></a>citus.stat\_statements\_purge\_interval (integer)

Définit la fréquence à laquelle le démon de maintenance supprime les enregistrements de [citus_stat_statements](reference-hyperscale-metadata.md#query-statistics-table) qui n’ont pas de correspondance pas dans `pg_stat_statements`. Cette valeur de configuration définit l’intervalle de temps entre les suppressions, en secondes, avec une valeur par défaut de 10. La valeur 0 désactive les suppressions.

```psql
SET citus.stat_statements_purge_interval TO 5;
```

Ce paramètre est effectif sur le coordinateur et peut être modifié au moment de l’exécution.

### <a name="data-loading"></a>Chargement des données

#### <a name="citusmulti_shard_commit_protocol-enum"></a>citus.multi\_shard\_commit\_protocol (enum)

Définit le protocole de validation à utiliser lors de l’exécution d’une copie sur une table distribuée de hachage. À chaque sélection élective de partition individuelle, la commande COPY est effectuée dans un bloc de transaction pour garantir qu’aucune donnée n’est ingérée si une erreur se produit pendant la copie. Toutefois, il existe un cas de défaillance particulier dans lequel la commande COPY est réussie sur toutes les sélections électives, mais une défaillance (matérielle) se produit avant la validation de toutes les transactions. Ce paramètre peut être utilisé pour éviter la perte de données dans ce cas en choisissant entre les protocoles de validation suivants :

-   **2pc :** (par défaut) Les transactions dans lesquelles la copie est effectuée sur les sélections électives de partition sont d’abord préparées à l’aide d\'une [une validation en deux phases](http://www.postgresql.org/docs/current/static/sql-prepare-transaction.html) de PostgreSQL puis validées. Les validations ayant échoué peuvent être récupérées ou annulées manuellement à l’aide de COMMIT PREPARED ou ROLLBACK PREPARED, respectivement.
    Lors de l’utilisation de 2pc, [max\_prepared\_transactions](http://www.postgresql.org/docs/current/static/runtime-config-resource.html) doit être augmenté sur tous les workers, généralement sur la même valeur que le nombre maximal de connexions\_.
-   **1pc :** Les transactions dans lesquelles la commande COPY est effectuée sur les sélections électives de partition sont validées en une seule fois. Les données peuvent être perdues si une validation échoue après la fin de la copie sur toutes les sélections électives (rare).

#### <a name="citusshard_replication_factor-integer"></a>citus.shard\_replication\_factor (integer)

Définit le facteur de réplication pour partitions, c’est-à-dire le nombre de nœuds sur lesquels les partitions seront placées, et la valeur par défaut est 1. Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur. La valeur idéale pour ce paramètre dépend de la taille du cluster et du taux d’échec du nœud.  Par exemple, vous souhaiterez peut-être augmenter ce facteur de réplication si vous exécutez des clusters de grande taille et observez les défaillances de nœuds plus fréquemment.

#### <a name="citusshard_count-integer"></a>citus.shard\_count (integer)

Définit le nombre de partitions pour les tables partitionnées par hachage et par défaut à 32.  Cette valeur est utilisée par l’UDF (fonction définie par l’utilisateur) [create_distributed_table](reference-hyperscale-functions.md#create_distributed_table) lors de la création de tables partitionnées par hachage. Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

#### <a name="citusshard_max_size-integer"></a>citus.shard\_max\_size (integer)

Définit la taille maximale que doit atteindre une partition avant qu’elle ne soit fractionnée et est par défaut de 1 Go. Lorsque la taille du fichier source (utilisée pour la mise en lots) pour une partition dépasse cette valeur de configuration, la base de données s\'assure qu’une nouvelle partition est créée. Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

### <a name="planner-configuration"></a>Configuration du planificateur

#### <a name="cituslimit_clause_row_fetch_count-integer"></a>citus.limit\_clause\_row\_fetch\_count (integer)

Définit le nombre de lignes à extraire par tâche pour l’optimisation de la clause LIMIT.
Dans certains cas, les requêtes SELECT avec des clauses LIMIT peuvent avoir besoin d’extraire toutes les lignes de chaque tâche pour générer des résultats. Dans ces cas-là, et lorsqu’une approximation produirait des résultats significatifs, cette valeur de configuration définit le nombre de lignes à extraire à partir de chaque partition. Les approximations de limites sont désactivées par défaut et ce paramètre est définit sur -1. Cette valeur peut être définie au moment de l’exécution et est effective sur le coordinateur.

#### <a name="cituscount_distinct_error_rate-floating-point"></a>citus.count\_distinct\_error\_rate (floating point)

Hyperscale (Citus) peut calculer le nombre count(distinct) approximatif à l’aide de l’extension postgresql-hll. Cette entrée de configuration définit le taux d’erreur souhaité lors du calcul du count(distinct). 0,0, qui est la valeur par défaut, désactive les approximations pour count(distinct); et 1,0 ne fournit aucune garantie quant à l’exactitude des résultats. Nous vous recommandons de définir ce paramètre sur 0,005 pour obtenir de meilleurs résultats. Cette valeur peut être définie au moment de l’exécution et est effective sur le coordinateur.

#### <a name="citustask_assignment_policy-enum"></a>citus.task\_assignment\_policy (enum)

> [!NOTE]
> Ce paramètre GUC s’applique uniquement lorsque [shard_replication_factor](reference-hyperscale-parameters.md#citusshard_replication_factor-integer) est supérieur à un, ou pour les requêtes sur [reference_tables](concepts-hyperscale-distributed-data.md#type-2-reference-tables).

Définit la stratégie à utiliser lors de l’affectation de tâches aux workers. Le coordinateur affecte des tâches aux workers en fonction des emplacements de partition. Cette valeur de configuration spécifie la stratégie à utiliser lors de ces affectations.
Actuellement, il existe trois stratégies d’attribution de tâches possibles qui peuvent être utilisées.

-   **greedy :** La stratégie greedy est la stratégie par défaut et vise à répartir uniformément les tâches entre les workers.
-   **round-robin :** La stratégie round-robin attribue des tâches aux workers dans un mode round robin alternant entre les différents réplicas. Cette stratégie permet une meilleure utilisation du cluster lorsque le nombre de partitions pour une table est faible par rapport au nombre de workers.
-   **first-replica :** La stratégie first-replica assigne des tâches en fonction de l’ordre d’insertion des placements (réplicas) pour les partitions. En d’autres termes, la requête de fragment pour une partition est assignée au worker qui a le premier réplica de cette partition.
    Cette méthode vous permet d’avoir des garanties fortes quant à quelle partition sera utilisée sur quel nœuds (autrement dit, des garanties fichiers restant en mémoire plus fortes).

Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

### <a name="intermediate-data-transfer"></a>Transfert des données intermédiaires

#### <a name="citusbinary_worker_copy_format-boolean"></a>citus.binary\_worker\_copy\_format (boolean)

Utilisez le format de copie binaire pour transférer des données intermédiaires entre les workers.
Au cours des jointures de tables volumineuses, Hyperscale (Citus) peut être amené à repartitionner dynamiquement et à répartir les données entre différents Workers. Par défaut, ces données sont transférées au format texte. L’activation de ce paramètre indique à la base de données d’utiliser le format de sérialisation binaire PostgreSQL pour transférer ces données. Ce paramètre est effectif sur les Workers et doit être modifié dans le fichier PostgreSQL. conf. Après avoir modifié le fichier de configuration, les utilisateurs peuvent envoyer un signal SIGHUP ou redémarrer le serveur pour que cette modification prenne effet.

#### <a name="citusbinary_master_copy_format-boolean"></a>citus.binary\_master\_copy\_format (booléen)

Utilisez le format de copie binaire pour transférer des données entre les coordinateurs et les Workers. Lors de l’exécution de requêtes distribuées, les Workers transfèrent leurs résultats intermédiaires au coordinateur pour l’agrégation finale. Par défaut, ces données sont transférées au format texte. L’activation de ce paramètre indique à la base de données d’utiliser le format de sérialisation binaire PostgreSQL pour transférer ces données.
Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

#### <a name="citusmax_intermediate_result_size-integer"></a>citus.max\_intermediate\_result\_size (integer)

Taille maximale, en Ko, des résultats intermédiaires pour les expressions de table commune qui ne peuvent pas être transférées vers les nœuds Worker pour exécution, et pour les sous-requêtes complexes. La valeur par défaut est 1 Go et la valeur -1 signifie aucune limite.
Les requêtes qui dépassent la limite sont annulées et génèrent un message d’erreur.

### <a name="ddl"></a>DDL

#### <a name="citusenable_ddl_propagation-boolean"></a>citus.enable\_ddl\_propagation (booléen)

Spécifie s’il faut propager automatiquement les modifications DDL du coordinateur à tous les Workers. La valeur par défaut est true. Étant donné que certaines modifications de schéma nécessitent un verrou d’accès exclusif sur les tables et que la propagation automatique s’applique à tous les traitements de manière séquentielle, elle peut rendre un cluster Hyperscale (Citus) temporairement moins réactif. Vous pouvez choisir de désactiver ce paramètre et de propager les modifications manuellement.

### <a name="executor-configuration"></a>Configuration de l’exécuteur

#### <a name="general"></a>Général

##### <a name="citusall_modifications_commutative"></a>citus.all\_modifications\_commutative

Hyperscale (Citus) applique les règles de commutativité et acquiert les verrous appropriés pour les opérations de modification afin de garantir l’exactitude du comportement. Par exemple, il suppose qu’une instruction INSERT est commutative avec une autre instruction INSERT, mais pas avec une instruction UPDATE ou DELETE. De même, il suppose qu’une instruction UPDATE ou DELETE ne commute pas avec une autre instruction UPDATE ou DELETE. Cette précaution signifie que les instructions UPDATE et DELETE nécessitent Hyperscale (Citus) pour acquérir des verrous plus forts.

Si vous avez des instructions UPDATE qui sont commutatives avec vos INSERT ou d’autres UPDATE, vous pouvez assouplir ces hypothèses de commutativité en affectant à ce paramètre la valeur true. Lorsque ce paramètre est défini sur true, toutes les commandes sont considérées comme commutatives et revendiquent un verrou partagé, ce qui peut améliorer le débit global. Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

##### <a name="citusremote_task_check_interval-integer"></a>citus.remote\_task\_check\_interval (integer)

Définit la fréquence à laquelle Hyperscale (Citus) vérifie les états des tâches gérés par l’exécuteur de suivi de tâches. La valeur par défaut est 10 ms. Le coordinateur attribue des tâches aux Workers, puis vérifie régulièrement la progression de chaque tâche. Cette valeur de configuration définit l’intervalle de temps entre deux contrôles. Ce paramètre est effectif sur le coordinateur et peut être défini au moment de l’exécution.

##### <a name="citustask_executor_type-enum"></a>citus.task\_executor\_type (enum)

Hyperscale (Citus) a trois types d’exécuteur pour exécuter des requêtes SELECT distribuées.  L’exécuteur souhaité peut être sélectionné en définissant ce paramètre de configuration. Les valeurs acceptables pour ce paramètre sont :

-   **adaptive :** Valeur par défaut. Elle est optimale pour les réponses rapides aux requêtes qui impliquent des agrégations et des jointures en colocation s’étendant sur plusieurs partitions.
-   **task-tracker :** L’exécuteur de suivi des tâches est adapté aux requêtes durables et complexes qui nécessitent une réutilisation aléatoire des données sur les nœuds Worker et une gestion efficace des ressources.
-   **real-time :** (déconseillé) A un objectif similaire à celui de l’exécuteur adaptatif, mais il est moins flexible et peut entraîner une plus grande sollicitation des connexions sur les nœuds Worker.

Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

##### <a name="citusmulti_task_query_log_level-enum-multi_task_logging"></a>citus.multi\_task\_query\_log\_level (enum) {#multi_task_logging}

Définit un niveau de journal pour toute requête qui génère plus d’une tâche (c’est-à-dire, qui atteint plusieurs partitions). La journalisation est utile lors d’une migration d’applications multi-entité, car vous pouvez choisir d’émettre des erreurs ou des avertissements pour de telles requêtes, pour les trouver et leur ajouter un filtre client\_id- Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur. La valeur par défaut de ce paramètre est \'off\'.

Les valeurs prises en charge pour cette enum sont les suivantes :

-   **off :** Désactivez la journalisation des requêtes qui génèrent plusieurs tâches (autrement dit, s’étendent sur plusieurs partitions)
-   **debug :** Journalise l’instruction au niveau de gravité DEBUG.
-   **log :** Journalise l’instruction au niveau de gravité LOG. La ligne de journal inclut la requête SQL qui a été exécutée.
-   **notice :** Journalise l’instruction au niveau de gravité NOTICE.
-   **warning :** Journalise l’instruction au niveau de gravité WARNING.
-   **error :** Journalise l’instruction au niveau de gravité ERROR.

Il peut être utile d’utiliser `error` lors des tests de développement, ainsi qu’un niveau de journalisation plus bas comme `log` au cours du déploiement réel.
Si vous choisissez `log` , les requêtes multitâches apparaitront dans les journaux de base de données avec la requête elle-même affichée après \"STATEMENT.\"

```
LOG:  multi-task query about to be executed
HINT:  Queries are split to multiple tasks if they have to be split into several queries on the workers.
STATEMENT:  select * from foo;
```

##### <a name="citusenable_repartition_joins-boolean"></a>citus.enable\_repartition\_joins (booléen)

En règle générale, la tentative d’exécution de jointures de répartition avec l’exécuteur adaptatif échoue avec un message d’erreur.  Toutefois, l’affectation de la valeur true à `citus.enable_repartition_joins` permet à Hyperscale (Citus) de basculer temporairement dans l’exécuteur de suivi des tâches pour effectuer la jointure.  La valeur par défaut est false.

#### <a name="task-tracker-executor-configuration"></a>Configuration de l’exécuteur du suivi des tâches

##### <a name="citustask_tracker_delay-integer"></a>citus.task\_tracker\_delay (integer)

Ce paramètre définit la durée de veille du suivi des tâches entre les phases de gestion des tâches et la valeur par défaut de 200 ms. Le processus du suivi des tâches se réveille régulièrement, passe en revue toutes les tâches qui lui sont affectées et planifie et exécute ces tâches.  Ensuite, le suivi des tâches se met en veille pendant une période de temps avant de parcourir ces tâches à nouveau.
Cette valeur de configuration détermine la durée de cette période de sommeil. Ce paramètre est effectif sur les Workers et doit être modifié dans le fichier PostgreSQL. conf. Après avoir modifié le fichier de configuration, les utilisateurs peuvent envoyer un signal SIGHUP ou redémarrer le serveur pour que cette modification prenne effet.

Ce paramètre peut être réduit pour diminuer le délai dû à l’exécuteur du suivi des tâches en réduisant l’intervalle de temps entre les phases de gestion.
La diminution du délai est utile dans les cas où les requêtes de partition sont courtes et, par conséquent, mettent régulièrement à jour leur état.

##### <a name="citusmax_assign_task_batch_size-integer"></a>citus.max\_assign\_task\_batch\_size (integer)

L’exécuteur du suivi des tâches sur le coordinateur assigne de façon synchrone des tâches par lots au démon sur les Workers. Ce paramètre définit le nombre maximal de tâches à assigner dans un lot unique. Le choix d’une taille de lot plus importante permet une attribution plus rapide des tâches. Toutefois, si le nombre de Workers est élevé, cela peut prendre plus de temps pour que tous les Workers obtiennent des tâches.  Ce paramètre peut être défini au moment de l’exécution et est effectif sur le coordinateur.

##### <a name="citusmax_running_tasks_per_node-integer"></a>citus.max\_running\_tasks\_per\_node (integer)

Le processus du suivi des tâches planifie et exécute les tâches qui lui sont assignées, le cas échéant. Cette valeur de configuration définit le nombre maximal de tâches à exécuter simultanément sur un nœud à tout moment donné et la valeur par défaut est 8.

La limite vous permet de ne pas avoir de nombreuses tâches sur le disque en même temps et d\'éviter la contention I/O de disque. Si vos requêtes sont servies à partir de mémoire ou de disques SSD, vous pouvez augmenter max\_running\_tasks\_per\_node sans grand problème.

##### <a name="cituspartition_buffer_size-integer"></a>citus.partition\_buffer\_size (integer)

Définit la taille de la mémoire tampon à utiliser pour les opérations de partition et la valeur par défaut est de 8 Mo.
Hyperscale (Citus) permet de repartitionner les données de table en plusieurs fichiers lorsque deux tables volumineuses sont jointes. Une fois cette mémoire tampon remplie, les données de nouveau partitionnées sont vidées dans des fichiers sur le disque.  Cette entrée de configuration peut être définie au moment de l’exécution et est effective sur les Workers.

#### <a name="explain-output"></a>Sortie explain

##### <a name="citusexplain_all_tasks-boolean"></a>citus.explain\_all\_tasks (booléen)

Par défaut, Hyperscale (Citus) affiche le résultat d’une tâche arbitraire unique lors de l’exécution d’[EXPLAIN](http://www.postgresql.org/docs/current/static/sql-explain.html) sur une requête distribuée. Dans la plupart des cas, le résultat d’explain sera similaire entre les tâches. Parfois, certaines tâches sont planifiées différemment ou ont des durées d’exécution bien supérieures. Dans ce cas, il peut être utile d’activer ce paramètre, après quoi le résultat EXPLAIN inclura toutes les tâches. L’appliquer à toutes les tâches peut entraîner un délai plus long pour EXPLAIN.

## <a name="postgresql-parameters"></a>Paramètres PostgreSQL

* [DateStyle](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-OUTPUT) : définit le format d'affichage des valeurs de date et d'heure
* [IntervalStyle](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-OUTPUT) : définit le format d'affichage des valeurs d'intervalle
* [TimeZone](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TIMEZONE) : définit le fuseau horaire pour l'affichage et l'interprétation des horodatages
* [application_name](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-APPLICATION-NAME) : définit le nom de l'application à signaler dans les statistiques et les journaux
* [array_nulls](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-ARRAY-NULLS) : permet l'entrée d'éléments NULL dans les tableaux
* [autovacuum](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM) : lance le sous-processus de nettoyage automatique
* [autovacuum_analyze_scale_factor](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-ANALYZE-SCALE-FACTOR) : nombre d'insertions, de mises à jour ou de suppressions de tuple avant l'analyse, exprimé en fraction de reltuples
* [autovacuum_analyze_threshold](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-ANALYZE-THRESHOLD) : nombre minimum d'insertions, de mises à jour ou de suppressions de tuple avant l'analyse
* [autovacuum_naptime](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-NAPTIME) : durée de veille entre deux exécutions du nettoyage automatique
* [autovacuum_vacuum_cost_delay](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-COST-DELAY) : délai lié au coût du nettoyage, en millisecondes, pour le nettoyage automatique
* [autovacuum_vacuum_cost_limit](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-COST-LIMIT) : montant du coût du nettoyage disponible avant la mise en veille, pour le nettoyage automatique
* [autovacuum_vacuum_scale_factor](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-SCALE-FACTOR) : nombre de mises à jour ou de suppressions de tuple avant le nettoyage, exprimé en fraction de reltuples
* [autovacuum_vacuum_threshold](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-THRESHOLD) : nombre minimum de mises à jour ou de suppressions de tuple avant le nettoyage
* [autovacuum_work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-AUTOVACUUM-WORK-MEM) : définit la mémoire maximum à utiliser par chaque processus Worker de nettoyage automatique
* [backend_flush_after](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BACKEND-FLUSH-AFTER) : nombre de pages après lequel les écritures précédemment effectuées sont vidées sur le disque
* [backslash_quote](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-BACKSLASH-QUOTE) : détermine si le caractère « \' » est autorisé dans les littéraux de chaîne
* [bgwriter_delay](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-DELAY) : durée de veille entre deux cycles de l'enregistreur exécuté en arrière-plan
* [bgwriter_flush_after](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-FLUSH-AFTER) : nombre de pages après lequel les écritures précédemment effectuées sont vidées sur le disque
* [bgwriter_lru_maxpages](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-LRU-MAXPAGES) : nombre maximum de pages LRU à vider par cycle de l'enregistreur exécuté en arrière-plan
* [bgwriter_lru_multiplier](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-BGWRITER-LRU-MULTIPLIER) : multiple de l'utilisation moyenne du tampon à libérer par cycle
* [bytea_output](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-BYTEA-OUTPUT) : définit le format de sortie pour bytea
* [check_function_bodies](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CHECK-FUNCTION-BODIES) : vérifie le corps des fonctions pendant l'exécution de CREATE FUNCTION
* [checkpoint_completion_target](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-CHECKPOINT-COMPLETION-TARGET) : temps passé à vider les tampons dont l'intégrité est compromise lors du point de contrôle, exprimé en fraction de l'intervalle entre deux points de contrôle
* [checkpoint_timeout](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-CHECKPOINT-TIMEOUT) : définit le délai maximum entre deux points de contrôle automatiques du WAL
* [checkpoint_warning](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-CHECKPOINT-WARNING) : déclenche des avertissements si des segments de point de contrôle sont remplis plus fréquemment
* [client_encoding](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-ENCODING) : définit l'encodage du jeu de caractères du client
* [client_min_messages](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-CLIENT-MIN-MESSAGES) : définit les niveaux des messages envoyés au client
* [commit_delay](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-COMMIT-DELAY) : définit le délai, en microsecondes, entre la validation de la transaction et le vidage de WAL sur le disque
* [commit_siblings](https://www.postgresql.org/docs/12/runtime-config-wal.html#GUC-COMMIT-SIBLINGS) : définit le nombre minimum de transactions ouvertes simultanées avant d'exécuter commit_delay
* [constraint_exclusion](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CONSTRAINT-EXCLUSION) : permet au planificateur d'utiliser des contraintes pour optimiser les requêtes
* [cpu_index_tuple_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CPU-INDEX-TUPLE-COST) : définit l'estimation par le planificateur du coût de traitement de chaque entrée d'index pendant une analyse d'index
* [cpu_operator_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CPU-OPERATOR-COST) : définit l'estimation par le planificateur du coût de traitement de chaque appel d'opérateur ou de fonction
* [cpu_tuple_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CPU-TUPLE-COST) : définit l'estimation par le planificateur du coût de traitement de chaque tuple (ligne)
* [cursor_tuple_fraction](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-CURSOR-TUPLE-FRACTION) : définit l'estimation par le planificateur de la fraction des lignes d'un curseur qui sera récupérée
* [deadlock_timeout](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-DEADLOCK-TIMEOUT) : définit la durée d'attente sur un verrou avant de rechercher un blocage, en millisecondes
* [debug_pretty_print](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.3.1.3) : met en retrait les affichages de l'arborescence d'analyse et de planification
* [debug_print_parse](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.2.1.3) : consigne l'arborescence d'analyse de chaque requête
* [debug_print_plan](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.2.1.3) : consigne le plan d'exécution de chaque requête
* [debug_print_rewritten](https://www.postgresql.org/docs/current/runtime-config-logging.html#id-1.6.6.11.5.2.2.1.3) : consigne l'arborescence d'analyse réécrite de chaque requête
* [default_statistics_target](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-DEFAULT-STATISTICS-TARGET) : définit la cible des statistiques par défaut
* [default_tablespace](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TABLESPACE) : définit l'espace disque logique par défaut où les tables et index doivent être créés
* [default_text_search_config](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TEXT-SEARCH-CONFIG) : définit la configuration par défaut de la recherche de texte
* [default_transaction_deferrable](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-DEFERRABLE) : définit l'état de report par défaut des nouvelles transactions
* [default_transaction_isolation](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-ISOLATION) : définit le niveau d'isolement de chaque nouvelle transaction
* [default_transaction_read_only](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-READ-ONLY) : définit l'état de lecture seule par défaut des nouvelles transactions
* default_with_oids : crée de nouvelles tables avec OID par défaut
* [effective_cache_size](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-EFFECTIVE-CACHE-SIZE) : définit l'hypothèse du planificateur concernant la taille du cache de disque
* [enable_bitmapscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-BITMAPSCAN) : permet au planificateur d'utiliser des plans d'analyse bitmap
* [enable_gathermerge](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-GATHERMERGE) : permet au planificateur d'utiliser des plans de regroupement et de fusion
* [enable_hashagg](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-HASHAGG) : permet au planificateur d'utiliser des plans d'agrégation hachée
* [enable_hashjoin](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-HASHJOIN) : permet au planificateur d'utiliser des plans de jointure hachée
* [enable_indexonlyscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-INDEXONLYSCAN) : permet au planificateur d'utiliser des plans d'analyse d'index uniquement
* [enable_indexscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-INDEXSCAN) : permet au planificateur d'utiliser des plans d'analyse d'index
* [enable_material](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-MATERIAL) : permet au planificateur d'utiliser la matérialisation
* [enable_mergejoin](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-MERGEJOIN) : permet au planificateur d'utiliser des plans de jointure de fusion
* [enable_nestloop](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-NESTLOOP) : permet au planificateur d'utiliser des plans de jointure de boucles imbriquées
* [enable_seqscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-SEQSCAN) : permet au planificateur d'utiliser des plans d'analyse séquentielle
* [enable_sort](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-SORT) : permet au planificateur d'utiliser des étapes de tri explicites
* [enable_tidscan](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-ENABLE-TIDSCAN) : permet au planificateur d'utiliser des plans d'analyse des TID
* [escape_string_warning](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-ESCAPE-STRING-WARNING) : signale la présence de barres obliques inverses d'échappement dans les littéraux de chaîne ordinaires
* [exit_on_error](https://www.postgresql.org/docs/current/runtime-config-error-handling.html#GUC-EXIT-ON-ERROR) : met fin à la session en cas d'erreur
* [extra_float_digits](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-EXTRA-FLOAT-DIGITS) : définit le nombre de chiffres affichés pour les valeurs à virgule flottante
* [force_parallel_mode](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-FORCE-PARALLEL-MODE) : force l'utilisation de fonctions de requêtes en parallèle
* [from_collapse_limit](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-FROM-COLLAPSE-LIMIT) : définit la taille de la liste FROM au-delà de laquelle les sous-requêtes ne sont pas réduites
* [geqo](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO) : active l'optimisation génétique des requêtes (GEQO)
* [geqo_effort](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-EFFORT) : GEQO - un effort est fait pour définir la valeur par défaut d'autres paramètres GEQO
* [geqo_generations](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-GENERATIONS) : GEQO - nombre d'itérations de l'algorithme
* [geqo_pool_size](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-POOL-SIZE) : GEQO - nombre d'individus au sein de la population
* [geqo_seed](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-SEED) : GEQO - seed pour la sélection de chemin aléatoire
* [geqo_selection_bias](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-SELECTION-BIAS) : GEQO - pression sélective au sein de la population
* [geqo_threshold](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-GEQO-THRESHOLD) : définit le seuil d'éléments FROM au-delà duquel l'optimisation GEQO est utilisée
* [gin_fuzzy_search_limit](https://www.postgresql.org/docs/current/runtime-config-client.html#id-1.6.6.14.5.2.2.1.3) : définit le résultat maximum autorisé pour une recherche exacte par GIN
* [gin_pending_list_limit](https://www.postgresql.org/docs/current/runtime-config-client.html#id-1.6.6.14.2.2.23.1.3) : définit la taille maximum de la liste en attente pour l'index GIN
* [idle_in_transaction_session_timeout](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-IDLE-IN-TRANSACTION-SESSION-TIMEOUT) : définit la durée maximum d'inactivité autorisée pour toute transaction
* [join_collapse_limit](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-JOIN-COLLAPSE-LIMIT) : définit la taille de la liste FROM au-delà de laquelle les constructions JOIN ne sont pas aplaties
* [lc_monetary](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LC-MONETARY) : définit les paramètres régionaux pour le formatage des valeurs monétaires
* [lc_numeric](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LC-NUMERIC) : définit les paramètres régionaux pour le formatage des nombres
* [lo_compat_privileges](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-LO-COMPAT-PRIVILEGES) : active le mode de compatibilité descendante pour les vérifications de privilèges sur les objets volumineux
* [lock_timeout](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LOCK-TIMEOUT) : définit la durée maximum autorisée (en millisecondes) pour toute attente liée à un verrou. 0 désactive ce paramètre
* [log_autovacuum_min_duration](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#) : définit le délai d'exécution minimum au-dessus duquel les actions de nettoyage automatique seront consignées
* [log_checkpoints](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-CHECKPOINTS) : consigne chaque point de contrôle
* [log_connections](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-CONNECTIONS) : consigne chaque connexion réussie
* [log_destination](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DESTINATION) : définit la destination de la sortie du journal du serveur
* [log_disconnections](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DISCONNECTIONS) : consigne la fin d'une session, en indiquant sa durée
* [log_duration](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DURATION) : consigne la durée de chaque instruction SQL terminée
* [log_error_verbosity](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-ERROR-VERBOSITY) : définit la verbosité des messages consignés
* [log_lock_waits](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-LOCK-WAITS) : consigne les longues attentes liées à un verrou
* [log_min_duration_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-DURATION-STATEMENT) : définit la durée d'exécution minimum (en millisecondes) au-dessus de laquelle les instructions seront consignées. -1 désactive les durées des instructions de journalisation
* [log_min_error_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-ERROR-STATEMENT) : permet de consigner toutes les instructions générant des erreurs à ce niveau ou à un niveau supérieur
* [log_min_messages](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-MIN-MESSAGES) : définit les niveaux des messages consignés
* [log_replication_commands](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-REPLICATION-COMMANDS) : consigne chaque commande de réplication
* [log_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-STATEMENT) : définit le type d'instructions consignées
* [log_statement_stats](https://www.postgresql.org/docs/current/runtime-config-statistics.html#id-1.6.6.12.3.2.1.1.3) : pour chaque requête, inscrit les statistiques de performances cumulées dans le journal du serveur
* [log_temp_files](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-TEMP-FILES) : consigne l'utilisation des fichiers temporaires dont la taille est supérieure à ce nombre de kilo-octets
* [maintenance_work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAINTENANCE-WORK-MEM) : définit la mémoire maximum à utiliser pour les opérations de maintenance
* [max_parallel_workers](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PARALLEL-WORKERS) : définit le nombre maximum de Workers parallèles qui peuvent être actifs en même temps
* [max_parallel_workers_per_gather](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PARALLEL-WORKERS-PER-GATHER) : définit le nombre maximum de processus parallèles par nœud exécuteur
* [max_pred_locks_per_page](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-MAX-PRED-LOCKS-PER-PAGE) : définit le nombre maximum de tuples verrouillés par prédicat pour chaque page
* [max_pred_locks_per_relation](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-MAX-PRED-LOCKS-PER-RELATION) : définit le nombre maximum de pages et de tuples verrouillés par prédicat pour chaque relation
* [max_standby_archive_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-ARCHIVE-DELAY) : définit le délai maximum avant l'annulation des requêtes lorsqu'un serveur de secours traite des données WAL archivées
* [max_standby_streaming_delay](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-STANDBY-STREAMING-DELAY) : définit le délai maximum avant l'annulation des requêtes lorsqu'un serveur de secours traite des données WAL diffusées en continu
* [max_sync_workers_per_subscription](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-MAX-SYNC-WORKERS-PER-SUBSCRIPTION) : nombre maximum de Workers de synchronisation de table par abonnement
* [max_wal_size](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-MAX-WAL-SIZE) : définit la taille du WAL qui déclenche un point de contrôle
* [min_parallel_index_scan_size](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-MIN-PARALLEL-INDEX-SCAN-SIZE) : définit la quantité minimum de données d'index pour une analyse parallèle
* [min_wal_size](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-MIN-WAL-SIZE) : définit la taille minimum de réduction du WAL
* [operator_precedence_warning](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-OPERATOR-PRECEDENCE-WARNING) : émet un avertissement pour les constructions qui ont changé de signification depuis PostgreSQL 9.4
* [parallel_setup_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-PARALLEL-SETUP-COST) : définit l'estimation par le planificateur du coût de démarrage des processus Worker pour l'interrogation en parallèle
* [parallel_tuple_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-PARALLEL-TUPLE-COST) : définit l'estimation par le planificateur du coût du passage de chaque tuple (ligne) du Worker au back-end master
* [pg_stat_statements.save](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8) : enregistre les statistiques de pg_stat_statements en cas d'arrêt du serveur
* [pg_stat_statements.track](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8) : sélectionne les instructions suivies par pg_stat_statements
* [pg_stat_statements.track_utility](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8) : détermine si les commandes de l'utilitaire sont suivies par pg_stat_statements
* [quote_all_identifiers](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-QUOTE-ALL-IDENTIFIERS) : lors de la génération de fragments SQL, délimite tous les identificateurs
* [random_page_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-RANDOM-PAGE-COST) : définit l'estimation par le planificateur du coût d'une page de disque récupérée de manière non séquentielle
* [row_security](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-ROW-SECURITY) : active la sécurité des lignes
* [search_path](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-SEARCH-PATH) : définit l'ordre de recherche des schémas pour les noms qui ne sont pas qualifiés par schéma
* [seq_page_cost](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-SEQ-PAGE-COST) : définit l'estimation par le planificateur du coût d'une page de disque récupérée de manière séquentielle
* [session_replication_role](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-SESSION-REPLICATION-ROLE) : définit le comportement de la session pour les déclencheurs et les règles de réécriture
* [standard_conforming_strings](https://www.postgresql.org/docs/current/runtime-config-compatible.html#id-1.6.6.16.2.2.7.1.3) : permet aux chaînes « ... » de traiter littéralement les barres obliques inverses
* [statement_timeout](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT) : définit la durée maximum autorisée (en millisecondes) pour toute instruction. 0 désactive ce paramètre
* [synchronize_seqscans](https://www.postgresql.org/docs/current/runtime-config-compatible.html#id-1.6.6.16.2.2.8.1.3) : permet des analyses séquentielles synchronisées
* [synchronous_commit](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-SYNCHRONOUS-COMMIT) : définit le niveau de synchronisation de la transaction actuelle
* [tcp_keepalives_count](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-KEEPALIVES-COUNT) : nombre maximum de retransmissions TCP Keepalive
* [tcp_keepalives_idle](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-KEEPALIVES-IDLE) : délai entre deux transmissions TCP Keepalive
* [tcp_keepalives_interval](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-KEEPALIVES-INTERVAL) : délai entre deux retransmissions TCP Keepalive
* [temp_buffers](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-TEMP-BUFFERS) : définit le nombre maximum de tampons temporaires utilisés par chaque session de base de données
* [temp_tablespaces](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TEMP-TABLESPACES) : définit le ou les espaces disque logiques à utiliser pour les tables temporaires et les fichiers de tri
* [track_activities](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-ACTIVITIES) : collecte des informations sur l'exécution des commandes
* [track_counts](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-COUNTS) : collecte des statistiques sur l'activité de la base de données
* [track_functions](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-FUNCTIONS) : collecte des statistiques au niveau des fonctions sur l'activité de la base de données
* [track_io_timing](https://www.postgresql.org/docs/current/runtime-config-statistics.html#GUC-TRACK-IO-TIMING) : collecte des statistiques de minutage pour l'activité d'E/S de la base de données
* [transform_null_equals](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-TRANSFORM-NULL-EQUALS) : traite « expr = NULL » sous la forme « expr IS NULL »
* [vacuum_cost_delay](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-DELAY) : délai lié au coût du nettoyage, en millisecondes
* [vacuum_cost_limit](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-LIMIT) : montant du coût du nettoyage disponible avant la mise en veille
* [vacuum_cost_page_dirty](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-PAGE-DIRTY) : coût du nettoyage pour une page dont l'intégrité est compromise par le nettoyage
* [vacuum_cost_page_hit](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-PAGE-HIT) : coût du nettoyage pour une page trouvée dans le cache de la mémoire tampon
* [vacuum_cost_page_miss](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-VACUUM-COST-PAGE-MISS) : coût du nettoyage pour une page introuvable dans le cache de la mémoire tampon
* [vacuum_defer_cleanup_age](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-VACUUM-DEFER-CLEANUP-AGE) : nombre de transactions en fonction desquelles les nettoyages VACUUM et HOT doivent être reportés, le cas échéant
* [vacuum_freeze_min_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-FREEZE-MIN-AGE) : âge minimum auquel VACUUM doit geler une ligne de la table
* [vacuum_freeze_table_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-FREEZE-TABLE-AGE) : âge auquel VACUUM doit analyser toute la table pour geler les tuples
* [vacuum_multixact_freeze_min_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-MULTIXACT-FREEZE-MIN-AGE) : âge minimum auquel VACUUM doit geler un MultiXactId sur une ligne de la table
* [vacuum_multixact_freeze_table_age](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-VACUUM-MULTIXACT-FREEZE-TABLE-AGE) : âge Multixact auquel VACUUM doit analyser toute la table pour geler les tuples
* [wal_receiver_status_interval](https://www.postgresql.org/docs/current/runtime-config-replication.html#GUC-WAL-RECEIVER-STATUS-INTERVAL) : définit l'intervalle maximum entre les rapports sur l'état du récepteur WAL
* [wal_writer_delay](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-WRITER-DELAY) : délai entre les vidages WAL effectués dans l'enregistreur WAL
* [wal_writer_flush_after](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-WRITER-FLUSH-AFTER) : quantité de WAL écrite par l'enregistreur WAL qui déclenche un vidage
* [work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-WORK-MEM) : définit la quantité de mémoire à utiliser par les opérations de tri internes et les tables de hachage avant d'écrire dans des fichiers des disques temporaires
* [xmlbinary](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-XMLBINARY) : détermine comment les valeurs binaires doivent être encodées en XML
* [xmloption](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-XMLOPTION) : détermine si les données XML des opérations d'analyse et de sérialisation implicites doivent être considérées comme des documents ou des fragments de contenu

## <a name="next-steps"></a>Étapes suivantes

* Les [options de configuration](concepts-hyperscale-configuration-options.md)de ressource d’un groupe de serveur Hyperscale (Citus) sont une autre forme de configuration.
* La base de données PostgreSQL sous-jacente a également des [paramètres de configuration](http://www.postgresql.org/docs/current/static/runtime-config.html).
