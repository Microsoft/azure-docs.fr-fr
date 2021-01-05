---
title: Déployer des modèles avec une image Docker personnalisée
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser une image de base Docker personnalisée pour déployer vos modèles Azure Machine Learning. Même si Azure Machine Learning fournit une image par défaut, vous pouvez également utiliser votre propre image de base.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sagopal
author: saachigopal
ms.reviewer: larryfr
ms.date: 11/16/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy, devx-track-azurecli
ms.openlocfilehash: 1ff4d7693a7e493ccb736ab9363fd26c93017c79
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695348"
---
# <a name="deploy-a-model-using-a-custom-docker-base-image"></a>Déployer un modèle à l’aide d’une image de base Docker personnalisée

Découvrez comment utiliser une image Docker personnalisée lors du déploiement de modèles formés avec Azure Machine Learning.

Azure Machine Learning utilise une image Docker de base par défaut si aucune image n’est spécifiée. Vous pouvez trouver l’image Docker spécifique utilisée avec `azureml.core.runconfig.DEFAULT_CPU_IMAGE`. Vous pouvez également utiliser des __environnements__ Azure Machine Learning pour sélectionner une image de base spécifique, ou utiliser une image personnalisée que vous fournissez.

Une image de base est utilisée comme point de départ lorsque vous créez une image pour un déploiement. Elle fournit le système d’exploitation et les composants sous-jacents. Le processus de déploiement ajoute ensuite des composants supplémentaires, tels que votre modèle, un environnement conda et d’autres ressources, à l’image.

En règle générale, vous créez une image de base personnalisée lorsque vous souhaitez utiliser Docker pour gérer vos dépendances, maintenir un contrôle plus étroit sur les versions des composants ou gagner du temps lors du déploiement. Vous pouvez également souhaiter installer les logiciels requis par votre modèle, où le processus d’installation prend beaucoup de temps. L’installation du logiciel lors de la création de l’image de base signifie que vous n’avez pas à l’installer pour chaque déploiement.

> [!IMPORTANT]
> Lorsque vous déployez un modèle, vous ne pouvez pas remplacer les principaux composants tels que le serveur web ou des composants IoT Edge. Ces composants fournissent un environnement de travail connu qui est testé et pris en charge par Microsoft.

> [!WARNING]
> Microsoft peut ne pas être en mesure de vous aider à résoudre les problèmes dus à une image personnalisée. Si vous rencontrez des problèmes, vous pouvez être invité à utiliser l’image par défaut ou l’une des images fournies par Microsoft pour voir si le problème est spécifique à votre image.

Ce document est divisé en deux sections :

* Créer une image de base personnalisée : présente des informations aux administrateurs et DevOps sur la création d’une image personnalisée et la configuration de l’authentification dans un registre Azure Container Registry à l’aide de l’interface CLI Azure et CLI Machine Learning.
* Déployer un modèle à l’aide d’une image de base personnalisée : présente des informations aux scientifiques des données et DevOps/Ingénieurs ML sur l’utilisation des images personnalisées lors du déploiement d’un modèle entraîné à partir du kit de développement logiciel (SDK) Python ou de l’interface CLI ML.

## <a name="prerequisites"></a>Prérequis

* Un espace de travail Azure Machine Learning. Pour plus d’informations, consultez l’article [Créer un espace de travail](how-to-manage-workspace.md).
* Le [Kit de développement logiciel (SDK) Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py). 
* [Interface de ligne de commande Azure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).
* [Extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md).
* Registre [Azure Container Registry](../container-registry/index.yml) ou autre registre Docker accessible sur Internet.
* Les étapes de ce document supposent que vous êtes familiarisé avec la création et l’utilisation d’un objet de __configuration de l’inférence__ dans le cadre du déploiement de modèle. Pour plus d’informations, consultez [Où déployer et comment](how-to-deploy-and-where.md).

## <a name="create-a-custom-base-image"></a>Créer une image de base personnalisée

Les informations contenues dans cette section supposent que vous utilisez un registre Azure Container Registry pour stocker les images Docker. Utilisez la check-list suivante quand vous prévoyez de créer des images personnalisées pour Azure Machine Learning :

* Allez-vous utiliser le registre Azure Container Registry créé pour l’espace de travail Azure Machine Learning ou un registre Azure Container Registry autonome ?

    Quand vous utilisez des images stockées dans le __registre de conteneurs pour l’espace de travail__, vous n’avez pas besoin de vous authentifier auprès du registre. L’authentification est gérée par l’espace de travail.

    > [!WARNING]
    > Le registre Azure Container Registry pour votre espace de travail est __créé la première fois que vous entraînez ou déployez un modèle__ à l’aide de l’espace de travail. Si vous avez créé un espace de travail, mais sans entraîner ni créer un modèle, aucun registre Azure Container Registry n’existe pour l’espace de travail.

    Quand vous utilisez des images stockées dans un __registre de conteneurs autonome__, vous devez configurer un principal de service qui a au moins un accès en lecture. Vous indiquez ensuite l’ID de principal de service (nom d’utilisateur) et le mot de passe à toute personne qui utilise des images du registre. Cela ne s’applique toutefois pas si vous définissez le registre de conteneurs comme étant accessible publiquement.

    Pour plus d’informations sur la création d’un registre Azure Container Registry privé, consultez [Créer un registre de conteneurs privé](../container-registry/container-registry-get-started-azure-cli.md).

    Pour plus d’informations sur l’utilisation des principaux de service avec Azure Container Registry, consultez [Authentification Azure Container Registry avec des principaux de service](../container-registry/container-registry-auth-service-principal.md).

* Azure Container Registry et informations sur l’image : Indiquez le nom de l’image à toute personne qui doit l’utiliser. Par exemple, une image nommée `myimage`, stockée dans un registre nommé `myregistry`, est référencée en tant que `myregistry.azurecr.io/myimage` lors de l’utilisation de l’image pour le déploiement de modèle

### <a name="image-requirements"></a>Exigences des images

Azure Machine Learning prend uniquement en charge les images Docker qui fournissent les logiciels suivants :
* Ubuntu 16.04 ou ultérieur.
* Conda 4.5.# ou ultérieur.
* Python 3.5+.

Pour utiliser des jeux de données, installez le package libfuse-dev. Veillez également à installer les packages d’espace utilisateur dont vous pouvez avoir besoin.

Azure ML conserve un ensemble d’images de base d’UC et de GPU publiées sur le Registre de conteneurs Microsoft, que vous pouvez éventuellement exploiter (ou référencer) au lieu de créer votre propre image personnalisée. Pour voir les fichiers Dockerfile pour ces images, reportez-vous au dépôt GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers).

Pour les images de GPU, Azure ML offre actuellement des images de base cuda9 et cuda10. Les principales dépendances installées dans ces images de base sont les suivantes :

| Dépendances | UC IntelMPI | UC OpenMPI | GPU IntelMPI | GPU OpenMPI |
| --- | --- | --- | --- | --- |
| miniconda | ==4.5.11 | ==4.5.11 | ==4.5.11 | ==4.5.11 |
| mpi | intelmpi==2018.3.222 |openmpi==3.1.2 |intelmpi==2018.3.222| openmpi==3.1.2 |
| cuda | - | - | 9.0/10.0 | 9.0/10.0/10.1 |
| cudnn | - | - | 7.4/7.5 | 7.4/7.5 |
| nccl | - | - | 2.4 | 2.4 |
| git | 2.7.4 | 2.7.4 | 2.7.4 | 2.7.4 |

Les images d’UC sont générées à partir d’ubuntu16.04. Les images de GPU pour cuda9 sont générées à partir de nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04. Les images de GPU pour cuda10 sont générées à partir de nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04.
<a id="getname"></a>

> [!IMPORTANT]
> Lorsque vous utilisez des images Docker personnalisées, il est recommandé de les épingler afin de mieux garantir la reproductibilité.

### <a name="get-container-registry-information"></a>Obtenir les informations du registre de conteneurs

Dans cette section, découvrez comment obtenir le nom du registre Azure Container Registry pour votre espace de travail Azure Machine Learning.

> [!WARNING]
> Le registre Azure Container Registry pour votre espace de travail est __créé la première fois que vous entraînez ou déployez un modèle__ à l’aide de l’espace de travail. Si vous avez créé un espace de travail, mais sans entraîner ni créer un modèle, aucun registre Azure Container Registry n’existe pour l’espace de travail.

Si vous avez déjà entraîné ou déployé des modèles à l’aide d’Azure Machine Learning, un registre de conteneurs a été créé pour votre espace de travail. Pour rechercher le nom de ce registre de conteneurs, effectuez les étapes suivantes :

1. Ouvrez un nouveau shell ou une invite de commandes et utilisez la commande suivante pour vous authentifier auprès de votre abonnement Azure :

    ```azurecli-interactive
    az login
    ```

    Suivez les invites pour vous authentifier auprès de l’abonnement.

    [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

2. Utilisez la commande suivante pour lister le registre de conteneurs pour l’espace de travail. Remplacez `<myworkspace>` par le nom de votre espace de travail Azure Machine Learning. Remplacez `<resourcegroup>` par le groupe de ressources Azure qui contient votre espace de travail :

    ```azurecli-interactive
    az ml workspace show -w <myworkspace> -g <resourcegroup> --query containerRegistry
    ```

    [!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

    Les informations renvoyées sont semblables à ce qui suit :

    ```text
    /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.ContainerRegistry/registries/<registry_name>
    ```

    La valeur `<registry_name>` est le nom du registre Azure Container Registry pour votre espace de travail.

### <a name="build-a-custom-base-image"></a>Créer une image de base personnalisée

Les étapes de cette section vous guident tout au long de la création d’une image Docker personnalisée dans votre registre Azure Container Registry. Pour des exemples de fichiers dockerfile, consultez le dépôt GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers).

1. Créez un fichier texte nommé `Dockerfile` et utilisez le texte suivant comme contenu :

    ```text
    FROM ubuntu:16.04

    ARG CONDA_VERSION=4.7.12
    ARG PYTHON_VERSION=3.7
    ARG AZUREML_SDK_VERSION=1.13.0
    ARG INFERENCE_SCHEMA_VERSION=1.1.0

    ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
    ENV PATH /opt/miniconda/bin:$PATH
    ENV DEBIAN_FRONTEND=noninteractive

    RUN apt-get update --fix-missing && \
        apt-get install -y wget bzip2 && \
        apt-get install -y fuse && \
        apt-get clean -y && \
        rm -rf /var/lib/apt/lists/*

    RUN useradd --create-home dockeruser
    WORKDIR /home/dockeruser
    USER dockeruser

    RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-${CONDA_VERSION}-Linux-x86_64.sh -O ~/miniconda.sh && \
        /bin/bash ~/miniconda.sh -b -p ~/miniconda && \
        rm ~/miniconda.sh && \
        ~/miniconda/bin/conda clean -tipsy
    ENV PATH="/home/dockeruser/miniconda/bin/:${PATH}"

    RUN conda install -y conda=${CONDA_VERSION} python=${PYTHON_VERSION} && \
        pip install azureml-defaults==${AZUREML_SDK_VERSION} inference-schema==${INFERENCE_SCHEMA_VERSION} &&\
        conda clean -aqy && \
        rm -rf ~/miniconda/pkgs && \
        find ~/miniconda/ -type d -name __pycache__ -prune -exec rm -rf {} \;
    ```

2. À partir d’un shell ou d’une invite de commandes, utilisez les éléments suivants pour vous authentifier auprès du registre Azure Container Registry. Remplacez `<registry_name>` par le nom du registre de conteneurs dans lequel vous voulez stocker l’image :

    ```azurecli-interactive
    az acr login --name <registry_name>
    ```

3. Pour charger le fichier Dockerfile et le générer, utilisez la commande suivante. Remplacez `<registry_name>` par le nom du registre de conteneurs dans lequel vous voulez stocker l’image :

    ```azurecli-interactive
    az acr build --image myimage:v1 --registry <registry_name> --file Dockerfile .
    ```

    > [!TIP]
    > Dans cet exemple, une étiquette `:v1` est appliquée à l’image. Si aucune étiquette n’est fournie, une étiquette `:latest` est appliquée.

    Pendant le processus de génération, les informations sont renvoyées à la ligne de commande. Si la génération a réussi, vous recevez un message similaire au texte suivant :

    ```text
    Run ID: cda was successful after 2m56s
    ```

Pour plus d’informations sur la génération d’images avec un registre Azure Container Registry, consultez [Générer et exécuter une image conteneur à l’aide d’Azure Container Registry Tasks](../container-registry/container-registry-quickstart-task-cli.md)

Pour plus d’informations sur le chargement d’images existantes sur un registre Azure Container Registry, consultez [Envoyer votre première image vers un registre de conteneurs Docker privé](../container-registry/container-registry-get-started-docker-cli.md).

## <a name="use-a-custom-base-image"></a>Utiliser une image de base personnalisée

Pour utiliser une image personnalisée, vous avez besoin des informations suivantes :

* __Nom de l’image__. Par exemple, `mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda:latest` est le chemin d’accès à une image Docker simple fournie par Microsoft.

    > [!IMPORTANT]
    > Pour les images personnalisées que vous avez créées, veillez à inclure toutes les étiquettes qui ont été utilisées avec l’image. Par exemple, si votre image a été créée avec une étiquette spécifique, telle que `:v1`. Si vous n’avez pas utilisé d’étiquette spécifique lors de la création de l’image, une étiquette `:latest` a été appliquée.

* Si l’image se trouve dans un __dépôt privé__, vous avez besoin des informations suivantes :

    * __Adresse du registre__. Par exemple : `myregistry.azureecr.io`.
    * __Nom d’utilisateur__ et __mot de passe__ d’un principal de service qui a accès en lecture au registre.

    Si vous n’avez pas ces informations, contactez l’administrateur du registre Azure Container Registry qui contient votre image.

### <a name="publicly-available-base-images"></a>Images de base disponibles publiquement

Microsoft fournit plusieurs images Docker dans un dépôt accessible publiquement, qui peuvent être utilisées dans les étapes décrites dans cette section :

| Image | Description |
| ----- | ----- |
| `mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda` | Image principale pour Azure Machine Learning |
| `mcr.microsoft.com/azureml/onnxruntime:latest` | Contient ONNX Runtime pour l’inférence de l’UC |
| `mcr.microsoft.com/azureml/onnxruntime:latest-cuda` | Contient ONNX Runtime et CUDA pour le GPU |
| `mcr.microsoft.com/azureml/onnxruntime:latest-tensorrt` | Contient ONNX Runtime et TensorRT pour le GPU |
| `mcr.microsoft.com/azureml/onnxruntime:latest-openvino-vadm` | Contient ONNX Runtime et OpenVINO pour la technologie Intel<sup></sup> Vision Accelerator Design basée sur les VPU Movidius<sup>TM</sup> MyriadX |
| `mcr.microsoft.com/azureml/onnxruntime:latest-openvino-myriad` | Contient ONNX Runtime et OpenVINO pour les clés USB Intel<sup></sup> Movidius<sup>TM</sup> |

Pour plus d’informations sur les images de base ONNX Runtime, consultez la [section ONNX Runtime dockerfiles](https://github.com/microsoft/onnxruntime/blob/master/dockerfiles/README.md) dans le dépôt GitHub.

> [!TIP]
> Dans la mesure où ces images sont disponibles publiquement, vous n’avez pas besoin de fournir une adresse, un nom d’utilisateur ni un mot de passe lors de leur utilisation.

Pour en savoir plus, consultez le répertoire [Azure Machine Learning containers](https://github.com/Azure/AzureML-Containers) sur GitHub.

### <a name="use-an-image-with-the-azure-machine-learning-sdk"></a>Utiliser une image avec le kit SDK Azure Machine Learning

Pour utiliser une image stockée dans **Azure Container Registry pour votre espace de travail**, ou un **registre de conteneurs accessible publiquement**, définissez les attributs [Environment](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py) suivants :

+ `docker.enabled=True`
+ `docker.base_image`: Définissez le registre et le chemin de l’image.

```python
from azureml.core.environment import Environment
# Create the environment
myenv = Environment(name="myenv")
# Enable Docker and reference an image
myenv.docker.enabled = True
myenv.docker.base_image = "mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda:latest"
```

Pour utiliser une image d’un __registre de conteneurs privé__ qui ne se trouve pas dans votre espace de travail, vous devez utiliser `docker.base_image_registry` pour spécifier l’adresse du dépôt ainsi qu’un nom d’utilisateur et un mot de passe :

```python
# Set the container registry information
myenv.docker.base_image_registry.address = "myregistry.azurecr.io"
myenv.docker.base_image_registry.username = "username"
myenv.docker.base_image_registry.password = "password"

myenv.inferencing_stack_version = "latest"  # This will install the inference specific apt packages.

# Define the packages needed by the model and scripts
from azureml.core.conda_dependencies import CondaDependencies
conda_dep = CondaDependencies()
# you must list azureml-defaults as a pip dependency
conda_dep.add_pip_package("azureml-defaults")
myenv.python.conda_dependencies=conda_dep
```

Vous devez ajouter azureml-defaults avec la version >= 1.0.45 en tant que dépendance PIP. Ce package contient les fonctionnalités nécessaires pour héberger le modèle en tant que service web. Vous devez également définir la valeur « latest » pour la propriété de l’environnement inferencing_stack_version, ce qui installera les packages APT spécifiques requis par le service web. 

Après avoir défini l’environnement, utilisez-le avec un objet [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig?preserve-view=true&view=azure-ml-py) pour définir l’environnement d’inférence dans lequel le modèle et le service web s’exécuteront.

```python
from azureml.core.model import InferenceConfig
# Use environment in InferenceConfig
inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
```

À ce stade, vous pouvez passer au déploiement. Par exemple, l’extrait de code suivant déploie un service web localement à l’aide de la configuration d’inférence et de l’image personnalisée :

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Pour plus d’informations sur le déploiement, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

Pour plus d’informations sur la personnalisation de votre environnement Python, consultez [Créer et gérer des environnements pour la formation et le déploiement](how-to-use-environments.md). 

### <a name="use-an-image-with-the-machine-learning-cli"></a>Utiliser une image avec l’interface CLI Machine Learning

> [!IMPORTANT]
> Actuellement, l’interface CLI Machine Learning peut utiliser des images du registre Azure Container Registry pour votre espace de travail ou des dépôts accessibles publiquement. Elle ne peut pas utiliser d’images de registres privés autonomes.

Avant de déployer un modèle à l’aide de l’interface de ligne de commande de Machine Learning, créez un [environnement](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py) qui utilise l’image personnalisée. Créez ensuite un fichier de configuration d’inférence qui référence l’environnement. Vous pouvez également définir l’environnement directement dans le fichier de configuration d’inférence. Le document JSON suivant montre comment référencer une image dans un registre de conteneurs public. Dans cet exemple, l’environnement est défini en ligne :

```json
{
    "entryScript": "score.py",
    "environment": {
        "docker": {
            "arguments": [],
            "baseDockerfile": null,
            "baseImage": "mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda:latest",
            "enabled": false,
            "sharedVolumes": true,
            "shmSize": null
        },
        "environmentVariables": {
            "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
        },
        "name": "my-deploy-env",
        "python": {
            "baseCondaEnvironment": null,
            "condaDependencies": {
                "channels": [
                    "conda-forge"
                ],
                "dependencies": [
                    "python=3.6.2",
                    {
                        "pip": [
                            "azureml-defaults",
                            "azureml-telemetry",
                            "scikit-learn",
                            "inference-schema[numpy-support]"
                        ]
                    }
                ],
                "name": "project_environment"
            },
            "condaDependenciesFile": null,
            "interpreterPath": "python",
            "userManagedDependencies": false
        },
        "version": "1"
    }
}
```

Ce fichier est utilisé avec la commande `az ml model deploy`. Le paramètre `--ic` est utilisé pour spécifier le fichier de configuration de l’inférence.

```azurecli
az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json --ct akscomputetarget
```

Pour plus d’informations sur le déploiement d’un modèle à l’aide de l’interface CLI ML, consultez la section « Inscription du modèle, profilage et déploiement » de l’article [Extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md#model-registration-profiling-deployment).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus en détail [comment et où déployer des modèles](how-to-deploy-and-where.md).
* Découvrez comment [entraîner et déployer des modèles Machine Learning à l’aide d’Azure Pipelines](/azure/devops/pipelines/targets/azure-machine-learning?view=azure-devops&preserve-view=true).