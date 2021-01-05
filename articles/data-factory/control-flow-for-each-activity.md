---
title: Activité ForEach dans Azure Data Factory
description: L’activité ForEach définit un flux de contrôle répétitif dans votre pipeline. Elle permet d’effectuer une itération sur une collection et d’exécuter des activités spécifiées.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/23/2019
ms.openlocfilehash: 71e96e6245d4cf922b82162e01a972264699f3ac
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499507"
---
# <a name="foreach-activity-in-azure-data-factory"></a>Activité ForEach dans Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’activité ForEach définit un flux de contrôle répétitif dans votre pipeline. Elle permet d’effectuer une itération sur une collection, et exécute des activités spécifiées dans une boucle. L’implémentation en boucle de cette activité est semblable à la structure d’exécution en boucle de Foreach dans les langages de programmation.

## <a name="syntax"></a>Syntaxe
Les propriétés sont décrites plus loin dans cet article. La propriété items est la collection, et chaque élément dans celle-ci est référencé à l’aide de `@item()`, comme illustré dans la syntaxe suivante :  

```json
{  
   "name":"MyForEachActivityName",
   "type":"ForEach",
   "typeProperties":{  
      "isSequential":"true",
        "items": {
            "value": "@pipeline().parameters.mySinkDatasetFolderPathCollection",
            "type": "Expression"
        },
      "activities":[  
         {  
            "name":"MyCopyActivity",
            "type":"Copy",
            "typeProperties":{  
               ...
            },
            "inputs":[  
               {  
                  "referenceName":"MyDataset",
                  "type":"DatasetReference",
                  "parameters":{  
                     "MyFolderPath":"@pipeline().parameters.mySourceDatasetFolderPath"
                  }
               }
            ],
            "outputs":[  
               {  
                  "referenceName":"MyDataset",
                  "type":"DatasetReference",
                  "parameters":{  
                     "MyFolderPath":"@item()"
                  }
               }
            ]
         }
      ]
   }
}

```

## <a name="type-properties"></a>Propriétés type

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
name | Nom de l’activité ForEach. | String | Oui
type | Doit être défini sur **ForEach** | String | Oui
isSequential | Spécifie si la boucle doit être exécutée de façon séquentielle ou parallèle.  Le nombre maximal d’itérations de boucle exécutables simultanément en parallèle est de 20. Par exemple, si vous avez une activité ForEach effectuant une itération sur une activité de copie portant sur 10 jeux de données de source et de récepteur différents avec la valeur **isSequential** définie sur False, toutes les copies sont exécutées en même temps. La valeur par défaut est FALSE. <br/><br/> Si la valeur de « isSequential » est définie sur False, assurez-vous qu’il existe une configuration correcte pour exécuter plusieurs exécutables. Autrement, cette propriété doit être utilisée avec précaution pour éviter des conflits d’écriture. Pour plus d’informations, voir la section [Exécution parallèle](#parallel-execution). | Boolean | Non. La valeur par défaut est FALSE.
batchCount | Nombre de lots à utiliser pour contrôler le nombre d’exécutions en parallèle (lorsque isSequential est défini sur false). Il s’agit de la limite supérieure de concurrence, mais l’activité ForEach n’atteindra pas toujours ce nombre lors de son exécution. | Entier (maximum 50) | Non. La valeur par défaut est 20.
Éléments | Expression qui retourne un tableau JSON auquel appliquer l’itération. | Expression (qui retourne un tableau JSON) | Oui
Activités | Activités à exécuter. | Liste des activités | Oui

## <a name="parallel-execution"></a>Exécution parallèle
Si la valeur de **isSequential** est définie sur False, l’activité effectue l’itération en parallèle avec un maximum de 20 itérations simultanées. Ce paramètre doit être utilisé avec précaution. Si les itérations simultanées écrivent dans le même dossier, mais dans des fichiers différents, cette approche convient. Si les itérations simultanées écrivent en même temps dans le même fichier, cette approche entraînera très probablement une erreur. 

## <a name="iteration-expression-language"></a>Langage d’expression de l’itération
Dans l’activité ForEach, fournissez un tableau sur lequel effectuer l’itération pour la propriété **items**. Utilisez `@item()` pour itérer sur une seule énumération dans l’activité ForEach. Par exemple, si **items** est un tableau [1, 2, 3], `@item()` retourne 1 dans la première itération, 2 dans la deuxième, et 3 dans la troisième.

## <a name="iterating-over-a-single-activity"></a>Itération sur une activité unique
**Scénario :** Copier à partir du même fichier source dans Stockage Blob Azure vers plusieurs fichiers de destination dans Stockage Blob Azure.

### <a name="pipeline-definition"></a>Définition de pipeline

```json
{
    "name": "<MyForEachPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyForEachActivity>",
                "type": "ForEach",
                "typeProperties": {
                    "isSequential": "true",
                    "items": {
                        "value": "@pipeline().parameters.mySinkDatasetFolderPath",
                        "type": "Expression"
                    },
                    "activities": [
                        {
                            "name": "MyCopyActivity",
                            "type": "Copy",
                            "typeProperties": {
                                "source": {
                                    "type": "BlobSource",
                                    "recursive": "false"
                                },
                                "sink": {
                                    "type": "BlobSink",
                                    "copyBehavior": "PreserveHierarchy"
                                }
                            },
                            "inputs": [
                                {
                                    "referenceName": "<MyDataset>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "MyFolderPath": "@pipeline().parameters.mySourceDatasetFolderPath"
                                    }
                                }
                            ],
                            "outputs": [
                                {
                                    "referenceName": "MyDataset",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "MyFolderPath": "@item()"
                                    }
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "mySourceDatasetFolderPath": {
                "type": "String"
            },
            "mySinkDatasetFolderPath": {
                "type": "String"
            }
        }
    }
}

```

### <a name="blob-dataset-definition"></a>Définition du jeu de données d'objet blob

```json
{  
   "name":"<MyDataset>",
   "properties":{  
      "type":"AzureBlob",
      "typeProperties":{  
         "folderPath":{  
            "value":"@dataset().MyFolderPath",
            "type":"Expression"
         }
      },
      "linkedServiceName":{  
         "referenceName":"StorageLinkedService",
         "type":"LinkedServiceReference"
      },
      "parameters":{  
         "MyFolderPath":{  
            "type":"String"
         }
      }
   }
}

```

### <a name="run-parameter-values"></a>Valeurs de paramètre d’exécution

```json
{
    "mySourceDatasetFolderPath": "input/",
    "mySinkDatasetFolderPath": [ "outputs/file1", "outputs/file2" ]
}

```

## <a name="iterate-over-multiple-activities"></a>Itérer sur plusieurs activités
Il est possible d’itérer sur plusieurs activités (par exemple : des activités web et de copie) dans une activité ForEach. Dans ce scénario, nous vous recommandons d’abstraire plusieurs activités dans un pipeline distinct. Ensuite, vous pouvez utiliser l’[activité ExecutePipeline](control-flow-execute-pipeline-activity.md) dans le pipeline avec l’activité ForEach pour appeler le pipeline séparé avec plusieurs activités. 


### <a name="syntax"></a>Syntaxe

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ForEach",
        "name": "<MyForEachMultipleActivities>"
        "typeProperties": {
          "isSequential": true,
          "items": {
            ...
          },
          "activities": [
            {
              "type": "ExecutePipeline",
              "name": "<MyInnerPipeline>"
              "typeProperties": {
                "pipeline": {
                  "referenceName": "<copyHttpPipeline>",
                  "type": "PipelineReference"
                },
                "parameters": {
                  ...
                },
                "waitOnCompletion": true
              }
            }
          ]
        }
      }
    ],
    "parameters": {
      ...
    }
  }
}

```

### <a name="example"></a>Exemple
**Scénario :** Itérer sur un pipeline interne au sein d’une activité ForEach avec l’activité d’exécution du pipeline. Le pipeline interne copie avec des définitions de schéma paramétrées.

#### <a name="master-pipeline-definition"></a>Définition du pipeline principal

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ForEach",
        "name": "MyForEachActivity",
        "typeProperties": {
          "isSequential": true,
          "items": {
            "value": "@pipeline().parameters.inputtables",
            "type": "Expression"
          },
          "activities": [
            {
              "type": "ExecutePipeline",
              "typeProperties": {
                "pipeline": {
                  "referenceName": "InnerCopyPipeline",
                  "type": "PipelineReference"
                },
                "parameters": {
                  "sourceTableName": {
                    "value": "@item().SourceTable",
                    "type": "Expression"
                  },
                  "sourceTableStructure": {
                    "value": "@item().SourceTableStructure",
                    "type": "Expression"
                  },
                  "sinkTableName": {
                    "value": "@item().DestTable",
                    "type": "Expression"
                  },
                  "sinkTableStructure": {
                    "value": "@item().DestTableStructure",
                    "type": "Expression"
                  }
                },
                "waitOnCompletion": true
              },
              "name": "ExecuteCopyPipeline"
            }
          ]
        }
      }
    ],
    "parameters": {
      "inputtables": {
        "type": "Array"
      }
    }
  }
}

```

#### <a name="inner-pipeline-definition"></a>Définition de pipeline interne

```json
{
  "name": "InnerCopyPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            }
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "name": "CopyActivity",
        "inputs": [
          {
            "referenceName": "sqlSourceDataset",
            "parameters": {
              "SqlTableName": {
                "value": "@pipeline().parameters.sourceTableName",
                "type": "Expression"
              },
              "SqlTableStructure": {
                "value": "@pipeline().parameters.sourceTableStructure",
                "type": "Expression"
              }
            },
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "sqlSinkDataset",
            "parameters": {
              "SqlTableName": {
                "value": "@pipeline().parameters.sinkTableName",
                "type": "Expression"
              },
              "SqlTableStructure": {
                "value": "@pipeline().parameters.sinkTableStructure",
                "type": "Expression"
              }
            },
            "type": "DatasetReference"
          }
        ]
      }
    ],
    "parameters": {
      "sourceTableName": {
        "type": "String"
      },
      "sourceTableStructure": {
        "type": "String"
      },
      "sinkTableName": {
        "type": "String"
      },
      "sinkTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="source-dataset-definition"></a>Définition du jeu de données de la source

```json
{
  "name": "sqlSourceDataset",
  "properties": {
    "type": "SqlServerTable",
    "typeProperties": {
      "tableName": {
        "value": "@dataset().SqlTableName",
        "type": "Expression"
      }
    },
    "structure": {
      "value": "@dataset().SqlTableStructure",
      "type": "Expression"
    },
    "linkedServiceName": {
      "referenceName": "sqlserverLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "SqlTableName": {
        "type": "String"
      },
      "SqlTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="sink-dataset-definition"></a>Définition du jeu de données du récepteur

```json
{
  "name": "sqlSinkDataSet",
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": {
        "value": "@dataset().SqlTableName",
        "type": "Expression"
      }
    },
    "structure": {
      "value": "@dataset().SqlTableStructure",
      "type": "Expression"
    },
    "linkedServiceName": {
      "referenceName": "azureSqlLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "SqlTableName": {
        "type": "String"
      },
      "SqlTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="master-pipeline-parameters"></a>Paramètres du pipeline principal
```json
{
    "inputtables": [
        {
            "SourceTable": "department",
            "SourceTableStructure": [
              {
                "name": "departmentid",
                "type": "int"
              },
              {
                "name": "departmentname",
                "type": "string"
              }
            ],
            "DestTable": "department2",
            "DestTableStructure": [
              {
                "name": "departmentid",
                "type": "int"
              },
              {
                "name": "departmentname",
                "type": "string"
              }
            ]
        }
    ]
    
}
```

## <a name="aggregating-outputs"></a>Agréger des sorties

Pour agréger les sorties de l’activité __foreach__, utilisez des _Variables_ et l’activité _Append Variable_.

Commencez par déclarer une _variable_ `array` dans le pipeline. Puis appelez l'activité _Append Variable_ dans chaque boucle __ForEach__. Vous pouvez ensuite récupérer l'agrégation à partir de votre tableau.

## <a name="limitations-and-workarounds"></a>Limitations et solutions de contournement

Voici quelques limitations de l’activité ForEach et des suggestions de solutions de contournement.

| Limitation | Solution de contournement |
|---|---|
| Vous ne pouvez pas imbriquer une boucle ForEach à l’intérieur d’une autre boucle ForEach (ou une boucle Until). | Concevez un pipeline à deux niveaux où le pipeline externe contenant la boucle ForEach externe itère sur un pipeline interne contenant la boucle imbriquée. |
| Pour chaque activité ForEach, `batchCount` a une valeur maximale de 50 pour le traitement parallèle et un maximum de 100 000 éléments. | Concevez un pipeline à deux niveaux où le pipeline externe contenant l’activité ForEach itère sur un pipeline interne. |
| | |

## <a name="next-steps"></a>Étapes suivantes
Consultez les autres activités de flux de contrôle prises en charge par Data Factory : 

- [Activité d’exécution du pipeline](control-flow-execute-pipeline-activity.md)
- [Activité d’obtention des métadonnées](control-flow-get-metadata-activity.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité Web](control-flow-web-activity.md)
