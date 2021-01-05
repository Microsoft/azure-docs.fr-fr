---
title: Transformer des données avec une activité Hive – Azure
description: Découvrez comment vous pouvez utiliser l’activité Hive dans Azure Data Factory v1 pour exécuter des requêtes Hive sur un cluster HDInsight à la demande ou sur votre propre cluster.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.assetid: 80083218-743e-4da8-bdd2-60d1c77b1227
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 8a44838076b80c1b745937cf44f241c40ce6e5c2
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510156"
---
# <a name="transform-data-using-hive-activity-in-azure-data-factory"></a>Transformer des données à l’aide d’une activité Hive dans Azure Data Factory 
> [!div class="op_single_selector" title1="Activités de transformation"]
> * [Activité Hive](data-factory-hive-activity.md) 
> * [Activité pig](data-factory-pig-activity.md)
> * [Activité MapReduce](data-factory-map-reduce.md)
> * [Activité de diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Activité Spark](data-factory-spark.md)
> * [Activité Exécution par lots Azure Machine Learning studio (classique)](data-factory-azure-ml-batch-execution-activity.md)
> * [Activité Mettre à jour une ressource Azure Machine Learning studio (classique)](data-factory-azure-ml-update-resource-activity.md)
> * [Activité de procédure stockée](data-factory-stored-proc-activity.md)
> * [Activité U-SQL Data Lake Analytics](data-factory-usql-activity.md)
> * [Activité personnalisée .NET](data-factory-use-custom-activities.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Transformer des données à l’aide d’une activité Hive dans Data Factory](../transform-data-using-hadoop-hive.md).

L’activité Hive HDInsight d’un [pipeline](data-factory-create-pipelines.md) Data Factory exécute des requêtes Hive sur [votre propre](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) cluster ou cluster [à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) HDInsight sous Windows ou Linux. Cet article s'appuie sur l'article [Activités de transformation des données](data-factory-data-transformation-activities.md) qui présente une vue d'ensemble de la transformation des données et les activités de transformation prises en charge.

> [!NOTE] 
> Si vous découvrez Azure Data Factory, lisez la [Présentation d’Azure Data Factory](data-factory-introduction.md) et suivez le didacticiel : [Générer votre premier pipeline de données](data-factory-build-your-first-pipeline.md) avant de lire cet article. 

## <a name="syntax"></a>Syntaxe

```JSON
{
    "name": "Hive Activity",
    "description": "description",
    "type": "HDInsightHive",
    "inputs": [
      {
        "name": "input tables"
      }
    ],
    "outputs": [
      {
        "name": "output tables"
      }
    ],
    "linkedServiceName": "MyHDInsightLinkedService",
    "typeProperties": {
      "script": "Hive script",
      "scriptPath": "<pathtotheHivescriptfileinAzureblobstorage>",
      "defines": {
        "param1": "param1Value"
      }
    },
   "scheduler": {
      "frequency": "Day",
      "interval": 1
    }
}
```
## <a name="syntax-details"></a>Détails de la syntaxe
| Propriété | Description | Obligatoire |
| --- | --- | --- |
| name |Nom de l’activité |Oui |
| description |Texte décrivant la raison motivant l’activité. |Non |
| type |HDinsightHive |Oui |
| inputs |Entrées utilisées par l’activité Hive |Non |
| outputs |Sorties produites par l’activité Hive |Oui |
| linkedServiceName |Référence au cluster HDInsight enregistré comme un service lié dans Data Factory |Oui |
| script |Spécifier le script en ligne Hive |Non |
| scriptPath |Stockez le script Hive dans un stockage d'objets blob Azure et indiquez le chemin d'accès au fichier. Utilisez la propriété ’script’ ou ’scriptPath’. Les deux propriétés ne peuvent pas être utilisées simultanément. Le nom de fichier respecte la casse. |Non |
| defines |Spécifier les paramètres sous forme de paires clé/valeur pour le référencement au sein du script Hive à l'aide de ’hiveconf’ |Non |

## <a name="example"></a>Exemple
Prenons un exemple d’analyse de journaux d’activité de jeux où vous souhaitez identifier le temps passé par les utilisateurs à jouer à des jeux créés par votre entreprise. 

Le journal suivant est un exemple de journal de jeu séparé par des virgules (`,`) et contenant les champs suivants : ProfileID, SessionStart, Duration, SrcIPAddress et GameType.

```
1809,2014-05-04 12:04:25.3470000,14,221.117.223.75,CaptureFlag
1703,2014-05-04 06:05:06.0090000,16,12.49.178.247,KingHill
1703,2014-05-04 10:21:57.3290000,10,199.118.18.179,CaptureFlag
1809,2014-05-04 05:24:22.2100000,23,192.84.66.141,KingHill
.....
```

**Script Hive** pour traiter ces données :

```
DROP TABLE IF EXISTS HiveSampleIn; 
CREATE EXTERNAL TABLE HiveSampleIn 
(
    ProfileID        string, 
    SessionStart     string, 
    Duration         int, 
    SrcIPAddress     string, 
    GameType         string
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION 'wasb://adfwalkthrough@<storageaccount>.blob.core.windows.net/samplein/'; 

DROP TABLE IF EXISTS HiveSampleOut; 
CREATE EXTERNAL TABLE HiveSampleOut 
(    
    ProfileID     string, 
    Duration     int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION 'wasb://adfwalkthrough@<storageaccount>.blob.core.windows.net/sampleout/';

INSERT OVERWRITE TABLE HiveSampleOut
Select 
    ProfileID,
    SUM(Duration)
FROM HiveSampleIn Group by ProfileID
```

Pour exécuter ce script Hive dans un pipeline Data Factory, vous devez effectuer les opérations suivantes :

1. Créez un service lié pour inscrire [votre propre cluster de calcul HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) ou configurer un [cluster de calcul HDInsight à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Appelons ce service lié « HDInsightLinkedService ».
2. Créez un [service lié](data-factory-azure-blob-connector.md) pour configurer la connexion au stockage d'objets blob Azure qui héberge les données. Appelons ce service lié « StorageLinkedService ».
3. Créez des [jeux de données](data-factory-create-datasets.md) pointant vers les données d'entrée et de sortie. Appelons le jeu de données d'entrée « HiveSampleIn » et le jeu de données de sortie « HiveSampleOut »
4. Copiez la requête Hive en tant que fichier dans le stockage d’objets blob Azure configuré à l’étape 2 ci-dessus. Si le stockage pour l’hébergement des données est différent de celui qui héberge ce fichier de requête, créez un service lié de stockage Azure distinct et référencez-le dans l’activité. Utilisez **scriptPath** pour spécifier le chemin d’accès au fichier de requête Hive et **scriptLinkedService** pour spécifier le stockage Azure qui contient le fichier de script. 
   
   > [!NOTE]
   > Vous pouvez également fournir le script en ligne Hive dans la définition d’activité à l’aide de la propriété **script** . Nous ne le recommandons pas, car tous les caractères spéciaux du script dans le document JSON doivent être placés dans une séquence d’échappement, ce qui risque d’entraîner des problèmes de débogage. La meilleure pratique consiste à suivre l’étape 4.
   > 
   > 
5. Créez un pipeline avec l’activité HDInsightHive. L’activité traite/transforme les données.

  ```json
  {
    "name": "HiveActivitySamplePipeline",
       "properties": {
    "activities": [
      {
        "name": "HiveActivitySample",
        "type": "HDInsightHive",
        "inputs": [
        {
          "name": "HiveSampleIn"
        }
        ],
             "outputs": [
               {
                "name": "HiveSampleOut"
               }
             ],
             "linkedServiceName": "HDInsightLinkedService",
             "typeproperties": {
                 "scriptPath": "adfwalkthrough\\scripts\\samplehive.hql",
                 "scriptLinkedService": "StorageLinkedService"
             },
              "scheduler": {
          "frequency": "Hour",
                   "interval": 1
             }
           }
      ]
    }
  }
  ```

6. Déployez le pipeline. Consultez l’article [Création de pipelines](data-factory-create-pipelines.md) pour plus de détails. 
7. Surveillez le pipeline à l'aide des vues de gestion et de surveillance Data Factory. Consultez l’article [Surveillance et gestion des pipelines Data Factory](data-factory-monitor-manage-pipelines.md) pour plus d'informations. 

## <a name="specifying-parameters-for-a-hive-script"></a>Spécification des paramètres d’un script Hive
Dans cet exemple, les journaux d’activité de jeux sont reçus quotidiennement dans le stockage blob Azure et sont conservés dans un dossier partitionné par date et par heure. Vous souhaitez paramétrer le script Hive et fournir dynamiquement l'emplacement du dossier d'entrée pendant l'exécution mais aussi produire la sortie partitionnée par date et par heure.

Pour utiliser le script Hive paramétré, procédez comme suit :

* Définissez les paramètres dans **defines**.

  ```JSON  
    {
        "name": "HiveActivitySamplePipeline",
          "properties": {
        "activities": [
             {
                "name": "HiveActivitySample",
                "type": "HDInsightHive",
                "inputs": [
                      {
                        "name": "HiveSampleIn"
                      }
                ],
                "outputs": [
                      {
                        "name": "HiveSampleOut"
                    }
                ],
                "linkedServiceName": "HDInsightLinkedService",
                "typeproperties": {
                      "scriptPath": "adfwalkthrough\\scripts\\samplehive.hql",
                      "scriptLinkedService": "StorageLinkedService",
                      "defines": {
                        "Input": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)",
                        "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
                      },
                       "scheduler": {
                          "frequency": "Hour",
                          "interval": 1
                    }
                }
              }
        ]
      }
    }
    ```
* Dans le script Hive, reportez-vous au paramètre **${hiveconf:parameterName}** . 
  
    ```
    DROP TABLE IF EXISTS HiveSampleIn; 
    CREATE EXTERNAL TABLE HiveSampleIn 
    (
        ProfileID     string, 
        SessionStart     string, 
        Duration     int, 
        SrcIPAddress     string, 
        GameType     string
    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:Input}'; 

    DROP TABLE IF EXISTS HiveSampleOut; 
    CREATE EXTERNAL TABLE HiveSampleOut 
    (
        ProfileID     string, 
        Duration     int
    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:Output}';

    INSERT OVERWRITE TABLE HiveSampleOut
    Select 
        ProfileID,
        SUM(Duration)
    FROM HiveSampleIn Group by ProfileID
    ```
  ## <a name="see-also"></a>Voir aussi
* [Activité pig](data-factory-pig-activity.md)
* [Activité MapReduce](data-factory-map-reduce.md)
* [Activité de diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md)
* [Appeler des programmes Spark](data-factory-spark.md)
* [Appeler des scripts R](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample)

