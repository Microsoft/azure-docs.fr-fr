---
title: Vue d’ensemble de l’utilisation de Linux Foundation Delta Lake dans Apache Spark pour Azure Synapse Analytics
description: Apprenez à utiliser Delta Lake dans Apache Spark pour Azure Synapse Analytics afin de créer et d’utiliser des tables avec des propriétés ACID.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.reviewer: euang
ms.topic: overview
ms.subservice: spark
ms.date: 07/28/2020
ms.author: euang
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: 655daeb0149228d78d5288b0e5d0d705a5743d28
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89008639"
---
# <a name="linux-foundation-delta-lake-overview"></a>Vue d’ensemble de Linux Foundation Delta Lake

Pour plus de clarté, cet article a été adapté à partir de son équivalent d’origine [ici](https://docs.delta.io/latest/quick-start.html). Cet article vous permet de découvrir rapidement les principales fonctionnalités de [Delta Lake](https://delta.io). L’article fournit des extraits de code qui montrent comment lire et écrire dans des tables Delta Lake à partir de requêtes interactives, par lots et de streaming. Les extraits de code sont également disponibles dans un ensemble de notebooks [PySpark ici](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/PySpark/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20Python.ipynb), [Scala ici](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/Scala/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20Scala.ipynb) et [C# ici](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/Spark.NET%20C%23/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20CSharp.ipynb)

Nous allons aborder les sujets suivants :

* Créer une table
* Lire les données
* Mettre à jour les données d’une table
* Remplacer les données d’une table
* Mettre à jour sous conditions sans remplacement
* Lire des versions antérieures de données à l’aide de Voyage dans le temps
* Écrire un flux de données dans une table
* Lire un flux de modifications à partir d’une table
* Prise en charge SQL

## <a name="configuration"></a>Configuration

Veillez à modifier l’extrait suivant en fonction de votre environnement.

:::zone pivot = "programming-language-python"

```python
import random

session_id = random.randint(0,1000000)
delta_table_path = "/delta/delta-table-{0}".format(session_id)

delta_table_path
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var sessionId = (new Random()).Next(10000000);
var deltaTablePath = $"/delta/delta-table-{sessionId}";

deltaTablePath
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val sessionId = scala.util.Random.nextInt(1000000)
val deltaTablePath = s"/delta/delta-table-$sessionId";
```

::: zone-end

Retourne comme résultat :

« /delta/delta-table-335323 »

## <a name="create-a-table"></a>Créer une table

Pour créer une table Delta Lake, écrivez un élément DataFrame à partir d’un DataFrame au format delta. Vous pouvez remplacer le format Parquet, CSV, JSON, et ainsi de suite, par delta.

Le code suivant vous montre comment créer une table Delta Lake à l’aide du schéma déduit à partir de votre DataFrame.

:::zone pivot = "programming-language-python"

```python
data = spark.range(0,5)
data.show()
data.write.format("delta").save(delta_table_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var data = spark.Range(0,5);
data.Show();
data.Write().Format("delta").Save(deltaTablePath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val data = spark.range(0, 5)
data.show
data.write.format("delta").save(deltaTablePath)
```

::: zone-end

Retourne comme résultat :

| id|
|---|
|  0|
|  1|
|  2|
|  3|
|  4|

## <a name="read-data"></a>Lire les données

Vous lisez les données de votre table Delta Lake en spécifiant le chemin des fichiers et le format delta.

:::zone pivot = "programming-language-python"

```python
df = spark.read.format("delta").load(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var df = spark.Read().Format("delta").Load(deltaTablePath);
df.Show()
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val df = spark.read.format("delta").load(deltaTablePath)
df.show()
```

::: zone-end

Retourne comme résultat :

| id|
|---|
|  1|
|  3|
|  4|
|  0|
|  2|

L’ordre des résultats est différent de celui indiqué ci-dessus, car aucune commande n’a été spécifiée explicitement avant la sortie des résultats.

## <a name="update-table-data"></a>Mettre à jour les données d’une table

Delta Lake prend en charge plusieurs opérations pour modifier les tables au moyen des API DataFrame standard, ce qui constitue une des améliorations majeures apportées par le format delta. L’exemple suivant exécute un traitement par lots pour remplacer les données de la table.

:::zone pivot = "programming-language-python"

```python
data = spark.range(5,10)
data.write.format("delta").mode("overwrite").save(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var data = spark.Range(5,10);
data.Write().Format("delta").Mode("overwrite").Save(deltaTablePath);
df.Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val data = spark.range(5, 10)
data.write.format("delta").mode("overwrite").save(deltaTablePath)
df.show()
```

::: zone-end

Retourne comme résultat :

| id|
|---|
|  7|
|  8|
|  5|
|  9|
|  6|

Ici, vous pouvez voir que les cinq enregistrements ont été mis à jour pour contenir de nouvelles valeurs.

## <a name="save-as-catalog-tables"></a>Enregistrer en tant que tables de catalogue

Delta Lake peut écrire dans des tables de catalogue gérées ou externes.

:::zone pivot = "programming-language-python"

```python
data.write.format("delta").saveAsTable("ManagedDeltaTable")
spark.sql("CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '{0}'".format(delta_table_path))
spark.sql("SHOW TABLES").show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
data.Write().Format("delta").SaveAsTable("ManagedDeltaTable");
spark.Sql($"CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '{deltaTablePath}'");
spark.Sql("SHOW TABLES").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
data.write.format("delta").saveAsTable("ManagedDeltaTable")
spark.sql(s"CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '$deltaTablePath'")
spark.sql("SHOW TABLES").show
```

::: zone-end

Retourne comme résultat :

|database|         tableName|isTemporary|
|--------|------------------|-----------|
| default|externaldeltatable|      false|
| default| manageddeltatable|      false|

À l’aide de ce code, vous avez créé une table dans le catalogue depuis un DataFrame existant ; elle est appelée « table gérée ». Ensuite, vous avez défini une nouvelle table externe dans le catalogue qui utilise un emplacement existant ; elle est appelée « table externe ». Dans la sortie, vous pouvez voir les deux tables répertoriées dans le catalogue, peu importe la façon dont elles ont été créées.

Vous pouvez maintenant examiner les propriétés étendues de ces deux tables

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE EXTENDED ManagedDeltaTable").show(truncate=False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql("DESCRIBE EXTENDED ManagedDeltaTable").Show(truncate: 0);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql("DESCRIBE EXTENDED ManagedDeltaTable").show(truncate=false)
```

::: zone-end

Retourne comme résultat :

|col_name                    |data_type                                                                                                    |comment|
|----------------------------|-------------------------------------------------------------------------------------------------------------|-------|
|id                          |bigint                                                                                                       |null   |
|                            |                                                                                                             |       |
|Informations détaillées sur la table  |                                                                                                             |       |
|Base de données                    |default                                                                                                      |       |
|Table de charge de travail                       |manageddeltatable                                                                                            |       |
|Propriétaire                       |trusted-service-user                                                                                         |       |
|Heure de création                |Sat Apr 25 00:35:34 UTC 2020                                                                                 |       |
|Dernier accès                 |Thu Jan 01 00:00:00 UTC 1970                                                                                 |       |
|Créé par                  |Spark 2.4.4.2.6.99.201-11401300                                                                              |       |
|Type                        |MANAGED                                                                                                      |       |
|Fournisseur                    |delta                                                                                                        |       |
|Propriétés de la table            |[transient_lastDdlTime=1587774934]                                                                           |       |
|Statistiques                  |2407 bytes                                                                                                   |       |
|Emplacement                    |abfss://data@<data lake>.dfs.core.windows.net/synapse/workspaces/<workspace name>/warehouse/manageddeltatable|       |
|Bibliothèque Serde               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                                                           |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                                                             |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                                                    |       |
|Propriétés de stockage          |[serialization.format=1]                                                                                     |       |

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE EXTENDED ExternalDeltaTable").show(truncate=False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql("DESCRIBE EXTENDED ExternalDeltaTable").Show(truncate: 0);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql("DESCRIBE EXTENDED ExternalDeltaTable").show(truncate=false)
```

::: zone-end

Retourne comme résultat :

|col_name                    |data_type                                                             |comment|
|----------------------------|----------------------------------------------------------------------|-------|
|id                          |bigint                                                                |null   |
|                            |                                                                      |       |
|Informations détaillées sur la table  |                                                                      |       |
|Base de données                    |default                                                               |       |
|Table de charge de travail                       |externaldeltatable                                                    |       |
|Propriétaire                       |trusted-service-user                                                  |       |
|Heure de création                |Sat Apr 25 00:35:38 UTC 2020                                          |       |
|Dernier accès                 |Thu Jan 01 00:00:00 UTC 1970                                          |       |
|Créé par                  |Spark 2.4.4.2.6.99.201-11401300                                       |       |
|Type                        |EXTERNAL                                                              |       |
|Fournisseur                    |DELTA                                                                 |       |
|Propriétés de la table            |[transient_lastDdlTime=1587774938]                                    |       |
|Emplacement                    |abfss://data@<data lake>.dfs.core.windows.net/delta/delta-table-587152|       |
|Bibliothèque Serde               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                    |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                      |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat             |       |
|Propriétés de stockage          |[serialization.format=1]                                              |       |

## <a name="conditional-update-without-overwrite"></a>Mettre à jour sous conditions sans remplacement

Delta Lake fournit des API programmatiques pour mettre à jour, supprimer et fusionner sous conditions (ce qui est communément appelé « une opération upsert ») des données dans des tables.

:::zone pivot = "programming-language-python"

```python
from delta.tables import *
from pyspark.sql.functions import *

delta_table = DeltaTable.forPath(spark, delta_table_path)

delta_table.update(
  condition = expr("id % 2 == 0"),
  set = { "id": expr("id + 100") })
delta_table.toDF().show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Delta;
using Microsoft.Spark.Extensions.Delta.Tables;
using Microsoft.Spark.Sql;
using static Microsoft.Spark.Sql.Functions;

var deltaTable = DeltaTable.ForPath(deltaTablePath);

deltaTable.Update(
  condition: Expr("id % 2 == 0"),
  set: new Dictionary<string, Column>(){{ "id", Expr("id + 100") }});
deltaTable.ToDF().Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
import io.delta.tables._
import org.apache.spark.sql.functions._

val deltaTable = DeltaTable.forPath(deltaTablePath)

// Update every even value by adding 100 to it
deltaTable.update(
  condition = expr("id % 2 == 0"),
  set = Map("id" -> expr("id + 100")))
deltaTable.toDF.show
```

::: zone-end

Retourne comme résultat :

| id|
|---|
|106|
|108|
|  5|
|  7|
|  9|

Ici, vous venez d’ajouter 100 à chaque ID pair.

:::zone pivot = "programming-language-python"

```python
delta_table.delete("id % 2 == 0")
delta_table.toDF().show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.Delete(condition: Expr("id % 2 == 0"));
deltaTable.ToDF().Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.delete(condition = expr("id % 2 == 0"))
deltaTable.toDF.show
```

::: zone-end

Retourne comme résultat :

| id|
|---|
|  5|
|  7|
|  9|

Remarquez que chaque ligne paire a été supprimée.

:::zone pivot = "programming-language-python"

```python
new_data = spark.range(0,20).alias("newData")

delta_table.alias("oldData")\
    .merge(new_data.alias("newData"), "oldData.id = newData.id")\
    .whenMatchedUpdate(set = { "id": lit("-1")})\
    .whenNotMatchedInsert(values = { "id": col("newData.id") })\
    .execute()

delta_table.toDF().show(100)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var newData = spark.Range(20).As("newData");

deltaTable
    .As("oldData")
    .Merge(newData, "oldData.id = newData.id")
    .WhenMatched()
        .Update(new Dictionary<string, Column>() {{"id", Lit("-1")}})
    .WhenNotMatched()
        .Insert(new Dictionary<string, Column>() {{"id", Col("newData.id")}})
    .Execute();

deltaTable.ToDF().Show(100);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val newData = spark.range(0, 20).toDF

deltaTable.as("oldData").
  merge(
    newData.as("newData"),
    "oldData.id = newData.id").
  whenMatched.
  update(Map("id" -> lit(-1))).
  whenNotMatched.
  insert(Map("id" -> col("newData.id"))).
  execute()

deltaTable.toDF.show()
```

::: zone-end

Retourne comme résultat :

| id|
|---|
| 18|
| 15|
| 19|
|  2|
|  1|
|  6|
|  8|
|  3|
| -1|
| 10|
| 13|
|  0|
| 16|
|  4|
| -1|
| 12|
| 11|
| 14|
| -1|
| 17|

Ici, vous avez une combinaison des données existantes. La valeur -1 a été affectée aux données existantes dans le chemin du code de mise à jour (WhenMatched). Les données qui ont été créées en haut de l’extrait de code, et qui ont été ajoutées via le chemin du code d’insertion (WhenNotMatched), ont également été ajoutées.

### <a name="history"></a>Historique

Delta Lake permet d’examiner l’historique d’une table. Autrement dit, les modifications apportées à la table Delta sous-jacente. La cellule ci-dessous montre avec quelle simplicité il est possible d’inspecter l’historique.

:::zone pivot = "programming-language-python"

```python
delta_table.history().show(20, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.History().Show(20, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.history.show(false)
```

::: zone-end

Retourne comme résultat :

|version|          timestamp|userId|userName|opération|                                                operationParameters| travail|notebook|clusterId|readVersion|isolationLevel|isBlindAppend|
|-------|-------------------|------|--------|---------|-------------------------------------------------------------------|----|--------|---------|-----------|--------------|-------------|
|      4|2020-04-25 00:36:27|  null|    null|    MERGE|                       [predicate -> (oldData.`ID` = newData.`ID`)]|null|    null|     null|          3|          null|        false|
|      3|2020-04-25 00:36:08|  null|    null|   Suppression|[predicate -> ["((`ID` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|null|    null|     null|          2|          null|        false|
|      2|2020-04-25 00:35:51|  null|    null|   UPDATE| [predicate -> ((ID#744L % cast(2 as bigint)) = cast(0 as bigint))]|null|    null|     null|          1|          null|        false|
|      1|2020-04-25 00:35:05|  null|    null|    WRITE|                             [mode -> Overwrite, partitionBy -> []]|null|    null|     null|          0|          null|        false|
|      0|2020-04-25 00:34:34|  null|    null|    WRITE|                         [mode -> ErrorIfExists, partitionBy -> []]|null|    null|     null|       null|          null|         true|

Ici, vous pouvez voir toutes les modifications apportées aux extraits de code ci-dessus.

## <a name="read-older-versions-of-data-using-time-travel"></a>Lire des versions antérieures de données à l’aide de Voyage dans le temps

Il est possible d’interroger les captures instantanées précédentes de votre table Delta Lake en utilisant une fonctionnalité appelée Voyage dans le temps. Si vous souhaitez accéder aux données que vous avez remplacées, vous pouvez interroger un instantané de la table préexistant au remplacement du premier jeu de données à l’aide de l’option versionAsOf.

Après avoir exécuté la cellule ci-dessous, vous devez voir le premier jeu de données existant avant que vous ne le remplaciez. Voyage dans le temps est une fonctionnalité extrêmement puissante, qui exploite les capacités du journal des transactions Delta Lake pour accéder aux données qui ne se trouvent plus dans la table. La suppression de l’option version 0 (ou la spécification de version 1) vous permet de voir à nouveau les données plus récentes. Pour plus d’informations, consultez [Interroger un instantané de table antérieur](https://docs.delta.io/latest/delta-batch.html#deltatimetravel).

:::zone pivot = "programming-language-python"

```python
df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var df = spark.Read().Format("delta").Option("versionAsOf", 0).Load(deltaTablePath);
df.Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val df = spark.read.format("delta").option("versionAsOf", 0).load(deltaTablePath)
df.show()
```

::: zone-end

Retourne comme résultat :

| id|
|---|
|  0|
|  1|
|  4|
|  3|
|  2|

Ici, vous pouvez constater que vous avez rétabli la version la plus ancienne des données.

## <a name="write-a-stream-of-data-to-a-table"></a>Écrire un flux de données dans une table

Vous pouvez également écrire dans une table Delta Lake à l’aide de Structured Streaming (Streaming structuré) de Spark. Le journal des transactions Delta Lake garantit un seul et unique traitement, même si d’autres flux ou requêtes par lot s’exécutent simultanément sur la table. Par défaut, les flux s’exécutent en mode ajout, ce qui adjoint de nouveaux enregistrements à la table.

Pour plus d’informations sur l’intégration de Delta Lake à Structured streaming, consultez [Lectures et écritures de streaming de table](https://docs.delta.io/latest/delta-streaming.html).

Dans les cellules ci-dessous, voici les actions que nous accomplissons :

* Cellule 30 Montrer les données ajoutées récemment
* Cellule 31 Inspecter l’historique
* Cellule 32 Arrêter le travail de streaming structuré
* Cellule 33 Inspecter l’historique <-- Vous remarquerez que les ajouts ont été arrêtés

Tout d’abord, vous allez configurer un travail de streaming Spark simple pour générer une séquence et faire en sorte que le travail s’écrive dans votre table Delta.

:::zone pivot = "programming-language-python"

```python
streaming_df = spark.readStream.format("rate").load()
stream = streaming_df\
    .selectExpr("value as id")\
    .writeStream\
    .format("delta")\
    .option("checkpointLocation", "/tmp/checkpoint-{0}".format(session_id))\
    .start(delta_table_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var streamingDf = spark.ReadStream().Format("rate").Load();
var stream = streamingDf.SelectExpr("value as id").WriteStream().Format("delta").Option("checkpointLocation", $"/tmp/checkpoint-{sessionId}").Start(deltaTablePath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val streamingDf = spark.readStream.format("rate").load()
val stream = streamingDf.select($"value" as "id").writeStream.format("delta").option("checkpointLocation", s"/tmp/checkpoint-$sessionId").start(deltaTablePath)
```

::: zone-end

## <a name="read-a-stream-of-changes-from-a-table"></a>Lire un flux de modifications à partir d’une table

Pendant que le flux s’écrit dans la table Delta Lake, vous pouvez aussi lire cette table en tant que source de streaming. Par exemple, vous pouvez démarrer une autre requête de streaming qui imprime toutes les modifications apportées à la table Delta Lake.

:::zone pivot = "programming-language-python"

```python
delta_table.toDF().sort(col("id").desc()).show(100)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.ToDF().Sort(Col("id").Desc()).Show(100);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.toDF.sort($"id".desc).show
```

::: zone-end

Retourne comme résultat :

| id|
|---|
| 19|
| 18|
| 17|
| 16|
| 15|
| 14|
| 13|
| 12|
| 11|
| 10|
|  8|
|  6|
|  4|
|  3|
|  2|
|  1|
|  0|
| -1|
| -1|
| -1|

:::zone pivot = "programming-language-python"

```python
delta_table.history().drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").show(20, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.History().Drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").Show(20, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.history.show
```

::: zone-end

Retourne comme résultat :

|version|          timestamp|       opération|                                                                  operationParameters|readVersion|
|-------|-------------------|----------------|-------------------------------------------------------------------------------------|-----------|
|      5|2020-04-25 00:37:09|STREAMING UPDATE|[outputMode -> Append, queryId -> d26b4f8a-7e5a-44f2-a5fb-23a7bd02aef7, epochId -> 0]|          4|
|      4|2020-04-25 00:36:27|           MERGE|                                         [predicate -> (oldData.`id` = newData.`id`)]|          3|
|      3|2020-04-25 00:36:08|          Suppression|                  [predicate -> ["((`id` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|          2|
|      2|2020-04-25 00:35:51|          UPDATE|                   [predicate -> ((id#744L % cast(2 as bigint)) = cast(0 as bigint))]|          1|
|      1|2020-04-25 00:35:05|           WRITE|                                               [mode -> Overwrite, partitionBy -> []]|          0|
|      0|2020-04-25 00:34:34|           WRITE|                                           [mode -> ErrorIfExists, partitionBy -> []]|       null|

Ici, vous supprimez certaines colonnes moins intéressantes pour simplifier l’affichage de l’historique.

:::zone pivot = "programming-language-python"

```python
stream.stop()
delta_table.history().drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").show(100, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
stream.Stop();
deltaTable.History().Drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").Show(100, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
stream.stop
deltaTable.history.show
```

Retourne comme résultat :

|version|          timestamp|       opération|                                                                  operationParameters|readVersion|
|-------|-------------------|----------------|-------------------------------------------------------------------------------------|-----------|
|      5|2020-04-25 00:37:09|STREAMING UPDATE|[outputMode -> Append, queryId -> d26b4f8a-7e5a-44f2-a5fb-23a7bd02aef7, epochId -> 0]|          4|
|      4|2020-04-25 00:36:27|           MERGE|                                         [predicate -> (oldData.`id` = newData.`id`)]|          3|
|      3|2020-04-25 00:36:08|          Suppression|                  [predicate -> ["((`id` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|          2|
|      2|2020-04-25 00:35:51|          UPDATE|                   [predicate -> ((id#744L % cast(2 as bigint)) = cast(0 as bigint))]|          1|
|      1|2020-04-25 00:35:05|           WRITE|                                               [mode -> Overwrite, partitionBy -> []]|          0|
|      0|2020-04-25 00:34:34|           WRITE|                                           [mode -> ErrorIfExists, partitionBy -> []]|       null|

::: zone-end

## <a name="convert-parquet-to-delta"></a>Convertir Parquet en Delta

Vous pouvez effectuer une conversion sur place du format Parquet en Delta.

Ici, vous allez tester si la table existante est au format delta.
:::zone pivot = "programming-language-python"

```python
parquet_path = "/parquet/parquet-table-{0}".format(session_id)
data = spark.range(0,5)
data.write.parquet(parquet_path)
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var parquetPath = $"/parquet/parquet-table-{sessionId}";
var data = spark.Range(0,5);
data.Write().Parquet(parquetPath);
DeltaTable.IsDeltaTable(parquetPath)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val parquetPath = s"/parquet/parquet-table-$sessionId"
val data = spark.range(0,5)
data.write.parquet(parquetPath)
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

Retourne comme résultat :

Faux

Vous allez maintenant convertir les données au format delta et vérifier que l’opération a réussi.

:::zone pivot = "programming-language-python"

```python
DeltaTable.convertToDelta(spark, "parquet.`{0}`".format(parquet_path))
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
DeltaTable.ConvertToDelta(spark, $"parquet.`{parquetPath}`");
DeltaTable.IsDeltaTable(parquetPath)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
DeltaTable.convertToDelta(spark, s"parquet.`$parquetPath`")
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

Retourne comme résultat :

Vrai

## <a name="sql-support"></a>Prise en charge SQL

Delta prend en charge les commandes utilitaires de table via SQL. Vous pouvez utiliser SQL pour :

* Obtenir l’historique d’une table Delta
* Nettoyer une table Delta
* Convertir un fichier Parquet en Delta

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE HISTORY delta.`{0}`".format(delta_table_path)).show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql($"DESCRIBE HISTORY delta.`{deltaTablePath}`").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql(s"DESCRIBE HISTORY delta.`$deltaTablePath`").show()
```

::: zone-end

Retourne comme résultat :

|version|          timestamp|userId|userName|       opération| operationParameters| travail|notebook|clusterId|readVersion|isolationLevel|isBlindAppend|
|-------|-------------------|------|--------|----------------|--------------------|----|--------|---------|-----------|--------------|-------------|
|      5|2020-04-25 00:37:09|  null|    null|STREAMING UPDATE|[outputMode -> Ap...|null|    null|     null|          4|          null|         true|
|      4|2020-04-25 00:36:27|  null|    null|           MERGE|[predicate -> (ol...|null|    null|     null|          3|          null|        false|
|      3|2020-04-25 00:36:08|  null|    null|          Suppression|[predicate -> ["(...|null|    null|     null|          2|          null|        false|
|      2|2020-04-25 00:35:51|  null|    null|          UPDATE|[predicate -> ((i...|null|    null|     null|          1|          null|        false|
|      1|2020-04-25 00:35:05|  null|    null|           WRITE|[mode -> Overwrit...|null|    null|     null|          0|          null|        false|
|      0|2020-04-25 00:34:34|  null|    null|           WRITE|[mode -> ErrorIfE...|null|    null|     null|       null|          null|         true|

:::zone pivot = "programming-language-python"

```python
spark.sql("VACUUM delta.`{0}`".format(delta_table_path)).show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql($"VACUUM delta.`{deltaTablePath}`").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql(s"VACUUM delta.`$deltaTablePath`").show()
```

::: zone-end

Retourne comme résultat :

|                path|
|--------------------|
|abfss://data@arca...|

Vous allez maintenant vérifier qu’une table n’est pas une table au format delta, puis vous allez la convertir au format delta à l’aide de Spark SQL et vérifier qu’elle a été correctement convertie.

:::zone pivot = "programming-language-python"

```python
parquet_id = random.randint(0,1000)
parquet_path = "/parquet/parquet-table-{0}-{1}".format(session_id, parquet_path)
data = spark.range(0,5)
data.write.parquet(parquet_path)
DeltaTable.isDeltaTable(spark, parquet_path)
spark.sql("CONVERT TO DELTA parquet.`{0}`".format(parquet_path))
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var parquetId =  (new Random()).Next(10000000);
var parquetPath = $"/parquet/parquet-table-{sessionId}-{parquetId}";
var data = spark.Range(0,5);
data.Write().Parquet(parquetPath);
DeltaTable.IsDeltaTable(parquetPath);
spark.Sql($"CONVERT TO DELTA parquet.`{parquetPath}`");
DeltaTable.IsDeltaTable(parquetPath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val parquetId = scala.util.Random.nextInt(1000)
val parquetPath = s"/parquet/parquet-table-$sessionId-$parquetId"
val data = spark.range(0,5)
data.write.parquet(parquetPath)
DeltaTable.isDeltaTable(parquetPath)
spark.sql(s"CONVERT TO DELTA parquet.`$parquetPath`")
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

Retourne comme résultat :

Vrai

Pour obtenir une documentation complète, consultez la page [Documentation de Delta Lake](https://docs.delta.io/latest/delta-intro.html).

Pour plus d’informations, consultez [Projet Delta Lake](https://github.com/delta-io/delta).

## <a name="next-steps"></a>Étapes suivantes

* [Documentation .NET pour Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
* [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
