---
title: Transformer des données à l’aide d’une activité Pig dans Azure Data Factory
description: Découvrez comment vous pouvez utiliser l’activité Pig dans Azure Data Factory v1 pour exécuter des scripts Pig sur un cluster HDInsight à la demande ou votre propre cluster HDInsight.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.assetid: 5af07a1a-2087-455e-a67b-a79841b4ada5
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 0ad84a0e848abda1b786958947b4081b11b139a7
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495343"
---
# <a name="transform-data-using-pig-activity-in-azure-data-factory"></a>Transformer des données à l’aide d’une activité Pig dans Azure Data Factory
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
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Transformer des données à l’aide d’une activité Pig dans Data Factory](../transform-data-using-hadoop-pig.md).


L’activité Pig de HDInsight d’un [pipeline](data-factory-create-pipelines.md) Data Factory exécute des requêtes Pig sur [votre propre](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) cluster ou le cluster [à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) HDInsight sous Windows ou Linux. Cet article s'appuie sur l'article [Activités de transformation des données](data-factory-data-transformation-activities.md) qui présente une vue d'ensemble de la transformation des données et les activités de transformation prises en charge.

> [!NOTE] 
> Si vous découvrez Azure Data Factory, lisez la [Présentation d’Azure Data Factory](data-factory-introduction.md) et suivez le didacticiel : [Générer votre premier pipeline de données](data-factory-build-your-first-pipeline.md) avant de lire cet article. 

## <a name="syntax"></a>Syntaxe

```JSON
{
  "name": "HiveActivitySamplePipeline",
  "properties": {
    "activities": [
      {
        "name": "Pig Activity",
        "description": "description",
        "type": "HDInsightPig",
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
          "script": "Pig script",
          "scriptPath": "<pathtothePigscriptfileinAzureblobstorage>",
          "defines": {
            "param1": "param1Value"
          }
        },
        "scheduler": {
          "frequency": "Day",
          "interval": 1
        }
      }
    ]
  }
}
```

## <a name="syntax-details"></a>Détails de la syntaxe

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| name |Nom de l’activité |Oui |
| description |Texte décrivant la raison motivant l’activité. |Non |
| type |HDinsightPig |Oui |
| inputs |Une ou plusieurs entrées utilisées par l'activité pig |Non |
| outputs |Une ou plusieurs sorties produites par l’activité pig |Oui |
| linkedServiceName |Référence au cluster HDInsight enregistré comme un service lié dans Data Factory |Oui |
| script |Spécifier le script en ligne pig |Non |
| scriptPath |Stockez le script pig dans un stockage d'objets blob Azure et indiquez le chemin d'accès au fichier. Utilisez la propriété ’script’ ou ’scriptPath’. Les deux propriétés ne peuvent pas être utilisées simultanément. Le nom de fichier respecte la casse. |Non |
| defines |Spécifier les paramètres sous forme de paires clé/valeur pour le référencement au sein du script pig |Non |

## <a name="example"></a>Exemple
Prenons un exemple d’analyse de journaux d’activité de jeux où vous souhaitez identifier le temps passé par les joueurs à jouer à des jeux créés par votre entreprise.

L’exemple de journal de jeu suivant est un fichier séparé par des virgules (,). Il contient les champs suivants : ProfileID, SessionStart, Duration, SrcIPAddress et GameType.

```
1809,2014-05-04 12:04:25.3470000,14,221.117.223.75,CaptureFlag
1703,2014-05-04 06:05:06.0090000,16,12.49.178.247,KingHill
1703,2014-05-04 10:21:57.3290000,10,199.118.18.179,CaptureFlag
1809,2014-05-04 05:24:22.2100000,23,192.84.66.141,KingHill
.....
```

Le **script pig** pour traiter ces données :

```
PigSampleIn = LOAD 'wasb://adfwalkthrough@anandsub14.blob.core.windows.net/samplein/' USING PigStorage(',') AS (ProfileID:chararray, SessionStart:chararray, Duration:int, SrcIPAddress:chararray, GameType:chararray);

GroupProfile = Group PigSampleIn all;

PigSampleOut = Foreach GroupProfile Generate PigSampleIn.ProfileID, SUM(PigSampleIn.Duration);

Store PigSampleOut into 'wasb://adfwalkthrough@anandsub14.blob.core.windows.net/sampleoutpig/' USING PigStorage (',');
```

Pour exécuter ce script pig dans un pipeline Data Factory, appliquez les étapes suivantes :

1. Créez un service lié pour inscrire [votre propre cluster de calcul HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) ou configurer un [cluster de calcul HDInsight à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Appelons ce service lié **HDInsightLinkedService**.
2. Créez un [service lié](data-factory-azure-blob-connector.md) pour configurer la connexion au stockage d'objets blob Azure qui héberge les données. Appelons ce service lié **StorageLinkedService**.
3. Créez des [jeux de données](data-factory-create-datasets.md) pointant vers les données d'entrée et de sortie. Appelons le jeu de données d’entrée **PigSampleIn** et le jeu de données de sortie **PigSampleOut**.
4. Copiez la requête Pig dans le fichier configuré par le stockage d’objets Blob Azure à l’étape #2. Si le stockage Azure qui héberge les données est différent de celui qui héberge le fichier de requête, créez un service de stockage Azure lié distinct. Consultez le service lié dans la configuration de l’activité. Utilisez **scriptPath** pour spécifier le chemin d’accès au fichier de script pig et **scriptLinkedService**. 
   
   > [!NOTE]
   > Vous pouvez également fournir le script en ligne pig dans la définition d’activité à l’aide de la propriété **script** . Cependant, cela n’est pas recommandé car tous les caractères spéciaux du script au sein du document JSON doivent être placés dans une séquence d’échappement, ce qui risque d’entraîner des problèmes de débogage. La meilleure pratique consiste à suivre l’étape 4.
   >
   >
5. Créez le pipeline avec l'activité HDInsightPig. Cette activité traite les données d’entrée en exécutant le script Pig sur un cluster HDInsight.

    ```JSON
    {
      "name": "PigActivitySamplePipeline",
      "properties": {
        "activities": [
          {
            "name": "PigActivitySample",
            "type": "HDInsightPig",
            "inputs": [
              {
                "name": "PigSampleIn"
              }
            ],
            "outputs": [
              {
                "name": "PigSampleOut"
              }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
              "scriptPath": "adfwalkthrough\\scripts\\enrichlogs.pig",
              "scriptLinkedService": "StorageLinkedService"
            },
            "scheduler": {
              "frequency": "Day",
              "interval": 1
            }
          }
        ]
      }
    }
    ```
6. Déployez le pipeline. Consultez l’article [Création de pipelines](data-factory-create-pipelines.md) pour plus de détails. 
7. Surveillez le pipeline à l'aide des vues de gestion et de surveillance Data Factory. Consultez l’article [Surveillance et gestion des pipelines Data Factory](data-factory-monitor-manage-pipelines.md) pour plus d'informations.

## <a name="specifying-parameters-for-a-pig-script"></a>Spécification des paramètres d’un script Pig
Prenons l’exemple suivant : des journaux d’activité de jeux sont reçus quotidiennement dans le stockage blob Azure et conservés dans un dossier partitionné selon la date et l’heure. Vous souhaitez paramétrer le script pig et fournir dynamiquement l'emplacement du dossier d'entrée pendant l'exécution mais aussi produire la sortie partitionnée par date et par heure.

Pour utiliser le script pig paramétré, procédez comme suit :

* Définissez les paramètres dans **defines**.

    ```JSON
    {
      "name": "PigActivitySamplePipeline",
      "properties": {
        "activities": [
          {
            "name": "PigActivitySample",
            "type": "HDInsightPig",
            "inputs": [
              {
                "name": "PigSampleIn"
              }
            ],
            "outputs": [
              {
                "name": "PigSampleOut"
              }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
              "scriptPath": "adfwalkthrough\\scripts\\samplepig.hql",
              "scriptLinkedService": "StorageLinkedService",
              "defines": {
                "Input": "$$Text.Format('wasb: //adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0: yyyy}/monthno={0:MM}/dayno={0: dd}/',SliceStart)",
                "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
              }
            },
            "scheduler": {
              "frequency": "Day",
              "interval": 1
            }
          }
        ]
      }
    }
    ```
* Dans le script pig, reportez-vous aux paramètres à l'aide de ’ **$parameterName**’ comme indiqué dans l'exemple suivant :

    ```
    PigSampleIn = LOAD '$Input' USING PigStorage(',') AS (ProfileID:chararray, SessionStart:chararray, Duration:int, SrcIPAddress:chararray, GameType:chararray);
    GroupProfile = Group PigSampleIn all;
    PigSampleOut = Foreach GroupProfile Generate PigSampleIn.ProfileID, SUM(PigSampleIn.Duration);
    Store PigSampleOut into '$Output' USING PigStorage (','); 
    ```

## <a name="see-also"></a>Voir aussi
* [Activité Hive](data-factory-hive-activity.md)
* [Activité MapReduce](data-factory-map-reduce.md)
* [Activité de diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md)
* [Appeler des programmes Spark](data-factory-spark.md)
* [Appeler des scripts R](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample)
