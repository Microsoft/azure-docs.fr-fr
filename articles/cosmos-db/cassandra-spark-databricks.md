---
title: Accéder à l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks
description: Cet article explique comment utiliser l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 5aaf31d8c629e0770b913485de175c38f708390c
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359623"
---
# <a name="access-azure-cosmos-db-cassandra-api-data-from-azure-databricks"></a>Accéder aux données de l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article explique comment utiliser l’API Cassandra Azure Cosmos DB à partir de Spark sur [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks).

## <a name="prerequisites"></a>Prérequis

* [Provisionner un compte de l’API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Passer en revue les principes fondamentaux de la connexion à l’API Cassandra Azure Cosmos DB](cassandra-spark-generic.md)

* [Provisionner un cluster Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal)

* [Consulter les exemples de code illustrant l’utilisation de l’API Cassandra](cassandra-spark-generic.md#next-steps)

* [Utiliser cqlsh pour la validation si vous le souhaitez](cassandra-spark-generic.md#connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **Configuration de l’instance de l’API Cassandra pour le connecteur Cassandra :**

  Le connecteur pour l’API Cassandra nécessite que les informations de connexion Cassandra soient initialisées dans le cadre du contexte Spark. Quand vous lancez un bloc-notes Databricks, le contexte spark est déjà initialisé et il n’est pas recommandé de l’arrêter et de le réinitialiser. Une solution consiste à ajouter la configuration de l’instance de l’API Cassandra à un niveau de cluster, dans la configuration du cluster spark. Cette opération s’effectue une seule fois par cluster. Ajoutez à la configuration Spark le code suivant, où chaque paire clé-valeur utilise un espace en guise de séparation :
 
  ```scala
  spark.cassandra.connection.host YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com
  spark.cassandra.connection.port 10350
  spark.cassandra.connection.ssl.enabled true
  spark.cassandra.auth.username YOUR_COSMOSDB_ACCOUNT_NAME
  spark.cassandra.auth.password YOUR_COSMOSDB_KEY
  ```

## <a name="add-the-required-dependencies"></a>Ajouter les dépendances nécessaires

* **Connecteur Spark Cassandra :** Pour intégrer l’API Cassandra Azure Cosmos DB à Spark, vous devez attacher le connecteur Cassandra au cluster Azure Databricks. Pour attacher le cluster :

  * Passez en revue la version du runtime de Databricks et la version de Spark. Ensuite, recherchez les [coordonnées Maven](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector) qui sont compatibles avec le connecteur Spark Cassandra, puis attachez-le au cluster. Consultez l’article [« Upload a Maven package or Spark package »](https://docs.databricks.com/user-guide/libraries.html) (Charger un package Maven ou un package Spark) pour savoir comment attacher la bibliothèque de connecteur au cluster. Par exemple, la coordonnée maven pour « Databricks Runtime version 4.3 », « Spark 2.3.1 » et « Scala 2.11 » est `spark-cassandra-connector_2.11-2.3.1`

* **Bibliothèque spécifique à l’API Cassandra Azure Cosmos DB :** une fabrique de connexion personnalisée est nécessaire pour configurer la stratégie de nouvelles tentatives à partir du connecteur Spark sur l’API Cassandra Azure Cosmos DB. Ajoutez les [coordonnées maven](https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.0.0/jar)`com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` pour attacher la bibliothèque au cluster.

## <a name="sample-notebooks"></a>Exemples de notebooks

Une liste [d’exemples de blocs-notes](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/main/notebooks/scala) Azure Databricks est téléchargeable à partir du dépôt GitHub. Ces exemples montrent comment se connecter à l’API Cassandra Azure Cosmos DB à partir de Spark et effectuer différentes opérations CRUD sur les données. Vous pouvez également [importer tous les blocs-notes](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/main/dbc) dans votre espace de travail du cluster Databricks, puis exécuter l’espace de travail. 

## <a name="accessing-azure-cosmos-db-cassandra-api-from-spark-scala-programs"></a>Accès à l’API Cassandra Azure Cosmos DB à partir de programmes Scala Spark

Les programmes Spark à exécuter en tant que processus automatisés sur Azure Databricks sont soumis au cluster au moyen du script [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html) et planifiés pour être exécutés par le biais de travaux Azure Databricks.

Les liens suivants vous aident à vous familiariser avec la génération de programmes Scala Spark pour interagir avec l’API Cassandra Azure Cosmos DB.
* [Guide pratique pour se connecter à l’API Cassandra Azure Cosmos DB à partir d’un programme Scala Spark](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/main/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)
* [How to run a Spark Scala program as an automated job on Azure Databricks](https://docs.azuredatabricks.net/user-guide/jobs.html) (Guide pratique pour exécuter un programme Scala Spark en tant que travail automatisé sur Azure Databricks)
* [Liste complète d’exemples de code illustrant l’utilisation de l’API Cassandra](cassandra-spark-generic.md#next-steps)

## <a name="next-steps"></a>Étapes suivantes

Familiarisez-vous avec la [création d’un compte d’API Cassandra, d’une base de données et d’une table](create-cassandra-api-account-java.md) à l’aide d’une application Java.