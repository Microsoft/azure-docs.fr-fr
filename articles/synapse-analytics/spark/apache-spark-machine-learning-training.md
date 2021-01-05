---
title: Effectuer l’apprentissage de modèles Machine Learning avec Apache Spark
description: Utiliser Apache Spark dans Azure Synapse Analytics pour effectuer l’apprentissage de modèles Machine Learning
author: midesa
ms.author: midesa
ms.reviewer: euang
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 09/13/2020
ms.openlocfilehash: 8250f7154cfb2739d065980284da67a200680d89
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95919077"
---
# <a name="train-machine-learning-models"></a>Effectuer l’apprentissage de modèles Machine Learning
Apache Spark dans Azure Synapse Analytics permet l’apprentissage automatique avec le Big Data, en offrant la possibilité d’extraire de précieuses informations de grandes quantités de données structurées, non structurées et à déplacement rapide. Il existe plusieurs options pour effectuer l’apprentissage de modèles Machine Learning à l’aide d’Azure Spark dans Azure Synapse Analytics : Apache Spark MLlib, Azure Machine Learning et diverses autres bibliothèques open source. 

## <a name="apache-sparkml-and-mllib"></a>Apache SparkML et MLlib
Apache Spark dans Azure Synapse Analytics est l’une des implémentations par Microsoft d’Apache Spark dans le cloud. Il fournit une infrastructure de traitement de données parallèle, open source et unifiée qui prend en charge le traitement en mémoire pour améliorer l’analytique du Big Data. Le moteur de traitement Spark est élaboré pour permettre des analyses rapides, simples d’utilisation et sophistiquées. De par ses capacités de calcul distribué en mémoire, Spark constitue le choix idéal pour les algorithmes itératifs utilisés dans l’apprentissage automatique et les calculs de graphiques. 

Il existe deux bibliothèques Machine Learning évolutives, qui offrent des fonctionnalités de modélisation d’algorithme à cet environnement distribué : MLlib et SparkML. MLib contient l’API d’origine qui vient au-dessus des RDD. SparkML est un package plus récent qui fournit une API de niveau supérieur reposant sur des trames de données pour construire des pipelines ML. SparkML ne prend pas en charge toutes les fonctionnalités de MLlib, mais remplace MLlib en tant que bibliothèque de Machine Learning standard de Spark.

> [!NOTE]
> 
> Vous pouvez en savoir plus sur la création d’un modèle SparkML en suivant ce [tutoriel](../spark/apache-spark-azure-machine-learning-tutorial.md).

## <a name="popular-libraries"></a>Bibliothèques populaires
Chaque pool Apache Spark dans Azure Synapse Analytics est fourni avec un ensemble de bibliothèques Machine Learning préchargées et populaires. Ces bibliothèques fournissent du code réutilisable que vous pouvez inclure dans vos programmes ou projets. Parmi les bibliothèques de Machine Learning pertinentes qui sont incluses par défaut, citons :
- [Scikit-learn](https://scikit-learn.org/stable/index.html) est l’une des bibliothèques les plus populaires de Machine Learning à nœud unique pour les algorithmes classiques de ML. Scikit-learn prend en charge la plupart des algorithmes d’apprentissage supervisés et non supervisés et peut également être utilisée pour l’exploration et l’analyse des données.
  
- [XGBoost](https://xgboost.readthedocs.io/en/latest/) est une bibliothèque populaire de Machine Learning qui contient des algorithmes optimisés pour la formation des arbres de décision et des forêts aléatoires. 
  
- [PyTorch](https://pytorch.org/) et [Tensorflow](https://www.tensorflow.org/) sont des bibliothèques puissantes de Deep Learning en Python. Au sein d’un pool Apache Spark dans Azure Synapse Analytics, vous pouvez utiliser ces bibliothèques pour construire des modèles de machine unique en fixant à zéro le nombre d’Exécuteurs sur votre pool. Même si Apache Spark ne fonctionne pas dans le cadre de cette configuration, il s’agit d’un moyen simple et économique de créer des modèles à une seule machine.

Pour en savoir plus sur les bibliothèques disponibles et les versions associées, consultez le [runtime Azure Synapse Analytics](../spark/apache-spark-version-support.md) publié.

## <a name="mmlspark"></a>MMLSpark
La bibliothèque Microsoft Machine Learning pour Apache Spark est [MMLSpark](https://github.com/Azure/mmlspark). Cette bibliothèque est conçue pour améliorer la productivité des scientifiques des données sur Spark, accroître le taux d’expérimentation et tirer parti des techniques de Machine Learning de pointe, notamment le Deep Learning, sur des jeux de données volumineux. 

MMLSpark crée une couche au-dessus des API de bas niveau de SparkML lors de la création de modèles ML évolutifs (comme des chaînes d’indexation), lors du formatage de données dans une disposition compatibles avec les algorithmes de Machine Learning et lors de l’assemblage de vecteurs de caractéristiques. La bibliothque MMLSpark simplifie ces opérations ainsi que d’autres tâches courantes permettant de créer des modèles dans PySpark.

## <a name="automated-ml-in-azure-machine-learning"></a>Machine learning automatisé dans Azure Machine Learning 
Azure Machine Learning est un environnement informatique qui vous permet d’effectuer l’apprentissage, le déploiement, l’automatisation, la gestion et le suivi des modèles Machine Learning. Le Machine Learning automatisé (AutoML) dans Azure Machine Learning accepte les données d’apprentissage et les paramètres de configuration, et itère automatiquement via des combinaisons de méthodes de normalisation/standardisation des caractéristiques, des modèles et des configurations d’hyperparamètres afin d’obtenir le meilleur modèle. 

Lors de l’utilisation d’AutoML dans Azure Synapse Analytics, vous pouvez tirer parti de l’intégration profonde entre les différents services afin de simplifier l’authentification et la formation des modèles. 

> [!NOTE]
> 
> Pour en savoir plus sur la création d’une expérience Azure Machine Learning AutoML, suivez ce [tutoriel](./spark/../apache-spark-azure-machine-learning-tutorial.md).

## <a name="azure-cognitive-services"></a>Azure Cognitive Services
[Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services) fournit des capacités d’apprentissage automatique pour résoudre des problèmes généraux tels que l’analyse de texte pour inférer des sentiments ou l’analyse d’images pour reconnaître des objets ou des visages. Pour utiliser ces services, vous n’avez pas besoin de connaissances spéciales en matière de Machine Learning ou de science des données. Un service cognitif fournit tout ou partie des composants d’une solution de Machine Learning : données, algorithme et modèle formé. Ces services requièrent une connaissance générale de vos données mais ne nécessitent aucune expérience en Machine Learning ou en science des données. Vous pouvez exploiter ces services cognitifs préformés automatiquement au sein d’Azure Synapse Analytics.

## <a name="next-steps"></a>Étapes suivantes
Cet article fournit une vue d’ensemble des différentes options d’apprentissage des modèles Machine Learning dans les pools Apache Spark dans Azure Synapse Analytics. Vous pouvez en savoir plus sur la formation des modèles en suivant le tutoriel ci-dessous :

- Exécuter des expériences ML automatisées à l’aide d’Azure Machine Learning et Azure Synapse Analytics : [Tutoriel AutoML](./spark/../apache-spark-autoscale.md) 
- Exécuter des expériences SparkML : [Tutoriel Apache SparkML](../spark/apache-spark-azure-machine-learning-tutorial.md)
- Afficher les bibliothèques par défaut : [Runtime Azure Synapse Analytics](../spark/apache-spark-version-support.md)