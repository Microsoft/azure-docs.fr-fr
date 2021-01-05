---
title: Créer et entraîner des modèles
titleSuffix: Azure Machine Learning
description: Découvrez comment effectuer l’apprentissage de modèles avec Azure Machine Learning. Explorez les différentes méthodes de formation et choisissez celle qui convient à votre projet.
services: machine-learning
ms.service: machine-learning
author: Blackmist
ms.author: larryfr
ms.subservice: core
ms.topic: conceptual
ms.date: 05/13/2020
ms.custom: devx-track-python
ms.openlocfilehash: 2448f5f778f19674aec63291acb72536c65ca6c9
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94555887"
---
# <a name="train-models-with-azure-machine-learning"></a>Former des modèles avec Azure Machine Learning

Azure Machine Learning vous permet d’effectuer l’apprentissage de vos modèles de plusieurs façons, allant de solutions Code First s’appuyant sur le Kit de développement logiciel (SDK) à des solutions à faible code telles que le Machine Learning automatisé et le concepteur visuel. Consultez la liste suivante pour identifier la méthode d’entraînement qui vous convient le mieux :

+ [SDK Azure Machine Learning pour Python](#python-sdk) : le SDK Python permet d’entraîner les modèles de plusieurs façons, chacune s’appuyant sur des fonctionnalités différentes.

    | Méthode d’entraînement | Description |
    | ----- | ----- |
    | [Configuration d’exécution](#run-configuration) | Une **façon classique d’effectuer l’apprentissage de modèles** consiste à utiliser un script d’apprentissage et une configuration de série de tests. La configuration d’exécution fournit les informations nécessaires à la configuration de l’environnement d’entraînement utilisé pour entraîner votre modèle. Vous pouvez spécifier votre script d’apprentissage, votre cible de calcul et votre environnement Azure ML dans votre configuration de série de tests, et exécuter un travail d’apprentissage. |
    | [Machine learning automatisé](#automated-machine-learning) | Le machine learning automatisée permet d’**entraîner des modèles sans avoir nécessairement de connaissances approfondies en science des données ou en programmation**. Pour les personnes spécialisées dans la science des données et la programmation, il permet de gagner du temps et d’économiser des ressources grâce à l’automatisation de la sélection d’algorithme et de l’ajustement des hyperparamètres. Avec le machine learning automatisé, nul besoin de se soucier de la définition d’une configuration d’exécution. |
    | [Pipeline de machine learning](#machine-learning-pipeline) | Les pipelines ne constituent pas une méthode d’entraînement différente. Il s’agit d’une **façon de définir un workflow en suivant des étapes modulaires réutilisables**, qui peuvent inclure l’entraînement. Les pipelines de machine learning prennent en charge l’utilisation du Machine Learning automatisé et la configuration de série de tests pour effectuer l’apprentissage des modèles. Les pipelines n’étant pas spécifiquement axés sur l’entraînement, les raisons d’utiliser un pipeline sont plus variées que les autres méthodes d’entraînement. En règle générale, vous pouvez utiliser un pipeline dans les cas suivants :<br>* Vous souhaitez **planifier des processus sans assistance** comme des tâches d’entraînement durables ou une préparation de données.<br>* Utilisation de **plusieurs étapes** coordonnées sur des ressources de calcul et des emplacements de stockage hétérogènes.<br>* Utilisation du pipeline comme **modèle réutilisable** pour des scénarios spécifiques, comme le réentraînement ou le scoring par lots.<br>* **Suivi et versioning des sources de données, entrées et sorties** pour votre workflow.<br>* Votre workflow est **implémenté par différentes équipes qui travaillent de façon indépendante sur des étapes spécifiques**. Les étapes peuvent ensuite être regroupées dans un pipeline pour implémenter le workflow. |

+ [Kit de développement logiciel (SDK) Azure Machine Learning pour R (préversion)](#r-sdk-preview) le SDK pour R utilise le package reticulate pour établir une liaison avec le SDK Python d’Azure Machine Learning. Cela vous permet d’accéder aux objets et méthodes principaux implémentés dans le SDK Python à partir de tout environnement R.

+ **Concepteur** : Le concepts Azure Machine Learning offre une introduction simple à l’apprentissage automatique pour la génération d’une preuve de concepts, ou pour les utilisateurs peu expérimentés en matière de codage. Elle permet d’entraîner des modèles par glisser-déposer via une interface utilisateur web. Vous pouvez utiliser du code Python en phase de conception ou entraîner des modèles sans écrire de code.

+ **CLI** : l’interface de ligne de commande (CLI) de machine learning propose des commandes pour les tâches courantes d’Azure Machine Learning et est souvent employée pour **écrire des scripts et automatiser les tâches**. Par exemple, après avoir créé un script d’entraînement ou un pipeline, vous pouvez utiliser l’interface CLI pour lancer un entraînement selon une planification ou quand les fichiers de données utilisés pour l’entraînement sont mis à jour. Pour les modèles d’entraînement, elle propose des commandes qui soumettent les tâches d’entraînement. Elle peut soumettre des tâches utilisant des configurations d’exécution ou des pipelines.

Chacune de ces méthodes d’entraînement peut utiliser différents types de ressources de calcul pour l’entraînement. Ces ressources sont communément appelées [__cibles de calcul__](concept-azure-machine-learning-architecture.md#compute-targets). Une cible de calcul peut être un ordinateur local ou une ressource cloud, comme une capacité de calcul Azure Machine Learning, Azure HDInsight ou une machine virtuelle distante.

## <a name="python-sdk"></a>Kit de développement logiciel (SDK) Python

Le SDK Azure Machine Learning pour Python vous permet de créer et exécuter des workflows de machine learning avec Azure Machine Learning. Vous pouvez interagir avec le service à partir d’une session Python interactive, de notebooks Jupyter, de Visual Studio Code ou d’un autre IDE.

* [Qu’est-ce que le SDK Azure Machine Learning pour Python ?](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py)
* [Installer/mettre à jour le SDK](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py)
* [Configurer un environnement de développement pour Azure Machine Learning](how-to-configure-environment.md)

### <a name="run-configuration"></a>Configuration de série de tests

Un travail d’apprentissage générique avec Azure Machine Learning peut être défini à l’aide de la classe [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py). La configuration de série de tests de script est ensuite utilisée avec vos scripts d’apprentissage pour effectuer l’apprentissage d’un modèle sur une cible de calcul.

Vous pouvez commencer avec une configuration d’exécution pour votre ordinateur local avant de passer si besoin à une autre pour une cible de calcul cloud. Quand vous changez de cible de calcul, vous ne changez que la configuration d’exécution utilisée. Une exécution consigne aussi des informations sur la tâche d’entraînement, notamment les entrées, les sorties et les journaux.

* [Qu’est une configuration d’exécution ?](concept-azure-machine-learning-architecture.md#run-configurations)
* [Tutoriel : Entraîner votre premier modèle ML](tutorial-1st-experiment-sdk-train.md)
* [Exemples : exemples Jupyter Notebook et P de modèles d’apprentissage](https://github.com/Azure/azureml-examples)
* [Procédure : Configurer une exécution d’apprentissage](how-to-set-up-training-targets.md)

### <a name="automated-machine-learning"></a>Machine Learning automatisé

Définissez les itérations, la configuration des hyperparamètres, la caractérisation et d’autres paramètres. Pendant l’entraînement, Azure Machine Learning teste différents algorithmes et paramètres en parallèle. L’entraînement s’arrête une fois qu’il a atteint les critères de sortie que vous avez définis.

> [!TIP]
> Outre le Kit de développement logiciel (SDK) Python, vous pouvez utiliser le Machine Learning automatisé via [Azure Machine Learning Studio](https://ml.azure.com).

* [Qu’est-ce que le machine learning automatisé ?](concept-automated-ml.md)
* [Tutoriel : Créer votre premier modèle de classification avec le machine learning automatisé](tutorial-first-experiment-automated-ml.md)
* [Tutoriel : Utiliser le machine learning automatisé pour prédire le prix des courses de taxi](tutorial-auto-train-models.md)
* [Exemples : exemples Jupyter Notebook pour le machine learning automatisé](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning)
* [Procédure : configurer des expériences de ML automatisé dans Python](how-to-configure-auto-train.md)
* [Procédure : entraîner automatiquement un modèle de prévision de série chronologique](how-to-auto-train-forecast.md)
* [Procédure : créer, explorer et déployer des expériences de Machine Learning automatisé avec Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md)

### <a name="machine-learning-pipeline"></a>Pipeline de machine learning

Les pipelines d’apprentissage automatique peuvent utiliser les méthodes d’apprentissage mentionnées précédemment. Les pipelines visent plus à créer un workflow et ne se limitent donc pas au simple entraînement de modèles. Dans un pipeline, vous pouvez effectuer l’apprentissage d’un modèle en utilisant le Machine Learning automatisé ou des configurations de série de tests.

* [En quoi consistent les pipelines ML dans Azure Machine Learning ?](concept-ml-pipelines.md)
* [Créer et exécuter des pipelines de machine learning avec le kit SDK Azure Machine Learning](how-to-create-your-first-pipeline.md)
* [Tutoriel : Utiliser des pipelines Azure Machine Learning pour le scoring par lots](tutorial-pipeline-batch-scoring-classification.md)
* [Exemples : exemples Jupyter Notebook pour les pipelines de machine learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines)
* [Exemples : pipeline avec machine learning automatisé](https://aka.ms/pl-automl)

### <a name="understand-what-happens-when-you-submit-a-training-job"></a>Fonctionnement d’un travail de formation

Le cycle de vie de formation Azure est constitué des éléments suivants :

1. Compression des fichiers dans votre dossier de projet, en ignorant ceux spécifiés dans _.amlignore_ ou _.gitignore_
1. Scale up de votre cluster de calcul 
1. Création ou téléchargement du Dockerfile sur le nœud de calcul 
    1. Le système calcule un code de hachage pour : 
        - l’image de base ; 
        - les étapes Docker personnalisées (voir [Déployer un modèle à l’aide d’une image de base Docker personnalisée](./how-to-deploy-custom-docker-image.md)) ;
        - la définition Conda YAML (voir [Créer et utiliser des environnements logiciels dans Azure Machine Learning](./how-to-use-environments.md)).
    1. Le système utilise ce code de hachage comme clé pour rechercher le Dockerfile dans l’espace de travail Azure Container Registry (ACR).
    1. Si le Dockerfile est introuvable, il recherche une correspondance dans l’ensemble d’ACR.
    1. Si le Dockerfile est introuvable, le système génère une nouvelle image (qui sera mise en cache et inscrite auprès de l’espace de travail ACR).
1. Téléchargement de votre Fichier projet compressé vers le stockage temporaire sur le nœud de calcul
1. Décompression du Fichier projet
1. Nœud de calcul exécutant `python <entry script> <arguments>`
1. Enregistrement des journaux, des fichiers de modèle et des autres fichiers écrits dans `./outputs` dans le compte de stockage associé à l’espace de travail
1. Scale down du calcul, notamment la suppression du stockage temporaire 

Si vous choisissez d’effectuer l’apprentissage sur votre ordinateur local (« Configurer en tant qu’exécution locale »), vous n’avez pas besoin d’utiliser Docker. Vous pouvez utiliser Docker localement si vous le souhaitez (voir la section [Configurer un pipeline de ML](./how-to-debug-pipelines.md) pour obtenir un exemple).

## <a name="r-sdk-preview"></a>Kit de développement logiciel (SDK) R (préversion)

Le SDK R vous permet d’utiliser le langage R avec Azure Machine Learning. Le SDK utilise le package reticulate pour établir une liaison avec le SDK Python d’Azure Machine Learning. Cela vous permet d’accéder aux méthodes et objets principaux implémentés dans le Kit de développement logiciel (SDK) Python à partir de tout environnement R.

Pour plus d’informations, consultez les articles suivants :

* [Tutoriel : Créer un modèle de régression logistique](tutorial-1st-r-experiment.md)
* [Référence relative au SDK Azure Machine Learning pour R](https://azure.github.io/azureml-sdk-for-r/index.html)

## <a name="azure-machine-learning-designer"></a>Concepteur Azure Machine Learning

Le concepteur vous permet d’effectuer l’apprentissage de modèles via une interface de glisser-déplacer dans votre navigateur web.

+ [Qu’est-ce que le concepteur ?](concept-designer.md)
+ [Tutoriel : Prédire le prix de voitures](tutorial-designer-automobile-price-train-score.md)

## <a name="many-models-solution-accelerator"></a>Accélérateur de solution de nombreux modèles

L’[accélérateur de solution de nombreux modèles](https://aka.ms/many-models) (préversion) s’appuie sur Azure Machine Learning et vous permet d’effectuer l’apprentissage, l’utilisation et la gestion de centaines, voire de milliers de modèles Machine Learning.

Par exemple, la création d’un modèle __pour chaque instance ou chaque individu__ dans les scénarios suivants peut mener à des résultats améliorés :

* Prédiction des ventes pour chaque magasin individuel
* Maintenance prédictive pour des centaines de puits de pétrole
* Personnalisation d’une expérience pour des utilisateurs individuels

Pour plus d’informations, consultez l’[Accélérateur de solution de nombreux modèles](https://aka.ms/many-models) sur GitHub.

## <a name="cli"></a>Interface de ligne de commande

L’interface CLI de machine learning est une extension d’Azure CLI. Elle propose des commandes CLI multiplateformes à utiliser avec Azure Machine Learning. En règle générale, l’interface CLI sert à automatiser les tâches, par exemple à entraîner un modèle Machine Learning.

* [Utiliser l’extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md)
* [MLOps sur Azure](https://github.com/microsoft/MLOps)

## <a name="vs-code"></a>VS Code

Vous pouvez utiliser l’extension VS Code pour exécuter et gérer vos travaux de formation. Pour en savoir plus, consultez le [guide de procédures de gestion des ressources VS Code](how-to-manage-resources-vscode.md#experiments).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [Configurer une exécution d’apprentissage](how-to-set-up-training-targets.md).