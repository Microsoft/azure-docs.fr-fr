---
title: Gérer et superviser l’importance de la charge de travail dans un pool SQL dédié
description: Découvrez comment gérer et superviser l’importance du niveau des demandes dans un pool SQL dédié pour Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: fb0a5fbf33b48521882646bf8fb5eb3fe5dacca6
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459212"
---
# <a name="manage-and-monitor-workload-importance-in-dedicated-sql-pool-for-azure-synapse-analytics"></a>Gérer et superviser l’importance des charges de travail dans un pool SQL dédié pour Azure Synapse Analytics

Gérez et supervisez l’importance du niveau des demandes de pool SQL dédié dans Azure Synapse à l’aide de vues de gestion dynamique (DMV) et de vues catalogue.

## <a name="monitor-importance"></a>Surveiller l’importance

Surveillez l’importance à l’aide de la nouvelle colonne d’importance dans la vue de gestion dynamique [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).
La requête de surveillance ci-dessous montre l’heure d’envoi et l’heure de début des requêtes. Passez en revue l’heure d’envoi et l’heure de début ainsi que l’importance pour voir son effet sur la planification.

```sql
SELECT s.login_name, r.status, r.importance, r.submit_time, r.start_time
  FROM sys.dm_pdw_exec_sessions s
  JOIN sys.dm_pdw_exec_requests r ON s.session_id = r.session_id
  WHERE r.resource_class is not null
ORDER BY r.start_time
```

Pour en savoir plus sur la planification des requêtes, utilisez les affichages catalogue.

## <a name="manage-importance-with-catalog-views"></a>Gérer l’importance avec les affichages catalogue

L’affichage catalogue sys.workload_management_workload_classifiers contient des informations sur les classifieurs. Pour exclure les classifieurs définis par le système qui mappent aux classes de ressources, exécutez le code suivant :

```sql
SELECT *
  FROM sys.workload_management_workload_classifiers
  WHERE classifier_id > 12
```

L’affichage catalogue [sys.workload_management_workload_classifier_details](/sql/relational-databases/system-catalog-views/sys-workload-management-workload-classifier-details-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) contient des informations sur les paramètres utilisés lors de la création du classifieur.  La requête ci-dessous montre que ExecReportsClassifier a été créé sur le paramètre ```membername``` pour des valeurs avec ExecutiveReports :

```sql
SELECT c.name,cd.classifier_type, classifier_value
  FROM sys.workload_management_workload_classifiers c
  JOIN sys.workload_management_workload_classifier_details cd
    ON cd.classifier_id = c.classifier_id
  WHERE c.name = 'ExecReportsClassifier'
```

![Résultats de la requête](./media/sql-data-warehouse-how-to-manage-and-monitor-workload-importance/wlm-query-results.png)

Pour simplifier la résolution des problèmes de classification, nous vous recommandons de supprimer les mappages de rôle de classe de ressources car vous créez des classifieurs de charge de travail. Le code ci-dessous retourne des appartenances existantes aux rôles de classe de ressources. Exécutez sp_droprolemember pour chaque ```membername``` retourné par la classe de ressource correspondante.
Voici un exemple de vérification de l’existence avant de déposer un classifieur de charge de travail :

```sql
IF EXISTS (SELECT 1 FROM sys.workload_management_workload_classifiers WHERE name = 'ExecReportsClassifier')
  DROP WORKLOAD CLASSIFIER ExecReportsClassifier;
GO
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la Classification, consultez [Classification de charge de travail](sql-data-warehouse-workload-classification.md).
- Pour plus d’informations sur l’importance, consultez [Importance de la charge de travail](sql-data-warehouse-workload-importance.md)

> [!div class="nextstepaction"]
> [Accédez à Configurer l’importance des charges de travail](sql-data-warehouse-how-to-configure-workload-importance.md)
