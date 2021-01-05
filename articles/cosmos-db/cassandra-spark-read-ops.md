---
title: Lecture des données de tables de l’API Cassandra à l’aide de Spark
titleSufix: Azure Cosmos DB
description: Cet article explique comment lire les données stockées des tables de l’API Cassandra dans Azure Cosmos DB.
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 06/02/2020
ms.custom: seodec18
ms.openlocfilehash: ceede96cbf3be12a6129e27d34e318e4c4163458
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93073494"
---
# <a name="read-data-from-azure-cosmos-db-cassandra-api-tables-using-spark"></a>Lire les données de tables de l’API Cassandra dans Azure Cosmos DB à l’aide de Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

 Cet article explique comment lire les données stockées dans l’API Cassandra Azure Cosmos DB à partir de Spark.

## <a name="cassandra-api-configuration"></a>Configuration de l’API Cassandra
```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra

//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")
//Throughput-related...adjust as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```
## <a name="dataframe-api"></a>API Dataframe

### <a name="read-table-using-sessionreadformat-command"></a>Lire une table avec la commande session.read.format

```scala
val readBooksDF = sqlContext
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load

readBooksDF.explain
readBooksDF.show
```
### <a name="read-table-using-sparkreadcassandraformat"></a>Lire une table avec la commande spark.read.cassandraFormat 

```scala
val readBooksDF = spark.read.cassandraFormat("books", "books_ks", "").load()
```

### <a name="read-specific-columns-in-table"></a>Lire des colonnes spécifiques d’une table

```scala
val readBooksDF = spark
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load
  .select("book_name","book_author", "book_pub_year")

readBooksDF.printSchema
readBooksDF.explain
readBooksDF.show
```

### <a name="apply-filters"></a>Appliquer des filtres

Vous pouvez effectuer un pushdown de prédicats vers la base de données pour optimiser encore les requêtes Spark. Un prédicat est une condition sur une requête, généralement dans la clause WHERE, qui retourne true ou false. Un pushdown de prédicat filtre les données dans la requête de base de données, réduisant ainsi le nombre d’entrées récupérées à partir de la base de données et améliorant les performances des requêtes. Par défaut, l’API Spark DataSet effectue automatiquement un pushdown des clauses WHERE valides vers la base de données. 

```scala
val df = spark.read.cassandraFormat("books", "books_ks").load
df.explain
val dfWithPushdown = df.filter(df("book_pub_year") > 1891)
dfWithPushdown.explain

readBooksDF.printSchema
readBooksDF.explain
readBooksDF.show
```

La section PushedFilters du plan physique comprend le filtre pushdown GreaterThan. 

:::image type="content" source="./media/cassandra-spark-read-ops/pushdown-predicates.png" alt-text="partitions":::

## <a name="rdd-api"></a>API pour le jeu de donnée distribué résilient

### <a name="read-table"></a>Lire une table
```scala
val bookRDD = sc.cassandraTable("books_ks", "books")
bookRDD.take(5).foreach(println)
```

### <a name="read-specific-columns-in-table"></a>Lire des colonnes spécifiques d’une table

```scala
val booksRDD = sc.cassandraTable("books_ks", "books").select("book_id","book_name").cache
booksRDD.take(5).foreach(println)
```

## <a name="sql-views"></a>Vues SQL 

### <a name="create-a-temporary-view-from-a-dataframe"></a>Créer une vue temporaire à partir d’un dataframe

```scala
spark
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load.createOrReplaceTempView("books_vw")
```

### <a name="run-queries-against-the-view"></a>Exécuter des requêtes sur la vue

```sql
select * from books_vw where book_pub_year > 1891
```

## <a name="next-steps"></a>Étapes suivantes

Voici des articles supplémentaires sur l’utilisation de l’API Cassandra Azure Cosmos DB à partir de Spark :
 
 * [Opérations d’upsert](cassandra-spark-upsert-ops.md)
 * [Opérations de suppression](cassandra-spark-delete-ops.md)
 * [Opérations d’agrégation](cassandra-spark-aggregation-ops.md)
 * [Opérations de copie de table](cassandra-spark-table-copy-ops.md)

