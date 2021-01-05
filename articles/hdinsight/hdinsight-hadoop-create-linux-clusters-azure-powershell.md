---
title: Créer des clusters Apache Hadoop à l’aide de PowerShell - Azure HDInsight
description: Découvrez comment créer des clusters Apache Hadoop, Apache HBase, Apache Storm ou Apache Spark sur Linux pour HDInsight à l’aide d’Azure PowerShell.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/18/2019
ms.openlocfilehash: 23ce0f83e4233837d56ba9045a9a356ebd1cc33f
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92541809"
---
# <a name="create-linux-based-clusters-in-hdinsight-using-azure-powershell"></a>Créer des clusters basés sur Linux dans HDInsight à l’aide d’Azure PowerShell

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Azure PowerShell est un environnement puissant de création de scripts vous permettant de contrôler et d’automatiser le déploiement et la gestion de vos charges de travail dans Microsoft Azure. Ce document fournit des informations sur la création d’un cluster HDInsight basé sur Linux à l’aide d’Azure PowerShell. Il inclut également un exemple de script.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Module Az [Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="create-cluster"></a>Créer un cluster

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Les procédures suivantes sont nécessaires pour créer un cluster HDInsight en utilisant Azure PowerShell :

* Création d’un groupe de ressources Azure
* Création d'un compte Azure Storage
* Création d'un conteneur d'objets blob Azure
* Création d'un cluster HDInsight

> [!NOTE]
> L’utilisation de PowerShell pour créer un cluster HDInsight avec Azure Data Lake Storage Gen2 n’est pas prise en charge actuellement.

Le script suivant montre comment créer un nouveau cluster :

[!code-powershell[main](../../powershell_scripts/hdinsight/create-cluster/create-cluster.ps1?range=5-71)]

Les valeurs que vous spécifiez pour la connexion du cluster sont utilisées pour créer le compte d’utilisateur Hadoop pour le cluster. Utilisez ce compte pour la connexion aux services hébergés sur le cluster, tels que des interfaces utilisateur ou des API REST.

Les valeurs que vous spécifiez pour l’utilisateur SSH sont utilisées pour créer l’utilisateur SSH pour le cluster. Utilisez ce compte pour démarrer une session SSH à distance sur le cluster et pour exécuter des tâches. Pour plus d’informations, consultez le document [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

> [!IMPORTANT]  
> Si vous envisagez d’utiliser plus de 32 nœuds de travail lors de la création du cluster ou en faisant évoluer le cluster après sa création, vous devez également spécifier une taille de nœud principal avec au moins 8 cœurs et 14 Go de RAM.
>
> Pour plus d’informations sur les tailles de nœud et les coûts associés, consultez [Tarification HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

La création d’un cluster peut prendre jusqu’à 20 minutes.

## <a name="create-cluster-configuration-object"></a>Création de cluster : objet de configuration

Vous pouvez également créer un objet de configuration HDInsight à l’aide de l’applet de commande [`New-AzHDInsightClusterConfig`](/powershell/module/az.hdinsight/new-azhdinsightclusterconfig). Il est ensuite possible de modifier cet objet de configuration pour activer des options de configuration supplémentaires pour votre cluster. Enfin, utilisez le paramètre `-Config` de l’applet de commande [`New-AzHDInsightCluster`](/powershell/module/az.hdinsight/new-azhdinsightcluster) pour utiliser cette configuration.

Le script suivant crée un objet de configuration pour configurer un R Server sur le type de cluster HDInsight. La configuration active un nœud de périmètre, RStudio et un compte de stockage supplémentaire.

[!code-powershell[main](../../powershell_scripts/hdinsight/create-cluster/create-cluster-with-config.ps1?range=59-99)]

> [!WARNING]  
> L’utilisation d’un compte de stockage dans un autre emplacement que le cluster HDInsight n’est pas prise en charge. Pour cet exemple, créez le compte de stockage supplémentaire dans le même emplacement que le serveur.

## <a name="customize-clusters"></a>Personnaliser des clusters

* Consultez [Personnalisation de clusters HDInsight à l’aide de Bootstrap](hdinsight-hadoop-customize-cluster-bootstrap.md#use-azure-powershell).
* Consultez [Personnalisation de clusters HDInsight à l’aide d’une action de script](hdinsight-hadoop-customize-cluster-linux.md).

## <a name="delete-the-cluster"></a>Supprimer le cluster

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="troubleshoot"></a>Dépanner

Si vous rencontrez des problèmes lors de la création de clusters HDInsight, reportez-vous aux [exigences de contrôle d’accès](hdinsight-hadoop-create-linux-clusters-portal.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez réussi à créer un cluster HDInsight, utilisez les ressources ci-dessous pour apprendre à utiliser votre cluster.

### <a name="apache-hadoop-clusters"></a>Clusters Apache Hadoop

* [Utilisation d’Apache Hive avec HDInsight](hadoop/hdinsight-use-hive.md)
* [Utilisation de MapReduce avec HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Clusters Apache HBase

* [Bien démarrer avec Apache HBase sur HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Développer des applications Java pour Apache HBase sur HDInsight](hbase/apache-hbase-build-java-maven-linux.md)

### <a name="storm-clusters"></a>Clusters Storm

* [Développement de topologies Java pour Storm sur HDInsight](storm/apache-storm-develop-java-topology.md)
* [Utilisation de composants Python dans Storm sur HDInsight](storm/apache-storm-develop-python-topology.md)
* [Déploiement et analyse des topologies avec Storm sur HDInsight](storm/apache-storm-deploy-monitor-topology-linux.md)

### <a name="apache-spark-clusters"></a>Clusters Apache Spark

* [Créer une application autonome avec Scala](spark/apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](spark/apache-spark-livy-rest-interface.md)
* [Apache Spark avec BI : effectuer une analyse interactive des données à l’aide de Spark dans HDInsight avec des outils BI](spark/apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : Utiliser Spark dans HDInsight pour prédire les résultats d’une inspection alimentaire](spark/apache-spark-machine-learning-mllib-ipython.md)