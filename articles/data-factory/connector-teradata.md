---
title: Copier des données depuis une base de données Teradata Vantage à l’aide d’Azure Data Factory
description: Découvrez le connecteur Teradata du service Data Factory, qui vous permet de copier des données Teradata Vantage vers des magasins de données pris en charge par Data Factory en tant que récepteurs.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 11/26/2020
ms.author: jingwang
ms.openlocfilehash: a48ac86e8f9814adef9be2360b2446335d368447
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296554"
---
# <a name="copy-data-from-teradata-vantage-by-using-azure-data-factory"></a>Copier des données depuis une base de données Teradata Vantage à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
>
> * [Version 1](v1/data-factory-onprem-teradata-connector.md)
> * [Version actuelle](connector-teradata.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité de copie dans Azure Data Factory pour copier des données à partir de Teradata Vantage. Il s’appuie sur la [vue d’ensemble de l’activité de copie](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Teradata est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier des données de Teradata Vantage vers tout magasin de données récepteur pris en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur Teradata prend en charge :

- Teradata **versions 14.10, 15.0, 15.10, 16.0, 16.10 et 16.20**.
- Copie des données avec l’authentification **De base**, **Windows** ou **LDAP**.
- Copie en parallèle à partir d’une source Teradata. Pour en savoir plus, voir [Copie en parallèle à partir de Teradata](#parallel-copy-from-teradata).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

Si vous utilisez un runtime d’intégration auto-hébergé, notez qu’il fournit un pilote Teradata intégré depuis la version 3.18. Vous n’avez pas besoin d’installer manuellement un pilote. Le pilote requiert l’installation du package Redistribuable Visual C++ pour Visual Studio 2012 Update 4 sur l’ordinateur du runtime d’intégration auto-hébergé. S’il n’est pas déjà installé, vous pouvez le télécharger [ici](https://www.microsoft.com/en-sg/download/details.aspx?id=30679).

## <a name="getting-started"></a>Prise en main

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory propres au connecteur Teradata.

## <a name="linked-service-properties"></a>Propriétés du service lié

Le service lié Teradata prend en charge les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété de type doit être définie sur **Teradata**. | Oui |
| connectionString | Spécifie les informations requises pour se connecter à l’instance de Teradata. Consultez les exemples suivants.<br/>Vous pouvez également définir un mot de passe dans Azure Key Vault et extraire la configuration `password` de la chaîne de connexion. Pour plus d’informations, consultez la section [Stocker des informations d’identification dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| username | Spécifiez le nom d’utilisateur associé à la connexion à Teradata. S’applique lors de l’utilisation de l’authentification Windows. | Non |
| mot de passe | Spécifiez un mot de passe pour le compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. Vous pouvez également choisir de [référencer un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). <br>S’applique lors de l’utilisation de l’authentification Windows ou du référencement du mot de passe dans Key Vault pour l’authentification de base. | Non |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion à la banque de données. Pour plus d’informations, consultez la section [Conditions préalables](#prerequisites). À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. |Non |

Autres propriétés de connexion que vous pouvez définir dans la chaîne de connexion selon votre cas :

| Propriété | Description | Valeur par défaut |
|:--- |:--- |:--- |
| UseDataEncryption | Spécifie s’il faut chiffrer toutes les communications avec la base de données Teradata. Les valeurs autorisées sont 0 ou 1.<br><br/>- **0 (désactivé, valeur par défaut)**  : Chiffre uniquement les informations d’authentification.<br/>- **1 (activé)**  : Chiffre toutes les données transmises entre le pilote et la base de données. | `0` |
| CharacterSet | Jeu de caractères à utiliser pour la session. Par exemple, `CharacterSet=UTF16`.<br><br/>Cette valeur peut être un jeu de caractères défini par l’utilisateur ou l’un des jeux de caractères prédéfinis suivants : <br/>- ASCII<br/>- UTF8<br/>- UTF16<br/>- LATIN1252_0A<br/>- LATIN9_0A<br/>- LATIN1_0A<br/>- Shift-JIS (Windows, compatible DOS, KANJISJIS_0S)<br/>- EUC (compatible Unix, KANJIEC_0U)<br/>- Mainframe IBM (KANJIEBCDIC5035_0I)<br/>- KANJI932_1S0<br/>- BIG5 (TCHBIG5_1R0)<br/>- GB (SCHGB2312_1T0)<br/>- SCHINESE936_6R0<br/>- TCHINESE950_8R0<br/>- NetworkKorean (HANGULKSC5601_2R4)<br/>- HANGUL949_7R0<br/>- ARABIC1256_6A0<br/>- CYRILLIC1251_2A0<br/>- HEBREW1255_5A0<br/>- LATIN1250_1A0<br/>- LATIN1254_7A0<br/>- LATIN1258_8A0<br/>- THAI874_4A0 | `ASCII` |
| MaxRespSize |Taille maximale de la mémoire tampon de réponse pour les requêtes SQL en Ko (kilo-octets). Par exemple, `MaxRespSize=‭10485760‬`.<br/><br/>Pour la version de base de données Teradata 16.00 (ou ultérieure), la valeur maximale est 7361536. Pour les connexions qui utilisent des versions antérieures, la valeur maximale est 1048576. | `65536` |
| MechanismName | Pour utiliser le protocole LDAP afin d’authentifier la connexion, spécifiez `MechanismName=LDAP`. | N/A |

**Exemple : utilisation de l’authentification de base**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>;Uid=<username>;Pwd=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple : utilisation d’une authentification Windows**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>",
            "username": "<username>",
            "password": "<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple d’utilisation de l’authentification LDAP**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "connectionString": "DBCName=<server>;MechanismName=LDAP;Uid=<username>;Pwd=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

> [!NOTE]
>
> La charge utile suivante est toujours prise en charge. Cependant, nous vous invitons à utiliser la nouvelle charge utile dorénavant.

**Charge utile précédente :**

```json
{
    "name": "TeradataLinkedService",
    "properties": {
        "type": "Teradata",
        "typeProperties": {
            "server": "<server>",
            "authenticationType": "<Basic/Windows>",
            "username": "<username>",
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

## <a name="dataset-properties"></a>Propriétés du jeu de données

Cette section fournit une liste des propriétés prises en charge par le jeu de données Teradata. Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md).

Pour copier des données à partir de Teradata, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur `TeradataTable`. | Oui |
| database | Nom de l’instance Teradata. | Non (si « query » dans la source de l’activité est spécifié) |
| table | Nom de la table dans l’instance Teradata. | Non (si « query » dans la source de l’activité est spécifié) |

**Exemple :**

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "TeradataTable",
        "typeProperties": {},
        "schema": [],        
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

> [!NOTE]
>
> Le jeu de données de type `RelationalTable` est toujours pris en charge. Toutefois, nous vous recommandons d’utiliser le nouveau jeu de données.

**Charge utile précédente :**

```json
{
    "name": "TeradataDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Teradata linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Cette section fournit une liste des propriétés prises en charge par la source Teradata. Pour obtenir la liste complète des sections et propriétés disponibles pour la définition des activités, consultez [Pipelines](concepts-pipelines-activities.md). 

### <a name="teradata-as-source"></a>Teradata en tant que source

>[!TIP]
>Pour charger efficacement des données à partir de Teradata à l'aide du partitionnement des données, consultez la section [Copie en parallèle à partir de Teradata](#parallel-copy-from-teradata).

Pour copier des données à partir de Teradata, les propriétés prises en charge dans la section **source** de l'activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source de l’activité de copie doit être définie sur `TeradataSource`. | Oui |
| query | Utiliser la requête SQL personnalisée pour lire les données. par exemple `"SELECT * FROM MyTable"`.<br>Lorsque vous activez la charge partitionnée, vous devez utiliser les paramètres de partition intégrés correspondants dans votre requête. Pour consulter des exemples, voir [Copie en parallèle à partir de Teradata](#parallel-copy-from-teradata). | Non (si la table du jeu de données est spécifiée) |
| partitionOptions | Spécifie les options de partitionnement des données utilisées pour charger des données à partir de Teradata. <br>Les valeurs autorisées sont : **Aucun** (par défaut), **Hachage** et **DynamicRange**.<br>Lorsqu’une option de partition est activée (autrement dit, pas `None`), le degré de parallélisme pour charger simultanément des données à partir de Teradata est contrôlé par le paramètre [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de l’activité de copie. | Non |
| partitionSettings | Spécifiez le groupe de paramètres pour le partitionnement des données. <br>S’applique lorsque l’option de partitionnement n’est pas `None`. | Non |
| partitionColumnName | Spécifiez le nom de la colonne source qu’utilisera le partitionnement par plages de valeurs ou hachage pour la copie en parallèle. S’il n’est pas spécifié, l’index primaire de la table sera automatiquement détecté et utilisé en tant que colonne de partition. <br>S’applique lorsque l’option de partitionnement est `Hash` ou `DynamicRange`. Si vous utilisez une requête pour récupérer des données sources, utilisez `?AdfHashPartitionCondition` ou `?AdfRangePartitionColumnName` dans la clause WHERE. Consultez l’exemple de la section [Copie en parallèle à partir de Teradata](#parallel-copy-from-teradata). | Non |
| partitionUpperBound | Valeur maximale de la colonne de partition à partir de laquelle copier des données. <br>S’applique lorsque de l’option de partition est `DynamicRange`. Si vous avez recours à une requête pour récupérer des données sources, utilisez `?AdfRangePartitionUpbound` dans la clause WHERE. Pour consulter un exemple, voir [Copie en parallèle à partir de Teradata](#parallel-copy-from-teradata). | Non |
| partitionLowerBound | Valeur minimale de la colonne de partition à partir de laquelle copier des données. <br>S’applique lorsque l’option de partitionnement est `DynamicRange`. Si vous utilisez une requête pour récupérer des données sources, utilisez `?AdfRangePartitionLowbound` dans la clause WHERE. Pour consulter un exemple, voir [Copie en parallèle à partir de Teradata](#parallel-copy-from-teradata). | Non |

> [!NOTE]
>
> La source de copie de type `RelationalSource` est toujours prise en charge, mais ne gère pas la nouvelle charge en parallèle intégrée à partir de Teradata (options de partition). Toutefois, nous vous recommandons d’utiliser le nouveau jeu de données.

**Exemple : copie de données à l’aide de la requête de base sans partition**

```json
"activities":[
    {
        "name": "CopyFromTeradata",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Teradata input dataset name>",
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
                "type": "TeradataSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="parallel-copy-from-teradata"></a>Copie en parallèle à partir de Teradata

Le connecteur Teradata de Data Factory propose un partitionnement de données intégré pour copier des données à partir de Teradata, en parallèle. Vous trouverez des options de partitionnement de données dans la table **Source** de l’activité de copie.

![Capture d’écran représentant les options de partition](./media/connector-teradata/connector-teradata-partition-options.png)

Lorsque vous activez la copie partitionnée, Data Factory exécute des requêtes en parallèle sur votre source Teradata, afin de charger des données par partitions. Le degré de parallélisme est contrôlé via le paramètre [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) sur l’activité de copie. Par exemple, si vous définissez `parallelCopies` sur quatre, Data Factory génère et exécute simultanément quatre requêtes basées l'option de partition et les paramètres que vous avez spécifiés, chacune récupérant des données à partir de Teradata.

Il vous est recommandé d’activer la copie en parallèle avec partitionnement des données notamment lorsque vous chargez une grande quantité de données à partir de Teradata. Voici quelques suggestions de configurations pour différents scénarios. Lors de la copie de données dans un magasin de données basé sur des fichiers, il est recommandé de les écrire dans un dossier sous la forme de plusieurs fichiers (spécifiez uniquement le nom du dossier). Les performances seront meilleures qu’avec l’écriture dans un seul fichier.

| Scénario                                                     | Paramètres suggérés                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Chargement complet à partir d’une table volumineuse.                                   | **Option de partition** : Hachage. <br><br/>Lors de l’exécution, Data Factory détecte automatiquement la colonne d’index principale, y applique un hachage et copie les données par partitions. |
| Chargez une grande quantité de données à l’aide d’une requête personnalisée.                 | **Option de partition** : Hachage.<br>**Requête**: `SELECT * FROM <TABLENAME> WHERE ?AdfHashPartitionCondition AND <your_additional_where_clause>`.<br>**Colonne de partition** : Spécifiez la colonne utilisée pour appliquer la partition par hachage. Si cette colonne n’est pas spécifiée, Data Factory détecte automatiquement la colonne PK de la table que vous avez spécifiée dans le jeu de données Teradata.<br><br>Lors de l’exécution, Data Factory remplace `?AdfHashPartitionCondition` par la logique de partition de hachage et l’envoie à Teradata. |
| Chargez une grande quantité de données à l’aide d’une requête personnalisée, qui dispose d’une colonne d’entiers avec valeur uniformément distribuée pour le partitionnement par plages de valeurs. | **Options de partition** : Partition dynamique par spécification de plages de valeurs.<br>**Requête**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Colonne de partition** : Spécifiez la colonne utilisée pour partitionner les données. Vous pouvez procéder au partitionnement par rapport à la colonne avec le type de données entier.<br>**Limite supérieure de partition** et **limite inférieure de partition** : Indiquez si vous souhaitez filtrer le contenu par rapport à la colonne de partition pour récupérer uniquement les données entre les plages inférieure et supérieure.<br><br>Lors de l’exécution, Data Factory remplace `?AdfRangePartitionColumnName`, `?AdfRangePartitionUpbound` et `?AdfRangePartitionLowbound` par le nom réel de la colonne et les plages de valeurs de chaque partition, et les envoie à Teradata. <br>Par exemple, si l’ID de la colonne de partition « ID » est défini sur une limite inférieure de 1 et une limite supérieure de 80, avec une copie en parallèle définie sur 4, Data Factory récupère les données via 4 partitions. Les ID sont inclus entre [1,20], [21, 40], [41, 60] et [61, 80], respectivement. |

**Exemple : requête avec partition par hachage**

```json
"source": {
    "type": "TeradataSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfHashPartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "Hash",
    "partitionSettings": {
        "partitionColumnName": "<hash_partition_column_name>"
    }
}
```

**Exemple : requête avec partition dynamique par spécification de plages de valeurs**

```json
"source": {
    "type": "TeradataSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<dynamic_range_partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="data-type-mapping-for-teradata"></a>Mappage de type de données pour Teradata

Lorsque vous copiez des données à partir de Teradata, les mappages suivants s’appliquent. Pour découvrir comment l’activité de copie mappe le schéma et le type de données la source au récepteur, consultez [Mappage de schéma dans l’activité de copie](copy-activity-schema-and-type-mapping.md).

| Type de données Teradata | Type de données intermédiaires d’Azure Data Factory |
|:--- |:--- |
| BigInt |Int64 |
| Objet blob |Byte[] |
| Byte |Byte[] |
| ByteInt |Int16 |
| Char |String |
| Clob |String |
| Date |DateTime |
| Decimal |Decimal |
| Double |Double |
| Graphic |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Integer |Int32 |
| Interval Day |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Day To Hour |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Day To Minute |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Day To Second |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Hour |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Hour To Minute |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Hour To Second |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Minute |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Minute To Second |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Month |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Second |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Year |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Interval Year To Month |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Number |Double |
| Période (date) |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Période (heure) |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Période (heure avec fuseau horaire) |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Période (timestamp) |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Période (timestamp avec fuseau horaire) |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| SmallInt |Int16 |
| Temps |TimeSpan |
| Time With Time Zone |TimeSpan |
| Timestamp |DateTime |
| Timestamp With Time Zone |DateTime |
| VarByte |Byte[] |
| VarChar |String |
| VarGraphic |Non pris en charge. Appliquer un cast explicite dans la requête source. |
| Xml |Non pris en charge. Appliquer un cast explicite dans la requête source. |


## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
