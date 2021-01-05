---
title: Surveillance et optimisation des performances
description: Vue d’ensemble des fonctionnalités de supervision et de réglage des performances ainsi que de la méthodologie dans Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: wiassaf, sstein
ms.date: 09/30/2020
ms.openlocfilehash: 3d28cf1a89ff455cfbed26aad234607ec8cd7e63
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490565"
---
# <a name="monitoring-and-performance-tuning-in-azure-sql-database-and-azure-sql-managed-instance"></a>Supervision et réglage des performances dans Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Pour superviser les performances d’une base de données dans Azure SQL Database et Azure SQL Managed Instance, commencez par superviser les ressources de processeur et d’E/S utilisées par votre charge de travail par rapport au niveau de performances de base de données choisi lors de la sélection d’un niveau de service et d’un niveau de performances spécifiques. Pour cela, Azure SQL Database et Azure SQL Managed Instance émettent des métriques de ressources qui peuvent être affichées dans le portail Azure ou avec l’un des outils de gestion SQL Server suivants : [Azure Data Studio](/sql/azure-data-studio/what-is) ou [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (SSMS).

Azure SQL Database fournit un certain nombre de conseillers en base de données pour proposer des recommandations sur le réglage des performances intelligentes et des options de réglage automatique afin d’améliorer les performances. En outre, Query Performance Insight affiche des détails sur les requêtes responsables de l’utilisation la plus intensive du processeur et des E/S pour les bases de données uniques et mises en pools.

Azure SQL Database et Azure SQL Managed Instance offrent des fonctionnalités de supervision et de réglage avancées qui s’appuient sur l’intelligence artificielle pour vous aider à résoudre les problèmes et à optimiser les performances de vos bases de données et solutions. Vous pouvez choisir de configurer l’[exportation en streaming](metrics-diagnostic-telemetry-logging-streaming-export-configure.md) de ces journaux [Intelligent Insights](intelligent-insights-overview.md) et d’autres journaux de ressources et métriques de bases de données vers l’une des différentes destinations à des fins de consommation et d’analyse, en particulier à l’aide de [SQL Analytics](../../azure-monitor/insights/azure-sql.md). Azure SQL Analytics est une solution de supervision du cloud avancée permettant de superviser dans un seul affichage les performances de toutes vos bases de données à grande échelle et sur plusieurs abonnements. Pour obtenir la liste des journaux et des métriques que vous pouvez exporter, consultez [Télémétrie de diagnostic pour l’exportation](metrics-diagnostic-telemetry-logging-streaming-export-configure.md#diagnostic-telemetry-for-export)

SQL Server possède ses propres capacités d’analyse et de diagnostic dont bénéficient SQL Database et SQL Managed Instance, comme le [magasin de requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) et les [vues de gestion dynamique (DMV)](/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views). Consultez [Surveillance avec des DMV](monitoring-with-dmvs.md) pour des scripts afin de surveiller un large éventail de problèmes de performances.

## <a name="monitoring-and-tuning-capabilities-in-the-azure-portal"></a>Capacités de surveillance et d’optimisation dans le portail Azure

Dans le portail Azure, Azure SQL Database et Azure SQL Managed Instance assurent la supervision des métriques de ressources. Azure SQL Database fournit des conseillers en base de données, et Query Performance Insight propose des recommandations de paramétrage des requêtes et une analyse des performances des requêtes. Dans le portail Azure, vous pouvez activer le paramétrage automatique pour les [serveurs SQL logiques](logical-servers.md) et leurs bases de données uniques et mises en pool.

> [!NOTE]
> Les bases de données dont l’utilisation est très faible peuvent apparaître dans le portail avec une utilisation inférieure à la réalité. En raison de la manière dont la télémétrie est émise lors de la conversion d’une valeur double en un nombre entier le plus proche, certaines quantités d’utilisation inférieures à 0,5 seront arrondies à 0, ce qui entraîne une perte de granularité de la télémétrie émise. Pour plus d’informations, consultez [Métriques faibles de base de données et de pool élastique arrondies à zéro](#low-database-and-elastic-pool-metrics-rounding-to-zero).

### <a name="azure-sql-database-and-azure-sql-managed-instance-resource-monitoring"></a>Supervision des ressources Azure SQL Database et Azure SQL Managed Instance

Vous pouvez superviser rapidement diverses métriques de ressources dans la vue **Métriques** du portail Azure. Ces métriques vous permettent de déterminer si une base de données atteint 100 % des ressources de processeur, de mémoire ou d’E/S. Un pourcentage DTU, de processeur ou d’E/S élevé indique que votre charge de travail peut nécessiter plus de ressources de processeur ou d’E/S. Cela peut également indiquer que certaines requêtes doivent être optimisées.

  ![Mesures de ressources](./media/monitor-tune-overview/resource-metrics.png)

### <a name="database-advisors-in-azure-sql-database"></a>Conseillers en base de données dans Azure SQL Database

Azure SQL Database inclut des [conseillers en base de données](database-advisor-implement-performance-recommendations.md) qui fournissent des recommandations de réglage des performances pour les bases de données uniques et mises en pool. Ces recommandations sont disponibles dans le portail Azure et dans [PowerShell](/powershell/module/az.sql/get-azsqldatabaseadvisor). Vous pouvez également activer le [réglage automatique](automatic-tuning-overview.md) afin qu’Azure SQL Database implémente automatiquement ces recommandations de réglage.

### <a name="query-performance-insight-in-azure-sql-database"></a>Query Performance Insight dans Azure SQL Database

[Query Performance Insight](query-performance-insight-use.md) affiche dans le portail Azure les performances des requêtes qui consomment le plus de ressources et dont l’exécution et la plus longue pour les bases de données uniques et mises en pool.

### <a name="low-database-and-elastic-pool-metrics-rounding-to-zero"></a>Métriques faibles de base de données et de pool élastique arrondies à zéro

À compter de septembre 2020, les bases de données dont l’utilisation est très faible peuvent apparaître dans le portail avec une utilisation inférieure à la réalité. En raison de la manière dont la télémétrie est émise lors de la conversion d’une valeur double en un nombre entier le plus proche, certaines quantités d’utilisation inférieures à 0,5 seront arrondies à 0, ce qui entraîne une perte de granularité de la télémétrie émise.

Par exemple : Imaginez une fenêtre d’une minute avec les quatre points de données suivants : 0,1, 0,1, 0,1, 0,1. Ces valeurs basses sont arrondies à 0, 0, 0, 0 et présentent une moyenne de 0. Si l’un des points de données est supérieur à 0,5, par exemple : 0,1, 0,1, 0,9, 0,1, ils sont arrondis à 0, 0, 1, 0 et présentent une moyenne de 0,25.

Métriques de base de données concernées :
- cpu_percent
- log_write_percent
- workers_percent
- sessions_percent
- physical_data_read_percent
- dtu_consumption_percent2
- xtp_storage_percent

Métriques de pool élastique concernées :
- cpu_percent
- physical_data_read_percent
- log_write_percent
- memory_usage_percent
- data_storage_percent
- peak_worker_percent
- peak_session_percent
- xtp_storage_percent
- allocated_data_storage_percent


## <a name="generate-intelligent-assessments-of-performance-issues"></a>Générer des évaluations intelligentes des problèmes de performances

[Intelligent Insights](intelligent-insights-overview.md) pour Azure SQL Database et Azure SQL Managed Instance utilise l’intelligence intégrée pour superviser en permanence l’utilisation de la base de données via l’intelligence artificielle et détecter des événements perturbateurs entraînant des performances médiocres. Intelligent Insights détecte automatiquement les problèmes de performances liés aux bases de données en fonction des temps d’attente d’exécution des requêtes, des erreurs ou des expirations de délais d’attente. Une fois un tel événement détecté, une analyse détaillée est effectuée, qui génère un journal de ressources (appelé SQLInsights) avec une [évaluation intelligente des problèmes](intelligent-insights-troubleshoot-performance.md). Cette évaluation se compose d’une analyse de cause racine du problème de performances de la base de données et, dans la mesure du possible, de recommandations pour une amélioration des performances.

Intelligent Insights est une fonctionnalité unique de l’intelligence intégrée Azure, qui apporte les avantages suivants :

- Surveillance proactive
- Analyse des performances personnalisée
- Détection anticipée de la dégradation des performances de la base de données
- Analyse de la cause racine des problèmes détectés
- Recommandations pour l’amélioration des performances
- Capacité de scale-out sur des centaines de milliers de bases de données
- Impact positif sur les ressources de DevOps et le coût total de possession

## <a name="enable-the-streaming-export-of-metrics-and-resource-logs"></a>Activer l’exportation en streaming des journaux de métriques et de ressources

Vous pouvez activer et configurer l’[exportation en streaming](metrics-diagnostic-telemetry-logging-streaming-export-configure.md) de la télémétrie de diagnostic vers l’une des différentes destinations, notamment le journal de ressources Intelligent Insights. Utilisez [SQL Analytics](../../azure-monitor/insights/azure-sql.md) et d’autres capacités pour utiliser cette télémétrie de diagnostic supplémentaire afin d’identifier et de résoudre les problèmes de performances.

Vous configurez les paramètres de diagnostic pour diffuser en continu des catégories de métriques et des journaux de ressources pour des bases de données individuelles, des bases de données mises en pool, des pools élastiques, des instances gérées et des bases de données d'instance vers l'une des ressources Azure suivantes.

### <a name="log-analytics-workspace-in-azure-monitor"></a>Espace de travail Log Analytics dans Azure Monitor

Vous pouvez diffuser en continu des journaux de métriques et de ressources vers un [espace de travail Log Analytics dans Azure Monitor](../../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace). Les données diffusées en continu ici peuvent être utilisées par [SQL Analytics](../../azure-monitor/insights/azure-sql.md), une solution de surveillance cloud uniquement permettant une surveillance intelligente de vos bases de données, avec des rapports de performances, des alertes et des recommandations d’atténuation. Les données diffusées en continu vers un espace de travail Log Analytics peuvent être analysées avec d’autres données de surveillance collectées, ce qui vous permet également d’exploiter d’autres fonctionnalités Azure Monitor, notamment les alertes et les visualisations.

### <a name="azure-event-hubs"></a>Hubs d'événements Azure

Vous pouvez diffuser en continu des journaux de métriques et de ressources vers [Azure Event Hubs](../../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs). Diffusez en continu des données télémétriques vers des hubs d’événements pour fournir les fonctionnalités suivantes :

- **Streaming des journaux vers des systèmes de journalisation et de télémétrie tiers**

  Diffusion en continu de toutes vos métriques et de tous vos journaux de ressources vers un Event Hub unique pour envoyer les données de journal vers un outil SIEM ou d’analytique des journaux d’activité tiers.
- **Créer une plateforme de journalisation et de télémétrie personnalisée**

  La nature hautement évolutive de l’abonnement aux hubs d’événements vous permet d’ingérer de manière flexible les journaux de métriques et de ressources dans une plateforme de télémétrie personnalisée. Pour plus d’informations, consultez [Conception et dimensionnement d’une plateforme de télémétrie à échelle mondiale sur Azure Event Hubs](https://azure.microsoft.com/documentation/videos/build-2015-designing-and-sizing-a-global-scale-telemetry-platform-on-azure-event-Hubs/).
- **Afficher l’intégrité du service en procédant au streaming des données de vers Power BI**

  Utilisez Event Hubs, Stream Analytics et Power BI pour transformer vos données de diagnostic en informations en temps réel sur vos services Azure. Consultez [Stream Analytics et Power BI : tableau de bord d’analyse en temps réel pour les données de streaming](../../stream-analytics/stream-analytics-power-bi-dashboard.md) pour plus de détails sur cette solution.

### <a name="azure-storage"></a>Stockage Azure

Diffusez en continu des journaux de métriques et de ressources vers le [Stockage Azure](../../azure-monitor/platform/resource-logs.md#send-to-azure-storage). Utilisez le Stockage Azure pour archiver de vastes quantités de données télémétriques de diagnostics pour une fraction du coût des deux options de streaming précédentes.

## <a name="use-extended-events"></a>Utiliser des événements étendus 

De plus, vous pouvez utiliser des [événements étendus](/sql/relational-databases/extended-events/extended-events) dans SQL Server pour des fonctionnalités avancées de supervision et de résolution des problèmes. L’architecture des événements étendus permet aux utilisateurs de collecter autant de données que nécessaire pour dépanner ou identifier un problème de performances. Pour plus d’informations sur l’utilisation d’événements étendus dans Azure SQL Database, consultez [Événements étendus dans Azure SQL Database](xevent-db-diff-from-svr.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les recommandations relatives aux performances intelligentes pour les bases de données uniques et mises en pool, consultez [Recommandations relatives aux performances de Database Advisor](database-advisor-implement-performance-recommendations.md).
- Pour plus d’informations sur l’analyse automatique des performances de base de données à l’aide des diagnostics automatisés et de l’analyse de la cause racine des problèmes de performances, consultez l’article [Intelligent Insights](intelligent-insights-overview.md).