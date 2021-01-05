---
title: Créer ou insérer des données dans l’API Cassandra Azure Cosmos DB à partir de Spark
description: Cet article explique en détail comment insérer des exemples de données dans des tables d’API Cassandra Azure Cosmos DB
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 3ec44d20b763a98683d9b947c94ad6be75180113
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93092234"
---
# <a name="createinsert-data-into-azure-cosmos-db-cassandra-api-from-spark"></a>Créer/insérer des données dans l’API Cassandra Azure Cosmos DB à partir de Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]
 
Cet article explique comment insérer des exemples de données dans une table de l’API Cassandra Azure Cosmos DB à partir de Spark.

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

### <a name="create-a-dataframe-with-sample-data"></a>Créer un dataframe avec des exemples de données

```scala
// Generate a dataframe containing five records
val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
).toDF("book_id", "book_author", "book_name", "book_pub_year")

//Review schema
booksDF.printSchema

//Print
booksDF.show
```

> [!NOTE]
> La fonctionnalité « Créer si inexistant », au niveau des lignes, n’est pas encore prise en charge.

### <a name="persist-to-azure-cosmos-db-cassandra-api"></a>Conserver dans l’API Cassandra Azure Cosmos DB

Lors de l’enregistrement de données, vous pouvez également définir des paramètres de durée de vie et de stratégie de cohérence, comme indiqué dans l’exemple suivant :

```scala
//Persist
booksDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000"))
  .save()
```

> [!NOTE]
> La durée de vie au niveau des colonnes n’est pas encore prise en charge.

#### <a name="validate-in-cqlsh"></a>Valider dans cqlsh

```sql
use books_ks;
select * from books;
```

## <a name="resilient-distributed-database-rdd-api"></a>API RDD (base de données distribuée résiliente)

### <a name="create-a-rdd-with-sample-data"></a>Créer une base de données distribuée résiliente avec des exemples de données
```scala
//Delete records created in the previous section 
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("delete from books_ks.books where book_id in ('b00300','b00001','b00023','b00501','b09999','b01001','b00999','b03999','b02999');"))

//Create RDD
val booksRDD = sc.parallelize(Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
))

//Review
booksRDD.take(2).foreach(println)
```

> [!NOTE]
> La fonctionnalité « Créer si inexistant » n’est pas encore prise en charge.

### <a name="persist-to-azure-cosmos-db-cassandra-api"></a>Conserver dans l’API Cassandra Azure Cosmos DB

Lors de l’enregistrement de données dans l’API Cassandra, vous pouvez également définir des paramètres de durée de vie et de stratégie de cohérence, comme indiqué dans l’exemple suivant :

```scala
import com.datastax.spark.connector.writer._

//Persist
booksRDD.saveToCassandra("books_ks", "books", SomeColumns("book_id", "book_author", "book_name", "book_pub_year"),writeConf = WriteConf(ttl = TTLOption.constant(900000),consistencyLevel = ConsistencyLevel.ALL))
```

#### <a name="validate-in-cqlsh"></a>Valider dans cqlsh

```sql
use books_ks;
select * from books;
```

## <a name="next-steps"></a>Étapes suivantes

Après avoir inséré des données dans la table de l’API Cassandra Azure Cosmos DB, passez aux articles suivants pour effectuer d’autres opérations sur les données stockées dans l’API Cassandra Cosmos DB :
 
* [Opérations de lecture](cassandra-spark-read-ops.md)
* [Opérations d’upsert](cassandra-spark-upsert-ops.md)
* [Opérations de suppression](cassandra-spark-delete-ops.md)
* [Opérations d’agrégation](cassandra-spark-aggregation-ops.md)
* [Opérations de copie de table](cassandra-spark-table-copy-ops.md)

