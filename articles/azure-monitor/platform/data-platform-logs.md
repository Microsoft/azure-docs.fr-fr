---
title: Journaux Azure Monitor
description: Décrit les journaux Azure Monitor qui sont utilisés pour une analyse avancée des données de supervision.
documentationcenter: ''
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 10/22/2020
ms.author: bwren
ms.openlocfilehash: 69fda41249ced8518e430af8305c0cb10822b214
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96559063"
---
# <a name="azure-monitor-logs-overview"></a>Vue d’ensemble de Journaux d’activité Azure Monitor
Les journaux Azure Monitor sont une fonctionnalité Azure Monitor qui collecte et organise les données de performances et de journal issues de [ressources supervisées](../monitor-reference.md). Les données provenant de différentes sources, telles que les [journaux de plateforme](platform-logs-overview.md) issus des services Azure, les données de journal et de performances issues des [agents de machines virtuelles](agents-overview.md) et les données d’utilisation et de performances issues des [applications](../app/app-insights-overview.md) peuvent être regroupées dans un espace de travail unique pour pouvoir être analysées ensemble à l’aide d’un langage de requête sophistiqué à même d’analyser rapidement des millions d’enregistrements. Vous pouvez effectuer une requête simple qui récupère simplement un ensemble spécifique d’enregistrements ou effectuer une analyse de données sophistiquée pour identifier les modèles critiques dans vos données de supervision. Utilisez les requêtes de journal et leurs résultats de manière interactive avec Log Analytics, utilisez-les dans des règles d’alerte pour être notifié des problèmes de manière proactive ou visualisez leurs résultats dans un classeur ou un tableau de bord.

> [!NOTE]
> Les journaux Azure Monitor représentent la moitié de la plateforme de données qui prend en charge Azure Monitor. L’autre moitié, constituée des [métriques Azure Monitor](data-platform-metrics.md), stocke les données numériques dans une base de données de série chronologique. Ainsi, ces données sont plus légères que les données des journaux Azure Monitor et sont capables de prendre en charge des scénarios en quasi-temps réel. Elles sont donc particulièrement utiles pour la création d’alertes et la détection rapide des problèmes. Toutefois, la fonctionnalité Métriques peut uniquement stocker des données numériques dans une structure particulière, tandis que Journaux d’activité peut stocker différents types de données, chacun ayant sa propre structure. Vous pouvez également effectuer des analyses complexes sur les données de journaux à l’aide de requêtes de journal qui ne peuvent pas être utilisées pour l’analyse des données de métriques.


## <a name="what-can-you-do-with-azure-monitor-logs"></a>Que pouvez-vous faire avec les journaux Azure Monitor ?
Le tableau suivant décrit quelques-unes des façons d’utiliser des journaux dans Azure Monitor :

|  |  |
|:---|:---|
| **Analyser** | Utiliser [Log Analytics](../log-query/log-analytics-tutorial.md) sur le portail Azure pour écrire des [requêtes de journal](../log-query/log-query-overview.md) et analyser de manière interactive des données de journal à l’aide d’un puissant moteur d’analyse |
| **Alert** | Configurer une [règle d’alerte de journal](alerts-log.md) qui envoie une notification ou prend une [action de façon automatique](action-groups.md) lorsque les résultats de la requête correspondent à un résultat spécifique. |
| **Visualiser** | Épingler les résultats de requête affichés sous forme de tableaux ou de graphiques dans un [tableau de bord Azure](../../azure-portal/azure-portal-dashboards.md).<br>Créer un [classeur](./workbooks-overview.md) à combiner avec plusieurs jeux de données dans un rapport interactif. <br>Exporter les résultats d’une requête vers [Power BI](powerbi.md) pour utiliser différentes visualisations et les partager avec les utilisateurs extérieurs à Azure.<br>Exporter les résultats d’une requête vers [Grafana](grafana-plugin.md) afin de profiter de ses tableaux de bord et de les combiner avec d’autres sources de données.|
| **Insights** | Prendre en charge des [insights](../monitor-reference.md#insights-and-core-solutions) qui offrent une expérience de supervision personnalisée pour des applications et services spécifiques.  |
| **Récupérer** | Accéder aux résultats de requête de journal à partir d’une ligne de commande à l’aide d’[Azure CLI](/cli/azure/ext/log-analytics/monitor/log-analytics).<br>Accéder aux résultats de requête de journal à partir d’une ligne de commande à l’aide de [cmdlets PowerShell](/powershell/module/az.operationalinsights).<br>Accéder aux résultats de requête de journal à partir d’une application personnalisée à l’aide d’une [API REST](https://dev.loganalytics.io/). |
| **Export** | Configurer l’[exportation automatisée des données de journal](logs-data-export.md) vers le compte de stockage Azure ou Azure Event Hubs.<br>Générer un flux de travail pour récupérer des données de journal et les copier vers un emplacement externe à l’aide de [Logic Apps](logicapp-flow-connector.md). |

![Vue d’ensemble des journaux d’activité](media/data-platform-logs/logs-overview.png)


## <a name="data-collection"></a>Collecte de données
Une fois que vous avez créé un espace de travail Log Analytics, vous devez configurer différentes sources pour envoyer leurs données. Aucune donnée n’est collectée automatiquement. Cette configuration diffère d’une source de données à l’autre. Par exemple, [créez des paramètres de diagnostic](diagnostic-settings.md) pour envoyer des journaux de ressources à partir de ressources Azure vers l’espace de travail. [Activez Azure Monitor pour machines virtuelles](../insights/vminsights-enable-overview.md) afin de collecter des données à partir de machines virtuelles. Configurez des [sources de données sur l’espace de travail](data-sources.md) pour collecter des événements et des données de performances supplémentaires.

- Consultez [Quels sont les éléments supervisés par Azure Monitor ?](../monitor-reference.md) afin d’obtenir la liste complète des sources de données que vous pouvez configurer pour envoyer des données aux journaux Azure Monitor.


## <a name="log-analytics-workspaces"></a>Espaces de travail Log Analytics
Les données collectées par les journaux Azure Monitor sont stockées dans un ou plusieurs [espaces de travail Log Analytics](./design-logs-deployment.md). L’espace de travail définit l’emplacement géographique des données, les droits d’accès définissant quels utilisateurs peuvent accéder aux données et les paramètres de configuration tels que le niveau tarifaire et la conservation des données.  

Vous devez créer au moins un espace de travail pour utiliser les journaux Azure Monitor. Un espace de travail unique peut suffire pour toutes vos données de supervision ou vous pouvez choisir de créer plusieurs espaces de travail selon vos besoins. Par exemple, vous pouvez avoir un espace de travail pour vos données de production et un autre pour les tests. 

- Consultez [Créer un espace de travail Log Analytics dans le portail Azure](../learn/quick-create-workspace.md) pour créer un espace de travail.
- Pour plus d’informations sur la création de plusieurs espaces de travail, consultez [Conception de votre déploiement de journaux Azure Monitor](design-logs-deployment.md).

## <a name="data-structure"></a>Structure de données
Les requêtes de journal récupèrent leurs données d’un espace de travail Log Analytics. Chaque espace de travail contient plusieurs tables qui sont organisées en colonnes distinctes avec plusieurs lignes de données. Chaque table est définie par un ensemble unique de colonnes partagées par les lignes de données fournies par la source de données. 

[![Azure Monitor Logs structure](media/data-platform-logs/logs-structure.png)](media/data-platform-logs/logs-structure.png#lightbox)


Les données de journal d’Application Insights sont également stockées dans Journaux d’activité Azure Monitor, mais elles sont stockées différemment selon la configuration de votre application. Pour une application basée sur un espace de travail, les données sont stockées dans un espace de travail Log Analytics dans un ensemble standard de tables pour contenir des données telles que les requêtes d’application, les exceptions et les affichages de page. Plusieurs applications peuvent utiliser le même espace de travail. Pour une application classique, les données ne sont pas stockées dans un espace de travail Log Analytics. Elles utilisent le même langage de requête, et vous créez et exécutez des requêtes à l’aide du même outil Log Analytics dans le portail Azure. Toutefois, les données des applications classiques sont stockées séparément les unes des autres. Leur structure générale est la même que celle des applications basées sur un espace de travail, bien que les noms de table et de colonne soient différents. Consultez [Modifications des ressources basées sur un espace de travail](../app/apm-tables.md) pour une comparaison détaillée du schéma des applications classiques et des applications basées sur l’espace de travail.


> [!NOTE]
> Nous proposons toujours une compatibilité descendante complète pour vos classeurs, alertes basées sur les journaux et requêtes de ressources Application Insights classiques au sein de l’expérience Application Insights. Pour interroger/afficher [le nouveau schéma ou la nouvelle structure de table basé sur un espace de travail](../app/apm-tables.md), vous devez d’abord accéder à votre espace de travail Log Analytics. Pendant la préversion, la sélection de **Journaux** dans les volets d’Application Insights vous donne accès à l’expérience de requête Application Insights classique. Consultez [Étendue de requête](../log-query/scope.md) pour plus d’informations.


[![Azure Monitor Logs structure for Application Insights](media/data-platform-logs/logs-structure-ai.png)](media/data-platform-logs/logs-structure-ai.png#lightbox)


## <a name="log-queries"></a>Requêtes dans les journaux
Les données sont extraites d’un espace de travail Log Analytics à l’aide d’une requête de journal, qui est une requête en lecture seule visant à traiter les données et à retourner des résultats. Les requêtes de journal sont écrites en [KQL (Langage de requête Kusto)](/azure/data-explorer/kusto/query/), langage de requête également utilisé par Azure Data Explorer. Vous pouvez écrire des requêtes de journal dans Log Analytics pour analyser de manière interactive leurs résultats, les utiliser dans des règles d’alerte pour être informé de manière proactive des problèmes ou inclure leurs résultats dans des classeurs ou des tableaux de bord. Les insights incluent des requêtes prédéfinies pour la prise en charge de leurs vues et classeurs.

- Consultez [Requêtes de journal dans Azure Monitor](log-query/../../log-query/log-query-overview.md) pour savoir exactement où sont utilisées les requêtes de journal et obtenir des références de tutoriels et autres documents pour vous aider à démarrer.

![Log Analytics](media/data-platform-logs/log-analytics.png)

## <a name="log-analytics"></a>Log Analytics
Utilisez Log Analytics, qui est un outil du portail Azure, pour modifier et exécuter des requêtes de journal et analyser leurs résultats de manière interactive. Vous pouvez ensuite utiliser les requêtes que vous créez pour prendre en charge d’autres fonctionnalités dans Azure Monitor telles que les alertes et les classeurs de requêtes de journal. Accédez à Log Analytics à partir de l’option **Journaux** dans le menu Azure Monitor ou à partir de la plupart des autres services du portail Azure.

- Pour obtenir une description de Log Analytics, consultez [Présentation de Log Analytics dans Azure Monitor](../log-query/log-analytics-overview.md). 
- Consultez [Tutoriel Log Analytics](../log-query/log-analytics-tutorial.md) pour parcourir les fonctionnalités de Log Analytics afin de créer une requête de journal simple et d’analyser ses résultats.



## <a name="relationship-to-azure-data-explorer"></a>Relation avec Azure Data Explorer
La fonctionnalité Journaux d’activité Azure Monitor repose sur Azure Data Explorer. Un espace de travail Log Analytics est à peu près l’équivalent d’une base de données dans Azure Data Explorer, les tables sont structurées de la même façon, et les deux utilisent le même langage de requête Kusto (KQL). L’utilisation de Log Analytics pour travailler avec des requêtes Azure Monitor dans le portail Azure est similaire à l’utilisation de l’interface utilisateur web d’Azure Data Explorer. Vous pouvez même [inclure des données provenant d’un espace de travail Log Analytics dans une requête Azure Data Explorer](/azure/data-explorer/query-monitor-data). 


## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [requêtes de journal](../log-query/log-query-overview.md) pour récupérer et analyser les données d’un espace de travail Log Analytics.
- Découvrez les [métriques dans Azure Monitor](data-platform-metrics.md).
- Découvrez les [données de surveillance disponibles](data-sources.md) pour différentes ressources dans Azure.
