---
title: Conserver les métadonnées et les listes de contrôle d'accès à l'aide de l'outil Copier l'activité d'Azure Data Factory
description: Apprenez à conserver les métadonnées et les listes de contrôle d'accès (ACL) pendant la copie à l'aide de l'outil Copier l'activité d'Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: jingwang
ms.openlocfilehash: a96b04df56dc7d5ea26463073d673275b8a4a8c4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015075"
---
#  <a name="preserve-metadata-and-acls-using-copy-activity-in-azure-data-factory"></a>Conserver les métadonnées et les listes de contrôle d'accès à l'aide de l'outil Copier l'activité d'Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Lorsque vous utilisez l'outil Copier l'activité d'Azure Data Factory pour copier des données de la source vers le récepteur, dans les scénarios suivants, vous pouvez également conserver les métadonnées et les listes de contrôle d'accès (ACL).

## <a name="preserve-metadata-for-lake-migration"></a><a name="preserve-metadata"></a> Conserver les métadonnées pour la migration entre lacs

Lorsque vous migrez des données d’un lac de données vers un autre, tel que [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) et [Stockage Fichier Azure](connector-azure-file-storage.md), vous pouvez choisir de conserver les métadonnées de fichier avec les données.

L'outil Copier l'activité prend en charge la conservation des attributs suivants lors de la copie des données :

- **Toutes les métadonnées spécifiées par le client** 
- Et les **cinq propriétés système intégrées au magasin de données** suivantes : `contentType`, `contentLanguage` (sauf pour Amazon S3), `contentEncoding`, `contentDisposition`, `cacheControl`.

**Gérer les différences dans les métadonnées :** Amazon S3 et Stockage Microsoft Azure autorisent des jeux de caractères différents dans les clés des métadonnées spécifiées par le client. Quand vous choisissez de conserver les métadonnées en utilisant la copie d’activité, ADF remplace automatiquement les caractères non valides par « _ ».

Si vous copiez des fichiers en l’état d’Amazon S3/Azure Data Lake Storage Gen2/Stockage Blob Azure/Stockage Fichier Azure vers Azure Data Lake Storage Gen2/Stockage Blob Azure/Stockage Fichier Azure au format binaire, vous pouvez trouver l’option **Conserver** sous l’onglet **Activité de copie** > **Paramètres** pour la création d’activité ou la page **Paramètres** de l’outil Copier des données.

![Copier l'activité - Conserver les métadonnées](./media/copy-activity-preserve-metadata/copy-activity-preserve-metadata.png)

Voici un exemple de configuration JSON de l'outil Copier l'activité (voir `preserve`) : 

```json
"activities":[
    {
        "name": "CopyAndPreserveMetadata",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AmazonS3ReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "Attributes"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset Amazon S3/Azure Blob/ADLS Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset for Azure Blob/ADLS Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="preserve-acls-from-data-lake-storage-gen1gen2-to-gen2"></a><a name="preserve-acls"></a> Conserver les listes de contrôle d’accès entre Data Lake Storage Gen1/Gen2 et Gen2

Quand vous mettez à niveau Azure Data Lake Storage Gen1 vers Gen2 ou que vous copiez des données de ADLS Gen2 vers ADLS Gen2, vous pouvez choisir de conserver les listes de contrôle d’accès (ACL) POSIX avec les fichiers de données. Pour plus d’informations sur le contrôle d'accès, voir [Contrôle d’accès dans Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md) et [Contrôle d’accès dans Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-access-control.md).

L'outil Copier l'activité prend en charge la conservation des types de listes de contrôle d'accès (ACL) suivants pendant la copie des données. Vous pouvez sélectionner un ou plusieurs types :

- **ACL** : copiez et conservez les listes de contrôle d’accès POSIX sur les fichiers et les répertoires. Cette option copie les ACL existantes complètes de la source vers le récepteur. 
- **Propriétaire** : copiez et conservez l’utilisateur propriétaire des fichiers et des répertoires. Un accès de super utilisateur pour le récepteur Data Lake Storage Gen2 est requis.
- **Groupe** : copiez et conservez le groupe propriétaire des fichiers et des répertoires. Un accès de super utilisateur pour le récepteur Data Lake Storage Gen2 ou l’utilisateur propriétaire (si ce dernier est également membre du groupe cible) est requis.

Si vous spécifiez une copie à partir d’un dossier, Data Factory réplique les ACL de ce dossier, ainsi que les fichiers et les répertoires qu’il contient, si `recursive` est défini sur true. Si vous spécifiez une copie à partir d’un seul fichier, les ACL sur ce fichier sont copiés.

>[!NOTE]
>Quand vous utilisez ADF pour conserver les ACL entre Data Lake Storage Gen1/Gen2 et Gen2, les ACL existantes des dossiers/fichiers correspondants du récepteur Gen2 sont remplacées.

>[!IMPORTANT]
>Lorsque vous choisissez de conserver les ACL, veillez à accorder des autorisations suffisantes à Data Factory pour qu’il fonctionne avec votre compte Data Lake Storage Gen2 récepteur. Par exemple, utilisez l’authentification par clé de compte ou attribuez le rôle de propriétaire des données Blob de stockage au principal de service ou à l’identité managée.

Si vous configurez Data Lake Storage Gen1/Gen2 comme source au format binaire ou avec l’option de copie binaire, et Data Lake Storage Gen2 comme récepteur au format binaire ou avec l’option de copie binaire, vous avez l’option **Conserver** dans la page **Paramètres** de l’outil Copier des données ou sous l’onglet **Copier l’activité** > **Paramètres** pour la création de l’activité.

![Conserver les ACL entre Data Lake Storage Gen1/Gen2 et Gen2](./media/connector-azure-data-lake-storage/adls-gen2-preserve-acl.png)

Voici un exemple de configuration JSON de l'outil Copier l'activité (voir `preserve`) : 

```json
"activities":[
    {
        "name": "CopyAndPreserveACLs",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AzureDataLakeStoreReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings": {
                    "type": "AzureBlobFSWriteSettings"
                }
            },
            "preserve": [
                "ACL",
                "Owner",
                "Group"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen1/Gen2 source>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Binary dataset name for Azure Data Lake Storage Gen2 sink>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

Voir les autres articles relatifs à l’activité de copie :

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Performances de l’activité de copie](copy-activity-performance.md)
