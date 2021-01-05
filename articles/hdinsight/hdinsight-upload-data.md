---
title: Charger des données pour des travaux Apache Hadoop dans HDInsight
description: Découvrez comment télécharger des données et y accéder pour des tâches Apache Hadoop dans HDInsight. Utilisez l’interface de ligne de commande Azure classique, l’Explorateur Stockage Azure, Azure PowerShell, la ligne de commande Hadoop ou Sqoop.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdiseo17may2017,seoapr2020
ms.date: 04/27/2020
ms.openlocfilehash: 47262e0dc7247e1d514eeb64512c21ff1ae527ab
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92534819"
---
# <a name="upload-data-for-apache-hadoop-jobs-in-hdinsight"></a>Charger des données pour des travaux Apache Hadoop dans HDInsight

HDInsight fournit un système HDFS (Hadoop Distributed File System) pour le Stockage Azure et Azure Data Lake Storage. Ce stockage comprend Gen1 et Gen2. Stockage Azure et Azure Data Lake Storage (Gen1 et Gen2) sont conçus en tant qu’extensions HDFS. Ils permettent à l’ensemble des composants de l’environnement Hadoop de fonctionner directement sur les données qu’il gère. Stockage Azure, Data Lake Storage Gen1, et Gen2 sont des systèmes de fichiers distincts. Les systèmes sont optimisés pour le stockage des données et des calculs sur ces données. Pour connaître les avantages que constitue l’utilisation du stockage Azure, consultez la page [Utilisation du stockage Azure avec HDInsight](hdinsight-hadoop-use-blob-storage.md). Voir aussi [Utiliser Data Lake Storage Gen1 avec HDInsight](hdinsight-hadoop-use-data-lake-storage-gen1.md) et [Utiliser Data Lake Storage Gen2 avec HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md).

## <a name="prerequisites"></a>Prérequis

Notez les prérequis suivants avant de démarrer :

* Un cluster Azure HDInsight. Pour obtenir des instructions, consultez [Démarrage rapide : Exécuter une tâche Spark sur Azure HDInsight à l’aide du portail Azure](hadoop/apache-hadoop-linux-tutorial-get-started.md).
* Connaissance des articles suivants :
    * [Utiliser Stockage Azure avec HDInsight](hdinsight-hadoop-use-blob-storage.md)
    * [Utiliser Data Lake Storage Gen1 avec HDInsight](hdinsight-hadoop-use-data-lake-storage-gen1.md)
    * [Utiliser Data Lake Storage Gen2 avec HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md)  

## <a name="upload-data-to-azure-storage"></a>Charger des données sur le Stockage Azure

### <a name="utilities"></a>Services

Microsoft fournit les utilitaires suivants pour utiliser le Stockage Azure :

| Outil | Linux | OS X |  Windows |
| --- |:---:|:---:|:---:|
| [Azure portal](../storage/blobs/storage-quickstart-blobs-portal.md) |✔ |✔ |✔ |
| [Azure CLI](../storage/blobs/storage-quickstart-blobs-cli.md) |✔ |✔ |✔ |
| [Azure PowerShell](../storage/blobs/storage-quickstart-blobs-powershell.md) | | |✔ |
| [AZCopy](../storage/common/storage-use-azcopy-v10.md) |✔ | |✔ |
| [Commande Hadoop](#hadoop-command-line) |✔ |✔ |✔ |

> [!NOTE]  
> La commande Hadoop est uniquement disponible sur le cluster HDInsight. La commande autorise uniquement le chargement de données du système de fichiers local sur le Stockage Azure.  

### <a name="hadoop-command-line"></a>Ligne de commande Hadoop

La ligne de commande Hadoop est utile uniquement pour stocker les données dans le blob du stockage Azure quand celles-ci sont déjà présentes sur le nœud principal du cluster.

Pour utiliser la commande Hadoop, vous devez d’abord vous connecter au nœud principal à l’aide de [SSH ou PuTTY](hdinsight-hadoop-linux-use-ssh-unix.md).

Une fois connecté, vous pouvez utiliser la syntaxe suivante pour télécharger un fichier dans le stockage.

```bash
hadoop fs -copyFromLocal <localFilePath> <storageFilePath>
```

Par exemple : `hadoop fs -copyFromLocal data.txt /example/data/data.txt`

Comme le système de fichiers par défaut pour HDInsight se trouve dans le Stockage Azure, /example/data.txt s’y trouve également. Vous pouvez également faire référence au fichier comme ceci :

`wasbs:///example/data/data.txt`

or

`wasbs://<ContainerName>@<StorageAccountName>.blob.core.windows.net/example/data/davinci.txt`

Pour la liste des autres commandes Hadoop qui fonctionnent avec des fichiers, voir [https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html)

> [!WARNING]  
> Sur les clusters Apache HBase, la taille de bloc par défaut durant l’écriture de données est de 256 Ko. Bien que cela fonctionne bien quand vous utilisez les API HBase ou les API REST, l’utilisation des commandes `hadoop` ou `hdfs dfs` pour écrire des données supérieures à ~12 Go génère une erreur. Pour plus d’informations, consultez [Exception de stockage pour l’écriture sur un objet blob](hdinsight-troubleshoot-hdfs.md#storage-exception-for-write-on-blob).

### <a name="graphical-clients"></a>Clients graphiques

Plusieurs applications fournissent également une interface graphique pour utiliser Azure Storage. Le tableau suivant répertorie quelques-unes de ces applications :

| Client | Linux | OS X |  Windows |
| --- |:---:|:---:|:---:|
| [Outils Microsoft Visual Studio pour HDInsight](hadoop/apache-hadoop-visual-studio-tools-get-started.md#explore-linked-resources) |✔ |✔ |✔ |
| [Azure Storage Explorer](../storage/blobs/storage-quickstart-blobs-storage-explorer.md) |✔ |✔ |✔ |
| [`Cerulea`](https://www.cerebrata.com/products/cerulean/features/azure-storage) | | |✔ |
| [CloudXplorer](https://clumsyleaf.com/products/cloudxplorer) | | |✔ |
| [CloudBerry Explorer for Microsoft Azure](https://www.cloudberrylab.com/free-microsoft-azure-explorer.aspx) | | |✔ |
| [Cyberduck](https://cyberduck.io/) | |✔ |✔ |

## <a name="mount-azure-storage-as-local-drive"></a>Monter le stockage Azure comme un lecteur Local

Consultez [Monter le stockage Azure comme un lecteur Local](/archive/blogs/bigdatasupport/mount-azure-blob-storage-as-local-drive).

## <a name="upload-using-services"></a>Effectuer un chargement en utilisant des services

### <a name="azure-data-factory"></a>Azure Data Factory

Le service Azure Data Factory est un service entièrement géré pour composer des services de stockage, de traitement et de déplacement de données dans des pipelines de production rationalisés, adaptables et fiables.

|Type de stockage|Documentation|
|----|----|
|Stockage Blob Azure|[Copier des données vers ou depuis le stockage Blob Azure à l’aide d’Azure Data Factory](../data-factory/connector-azure-blob-storage.md)|
|Azure Data Lake Storage Gen1|[Copier des données vers ou depuis Azure Data Lake Storage Gen1 à l'aide d'Azure Data Factory](../data-factory/connector-azure-data-lake-store.md)|
|Azure Data Lake Storage Gen2 |[Charger des données dans Azure Data Lake Storage Gen2 avec Azure Data Factory](../data-factory/load-azure-data-lake-storage-gen2.md)|

### <a name="apache-sqoop"></a>Apache Sqoop

Sqoop est un outil conçu pour transférer des données entre Hadoop et des bases de données relationnelles. Utilisez-le pour importer des données à partir d’un système de gestion de base de données relationnelle (SGBDR), comme SQL Server, MySQL ou Oracle. Puis dans le système de fichiers Hadoop Distributed File System (HDFS). Transformez les données dans Hadoop avec MapReduce ou Hive, puis exportez de nouveau les données dans un SGBDR.

Pour plus d’informations, consultez [Utilisation de Sqoop avec HDInsight](hadoop/hdinsight-use-sqoop.md).

### <a name="development-sdks"></a>Kits de développement logiciel (SDK) de développement

Le Stockage Azure est également accessible à l’aide d’un SDK Azure dans les langages de programmation suivants :

* .NET
* Java
* Node.js
* PHP
* Python
* Ruby

Pour plus d'informations sur l'installation des kits de développement logiciel (SDK) Azure, consultez [Téléchargements Azure](https://azure.microsoft.com/downloads/)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment obtenir des données avec HDInsight, consultez les articles suivants pour apprendre à effectuer des analyses :

* [Prise en main d’Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Envoi de tâches Apache Hadoop par programmation](hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [Utilisation d’Apache Hive avec HDInsight](hadoop/hdinsight-use-hive.md)