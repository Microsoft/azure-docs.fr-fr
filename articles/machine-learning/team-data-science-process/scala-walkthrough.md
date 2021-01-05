---
title: Science des données à l’aide de Scala et de Spark sur Azure - Team Data Science Process
description: Comment utiliser Scala pour les tâches d’apprentissage automatique supervisées avec des packages MLlib et Spark ML sur un cluster Azure HDInsight Spark.
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
ms.openlocfilehash: 9ae4549fe343422bbf60275a97768ca407f2dc7c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321369"
---
# <a name="data-science-using-scala-and-spark-on-azure"></a>Science des données à l’aide de Scala et Spark sur Azure
Ce article vous montre comment utiliser Scala pour les tâches d’apprentissage automatique supervisées avec la bibliothèque d’apprentissage automatique évolutif (MLlib) Spark et des packages SparkML sur un cluster Azure HDInsight Spark. Elle vous guide à travers les tâches qui constituent le [processus de science des données](./index.yml): ingestion et exploration des données, visualisation, conception de fonctionnalités et consommation de modèles. Les modèles de cet article incluent la régression logistique et linéaire, les forêts aléatoires et les arbres GBT (Gradient Boosted Tree), en plus de deux tâches d’apprentissage automatique supervisées courantes :

* Problème de régression : prédiction du montant du pourboire ($) pour une course de taxi
* Classification binaire : prédiction de la réception d’un pourboire ou non (1/0) pour une course en taxi

Le processus de modélisation nécessite une formation et une évaluation sur des jeux de données de test avec des mesures de précision pertinentes. Cet article explique également comment stocker ces modèles dans le stockage d’objets blob Azure et comment noter et évaluer leurs performances de prédiction. Il couvre également les rubriques plus avancées liées à l’optimisation des modèles à l’aide de la validation croisée et du balayage hyperparamétrique. Les données utilisées représentent un échantillon du jeu de données NYC Taxi Trip and Fare 2013 disponible sur GitHub.

[Scala](https://www.scala-lang.org/), un langage basé sur la machine virtuelle Java, intègre des concepts du langage fonctionnel et orienté objet. C’est un langage évolutif qui convient pour le traitement distribué dans le cloud et s’exécute sur des clusters Azure Spark.

[Spark](https://spark.apache.org/) est une infrastructure de traitement en parallèle open source qui prend en charge le traitement en mémoire pour accroître les performances des applications d’analytique de Big Data. Le moteur de traitement Spark est élaboré pour permettre des analyses rapides, simples d’utilisation et sophistiquées. De par ses capacités de calcul distribué en mémoire, Spark constitue le choix idéal pour les algorithmes itératifs utilisés dans l'apprentissage automatique et les calculs de graphiques. Le package [spark.ml](https://spark.apache.org/docs/latest/ml-guide.html) fournit un ensemble d’API de haut niveau basées sur des trames de données qui vous permettent de créer et d’ajuster des pipelines d’apprentissage automatique pratique. [MLlib](https://spark.apache.org/mllib/) est la bibliothèque évolutive d’apprentissage automatique de Spark. Elle apporte des fonctionnalités de modélisation à cet environnement distribué.

[HDInsight Spark](../../hdinsight/spark/apache-spark-overview.md) est l’offre Azure de Spark Open Source. Il prend également en charge les notebooks Jupyter Scala sur le cluster Spark, qui peuvent exécuter des requêtes interactives SQL Spark pour transformer, filtrer et visualiser les données stockées dans les objets blob Azure. Les extraits de code Scala de cet article qui fournissent les solutions et montrent les tracés pertinents permettant de visualiser les données s’exécutent dans des notebooks Jupyter installés sur les clusters Spark. Les étapes de modélisation dans ces rubriques contiennent du code qui vous montre comment former, évaluer, enregistrer et consommer chaque type de modèle.

Les étapes d’installation et le code présentés dans cet article s’appliquent à Azure HDInsight 3.4 Spark 1.6. Toutefois, le code affiché dans cet article et dans les [notebooks Jupyter Scala](https://github.com/Azure-Samples/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Scala/Exploration-Modeling-and-Scoring-using-Scala.ipynb) est générique et devrait fonctionner sur n’importe quel cluster Spark. Les étapes de configuration et de gestion de cluster peuvent être légèrement différentes de celles indiquées dans cet article, si vous n’utilisez pas HDInsight Spark.

> [!NOTE]
> Pour une rubrique qui vous montre comment utiliser Python plutôt que Scala pour effectuer des tâches de processus de science des données de bout en bout, consultez [Science des données avec Spark sur Azure HDInsight](spark-overview.md).
> 
> 

## <a name="prerequisites"></a>Prérequis
* Vous devez avoir un abonnement Azure. Si vous n’en avez pas, [obtenez une version d’évaluation gratuite Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).
* Vous avez besoin d’un cluster Azure HDInsight 3.4 Spark 1.6 pour effectuer les procédures suivantes. Pour créer un cluster, consultez les instructions de l’article [Prise en main : Créer Apache Spark sur Azure HDInsight](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md). Spécifiez le type et la version du cluster à partir du menu **Sélectionner le type de cluster** .

![Configuration de type cluster HDInsight](./media/scala-walkthrough/spark-cluster-on-portal.png)

> [!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]
> 
> 

Pour obtenir une description des données NYC Taxi Trip Taxi et pour savoir comment exécuter du code à partir d’un notebook Jupyter sur le cluster Spark, consultez les sections pertinentes de [Vue d’ensemble de la science des données utilisant Spark sur Azure HDInsight](spark-overview.md).  

## <a name="execute-scala-code-from-a-jupyter-notebook-on-the-spark-cluster"></a>Exécuter le code Scala à partir d’un notebook Jupyter sur le cluster Spark
Vous pouvez lancer un notebook Jupyter à partir du portail Azure. Recherchez le cluster Spark sur votre tableau de bord, puis cliquez dessus pour accéder à la page de gestion de votre cluster. Cliquez ensuite sur **Tableaux de bord du cluster** puis sur **Notebook Jupyter** pour ouvrir le notebook associé au cluster Spark.

![Tableau de bord de cluster et notebooks Jupyter](./media/scala-walkthrough/spark-jupyter-on-portal.png)

Vous pouvez également accéder aux notebooks Jupyter à l’adresse https://&lt;clustername&gt;.azurehdinsight.net/jupyter. Remplacez *clustername* par le nom de votre cluster. Vous avez besoin du mot de passe de votre compte d’administrateur pour accéder aux notebooks Jupyter.

![Accéder à des notebooks Jupyter en utilisant le nom du cluster](./media/scala-walkthrough/spark-jupyter-notebook.png)

Sélectionnez **Scala** pour afficher un répertoire contenant quelques exemples de notebooks préconfigurés qui utilisent l’API PySpark. Le notebook Modélisation d’exploration et notation avec Scala.ipynb contenant les exemples de code pour cet ensemble de rubriques Spark est disponible sur [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/Scala).

Vous pouvez télécharger le notebook directement de GitHub sur le serveur de notebooks Jupyter de votre cluster Spark. Sur votre page d’accueil Jupyter, cliquez sur le bouton **Télécharger** . Dans l’Explorateur de fichiers, collez l’URL GitHub (contenu brut) du notebook Scala, puis cliquez sur **Ouvrir**. Le notebook Scala est disponible à l’URL suivante :

[Exploration-Modeling-and-Scoring-using-Scala.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Scala/Exploration-Modeling-and-Scoring-using-Scala.ipynb)

## <a name="setup-preset-spark-and-hive-contexts-spark-magics-and-spark-libraries"></a>Paramétrage : contextes Spark et Hive prédéfinis, commandes magiques Spark et bibliothèques Spark
### <a name="preset-spark-and-hive-contexts"></a>Contextes Spark et Hive prédéfinis

```scala
# SET THE START TIME
import java.util.Calendar
val beginningTime = Calendar.getInstance().getTime()
```

Les noyaux Spark fournis avec les notebooks Jupyter contiennent des contextes prédéfinis. Vous n’avez pas besoin de définir explicitement les contextes Spark ou Hive avant de commencer à travailler avec l'application que vous développez. Les contextes prédéfinis sont les suivants :

* `sc` pour SparkContext
* `sqlContext` pour HiveContext

### <a name="spark-magics"></a>Commandes magiques de Spark
Le noyau Spark fournit certaines « commandes magiques » prédéfinies, qui sont des commandes spéciales que vous pouvez appeler avec `%%`. Deux de ces commandes sont utilisées dans les exemples de code suivants.

* `%%local` indique que le code des lignes suivantes est exécuté localement. Le code doit être un code Scala valide.
* `%%sql -o <variable name>` exécute une requête Hive sur `sqlContext`. Si le paramètre `-o` est passé, le résultat de la requête est conservé dans le contexte Scala `%%local` en tant que tableau de données Spark.

Pour plus d’informations sur les noyaux pour notebooks Jupyter et sur leurs « commandes magiques » prédéfinies appelées avec `%%` (par exemple, `%%local`), consultez [Noyaux disponibles pour les blocs-notes Jupyter avec les clusters HDInsight Spark Linux sur HDInsight](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

### <a name="import-libraries"></a>Importer les bibliothèques
Importez les bibliothèques Spark, MLlib et autres dont vous aurez besoin à l’aide du code suivant.

```scala
# IMPORT SPARK AND JAVA LIBRARIES
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.functions._
import java.text.SimpleDateFormat
import java.util.Calendar
import sqlContext.implicits._
import org.apache.spark.sql.Row

# IMPORT SPARK SQL FUNCTIONS
import org.apache.spark.sql.types.{StructType, StructField, StringType, IntegerType, FloatType, DoubleType}
import org.apache.spark.sql.functions.rand

# IMPORT SPARK ML FUNCTIONS
import org.apache.spark.ml.Pipeline
import org.apache.spark.ml.feature.{StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer, Binarizer}
import org.apache.spark.ml.tuning.{ParamGridBuilder, TrainValidationSplit, CrossValidator}
import org.apache.spark.ml.regression.{LinearRegression, LinearRegressionModel, RandomForestRegressor, RandomForestRegressionModel, GBTRegressor, GBTRegressionModel}
import org.apache.spark.ml.classification.{LogisticRegression, LogisticRegressionModel, RandomForestClassifier, RandomForestClassificationModel, GBTClassifier, GBTClassificationModel}
import org.apache.spark.ml.evaluation.{BinaryClassificationEvaluator, RegressionEvaluator, MulticlassClassificationEvaluator}

# IMPORT SPARK MLLIB FUNCTIONS
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.classification.{LogisticRegressionWithLBFGS, LogisticRegressionModel}
import org.apache.spark.mllib.regression.{LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel}
import org.apache.spark.mllib.tree.{GradientBoostedTrees, RandomForest}
import org.apache.spark.mllib.tree.configuration.BoostingStrategy
import org.apache.spark.mllib.tree.model.{GradientBoostedTreesModel, RandomForestModel, Predict}
import org.apache.spark.mllib.evaluation.{BinaryClassificationMetrics, MulticlassMetrics, RegressionMetrics}

# SPECIFY SQLCONTEXT
val sqlContext = new SQLContext(sc)
```

## <a name="data-ingestion"></a>Ingestion de données
La première étape du processus de science des données consiste à ingérer les données à analyser. Vous récupérez les données à partir de sources externes ou de systèmes dans lesquels elles résident afin de les injecter dans votre environnement de modélisation et d’exploration de données. Dans cet article, les données ingérées représentent un échantillon joint de 0,1 % des trajets et frais de taxi (stocké sous forme de fichier .tsv). L’environnement de modélisation et d’exploration de données est Spark. Cette section contient le code permettant d’effectuer la série de tâches suivante :

1. Définir les chemins d’accès pour le stockage des données et du modèle.
2. Lire le jeu de données en entrée (stocké dans un fichier .tsv).
3. Définir un schéma pour les données et nettoyer les données.
4. Créer une trame de données propre, puis la mettre en mémoire cache.
5. Enregistrer les données sous forme de table temporaire dans SQLContext.
6. Interroger la table et importer les résultats dans une trame de données.

### <a name="set-directory-paths-for-storage-locations-in-azure-blob-storage"></a>Définir les chemins d’accès aux emplacements de stockage dans le stockage d’objets blob Azure.
Spark peut lire et écrire vers le stockage d’objets blob Azure. Vous pouvez utiliser Spark pour traiter n’importe quelles données existantes puis stocker à nouveau les résultats dans le stockage d’objets blob.

Pour enregistrer des modèles ou des fichiers dans le stockage d’objets blob, vous devez spécifier correctement le chemin d’accès. Référencez le conteneur par défaut associé au cluster Spark à l’aide d’un chemin commençant par `wasb:///`. Référencez d’autres emplacements en utilisant `wasb://`.

L’exemple de code suivant spécifie l’emplacement des données d’entrée pour la lecture et le chemin d’accès à un stockage d’objets blob qui est associé au cluster Spark dans lequel le modèle sera enregistré.

```scala
# SET PATHS TO DATA AND MODEL FILE LOCATIONS
# INGEST DATA AND SPECIFY HEADERS FOR COLUMNS
val taxi_train_file = sc.textFile("wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv")
val header = taxi_train_file.first;

# SET THE MODEL STORAGE DIRECTORY PATH
# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS REQUIRED.
val modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/";
```

### <a name="import-data-create-an-rdd-and-define-a-data-frame-according-to-the-schema"></a>Importer des données, créer un objet RDD et définir une trame de données en fonction du schéma

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# DEFINE THE SCHEMA BASED ON THE HEADER OF THE FILE
val sqlContext = new SQLContext(sc)
val taxi_schema = StructType(
    Array(
        StructField("medallion", StringType, true),
        StructField("hack_license", StringType, true),
        StructField("vendor_id", StringType, true),
        StructField("rate_code", DoubleType, true),
        StructField("store_and_fwd_flag", StringType, true),
        StructField("pickup_datetime", StringType, true),
        StructField("dropoff_datetime", StringType, true),
        StructField("pickup_hour", DoubleType, true),
        StructField("pickup_week", DoubleType, true),
        StructField("weekday", DoubleType, true),
        StructField("passenger_count", DoubleType, true),
        StructField("trip_time_in_secs", DoubleType, true),
        StructField("trip_distance", DoubleType, true),
        StructField("pickup_longitude", DoubleType, true),
        StructField("pickup_latitude", DoubleType, true),
        StructField("dropoff_longitude", DoubleType, true),
        StructField("dropoff_latitude", DoubleType, true),
        StructField("direct_distance", StringType, true),
        StructField("payment_type", StringType, true),
        StructField("fare_amount", DoubleType, true),
        StructField("surcharge", DoubleType, true),
        StructField("mta_tax", DoubleType, true),
        StructField("tip_amount", DoubleType, true),
        StructField("tolls_amount", DoubleType, true),
        StructField("total_amount", DoubleType, true),
        StructField("tipped", DoubleType, true),
        StructField("tip_class", DoubleType, true)
        )
    )

# CAST VARIABLES ACCORDING TO THE SCHEMA
val taxi_temp = (taxi_train_file.map(_.split("\t"))
                        .filter((r) => r(0) != "medallion")
                         .map(p => Row(p(0), p(1), p(2),
                            p(3).toDouble, p(4), p(5), p(6), p(7).toDouble, p(8).toDouble, p(9).toDouble, p(10).toDouble,
                            p(11).toDouble, p(12).toDouble, p(13).toDouble, p(14).toDouble, p(15).toDouble, p(16).toDouble,
                            p(17), p(18), p(19).toDouble, p(20).toDouble, p(21).toDouble, p(22).toDouble,
                            p(23).toDouble, p(24).toDouble, p(25).toDouble, p(26).toDouble)))


# CREATE AN INITIAL DATA FRAME AND DROP COLUMNS, AND THEN CREATE A CLEANED DATA FRAME BY FILTERING FOR UNWANTED VALUES OR OUTLIERS
val taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)

val taxi_df_train_cleaned = (taxi_train_df.drop(taxi_train_df.col("medallion"))
        .drop(taxi_train_df.col("hack_license")).drop(taxi_train_df.col("store_and_fwd_flag"))
        .drop(taxi_train_df.col("pickup_datetime")).drop(taxi_train_df.col("dropoff_datetime"))
        .drop(taxi_train_df.col("pickup_longitude")).drop(taxi_train_df.col("pickup_latitude"))
        .drop(taxi_train_df.col("dropoff_longitude")).drop(taxi_train_df.col("dropoff_latitude"))
        .drop(taxi_train_df.col("surcharge")).drop(taxi_train_df.col("mta_tax"))
        .drop(taxi_train_df.col("direct_distance")).drop(taxi_train_df.col("tolls_amount"))
        .drop(taxi_train_df.col("total_amount")).drop(taxi_train_df.col("tip_class"))
        .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200"));

# CACHE AND MATERIALIZE THE CLEANED DATA FRAME IN MEMORY
taxi_df_train_cleaned.cache()
taxi_df_train_cleaned.count()

# REGISTER THE DATA FRAME AS A TEMPORARY TABLE IN SQLCONTEXT
taxi_df_train_cleaned.registerTempTable("taxi_train")

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Output:**

Durée d’exécution de la cellule : 8 secondes.

### <a name="query-the-table-and-import-results-in-a-data-frame"></a>Interrogez la table et importez les résultats dans une trame de données.
Interrogez ensuite la table pour les données de prix, de passagers et de pourboires, filtrez les données endommagées et éloignées, et imprimez plusieurs lignes.

```scala
# QUERY THE DATA
val sqlStatement = """
    SELECT fare_amount, passenger_count, tip_amount, tipped
    FROM taxi_train
    WHERE passenger_count > 0 AND passenger_count < 7
    AND fare_amount > 0 AND fare_amount < 200
    AND payment_type in ('CSH', 'CRD')
    AND tip_amount > 0 AND tip_amount < 25
"""
val sqlResultsDF = sqlContext.sql(sqlStatement)

# SHOW ONLY THE TOP THREE ROWS
sqlResultsDF.show(3)
```

**Output:**

| fare_amount | passenger_count | tip_amount | tipped |
| --- | --- | --- | --- |
|        13,5 |1.0 |2,9 |1.0 |
|        16,0 |2 |3.4 |1.0 |
|        10.5 |2 |1.0 |1.0 |

## <a name="data-exploration-and-visualization"></a>Exploration et visualisation de données
Une fois les données intégrées dans Spark, l’étape suivante du processus de science des données consiste à mieux comprendre les données par l’exploration et la visualisation. Dans cette section, nous examinons les données du taxi à l’aide de requêtes SQL. Ensuite, nous importons les résultats dans une trame de données pour tracer les variables cibles et les fonctionnalités potentielles pour l’examen visuel à l’aide de la fonctionnalité Jupyter de visualisation automatique.

### <a name="use-local-and-sql-magic-to-plot-data"></a>Utiliser les fonctions magiques locales et SQL pour tracer les données
Par défaut, la sortie de tout extrait de code que vous exécutez à partir d’un notebook Jupyter est disponible dans le contexte de la session qui est persistante sur les nœuds de travail. Si vous souhaitez enregistrer un trajet sur les nœuds de travail pour chaque calcul et si toutes les données dont vous avez besoin pour votre calcul sont disponibles localement sur le nœud du serveur Jupyter (qui est le nœud principal), vous pouvez utiliser la fonction magique `%%local` pour exécuter l’extrait de code sur le serveur Jupyter.

* **Commande magique SQL** (`%%sql`). Le noyau HDInsight Spark prend en charge les requêtes HiveQL inline faciles exécutées sur SQLContext. L’argument (`-o VARIABLE_NAME`) conserve la sortie de la requête SQL en tant que tableau de données Pandas sur le serveur Jupyter. Ce paramètre signifie que la sortie sera disponible en mode local.
* **Commande magique** `%%local`. La commande magique `%%local` exécute le code localement sur le serveur Jupyter, qui est le nœud principal du cluster HDInsight. En général, vous utilisez la commande magique `%%local` conjointement avec la commande magique `%%sql` et le paramètre `-o`. Le paramètre `-o` conserve la sortie de la requête SQL localement, puis la commande magique `%%local` déclenche l’ensemble suivant d’extrait de code pour une exécution locale sur la sortie des requêtes SQL conservées localement.

### <a name="query-the-data-by-using-sql"></a>Interroger les données en utilisant SQL
Cette requête retrouve les courses de taxi par montant du trajet, nombre de passagers et montant du pourboire.

```scala
# RUN THE SQL QUERY
%%sql -q -o sqlResults
SELECT fare_amount, passenger_count, tip_amount, tipped FROM taxi_train WHERE passenger_count > 0 AND passenger_count < 7 AND fare_amount > 0 AND fare_amount < 200 AND payment_type in ('CSH', 'CRD') AND tip_amount > 0 AND tip_amount < 25
```

Dans le code suivant, la commande magique `%%local` crée une trame de données locale, sqlResults. Vous pouvez utiliser sqlResults pour tracer les données à l’aide de matplotlib.

> [!TIP]
> Cette commande magique locale est utilisée plusieurs fois dans cet article. Si votre jeu de données est important, échantillonnez pour créer un tableau de données adapté à la mémoire locale.
> 
> 

### <a name="plot-the-data"></a>Tracer les données
Vous pouvez tracer les données à l’aide du code Python lorsque la trame de données se situe dans le contexte local en tant que trame de données Pandas.

```scala
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES.
# CLICK THE TYPE OF PLOT TO GENERATE (LINE, AREA, BAR, ETC.)
sqlResults
```

 Le noyau Spark visualise automatiquement la sortie des requêtes SQL (HiveQL) après avoir exécuté le code. Vous pouvez choisir entre plusieurs types de visualisations :

* Table de charge de travail
* Secteurs
* Lignes
* Domaine
* Barres

Voici le code permettant de tracer les données :

```scala
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
import matplotlib.pyplot as plt
%matplotlib inline

# PLOT TIP BY PAYMENT TYPE AND PASSENGER COUNT
ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
ax1.set_title('Tip amount distribution')
ax1.set_xlabel('Tip Amount ($)')
ax1.set_ylabel('Counts')
plt.suptitle('')
plt.show()

# PLOT TIP BY PASSENGER COUNT
ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
ax2.set_title('Tip amount by Passenger count')
ax2.set_xlabel('Passenger count')
ax2.set_ylabel('Tip Amount ($)')
plt.suptitle('')
plt.show()

# PLOT TIP AMOUNT BY FARE AMOUNT; SCALE POINTS BY PASSENGER COUNT
ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
ax.set_title('Tip amount by Fare amount')
ax.set_xlabel('Fare Amount ($)')
ax.set_ylabel('Tip Amount ($)')
plt.axis([-2, 80, -2, 20])
plt.show()
```

**Output:**

![Histogramme de montant de pourboire](./media/scala-walkthrough/plot-tip-amount-histogram.png)

![Montant de pourboire par nombre de passagers](./media/scala-walkthrough/plot-tip-amount-by-passenger-count.png)

![Montant du pourboire par montant de la course](./media/scala-walkthrough/plot-tip-amount-by-fare-amount.png)

## <a name="create-features-and-transform-features-and-then-prep-data-for-input-into-modeling-functions"></a>Créer des fonctions et fonctionnalités de transformation puis préparer les données à intégrer dans les fonctions de modélisation
Pour utiliser les fonctions de modélisation basées sur l’arborescence de Spark ML et MLlib, vous devez préparer la cible et les fonctionnalités à l’aide de diverses techniques, telles que le placement, l’indexation, l’encodage à chaud et la vectorisation. Voici les procédures à suivre dans cette section :

1. Créer une caractéristique en **regroupant** les heures dans des périodes de trafic
2. Appliquer une **indexation et un encodage « à chaud »** à des fonctionnalités catégorielles
3. **Échantillonner et diviser le jeu de données** en fractions de formation et de test
4. **Spécifier la variable de formation et les fonctionnalités** , puis créer des jeux de données distribués résilients (RDD) ou trames de données d’entrée libellés de formation et de test indexés ou encodés à chaud
5. Automatiquement **classer et vectoriser les fonctionnalités et cibles** pour une utilisation en tant qu’entrées pour les modèles Machine Learning

### <a name="create-a-new-feature-by-binning-hours-into-traffic-time-buckets"></a>Créer une caractéristique en regroupant les heures dans des périodes de trafic
Ce code vous montre comment créer une nouvelle caractéristique en regroupant les heures dans des périodes de trafic et comment mettre en cache la trame de données obtenue en mémoire. Là où les jeux de données distribués résilients (RDD) et les trames de données sont utilisés de manière répétitive, la mise en cache réduit les temps d’exécution. Par conséquent, nous allons mettre en cache les RDD et les trames de données à plusieurs stades de la procédure suivante.

```scala
# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
val sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night"
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush"
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_train
"""
val taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE THE DATA FRAME IN MEMORY AND MATERIALIZE THE DATA FRAME IN MEMORY
taxi_df_train_with_newFeatures.cache()
taxi_df_train_with_newFeatures.count()
```

### <a name="indexing-and-one-hot-encoding-of-categorical-features"></a>Indexation et encodage « à chaud » des fonctionnalités catégorielles
Les fonctions de modélisation et de prédiction de MLlib requièrent des caractéristiques avec des données d’entrée catégorielles à indexer ou à encoder avant leur utilisation. Cette section vous montre comment indexer ou encoder les caractéristiques catégorielles à intégrer dans les fonctions de modélisation.

Selon le modèle, vous devez indexer ou encoder vos modèles différemment. Par exemple, les modèles de régression logistique et linéaire nécessitent un encodage « à chaud ». Par exemple, une fonction avec trois catégories peut être étendue sur trois colonnes de fonctionnalités. Chaque colonne contient la valeur 0 ou 1 selon la catégorie d’une observation. MLlib fournit la fonction [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) pour l’encodage « à chaud ». Cet encodeur mappe une colonne d’index de libellé à une colonne de vecteurs binaires, contenant au plus une seule une valeur. Avec cet encodage, les algorithmes qui attendent des caractéristiques numériques, comme la régression logistique, peuvent être appliqués à des variables catégorielles.

Ici, vous transformez uniquement quatre variables pour afficher des exemples, qui sont des chaînes de caractères. Vous pouvez également indexer d’autres variables, telles que le jour de la semaine, représentées par des valeurs numériques, en tant que variables catégorielles.

Pour l’indexation, utilisez `StringIndexer()`, et pour l’encodage « à chaud », utilisez les fonctions `OneHotEncoder()` de MLlib. Voici le code permettant d’indexer et d’encoder des caractéristiques catégorielles :

```scala
# CREATE INDEXES AND ONE-HOT ENCODED VECTORS FOR SEVERAL CATEGORICAL FEATURES

# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# INDEX AND ENCODE VENDOR_ID
val stringIndexer = new StringIndexer().setInputCol("vendor_id").setOutputCol("vendorIndex").fit(taxi_df_train_with_newFeatures)
val indexed = stringIndexer.transform(taxi_df_train_with_newFeatures)
val encoder = new OneHotEncoder().setInputCol("vendorIndex").setOutputCol("vendorVec")
val encoded1 = encoder.transform(indexed)

# INDEX AND ENCODE RATE_CODE
val stringIndexer = new StringIndexer().setInputCol("rate_code").setOutputCol("rateIndex").fit(encoded1)
val indexed = stringIndexer.transform(encoded1)
val encoder = new OneHotEncoder().setInputCol("rateIndex").setOutputCol("rateVec")
val encoded2 = encoder.transform(indexed)

# INDEX AND ENCODE PAYMENT_TYPE
val stringIndexer = new StringIndexer().setInputCol("payment_type").setOutputCol("paymentIndex").fit(encoded2)
val indexed = stringIndexer.transform(encoded2)
val encoder = new OneHotEncoder().setInputCol("paymentIndex").setOutputCol("paymentVec")
val encoded3 = encoder.transform(indexed)

# INDEX AND TRAFFIC TIME BINS
val stringIndexer = new StringIndexer().setInputCol("TrafficTimeBins").setOutputCol("TrafficTimeBinsIndex").fit(encoded3)
val indexed = stringIndexer.transform(encoded3)
val encoder = new OneHotEncoder().setInputCol("TrafficTimeBinsIndex").setOutputCol("TrafficTimeBinsVec")
val encodedFinal = encoder.transform(indexed)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Output:**

Durée d’exécution de la cellule : 4 secondes.

### <a name="sample-and-split-the-data-set-into-training-and-test-fractions"></a>Échantillonner et diviser le jeu de données en fractions de formation et de test
Ce code crée un échantillonnage aléatoire des données (25 %, dans cet exemple). Bien que cet échantillonnage ne soit pas nécessaire dans cet exemple en raison de la taille du jeu de données, cet article vous montre comment créer un échantillon afin de savoir comment l’utiliser pour vos problèmes en cas de nécessité. Lorsque les échantillons sont volumineux, cela permet de gagner beaucoup de temps pendant l’apprentissage des modèles. Nous divisons ensuite l’échantillon en une partie d’apprentissage (75 %, dans cet exemple) et une partie de test (25 %n dans cet exemple) à utiliser dans la modélisation de la classification et de la régression.

Nous ajoutons un nombre aléatoire (entre 0 et 1) pour chaque ligne (dans une colonne « rand ») qui peut être utilisé pour sélectionner des plis de validation croisée pendant la formation.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# SPECIFY SAMPLING AND SPLITTING FRACTIONS
val samplingFraction = 0.25;
val trainingFraction = 0.75;
val testingFraction = (1-trainingFraction);
val seed = 1234;
val encodedFinalSampledTmp = encodedFinal.sample(withReplacement = false, fraction = samplingFraction, seed = seed)
val sampledDFcount = encodedFinalSampledTmp.count().toInt

val generateRandomDouble = udf(() => {
    scala.util.Random.nextDouble
})

# ADD A RANDOM NUMBER FOR CROSS-VALIDATION
val encodedFinalSampled = encodedFinalSampledTmp.withColumn("rand", generateRandomDouble());

# SPLIT THE SAMPLED DATA FRAME INTO TRAIN AND TEST, WITH A RANDOM COLUMN ADDED FOR DOING CROSS-VALIDATION (SHOWN LATER)
# INCLUDE A RANDOM COLUMN FOR CREATING CROSS-VALIDATION FOLDS
val splits = encodedFinalSampled.randomSplit(Array(trainingFraction, testingFraction), seed = seed)
val trainData = splits(0)
val testData = splits(1)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Output:**

Durée d’exécution de la cellule : 2 secondes.

### <a name="specify-training-variable-and-features-and-then-create-indexed-or-one-hot-encoded-training-and-testing-input-labeled-point-rdds-or-data-frames"></a>Spécifiez la variable de formation et les fonctionnalités puis créez des objets RDD ou trames de données d’entrée libellés de formation et de test indexés ou encodés à chaud.
Cette section contient le code qui vous montre comment indexer des données textuelles catégorielles en type point étiqueté et les encoder pour vous permettre de former et de tester la régression logistique de MLlib et d’autres modèles de classification. Les objets point étiquetés sont des RDD mis en forme en tant que données d’entrée utilisables par la plupart des algorithmes Machine Learning dans MLlib. Un [point étiqueté](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) est un vecteur local, dense ou fragmenté, associé à un libellé/une réponse.

Dans ce code, vous spécifiez la variable cible (dépendante) et les fonctionnalités à utiliser pour former des modèles. Vous créez ensuite des objets RDD ou trames de données d’entrée libellés de formation et de test indexés ou encodés à chaud.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# MAP NAMES OF FEATURES AND TARGETS FOR CLASSIFICATION AND REGRESSION PROBLEMS
val featuresIndOneHot = List("paymentVec", "vendorVec", "rateVec", "TrafficTimeBinsVec", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount").map(encodedFinalSampled.columns.indexOf(_))
val featuresIndIndex = List("paymentIndex", "vendorIndex", "rateIndex", "TrafficTimeBinsIndex", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount").map(encodedFinalSampled.columns.indexOf(_))

# SPECIFY THE TARGET FOR CLASSIFICATION ('tipped') AND REGRESSION ('tip_amount') PROBLEMS
val targetIndBinary = List("tipped").map(encodedFinalSampled.columns.indexOf(_))
val targetIndRegression = List("tip_amount").map(encodedFinalSampled.columns.indexOf(_))

# CREATE INDEXED LABELED POINT RDD OBJECTS
val indexedTRAINbinary = trainData.rdd.map(r => LabeledPoint(r.getDouble(targetIndBinary(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
val indexedTESTbinary = testData.rdd.map(r => LabeledPoint(r.getDouble(targetIndBinary(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
val indexedTRAINreg = trainData.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
val indexedTESTreg = testData.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))

# CREATE INDEXED DATA FRAMES THAT YOU CAN USE TO TRAIN BY USING SPARK ML FUNCTIONS
val indexedTRAINbinaryDF = indexedTRAINbinary.toDF()
val indexedTESTbinaryDF = indexedTESTbinary.toDF()
val indexedTRAINregDF = indexedTRAINreg.toDF()
val indexedTESTregDF = indexedTESTreg.toDF()

# CREATE ONE-HOT ENCODED (VECTORIZED) DATA FRAMES THAT YOU CAN USE TO TRAIN BY USING SPARK ML FUNCTIONS
val assemblerOneHot = new VectorAssembler().setInputCols(Array("paymentVec", "vendorVec", "rateVec", "TrafficTimeBinsVec", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount")).setOutputCol("features")
val OneHotTRAIN = assemblerOneHot.transform(trainData)
val OneHotTEST = assemblerOneHot.transform(testData)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Output:**

Durée d’exécution de la cellule : 4 secondes.

### <a name="automatically-categorize-and-vectorize-features-and-targets-to-use-as-inputs-for-machine-learning-models"></a>Automatiquement classer et vectoriser les fonctionnalités et cibles pour une utilisation en tant qu’entrées pour les modèles Machine Learning.
Utilisez Spark ML pour classer correctement la cible et les fonctionnalités à utiliser dans les fonctions de modélisation en arborescence. Le code effectue deux tâches :

* Crée une cible binaire pour la classification en affectant une valeur de 0 ou 1 pour chaque point de données compris entre 0 et 1 avec une valeur de seuil de 0,5.
* Classe automatiquement les fonctionnalités. Si le nombre de valeurs numériques distinctes pour une des fonctionnalités est inférieur à 32, cette fonctionnalité est classée.

Voici le code de ces deux tâches.

```scala
# CATEGORIZE FEATURES AND BINARIZE THE TARGET FOR THE BINARY CLASSIFICATION PROBLEM

# TRAIN DATA
val indexer = new VectorIndexer().setInputCol("features").setOutputCol("featuresCat").setMaxCategories(32)
val indexerModel = indexer.fit(indexedTRAINbinaryDF)
val indexedTrainwithCatFeat = indexerModel.transform(indexedTRAINbinaryDF)
val binarizer: Binarizer = new Binarizer().setInputCol("label").setOutputCol("labelBin").setThreshold(0.5)
val indexedTRAINwithCatFeatBinTarget = binarizer.transform(indexedTrainwithCatFeat)

# TEST DATA
val indexerModel = indexer.fit(indexedTESTbinaryDF)
val indexedTrainwithCatFeat = indexerModel.transform(indexedTESTbinaryDF)
val binarizer: Binarizer = new Binarizer().setInputCol("label").setOutputCol("labelBin").setThreshold(0.5)
val indexedTESTwithCatFeatBinTarget = binarizer.transform(indexedTrainwithCatFeat)

# CATEGORIZE FEATURES FOR THE REGRESSION PROBLEM
# CREATE PROPERLY INDEXED AND CATEGORIZED DATA FRAMES FOR TREE-BASED MODELS

# TRAIN DATA
val indexer = new VectorIndexer().setInputCol("features").setOutputCol("featuresCat").setMaxCategories(32)
val indexerModel = indexer.fit(indexedTRAINregDF)
val indexedTRAINwithCatFeat = indexerModel.transform(indexedTRAINregDF)

# TEST DATA
val indexerModel = indexer.fit(indexedTESTbinaryDF)
val indexedTESTwithCatFeat = indexerModel.transform(indexedTESTregDF)
```


## <a name="binary-classification-model-predict-whether-a-tip-should-be-paid"></a>Modèle de classification binaire : Prédire si un pourboire devrait être payé
Dans cette section, vous créez trois types de modèles de classification binaire pour prédire si un pourboire devrait être payé :

* Un **modèle de régression logistique** à l’aide de la fonction Spark ML `LogisticRegression()`
* Un **modèle de classification par forêts aléatoires** à l’aide de la fonction Spark ML `RandomForestClassifier()`
* Un **modèle de classification par arbres Gradient Boosting** à l’aide de la fonction MLlib `GradientBoostedTrees()`

### <a name="create-a-logistic-regression-model"></a>Créer un modèle de régression logistique
Nous créons ensuite un modèle de régression logistique à l’aide de la fonction Spark ML `LogisticRegression()` . Vous créez le modèle de génération de code dans une série d’étapes :

1. **Formation des données du modèle** avec un jeu de paramètres.
2. **Évaluation de modèle** sur un jeu de données de test avec des mesures.
3. **Enregistrement du modèle** dans le stockage d’objets blob en vue d’une utilisation ultérieure.
4. **Notation du modèle** sur les données de test.
5. **Représentation graphique des résultats** avec des courbes ROC (Receiver Operating Characteristic).

Voici le code pour ces procédures :

```scala
# CREATE A LOGISTIC REGRESSION MODEL
val lr = new LogisticRegression().setLabelCol("tipped").setFeaturesCol("features").setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)
val lrModel = lr.fit(OneHotTRAIN)

# PREDICT ON THE TEST DATA SET
val predictions = lrModel.transform(OneHotTEST)

# SELECT `BinaryClassificationEvaluator()` TO COMPUTE THE TEST ERROR
val evaluator = new BinaryClassificationEvaluator().setLabelCol("tipped").setRawPredictionCol("probability").setMetricName("areaUnderROC")
val ROC = evaluator.evaluate(predictions)
println("ROC on test data = " + ROC)

# SAVE THE MODEL
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "LogisticRegression__"
val filename = modelDir.concat(modelName).concat(datestamp)
lrModel.save(filename);
```

Chargez, notez et enregistrez les résultats.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# LOAD THE SAVED MODEL AND SCORE THE TEST DATA SET
val savedModel = org.apache.spark.ml.classification.LogisticRegressionModel.load(filename)
println(s"Coefficients: ${savedModel.coefficients} Intercept: ${savedModel.intercept}")

# SCORE THE MODEL ON THE TEST DATA
val predictions = savedModel.transform(OneHotTEST).select("tipped","probability","rawPrediction")
predictions.registerTempTable("testResults")

# SELECT `BinaryClassificationEvaluator()` TO COMPUTE THE TEST ERROR
val evaluator = new BinaryClassificationEvaluator().setLabelCol("tipped").setRawPredictionCol("probability").setMetricName("areaUnderROC")
val ROC = evaluator.evaluate(predictions)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE ROC RESULTS
println("ROC on test data = " + ROC)
```

**Output:**

ROC sur les données de test = 0,9827381497557599

Utilisez Python sur des trames de données Pandas locales pour tracer la courbe ROC.

```scala
# QUERY THE RESULTS
%%sql -q -o sqlResults
SELECT tipped, probability from testResults


# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline
from sklearn.metrics import roc_curve,auc

sqlResults['probFloat'] = sqlResults.apply(lambda row: row['probability'].values()[0][1], axis=1)
predictions_pddf = sqlResults[["tipped","probFloat"]]

# PREDICT THE ROC CURVE
# predictions_pddf = sqlResults.rename(columns={'_1': 'probability', 'tipped': 'label'})
prob = predictions_pddf["probFloat"]
fpr, tpr, thresholds = roc_curve(predictions_pddf['tipped'], prob, pos_label=1);
roc_auc = auc(fpr, tpr)

# PLOT THE ROC CURVE
plt.figure(figsize=(5,5))
plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
plt.plot([0, 1], [0, 1], 'k--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend(loc="lower right")
plt.show()
```

**Output:**

![Courbe ROC de remise de pourboire ou non](./media/scala-walkthrough/plot-roc-curve-tip-or-not.png)

### <a name="create-a-random-forest-classification-model"></a>Créer un modèle de classification par forêts aléatoires
Nous créons maintenant un modèle de classification de forêt aléatoire en utilisant la fonction Spark ML `RandomForestClassifier()` puis évaluons le modèle de données de test.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# CREATE THE RANDOM FOREST CLASSIFIER MODEL
val rf = new RandomForestClassifier().setLabelCol("labelBin").setFeaturesCol("featuresCat").setNumTrees(10).setSeed(1234)

# FIT THE MODEL
val rfModel = rf.fit(indexedTRAINwithCatFeatBinTarget)
val predictions = rfModel.transform(indexedTESTwithCatFeatBinTarget)

# EVALUATE THE MODEL
val evaluator = new MulticlassClassificationEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("f1")
val Test_f1Score = evaluator.evaluate(predictions)
println("F1 score on test data: " + Test_f1Score);

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# CALCULATE BINARY CLASSIFICATION EVALUATION METRICS
val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setRawPredictionCol("probability").setMetricName("areaUnderROC")
val ROC = evaluator.evaluate(predictions)
println("ROC on test data = " + ROC)
```

**Output:**

ROC sur les données de test = 0.9847103571552683

### <a name="create-a-gbt-classification-model"></a>Créer un modèle de classification GBT
Nous créons ensuite un modèle de classification GBT en utilisant la fonction MLlib `GradientBoostedTrees()` puis évaluons le modèle de données de test.

```scala
# TRAIN A GBT CLASSIFICATION MODEL BY USING MLLIB AND A LABELED POINT

# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# DEFINE THE GBT CLASSIFICATION MODEL
val boostingStrategy = BoostingStrategy.defaultParams("Classification")
boostingStrategy.numIterations = 20
boostingStrategy.treeStrategy.numClasses = 2
boostingStrategy.treeStrategy.maxDepth = 5
boostingStrategy.treeStrategy.categoricalFeaturesInfo = Map[Int, Int]((0,2),(1,2),(2,6),(3,4))

# TRAIN THE MODEL
val gbtModel = GradientBoostedTrees.train(indexedTRAINbinary, boostingStrategy)

# SAVE THE MODEL IN BLOB STORAGE
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "GBT_Classification__"
val filename = modelDir.concat(modelName).concat(datestamp)
gbtModel.save(sc, filename);

# EVALUATE THE MODEL ON TEST INSTANCES AND THE COMPUTE TEST ERROR
val labelAndPreds = indexedTESTbinary.map { point =>
  val prediction = gbtModel.predict(point.features)
  (point.label, prediction)
}
val testErr = labelAndPreds.filter(r => r._1 != r._2).count.toDouble / indexedTRAINbinary.count()
//println("Learned classification GBT model:\n" + gbtModel.toDebugString)
println("Test Error = " + testErr)

# USE BINARY AND MULTICLASS METRICS TO EVALUATE THE MODEL ON THE TEST DATA
val metrics = new MulticlassMetrics(labelAndPreds)
println(s"Precision: ${metrics.precision}")
println(s"Recall: ${metrics.recall}")
println(s"F1 Score: ${metrics.fMeasure}")

val metrics = new BinaryClassificationMetrics(labelAndPreds)
println(s"Area under PR curve: ${metrics.areaUnderPR}")
println(s"Area under ROC curve: ${metrics.areaUnderROC}")

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE ROC METRIC
println(s"Area under ROC curve: ${metrics.areaUnderROC}")
```

**Output:**

Zone sous courbe ROC : 0.9846895479241554

## <a name="regression-model-predict-tip-amount"></a>Modèle de régression : prédire le montant du pourboire
Dans cette section, vous créez deux types de modèles de régression pour prédire le montant du pourboire :

* Un **modèle de régression linéaire régularisée** à l’aide de la fonction Spark ML `LinearRegression()`. Vous allez enregistrer le modèle et évaluer le modèle de données de test.
* Un **modèle de régression par arbres Gradient Boosting** à l’aide de la fonction `GBTRegressor()` de Spark ML.

### <a name="create-a-regularized-linear-regression-model"></a>Créer un modèle de régression linéaire régularisée

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# CREATE A REGULARIZED LINEAR REGRESSION MODEL BY USING THE SPARK ML FUNCTION AND DATA FRAMES
val lr = new LinearRegression().setLabelCol("tip_amount").setFeaturesCol("features").setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)

# FIT THE MODEL BY USING DATA FRAMES
val lrModel = lr.fit(OneHotTRAIN)
println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

# SUMMARIZE THE MODEL OVER THE TRAINING SET AND PRINT METRICS
val trainingSummary = lrModel.summary
println(s"numIterations: ${trainingSummary.totalIterations}")
println(s"objectiveHistory: ${trainingSummary.objectiveHistory.toList}")
trainingSummary.residuals.show()
println(s"RMSE: ${trainingSummary.rootMeanSquaredError}")
println(s"r2: ${trainingSummary.r2}")

# SAVE THE MODEL IN AZURE BLOB STORAGE
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "LinearRegression__"
val filename = modelDir.concat(modelName).concat(datestamp)
lrModel.save(filename);

# PRINT THE COEFFICIENTS
println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

# SCORE THE MODEL ON TEST DATA
val predictions = lrModel.transform(OneHotTEST)

# EVALUATE THE MODEL ON TEST DATA
val evaluator = new RegressionEvaluator().setLabelCol("tip_amount").setPredictionCol("prediction").setMetricName("r2")
val r2 = evaluator.evaluate(predictions)
println("R-sqr on test data = " + r2)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```


**Output:**

Durée d’exécution de la cellule : 13 secondes.

```scala
# LOAD A SAVED LINEAR REGRESSION MODEL FROM BLOB STORAGE AND SCORE A TEST DATA SET

# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# LOAD A SAVED LINEAR REGRESSION MODEL FROM AZURE BLOB STORAGE
val savedModel = org.apache.spark.ml.regression.LinearRegressionModel.load(filename)
println(s"Coefficients: ${savedModel.coefficients} Intercept: ${savedModel.intercept}")

# SCORE THE MODEL ON TEST DATA
val predictions = savedModel.transform(OneHotTEST).select("tip_amount","prediction")
predictions.registerTempTable("testResults")

# EVALUATE THE MODEL ON TEST DATA
val evaluator = new RegressionEvaluator().setLabelCol("tip_amount").setPredictionCol("prediction").setMetricName("r2")
val r2 = evaluator.evaluate(predictions)
println("R-sqr on test data = " + r2)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE RESULTS
println("R-sqr on test data = " + r2)
```

**Output:**

R-sqr sur les données de test = 0,5960320470835743

Interrogez ensuite les résultats de test sous forme d’une trame de données puis utilisez AutoVizWidget et matplotlib pour les visualiser.

```sql
# RUN A SQL QUERY
%%sql -q -o sqlResults
select * from testResults

# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES
# CLICK THE TYPE OF PLOT TO GENERATE (LINE, AREA, BAR, AND SO ON)
sqlResults
```

Le code crée un tableau de données local à partir de la sortie de la requête et il trace les données. La commande magique `%%local` crée un tableau de données local, `sqlResults`, que vous pouvez utiliser pour le tracé avec matplotlib.

> [!NOTE]
> Cette commande magique Spark est utilisée plusieurs fois lors de cet article. Si la quantité de données est élevée, vous devez échantillonner pour créer un tableau de données adapté à la mémoire locale.
> 
> 

Créez des tracés en utilisant matplotlib de Python.

```scala
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
sqlResults
%matplotlib inline
import numpy as np

# PLOT THE RESULTS
ax = sqlResults.plot(kind='scatter', figsize = (6,6), x='tip_amount', y='prediction', color='blue', alpha = 0.25, label='Actual vs. predicted');
fit = np.polyfit(sqlResults['tip_amount'], sqlResults['prediction'], deg=1)
ax.set_title('Actual vs. Predicted Tip Amounts ($)')
ax.set_xlabel("Actual")
ax.set_ylabel("Predicted")
#ax.plot(sqlResults['tip_amount'], fit[0] * sqlResults['prediction'] + fit[1], color='magenta')
plt.axis([-1, 15, -1, 8])
plt.show(ax)
```

**Output:**

![Montant du pourboire : réel vs. prédit](./media/scala-walkthrough/plot-actual-vs-predicted-tip-amount.png)

### <a name="create-a-gbt-regression-model"></a>Créer un modèle de régression GBT
Créez un modèle de régression GBT à l’aide de la fonction Spark ML `GBTRegressor()` puis évaluez le modèle de données de test.

Les [Gradient Boosting Trees](https://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) sont des ensembles d’arbres de décision. Ils effectuent l’apprentissage des arbres de décision de façon itérative pour réduire une fonction de perte. Vous pouvez utiliser des GBT pour la régression et la classification. Ils gèrent les caractéristiques catégorielles, ne requièrent aucune mise à l’échelle des caractéristiques et peuvent capturer les non-linéarités ainsi que les interactions entre les caractéristiques. Vous pouvez également les utiliser dans le paramétrage de classification multiclasse.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# TRAIN A GBT REGRESSION MODEL
val gbt = new GBTRegressor().setLabelCol("label").setFeaturesCol("featuresCat").setMaxIter(10)
val gbtModel = gbt.fit(indexedTRAINwithCatFeat)

# MAKE PREDICTIONS
val predictions = gbtModel.transform(indexedTESTwithCatFeat)

# COMPUTE TEST SET R2
val evaluator = new RegressionEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("r2")
val Test_R2 = evaluator.evaluate(predictions)


# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE RESULTS
println("Test R-sqr is: " + Test_R2);
```

**Output:**

Le R-sqr de test est : 0.7655383534596654

## <a name="advanced-modeling-utilities-for-optimization"></a>Utilitaires de modélisation avancée pour l’optimisation
Dans cette section, vous exécutez les utilitaires Machine Learning que les développeurs utilisent souvent pour l’optimisation du modèle. Plus précisément, vous pouvez optimiser les modèles Machine Learning de trois façons à l’aide du balayage paramétrique et de la validation croisée :

* Fractionnez les données en ensembles de formation et de validation, optimisez le modèle à l’aide du balayage hyperparamétrique sur un jeu formation, puis effectuer une évaluation sur un jeu de validation (régression linéaire)
* Optimiser le modèle à l’aide de la validation croisée et du balayage hyperparamétrique, à l’aide de la fonction CrossValidator de Spark ML (classification binaire)
* Optimisez le modèle en utilisant un code de validation croisée et de balayage paramétrique pour utiliser toute fonction Machine Learning et tout jeu de paramètres (régression linéaire)

**validation croisée** est une technique qui évalue la généralisation d’un modèle formé sur un jeu connu de données pour la prédiction des caractéristiques d’un jeu de données sur lequel il n’a pas été formé. L’idée derrière cette technique est de former un modèle sur un jeu de données connues, puis d’évaluer la précision de ses prédictions sur un jeu de données indépendant. Une implémentation commune consiste à diviser un jeu de données en plis *k* , puis de former le modèle par la méthode tourniquet (round robin) sur tous les plis sauf un.

**optimisation hyperparamétrique** consiste à choisir un jeu d’hyperparamètres pour un algorithme d’apprentissage, généralement dans le but d’optimiser la mesure des performances de l’algorithme sur un jeu de données indépendant. Un hyperparamètre est une valeur que vous devez spécifier à l’extérieur de la procédure de formation du modèle. Les hypothèses concernant ces valeurs hyperparamétriques peuvent affecter la flexibilité et la précision du modèle. Les arbres de décision ont des hyperparamètres, tels que la profondeur voulue et le nombre de feuilles de l’arbre. Vous devez définir un terme de pénalité en cas d’erreur de classification pour une machine à vecteurs de support (SVM).

Une façon courante d’effectuer l’optimisation hyperparamétrique consiste à utiliser la recherche par grille, également appelée **un balayage paramétrique**. Dans une recherche par grille, une recherche exhaustive d’un algorithme d’apprentissage est effectuée sur les valeurs d’un sous-ensemble spécifié de l’espace hyperparamétrique. La validation croisée peut fournir une mesure de performance permettant de trier les résultats optimaux produits par l’algorithme de recherche par grille. Si vous utilisez un balayage hyperparamétrique de validation croisée, vous pouvez limiter certains problèmes comme le surajustement d’un modèle pour les données de formation. De cette façon, le modèle conserve sa capacité à s’appliquer au jeu de données général à partir duquel les données de formation ont été extraites.

### <a name="optimize-a-linear-regression-model-with-hyper-parameter-sweeping"></a>Optimiser un modèle de régression linéaire avec le balayage paramétrique
Fractionnez ensuite les données en ensembles de formation et de validation, utilisez le balayage hyperparamétrique sur un jeu de formation pour optimiser le modèle, puis effectuez une évaluez sur un jeu de validation (régression linéaire).

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# RENAME `tip_amount` AS A LABEL
val OneHotTRAINLabeled = OneHotTRAIN.select("tip_amount","features").withColumnRenamed(existingName="tip_amount",newName="label")
val OneHotTESTLabeled = OneHotTEST.select("tip_amount","features").withColumnRenamed(existingName="tip_amount",newName="label")
OneHotTRAINLabeled.cache()
OneHotTESTLabeled.cache()

# DEFINE THE ESTIMATOR FUNCTION: `THE LinearRegression()` FUNCTION
val lr = new LinearRegression().setLabelCol("label").setFeaturesCol("features").setMaxIter(10)

# DEFINE THE PARAMETER GRID
val paramGrid = new ParamGridBuilder().addGrid(lr.regParam, Array(0.1, 0.01, 0.001)).addGrid(lr.fitIntercept).addGrid(lr.elasticNetParam, Array(0.1, 0.5, 0.9)).build()

# DEFINE THE PIPELINE WITH A TRAIN/TEST VALIDATION SPLIT (75% IN THE TRAINING SET), AND THEN THE SPECIFY ESTIMATOR, EVALUATOR, AND PARAMETER GRID
val trainPct = 0.75
val trainValidationSplit = new TrainValidationSplit().setEstimator(lr).setEvaluator(new RegressionEvaluator).setEstimatorParamMaps(paramGrid).setTrainRatio(trainPct)

# RUN THE TRAIN VALIDATION SPLIT AND CHOOSE THE BEST SET OF PARAMETERS
val model = trainValidationSplit.fit(OneHotTRAINLabeled)

# MAKE PREDICTIONS ON THE TEST DATA BY USING THE MODEL WITH THE COMBINATION OF PARAMETERS THAT PERFORMS THE BEST
val testResults = model.transform(OneHotTESTLabeled).select("label", "prediction")

# COMPUTE TEST SET R2
val evaluator = new RegressionEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("r2")
val Test_R2 = evaluator.evaluate(testResults)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

println("Test R-sqr is: " + Test_R2);
```

**Output:**

Le R-sqr de test est : 0.6226484708501209

### <a name="optimize-the-binary-classification-model-by-using-cross-validation-and-hyper-parameter-sweeping"></a>Optimiser le modèle de classification binaire à l’aide de la validation croisée et du balayage hyperparamétrique
Cette section vous montre comment optimiser un modèle de classification binaire à l’aide de la validation croisée et du balayage hyperparamétrique. Elle utilise la fonction Spark ML `CrossValidator` .

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# CREATE DATA FRAMES WITH PROPERLY LABELED COLUMNS TO USE WITH THE TRAIN AND TEST SPLIT
val indexedTRAINwithCatFeatBinTargetRF = indexedTRAINwithCatFeatBinTarget.select("labelBin","featuresCat").withColumnRenamed(existingName="labelBin",newName="label").withColumnRenamed(existingName="featuresCat",newName="features")
val indexedTESTwithCatFeatBinTargetRF = indexedTESTwithCatFeatBinTarget.select("labelBin","featuresCat").withColumnRenamed(existingName="labelBin",newName="label").withColumnRenamed(existingName="featuresCat",newName="features")
indexedTRAINwithCatFeatBinTargetRF.cache()
indexedTESTwithCatFeatBinTargetRF.cache()

# DEFINE THE ESTIMATOR FUNCTION
val rf = new RandomForestClassifier().setLabelCol("label").setFeaturesCol("features").setImpurity("gini").setSeed(1234).setFeatureSubsetStrategy("auto").setMaxBins(32)

# DEFINE THE PARAMETER GRID
val paramGrid = new ParamGridBuilder().addGrid(rf.maxDepth, Array(4,8)).addGrid(rf.numTrees, Array(5,10)).addGrid(rf.minInstancesPerNode, Array(100,300)).build()

# SPECIFY THE NUMBER OF FOLDS
val numFolds = 3

# DEFINE THE TRAIN/TEST VALIDATION SPLIT (75% IN THE TRAINING SET)
val CrossValidator = new CrossValidator().setEstimator(rf).setEvaluator(new BinaryClassificationEvaluator).setEstimatorParamMaps(paramGrid).setNumFolds(numFolds)

# RUN THE TRAIN VALIDATION SPLIT AND CHOOSE THE BEST SET OF PARAMETERS
val model = CrossValidator.fit(indexedTRAINwithCatFeatBinTargetRF)

# MAKE PREDICTIONS ON THE TEST DATA BY USING THE MODEL WITH THE COMBINATION OF PARAMETERS THAT PERFORMS THE BEST
val testResults = model.transform(indexedTESTwithCatFeatBinTargetRF).select("label", "prediction")

# COMPUTE THE TEST F1 SCORE
val evaluator = new MulticlassClassificationEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("f1")
val Test_f1Score = evaluator.evaluate(testResults)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Output:**

Durée d’exécution de la cellule : 33 secondes.

### <a name="optimize-the-linear-regression-model-by-using-custom-cross-validation-and-parameter-sweeping-code"></a>Optimiser le modèle de régression linéaire à l’aide de code personnalisé de validation croisée et de balayage paramétrique
Nous optimisons ensuite le modèle à l’aide d’un code personnalisé et identifions les meilleurs paramètres de modèle en utilisant le critère de précision le plus élevé. Puis nous créons le modèle final, évaluons le modèle sur des données de test et enregistrons le modèle dans un stockage d’objets Blob. Enfin, nous chargeons le modèle, notons les données de test et évaluons la précision.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# DEFINE THE PARAMETER GRID AND THE NUMBER OF FOLDS
val paramGrid = new ParamGridBuilder().addGrid(rf.maxDepth, Array(5,10)).addGrid(rf.numTrees, Array(10,25,50)).build()

val nFolds = 3
val numModels = paramGrid.size
val numParamsinGrid = 2

# SPECIFY THE NUMBER OF CATEGORIES FOR CATEGORICAL VARIABLES
val categoricalFeaturesInfo = Map[Int, Int]((0,2),(1,2),(2,6),(3,4))

var maxDepth = -1
var numTrees = -1
var param = ""
var paramval = -1
var validateLB = -1.0
var validateUB = -1.0
val h = 1.0 / nFolds;
val RMSE  = Array.fill(numModels)(0.0)

# CREATE K-FOLDS
val splits = MLUtils.kFold(indexedTRAINbinary, numFolds = nFolds, seed=1234)


# LOOP THROUGH K-FOLDS AND THE PARAMETER GRID TO GET AND IDENTIFY THE BEST PARAMETER SET BY LEVEL OF ACCURACY
for (i <- 0 to (nFolds-1)) {
    validateLB = i * h
    validateUB = (i + 1) * h
    val validationCV = trainData.filter($"rand" >= validateLB  && $"rand" < validateUB)
    val trainCV = trainData.filter($"rand" < validateLB  || $"rand" >= validateUB)
    val validationLabPt = validationCV.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)));
    val trainCVLabPt = trainCV.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)));
    validationLabPt.cache()
    trainCVLabPt.cache()

    for (nParamSets <- 0 to (numModels-1)) {
        for (nParams <- 0 to (numParamsinGrid-1)) {
            param = paramGrid(nParamSets).toSeq(nParams).param.toString.split("__")(1)
            paramval = paramGrid(nParamSets).toSeq(nParams).value.toString.toInt
            if (param == "maxDepth") {maxDepth = paramval}
            if (param == "numTrees") {numTrees = paramval}
        }
        val rfModel = RandomForest.trainRegressor(trainCVLabPt, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                  numTrees=numTrees, maxDepth=maxDepth,
                                                  featureSubsetStrategy="auto",impurity="variance", maxBins=32)
        val labelAndPreds = validationLabPt.map { point =>
                                                 val prediction = rfModel.predict(point.features)
                                                  ( prediction, point.label )
                                                }
        val validMetrics = new RegressionMetrics(labelAndPreds)
        val rmse = validMetrics.rootMeanSquaredError
        RMSE(nParamSets) += rmse
    }
    validationLabPt.unpersist();
    trainCVLabPt.unpersist();
}
val minRMSEindex = RMSE.indexOf(RMSE.min)

# GET THE BEST PARAMETERS FROM A CROSS-VALIDATION AND PARAMETER SWEEP
var best_maxDepth = -1
var best_numTrees = -1
for (nParams <- 0 to (numParamsinGrid-1)) {
    param = paramGrid(minRMSEindex).toSeq(nParams).param.toString.split("__")(1)
    paramval = paramGrid(minRMSEindex).toSeq(nParams).value.toString.toInt
    if (param == "maxDepth") {best_maxDepth = paramval}
    if (param == "numTrees") {best_numTrees = paramval}
}

# CREATE THE BEST MODEL WITH THE BEST PARAMETERS AND A FULL TRAINING DATA SET
val best_rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                  numTrees=best_numTrees, maxDepth=best_maxDepth,
                                                  featureSubsetStrategy="auto",impurity="variance", maxBins=32)

# SAVE THE BEST RANDOM FOREST MODEL IN BLOB STORAGE
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "BestCV_RF_Regression__"
val filename = modelDir.concat(modelName).concat(datestamp)
best_rfModel.save(sc, filename);

# PREDICT ON THE TRAINING SET WITH THE BEST MODEL AND THEN EVALUATE
val labelAndPreds = indexedTESTreg.map { point =>
                                        val prediction = best_rfModel.predict(point.features)
                                        ( prediction, point.label )
                                       }

val test_rmse = new RegressionMetrics(labelAndPreds).rootMeanSquaredError
val test_rsqr = new RegressionMetrics(labelAndPreds).r2

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");


# LOAD THE MODEL
val savedRFModel = RandomForestModel.load(sc, filename)

val labelAndPreds = indexedTESTreg.map { point =>
                                        val prediction = savedRFModel.predict(point.features)
                                        ( prediction, point.label )
                                       }
# TEST THE MODEL
val test_rmse = new RegressionMetrics(labelAndPreds).rootMeanSquaredError
val test_rsqr = new RegressionMetrics(labelAndPreds).r2
```

**Output:**

Durée d’exécution de la cellule : 61 secondes.

## <a name="consume-spark-built-machine-learning-models-automatically-with-scala"></a>Utiliser des modèles Machine Learning basés sur Spark générés automatiquement avec Scala
Pour une vue d’ensemble des rubriques qui vous guident à travers les tâches qui constituent le processus de science des données dans Azure, consultez [processus de science des données pour les équipes](./index.yml).

[Procédures pas à pas du processus TDSP (Team Data Science Process)](walkthroughs.md) décrit les autres procédures pas à pas complètent illustrant les étapes du processus TDSP pour des scénarios spécifiques. Les procédures pas à pas montrent également comment combiner les outils et services dans le cloud et sur site dans un flux de travail ou un pipeline pour créer une application intelligente.

[Noter les modèles Machine Learning créés avec Spark](spark-model-consumption.md) vous montre comment utiliser du code Scala pour charger automatiquement et noter les nouveaux jeux de données avec des modèles Machine Learning basés sur Spark et enregistrés dans des objets blob Azure. Vous pouvez suivre les instructions fournies et simplement remplacer le code Python par le code Scala de cet article pour activer la consommation automatisée.