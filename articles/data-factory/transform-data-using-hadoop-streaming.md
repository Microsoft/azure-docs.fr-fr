---
title: Transformer des données à l’aide d’une activité de streaming Hadoop
description: Explique comment utiliser l’activité de diffusion en continu Hadoop dans Azure Data Factory pour transformer des données en exécutant des programmes de diffusion en continu Hadoop sur un cluster Hadoop.
author: nabhishek
ms.author: abnarain
manager: shwang
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/08/2020
ms.openlocfilehash: 85dd75b2af5d14d835db8aacc415069a2d67298e
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92631833"
---
# <a name="transform-data-using-hadoop-streaming-activity-in-azure-data-factory"></a>Transformer des données à l’aide d’une activité de diffusion en continu Hadoop dans Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-hadoop-streaming-activity.md)
> * [Version actuelle](transform-data-using-hadoop-streaming.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’activité de diffusion en continu HDInsight dans un [pipeline](concepts-pipelines-activities.md) Data Factory exécute des programmes de diffusion en continu Hadoop sur votre cluster HDInsight [propre](compute-linked-services.md#azure-hdinsight-linked-service) ou [à la demande](compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Cet article s'appuie sur l'article [Activités de transformation des données](transform-data.md) qui présente une vue d'ensemble de la transformation des données et les activités de transformation prises en charge.

Si vous découvrez Azure Data Factory, lisez la [présentation d’Azure Data Factory](introduction.md) et suivez le [Didacticiel : Transformer des données](tutorial-transform-data-spark-powershell.md) avant de lire cet article. 

## <a name="json-sample"></a>Exemple JSON
```json
{
    "name": "Streaming Activity",
    "description": "Description",
    "type": "HDInsightStreaming",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "mapper": "MyMapper.exe",
        "reducer": "MyReducer.exe",
        "combiner": "MyCombiner.exe",
        "fileLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "filePaths": [
            "<containername>/example/apps/MyMapper.exe",
            "<containername>/example/apps/MyReducer.exe",
            "<containername>/example/apps/MyCombiner.exe"
        ],
        "input": "wasb://<containername>@<accountname>.blob.core.windows.net/example/input/MapperInput.txt",
        "output": "wasb://<containername>@<accountname>.blob.core.windows.net/example/output/ReducerOutput.txt",
        "commandEnvironment": [
            "CmdEnvVarName=CmdEnvVarValue"
        ],
        "getDebugInfo": "Failure",
        "arguments": [
            "SampleHadoopJobArgument1"
        ],
        "defines": {
            "param1": "param1Value"
        }
    }
}
```

## <a name="syntax-details"></a>Détails de la syntaxe

| Propriété          | Description                              | Obligatoire |
| ----------------- | ---------------------------------------- | -------- |
| name              | Nom de l’activité                     | Oui      |
| description       | Texte décrivant la raison motivant l’activité. | Non       |
| type              | Pour l’activité de diffusion en continu Hadoop, le type d’activité est HDInsightStreaming. | Oui      |
| linkedServiceName | Référence au cluster HDInsight enregistré en tant que service lié dans Data Factory. Pour en savoir plus sur ce service lié, consultez l’article [Services liés de calcul](compute-linked-services.md). | Oui      |
| mappeur            | Spécifie le nom de l’exécutable du mappeur. | Oui      |
| raccord de réduction           | Spécifie le nom de l’exécutable du raccord de réduction. | Oui      |
| combinateur          | Spécifie le nom de l’exécutable du combinateur. | Non       |
| fileLinkedService | Référence à un service lié de stockage Azure utilisée pour stocker les programmes du mappeur, du combinateur et du raccord de réduction à exécuter. Seuls les services liés **[Stockage Blob Azure](./connector-azure-blob-storage.md)** et **[ADLS Gen2](./connector-azure-data-lake-storage.md)** sont pris en charge ici. Si vous ne spécifiez pas ce service lié, le service lié Stockage Azure défini dans le service lié HDInsight est utilisé. | Non       |
| filePath          | Fournissez un tableau du chemin vers les programmes du mappeur, du combinateur et du raccord de réduction stockés dans le stockage Azure référencé par fileLinkedService. Le chemin d'accès respecte la casse. | Oui      |
| entrée             | Spécifie le chemin WASB vers le fichier d’entrée du mappeur. | Oui      |
| sortie            | Spécifie le chemin WASB vers le fichier de sortie du raccord de réduction. | Oui      |
| getDebugInfo      | Spécifie quand les fichiers journaux sont copiés vers le stockage Azure utilisé par le cluster HDInsight (ou) spécifié par scriptLinkedService. Valeurs autorisées : None, Always ou Failure. Valeur par défaut : Aucun. | Non       |
| arguments         | Spécifie un tableau d’arguments pour un travail Hadoop. Les arguments sont passés sous la forme d’arguments de ligne de commande à chaque tâche. | Non       |
| defines           | Spécifier les paramètres sous forme de paires clé/valeur pour le référencement au sein du script Hive. | Non       | 

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants qui expliquent comment transformer des données par d’autres moyens : 

* [Activité U-SQL](transform-data-using-data-lake-analytics.md)
* [Activité Hive](transform-data-using-hadoop-hive.md)
* [Activité pig](transform-data-using-hadoop-pig.md)
* [Activité MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Activité Spark](transform-data-using-spark.md)
* [Activité personnalisée .NET](transform-data-using-dotnet-custom-activity.md)
* [Activité Exécution par lots Azure Machine Learning studio (classique)](transform-data-using-machine-learning.md)
* [Activité de procédure stockée](transform-data-using-stored-procedure.md)