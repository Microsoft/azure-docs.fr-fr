---
title: 'Tutoriel : Créer une application d’apprentissage automatique avec Apache Spark MLlib'
description: Tutoriel sur la façon d’utiliser Spark MLlib pour créer une application d’apprentissage automatique qui analyse un jeu de données en utilisant une classification par régression logistique.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.reviewer: jrasnick
ms.topic: tutorial
ms.subservice: machine-learning
ms.date: 04/15/2020
ms.author: euang
ms.openlocfilehash: 50429696c4cbe10c4723f6d4bb9c9499d9b775c5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450417"
---
# <a name="tutorial-build-a-machine-learning-app-with-apache-spark-mllib-and-azure-synapse-analytics"></a>Tutoriel : Créer une application d’apprentissage automatique avec Apache Spark MLlib et Azure Synapse Analytics

Cet article explique comment utiliser Apache Spark [MLlib](https://spark.apache.org/mllib/) pour créer une application d’apprentissage automatique qui effectue une analyse prédictive simple sur un jeu de données ouvert Azure. Spark fournit des bibliothèques d’apprentissage automatique intégrées. Cet exemple utilise un *classification* par régression logistique.

SparkML et MLlib sont des bibliothèques principales Spark qui fournissent de nombreux utilitaires pour l’exécution de tâches de machine learning. Certains de ces utilitaires conviennent pour les tâches suivantes :

- classification ;
- régression ;
- Clustering
- Modélisation de rubrique
- Décomposition de valeur singulière (SVD) et analyse des composants principaux (PCA)
- Hypothèse de test et de calcul des exemples de statistiques

## <a name="understand-classification-and-logistic-regression"></a>Comprendre la classification et la régression logistique

Une *classification*, tâche de Machine Learning très courante, est le processus de tri de données d’entrée par catégories. Un algorithme de classification doit déterminer comment attribuer des *étiquettes* aux données d’entrée que vous fournissez. Par exemple, on peut imaginer un algorithme apprentissage automatique qui accepte des informations de stock en entrée et divise le stock en deux catégories : ce qu’il faut vendre et ce qu’il faut conserver.

Une *régression logistique* est un algorithme que vous utilisez pour effectuer une classification. L’API de régression logistique de Spark est utile pour la *classification binaire* ou pour classer les données d’entrée dans un des deux groupes. Pour plus d’informations sur la régression logistique, consultez [Wikipedia](https://en.wikipedia.org/wiki/Logistic_regression).

En résumé, le processus de régression logistique génère une *fonction logistique* qui peut être utilisée pour prédire la probabilité qu’un vecteur d’entrée appartienne à un groupe ou à l’autre.

## <a name="predictive-analysis-example-on-nyc-taxi-data"></a>Exemple d’analyse prédictive sur les données des taxis de New York

Dans cet exemple, vous utilisez Spark pour effectuer une analyse prédictive sur les données de pourboires liés aux courses de taxi à New York. Les données sont disponibles dans [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Ce sous-ensemble du jeu de données contient des données sur les courses de taxis jaunes, qui incluent des informations sur chaque course, les heures et lieux de départ et d’arrivé, le prix et d’autres attributs intéressants.

> [!IMPORTANT]
> Des frais supplémentaires peuvent être facturés pour l’extraction de ces données à partir de leur emplacement de stockage.

Dans les étapes suivantes, vous allez développer un modèle pour prédire si un voyage particulier inclut ou non un pourboire.

## <a name="create-an-apache-spark--machine-learning-model"></a>Créer un modèle de machine learning Apache Spark

1. Créez un bloc-notes à l’aide du noyau PySpark. Pour obtenir des instructions, consultez [Créer un bloc-notes](../quickstart-apache-spark-notebook.md#create-a-notebook).
2. Importez les types requis pour cette application. Copiez et collez le code suivant dans une cellule vide, puis appuyez sur **Maj + Entrée** ou exécutez la cellule en utilisant l’icône de lecture bleue à gauche du code.

    ```python
    import matplotlib.pyplot as plt
    from datetime import datetime
    from dateutil import parser
    from pyspark.sql.functions import unix_timestamp, date_format, col, when
    from pyspark.ml import Pipeline
    from pyspark.ml import PipelineModel
    from pyspark.ml.feature import RFormula
    from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorIndexer
    from pyspark.ml.classification import LogisticRegression
    from pyspark.mllib.evaluation import BinaryClassificationMetrics
    from pyspark.ml.evaluation import BinaryClassificationEvaluator
    ```

    Grâce au noyau PySpark, il est inutile de créer des contextes explicitement. Le contexte Spark est créé automatiquement pour vous lorsque vous exécutez la première cellule de code.

## <a name="construct-the-input-dataframe"></a>Construire une trame de données d’entrée

Étant donné que les données brutes sont au format Parquet, vous pouvez utiliser le contexte Spark pour extraire le fichier en mémoire directement en tant que tramedonnées. Le code ci-dessous utilise les options par défaut, mais il est possible de forcer le mappage des types de données et d’autres attributs de schéma si nécessaire.

1. Exécutez les lignes suivantes pour créer une tramedonnées Spark en collant le code dans une nouvelle cellule. Cela récupère les données via l’API Open Datasets. L’extraction de toutes ces données génère environ 1,5 milliard de lignes. Selon la taille de votre pool Apache Spark serverless, les données brutes peuvent être trop volumineuses ou leur exploitation peut prendre trop de temps. Vous pouvez filtrer ces données pour en réduire le volume. L'exemple de code suivant utilise start_date et end_date pour appliquer un filtre qui renvoie un seul mois de données.

    ```python
    from azureml.opendatasets import NycTlcYellow

    end_date = parser.parse('2018-06-06')
    start_date = parser.parse('2018-05-01')
    nyc_tlc = NycTlcYellow(start_date=start_date, end_date=end_date)
    filtered_df = nyc_tlc.to_spark_dataframe()
    ```

2. L’inconvénient du filtrage simple est que, du point de vue statistique, il peut introduire un biais dans les données. Une autre approche consiste à utiliser l’échantillonnage intégré dans Spark. Le code suivant réduit le jeu de données à environ 2000 lignes, s’il est appliqué après le code ci-dessus. Cette étape d’échantillonnage peut être utilisée à la place du filtre simple ou conjointement avec celui-ci.

    ```python
    # To make development easier, faster and less expensive down sample for now
    sampled_taxi_df = filtered_df.sample(True, 0.001, seed=1234)
    ```

3. Il est désormais possible d’examiner les données pour voir ce qui a été lu. Il est généralement préférable d’examiner les données avec un sous-ensemble plutôt qu’avec le jeu complet en fonction de la taille du jeu de données. Le code suivant permet d’afficher les données de deux façons : la première est basique et la deuxième offre une expérience de grille beaucoup plus riche, ainsi que la possibilité de visualiser les données sous forme graphique.

    ```python
    #sampled_taxi_df.show(5)
    display(sampled_taxi_df)
    ```

4. Selon la taille de la taille du jeu de données généré et la nécessité ou non d’expérimenter ou d’exécuter le bloc-notes plusieurs fois, il peut être préférable de mettre en cache le jeu de données localement dans l’espace de travail. Il existe trois façons d'effectuer une mise en cache explicite :

   - enregistrer la tramedonnées localement en tant que fichier ;
   - enregistrer la tramedonnées sous la forme d’une table ou d’un affichage temporaires ;
   - enregistrer la tramedonnées en tant que table permanente.

Les deux premières approches sont incluses dans les exemples de code suivants.

La création d’une table ou d’un affichage temporaires fournit différents chemins d’accès aux données, mais ne dure que le temps de la session d’instance Spark.

```Python
sampled_taxi_df.createOrReplaceTempView("nytaxi")
```

## <a name="prepare-the-data"></a>Préparer les données

Les données dans leur forme brute sont souvent inappropriées pour passer directement à un modèle. Une série d’actions sur les données sont nécessaires afin de les rendre utilisables pour le modèle.

Le code ci-après exécute les quatre classes d’opérations suivantes :

- suppression des valeurs hors normes/incorrectes par filtrage ;
- suppression des colonnes superflues ;
- caractérisation, c’est-à-dire création de nouvelles colonnes dérivées des données brutes pour renforcer l’efficacité de fonctionnement du modèle ;
- étiquetage - car vous effectuez une classification binaire (selon qu'il y a ou non un pourboire associé à une course donnée) nécessitant la conversion du montant du pourboire en valeur 0 ou 1.

```python
taxi_df = sampled_taxi_df.select('totalAmount', 'fareAmount', 'tipAmount', 'paymentType', 'rateCodeId', 'passengerCount'\
                                , 'tripDistance', 'tpepPickupDateTime', 'tpepDropoffDateTime'\
                                , date_format('tpepPickupDateTime', 'hh').alias('pickupHour')\
                                , date_format('tpepPickupDateTime', 'EEEE').alias('weekdayString')\
                                , (unix_timestamp(col('tpepDropoffDateTime')) - unix_timestamp(col('tpepPickupDateTime'))).alias('tripTimeSecs')\
                                , (when(col('tipAmount') > 0, 1).otherwise(0)).alias('tipped')
                                )\
                        .filter((sampled_taxi_df.passengerCount > 0) & (sampled_taxi_df.passengerCount < 8)\
                                & (sampled_taxi_df.tipAmount >= 0) & (sampled_taxi_df.tipAmount <= 25)\
                                & (sampled_taxi_df.fareAmount >= 1) & (sampled_taxi_df.fareAmount <= 250)\
                                & (sampled_taxi_df.tipAmount < sampled_taxi_df.fareAmount)\
                                & (sampled_taxi_df.tripDistance > 0) & (sampled_taxi_df.tripDistance <= 100)\
                                & (sampled_taxi_df.rateCodeId <= 5)
                                & (sampled_taxi_df.paymentType.isin({"1", "2"}))
                                )
```

Une deuxième passe est ensuite effectuée sur les données pour ajouter les caractéristiques finales.

```Python
taxi_featurised_df = taxi_df.select('totalAmount', 'fareAmount', 'tipAmount', 'paymentType', 'passengerCount'\
                                                , 'tripDistance', 'weekdayString', 'pickupHour','tripTimeSecs','tipped'\
                                                , when((taxi_df.pickupHour <= 6) | (taxi_df.pickupHour >= 20),"Night")\
                                                .when((taxi_df.pickupHour >= 7) & (taxi_df.pickupHour <= 10), "AMRush")\
                                                .when((taxi_df.pickupHour >= 11) & (taxi_df.pickupHour <= 15), "Afternoon")\
                                                .when((taxi_df.pickupHour >= 16) & (taxi_df.pickupHour <= 19), "PMRush")\
                                                .otherwise(0).alias('trafficTimeBins')
                                              )\
                                       .filter((taxi_df.tripTimeSecs >= 30) & (taxi_df.tripTimeSecs <= 7200))
```

## <a name="create-a-logistic-regression-model"></a>Créer un modèle de régression logistique

La dernière tâche consiste à convertir les données étiquetées dans un format qui peut être analysé par régression logistique. L’entrée dans un algorithme de régression logistique doit être un jeu de *paires de vecteurs étiquette-caractéristique*, où le *vecteur caractéristique* est un vecteur de nombres représentant le point d’entrée. Nous devons donc convertir les colonnes catégoriques en nombres. Les colonnes `trafficTimeBins` et `weekdayString` doivent être converties en représentations sous forme d’entiers. Il existe plusieurs approches de la conversion. Celle adoptée dans cet exemple est l’approche courante *OneHotEncoding*.

```python
# Since the sample uses an algorithm that only works with numeric features, convert them so they can be consumed
sI1 = StringIndexer(inputCol="trafficTimeBins", outputCol="trafficTimeBinsIndex")
en1 = OneHotEncoder(dropLast=False, inputCol="trafficTimeBinsIndex", outputCol="trafficTimeBinsVec")
sI2 = StringIndexer(inputCol="weekdayString", outputCol="weekdayIndex")
en2 = OneHotEncoder(dropLast=False, inputCol="weekdayIndex", outputCol="weekdayVec")

# Create a new dataframe that has had the encodings applied
encoded_final_df = Pipeline(stages=[sI1, en1, sI2, en2]).fit(taxi_featurised_df).transform(taxi_featurised_df)
```

Cette action se traduit par une nouvelle tramedonnées avec toutes les colonnes au format approprié pour effectuer l’apprentissage d’un modèle.

## <a name="train-a-logistic-regression-model"></a>Effectuer l’apprentissage d’un modèle de régression logistique

La première tâche consiste à fractionner le jeu de données en un jeu d’apprentissage et un jeu de test ou de validation. Le fractionnement ici est arbitraire et nous vous recommander d’expérimenter différents paramètres de fractionnement pour voir s’ils ont une incidence sur le modèle.

```python
#Decide on the split between training and testing data from the dataframe
trainingFraction = 0.7
testingFraction = (1-trainingFraction)
seed = 1234

# Split the dataframe into test and training dataframes
train_data_df, test_data_df = encoded_final_df.randomSplit([trainingFraction, testingFraction], seed=seed)
```

Maintenant qu’il y a deux tramedonnées, la tâche suivante consiste à créer la formule modèle et à l’exécuter sur la tramedonnées d’apprentissage, puis à valider par rapport à la tramedonnées de test. Vous devez expérimenter différentes versions de la formule modèle pour voir l’impact de différentes combinaisons.

> [!Note]
> Pour enregistrer le modèle, vous aurez besoin du rôle Azure « Contributeur aux données blob du stockage ». Sous votre compte de stockage, accédez à Access Control (IAM), puis sélectionnez **Ajouter une attribution de rôle**. Attribuez le rôle Azure « Contributeur aux données blob du stockage » à votre serveur SQL Database. Seuls les membres dotés du privilège Propriétaire peuvent effectuer cette étape. Pour découvrir les divers rôles intégrés Azure, consultez ce [guide](../../role-based-access-control/built-in-roles.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

```python
## Create a new LR object for the model
logReg = LogisticRegression(maxIter=10, regParam=0.3, labelCol = 'tipped')

## The formula for the model
classFormula = RFormula(formula="tipped ~ pickupHour + weekdayVec + passengerCount + tripTimeSecs + tripDistance + fareAmount + paymentType+ trafficTimeBinsVec")

## Undertake training and create an LR model
lrModel = Pipeline(stages=[classFormula, logReg]).fit(train_data_df)

## Saving the model is optional but its another form of inter session cache
datestamp = datetime.now().strftime('%m-%d-%Y-%s')
fileName = "lrModel_" + datestamp
logRegDirfilename = fileName
lrModel.save(logRegDirfilename)

## Predict tip 1/0 (yes/no) on the test dataset, evaluation using AUROC
predictions = lrModel.transform(test_data_df)
predictionAndLabels = predictions.select("label","prediction").rdd
metrics = BinaryClassificationMetrics(predictionAndLabels)
print("Area under ROC = %s" % metrics.areaUnderROC)
```

La sortie de cette cellule est :

```shell
Area under ROC = 0.9779470729751403
```

## <a name="create-a-visual-representation-of-the-prediction"></a>Créer une représentation visuelle de la prédiction

Vous pouvez désormais construire une visualisation finale pour faciliter l’examen des résultats de ce test. Une [courbe ROC](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) est un moyen d’examiner le résultat.

```python
## Plot the ROC curve, no need for pandas as this uses the modelSummary object
modelSummary = lrModel.stages[-1].summary

plt.plot([0, 1], [0, 1], 'r--')
plt.plot(modelSummary.roc.select('FPR').collect(),
         modelSummary.roc.select('TPR').collect())
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.show()
```

![Courbe ROC pour le modèle de régression logistique pourboires](./media/apache-spark-machine-learning-mllib-notebook/nyc-taxi-roc.png)

## <a name="shut-down-the-spark-instance"></a>Arrêter l’instance Spark

Une fois que vous avez exécuté l’application, arrêtez le bloc-notes pour libérer les ressources en fermant l’onglet, ou sélectionnez **Terminer la session** dans le panneau d’État au bas du bloc-notes.

## <a name="see-also"></a>Voir aussi

- [Vue d’ensemble : Apache Spark sur Azure Synapse Analytics](apache-spark-overview.md)

## <a name="next-steps"></a>Étapes suivantes

- [Documentation .NET pour Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
- [Documentation officielle Apache Spark](https://spark.apache.org/docs/2.4.5/)

>[!NOTE]
> Une partie de la documentation Apache Spark officielle repose sur l’utilisation de la console Spark qui n’est pas disponible sur Azure Synapse Spark. Utilisez à la place les expériences [notebook](../quickstart-apache-spark-notebook.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) ou [IntelliJ](../spark/intellij-tool-synapse.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
