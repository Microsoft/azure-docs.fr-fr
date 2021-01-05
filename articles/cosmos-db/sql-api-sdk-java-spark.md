---
title: Notes de publication et ressources de Cosmos DB Apache Spark Connector pour l’API SQL
description: Découvrez Azure Cosmos DB Apache Spark Connector pour l’API SQL, notamment les dates de mise en production, les dates de mise hors service et les modifications apportées entre chaque version du Kit de développement logiciel (SDK) Java asynchrone SQL Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 7ff75bf9d45803dd96927bcf7c70e7c7912db979
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097156"
---
# <a name="azure-cosmos-db-apache-spark-connector-for-core-sql-api-release-notes-and-resources"></a>Azure Cosmos DB Apache Spark Connector pour l’API Core (SQL) : Notes de publication et ressources
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET v3](sql-api-sdk-dotnet-standard.md)
> * [SDK .NET v2](sql-api-sdk-dotnet.md)
> * [SDK .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK .NET Change Feed v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.JS](sql-api-sdk-node.md)
> * [SDK Java v4](sql-api-sdk-java-v4.md)
> * [SDK Java Async v2](sql-api-sdk-async-java.md)
> * [SDK Java Sync v2](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Spark Connector](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [API REST Resource Provider](/rest/api/cosmos-db-resource-provider/)
> * [SQL](./sql-query-getting-started.md)
> * [Exécuteur en bloc – .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Exécuteur en bloc – Java](sql-api-sdk-bulk-executor-java.md)

Vous pouvez accélérer l’analytique du Big Data à l’aide d’Azure Cosmos DB Apache Spark Connector pour Core (SQL). Spark Connector vous permet d’exécuter des tâches [Spark](https://spark.apache.org/) sur les données stockées dans Azure Cosmos DB. Le traitement par lots et des flux de données sont pris en charge.

Vous pouvez utiliser le connecteur avec [Azure Databricks](https://azure.microsoft.com/services/databricks) ou [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/), qui fournissent des clusters Spark gérés sur Azure. Le tableau suivant renseigne sur les versions prises en charge :

| Composant | Version |
|---------|-------|
| Apache Spark | 2.4. *x* , 2.3. *x* , 2.2. *x* et 2.1. *x* |
| Scala | 2.11 |
| Azure Databricks (version du runtime) | Ultérieure à 3.4 |

> [!WARNING]
> Ce connecteur prend en charge l’API de base (SQL) d’Azure Cosmos DB.
> Pour l’API Cosmos DB pour MongoDB, utilisez le [connecteur MongoDB pour Spark](https://docs.mongodb.com/spark-connector/master/).
> Pour l’API Cosmos DB pour Cassandra, utilisez le [connecteur Cassandra Spark](https://github.com/datastax/spark-cassandra-connector).
>

## <a name="resources"></a>Ressources

| Ressource | Lien |
|---|---|
| **Téléchargement du Kit de développement logiciel (SDK)** | [Télécharger le dernier .jar](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG), [Maven](https://search.maven.org/search?q=a:azure-cosmosdb-spark_2.4.0_2.11) |
|**Documentation de l’API** | [Référence Spark Connector]() |
|**Contribution au Kit de développement logiciel (SDK)** | [Azure Cosmos DB Connector pour Apache Spark sur GitHub](https://github.com/Azure/azure-cosmosdb-spark) | 
|**Prise en main** | [Accélérer l’analytique en temps réel des Big Data au moyen du connecteur Apache Spark-Azure Cosmos DB](./spark-connector.md#bk_working_with_connector) <br> [Utiliser Apache Spark Structured Streaming avec Apache Kafka et Azure Cosmos DB](../hdinsight/apache-kafka-spark-structured-streaming-cosmosdb.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json) | 

## <a name="release-history"></a>Historique des mises en production

### <a name="330"></a>3.3.0
#### <a name="new-features"></a>Nouvelles fonctionnalités
- Ajoute une nouvelle option de configuration, `changefeedstartfromdatetime`, qui peut être utilisée pour spécifier l’heure de début du flux de modification. Pour plus d’informations, consultez [Options de configuration](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuration-references).

### <a name="320"></a>3.2.0
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
- Corrige une régression qui entraînait une consommation de mémoire excessive sur les exécuteurs pour les grands jeux de résultats (par exemple, avec des millions de lignes), aboutissant à l’erreur `java.lang.OutOfMemoryError: GC overhead limit exceeded`.

### <a name="311"></a>3.1.1
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige un cas marginal où l’`ID` d’un point de contrôle de diffusion en continu contient la barre verticale (|) avec la configuration `ChangeFeedMaxPagesPerBatch`.

### <a name="310"></a>3.1.0
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajoute la prise en charge des mises à jour en bloc lors de l’utilisation de clés de partition imbriquées.
* Ajoute la prise en charge des types de données Decimal et Float lors des opérations d’écriture dans Azure Cosmos DB.
* Ajoute la prise en charge des types Timestamp lors de l’utilisation de la valeur Long (époque Unix).

### <a name="308"></a>3.0.8
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige l’exception Typecast qui se produit lorsque la configuration `WriteThroughputBudget` est utilisée.

### <a name="307"></a>3.0.7
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Ajoute des informations d’erreur pour les échecs en bloc à l’exception et au journal.

### <a name="306"></a>3.0.6
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige les problèmes de point de contrôle de diffusion en continu.

### <a name="305"></a>3.0.5
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Pour réduire le bruit, corrige le niveau de journalisation d’un message laissé involontairement avec le niveau ERROR.

### <a name="304"></a>3.0.4
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige un bogue dans la diffusion en continu structurée pendant les fractionnements de partition. Le bogue peut entraîner l’absence de certains enregistrements de flux de modification ou générer des exceptions Null pour les écritures de point de contrôle.

### <a name="303"></a>3.0.3
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige un bogue qui fait qu’un schéma personnalisé fourni pour readStream est ignoré.

### <a name="302"></a>3.0.2
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige la régression (le fichier JAR sans ombrage inclut toutes les dépendances ombrées) qui augmente le temps de génération de 50 %.

### <a name="301"></a>3.0.1
#### <a name="key-bug-fixes"></a>Principaux bogues résolus
* Corrige un problème de dépendance entraînant un échec Direct Transport over TCP avec RequestTimeoutException.

### <a name="300"></a>3.0.0
#### <a name="new-features"></a>Nouvelles fonctionnalités
* Améliore la gestion des connexions et le regroupement de connexions pour réduire le nombre d’appels de métadonnées.

## <a name="faq"></a>Questions fréquentes (FAQ)
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Découvrez plus en détail [Apache Spark](https://spark.apache.org/).
