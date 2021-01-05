---
title: Utiliser ScaleR et SparkR avec Azure HDInsight
description: Utiliser ScaleR et SparkR pour la manipulation de données et le développement de modèles avec ML Services sur Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/26/2019
ms.openlocfilehash: c12398ceacf8495a05037422a6501dc8138abc10
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92628692"
---
# <a name="combine-scaler-and-sparkr-in-hdinsight"></a>Combiner ScaleR et SparkR dans HDInsight

Ce document montre comment prévoir des retards d’arrivée de vol à l’aide d’un modèle de régression logistique **ScaleR**. L’exemple utilise des données météorologiques et de retards de vol associées via **SparkR**.

Bien que les deux packages s’exécutent sur le moteur d’exécution de Spark d’Apache Hadoop, le partage des données en mémoire est bloqué pour eux, car ils requièrent chacun leur propre session Spark respective. En attendant que ce problème soit résolu dans une prochaine version de ML Server, la solution de contournement consiste à gérer des sessions Spark qui ne se recouvrent pas et à échanger les données via des fichiers intermédiaires. Les instructions fournies ici montrent que ces exigences sont faciles à obtenir.

Cet exemple a été initialement partagé lors d’une intervention de Mario Inchiosa et Roni Burd dans le cadre de la conférence Strata 2016. Vous pouvez retrouver cette intervention sur la page [Building a Scalable Data Science Platform with R](https://channel9.msdn.com/blogs/Cloud-and-Enterprise-Premium/Building-A-Scalable-Data-Science-Platform-with-R-and-Hadoop) (Création d’une plateforme de science de données évolutive avec R).

Le code a été écrit à l’origine pour ML Server s’exécutant sur Spark dans un cluster HDInsight sur Azure. Mais le concept de l’utilisation combinée de SparkR et de ScaleR dans un seul script est également valide dans le contexte d’environnements locaux.

La procédure décrite dans ce document suppose que vous disposez d’un niveau de connaissances intermédiaire de R et de la bibliothèque [ScaleR](/machine-learning-server/r/concept-what-is-revoscaler) de ML Server. Vous découvrirez également [SparkR](https://spark.apache.org/docs/2.1.0/sparkr.html) tout au long de ce scénario.

## <a name="the-airline-and-weather-datasets"></a>Jeux de données météorologiques et jeux de données des compagnies aériennes

Les données de vol sont disponibles à partir des [archives du gouvernement des États-Unis](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236). Elles sont également disponibles au format .zip dans [AirOnTimeCSV.zip](https://packages.revolutionanalytics.com/datasets/AirOnTime87to12/AirOnTimeCSV.zip).

Les données météorologiques peuvent être téléchargées tous les mois sous forme de fichiers zip au format brut à partir du [référentiel de la National Oceanic and Atmospheric Administration](https://www.ncdc.noaa.gov/orders/qclcd/). Pour cet exemple, téléchargez les données de mai 2007 à décembre 2012. Utilisez les fichiers de données de toutes les heures et le fichier `YYYYMMMstation.txt` dans tous les zips.

## <a name="setting-up-the-spark-environment"></a>Configuration de l’environnement Spark

Utilisez le code suivant pour configurer l’environnement Spark :

```
workDir        <- '~'  
myNameNode     <- 'default' 
myPort         <- 0
inputDataDir   <- 'wasb://hdfs@myAzureAccount.blob.core.windows.net'
hdfsFS         <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

# create a persistent Spark session to reduce startup times 
#   (remember to stop it later!)
 
sparkCC        <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort, persistentRun=TRUE)

# create working directories 

rxHadoopMakeDir('/user')
rxHadoopMakeDir('user/RevoShare')
rxHadoopMakeDir('user/RevoShare/remoteuser')

(dataDir <- '/share')
rxHadoopMakeDir(dataDir)
rxHadoopListFiles(dataDir) 

setwd(workDir)
getwd()

# version of rxRoc that runs in a local CC 
rxRoc <- function(...){
  rxSetComputeContext(RxLocalSeq())
  roc <- RevoScaleR::rxRoc(...)
  rxSetComputeContext(sparkCC)
  return(roc)
}

logmsg <- function(msg) { cat(format(Sys.time(), "%Y-%m-%d %H:%M:%S"),':',msg,'\n') } 
t0 <- proc.time() 

#..start 

logmsg('Start') 
(trackers <- system("mapred job -list-active-trackers", intern = TRUE))
logmsg(paste('Number of task nodes=',length(trackers)))
```

Ensuite, ajoutez `Spark_Home` au chemin de recherche des packages R. Le fait d’ajouter cet élément au chemin de recherche vous permet d’utiliser SparkR et d’initialiser une session SparkR :

```
#..setup for use of SparkR  

logmsg('Initialize SparkR') 

.libPaths(c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib"), .libPaths()))
library(SparkR)

sparkEnvir <- list(spark.executor.instances = '10',
                   spark.yarn.executor.memoryOverhead = '8000')

sc <- sparkR.init(
  sparkEnvir = sparkEnvir,
  sparkPackages = "com.databricks:spark-csv_2.10:1.3.0"
)

sqlContext <- sparkRSQL.init(sc)
```

## <a name="preparing-the-weather-data"></a>Préparation des données météorologiques

Pour préparer les données météorologiques, créez des sous-ensembles dans les colonnes nécessaires pour la modélisation : 

- « Visibility »
- « DryBulbCelsius »
- « DewPointCelsius »
- « RelativeHumidity »
- « WindSpeed »
- « Altimeter »

Ensuite, ajoutez un code d’aéroport associé à la station météorologique et convertissez l’heure locale au format UTC.

Commencez par créer un fichier pour mapper les informations de station météorologique (WBAN) à un code d’aéroport. Le code suivant lit chacun des fichiers de données horaires météorologiques brutes, les ajoute en tant que sous-ensembles aux colonnes dont nous avons besoin, fusionne le fichier de mappage de la station météorologique, ajuste les dates à l’heure UTC, puis écrit une nouvelle version du fichier :

```
# Look up AirportID and Timezone for WBAN (weather station ID) and adjust time

adjustTime <- function(dataList)
{
  dataset0 <- as.data.frame(dataList)
  
  dataset1 <- base::merge(dataset0, wbanToAirIDAndTZDF1, by = "WBAN")

  if(nrow(dataset1) == 0) {
    dataset1 <- data.frame(
      Visibility = numeric(0),
      DryBulbCelsius = numeric(0),
      DewPointCelsius = numeric(0),
      RelativeHumidity = numeric(0),
      WindSpeed = numeric(0),
      Altimeter = numeric(0),
      AdjustedYear = numeric(0),
      AdjustedMonth = numeric(0),
      AdjustedDay = integer(0),
      AdjustedHour = integer(0),
      AirportID = integer(0)
    )
    
    return(dataset1)
  }
  
  Year <- as.integer(substr(dataset1$Date, 1, 4))
  Month <- as.integer(substr(dataset1$Date, 5, 6))
  Day <- as.integer(substr(dataset1$Date, 7, 8))
  
  Time <- dataset1$Time
  Hour <- ceiling(Time/100)
  
  Timezone <- as.integer(dataset1$TimeZone)
  
  adjustdate = as.POSIXlt(sprintf("%4d-%02d-%02d %02d:00:00", Year, Month, Day, Hour), tz = "UTC") + Timezone * 3600

  AdjustedYear = as.POSIXlt(adjustdate)$year + 1900
  AdjustedMonth = as.POSIXlt(adjustdate)$mon + 1
  AdjustedDay   = as.POSIXlt(adjustdate)$mday
  AdjustedHour  = as.POSIXlt(adjustdate)$hour
  
  AirportID = dataset1$AirportID
  Weather = dataset1[,c("Visibility", "DryBulbCelsius", "DewPointCelsius", "RelativeHumidity", "WindSpeed", "Altimeter")]
  
  data.set = data.frame(cbind(AdjustedYear, AdjustedMonth, AdjustedDay, AdjustedHour, AirportID, Weather))
  
  return(data.set)
}

wbanToAirIDAndTZDF <- read.csv("wban-to-airport-id-tz.csv")

colInfo <- list(
  WBAN = list(type="integer"),
  Date = list(type="character"),
  Time = list(type="integer"),
  Visibility = list(type="numeric"),
  DryBulbCelsius = list(type="numeric"),
  DewPointCelsius = list(type="numeric"),
  RelativeHumidity = list(type="numeric"),
  WindSpeed = list(type="numeric"),
  Altimeter = list(type="numeric")
)

weatherDF <- RxTextData(file.path(inputDataDir, "WeatherRaw"), colInfo = colInfo)

weatherDF1 <- RxTextData(file.path(inputDataDir, "Weather"), colInfo = colInfo,
                filesystem=hdfsFS)

rxSetComputeContext("localpar")
rxDataStep(weatherDF, outFile = weatherDF1, rowsPerRead = 50000, overwrite = T,
           transformFunc = adjustTime,
           transformObjects = list(wbanToAirIDAndTZDF1 = wbanToAirIDAndTZDF))
```

## <a name="importing-the-airline-and-weather-data-to-spark-dataframes"></a>Importation des données météorologiques et des données de compagnies aériennes vers Spark DataFrames

Nous allons maintenant utiliser la fonction [read.df()](https://spark.apache.org/docs/latest/api/R/read.df.html) de SparkR pour importer les données météorologiques et des compagnies aériennes vers Spark DataFrame. Cette fonction, comme beaucoup d’autres méthodes Spark, est exécutée en différé, ce qui signifie qu’elle est placée en file d’attente en vue de l’exécution, mais qu’elle n’est exécutée que lorsque cela est nécessaire.

```
airPath     <- file.path(inputDataDir, "AirOnTime08to12CSV")
weatherPath <- file.path(inputDataDir, "Weather") # pre-processed weather data
rxHadoopListFiles(airPath) 
rxHadoopListFiles(weatherPath) 

# create a SparkR DataFrame for the airline data

logmsg('create a SparkR DataFrame for the airline data') 
# use inferSchema = "false" for more robust parsing
airDF <- read.df(sqlContext, airPath, source = "com.databricks.spark.csv", 
                 header = "true", inferSchema = "false")

# Create a SparkR DataFrame for the weather data

logmsg('create a SparkR DataFrame for the weather data') 
weatherDF <- read.df(sqlContext, weatherPath, source = "com.databricks.spark.csv", 
                     header = "true", inferSchema = "true")
```

## <a name="data-cleansing-and-transformation"></a>Nettoyage et transformation des données

Ensuite, nous procédons à un nettoyage des données de la compagnie aérienne que nous avons importées pour renommer des colonnes. Nous ne conservons que les variables nécessaires et arrondissons les heures planifiées du départ à l’heure la plus proche pour permettre la fusion avec les dernières données météorologiques au départ :

```
logmsg('clean the airline data') 
airDF <- rename(airDF,
                ArrDel15 = airDF$ARR_DEL15,
                Year = airDF$YEAR,
                Month = airDF$MONTH,
                DayofMonth = airDF$DAY_OF_MONTH,
                DayOfWeek = airDF$DAY_OF_WEEK,
                Carrier = airDF$UNIQUE_CARRIER,
                OriginAirportID = airDF$ORIGIN_AIRPORT_ID,
                DestAirportID = airDF$DEST_AIRPORT_ID,
                CRSDepTime = airDF$CRS_DEP_TIME,
                CRSArrTime =  airDF$CRS_ARR_TIME
)

# Select desired columns from the flight data. 
varsToKeep <- c("ArrDel15", "Year", "Month", "DayofMonth", "DayOfWeek", "Carrier", "OriginAirportID", "DestAirportID", "CRSDepTime", "CRSArrTime")
airDF <- select(airDF, varsToKeep)

# Apply schema
coltypes(airDF) <- c("character", "integer", "integer", "integer", "integer", "character", "integer", "integer", "integer", "integer")

# Round down scheduled departure time to full hour.
airDF$CRSDepTime <- floor(airDF$CRSDepTime / 100)
```

Nous allons maintenant effectuer des opérations similaires sur les données météorologiques :

```
# Average weather readings by hour
logmsg('clean the weather data') 
weatherDF <- agg(groupBy(weatherDF, "AdjustedYear", "AdjustedMonth", "AdjustedDay", "AdjustedHour", "AirportID"), Visibility="avg",
                  DryBulbCelsius="avg", DewPointCelsius="avg", RelativeHumidity="avg", WindSpeed="avg", Altimeter="avg"
                  )

weatherDF <- rename(weatherDF,
                    Visibility = weatherDF$'avg(Visibility)',
                    DryBulbCelsius = weatherDF$'avg(DryBulbCelsius)',
                    DewPointCelsius = weatherDF$'avg(DewPointCelsius)',
                    RelativeHumidity = weatherDF$'avg(RelativeHumidity)',
                    WindSpeed = weatherDF$'avg(WindSpeed)',
                    Altimeter = weatherDF$'avg(Altimeter)'
)
```

## <a name="joining-the-weather-and-airline-data"></a>Joindre les données météorologiques et les données de compagnies aériennes

Nous allons maintenant utiliser la fonction [join()](https://spark.apache.org/docs/latest/api/R/join.html) de SparkR pour effectuer une jonction externe gauche des données météorologiques et des données de compagnies aériennes par ID et date et heure de l’aéroport de départ. La jointure externe permet de conserver tous les enregistrements de données de compagnies aériennes même s’il n’existe aucune donnée météorologique correspondante. Une fois la jonction effectuée, nous supprimons les colonnes redondantes et renommons les colonnes conservées afin d’éliminer le préfixe DataFrame entrant émanant de la jonction.

```
logmsg('Join airline data with weather at Origin Airport')
joinedDF <- SparkR::join(
  airDF,
  weatherDF,
  airDF$OriginAirportID == weatherDF$AirportID &
    airDF$Year == weatherDF$AdjustedYear &
    airDF$Month == weatherDF$AdjustedMonth &
    airDF$DayofMonth == weatherDF$AdjustedDay &
    airDF$CRSDepTime == weatherDF$AdjustedHour,
  joinType = "left_outer"
)

# Remove redundant columns
vars <- names(joinedDF)
varsToDrop <- c('AdjustedYear', 'AdjustedMonth', 'AdjustedDay', 'AdjustedHour', 'AirportID')
varsToKeep <- vars[!(vars %in% varsToDrop)]
joinedDF1 <- select(joinedDF, varsToKeep)

joinedDF2 <- rename(joinedDF1,
                    VisibilityOrigin = joinedDF1$Visibility,
                    DryBulbCelsiusOrigin = joinedDF1$DryBulbCelsius,
                    DewPointCelsiusOrigin = joinedDF1$DewPointCelsius,
                    RelativeHumidityOrigin = joinedDF1$RelativeHumidity,
                    WindSpeedOrigin = joinedDF1$WindSpeed,
                    AltimeterOrigin = joinedDF1$Altimeter
)
```

De la même manière, nous joindrons les données météorologiques et les données de compagnies aériennes en fonction de l’ID et de la date et heure de l’aéroport d’arrivée :

```
logmsg('Join airline data with weather at Destination Airport')
joinedDF3 <- SparkR::join(
  joinedDF2,
  weatherDF,
  airDF$DestAirportID == weatherDF$AirportID &
    airDF$Year == weatherDF$AdjustedYear &
    airDF$Month == weatherDF$AdjustedMonth &
    airDF$DayofMonth == weatherDF$AdjustedDay &
    airDF$CRSDepTime == weatherDF$AdjustedHour,
  joinType = "left_outer"
)

# Remove redundant columns
vars <- names(joinedDF3)
varsToDrop <- c('AdjustedYear', 'AdjustedMonth', 'AdjustedDay', 'AdjustedHour', 'AirportID')
varsToKeep <- vars[!(vars %in% varsToDrop)]
joinedDF4 <- select(joinedDF3, varsToKeep)

joinedDF5 <- rename(joinedDF4,
                    VisibilityDest = joinedDF4$Visibility,
                    DryBulbCelsiusDest = joinedDF4$DryBulbCelsius,
                    DewPointCelsiusDest = joinedDF4$DewPointCelsius,
                    RelativeHumidityDest = joinedDF4$RelativeHumidity,
                    WindSpeedDest = joinedDF4$WindSpeed,
                    AltimeterDest = joinedDF4$Altimeter
                    )
```

## <a name="save-results-to-csv-for-exchange-with-scaler"></a>Enregistrer les résultats au format CSV pour l’échange avec ScaleR

Les jonctions nécessaires avec SparkR sont alors terminées. Nous enregistrons les données du fichier final Spark DataFrame « joinedDF5 » dans un fichier CSV pour les envoyer dans ScaleR, puis nous fermons la session SparkR. Nous indiquons explicitement à SparkR d’enregistrer le fichier CSV résultant en 80 partitions distinctes afin d’établir un parallélisme suffisant dans le traitement ScaleR :

```
logmsg('output the joined data from Spark to CSV') 
joinedDF5 <- repartition(joinedDF5, 80) # write.df below will produce this many CSVs

# write result to directory of CSVs
write.df(joinedDF5, file.path(dataDir, "joined5Csv"), "com.databricks.spark.csv", "overwrite", header = "true")

# We can shut down the SparkR Spark context now
sparkR.stop()

# remove non-data files
rxHadoopRemove(file.path(dataDir, "joined5Csv/_SUCCESS"))
```

## <a name="import-to-xdf-for-use-by-scaler"></a>Importer vers XDF pour une utilisation par ScaleR

Nous pouvons utiliser le fichier CSV de la compagnie aérienne associée et les données météorologiques tels quels pour la modélisation via une source de données de texte ScaleR. Mais nous l’importons d’abord vers un fichier XDF, plus efficace lors de l’exécution de plusieurs opérations sur le jeu de données :

```
logmsg('Import the CSV to compressed, binary XDF format') 

# set the Spark compute context for ML Services 
rxSetComputeContext(sparkCC)
rxGetComputeContext()

colInfo <- list(
  ArrDel15 = list(type="numeric"),
  Year = list(type="factor"),
  Month = list(type="factor"),
  DayofMonth = list(type="factor"),
  DayOfWeek = list(type="factor"),
  Carrier = list(type="factor"),
  OriginAirportID = list(type="factor"),
  DestAirportID = list(type="factor"),
  RelativeHumidityOrigin = list(type="numeric"),
  AltimeterOrigin = list(type="numeric"),
  DryBulbCelsiusOrigin = list(type="numeric"),
  WindSpeedOrigin = list(type="numeric"),
  VisibilityOrigin = list(type="numeric"),
  DewPointCelsiusOrigin = list(type="numeric"),
  RelativeHumidityDest = list(type="numeric"),
  AltimeterDest = list(type="numeric"),
  DryBulbCelsiusDest = list(type="numeric"),
  WindSpeedDest = list(type="numeric"),
  VisibilityDest = list(type="numeric"),
  DewPointCelsiusDest = list(type="numeric")
)

joinedDF5Txt <- RxTextData(file.path(dataDir, "joined5Csv"),
                           colInfo = colInfo, fileSystem = hdfsFS)
rxGetInfo(joinedDF5Txt) 

destData <- RxXdfData(file.path(dataDir, "joined5XDF"), fileSystem = hdfsFS)

rxImport(inData = joinedDF5Txt, destData, overwrite = TRUE)

rxGetInfo(destData, getVarInfo = T)

# File name: /user/RevoShare/dev/delayDataLarge/joined5XDF 
# Number of composite data files: 80 
# Number of observations: 148619655 
# Number of variables: 22 
# Number of blocks: 320 
# Compression type: zlib 
# Variable information: 
#   Var 1: ArrDel15, Type: numeric, Low/High: (0.0000, 1.0000)
# Var 2: Year
# 26 factor levels: 1987 1988 1989 1990 1991 ... 2008 2009 2010 2011 2012
# Var 3: Month
# 12 factor levels: 10 11 12 1 2 ... 5 6 7 8 9
# Var 4: DayofMonth
# 31 factor levels: 1 3 4 5 7 ... 29 30 2 18 31
# Var 5: DayOfWeek
# 7 factor levels: 4 6 7 1 3 2 5
# Var 6: Carrier
# 30 factor levels: PI UA US AA DL ... HA F9 YV 9E VX
# Var 7: OriginAirportID
# 374 factor levels: 15249 12264 11042 15412 13930 ... 13341 10559 14314 11711 10558
# Var 8: DestAirportID
# 378 factor levels: 13303 14492 10721 11057 13198 ... 14802 11711 11931 12899 10559
# Var 9: CRSDepTime, Type: integer, Low/High: (0, 24)
# Var 10: CRSArrTime, Type: integer, Low/High: (0, 2400)
# Var 11: RelativeHumidityOrigin, Type: numeric, Low/High: (0.0000, 100.0000)
# Var 12: AltimeterOrigin, Type: numeric, Low/High: (28.1700, 31.1600)
# Var 13: DryBulbCelsiusOrigin, Type: numeric, Low/High: (-46.1000, 47.8000)
# Var 14: WindSpeedOrigin, Type: numeric, Low/High: (0.0000, 81.0000)
# Var 15: VisibilityOrigin, Type: numeric, Low/High: (0.0000, 90.0000)
# Var 16: DewPointCelsiusOrigin, Type: numeric, Low/High: (-41.7000, 29.0000)
# Var 17: RelativeHumidityDest, Type: numeric, Low/High: (0.0000, 100.0000)
# Var 18: AltimeterDest, Type: numeric, Low/High: (28.1700, 31.1600)
# Var 19: DryBulbCelsiusDest, Type: numeric, Low/High: (-46.1000, 53.9000)
# Var 20: WindSpeedDest, Type: numeric, Low/High: (0.0000, 136.0000)
# Var 21: VisibilityDest, Type: numeric, Low/High: (0.0000, 88.0000)
# Var 22: DewPointCelsiusDest, Type: numeric, Low/High: (-43.0000, 29.0000)

finalData <- RxXdfData(file.path(dataDir, "joined5XDF"), fileSystem = hdfsFS)

```

## <a name="splitting-data-for-training-and-test"></a>Fractionnement des données pour le test et l’apprentissage

Nous utilisons rxDataStep pour fractionner les données 2012 pour le test et nous conservons le reste pour la formation :

```
# split out the training data

logmsg('split out training data as all data except year 2012')
trainDS <- RxXdfData( file.path(dataDir, "finalDataTrain" ),fileSystem = hdfsFS)

rxDataStep( inData = finalData, outFile = trainDS,
            rowSelection = ( Year != 2012 ), overwrite = T )

# split out the testing data

logmsg('split out the test data for year 2012') 
testDS <- RxXdfData( file.path(dataDir, "finalDataTest" ), fileSystem = hdfsFS)

rxDataStep( inData = finalData, outFile = testDS,
            rowSelection = ( Year == 2012 ), overwrite = T )

rxGetInfo(trainDS)
rxGetInfo(testDS)
```

## <a name="train-and-test-a-logistic-regression-model"></a>Effectuer l’apprentissage et tester un modèle de régression logistique

Nous sommes maintenant prêts à créer un modèle. Pour afficher l’influence des données météorologiques sur le retard à l’heure d’arrivée, nous utilisons la routine de régression logistique de ScaleR. Nous les utilisons pour savoir si un retard d’arrivée de plus de 15 minutes est influencé par la météo dans les aéroports de départ et d’arrivée :

```
logmsg('train a logistic regression model for Arrival Delay > 15 minutes') 
formula <- as.formula(ArrDel15 ~ Year + Month + DayofMonth + DayOfWeek + Carrier +
                     OriginAirportID + DestAirportID + CRSDepTime + CRSArrTime + 
                     RelativeHumidityOrigin + AltimeterOrigin + DryBulbCelsiusOrigin +
                     WindSpeedOrigin + VisibilityOrigin + DewPointCelsiusOrigin + 
                     RelativeHumidityDest + AltimeterDest + DryBulbCelsiusDest +
                     WindSpeedDest + VisibilityDest + DewPointCelsiusDest
                   )

# Use the scalable rxLogit() function but set max iterations to 3 for the purposes of 
# this exercise 

logitModel <- rxLogit(formula, data = trainDS, maxIterations = 3)

base::summary(logitModel)
```

Nous allons maintenant voir comment ce modèle fonctionne sur les données de test en effectuant des prédictions, et en étudiant la courbe ROC et l’aire sous la courbe.

```
# Predict over test data (Logistic Regression).

logmsg('predict over the test data') 
logitPredict <- RxXdfData(file.path(dataDir, "logitPredict"), fileSystem = hdfsFS)

# Use the scalable rxPredict() function

rxPredict(logitModel, data = testDS, outData = logitPredict,
          extraVarsToWrite = c("ArrDel15"), 
          type = 'response', overwrite = TRUE)

# Calculate ROC and Area Under the Curve (AUC).

logmsg('calculate the roc and auc') 
logitRoc <- rxRoc("ArrDel15", "ArrDel15_Pred", logitPredict)
logitAuc <- rxAuc(logitRoc)
head(logitAuc)
logitAuc

plot(logitRoc)
```

## <a name="scoring-elsewhere"></a>Notation depuis un autre emplacement

Nous pouvons également utiliser le modèle pour évaluer les données sur une autre plateforme. En l’enregistrant dans un fichier RDS, puis en transférant et en important ce fichier RDS vers un environnement de scoring de destination, tel que les services Microsoft SQL Server R Services. Lors de cette opération, il est important de vérifier que les niveaux de facteur des données à évaluer correspondent à ceux sur lesquels le modèle a été généré. Pour ce faire, il suffit d’extraire et d’enregistrer les informations des colonnes associées aux données de modélisation via la fonction `rxCreateColInfo()` de ScaleR, puis d’appliquer ces informations de colonnes à la source de données d’entrée pour émettre des prévisions. Dans l’exemple de code suivant, nous avons enregistré quelques lignes du jeu de données test, puis extrait et utilisé les informations de colonnes à partir de cet exemple dans le script de prédiction :

```
# save the model and a sample of the test dataset 

logmsg('save serialized version of the model and a sample of the test data')
rxSetComputeContext('localpar') 
saveRDS(logitModel, file = "logitModel.rds")
testDF <- head(testDS, 1000)  
saveRDS(testDF    , file = "testDF.rds"    )
list.files()

rxHadoopListFiles(file.path(inputDataDir,''))
rxHadoopListFiles(dataDir)

# stop the spark engine 
rxStopEngine(sparkCC) 

logmsg('Done.')
elapsed <- (proc.time() - t0)[3]
logmsg(paste('Elapsed time=',sprintf('%6.2f',elapsed),'(sec)\n\n'))
```

## <a name="summary"></a>Résumé

Dans cet article, nous avons indiqué comment il est possible de combiner l’utilisation de SparkR pour la manipulation de données avec ScaleR pour le développement de modèles dans Hadoop Spark. Ce scénario nécessite de maintenir les sessions de Spark séparées, en n’exécutant qu’une session à la fois et d’échanger des données via des fichiers CSV. Bien que simple, ce processus devrait être encore simplifié dans une prochaine version de ML Services, quand SparkR et ScaleR pourront partager une session Spark et, par conséquent, Spark DataFrames.

## <a name="next-steps-and-more-information"></a>Étapes suivantes et informations supplémentaires

- Pour plus d’informations sur l’utilisation de ML Server sur Apache Spark, consultez le [Guide de prise en main](/machine-learning-server/r/how-to-revoscaler-spark).

- Pour obtenir des informations sur ML Services sur HDInsight, consultez [Vue d’ensemble de ML Services sur HDInsight](r-server/r-server-overview.md).

Pour plus d’informations sur l’utilisation de SparkR, consultez les pages suivantes :

- [Document Apache SparkR](https://spark.apache.org/docs/2.1.0/sparkr.html)

- [Vue d’ensemble de SparkR](/azure/databricks/spark/latest/sparkr/overview)
