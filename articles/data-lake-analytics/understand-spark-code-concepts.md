---
title: Comprendre les concepts de code Apache Spark pour les développeurs U-SQL d’Azure Data Lake Analytics.
description: Cet article décrit les concepts d’Apache Spark pour aider les développeurs U-SQL à comprendre les concepts du code Spark.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.custom: Understand-apache-spark-code-concepts
ms.date: 10/15/2019
ms.openlocfilehash: 2abd5882e310b17c633a82009f44624fad156f14
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221126"
---
# <a name="understand-apache-spark-code-for-u-sql-developers"></a>Comprendre le code Apache Spark pour les développeurs U-SQL

Cette section fournit des conseils de haut niveau sur la transformation des scripts U-SQL en Apache Spark.

- Il commence par une [comparaison des paradigmes de traitement des deux langages informatiques](#understand-the-u-sql-and-spark-language-and-processing-paradigms).
- Fournit des conseils sur la façon de :
   - [Transformer des scripts](#transform-u-sql-scripts), y compris des [expressions d’ensemble de lignes](#transform-u-sql-rowset-expressions-and-sql-based-scalar-expressions) d’U-SQL
   - [Code .NET](#transform-net-code)
   - [Types de données](#transform-typed-values)
   - [Objets de catalogue](#transform-u-sql-catalog-objects)

## <a name="understand-the-u-sql-and-spark-language-and-processing-paradigms"></a>Comprendre les paradigmes de langage et de traitement U-SQL et Spark

Avant de commencer à migrer les scripts U-SQL d’Azure Data Lake Analytics vers Spark, il est utile de comprendre la philosophie générale du langage et du traitement des deux systèmes.

U-SQL est un langage de requête déclaratif de type SQL qui utilise un paradigme de flux de données et vous permet d’incorporer et d’effectuer facilement un scale-out du code utilisateur écrit dans écrit en.NET (par exemple C#), Python et R. Les extensions utilisateur peuvent implémenter des expressions simples ou des fonctions définies par l’utilisateur, mais elles peuvent également fournir à l’utilisateur la possibilité d’implémenter des opérateurs définis par l’utilisateur qui implémentent des opérateurs personnalisés pour effectuer des transformations au niveau de l’ensemble de lignes, des extractions et l’écriture de la sortie.

Spark est une infrastructure Scale-out qui offre plusieurs liaisons de langage Scala, Java, Python, .NET, etc. et dans laquelle vous écrivez principalement votre code dans l’un de ces langages, créez des abstractions de données appelées jeux de données distribués résilients (RDD), des trames et jeux de données, puis utilisez un langage spécifique à un domaine (DSL) de type LINQ pour les transformer. Il fournit également SparkSQL sous la forme d’un sous-langage déclaratif sur les abstractions de trames et jeux de données. Le DSL prévoit deux catégories d’opérations, de transformations et d’actions. L’application de transformations aux abstractions de données n’exécutera pas la transformation, mais générera à la place le plan d’exécution qui sera soumis pour évaluation avec une action (par exemple, en écrivant le résultat dans une table ou un fichier temporaire, ou en imprimant le résultat).

Ainsi, lors de la traduction d’un script U-SQL en programme Spark, vous devrez décider quel langage vous voulez utiliser pour générer au moins l’abstraction de la trame de données (qui est actuellement la plus fréquemment utilisée) et si vous souhaitez écrire les transformations déclaratives de dataflow en utilisant DSL ou SparkSQL. Dans certains cas plus complexes, vous devrez peut-être fractionner votre script U-SQL en une séquence de Spark et d’autres étapes implémentées avec Azure Batch ou Azure Functions.

En outre, Azure Data Lake Analytics offre U-SQL dans un environnement de service de travail serverless, tandis qu’Azure Databricks et Azure HDInsight offrent Spark sous la forme d’un service de clusters. Lorsque vous transformez votre application, vous devez prendre en compte les implications de la création, du dimensionnement, de la mise à l’échelle et de la déclassement des clusters.

## <a name="transform-u-sql-scripts"></a>Transformer des scripts U-SQL

Les scripts U-SQL suivent le modèle de traitement suivant :

1. Les données sont lues à partir de fichiers non structurés, à l’aide de l’instruction `EXTRACT`, une spécification d’emplacement ou de jeu de fichiers, de l’extracteur intégré ou défini par l’utilisateur et du schéma souhaité, ou à partir de tables U-SQL (tables managées ou externes). Il est représenté sous la forme d’un ensemble de lignes.
2. Les ensembles de lignes sont transformés en plusieurs instructions U-SQL qui appliquent des expressions U-SQL à ces ensembles et en produisent de nouveaux.
3. Enfin, les ensembles de lignes résultants sont générés soit dans des fichiers à l’aide de l’instruction `OUTPUT` qui spécifie les emplacements et d’un générateur de sortie intégré ou défini par l’utilisateur, soit dans une table U-SQL.

Le script est évalué de manière différée, ce qui signifie que chaque étape d’extraction et de transformation est composée dans une arborescence de l’expression et évaluée globalement (le dataflow).

Les programmes Spark sont similaires en ce sens que vous utiliseriez des connecteurs Spark pour lire les données et créer les trames de données, puis appliqueriez les transformations sur le trames de données en utilisant soit le DSL de type LINQ soit SparkSQL, pour enfin écrire le résultat dans des fichiers, des tables Spark temporaires, certains types de langages de programmation ou la console.

## <a name="transform-net-code"></a>Transformer le code .NET

Le langage d’expression d’U-SQL est C# et il offre plusieurs moyens d’effectuer un scale-out du code .NET personnalisé.

Dans la mesure où Spark ne prend pas en charge l’exécution de code .NET en mode natif, vous devrez réécrire vos expressions dans une expression Spark, Scala, Java ou Python équivalente ou trouver un moyen d’appeler votre code .NET. Si votre script utilise des bibliothèques .NET, vous disposez des options suivantes :

- Traduisez votre code .NET en Scala ou Python.
- Fractionnez votre script U-SQL en plusieurs étapes, en utilisant les processus Azure Batch pour appliquer les transformations .NET (si vous pouvez bénéficier d’une mise à l’échelle acceptable).
- Utilisez une liaison de langage .NET appelée Moebius, disponible en open source. Ce projet n’est pas dans un état pris en charge.

Dans tous les cas, si vous avez une grande quantité de logique .NET dans vos scripts U-SQL, veuillez nous contacter par le biais de votre représentant de compte Microsoft pour obtenir de l’aide.

Les informations suivantes concernent les différents cas d’utilisation de .NET et C# dans les scripts U-SQL.

### <a name="transform-scalar-inline-u-sql-c-expressions"></a>Transformer des expressions U-SQL C# inlined scalaires

Le langage d’expression de U-SQL est C#. La plupart des expressions U-SQL inlined scalaires sont implémentées en mode natif pour améliorer les performances, tandis que les expressions plus complexes peuvent être exécutées en appelant le .NET Framework.

Spark possède son propre langage d’expression scalaire (dans le cadre du DSL ou de SparkSQL) et permet l’appel de fonctions définies par l’utilisateur écrites dans son langage d’hébergement.

Si vous avez des expressions scalaires en U-SQL, vous devez d’abord trouver l’expression scalaire Spark la plus appropriée en mode natif pour obtenir les meilleures performances, puis mapper les autres expressions dans une fonction définie par l’utilisateur du langage d’hébergement Spark de votre choix.

Sachez que .NET et C# ont une sémantique de type différente de celle des langages d’hébergement Spark et du DSL de Spark. Voir [ci-dessous](#transform-typed-values) pour plus d’informations sur les différences entre les types de systèmes.

### <a name="transform-user-defined-scalar-net-functions-and-user-defined-aggregators"></a>Transformer des fonctions .NET scalaires et agrégateurs définis par l’utilisateur

U-SQL permet d’appeler des fonctions .NET scalaires arbitraires ainsi que des agrégateurs définis par l’utilisateur écrits en .NET.

Spark offre également la prise en charge des fonctions et agrégateurs définis par l’utilisateur écrits dans la plupart de ses langages d’hébergement qui peuvent être appelés à partir du DSL de Spark et de SparkSQL.

### <a name="transform-user-defined-operators-udos"></a>Transformer des opérateurs définis par l’utilisateur (UDO)

U-SQL fournit plusieurs catégories d’opérateurs définis par l’utilisateur (UDO), tels que des extracteurs, des générateurs de sortie, des réducteurs, des processeurs, des applicateurs et des combinateurs qui peuvent être écrits en .NET (et, dans une certaine mesure, dans Python et R).

Spark n’offre pas le même modèle d’extensibilité pour les opérateurs, mais offre des fonctionnalités équivalentes pour certains.

L’équivalent Spark des extracteurs et des générateurs de sortie est les connecteurs Spark. Pour de nombreux extracteurs U-SQL, vous pouvez trouver un connecteur équivalent dans la communauté Spark. Pour les autres, vous devrez écrire un connecteur personnalisé. Si l’extracteur U-SQL est complexe et utilise plusieurs bibliothèques .NET, il peut être préférable de créer un connecteur en Scala qui utilise l’interopérabilité pour appeler la bibliothèque .NET qui effectue le traitement réel des données. Dans ce cas, vous devrez déployer le runtime .NET Core sur le cluster Spark et vous assurer que les bibliothèques .NET référencées sont conformes au .NET Standard 2.0.

Les autres types d’UDO U-SQL devront être réécrits à l’aide de fonctions et d’agrégateurs définis par l’utilisateur et de l’expression DLS de Spark ou SparkSQL sémantiquement appropriée. Par exemple, un processeur peut être mappé à une SÉLECTION d’un grand nombre d’appels UDF, empaqueté en tant que fonction qui prend une trame de données comme argument et retourne une trame de données.

### <a name="transform-u-sqls-optional-libraries"></a>Transformer les bibliothèques facultatives U-SQL

U-SQL fournit un ensemble de bibliothèques facultatives et de démonstration qui offrent la prise en charge de [Python](data-lake-analytics-u-sql-python-extensions.md), [R](data-lake-analytics-u-sql-r-extensions.md), [JSON, XML et AVRO](https://github.com/Azure/usql/tree/master/Examples/DataFormats) et certaines [fonctionnalités de services cognitifs](data-lake-analytics-u-sql-cognitive.md).

Spark offre sa propre intégration de Python et R, pySpark et SparkR respectivement, et fournit des connecteurs pour lire et écrire JSON, XML et AVRO.

Si vous avez besoin de transformer un script référençant les bibliothèques de services cognitifs, nous vous recommandons de nous contacter par l’intermédiaire de votre représentant de compte Microsoft.

## <a name="transform-typed-values"></a>Transformer les valeurs typées

Étant donné que le système de type U-SQL est basé sur le système de type.NET et que Spark a son propre système de type, qui est influencé par la liaison du langage hôte, vous devrez vous assurer que les types que vous utilisez sont proches et, pour certains types, les plages, la précision et/ou la mise à l’échelle peuvent être légèrement différentes. En outre, U-SQL et Spark traitent les valeurs `null` différemment.

### <a name="data-types"></a>Types de données

Le tableau suivant donne les types équivalents en Spark, Scala et PySpark pour les types U-SQL donnés.

| U-SQL | Spark |  Scala | PySpark |
| ------ | ------ | ------ | ------ |
|`byte`       ||||
|`sbyte`      |`ByteType` |`Byte` | `ByteType`|
|`int`        |`IntegerType` |`Int` | `IntegerType`|
|`uint`       ||||
|`long`       |`LongType` |`Long` | `LongType`|
|`ulong`      ||||
|`float`      |`FloatType` |`Float` | `FloatType`|
|`double`     |`DoubleType` |`Double` | `DoubleType`|
|`decimal`    |`DecimalType` |`java.math.BigDecimal` | `DecimalType`|
|`short`      |`ShortType` |`Short` | `ShortType`|
|`ushort`     ||||
|`char`   | |`Char`||
|`string` |`StringType` |`String` |`StringType` |
|`DateTime`   |`DateType`, `TimestampType` |`java.sql.Date`, `java.sql.Timestamp` | `DateType`, `TimestampType`|
|`bool`   |`BooleanType` |`Boolean` | `BooleanType`|
|`Guid`   ||||
|`byte[]` |`BinaryType` |`Array[Byte]` | `BinaryType`|
|`SQL.MAP<K,V>`   |`MapType(keyType, valueType, valueContainsNull)` |`scala.collection.Map` | `MapType(keyType, valueType, valueContainsNull=True)`|
|`SQL.ARRAY<T>`   |`ArrayType(elementType, containsNull)` |`scala.collection.Seq` | `ArrayType(elementType, containsNull=True)`|

Pour plus d'informations, consultez les pages suivantes :

- [org.apache.spark.sql.types](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.types.package)
- [Types Spark SQL et DataFrames](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
- [Types valeur Scala](https://www.scala-lang.org/api/current/scala/AnyVal.html)
- [pyspark.sql.types](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#module-pyspark.sql.types)

### <a name="treatment-of-null"></a>Traitement de la valeur NULL

En Spark, les types par défaut autorisent les valeurs NULL alors qu’en U-SQL, vous marquez explicitement une valeur scalaire qui ne représente pas un objet comme pouvant accepter la valeur Null. Bien que Spark vous permette de définir une colonne comme n’acceptant pas la valeur Null, cela n’appliquera pas la contrainte et [peut conduire à un résultat erroné](https://medium.com/@weshoffman/apache-spark-parquet-and-troublesome-nulls-28712b06f836).

En Spark, NULL indique que la valeur est inconnue. Une valeur NULL Spark est différente de toute valeur, y compris elle-même. Les comparaisons entre deux valeurs NULL Spark, ou entre une valeur NULL et toute autre valeur, retournent le résultat Inconnu, car la valeur de chaque NULL est inconnue.  

Ce comportement est différent d’U-SQL, qui suit la sémantique C# où `null` est différente de toute valeur, mais égale à elle-même.  

Ainsi, une instruction `SELECT` SparkSQL qui utilise `WHERE column_name = NULL` retourne zéro ligne, même s’il existe des valeurs NULL dans `column_name`, alors qu’en U-SQL, elle retourne les lignes où `column_name` est défini sur `null`. De même, une instruction `SELECT` Spark qui utilise `WHERE column_name != NULL` retourne zéro ligne, même s’il existe des valeurs non Null dans `column_name`, alors qu’en U-SQL, elle retourne les lignes qui contiennent les valeurs non Null. Par conséquent, si vous souhaitez la sémantique U-SQL de vérification des valeurs Null, vous devez utiliser [isnull](https://spark.apache.org/docs/2.3.0/api/sql/index.html#isnull)) et [isnotnull](https://spark.apache.org/docs/2.3.0/api/sql/index.html#isnotnull) respectivement (ou leur équivalent DSL).

## <a name="transform-u-sql-catalog-objects"></a>Transformer des objets de catalogue U-SQL

L’une des principales différences réside dans le fait que les scripts U-SQL peuvent utiliser ses objets de catalogue, dont un grand nombre n’ont pas d’équivalent Spark direct.

Spark assure la prise en charge des concepts du magasin Hive Meta, principalement des bases de données et des tables, de sorte que vous pouvez mapper des bases de données et des schémas U-SQL à des bases de données Hive, et des tables U-SQL à des tables Spark (voir [Déplacement de données stockées dans des tables U-SQL](understand-spark-data-formats.md#move-data-stored-in-u-sql-tables)), mais n’offre aucune prise en charge des affichages, des Fonctions table (TVF), des procédures stockées, des assemblys U-SQL, des sources de données externes, etc.

Les objets de code U-SQL, tels que les affichages, les TVF, les procédures stockées et les assemblys, peuvent être modélisés à l’aide de fonctions de code et de bibliothèques en Spark et référencés en utilisant les mécanismes d’abstraction de fonctions et de procédures du langage hôte (par exemple, en important des modules Python ou en faisant référence à des fonctions Scala).

Si le catalogue U-SQL a été utilisé pour partager des données et des objets de code entre des projets et des équipes, les mécanismes équivalents doivent être utilisés pour le partage (par exemple, Maven pour le partage d’objets de code).

## <a name="transform-u-sql-rowset-expressions-and-sql-based-scalar-expressions"></a>Transformer des expressions d’ensemble de lignes U-SQL et des expressions scalaires basées sur SQL

Le langage de base d’U-SQL transforme des ensembles de lignes et repose sur SQL. Voici une liste non exhaustive des expressions d’ensemble de lignes les plus courantes offertes en U-SQL :

- `SELECT`/`FROM`/`WHERE`/`GROUP BY`+Aggregates+`HAVING`/`ORDER BY`+`FETCH`
- Expressions `INNER`/`OUTER`/`CROSS`/`SEMI` `JOIN`
- Expressions `CROSS`/`OUTER` `APPLY`
- Expressions `PIVOT`/`UNPIVOT`
- Constructeur d’ensemble de lignes `VALUES`

- Expressions définies `UNION`/`OUTER UNION`/`INTERSECT`/`EXCEPT`

En outre, U-SQL fournit une variété d’expressions scalaires basées sur SQL, telles que

- des expressions de fenêtrage `OVER`
- plusieurs agrégateurs intégrés et fonctions de classement (`SUM`, `FIRST`, etc.)
- Voici quelques-unes des expressions scalaires SQL les plus familières : `CASE`, `LIKE`, (`NOT`) `IN`, `AND`, `OR`, etc.

Spark propose des expressions équivalentes dans sa forme DSL et SparkSQL pour la plupart de ces expressions. Certaines des expressions non prises en charge en mode natif en Spark devront être réécrites à l’aide d’une combinaison des expressions Spark natives et des modèles sémantiquement équivalents. Par exemple, `OUTER UNION` doit être traduit dans la combinaison équivalente de projections et d’unions.

En raison du traitement différent des valeurs NULL, une jointure U-SQL correspondra toujours à une ligne si les deux colonnes comparées contiennent une valeur NULL, tandis qu’une jointure en Spark ne correspondra pas à ces colonnes à moins que des vérifications des valeurs Null explicites soient ajoutées.

## <a name="transform-other-u-sql-concepts"></a>Transformer d’autres concepts U-SQL

U-SQL offre également un large éventail d’autres fonctionnalités et concepts, tels que les requêtes fédérées par rapport aux bases de données SQL Server, les paramètres, les scalaires, les variables d’expression lambda, les variables système et les conseils `OPTION`.

### <a name="federated-queries-against-sql-server-databasesexternal-tables"></a>Requêtes fédérées par rapport à des bases de données SQL Server/tables externes

U-SQL fournit une source de données et des tables externes, ainsi que des requêtes directes par rapport à Azure SQL Database. Si Spark n’offre pas les mêmes abstractions d’objet, il fournit un [connecteur Spark pour Azure SQL Database](../azure-sql/database/spark-connector.md) qui peut être utilisé pour interroger des bases de données SQL.

### <a name="u-sql-parameters-and-variables"></a>Paramètres et variables U-SQL

Les paramètres et les variables utilisateur ont des concepts équivalents en Spark et dans leurs langages d’hébergement.

Par exemple, en Scala, vous pouvez définir une variable avec le mot clé `var` :

```
var x = 2 * 3;
println(x)
```

Les variables système d’U-SQL (variables commençant par `@@`) peuvent être divisées en deux catégories :

- Variables système paramétrables qui peuvent être définies sur des valeurs spécifiques pour avoir un impact sur le comportement des scripts
- Variables système d’information qui interrogent les informations au niveau du système et du travail

La plupart des variables système paramétrables n’ont pas d’équivalent direct en Spark. Certaines des variables système d’information peuvent être modélisées en passant les informations en tant qu’arguments lors de l’exécution des tâches. D’autres peuvent avoir une fonction équivalente dans le langage d’hébergement de Spark.

### <a name="u-sql-hints"></a>Conseils U-SQL

U-SQL offre plusieurs façons syntaxiques de fournir des conseils à l’optimiseur de requête et au moteur d’exécution :  

- Paramétrage d’une variable système U-SQL
- une clause `OPTION` associée à l’expression de l’ensemble de lignes pour fournir un conseil relatif aux données ou au plan
- un conseil de jointure dans la syntaxe de l’expression de jointure (par exemple, `BROADCASTLEFT`)

L’optimiseur de requête basé sur les coûts de Spark possède ses propres fonctionnalités pour fournir des conseils et optimiser les performances des requêtes. Veuillez vous reporter à la documentation correspondante.

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les formats de données Spark pour les développeurs U-SQL](understand-spark-data-formats.md)
- [.NET pour Apache Spark](/dotnet/spark/what-is-apache-spark-dotnet)
- [Mettre à niveau vos solutions d’analytique de Big Data d’Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-migrate-gen1-to-gen2.md)
- [Transformer des données à l’aide d’une activité Spark dans Azure Data Factory](../data-factory/transform-data-using-spark.md)
- [Transformer des données à l’aide d’une activité Hadoop Hive dans Azure Data Factory](../data-factory/transform-data-using-hadoop-hive.md)
- [Présentation d’Apache Spark dans Azure HDInsight](../hdinsight/spark/apache-spark-overview.md)