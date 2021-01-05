---
title: Copier et transformer des données dans Snowflake
description: Découvrez comment copier et transformer des données dans Snowflake à l’aide de Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/08/2020
ms.openlocfilehash: 49e4a6f7f8c268669a94796257d5740ec6f4e6ff
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96902083"
---
# <a name="copy-and-transform-data-in-snowflake-by-using-azure-data-factory"></a>Copier et transformer des données dans Snowflake en utilisant Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article indique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données depuis et vers Snowflake et comment utiliser Data Flow pour transformer les données dans Snowflake. Pour plus d’informations sur Data Factory, consultez [l’article d’introduction](introduction.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Snowflake est pris en charge pour les activités suivantes :

- [Activité de copie](copy-activity-overview.md) avec une table [Prise en charge de la matrice source/récepteur](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Pour l’activité de copie, ce connecteur Snowflake prend en charge les fonctions suivantes :

- Copiez des données à partir de Snowflake qui utilise la commande [COPY into [emplacement]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html) de Snowflake pour obtenir les meilleures performances.
- Copiez des données dans Snowflake qui utilise la commande [COPY into [table]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html) de Snowflake pour obtenir les meilleures performances. Il prend en charge Snowflake sur Azure. 

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes donnent des précisions sur les propriétés utilisées qui définissent des entités Data Factory propres à un connecteur Snowflake.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour un service lié Snowflake sont les suivantes.

| Propriété         | Description                                                  | Obligatoire |
| :--------------- | :----------------------------------------------------------- | :------- |
| type             | La propriété type doit être définie sur **Snowflake**.              | Oui      |
| connectionString | Spécifie les informations requises pour se connecter à l’instance de Snowflake. Vous pouvez choisir de placer le mot de passe ou la chaîne de connexion entière dans Azure Key Vault. Pour plus d'informations, reportez-vous aux exemples sous le tableau et à l'article [Stocker des informations d'identification dans Azure Key Vault](store-credentials-in-key-vault.md).<br><br>Paramètres standard :<br>- **Nom du compte :** [nom de compte complet](https://docs.snowflake.net/manuals/user-guide/connecting.html#your-snowflake-account-name) de votre compte Snowflake (incluant des segments supplémentaires identifiant la région et la plateforme cloud), par exemple, xy12345.east-us-2.azure.<br/>- **Nom d’utilisateur :** ID de connexion de l’utilisateur pour la connexion.<br>- **Mot de passe :** Mot de passe de l’utilisateur.<br>- **Base de données :** base de données par défaut à utiliser une fois connecté. Il doit s’agir d’une base de données existante pour laquelle le rôle spécifié dispose de privilèges.<br>- **Entrepôt :** entrepôt virtuel à utiliser une fois connecté. Il doit s’agir d’un entrepôt existant pour lequel le rôle spécifié dispose de privilèges.<br>- **Rôle :** rôle de contrôle d’accès par défaut à utiliser dans la session Snowflake. Le rôle spécifié doit être un rôle existant qui a déjà été affecté à l’utilisateur spécifié. Le rôle par défaut est PUBLIC. | Oui      |
| connectVia       | Le [runtime d’intégration](concepts-integration-runtime.md) utilisé pour la connexion à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé si votre banque de données se trouve sur un réseau privé. À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. | Non       |

**Exemple :**

```json
{
    "name": "SnowflakeLinkedService",
    "properties": {
        "type": "Snowflake",
        "typeProperties": {
            "connectionString": "jdbc:snowflake://<accountname>.snowflakecomputing.com/?user=<username>&db=<database>&warehouse=<warehouse>&role=<myRole>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Mot de passe dans Azure Key Vault :**

```json
{
    "name": "SnowflakeLinkedService",
    "properties": {
        "type": "Snowflake",
        "typeProperties": {
            "connectionString": "jdbc:snowflake://<accountname>.snowflakecomputing.com/?user=<username>&db=<database>&warehouse=<warehouse>&role=<myRole>",
            "password": {
                "type": "AzureKeyVaultSecret",
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>",
                    "type": "LinkedServiceReference"
                }, 
                "secretName": "<secretName>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). 

Les propriétés suivantes sont prises en charge pour le jeu de données Snowflake.

| Propriété  | Description                                                  | Obligatoire                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | La propriété type du jeu de données doit être définie sur **SnowflakeTable**. | Oui                         |
| schéma | Nom du schéma. Notez que le nom du schéma respecte la casse dans ADF. |Non pour Source, Oui pour Récepteur  |
| table | Nom de la table/vue. Notez que le nom de la table respecte la casse dans ADF. |Non pour Source, Oui pour Récepteur  |

**Exemple :**

```json
{
    "name": "SnowflakeDataset",
    "properties": {
        "type": "SnowflakeTable",
        "typeProperties": {
            "schema": "<Schema name for your Snowflake database>",
            "table": "<Table name for your Snowflake database>"
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

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par Snowflake en tant que source et récepteur.

### <a name="snowflake-as-the-source"></a>Snowflake en tant que source

Un connecteur Snowflake utilise la commande [COPY into [emplacement]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html) de Snowflake pour obtenir les meilleures performances.

Si le magasin de données récepteur et le format sont pris en charge en mode natif par la commande COPY de Snowflake, vous pouvez utiliser l’activité de copie pour copier directement de Snowflake vers un récepteur. Pour plus d’informations, consultez [Copie directe à l’aide de Snowflake](#direct-copy-from-snowflake). Dans le cas contraire, utilisez la [copie intermédiaire intégrée à Snowflake](#staged-copy-from-snowflake).

Pour copier des données à partir de Snowflake, les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes.

| Propriété                     | Description                                                  | Obligatoire |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | La propriété type de la source de l’activité de copie doit être définie sur **SnowflakeSource**. | Oui      |
| query          | Spécifie la requête SQL pour lire les données de Snowflake. Si les noms du schéma, de la table et des colonnes contiennent des minuscules, citez l’identificateur d’objet dans la requête, par exemple `select * from "schema"."myTable"`.<br>L’exécution de la procédure stockée n'est pas prise en charge. | Non       |
| exportSettings | Paramètres avancés utilisés pour récupérer des données de Snowflake. Vous pouvez configurer pris en charge par la commande COPY into que Data Factory empruntera lorsque vous appelez l’instruction. | Non       |
| ***Sous `exportSettings` :** _ |  |  |
| type | Type de commande d’exportation, défini sur _*SnowflakeExportCopyCommand**. | Oui |
| additionalCopyOptions | Options de copie supplémentaires, fournies sous la forme d’un dictionnaire de paires clé-valeur. Exemples : MAX_FILE_SIZE, OVERWRITE. Pour plus d'informations, consultez [Options de copie de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html#copy-options-copyoptions). | Non |
| additionalFormatOptions | Options de format de fichier supplémentaires, fournies à la commande COPY sous la forme d’un dictionnaire de paires clé-valeur. Exemples : DATE_FORMAT, TIME_FORMAT, TIMESTAMP_FORMAT. Pour plus d’informations, consultez [Options de type de format de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location.html#format-type-options-formattypeoptions). | Non |

#### <a name="direct-copy-from-snowflake"></a>Copie directe à partir de Snowflake

Si le magasin de données récepteur et le format remplissent les critères décrits dans cette section, vous pouvez utiliser l’activité de copie pour effectuer une copie directe de Snowflake vers un récepteur. Data Factory vérifie les paramètres et fait échouer l’exécution de l’activité de copie si les critères suivants ne sont pas satisfaits :

- Le **service lié au récepteur** est le [**stockage d’objets blob Azure**](connector-azure-blob-storage.md) avec l’authentification par **signature d’accès partagé**. Si vous souhaitez copier directement des données dans Azure Data Lake Storage Gen2 dans le format pris en charge suivant, vous pouvez créer un service lié Azure Blob avec une authentification par SAP sur votre compte ADLS Gen2, afin d’éviter d’utiliser la [copie intermédiaire provenant de Snowflake](#staged-copy-from-snowflake).

- Le **format de données du récepteur** est **Parquet**, **Texte délimité** ou **JSON** avec les configurations suivantes :

    - Pour le format **Parquet**, le codec de compression est **Aucun**, **Snappy** ou **Lzo**.
    - Pour le format **texte délimité** :
        - `rowDelimiter` est **\r\n**, ou n’importe quel caractère unique.
        - `compression` peut être **aucune compression**, **gzip**, **bzip2**, ou **deflate**.
        - `encodingName` conserve sa valeur par défaut ou est défini sur **utf-8**.
        - `quoteChar` est un **guillemet double**, un **guillemet simple** ou une **chaîne vide** (sans guillemets).
    - Pour le format **JSON**, la copie directe ne prend en charge que le cas où la table Snowflake de la source ou le résultat de la requête ne possède qu’une seule colonne, dont les données sont de type **VARIANT**, **OBJET** ou **TABLEAU**.
        - `compression` peut être **aucune compression**, **gzip**, **bzip2**, ou **deflate**.
        - `encodingName` conserve sa valeur par défaut ou est défini sur **utf-8**.
        - `filePattern` dans le récepteur de l’activité de copie conserve sa valeur par défaut ou est défini sur **setOfObjects**.
- Dans la source de l’activité de copie, `additionalColumns` n’est pas spécifié.
- Le mappage de colonnes n’est pas spécifié.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Snowflake input dataset name>",
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
                "type": "SnowflakeSource",
                "sqlReaderQuery": "SELECT * FROM MYTABLE",
                "exportSettings": {
                    "type": "SnowflakeExportCopyCommand",
                    "additionalCopyOptions": {
                        "MAX_FILE_SIZE": "64000000",
                        "OVERWRITE": true
                    },
                    "additionalFormatOptions": {
                        "DATE_FORMAT": "'MM/DD/YYYY'"
                    }
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="staged-copy-from-snowflake"></a>Copie intermédiaire à partir de Snowflake

Lorsque le magasin de données récepteur ou le format n’est pas compatible en mode natif avec la commande COPY de Snowflake, comme indiqué dans la dernière section, activez la copie intermédiaire intégrée à l’aide d’une instance de stockage intermédiaire d’objets blob Azure. La fonctionnalité de copie intermédiaire offre également un meilleur débit. Data Factory exporte des données à partir de Snowflake dans le stockage intermédiaire, puis copie les données sur le récepteur et nettoie vos données temporaires sur le stockage intermédiaire. Pour plus d’informations sur la copie de données à l’aide de la mise en lots, consultez [Copie intermédiaire](copy-activity-performance-features.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié au Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) qui fait référence au compte de stockage Azure en tant qu’intermédiaire. Spécifiez ensuite les propriétés `enableStaging` et `stagingSettings` dans l’activité de copie.

> [!NOTE]
> Le service lié au Stockage Blob Azure intermédiaire doit utiliser l’authentification par signature d’accès partagé, comme exigé par la commande COPY de Snowflake. 

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Snowflake input dataset name>",
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
                "type": "SnowflakeSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
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

### <a name="snowflake-as-sink"></a>Snowflake en tant que récepteur

Un connecteur Snowflake utilise la commande [COPY into [table]](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html) de Snowflake pour obtenir les meilleures performances. Il prend en charge l’écriture de données vers Snowflake sur Azure.

Si le magasin de données source et le format sont pris en charge en mode natif par la commande COPY de Snowflake, vous pouvez utiliser l’activité de copie pour copier directement d’une source vers Snowflake. Pour plus d’informations, consultez [Copie directe vers Snowflake](#direct-copy-to-snowflake). Dans le cas contraire, utilisez la [copie intermédiaire intégrée vers Snowflake](#staged-copy-to-snowflake).

Pour copier des données dans Snowflake, les propriétés suivantes sont prises en charge dans la section **récepteur** de l’activité de copie.

| Propriété          | Description                                                  | Obligatoire                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| type              | La propriété type du récepteur de l'activité de copie, définie sur **SnowflakeSink**. | Oui                                           |
| preCopyScript     | Spécifiez une requête SQL pour l’activité de copie à exécuter avant l’écriture de données dans Snowflake à chaque exécution. Utilisez cette propriété pour nettoyer les données préchargées. | Non                                            |
| importSettings | Paramètres avancés utilisés pour écrire des données dans Snowflake. Vous pouvez configurer pris en charge par la commande COPY into que Data Factory empruntera lorsque vous appelez l’instruction. | Non |
| **_Sous `importSettings` :_* _ |                                                              |  |
| type | Type de commande d’importation, défini sur _*SnowflakeImportCopyCommand**. | Oui |
| additionalCopyOptions | Options de copie supplémentaires, fournies sous la forme d’un dictionnaire de paires clé-valeur. Exemples : ON_ERROR, FORCE, LOAD_UNCERTAIN_FILES. Pour plus d'informations, consultez [Options de copie de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html#copy-options-copyoptions). | Non |
| additionalFormatOptions | Options de format de fichier supplémentaires, fournies à la commande COPY sous la forme d’un dictionnaire de paires clé-valeur. Exemples : DATE_FORMAT, TIME_FORMAT, TIMESTAMP_FORMAT. Pour plus d’informations, consultez [Options de type de format de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table.html#format-type-options-formattypeoptions). | Non |

#### <a name="direct-copy-to-snowflake"></a>Copie directe vers Snowflake

Si le magasin de données source et le format remplissent les critères décrits dans cette section, vous pouvez utiliser l’activité de copie pour effectuer une copie directe depuis une source vers Snowflake. Azure Data Factory vérifie les paramètres et fait échouer l’exécution de l’activité de copie si les critères suivants ne sont pas satisfaits :

- Le **service lié à la source** est le [**stockage d’objets blob Azure**](connector-azure-blob-storage.md) avec l’authentification par **signature d’accès partagé**. Si vous souhaitez copier directement des données depuis Azure Data Lake Storage Gen2 dans le format pris en charge suivant, vous pouvez créer un service lié Azure Blob avec une authentification par SAP sur votre compte ADLS Gen2, afin d’éviter d’utiliser la [copie intermédiaire vers Snowflake](#staged-copy-to-snowflake).

- Le **format de données de la source** est **Parquet**, **Texte délimité** ou **JSON** avec les configurations suivantes :

    - Pour le format **Parquet**, le codec de compression est **Aucun** ou **Snappy**.

    - Pour le format **texte délimité** :
        - `rowDelimiter` est **\r\n**, ou n’importe quel caractère unique. Si le délimiteur de ligne n’est pas « \r\n », `firstRowAsHeader` doit être **false** et `skipLineCount` n’est pas spécifié.
        - `compression` peut être **aucune compression**, **gzip**, **bzip2**, ou **deflate**.
        - `encodingName` est conservé comme valeur par défaut ou défini sur "UTF-8", "UTF-16", "UTF-16BE", "UTF-32", "UTF-32BE", "BIG5", "EUC-JP", "EUC-KR", "GB18030", "ISO-2022-JP", "ISO-2022-KR", "ISO-8859-1", "ISO-8859-2", "ISO-8859-5", "ISO-8859-6", "ISO-8859-7", "ISO-8859-8", "ISO-8859-9", "WINDOWS-1250", "WINDOWS-1251", "WINDOWS-1252", "WINDOWS-1253", "WINDOWS-1254", "WINDOWS-1255".
        - `quoteChar` est un **guillemet double**, un **guillemet simple** ou une **chaîne vide** (sans guillemets).
    - Pour le format **JSON**, la copie directe ne prend en charge que le cas où la table Snowflake du récepteur ne possède qu’une seule colonne, dont les données sont de type **VARIANT**, **OBJET** ou **TABLEAU**.
        - `compression` peut être **aucune compression**, **gzip**, **bzip2**, ou **deflate**.
        - `encodingName` conserve sa valeur par défaut ou est défini sur **utf-8**.
        - Le mappage de colonnes n’est pas spécifié.

- Dans la source de l’activité de copie : 

   -  `additionalColumns` n'est pas spécifié.
   - Si votre source est un dossier, `recursive` a la valeur true.
   - `prefix`, `modifiedDateTimeStart`, `modifiedDateTimeEnd` et `enablePartitionDiscovery` ne sont pas spécifiés.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Snowflake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SnowflakeSink",
                "importSettings": {
                    "type": "SnowflakeImportCopyCommand",
                    "copyOptions": {
                        "FORCE": "TRUE",
                        "ON_ERROR": "SKIP_FILE",
                    },
                    "fileFormatOptions": {
                        "DATE_FORMAT": "YYYY-MM-DD",
                    }
                }
            }
        }
    }
]
```

#### <a name="staged-copy-to-snowflake"></a>Copie intermédiaire vers Snowflake

Lorsque le format ou le magasin de données source n’est pas compatible en mode natif avec la commande COPY de Snowflake, comme indiqué dans la dernière section, activez la copie intermédiaire intégrée à l’aide d’une instance de Stockage Blob Azure temporaire. La fonctionnalité de copie intermédiaire offre également un meilleur débit. Data Factory convertit automatiquement les données pour répondre aux exigences de format de données de Snowflake. Il appelle ensuite la commande COPY pour charger les données dans Snowflake. Pour finir, il nettoie vos données temporaires du stockage Blob. Pour plus d’informations sur la copie de données à l’aide de la mise en lots, consultez [Copie intermédiaire](copy-activity-performance-features.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié au Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) qui fait référence au compte de stockage Azure en tant qu’intermédiaire. Spécifiez ensuite les propriétés `enableStaging` et `stagingSettings` dans l’activité de copie.

> [!NOTE]
> Le service lié au Stockage Blob Azure intermédiaire doit utiliser l’authentification par signature d’accès partagé, comme exigé par la commande COPY de Snowflake.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToSnowflake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Snowflake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "SnowflakeSink"
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

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Lors de la transformation de données dans le flux de données de mappage, vous pouvez lire et écrire dans les tables de Snowflake. Pour plus d’informations, consultez la [transformation de la source](data-flow-source.md) et la [transformation du récepteur](data-flow-sink.md) dans le flux de données de mappage. Vous pouvez choisir d’utiliser un jeu de données Snowflake ou un [jeu de données inlined](data-flow-source.md#inline-datasets) en tant que type de source et de récepteur.

### <a name="source-transformation"></a>Transformation de la source

Le tableau ci-dessous répertorie les propriétés prises en charge par la source Snowflake. Vous pouvez modifier ces propriétés sous l’onglet **Options de la source**. Le connecteur utilise le [transfert de données interne](https://docs.snowflake.com/en/user-guide/spark-connector-overview.html#internal-data-transfer) de Snowflake.

| Nom | Description | Obligatoire | Valeurs autorisées | Propriété du script de flux de données |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Table de charge de travail | Si vous sélectionnez Table comme entrée, le flux de données extraira toutes les données de la table spécifiée dans le jeu de données Snowflake ou dans les options source lorsque vous utilisez le jeu de données inlined. | Non | String | *(pour le jeu de données inlined uniquement)*<br>tableName<br>schemaName |
| Requête | Si vous sélectionnez Requête comme entrée, entrez une requête pour extraire les données de Snowflake. Ce paramètre remplace toute table que vous avez choisie dans le jeu de données.<br>Si les noms du schéma, de la table et des colonnes contiennent des minuscules, citez l’identificateur d’objet dans la requête, par exemple `select * from "schema"."myTable"`. | Non | String | query |

#### <a name="snowflake-source-script-examples"></a>Exemples de scripts de source Snowflake

Quand vous utilisez un jeu de données Snowflake comme type de source, le script de flux de données associé est le suivant :

```
source(allowSchemaDrift: true,
    validateSchema: false,
    query: 'select * from MYTABLE',
    format: 'query') ~> SnowflakeSource
```

Si vous utilisez un jeu de données inlined, le script de flux de données associé est le suivant :

```
source(allowSchemaDrift: true,
    validateSchema: false,
    format: 'query',
    query: 'select * from MYTABLE',
    store: 'snowflake') ~> SnowflakeSource
```

### <a name="sink-transformation"></a>Transformation du récepteur

Le tableau ci-dessous répertorie les propriétés prises en charge par le récepteur Snowflake. Vous pouvez modifier ces propriétés sous l’onglet **Paramètres**. Lorsque vous utilisez un jeu de données inlined, vous verrez des paramètres supplémentaires qui sont les mêmes que les propriétés décrites dans la section [Propriétés du jeu de données](#dataset-properties). Le connecteur utilise le [transfert de données interne](https://docs.snowflake.com/en/user-guide/spark-connector-overview.html#internal-data-transfer) de Snowflake.

| Nom | Description | Obligatoire | Valeurs autorisées | Propriété du script de flux de données |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Mettre à jour la méthode | Spécifiez quelles opérations sont autorisées sur votre destination Snowflake.<br>Pour mettre à jour, effectuer un upsert ou supprimer des lignes, une [transformation de modification de ligne](data-flow-alter-row.md) est requise afin de baliser les lignes relatives à ces actions. | Oui | `true` ou `false` | deletable <br/>insertable <br/>updateable <br/>upsertable |
| Colonnes clés | Pour les mises à jour, les opérations upsert et les suppressions, une ou plusieurs colonnes clés doivent être définies afin de déterminer la ligne à modifier. | Non | Array | clés |
| Action table | Détermine si toutes les lignes de la table de destination doivent être recréées ou supprimées avant l’écriture.<br>- **Aucun** : Aucune action ne sera effectuée sur la table.<br>- **Recréer** : La table sera supprimée et recréée. Obligatoire en cas de création dynamique d’une nouvelle table.<br>- **Tronquer** : Toutes les lignes de la table cible seront supprimées. | Non | `true` ou `false` | recreate<br/>truncate |

#### <a name="snowflake-sink-script-examples"></a>Exemples de scripts de récepteur Snowflake

Quand vous utilisez un jeu de données Snowflake comme type de récepteur, le script de flux de données associé est le suivant :

```
IncomingStream sink(allowSchemaDrift: true,
    validateSchema: false,
    deletable:true,
    insertable:true,
    updateable:true,
    upsertable:false,
    keys:['movieId'],
    format: 'table',
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> SnowflakeSink
```

Si vous utilisez un jeu de données inlined, le script de flux de données associé est le suivant :

```
IncomingStream sink(allowSchemaDrift: true,
    validateSchema: false,
    format: 'table',
    tableName: 'table',
    schemaName: 'schema',
    deletable: true,
    insertable: true,
    updateable: true,
    upsertable: false,
    store: 'snowflake',
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> SnowflakeSink
```

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour plus d’informations sur les propriétés, consultez [Activité de recherche](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs par l’activité de copie dans Data Factory, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
