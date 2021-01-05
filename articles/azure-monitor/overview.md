---
title: Vue d’ensemble d’Azure Monitor | Microsoft Docs
description: Vue d’ensemble des services et fonctionnalités Microsoft qui contribuent à une stratégie de surveillance complète de vos services et applications Azure.
ms.subservice: ''
ms.topic: overview
author: bwren
ms.author: bwren
ms.date: 11/17/2019
ms.openlocfilehash: 7c48311612d48ef616e5b4c0eefaaa0ae7bb2e84
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451266"
---
# <a name="azure-monitor-overview"></a>Vue d’ensemble d’Azure Monitor

Azure Monitor vous aide à optimiser la disponibilité et les performances de vos applications et services. Il offre une solution complète pour collecter, analyser et exploiter des données de télémétrie de vos environnements cloud et locaux. Ces informations vous aident à comprendre le fonctionnement de vos applications et à identifier de façon proactive les problèmes qui les affectent et les ressources dont elles dépendent.

Voici quelques exemples de ce que vous pouvez faire avec Azure Monitor :

- Détecter et diagnostiquer les problèmes affectant les applications et les dépendances avec [Application Insights](app/app-insights-overview.md)
- Mettre en corrélation les problèmes d’infrastructure avec [Azure Monitor pour machines virtuelles](insights/vminsights-overview.md) et [Azure Monitor pour conteneurs](insights/container-insights-overview.md)
- Explorer vos données de supervision avec [Log Analytics](log-query/log-query-overview.md) pour résoudre les problèmes et établir des diagnostics approfondis
- Prendre en charge des opérations à grande échelle avec des [alertes intelligentes](platform/alerts-smartgroups-overview.md) et des [actions automatisées](platform/alerts-action-rules.md)
- Créer des visualisations avec les [tableaux de bord](learn/tutorial-logs-dashboards.md) et [classeurs](platform/workbooks-overview.md) Azure
- Collecter des données à partir de [ressources supervisées](./monitor-reference.md) à l’aide de [métriques Azure Monitor](./platform/data-platform-metrics.md)

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4qXeL]


[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="overview"></a>Vue d’ensemble

Le diagramme suivant donne une vue d’ensemble d’Azure Monitor. Au centre du diagramme se trouvent les magasins de données pour les métriques et les journaux d’activité, qui sont les deux types fondamentaux d’utilisation des données par Azure Monitor. Sur la gauche se trouvent les [sources de données de supervision](platform/data-sources.md) qui remplissent ces [magasins de données](platform/data-platform.md). Sur la droite figurent les différentes actions qu’Azure Monitor effectue avec ces données collectées. Il s’agit d’actions telles que l’analyse, la création d’alertes et le streaming vers des systèmes externes.

![Vue d’ensemble d’Azure Monitor](media/overview/overview.png)

## <a name="monitoring-data-platform"></a>Plateforme de données de surveillance

Toutes les données collectées par Azure Monitor font partie d’un des deux types fondamentaux, [les métriques et les journaux d’activité](platform/data-platform.md). Les [métriques](platform/data-platform-metrics.md) sont des valeurs numériques décrivant un aspect d’un système à un moment précis dans le temps. Elles sont légères et capables de prendre en charge des scénarios en quasi-temps réel. Les [journaux d’activité](platform/data-platform-logs.md) contiennent différents types de données organisées en enregistrements, avec différents jeux de propriétés pour chaque type. Les données de télémétrie, comme les événements et les traces, sont stockées sous forme de journaux d’activité en plus des données de performances, afin qu’elles puissent être combinées pour analyse.

Pour de nombreuses ressources Azure, vous verrez les données collectées par Azure Monitor directement sur la page Vue d’ensemble correspondante sur le portail Azure. Par exemple, jetez un œil à toutes les machines virtuelles pour voir plusieurs graphiques affichant les mesures de performances. Cliquez sur l’un des graphiques pour ouvrir les données dans [Metrics Explorer](platform/metrics-charts.md) sur le portail Azure, ce qui vous permet de représenter les valeurs de plusieurs métriques dans un graphique au fil du temps.  Vous pouvez afficher les graphiques de manière interactive ou les épingler au tableau de bord pour les voir avec d’autres visualisations.

![Diagramme montrant le flux des données de métriques dans Metrics Explorer à utiliser dans les visualisations](media/overview/metrics.png)

Les données de journal collectées par Azure Monitor peuvent être analysées à l’aide de [requêtes](log-query/log-query-overview.md) qui permettent de récupérer, consolider et analyser rapidement les données collectées.  Vous pouvez créer et tester des requêtes à l’aide de [Log Analytics](./log-query/log-query-overview.md) dans le portail Azure. Vous pouvez ensuite analyser directement les données avec différents outils ou enregistrer les requêtes pour les utiliser avec des [visualisations](visualizations.md) ou des [règles d’alerte](platform/alerts-overview.md).

Azure Monitor utilise une version du [langage de requête Kusto](/azure/kusto/query/) qui est adapté aux requêtes simples dans les journaux, mais comprend également des fonctionnalités avancées telles que les agrégations, les jointures et l’analytique intelligente. Il existe [plusieurs leçons](log-query/get-started-queries.md) pour vous aider à apprendre le langage de requête.  Des conseils particuliers sont fournis aux utilisateurs qui connaissent déjà [SQL](log-query/sql-cheatsheet.md) et [Splunk](log-query/splunk-cheatsheet.md).

![Diagramme montrant le flux des données des journaux dans Log Analytics à des fins d’analyse](media/overview/logs.png)

## <a name="what-data-does-azure-monitor-collect"></a>Quelles sont les données collectées par Azure Monitor ?

Azure Monitor peut recueillir des données de [diverses sources](monitor-reference.md), telles que votre application, tous les systèmes d’exploitation et services sur lesquels elle s’appuie, et jusqu’à la plateforme elle-même. Azure Monitor collecte des données à partir de chacun des niveaux suivants :

- **Données de surveillance de l’application** : données concernant les performances et la fonctionnalité du code que vous avez écrit, quelle que soit la plateforme.
- **Données de surveillance du système d’exploitation invité** : données concernant le système d’exploitation sur lequel votre application est exécutée. Il peut s’agir d’Azure, d’un autre cloud ou d’un système local. 
- **Données de surveillance des ressources Azure** : données concernant le fonctionnement d’une ressource Azure.
- **Données de surveillance d’abonnement Azure** : données concernant le fonctionnement et la gestion d’un abonnement Azure, mais aussi données concernant l’intégrité et le fonctionnement d’Azure. 
- **Données de surveillance de locataire Azure** : données concernant le fonctionnement des services Azure au niveau du locataire, tels qu’Azure Active Directory.

Dès que vous créez un abonnement Azure et commencez à ajouter des ressources telles que des machines virtuelles et des applications web, Azure Monitor commence la collecte de données.  Les [journaux d’activité](platform/platform-logs-overview.md) enregistrent la création et la modification des ressources. Les [métriques](platform/data-platform.md) vous indiquent les performances de la ressource et les ressources qu’elle consomme. 

[Activez les diagnostics](platform/platform-logs-overview.md) pour étendre les données que vous collectez au fonctionnement interne des ressources.  [Ajoutez un agent](platform/agents-overview.md) aux ressources de calcul pour collecter les données de télémétrie à partir de leurs systèmes d’exploitation invités. 

Activez la supervision de votre application avec [Application Insights](app/app-insights-overview.md) afin de collecter des informations détaillées sur les vues de pages, les requêtes d’application et les exceptions. Vérifiez la disponibilité de votre application en configurant un [test de disponibilité](app/monitor-web-app-availability.md) pour simuler le trafic utilisateur.

### <a name="custom-sources"></a>Sources personnalisées

Azure Monitor peut collecter des données de journal à partir de n’importe quel client REST à l’aide de [l’API de collecteur de données](platform/data-collector-api.md). Cela vous permet de créer des scénarios de supervision personnalisés et d’étendre la supervision à des ressources qui n’exposent pas de données de télémétrie via d’autres sources.

## <a name="insights"></a>Insights
La supervision des données est utile uniquement si elle permet d’augmenter votre visibilité sur le fonctionnement de votre environnement informatique. Les [insights](monitor-reference.md#insights-and-core-solutions) offrent une expérience de supervision personnalisée pour des services Azure spécifiques. Ils nécessitent une configuration minimale et augmentent votre visibilité sur le fonctionnement des ressources critiques.

### <a name="application-insights"></a>Application Insights
[Application Insights](app/app-insights-overview.md) surveille la disponibilité, les performances et l’utilisation de votre application web, qu’elle soit hébergée dans le cloud ou localement. Cette solution s’appuie sur la puissante plateforme d’analyse de données dans Azure Monitor pour vous fournir des insights détaillés sur le fonctionnement de votre application. Elle vous permet de diagnostiquer les erreurs sans attendre qu’un utilisateur les signale. Application Insights inclut des points de connexion à divers outils de développement et s’intègre à Visual Studio pour prendre en charge vos processus DevOps.

![App Insights](media/overview/app-insights.png)

### <a name="azure-monitor-for-containers"></a>Azure Monitor pour des conteneurs
[Azure Monitor pour conteneurs](insights/container-insights-overview.md) supervise les performances des charges de travail de conteneur déployées sur des clusters Kubernetes managés hébergés sur AKS (Azure Kubernetes Service). Cela vous permet de visualiser les performances en collectant des métriques à partir des contrôleurs, des nœuds et des conteneurs qui sont disponibles dans Kubernetes par le biais de l’API Metrics. Les journaux d’activité de conteneur sont aussi collectés.  Une fois que vous avez activé la supervision des clusters Kubernetes, ces métriques et ces journaux d’activité sont automatiquement collectés à l’aide d’une version conteneurisée de l’agent Log Analytics pour Linux.

![Intégrité du conteneur](media/overview/container-insights.png)

### <a name="azure-monitor-for-vms"></a>Azure Monitor pour machines virtuelles
[Azure Monitor pour machines virtuelles](insights/vminsights-overview.md) supervise vos machines virtuelles Azure à grande échelle. Il analyse les performances et l’intégrité de vos machines virtuelles Windows et Linux, et identifie leurs différents processus et les dépendances interconnectées envers des processus externes. Cette solution inclut la prise en charge de la supervision des performances et des dépendances des applications pour les machines virtuelles hébergées sur site ou par un autre fournisseur cloud.  


![Insights de machine virtuelle](media/overview/vm-insights.png)


## <a name="responding-to-critical-situations"></a>Réagir à des situations critiques
En plus de vous permettre d’analyser les données de supervision de manière interactive, une solution de supervision efficace doit pouvoir répondre de manière proactive à des conditions critiques identifiées dans les données qu’elle collecte. Il peut par exemple s’agir d’envoyer un SMS ou un courrier à un administrateur en charge de l’examen d’un problème. Ou de lancer un processus automatisé qui essaie de corriger une condition d’erreur.


### <a name="alerts"></a>Alertes
Les [alertes dans Azure Monitor](platform/alerts-overview.md) vous avertissent de manière proactive en cas de condition critique, et sont susceptibles d’essayer de prendre des mesures correctives. Les règles d’alerte basées sur les métriques fournissent des alertes en quasi-temps réel basées sur des valeurs numériques. Les règles basées sur les journaux autorisent une logique complexe entre les données de plusieurs sources.

Les règles d’alerte dans Azure Monitor utilisent des [groupes d’actions](platform/action-groups.md), qui contiennent des ensembles uniques de destinataires et d’actions qui peuvent être partagés entre plusieurs règles. Selon vos besoins, les groupes d’actions peuvent effectuer des actions telles que l’utilisation de webhooks pour que les alertes démarrent des actions externes ou pour l’intégration à vos outils ITSM.

![Capture d’écran montrant des alertes dans Azure Monitor, avec le niveau de gravité, le nombre total d’alertes et d’autres informations](media/overview/alerts.png)

### <a name="autoscale"></a>Mise à l’échelle automatique
La mise à l’échelle automatique vous permet de disposer de la bonne quantité de ressources en cours d’exécution pour gérer la charge sur votre application. Créez des règles qui utilisent des métriques collectées par Azure Monitor afin de déterminer quand ajouter automatiquement des ressources lorsque la charge augmente. Dépensez moins en supprimant les ressources inactives. Vous spécifiez un nombre minimal et maximal d’instances et la logique permettant de définir quand augmenter ou diminuer les ressources.

![Diagramme illustrant la mise à l’échelle automatique, avec plusieurs serveurs sur une ligne intitulée Temps processeur > 80 %, dont deux serveurs marqués comme Minimum, trois comme Capacité actuelle et cinq comme Maximum](media/overview/autoscale.png)

## <a name="visualizing-monitoring-data"></a>Affichage des données de supervision
Les [visualisations](visualizations.md), telles que les tables et les graphiques, sont des outils efficaces pour résumer les données de supervision et les proposer à différents publics. Azure Monitor dispose de ses propres fonctionnalités de visualisation des données de supervision et s’appuie sur d’autres services Azure pour leur publication auprès de différents publics.

### <a name="dashboards"></a>Tableaux de bord
Les [tableaux de bord Azure](../azure-portal/azure-portal-dashboards.md) vous permettent de combiner différentes sortes de données dans un même volet du [portail Azure](https://portal.azure.com). Si vous le souhaitez, vous pouvez partager le tableau de bord avec d’autres utilisateurs d’Azure. Ajoutez la sortie de n’importe quelle requête de journal ou graphique de métriques à un tableau de bord Azure. Par exemple, vous pouvez créer un tableau de bord qui combine des vignettes affichant un graphique de métriques, un tableau de journaux d’activité, un graphique d’utilisation provenant d’Application Insights et la sortie d’une requête de journal d’activité.

![Capture d’écran présentant un tableau de bord Azure, qui comprend les vignettes Application et Sécurité ainsi que d’autres informations personnalisables](media/overview/dashboard.png)

### <a name="workbooks"></a>Workbooks
Les [classeurs](platform/workbooks-overview.md) fournissent un canevas flexible pour l’analyse des données et la création de rapports visuels enrichis dans le portail Azure. Ils vous permettent d’exploiter plusieurs sources de données à travers l’écosystème Azure et de les combiner dans des expériences interactives unifiées. Utilisez les classeurs fournis avec Insights ou créez vos propres classeurs à partir de modèles prédéfinis.


![Exemple de classeurs](media/overview/workbooks.png)

### <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com) est un service d’analytique métier qui fournit des visualisations interactives pour diverses sources de données. Il est idéal pour mettre des données à la disposition d’autres personnes internes ou externes à votre organisation. Vous pouvez configurer Power BI pour [importer automatiquement les données de journal à partir d’Azure Monitor](./platform/powerbi.md) afin de tirer parti de ces visualisations supplémentaires.


![Power BI](media/overview/power-bi.png)


## <a name="integrate-and-export-data"></a>Intégrer et exporter des données
Vous devrez souvent intégrer Azure Monitor à d’autres systèmes et créer des solutions personnalisées qui utilisent vos données de supervision. D’autres services Azure fonctionnent avec Azure Monitor en vue de fournir cette intégration.

### <a name="event-hub"></a>Event Hub
[Azure Event Hubs](../event-hubs/index.yml) est une plateforme de streaming et un service d’ingestion d’événements. Il peut transformer et stocker des données à l’aide de n’importe quel fournisseur d’analytique en temps réel ou adaptateur de stockage/traitement par lot. Utilisez Event Hubs pour [diffuser en continu les données Azure Monitor](platform/stream-monitoring-data-event-hubs.md) auprès des outils SIEM (Security Information and Event Management) et de supervision.


### <a name="logic-apps"></a>Logic Apps
[Logic Apps](https://azure.microsoft.com/services/logic-apps) est un service qui vous permet d’automatiser les tâches et processus métier à l’aide de flux de travail qui s’intègrent à différents systèmes et services. Parmi les activités disponibles, certaines lisent et écrivent des métriques et des journaux dans Azure Monitor. Cela vous permet de créer des workflows qui s’intègrent à un large éventail d’autres systèmes.


### <a name="api"></a>API
Plusieurs API sont disponibles pour lire et écrire les métriques et les journaux d’activité vers et depuis Azure Monitor en plus d’accéder aux alertes générées. Vous pouvez également configurer et récupérer les alertes. Cela vous offre essentiellement des possibilités illimitées pour créer des solutions personnalisées qui s’intègrent à Azure Monitor.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus :

* [Métriques et journaux d’activité](platform/data-platform.md) pour les données collectées par Azure Monitor.
* [Sources de données](platform/data-sources.md) pour découvrir comment les différents composants de votre application envoient des données de télémétrie.
* [Requêtes de journal](log-query/log-query-overview.md) pour l’analyse des données collectées.
* [Bonnes pratiques](/azure/architecture/best-practices/monitoring) pour la supervision des applications et services cloud.
