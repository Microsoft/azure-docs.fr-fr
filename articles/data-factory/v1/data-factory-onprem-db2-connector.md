---
title: Déplacer des données de DB2 à l’aide d’Azure Data Factory
description: Découvrez comment déplacer des données depuis une base de données DB2 locale à l’aide de l’activité de copie dans Azure Data Factory
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: c1644e17-4560-46bb-bf3c-b923126671f1
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: e5d2c6b0460c3a7566adb17601aceb57e57f4d0b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001063"
---
# <a name="move-data-from-db2-by-using-azure-data-factory-copy-activity"></a>Déplacer des données depuis DB2 à l’aide de l’activité de copie dans Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-onprem-db2-connector.md)
> * [Version 2 (version actuelle)](../connector-db2.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur DB2 dans V2](../connector-db2.md).


Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données d’une base de données DB2 locale dans un autre magasin de données. Vous pouvez copier des données dans n’importe quel magasin figurant dans la liste des récepteurs pris en charge que vous trouverez dans l’article sur les [activités de déplacement de données dans Data Factory](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Cette rubrique s’appuie sur l’article concernant Data Factory, qui présente une vue d’ensemble du déplacement de données à l’aide de l’activité de copie et fournit une liste des combinaisons de magasins de données prises en charge. 

Actuellement, Data Factory prend uniquement en charge le déplacement de données depuis une base de données DB2 vers un [magasin de données récepteur pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats), et non le déplacement de données depuis d’autres magasins de données vers une base de données DB2.

## <a name="prerequisites"></a>Prérequis
Data Factory prend en charge la connexion à une base de données DB2 locale à l’aide de la [passerelle de gestion des données](data-factory-data-management-gateway.md). Pour obtenir des instructions détaillées sur la configuration du pipeline de données de la passerelle pour déplacer des données, consultez l’article [Déplacement de données entre des sources locales et le cloud](data-factory-move-data-between-onprem-and-cloud.md).

Une passerelle est requise même si la base de données DB2 est hébergée sur une machine virtuelle Azure IaaS. Vous pouvez installer la passerelle sur la même machine virtuelle IaaS que le magasin de données, ou sur une autre machine virtuelle pourvu que la passerelle puisse se connecter à la base de données.

La passerelle de gestion des données fournit un pilote DB2 intégré. Par conséquent, vous n’avez pas besoin d’installer manuellement de pilote lors de la copie de données depuis DB2.

> [!NOTE]
> Pour obtenir des conseils sur la résolution des problèmes de connexion/passerelle, consultez l’article [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues).


## <a name="supported-versions"></a>Versions prises en charge
Le connecteur DB2 Data Factory prend en charge les plateformes et versions IBM DB2 suivantes avec les versions 9, 10 et 11 de SQL Access Manager Distributed Relational Database Architecture (DRDA) :

* IBM DB2 pour z/OS version 11.1
* IBM DB2 pour z/OS version 10.1
* IBM DB2 pour i (AS400) version 7.2
* IBM DB2 pour i (AS400) version 7.1
* IBM DB2 pour Linux, UNIX et Windows (LUW) version 11
* IBM DB2 pour LUW version 10.5
* IBM DB2 pour LUW version 10.1

> [!TIP]
> Si vous recevez le message d’erreur « Le package correspondant à une requête d’exécution d’instruction SQL est introuvable. SQLSTATE = 51002 SQLCODE =-805 », la raison en est qu’un package nécessaire n’est pas créé pour l’utilisateur normal sur le système d’exploitation. Pour résoudre le problème, suivez ces instructions en fonction de votre type de serveur DB2 :
> - DB2 pour i (AS400) : demandez à un utilisateur chevronné de créer la collection pour l’utilisateur normal avant d’exécuter l’activité de copie. Pour créer la collection, utilisez la commande : `create collection <username>`
> - DB2 pour z/OS ou LUW : utilisez un compte doté de privilèges élevés (utilisateur chevronné ou administrateur disposant d’autorités de package et d’autorisations BIND, BINDADD, GRANT EXECUTE TO PUBLIC) pour exécuter une fois l’activité de copie. Le package nécessaire est automatiquement créé au cours de la copie. Vous pouvez ensuite revenir au mode utilisateur normal pour vos copies suivantes.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie pour déplacer des données d’un magasin de données DB2 local à l’aide de différents outils et API : 

- Le moyen le plus simple de créer un pipeline consiste à utiliser l’Assistant de copie Azure Data Factory. Pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant de copie, consultez la page [Didacticiel : Créer un pipeline à l’aide de l’Assistant de copie](data-factory-copy-data-wizard-tutorial.md). 
- Vous pouvez également utiliser des outils pour créer un pipeline, notamment Visual Studio, Azure PowerShell, un modèle Azure Resource Manager, l’API .NET et l’API REST. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). 

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de services liés pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de jeux de données pour représenter les données d’entrée et de sortie de l’opération de copie. 
3. Création d’un pipeline avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie. 

Lorsque vous utilisez l’Assistant de copie, les définitions JSON des entités de services liés, jeux de données et pipeline Data Factory sont automatiquement créées pour vous. Lorsque vous utilisez des outils ou API (à l’exception de l’API .NET), vous devez définir les entités Data Factory à l’aide du format JSON. L’exemple JSON : la copie des données de DB2 vers Stockage Azure Blob affiche les définitions JSON des entités Data Factory utilisées pour copier des données à partir d'un magasin de données DB2 local.

Les sections suivantes contiennent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres à un magasin de données DB2.

## <a name="db2-linked-service-properties"></a>Propriétés du service lié DB2
Le tableau suivant répertorie les propriétés JSON spécifiques d’un service lié DB2.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **type** |Cette propriété doit être définie sur **OnPremisesDb2**. |Oui |
| **server** |Nom du serveur DB2. |Oui |
| **database** |Nom de la base de données DB2. |Oui |
| **schema** |Nom du schéma dans la base de données DB2. Cette propriété est sensible à la casse. |Non |
| **authenticationType** |Type d'authentification utilisé pour se connecter à la base de données DB2. Les valeurs possibles sont les suivantes : Anonymous, Basic et Windows. |Oui |
| **username** |Nom du compte d’utilisateur si vous utilisez l’authentification de base ou Windows. |Non |
| **mot de passe** |Mot de passe du compte d’utilisateur. |Non |
| **gatewayName** |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données DB2 locale. |Oui |

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la **structure**, la **disponibilité** et la **stratégie** d’un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, stockage Azure Blob, stockage Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section **typeProperties** pour un jeu de données de type **RelationalTable** (qui inclut le jeu de données DB2) a la propriété suivante :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **tableName** |Nom de la table dans l'instance de base de données DB2 à laquelle le service lié fait référence. Cette propriété est sensible à la casse. |Non (si la propriété **query** d’une activité de copie de type **RelationalSource** est spécifiée) |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste des sections et des propriétés disponibles pour la définition des activités de copie, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés d’activité de copie comme le **nom**, la **description**, la table d’**entrée**, la table de **sortie** et la **stratégie** sont disponibles pour tous les types d’activités. Les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Pour l’activité de copie, lorsque la source est de type **RelationalSource** (qui inclut DB2), les propriétés suivantes sont disponibles dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| **query** |Utilise la requête personnalisée pour lire les données. |Chaîne de requête SQL. Par exemple : `"query": "select * from "MySchema"."MyTable""` |Non (si la propriété **tableName** d’un jeu de données est spécifié) |

> [!NOTE]
> Les noms de schéma et de table respectent la casse. Dans l’instruction de requête, mettez les noms de propriétés entre "" (guillemets doubles).

## <a name="json-example-copy-data-from-db2-to-azure-blob-storage"></a>Exemple JSON : copier des données de DB2 vers Stockage Blob Azure
Cet exemple présente des exemples de définition JSON, que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). L’exemple montre comment copier des données depuis une base de données DB2 dans le stockage Blob. Les données peuvent toutefois être copiées dans [tout magasin de données récepteur pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) à l’aide de l’activité de copie dans Azure Data Factory.

L’exemple contient les entités Data Factory suivantes :

- Un service lié DB2 de type [OnPremisesDb2](data-factory-onprem-db2-connector.md)
- Un service lié de stockage Azure Blob de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
- Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [RelationalTable](data-factory-onprem-db2-connector.md#dataset-properties)
- Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties)
- Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise les propriétés [RelationalSource](data-factory-onprem-db2-connector.md#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)

L'exemple copie toutes les heures les données de résultat d’une requête de base de données DB2 vers un objet Blob Azure. Les propriétés JSON utilisées dans l’exemple sont décrites dans les sections suivant les définitions des entités.

Pour commencer, installez et configurez une passerelle de données. Les instructions se trouvent dans l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md).

**Service lié DB2**

```json
{
    "name": "OnPremDb2LinkedService",
    "properties": {
        "type": "OnPremisesDb2",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

**Service lié Azure Blob Storage**

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorageLinkedService",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<AccountName>;AccountKey=<AccountKey>"
        }
    }
}
```

**Jeu de données d’entrée DB2**

L'exemple suppose que vous avez créé une table nommée « MyTable » dans DB2 qui contient une colonne appelée « timestamp » pour les données de série chronologique.

La propriété **external** est définie sur « true ». Ce paramètre informe le service Data Factory que ce jeu de données est externe à la fabrique de données et non produit par une activité dans la fabrique de données. Notez que la propriété **type** est définie sur **RelationalTable**.


```json
{
    "name": "Db2DataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremDb2LinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Les données sont écrites dans un nouvel objet blob toutes les heures en configurant la propriété **frequency** sur « Hour » et la propriété **interval** sur 1. La propriété **folderPath** de l’objet blob est évaluée dynamiquement en fonction de l’heure de début de la section en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et l’heure de l’heure de début.

```json
{
    "name": "AzureBlobDb2DataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/db2/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Pipeline pour l'activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie spécifiées, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition JSON du pipeline, le type **source** est défini sur **RelationalSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **query** sélectionne les données de la table « Orders ».

```json
{
    "name": "CopyDb2ToBlob",
    "properties": {
        "description": "pipeline for the copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "select * from \"Orders\""
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "inputs": [
                    {
                        "name": "Db2DataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobDb2DataSet"
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
                "name": "Db2ToBlob"
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```

## <a name="type-mapping-for-db2"></a>Mappage de type pour DB2
Comme mentionné dans l’article consacré aux [activités de déplacement des données](data-factory-data-movement-activities.md), l’activité de copie convertit automatiquement un type source en type récepteur à l’aide de l’approche en deux étapes suivante :

1. Conversion d’un type natif source en type .NET
2. Conversion d’un type .NET en type récepteur natif

Les mappages suivants sont utilisés lorsque l’activité de copie convertit les données de type DB2 en type .NET :

| Type de base de données DB2 | Type .NET Framework |
| --- | --- |
| SmallInt |Int16 |
| Integer |Int32 |
| BigInt |Int64 |
| Real |Unique |
| Double |Double |
| Float |Double |
| Decimal |Decimal |
| DecimalFloat |Decimal |
| Numérique |Decimal |
| Date |DateTime |
| Temps |TimeSpan |
| Timestamp |DateTime |
| Xml |Byte[] |
| Char |String |
| VarChar |String |
| LongVarChar |String |
| DB2DynArray |String |
| Binary |Byte[] |
| VarBinary |Byte[] |
| LongVarBinary |Byte[] |
| Graphic |String |
| VarGraphic |String |
| LongVarGraphic |String |
| Clob |String |
| Objet blob |Byte[] |
| DbClob |String |
| SmallInt |Int16 |
| Integer |Int32 |
| BigInt |Int64 |
| Real |Unique |
| Double |Double |
| Float |Double |
| Decimal |Decimal |
| DecimalFloat |Decimal |
| Numérique |Decimal |
| Date |DateTime |
| Temps |TimeSpan |
| Timestamp |DateTime |
| Xml |Byte[] |
| Char |String |

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur
Pour savoir comment mapper des colonnes du jeu de données source à des colonnes du jeu de données récepteur, consultez [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-reads-from-relational-sources"></a>Lectures renouvelées de sources relationnelles
Lorsque vous copiez des données à partir d’un magasin de données relationnel, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer la propriété de **stratégie** de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Assurez-vous que les mêmes données sont lues, quel que soit le nombre d’exécutions de la tranche et quelle que soit la manière dont vous la réexécutez. Pour plus d’informations, consultez [Lectures renouvelées de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Performances et réglage
Pour en savoir plus sur les facteurs clés affectant les performances de l’activité de copie et les différentes manières de les optimiser, consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md).
