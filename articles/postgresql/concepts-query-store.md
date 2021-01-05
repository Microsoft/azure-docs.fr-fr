---
title: Magasin des requêtes - Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit la fonctionnalité Magasin des requêtes dans Azure Database pour PostgreSQL - Serveur unique.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 07/01/2020
ms.openlocfilehash: 5dff78989eef17f95d8b8dd108baafc53a3f761a
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657020"
---
# <a name="monitor-performance-with-the-query-store"></a>Superviser les performances avec le Magasin des requêtes

**S’applique à :** Azure Database pour PostgreSQL - Serveur unique versions 9.6 et ultérieures

La fonctionnalité Magasin des requêtes dans Azure Database pour PostgreSQL fournit un moyen de suivre les performances des requêtes dans le temps. Le Magasin des requêtes simplifie la résolution des problèmes de performances en vous aidant à identifier rapidement les requête dont l’exécution est la plus longue et qui consomment le plus de ressources. Le Magasin des requêtes capture automatiquement un historique des requêtes et des statistiques d’exécution, et les conserve pour que vous les passiez en revue. Il sépare les données par fenêtres de temps afin que vous puissiez voir les modèles d’utilisation des bases de données. Les données de tous les utilisateurs, des bases de données et des requêtes sont stockées dans une base de données nommée **azure_sys** dans l’instance Azure Database pour PostgreSQL.

> [!IMPORTANT]
> Ne modifiez pas la base de données **azure_sys** ou ses schémas. Si vous le faites, le Magasin des requêtes et les fonctionnalités de performances associées ne fonctionneront pas correctement.

## <a name="enabling-query-store"></a>Activation du Magasin des requêtes
Le Magasin des requêtes étant une fonctionnalité avec option d’adhésion, elle n’est pas activée par défaut sur un serveur. Le magasin est activé ou désactivé de façon globale pour toutes les bases de données se trouvant sur un serveur donné, et ne peut pas être activé ou désactivé par base de données.

### <a name="enable-query-store-using-the-azure-portal"></a>Activer le Magasin des requêtes à l’aide du portail Azure
1. Connectez-vous au portail Azure, puis sélectionnez votre serveur Azure Database pour PostgreSQL.
2. Sélectionnez **Paramètres du serveur** dans la section **Paramètres** du menu.
3. Recherchez le paramètre `pg_qs.query_capture_mode`.
4. Définissez la valeur sur `TOP` et cliquez sur **Enregistrer**.

Pour activer les statistiques d’attente dans votre magasin des requêtes, procédez comme suit : 
1. Recherchez le paramètre `pgms_wait_sampling.query_capture_mode`.
1. Définissez la valeur sur `ALL` et cliquez sur **Enregistrer**.


Vous pouvez également définir ces paramètres à l’aide de l’interface Azure CLI.
```azurecli-interactive
az postgres server configuration set --name pg_qs.query_capture_mode --resource-group myresourcegroup --server mydemoserver --value TOP
az postgres server configuration set --name pgms_wait_sampling.query_capture_mode --resource-group myresourcegroup --server mydemoserver --value ALL
```

Autoriser jusqu’à 20 minutes de conservation pour le premier lot de données dans la base de données azure_sys.

## <a name="information-in-query-store"></a>Informations dans le Magasin des requêtes
Le Magasin des requêtes comprend deux magasins :
- Un magasin des statistiques d’exécution pour conserver les informations sur les statistiques d’exécution des requêtes.
- Un magasin des statistiques d’attente pour conserver les informations sur les statistiques d’attente.

Les scénarios courants pour l’utilisation du Magasin des requêtes sont notamment les suivants :
- Détermination du nombre de fois où une requête a été exécutée dans une fenêtre de temps donnée
- Comparaison de la durée d’exécution moyenne d’une requête sur des fenêtres de temps pour voir les deltas importants
- Identification des requêtes dont l’exécution est la plus longue au cours des X dernières heures
- Identification des N premières requêtes en attente de ressources
- Compréhension de la nature de l’attente pour une requête particulière

Pour réduire l’utilisation de l’espace, les statistiques d’exécution du runtime dans le magasin des statistiques d’exécution sont agrégées pendant une fenêtre de temps configurable fixe. Les informations contenues dans ces magasins sont visibles en interrogeant les vues des magasins de requêtes.

## <a name="access-query-store-information"></a>Accéder aux informations du Magasin des requêtes

Les données du Magasin des requêtes sont stockées dans la base de données azure_sys sur votre serveur Postgres. 

La requête suivante retourne des informations sur les requêtes du Magasin des requêtes :
```sql
SELECT * FROM query_store.qs_view; 
``` 

Ou cette requête pour les statistiques d’attente :
```sql
SELECT * FROM query_store.pgms_wait_sampling_view;
```

## <a name="finding-wait-queries"></a>Recherche de requêtes d’attente
Les types d’événements d’attente combinent différents événements d’attente dans des compartiments par similarité. Le Magasin des requêtes fournit le type d’événement d’attente, le nom d’événement d’attente spécifique et la requête en question. Pouvoir mettre en corrélation ces informations d’attente avec les statistiques d’exécution de requête vous permet de mieux comprendre ce qui contribue aux caractéristiques de performances des requêtes.

Voici quelques exemples illustrant la façon d’obtenir plus d’insights dans votre charge de travail à l’aide des statistiques d’attente dans le Magasin des requêtes :

| **Observation** | **Action** |
|---|---|
|Attentes de verrous élevés | Vérifiez les textes de requêtes pour les requêtes affectées et identifiez les entités cibles. Recherchez dans le Magasin des requêtes d’autres requêtes modifiant la même entité, qui est fréquemment exécutée et/ou dont la durée d’exécution est longue. Après avoir identifié ces requêtes, envisagez de changer la logique d’application pour améliorer l’accès concurrentiel, ou utilisez un niveau d’isolation moins restrictif.|
| Attentes d’E/S de mémoire tampon élevées | Recherchez les requêtes comportant un grand nombre de lectures physiques dans le Magasin des requêtes. Si elles correspondent aux requêtes ayant des attentes d’E/S élevées, envisagez d’introduire un index sur l’entité sous-jacente, afin de faire des recherches plutôt que des analyses. Cela réduit la surcharge d’E/S des requêtes. Consultez les **Recommandations en matière de performances** pour votre serveur dans le portail afin de voir s’il existe des recommandations relatives aux index adaptées à ce serveur qui optimiseraient les requêtes.|
| Attentes de mémoire élevées | Recherchez les principales requêtes consommatrices de mémoire dans le Magasin des requêtes. Ces requêtes retardent probablement davantage la progression des requêtes affectées. Consultez les **Recommandations en matière de performances** pour votre serveur dans le portail afin de voir s’il existe des recommandations relatives aux index qui optimiseraient ces requêtes.|

## <a name="configuration-options"></a>Options de configuration
Quand le Magasin des requêtes est activé, il enregistre les données dans des fenêtres d’agrégation de 15 minutes, avec jusqu’à 500 requêtes distinctes par fenêtre. 

Les options suivantes sont disponibles pour la configuration des paramètres du Magasin des requêtes.

| **Paramètre** | **Description** | **Par défaut** | **Plage**|
|---|---|---|---|
| pg_qs.query_capture_mode | Définit les instructions qui sont suivies. | Aucun | none, top, all |
| pg_qs.max_query_text_length | Définit la longueur maximale de requête qui peut être enregistrée. Les requêtes les plus longues sont tronquées. | 6000 | 100 - 10 000 |
| pg_qs.retention_period_in_days | Définit la période de conservation. | 7 | 1 - 30 |
| pg_qs.track_utility | Définit si les commandes d’utilitaire sont suivies | sur | on, off |

Les options suivantes s’appliquent spécifiquement aux statistiques d’attente.

| **Paramètre** | **Description** | **Par défaut** | **Plage**|
|---|---|---|---|
| pgms_wait_sampling.query_capture_mode | Définit les instructions qui sont suivies pour les statistiques d’attente. | Aucun | none, all|
| Pgms_wait_sampling.history_period | Définissez la fréquence, en millisecondes, à laquelle les événements d’attente sont échantillonnés. | 100 | 1 - 600 000 |

> [!NOTE] 
> **pg_qs.query_capture_mode** remplace **pgms_wait_sampling.query_capture_mode**. Si pg_qs.query_capture_mode a la valeur NONE, le paramètre pgms_wait_sampling.query_capture_mode n’a aucun effet.


Utilisez le [portail Azure](howto-configure-server-parameters-using-portal.md) ou l’interface [Azure CLI](howto-configure-server-parameters-using-cli.md) pour obtenir ou définir une valeur différente pour un paramètre.

## <a name="views-and-functions"></a>Vues et fonctions
Affichez et gérez le Magasin des requêtes à l’aide des fonctions et vues suivantes. Quiconque dans le rôle public PostgreSQL peut utiliser ces vues pour afficher les données du Magasin des requêtes. Ces vues sont disponibles uniquement dans la base de données **azure_sys**.

Les requêtes sont normalisées en examinant leur structure après la suppression des littéraux et des constantes. Si deux requêtes sont identiques à l’exception des valeurs littérales, elle ont le même hachage.

### <a name="query_storeqs_view"></a>query_store.qs_view
Cette vue retourne toutes les données du Magasin des requêtes. Il existe une ligne pour chaque ID de base de données distinct, ID d’utilisateur et ID de requête. 

|**Nom**   |**Type** | **Informations de référence**  | **Description**|
|---|---|---|---|
|runtime_stats_entry_id |bigint | | ID de la table runtime_stats_entries|
|user_id    |oid    |pg_authid.oid  |OID de l’utilisateur qui a exécuté l’instruction|
|db_id  |oid    |pg_database.oid    |OID de la base de données dans laquelle l’instruction a été exécutée|
|query_id   |bigint  || Code de hachage interne, calculé à partir de l’arborescence d’analyse de l’instruction|
|query_sql_text |Varchar(10000)  || Texte d’une instruction représentative. Différentes requêtes ayant la même structure sont regroupées en clusters ; ce texte est le texte de la première des requêtes du cluster.|
|plan_id    |bigint |   |ID du plan correspondant à cette requête, pas encore disponible|
|start_time |timestamp  ||  Les requêtes sont agrégées par intervalle de planification : la durée d’un compartiment est de 15 minutes par défaut. Il s’agit de l’heure de début correspondant à l’intervalle de planification pour cette entrée.|
|end_time   |timestamp  ||  Heure de fin correspondant à l’intervalle de planification pour cette entrée.|
|calls  |bigint  || Nombre de fois où la requête s’est exécutée|
|total_time |double précision   ||  Durée totale d’exécution de la requête, en millisecondes|
|min_time   |double précision   ||  Durée minimale d’exécution de la requête, en millisecondes|
|max_time   |double précision   ||  Durée maximale d’exécution de la requête, en millisecondes|
|mean_time  |double précision   ||  Durée moyenne d’exécution de la requête, en millisecondes|
|stddev_time|   double précision    ||  Écart type de la durée d’exécution de la requête, en millisecondes |
|rows   |bigint ||  Nombre total de lignes récupérées ou affectées par l’instruction|
|shared_blks_hit|   bigint  ||  Nombre total d’accès au cache de blocs partagés par l’instruction|
|shared_blks_read|  bigint  ||  Nombre total de blocs partagés lus par l’instruction|
|shared_blks_dirtied|   bigint   || Nombre total de blocs partagés modifiés par l’instruction |
|shared_blks_written|   bigint  ||  Nombre total de blocs partagés écrits par l’instruction|
|local_blks_hit|    bigint ||   Nombre total d’accès au cache de blocs locaux par l’instruction|
|local_blks_read|   bigint   || Nombre total de blocs locaux lus par l’instruction|
|local_blks_dirtied|    bigint  ||  Nombre total de blocs locaux modifiés par l’instruction|
|local_blks_written|    bigint  ||  Nombre total de blocs locaux écrits par l’instruction|
|temp_blks_read |bigint  || Nombre total de blocs temporaires lus par l’instruction|
|temp_blks_written| bigint   || Nombre total de blocs temporaires écrits par l’instruction|
|blk_read_time  |double précision    || Durée totale passée par l’instruction à lire des blocs, en millisecondes (si track_io_timing est activé ; sinon, zéro)|
|blk_write_time |double précision    || Durée totale passée par l’instruction à écrire des blocs, en millisecondes (si track_io_timing est activé ; sinon, zéro)|
    
### <a name="query_storequery_texts_view"></a>query_store.query_texts_view
Cette vue retourne les données du texte des requêtes du Magasin des requêtes. Il existe une ligne pour chaque valeur query_text distincte.

| **Nom** | **Type** | **Description** |
|--|--|--|
| query_text_id | bigint | ID de la table query_texts |
| query_sql_text | Varchar(10000) | Texte d’une instruction représentative. Différentes requêtes ayant la même structure sont regroupées en clusters ; ce texte est le texte de la première des requêtes du cluster. |

### <a name="query_storepgms_wait_sampling_view"></a>query_store.pgms_wait_sampling_view
Cette vue retourne les données des événements d’attente du Magasin des requêtes. Il existe une ligne pour chaque ID de base de données, ID d’utilisateur, ID de requête et événement distinct.

| **Nom** | **Type** | **Informations de référence** | **Description** |
|--|--|--|--|
| user_id | oid | pg_authid.oid | OID de l’utilisateur qui a exécuté l’instruction |
| db_id | oid | pg_database.oid | OID de la base de données dans laquelle l’instruction a été exécutée |
| query_id | bigint |  | Code de hachage interne, calculé à partir de l’arborescence d’analyse de l’instruction |
| event_type | text |  | Type d’événement pour lequel le backend est en attente |
| événement | text |  | Nom de l’événement d’attente si le backend est actuellement en attente |
| calls | Integer |  | Nombre du même événement capturé |

### <a name="functions"></a>Fonctions

Query_store.qs_reset() retourne void

`qs_reset` ignore toutes les statistiques collectées jusqu’à présent par le Magasin des requêtes. Cette fonction peut uniquement être exécutée par le rôle d’administrateur de serveur.

Query_store.staging_data_reset() retourne void

`staging_data_reset` ignore toutes les statistiques collectées en mémoire par le Magasin des requêtes (autrement dit, les données en mémoire qui n’ont pas encore été vidées dans la base de données). Cette fonction peut uniquement être exécutée par le rôle d’administrateur de serveur.


## <a name="azure-monitor"></a>Azure Monitor
Azure Database pour PostgreSQL est intégré aux [paramètres de diagnostic d’Azure Monitor](../azure-monitor/platform/diagnostic-settings.md). Les paramètres de diagnostic vous permettent d’envoyer vos journaux Postgres au format JSON aux [journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md) à des fins d’analytique et d’alerte, à Event Hubs pour la diffusion en continu et à Stockage Azure pour l’archivage.

>[!IMPORTANT]
> Cette fonctionnalité de diagnostic n’est disponible que dans les niveaux tarifaires Usage général et Mémoire optimisée.

### <a name="configure-diagnostic-settings"></a>Configurer les paramètres de diagnostic
Vous pouvez activer les paramètres de diagnostic pour votre serveur Postgres à l’aide du portail Azure, de l’interface CLI, de l’API REST et de PowerShell. Les catégories de journaux à configurer sont **QueryStoreRuntimeStatistics** et **QueryStoreWaitStatistics**. 

Pour activer les journaux de ressources à l’aide du portail Azure :

1. Dans le portail, accédez à Paramètres de diagnostic dans le menu de navigation de votre serveur Postgres.
2. Sélectionnez Ajouter un paramètre de diagnostic.
3. Donnez un nom à ce paramètre.
4. Sélectionnez le point de terminaison de votre choix (compte de stockage, hub d’événements, analytique des journaux).
5. Sélectionnez les types de journaux **QueryStoreRuntimeStatistics** et **QueryStoreWaitStatistics**.
6. Enregistrez votre paramètre.

Pour activer ce paramètre à l’aide de PowerShell, de l’interface CLI ou de l’API REST, consultez l’[article sur les paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md).

### <a name="json-log-format"></a>Format de journal JSON
Les tableaux suivants décrivent les champs pour les deux types de journaux. En fonction du point de terminaison de sortie choisi, les champs et l’ordre dans lequel ils apparaissent peuvent varier.

#### <a name="querystoreruntimestatistics"></a>QueryStoreRuntimeStatistics
|**Champ** | **Description** |
|---|---|
| TimeGenerated [UTC] | Horodatage du moment où le journal a été enregistré en UTC |
| ResourceId | URI de ressource Azure du serveur Postgres |
| Category | `QueryStoreRuntimeStatistics` |
| NomOpération | `QueryStoreRuntimeStatisticsEvent` |
| LogicalServerName_s | Nom du serveur Postgres | 
| runtime_stats_entry_id_s | ID de la table runtime_stats_entries |
| user_id_s | OID de l’utilisateur qui a exécuté l’instruction |
| db_id_s | OID de la base de données dans laquelle l’instruction a été exécutée |
| query_id_s | Code de hachage interne, calculé à partir de l’arborescence d’analyse de l’instruction |
| end_time_s | Heure de fin correspondant à l’intervalle de planification pour cette entrée |
| calls_s | Nombre de fois où la requête s’est exécutée |
| total_time_s | Durée totale d’exécution de la requête, en millisecondes |
| min_time_s | Durée minimale d’exécution de la requête, en millisecondes |
| max_time_s | Durée maximale d’exécution de la requête, en millisecondes |
| mean_time_s | Durée moyenne d’exécution de la requête, en millisecondes |
| ResourceGroup | Groupe de ressources | 
| SubscriptionId | Votre ID d’abonnement |
| ResourceProvider | `Microsoft.DBForPostgreSQL` | 
| Ressource | Nom du serveur Postgres |
| ResourceType | `Servers` | 


#### <a name="querystorewaitstatistics"></a>QueryStoreWaitStatistics
|**Champ** | **Description** |
|---|---|
| TimeGenerated [UTC] | Horodatage du moment où le journal a été enregistré en UTC |
| ResourceId | URI de ressource Azure du serveur Postgres |
| Category | `QueryStoreWaitStatistics` |
| NomOpération | `QueryStoreWaitEvent` |
| user_id_s | OID de l’utilisateur qui a exécuté l’instruction |
| db_id_s | OID de la base de données dans laquelle l’instruction a été exécutée |
| query_id_s | Code de hachage interne de la requête |
| calls_s | Nombre du même événement capturé |
| event_type_s | Type d’événement pour lequel le backend est en attente |
| event_s | Nom de l’événement d’attente si le back-end est actuellement en attente |
| start_time_t | Heure de début de l’événement |
| end_time_s | Heure de fin de l’événement | 
| LogicalServerName_s | Nom du serveur Postgres | 
| ResourceGroup | Groupe de ressources | 
| SubscriptionId | Votre ID d’abonnement |
| ResourceProvider | `Microsoft.DBForPostgreSQL` | 
| Ressource | Nom du serveur Postgres |
| ResourceType | `Servers` | 

## <a name="limitations-and-known-issues"></a>Limitations et problèmes connus
- Si un serveur PostgreSQL a le paramètre default_transaction_read_only activé, le Magasin des requêtes ne peut pas capturer les données.
- La fonctionnalité Magasin des requêtes peut être interrompue si elle rencontre des requêtes Unicode longues (>= 6 000 octets).
- Les [réplicas en lecture](concepts-read-replicas.md) répliquent les données du magasin des requêtes qui se trouve sur le serveur principal. Cela signifie que le magasin des requêtes d’un réplica en lecture ne fournit pas de statistiques concernant les requêtes qui sont exécutées sur le réplica en lecture.


## <a name="next-steps"></a>Étapes suivantes
- Découvrez plus en détail les [scénarios où le Magasin des requêtes peut être particulièrement utile](concepts-query-store-scenarios.md).
- Découvrez plus en détail les [bonnes pratiques relatives à l’utilisation du Magasin des requêtes](concepts-query-store-best-practices.md).
