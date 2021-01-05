---
title: Opérations DDL dans l’API Cassandra Azure Cosmos DB à partir de Spark
description: Cet article décrit en détail les opérations DDL d’espace de clés et de table dans l’API Cassandra Azure Cosmos DB à partir de Spark.
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/07/2020
ms.openlocfilehash: 73d31fff362807937cbd87b8e1313cf601909802
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93092175"
---
# <a name="ddl-operations-in-azure-cosmos-db-cassandra-api-from-spark"></a>Opérations DDL dans l’API Cassandra Azure Cosmos DB à partir de Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article décrit en détail les opérations DDL d’espace de clés et de table dans l’API Cassandra Azure Cosmos DB à partir de Spark.

## <a name="cassandra-api-related-configuration"></a>Configuration liée à l’API Cassandra 

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

## <a name="keyspace-ddl-operations"></a>Opérations DDL d’espace de clés

### <a name="create-a-keyspace"></a>Créer un espace de clés

```scala
//Cassandra connector instance
val cdbConnector = CassandraConnector(sc)

// Create keyspace
cdbConnector.withSessionDo(session => session.execute("CREATE KEYSPACE IF NOT EXISTS books_ks WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1 } "))
```

#### <a name="validate-in-cqlsh"></a>Valider dans cqlsh

Exécutez la commande suivante dans cqlsh ; vous devriez voir l’espace de clés créé précédemment.

```bash
DESCRIBE keyspaces;
```

### <a name="drop-a-keyspace"></a>Supprimer un espace de clés

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP KEYSPACE books_ks"))
```

#### <a name="validate-in-cqlsh"></a>Valider dans cqlsh

```bash
DESCRIBE keyspaces;
```
## <a name="table-ddl-operations"></a>Opérations DDL de table

**Considérations :**  

- Le débit peut être assigné au niveau de la table à l’aide de l’instruction create table.  
- Une clé de partition peut stocker 20 Go de données.  
- Un enregistrement peut stocker un maximum de 2 Mo de données.  
- Une plage de clés de partition peut stocker plusieurs clés de partition.

### <a name="create-a-table"></a>Créer une table

```scala
cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks1.books(book_id TEXT,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT, PRIMARY KEY(book_id,book_pub_year)) WITH cosmosdb_provisioned_throughput=4000 , WITH default_time_to_live=630720000;"))
```

#### <a name="validate-in-cqlsh"></a>Valider dans cqlsh

Exécutez la commande suivante dans cqlsh ; vous devriez voir la table nommée «books» : 

```bash
USE books_ks;
DESCRIBE books;
```

Les valeurs de durée de vie par défaut et de débit provisionnées ne figurent pas dans la sortie de la commande précédente ; vous pouvez les obtenir à partir du portail.

### <a name="alter-table"></a>Modifier une table

Vous pouvez modifier les valeurs suivantes à l’aide de la commande alter table :

* Débit provisionné 
* Durée de vie
<br>Les modifications de colonne ne sont pas prises en charge.

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("ALTER TABLE books_ks.books WITH cosmosdb_provisioned_throughput=8000, WITH default_time_to_live=0;"))
```

### <a name="drop-table"></a>Supprimer une table

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP TABLE IF EXISTS books_ks.books;"))
```

#### <a name="validate-in-cqlsh"></a>Valider dans cqlsh

Exécutez la commande suivante dans cqlsh ; vous devriez voir que la table « books » n’est plus disponible :

```bash
USE books_ks;
DESCRIBE tables;
```

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé l’espace de clés et la table, passez aux articles suivants, qui traitent, entre autres, des opérations CRUD :
 
* [Opérations de création et d’insertion](cassandra-spark-create-ops.md)  
* [Opérations de lecture](cassandra-spark-read-ops.md)  
* [Opérations d’upsert](cassandra-spark-upsert-ops.md)  
* [Opérations de suppression](cassandra-spark-delete-ops.md)  
* [Opérations d’agrégation](cassandra-spark-aggregation-ops.md)  
* [Opérations de copie de table](cassandra-spark-table-copy-ops.md)  
