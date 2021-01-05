---
title: Créer un pipeline d’apprentissage automatique Apache Spark - Azure HDInsight
description: Utilisez la bibliothèque de Machine Learning Apache Spark pour créer des pipelines de données dans Azure HDInsight.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 07/22/2019
ms.openlocfilehash: c270e9865aff30184ea236f56ab20ede78c5d577
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86075448"
---
# <a name="create-an-apache-spark-machine-learning-pipeline"></a>Créer un pipeline d’apprentissage automatique Apache Spark

La bibliothèque scalable d’apprentissage automatique (MLlib) Apache Spark apporte des fonctionnalités de modélisation à un environnement distribué. Le package Spark [`spark.ml`](https://spark.apache.org/docs/latest/ml-pipeline.html) est un ensemble d’API générales reposant sur des DataFrames. Ces API vous permettent de créer et de régler des pipelines d’apprentissage automatique pratiques.  *L’apprentissage automatique Spark* fait référence à cette API basée sur les DataFrames MLlib, et non à l’API de pipeline plus ancienne basée sur les RDD.

Un pipeline d’apprentissage automatique est un flux de travail complet combinant plusieurs algorithmes d’apprentissage automatique. Il peut y avoir de nombreuses étapes nécessaires pour traiter et apprendre à partir des données, nécessitant une séquence d’algorithmes. Les pipelines définissent les phases et l’ordre d’un processus d’apprentissage automatique. Dans MLlib, les phases d’un pipeline sont représentées par une séquence spécifique de PipelineStages, où un Transformer et un Estimator effectuent chacun des tâches.

Un Transformer est un algorithme qui transforme un DataFrame en un autre à l’aide de la méthode `transform()`. Par exemple, un Transformer de fonctionnalité peut lire une colonne d’un DataFrame, la mapper à une autre colonne et générer un nouveau DataFrame avec la colonne mappée ajoutée.

Un Estimator est une abstraction d’algorithmes d’apprentissage chargée de l’ajustement ou de la formation sur un jeu de données afin de produire un Transformer. Un Estimator implémente une méthode nommée `fit()`, qui accepte un DataFrame et génère un DataFrame, qui est un Transformer.

Chaque instance sans état d’un Transformer ou d’un Estimator a son propre identificateur unique, qui est utilisé lors de la spécification des paramètres. Tous deux utilisent une API uniforme pour spécifier ces paramètres.

## <a name="pipeline-example"></a>Exemple de pipeline

Pour illustrer une utilisation pratique d’un pipeline d’apprentissage automatique, cet exemple utilise l’exemple de fichier de données `HVAC.csv` qui est préchargé sur le stockage par défaut pour votre cluster HDInsight, c’est-à-dire le stockage Azure ou Data Lake Storage. Pour afficher le contenu du fichier, accédez au répertoire `/HdiSamples/HdiSamples/SensorSampleData/hvac`. `HVAC.csv` contient un ensemble d’heures avec des températures cibles et réelles pour des systèmes CVC (*chauffage, ventilation et climatisation*) dans différents bâtiments. L’objectif est de former le modèle sur les données et de produire une prédiction de température pour un bâtiment donné.

Le code suivant :

1. Définit un `LabeledDocument`, qui stocke le `BuildingID`, `SystemInfo` (l’âge et l’identificateur d’un système) et un `label` (1.0 si le bâtiment est trop chaud, 0.0 dans le cas contraire).
2. Crée une fonction d’analyseur personnalisée `parseDocument` qui prend une ligne de données et détermine si le bâtiment est « chaud » en comparant la température réelle à la température cible.
3. Applique l’analyseur lors de l’extraction des données sources.
4. Crée les données d’apprentissage.

```python
from pyspark.ml import Pipeline
from pyspark.ml.classification import LogisticRegression
from pyspark.ml.feature import HashingTF, Tokenizer
from pyspark.sql import Row

# The data structure (column meanings) of the data array:
# 0 Date
# 1 Time
# 2 TargetTemp
# 3 ActualTemp
# 4 System
# 5 SystemAge
# 6 BuildingID

LabeledDocument = Row("BuildingID", "SystemInfo", "label")

# Define a function that parses the raw CSV file and returns an object of type LabeledDocument


def parseDocument(line):
    values = [str(x) for x in line.split(',')]
    if (values[3] > values[2]):
        hot = 1.0
    else:
        hot = 0.0

    textValue = str(values[4]) + " " + str(values[5])

    return LabeledDocument((values[6]), textValue, hot)


# Load the raw HVAC.csv file, parse it using the function
data = sc.textFile(
    "wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

documents = data.filter(lambda s: "Date" not in s).map(parseDocument)
training = documents.toDF()
```

Cet exemple de pipeline comporte trois phases : `Tokenizer` et `HashingTF` (deux Transformers) et `Logistic Regression` (un Estimator).  Les données analysées et extraites dans le DataFrame `training` transitent par le pipeline quand `pipeline.fit(training)` est appelé.

1. La première phase, `Tokenizer`, fractionne la colonne d’entrée `SystemInfo` (comprenant les valeurs d’identificateur et d’âge du système) dans une colonne de sortie `words`. Cette nouvelle colonne `words` est ajoutée au DataFrame. 
2. La deuxième phase, `HashingTF`, convertit la nouvelle colonne `words` en vecteurs de caractéristiques. Cette nouvelle colonne `features` est ajoutée au DataFrame. Ces deux premières phases sont des Transformers. 
3. Comme la troisième phase, `LogisticRegression`, est un Estimator, le pipeline appelle la méthode `LogisticRegression.fit()` pour produire un `LogisticRegressionModel`. 

```python
tokenizer = Tokenizer(inputCol="SystemInfo", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
lr = LogisticRegression(maxIter=10, regParam=0.01)

# Build the pipeline with our tokenizer, hashingTF, and logistic regression stages
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])

model = pipeline.fit(training)
```

Pour afficher les nouvelles colonnes `words` et `features` ajoutées par les Transformers `Tokenizer` et `HashingTF`, et un exemple de l’Estimator `LogisticRegression`, exécutez une méthode `PipelineModel.transform()` sur le DataFrame d’origine. Dans le code de production, l’étape suivante consisterait à passer un DataFrame de test pour valider la formation.

```python
peek = model.transform(training)
peek.show()

# Outputs the following:
+----------+----------+-----+--------+--------------------+--------------------+--------------------+----------+
|BuildingID|SystemInfo|label|   words|            features|       rawPrediction|         probability|prediction|
+----------+----------+-----+--------+--------------------+--------------------+--------------------+----------+
|         4|     13 20|  0.0|[13, 20]|(262144,[250802,2...|[0.11943986671420...|[0.52982451901740...|       0.0|
|        17|      3 20|  0.0| [3, 20]|(262144,[89074,25...|[0.17511205617446...|[0.54366648775222...|       0.0|
|        18|     17 20|  1.0|[17, 20]|(262144,[64358,25...|[0.14620993833623...|[0.53648750722548...|       0.0|
|        15|      2 23|  0.0| [2, 23]|(262144,[31351,21...|[-0.0361327091023...|[0.49096780538523...|       1.0|
|         3|      16 9|  1.0| [16, 9]|(262144,[153779,1...|[-0.0853679939336...|[0.47867095324139...|       1.0|
|         4|     13 28|  0.0|[13, 28]|(262144,[69821,25...|[0.14630166986618...|[0.53651031790592...|       0.0|
|         2|     12 24|  0.0|[12, 24]|(262144,[187043,2...|[-0.0509556393066...|[0.48726384581522...|       1.0|
|        16|     20 26|  1.0|[20, 26]|(262144,[128319,2...|[0.33829638728900...|[0.58377663577684...|       0.0|
|         9|      16 9|  1.0| [16, 9]|(262144,[153779,1...|[-0.0853679939336...|[0.47867095324139...|       1.0|
|        12|       6 5|  0.0|  [6, 5]|(262144,[18659,89...|[0.07513008136562...|[0.51877369045183...|       0.0|
|        15|     10 17|  1.0|[10, 17]|(262144,[64358,25...|[-0.0291988646553...|[0.49270080242078...|       1.0|
|         7|      2 11|  0.0| [2, 11]|(262144,[212053,2...|[0.03678030020834...|[0.50919403860812...|       0.0|
|        15|      14 2|  1.0| [14, 2]|(262144,[109681,2...|[0.06216423725633...|[0.51553605651806...|       0.0|
|         6|       3 2|  0.0|  [3, 2]|(262144,[89074,21...|[0.00565582077537...|[0.50141395142468...|       0.0|
|        20|     19 22|  0.0|[19, 22]|(262144,[139093,2...|[-0.0769288695989...|[0.48077726176073...|       1.0|
|         8|     19 11|  0.0|[19, 11]|(262144,[139093,2...|[0.04988910033929...|[0.51246968885151...|       0.0|
|         6|      15 7|  0.0| [15, 7]|(262144,[77099,20...|[0.14854929135994...|[0.53706918109610...|       0.0|
|        13|      12 5|  0.0| [12, 5]|(262144,[89689,25...|[-0.0519932532562...|[0.48700461408785...|       1.0|
|         4|      8 22|  0.0| [8, 22]|(262144,[98962,21...|[-0.0120753606650...|[0.49698119651572...|       1.0|
|         7|      17 5|  0.0| [17, 5]|(262144,[64358,89...|[-0.0721054054871...|[0.48198145477106...|       1.0|
+----------+----------+-----+--------+--------------------+--------------------+--------------------+----------+

only showing top 20 rows
```

L’objet `model` peut maintenant être utilisé pour effectuer des prédictions. Pour obtenir l’exemple complet de cette application d’apprentissage automatique et des instructions expliquant pas à pas comment l’exécuter, consultez [Créer des applications de Machine Learning Apache Spark sur Azure HDInsight](apache-spark-ipython-notebook-machine-learning.md).

## <a name="see-also"></a>Voir aussi

* [Science des données avec Scala et Apache Spark sur Azure](../../machine-learning/team-data-science-process/scala-walkthrough.md)
