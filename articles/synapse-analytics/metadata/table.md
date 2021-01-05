---
title: Tables de métadonnées partagées
description: Azure Synapse Analytics fournit un modèle de métadonnées partagées, où le fait de créer une table dans le pool Apache Spark serverless la rend accessible depuis le pool SQL serverless et le pool SQL dédié sans duplication des données.
services: sql-data-warehouse
author: MikeRys
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: metadata
ms.date: 05/01/2020
ms.author: mrys
ms.reviewer: jrasnick
ms.custom: devx-track-csharp
ms.openlocfilehash: 9ee18edd563d94a85dedf48b7a4d6df394c09707
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461388"
---
# <a name="azure-synapse-analytics-shared-metadata-tables"></a>Tables de métadonnées partagées Azure Synapse Analytics


Azure Synapse Analytics permet aux différents moteurs de calcul d’espace de travail de partager des bases de données et des tables de type Parquet entre ses pools Apache Spark et le pool SQL serverless.

Une fois qu’une base de données a été créée par un travail Spark, vous pouvez y créer des tables avec Spark qui utilisent Parquet comme format de stockage. Ces tables sont immédiatement disponibles pour l’interrogation par n’importe lequel des pools Spark de l’espace de travail Azure Synapse. Elles peuvent également être utilisées à partir de n’importe quel travail Spark, soumis à certaines autorisations.

Les tables Spark créées, gérées et externes sont également mises à disposition comme tables externes sous le même nom dans la base de données synchronisée correspondante dans le pool SQL serverless. [Exposition d’une table Spark dans SQL](#expose-a-spark-table-in-sql) fournit plus de détails sur la synchronisation des tables.

Comme elles sont synchronisées avec le pool SQL serverless de façon asynchrone, les tables apparaissent après un certain délai.

## <a name="manage-a-spark-created-table"></a>Gérer une table créée avec Spark

Utilisez Spark pour gérer les bases de données créées avec Spark. Par exemple, supprimez-la via un travail de pool Apache Spark serverless et créez-y des tables depuis Spark.

Si vous créez des objets dans une base de données de ce type à partir du pool SQL serverless ou que vous essayez de supprimer la base de données, l’opération va réussir, mais la base de données Spark d’origine ne sera pas modifiée.

## <a name="expose-a-spark-table-in-sql"></a>Exposer une table Spark dans SQL

### <a name="shared-spark-tables"></a>Tables Spark partagées

Spark fournit deux types de tables qu’Azure Synapse expose automatiquement dans SQL :

- Tables gérées

  Spark offre de nombreuses options pour le stockage des données dans des tables gérées, telles que TEXT, CSV, JSON, JDBC, PARQUET, ORC, HIVE, DELTA et LIBSVM. Ces fichiers sont normalement stockés dans le répertoire `warehouse` dans lequel sont stockées les données des tables gérées.

- Tables externes

  Spark fournit également des moyens de créer des tables externes sur des données existantes, soit en spécifiant l’option `LOCATION`, soit en utilisant le format Hive. Ces tables externes peuvent être de divers formats de données, notamment Parquet.

Azure Synapse partage actuellement uniquement les tables Spark gérées et externes qui stockent leurs données au format Parquet avec les moteurs SQL. Les tables d’autres formats ne sont pas automatiquement synchronisées. Vous pourrez peut-être synchroniser ces tables de manière explicite en tant que table externe dans votre propre base de données SQL si le moteur SQL prend en charge le format sous-jacent de la table.

### <a name="share-spark-tables"></a>Partager des tables Spark

Les tables Spark gérées et externes partageables sont exposées dans le moteur SQL en tant que tables externes avec les propriétés suivantes :

- La source de données de la table externe SQL est la source de données représentant le dossier d’emplacement de la table Spark.
- Le format de fichier de la table externe SQL est Parquet.
- Les informations d’identification d’accès de la table externe SQL sont directes.

Étant donné que tous les noms de tables Spark sont des noms de tables SQL valides et que tous les noms de colonnes Spark sont des noms de colonnes SQL valides, les noms de tables et de colonnes Spark sont utilisés pour la table externe SQL.

Les tables Spark fournissent des types de données différents de ceux des moteurs SQL Synapse. Le tableau suivant mappe les types de données de table Spark aux types SQL :

| Type de données Spark | Type de données SQL | Commentaires |
|---|---|---|
| `byte`      | `smallint`       ||
| `short`     | `smallint`       ||
| `integer`   |    `int`            ||
| `long`      |    `bigint`         ||
| `float`     | `real`           |<!-- need precision and scale-->|
| `double`    | `float`          |<!-- need precision and scale-->|
| `decimal`      | `decimal`        |<!-- need precision and scale-->|
| `timestamp` |    `datetime2`      |<!-- need precision and scale-->|
| `date`      | `date`           ||
| `string`    |    `varchar(max)`   | Avec classement `Latin1_General_100_BIN2_UTF8` |
| `binary`    |    `varbinary(max)` ||
| `boolean`   |    `bit`            ||
| `array`     |    `varchar(max)`   | Sérialise en JSON avec classement `Latin1_General_100_BIN2_UTF8` |
| `map`       |    `varchar(max)`   | Sérialise en JSON avec classement `Latin1_General_100_BIN2_UTF8` |
| `struct`    |    `varchar(max)`   | Sérialise en JSON avec classement `Latin1_General_100_BIN2_UTF8` |

<!-- TODO: Add precision and scale to the types mentioned above -->

## <a name="security-model"></a>Modèle de sécurité

Les bases de données et les tables Spark, ainsi que leurs représentations synchronisées dans le moteur SQL, sont sécurisées au niveau du stockage sous-jacent. Puisqu’elles n’ont actuellement pas d’autorisations sur les objets eux-mêmes, les objets sont visibles dans l’Explorateur d’objets.

Le principal de sécurité qui crée une table gérée est considéré comme le propriétaire de cette table et dispose de tous les droits sur la table ainsi que sur les dossiers et fichiers sous-jacents. En outre, le propriétaire de la base de données devient automatiquement copropriétaire de la table.

Si vous créez une table externe SQL ou Spark avec authentification directe, les données sont sécurisées uniquement au niveau des dossiers et des fichiers. Si quelqu’un interroge ce type de table externe, l’identité de sécurité de l’expéditeur de la requête est transmise au système de fichiers, qui vérifiera les droits d’accès.

Pour plus d’informations sur la façon de définir des autorisations sur les dossiers et les fichiers, consultez [Base de données partagée Azure Synapse Analytics](database.md).

## <a name="examples"></a>Exemples

### <a name="create-a-managed-table-backed-by-parquet-in-spark-and-query-from-serverless-sql-pool"></a>Créer une table gérée au format Parquet dans Spark et interroger à partir du pool SQL serverless

Dans ce scénario, vous disposez d’une base de données Spark nommée `mytestdb`. Consultez [Créer une base de données Spark et s’y connecter avec le pool SQL serverless](database.md#create-and-connect-to-spark-database-with-serverless-sql-pool).

Créez une table Spark gérée avec SparkSQL en exécutant la commande suivante :

```sql
    CREATE TABLE mytestdb.myParquetTable(id int, name string, birthdate date) USING Parquet
```

Cette commande crée la table `myParquetTable` dans la base de données `mytestdb`. Après un bref délai, vous pouvez voir la table dans votre pool SQL serverless. Exécutez par exemple l’instruction suivante à partir de votre pool SQL serverless.

```sql
    USE mytestdb;
    SELECT * FROM sys.tables;
```

Vérifiez que `myParquetTable` est inclus dans les résultats.

>[!NOTE]
>Une table qui n’utilise pas Parquet comme format de stockage n’est pas synchronisée.

Ensuite, insérez des valeurs dans la table à partir de Spark, par exemple avec les instructions Spark C# suivantes dans un notebook C# :

```csharp
using Microsoft.Spark.Sql.Types;

var data = new List<GenericRow>();

data.Add(new GenericRow(new object[] { 1, "Alice", new Date(2010, 1, 1)}));
data.Add(new GenericRow(new object[] { 2, "Bob", new Date(1990, 1, 1)}));

var schema = new StructType
    (new List<StructField>()
        {
            new StructField("id", new IntegerType()),
            new StructField("name", new StringType()),
            new StructField("birthdate", new DateType())
        }
    );

var df = spark.CreateDataFrame(data, schema);
df.Write().Mode(SaveMode.Append).InsertInto("mytestdb.myParquetTable");
```

Vous pouvez maintenant lire les données de votre pool SQL serverless comme suit :

```sql
SELECT * FROM mytestdb.dbo.myParquetTable WHERE name = 'Alice';
```

Vous devez obtenir la ligne suivante comme résultat :

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

### <a name="create-an-external-table-backed-by-parquet-in-spark-and-query-from-serverless-sql-pool"></a>Créer une table externe au format Parquet dans Spark et interroger à partir du pool SQL serverless

Dans cet exemple, vous allez créer une table Spark externe sur les fichiers de données Parquet qui ont été créés dans l’exemple précédent pour la table gérée.

Par exemple, avec SparkSQL, exécutez :

```sql
CREATE TABLE mytestdb.myExternalParquetTable
    USING Parquet
    LOCATION "abfss://<fs>@arcadialake.dfs.core.windows.net/synapse/workspaces/<synapse_ws>/warehouse/mytestdb.db/myparquettable/"
```

Remplacez l’espace réservé `<fs>` par le nom du système de fichiers qui est le système de fichiers par défaut de l’espace de travail, et l’espace réservé `<synapse_ws>` par le nom de l’espace de travail Synapse que vous utilisez pour exécuter cet exemple.

L’exemple précédent crée la table `myExtneralParquetTable` dans la base de données `mytestdb`. Après un bref délai, vous pouvez voir la table dans votre pool SQL serverless. Exécutez par exemple l’instruction suivante à partir de votre pool SQL serverless.

```sql
USE mytestdb;
SELECT * FROM sys.tables;
```

Vérifiez que `myExternalParquetTable` est inclus dans les résultats.

Vous pouvez maintenant lire les données de votre pool SQL serverless comme suit :

```sql
SELECT * FROM mytestdb.dbo.myExternalParquetTable WHERE name = 'Alice';
```

Vous devez obtenir la ligne suivante comme résultat :

```
id | name | birthdate
---+-------+-----------
1 | Alice | 2010-01-01
```

## <a name="next-steps"></a>Étapes suivantes

- [Apprenez-en davantage sur les métadonnées partagées Azure Synapse Analytics](overview.md)
- [En savoir plus sur la base de données de métadonnées partagées Azure Synapse Analytics](database.md)


