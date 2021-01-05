---
title: Utilisation de l’API Cassandra Azure Cosmos DB à partir de Spark
description: Cet article est la page principale de l’intégration de l’API Cassandra Cosmos DB à partir de Spark.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/01/2019
ms.openlocfilehash: d25e168e342e22af9dc41d31dd7e18530aaa22b8
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93090509"
---
# <a name="connect-to-azure-cosmos-db-cassandra-api-from-spark"></a>Connecter l’API Cassandra Azure Cosmos DB à partir de Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article fait partie d’une série d’articles sur l’intégration de l’API Cassandra Azure Cosmos DB à partir de Spark. Ces articles abordent la connectivité, les opérations du langage de définition de données (DDL), les opérations du langage de manipulation de données (DML) et l’intégration avancée de l’API Cassandra Azure Cosmos DB à partir de Spark. 

## <a name="prerequisites"></a>Prérequis
* [Provisionnez un compte d’API Cassandra Azure Cosmos DB.](create-cassandra-dotnet.md#create-a-database-account)

* Provisionner votre choix d’environnement Spark [[Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal) | [Azure HDInsight-Spark](../hdinsight/spark/apache-spark-jupyter-spark-sql.md) | Autres].

## <a name="dependencies-for-connectivity"></a>Dépendances pour la connectivité
* **Connecteur Spark pour Cassandra :** le connecteur Spark est utilisé pour se connecter à l’API Cassandra Azure Cosmos DB.  Identifiez et utilisez la version du connecteur située dans le [dépôt central Maven]( https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector) qui est compatible avec les versions Spark et Scala de votre environnement Spark.

* **Bibliothèque d’assistance Azure Cosmos DB pour l’API Cassandra :** en plus du connecteur Spark, vous avez besoin d’une autre bibliothèque appelée [azure-cosmos-cassandra-spark-helper]( https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.0.0/jar) à partir d’Azure Cosmos DB. Cette bibliothèque contient des classes personnalisées de fabrique de connexion et de stratégie de nouvelle tentative.

  La stratégie de nouvelles tentatives dans Azure Cosmos DB est configurée pour gérer les exceptions de code d’état HTTP 429 (« Requêtes trop nombreuses »). L’API Cassandra Azure Cosmos DB traduit ces exceptions en erreurs surchargées sur le protocole natif Cassandra, et vous pouvez réessayer avec des interruptions. Comme Azure Cosmos DB utilise le modèle de débit provisionné, des exceptions de limitation du taux de requêtes se produisent quand les taux d’entrée/de sortie augmentent. La stratégie de nouvelles tentatives protège vos travaux Spark contre les pics de données qui dépassent momentanément le débit alloué pour votre conteneur.

  > [!NOTE] 
  > La stratégie de nouvelles tentatives peut protéger vos travaux Spark contre les pics momentanés uniquement. Si vous n’avez pas configuré suffisamment d’unités de requête nécessaires pour exécuter votre charge de travail, la stratégie de nouvelles tentatives n’est pas applicable et la classe de stratégie de nouvelles tentatives lève à nouveau l’exception.

* **Informations de connexion du compte Azure Cosmos DB :** Nom du compte, point de terminaison du compte et clé de votre API Cassandra d’Azure.
    
## <a name="spark-connector-throughput-configuration-parameters"></a>Paramètres de configuration du débit du connecteur Spark

Le tableau suivant répertorie les paramètres de configuration du débit spécifique à l’API Cassandra Azure Cosmos DB fournis par le connecteur. Pour obtenir une liste détaillée de tous les paramètres de configuration, consultez la page [Informations de référence sur la configuration](https://github.com/datastax/spark-cassandra-connector/blob/master/doc/reference.md) du dépôt GitHub du connecteur Cassandra Spark.

| **Nom de la propriété** | **Valeur par défaut** | **Description** |
|---------|---------|---------|
| spark.cassandra.output.batch.size.rows |  1 |Nombre de lignes par lot. Définissez ce paramètre sur 1. Ce paramètre est utilisé pour obtenir un débit plus élevé pour les charges de travail importantes. |
| spark.cassandra.connection.connections_per_executor_max  | None | Nombre maximal de connexions par nœud et par exécuteur. 10*n équivaut à 10 connexions par nœud dans un cluster Cassandra de n nœuds. Par conséquent, si vous avez besoin de 5 connexions par nœud et par exécuteur pour un cluster Cassandra à 5 nœuds, vous devez définir cette configuration sur 25. Modifiez cette valeur en fonction du degré de parallélisme ou du nombre d’exécuteurs pour lesquels vos travaux Spark sont configurés.   |
| spark.cassandra.output.concurrent.writes  |  100 | Définit le nombre d’écritures parallèles qui peuvent se produire par exécuteur. Étant donné que vous avez défini « batch.size.rows » sur 1, veillez à augmenter cette valeur en conséquence. Modifiez cette valeur selon le degré de parallélisme ou le débit que vous souhaitez obtenir pour votre charge de travail. |
| spark.cassandra.concurrent.reads |  512 | Définit le nombre de lectures parallèles qui peuvent se produire par exécuteur. Modifiez cette valeur selon le degré de parallélisme ou le débit que vous souhaitez obtenir pour votre charge de travail.  |
| spark.cassandra.output.throughput_mb_per_sec  | None | Définit le débit d’écriture total par exécuteur. Ce paramètre peut être utilisé comme limite supérieure pour le débit de vos travaux Spark et le baser sur le débit provisionné de votre conteneur Cosmos.   |
| spark.cassandra.input.reads_per_sec| None   | Définit le débit de lecture total par exécuteur. Ce paramètre peut être utilisé comme limite supérieure pour le débit de vos travaux Spark et le baser sur le débit provisionné de votre conteneur Cosmos.  |
| spark.cassandra.output.batch.grouping.buffer.size |  1 000  | Définit le nombre de lots par tâche Spark qui peuvent être stockés en mémoire avant d’être envoyés à l’API Cassandra. |
| spark.cassandra.connection.keep_alive_ms | 60000 | Définit la période pendant laquelle les connexions inutilisées sont disponibles. | 

Ajustez le débit et le degré de parallélisme de ces paramètres en fonction de la charge de travail que vous prévoyez pour vos travaux Spark et du débit que vous avez provisionné pour votre compte Cosmos DB.

## <a name="connecting-to-azure-cosmos-db-cassandra-api-from-spark"></a>Connexion à l’API Cassandra Azure Cosmos DB à partir de Spark

### <a name="cqlsh"></a>cqlsh
Les commandes suivantes décrivent en détail comment vous connecter à l’API Cassandra Azure Cosmos DB à partir de cqlsh.  Cela est utile pour la validation quand vous exécutez les exemples dans Spark.<br>
**À partir d’Unix/Linux/Mac :**

```bash
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false
cqlsh.py YOUR-COSMOSDB-ACCOUNT-NAME.cassandra.cosmosdb.azure.com 10350 -u YOUR-COSMOSDB-ACCOUNT-NAME -p YOUR-COSMOSDB-ACCOUNT-KEY --ssl
```

### <a name="1--azure-databricks"></a>1.  Azure Databricks
L’article ci-dessous aborde le provisionnement de clusters Azure Databricks, la configuration de clusters pour la connexion à l’API Cassandra Azure Cosmos DB et plusieurs exemples de bloc-notes qui traitent des opérations DDL et des opérations DML, entre autres.<BR>
[Utiliser l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks](cassandra-spark-databricks.md)<BR>
  
### <a name="2--azure-hdinsight-spark"></a>2.  Azure HDInsight-Spark
L’article ci-dessous aborde le service HDinsight-Spark, le provisionnement, la configuration de clusters pour la connexion à l’API Cassandra Azure Cosmos DB et plusieurs exemples de bloc-notes qui traitent des opérations DDL et des opérations DML, entre autres.<BR>
[Utiliser l’API Cassandra Azure Cosmos DB à partir d’Azure HDInsight-Spark](cassandra-spark-hdinsight.md)
 
### <a name="3--spark-environment-in-general"></a>3.  Généralités sur l’environnement Spark
Alors que les sections ci-dessus étaient spécifiques aux services PaaS basés sur Azure Spark, cette section traite de n’importe quel environnement Spark général.  Les dépendances du connecteur, les importations et la configuration de session Spark sont décrites en détail ci-dessous. La section « Étapes suivantes » présente des exemples de code pour les opérations DDL et les opérations DML, entre autres.  

#### <a name="connector-dependencies"></a>Dépendances du connecteur :

1. Ajouter les coordonnées Maven pour obtenir le [connecteur Cassandra pour Spark](cassandra-spark-generic.md#dependencies-for-connectivity)
2. Ajouter les coordonnées Maven pour la [bibliothèque d’assistance Azure Cosmos DB](cassandra-spark-generic.md#dependencies-for-connectivity) pour l’API Cassandra

#### <a name="imports"></a>Importations :

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra
```

#### <a name="spark-session-configuration"></a>Configuration de session Spark :

```scala
//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

//Throughput-related. You can adjust the values as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants illustrent l’intégration Spark à l’API Cassandra Azure Cosmos DB. 
 
* [Opérations DDL](cassandra-spark-ddl-ops.md)
* [Opérations de création et d’insertion](cassandra-spark-create-ops.md)
* [Opérations de lecture](cassandra-spark-read-ops.md)
* [Opérations d’upsert](cassandra-spark-upsert-ops.md)
* [Opérations de suppression](cassandra-spark-delete-ops.md)
* [Opérations d’agrégation](cassandra-spark-aggregation-ops.md)
* [Opérations de copie de table](cassandra-spark-table-copy-ops.md)