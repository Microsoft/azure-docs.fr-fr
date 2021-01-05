---
title: Entraîner des modèles Keras de deep learning
titleSuffix: Azure Machine Learning
description: Découvrez comment entraîner et enregistrer un modèle de classification de réseau neuronal profond Keras s’exécutant sur TensorFlow à l’aide d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.reviewer: peterlu
ms.date: 09/28/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: a7d55c6e550000d2dd6c2930d95086ec433c246b
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93361095"
---
# <a name="train-keras-models-at-scale-with-azure-machine-learning"></a>Entraîner des modèles Keras à grande échelle avec Azure Machine Learning

Dans cet article, découvrez comment exécuter vos scripts d’entraînement Keras sur Azure Machine Learning.

L’exemple de code dans cet article montre comment entraîner et inscrire un modèle de classification Keras créé à l’aide du back-end TensorFlow avec Azure Machine Learning. Il utilise le jeu de données populaire [MNIST](http://yann.lecun.com/exdb/mnist/) pour classer les nombres manuscrits à l’aide d’un réseau neuronal profond (DNN) construit à l’aide de la [bibliothèque Python Keras](https://keras.io) s’exécutant par-dessus [TensorFlow](https://www.tensorflow.org/overview).

Keras est une API de réseau neuronal de haut niveau capable de s’exécuter par-dessus d’autres infrastructures DNN populaires afin de simplifier le développement. Azure Machine Learning vous permet d’effectuer rapidement un scale-out des tâches d’entraînement à l’aide de ressources de calcul cloud élastiques. Vous pouvez également suivre vos sessions de formation, contrôler les versions des modèles, déployer les modèles, et bien plus encore.

Que vous développiez un modèle Keras de A à Z ou importiez un modèle existant dans le cloud, Azure Machine Learning peut vous aider à créer des modèles prêts pour la production.

> [!NOTE]
> Si vous utilisez l’API Keras **tf.keras** intégrée à TensorFlow et non le package Keras autonome, reportez-vous plutôt à [Entraîner des modèles TensorFlow](how-to-train-tensorflow.md).

## <a name="prerequisites"></a>Prérequis

Exécutez ce code sur l’un de ces environnements :

- Instance de calcul Azure Machine Learning : pas de téléchargement ni d’installation nécessaire

     - Suivre le [Tutoriel : Configurer l’environnement et l’espace de travail](tutorial-1st-experiment-sdk-setup.md) pour créer un serveur de notebook dédié préchargé avec le kit SDK et l’exemple de dépôt.
    - Dans le dossier des exemples du serveur de notebook, recherchez un notebook terminé et développé en accédant à ce répertoire : le dossier **how-to-use-azureml > ml-frameworks > keras > train-hyperparameter-tune-deploy-with-keras**.

 - Votre propre serveur de notebooks Jupyter

    - [Installer le SDK Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) (>= 1.15.0).
    - [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).
    - [Téléchargez les exemples de fichiers de script](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras) `keras_mnist.py` et `utils.py`

    Vous trouverez également une [version Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras/train-hyperparameter-tune-deploy-with-keras.ipynb) complète de ce guide sur la page des exemples GitHub. Le notebook inclut des sections développées couvrant l’optimisation des hyperparamètres intelligents, les modèles de déploiement et les widgets de notebook.

## <a name="set-up-the-experiment"></a>Configurer l’expérience

Cette section configure l’expérience d’entraînement via le chargement des packages Python requis, l’initialisation d’un espace de travail, la création du FileDataset pour les données d’entraînement d’entrée, la création de la cible de calcul et la définition de l’environnement d’entraînement.

### <a name="import-packages"></a>Importer des packages

Tout d’abord, importez les bibliothèques Python nécessaires.

```Python
import os
import azureml
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import Workspace, Run
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
```

### <a name="initialize-a-workspace"></a>Initialiser un espace de travail

L’[espace de travail Azure Machine Learning](concept-workspace.md) est la ressource de niveau supérieur du service. Il vous fournit un emplacement centralisé dans lequel utiliser tous les artefacts que vous créez. Dans le kit de développement logiciel (SDK) Python, vous pouvez accéder aux artefacts de l’espace de travail en créant un objet [`workspace`](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py).

Créez un objet d’espace de travail à partir du fichier `config.json` créé dans la [section Conditions préalables](#prerequisites).

```Python
ws = Workspace.from_config()
```

### <a name="create-a-file-dataset"></a>Créer un jeu de données de fichier

Un objet `FileDataset` référence un ou plusieurs fichiers dans votre magasin de données d’espace de travail ou vos URL publiques. Les fichiers peuvent être de n’importe quel format, et la classe vous offre la possibilité de télécharger ou de monter les fichiers dans votre calcul. En créant un `FileDataset`, vous créez une référence à l’emplacement de la source de données. Si vous avez appliqué des transformations au jeu de données, elles seront également stockées dans le jeu de données. Les données restant à leur emplacement existant, aucun coût de stockage supplémentaire n’est encouru. Pour plus d’informations, consultez le guide de [procédures](./how-to-create-register-datasets.md) sur le package `Dataset`.

```python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
dataset = Dataset.File.from_files(path=web_paths)
```

Vous pouvez utiliser la méthode `register()` pour inscrire des jeux de données dans votre espace de travail afin de pouvoir les partager avec d’autres personnes, les réutiliser dans différentes expériences et y faire référence par nom dans votre script d’entraînement.

```python
dataset = dataset.register(workspace=ws,
                           name='mnist-dataset',
                           description='training and test dataset',
                           create_new_version=True)
```

### <a name="create-a-compute-target"></a>Créer une cible de calcul

Créez une cible de calcul sur laquelle vous exécuterez votre tâche d’entraînement. Dans cet exemple, créez un cluster de calcul Azure Machine Learning compatible avec le GPU.

```Python
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6',
                                                           max_nodes=4)

    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

Pour plus d’informations sur les cibles de calcul, consultez l’article [Qu’est-ce qu’une cible de calcul](concept-compute-target.md).

### <a name="define-your-environment"></a>Définir votre environnement

Définissez l’[environnement](concept-environments.md) Azure ML qui encapsule les dépendances de votre script d’entraînement.

Tout d’abord, définissez vos dépendances conda dans un fichier YAML ; dans cet exemple, le fichier est nommé `conda_dependencies.yml`.

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.0.0
  - keras<=2.3.1
  - matplotlib
```

Créez un environnement Azure ML à partir de cette spécification de l’environnement conda. L’environnement sera empaqueté dans un conteneur Docker au moment de l’exécution.

Par défaut, si aucune image de base n’est spécifiée, Azure ML utilise une image de processeur `azureml.core.environment.DEFAULT_CPU_IMAGE` comme image de base. Dans la mesure où cet exemple exécute l’entraînement sur un cluster GPU, vous devez spécifier une image de base GPU avec les dépendances et les pilotes GPU nécessaires. Azure ML gère un ensemble d’images de base publiées sur Microsoft Container Registry (MCR) que vous pouvez utiliser ; consultez le dépôt GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) pour plus d’informations.

```python
keras_env = Environment.from_conda_specification(name='keras-env', file_path='conda_dependencies.yml')

# Specify a GPU base image
keras_env.docker.enabled = True
keras_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.0-cudnn7-ubuntu18.04'
```

Pour plus d’informations sur la création et l’utilisation d’environnements, consultez [Créer et utiliser des environnements logiciels dans Azure Machine Learning](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Configurer et envoyer votre exécution d’entraînement

### <a name="create-a-scriptrunconfig"></a>Créer un ScriptRunConfig
Commencez par récupérer les données du magasin de données de l’espace de travail à l’aide de la classe `Dataset`.

```python
dataset = Dataset.get_by_name(ws, 'mnist-dataset')

# list the files referenced by mnist-dataset
dataset.to_path()
```

Créez un objet ScriptRunConfig pour spécifier les détails de configuration de votre travail d’entraînement, y compris votre script d’entraînement, l’environnement à utiliser et la cible de calcul sur laquelle effectuer l’exécution.

Les arguments de votre script d’entrainement sont transmis via la ligne de commande s’ils sont spécifiés dans le paramètre `arguments`. Le DatasetConsumptionConfig pour notre FileDataset est passé comme argument au script d’entraînement, pour l’argument `--data-folder`. Azure ML résout ce DatasetConsumptionConfig sur le point de montage du magasin de données de soutien, qui est ensuite accessible à partir du script d’entraînement.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', dataset.as_mount(),
        '--batch-size', 50,
        '--first-layer-neurons', 300,
        '--second-layer-neurons', 100,
        '--learning-rate', 0.001]

src = ScriptRunConfig(source_directory=script_folder,
                      script='keras_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=keras_env)
```

Pour plus d’informations sur la configuration des travaux avec ScriptRunConfig, consultez [Configurer et envoyer des exécutions d’entraînement](how-to-set-up-training-targets.md).

> [!WARNING]
> Si vous utilisiez l’estimateur TensorFlow pour configurer vos travaux d’entraînement Keras, notez que les estimateurs seront dépréciés dans une prochaine version du SDK Azure ML. Avec le SDK Azure ML versions 1.15.0 et ultérieures, ScriptRunConfig est la méthode recommandée pour configurer des travaux d’entraînement, y compris ceux utilisant des frameworks DL.

### <a name="submit-your-run"></a>Envoyer votre exécution

L’[objet d’exécution](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py) fournit l’interface à l’historique des exécutions pendant et après l’exécution de la tâche.

```Python
run = Experiment(workspace=ws, name='keras-mnist').submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="what-happens-during-run-execution"></a>Ce qui se passe lors de l’exécution
Quand l’exécution est lancée, elle passe par les phases suivantes :

- **Préparation** : une image docker est créée en fonction de l’environnement défini. L’image est chargée dans le registre de conteneurs de l’espace de travail et mise en cache pour des exécutions ultérieures. Les journaux sont également transmis en continu à l’historique des exécutions et peuvent être affichés afin de surveiller la progression. Si un environnement organisé est spécifié à la place, l’image mise en cache qui renferme cet environnement organisé est utilisée.

- **Mise à l’échelle**  : le cluster tente de monter en puissance si le cluster Batch AI nécessite plus de nœuds pour l’exécution que la quantité disponible actuellement.

- **En cours d’exécution**  : tous les scripts dans le dossier de script sont chargés dans la cible de calcul, les magasins de données sont montés ou copiés, puis `script` est exécuté. Les sorties issues de stdout et du dossier **./logs** sont transmises en continu à l’historique des exécutions et peuvent être utilisées pour superviser l’exécution.

- **Post-traitement** : le dossier **./outputs** de l’exécution est copié dans l’historique des exécutions.

## <a name="register-the-model"></a>Inscrire le modèle

Une fois que vous avez entraîné le modèle, vous pouvez l’inscrire sur votre espace de travail. L’inscription du modèle vous permet de stocker vos modèles et de suivre leurs versions dans votre espace de travail afin de simplifier [la gestion et le déploiement des modèles](concept-model-management-and-deployment.md).

```Python
model = run.register_model(model_name='keras-mnist', model_path='outputs/model')
```

> [!TIP]
> Le guide pratique de déploiement contient une section sur l’inscription des modèles, mais vous pouvez passer directement à la [création d’une cible de calcul](how-to-deploy-and-where.md#choose-a-compute-target) pour le déploiement, puisque vous disposez déjà d’un modèle inscrit.

Vous pouvez également télécharger une copie du modèle. Cela peut être utile pour effectuer un travail de validation de modèle supplémentaire localement. Dans le script d’entraînement `keras_mnist.py`, un objet de sauvegarde TensorFlow conserve le modèle dans un dossier local (local dans la cible de calcul). Vous pouvez utiliser l’objet d’exécution pour télécharger une copie à partir de l’historique des exécutions.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez entraîné et inscrit un modèle Keras sur Azure Machine Learning. Pour savoir comment déployer un modèle, passez à notre article relatif aux modèles de déploiement.

* [Comment et où déployer des modèles ?](how-to-deploy-and-where.md)
* [Effectuer le suivi des métriques d’exécution pendant l’entraînement](how-to-track-experiments.md)
* [Optimiser les hyperparamètres](how-to-tune-hyperparameters.md)
* [Déployer un modèle entraîné](how-to-deploy-and-where.md)
* [Architecture de référence de la formation du Deep Learning distribué dans Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
