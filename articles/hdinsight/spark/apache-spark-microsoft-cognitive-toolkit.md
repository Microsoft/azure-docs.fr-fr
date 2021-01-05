---
title: Microsoft Cognitive Toolkit avec Apache Spark – Azure HDInsight
description: Découvrez comment un modèle entraîné de formation approfondie Microsoft Cognitive Toolkit peut être appliqué à un jeu de données avec l’API Python Spark dans un cluster Azure HDInsight Spark.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/14/2020
ms.openlocfilehash: 7fa3355ff7e61d624d57da68d0f835faa7ca3736
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505424"
---
# <a name="use-microsoft-cognitive-toolkit-deep-learning-model-with-azure-hdinsight-spark-cluster"></a>Utiliser le modèle de formation approfondie Microsoft Cognitive Toolkit avec un cluster Azure HDInsight Spark

Dans cet article, vous suivez les étapes ci-dessous.

1. Exécuter un script personnalisé pour installer [Microsoft Cognitive Toolkit](/cognitive-toolkit/) sur un cluster Azure HDInsight Spark.

2. Charger un bloc-notes [Jupyter Notebook](https://jupyter.org/) sur le cluster [Apache Spark](https://spark.apache.org/) pour voir comment appliquer un modèle entraîné d’apprentissage profond Microsoft Cognitive Toolkit aux fichiers d’un compte de Stockage Blob Azure avec l’[API Spark Python (PySpark)](https://spark.apache.org/docs/latest/api/python/index.html).

## <a name="prerequisites"></a>Conditions préalables requises

* Un cluster Apache Spark sur HDInsight. Consultez [Créer un cluster Apache Spark](./apache-spark-jupyter-spark-sql-use-portal.md).

* Connaissances sur l’utilisation des blocs-notes Jupyter Notebook avec Spark sur HDInsight. Pour plus d’informations, consultez [Charger des données et exécuter des requêtes sur un cluster Apache Spark dans Azure HDInsight](./apache-spark-load-data-run-query.md).

## <a name="how-does-this-solution-flow"></a>Comment fonctionne cette solution ?

Cette solution est divisée entre cet article et un bloc-notes Jupyter que vous chargez dans le cadre de cet article. Dans cet article, vous suivrez les étapes ci-dessous :

* Exécuter une action de script sur un cluster HDInsight Spark pour installer Microsoft Cognitive Toolkit et les packages Python.
* Chargez le bloc-notes Jupyter qui exécute la solution sur le cluster HDInsight Spark.

Les étapes restantes suivantes sont traitées dans le bloc-notes Jupyter.

* Charger des exemples d’images dans un RDD (Spark Resilient Distributed Dataset).
  * Charger des modules et prédéfinir des paramètres
  * Télécharger le jeu de données localement sur le cluster Spark
  * Convertir le jeu de données en RDD
* Attribuer un score aux images à l’aide d’un modèle Cognitive Toolkit entraîné
  * Télécharger le modèle Cognitive Toolkit entraîné vers le cluster Spark
  * Définir les fonctions utilisées par les nœuds de travail
  * Attribuer un score aux images sur les nœuds de travail
  * Évaluer la précision du modèle

## <a name="install-microsoft-cognitive-toolkit"></a>Installer Microsoft Cognitive Toolkit

Vous pouvez installer Microsoft Cognitive Toolkit sur un cluster Spark avec une action de script. L’action de script utilise des scripts personnalisés pour installer sur le cluster des composants qui ne sont pas disponibles par défaut. Vous pouvez utiliser le script personnalisé sur le portail Azure, avec le Kit de développement logiciel (SDK) .NET HDInsight ou Azure PowerShell. Vous pouvez également utiliser le script pour installer la boîte à outils lors de la création du cluster ou lorsque le cluster est prêt à fonctionner.

Dans cet article, nous utilisons le portail pour installer la boîte à outils une fois le cluster créé. Pour connaître d’autres façons d’exécuter le script personnalisé, consultez la page [Personnaliser des clusters HDInsight avec une action de script](../hdinsight-hadoop-customize-cluster-linux.md).

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Pour connaître les instructions liées à l’utilisation du portail Azure afin d'exécuter une action de script, consultez la page [Personnaliser des clusters HDInsight avec une action de script](../hdinsight-hadoop-customize-cluster-linux.md#script-action-during-cluster-creation). Veillez à fournir les entrées suivantes pour installer Microsoft Cognitive Toolkit. Utilisez les valeurs suivantes pour votre action de script :

|Propriété |Valeur |
|---|---|
|Type de script|- Personnalisé|
|Name| Installer MCT|
|URI de script bash|`https://raw.githubusercontent.com/Azure-Samples/hdinsight-pyspark-cntk-integration/master/cntk-install.sh`|
|Types de nœud :|Head, Worker|
|Paramètres|None|

## <a name="upload-the-jupyter-notebook-to-azure-hdinsight-spark-cluster"></a>Charger le bloc-notes Jupyter sur le cluster Azure HDInsight Spark

Pour utiliser le Microsoft Cognitive Toolkit avec le cluster Azure HDInsight Spark, vous devez charger le bloc-notes Jupyter **CNTK_model_scoring_on_Spark_walkthrough.ipynb** sur le cluster Azure HDInsight Spark. Ce bloc-notes est disponible sur GitHub à l’adresse [https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration](https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration).

1. Téléchargez et décompressez [https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration](https://github.com/Azure-Samples/hdinsight-pyspark-cntk-integration).

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/jupyter`, où `CLUSTERNAME` est le nom de votre cluster.

1. Dans le bloc-notes Jupyter, sélectionnez **Charger** dans l’angle supérieur droit, puis accédez au téléchargement et sélectionnez le fichier `CNTK_model_scoring_on_Spark_walkthrough.ipynb`.

    ![Charger un bloc-notes Jupyter sur un cluster Azure HDInsight Spark](./media/apache-spark-microsoft-cognitive-toolkit/hdinsight-microsoft-cognitive-toolkit-load-jupyter-notebook.png "Charger un bloc-notes Jupyter sur un cluster Azure HDInsight Spark")

1. Sélectionnez **Charger** à nouveau.

1. Une fois le bloc-notes chargé, cliquez sur son nom, puis suivez les instructions qui se trouvent dans le bloc-notes même pour charger le jeu de données et suivre les instructions de l’article.

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble : Apache Spark sur Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Scénarios

* [Apache Spark avec BI : effectuer une analyse interactive des données à l’aide de Spark dans HDInsight avec des outils BI](apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : Utiliser Spark dans HDInsight pour analyser la température d’un bâtiment à l’aide de données issues des systèmes de chauffage, de ventilation et de climatisation](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark avec Machine Learning : Utiliser Spark dans HDInsight pour prédire les résultats d’une inspection alimentaire](apache-spark-machine-learning-mllib-ipython.md)
* [Analyse des journaux de site web à l’aide d’Apache Spark dans HDInsight](apache-spark-custom-library-website-log-analysis.md)
* [Analyse de données de télémétrie Application Insight avec Spark dans HDInsight](apache-spark-analyze-application-insight-logs.md)

### <a name="create-and-run-applications"></a>Création et exécution d’applications

* [Créer une application autonome avec Scala](apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Outils et extensions

* [Utilisation du plugin d’outils HDInsight pour IntelliJ IDEA pour créer et soumettre des applications Spark Scala](apache-spark-intellij-tool-plugin.md)
* [Utiliser le plug-in Azure HDInsight Tools pour IntelliJ IDEA afin de déboguer des applications Apache Spark à distance](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Utiliser des blocs-notes Apache Zeppelin avec un cluster Apache Spark sur HDInsight](apache-spark-zeppelin-notebook.md)
* [Noyaux disponibles pour bloc-notes Jupyter dans un cluster Apache Spark pour HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Utiliser des packages externes avec des blocs-notes Jupyter](apache-spark-jupyter-notebook-use-external-packages.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Gestion des ressources

* [Gérer les ressources du cluster Apache Spark dans Azure HDInsight](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight (Suivi et débogage des tâches en cours d’exécution sur un cluster Apache Spark dans HDInsight)](apache-spark-job-debugging.md)
