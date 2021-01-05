---
title: Apache Sqoop avec Apache Hadoop - Azure HDInsight
description: Apprenez à utiliser Apache Sqoop pour échanger des données entre Apache Hadoop sur HDInsight et Azure SQL Database.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 11/28/2019
ms.openlocfilehash: 6cd522192c498d628490430c46c18c8ae08ad2e2
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547008"
---
# <a name="use-apache-sqoop-to-import-and-export-data-between-apache-hadoop-on-hdinsight-and-azure-sql-database"></a>Utiliser Apache Sqoop pour échanger des données entre Apache Hadoop sur HDInsight et Azure SQL Database

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Apprenez à utiliser Apache Sqoop pour échanger des données entre un cluster Apache Hadoop dans Azure HDInsight et Azure SQL Database ou Microsoft SQL Server. Les étapes décrites dans ce document utilisent la commande `sqoop` directement à partir du nœud principal du cluster Hadoop. Vous utilisez SSH pour vous connecter au nœud principal et exécutez les commandes décrites dans ce document. Cet article est la suite de [Utiliser Apache Sqoop avec Hadoop dans HDInsight](./hdinsight-use-sqoop.md).

## <a name="prerequisites"></a>Prérequis

* Avoir effectué [Configurer un environnement de test](./hdinsight-use-sqoop.md#create-cluster-and-sql-database) dans [Utiliser Apache Sqoop avec Hadoop dans HDInsight](./hdinsight-use-sqoop.md).

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Vous êtes familiarisé avec Sqoop. Pour plus d’informations, consultez le [Guide de l’utilisateur Sqoop](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html).

## <a name="set-up"></a>Configurer

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Pour faciliter l’utilisation, définissez des variables. Remplacez `PASSWORD`, `MYSQLSERVER`et `MYDATABASE` par les valeurs pertinentes, puis entrez les commandes suivantes :

    ```bash
    export password='PASSWORD'
    export sqlserver="MYSQLSERVER"
    export database="MYDATABASE"


    export serverConnect="jdbc:sqlserver://$sqlserver.database.windows.net:1433;user=sqluser;password=$password"
    export serverDbConnect="jdbc:sqlserver://$sqlserver.database.windows.net:1433;user=sqluser;password=$password;database=$database"
    ```

## <a name="sqoop-export"></a>Exportation de Sqoop

De Hive vers SQL.

1. Pour vérifier que Sqoop peut voir votre base de données, entrez la commande ci-dessous dans votre connexion SSH ouverte. Cette commande retourne une liste de bases de données.

    ```bash
    sqoop list-databases --connect $serverConnect
    ```

1. Entrez la commande suivante pour afficher une liste des tables de la base de données spécifiée :

    ```bash
    sqoop list-tables --connect $serverDbConnect
    ```

1. Pour exporter des données depuis la table Hive `hivesampletable` vers la table `mobiledata` dans votre base de données, entrez la commande ci-dessous dans votre connexion SSH ouverte :

    ```bash
    sqoop export --connect $serverDbConnect \
    -table mobiledata \
    --hcatalog-table hivesampletable
    ```

1. Pour vérifier que les données ont été exportées, exécutez les requêtes suivantes à partir de votre connexion SSH pour voir les données exportées :

    ```bash
    sqoop eval --connect $serverDbConnect \
    --query "SELECT COUNT(*) from dbo.mobiledata WITH (NOLOCK)"


    sqoop eval --connect $serverDbConnect \
    --query "SELECT TOP(10) * from dbo.mobiledata WITH (NOLOCK)"
    ```

## <a name="sqoop-import"></a>Importation de Sqoop

De SQL vers le stockage Azure.

1. Entrez la commande ci-dessous dans votre connexion SSH ouverte pour importer des données de la table `mobiledata` dans SQL vers le répertoire `wasbs:///tutorials/usesqoop/importeddata` sur HDInsight. Les champs dans les données sont séparés par un caractère de tabulation et les lignes se terminent par un caractère de nouvelle ligne.

    ```bash
    sqoop import --connect $serverDbConnect \
    --table mobiledata \
    --target-dir 'wasb:///tutorials/usesqoop/importeddata' \
    --fields-terminated-by '\t' \
    --lines-terminated-by '\n' -m 1
    ```

1. Vous pouvez aussi spécifier une table Hive :

    ```bash
    sqoop import --connect $serverDbConnect \
    --table mobiledata \
    --target-dir 'wasb:///tutorials/usesqoop/importeddata2' \
    --fields-terminated-by '\t' \
    --lines-terminated-by '\n' \
    --create-hive-table \
    --hive-table mobiledata_imported2 \
    --hive-import -m 1
    ```

1. Une fois l’importation terminée, entrez la commande suivante dans votre connexion SSH ouverte pour lister les données dans le nouveau répertoire :

    ```bash
    hadoop fs -tail /tutorials/usesqoop/importeddata/part-m-00000
    ```

1. Utilisez [beeline](./apache-hadoop-use-hive-beeline.md) pour vérifier que la table a été créée dans Hive.

    1. Se connecter

        ```bash
        beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
        ```

    1. Exécutez les requêtes suivantes l’une après l’autre et examinez la sortie :

        ```hql
        show tables;
        describe mobiledata_imported2;
        SELECT COUNT(*) FROM mobiledata_imported2;
        SELECT * FROM mobiledata_imported2 LIMIT 10;
        ```

    1. Quittez beeline en exécutant `!exit`.

## <a name="limitations"></a>Limites

* Exportation en bloc : avec HDInsight sous Linux, le connecteur Sqoop utilisé pour exporter des données vers SQL ne prend pas en charge les insertions en bloc.

* Traitement par lots : avec HDInsight sous Linux, lorsque vous utilisez le commutateur `-batch` pour effectuer des insertions, Sqoop effectue plusieurs insertions plutôt qu’un traitement par lots des opérations d’insertion.

## <a name="important-considerations"></a>Points importants à prendre en compte

* HDInsight et SQL Server doivent figurer sur le même réseau Azure Virtual Network.

    Pour voir un exemple, consultez le document [Connecter HDInsight à votre réseau local](./../connect-on-premises-network.md).

    Pour plus d’informations sur l’utilisation de HDInsight avec Azure Virtual Network, voir le document [Étendre HDInsight avec Azure Virtual Network](../hdinsight-plan-virtual-network-deployment.md). Pour plus d’informations sur Azure Virtual Network, voir [Présentation du réseau virtuel](../../virtual-network/virtual-networks-overview.md).

* Le serveur SQL Server doit également être configuré pour autoriser l’authentification SQL. Pour plus d’informations, consultez le document [Choisir un mode d’authentification](/sql/relational-databases/security/choose-an-authentication-mode).

* Vous devez configurer SQL Server pour qu’il accepte les connexions à distance. Pour plus d’informations, consultez le document [Résolution des problèmes de connexion au moteur de base de données SQL Server](https://social.technet.microsoft.com/wiki/contents/articles/2102.how-to-troubleshoot-connecting-to-the-sql-server-database-engine.aspx).

## <a name="next-steps"></a>Étapes suivantes

Vous maîtrisez à présent l’utilisation de Sqoop. Pour plus d'informations, consultez les rubriques suivantes :

* [Utiliser Apache Oozie avec HDInsight](../hdinsight-use-oozie-linux-mac.md) : Utilisez l’action Sqoop dans un workflow Oozie.
* [Analyser des données sur les retards de vol avec HDInsight](../interactive-query/interactive-query-tutorial-analyze-flight-data.md) : Utilisez Interactive Query pour analyser des données sur les retards de vol, puis utilisez Sqoop pour exporter ces données vers une base de données dans Azure.
* [Chargez des données dans HDInsight](../hdinsight-upload-data.md) : Découvrez d’autres méthodes pour charger des données dans HDInsight ou le stockage Blob Azure.