---
title: Configurer une exécution d’entraînement
titleSuffix: Azure Machine Learning
description: Effectuez l’apprentissage de votre modèle Machine Learning sur divers environnements de formation (cibles de calcul). Vous pouvez facilement basculer entre différents environnements d’entraînement.
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 09/28/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q1
ms.openlocfilehash: 2197d5be91af4c93e9691e1dc2b953198669deaf
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027405"
---
# <a name="configure-and-submit-training-runs"></a>Configurer et soumettre des exécutions d’entraînement

Dans cet article, vous allez apprendre à configurer et à soumettre des exécutions Azure Machine Learning pour entraîner vos modèles. Des extraits de code décrivent les éléments clés de la configuration et de la soumission d'un script de formation.  Utilisez ensuite l'un des [exemples de notebooks](#notebooks) pour accéder aux exemples fonctionnels complets de bout en bout.

Lors de l’entraînement, il est courant de démarrer sur votre ordinateur local, puis d’effectuer un scale-out vers un cluster cloud. Avec Azure Machine Learning, vous pouvez exécuter votre script sur différentes cibles de calcul sans avoir à modifier votre script de formation.

Il vous suffit de définir l’environnement pour chaque cible de calcul dans une **configuration d’exécution de script**.  Ensuite, lorsque vous souhaitez exécuter votre expérience de formation sur une autre cible de calcul, spécifiez la configuration de série de tests pour celle-ci.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui
* Le [kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) (>= 1.13.0)
* Un [espace de travail Azure Machine Learning](how-to-manage-workspace.md), `ws`
* Une cible de calcul, `my_compute_target`.  [Créer une cible de calcul](how-to-create-attach-compute-studio.md) 

## <a name="whats-a-script-run-configuration"></a><a name="whats-a-run-configuration"></a>Qu’est-ce qu’une configuration d’exécution de script ?
Un objet [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) sert à configurer les informations nécessaires pour soumettre une exécution d’entraînement dans le cadre d’une expérience.

Vous soumettez votre expérience de formation à l’aide d’un objet ScriptRunConfig.  Cet objet inclut ce qui suit :

* **source_directory** : Répertoire source contenant votre script d’apprentissage
* **script** : script d’entraînement à exécuter
* **compute_target** : cible de calcul sur laquelle effectuer l’exécution
* **environment** : environnement à utiliser lors de l’exécution du script
* Et d’autres options configurables (consultez la [documentation de référence](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) pour plus d’informations)

## <a name="train-your-model"></a><a id="submit"></a>Effectuer l’apprentissage de votre modèle

Le modèle de code pour soumettre une série de tests d’apprentissage est le même pour tous les types de cibles de calcul :

1. Créer une expérience à exécuter
1. Créer un environnement dans lequel le script sera exécuté
1. Créer un objet ScriptRunConfig, qui spécifie la cible de calcul et l’environnement
1. Soumettre l’exécution
1. Attendre la fin de l’exécution

Vous pouvez également :

* Soumettre une série de tests HyperDrive pour un [réglage d’hyperparamètre](how-to-tune-hyperparameters.md).
* Soumettre une expérience via l’[extension VS Code](tutorial-train-deploy-image-classification-model-vscode.md#train-the-model).

## <a name="create-an-experiment"></a>Créer une expérience

Créez une expérience dans votre espace de travail.

```python
from azureml.core import Experiment

experiment_name = 'my_experiment'
experiment = Experiment(workspace=ws, name=experiment_name)
```

## <a name="select-a-compute-target"></a>Sélectionner une cible de calcul

Sélectionnez la cible de calcul sur laquelle votre script d’entraînement sera exécuté. Si aucune cible de calcul n’est spécifiée dans l’objet ScriptRunConfig, ou si `compute_target='local'`, Azure ML exécute votre script localement. 

L’exemple de code fourni dans cet article suppose que vous avez déjà créé une cible de calcul `my_compute_target` comme indiqué dans la section « Prérequis ».

## <a name="create-an-environment"></a>Créer un environnement
Les [environnements](concept-environments.md) Azure Machine Learning sont une encapsulation de l’environnement dans lequel votre formation Machine Learning se produit. Ils spécifient les packages, image Docker, variables d’environnement et paramètres logiciels Python autour de vos scripts d’entraînement et de scoring. Ils spécifient également les temps d’exécution (Python, Spark ou Docker).

Vous pouvez définir votre propre environnement ou utiliser un environnement organisé Azure ML. Les [environnements organisés](./how-to-use-environments.md#use-a-curated-environment) sont des environnements prédéfinis qui sont disponibles dans votre espace de travail par défaut. Ces environnements s’appuient sur des images Docker mises en cache, ce qui réduit le coût de préparation de l’exécution. Consultez [Environnements organisés Azure Machine Learning](./resource-curated-environments.md) pour obtenir la liste complète des environnements organisés disponibles.

Pour une cible de calcul à distance, vous pouvez utiliser l’un de ces environnements organisés courants pour commencer :

```python
from azureml.core import Workspace, Environment

ws = Workspace.from_config()
myenv = Environment.get(workspace=ws, name="AzureML-Minimal")
```

Pour plus d’informations et pour obtenir des détails sur les environnements, consultez [Créer et utiliser des environnements logiciels dans Azure Machine Learning](how-to-use-environments.md).
  
### <a name="local-compute-target"></a><a name="local"></a>Cible de calcul locale

Si votre cible de calcul est votre **ordinateur local**, vous devez vous assurer que tous les packages nécessaires sont disponibles dans l’environnement Python où le script s’exécute.  Utilisez `python.user_managed_dependencies` pour utiliser votre environnement Python actuel (ou celui sous le chemin d’accès que vous spécifiez).

```python
from azureml.core import Environment

myenv = Environment("user-managed-env")
myenv.python.user_managed_dependencies = True

# You can choose a specific Python environment by pointing to a Python path 
# myenv.python.interpreter_path = '/home/johndoe/miniconda3/envs/myenv/bin/python'
```

## <a name="create-the-script-run-configuration"></a>Créer la configuration d’exécution du script

Maintenant que vous avez une cible de calcul (`my_compute_target`) et un environnement (`myenv`), créez une configuration d’exécution de script qui exécute votre script de formation (`train.py`) situé dans votre répertoire `project_folder` :

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=project_folder,
                      script='train.py',
                      compute_target=my_compute_target,
                      environment=myenv)

# Set compute target
# Skip this if you are running on your local computer
script_run_config.run_config.target = my_compute_target
```

Si vous ne spécifiez pas d’environnement, un environnement par défaut est créé pour vous.

Si vous souhaitez passer des arguments de ligne de commande à votre script d’entraînement, vous pouvez les spécifier par le biais du paramètre **`arguments`** du constructeur ScriptRunConfig, par exemple `arguments=['--arg1', arg1_val, '--arg2', arg2_val]`.

Si vous souhaitez remplacer la durée maximale par défaut autorisée pour l’exécution, vous pouvez le faire avec le paramètre **`max_run_duration_seconds`** . Le système tente d’annuler automatiquement l’exécution si elle prend plus de temps que cette valeur.

### <a name="specify-a-distributed-job-configuration"></a>Spécifier une configuration de travail distribué
Si vous souhaitez exécuter un travail d’entraînement distribué, fournissez la configuration propre au travail distribué au paramètre **`distributed_job_config`** . Les types de configuration pris en charge sont les suivants : [MpiConfiguration](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?preserve-view=true&view=azure-ml-py), [TensorflowConfiguration](/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration?preserve-view=true&view=azure-ml-py) et [PyTorchConfiguration](/python/api/azureml-core/azureml.core.runconfig.pytorchconfiguration?preserve-view=true&view=azure-ml-py). 

Pour plus d’informations et des exemples sur l’exécution des travaux distribués Horovod, TensorFlow et PyTorch, consultez :

* [Entraîner des modèles TensorFlow](./how-to-train-tensorflow.md#distributed-training)
* [Entraîner des modèles PyTorch](./how-to-train-pytorch.md#distributed-training)

## <a name="submit-the-experiment"></a>Soumettre l’expérimentation

```python
run = experiment.submit(config=src)
run.wait_for_completion(show_output=True)
```

> [!IMPORTANT]
> Quand vous soumettez la série de tests d’apprentissage, un instantané du répertoire contenant vos scripts d’apprentissage est créé et envoyé à la cible de calcul. Il est également stocké dans le cadre de l’expérience dans votre espace de travail. Si vous modifiez des fichiers et que vous soumettez à nouveau l’exécution, seuls les fichiers modifiés seront téléchargés.
>
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
> 
> Pour plus d’informations sur les instantanés, consultez [Instantanés](concept-azure-machine-learning-architecture.md#snapshots).

> [!IMPORTANT]
> **Dossiers spéciaux** : deux dossiers, *outputs* et *logs*, reçoivent un traitement spécial de la part d’Azure Machine Learning. Pendant l’entraînement, quand vous écrivez des fichiers dans les dossiers *outputs* et *logs* relatifs au répertoire racine (`./outputs` et `./logs`, respectivement), les fichiers sont automatiquement chargés dans votre historique des exécutions. Vous pouvez ainsi y accéder une fois l’exécution terminée.
>
> Pour créer des artefacts pendant l’entraînement (par exemple, des fichiers de modèle, des points de contrôle, des fichiers de données ou des images tracées), écrivez-les dans le dossier `./outputs`.
>
> De même, vous pouvez écrire tous les journaux d’activité de votre exécution d’entraînement dans le dossier `./logs`. Pour utiliser [l’intégration TensorBoard](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/tensorboard/export-run-history-to-tensorboard/export-run-history-to-tensorboard.ipynb) d’Azure Machine Learning, écrivez vos journaux d’activité TensorBoard dans ce dossier. Lorsque votre exécution est en cours, vous pouvez lancer TensorBoard et diffuser ces journaux d’activité en continu.  Plus tard, vous pourrez également restaurer les journaux d’activité à partir de l’une de vos exécutions précédentes.
>
> Par exemple, pour télécharger sur votre ordinateur local un fichier écrit dans le dossier *outputs* après votre exécution d’entraînement distant : `run.download_file(name='outputs/my_output_file', output_file_path='my_destination_path')`

## <a name="git-tracking-and-integration"></a><a id="gitintegration"></a>Intégration et suivi Git

Lorsque vous lancez une exécution d’entraînement où le répertoire source est un répertoire Git local, les informations relatives au répertoire sont stockées dans l’historique des exécutions. Pour plus d’informations, consultez [Obtenir une intégration pour Azure Machine Learning](concept-train-model-git-integration.md).

## <a name="notebook-examples"></a><a name="notebooks"></a>Exemples de notebooks

Consultez ces notebooks pour obtenir des exemples de configuration d’exécutions pour différents scénarios d’entraînement :
* [Entraînement sur différentes cibles de calcul](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [Entraînement avec des frameworks ML](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Former un modèle](tutorial-train-models-with-aml.md) utilise une cible de calcul gérée pour former un modèle.
* Découvrez comment entraîner des modèles avec des frameworks ML spécifiques, comme [Scikit-learn](how-to-train-scikit-learn.md), [TensorFlow](how-to-train-tensorflow.md) et [PyTorch](how-to-train-pytorch.md).
* Découvrez comment [optimiser efficacement les hyperparamètres](how-to-tune-hyperparameters.md) afin de générer des modèles plus efficaces.
* Une fois le modèle formé, découvrez [comment et où déployer les modèles](how-to-deploy-and-where.md).
* Consultez la documentation de référence du SDK de la [classe ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py).
* [Utiliser Azure Machine Learning avec des réseaux virtuels Azure](./how-to-network-security-overview.md)
