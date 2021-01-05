---
title: Format Binaire dans Azure Data Factory
description: Cette rubrique décrit comment traiter le format Binaire dans Azure Data Factory.
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: jingwang
ms.openlocfilehash: be4338942057cf52bf26b7fd1e15f77e724d49f8
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348906"
---
# <a name="binary-format-in-azure-data-factory"></a>Format Binaire dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Le format Binaire est pris en charge pour les connecteurs suivants : [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [Système de fichiers](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) et [SFTP](connector-sftp.md).

Vous pouvez utiliser un jeu de données Binaire dans l’[activité Copie](copy-activity-overview.md), l'[activité GetMetadata](control-flow-get-metadata-activity.md) ou l’[activité Suppression](delete-activity.md). Lors de l’utilisation d’un jeu de données binaire, ADF n’analyse pas le contenu du fichier, mais le traite tel quel. 

>[!NOTE]
>Lors de l’utilisation d’un jeu de données Binaire dans l’activité de copie, vous pouvez uniquement effectuer une copie à partir d’un jeu de données binaire vers un jeu de données binaire.

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données Binaire.

| Propriété         | Description                                                  | Obligatoire |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propriété type du jeu de données doit être définie sur **Binaire**. | Oui      |
| location         | Paramètres d’emplacement du ou des fichiers. Chaque connecteur basé sur un fichier possède ses propres type d’emplacement et propriétés prises en charge sous `location`. **Consultez les détails dans l’article du connecteur -> section des propriétés du jeu de données**. | Oui      |
| compression | Groupe de propriétés pour configurer la compression de fichier. Configurez cette section lorsque vous souhaitez effectuer la compression/décompression lors de l’exécution de l’activité. | Non |
| type | Le codec de compression utilisé pour lire/écrire des fichiers binaires. <br>Les valeurs autorisées sont **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **Tar** et **TarGzip**. <br>**Remarque :** Lorsque l’activité de copie est utilisée pour décompresser un ou plusieurs fichiers **ZipDeflate**/**TarGzip**/**Tar** et écrire dans un magasin de données récepteur basé sur des fichiers, les fichiers sont par défaut extraits dans le dossier `<path specified in dataset>/<folder named as source compressed file>/`. Utilisez `preserveZipFileNameAsFolder`/`preserveCompressionFileNameAsFolder` sur [source de l’activité de copie](#binary-as-source) pour déterminer si le nom du ou des fichiers compressés doit être conservé comme structure de dossier.| Non       |
| level | Le taux de compression. Appliquez quand le jeu de données est utilisé dans le récepteur d’activité de copie.<br>Les valeurs autorisées sont **Optimal** ou **Fastest**.<br>- **Fastest (le plus rapide) :** l’opération de compression doit se terminer le plus rapidement possible, même si le fichier résultant n’est pas compressé de façon optimale.<br>- **Optimal** : l’opération de compression doit aboutir à une compression optimale, même si elle prend plus de temps. Pour plus d’informations, consultez la rubrique [Niveau de compression](/dotnet/api/system.io.compression.compressionlevel) . | Non       |

Voici un exemple de jeu de données Binaire sur Stockage Blob Azure :

```json
{
    "name": "BinaryDataset",
    "properties": {
        "type": "Binary",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "ZipDeflate"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source et le récepteur binaires.

>[!NOTE]
>Lors de l’utilisation d’un jeu de données Binaire dans l’activité de copie, vous pouvez uniquement effectuer une copie à partir d’un jeu de données binaire vers un jeu de données binaire.

### <a name="binary-as-source"></a>Binaire en tant que source

Les propriétés prises en charge dans la section **_\_source\*** * de l’activité de copie sont les suivantes.

| Propriété      | Description                                                  | Obligatoire |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propriété type de la source de l’activité de copie doit être définie sur **BinarySource**. | Oui      |
| formatSettings | Un groupe de propriétés. Reportez-vous au tableau **Paramètres de lecture binaire** ci-dessous. | Non       |
| storeSettings | Un groupe de propriétés sur la façon de lire les données d’un magasin de données. Chaque connecteur basé sur un fichier possède ses propres paramètres de lecture pris en charge sous `storeSettings`. **Consultez les détails dans l’article du connecteur -> section des propriétés de l’activité de copie**. | Non       |

**Paramètres de lecture binaire** pris en charge sous `formatSettings` :

| Propriété      | Description                                                  | Obligatoire |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | Le type de formatSettings doit être défini sur **BinaryReadSettings**. | Oui      |
| compressionProperties | Groupe de propriétés permettant de décompresser les données d’un codec de compression spécifique. | Non       |
| preserveZipFileNameAsFolder<br>(*sous `compressionProperties`->`type` en tant que `ZipDeflateReadSettings`* ) | S’applique lorsque le jeu de données d’entrée est configuré avec la compression **ZipDeflate**. Indique si le nom du fichier zip source doit être conservé en tant que structure de dossiers lors de la copie.<br>– Lorsque la valeur est définie sur **true (par défaut)** , Data Factory écrit les fichiers décompressés dans `<path specified in dataset>/<folder named as source zip file>/`.<br>– Lorsque la valeur est définie sur **false**, Data Factory écrit les fichiers décompressés directement dans `<path specified in dataset>`. Assurez-vous de ne pas avoir de noms de fichiers dupliqués dans les différents fichiers zip sources afin d’éviter toute course ou tout comportement inattendu.  | Non |
| preserveCompressionFileNameAsFolder<br>(*sous `compressionProperties`->`type` comme `TarGZipReadSettings` ou `TarReadSettings`* ) | S’applique lorsque le jeu de données d’entrée est configuré avec la compression **TarGzip**/**Tar**. Indique si le nom du fichier source compressé doit être conservé en tant que structure de dossiers lors de la copie.<br>– Lorsque la valeur est définie sur **true (par défaut)** , Data Factory écrit les fichiers décompressés dans `<path specified in dataset>/<folder named as source compressed file>/`. <br>– Lorsque la valeur est définie sur **false**, Data Factory écrit les fichiers décompressés directement dans `<path specified in dataset>`. Assurez-vous de ne pas avoir de noms de fichiers dupliqués dans les différents fichiers sources afin d’éviter toute course ou tout comportement inattendu. | Non |

```json
"activities": [
    {
        "name": "CopyFromBinary",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true,
                    "deleteFilesAfterCompletion": true
                },
                "formatSettings": {
                    "type": "BinaryReadSettings",
                    "compressionProperties": {
                        "type": "ZipDeflateReadSettings",
                        "preserveZipFileNameAsFolder": false
                    }
                }
            },
            ...
        }
        ...
    }
]
```

### <a name="binary-as-sink"></a>Binaire en tant que récepteur

Les propriétés prises en charge dans la section **_\_récepteur\*** * de l’activité de copie sont les suivantes.

| Propriété      | Description                                                  | Obligatoire |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propriété type de la source d’activité de copie doit être définie sur **BinarySink**. | Oui      |
| storeSettings | Groupe de propriétés sur la méthode d’écriture de données dans un magasin de données. Chaque connecteur basé sur un fichier possède ses propres paramètres d’écriture pris en charge sous `storeSettings`. **Consultez les détails dans l’article du connecteur -> section des propriétés de l’activité de copie**. | Non       |

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)
- [Supprimer l’activité](delete-activity.md)