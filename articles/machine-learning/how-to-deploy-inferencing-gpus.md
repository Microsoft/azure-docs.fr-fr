---
title: Déployer un modèle pour une inférence avec GPU
titleSuffix: Azure Machine Learning
description: Cet article explique comment utiliser Azure Machine Learning pour déployer un modèle Deep Learning TensorFlow compatible GPU en tant que service web et noter les demandes d’inférence.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: vaidyas
author: csteegz
ms.reviewer: larryfr
ms.date: 06/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy
ms.openlocfilehash: 8480a0b8722fbfff0f1d8a8fafc1a64f38d21d6e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93307208"
---
# <a name="deploy-a-deep-learning-model-for-inference-with-gpu"></a>Déployer un modèle de deep learning pour une inférence avec GPU


Cet article explique comment utiliser Azure Machine Learning pour déployer un modèle compatible GPU en tant que service web. Les informations contenues dans cet article sont basées sur le déploiement d’un modèle sur Azure Kubernetes service (AKS). Le cluster AKS fournit une ressource GPU qui est utilisée par le modèle pour l’inférence.

L’inférence, ou notation de modèle, est la phase au cours de laquelle le modèle déployé est utilisé pour effectuer des prédictions. L’utilisation de GPU au lieu d’UC offre des avantages en termes de performances pour le calcul hautement parallélisable.

> [!IMPORTANT]
> Pour les déploiements de service web, l’inférence GPU est uniquement prise en charge sur Azure Kubernetes Service. Pour l'inférence à l’aide d’un __pipeline d'apprentissage machine__ , les GPU sont uniquement pris en charge sur Azure Machine Learning Compute. Pour plus d’informations sur l’utilisation des pipelines Azure Machine Learning, consultez [Tutoriel : Créer un pipeline Azure Machine Learning pour le scoring par lots](tutorial-pipeline-batch-scoring-classification.md). 

> [!TIP]
> Bien que les extraits de code de cet article utilisent un modèle TensorFlow, vous pouvez appliquer les informations à n’importe quel framework Machine Learning qui prend en charge les GPU.

> [!NOTE]
> Les informations contenues dans cet article s’appuient sur l’article[How to deploy to Azure Kubernetes Service (Comment déployer sur Azure Kubernetes Service)](how-to-deploy-azure-kubernetes-service.md). L’article « How to deploy to Azure Kubernetes Service » couvre généralement le déploiement sur AKS,mais cet article traite du déploiement spécifique au GPU.

## <a name="prerequisites"></a>Prérequis

* Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Un environnement de développement Python dans lequel le SDK Azure Machine Learning est installé. Pour plus d’informations, consultez [Kit SDK Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).  

* Un modèle inscrit qui utilise un GPU.

    * Pour savoir comment inscrire des modèles, voir [Déployer des modèles](how-to-deploy-and-where.md#registermodel).

    * Pour créer et inscrire le modèle Tensorflow utilisé pour créer ce document, consultez [Entraîner le modèle Tensorflow](how-to-train-tensorflow.md).

* Compréhension générale de [comment et où déployer des modèles](how-to-deploy-and-where.md).

## <a name="connect-to-your-workspace"></a>Se connecter à un espace de travail

Pour vous connecter à un espace de travail existant, utilisez le code suivant :

> [!IMPORTANT]
> Cet extrait de code s’attend à ce que la configuration de l’espace de travail soit enregistrée dans le répertoire actif ou son parent. Pour plus d’informations sur la création d’un espace de travail, consultez [Créer et gérer les espaces de travail Azure Machine Learning](how-to-manage-workspace.md).   Pour plus d’informations sur l’enregistrement de la configuration dans un fichier, consultez [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).

```python
from azureml.core import Workspace

# Connect to the workspace
ws = Workspace.from_config()
```

## <a name="create-a-kubernetes-cluster-with-gpus"></a>Créer un cluster Kubernetes avec des GPU

Azure Kubernetes Service fournit de nombreuses options de GPU. Vous pouvez utiliser celle de votre choix pour l’inférence de modèle. Pour obtenir la liste complète des fonctionnalités et des coûts, voir la [ liste des machines virtuelles série N](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#n-series).

Le code suivant montre comment associer un cluster AKS pour votre espace de travail :

```python
from azureml.core.compute import ComputeTarget, AksCompute
from azureml.exceptions import ComputeTargetException

# Choose a name for your cluster
aks_name = "aks-gpu"

# Check to see if the cluster already exists
try:
    aks_target = ComputeTarget(workspace=ws, name=aks_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    # Provision AKS cluster with GPU machine
    prov_config = AksCompute.provisioning_configuration(vm_size="Standard_NC6")

    # Create the cluster
    aks_target = ComputeTarget.create(
        workspace=ws, name=aks_name, provisioning_configuration=prov_config
    )

    aks_target.wait_for_completion(show_output=True)
```

> [!IMPORTANT]
> Azure vous facture tant que le cluster AKS existe. Veillez à supprimer votre cluster AKS lorsque vous n’en avec plus besoin.

Pour plus d’informations sur l’utilisation d’AKS avec Azure Machine Learning, consultez [Guide pratique pour déployer sur Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

## <a name="write-the-entry-script"></a>Écrire le script d’entrée

Le script d’entrée reçoit les données envoyées au service web, les transmet au modèle et retourne les résultats du scoring. Le script suivant charge le modèle Tensorflow au démarrage, puis utilise le modèle pour effectuer le scoring des données.

> [!TIP]
> Le script d’entrée est spécifique de votre modèle. Par exemple, le script doit connaître le framework à utiliser avec votre modèle, les formats de données, etc.

```python
import json
import numpy as np
import os
import tensorflow as tf

from azureml.core.model import Model


def init():
    global X, output, sess
    tf.reset_default_graph()
    model_root = os.getenv('AZUREML_MODEL_DIR')
    # the name of the folder in which to look for tensorflow model files
    tf_model_folder = 'model'
    saver = tf.train.import_meta_graph(
        os.path.join(model_root, tf_model_folder, 'mnist-tf.model.meta'))
    X = tf.get_default_graph().get_tensor_by_name("network/X:0")
    output = tf.get_default_graph().get_tensor_by_name("network/output/MatMul:0")

    sess = tf.Session()
    saver.restore(sess, os.path.join(model_root, tf_model_folder, 'mnist-tf.model'))


def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # make prediction
    out = output.eval(session=sess, feed_dict={X: data})
    y_hat = np.argmax(out, axis=1)
    return y_hat.tolist()
```

Ce fichier est nommé `score.py`. Pour plus d’informations sur les scripts d’entrée, consultez la page [Comment et où déployer](how-to-deploy-and-where.md).

## <a name="define-the-conda-environment"></a>Définir l’environnement Conda

Le fichier d’environnement Conda spécifie les dépendances pour le service. Il inclut les dépendances requises aussi bien par le modèle que par le script d’entrée. Notez que vous devez indiquer azureml-defaults avec la version >= 1.0.45 en tant que dépendance pip, car elle contient les fonctionnalités nécessaires pour héberger le modèle en tant que service web. Le code YAML suivant définit l’environnement pour un modèle Tensorflow. Il spécifie `tensorflow-gpu`, qui utilise le GPU utilisé dans ce déploiement :

```yaml
name: project_environment
dependencies:
  # The python interpreter version.
  # Currently Azure ML only supports 3.5.2 and later.
- python=3.6.2

- pip:
  # You must list azureml-defaults as a pip dependency
  - azureml-defaults>=1.0.45
  - numpy
  - tensorflow-gpu=1.12
channels:
- conda-forge
```

Pour cet exemple, le fichier est enregistré sous nom `myenv.yml`.

## <a name="define-the-deployment-configuration"></a>Définir la configuration du déploiement

> [!IMPORTANT]
> AKS n’autorise pas les pods à partager des GPU ; vous ne pouvez avoir qu’autant de réplicas d’un service web compatible GPU qu’il y a de GPU dans le cluster.

La configuration du déploiement définit l’environnement Azure Kubernetes Service utilisé pour exécuter le service web :

```python
from azureml.core.webservice import AksWebservice

gpu_aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                    num_replicas=3,
                                                    cpu_cores=2,
                                                    memory_gb=4)
```

Pour plus d’informations, consultez la documentation de référence pour [AksService.deploy_configuration](/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py&preserve-view=true#&preserve-view=truedeploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-).

## <a name="define-the-inference-configuration"></a>Définir la configuration de l’inférence

La configuration de l'inférence pointe vers le script d’entrée et un objet d’environnement utilisant une image Docker avec prise en charge GPU. Notez que le fichier YAML utilisé pour la définition de l’environnement doit répertorier azureml-defaults avec la version >= 1.0.45 en tant que dépendance pip, car elle contient les fonctionnalités nécessaires pour héberger le modèle en tant que service web.

```python
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment, DEFAULT_GPU_IMAGE

myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
myenv.docker.base_image = DEFAULT_GPU_IMAGE
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
```

Pour plus d’informations sur les environnements , consultez [Créer et gérer des environnements pour la formation et le déploiement](how-to-use-environments.md).
Pour plus d’informations, consultez la documentation de référence pour [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig?preserve-view=true&view=azure-ml-py).

## <a name="deploy-the-model"></a>Déployer le modèle

Déployez le modèle sur votre cluster AKS et attendez qu’il crée votre service.

```python
from azureml.core.model import Model

# Name of the web service that is deployed
aks_service_name = 'aks-dnn-mnist'
# Get the registerd model
model = Model(ws, "tf-dnn-mnist")
# Deploy the model
aks_service = Model.deploy(ws,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=gpu_aks_config,
                           deployment_target=aks_target,
                           name=aks_service_name)

aks_service.wait_for_deployment(show_output=True)
print(aks_service.state)
```

Pour plus d’informations, consultez la documentation de référence sur [le modèle](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py).

## <a name="issue-a-sample-query-to-your-service"></a>Envoyer un exemple de requête à votre service

Envoyez une requête test au modèle déployé. Lorsque vous envoyez une image jpeg au modèle, celui-ci note l’image. L’exemple de code suivant télécharge des données de test, puis sélectionne une image de test aléatoire à envoyer au service.

```python
# Used to test your webservice
import os
import urllib
import gzip
import numpy as np
import struct
import requests

# load compressed MNIST gz files and return numpy arrays
def load_data(filename, label=False):
    with gzip.open(filename) as gz:
        struct.unpack('I', gz.read(4))
        n_items = struct.unpack('>I', gz.read(4))
        if not label:
            n_rows = struct.unpack('>I', gz.read(4))[0]
            n_cols = struct.unpack('>I', gz.read(4))[0]
            res = np.frombuffer(gz.read(n_items[0] * n_rows * n_cols), dtype=np.uint8)
            res = res.reshape(n_items[0], n_rows * n_cols)
        else:
            res = np.frombuffer(gz.read(n_items[0]), dtype=np.uint8)
            res = res.reshape(n_items[0], 1)
    return res

# one-hot encode a 1-D array
def one_hot_encode(array, num_of_classes):
    return np.eye(num_of_classes)[array.reshape(-1)]

# Download test data
os.makedirs('./data/mnist', exist_ok=True)
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz', filename='./data/mnist/test-images.gz')
urllib.request.urlretrieve('http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz', filename='./data/mnist/test-labels.gz')

# Load test data from model training
X_test = load_data('./data/mnist/test-images.gz', False) / 255.0
y_test = load_data('./data/mnist/test-labels.gz', True).reshape(-1)

# send a random row from the test set to score
random_index = np.random.randint(0, len(X_test)-1)
input_data = "{\"data\": [" + str(list(X_test[random_index])) + "]}"

api_key = aks_service.get_keys()[0]
headers = {'Content-Type': 'application/json',
           'Authorization': ('Bearer ' + api_key)}
resp = requests.post(aks_service.scoring_uri, input_data, headers=headers)

print("POST to url", aks_service.scoring_uri)
print("label:", y_test[random_index])
print("prediction:", resp.text)
```

Pour plus d’informations sur la création d’une application cliente, consultez [Créer un client pour consommer un service web déployé](how-to-consume-web-service.md).

## <a name="clean-up-the-resources"></a>Nettoyer les ressources

Si vous avez créé le cluster AKS spécifiquement pour cet exemple, supprimez vos ressources une fois que vous avez terminé.

> [!IMPORTANT]
> Azure vous facture en fonction de la durée de déploiement du cluster AKS. Veillez à le nettoyer lorsque vous n’en avez plus besoin.

```python
aks_service.delete()
aks_target.delete()
```

## <a name="next-steps"></a>Étapes suivantes

* [Déployer le modèle sur FPGA](how-to-deploy-fpga-web-service.md)
* [Déployer le modèle avec ONNX](concept-onnx.md#deploy-onnx-models-in-azure)
* [Effectuer l’apprentissage de modèles DNN Tensorflow](how-to-train-tensorflow.md)