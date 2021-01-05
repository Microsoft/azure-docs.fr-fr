---
title: Interroger les données exportées dans Azure Monitor avec Azure Data Explorer (préversion)
description: Utilisez Azure Data Explorer pour interroger les données exportées à partir de votre espace de travail Log Analytics vers un compte de stockage Azure.
ms.subservice: logs
author: orens
ms.author: bwren
ms.reviewer: bwren
ms.topic: conceptual
ms.date: 10/13/2020
ms.openlocfilehash: 2735aec0f95f5e282bb1dffba6e4f42f966cf117
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186539"
---
# <a name="query-exported-data-from-azure-monitor-using-azure-data-explorer-preview"></a>Interroger les données exportées dans Azure Monitor avec Azure Data Explorer (préversion)
L’exportation de données à partir d’Azure Monitor vers un compte de stockage Azure permet une conservation à faible coût et la réallocation des journaux dans des régions différentes. Utilisez Azure Data Explorer pour interroger les données exportées à partir de vos espaces de travail Log Analytics. Une fois configurées, les tables prises en charge qui sont envoyées à partir de vos espaces de travail vers un compte de stockage Azure seront disponibles en tant que source de données pour Azure Data Explorer.

Le flux de processus est le suivant : 

1.  Exportez les données de l’espace de travail Log Analytics vers un compte de stockage Azure.
2.  Créez une table externe dans votre cluster Azure Data Explorer et un mappage pour les types de données.
3.  Interrogez les données depuis Azure Data Explorer.

:::image type="content" source="media/azure-data-explorer-query-storage/exported-data-query.png" alt-text="Flux d’interrogation des données exportées Azure Data Explorer.":::



## <a name="send-data-to-azure-storage"></a>Envoyer des données vers le stockage Azure
Les journaux Azure Monitor peuvent être exportés vers un compte de stockage Azure à l’aide d’une des options suivantes.

- Pour exporter toutes les données de votre espace de travail Log Analytics vers un hub d’événements ou un compte de stockage Azure, utilisez la fonctionnalité d’exportation de données de l’espace de travail Log Analytics des journaux Azure Monitor. Consultez [Exportation des données de l’espace de travail Log Analytics dans Azure Monitor (préversion)](logs-data-export.md).
- Exportation planifiée à partir d’une requête de journal à l’aide d’une application logique. Cela est similaire à la fonctionnalité d’exportation de données, mais vous permet d’envoyer des données filtrées ou agrégées vers le stockage Azure. Toutefois, cette méthode est sujette aux [limites des requêtes de journal](../service-limits.md#log-analytics-workspaces). Consultez [Archiver des données de l’espace de travail Log Analytics dans le stockage Azure à l’aide d’une application logique](logs-export-logic-app.md).
- Exportation unique à l’aide d’une application logique. Consultez [Connecteur Azure Monitor Logs pour Logic Apps et Power Automate](logicapp-flow-connector.md).
- Exportation unique vers l’ordinateur local à l’aide d’un script PowerShell. Consultez [Invoke-AzOperationalInsightsQueryExport](https://www.powershellgallery.com/packages/Invoke-AzOperationalInsightsQueryExport).

> [!TIP]
> Vous pouvez utiliser un cluster Azure Data Explorer existant ou créer un nouveau cluster dédié avec les configurations nécessaires.

## <a name="create-an-external-table-located-in-azure-blob-storage"></a>Créer une table externe dans le Stockage Blob Azure
Utilisez des [tables externes](/azure/data-explorer/kusto/query/schema-entities/externaltables) pour lier Azure Data Explorer à un compte de stockage Azure. Une table externe est une entité de schéma Kusto qui fait référence à des données stockées en dehors d’une base de données Kusto. Comme les tables, une table externe présente un schéma bien défini. Contrairement aux tables, les données sont stockées et gérées en dehors d’un cluster Kusto. Les données exportées de la section précédente sont enregistrées dans des lignes JSON.

Pour créer une référence, vous avez besoin du schéma de la table exportée. Utilisez l’opérateur [getschema](/azure/data-explorer/kusto/query/getschemaoperator) de Log Analytics pour récupérer ces informations, notamment les colonnes de la table et leurs types de données.

:::image type="content" source="media\azure-data-explorer-query-storage\exported-data-map-schema.jpg" alt-text="Schéma de table Log Analytics.":::

Vous pouvez maintenant utiliser la sortie pour créer la requête Kusto afin de construire la table externe.
En suivant les instructions fournies dans [Créer et modifier des tables externes dans Stockage Azure ou Azure Data Lake](/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake), créez une table externe dans un format JSON, puis exécutez la requête à partir de votre base de données Azure Data Explorer.

>[!NOTE]
>La création de la table externe se base sur deux processus. Le premier consiste à créer la table externe, tandis que le second crée le mappage.

Le script PowerShell suivant crée les commandes [create](/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake#create-external-table-mapping) pour la table et le mappage.

```powershell
PARAM(
    $resourcegroupname, #The name of the Azure resource group
    $TableName, # The Log Analytics table you wish to convert to external table
    $MapName, # The name of the map
    $subscriptionId, #The ID of the subscription
    $WorkspaceId, # The Log Analytics WorkspaceId
    $WorkspaceName, # The Log Analytics workspace name
    $BlobURL, # The Blob URL where the data is saved
    $ContainerAccessKey, # The blob container Access Key (Option to add a SAS url)
    $ExternalTableName = $null # The External Table name, null to use the same name
)

if($null -eq $ExternalTableName) {
    $ExternalTableName = $TableName
}

$query = $TableName + ' | getschema | project ColumnName, DataType'

$output = (Invoke-AzOperationalInsightsQuery -WorkspaceId $WorkspaceId -Query $query).Results

$FirstCommand = @()
$SecondCommand = @()

foreach ($record in $output) {
    if ($record.DataType -eq 'System.DateTime') {
        $dataType = 'datetime'
    } else {
        $dataType = 'string'
    }
    $FirstCommand += $record.ColumnName + ":" + "$dataType" + ","
    $SecondCommand += "{`"column`":" + "`"" + $record.ColumnName + "`"," + "`"datatype`":`"$dataType`",`"path`":`"$." + $record.ColumnName + "`"},"
}
$schema = ($FirstCommand -join '') -replace ',$'
$mapping = ($SecondCommand -join '') -replace ',$'

$CreateExternal = @'
.create external table {0} ({1})
kind=blob
partition by (TimeGeneratedPartition:datetime = bin(TimeGenerated, 1min))
pathformat = (datetime_pattern("'y='yyyy'/m='MM'/d='dd'/h='HH'/m='mm", TimeGeneratedPartition))
dataformat=multijson
(
   h@'{2}/WorkspaceResourceId=/subscriptions/{4}/resourcegroups/{6}/providers/microsoft.operationalinsights/workspaces/{5};{3}'
)
with
(
   docstring = "Docs",
   folder = "ExternalTables"
)
'@ -f $TableName, $schema, $BlobURL, $ContainerAccessKey, $subscriptionId, $WorkspaceName, $resourcegroupname,$WorkspaceId

$createMapping = @'
.create external table {0} json mapping "{1}" '[{2}]'
'@ -f $ExternalTableName, $MapName, $mapping

Write-Host -ForegroundColor Red 'Copy and run the following commands (one by one), on your Azure Data Explorer cluster query window to create the external table and mappings:'
write-host -ForegroundColor Green $CreateExternal
Write-Host -ForegroundColor Green $createMapping
```

L’image suivante montre un exemple de sortie.

:::image type="content" source="media/azure-data-explorer-query-storage/external-table-create-command-output.png" alt-text="Sortie des commandes create external table.":::

[![Exemple de sortie](media/azure-data-explorer-query-storage/external-table-create-command-output.png)](media/azure-data-explorer-query-storage/external-table-create-command-output.png#lightbox)

>[!TIP]
>* Copiez, collez, puis exécutez la sortie du script dans votre outil client Azure Data Explorer pour créer la table et le mappage.
>* Pour utiliser toutes les données à l’intérieur du conteneur, modifiez le script et remplacez l’URL par « https://your.blob.core.windows.net/containername;SecKey »

## <a name="query-the-exported-data-from-azure-data-explorer"></a>Interroger les données exportées à partir d’Azure Data Explorer 

Après avoir configuré le mappage, vous pouvez interroger les données exportées à partir d’Azure Data Explorer. Votre requête exige la fonction [external_table](/azure/data-explorer/kusto/query/externaltablefunction), comme dans l’exemple suivant.

```kusto
external_table("HBTest","map") | take 10000
```

[![Interroger des données exportées dans Log Analytics](media/azure-data-explorer-query-storage/external-table-query.png)](media/azure-data-explorer-query-storage/external-table-query.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [écrire des requêtes dans Azure Data Explorer](/azure/data-explorer/write-queries).