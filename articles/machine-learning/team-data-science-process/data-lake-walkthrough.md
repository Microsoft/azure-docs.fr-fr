---
title: Science des données évolutive avec Azure Data Lake - Team Data Science Process
description: Comment utiliser Azure Data Lake pour effectuer des tâches d’exploration de donnés et de classification binaire sur un jeu de données.
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
ms.openlocfilehash: e6eb0be4d9946907dc5bb2f22b27530a27a37aec
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021450"
---
# <a name="scalable-data-science-with-azure-data-lake-an-end-to-end-walkthrough"></a>Science des données évolutive avec Azure Data Lake : procédure complète
Cette procédure de bout en bout montre comment utiliser Azure Data Lake pour effectuer des tâches d’exploration de données et de classification binaire sur un échantillon de jeu de données NYC taxi trip and fare afin de prédire si le pourboire est compris dans le prix du billet. Elle vous guide tout au long du [processus de science des données pour les équipes](./index.yml)de bout en bout, depuis l’acquisition de données à l’apprentissage du modèle et au déploiement d’un service web qui publie le modèle.

## <a name="technologies"></a>Technologies

Les technologies suivantes sont utilisées lors de cette procédure pas à pas.
* Service Analytique Azure Data Lake
* U-SQL et Visual Studio
* Python
* Azure Machine Learning
* Scripts


### <a name="azure-data-lake-analytics"></a>Service Analytique Azure Data Lake
[Microsoft Azure Data Lake](https://azure.microsoft.com/solutions/data-lake/) dispose de toutes les fonctionnalités requises pour faciliter le travail des scientifiques de données relatif au stockage des données de toute taille, forme et vitesse et à la réalisation du traitement des données, d’analyses avancées et de modélisation de l’apprentissage automatique de manière hautement évolutive et à moindre coût.   Le paiement s’effectue par travail effectué, seulement lorsque les données sont réellement en cours de traitement. Azure Data Lake Analytics inclut U-SQL, un langage mêlant la nature déclarative du langage SQL à la puissance expressive de C# pour offrir une capacité de requête distribuée évolutive. Elle vous permet de traiter des données non structurées en appliquant des schémas lors de la lecture et d’intégrer une logique personnalisée et des fonctions définies par l’utilisateur (UDF), et a l’extensibilité nécessaire pour permettre un contrôle précis de l’exécution à grande échelle. Pour en savoir plus sur la philosophie de conception sous-jacente à U-SQL, consultez [Billet de blog Visual Studio](https://blogs.msdn.microsoft.com/visualstudio/2015/09/28/introducing-u-sql-a-language-that-makes-big-data-processing-easy/).

Data Lake Analytics est également un composant essentiel de Cortana Analytics Suite et fonctionne avec Azure Synapse Analytics, Power BI et Data Factory. Cette combinaison vous permet de disposer d'une plateforme complète de Big Data et d'analyse avancée dans le cloud.

Cette procédure pas à pas commence par la description de l'installation des conditions préalables et des ressources nécessaires pour effectuer les tâches du processus de science des données. Elle décrit ensuite les étapes de traitement des données avec U-SQL et conclut en montrant comment utiliser Python et Hive avec Azure Machine Learning Studio (classique) pour générer et déployer les modèles prévisionnels.

### <a name="u-sql-and-visual-studio"></a>U-SQL et Visual Studio
Cette procédure pas à pas recommande d’utiliser Visual Studio pour modifier des scripts U-SQL afin de traiter le jeu de données. Ces scripts sont décrits ici et fournis dans un fichier distinct. Cette procédure inclut l’ingestion, l’exploration et l’échantillonnage des données. Elle montre également comment exécuter un travail de script U-SQL à partir du portail Azure. Des tables Hive sont créées pour les données se trouvant dans un cluster HDInsight associé afin de faciliter la création et le déploiement d’un modèle de classification binaire dans Azure Machine Learning Studio.

### <a name="python"></a>Python
Cette procédure pas à pas contient également une section qui montre comment créer et déployer un modèle prévisionnel à l’aide de Python avec Azure Machine Learning Studio. Elle fournit un bloc-notes Jupyter avec les scripts Python pour les différentes étapes de ce processus. Le bloc-notes inclut le code de certaines étapes d’ingénierie de fonctionnalités supplémentaires et de modèles construction, comme la modélisation de la régression et de la classification multiclasse qui s’ajoute au modèle de classification binaire présenté ici. La tâche de régression consiste à prédire le montant du pourboire en fonction d’autres fonctionnalités liées au pourboire.

### <a name="azure-machine-learning"></a>Azure Machine Learning 
Azure Machine Learning Studio (classique) est utilisé pour créer et déployer les modèles prédictifs à l'aide de deux approches : d'abord avec des scripts Python, puis avec des tables Hive sur un cluster HDInsight (Hadoop).

### <a name="scripts"></a>Scripts
Seules les principales étapes sont décrites dans cette procédure pas à pas. Vous pouvez télécharger la version complète du **script U-SQL** et du **bloc-notes Jupyter** à partir de [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough).

## <a name="prerequisites"></a>Prérequis
Avant de commencer ces rubriques, vous devez disposer des éléments suivants :

* Un abonnement Azure. Si vous n’en avez pas, consultez [Obtenir une version d’évaluation gratuite Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).
* [Recommandé] Visual Studio 2013 ou ultérieur. Si vous ne disposez pas d’une de ces versions, vous pouvez télécharger une version Community gratuite depuis [Visual Studio Community](https://www.visualstudio.com/vs/community/).

> [!NOTE]
> Vous pouvez également utiliser le portail Azure au lieu de Visual Studio pour envoyer des requêtes Azure Data Lake. Vous trouverez des instructions sur la façon de procéder avec Visual Studio et sur le portail dans la section intitulée **Traiter des données avec U-SQL**.
>
>


## <a name="prepare-data-science-environment-for-azure-data-lake"></a>Préparer l’environnement de science des données pour Azure Data Lake
Pour préparer l’environnement de science des données pour cette procédure pas à pas, créez les ressources suivantes :

* Azure Data Lake Storage (ADLS)
* Azure Data Lake Analytics (ADLA)
* Compte de stockage d’objets blob Azure
* Compte Azure Machine Learning Studio (classique)
* Outils Azure Data Lake pour Visual Studio (Recommandé)

Cette section fournit des instructions sur la création de chacune de ces ressources. Si vous choisissez d’utiliser des tables Hive avec Azure Machine Learning, plutôt que Python, pour générer un modèle, vous devez également configurer un cluster HDInsight (Hadoop). Cette autre procédure est décrite dans la section Option 2.


> [!NOTE]
> **Azure Data Lake Store** peut être créé séparément ou au moment de la création d’**Azure Data Lake Analytics** comme stockage par défaut. Vous trouverez des références à des instructions sur la création de chacune des ressources. Toutefois, il n’est pas nécessaire de créer séparément le compte de stockage Data Lake.
>
>

### <a name="create-an-azure-data-lake-storage"></a>Créer une instance d'Azure Data Lake Storage


Créez un ADLS à partir du [portail Azure](https://portal.azure.com). Pour en savoir plus, consultez [Créer un cluster HDInsight avec Data Lake Store à l’aide du portail Azure](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md). Veillez à définir l’identité AAD de cluster dans le panneau **DataSource** de **Configuration facultative** décrit ici.

 ![3](./media/data-lake-walkthrough/3-create-ADLS.PNG)

### <a name="create-an-azure-data-lake-analytics-account"></a>Créer un compte Azure Data Lake Analytics
Créez un compte ADLA à partir du [portail Azure](https://portal.azure.com). Pour en savoir plus, consultez [Didacticiel : Prise en main du service Azure Data Lake Analytics à l’aide du portail Azure](../../data-lake-analytics/data-lake-analytics-get-started-portal.md).

 ![4](./media/data-lake-walkthrough/4-create-ADLA-new.PNG)

### <a name="create-an-azure-blob-storage-account"></a>Créer un compte de stockage d’objets blob Azure
Créez votre compte de stockage d’objets blob Azure à partir du [portail Azure](https://portal.azure.com). Pour en savoir plus, consultez la section Créer un compte de stockage de l'article [À propos des comptes de stockage Azure](../../storage/common/storage-account-create.md).

 ![5](./media/data-lake-walkthrough/5-Create-Azure-Blob.PNG)

### <a name="set-up-an-azure-machine-learning-studio-classic-account"></a>Configurer un compte Azure Machine Learning Studio (classique)
Inscrivez-vous/Connectez-vous à Azure Machine Learning Studio (classique) à partir de la page [Azure Machine Learning Studio](https://azure.microsoft.com/services/machine-learning/). Cliquez sur le bouton **Commencer dès maintenant** , puis choisissez un « espace de travail gratuit » ou un « espace de travail standard ». Vous pouvez maintenant créer des expériences dans Azure Machine Learning Studio.

### <a name="install-azure-data-lake-tools-recommended"></a>Installer les outils Azure Data Lake [Recommandé]
Installez les outils Azure Data Lake pour votre version de Visual Studio sur la page [Outils Azure Data Lake pour Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504).

 ![6](./media/data-lake-walkthrough/6-install-ADL-tools-VS.PNG)

Une fois l'installation terminée, ouvrez Visual Studio. L’onglet Data Lake doit apparaître dans le menu supérieur. Vos ressources Azure doivent apparaître dans le volet gauche, lorsque vous vous connectez à votre compte Azure.

 ![7](./media/data-lake-walkthrough/7-install-ADL-tools-VS-done.PNG)

## <a name="the-nyc-taxi-trips-dataset"></a>Jeu de données NYC Taxi Trips
Le jeu de données utilisé ici ([Jeu de données NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/)) est disponible au public. Les données NYC Taxi Trip sont constituées de fichiers CSV compressés d’une taille totale approximative de 20 Go (soit environ 48 Go après la décompression des fichiers), correspondant à plus de 173 millions de courses et au prix de chacune. Chaque enregistrement de course inclut le lieu et l’heure de prise en charge et de dépose, le numéro de licence (du chauffeur) rendu anonyme et le numéro de médaillon (numéro d’identification unique) du taxi. Les données portent sur toutes les courses effectuées en 2013 et sont fournies dans les deux jeux de données ci-après pour chaque mois :

Le fichier CSV trip_data contient les détails de chaque course, comme le nombre de passagers, les points d’embarquement et de débarquement, la durée du trajet et la distance parcourue. Voici quelques exemples d’enregistrements :

`medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`
`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`
`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`
`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`
`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`
`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

Le fichier CSV trip_fare contient les détails du prix de chaque course, comme le type de paiement, le prix de la course, les suppléments et les taxes, les pourboires et les péages, ainsi que le montant total réglé. Voici quelques exemples d’enregistrements :

`medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`
`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`
`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`
`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`
`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`
`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

La clé unique permettant de joindre trip\_data et trip\_fare se compose des trois champs suivants : medallion (médaillon), hack\_licence (licence de taxi) et pickup\_datetime (date et heure d’embarquement). Les fichiers CSV bruts sont accessibles à partir d'un objet blob Stockage Azure. Le script U-SQL pour cette jointure se trouve dans la section [Jointure entre des tables trip et fare](#join) .

## <a name="process-data-with-u-sql"></a>Traiter des données avec U-SQL
Les tâches de traitement des données illustrées dans cette section incluent l’ingestion, la vérification de la qualité, l’exploration et l’échantillonnage des données. Nous montrons également comment joindre des tables relatives aux courses et aux tarifs. La section finale montre comment exécuter un travail de script U-SQL à partir du portail Azure. Voici les liens vers chaque sous-section :

* [Ingestion de données : données lues à partir d’un objet blob public](#ingest)
* [Contrôles de qualité des données](#quality)
* [Exploration des données](#explore)
* [Joindre des tables relatives aux courses et aux tarifs](#join)
* [Échantillonnage des données](#sample)
* [Exécuter des travaux U-SQL](#run)

Ces scripts sont décrits ici et fournis dans un fichier distinct. Vous pouvez télécharger la version complète des **scripts U-SQL** à partir de [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough).

Pour exécuter U-SQL, ouvrez Visual Studio, cliquez sur **Fichier--> Nouveau--> Projet**, choisissez **Projet U-SQL**, nommez-le et enregistrez-le dans un dossier.

![8](./media/data-lake-walkthrough/8-create-USQL-project.PNG)

> [!NOTE]
> Il est possible d’utiliser le portail Azure pour exécuter U-SQL au lieu de Visual Studio. Vous pouvez accéder à la ressource Azure Data Lake Analytics sur le portail et envoyer des requêtes directement comme l’illustre la figure suivante :
>
>

![9](./media/data-lake-walkthrough/9-portal-submit-job.PNG)

### <a name="data-ingestion-read-in-data-from-public-blob"></a><a name="ingest"></a>Ingestion de données : données lues à partir d’un objet blob public

L’emplacement des données dans l’objet blob Azure, auquel il est fait référence sous la forme **wasb://container\_name\@blob\_storage\_account\_name.blob.core.windows.net/blob_name**, peut être extrait à l’aide de **Extractors.Csv()** . Remplacez vos propres noms de conteneur et de compte de stockage dans les scripts suivants pour container\_name\@blob\_storage\_account\_name dans l’adresse wasb. Les noms de fichiers étant au même format, il est possibile d'utiliser **trip\_data\_\{\*\}.csv** pour lire les 12 fichiers de course.

```sql
///Read in Trip data
@trip0 =
    EXTRACT
    medallion string,
    hack_license string,
    vendor_id string,
    rate_code string,
    store_and_fwd_flag string,
    pickup_datetime string,
    dropoff_datetime string,
    passenger_count string,
    trip_time_in_secs string,
    trip_distance string,
    pickup_longitude string,
    pickup_latitude string,
    dropoff_longitude string,
    dropoff_latitude string
// This is reading 12 trip data from blob
FROM "wasb://container_name@blob_storage_account_name.blob.core.windows.net/nyctaxitrip/trip_data_{*}.csv"
USING Extractors.Csv();
```

Dans la mesure où la première ligne comporte des en-têtes, vous devez les supprimer et modifier les types de colonne afin de les adapter. Vous pouvez enregistrer les données traitées dans Azure Data Lake Storage à l’aide de **swebhdfs://nom_data_lake_storage.azuredatalakestorage.net/nom_dossier/nom_fichier** ou dans un compte de stockage Blob Azure à l’aide de **wasb://container_name\@blob_storage_account_name.blob.core.windows.net/blob_name**.

```sql
// change data types
@trip =
    SELECT
    medallion,
    hack_license,
    vendor_id,
    rate_code,
    store_and_fwd_flag,
    DateTime.Parse(pickup_datetime) AS pickup_datetime,
    DateTime.Parse(dropoff_datetime) AS dropoff_datetime,
    Int32.Parse(passenger_count) AS passenger_count,
    Double.Parse(trip_time_in_secs) AS trip_time_in_secs,
    Double.Parse(trip_distance) AS trip_distance,
    (pickup_longitude==string.Empty ? 0: float.Parse(pickup_longitude)) AS pickup_longitude,
    (pickup_latitude==string.Empty ? 0: float.Parse(pickup_latitude)) AS pickup_latitude,
    (dropoff_longitude==string.Empty ? 0: float.Parse(dropoff_longitude)) AS dropoff_longitude,
    (dropoff_latitude==string.Empty ? 0: float.Parse(dropoff_latitude)) AS dropoff_latitude
FROM @trip0
WHERE medallion != "medallion";

////output data to ADL
OUTPUT @trip
TO "swebhdfs://data_lake_storage_name.azuredatalakestore.net/nyctaxi_folder/demo_trip.csv"
USING Outputters.Csv();

////Output data to blob
OUTPUT @trip
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_trip.csv"
USING Outputters.Csv();
```

De même, vous pouvez lire dans les jeux de données de prix. Cliquez avec le bouton droit sur Azure Data Lake Storage. Vous pouvez choisir de consulter vos données sous **Portail Azure --> Explorateur de données** ou dans l'**Explorateur de fichiers** de Visual Studio.

 ![10](./media/data-lake-walkthrough/10-data-in-ADL-VS.PNG)

 ![11](./media/data-lake-walkthrough/11-data-in-ADL.PNG)

### <a name="data-quality-checks"></a><a name="quality"></a>Contrôles de qualité des données
Une fois les tables relatives aux courses et aux tarifs lues, les contrôles de qualité des données peuvent être effectués de la façon suivante. Les fichiers CSV qui en résultent peuvent être exportés vers le stockage d'objets blob Azure ou vers Azure Data Lake Storage.

Recherchez le nombre de médaillons et le nombre unique de médaillons :

```sql
///check the number of medallions and unique number of medallions
@trip2 =
    SELECT
    medallion,
    vendor_id,
    pickup_datetime.Month AS pickup_month
    FROM @trip;

@ex_1 =
    SELECT
    pickup_month,
    COUNT(medallion) AS cnt_medallion,
    COUNT(DISTINCT(medallion)) AS unique_medallion
    FROM @trip2
    GROUP BY pickup_month;
    OUTPUT @ex_1
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_1.csv"
USING Outputters.Csv();
```

Recherchez les medaillons ayant plus de 100 courses :

```sql
///find those medallions that had more than 100 trips
@ex_2 =
    SELECT medallion,
           COUNT(medallion) AS cnt_medallion
    FROM @trip2
    //where pickup_datetime >= "2013-01-01t00:00:00.0000000" and pickup_datetime <= "2013-04-01t00:00:00.0000000"
    GROUP BY medallion
    HAVING COUNT(medallion) > 100;
    OUTPUT @ex_2
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_2.csv"
USING Outputters.Csv();
```

Recherchez les enregistrements non valides en termes de pickup_longitude :

```sql
///find those invalid records in terms of pickup_longitude
@ex_3 =
    SELECT COUNT(medallion) AS cnt_invalid_pickup_longitude
    FROM @trip
    WHERE
    pickup_longitude <- 90 OR pickup_longitude > 90;
    OUTPUT @ex_3
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_3.csv"
USING Outputters.Csv();
```

Recherchez les valeurs manquantes pour certaines variables :

```sql
//check missing values
@res =
    SELECT *,
           (medallion == null? 1 : 0) AS missing_medallion
    FROM @trip;

@trip_summary6 =
    SELECT
        vendor_id,
    SUM(missing_medallion) AS medallion_empty,
    COUNT(medallion) AS medallion_total,
    COUNT(DISTINCT(medallion)) AS medallion_total_unique
    FROM @res
    GROUP BY vendor_id;
OUTPUT @trip_summary6
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_16.csv"
USING Outputters.Csv();
```

### <a name="data-exploration"></a><a name="explore"></a>Exploration des données
Effectuez des explorations de données avec les scripts suivants pour obtenir une meilleure compréhension des données.

Recherchez la répartition des courses avec et sans pourboire :

```sql
///tipped vs. not tipped distribution
@tip_or_not =
    SELECT *,
           (tip_amount > 0 ? 1: 0) AS tipped
    FROM @fare;

@ex_4 =
    SELECT tipped,
           COUNT(*) AS tip_freq
    FROM @tip_or_not
    GROUP BY tipped;
    OUTPUT @ex_4
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_4.csv"
USING Outputters.Csv();
```

Recherchez la répartition du montant du pourboire avec les valeurs limites : 0, 5, 10 et 20 dollars.

```sql
//tip class/range distribution
@tip_class =
    SELECT *,
           (tip_amount >20? 4: (tip_amount >10? 3:(tip_amount >5 ? 2:(tip_amount > 0 ? 1: 0)))) AS tip_class
    FROM @fare;
@ex_5 =
    SELECT tip_class,
           COUNT(*) AS tip_freq
    FROM @tip_class
    GROUP BY tip_class;
    OUTPUT @ex_5
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_5.csv"
USING Outputters.Csv();
```

Recherchez les statistiques de base de la distance de la course :

```sql
// find basic statistics for trip_distance
@trip_summary4 =
    SELECT
        vendor_id,
        COUNT(*) AS cnt_row,
        MIN(trip_distance) AS min_trip_distance,
        MAX(trip_distance) AS max_trip_distance,
        AVG(trip_distance) AS avg_trip_distance
    FROM @trip
    GROUP BY vendor_id;
OUTPUT @trip_summary4
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_14.csv"
USING Outputters.Csv();
```

Recherchez les percentiles de distance de la course :

```sql
// find percentiles of trip_distance
@trip_summary3 =
    SELECT DISTINCT vendor_id AS vendor,
                    PERCENTILE_DISC(0.25) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc,
                    PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc,
                    PERCENTILE_DISC(0.75) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc
    FROM @trip;
   // group by vendor_id;
OUTPUT @trip_summary3
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_13.csv"
USING Outputters.Csv();
```

### <a name="join-trip-and-fare-tables"></a><a name="join"></a>Joindre des tables relatives aux courses et aux tarifs
Les tables relatives aux courses et aux tarifs peuvent être jointes par médaillon, hack_license et pickup_time.

```sql
//join trip and fare table

@model_data_full =
SELECT t.*,
f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,  f.total_amount, f.tip_amount,
(f.tip_amount > 0 ? 1: 0) AS tipped,
(f.tip_amount >20? 4: (f.tip_amount >10? 3:(f.tip_amount >5 ? 2:(f.tip_amount > 0 ? 1: 0)))) AS tip_class
FROM @trip AS t JOIN  @fare AS f
ON   (t.medallion == f.medallion AND t.hack_license == f.hack_license AND t.pickup_datetime == f.pickup_datetime)
WHERE   (pickup_longitude != 0 AND dropoff_longitude != 0 );

//// output to blob
OUTPUT @model_data_full
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_7_full_data.csv"
USING Outputters.Csv();

////output data to ADL
OUTPUT @model_data_full
TO "swebhdfs://data_lake_storage_name.azuredatalakestore.net/nyctaxi_folder/demo_ex_7_full_data.csv"
USING Outputters.Csv();
```

Pour chaque niveau de nombre de passagers, calculez le nombre d’enregistrements, le montant moyen du pourboire, l’écart du montant du pourboire et le pourcentage de courses avec pourboire.

```sql
// contingency table
@trip_summary8 =
    SELECT passenger_count,
           COUNT(*) AS cnt,
           AVG(tip_amount) AS avg_tip_amount,
           VAR(tip_amount) AS var_tip_amount,
           SUM(tipped) AS cnt_tipped,
           (float)SUM(tipped)/COUNT(*) AS pct_tipped
    FROM @model_data_full
    GROUP BY passenger_count;
    OUTPUT @trip_summary8
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_17.csv"
USING Outputters.Csv();
```

### <a name="data-sampling"></a><a name="sample"></a>Échantillonnage des données
Tout d’abord, sélectionnez aléatoirement 0,1 % des données à partir de la table jointe :

```sql
//random select 1/1000 data for modeling purpose
@addrownumberres_randomsample =
SELECT *,
        ROW_NUMBER() OVER() AS rownum
FROM @model_data_full;

@model_data_random_sample_1_1000 =
SELECT *
FROM @addrownumberres_randomsample
WHERE rownum % 1000 == 0;

OUTPUT @model_data_random_sample_1_1000
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_7_random_1_1000.csv"
USING Outputters.Csv();
```

Ensuite, procédez à un échantillonnage stratifié par variable binaire tip_class :

```sql
//stratified random select 1/1000 data for modeling purpose
@addrownumberres_stratifiedsample =
SELECT *,
        ROW_NUMBER() OVER(PARTITION BY tip_class) AS rownum
FROM @model_data_full;

@model_data_stratified_sample_1_1000 =
SELECT *
FROM @addrownumberres_stratifiedsample
WHERE rownum % 1000 == 0;
//// output to blob
OUTPUT @model_data_stratified_sample_1_1000
TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_9_stratified_1_1000.csv"
USING Outputters.Csv();
////output data to ADL
OUTPUT @model_data_stratified_sample_1_1000
TO "swebhdfs://data_lake_storage_name.azuredatalakestore.net/nyctaxi_folder/demo_ex_9_stratified_1_1000.csv"
USING Outputters.Csv();
```

### <a name="run-u-sql-jobs"></a><a name="run"></a>Exécuter des travaux U-SQL
Après avoir modifié les scripts U-SQL, vous pouvez les envoyer au serveur à l'aide de votre compte Azure Data Lake Analytics. Cliquez sur **Data Lake**, **Envoyer le travail**, sélectionnez votre **Compte Analytics**, choisissez **Parallélisme**, puis cliquez sur le bouton **Envoyer**.

 ![12](./media/data-lake-walkthrough/12-submit-USQL.PNG)

Lorsque le travail est compilé avec succès, son état s’affiche dans Visual Studio pour la surveillance. Une fois le travail terminé, vous pouvez même relancer le processus d'exécution de celui-ci et découvrir les étapes à suivre pour améliorer son efficacité. Vous pouvez également accéder au portail Azure pour vérifier l’état de vos travaux U-SQL.

 ![13](./media/data-lake-walkthrough/13-USQL-running-v2.PNG)

 ![14](./media/data-lake-walkthrough/14-USQL-jobs-portal.PNG)

Vous pouvez maintenant vérifier les fichiers de sortie dans le stockage d’objets blob Azure ou sur le portail Azure. Utilisez les exemples de données stratifiées pour notre modélisation lors de l’étape suivante.

 ![15](./media/data-lake-walkthrough/15-U-SQL-output-csv.PNG)

 ![16](./media/data-lake-walkthrough/16-U-SQL-output-csv-portal.PNG)

## <a name="build-and-deploy-models-in-azure-machine-learning"></a>Créer et déployer des modèles dans Azure Machine Learning
Deux options permettant d’extraire des données vers Azure Machine Learning afin de créer et de

* La première option consiste à utiliser les données échantillonnées écrites dans un objet blob Azure (dans l’étape **Échantillonnage des données** ci-dessus) et Python pour créer et déployer des modèles à partir d’Azure Machine Learning.
* La deuxième option consiste à interroger les données d’Azure Data Lake directement à l’aide d’une requête Hive. Cette option nécessite la création d’un cluster HDInsight ou l’utilisation d’un cluster HDInsight existant, où les tables Hive pointent vers les données sur les taxis new-yorkais d’Azure Data Lake Storage.  Ces deux options sont présentées dans les sections suivantes.

## <a name="option-1-use-python-to-build-and-deploy-machine-learning-models"></a>Option 1 : Utiliser Python pour créer et déployer des modèles Machine Learning
Pour générer et déployer des modèles d’apprentissage automatique avec Python, créez un bloc-notes Jupyter sur votre ordinateur local ou dans Azure Machine Learning Studio. Le bloc-notes Jupyter Notebook fourni sur [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough) contient l'intégralité du code nécessaire à l'exploration et à la visualisation des données, ainsi qu'au développement, à la modélisation et au déploiement de fonctionnalités. Cet article aborde uniquement le déploiement et la modélisation.

### <a name="import-python-libraries"></a>Importer des bibliothèques Python
Pour exécuter l’exemple de bloc-notes Jupyter ou le fichier de script Python, vous avez besoin des packages Python suivants. Si vous utilisez le service Azure Machine Learning Notebook, ces packages ont été préinstallés.

```python
import pandas as pd
from pandas import Series, DataFrame
import numpy as np
import matplotlib.pyplot as plt
from time import time
import pyodbc
import os
from azure.storage.blob import BlobService
import tables
import time
import zipfile
import random
import sklearn
from sklearn.linear_model import LogisticRegression
from sklearn.cross_validation import train_test_split
from sklearn import metrics
from __future__ import division
from sklearn import linear_model
from azureml import services
```

### <a name="read-in-the-data-from-blob"></a>Lire les données à partir d’un objet blob
* Chaîne de connexion

  ```text
  CONTAINERNAME = 'test1'
  STORAGEACCOUNTNAME = 'XXXXXXXXX'
  STORAGEACCOUNTKEY = 'YYYYYYYYYYYYYYYYYYYYYYYYYYYY'
  BLOBNAME = 'demo_ex_9_stratified_1_1000_copy.csv'
  blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
  ```

* Lire sous forme de texte

  ```text
  t1 = time.time()
  data = blob_service.get_blob_to_text(CONTAINERNAME,BLOBNAME).split("\n")
  t2 = time.time()
  print(("It takes %s seconds to read in "+BLOBNAME) % (t2 - t1))
  ```

  ![17](./media/data-lake-walkthrough/17-python_readin_csv.PNG)

* Ajouter des noms de colonnes et des colonnes distinctes

  ```text
  colnames = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime',
  'passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude',
  'payment_type', 'fare_amount', 'surcharge', 'mta_tax', 'tolls_amount',  'total_amount', 'tip_amount', 'tipped', 'tip_class', 'rownum']
  df1 = pd.DataFrame([sub.split(",") for sub in data], columns = colnames)
  ```

* Modifier des colonnes pour les faire passer au format numérique

  ```text
  cols_2_float = ['trip_time_in_secs','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude',
  'fare_amount', 'surcharge','mta_tax','tolls_amount','total_amount','tip_amount', 'passenger_count','trip_distance'
  ,'tipped','tip_class','rownum']
  for col in cols_2_float:
      df1[col] = df1[col].astype(float)
  ```

### <a name="build-machine-learning-models"></a>Créer des modèles Machine Learning
Ici, vous créez un modèle de classification binaire pour prédire s’il s’agit d’une course avec ou sans pourboire. Dans le bloc-notes Jupyter, vous trouverez deux autres modèles : un modèle de classification multiclasse et un modèle de régression.

* Vous devez d’abord créer des variables factices pouvant être utilisées dans des modèles scikit-learn

  ```python
  df1_payment_type_dummy = pd.get_dummies(df1['payment_type'], prefix='payment_type_dummy')
  df1_vendor_id_dummy = pd.get_dummies(df1['vendor_id'], prefix='vendor_id_dummy')
  ```

* Créer la trame de données pour la modélisation

  ```python
  cols_to_keep = ['tipped', 'trip_distance', 'passenger_count']
  data = df1[cols_to_keep].join([df1_payment_type_dummy,df1_vendor_id_dummy])

  X = data.iloc[:,1:]
  Y = data.tipped
    ```

* Apprentissage et test de fractionnement de 60 à 40

  ```python
  X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.4, random_state=0)
  ```

* Régression logistique dans le jeu d’apprentissage

  ```python
  model = LogisticRegression()
  logit_fit = model.fit(X_train, Y_train)
  print ('Coefficients: \n', logit_fit.coef_)
  Y_train_pred = logit_fit.predict(X_train)
  ```

    ![c1](./media/data-lake-walkthrough/c1-py-logit-coefficient.PNG)

* Noter le jeu de données de test

  ```python
  Y_test_pred = logit_fit.predict(X_test)
  ```

* Calculer les mesures d’évaluation

  ```python
  fpr_train, tpr_train, thresholds_train = metrics.roc_curve(Y_train, Y_train_pred)
  print fpr_train, tpr_train, thresholds_train

  fpr_test, tpr_test, thresholds_test = metrics.roc_curve(Y_test, Y_test_pred)
  print fpr_test, tpr_test, thresholds_test

  #AUC
  print metrics.auc(fpr_train,tpr_train)
  print metrics.auc(fpr_test,tpr_test)

  #Confusion Matrix
  print metrics.confusion_matrix(Y_train,Y_train_pred)
  print metrics.confusion_matrix(Y_test,Y_test_pred)
  ```

    ![c2](./media/data-lake-walkthrough/c2-py-logit-evaluation.PNG)

### <a name="build-web-service-api-and-consume-it-in-python"></a>Créer une API de service web et l’exploiter dans Python
Vous voulez rendre le modèle Machine Learning opérationnel une fois créé. Ici, un modèle logistique binaire est utilisé comme exemple. Assurez-vous que la version de Scikit-learn disponible sur votre ordinateur local est bien la version 0.15.1 (Azure Machine Learning Studio utilise déjà cette version ou une version ultérieure).

* Recherchez vos informations d’identification de l’espace de travail dans les paramètres Azure Machine Learning Studio (classique). Dans Azure Machine Learning Studio, cliquez sur **Paramètres** --> **Nom** --> **Jetons d'autorisation**.

    ![c3](./media/data-lake-walkthrough/c3-workspace-id.PNG)

  ```output
  workspaceid = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'
  auth_token = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'
  ```

* Créer un service web

  ```python
  @services.publish(workspaceid, auth_token)
  @services.types(trip_distance = float, passenger_count = float, payment_type_dummy_CRD = float, payment_type_dummy_CSH=float, payment_type_dummy_DIS = float, payment_type_dummy_NOC = float, payment_type_dummy_UNK = float, vendor_id_dummy_CMT = float, vendor_id_dummy_VTS = float)
  @services.returns(int) #0, or 1
  def predictNYCTAXI(trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH,payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS ):
      inputArray = [trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH, payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS]
      return logit_fit.predict(inputArray)
  ```

* Obtenir des informations d’identification du service web

  ```python
  url = predictNYCTAXI.service.url
  api_key =  predictNYCTAXI.service.api_key

  print url
  print api_key

  @services.service(url, api_key)
  @services.types(trip_distance = float, passenger_count = float, payment_type_dummy_CRD = float, payment_type_dummy_CSH=float,payment_type_dummy_DIS = float, payment_type_dummy_NOC = float, payment_type_dummy_UNK = float, vendor_id_dummy_CMT = float, vendor_id_dummy_VTS = float)
  @services.returns(float)
  def NYCTAXIPredictor(trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH,payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS ):
      pass
  ```

* Appeler une API de service web En règle générale, patientez 5 à 10 secondes après l'étape précédente.

  ```python
  NYCTAXIPredictor(1,2,1,0,0,0,0,0,1)
  ```

    ![c4](./media/data-lake-walkthrough/c4-call-API.PNG)

## <a name="option-2-create-and-deploy-models-directly-in-azure-machine-learning"></a>Option n°2 : Créer et déployer des modèles directement dans Azure Machine Learning
Azure Machine Learning Studio (classique) peut lire des données directement à partir d'Azure Data Lake Storage, puis être utilisé pour créer et déployer des modèles. Cette approche utilise une table Hive qui pointe vers Azure Data Lake Storage. Un cluster Azure HDInsight distinct doit être approvisionné pour la table Hive. 

### <a name="create-an-hdinsight-linux-cluster"></a>Créer un cluster HDInsight Linux
Créez un cluster HDInsight (Linux) à partir du [portail Azure](https://portal.azure.com). Pour plus d'informations, consultez la section **Créer un cluster HDInsight avec accès à Azure Data Lake Storage** de l'article [Créer un cluster HDInsight avec Data Lake Store à l'aide du portail Azure](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md).

 ![18](./media/data-lake-walkthrough/18-create_HDI_cluster.PNG)

### <a name="create-hive-table-in-hdinsight"></a>Créer une table Hive dans HDInsight
Vous allez maintenant créer des tables Hive pour les utiliser dans Azure Machine Learning Studio (classique), au sein du cluster HDInsight, à l'aide des données stockées dans Azure Data Lake Storage lors de l'étape précédente. Accédez au cluster HDInsight créé. Cliquez sur **Paramètres** --> **Propriétés** --> **Identité AAD de cluster** --> **Accès ADLS**. Assurez-vous que votre compte Azure Data Lake Storage a été ajouté à la liste, avec des droits de lecture, d'écriture et d'exécution.

 ![19](./media/data-lake-walkthrough/19-HDI-cluster-add-ADLS.PNG)

Puis cliquez sur **Tableau de bord** en regard du bouton **Paramètres** et une fenêtre contextuelle s’affiche. Cliquez sur **Affichage Hive** dans l’angle supérieur droit de la page pour afficher **Éditeur de requête**.

 ![20](./media/data-lake-walkthrough/20-HDI-dashboard.PNG)

 ![21](./media/data-lake-walkthrough/21-Hive-Query-Editor-v2.PNG)

Collez les scripts Hive suivants pour créer une table. Vous trouverez l'emplacement de la source de données dans la référence Azure Data Lake Storage sous la forme suivante : **adl://data_lake_store_name.azuredatalakestore.net:443/nom_dossier nom_fichier**.

```hiveql
CREATE EXTERNAL TABLE nyc_stratified_sample
(
    medallion string,
    hack_license string,
    vendor_id string,
    rate_code string,
    store_and_fwd_flag string,
    pickup_datetime string,
    dropoff_datetime string,
    passenger_count string,
    trip_time_in_secs string,
    trip_distance string,
    pickup_longitude string,
    pickup_latitude string,
    dropoff_longitude string,
    dropoff_latitude string,
  payment_type string,
  fare_amount string,
  surcharge string,
  mta_tax string,
  tolls_amount string,
  total_amount string,
  tip_amount string,
  tipped string,
  tip_class string,
  rownum string
  )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
LOCATION 'adl://data_lake_storage_name.azuredatalakestore.net:443/nyctaxi_folder/demo_ex_9_stratified_1_1000_copy.csv';
```

Au terme de l'exécution de la requête, les résultats apparaissent comme suit :

 ![22](./media/data-lake-walkthrough/22-Hive-Query-results.PNG)

### <a name="build-and-deploy-models-in-azure-machine-learning-studio"></a>Créer et déployer des modèles dans Azure Machine Studio
Vous êtes maintenant prêts à créer et déployer un modèle qui prédit si un pourboire est payé avec Azure Machine Learning. L’échantillon stratifié de données est prêt à être utilisé dans ce problème de classification binaire (pourboire ou non). Les modèles prévisionnels utilisant la classification multiclasse (tip_class) et la régression (tip_amount) peuvent également être créés et déployés avec Azure Machine Learning Studio, mais ici nous expliquons seulement comment gérer le cas avec le modèle de classification binaire.

1. Importez les données dans Azure Machine Learning Studio (classique) avec le module **Importer les données**, disponible dans la section **Entrée et sortie des données**. Pour plus d’informations, consultez la page de référence du module [Importer les données](/azure/machine-learning/studio-module-reference/import-data) .
2. Sélectionnez **Requête Hive** comme **Source de données** dans le panneau **Propriétés**.
3. Collez le script Hive suivant dans l’éditeur de **requête de base de données Hive**

    ```hiveql
    select * from nyc_stratified_sample;
    ```

4. Entrez l'URI du cluster HDInsight (disponible sur le portail Azure), les informations d'identification Hadoop, l'emplacement des données de sortie et le nom du compte/la clé/le nom du conteneur Azure Storage.

   ![23](./media/data-lake-walkthrough/23-reader-module-v3.PNG)

La figure suivante montre un exemple d’expérience de classification binaire qui lit les données à partir de la table Hive :

 ![24](./media/data-lake-walkthrough/24-AML-exp.PNG)

Une fois l’expérience créée, cliquez sur **Configurer le service web** --> **Service web prédictif**

 ![25](./media/data-lake-walkthrough/25-AML-exp-deploy.PNG)

Exécutez l’expérience de notation créée automatiquement. Une fois terminée, cliquez sur **Déployer le service web**

 ![26](./media/data-lake-walkthrough/26-AML-exp-deploy-web.PNG)

Le tableau de bord du service web s’affiche brièvement :

 ![27](./media/data-lake-walkthrough/27-AML-web-api.PNG)

## <a name="summary"></a>Résumé
En suivant cette procédure pas à pas, vous avez créé un environnement de science des données pour développer des solutions de bout en bout évolutives dans Azure Data Lake. Cet environnement a été utilisé pour analyser un jeu de données public volumineux à travers les étapes du processus de science des données, de l’acquisition des données au déploiement du modèle en tant que service web, en passant par l’apprentissage du modèle. U-SQL a été utilisé pour traiter, explorer et échantillonner les données. Python et Hive ont été utilisés avec Azure Machine Learning Studio (classique) pour créer et déployer des modèles prédictifs.

## <a name="whats-next"></a>Quelle est l’étape suivante ?
Le parcours d’apprentissage du [processus TDSP (Team Data Science Process)](./index.yml) fournit des liens vers des rubriques qui décrivent chaque étape du processus d’analyse avancée. Il existe une série de procédures pas à pas détaillées à la page [Procédures pas à pas du processus TDSP (Team Data Science Process)](walkthroughs.md) qui mettent en évidence l’utilisation des ressources et des services dans les différents scénarios d’analyse prédictive :

* [Processus TDSP (Team Data Science Process) en action : utilisation d'Azure Synapse Analytics](sqldw-walkthrough.md)
* [Processus TDSP (Team Data Science Process) en action : utilisation de clusters Hadoop HDInsight](hive-walkthrough.md)
* [Processus TDSP (Team Data Science Process) : utilisation de SQL Server](sql-walkthrough.md)
* [Vue d’ensemble du processus de science des données à l’aide de Spark sur Azure HDInsight](spark-overview.md)