---
title: Entraîner des modèles PyTorch de deep learning
titleSuffix: Azure Machine Learning
description: Découvrez comment exécuter vos scripts d’entraînement PyTorch à l’échelle de l’entreprise à l’aide d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.reviewer: peterlu
ms.date: 12/10/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: ed368615395614bc0d3e9a6f06727da8c64d8486
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559639"
---
# <a name="train-pytorch-models-at-scale-with-azure-machine-learning"></a>Entraîner des modèles PyTorch à grande échelle avec Azure Machine Learning

Dans cet article, découvrez comment exécuter vos scripts d’entraînement [PyTorch](https://pytorch.org/) à l’échelle de l’entreprise à l’aide d’Azure Machine Learning.

Les exemples de scripts dans cet article classifient des images de poulets et de dindes pour créer un réseau neuronal de Deep Learning (DNN) basé sur le [tutoriel](https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html) sur l’apprentissage de transfert de PyTorch. L’apprentissage de transfert est une technique qui applique les connaissances acquises lors de la résolution d’un problème à un problème différent, mais connexe. Cela permet de raccourcir le processus de formation en exigeant moins de données, de temps et de ressources de calcul qu’une formation partant de zéro.

Que vous soyez en train de former un modèle PyTorch d’apprentissage profond ou que vous déposez un modèle existant dans le Cloud, vous pouvez utiliser Azure Machine Learning pour effectuer un scale-out des tâches de formation Open source à l’aide des ressources de calcul de Cloud élastique. Vous pouvez créer, déployer, mettre à jour et surveiller des modèles de niveau production avec Azure Machine Learning. 

## <a name="prerequisites"></a>Prérequis

Exécutez ce code sur l’un de ces environnements :

- Instance de calcul Azure Machine Learning : pas de téléchargement ni d’installation nécessaire

    - Suivre le [Tutoriel : Configurer l’environnement et l’espace de travail](tutorial-1st-experiment-sdk-setup.md) pour créer un serveur de notebook dédié préchargé avec le kit SDK et l’exemple de dépôt.
    - Dans le dossier des exemples de Deep Learning sur le serveur de notebook, recherchez un notebook terminé et développé en accédant à ce répertoire : le dossier **how-to-use-azureml > ml-frameworks > pytorch > train-hyperparameter-tune-deploy-with-pytorch**. 
 
 - Votre propre serveur de notebooks Jupyter
    - [Installer le SDK Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) (>= 1.15.0).
    - [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).
    - [Télécharger les exemples de fichiers de script](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch/train-hyperparameter-tune-deploy-with-pytorch) `pytorch_train.py`
     
    Vous trouverez également une [version Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/train-hyperparameter-tune-deploy-with-pytorch/train-hyperparameter-tune-deploy-with-pytorch.ipynb) complète de ce guide sur la page des exemples GitHub. Le notebook inclut des sections développées couvrant l’optimisation des hyperparamètres intelligents, les modèles de déploiement et les widgets de notebook.

## <a name="set-up-the-experiment"></a>Configurer l’expérience

Cette section configure l’expérience d’entraînement via le chargement des packages Python requis, l’initialisation d’un espace de travail, la création de la cible de calcul et la définition de l’environnement d’entraînement.

### <a name="import-packages"></a>Importer des packages

Tout d’abord, importez les bibliothèques Python nécessaires.

```Python
import os
import shutil

from azureml.core.workspace import Workspace
from azureml.core import Experiment
from azureml.core import Environment

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
```

### <a name="initialize-a-workspace"></a>Initialiser un espace de travail

L’[espace de travail Azure Machine Learning](concept-workspace.md) est la ressource de niveau supérieur du service. Il vous fournit un emplacement centralisé dans lequel utiliser tous les artefacts que vous créez. Dans le kit de développement logiciel (SDK) Python, vous pouvez accéder aux artefacts de l’espace de travail en créant un objet [`workspace`](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py).

Créez un objet d’espace de travail à partir du fichier `config.json` créé dans la [section Conditions préalables](#prerequisites).

```Python
ws = Workspace.from_config()
```

### <a name="get-the-data"></a>Obtenir les données

Le jeu de données se compose de 120 images de formation par dindes et poules, avec 100 images de validations par classe. Nous téléchargeons et extrayons le jeu de données dans le cadre de notre script de formation `pytorch_train.py`. Les images sont un sous-ensemble du [Jeu de données Open Images v5](https://storage.googleapis.com/openimages/web/index.html).

### <a name="prepare-training-script"></a>Préparer un script d’apprentissage

Dans ce didacticiel, le script de formation, `pytorch_train.py`, est déjà fourni. En pratique, vous pouvez prendre n’importe quel script d’entraînement tel quel et l’exécuter avec Azure Machine Learning.

Créez un dossier pour votre ou vos scripts d’entraînement.

```python
project_folder = './pytorch-birds'
os.makedirs(project_folder, exist_ok=True)
shutil.copy('pytorch_train.py', project_folder)
```

### <a name="create-a-compute-target"></a>Créer une cible de calcul

Créez une cible de calcul sur laquelle vous exécuterez votre tâche PyTorch. Dans cet exemple, créez un cluster de calcul Azure Machine Learning compatible avec le GPU.

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

Pour définir l’[environnement](concept-environments.md) Azure ML qui encapsule les dépendances de votre script d’entraînement, vous pouvez définir un environnement personnalisé ou utiliser un environnement Azure ML organisé.

#### <a name="use-a-curated-environment"></a>Utiliser un environnement organisé
Azure ML fournit des environnements prédéfinis et organisés si vous ne souhaitez pas définir votre propre environnement. Azure ML intègre plusieurs environnements organisés GPU et processeur pour PyTorch correspondant à différentes versions de PyTorch. Pour plus d’informations, consultez [ceci](resource-curated-environments.md).

Si vous souhaitez utiliser un environnement organisé, vous pouvez exécuter la commande suivante à la place :

```python
curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)
```

Pour voir les packages inclus dans l’environnement organisé, vous pouvez écrire les dépendances conda sur le disque :
```python
pytorch_env.save_to_directory(path=curated_env_name)
```

Assurez-vous que l’environnement organisé comprend toutes les dépendances requises par votre script d’entraînement. Si ce n’est pas le cas, vous devez modifier l’environnement pour qu’il inclue les dépendances manquantes. Notez que si l’environnement est modifié, vous devez lui attribuer un nouveau nom, car le préfixe « AzureML » est réservé aux environnements organisés. Si vous avez modifié le fichier YAML de dépendances conda, vous pouvez créer un environnement à partir de celui-ci avec un nouveau nom, par exemple :
```python
pytorch_env = Environment.from_conda_specification(name='pytorch-1.6-gpu', file_path='./conda_dependencies.yml')
```

Si, à la place, vous avez directement modifié l’objet de l’environnement organisé, vous pouvez cloner cet environnement avec un nouveau nom :
```python
pytorch_env = pytorch_env.clone(new_name='pytorch-1.6-gpu')
```

#### <a name="create-a-custom-environment"></a>Créer un environnement personnalisé

Vous pouvez également créer votre propre environnement Azure ML qui encapsule les dépendances de votre script d’entraînement.

Tout d’abord, définissez vos dépendances conda dans un fichier YAML ; dans cet exemple, le fichier est nommé `conda_dependencies.yml`.

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - torch==1.6.0
  - torchvision==0.7.0
  - future==0.17.1
  - pillow
```

Créez un environnement Azure ML à partir de cette spécification de l’environnement conda. L’environnement sera empaqueté dans un conteneur Docker au moment de l’exécution.

Par défaut, si aucune image de base n’est spécifiée, Azure ML utilise une image de processeur `azureml.core.environment.DEFAULT_CPU_IMAGE` comme image de base. Dans la mesure où cet exemple exécute l’entraînement sur un cluster GPU, vous devez spécifier une image de base GPU avec les dépendances et les pilotes GPU nécessaires. Azure ML gère un ensemble d’images de base publiées sur Microsoft Container Registry (MCR) que vous pouvez utiliser ; consultez le dépôt GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) pour plus d’informations.

```python
pytorch_env = Environment.from_conda_specification(name='pytorch-1.6-gpu', file_path='./conda_dependencies.yml')

# Specify a GPU base image
pytorch_env.docker.enabled = True
pytorch_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.1-cudnn7-ubuntu18.04'
```

> [!TIP]
> Si vous le souhaitez, vous pouvez simplement capturer toutes vos dépendances directement dans un Dockerfile ou une image Docker personnalisé et créer votre environnement à partir de là. Pour plus d’informations, consultez [Effectuer l’entraînement avec une image personnalisée](how-to-train-with-custom-image.md).

Pour plus d’informations sur la création et l’utilisation d’environnements, consultez [Créer et utiliser des environnements logiciels dans Azure Machine Learning](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Configurer et envoyer votre exécution d’entrainement

### <a name="create-a-scriptrunconfig"></a>Créer un ScriptRunConfig

Créez un objet [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) pour spécifier les détails de configuration de votre travail d’entraînement, y compris votre script d’entraînement, l’environnement à utiliser et la cible de calcul sur laquelle effectuer l’exécution. Les arguments de votre script d’entraînement sont transmis via la ligne de commande s’ils sont spécifiés dans le paramètre `arguments`. 

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=project_folder,
                      script='pytorch_train.py',
                      arguments=['--num_epochs', 30, '--output_dir', './outputs'],
                      compute_target=compute_target,
                      environment=pytorch_env)
```

> [!WARNING]
> Azure Machine Learning exécute des scripts d’apprentissage en copiant l’intégralité du répertoire source. Si vous avez des données sensibles que vous ne souhaitez pas charger, utilisez un [fichier .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) ou ne l’incluez pas dans le répertoire source. Au lieu de cela, accédez à vos données à l’aide d’un [jeu de données](how-to-train-with-datasets.md) ML.

Pour plus d’informations sur la configuration des travaux avec ScriptRunConfig, consultez [Configurer et envoyer des exécutions d’entraînement](how-to-set-up-training-targets.md).

> [!WARNING]
> Si vous utilisiez l’estimateur PyTorch pour configurer vos travaux d’entraînement PyTorch, notez que les estimateurs seront dépréciés dans une prochaine version du SDK Azure ML. Avec le SDK Azure ML versions 1.15.0 et ultérieures, ScriptRunConfig est la méthode recommandée pour configurer des travaux d’entraînement, y compris ceux utilisant des frameworks DL.

## <a name="submit-your-run"></a>Envoyer votre exécution

L’[objet d’exécution](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py) fournit l’interface à l’historique des exécutions pendant et après l’exécution de la tâche.

```Python
run = Experiment(ws, name='pytorch-birds').submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="what-happens-during-run-execution"></a>Ce qui se passe lors de l’exécution
Quand l’exécution est lancée, elle passe par les phases suivantes :

- **Préparation** : une image docker est créée en fonction de l’environnement défini. L’image est chargée dans le registre de conteneurs de l’espace de travail et mise en cache pour des exécutions ultérieures. Les journaux sont également transmis en continu à l’historique des exécutions et peuvent être affichés afin de surveiller la progression. Si un environnement organisé est spécifié à la place, l’image mise en cache qui stocke cet environnement organisé est utilisée.

- **Mise à l’échelle** : le cluster tente de monter en puissance si le cluster Batch AI nécessite plus de nœuds pour l’exécution que la quantité disponible actuellement.

- **En cours d’exécution** : tous les scripts dans le dossier de script sont chargés dans la cible de calcul, les magasins de données sont montés ou copiés, puis `script` est exécuté. Les sorties issues de stdout et du dossier **./logs** sont transmises en continu à l’historique des exécutions et peuvent être utilisées pour superviser l’exécution.

- **Post-traitement** : le dossier **./outputs** de l’exécution est copié dans l’historique des exécutions.

## <a name="register-or-download-a-model"></a>Inscrire ou télécharger un modèle

Une fois que vous avez entraîné le modèle, vous pouvez l’inscrire sur votre espace de travail. L’inscription du modèle vous permet de stocker vos modèles et de suivre leurs versions dans votre espace de travail afin de simplifier [la gestion et le déploiement des modèles](concept-model-management-and-deployment.md).

```Python
model = run.register_model(model_name='pytorch-birds', model_path='outputs/model.pt')
```

> [!TIP]
> Le guide pratique de déploiement contient une section sur l’inscription des modèles, mais vous pouvez passer directement à la [création d’une cible de calcul](how-to-deploy-and-where.md#choose-a-compute-target) pour le déploiement, puisque vous disposez déjà d’un modèle inscrit.

Vous pouvez aussi télécharger une copie locale du modèle à l’aide de l’objet d’exécution. Dans le script de formation `pytorch_train.py`, un objet de sauvegarde PyTorch conserve le modèle dans un dossier local (local dans la cible de calcul). Vous pouvez utiliser l’objet d’exécution pour télécharger une copie.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

# Download the model from run history
run.download_file(name='outputs/model.pt', output_file_path='./model/model.pt'), 
```

## <a name="distributed-training"></a>Entraînement distribué

Azure Machine Learning prend également en charge les tâches PyTorch distribuées multinœuds, ce qui vous permet de mettre à l’échelle vos charges de travail d’entraînement. Vous pouvez facilement exécuter des tâches distribuées PyTorch, et Azure ML gère l’orchestration à votre place.

Azure ML prend en charge l’exécution des tâches PyTorch distribuées avec le module DistributedDataParallel intégré de Horovod et PyTorch.

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) est une infrastructure open source réduite pour la formation distribuée, développée par Uber. Elle offre un moyen facile d’écrire du code PyTorch distribué pour l’entraînement.

Votre code d’entraînement doit être instrumenté avec Horovod pour l’entraînement distribué. Pour plus d’informations sur l’utilisation de Horovod avec PyTorch, consultez la [documentation de Horovod](https://horovod.readthedocs.io/en/stable/pytorch.html).

De plus, assurez-vous que votre environnement d’entraînement comprend le package **horovod**. Si vous utilisez un environnement organisé PyTorch, horovod est déjà inclus en tant qu’une des dépendances. Si vous utilisez votre propre environnement, assurez-vous que la dépendance horovod est incluse, par exemple :

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - torch==1.6.0
  - torchvision==0.7.0
  - horovod==0.19.5
```

Pour exécuter une tâche distribuée à l’aide de MPI/Horovod sur Azure ML, vous devez spécifier un [MpiConfiguration](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?preserve-view=true&view=azure-ml-py) au paramètre `distributed_job_config` du constructeur ScriptRunConfig. Le code ci-dessous configure une tâche distribuée à 2 nœuds exécutant un processus par nœud. Si vous souhaitez également exécuter plusieurs processus par nœud (par exemple, si votre référence SKU de cluster contient plusieurs GPU), spécifiez en plus le paramètre `process_count_per_node` dans MpiConfiguration (la valeur par défaut est `1`).

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory=project_folder,
                      script='pytorch_horovod_mnist.py',
                      compute_target=compute_target,
                      environment=pytorch_env,
                      distributed_job_config=MpiConfiguration(node_count=2))
```

Pour un tutoriel complet sur l’exécution d’un entraînement PyTorch distribué avec Horovod sur Azure ML, consultez [Entraînement PyTorch distribué avec Horovod](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/distributed-pytorch-with-horovod).

### <a name="distributeddataparallel"></a>DistributedDataParallel
Si vous utilisez le module [DistributedDataParallel](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html) intégré de PyTorch qui est créé à l’aide du package **torch.distributed** dans votre code d’entraînement, vous pouvez également lancer la tâche distribuée via Azure ML.

Pour exécuter une tâche PyTorch distribué avec DistributedDataParallel, spécifiez un [PyTorchConfiguration](/python/api/azureml-core/azureml.core.runconfig.pytorchconfiguration?preserve-view=true&view=azure-ml-py) au paramètre `distributed_job_config` du constructeur ScriptRunConfig. Pour utiliser le back-end NCCL pour torch.distributed, spécifiez `communication_backend='Nccl'` dans le PyTorchConfiguration. Le code ci-dessous configure une tâche distribuée à 2 nœuds. Le back-end NCCL est le back-end recommandé pour l’entraînement du GPU distribué PyTorch.

Pour les tâches PyTorch distribuées configurées via PyTorchConfiguration, Azure ML définit les variables d’environnement suivantes sur les nœuds de la cible de calcul :

* `AZ_BATCHAI_PYTORCH_INIT_METHOD` : URL pour l’initialisation du système de fichiers partagés du groupe de processus
* `AZ_BATCHAI_TASK_INDEX` : classement global du processus Worker

Vous pouvez spécifier ces variables d’environnement aux arguments correspondants du script d’entraînement via le paramètre `arguments` de ScriptRunConfig.

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import PyTorchConfiguration

args = ['--dist-backend', 'nccl',
        '--dist-url', '$AZ_BATCHAI_PYTORCH_INIT_METHOD',
        '--rank', '$AZ_BATCHAI_TASK_INDEX',
        '--world-size', 2]

src = ScriptRunConfig(source_directory=project_folder,
                      script='pytorch_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=pytorch_env,
                      distributed_job_config=PyTorchConfiguration(communication_backend='Nccl', node_count=2))
```

Si vous préférez utiliser le back-end Gloo pour l’entraînement distribué, spécifiez `communication_backend='Gloo'` à la place. Le back-end Gloo est recommandé pour l’entraînement sur processeur distribué.

Pour un tutoriel complet sur l’exécution d’un entraînement PyTorch distribué sur Azure ML, consultez [Entraînement PyTorch distribué avec DistributedDataParallel](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/distributed-pytorch-with-nccl-gloo).

## <a name="export-to-onnx"></a>Exporter dans ONNX

Pour optimiser l’inférence avec le [Runtime ONNX](concept-onnx.md), convertissez votre modèle PyTorch formé au format ONNX. L’inférence, ou notation du modèle, est la phase où le modèle déployé est utilisé pour la prédiction, généralement sur des données de production. Pour obtenir un exemple, consultez le [tutoriel](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez entraîné et inscrit un réseau neuronal Deep Learning à l’aide de PyTorch sur Azure Machine Learning. Pour savoir comment déployer un modèle, passez à notre article relatif aux modèles de déploiement.

* [Comment et où déployer des modèles ?](how-to-deploy-and-where.md)
* [Effectuer le suivi des métriques d’exécution pendant l’entraînement](how-to-track-experiments.md)
* [Optimiser les hyperparamètres](how-to-tune-hyperparameters.md)
* [Déployer un modèle entraîné](how-to-deploy-and-where.md)
* [Architecture de référence de la formation du Deep Learning distribué dans Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
