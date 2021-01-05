---
title: Journaux de requêtes à partir d’Azure Automation Start/Stop VMs during off-hours
description: Cet article explique comment interroger les données de journal générées par la solution Start/Stop VMs during off-hours à partir d'Azure Monitor.
services: automation
ms.subservice: process-automation
ms.date: 04/01/2020
ms.topic: conceptual
ms.openlocfilehash: de013b6ccd924f50ffe12fcba1285b121eece5f7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83827554"
---
# <a name="query-logs-from-startstop-vms-during-off-hours"></a>Interroger les journaux à partir de Start/Stop VMs during off-hours

Azure Automation transmet deux types d'enregistrements à l'espace de travail Log Analytics lié : les journaux d'activité des tâches et les flux de tâches. Cet article passe en revue les données disponibles pour les [requête](../azure-monitor/log-query/log-query-overview.md) dans Azure Monitor.

## <a name="job-logs"></a>Journaux d’activité de tâches

|Propriété | Description|
|----------|----------|
|Appelant |  Ce qui a initié l’opération. Les valeurs possibles sont une adresse de messagerie ou un système pour les travaux planifiés.|
|Category | Classification du type de données. Pour Automation, la valeur est JobLogs.|
|CorrelationId | GUID représentant l’ID de corrélation du travail du runbook.|
|JobId | GUID représentant l’ID du travail du runbook.|
|operationName | Spécifie le type d’opération exécutée dans Azure. Pour Automation, la valeur est Job.|
|resourceId | Spécifie le type de ressource dans Azure. Pour Automation, la valeur est le compte Automation associé au runbook.|
|ResourceGroup | Spécifie le nom du groupe de ressources de la tâche du runbook.|
|ResourceProvider | Spécifie le service qui fournit les ressources que vous pouvez déployer et gérer. Pour Automation, la valeur est Azure Automation.|
|ResourceType | Spécifie le type de ressource dans Azure. Pour Automation, la valeur est le compte Automation associé au runbook.|
|resultType | L’état du travail du runbook. Les valeurs possibles sont les suivantes :<br>Démarré<br>Arrêté<br>Interrompu<br>Échec<br>- Succeeded|
|resultDescription | Décrit l’état résultant du travail du runbook. Les valeurs possibles sont les suivantes :<br>- Job is started<br>- Job Failed<br>- Job Completed|
|RunbookName | Spécifie le nom du runbook.|
|SourceSystem | Spécifie le système source pour les données soumises. Pour Automation, la valeur est OpsManager.|
|StreamType | Spécifie le type d’événement. Les valeurs possibles sont les suivantes :<br>- Verbose<br>Sortie<br>error<br>Avertissement|
|SubscriptionId | Spécifie l’ID d’abonnement de la tâche.
|Temps | Date et heure d’exécution du travail du runbook.|

## <a name="job-streams"></a>Flux de tâches

|Propriété | Description|
|----------|----------|
|Appelant |  Ce qui a initié l’opération. Les valeurs possibles sont une adresse de messagerie ou un système pour les travaux planifiés.|
|Category | Classification du type de données. Pour Automation, la valeur est JobStreams.|
|JobId | GUID représentant l’ID du travail du runbook.|
|operationName | Spécifie le type d’opération exécutée dans Azure. Pour Automation, la valeur est Job.|
|ResourceGroup | Spécifie le nom du groupe de ressources de la tâche du runbook.|
|resourceId | Spécifie l’ID de ressource dans Azure. Pour Automation, la valeur est le compte Automation associé au runbook.|
|ResourceProvider | Spécifie le service qui fournit les ressources que vous pouvez déployer et gérer. Pour Automation, la valeur est Azure Automation.|
|ResourceType | Spécifie le type de ressource dans Azure. Pour Automation, la valeur est le compte Automation associé au runbook.|
|resultType | Résultat de la tâche du runbook au moment où l’événement a été généré. Une valeur possible est :<br>- InProgress|
|resultDescription | Inclut le flux de sortie du runbook.|
|RunbookName | Nom du runbook.|
|SourceSystem | Spécifie le système source pour les données soumises. Pour Automation, la valeur est OpsManager.|
|StreamType | Type de flux de travail. Les valeurs possibles sont les suivantes :<br>- Progress<br>Sortie<br>Avertissement<br>error<br>DEBUG<br>- Verbose|
|Temps | Date et heure d’exécution du travail du runbook.|

Quand vous effectuez une recherche de journal qui retourne des enregistrements de catégorie de **JobLogs** ou **JobStreams**, vous pouvez sélectionner la vue **JobLogs** ou **JobStreams** pour afficher un ensemble de vignettes résumant les informations retournées par la recherche.

## <a name="sample-log-searches"></a>Exemples de recherches dans les journaux

Le tableau suivant fournit des exemples de recherches de journaux pour les enregistrements de tâches collectés par Start/Stop VMs during off-hours.

|Requête | Description|
|----------|----------|
|Rechercher les tâches du runbook ScheduledStartStop_Parent terminées avec succès | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "ScheduledStartStop_Parent" ) <br>&#124;  where ( ResultType == "Completed" )  <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|Rechercher les tâches du runbook ScheduledStartStop_Parent qui ont échoué | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "ScheduledStartStop_Parent" ) <br>&#124;  where ( ResultType == "Failed" )  <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|Rechercher les tâches du runbook SequencedStartStop_Parent terminées avec succès | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "SequencedStartStop_Parent" ) <br>&#124;  where ( ResultType == "Completed" ) <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|
|Rechercher les tâches du runbook SequencedStartStop_Parent qui ont échoué | <code>search Category == "JobLogs" <br>&#124;  where ( RunbookName_s == "SequencedStartStop_Parent" ) <br>&#124;  where ( ResultType == "Failed" ) <br>&#124;  summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h) <br>&#124;  sort by TimeGenerated desc</code>|

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer la fonctionnalité, consultez [Configurer Start/Stop VMs during off-hours](automation-solution-vm-management-config.md).
* Pour plus d’informations sur les alertes de journal pendant le déploiement des fonctionnalités, consultez [créer des alertes de journal avec Azure Monitor](../azure-monitor/platform/alerts-log.md).
* Pour résoudre les erreurs liées à la fonctionnalité, consultez [Résoudre les problèmes liés à Start/Stop VMs during off-hours](troubleshoot/start-stop-vm.md).