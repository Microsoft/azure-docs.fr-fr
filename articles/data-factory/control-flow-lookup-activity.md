---
title: Activité de recherche dans Azure Data Factory
description: Découvrez comment utiliser l’activité Lookup pour rechercher une valeur à partir d’une source externe. Cette sortie peut être référencée davantage par des activités complémentaires.
services: data-factory
documentationcenter: ''
author: linda33wj
ms.author: jingwang
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 66a17b61fef652160dc6d4a02bf330adbf0c7362
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006818"
---
# <a name="lookup-activity-in-azure-data-factory"></a>Activité de recherche dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’activité Lookup peut récupérer un jeu de données à partir de n’importe quelle source de données compatible Azure Data Factory. Utilisez-la dans le scénario suivant :
- Déterminez de manière dynamique sur quels objets intervenir dans une activité ultérieure, au lieu de coder en dur le nom d’objet. Les fichiers et les tables constituent quelques exemples d’objet.

L’activité Lookup lit et retourne le contenu d’une table ou d’un fichier de configuration. Elle retourne également le résultat de l’exécution d’une requête ou d’une procédure stockée. La sortie de l’activité Lookup peut être utilisée dans une activité de transformation ou de copie ultérieure s’il s’agit d’une valeur singleton. La sortie peut être utilisée dans une activité ForEach, s’il s’agit d’un tableau d’attributs.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Les sources de données suivantes sont prises en charge pour l’activité Lookup. 

L’activité de recherche peut retourner jusqu’à 5 000 lignes ; si le jeu de résultats contient plus d’enregistrements, les 5 000 premières lignes sont retournées. La sortie de l’activité de recherche prend en charge une taille maximale d’environ 4 Mo ; l’activité échoue si la taille dépasse la limite. Actuellement, la durée la plus longue pour l’activité Lookup avant l’expiration du délai d’attente est de 24 heures.

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores-for-lookup-activity.md)]

## <a name="syntax"></a>Syntaxe

```json
{
    "name":"LookupActivity",
    "type":"Lookup",
    "typeProperties":{
        "source":{
            "type":"<source type>"
        },
        "dataset":{
            "referenceName":"<source dataset name>",
            "type":"DatasetReference"
        },
        "firstRowOnly":<true or false>
    }
}
```

## <a name="type-properties"></a>Propriétés type

Nom | Description | Type | Requis ?
---- | ----------- | ---- | --------
dataset | Fournit la référence de jeu de données pour la recherche. Pour plus d’informations, voir la section **Propriétés du jeu de données** dans chaque article traitant du connecteur correspondant. | Paire clé/valeur | Oui
source | Contient des propriétés sources spécifiques au jeu de données, identiques à la source de l’activité Copy. Pour plus d’informations, consultez la section **Propriétés de l’activité Copy** dans chaque article traitant du connecteur correspondant. | Paire clé/valeur | Oui
firstRowOnly | Indique s’il faut retourner uniquement la première ligne ou toutes les lignes. | Boolean | Non. Par défaut, il s’agit de `true`.

> [!NOTE]
> 
> * Les colonnes sources avec le type **ByteArray** ne sont pas prises en charge.
> * Il n’y a pas de prise en charge pour **Structure** dans les définitions des jeux de données. Pour les fichiers de format texte, utilisez la ligne d’en-tête pour mentionner le nom de la colonne.
> * Si votre source de recherche est un fichier JSON, le paramètre `jsonPathDefinition` pour la mise en forme de l’objet JSON n’est pas pris en charge. Les objets entiers sont récupérés.

## <a name="use-the-lookup-activity-result"></a>Utiliser le résultat de l’activité de recherche

Le résultat de la recherche est retourné dans la section `output` du résultat de l’exécution d’activité.

* **Quand `firstRowOnly` a la valeur `true` (par défaut)** , le format de la sortie se présente comme dans le code suivant. Le résultat de la recherche se trouve dans une clé `firstRow` fixe. Pour utiliser le résultat dans une activité suivante, utilisez le modèle de `@{activity('LookupActivity').output.firstRow.table}`.

    ```json
    {
        "firstRow":
        {
            "Id": "1",
            "schema":"dbo",
            "table":"Table1"
        }
    }
    ```

* **Quand `firstRowOnly` a la valeur `false`** , le format de la sortie se présente comme dans le code suivant. Un champ `count` indique le nombre d’enregistrements qui sont retournés. Les valeurs détaillées sont affichées sous un tableau `value` fixe. Dans ce cas, l’activité Lookup est suivie d’une [activité Foreach](control-flow-for-each-activity.md). Vous passez le tableau `value` au champ `items` de l’activité ForEach en utilisant le modèle `@activity('MyLookupActivity').output.value`. Pour accéder aux éléments du tableau `value`, utilisez la syntaxe suivante : `@{activity('lookupActivity').output.value[zero based index].propertyname}`. par exemple `@{activity('lookupActivity').output.value[0].schema}`.

    ```json
    {
        "count": "2",
        "value": [
            {
                "Id": "1",
                "schema":"dbo",
                "table":"Table1"
            },
            {
                "Id": "2",
                "schema":"dbo",
                "table":"Table2"
            }
        ]
    } 
    ```

## <a name="example"></a>Exemple

Dans cet exemple, le pipeline comprend deux activités : **Lookup** et **Copy**. L’activité Copy copie les données d’une table SQL de votre instance Azure SQL Database dans le stockage Blob Azure. Le nom de la table SQL est stocké dans un fichier JSON dans le stockage Blob. L’activité Lookup recherche le nom de la table lors de l’exécution. Le fichier JSON est modifié de manière dynamique avec cette approche. Vous n’avez pas besoin de redéployer les pipelines ou les jeux de données. 

Cet exemple illustre une recherche pour la première ligne uniquement. Pour effectuer une recherche portant sur toutes les lignes et chaîner les résultats avec l’activité ForEach, consultez les exemples dans [Copier plusieurs tables en bloc à l’aide d’Azure Data Factory](tutorial-bulk-copy.md).


### <a name="pipeline"></a>Pipeline

- L’activité Lookup est configurée pour utiliser **LookupDataset**, qui fait référence à un emplacement dans le stockage Blob Azure. L’activité Lookup lit le nom de la table SQL à partir d’un fichier JSON dans cet emplacement. 
- L’activité Copy utilise la sortie de l’activité Lookup, qui correspond au nom de la table SQL. La propriété **tableName** dans **SourceDataset** est configurée pour utiliser la sortie de l’activité Lookup. L’activité Copy copie les données de la table SQL dans un emplacement du stockage Blob Azure. L’emplacement est spécifié par la propriété **SinkDataset**. 

```json
{
    "name": "LookupPipelineDemo",
    "properties": {
        "activities": [
            {
                "name": "LookupActivity",
                "type": "Lookup",
                "dependsOn": [],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "source": {
                        "type": "JsonSource",
                        "storeSettings": {
                            "type": "AzureBlobStorageReadSettings",
                            "recursive": true
                        },
                        "formatSettings": {
                            "type": "JsonReadSettings"
                        }
                    },
                    "dataset": {
                        "referenceName": "LookupDataset",
                        "type": "DatasetReference"
                    },
                    "firstRowOnly": true
                }
            },
            {
                "name": "CopyActivity",
                "type": "Copy",
                "dependsOn": [
                    {
                        "activity": "LookupActivity",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "userProperties": [],
                "typeProperties": {
                    "source": {
                        "type": "AzureSqlSource",
                        "sqlReaderQuery": {
                            "value": "select * from [@{activity('LookupActivity').output.firstRow.schema}].[@{activity('LookupActivity').output.firstRow.table}]",
                            "type": "Expression"
                        },
                        "queryTimeout": "02:00:00",
                        "partitionOption": "None"
                    },
                    "sink": {
                        "type": "DelimitedTextSink",
                        "storeSettings": {
                            "type": "AzureBlobStorageWriteSettings"
                        },
                        "formatSettings": {
                            "type": "DelimitedTextWriteSettings",
                            "quoteAllText": true,
                            "fileExtension": ".txt"
                        }
                    },
                    "enableStaging": false,
                    "translator": {
                        "type": "TabularTranslator",
                        "typeConversion": true,
                        "typeConversionSettings": {
                            "allowDataTruncation": true,
                            "treatBooleanAsNumber": false
                        }
                    }
                },
                "inputs": [
                    {
                        "referenceName": "SourceDataset",
                        "type": "DatasetReference",
                        "parameters": {
                            "schemaName": {
                                "value": "@activity('LookupActivity').output.firstRow.schema",
                                "type": "Expression"
                            },
                            "tableName": {
                                "value": "@activity('LookupActivity').output.firstRow.table",
                                "type": "Expression"
                            }
                        }
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "SinkDataset",
                        "type": "DatasetReference",
                        "parameters": {
                            "schema": {
                                "value": "@activity('LookupActivity').output.firstRow.schema",
                                "type": "Expression"
                            },
                            "table": {
                                "value": "@activity('LookupActivity').output.firstRow.table",
                                "type": "Expression"
                            }
                        }
                    }
                ]
            }
        ],
        "annotations": [],
        "lastPublishTime": "2020-08-17T10:48:25Z"
    }
}
```

### <a name="lookup-dataset"></a>Jeu de données de recherche

Le jeu de données **lookup** est le fichier **sourcetable.json** dans le dossier de recherche du stockage Azure spécifié par le type **AzureBlobStorageLinkedService**. 

```json
{
    "name": "LookupDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "Json",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": "sourcetable.json",
                "container": "lookup"
            }
        }
    }
}
```

### <a name="source-dataset-for-copy-activity"></a>Jeu de données **source** pour l’activité Copy

Le jeu de données **source** utilise la sortie de l’activité Lookup, qui correspond au nom de la table SQL. L’activité Copy copie les données de cette table SQL dans un emplacement du stockage Blob Azure. L’emplacement est indiqué par le jeu de données **sink**. 

```json
{
    "name": "SourceDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureSqlDatabase",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "schemaName": {
                "type": "string"
            },
            "tableName": {
                "type": "string"
            }
        },
        "annotations": [],
        "type": "AzureSqlTable",
        "schema": [],
        "typeProperties": {
            "schema": {
                "value": "@dataset().schemaName",
                "type": "Expression"
            },
            "table": {
                "value": "@dataset().tableName",
                "type": "Expression"
            }
        }
    }
}
```

### <a name="sink-dataset-for-copy-activity"></a>Jeu de données **sink** pour l’activité Copy

L’activité Copy copie les données de la table SQL dans le fichier **filebylookup.csv** du dossier **csv** du stockage Azure. Le fichier est spécifié par la propriété **AzureBlobStorageLinkedService**. 

```json
{
    "name": "SinkDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AzureBlobStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "schema": {
                "type": "string"
            },
            "table": {
                "type": "string"
            }
        },
        "annotations": [],
        "type": "DelimitedText",
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "fileName": {
                    "value": "@{dataset().schema}_@{dataset().table}.csv",
                    "type": "Expression"
                },
                "container": "csv"
            },
            "columnDelimiter": ",",
            "escapeChar": "\\",
            "quoteChar": "\""
        },
        "schema": []
    }
}
```

### <a name="sourcetablejson"></a>sourcetable.json

Vous pouvez utiliser deux types de formats pour le fichier **sourcetable.json**.

#### <a name="set-of-objects"></a>Ensemble d’objets

```json
{
   "Id":"1",
   "schema":"dbo",
   "table":"Table1"
}
{
   "Id":"2",
   "schema":"dbo",
   "table":"Table2"
}
```

#### <a name="array-of-objects"></a>Tableau d’objets

```json
[ 
    {
        "Id": "1",
        "schema":"dbo",
        "table":"Table1"
    },
    {
        "Id": "2",
        "schema":"dbo",
        "table":"Table2"
    }
]
```

## <a name="limitations-and-workarounds"></a>Limitations et solutions de contournement

Voici quelques limitations de l’activité de recherche et des suggestions de solutions de contournement.

| Limitation | Solution de contournement |
|---|---|
| L’activité de recherche a un maximum de 5 000 lignes et une taille maximale de 2 Mo. | Concevoir un pipeline sur deux niveaux où le pipeline externe itère sur un pipeline interne, qui récupère les données ne dépassant pas le nombre maximal de lignes ou la taille de conception. |
| | |

## <a name="next-steps"></a>Étapes suivantes
Consultez les autres activités de flux de contrôle prises en charge par Data Factory : 

- [Activité d’exécution du pipeline](control-flow-execute-pipeline-activity.md)
- [Activité ForEach](control-flow-for-each-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)
- [Activité Web](control-flow-web-activity.md)
