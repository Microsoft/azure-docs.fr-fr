---
title: 'Tutoriel : Opérations ETL avec Interactive Query - Azure HDInsight'
description: 'Tutoriel : Découvrir comment extraire des données d’un jeu de données CSV brut. Transformez-le avec Interactive Query sur HDInsight. Chargez ensuite les données transformées dans Azure SQL Database à l’aide d’Apache Sqoop.'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,mvc
ms.date: 07/02/2019
ms.openlocfilehash: 730a6bfa627eafcab799fc811db4e20a1d4cec48
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92534581"
---
# <a name="tutorial-extract-transform-and-load-data-using-interactive-query-in-azure-hdinsight"></a>Tutoriel : Extraire, transformer et charger des données à l’aide d’Interactive Query sur Azure HDInsight

Dans ce tutoriel, vous téléchargez un fichier de données brutes CSV de données de vol disponibles publiquement. Importez-le dans le stockage de cluster HDInsight, puis transformez les données avec Interactive Query dans Azure HDInsight. Une fois les données transformées, chargez-les dans une base de données dans Azure SQL Database à l’aide d’[Apache Sqoop](https://sqoop.apache.org/).

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Télécharger l’exemple des données de vol
> * Charger des données vers un cluster HDInsight
> * Transformer les données à l’aide d’Interactive Query
> * Créer une table dans une base de données dans Azure SQL Database
> * Utiliser Sqoop pour exporter des données vers une base de données Azure SQL Database

## <a name="prerequisites"></a>Prérequis

* Un cluster Interactive Query sur HDInsight. Consultez [Création clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **Interactive Query** pour le **Type de cluster** .

* base de données Azure SQL Database. Vous utilisez la base de données en tant que magasin de données de destination. Si vous n’avez pas de base de données dans Azure SQL Database, consultez [Créer une base de données dans Azure SQL Database dans le portail Azure](../../azure-sql/database/single-database-create-quickstart.md).

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="download-the-flight-data"></a>Téléchargement des données de vol

1. Accédez à [Research and Innovative Technology Administration, Bureau of Transportation Statistics](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time).

2. Dans la page, effacez tous les champs, puis sélectionnez les valeurs suivantes :

   | Nom | Valeur |
   | --- | --- |
   | Filtre année |2019 |
   | Filtre période |Janvier |
   | Champs |`Year, FlightDate, Reporting_Airline, DOT_ID_Reporting_Airline, Flight_Number_Reporting_Airline, OriginAirportID, Origin, OriginCityName, OriginState, DestAirportID, Dest, DestCityName, DestState, DepDelayMinutes, ArrDelay, ArrDelayMinutes, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay`. |

3. Sélectionnez **Télécharger** . Vous obtenez un fichier .zip avec les champs de données que vous avez sélectionnés.

## <a name="upload-data-to-an-hdinsight-cluster"></a>Charger des données vers un cluster HDInsight

De nombreuses méthodes permettent de charger des données vers l’espace de stockage associé à un cluster HDInsight. Dans cette section, vous utilisez `scp` pour charger des données. Pour en savoir plus sur les autres façons de charger des données, consultez [Charger des données dans HDInsight](../hdinsight-upload-data.md).

1. Chargez le fichier .zip sur le nœud principal du cluster HDInsight. Modifiez la commande ci-dessous en remplaçant `FILENAME` par le nom du fichier .zip et `CLUSTERNAME` par le nom du cluster HDInsight. Ensuite, ouvrez une invite de commandes, définissez votre répertoire de travail sur l’emplacement du fichier, puis entrez la commande.

    ```cmd
    scp FILENAME.zip sshuser@CLUSTERNAME-ssh.azurehdinsight.net:FILENAME.zip
    ```

    Si vous y êtes invité, entrez oui ou non pour continuer. Le texte n’est pas visible dans la fenêtre à mesure que vous l’entrez.

2. À la fin du chargement, connectez-vous au cluster par l’intermédiaire de SSH. Modifiez la commande ci-dessous en remplaçant `CLUSTERNAME` par le nom du cluster HDInsight. Puis, entrez la commande suivante :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Configurez la variable d’environnement une fois qu’une connexion SSH a été établie. Remplacez `FILE_NAME`, `SQL_SERVERNAME`, `SQL_DATABASE`, `SQL_USER` et `SQL_PASWORD` par les valeurs appropriées. Ensuite, entrez la commande :

    ```bash
    export FILENAME=FILE_NAME
    export SQLSERVERNAME=SQL_SERVERNAME
    export DATABASE=SQL_DATABASE
    export SQLUSER=SQL_USER
    export SQLPASWORD='SQL_PASWORD'
    ```

4. Décompressez le fichier .zip en entrant la commande ci-dessous :

    ```bash
    unzip $FILENAME.zip
    ```

5. Créez un répertoire de stockage HDInsight, puis copiez le fichier .csv dans le répertoire en entrant la commande ci-dessous :

    ```bash
    hdfs dfs -mkdir -p /tutorials/flightdelays/data
    hdfs dfs -put $FILENAME.csv /tutorials/flightdelays/data/
    ```

## <a name="transform-data-using-a-hive-query"></a>Transformer des données à l’aide d’une requête Hive

De nombreuses méthodes permettent d’exécuter un travail Hive sur un cluster HDInsight. Dans cette section, vous allez utiliser [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline%E2%80%93CommandLineShell) pour exécuter un travail Hive. Pour plus d’informations sur les autres méthodes d’exécution d’un travail Hive, consultez [Utilisation d’Apache Hive sur HDInsight](../hadoop/hdinsight-use-hive.md).

Dans le cadre du travail Hive, vous allez importer les données du fichier .csv dans une table Hive nommée **Delays** .

1. À partir de l’invite SSH que vous avez déjà pour le cluster HDInsight, utilisez la commande suivante pour créer et modifier un fichier nommé **flightdelays.hql**  :

    ```bash
    nano flightdelays.hql
    ```

2. Utilisez le texte suivant comme contenu de ce fichier :

    ```hiveql
    DROP TABLE delays_raw;
    -- Creates an external table over the csv file
    CREATE EXTERNAL TABLE delays_raw (
        YEAR string,
        FL_DATE string,
        UNIQUE_CARRIER string,
        CARRIER string,
        FL_NUM string,
        ORIGIN_AIRPORT_ID string,
        ORIGIN string,
        ORIGIN_CITY_NAME string,
        ORIGIN_CITY_NAME_TEMP string,
        ORIGIN_STATE_ABR string,
        DEST_AIRPORT_ID string,
        DEST string,
        DEST_CITY_NAME string,
        DEST_CITY_NAME_TEMP string,
        DEST_STATE_ABR string,
        DEP_DELAY_NEW float,
        ARR_DELAY_NEW float,
        CARRIER_DELAY float,
        WEATHER_DELAY float,
        NAS_DELAY float,
        SECURITY_DELAY float,
        LATE_AIRCRAFT_DELAY float)
    -- The following lines describe the format and location of the file
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE
    LOCATION '/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays AS
    SELECT YEAR AS year,
        FL_DATE AS flight_date,
        substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier,
        substring(CARRIER, 2, length(CARRIER) -1) AS carrier,
        substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num,
        ORIGIN_AIRPORT_ID AS origin_airport_id,
        substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code,
        substring(ORIGIN_CITY_NAME, 2) AS origin_city_name,
        substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr,
        DEST_AIRPORT_ID AS dest_airport_id,
        substring(DEST, 2, length(DEST) -1) AS dest_airport_code,
        substring(DEST_CITY_NAME,2) AS dest_city_name,
        substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr,
        DEP_DELAY_NEW AS dep_delay_new,
        ARR_DELAY_NEW AS arr_delay_new,
        CARRIER_DELAY AS carrier_delay,
        WEATHER_DELAY AS weather_delay,
        NAS_DELAY AS nas_delay,
        SECURITY_DELAY AS security_delay,
        LATE_AIRCRAFT_DELAY AS late_aircraft_delay
    FROM delays_raw;
    ```

3. Pour enregistrer le fichier, appuyez successivement sur **Ctrl + X** , **y** et Entrée.

4. Pour démarrer Hive et exécuter le fichier **flightdelays.hql** , utilisez la commande suivante :

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
    ```

5. Dès l’exécution du script **flightdelays.hql** terminée, utilisez la commande suivante pour ouvrir une session Beeline interactive :

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

6. Lorsque vous recevez l’invite `jdbc:hive2://localhost:10001/>`, utilisez la requête suivante pour récupérer des données à partir des données relatives aux retards de vol qui ont été importées :

    ```hiveql
    INSERT OVERWRITE DIRECTORY '/tutorials/flightdelays/output'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    SELECT regexp_replace(origin_city_name, '''', ''),
        avg(weather_delay)
    FROM delays
    WHERE weather_delay IS NOT NULL
    GROUP BY origin_city_name;
    ```

    Cette requête récupère la liste des villes qui ont enregistré des retards liés aux conditions météo, ainsi que le temps de retard moyen, et l’enregistre dans `/tutorials/flightdelays/output`. Sqoop lit ensuite les données à partir de cet emplacement et les exporte vers Azure SQL Database.

7. Pour quitter Beeline, entrez `!quit` à l’invite de commandes.

## <a name="create-a-sql-database-table"></a>Créer une table de base de données SQL

Il existe de nombreuses façons de se connecter à la base de données SQL et de créer une table. Les étapes suivantes utilisent [FreeTDS](http://www.freetds.org/) à partir du cluster HDInsight.

1. Pour installer FreeTDS, utilisez la commande suivante à partir de la connexion SSH ouverte avec le cluster :

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

2. Dès l’installation terminée, utilisez la commande suivante pour vous connecter à SQL Database.

    ```bash
    TDSVER=8.0 tsql -H $SQLSERVERNAME.database.windows.net -U $SQLUSER -p 1433 -D $DATABASE -P $SQLPASWORD
    ```

    Le résultat ressemble au texte suivant :

    ```
    locale is "en_US.UTF-8"
    locale charset is "UTF-8"
    using default charset "UTF-8"
    Default database being set to <yourdatabase>
    1>
    ```

3. À l’invite de commandes `1>` , entrez les lignes suivantes :

    ```hiveql
    CREATE TABLE [dbo].[delays](
    [origin_city_name] [nvarchar](50) NOT NULL,
    [weather_delay] float,
    CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED
    ([origin_city_name] ASC))
    GO
    ```

    Une fois l’instruction `GO` entrée, les instructions précédentes sont évaluées. Cette instruction crée une table nommée **delays** avec un index cluster.

    Utilisez la requête suivante pour vérifier que la table a été créée :

    ```hiveql
    SELECT * FROM information_schema.tables
    GO
    ```

    Le résultat ressemble au texte suivant :

    ```
    TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
    databaseName       dbo             delays        BASE TABLE
    ```

4. Entrée `exit` at the `1>` .

## <a name="export-data-to-sql-database-using-apache-sqoop"></a>Exporter des données vers SQL Database à l’aide d’Apache Sqoop

Dans les sections précédentes, vous avez copié les données transformées dans `/tutorials/flightdelays/output`. Dans cette section, vous allez utiliser Sqoop pour exporter les données de `/tutorials/flightdelays/output` vers la table que vous avez créée dans Azure SQL Database.

1. Pour vérifier que Sqoop peut voir votre base de données SQL, entrez la commande ci-dessous :

    ```bash
    sqoop list-databases --connect jdbc:sqlserver://$SQLSERVERNAME.database.windows.net:1433 --username $SQLUSER --password $SQLPASWORD
    ```

    Cette commande retourne une liste de bases de données, dont la base de données dans laquelle vous avez créé précédemment la table `delays`.

2. Pour exporter des données depuis `/tutorials/flightdelays/output` vers la table `delays`, entrez la commande ci-dessous :

    ```bash
    sqoop export --connect "jdbc:sqlserver://$SQLSERVERNAME.database.windows.net:1433;database=$DATABASE" --username $SQLUSER --password $SQLPASWORD --table 'delays' --export-dir '/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
    ```

    Sqoop se connecte à la base de données qui contient la table `delays` et exporte les données du répertoire `/tutorials/flightdelays/output` vers la table `delays`.

3. Après l’exécution de la commande sqoop, connectez-vous à la base de données à l’aide de l’utilitaire tsql en entrant la commande ci-dessous :

    ```bash
    TDSVER=8.0 tsql -H $SQLSERVERNAME.database.windows.net -U $SQLUSER -p 1433 -D $DATABASE -P $SQLPASWORD
    ```

    Utilisez les instructions suivantes pour vérifier que les données ont été exportées dans la table des retards :

    ```sql
    SELECT * FROM delays
    GO
    ```

    Vous devez voir une liste des données dans la table. La table inclut le nom de la ville et le retard de vol moyen pour la ville en question.

    Tapez `exit` pour quitter l’utilitaire tsql.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir terminé ce didacticiel, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées dans le stockage Azure. Vous pouvez ainsi supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même quand vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, mieux vaut supprimer les clusters quand ils ne sont pas utilisés.

Pour supprimer un cluster, consultez [Supprimer un cluster HDInsight à l’aide de votre navigateur, de PowerShell ou d’Azure CLI](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez utilisé un fichier de données brutes CSV, l’avez importé dans un espace de stockage en cluster HDInsight, puis avez transformé les données à l’aide d’Interactive Query sur Azure HDInsight.  Passez au didacticiel suivant pour en savoir plus sur le connecteur d’entrepôt Apache Hive.

> [!div class="nextstepaction"]
> [Intégrer Apache Spark et Apache Hive au le connecteur d’entrepôt Hive](./apache-hive-warehouse-connector.md)