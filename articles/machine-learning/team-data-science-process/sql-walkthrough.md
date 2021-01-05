---
title: Construire et déployer un modèle dans une machine virtuelle SQL Server - Processus Team Data Science Process
description: Créez et déployez un modèle Machine Learning à l’aide de SQL Server sur une machine virtuelle Azure avec un ensemble de données disponible publiquement.
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
ms.openlocfilehash: 580181aaaea975ee07bcec8108297079c5373b92
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96007407"
---
# <a name="the-team-data-science-process-in-action-using-sql-server"></a>Processus TDSP (Team Data Science Process) en action : utilisation de SQL Server
Dans ce didacticiel, vous allez explorer le processus de création et de déploiement d’un modèle d’apprentissage automatique à l’aide de SQL Server et d’un jeu de données disponible publiquement, le jeu de données [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/). La procédure suit un flux de travail de science des données standard : ingérer et explorer les données, concevoir des fonctionnalités pour faciliter l’apprentissage, puis générer et déployer un modèle.

## <a name="nyc-taxi-trips-dataset-description"></a><a name="dataset"></a>Description du jeu de données NYC Taxi Trips
Les données NYC Taxi Trip se présentent sous la forme de fichiers CSV compressés d’une taille totale approximative de 20 Go (soit environ 48 Go après la décompression des fichiers), contenant plus de 173 millions de courses et le prix de chacune. Chaque enregistrement de course inclut le lieu et l’heure d’embarquement et de débarquement, le numéro de licence (du chauffeur) rendu anonyme et le numéro de médaillon (numéro d’identification unique) du taxi. Les données portent sur toutes les courses effectuées en 2013 et sont fournies dans les deux jeux de données ci-après pour chaque mois :

1. Le fichier CSV trip_data contient les détails de chaque course, comme le nombre de passagers, les points d’embarquement et de débarquement, la durée du trajet et la distance parcourue. Voici quelques exemples d’enregistrements :
   
    `medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

    `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

2. Le fichier CSV trip_fare contient les détails du prix de chaque course, comme le type de paiement, le prix de la course, les suppléments et les taxes, les pourboires et les péages, ainsi que le montant total réglé. Voici quelques exemples d’enregistrements :
   
    `medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

    `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

La clé unique permettant de joindre trip\_data et trip\_fare se compose des champs suivants : medallion (médaillon), hack\_licence (licence de taxi) et pickup\_datetime (date et heure d’embarquement).

## <a name="examples-of-prediction-tasks"></a><a name="mltasks"></a>Exemples de tâches de prédiction
Nous allons formuler trois problèmes de prédiction reposant sur le champ *tip\_amount* (montant du pourboire), à savoir :

* Classification binaire : Prédire si un pourboire a ou non été versé pour une course ; autrement dit, une valeur *tip\_amount* supérieure à 0 $ constitue un exemple positif, alors qu’une valeur *tip\_amount* de 0 $ est un exemple négatif.
* Classification multiclasse : Prédire la fourchette des pourboires versés pour une course. Nous divisons la valeur *tip\_amount* en cinq compartiments ou classes :

   `Class 0 : tip_amount = $0`

   `Class 1 : tip_amount > $0 and tip_amount <= $5`

   `Class 2 : tip_amount > $5 and tip_amount <= $10`

   `Class 3 : tip_amount > $10 and tip_amount <= $20`

   `Class 4 : tip_amount > $20`

* Tâche de régression : Prédire le montant du pourboire versé pour une course.  

## <a name="setting-up-the-azure-data-science-environment-for-advanced-analytics"></a><a name="setup"></a>Configuration de l’environnement de science des données Azure pour l’analyse avancée
Comme vous l’explique le guide [Planifier votre environnement](plan-your-environment.md) , vous disposez de plusieurs options pour travailler sur le jeu de données NYC Taxi Trips dans Azure :

* manipuler les données dans des objets blob Azure, puis les modéliser dans Azure Machine Learning ;
* charger les données dans une base de données SQL Server, puis les modéliser dans Azure Machine Learning.

Dans ce tutoriel, nous allons décrire l’importation en parallèle de données en bloc dans une instance SQL Server, l’exploration des données, la conception de fonctionnalités et le sous-échantillonnage avec SQL Server Management Studio et IPython Notebook. Des [exemples de scripts](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) et de [notebooks IPython](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/iPythonNotebooks) sont publiquement accessibles dans GitHub. Un exemple de notebook IPython permettant de travailler sur les données d’objets blob d’Azure est également disponible au même emplacement.

Pour configurer votre environnement de science des données Azure :

1. [Créez un compte de stockage](../../storage/common/storage-account-create.md)
2. [Création d’un espace de travail Microsoft Azure Machine Learning](../classic/create-workspace.md)
3. [Approvisionnez une machine virtuelle de science des données](../data-science-virtual-machine/overview.md), qui fournit un serveur SQL Server et un serveur Notebook IPython.
   
   > [!NOTE]
   > Les exemples de scripts et de notebooks IPython seront téléchargés sur votre machine virtuelle de science des données lors du processus de configuration. À l’issue du script de post-installation de la machine virtuelle, les exemples figureront dans la bibliothèque Documents de votre machine virtuelle :  
   > 
   > * Exemples de scripts : `C:\Users\<user_name>\Documents\Data Science Scripts`  
   > * Exemples de notebooks IPython : `C:\Users\<user_name>\Documents\IPython Notebooks\DataScienceSamples`  
   >   where `<user_name>` est le nom de connexion de votre machine virtuelle Windows. Nous désignerons les dossiers d’exemples sous la forme **Exemples de scripts** et **Exemples de notebooks IPython**.
   > 
   > 

Selon la taille du jeu de données, l’emplacement source des données et l’environnement cible Azure sélectionné, ce scénario est semblable au [Scénario \#5 : Jeu de données volumineux dans des fichiers locaux, ciblant SQL Server dans une machine virtuelle Azure](plan-sample-scenarios.md#largelocaltodb).

## <a name="get-the-data-from-public-source"></a><a name="getdata"></a>Récupérer les données de la source publique
Pour récupérer le jeu de données [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/) depuis son emplacement public, vous pouvez utiliser l’une des méthodes décrites dans l’article [Déplacer des données vers et depuis le stockage d’objets blob Azure](move-azure-blob.md) afin de copier les données dans votre nouvelle machine virtuelle.

Pour copier les données à l’aide d’AzCopy :

1. Connectez-vous à votre machine virtuelle.
2. Créez un répertoire dans le disque de données de la machine virtuelle (remarque : n’utilisez pas le disque temporaire fourni avec la machine virtuelle comme disque de données).
3. Dans une fenêtre d’invite de commandes, exécutez la ligne de commande Azcopy ci-après en remplaçant la chaîne <chemin_dossier_de_données> par votre dossier de données créé à l’étape 2 :

    ```console
    "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:https://nyctaxitrips.blob.core.windows.net/data /Dest:<path_to_data_folder> /S
    ```

    Une fois AzCopy exécuté, le dossier de données doit contenir 24 fichiers CSV compressés (12 pour trip\_data et 12 pour trip\_fare).
4. Décompressez les fichiers téléchargés. Notez le dossier dans lequel résident les fichiers décompressés. Ce dossier sera désigné par la chaîne <chemin\_des\_fichiers\_de_données\>.

## <a name="bulk-import-data-into-sql-server-database"></a><a name="dbload"></a>Importer en bloc les données dans une base de données SQL Server
Les performances de chargement et de transfert de grandes quantités de données dans une base de données SQL et les requêtes ultérieures peuvent être optimisées avec des *tables et de vues partitionnées*. Dans cette section, nous allons suivre les instructions de l’article [Importations de données en bloc en parallèle à l’aide de tables de partition SQL](parallel-load-sql-partitioned-tables.md) pour créer une base de données et charger les données dans des tables partitionnées en parallèle.

1. Pendant que vous êtes connecté à votre machine virtuelle, démarrez **SQL Server Management Studio**.
2. Connectez-vous à l’aide de l’authentification Windows.
   
    ![Connecter SSMS][12]
3. Si vous n’avez pas encore modifié le mode d’authentification de SQL Server et créé un utilisateur de connexion SQL, ouvrez le fichier de script appelé **change\_auth.sql** dans le dossier **Exemples de scripts**. Changez le nom d’utilisateur et le mot de passe par défaut. Cliquez sur **Exécuter** dans la barre d’outils pour exécuter le script.
   
    ![Exécuter le script][13]
4. Vérifiez et/ou modifiez les dossiers de base de données et de journaux par défaut de SQL Server pour vous assurer que les bases de données nouvellement créées seront stockées dans un disque de données. L’image de machine virtuelle SQL Server optimisée pour les charges d’entreposage de données est préconfigurée avec des disques de données et de journaux. Si votre machine virtuelle ne comportait aucun disque de données et que vous avez ajouté des disques durs virtuels lors du processus de configuration de la machine virtuelle, modifiez les dossiers par défaut en procédant comme suit :
   
   * Cliquez avec le bouton droit sur le nom du serveur SQL Server dans le panneau de gauche, puis cliquez sur **Propriétés**.
     
       ![Propriétés SQL Server][14]
   * Dans la liste **Sélectionner une page** figurant sur la gauche, sélectionnez **Paramètres de la base de données**.
   * Vérifiez et/ou redéfinissez le champ **Emplacements de la base de données par défaut** sur les emplacements **Disque de données** de votre choix. Cet emplacement est l’endroit dans lequel réside les nouvelles bases de données si elles ont été créées avec les paramètres par défaut.
     
       ![Valeurs par défaut de la base de données SQL][15]  
5. Pour créer une base de données et un ensemble de groupes de fichiers destinés à contenir les tables partitionnées, ouvrez l’exemple de script **create\_db\_default.sql**. Ce script créera une base de données nommée **TaxiNYC** et 12 groupes de fichiers à l’emplacement des données par défaut. Chaque groupe de fichiers contiendra un mois de données trip\_data et trip\_fare. Si vous le souhaitez, modifiez le nom de la base de données. Cliquez sur **Exécuter** pour exécuter le script.
6. Créez ensuite deux tables de partition, l’une pour les données trip\_data et l’autre pour les données trip\_fare. Ouvrez l’exemple de script **create\_partitioned\_table.sql**, qui va :
   
   * Créer une fonction de partition pour fractionner les données par mois.
   * Créer un schéma de partition pour mapper les données de chaque mois sur un groupe de fichiers distinct.
   * Créer deux tables partitionnées mappées sur le schéma de partition : **nyctaxi\_trip** contiendra les données de trip\_data et **nyctaxi\_fare** contiendra les données de trip\_fare.
     
     Cliquez sur **Exécuter** pour exécuter le script et créer les tables partitionnées.
7. Le dossier **Exemples de scripts** comporte deux exemples de scripts PowerShell présentant les procédures d’importations de données en bloc en parallèle dans des tables SQL Server.
   
   * **bcp\_parallel\_generic.ps1** est un script générique permettant d’effectuer des importations de données en bloc en parallèle dans une table. Modifiez ce script pour définir les variables d’entrée et les variables cibles, comme indiqué dans les lignes de commentaire du script.
   * **bcp\_parallel\_nyctaxi.ps1** est une version préconfigurée du script générique que vous pouvez utiliser pour charger les deux tables relatives aux données NYC Taxi Trips.  
8. Cliquez avec le bouton droit sur le nom de script **bcp\_parallel\_nyctaxi.ps1**, puis cliquez sur **Modifier** pour l’ouvrir dans PowerShell. Examinez les variables prédéfinies et modifiez-les en fonction des valeurs que vous avez sélectionnées pour le nom de base de données, le dossier de données d’entrée, le dossier de journaux cibles et les chemins des exemples de fichiers de format **nyctaxi_trip.xml** et **nyctaxi\_fare.xml** (fournis dans le dossier **Exemples de scripts**).
   
    ![Importer les données][16]
   
    Vous pouvez également sélectionner le mode d’authentification (le mode par défaut est l’authentification Windows). Cliquez sur la flèche de couleur verte située dans la barre d’outils pour exécuter le script. Le script lancera 24 opérations d’importation en bloc en parallèle, 12 par table partitionnée. Vous pouvez suivre la progression de l’importation des données en ouvrant le dossier de données par défaut de SQL Server tel que défini ci-dessus.
9. Le script PowerShell indique les heures de début et de fin. Une fois toutes les importations en bloc effectuées, l’heure de fin est spécifiée. Consultez le dossier de journaux cibles pour vous assurer que les importations en bloc se sont déroulées sans erreur.
10. Votre base de données est désormais prête pour vos opérations d’exploration, de conception de fonctionnalités, etc. Les tables étant partitionnées en fonction du champ **pickup\_datetime**, les requêtes qui incluent des conditions **pickup\_datetime** dans la clause **WHERE** tireront avantage du schéma de partition.
11. Dans **SQL Server Management Studio**, explorez l’exemple de script **sample\_queries.sql** fourni. Pour exécuter l’un des exemples de requêtes, mettez en surbrillance les lignes de la requête concernées, puis cliquez sur **Exécuter** dans la barre d’outils.
12. Les données NYC Taxi Trips sont chargées dans deux tables distinctes. Pour améliorer les opérations de jointure, il est vivement recommandé d’indexer les tables. L’exemple de script **create\_partitioned\_index.sql** crée des index partitionnés sur la clé de jointure composite **medallion, hack\_license et pickup\_datetime**.

## <a name="data-exploration-and-feature-engineering-in-sql-server"></a><a name="dbexplore"></a>Exploration des données et conception de fonctionnalités dans SQL Server
Dans cette section, nous allons effectuer des tâches d’exploration des données et de génération de fonctionnalités en exécutant des requêtes SQL directement dans **SQL Server Management Studio** à l’aide de la base de données SQL Server créée précédemment. Le dossier **Exemples de scripts** comporte un exemple de script nommé **sample\_queries.sql**. Modifiez ce script de façon à changer le nom de la base de données s’il est différent du nom par défaut : **TaxiNYC**.

Dans cet exercice, nous allons :

* Nous connecter à **SQL Server Management Studio** à l’aide de l’authentification Windows ou SQL et des nom et mot de passe de connexion SQL.
* explorer les distributions de données de quelques champs portant sur différentes périodes ;
* examiner la qualité des données des champs de longitude et de latitude ;
* Générer des étiquettes de classification binaire et multiclasse reposant sur la valeur **tip\_amount**.
* Générer des fonctionnalités et calculer/comparer les distances des trajets.
* Joindre les deux tables et extraire un échantillon aléatoire que nous utiliserons pour créer les modèles.

Lorsque vous êtes prêt à utiliser Azure Machine Learning, vous pouvez au choix :  

1. enregistrer la requête SQL finale d’extraction et d’échantillonnage des données et copier-coller cette requête directement dans un module [Importer les données][import-data] d’Azure Machine Learning, ou
2. stocker les données échantillonnées et générées que vous prévoyez d’utiliser pour la création de modèles dans une nouvelle table de base de données et utiliser cette table dans le module [Importer les données][import-data] d’Azure Machine Learning.

Dans cette section, nous allons enregistrer la requête finale d’extraction et d’échantillonnage des données. La seconde méthode est décrite à la section [Exploration des données et conception de fonctionnalités dans Notebook IPython](#ipnb) .

Pour vérifier rapidement le nombre de lignes et de colonnes des tables précédemment remplies à l’aide des importations en bloc en parallèle, procédez comme suit :

- Afficher le nombre de lignes dans la table <nyctaxi_trip> sans analyse de table : `SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('nyctaxi_trip')`
- Afficher le nombre de colonnes de la table <nyctaxi_trip> : `SELECT COUNT(*) FROM information_schema.columns WHERE table_name = 'nyctaxi_trip'`

#### <a name="exploration-trip-distribution-by-medallion"></a>Exploration : Distribution des courses par médaillon
Cet exemple identifie le médaillon (numéro de taxi) sur plus de 100 courses au cours d’une période donnée. Cette requête tire avantage de l’accès aux tables partitionnées, car elle est conditionnée par le schéma de partition de **pickup\_datetime**. L’exécution d’une requête portant sur le jeu de données complet tire également profit de l’analyse d’index et/ou de table partitionnée.

```sql
SELECT medallion, COUNT(*)
FROM nyctaxi_fare
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
GROUP BY medallion
HAVING COUNT(*) > 100
```

#### <a name="exploration-trip-distribution-by-medallion-and-hack_license"></a>Exploration : Distribution des courses par médaillon et par licence de taxi

```sql
SELECT medallion, hack_license, COUNT(*)
FROM nyctaxi_fare
WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
GROUP BY medallion, hack_license
HAVING COUNT(*) > 100
```

#### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>Évaluation de la qualité des données : Vérifier les enregistrements indiquant une longitude et/ou une latitude incorrectes
Cet exemple vérifie si l’un des champs de longitude et/ou de latitude contient une valeur incorrecte (le nombre de degrés doit être compris entre -90 et 90) ou présente des coordonnées (0, 0).

```sql
SELECT COUNT(*) FROM nyctaxi_trip
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
OR    (pickup_longitude = '0' AND pickup_latitude = '0')
OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))
```

#### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>Exploration : Pourboire payé et avec et sans pourboire
Cet exemple recherche le nombre de courses avec et sans pourboire sur une période donnée (ou dans le jeu de données complet si la requête porte sur l’année entière). Cette distribution reflète la distribution des étiquettes binaires à utiliser par la suite pour la modélisation de classification binaire.

```sql
SELECT tipped, COUNT(*) AS tip_freq FROM (
  SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
  FROM nyctaxi_fare
  WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tipped
```

#### <a name="exploration-tip-classrange-distribution"></a>Exploration : Distribution des classes/fourchettes de pourboires
Cet exemple calcule la distribution des fourchettes de pourboires sur une période donnée (ou dans le jeu de données complet si la requête porte sur l’année entière). Cette distribution des classes d’étiquette sera utilisée par la suite pour la modélisation de classification multiclasse.

```sql
SELECT tip_class, COUNT(*) AS tip_freq FROM (
    SELECT CASE
        WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM nyctaxi_fare
WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tip_class
```

#### <a name="exploration-compute-and-compare-trip-distance"></a>Exploration : Calculer et comparer les distances des courses
Cet exemple convertit la longitude et la latitude des points d’embarquement et de débarquement en points géographiques SQL, calcule la distance des trajets en se basant sur la différence entre ces points géographiques et renvoie un échantillon aléatoire des résultats pour comparaison. Cet exemple limite les résultats aux coordonnées valides en utilisant la requête d’évaluation de la qualité des données précédemment décrite.

```sql
SELECT
pickup_location=geography::STPointFromText('POINT(' + pickup_longitude + ' ' + pickup_latitude + ')', 4326)
,dropoff_location=geography::STPointFromText('POINT(' + dropoff_longitude + ' ' + dropoff_latitude + ')', 4326)
,trip_distance
,computedist=round(geography::STPointFromText('POINT(' + pickup_longitude + ' ' + pickup_latitude + ')', 4326).STDistance(geography::STPointFromText('POINT(' + dropoff_longitude + ' ' + dropoff_latitude + ')', 4326))/1000, 2)
FROM nyctaxi_trip
tablesample(0.01 percent)
WHERE CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND   CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

#### <a name="feature-engineering-in-sql-queries"></a>Conception de fonctionnalités dans des requêtes SQL
Vous pouvez également utiliser les requêtes de génération d’étiquettes et de conversion de données géographiques pour générer des étiquettes/fonctionnalités en supprimant la partie décompte. D’autres exemples SQL de conception de fonctionnalités sont fournis à la section [Exploration des données et conception de fonctionnalités dans Notebook IPython](#ipnb) . L’exécution des requêtes de génération de fonctionnalités se révèle plus efficace sur la totalité du jeu de données ou sur un large sous-échantillon de ce jeu par le biais de requêtes SQL qui s’exécutent directement sur l’instance de base de données SQL Server. Les requêtes sont exécutables dans **SQL Server Management Studio**, dans IPython Notebook ou dans tout autre outil/environnement de développement pouvant accéder à la base de données localement ou à distance.

#### <a name="preparing-data-for-model-building"></a>Préparation des données pour la création de modèles
La requête ci-après joint les tables **nyctaxi\_trip** et **nyctaxi\_fare**, génère une étiquette de classification binaire **tipped** et une étiquette de classification multiclasse **tip\_class**, puis extrait un échantillon aléatoire de 1 % des données de l’intégralité du jeu de données joint. Vous pouvez ensuite copier cette requête et la coller directement dans le module [Importer les données][import-data] d’[Azure Machine Learning Studio](https://studio.azureml.net) pour permettre l’ingestion directe des données de l’instance de base de données SQL Server dans Azure. La requête exclut les enregistrements qui présentent des coordonnées (0, 0) incorrectes.

```sql
SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
    CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
    CASE WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM nyctaxi_trip t, nyctaxi_fare f
TABLESAMPLE (1 percent)
WHERE t.medallion = f.medallion
AND   t.hack_license = f.hack_license
AND   t.pickup_datetime = f.pickup_datetime
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

## <a name="data-exploration-and-feature-engineering-in-ipython-notebook"></a><a name="ipnb"></a>Exploration des données et conception de fonctionnalités dans Notebook IPython
Dans cette section, nous allons effectuer des tâches d’exploration des données et de génération de fonctionnalités en exécutant des requêtes Python et SQL dans la base de données SQL Server créée précédemment. Le dossier **Exemples de notebooks IPython** comporte un exemple de notebook IPython nommé **machine-Learning-data-science-process-sql-story.ipynb**. Ce notebook est également disponible sur [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/iPythonNotebooks).

Lorsque vous travaillez sur des données volumineuses, suivez ces étapes conseillées :

* Lisez un petit échantillon des données dans une trame de données en mémoire.
* Procédez à certaines visualisations et explorations à l’aide des données échantillonnées.
* Expérimentez la conception de fonctionnalités en utilisant les données échantillonnées.
* Pour exécuter de plus vastes tâches d’exploration des données, de manipulation des données et de conception de fonctionnalités, utilisez Python pour émettre des requêtes SQL directement sur la base de données SQL Server dans la machine virtuelle Azure.
* Déterminez la taille d’échantillon à utiliser pour la création de modèles Azure Machine Learning.

Lorsque vous êtes prêt à utiliser Azure Machine Learning, vous pouvez effectuer l’une des deux opérations suivantes :  

1. enregistrer la requête SQL finale d’extraction et d’échantillonnage des données et copier-coller cette requête directement dans un module [Importer les données][import-data] d’Azure Machine Learning. Cette méthode est décrite dans la section [Création de modèles dans Azure Machine Learning](#mlmodel) .    
2. Stocker les données échantillonnées et générées que vous prévoyez d’utiliser pour la création de modèles dans une nouvelle table de base de données et utiliser cette table dans le module [Importer les données][import-data].

Vous trouverez ci-dessous quelques exemples d’exploration des données, de visualisation des données et de conception de fonctionnalités. Pour découvrir d’autres exemples, reportez-vous à l’exemple de notebook IPython SQL figurant dans le dossier **Exemples de notebooks IPython** .

#### <a name="initialize-database-credentials"></a>Initialiser les informations d’identification de la base de données
Initialisez vos paramètres de connexion à la base de données dans les variables suivantes :

```sql
SERVER_NAME=<server name>
DATABASE_NAME=<database name>
USERID=<user name>
PASSWORD=<password>
DB_DRIVER = <database server>
```

#### <a name="create-database-connection"></a>Créer une connexion à la base de données

```sql
CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
conn = pyodbc.connect(CONNECTION_STRING)
```

#### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_trip"></a>Afficher le nombre de lignes et de colonnes de la table nyctaxi_trip

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('nyctaxi_trip')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('nyctaxi_trip')
''', conn)

print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Nombre total de lignes = 173 179 759  
* Nombre total de colonnes = 14

#### <a name="read-in-a-small-data-sample-from-the-sql-server-database"></a>Lire un petit échantillon de données depuis la base de données SQL Server

```sql
t0 = time.time()

query = '''
    SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
        f.tolls_amount, f.total_amount, f.tip_amount
    FROM nyctaxi_trip t, nyctaxi_fare f
    TABLESAMPLE (0.05 PERCENT)
    WHERE t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
'''

df1 = pd.read_sql(query, conn)

t1 = time.time()
print 'Time to read the sample table is %f seconds' % (t1-t0)

print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])
```

Temps de lecture de la table d’échantillon = 6,492000 secondes  
Nombre de lignes et de colonnes récupérées = (84 952, 21)

#### <a name="descriptive-statistics"></a>Statistiques descriptives
Nous sommes désormais prêts à explorer les données échantillonnées. Nous sommes prêts à examiner les statistiques descriptives relatives au champ **trip\_distance** (ou à tout autre champ) :

```sql
df1['trip_distance'].describe()
```

#### <a name="visualization-box-plot-example"></a>Visualisation : Exemple de diagramme à surfaces
Nous examinons ensuite le diagramme à surfaces concernant la distance des trajets afin de visualiser les quantiles.

```sql
df1.boxplot(column='trip_distance',return_type='dict')
```

![Diagramme #1][1]

#### <a name="visualization-distribution-plot-example"></a>Visualisation : Exemple de diagramme de distribution

```sql
fig = plt.figure()
ax1 = fig.add_subplot(1,2,1)
ax2 = fig.add_subplot(1,2,2)
df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
df1['trip_distance'].hist(ax=ax2, bins=100, color='k')
```

![Diagramme #2][2]

#### <a name="visualization-bar-and-line-plots"></a>Visualisation : Diagrammes en bâtons et linéaires
Dans cet exemple, nous compartimentons la distance des trajets en cinq zones et nous visualisons les résultats de cette opération.

```sql
trip_dist_bins = [0, 1, 2, 4, 10, 1000]
df1['trip_distance']
trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
trip_dist_bin_id
```

Nous pouvons représenter la distribution des compartiments ci-dessus dans un diagramme en bâtons ou dans un diagramme linéaire en procédant comme ci-dessous.

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')
```

![Diagramme #3][3]

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')
```
![Diagramme #4][4]

#### <a name="visualization-scatterplot-example"></a>Visualisation : Exemple de diagramme de dispersion
Nous représentons le diagramme de dispersion entre **trip\_time\_in\_secs** et **trip\_distance** pour déterminer s’il existe une corrélation.

```sql
plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])
```

![Diagramme #6][6]

De la même façon, nous pouvons vérifier la relation entre **rate\_code** et **trip\_distance**.

```sql
plt.scatter(df1['passenger_count'], df1['trip_distance'])
```

![Diagramme #8][8]

### <a name="sub-sampling-the-data-in-sql"></a>Sous-échantillonnage des données dans SQL
Quand vous préparez les données pour la création de modèles dans [Azure Machine Learning Studio](https://studio.azureml.net), vous pouvez soit utiliser la **requête SQL directement dans le module Importer les données**, soit stocker les données générées et échantillonnées dans une nouvelle table que vous pouvez utiliser dans le module [Importer les données][import-data] à l’aide d’une simple requête **SELECT * FROM <nouveau\_nom\_de\_votre_table>** .

Dans cette section, nous allons créer une table destinée à contenir les données échantillonnées et générées. Un exemple de requête SQL directe pour la création de modèles est fourni à la section [Exploration des données et conception de fonctionnalités dans SQL Server](#dbexplore) .

#### <a name="create-a-sample-table-and-populate-with-1-of-the-joined-tables-drop-table-first-if-it-exists"></a>Créer une table d’échantillon et la remplir avec 1 % des données des tables jointes en commençant par supprimer la table si elle existe
Dans cette section, nous allons joindre les tables **nyctaxi\_trip** et **nyctaxi\_fare**, extraire un échantillon aléatoire de 1 % des données et stocker les données échantillonnées dans une nouvelle table nommée **nyctaxi\_one\_percent** :

```sql
cursor = conn.cursor()

drop_table_if_exists = '''
    IF OBJECT_ID('nyctaxi_one_percent', 'U') IS NOT NULL DROP TABLE nyctaxi_one_percent
'''

nyctaxi_one_percent_insert = '''
    SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount
    INTO nyctaxi_one_percent
    FROM nyctaxi_trip t, nyctaxi_fare f
    TABLESAMPLE (1 PERCENT)
    WHERE t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
    AND   pickup_longitude <> '0' AND dropoff_longitude <> '0'
'''

cursor.execute(drop_table_if_exists)
cursor.execute(nyctaxi_one_percent_insert)
cursor.commit()
```

### <a name="data-exploration-using-sql-queries-in-ipython-notebook"></a>Exploration des données à l’aide de requêtes SQL dans Notebook IPython
Dans cette section, nous allons explorer les distributions de données à l’aide de l’échantillon de 1 % des données que nous avons stocké dans la table créée ci-dessus. Vous pouvez effectuer des explorations similaires à l’aide des tables d’origine, le cas échéant en utilisant **TABLESAMPLE** pour limiter l’échantillon d’exploration ou en restreignant les résultats à une période spécifique au moyen des partitions **pickup\_datetime**, comme décrit à la section [Exploration des données et conception de fonctionnalités dans SQL Server](#dbexplore).

#### <a name="exploration-daily-distribution-of-trips"></a>Exploration : Distribution quotidienne des courses

```sql
query = '''
    SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
    FROM nyctaxi_one_percent
    GROUP BY CONVERT(date, dropoff_datetime)
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-per-medallion"></a>Exploration : Distribution des courses par médaillon

```sql
query = '''
    SELECT medallion,count(*) AS c
    FROM nyctaxi_one_percent
    GROUP BY medallion
'''

pd.read_sql(query,conn)
```

### <a name="feature-generation-using-sql-queries-in-ipython-notebook"></a>Génération de fonctionnalités à l’aide de requêtes SQL dans Notebook IPython
Dans cette section, nous allons générer de nouvelles étiquettes et fonctionnalités directement par le biais de requêtes SQL, en utilisant la table d’échantillon de 1 % des données que nous avons créée à la section précédente.

#### <a name="label-generation-generate-class-labels"></a>Génération d’étiquettes : Générer des étiquettes de classe
Dans l’exemple suivant, nous générons deux jeux d’étiquettes à utiliser pour la modélisation :

1. Étiquettes de classe binaires **tipped** (prédisant si un pourboire va être versé)
2. Étiquettes multiclasses **tip\_class** (prédisant le compartiment ou la fourchette du pourboire).

```sql   
    nyctaxi_one_percent_add_col = '''
        ALTER TABLE nyctaxi_one_percent ADD tipped bit, tip_class int
    '''
   
    cursor.execute(nyctaxi_one_percent_add_col)
    cursor.commit()
   
    nyctaxi_one_percent_update_col = '''
        UPDATE nyctaxi_one_percent
        SET
           tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
           tip_class = CASE WHEN (tip_amount = 0) THEN 0
                            WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                            WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                            WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                            ELSE 4
                        END
    '''
   
    cursor.execute(nyctaxi_one_percent_update_col)
    cursor.commit()
```

#### <a name="feature-engineering-count-features-for-categorical-columns"></a>Conception de fonctionnalités : Fonctionnalités de décompte pour les colonnes catégorielles
Cet exemple transforme un champ catégoriel en champ numérique en remplaçant chaque catégorie par le nombre de ses occurrences dans les données.

```sql
nyctaxi_one_percent_insert_col = '''
    ALTER TABLE nyctaxi_one_percent ADD cmt_count int, vts_count int
'''

cursor.execute(nyctaxi_one_percent_insert_col)
cursor.commit()

nyctaxi_one_percent_update_col = '''
    WITH B AS
    (
        SELECT medallion, hack_license,
            SUM(CASE WHEN vendor_id = 'cmt' THEN 1 ELSE 0 END) AS cmt_count,
            SUM(CASE WHEN vendor_id = 'vts' THEN 1 ELSE 0 END) AS vts_count
        FROM nyctaxi_one_percent
        GROUP BY medallion, hack_license
    )

    UPDATE nyctaxi_one_percent
    SET nyctaxi_one_percent.cmt_count = B.cmt_count,
        nyctaxi_one_percent.vts_count = B.vts_count
    FROM nyctaxi_one_percent A INNER JOIN B
    ON A.medallion = B.medallion AND A.hack_license = B.hack_license
'''

cursor.execute(nyctaxi_one_percent_update_col)
cursor.commit()
```

#### <a name="feature-engineering-bin-features-for-numerical-columns"></a>Conception de fonctionnalités : Fonctionnalités de compartimentage pour les colonnes numériques
Cet exemple transforme un champ numérique continu en plages de catégories prédéfinies ; autrement dit, il transforme un champ numérique en champ catégoriel.

```sql
nyctaxi_one_percent_insert_col = '''
    ALTER TABLE nyctaxi_one_percent ADD trip_time_bin int
'''

cursor.execute(nyctaxi_one_percent_insert_col)
cursor.commit()

nyctaxi_one_percent_update_col = '''
    WITH B(medallion,hack_license,pickup_datetime,trip_time_in_secs, BinNumber ) AS
    (
        SELECT medallion,hack_license,pickup_datetime,trip_time_in_secs,
        NTILE(5) OVER (ORDER BY trip_time_in_secs) AS BinNumber from nyctaxi_one_percent
    )

    UPDATE nyctaxi_one_percent
    SET trip_time_bin = B.BinNumber
    FROM nyctaxi_one_percent A INNER JOIN B
    ON A.medallion = B.medallion
    AND A.hack_license = B.hack_license
    AND A.pickup_datetime = B.pickup_datetime
'''

cursor.execute(nyctaxi_one_percent_update_col)
cursor.commit()
```

#### <a name="feature-engineering-extract-location-features-from-decimal-latitudelongitude"></a>Conception de fonctionnalités : Extraire des fonctionnalités d’emplacement des valeurs décimales de latitude/longitude
Cet exemple décompose la représentation décimale d’un champ de latitude ou de longitude en plusieurs champs d’emplacement de différentes granularités, tels que pays/région, ville, bloc, etc. Les nouveaux champs géographiques ne sont pas mappés sur des emplacements réels. Pour plus d’informations sur le mappage des emplacements associés à un géocode, consultez l’article consacré aux [Services REST de Bing Cartes](/bingmaps/rest-services/locations/find-a-location-by-point).

```sql
nyctaxi_one_percent_insert_col = '''
    ALTER TABLE nyctaxi_one_percent
    ADD l1 varchar(6), l2 varchar(3), l3 varchar(3), l4 varchar(3),
        l5 varchar(3), l6 varchar(3), l7 varchar(3)
'''

cursor.execute(nyctaxi_one_percent_insert_col)
cursor.commit()

nyctaxi_one_percent_update_col = '''
    UPDATE nyctaxi_one_percent
    SET l1=round(pickup_longitude,0)
        , l2 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 1 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),1,1) ELSE '0' END     
        , l3 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 2 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),2,1) ELSE '0' END     
        , l4 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 3 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),3,1) ELSE '0' END     
        , l5 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 4 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),4,1) ELSE '0' END     
        , l6 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 5 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),5,1) ELSE '0' END     
        , l7 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 6 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),6,1) ELSE '0' END
'''

cursor.execute(nyctaxi_one_percent_update_col)
cursor.commit()
```

#### <a name="verify-the-final-form-of-the-featurized-table"></a>Vérifiez le format final de la table affichée

```sql
query = '''SELECT TOP 100 * FROM nyctaxi_one_percent'''
pd.read_sql(query,conn)
```

Nous pouvons à présent passer aux phases de création et de déploiement de modèles dans [Azure Machine Learning](https://studio.azureml.net). Les données sont prêtes pour tous les problèmes de prédiction identifiés précédemment, à savoir :

1. Classification binaire : Prédire si un pourboire a ou non été versé pour une course.
2. Classification multiclasse : Prédire la fourchette du pourboire versé en fonction des classes précédemment définies.
3. Tâche de régression : Prédire le montant du pourboire versé pour une course.  

## <a name="building-models-in-azure-machine-learning"></a><a name="mlmodel"></a>Création de modèles dans Azure Machine Learning
Pour démarrer l’exercice de modélisation, connectez-vous à votre espace de travail Azure Machine Learning. Si vous n’avez pas encore créé d’espace de travail d’apprentissage automatique, consultez l’article [Créer un espace de travail Azure Machine Learning](../classic/create-workspace.md).

1. Pour plus d’informations sur la prise en main d’Azure Machine Learning, consultez la page [Azure Machine Learning Studio - De quoi s’agit-il ?](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
2. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net).
3. La page d’accueil de Studio permet d’accéder à une multitude d’informations, de vidéos, de didacticiels, de liens vers la documentation de référence des modules et d’autres ressources. Pour plus d’informations sur Azure Machine Learning, consultez le [Centre de documentation Azure Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/).

Une expérience de formation classique se déroule comme suit :

1. Création d’une expérience à l’aide du bouton **+NOUVEAU** .
2. Transfert des données vers Azure Machine Learning.
3. Prétraitement, transformation et manipulation des données en fonction des besoins.
4. Génération des fonctionnalités requises.
5. Fractionnement des données sous forme de jeux de données d’apprentissage/de validation/de test (ou utilisation de jeux de données distincts pour chacune de ces opérations).
6. Sélection d’un ou de plusieurs algorithmes d’apprentissage automatique en fonction du problème d’apprentissage à résoudre. Exemples : classification binaire, classification multiclasse, régression.
7. Formation d’un ou de plusieurs modèles à l’aide du jeu de données d’apprentissage.
8. Notation du jeu de données de validation à l’aide des modèles formés.
9. Évaluation des modèles afin de calculer les métriques pertinents pour le problème d’apprentissage.
10. Ajustement des modèles et sélection du meilleur modèle à déployer.

Dans cet exercice, nous avons déjà exploré et généré les données dans SQL Server, et déterminé la taille de l’échantillon à recevoir dans Azure Machine Learning. Pour créer les modèles de prédiction souhaités :

1. Récupérez les données dans Azure Machine Learning avec le module [Importer les données][import-data], disponible dans la section **Entrée et sortie des données**. Pour plus d’informations, consultez la page de référence du module [Importer les données][import-data].
   
    ![Importation de données Azure Machine Learning][17]
2. Dans le panneau **Propriétés**, sélectionnez **Base de données Azure SQL** dans le champ **Source de données**.
3. Dans le champ **Nom du serveur de base de données** , entrez le nom DNS de la base de données. Format : `tcp:<your_virtual_machine_DNS_name>,1433`
4. Dans le champ **Nom de la base de données** , entrez le nom de la base de données.
5. Entrez le **nom d’utilisateur SQL** dans le champ **Nom de compte d’utilisateur du serveur**, et le **mot de passe** dans le champ **Mot de passe de compte d’utilisateur du serveur**.
7. Dans la zone de texte **Requête de base de données** , collez la requête qui extrait les champs de base de données nécessaires (y compris les champs calculés tels que les étiquettes) et qui sous-échantillonne les données pour obtenir la taille d’échantillon souhaitée.

La figure ci-après illustre un exemple d’expérience de classification binaire lisant les données directement dans la base de données SQL Server. Vous pouvez créer des expériences similaires pour les problèmes de classification multiclasse et de régression.

![Formation Azure Machine Learning Studio][10]

> [!IMPORTANT]
> Dans les exemples de requêtes d’extraction et d’échantillonnage de données de modélisation qui sont fournis aux sections précédentes, **toutes les étiquettes des trois exercices de modélisation sont incluses dans la requête**. Dans chacun des exercices de modélisation, une étape (obligatoire) importante consiste à **exclure** les étiquettes superflues pour les deux autres problèmes, ainsi que toute autre **fuite cible**. Par exemple, si vous avez recours à la classification binaire, utilisez l’étiquette **tipped** et excluez les champs **tip\_class**, **tip\_amount** et **total\_amount**. Les derniers champs sont des fuites cibles, car ils impliquent le pourboire versé.
> 
> Pour exclure les colonnes superflues et/ou les fuites cibles, vous pouvez utiliser le module [Sélectionner des colonnes dans le jeu de données][select-columns] ou [Modifier les métadonnées][edit-metadata]. Pour plus d’informations, consultez les pages de référence [Sélectionner des colonnes dans le jeu de données][select-columns] et [Modifier les métadonnées][edit-metadata].
> 
> 

## <a name="deploying-models-in-azure-machine-learning"></a><a name="mldeploy"></a>Déploiement de modèles dans Azure Machine Learning
Lorsque votre modèle est prêt, vous pouvez facilement le déployer sous la forme d’un service web directement à partir de l’expérience. Pour plus d’informations sur le déploiement de services web Azure Machine Learning, consultez [Déployer un service web Azure Machine Learning](../classic/deploy-a-machine-learning-web-service.md).

Pour déployer un nouveau service web, vous devez :

1. créer une expérience de notation ;
2. déployer le service web.

Pour créer une expérience de notation à partir d’une expérience de formation **terminée**, cliquez sur **CRÉER UNE EXPÉRIENCE DE NOTATION** dans la barre d’action inférieure.

![Notation Azure][18]

Azure Machine Learning va essayer de créer une expérience de notation reposant sur les composants de l’expérience d’apprentissage. Il va notamment :

1. enregistrer le modèle formé et supprimer les modules de formation de modèle ;
2. identifier un **port d’entrée** logique pour représenter le schéma de données d’entrée attendu ;
3. identifier un **port de sortie** logique pour représenter le schéma de sortie de service web attendu.

Une fois l’expérience de notation créée, passez-la en revue et ajustez-la selon vos besoins. Un ajustement classique consiste à remplacer le jeu de données d’entrée et/ou la requête par un autre jeu excluant les champs d’étiquette, étant donné que ces étiquettes ne seront pas disponibles dans le schéma lors de l’appel du service. Il est également conseillé de restreindre la taille du jeu de données d’entrée et/ou de la requête au nombre d’enregistrements suffisants pour indiquer le schéma d’entrée. Pour le port de sortie, il est courant d’exclure tous les champs d’entrée et de n’inclure que les valeurs **Étiquettes notées** et **Probabilités notées** dans la sortie à l’aide du module [Sélectionner des colonnes dans le jeu de données][select-columns].

La figure ci-après illustre un exemple d’expérience de notation. Quand vous êtes prêt à effectuer le déploiement, cliquez sur le bouton **PUBLIER LE SERVICE WEB** de la barre d’action inférieure.

![Publication Azure Machine Learning][11]

Pour résumer, ce didacticiel pas à pas vous a décrit comment créer un environnement de science des données Azure et comment manipuler un jeu de données public volumineux, depuis l’acquisition des données jusqu’au déploiement d’un service web Azure Machine Learning en passant par la formation de modèles.

### <a name="license-information"></a>Informations de licence
Cet exemple de procédure pas à pas et les scripts et notebooks IPython qui lui sont associés sont partagés par Microsoft sous licence MIT. Pour plus d’informations, consultez le fichier LICENSE.txt figurant dans le répertoire de l’exemple de code sur GitHub.

### <a name="references"></a>References
•    [Page de téléchargement des jeux de données NYC Taxi Trips par Andrés Monroy (en anglais)](https://www.andresmh.com/nyctaxitrips/)  
•    [Page de partage des données relatives aux courses en taxi new-yorkais par Chris Whong (en anglais)](https://chriswhong.com/open-data/foil_nyc_taxi/)   
•    [Page de recherche et de statistiques de la Commission des services de taxis et de limousines de la ville de New York (en anglais)](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[1]: ./media/sql-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sql-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sql-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sql-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sql-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sql-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sql-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sql-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sql-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sql-walkthrough/azuremltrain.png
[11]: ./media/sql-walkthrough/azuremlpublish.png
[12]: ./media/sql-walkthrough/ssmsconnect.png
[13]: ./media/sql-walkthrough/executescript.png
[14]: ./media/sql-walkthrough/sqlserverproperties.png
[15]: ./media/sql-walkthrough/sqldefaultdirs.png
[16]: ./media/sql-walkthrough/bulkimport.png
[17]: ./media/sql-walkthrough/amlreader.png
[18]: ./media/sql-walkthrough/amlscoring.png


<!-- Module References -->
[edit-metadata]: /azure/machine-learning/studio-module-reference/edit-metadata
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[import-data]: /azure/machine-learning/studio-module-reference/import-data