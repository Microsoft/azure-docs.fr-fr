---
title: Journaux d’audit - Azure Database pour MySQL
description: Décrit les journaux d’audit disponibles dans Azure Database pour MySQL et les paramètres disponibles pour l’activation des niveaux de journalisation.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 6/24/2020
ms.openlocfilehash: aac8e6e04b8bac7a1d27fefc780fb9fadb3122dc
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94534175"
---
# <a name="audit-logs-in-azure-database-for-mysql"></a>Journaux d’audit dans Azure Database pour MySQL

Dans Azure Database pour MySQL, le journal d’audit est disponible pour les utilisateurs. Le journal d’audit peut être utilisé pour suivre l’activité au niveau de la base de données et est couramment utilisé à des fins de conformité.

## <a name="configure-audit-logging"></a>Configurer l’enregistrement d’audit

>[!IMPORTANT]
> Il est recommandé de ne consigner que les types d’événements et les utilisateurs requis à des fins d’audit pour garantir que les performances de votre serveur ne sont pas fortement affectées.

Par défaut, le journal d’audit est désactivé. Pour l’activer, affectez la valeur ON à `audit_log_enabled`.

Les autres paramètres que vous pouvez ajuster incluent :

- `audit_log_events` : contrôle les événements à enregistrer. Consultez le tableau ci-dessous pour des événements d’audit spécifiques.
- `audit_log_include_users`: Utilisateurs MySQL à inclure pour la journalisation. La valeur par défaut de ce paramètre est vide, ce qui inclut tous les utilisateurs pour la journalisation. Il a une plus grande priorité que `audit_log_exclude_users`. La longueur maximale du paramètre est de 512 caractères.
- `audit_log_exclude_users`: utilisateurs MySQL à exclure de la journalisation. La longueur maximale du paramètre est de 512 caractères.

> [!NOTE]
> `audit_log_include_users` a une priorité plus élevée que `audit_log_exclude_users`. Par exemple, si `audit_log_include_users` = `demouser` et `audit_log_exclude_users` = `demouser`, l’utilisateur sera inclus dans les journaux d’audit, car `audit_log_include_users` a une priorité plus élevée.

| **Event** | **Description** |
|---|---|
| `CONNECTION` | - Lancement de la connexion (réussite ou échec) <br> - Réauthentification d’utilisateur avec un autre utilisateur/mot de passe en cours de session <br> - Arrêt de la connexion |
| `DML_SELECT`| Requêtes SELECT |
| `DML_NONSELECT` | Requêtes INSERT/DELETE/UPDATE |
| `DML` | DML = DML_SELECT + DML_NONSELECT |
| `DDL` | Requêtes telles que « DROP DATABASE » |
| `DCL` | Requêtes telles que « GRANT PERMISSION » |
| `ADMIN` | Requêtes telles que « SHOW STATUS » |
| `GENERAL` | Tout dans DML_SELECT, DML_NONSELECT, DML, DDL, DCL et ADMIN |
| `TABLE_ACCESS` | - Uniquement disponible pour MySQL 5.7 <br> - Instructions de lecture de table, telles que SELECT ou INSERT INTO... SELECT <br> - Instructions de suppression de table, telles que DELETE ou TRUNCATE TABLE <br> - Instructions d’insertion de table, telles que INSERT ou REPLACE <br> - Instructions de mise à jour de table, telles que UPDATE |

## <a name="access-audit-logs"></a>Accéder aux journaux d’audit

Les journaux d’audit sont intégrés aux journaux de diagnostic Azure Monitor. Une fois que vous avez activé les journaux d’audit sur votre serveur MySQL, vous pouvez les transmettre vers des journaux Azure Monitor, des Event Hubs ou Stockage Azure. Pour en savoir plus sur l’activation des journaux de diagnostic dans le portail Azure, consultez l’[article sur le portail de journal d’audit](howto-configure-audit-logs-portal.md#set-up-diagnostic-logs).

## <a name="diagnostic-logs-schemas"></a>Schémas des journaux de diagnostic

Les sections suivantes décrivent la sortie des journaux d’audit MySQL en fonction du type d’événement. En fonction de la méthode de sortie, les champs inclus et l’ordre dans lequel ils apparaissent peuvent varier.

### <a name="connection"></a>Connexion

| **Propriété** | **Description** |
|---|---|
| `TenantId` | Votre ID d’abonné |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Horodatage du moment où le journal a été enregistré en UTC |
| `Type` | Type de journal. Toujours `AzureDiagnostics` |
| `SubscriptionId` | GUID de l’abonnement auquel appartient le serveur |
| `ResourceGroup` | Nom du groupe de ressources auquel le serveur appartient |
| `ResourceProvider` | Nom du fournisseur de ressources. Toujours `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de ressource |
| `Resource` | Nom du serveur |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Nom du serveur |
| `event_class_s` | `connection_log` |
| `event_subclass_s` | `CONNECT`, `DISCONNECT`, `CHANGE USER` (uniquement disponible pour MySQL 5.7) |
| `connection_id_d` | ID de connexion unique généré par MySQL |
| `host_s` | Vide |
| `ip_s` | Adresse IP du client qui se connecte à MySQL |
| `user_s` | Nom de l’utilisateur qui exécute la requête |
| `db_s` | Nom de la base de données connectée |
| `\_ResourceId` | URI de ressource |

### <a name="general"></a>Général

Le schéma ci-dessous s’applique aux types d’événements GENERAL, DML_SELECT, DML_NONSELECT, DML, DDL, DCL et ADMIN.

> [!NOTE]
> Pour `sql_text`, le journal est tronqué s’il dépasse 2 048 caractères.

| **Propriété** | **Description** |
|---|---|
| `TenantId` | Votre ID d’abonné |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Horodatage du moment où le journal a été enregistré en UTC |
| `Type` | Type de journal. Toujours `AzureDiagnostics` |
| `SubscriptionId` | GUID de l’abonnement auquel appartient le serveur |
| `ResourceGroup` | Nom du groupe de ressources auquel le serveur appartient |
| `ResourceProvider` | Nom du fournisseur de ressources. Toujours `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de ressource |
| `Resource` | Nom du serveur |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Nom du serveur |
| `event_class_s` | `general_log` |
| `event_subclass_s` | `LOG`, `ERROR`, `RESULT` (uniquement disponible pour MySQL 5.6) |
| `event_time` | Heure de début de la requête dans l’horodatage UTC |
| `error_code_d` | Code d’erreur en cas d’échec de la requête. `0` signifie aucune erreur |
| `thread_id_d` | ID du thread qui a exécuté la requête |
| `host_s` | Vide |
| `ip_s` | Adresse IP du client qui se connecte à MySQL |
| `user_s` | Nom de l’utilisateur qui exécute la requête |
| `sql_text_s` | Texte de la requête complète |
| `\_ResourceId` | URI de ressource |

### <a name="table-access"></a>Accès à la table

> [!NOTE]
> Les journaux d’accès à la table sont uniquement des sorties pour MySQL 5.7.<br>Pour `sql_text`, le journal est tronqué s’il dépasse 2 048 caractères.

| **Propriété** | **Description** |
|---|---|
| `TenantId` | Votre ID d’abonné |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Horodatage du moment où le journal a été enregistré en UTC |
| `Type` | Type de journal. Toujours `AzureDiagnostics` |
| `SubscriptionId` | GUID de l’abonnement auquel appartient le serveur |
| `ResourceGroup` | Nom du groupe de ressources auquel le serveur appartient |
| `ResourceProvider` | Nom du fournisseur de ressources. Toujours `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | URI de ressource |
| `Resource` | Nom du serveur |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Nom du serveur |
| `event_class_s` | `table_access_log` |
| `event_subclass_s` | `READ`, `INSERT`, `UPDATE` ou `DELETE` |
| `connection_id_d` | ID de connexion unique généré par MySQL |
| `db_s` | Nom de la base de données sollicitée |
| `table_s` | Nom de la table sollicitée |
| `sql_text_s` | Texte de la requête complète |
| `\_ResourceId` | URI de ressource |

## <a name="analyze-logs-in-azure-monitor-logs"></a>Analyser les journaux dans Azure Monitor

Une fois vos journaux d’audit dirigés vers les journaux Azure Monitor par le biais des journaux de diagnostic, vous pouvez effectuer une analyse plus poussée de vos événements audités. Voici quelques exemples de requêtes pour vous aider à commencer. Veillez à mettre à jour les exemples ci-dessous avec le nom de votre serveur.

- Lister les événements généraux (GENERAL) sur un serveur spécifique

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs' and event_class_s == "general_log"
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last 
    ```

- Lister les événements de connexion (CONNECTION) sur un serveur spécifique

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs' and event_class_s == "connection_log"
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ```

- Récapituler les événements audités sur un serveur spécifique

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | summarize count() by event_class_s, event_subclass_s, user_s, ip_s
    ```

- Représenter sous forme de graphe la distribution des types d’événements d’audit sur un serveur spécifique :

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart 
    ```

- Lister les événements audités de tous les serveurs MySQL sur lesquels les journaux de diagnostic sont activés pour les journaux d’audit :

    ```kusto
    AzureDiagnostics
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ``` 

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour configurer les journaux d’audit dans le portail Azure](howto-configure-audit-logs-portal.md)