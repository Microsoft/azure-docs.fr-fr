---
title: Utiliser des clusters Azure HDInsight Hadoop sur un jeu de données de 1 To - Team Data Science Process
description: Utilisation du processus TDSP (Team Data Science Process) pour un scénario de bout en bout employant un cluster Hadoop HDInsight pour créer et déployer un modèle à l'aide d'un groupe de données volumineux (1 To), disponible publiquement
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
ms.openlocfilehash: e66bd0a4e56f63185d8361355d6cf8e0e29bc30b
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305933"
---
# <a name="the-team-data-science-process-in-action---using-an-azure-hdinsight-hadoop-cluster-on-a-1-tb-dataset"></a>Processus TDSP (Team Data Science Process) en action : utilisation d’un cluster Azure HDInsight Hadoop sur un jeu de données de 1 To

Cette procédure pas à pas vous explique comment utiliser le processus TDSP (Team Data Science Process) avec un scénario complet au moyen d’un [cluster Azure HDInsight Hadoop](https://azure.microsoft.com/services/hdinsight/) pour effectuer des opérations sur un des jeux de données [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/), disponibles publiquement, telles que le stockage, l’exploration, la conception de fonctionnalités et la réduction d’échantillon. Elle utilise Azure Machine Learning pour créer un modèle de classification binaire sur ces données. Elle vous montre également comment publier un de ces modèles en tant que service web.

Il est également possible d'utiliser un interpréteur IPython notebook pour accomplir les tâches présentées dans cette procédure pas à pas. Les utilisateurs qui souhaitent essayer cette approche doivent consulter la rubrique [Procédure pas à pas Criteo à l’aide d’une connexion Hive ODBC](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-hive-walkthrough-criteo.ipynb) .

## <a name="criteo-dataset-description"></a><a name="dataset"></a>Description du groupe de données Criteo
Les données Criteo représentent un jeu de données de prédiction de clic, correspondant à 370 Go de fichiers TSV compressés au format gzip (~1,30 To de fichiers non compressés), comprenant plus de 4,3 milliards d’enregistrements. Elles proviennent de 24 jours de données de clic, disponibles via [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/). Pour la commodité des scientifiques de données, les données mises à notre disposition pour pouvoir les expérimenter ont été décompressées.

Chaque enregistrement de ce groupe de données est constitué de 40 colonnes :

* la première colonne est une colonne étiquette qui vous indique si un utilisateur clique sur un **ajout** (valeur 1) ou non (valeur 0) ;
* Les 13 colonnes suivantes sont numériques, et
* les 26 dernières sont des colonnes catégorielles

Les colonnes sont anonymes et utilisent une série de noms énumérés : « Col1 » (pour la colonne d’étiquette) à « Col40 » (pour la dernière colonne de catégorie).

Voici un extrait des 20 premières colonnes de deux observations (lignes) provenant de ce groupe de données :

> `Col1    Col2    Col3    Col4    Col5    Col6    Col7    Col8    Col9    Col10    Col11    Col12    Col13    Col14    Col15            Col16            Col17            Col18            Col19        Col20`
>
> `0       40      42      2       54      3       0       0       2       16      0       1       4448    4       1acfe1ee        1b2ff61f        2e8b2631        6faef306        c6fc10d3    6fcd6dcb`
>
> `0               24              27      5               0       2       1               3       10064           9a8cb066        7a06385f        417e6103        2170fc56        acf676aa    6fcd6dcb`

Des valeurs sont manquantes dans les colonnes numériques et catégorielles de ce groupe de données. Une méthode simple pour gérer les valeurs manquantes est détaillée. Des informations supplémentaires relatives aux données sont disponibles lors de leur stockage dans les tables Hive.

**Définition :** *Taux de clic (CTR) :* cette métrique donne le pourcentage de clics effectués dans les données. Dans ce groupe de données Criteo, le taux de clic est d’environ 3,3 %, soit de 0,033.

## <a name="examples-of-prediction-tasks"></a><a name="mltasks"></a>Exemples de tâches de prédiction
Cette procédure pas à pas aborde deux exemples de problèmes de prédiction :

1. **Classification binaire** : prédit si un utilisateur a cliqué ou non sur une annonce :

   * Classe 0 : Aucun clic
   * Classe 1 : Cliquez sur
2. **Régression** : prédit la probabilité d’un clic effectué sur une annonce à partir de fonctionnalités utilisateur.

## <a name="set-up-an-hdinsight-hadoop-cluster-for-data-science"></a><a name="setup"></a>Configuration d’un cluster Hadoop HDInsight pour la science des données
> [!NOTE]
> Cette étape est généralement une tâche d’ **administration**.

Configurez votre environnement de science des données Azure pour créer des solutions d'analyse prédictives avec les clusters HDInsight en trois étapes :

1. [Créer un compte de stockage](../../storage/common/storage-account-create.md) : ce compte de stockage est utilisé pour stocker des données dans un stockage Blob Azure. Les données utilisées dans les clusters HDInsight sont stockées ici.
2. [Personnaliser les clusters Azure HDInsight Hadoop pour la science des données](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md) : Cette étape crée un cluster Hadoop Azure HDInsight avec Anaconda Python 2.7 64 bits installé sur tous les nœuds. Deux étapes importantes (décrites dans cette rubrique) doivent être suivies lors de la personnalisation du cluster HDInsight.

   * Liez le compte de stockage créé à l’étape 1 à votre cluster HDInsight une fois sa création terminée. Ce compte de stockage est utilisé pour accéder aux données qui peuvent être traitées au sein du cluster.
   * Activez l’accès à distance sur le nœud principal du cluster après sa création. Souvenez-vous des informations d’identification de l’accès à distance que vous indiquez ici (différentes de celles qui sont spécifiées à la création du cluster) : effectuez les procédures suivantes.
3. [Créez un espace de travail Azure Machine Learning Studio (classique)](../classic/create-workspace.md) : espace de travail Azure Machine Learning utilisé pour créer des modèles d’apprentissage automatique après avoir exploré des données initiales et réduit l’échantillon sur le cluster HDInsight.

## <a name="get-and-consume-data-from-a-public-source"></a><a name="getdata"></a>Récupération et utilisation des données provenant d’une source publique
Pour accéder au groupe de données [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/) , cliquez sur le lien, acceptez les conditions d'utilisation et saisissez un nom. En voici l’illustration :

![Accepter les conditions de Criteo](./media/hive-criteo-walkthrough/hLxfI2E.png)

Cliquez sur **Poursuivre le téléchargement** pour en savoir plus sur le jeu de données et sa disponibilité.

Les données résident dans un emplacement [Stockage Blob Azure](../../storage/blobs/storage-quickstart-blobs-dotnet.md) : wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/. « wasb » fait référence à l'emplacement de stockage d'objets blob Azure.

1. Les données de ce stockage d’objets blob Azure sont constituées de trois sous-dossiers de données décompressées :

   1. Le sous-dossier *raw/count/* contient les 21 premiers jours de données, de day\_00 à day\_20.
   2. Le sous-dossier *raw/train/* est constitué d’un seul jour de données, day\_21.
   3. Le sous-dossier *brut/test/* est constitué de deux jours de données, day\_22 et day\_23.
2. Les données brutes gzip sont également disponibles dans le dossier principal *raw/* en tant que day_NN.gz, où NN va de 00 à 23.

Une autre approche vous permettant d’accéder, d’explorer et de modéliser ces données ne nécessitant aucun téléchargement local est expliquée plus loin dans cette procédure pas à pas lors de la création de tables Hive.

## <a name="log-in-to-the-cluster-headnode"></a><a name="login"></a>Connexion au nœud principal du cluster
Pour vous connecter au nœud principal du cluster, utilisez le [portail Azure](https://ms.portal.azure.com) afin de localiser le cluster. Cliquez sur l’icône d’éléphant HDInsight située sur la gauche et double-cliquez ensuite sur le nom de votre cluster. Accédez à l’onglet **Configuration** , double-cliquez sur l’icône CONNECTER en bas de la page et entrez vos informations d’identification pour l’accès à distance lorsque vous y êtes invité ; vous êtes alors redirigé vers le nœud principal du cluster.

Une première connexion au nœud principal de cluster ressemble généralement à ceci :

![Connexion au cluster](./media/hive-criteo-walkthrough/Yys9Vvm.png)

Sur la gauche se trouve la « ligne de commande Hadoop », qui nous permet d’explorer les données. Notez également la présence de deux URL utiles : « État Yarn Hadoop » et « Nœud de nom Hadoop ». L'URL de l'état Yarn indique la progression de la tâche et l'URL du nœud de nom vous fournit des informations relatives à la configuration du cluster.

Vous êtes désormais prêt à entamer la première partie de la procédure pas à pas : l’exploration de données à l’aide de Hive et la préparation de données pour Azure Machine Learning.

## <a name="create-hive-database-and-tables"></a><a name="hive-db-tables"></a> Création de la base de données et des tables Hive
Pour créer des tables Hive pour le jeu de données Criteo, ouvrez la * *_ligne de commande Hadoop_* _ sur le bureau du nœud principal, puis saisissez le répertoire Hive en entrant la commande suivante :

```console
cd %hive_home%\bin
```

> [!NOTE]
> Exécutez, dans cette procédure pas à pas, toutes les commandes Hive depuis l’invite de l’emplacement/du répertoire Hive. Tout problème lié au chemin d’accès sera résolu automatiquement. Vous pouvez utiliser les termes « invite du répertoire Hive », « invite de l’emplacement/du répertoire Hive » et « ligne de commande Hadoop » de manière interchangeable.
>
> [!NOTE]
> Pour exécuter une requête Hive, il est toujours possible d’utiliser les commandes suivantes :
>
> `cd %hive_home%\bin`
> `hive`

Lorsque Hive REPL apparaît avec un signe « hive > », coupez-collez simplement la requête pour l'exécuter.

Le code suivant crée une base de données « criteo » et génère ensuite quatre tables :

_ une *table de comptage* du jour day\_00 au jour day\_20 ;
* une *table à utiliser comme jeu de données d’apprentissage* reposant sur day\_21 ; et
* deux *tables à utiliser comme jeux de données de test* reposant sur day\_22 et day\_23 respectivement.

Fractionnez le jeu de données de test en deux tables différentes, car l’un des jours est un jour férié. L’objectif est de déterminer si le modèle peut détecter les différences entre un jour férié et jour non férié à partir du taux de clics.

Le script [sample&#95;hive&#95;create&#95;criteo&#95;database&#95;and&#95;tables.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_criteo_database_and_tables.hql) est affiché ici par commodité :

```hiveql
CREATE DATABASE IF NOT EXISTS criteo;
DROP TABLE IF EXISTS criteo.criteo_count;
CREATE TABLE criteo.criteo_count (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/count';

DROP TABLE IF EXISTS criteo.criteo_train;
CREATE TABLE criteo.criteo_train (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/train';

DROP TABLE IF EXISTS criteo.criteo_test_day_22;
CREATE TABLE criteo.criteo_test_day_22 (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/test/day_22';

DROP TABLE IF EXISTS criteo.criteo_test_day_23;
CREATE TABLE criteo.criteo_test_day_23 (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/test/day_23';
```

Toutes ces tables étant externes, vous pouvez pointer vers leurs emplacements de stockage Blob Azure (wasb).

**Il existe deux manières d’exécuter une requête Hive :**

* **Utilisation de la ligne de commande Hive REPL**. La première consiste à émettre une commande « hive » et à copier-coller une requête dans la ligne de commande Hive REPL :

  ```console
  cd %hive_home%\bin
  hive
  ```

     Dans la ligne de commande REPL, coupez-collez la requête qu’elle exécute.
* **Enregistrement des requêtes dans un fichier et Exécution de la commande** : La seconde consiste à enregistrer les requêtes dans un fichier .hql ( [sample&#95;hive&#95;create&#95;criteo&#95;database&#95;and&#95;tables.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_criteo_database_and_tables.hql)), puis à utiliser la commande suivante pour exécuter la requête :

  ```console
  hive -f C:\temp\sample_hive_create_criteo_database_and_tables.hql
  ```

### <a name="confirm-database-and-table-creation"></a>Confirmation de la création de la base de données et des tables
Confirmez ensuite la création de la base de données en utilisant la commande suivante depuis l’invite de l’emplacement/du répertoire Hive :

```console
hive -e "show databases;"
```

Cela donne :

```output
criteo
default
Time taken: 1.25 seconds, Fetched: 2 row(s)
```

Ceci confirme la création de la nouvelle base de données, « criteo ».

Pour découvrir quelles tables ont été créées, exécutez simplement la commande suivante depuis l’invite de l’emplacement/du répertoire Hive :

```output
hive -e "show tables in criteo;"
```

La sortie suivante s’affiche :

```ouput
criteo_count
criteo_test_day_22
criteo_test_day_23
criteo_train
Time taken: 1.437 seconds, Fetched: 4 row(s)
```

## <a name="data-exploration-in-hive"></a><a name="exploration"></a> Exploration des données dans Hive
Vous êtes désormais prêt à effectuer une exploration de données de base dans Hive. Vous commencez par compter le nombre d’exemples dans les tables de données de formation et de test.

### <a name="number-of-train-examples"></a>Nombre d'exemples de formation
Le contenu de [sample&#95;hive&#95;count&#95;train&#95;table&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_train_table_examples.hql) est présenté ici :

```hiveql
SELECT COUNT(*) FROM criteo.criteo_train;
```

Cela donne :

```output
192215183
Time taken: 264.154 seconds, Fetched: 1 row(s)
```

Vous pouvez également exécuter la commande suivante depuis l’invite de l’emplacement/du répertoire Hive :

```console
hive -f C:\temp\sample_hive_count_criteo_train_table_examples.hql
```

### <a name="number-of-test-examples-in-the-two-test-datasets"></a>Nombre d'exemples de test dans les deux groupes de données de test
Comptez à présent le nombre d’exemples de test dans les deux jeux de données de test. Le contenu de [sample&#95;hive&#95;count&#95;criteo&#95;test&#95;day&#95;22&#95;table&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_criteo_test_day_22_table_examples.hql) est présenté ici :

```hiveql
SELECT COUNT(*) FROM criteo.criteo_test_day_22;
```

Cela donne :

```output
189747893
Time taken: 267.968 seconds, Fetched: 1 row(s)
```

Comme d’habitude, vous pouvez également appeler le script depuis l’invite de l’emplacement/du répertoire Hive en exécutant la commande suivante :

```console
hive -f C:\temp\sample_hive_count_criteo_test_day_22_table_examples.hql
```

Enfin, vous examinez le nombre d’exemples de test dans le jeu de données de test basé sur le day\_23.

La commande à exécuter est semblable à celle présentée (consultez [sample&#95;hive&#95;count&#95;criteo&#95;test&#95;day&#95;23&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_criteo_test_day_23_examples.hql)) :

```hiveql
SELECT COUNT(*) FROM criteo.criteo_test_day_23;
```

Cela donne :

```output
178274637
Time taken: 253.089 seconds, Fetched: 1 row(s)
```

### <a name="label-distribution-in-the-train-dataset"></a>Distribution d'étiquette dans le groupe de données de formation
La distribution d'étiquette dans le groupe de données de formation est intéressante. Pour la découvrir, affichez le contenu de [sample&#95;hive&#95;criteo&#95;label&#95;distribution&#95;train&#95;table.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_label_distribution_train_table.hql) :

```hiveql
SELECT Col1, COUNT(*) AS CT FROM criteo.criteo_train GROUP BY Col1;
```

Cela génère la distribution d'étiquette suivante :

```output
1       6292903
0       185922280
Time taken: 459.435 seconds, Fetched: 2 row(s)
```

Le pourcentage d’étiquettes positives est d’environ 3,3 % (cohérent avec le groupe de données d’origine).

### <a name="histogram-distributions-of-some-numeric-variables-in-the-train-dataset"></a>Les distributions de l’histogramme de certaines variables numériques dans le groupe de données de formation
Vous pouvez utiliser la fonction native « histogram\_numeric » de Hive pour découvrir à quoi ressemble la distribution des variables numériques. Voici le contenu de [sample&#95;hive&#95;criteo&#95;histogram&#95;numeric.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_histogram_numeric.hql) :

```hiveql
SELECT CAST(hist.x as int) as bin_center, CAST(hist.y as bigint) as bin_height FROM
    (SELECT
    histogram_numeric(col2, 20) as col2_hist
    FROM
    criteo.criteo_train
    ) a
    LATERAL VIEW explode(col2_hist) exploded_table as hist;
```

Cela génère le résultat suivant :

```output
26      155878415
2606    92753
6755    22086
11202   6922
14432   4163
17815   2488
21072   1901
24113   1283
27429   1225
30818   906
34512   723
38026   387
41007   290
43417   312
45797   571
49819   428
53505   328
56853   527
61004   160
65510   3446
Time taken: 317.851 seconds, Fetched: 20 row(s)
```

La combinaison LATERAL VIEW - explode dans Hive permet de générer une sortie similaire à SQL au lieu de la liste habituelle. Dans ce tableau, la première colonne correspond au centre de l’emplacement, et le second à la fréquence de l’emplacement.

### <a name="approximate-percentiles-of-some-numeric-variables-in-the-train-dataset"></a>Les centiles approximatifs de certaines variables numériques dans le groupe de données de formation
Le calcul de centiles approximatifs avec des variables numériques est également intéressant. Le «percentile\_approx» natif de Hive le fait pour nous. [sample&#95;hive&#95;criteo&#95;approximate&#95;percentiles.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_approximate_percentiles.hql) contient :

```hiveql
SELECT MIN(Col2) AS Col2_min, PERCENTILE_APPROX(Col2, 0.1) AS Col2_01, PERCENTILE_APPROX(Col2, 0.3) AS Col2_03, PERCENTILE_APPROX(Col2, 0.5) AS Col2_median, PERCENTILE_APPROX(Col2, 0.8) AS Col2_08, MAX(Col2) AS Col2_max FROM criteo.criteo_train;
```

Cela donne :

```output
1.0     2.1418600917169246      2.1418600917169246    6.21887086390288 27.53454893115633       65535.0
Time taken: 564.953 seconds, Fetched: 1 row(s)
```

De manière générale, la distribution de centiles est étroitement liée à la distribution de l’histogramme de n’importe quelle variable numérique.

### <a name="find-number-of-unique-values-for-some-categorical-columns-in-the-train-dataset"></a>Recherchez le nombre de valeurs uniques pour certaines colonnes catégorielles dans le groupe de données de formation
Pour poursuivre l’exploration de données, recherchez, pour certaines colonnes catégorielles, le nombre de valeurs uniques utilisées. Pour ce faire, affichez le contenu de [sample&#95;hive&#95;criteo&#95;unique&#95;values&#95;categoricals.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_unique_values_categoricals.hql) :

```hiveql
SELECT COUNT(DISTINCT(Col15)) AS num_uniques FROM criteo.criteo_train;
```

Cela donne :

```output
19011825
Time taken: 448.116 seconds, Fetched: 1 row(s)
```

Col15 a des valeurs uniques 19M ! L’utilisation des techniques naïves, telles que « l’encodage à chaud » pour encoder des variables catégorielles de grande dimension, est tout bonnement impossible. Une technique puissante et robuste appelée [Apprentissage à l’aide de compteurs](/archive/blogs/machinelearning/big-learning-made-easy-with-counts) est notamment expliquée et présentée pour résoudre ce problème de manière efficace.

Enfin, examinez le nombre de valeurs uniques pour d’autres colonnes catégorielles. [sample&#95;hive&#95;criteo&#95;unique&#95;values&#95;multiple&#95;categoricals.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_unique_values_multiple_categoricals.hql) contient :

```hiveql
SELECT COUNT(DISTINCT(Col16)), COUNT(DISTINCT(Col17)),
COUNT(DISTINCT(Col18), COUNT(DISTINCT(Col19), COUNT(DISTINCT(Col20))
FROM criteo.criteo_train;
```

Cela donne :

```output
30935   15200   7349    20067   3
Time taken: 1933.883 seconds, Fetched: 1 row(s)
```

Là encore, notez qu’à l’exception de Col20, toutes les autres colonnes possèdent des valeurs uniques.

### <a name="co-occurrence-counts-of-pairs-of-categorical-variables-in-the-train-dataset"></a>Nombre de cooccurrences de paires de variables catégorielles dans le jeu de données de formation

Les répartitions du nombre de paires de variables catégorielles sont également intéressantes. Il peut être déterminé à l’aide du code de [sample&#95;hive&#95;criteo&#95;paired&#95;categorical&#95;counts.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_paired_categorical_counts.hql) :

```hiveql
SELECT Col15, Col16, COUNT(*) AS paired_count FROM criteo.criteo_train GROUP BY Col15, Col16 ORDER BY paired_count DESC LIMIT 15;
```

Inversez l’ordre des nombres par leur occurrence et penchez-vous, dans ce cas, sur les 15 meilleurs. Cela nous donne :

```output
ad98e872        cea68cd3        8964458
ad98e872        3dbb483e        8444762
ad98e872        43ced263        3082503
ad98e872        420acc05        2694489
ad98e872        ac4c5591        2559535
ad98e872        fb1e95da        2227216
ad98e872        8af1edc8        1794955
ad98e872        e56937ee        1643550
ad98e872        d1fade1c        1348719
ad98e872        977b4431        1115528
e5f3fd8d        a15d1051        959252
ad98e872        dd86c04a        872975
349b3fec        a52ef97d        821062
e5f3fd8d        a0aaffa6        792250
265366bf        6f5c7c41        782142
Time taken: 560.22 seconds, Fetched: 15 row(s)
```

## <a name="down-sample-the-datasets-for-azure-machine-learning"></a><a name="downsample"></a> Réduction de l’échantillon des groupes de données pour Azure Machine Learning
Après avoir exploré les jeux de données et montré comment effectuer ce type d’exploration pour toutes les variables (y compris les combinaisons), réduisez l’échantillon des jeux de données pour pouvoir créer des modèles dans Azure Machine Learning. N’oubliez pas le problème rencontré : dans un ensemble d’exemples d’attributs (valeurs de fonctionnalité de Col2 à Col40), prédisez si Col1 est défini par la valeur 0 (aucun clic) ou la valeur 1 (clic).

Pour réduire l’échantillon des jeux de données de formation et de test à 1 % de leur taille d’origine, utilisez la fonction RAND() native de Hive. Le script suivant, [sample&#95;hive&#95;criteo&#95;downsample&#95;train&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_train_dataset.hql), effectue cette opération pour le jeu de données de formation :

```hiveql
CREATE TABLE criteo.criteo_train_downsample_1perc (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

---Now downsample and store in this table

INSERT OVERWRITE TABLE criteo.criteo_train_downsample_1perc SELECT * FROM criteo.criteo_train WHERE RAND() <= 0.01;
```

Cela donne :

```output
Time taken: 12.22 seconds
Time taken: 298.98 seconds
```

Le script [sample&#95;hive&#95;criteo&#95;downsample&#95;test&#95;day&#95;22&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_test_day_22_dataset.hql) effectue ceci pour les données de test, du day\_22 :

```hiveql
--- Now for test data (day_22)

CREATE TABLE criteo.criteo_test_day_22_downsample_1perc (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

INSERT OVERWRITE TABLE criteo.criteo_test_day_22_downsample_1perc SELECT * FROM criteo.criteo_test_day_22 WHERE RAND() <= 0.01;
```

Cela donne :

```output
Time taken: 1.22 seconds
Time taken: 317.66 seconds
```

Puis, le script [sample&#95;hive&#95;criteo&#95;downsample&#95;test&#95;day&#95;23&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_test_day_23_dataset.hql) effectue ceci pour les données de test du day\_23 :

```hiveql
--- Finally test data day_23
CREATE TABLE criteo.criteo_test_day_23_downsample_1perc (
col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 srical feature; tring)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;

INSERT OVERWRITE TABLE criteo.criteo_test_day_23_downsample_1perc SELECT * FROM criteo.criteo_test_day_23 WHERE RAND() <= 0.01;
```

Cela donne :

```output
Time taken: 1.86 seconds
Time taken: 300.02 seconds
```

Vous êtes ainsi prêt à utiliser nos jeux de données de formation et de test à échantillon réduit pour créer des modèles dans Azure Machine Learning.

Intéressons-nous à un dernier composant important avant de passer à Azure Machine Learning, qui concerne la table de comptage. Dans la sous-section suivante, la table de comptage est décrite en détail.

## <a name="a-brief-discussion-on-the-count-table"></a><a name="count"></a> Une brève discussion sur la table de comptage
Comme vous l’avez remarqué, plusieurs variables catégorielles ont une dimensionnalité élevée. Dans la procédure pas à pas, une technique puissante appelée [Apprentissage à l’aide de compteurs](/archive/blogs/machinelearning/big-learning-made-easy-with-counts) est présentée pour encoder ces variables de manière fiable et efficace. Plus d'informations sur cette technique sont indiquées dans le lien fourni.

>[!NOTE]
>Dans cette procédure pas à pas, penchez-vous sur l’utilisation de tables de comptage permettant de produire des représentations compactes de fonctionnalités catégorielles de grande dimension. Ce n’est pas la seule manière d’encoder des fonctionnalités catégorielles ; pour plus d’informations sur les autres techniques, les utilisateurs intéressés peuvent consulter les rubriques [Encodage à chaud](https://en.wikipedia.org/wiki/One-hot) et [Hachage de caractéristiques](https://en.wikipedia.org/wiki/Feature_hashing).
>

Pour créer des tables de comptage sur les données numériques, utilisez les données dans le dossier raw/count. Dans la rubrique de modélisation, les utilisateurs apprennent à créer de toutes pièces ces tables de comptage pour fonctionnalités catégorielles, ou également à utiliser une table de comptage prédéfinie pour leurs explorations. Dans la suite, lorsque nous utilisons le terme de « tables de comptage prédéfinies », nous entendons par là l’utilisation des tables de comptage fournies. Vous trouverez des instructions détaillées sur l’accès à ces tables dans la section suivante.

## <a name="build-a-model-with-azure-machine-learning"></a><a name="aml"></a> Création de modèles dans Azure Machine Learning
Notre processus de création de modèles dans Azure Machine Learning se déroule comme suit :

1. [Récupération des données des tables Hive dans Azure Machine Learning](#step1)
2. [Création de l’expérience : nettoyage des données et caractérisation des tables de comptage](#step2)
3. [Création, formation et notation du modèle](#step3)
4. [Évaluation du modèle](#step4)
5. [Publication du modèle comme service web](#step5)

Vous êtes désormais prêt à créer des modèles dans Azure Machine Learning Studio. Nos données à échantillon réduit sont enregistrées en tant que tables Hive dans le cluster. Utilisez le module **Importer des données** d’Azure Machine Learning pour lire ces données. Les informations d’identification permettant d’accéder au compte de stockage de ce cluster sont indiquées ci-après.

### <a name="step-1-get-data-from-hive-tables-into-azure-machine-learning-using-the-import-data-module-and-select-it-for-a-machine-learning-experiment"></a><a name="step1"></a> Étape 1 : Récupérer des données des tables Hive dans Azure Machine Learning à l’aide du module Importer des données et l’utiliser pour une expérience d’apprentissage automatique
Commencez par sélectionner **+NOUVEAU** -> **EXPÉRIENCE** -> **Expérience vide**. Ensuite, dans la zone **Recherche** en haut à gauche, recherchez « Importer des données ». Effectuez un glisser-déplacer du module **Importer des données** sur le canevas d’expérience (partie centrale de l’écran) afin d’utiliser le module pour l’accès aux données.

Voici à quoi ressemble le module **Importer les données** lors de la récupération des données d’une table Hive :

![Import Data obtient des données](./media/hive-criteo-walkthrough/i3zRaoj.png)

Pour le module **Importer des données** , les valeurs des paramètres qui sont fournies dans le graphique servent uniquement d’exemples de valeurs que vous devez fournir. Voici quelques instructions générales portant sur la manière de définir le jeu de paramètres pour le module **Importer des données** .

1. Choisissez « Requête Hive » pour la **source de données**
2. Dans la zone de **requête de base de données** Hive, une simple opération SELECT * FROM <votre\_base\_de\_données.votre\_table> - suffit.
3. **URI du serveur Hcatalog** : si votre cluster se nomme « abc », vous avez donc : https:\//abc.azurehdinsight.net
4. **Nom du compte utilisateur Hadoop** : nom d’utilisateur choisi lors de la mise en service du cluster. PAS le nom d'utilisateur à distance.
5. **Mot de passe du compte utilisateur Hadoop** : mot de passe pour le nom d’utilisateur choisi lors de la mise en service du cluster. PAS le mot de passe de l'accès à distance.
6. **Emplacement des données de sortie** : Choisir « Azure »
7. **Nom du compte de Stockage Azure**  : le compte de stockage associé au cluster
8. **Clé du compte de Stockage Azure**  : la clé du compte de stockage associé au cluster.
9. **Nom de conteneur Azure** : si le nom du cluster est « abc », il se nommera tout simplement « abc ».

Dès lors que le module **Importer des données** a récupéré les données (une coche verte est affichée sur le module), enregistrez-les en tant que jeu de données (avec le nom de votre choix). Cela ressemble à :

![Import Data enregistre des données](./media/hive-criteo-walkthrough/oxM73Np.png)

Cliquez avec le bouton droit sur le port de sortie du module **Importer des données** . Ceci fait apparaître une option **Enregistrer comme dataset** et une option **Visualiser**. L’option **Visualiser** , si vous cliquez dessus, fera apparaître les 100 lignes de données, ainsi qu’un panneau, situé à droite, qui sera très utile pour certaines statistiques de résumé. Pour enregistrer les données, sélectionnez simplement **Enregistrer en tant que groupe de données** et suivez les instructions.

Pour sélectionner le groupe de données enregistré et l’utiliser dans une expérience d’apprentissage automatique, recherchez les groupes de données à l’aide de la zone **Recherche** visible dans l’illustration ci-dessous. Puis tapez simplement une partie du nom attribué au groupe de données pour accéder à celui-ci et faites-le glisser sur le panneau principal. En le déposant sur le panneau principal, ce groupe de données est sélectionné pour être utilisé dans la modélisation de l’apprentissage automatique.

![Faites glisser l’ensemble de données sur le panneau principal](./media/hive-criteo-walkthrough/cl5tpGw.png)

> [!NOTE]
> Réalisez cette opération pour les groupes de données de formation et de test. En outre, n'oubliez pas d'utiliser le nom de la base de données et les noms de tables attribués à cet effet. Les valeurs de la capture d’écran sont utilisées à simple titre d’illustration.\**
>
>

### <a name="step-2-create-an-experiment-in-azure-machine-learning-to-predict-clicks--no-clicks"></a><a name="step2"></a> Étape 2 : Créer une expérience dans Azure Machine Learning pour prédire les clics effectués/non effectués
Notre expérience Azure Machine Learning Studio (classique) ressemble à ceci :

![Expérience Machine Learning](./media/hive-criteo-walkthrough/xRpVfrY.png)

Examinez maintenant les composants clés de cette expérience. Faites d’abord glisser nos jeux de données d’entraînement et de test sur le canevas de l’expérience.

#### <a name="clean-missing-data"></a>Nettoyage des données manquantes
Le module **Nettoyer des données manquantes** , comme son nom l’indique, nettoie les données manquantes via des méthodes qui peuvent être spécifiées par l’utilisateur. Regardez dans ce module pour voir ceci :

![Nettoyage des données manquantes](./media/hive-criteo-walkthrough/0ycXod6.png)

Remplacez ici toutes les valeurs manquantes par 0. D'autres options, mentionnées dans les listes déroulantes du module, sont également proposées.

#### <a name="feature-engineering-on-the-data"></a>Conception de fonctionnalités sur les données
Certaines fonctionnalités catégorielles de jeux de données volumineux peuvent rassembler des millions de valeurs uniques. L’utilisation des techniques naïves, telles que l’encodage à chaud pour représenter des fonctionnalités catégorielles de grande dimension, est tout bonnement impossible. Cette procédure pas à pas explique comment utiliser les fonctionnalités de comptage à l’aide de modules Azure Machine Learning intégrés pour générer des représentations compactes de ces variables catégorielles de grande dimension. Le résultat final est un modèle de plus petite taille, un entraînement plus rapide et des métriques de performances comparables à l’utilisation d’autres techniques.

##### <a name="building-counting-transforms"></a>Création de transformations de comptage
Pour créer des fonctionnalités de comptage, utilisez le module **Créer une transformation de comptage** qui est disponible dans Azure Machine Learning. Le module se présente ainsi :

![Créer des propriétés de module de transformation de comptage](./media/hive-criteo-walkthrough/e0eqKtZ.png)
![Créer un module de transformation de comptage](./media/hive-criteo-walkthrough/OdDN0vw.png)

> [!IMPORTANT]
> Dans la zone **Nombre de colonnes** , entrez les colonnes sur lesquelles vous souhaitez effectuer un comptage. En règle générale, il s'agit de colonnes catégorielles de grande dimension (comme indiqué). Rappelez-vous que le jeu de données Criteo possède 26 colonnes catégorielles : de Col15 à Col40. Ici, effectuez un comptage sur chacune d’elles et donnez leurs index (de 15 à 40 séparés par des virgules, comme indiqué).
>

Pour utiliser le module en mode MapReduce (adapté aux grands jeux de données), vous devez accéder à un cluster HDInsight Hadoop (celui utilisé pour l’exploration de la fonctionnalité peut être réutilisé à cet effet) et ses informations d’identification. Les figures précédentes illustrent les valeurs renseignées (remplacez les valeurs fournies à titre d’illustration avec celles adaptées à votre propre cas d’utilisation).

![Paramètres du module](./media/hive-criteo-walkthrough/05IqySf.png)

La figure ci-dessus illustre comment entrer l’emplacement de l’objet blob en entrée. Cet emplacement comporte les données réservées pour la création de tables de comptage.

Une fois l’exécution de ce module terminée, enregistrez la transformation pour une utilisation ultérieure en cliquant avec le bouton droit sur le module et en sélectionnant l’option **Enregistrer en tant que transformation** :

![Option « Enregistrer en tant que transformation »](./media/hive-criteo-walkthrough/IcVgvHR.png)

Dans notre architecture d'expérience illustrée ci-dessus, le jeu de données « ytransform2 » correspond précisément à une transformation de nombre enregistrée. Pour le reste de cette expérience, nous partons du principe que le lecteur a utilisé un module **Créer une transformation de comptage** sur certaines données pour générer des nombres et peut ensuite utiliser ces nombres pour générer des fonctionnalités de comptage sur les jeux de données de formation et de test.

##### <a name="choosing-what-count-features-to-include-as-part-of-the-train-and-test-datasets"></a>Sélection des fonctionnalités de comptage à inclure dans les jeux de données d'apprentissage et de test
Lorsqu’une transformation de comptage est prête, l’utilisateur peut choisir les fonctionnalités à inclure dans les jeux de données de formation et de test à l’aide du module **Modifier les paramètres de la table de comptage**. Par souci d’exhaustivité, ce module est présenté ici. Mais dans un souci de simplification, nous ne l’utilisons pas dans notre expérience.

![Modifier les paramètres de la table de comptage](./media/hive-criteo-walkthrough/PfCHkVg.png)

Dans ce cas, comme vous pouvez le constater, les probabilités de journalisation doivent être utilisées et la colonne d’interruption ignorée. Vous pouvez également définir des paramètres, tels que le seuil d’emplacement de la corbeille, le nombre de pseudo-exemples précédents à ajouter pour le lissage et s’il faut utiliser le bruit Laplacien ou non. Il s'agit là de fonctionnalités avancées et il est à noter que les valeurs par défaut sont un bon point de départ pour les utilisateurs qui débutent dans ce type de génération de fonctionnalité.

##### <a name="data-transformation-before-generating-the-count-features"></a>Transformation des données avant la génération des fonctionnalités de comptage
Il est maintenant temps d’aborder un point important de la transformation de nos données de formation et de test avant de générer réellement les fonctions de comptage. Deux modules **Exécuter le script R** sont utilisés avant que la transformation de comptage soit appliquée à nos données.

![Modules Exécuter le script R](./media/hive-criteo-walkthrough/aF59wbc.png)

Voici le premier script R :

![Premier script R](./media/hive-criteo-walkthrough/3hkIoMx.png)

Ce script R renomme nos colonnes « Col1 » à « Col40 ». La transformation de nombre attend des noms dans ce format.

Le deuxième script R équilibre la distribution entre les classes positives et négatives (classes 1 et 0 respectivement) en réduisant l’échantillon de la classe négative. Le script R suivant montre comment procéder :

![Deuxième script R](./media/hive-criteo-walkthrough/91wvcwN.png)

Dans ce script R simple, le « pos\_neg\_ratio » est utilisé pour définir l’équilibre entre les classes positives et négatives. Ceci est important car l’amélioration du déséquilibre des classes présente, en général, des avantages en matière de performances pour les problèmes de classification où la distribution des classes est déséquilibrée (n’oubliez pas qu’ici nous avons 3,3 % de classe positive et 96,7 % de classe négative).

##### <a name="applying-the-count-transformation-on-our-data"></a>Application de la transformation de nombre à nos données
Enfin, nous pouvons utiliser le module **Appliquer la transformation** pour appliquer les transformations de comptage à nos jeux de données de formation et de test. Ce module prend la transformation de nombre enregistrée comme une entrée et les jeux de données d'apprentissage ou de test comme l'autre entrée, et il renvoie les données avec des fonctionnalités de nombre. En voici l’illustration :

![Appliquer le module de transformation](./media/hive-criteo-walkthrough/xnQvsYf.png)

##### <a name="an-excerpt-of-what-the-count-features-look-like"></a>Un exemple de l'aspect des fonctionnalités de nombre
Il est intéressant de voir à quoi les fonctionnalités de nombre ressemblent dans notre cas. En voici un exemple :

![Fonctionnalités de comptage](./media/hive-criteo-walkthrough/FO1nNfw.png)

Cet exemple montre que pour les colonnes sur lesquelles nous avons exécuté le comptage, vous obtenez les nombres et les probabilités de journalisation, en plus des interruptions pertinentes.

Vous êtes maintenant prêt à créer un modèle Azure Machine Learning à l’aide de ces jeux de données transformés. La section suivante explique comment procéder.

### <a name="step-3-build-train-and-score-the-model"></a><a name="step3"></a> Étape 3 : Création, formation et notation du modèle

#### <a name="choice-of-learner"></a>Choix de l'apprenant
Vous devez tout d’abord choisir un apprenant. Utilisez un arbre de décision optimisé à deux classes comme apprenant. Voici les options par défaut pour cet apprenant :

![Paramètres de l’arbre de décision optimisé à deux classes](./media/hive-criteo-walkthrough/bH3ST2z.png)

Pour l’expérience, choisissez les valeurs par défaut. Les valeurs par défaut sont significatives et permettent d’obtenir rapidement des bases de référence pour les performances. Vous pouvez améliorer les performances en balayant les paramètres si vous le souhaitez, une fois que vous avez obtenu une ligne de base.

#### <a name="train-the-model"></a>Effectuer l’apprentissage du modèle
Pour la formation, appelez simplement un module **Former le modèle**. Les deux entrées dans celui-ci sont l'apprenant Arbre de décision optimisé à deux classes et notre jeu de données d'apprentissage. En voici l’illustration :

![Module de formation de modèle](./media/hive-criteo-walkthrough/2bZDZTy.png)

#### <a name="score-the-model"></a>Noter le modèle
Une fois que vous avez formé un modèle, vous êtes prêt à noter le jeu de données de test et à évaluer ses performances. Pour cela, utilisez le module **Noter le modèle** illustré ci-dessous, ainsi qu’un module **Évaluer le modèle** :

![Score Model module](./media/hive-criteo-walkthrough/fydcv6u.png)

### <a name="step-4-evaluate-the-model"></a><a name="step4"></a> Étape 4 : Évaluer le modèle
Enfin, vous devez analyser les performances du modèle. Pour les deux problèmes de classification (binaire) à deux classes, l’ASC est généralement une très bonne mesure. Pour visualiser cette courbe, raccordez le module **Scorer le modèle** à un module **Évaluer le modèle**. Cliquer sur **Visualiser** sur le module **Évaluer le modèle** génère un graphique semblable à celui-ci :

![Évaluation du modèle du module BDT](./media/hive-criteo-walkthrough/0Tl0cdg.png)

Pour des problèmes de classification binaire (à deux classes), l'aire sous la courbe (ASC) est une très bonne mesure de précision des prédictions. La section suivante présente nos résultats à l’aide de ce modèle sur notre jeu de données de test. Cliquez avec le bouton droit sur le port de sortie du module **Évaluer le modèle** et sélectionnez **Visualiser**.

![Visualisation du module Évaluer le modèle](./media/hive-criteo-walkthrough/IRfc7fH.png)

### <a name="step-5-publish-the-model-as-a-web-service"></a><a name="step5"></a> Étape 5 : Publication du modèle comme service web
La possibilité de publier un modèle Azure Machine Learning en tant que services Web avec un minimum de complications est une fonctionnalité utile qui se doit d'être largement accessible. Une fois cela fait, tout utilisateur peut effectuer des appels vers le service Web avec des données d'entrée pour lesquelles il a besoin de prédictions et le service Web utilise le modèle pour renvoyer ces prédictions.

Sélectionnez d’abord notre modèle entraîné comme objet Modèle entraîné en cliquant avec le bouton droit sur le module **Entraîner le modèle** et en utilisant l’option **Enregistrer en tant que modèle entraîné**.

Ensuite, nous devons créer des ports d’entrée et de sortie pour notre service web :

* un port d’entrée prend les données au même format que les données pour lesquelles vous avez besoin de prédictions,
* un port de sortie renvoie les Étiquettes notées et les probabilités associées.

#### <a name="select-a-few-rows-of-data-for-the-input-port"></a>Sélectionnez quelques lignes de données pour le port d'entrée
Il est pratique d'utiliser un module **Transformation Apply SQL** pour ne sélectionner que 10 lignes en tant que données de port d'entrée. Sélectionnez uniquement ces lignes de données pour notre port d’entrée à l’aide de la requête SQL indiquée ici :

![Données du port d'entrée](./media/hive-criteo-walkthrough/XqVtSxu.png)

#### <a name="web-service"></a>Service Web
Vous êtes désormais prêt à exécuter une petite expérience qui peut être utilisée pour publier notre service web.

#### <a name="generate-input-data-for-webservice"></a>Générer des données d'entrée pour le service Web
Puisque la table de comptage est volumineuse, utilisez, avant toute chose, quelques lignes de données de test et générez des données de sortie à l’aide des fonctionnalités de comptage. Cette sortie peut servir de format de données d’entrée pour notre service web, comme illustré ici :

![Création des données d'entrée BDT](./media/hive-criteo-walkthrough/OEJMmst.png)

> [!NOTE]
> Pour le format de données d’entrée, utilisez la SORTIE du module **Caractériseur de comptage**. Une fois l’expérience terminée, enregistrez la sortie du module **Caractériseur de comptage** en tant que groupe de données. Ce jeu de données sera utilisé pour les données d’entrée dans le service web.
>
>

#### <a name="scoring-experiment-for-publishing-webservice"></a>Expérience d’évaluation de la publication du service Web
Tout d’abord, la structure essentielle est un module **Scorer le modèle** qui accepte notre objet de modèle entraîné, ainsi que quelques lignes de données d’entrée que nous avons créées lors des étapes précédentes avec le module **Caractériseur de comptage**. Utilisez « Sélectionner des colonnes dans le jeu de données » pour projeter les étiquettes notées et les probabilités de notes.

![Sélectionner des colonnes dans le jeu de données](./media/hive-criteo-walkthrough/kRHrIbe.png)

Notez comment le module **Sélectionner des colonnes dans le jeu de données** peut être utilisé pour le « filtrage » des données d'un groupe de données. Le contenu est présenté ici :

![Filtrage avec le module Sélectionner des colonnes dans le jeu de données](./media/hive-criteo-walkthrough/oVUJC9K.png)

Pour obtenir les ports d’entrée et de sortie bleus, vous cliquez simplement sur **préparer le service web** , situé en bas à droite. L’exécution de cette expérience permet également de publier le service web. Pour ce faire, cliquez sur l’icône **PUBLIER LE SERVICE WEB** située en bas à droite :

![Publication du service Web](./media/hive-criteo-walkthrough/WO0nens.png)

Une fois le service web publié, vous êtes redirigé vers une page similaire à celle illustrée ci-dessous :

![Tableau de bord du service web](./media/hive-criteo-walkthrough/YKzxAA5.png)

Notez les deux liens pour les services web sur le côté gauche :

* Le service **REQUÊTE/RÉPONSE** (ou RRS) est destiné à des prédictions uniques, et a été utilisé dans cet atelier.
* Le service d' **EXÉCUTION DE LOTS** (BES) est utilisé pour les prédictions de lots et requiert que les données en entrée sur lesquelles sont effectuées des prédictions, résident dans un stockage d'objets blob Azure.

Cliquer sur le lien **REQUÊTE/RÉPONSE** nous redirige vers une page qui nous fournit du code prédéfini en C#, python et R. Ce code peut être utilisé pour appeler le service web. La clé API doit être utilisée sur cette page pour l’authentification.

Copier ce code python sur une nouvelle cellule dans l'interpréteur IPython notebook s'avère très pratique.

Voici un segment de code Python comportant la clé API appropriée.

![Code Python](./media/hive-criteo-walkthrough/f8N4L4g.png)

La clé API par défaut a été remplacée par la clé API de notre service web. Cliquez sur **Exécuter** sur cette cellule dans un IPython notebook pour générer la réponse suivante :

![Réponse IPython](./media/hive-criteo-walkthrough/KSxmia2.png)

Pour les deux exemples de test sur lesquels vous vous êtes penché dans le framework JSON du script Python, vous obtenez des réponses sous forme d’« Étiquettes scorées, de probabilités scorées ». Dans ce cas, choisissez les valeurs par défaut que le code prédéfini fournit (0 pour toutes les colonnes numériques et la chaîne « valeur » pour toutes les colonnes catégorielles).

En conclusion, notre procédure pas à pas explique comment gérer un jeu de données à grande échelle avec Azure Machine Learning. Vous avez démarré avec un téraoctet de données, créé un modèle de prévision que vous avez ensuite déployé en tant que service web dans le cloud.