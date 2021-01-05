---
title: Exploration des données dans un cluster Hadoop (Team Data Science Process)
description: Utilisation du processus TDSP (Team Data Science Process) pour un scénario de bout en bout employant un cluster Hadoop HDInsight pour créer et déployer un modèle.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 53f50e98bcec4b8ace342808f0bcfd96770834b0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002219"
---
# <a name="the-team-data-science-process-in-action-use-azure-hdinsight-hadoop-clusters"></a>Team Data Science Process en action : Utilisation des clusters Azure HDInsight Hadoop
Dans cette procédure pas à pas, nous utilisons le [processus TDSP (Team Data Science Process)](overview.md) dans un scénario de bout en bout. Nous utilisons un [cluster Azure Hadoop HDInsight](https://azure.microsoft.com/services/hdinsight/) pour effectuer des opérations sur le jeu de données [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/) disponible publiquement, telles que le stockage, l’exploration, la conception de fonctionnalités et la réduction de l’échantillon de données. Pour gérer les tâches prédictives de classification et de régression binaires et multiclasses, nous créons des modèles de données avec Azure Machine Learning. 

Pour une procédure pas à pas indiquant comment gérer un plus grand jeu de données, consultez [Team Data Science Process : utiliser des clusters Azure HDInsight Hadoop sur un jeu de données de 1 To](hive-criteo-walkthrough.md).

Vous pouvez également utiliser un notebook IPython pour accomplir les tâches présentées dans cette procédure pas à pas qui utilise le jeu de données de 1 To. Pour plus d’informations, consultez [Criteo walkthrough using a Hive ODBC connection](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-hive-walkthrough-criteo.ipynb) (Procédure pas à pas Criteo à l’aide d’une connexion Hive ODBC).

## <a name="nyc-taxi-trips-dataset-description"></a><a name="dataset"></a>Description du jeu de données NYC Taxi Trips
Les données NYC Taxi Trip sont constituées d’environ 20 Go de fichiers CSV (valeurs séparées par des virgules) compressés (~ 48 Go non compressés). Ces données comptent plus de 173 millions de courses individuelles et incluent les tarifs payés pour chaque course. Chaque enregistrement de course inclut le lieu et l’heure d’embarquement et de débarquement, le numéro de licence (du chauffeur) rendu anonyme et le numéro de médaillon (ID unique) du taxi. Les données portent sur toutes les courses effectuées en 2013 et sont fournies dans les deux jeux de données ci-après pour chaque mois :

- Les fichiers CSV trip_data contiennent les détails des courses, comme le nombre de passagers, les points d’embarquement et de débarquement, la durée du trajet et la distance parcourue. Voici quelques exemples d’enregistrements :

  `medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

  `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

- Les fichiers CSV trip_fare contiennent des informations sur le prix payé pour chaque course, comme le type de paiement, le montant, la surcharge et les taxes, les pourboires et les péages, ainsi que le montant total réglé. Voici quelques exemples d’enregistrements :

  `medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

  `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

  `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

  `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

La clé unique permettant de joindre trip\_data et trip\_fare se compose des champs suivants : medallion (médaillon), hack\_licence (licence de taxi) et pickup\_datetime (date et heure d’embarquement). Pour obtenir tous les détails pertinents pour un voyage en particulier, il suffit de joindre ces trois clés.

## <a name="examples-of-prediction-tasks"></a><a name="mltasks"></a>Exemples de tâches de prédiction
Déterminer le type de prédictions que vous souhaitez baser sur l’analyse de données pour clarifier les tâches d’exécution requises. Voici trois exemples de problèmes de prédiction que nous allons traiter dans cette procédure pas à pas, tous étant basés sur le montant du *pourboire\_*  :

- **Classification binaire** : Prédire si un pourboire a ou non été versé pour une course. Autrement dit, un *tip\_amount* qui est supérieur à 0 $ USD est un exemple positif, tandis qu’un *tip\_amount* égal à 0 $ USD est un exemple négatif.

  - Classe 0 : tip_amount = 0 $
  - Classe 1 : tip_amount > 0 $

- **Classification multiclasse** : Prédire la fourchette de montants des pourboires versés pour une course. Nous divisons la valeur *tip\_amount* en cinq classes :

  - Classe 0 : tip_amount = 0 $
  - Classe 1 : tip_amount > 0 $ et tip_amount <= 5 $
  - Classe 2 : tip_amount > 5 $ et tip_amount <= 10 $
  - Classe 3 : tip_amount > 10 $ et tip_amount <= 20 $
  - Classe 4 : tip_amount > 20 $

- **Tâche de régression** : Prédire le montant du pourboire versé pour une course.  

## <a name="set-up-an-hdinsight-hadoop-cluster-for-advanced-analytics"></a><a name="setup"></a>Configuration d’un cluster Hadoop HDInsight pour une analyse avancée
> [!NOTE]
> Il s’agit généralement d’une tâche d’administration.
> 
> 

Vous pouvez configurer un environnement Azure pour une analyse avancée qui utilise un cluster HDInsight en trois étapes :

1. [Créer un compte de stockage](../../storage/common/storage-account-create.md) : Ce compte de stockage est utilisé pour stocker des données dans un stockage Blob Azure. Les données utilisées dans les clusters HDInsight résident également ici.
2. [Personnaliser des clusters Hadoop Azure HDInsight pour le processus et la technologie d'analyse avancée](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md). Cette étape crée un cluster Hadoop HDInsight avec Anaconda Python 2.7 64 bits installé sur tous les nœuds. Il existe deux étapes importantes à retenir lors de la personnalisation de votre cluster HDInsight.
   
   * Rappelez-vous de lier le compte de stockage créé à l’étape 1 à votre cluster HDInsight, lorsque vous le créez. Ce compte de stockage accède aux données qui peuvent être traitées au sein du cluster.
   * Après avoir créé le cluster, activez l’accès à distance au nœud principal du cluster. Accédez à l’onglet **Configuration**, puis sélectionnez **Activation à distance**. Cette étape fournit les informations d'identification d'utilisateur utilisées pour la connexion à distance.
3. [Créer un espace de travail Microsoft Azure Machine Learning](../classic/create-workspace.md) : Cet espace de travail vous permet de générer des modèles Machine Learning. Cette tâche est entamée après avoir effectué une exploration de données initiales et une réduction de l’échantillon à l’aide du cluster HDInsight.

## <a name="get-the-data-from-a-public-source"></a><a name="getdata"></a>Obtenir les données auprès d’une source publique
> [!NOTE]
> Il s’agit généralement d’une tâche d’administration.
> 
> 

Pour copier le jeu de données [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/) sur votre machine depuis son emplacement public, utilisez l’une des méthodes décrites dans [Déplacer des données vers et depuis le stockage Blob Azure](move-azure-blob.md).

Nous décrivons ici comment utiliser AzCopy pour transférer les fichiers contenant des données. Pour télécharger et installer AzCopy, suivez les instructions dans [Prise en main de l’utilitaire de ligne de commande AzCopy](../../storage/common/storage-use-azcopy-v10.md).

1. Dans une fenêtre d’invite de commandes, exécutez les commandes AzCopy suivantes en remplaçant *\<path_to_data_folder>* par la destination souhaitée :

    ```console
    "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:https://nyctaxitrips.blob.core.windows.net/data /Dest:<path_to_data_folder> /S
    ```

1. Une fois la copie terminée, vous verrez un total de 24 fichiers compressés dans le dossier de données choisi. Décompressez les fichiers téléchargés dans le même répertoire sur votre ordinateur local. Prenez note du dossier où résident les fichiers décompressés. Ce dossier est désigné *\<path\_to\_unzipped_data\_files\>* ci-après.

## <a name="upload-the-data-to-the-default-container-of-the-hdinsight-hadoop-cluster"></a><a name="upload"></a>Charger les données dans le conteneur par défaut du cluster Hadoop HDInsight
> [!NOTE]
> Il s’agit généralement d’une tâche d’administration.
> 
> 

Dans les commandes AzCopy suivantes, remplacez les paramètres suivants par les valeurs réelles que vous avez spécifiées lors de la création du cluster Hadoop et lors de la décompression des fichiers de données.

* * **\<path_to_data_folder>** _ Répertoire (avec le chemin) sur votre machine qui contient les fichiers de données décompressés.  
_ * **\<storage account name of Hadoop cluster>** _ Compte de stockage associé à votre cluster HDInsight.
_ * **\<default container of Hadoop cluster>** _ Conteneur par défaut utilisé par votre cluster. Le nom du conteneur par défaut est généralement le même que celui du cluster. Par exemple, si le cluster est appelé « abc123.azurehdinsight.net », le conteneur par défaut est abc123.
_ * **\<storage account key>** _ Clé du compte de stockage utilisé par votre cluster.

À partir d’une invite de commandes ou d’une fenêtre Windows PowerShell, exécutez les deux commandes AzCopy suivantes.

Cette commande permet de charger les données relatives aux courses dans le répertoire _*_nyctaxitripraw_*_ sur le conteneur par défaut du cluster Hadoop.

```console
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:<path_to_unzipped_data_files> /Dest:https://<storage account name of Hadoop cluster>.blob.core.windows.net/<default container of Hadoop cluster>/nyctaxitripraw /DestKey:<storage account key> /S /Pattern:trip_data__.csv
```

Cette commande permet de charger les données de prix dans le répertoire ***nyctaxifareraw** _ sur le conteneur par défaut du cluster Hadoop.

```console
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:<path_to_unzipped_data_files> /Dest:https://<storage account name of Hadoop cluster>.blob.core.windows.net/<default container of Hadoop cluster>/nyctaxifareraw /DestKey:<storage account key> /S /Pattern:trip_fare__.csv
```

Les données doivent désormais se trouver dans le stockage Blob et prêtes à être utilisées au sein du cluster HDInsight.

## <a name="sign-in-to-the-head-node-of-hadoop-cluster-and-prepare-for-exploratory-data-analysis"></a><a name="#download-hql-files"></a>Se connecter au nœud principal du cluster Hadoop et préparer une analyse exploratoire des données
> [!NOTE]
> Il s’agit généralement d’une tâche d’administration.
> 
> 

Pour accéder au nœud principal du cluster afin d’exécuter une analyse exploratoire des données et une réduction de l’échantillon des données, suivez la procédure décrite dans [Accéder au nœud principal du cluster Hadoop](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md).

Dans cette procédure pas à pas, nous utilisons principalement les requêtes écrites dans [Hive](https://hive.apache.org/), un langage de requête similaire à SQL, pour effectuer des explorations de données préliminaires. Les requêtes Hive sont stockées dans des fichiers .hql. Nous réduisons ensuite l’échantillon de ces données à utiliser avec Machine Learning pour la création de modèles.

Pour préparer le cluster à une analyse exploratoire des données, téléchargez les fichiers .hql contenant les scripts Hive pertinents de [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) dans un répertoire local (C:\temp) sur le nœud principal. Ouvrez l’invite de commandes dans le nœud principal du cluster et exécutez les deux commandes suivantes :

```console
set script='https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/DataScienceProcess/DataScienceScripts/Download_DataScience_Scripts.ps1'

@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString(%script%))"
```

Ces deux commandes téléchargent tous les fichiers .hql nécessaires pour cette procédure pas à pas dans le répertoire local ***C:\temp&#92;** _ du nœud principal.

## <a name="create-hive-database-and-tables-partitioned-by-month"></a><a name="#hive-db-tables"></a>Créer la base de données Hive et les tables partitionnées par mois
> [!NOTE]
> Cette tâche est généralement destinée à un administrateur.
> 
> 

Vous êtes maintenant prêts à créer des tables Hive pour le jeu de données NYC taxi.
Dans le nœud principal du cluster Hadoop, ouvrez la ligne de commande Hadoop sur le bureau du nœud principal. Accédez au répertoire Hive en exécutant la commande suivante :

```console
cd %hive_home%\bin
```

> [!NOTE]
> Exécutez, dans cette procédure pas à pas, toutes les commandes Hive depuis l’invite de l’emplacement/du répertoire Hive. Tout problème lié au chemin d’accès est traité automatiquement. Nous utiliserons les termes « Invite du répertoire Hive », « Invite de l’emplacement/du répertoire Hive » et « Ligne de commande Hadoop » de manière interchangeable dans cette procédure pas à pas.
> 
> 

À partir de l'invite du répertoire Hive, exécutez la commande suivante dans la ligne de commande Hadoop du nœud principal qui crée une base de données et des tables Hive :

```console
hive -f "C:\temp\sample_hive_create_db_and_tables.hql"
```

Voici le contenu du fichier _ *C:\temp\sample\_hive\_create\_db\_and\_tables.hql** qui crée la base de données Hive **nyctaxidb** et les tables **trip** et **fare**.

```hiveql
create database if not exists nyctaxidb;

create external table if not exists nyctaxidb.trip
(
    medallion string,
    hack_license string,
    vendor_id string,
    rate_code string,
    store_and_fwd_flag string,
    pickup_datetime string,
    dropoff_datetime string,
    passenger_count int,
    trip_time_in_secs double,
    trip_distance double,
    pickup_longitude double,
    pickup_latitude double,
    dropoff_longitude double,
    dropoff_latitude double)  
PARTITIONED BY (month int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/trip' TBLPROPERTIES('skip.header.line.count'='1');

create external table if not exists nyctaxidb.fare
(
    medallion string,
    hack_license string,
    vendor_id string,
    pickup_datetime string,
    payment_type string,
    fare_amount double,
    surcharge double,
    mta_tax double,
    tip_amount double,
    tolls_amount double,
    total_amount double)
PARTITIONED BY (month int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/fare' TBLPROPERTIES('skip.header.line.count'='1');
```

Ce script Hive crée deux tables :

* La table **trip** contient le détail du trajet de chaque course (détails du chauffeur, heure d’embarquement, durée de la course et distance parcourue).
* La table **fare** contient le détail des prix (montant de la course, montant des pourboires, péages et surcharges).

Si vous avez besoin d’aide sur ces procédures ou si vous souhaitez examiner d’autres solutions, consultez la section [Envoyer des requêtes Hive directement depuis la ligne de commande Hadoop](move-hive-tables.md#submit).

## <a name="load-data-to-hive-tables-by-partitions"></a><a name="#load-data"></a>Charger les données dans des tables Hive par partitions
> [!NOTE]
> Cette tâche est généralement destinée à un administrateur.
> 
> 

Le jeu de données taxi NYC a un partitionnement naturel par mois, qui nous permet d’accélérer les temps de traitement et de requête. Les commandes PowerShell suivantes (émises à partir du répertoire Hive à l’aide de la ligne de commande Hadoop) chargent des données dans des tables Hive « trip » et « fare » partitionnées par mois.

```powershell
for /L %i IN (1,1,12) DO (hive -hiveconf MONTH=%i -f "C:\temp\sample_hive_load_data_by_partitions.hql")
```

Le fichier **sample\_hive\_load\_data\_by\_partitions.hql** contient les commandes **LOAD** suivantes :

```hiveql
LOAD DATA INPATH 'wasb:///nyctaxitripraw/trip_data_${hiveconf:MONTH}.csv' INTO TABLE nyctaxidb.trip PARTITION (month=${hiveconf:MONTH});
LOAD DATA INPATH 'wasb:///nyctaxifareraw/trip_fare_${hiveconf:MONTH}.csv' INTO TABLE nyctaxidb.fare PARTITION (month=${hiveconf:MONTH});
```

Un certain nombre de requêtes Hive utilisées ici dans le processus d’exploration impliquent la recherche dans une ou deux partitions seulement. Mais ces requêtes peuvent être exécutées pour l’ensemble du jeu de données.

### <a name="show-databases-in-the-hdinsight-hadoop-cluster"></a><a name="#show-db"></a>Afficher les bases de données dans le cluster Hadoop HDInsight
Pour afficher les bases de données créées dans le cluster Hadoop HDInsight dans la fenêtre de commande Hadoop, exécutez la commande suivante dans la ligne de commande Hadoop :

```console
hive -e "show databases;"
```

### <a name="show-the-hive-tables-in-the-nyctaxidb-database"></a><a name="#show-tables"></a>Afficher les tables Hive dans la base de données **nyctaxidb**
Pour afficher les tables dans la base de données **nyctaxidb**, exécutez la commande suivante dans la ligne de commande Hadoop :

```console
hive -e "show tables in nyctaxidb;"
```

Nous pouvons confirmer que les tables sont partitionnées en exécutant la commande suivante :

```console
hive -e "show partitions nyctaxidb.trip;"
```

Voici la sortie attendue :

```output
month=1
month=10
month=11
month=12
month=2
month=3
month=4
month=5
month=6
month=7
month=8
month=9
Time taken: 2.075 seconds, Fetched: 12 row(s)
```

De même, nous pouvons vérifier que la table « fare » est partitionnée en exécutant la commande suivante :

```console
hive -e "show partitions nyctaxidb.fare;"
```

Voici la sortie attendue :

```output
month=1
month=10
month=11
month=12
month=2
month=3
month=4
month=5
month=6
month=7
month=8
month=9
Time taken: 1.887 seconds, Fetched: 12 row(s)
```

## <a name="data-exploration-and-feature-engineering-in-hive"></a><a name="#explore-hive"></a>Exploration des données et ingénierie des fonctionnalités dans Hive
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Vous pouvez utiliser des requêtes Hive pour les tâches d’exploration des données et d’ingénierie des fonctionnalités pour les données chargées dans les tables Hive. Voici des exemples de ces tâches :

* Affichez les 10 premiers enregistrements des deux tables.
* explorer les distributions de données de quelques champs portant sur différentes périodes ;
* examiner la qualité des données des champs de longitude et de latitude ;
* Générer des étiquettes de classification binaire et multiclasse reposant sur la valeur « tip amount ».
* Générez des fonctionnalités en calculant les distances des trajets directs.

### <a name="exploration-view-the-top-10-records-in-table-trip"></a>Exploration : Afficher les 10 premiers enregistrements de la table trip
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Pour avoir un aperçu des données, examinez 10 enregistrements de chaque table. Pour analyser les enregistrements, exécutez les deux requêtes suivantes séparément depuis l’invite de commande du répertoire Hive dans la console de la ligne de commande Hadoop.

Pour obtenir les 10 premiers enregistrements dans la table « trip » du premier mois :

```console
hive -e "select * from nyctaxidb.trip where month=1 limit 10;"
```

Pour obtenir les 10 premiers enregistrements dans la table « fare » du premier mois :

```console
hive -e "select * from nyctaxidb.fare where month=1 limit 10;"
```

Vous pouvez enregistrer les enregistrements dans un fichier pour en faciliter la lecture en apportant une petite modification à la requête précédente :

```console
hive -e "select * from nyctaxidb.fare where month=1 limit 10;" > C:\temp\testoutput
```

### <a name="exploration-view-the-number-of-records-in-each-of-the-12-partitions"></a>Exploration : Afficher le nombre d’enregistrements dans chacune des 12 partitions
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

La façon dont le nombre de courses varie au cours de l'année civile est intéressante. Le regroupement par mois affiche la distribution des courses.

```console
hive -e "select month, count(*) from nyctaxidb.trip group by month;"
```

Cette commande génère la sortie suivante :

```output
1       14776615
2       13990176
3       15749228
4       15100468
5       15285049
6       14385456
7       13823840
8       12597109
9       14107693
10      15004556
11      14388451
12      13971118
Time taken: 283.406 seconds, Fetched: 12 row(s)
```

Ici, la première colonne est le mois et la seconde est le nombre de courses pour ce mois.

Nous pouvons également compter le nombre total d’enregistrements dans notre jeu de données de courses en exécutant la commande suivante à l’invite du répertoire Hive :

```console
hive -e "select count(*) from nyctaxidb.trip;"
```

Cette commande interrompt :

```output
173179759
Time taken: 284.017 seconds, Fetched: 1 row(s)
```

À l’aide des commandes similaires à celles indiquées pour le jeu de données trip, nous pouvons émettre des requêtes Hive à partir de l’invite du répertoire Hive pour le jeu de données fare afin de valider le nombre d’enregistrements.

```console
hive -e "select month, count(*) from nyctaxidb.fare group by month;"
```

Cette commande génère cette sortie :

```output
1       14776615
2       13990176
3       15749228
4       15100468
5       15285049
6       14385456
7       13823840
8       12597109
9       14107693
10      15004556
11      14388451
12      13971118
Time taken: 253.955 seconds, Fetched: 12 row(s)
```

Le même nombre de courses effectuées par mois est retourné pour les deux jeux de données. Cela est la première confirmation que les données ont été chargées correctement.

Vous pouvez compter le nombre total d’enregistrements dans le jeu de données fare à l’aide de la commande suivante à partir de l’invite du répertoire Hive :

```console
hive -e "select count(*) from nyctaxidb.fare;"
```

Cette commande interrompt :

```output
173179759
Time taken: 186.683 seconds, Fetched: 1 row(s)
```

Le nombre total d’enregistrements dans les deux tables est également identique. Cela est la deuxième confirmation que les données ont été chargées correctement.

### <a name="exploration-trip-distribution-by-medallion"></a>Exploration : Distribution des courses par médaillon
> [!NOTE]
> Cette analyse est généralement une tâche de scientifique des données.
> 
> 

Cet exemple identifie le médaillon (numéro de taxi) sur plus de 100 courses au cours d’une période donnée. La requête a accès aux tables partitionnées, car elle est conditionnée par la variable de partition **month**. Les résultats de la requête sont écrits dans un fichier local **queryoutput.tsv** dans `C:\temp` sur le nœud principal.

```console
hive -f "C:\temp\sample_hive_trip_count_by_medallion.hql" > C:\temp\queryoutput.tsv
```

Voici le contenu du fichier **sample\_hive\_trip\_count\_by\_medallion.hql** pour l’inspection.

```hiveql
SELECT medallion, COUNT(*) as med_count
FROM nyctaxidb.fare
WHERE month<=3
GROUP BY medallion
HAVING med_count > 100
ORDER BY med_count desc;
```

Le médaillon dans le jeu de données NYC taxi identifie un seul taxi. Vous pouvez identifier les taxis « occupés » par comparaison en demandant quels taxis ont effectué plus d’un certain nombre d’allers-retours sur une période donnée. L’exemple suivant identifie les taxis qui ont effectué plus d’une centaine de courses durant les trois premiers mois et enregistre les résultats de la requête dans un fichier local,**C:\temp\queryoutput.tsv**.

Voici le contenu du fichier **sample\_hive\_trip\_count\_by\_medallion.hql** pour l’inspection.

```hiveql
SELECT medallion, COUNT(*) as med_count
FROM nyctaxidb.fare
WHERE month<=3
GROUP BY medallion
HAVING med_count > 100
ORDER BY med_count desc;
```

À partir de l’invite du répertoire Hive, exécutez la commande suivante :

```console
hive -f "C:\temp\sample_hive_trip_count_by_medallion.hql" > C:\temp\queryoutput.tsv
```

### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>Exploration : Distribution des courses par médaillon et par licence de taxi
> [!NOTE]
> Cette tâche est généralement destinée à un scientifique des données.
> 
> 

Lors de l'exploration d'un jeu de données, nous devons examiner fréquemment le nombre de distributions des groupes de valeurs. Cette section fournit un exemple de procédure à suivre pour effectuer cette analyse pour les chauffeurs et les taxis.

Le fichier **sample\_hive\_trip\_count\_by\_medallion\_license.hql** regroupe le jeu de données fare sur **medallion** et **hack_license**, et renvoie le nombre de chaque combinaison. Son contenu est présenté ci-dessous :

```hiveql
SELECT medallion, hack_license, COUNT(*) as trip_count
FROM nyctaxidb.fare
WHERE month=1
GROUP BY medallion, hack_license
HAVING trip_count > 100
ORDER BY trip_count desc;
```

Cette requête renvoie les combinaisons de taxi et de chauffeur classées par ordre décroissant de courses.

À partir de l'invite du répertoire Hive, exécutez :

```console
hive -f "C:\temp\sample_hive_trip_count_by_medallion_license.hql" > C:\temp\queryoutput.tsv
```

Les résultats de la requête sont écrits dans un fichier local, **C:\temp\queryoutput.tsv**.

### <a name="exploration-assessing-data-quality-by-checking-for-invalid-longitude-or-latitude-records"></a>Exploration : Évaluation de la qualité des données en recherchant les enregistrements de longitude ou de latitude non valides
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Un objectif commun d'une analyse exploratoire des données est d'éliminer les enregistrements non valides ou incorrects. L'exemple de cette section détermine si les champs de latitude ou de longitude contiennent une valeur en dehors de la zone NYC. Dans la mesure où il est probable que la valeur de latitude-longitude de ces enregistrements soit erronée, nous souhaitons l’éliminer des données devant être utilisées pour la modélisation.

Voici le contenu du fichier **sample\_hive\_quality\_assessment.hql** pour l’inspection.

```hiveql
    SELECT COUNT(*) FROM nyctaxidb.trip
    WHERE month=1
    AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND -30
    OR    CAST(pickup_latitude AS float) NOT BETWEEN 30 AND 90
    OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND -30
    OR    CAST(dropoff_latitude AS float) NOT BETWEEN 30 AND 90);
```

À partir de l'invite du répertoire Hive, exécutez :

```console
hive -S -f "C:\temp\sample_hive_quality_assessment.hql"
```

L’argument *-S* inclus dans la commande supprime l’affichage de l’état des travaux Map/Reduce Hive. Cette comment est utile, car elle rend l’affichage de la sortie de la requête Hive plus lisible.

### <a name="exploration-binary-class-distributions-of-trip-tips"></a>Exploration : Distributions de classe binaire des pourboires de course
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Pour le problème de classification binaire présenté dans la section [Exemples de tâches de prédiction](hive-walkthrough.md#mltasks) , il est utile de savoir si un pourboire a été donné ou non. Cette distribution de pourboires est binaire :

* pourboire donné (classe 1, tip\_amount > 0 $)  
* aucun pourboire (classe 0, tip\_amount = 0 $)

Le fichier **sample\_hive\_tipped\_frequencies.hql** suivant indique quelle commande exécuter :

```hiveql
SELECT tipped, COUNT(*) AS tip_freq
FROM
(
    SELECT if(tip_amount > 0, 1, 0) as tipped, tip_amount
    FROM nyctaxidb.fare
)tc
GROUP BY tipped;
```

À partir de l'invite du répertoire Hive, exécutez :

```console
hive -f "C:\temp\sample_hive_tipped_frequencies.hql"
```


### <a name="exploration-class-distributions-in-the-multiclass-setting"></a>Exploration : Distributions de classe dans le paramètre multiclasse
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Pour le problème de classification multiclasse décrit dans la section [Exemples de tâches de prédiction](hive-walkthrough.md#mltasks), ce jeu de données se prête également à une classification naturelle pour prédire la quantité de pourboires donnés. Nous pouvons utiliser des compartiments pour définir les montants de pourboires dans la requête. Pour obtenir les distributions de classe pour les différents montants de pourboire, utilisez le fichier **sample\_hive\_tip\_range\_frequencies.hql**. Son contenu est présenté ci-dessous.

```hiveql
SELECT tip_class, COUNT(*) AS tip_freq
FROM
(
    SELECT if(tip_amount=0, 0,
        if(tip_amount>0 and tip_amount<=5, 1,
        if(tip_amount>5 and tip_amount<=10, 2,
        if(tip_amount>10 and tip_amount<=20, 3, 4)))) as tip_class, tip_amount
    FROM nyctaxidb.fare
)tc
GROUP BY tip_class;
```

Exécutez la commande suivante à partir de la console de ligne de commande Hadoop :

```console
hive -f "C:\temp\sample_hive_tip_range_frequencies.hql"
```

### <a name="exploration-compute-the-direct-distance-between-two-longitude-latitude-locations"></a>Exploration : Calcul de la distance directe entre deux emplacements de latitude-longitude
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Vous souhaiterez peut-être savoir s’il existe une différence entre la distance directe entre deux emplacements et la distance de course réelle du taxi. Un passager peut être moins susceptible de donner un pourboire s’il se rend compte que le chauffeur a pris intentionnellement un itinéraire plus long.

Pour afficher la comparaison entre la distance de course réelle et la [distance Haversine](https://en.wikipedia.org/wiki/Haversine_formula) entre deux points de latitude-longitude (la distance orthodromique), vous pouvez utiliser les fonctions trigonométriques disponibles au sein de Hive :

```hiveql
set R=3959;
set pi=radians(180);

insert overwrite directory 'wasb:///queryoutputdir'

select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude, trip_distance, trip_time_in_secs,
${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
 *${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
 *cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
 /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
 +cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*
 pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance
from nyctaxidb.trip
where month=1
and pickup_longitude between -90 and -30
and pickup_latitude between 30 and 90
and dropoff_longitude between -90 and -30
and dropoff_latitude between 30 and 90;
```

Dans la requête précédente, R est le rayon de la Terre en miles et pi est converti en radians. Les points de latitude-longitude sont filtrés pour supprimer les valeurs éloignées de la zone NYC.

Dans ce cas, nous écrivons les résultats sur un répertoire nommé **queryoutputdir**. La séquence des commandes suivantes crée d’abord ce répertoire de sortie, puis exécute la commande Hive.

À partir de l'invite du répertoire Hive, exécutez :

```hiveql
hdfs dfs -mkdir wasb:///queryoutputdir

hive -f "C:\temp\sample_hive_trip_direct_distance.hql"
```

Les résultats de la requête sont consignés dans neuf blobs Azure **queryoutputdir/000000\_0** à  **queryoutputdir/000008\_0**, situés dans le conteneur par défaut du cluster Hadoop.

Pour connaître la taille des objets blob individuels, exécutez la commande suivante à partir de l’invite du répertoire Hive :

```hiveql
hdfs dfs -ls wasb:///queryoutputdir
```

Pour afficher le contenu d’un fichier donné, par exemple, **000000\_0**, utilisez la commande Hadoop `copyToLocal`.

```hiveql
hdfs dfs -copyToLocal wasb:///queryoutputdir/000000_0 C:\temp\tempfile
```

> [!WARNING]
> `copyToLocal` peut être très lent pour les fichiers volumineux et n’est pas recommandé pour une utilisation avec ceux-ci.  
> 
> 

Le principal avantage lié au fait que ces données résident dans un objet blob Azure est que nous pouvons explorer les données au sein de Machine Learning à l’aide du module [Importer des données][import-data].

## <a name="down-sample-data-and-build-models-in-machine-learning"></a><a name="#downsample"></a>Réduire l’échantillon des données et créer des modèles dans Machine Learning
> [!NOTE]
> Il s’agit généralement d’une tâche de scientifique des données.
> 
> 

Après la phase d’analyse exploratoire des données, nous sommes prêts à réduire l’échantillon des données pour créer des modèles dans Machine Learning. Dans cette section, nous montrons comment utiliser une requête Hive pour réduire l’échantillon des données. Machine Learning y accède ensuite à partir du module [Importer des données][import-data].

### <a name="down-sampling-the-data"></a>Réduire l’échantillonnage des données
Il existe deux étapes dans cette procédure. Tout d’abord nous regroupons les tables **nyctaxidb.trip** et **nyctaxidb.fare** sur trois clés présentes dans tous les enregistrements : **medallion**, **hack\_license** et **pickup\_datetime**. Nous générons ensuite une étiquette de classification binaire **avec pourboire** et une étiquette de classification multiclasse, **tip\_class**.

Pour pouvoir utiliser les échantillons de données réduits directement à partir du module [Importer des données][import-data] dans Machine Learning, vous devez stocker les résultats de la requête précédente dans une table Hive interne. Dans ce qui suit, nous créons une table interne Hive et remplissons son contenu avec les données regroupées et à échantillon réduit.

La requête applique des fonctions Hive standard directement pour générer les paramètres de temps suivants dans le champ **pickup\_datetime** :
- heure de la journée
- semaine de l’année
- jour de la semaine (« 1 » pour lundi et « 7 » pour dimanche)

La requête génère également la distance directe entre les lieux d’embarquement et de débarquement. Pour obtenir la liste complète de ces fonctions, consultez [UDF LanguageManual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF).

Ensuite, cette requête réduit l’échantillon des données pour que ses résultats tiennent dans Machine Learning Studio. Seulement 1 pourcent environ du jeu de données d’origine est importé dans le Studio.

Voici le contenu du fichier **sample\_hive\_prepare\_for\_aml\_full.hql** qui prépare les données pour la création du modèle dans Machine Learning :

```hiveql
set R = 3959;
set pi=radians(180);

create table if not exists nyctaxidb.nyctaxi_downsampled_dataset (

medallion string,
hack_license string,
vendor_id string,
rate_code string,
store_and_fwd_flag string,
pickup_datetime string,
dropoff_datetime string,
pickup_hour string,
pickup_week string,
weekday string,
passenger_count int,
trip_time_in_secs double,
trip_distance double,
pickup_longitude double,
pickup_latitude double,
dropoff_longitude double,
dropoff_latitude double,
direct_distance double,
payment_type string,
fare_amount double,
surcharge double,
mta_tax double,
tip_amount double,
tolls_amount double,
total_amount double,
tipped string,
tip_class string
)
row format delimited fields terminated by ','
lines terminated by '\n'
stored as textfile;

--- now insert contents of the join into the above internal table

insert overwrite table nyctaxidb.nyctaxi_downsampled_dataset
select
t.medallion,
t.hack_license,
t.vendor_id,
t.rate_code,
t.store_and_fwd_flag,
t.pickup_datetime,
t.dropoff_datetime,
hour(t.pickup_datetime) as pickup_hour,
weekofyear(t.pickup_datetime) as pickup_week,
from_unixtime(unix_timestamp(t.pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),'u') as weekday,
t.passenger_count,
t.trip_time_in_secs,
t.trip_distance,
t.pickup_longitude,
t.pickup_latitude,
t.dropoff_longitude,
t.dropoff_latitude,
t.direct_distance,
f.payment_type,
f.fare_amount,
f.surcharge,
f.mta_tax,
f.tip_amount,
f.tolls_amount,
f.total_amount,
if(tip_amount>0,1,0) as tipped,
if(tip_amount=0,0,
if(tip_amount>0 and tip_amount<=5,1,
if(tip_amount>5 and tip_amount<=10,2,
if(tip_amount>10 and tip_amount<=20,3,4)))) as tip_class

from
(
select
medallion,
hack_license,
vendor_id,
rate_code,
store_and_fwd_flag,
pickup_datetime,
dropoff_datetime,
passenger_count,
trip_time_in_secs,
trip_distance,
pickup_longitude,
pickup_latitude,
dropoff_longitude,
dropoff_latitude,
${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
*${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
*cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
/sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
+cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance,
rand() as sample_key

from nyctaxidb.trip
where pickup_latitude between 30 and 90
    and pickup_longitude between -90 and -30
    and dropoff_latitude between 30 and 90
    and dropoff_longitude between -90 and -30
)t
join
(
select
medallion,
hack_license,
vendor_id,
pickup_datetime,
payment_type,
fare_amount,
surcharge,
mta_tax,
tip_amount,
tolls_amount,
total_amount
from nyctaxidb.fare
)f
on t.medallion=f.medallion and t.hack_license=f.hack_license and t.pickup_datetime=f.pickup_datetime
where t.sample_key<=0.01
```

Pour exécuter cette requête à partir de l’invite du répertoire Hive :

```console
hive -f "C:\temp\sample_hive_prepare_for_aml_full.hql"
```

Nous avons maintenant une table interne **nyctaxidb.nyctaxi_downsampled_dataset**, qui est accessible à l’aide du module [Importer des données][import-data] de Machine Learning. En outre, nous pouvons utiliser ce jeu de données pour créer des modèles Machine Learning.  

### <a name="use-the-import-data-module-in-machine-learning-to-access-the-down-sampled-data"></a>Utiliser le module Importer des données dans Machine Learning pour accéder aux données à échantillon réduit
Pour générer des requêtes Hive dans le module [Importer des données][import-data] de Machine Learning, vous devez accéder à un espace de travail Machine Learning. Vous avez également besoin d’accéder aux informations d’identification du cluster et à son compte de stockage associé.

Voici certains détails sur le module [Importer des données][import-data] et les paramètres à entrer :

**URI du serveur HCatalog** : si le nom du cluster est **abc123**, utilisez : https:\//abc123.azurehdinsight.net.

**Nom du compte utilisateur Hadoop** : Nom d’utilisateur choisi pour le cluster (et non le nom d’utilisateur de l’accès à distance).

**Mot de passe du compte utilisateur Hadoop** : Mot de passe choisi pour le cluster (et non le mot de passe d’accès à distance).

**Emplacement des données de sortie** : Il s’agit d’un emplacement Azure.

**Nom du compte de Stockage Azure** : Nom du compte de stockage par défaut associé au cluster.

**Nom de conteneur Azure** : Nom du conteneur par défaut du cluster. Il correspond généralement à celui du cluster. Pour un cluster nommé **abc123**, le nom est abc123.

> [!IMPORTANT]
> Toute table que nous souhaitons interroger à l’aide du module [Importer des données][import-data] dans Machine Learning doit être une table interne.
> 
> 

Voici comment déterminer si une table **T** dans une base de données **D.db** est une table interne. À partir de l’invite du répertoire Hive, exécutez la commande suivante :

```hiveql
hdfs dfs -ls wasb:///D.db/T
```

Si la table est une table interne et qu’elle est remplie, son contenu doit s’afficher ici.

Pour déterminer si une table est une table interne, il est également possible d’utiliser l’Explorateur Stockage Azure. Utilisez-le pour accéder au nom de conteneur par défaut du cluster, puis filtrez par nom de table. Si la table et son contenu s'affichent, cela confirme qu'il s’agit d’une table interne.

Voici une capture d’écran de la requête Hive et du module [Importer des données][import-data] :

![Capture d’écran de requête Hive pour le module Importer des données](./media/hive-walkthrough/1eTYf52.png)

Étant donné que nos données à échantillon réduit résident dans le conteneur par défaut, la requête Hive obtenue de Machine Learning est simple. Simplement **SELECT * FROM nyctaxidb.nyctaxi\_downsampled\_data**.

Le jeu de données peut maintenant être utilisé comme point de départ pour créer des modèles Machine Learning.

### <a name="build-models-in-machine-learning"></a><a name="mlmodel"></a>Créer des modèles dans Machine Learning
Vous pouvez maintenant passer aux phases de création et de déploiement de modèles dans [Machine Learning](https://studio.azureml.net). Les données sont exploitables pour répondre aux problèmes de prédiction identifiés précédemment :

- **Classification binaire** : Prédire si un pourboire a ou non été versé pour une course.

  **Apprenant utilisé :** Régression logistique à deux classes

  a. Pour ce problème, l’étiquette (ou classe) cible est **avec pourboire**. Le jeu de données original à l’échantillon réduit dispose de quelques colonnes qui sont des fuites cibles pour cette expérience de classification. En particulier : **tip\_class**, **tip\_amount** et **total\_amount** révèlent des informations sur l’étiquette cible qui ne sont pas disponibles au moment du test. Nous supprimons ces colonnes du compte à l’aide du module [Sélectionner des colonnes dans le jeu de données][select-columns].

  Le diagramme suivant illustre notre expérience pour prédire si un pourboire a été versé pour une course donnée :

  ![Diagramme illustrant l’expérience pour prédire si le pourboire a été payé](./media/hive-walkthrough/QGxRz5A.png)

  b. Pour cette expérience, nos distributions d'étiquette cible ont été d’environ 1:1.

   Le graphique suivant montre la distribution d’étiquettes de classe de pourboire pour le problème de classification binaire :

  ![Graphique de distribution d’étiquettes de classe de pourboire](./media/hive-walkthrough/9mM4jlD.png)

    Par conséquent, nous obtenons une zone sous la courbe (AUC) de 0,987 comme indiqué dans la figure suivante :

  ![Graphique de valeur AUC](./media/hive-walkthrough/8JDT0F8.png)

- **Classification multiclasse** : Prédire le montant des pourboires réglés pour la course, à l’aide des classes précédemment définies.

  **Apprenant utilisé :** Régression logistique multiclasse

  a. Pour ce problème, notre cible (ou classe) est **tip\_class**, ce qui peut prendre une des cinq valeurs suivantes (0,1,2,3,4). Comme dans le cas de classification binaire, nous avons quelques colonnes qui sont des fuites cibles pour cette expérience. En particulier : **avec pourboire**, **tip\_amount** et **total\_amount** révèlent des informations sur l’étiquette cible qui ne sont pas disponibles au moment du test. Nous supprimons ces colonnes à l’aide du module [Sélectionner des colonnes dans le jeu de données][select-columns].

  Le diagramme suivant illustre l’expérience pour prédire dans quel emplacement un pourboire est susceptible de se trouver. Les emplacements sont : classe 0 : pourboire = 0 $, classe 1 : pourboire > 0 $ et pourboire <= 5 $, classe 2 : pourboire > 5 $ et pourboire <= 10 $, classe 3 : pourboire > 10 $ et pourboire <= 20 $, et classe 4 : pourboire > 20 $.

  ![Diagramme illustrant l’expérience pour prédire l’emplacement du pourboire](./media/hive-walkthrough/5ztv0n0.png)

  Nous montrons maintenant à quoi ressemble la distribution de classe test réelle. La classe 0 et la classe 1 prévalent, et les autres classes sont rares.

  ![Graphique de distribution de classe de test](./media/hive-walkthrough/Vy1FUKa.png)

  b. Pour cette expérience, nous utilisons une matrice de confusion pour consulter la précision des prédictions comme indiqué ici :

  ![Matrice de confusion](./media/hive-walkthrough/cxFmErM.png)

  Même si les précisions des classes sur les classes les plus courantes sont bonnes, le modèle n’effectue pas un bon travail « d’apprentissage » sur les classes plus rares.

- **Tâche de régression** : Prédire le montant du pourboire versé pour une course.

  **Apprenant utilisé :** Arbre de décision optimisé

  a. Pour ce problème, l’étiquette (ou classe) cible est **tip\_amount**. Les fuites cibles dans ce cas sont : **avec pourboire**, **tip\_class** et **total\_amount**. Toutes ces variables révèlent des informations sur le montant du pourboire, qui sont en général indisponibles au moment du test. Nous supprimons ces colonnes à l’aide du module [Sélectionner des colonnes dans le jeu de données][select-columns].

  Le diagramme suivant illustre l’expérience pour prédire la quantité de pourboire donné :

  ![Diagramme illustrant l’expérience pour prédire le montant du pourboire](./media/hive-walkthrough/11TZWgV.png)

  b. Pour les problèmes de régression, nous évaluons la précision des prédictions en examinant l’erreur au carré dans les prédictions et le coefficient de détermination :

  ![Capture d’écran de statistiques de prédiction](./media/hive-walkthrough/Jat9mrz.png)

  Ici, le coefficient de détermination est de 0,709, ce qui signifie que 71 pourcent environ de la variance est expliquée par les coefficients modèles.

> [!IMPORTANT]
> Pour en savoir plus sur Machine Learning, comment y accéder et comment l’utiliser, consultez [Qu’est-ce que Machine Learning ?](../classic/index.yml). La [galerie Azure AI](https://gallery.cortanaintelligence.com/) couvre en outre une large gamme d’expériences et fournit une présentation approfondie des fonctionnalités de Machine Learning.
> 
> 

## <a name="license-information"></a>Informations de licence
Ce didacticiel et ses scripts associés sont partagés par Microsoft sous la licence MIT. Pour en savoir plus, consultez le fichier **LICENSE.txt** figurant dans le répertoire de l’exemple de code sur GitHub.

## <a name="references"></a>References
•    [Page de téléchargement des jeux de données NYC Taxi Trips par Andrés Monroy (en anglais)](https://www.andresmh.com/nyctaxitrips/)  
•    [Page de partage des données relatives aux courses en taxi new-yorkais par Chris Whong (en anglais)](https://chriswhong.com/open-data/foil_nyc_taxi/)   
•    [Page de recherche et de statistiques de la Commission des services de taxis et de limousines de la ville de New York (en anglais)](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[2]: ./media/hive-walkthrough/output-hive-results-3.png
[11]: ./media/hive-walkthrough/hive-reader-properties.png
[12]: ./media/hive-walkthrough/binary-classification-training.png
[13]: ./media/hive-walkthrough/create-scoring-experiment.png
[14]: ./media/hive-walkthrough/binary-classification-scoring.png
[15]: ./media/hive-walkthrough/amlreader.png

<!-- Module References -->
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[import-data]: /azure/machine-learning/studio-module-reference/import-data