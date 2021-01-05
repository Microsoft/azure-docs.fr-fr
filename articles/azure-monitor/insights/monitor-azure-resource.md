---
title: Superviser des ressources Azure avec Azure Monitor | Microsoft Docs
description: Décrit comment collecter et analyser des données de supervision à partir de ressources dans Azure en utilisant Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/08/2019
ms.openlocfilehash: d365b13ef8fb7ed9676bb00919315dc6fc9773ee
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184584"
---
# <a name="monitoring-azure-resources-with-azure-monitor"></a>Supervision de ressources Azure avec Azure Monitor
Lorsque vous avez des applications critiques et des processus métier basés sur des ressources Azure, vous voulez superviser ces ressources pour connaître leur disponibilité, leurs performances et leur fonctionnement. Cet article décrit les données de supervision générées par les ressources Azure et comment vous pouvez utiliser les fonctionnalités d’Azure Monitor pour analyser ces données et créer des alertes.

> [!IMPORTANT]
> Cet article s’applique à tous les services Azure qui utilisent Azure Monitor. Les ressources de calcul, notamment les machines virtuelles et App Service, génèrent les mêmes données de supervision que celles décrites ici, mais disposent aussi d’un système d’exploitation invité également capable de générer des journaux et des métriques. Consultez la documentation sur la supervision de ces services pour plus d’informations sur la façon de collecter et d’analyser ces données.

## <a name="what-is-azure-monitor"></a>Qu’est-ce qu’Azure Monitor ?
Azure Monitor est un service de supervision de pile complète dans Azure qui fournit un ensemble complet de fonctionnalités pour superviser vos ressources Azure en plus des ressources figurant dans d’autres clouds et en local. La [plateforme de données Azure Monitor](../platform/data-platform.md) collecte des données dans des [journaux d’activité](../platform/data-platform-logs.md) et des [métriques](../platform/data-platform-metrics.md) et permet de les analyser ensemble à l’aide d’un ensemble complet d’outils de supervision. Consultez la liste complète des applications et services qui peuvent être supervisés par Azure Monitor dans [Quels sont les éléments supervisés par Azure Monitor ?](../monitor-reference.md)

Dès que vous créez une ressource Azure, Azure Monitor est activé et commence à collecter des métriques et des journaux d’activité que vous pouvez [consulter et analyser dans le portail Azure](#monitoring-in-the-azure-portal). Avec une certaine configuration, vous pouvez collecter des données de supervision supplémentaires et activer des fonctionnalités supplémentaires. Consultez [Données de supervision](#monitoring-data) ci-dessous, pour plus d’informations sur les exigences de configuration.


## <a name="costs-associated-with-monitoring"></a>Coûts associés à la supervision
Aucun coût n’est associé à l’analyse des données de supervision collectées par défaut. Notamment :

- Collecte des métriques de plateforme et analyse avec Metrics Explorer.
- Collecte du journal d’activité et analyse de ce dernier dans le Portail Azure.
- Création d’une règle d’alerte du journal d’activité.

Aucun coût Azure Monitor ne s’applique à la collecte et à l’exportation des journaux et des métriques, mais des coûts connexes peuvent être associés à la destination :

- Coûts associés à l’ingestion et à la conservation des données lors de la collecte des journaux et des métriques dans l’espace de travail Log Analytics. Consultez [Tarification Azure Monitor pour Log Analytics](https://azure.microsoft.com/pricing/details/monitor/).
- Coûts associés au stockage des données lors de la collecte des journaux et des métriques dans un compte de stockage Azure. Consultez [Tarification du Stockage Azure pour le stockage des objets blob](https://azure.microsoft.com/pricing/details/storage/blobs/).
- Coûts associés à la diffusion vers un hub d’événements lors du transfert des journaux et des métriques vers Azure Event Hubs. Consultez [Tarification Azure Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

Il peut y avoir des coûts Azure Monitor associés aux opérations ci-dessous. Consultez [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) :

- Exécution d’une requête de journal.
- Création d’une règle d’alerte de requête de journal ou de métrique.
- Envoi d’une notification à partir d’une règle d’alerte quelconque.
- Accès aux métriques via une API.

## <a name="monitoring-data"></a>Données de surveillance
Les ressources dans Azure génèrent les [journaux](../platform/data-platform-logs.md) et [métriques](../platform/data-platform-metrics.md) affichés dans le schéma suivant. Reportez-vous à la documentation de chaque service Azure pour obtenir les données spécifiques qu’il génère et les autres solutions ou insights qu’il fournit.

![Vue d’ensemble](media/monitor-azure-resource/logs-metrics.png)



- [Métriques de plateforme](../platform/data-platform-metrics.md) – Valeurs numériques qui sont collectées automatiquement à intervalles réguliers et qui décrivent un certain aspect d’une ressource à un moment donné. 
- [Journaux de ressources](../platform/platform-logs-overview.md) – Fournissent des insights sur les opérations effectuées au sein d’une ressource Azure (plan de données), par exemple l’obtention d’un secret à partir d’un coffre de clés ou l’envoi d’une demande à une base de données. Le contenu et la structure des journaux de ressources varient en fonction du service Azure et du type de ressource.
- [Journal d’activité](../platform/platform-logs-overview.md) – Fournit un insight sur les opérations effectuées sur chaque ressource Azure dans l’abonnement à partir de l’extérieur (plan de gestion), par exemple la création d’une ressource ou le démarrage d’une machine virtuelle. Ces informations répondent aux questions quoi, qui et quand concernant toutes les opérations d’écriture (PUT, POST, DELETE) effectuées sur les ressources dans le cadre de votre abonnement.


## <a name="configuration-requirements"></a>Exigences de configuration

### <a name="configure-monitoring"></a>Configuration de l’analyse
Certaines données de supervision sont collectées automatiquement, mais vous devrez peut-être adapter votre configuration en fonction de vos besoins. Consultez les informations ci-dessous pour obtenir des informations spécifiques pour chaque type de données de supervision.

- [Métriques de plateforme](../platform/data-platform-metrics.md) – Les métriques de plateforme sont collectées automatiquement dans [Métriques Azure Monitor](../platform/data-platform-metrics.md) sans qu’aucune configuration soit requise. Créez un paramètre de diagnostic pour envoyer des entrées aux journaux Azure Monitor ou les transférer en dehors d’Azure.
- [Journaux de ressources](../platform/platform-logs-overview.md) – Les journaux de ressources sont générés automatiquement par les ressources Azure, mais ne sont pas collectés sans paramètre de diagnostic.  Créez un paramètre de diagnostic pour envoyer des entrées aux journaux Azure Monitor ou les transférer en dehors d’Azure.
- [Journal d’activité](../platform/platform-logs-overview.md) – Le journal d’activité est collecté automatiquement sans aucune configuration requise et peut être affiché dans le portail Azure. Créez un paramètre de diagnostic pour les copier dans les journaux Azure Monitor ou pour les transférer en dehors d’Azure.

### <a name="log-analytics-workspace"></a>Espace de travail Log Analytics
La collecte de données dans les journaux d’activité Azure Monitor nécessite un espace de travail Log Analytics. Vous pouvez commencer à superviser votre service rapidement en créant un espace de travail, mais il peut être utile d’utiliser un espace de travail qui collecte des données à partir d’autres services. Consultez [Créer un espace de travail Log Analytics dans le Portail Azure](../learn/quick-create-workspace.md) pour plus d’informations sur la création d’un espace de travail et [Conception de votre déploiement de journaux Azure Monitor](../platform/design-logs-deployment.md) afin de déterminer la meilleure conception d’espace de travail pour vos besoins. Si vous utilisez un espace de travail existant dans votre organisation, vous aurez besoin des autorisations appropriées, comme décrit dans [Gérer l’accès aux données du journal et les espaces de travail dans Azure Monitor](../platform/manage-access.md). 





## <a name="diagnostic-settings"></a>Paramètres de diagnostic
Les paramètres de diagnostic définissent où les journaux de ressources et les métriques d’une ressource particulière doivent être envoyés. Les destinations possibles sont les suivantes :

- [Espace de travail Log Analytics](../platform/resource-logs.md#send-to-log-analytics-workspace) qui permet d’analyser les données avec d’autres données de supervision collectées par Azure Monitor à l’aide de requêtes de journal puissantes, ainsi que d’exploiter d’autres fonctionnalités Azure Monitor, telles que les alertes et les visualisations de journal. 
- [Hubs d’événements](../platform/resource-logs.md#send-to-azure-event-hubs) pour envoyer en streaming des données vers des systèmes externes tels que des solutions SIEM tierces et d’autres solutions Log Analytics. 
- [Compte de stockage Azure](../platform/resource-logs.md#send-to-azure-storage) qui est utile à des fins d’audit, d’analyse statique ou de sauvegarde.

Suivez la procédure indiquée dans [Créer un paramètre de diagnostic pour collecter des journaux et métriques de plateforme dans Azure](../platform/diagnostic-settings.md) pour créer et gérer les paramètres de diagnostic via le portail Azure. Consultez [Créer un paramètre de diagnostic dans Azure à l’aide d’un modèle Resource Manager](../samples/resource-manager-diagnostic-settings.md) pour les définir dans un modèle et activer la supervision complète d’une ressource lors de sa création.


## <a name="monitoring-in-the-azure-portal"></a>Surveillance dans le portail Azure
 Vous pouvez accéder aux données de supervision de la plupart des ressources Azure à partir du menu de la ressource dans le Portail Azure. Cela vous donne accès aux données d’une ressource unique à l’aide des outils Azure Monitor standard. Certains services Azure fournissent des options différentes et vous devez vous référer à la documentation de ces services pour obtenir des informations supplémentaires. Utilisez le menu d’**Azure Monitor** pour analyser les données provenant de toutes les ressources supervisées. 

### <a name="overview"></a>Vue d’ensemble
De nombreux services incluent des données de supervision dans leur page **Vue d’ensemble**, offrant un aperçu rapide de leur fonctionnement. En général, cela se base sur un sous-ensemble de métriques de plateforme stockées dans les métriques Azure Monitor. D’autres options de supervision sont généralement disponibles dans une section **Supervision** du menu du service.

![Page Vue d’ensemble](media/monitor-azure-resource/overview-page.png)


### <a name="insights-and-solutions"></a>Insights et solutions 
Certains services fournissent des outils dépassant les fonctionnalités standard d’Azure Monitor. Des [insights](../monitor-reference.md) fournissent une expérience de supervision personnalisée basée sur les fonctionnalités standard et la plateforme de données d’Azure Monitor. Des [solutions](./solutions.md) fournissent une logique de supervision prédéfinie basée sur les journaux d’activité Azure Monitor. 

Si un service dispose d’un insight Azure Monitor, vous pouvez y accéder à partir de **Supervision** dans le menu de chaque ressource. Accédez à tous les insights et à toutes les solutions à partir du menu **Azure Monitor**.

![Insights dans le portail Azure](media/monitor-azure-resource/insights.png)

### <a name="metrics"></a>Mesures
Analysez les métriques dans le Portail Azure à l’aide de **Metrics Explorer**, disponible à partir de l’élément de menu [Métriques](../platform/metrics-getting-started.md) pour la plupart des services. Cet outil vous permet d’utiliser des métriques individuelles ou d’en combiner plusieurs pour identifier les corrélations et les tendances. 

- Consultez [Bien démarrer avec Azure Metrics Explorer](../platform/metrics-getting-started.md) pour découvrir les principes de base de l’utilisation de Metrics Explorer.
- Consultez [Fonctionnalités avancées d’Azure Metrics Explorer](../platform/metrics-charts.md) pour découvrir les fonctionnalités avancées de Metrics Explorer, telles que l’utilisation de plusieurs métriques et l’application de filtres et d’un fractionnement.

![Explorateur de métriques dans le portail Azure](media/monitor-azure-resource/metrics.png)


### <a name="activity-log"></a>Journal d’activité 
Affichez les entrées du journal d’activité dans le Portail Azure avec le filtre initial défini sur la ressource actuelle. Copiez le journal d’activité dans un espace de travail Log Analytics pour y accéder et l’utiliser dans des classeurs et des requêtes de journal. 

- Consultez [Afficher et récupérer les événements du journal d’activité Azure](../platform/activity-log.md#view-the-activity-log) pour plus d’informations sur l’affichage du journal d’activité et la récupération des entrées à l’aide de différentes méthodes.
- Consultez la documentation de votre service Azure pour connaître les événements spécifiques qui sont consignés.

![Journal d’activité](media/monitor-azure-resource/activity-log.png)

### <a name="azure-monitor-logs"></a>Journaux Azure Monitor
Les journaux Azure Monitor centralisent les journaux et les métriques à partir de plusieurs services et d’autres sources de données pour qu’ils soient analysés à l’aide d’un puissant outil de requête. Comme cela est décrit ci-dessus, créez un paramètre de diagnostic pour collecter les métriques de plateforme, le journal d’activité et les journaux de ressources dans un espace de travail Log Analytics dans Azure Monitor.

[Log Analytics](../log-query/log-analytics-tutorial.md) vous permet de travailler avec des [requêtes de journal](../log-query/log-query-overview.md), qui sont une fonctionnalité puissante d’Azure Monitor qui vous permet d’effectuer une analyse avancée des données de journal à l’aide d’un langage de requête complet. Ouvrez Log Analytics à partir de **Journaux** dans le menu **Supervision** pour qu’une ressource Azure utilise les requêtes de journal avec la ressource en tant qu’[étendue de requête](../log-query/scope.md#query-scope). Cela vous permet d’analyser les données de plusieurs tables uniquement pour cette ressource. Utilisez **Journaux** dans le menu d’Azure Monitor pour accéder aux journaux de toutes les ressources. 

- Consultez [Bien démarrer avec les requêtes de journal dans Azure Monitor](../log-query/get-started-queries.md) pour bénéficier d’un tutoriel sur le langage de requêtes utilisé pour écrire des requêtes de journal.
- Consultez [Collecter les journaux d’activité de ressources Azure dans l’espace de travail Log Analytics dans Azure Monitor](../platform/resource-logs.md#send-to-log-analytics-workspace) pour plus d’informations sur la façon dont les journaux de ressources sont collectés dans les journaux Azure Monitor et la façon d’accéder à ces derniers dans une requête.
- Consultez [Mode de collecte](../platform/resource-logs.md#send-to-log-analytics-workspace) pour obtenir une explication sur la façon dont les données des journaux de ressources sont structurées dans les journaux Azure Monitor.
- Consultez la documentation de chaque service Azure pour plus d’informations sur sa table dans les journaux Azure Monitor.

![Log Analytics dans le portail Azure](media/monitor-azure-resource/logs.png)

## <a name="monitoring-from-command-line"></a>Supervision à partir de la ligne de commande
Vous pouvez accéder aux données de supervision collectées depuis votre ressource à partir d’une ligne de commande ou les inclure dans un script à l’aide d’[Azure PowerShell](/powershell/azure/) ou de l’[interface de ligne de commande Azure](/cli/azure/). 

- Pour accéder aux données de métriques à partir de l’interface CLI, consultez [Référence sur les métriques CLI](/cli/azure/monitor/metrics).
- Pour accéder aux données des journaux d’activité Azure Monitor à l’aide d’une requête de journal à partir de l’interface CLI, consultez [Référence sur Log Analytics CLI](/cli/azure/ext/log-analytics/monitor/log-analytics).
- Pour accéder aux données de métriques à partir d’Azure PowerShell, consultez [Référence sur les métriques Azure PowerShell](/powershell/module/azurerm.insights/get-azurermmetric).
- Pour accéder aux données des journaux d’activité Azure Monitor à l’aide d’une requête de journal à partir d’Azure PowerShell, consultez [Référence sur les requêtes de journal Azure PowerShell](/powershell/module/az.operationalinsights/Invoke-AzOperationalInsightsQuery).

## <a name="monitoring-from-rest-api"></a>Supervision à partir de l’API REST
Incluez les données de supervision collectées à partir de votre ressource dans une application personnalisée à l’aide d’une API REST.

- Pour plus d’informations sur l’accès aux métriques à partir de l’API REST d’Azure Monitor, consultez [Procédure pas à pas d’utilisation de l’API REST d’Azure Monitor](../platform/rest-api-walkthrough.md).
- Pour plus d’informations sur l’accès aux données des journaux d’activité Azure Monitor à l’aide d’une requête de journal à partir d’Azure PowerShell, consultez [API REST d’Azure Log Analytics](https://dev.loganalytics.io/).

## <a name="alerts"></a>Alertes
Les [alertes](../platform/alerts-overview.md) vous avertissent de façon proactive et peuvent entreprendre une action lorsque des conditions significatives sont détectées dans vos données de supervision. Vous créez une règle d’alerte qui définit une cible pour l’alerte, les conditions indiquant si une alerte doit être créée, et les actions éventuelles à entreprendre en réponse.

Différents types de données de supervision sont utilisés pour différents types de règles d’alerte.

- [Alerte de journal d’activité](../platform/alerts-activity-log.md) – Créez une alerte quand une entrée est créée dans le journal d’activité, qui correspond à des critères spécifiques. Cela vous permet d’être averti par exemple lorsqu’un type particulier de ressource est créé ou en cas d’échec d’une modification de configuration.
- [Alerte de métrique](../platform/alerts-metric.md)– Créez une alerte quand une valeur de métrique dépasse un seuil particulier. Les alertes de métriques sont plus réactives que les autres alertes et peuvent être résolues automatiquement lorsque le problème est résolu.
- [Alerte de requête de journal](../platform/alerts-log.md) – Exécutez une requête de journal à intervalles réguliers et créez une alerte si une condition particulière est détectée. Cela vous permet d’effectuer une analyse complexe sur plusieurs jeux de données.

Utilisez **Alertes** dans le menu d’une ressource pour afficher les alertes et gérer les règles d’alerte pour cette ressource. Seules les alertes de journal d’activité et les alertes de métriques utilisent des ressources Azure individuelles en tant que cibles. Les alertes de requête de journal utilisent un espace de travail Log Analytics en tant que cible et sont basées sur une requête qui peut accéder à tous les journaux stockés dans cet espace de travail. Utilisez le menu d’Azure Monitor pour afficher et gérer les alertes pour toutes les ressources, et pour gérer les règles d’alerte de requête de journal.

- Consultez les articles relatifs aux différents types d’alertes ci-dessus pour plus d’informations sur la création de règles d’alerte.
- Consultez [Créer et gérer des groupes d’actions sur le Portail Azure](../platform/action-groups.md) pour plus d’informations sur la création d’un groupe d’actions vous permettant de gérer les réponses aux alertes.



## <a name="next-steps"></a>Étapes suivantes

* Consultez [Services, schémas et catégories pris en charge pour les journaux de ressources Azure](../platform/resource-logs-schema.md) pour plus de détails sur les journaux de ressources des différents services Azure.