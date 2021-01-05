---
title: Déplacement de données à partir d’Amazon Simple Storage Service à l’aide d’Azure Data Factory
description: Découvrez comment déplacer des données à partir d’Amazon Simple Storage Service (S3) à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 636d3179-eba8-4841-bcb4-3563f6822a26
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: b23be9901df7ca435f412d9f49e1a7ad88382ade
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84711916"
---
# <a name="move-data-from-amazon-simple-storage-service-by-using-azure-data-factory"></a>Déplacement de données à partir d’Amazon Simple Storage Service à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-amazon-simple-storage-service-connector.md)
> * [Version 2 (version actuelle)](../connector-amazon-simple-storage-service.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur Amazon S3 dans V2](../connector-amazon-simple-storage-service.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données à partir d’un système Amazon Simple Storage Service (S3). Il s’appuie sur l’article relatif aux [activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement des données avec l’activité de copie.

Vous pouvez copier les données d’Amazon S3 dans tout magasin de données récepteur pris en charge. Consultez le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) pour obtenir la liste des magasins de données pris en charge en tant que récepteurs par l’activité de copie. Actuellement, Data Factory prend uniquement en charge le déplacement de données d’Amazon S3 vers d’autres magasins de données, mais pas l’inverse.

## <a name="required-permissions"></a>Autorisations requises
Pour copier des données à partir d’Amazon S3, assurez-vous que vous disposez des autorisations suivantes :

* `s3:GetObject` et `s3:GetObjectVersion` pour les opérations d’objet Amazon S3 ;
* `s3:ListBucket` pour les opérations de compartiment Amazon S3. Si vous utilisez l’Assistant de copie de Data Factory, l’élément `s3:ListAllMyBuckets` est également requis.

Pour obtenir la liste complète des autorisations Amazon S3, consultez l’article [Spécification des autorisations d’une stratégie](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html).

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’une source Amazon S3 à l’aide de différents outils ou API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie**. Pour obtenir une procédure pas à pas rapide, consultez [Tutoriel : Créer un pipeline à l’aide de l’Assistant de copie](data-factory-copy-data-wizard-tutorial.md).

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, **modèle Azure Resource Manager**, **.NET API** et **API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez les outils ou API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory à l’aide du format JSON. Pour un exemple avec des définitions JSON pour les entités Data Factory servant à copier des données à partir d’un magasin de données Amazon S3, consultez la section [Exemple JSON : Copier des données depuis Amazon S3 vers Stockage Blob Azure](#json-example-copy-data-from-amazon-s3-to-azure-blob-storage) de cet article.

> [!NOTE]
> Pour plus d’informations sur les formats de fichier et de compression pris en charge pour une activité de copie, consultez [Formats de fichier et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md).

Les sections suivantes fournissent des informations sur les propriétés JSON utilisées pour définir les entités Data Factory spécifiques à Amazon S3.

## <a name="linked-service-properties"></a>Propriétés du service lié
Un service lié lie un magasin de données à une fabrique de données. Pour lier votre magasin de données Amazon S3 à votre fabrique de données, vous devez créer un service lié de type **AwsAccessKey**. La table suivante fournit la description des éléments JSON spécifiques au service lié Amazon S3 (AwsAccessKey).

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| accessKeyID |ID de la clé d’accès secrète. |string |Oui |
| secretAccessKey |La clé d’accès secrète elle-même. |Chaîne secrète chiffrée |Oui |

>[!NOTE]
>Ce connecteur nécessite des clés d’accès pour le compte IAM pour copier des données à partir d’Amazon S3. Les [informations d’identification de sécurité temporaires](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) ne sont pas prises en charge.
>

Voici un exemple :

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour spécifier un jeu de données afin de représenter les données d’entrée dans un Stockage Blob Azure, définissez la propriété de type du jeu de données sur **AmazonS3**. Définissez la propriété **linkedServiceName** du jeu de données sur le nom du service lié Amazon S3. Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, voir [Création de jeux de données](data-factory-create-datasets.md). 

Les sections comme la structure, la disponibilité et la stratégie sont similaires pour tous les types de jeux de données (par exemple, SQL Database, Azure Blob et Azure Table). La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement des données dans le magasin de données. La section **typeProperties** du jeu de données de type **AmazonS3** (comprenant le jeu de données Amazon S3) présente les propriétés suivantes :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| bucketName |Le nom de compartiment S3. |String |Oui |
| key |La clé d’objet S3. |String |Non |
| prefix |Préfixe de la clé d’objet S3. Les objets dont les clés commencent par ce préfixe sont sélectionnés. S’applique uniquement lorsque la clé est vide. |String |Non |
| version |La version de l’objet S3 si le contrôle de version S3 est activé. |String |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, voir les sections [Format Text](data-factory-supported-file-and-compression-formats.md#text-format), [Format JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [Format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. | |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Les types pris en charge sont les suivants : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Les niveaux pris en charge sont : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). | |Non |


> [!NOTE]
> **bucketName + key** spécifient l’emplacement de l’objet S3 où le compartiment est le conteneur racine pour les objets S3 et la clé est le chemin d’accès complet à l’objet S3.

### <a name="sample-dataset-with-prefix"></a>Exemple de jeu de données avec le préfixe

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "prefix": "testFolder/test",
            "bucketName": "testbucket",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
### <a name="sample-dataset-with-version"></a>Exemple de jeu de données (avec la version)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "key": "testFolder/test.orc",
            "bucketName": "testbucket",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

### <a name="dynamic-paths-for-s3"></a>Chemins d’accès dynamiques pour S3
L’exemple précédent utilise des valeurs fixes pour les propriétés **key** et **bucketName** dans le jeu de données Amazon S3.

```json
"key": "testFolder/test.orc",
"bucketName": "testbucket",
```

Vous pouvez demander à Data Factory de calculer ces propriétés dynamiquement au moment de l’exécution à l’aide de variables système telles que SliceStart.

```json
"key": "$$Text.Format('{0:MM}/{0:dd}/test.orc', SliceStart)"
"bucketName": "$$Text.Format('{0:yyyy}', SliceStart)"
```

Vous pouvez faire de même pour la propriété **prefix** d’un jeu de données Amazon S3. Pour obtenir la liste des fonctions et variables prises en charge, consultez [Variables système et fonctions Data Factory](data-factory-functions-variables.md).

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et propriétés disponibles pour la définition des activités, voir [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d'entrée et de sortie et les différentes stratégies sont disponibles pour tous les types d'activités. Les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Pour l’activité de copie, les propriétés dépendent des types de sources et de récepteurs. Lorsque la source de l’activité de copie est de type **FileSystemSource** (qui inclut Amazon S3), la propriété suivante est disponible dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Spécifie s’il faut répertorier de manière récursive les objets S3 sous le répertoire. |true/false |Non |

## <a name="json-example-copy-data-from-amazon-s3-to-azure-blob-storage"></a>Exemple JSON : Copier des données depuis Amazon S3 vers Stockage Blob Azure
Cet exemple indique comment copier des données à partir d’Amazon S3 vers un stockage Blob Azure. Toutefois, les données peuvent être copiées directement vers [l’un des récepteurs pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), via l’activité de copie de Data Factory.

Cet exemple fournit des définitions JSON pour les entités Data Factory suivantes. Vous pouvez utiliser ces définitions pour créer un pipeline afin de copier des données depuis Amazon S3 vers un Stockage Blob Azure à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou de [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md).   

* Un service lié de type [AwsAccessKey](#linked-service-properties).
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [AmazonS3](#dataset-properties).
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [pipeline](data-factory-create-pipelines.md) avec activité de copie qui utilise [FileSystemSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie des données à partir d’Amazon S3 vers un objet blob Azure toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

### <a name="amazon-s3-linked-service"></a>Service lié Amazon S3

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Service lié Stockage Azure

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

### <a name="amazon-s3-input-dataset"></a>Jeu de données d’entrée Amazon S3

La définition de **"external": true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory. Définissez cette propriété sur true sur un jeu de données d’entrée qui n’est pas produit par une activité dans le pipeline.

```json
    {
        "name": "AmazonS3InputDataset",
        "properties": {
            "type": "AmazonS3",
            "linkedServiceName": "AmazonS3LinkedService",
            "typeProperties": {
                "key": "testFolder/test.orc",
                "bucketName": "testbucket",
                "format": {
                    "type": "OrcFormat"
                }
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": true
        }
    }
```


### <a name="azure-blob-output-dataset"></a>Jeu de données de sortie d’objet Blob Azure

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et la partie heure de l’heure de début.

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/fromamazons3/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```


### <a name="copy-activity-in-a-pipeline-with-an-amazon-s3-source-and-a-blob-sink"></a>Activité de copie dans un pipeline avec une source Amazon S3 et un récepteur blob

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie, et qui est planifiée pour s’exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **FileSystemSource** et le type **sink** est défini sur **BlobSink**.

```json
{
    "name": "CopyAmazonS3ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "FileSystemSource",
                        "recursive": true
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "AmazonS3InputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "AmazonS3ToBlob"
            }
        ],
        "start": "2014-08-08T18:00:00Z",
        "end": "2014-08-08T19:00:00Z"
    }
}
```
> [!NOTE]
> Pour savoir comment mapper des colonnes d’un jeu de données source à des colonnes d’un jeu de données récepteur, voir [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

* Pour découvrir les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser, voir [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md).

* Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, voir [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
