---
title: Journaux des requêtes lentes – Azure Database pour MySQL – Serveur flexible
description: Décrit les journaux des requêtes lentes disponibles dans le Serveur flexible Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: dde9575a70ea80ad262bc01bb9d5d0015c803427
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94543015"
---
# <a name="slow-query-logs-in-azure-database-for-mysql-flexible-server-preview"></a>Journaux des requêtes lentes dans le serveur flexible Azure Database pour MySQL (préversion)

> [!IMPORTANT] 
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Dans le serveur flexible Azure Database pour MySQL, le journal des requêtes lentes disponible peut être configuré et consulté par les utilisateurs. Les journaux des requêtes lentes sont désactivés par défaut et peuvent être activés pour faciliter l’identification de goulots d’étranglement des performances lors de la résolution des problèmes.

Pour plus d’informations sur le journal des requêtes lentes MySQL, consultez la [section sur le journal des requêtes lentes](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) dans la documentation du moteur MySQL.

## <a name="configure-slow-query-logging"></a>Configurer la journalisation des requêtes lentes 
Par défaut, le journal des requêtes lentes est désactivé. Pour activer les journaux, définissez le paramètre de serveur `slow_query_log` sur *ACTIVÉ*. Vous pouvez configurer ce paramètre à l’aide du portail Azure ou d’Azure CLI <!-- add link to server parameter-->. 

Les autres paramètres que vous pouvez ajuster pour contrôler le comportement de la journalisation des requêtes lentes sont les suivants :

- **long_query_time** : consignez une requête si sa durée d’exécution est supérieure à `long_query_time` (en secondes). La valeur par défaut est 10 secondes.
- **log_slow_admin_statements** : détermine si les instructions administratives (par exemple, `ALTER_TABLE`, `ANALYZE_TABLE`) sont journalisées.
- **log_queries_not_using_indexes** : détermine si les requêtes qui n’utilisent pas d’index sont journalisées.
- **log_throttle_queries_not_using_indexes** : limite le nombre de requêtes hors index qui peuvent être écrites dans le journal des requêtes lentes. Ce paramètre prend effet lorsque `log_queries_not_using_indexes` est défini sur *ACTIVÉ*

> [!IMPORTANT]
> Si vos tables ne sont pas indexées, la définition des paramètres `log_queries_not_using_indexes` et `log_throttle_queries_not_using_indexes` sur **ACTIVÉ** peut affecter les performances de MySQL, car toutes les requêtes exécutées sur ces tables non indexées sont écrites dans le journal des requêtes lentes.

Consultez [la documentation MySQL consacrée au journal des requêtes lentes](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) pour obtenir une description complète des paramètres du journal des requêtes lentes.

## <a name="access-slow-query-logs"></a>Accéder aux journaux des requêtes lentes

Les journaux des requêtes lentes sont intégrés aux paramètres de diagnostic d’Azure Monitor. Une fois que vous avez activé les journaux des requêtes lentes sur votre serveur flexible MySQL, vous pouvez les transmettre aux journaux Azure Monitor, à Event Hubs ou au Stockage Azure. Pour en savoir plus sur les paramètres de diagnostic, consultez la [Documentation des journaux de diagnostic](../../azure-monitor/platform/platform-logs-overview.md). Pour en savoir plus sur l’activation des paramètres de diagnostic dans le portail Azure, consultez l’[article du portail sur le journal des requêtes lentes](how-to-configure-slow-query-logs-portal.md#set-up-diagnostics).

Le tableau suivant décrit la sortie du journal des requêtes lentes. En fonction de la méthode de sortie, les champs inclus et l’ordre dans lequel ils apparaissent peuvent varier.

| **Propriété** | **Description** |
|---|---|
| `TenantId` | Votre ID d’abonné |
| `SourceSystem` | `Azure` |
| `TimeGenerated` [UTC] | Horodatage du moment où le journal a été enregistré en UTC |
| `Type` | Type de journal. Toujours `AzureDiagnostics` |
| `SubscriptionId` | GUID de l’abonnement auquel appartient le serveur |
| `ResourceGroup` | Nom du groupe de ressources auquel le serveur appartient |
| `ResourceProvider` | Nom du fournisseur de ressources. Toujours `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de ressource |
| `Resource` | Nom du serveur |
| `Category` | `MySqlSlowLogs` |
| `OperationName` | `LogEvent` |
| `Logical_server_name_s` | Nom du serveur |
| `start_time_t` [UTC] | Heure de début de la requête |
| `query_time_s` | Durée totale d’exécution de la requête en secondes |
| `lock_time_s` | Durée totale en secondes pendant laquelle la requête a été verrouillée |
| `user_host_s` | Nom d’utilisateur |
| `rows_sent_s` | Nombre de lignes envoyées |
| `rows_examined_s` | Nombre de lignes examinées |
| `last_insert_id_s` | [last_insert_id](https://dev.mysql.com/doc/refman/5.7/en/information-functions.html#function_last-insert-id) |
| `insert_id_s` | ID de l’insertion |
| `sql_text_s` | Requête complète |
| `server_id_s` | ID du serveur |
| `thread_id_s` | ID du thread |
| `\_ResourceId` | URI de ressource |

> [!Note]
> Pour `sql_text_s`, le journal est tronqué s’il dépasse 2 048 caractères.

## <a name="analyze-logs-in-azure-monitor-logs"></a>Analyser les journaux dans Azure Monitor

Une fois vos journaux des requêtes lentes canalisés vers des journaux Azure Monitor via des journaux de diagnostic, vous pouvez effectuer une analyse plus poussée de vos requêtes lentes. Voici quelques exemples de requêtes pour vous aider à commencer. Veillez à mettre à jour les exemples ci-dessous avec le nom de votre serveur.

- Requêtes de plus de 10 secondes sur un serveur particulier

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```

- Lister les 5 requêtes les plus longues sur un serveur particulier

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | order by query_time_d desc
    | take 5
    ```

- Résumer les requêtes lentes par temps de requête minimal, maximal, moyen et d’écart type sur un serveur particulier

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count(), min(query_time_d), max(query_time_d), avg(query_time_d), stdev(query_time_d), percentile(query_time_d, 95) by LogicalServerName_s
    ```

- Représenter sous forme de graphique la distribution de requête lente sur un serveur particulier

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart
    ```

- Afficher les requêtes de plus de 10 secondes sur tous les serveurs MySQL avec les journaux de diagnostic activés

    ```Kusto
    AzureDiagnostics
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```    
    
## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [journaux d'audit](concepts-audit-logs.md)
- Configurer les journaux des requêtes lentes à partir du [portail Azure](how-to-configure-slow-query-logs-portal.md)
<!-- - [How to configure slow query logs from the Azure CLI](howto-configure-server-logs-in-cli.md). -->