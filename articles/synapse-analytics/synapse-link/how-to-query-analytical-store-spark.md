---
title: Interagir avec Azure Cosmos DB à l'aide d'Apache Spark dans Azure Synapse Link (préversion)
description: Comment interagir avec Azure Cosmos DB à l'aide d'Apache Spark dans Azure Synapse Link
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 09/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 28af603c0969419cd2e7b8683373faf3838e2242
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458937"
---
# <a name="interact-with-azure-cosmos-db-using-apache-spark-in-azure-synapse-link"></a>Interagir avec Azure Cosmos DB à l’aide d’Apache Spark dans Azure Synapse Link

Dans cet article, vous allez apprendre à interagir avec Azure Cosmos DB à l'aide de Synapse Apache Spark. Avec sa prise en charge complète de Scala, Python, SparkSQL et C#, Synapse Apache Spark est au cœur des scénarios d'analyse, d'Engineering données, de science des données et d'exploration des données d'[Azure Synapse Link pour Azure Cosmos DB](../../cosmos-db/synapse-link.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Les fonctionnalités suivantes sont prises en charge lors de l'interaction avec Azure Cosmos DB :
* Synapse Apache Spark vous permet d'analyser les données des conteneurs Azure Cosmos DB activés avec Azure Synapse Link en quasi-temps réel sans affecter les performances de vos charges de travail transactionnelles. Les deux options suivantes sont disponibles pour interroger le [magasin analytique](../../cosmos-db/analytical-store-introduction.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) Azure Cosmos DB à partir de Spark :
    + Charger dans un DataFrame Spark
    + Créer une table Spark
* Synapse Apache Spark vous permet également d'ingérer des données dans Azure Cosmos DB. Il convient de noter que les données sont toujours ingérées dans les conteneurs Azure Cosmos DB par le biais du magasin transactionnel. Lorsque Synapse Link est activé, les nouvelles insertions, mises à jour et suppressions sont automatiquement synchronisées avec le magasin analytique.
* Synapse Apache Spark prend également en charge le flux structuré Spark avec Azure Cosmos DB comme source et comme récepteur. 

Les sections suivantes présentent la syntaxe des fonctionnalités ci-dessus. Les mouvements au sein de l'espace de travail Azure Synapse Analytics sont conçus afin de fournir une expérience prête à l'emploi facile à prendre en main. Les mouvements sont visibles lorsque vous cliquez avec le bouton droit sur un conteneur Azure Cosmos DB dans l'onglet **Données** de l'espace de travail Synapse. Avec les mouvements, vous pouvez rapidement générer du code et l’adapter à vos besoins. Les mouvements sont également bien adaptés pour découvrir des données en un seul clic.

## <a name="query-azure-cosmos-db-analytical-store"></a>Interroger le magasin analytique Azure Cosmos DB

Avant d'examiner les deux options disponibles pour interroger le magasin analytique Azure Cosmos DB, à savoir le chargement dans un DataFrame Spark et la création d'une table Spark, il convient d'explorer les différences d'expérience afin de pouvoir choisir l'option adéquate.

La différence d'expérience réside dans le fait de savoir si les modifications de données sous-jacentes du conteneur Azure Cosmos DB doivent être automatiquement reflétées dans l'analyse effectuée dans Spark. Lorsqu'un DataFrame Spark est enregistré ou qu'une table Spark est créée dans le magasin analytique d'un conteneur, les métadonnées entourant l'instantané actuel des données dans le magasin analytique sont extraites vers Spark pour une transmission efficace de l'analyse ultérieure. Il convient de noter que dans la mesure où Spark suit une stratégie d'évaluation différée, si aucune action n'est appelée sur le DataFrame Spark ou si aucune requête SparkSQL n'est exécutée sur la table Spark, les données réelles ne sont pas extraites du magasin analytique du conteneur sous-jacent.

Dans le cas du **chargement dans un DataFrame Spark**, les métadonnées extraites sont mises en cache pendant toute la durée de vie de la session Spark, et les actions ultérieures appelées sur le DataFrame sont évaluées par rapport à l'instantané du magasin analytique au moment de la création du DataFrame.

En revanche, dans le cas de la **création d'une table Spark**, les métadonnées de l'état du magasin analytique ne sont pas mises en cache dans Spark et sont rechargées chaque fois qu'une requête SparkSQL est exécutée sur la table Spark.

Par conséquent, le choix entre le chargement dans un DataFrame Spark et la création d'une table Spark dépend du type d'évaluation que vous souhaitez effectuer pour votre analyse Spark : par rapport à un instantané fixe du magasin analytique ou par rapport au dernier instantané du magasin analytique.

> [!NOTE]
> Pour interroger l'API Azure Cosmos DB des comptes Mongo DB, découvrez-en plus sur la [représentation du schéma de fidélité optimale](../../cosmos-db/analytical-store-introduction.md#analytical-schema) dans le magasin analytique et sur les noms de propriété étendue à utiliser.

### <a name="load-to-spark-dataframe"></a>Charger dans un DataFrame Spark

Dans cet exemple, vous allez créer un DataFrame Spark qui pointe vers le magasin analytique Azure Cosmos DB. Vous pourrez ensuite effectuer une analyse supplémentaire en appelant des actions Spark sur le DataFrame. Cette opération n’a aucun impact sur le magasin transactionnel.

La syntaxe **Python** est la suivante :
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

df = spark.read.format("cosmos.olap")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .load()
```

La syntaxe **Scala** équivalente est la suivante :
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>").
    load()
```

### <a name="create-spark-table"></a>Créer une table Spark

Dans cet exemple, vous allez créer une table Spark qui pointe vers le magasin analytique Azure Cosmos DB. Vous pourrez ensuite effectuer une analyse supplémentaire en appelant des requêtes SparkSQL sur la table. Cette opération n'a aucun impact sur le magasin transactionnel et n'entraîne aucun déplacement de données. Si vous décidez de supprimer cette table Spark, le conteneur Azure Cosmos DB sous-jacent et le magasin analytique correspondant ne seront pas affectés. 

Ce scénario permet de réutiliser des tables Spark à l'aide d'outils tiers et de fournir une accessibilité aux données sous-jacentes au moment de l'exécution.

La syntaxe permettant de créer une table Spark est la suivante :
```sql
%%sql
-- To select a preferred list of regions in a multi-region Azure Cosmos DB account, add spark.cosmos.preferredRegions '<Region1>,<Region2>' in the config options

create table call_center using cosmos.olap options (
    spark.synapse.linkedService '<enter linked service name>',
    spark.cosmos.container '<enter container name>'
)
```

> [!NOTE]
> Pour les scénarios dans lesquels le schéma du conteneur Azure Cosmos DB sous-jacent change au fil du temps, et si vous souhaitez que le schéma mis à jour se reflète automatiquement dans les requêtes adressées à la table Spark, vous pouvez définir l'option `spark.cosmos.autoSchemaMerge` sur `true` dans les options de la table Spark.


## <a name="write-spark-dataframe-to-azure-cosmos-db-container"></a>Écrire un DataFrame Spark dans un conteneur Azure Cosmos DB

Dans cet exemple, vous allez écrire un DataFrame Spark dans un conteneur Azure Cosmos DB. Cette opération aura un impact sur les performances des charges de travail transactionnelles et consommera une partie des unités de requête approvisionnées sur le conteneur Azure Cosmos DB ou la base de données partagée.

La syntaxe **Python** est la suivante :
```python
# Write a Spark DataFrame into an Azure Cosmos DB container
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

YOURDATAFRAME.write.format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .mode('append')\
    .save()
```

La syntaxe **Scala** équivalente est la suivante :
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

import org.apache.spark.sql.SaveMode

df.write.format("cosmos.oltp").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>"). 
    option("spark.cosmos.write.upsertEnabled", "true").
    mode(SaveMode.Overwrite).
    save()
```

## <a name="load-streaming-dataframe-from-container"></a>Charger la diffusion en continu de tramedonnées à partir d’un conteneur
Dans ce mouvement, vous utiliserez la fonctionnalité de diffusion en continu Spark pour charger des données à partir d’un conteneur dans un tramedonnées. Les données seront stockées dans le compte du lac de données principal (et le système de fichiers) que vous avez connecté à l’espace de travail. 
> [!NOTE]
> Si vous souhaitez référencer des bibliothèques externes dans Synapse Apache Spark, cliquez [ici](#external-library-management) pour en savoir plus. Par exemple, si vous souhaitez ingérer un DataFrame Spark dans un conteneur de l'API Cosmos DB pour Mongo DB, vous pouvez utiliser le connecteur Mongo DB pour Spark disponible [ici](https://docs.mongodb.com/spark-connector/master/).

## <a name="load-streaming-dataframe-from-azure-cosmos-db-container"></a>Charger un DataFrame de diffusion en continu à partir d'un conteneur Azure Cosmos DB
Dans cet exemple, vous allez utiliser la fonctionnalité de flux structuré de Spark pour charger les données d'un conteneur Azure Cosmos DB dans un DataFrame de diffusion en continu Spark à l'aide de la fonctionnalité de flux de modification d'Azure Cosmos DB. Les données de point de contrôle utilisées par Spark seront stockées sur le compte (et dans le système de fichiers) du lac de données principal que vous avez connecté à l'espace de travail.

Si le dossier */localReadCheckpointFolder* n'existe pas (dans l'exemple ci-dessous), il est automatiquement créé. Cette opération aura un impact sur les performances des charges de travail transactionnelles et consommera une partie des unités de requête approvisionnées sur le conteneur Azure Cosmos DB ou la base de données partagée.

La syntaxe **Python** est la suivante :
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

dfStream = spark.readStream\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .option("spark.cosmos.changeFeed.readEnabled", "true")\
    .option("spark.cosmos.changeFeed.startFromTheBeginning", "true")\
    .option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder")\
    .option("spark.cosmos.changeFeed.queryName", "streamQuery")\
    .load()
```

La syntaxe **Scala** équivalente est la suivante :
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val dfStream = spark.readStream.
    format("cosmos.oltp").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>").
    option("spark.cosmos.changeFeed.readEnabled", "true").
    option("spark.cosmos.changeFeed.startFromTheBeginning", "true").
    option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder").
    option("spark.cosmos.changeFeed.queryName", "streamQuery").
    load()
```

## <a name="write-streaming-dataframe-to-azure-cosmos-db-container"></a>Écrire un DataFrame de diffusion en continu à partir d'un conteneur Azure Cosmos DB
Dans cet exemple, vous allez écrire un DataFrame de diffusion en continu dans un conteneur Azure Cosmos DB. Cette opération aura un impact sur les performances des charges de travail transactionnelles et consommera une partie des unités de requête approvisionnées sur le conteneur Azure Cosmos DB ou la base de données partagée. Si le dossier */localWriteCheckpointFolder* n'existe pas (dans l'exemple ci-dessous), il est automatiquement créé. 

La syntaxe **Python** est la suivante :
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

streamQuery = dfStream\
        .writeStream\
        .format("cosmos.oltp")\
        .outputMode("append")\
        .option("checkpointLocation", "/localWriteCheckpointFolder")\
        .option("spark.synapse.linkedService", "<enter linked service name>")\
        .option("spark.cosmos.container", "<enter container name>")\
        .option("spark.cosmos.connection.mode", "gateway")\
        .start()

streamQuery.awaitTermination()
```

La syntaxe **Scala** équivalente est la suivante :
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val query = dfStream.
            writeStream.
            format("cosmos.oltp").
            outputMode("append").
            option("checkpointLocation", "/localWriteCheckpointFolder").
            option("spark.synapse.linkedService", "<enter linked service name>").
            option("spark.cosmos.container", "<enter container name>").
            option("spark.cosmos.connection.mode", "gateway").
            start()

query.awaitTermination()
```

## <a name="external-library-management"></a>Gestion de la bibliothèque externe

Dans cet exemple, vous allez apprendre à référencer des bibliothèques externes à partir de fichiers JAR lors de l'utilisation de notebooks Spark dans des espaces de travail Synapse Apache Spark. Vous pouvez placer les fichiers JAR dans un conteneur du compte du lac de données principal que vous avez connecté à l'espace de travail, puis ajouter l'instruction `%configure` suivante dans votre notebook Spark :

```cmd
%%configure -f
{
    "jars": [
        "abfss://<storage container name>@<data lake account name>.dfs.core.windows.net/<path to jar>"
    ]
}
```
Si vous souhaitez envoyer des définitions de tâches Spark distantes à un pool Apache Spark serverless, vous pouvez apprendre à référencer des bibliothèques externes en suivant ce [tutoriel](../spark/apache-spark-job-definitions.md).

## <a name="next-steps"></a>Étapes suivantes

* [Exemples permettant de prendre en main Azure Synapse Link sur GitHub](https://aka.ms/cosmosdb-synapselink-samples).
* [Découvrez les fonctionnalités prises en charge dans Azure Synapse Link pour Azure Cosmos DB](./concept-synapse-link-cosmos-db-support.md)
* [Se connecter à Synapse Link pour Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md)
