---
title: Utiliser Apache Spark pour lire et écrire des données dans Azure SQL Database
description: Découvrez comment configurer une connexion entre le cluster HDInsight Spark et une instance Azure SQL Database. Pour lire, écrire et transmettre en continu des données dans une base de données SQL
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/20/2020
ms.openlocfilehash: d979a68f4e3aa0071fb7654647610af1fbf95e90
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023974"
---
# <a name="use-hdinsight-spark-cluster-to-read-and-write-data-to-azure-sql-database"></a>Utiliser le cluster HDInsight Spark pour lire et écrire des données dans Azure SQL Database

Découvrez comment connecter un cluster Apache Spark dans Azure HDInsight à une instance Azure SQL Database. Puis lisez, écrivez et transmettez en continu des données dans une base de données SQL. Les instructions de cet article utilisent un document Jupyter Notebook pour exécuter les extraits de code Scala. Toutefois, vous pouvez créer une application autonome en Scala ou Python et effectuer les mêmes tâches.

## <a name="prerequisites"></a>Prérequis

* Un cluster Azure HDInsight Spark.  Suivez les instructions indiquées dans [Créer un cluster Apache Spark dans Azure HDInsight](apache-spark-jupyter-spark-sql.md).

* Azure SQL Database. Suivez les instructions indiquées dans [Créer une base de données dans Azure SQL Database](../../azure-sql/database/single-database-create-quickstart.md). Vérifiez que vous créez une base de données avec l’exemple de schéma et de données **AdventureWorksLT**. En outre, assurez-vous de créer une règle de pare-feu de niveau serveur pour autoriser l’adresse IP de votre client à accéder à la base de données SQL. Les instructions pour ajouter la règle de pare-feu sont disponibles dans le même article. Après avoir créé votre base de données SQL, veillez à garder les valeurs suivantes à portée de main. Vous en avez besoin pour vous connecter à la base de données à partir d’un cluster Spark.

    * Nom du serveur.
    * Nom de la base de données.
    * Nom et mot de passe de l’administrateur de la base de données Azure SQL.

* SQL Server Management Studio (SSMS). Suivez les instructions indiquées dans [Utilisez SQL Server Management Studio pour vous connecter et exécuter des requêtes](../../azure-sql/database/connect-query-ssms.md).

## <a name="create-a-jupyter-notebook"></a>Créer un bloc-notes Jupyter Notebook

Commencez par créer un document Jupyter Notebook devant être associé au cluster Spark. Ce bloc-notes vous permet d’exécuter les extraits de code utilisés dans cet article.

1. À partir du [portail Azure](https://portal.azure.com/), ouvrez votre cluster.
1. Sur la droite, sous **Tableaux de bord du cluster**, sélectionnez **Bloc-notes Jupyter**.  Si vous ne voyez pas **Tableaux de bord du cluster**, sélectionnez **Vue d’ensemble** dans le menu de gauche. Si vous y êtes invité, entrez les informations d’identification d’administrateur pour le cluster.

    ![Notebook Jupyter sur Apache Spark](./media/apache-spark-connect-to-sql-database/hdinsight-spark-cluster-dashboard-jupyter-notebook.png "Notebook Jupyter sur Spark")

   > [!NOTE]  
   > Vous pouvez également accéder au bloc-notes Jupyter sur le cluster Spark en ouvrant l’URL suivante dans votre navigateur. Remplacez **CLUSTERNAME** par le nom de votre cluster.
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`

1. Dans le bloc-notes Jupyter, à partir de l’angle supérieur droit, cliquez sur **Nouveau**, puis cliquez sur **Spark** pour créer un bloc-notes Scala. Les bloc-notes Jupyter sur le cluster HDInsight Spark fournissent également le noyau **PySpark** pour les applications Python2 et le noyau **PySpark3** pour les applications Python3. Pour cet article, nous créons un bloc-notes Scala.

    ![Noyaux pour le Notebook Jupyter sur Spark](./media/apache-spark-connect-to-sql-database/kernel-jupyter-notebook-on-spark.png "Noyaux pour le Notebook Jupyter sur Spark")

    Pour plus d’informations sur les noyaux, consultez [Noyaux de blocs-notes Jupyter avec les clusters Apache Spark dans HDInsight](apache-spark-jupyter-notebook-kernels.md).

   > [!NOTE]  
   > Dans cet article, nous utilisons un noyau Spark (Scala), car envoyer des données depuis Spark vers Microsoft Azure SQL Database n’est pris en charge que dans Scala et Java. Bien que la lecture et l’écriture dans la base de données SQL puissent être effectuées à l’aide de Python, par souci de cohérence dans cet article, nous utilisons Scala pour ces trois opérations.

1. Un nouveau notebook s’ouvre, nommé **Sans titre** par défaut. Cliquez sur le nom du bloc-notes, puis entrez le nom de votre choix.

    ![Fournir un nom au notebook](./media/apache-spark-connect-to-sql-database/hdinsight-spark-jupyter-notebook-name.png "Fournir un nom pour le bloc-notes")

Vous pouvez maintenant commencer à créer votre application.

## <a name="read-data-from-azure-sql-database"></a>Lire des données à partir d’Azure SQL Database

Dans cette section, vous lisez des données à partir d’une table (par exemple, **SalesLT.Address**) de la base de données AdventureWorks.

1. Dans une cellule de code d’un nouveau notebook Jupyter, collez l’extrait de code suivant et remplacez les valeurs d’espace réservé par les valeurs de votre base de données.

    ```scala
    // Declare the values for your database

    val jdbcUsername = "<SQL DB ADMIN USER>"
    val jdbcPassword = "<SQL DB ADMIN PWD>"
    val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
    val jdbcPort = 1433
    val jdbcDatabase ="<AZURE SQL DB NAME>"
    ```

    Appuyez sur **Maj+Entrée** pour exécuter la cellule de code.  

1. Utilisez l’extrait de code ci-après pour créer une URL JDBC que vous pouvez passer aux API de dataframe Spark. Le code crée un objet `Properties` pour contenir les paramètres. Collez l’extrait de code dans une cellule de code, puis appuyez sur **Maj+Entrée** pour l’exécuter.

    ```scala
    import java.util.Properties

    val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=60;"
    val connectionProperties = new Properties()
    connectionProperties.put("user", s"${jdbcUsername}")
    connectionProperties.put("password", s"${jdbcPassword}")
    ```

1. Utilisez l’extrait de code ci-après pour créer un dataframe avec les données d’une table de votre base de données. Dans cet extrait de code, nous utilisons une table `SalesLT.Address` disponible qui fait partie de la base de données **AdventureWorksLT**. Collez l’extrait de code dans une cellule de code, puis appuyez sur **Maj+Entrée** pour l’exécuter.

    ```scala
    val sqlTableDF = spark.read.jdbc(jdbc_url, "SalesLT.Address", connectionProperties)
    ```

1. Vous pouvez à présent effectuer des opérations sur le dataframe, telles que l’obtention du schéma de données :

    ```scala
    sqlTableDF.printSchema
    ```

    Le résultat ressemble à l’image suivante :

    ![sortie de schéma](./media/apache-spark-connect-to-sql-database/read-from-sql-schema-output.png "sortie de schéma")

1. Vous pouvez également effectuer des opérations telles que la récupération des 10 premières lignes.

    ```scala
    sqlTableDF.show(10)
    ```

1. Ou bien, récupérez des colonnes spécifiques du jeu de données.

    ```scala
    sqlTableDF.select("AddressLine1", "City").show(10)
    ```

## <a name="write-data-into-azure-sql-database"></a>Écrire des données dans Azure SQL Database

Dans cette section, nous utilisons un exemple de fichier CSV disponible sur le cluster pour créer une table dans votre base de données et la remplir avec des données. L’exemple de fichier CSV (**HVAC.csv**) est disponible sur tous les clusters HDInsight à l’emplacement `HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv`.

1. Dans une cellule de code d’un nouveau notebook Jupyter, collez l’extrait de code suivant et remplacez les valeurs d’espace réservé par les valeurs de votre base de données.

    ```scala
    // Declare the values for your database

    val jdbcUsername = "<SQL DB ADMIN USER>"
    val jdbcPassword = "<SQL DB ADMIN PWD>"
    val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
    val jdbcPort = 1433
    val jdbcDatabase ="<AZURE SQL DB NAME>"
    ```

    Appuyez sur **Maj+Entrée** pour exécuter la cellule de code.  

1. L’extrait de code ci-après pour créer une URL JDBC que vous pouvez passer aux API de dataframe Spark. Le code crée un objet `Properties` pour contenir les paramètres. Collez l’extrait de code dans une cellule de code, puis appuyez sur **Maj+Entrée** pour l’exécuter.

    ```scala
    import java.util.Properties

    val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=60;"
    val connectionProperties = new Properties()
    connectionProperties.put("user", s"${jdbcUsername}")
    connectionProperties.put("password", s"${jdbcPassword}")
    ```

1. Utilisez l’extrait de code ci-après pour extraire le schéma des données dans HVAC.csv et l’utiliser pour charger les données à partir du fichier CSV dans un dataframe, `readDf`. Collez l’extrait de code dans une cellule de code, puis appuyez sur **Maj+Entrée** pour l’exécuter.

    ```scala
    val userSchema = spark.read.option("header", "true").csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv").schema
    val readDf = spark.read.format("csv").schema(userSchema).load("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    ```

1. Utilisez le dataframe `readDf` pour créer une table temporaire, `temphvactable`. Ensuite, utilisez la table temporaire pour créer une table hive, `hvactable_hive`.

    ```scala
    readDf.createOrReplaceTempView("temphvactable")
    spark.sql("create table hvactable_hive as select * from temphvactable")
    ```

1. Enfin, utilisez la table hive pour créer une table dans votre base de données. L’extrait de code suivant crée `hvactable` dans Azure SQL Database.

    ```scala
    spark.table("hvactable_hive").write.jdbc(jdbc_url, "hvactable", connectionProperties)
    ```

1. Connectez-vous à Azure SQL Database à l’aide de SSMS et vérifiez que vous y voyez une table `dbo.hvactable`.

    a. Démarrez SSMS et connectez-vous à Azure SQL Database en fournissant les informations de connexion, comme indiqué dans la capture d’écran ci-dessous.

    ![Se connecter à SQL Database avec SSMS1](./media/apache-spark-connect-to-sql-database/connect-to-sql-db-ssms.png "Se connecter à SQL Database avec SSMS1")

    b. À partir de l’**Explorateur d’objets**, développez la base de données et le nœud Table pour voir la table **dbo.hvactable** créée.

    ![Se connecter à SQL Database avec SSMS2](./media/apache-spark-connect-to-sql-database/connect-to-sql-db-ssms-locate-table.png "Se connecter à SQL Database avec SSMS2")

1. Exécutez une requête dans SSMS pour voir les colonnes de la table.

    ```sql
    SELECT * from hvactable
    ```

## <a name="stream-data-into-azure-sql-database"></a>Envoyer des données dans Azure SQL Database

Dans cette section, nous envoyons des données dans la table `hvactable` que vous avez créée à la section précédente.

1. Dans un premier temps, vérifiez que la table `hvactable` ne contient aucun enregistrement. À l’aide de SSMS, exécutez la requête suivante sur la table.

    ```sql
    TRUNCATE TABLE [dbo].[hvactable]
    ```

1. Créez un bloc-notes Jupyter sur le cluster HDInsight Spark. Dans une cellule de code, collez l’extrait suivant, puis appuyez sur **Maj+Entrée** :

    ```scala
    import org.apache.spark.sql._
    import org.apache.spark.sql.types._
    import org.apache.spark.sql.functions._
    import org.apache.spark.sql.streaming._
    import java.sql.{Connection,DriverManager,ResultSet}
    ```

1. Nous envoyons des données depuis le fichier **HVAC.csv** vers la table `hvactable`. Le fichier HVAC.csv est disponible sur le cluster à l’emplacement `/HdiSamples/HdiSamples/SensorSampleData/HVAC/`. Dans l’extrait de code suivant, nous commençons par récupérer le schéma des données à envoyer. Ensuite, nous créons un dataframe de streaming à l’aide de ce schéma. Collez l’extrait de code dans une cellule de code, puis appuyez sur **Maj+Entrée** pour l’exécuter.

    ```scala
    val userSchema = spark.read.option("header", "true").csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv").schema
    val readStreamDf = spark.readStream.schema(userSchema).csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/") 
    readStreamDf.printSchema
    ```

1. La sortie montre le schéma du fichier **HVAC.csv**. La table `hvactable` a également le même schéma. La sortie répertorie les colonnes de la table.

    ![`hdinsight Apache Spark schema table`](./media/apache-spark-connect-to-sql-database/hdinsight-schema-table.png "Schéma de table")

1. Enfin, utilisez l’extrait de code suivant pour lire des données du fichier HVAC.csv et les envoyer vers la table `hvactable` de votre base de données. Collez l’extrait de code dans une cellule de code, remplacez les valeurs d’espace réservé par les valeurs de votre base de données, puis appuyez sur **Maj+Entrée** pour l’exécuter.

    ```scala
    val WriteToSQLQuery  = readStreamDf.writeStream.foreach(new ForeachWriter[Row] {
        var connection:java.sql.Connection = _
        var statement:java.sql.Statement = _

        val jdbcUsername = "<SQL DB ADMIN USER>"
        val jdbcPassword = "<SQL DB ADMIN PWD>"
        val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
        val jdbcPort = 1433
        val jdbcDatabase ="<AZURE SQL DB NAME>"
        val driver = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
        val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"

        def open(partitionId: Long, version: Long):Boolean = {
        Class.forName(driver)
        connection = DriverManager.getConnection(jdbc_url, jdbcUsername, jdbcPassword)
        statement = connection.createStatement
        true
        }

        def process(value: Row): Unit = {
        val Date  = value(0)
        val Time = value(1)
        val TargetTemp = value(2)
        val ActualTemp = value(3)
        val System = value(4)
        val SystemAge = value(5)
        val BuildingID = value(6)  

        val valueStr = "'" + Date + "'," + "'" + Time + "'," + "'" + TargetTemp + "'," + "'" + ActualTemp + "'," + "'" + System + "'," + "'" + SystemAge + "'," + "'" + BuildingID + "'"
        statement.execute("INSERT INTO " + "dbo.hvactable" + " VALUES (" + valueStr + ")")
        }

        def close(errorOrNull: Throwable): Unit = {
        connection.close
        }
        })

    var streamingQuery = WriteToSQLQuery.start()
    ```

1. Vérifiez que les données sont diffusées vers la table `hvactable` en exécutant la requête suivante dans SQL Server Management Studio (SSMS). Chaque fois que vous exécutez la requête, elle indique un nombre croissant de lignes dans la table.

    ```sql
    SELECT COUNT(*) FROM hvactable
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser le cluster HDInsight Spark pour analyser les données dans Data Lake Storage](apache-spark-use-with-data-lake-store.md)
* [Charger des données et exécuter des requêtes sur un cluster Apache Spark dans Azure HDInsight](apache-spark-load-data-run-query.md)
* [Utiliser Apache Spark Structured Streaming avec Apache Kafka sur HDInsight](../hdinsight-apache-kafka-spark-structured-streaming.md)
