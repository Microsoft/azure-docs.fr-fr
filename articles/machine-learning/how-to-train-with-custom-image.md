---
title: Former un modèle à l’aide d’une image Docker personnalisée
titleSuffix: Azure Machine Learning
description: Apprenez à utiliser vos propres images Docker ou celles de Microsoft et à effectuer l’apprentissage de modèles dans Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sagopal
author: saachigopal
ms.date: 10/20/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: cb8e71d9f94441f79dd7ce2fd5ee6458987563c3
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97795918"
---
# <a name="train-a-model-by-using-a-custom-docker-image"></a>Former un modèle à l’aide d’une image Docker personnalisée

Dans cet article, découvrez comment utiliser une image Docker personnalisée lors du déploiement de modèles formés avec Azure Machine Learning. Vous utiliserez les exemples de scripts de cet article pour classifier les images de PET en créant un réseau neuronal de convolution. 

Azure Machine Learning fournit une image de base par défaut. Vous pouvez également utiliser des environnements de Azure Machine Learning pour spécifier une autre image de base, telle que l’une des [images de base Azure Machine Learning ](https://github.com/Azure/AzureML-Containers) gérées ou votre propre [image personnalisée](how-to-deploy-custom-docker-image.md#create-a-custom-base-image). Les images de base personnalisées vous permettent de gérer précisément vos dépendances et d’exercer un contrôle rigoureux des versions des composants lors de l’exécution des travaux de formation.

## <a name="prerequisites"></a>Prérequis

Exécutez le code sur l’un de ces environnements :

* Instance de calcul Azure Machine Learning (ni téléchargement ni installation nécessaires) :
  * Pour créer un serveur Notebook dédié préchargé avec le kit de développement logiciel (SDK) et l’exemple de référentiel, effectuez[la configuration de l’environnement et de l’espace de travail](tutorial-1st-experiment-sdk-setup.md).
  * Dans le [référentiel d’exemples](https://github.com/Azure/azureml-examples) d’Azure Machine Learning, recherchez un bloc-notes terminé en accédant au répertoire :**notebooks** > **fastai** > **train-pets-resnet34.ipynb**. 
* Votre propre serveur de Jupyter Notebook :
  * Créer un [fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).
  * Installez le [Kit de développement logiciel (SDK) Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py). 
  * Créez un [registre de conteneurs Azure](../container-registry/index.yml) ou autre registre Docker accessible sur Internet.

## <a name="set-up-a-training-experiment"></a>Configurer une expérience de formation

Dans cette section, vous allez configurer votre expérience de formation en initialisant un espace de travail, en définissant votre environnement et en configurant une cible de calcul.

### <a name="initialize-a-workspace"></a>Initialiser un espace de travail

L’[espace de travail Azure Machine Learning](concept-workspace.md) est la ressource de niveau supérieur du service. Il vous offre un emplacement centralisé pour travailler avec tous les artefacts que vous créez. Dans le kit de développement logiciel (SDK) Python, vous pouvez accéder aux artefacts de l’espace de travail en créant un objet [`Workspace`](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py).

Créez un objet `Workspace` à partir du fichier config.son que vous avez créé en tant que [prérequis](#prerequisites).

```Python
from azureml.core import Workspace

ws = Workspace.from_config()
```

### <a name="define-your-environment"></a>Définir votre environnement

Créer un`Environment` objet et activez Docker.

```python
from azureml.core import Environment

fastai_env = Environment("fastai2")
fastai_env.docker.enabled = True
```

L’image de base spécifiée dans le code suivant prend en charge la bibliothèque fast.ai, qui permet des fonctionnalités de deep-learning distribuées. Pour plus d’informations, consultez le [référentiel Docker Hub fast.ai](https://hub.docker.com/u/fastdotai). 

Lorsque vous utilisez votre image Docker personnalisée, vous avez peut-être déjà configuré votre environnement Python correctement. Dans ce cas, définissez l’`user_managed_dependencies`indicateur sur `True` pour utiliser l’environnement Python intégré de votre image personnalisée. Par défaut, Azure Machine Learning crée un environnement Conda avec les dépendances que vous avez spécifiées. Le service exécute le script dans cet environnement au lieu d’utiliser des bibliothèques Python que vous avez installées sur l’image de base.

```python
fastai_env.docker.base_image = "fastdotai/fastai2:latest"
fastai_env.python.user_managed_dependencies = True
```

#### <a name="use-a-private-container-registry-optional"></a>Utiliser un registre de conteneurs privé (facultatif)

Pour utiliser une image à partir d’un registre de conteneurs privé qui ne se trouve pas dans votre espace de travail, utilisez `docker.base_image_registry` pour spécifier l’adresse du référentiel ainsi qu’un nom d’utilisateur et un mot de passe :

```python
# Set the container registry information.
fastai_env.docker.base_image_registry.address = "myregistry.azurecr.io"
fastai_env.docker.base_image_registry.username = "username"
fastai_env.docker.base_image_registry.password = "password"
```

#### <a name="use-a-custom-dockerfile-optional"></a>Utiliser un fichier Dockerfile personnalisé (facultatif)

Il est également possible d’utiliser un fichier Dockerfile personnalisé. Utilisez cette approche si vous devez installer des packages non-Python en tant que dépendances. N’oubliez pas de définir l’image de base sur `None`.

```python 
# Specify Docker steps as a string. 
dockerfile = r"""
FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04
RUN echo "Hello from custom container!"
"""

# Set the base image to None, because the image is defined by Dockerfile.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = dockerfile

# Alternatively, load the string from a file.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = "./Dockerfile"
```

Pour plus d’informations sur la création et la gestion d’environnements Azure Machine Learning, consultez [créer et utiliser des environnements logiciels](how-to-use-environments.md). 

### <a name="create-or-attach-a-compute-target"></a>Créer ou attacher une cible de calcul

Vous devez créer une [cible de calcul](concept-azure-machine-learning-architecture.md#compute-targets) pour la formation de votre modèle. Dans ce tutoriel, vous allez créer `AmlCompute` comme ressource de calcul de formation.

La création de `AmlCompute` prend quelques minutes. Si la`AmlCompute` ressource figure déjà dans votre espace de travail, ce code ignore le processus de création.

À l’instar d’autres services Azure, certaines ressources (par exemple, `AmlCompute`) associées au service Azure Machine Learning présentent des limites. Pour plus d’informations, consultez [Limites par défaut et comment demander un quota plus élevé](how-to-manage-quotas.md).

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# Choose a name for your cluster.
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target.')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6',
                                                           max_nodes=4)

    # Create the cluster.
    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True)

# Use get_status() to get a detailed status for the current AmlCompute.
print(compute_target.get_status().serialize())
```

## <a name="configure-your-training-job"></a>Configurer votre tâche d’entraînement

Pour ce tutoriel, utilisez le script de formation *train.py* sur [GitHub](https://github.com/Azure/azureml-examples/blob/main/workflows/train/fastai/pets/src/train.py). Dans la pratique, vous pouvez utiliser n’importe quel script de formation personnalisé et l’exécuter, comme c’est le cas, avec Azure Machine Learning.

Créez `ScriptRunConfig` une ressource pour configurer votre travail en vue de son exécution sur la[cible de calcul](how-to-set-up-training-targets.md)souhaitée.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory='fastai-example',
                      script='train.py',
                      compute_target=compute_target,
                      environment=fastai_env)
```

## <a name="submit-your-training-job"></a>Envoyer votre tâche d’entraînement

Lorsque vous soumettez une exécution d’apprentissage à l’aide d’un `ScriptRunConfig` objet, la `submit` méthode retourne un objet de type `ScriptRun`. L’objet `ScriptRun` retourné vous donne un accès par programmation aux informations sur l’exécution de l’apprentissage. 

```python
from azureml.core import Experiment

run = Experiment(ws,'Tutorial-fastai').submit(src)
run.wait_for_completion(show_output=True)
```

> [!WARNING]
> Azure Machine Learning exécute des scripts d’apprentissage en copiant l’intégralité du répertoire source. Si vous avez des données sensibles que vous ne souhaitez pas charger, utilisez un [fichier .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) ou ne l’incluez pas dans le répertoire source. Au lieu de cela, accédez à vos données à l’aide d’un [magasin de données](/python/api/azureml-core/azureml.data?preserve-view=true&view=azure-ml-py).

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez formé un modèle à l’aide d’une image Docker personnalisée. Pour en savoir plus sur Azure Machine Learning, consultez les articles suivants :
* [Suivre les métriques d’exécution ](how-to-track-experiments.md) durant la formation.
* [Déployer un modèle](how-to-deploy-custom-docker-image.md) à l’aide d’une image Docker personnalisée.
