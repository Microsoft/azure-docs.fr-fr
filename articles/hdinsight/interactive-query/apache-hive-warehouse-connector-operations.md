---
title: Opérations d’Apache Spark prises en charge par Hive Warehouse Connector dans Azure HDInsight
description: Découvrez les différentes fonctionnalités de Hive Warehouse Connector sur Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/22/2020
ms.openlocfilehash: ed3dbe4cb5b9d8b50c028a68feeded170130bfb8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86085835"
---
# <a name="apache-spark-operations-supported-by-hive-warehouse-connector-in-azure-hdinsight"></a>Opérations d’Apache Spark prises en charge par Hive Warehouse Connector dans Azure HDInsight

Cet article présente les opérations basées sur Spark prises en charge par Hive Warehouse Connector (HWC). Tous les exemples ci-dessous seront exécutés via l’interpréteur de commandes Apache Spark.

## <a name="prerequisite"></a>Configuration requise

Suivez les étapes de [Configuration de Hive Warehouse Connector](./apache-hive-warehouse-connector.md#hive-warehouse-connector-setup).

## <a name="getting-started"></a>Prise en main

Pour démarrer une session spark-shell, procédez comme suit :

1. Utilisez une [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster Apache Spark. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. À partir de votre session SSH, exécutez la commande suivante afin de noter la version `hive-warehouse-connector-assembly` :

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

1. Modifiez le code ci-dessous avec la version `hive-warehouse-connector-assembly` identifiée ci-dessus. Exécutez ensuite la commande pour démarrer l’interpréteur de commandes Spark :

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<STACK_VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    ```

1. Après avoir démarré l’interpréteur de commandes de Spark, une instance du connecteur d'entrepôt Hive peut être lancée en utilisant les commandes suivantes :

    ```scala
    import com.hortonworks.hwc.HiveWarehouseSession
    val hive = HiveWarehouseSession.session(spark).build()
    ```

## <a name="creating-spark-dataframes-using-hive-queries"></a>Création de DataFrames Spark à partir de requêtes Hive

Les résultats de toutes les requêtes utilisant la bibliothèque HWC sont renvoyés sous forme de DataFrame. Les exemples suivants montrent comment créer une requête Hive de base.

```scala
hive.setDatabase("default")
val df = hive.executeQuery("select * from hivesampletable")
df.filter("state = 'Colorado'").show()
```

Les résultats de la requête sont des DataFrames Spark, qui peuvent être utilisés avec les bibliothèques Spark comme MLIB et SparkSQL.

## <a name="writing-out-spark-dataframes-to-hive-tables"></a>Écriture de DataFrames Spark dans des tables Hive

Spark ne prend pas en charge en mode natif l’écriture dans des tables ACID managées de Hive. HWC vous permet cependant d’écrire n’importe quel DataFrame dans une table Hive. Vous pouvez voir cette fonctionnalité en action dans l'exemple suivant :

1. Créez une table appelée `sampletable_colorado` et spécifiez ses colonnes en utilisant la commande suivante :

    ```scala
    hive.createTable("sampletable_colorado").column("clientid","string").column("querytime","string").column("market","string").column("deviceplatform","string").column("devicemake","string").column("devicemodel","string").column("state","string").column("country","string").column("querydwelltime","double").column("sessionid","bigint").column("sessionpagevieworder","bigint").create()
    ```

1. Filtrez la table `hivesampletable`, où la colonne `state` est égale à `Colorado`. Cette requête Hive retourne un DataFrame Spark et est enregistrée dans la table Hive `sampletable_colorado` à l’aide de la fonction `write`.

    ```scala
    hive.table("hivesampletable").filter("state = 'Colorado'").write.format("com.hortonworks.spark.sql.hive.llap.HiveWarehouseConnector").mode("append").option("table","sampletable_colorado").save()
    ```

1. Visualisez les résultats avec la commande suivante :

    ```scala
    hive.table("sampletable_colorado").show()
    ```
    
    ![Connecteur d’entrepôt Hive - Affichage de la table Hive](./media/apache-hive-warehouse-connector/hive-warehouse-connector-show-hive-table.png)


## <a name="structured-streaming-writes"></a>Écritures à l’aide du streaming structuré

Le connecteur d’entrepôt Hive vous permet d’utiliser le streaming Spark pour écrire des données dans des tables Hive.

> [!IMPORTANT]
> Les écritures de streaming structuré ne sont pas prises en charge dans les clusters Spark 4.0 à extension ESP.

Suivez les étapes ci-dessous pour ingérer les données d’un flux Spark sur le port localhost 9999 dans une table Hive. Hive Warehouse Connector.

1. À partir de votre interpréteur de commandes Spark ouvert, démarrez un flux Spark à l’aide de la commande suivante :

    ```scala
    val lines = spark.readStream.format("socket").option("host", "localhost").option("port",9999).load()
    ```

1. Procédez comme suit pour générer les données du flux Spark que vous avez créé :
    1. Ouvrez une deuxième session SSH sur le même cluster Spark.
    1. À l’invite de commandes, tapez `nc -lk 9999`. Cette commande utilise l'utilitaire netcat pour envoyer des données de la ligne de commande au port spécifié.

1. Revenez à la première session SSH et créez une nouvelle table Hive pour contenir les données de streaming. Dans l’interpréteur de commandes de Spark, entrez la commande suivante :

    ```scala
    hive.createTable("stream_table").column("value","string").create()
    ```

1. Puis, écrivez les données de streaming dans la table nouvellement créée à l’aide de la commande suivante :

    ```scala
    lines.filter("value = 'HiveSpark'").writeStream.format("com.hortonworks.spark.sql.hive.llap.streaming.HiveStreamingDataSource").option("database", "default").option("table","stream_table").option("metastoreUri",spark.conf.get("spark.datasource.hive.warehouse.metastoreUri")).option("checkpointLocation","/tmp/checkpoint1").start()
    ```

    >[!Important]
    > Les options `metastoreUri` et `database` doivent actuellement être définies manuellement en raison d'un problème connu dans Apache Spark. Pour plus d'informations sur ce problème, voir [SPARK-25460](https://issues.apache.org/jira/browse/SPARK-25460).

1. Revenez à la deuxième session SSH et entrez les valeurs suivantes :

    ```bash
    foo
    HiveSpark
    bar
    ```

1. Revenez à la première session SSH et notez la brève activité. Pour afficher les données, utilisez la commande suivante :

    ```scala
    hive.table("stream_table").show()
    ```

Utilisez **Ctrl + C** pour arrêter netcat sur la deuxième session SSH. Utilisez `:q` pour quitter l’interpréteur de commandes de Spark sur la première session SSH.

## <a name="next-steps"></a>Étapes suivantes

* [Intégration de HWC avec Apache Spark et Apache Hive](./apache-hive-warehouse-connector.md)
* [Utiliser Interactive Query avec HDInsight](./apache-interactive-query-get-started.md)
* [Intégration de HWC avec Apache Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)
