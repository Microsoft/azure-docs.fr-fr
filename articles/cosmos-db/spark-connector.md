---
title: Connecter Apache Spark à Azure Cosmos DB
description: Découvrez le connecteur Spark Azure Cosmos DB qui vous permet de connecter Apache Spark à Azure Cosmos DB.
author: tknandu
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/21/2019
ms.author: ramkris
ms.openlocfilehash: 06498a27b95a72148497efd2d1e600d802414359
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359555"
---
# <a name="accelerate-big-data-analytics-by-using-the-apache-spark-to-azure-cosmos-db-connector"></a>Accélérer l’analytique en temps réel des Big Data au moyen du connecteur Apache Spark-Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Vous pouvez exécuter des tâches [Spark](https://spark.apache.org/) avec des données stockées dans Azure Cosmos DB à l’aide du connecteur Spark de Cosmos DB. Cosmos peut servir au traitement de lots et de flux, et en tant que couche de service pour un accès à faible latence.

Vous pouvez utiliser le connecteur avec [Azure Databricks](https://azure.microsoft.com/services/databricks) ou [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/), qui fournissent des clusters Spark gérés sur Azure. Le tableau suivant renseigne sur les versions Spark prises en charge.

| Composant | Version |
|---------|-------|
| Apache Spark | 2.4.x, 2.3.x, 2.2.x et 2.1.x |
| Scala | 2.11 |
| Version du runtime Azure Databricks | > 3.4 |

> [!WARNING]
> Ce connecteur prend en charge l’API de base (SQL) d’Azure Cosmos DB.
> Pour l’API Cosmos DB pour MongoDB, utilisez le [connecteur MongoDB Spark](https://docs.mongodb.com/spark-connector/master/).
> Pour l’API Cosmos DB pour Cassandra, utilisez le [connecteur Cassandra Spark](https://github.com/datastax/spark-cassandra-connector).

> [!IMPORTANT]
> Le connecteur Azure Cosmos DB Spark n’est actuellement pas pris en charge sur les comptes [serverless](serverless.md). Ce problème sera traité quand l’offre serverless sera en disponibilité générale.

## <a name="quickstart"></a>Démarrage rapide

* Suivez les étapes de [Prise en main avec le kit de développement logiciel (SDK) Java](./create-sql-api-java.md) pour configurer un compte Cosmos DB, et renseignez quelques données.
* Suivez les étapes de [Prise en main avec Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal) pour configurer un espace de travail et un cluster Azure Databricks.
* Vous pouvez désormais créer de nouveaux notebooks et importer la bibliothèque de connecteur Cosmos DB. Accédez à [Utiliser le connecteur Cosmos DB](#bk_working_with_connector) pour plus d’informations sur comment configurer votre espace de travail.
* La section suivante offre des extraits de code sur la façon de lire et écrire avec le connecteur.

### <a name="batch-reads-from-cosmos-db"></a>Lectures de lots à partir de Cosmos DB

L’extrait de code suivant montre comment créer une tramedonnées Spark à lire à partir de Cosmos DB dans PySpark.

```python
# Read Configuration
readConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_pcoll",
    "query_custom": "SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'" // Optional
}

# Connect via azure-cosmosdb-spark to create Spark DataFrame
flights = spark.read.format(
    "com.microsoft.azure.cosmosdb.spark").options(**readConfig).load()
flights.count()
```

Et le même extrait de code dans Scala :

```scala
// Import Necessary Libraries
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.config.Config

// Read Configuration
val readConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_pcoll",
  "query_custom" -> "SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'" // Optional
))

// Connect via azure-cosmosdb-spark to create Spark DataFrame
val flights = spark.read.cosmosDB(readConfig)
flights.count()
```

### <a name="batch-writes-to-cosmos-db"></a>Écritures de lots dans Cosmos DB

L’extrait de code suivant montre comment écrire une tramedonnées dans Cosmos DB dans PySpark.

```python
# Write configuration
writeConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_fromsea",
    "Upsert": "true"
}

# Write to Cosmos DB from the flights DataFrame
flights.write.mode("overwrite").format("com.microsoft.azure.cosmosdb.spark").options(
    **writeConfig).save()
```

Et le même extrait de code dans Scala :

```scala
// Write configuration

val writeConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_fromsea",
  "Upsert" -> "true"
))

// Write to Cosmos DB from the flights DataFrame
import org.apache.spark.sql.SaveMode
flights.write.mode(SaveMode.Overwrite).cosmosDB(writeConfig)
```

### <a name="streaming-reads-from-cosmos-db"></a>Lectures de diffusion en continu à partir de Cosmos DB

L’extrait de code suivant montre comment se connecter à Azure Cosmos DB et lire des flux de modification.

```python
# Read Configuration
readConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_pcoll",
    "ReadChangeFeed": "true",
    "ChangeFeedQueryName": "Departure-Delays",
    "ChangeFeedStartFromTheBeginning": "false",
    "InferStreamSchema": "true",
    "ChangeFeedCheckpointLocation": "dbfs:/Departure-Delays"
}


# Open a read stream to the Cosmos DB Change Feed via azure-cosmosdb-spark to create Spark DataFrame
changes = (spark
           .readStream
           .format("com.microsoft.azure.cosmosdb.spark.streaming.CosmosDBSourceProvider")
           .options(**readConfig)
           .load())
```
Et le même extrait de code dans Scala :

```scala
// Import Necessary Libraries
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.config.Config

// Read Configuration
val readConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_pcoll",
  "ReadChangeFeed" -> "true",
  "ChangeFeedQueryName" -> "Departure-Delays",
  "ChangeFeedStartFromTheBeginning" -> "false",
  "InferStreamSchema" -> "true",
  "ChangeFeedCheckpointLocation" -> "dbfs:/Departure-Delays"
))

// Open a read stream to the Cosmos DB Change Feed via azure-cosmosdb-spark to create Spark DataFrame
val df = spark.readStream.format(classOf[CosmosDBSourceProvider].getName).options(readConfig).load()
```

### <a name="streaming-writes-to-cosmos-db"></a>Écritures de diffusion en continu dans Cosmos DB

L’extrait de code suivant montre comment écrire une tramedonnées dans Cosmos DB dans PySpark.

```python
# Write configuration
writeConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_fromsea",
    "Upsert": "true",
    "WritingBatchSize": "500",
    "CheckpointLocation": "/checkpointlocation_write1"
}

# Write to Cosmos DB from the flights DataFrame
changeFeed = (changes
              .writeStream
              .format("com.microsoft.azure.cosmosdb.spark.streaming.CosmosDBSinkProvider")
              .outputMode("append")
              .options(**writeconfig)
              .start())
```

Et le même extrait de code dans Scala :

```scala
// Write configuration

val writeConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_fromsea",
  "Upsert" -> "true",
  "WritingBatchSize" -> "500",
  "CheckpointLocation" -> "/checkpointlocation_write1"
))

// Write to Cosmos DB from the flights DataFrame
df
.writeStream
.format(classOf[CosmosDBSinkProvider].getName)
.options(writeConfig)
.start()
```
Pour plus d’extraits de code et d’exemples de bout en bout, consultez [Jupyter](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples/notebooks).

## <a name="working-with-the-connector"></a><a name="bk_working_with_connector"></a> Utilisation du connecteur

Vous pouvez créer le connecteur à partir de la source dans GitHub, ou télécharger les fichiers uber jars à partir de Maven via les liens ci-dessous.

| Spark | Scala | Version la plus récente |
|---|---|---|
| 2.4.0 | 2.11 | [azure-cosmosdb-spark_lkg_version](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG)
| 2.3.0 | 2.11 | [azure-cosmosdb-spark_2.3.0_2.11_1.3.3](https://search.maven.org/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.3.0_2.11/1.3.3/jar)
| 2.2.0 | 2.11 | [azure-cosmosdb-spark_2.2.0_2.11_1.1.1](https://search.maven.org/#artifactdetails%7Ccom.microsoft.azure%7Cazure-cosmosdb-spark_2.2.0_2.11%7C1.1.1%7Cjar)
| 2.1.0 | 2.11 | [azure-cosmosdb-spark_2.1.0_2.11_1.2.2](https://search.maven.org/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.1.0_2.11/1.2.2/jar)

### <a name="using-databricks-notebooks"></a>Utilisation des notebooks Databricks

Créez une bibliothèque avec votre espace de travail Databricks en suivant les instructions du Guide Azure Databricks > [Utiliser le connecteur Spark Azure Cosmos DB](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/cosmosdb-connector.html)

> [!NOTE]
> La page **Utiliser le connecteur Spark Azure Cosmos DB** n'est actuellement pas à jour. Au lieu de télécharger les six fichiers jars distincts dans six bibliothèques différentes, vous pouvez télécharger le fichier uber jar depuis Maven sur [azure-cosmosdb-spark_lkg_version](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG) et installer ce fichier/cette bibliothèque.
> 

### <a name="using-spark-cli"></a>Utilisation de spark-cli

Pour utiliser le connecteur avec l’interface de ligne de commande spark-cli (`spark-shell`, `pyspark`, `spark-submit`), vous pouvez utiliser le paramètre `--packages` avec les [coordonnées maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.4.0_2.11) du connecteur.

```sh
spark-shell --master yarn --packages "com.microsoft.azure:azure-cosmosdb-spark_2.4.0_2.11:1.4.0"

```

### <a name="using-jupyter-notebooks"></a>Utilisation de notebooks Jupyter

Si vous utilisez des notebooks Jupyter au sein d’HDInsight, vous pouvez utiliser la cellule `%%configure` spark-magic pour spécifier les coordonnées maven du connecteur.

```python
{ "name":"Spark-to-Cosmos_DB_Connector",
  "conf": {
    "spark.jars.packages": "com.microsoft.azure:azure-cosmosdb-spark_2.4.0_2.11:1.4.0",
    "spark.jars.excludes": "org.scala-lang:scala-reflect"
   }
   ...
}
```

> Remarque : l’inclusion de `spark.jars.excludes` est spécifique afin de supprimer les conflits potentiels entre le connecteur, Apache Spark et Livy.

### <a name="build-the-connector"></a>Créer le connecteur

Actuellement, le projet de ce connecteur utilise `maven`. Pour créer sans dépendances, vous pouvez exécuter :

```sh
mvn clean package
```

## <a name="working-with-our-samples"></a>Utilisation de nos exemples

Le [référentiel GitHub Cosmos DB Spark](https://github.com/Azure/azure-cosmosdb-spark) dispose des exemples de bloc-notes et de scripts suivants, que vous pouvez essayer.

* **Performances de ponctualité des vols avec Spark et Cosmos DB (Seattle)** [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/On-Time%20Flight%20Performance%20with%20Spark%20and%20Cosmos%20DB%20-%20Seattle.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/On-Time%20Flight%20Performance%20with%20Spark%20and%20Cosmos%20DB%20-%20Seattle.html) : Connectez Spark à Cosmos DB à l’aide du service Jupyter notebook pour présenter Spark SQL, GraphFrames et prédire les retards de vol à l’aide de pipelines ML.
* **Twitter Source avec Apache Spark et Azure Cosmos DB Change Feed** : [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Twitter%20with%20Spark%20and%20Azure%20Cosmos%20DB%20Change%20Feed.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Twitter%20with%20Spark%20and%20Azure%20Cosmos%20DB%20Change%20Feed.html)
* **À l’aide d’Apache Spark pour interroger Cosmos DB Graphs** : [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Using%20Apache%20Spark%20to%20query%20Cosmos%20DB%20Graphs.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Using%20Apache%20Spark%20to%20query%20Cosmos%20DB%20Graphs.html)
* **[Connectez Azure Databricks à Azure Cosmos DB](https://docs.databricks.com/spark/latest/data-sources/azure/cosmosdb-connector.html)** à l’aide de `azure-cosmosdb-spark`.  Est aussi lié ici une version d’Azure Databricks du [notebook On-Time Flight Performance](https://github.com/dennyglee/databricks/tree/master/notebooks/Users/denny%40databricks.com/azure-databricks).
* **[Architecture lambda avec Azure Cosmos DB et HDInsight (Apache Spark)](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/readme.md)** : Vous pouvez réduire les frais opérationnels de gestion des pipelines big data à l’aide de Cosmos DB et Spark.

## <a name="more-information"></a>Informations complémentaires

Informations complémentaires sur le `azure-cosmosdb-spark` [wiki](https://github.com/Azure/azure-cosmosdb-spark/wiki), dont :

* [Guide d’utilisation du connecteur Spark Azure Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Azure-Cosmos-DB-Spark-Connector-User-Guide)
* [Exemples d’agrégations](https://github.com/Azure/azure-documentdb-spark/wiki/Aggregations-Examples)

### <a name="configuration-and-setup"></a>Configuration et installation

* [Configuration du connecteur Spark](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuration-references)
* [Configuration Spark-Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Spark-to-Cosmos-DB-Connector-Setup) (en cours)
* [Configuration de Power BI Direct Query dans Azure Cosmos DB via Apache Spark (HDI)](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuring-Power-BI-Direct-Query-to-Azure-Cosmos-DB-via-Apache-Spark-(HDI))

### <a name="troubleshooting"></a>Dépannage

* [Utilisation d’agrégats Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Troubleshooting:-Using-Cosmos-DB-Aggregates)
* [Problèmes connus](https://github.com/Azure/azure-cosmosdb-spark/wiki/Known-Issues)

### <a name="performance"></a>Performances

* [Conseils sur les performances](https://github.com/Azure/azure-cosmosdb-spark/wiki/Performance-tips)
* [Séries de tests de requête](https://github.com/Azure/azure-documentdb-spark/wiki/Query-Test-Runs)
* [Séries de tests d’écriture](https://github.com/Azure/azure-cosmosdb-spark/wiki/Writing-Test-Runs)

### <a name="change-feed"></a>Modifier le flux

* [Diffuser des modifications de traitement avec le flux de modification Azure CosmosDB et Apache Spark](https://github.com/Azure/azure-cosmosdb-spark/wiki/Stream-Processing-Changes-using-Azure-Cosmos-DB-Change-Feed-and-Apache-Spark)
* [Démonstrations de flux de modification](https://github.com/Azure/azure-cosmosdb-spark/wiki/Change-Feed-demos)
* [Démonstrations de streaming structuré](https://github.com/Azure/azure-cosmosdb-spark/wiki/Structured-Stream-demos)

### <a name="monitoring"></a>Surveillance

* [Surveillance des tâches Spark avec Application Insights](https://github.com/Azure/azure-cosmosdb-spark/tree/2.3/samples/monitoring)

## <a name="next-steps"></a>Étapes suivantes

Si cela n’est pas déjà fait, téléchargez le connecteur Spark-Azure Cosmos DB à partir du dépôt GitHub [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark). Explorez les ressources supplémentaires suivantes dans le dépôt :

* [Aggregations Examples](https://github.com/Azure/azure-cosmosdb-spark/wiki/Aggregations-Examples) (Exemples d’agrégations)
* [Sample Scripts and Notebooks](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples) (Exemples de scripts et Notebooks)

Vous pouvez également consulter [Apache Spark SQL, DataFrames, and Datasets Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html) (Guide sur Apache Spark SQL, les tableaux de données et les jeux de données) et l’article [Apache Spark sur Azure HDInsight](../hdinsight/spark/apache-spark-jupyter-spark-sql.md).