---
title: Solution Azure SQL Analytics dans Azure Monitor | Microsoft Docs
description: La solution Azure SQL Analytics vous permet de gérer vos bases de données Azure SQL.
ms.subservice: logs
ms.topic: conceptual
author: danimir
ms.author: danil
ms.date: 09/19/2020
ms.reviewer: carlrab
ms.openlocfilehash: 0015138f4da9f66e2f9148e468dd1b5543ae0c4b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91397077"
---
# <a name="monitor-azure-sql-database-using-azure-sql-analytics-preview"></a>Superviser Azure SQL Database avec Azure SQL Analytics (préversion)

![Symbole Azure SQL Analytics](./media/azure-sql/azure-sql-symbol.png)

Azure SQL Analytics est une solution cloud avancée permettant de superviser dans un seul affichage les performances de toutes vos bases de données Azure SQL à grande échelle et sur plusieurs abonnements. Azure SQL Analytics collecte et visualise des métriques de performances clés à l’aide d’une intelligence intégrée pour résoudre les problèmes de performances.

À l’aide de ces métriques collectées, vous pouvez créer des règles et des alertes de surveillance personnalisées. Azure SQL Analytics vous aide à identifier les problèmes sur chacune des couches de votre pile applicative. Elle utilise les données de diagnostic Azure avec les vues Azure Monitor pour présenter les données relatives à l’ensemble de vos bases de données Azure SQL dans un espace de travail Log Analytics unique. Azure Monitor prend en charge la collecte, la mise en correspondance et l’affichage des données structurées et non structurées.

Pour une présentation pratique sur l'utilisation de la solution Azure SQL Analytics et des scénarios d'utilisation standard, regardez la vidéo incorporée :

>[!VIDEO https://www.youtube.com/embed/j-NDkN4GIzg]
>

## <a name="connected-sources"></a>Sources connectées

Azure SQL Analytics est une solution de surveillance cloud uniquement prenant en charge la diffusion en continu de la télémétrie de diagnostic pour toutes vos bases de données Azure SQL. Étant donné qu’Azure SQL Analytics n’utilise pas d’agents pour se connecter à Azure Monitor, elle ne prend pas en charge la supervision des serveurs SQL Server hébergés localement ou dans des machines virtuelles.

| Source connectée | Prise en charge | Description |
| --- | --- | --- |
| [Paramètres de diagnostic](../platform/diagnostic-settings.md) | **Oui** | Les métriques Azure et les données des journaux sont envoyées à Azure Monitor directement par Azure. |
| [Compte Azure Storage](../platform/resource-logs.md#send-to-log-analytics-workspace) | Non | Azure Monitor ne lit pas les données d’un compte de stockage. |
| [Agents Windows](../platform/agent-windows.md) | Non | Les agents Windows directs ne sont pas utilisés par Azure SQL Analytics. |
| [Agents Linux](../learn/quick-collect-linux-computer.md) | Non | Les agents Linux directs ne sont pas utilisés par Azure SQL Analytics. |
| [Groupe d’administration de Microsoft System Center Operations Manager](../platform/om-agents.md) | Non | Azure SQL Analytics n’utilise pas de connexion directe entre l’agent Operations Manager et Azure Monitor. |

## <a name="azure-sql-analytics-options"></a>Options Azure SQL Analytics

Le tableau ci-dessous présente les options prises en charge pour deux versions du tableau de bord Azure SQL Analytics : l’une pour Azure SQL Database, et l’autre pour les bases de données Azure SQL Managed Instance.

| Option Azure SQL Analytics | Description | Prise en charge de SQL Database | Prise en charge de SQL Managed Instance |
| --- | ------- | ----- | ----- |
| Ressource par type | Perspective comptabilisant toutes les ressources surveillées. | Oui | Oui |
| Insights | Permet une exploration hiérarchique des performances dans Intelligent Insights. | Oui | Oui |
| Erreurs | Permet une exploration hiérarchique des erreurs SQL qui se sont produites dans les bases de données. | Oui | Oui |
| Délais d’expiration | Permet une exploration hiérarchique des délais d’expiration SQL qui se sont produits dans les bases de données. | Oui | Non |
| Blocages | Permet une exploration hiérarchique des blocages SQL qui se sont produits dans les bases de données. | Oui | Non |
| Attentes de la base de données | Permet une exploration hiérarchique des statistiques d’attente SQL au niveau des bases de données. Inclut des résumés du temps d’attente total et de la durée d’attente par type d’attente. |Oui | Non |
| Durée de la requête | Permet une exploration hiérarchique des statistiques d’exécution de la requête, par exemple la durée de la requête, l’utilisation du processeur, l’utilisation des E/S de données, l’utilisation des E/S du journal. | Oui | Oui |
| Attentes de requête | Permet une exploration hiérarchique des statistiques d’attente de requête par catégorie d’attente. | Oui | Oui |

## <a name="configuration"></a>Configuration

Procédez de la manière décrite dans [Ajouter des solutions Azure Monitor à partir de la Galerie Solutions](./solutions.md) pour ajouter Azure SQL Analytics (préversion) à votre espace de travail Log Analytics.

### <a name="configure-azure-sql-database-to-stream-diagnostics-telemetry"></a>Configurer Azure SQL Database pour diffuser en continu les données de télémétrie de diagnostic

Après avoir créé une solution Azure SQL Analytics dans votre espace de travail, **configurez chacune** des ressources que vous souhaitez surveiller pour diffuser en continu sa télémétrie de diagnostic vers Azure SQL Analytics. Suivez les instructions détaillées de cette page :

- Activez Azure Diagnostics pour votre base de données pour [diffuser en continu les données de télémétrie de diagnostic vers Azure SQL Analytics](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md).

La page ci-dessus fournit également des instructions sur l’activation de la prise en charge de la surveillance de plusieurs abonnements Azure depuis un espace de travail Azure SQL Analytics faisant office de tableau de bord unique.

## <a name="using-azure-sql-analytics"></a>Utilisation d’Azure SQL Analytics

Lorsque vous ajoutez Azure SQL Analytics à votre espace de travail, la vignette Azure SQL Analytics y est également ajoutée ; elle apparaît dans la Vue d’ensemble. Sélectionnez le lien Afficher le résumé pour charger le contenu de la vignette.

![Vignette de résumé d’Azure SQL Analytics](./media/azure-sql/azure-sql-sol-tile-01.png)

Une fois chargée, la vignette indique le nombre de bases de données et de pools élastiques dans SQL Database ainsi que le nombre d’instances et de bases données d’instance dans SQL Managed Instance dont Azure SQL Analytics reçoit les données de télémétrie de diagnostic.

![Vignette Azure SQL Analytics](./media/azure-sql/azure-sql-sol-tile-02.png)

Azure SQL Analytics fournit deux vues distinctes, une pour la surveillance de SQL Database et l’autre pour la surveillance de SQL Managed Instance.

Pour visualiser le tableau de bord de supervision d’Azure SQL Analytics pour SQL Database, cliquez sur la partie supérieure de la vignette. Pour visualiser le tableau de bord de supervision d’Azure SQL Analytics pour SQL Managed Instance, cliquez sur la partie inférieure de la vignette.

### <a name="viewing-azure-sql-analytics-data"></a>Affichage des données Azure SQL Analytics

Ce tableau de bord propose un aperçu de toutes les bases de données surveillées selon différentes perspectives. Pour que ces différentes perspectives fonctionnent, vous devez activer les métriques ou les journaux d’activité appropriés sur les ressources SQL, celles dont la diffusion doit s’effectuer en continu sur l’espace de travail Log Analytics.

Si certaines métriques ou certains journaux d’activité ne sont pas diffusés en continu vers Azure Monitor, les informations de supervision n’apparaissent pas dans les vignettes d’Azure SQL Analytics.

### <a name="sql-database-view"></a>Vue SQL Database

Lorsque la vignette Azure SQL Analytics de la base de données est sélectionnée, le tableau de bord de monitoring s’affiche.

![Capture d'écran représentant le tableau de bord de surveillance.](./media/azure-sql/azure-sql-sol-overview.png)

En sélectionnant l’une des vignettes, vous ouvrez le rapport détaillé d’une perspective en particulier. Une fois la perspective sélectionnée, le rapport détaillé est ouvert.

![Capture d'écran représentant le rapport détaillé d'une perspective spécifique.](./media/azure-sql/azure-sql-sol-metrics.png)

Chaque perspective de cette vue fournit des récapitulatifs aux niveaux de l’abonnement, du serveur, du pool élastique et de la base de données. En outre, sur la droite, chaque perspective présente une vision spécifique au rapport. Pour approfondir davantage, sélectionnez l’abonnement, le serveur, le pool ou la base de données dans la liste.

### <a name="sql-managed-instance-view"></a>Vue SQL Managed Instance

Lorsque la vignette Azure SQL Analytics des bases de données est sélectionnée, le tableau de bord de monitoring s’affiche.

![Vue d’ensemble d’Azure SQL Analytics](./media/azure-sql/azure-sql-sol-overview-mi.png)

En sélectionnant l’une des vignettes, vous ouvrez le rapport détaillé d’une perspective en particulier. Une fois la perspective sélectionnée, le rapport détaillé est ouvert.

La sélection de la vue SQL Managed Instance présente des détails concernant l’utilisation de l’instance, les bases de données d’instance et les données de télémétrie pour les requêtes exécutées sur l’instance managée.

![Délais d’expiration d’Azure SQL Analytics](./media/azure-sql/azure-sql-sol-metrics-mi.png)

### <a name="intelligent-insights-report"></a>Rapport Intelligent Insights

Azure SQL Database [Intelligent Insights](../../azure-sql/database/intelligent-insights-overview.md) permet de suivre les performances de toutes les bases de données Azure SQL. Toutes les analyses Intelligent Insights collectées peut être visualisées et sont accessibles par le biais de la perspective Insights.

![Insights Azure SQL Analytics](./media/azure-sql/azure-sql-sol-insights.png)

### <a name="elastic-pools-and-database-reports"></a>Rapports de base de données et de pool élastique

Les pools élastiques et les bases de données ont leurs propres rapports qui affichent toutes les données collectées pour la ressource, dans le temps imparti.

![Base de données Azure SQL Analytics](./media/azure-sql/azure-sql-sol-database.png)

![Pool élastique SQL Azure](./media/azure-sql/azure-sql-sol-pool.png)

### <a name="query-reports"></a>Rapports de requête

En vous aidant de la durée de la requête et des attentes de requête, vous pouvez mettre en corrélation les performances d’une requête dans le rapport de requête. Ce rapport compare les performances des requêtes entre les différentes bases de données. Il identifie également les bases de données qui effectuent la requête sélectionnée correctement et celles qui le font lentement.

![Requêtes Azure SQL Analytics](./media/azure-sql/azure-sql-sol-queries.png)

## <a name="permissions"></a>Autorisations

Pour pouvoir utiliser Azure SQL Analytics, les utilisateurs doivent avoir au moins reçu le rôle de Lecteur dans Azure. Toutefois, ils ne seront pas autorisés pour autant à voir le texte de la requête ou à effectuer des actions de paramétrage automatique. Dans Azure, les rôles permettant d’utiliser Azure SQL Analytics dans son intégralité sont les suivants : Propriétaire, Contributeur, Contributeur SQL DB et Contributeur SQL Server. Vous pouvez également créer un rôle personnalisé dans le portail disposant uniquement des autorisations spécifiques nécessaires à l’utilisation d’Azure SQL Analytics, sans accès à la gestion des autres ressources.

### <a name="creating-a-custom-role-in-portal"></a>Création d’un rôle personnalisé dans le portail

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Sachant que certaines organisations appliquent un contrôle strict des autorisations dans Azure, le script PowerShell suivant permet de créer un rôle personnalisé « Opérateur de monitoring SQL Analytics » sur le Portail Azure avec les autorisations de lecture et d’écriture minimales requises pour pouvoir utiliser pleinement Azure SQL Analytics.

Remplacez « {SubscriptionId} » dans le script ci-dessous par votre ID d’abonnement Azure, puis exécutez le script avec un rôle Propriétaire ou Contributeur dans Azure.

   ```powershell
    Connect-AzAccount
    Select-AzSubscription {SubscriptionId}
    $role = Get-AzRoleDefinition -Name Reader
    $role.Name = "SQL Analytics Monitoring Operator"
    $role.Description = "Lets you monitor database performance with Azure SQL Analytics as a reader. Does not allow change of resources."
    $role.IsCustom = $true
    $role.Actions.Add("Microsoft.SQL/servers/databases/read");
    $role.Actions.Add("Microsoft.SQL/servers/databases/topQueries/queryText/*");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/write");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/recommendedActions/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/recommendedActions/write");
    $role.Actions.Add("Microsoft.Sql/servers/databases/automaticTuning/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/automaticTuning/write");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/read");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/write");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/recommendedActions/read");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/recommendedActions/write");
    $role.Actions.Add("Microsoft.Resources/deployments/write");
    $role.AssignableScopes = "/subscriptions/{SubscriptionId}"
    New-AzRoleDefinition $role
   ```

Une fois le nouveau rôle créé, attribuez ce rôle à chaque utilisateur ayant besoin de bénéficier d’une autorisation personnalisée pour utiliser Azure SQL Analytics.

## <a name="analyze-data-and-create-alerts"></a>Analyser les données et créer des alertes

L’analyse de données dans Azure SQL Analytics repose sur le [langage Log Analytics](../log-query/get-started-queries.md) pour vos requêtes et génération de rapports personnalisées. Vous trouverez dans [Métriques et journaux d’activité disponibles](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md#metrics-and-logs-available) la description des données collectées dans la ressource de base de données qui sont disponibles pour les requêtes personnalisées.

Dans Azure SQL Analytics, les alertes automatisées nécessitent d’écrire une requête Log Analytics qui déclenche une alerte lorsqu’une condition est remplie. Vous trouverez ci-dessous plusieurs exemples de requêtes Log Analytics pour lesquelles une alerte peut être configurée dans Azure SQL Analytics.

### <a name="creating-alerts-for-azure-sql-database"></a>Création d’alertes pour Azure SQL Database

Vous pouvez facilement [créer des alertes](../platform/alerts-metric.md) avec les données provenant de ressources Azure SQL Database. Voici quelques [requêtes dans les journaux](../log-query/log-query-overview.md) qui vous seront utiles pour créer des alertes :

#### <a name="high-cpu"></a>UC élevée

```
AzureMetrics
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/DATABASES/"
| where MetricName=="cpu_percent"
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
>
> - La condition préalable à la configuration de cette alerte est que les bases de données surveillées transmettent en continu les métriques de base à Azure SQL Analytics.
> - Remplacez la valeur MetricName cpu_percent par dtu_consumption_percent pour obtenir des résultats de DTU élevés à la place.

#### <a name="high-cpu-on-elastic-pools"></a>UC élevée sur les pools élastiques

```
AzureMetrics
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/ELASTICPOOLS/"
| where MetricName=="cpu_percent"
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
>
> - La condition préalable à la configuration de cette alerte est que les bases de données surveillées transmettent en continu les métriques de base à Azure SQL Analytics.
> - Remplacez la valeur MetricName cpu_percent par dtu_consumption_percent pour obtenir des résultats de DTU élevés à la place.

#### <a name="storage-in-average-above-95-in-the-last-1-hr"></a>Stockage supérieur à 95 % en moyenne au cours de la dernière heure

```
let time_range = 1h;
let storage_threshold = 95;
AzureMetrics
| where ResourceId contains "/DATABASES/"
| where MetricName == "storage_percent"
| summarize max_storage = max(Average) by ResourceId, bin(TimeGenerated, time_range)
| where max_storage > storage_threshold
| distinct ResourceId
```

> [!NOTE]
>
> - La condition préalable à la configuration de cette alerte est que les bases de données surveillées transmettent en continu les métriques de base à Azure SQL Analytics.
> - Cette requête nécessite une règle d’alerte pour être configurée de manière à déclencher une alerte lorsqu’il y a des résultats (résultats > 0) à la requête, indiquant que la condition existe sur certaines bases de données. La sortie est une liste de ressources de base de données qui se situent au-dessus de storage_threshold dans le time_range défini.
> - La sortie est une liste de ressources de base de données qui se situent au-dessus de storage_threshold dans le time_range défini.

#### <a name="alert-on-intelligent-insights"></a>Alerte sur Intelligent Insights

> [!IMPORTANT]
> Si une base de données fonctionne bien et qu'aucun rapport Intelligent Insights n'a été généré, cette requête échoue et le message d'erreur suivant apparaît : Échec de la résolution de l'expression scalaire « rootCauseAnalysis_s ». Ce comportement est normal lorsqu'aucun rapport Intelligent Insights n'est généré pour la base de données.

```
let alert_run_interval = 1h;
let insights_string = "hitting its CPU limits";
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| where TimeGenerated > ago(alert_run_interval)
| where rootCauseAnalysis_s contains insights_string
| distinct ResourceId
```

> [!NOTE]
>
> - La condition préalable à la configuration de cette alerte est que les bases de données surveillées transmettent en continu le journal de diagnostic SQLInsights à Azure SQL Analytics.
> - Cette requête nécessite la configuration d’une règle d’alerte pour une exécution à la même fréquence qu’alert_run_interval afin d’éviter des résultats en double. La règle doit être configurée pour déclencher l’alerte lorsqu’il y a des résultats (résultats > 0) à la requête.
> - Personnalisez l’alert_run_interval afin de spécifier l’intervalle de temps pour vérifier si la condition est remplie sur les bases de données configurées pour transmettre en continu le journal SQLInsights à Azure SQL Analytics.
> - Personnalisez l’insights_string pour capturer la sortie du texte d’analyse de cause racine Insights. Ce texte est identique à celui affiché dans l’interface utilisateur d’Azure SQL Analytics que vous pouvez utiliser à partir des insights existants. Vous pouvez également utiliser la requête ci-dessous pour voir le texte de tous les éléments Insights générés sur votre abonnement. Utilisez la sortie de la requête pour collecter les chaînes distinctes et configurer des alertes sur Insights.

```
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| distinct rootCauseAnalysis_s
```

### <a name="creating-alerts-for-sql-managed-instance"></a>Création d’alertes pour SQL Managed Instance

#### <a name="storage-is-above-90"></a>Le stockage est supérieur à 90 %

```
let storage_percentage_threshold = 90;
AzureDiagnostics
| where Category =="ResourceUsageStats"
| summarize (TimeGenerated, calculated_storage_percentage) = arg_max(TimeGenerated, todouble(storage_space_used_mb_s) *100 / todouble (reserved_storage_mb_s))
   by ResourceId
| where calculated_storage_percentage > storage_percentage_threshold
```

> [!NOTE]
>
> - La condition préalable à la configuration de cette alerte est que la diffusion en continu du journal ResourceUsageStats vers Azure SQL Analytics soit activée pour l’instance gérée supervisée.
> - Cette requête nécessite qu’une règle d’alerte soit configurée de manière à déclencher une alerte lorsqu’il existe des résultats (résultats > 0) pour la requête, indiquant que la condition existe sur l’instance gérée. La sortie correspond au pourcentage d’utilisation du stockage sur l’instance gérée.

#### <a name="cpu-average-consumption-is-above-95-in-the-last-1-hr"></a>Consommation moyenne de l’UC supérieure à 95 % au cours de la dernière heure

```
let cpu_percentage_threshold = 95;
let time_threshold = ago(1h);
AzureDiagnostics
| where Category == "ResourceUsageStats" and TimeGenerated > time_threshold
| summarize avg_cpu = max(todouble(avg_cpu_percent_s)) by ResourceId
| where avg_cpu > cpu_percentage_threshold
```

> [!NOTE]
>
> - La condition préalable à la configuration de cette alerte est que la diffusion en continu du journal ResourceUsageStats vers Azure SQL Analytics soit activée pour l’instance gérée supervisée.
> - Cette requête nécessite qu’une règle d’alerte soit configurée de manière à déclencher une alerte lorsqu’il existe des résultats (résultats > 0) pour la requête, indiquant que la condition existe sur l’instance gérée. La sortie est la proportion moyenne d’utilisation de l’UC au cours de la période définie sur l’instance gérée.

### <a name="pricing"></a>Tarifs

Azure SQL Analytics est gratuite, mais toute consommation de la télémétrie de diagnostics au-delà des unités gratuites d’ingestion de données qui sont allouées chaque mois est facturée. Consultez [Tarifs de Log Analytics](https://azure.microsoft.com/pricing/details/monitor). Les unités gratuites d’ingestion de données fournies permettent de superviser gratuitement plusieurs bases de données chaque mois. Des bases de données plus actives, traitant des charges de travail plus volumineuses, ingèrent plus de données que les bases de données inactives. Vous pouvez facilement suivre votre consommation d’ingestion de données dans Azure SQL Analytics en sélectionnant l’espace de travail OMS dans le menu de navigation d’Azure SQL Analytics, puis en sélectionnant Utilisation et estimation des coûts.

## <a name="next-steps"></a>Étapes suivantes

- Utilisez les [requêtes de journaux dans Azure Monitor](../log-query/log-query-overview.md) pour afficher des données détaillées Azure SQL.
- [Créer un tableau de bord personnalisé](../learn/tutorial-logs-dashboards.md) comportant les données Azure SQL.
- [Create and manage alert rules in Log Analytics with the OMS portal](../platform/alerts-overview.md) (Créer et gérer des règles d’alerte dans Log Analytics dans Log Analytics à l’aide du portail OMS).

