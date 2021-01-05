---
title: Notebooks Azure Synapse Studio
description: Ce tutoriel fournit une vue d’ensemble de la façon de créer un tableau de bord Power BI à l’aide d’Apache Spark et d’un pool SQL serverless.
services: synapse-analytics
author: midesa
ms.author: midesa
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: spark
ms.topic: tutorial
ms.date: 11/16/2020
ms.openlocfilehash: 791cab369dcbf9cab8d1256377cfee4a433c21b9
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450904"
---
# <a name="tutorial-create-a-power-bi-report-using-apache-spark-and-azure-synapse-analytics"></a>Tutoriel : Créer un rapport Power BI à l’aide d’Apache Spark et d’Azure Synapse Analytics

Les organisations ont souvent besoin de traiter de grands volumes de données avant de servir les principales parties prenantes de l’entreprise. Dans ce tutoriel, vous allez apprendre à tirer parti des expériences intégrées dans Azure Synapse Analytics pour traiter des données à l’aide d’Apache Spark et les fournir ultérieurement aux utilisateurs finaux via Power BI et SQL serverless.

## <a name="before-you-begin"></a>Avant de commencer
- [Espace de travail Azure Synapse Analytics](../quickstart-create-workspace.md) avec un compte de stockage ADLS Gen2 configuré comme stockage par défaut. 
- Espace de travail Power BI et Power BI Desktop pour visualiser les données. Pour plus d’informations, consultez [Créer un espace de travail Power BI](https://docs.microsoft.com/power-bi/service-create-the-new-workspaces) et [Installer Power BI Desktop](https://powerbi.microsoft.com/downloads/).
- Service lié pour connecter vos espaces de travail Azure Synapse Analytics et Power BI. Pour plus d’informations, consultez [Lier à un espace de travail Power BI](../quickstart-power-bi.md).
- Pool Apache Spark serverless dans votre espace de travail Synapse Analytics. Pour plus d’informations, consultez [Créer un pool Apache Spark serverless](../quickstart-create-apache-spark-pool-studio.md).
  
## <a name="download-and-prepare-the-data"></a>Télécharger et préparer les données
Dans cet exemple, vous utiliserez Apache Spark pour effectuer une analyse sur les données de pourboires liés aux courses de taxi à New York. Les données sont disponibles dans [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Ce sous-ensemble du jeu de données contient des données sur les courses de taxis jaunes, qui incluent des informations sur chaque course, les heures et lieux de départ et d’arrivé, le prix et d’autres attributs intéressants.

1. Exécutez les lignes suivantes pour créer une tramedonnées Spark en collant le code dans une nouvelle cellule. Cela récupère les données via l’API Open Datasets. L’extraction de toutes ces données génère environ 1,5 milliard de lignes. L'exemple de code suivant utilise start_date et end_date pour appliquer un filtre qui renvoie un seul mois de données.
   
   ```python
    from azureml.opendatasets import NycTlcYellow
    from dateutil import parser
    from datetime import datetime

    end_date = parser.parse('2018-06-06')
    start_date = parser.parse('2018-05-01')
    nyc_tlc = NycTlcYellow(start_date=start_date, end_date=end_date)
    filtered_df = nyc_tlc.to_spark_dataframe()
   ```
2. À l’aide d’Apache Spark SQL, nous allons créer une base de données appelée NycTlcTutorial. Nous utiliserons cette base de données pour stocker les résultats de notre traitement de données.
   ```python
   %%pyspark
    spark.sql("CREATE DATABASE IF NOT EXISTS NycTlcTutorial")
   ```
3. Ensuite, nous utiliserons des opérations de dataframe Spark pour traiter les données. Dans le code suivant, nous effectuons les transformations suivantes :
   1. Suppression des colonnes superflues.
   2. suppression des valeurs hors normes/incorrectes par filtrage ;
   3. Création de nouvelles fonctionnalités comme ```tripTimeSecs``` et ```tipped``` pour une analyse supplémentaire.
    ```python
    from pyspark.sql.functions import unix_timestamp, date_format, col, when

    taxi_df = filtered_df.select('totalAmount', 'fareAmount', 'tipAmount', 'paymentType', 'rateCodeId', 'passengerCount'\
                                    , 'tripDistance', 'tpepPickupDateTime', 'tpepDropoffDateTime'\
                                    , date_format('tpepPickupDateTime', 'hh').alias('pickupHour')\
                                    , date_format('tpepPickupDateTime', 'EEEE').alias('weekdayString')\
                                    , (unix_timestamp(col('tpepDropoffDateTime')) - unix_timestamp(col('tpepPickupDateTime'))).alias('tripTimeSecs')\
                                    , (when(col('tipAmount') > 0, 1).otherwise(0)).alias('tipped')
                                    )\
                            .filter((filtered_df.passengerCount > 0) & (filtered_df.passengerCount < 8)\
                                    & (filtered_df.tipAmount >= 0) & (filtered_df.tipAmount <= 25)\
                                    & (filtered_df.fareAmount >= 1) & (filtered_df.fareAmount <= 250)\
                                    & (filtered_df.tipAmount < filtered_df.fareAmount)\
                                    & (filtered_df.tripDistance > 0) & (filtered_df.tripDistance <= 100)\
                                    & (filtered_df.rateCodeId <= 5)
                                    & (filtered_df.paymentType.isin({"1", "2"})))
    ```
4. Enfin, nous allons enregistrer notre dataframe à l’aide de la méthode Apache Spark ```saveAsTable```. Cela vous permettra ultérieurement d’exécuter une requête et de vous connecter à la même table à l’aide de pools SQL serverless.
  ```python
     taxi_df.write.mode("overwrite").saveAsTable("NycTlcTutorial.nyctaxi")
  ```
   
## <a name="query-data-using-serverless-sql-pools"></a>Interroger les données en utilisant des pools SQL serverless
Azure Synapse Analytics permet aux différents moteurs de calcul d’espace de travail de partager des bases de données et des tables entre ses pools Apache Spark serverless et son pool SQL serverless. Cela est possible grâce à la capacité Synapse de [gestion des métadonnées partagées](../metadata/overview.md). En conséquence, les bases de données créées avec Spark et leurs tables Parquet deviennent visibles dans le pool SQL serverless de l’espace de travail.

Pour interroger votre table Apache Spark à l’aide de votre pool SQL serverless :
   1. Une fois que vous avez enregistré votre table Apache Spark, basculez vers l’onglet **données**.
   
   2. Sous **Espaces de travail**, recherchez la table Apache Spark que vous venez de créer et sélectionnez **Nouveau script SQL** puis **Sélectionner les 100 premières lignes**. 
      
      :::image type="content" source="../spark/media/apache-spark-power-bi/query-spark-table-with-sql.png" alt-text="Requête SQL." border="true":::

   3. Vous pouvez continuer à affiner votre requête ou même visualiser vos résultats à l’aide des options de graphiques SQL.

## <a name="connect-to-power-bi"></a>Se connecter à Power BI
Ensuite, nous allons connecter notre pool SQL serverless à notre espace de travail Power BI. Une fois que vous avez connecté votre espace de travail, vous pouvez créer des rapports Power BI directement à partir d’Azure Synapse Analytics et de Power BI Desktop.

>[!Note]
> Avant de commencer, vous devez configurer un service lié à votre [espace de travail Power BI](../quickstart-power-bi.md) et télécharger [Power BI Desktop](https://docs.microsoft.com/power-bi/service-create-the-new-workspaces).  

Pour connecter notre pool SQL serverless à notre espace de travail Power BI :

1.  Accédez à l’onglet **Jeux de données Power BI** et sélectionnez l’option **+ Nouveau jeu de données**. À partir de l’invite, téléchargez le fichier .pbids à partir de la base de données SQL Analytics que vous souhaitez utiliser comme source de données. 
   :::image type="content" source="../spark/media/apache-spark-power-bi/power-bi-datasets.png" alt-text="Jeux de données Power BI." border="true":::

2.  Ouvrez le fichier avec Power BI Desktop pour créer un jeu de données. Une fois que vous avez ouvert le fichier, connectez-vous à la base de données SQL Server à l’aide du **compte Microsoft** et de l’option **Importer**. 
   

## <a name="create-a-power-bi-report"></a>Créer un rapport Power BI
1. Dans Power BI Desktop, vous pouvez maintenant ajouter un graphique **Influenceurs clés** à votre rapport.
   
   1. Faites glisser la colonne Moyenne de *tipAmount* jusqu’à l’axe **Analyser**.
   
   2. Faites glisser les colonnes **weekdayString**, Moyenne de **tripDistance** et Moyenne de **tripTimeSecs** jusqu’à l’axe **Expliquer par**. 
   
   :::image type="content" source="../spark/media/apache-spark-power-bi/power-bi-desktop.png" alt-text="Power BI Desktop." border="true":::

2. Sous l’onglet Accueil de Power BI Desktop, sélectionnez **Publier** et **Enregistrer** les modifications. Entrez un nom de fichier et enregistrez ce rapport dans l’*espace de travail NycTaxiTutorial*.
   
3. En outre, vous pouvez également créer des visualisations Power BI à partir de votre espace de travail Azure Synapse Analytics. Pour ce faire, accédez à l’onglet **Développer** dans votre espace de travail Azure Synapse, et ouvrez l’onglet Power BI. À partir de là, vous pouvez sélectionner votre rapport et continuer à créer des visualisations supplémentaires. 
   
   :::image type="content" source="../spark/media/apache-spark-power-bi/power-bi-synapse.png" alt-text="Espace de travail Azure Synapse Analytics." border="true":::

Pour plus d’informations sur la façon de créer un jeu de données via SQL serverless et de vous connecter à Power BI, vous pouvez consulter ce tutoriel sur la [connexion à Power BI Desktop](../../synapse-analytics/sql/tutorial-connect-power-bi-desktop.md).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les fonctionnalités de visualisation des données dans Azure Synapse Analytics, consultez les documents et les tutoriels suivants :
   - [Visualiser des données avec des pools Apache Spark serverless](../spark/apache-spark-data-visualization-tutorial.md)
   - [Vue d’ensemble de la visualisation des données avec des pools Apache Spark](../spark/apache-spark-data-visualization.md)
