---
title: Créer Azure HDInsight – Azure Data Lake Storage Gen2 – Azure CLI
description: Découvrez comment utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight en utilisant Azure CLI.
author: guyhay
ms.author: guyhay
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020, devx-track-azurecli
ms.date: 09/17/2020
ms.openlocfilehash: 4ffe5acce831435d3771a2eb48a90b16f584c92a
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97704008"
---
# <a name="create-a-cluster-with-data-lake-storage-gen2-using-azure-cli"></a>Créer un cluster avec Data Lake Storage Gen2 à l’aide d’Azure CLI

Pour créer un cluster HDInsight qui utilise Data Lake Storage Gen2 comme stockage, effectuez les étapes suivantes.

## <a name="prerequisites"></a>Prérequis

- Si vous ne connaissez pas Azure Data Lake Storage Gen2, consultez la [section Vue d’ensemble](hdinsight-hadoop-use-data-lake-storage-gen2.md). 
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour exécuter les exemples de script d’Azure CLI, vous disposez de trois options :
    - Utilisez [Azure Cloud Shell](../cloud-shell/overview.md) à partir du portail Azure (voir section suivante).
    - Utilisez l’interface intégrée Azure Cloud Shell via le bouton « Essayer », situé dans le coin supérieur droit de chaque bloc de code.
    - [Installez la dernière version d’Azure CLI](/cli/azure/install-azure-cli) (2.0.13 ou ultérieure) si vous préférez utiliser une console CLI locale. Connectez-vous à Azure avec la commande `az login` en utilisant un compte associé à l’abonnement Azure sous lequel vous souhaitez déployer l’identité managée affectée par l’utilisateur.Azure CLI. 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Vous pouvez [télécharger un exemple de fichier de modèle](https://github.com/Azure-Samples/hdinsight-data-lake-storage-gen2-templates/blob/master/hdinsight-adls-gen2-template.json) et [télécharger un exemple de fichier de paramètres](https://github.com/Azure-Samples/hdinsight-data-lake-storage-gen2-templates/blob/master/parameters.json). Avant d’utiliser le modèle et l’extrait de code Azure CLI ci-dessous, remplacez les espaces réservés suivants par leurs valeurs correctes :

| Espace réservé | Description |
|---|---|
| `<SUBSCRIPTION_ID>` | L’ID de votre abonnement Azure |
| `<RESOURCEGROUPNAME>` | Le groupe de ressources dans lequel vous souhaitez créer le cluster et le compte de stockage. |
| `<MANAGEDIDENTITYNAME>` | Nom de l’identité managée qui recevra les autorisations sur votre compte de stockage avec Azure Data Lake Storage Gen2. |
| `<STORAGEACCOUNTNAME>` | Nouveau compte de stockage avec Azure Data Lake Storage Gen2 qui sera créé. |
| `<FILESYSTEMNAME>`  | Nom du système de fichiers que ce cluster doit utiliser dans le compte de stockage. |
| `<CLUSTERNAME>` | Nom de votre cluster HDInsight |
| `<PASSWORD>` | Le mot de passe que vous avez choisi pour vous connecter au cluster à l’aide de SSH et du tableau de bord Ambari. |

L’extrait de code ci-dessous effectue les étapes initiales suivantes :

1. Il établit la connexion à votre compte Azure.
1. Il définit l’abonnement actif où les opérations de création seront effectuées.
1. Il crée un groupe de ressources pour les nouvelles activités de déploiement.
1. Il crée une identité managée attribuée par l’utilisateur.
1. Il ajoute une extension à Azure CLI pour utiliser les fonctionnalités de Data Lake Storage Gen2.
1. Il crée un compte de stockage avec Data Lake Storage Gen2 en utilisant l’indicateur `--hierarchical-namespace true`.

```azurecli
az login
az account set --subscription <SUBSCRIPTION_ID>

# Create resource group
az group create --name <RESOURCEGROUPNAME> --location eastus

# Create managed identity
az identity create -g <RESOURCEGROUPNAME> -n <MANAGEDIDENTITYNAME>

az extension add --name storage-preview

az storage account create --name <STORAGEACCOUNTNAME> \
    --resource-group <RESOURCEGROUPNAME> \
    --location eastus --sku Standard_LRS \
    --kind StorageV2 --hierarchical-namespace true
```

Ensuite, connectez-vous au portail. Ajoutez la nouvelle identité managée affectée par l’utilisateur au rôle **Contributeur aux données Blob du stockage** sur le compte de stockage. Cette étape est décrite à l’étape 3 sous [Utilisation du Portail Azure](hdinsight-hadoop-use-data-lake-storage-gen2.md).

 > [!IMPORTANT]
 > Assurez-vous que votre compte de stockage dispose de l’identité affectée par l’utilisateur avec les autorisations de rôle **Contributeur aux données Blob du stockage**. Sinon, la création du cluster échouera.

```azurecli
az deployment group create --name HDInsightADLSGen2Deployment \
    --resource-group <RESOURCEGROUPNAME> \
    --template-file hdinsight-adls-gen2-template.json \
    --parameters parameters.json
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir terminé ce tutoriel, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées dans le stockage Azure. Vous pouvez ainsi supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même quand vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, mieux vaut supprimer les clusters quand ils ne sont pas utilisés.

Entrez tout ou partie des commandes suivantes pour supprimer des ressources :

```azurecli-interactive
# Remove cluster
az hdinsight delete \
    --name $clusterName \
    --resource-group $resourceGroupName

# Remove storage container
az storage container delete \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --name $AZURE_STORAGE_CONTAINER

# Remove storage account
az storage account delete \
    --name $AZURE_STORAGE_ACCOUNT \
    --resource-group $resourceGroupName

# Remove resource group
az group delete \
    --name $resourceGroupName
```

## <a name="troubleshoot"></a>Dépanner

Si vous rencontrez des problèmes lors de la création de clusters HDInsight, reportez-vous aux [exigences de contrôle d’accès](./hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé un cluster HDInsight. Apprenez maintenant à l’utiliser.

### <a name="apache-spark-clusters"></a>Clusters Apache Spark

* [Personnaliser des clusters HDInsight Linux à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md)
* [Créer une application autonome avec Scala](spark/apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](spark/apache-spark-livy-rest-interface.md)
* [Apache Spark avec BI : effectuer une analyse interactive des données à l’aide de Spark sur HDInsight avec des outils décisionnels](spark/apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour prédire les résultats de l’inspection d’aliments](spark/apache-spark-machine-learning-mllib-ipython.md)

### <a name="apache-hadoop-clusters"></a>Clusters Apache Hadoop

* [Utilisation d’Apache Hive avec HDInsight](hadoop/hdinsight-use-hive.md)
* [Utilisation de MapReduce avec HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Clusters Apache HBase

* [Bien démarrer avec Apache HBase sur HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Développer des applications Java pour Apache HBase sur HDInsight](hbase/apache-hbase-build-java-maven-linux.md)
