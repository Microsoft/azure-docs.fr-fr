---
title: Vue d’ensemble du Machine Learning - Azure HDInsight
description: Vue d’ensemble des options de Machine Learning Big Data des clusters dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/06/2019
ms.openlocfilehash: a3c050b414e322f08fd45a569a22cea2db591b99
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91302687"
---
# <a name="machine-learning-on-hdinsight"></a>Machine Learning sur HDInsight

HDInsight autorise un Machine Learning sur le Big Data, en permettant d’extraire de précieuses informations de grandes quantités (pétaoctets voire exaoctets) de structurées et non structurées, ainsi que de données à déplacement rapide. Il existe plusieurs options de machine learning dans HDInsight :  SparkML et Apache Spark MLlib, R, Apache Hive et Microsoft Cognitive Toolkit.

## <a name="sparkml-and-mllib"></a>SparkML et MLlib

[HDInsight Spark](spark/apache-spark-overview.md) est une offre d’[Apache Spark](https://spark.apache.org/) hébergée sur Azure, une infrastructure open source de traitement parallèle des données qui utilise le traitement en mémoire pour améliorer les performances des analyses de Big Data. Le moteur de traitement Spark est élaboré pour permettre des analyses rapides, simples d’utilisation et sophistiquées. De par ses capacités de calcul distribué en mémoire, Spark constitue le choix idéal pour les algorithmes itératifs utilisés dans l’apprentissage automatique et les calculs de graphiques. Il existe deux bibliothèques de machine learning scalables qui apportent des fonctionnalités de modélisation d’algorithme à cet environnement distribué : MLlib et SparkML. MLib contient l’API d’origine qui vient au-dessus des RDD. SparkML est un package plus récent qui fournit une API de niveau supérieur reposant sur des trames de données pour construire des pipelines ML. SparkML ne prend pas en charge toutes les fonctionnalités de MLlib, mais remplace MLlib en tant que bibliothèque de Machine Learning standard de Spark.

La bibliothèque Microsoft Machine Learning pour Apache Spark est [MMLSpark](https://github.com/Azure/mmlspark). Cette bibliothèque est conçue pour améliorer la productivité des chercheurs de données sur Spark, accroître le taux d’expérimentation et pour tirer parti des techniques de Machine Learning de pointe, notamment l’apprentissage profond, sur des jeux de données très volumineux. MMLSpark crée une couche au-dessus des API de bas niveau de SparkML lors de la création de modèles ML évolutifs (comme des chaînes d’indexation), lors du formatage de données dans une disposition compatibles avec les algorithmes de Machine Learning et lors de l’assemblage de vecteurs de caractéristiques. La bibliothque MMLSpark simplifie ces opérations ainsi que d’autres tâches courantes permettant de créer des modèles dans PySpark.

## <a name="r"></a>R

[R](https://www.r-project.org/) est actuellement le langage de programmation statistique le plus populaire au monde. C’est un outil de visualisation des données open source, dont la communauté compte plus de 2,5 millions d’utilisateurs en pleine croissance. Avec sa base d’utilisateurs en plein essor et plus de 8 000 packages créés collectivement, R est un choix probable pour de nombreuses entreprises qui ont besoin du Machine Learning. Vous pouvez créer un cluster HDInsight avec ML Services prêt à être utilisé avec des modèles et des jeux de données volumineux. Cette fonctionnalité fournit aux chercheurs de données et aux statisticiens une interface R familière et évolutive à la demande via HDInsight, sans les tâches fastidieuses de création et de maintenance de cluster.

![Formation à la prédiction avec R Server](./media/hdinsight-machine-learning-overview/training-for-prediction.png)

Le nœud de périmètre d’un cluster fournit un lieu d’accueil pratique pour la connexion au cluster et l’exécution de vos scripts R.  Vous pouvez également exécuter des scripts R sur les différents nœuds du cluster à l’aide des contextes de calcul Hadoop Map Reduce ou Spark de ScaleR.

ML Services sur HDInsight avec Spark vous permet de paralléliser la formation sur les nœuds d’un cluster en utilisant un contexte de calcul Spark. Vous pouvez exécuter des scripts R directement sur le nœud de périphérie, grâce à tous les cœurs disponibles en parallèle, en fonction des besoins. L’autre solution consiste à exécuter votre code à partir du nœud de périphérie pour amorcer le traitement qui est réparti entre tous les nœuds du cluster. ML Services sur HDInsight avec Spark vous permet aussi de paralléliser les fonctions de packages R open source, si vous le souhaitez.

## <a name="azure-machine-learning-and-apache-hive"></a>Azure Machine Learning et Apache Hive

Azure Machine Learning fournit des outils de modélisation d’analyses prédictives et un service complètement managé vous permettant de déployer vos modèles prédictifs sous la forme de services web prêts à l’emploi. Azure Machine Learning est une solution complète d’analyse prédictive sur le cloud, que vous pouvez utiliser pour créer, tester, mettre en service et gérer rapidement des modèles prédictifs. Effectuez une sélection dans une volumineuse bibliothèque d’algorithmes, utilisez un studio web pour générer des modèles et déployez facilement votre modèle en tant que service web.

![Vue d’ensemble de Microsoft Azure Machine Learning](./media/hdinsight-machine-learning-overview/azure-machine-learning.png)

Créez des fonctionnalités pour les données dans un cluster HDInsight Hadoop à l’aide de [requêtes Hive](../machine-learning/team-data-science-process/create-features-hive.md). La *conception de fonctionnalités* tente d’augmenter la puissance prédictive des algorithmes d’apprentissage en créant des fonctionnalités à partir des données brutes qui facilitent le processus d’apprentissage. Vous pouvez exécuter des requêtes HiveQL à partir d’Azure Machine Learning Studio (Classic) et accéder aux données traitées dans Hive et stockées dans le stockage d’objets blob à l’aide du [module Importer des données](../machine-learning/classic/import-data.md).

## <a name="microsoft-cognitive-toolkit"></a>Microsoft Cognitive Toolkit

L’[apprentissage profond](https://www.microsoft.com/en-us/research/group/dltc/) est une branche du Machine Learning qui utilise les réseaux neuronaux profonds, à l’image des processus biologiques du cerveau humain. De nombreux chercheurs considèrent l’apprentissage profond comme une approche prometteuse de l’intelligence artificielle. Les traducteurs de langue parlée, les systèmes de reconnaissance d’image et le raisonnement automatique sont quelques exemples d’apprentissage profond.

Dans le cadre de son travail sur l’apprentissage profond, Microsoft a développé [Microsoft Cognitive Toolkit](https://www.microsoft.com/en-us/cognitive-toolkit/), un outil open source gratuit et facile à utiliser. Cette boîte à outils est largement utilisée par un grand nombre de produits Microsoft, par des entreprises du monde entier qui doivent déployer l’apprentissage profond à grande échelle ainsi que par des étudiants intéressés par les techniques et les algorithmes les plus récents.

## <a name="see-also"></a>Voir aussi

### <a name="scenarios"></a>Scénarios

* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour l’analyse de la température de bâtiments à l’aide des données des systèmes HVAC](spark/apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark avec Machine Learning : utiliser Spark dans HDInsight pour prédire les résultats de l’inspection d’aliments](spark/apache-spark-machine-learning-mllib-ipython.md)
* [Générer des recommandations de films avec Apache Mahout](hadoop/apache-hadoop-mahout-linux-mac.md)
* [Apache Hive et Azure Machine Learning](../machine-learning/team-data-science-process/create-features-hive.md)
* [Apache Hive et Azure Machine Learning de bout en bout](../machine-learning/team-data-science-process/hive-walkthrough.md)
* [Machine Learning avec Apache Spark sur HDInsight](../machine-learning/team-data-science-process/spark-overview.md)

### <a name="deep-learning-resources"></a>Ressources d’apprentissage profond

* [Utiliser le modèle de formation approfondie Microsoft Cognitive Toolkit avec un cluster Azure HDInsight Spark](spark/apache-spark-microsoft-cognitive-toolkit.md)
* [Infrastructures de Deep Learning et AI sur la Data Science Virtual Machine (DSVM)](../machine-learning/data-science-virtual-machine/dsvm-tools-deep-learning-frameworks.md)
