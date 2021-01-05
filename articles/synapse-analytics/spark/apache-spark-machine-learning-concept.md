---
title: Machine Learning avec Apache Spark
description: Cet article fournit une vue d’ensemble conceptuelle des capacités de Machine Learning et de science des données disponibles via Apache Spark sur Azure Synapse Analytics.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 11/13/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.openlocfilehash: ced78955c71f37a5a6f5231f61e5327428834f00
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95918968"
---
# <a name="machine-learning-with-apache-spark"></a>Machine Learning avec Apache Spark

Apache Spark dans Azure Synapse Analytics permet l’apprentissage automatique avec le Big Data, en offrant la possibilité d’extraire de précieuses informations de grandes quantités de données structurées, non structurées et à déplacement rapide. 

Cette section inclut une vue d’ensemble et des tutoriels pour les workflows Machine Learning, notamment l’analyse exploratoire des données, l’ingénierie des caractéristiques, la formation du modèle, le scoring du modèle et le déploiement.  

## <a name="synapse-runtime"></a>Runtime Synapse 
Le runtime Synapse est un environnement organisé qui est optimisé pour la science des données et le Machine Learning. Le runtime Synapse offre par défaut une gamme de builds et de bibliothèques open source populaires dans le Kit de développement logiciel (SDK) Azure Machine Learning. Le runtime Synapse inclut également de nombreuses bibliothèques externes, notamment PyTorch, Scikit-learn, XGBoost, etc.

Pour en savoir plus sur les bibliothèques disponibles et les versions associées, consultez le [runtime Azure Synapse Analytics](../spark/apache-spark-version-support.md) publié.

## <a name="exploratory-data-analysis"></a>Analyse exploratoire des données
Lors de l’utilisation d’Apache Spark dans Azure Synapse Analytics, il existe différentes options intégrées pour vous aider à visualiser vos données, notamment des options de graphique pour notebook Synapse, l’accès aux bibliothèques open source populaires telles que Seaborn et Matplotlib, ainsi que l’intégration à Synapse SQL et à Power BI.

Pour en savoir plus sur les options de visualisation et d’analyse des données, consultez l’article sur la [visualisation des données à l’aide des notebooks Azure Synapse](../spark/apache-spark-data-visualization.md).

## <a name="feature-engineering"></a>Ingénierie des caractéristiques
Par défaut, le runtime Synapse comprend un ensemble de bibliothèques couramment utilisées pour l’ingénierie des caractéristiques. Pour les jeux de données volumineux, vous pouvez utiliser Spark SQL, MLlib et Koalas pour l’ingénierie des caractéristiques. Pour les jeux de données plus petits, les bibliothèques tierces comme Numpy, Pandas et Scikit-learn fournissent également des méthodes utiles pour ces scénarios.

## <a name="train-models"></a>Entraîner des modèles
Il existe plusieurs options pour effectuer l’apprentissage de modèles Machine Learning à l’aide d’Azure Spark dans Azure Synapse Analytics : Apache Spark MLlib, Azure Machine Learning et diverses autres bibliothèques open source. 

Pour plus d’informations sur les capacités de Machine Learning, consultez l’article sur la [formation des modèles dans Azure Synapse Analytics](../spark/apache-spark-machine-learning-training.md).

### <a name="sparkml-and-mllib"></a>SparkML et MLlib
De par ses capacités de calcul distribué en mémoire, Spark constitue le choix idéal pour les algorithmes itératifs utilisés dans l’apprentissage automatique et les calculs de graphiques. ```spark.ml``` fournit un ensemble uniforme d’API de haut niveau qui aident les utilisateurs à créer et à régler des pipelines Machine Learning. Pour en savoir plus sur ```spark.ml```, vous pouvez consulter le [guide de programmation ML Apache Spark](https://spark.apache.org/docs/1.2.2/ml-guide.html).

### <a name="azure-machine-learning-automl"></a>Azure Machine Learning AutoML
[Azure Machine Learning AutoML](https://docs.microsoft.com/azure/machine-learning/concept-automated-ml) (Machine Learning automatisé) permet d’automatiser le processus de développement de modèles Machine Learning. Il permet aux chercheurs de données, analystes et développeurs de créer des modèles ML à grande échelle, efficaces et productifs, tout en maintenant la qualité du modèle. Les composants permettant de faire fonctionner le Kit de développement logiciel (SDK) Azure Machine Learning AutoML sont directement générés dans le runtime Synapse.

### <a name="open-source-libraries"></a>Bibliothèques open source
Chaque pool Apache Spark dans Azure Synapse Analytics est fourni avec un ensemble de bibliothèques Machine Learning préchargées et populaires.  Parmi les bibliothèques de Machine Learning pertinentes qui sont incluses par défaut, citons :

- [Scikit-learn](https://scikit-learn.org/stable/index.html) est l’une des bibliothèques les plus populaires de Machine Learning à nœud unique pour les algorithmes classiques de ML. Scikit-learn prend en charge la plupart des algorithmes d’apprentissage supervisés et non supervisés et peut également être utilisée pour l’exploration et l’analyse des données.
  
- [XGBoost](https://xgboost.readthedocs.io/en/latest/) est une bibliothèque populaire de Machine Learning qui contient des algorithmes optimisés pour la formation des arbres de décision et des forêts aléatoires. 
  
- [PyTorch](https://pytorch.org/) et [Tensorflow](https://www.tensorflow.org/) sont des bibliothèques puissantes de Deep Learning en Python. Au sein d’un pool Apache Spark dans Azure Synapse Analytics, vous pouvez utiliser ces bibliothèques pour construire des modèles de machine unique en fixant à zéro le nombre d’Exécuteurs sur votre pool. Même si Apache Spark ne fonctionne pas dans le cadre de cette configuration, il s’agit d’un moyen simple et économique de créer des modèles à une seule machine.

## <a name="track-model-development"></a>Suivre le développement d’un modèle
[MLFlow](https://www.mlflow.org/) est une bibliothèque open source permettant de gérer le cycle de vie de vos expériences de Machine Learning. MLFlow Tracking est un composant de MLflow qui consigne et assure le suivi de vos métriques d'exécution d'apprentissage et de vos artefacts de modèle. Pour en savoir plus sur la façon dont vous pouvez utiliser MLFlow Tracking via Azure Synapse Analytics et Azure Machine Learning, consultez ce tutoriel sur [la façon d’utiliser MLFlow](https://docs.microsoft.com/azure/machine-learning/how-to-use-mlflow).

## <a name="model-scoring"></a>Scoring du modèle
Le scoring de modèle, ou inférence, est la phase au cours de laquelle un modèle est utilisé pour effectuer des prédictions. Pour le scoring de modèles avec SparkML ou MLlib, vous pouvez tirer parti des méthodes Spark natives pour effectuer directement l’inférence sur un DataFrame Spark. Pour les autres bibliothèques open source et types de modèles, vous pouvez également créer une FDU Spark pour effectuer un scale-out de l’inférence sur des jeux de données volumineux. Pour les jeux de données plus petits, vous pouvez également utiliser les méthodes d’inférence de modèle natif fournies par la bibliothèque.

## <a name="register-and-serve-models"></a>Inscrire et mettre en service des modèles
L’inscription d’un modèle vous permet de stocker, de gérer la version et d’effectuer le suivi des métadonnées sur les modèles dans votre espace de travail. Une fois que vous avez terminé la formation de votre modèle, vous pouvez inscrire votre modèle au [registre des modèles Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/concept-model-management-and-deployment#register-package-and-deploy-models-from-anywhere). Une fois inscrits, les modèles ONNX peuvent également être utilisés pour [enrichir les données](../machine-learning/tutorial-sql-pool-model-scoring-wizard.md) stockées dans des pools SQL dédiés.

## <a name="next-steps"></a>Étapes suivantes
Pour vous familiariser avec le Machine Learning dans Azure Synapse Analytics, consultez les tutoriels suivants :
- [Analyser des données avec Azure Synapse Notebooks](../spark/apache-spark-data-visualization-tutorial.md)

- [Effectuer l’apprentissage d’un modèle Machine Learning avec AutoML](../spark/apache-spark-azure-machine-learning-tutorial.md)

- [Effectuer l’apprentissage d’un modèle Machine Learning avec Apache Spark MLlib](../spark/apache-spark-machine-learning-mllib-notebook.md)
  