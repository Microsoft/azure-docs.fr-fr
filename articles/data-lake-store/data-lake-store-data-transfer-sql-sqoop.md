---
title: Copier des données entre Data Lake Storage Gen1 et Azure SQL - Sqoop | Microsoft Docs
description: Utiliser Sqoop pour copier des données entre Azure SQL Database et Azure Data Lake Storage Gen1
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 07/30/2019
ms.author: twooley
ms.openlocfilehash: 9bb787138267fd8a9fab4dea233c1c828b457d67
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109185"
---
# <a name="copy-data-between-data-lake-storage-gen1-and-azure-sql-database-using-sqoop"></a>Copier des données entre Data Lake Storage Gen1 et Azure SQL Database à l’aide de Sqoop

Découvrez comment utiliser Apache Sqoop pour importer et exporter des données entre Azure SQL Database et Azure Data Lake Storage Gen1.

## <a name="what-is-sqoop"></a>Qu'est-ce que Sqoop ?

Les applications Big Data sont un choix naturel pour le traitement des données non structurées et semi-structurées, telles que les journaux d’activité et les fichiers. Toutefois, il vous faudra peut-être également traiter des données structurées stockées dans des bases de données relationnelles.

[Apache Sqoop](https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html) est un outil conçu pour transférer des données entre des bases de données relationnelles et un référentiel Big Data, par exemple Azure Data Lake Storage Gen1. Vous pouvez l’utiliser pour importer des données à partir d’un système de gestion de base de données relationnelle (SGBDR), comme Azure SQL Database, dans Azure Data Lake Storage Gen1. Vous pouvez ensuite transformer et analyser les données à l’aide de charges de travail Big Data, puis exporter les données dans un SGBDR. Dans cet article, vous utilisez une base de données dans Azure SQL Database comme base de données relationnelle pour les opérations d’importation et d’exportation.

## <a name="prerequisites"></a>Prérequis

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen1**. Pour savoir comment créer le compte, consultez [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* **Cluster Azure HDInsight** avec accès à un compte Data Lake Storage Gen1. Voir [Créer un cluster HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Cet article suppose que vous disposez d’un cluster Linux HDInsight avec accès à Azure Data Lake Storage Gen1.
* **Azure SQL Database**. Pour obtenir des instructions sur la création d’une base de données dans Azure SQL Database, consultez [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).

## <a name="create-sample-tables-in-the-database"></a>Créer des exemples de tables dans la base de données

1. Pour commencer, créez deux exemples de tables dans la base de données. Utilisez [SQL Server Management Studio](../azure-sql/database/connect-query-ssms.md) ou Visual Studio pour vous connecter à la base de données, puis exécutez les requêtes suivantes.

    **Créer Table1**

    ```tsql
    CREATE TABLE [dbo].[Table1](
    [ID] [int] NOT NULL,
    [FName] [nvarchar](50) NOT NULL,
    [LName] [nvarchar](50) NOT NULL,
     CONSTRAINT [PK_Table_1] PRIMARY KEY CLUSTERED
           (
                  [ID] ASC
           )
    ) ON [PRIMARY]
    GO
    ```

    **Créer Table2**

    ```tsql
    CREATE TABLE [dbo].[Table2](
    [ID] [int] NOT NULL,
    [FName] [nvarchar](50) NOT NULL,
    [LName] [nvarchar](50) NOT NULL,
     CONSTRAINT [PK_Table_2] PRIMARY KEY CLUSTERED
           (
                  [ID] ASC
           )
    ) ON [PRIMARY]
    GO
    ```

1. Exécutez la commande suivante pour ajouter des exemples de données à **Table1**. Laissez **Table2** vide. Plus tard, vous importerez des données de **Table1** dans Azure Data Lake Storage Gen1. Ensuite, vous exporterez les données d’Azure Data Lake Storage Gen1 vers **Table2**.

    ```tsql
    INSERT INTO [dbo].[Table1] VALUES (1,'Neal','Kell'), (2,'Lila','Fulton'), (3, 'Erna','Myers'), (4,'Annette','Simpson');
    ```

## <a name="use-sqoop-from-an-hdinsight-cluster-with-access-to-data-lake-storage-gen1"></a>Utiliser Sqoop à partir d’un cluster HDInsight avec accès à Azure Data Lake Storage Gen1

Un cluster HDInsight dispose déjà des packages Sqoop. Si vous avez configuré le cluster HDInsight pour utiliser Data Lake Storage Gen1 comme un espace de stockage supplémentaire, vous pouvez utiliser Sqoop (sans aucune modification de configuration) pour importer/exporter des données entre une base de données relationnelle telle qu’Azure SQL Database et un compte Data Lake Storage Gen1.

1. Pour les besoins de cet article, nous supposons que vous avez créé un cluster Linux ; vous devez donc utiliser SSH pour vous connecter au cluster. Consultez [Connexion à un cluster HDInsight sous Linux](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

1. Vérifiez si vous pouvez accéder au compte Data Lake Storage Gen1 à partir du cluster. Exécutez la commande suivante depuis une invite SSH :

    ```console
    hdfs dfs -ls adl://<data_lake_storage_gen1_account>.azuredatalakestore.net/
    ```

   Cette commande fournit une liste des fichiers/dossiers du compte Data Lake Storage Gen1.

### <a name="import-data-from-azure-sql-database-into-data-lake-storage-gen1"></a>Importer des données à partir d’Azure SQL Database dans Data Lake Storage Gen1

1. Accédez au répertoire dans lequel se trouvent les packages Sqoop. En général, cet emplacement est `/usr/hdp/<version>/sqoop/bin`.

1. Importez les données de **Table1** dans le compte Data Lake Storage Gen1. Utilisez la syntaxe suivante :

    ```console
    sqoop-import --connect "jdbc:sqlserver://<sql-database-server-name>.database.windows.net:1433;username=<username>@<sql-database-server-name>;password=<password>;database=<sql-database-name>" --table Table1 --target-dir adl://<data-lake-storage-gen1-name>.azuredatalakestore.net/Sqoop/SqoopImportTable1
    ```

   L’espace réservé **sql-database-server-name** représente le nom du serveur sur lequel s’exécute la base de données. L’espace réservé **sql-database-name** désigne le nom de la base de données.

   Par exemple,

    ```console
    sqoop-import --connect "jdbc:sqlserver://mysqoopserver.database.windows.net:1433;username=user1@mysqoopserver;password=<password>;database=mysqoopdatabase" --table Table1 --target-dir adl://myadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1
    ```

1. Vérifiez que les données ont été transférées vers le compte Data Lake Storage Gen1. Exécutez la commande suivante :

    ```console
    hdfs dfs -ls adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/
    ```

   La sortie suivante s'affiche.

    ```console
    -rwxrwxrwx   0 sshuser hdfs          0 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/_SUCCESS
    -rwxrwxrwx   0 sshuser hdfs         12 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00000
    -rwxrwxrwx   0 sshuser hdfs         14 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00001
    -rwxrwxrwx   0 sshuser hdfs         13 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00002
    -rwxrwxrwx   0 sshuser hdfs         18 2016-02-26 21:09 adl://hdiadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1/part-m-00003
    ```

   Chaque fichier **part-m-*** correspond à une ligne dans la table source **Table1**.

### <a name="export-data-from-data-lake-storage-gen1-into-azure-sql-database"></a>Vous pouvez afficher le contenu des fichiers part-m-* à vérifier.

1. Exporter les données de Data Lake Storage Gen1 dans Azure SQL Database Exportez les données du compte Data Lake Storage Gen1 vers la table vide, **Table2**, dans la base de données Azure SQL.

    ```console
    sqoop-export --connect "jdbc:sqlserver://<sql-database-server-name>.database.windows.net:1433;username=<username>@<sql-database-server-name>;password=<password>;database=<sql-database-name>" --table Table2 --export-dir adl://<data-lake-storage-gen1-name>.azuredatalakestore.net/Sqoop/SqoopImportTable1 --input-fields-terminated-by ","
    ```

   Utilisez la syntaxe suivante.

    ```console
    sqoop-export --connect "jdbc:sqlserver://mysqoopserver.database.windows.net:1433;username=user1@mysqoopserver;password=<password>;database=mysqoopdatabase" --table Table2 --export-dir adl://myadlsg1store.azuredatalakestore.net/Sqoop/SqoopImportTable1 --input-fields-terminated-by ","
    ```

1. Par exemple, Vérifiez que les données ont été chargées sur la table de la base de données SQL.

    ```tsql
    SELECT * FROM TABLE2
    ```

   Utilisez [SQL Server Management Studio](../azure-sql/database/connect-query-ssms.md) ou Visual Studio pour vous connecter à la base de données Azure SQL, puis exécutez la requête suivante.

    ```output
     ID  FName    LName
    -------------------
    1    Neal     Kell
    2    Lila     Fulton
    3    Erna     Myers
    4    Annette  Simpson
    ```

## <a name="performance-considerations-while-using-sqoop"></a>Cette commande doit produire le résultat suivant.

Considérations de performances lors de l’utilisation de Sqoop

## <a name="next-steps"></a>Pour plus d’informations sur les performances d’optimisation de votre tâche Sqoop pour copier des données vers Data Lake Storage Gen1, consultez le [billet de blog sur les performances Sqoop](/archive/blogs/shanyu/performance-tuning-for-hdinsight-storm-and-microsoft-azure-eventhubs).

* Étapes suivantes
* [Copier des données d’objets blob Stockage Azure vers Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)