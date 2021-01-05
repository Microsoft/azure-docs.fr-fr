---
title: Transférer des données de travaux Azure Automation aux journaux d’activité Azure Monitor
description: Cet article explique comment envoyer l’état d’un travail, ainsi que les flux de travaux d’un runbook, aux journaux Azure Monitor.
services: automation
ms.subservice: process-automation
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 2e7e798967541748b5572994d48cb5bdf7474cb1
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182867"
---
# <a name="forward-azure-automation-job-data-to-azure-monitor-logs"></a>Transférer des données de travaux Azure Automation aux journaux d’activité Azure Monitor

Azure Automation peut envoyer un état de tâche du Runbook et des flux de tâches à votre espace de travail Log Analytics. Ce processus n’implique aucune liaison d’espace de travail et est complètement indépendant. Les journaux d’activité de travail et les flux de travail sont visibles dans le portail Azure, ou avec PowerShell, pour des travaux individuels. Cela vous permet d’effectuer des enquêtes simples. Avec les journaux Azure Monitor, vous pouvez désormais :

* Obtenez des insights sur l'état de vos travaux Automation.
* Déclencher un e-mail ou une alerte en fonction du statut de votre travail de runbook (par exemple, échec ou état suspendu).
* Écrire des requêtes avancées sur plusieurs flux de travaux.
* Mettre en corrélation des travaux sur différents comptes Automation.
* Utiliser des vues et des requêtes de recherche personnalisées pour visualiser les résultats de votre runbook, l’état du travail de runbook et d’autres indicateurs ou mesures clés associées.

## <a name="prerequisites"></a>Prérequis

Pour commencer à envoyer vos journaux d’activité Automation à des journaux Azure Monitor, vous devez disposer des éléments suivants :

* La version la plus récente d’[Azure PowerShell](/powershell/azure/).

* Un espace de travail Log Analytics et son ID de ressource. Pour plus d’informations, consultez [Prise en main des journaux Azure Monitor](../azure-monitor/overview.md).

* L’ID de ressource de votre compte Azure Automation.

## <a name="how-to-find-resource-ids"></a>Comment trouver les ID de ressource

1. Utilisez la commande suivante pour rechercher l’ID de ressource de votre compte Azure Automation :

    ```powershell-interactive
    # Find the ResourceId for the Automation account
    Get-AzResource -ResourceType "Microsoft.Automation/automationAccounts"
    ```

2. Copiez la valeur de **ResourceID**.

3. Utilisez la commande suivante pour rechercher l’ID de ressource de votre espace de travail Log Analytics :

    ```powershell-interactive
    # Find the ResourceId for the Log Analytics workspace
    Get-AzResource -ResourceType "Microsoft.OperationalInsights/workspaces"
    ```

4. Copiez la valeur de **ResourceID**.

Pour retourner les résultats d’un groupe de ressources spécifique, incluez le paramètre `-ResourceGroupName`. Pour plus d’informations, voir [Get-AzResource](/powershell/module/az.resources/get-azresource).

Si vous avez plusieurs comptes ou espaces de travail Automation dans la sortie des commandes précédentes, vous pouvez trouver le nom et d’autres propriétés associées qui font partie de l’ID de ressource complet de votre compte Automation en procédant de la façon suivante :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur le portail Azure, sélectionnez votre compte Automation dans la page **Comptes Automation**.
1. Dans la page du compte Automation sélectionné, sous **Paramètres du compte**, sélectionnez **Propriétés**.
1. Dans la page **Propriétés**, notez les détails indiqués ci-dessous.

    ![Propriétés du compte Automation](media/automation-manage-send-joblogs-log-analytics/automation-account-properties.png).

## <a name="configure-diagnostic-settings"></a>Configurer les paramètres de diagnostic

Les paramètres de diagnostic d’Automation prennent en charge le transfert des journaux de plateforme et des données de métriques suivants :

* JobLogs
* JobStreams
* DSCNodeStatus
* Métriques : Nombre total de travaux, Nombre total d’exécutions de déploiement de mises à jour de machines, Nombre total d’exécutions de déploiement de mises à jour

Pour commencer à envoyer vos journaux Automation aux journaux d’activité d’Azure Monitor, consultez [Créer des paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md) pour comprendre la fonction et les méthodes disponibles pour configurer les paramètres de diagnostic afin d’envoyer les journaux de la plateforme.

## <a name="azure-monitor-log-records"></a>Enregistrements de journaux Azure Monitor

Les diagnostics d’Azure Automation créent deux types d’enregistrements dans les journaux Azure Monitor, marqué comme `AzureDiagnostics`. Les tableaux des sections suivantes sont des exemples d’enregistrements générés par Azure Automation et de types de données s'affichant dans les résultats de recherche dans les journaux.

### <a name="job-logs"></a>Journaux d’activité de tâches

| Propriété | Description |
| --- | --- |
| TimeGenerated |Date et heure d’exécution du travail du runbook. |
| RunbookName_s |Nom du runbook. |
| Caller_s |Appelant ayant initié l’opération. Les valeurs possibles sont une adresse de messagerie ou un système pour les travaux planifiés. |
| Tenant_g | GUID identifiant le locataire pour l’appelant. |
| JobId_g |GUID identifiant le travail du runbook. |
| ResultType |L’état du travail du runbook. Les valeurs possibles sont les suivantes :<br>- Nouveau<br>Créé<br>Démarré<br>Arrêté<br>Interrompu<br>Échec<br>- Terminé |
| Category | Classification du type de données. Pour Automation, la valeur est JobLogs. |
| NomOpération | Type d’opération exécutée dans Azure. Pour Automation, la valeur est Job. |
| Ressource | Nom du compte Automation |
| SourceSystem | Système utilisé par les journaux d'activité Azure Monitor pour collecter les données. La valeur est toujours Azure pour les diagnostics Azure. |
| resultDescription |État résultant du travail du runbook. Les valeurs possibles sont les suivantes :<br>- Job is started<br>- Job Failed<br>- Job Completed |
| CorrelationId |ID de corrélation du travail du runbook. |
| ResourceId |ID de ressource de compte Azure Automation du runbook. |
| SubscriptionId | GUID d’abonnement Azure pour le compte Automation. |
| ResourceGroup | Nom du groupe de ressources du compte Automation. |
| ResourceProvider | Fournisseur de ressources. La valeur est MICROSOFT.AUTOMATION. |
| ResourceType | Type de ressource. La valeur est AUTOMATIONACCOUNTS. |

### <a name="job-streams"></a>Flux de tâches
| Propriété | Description |
| --- | --- |
| TimeGenerated |Date et heure d’exécution du travail du runbook. |
| RunbookName_s |Nom du runbook. |
| Caller_s |Appelant ayant initié l’opération. Les valeurs possibles sont une adresse de messagerie ou un système pour les travaux planifiés. |
| StreamType_s |Type de flux de travail. Les valeurs possibles sont les suivantes :<br>progress<br>Sortie<br>Avertissement<br>error<br>DEBUG<br>- Verbose |
| Tenant_g | GUID identifiant le locataire pour l’appelant. |
| JobId_g |GUID identifiant le travail du runbook. |
| ResultType |L’état du travail du runbook. Les valeurs possibles sont les suivantes :<br>- In Progress |
| Category | Classification du type de données. Pour Automation, la valeur est JobStreams. |
| NomOpération | Type d’opération exécutée dans Azure. Pour Automation, la valeur est Job. |
| Ressource | Nom du compte Automation |
| SourceSystem | Système utilisé par les journaux d'activité Azure Monitor pour collecter les données. La valeur est toujours Azure pour les diagnostics Azure. |
| resultDescription |Description incluant le flux de sortie du runbook. |
| CorrelationId |ID de corrélation du travail du runbook. |
| ResourceId |ID de ressource de compte Azure Automation du runbook. |
| SubscriptionId | GUID d’abonnement Azure pour le compte Automation. |
| ResourceGroup | Nom du groupe de ressources du compte Automation. |
| ResourceProvider | Fournisseur de ressources. La valeur est MICROSOFT.AUTOMATION. |
| ResourceType | Type de ressource. La valeur est AUTOMATIONACCOUNTS. |

## <a name="view-automation-logs-in-azure-monitor-logs"></a>Afficher les journaux Automation dans les journaux Azure Monitor

Après avoir commencé à envoyer vos flux de travail journaux Automation aux journaux d’activité d’Azure Monitor, nous allons voir comment les utiliser dans les journaux d’activité d’Azure Monitor.

Pour afficher les journaux d’activité, exécutez la requête suivante :`AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION"`

### <a name="send-an-email-when-a-runbook-job-fails-or-suspends"></a>Envoyer un e-mail en cas d’échec ou de suspension d’un travail de runbook

Les étapes suivantes montrent comment configurer des alertes dans Azure Monitor pour être averti en cas de problème avec un travail de runbook.

Pour créer une règle d’alerte, commencez par créer une recherche de journal pour les enregistrements de travaux de runbook qui doivent appeler l’alerte. Cliquez sur le bouton **Alerte** pour créer et configurer la règle d’alerte.

1. Dans la page de présentation de l’espace de travail Log Analytics, cliquez sur **Afficher les journaux**.

2. Créez une requête de recherche dans les journaux pour votre alerte en tapant la recherche suivante dans le champ de requête : `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended")`<br><br>Vous pouvez également effectuer un regroupement par nom de runbook en utilisant : `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended") | summarize AggregatedValue = count() by RunbookName_s`

   Si vous avez configuré des journaux d’activité dans votre espace de travail sur plusieurs abonnements ou comptes Automation, vous pouvez également regrouper vos alertes par abonnement ou par compte Automation. Le nom du compte Automation se trouve dans le champ `Resource` de la recherche de `JobLogs`.

3. Pour ouvrir l’écran **Créer une règle**, cliquez sur **Nouvelle règle d’alerte** en haut de la page. Pour plus d’informations sur les options de configuration de l’alerte, voir [Alertes de journaux dans Azure](../azure-monitor/platform/alerts-unified-log.md).

### <a name="find-all-jobs-that-have-completed-with-errors"></a>Rechercher tous les travaux qui ont rencontré des erreurs

En plus des alertes concernant les échecs, vous pouvez déterminer lorsqu’une tâche de runbook comporte une erreur sans fin d’exécution. Dans ce cas, PowerShell génère un flux d’erreur, mais les erreurs sans fin d’exécution ne provoquent la suspension ou l’échec du travail.

1. Dans votre espace de travail Log Analytics, cliquez sur **Journaux**.

2. Dans le champ de requête de recherche, entrez `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and StreamType_s == "Error" | summarize AggregatedValue = count() by JobId_g`.

3. Cliquez sur le bouton **Rechercher**.

### <a name="view-job-streams-for-a-job"></a>Afficher les flux de travail pour un travail

Lorsque vous déboguez un travail, vous pouvez également examiner les flux de travaux. La requête ci-dessous montre tous les flux pour un seul travail avec le GUID `2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0` :

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and JobId_g == "2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0"
| sort by TimeGenerated asc
| project ResultDescription
```

### <a name="view-historical-job-status"></a>Afficher l’état de travail historique

Vous pouvez enfin souhaiter visualiser l’historique de vos travaux dans le temps. Vous pouvez utiliser cette requête pour rechercher l’état de vos travaux au fil du temps.

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and ResultType != "started"
| summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h)
```

![Graphique de l’état de la tâche historique de Log Analytics](media/automation-manage-send-joblogs-log-analytics/historical-job-status-chart.png)

### <a name="filter-job-status-output-converted-into-a-json-object"></a>Filtrer la sortie de l’état du travail convertie en objet JSON

Récemment, nous avons modifié le comportement relatif à la façon dont les données du journal Automation sont écrites dans la table `AzureDiagnostics` du service Log Analytics, où elles ne décomposent plus les propriétés JSON en champs distincts. Si vous avez configuré votre runbook pour qu’il formate des objets dans le flux de sortie au format JSON en tant que colonnes distinctes, vous devez reconfigurer vos requêtes pour analyser ce champ dans un objet JSON afin d’accéder à ces propriétés. Pour ce faire, utilisez [parsejson](/azure/data-explorer/kusto/query/samples?pivots=#parsejson) pour accéder à un élément JSON spécifique dans un chemin d’accès connu.

Par exemple, un runbook formate la propriété *ResultDescription* dans le flux de sortie au format JSON avec plusieurs champs. Pour rechercher les travaux qui sont à l’état d’échec, comme spécifié dans un champ appelé **État**, utilisez cet exemple de requête pour rechercher la propriété *ResultDescription* dont l’état est **Échec** :

```kusto
AzureDiagnostics
| where Category == 'JobStreams'
| extend jsonResourceDescription = parse_json(ResultDescription)
| where jsonResourceDescription.Status == 'Failed'
```

![Format JSON du flux de la tâche historique de Log Analytics](media/automation-manage-send-joblogs-log-analytics/job-status-format-json.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour savoir comment construire des requêtes de recherche et consulter les journaux d’activité de travaux Automation avec les journaux Azure Monitor, consultez [Recherches dans les journaux d’activité dans les journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md).
* Pour savoir comment créer et récupérer la sortie et les messages d’erreur de runbooks, consultez [Superviser la sortie des runbooks](automation-runbook-output-and-messages.md).
* Pour plus d’informations sur l’exécution des runbooks, sur la supervision des travaux de runbook et sur d’autres détails techniques, consultez [Exécution d’un Runbook dans Azure Automation](automation-runbook-execution.md).
* Pour plus d’informations sur les journaux Azure Monitor et sur les sources de collecte de données, consultez la page [Collecter des données de stockage Azure dans les journaux Azure Monitor – Vue d’ensemble](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).
* Pour obtenir de l’aide sur la résolution des problèmes Log Analytics, consultez [Dépannage si Log Analytics ne collecte plus de données](../azure-monitor/platform/manage-cost-storage.md#troubleshooting-why-log-analytics-is-no-longer-collecting-data).