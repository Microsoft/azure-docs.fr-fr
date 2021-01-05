---
title: Copier des données vers et depuis WASB dans Azure Data Lake Storage Gen1 à l’aide de DistCp
description: Utilisez l’outil DistCp pour copier des données entre des objets blob de stockage Azure et Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 01/03/2020
ms.author: twooley
ms.openlocfilehash: c608f357eb1eff9fd36e583b98d26250a71cb923
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85515686"
---
# <a name="use-distcp-to-copy-data-between-azure-storage-blobs-and-azure-data-lake-storage-gen1"></a>Utiliser l’outil DistCp pour copier des données entre des objets blob de stockage Azure et Azure Data Lake Storage Gen1

> [!div class="op_single_selector"]
> * [Utilisation de DistCp](data-lake-store-copy-data-wasb-distcp.md)
> * [Utilisation d’AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
>
>

Si vous avez un cluster HDInsight avec un accès à Azure Data Lake Storage Gen1, vous pouvez utiliser des outils de l’écosystème Hadoop comme DistCp pour copier des données vers et à partir d’un stockage de cluster HDInsight (WASB) dans un compte Data Lake Storage Gen1. Cet article explique comment utiliser l’outil DistCp.

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Cluster Azure HDInsight** avec accès à un compte Data Lake Storage Gen1. Voir [Créer un cluster HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Veillez à activer le Bureau à distance pour le cluster.

## <a name="use-distcp-from-an-hdinsight-linux-cluster"></a>Utiliser DistCp à partir d’un cluster HDInsight Linux

Un cluster HDInsight est fourni avec l’outil DistCp, que vous pouvez utiliser pour copier dans un cluster HDInsight des données provenant de différentes sources. Si vous avez configuré le cluster HDInsight pour utiliser Data Lake Storage Gen1 comme espace de stockage supplémentaire, vous pouvez utiliser DistCp tel quel pour copier des données vers et depuis un compte Data Lake Storage Gen1. Dans cette section, nous allons voir comment utiliser l’outil DistCp.

1. À partir de votre Bureau, utilisez SSH pour la connexion au cluster. Consultez [Connexion à un cluster HDInsight sous Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md). Exécutez les commandes de l’invite SSH.

1. Vérifiez si vous pouvez accéder aux objets blob de stockage Azure (WASB). Exécutez la commande suivante :

   ```
   hdfs dfs –ls wasb://<container_name>@<storage_account_name>.blob.core.windows.net/
   ```

   Vous obtenez la liste du contenu de l’objet blob de stockage.

1. Là encore, vérifiez si vous pouvez accéder au compte Data Lake Storage Gen1 à partir du cluster. Exécutez la commande suivante :

   ```
   hdfs dfs -ls adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/
   ```

    Cette commande fournit la liste des fichiers et dossiers du compte Data Lake Storage Gen1.

1. Utilisez DistCp pour copier des données de WASB vers un compte Data Lake Storage Gen1.

   ```
   hadoop distcp wasb://<container_name>@<storage_account_name>.blob.core.windows.net/example/data/gutenberg adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/myfolder
   ```

    Cette commande copie le contenu du dossier **/example/data/gutenberg/** de WASB vers le répertoire **/myfolder** du compte Data Lake Storage Gen1.

1. De la même manière, utilisez DistCp pour copier des données d’un compte Data Lake Storage Gen1 vers WASB.

   ```
   hadoop distcp adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/myfolder wasb://<container_name>@<storage_account_name>.blob.core.windows.net/example/data/gutenberg
   ```

    Cette commande copie le contenu du répertoire **/myfolder** du compte Data Lake Storage Gen1 vers le dossier **/example/data/gutenberg/** de WASB.

## <a name="performance-considerations-while-using-distcp"></a>Considérations de performances lors de l’utilisation de DistCp

La précision la plus basse de l’outil DistCp étant un fichier unique, la définition du nombre maximal de copies simultanées est le paramètre le plus important à optimiser par rapport à Data Lake Storage Gen1. Vous pouvez contrôler le nombre de copies simultanées en définissant le paramètre de nombre de mappeurs (« m ») sur la ligne de commande. Ce paramètre spécifie le nombre maximal de mappeurs utilisés pour copier les données. La valeur par défaut est 20.

Exemple :

```
 hadoop distcp wasb://<container_name>@<storage_account_name>.blob.core.windows.net/example/data/gutenberg adl://<data_lake_storage_gen1_account>.azuredatalakestore.net:443/myfolder -m 100
```

### <a name="how-to-determine-the-number-of-mappers-to-use"></a>Méthode permettant de déterminer le nombre de mappeurs à utiliser

Voici quelques conseils à suivre.

* **Étape 1 : Déterminer la quantité totale de mémoire YARN** : la première étape consiste à déterminer la mémoire YARN disponible pour le cluster sur lequel vous exécutez la tâche DistCp. Ces informations sont disponibles dans le portail de Ambari associé au cluster. Accédez à YARN et affichez l’onglet **Configurations** pour voir la mémoire YARN. Pour obtenir la mémoire YARN totale, multipliez la mémoire YARN par nœud par le nombre de nœuds dans votre cluster.

* **Étape 2 : Calculer le nombre de mappeurs** - La valeur de **m** est égale au quotient de la mémoire YARN totale divisée par la taille du conteneur YARN. Les informations sur la taille du conteneur YARN sont également disponibles dans le portail Ambari. Accédez à YARN et affichez l’onglet **Configurations**. La taille du conteneur YARN s’affiche dans cette fenêtre. L’équation pour obtenir le nombre de mappeurs (**m**) est :

   `m = (number of nodes * YARN memory for each node) / YARN container size`

Exemple :

Supposons que vous avez quatre nœuds D14v2s dans le cluster et que vous voulez transférer 10 To de données à partir de 10 dossiers différents. Chaque dossier contient différentes quantités de données, et la taille des fichiers dans chaque dossier est différente.

* Total de mémoire YARN : à partir du portail Ambari, vous déterminez que la mémoire YARN est de 96 Go pour un nœud D14. Ainsi, la mémoire YARN totale pour un cluster à quatre nœuds est :

   `YARN memory = 4 * 96GB = 384GB`

* Nombre de mappeurs : à partir du portail Ambari, vous déterminez que la taille du conteneur YARN est de 3072 pour un nœud de cluster D14. Par conséquent, le nombre de mappeurs est :

   `m = (4 nodes * 96GB) / 3072MB = 128 mappers`

Si d’autres applications utilisent de la mémoire, vous pouvez choisir d’utiliser uniquement une partie de la mémoire YARN de votre cluster pour DistCp.

### <a name="copying-large-datasets"></a>Copie de jeux de données volumineux

Lorsque la taille du jeu de données à déplacer est volumineuse (par exemple, > 1 To) ou si vous avez de nombreux dossiers différents, envisagez d’utiliser plusieurs travaux DistCp. Il n’y a probablement aucun gain de performance, mais cela répartit les travaux. Ainsi, si un travail échoue, il vous suffit de redémarrer le travail en question à la place du travail complet.

### <a name="limitations"></a>Limites

* DistCp tente de créer des mappeurs similaires en taille pour optimiser les performances. L’augmentation du nombre de mappeurs n’améliore pas nécessairement les performances.

* DistCp est limité à un seul mappeur par fichier. Par conséquent, vous ne devez pas avoir plus de mappeurs que de fichiers. Comme DistCp peut affecter un seul mappeur à un fichier, cela limite la quantité d’exécutions simultanées utilisables pour copier des fichiers volumineux.

* Si vous avez un petit nombre de fichiers volumineux, divisez-les en segments de fichier de 256 Mo pour avoir un meilleur potentiel de simultanéité.

* Si vous copiez à partir d’un compte de stockage d’objets blob Azure, votre travail de copie peut être limité côté stockage d’objets blob. Cela dégrade les performances de votre tâche de copie. Pour en savoir plus sur les limites du stockage d’objets blob Azure, consultez les limites du stockage Azure dans [Abonnement Azure et limites du service](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="see-also"></a>Voir aussi

* [Copier des données à partir d’objets blob de stockage Azure vers Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
