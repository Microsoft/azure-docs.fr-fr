---
title: Créer et déployer un modèle à l’aide d'Azure Synapse Analytics - Team Data Science Process
description: Créez et déployez un modèle Machine Learning à l’aide d'Azure Synapse Analytics avec un jeu de données disponible publiquement.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, devx-track-python, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: b638cb2b33f24220e7ceb852402862c707cc7bc6
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93315996"
---
# <a name="the-team-data-science-process-in-action-using-azure-synapse-analytics"></a>Processus TDSP (Team Data Science Process) en action : utilisation d'Azure Synapse Analytics
Dans ce tutoriel, nous vous guidons dans la création et le déploiement d’un modèle d’apprentissage automatique utilisant Azure Synapse Analytics pour un jeu de données disponible publiquement, le jeu de données [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/). Le modèle de classification binaire créé consiste à prédire si un trajet va faire l’objet d’un pourboire.  Les modèles incluent une classification multiclasse (qu'il y ait pourboire ou non) et une régression (distribution des montants des pourboires versés).

La procédure suit le flux de travail [processus TDSP (Team Data Science Process)](./index.yml) . Nous montrons comment configurer un environnement de science des données, comment charger les données dans Azure Synapse Analytics et comment utiliser Azure Synapse Analytics ou un IPython Notebook pour explorer les données et les caractéristiques d’ingénierie à modéliser. Nous expliquons ensuite comment générer et déployer un modèle avec Azure Machine Learning.

## <a name="the-nyc-taxi-trips-dataset"></a><a name="dataset"></a>Jeu de données NYC Taxi Trips
Les données NYC Taxi Trip sont constituées de fichiers CSV compressés d’une taille totale approximative de 20 Go (soit environ 48 Go après la décompression des fichiers), correspondant à plus de 173 millions de courses et au prix de chacune. Chaque enregistrement de course inclut le lieu et l’heure de prise en charge et de dépose, le numéro de licence (du chauffeur) rendu anonyme et le numéro de médaillon (numéro d’identification unique) du taxi. Les données portent sur toutes les courses effectuées en 2013 et sont fournies dans les deux jeux de données ci-après pour chaque mois :

1. Le fichier **trip_data.csv** contient les détails de chaque course, comme le nombre de passagers, les points d’embarquement et de débarquement, la durée du trajet et la distance parcourue. Voici quelques exemples d’enregistrements :

`medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

2. Le fichier **trip_fare.csv** contient les détails du prix de chaque course, comme le type de paiement, le prix de la course, les suppléments et les taxes, les pourboires et les péages, ainsi que le montant total réglé. Voici quelques exemples d’enregistrements :

`medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

La **clé unique** utilisée pour joindre trip\_data et trip\_fare se compose des trois champs suivants :

* medallion (médaillon),
* hack\_license (licence de taxi) et
* pickup\_datetime (date et heure d’embarquement).

## <a name="address-three-types-of-prediction-tasks"></a><a name="mltasks"></a>Traiter trois types de tâches de prédiction
Nous formulons trois problèmes de prédiction reposant sur la valeur *tip\_amount* pour illustrer trois genres de tâches de modélisation :

1. **Classification binaire** : Prédire si un pourboire a ou non été versé pour une course ; autrement dit, une valeur *tip\_amount* supérieure à 0 $ constitue un exemple positif, alors qu’une valeur *tip\_amount* de 0 $ est un exemple négatif.
2. **Classification multiclasse** : Prédire la fourchette des pourboires versés pour une course. Nous divisons la valeur *tip\_amount* en cinq compartiments ou classes :

`Class 0 : tip_amount = $0`

`Class 1 : tip_amount > $0 and tip_amount <= $5`

`Class 2 : tip_amount > $5 and tip_amount <= $10`

`Class 3 : tip_amount > $10 and tip_amount <= $20`

`Class 4 : tip_amount > $20`

3. **Tâche de régression** : Prédire le montant du pourboire versé pour une course.

## <a name="set-up-the-azure-data-science-environment-for-advanced-analytics"></a><a name="setup"></a>Configurer l’environnement de science des données Azure pour l’analyse avancée
Pour configurer votre environnement de science des données Azure, procédez comme suit :

**Créez votre propre compte de stockage d’objets blob Azure**

* Quand vous approvisionnez votre propre espace de stockage d’objets blob Azure, choisissez un emplacement géographique pour celui-ci dans le **USA Centre Sud** , ou aussi près que possible de cette région, où sont stockées les données NYC Taxi. Les données sont copiées à l’aide d’AzCopy du conteneur de stockage d’objets blob publics vers un conteneur de votre propre compte de stockage. La rapidité d’exécution de cette tâche (étape 4) est proportionnelle à la proximité de votre espace de stockage d’objets blob Azure avec la région USA Centre Sud.
* Pour créer votre propre compte Stockage Azure, suivez la procédure décrite dans [À propos des comptes Stockage Azure](../../storage/common/storage-account-create.md). Notez les informations d’identification suivantes du compte de stockage, car vous en aurez besoin ultérieurement dans cette procédure.

  * **Nom du compte de stockage**
  * **Clé du compte de stockage**
  * **Nom du conteneur** (dans lequel vous souhaitez stocker les données dans l’espace de stockage d’objets blob Azure)

**Approvisionnez votre instance Azure Synapse Analytics.**
Pour provisionner une instance Azure Synapse Analytics, suivez la documentation [Créer et interroger une instance Azure Synapse Analytics avec le portail Azure](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md). Assurez-vous de prendre note des informations d’identification suivantes d'Azure Synapse Analytics dont vous aurez besoin dans les étapes ultérieures.

* **Nom du serveur** : \<server Name>.database.windows.net
* **Nom SQL DW (base de données)**
* **Nom d’utilisateur**
* **Mot de passe**

**Installez Visual Studio et SQL Server Data Tools.** Pour obtenir des instructions, consultez [Bien démarrer avec Visual Studio 2019 pour Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-install-visual-studio.md).

**Connectez-vous à Azure Synapse Analytics avec Visual Studio.** Pour obtenir des instructions, consultez les étapes 1 et 2 décrites dans [Se connecter à SQL Analytics dans Azure Synapse Analytics](../../synapse-analytics/sql/connect-overview.md).

> [!NOTE]
> Exécutez la requête SQL suivante sur la base de données que vous avez créée dans Azure Synapse Analytics (au lieu de la requête fournie à l’étape 3 de la rubrique connexion) pour **créer une clé principale**.
>
>

```sql
BEGIN TRY
       --Try to create the master key
    CREATE MASTER KEY
END TRY
BEGIN CATCH
       --If the master key exists, do nothing
END CATCH;
```

**Créez un espace de travail Azure Machine Learning dans votre abonnement Azure.** Pour connaître les instructions à suivre, consultez l’article [Création d’un espace de travail Azure Machine Learning](../classic/create-workspace.md).

## <a name="load-the-data-into-azure-synapse-analytics"></a><a name="getdata"></a>Charger les données dans Azure Synapse Analytics
Ouvrez une console de commandes Windows PowerShell. Exécutez les commandes PowerShell suivantes pour télécharger les fichiers d’exemple de script SQL que nous partageons avec vous sur GitHub dans un répertoire local que vous spécifiez avec le paramètre *-DestDir*. Vous pouvez remplacer la valeur du paramètre *-DestDir* par un répertoire local. Si *-DestDir* n’existe pas, il est créé par le script PowerShell.

> [!NOTE]
> Vous devrez peut-être sélectionner **Exécuter en tant qu’administrateur** au moment de l’exécution du script PowerShell suivant si le privilège Administrateur est nécessaire pour créer le répertoire *DestDir* ou pour y écrire.
>
>

```azurepowershell
$source = "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/Download_Scripts_SQLDW_Walkthrough.ps1"
$ps1_dest = "$pwd\Download_Scripts_SQLDW_Walkthrough.ps1"
$wc = New-Object System.Net.WebClient
$wc.DownloadFile($source, $ps1_dest)
.\Download_Scripts_SQLDW_Walkthrough.ps1 –DestDir 'C:\tempSQLDW'
```

Une fois le script exécuté, *-DestDir* devient votre répertoire de travail. Vous devez voir un écran semblable à ce qui suit :

![Modification du répertoire de travail actuel][19]

Dans *-DestDir* , exécutez le script PowerShell suivant en mode administrateur :

```azurepowershell
./SQLDW_Data_Import.ps1
```

Lorsque le script PowerShell s’exécute pour la première fois, vous devez entrer les informations d'Azure Synapse Analytics et de votre compte de stockage d’objets blob Azure. À l’issue de la première exécution de ce script PowerShell, les informations d’identification que vous avez entrées sont écrites dans le fichier de configuration SQLDW.conf dans le répertoire de travail actuel. L’exécution suivante de ce fichier de script PowerShell permet de lire tous les paramètres nécessaires de ce fichier de configuration. Si vous devez en modifier certains, vous pouvez choisir d’entrer les paramètres dans l’écran dès l’invite en supprimant ce fichier de configuration et en entrant les valeurs des paramètres comme demandé ou vous pouvez décider de changer ces valeurs en modifiant le fichier SQLDW.conf dans votre répertoire *-DestDir* .

> [!NOTE]
> Pour éviter des conflits de noms de schéma avec ceux qui existent déjà dans Azure Synapse Analytics, au moment de la lecture directe des paramètres du fichier SQLDW.conf, un nombre aléatoire à 3 chiffres est ajouté au nom du schéma du fichier SQLDW.conf comme nom de schéma par défaut pour chaque exécution. Le script PowerShell peut vous demander un nom de schéma : le nom peut être spécifié à la discrétion de l’utilisateur.
>
>

Ce fichier de **script PowerShell** exécute les tâches suivantes :

* **Télécharge et installe AzCopy** , si AzCopy n’est pas déjà installé.

  ```azurepowershell
  $AzCopy_path = SearchAzCopy
  if ($AzCopy_path -eq $null){
         Write-Host "AzCopy.exe is not found in C:\Program Files*. Now, start installing AzCopy..." -ForegroundColor "Yellow"
      InstallAzCopy
      $AzCopy_path = SearchAzCopy
  }
      $env_path = $env:Path
      for ($i=0; $i -lt $AzCopy_path.count; $i++){
          if ($AzCopy_path.count -eq 1){
              $AzCopy_path_i = $AzCopy_path
          } else {
              $AzCopy_path_i = $AzCopy_path[$i]
          }
          if ($env_path -notlike '*' +$AzCopy_path_i+'*'){
              Write-Host $AzCopy_path_i 'not in system path, add it...'
              [Environment]::SetEnvironmentVariable("Path", "$AzCopy_path_i;$env_path", "Machine")
              $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
              $env_path = $env:Path
          }
  ```

* **Copie des données vers votre compte de stockage privé d’objets blob** depuis l’espace de stockage public d’objets blob avec AzCopy.

  ```azurepowershell
  Write-Host "AzCopy is copying data from public blob to yo storage account. It may take a while..." -ForegroundColor "Yellow"
  $start_time = Get-Date
  AzCopy.exe /Source:$Source /Dest:$DestURL /DestKey:$StorageAccountKey /S
  $end_time = Get-Date
  $time_span = $end_time - $start_time
  $total_seconds = [math]::Round($time_span.TotalSeconds,2)
  Write-Host "AzCopy finished copying data. Please check your storage account to verify." -ForegroundColor "Yellow"
  Write-Host "This step (copying data from public blob to your storage account) takes $total_seconds seconds." -ForegroundColor "Green"
  ```

* **Charge les données à l’aide de Polybase (en exécutant LoadDataToSQLDW.sql) dans Azure Synapse Analytics** à partir de votre compte de stockage privé d’objets blob avec les commandes suivantes.

  * Créer un schéma

    ```sql
    EXEC (''CREATE SCHEMA {schemaname};'');
    ```

  * Créer un fichier d’informations d’identification de niveau base de données

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL {KeyAlias}
    WITH IDENTITY = ''asbkey'' ,
    Secret = ''{StorageAccountKey}''
    ```

  * Créer une source de données externe pour un objet blob de stockage Azure

    ```sql
    CREATE EXTERNAL DATA SOURCE {nyctaxi_trip_storage}
    WITH
    (
        TYPE = HADOOP,
        LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
        CREDENTIAL = {KeyAlias}
    )
    ;

    CREATE EXTERNAL DATA SOURCE {nyctaxi_fare_storage}
    WITH
    (
        TYPE = HADOOP,
        LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
        CREDENTIAL = {KeyAlias}
    )
    ;
    ```

  * Créer un format de fichier externe pour un fichier csv. Les données ne sont pas compressées et les champs sont séparés par le caractère barre verticale.

    ```sql
    CREATE EXTERNAL FILE FORMAT {csv_file_format}
    WITH
    (
        FORMAT_TYPE = DELIMITEDTEXT,
        FORMAT_OPTIONS
        (
            FIELD_TERMINATOR ='','',
            USE_TYPE_DEFAULT = TRUE
        )
    )
    ;
    ```

  * Créer des tables externes pour les prix et les courses correspondant au jeu de données NYC Taxi dans le stockage d’objets blob Azure.

    ```sql
    CREATE EXTERNAL TABLE {external_nyctaxi_fare}
    (
        medallion varchar(50) not null,
        hack_license varchar(50) not null,
        vendor_id char(3),
        pickup_datetime datetime not null,
        payment_type char(3),
        fare_amount float,
        surcharge float,
        mta_tax float,
        tip_amount float,
        tolls_amount float,
        total_amount float
    )
    with (
        LOCATION    = ''/nyctaxifare/'',
        DATA_SOURCE = {nyctaxi_fare_storage},
        FILE_FORMAT = {csv_file_format},
        REJECT_TYPE = VALUE,
        REJECT_VALUE = 12
    )

      CREATE EXTERNAL TABLE {external_nyctaxi_trip}
      (
             medallion varchar(50) not null,
             hack_license varchar(50)  not null,
             vendor_id char(3),
             rate_code char(3),
             store_and_fwd_flag char(3),
             pickup_datetime datetime  not null,
             dropoff_datetime datetime,
             passenger_count int,
             trip_time_in_secs bigint,
             trip_distance float,
             pickup_longitude varchar(30),
             pickup_latitude varchar(30),
             dropoff_longitude varchar(30),
             dropoff_latitude varchar(30)
      )
      with (
          LOCATION    = ''/nyctaxitrip/'',
          DATA_SOURCE = {nyctaxi_trip_storage},
          FILE_FORMAT = {csv_file_format},
          REJECT_TYPE = VALUE,
          REJECT_VALUE = 12
      )
    ```

    - Charger les données à partir des tables externes du stockage d’objets blob Azure dans Azure Synapse Analytics

      ```sql
      CREATE TABLE {schemaname}.{nyctaxi_fare}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      SELECT *
      FROM   {external_nyctaxi_fare}
      ;

      CREATE TABLE {schemaname}.{nyctaxi_trip}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      SELECT *
      FROM   {external_nyctaxi_trip}
      ;
      ```

    - Crée un exemple de table de données (NYCTaxi_Sample) et y insère des données résultant de la sélection de requêtes SQL dans les tables des courses et des prix. (Certaines étapes de cette procédure nécessitent l’utilisation de cette table d’exemple.)

      ```sql
      CREATE TABLE {schemaname}.{nyctaxi_sample}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      (
          SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount,
          tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
          tip_class = CASE
                  WHEN (tip_amount = 0) THEN 0
                  WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                  WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                  WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                  ELSE 4
              END
          FROM {schemaname}.{nyctaxi_trip} t, {schemaname}.{nyctaxi_fare} f
          WHERE datepart("mi",t.pickup_datetime) = 1
          AND t.medallion = f.medallion
          AND   t.hack_license = f.hack_license
          AND   t.pickup_datetime = f.pickup_datetime
          AND   pickup_longitude <> ''0''
          AND   dropoff_longitude <> ''0''
      )
      ;
      ```

L’emplacement géographique de vos comptes de stockage a une incidence sur les délais de chargement.

> [!NOTE]
> Selon l’emplacement géographique de votre compte de stockage privé d’objets blob, le processus de copie des données d’un objet blob public vers votre compte de stockage privé peut prendre environ 15 minutes, voire plus, tandis que le processus de chargement des données de votre compte de stockage vers Azure Synapse Analytics peut prendre 20 minutes ou plus.
>
>

Vous devez décider ce que vous souhaitez faire si vous avez des fichiers source et de destination en double.

> [!NOTE]
> Si les fichiers .csv à copier de l’espace de stockage public d’objets blob vers votre compte de stockage privé d’objets blob existent déjà dans ce dernier, AzCopy vous demande si vous souhaitez les remplacer. Si vous ne le souhaitez pas, entrez **n** à l’invite. Si vous souhaitez les remplacer **tous** , entrez **a** à l’invite. Vous pouvez également entrer **y** pour remplacer les fichiers .csv un par un.
>
>

![Sortie AzCopy][21]

Vous pouvez utiliser vos propres données. Si vos données sont stockées sur votre ordinateur sur site dans votre application réelle, vous pouvez toujours utiliser AzCopy pour charger les données locales vers l’espace de stockage privé d’objets blob Azure. Vous devez uniquement modifier l’emplacement **Source** , `$Source = "http://getgoing.blob.core.windows.net/public/nyctaxidataset"`, dans la commande AzCopy du fichier de script PowerShell et le remplacer par le répertoire local qui contient vos données.

> [!TIP]
> Si vos données figurent déjà dans votre espace de stockage privé d’objets blob Azure de votre application réelle, vous pouvez ignorer l’étape AzCopy dans le script PowerShell et charger directement les données vers Azure Synapse Analytics. Pour effectuer cette opération, vous devez modifier le script afin de l’adapter au format de vos données.
>
>

Ce script Powershell relie également les informations d’Azure Synapse Analytics aux fichiers d’exemple d’exploration de données SQLDW_Explorations.sql, SQLDW_Explorations.ipynb et SQLDW_Explorations_Scripts.py afin que ces trois fichiers soient prêts à être essayés dès la fin de l’exécution du script PowerShell.

À l’issue d’une exécution réussie, un écran semblable à ce qui suit s’affiche :

![Sortie d’une exécution de script réussie][20]

## <a name="data-exploration-and-feature-engineering-in-azure-synapse-analytics"></a><a name="dbexplore"></a>Exploration des données et conception de fonctionnalités dans Azure Synapse Analytics
Dans cette section, nous effectuons une exploration des données et une génération de caractéristiques en exécutant des requêtes SQL directement dans Azure Synapse Analytics à l’aide de **Visual Studio Data Tools**. Toutes les requêtes SQL utilisées dans cette section se trouvent dans l’exemple de script nommé *SQLDW_Explorations.sql*. Ce fichier a déjà été téléchargé dans votre répertoire local par le script PowerShell. Vous pouvez également le récupérer à partir de [GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/SQLDW_Explorations.sql), mais les informations d’Azure Synapse Analytics ne sont pas reliées à ce fichier situé dans GitHub.

Connectez-vous à Azure Synapse Analytics en utilisant Visual Studio avec le nom et le mot de passe de connexion d'Azure Synapse Analytics et ouvrez l’ **Explorateur d’objets SQL Server** pour vérifier que la base de données et les tables ont été importées. Récupérez le fichier *SQLDW_Explorations.sql*.

> [!NOTE]
> Pour ouvrir un éditeur de requête Parallel Data Warehouse (PDW), utilisez la commande **Nouvelle requête** pendant que votre PDW est sélectionné dans **l’Explorateur d’objets SQL**. L’éditeur de requête SQL standard n’est pas pris en charge par PDW.
>
>

Voici les types de tâche d’exploration des données et de génération de fonctionnalités effectués dans cette section :

* explorer les distributions de données de quelques champs portant sur différentes périodes ;
* examiner la qualité des données des champs de longitude et de latitude ;
* Générer des étiquettes de classification binaire et multiclasse reposant sur la valeur **tip\_amount**.
* Générer des fonctionnalités et calculer/comparer les distances des trajets.
* Joindre les deux tables et extraire un échantillon aléatoire que nous utiliserons pour créer les modèles.

### <a name="data-import-verification"></a>Vérification de l’importation des données
Les requêtes ci-après procèdent à une vérification rapide du nombre de lignes et de colonnes des tables précédemment remplies à l’aide des importations en bloc en parallèle de Polybase.

-- Indiquer le nombre de lignes dans la table <nyctaxi_trip> sans analyse de la table

   ```sql
   SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
   ```

-- Indiquer le nombre de colonnes dans la table <nyctaxi_trip>

   ```sql
   SELECT COUNT(*) FROM information_schema.columns WHERE table_name = '<nyctaxi_trip>' AND table_schema = '<schemaname>'
   ```

**Output:** vous devez obtenir 173 179 759 lignes et 14 colonnes.

### <a name="exploration-trip-distribution-by-medallion"></a>Exploration : Distribution des courses par médaillon
Cet exemple de requête identifie les médaillons (numéros de taxi) qui ont effectué plus de 100 courses au cours d’une période spécifiée. Cette requête tire avantage de l’accès aux tables partitionnées, car elle est conditionnée par le schéma de partition de **pickup\_datetime**. L’exécution d’une requête portant sur le jeu de données complet tire également profit de l’analyse d’index et/ou de table partitionnée.

```sql
SELECT medallion, COUNT(*)
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
GROUP BY medallion
HAVING COUNT(*) > 100
```

**Output:** la requête doit retourner une table dont les lignes recensent les 13 369 médaillons (taxis) et le nombre correspondant de courses effectuées en 2013. La dernière colonne contient le nombre de courses effectuées.

### <a name="exploration-trip-distribution-by-medallion-and-hack_license"></a>Exploration : Distribution des courses par médaillon et par licence de taxi
Cet exemple identifie les médaillons (numéros de taxi) et les numéros hack_license (chauffeurs) qui ont effectué plus de 100 courses au cours d’une période spécifiée.

```sql
SELECT medallion, hack_license, COUNT(*)
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
GROUP BY medallion, hack_license
HAVING COUNT(*) > 100
```

**Output:** la requête doit retourner une table de 13 369 lignes recensant les 13 369 ID de voiture/chauffeur qui ont effectué plus de 100 courses en 2013. La dernière colonne contient le nombre de courses effectuées.

### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>Évaluation de la qualité des données : Vérifier les enregistrements indiquant une longitude et/ou une latitude incorrectes
Cet exemple vérifie si l’un des champs de longitude et/ou de latitude contient une valeur incorrecte (le nombre de degrés doit être compris entre -90 et 90) ou présente des coordonnées (0, 0).

```sql
SELECT COUNT(*) FROM <schemaname>.<nyctaxi_trip>
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
OR    (pickup_longitude = '0' AND pickup_latitude = '0')
OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))
```

**Output:** la requête retourne 837 467 courses dont les champs de longitude et/ou de latitude ne sont pas valides.

### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>Exploration : Distribution des courses avec et sans pourboire
Cet exemple détermine le nombre de courses qui ont fait l’objet d’un pourboire et celles qui n’ont pas donné lieu à un pourboire pendant une période spécifiée (ou dans le jeu de données complet si la période couvre l’année complète comme dans le cas présent). Cette distribution reflète la distribution des étiquettes binaires à utiliser par la suite pour la modélisation de classification binaire.

```sql
SELECT tipped, COUNT(*) AS tip_freq FROM (
  SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
  FROM <schemaname>.<nyctaxi_fare>
  WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tipped
```

**Output:** la requête doit retourner la distribution des courses avec et sans pourboire pour l’année 2013 suivante : 90 447 622 avec pourboire et 82 264 709 sans pourboire.

### <a name="exploration-tip-classrange-distribution"></a>Exploration : Distribution des classes/fourchettes de pourboires
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
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tip_class
```

**Output:**

| tip_class | tip_freq |
| --- | --- |
| 1 |82230915 |
| 2 |6198803 |
| 3 |1932223 |
| 0 |82264625 |
| 4 |85765 |

### <a name="exploration-compute-and-compare-trip-distance"></a>Exploration : Calculer et comparer les distances des courses
Cet exemple convertit la longitude et la latitude des points d’embarquement et de débarquement en points géographiques SQL, calcule la distance des trajets en se basant sur la différence entre ces points géographiques et renvoie un échantillon aléatoire des résultats pour comparaison. Cet exemple limite les résultats aux coordonnées valides en utilisant la requête d’évaluation de la qualité des données précédemment décrite.

```sql
/****** Object:  UserDefinedFunction [dbo].[fnCalculateDistance] ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
  DROP FUNCTION fnCalculateDistance
GO

-- User-defined function to calculate the direct distance  in mile between two geographical coordinates.
CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

RETURNS float
AS
BEGIN
      DECLARE @distance decimal(28, 10)
      -- Convert to radians
      SET @Lat1 = @Lat1 / 57.2958
      SET @Long1 = @Long1 / 57.2958
      SET @Lat2 = @Lat2 / 57.2958
      SET @Long2 = @Long2 / 57.2958
      -- Calculate distance
      SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
      --Convert to miles
      IF @distance <> 0
      BEGIN
        SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
      END
      RETURN @distance
END
GO

SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
FROM <schemaname>.<nyctaxi_trip>
WHERE datepart("mi",pickup_datetime)=1
AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND pickup_longitude != '0' AND dropoff_longitude != '0'
```

### <a name="feature-engineering-using-sql-functions"></a>Conception de fonctionnalités à l’aide de fonctions SQL
Les fonctions SQL peuvent parfois s’avérer efficaces pour la conception des fonctionnalités. Dans cette procédure, nous avons défini une fonction SQL pour calculer la distance directe séparant les lieux d’embarquement et de débarquement. Vous pouvez exécuter les scripts SQL ci-dessous dans **Visual Studio Data Tools**.

Voici le script SQL qui définit la fonction de distance.

```sql
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
  DROP FUNCTION fnCalculateDistance
GO

-- User-defined function calculate the direct distance between two geographical coordinates.
CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

RETURNS float
AS
BEGIN
      DECLARE @distance decimal(28, 10)
      -- Convert to radians
      SET @Lat1 = @Lat1 / 57.2958
      SET @Long1 = @Long1 / 57.2958
      SET @Lat2 = @Lat2 / 57.2958
      SET @Long2 = @Long2 / 57.2958
      -- Calculate distance
      SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
      --Convert to miles
      IF @distance <> 0
      BEGIN
        SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
      END
      RETURN @distance
END
GO
```

Voici un exemple d’appel de cette fonction pour générer des fonctionnalités dans votre requête SQL :

-- Exemple de requête pour appeler la fonction afin de créer des caractéristiques

   ```sql
SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
FROM <schemaname>.<nyctaxi_trip>
WHERE datepart("mi",pickup_datetime)=1
AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND pickup_longitude != '0' AND dropoff_longitude != '0'
   ```

**Output:** cette requête génère une table (de 2 803 538 lignes) indiquant les latitudes et longitudes des points de prise en charge et de dépose, ainsi que les distances directes correspondantes en miles. Voici les résultats pour les trois premières lignes :

| (Numéro de ligne) | pickup_latitude | pickup_longitude | dropoff_latitude | dropoff_longitude | DirectDistance |
| --- | --- | --- | --- | --- | --- |
| 1 |40,731804 |-74,001083 |40,736622 |-73,988953 |0,7169601222 |
| 2 |40,715794 |-74,010635 |40,725338 |-74,00399 |0,7448343721 |
| 3 |40.761456 |-73.999886 |40.766544 |-73.988228 |0,7037227967 |

### <a name="prepare-data-for-model-building"></a>Préparer les données pour la création de modèles
La requête ci-après joint les tables **nyctaxi\_trip** et **nyctaxi\_fare** , génère une étiquette de classification binaire **tipped** et une étiquette de classification multiclasse **tip\_class** , puis extrait un échantillon des données de l’intégralité du jeu de données joint. L’échantillonnage est effectué en récupérant un sous-ensemble des courses basé sur l’heure d’embarquement.  Vous pouvez ensuite copier cette requête et la coller directement dans le module [Importer les données][import-data] d’[Azure Machine Learning Studio (classique)](https://studio.azureml.net) pour permettre l’ingestion directe des données de l’instance SQL Database dans Azure. La requête exclut les enregistrements qui présentent des coordonnées (0, 0) incorrectes.

```sql
SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
    CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
    CASE WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
WHERE datepart("mi",t.pickup_datetime) = 1
AND   t.medallion = f.medallion
AND   t.hack_license = f.hack_license
AND   t.pickup_datetime = f.pickup_datetime
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

Lorsque vous êtes prêt à utiliser Azure Machine Learning, vous pouvez au choix :

1. enregistrer la requête SQL finale d’extraction et d’échantillonnage des données et copier-coller cette requête directement dans un module Importer les données [import-data] d’Azure Machine Learning, ou
2. stocker les données échantillonnées et générées que vous envisagez d’utiliser pour la création de modèles dans une nouvelle table Azure Synapse Analytics et utiliser cette table dans le module [Importer les données][import-data] d’Azure Machine Learning. Le script PowerShell de l’étape précédente a effectué cette tâche pour vous. Vous pouvez lire directement cette table dans le module Importer les données.

## <a name="data-exploration-and-feature-engineering-in-ipython-notebook"></a><a name="ipnb"></a>Exploration des données et conception de fonctionnalités dans IPython Notebook
Dans cette section, nous allons effectuer des tâches d’exploration des données et de génération de fonctionnalités en exécutant des requêtes Python et SQL dans l'instance Azure Synapse Analytics créée précédemment. Un exemple d’IPython Notebook nommé **SQLDW_Explorations.ipynb** et le fichier de script Python **SQLDW_Explorations_Scripts.py** ont été téléchargés dans votre répertoire local. Ils sont également disponibles sur [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/SQLDW). Ces deux fichiers sont identiques dans les scripts Python. Le fichier de script Python vous est fourni dans le cas où vous ne disposeriez pas d’un serveur IPython Notebook. Ces deux exemples de fichier Python sont conçus sous **Python 2.7**.

Les informations d’Azure Synapse Analytics nécessaires dans l’exemple IPython Notebook et dans le fichier de script Python téléchargés sur votre ordinateur local ont été précédemment reliées par le script PowerShell. Elles peuvent être exécutées sans aucune modification.

Si vous avez déjà configuré un espace de travail Azure Machine Learning, vous pouvez directement charger l’exemple IPython Notebook dans le service AzureML IPython Notebook et lancer son exécution. Pour charger l’exemple dans le service AzureML IPython Notebook, effectuez les étapes suivantes :

1. Connectez-vous à votre espace de travail Azure Machine Learning, et cliquez successivement sur **Studio** en haut de l’écran, puis sur **NOTEBOOKS** sur la gauche de la page web.

    ![Cliquer sur Studio puis sur NOTEBOOKS][22]
2. Cliquez sur **NOUVEAU** dans le coin inférieur gauche de la page web, puis sélectionnez **Python 2**. Indiquez alors un nom pour le notebook et cochez la case pour créer le IPython Notebook vide.

    ![Cliquer sur Nouveau, puis sélectionner Python 2][23]
3. Cliquez sur le symbole **Jupyter** dans le coin supérieur gauche du nouvel IPython Notebook.

    ![Cliquer sur le symbole Jupyter][24]
4. Effectuez un glisser-déplacer de l’exemple d’IPython Notebook vers la page **d’arborescence** du service AzureML IPython Notebook, puis cliquez sur **Charger**. L’exemple de IPython Notebook est alors chargé vers le service AzureML IPython Notebook.

    ![Cliquez sur Charger][25]

Pour exécuter l’exemple de IPython Notebook ou le fichier de script Python, vous avez besoin des packages Python ci-dessous. Si vous utilisez le service AzureML IPython Notebook, ces packages ont été préinstallés.

- pandas
- numpy
- matplotlib
- pyodbc
- PyTables

Lorsque vous générez des solutions analytiques avancées comportant de nombreuses données dans Azure Machine Learning, voici la séquence recommandée :

* Lisez un petit échantillon des données dans une trame de données en mémoire.
* Procédez à certaines visualisations et explorations à l’aide des données échantillonnées.
* Expérimentez la conception de fonctionnalités en utilisant les données échantillonnées.
* Pour exécuter de plus vastes tâches d’exploration des données, de manipulation des données et de conception de fonctionnalités, utilisez Python pour émettre des requêtes SQL directement dans Azure Synapse Analytics.
* Déterminez la taille d’échantillon qui convient pour la création de modèles Azure Machine Learning.

Vous trouverez ci-dessous quelques exemples d’exploration des données, de visualisation des données et de conception de fonctionnalités. D’autres exemples d’exploration des données se trouvent dans l’exemple de IPython Notebook et le fichier d’exemple de script Python.

### <a name="initialize-database-credentials"></a>Initialiser les informations d’identification de la base de données
Initialisez vos paramètres de connexion à la base de données dans les variables suivantes :

```sql
SERVER_NAME=<server name>
DATABASE_NAME=<database name>
USERID=<user name>
PASSWORD=<password>
DB_DRIVER = <database driver>
```

### <a name="create-database-connection"></a>Créer une connexion à la base de données
Voici la chaîne de connexion qui crée la connexion à la base de données.

```sql
CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
conn = pyodbc.connect(CONNECTION_STRING)
```

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_trip"></a>Afficher le nombre de lignes et de colonnes de la table <nyctaxi_trip>

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('<nyctaxi_trip>') AND table_schema = ('<schemaname>')
''', conn)

print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Nombre total de lignes = 173 179 759
* Nombre total de colonnes = 14

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_fare"></a>Afficher le nombre de lignes et de colonnes de la table <nyctaxi_fare>

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_fare>')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('<nyctaxi_fare>') AND table_schema = ('<schemaname>')
''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Nombre total de lignes = 173 179 759
* Nombre total de colonnes = 11

### <a name="read-in-a-small-data-sample-from-the-azure-synapse-analytics-database"></a>Lire un petit échantillon de données depuis la base de données Azure Synapse Analytics

```sql
t0 = time.time()

query = '''
    SELECT TOP 10000 t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
        f.tolls_amount, f.total_amount, f.tip_amount
    FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
    WHERE datepart("mi",t.pickup_datetime) = 1
    AND   t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
'''

df1 = pd.read_sql(query, conn)

t1 = time.time()
print 'Time to read the sample table is %f seconds' % (t1-t0)

print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])
```

Temps de lecture de la table d’exemple = 14,096495 secondes.
Nombre de lignes et de colonnes récupérées = (1000, 21)

### <a name="descriptive-statistics"></a>Statistiques descriptives
Vous êtes désormais prêt à explorer les données échantillonnées. Nous allons commencer par examiner certaines statistiques descriptives relatives au champ **trip\_distance** (ou à tout autre champ que vous choisissez de spécifier).

```sql
df1['trip_distance'].describe()
```

### <a name="visualization-box-plot-example"></a>Visualisation : Exemple de diagramme à surfaces
Nous examinons ensuite le diagramme à surfaces concernant la distance des courses afin de visualiser les quantiles.

```sql
df1.boxplot(column='trip_distance',return_type='dict')
```

![Sortie de diagramme à surfaces][1]

### <a name="visualization-distribution-plot-example"></a>Visualisation : Exemple de diagramme de distribution
Graphiques qui visualisent la distribution et histogramme correspondant aux distances des courses échantillonnées.

```sql
fig = plt.figure()
ax1 = fig.add_subplot(1,2,1)
ax2 = fig.add_subplot(1,2,2)
df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
df1['trip_distance'].hist(ax=ax2, bins=100, color='k')
```

![Sortie de diagramme de distribution][2]

### <a name="visualization-bar-and-line-plots"></a>Visualisation : Diagrammes en bâtons et linéaires
Dans cet exemple, nous compartimentons la distance des trajets en cinq zones et nous visualisons les résultats de cette opération.

```sql
trip_dist_bins = [0, 1, 2, 4, 10, 1000]
df1['trip_distance']
trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
trip_dist_bin_id
```

Nous pouvons représenter la distribution des compartiments ci-dessus dans un diagramme en bâtons ou dans un diagramme linéaire comme suit :

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')
```

![Sortie de diagramme en bâtons][3]

and

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')
```

![Sortie de diagramme linéaire][4]

### <a name="visualization-scatterplot-examples"></a>Visualisation : Exemple de diagramme de dispersion
Nous représentons le diagramme de dispersion entre **trip\_time\_in\_secs** et **trip\_distance** pour déterminer s’il existe une corrélation.

```sql
plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])
```

![Sortie de diagramme de dispersion de la relation entre l’heure et la distance][6]

De la même façon, nous pouvons vérifier la relation entre **rate\_code** et **trip\_distance**.

```sql
plt.scatter(df1['passenger_count'], df1['trip_distance'])
```

![Sortie de diagramme de dispersion de la relation entre le code et la distance][8]

### <a name="data-exploration-on-sampled-data-using-sql-queries-in-ipython-notebook"></a>Exploration des données échantillonnées à l’aide de requêtes SQL dans IPython Notebook
Dans cette section, nous allons explorer les distributions de données à l’aide de l’échantillon de données que nous avons stocké dans la table créée ci-dessus. Des explorations similaires peuvent être effectuées à l’aide des tables d’origine.

#### <a name="exploration-report-number-of-rows-and-columns-in-the-sampled-table"></a>Exploration : Afficher le nombre de lignes et de colonnes dans la table échantillonnée

```sql
nrows = pd.read_sql('''SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_sample>')''', conn)
print 'Number of rows in sample = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''SELECT count(*) FROM information_schema.columns WHERE table_name = ('<nyctaxi_sample>') AND table_schema = '<schemaname>'''', conn)
print 'Number of columns in sample = %d' % ncols.iloc[0,0]
```

#### <a name="exploration-tippednot-tripped-distribution"></a>Exploration : Distribution avec et sans pourboire

```sql
query = '''
SELECT tipped, count(*) AS tip_freq
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY tipped
    '''

    pd.read_sql(query, conn)
```

#### <a name="exploration-tip-class-distribution"></a>Exploration : Distribution des classes de pourboires

```sql
query = '''
    SELECT tip_class, count(*) AS tip_freq
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY tip_class
'''

tip_class_dist = pd.read_sql(query, conn)
```

#### <a name="exploration-plot-the-tip-distribution-by-class"></a>Exploration : Tracer la distribution des pourboires par classe

```sql
tip_class_dist['tip_freq'].plot(kind='bar')
```

![Diagramme n°26][26]

#### <a name="exploration-daily-distribution-of-trips"></a>Exploration : Distribution quotidienne des courses

```sql
query = '''
    SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY CONVERT(date, dropoff_datetime)
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-per-medallion"></a>Exploration : Distribution des courses par médaillon

```sql
query = '''
    SELECT medallion,count(*) AS c
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY medallion
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>Exploration : Distribution des courses par médaillon et par licence de taxi

```sql
query = '''select medallion, hack_license,count(*) from <schemaname>.<nyctaxi_sample> group by medallion, hack_license'''
pd.read_sql(query,conn)
```

#### <a name="exploration-trip-time-distribution"></a>Exploration : Distribution des durées de course

```sql
query = '''select trip_time_in_secs, count(*) from <schemaname>.<nyctaxi_sample> group by trip_time_in_secs order by count(*) desc'''
pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distance-distribution"></a>Exploration : Distribution des distances de course

```sql
query = '''select floor(trip_distance/5)*5 as tripbin, count(*) from <schemaname>.<nyctaxi_sample> group by floor(trip_distance/5)*5 order by count(*) desc'''
pd.read_sql(query,conn)
```

#### <a name="exploration-payment-type-distribution"></a>Exploration : Distribution des types de paiement

```sql
query = '''select payment_type,count(*) from <schemaname>.<nyctaxi_sample> group by payment_type'''
pd.read_sql(query,conn)
```

#### <a name="verify-the-final-form-of-the-featurized-table"></a>Vérifiez le format final de la table affichée

```sql
query = '''SELECT TOP 100 * FROM <schemaname>.<nyctaxi_sample>'''
pd.read_sql(query,conn)
```

## <a name="build-models-in-azure-machine-learning"></a><a name="mlmodel"></a>Créer des modèles dans Azure Machine Learning
Nous pouvons à présent passer aux phases de création et de déploiement de modèles dans [Azure Machine Learning](https://studio.azureml.net). Les données sont prêtes à être utilisées dans tous les problèmes de prédiction identifiés précédemment, à savoir :

1. **Classification binaire** : Prédire si un pourboire a ou non été versé pour une course.
2. **Classification multiclasse** : Prédire la fourchette du pourboire versé en fonction des classes précédemment définies.
3. **Tâche de régression** : Prédire le montant du pourboire versé pour une course.

Pour démarrer l’exercice de modélisation, connectez-vous à votre espace de travail **Azure Machine Learning (classique)** . Si vous n’avez pas encore créé d’espace de travail Machine Learning, consultez l’article [Créer un espace de travail Azure Machine Learning Studio (classique)](../classic/create-workspace.md).

1. Pour bien démarrer avec Azure Machine Learning, consultez la page [Azure Machine Learning Studio (classique) - De quoi s’agit-il ?](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
2. Connectez-vous à [Azure Machine Learning Studio (classique)](https://studio.azureml.net).
3. La page d’accueil de Machine Learning Studio (classique) permet d’accéder à une multitude d’informations, notamment des vidéos, des tutoriels, des liens vers la documentation de référence des modules et d’autres ressources. Pour plus d’informations sur Azure Machine Learning, consultez le [Centre de documentation Azure Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/).

Une expérience de formation classique se déroule comme suit :

1. Création d’une expérience à l’aide du bouton **+NOUVEAU** .
2. Obtention des données dans Azure Machine Learning Studio (classique).
3. Prétraitement, transformation et manipulation des données en fonction des besoins.
4. Génération des fonctionnalités requises.
5. Fractionnement des données sous forme de jeux de données d’apprentissage/de validation/de test (ou utilisation de jeux de données distincts pour chacune de ces opérations).
6. Sélection d’un ou de plusieurs algorithmes d’apprentissage automatique en fonction du problème d’apprentissage à résoudre. Exemples : classification binaire, classification multiclasse, régression.
7. Formation d’un ou de plusieurs modèles à l’aide du jeu de données d’apprentissage.
8. Notation du jeu de données de validation à l’aide des modèles formés.
9. Évaluation des modèles afin de calculer les métriques pertinents pour le problème d’apprentissage.
10. Ajustement des modèles et sélection du meilleur modèle à déployer.

Dans cet exercice, nous avons déjà exploré et généré les données dans Azure Synapse Analytics, et déterminé la taille de l’échantillon à ingérer dans Azure Machine Learning Studio (classique). Pour créer un ou plusieurs des modèles de prédiction, procédez comme suit :

1. Importez les données dans Azure Machine Learning Studio (classique) avec le module [Importer les données][import-data], disponible dans la section **Entrée et sortie des données**. Pour plus d’informations, consultez la page de référence du module [Importer les données][import-data].

    ![Importer les données Azure ML][17]
2. Dans le panneau **Propriétés** , sélectionnez **Base de données Azure SQL** dans le champ **Source de données**.
3. Dans le champ **Nom du serveur de base de données** , entrez le nom DNS de la base de données. Format : `tcp:<your_virtual_machine_DNS_name>,1433`
4. Dans le champ **Nom de la base de données** , entrez le nom de la base de données.
5. Entrez le *nom d’utilisateur SQL* dans le champ **Nom de compte d’utilisateur du serveur** , et le *mot de passe* dans le champ **Mot de passe de compte d’utilisateur du serveur**.
7. Dans la zone de texte **Requête de base de données** , collez la requête qui extrait les champs de base de données nécessaires (y compris les champs calculés tels que les étiquettes) et qui sous-échantillonne les données pour obtenir la taille d’échantillon souhaitée.

Un exemple d’expérience de classification binaire lisant directement les données de la base de données Azure Synapse Analytics est illustré dans la figure ci-dessous (pensez à remplacer les noms des tables nyctaxi_trip et nyctaxi_fare par le nom du schéma et les noms des tables que vous avez utilisés dans votre procédure). Vous pouvez créer des expériences similaires pour les problèmes de classification multiclasse et de régression.

![Formation Azure Machine Learning][10]

> [!IMPORTANT]
> Dans les exemples de requêtes d’extraction et d’échantillonnage de données de modélisation qui sont fournis aux sections précédentes, **toutes les étiquettes des trois exercices de modélisation sont incluses dans la requête**. Dans chacun des exercices de modélisation, une étape (obligatoire) importante consiste à **exclure** les étiquettes superflues pour les deux autres problèmes, ainsi que toute autre **fuite cible**. Par exemple, si vous avez recours à la classification binaire, utilisez l’étiquette **tipped** et excluez les champs **tip\_class** , **tip\_amount** et **total\_amount**. Les derniers champs sont des fuites cibles, car ils impliquent le pourboire versé.
>
> Pour exclure les colonnes superflues ou les fuites cibles, vous pouvez utiliser le module [Sélectionner des colonnes dans le jeu de données][select-columns] ou [Modifier les métadonnées][edit-metadata]. Pour plus d’informations, consultez les pages de référence [Sélectionner des colonnes dans le jeu de données][select-columns] et [Modifier les métadonnées][edit-metadata].
>
>

## <a name="deploy-models-in-azure-machine-learning"></a><a name="mldeploy"></a>Déployer des modèles dans Azure Machine Learning
Lorsque votre modèle est prêt, vous pouvez facilement le déployer sous la forme d’un service web directement à partir de l’expérience. Pour plus d’informations sur le déploiement de services web Azure Machine Learning, consultez [Déployer un service web Azure Machine Learning](../classic/deploy-a-machine-learning-web-service.md).

Pour déployer un nouveau service web, vous devez :

1. créer une expérience de notation ;
2. déployer le service web.

Pour créer une expérience de notation à partir d’une expérience de formation **terminée** , cliquez sur **CRÉER UNE EXPÉRIENCE DE NOTATION** dans la barre d’action inférieure.

![Notation Azure][18]

Azure Machine Learning va essayer de créer une expérience de notation reposant sur les composants de l’expérience d’apprentissage. Il va notamment :

1. enregistrer le modèle formé et supprimer les modules de formation de modèle ;
2. identifier un **port d’entrée** logique pour représenter le schéma de données d’entrée attendu ;
3. identifier un **port de sortie** logique pour représenter le schéma de sortie de service web attendu.

Une fois l’expérience de notation créée, examinez les résultats et procédez aux ajustements nécessaires. Un ajustement classique consiste à remplacer le jeu de données d’entrée ou la requête par un autre jeu excluant les champs d’étiquette, car ces champs d'étiquette ne seront pas mappés vers le schéma lors de l’appel du service. Il est également conseillé de restreindre la taille du jeu de données d’entrée et/ou de la requête au nombre d’enregistrements suffisants pour indiquer le schéma d’entrée. Pour le port de sortie, il est courant d’exclure tous les champs d’entrée et de n’inclure que les valeurs **Étiquettes notées** et **Probabilités notées** dans la sortie à l’aide du module [Sélectionner des colonnes dans le jeu de données][select-columns].

La figure ci-après illustre un exemple d’expérience de notation. Quand vous êtes prêt à effectuer le déploiement, cliquez sur le bouton **PUBLIER LE SERVICE WEB** de la barre d’action inférieure.

![Publication Azure Machine Learning][11]

## <a name="summary"></a>Résumé
Pour résumer ce didacticiel pas à pas, vous avez créé un environnement de science des données Azure, manipulé un jeu de données public volumineux, depuis l’acquisition des données et la formation de modèles, via le processus TDSP (Team Data Science Process), jusqu’au déploiement d’un service web Azure Machine Learning.

### <a name="license-information"></a>Informations de licence
Cet exemple de procédure pas à pas et les scripts et notebooks IPython qui lui sont associés sont partagés par Microsoft sous licence MIT. Pour plus d’informations, consultez le fichier LICENSE.txt figurant dans le répertoire de l’exemple de code sur GitHub.

## <a name="references"></a>References
- [Page de téléchargement des jeux de données NYC Taxi Trips par Andrés Monroy](https://www.andresmh.com/nyctaxitrips/)
- [Page de partage des données relatives aux courses en taxi new-yorkais par Chris Whong](https://chriswhong.com/open-data/foil_nyc_taxi/)
- [Page de recherche et de statistiques de la Commission des services de taxis et de limousines de la ville de New York](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[1]: ./media/sqldw-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sqldw-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sqldw-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sqldw-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sqldw-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sqldw-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sqldw-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sqldw-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sqldw-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sqldw-walkthrough/azuremltrain.png
[11]: ./media/sqldw-walkthrough/azuremlpublish.png
[12]: ./media/sqldw-walkthrough/ssmsconnect.png
[13]: ./media/sqldw-walkthrough/executescript.png
[14]: ./media/sqldw-walkthrough/sqlserverproperties.png
[15]: ./media/sqldw-walkthrough/sqldefaultdirs.png
[16]: ./media/sqldw-walkthrough/bulkimport.png
[17]: ./media/sqldw-walkthrough/amlreader.png
[18]: ./media/sqldw-walkthrough/amlscoring.png
[19]: ./media/sqldw-walkthrough/ps_download_scripts.png
[20]: ./media/sqldw-walkthrough/ps_load_data.png
[21]: ./media/sqldw-walkthrough/azcopy-overwrite.png
[22]: ./media/sqldw-walkthrough/ipnb-service-aml-1.png
[23]: ./media/sqldw-walkthrough/ipnb-service-aml-2.png
[24]: ./media/sqldw-walkthrough/ipnb-service-aml-3.png
[25]: ./media/sqldw-walkthrough/ipnb-service-aml-4.png
[26]: ./media/sqldw-walkthrough/tip_class_hist_1.png


<!-- Module References -->
[edit-metadata]: /azure/machine-learning/studio-module-reference/edit-metadata
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[import-data]: /azure/machine-learning/studio-module-reference/import-data