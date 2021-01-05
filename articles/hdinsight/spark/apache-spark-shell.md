---
title: Utiliser un interpréteur de commandes Spark interactif dans Azure HDInsight
description: Un interpréteur de commandes Spark interactif fournit un processus de lecture-exécution-impression pour l’exécution des commandes Spark les unes après les autres et l’affichage des résultats.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 02/10/2020
ms.openlocfilehash: 84298c9073f00f0388a9bcb7405369d7c60bcce1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86081177"
---
# <a name="run-apache-spark-from-the-spark-shell"></a>Exécuter Apache Spark depuis l’interpréteur de commandes Spark

Un interpréteur de commandes [Apache Spark](https://spark.apache.org/) interactif fournit un environnement REPL (boucle de lecture-exécution-impression) pour l’exécution des commandes Spark les unes après les autres et l’affichage des résultats. Ce processus est utile pour le développement et le débogage. Spark fournit un interpréteur de commandes pour chacune des langues qu’il prend en charge : Scala, Python et R.

## <a name="run-an-apache-spark-shell"></a>Exécuter un interpréteur de commandes Apache Spark

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Spark fournit des interpréteurs de commandes pour Scala (spark-shell) et Python (pyspark). Dans votre session SSH, entrez *une* des commandes suivantes :

    ```bash
    spark-shell

    # Optional configurations
    # spark-shell --num-executors 4 --executor-memory 4g --executor-cores 2 --driver-memory 8g --driver-cores 4
    ```

    ```bash
    pyspark

    # Optional configurations
    # pyspark --num-executors 4 --executor-memory 4g --executor-cores 2 --driver-memory 8g --driver-cores 4
    ```

    Si vous envisagez d’utiliser une configuration facultative, commencez par consulter [Exception OutOfMemoryError pour Apache Spark](./apache-spark-troubleshoot-outofmemory.md).

1. Voici quelques exemples de commandes de base. Choisissez le langage approprié :

    ```spark-shell
    val textFile = spark.read.textFile("/example/data/fruits.txt")
    textFile.first()
    textFile.filter(line => line.contains("apple")).show()
    ```

    ```pyspark
    textFile = spark.read.text("/example/data/fruits.txt")
    textFile.first()
    textFile.filter(textFile.value.contains("apple")).show()
    ```

1. Interrogez un fichier CSV. Notez que le langage ci-dessous fonctionne pour `spark-shell` et `pyspark`.

    ```scala
    spark.read.csv("/HdiSamples/HdiSamples/SensorSampleData/building/building.csv").show()
    ```

1. Interrogez un fichier CSV et stockez les résultats dans une variable :

    ```spark-shell
    var data = spark.read.format("csv").option("header", "true").option("inferSchema", "true").load("/HdiSamples/HdiSamples/SensorSampleData/building/building.csv")
    ```

    ```pyspark
    data = spark.read.format("csv").option("header", "true").option("inferSchema", "true").load("/HdiSamples/HdiSamples/SensorSampleData/building/building.csv")
    ```

1. Affichez les résultats :

    ```spark-shell
    data.show()
    data.select($"BuildingID", $"Country").show(10)
    ```

    ```pyspark
    data.show()
    data.select("BuildingID", "Country").show(10)
    ```

1. Quitter

    ```spark-shell
    :q
    ```

    ```pyspark
    exit()
    ```

## <a name="sparksession-and-sparkcontext-instances"></a>Instances de SparkSession et SparkContext

Par défaut, lorsque vous exécutez l’interpréteur de commandes, des instances de SparkSession et SparkContext sont automatiquement instanciées pour vous.

Pour accéder à l’instance de SparkSession, entrez `spark`. Pour accéder à l’instance de SparkContext, entrez `sc`.

## <a name="important-shell-parameters"></a>Paramètres importants d’interpréteur de commandes

La commande d’interpréteur de commandes Spark (`spark-shell` ou `pyspark`) prend en charge de nombreux paramètres de ligne de commande. Pour afficher une liste complète des paramètres, démarrez l’interpréteur de commandes Spark avec le commutateur `--help`. Certains de ces paramètres peuvent s’appliquer uniquement à `spark-submit`, que l’interpréteur de commandes Spark encapsule.

| commutateur | description | exemple |
| --- | --- | --- |
| --master MASTER_URL | Spécifie l’URL principale. Dans HDInsight, cette valeur est toujours `yarn`. | `--master yarn`|
| --jars JAR_LIST | Liste séparée par des virgules des fichiers JAR locaux à inclure dans les classpaths du pilote et de l’exécuteur. Dans HDInsight, cette liste est composée de chemins au système de fichiers par défaut dans Data Lake Storage ou Stockage Azure. | `--jars /path/to/examples.jar` |
| --packages MAVEN_COORDS | Liste séparée par des virgules des coordonnées Maven des fichiers JAR à inclure dans les classpaths du pilote et de l’exécuteur. Recherche dans le référentiel Maven local, puis dans le référentiel Maven central, et enfin dans tout autre référentiel distant spécifié avec `--repositories`. Le format des coordonnées est *groupId*:*artifactId*:*version*. | `--packages "com.microsoft.azure:azure-eventhubs:0.14.0"`|
| --py-files LIST | Pour Python uniquement, une liste séparée par des virgules des fichiers .zip, .egg ou .py à placer dans le PYTHONPATH. | `--pyfiles "samples.py"` |

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Présentation d’Apache Spark sur Azure HDInsight](apache-spark-overview.md) pour obtenir une vue d’ensemble.
- Consultez [Créer un cluster Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md) pour utiliser des clusters Spark et SparkSQL.
- Consultez [Vue d’ensemble d’Apache Spark Structured Streaming](apache-spark-streaming-overview.md) pour écrire des applications traitant des données de diffusion en continu avec Spark.
