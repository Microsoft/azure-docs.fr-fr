---
title: Copier Synapse Link pour les données Azure Cosmos DB dans un pool SQL dédié avec Apache Spark
description: Charger les données dans un dataframe Spark, organiser les données et les charger dans une table de pools SQL dédiés
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 08/10/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 13891f9614e658be39adbb69fed1503a0c66d5e4
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93309212"
---
# <a name="copy-data-from-azure-cosmos-db-into-a-dedicated-sql-pool-using-apache-spark"></a>Copier des données depuis Azure Cosmos DB vers un pool SQL dédié avec Apache Spark

Azure Synapse Link pour Azure Cosmos DB permet aux utilisateurs d’exécuter une analytique en quasi-temps réel sur les données opérationnelles dans Azure Cosmos DB. Toutefois, il peut arriver que certaines données doivent être agrégées et enrichies pour traiter les utilisateurs de l’entrepôt de données. L’organisation et l’exportation de données Synapse Link peuvent s’effectuer à l’aide de quelques cellules seulement d’un notebook.

## <a name="prerequisites"></a>Prérequis
* [Provisionner un espace de travail Synapse](../quickstart-create-workspace.md) avec :
    * [Pool Apache Spark serverless](../quickstart-create-apache-spark-pool-studio.md)
    * [Pool SQL dédié](../quickstart-create-sql-pool-studio.md)
* [Provisionner un compte Cosmos DB à l’aide d’un conteneur HTAP avec des données](../../cosmos-db/configure-synapse-link.md)
* [Connecter le conteneur HTAP d’Azure Cosmos DB à l’espace de travail](./how-to-connect-synapse-link-cosmos-db.md)
* [Disposer de la configuration appropriée pour importer des données dans un pool SQL dédié à partir de Spark](../spark/synapse-spark-sql-pool-import-export.md)

## <a name="steps"></a>Étapes
Dans ce tutoriel, vous allez vous connecter au magasin analytique pour qu’il n’y ait aucun impact sur le magasin transactionnel (aucune unité de requête ne sera consommée). Nous allons suivre les étapes suivantes :
1. Lire le conteneur HTAP Cosmos DB dans un dataframe Spark
2. Agréger les résultats dans un nouveau dataframe
3. Ingérer les données dans un pool SQL dédié

[![Étapes Spark vers SQL 1](../media/synapse-link-spark-to-sql/synapse-spark-to-sql.png)](../media/synapse-link-spark-to-sql/synapse-spark-to-sql.png#lightbox)

## <a name="data"></a>Données
Dans cet exemple, nous utilisons un conteneur HTAP appelé **RetailSales**. Il fait partie d’un service lié appelé **ConnectedData** et présente le schéma suivant :
* _rid: string (nullable = true)
* _ts: long (nullable = true)
* logQuantity: double (nullable = true)
* productCode: string (nullable = true)
* quantity: long (nullable = true)
* price: long (nullable = true)
* id: string (nullable = true)
* advertising: long (nullable = true)
* storeId: long (nullable = true)
* weekStarting: long (nullable = true)
* _etag: string (nullable = true)

Nous allons regrouper les ventes ( *quantity* , *revenue* (prix x quantité) par *productCode* et *weekStarting* à des fins de création de rapports. Nous terminerons par l’exportation de ces données dans une table de pools SQL dédiés appelée **dbo.productsales**.

## <a name="configure-a-spark-notebook"></a>Configurer un notebook Spark
Créez un notebook Spark avec Scala, Spark (Scala) étant le langage principal. Nous utilisons le paramètre par défaut du notebook pour la session.

## <a name="read-the-data-in-spark"></a>Lire les données dans Spark
Lisez le conteneur HTAP Cosmos DB avec Spark dans un dataframe au niveau de la première cellule.

```java
val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "ConnectedData").
    option("spark.cosmos.container", "RetailSales").
    load()
```

## <a name="aggregate-the-results-in-a-new-dataframe"></a>Agréger les résultats dans un nouveau dataframe

Dans la deuxième cellule, nous exécutons la transformation et les agrégats nécessaires pour le nouveau dataframe avant de le charger dans une base de données de pools SQL dédiés.

```java
// Select relevant columns and create revenue
val df_olap_step1 = df_olap.select("productCode","weekStarting","quantity","price").withColumn("revenue",col("quantity")*col("price"))
//Aggregate revenue, quantity sold and avg. price by week and product ID
val df_olap_aggr = df_olap_step1.groupBy("productCode","weekStarting").agg(sum("quantity") as "Sum_quantity",sum("revenue") as "Sum_revenue").
    withColumn("AvgPrice",col("Sum_revenue")/col("Sum_quantity"))
```

## <a name="load-the-results-into-a-dedicated-sql-pool"></a>Charger les résultats dans un pool SQL dédié

Dans la troisième cellule, nous chargeons les données dans un pool SQL dédié. Cette opération crée automatiquement une table externe temporaire, une source de données externe et un format de fichier externe qui seront supprimés une fois le travail terminé.

```java
df_olap_aggr.write.sqlanalytics("userpool.dbo.productsales", Constants.INTERNAL)
```

## <a name="query-the-results-with-sql"></a>Interroger les résultats avec SQL

Vous pouvez interroger le résultat au moyen d’une requête SQL simple, telle que le script SQL suivant :
```sql
SELECT  [productCode]
,[weekStarting]
,[Sum_quantity]
,[Sum_revenue]
,[AvgPrice]
 FROM [dbo].[productsales]
```

Votre requête doit présenter les résultats suivants en mode graphique : [![Étapes Spark vers SQL 2](../media/synapse-link-spark-to-sql/sql-script-spark-sql.png)](../media/synapse-link-spark-to-sql/sql-script-spark-sql.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes
* [Interrogation du magasin analytique Azure Cosmos DB avec Apache Spark](./how-to-query-analytical-store-spark.md)