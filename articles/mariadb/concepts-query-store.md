---
title: Magasin de données des requêtes - Azure Database for MariaDB
description: La fonctionnalité Magasin des requêtes dans Azure Database for MariaDB vous aide à suivre les performances dans le temps.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: bca995f8b2cea33266e032b543abb18ee7140f3f
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541179"
---
# <a name="monitor-azure-database-for-mariadb-performance-with-query-store"></a>Surveiller les performances de Azure Database for MariaDB avec Magasin de données des requêtes

**S’applique à :** Azure Database for MariaDB 10.2

La fonctionnalité Magasin de données des requêtes dans Azure Database pour Mariadb fournit un moyen de suivre les performances des requêtes dans le temps. Le Magasin des requêtes simplifie la résolution des problèmes de performances en vous aidant à identifier rapidement les requête dont l’exécution est la plus longue et qui consomment le plus de ressources. Le Magasin des requêtes capture automatiquement un historique des requêtes et des statistiques d’exécution, et les conserve pour que vous les passiez en revue. Il sépare les données par fenêtres de temps afin que vous puissiez voir les modèles d’utilisation des bases de données. Les données de tous les utilisateurs, des bases de données et des requêtes sont stockées dans une base de données de schéma **mysql** dans l’instance Azure Database for MariaDB.

## <a name="common-scenarios-for-using-query-store"></a>Scénarios courants pour l’utilisation de Magasin de données des requêtes

Le magasin de données des requêtes peut être utilisé dans de nombreux scénarios, dont les suivants :

- Détection des requêtes de régression
- Détermination du nombre de fois où une requête a été exécutée dans une fenêtre de temps donnée
- Comparaison de la durée d’exécution moyenne d’une requête sur des fenêtres de temps pour voir les deltas importants

## <a name="enabling-query-store"></a>Activation du Magasin des requêtes

Le Magasin des requêtes étant une fonctionnalité avec option d’adhésion, elle n’est pas activée par défaut sur un serveur. Le magasin est activé ou désactivé de façon globale pour toutes les bases de données se trouvant sur un serveur donné, et ne peut pas être activé ou désactivé par base de données.

### <a name="enable-query-store-using-the-azure-portal"></a>Activer le Magasin des requêtes à l’aide du portail Azure

1. Connectez-vous au portail Azure, puis sélectionnez votre serveur Azure Database for MariaDB.
1. Sélectionnez **Paramètres du serveur** dans la section **Paramètres** du menu.
1. Recherchez le paramètre query_store_capture_mode.
1. Affectez ALL à la valeur et cliquez sur **Enregistrer**.

Pour activer les statistiques d’attente dans votre magasin des requêtes, procédez comme suit :

1. Recherchez le paramètre query_store_wait_sampling_capture.
1. Affectez ALL à la valeur et cliquez sur **Enregistrer**.

Autoriser jusqu’à 20 minutes de conservation pour le premier lot de données dans la base de données mysql.

## <a name="information-in-query-store"></a>Informations dans le Magasin des requêtes

Le Magasin des requêtes comprend deux magasins :

- Le magasin des statistiques d’exécution pour conserver les informations sur les statistiques d’exécution des requêtes.
- Le magasin des statistiques d’attente pour conserver les informations sur les statistiques d’attente.

Pour réduire l’utilisation de l’espace, les statistiques d’exécution du runtime dans le magasin des statistiques d’exécution sont agrégées pendant une fenêtre de temps configurable fixe. Les informations contenues dans ces magasins sont visibles en interrogeant les vues des magasins de requêtes.

La requête suivante retourne des informations sur les requêtes du Magasin des requêtes :

```sql
SELECT * FROM mysql.query_store;
```

Ou cette requête pour les statistiques d’attente :

```sql
SELECT * FROM mysql.query_store_wait_stats;
```

## <a name="finding-wait-queries"></a>Recherche de requêtes d’attente

> [!NOTE]
> Les statistiques d’attente ne doivent pas être activées pendant les heures de pic de charges de travail ou être activées indéfiniment pour les charges de travail sensibles. <br>Pour les charges de travail dont l’exécution implique une utilisation élevée du processeur ou qui sont exécutées sur des serveurs configurés avec des vCores inférieurs, activez les statistiques d’attente avec prudence. Elles ne doivent pas être activées indéfiniment. 

Les types d’événements d’attente combinent différents événements d’attente dans des compartiments par similarité. Le Magasin des requêtes fournit le type d’événement d’attente, le nom d’événement d’attente spécifique et la requête en question. Pouvoir mettre en corrélation ces informations d’attente avec les statistiques d’exécution de requête vous permet de mieux comprendre ce qui contribue aux caractéristiques de performances des requêtes.

Voici quelques exemples illustrant la façon d’obtenir plus d’insights dans votre charge de travail à l’aide des statistiques d’attente dans le Magasin des requêtes :

| **Observation** | **Action** |
|---|---|
|Attentes de verrous élevés | Vérifiez les textes de requêtes pour les requêtes affectées et identifiez les entités cibles. Recherchez dans le Magasin des requêtes d’autres requêtes modifiant la même entité, qui est fréquemment exécutée et/ou dont la durée d’exécution est longue. Après avoir identifié ces requêtes, envisagez de changer la logique d’application pour améliorer l’accès concurrentiel, ou utilisez un niveau d’isolation moins restrictif. |
|Attentes d’E/S de mémoire tampon élevées | Recherchez les requêtes comportant un grand nombre de lectures physiques dans le Magasin des requêtes. Si elles correspondent aux requêtes ayant des attentes d’E/S élevées, envisagez d’introduire un index sur l’entité sous-jacente, afin de faire des recherches plutôt que des analyses. Cela réduit la surcharge d’E/S des requêtes. Consultez les **Recommandations en matière de performances** pour votre serveur dans le portail afin de voir s’il existe des recommandations relatives aux index adaptées à ce serveur qui optimiseraient les requêtes. |
|Attentes de mémoire élevées | Recherchez les principales requêtes consommatrices de mémoire dans le Magasin des requêtes. Ces requêtes retardent probablement davantage la progression des requêtes affectées. Consultez les **Recommandations en matière de performances** pour votre serveur dans le portail afin de voir s’il existe des recommandations relatives aux index qui optimiseraient ces requêtes.|

## <a name="configuration-options"></a>Options de configuration

Quand le Magasin des requêtes est activé, il enregistre les données dans des fenêtres d’agrégation de 15 minutes, avec jusqu’à 500 requêtes distinctes par fenêtre.

Les options suivantes sont disponibles pour la configuration des paramètres du Magasin des requêtes.

| **Paramètre** | **Description** | **Par défaut** | **Plage** |
|---|---|---|---|
| query_store_capture_mode | Permet d’activer/de désactiver la fonctionnalité magasin de données des requêtes en fonction de la valeur. Remarque : Si performance_schema est désactivé, l’activation de query_store_capture_mode active performance_schema et un sous-ensemble d’instruments de schéma de performances requis pour la fonctionnalité. | ALL | NONE, ALL |
| query_store_capture_interval | Le magasin de données des requêtes capture l’intervalle en minutes. Permet de spécifier l’intervalle dans lequel les métriques des requêtes sont agrégées | 15 | 5 - 60 |
| query_store_capture_utility_queries | L’activation/La désactivation permet de capturer toutes les requêtes d’utilitaire qui s’exécute dans le système. | Non | OUI, NON |
| query_store_retention_period_in_days | Période de conservation des données du magasin de données des requêtes exprimée en jours. | 7 | 1 - 30 |

Les options suivantes s’appliquent spécifiquement aux statistiques d’attente.

| **Paramètre** | **Description** | **Par défaut** | **Plage** |
|---|---|---|---|
| query_store_wait_sampling_capture_mode | Permet d’activer/de désactiver les statistiques d’attente. | Aucune | NONE, ALL |
| query_store_wait_sampling_frequency | Permet de modifier la fréquence de wait-sampling en secondes. 5 à 300 secondes. | 30 | 5-300 |

> [!NOTE]
> Actuellement **query_store_capture_mode** remplace cette configuration, ce qui signifie que **query_store_capture_mode** et **query_store_wait_sampling_capture_mode** doivent être activés pour TOUS afin que les statistiques d’attente fonctionnent. Si **query_store_capture_mode** est désactivé, les statistiques d’attente sont également désactivées dans la mesure où les statistiques d’attente utilisent le performance_schema activé et le query_text capturé par le magasin de données des requêtes.

Utilisez le [portail Azure](howto-server-parameters.md) pour obtenir ou définir une valeur différente pour un paramètre.

## <a name="views-and-functions"></a>Vues et fonctions

Affichez et gérez le Magasin des requêtes à l’aide des fonctions et vues suivantes. Toute personne du [select privilege public role](howto-create-users.md#create-additional-admin-users) peut utiliser ces vues pour afficher les données du magasin de données des requêtes. Ces vues sont disponibles uniquement dans la base de données **mysql**.

Les requêtes sont normalisées en examinant leur structure après la suppression des littéraux et des constantes. Si deux requêtes sont identiques à l’exception des valeurs littérales, elle ont le même hachage.

### <a name="mysqlquery_store"></a>mysql.query_store

Cette vue retourne toutes les données du Magasin des requêtes. Il existe une ligne pour chaque ID de base de données distinct, ID d’utilisateur et ID de requête.

| **Nom** | **Type de données** | **IS_NULLABLE** | **Description** |
|---|---|---|---|
| `schema_name`| varchar(64) | Non | Nom du schéma |
| `query_id`| bigint(20) | Non| ID unique généré pour la requête spécifique, si la même requête s’exécute dans un schéma différent, un nouvel ID sera généré |
| `timestamp_id` | timestamp| Non| Horodatage dans lequel la requête est exécutée. Celui-ci repose sur la configuration de query_store_interval|
| `query_digest_text`| longtext| Non| Texte de requête normalisé après la suppression de tous les littéraux|
| `query_sample_text` | longtext| Non| Première apparition de la requête réelle avec des littéraux|
| `query_digest_truncated` | bit| YES| Indique si le texte de requête a été tronqué. La valeur sera Oui si la requête est supérieure à 1 Ko|
| `execution_count` | bigint(20)| Non| Nombre d’exécutions de la requête pour cet ID d’horodatage/pendant la période d’intervalle configurée|
| `warning_count` | bigint(20)| Non| Nombre d’avertissements générés par cette requête au cours de l’intervalle|
| `error_count` | bigint(20)| Non| Nombre d’erreurs générées par cette requête au cours de l’intervalle|
| `sum_timer_wait` | double| YES| Durée d’exécution totale de cette requête au cours de l’intervalle|
| `avg_timer_wait` | double| YES| Durée d’exécution moyenne de cette requête au cours de l’intervalle|
| `min_timer_wait` | double| YES| Durée d’exécution minimale pour cette requête|
| `max_timer_wait` | double| YES| Durée d’exécution maximale|
| `sum_lock_time` | bigint(20)| Non| Temps total passé pour tous les verrous pour cette exécution de requête au cours de cette période|
| `sum_rows_affected` | bigint(20)| Non| Nombre de lignes affectées|
| `sum_rows_sent` | bigint(20)| Non| Nombre de lignes envoyées au client|
| `sum_rows_examined` | bigint(20)| Non| Nombre de lignes examinées|
| `sum_select_full_join` | bigint(20)| Non| Nombre de jointures complètes|
| `sum_select_scan` | bigint(20)| Non| Nombre d’analyses sélectionnées |
| `sum_sort_rows` | bigint(20)| Non| Nombre de lignes triées|
| `sum_no_index_used` | bigint(20)| Non| Nombre de fois que la requête n’a pas utilisé tous les index|
| `sum_no_good_index_used` | bigint(20)| Non| Nombre de fois que le moteur d’exécution de la requête n’a pas utilisé les index appropriés|
| `sum_created_tmp_tables` | bigint(20)| Non| Nombre total de tables temporaires créées|
| `sum_created_tmp_disk_tables` | bigint(20)| Non| Nombre total de tables temporaires créées dans le disque (génère des E/S)|
| `first_seen` | timestamp| Non| La première occurrence (UTC) de la requête au cours de la période d’agrégation|
| `last_seen` | timestamp| Non| La dernière occurrence (UTC) de la requête au cours de cette période d’agrégation|

### <a name="mysqlquery_store_wait_stats"></a>mysql.query_store_wait_stats

Cette vue retourne les données des événements d’attente du Magasin des requêtes. Il existe une ligne pour chaque ID de base de données, ID d’utilisateur, ID de requête et événement distinct.

| **Nom**| **Type de données** | **IS_NULLABLE** | **Description** |
|---|---|---|---|
| `interval_start` | timestamp | Non| Début de l’intervalle (incrément de 15 minutes)|
| `interval_end` | timestamp | Non| Fin de l’intervalle (incrément de 15 minutes)|
| `query_id` | bigint(20) | Non| ID unique généré sur la requête normalisée (à partir du magasin de données des requêtes)|
| `query_digest_id` | varchar(32) | Non| Texte de requête normalisé après suppression de tous les littéraux (à partir du magasin de données des requêtes) |
| `query_digest_text` | longtext | Non| Première apparition de la requête réelle avec des littéraux (à partir du magasin de données des requêtes) |
| `event_type` | varchar(32) | Non| Catégorie de l’événement d’attente |
| `event_name` | varchar(128) | Non| Nom de l’événement d’attente |
| `count_star` | bigint(20) | Non| Nombre d’événements d’attente échantillonnés au cours de l’intervalle pour la requête |
| `sum_timer_wait_ms` | double | Non| Temps d’attente total (en millisecondes) de cette requête pendant l’intervalle |

### <a name="functions"></a>Fonctions

| **Nom**| **Description** |
|---|---|
| `mysql.az_purge_querystore_data(TIMESTAMP)` | Supprime définitivement toutes les données du magasin de données des requêtes avant l’horodatage donné |
| `mysql.az_procedure_purge_querystore_event(TIMESTAMP)` | Supprime définitivement toutes les données des événements d’attente du magasin de données des requêtes avant l’horodatage donné |
| `mysql.az_procedure_purge_recommendation(TIMESTAMP)` | Supprime les recommandations dont le délai d’expiration est antérieur à l’horodatage donné |

## <a name="limitations-and-known-issues"></a>Limitations et problèmes connus

- Si le paramètre `default_transaction_read_only` est activé sur un serveur MariaDB, le magasin de données des requêtes ne peut pas capturer les données.
- La fonctionnalité Magasin de données des requêtes peut être interrompue si elle rencontre des requêtes Unicode longues (\>= 6 000 octets).
- La période de rétention pour les statistiques d’attente est de 24 heures.
- Les statistiques d’attente utilisent des exemples pour capturer une fraction d’événements. La fréquence peut être modifiée à l’aide du paramètre `query_store_wait_sampling_frequency`.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’[analyse des performances des requêtes](concepts-query-performance-insight.md)
