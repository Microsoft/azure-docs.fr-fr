---
title: Intégration aux journaux Azure Monitor
description: Cet article explique comment envoyer des données de rapport DSC (Desired State Configuration) entre Azure Automation State Configuration et les journaux Azure Monitor.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.date: 11/06/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: f66c710901b129cb6b138fff126e67615c790714
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183666"
---
# <a name="integrate-with-azure-monitor-logs"></a>Intégration aux journaux Azure Monitor

Azure Automation State Configuration conserve les données d’état des nœuds pendant 30 jours. Vous pouvez envoyer les données d’état des nœuds à votre espace de travail Log Analytics si vous préférez conserver ces données plus longtemps. L’état de conformité est visible dans le portail Azure, ou avec PowerShell, pour les nœuds et les ressources DSC individuelles dans les configurations de nœud. 

Les journaux Azure Monitor offrent une plus grande visibilité opérationnelle sur vos données Automation State Configuration et peuvent vous aider à traiter les incidents plus rapidement. Avec les journaux Azure Monitor, vous pouvez :

- Obtenir des informations de conformité pour les nœuds gérés et les ressources individuelles.
- Déclencher un e-mail ou une alerte en fonction de l’état de conformité.
- Écrire des requêtes avancées dans vos nœuds gérés.
- Mettre en corrélation l’état de conformité dans les comptes Automation.
- Utiliser des vues et des requêtes de recherche personnalisées pour visualiser les résultats de votre runbook, l’état du travail de runbook et d’autres indicateurs ou mesures clés associées.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Prérequis

Pour commencer à envoyer vos rapports Automation State Configuration vers les journaux Azure Monitor, vous devez disposer des éléments suivants :

- Version de novembre 2016 ou une version ultérieure d’[Azure PowerShell](/powershell/azure/) (v2.3.0).
- Un compte Azure Automation. Pour plus d’informations, consultez [Présentation d’Azure Automation](automation-intro.md).
- Un espace de travail Log Analytics avec une offre de service Automation & Control. Pour plus d’informations, voir [Bien démarrer avec Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md).
- Au moins un nœud Azure Automation State Configuration. Pour plus d’informations, consultez [Intégration des machines pour la gestion avec Azure Automation State Configuration](automation-dsc-onboarding.md).
- Module [xDscDiagnostics](https://www.powershellgallery.com/packages/xDscDiagnostics/2.7.0.0) , version 2.7.0.0 ou ultérieure. Pour les étapes d’installation, consultez [Résoudre les problèmes de Configuration d’état souhaité d’Azure Automation](./troubleshoot/desired-state-configuration.md).

## <a name="set-up-integration-with-azure-monitor-logs"></a>Configurer l’intégration aux journaux d’Azure Monitor

Pour commencer l’importation de données d’Azure Automation State Configuration vers les journaux Azure Monitor, effectuez les étapes suivantes :

1. Dans PowerShell, connectez-vous à votre compte Azure. Consultez [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps).
1. Récupérez l’ID de ressource de votre compte Automation en exécutant l’applet de commande PowerShell suivante. Si vous avez plusieurs comptes Automation, choisissez l’ID de ressource pour le compte que vous voulez configurer.

   ```powershell
   # Find the ResourceId for the Automation account
   Get-AzResource -ResourceType 'Microsoft.Automation/automationAccounts'
   ```

1. Obtenez l’ID de ressource de votre espace de travail Log Analytics en exécutant l’applet de commande PowerShell suivante. Si vous avez plusieurs espaces de travail, choisissez l’ID de ressource de l’espace de travail que vous voulez configurer.

   ```powershell
   # Find the ResourceId for the Log Analytics workspace
   Get-AzResource -ResourceType 'Microsoft.OperationalInsights/workspaces'
   ```

1. Exécutez la cmdlet PowerShell suivante, en remplaçant `<AutomationResourceId>` et `<WorkspaceResourceId>` par les valeurs `ResourceId` de chacune des étapes précédentes.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId <AutomationResourceId> -WorkspaceId <WorkspaceResourceId> -Enabled $true -Category 'DscNodeStatus'
   ```

1. Si vous voulez arrêter l’importation de données d’Azure Automation State Configuration dans les journaux Azure Monitor, exécutez l’applet de commande PowerShell suivante.

   ```powershell
   Set-AzDiagnosticSetting -ResourceId <AutomationResourceId> -WorkspaceId <WorkspaceResourceId> -Enabled $false -Category 'DscNodeStatus'
   ```

## <a name="view-the-state-configuration-logs"></a>Afficher les journaux d’activité State Configuration

Après avoir configuré l’intégration aux journaux Azure Monitor pour vos données Automation State Configuration, vous pouvez les visualiser en sélectionnant **Journaux** dans la section **Supervision** dans le volet gauche de la page Configuration de l’état (DSC).

![Journaux d’activité](media/automation-dsc-diagnostics/automation-dsc-logs-toc-item.png)

Le volet Recherche dans les journaux s’ouvre avec une région de requête étendue à votre ressource de compte Automation. Vous pouvez rechercher des opérations DSC dans les journaux State Configuration en recherchant dans les journaux d’Azure Monitor. Les enregistrements pour les opérations DSC sont stockés dans la table `AzureDiagnostics`. Par exemple, pour rechercher des nœuds qui ne sont pas conformes, tapez la requête suivante.

```AzureDiagnostics
| where Category == 'DscNodeStatus' 
| where OperationName contains 'DSCNodeStatusData'
| where ResultType != 'Compliant'
```

Détails du filtrage :

* Filtrez sur `DscNodeStatusData` pour renvoyer les opérations pour chaque nœud State Configuration.
* Filtrez sur `DscResourceStatusData` pour renvoyer les opérations pour chaque ressource DSC appelée dans la configuration de nœud appliquée à cette ressource. 
* Filtrez sur `DscResourceStatusData` pour renvoyer les informations d’erreur pour les ressources DSC ayant échoué.

Pour plus d’informations sur la construction des requêtes de journal pour rechercher des données, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

### <a name="send-an-email-when-a-state-configuration-compliance-check-fails"></a>Envoyer un e-mail si une vérification de la conformité State Configuration échoue

L’une des demandes majeures de nos clients est de pouvoir envoyer un e-mail ou un message texte lorsqu’une erreur survient avec une configuration DSC.

Pour créer une règle d’alerte, commencez par créer une recherche dans les journaux pour les enregistrements de rapport State Configuration qui doivent appeler l’alerte. Cliquez sur le bouton **Nouvelle règle d’alerte** pour créer et configurer la règle d’alerte.

1. Dans la page Vue d’ensemble de l’espace de travail Log Analytics, cliquez sur **Journaux**.
1. Créez une requête de recherche dans les journaux pour votre alerte en tapant la recherche suivante dans le champ de requête : `Type=AzureDiagnostics Category='DscNodeStatus' NodeName_s='DSCTEST1' OperationName='DscNodeStatusData' ResultType='Failed'`

   Si vous avez configuré des journaux d’activité dans votre espace de travail à partir de plusieurs abonnements ou comptes Automation, vous pouvez également regrouper vos alertes par abonnement ou par compte Automation. Dérivez le nom du compte Automation du champ `Resource` dans la recherche des enregistrements `DscNodeStatusData`.
1. Pour ouvrir l’écran **Créer une règle**, cliquez sur **Nouvelle règle d’alerte** en haut de la page. 

Pour plus d’informations sur les options de configuration de l’alerte, consultez [Créer une règle d’alerte](../azure-monitor/platform/alerts-metric.md).

### <a name="find-failed-dsc-resources-across-all-nodes"></a>Rechercher les ressources DSC ayant échoué dans tous les nœuds

Un des avantages de l’utilisation des journaux Azure Monitor est que vous pouvez rechercher les vérifications avortées dans les nœuds. Pour rechercher toutes les instances de ressources DSC qui ont échoué :

1. Dans la page Vue d’ensemble de l’espace de travail Log Analytics, cliquez sur **Journaux**.
1. Créez une requête de recherche dans les journaux pour votre alerte en tapant la recherche suivante dans le champ de requête : `Type=AzureDiagnostics Category='DscNodeStatus' OperationName='DscResourceStatusData' ResultType='Failed'`.

### <a name="view-historical-dsc-node-status"></a>Afficher l’état de nœud DSC historique

Pour visualiser l’historique de l’état de votre nœud DSC au fil du temps, vous pouvez utiliser cette requête :

`Type=AzureDiagnostics ResourceProvider="MICROSOFT.AUTOMATION" Category=DscNodeStatus NOT(ResultType="started") | measure Count() by ResultType interval 1hour`

Cette requête affiche un graphique de l’état du nœud au fil du temps.

## <a name="azure-monitor-logs-records"></a>Enregistrements de journaux Azure Monitor

Les diagnostics d’Azure Automation créent deux catégories d’enregistrements dans les journaux Azure Monitor :

* Données d'état du nœud (`DscNodeStatusData`)
* Données d'état de la ressource (`DscResourceStatusData`)

### <a name="dscnodestatusdata"></a>DscNodeStatusData

| Propriété | Description |
| --- | --- |
| TimeGenerated |Date et heure d’exécution de la vérification de conformité. |
| NomOpération |`DscNodeStatusData`. |
| ResultType |Valeur indiquant si le nœud est conforme. |
| NodeName_s |Le nom du nœud géré. |
| NodeComplianceStatus_s |Valeur d’état indiquant si le nœud est conforme. |
| DscReportStatus |Valeur d’état indiquant si la vérification de conformité s’est correctement exécutée. |
| ConfigurationMode | Mode utilisé pour appliquer la configuration au nœud. Les valeurs possibles sont les suivantes : <ul><li>`ApplyOnly`: DSC applique la configuration et ne fait rien de plus, sauf si une nouvelle configuration est envoyée au nœud cible ou quand une nouvelle configuration est extraite d’un serveur. Après l’application initiale d’une nouvelle configuration, DSC ne vérifie pas l’écart par rapport à un état configuré précédemment. DSC tente d’appliquer la configuration jusqu’à ce que l’opération réussisse avant que la valeur `ApplyOnly` prenne effet. </li><li>`ApplyAndMonitor`: Il s’agit de la valeur par défaut. Le gestionnaire de configuration locale applique toutes les nouvelles configurations. Après l’application initiale d’une nouvelle configuration, si le nœud cible s’écarte de l’état désiré, DSC signale l’écart dans les journaux d’activité. DSC tente d’appliquer la configuration jusqu’à ce que l’opération réussisse avant que la valeur `ApplyAndMonitor` prenne effet.</li><li>`ApplyAndAutoCorrect`: indique à DSC d’appliquer chaque nouvelle configuration. Après l’application initiale d’une nouvelle configuration, si le nœud cible s’écarte de l’état désiré, DSC signale l’écart dans les journaux d’activité puis applique à nouveau la configuration actuelle.</li></ul> |
| HostName_s | Le nom du nœud géré. |
| IPAddress | L’adresse IPv4 du nœud géré. |
| Category | `DscNodeStatus`. |
| Ressource | Le nom du compte Azure Automation. |
| Tenant_g | GUID identifiant le locataire pour l’appelant. |
| NodeId_g | GUID qui identifie le nœud géré. |
| DscReportId_g | GUID qui identifie le rapport. |
| LastSeenTime_t | Date et heure auxquelles le rapport a été consulté pour la dernière fois. |
| ReportStartTime_t | Date et heure auxquelles le rapport a été commencé. |
| ReportEndTime_t | Date et heure auxquelles le rapport a été terminé. |
| NumberOfResources_d | Le nombre de ressources DSC appelées dans la configuration appliquée au nœud. |
| SourceSystem | Système source identifiant la façon dont les journaux Azure Monitor ont collecté les données. Toujours `Azure` pour les diagnostics Azure. |
| ResourceId |Identificateur de ressource du compte Azure Automation. |
| resultDescription | Description de la ressource pour cette opération. |
| SubscriptionId | ID d’abonnement Azure (GUID) pour le compte Automation. |
| ResourceGroup | Nom du groupe de ressources du compte Automation. |
| ResourceProvider | MICROSOFT.AUTOMATION. |
| ResourceType | AUTOMATIONACCOUNTS. |
| CorrelationId | GUID représentant l’identificateur de corrélation du rapport de conformité. |

### <a name="dscresourcestatusdata"></a>DscResourceStatusData

| Propriété | Description |
| --- | --- |
| TimeGenerated |Date et heure d’exécution de la vérification de conformité. |
| NomOpération |`DscResourceStatusData`.|
| ResultType |Indique si la ressource est conforme. |
| NodeName_s |Le nom du nœud géré. |
| Category | DscNodeStatus. |
| Ressource | Le nom du compte Azure Automation. |
| Tenant_g | GUID identifiant le locataire pour l’appelant. |
| NodeId_g |GUID qui identifie le nœud géré. |
| DscReportId_g |GUID qui identifie le rapport. |
| DscResourceId_s |Le nom de l’instance de ressource DSC. |
| DscResourceName_s |Le nom de la ressource DSC. |
| DscResourceStatus_s |Indique si la ressource DSC est conforme. |
| DscModuleName_s |Le nom du module PowerShell qui contient la ressource DSC. |
| DscModuleVersion_s |La version du module PowerShell qui contient la ressource DSC. |
| DscConfigurationName_s |Le nom de la configuration appliquée au nœud. |
| ErrorCode_s | Le code d’erreur en cas d’échec de la ressource. |
| ErrorMessage_s |Le message d’erreur en cas d’échec de la ressource. |
| DscResourceDuration_d |La durée, en secondes, d’exécution de la ressource DSC. |
| SourceSystem | Mode de collecte de données employé pour les journaux Azure Monitor. Toujours `Azure` pour les diagnostics Azure. |
| ResourceId |Identificateur du compte Azure Automation. |
| resultDescription | La description de cette opération. |
| SubscriptionId | ID d’abonnement Azure (GUID) pour le compte Automation. |
| ResourceGroup | Nom du groupe de ressources du compte Automation. |
| ResourceProvider | MICROSOFT.AUTOMATION. |
| ResourceType | AUTOMATIONACCOUNTS. |
| CorrelationId |GUID représentant l’ID de corrélation du rapport de conformité. |

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble, consultez [Vue d’ensemble d’Azure Automation State Configuration](automation-dsc-overview.md).
- Pour commencer, consultez [Bien démarrer avec Azure Automation State Configuration](automation-dsc-getting-started.md).
- Pour découvrir comment compiler des configurations DSC pour pouvoir les affecter à des nœuds cibles, consultez [Compiler des configurations DSC dans Azure Automation State Configuration](automation-dsc-compile.md).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
- Pour obtenir un exemple d’utilisation d’Azure Automation State Configuration dans un pipeline de déploiement continu, voir [Configurer un déploiement continu avec Chocolatey](automation-dsc-cd-chocolatey.md).
- Pour savoir comment construire différentes requêtes de recherche et examiner les journaux Automation State Configuration avec les journaux Azure Monitor, consultez [Recherches dans les journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md).
- Pour plus d’informations sur les journaux Azure Monitor et sur les sources de collecte de données, consultez la page [Collecter des données de stockage Azure dans les journaux Azure Monitor – Vue d’ensemble](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).