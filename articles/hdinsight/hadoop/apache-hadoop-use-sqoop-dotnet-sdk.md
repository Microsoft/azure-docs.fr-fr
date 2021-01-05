---
title: Exécuter des tâches Apache Sqoop avec .NET et HDInsight - Azure
description: Découvrez comment utiliser le kit de développement logiciel HDInsight .NET pour exécuter l’exportation et l’importation d’Apache Sqoop entre un cluster Apache Hadoop et une base de données Azure SQL.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, hdiseo17may2017, devx-track-csharp
ms.date: 01/14/2020
ms.openlocfilehash: bc4b3019cb8308ea19d86eea7bf886063ea28563
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89008282"
---
# <a name="run-apache-sqoop-jobs-by-using-net-sdk-for-apache-hadoop-in-hdinsight"></a>Exécuter des tâches Apache Sqoop avec le SDK .NET pour Apache Hadoop dans HDInsight

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Découvrez comment utiliser le kit de développement logiciel .NET Azure HDInsight pour exécuter des tâches Apache Sqoop dans HDInsight afin d’effectuer des opérations d’importation et d’exportation entre un cluster HDInsight et Azure SQL Database ou SQL Server Database.

## <a name="prerequisites"></a>Prérequis

* Avoir effectué [Configurer un environnement de test](./hdinsight-use-sqoop.md#create-cluster-and-sql-database) dans [Utiliser Apache Sqoop avec Hadoop dans HDInsight](./hdinsight-use-sqoop.md).

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/).

* Vous êtes familiarisé avec Sqoop. Pour plus d’informations, consultez le [Guide de l’utilisateur Sqoop](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html).

## <a name="use-sqoop-on-hdinsight-clusters-with-the-net-sdk"></a>Utiliser Sqoop sur des clusters HDInsight avec le Kit de développement logiciel (SDK) .NET

Le Kit de développement logiciel (SDK) .NET HDInsight fournit des bibliothèques clientes .NET facilitant l'utilisation des clusters HDInsight à partir de .NET. Dans cette section, vous allez créer une application console C# pour exporter l’`hivesampletable` vers la table Azure SQL Database que vous avez créée à partir des conditions préalables.

## <a name="set-up"></a>Configurer

1. Démarrez Visual Studio et créez une application console C#.

1. Accédez à **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**, puis exécutez la commande suivante :

    ```
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

## <a name="sqoop-export"></a>Exportation de Sqoop

De Hive vers SQL Server.  Cet exemple exporte des données à partir de la table Hive `hivesampletable` vers la table `mobiledata` dans SQL Database.

1. Utilisez le code suivant dans le fichier Program.cs. Modifiez le code pour définir les valeurs de `ExistingClusterName`et de `ExistingClusterPassword`.

    ```csharp
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;
    
    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;
    
            private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
            private const string ExistingClusterPassword = "<Cluster User Password>";
            private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";
            private const string ExistingClusterUsername = "admin";
    
            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");
    
                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
    
                SubmitSqoopJob();
    
                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }
    
            private static void SubmitSqoopJob()
            {
                var sqlDatabaseServerName = ExistingClusterName + "dbserver";
                var sqlDatabaseLogin = "sqluser";
                var sqlDatabaseLoginPassword = ExistingClusterPassword;
                var sqlDatabaseDatabaseName = ExistingClusterName + "db";
    
                // Connection string for using Azure SQL Database; Comment if using SQL Server
                var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ".database.windows.net;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
    
                // Connection string for using SQL Server; Uncomment if using SQL Server
                // var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
    
                //sqoop start
                var tableName = "mobiledata";
    
                var parameters = new SqoopJobSubmissionParameters
                {
                     Command = "export --connect " + connectionString + " --table " + tableName + " --hcatalog-table hivesampletable"
                };
                //sqoop end
    
                System.Console.WriteLine("Submitting the Sqoop job to the cluster...");
                var response = _hdiJobManagementClient.JobManagement.SubmitSqoopJob(parameters);
                System.Console.WriteLine("Validating that the response is as expected...");
                System.Console.WriteLine("Response status code is " + response.StatusCode);
                System.Console.WriteLine("Validating the response object...");
                System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
            }
        }
    }
    ```

1. Pour lancer le programme, appuyez sur la touche **F5**.

## <a name="sqoop-import"></a>Importation de Sqoop

De SQL Server à Stockage Azure. Cet exemple dépend de l’exportation ci-dessus effectuée.  Cet exemple importe des données de la table `mobiledata` dans SQL Database vers le répertoire `wasb:///tutorials/usesqoop/importeddata` sur le compte de stockage par défaut du cluster.

1. Remplacez le code ci-dessus dans le bloc `//sqoop start //sqoop end` par le code suivant :

    ```csharp
    var tableName = "mobiledata";
    var exportDir = "/tutorials/usesqoop/importeddata";
    
    var parameters = new SqoopJobSubmissionParameters
    {
        Command = "import --connect " + connectionString + " --table " + tableName + " --target-dir " +  exportDir + " --fields-terminated-by \\t --lines-terminated-by \\n -m 1"
    };
    ```

1. Pour lancer le programme, appuyez sur la touche **F5**.

## <a name="limitations"></a>Limites

HDInsight basé sur Linux présente les limitations suivantes :

* Exportation en bloc : le connecteur Sqoop utilisé pour exporter des données vers Microsoft SQL Server ou Azure SQL Database ne prend actuellement pas en charge les insertions en bloc.

* Traitement par lot : lorsque vous utilisez le commutateur `-batch`, Sqoop effectue plusieurs insertions au lieu de traiter par lot les opérations d’insertion.

## <a name="next-steps"></a>Étapes suivantes

Vous maîtrisez à présent l’utilisation de Sqoop. Pour plus d'informations, consultez les rubriques suivantes :

* [Utiliser Apache Oozie avec HDInsight](../hdinsight-use-oozie-linux-mac.md) : Utilisez l’action Sqoop dans un workflow Oozie.
* [Chargez des données dans HDInsight](../hdinsight-upload-data.md) : découvrez d’autres méthodes pour charger des données dans HDInsight ou le stockage Blob Azure.
