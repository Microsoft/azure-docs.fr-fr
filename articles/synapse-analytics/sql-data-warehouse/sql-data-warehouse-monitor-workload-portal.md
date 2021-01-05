---
title: Superviser une charge de travail dans le portail Azure
description: Surveiller Synapse SQL à l’aide du portail Azure
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.openlocfilehash: 53c68f44273b14658bd4d95ce7cf3cf847c077d0
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93025950"
---
# <a name="monitor-workload---azure-portal"></a>Superviser une charge de travail dans le portail Azure

Cet article explique comment utiliser le portail Azure pour superviser votre charge de travail. Cela inclut la configuration des journaux Azure Monitor pour examiner les tendances de l’exécution des requêtes et de la charge de travail à l’aide de l’analytique des journaux pour [SQL Synapse](https://azure.microsoft.com/blog/workload-insights-with-sql-data-warehouse-delivered-through-azure-monitor-diagnostic-logs-pass/).

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Pool SQL : Nous collecterons les journaux pour un pool SQL. Si vous n’avez pas de pool SQL provisionné, consultez les instructions fournies dans [Créer un pool SQL](load-data-from-azure-blob-storage-using-polybase.md).

## <a name="create-a-log-analytics-workspace"></a>Créer un espace de travail Log Analytics

Accéder au panneau de navigation des espaces de travail Log Analytics et créer un espace de travail

![Espaces de travail Log Analytics](./media/sql-data-warehouse-monitor-workload-portal/log_analytics_workspaces.png)

![Capture d’écran montrant les espaces de travail Log Analytics où vous pouvez sélectionner Ajouter.](./media/sql-data-warehouse-monitor-workload-portal/add_analytics_workspace.png)

![Capture d’écran montrant l’espace de travail Log Analytics où vous pouvez entrer des valeurs.](./media/sql-data-warehouse-monitor-workload-portal/add_analytics_workspace_2.png)

Pour plus d’informations sur les espaces de travail, consultez cette [documentation](../../azure-monitor/learn/quick-create-workspace.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.jsond#create-a-workspace).

## <a name="turn-on-resource-logs"></a>Activer les journaux des ressources

Configurez les paramètres de diagnostic pour émettre des journaux à partir de votre pool SQL. Les journaux se composent de vues de données de télémétrie équivalentes aux vues de gestion dynamique (DMV) de résolution des problèmes de performances les plus fréquentes. À l’heure actuelle, les vues suivantes sont prises en charge :

- [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

![Activation des journaux de ressources](./media/sql-data-warehouse-monitor-workload-portal/enable_diagnostic_logs.png)

Les journaux peuvent être émis dans Stockage Azure, Stream Analytics ou Log Analytics. Pour ce tutoriel, sélectionnez Log Analytics.

![Spécifier des journaux](./media/sql-data-warehouse-monitor-workload-portal/specify_logs.png)

## <a name="run-queries-against-log-analytics"></a>Exécuter des requêtes sur Log Analytics

Accédez à votre espace de travail Log Analytics pour effectuer les étapes suivantes :

- Analyser les journaux à l’aide de requêtes et enregistrer les requêtes pour les réutiliser
- Enregistrer des requêtes pour les réutiliser
- Créer des alertes de journal
- Épingler des résultats de requête sur un tableau de bord

Pour plus d’informations sur les fonctionnalités des requêtes de journal, consultez cette [documentation](../../azure-monitor/log-query/query-language.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

![Éditeur d’espace de travail Log Analytics](./media/sql-data-warehouse-monitor-workload-portal/log_analytics_workspace_editor.png)

![Requêtes d’espace de travail Log Analytics](./media/sql-data-warehouse-monitor-workload-portal/log_analytics_workspace_queries.png)

## <a name="sample-log-queries"></a>Exemples de requêtes de journal

```Kusto
//List all queries
AzureDiagnostics
| where Category contains "ExecRequests"
| project TimeGenerated, StartTime_t, EndTime_t, Status_s, Command_s, ResourceClass_s, duration=datetime_diff('millisecond',EndTime_t, StartTime_t)
```

```Kusto
//Chart the most active resource classes
AzureDiagnostics
| where Category contains "ExecRequests"
| where Status_s == "Completed"
| summarize totalQueries = dcount(RequestId_s) by ResourceClass_s
| render barchart
```

```Kusto
//Count of all queued queries
AzureDiagnostics
| where Category contains "waits"
| where Type_s == "UserConcurrencyResourceType"
| summarize totalQueuedQueries = dcount(RequestId_s)
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez installé et configuré des journaux Azure monitor, [personnalisez les tableaux de bord Azure](../../azure-portal/azure-portal-dashboards.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) pour les partager avec votre équipe.
