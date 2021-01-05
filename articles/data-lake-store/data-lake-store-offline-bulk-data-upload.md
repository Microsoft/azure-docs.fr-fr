---
title: Charger un jeu de données volumineux vers Azure Data Lake Storage Gen1 – méthodes hors connexion
description: Utiliser le service Import/Export pour copier les données du stockage de blobs Azure vers Azure Data Lake Storage Gen1
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: d04a5c0e53e9a5db8bba03a5a9e9d95b87a8b5a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85855669"
---
# <a name="use-the-azure-importexport-service-for-offline-copy-of-data-to-data-lake-storage-gen1"></a>Utiliser le service Azure Import/Export pour copier les données dans Data Lake Storage Gen1 hors connexion

Dans cet article, vous allez découvrir comment copier de très grands jeux de données (> 200 Go) dans Data Lake Storage Gen1 à l’aide de méthodes de copie hors connexion, comme le [service Azure Import/Export](../storage/common/storage-import-export-service.md). Plus précisément, la taille du fichier utilisé comme exemple dans cet article est de 339 420 860 416 octets ou environ 319 Go sur disque. Appelons ce fichier 319GB.tsv.

Le service Azure Import/Export vous aide à transférer de façon plus sécurisée des volumes importants de données vers Stockage Blob Azure en expédiant des disques durs vers un centre de données Azure.

## <a name="prerequisites"></a>Prérequis

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Storage**.
* **Un compte Azure Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).

## <a name="prepare-the-data"></a>Préparer les données

Avant d’utiliser le service Import/Export, scindez le fichier de données à transférer **en copies de moins de 200 Go**. L’outil d’importation ne fonctionne pas avec des fichiers de plus de 200 Go. Dans cet article, nous fractionnons le fichier en blocs de 100 Go. Pour ce faire, utilisez [Cygwin](https://cygwin.com/install.html). Cygwin prend en charge les commandes Linux. Dans ce cas, utilisez la commande suivante :

```console
split -b 100m 319GB.tsv
```

L’opération split crée des fichiers portant les noms suivants.

* *319GB.tsv-part-aa*
* *319GB.tsv-part-ab*
* *319GB.tsv-part-ac*
* *319GB.tsv-part-ad*

## <a name="get-disks-ready-with-data"></a>Préparer les disques avec les données

Suivez les instructions relatives à [l’utilisation du Service Azure Import/Export](../storage/common/storage-import-export-service.md) (sous la section **Préparation des lecteurs**) pour préparer vos disques durs. Voici la séquence générale :

1. Procurez-vous un disque dur utilisable pour le service Import/Export Azure.
2. Identifiez un compte de stockage Azure où les données seront copiées une fois le disque expédié vers le centre de données Azure.
3. Utilisez [l’outil Azure Import/Export](https://go.microsoft.com/fwlink/?LinkID=301900&clcid=0x409), qui est un utilitaire de ligne de commande. Voici un exemple d’extrait de code illustrant l’utilisation de l’outil.

    ```
    WAImportExport PrepImport /sk:<StorageAccountKey> /t: <TargetDriveLetter> /format /encrypt /logdir:e:\myexportimportjob\logdir /j:e:\myexportimportjob\journal1.jrn /id:myexportimportjob /srcdir:F:\demo\ExImContainer /dstdir:importcontainer/vf1/
    ```
    Pour obtenir d’autres exemples d’extraits, consultez [Using the Azure Import/Export service](../storage/common/storage-import-export-service.md) (Utilisation du service Azure Import/Export).
4. La commande précédente crée un fichier journal à l’emplacement spécifié. Utilisez ce fichier journal pour créer un travail d’importation à partir du [portail Azure](https://portal.azure.com).

## <a name="create-an-import-job"></a>Créer une tâche d’importation

Vous pouvez maintenant créer un travail d’importation en suivant les instructions relatives à [l’utilisation du Service Azure Import/Export](../storage/common/storage-import-export-service.md) (sous la section **Création de la tâche d’importation**). Pour ce travail d’importation, entre autres détails, indiquez le fichier journal créé durant la préparation des lecteurs de disque.

## <a name="physically-ship-the-disks"></a>Expédier les disques physiquement

Vous pouvez désormais expédier physiquement les disques vers un centre de données Azure. Les données y seront copiées dans les objets blob Stockage Azure que vous avez fournis durant la création du travail d’importation. En outre, pendant la création du travail, si vous avez choisi d’indiquer les informations de suivi plus tard, vous pouvez maintenant revenir à votre travail d’importation et mettre à jour le numéro de suivi.

## <a name="copy-data-from-blobs-to-data-lake-storage-gen1"></a>Copier des données de blobs vers Data Lake Storage Gen1

Une fois que l’état du travail d’importation indique que celui-ci est terminé, vous pouvez vérifier si les données sont disponibles dans les objets blob Stockage Azure que vous aviez spécifiés. Vous pouvez ensuite utiliser différentes méthodes pour déplacer ces données depuis les objets blob vers Azure Data Lake Storage Gen1. Pour connaître toutes les options disponibles pour le chargement de données, consultez [Réception de données dans Azure Data Lake Storage Gen1](data-lake-store-data-scenarios.md#ingest-data-into-data-lake-storage-gen1).

Dans cette section, nous vous fournissons les définitions JSON grâce auxquelles vous pouvez créer un pipeline Azure Data Factory en vue de copier les données. Vous pouvez utiliser ces définitions JSON à partir du [portail Azure](../data-factory/tutorial-copy-data-portal.md) ou [Visual Studio](../data-factory/tutorial-copy-data-dot-net.md).

### <a name="source-linked-service-azure-storage-blob"></a>Service lié source (objet blob Stockage Azure)

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="target-linked-service-data-lake-storage-gen1"></a>Service lié cible (Data Lake Storage Gen1)

```JSON
{
    "name": "AzureDataLakeStorageGen1LinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "description": "",
        "typeProperties": {
            "authorization": "<Click 'Authorize' to allow this data factory and the activities it runs to access this Data Lake Storage Gen1 account with your access rights>",
            "dataLakeStoreUri": "https://<adlsg1_account_name>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<OAuth session id from the OAuth authorization session. Each session id is unique and may only be used once>"
        }
    }
}
```

### <a name="input-data-set"></a>Jeu de données d’entrée

```JSON
{
    "name": "InputDataSet",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "importcontainer/vf1/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

### <a name="output-data-set"></a>Jeu de données de sortie

```JSON
{
"name": "OutputDataSet",
"properties": {
  "published": false,
  "type": "AzureDataLakeStore",
  "linkedServiceName": "AzureDataLakeStorageGen1LinkedService",
  "typeProperties": {
    "folderPath": "/importeddatafeb8job/"
    },
  "availability": {
    "frequency": "Hour",
    "interval": 1
    }
  }
}
```

### <a name="pipeline-copy-activity"></a>Pipeline (activité de copie)

```JSON
{
    "name": "CopyImportedData",
    "properties": {
        "description": "Pipeline with copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataSet"
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
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity"
            }
        ],
        "start": "2016-02-08T22:00:00Z",
        "end": "2016-02-08T23:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
```

Pour plus d’informations, consultez [Move data from Azure Storage blob to Azure Data Lake Storage Gen1 using Azure Data Factory](../data-factory/connector-azure-data-lake-store.md) (Déplacer des données à partir d’un objet blob Stockage Azure vers Azure Data Lake Storage Gen1 à l’aide d’Azure Data Factory).

## <a name="reconstruct-the-data-files-in-data-lake-storage-gen1"></a>Reconstruire les fichiers de données dans Data Lake Storage Gen1

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Nous avons commencé avec un fichier de 319 Go, que nous avons scindé en fichiers de plus petite taille afin de pouvoir le transférer à l’aide du service Azure Import/Export. Les données se trouvant désormais dans Azure Data Lake Storage Gen1, nous pouvons reconstruire le fichier à sa taille d’origine. Pour ce faire, vous pouvez utiliser les applets de commande Azure PowerShell suivante.

```PowerShell
# Login to our account
Connect-AzAccount

# List your subscriptions
Get-AzSubscription

# Switch to the subscription you want to work with
Set-AzContext -SubscriptionId
Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"

# Join  the files
Join-AzDataLakeStoreItem -AccountName "<adlsg1_account_name" -Paths "/importeddatafeb8job/319GB.tsv-part-aa","/importeddatafeb8job/319GB.tsv-part-ab", "/importeddatafeb8job/319GB.tsv-part-ac", "/importeddatafeb8job/319GB.tsv-part-ad" -Destination "/importeddatafeb8job/MergedFile.csv"
```

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
