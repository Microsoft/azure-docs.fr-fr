---
title: Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight
description: Découvrez comment utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020, devx-track-azurecli
ms.date: 04/24/2020
ms.openlocfilehash: cc17cd23ae197db25fed440eb249f2cf069d4859
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92744582"
---
# <a name="use-azure-data-lake-storage-gen2-with-azure-hdinsight-clusters"></a>Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight

[Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md) est un service de stockage cloud dédié à l’analytique Big Data et intégré au [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md). Data Lake Storage Gen2 réunit les capacités du Stockage Blob Azure et d’Azure Data Lake Storage Gen1. Le service qui en résulte offre les fonctionnalités d’Azure Data Lake Storage Gen1, notamment la sémantique du système de fichiers, la sécurité au niveau des répertoires et des fichiers et l’adaptabilité. En ce qui concerne les fonctionnalités à faible coût du stockage Blob Azure, elles comprennent le stockage hiérarchisé, la haute disponibilité et la récupération d’urgence.

Pour une comparaison complète des options de création de cluster à l'aide de Data Lake Storage Gen2, consultez [Comparer les options de stockage à utiliser avec les clusters Azure HDInsight](hdinsight-hadoop-compare-storage-options.md).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="data-lake-storage-gen2-availability"></a>Disponibilité de Data Lake Storage Gen2

Data Lake Storage Gen2 est disponible comme option de stockage pour presque tous les types de clusters Azure HDInsight en tant que compte par défaut et compte de stockage supplémentaire. HBase, en revanche, ne peut avoir qu’un seul compte avec Data Lake Storage Gen2.

> [!Note]  
> Après avoir sélectionné Data Lake Storage Gen2 comme **type de stockage principal** , il n’est pas possible de sélectionner Data Lake Storage Gen1 comme stockage supplémentaire.

## <a name="create-hdinsight-clusters-using-data-lake-storage-gen2"></a>Création de clusters HDInsight avec Data Lake Storage Gen2

Suivez les liens ci-dessous pour obtenir des instructions détaillées sur la création de clusters HDInsight avec accès à Data Lake Storage Gen2.

* [Utilisation du portail](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2-portal.md)
* [Utilisation de l’interface de ligne de commande Azure](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2-azure-cli.md)
* PowerShell n’est pas pris en charge actuellement pour créer un cluster HDInsight avec Azure Data Lake Storage Gen2.

## <a name="access-control-for-data-lake-storage-gen2-in-hdinsight"></a>Contrôle d’accès à Data Lake Storage Gen2 dans HDInsight

### <a name="what-kinds-of-permissions-does-data-lake-storage-gen2-support"></a>Quels sont les types d’autorisation pris en charge par Data Lake Storage Gen2 ?

Data Lake Storage Gen2 utilise un modèle de contrôle d’accès qui prend en charge le contrôle d’accès en fonction du rôle Azure (RBAC Azure) et les listes de contrôle d’accès (ACL) POSIX. Data Lake Storage Gen1 prend en charge les listes de contrôle d’accès uniquement pour contrôler l’accès aux données.

Azure RBAC utilise des attributions de rôles pour appliquer efficacement des ensembles d’autorisations aux utilisateurs, groupes et principaux de service des ressources Azure. En règle générale, ces ressources Azure sont limitées aux ressources de niveau supérieur (par exemple, les comptes de stockage Blob Azure). Pour le Stockage Blob Azure, de même que Data Lake Storage Gen2, ce mécanisme a été étendu à la ressource du système de fichiers.

Pour plus d’informations sur les autorisations de fichiers avec Azure RBAC, consultez [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../storage/blobs/data-lake-storage-access-control-model.md#role-based-access-control).

Pour plus d’informations sur les autorisations de fichiers avec des listes ACL, consultez [Listes de contrôle d’accès sur les fichiers et répertoires](../storage/blobs/data-lake-storage-access-control.md).

### <a name="how-do-i-control-access-to-my-data-in-data-lake-storage-gen2"></a>Comment contrôler l’accès à mes données dans Data Lake Storage Gen2 ?

La capacité de votre cluster HDInsight à accéder aux fichiers dans Data Lake Storage Gen2 est contrôlée par le biais d’identités managées. Une identité managée est une identité inscrite dans Azure Active Directory (Azure AD) dont les informations d’identification sont gérées par Azure. Grâce aux identités managées, il n’est pas nécessaire d’inscrire des principaux de service dans Azure AD, ni de gérer des informations d’identification, par exemple des certificats.

Les services Azure utilisent deux types d’identités managées : celles affectées par le système et celles affectées par l’utilisateur. HDInsight utilise des identités managées affectées par l’utilisateur pour accéder à Data Lake Storage Gen2. Une `user-assigned managed identity` est créée en tant que ressource Azure autonome. Via un processus de création, Azure crée une identité dans le locataire Azure AD approuvé par l’abonnement en cours d’utilisation. Une fois l’identité créée, elle peut être affectée à une ou plusieurs instances de service Azure.

Le cycle de vie d’une identité attribuée par l’utilisateur est géré séparément du cycle de vie des instances de service Azure auxquelles elle est affectée. Pour en savoir plus sur les identités managées, consultez la section [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md).

### <a name="how-do-i-set-permissions-for-azure-ad-users-to-query-data-in-data-lake-storage-gen2-by-using-hive-or-other-services"></a>Comment définir des autorisations pour que les utilisateurs Azure AD puissent interroger des données dans Data Lake Storage Gen2 avec Hive ou d’autres services ?

Pour définir des autorisations pour que les utilisateurs puissent interroger des données, utilisez les groupes de sécurité Azure AD comme principal affecté dans les listes de contrôle d’accès. N’attribuez pas directement des autorisations d’accès aux fichiers à des utilisateurs ou principaux de service individuels. Grâce aux groupes de sécurité Azure AD qui contrôlent le flux d’autorisations, vous pouvez ajouter et supprimer des utilisateurs ou des principaux de service sans réappliquer des ACL à une structure de répertoires entière. Il vous suffit d’ajouter ou de supprimer les utilisateurs du groupe de sécurité Azure AD approprié. Les listes de contrôle d’accès ne sont pas héritées. Par conséquent, pour les réappliquer, vous devez les mettre à jour dans chaque fichier et sous-répertoire.

## <a name="access-files-from-the-cluster"></a>Accès aux fichiers à partir du cluster

Il existe plusieurs méthodes pour accéder aux fichiers dans Data Lake Storage Gen2 à partir d’un cluster HDInsight.

* **Utilisation du nom complet** . Avec cette approche, vous fournissez le chemin d’accès complet au fichier auquel vous souhaitez accéder.

    ```
    abfs://<containername>@<accountname>.dfs.core.windows.net/<file.path>/
    ```

* **Utilisation du format de chemin d’accès raccourci** . Avec cette approche, vous remplacez le chemin d’accès à la racine du cluster par :

    ```
    abfs:///<file.path>/
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
    hdfs dfs -mkdir abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -mkdir abfs:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. Copiez les données du stockage local vers le stockage en cluster.

    ```bash
    hdfs dfs -copyFromLocal testFile.txt  abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt  abfs:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt  /sampledata3/
    ```

1. Affichez le contenu du répertoire sur le stockage en cluster.

    ```bash
    hdfs dfs -ls abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -ls abfs:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

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
LOCATION 'abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/example/data/';
LOCATION 'abfs:///example/data/';
LOCATION '/example/data/';
```

## <a name="next-steps"></a>Étapes suivantes

* [Azure HDInsight integration with Data Lake Storage Gen2 preview - ACL and security update](https://azure.microsoft.com/blog/azure-hdinsight-integration-with-data-lake-storage-gen-2-preview-acl-and-security-update/)
* [Présentation d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Tutoriel : Extraire, transformer et charger des données à l’aide d’Interactive Query sur Azure HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
