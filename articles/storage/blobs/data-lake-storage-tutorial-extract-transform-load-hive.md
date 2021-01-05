---
title: 'Tutoriel : Extraire, transformer et charger des données avec Azure HDInsight'
description: Dans ce tutoriel, vous allez découvrir comment extraire des données d’un jeu de données CSV brut, comment les transformer à l’aide d’Apache Hive sur Azure HDInsight, puis comment charger les données transformées dans Azure SQL Database à l’aide de Sqoop.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 11/19/2019
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: 7640b7a6053532360da4c908089aecfe163bd3de
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912652"
---
# <a name="tutorial-extract-transform-and-load-data-by-using-azure-hdinsight"></a>Tutoriel : Extraire, transformer et charger des données avec Azure HDInsight

Dans ce tutoriel, vous effectuez une opération ETL : extraction, transformation et chargement de données. Vous prenez un fichier de données CSV brut, vous l’importez dans un cluster Azure HDInsight, le transformez avec Apache Hive, puis vous le chargez dans une base de données Azure SQL avec Apache Sqoop.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Extrayez et chargez les données dans un cluster HDInsight.
> * Transformez les données à l’aide d’Apache Hive.
> * Chargez les données dans une base de données Azure SQL à l’aide de Sqoop.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* **Un compte de stockage Azure Data Lake Storage Gen2 configuré pour HDInsight**

    Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md).

* **Un cluster Hadoop Linux sur HDInsight**

    Consultez [Démarrage rapide : Bien démarrer avec Apache Hadoop et Apache Hive dans Azure HDInsight à l’aide du portail Azure](../../hdinsight/hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md).

* **Azure SQL Database** : Vous allez utiliser une base de données Azure SQL comme magasin de données cible. Si vous n’avez pas de base de données dans SQL Database, consultez [Créer une base de données dans Azure SQL Database dans le portail Azure](../../azure-sql/database/single-database-create-quickstart.md).

* **Azure CLI** : Si vous n’avez pas installé l’interface de ligne de commande Azure, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

* **Un client Secure Shell (SSH)** : Pour plus d’informations, consultez [Se connecter à HDInsight (Hadoop) à l’aide de SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="download-the-flight-data"></a>Téléchargement des données de vol

1. Accédez à [Research and Innovative Technology Administration, Bureau of Transportation Statistics](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time).

2. Sur la page, sélectionnez les valeurs suivantes :

   | Nom | Valeur |
   | --- | --- |
   | Filtre année |2013 |
   | Filtre période |Janvier |
   | Champs |Year, FlightDate, Reporting_Airline, IATA_CODE_Reporting_Airline, Flight_Number_Reporting_Airline, OriginAirportID, Origin, OriginCityName, OriginState, DestAirportID, Dest, DestCityName, DestState, DepDelayMinutes, ArrDelay, ArrDelayMinutes, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay. |
   
   Effacez tous les autres champs.

3. Sélectionnez **Télécharger**. Vous obtenez un fichier .zip avec les champs de données que vous avez sélectionnés.

## <a name="extract-and-upload-the-data"></a>Extraire et charger les données

Dans cette section, vous allez charger des données dans votre cluster HDInsight, puis copier ces données dans votre compte Data Lake Storage Gen2.

1. Ouvrez une invite de commandes et utilisez la commande Secure Copy (Scp) suivante pour charger le fichier .zip sur le nœud principal du cluster HDInsight :

   ```bash
   scp <file-name>.zip <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net:<file-name.zip>
   ```

   * Remplacez l’espace réservé `<file-name>` par le nom du fichier .zip.
   * Remplacez l’espace réservé `<ssh-user-name>` par la connexion SSH du cluster HDInsight.
   * Remplacez l’espace réservé `<cluster-name>` par le nom du cluster HDInsight.

   Si vous utilisez un mot de passe pour authentifier votre connexion SSH, vous êtes invité à l’indiquer.

   Si vous utilisez une clé publique, vous pouvez avoir besoin d’utiliser le paramètre `-i` et de spécifier le chemin de la clé privée correspondante. Par exemple : `scp -i ~/.ssh/id_rsa <file_name>.zip <user-name>@<cluster-name>-ssh.azurehdinsight.net:`.

2. À la fin du chargement, connectez-vous au cluster par l’intermédiaire de SSH. À l’invite de commandes, entrez la commande suivante :

   ```bash
   ssh <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net
   ```

3. Utilisez la commande suivante pour décompresser le fichier .zip :

   ```bash
   unzip <file-name>.zip
   ```

   La commande extrait un fichier **.csv**.

4. Utilisez la commande suivante pour créer le conteneur Data Lake Storage Gen2.

   ```bash
   hadoop fs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/
   ```

   Remplacez l’espace réservé `<container-name>` par le nom que vous souhaitez donner à votre conteneur.

   Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

5. Utilisez la commande suivante pour créer un répertoire.

   ```bash
   hdfs dfs -mkdir -p abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data
   ```

6. Utilisez la commande suivante pour copier le fichier *.csv* dans le répertoire :

   ```bash
   hdfs dfs -put "<file-name>.csv" abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data/
   ```

   Utilisez des guillemets autour du nom de fichier si celui-ci contient des espaces ou des caractères spéciaux.

## <a name="transform-the-data"></a>Transformer les données

Dans cette section, vous utilisez Beeline pour exécuter un travail Apache Hive.

Dans le cadre du travail Apache Hive, vous allez importer les données du fichier .csv dans une table Apache Hive nommée **delays**.

1. À partir de l’invite SSH que vous avez déjà pour le cluster HDInsight, utilisez la commande suivante pour créer et modifier un fichier nommé **flightdelays.hql** :

   ```bash
   nano flightdelays.hql
   ```

2. Modifiez le texte suivant en remplaçant les espaces réservés `<container-name>` et `<storage-account-name>` par le nom de votre conteneur et de votre compte de stockage. Ensuite, copiez et collez le texte dans la console nano en appuyant sur la touche Maj pendant que vous cliquez sur le bouton droit de la souris.

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
    LOCATION 'abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays
    LOCATION 'abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/processed'
    AS
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

3. Enregistrez le fichier en appuyant sur Ctrl+X, puis tapez `Y` lorsque vous y êtes invité.

4. Pour démarrer Hive et exécuter le fichier **flightdelays.hql**, utilisez la commande suivante :

   ```bash
   beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
   ```

5. Dès l’exécution du script __flightdelays.hql__ terminée, utilisez la commande suivante pour ouvrir une session Beeline interactive :

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

   Cette requête récupère la liste des villes qui ont enregistré des retards liés aux conditions météo, ainsi que le temps de retard moyen, et l’enregistre dans `abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output`. Sqoop lit ensuite les données à partir de cet emplacement et les exporte vers Azure SQL Database.

7. Pour quitter Beeline, entrez `!quit` à l’invite de commandes.

## <a name="create-a-sql-database-table"></a>Créer une table de base de données SQL

Pour cette opération, vous avez besoin du nom du serveur de SQL Database. Effectuez ces étapes pour rechercher le nom de votre serveur.

1. Accédez au [portail Azure](https://portal.azure.com).

2. Sélectionnez **Bases de données SQL**.

3. Filtrez sur le nom de la base de données que vous choisissez d’utiliser. Le nom du serveur est répertorié dans la colonne **Nom du serveur**.

4. Filtrez sur le nom de la base de données que vous voulez utiliser. Le nom du serveur est répertorié dans la colonne **Nom du serveur**.

    ![Obtenir les détails du serveur SQL Azure](./media/data-lake-storage-tutorial-extract-transform-load-hive/get-azure-sql-server-details.png "Obtenir les détails du serveur SQL Azure")

    Il existe de nombreuses façons de se connecter à la base de données SQL et de créer une table. Les étapes suivantes utilisent [FreeTDS](https://www.freetds.org/) à partir du cluster HDInsight.

5. Pour installer FreeTDS, utilisez la commande suivante à partir d’une connexion SSH avec le cluster :

   ```bash
   sudo apt-get --assume-yes install freetds-dev freetds-bin
   ```

6. Une fois l’installation terminée, utilisez la commande suivante pour vous connecter à SQL Database.

   ```bash
   TDSVER=8.0 tsql -H '<server-name>.database.windows.net' -U '<admin-login>' -p 1433 -D '<database-name>'
    ```
   * Remplacez l’espace réservé `<server-name>` par le nom du serveur SQL logique.

   * Remplacez l’espace réservé `<admin-login>` par la connexion administrateur SQL Database.

   * Remplacez l’espace réservé `<database-name>` par le nom de la base de données.

   Lorsque vous y êtes invité, entrez le mot de passe de connexion administrateur de SQL Database.

   Le résultat ressemble au texte suivant :

   ```
   locale is "en_US.UTF-8"
   locale charset is "UTF-8"
   using default charset "UTF-8"
   Default database being set to sqooptest
   1>
   ```

7. À l’invite `1>`, entrez les instructions suivantes:

   ```hiveql
   CREATE TABLE [dbo].[delays](
   [OriginCityName] [nvarchar](50) NOT NULL,
   [WeatherDelay] float,
   CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED
   ([OriginCityName] ASC))
   GO
   ```

8. Une fois l’instruction `GO` entrée, les instructions précédentes sont évaluées.

   La requête crée une table nommée **delays** avec un index cluster.

9. Utilisez la requête suivante pour vérifier que la table est créée :

   ```hiveql
   SELECT * FROM information_schema.tables
   GO
   ```

   Le résultat ressemble au texte suivant :

   ```
   TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
   databaseName       dbo             delays        BASE TABLE
   ```

10. Entrée `exit` at the `1>` .

## <a name="export-and-load-the-data"></a>Exporter et charger les données

Dans les sections précédentes, vous avez copié les données transformées à l’emplacement `abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output`. Dans cette section, vous allez utiliser Sqoop pour exporter les données de `abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output` vers la table que vous avez créée dans la base de données Azure SQL.

1. Utilisez la commande suivante pour vérifier que Sqoop peut voir votre base de données SQL :

   ```bash
   sqoop list-databases --connect jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433 --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD>
   ```

   La commande retourne une liste de bases de données, dont la base de données dans laquelle vous avez créé la table **delays**.

2. Utilisez la commande suivante pour exporter les données de la table **hivesampletable** vers la table **delays** :

   ```bash
   sqoop export --connect 'jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433;database=<DATABASE_NAME>' --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD> --table 'delays' --export-dir 'abfs://<container-name>@.dfs.core.windows.net/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
   ```

   Sqoop se connecte à la base de données qui contient la table **delays** et exporte les données du répertoire `/tutorials/flightdelays/output` vers la table **delays**.

3. Après l’exécution de la commande `sqoop`, connectez-vous à la base de données à l’aide de l’utilitaire tsql :

   ```bash
   TDSVER=8.0 tsql -H <SERVER_NAME>.database.windows.net -U <ADMIN_LOGIN> -P <ADMIN_PASSWORD> -p 1433 -D <DATABASE_NAME>
   ```

4. Utilisez les instructions suivantes pour vérifier que les données ont été exportées dans la table **delays** :

   ```sql
   SELECT * FROM delays
   GO
   ```

   Vous devez voir une liste des données dans la table. La table inclut le nom de la ville et le retard de vol moyen pour la ville en question.

5. Entrez `exit` pour quitter l’utilitaire tsql.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Toutes les ressources utilisées dans ce tutoriel existent déjà. Aucun nettoyage n’est nécessaire.

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir d’autres façons d’utiliser les données dans HDInsight, consultez l’article suivant :

> [!div class="nextstepaction"]
> [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)