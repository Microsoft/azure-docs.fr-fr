---
title: Déplacer des données à partir d’Amazon Redshift avec Azure Data Factory
description: Découvrez comment déplacer des données à partir d’Amazon Redshift à l’aide de l’activité de copie d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 01d15078-58dc-455c-9d9d-98fbdf4ea51e
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: c0dcaec9c8e9a310af1fd6fc319e0784694610e2
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463081"
---
# <a name="move-data-from-amazon-redshift-using-azure-data-factory"></a>Déplacer des données depuis Amazon Redshift à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-amazon-redshift-connector.md)
> * [Version 2 (version actuelle)](../connector-amazon-redshift.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [Connecteur Amazon Redshift dans V2](../connector-amazon-redshift.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données à partir de Amazon Redshift. L’article s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

Actuellement, Data Factory prend uniquement en charge le déplacement de données d’Amazon Redshift vers une [banque de données réceptrice prise en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), et non le déplacement de données d’autres banques de données vers Amazon Redshift.

> [!TIP]
> Pour obtenir de meilleures performances lors de la copie de grandes quantités de données d’Amazon Redshift, utilisez le mécanisme Redshift intégré **UNLOAD** via Amazon Simple Storage Service (Amazon S3). Pour plus d’informations, consultez la section [Utiliser UNLOAD pour copier des données à partir d’Amazon Redshift](#use-unload-to-copy-data-from-amazon-redshift).

## <a name="prerequisites"></a>Prérequis
* Si vous déplacez des données vers un magasin de données local, vous devez installer la [passerelle de gestion des données](data-factory-data-management-gateway.md) sur une machine locale. Accordez l’accès d’une passerelle au cluster Amazon Redshift à l’aide de l’adresse IP de l’ordinateur local. Pour obtenir des instructions, consultez la rubrique relative à l’[autorisation d’accès au cluster](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html).
* Pour déplacer des données vers une banque de données Azure, procédez de la manière décrite dans [calcul de l’adresse IP et des plages SQL utilisés par les centres de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653).

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie afin de déplacer les données d’une source Amazon Redshift à l’aide de différents outils et API.

Le moyen le plus simple de créer un pipeline consiste à utiliser l’Assistant de copie Azure Data Factory. Pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant de copie, consultez la page [Didacticiel : Créer un pipeline à l’aide de l’Assistant de copie](data-factory-copy-data-wizard-tutorial.md).

Vous pouvez également créer un pipeline à l’aide de Visual Studio, d’Azure PowerShell ou d’autres outils. Les modèles Azure Resource Manager, l’API .NET ou l’API REST peuvent également être utilisés pour créer le pipeline. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de services liés pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de jeux de données pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un pipeline avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant copie, des définitions JSON sont automatiquement créées pour ces entités Data Factory. Lorsque vous utilisez des outils ou API (à l’exception de l’API .NET), vous devez définir les entités Data Factory à l’aide du format JSON. L’exemple JSON : copier des données d’Amazon Redshift vers Stockage Blob Azure montre des définitions JSON pour les entités Data Factory utilisées pour copier des données d’une banque de données Amazon Redshift.

Les sections suivantes décrivent les propriétés JSON utilisées pour définir les entités Data Factory pour Amazon Redshift.

## <a name="linked-service-properties"></a>Propriétés du service lié

Le tableau suivant décrit les éléments JSON spécifiques du service lié Amazon Redshift.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **type** |Cette propriété doit être définie sur **AmazonRedshift**. |Oui |
| **server** |Nom d’hôte ou adresse IP du serveur Amazon Redshift. |Oui |
| **port** |Le numéro du port TCP utilisé par le serveur Amazon Redshift pour écouter les connexions clientes. |Non (valeur par défaut : 5439) |
| **database** |Nom de la base de données Amazon Redshift. |Oui |
| **username** |Nom de l’utilisateur ayant accès à la base de données. |Oui |
| **mot de passe** |Mot de passe du compte d’utilisateur. |Oui |

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections **Structure**, **Disponibilité** et **Stratégie** sont identiques pour tous les types de jeux de données. Les types de jeux de données sont, par exemple, Azure SQL, Stockage Blob Azure et Stockage Table Azure.

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement des données dans la banque. La section **typeProperties** pour le jeu de données de type **RelationalTable**, qui inclut le jeu de données Amazon Redshift, a les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **tableName** |Nom de la table dans la base de données Amazon Redshift à laquelle le service lié fait référence. |Non (si la propriété **query** d’une activité de copie de type **RelationalSource** est spécifiée) |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés **Nom**, **Description**, Table d’**entrée**, Table de **sortie** et **Stratégie** sont disponibles pour tous les types d’activités. Les propriétés disponibles dans la section **typeProperties** varient pour chaque type d’activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Pour une activité de copie, quand une source est de type **AmazonRedshiftSource**, les propriétés suivantes sont disponibles dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **query** | Utilise la requête personnalisée pour lire les données. |Non (si la propriété **tableName** d’un jeu de données est spécifié) |
| **redshiftUnloadSettings** | Contient le groupe de propriétés lors de l’utilisation de la commande Redshift **UNLOAD**. | Non |
| **s3LinkedServiceName** | Amazon S3 à utiliser en tant que magasin temporaire. Le service lié est spécifié au moyen d’un nom Azure Data Factory de type **AwsAccessKey**. | Obligatoire lorsque vous utilisez la propriété **redshiftUnloadSettings** |
| **bucketName** | Indique le compartiment Amazon S3 à utiliser pour stocker les données intermédiaires. Si cette propriété n’est pas fournie, l’activité de copie génère automatiquement un compartiment. | Obligatoire lorsque vous utilisez la propriété **redshiftUnloadSettings** |

Vous pouvez également utiliser le type **RelationalSource**, qui inclut Amazon Redshift, avec la propriété suivante dans la section **typeProperties**. Notez que ce type de source ne prend pas en charge la commande **UNLOAD** de Redshift.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **query** |Utilise la requête personnalisée pour lire les données. | Non (si la propriété **tableName** d’un jeu de données est spécifié) |

## <a name="use-unload-to-copy-data-from-amazon-redshift"></a>Utiliser UNLOAD pour copier des données à partir d’Amazon Redshift

La commande [**UNLOAD**](https://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html) d’Amazon Redshift décharge les résultats d’une requête dans un ou plusieurs fichiers sur Amazon S3. Cette commande est recommandée par Amazon pour la copie de jeux de données volumineux à partir de Redshift.

**Exemple : Copier des données à partir d’Amazon Redshift vers Azure Synapse Analytics**

Cet exemple copie des données d’Amazon Redshift vers Azure Synapse Analytics. L’exemple utilise la commande Redshift **UNLOAD**, les données de copie intermédiaire et Microsoft PolyBase.

Pour cet exemple de cas d’usage, l’activité de copie décharge tout d’abord les données d’Amazon Redshift vers Amazon S3 selon la configuration de l’option **redshiftUnloadSettings**. Les données sont ensuite copiées d’Amazon S3 vers le stockage Blob Azure, comme indiqué dans l’option **stagingSettings**. Enfin, PolyBase charge les données dans Azure Synapse Analytics. Tous les formats intermédiaires sont gérés par l’activité de copie.

![Copier le workflow à partir d’Amazon Redshift vers Azure Synapse Analytics](media/data-factory-amazon-redshift-connector/redshift-to-sql-dw-copy-workflow.png)

```json
{
    "name": "CopyFromRedshiftToSQLDW",
    "type": "Copy",
    "typeProperties": {
        "source": {
            "type": "AmazonRedshiftSource",
            "query": "select * from MyTable",
            "redshiftUnloadSettings": {
                "s3LinkedServiceName":"MyAmazonS3StorageLinkedService",
                "bucketName": "bucketForUnload"
            }
        },
        "sink": {
            "type": "SqlDWSink",
            "allowPolyBase": true
        },
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": "MyAzureStorageLinkedService",
            "path": "adfstagingcopydata"
        },
        "cloudDataMovementUnits": 32
        .....
    }
}
```

## <a name="json-example-copy-data-from-amazon-redshift-to-azure-blob-storage"></a>Exemple JSON : copie de données à partir d’Amazon Redshift vers le stockage d’objets blob Azure
Cet exemple indique comment copier des données à partir d’une base de données Amazon Redshift locale vers un système de Stockage Blob Azure. Les données peuvent être copiées directement vers n’importe quel [récepteur pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) à l’aide de l’activité de copie.

L’exemple contient les entités de fabrique de données suivantes :

* Un service lié de type [AmazonRedshift](#linked-service-properties)
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [RelationalTable](#dataset-properties)
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties)
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise les propriétés [RelationalSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)

L’exemple copie toutes les heures les données de résultat d’une requête d’Amazon Redshift vers un stockage Blob Azure. Les propriétés JSON utilisées dans l’exemple sont décrites dans les sections suivant les définitions des entités.

**Service lié Amazon Redshift**

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties":
    {
        "type": "AmazonRedshift",
        "typeProperties":
        {
            "server": "< The IP address or host name of the Amazon Redshift server >",
            "port": "<The number of the TCP port that the Amazon Redshift server uses to listen for client connections.>",
            "database": "<The database name of the Amazon Redshift database>",
            "username": "<username>",
            "password": "<password>"
        }
    }
}
```

**Service lié Azure Blob Storage**

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
**Jeu de données d’entrée Amazon Redshift**

La propriété **external** est définie sur « true » pour informer le service Data Factory que le jeu de données est externe à la fabrique de données. Ce paramètre de propriété indique que le jeu de données n’est pas produit par une activité de la fabrique de données. Définissez la propriété sur true sur un jeu de données d’entrée qui n’est pas produit par une activité dans le pipeline.

```json
{
    "name": "AmazonRedshiftInputDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "AmazonRedshiftLinkedService",
        "typeProperties": {
            "tableName": "<Table name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

**Jeu de données de sortie d'objet Blob Azure**

Les données sont écrites dans un nouvel objet blob toutes les heures en configurant la propriété **frequency** sur « Hour » et la propriété **interval** sur 1. La propriété **folderPath** du blob est évaluée dynamiquement. La valeur de propriété est fondée sur l’heure de début de la tranche en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et la partie heure de l’heure de début.

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/fromamazonredshift/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Activité de copie dans un pipeline, avec une source Azure Redshift (de type RelationalSource) et un récepteur blob**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie. Le pipeline est planifié pour s’exécuter toutes les heures. Dans la définition JSON du pipeline, le type **source** est défini sur **RelationalSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **query** sélectionne les données à copier de la dernière heure.

```json
{
    "name": "CopyAmazonRedshiftToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "AmazonRedshiftSource",
                        "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)",
                        "redshiftUnloadSettings": {
                            "s3LinkedServiceName":"myS3Storage",
                            "bucketName": "bucketForUnload"
                        }
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    },
                    "cloudDataMovementUnits": 32
                },
                "inputs": [
                    {
                        "name": "AmazonRedshiftInputDataset"
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
                "name": "AmazonRedshiftToBlob"
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```
### <a name="type-mapping-for-amazon-redshift"></a>Mappage de type pour Amazon Redshift
Comme indiqué dans l’article consacré aux [activités de déplacement des données](data-factory-data-movement-activities.md), l’activité de copie convertit automatiquement un type source en type récepteur. Les types sont convertis à l’aide d’une approche en deux étapes :

1. Conversion d’un type natif source en type .NET
2. Conversion d’un type .NET en type récepteur natif

Les mappages suivants sont utilisés lorsque l’activité de copie convertit les données de type Amazon Redshift en type .NET :

| Type Amazon Redshift | Type .NET |
| --- | --- |
| SMALLINT |Int16 |
| INTEGER |Int32 |
| bigint |Int64 |
| DECIMAL |Decimal |
| real |Unique |
| DOUBLE PRECISION |Double |
| BOOLEAN |String |
| CHAR |String |
| VARCHAR |String |
| DATE |DateTime |
| timestamp |DateTime |
| TEXT |String |

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur
Pour savoir comment mapper des colonnes du jeu de données source à des colonnes du jeu de données récepteur, consultez [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-reads-from-relational-sources"></a>Lectures renouvelées de sources relationnelles
Lorsque vous copiez des données à partir d’un magasin de données relationnel, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer la **stratégie** de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Assurez-vous que les mêmes données sont lues, quel que soit le nombre d’exécutions de la tranche. Assurez-vous également qu’elles sont lues quelle que soit la façon dont vous réexécutez la tranche. Pour plus d’informations, consultez [Lectures renouvelées de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Performances et réglage
Pour en savoir plus sur les facteurs clés affectant les performances de l’activité de copie et les différentes manières de les optimiser, consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md).

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
