---
title: Copier des données vers et depuis Azure Databricks Delta Lake
description: Découvrez comment copier des données depuis et vers Azure Databricks Delta Lake à l’aide d’une activité de copie dans un pipeline Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/24/2020
ms.openlocfilehash: c7e8f96e7917173aaec308b8ae5218684a722483
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507470"
---
# <a name="copy-data-to-and-from-azure-databricks-delta-lake-by-using-azure-data-factory"></a>Copier des données vers et à partir d’Azure Databricks Delta Lake avec Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données depuis/vers Azure Databricks Delta Lake. Il s’appuie sur l’article [Activité de copie dans Azure Data Factory](copy-activity-overview.md), qui constitue une présentation de l’activité de copie.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Azure Databricks Delta Lake est pris en charge pour les activités suivantes :

- [Activité de copie](copy-activity-overview.md) avec une table [Prise en charge de la matrice source/récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

En général, Azure Data Factory prend en charge Delta Lake avec les fonctionnalités suivantes pour répondre à vos différents besoins.

- L’activité de copie prend en charge le connecteur Azure Databricks Delta Lake pour copier des données d’un magasin de données source pris en charge vers une table Azure Databricks Delta Lake, et d’une table Delta Lake vers un magasin de données récepteur pris en charge. Elle tire parti de votre cluster Databricks pour effectuer le déplacement des données. Pour plus d’informations, consultez la [section des prérequis](#prerequisites).
- Le [flux de données de mappage](concepts-data-flow-overview.md) prend en charge le [format Delta](format-delta.md) générique sur Stockage Azure en tant que source et récepteur pour lire et écrire des fichiers Delta pour l’ETL sans code, et s’exécute sur un Azure Integration Runtime géré.
- Les [activités Databricks](transform-data-databricks-notebook.md) prennent en charge l’orchestration de votre charge de travail ETL centrée sur le code ou de Machine Learning sur Delta Lake.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce connecteur Azure Databricks Delta Lake, vous devez configurer un cluster dans Azure Databricks.

- Pour copier des données vers Delta Lake, l’activité de copie appelle le cluster Azure Databricks pour lire des données à partir d’un Stockage Azure, qui est votre source originale ou une zone de transit dans laquelle Data Factory écrit d’abord les données source via la copie intermédiaire intégrée. En savoir plus sur [Delta Lake en tant que source](#delta-lake-as-source).
- De même, pour copier des données depuis Delta Lake, l’activité de copie appelle le cluster Azure Databricks pour écrire des données à partir d’un Stockage Azure, qui est soit votre récepteur original soit une zone de transit de laquelle Data Factory continue d’écrire des données vers le récepteur final via la copie intermédiaire intégrée. En savoir plus sur [Delta Lake en tant que récepteur](#delta-lake-as-sink).

Le cluster Databricks doit avoir accès à un compte Stockage Blob Azure ou Azure Data Lake Storage Gen2, à la fois au conteneur de stockage/système de fichiers utilisé pour la source/le récepteur/le transit, et au conteneur/système de fichiers dans lequel vous voulez écrire les tables Delta Lake.

- Pour utiliser **Azure Data Lake Storage Gen2**, vous pouvez configurer un **principal de service** ou une **clé d’accès de compte de stockage** sur le cluster Databricks dans le cadre de la configuration Apache Spark. Suivez la procédure décrite dans [Accéder directement avec le principal de service](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-with-service-principal-and-oauth-20) ou [Accéder directement avec la clé d’accès du compte de stockage](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-using-the-storage-account-access-key).

- Pour utiliser **Stockage Blob Azure**, vous pouvez configurer une **clé d’accès de compte de stockage** ou un **jeton SAS** sur le cluster Databricks dans le cadre de la configuration Apache Spark. Suivez la procédure décrite dans [Accéder à Stockage Blob Azure à l’aide de l’API RDD](/azure/databricks/data/data-sources/azure/azure-storage#access-azure-blob-storage-using-the-rdd-api).

Pendant l’exécution de l’activité de copie, si le cluster que vous avez configuré a été arrêté, Data Factory le redémarre automatiquement. Si vous créez un pipeline à l’aide de l’interface utilisateur de création Data Factory, pour les opérations telles que l’aperçu des données, vous devez avoir un cluster actif, car Data Factory ne démarre pas le cluster en votre nom.

#### <a name="specify-the-cluster-configuration"></a>Spécifier la configuration du cluster

1. Dans le menu déroulant **Mode du cluster**, sélectionnez **Standard**.

2. Dans le menu déroulant **Version du runtime Databricks**, sélectionnez une version de runtime Databricks.

3. Activez [Optimisation automatique](/azure/databricks/delta/optimizations/auto-optimize) en ajoutant les propriétés suivantes à votre [configuration Spark](/azure/databricks/clusters/configure#spark-config) :

   ```
   spark.databricks.delta.optimizeWrite.enabled true
   spark.databricks.delta.autoCompact.enabled true
   ```

4. Configurez votre cluster en fonction de vos besoins en matière d’intégration et de mise à l’échelle.

Pour plus d’informations sur la configuration du cluster, consultez [Configurer des clusters](/azure/databricks/clusters/configure).

## <a name="get-started"></a>Prise en main

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes donnent des précisions sur les propriétés utilisées qui définissent des entités Data Factory propres à un connecteur Azure Databricks Delta Lake.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés suivantes sont prises en charge pour un service lié à Azure Databricks Delta Lake.

| Propriété    | Description                                                  | Obligatoire |
| :---------- | :----------------------------------------------------------- | :------- |
| type        | La propriété de type doit être définie sur **AzureDatabricksDeltaLake**. | Oui      |
| domaine      | Spécifiez l’URL de l’espace de travail Azure Databricks, par exemple `https://adb-xxxxxxxxx.xx.azuredatabricks.net`. |          |
| clusterId   | Spécifiez l’ID de cluster d’un cluster existant. Il doit s’agir d’un cluster interactif déjà créé. <br>Vous pouvez trouver l’ID de cluster d’un cluster interactif sur l’espace de travail Databricks -> Clusters -> Nom du cluster interactif -> Configuration -> Balises. [En savoir plus](/azure/databricks/clusters/configure#cluster-tags) |          |
| accessToken | Un jeton d’accès est requis pour que la fabrique de données s’authentifie auprès d’Azure Databricks. Un jeton d’accès doit être généré à partir de l’espace de travail Databricks. Des étapes plus détaillées pour rechercher le jeton d’accès sont disponibles [ici](/azure/databricks/dev-tools/api/latest/authentication#generate-token). |          |
| connectVia  | Le [runtime d’intégration](concepts-integration-runtime.md) utilisé pour la connexion à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé si votre banque de données se trouve sur un réseau privé. À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. | Non       |

**Exemple :**

```json
{
    "name": "AzureDatabricksDeltaLakeLinkedService",
    "properties": {
        "type": "AzureDatabricksDeltaLake",
        "typeProperties": {
            "domain": "https://adb-xxxxxxxxx.xx.azuredatabricks.net",
            "clusterId": "<cluster id>",
            "accessToken": {
                "type": "SecureString", 
                "value": "<access token>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). 

Les propriétés suivantes sont prises en charge pour le jeu de données Azure Databricks Delta Lake.

| Propriété  | Description                                                  | Obligatoire                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | La propriété type du jeu de données doit être définie sur **AzureDatabricksDeltaLakeDataset**. | Oui                         |
| database | Nom de la base de données. |Non pour Source, Oui pour Récepteur  |
| table | Nom de la table Delta. |Non pour Source, Oui pour Récepteur  |

**Exemple :**

```json
{
    "name": "AzureDatabricksDeltaLakeDataset",
    "properties": {
        "type": "AzureDatabricksDeltaLakeDataset",
        "typeProperties": {
            "database": "<database name>",
            "table": "<delta table name>"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "linkedServiceName": {
            "referenceName": "<name of linked service>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par Azure Databricks Delta Lake en tant que source et récepteur.

### <a name="delta-lake-as-source"></a>Delta Lake en tant que source

Pour copier des données à partir d’Azure Databricks Delta Lake, les propriétés suivantes sont prises en charge dans la section **source** de l’activité de copie.

| Propriété                     | Description                                                  | Obligatoire |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | La propriété type de la source de l’activité de copie doit être définie sur **AzureDatabricksDeltaLakeSource**. | Oui      |
| query          | Spécifier la requête SQL pour lire les données. Pour le contrôle du voyage dans le temps, suivez le modèle ci-dessous :<br>- `SELECT * FROM events TIMESTAMP AS OF timestamp_expression`<br>- `SELECT * FROM events VERSION AS OF version` | Non       |
| exportSettings | Paramètres avancés utilisés pour récupérer des données de la table Delta. | Non       |
| ***Sous `exportSettings` :** _ |  |  |
| type | Type de la commande d’exportation, définie sur _*AzureDatabricksDeltaLakeExportCommand**. | Oui |
| dateFormat | Définissez le format du type date sur une chaîne avec un format de date. Les formats de date personnalisés suivent les formats du [modèle datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). En l’absence de spécification, la valeur par défaut est `yyyy-MM-dd`. | Non |
| timestampFormat | Définissez le format du type timestamp sur une chaîne avec un format timestamp. Les formats de date personnalisés suivent les formats du [modèle datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). En l’absence de spécification, la valeur par défaut est `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]`. | Non |

#### <a name="direct-copy-from-delta-lake"></a>Copie directe à partir de Delta Lake

Si le format et le magasin de données récepteur remplissent les critères décrits dans cette section, vous pouvez utiliser l’activité de copie pour effectuer une copie directe de la table Delta Azure Databricks vers un récepteur. Data Factory vérifie les paramètres et fait échouer l’exécution de l’activité de copie si les critères suivants ne sont pas satisfaits :

- Le **service lié au récepteur** est [Stockage Blob Azure](connector-azure-blob-storage.md) ou [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md). Les informations d’identification du compte doivent être préconfigurées dans la configuration du cluster Azure Databricks. Pour plus d’informations, consultez les [Prérequis](#prerequisites).

- Le **format de données du récepteur** est **Parquet**, **Texte délimité** ou **Avro** avec les configurations suivantes, et pointe vers un dossier plutôt qu’un fichier.

    - Pour le format **Parquet**, le codec de compression est **aucun**, **snappy** ou **gzip**.
    - Pour le format **texte délimité** :
        - `rowDelimiter` est un caractère unique quelconque.
        - `compression` peut être **aucun**, **bzip2** ou **gzip**.
        - `encodingName` UTF-7 n’est pas pris en charge.
    - Pour le format **Avro**, le codec de compression est **aucun**, **deflate** ou **snappy**.

- Dans la source de l’activité de copie, `additionalColumns` n’est pas spécifié.
- Si vous copiez des données dans du texte délimité, dans le récepteur de l’activité de copie, `fileExtension` doit être « .csv ».
- Dans le mappage de l’activité de copie, la conversion de type n’est pas activée.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="staged-copy-from-delta-lake"></a>Copie intermédiaire à partir de Delta Lake

Lorsque le format ou le magasin de données récepteur ne respecte pas les critères de la copie directe, comme indiqué dans la dernière section, activez la copie intermédiaire intégrée à l’aide d’une instance de stockage Azure intermédiaire. La fonctionnalité de copie intermédiaire offre également un meilleur débit. Data Factory exporte les données d’Azure Databricks Delta Lake vers le stockage intermédiaire, les copie sur le récepteur, puis nettoie vos données temporaires sur le stockage intermédiaire. Pour plus d’informations sur la copie de données à l’aide de la mise en lots, consultez [Copie intermédiaire](copy-activity-performance-features.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) ou un [service lié Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) qui fait référence au compte de stockage comme intermédiaire temporaire. Spécifiez ensuite les propriétés `enableStaging` et `stagingSettings` dans l’activité de copie.

>[!NOTE]
>Les informations d’identification du compte de stockage intermédiaire doivent être préconfigurées dans la configuration du cluster Azure Databricks. Pour plus d’informations, consultez les [Prérequis](#prerequisites).

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

### <a name="delta-lake-as-sink"></a>Delta Lake en tant que récepteur

Pour copier des données vers Azure Databricks Delta Lake, les propriétés suivantes sont prises en charge dans la section **récepteur** de l’activité de copie.

| Propriété      | Description                                                  | Obligatoire |
| :------------ | :----------------------------------------------------------- | :------- |
| type          | Propriété type du récepteur de l’activité de copie, définie sur **AzureDatabricksDeltaLakeSink**. | Oui      |
| preCopyScript | Spécifiez une requête SQL pour l’activité de copie à exécuter avant l’écriture de données dans la table Delta Databricks à chaque exécution. Vous pouvez utiliser cette propriété pour nettoyer les données préchargées ou ajouter une instruction Tronquer la table ou Vider. | Non       |
| importSettings | Paramètres avancés utilisés pour écrire des données dans la table Delta. | Non |
| **_Sous `importSettings` :_* _ |                                                              |  |
| type | Type de la commande d’importation, définie sur _*AzureDatabricksDeltaLakeImportCommand**. | Oui |
| dateFormat | Définissez le format de la chaîne sur un type date avec un format de date. Les formats de date personnalisés suivent les formats du [modèle datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). En l’absence de spécification, la valeur par défaut est `yyyy-MM-dd`. | Non |
| timestampFormat | Définissez le format de la chaîne sur un type timestamp avec un format timestamp. Les formats de date personnalisés suivent les formats du [modèle datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). En l’absence de spécification, la valeur par défaut est `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]`. | Non |

#### <a name="direct-copy-to-delta-lake"></a>Copie directe vers Delta Lake

Si le format et le magasin de données source remplissent les critères décrits dans cette section, vous pouvez utiliser l’activité de copie pour effectuer une copie directe depuis une source vers Azure Databricks Delta Lake. Azure Data Factory vérifie les paramètres et fait échouer l’exécution de l’activité de copie si les critères suivants ne sont pas satisfaits :

- Le **service lié à la source** est [Stockage Blob Azure](connector-azure-blob-storage.md) ou [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md). Les informations d’identification du compte doivent être préconfigurées dans la configuration du cluster Azure Databricks. Pour plus d’informations, consultez les [Prérequis](#prerequisites).

- Le **format de données de la source** est **Parquet**, **Texte délimité** ou **Avro** avec les configurations suivantes, et pointe vers un dossier plutôt qu’un fichier.

    - Pour le format **Parquet**, le codec de compression est **aucun**, **snappy** ou **gzip**.
    - Pour le format **texte délimité** :
        - `rowDelimiter` est la valeur par défaut ou n’importe quel caractère unique.
        - `compression` peut être **aucun**, **bzip2** ou **gzip**.
        - `encodingName` UTF-7 n’est pas pris en charge.
    - Pour le format **Avro**, le codec de compression est **aucun**, **deflate** ou **snappy**.

- Dans la source de l’activité de copie : 

    - `wildcardFileName` contient uniquement le caractère générique `*` mais pas `?`, et `wildcardFolderName` n’est pas spécifié.
    - `prefix`, `modifiedDateTimeStart`, `modifiedDateTimeEnd` et `enablePartitionDiscovery` ne sont pas spécifiés.
    - `additionalColumns` n'est pas spécifié.

- Dans le mappage de l’activité de copie, la conversion de type n’est pas activée.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            }
        }
    }
]
```

#### <a name="staged-copy-to-delta-lake"></a>Copie intermédiaire vers Delta Lake

Lorsque le format ou le magasin de données source ne respecte pas les critères de la copie directe, comme indiqué dans la dernière section, activez la copie intermédiaire intégrée à l’aide d’une instance de stockage Azure intermédiaire. La fonctionnalité de copie intermédiaire offre également un meilleur débit. Data Factory convertit automatiquement les données pour respecter les exigences du format de données dans le stockage intermédiaire, puis charge les données dans Delta Lake à partir de là. Enfin, il nettoie vos données temporaires sur le stockage. Pour plus d’informations sur la copie de données à l’aide de la mise en lots, consultez [Copie intermédiaire](copy-activity-performance-features.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) ou un [service lié Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) qui fait référence au compte de stockage comme intermédiaire temporaire. Spécifiez ensuite les propriétés `enableStaging` et `stagingSettings` dans l’activité de copie.

>[!NOTE]
>Les informations d’identification du compte de stockage intermédiaire doivent être préconfigurées dans la configuration du cluster Azure Databricks. Pour plus d’informations, consultez les [Prérequis](#prerequisites).

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

## <a name="monitoring"></a>Surveillance

Azure Data Factory fournit la même [expérience de surveillance de l’activité de copie](copy-activity-monitoring.md) que les autres connecteurs. De plus, le chargement des données depuis et vers Delta Lake étant exécuté sur votre cluster Azure Databricks, vous pouvez également [consulter les journaux de cluster détaillés](/azure/databricks/clusters/clusters-manage#--view-cluster-logs) et [surveiller les performances](/azure/databricks/clusters/clusters-manage#--monitor-performance).

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour plus d’informations sur les propriétés, consultez [Activité de recherche](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs par l’activité de copie dans Data Factory, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).