---
title: Copier des données vers ou à partir d’Oracle avec Data Factory
description: Découvrez comment copier des données vers et à partir d’une base de données Oracle locale à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 3c20aa95-a8a1-4aae-9180-a6a16d64a109
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/15/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 1aa8708701af37834ae3b6cdc42de9c691ccacec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86084288"
---
# <a name="copy-data-to-or-from-oracle-on-premises-by-using-azure-data-factory"></a>Copier des données vers ou à partir d’une instance locale d’Oracle à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-onprem-oracle-connector.md)
> * [Version 2 (version actuelle)](../connector-oracle.md)

> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory. Si vous utilisez la version actuelle du service Azure Data Factory, consultez [Connecteur Oracle dans V2](../connector-oracle.md).


Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données vers ou à partir d’une base de données Oracle locale. Il s’appuie sur l’article relatif aux [activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement des données avec l’activité de copie.

## <a name="supported-scenarios"></a>Scénarios pris en charge

Vous pouvez copier des données *d’une base de données Oracle* vers les magasins de données suivants :

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

Vous pouvez copier des données des magasins de données suivants *vers une base de données Oracle* :

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="prerequisites"></a>Prérequis

Data Factory prend en charge la connexion à des sources Oracle locales à l’aide de la passerelle de gestion des données. Pour plus d’informations sur la passerelle de gestion des données, consultez [Passerelle de gestion des données](data-factory-data-management-gateway.md). Pour obtenir des instructions détaillées sur la configuration de la passerelle dans un pipeline de données en vue de déplacer des données, consultez l’article [Déplacer les données d’une source locale vers le cloud](data-factory-move-data-between-onprem-and-cloud.md).

Une passerelle est nécessaire, même si la base de données Oracle est hébergée sur une machine virtuelle Azure IaaS. Vous pouvez installer la passerelle sur la même machine virtuelle IaaS que le magasin de données, ou sur une autre machine virtuelle, pourvu que la passerelle puisse se connecter à la base de données.

> [!NOTE]
> Pour obtenir des conseils sur la résolution des problèmes de connexion ou de passerelle, consultez [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).

## <a name="supported-versions-and-installation"></a>Versions prises en charge et installation

Ce connecteur Oracle prend en charge deux versions de pilotes :

- **Pilote Microsoft pour Oracle (recommandé)**  : À compter de la version 2.7 de la passerelle de gestion des données, le pilote Microsoft pour Oracle est installé automatiquement avec la passerelle. Vous n’avez pas besoin d’installer ou de mettre à jour le pilote pour établir une connectivité à Oracle. Avec ce pilote, vous pouvez également constater de meilleures performances de copie. Voici les versions prises en charge des bases de données Oracle :
  - Oracle 12c R1 (12.1)
  - Oracle 11g R1, R2 (11.1, 11.2)
  - Oracle 10g R1, R2 (10.1, 10.2)
  - Oracle 9i R1, R2 (9.0.1, 9.2)
  - Oracle 8i R3 (8.1.7)

    > [!NOTE]
    > Le serveur proxy Oracle n’est pas pris en charge.

    > [!IMPORTANT]
    > Le pilote Microsoft pour Oracle prend uniquement en charge la copie de données à partir d’Oracle. Le pilote ne prend pas en charge l’écriture de données dans Oracle. Ce pilote n’est pas pris en charge par la fonctionnalité de connexion de test sous l’onglet **Diagnostics** de la passerelle de gestion des données. Cependant, vous avez la possibilité de valider la connectivité à l’aide de l’Assistant Copie.
    >

- **Fournisseur de données Oracle pour .NET** : Vous pouvez utiliser le fournisseur de données Oracle pour copier des données vers ou à partir d’Oracle. Ce composant est inclus dans [Oracle Data Access Components for Windows](https://www.oracle.com/technetwork/topics/dotnet/downloads/). Installez la version appropriée (32 ou 64 bits) sur l’ordinateur où est installée la passerelle. Le [Fournisseur de données Oracle pour .NET 12.1](https://docs.oracle.com/database/121/ODPNT/InstallSystemRequirements.htm#ODPNT149) peut accéder à Oracle Database 10g Release 2 ou versions ultérieures.

    Si vous sélectionnez **XCopy Installation** (Installation de XCopy), effectuez les étapes décrites dans le fichier readme.htm. Nous vous recommandons de sélectionner le programme d’installation qui comprend une interface utilisateur (et non le programme d’installation XCopy).

    Après avoir installé le fournisseur, redémarrez le service hôte de la passerelle de gestion des données sur votre ordinateur, à l’aide de l’applet Services ou du Gestionnaire de configuration de la passerelle de gestion des données.

Si vous utilisez l’Assistant Copie pour créer le pipeline de copie, le type de pilote sera déterminé automatiquement. Le pilote Microsoft est utilisé par défaut, sauf si votre version de la passerelle est antérieure à la version 2.7, ou si vous sélectionnez Oracle en tant que récepteur.

## <a name="get-started"></a>Bien démarrer

Vous pouvez créer un pipeline ayant une activité de copie. Le pipeline déplace les données vers ou à partir d’une base de données Oracle locale, à l’aide de différents outils ou API.

Le moyen le plus simple de créer un pipeline consiste à utiliser l’Assistant de copie. Voir le [tutoriel : Créer un pipeline à l’aide de l’Assistant Copie de données](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.

Vous pouvez également utiliser un des outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, un **modèle Azure Resource Manager**, l’**API .NET** ou l’**API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline ayant une activité de copie, consultez le [Tutoriel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, effectuez les étapes suivantes pour créer un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur :

1. Création d'une **fabrique de données**. Une fabrique de données peut contenir un ou plusieurs pipelines.
2. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données. Par exemple, si vous copiez des données d’une base de données Oracle vers le Stockage Blob Azure, créez deux services liés pour lier votre base de données Oracle et votre compte de stockage Azure à votre fabrique de données. Pour les propriétés du service lié qui sont propres à Oracle, consultez [Propriétés du service lié](#linked-service-properties).
3. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie. Dans l’exemple de l’étape précédente, vous créez un jeu de données pour spécifier la table de votre base de données Oracle qui doit contenir les données d’entrée. Créez un autre jeu de données pour spécifier le conteneur d’objets blob et le dossier qui contient les données copiées à partir de la base de données Oracle. Pour plus d’informations sur les propriétés de jeu de données qui sont propres à Oracle, consultez [Propriétés du jeu de données](#dataset-properties).
4. Créez un **pipeline** comprenant une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie. Dans l’exemple précédent, vous utilisez **OracleSource** comme source et **BlobSink** comme récepteur pour l’activité de copie. De la même façon, si vous copiez des données du Stockage Blob Azure vers une base de données Oracle, vous devez utiliser **BlobSource** et **OracleSink** dans l’activité de copie. Pour connaître les propriétés d’activité de copie qui sont propres à une base de données Oracle, consultez [Propriétés de l’activité de copie](#copy-activity-properties). Pour plus d’informations sur l’utilisation d’un magasin de données comme source ou comme récepteur, sélectionnez le lien de la section précédente correspondant à votre magasin de données.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées. Lorsque vous utilisez les outils ou API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory à l’aide du format JSON. Pour obtenir des exemples comportant des définitions JSON d’entités Data Factory utilisées dans le but de copier des données vers ou à partir d’une base de données Oracle locale, consultez Exemples JSON.

Les sections suivantes fournissent des informations sur les propriétés JSON qui sont utilisées pour définir les entités Data Factory.

## <a name="linked-service-properties"></a>Propriétés du service lié

Le tableau suivant décrit les éléments JSON qui sont propres au service lié Oracle.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété **type** doit être définie sur **OnPremisesOracle**. |Oui |
| driverType | Spécifiez le pilote à utiliser pour copier les données vers ou à partir de la base de données Oracle. Les valeurs autorisées sont **Microsoft** ou **ODP** (par défaut). Pour plus d’informations sur le pilote, consultez [Version prise en charge et installation](#supported-versions-and-installation). | Non |
| connectionString | Spécifiez les informations nécessaires pour se connecter à l’instance de base de données Oracle pour la propriété **connectionString**. | Oui |
| gatewayName | Nom de la passerelle utilisée pour se connecter au serveur Oracle local |Oui |

**Exemple : Utilisation du pilote Microsoft**

> [!TIP]
> Si vous voyez l’erreur « ORA-01025: UPI parameter out of range » et que votre version d’Oracle est 8i, ajoutez `WireProtocolMode=1` à votre chaîne de connexion, puis réessayez :

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<service ID>;User Id=<user name>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Exemple : Utilisation du pilote ODP**

Pour plus d’informations sur les formats autorisés, consultez [Fournisseur de données Oracle pour .NET ODP](https://www.connectionstrings.com/oracle-data-provider-for-net-odp-net/).

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host name>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<service ID>))); User Id=<user name>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md).

Les sections d’un fichier JSON de jeu de données, comme la structure, la disponibilité et la stratégie, sont similaires pour tous les types de jeux de données (par exemple, Oracle, Stockage Blob Azure et Table de stockage Azure).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section **typeProperties** du jeu de données de type **OracleTable** a les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans la base de données Oracle à laquelle le service lié fait référence. |Non (si **oracleReaderQuery** ou **OracleSource** est spécifié) |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés permettant de définir des activités, consultez [Création de pipelines](data-factory-create-pipelines.md).

Les propriétés comme le nom, la description, les tables d’entrée et de sortie, et la stratégie, sont disponibles pour tous les types d’activités.

> [!NOTE]
> L’activité de copie accepte uniquement une entrée et produit une seule sortie.

Les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Les propriétés de l’activité de copie dépendent du type de la source et du récepteur.

### <a name="oraclesource"></a>OracleSource

Dans le cas d’une activité de copie, quand la source est de type **OracleSource**, les propriétés suivantes sont disponibles dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| oracleReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : « select \* from **MyTable** ». <br/><br/>Si non spécifié, l’instruction SQL est exécutée : « select \* from **MyTable** ». |Non<br />(si **tableName** de **dataset** est spécifié) |

### <a name="oraclesink"></a>OracleSink

**OracleSink** prend en charge les propriétés suivantes :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| writeBatchTimeout |Temps d’attente avant expiration de l’opération d’insertion de lot. |**timespan**<br/><br/> Exemple : 00:30:00 (30 minutes) |Non |
| writeBatchSize |Insère des données dans la table SQL quand la taille de la mémoire tampon atteint la valeur de **writeBatchSize**. |Nombre entier (nombre de lignes) |Non (valeur par défaut : 100) |
| sqlWriterCleanupScript |Spécifie une requête pour exécuter l’activité de copie afin que les données d’un segment spécifique soient nettoyées. |Une instruction de requête. |Non |
| sliceIdentifierColumnName |Spécifie le nom de la colonne qui doit être remplie avec un identificateur de segment généré automatiquement pour l’activité de copie. La valeur de **sliceIdentifierColumnName** est utilisée pour nettoyer les données d’un segment quand celui-ci est réexécuté. |Nom d’une colonne avec le type de données **binary(32)** . |Non |

## <a name="json-examples-for-copying-data-to-and-from-the-oracle-database"></a>Exemples JSON pour copier des données vers et à partir de la base de données Oracle

Les exemples suivants présentent des exemples de définitions JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données vers ou à partir d’une base de données Oracle ou du Stockage Blob Azure. Toutefois, il est possible de copier des données dans l’un des récepteurs répertoriés dans [Magasins et formats de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), à l’aide de l’activité de copie dans Azure Data Factory.

**Exemple : Copie de données d’Oracle vers Stockage Blob Azure**

L’exemple contient les entités Data Factory suivantes :

* Un service lié de type [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties).
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties).
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [OracleSource](data-factory-onprem-oracle-connector.md#copy-activity-properties) comme source et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) comme récepteur.

L’exemple copie toutes les heures les données d’une table d’une base de données Oracle locale vers un objet blob. Pour plus d’informations sur les propriétés qui sont utilisées dans les exemples, consultez les sections qui suivent chaque exemple.

**Service lié Oracle**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString":"Host=<host>;Port=<port>;Sid=<service ID>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Service lié Azure Blob Storage**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

**Jeu de données d’entrée Oracle**

L’exemple suppose que vous avez créé une table nommée **MyTable** dans Oracle. Il contient une colonne appelée **timestampcolumn** pour les données de série chronologique.

Définition de **external** : la valeur **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory, et non produit par une activité dans Data Factory.

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2014-02-27T12:00:00",
            "frequency": "Hour"
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

**Jeu de données de sortie d'objet Blob Azure**

Les données sont écrites dans un nouvel objet blob toutes les heures (**fréquence** : **heure**, **intervalle** : **1**). Le chemin du dossier et le nom de fichier de l’objet blob sont évalués dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin du dossier utilise l’année, le mois, le jour et l’heure de début.

```json
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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
            ],
            "format": {
                "type": "TextFormat",
                "columnDelimiter": "\t",
                "rowDelimiter": "\n"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Pipeline avec une activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie, et qui est planifiée pour s’exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **OracleSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **oracleReaderQuery** sélectionne les données de la dernière heure qui sont à copier.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for a copy activity",
        "activities":[
            {
                "name": "OracletoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": " OracleInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "OracleSource",
                        "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

**Exemple : Copie de données de Stockage Blob Azure vers Oracle**

Cet exemple indique comment copier des données d’un compte de stockage d’objets blob Azure vers une base de données Oracle locale. Toutefois, vous pouvez copier les données *directement* dans l’une des sources répertoriées dans [Magasins et formats de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), à l’aide de l’activité de copie dans Azure Data Factory.

L’exemple contient les entités Data Factory suivantes :

* Un service lié de type [OnPremisesOracle](data-factory-onprem-oracle-connector.md#linked-service-properties).
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [OracleTable](data-factory-onprem-oracle-connector.md#dataset-properties).
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) comme source et [OracleSink](data-factory-onprem-oracle-connector.md#copy-activity-properties) comme récepteur.

L’exemple copie chaque heure les données d’une objet blob vers une table d’une base de données Oracle locale. Pour plus d’informations sur les propriétés qui sont utilisées dans les exemples, consultez les sections qui suivent chaque exemple.

**Service lié Oracle**

```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "connectionString": "Data Source=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<host name>)(PORT=<port number>))(CONNECT_DATA=(SERVICE_NAME=<service ID>)));
            User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

**Service lié Azure Blob Storage**

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

**Jeu de données d'entrée d'objet Blob Azure**

Les données sont récupérées depuis un nouvel objet blob toutes les heures (**fréquence** : **heure**, **intervalle** : **1**). Le chemin du dossier et le nom de fichier de l’objet blob sont évalués dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin du dossier utilise l’année, le mois et le jour de début. Le nom de fichier utilise la partie Heure du début. Définition de **external** : la valeur **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory, et non produit par une activité dans Data Factory.

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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
                }
            ],
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

**Jeu de données de sortie Oracle**

L’exemple suppose que vous avez créé une table nommée **MyTable** dans Oracle. Créez la table dans Oracle avec le nombre de colonnes que le fichier CSV d’objets blob doit contenir. De nouvelles lignes sont ajoutées à la table toutes les heures.

```json
{
    "name": "OracleOutput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Day",
            "interval": "1"
        }
    }
}
```

**Pipeline avec une activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie, et qui est planifiée pour s’exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **BlobSource** et le type **sink** est défini sur **OracleSink**.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-05T19:00:00",
        "description":"pipeline with a copy activity",
        "activities":[
            {
                "name": "AzureBlobtoOracle",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "OracleOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "OracleSink"
                    }
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```


## <a name="troubleshooting-tips"></a>Conseils de dépannage

### <a name="problem-1-net-framework-data-provider"></a>Problème 1 : Fournisseur de données .NET Framework

**Message d’erreur**

```text
Copy activity met invalid parameters: 'UnknownParameterName', Detailed message: Unable to find the requested .NET Framework Data Provider. It may not be installed.
```

**Causes possibles**

* Le fournisseur de données .NET Framework pour Oracle n’a pas été installé.
* Le fournisseur de données .NET Framework pour Oracle a été installé pour .NET Framework 2.0 et est introuvable dans les dossiers de .NET Framework 4.0.

**Résolution :**

* Si vous n’avez pas installé le fournisseur .NET pour Oracle, [installez-le](https://www.oracle.com/technetwork/topics/dotnet/downloads/), puis réessayez.
* Si vous obtenez le message d’erreur, même après l’installation du fournisseur, effectuez les étapes suivantes :
    1. Ouvrez le fichier machine.config de .NET 2.0 dans le dossier : <disque système\>:\Windows\Microsoft.NET\Framework64\v2.0.50727\CONFIG\machine.config.
    2. Recherchez le **Fournisseur de données Oracle pour .NET**. Vous devez trouver une entrée comme celle de l’exemple suivant sous **system.data** > **DbProviderFactories** : `<add name="Oracle Data Provider for .NET" invariant="Oracle.DataAccess.Client" description="Oracle Data Provider for .NET" type="Oracle.DataAccess.Client.OracleClientFactory, Oracle.DataAccess, Version=2.112.3.0, Culture=neutral, PublicKeyToken=89b483f429c47342" />`
* Copiez cette entrée dans le fichier machine.config qui est situé dans le dossier .NET 4.0 suivant : <disque système\>:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config. Ensuite, remplacez la version par 4.xxx.x.x.
* Installez <Chemin d’installation d’ODP.NET\>\11.2.0\client_1\odp.net\bin\4\Oracle.DataAccess.dll dans le Global Assembly Cache (GAC) en exécutant **gacutil /i [chemin du fournisseur]** .

### <a name="problem-2-datetime-formatting"></a>Problème 2 : Mise en forme de la date et de l’heure

**Message d’erreur**

```text
Message=Operation failed in Oracle Database with the following error: 'ORA-01861: literal does not match format string'.,Source=,''Type=Oracle.DataAccess.Client.OracleException,Message=ORA-01861: literal does not match format string,Source=Oracle Data Provider for .NET,'.
```

**Résolution :**

Vous devez ajuster la chaîne de requête en fonction de la configuration des dates dans votre base de données Oracle. Voici un exemple (utilisant la fonction **to_date**) :

```console   
"oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= to_date(\\'{0:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') AND timestampcolumn < to_date(\\'{1:MM-dd-yyyy HH:mm}\\',\\'MM/DD/YYYY HH24:MI\\') ', WindowStart, WindowEnd)"
```


## <a name="type-mapping-for-oracle"></a>Mappage de type pour Oracle

Comme mentionné dans [Activités de déplacement de données](data-factory-data-movement-activities.md), l’activité de copie convertit automatiquement les types source en types récepteur à l’aide de l’approche en deux étapes suivante :

1. Conversion de types sources natifs en types .NET
2. Conversion d’un type .NET en type récepteur natif

Quand vous déplacez des données à partir d’Oracle, les mappages suivants sont utilisés pour convertir le type de données Oracle en type .NET, et vice versa :

| Type de données Oracle | Type de données .NET Framework. |
| --- | --- |
| BFILE |Byte[] |
| BLOB |Byte[]<br/>(uniquement pris en charge sur Oracle 10g et versions ultérieures lorsque vous utilisez le pilote Microsoft) |
| CHAR |String |
| CLOB |String |
| DATE |DateTime |
| FLOAT |Décimale, chaîne (si précision > 28) |
| INTEGER |Décimale, chaîne (si précision > 28) |
| INTERVAL YEAR TO MONTH |Int32 |
| INTERVAL DAY TO SECOND |TimeSpan |
| LONG |String |
| LONG RAW |Byte[] |
| NCHAR |String |
| NCLOB |String |
| NUMBER |Décimale, chaîne (si précision > 28) |
| NVARCHAR2 |String |
| RAW |Byte[] |
| ROWID |String |
| timestamp |DateTime |
| TIMESTAMP WITH LOCAL TIME ZONE |DateTime |
| TIMESTAMP WITH TIME ZONE |DateTime |
| UNSIGNED INTEGER |Number |
| VARCHAR2 |String |
| XML |String |

> [!NOTE]
> Les types de données **INTERVAL YEAR TO MONTH** et **INTERVAL DAY TO SECOND** ne sont pas pris en charge lors de l’utilisation du pilote Microsoft.

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur

Pour savoir comment mapper des colonnes du jeu de données source à des colonnes du jeu de données récepteur, consultez [Mappage des colonnes d’un jeu de données dans Data Factory](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Lecture renouvelée de sources relationnelles

Lorsque vous copiez des données à partir d’un magasin de données relationnel, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter un segment manuellement. Vous pouvez également configurer une stratégie de nouvelles tentatives pour un jeu de données, afin qu’un segment soit réexécuté en cas de défaillance. Lorsqu’un segment est réexécuté, que ce soit manuellement ou à l’aide d’une stratégie de nouvelles tentatives, vous devez vous assurer que les mêmes données sont lues et ce, quel que soit le nombre d’exécutions du segment. Pour plus d’informations, consultez [Lecture renouvelée de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Performances et réglage

Pour plus d’informations sur les facteurs clés affectant les performances du déplacement de données dans Azure Data Factory (activité de copie), consultez le [Guide des performances et de l’optimisation de l’activité de copie](data-factory-copy-activity-performance.md). Vous pouvez également découvrir les différentes façons d’optimiser la copie de données.
