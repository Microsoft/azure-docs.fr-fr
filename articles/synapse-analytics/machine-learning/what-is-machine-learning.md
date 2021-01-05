---
title: Machine Learning dans Azure Synapse Analytics
description: Vue d’ensemble des fonctionnalités de Machine Learning dans Azure Synapse Analytics.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: overview
ms.reviewer: jrasnick, garye
ms.date: 09/25/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: 906d3d28aabf8f6ecd6e04c38b4519937fa95c2b
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092157"
---
# <a name="machine-learning-capabilities-in-azure-synapse-analytics"></a>Fonctionnalités de Machine Learning dans Azure Synapse Analytics

Azure Synapse Analytics propose plusieurs fonctionnalités de Machine Learning dans Azure Synapse Analytics. Cet article fournit une vue d’ensemble de la façon dont vous pouvez recourir au Machine Learning dans le contexte d’Azure Synapse.

Cette vue d’ensemble couvre les différentes fonctionnalités de Synapse relatives au Machine Learning, du point de vue du processus de science des données.

Vous connaissez peut-être le déroulement d’un processus typique de science des données. Il s’agit d’un processus bien connu que la plupart des projets de Machine Learning suivent.

À un niveau élevé, le processus reprend les étapes suivantes :
* (Présentation de l’entreprise)
* Acquisition de données et compréhension
* Modélisation
* Déploiement et notation du modèle

Cet article décrit les fonctionnalités d’Azure Synapse Machine Learning dans différents moteurs d’analyse, dans la perspective du processus de science des données. Pour chaque étape du processus de science des données, les fonctionnalités Azure Synapse susceptibles de vous aider sont récapitulées.

## <a name="azure-synapse-machine-learning-capabilities"></a>Fonctionnalités de Machine Learning Azure Synapse

### <a name="data-acquisition-and-understanding"></a>Acquisition de données et compréhension

La plupart des projets de Machine Learning impliquent des étapes bien établies, et l’une de celles-ci consiste à accéder aux données et à les comprendre.

#### <a name="data-source-and-pipelines"></a>Sources de données et pipelines

Grâce à [Azure Data Factory](/azure/data-factory/introduction), une partie intégrée nativement d’Azure Synapse, un ensemble efficace d’outils est disponible pour l’ingestion de données et les pipelines d’orchestration de données. Cela vous permet de créer facilement des pipelines de données pour accéder aux données et les convertir dans un format qui peut être utilisé pour le Machine Learning. [En savoir plus sur les pipelines de données](/azure/data-factory/concepts-pipelines-activities?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) dans Synapse. 

#### <a name="data-preparation-and-explorationvisualization"></a>Préparation et exploration/visualisation des données

Une partie importante du processus de Machine Learning consiste à comprendre les données grâce à l’exploration et à des visualisations.

Selon l’emplacement de stockage des données, Synapse offre un ensemble d’outils pour les explorer et les préparer pour l’analyse et le Machine Learning. L’une des méthodes les plus rapides pour prendre en main l’exploration des données consiste à utiliser des pools serverless SQL ou Apache Spark, directement sur les données dans le lac de données.

* [Apache Spark for Azure Synapse](../spark/apache-spark-overview.md) offre des fonctionnalités de transformation, de préparation et d’exploration de vos données à grande échelle. Ces pools Spark proposent des outils tels que PySpark/Python, Scala et .NET pour le traitement des données à grande échelle. À l’aide de bibliothèques de visualisation puissantes, l’expérience d’exploration des données peut être améliorée pour mieux comprendre les données. [En savoir plus sur l’exploration et la visualisation des données dans Synapse à l’aide de Spark](../get-started-analyze-spark.md).

* [Les pools SQL serverless](../sql/on-demand-workspace-overview.md) offrent un moyen d’explorer les données à l’aide de TSQL directement sur le lac de données. Les pools SQL serverless proposent également des visualisations intégrées dans Synapse Studio. [En savoir plus sur l’exploration des données avec les pools SQL serverless](../get-started-analyze-sql-on-demand.md).

### <a name="modeling"></a>Modélisation

Dans Azure Synapse, l’apprentissage de modèles de Machine Learning peut être exécuté sur les pools Apache Spark avec des outils tels que PySpark/Python, Scala ou .NET.

#### <a name="train-models-on-spark-pools-with-mllib"></a>Effectuer l’apprentissage de modèles sur des pools Spark avec MLlib

L’apprentissage des modèles Machine Learning peut être effectué avec différents algorithmes et bibliothèques. [Spark MLlib](http://spark.apache.org/docs/latest/ml-guide.html) offre des algorithmes Machine Learning évolutifs qui peuvent aider à résoudre la plupart des problèmes de Machine Learning classiques. [Ce didacticiel](../spark/apache-spark-machine-learning-mllib-notebook.md) explique comment effectuer l’apprentissage d’un modèle à l’aide de MLlib dans Synapse.

En plus de MLlib, les bibliothèques populaires comme [Scikit Learn](https://scikit-learn.org/stable/) peuvent également servir à développer des modèles. Pour plus d’informations sur l’installation des bibliothèques sur les pools Synapse Spark, consultez [Gérer des bibliothèques pour Apache Spark dans Azure Synapse Analytics](../spark/apache-spark-azure-portal-add-libraries.md).

#### <a name="train-models-with-azure-machine-learning-automated-ml"></a>Entraînement des modèles avec ML automatisé d’Azure Machine Learning

Une autre façon d’effectuer l’apprentissage des modèles de Machine Learning, qui ne nécessite pas de connaissances préalables en Machine Learning, consiste à utiliser ML automatisé. [ML automatisé](/azure/machine-learning/concept-automated-ml) est une fonctionnalité qui entraîne automatiquement un ensemble de modèles de Machine Learning et permet à l’utilisateur de sélectionner le modèle le plus approprié en fonction de métriques spécifiques. Grâce à une intégration transparente à Azure Machine Learning à partir d’Azure Synapse Notebooks, les utilisateurs peuvent facilement tirer parti de ML automatisé dans Synapse avec l’authentification Azure Active Directory directe.  Cela signifie que vous devez uniquement pointer vers votre espace de travail Azure Machine Learning et que vous n’avez pas besoin d’entrer d’informations d’identification. Voici un [tutoriel sur ML automatisé](../spark/apache-spark-azure-machine-learning-tutorial.md) qui décrit comment entraîner des modèles à l’aide de ML automatisé d’Azure Machine Learning sur les pools Synapse Spark.

### <a name="model-deployment-and-scoring"></a>Déploiement et notation du modèle

Les modèles qui ont été formés soit dans Azure Synapse, soit en dehors d’Azure Synapse peuvent être facilement utilisés pour le scoring par lot. Actuellement, dans Synapse, vous pouvez exécuter le scoring par lot de deux manières.

* Vous pouvez utiliser la fonction [TSQL PREDICT](../sql-data-warehouse/sql-data-warehouse-predict.md) dans des pools Synapse SQL pour exécuter vos prédictions directement là où résident vos données. Cette fonction puissante et évolutive vous permet d’enrichir vos données sans déplacer les données de votre entrepôt de données. Une nouvelle [expérience guidée du modèle de Machine Learning dans Synapse Studio](https://aka.ms/synapse-ml-ui) a été introduite dans laquelle vous pouvez déployer un modèle ONNX à partir du registre de modèle Azure Machine Learning dans des pools Synapse SQL pour le scoring par lot à l’aide de PREDICT.

* Une autre option pour le scoring par lot des modèles de Machine Learning dans Azure Synapse consiste à tirer parti des pools Apache Spark pour Azure synapse. Selon les bibliothèques utilisées pour l’apprentissage des modèles, vous pouvez utiliser une expérience de code pour exécuter votre scoring par lot.

## <a name="next-steps"></a>Étapes suivantes

* [Bien démarrer avec Azure Synapse Analytics](../get-started.md)
* [Créer un espace de travail](../get-started-create-workspace.md)
* [Démarrage rapide : Créer un nouveau service lié Azure Machine Learning dans Synapse](quickstart-integrate-azure-machine-learning.md)
* [Tutoriel : Assistant Scoring de modèle Machine learning - Pool SQL dédié](tutorial-sql-pool-model-scoring-wizard.md)
