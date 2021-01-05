---
title: Vue d’ensemble d’une requête interservice entre Azure Monitor et Azure Data Explorer (préversion)
description: Interroger des données Azure Data Explorer via des outils Azure Log Analytics, et inversement, pour joindre et analyser toutes vos données au même endroit.
author: orens
ms.author: bwren
ms.reviewer: bwren
ms.subservice: logs
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: 0af9111223bb16f1c5843223e7fe1661ce99c623
ms.sourcegitcommit: 003ac3b45abcdb05dc4406661aca067ece84389f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96749227"
---
# <a name="cross-service-query---azure-monitor-and-azure-data-explorer-preview"></a>Requête interservice – Azure Monitor et Azure Data Explorer (préversion)
Créez des requêtes interservices entre [Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/), [Application Insights](/azure/azure-monitor/app/app-insights-overview) et [Log Analytics](/azure/azure-monitor/platform/data-platform-logs).
## <a name="azure-monitor-and-azure-data-explorer-cross-service-querying"></a>Interrogation interservice Azure Monitor et Azure Data Explorer
Cette expérience vous permet de [créer des requêtes interservices entre Azure Data Explorer et Azure Monitor](https://docs.microsoft.com/azure/data-explorer/query-monitor-data) et de [créer des requêtes interservices entre Azure Monitor et Azure Data Explorer](https://docs.microsoft.com/azure/azure-monitor/platform/azure-monitor-data-explorer-proxy).

:::image type="content" source="media\azure-data-explorer-monitor-proxy\azure-data-explorer-monitor-flow.png" alt-text="Flux de proxy Azure Data Explorer.":::

Par exemple, (interrogation d’Azure Data Explorer à partir de Log Analytics) :
```kusto
CustomEvents | where aField == 1
| join (adx("Help/Samples").TableName | where secondField == 3) on $left.Key == $right.key
```
Où la requête externe interroge une table de l’espace de travail, puis se joint à une autre table dans un cluster Azure Data Explorer (dans ce cas, clustername=help, databasename=samples) en utilisant la nouvelle fonction « adx() », comme vous pouvez le faire pour interroger un autre espace de travail à partir du texte de la requête.

> [!NOTE]
> * La fonctionnalité permettant d'interroger les données Azure Monitor à partir d'Azure Data Explorer, soit directement à partir des outils clients Azure Data Explorer, soit indirectement en exécutant une requête sur un cluster Azure Data Explorer, est en préversion.
> * Pour toute question, contactez l'équipe [Requête interservices](mailto:adxproxy@microsoft.com).

## <a name="query-exported-log-analytics-data-from-azure-blob-storage-account"></a>Interroger des données Log Analytics exportées à partir du compte Stockage Blob Azure

L’exportation de données à partir d’Azure Monitor vers un compte de stockage Azure permet une conservation à faible coût et la réallocation des journaux dans des régions différentes.

Utilisez Azure Data Explorer pour interroger les données exportées à partir de vos espaces de travail Log Analytics. Une fois configurées, les tables prises en charge qui sont envoyées à partir de vos espaces de travail vers un compte de stockage Azure seront disponibles en tant que source de données pour Azure Data Explorer. [Interroger les données exportées dans Azure Monitor avec Azure Data Explorer (préversion)](https://docs.microsoft.com/azure/azure-monitor/platform/azure-data-explorer-query-storage)

:::image type="content" source="media\azure-data-explorer-query-storage\exported-data-query.png" alt-text="Requête Azure Data Explorer à partir d’un flux de stockage":::

>[!tip] 
> * Pour exporter toutes les données de votre espace de travail Log Analytics vers un hub d’événements ou un compte de stockage Azure, utilisez la fonctionnalité d’exportation de données de l’espace de travail Log Analytics des journaux Azure Monitor. Voir [Exportation des données de l’espace de travail Log Analytics dans Azure Monitor (préversion)](https://docs.microsoft.com/azure/data-explorer/query-monitor-data).

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus :
* [Créer des requêtes interservices entre Azure Data Explorer et Azure Monitor](https://docs.microsoft.com/azure/data-explorer/query-monitor-data). Interroger des données Azure Monitor à partir d’Azure Data Explorer
* [Créer des requêtes interservices entre Azure Monitor et Azure Data Explorer](https://docs.microsoft.com/azure/azure-monitor/platform/azure-monitor-data-explorer-proxy). Interroger des données Azure Data Explorer à partir d’Azure Monitor
* [Exportation des données de l’espace de travail Log Analytics dans Azure Monitor (préversion)](https://docs.microsoft.com/azure/data-explorer/query-monitor-data). Liez et interrogez le compte Stockage Blob Azure avec les données exportées Log Analytics.