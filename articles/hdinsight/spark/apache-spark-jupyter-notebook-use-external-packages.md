---
title: Utiliser des packages Maven personnalisés avec Jupyter dans Spark - Azure HDInsight
description: Cette section comporte des instructions détaillées sur la façon de configurer des blocs-notes Jupyter disponibles avec des clusters Spark HDInsight pour utiliser des packages Maven personnalisés.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/22/2019
ms.openlocfilehash: 772b136c00dc9c20f8bc35d7ebb324175a56e885
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90061714"
---
# <a name="use-external-packages-with-jupyter-notebooks-in-apache-spark-clusters-on-hdinsight"></a>Utilisation de packages externes avec les blocs-notes Jupyter dans des clusters Apache Spark sur HDInsight

Découvrez comment configurer un bloc-notes [Jupyter Notebook](https://jupyter.org/) dans un cluster Apache Spark sur HDInsight pour utiliser des packages Apache **Maven** externes bénéficiant de la contribution de la communauté, qui ne sont pas inclus dans le cluster.

Vous pouvez rechercher le [référentiel Maven](https://search.maven.org/) pour obtenir la liste complète des packages disponibles. Vous pouvez également obtenir une liste des packages disponibles à partir d’autres sources. Par exemple, une liste complète des packages bénéficiant de la contribution de la communauté est disponible sur le site [Spark Packages](https://spark-packages.org/)(Packages Spark).

Dans cet article, vous allez apprendre à utiliser le package [spark-csv](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar) avec le bloc-notes Jupyter.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Spark sur HDInsight. Pour obtenir des instructions, consultez [Création de clusters Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).

* Connaissances sur l’utilisation des blocs-notes Jupyter Notebook avec Spark sur HDInsight. Pour plus d’informations, consultez [Charger des données et exécuter des requêtes sur un cluster Apache Spark dans Azure HDInsight](./apache-spark-load-data-run-query.md).

* Le [schéma d’URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme) de votre principal espace de stockage de clusters. Il s’agirait de `wasb://` pour Stockage Azure, de `abfs://` pour Azure Data Lake Storage Gen2 ou de `adl://` pour Azure Data Lake Storage Gen1. Si l’option de transfert sécurisé est activée pour Stockage Azure ou Data Lake Storage Gen2, l’URI serait `wasbs://` ou `abfss://`, respectivement. Consultez également l’article dédié au [transfert sécurisé](../../storage/common/storage-require-secure-transfer.md).

## <a name="use-external-packages-with-jupyter-notebooks"></a>Utiliser des packages externes avec les blocs-notes Jupyter

1. Accédez à `https://CLUSTERNAME.azurehdinsight.net/jupyter`, où `CLUSTERNAME` est le nom de votre cluster Spark.

1. Créer un nouveau bloc-notes. Sélectionnez **Nouveau**, puis sélectionnez **Spark**.

    ![Créer un notebook Jupyter Spark](./media/apache-spark-jupyter-notebook-use-external-packages/hdinsight-spark-create-notebook.png "Créer un bloc-notes Jupyter")

1. Un nouveau bloc-notes est créé et ouvert sous le nom Untitled.pynb. Sélectionnez le nom du bloc-notes en haut, puis entrez un nom convivial.

    ![Fournir un nom au notebook](./media/apache-spark-jupyter-notebook-use-external-packages/hdinsight-spark-name-notebook.png "Fournir un nom pour le bloc-notes")

1. Vous allez utiliser la commande magique `%%configure` pour configurer le bloc-notes afin d’utiliser un package externe. Dans les blocs-notes utilisant des packages externes, veillez à appeler la commande magique `%%configure` dans la première cellule de code. Cela garantit que le noyau est configuré pour utiliser le package avant le démarrage de la session.

    >[!IMPORTANT]  
    >Si vous oubliez de configurer le noyau dans la première cellule, vous pouvez utiliser `%%configure` avec le paramètre `-f`. Toutefois, cette opération redémarrera la session et entraînera la perte de toute la progression.

    | Version de HDInsight | Commande |
    |-------------------|---------|
    | Pour HDInsight 3.5 et HDInsight 3.6 | `%%configure`<br>`{ "conf": {"spark.jars.packages": "com.databricks:spark-csv_2.11:1.5.0" }}`|
    |Pour HDInsight 3.3 et HDInsight 3.4 | `%%configure` <br>`{ "packages":["com.databricks:spark-csv_2.10:1.4.0"] }`|

1. L’extrait de code ci-dessus attend une liste de coordonnées maven pour le package externe du référentiel central Maven. Dans cet extrait de code, `com.databricks:spark-csv_2.11:1.5.0` est la coordonnée maven pour le package **spark-csv** . Voici comment vous construire les coordonnées d’un package.

    a. Recherchez le package dans le référentiel Maven. Dans cet article, nous utilisons [spark-csv](https://mvnrepository.com/artifact/com.databricks/spark-csv).

    b. À partir du référentiel, rassemblez les valeurs pour **GroupId**, **ArtifactId** et **Version**. Vérifiez que les valeurs que vous collectez correspondent à votre cluster. Dans ce cas, nous utilisons un package Scala 2.11 et Spark 1.5.0, mais il peut être nécessaire de sélectionner des versions différentes pour la version appropriée de Scala ou de Spark dans votre cluster. Vous pouvez trouver la version de Scala sur votre cluster en exécutant `scala.util.Properties.versionString` sur le noyau Spark Jupyter ou sur spark-submit. Vous pouvez trouver la version de Spark sur votre cluster en exécutant `sc.version` sur les notebooks Jupyter.

    ![Utiliser des packages externes avec les notebooks Jupyter](./media/apache-spark-jupyter-notebook-use-external-packages/use-external-packages-with-jupyter.png "Utiliser des packages externes avec les blocs-notes Jupyter")

    c. Concaténez les trois valeurs séparées par deux-points ( **:** ).

    ```scala
    com.databricks:spark-csv_2.11:1.5.0
    ```

1. Exécutez la cellule de code avec la commande magique `%%configure` . Cela configurera la session Livy sous-jacente pour utiliser le package fourni. Dans les cellules suivantes du bloc-notes, vous pouvez maintenant utiliser le package, comme indiqué ci-dessous.

    ```scala
    val df = spark.read.format("com.databricks.spark.csv").
    option("header", "true").
    option("inferSchema", "true").
    load("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    ```

    Pour HDInsight 3.4 et versions antérieures, utilisez l’extrait de code suivant.

    ```scala
    val df = sqlContext.read.format("com.databricks.spark.csv").
    option("header", "true").
    option("inferSchema", "true").
    load("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    ```

1. Vous pouvez ensuite exécuter les extraits de code, comme illustré ci-dessous, pour afficher les données issues du tableau de données créé lors de l’étape précédente.

    ```scala
    df.show()
   
    df.select("Time").count()
    ```

## <a name="see-also"></a><a name="seealso"></a>Voir aussi

* [Vue d’ensemble : Apache Spark sur Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Scénarios

* [Apache Spark avec BI : effectuer une analyse interactive des données à l’aide de Spark sur HDInsight avec des outils décisionnels](apache-spark-use-bi-tools.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour l’analyse de la température de bâtiments à l’aide des données des systèmes HVAC](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour prédire les résultats de l’inspection d’aliments](apache-spark-machine-learning-mllib-ipython.md)
* [Analyse des journaux de site web à l’aide d’Apache Spark dans HDInsight](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>Création et exécution d’applications

* [Créer une application autonome avec Scala](apache-spark-create-standalone-application.md)
* [Exécuter des tâches à distance avec Apache Livy sur un cluster Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Outils et extensions

* [Utilisation de packages externes Python avec les blocs-notes Jupyter dans des clusters Apache Spark sur HDInsight Linux](apache-spark-python-package-installation.md)
* [Utilisation du plugin d’outils HDInsight pour IntelliJ IDEA pour créer et soumettre des applications Spark Scala](apache-spark-intellij-tool-plugin.md)
* [Utiliser le plug-in Azure HDInsight Tools pour IntelliJ IDEA afin de déboguer des applications Apache Spark à distance](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Utiliser des blocs-notes Apache Zeppelin avec un cluster Apache Spark sur HDInsight](apache-spark-zeppelin-notebook.md)
* [Noyaux accessibles à Jupyter Notebook dans le cluster Apache Spark pour HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster (Installer Jupyter sur un ordinateur et se connecter au cluster Spark sur HDInsight)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Gestion des ressources

* [Gérer les ressources du cluster Apache Spark dans Azure HDInsight](apache-spark-resource-manager.md)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight (Suivi et débogage des tâches en cours d’exécution sur un cluster Apache Spark dans HDInsight)](apache-spark-job-debugging.md)
