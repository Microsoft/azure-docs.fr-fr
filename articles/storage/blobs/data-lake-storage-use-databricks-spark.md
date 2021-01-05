---
title: 'Tutoriel : Azure Data Lake Storage Gen2, Azure Databricks et Spark | Microsoft Docs'
description: Ce tutoriel montre comment exécuter des requêtes Spark sur un cluster Azure Databricks, afin d’accéder aux données dans un compte de stockage Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 11/19/2019
ms.author: normesta
ms.reviewer: dineshm
ms.custom: devx-track-python
ms.openlocfilehash: 5fce5871b4bd6c3e2353f7df04018e88b86ec4c7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912516"
---
# <a name="tutorial-azure-data-lake-storage-gen2-azure-databricks--spark"></a>Tutoriel : Azure Data Lake Storage Gen2, Azure Databricks et Spark

Ce tutoriel vous montre comment connecter un cluster Azure Databricks aux données contenues dans un compte de stockage Azure compatible avec Azure Data Lake Storage Gen2. Cette connexion vous permet d’exécuter en mode natif des requêtes et analyses sur des données à partir de votre cluster.

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Créer un cluster Databricks
> * Ingérer des données non structurées dans un compte de stockage
> * Exécuter une analytique sur vos données dans le stockage Blob

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Créez un compte Azure Data Lake Storage Gen2.

  Consultez [Créer un compte de stockage à utiliser avec Azure Data Lake Storage Gen2](create-data-lake-storage-account.md).

* Vérifiez que le [rôle Contributeur aux données Blob du stockage](../common/storage-auth-aad-rbac-portal.md) est attribué à votre compte d’utilisateur.

* Installez AzCopy v10. Consultez [Transférer des données avec AzCopy v10](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

* Créer un principal de service. Consultez [Procédure : Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md).

  Vous devrez faire certaines choses spécifiques pendant que vous suivrez les étapes décrites dans cet article.

  :heavy_check_mark: Au cours des étapes décrites dans la section [Attribuer un rôle à l’application](../../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) de l’article, veillez à affecter le rôle **Contributeur aux données Blob du stockage** au principal de service.

  > [!IMPORTANT]
  > Veillez à attribuer le rôle dans l’étendue du compte de stockage Data Lake Storage Gen2. Vous pouvez attribuer un rôle à l’abonnement ou au groupe de ressources parent, mais des erreurs d’autorisation sont générées tant que ces attributions de rôles ne sont pas propagées au compte de stockage.

  :heavy_check_mark: Au cours des étapes indiquées dans la section [Obtenir les valeurs pour la connexion](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de l’article, collez les valeurs de l’ID de locataire, de l’ID d’application et du secret client dans un fichier texte. Vous en aurez besoin bientôt.

### <a name="download-the-flight-data"></a>Téléchargement des données de vol

Ce tutoriel utilise des données de vol issues du Bureau of Transportation Statistics pour montrer comment effectuer une opération ETL. Vous devez télécharger ces données pour suivre ce tutoriel.

1. Accédez à [Research and Innovative Technology Administration, Bureau of Transportation Statistics](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time).

2. Cochez la case **Prezipped file** (Fichier précompressé) pour sélectionner tous les champs de données.

3. Sélectionnez le bouton **Download** (Télécharger) et enregistrez les résultats sur votre ordinateur. 

4. Décompressez le contenu du fichier compressé et notez le nom du fichier et son chemin. Vous aurez besoin de ces informations lors d’une étape ultérieure.

## <a name="create-an-azure-databricks-service"></a>Créer un service Azure Databricks

Dans cette section, vous créez un service Azure Databricks en utilisant le portail Azure.

1. Dans le portail Azure, sélectionnez **Créer une ressource** >  **Analytique** > **Azure Databricks**.

    ![Databricks sur le portail Azure](./media/data-lake-storage-use-databricks-spark/azure-databricks-on-portal.png "Databricks sur le portail Azure")

2. Sous **Service Azure Databricks**, renseignez les valeurs suivantes pour créer un service Databricks :

    |Propriété  |Description  |
    |---------|---------|
    |**Nom de l’espace de travail**     | Fournissez un nom pour votre espace de travail Databricks.  |
    |**Abonnement**     | Sélectionnez votre abonnement Azure dans la liste déroulante.        |
    |**Groupe de ressources**     | Indiquez si vous souhaitez créer un groupe de ressources Azure ou utiliser un groupe existant. Un groupe de ressources est un conteneur réunissant les ressources associées d’une solution Azure. Pour plus d’informations, consultez [Présentation des groupes de ressources Azure](../../azure-resource-manager/management/overview.md). |
    |**Lieu**     | Sélectionnez **USA Ouest 2**. Pour les autres régions disponibles, consultez [Disponibilité des services Azure par région](https://azure.microsoft.com/regions/services/).       |
    |**Niveau tarifaire**     |  Sélectionnez **Standard**.     |

    ![Créer un espace de travail Azure Databricks](./media/data-lake-storage-use-databricks-spark/create-databricks-workspace.png "Créer un service Azure Databricks")

3. La création du compte prend quelques minutes. Pour superviser l’état de l’opération, regardez la barre de progression située en haut.

4. Sélectionnez **Épingler au tableau de bord**, puis sélectionnez **Créer**.

## <a name="create-a-spark-cluster-in-azure-databricks"></a>Créer un cluster Spark dans Azure Databricks

1. Dans le portail Azure, accédez au service Databricks que vous avez créé, puis sélectionnez **Initialiser l’espace de travail**.

2. Vous êtes redirigé vers le portail Azure Databricks. Dans le portail, sélectionnez **Cluster**.

    ![Databricks sur Azure](./media/data-lake-storage-use-databricks-spark/databricks-on-azure.png "Databricks sur Azure")

3. Dans la page **Nouveau cluster**, renseignez les valeurs pour créer un cluster.

    ![Créer un cluster Databricks Spark sur Azure](./media/data-lake-storage-use-databricks-spark/create-databricks-spark-cluster.png "Créer un cluster Databricks Spark sur Azure")

    Renseignez les champs suivants et acceptez les valeurs par défaut pour les autres champs :

    - Entrez un nom pour le cluster.
     
    - Veillez à cocher la case **Arrêter après 120 minutes d’inactivité**. Spécifiez une durée (en minutes) pour arrêter le cluster, si le cluster n’est pas utilisé.

4. Sélectionnez **Créer un cluster**. Une fois que le cluster est en cours d’exécution, vous pouvez y attacher des notebooks et exécuter des travaux Spark.

## <a name="ingest-data"></a>Réception de données

### <a name="copy-source-data-into-the-storage-account"></a>Copier des données sources dans le compte de stockage

Utilisez AzCopy pour copier des données de votre fichier *.csv* dans votre compte Data Lake Storage Gen2.

1. Ouvrez une fenêtre d’invite de commandes et entrez la commande suivante pour vous connecter à votre compte de stockage.

   ```bash
   azcopy login
   ```

   Suivez les instructions qui apparaissent dans la fenêtre d’invite de commandes pour authentifier votre compte d’utilisateur.

2. Pour copier des données du compte *.csv*, entrez la commande suivante.

   ```bash
   azcopy cp "<csv-folder-path>" https://<storage-account-name>.dfs.core.windows.net/<container-name>/folder1/On_Time.csv
   ```

   * Remplacez la valeur d’espace réservé `<csv-folder-path>` par le chemin du fichier *.csv*.

   * Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

   * Remplacez la valeur d’espace réservé `<container-name>` par le nom d’un conteneur dans votre compte de stockage.

## <a name="create-a-container-and-mount-it"></a>Créer et monter un conteneur

Dans cette section, vous allez créer un conteneur et un dossier dans votre compte de stockage.

1. Dans le [portail Azure](https://portal.azure.com), accédez au service Azure Databricks que vous avez créé, puis sélectionnez **Initialiser l’espace de travail**.

2. À gauche, sélectionnez **Espace de travail**. Dans la liste déroulante **Espace de travail**, sélectionnez **Créer** > **Notebook**.

    ![Créer un notebook dans Databricks](./media/data-lake-storage-use-databricks-spark/databricks-create-notebook.png "Créer un notebook dans Databricks")

3. Dans la boîte de dialogue **Créer un bloc-notes**, entrez un nom pour le bloc-notes. Sélectionnez **Python** comme langage, puis sélectionnez le cluster Spark que vous avez créé précédemment.

4. Sélectionnez **Create** (Créer).

5. Copiez et collez le bloc de code suivant dans la première cellule, mais n’exécutez pas ce code pour l’instant.

    ```Python
    configs = {"fs.azure.account.auth.type": "OAuth",
           "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
           "fs.azure.account.oauth2.client.id": "<appId>",
           "fs.azure.account.oauth2.client.secret": "<clientSecret>",
           "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<tenant>/oauth2/token",
           "fs.azure.createRemoteFileSystemDuringInitialization": "true"}

    dbutils.fs.mount(
    source = "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/folder1",
    mount_point = "/mnt/flightdata",
    extra_configs = configs)
    ```

18. Dans ce bloc de code, remplacez les valeurs d’espace réservé `appId`, `clientSecret`, `tenant` et `storage-account-name` par celles que vous avez collectées au moment de la finalisation des prérequis de ce tutoriel. Remplacez la valeur d’espace réservé `container-name` par le nom du conteneur.

19. Appuyez sur les touches **Maj +Entrée** pour exécuter le code de ce bloc.

   Laissez ce notebook ouvert car vous allez y ajouter des commandes plus tard.

### <a name="use-databricks-notebook-to-convert-csv-to-parquet"></a>Utiliser Databricks Notebook pour convertir CSV en Parquet

Dans le notebook que vous avez créé précédemment, ajoutez une nouvelle cellule et collez-y le code suivant. 

```python
# Use the previously established DBFS mount point to read the data.
# create a data frame to read data.

flightDF = spark.read.format('csv').options(
    header='true', inferschema='true').load("/mnt/flightdata/*.csv")

# read the airline csv file and write the output to parquet format for easy query.
flightDF.write.mode("append").parquet("/mnt/flightdata/parquet/flights")
print("Done")
```

## <a name="explore-data"></a>Explorer des données

Dans une nouvelle cellule, collez le code suivant pour obtenir la liste des fichiers CSV téléchargés par le biais d’AzCopy.

```python
import os.path
import IPython
from pyspark.sql import SQLContext
display(dbutils.fs.ls("/mnt/flightdata"))
```

Pour créer un nouveau fichier et répertorier les fichiers dans le dossier *parquet/flights* dossier, exécutez ce script :

```python
dbutils.fs.put("/mnt/flightdata/1.txt", "Hello, World!", True)
dbutils.fs.ls("/mnt/flightdata/parquet/flights")
```

Avec ces exemples de code, vous avez exploré la nature hiérarchique de HDFS avec des données stockées dans un compte de stockage compatible avec Azure Data Lake Storage Gen2.

## <a name="query-the-data"></a>Interroger les données

Vous pouvez ensuite commencer à interroger les données que vous avez chargées dans votre compte de stockage. Entrez chacun des blocs de code suivants dans **Cmd 1** et appuyez sur **Cmd + Entrée** pour exécuter le script Python.

Pour créer des trames de données pour vos sources de données, exécutez le script suivant :

* Remplacez la valeur d’espace réservé `<csv-folder-path>` par le chemin du fichier *.csv*.

```python
# Copy this into a Cmd cell in your notebook.
acDF = spark.read.format('csv').options(
    header='true', inferschema='true').load("/mnt/flightdata/On_Time.csv")
acDF.write.parquet('/mnt/flightdata/parquet/airlinecodes')

# read the existing parquet file for the flights database that was created earlier
flightDF = spark.read.format('parquet').options(
    header='true', inferschema='true').load("/mnt/flightdata/parquet/flights")

# print the schema of the dataframes
acDF.printSchema()
flightDF.printSchema()

# print the flight database size
print("Number of flights in the database: ", flightDF.count())

# show the first 20 rows (20 is the default)
# to show the first n rows, run: df.show(n)
acDF.show(100, False)
flightDF.show(20, False)

# Display to run visualizations
# preferably run this in a separate cmd cell
display(flightDF)
```

Entrez ce script pour exécuter des requêtes d’analyse basiques sur les données.

```python
# Run each of these queries, preferably in a separate cmd cell for separate analysis
# create a temporary sql view for querying flight information
FlightTable = spark.read.parquet('/mnt/flightdata/parquet/flights')
FlightTable.createOrReplaceTempView('FlightTable')

# create a temporary sql view for querying airline code information
AirlineCodes = spark.read.parquet('/mnt/flightdata/parquet/airlinecodes')
AirlineCodes.createOrReplaceTempView('AirlineCodes')

# using spark sql, query the parquet file to return total flights in January and February 2016
out1 = spark.sql("SELECT * FROM FlightTable WHERE Month=1 and Year= 2016")
NumJan2016Flights = out1.count()
out2 = spark.sql("SELECT * FROM FlightTable WHERE Month=2 and Year= 2016")
NumFeb2016Flights = out2.count()
print("Jan 2016: ", NumJan2016Flights, " Feb 2016: ", NumFeb2016Flights)
Total = NumJan2016Flights+NumFeb2016Flights
print("Total flights combined: ", Total)

# List out all the airports in Texas
out = spark.sql(
    "SELECT distinct(OriginCityName) FROM FlightTable where OriginStateName = 'Texas'")
print('Airports in Texas: ', out.show(100))

# find all airlines that fly from Texas
out1 = spark.sql(
    "SELECT distinct(Reporting_Airline) FROM FlightTable WHERE OriginStateName='Texas'")
print('Airlines that fly to/from Texas: ', out1.show(100, False))
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources du compte de stockage, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"] 
> [Extraire, transformer et charger des données à l’aide d’Apache Hive sur Azure HDInsight](data-lake-storage-tutorial-extract-transform-load-hive.md)