---
title: Requêtes de journal dans Azure Monitor
description: Informations de référence sur le langage de requête Kusto utilisé par Azure Monitor. Inclut les éléments supplémentaires spécifiques à Azure Monitor et les éléments non pris en charge dans les requêtes de journal Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/09/2020
ms.openlocfilehash: 6174bcbe5a014cff8dbd8dff242880d7f0ef7aa0
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491390"
---
# <a name="log-queries-in-azure-monitor"></a>Requêtes de journal dans Azure Monitor
Azure Monitor Logs repose sur Azure Data Explorer, et les requêtes de journal sont écrites à l’aide du même langage de requête Kusto (KQL). Il s’agit d’un langage riche conçu pour être facile à lire et écrire. Vous pourrez donc commencer écrire des requêtes avec un minimum de conseils.

Dans Azure Monitor, vous utiliserez les zones suivantes pour créer des requêtes :

- [Log Analytics](../log-query/log-analytics-overview.md). Principal outil dans le portail Azure pour la modification de requêtes de journal et l’analyse interactive de leurs résultats. Même si vous envisagez d’utiliser une requête de journal ailleurs dans Azure Monitor, vous l’écrirez et la testerez généralement dans Log Analytics avant de la copier dans son emplacement final.
- [Règles d’alerte de journal](../platform/alerts-overview.md). Identifient de façon proactive les problèmes à partir des données dans votre espace de travail.  Chaque règle d’alerte est basée sur une requête dans les journaux qui est exécutée automatiquement à intervalles réguliers.  Les résultats sont inspectés pour déterminer si une alerte doit être créée.
- [Classeurs](../platform/workbooks-overview.md). Incluez les résultats des requêtes de journal en utilisant différentes visualisations dans les rapports visuels interactifs du portail Azure.
- [Tableaux de bord Azure](../learn/tutorial-logs-dashboards.md). Épinglez les résultats de n’importe quelle requête dans un tableau de bord Azure afin de visualiser les données de journal et les métriques ensemble et de partager ces informations avec d’autres utilisateurs Azure si vous le souhaitez.
- [Logic Apps](../platform/logicapp-flow-connector.md).  Utilisez les résultats d’une requête de journal dans un workflow automatisé à l’aide de Logic Apps.
- [PowerShell](/powershell/module/az.operationalinsights/get-azoperationalinsightssearchresult). Utilisez les résultats d’une requête de journal dans un script PowerShell à partir d’une ligne de commande ou d’un runbook Azure Automation qui utilise Get-AzOperationalInsightsSearchResults.
- [API Journaux d’activité Azure Monitor](https://dev.loganalytics.io). Récupérez les données de journal de l’espace de travail à partir de n’importe quel client API REST.  La demande API comprend une requête qui est exécutée sur Azure Monitor pour déterminer les données à récupérer.

## <a name="getting-started"></a>Prise en main
La meilleure façon de commencer à apprendre à écrire des requêtes de journal à l’aide de KQL consiste à tirer parti des tutoriels et exemples disponibles.

- [Tutoriel Log Analytics](log-analytics-tutorial.md) : tutoriel sur l’utilisation des fonctionnalités Log Analytics, l’outil que vous utiliserez dans le portail Azure pour modifier et exécuter des requêtes. Il vous permet également d’écrire des requêtes simples sans utiliser directement le langage de requête. Si vous n’avez jamais utilisé Log Analytics, commencez ici pour découvrir l’outil que vous utiliserez avec les autres tutoriels et exemples.
- [Tutoriel KQL](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor) : procédure pas à pas présentant les concepts de base et les opérateurs courants KQL. C’est le meilleur endroit pour commencer à vous familiariser avec le langage lui-même et la structure des requêtes de journal. 
- [Exemples de requêtes](example-queries.md) : description des exemples de requêtes disponibles dans Log Analytics. Vous pouvez utiliser les requêtes sans modification, ou les utiliser comme exemples pour apprendre KQL.
- [Exemples de requêtes](/azure/data-explorer/kusto/query/samples?pivots=azuremonitor) : exemples de requêtes illustrant différents concepts.



## <a name="reference-documentation"></a>Documentation de référence
Une [documentation concernant KQL](/azure/data-explorer/kusto/query/), y compris la référence de l’ensemble des commandes et des opérateurs, est disponible dans la documentation de l’Explorateur de données Azure. Même si vous maîtrisez KQL, vous utiliserez régulièrement la référence pour examiner de nouvelles commandes et de nouveaux scénarios.


## <a name="language-differences"></a>Différences de langage
Même si Azure Monitor utilise le même KQL que l’Explorateur de données Azure, il existe quelques différences. La documentation KQL spécifie les opérateurs qui ne sont pas pris en charge par Azure Monitor ou proposent des fonctionnalités différentes. Les opérateurs spécifiques à Azure Monitor sont documentés dans le contenu Azure Monitor. Les sections suivantes dressent la liste des différences entre les versions du langage pour une référence rapide.

### <a name="statements-not-supported-in-azure-monitor"></a>Instructions non prises en charge dans Azure Monitor

* [Alias](/azure/kusto/query/aliasstatement)
* [Paramètres de requête](/azure/kusto/query/queryparametersstatement)

### <a name="functions-not-supported-in-azure-monitor"></a>Fonctions non prises en charge dans Azure Monitor

* [cluster()](/azure/kusto/query/clusterfunction)
* [cursor_after()](/azure/kusto/query/cursorafterfunction)
* [cursor_before_or_at()](/azure/kusto/query/cursorbeforeoratfunction)
* [cursor_current(), current_cursor()](/azure/kusto/query/cursorcurrent)
* [database()](/azure/kusto/query/databasefunction)
* [current_principal()](/azure/kusto/query/current-principalfunction)
* [extent_id()](/azure/kusto/query/extentidfunction)
* [extent_tags()](/azure/kusto/query/extenttagsfunction)

### <a name="operators-not-supported-in-azure-monitor"></a>Opérateurs non pris en charge dans Azure Monitor

* [Jointure entre plusieurs clusters](/azure/kusto/query/joincrosscluster)

### <a name="plugins-not-supported-in-azure-monitor"></a>Plug-ins non pris en charge dans Azure Monitor

* [Plug-in Python](/azure/kusto/query/pythonplugin)
* [plug-in sql_request](/azure/kusto/query/sqlrequestplugin)


### <a name="additional-operators-in-azure-monitor"></a>Opérateurs supplémentaires dans Azure Monitor
Les opérateurs suivants prennent en charge des fonctionnalités propres à Azure Monitor et ne sont pas disponibles en dehors d'Azure Monitor.

* [app()](app-expression.md)
* [resource()](resource-expression.md)
* [workspace()](workspace-expression.md)

## <a name="next-steps"></a>Étapes suivantes
- Suive un [didacticiel sur l’écriture de requêtes](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor).
- Accédez à la [documentation de référence complète du langage de requête Kusto](/azure/kusto/query/).

