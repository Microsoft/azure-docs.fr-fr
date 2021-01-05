---
title: 'Tutoriel : Diffusion en continu Apache Spark et Apache Kafka – Azure HDInsight'
description: Découvrez comment utiliser la diffusion en continu Apache Spark pour échanger des flux de données avec Apache Kafka. Dans ce didacticiel, vous diffusez des données avec un bloc-notes Jupyter à partir de Spark sur HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,seodec18,seoapr2020
ms.date: 04/22/2020
ms.openlocfilehash: 8aa7401a2ee7a0d87736e6b18fc814f983e2afa0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82184195"
---
# <a name="tutorial-use-apache-spark-structured-streaming-with-apache-kafka-on-hdinsight"></a>Tutoriel : Utiliser Apache Spark Structured Streaming avec Apache Kafka sur HDInsight

Ce tutoriel montre comment utiliser [Apache Spark Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide) pour lire et écrire des données avec [Apache Kafka](./kafka/apache-kafka-introduction.md) sur Azure HDInsight.

Spark Structured Streaming est un moteur de traitement de flux basé sur Spark SQL. Il vous permet d’exprimer des calculs de diffusion en continu de la même façon que pour les calculs de lot sur les données statiques.  

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Utilisez un modèle Azure Resource Manager pour créer des clusters.
> * Utiliser Spark Structured Streaming avec Kafka

Quand vous avez terminé les étapes décrites dans ce document, n’oubliez pas de supprimer les clusters pour éviter des frais supplémentaires.

## <a name="prerequisites"></a>Prérequis

* jq, processeur JSON en ligne de commande.  Voir [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

* Connaissances sur l’utilisation des blocs-notes [Jupyter Notebook](https://jupyter.org/) avec Spark sur HDInsight. Pour plus d’informations, consultez le document [Charger des données et exécuter des requêtes sur un cluster Apache Spark dans Azure HDInsight](spark/apache-spark-load-data-run-query.md).

* Connaissance du langage de programmation Scala. Le code utilisé dans ce didacticiel est écrit dans Scala.

* Connaissances sur la création des rubriques Kafka. Pour plus d’informations, consultez le document [Démarrage rapide : Créer un cluster Apache Kafka sur HDInsight](kafka/apache-kafka-get-started.md).

> [!IMPORTANT]  
> Les étapes décrites dans ce document nécessitent un groupe de ressources Azure contenant à la fois un Spark sur HDInsight et un Kafka sur un cluster HDInsight. Ces clusters sont tous deux situés dans un réseau virtuel Azure, ce qui permet au cluster Spark de communiquer directement avec le cluster Kafka.
>
> Pour des raisons pratiques, ce document renvoie à un modèle permettant de créer toutes les ressources Azure requises.
>
> Pour plus d’informations sur l’utilisation de HDInsight dans un réseau virtuel, consultez le document [Planifier un réseau virtuel pour HDInsight](hdinsight-plan-virtual-network-deployment.md).

## <a name="structured-streaming-with-apache-kafka"></a>Structured Streaming avec Apache Kafka

Spark Structured Streaming est un moteur de traitement de flux basé sur le moteur Spark SQL. Lorsque vous utilisez Structured Streaming, vous pouvez écrire les requêtes de streaming de la même façon que vous écrivez les requêtes par lots.

Les extraits de code suivants illustrent la lecture à partir de Kafka et le stockage dans le fichier. Le premier est une opération de traitement par lot, tandis que le second est une opération de diffusion en continu :

```scala
// Read a batch from Kafka
val kafkaDF = spark.read.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()

// Select data and write to file
kafkaDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .write
                .format("parquet")
                .option("path","/example/batchtripdata")
                .option("checkpointLocation", "/batchcheckpoint")
                .save()
```

```scala
// Stream from Kafka
val kafkaStreamDF = spark.readStream.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()

// Select data from the stream and write to file
kafkaStreamDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .writeStream
                .format("parquet")
                .option("path","/example/streamingtripdata")
                .option("checkpointLocation", "/streamcheckpoint")
                .start.awaitTermination(30000)
```

Dans les deux extraits de code, les données sont lues à partir de Kafka et écrites dans le fichier. Voici les différences qui séparent les exemples :

| Batch | Diffusion en continu |
| --- | --- |
| `read` | `readStream` |
| `write` | `writeStream` |
| `save` | `start` |

L’opération de streaming utilise également `awaitTermination(30000)`, ce qui bloque le flux après 30 000 ms.

Pour utiliser Structured Streaming avec Kafka, votre projet doit avoir une dépendance sur le package `org.apache.spark : spark-sql-kafka-0-10_2.11`. La version de ce package doit correspondre à la version de Spark sur HDInsight. Pour Spark 2.2.0 (disponible dans HDInsight 3.6), les informations de dépendance relatives à différents types de projet se trouvent sur [https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar](https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar).

Pour le notebook Jupyter qui est utilisé dans ce tutoriel, la cellule suivante charge cette dépendance de package :

```
%%configure -f
{
    "conf": {
        "spark.jars.packages": "org.apache.spark:spark-sql-kafka-0-10_2.11:2.2.0",
        "spark.jars.excludes": "org.scala-lang:scala-reflect,org.apache.spark:spark-tags_2.11"
    }
}
```

## <a name="create-the-clusters"></a>Création des clusters

Apache Kafka sur HDInsight ne donne pas accès aux répartiteurs Kafka sur l’Internet public. Tout ce qui utilise Kafka doit se trouver sur le même réseau virtuel Azure. Dans ce didacticiel, les clusters Kafka et Spark se trouvent dans le même réseau virtuel Azure.

Le diagramme suivant illustre les flux de communication entre Spark et Kafka :

![Diagramme des clusters Spark et Kafka dans un réseau virtuel Azure](./media/hdinsight-apache-kafka-spark-structured-streaming/apache-spark-kafka-vnet.png)

> [!NOTE]  
> Le service Kafka est limité à la communication au sein du réseau virtuel. L’accès aux autres services sur le cluster, tels que SSH et Ambari, se fait via Internet. Pour plus d’informations sur les ports publics disponibles avec HDInsight, consultez [Ports et URI utilisés par HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Pour créer un réseau virtuel Azure puis les clusters Kafka et Spark qu’il contient, procédez comme suit :

1. Utilisez le bouton suivant pour vous connecter à Azure et ouvrir le modèle dans le portail Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-spark-kafka-structured-streaming%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apache-kafka-spark-structured-streaming/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Le modèle Azure Resource Manager se trouve dans **https://raw.githubusercontent.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming/master/azuredeploy.json** .

    Ce modèle crée les ressources suivantes :

   * Un cluster Kafka sur HDInsight 3.6.
   * Un cluster Spark 2.2.0 sur HDInsight 3.6.
   * Un réseau virtuel Azure, qui contient les clusters HDInsight.

     > [!IMPORTANT]  
     > Le bloc-notes de diffusion en continu structurée utilisé dans ce didacticiel nécessite Spark 2.2.0 sur HDInsight 3.6. Si vous utilisez une version antérieure de Spark sur HDInsight, vous recevez des erreurs lors de l’utilisation du bloc-notes.

2. Utilisez les informations suivantes pour renseigner les entrées dans la section **Modèle personnalisé** :

    | Paramètre | Valeur |
    | --- | --- |
    | Abonnement | Votre abonnement Azure |
    | Resource group | Le groupe de ressources qui contient les ressources. |
    | Emplacement | La région Azure dans laquelle sont créées les ressources. |
    | Nom du cluster Spark | Le nom du cluster Spark. Les six premiers caractères doivent être différents du nom de cluster Kafka. |
    | Nom du cluster Kafka | Le nom du cluster Kafka. Les six premiers caractères doivent être différents du nom de cluster Spark. |
    | Nom d’utilisateur de connexion au cluster | Le nom d’utilisateur administrateur pour l’accès aux clusters. |
    | Mot de passe de connexion au cluster | Le mot de passe d’utilisateur administrateur pour l’accès aux clusters. |
    | Nom d’utilisateur SSH | L’utilisateur SSH à créer pour l’accès aux clusters. |
    | Mot de passe SSH | Le mot de passe de l’utilisateur SSH. |

    ![Capture d’écran du modèle personnalisé](./media/hdinsight-apache-kafka-spark-structured-streaming/spark-kafka-template.png)

3. Passez en revue les **termes et conditions**, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.

4. Sélectionnez **Achat**.

> [!NOTE]  
> La création des clusters peut prendre jusqu’à 20 minutes.

## <a name="use-spark-structured-streaming"></a>Utiliser Spark Structured Streaming

Cet exemple montre comment utiliser Spark Structured Streaming avec Kafka sur HDInsight Il utilise des données concernant des courses de taxi, qui sont fournies par la ville de New York.  Le jeu de données utilisé par ce notebook provient du site [2016 Green Taxi Trip Data](https://data.cityofnewyork.us/Transportation/2016-Green-Taxi-Trip-Data/hvrh-b6nb).

1. Collectez les informations concernant l’hôte. Utilisez les commandes curl et [jq](https://stedolan.github.io/jq/) ci-dessous pour obtenir les informations concernant les hôtes Kafka ZooKeeper et les hôtes de répartiteur. Ces commandes sont conçues pour être utilisées dans une invite de commandes Windows. Pour les autres environnements, de légères différences sont à prévoir. Remplacez `KafkaCluster` par le nom de votre cluster Kafka, et remplacez `KafkaPassword` par le mot de passe de connexion de votre cluster. Ensuite, remplacez `C:\HDI\jq-win64.exe` par le chemin de votre installation jq. Entrez les commandes dans une invite de commandes Windows, puis enregistrez la sortie en vue de l’utiliser ultérieurement.

    ```cmd
    REM Enter cluster name in lowercase

    set CLUSTERNAME=KafkaCluster
    set PASSWORD=KafkaPassword

    curl -u admin:%PASSWORD% -G "https://%CLUSTERNAME%.azurehdinsight.net/api/v1/clusters/%CLUSTERNAME%/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | C:\HDI\jq-win64.exe -r "["""\(.host_components[].HostRoles.host_name):2181"""] | join(""",""")"

    curl -u admin:%PASSWORD% -G "https://%CLUSTERNAME%.azurehdinsight.net/api/v1/clusters/%CLUSTERNAME%/services/KAFKA/components/KAFKA_BROKER" | C:\HDI\jq-win64.exe -r "["""\(.host_components[].HostRoles.host_name):9092"""] | join(""",""")"
    ```

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/jupyter`, où `CLUSTERNAME` est le nom de votre cluster. Lorsque vous y êtes invité, entrez l’identifiant de connexion (admin) et le mot de passe du cluster utilisés lors de la création du cluster.

1. Sélectionnez **Nouveau > Spark** pour créer un notebook.

1. Le streaming Spark utilise le microtraitement par lots, ce qui signifie que les données sont fournies sous forme de lots et que des exécuteurs s’exécutent sur les lots de données. Si le délai d’inactivité de l’exécuteur est inférieur au temps nécessaire au traitement du lot, l’exécuteur est constamment ajouté puis supprimé. Si le délai d’inactivité de l’exécuteur est supérieur à la durée du lot, l’exécuteur n’est jamais supprimé. Par conséquent, **nous vous recommandons de désactiver l’allocation dynamique en définissant spark.dynamicAllocation.enabled sur false lors de l’exécution d’applications de streaming**.

    Chargez les packages utilisés par le notebook en entrant les informations suivantes dans une cellule de notebook. Exécutez la commande à l’aide de **CTRL + ENTRÉE**.

    ```configuration
    %%configure -f
    {
        "conf": {
            "spark.jars.packages": "org.apache.spark:spark-sql-kafka-0-10_2.11:2.2.0",
            "spark.jars.excludes": "org.scala-lang:scala-reflect,org.apache.spark:spark-tags_2.11",
            "spark.dynamicAllocation.enabled": false
        }
    }
    ```

1. Créez la rubrique Kafka. Modifiez la commande ci-dessous en remplaçant `YOUR_ZOOKEEPER_HOSTS` par les informations sur l’hôte Zookeeper qui ont été extraites lors de la première étape. Entrez la commande modifiée dans votre notebook Jupyter pour créer la rubrique `tripdata`.

    ```scala
    %%bash
    export KafkaZookeepers="YOUR_ZOOKEEPER_HOSTS"

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic tripdata --zookeeper $KafkaZookeepers
    ```

1. Récupérez les données concernant les courses de taxi. Entrez la commande dans la cellule suivante pour charger les données sur les courses de taxi de New York. Les données sont chargées dans une trame de données qui est ensuite affichée comme sortie de la cellule.

    ```scala
    import spark.implicits._

    // Load the data from the New York City Taxi data REST API for 2016 Green Taxi Trip Data
    val url="https://data.cityofnewyork.us/resource/pqfs-mqru.json"
    val result = scala.io.Source.fromURL(url).mkString

    // Create a dataframe from the JSON data
    val taxiDF = spark.read.json(Seq(result).toDS)

    // Display the dataframe containing trip data
    taxiDF.show()
    ```

1. Définissez les informations concernant les hôtes de répartiteur Kafka. Remplacez `YOUR_KAFKA_BROKER_HOSTS` par les informations concernant les hôtes de répartiteur que vous avez extraites à l’étape 1.  Entrez la commande modifiée dans la cellule suivante du notebook Jupyter.

    ```scala
    // The Kafka broker hosts and topic used to write to Kafka
    val kafkaBrokers="YOUR_KAFKA_BROKER_HOSTS"
    val kafkaTopic="tripdata"

    println("Finished setting Kafka broker and topic configuration.")
    ```

1. Envoyez les données vers Kafka. Dans la commande suivante, le champ `vendorid`est utilisé comme la valeur de clé du message Kafka. La clé est utilisée par Kafka lors du partitionnement des données. Tous les champs sont stockés dans le message Kafka en tant que valeur de chaîne JSON. Entrez la commande suivante dans Jupyter pour enregistrer les données dans Kafka à l’aide d’une requête par lot.

    ```scala
    // Select the vendorid as the key and save the JSON string as the value.
    val query = taxiDF.selectExpr("CAST(vendorid AS STRING) as key", "to_JSON(struct(*)) AS value").write.format("kafka").option("kafka.bootstrap.servers", kafkaBrokers).option("topic", kafkaTopic).save()

    println("Data sent to Kafka")
    ```

1. Déclarez un schéma. La commande suivante montre comment utiliser un schéma lorsque vous lisez des données JSON dans Kafka. Entrez la commande dans la cellule Jupyter suivante.

    ```scala
    // Import bits useed for declaring schemas and working with JSON data
    import org.apache.spark.sql._
    import org.apache.spark.sql.types._
    import org.apache.spark.sql.functions._

    // Define a schema for the data
    val schema = (new StructType).add("dropoff_latitude", StringType).add("dropoff_longitude", StringType).add("extra", StringType).add("fare_amount", StringType).add("improvement_surcharge", StringType).add("lpep_dropoff_datetime", StringType).add("lpep_pickup_datetime", StringType).add("mta_tax", StringType).add("passenger_count", StringType).add("payment_type", StringType).add("pickup_latitude", StringType).add("pickup_longitude", StringType).add("ratecodeid", StringType).add("store_and_fwd_flag", StringType).add("tip_amount", StringType).add("tolls_amount", StringType).add("total_amount", StringType).add("trip_distance", StringType).add("trip_type", StringType).add("vendorid", StringType)
    // Reproduced here for readability
    //val schema = (new StructType)
    //   .add("dropoff_latitude", StringType)
    //   .add("dropoff_longitude", StringType)
    //   .add("extra", StringType)
    //   .add("fare_amount", StringType)
    //   .add("improvement_surcharge", StringType)
    //   .add("lpep_dropoff_datetime", StringType)
    //   .add("lpep_pickup_datetime", StringType)
    //   .add("mta_tax", StringType)
    //   .add("passenger_count", StringType)
    //   .add("payment_type", StringType)
    //   .add("pickup_latitude", StringType)
    //   .add("pickup_longitude", StringType)
    //   .add("ratecodeid", StringType)
    //   .add("store_and_fwd_flag", StringType)
    //   .add("tip_amount", StringType)
    //   .add("tolls_amount", StringType)
    //   .add("total_amount", StringType)
    //   .add("trip_distance", StringType)
    //   .add("trip_type", StringType)
    //   .add("vendorid", StringType)

    println("Schema declared")
    ```

1. Sélectionnez les données et démarrez le flux de données. La commande suivante montre comment récupérer des données à partir de Kafka à l’aide d’une requête par lots et écrire les résultats dans HDFS, sur le cluster Spark. Dans cet exemple, `select` récupère le message (champ de valeur) à partir de Kafka et lui applique le schéma. Les données sont ensuite écrites dans HDFS (WASB ou ADL) au format parquet. Entrez la commande dans la cellule Jupyter suivante.

    ```scala
    // Read a batch from Kafka
    val kafkaDF = spark.read.format("kafka").option("kafka.bootstrap.servers", kafkaBrokers).option("subscribe", kafkaTopic).option("startingOffsets", "earliest").load()

    // Select data and write to file
    val query = kafkaDF.select(from_json(col("value").cast("string"), schema) as "trip").write.format("parquet").option("path","/example/batchtripdata").option("checkpointLocation", "/batchcheckpoint").save()

    println("Wrote data to file")
    ```

1. Vous pouvez vérifier que les fichiers ont été créés en entrant la commande dans la cellule Jupyter suivante. Celle-ci affiche la liste des fichiers qui se trouvent dans le répertoire `/example/batchtripdata`.

    ```scala
    %%bash
    hdfs dfs -ls /example/batchtripdata
    ```

1. Dans l’exemple précédent, nous avons utilisé une requête par lot. La commande suivante montre comment faire la même chose à l’aide d’une requête de streaming. Entrez la commande dans la cellule Jupyter suivante.

    ```scala
    // Stream from Kafka
    val kafkaStreamDF = spark.readStream.format("kafka").option("kafka.bootstrap.servers", kafkaBrokers).option("subscribe", kafkaTopic).option("startingOffsets", "earliest").load()

    // Select data from the stream and write to file
    kafkaStreamDF.select(from_json(col("value").cast("string"), schema) as "trip").writeStream.format("parquet").option("path","/example/streamingtripdata").option("checkpointLocation", "/streamcheckpoint").start.awaitTermination(30000)
    println("Wrote data to file")
    ```

1. Exécutez la cellule suivante pour vérifier que les fichiers ont bien été écrits par la requête de streaming.

    ```scala
    %%bash
    hdfs dfs -ls /example/streamingtripdata
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce didacticiel, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le cluster HDInsight associé et toutes les autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources à l’aide du portail Azure :

1. Sur le [portail Azure](https://portal.azure.com/), développez le menu de gauche pour ouvrir le menu des services, puis sélectionnez __Groupes de ressources__ pour voir la liste de vos groupes de ressources.
2. Recherchez le groupe de ressources à supprimer, puis faites un clic droit sur le bouton __Plus__ (...) se trouvant à droite de la liste.
3. Sélectionnez __Supprimer le groupe de ressources__ et confirmez.

> [!WARNING]  
> La facturation du cluster HDInsight démarre à la création du cluster et s’arrête à sa suppression. La facturation est effectuée au prorata des minutes écoulées. Par conséquent, vous devez toujours supprimer votre cluster lorsqu’il n’est plus utilisé.
>
> La suppression d’un cluster Kafka sur HDInsight supprime toutes les données stockées dans Kafka.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser Apache Spark Structured Streaming pour écrire et lire des données à partir d’Apache Kafka sur HDInsight. Utilisez le lien suivant pour découvrir comment utiliser Apache Storm avec Kafka.

> [!div class="nextstepaction"]
> [Utiliser Apache Storm avec Apache Kafka](hdinsight-apache-storm-with-kafka.md)
