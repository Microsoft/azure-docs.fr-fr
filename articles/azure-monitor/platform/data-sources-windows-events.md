---
title: Collecter les sources de données du journal des événements Windows avec l’agent Log Analytics dans Azure Monitor
description: Décrit comment configurer la collecte des journaux d’événements Windows et par Azure Monitor et les détails des enregistrements créés.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/21/2020
ms.openlocfilehash: 109e96f862ec2f3ddf879bccba114c44aecfe3c8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012593"
---
# <a name="collect-windows-event-log-data-sources-with-log-analytics-agent"></a>Collecter les sources de données du journal des événements Windows avec l’agent Log Analytics
Les journaux d’événements Windows sont les [sources de données](agent-data-sources.md) les plus courantes pour les agents Log Analytics sur Windows puisque de nombreuses applications écrivent dans le journal des événements Windows.  Vous pouvez collecter des événements à partir de journaux d’activité standard tels que Système et Application, ou spécifier des journaux d’activité personnalisés créés par les applications que vous souhaitez surveiller.

> [!IMPORTANT]
> Cet article traite de la collecte d’événements Windows avec l’[agent Log Analytics](log-analytics-agent.md), qui est un des agents utilisés par Azure Monitor. D’autres agents collectent des données différentes et sont configurés différemment. Pour obtenir la liste des agents disponibles et les données qu’ils peuvent collecter, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md).

![Événements Windows](media/data-sources-windows-events/overview.png)     

## <a name="configuring-windows-event-logs"></a>Configuration des journaux d’événements Windows
Configurez les journaux d’événements Windows à partir du [menu Données dans les paramètres avancés](agent-data-sources.md#configuring-data-sources) de l’espace de travail Log Analytics.

Azure Monitor collecte uniquement les événements des journaux d’événements Windows spécifiés dans les paramètres.  Vous pouvez ajouter un journal d’événements en tapant le nom du journal puis en cliquant sur **+** .  Pour chaque journal, seuls les événements avec les niveaux de gravité sélectionnés sont collectés.  Vérifiez les niveaux de gravité du journal que vous souhaitez collecter.  Vous ne pouvez pas fournir d’autres critères supplémentaires pour filtrer les événements.

Lorsque vous tapez le nom d’un journal d’événements, Azure Monitor suggère des noms communs de journaux d’événements. Si le journal que vous voulez ajouter n’apparaît pas dans la liste, vous pouvez l’ajouter en saisissant le nom complet du journal. Vous trouverez le nom complet du journal à l’aide de l’Observateur d’événements. Dans l’Observateur d’événements, ouvrez la page *Propriétés* du journal et copiez la chaîne du champ *Nom complet*.

![Configurer les événements Windows](media/data-sources-windows-events/configure.png)

> [!NOTE]
> Les événements critiques du journal d’événements Windows auront un niveau de gravité « Erreur » dans les journaux d’Azure Monitor.

## <a name="data-collection"></a>Collecte de données
Azure Monitor collecte chaque événement correspondant à un niveau de gravité sélectionné à partir d’un journal d’événements surveillé à mesure que l’événement est créé.  L’agent enregistre sa position dans chaque journal des événements à partir duquel il collecte.  Si l’agent est mis hors connexion pendant un moment, il collecte les événements à partir de là où il s’était arrêté, même si ces événements ont été créés lorsque l’agent était hors connexion.  Il est possible que ces événements ne soient pas collectés si des événements non collectés d’un journal sont écrasés pendant que l’agent est hors connexion.

>[!NOTE]
>Azure Monitor ne collecte pas les événements d’audit créés par SQL Server à partir de la source *MSSQLSERVER* avec l’ID d’événement 18453 qui contient les mots clés - *Classic* ou *Audit Success* et le mot clé *0xa0000000000000*.
>

## <a name="windows-event-records-properties"></a>Propriétés des enregistrements d’événements Windows
Les enregistrements d'événements Windows sont de type **Événement** et possèdent les propriétés affichées dans le tableau suivant :

| Propriété | Description |
|:--- |:--- |
| Computer |Nom de l'ordinateur à partir duquel l'événement a été collecté. |
| EventCategory |Catégorie de l’événement. |
| EventData |Toutes les données d'événement au format brut. |
| EventID |Numéro de l’événement. |
| EventLevel |Gravité de l'événement au format numérique. |
| EventLevelName |Gravité de l'événement au format texte. |
| EventLog |Nom du journal des événements à partir duquel l'événement a été collecté. |
| ParameterXml |Valeurs des paramètres d'événement au format XML. |
| ManagementGroupName |Nom du groupe d’administration pour les agents System Center Operations Manager.  Pour les autres agents, cette valeur est `AOI-<workspace ID>`. |
| RenderedDescription |Description de l'événement avec les valeurs de paramètres |
| Source |Source de l'événement. |
| SourceSystem |Type d’agent auprès duquel l’événement a été collecté. <br> Ops Manager – Agent Windows, connexion directe ou géré par Operations Manager <br> Linux – Tous les agents Linux  <br> AzureStorage – Diagnostics Azure |
| TimeGenerated |Date et heure de création de l’événement dans Windows. |
| UserName |Nom d'utilisateur du compte qui a consigné l'événement. |

## <a name="log-queries-with-windows-events"></a>Requêtes de journaux avec des événements Windows
Le tableau suivant fournit plusieurs exemples de requêtes qui extraient des enregistrements d’événements Windows.

| Requête | Description |
|:---|:---|
| Événement |Tous les événements Windows. |
| Événement &#124; où valeur EventLevelName == « erreur » |Tous les événements Windows avec la gravité de l'erreur. |
| Événement &#124; résumer count() par source |Nombre d’événements Windows par source. |
| Événement &#124; où valeur EventLevelName == « erreur » &#124; résumer count() par source |Nombre d’événements d’erreur Windows par source. |


## <a name="next-steps"></a>Étapes suivantes
* Configurez Log Analytics pour collecter d’autres [sources de données](agent-data-sources.md) à analyser.
* Découvrez les [requêtes dans les journaux](../log-query/log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions.  
* Configurez la [collecte des compteurs de performances](data-sources-performance-counters.md) à partir de vos agents Windows.
