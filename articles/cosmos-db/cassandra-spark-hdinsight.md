---
title: Accéder à l’API Cassandra Azure Cosmos DB à partir de Spark sur YARN avec HDInsight
description: Cet article explique comment utiliser l’API Cassandra Azure Cosmos DB à partir de Spark sur YARN avec HDInsight.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: e09ac39037024b5c360af4bb6ada26590b142924
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359589"
---
# <a name="access-azure-cosmos-db-cassandra-api-from-spark-on-yarn-with-hdinsight"></a>Accéder à l’API Cassandra Azure Cosmos DB à partir de Spark sur YARN avec HDInsight
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article explique comment accéder à l’API Cassandra Azure Cosmos DB à partir de Spark sur YARN avec HDInsight-Spark au moyen de spark-shell. HDInsight est le service PaaS Hadoop Hortonworks de Microsoft sur Azure, qui tire parti du stockage d’objets pour HDFS et se présente sous diverses variantes, notamment [Spark](../hdinsight/spark/apache-spark-overview.md).  Bien que le contenu de ce document référence HDInsight-Spark, il est applicable à toutes les distributions Hadoop.  

## <a name="prerequisites"></a>Conditions préalables requises

* [Provisionner l’API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Passer en revue les principes fondamentaux de la connexion à l’API Cassandra Azure Cosmos DB](cassandra-spark-generic.md)

* [Provisionner un cluster HDInsight-Spark](../hdinsight/spark/apache-spark-jupyter-spark-sql.md)

* [Consulter les exemples de code illustrant l’utilisation de l’API Cassandra](cassandra-spark-generic.md#next-steps)

* [Utiliser cqlsh pour la validation si vous le souhaitez](cassandra-spark-generic.md#connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **Configuration de l’API Cassandra dans Spark2** : le connecteur Spark pour Cassandra nécessite que les informations de la connexion Cassandra soient initialisées dans le cadre du contexte Spark. Quand vous lancez un bloc-notes Jupyter, la session et le contexte spark sont déjà initialisés ; nous vous déconseillons d’arrêter et de réinitialiser le contexte Spark tant qu’il n’est pas doté de la configuration complète définie dans le cadre du démarrage de bloc-notes Jupyter HDInsight par défaut. Une solution de contournement consiste à ajouter les détails de l’instance Cassandra à la configuration du service Spark2 Ambari. Cette opération est nécessaire une seule fois par cluster qui requiert un redémarrage du service Spark2.
 
  1. Accédez au service Spark2 Ambari, puis sélectionnez les configurations.

  2. Ensuite, accédez aux valeurs spark2-defaults personnalisées et ajoutez une nouvelle propriété avec le code suivant, puis redémarrez le service Spark2 :

  ```scala
  spark.cassandra.connection.host=YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com<br>
  spark.cassandra.connection.port=10350<br>
  spark.cassandra.connection.ssl.enabled=true<br>
  spark.cassandra.auth.username=YOUR_COSMOSDB_ACCOUNT_NAME<br>
  spark.cassandra.auth.password=YOUR_COSMOSDB_KEY<br>
  ```

## <a name="access-azure-cosmos-db-cassandra-api-from-spark-shell"></a>Accéder à l’API Cassandra Azure Cosmos DB à partir de l’interpréteur de commandes Spark

L’interpréteur de commandes Spark est utilisé à des fins de test et d’exploration.

* Lancez spark-shell avec les dépendances maven requises compatibles avec la version Spark de votre cluster.

  ```scala
  spark-shell --packages "com.datastax.spark:spark-cassandra-connector_2.11:2.3.0,com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0"
  ```

* Exécutez certaines opérations DDL et DML.

  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}

  import spark.implicits._
  import org.apache.spark.sql.functions._
  import org.apache.spark.sql.Column
  import org.apache.spark.sql.types.{StructType, StructField, StringType, IntegerType,LongType,FloatType,DoubleType, TimestampType}
  import org.apache.spark.sql.cassandra._

  //Spark connector
  import com.datastax.spark.connector._
  import com.datastax.spark.connector.cql.CassandraConnector

  //CosmosDB library for multiple retry
  import com.microsoft.azure.cosmosdb.cassandra

  // Specify connection factory for Cassandra
  spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

  // Parallelism and throughput configs
  spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
  spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
  spark.conf.set("spark.cassandra.output.concurrent.writes", "100")
  spark.conf.set("spark.cassandra.concurrent.reads", "512")
  spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
  spark.conf.set("spark.cassandra.connection.keep_alive_ms", "60000000") //Increase this number as needed
  ```

* Exécutez des opérations CRUD.

  ```scala
  //1) Create table if it does not exist
  val cdbConnector = CassandraConnector(sc)
  cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT PRIMARY KEY,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT) WITH cosmosdb_provisioned_throughput=4000;"))

  //2) Delete data from potential prior runs
  cdbConnector.withSessionDo(session => session.execute("DELETE FROM books_ks.books WHERE book_id IN ('b00300','b00001','b00023','b00501','b09999','b01001','b00999','b03999','b02999','b000009');"))

  //3) Generate a few rows
  val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887,11.33),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890,22.45),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892,19.83),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893,14.22),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901,12.25)
  ).toDF("book_id", "book_author", "book_name", "book_pub_year","book_price")

  //4) Persist
  booksDF.write.mode("append").format("org.apache.spark.sql.cassandra").options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000")).save()

  //5) Read the data in the table
  spark.read.format("org.apache.spark.sql.cassandra").options(Map( "table" -> "books", "keyspace" -> "books_ks")).load.show
  ```

## <a name="access-azure-cosmos-db-cassandra-api-from-jupyter-notebooks"></a>Accéder à l’API Cassandra Azure Cosmos DB à partir de blocs-notes Jupyter

HDInsight-Spark est fourni avec les services de bloc-notes Zeppelin et Jupyter. Ce sont deux environnements de bloc-notes basés sur le web qui prennent en charge Scala et Python. Les blocs-notes sont parfaits pour l’analytique exploratoire interactive et la collaboration, mais ne sont pas conçus pour les processus opérationnels ou de production.

Les blocs-notes Jupyter suivants peuvent être chargés dans votre cluster HDInsight Spark et fournissent des exemples d’utilisation de l’API Cassandra Azure Cosmos DB immédiatement exploitables. Veillez à consulter le premier bloc-notes `1.0-ReadMe.ipynb`, qui contient la configuration du service Spark requise pour la connexion à l’API Cassandra Azure Cosmos DB.

Téléchargez ces blocs-notes sous [azure-cosmos-db-cassandra-api-spark-notebooks-jupyter](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-jupyter/blob/main/scala/) sur votre machine.
  
### <a name="how-to-upload"></a>Procédure à suivre pour le chargement :
Quand vous lancez Jupyter, accédez à Scala. Créez un répertoire, puis chargez-y les blocs-notes. Le bouton de chargement est situé en haut, à droite.  

### <a name="how-to-run"></a>Procédure à suivre pour l’exécution :
Exécuter les blocs-notes et chaque cellule de bloc-notes séquentiellement.  Cliquez sur le bouton d’exécution en haut de chaque bloc-notes pour exécuter toutes les cellules, ou appuyez sur Maj+Entrée pour chaque cellule.

## <a name="access-with-azure-cosmos-db-cassandra-api-from-your-spark-scala-program"></a>Accéder à l’API Cassandra Azure Cosmos DB à partir de votre programme Scala Spark

Pour les processus automatisés dans l’environnement de production, les programmes Spark sont envoyés au cluster au moyen du script [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html).

## <a name="next-steps"></a>Étapes suivantes

* [Guide pratique pour créer un programme Scala Spark dans un IDE et l’envoyer au cluster HDInsight Spark via Livy en vue de son exécution](../hdinsight/spark/apache-spark-create-standalone-application.md)

* [Guide pratique pour se connecter à l’API Cassandra Azure Cosmos DB à partir d’un programme Scala Spark](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/main/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)

* [Liste complète d’exemples de code illustrant l’utilisation de l’API Cassandra](cassandra-spark-generic.md)
