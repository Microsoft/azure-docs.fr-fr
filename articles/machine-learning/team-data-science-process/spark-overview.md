---
title: Science des données à l’aide de Spark sur Azure HDInsight - Team Data Science Process
description: La boîte à outils MLlib de Spark offre de nombreuses fonctionnalités de modélisation Machine Learning (ML) à cet environnement distribué.
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
ms.openlocfilehash: 1dd82fb00c55e3676929999f204eae8755671038
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93314749"
---
# <a name="overview-of-data-science-using-spark-on-azure-hdinsight"></a>Vue d’ensemble de la science des données à l’aide de Spark sur Azure HDInsight

Cet ensemble de rubriques montre comment utiliser HDInsight Spark pour effectuer des tâches de science des données courantes, telles que l’ingestion de données, la conception de fonctionnalités, la modélisation et l’évaluation de modèle. Les données utilisées sont un échantillon du jeu de données NYC Taxi Trip and Fare 2013. Les modèles conçus incluent la régression logistique, la régression linéaire, les forêts aléatoires et les arbres GBT (Gradient Boosted Tree). Les rubriques montrent également comment stocker ces modèles dans le stockage d’objets blob Azure (WASB) et comment noter et évaluer leurs performances de prédiction. D’autres rubriques plus avancées décrivent comment former des modèles par validation croisée et balayage hyperparamétrique. Cette rubrique de présentation référence également les rubriques qui décrivent comment configurer le cluster Spark dont vous avez besoin pour effectuer les étapes des procédures pas à pas fournies.

## <a name="spark-and-mllib"></a>Spark et MLlib
[Spark](https://spark.apache.org/) est une infrastructure de traitement en parallèle open source qui prend en charge le traitement en mémoire pour accroître les performances des applications d’analyse de Big Data. Le moteur de traitement Spark est élaboré pour permettre des analyses rapides, simples d’utilisation et sophistiquées. De par ses capacités de calcul distribué en mémoire, Spark constitue le choix idéal pour les algorithmes itératifs utilisés dans l’apprentissage automatique et les calculs de graphiques. [MLlib](https://spark.apache.org/mllib/) est la bibliothèque évolutive d’apprentissage automatique de Spark. Elle apporte des fonctionnalités de modélisation d’algorithme à cet environnement distribué.

## <a name="hdinsight-spark"></a>HDInsight Spark
[HDInsight Spark](../../hdinsight/spark/apache-spark-overview.md) est l’offre Azure de Spark Open Source. Elle prend également en charge les **blocs-notes Jupyter PySpark** sur le cluster Spark, qui peuvent exécuter des requêtes interactives SQL Spark pour transformer, filtrer et visualiser les données stockées dans des objets blob Azure (WASB). PySpark est l’API Python pour Spark. Les extraits de code qui fournissent les solutions et montrent les tracés pertinents permettant de visualiser les données ici s’exécutent dans des notebooks Jupyter installés sur les clusters Spark. Les étapes de modélisation dans ces rubriques contiennent du code qui montre comment former, évaluer, enregistrer et consommer chaque type de modèle.

## <a name="setup-spark-clusters-and-jupyter-notebooks"></a>Paramétrage : Clusters Spark et notebooks Jupyter
Les étapes de configuration et le code fournis dans cette procédure pas à pas concernent HDInsight Spark 1.6. Mais des notebooks Jupyter sont fournis pour les clusters HDInsight Spark 1.6 et Spark 2.0. Une description des notebooks et des liens vers ceux-ci sont fournis dans le fichier [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) correspondant au dépôt GitHub qui les contient. En outre, le code présenté ici et dans les notebooks liés est générique et doit fonctionner sur n’importe quel cluster Spark. Si vous n’utilisez pas HDInsight Spark, les étapes de configuration et de gestion de cluster peuvent être légèrement différentes de celles indiquées ici. Pour plus de commodité, voici les liens vers les Blocs-notes Jupyter pour Spark 1.6 (à exécuter dans le noyau pySpark du serveur de Bloc-notes Jupyter) et Spark 2.0 (à exécuter dans le noyau pySpark3 du serveur Jupyter Notebook) :

### <a name="spark-16-notebooks"></a>Notebooks Spark 1.6
Ces blocs-notes doivent être exécutés dans le noyau pySpark du serveur de Bloc-notes Jupyter.

- [pySpark-machine-learning-data-science-spark-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-data-exploration-modeling.ipynb) : ce fichier donne des informations sur l’exploration, la modélisation et le scoring de données suivant différents algorithmes.
- [pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb) : inclut des rubriques dans notebook #1, et le développement de modèles avec optimisation des hyperparamètres et validation croisée.
- [pySpark-machine-learning-data-science-spark-model-consumption.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb) : montre comment utiliser un modèle enregistré à l’aide de Python sur des clusters HDInsight.

### <a name="spark-20-notebooks"></a>Notebooks Spark 2.0
Ces blocs-notes doivent être exécutés dans le noyau pySpark3 du serveur de Bloc-notes Jupyter.

- [Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb) : ce fichier donne des informations sur l’exploration, la modélisation et le scoring de données dans des clusters Spark 2.0 à l’aide du jeu de données des courses et tarifs de taxi à New York décrit [ici](#the-nyc-2013-taxi-data). Ce bloc-notes peut être un bon point de départ pour explorer rapidement le code que nous avons fourni pour Spark 2.0. Pour un bloc-notes plus détaillé analysant les données sur les taxis de New York, consultez le bloc-notes suivant de cette liste. Consultez les notes à la suite de cette liste comparant ces bloc-notes.
- [Spark2.0-pySpark3_NYC_Taxi_Tip_Regression.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0_pySpark3_NYC_Taxi_Tip_Regression.ipynb) : ce fichier montre comment effectuer des opérations de data wrangling (Spark SQL et trames de données), d’exploration, de modélisation et de scoring l’aide du jeu de données des courses et tarifs de taxi à New York décrit [ici](#the-nyc-2013-taxi-data).
- [Spark2.0-pySpark3_Airline_Departure_Delay_Classification.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0_pySpark3_Airline_Departure_Delay_Classification.ipynb) : ce fichier montre comment effectuer des opérations de data wrangling (Spark SQL et trames de données), d’exploration, de modélisation et de scoring l’aide du célèbre jeu de données des départs à l’heure des compagnies aériennes de 2011 et 2012. Nous avons intégré le jeu de données de compagnies aériennes avec les données météorologiques des aéroports (par exemple, vitesse du vent, température, altitude, etc.) avant la modélisation. Ces fonctionnalités météo peuvent donc être incluses dans le modèle.

<!-- -->

> [!NOTE]
> Le jeu de données de compagnies aériennes a été ajouté aux notebooks Spark 2.0 pour mieux illustrer l’utilisation des algorithmes de classification. Consultez les liens suivants pour plus d’informations sur le jeu de données sur les compagnies aériennes et celui sur les données météorologiques :
> 
> - Données de départs prévus de compagnie aérienne : [https://www.transtats.bts.gov/ONTIME/](https://www.transtats.bts.gov/ONTIME/)
> 
> - Données météorologiques d’aéroport : [https://www.ncdc.noaa.gov/](https://www.ncdc.noaa.gov/)

<!-- -->

<!-- -->

> [!NOTE]
> L’exécution des blocs-notes Spark 2.0 sur les jeux de données « NYC taxi and airline flight delay » peut prendre 10 minutes ou plus (selon la taille de votre cluster HDI). Le premier bloc-notes de la liste ci-dessus montre de nombreux aspects de l’exploration de données, de la visualisation et de l’apprentissage du modèle ML dans un bloc-notes dont l’exécution est moins longue avec un jeu de données New York sous-échantillonné, dans lequel les fichiers de courses et de taxi ont été rassemblés au préalable : [Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb). Ce bloc-notes met beaucoup moins de temps à s’exécuter (deux ou trois minutes) et peut être un bon point de départ pour explorer rapidement le code que nous avons fourni pour Spark 2.0.

<!-- -->

Pour obtenir des conseils sur l’utilisation d’un modèle Spark 2.0 et la consommation de modèle à des fins de notation, consultez le [document Spark 1.6 sur la consommation](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb) pour obtenir un exemple des étapes requises. Pour utiliser cet exemple sur Spark 2.0, remplacez le fichier de code Python par [ce fichier](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Python/Spark2.0_ConsumeRFCV_NYCReg.py).

### <a name="prerequisites"></a>Prérequis

Les procédures ci-dessous sont relatives à Spark 1.6. Pour la version Spark 2.0, utilisez les blocs-notes décrits et référencés précédemment.

1. Vous devez avoir un abonnement Azure. Si vous n’en avez pas, consultez [Obtenir une version d’évaluation gratuite Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

2. Vous avez besoin d’un cluster Spark 1.6 pour effectuer cette procédure pas à pas. Pour en créer un, consultez les instructions fournies dans [Prise en main : Créer Apache Spark sur Azure HDInsight](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md). Vous spécifiez le type et la version du cluster à partir du menu **Sélectionner le type de cluster** .

![Configurer le cluster](./media/spark-overview/spark-cluster-on-portal.png)

<!-- -->

> [!NOTE]
> Pour une rubrique qui montre comment utiliser Scala plutôt que Python pour effectuer des tâches de processus de science des données de bout en bout, consultez [Science des données à l’aide de Scala avec Spark sur Azure](scala-walkthrough.md).
>
>

<!-- -->

> [!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]
>
>

## <a name="the-nyc-2013-taxi-data"></a>Données de NYC 2013 Taxi
Pesant environ 20 Go au format compressé (ou 48 Go au format non compressé), le jeu de données NYC Taxi Trip contient des fichiers CSV (valeurs séparées par des virgules) concernant plus de 173 millions de trajets et le prix réglé pour chacun d’entre eux. Chaque enregistrement de course inclut le lieu et l’heure d’embarquement et de débarquement, le numéro de licence (du chauffeur) rendu anonyme et le numéro de médaillon (numéro d’identification unique) du taxi. Les données portent sur toutes les courses effectuées en 2013 et sont fournies dans les deux jeux de données ci-après pour chaque mois :

1. Les fichiers CSV trip_data contiennent les détails de chaque course, comme le nombre de passagers, les points d’embarquement et de débarquement, la durée du trajet et la distance parcourue. Voici quelques exemples d’enregistrements :

   `medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

   `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

   `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

   `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

   `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

   `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

2. Les fichiers CSV trip_fare contiennent des informations sur le prix payé pour chaque trajet, comme le type de paiement, le montant, la surcharge et les taxes, les pourboires et péages, ainsi que le montant total réglé. Voici quelques exemples d’enregistrements :

   `medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

   `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

   `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

   `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

   `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

   `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

Nous avons pris un échantillon représentant 0,1 % de ces fichiers, et joint les fichiers CSV trip\_data et trip\_fare dans un jeu de données unique à utiliser comme jeu de données d’entrée pour cette procédure pas à pas. La clé unique permettant de joindre trip\_data et trip\_fare se compose des champs suivants : medallion (médaillon), hack\_licence (licence de taxi) et pickup\_datetime (date et heure d’embarquement). Chaque enregistrement du jeu de données contient les attributs suivants qui représentent un trajet NYC Taxy :

| Champ | Brève description |
| --- | --- |
| medallion |Médaillon de taxi anonymisé (id unique de taxi) |
| hack_license |Numéro de licence Hackney Transport anonymisé |
| vendor_id |ID du fournisseur de taxi |
| rate_code |Tarification du taxi NYC |
| store_and_fwd_flag |Indicateur de stockage et de transmission |
| pickup_datetime |Date et heure de départ |
| dropoff_datetime |Date et heure d’arrivée |
| pickup_hour |Heure de départ |
| pickup_week |Semaine de l’année correspondant au départ |
| weekday |Jour de la semaine (de 1 à 7) |
| passenger_count |Nombre de passagers dans un trajet en taxi |
| trip_time_in_secs |Durée du trajet en secondes |
| trip_distance |Distance du trajet en miles |
| pickup_longitude |Longitude de départ |
| pickup_latitude |Latitude de départ |
| dropoff_longitude |Longitude d’arrivée |
| dropoff_latitude |Latitude d’arrivée |
| direct_distance |Distance directe entre les emplacements de départ et d’arrivée |
| payment_type |Type de paiement (espèces, carte de crédit, etc.). |
| fare_amount |Montant du trajet |
| surcharge |Surcharge |
| mta_tax |Taxe sur le transport en métro MTA |
| tip_amount |Montant du pourboire |
| tolls_amount |Montant des péages |
| total_amount |Montant total |
| tipped |Pourboire payé (0/1 pour non ou oui) |
| tip_class |Classe de pourboire (0 : 0 $, 1 : 0-5 $ 2 : 6-10 $, 3 : 11 à 20 $, 4: 20 $ et +) |

## <a name="execute-code-from-a-jupyter-notebook-on-the-spark-cluster"></a>Exécuter le code à partir d’un notebook Jupyter sur le cluster Spark
Vous pouvez lancer le notebook Jupyter à partir du portail Azure. Recherchez votre cluster Spark sur votre tableau de bord, puis cliquez dessus pour accéder à la page de gestion de votre cluster. Pour ouvrir le bloc-notes associé au cluster Spark, cliquez sur **Tableaux de bord du cluster** -> **Jupyter Notebook** .

![Tableaux de bord des clusters](./media/spark-overview/spark-jupyter-on-portal.png)

Vous pouvez également accéder à * *_`https://CLUSTERNAME.azurehdinsight.net/jupyter`_* _ pour accéder aux Jupyter Notebooks. Remplacez la partie CLUSTERNAME de cette URL par le nom de votre propre cluster. Pour accéder aux blocs-notes, vous avez besoin du mot de passe de votre compte d’administrateur.

![Parcourez Jupyter Notebooks](./media/spark-overview/spark-jupyter-notebook.png)

Sélectionnez PySpark pour afficher un répertoire contenant quelques exemples de notebooks préconfigurés qui utilisent l’API PySpark. Les notebooks contenant les exemples de code de cette suite de rubriques Spark sont disponibles sur [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark)

Vous pouvez télécharger les notebooks directement de [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark) sur le serveur de notebooks Jupyter de votre cluster Spark. Sur la page d’accueil de votre Jupyter, cliquez sur le bouton _ *Chargement* * dans la partie droite de l’écran. Cette action ouvre un explorateur de fichiers. Vous pouvez coller l’URL GitHub (contenu brut) du notebook et cliquer sur **Ouvrir**.

Le nom de fichier réapparaît dans votre liste de fichiers Jupyter avec un bouton **Charger**. Cliquez sur ce bouton **Télécharger** . Le bloc-notes a été importé. Répétez ces étapes pour télécharger les autres blocs-notes de cette procédure.

> [!TIP]
> Vous pouvez cliquer avec le bouton droit sur les liens de votre navigateur, puis sélectionner **Copier le lien** pour obtenir l’URL du contenu brut de GitHub. Vous pouvez coller celle-ci dans la boîte de dialogue Charger de l’Explorateur de fichiers de Jupyter.
> 
> 

Vous pouvez désormais :

* Consultez le code en cliquant sur le bloc-notes.
* Exécutez chaque cellule en appuyant sur **MAJ-ENTRÉE**.
* Exécutez le bloc-notes entier en cliquant sur **Cellule** -> **Exécuter**.
* Utilisez la visualisation automatique des requêtes.

> [!TIP]
> Le noyau Pyspark visualise automatiquement la sortie des requêtes SQL (HiveQL). Vous pouvez sélectionner différents types de visualisations (tables, secteurs, lignes, zones ou barres) à l’aide des boutons de menu **Type** dans le notebook :
>
>

![Courbe ROC de régression logistique pour une approche générique](./media/spark-overview/pyspark-jupyter-autovisualization.png)

## <a name="whats-next"></a>Quelle est l’étape suivante ?
Maintenant que vous avez configuré un cluster HDInsight Spark et téléchargé les blocs-notes Jupyter, vous êtes prêt à appliquer les procédures correspondant aux trois blocs-notes PySpark. Elles montrent comment explorer vos données, puis créer et utiliser des modèles. Le bloc-notes d’exploration et de modélisation avancées des données montre comment inclure une validation croisée, un balayage hyperparamétrique et une évaluation du modèle.

**Exploration et modélisation des données avec Spark :** explorez le jeu de données et créez, notez et évaluez les modèles de Machine Learning, en procédant de la manière décrite dans la rubrique [Créer des modèles de classification binaire et de régression pour des données avec la boîte à outils Spark MLlib](spark-data-exploration-modeling.md).

**Utilisation des modèles :** pour découvrir comment évaluer les modèles de classification et de régression créés dans cette rubrique, consultez la page [Noter et évaluer des modèles de Machine Learning créés avec Spark](spark-model-consumption.md).

**Validation croisée et balayage hyperparamétrique** : pour savoir comment entraîner des modèles avec la validation croisée et le balayage hyperparamétrique, voir [Exploration et modélisation avancées des données avec Spark](spark-advanced-data-exploration-modeling.md).