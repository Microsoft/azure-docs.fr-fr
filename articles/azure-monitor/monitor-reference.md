---
title: Éléments supervisés par Azure Monitor
description: Informations de référence sur tous les services et autres ressources supervisés par Azure Monitor.
ms.subservice: ''
ms.topic: conceptual
author: rboucher
ms.author: robb
ms.date: 08/15/2020
ms.openlocfilehash: 08d30fb72398c4b43422eb21f132d5fddd5502b7
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96853139"
---
# <a name="what-is-monitored-by-azure-monitor"></a>Quels sont les éléments supervisés par Azure Monitor ?
Cet article décrit les différentes applications et services supervisés par Azure Monitor. 

## <a name="insights-and-core-solutions"></a>Insights et solutions principales
Les insights et solutions principales sont considérés comme faisant partie d’Azure Monitor et suivent les contrats de niveau de service et de support pour Azure. Ils sont pris en charge dans toutes les régions Azure où Azure Monitor est disponible.

### <a name="insights"></a>Insights

Les Insights offrent une expérience de surveillance personnalisée pour des applications et services spécifiques. Ils collectent et analysent à la fois les journaux et les métriques.

| Insight | Description |
|:---|:---|
| [Application Insights](app/app-insights-overview.md) | Service extensible Gestion des performances des applications (APM) pour superviser votre application web en temps réel sur n’importe quelle plateforme. |
| [Azure Monitor pour conteneurs](insights/container-insights-overview.md) | Supervise les performances des charges de travail de conteneur déployées sur Azure Container Instances ou sur des clusters Kubernetes managés hébergés sur Azure Kubernetes Service (AKS). |
| [Azure Monitor pour Cosmos DB](insights/cosmosdb-insights-overview.md) | Fournit une vue des performances globales, des défaillances, de la capacité et de l’intégrité opérationnelle de toutes vos ressources Azure Cosmos DB dans une expérience interactive unifiée. |
| [Azure Monitor pour réseaux (préversion)](insights/network-insights-overview.md) | Fournit une vue complète de l’intégrité et des métriques pour toutes vos ressources réseau. La fonctionnalité de recherche avancée vous aide à identifier les dépendances de ressources, par l’activation de scénarios tels que l’identification des ressources qui hébergent votre site Web, en recherchant simplement le nom de votre site Web. |
[Azure Monitor pour groupes de ressources (préversion)](insights/resource-group-insights.md) |  Facilite la classification et le diagnostic des problèmes que rencontrent vos ressources, tout en offrant un contexte en lien avec l’intégrité et les performances du groupe de ressources dans son ensemble. |
| [Azure Monitor pour le stockage](insights/storage-insights-overview.md) | Fournit une supervision complète de vos comptes de stockage Azure grâce à une vue unifiée des performances, des capacités et de la disponibilité de vos services de stockage Azure. |
| [Azure Monitor pour machines virtuelles](insights/vminsights-overview.md) | Supervise les machines virtuelles et groupes de machines virtuelles identiques à grande échelle. Il analyse les performances et l’intégrité des machines virtuelles Windows et Linux, et supervise leurs processus et dépendances vis-à-vis d’autres ressources et des processus externes. |
| [Azure Monitor pour Key Vault (préversion)](./insights/key-vault-insights-overview.md) | Offre une analyse complète de vos coffres de clés en fournissant une vue unifiée des requêtes, des performances, des échecs et de la latence de Key Vault. |
| [Azure Monitor pour Azure Cache pour Redis (préversion)](insights/redis-cache-insights-overview.md) |  Offre une vue unifiée et interactive des performances globales, des défaillances, de la capacité et de l’intégrité opérationnelle. |


### <a name="core-solutions"></a>Solutions principales

Les solutions sont basées sur des requêtes de journal et des vues personnalisées pour une application ou un service particulier. Elles collectent et analysent uniquement les journaux et sont dépréciées au fil du temps en faveur des insights.

| Solution | Description |
|:---|:---|
| [Intégrité des agents](insights/solution-agenthealth.md) | Analyse l’intégrité et la configuration des agents Log Analytics. |
| [Gestion des alertes](platform/alert-management-solution.md) | Analyse les alertes collectées à partir de System Center Operations Manager, Nagios ou Zabbix. |
| [Service Map](insights/service-map.md) | Détecte automatiquement les composants d’application sur les systèmes Windows et Linux, et cartographie la communication entre les différents services. |



## <a name="azure-services"></a>Services Azure
Le tableau suivant liste les services Azure et les données qu’ils collectent dans Azure Monitor. 

- Métriques : le service collecte automatiquement les métriques dans les métriques Azure Monitor. 
- Journaux : le service prend en charge les paramètres de diagnostic qui peuvent collecter les journaux et les métriques de la plateforme dans les journaux Azure Monitor.
- Insight : un insight est disponible pour le service, qui procure une expérience de supervision personnalisée de ce dernier.

| Service | Mesures | Journaux d’activité | Insight | Notes |
|:---|:---|:---|:---|:---|
|Active Directory | Non | Oui | [Oui](../active-directory/reports-monitoring/howto-use-azure-monitor-workbooks.md) |  |
|Active Directory B2C | Non | Non | Non |  |
|Active Directory Domain Services | Non | Oui | Non |  |
|Journal d’activité | Non | Oui | Non | |
|Protection avancée contre les menaces | Non | Non | Non |  |
|Advisor | Non | Non | Non |  |
|AI Builder | Non | Non | Non |  |
|Analysis Services | Oui | Oui | Non |  |
|API pour FHIR | Non | Non | Non |  |
|Gestion des API | Oui | Oui | Non |  |
|App Service | Oui | Oui | Non |  |
|AppConfig | Non | Non | Non |  |
|Application Gateway | Oui | Oui | Non |  |
|Service d’attestation | Non | Non | Non |  |
|Automatisation | Oui | Oui | Non |  |
|Azure Service Manager (RDFE) | Non | Non | Non |  |
|Sauvegarde | Non | Oui | Non |  |
|Bastion | Non | Non | Non |  |
|Batch | Oui | Oui | Non |  |
|Batch AI | Non | Non | Non |  |
|Service Blockchain | Non | Oui | Non |  |
|Blueprints | Non | Non | Non |  |
|Service de robot | Non | Non | Non |  |
|Cloud Services | Oui | Oui | Non | Agent nécessaire pour superviser le système d’exploitation invité et les workflows.  |
|Cloud Shell | Non | Non | Non |  |
|Cognitive Services | Oui | Oui | Non |  |
|Container Instances | Oui | Non | Non |  |
|Container Registry | Oui | Oui | Non |  |
|Réseau de distribution de contenu (CDN) | Non | Oui | Non |  |
|Cosmos DB | Oui | Oui | [Oui](insights/cosmosdb-insights-overview.md) |  |
|Cost Management | Non | Non | Non |  |
|Data Box | Non | Non | Non |  |
|Catalogue de données Gen2 | Non | Non | Non |  |
|Explorateur de données | Oui | Oui | Non |  |
|Data Factory | Oui | Oui | Non |  |
|Data Factory v2 | Non | Oui | Non |  |
|Partage de données | Non | Non | Non |  |
|Database for MariaDB | Oui | Oui | Non |  |
|Database pour MySQL | Oui | Oui | Non |  |
|Database pour PostgreSQL | Oui | Oui | Non |  |
|Database Migration Service | Non | Non | Non |  |
|Databricks | Non | Oui | Non |  |
|Protection DDOS | Oui | Oui | Non |  |
|DevOps | Non | Non | Non |  |
|DNS | Oui | Non | Non |  |
|Noms de domaine | Non | Non | Non |  |
|DPS | Non | Non | Non |  |
|Dynamics 365 Customer Engagement | Non | Non | Non |  |
|Dynamics 365 Finance and Operations | Non | Non | Non |  |
|Event Grid | Oui | Non | Non |  |
|Event Hubs | Oui | Oui | Non |  |
|ExpressRoute | Oui | Oui | Non |  |
|Pare-feu | Oui | Oui | Non |  |
|Front Door | Oui | Oui | Non |  |
|Fonctions | Oui | Oui | Non |  |
|HDInsight | Non | Oui | Non |  |
|HPC Cache | Non | Non | Non |  |
|Information Protection | Non | Oui | Non |  |
|Intune | Non | Oui | Non |  |
|IoT Central | Non | Non | Non |  |
|IoT Hub | Oui | Oui | Non |  |
|Key Vault | Oui | Oui | [Oui](./insights/key-vault-insights-overview.md) |  |
|Kubernetes Service (AKS) | Non | Non | [Oui](insights/container-insights-overview.md)  |  |
|Load Balancer | Oui | Non | Non |  |
|Logic Apps | Oui | Oui | Non |  |
|Machine Learning Service | Non | Non | Non |  |
|Applications gérées  | Non | Non | Non |  |
|Cartes  | Non | Non | Non |  |
|Media Services | Oui | Oui | Non |  |
|Bureau géré Microsoft | Non | Non | Non |  |
|Microsoft PowerApps | Non | Non | Non |  |
|Microsoft Social Engagement | Non | Non | Non |  |
|Microsoft Stream | Oui | Oui | Non |  |
|Migrer | Non | Non | Non |  |
|Azure Multi-Factor Authentication | Non | Oui | Non |  |
|Network Watcher | Oui | Oui | Non |  |
|Notification Hubs | Oui | Non | Non |  |
|Ouvrir des jeux de données | Non | Non | Non |  |
|Stratégie | Non | Non | Non |  |
|Power Automate | Non | Non | Non |  |
|Power BI Embedded | Oui | Oui | Non |  |
|Private Link | Non | Non | Non |  |
|Project Spool Communication Platform | Non | Non | Non |  |
|Red Hat OpenShift | Non | Non | Non |  |
|Cache Redis | Oui | Oui | [Oui](insights/redis-cache-insights-overview.md) | |
|Resource Graph | Non | Non | Non |  |
|Gestionnaire de ressources | Non | Non | Non |  |
|Retail Search – by Bing | Non | Non | Non |  |
|Recherche | Oui | Oui | Non |  |
|Service Bus | Oui | Oui | Non |  |
|Service Fabric | Non | Oui | Non | Agent nécessaire pour superviser le système d’exploitation invité et les workflows.  |
|Portail d’inscription | Non | Non | Non |  |
|Site Recovery | Non | Oui | Non |  |
|Service Spring Cloud | Non | Non | Non |  |
|Azure Synapse Analytics | Oui | Oui | Non |  |
|SQL Database | Oui | Oui | Non |  |
|SQL Server Stretch Database | Oui | Oui | Non |  |
|Pile | Non | Non | Non |  |
|Stockage | Oui | Non | [Oui](insights/storage-insights-overview.md) |  |
|Cache de stockage | Non | Non | Non |  |
|Services de synchronisation de stockage | Non | Non | Non |  |
|Stream Analytics | Oui | Oui | Non |  |
|Time Series Insights | Oui | Oui | Non |  |
|TINA | Non | Non | Non |  |
|Traffic Manager | Oui | Oui | Non |  |
|Impression universelle | Non | Non | Non |  |
|Virtual Machine Scale Sets | Non | Oui | [Oui](insights/vminsights-overview.md) | Agent nécessaire pour superviser le système d’exploitation invité et les workflows. |
|Virtual Machines | Oui | Oui | [Oui](insights/vminsights-overview.md) | Agent nécessaire pour superviser le système d’exploitation invité et les workflows. |
|Réseau virtuel | Oui | Oui | [Oui](insights/network-insights-overview.md) |  |
|Réseau virtuel - Journaux de flux NSG | Non | Oui | Non |  |
|Passerelle VPN | Oui | Oui | Non |  |
|Windows Virtual Desktop | Non | Non | Non |  |

## <a name="virtual-machine-agents"></a>Agents de machine virtuelle
Le tableau suivant répertorie les agents qui peuvent collecter des données à partir du système d’exploitation invité des machines virtuelles et envoyer des données à Monitor. Chaque agent peut collecter des données différentes et les envoyer à des métriques ou des journaux dans Azure Monitor. 

Pour plus d’informations sur les données que chaque agent peut collecter, consultez [Vue d’ensemble des agents Azure Monitor](platform/agents-overview.md).

| Agent |  Mesures | Journaux d’activité |
|:---|:---|:---|:---|
| [Agent Azure Monitor (préversion)](platform/azure-monitor-agent-overview.md) | Oui | Oui |
| [Agent Log Analytics](platform/log-analytics-agent.md) | Non | Oui|
| [Extension de diagnostic](platform/diagnostics-extension-overview.md) | Oui | Non |
| [Agent Telegraf](platform/collect-custom-metrics-linux-telegraf.md) | Oui | Non |
| [Agent de dépendances](insights/vminsights-enable-overview.md) | Non | Oui |


## <a name="product-integrations"></a>Intégrations de produit
Les services et solutions figurant dans le tableau suivant stockent leurs données dans un espace de travail Log Analytics afin qu’elles puissent être analysées avec les autres données de journal collectées par Azure Monitor.

| Produit/Service | Description |
|:---|:---|
| [Azure Automation](../automation/index.yml) | Gérez les mises à jour du système d’exploitation et effectuez le suivi des modifications sur les ordinateurs Windows et Linux. Consultez [Change Tracking](../automation/change-tracking/overview.md) et [Update Management](../automation/update-management/overview.md). |
| [Azure Information Protection ](/azure/information-protection/) | Classifiez et éventuellement protégez les documents et les e-mails. Consultez [Rapports centraux d’Azure Information Protection](/azure/information-protection/reports-aip#configure-a-log-analytics-workspace-for-the-reports). |
| [Centre de sécurité Azure](../security-center/index.yml) | Collectez et analysez les événements de sécurité et effectue une analyse des menaces. Consultez [Collecte de données dans Azure Security Center](../security-center/security-center-enable-data-collection.md). |
| [Azure Sentinel](../sentinel/index.yml) | Se connecte à différentes sources, notamment Office 365 et Amazon Web Services CloudTrail. Voir [Connecter des sources de données](../sentinel/connect-data-sources.md). |
| [Microsoft Intune](/intune/) | Créez un paramètre de diagnostic pour envoyer des journaux à Azure Monitor. Consultez [Envoyer les données de journal à des comptes de stockage, des hubs d’événements ou Log Analytics dans Intune (préversion)](/intune/fundamentals/review-logs-using-azure-monitor).  |
| Réseau  | [Network Performance Monitor](insights/network-performance-monitor.md) : supervisez la connectivité et les performances réseau aux points de terminaison de service et d’application.<br>[Azure Application Gateway](insights/azure-networking-analytics.md#azure-application-gateway-analytics) : analysez les journaux et les métriques d’Azure Application Gateway.<br>[Traffic Analytics](../network-watcher/traffic-analytics.md) : analyse les journaux de flux de groupe de sécurité réseau Network Watcher pour fournir des informations sur le flux de trafic dans votre cloud Azure. |
| [Office 365](insights/solution-office-365.md) | Supervisez votre environnement Office 365. Version mise à jour avec une intégration améliorée disponible par le biais d’Azure Sentinel. |
| [SQL Analytics](insights/azure-sql.md) | Supervisez à grande échelle les performances des bases de données Azure SQL Database et des instances Azure SQL Managed Instance entre plusieurs abonnements. |
| [Surface Hub](insights/surface-hubs.md) | Effectuez le suivi de l’intégrité et de l’utilisation des appareils Surface Hub. |
| [System Center Operations Manager](/system-center/scom) | Collectez les données des agents Operations Manager en connectant leur groupe d’administration à Azure Monitor. Consultez [Connecter Operations Manager à Azure Monitor](platform/om-agents.md).<br> Évaluez les risques et l’intégrité de votre groupe d’administration System Center Operations Manager avec la solution d’[évaluation d’Operations Manager](insights/scom-assessment.md). |
| [Salles Microsoft Teams](/microsoftteams/room-systems/azure-monitor-deploy) | Gestion intégrée de bout en bout des appareils Salles Microsoft Teams. |
| [Visual Studio App Center](/appcenter/) | Générez, testez et distribuez des applications, puis supervisez leur état et leur utilisation. Consultez See [Commencer à analyser votre application mobile avec App Center et Application Insights](learn/mobile-center-quickstart.md). |
| Windows | [Windows Update Compliance](/windows/deployment/update/update-compliance-get-started) : évaluez les mises à niveau de votre bureau Windows.<br>[Desktop Analytics](/configmgr/desktop-analytics/overview) : s’intègre à Configuration Manager pour fournir des insights et une intelligence pour prendre des décisions plus éclairées concernant l’état de préparation aux mises à jour de vos clients Windows. |



## <a name="other-solutions"></a>Autres solutions
D’autres solutions sont disponibles pour la supervision des différents services et applications ; toutefois, en raison de l’arrêt de leur développement, il est possible qu’elles ne soient pas disponibles dans toutes les régions. Elles sont couvertes par le contrat de niveau de service d’ingestion de données Azure Log Analytics.

| Solution | Description |
|:---|:---|
| [Contrôle d’intégrité d’Active Directory](insights/ad-assessment.md) | Évaluez les risques et l’intégrité de vos environnements Active Directory. |
| [Active Directory Replication Status](insights/ad-replication-status.md) | Supervise régulièrement votre environnement Active Directory pour déterminer si des échecs de réplication se produisent. |
| [Analyse Log Analytics des activités](platform/activity-log.md#activity-log-analytics-monitoring-solution) | Affichez les entrées du journal d’activité. |
| [DNS Analytics (préversion)](insights/dns-analytics.md) | Collecte, analyse et met en corrélation les journaux d’audit et d’analyse DNS Windows et d’autres données connexes de vos serveurs DNS. |
| [Cloud Foundry](../cloudfoundry/cloudfoundry-oms-nozzle.md) | Collectez, consultez et analysez l’intégrité et les métriques de performance de Cloud Foundry, et ce, sur plusieurs déploiements. |
| [Containers](insights/containers.md) | Affichez et gérez les hôtes de conteneur Docker et Windows. |
| [Évaluations à la demande](/services-hub/health/getting_started_with_on_demand_assessments) | Évaluez et optimisez la disponibilité, la sécurité et les performances de vos environnements de technologie Microsoft locaux, hybrides et cloud. |
| [Contrôle d’intégrité de SQL](insights/sql-assessment.md) | Évaluez les risques et l’intégrité de vos environnements SQL Server.  |
| [Données de communication](insights/wire-data.md) | Données de performances et réseau regroupées, collectées à partir d’ordinateurs connectés à Windows et à Linux avec l’agent Log Analytics. |

## <a name="third-party-integration"></a>Intégration tierce

| Solution | Description |
|:---|:---|
| [ITSM](platform/itsmc-overview.md) | Le connecteur de gestion des services informatiques (ITSMC) vous permet de connecter Azure et un produit/service ITSM pris en charge.  |


## <a name="resources-outside-of-azure"></a>Ressources situées en dehors d’Azure
Azure Monitor peut collecter des données à partir de ressources situées en dehors d’Azure à l’aide des méthodes listées dans le tableau suivant.

| Ressource | Méthode |
|:---|:---|
| Applications | Supervisez les applications web situées en dehors d’Azure à l’aide d’Application Insights. Voir [Présentation d’Application Insights](./app/app-insights-overview.md). |
| Machines virtuelles | Utilisez les agents pour collecter des données à partir du système d’exploitation invité des machines virtuelles dans d’autres environnements cloud ou en local. Consultez [Vue d’ensemble des agents Azure Monitor](platform/agents-overview.md). |
| Client d’API REST | Différentes API sont disponibles pour écrire des données dans les métriques et journaux Azure Monitor à partir de n’importe quel client d’API REST. Consultez [Envoyer des données de journal à Azure Monitor à l’aide de l’API Collecte de données HTTP](platform/data-collector-api.md) pour les journaux et [Envoyer les métriques personnalisées d’une ressource Azure à un magasin de métriques Azure Monitor à l’aide d’une API REST](platform/metrics-store-custom-rest-api.md) pour les métriques. |



## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur la [plateforme de données Azure Monitor qui stocke les journaux et les métriques collectés par les insights et les solutions](platform/data-platform.md).
- Suivez un [tutoriel sur la supervision d’une ressource Azure](learn/tutorial-resource-logs.md).
- Suivez un [tutoriel sur l’écriture d’une requête de journal pour analyser des données dans les journaux Azure Monitor](learn/tutorial-resource-logs.md).
- Suivez un [tutoriel sur la création d’un graphique de métriques pour analyser des données dans les métriques Azure Monitor](learn/tutorial-metrics-explorer.md).

