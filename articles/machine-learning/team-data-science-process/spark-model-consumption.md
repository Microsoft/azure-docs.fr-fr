---
title: Opérationnaliser les modèles Machine Learning créés avec Spark– Processus TDSP (Team Data Science Process)
description: Comment charger et noter les modèles d’apprentissage stockés dans Azure Blob Storage (WASB) avec Python.
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
ms.openlocfilehash: bb38a76de41885b6f39a1c6dce7c44bcb52a4d60
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86027441"
---
# <a name="operationalize-spark-built-machine-learning-models"></a>Faire fonctionner les modèles Machine Learning créés avec Spark

Cette rubrique montre comment faire fonctionner un modèle Machine Learning (ML) à l’aide de Python sur des clusters HDInsight Spark. Elle décrit comment charger des modèles Machine Learning créés avec Spark MLlib et stockés dans Azure Blob Storage (WASB), et explique comment les noter avec des jeux de données également stockés dans WASB. Il montre comment prétraiter les données d’entrée, transformer les caractéristiques à l’aide des fonctions d’indexation et d’encodage du kit d’outils MLlib, et comment créer un objet de données point étiqueté, utilisable comme entrée de notation avec les modèles ML. Les modèles utilisés pour la notation sont les suivants : Régression linéaire, Régression logistique, Modèles de forêts aléatoires et Modèles GBT (Gradient Boosting Tree).

## <a name="spark-clusters-and-jupyter-notebooks"></a>Clusters Spark et notebooks Jupyter
Cette procédure pas à pas fournit les étapes d’installation et le code permettant de faire fonctionner un modèle ML pour un cluster HDInsight Spark 1.6 ainsi que pour un cluster Spark 2.0. Le code de ces procédures est également fourni dans les notebooks Jupyter.

### <a name="notebook-for-spark-16"></a>Notebook pour Spark 1.6
Le notebook Jupyter [pySpark-machine-learning-data-science-spark-model-consumption.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb) montre comment utiliser un modèle enregistré à l’aide de Python sur des clusters HDInsight. 

### <a name="notebook-for-spark-20"></a>Notebook pour Spark 2.0
Pour modifier le notebook Jupyter pour Spark 1.6 afin de l’utiliser avec un cluster HDInsight Spark 2.0, remplacez le fichier de code Python par [ce fichier](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Python/Spark2.0_ConsumeRFCV_NYCReg.py). Ce code montre comment utiliser les modèles créés dans Spark 2.0.


## <a name="prerequisites"></a>Prérequis

1. Vous avez besoin d’un compte Azure et d’un cluster HDInsight Spark 1.6 (ou Spark 2.0) pour effectuer cette procédure pas à pas. Pour obtenir des instructions sur la façon de satisfaire à ces exigences, voir [Vue d’ensemble de la science des données à l’aide de Spark sur Azure HDInsight](spark-overview.md). Cette rubrique contient également une description des données NYC 2013 Taxi utilisées ici, et des instructions sur l’exécution de code à partir d’un bloc-notes Jupyter sur le cluster Spark. 
2. Créez les modèles Machine Learning à noter ici en appliquant la procédure de la rubrique [Exploration et modélisation des données avec Spark](spark-data-exploration-modeling.md) pour le cluster Spark 1.6 ou les notebooks Spark 2.0. 
3. Les notebooks Spark 2.0 utilisent un jeu de données supplémentaire pour la tâche de classification, le jeu de données bien connu sur les départs à l’heure des compagnies aériennes pour les années 2011 et 2012. Une description des notebooks et des liens vers ceux-ci sont fournis dans le fichier [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) correspondant au dépôt GitHub qui les contient. En outre, le code présenté ici et dans les notebooks liés est générique et doit fonctionner sur n’importe quel cluster Spark. Si vous n’utilisez pas HDInsight Spark, les étapes de configuration et de gestion de cluster peuvent être légèrement différentes de celles indiquées ici. 

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="setup-storage-locations-libraries-and-the-preset-spark-context"></a>Configuration : emplacements de stockage, bibliothèques et contexte Spark prédéfini
Spark peut lire et écrire dans un objet blob Stockage Azure (également appelé WASB). Donc, vos données stockées dedans sont exploitables par Spark et les résultats peuvent être stockés à nouveau dans WASB.

Pour enregistrer les modèles ou les fichiers dans WASB, le chemin d’accès doit être correctement spécifié. Le conteneur par défaut associé au cluster Spark peut être référencé à l’aide d’un chemin commençant par : *« wasb// »* . L’exemple de code suivant spécifie l’emplacement des données à lire et le chemin d’accès au répertoire de stockage dans lequel la sortie du modèle est enregistrée. 

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>Définir les chemins d’accès aux emplacements de stockage dans WASB
Les modèles sont enregistrés dans : wasb:///user/remoteuser/NYCTaxi/Models. Si ce chemin d’accès n’est pas défini correctement, les modèles ne sont pas chargés en vue de leur notation.

Les résultats notés sont enregistrés dans : wasb:///user/remoteuser/NYCTaxi/ScoredResults. Si le chemin d’accès au dossier est incorrect, les résultats ne sont pas enregistrés dans ce dossier.   

> [!NOTE]
> Les chemins des fichiers peuvent être copiés et collés dans les espaces réservés à cet effet dans le code à partir de la sortie de la dernière cellule du notebook **machine-learning-data-science-spark-data-exploration-modeling.ipynb** .   
> 
> 

Voici le code pour définir les chemins de répertoires : 

```python
# LOCATION OF DATA TO BE SCORED (TEST DATA)
taxi_test_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Test.tsv";

# SET THE MODEL STORAGE DIRECTORY PATH 
# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 

# SET SCORDED RESULT DIRECTORY PATH
# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
scoredResultDir = "wasb:///user/remoteuser/NYCTaxi/ScoredResults/"; 

# FILE LOCATIONS FOR THE MODELS TO BE SCORED
logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-04-1817_40_35.796789"
linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-04-1817_44_00.993832"
randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-04-1817_42_58.899412"
randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-04-1817_44_27.204734"
BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-04-1817_43_16.354770"
BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-04-1817_44_46.206262"

# RECORD START TIME
import datetime
datetime.datetime.now()
```

**SORTIE :**

datetime.datetime(2016, 4, 25, 23, 56, 19, 229403)

### <a name="import-libraries"></a>Importer les bibliothèques
Définir le contexte Spark et importer les bibliothèques nécessaires avec le code suivant

```python
#IMPORT LIBRARIES
import pyspark
from pyspark import SparkConf
from pyspark import SparkContext
from pyspark.sql import SQLContext
import matplotlib
import matplotlib.pyplot as plt
from pyspark.sql import Row
from pyspark.sql.functions import UserDefinedFunction
from pyspark.sql.types import *
import atexit
from numpy import array
import numpy as np
import datetime
```

### <a name="preset-spark-context-and-pyspark-magics"></a>Contexte Spark prédéfini et commandes magiques PySpark
Les noyaux PySpark fournis avec les blocs-notes Jupyter comprennent un contexte prédéfini. Par conséquent, vous n’avez pas besoin de définir explicitement les contextes Spark ou Hive avant de commencer à utiliser l’application que vous développez. Ces contextes sont disponibles par défaut.

* sc : pour Spark 
* sqlContext : pour Hive

Le noyau PySpark fournit certaines « commandes magiques » prédéfinies, qui sont des commandes spéciales que vous pouvez appeler avec %%. Deux de ces commandes sont utilisées dans ces exemples de code.

* **%%local** Indique que le code des lignes suivantes est exécuté localement. Le code doit être du code Python valide.
* **%%sql -o \<variable name>** 
* Exécute une requête Hive sur sqlContext. Si le paramètre -o est passé, le résultat de la requête est conservé dans le contexte Python %%local en tant que tableau de données Pandas.

Pour plus d’informations sur les noyaux pour blocs-notes Jupyter et sur les « commandes magiques » qu’ils fournissent, voir [Noyaux disponibles pour les blocs-notes Jupyter avec les clusters HDInsight Spark Linux sur HDInsight](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

## <a name="ingest-data-and-create-a-cleaned-data-frame"></a>Recevoir les données et créer une trame de données nettoyée
Cette section contient le code d’une série de tâches nécessaires à la réception de l’échantillon de données à modéliser. Lire un échantillon de 0,1 % du fichier contenant les trajets et prix de taxi (stocké dans un fichier TSV), formater les données et créer une trame de données propre.

Les fichiers relatifs aux courses en taxi et aux tarifs ont été joints conformément à la procédure décrite à la rubrique : [Processus TDSP (Team Data Science Process) en action : utilisation de clusters Hadoop HDInsight](hive-walkthrough.md).

```python
# INGEST DATA AND CREATE A CLEANED DATA FRAME

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT FILE FROM PUBLIC BLOB
taxi_test_file = sc.textFile(taxi_test_file_loc)

# GET SCHEMA OF THE FILE FROM HEADER
taxi_header = taxi_test_file.filter(lambda l: "medallion" in l)

# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
taxi_temp = taxi_test_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))

# GET SCHEMA OF THE FILE FROM HEADER
schema_string = taxi_test_file.first()
fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
fields[7].dataType = IntegerType() #Pickup hour
fields[8].dataType = IntegerType() # Pickup week
fields[9].dataType = IntegerType() # Weekday
fields[10].dataType = IntegerType() # Passenger count
fields[11].dataType = FloatType() # Trip time in secs
fields[12].dataType = FloatType() # Trip distance
fields[19].dataType = FloatType() # Fare amount
fields[20].dataType = FloatType() # Surcharge
fields[21].dataType = FloatType() # Mta_tax
fields[22].dataType = FloatType() # Tip amount
fields[23].dataType = FloatType() # Tolls amount
fields[24].dataType = FloatType() # Total amount
fields[25].dataType = IntegerType() # Tipped or not
fields[26].dataType = IntegerType() # Tip class
taxi_schema = StructType(fields)

# CREATE DATA FRAME
taxi_df_test = sqlContext.createDataFrame(taxi_temp, taxi_schema)

# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
taxi_df_test_cleaned = taxi_df_test.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
    .drop('direct_distance').drop('surcharge')\
    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_test_cleaned.cache()
taxi_df_test_cleaned.count()

# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
taxi_df_test_cleaned.registerTempTable("taxi_test")

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 46,37 secondes

## <a name="prepare-data-for-scoring-in-spark"></a>Préparer les données à la notation dans Spark
Cette section montre comment indexer, encoder et mettre à l’échelle des caractéristiques catégorielles en vue de leur utilisation dans les algorithmes d’apprentissage contrôlé MLlib pour la classification et la régression.

### <a name="feature-transformation-index-and-encode-categorical-features-for-input-into-models-for-scoring"></a>Transformation de caractéristiques : indexer et encoder des caractéristiques catégorielles en vue de leur utilisation dans des modèles à noter
Cette section montre comment indexer les données catégorielles à l’aide d’un `StringIndexer` et encoder les caractéristiques avec entrée de `OneHotEncoder` dans les modèles.

[StringIndexer](https://spark.apache.org/docs/latest/ml-features.html#stringindexer) encode une colonne de libellés en une colonne d’index de libellés. Les index sont classés par fréquence de libellé. 

[OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) mappe une colonne d’index de libellés à une colonne de vecteurs binaires, contenant au plus une valeur 1. Cet encodage autorise les algorithmes qui appliquent des caractéristiques numériques continues, comme la régression logistique, à des caractéristiques catégorielles.

```python
#INDEX AND ONE-HOT ENCODE CATEGORICAL FEATURES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer

# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_test 
"""
taxi_df_test_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_test_with_newFeatures.cache()
taxi_df_test_with_newFeatures.count()

# INDEX AND ONE-HOT ENCODING
stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
model = stringIndexer.fit(taxi_df_test_with_newFeatures) # Input data-frame is the cleaned one from above
indexed = model.transform(taxi_df_test_with_newFeatures)
encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
encoded1 = encoder.transform(indexed)

# INDEX AND ENCODE RATE_CODE
stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
model = stringIndexer.fit(encoded1)
indexed = model.transform(encoded1)
encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
encoded2 = encoder.transform(indexed)

# INDEX AND ENCODE PAYMENT_TYPE
stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
model = stringIndexer.fit(encoded2)
indexed = model.transform(encoded2)
encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
encoded3 = encoder.transform(indexed)

# INDEX AND ENCODE TRAFFIC TIME BINS
stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
model = stringIndexer.fit(encoded3)
indexed = model.transform(encoded3)
encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
encodedFinal = encoder.transform(indexed)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 5,37 secondes

### <a name="create-rdd-objects-with-feature-arrays-for-input-into-models"></a>Créer des objets RDD avec des tableaux de caractéristiques à intégrer dans des modèles
Cette section contient le code montrant comment indexer des données textuelles catégorielles en un objet RDD et les encoder linéairement pour qu’elles puissent former et tester la régression logistique de MLlib et d’autres modèles de classification. Les données indexées sont stockées dans des objets [RDD (Resilient Distributed Dataset)](https://spark.apache.org/docs/latest/api/java/org/apache/spark/rdd/RDD.html) . Les objets RDD constituent l’abstraction de base dans Spark. Un objet RDD représente une collection immuable et partitionnée d’éléments qui peuvent faire l’objet d’un traitement en parallèle avec Spark.

Il contient également du code montrant comment mettre à l’échelle des données avec le `StandardScalar` fourni par MLlib, en vue d’une utilisation dans la régression linéaire avec SGD (Stochastic Gradient Descent), un algorithme répandu permettant de former une large gamme de modèles Machine Learning. [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) permet de mettre à l’échelle les caractéristiques en fonction de la variance d’unité. La mise à l’échelle des caractéristiques, également appelée normalisation des données, garantit que les caractéristiques aux valeurs très dispersées sont pondérées dans la fonction cible. 

```python
# CREATE RDD OBJECTS WITH FEATURE ARRAYS FOR INPUT INTO MODELS

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT LIBRARIES
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.feature import StandardScaler, StandardScalerModel
from pyspark.mllib.util import MLUtils
from numpy import array

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingBinary(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    return  features

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC REGRESSION MODELS
def parseRowOneHotBinary(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    return  features

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingRegression(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    return  features

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
def parseRowOneHotRegression(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    return  features

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTESTbinary = encodedFinal.map(parseRowIndexingBinary)
oneHotTESTbinary = encodedFinal.map(parseRowOneHotBinary)

# FOR REGRESSION CLASSIFICATION TRAINING AND TESTING
indexedTESTreg = encodedFinal.map(parseRowIndexingRegression)
oneHotTESTreg = encodedFinal.map(parseRowOneHotRegression)

# SCALING FEATURES FOR LINEARREGRESSIONWITHSGD MODEL
scaler = StandardScaler(withMean=False, withStd=True).fit(oneHotTESTreg)
oneHotTESTregScaled = scaler.transform(oneHotTESTreg)

# CACHE RDDS IN MEMORY
indexedTESTbinary.cache();
oneHotTESTbinary.cache();
indexedTESTreg.cache();
oneHotTESTreg.cache();
oneHotTESTregScaled.cache();

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 11,72 secondes

## <a name="score-with-the-logistic-regression-model-and-save-output-to-blob"></a>Noter avec le modèle de régression logistique et enregistrer la sortie dans l’objet BLOB
Le code de cette section montre comment charger un modèle de régression logistique enregistré dans Azure Blob Storage et l’utiliser pour prédire si un pourboire est payé lors d’un trajet en taxi, le noter avec des mesures de classification standard, l’enregistrer puis représenter graphiquement les résultats dans Blob Storage. Les résultats notés sont stockés dans des objets RDD. 

```python
# SCORE AND EVALUATE LOGISTIC REGRESSION MODEL

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT LIBRARIES
from pyspark.mllib.classification import LogisticRegressionModel

## LOAD SAVED MODEL
savedModel = LogisticRegressionModel.load(sc, logisticRegFileLoc)
predictions = oneHotTESTbinary.map(lambda features: (float(savedModel.predict(features))))

## SAVE SCORED RESULTS (RDD) TO BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp + ".txt";
dirfilename = scoredResultDir + logisticregressionfilename;
predictions.saveAsTextFile(dirfilename)


# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 19,22 secondes

## <a name="score-a-linear-regression-model"></a>Noter un modèle de régression linéaire
Nous avons utilisé [LinearRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) pour former un modèle de régression linéaire utilisant SGD (Stochastic Gradient Descent) à des fins d’optimisation pour prédire le montant des pourboires payés. 

Le code de cette section montre comment charger un modèle de régression linéaire à partir d’Azure Blob Storage, le noter avec des variables mises à l’échelle, puis réenregistrer les résultats dans l’objet blob.

```python
#SCORE LINEAR REGRESSION MODEL

# RECORD START TIME
timestart = datetime.datetime.now()

#LOAD LIBRARIES
from pyspark.mllib.regression import LinearRegressionWithSGD, LinearRegressionModel

# LOAD MODEL AND SCORE USING ** SCALED VARIABLES **
savedModel = LinearRegressionModel.load(sc, linearRegFileLoc)
predictions = oneHotTESTregScaled.map(lambda features: (float(savedModel.predict(features))))

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
dirfilename = scoredResultDir + linearregressionfilename;
predictions.saveAsTextFile(dirfilename)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 16,63 secondes

## <a name="score-classification-and-regression-random-forest-models"></a>Noter les modèles Forêts aléatoires de classification et de régression
Le code de cette section montre comment charger les modèles Forêts aléatoires de classification et de régression enregistrés dans Azure Blob Storage, noter leurs performances avec des mesures standard de classificateur et de régression, puis réenregistrer les résultats dans Azure Blob Storage.

[forêts aléatoires](https://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) sont des ensembles d’arbres de décision.  Elles combinent de nombreux arbres de décision pour réduire le risque de surajustement. Elles gèrent les caractéristiques catégorielles, prennent en compte le paramètre de classification multiclasse, ne requièrent aucune mise à l’échelle des caractéristiques et peuvent capturer les non-linéarités ainsi que les interactions entre les caractéristiques. Les forêts aléatoires constituent l’un des modèles Machine Learning les plus performants pour la classification et la régression.

[spark.mllib](https://spark.apache.org/mllib/) prend en charge les forêts aléatoires pour la classification binaire et multiclasse et pour la régression, à l’aide des caractéristiques continues et catégorielles. 

```python
# SCORE RANDOM FOREST MODELS FOR CLASSIFICATION AND REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT MLLIB LIBRARIES    
from pyspark.mllib.tree import RandomForest, RandomForestModel


# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
savedModel = RandomForestModel.load(sc, randomForestClassificationFileLoc)
predictions = savedModel.predict(indexedTESTbinary)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfclassificationfilename = "RandomForestClassification_" + datestamp + ".txt";
dirfilename = scoredResultDir + rfclassificationfilename;
predictions.saveAsTextFile(dirfilename)


# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
savedModel = RandomForestModel.load(sc, randomForestRegFileLoc)
predictions = savedModel.predict(indexedTESTreg)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfregressionfilename = "RandomForestRegression_" + datestamp + ".txt";
dirfilename = scoredResultDir + rfregressionfilename;
predictions.saveAsTextFile(dirfilename)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 31,07 secondes

## <a name="score-classification-and-regression-gradient-boosting-tree-models"></a>Noter les modèles GBT de classification et de régression
Le code de cette section montre comment charger les modèles GBT de classification et de régression enregistrés dans Azure Blob Storage, noter leurs performances avec des mesures standard de classificateur et de régression, puis réenregistrer les résultats dans Azure Blob Storage. 

**spark.mllib** prend en charge les arbres GBTS pour la classification binaire et la régression en utilisant des caractéristiques continues et catégorielles. 

Les [GBTS (Gradient Boosting Trees)](https://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) sont des ensembles d’arbres de décision. Ils aident les arbres de décision à minimiser itérativement une fonction de perte. Ils gèrent les caractéristiques catégorielles, ne demandent aucune mise à l’échelle des caractéristiques et peuvent capturer les non-linéarités ainsi que les interactions entre les caractéristiques. Cet algorithme s’utilise également dans le paramétrage de classification multiclasse.

```python
# SCORE GRADIENT BOOSTING TREE MODELS FOR CLASSIFICATION AND REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT MLLIB LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

#LOAD AND SCORE THE MODEL
savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeClassificationFileLoc)
predictions = savedModel.predict(indexedTESTbinary)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp + ".txt";
dirfilename = scoredResultDir + btclassificationfilename;
predictions.saveAsTextFile(dirfilename)


# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

# LOAD AND SCORE MODEL 
savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeRegressionFileLoc)
predictions = savedModel.predict(indexedTESTreg)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btregressionfilename = "GradientBoostingTreeRegression_" + datestamp + ".txt";
dirfilename = scoredResultDir + btregressionfilename;
predictions.saveAsTextFile(dirfilename)


# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**SORTIE :**

Durée d’exécution de la cellule ci-dessus : 14,6 secondes

## <a name="clean-up-objects-from-memory-and-print-scored-file-locations"></a>Nettoyer des objets de la mémoire et imprimer les emplacements de fichier notés

```python
# UNPERSIST OBJECTS CACHED IN MEMORY
taxi_df_test_cleaned.unpersist()
indexedTESTbinary.unpersist();
oneHotTESTbinary.unpersist();
indexedTESTreg.unpersist();
oneHotTESTreg.unpersist();
oneHotTESTregScaled.unpersist();


# PRINT OUT PATH TO SCORED OUTPUT FILES
print "logisticRegFileLoc: " + logisticregressionfilename;
print "linearRegFileLoc: " + linearregressionfilename;
print "randomForestClassificationFileLoc: " + rfclassificationfilename;
print "randomForestRegFileLoc: " + rfregressionfilename;
print "BoostedTreeClassificationFileLoc: " + btclassificationfilename;
print "BoostedTreeRegressionFileLoc: " + btregressionfilename;
```

**SORTIE :**

logisticRegFileLoc: LogisticRegressionWithLBFGS_2016-05-0317_22_38.953814.txt

linearRegFileLoc: LinearRegressionWithSGD_2016-05-0317_22_58.878949

randomForestClassificationFileLoc: RandomForestClassification_2016-05-0317_23_15.939247.txt

randomForestRegFileLoc: RandomForestRegression_2016-05-0317_23_31.459140.txt

BoostedTreeClassificationFileLoc: GradientBoostingTreeClassification_2016-05-0317_23_49.648334.txt

BoostedTreeRegressionFileLoc: GradientBoostingTreeRegression_2016-05-0317_23_56.860740.txt

## <a name="consume-spark-models-through-a-web-interface"></a>Utiliser les modèles Spark via une interface web
Spark fournit un mécanisme permettant de soumettre à distance des travaux par lots ou des requêtes interactives via une interface REST dotée d’un composant appelé Livy. Par défaut, Livy est activé sur votre cluster HDInsight Spark. Pour plus d'informations sur Livy, consultez la page [Envoi de travaux Spark à distance à l’aide de Livy](../../hdinsight/spark/apache-spark-livy-rest-interface.md). 

Vous pouvez utiliser Livy pour envoyer à distance un travail qui note un fichier stocké dans un objet blob Azure, puis consigne les résultats dans un autre objet blob. Pour ce faire, téléchargez le script Python à partir de  
[Github](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/Spark/Python/ConsumeGBNYCReg.py) dans l’objet blob du cluster Spark. Vous pouvez utiliser un outil tel que l’**Explorateur de stockage Microsoft Azure** ou **AzCopy** pour copier le script dans l’objet blob de cluster. Dans le cas présent, nous avons chargé le script ***wasb:///example/python/ConsumeGBNYCReg.py***.   

> [!NOTE]
> Les clés d’accès dont vous avez besoin se trouvent sur le portail du compte de stockage associé au cluster Spark. 
> 
> 

Une fois chargé à cet emplacement, ce script s’exécute au sein du cluster Spark dans un contexte distribué. Il charge le modèle et exécute les prévisions sur les fichiers d’entrée en fonction du modèle.  

Vous pouvez exécuter ce script à distance en effectuant une simple requête HTTPS/REST sur Livy.  Voici une commande curl permettant de créer la requête HTTP qui appelle le script Python à distance. Remplacez CLUSTERLOGIN, CLUSTERPASSWORD et CLUSTERNAME par les valeurs appropriées pour votre cluster Spark.

```console
# CURL COMMAND TO INVOKE PYTHON SCRIPT WITH HTTP REQUEST

curl -k --user "CLUSTERLOGIN:CLUSTERPASSWORD" -X POST --data "{\"file\": \"wasb:///example/python/ConsumeGBNYCReg.py\"}" -H "Content-Type: application/json" https://CLUSTERNAME.azurehdinsight.net/livy/batches
```

Vous pouvez utiliser n’importe quel langage sur le système distant pour appeler la tâche Spark via Livy, par un simple appel HTTPS avec l’authentification de base.   

> [!NOTE]
> Il vaut mieux utiliser la bibliothèque de requêtes Python pour effectuer cet appel HTTP, mais elle n’est pas installée par défaut dans Azure Functions. C’est pourquoi les anciennes bibliothèques HTTP sont utilisées.   
> 
> 

Voici le code Python pour l’appel HTTP :

```python
#MAKE AN HTTPS CALL ON LIVY. 

import os

# OLDER HTTP LIBRARIES USED HERE INSTEAD OF THE REQUEST LIBRARY AS THEY ARE AVAILABLE BY DEFAULT
import httplib, urllib, base64

# REPLACE VALUE WITH ONES FOR YOUR SPARK CLUSTER
host = '<spark cluster name>.azurehdinsight.net:443'
username='<username>'
password='<password>'

#AUTHORIZATION
conn = httplib.HTTPSConnection(host)
auth = base64.encodestring('%s:%s' % (username, password)).replace('\n', '')
headers = {'Content-Type': 'application/json', 'Authorization': 'Basic %s' % auth}

# SPECIFY THE PYTHON SCRIPT TO RUN ON THE SPARK CLUSTER
# IN THE FILE PARAMETER OF THE JSON POST REQUEST BODY
r=conn.request("POST", '/livy/batches', '{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}', headers )
response = conn.getresponse().read()
print(response)
conn.close()
```

Vous pouvez également ajouter ce code Python dans [Azure Functions](https://azure.microsoft.com/documentation/services/functions/) pour soumettre un travail Spark qui évalue un objet blob en fonction de divers événements, comme un minuteur, une création ou la mise à jour d’un objet blob. 

Si vous préférez vous passer de code, utilisez [Azure Logic Apps](https://azure.microsoft.com/documentation/services/app-service/logic/) pour appeler la notation groupée Spark en définissant une action HTTP dans le **Concepteur d’applications logiques** et en définissant ses paramètres. 

* Sur le portail Azure, créez une application logique en sélectionnant **+Nouveau** -> **Web + Mobile** -> **Application logique**. 
* Pour afficher le **Concepteur d’applications logiques**, entrez le nom de l’application logique et le plan App Service.
* Sélectionnez une action HTTP, puis entrez les paramètres indiqués dans la figure suivante :

![Concepteur Logic Apps](./media/spark-model-consumption/spark-logica-app-client.png)

## <a name="whats-next"></a>Quelle est l’étape suivante ?
**Validation croisée et balayage hyperparamétrique** : consultez la page [Exploration et modélisation avancées des données avec Spark](spark-advanced-data-exploration-modeling.md) pour savoir comment entraîner les modèles à l’aide de la validation croisée et du balayage hyperparamétrique.

