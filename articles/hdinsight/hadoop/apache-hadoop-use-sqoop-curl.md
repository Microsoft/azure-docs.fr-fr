---
title: Utilisez Curl pour exporter des données avec Apache Sqoop dans Azure HDInsight
description: Découvrez comment transmettre à distance des tâches Apache Sqoop vers Azure HDInsight à l’aide de Curl.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/06/2020
ms.openlocfilehash: 9104be9975568c52f6a96994a0afb782a406fe4e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86076264"
---
# <a name="run-apache-sqoop-jobs-in-hdinsight-with-curl"></a>Exécuter des tâches Apache Sqoop dans HDInsight avec Curl

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Apprenez à utiliser Curl pour exécuter des tâches Apache Sqoop sur un cluster Apache Hadoop dans HDInsight. Cet article montre comment exporter des données à partir du stockage Azure et les importer dans une base de données SQL Server à l’aide de Curl. Cet article est la suite de [Utiliser Apache Sqoop avec Hadoop dans HDInsight](./hdinsight-use-sqoop.md).

Curl est utilisé pour illustrer comment interagir avec HDInsight en utilisant des demandes HTTP brutes pour exécuter, analyser et récupérer des travaux Sqoop. Cela fonctionne à l’aide de l’API REST WebHCat (anciennement Templeton) fournie par votre cluster HDInsight.

## <a name="prerequisites"></a>Prérequis

* Avoir effectué [Configurer un environnement de test](./hdinsight-use-sqoop.md#create-cluster-and-sql-database) dans [Utiliser Apache Sqoop avec Hadoop dans HDInsight](./hdinsight-use-sqoop.md).

* Un client pour interroger Azure SQL Database. Envisagez d’utiliser [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) ou [Visual Studio Code](../../azure-sql/database/connect-query-vscode.md).

* [Curl](https://curl.haxx.se/). Curl est un outil pour transférer des données depuis ou vers un cluster HDInsight.

* [jq](https://stedolan.github.io/jq/). L’utilitaire jq est utilisé pour traiter les données JSON renvoyées à partir de demandes REST.

* Vous êtes familiarisé avec Sqoop. Pour plus d’informations, consultez le [Guide de l’utilisateur Sqoop](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html).

## <a name="submit-apache-sqoop-jobs-by-using-curl"></a>Envoyer les travaux Apache Sqoop avec Curl

Utilisez Curl pour exporter des données à l’aide d’Apache Sqoop à partir du stockage Azure pour SQL Server.

> [!NOTE]  
> Lorsque vous utilisez Curl ou toute autre communication REST avec WebHCat, vous devez authentifier les demandes en fournissant le nom d'utilisateur et le mot de passe de l'administrateur du cluster HDInsight. Vous devez également utiliser le nom du cluster dans l’URI (Uniform Resource Identifier) utilisé pour envoyer les demandes au serveur.

Pour les commandes de cette section, remplacez `USERNAME` par l’utilisateur à authentifier sur le cluster et `PASSWORD` par le mot de passe du compte d’utilisateur. Remplacez `CLUSTERNAME` par le nom de votre cluster.

L’API REST est sécurisée à l’aide de l’ [authentification de base](https://en.wikipedia.org/wiki/Basic_access_authentication). Vous devez toujours effectuer les demandes à l’aide du protocole Secure HTTP (HTTPS) pour aider à vous assurer que vos informations d’identification sont envoyées en toute sécurité sur le serveur.

1. Pour faciliter l’utilisation, définissez les variables ci-dessous. Cet exemple est fondé sur un environnement Windows ; vous devez le modifier si nécessaire pour votre environnement.

    ```cmd
    set CLUSTERNAME=
    set USERNAME=admin
    set PASSWORD=
    set SQLDATABASESERVERNAME=
    set SQLDATABASENAME=
    set SQLPASSWORD=
    set SQLUSER=sqluser
    ```

1. À partir d’une ligne de commande, exécutez la commande suivante pour vérifier que vous pouvez vous connecter à votre cluster HDInsight.

    ```cmd
    curl -u %USERNAME%:%PASSWORD% -G https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/status
    ```

    Vous devez recevoir une réponse ayant l'aspect suivant :

    ```json
    {"status":"ok","version":"v1"}
    ```

1. Pour envoyer un travail Sqoop, utilisez la commande suivante :

    ```cmd
    curl -u %USERNAME%:%PASSWORD% -d user.name=%USERNAME% -d command="export --connect jdbc:sqlserver://%SQLDATABASESERVERNAME%.database.windows.net;user=%SQLUSER%@%SQLDATABASESERVERNAME%;password=%PASSWORD%;database=%SQLDATABASENAME% --table log4jlogs --export-dir /example/data/sample.log --input-fields-terminated-by \0x20 -m 1" -d statusdir="wasb:///example/data/sqoop/curl" https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/sqoop
    ```

    Les paramètres utilisés dans cette commande sont les suivants :

   * **-d** : étant donné que `-G` n’est pas utilisé, la requête correspond par défaut à la méthode POST. `-d` spécifie les valeurs de données envoyées avec la demande.

       * **user.name** : l’utilisateur qui exécute la commande.

       * **command** : commande Sqoop à exécuter.

       * **statusdir** : le répertoire où seront enregistrés les statuts de cette tâche.

     Cette commande va retourner un ID de tâche qui peut être utilisé pour vérifier l’état de la tâche.

       ```json
       {"id":"job_1415651640909_0026"}
       ```

1. Pour vérifier le statut de la tâche, utilisez la commande suivante. Remplacez `JOBID` par la valeur renvoyée à l’étape précédente. Par exemple, si la valeur de retour était `{"id":"job_1415651640909_0026"}`, alors `JOBID` devrait être `job_1415651640909_0026`. Modifiez l’emplacement de `jq` en fonction de vos besoins.

    ```cmd
    set JOBID=job_1415651640909_0026

    curl -G -u %USERNAME%:%PASSWORD% -d user.name=%USERNAME% https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/jobs/%JOBID% | C:\HDI\jq-win64.exe .status.state
    ```

    Si le travail est terminé, l’état est **TERMINÉ**.

   > [!NOTE]  
   > Cette demande Curl retourne un document JSON (JavaScript Object Notation) avec des informations sur la tâche ; jq est utilisé pour récupérer uniquement la valeur de statut.

1. Une fois que le statut de la tâche est passé à **TERMINÉ**, vous pouvez récupérer les résultats depuis le stockage blob Azure. Le paramètre `statusdir` transmis avec la requête contient l’emplacement du fichier de sortie. Dans notre cas `wasb:///example/data/sqoop/curl`. Cette adresse stocke la sortie de la tâche dans le répertoire `example/data/sqoop/curl` sur le conteneur de stockage par défaut utilisé par votre cluster HDInsight.

    Vous pouvez utiliser le portail Azure pour accéder aux objets BLOB stderr et stdout.

1. Pour vérifier que les données ont été exportées, utilisez les requêtes suivantes à partir de votre client SQL pour voir les données exportées :

    ```sql
    SELECT COUNT(*) FROM [dbo].[log4jlogs] WITH (NOLOCK);
    SELECT TOP(25) * FROM [dbo].[log4jlogs] WITH (NOLOCK);
    ```

## <a name="limitations"></a>Limites

* Exportation en bloc : Avec HDInsight sous Linux, le connecteur Sqoop utilisé pour exporter des données vers Microsoft SQL Server ou Azure SQL Database ne prend pas en charge les insertions en bloc.
* Traitement par lots : avec HDInsight sous Linux, lorsque vous utilisez le commutateur `-batch` pour effectuer des insertions, Sqoop effectue plusieurs insertions plutôt qu’un traitement par lots des opérations d’insertion.

## <a name="summary"></a>Résumé

Comme illustré dans ce document, vous pouvez utiliser une demande HTTP brute pour exécuter, surveiller et afficher les résultats des travaux Sqoop sur votre cluster HDInsight.

Pour plus d’informations sur l’interface REST utilisée dans cet article, consultez le <a href="https://sqoop.apache.org/docs/1.99.3/RESTAPI.html" target="_blank">Guide de l’API REST Apache Sqoop</a>.

## <a name="next-steps"></a>Étapes suivantes

[Utiliser Apache Sqoop avec Apache Hadoop sur HDInsight](hdinsight-use-sqoop.md)

Pour d’autres articles HDInsight impliquant curl :

* [Créer des clusters Apache Hadoop à l’aide de l’API REST Azure](../hdinsight-hadoop-create-linux-clusters-curl-rest.md)
* [Exécuter des requêtes Apache Hive avec Apache Hadoop dans HDInsight à l’aide de REST](apache-hadoop-use-hive-curl.md)
* [Exécuter des tâches MapReduce avec Apache Hadoop sur HDInsight à l’aide de REST](apache-hadoop-use-mapreduce-curl.md)
