---
title: Analytique sur HDInsight Spark avec PySpark, Scala - Team Data Science Process
description: Exemples du processus TDSP (Team Data Science Process) qui montrent comment utiliser PySpark et Scala sur Azure HDInsight Spark.
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
ms.openlocfilehash: 1662af6cd3499fdf851d4e1bd8a0db48da7635b4
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93320139"
---
# <a name="hdinsight-spark-data-science-walkthroughs-using-pyspark-and-scala-on-azure"></a>Guides de la science des données HDInsight Spark avec PySpark et Scala sur Azure

Ces guides utilisent PySpark et Scala sur un cluster Azure Spark pour effectuer des analyses prédictives. Ils suivent les étapes décrites dans le processus TDSP (Team Data Science Process). Vous trouverez une vue d’ensemble du processus TDSP sur la page [Processus de science des données](overview.md) Vous trouverez une vue d’ensemble de Spark sur HDInsight sur la page [Présentation de Spark sur HDInsight](../../hdinsight/spark/apache-spark-overview.md).

D’autres guides de la science des données qui appliquent le processus TDSP sont regroupés en fonction de la **plateforme** qu’ils utilisent. Consultez [Procédures pas à pas du processus TDSP (Team Data Science Process)](walkthroughs.md) pour connaître les détails de ces exemples.

## <a name="predict-taxi-tips-using-pyspark-on-azure-spark"></a>Prédire les pourboires laissés aux taxis avec PySpark sur Azure Spark

Avec les données sur les taxis de New York, le guide [Utiliser Spark sur Azure HDInsight](spark-overview.md) prédit si un pourboire sera laissé, ainsi que la distribution attendue des montants. Cet exemple utilise le processus TDSP (Team Data Science Process) dans un scénario qui s’appuie sur un [cluster Azure HDInsight Spark](https://azure.microsoft.com/services/hdinsight/) pour stocker et explorer des données, et effectuer du feature engineering à partir du jeu de données public de courses et de tarifs des taxis new-yorkais. Cette rubrique de vue d’ensemble utilise un cluster HDInsight Spark et des notebooks Jupyter PySpark. Ceux-ci indiquent comment explorer les données, puis créer et utiliser des modèles. Le bloc-notes d’exploration et de modélisation avancées des données montre comment inclure une validation croisée, un balayage hyperparamétrique et une évaluation du modèle.

### <a name="data-exploration-and-modeling-with-spark"></a>Exploration et modélisation des données avec Spark 
explorez le jeu de données et créez, notez et évaluez les modèles de Machine Learning, en procédant de la manière décrite dans la rubrique [Créer des modèles de classification binaire et de régression pour des données avec la boîte à outils Spark MLlib](spark-data-exploration-modeling.md).

### <a name="model-consumption"></a>Utilisation des modèles
pour découvrir comment évaluer les modèles de classification et de régression créés dans cette rubrique, consultez la page [Noter et évaluer des modèles de Machine Learning créés avec Spark](spark-model-consumption.md).

### <a name="cross-validation-and-hyperparameter-sweeping"></a>Validation croisée et balayage hyperparamétrique
consultez la page [Exploration et modélisation avancées des données avec Spark](spark-advanced-data-exploration-modeling.md) pour savoir comment entraîner les modèles à l’aide de la validation croisée et du balayage hyperparamétrique.


## <a name="predict-taxi-tips-using-scala-on-azure-spark"></a>Prédire les pourboires laissés aux taxis avec Scala sur Azure Spark

Le guide [Utiliser Scala avec Spark sur Azure](scala-walkthrough.md) prédit si un pourboire sera laissé, ainsi que la distribution attendue des montants. Il montre comment utiliser Scala pour les tâches de Machine Learning supervisé avec la bibliothèque de Machine Learning (MLlib) Spark et les packages SparkML sur un cluster Azure HDInsight Spark. Elle vous guide à travers les tâches qui constituent le [processus de science des données](./index.yml): ingestion et exploration des données, visualisation, conception de fonctionnalités et consommation de modèles. Les modèles conçus incluent la régression logistique, la régression linéaire, les forêts aléatoires et les arbres GBT (Gradient Boosted Tree).


## <a name="next-steps"></a>Étapes suivantes

Pour une vue d’ensemble du processus TDSP (Team Data Science Process), consultez [Vue d’ensemble du processus TDSP (Team Data Science Process)](overview.md).

Pour en savoir plus sur le cycle de vie du processus TDSP (Team Data Science Process), consultez [Cycle de vie du processus TDSP (Team Data Science Process)](lifecycle.md). Ce cycle de vie présente les étapes, du début à la fin, que les projets suivent généralement lorsqu’ils sont exécutés.