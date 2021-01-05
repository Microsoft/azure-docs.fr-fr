---
title: Présentation d’Azure Machine Learning
description: Azure Machine Learning est une solution de science des données intégrée pour scientifiques des données et MLops, qui permet de modéliser et de déployer des applications ML à l’échelle du cloud.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: overview
author: j-martens
ms.author: jmartens
ms.date: 11/04/2019
ms.custom: devx-track-python
ms.openlocfilehash: 85080c018749f9aab716eff0a4527363da987007
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620475"
---
# <a name="what-is-azure-machine-learning"></a>Qu'est-ce que Azure Machine Learning ?

Dans cet article, vous allez découvrir Azure Machine Learning, un environnement basé sur le cloud que vous pouvez utiliser pour entraîner, déployer, automatiser, gérer et suivre des modèles ML. 

Azure Machine Learning peut être utilisé pour tout type de machine learning, du machine learning classique à l’apprentissage profond, supervisé et non supervisé. Que vous préfériez écrire du code Python ou R avec le SDK ou utiliser des options sans code/peu de code dans le [studio](#build-ml-models-in-the-studio), vous pouvez créer, entraîner et gérer des modèles Machine Learning et Deep Learning dans un espace de travail Azure Machine Learning. 

Commencez à vous entraîner sur votre ordinateur local, puis effectuez un scale-out sur le cloud. 

Le service interagit également avec les outils open source populaires d’apprentissage profond et d’apprentissage par renforcement tels que PyTorch, TensorFlow, scikit-Learn et Ray RLlib. 

> [!VIDEO https://channel9.msdn.com/Events/Connect/Microsoft-Connect--2018/D240/player]

> [!Tip]
> **Version d’évaluation gratuite !**  Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui. Vous obtenez des crédits à dépenser en services Azure. Une fois ceux-ci épuisés, vous pouvez conserver le compte et utiliser les [services Azure gratuits](https://azure.microsoft.com/free/). Votre carte de crédit n’est pas débitée tant que vous n’avez pas explicitement modifié vos paramètres pour demander à l’être.


## <a name="what-is-machine-learning"></a>L’apprentissage automatique - De quoi s’agit-il ?

Machine Learning est une technique de science des données qui permet aux ordinateurs d’utiliser des données existantes afin de prévoir les tendances, les résultats et les comportements futurs. Grâce au machine learning, les ordinateurs apprennent sans programmation explicite.

Les prévisions ou prédictions générées à partir de Machine Learning peuvent rendre les applications et les appareils plus intelligents. Par exemple, quand vous faites vos achats en ligne, le machine learning permet de recommander d’autres produits que vous êtes susceptible de vouloir, en fonction de ce que vous avez acheté. Ou, lorsque vous utilisez votre carte de crédit, le machine learning compare la transaction à une base de données de transactions et aide la banque à détecter des fraudes. Et lorsque votre robot aspirateur nettoie une pièce, le machine learning l’aide à déterminer si le travail est terminé.

## <a name="machine-learning-tools-to-fit-each-task"></a>Outils de Machine Learning adaptés à chaque tâche 

Azure Machine Learning fournit tous les outils dont les développeurs et les scientifiques de données ont besoin pour leurs flux de travaux Machine Learning, notamment :
+ Le [concepteur Azure Machine Learning](tutorial-designer-automobile-price-train-score.md) : faites glisser et déposez des modules pour générer vos expériences, puis déployez des pipelines.

+ Notebooks Jupyter : utilisez nos [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks) ou créez vos propres notebooks pour tirer parti de nos exemples de <a href="/python/api/overview/azure/ml/intro?view=azure-ml-py" target="_blank">Kits de développement logiciel (SDK) Python</a> pour votre Machine Learning. 

+ Utilisez des scripts R ou des notebooks dans lesquels vous utilisez le <a href="https://azure.github.io/azureml-sdk-for-r/reference/index.html" target="_blank">Kit de développement logiciel (SDK) pour R</a> pour écrire votre propre code, ou utilisez les modules R dans le concepteur.

+ + L’[accélérateur de solution de nombreux modèles](https://aka.ms/many-models) (préversion) s’appuie sur Azure Machine Learning et vous permet d’effectuer l’apprentissage, l’utilisation et la gestion de centaines, voire de milliers de modèles Machine Learning.

+ [Extension Machine Learning pour les utilisateurs Visual Studio Code](tutorial-setup-vscode-extension.md)

+ [Interface CLI Machine Learning](reference-azure-machine-learning-cli.md)

+ Infrastructures open source telles que PyTorch, TensorFlow et scikit-learn, et bien d’autres encore

+ [Formation par renforcement](how-to-use-reinforcement-learning.md) avec Ray RLlib

Vous pouvez même utiliser [MLflow pour suivre des métriques et déployer des modèles](how-to-use-mlflow.md) ou Kubeflow pour [générer des pipelines de flux de travail de bout en bout](https://www.kubeflow.org/docs/azure/).

## <a name="build-ml-models-in-python-or-r"></a>Générer des modèles ML en Python ou R

Commencez l’entraînement sur votre ordinateur local en utilisant le <a href="/python/api/overview/azure/ml/intro?view=azure-ml-py" target="_blank">Kit de développement logiciel (SDK) Python</a> ou le <a href="https://azure.github.io/azureml-sdk-for-r/reference/index.html" target="_blank">Kit de développement logiciel (SDK) R</a> Azure Machine Learning. Vous pouvez ensuite effectuer un scale-out sur le cloud. 

Les nombreuses [cibles de calcul](how-to-create-attach-compute-studio.md) disponibles, comme la Capacité de calcul Azure Machine Learning et [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks), ainsi que les [services de réglage d’hyperparamètres avancés](how-to-tune-hyperparameters.md), vous permettent de générer de meilleurs modèles plus rapidement en tirant parti de la puissance du cloud.

Vous pouvez aussi [automatiser l’entraînement et l’optimisation de modèle](tutorial-auto-train-models.md) à l’aide du SDK.

## <a name="build-ml-models-in-the-studio"></a>Créer des modèles ML dans le studio

Le [studio Azure Machine Learning](https://studio.azureml.net) est un portail web dans Azure Machine Learning qui propose des options sans code ou avec peu de code pour entraîner et déployer des modèles ainsi que pour gérer les ressources. Le studio est intégré au SDK Azure Machine Learning pour une expérience simplifiée. Pour plus d’informations, consultez [Qu’est-ce que le studio Azure Machine Learning ?](overview-what-is-machine-learning-studio.md)

+ **Concepteur Azure Machine Learning**

  Utilisez le [concepteur](concept-designer.md) pour entraîner et déployer des modèles Machine Learning sans écrire de code. Essayez le [tutoriel du concepteur](tutorial-designer-automobile-price-train-score.md) pour commencer. 

  ![GIF animé de l’interface de type glisser-déposer du concepteur Azure Machine Learning](media/concept-designer/designer-drag-and-drop.gif)

+ **Suivre les expériences**

  Découvrez comment [suivre et visualiser les expériences de science des données](tutorial-first-experiment-automated-ml.md) dans le studio. 

    ![Détails de l’exécution dans le studio Azure Machine Learning](media/how-to-track-experiments/experimentation-tab.gif)


+ **Et bien plus encore...**

  Visitez le studio Azure Machine Learning sur [ml.azure.com](https://studio.azureml.net). 


## <a name="mlops-deploy--lifecycle-management"></a>MLOps : Déployer et gestion du cycle de vie
Quand vous disposez du modèle adéquat, vous pouvez facilement l’utiliser dans un service web, sur un appareil IoT ou à partir de Power BI. Pour plus d’informations, consultez l’article qui explique [comment et où effectuer le déploiement](how-to-deploy-and-where.md).

Vous pouvez ensuite gérer vos modèles déployés à l’aide du [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py), d’[Azure Machine Learning Studio](https://ml.azure.com) ou de l’[interface CLI Machine Learning](reference-azure-machine-learning-cli.md).

Une fois déployé, votre modèle peut retourner des prédictions [en temps réel](how-to-consume-web-service.md) ou [de manière asynchrone](./tutorial-pipeline-batch-scoring-classification.md) sur de grandes quantités de données.

Et avec les [pipelines de machine learning](concept-ml-pipelines.md) avancés, vous pouvez collaborer à chaque étape, de la préparation des données jusqu’à l’entraînement et à l’évaluation des modèles, en passant par le déploiement. Les pipelines vous permettent d’effectuer les opérations suivantes :

* Automatiser le processus de Machine Learning de bout en bout dans le cloud
* Réutiliser des composants et réexécuter les étapes seulement quand c’est nécessaire
* Utiliser des ressources de calcul différentes à chaque étape
* Exécuter des tâches de scoring par lots

Si vous souhaitez utiliser des scripts pour automatiser votre flux de travail de Machine Learning, l’[interface CLI Machine Learning](reference-azure-machine-learning-cli.md) fournit des outils de ligne de commande qui effectuent des tâches courantes telles que l’envoi d’une séquence d’entraînement ou le déploiement d’un modèle.

Pour commencer à utiliser Azure Machine Learning, consultez [Étapes suivantes](#next-steps).

## <a name="integration-with-other-services"></a>Intégration à d’autres services

Azure Machine Learning fonctionne avec d’autres services sur la plateforme Azure et s’intègre également à des outils open source, comme Git et MLFlow.

+ Des cibles de calcul comme __Azure Kubernetes Service__, __Azure Container Instances__, __Azure Databricks__, __Azure Data Lake Analytics__ et __Azure HDInsight__. Pour plus d’informations sur les cibles de calcul, consultez [Présentation des cibles de calcul](concept-compute-target.md).
+ __Azure Event Grid__. Pour plus d’informations, consultez [Consommer des événements Azure Machine Learning](./how-to-use-event-grid.md).
+ __Azure Monitor__. Pour plus d’informations, consultez [Supervision d’Azure Machine Learning](monitor-azure-machine-learning.md).
+ Des magasins de données comme __Comptes de stockage Azure__, __Azure Data Lake Storage__, __Azure SQL Database__, __Azure Database pour PostgreSQL__ et __Azure Open Datasets__. Pour plus d’informations, consultez [Accéder aux données dans les services de stockage Azure](how-to-access-data.md) et [Créer des jeux de données avec Azure Open Datasets](how-to-create-register-datasets.md).
+ __Réseaux virtuels Azure__. Pour plus d’informations, voir [Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels](how-to-network-security-overview.md).
+ __Azure Pipelines__. Pour plus d’informations, consultez [Entraîner et déployer des modèles Machine Learning](/azure/devops/pipelines/targets/azure-machine-learning).
+ __Journaux du dépôt Git__. Pour plus d’informations, consultez [Intégration de Git](concept-train-model-git-integration.md).
+ __MLFlow__. Pour plus d’informations, consultez [MLflow pour suivre des métriques et déployer des modèles](how-to-use-mlflow.md) 
+ __Kubeflow__. Pour plus d’informations, consultez [Générer des pipelines de workflows de bout en bout](https://www.kubeflow.org/docs/azure/).

### <a name="secure-communications"></a>Communications sécurisées

Votre compte de stockage Azure, vos cibles de calcul et d’autres ressources peuvent être utilisées de façon sécurisée au sein d’un réseau virtuel pour entraîner des modèles et effectuer l’inférence. Pour plus d’informations, voir [Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels](how-to-network-security-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Créez votre première expérience avec la méthode de votre choix :
- + [Bien démarrer avec votre propre environnement de développement](tutorial-1st-experiment-sdk-setup-local.md)
  + [Utiliser des notebooks Jupyter sur une instance de calcul pour entraîner et déployer des modèles ML](tutorial-1st-experiment-sdk-setup.md)
  + [Utiliser R Markdown pour entraîner et déployer des modèles ML](tutorial-1st-r-experiment.md) 
  + [Utiliser le Machine Learning automatisé pour entraîner et déployer des modèles ML](tutorial-first-experiment-automated-ml.md) 
  + [Utiliser les fonctionnalités glisser-déplacer du concepteur pour entraîner et déployer](tutorial-designer-automobile-price-train-score.md) 
  + [Utiliser l’interface CLI Machine Learning pour entraîner et déployer un modèle](tutorial-train-deploy-model-cli.md)

- Découvrez les [pipelines de machine learning](concept-ml-pipelines.md) pour générer, optimiser et gérer vos scénarios d’apprentissage.

- Consultez l’article détaillé [Architecture et concepts Azure Machine Learning](concept-azure-machine-learning-architecture.md).
