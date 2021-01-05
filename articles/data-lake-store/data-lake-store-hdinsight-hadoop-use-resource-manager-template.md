---
title: Modèle – Cluster HDInsight avec Data Lake Storage Gen1
description: Utilisez des modèles Azure Resource Manager pour créer et utiliser des clusters Azure HDInsight avec Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: a7283ad4c4c61ecc293a55ffc4cb9626bb28d630
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108726"
---
# <a name="create-an-hdinsight-cluster-with-azure-data-lake-storage-gen1-using-azure-resource-manager-template"></a>Créer un cluster HDInsight avec Data Lake Storage Gen1 à l’aide d’un modèle Azure Resource Manager
> [!div class="op_single_selector"]
> * [Utilisation du portail](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [Utilisation de PowerShell (pour le stockage par défaut)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [Utilisation de PowerShell (pour le stockage supplémentaire)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [Utilisation du gestionnaire des ressources](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)
>
>

Découvrez comment utiliser Azure PowerShell pour configurer un cluster HDInsight avec Azure Data Lake Storage Gen1 **comme stockage supplémentaire**.

Pour les types de clusters pris en charge, Data Lake Storage Gen1 peut être utilisé comme compte de stockage par défaut ou supplémentaire. Lorsque Data Lake Storage Gen1 est utilisé comme espace de stockage supplémentaire, le compte de stockage par défaut pour les clusters est toujours le Stockage Blob Azure (WASB), et les fichiers associés au cluster (par exemple, les journaux d’activité, etc.) sont écrits dans le stockage par défaut, tandis que les données que vous souhaitez traiter peuvent être stockées dans un compte Data Lake Storage Gen1. L’utilisation de Data Lake Storage Gen1 en tant que compte de stockage supplémentaire n’affecte pas les performances ni la capacité de lecture/écriture sur le stockage à partir du cluster.

## <a name="using-data-lake-storage-gen1-for-hdinsight-cluster-storage"></a>Utilisation de Data Lake Storage Gen1 pour le stockage de cluster HDInsight

Voici quelques considérations importantes pour l’utilisation de HDInsight avec Data Lake Storage Gen1 :

* L’option permettant de créer des clusters HDInsight avec accès à Data Lake Storage Gen1 comme stockage par défaut est disponible si vous utilisez HDInsight versions 3.5 et 3.6.

* L’option permettant de créer des clusters HDInsight avec accès au Data Lake Storage Gen1 comme stockage supplémentaire est disponible si vous utilisez HDInsight versions 3.2, 3.4, 3.5 et 3.6.

Dans cet article, nous approvisionnons un cluster Hadoop avec Data Lake Storage Gen1 comme stockage supplémentaire. Pour obtenir des instructions sur la création d’un cluster Hadoop avec Data Lake Storage Gen1 comme stockage par défaut, consultez [Créer un cluster HDInsight avec Data Lake Storage Gen1 à l’aide du portail Azure](data-lake-store-hdinsight-hadoop-use-portal.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1.0 ou version ultérieure**. Consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).
* **Principal du service Azure Active Directory**. Les étapes de ce didacticiel indiquent comment créer un principal du service dans Azure AD. Toutefois, vous devez être administrateur Azure AD pour pouvoir créer un principal du service. Si vous êtes administrateur Azure AD, vous pouvez ignorer ce prérequis et poursuivre le didacticiel.

    **Si vous n’êtes pas administrateur Azure AD**, vous ne pouvez pas effectuer les étapes nécessaires à la création d’un principal du service. Dans ce cas, votre administrateur Azure AD doit d’abord créer un principal de service. Vous pourrez ensuite créer un cluster HDInsight avec Data Lake Storage Gen1. En outre, le principal du service doit être créé à l’aide d’un certificat, comme décrit dans [Create a service principal with certificate](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-certificate-from-certificate-authority) (Créer un principal du service avec certificat).

## <a name="create-an-hdinsight-cluster-with-data-lake-storage-gen1"></a>Créer un cluster HDInsight avec Data Lake Storage Gen1
Le modèle Resource Manager et les prérequis pour l’utiliser sont disponibles sur GitHub à la page [Déployer un cluster HDInsight Linux avec un nouveau compte Data Lake Storage Gen1](https://github.com/Azure/azure-quickstart-templates/tree/master/201-hdinsight-datalake-store-azure-storage). Suivez les instructions de ce lien pour créer un cluster HDInsight avec Azure Data Lake Storage Gen1 comme stockage supplémentaire.

Les instructions du lien mentionné ci-dessus requièrent PowerShell. Avant de commencer à suivre ces instructions, vérifiez que vous vous connectez à votre compte Azure. Sur votre bureau, ouvrez une nouvelle fenêtre Azure PowerShell et entrez les extraits de code suivants. À l’invite de connexion, vérifiez que vous vous connectez en tant qu’administrateur/propriétaire de l’abonnement :

```
# Log in to your Azure account
Connect-AzAccount

# List all the subscriptions associated to your account
Get-AzSubscription

# Select a subscription
Set-AzContext -SubscriptionId <subscription ID>
```

Le modèle déploie ces types de ressources :

* [Microsoft.DataLakeStore/accounts](/azure/templates/microsoft.datalakestore/accounts)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.HDInsight/clusters](/azure/templates/microsoft.hdinsight/clusters)

## <a name="upload-sample-data-to-data-lake-storage-gen1"></a>Télécharger des exemples de données sur Data Lake Storage Gen1
Le modèle Resource Manager crée un compte de stockage avec Data Lake Storage Gen1 et l’associe au cluster HDInsight. À présent, vous devez charger des exemples de données sur le Data Lake Storage Gen1. Vous aurez besoin de ces données plus loin dans ce tutoriel afin d’exécuter des travaux à partir d’un cluster HDInsight ayant accès aux données du compte de stockage avec Data Lake Storage Gen1. Pour plus d’informations sur le chargement de données, consultez [Charger un fichier dans Data Lake Storage Gen1](data-lake-store-get-started-portal.md#uploaddata). Si vous recherchez des exemples de données à charger, vous pouvez récupérer le dossier **Données Ambulance** dans le [Référentiel Git Azure Data Lake](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData).

## <a name="set-relevant-acls-on-the-sample-data"></a>Définir les ACL appropriées sur les exemples de données
Afin de s’assurer que les exemples de données que vous chargez soient accessibles à partir du cluster HDInsight, vous devez vérifier que l’application Azure AD qui est utilisée pour établir l’identité entre le cluster HDInsight et Data Lake Storage Gen1 a accès au fichier/dossier auquel vous essayez d’accéder. Pour ce faire, procédez comme suit.

1. Recherchez le nom de l’application Azure AD qui est associée au cluster HDInsight et au compte de stockage avec Data Lake Storage Gen1. Pour ce faire, vous pouvez ouvrir le panneau du cluster HDInsight que vous avez créé à l’aide du modèle Resource Manager, cliquer sur l’onglet **Identité Azure AD de cluster** et rechercher la valeur de **Nom d’affichage du principal du service**.
2. À présent, fournissez un accès à cette application Azure AD sur le fichier/dossier auquel vous souhaitez accéder à partir du cluster HDInsight. Pour définir les ACL adéquates sur le fichier/dossier dans Data Lake Storage Gen1, consultez [Sécurisation des données dans Data Lake Storage Gen1](data-lake-store-secure-data.md#filepermissions).

## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-data-lake-storage-gen1"></a>Exécuter des tâches de test sur le cluster HDInsight pour utiliser Data Lake Storage Gen1
Après avoir configuré un cluster HDInsight, vous pouvez exécuter des travaux test sur le cluster pour vérifier que le cluster HDInsight peut accéder à Data Lake Storage Gen1. Pour ce faire, nous allons exécuter un exemple de tâche Hive qui crée une table avec les exemples de données que vous avez chargés précédemment dans votre compte de stockage avec Data Lake Storage Gen1.

Dans cette section, vous allez utiliser le protocole SSH pour accéder à un cluster HDInsight Linux et exécuter l’exemple de requête Hive. Si vous utilisez un client Windows, nous vous recommandons d’utiliser **PuTTY**, qui peut être téléchargé sur [https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Pour plus d’informations sur l’utilisation de PuTTY, consultez la rubrique [Utilisation de SSH avec Hadoop Linux dans HDInsight à partir de Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Une fois connecté, démarrez l'interface de ligne de commande Hive à l'aide de la commande suivante :

   ```
   hive
   ```
2. À l’aide de l’interface de ligne de commande, entrez les instructions suivantes pour créer une table nommée **vehicles** en utilisant les exemples de données dans Data Lake Storage Gen1 :

   ```
   DROP TABLE vehicles;
   CREATE EXTERNAL TABLE vehicles (str string) LOCATION 'adl://<mydatalakestoragegen1>.azuredatalakestore.net:443/';
   SELECT * FROM vehicles LIMIT 10;
   ```

   Vous devez obtenir une sortie similaire à la suivante :

   ```
   1,1,2014-09-14 00:00:03,46.81006,-92.08174,51,S,1
   1,2,2014-09-14 00:00:06,46.81006,-92.08174,13,NE,1
   1,3,2014-09-14 00:00:09,46.81006,-92.08174,48,NE,1
   1,4,2014-09-14 00:00:12,46.81006,-92.08174,30,W,1
   1,5,2014-09-14 00:00:15,46.81006,-92.08174,47,S,1
   1,6,2014-09-14 00:00:18,46.81006,-92.08174,9,S,1
   1,7,2014-09-14 00:00:21,46.81006,-92.08174,53,N,1
   1,8,2014-09-14 00:00:24,46.81006,-92.08174,63,SW,1
   1,9,2014-09-14 00:00:27,46.81006,-92.08174,4,NE,1
   1,10,2014-09-14 00:00:30,46.81006,-92.08174,31,N,1
   ```


## <a name="access-data-lake-storage-gen1-using-hdfs-commands"></a>Accéder à Data Lake Storage Gen1 avec les commandes HDFS
Une fois que vous avez configuré le cluster HDInsight pour qu’il utilise Data Lake Storage Gen1, vous pouvez utiliser les commandes de l’interpréteur de commandes HDFS pour accéder au magasin.

Dans cette section, vous allez utiliser le protocole SSH pour accéder à un cluster HDInsight Linux et exécuter les commandes HDFS. Si vous utilisez un client Windows, nous vous recommandons d’utiliser **PuTTY**, qui peut être téléchargé sur [https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Pour plus d’informations sur l’utilisation de PuTTY, consultez la rubrique [Utilisation de SSH avec Hadoop Linux dans HDInsight à partir de Windows](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

Une fois connecté, utilisez la commande suivante du système de fichiers HDFS pour lister les fichiers du compte de stockage avec Data Lake Storage Gen1.

```
hdfs dfs -ls adl://<storage account with Data Lake Storage Gen1 name>.azuredatalakestore.net:443/
```

Le fichier que vous avez téléchargé dans Data Lake Storage Gen1 doit y figurer.

```
15/09/17 21:41:15 INFO web.CaboWebHdfsFileSystem: Replacing original urlConnectionFactory with org.apache.hadoop.hdfs.web.URLConnectionFactory@21a728d6
Found 1 items
-rwxrwxrwx   0 NotSupportYet NotSupportYet     671388 2015-09-16 22:16 adl://mydatalakestoragegen1.azuredatalakestore.net:443/mynewfolder
```

Vous pouvez également utiliser la commande `hdfs dfs -put` pour charger des fichiers dans Data Lake Storage Gen1, puis utiliser `hdfs dfs -ls` pour vérifier si les fichiers ont été chargés avec succès.


## <a name="next-steps"></a>Étapes suivantes
* [Copier des données d’objets blob Stockage Azure vers Data Lake Storage Gen1](data-lake-store-copy-data-wasb-distcp.md)
* [Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen1.md)