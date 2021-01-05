---
title: Interroger des données depuis un stockage Azure compatible avec HDFS - Azure HDInsight
description: Découvrez comment interroger des données à partir du stockage Azure et d’Azure Data Lake Storage pour stocker les résultats de votre analyse.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: ead9b775b8c61d0d89abd4821bef2b1aaaea0d76
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547433"
---
# <a name="use-azure-storage-with-azure-hdinsight-clusters"></a>Utiliser le stockage Azure avec des clusters Azure HDInsight

Vous pouvez stocker des données dans [Stockage Blob Azure](../storage/common/storage-introduction.md), [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) ou [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md). Vous pouvez également choisir une combinaison de ces options. Ces options de stockage vous permettent de supprimer des clusters HDInsight servant aux calculs, sans perte de données utilisateur.

Apache Hadoop prend en charge une notion de système de fichiers par défaut. Le système de fichiers par défaut implique un schéma et une autorité par défaut. Il peut également être utilisé pour résoudre les chemins d'accès relatifs. Pendant le processus de création du cluster HDInsight, vous pouvez spécifier un conteneur de blobs dans Stockage Azure comme système de fichiers par défaut. Sinon, utilisez HDInsight 3.6 pour sélectionner Stockage Blob Azure ou Azure Data Lake Storage Gen1/Azure Data Lake Storage Gen2 comme système de fichiers par défaut avec quelques exceptions. Pour la prise en charge de l’utilisation de Data Lake Storage Gen1 en tant que stockage associé et par défaut, consultez [Disponibilité pour le cluster HDInsight](./hdinsight-hadoop-use-data-lake-storage-gen1.md#availability-for-hdinsight-clusters).

Dans cet article, vous découvrez le fonctionnement du stockage Azure avec des clusters HDInsight. 
* Pour savoir comment Data Lake Storage Gen1 fonctionne avec les clusters HDInsight, consultez [Utiliser Azure Data Lake Storage Gen1 avec des clusters HDInsight](./hdinsight-hadoop-use-data-lake-storage-gen1.md).
* Pour savoir comment Data Lake Storage Gen2 fonctionne avec les clusters HDInsight, consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](./hdinsight-hadoop-use-data-lake-storage-gen2.md).
* Pour plus d’informations sur la création d’un cluster HDInsight, consultez [Créer des clusters Apache Hadoop dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).

> [!IMPORTANT]  
> Le type de compte de stockage **BlobStorage** peut uniquement être utilisé comme stockage secondaire pour les clusters HDInsight.

| Type de compte de stockage | Services pris en charge | Niveaux de performances pris en charge |Niveaux de performances non pris en charge| Niveaux d’accès pris en charge |
|----------------------|--------------------|-----------------------------|---|------------------------|
| StorageV2 (usage général v2)  | Objet blob     | standard                    |Premium| Chaud, froid, archive\*   |
| Stockage (v1 universel)   | Objet blob     | standard                    |Premium| N/A                    |
| BlobStorage                    | Objet blob     | standard                    |Premium| Chaud, froid, archive\*   |

Nous vous déconseillons d’utiliser le conteneur d’objets blob par défaut pour stocker des données d’entreprise. Nous vous recommandons de supprimer le conteneur d’objets blob par défaut après chaque utilisation pour réduire les coûts de stockage. Le conteneur par défaut contient les journaux des applications et du système. Assurez-vous de récupérer les journaux d’activité avant de supprimer le conteneur.

Le partage d’un conteneur blob en tant que système de fichiers par défaut sur plusieurs clusters n’est pas pris en charge.

> [!NOTE]  
> Le niveau d’accès archive est un niveau hors connexion dont la latence de récupération de plusieurs heures n’est pas recommandée pour une utilisation avec HDInsight. Pour plus d’informations, voir [Niveau d’accès archive](../storage/blobs/storage-blob-storage-tiers.md#archive-access-tier).

## <a name="access-files-from-within-cluster"></a>Accès aux fichiers à partir de l’intérieur d’un cluster

Il existe plusieurs méthodes pour accéder aux fichiers dans Data Lake Storage à partir d’un cluster HDInsight. Le modèle d’URI offre à la fois un accès non chiffré (avec le préfixe *wasb:* ) et un accès chiffré TLS (avec *wasbs* ). Dans la mesure du possible, nous vous recommandons d’utiliser *wasbs* , même lorsqu’il s’agit d’accéder à des données qui résident dans la même région Azure.

* **Utilisation du nom complet** . Avec cette approche, vous fournissez le chemin d’accès complet au fichier auquel vous souhaitez accéder.

    ```
    wasb://<containername>@<accountname>.blob.core.windows.net/<file.path>/
    wasbs://<containername>@<accountname>.blob.core.windows.net/<file.path>/
    ```

* **Utilisation du format de chemin d’accès raccourci** . Avec cette approche, vous remplacez le chemin d’accès à la racine du cluster par :

    ```
    wasb:///<file.path>/
    wasbs:///<file.path>/
    ```

* **Utilisation du chemin d’accès relatif** . Avec cette approche, vous fournissez uniquement le chemin d’accès relatif au fichier auquel vous souhaitez accéder.

    ```
    /<file.path>/
    ```

### <a name="data-access-examples"></a>Exemples d’accès aux données

Les exemples sont basés sur une [connexion ssh](./hdinsight-hadoop-linux-use-ssh-unix.md) au nœud principal du cluster. Les exemples utilisent les trois schémas d’URI. Remplacez `CONTAINERNAME` et `STORAGEACCOUNT` par les valeurs correspondantes

#### <a name="a-few-hdfs-commands"></a>Quelques commandes hdfs

1. Créez un fichier sur le stockage local.

    ```bash
    touch testFile.txt
    ```

1. Créez des répertoires sur le stockage en cluster.

    ```bash
    hdfs dfs -mkdir wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/sampledata1/
    hdfs dfs -mkdir wasbs:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. Copiez les données du stockage local vers le stockage en cluster.

    ```bash
    hdfs dfs -copyFromLocal testFile.txt  wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt  wasbs:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt  /sampledata3/
    ```

1. Affichez le contenu du répertoire sur le stockage en cluster.

    ```bash
    hdfs dfs -ls wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/sampledata1/
    hdfs dfs -ls wasbs:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

> [!NOTE]  
> Lorsque vous utilisez des objets blob hors de HDInsight, la plupart des utilitaires ne reconnaissent pas le format WASB et attendent plutôt un format de chemin d’accès basique, comme `example/jars/hadoop-mapreduce-examples.jar`.

#### <a name="creating-a-hive-table"></a>Création d’une table Hive

Trois emplacements de fichiers sont indiqués à titre d’illustration. Pour l’exécution réelle, n’utilisez qu’une seule des entrées `LOCATION`.

```hql
DROP TABLE myTable;
CREATE EXTERNAL TABLE myTable (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE
LOCATION 'wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/example/data/';
LOCATION 'wasbs:///example/data/';
LOCATION '/example/data/';
```

## <a name="access-files-from-outside-cluster"></a>Accès aux fichiers à partir de l’extérieur d’un cluster

Microsoft fournit les outils suivants pour utiliser Stockage Azure :

| Outil | Linux | OS X | Windows |
| --- |:---:|:---:|:---:|
| [Azure portal](../storage/blobs/storage-quickstart-blobs-portal.md) |✔ |✔ |✔ |
| [Azure CLI](../storage/blobs/storage-quickstart-blobs-cli.md) |✔ |✔ |✔ |
| [Azure PowerShell](../storage/blobs/storage-quickstart-blobs-powershell.md) | | |✔ |
| [AZCopy](../storage/common/storage-use-azcopy-v10.md) |✔ | |✔ |

## <a name="identify-storage-path-from-ambari"></a>Identifier le chemin de stockage à partir d’Ambari

* Pour identifier le chemin d'accès complet à la mémoire par défaut configurée, naviguez jusqu'à :

    **HDFS** > **Configs** , puis entrez `fs.defaultFS` dans la zone de saisie du filtre.

* Pour vérifier si le magasin wasb est configuré comme stockage secondaire, naviguez jusqu'à :

    **HDFS** > **Configs** , puis entrez `blob.core.windows.net` dans la zone de saisie du filtre.

Pour obtenir le chemin d’accès à l’aide de l’API REST Ambari, voir [Obtenir le stockage par défaut](./hdinsight-hadoop-manage-ambari-rest-api.md#get-the-default-storage).

## <a name="blob-containers"></a>Conteneurs d’objets blob

Pour utiliser des objets blob, commencez par créer un [compte Stockage Azure](../storage/common/storage-account-create.md). À cette étape, vous spécifiez la région Azure dans laquelle le compte de stockage est créé. Le cluster et le compte de stockage doivent être hébergés dans la même région. La base de données SQL Server de metastore Hive et la base de données SQL Server de metastore Apache Oozie doivent se trouver dans la même région.

Où qu’il réside, chaque objet blob que vous créez appartient à un conteneur de votre compte de stockage Azure. Ce conteneur peut être un blob existant créé en dehors de HDInsight. Il peut également s’agir d’un conteneur créé pour un cluster HDInsight.

Le conteneur d’objets blob par défaut stocke les informations spécifiques de cluster telles que l’historique et les journaux d’activité des travaux. Ne partagez pas un conteneur d’objets blob par défaut avec plusieurs clusters HDInsight. Cette action est susceptible d’endommager l’historique des travaux. Il est recommandé d’utiliser un conteneur différent pour chaque cluster. Placez les données partagées sur un compte de stockage lié spécifié pour tous les clusters concernés plutôt que sur le compte de stockage par défaut. Pour plus d'informations sur la configuration des comptes de stockage liés, consultez [Créer des clusters HDInsight](hdinsight-hadoop-provision-linux-clusters.md). Vous pouvez, toutefois, réutiliser un conteneur de stockage par défaut une fois le cluster HDInsight d'origine supprimé. Pour les clusters HBase, vous pouvez conserver le schéma et les données de la table HBase en créant un cluster HBase à l’aide du conteneur de blobs par défaut utilisé par un cluster HBase supprimé.

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

## <a name="use-additional-storage-accounts"></a>Utiliser des comptes de stockage supplémentaires

Lorsque vous créez un cluster HDInsight, vous spécifiez le compte de stockage Azure que vous souhaitez lui associer. Vous pouvez également ajouter d’autres comptes de stockage à partir du même abonnement Azure ou à partir d’autres abonnements Azure pendant le processus de création ou à l’issue de la création d’un cluster. Pour en savoir plus sur l'ajout de comptes de stockage supplémentaires, consultez la rubrique [Création de clusters HDInsight](hdinsight-hadoop-provision-linux-clusters.md).

> [!WARNING]  
> L’utilisation d’un compte de stockage supplémentaire dans un autre emplacement que le cluster HDInsight n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser un stockage Azure compatible avec HDFS avec HDInsight. Ce stockage vous permet de créer des solutions à long terme et adaptables d’acquisition et d’archivage de données et d’utiliser HDInsight pour déverrouiller les informations des données structurées et non structurées stockées.

Pour plus d'informations, consultez les pages suivantes :

* [Démarrage rapide : Créer un cluster Apache Hadoop](hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* [Tutoriel : Créer des clusters HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
* [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md)
* [Téléchargement de données vers HDInsight](hdinsight-upload-data.md)
* [Tutoriel : Extraire, transformer et charger des données à l’aide d’Interactive Query sur Azure HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
* [Utiliser des signatures d’accès partagé Stockage Azure pour restreindre l’accès aux données avec HDInsight](hdinsight-storage-sharedaccesssignature-permissions.md)