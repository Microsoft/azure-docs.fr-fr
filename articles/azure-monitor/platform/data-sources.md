---
title: Sources de données dans Azure Monitor | Microsoft Docs
description: Décrit les données disponibles pour surveiller l’intégrité et les performances de vos ressources Azure, et des applications s’exécutant sur celles-ci.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/06/2020
ms.openlocfilehash: c6b976ca1beb9beb78f7c56d7ae512c1f499c2c5
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186522"
---
# <a name="sources-of-monitoring-data-for-azure-monitor"></a>Sources des données de surveillance pour Azure Monitor
Azure Monitor repose sur une [plateforme commune de données de surveillance](data-platform.md) qui inclut des [journaux](data-platform-logs.md) et des [métriques](data-platform-metrics.md). La collecte des données dans cette plate-forme permet d'analyser ensemble des données provenant de plusieurs ressources à l'aide d'un ensemble commun d'outils dans Azure Monitor. Les données de surveillance peuvent également être envoyées à d'autres endroits pour étayer certains scénarios, et certaines ressources peuvent écrire des données sur d'autres sites afin de consigner ces informations dans des journaux ou des métriques.

Cet article décrit les différentes sources de données de surveillance recueillies par Azure Monitor en plus des données de surveillance créées par les ressources Azure. Vous pouvez utiliser les liens fournis afin d’obtenir des informations détaillées sur la configuration requise pour collecter ces données à différents endroits.

## <a name="application-tiers"></a>Couches de l'application

Les sources de données de surveillance des applications Azure peuvent être organisées en niveaux, les niveaux les plus élevés représentant votre application elle-même, et les niveaux inférieurs des composants de la plate-forme Azure. La méthode d'accès aux données de chaque niveau varie. Les niveaux d'application sont résumés dans le tableau ci-dessous, et les sources des données de surveillance pour chaque niveau sont présentées dans les sections suivantes. Consultez [Emplacements des données de surveillance dans Azure](../monitor-reference.md) pour une description de chaque emplacement de données et comment accéder à ses données.


![Niveaux de surveillance](../media/overview/overview.png)


### <a name="azure"></a>Azure
Le tableau suivant décrit brièvement les niveaux d'application spécifiques à Azure. Suivez le lien pour obtenir plus de détails sur chacun de ces niveaux dans les sections ci-dessous.

| Niveau | Description | Méthode de collection |
|:---|:---|:---|
| [Locataire Azure](#azure-tenant) | données concernant le fonctionnement des services Azure au niveau du locataire, tels qu’Azure Active Directory. | Affichez les données AAD dans le portail ou configurez la collection vers Azure Monitor à l'aide d'un paramètre de diagnostic de locataire. |
| [Abonnement Azure](#azure-subscription) | Données relatives à l’intégrité et à la gestion des services inter-ressources de votre abonnement Azure, par exemple Resource Manager et Service Health. | Afficher les données dans le portail ou configurer la collection pour Azure Monitor à l'aide d'un profil de journal. |
| [Ressources Azure](#azure-resources) |  Données sur le fonctionnement et les performances de chaque ressource Azure. | Métriques collectées automatiquement, affichées dans Metrics Explorer.<br>Configurez les paramètres de diagnostic pour collecter des journaux dans Azure Monitor.<br>Solutions de surveillance et informations disponibles pour une surveillance plus détaillée de types de ressources spécifiques. |

### <a name="azure-other-cloud-or-on-premises"></a>Azure, autre cloud ou local 
Le tableau suivant décrit brièvement les niveaux d'application utilisables dans Azure, dans un autre cloud ou en local. Suivez le lien pour obtenir plus de détails sur chacun de ces niveaux dans les sections ci-dessous.

| Niveau | Description | Méthode de collection |
|:---|:---|:---|
| [Système d’exploitation (invité)](#operating-system-guest) | Données concernant le système d'exploitation et les ressources de calcul. | Installez l’agent Log Analytics pour collecter les sources de données client dans Azure Monitor et l’agent de dépendances pour collecter les dépendances prenant en charge Azure Monitor pour machines virtuelles.<br>Pour les machines virtuelles Azure, installez l’extension Azure Diagnostic pour collecter les journaux et les métriques dans Azure Monitor. |
| [Code de l’application](#application-code) | Données sur les performances et les fonctionnalités de l'application et du code réels, y compris les traces de performances, les journaux d'application et la télémétrie utilisateur. | Instrumentez votre code pour collecter des données dans Application Insights. |
| [Sources personnalisées](#custom-sources) | Données provenant de services externes ou d'autres composants ou appareils. | Collectez les données de journaux ou de métriques dans Azure Monitor à partir de n'importe quel client REST. |

## <a name="azure-tenant"></a>Client Azure
Les données de télémétrie liées à votre locataire Azure sont collectées à partir des services à l’échelle du locataire, tels qu’Azure Active Directory.

![Collecte de locataire Azure](media/data-sources/tenant.png)

### <a name="azure-active-directory-audit-logs"></a>Journaux d’audit d’Azure Active Directory
Les [rapports Azure Active Directory](../../active-directory/reports-monitoring/overview-reports.md) contiennent l’historique des connexions et la piste d’audit des modifications apportées à un client particulier. 

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journaux Azure Monitor | Configurez les journaux Azure AD à collecter dans Azure Monitor pour les analyser avec d'autres données de surveillance. | [Intégrer des journaux d’activité Azure AD aux journaux d’activité Azure Monitor (préversion)](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) |
| Stockage Azure | Exportez les journaux Azure AD vers Stockage Azure à des fins d’archivage. | [Tutoriel : Archiver des journaux d’activité Azure AD dans un compte de stockage Azure (préversion)](../../active-directory/reports-monitoring/quickstart-azure-monitor-route-logs-to-storage-account.md) |
| Event Hub | Diffusez en continu des journaux Azure AD sur d’autres emplacements à l'aide d’Event Hubs. | [Tutoriel : Diffuser en continu des journaux d’activité Azure Active Directory sur un hub d’événements Azure (préversion)](../../active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub.md). |



## <a name="azure-subscription"></a>Abonnement Azure
Télémétrie relative à l’intégrité et au fonctionnement de votre abonnement Azure.

![Abonnement Azure](media/data-sources/azure-subscription.png)

### <a name="azure-activity-log"></a>Journal d’activité Azure 
Le [journal d’activité Azure](platform-logs-overview.md) comprend des enregistrements relatifs à l’intégrité du service ainsi qu’aux changements de configuration apportés aux ressources de votre abonnement Azure. Le journal d’activité est disponible pour toutes les ressources Azure et représente leur affichage _externe_.

| Destination | Description | Informations de référence |
|:---|:---|
| Journal d’activité | Le journal d'activité est collecté dans sa propre banque de données que vous pouvez consulter depuis le menu Azure Monitor ou utiliser pour créer des alertes de journal d'activité. | [Interroger le journal d’activité dans le portail Azure](./activity-log.md#view-the-activity-log) |
| Journaux Azure Monitor | Configurez les journaux Azure Monitor pour collecter le journal d'activité et l'analyser avec d'autres données de surveillance. | [Collecter et analyser les journaux d’activité Azure dans l’espace de travail Log Analytics dans Azure Monitor](./activity-log.md) |
| Stockage Azure | Exportez le journal d'activité vers Stockage Azure à des fins d’archivage. | [Archiver le journal d’activité](./resource-logs.md#send-to-azure-storage)  |
| Event Hubs | Diffuser en continu le journal d’activité sur d’autres emplacements à l'aide d’Event Hubs | [Diffuser en continu le journal d’activité vers Event Hubs](./resource-logs.md#send-to-azure-event-hubs). |

### <a name="azure-service-health"></a>Azure Service Health
[Azure Service Health](../../service-health/service-health-overview.md) fournit des informations sur l’intégrité des services Azure compris dans votre abonnement, dont dépendent votre application et vos ressources.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journal d’activité<br>Journaux Azure Monitor | Les enregistrements de Service Health étant stockés dans le journal d’activité Azure, vous pouvez les afficher dans le portail Azure ou utiliser ce journal d'activité pour effectuer d’autres activités. | [Afficher les notifications d’intégrité du service à l’aide du Portail Azure](../../service-health/service-notifications.md) |


## <a name="azure-resources"></a>Ressources Azure
Les journaux de diagnostic au niveau métriques et ressources fournissent des informations sur le fonctionnement _interne_ des ressources Azure. Ceux-ci sont disponibles pour la plupart des services d'Azure, et les solutions de surveillance et les informations recueillies collectent des données supplémentaires pour des services particuliers.

![Collection de ressources Azure](media/data-sources/data-source-azure-resources.svg)


### <a name="platform-metrics"></a>Métriques de plateforme 
La plupart des services Azure envoient des [métriques de plateforme](data-platform-metrics.md) reflétant leurs performances et leur fonctionnement directement vers la base de données de métriques. Les [métriques spécifiques varient pour chaque type de ressource](metrics-supported.md). 

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Métriques Azure Monitor | Les métriques de la plate-forme seront inscrites dans la base de données de métriques Azure Monitor, sans aucune configuration. Accédez aux métriques de la plate-forme depuis Metrics Explorer.  | [Bien démarrer avec Azure Metrics Explorer](metrics-getting-started.md)<br>[Mesures prises en charge avec Azure Monitor](metrics-supported.md) |
| Journaux Azure Monitor | Copiez les métriques de la plate-forme dans les journaux pour l'analyse des tendances et d'autres analyses à l'aide de Log Analytics. | [Diagnostics Azure directement dans Log Analytics](./resource-logs.md#send-to-log-analytics-workspace) |
| Event Hubs | Diffusez en continu des métriques sur d’autres emplacements à l'aide d’Event Hubs. |[Diffuser des données de surveillance Azure vers un hub d’événements pour les utiliser dans un outil externe](stream-monitoring-data-event-hubs.md) |

### <a name="resource-logs"></a>Journaux d’activité de ressources
Les [journaux de ressources](platform-logs-overview.md) fournissent des insights sur le fonctionnement _interne_ d’une ressource Azure.  Les journaux de ressources sont créés automatiquement, mais vous devez créer un paramètre de diagnostic pour spécifier une destination pour eux.

La configuration requise et le contenu des journaux de ressources varient selon le type de ressource, et tous les services ne créent pas encore de tels journaux. Consultez [Services, schémas et catégories pris en charge pour les journaux de ressources Azure](./resource-logs-schema.md) pour plus d’informations sur chaque service et des liens vers les procédures de configuration détaillées. Si le service n’est pas répertorié dans cet article, alors il ne crée actuellement pas de journaux de ressources.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journaux Azure Monitor | Envoyez les journaux d’activité de ressources aux journaux Azure Monitor pour analyse avec d’autres données de journaux collectées. | [Collecter les journaux d’activité de ressources Azure dans l’espace de travail Log Analytics dans Azure Monitor](./resource-logs.md#send-to-azure-storage) |
| Stockage | Envoyez les journaux d’activité de ressources vers Stockage Azure à des fins d’archivage. | [Archiver les journaux d’activité de ressources Azure](./resource-logs.md#send-to-log-analytics-workspace) |
| Event Hubs | Diffusez en continu des journaux d’activité de ressources sur d’autres emplacements à l’aide d’Event Hubs. |[Streaming des journaux d’activité de ressources Azure sur un hub d’événements](./resource-logs.md#send-to-azure-event-hubs) |

## <a name="operating-system-guest"></a>Système d’exploitation (invité)
Les ressources de calcul dans Azure, dans d’autres clouds et en local ont un système d’exploitation invité à surveiller. Avec l’installation d’un ou plusieurs agents, vous pouvez collecter la télémétrie de l’invité dans Azure Monitor pour l’analyser avec les mêmes outils de surveillance qu’utilisent les services Azure eux-mêmes.

![Collection de ressources de calcul Azure](media/data-sources/compute-resources.png)

### <a name="azure-diagnostic-extension"></a>Extension Azure Diagnostics
L'activation de l'extension Azure Diagnostics pour les machines virtuelles Azure vous permet de collecter les journaux et les métriques du système d'exploitation invité des ressources de calcul Azure, y compris les rôles Web et Worker Azure Cloud Service (classique), les machines virtuelles, les groupes de machines virtuelles identiques et Service Fabric.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Stockage | L’extension Diagnostics Azure écrit toujours dans un compte Stockage Azure. | [Installer et configurer l’extension Diagnostics Azure pour Azure (WAD)](diagnostics-extension-windows-install.md)<br>[Utiliser l’extension de diagnostic Linux pour superviser les métriques et les journaux](../../virtual-machines/extensions/diagnostics-linux.md) |
| Métriques Azure Monitor | Lorsque vous configurez l'extension Diagnostics Extension pour collecter les compteurs de performances, ces informations sont écrites dans la base de données de métriques Azure Monitor. | [Envoyer des métriques de système d’exploitation invité vers le magasin de métriques d’Azure Monitor à l’aide d’un modèle Resource Manager pour une machine virtuelle Windows](collect-custom-metrics-guestos-resource-manager-vm.md) |
| Event Hubs | Configurez l'extension Diagnostics pour diffuser en continu les données sur d'autres emplacements à l'aide d’Event Hubs.  | [Streaming des données Diagnostics Azure à l’aide d’Event Hubs](diagnostics-extension-stream-event-hubs.md)<br>[Utiliser l’extension de diagnostic Linux pour superviser les métriques et les journaux](../../virtual-machines/extensions/diagnostics-linux.md) |
| Journaux Application Insights | Collectez les journaux et les compteurs de performances de la ressource de calcul prenant en charge votre application pour les analyser avec d'autres données de l'application. | [Envoyer des données de diagnostic de service cloud, de machine virtuelle ou de Service Fabric à Application Insights](diagnostics-extension-to-application-insights.md) |


### <a name="log-analytics-agent"></a>Agent Log Analytics 
Installez l'agent Log Analytics pour une surveillance et une gestion complètes de vos machines virtuelles Windows ou Linux. La machine virtuelle peut s’exécuter dans Azure, dans un autre cloud ou localement.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journaux Azure Monitor | L'agent Log Analytics se connecte à Azure Monitor soit directement, soit par l'intermédiaire du System Center Operations Manager et vous permet de collecter des données à partir de sources de données que vous configurez ou de solutions de surveillance qui fournissent des informations supplémentaires sur les applications exécutées sur la machine virtuelle. | [Sources de données d’agent dans Azure Monitor](agent-data-sources.md)<br>[Connecter Operations Manager à Azure Monitor](om-agents.md) |
| Stockage de machine virtuelle | Azure Monitor pour machines virtuelles utilise l’agent Log Analytics pour stocker les informations d’intégrité dans un emplacement personnalisé. Pour en savoir plus, consultez la section suivante.  |


### <a name="azure-monitor-for-vms"></a>Azure Monitor pour machines virtuelles 
[Azure Monitor pour machines virtuelles](../insights/vminsights-overview.md) fournit une expérience de supervision personnalisée pour les machines virtuelles en offrant des fonctionnalités qui surpassent les fonctionnalités de base d’Azure Monitor. Il nécessite un agent de dépendances sur les machines virtuelles Windows et Linux qui s’intègrent à l’agent Log Analytics pour collecter des données découvertes relatives aux processus en cours d’exécution sur la machine virtuelle et aux dépendances de processus externes.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journaux Azure Monitor | Stocke les données sur les processus et les dépendances sur l'agent. | [Utilisation de la fonctionnalité Map d’Azure Monitor pour machines virtuelles (préversion) dans le but de comprendre les composants d’application](../insights/vminsights-maps.md) |



## <a name="application-code"></a>Code de l’application
La surveillance détaillée des applications dans Azure Monitor se fait avec [Application Insights](/azure/application-insights/), qui collecte les données des applications s'exécutant sur différentes plateformes. L’application peut s’exécuter dans Azure, dans un autre cloud ou localement.

![Collecte de données d’application](media/data-sources/applications.png)


### <a name="application-data"></a>Données d'application
Lorsque vous activez Application Insights pour une application en installant un package d’instrumentation, la solution collecte des métriques et journaux d’activité liés aux performances et au fonctionnement de l’application. Application Insights stocke les données qu'il collecte dans la même plate-forme de données Azure Monitor utilisée par d'autres sources de données. La solution inclut des outils étendus pour analyser ces données, mais vous pouvez également les analyser avec des données provenant d’autres sources à l’aide d’outils tels que Metrics Explorer et Log Analytics.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journaux Azure Monitor | Données opérationnelles concernant votre application, notamment les pages consultées, les demandes d'application, les exceptions et les traces. | [Analyser les données de journal dans Azure Monitor](../log-query/log-query-overview.md) |
|                    | Informations sur les dépendances entre les composants d’application pour prendre en charge la cartographie d'application et la corrélation de la télémétrie. | [Corrélation de télémétrie dans Application Insights](../app/correlation.md) <br> [Plan de l’application](../app/app-map.md) |
|            | Résultats de tests de disponibilité qui évaluent la disponibilité et la réactivité de votre application à partir de différents emplacements sur l’Internet public. | [Analyse de la disponibilité et de la réactivité d'un site Web](../app/monitor-web-app-availability.md) |
| Métriques Azure Monitor | Application Insights collecte des métriques décrivant les performances et le fonctionnement de l'application en plus des métriques personnalisées que vous définissez dans votre application avec la base de données de métriques Azure Monitor. | [Métriques basées sur le journal et pré-agrégées dans Application Insights](../app/pre-aggregated-metrics-log-metrics.md)<br>[API Application Insights pour les événements et les mesures personnalisés](../app/api-custom-events-metrics.md) |
| Stockage Azure | Envoyez les données d'application à Stockage Azure à des fins d’archivage. | [Exporter la télémétrie depuis Application Insights](../app/export-telemetry.md) |
|            | Les détails des tests de disponibilité sont stockés dans Stockage Azure. Utilisez Application Insights sur le portail Azure pour télécharger des analyses locales. Les résultats des tests de disponibilité sont stockés dans les journaux d’activité Azure Monitor. | [Analyse de la disponibilité et de la réactivité d'un site Web](../app/monitor-web-app-availability.md) |
|            | Les données de la trace du profileur sont stockées dans Stockage Azure. Utilisez Application Insights sur le portail Azure pour télécharger des analyses locales.  | [Profiler des applications de production dans Azure avec Application Insights](../app/profiler-overview.md) 
|            | Les données de l’instantané de débogage capturées pour un sous-ensemble d'exceptions sont stockées dans Stockage Azure. Utilisez Application Insights sur le portail Azure pour télécharger des analyses locales.  | [Fonctionnement des captures instantanées](../app/snapshot-debugger.md#how-snapshots-work) |

## <a name="monitoring-solutions-and-insights"></a>Solutions de supervision et Insights
Les [solutions de supervision](../insights/solutions.md) et [Insights](../monitor-reference.md) collectent des données pour fournir des insights supplémentaires sur le fonctionnement d’un service ou d’une application particuliers. Elles peuvent traiter des ressources de différents niveaux d'application et même sur plusieurs niveaux.

### <a name="monitoring-solutions"></a>Solutions de supervision

| Destination | Description | Informations de référence
|:---|:---|:---|
| Journaux Azure Monitor | Les solutions de surveillance recueillent des données dans les journaux d’activité Azure Monitor, qui peuvent les analyser à l’aide du langage de requête, ou dans des [affichages](view-designer.md) qui sont généralement inclus dans la solution. | [Détails sur la collecte de données pour les solutions de gestion dans Azure](../monitor-reference.md) |


### <a name="azure-monitor-for-containers"></a>Azure Monitor pour des conteneurs
[Azure Monitor pour des conteneurs](../insights/container-insights-overview.md) fournit une expérience de surveillance personnalisée pour [Azure Kubernetes Service (AKS)](../../aks/index.yml). Elle recueille des données supplémentaires sur ces ressources décrites dans le tableau suivant.

| Destination | Description | Informations de référence |
|:---|:---|:---|
| Journaux Azure Monitor | Stocke les données de surveillance pour AKS, y compris l'inventaire, les journaux et les événements. Les données métriques sont également stockées dans les journaux afin de tirer parti de sa fonctionnalité d'analyse dans le portail. | [Comprendre les performances du cluster AKS avec Azure Monitor pour conteneurs](../insights/container-insights-analyze.md) |
| Métriques Azure Monitor | Les données métriques sont stockées dans la base de données de métriques pour gérer la visualisation et les alertes. | [Afficher les métriques du conteneur dans Metrics Explorer](../insights/container-insights-analyze.md#view-container-metrics-in-metrics-explorer). |
| Azure Kubernetes Service | Fournit un accès direct à vos journaux de conteneur (stdout/stderror), événements et mesures de pod Azure Kubernetes Service (AKS) dans le portail. | [Comment afficher les journaux, événements et mesures de pod Kubernetes en temps réel ](../insights/container-insights-livedata-overview.md) |

### <a name="azure-monitor-for-vms"></a>Azure Monitor pour machines virtuelles
[Azure Monitor pour machines virtuelles](../insights/vminsights-overview.md) fournit une expérience personnalisée pour la surveillance des machines virtuelles. Une description des données recueillies par Azure Monitor pour les machines virtuelles est incluse dans la section [Système d'exploitation (invité)](#operating-system-guest) ci-dessus.

## <a name="custom-sources"></a>Sources personnalisées
Outre les niveaux standard d’une application, vous devrez peut-être surveiller d’autres ressources qui ont des données de télémétrie qui ne peuvent pas être collectées avec les autres sources de données. Pour ces ressources, écrivez ces données dans les métriques ou les journaux Azure Monitor à l'aide d'une API Azure Monitor.

![Collection personnalisée](media/data-sources/custom.png)

| Destination | Méthode | Description | Informations de référence |
|:---|:---|:---|:---|
| Journaux Azure Monitor | API du collecteur de données | Collectez les données du journal à partir de n'importe quel client REST et stockez-les dans l'espace de travail Log Analytics. | [Transmettre des données à Azure Monitor avec l’API Collecteur de données HTTP (préversion publique)](data-collector-api.md) |
| Métriques Azure Monitor | API de métriques personnalisée | Collectez les données métriques à partir de n'importe quel client REST et stockez-les dans la base de données de métriques Azure Monitor. | [Envoyer les métriques personnalisées d’une ressource Azure à un magasin de métriques Azure Monitor à l’aide d’une API REST](metrics-store-custom-rest-api.md) |


## <a name="other-services"></a>Autres services
D'autres services Azure écrivent les données sur la plate-forme de données Azure Monitor. Ceci vous permet d'analyser les données collectées par ces services avec les données collectées par Azure Monitor et de bénéficier des mêmes outils d'analyse et de visualisation.

| Service | Destination | Description | Informations de référence |
|:---|:---|:---|:---|
| [Centre de sécurité Azure](../../security-center/index.yml) | Journaux Azure Monitor | Azure Security Center stocke les données de sécurité qu'il collecte dans un espace de travail Log Analytics afin de les analyser avec d'autres données collectées par Azure Monitor.  | [Collecte de données dans Azure Security Center](../../security-center/security-center-enable-data-collection.md) |
| [Azure Sentinel](../../sentinel/index.yml) | Journaux Azure Monitor | Azure Sentinel stocke dans un espace de travail Log Analytics les données de sécurité qu'il collecte de différentes source de données afin de les analyser avec d'autres données collectées par Azure Monitor.  | [Connecter des sources de données](../../sentinel/quickstart-onboard.md) |


## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur les [types de données de supervision collectés par Azure Monitor](data-platform.md) et sur l’affichage et l’analyse de ces données.
- Énumérez les [différents endroits où les ressources Azure stockent les données](../monitor-reference.md) et comment vous pouvez y accéder.