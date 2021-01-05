---
title: Déplacer des données à partir d’un serveur FTP à l’aide d’Azure Data Factory
description: Découvrez comment déplacer des données depuis un serveur FTP à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: eea3bab0-a6e4-4045-ad44-9ce06229c718
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/02/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: eeeb122d240d8c3eae4ebe1650f67cf0e4b9dac6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001641"
---
# <a name="move-data-from-an-ftp-server-by-using-azure-data-factory"></a>Déplacer des données à partir d’un serveur FTP à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-ftp-connector.md)
> * [Version 2 (version actuelle)](../connector-ftp.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur FTP dans V2](../connector-ftp.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données à partir d’un serveur FTP. Il s’appuie sur l’article relatif aux [activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement des données avec l’activité de copie.

Vous pouvez copier les données d’un serveur FTP dans tout magasin de données récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que récepteurs par l’activité de copie, voir le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Actuellement, Data Factory prend uniquement en charge le déplacement de données d’un serveur FTP vers d’autres magasins de données, mais pas l’inverse. Il prend en charge à la fois les serveurs FTP locaux et cloud.

> [!NOTE]
> L’activité de copie ne supprime pas le fichier source une fois celui-ci copié vers la destination. Si vous devez supprimer le fichier source après copie, créez une activité personnalisée pour supprimer le fichier et utilisez l’activité dans le pipeline.

## <a name="enable-connectivity"></a>Activez la connectivité.
Si vous déplacez des données d’un serveur FTP **local** vers un magasin de données cloud (par exemple, Stockage Blob Azure), installez et utilisez la passerelle de gestion des données. La passerelle de gestion des données est un agent client qui est installé sur votre ordinateur local et permet aux services cloud de se connecter à une ressource locale. Pour plus de détails, voir [Passerelle de gestion des données](data-factory-data-management-gateway.md). Pour obtenir des instructions détaillées sur la configuration de la passerelle et son utilisation, voir [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md). Vous utilisez la passerelle pour vous connecter à un serveur FTP, même si le serveur est sur une machine virtuelle IaaS Azure.

Vous pouvez installer la passerelle sur le même ordinateur local ou sur la machine virtuelle IaaS Azure que le serveur FTP. Toutefois, nous vous recommandons d’installer la passerelle sur un ordinateur/une machine virtuelle IaaS distincts afin d’éviter les conflits de ressources, ainsi que pour obtenir de meilleures performances. Lorsque vous installez la passerelle sur un ordinateur distinct, l’ordinateur doit être en mesure au serveur FTP.

## <a name="get-started"></a>Bien démarrer
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’une source FTP à l’aide de différents outils ou API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie de Data Factory**. Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Data Factory Copy](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide.

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **PowerShell**, **modèle Azure Resource Manager**, **.NET API** et **REST API**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez les outils ou API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory à l’aide du format JSON. Pour un exemple contenant des définitions JSON pour les entités Data Factory servant à copier des données à partir d’un magasin de données FTP, consultez la section [Exemple JSON : copier des données d’un serveur FTP vers Stockage Blob Azure](#json-example-copy-data-from-ftp-server-to-azure-blob) de cet article.

> [!NOTE]
> Pour plus d’informations sur les formats de fichier et de compression pris en charge à utiliser, voir [Formats de fichier et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md).

Les sections suivantes fournissent des informations sur les propriétés JSON utilisées pour définir des entités Data Factory spécifiques pour FTP :

## <a name="linked-service-properties"></a>Propriétés du service lié
Le tableau suivant décrit les éléments JSON spécifiques pour un service FTP lié.

| Propriété | Description | Obligatoire | Default |
| --- | --- | --- | --- |
| type |Définissez ceci sur FtpServer. |Oui |&nbsp; |
| host |Spécifiez le nom ou l’adresse IP du serveur FTP. |Oui |&nbsp; |
| authenticationType |Spécifiez le type d’authentification. |Oui |Basic, anonyme |
| username |Spécifiez l’utilisateur ayant accès au serveur FTP. |Non |&nbsp; |
| mot de passe |Spécifiez le mot de passe de l’utilisateur (username). |Non |&nbsp; |
| encryptedCredential |Spécifiez les informations d’identification chiffrées pour accéder au serveur FTP. |Non |&nbsp; |
| gatewayName |Spécifiez le nom de la passerelle dans Passerelle de gestion des données pour la connexion à un serveur FTP local |Non |&nbsp; |
| port |Spécifiez le port sur lequel le serveur FTP écoute. |Non |21 |
| enableSsl |Indiquez si vous souhaitez utiliser FTP sur un canal SSL/TLS. |Non |true |
| enableServerCertificateValidation |Indiquez si vous souhaitez activer la validation des certificats TLS/SSL lors de l’utilisation de FTP sur un canal SSL/TLS. |Non |true |

>[!NOTE]
>Le connecteur FTP prend en charge l’accès au serveur FTP avec aucun chiffrement ou le chiffrement SSL/TLS explicite ; Il ne prend pas en charge le chiffrement SSL/TLS implicite.

### <a name="use-anonymous-authentication"></a>Utiliser une authentification anonyme

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "authenticationType": "Anonymous",
              "host": "myftpserver.com"
        }
    }
}
```

### <a name="use-username-and-password-in-plain-text-for-basic-authentication"></a>Utiliser un nom d’utilisateur et d’un mot de passe en texte brut pour l’authentification de base

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
    "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```

### <a name="use-port-enablessl-enableservercertificatevalidation"></a>Utiliser un port, enableSsl, enableServerCertificateValidation

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456",
            "port": "21",
            "enableSsl": true,
            "enableServerCertificateValidation": true
        }
    }
}
```

### <a name="use-encryptedcredential-for-authentication-and-gateway"></a>Utiliser encryptedCredential pour l’authentification et la passerelle

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "gatewayName": "mygateway"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, voir [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données.

La section **typeProperties** est différente pour chaque type de jeu de données. Elle fournit des informations spécifiques au type de jeu de données. La section **typeProperties** pour un jeu de données de type **FileShare** a les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Sous-chemin d’accès au dossier. Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Pour obtenir des exemples, consultez la section Exemples de définitions de jeux de données et de services liés.<br/><br/>Vous pouvez également combiner cette propriété avec **partitionBy** pour que les chemins d’accès soient basés sur les dates et heures de début et de fin de la tranche. |Oui |
| fileName |Spécifiez le nom du fichier dans l’élément **folderPath** si vous souhaitez que la table se réfère à un fichier spécifique du dossier. Si vous ne spécifiez aucune valeur pour cette propriété, le tableau pointe vers tous les fichiers du dossier.<br/><br/>Lorsque **fileName** n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré est au format suivant : <br/><br/>`Data.<Guid>.txt` (Exemple : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |Non |
| fileFilter |Spécifiez un filtre à utiliser pour sélectionner un sous-ensemble de fichiers dans le **folderPath** plutôt que tous les fichiers.<br/><br/>Les valeurs autorisées sont : `*` (plusieurs caractères) et `?` (caractère unique).<br/><br/>Exemple 1 : `"fileFilter": "*.log"`<br/>Exemple 2 : `"fileFilter": 2014-1-?.txt"`<br/><br/> **fileFilter** s’applique à un jeu de données FileShare d’entrée. Cette propriété n’est pas pris en charge avec le Système de fichiers DFS Hadoop (HDFS). |Non |
| partitionedBy |Utilisé pour spécifier un **folderPath** et un **fileName** dynamiques pour des données de série chronologique. Par exemple, vous pouvez spécifier un **folderPath** paramétré pour chaque heure de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, vois les sections [Format Text](data-factory-supported-file-and-compression-formats.md#text-format), [Format Json](data-factory-supported-file-and-compression-formats.md#json-format), [Format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [Format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section Format dans les définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Les types pris en charge sont **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Les niveaux pris en charge sont **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |
| useBinaryTransfer |Spécifiez s’il faut utiliser le mode de transfert binaire. Les valeurs sont true pour le mode binaire (c’est la valeur par défaut) et false pour ASCII. Cette propriété peut être utilisée uniquement quand le service lié associé est de type FtpServer. |Non |

> [!NOTE]
> **fileName** et **fileFilter** ne peuvent pas être utilisés simultanément.

### <a name="use-the-partionedby-property"></a>Utiliser la propriété partitionedBy
Comme mentionné dans la section précédente, vous pouvez spécifier un **folderPath** et un **fileName** dynamiques pour les données de série chronologique avec la propriété **partitionedBy**.

Pour découvrir les jeux de données de série chronologique, la planification et les segments, voir [Création de jeux de données](data-factory-create-datasets.md), [Planification et exécution](data-factory-scheduling-and-execution.md) et [Création de pipelines](data-factory-create-pipelines.md).

#### <a name="sample-1"></a>Exemple 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
Dans cet exemple, {Slice} est remplacé par la valeur de la variable système Data Factory SliceStart au format spécifié (AAAAMMJJHH). SliceStart fait référence à l'heure de début du segment. Le chemine d’accès du dossier est différent pour chaque segment (par exemple : wikidatagateway/wikisampledataout/2014100103 ou wikidatagateway/wikisampledataout/2014100104).

#### <a name="sample-2"></a>Exemple 2

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
[
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
Dans cet exemple, l’année, le mois, le jour et l’heure de SliceStart sont extraits dans des variables distinctes qui sont utilisées par les propriétés **folderPath** et **fileName**.

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et propriétés disponibles pour la définition des activités, voir [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d'entrée et de sortie et les différentes stratégies sont disponibles pour tous les types d'activités.

En revanche, les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Pour l’activité de copie, les propriétés de type dépendent des types de sources et de récepteurs.

Dans une activité de copie, quand la source est de type **FileSystemSource**, les propriétés suivantes sont disponibles dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. |True, False (par défaut) |Non |

## <a name="json-example-copy-data-from-ftp-server-to-azure-blob"></a>Exemple JSON : copier des données d’un serveur FTP vers Stockage Blob Azure
Cet exemple montre comment copier des données à partir d’un serveur FTP vers un stockage Blob Azure. Toutefois, il est possible de copier des données directement vers tout récepteur indiqué dans [magasins et formats de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) à l’aide de l’activité de copie dans Data Factory.

Les exemples suivants présentent des exemples de définitions de JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) :

* Un service lié de type [FtpServer](#linked-service-properties).
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [FileShare](#dataset-properties)
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties)
* Un [pipeline](data-factory-create-pipelines.md) avec activité de copie qui utilise [FileSystemSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties)

L’exemple copie des données d’un serveur FTP vers un objet blob Azure toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

### <a name="ftp-linked-service"></a>Service lié FTP

Cet exemple utilise une authentification de base avec le nom d’utilisateur et le mot de passe en texte brut. Vous pouvez également utiliser une des méthodes suivantes :

* Authentification anonyme
* Authentification de base avec des informations d’identification chiffrées
* FTP sur SSL/TLS (FTPS)

Pour connaître les différents types d’authentification que vous pouvez utiliser, voir [Service lié FTP](#linked-service-properties).

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```
### <a name="azure-storage-linked-service"></a>Service lié Stockage Azure

```JSON
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
### <a name="ftp-input-dataset"></a>Jeu de données d’entrée FTP

Ce jeu de données fait référence au dossier SFTP `mysharedfolder` et au fichier `test.csv`. Le pipeline copie le fichier vers la destination.

La définition de **external** sur **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à la fabrique de données et non produit par une activité dans celle-ci.

```JSON
{
  "name": "FTPFileInput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": "FTPLinkedService",
    "typeProperties": {
      "folderPath": "mysharedfolder",
      "fileName": "test.csv",
      "useBinaryTransfer": true
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

### <a name="azure-blob-output-dataset"></a>Jeu de données de sortie d’objet Blob Azure

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et la partie heure de l’heure de début.

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/ftp/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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


### <a name="a-copy-activity-in-a-pipeline-with-file-system-source-and-blob-sink"></a>Activité de copie dans un pipeline avec une source Système de fichiers et un récepteur blob

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie, et qui est planifiée pour s’exécuter toutes les heures. Dans la définition JSON du pipeline, le type **source** est défini sur **FileSystemSource** et le type **sink** sur **BlobSink**.

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "FTPToBlobCopy",
            "inputs": [{
                "name": "FtpFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-08-24T18:00:00Z",
        "end": "2016-08-24T19:00:00Z"
    }
}
```
> [!NOTE]
> Pour savoir comment mapper des colonnes d’un jeu de données source sur des colonnes d’un jeu de données récepteur, consultez [Mappage de colonnes des jeux de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

* Pour découvrir les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser, voir [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md).

* Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, voir [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
