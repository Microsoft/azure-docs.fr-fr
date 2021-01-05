---
title: Copier des données dans Azure Data Lake Storage Gen2 à l'aide de DistCp | Microsoft Docs
description: Copiez des données vers et depuis Azure Data Lake Storage Gen2 à l’aide de l’outil de copie distribuée d’Apache Hadoop (DistCp).
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 12/06/2018
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: e69a97a86a357fb36dde572f292b5cac7963d14a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912482"
---
# <a name="use-distcp-to-copy-data-between-azure-storage-blobs-and-azure-data-lake-storage-gen2"></a>Utiliser DistCp pour copier des données entre Azure Storage Blob et Azure Data Lake Storage Gen2

Vous pouvez utiliser [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html) pour copier des données entre un compte de stockage V2 universel et un compte de stockage V2 universel avec espace de noms hiérarchique activé. Cet article fournit des instructions concernant l’utilisation de l’outil DistCp.

DistCp offre différents paramètres de ligne de commande, et nous vous encourageons vivement à lire cet article afin d’optimiser votre utilisation de cet outil. Cet article explique les fonctionnalités de base tout en mettant l’accent sur son utilisation pour la copie de données vers un compte avec espace de noms hiérarchique activé.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* Un compte Stockage Azure existant sans fonctionnalité Data Lake Storage Gen2 (espace de noms hiérarchique) activée.
* Un compte Stockage Azure avec fonctionnalités Data Lake Storage Gen2 (espace de noms hiérarchique) activées. Pour obtenir des instructions sur la façon d’en créer un, consultez [Créer un compte de stockage Azure](../common/storage-account-create.md)
* Un conteneur créé dans le compte de stockage avec l’espace de noms hiérarchique activé.
* Un cluster Azure HDInsight ayant accès à un compte de stockage avec la fonctionnalité d’espace de noms hiérarchique activée. Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). Veillez à activer le Bureau à distance pour le cluster.

## <a name="use-distcp-from-an-hdinsight-linux-cluster"></a>Utiliser DistCp à partir d’un cluster HDInsight Linux

Un cluster HDInsight est fourni avec l’utilitaire DistCp, que vous pouvez utiliser pour copier dans un cluster HDInsight des données provenant de différentes sources. Si vous avez configuré le cluster HDInsight pour utiliser à la fois le Stockage Blob Azure et Azure Data Lake Storage, vous pouvez aussi vous servir de l’utilitaire DistCp tel quel pour copier des données de l’un à l’autre. Dans cette section, nous allons voir comment utiliser l’utilitaire DistCp.

1. Créez une session SSH sur votre cluster HDI. Consultez [Connexion à un cluster HDInsight sous Linux](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

2. Vérifiez si vous pouvez accéder à votre compte V2 universel existant (sans espace de noms hiérarchique activé).

    ```bash
    hdfs dfs –ls wasbs://<container-name>@<storage-account-name>.blob.core.windows.net/
    ```

   Vous devriez obtenir la liste du contenu du conteneur.

3. De même, vérifiez si vous pouvez accéder au compte de stockage avec espace de noms hiérarchique activé à partir du cluster. Exécutez la commande suivante :

    ```bash
    hdfs dfs -ls abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/
    ```

    La sortie doit fournir la liste des fichiers/dossiers du compte Data Lake Storage.

4. Utilisez DistCp pour copier des données de WASB vers un compte Data Lake Storage.

    ```bash
    hadoop distcp wasbs://<container-name>@<storage-account-name>.blob.core.windows.net/example/data/gutenberg abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/myfolder
    ```

    Cette commande copie le contenu du dossier **/example/data/gutenberg/** du stockage Blob vers **/myfolder** dans le compte Data Lake Storage.

5. De même, utilisez DistCp pour copier des données d’un compte Data Lake Storage vers le Stockage Blob (WASB).

    ```bash
    hadoop distcp abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/myfolder wasbs://<container-name>@<storage-account-name>.blob.core.windows.net/example/data/gutenberg
    ```

    Cette commande copie le contenu du répertoire **/myfolder** du compte Data Lake Store vers le dossier **/example/data/gutenberg/** de WASB.

## <a name="performance-considerations-while-using-distcp"></a>Considérations de performances lors de l’utilisation de DistCp

La granularité la plus basse de DistCp étant un fichier unique, la définition du nombre maximal de copies simultanées est le paramètre le plus important à optimiser par rapport à Data Lake Storage. Le nombre de copies simultanées est égal au paramètre de nombre de mappeurs (**m**) sur la ligne de commande. Ce paramètre spécifie le nombre maximal de mappeurs utilisés pour copier les données. La valeur par défaut est 20.

**Exemple**

```bash
hadoop distcp -m 100 wasbs://<container-name>@<storage-account-name>.blob.core.windows.net/example/data/gutenberg abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/myfolder
```

### <a name="how-do-i-determine-the-number-of-mappers-to-use"></a>Comment déterminer le nombre de mappeurs à utiliser ?

Voici quelques conseils à suivre.

* **Étape 1 : Déterminer la mémoire totale disponible pour la file d’attente d’application YARN « par défaut »**  - La première étape consiste à déterminer la quantité de mémoire disponible pour la file d’attente d’application YARN « par défaut ». Ces informations sont disponibles dans le portail de Ambari associé au cluster. Accédez à YARN et affichez l’onglet Configurations pour voir la mémoire YARN disponible pour la file d’attente d’application « par défaut ». Il s’agit de la mémoire totale disponible pour votre travail DistCp (qui est en fait un travail MapReduce).

* **Étape 2 : Calculer le nombre de mappeurs** - La valeur de **m** est égale au quotient de la mémoire YARN totale divisée par la taille du conteneur YARN. Les informations sur la taille du conteneur YARN sont également disponibles dans le portail Ambari. Accédez à YARN et affichez l’onglet Configurations. La taille du conteneur YARN s’affiche dans cette fenêtre. L’équation pour obtenir le nombre de mappeurs (**m**) est

    m = (nombre de nœuds * mémoire YARN pour chaque nœud) / Taille du conteneur YARN

**Exemple**

Supposons que vous avez un cluster 4x D14v2s et que vous essayez de transférer 10 To de données à partir de 10 dossiers différents. Chaque dossier contient différentes quantités de données, et la taille des fichiers dans chaque dossier est différente.

* **Total de mémoire YARN** : à partir du portail Ambari, vous déterminez que la mémoire YARN est de 96 Go pour un nœud D14. Ainsi, la mémoire YARN totale pour un cluster à quatre nœuds est : 

    Mémoire YARN = 4 * 96 Go = 384 Go

* **Nombre de mappeurs** : à partir du portail Ambari, vous déterminez que la taille du conteneur YARN est de 3 072 Mo pour un nœud de cluster D14. Par conséquent, le nombre de mappeurs est :

    m = (4 nœuds * 96 Go) / 3 072 Mo = 128 mappeurs

Si d’autres applications utilisent de la mémoire, vous pouvez choisir d’utiliser uniquement une partie de la mémoire de YARN de votre cluster pour DistCp.

### <a name="copying-large-datasets"></a>Copie de jeux de données volumineux

Lorsque la taille du jeu de données à déplacer est volumineuse (par exemple > 1 To) ou si vous avez de nombreux dossiers, vous devez envisager d’utiliser plusieurs tâches DistCp. Il n’y a probablement aucun gain de performance, mais cela répartit les tâches. Ainsi, si une tâche échoue, il vous suffit de redémarrer la tâche en question plutôt que l’ensemble complet.

### <a name="limitations"></a>Limites

* DistCp tente de créer des mappeurs similaires en taille pour optimiser les performances. L’augmentation du nombre de mappeurs n’améliore pas nécessairement les performances.

* DistCp est limité à un seul mappeur par fichier. Par conséquent, vous ne devez pas avoir plus de mappeurs que de fichiers. Étant donné que DistCp peut affecter un seul mappeur par fichier, cela limite la quantité d’exécutions simultanées utilisables pour copier des fichiers volumineux.

* Si vous avez un petit nombre de fichiers volumineux, vous devez les fractionner en plusieurs segments de fichier de 256 Mo pour avoir un meilleur potentiel de simultanéité.