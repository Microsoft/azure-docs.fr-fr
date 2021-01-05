---
title: 'Tutoriel : Utiliser R dans un contexte de calcul Spark dans Azure HDInsight'
description: 'Tutoriel : Bien démarrer avec R et Spark sur un cluster Azure HDInsight Machine Learning Services.'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/21/2019
ms.openlocfilehash: 6e727bc7ad8b7f0b8a04c48f3abd1f1ac0806c66
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545903"
---
# <a name="tutorial-use-r-in-a-spark-compute-context-in-azure-hdinsight"></a>Tutoriel : Utiliser R dans un contexte de calcul Spark dans Azure HDInsight

Ce tutoriel fournit une présentation étape par étape de l’utilisation des fonctions R dans Apache Spark qui s’exécutent sur un cluster Azure HDInsight Machine Learning Services.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Télécharger les exemples de données vers le stockage local
> * Copier les données dans le stockage par défaut
> * Configurer un jeu de données
> * Créer des sources de données
> * Créer un contexte de calcul pour Spark
> * Ajuster un modèle linéaire
> * Utiliser les fichiers XDF composites
> * Convertir de XDF en CSV

## <a name="prerequisites"></a>Prérequis

* Un cluster Azure HDInsight Machine Learning Services. Consultez [Créer des clusters Apache Hadoop dans HDInsight à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **ML Services** comme **Type de cluster** .

## <a name="connect-to-rstudio-server"></a>Se connecter à RStudio Server

RStudio Server s’exécute sur le nœud de périphérie du cluster. Accédez au site suivant (où *CLUSTERNAME* dans l’URL correspond au nom du cluster HDInsight Machine Learning Services que vous avez créé) :

```
https://CLUSTERNAME.azurehdinsight.net/rstudio/
```

La première fois que vous vous connectez, vous vous authentifiez deux fois. À la première invite d’authentification, fournissez le nom d’utilisateur et le mot de passe de l’administrateur du cluster (par défaut, il s’agit d’ *admin* ). À la deuxième invite d’authentification, fournissez le nom d’utilisateur et le mot de passe SSH (par défaut, il s’agit de *sshuser* ). Les connexions suivantes ne nécessitent que les informations d’identification SSH.

## <a name="download-the-sample-data-to-local-storage"></a>Télécharger les exemples de données vers le stockage local

Le jeu de données *Airline 2012 On-Time Data Set* se compose de 12 fichiers séparés par des virgules qui contiennent des informations sur les arrivées et les départs de tous les vols commerciaux aux États-Unis au cours de l’année 2012. Ce jeu de données est volumineux, avec plus de six millions d’observations.

1. Initialisez quelques variables d’environnement. Dans la console RStudio Server, entrez le code suivant :

    ```R
    bigDataDirRoot <- "/tutorial/data" # root directory on cluster default storage
    localDir <- "/tmp/AirOnTimeCSV2012" # directory on edge node
    remoteDir <- "https://packages.revolutionanalytics.com/datasets/AirOnTimeCSV2012" # location of data
    ```

1. Dans le volet droit, sélectionnez l’onglet **Environnement** . Les variables sont affichées sous **Valeurs** .

    ![Console web HDInsight RStudio](./media/ml-services-tutorial-spark-compute/hdinsight-rstudio-image.png)

1. Créez un répertoire local et téléchargez les exemples de données. Dans RStudio, entrez le code suivant :

    ```R
    # Create local directory
    dir.create(localDir)
    
    # Download data to the tmp folder(local)
    download.file(file.path(remoteDir, "airOT201201.csv"), file.path(localDir, "airOT201201.csv"))
    download.file(file.path(remoteDir, "airOT201202.csv"), file.path(localDir, "airOT201202.csv"))
    download.file(file.path(remoteDir, "airOT201203.csv"), file.path(localDir, "airOT201203.csv"))
    download.file(file.path(remoteDir, "airOT201204.csv"), file.path(localDir, "airOT201204.csv"))
    download.file(file.path(remoteDir, "airOT201205.csv"), file.path(localDir, "airOT201205.csv"))
    download.file(file.path(remoteDir, "airOT201206.csv"), file.path(localDir, "airOT201206.csv"))
    download.file(file.path(remoteDir, "airOT201207.csv"), file.path(localDir, "airOT201207.csv"))
    download.file(file.path(remoteDir, "airOT201208.csv"), file.path(localDir, "airOT201208.csv"))
    download.file(file.path(remoteDir, "airOT201209.csv"), file.path(localDir, "airOT201209.csv"))
    download.file(file.path(remoteDir, "airOT201210.csv"), file.path(localDir, "airOT201210.csv"))
    download.file(file.path(remoteDir, "airOT201211.csv"), file.path(localDir, "airOT201211.csv"))
    download.file(file.path(remoteDir, "airOT201212.csv"), file.path(localDir, "airOT201212.csv"))
    ```

    Le téléchargement devrait prendre environ 9 minutes 30.

## <a name="copy-the-data-to-default-storage"></a>Copier les données dans le stockage par défaut

L’emplacement HDFS (Hadoop Distributed File System) est spécifié avec la variable `airDataDir`. Dans RStudio, entrez le code suivant :

```R
# Set directory in bigDataDirRoot to load the data into
airDataDir <- file.path(bigDataDirRoot,"AirOnTimeCSV2012")

# Create directory (default storage)
rxHadoopMakeDir(airDataDir)

# Copy data from local storage to default storage
rxHadoopCopyFromLocal(localDir, bigDataDirRoot)
    
# Optional. Verify files
rxHadoopListFiles(airDataDir)
```

Cette étape devrait prendre environ 10 secondes.

## <a name="set-up-a-dataset"></a>Configurer un jeu de données

1. Créez un objet de système de fichiers qui utilise les valeurs par défaut. Dans RStudio, entrez le code suivant :

    ```R
    # Define the HDFS (WASB) file system
    hdfsFS <- RxHdfsFileSystem()
    ```

1. Les fichiers CSV originaux ayant des noms de variables plutôt complexes, vous fournissez une liste *colInfo* pour simplifier leur gestion. Dans RStudio, entrez le code suivant :

    ```R
    airlineColInfo <- list(
         MONTH = list(newName = "Month", type = "integer"),
        DAY_OF_WEEK = list(newName = "DayOfWeek", type = "factor",
            levels = as.character(1:7),
            newLevels = c("Mon", "Tues", "Wed", "Thur", "Fri", "Sat",
                          "Sun")),
        UNIQUE_CARRIER = list(newName = "UniqueCarrier", type =
                                "factor"),
        ORIGIN = list(newName = "Origin", type = "factor"),
        DEST = list(newName = "Dest", type = "factor"),
        CRS_DEP_TIME = list(newName = "CRSDepTime", type = "integer"),
        DEP_TIME = list(newName = "DepTime", type = "integer"),
        DEP_DELAY = list(newName = "DepDelay", type = "integer"),
        DEP_DELAY_NEW = list(newName = "DepDelayMinutes", type =
                             "integer"),
        DEP_DEL15 = list(newName = "DepDel15", type = "logical"),
        DEP_DELAY_GROUP = list(newName = "DepDelayGroups", type =
                               "factor",
           levels = as.character(-2:12),
           newLevels = c("< -15", "-15 to -1","0 to 14", "15 to 29",
                         "30 to 44", "45 to 59", "60 to 74",
                         "75 to 89", "90 to 104", "105 to 119",
                         "120 to 134", "135 to 149", "150 to 164",
                         "165 to 179", ">= 180")),
        ARR_DELAY = list(newName = "ArrDelay", type = "integer"),
        ARR_DELAY_NEW = list(newName = "ArrDelayMinutes", type =
                             "integer"),  
        ARR_DEL15 = list(newName = "ArrDel15", type = "logical"),
        AIR_TIME = list(newName = "AirTime", type =  "integer"),
        DISTANCE = list(newName = "Distance", type = "integer"),
        DISTANCE_GROUP = list(newName = "DistanceGroup", type =
                             "factor",
         levels = as.character(1:11),
         newLevels = c("< 250", "250-499", "500-749", "750-999",
             "1000-1249", "1250-1499", "1500-1749", "1750-1999",
             "2000-2249", "2250-2499", ">= 2500")))
    
    varNames <- names(airlineColInfo)
    ```

## <a name="create-data-sources"></a>Créer des sources de données

Dans un contexte de calcul Spark, vous pouvez créer des sources de données en utilisant les fonctions suivantes :

|Fonction | Description |
|---------|-------------|
|`RxTextData` | Texte délimité en tant que source de données. |
|`RxXdfData` | Données dans le format de fichier de données XDF. Dans RevoScaleR, le format de fichier XDF est modifié pour que Hadoop puisse stocker les données dans un ensemble composite de fichiers plutôt que dans un seul fichier. |
|`RxHiveData` | Génère un objet Source de données Hive.|
|`RxParquetData` | Génère un objet Source de données Parquet.|
|`RxOrcData` | Génère un objet Source de données Orc.|

Créez un objet [RxTextData](/machine-learning-server/r-reference/revoscaler/rxtextdata) en utilisant les fichiers que vous avez copiés dans HDFS. Dans RStudio, entrez le code suivant :

```R
airDS <- RxTextData( airDataDir,
                        colInfo = airlineColInfo,
                        varsToKeep = varNames,
                        fileSystem = hdfsFS ) 
```

## <a name="create-a-compute-context-for-spark"></a>Créer un contexte de calcul pour Spark

Pour charger des données et exécuter des analyses sur les nœuds Worker, vous devez définir le contexte de calcul dans votre script sur [RxSpark](/machine-learning-server/r-reference/revoscaler/rxspark). Dans ce contexte, les fonctions R répartissent automatiquement la charge de travail sur tous les nœuds Worker sans exigence intégrée pour la gestion des tâches ou la file d’attente. Le contexte de calcul Spark est établi par `RxSpark` ou `rxSparkConnect()` pour créer ce contexte, et utilise `rxSparkDisconnect()` pour revenir à un contexte de calcul local. Dans RStudio, entrez le code suivant :

```R
# Define the Spark compute context
mySparkCluster <- RxSpark()

# Set the compute context
rxSetComputeContext(mySparkCluster)
```

## <a name="fit-a-linear-model"></a>Ajuster un modèle linéaire

1. Utilisez la fonction [rxLinMod](/machine-learning-server/r-reference/revoscaler/rxlinmod) pour adapter un modèle linéaire en utilisant votre source de données `airDS`. Dans RStudio, entrez le code suivant :

    ```R
    system.time(
         delayArr <- rxLinMod(ArrDelay ~ DayOfWeek, data = airDS,
              cube = TRUE)
    )
    ```
    
    Cette étape devrait prendre deux à trois minutes.

1. Affichez les résultats. Dans RStudio, entrez le code suivant :

    ```R
    summary(delayArr)
    ```

    Les résultats suivants doivent s’afficher :

    ```output
    Call:
    rxLinMod(formula = ArrDelay ~ DayOfWeek, data = airDS, cube = TRUE)
    
    Cube Linear Regression Results for: ArrDelay ~ DayOfWeek
    Data: airDataXdf (RxXdfData Data Source)
    File name: /tutorial/data/AirOnTimeCSV2012
    Dependent variable(s): ArrDelay
    Total independent variables: 7 
    Number of valid observations: 6005381
    Number of missing observations: 91381 
     
    Coefficients:
                   Estimate Std. Error t value Pr(>|t|)     | Counts
    DayOfWeek=Mon   3.54210    0.03736   94.80 2.22e-16 *** | 901592
    DayOfWeek=Tues  1.80696    0.03835   47.12 2.22e-16 **_ | 855805
    DayOfWeek=Wed   2.19424    0.03807   57.64 2.22e-16 _*_ | 868505
    DayOfWeek=Thur  4.65502    0.03757  123.90 2.22e-16 _*_ | 891674
    DayOfWeek=Fri   5.64402    0.03747  150.62 2.22e-16 _*_ | 896495
    DayOfWeek=Sat   0.91008    0.04144   21.96 2.22e-16 _*_ | 732944
    DayOfWeek=Sun   2.82780    0.03829   73.84 2.22e-16 _*_ | 858366
    ---
    Signif. codes:  0 ‘_*_’ 0.001 ‘_*’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    Residual standard error: 35.48 on 6005374 degrees of freedom
    Multiple R-squared: 0.001827 (as if intercept included)
    Adjusted R-squared: 0.001826 
    F-statistic:  1832 on 6 and 6005374 DF,  p-value: < 2.2e-16 
    Condition number: 1 
    ```

    Les résultats indiquent que vous avez traité toutes les données, six millions d’observations, en utilisant tous les fichiers CSV dans le répertoire spécifié. Comme vous avez spécifié `cube = TRUE`, vous avez un coefficient estimé pour chaque jour de la semaine (et non pour l’interception).

## <a name="use-composite-xdf-files"></a>Utiliser les fichiers XDF composites

Comme vous l’avez vu, vous pouvez analyser des fichiers CSV directement avec R sur Hadoop. Toutefois, vous pouvez effectuer l’analyse plus rapidement si vous stockez les données dans un format plus efficace. Le format de fichier R XDF est efficace, mais il est quelque peu modifié pour HDFS afin que les fichiers individuels demeurent dans un seul bloc HDFS. (La taille du bloc HDFS varie d’une installation à l’autre, mais elle est généralement de 64 Mo ou 128 Mo.) 

Quand vous utilisez [rxImport](/machine-learning-server/r-reference/revoscaler/rximport) sur Hadoop pour créer un ensemble de fichiers XDF composites, vous spécifiez une source de données `RxTextData` telle que `AirDS` comme inData et une source de données `RxXdfData` avec un système de fichiers HDFS comme argument outFile. Vous pouvez ensuite utiliser l’objet `RxXdfData` comme argument de données lors des analyses R ultérieures.

1. Définissez un objet `RxXdfData`. Dans RStudio, entrez le code suivant :

    ```R
    airDataXdfDir <- file.path(bigDataDirRoot,"AirOnTimeXDF2012")
    
    airDataXdf <- RxXdfData( airDataXdfDir,
                            fileSystem = hdfsFS )
    ```

1. Définissez une taille de bloc de 250 000 lignes et spécifiez que nous lisons toutes les données. Dans RStudio, entrez le code suivant :

    ```R
    blockSize <- 250000
    numRowsToRead = -1
    ```

1. Importez les données à l’aide de `rxImport`. Dans RStudio, entrez le code suivant :

    ```R
    rxImport(inData = airDS,
             outFile = airDataXdf,
             rowsPerRead = blockSize,
             overwrite = TRUE,
             numRows = numRowsToRead )
    ```
    
    Cette étape devrait prendre quelques minutes.

1. Réestimez le même modèle linéaire en utilisant la nouvelle source de données plus rapide. Dans RStudio, entrez le code suivant :

    ```R
    system.time(
         delayArr <- rxLinMod(ArrDelay ~ DayOfWeek, data = airDataXdf,
              cube = TRUE)
    )
    ```
    
    Cette étape devrait prendre moins d’une minute.

1. Affichez les résultats. Les résultats doivent être identiques à ceux des fichiers CSV. Dans RStudio, entrez le code suivant :

    ```R
    summary(delayArr)
    ```

## <a name="convert-xdf-to-csv"></a>Convertir de XDF en CSV

### <a name="in-a-spark-context"></a>Dans un contexte Spark

Si vous avez converti vos fichiers CSV au format de fichier XDF afin d’optimiser les performances lors l’exécution des analyses, mais que vous souhaitez maintenant reconvertir vos données en CSV, vous pouvez le faire à l’aide de [rxDataStep](/machine-learning-server/r-reference/revoscaler/rxdatastep).

Pour créer un dossier de fichiers CSV, commencez par créer un objet `RxTextData` en utilisant un nom de répertoire comme argument de fichier. Cet objet représente le dossier dans lequel créer les fichiers CSV. Ce répertoire est créé lorsque vous exécutez `rxDataStep`. Ensuite, pointez sur cet objet `RxTextData` dans l’argument `outFile` du `rxDataStep`. Chaque fichier CSV créé est nommé en fonction du nom du répertoire et suivi d’un numéro.

Supposez que vous souhaitez écrire un dossier de fichiers CSV dans le système de fichiers HDFS à partir de votre XDF composite `airDataXdf` après avoir effectué la régression logistique et la prédiction, de sorte que les nouveaux fichiers CSV contiennent les valeurs prévues et les résidus. Dans RStudio, entrez le code suivant :

```R
airDataCsvDir <- file.path(bigDataDirRoot,"AirDataCSV2012")
airDataCsvDS <- RxTextData(airDataCsvDir,fileSystem=hdfsFS)
rxDataStep(inData=airDataXdf, outFile=airDataCsvDS)
```

Cette étape devrait prendre environ 2 minutes 30.

Le `rxDataStep` a écrit un fichier CSV pour chaque fichier XDFD présent dans le fichier XDF composite en entrée. Il s’agit du comportement par défaut pour l’écriture de fichiers CSV à partir de fichiers XDF composites vers HDFS quand le contexte de calcul est défini sur `RxSpark`.

### <a name="in-a-local-context"></a>Dans un contexte local

Quand vous avez terminé d’effectuer vos analyses, vous pouvez également rétablir votre contexte de calcul sur `local` afin de profiter de deux arguments dans `RxTextData` qui vous donnent un peu plus de contrôle lorsque vous écrivez des fichiers CSV dans HDFS : `createFileSet` et `rowsPerOutFile`. Quand vous affectez la valeur `TRUE` à `createFileSet`, un dossier de fichiers CSV est écrit dans le répertoire spécifié. Quand vous affectez la valeur `FALSE` à `createFileSet`, un seul fichier CSV est écrit. Vous pouvez affecter une valeur entière au deuxième argument, `rowsPerOutFile`, pour indiquer combien de lignes écrire dans chaque fichier CSV quand `createFileSet` a la valeur `TRUE`.

Dans RStudio, entrez le code suivant :

```R
rxSetComputeContext("local")
airDataCsvRowsDir <- file.path(bigDataDirRoot,"AirDataCSVRows2012")
airDataCsvRowsDS <- RxTextData(airDataCsvRowsDir, fileSystem=hdfsFS, createFileSet=TRUE, rowsPerOutFile=1000000)
rxDataStep(inData=airDataXdf, outFile=airDataCsvRowsDS)
```

Cette étape devrait prendre environ 10 minutes.

Quand vous utilisez un contexte de calcul `RxSpark`, `createFileSet` prend la valeur par défaut `TRUE` et `rowsPerOutFile` n’a aucun effet. Ainsi, si vous souhaitez créer un seul fichier CSV ou personnaliser le nombre de lignes par fichier, vous devez exécutez `rxDataStep` dans un contexte de calcul `local` (les données peuvent toujours résider dans le système de fichiers HDFS).

## <a name="final-steps"></a>Étapes finales

1. Nettoyez les données. Dans RStudio, entrez le code suivant :

    ```R
    rxHadoopRemoveDir(airDataDir)
    rxHadoopRemoveDir(airDataXdfDir)
    rxHadoopRemoveDir(airDataCsvDir)
    rxHadoopRemoveDir(airDataCsvRowsDir)
    rxHadoopRemoveDir(bigDataDirRoot)
    ```

1. Arrêtez l’application Spark distante. Dans RStudio, entrez le code suivant :

    ```R
    rxStopEngine(mySparkCluster)
    ```

1. Quittez la session R. Dans RStudio, entrez le code suivant :

    ```R
    quit()
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir terminé ce didacticiel, vous souhaiterez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées dans Stockage Azure. Vous pouvez donc supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même quand vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, mieux vaut supprimer les clusters quand ils ne sont pas utilisés.

Pour supprimer un cluster, consultez [Supprimer un cluster HDInsight à l’aide de votre navigateur, de PowerShell ou d’Azure CLI](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser des fonctions R dans Apache Spark qui s’exécutent sur un cluster HDInsight Machine Learning Services. Pour plus d’informations, consultez les articles suivants :

* [Options de contexte de calcul pour un cluster Azure HDInsight Machine Learning Services](r-server-compute-contexts.md)
* [Fonctions R pour Spark sur Hadoop](/machine-learning-server/r-reference/revoscaler/revoscaler-hadoop-functions)