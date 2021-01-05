---
title: Modèle de hautes performances avec Triton (préversion)
titleSuffix: Azure Machine Learning
description: Apprenez à déployer votre modèle à l’aide du logiciel NVIDIA Triton Inference Server dans Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 09/23/2020
ms.topic: conceptual
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: c5db04a673c1cdc0c0f24e128f340f4ae55fea81
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825512"
---
# <a name="high-performance-serving-with-triton-inference-server-preview"></a>Haute performance avec un serveur Triton Inference (préversion) 

Découvrez comment utiliser [le serveur Triton Inference NVIDIA](https://developer.nvidia.com/nvidia-triton-inference-server) pour améliorer les performances du service Web utilisé pour l’inférence de modèle.

Déployer un modèle pour une inférence peut se faire comme un service Web. Par exemple, un déploiement vers Azure Container Instances et Azure Kubernetes Service. Par défaut, Azure Machine Learning utilise une infrastructure Web à thread unique, *à usage général* pour les déploiements de service Web.

Triton est une infrastructure *optimisée pour l’inférence*. Elle offre une meilleure utilisation des GPU et une inférence plus économique. Côté serveur, elle traite par lots les requêtes entrantes et envoie ces lots pour l’inférence. Le traitement par lot mieux fait appel aux ressources GPU et constitue un élément clé des performances de Triton.

> [!IMPORTANT]
> L’utilisation de Triton pour le déploiement à partir de Azure Machine Learning est actuellement en __préversion__. Les fonctions en préversion peuvent ne pas être couvertes par le service client. Pour plus d’informations, consulter [Conditions d’utilisation supplémentaires des Préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

> [!TIP]
> Les extraits de code de ce document sont fournis à titre d’illustration et n’affichent pas une solution complète. Pour obtenir un exemple de code fonctionnel, consultez les [exemples de bout en bout de Triton dans Azure Machine Learning](https://github.com/Azure/azureml-examples/tree/main/tutorials).

## <a name="prerequisites"></a>Prérequis

* Un **abonnement Azure**. Si vous n’en avez pas, essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).
* Vous savez [comment et où déployer un modèle](how-to-deploy-and-where.md) avec Azure Machine Learning.
* Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/?view=azure-ml-py) **ou** [Azure CLI](/cli/azure/?view=azure-cli-latest) et [l’extension de Machine Learning](reference-azure-machine-learning-cli.md).
* Une installation fonctionnelle de Docker pour le test local. Pour plus d’informations sur l’installation et la validation de Docker, consultez [Orientation et installation](https://docs.docker.com/get-started/) dans la documentation.

## <a name="architectural-overview"></a>Vue d’ensemble de l’architecture

Avant de tenter d’utiliser Triton pour votre propre modèle, il est important de comprendre comment il fonctionne avec Azure Machine Learning et comment il est comparé à un déploiement par défaut.

**Déploiement par défaut sans Triton**

* Plusieurs rôles de travail [Gunicorn](https://gunicorn.org/) démarrent pour gérer simultanément les requêtes entrantes.
* Ces rôles de travail gèrent le prétraitement, l’appel du modèle et le post-traitement. 
* Les requêtes d’inférence utilisent __l’URI de scoring__. Par exemple : `https://myserevice.azureml.net/score`.

:::image type="content" source="./media/how-to-deploy-with-triton/normal-deploy.png" alt-text="Diagramme d’architecture de déploiement normal, non Triton":::

### <a name="setting-the-number-of-workers"></a>Définition du nombre de workers

Pour définir le nombre de workers dans votre déploiement, définissez la variable d’environnement `WORKER_COUNT`. Étant donné que vous avez un objet [Environnement](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py) nommé `env`, vous pouvez procéder comme suit :

```{py}
env.environment_variables["WORKER_COUNT"] = "1"
```

Cela indique à Azure ML d’activer le nombre de workers que vous spécifiez.


**Déploiement de configuration de l’inférence avec Triton**

* Plusieurs rôles de travail [Gunicorn](https://gunicorn.org/) démarrent pour gérer simultanément les requêtes entrantes.
* Les requêtes sont transmises au **serveur Triton**. 
* Triton traite les requêtes par lots pour optimiser l’utilisation du GPU.
* Le client utilise __l’URI de scoring__ pour effectuer des requêtes. Par exemple : `https://myserevice.azureml.net/score`.

:::image type="content" source="./media/how-to-deploy-with-triton/inferenceconfig-deploy.png" alt-text="Déploiement d’Inferenceconfig avec Triton":::

Le flux de travail pour l’utilisation de Triton pour votre modèle de déploiement est le suivant :

1. Vérifiez que Triton peut servir votre modèle.
1. Vérifiez que vous pouvez envoyer des requêtes à votre modèle déployé par Triton.
1. Incorporez votre code spécifique à Triton dans votre déploiement AML.

## <a name="verify-that-triton-can-serve-your-model"></a>Vérifier que Triton peut servir votre modèle

Tout d’abord, suivez les étapes ci-dessous pour vérifier que le serveur d’inférence Triton peut servir votre modèle.

### <a name="optional-define-a-model-config-file"></a>(Facultati) Définir un fichier config de modèle

Le fichier config de modèle indique à Triton le nombre d’entrées attendues et les dimensions de ces entrées. Pour plus d’informations sur la création du fichier config, consultez [Configuration de modèle](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/model_configuration.html) dans la documentation NVIDIA.

> [!TIP]
> Nous utilisons l’option `--strict-model-config=false` lors du démarrage du serveur Triton Inference, ce qui signifie que vous n’avez pas besoin de fournir un fichier `config.pbtxt` pour les modèles ONNX ou TensorFlow.
> 
> Pour plus d’informations sur la création du fichier config, consultez [Configuration de modèle](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/model_configuration.html#generated-model-configuration) dans la documentation NVIDIA.

### <a name="use-the-correct-directory-structure"></a>Utilisez la structure de répertoire appropriée

Lors de l’inscription d’un modèle avec Azure Machine Learning, vous pouvez inscrire des fichiers individuels ou une structure de répertoires. Pour utiliser Triton, l’inscription du modèle doit être pour une structure de répertoires qui contient un répertoire nommé `triton`. La structure générale de ce répertoire est la suivante :

```bash
models
    - triton
        - model_1
            - model_version
                - model_file
                - config_file
        - model_2
            ...
```

> [!IMPORTANT]
> Cette structure de répertoire est un référentiel de modèles Triton et est nécessaire pour que votre ou vos modèles fonctionnent avec Triton. Pour plus d’informations, consultez [Triton Model Repositories](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/model_repository.html) dans la documentation NVIDIA.

### <a name="test-with-triton-and-docker"></a>Test avec Triton et Docker

Pour tester votre modèle et vous assurer qu’il s’exécute avec Triton, vous pouvez utiliser Docker. Les commandes suivantes extraient le conteneur Triton sur votre ordinateur local, puis démarrent le serveur Triton :

1. Pour extraire l’image du serveur Triton sur votre ordinateur local, utilisez la commande suivante :

    ```bash
    docker pull nvcr.io/nvidia/tritonserver:20.09-py3
    ```

1. Pour démarrer le serveur Triton, utilisez la commande suivante. Remplacez `<path-to-models/triton>` par le chemin d’accès au référentiel de modèle Triton qui contient vos modèles :

    ```bash
    docker run --rm -ti -v<path-to-models/triton>:/models nvcr.io/nvidia/tritonserver:20.09-py3 tritonserver --model-repository=/models --strict-model-config=false
    ```

    > [!IMPORTANT]
    > Si vous utilisez Windows, vous pouvez être invité à autoriser les connexions réseau à ce processus la première fois que vous exécutez la commande. Si c’est le cas, sélectionnez cette option pour activer l’accès.

    Une fois démarrées, les informations similaires au texte suivant sont enregistrées dans la ligne de commande :

    ```bash
    I0923 19:21:30.582866 1 http_server.cc:2705] Started HTTPService at 0.0.0.0:8000
    I0923 19:21:30.626081 1 http_server.cc:2724] Started Metrics Service at 0.0.0.0:8002
    ```

    La première ligne indique l’adresse du service Web. Dans ce cas, `0.0.0.0:8000`, qui est le même que `localhost:8000`.

1. Utilisez un utilitaire tel que la boucle pour accéder au point de terminaison d’intégrité.

    ```bash
    curl -L -v -i localhost:8000/v2/health/ready
    ```

    Cette commande renvoie des informations similaires aux suivantes. Notez le `200 OK`. Cet état indique que le serveur Web est en cours d’exécution.

    ```bash
    *   Trying 127.0.0.1:8000...
    * Connected to localhost (127.0.0.1) port 8000 (#0)
    > GET /v2/health/ready HTTP/1.1
    > Host: localhost:8000
    > User-Agent: curl/7.71.1
    > Accept: */*
    >
    * Mark bundle as not supporting multiuse
    < HTTP/1.1 200 OK
    HTTP/1.1 200 OK
    ```

Au-delà d’un contrôle d’intégrité de base, vous pouvez créer un client pour envoyer des données à Triton pour l’inférence. Pour plus d’informations sur la création d’un client, consultez les [exemples de clients](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/client_example.html) dans la documentation NVIDIA. Il existe également des exemples [Python dans le GitHub Triton](https://github.com/triton-inference-server/server/tree/master/src/clients/python/examples).

Pour plus d’informations sur l’exécution de Triton à l’aide de Docker, consultez [Running Triton on a system with a GPU](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/run.html#running-triton-on-a-system-with-a-gpu) et [Running Triton on a system without a GPU](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/run.html#running-triton-on-a-system-without-a-gpu).

### <a name="register-your-model"></a>Inscrire votre modèle

Maintenant que vous avez vérifié que votre modèle fonctionne avec Triton, inscrivez-le avec Azure Machine Learning. L’inscription de modèle stocke vos fichiers de modèle dans l’espace de travail Azure Machine Learning et sont utilisés lors du déploiement avec le kit de développement logiciel (SDK) Python et Azure CLI.

Les exemples suivants montrent comment inscrire un modèle :

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.model import Model

model = Model.register(
    model_path=os.path.join("..", "triton"),
    model_name="bidaf_onnx",
    tags={'area': "Natural language processing", 'type': "Question answering"},
    description="Question answering model from ONNX model zoo",
    workspace=ws
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml model register --model-path='triton' \
--name='bidaf_onnx' \
--workspace-name='<my_workspace>'
```
---

<a id="processing"></a>

## <a name="verify-you-can-call-into-your-model"></a>Vérifiez que vous pouvez appeler votre modèle

Après avoir vérifié que le Triton est capable de servir votre modèle, vous pouvez ajouter du code de pré-traitement et de post-traitement en définissant un _script d’entrée_. Ce fichier est nommé `score.py`. Pour plus d’informations sur le script d’entrée, consultez [Définir le code d’entrée](how-to-deploy-and-where.md#define-an-entry-script).

Les deux étapes principales sont l’initialisation d’un client HTTP Triton dans votre méthode `init()` et l’appel de ce client dans votre fonction `run()`.

### <a name="initialize-the-triton-client"></a>Initialiser le client Triton

Incluez un code comme l’exemple suivant dans votre fichier `score.py`. Triton dans Azure Machine Learning s’attend à être traité sur localhost, port 8000. Dans ce cas, localhost se trouve à l’intérieur de l’image de Docker pour ce déploiement, et non un port sur votre ordinateur local :

> [!TIP]
> Le package PIP `tritonhttpclient` est inclus dans l’environnement `AzureML-Triton` organisé. Il n’est donc pas nécessaire de le spécifier en tant que dépendance PIP.

```python
import tritonhttpclient

def init():
    global triton_client
    triton_client = tritonhttpclient.InferenceServerClient(url="localhost:8000")
```

### <a name="modify-your-scoring-script-to-call-into-triton"></a>Modifiez votre script de scoring pour appeler Triton

L’exemple suivant montre comment demander dynamiquement les métadonnées du modèle :

> [!TIP]
> Vous pouvez demander dynamiquement les métadonnées des modèles qui ont été chargés avec Triton à l’aide de la méthode `.get_model_metadata` du client Triton. Pour obtenir un exemple d’utilisation, consultez les [exemples de notebooks](https://aka.ms/triton-aml-sample).

```python
input = tritonhttpclient.InferInput(input_name, data.shape, datatype)
input.set_data_from_numpy(data, binary_data=binary_data)

output = tritonhttpclient.InferRequestedOutput(
         output_name, binary_data=binary_data, class_count=class_count)

# Run inference
res = triton_client.infer(model_name,
                          [input]
                          request_id='0',
                          outputs=[output])

```

<a id="redeploy"></a>

## <a name="redeploy-with-an-inference-configuration"></a>Redéployer avec une configuration d’inférence

Une configuration d’inférence vous permet d’utiliser un script d’entrée, ainsi que le processus de déploiement Azure Machine Learning à l’aide du kit de développement logiciel (SDK) Python ou de Azure CLI.

> [!IMPORTANT]
> Vous devez spécifier le `AzureML-Triton` [d’environnement curé](./resource-curated-environments.md).
>
> L’exemple de code Python clone `AzureML-Triton` dans un autre environnement appelé `My-Triton`. Le code Azure CLI utilise également cet environnement. Pour plus d’informations sur le clonage d’un environnement, consultez la référence [Environnement.Clone()](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py#clone-new-name-).

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import LocalWebservice
from azureml.core import Environment
from azureml.core.model import InferenceConfig


local_service_name = "triton-bidaf-onnx"
env = Environment.get(ws, "AzureML-Triton").clone("My-Triton")

for pip_package in ["nltk"]:
    env.python.conda_dependencies.add_pip_package(pip_package)

inference_config = InferenceConfig(
    entry_script="score_bidaf.py",  # This entry script is where we dispatch a call to the Triton server
    source_directory=os.path.join("..", "scripts"),
    environment=env
)

local_config = LocalWebservice.deploy_configuration(
    port=6789
)

local_service = Model.deploy(
    workspace=ws,
    name=local_service_name,
    models=[model],
    inference_config=inference_config,
    deployment_config=local_config,
    overwrite=True)

local_service.wait_for_deployment(show_output = True)
print(local_service.state)
# Print the URI you can use to call the local deployment
print(local_service.scoring_uri)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

> [!TIP]
> Pour plus d’informations sur la création d’une configuration d'inférence, consultez [Schéma de configuration de l’inférence](./reference-azure-machine-learning-cli.md#inference-configuration-schema).

```azurecli
az ml model deploy -n triton-densenet-onnx \
-m densenet_onnx:1 \
--ic inference-config.json \
-e My-Triton --dc deploymentconfig.json \
--overwrite --compute-target=aks-gpu
```

---

Après le déploiement, l’URI de scoring s’affiche. Pour ce déploiement local, ce sera `http://localhost:6789/score`. Si vous déployez sur le Cloud, vous pouvez utiliser la commande CLI [az ml service show](/cli/azure/ext/azure-cli-ml/ml/service?view=azure-cli-latest#ext_azure_cli_ml_az_ml_service_show) pour récupérer l’URI de scoring.

Pour plus d’informations sur la création d’un client qui envoie des demandes d’inférence à l’URI de score, consultez [Consommer un modèle déployé en tant que service Web](how-to-consume-web-service.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous prévoyez de continuer à utiliser l’espace de travail Azure Machine Learning, mais que vous souhaitez supprimer le service déployé, utilisez les options suivantes :

# <a name="python"></a>[Python](#tab/python)

```python
local_service.delete()
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az ml service delete -n triton-densenet-onnx
```

---

## <a name="next-steps"></a>Étapes suivantes

* [Consulter des exemples de bout-en-bout de Triton dans Azure Machine Learning](https://aka.ms/aml-triton-sample)
* Consulter [les exemples client Triton](https://github.com/triton-inference-server/server/tree/master/src/clients/python/examples)
* Lire la [documentation du serveur Triton Inference](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/index.html)
* [Résoudre des problèmes d’échec de déploiement](how-to-troubleshoot-deployment.md)
* [Déployer dans Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Mettre à jour un service web](how-to-deploy-update-web-service.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)