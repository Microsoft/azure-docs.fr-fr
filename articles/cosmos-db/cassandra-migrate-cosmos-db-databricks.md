---
title: Migrer des données d’Apache Cassandra vers l'API Cassandra Azure Cosmos DB à l'aide de Databricks (Spark)
description: Découvrez comment migrer des données d'une base de données Apache Cassandra vers l'API Cassandra Azure Cosmos DB à l'aide d’Azure Databricks et Spark.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 11/16/2020
ms.author: thvankra
ms.reviewer: thvankra
ms.openlocfilehash: 74088d749279ab72851e714a50b558dc2adbc0d7
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516548"
---
# <a name="migrate-data-from-cassandra-to-azure-cosmos-db-cassandra-api-account-using-azure-databricks"></a>Migrer des données de Cassandra vers un compte API Cassandra Azure Cosmos DB à l'aide d’Azure Databricks
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

L'API Cassandra d'Azure Cosmos DB est devenue un excellent choix pour les charges de travail d'entreprise exécutées sur Apache Cassandra, notamment pour les raisons suivantes : 

* **Aucune surcharge liée à la gestion et à la surveillance :** l'API élimine les surcharges liées à la gestion et à la surveillance d'une multitude de paramètres dans les fichiers du système d'exploitation, les fichiers JVM et les fichiers yaml, ainsi qu'à leurs interactions.

* **Réduction significative des coûts :** vous pouvez réaliser des économies grâce à Azure Cosmos DB, notamment sur les machines virtuelles, la bande passante et les licences applicables. En outre, vous n'avez pas à gérer les coûts liés aux centres de données, aux serveurs, au stockage SSD, à la mise en réseau et à la consommation électrique. 

* **Possibilité d'utiliser le code et les outils existants** : Azure Cosmos DB fournit une compatibilité au niveau du protocole filaire avec les SDK et outils Cassandra existants. Cette compatibilité garantit la possibilité d’utiliser votre code base de code avec l’API Cassandra Azure Cosmos DB sans changements majeurs.

Il existe plusieurs façons de migrer des charges de travail de base de données d'une plateforme vers une autre. [Azure Databricks](https://azure.microsoft.com/services/databricks/) est une offre de plateforme en tant que service pour [Apache Spark](https://spark.apache.org/) permettant d’effectuer des migrations hors connexion à grande échelle. Cet article décrit les étapes à suivre pour migrer des données d’espaces de clés/de tables Apache Cassandra vers l'API Cassandra Azure Cosmos DB à l'aide d’Azure Databricks.

## <a name="prerequisites"></a>Prérequis

* [Provisionner un compte de l’API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Passer en revue les principes fondamentaux de la connexion à l’API Cassandra Azure Cosmos DB](cassandra-spark-generic.md)

* Passez en revue les [fonctionnalités prises en charge dans l’API Cassandra Azure Cosmos DB](cassandra-support.md) pour garantir la compatibilité.

* Assurez-vous que vous avez déjà créé un espace de clé vide et des tables dans votre compte d’API Cassandra Azure Cosmos DB cible.

* [Utiliser cqlsh ou un interpréteur de commandes hébergé à des fins de validation](cassandra-support.md#hosted-cql-shell-preview)

## <a name="provision-an-azure-databricks-cluster"></a>Provisionner un cluster Azure Databricks

Vous pouvez suivre les instructions pour [approvisionner un cluster Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal). Notez cependant qu’Apache Spark 3.x n’est actuellement pas pris en charge pour le connecteur Apache Cassandra. Vous devez approvisionner un runtime Databricks avec une version v2.x prise en charge d’Apache Spark. Nous vous recommandons de sélectionner une version du runtime de Databricks qui prend en charge la version la plus récente de Spark 2.x, avec la version 2.11 ou antérieure de Scala :

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-runtime.png" alt-text="Runtime Databricks":::


## <a name="add-dependencies"></a>Ajout de dépendances

Vous devez ajouter la bibliothèque du connecteur Apache Spark Cassandra à votre cluster pour vous connecter aux points de terminaison natifs et Cosmos DB. Dans votre cluster, sélectionnez bibliothèques-> installer nouveau -> maven -> rechercher des packages :

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-search-packages.png" alt-text="Databricks - Rechercher des packages":::

Entrez `Cassandra` dans la zone de recherche, sélectionnez le dernier référentiel maven `spark-cassandra-connector` disponible, puis sélectionnez Installer :

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-search-packages-2.png" alt-text="Databricks - Sélectionner des packages":::

> [!NOTE]
> Veillez à redémarrer le cluster Databricks après l’installation de la bibliothèque du connecteur Cassandra.

## <a name="create-scala-notebook-for-migration"></a>Créer un notebook Scala pour la migration

Créez un notebook Scala dans Databricks avec le code suivant. Remplacez vos configurations Cassandra source et cible par les informations d’identification correspondantes, ainsi que les espaces de clés et tables sources/cibles, puis exécutez :

```scala
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql._
import org.apache.spark.SparkContext

// source cassandra configs
val nativeCassandra = Map( 
    "spark.cassandra.connection.host" -> "<Source Cassandra Host>",
    "spark.cassandra.connection.port" -> "9042",
    "spark.cassandra.auth.username" -> "<USERNAME>",
    "spark.cassandra.auth.password" -> "<PASSWORD>",
    "spark.cassandra.connection.ssl.enabled" -> "false",
    "keyspace" -> "<KEYSPACE>",
    "table" -> "<TABLE>"
)

//target cassandra configs
val cosmosCassandra = Map( 
    "spark.cassandra.connection.host" -> "<USERNAME>.cassandra.cosmos.azure.com",
    "spark.cassandra.connection.port" -> "10350",
    "spark.cassandra.auth.username" -> "<USERNAME>",
    "spark.cassandra.auth.password" -> "<PASSWORD>",
    "spark.cassandra.connection.ssl.enabled" -> "true",
    "keyspace" -> "<KEYSPACE>",
    "table" -> "<TABLE>",
    //throughput related settings below - tweak these depending on data volumes. 
    "spark.cassandra.output.batch.size.rows"-> "1",
    "spark.cassandra.connection.connections_per_executor_max" -> "10",
    "spark.cassandra.output.concurrent.writes" -> "1000",
    "spark.cassandra.concurrent.reads" -> "512",
    "spark.cassandra.output.batch.grouping.buffer.size" -> "1000",
    "spark.cassandra.connection.keep_alive_ms" -> "600000000"
)

//Read from native Cassandra
val DFfromNativeCassandra = sqlContext
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(nativeCassandra)
  .load
  
//Write to CosmosCassandra
DFfromNativeCassandra
  .write
  .format("org.apache.spark.sql.cassandra")
  .options(cosmosCassandra)
  .save
```

> [!NOTE]
> Les configurations `spark.cassandra.output.concurrent.writes` et `connections_per_executor_max` sont importantes pour éviter la [limitation du débit](/samples/azure-samples/azure-cosmos-cassandra-java-retry-sample/azure-cosmos-db-cassandra-java-retry-sample/) qui se produit lorsque les demandes de Cosmos DB dépassent le débit approvisionné ([unités de requête](./request-units.md)). Vous devrez peut-être ajuster ces paramètres en fonction du nombre d’exécuteurs du cluster Spark, voire de la taille (et par conséquent du coût RU) de chaque enregistrement écrit dans les tables cibles.

## <a name="next-steps"></a>Étapes suivantes

* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md) 
* [Meilleures pratiques en matière de clés de partition](partitioning-overview.md#choose-partitionkey)
* Articles [Estimer le nombre d'unités de requête/seconde à l'aide du planificateur de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md)
* [Mise à l’échelle élastique dans l'API Cassandra Azure Cosmos DB](manage-scale-cassandra.md)