---
title: Qu'est-ce qu'Interactive Query dans Azure HDInsight ?
description: Introduction à Interactive Query, également appelé Apache Hive LLAP, dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive
ms.date: 03/03/2020
ms.openlocfilehash: e133e08e333cb478269a93cce963566e195d6949
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "78271952"
---
# <a name="what-is-interactive-query-in-azure-hdinsight"></a>Qu'est-ce qu'Interactive Query dans Azure HDInsight

Le cluster Interactive Query (également appelé Apache Hive LLAP ou [Low Latency Analytical Processing](https://cwiki.apache.org/confluence/display/Hive/LLAP)) est un [type de cluster](../hdinsight-hadoop-provision-linux-clusters.md#cluster-type) Azure HDInsight. Interactive Query prend en charge la mise en mémoire cache, ce qui accélère les requêtes Apache Hive et les rend beaucoup plus interactives. Les clients utilisent Interactive Query pour interroger des données stockées dans le stockage Azure et Azure Data Lake Storage de manière extrêmement rapide. Une requête interactive permet facilement aux développeurs et scientifiques des données de travailler avec des données volumineuses (big data) à l’aide des outils décisionnels qu’ils préfèrent. HDInsight Interactive Query prend en charge plusieurs outils pour accéder aux données volumineuses de façon facile.

[!INCLUDE [hdinsight-price-change](../../../includes/hdinsight-enhancements.md)]

Les clusters Interactive Query sont différents des clusters Apache Hadoop. Ils contiennent uniquement le service Hive.

Vous pouvez accéder au service Hive dans le cluster Interactive Query uniquement par le biais de la vue Apache Ambari Hive, de Beeline et du pilote ODBC Microsoft Hive. Vous ne pouvez pas y accéder via la console Hive, Templeton, Azure Classic CLI ou Azure PowerShell.

## <a name="create-an-interactive-query-cluster"></a>Créer un cluster Interactive Query

Pour obtenir des informations sur la création d’un cluster HDInsight, consultez [Créer des clusters Apache Hadoop dans HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Choisissez le type de cluster Interactive Query.

> [!IMPORTANT]
> La taille minimale du nœud principal pour les clusters Interactive Query est Standard_D13_v2. Pour plus d’informations, consultez le [Tableau de dimensionnement des machines virtuelles Azure](../../cloud-services/cloud-services-sizes-specs.md#dv2-series).

## <a name="execute-apache-hive-queries-from-interactive-query"></a>Exécuter des requêtes Apache Hive à partir du cluster Interactive Query

Pour exécuter des requêtes Hive, vous disposez des options suivantes :

|Méthode |Description |
|---|---|
|Microsoft Power BI|Consultez [Visualiser des données Interactive Query Apache Hive à l’aide de Power BI dans Azure HDInsight](./apache-hadoop-connect-hive-power-bi-directquery.md) et [Visualiser des données volumineuses (« Big Data ») avec Power BI dans Azure HDInsight](../hadoop/apache-hadoop-connect-hive-power-bi.md).|
|Visual Studio|Consultez [Se connecter à Azure HDInsight et exécuter des requêtes Apache Hive avec Data Lake Tools pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#run-interactive-apache-hive-queries).|
|Visual Studio Code|Consultez [Utiliser Visual Studio Code pour Apache Hive, LLAP ou pySpark](../hdinsight-for-vscode.md).|
|Vue Apache Ambari Hive|Consultez [Utiliser la vue Apache Hive avec Apache Hadoop dans Azure HDInsight](../hadoop/apache-hadoop-use-hive-ambari-view.md). La vue Hive n’est pas disponible pour HDInsight 4.0.|
|Apache Beeline|Consultez [Utiliser Apache Hive avec Apache Hadoop dans HDInsight via Beeline](../hadoop/apache-hadoop-use-hive-beeline.md). Vous pouvez utiliser Beeline à partir du nœud principal ou d’un nœud de périphérie vide. L’utilisation de Beeline à partir d’un nœud de périphérie vide est recommandée. Pour plus d’informations sur la création d’un cluster HDInsight avec un nœud de périphérie vide, consultez [Utiliser des nœuds de périphérie vides dans HDInsight](../hdinsight-apps-use-edge-node.md).|
|Hive ODBC|Consultez [Connexion d’Excel à Apache Hadoop à l’aide du pilote ODBC Microsoft Hive](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).|

Pour rechercher la chaîne de connexion Java Database Connectivity (JDBC) :

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, où `CLUSTERNAME` est le nom de votre cluster.
1. Pour copier l’URL, sélectionnez l’icône du Presse-papiers :

   ![HDInsight Hadoop Interactive Query LLAP JDBC](./media/apache-interactive-query-get-started/hdinsight-hadoop-use-interactive-hive-jdbc.png)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [créer des clusters Interactive Query dans HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).
* Découvrez comment [visualiser des Big Data à l’aide de Power BI dans Azure HDInsight](../hadoop/apache-hadoop-connect-hive-power-bi.md).
* Découvrez comment [utiliser Apache Zeppelin pour exécuter des requêtes Apache Hive dans Azure HDInsight](../interactive-query/hdinsight-connect-hive-zeppelin.md).
