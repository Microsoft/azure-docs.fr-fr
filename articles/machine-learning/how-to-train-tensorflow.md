---
title: Formation et déploiement d’un modèle TensorFlow
titleSuffix: Azure Machine Learning
description: Découvrez la façon dont Azure Machine Learning vous permet d’effectuer un scale-out d’un travail de formation TensorFlow à l’aide de ressources de calcul de cloud élastique.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.date: 09/28/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 9b8d48139e6cbabfbc5bf63f85d2d03c64d7efd9
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542284"
---
# <a name="train-tensorflow-models-at-scale-with-azure-machine-learning"></a>Entraîner des modèles TensorFlow à grande échelle avec Azure Machine Learning

Dans cet article, découvrez comment exécuter vos scripts d’entraînement [TensorFlow](https://www.tensorflow.org/overview) à grande échelle avec Azure Machine Learning.

Cet exemple forme et inscrit un modèle TensorFlow pour classer les chiffres manuscrits à l’aide d’un réseau neuronal profond (DNN).

Que vous développiez complètement un modèle TensorFlow ou que vous transmettiez un [modèle existant](how-to-deploy-existing-model.md) dans le cloud, vous pouvez utiliser Azure Machine Learning pour effectuer un scale-out des travaux de formation Open source afin de créer, déployer, versionner et surveiller des modèles de niveau production.

## <a name="prerequisites"></a>Prérequis

Exécutez ce code sur l’un de ces environnements :

 - Instance de calcul Azure Machine Learning : pas de téléchargement ni d’installation nécessaire

     - Suivre le [Tutoriel : Configurer l’environnement et l’espace de travail](tutorial-1st-experiment-sdk-setup.md) pour créer un serveur de notebook dédié préchargé avec le kit SDK et l’exemple de dépôt.
    - Dans le dossier des exemples de Deep Learning sur le serveur de notebooks, recherchez un notebook finalisé et développé en accédant à ce répertoire : dossier **how-to-use-azureml > ml-frameworks > tensorflow > train-hyperparameter-tune-deploy-with-tensorflow**. 
 
 - Votre propre serveur de notebooks Jupyter

    - [Installer le SDK Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) (>= 1.15.0).
    - [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).
    - [Téléchargez les exemples de fichiers de script](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow/train-hyperparameter-tune-deploy-with-tensorflow) `tf_mnist.py` et `utils.py`
     
    Vous trouverez également une [version Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/train-hyperparameter-tune-deploy-with-tensorflow/train-hyperparameter-tune-deploy-with-tensorflow.ipynb) complète de ce guide sur la page des exemples GitHub. Le notebook inclut des sections développées couvrant l’optimisation des hyperparamètres intelligents, les modèles de déploiement et les widgets de notebook.

## <a name="set-up-the-experiment"></a>Configurer l’expérience

Cette section configure l’expérience d’entraînement via le chargement des packages Python requis, l’initialisation d’un espace de travail, la création de la cible de calcul et la définition de l’environnement d’entraînement.

### <a name="import-packages"></a>Importer des packages

Tout d’abord, importez les bibliothèques Python nécessaires.

```Python
import os
import urllib
import shutil
import azureml

from azureml.core import Experiment
from azureml.core import Workspace, Run
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

Utilisez la méthode `register()` pour inscrire des jeux de données dans votre espace de travail afin de pouvoir les partager avec d’autres personnes, les réutiliser dans différentes expériences et y faire référence par nom dans votre script d’entraînement.

```python
dataset = dataset.register(workspace=ws,
                           name='mnist-dataset',
                           description='training and test dataset',
                           create_new_version=True)

# list the files referenced by dataset
dataset.to_path()
```

### <a name="create-a-compute-target"></a>Créer une cible de calcul

Créez une cible de calcul sur laquelle vous exécuterez votre tâche TensorFlow. Dans cet exemple, créez un cluster de calcul Azure Machine Learning compatible avec le GPU.

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
Azure ML fournit des environnements prédéfinis et organisés si vous ne souhaitez pas définir votre propre environnement. Azure ML intègre plusieurs environnements organisés processeur et GPU pour TensorFlow correspondant à différentes versions de TensorFlow. Pour plus d’informations, consultez [ceci](resource-curated-environments.md).

Si vous souhaitez utiliser un environnement organisé, vous pouvez exécuter la commande suivante à la place :

```python
curated_env_name = 'AzureML-TensorFlow-2.2-GPU'
tf_env = Environment.get(workspace=ws, name=curated_env_name)
```

Pour voir les packages inclus dans l’environnement organisé, vous pouvez écrire les dépendances conda sur le disque :
```python
tf_env.save_to_directory(path=curated_env_name)
```

Assurez-vous que l’environnement organisé comprend toutes les dépendances requises par votre script d’entraînement. Si ce n’est pas le cas, vous devez modifier l’environnement pour qu’il inclue les dépendances manquantes. Notez que si l’environnement est modifié, vous devez lui attribuer un nouveau nom, car le préfixe « AzureML » est réservé aux environnements organisés. Si vous avez modifié le fichier YAML de dépendances conda, vous pouvez créer un environnement à partir de celui-ci avec un nouveau nom, par exemple :
```python
tf_env = Environment.from_conda_specification(name='tensorflow-2.2-gpu', file_path='./conda_dependencies.yml')
```

Si, à la place, vous avez directement modifié l’objet de l’environnement organisé, vous pouvez cloner cet environnement avec un nouveau nom :
```python
tf_env = tf_env.clone(new_name='tensorflow-2.2-gpu')
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
  - tensorflow-gpu==2.2.0
```

Créez un environnement Azure ML à partir de cette spécification de l’environnement conda. L’environnement sera empaqueté dans un conteneur Docker au moment de l’exécution.

Par défaut, si aucune image de base n’est spécifiée, Azure ML utilise une image de processeur `azureml.core.environment.DEFAULT_CPU_IMAGE` comme image de base. Dans la mesure où cet exemple exécute l’entraînement sur un cluster GPU, vous devez spécifier une image de base GPU avec les dépendances et les pilotes GPU nécessaires. Azure ML gère un ensemble d’images de base publiées sur Microsoft Container Registry (MCR) que vous pouvez utiliser ; consultez le dépôt GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) pour plus d’informations.

```python
tf_env = Environment.from_conda_specification(name='tensorflow-2.2-gpu', file_path='./conda_dependencies.yml')

# Specify a GPU base image
tf_env.docker.enabled = True
tf_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.1-cudnn7-ubuntu18.04'
```

> [!TIP]
> Si vous le souhaitez, vous pouvez simplement capturer toutes vos dépendances directement dans un Dockerfile ou une image Docker personnalisé et créer votre environnement à partir de là. Pour plus d’informations, consultez [Effectuer l’entraînement avec une image personnalisée](how-to-train-with-custom-image.md).

Pour plus d’informations sur la création et l’utilisation d’environnements, consultez [Créer et utiliser des environnements logiciels dans Azure Machine Learning](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Configurer et envoyer votre exécution d’entraînement

### <a name="create-a-scriptrunconfig"></a>Créer un ScriptRunConfig

Créez un objet [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) pour spécifier les détails de configuration de votre travail d’entraînement, y compris votre script d’entraînement, l’environnement à utiliser et la cible de calcul sur laquelle effectuer l’exécution. Les arguments de votre script d’entraînement sont transmis via la ligne de commande s’ils sont spécifiés dans le paramètre `arguments`.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', dataset.as_mount(),
        '--batch-size', 64,
        '--first-layer-neurons', 256,
        '--second-layer-neurons', 128,
        '--learning-rate', 0.01]

src = ScriptRunConfig(source_directory=script_folder,
                      script='tf_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=tf_env)
```

> [!WARNING]
> Azure Machine Learning exécute des scripts d’apprentissage en copiant l’intégralité du répertoire source. Si vous avez des données sensibles que vous ne souhaitez pas charger, utilisez un [fichier .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) ou ne l’incluez pas dans le répertoire source. Au lieu de cela, accédez à vos données à l’aide d’un [jeu de données](how-to-train-with-datasets.md) ML.

Pour plus d’informations sur la configuration des travaux avec ScriptRunConfig, consultez [Configurer et envoyer des exécutions d’entraînement](how-to-set-up-training-targets.md).

> [!WARNING]
> Si vous utilisiez l’estimateur TensorFlow pour configurer vos travaux d’entraînement TensorFlow, notez que les estimateurs seront dépréciés dans une prochaine version du SDK Azure ML. Avec le SDK Azure ML versions 1.15.0 et ultérieures, ScriptRunConfig est la méthode recommandée pour configurer des travaux d’entraînement, y compris ceux utilisant des frameworks DL.

### <a name="submit-a-run"></a>Envoyer une exécution

L’[objet d’exécution](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py) fournit l’interface à l’historique des exécutions pendant et après l’exécution de la tâche.

```Python
run = Experiment(workspace=ws, name='tf-mnist').submit(src)
run.wait_for_completion(show_output=True)
```
### <a name="what-happens-during-run-execution"></a>Ce qui se passe lors de l’exécution
Quand l’exécution est lancée, elle passe par les phases suivantes :

- **Préparation** : une image docker est créée en fonction de l’environnement défini. L’image est chargée dans le registre de conteneurs de l’espace de travail et mise en cache pour des exécutions ultérieures. Les journaux sont également transmis en continu à l’historique des exécutions et peuvent être affichés afin de surveiller la progression. Si un environnement organisé est spécifié à la place, l’image mise en cache qui renferme cet environnement organisé est utilisée.

- **Mise à l’échelle** : le cluster tente de monter en puissance si le cluster Batch AI nécessite plus de nœuds pour l’exécution que la quantité disponible actuellement.

- **En cours d’exécution** : tous les scripts dans le dossier de script sont chargés dans la cible de calcul, les magasins de données sont montés ou copiés, puis `script` est exécuté. Les sorties issues de stdout et du dossier **./logs** sont transmises en continu à l’historique des exécutions et peuvent être utilisées pour superviser l’exécution.

- **Post-traitement** : le dossier **./outputs** de l’exécution est copié dans l’historique des exécutions.

## <a name="register-or-download-a-model"></a>Inscrire ou télécharger un modèle

Une fois que vous avez entraîné le modèle, vous pouvez l’inscrire sur votre espace de travail. L’inscription du modèle vous permet de stocker vos modèles et de suivre leurs versions dans votre espace de travail afin de simplifier [la gestion et le déploiement des modèles](concept-model-management-and-deployment.md). Facultatif : en spécifiant les paramètres `model_framework`, `model_framework_version` et `resource_configuration`, le modèle de déploiement sans code devient disponible. Cela vous permet de déployer directement votre modèle en tant que service web à partir du modèle inscrit, et l’objet `ResourceConfiguration` définit la ressource de calcul pour le service web.

```Python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = run.register_model(model_name='tf-mnist', 
                           model_path='outputs/model',
                           model_framework=Model.Framework.TENSORFLOW,
                           model_framework_version='2.0',
                           resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5))
```

Vous pouvez aussi télécharger une copie locale du modèle à l’aide de l’objet d’exécution. Dans le script de formation `tf_mnist.py`, un objet de sauvegarde TensorFlow conserve le modèle dans un dossier local (local dans la cible de calcul). Vous pouvez utiliser l’objet d’exécution pour télécharger une copie.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)
run.download_files(prefix='outputs/model', output_directory='./model', append_prefix=False)
```

## <a name="distributed-training"></a>Entraînement distribué

Azure Machine Learning prend également en charge les tâches TensorFlow distribuées multinœuds, ce qui vous permet de mettre à l’échelle vos charges de travail d’entraînement. Vous pouvez facilement exécuter des tâches distribuées TensorFlow, et Azure ML gère l’orchestration à votre place.

Azure ML prend en charge l’exécution des tâches TensorFlow distribuées avec l’API d’entraînement distribuée intégrée de Horovod et TensorFlow.

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) est une infrastructure open source réduite pour la formation distribuée, développée par Uber. Elle offre un moyen facile d’écrire du code TensorFlow distribué pour l’entraînement.

Votre code d’entraînement doit être instrumenté avec Horovod pour l’entraînement distribué. Pour plus d’informations sur l’utilisation de Horovod avec TensorFlow, consultez la documentation de Horovod :

Pour plus d’informations sur l’utilisation de Horovod avec TensorFlow, consultez la documentation de Horovod :

* [Horovod avec TensorFlow](https://github.com/horovod/horovod/blob/master/docs/tensorflow.rst)
* [Horovod avec l’API Keras de TensorFlow](https://github.com/horovod/horovod/blob/master/docs/keras.rst)

De plus, assurez-vous que votre environnement d’entraînement comprend le package **horovod**. Si vous utilisez un environnement organisé TensorFlow, horovod est déjà inclus en tant qu’une des dépendances. Si vous utilisez votre propre environnement, assurez-vous que la dépendance horovod est incluse, par exemple :

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.2.0
  - horovod==0.19.5
```

Pour exécuter une tâche distribuée à l’aide de MPI/Horovod sur Azure ML, vous devez spécifier un [MpiConfiguration](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?preserve-view=true&view=azure-ml-py) au paramètre `distributed_job_config` du constructeur ScriptRunConfig. Le code ci-dessous configure une tâche distribuée à 2 nœuds exécutant un processus par nœud. Si vous souhaitez également exécuter plusieurs processus par nœud (par exemple, si votre référence SKU de cluster contient plusieurs GPU), spécifiez en plus le paramètre `process_count_per_node` dans MpiConfiguration (la valeur par défaut est `1`).

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory=project_folder,
                      script='tf_horovod_word2vec.py',
                      arguments=['--input_data', dataset.as_mount()],
                      compute_target=compute_target,
                      environment=tf_env,
                      distributed_job_config=MpiConfiguration(node_count=2))
```

Pour un tutoriel complet sur l’exécution d’un entraînement TensorFlow distribué avec Horovod sur Azure ML, consultez [Entraînement TensorFlow distribué avec Horovod](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/distributed-tensorflow-with-horovod).

### <a name="tfdistribute"></a>tf.distribute

Si vous utilisez l’[entraînement TensorFlow distribué natif](https://www.tensorflow.org/guide/distributed_training) dans votre code d’entraînement, par exemple l’API `tf.distribute.Strategy` TensorFlow 2.x, vous pouvez également lancer la tâche distribuée par le biais d’Azure ML. 

Pour ce faire, spécifiez un [TensorflowConfiguration](/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration?preserve-view=true&view=azure-ml-py) au paramètre `distributed_job_config` du constructeur ScriptRunConfig. Si vous utilisez `tf.distribute.experimental.MultiWorkerMirroredStrategy`, spécifiez le `worker_count` dans le TensorflowConfiguration correspondant au nombre de nœuds de votre travail d’entraînement.

```python
import os
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import TensorflowConfiguration

distr_config = TensorflowConfiguration(worker_count=2, parameter_server_count=0)

model_path = os.path.join("./outputs", "keras-model")

src = ScriptRunConfig(source_directory=source_dir,
                      script='train.py',
                      arguments=["--epochs", 30, "--model-dir", model_path],
                      compute_target=compute_target,
                      environment=tf_env,
                      distributed_job_config=distr_config)
```

Dans TensorFlow, la variable d’environnement `TF_CONFIG` est requise pour l’entraînement sur plusieurs machines. Azure ML configure et définit la variable `TF_CONFIG` de manière appropriée pour chaque Worker avant d’exécuter votre script d’entraînement. Si nécessaire, vous pouvez accéder à `TF_CONFIG` à partir de votre script d’entraînement via `os.environ['TF_CONFIG']`.

Exemple de structure de `TF_CONFIG` définie sur un nœud Worker principal :
```JSON
TF_CONFIG='{
    "cluster": {
        "worker": ["host0:2222", "host1:2222"]
    },
    "task": {"type": "worker", "index": 0},
    "environment": "cloud"
}'
```

Si votre script d’entraînement utilise la stratégie de serveur de paramètres pour l’entraînement distribué, par exemple, pour la version de TensorFlow 1.x héritée, vous devez également spécifier le nombre de serveurs de paramètres à utiliser dans la tâche, par exemple `distr_config = TensorflowConfiguration(worker_count=2, parameter_server_count=1)`.

## <a name="deploy-a-tensorflow-model"></a>Déployer un modèle TensorFlow

Le guide pratique de déploiement contient une section sur l’inscription des modèles, mais vous pouvez passer directement à la [création d’une cible de calcul](how-to-deploy-and-where.md#choose-a-compute-target) pour le déploiement, puisque vous disposez déjà d’un modèle inscrit.

### <a name="preview-no-code-model-deployment"></a>(Préversion) Modèle de déploiement sans code

Au lieu de l’itinéraire de déploiement traditionnel, vous pouvez également utiliser la fonctionnalité de déploiement sans code (préversion) pour TensorFlow. En inscrivant votre modèle comme indiqué ci-dessus avec les paramètres `model_framework`, `model_framework_version` et `resource_configuration`, vous pouvez simplement utiliser la fonction statique `deploy()` pour déployer votre modèle.

```python
service = Model.deploy(ws, "tensorflow-web-service", [model])
```

Le [guide pratique](how-to-deploy-and-where.md) complet couvre de manière plus approfondie le déploiement dans Azure Machine Learning.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez effectué l’apprentissage d’un modèle TensorFlow et l’avez inscrit et vous avez découvert les options de déploiement. Consultez ces autres articles pour en savoir plus sur Azure Machine Learning.

* [Effectuer le suivi des métriques d’exécution pendant l’entraînement](how-to-track-experiments.md)
* [Optimiser les hyperparamètres](how-to-tune-hyperparameters.md)
* [Architecture de référence de la formation du Deep Learning distribué dans Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
