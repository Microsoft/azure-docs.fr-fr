---
title: Activité Switch dans Azure Data Factory
description: L’activité Switch vous permet de contrôler le flux de traitement en fonction d’une condition.
services: data-factory
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/08/2019
ms.openlocfilehash: fd5bb37d54183020e233774a778d27f5d4588bb4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486003"
---
# <a name="switch-activity-in-azure-data-factory"></a>Activité Switch dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’activité Switch fournit les mêmes fonctionnalités qu’une instruction switch dans les langages de programmation. Elle évalue un ensemble d’activités correspondant à un cas qui coïncide avec l’évaluation des conditions.

## <a name="syntax"></a>Syntaxe

```json

{
   "name": "<Name of the activity>",
    "type": "Switch",
    "typeProperties": {
        "expression": {
            "value": "<expression that evaluates to some string value>",
            "type": "Expression"
        },
        "cases": [
            {
                "value": "<string value that matches expression evaluation>",
                "activities": [
                    {
                        "<Activity 1 definition>"
                    },
                    {
                        "<Activity 2 definition>"
                    },
                    {
                        "<Activity N definition>"
                    }
                ]
            }           
        ],
        "defaultActivities": [
            {
                "<Activity 1 definition>"
            },
            {
                "<Activity 2 definition>"
            },
            {
                "<Activity N definition>"
            }
        ]
    }
}
```

## <a name="type-properties"></a>Propriétés type

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
name | Nom de l’activité Switch. | String | Oui
type | Doit être défini sur *Switch** | String | Oui
expression | Expression qui doit retourner une valeur de chaîne | Expression avec un résultat de type chaîne | Oui
cas | Ensemble de cas qui contiennent une valeur et ensemble d’activités à exécuter quand la valeur correspond à l’évaluation de l’expression. Doit fournir au moins un cas. La limite maximale est de 25 cas. | Tableau d’objets cas | Oui
defaultActivities | Ensemble d’activités qui sont exécutées quand les critères de l’évaluation de l’expression ne sont pas satisfaits. | Tableau d’activités | Oui

## <a name="example"></a>Exemple

Le pipeline dans cet exemple copie les données depuis un dossier d’entrée vers un dossier de sortie. Le dossier de sortie est déterminé par la valeur du paramètre de pipeline routeSelection.

> [!NOTE]
> Cette section fournit des définitions JSON et des exemples de commandes PowerShell pour exécuter le pipeline. Pour une procédure pas à pas avec des instructions détaillées permettant de créer un pipeline Data Factory en utilisant des définitions JSON et Azure PowerShell, consultez [Didacticiel : créer une fabrique de données avec Azure PowerShell](quickstart-create-data-factory-powershell.md).

### <a name="pipeline-with-switch-activity-adfv2quickstartpipelinejson"></a>Pipeline avec l’activité Switch (Adfv2QuickStartPipeline.json)

```json
{
    "name": "Adfv2QuickStartPipeline",
    "properties": {
        "activities": [
            {
                "name": "MySwitch",
                "type": "Switch",
                "typeProperties": {
                    "expression": {
                        "value": "@pipeline().parameters.routeSelection",
                        "type": "Expression"
                    },
                    "cases": [
                        {
                            "value": "1",
                            "activities": [
                                {
                                    "name": "CopyFromBlobToBlob1",
                                    "type": "Copy",
                                    "inputs": [
                                        {
                                            "referenceName": "BlobDataset",
                                            "parameters": {
                                                "path": "@pipeline().parameters.inputPath"
                                            },
                                            "type": "DatasetReference"
                                        }
                                    ],
                                    "outputs": [
                                        {
                                            "referenceName": "BlobDataset",
                                            "parameters": {
                                                "path": "@pipeline().parameters.outputPath1",
                                            },
                                            "type": "DatasetReference"
                                        }
                                    ],
                                    "typeProperties": {
                                        "source": {
                                            "type": "BlobSource"
                                        },
                                        "sink": {
                                            "type": "BlobSink"
                                        }
                                    }
                                }
                            ]
                        },
                        {
                            "value": "2",    
                                "activities": [
                                    {
                                        "name": "CopyFromBlobToBlob2",
                                        "type": "Copy",
                                        "inputs": [
                                            {
                                                "referenceName": "BlobDataset",
                                                "parameters": {
                                                    "path": "@pipeline().parameters.inputPath",
                                                },
                                                "type": "DatasetReference"
                                            }
                                        ],
                                        "outputs": [
                                            {
                                                "referenceName": "BlobDataset",
                                                "parameters": {
                                                    "path": "@pipeline().parameters.outputPath2",
                                                },
                                                "type": "DatasetReference"
                                            }
                                        ],
                                        "typeProperties": {
                                            "source": {
                                                "type": "BlobSource"
                                            },
                                            "sink": {
                                                "type": "BlobSink"
                                            }
                                        }
                                    }
                                ]
                            },
                            {
                                "value": "3",
                                "activities": [
                                    {
                                        "name": "CopyFromBlobToBlob3",
                                        "type": "Copy",
                                        "inputs": [
                                                {
                                                "referenceName": "BlobDataset",
                                                "parameters": {
                                                    "path": "@pipeline().parameters.inputPath",
                                                },
                                                "type": "DatasetReference"
                                            }
                                        ],
                                        "outputs": [
                                            {
                                                "referenceName": "BlobDataset",
                                                "parameters": {
                                                    "path": "@pipeline().parameters.outputPath3",
                                                },
                                                "type": "DatasetReference"
                                            }
                                        ],
                                        "typeProperties": {
                                            "source": {
                                                "type": "BlobSource"
                                            },
                                            "sink": {
                                                "type": "BlobSink"
                                            }
                                        }
                                    }
                                ]
                        },
                    ],
                    "defaultActivities": []
                }                    
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputPath1": {
                "type": "String"
            },
            "outputPath2": {
                "type": "String"
            },
            "outputPath3": {
                "type": "String"
            },
            "routeSelection": {
                "type": "String"
            }
        }
    }
}

```

### <a name="azure-storage-linked-service-azurestoragelinkedservicejson"></a>Service lié Stockage Azure (AzureStorageLinkedService.json)

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<Azure Storage account name>;AccountKey=<Azure Storage account key>"
        }
    }
}
```

### <a name="parameterized-azure-blob-dataset-blobdatasetjson"></a>Jeu de données d’objets Blob Azure paramétrable (BlobDataset.json)

Le pipeline définit **folderPath** sur la valeur de paramètre **outputPath1** ou **outputPath2** du pipeline. 

```json
{
    "name": "BlobDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": {
                "value": "@{dataset().path}",
                "type": "Expression"
            }
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "path": {
                "type": "String"
            }
        }
    }
}
```

### <a name="pipeline-parameter-json-pipelineparametersjson"></a>Code JSON des paramètres du pipeline (PipelineParameters.json)

```json
{
    "inputPath": "adftutorial/input",
    "outputPath1": "adftutorial/outputCase1",
    "outputPath2": "adftutorial/outputCase2",
    "outputPath2": "adftutorial/outputCase3",
    "routeSelection": "1"
}
```

### <a name="powershell-commands"></a>Commandes PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Ces commandes supposent que vous avez enregistré les fichiers JSON dans le dossier : C:\ADF. 

```powershell
Connect-AzAccount
Select-AzSubscription "<Your subscription name>"

$resourceGroupName = "<Resource Group Name>"
$dataFactoryName = "<Data Factory Name. Must be globally unique>";
Remove-AzDataFactoryV2 $dataFactoryName -ResourceGroupName $resourceGroupName -force


Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName
Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -DefinitionFile "C:\ADF\AzureStorageLinkedService.json"
Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "BlobDataset" -DefinitionFile "C:\ADF\BlobDataset.json"
Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "Adfv2QuickStartPipeline" -DefinitionFile "C:\ADF\Adfv2QuickStartPipeline.json"
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName "Adfv2QuickStartPipeline" -ParameterFile C:\ADF\PipelineParameters.json
while ($True) {
    $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

    if ($run) {
        if ($run.Status -ne 'InProgress') {
            Write-Host "Pipeline run finished. The status is: " $run.Status -foregroundcolor "Yellow"
            $run
            break
        }
        Write-Host  "Pipeline is running...status: InProgress" -foregroundcolor "Yellow"
    }

    Start-Sleep -Seconds 30
}
Write-Host "Activity run details:" -foregroundcolor "Yellow"
$result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
$result

Write-Host "Activity 'Output' section:" -foregroundcolor "Yellow"
$result.Output -join "`r`n"

Write-Host "\nActivity 'Error' section:" -foregroundcolor "Yellow"
$result.Error -join "`r`n"
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les autres activités de flux de contrôle prises en charge par Data Factory : 

- [Activité IfCondition](control-flow-if-condition-activity.md)
- [Activité d’exécution du pipeline](control-flow-execute-pipeline-activity.md)
- [Pour chaque activité](control-flow-for-each-activity.md)
- [Activité d’obtention des métadonnées](control-flow-get-metadata-activity.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité Web](control-flow-web-activity.md)
