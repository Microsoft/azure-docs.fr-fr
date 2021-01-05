---
title: Comment et où déployer des modèles ?
titleSuffix: Azure Machine Learning
description: Découvrez comment et où déployer vos modèles Azure Machine Learning service, notamment Azure Container Instances, Azure Kubernetes Service, Azure IoT Edge et FPGA.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.reviewer: larryfr
ms.date: 12/11/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy, devx-track-azurecli
ms.openlocfilehash: dd7a336df5ff187b874876db32abb45915e00f3b
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739380"
---
# <a name="deploy-models-with-azure-machine-learning"></a>Déployer des modèles avec Azure Machine Learning

Découvrez comment déployer votre modèle Machine Learning en tant que service web dans le cloud Azure ou sur des appareils Azure IoT Edge.

Le workflow est le même, quel que soit l’endroit où vous déployez votre modèle :

1. Inscrire le modèle (facultatif, cf. ci-dessous).
1. Préparer une configuration d’inférence (sauf en cas de [déploiement sans code](./how-to-deploy-no-code-deployment.md)).
1. Préparer un script d’entrée (sauf en cas de [déploiement sans code](./how-to-deploy-no-code-deployment.md)).
1. Choisir une cible de calcul.
1. Déployer le modèle sur la cible de calcul.
1. Tester le service web qui en résulte.

Pour plus d’informations sur les concepts impliqués dans le workflow de déploiement, consultez [Déployer, gérer et superviser des modèles avec Azure Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Prérequis

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

- Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).
- Un modèle Si vous n’avez pas de modèle entraîné, vous pouvez utiliser les fichiers de modèle et de dépendance fournis dans [ce tutoriel](https://aka.ms/azml-deploy-cloud).
- L’[extension de l’interface de ligne de commande (CLI) Azure pour le service Machine Learning](reference-azure-machine-learning-cli.md).

# <a name="python"></a>[Python](#tab/python)

- Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).
- Un modèle Si vous n’avez pas de modèle entraîné, vous pouvez utiliser les fichiers de modèle et de dépendance fournis dans [ce tutoriel](https://aka.ms/azml-deploy-cloud).
- Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).

---

## <a name="connect-to-your-workspace"></a>Se connecter à un espace de travail

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Suivez les instructions de la documentation Azure CLI pour [configurer votre contexte d’abonnement](/cli/azure/manage-azure-subscriptions-azure-cli#change-the-active-subscription).

Procédez ensuite comme suit :

```azurecli-interactive
az ml workspace list --resource-group=<my resource group>
```

pour afficher les espaces de travail auxquels vous avez accès.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core import Workspace
ws = Workspace.from_config(path=".file-path/ws_config.json")
```

Pour plus d’informations sur l’utilisation du kit SDK pour se connecter à un espace de travail, consultez la documentation sur le [kit SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true#workspace).


---


## <a name="register-your-model-optional"></a><a id="registermodel"></a> Inscription du modèle (facultatif)

Un modèle inscrit est un conteneur logique pour un ou plusieurs fichiers qui composent votre modèle. Par exemple, si vous disposez d’un modèle stocké dans plusieurs fichiers, vous pouvez enregistrer ces derniers en tant que modèle unique dans l’espace de travail. Après l’inscription des fichiers, vous pouvez télécharger ou déployer le modèle inscrit et recevoir tous les fichiers que vous avez inscrits.

> [!TIP] 
> Il est recommandé, mais non obligatoire d’inscrire un modèle pour le suivi des versions. Si vous préférez continuer sans inscrire de modèle, vous devez spécifier un répertoire source dans [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig?preserve-view=true&view=azure-ml-py) ou [inferenceconfig.json](./reference-azure-machine-learning-cli.md#inference-configuration-schema) et veiller à ce que votre modèle se trouve dans ce répertoire source.

> [!TIP]
> Quand vous inscrivez un modèle, vous fournissez le chemin d’un emplacement cloud (à partir d’une exécution d’entraînement) ou d’un répertoire local. Ce chemin sert uniquement à localiser les fichiers à charger dans le cadre du processus d’inscription. Il ne doit pas forcément correspondre au chemin utilisé dans le script d’entrée. Pour plus d’informations, consultez [Localiser les fichiers de modèles dans votre script d’entrée](./how-to-deploy-advanced-entry-script.md#load-registered-models).

> [!IMPORTANT]
> Lors de l’utilisation de l’option Filtrer par `Tags` sur la page Modèles d’Azure Machine Learning Studio au lieu de `TagName : TagValue`, les clients doivent indiquer `TagName=TagValue` (sans espace)

Les exemples suivants montrent comment inscrire un modèle.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Inscrire un modèle à partir d’une exécution de formation Azure ML

```azurecli-interactive
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --run-id myrunid --tag area=mnist
```

[!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

Le paramètre `--asset-path` fait référence à l’emplacement cloud du modèle. Dans cet exemple, le chemin d’un fichier unique est utilisé. Pour inclure plusieurs fichiers dans l’inscription du modèle, définissez `--asset-path` avec le chemin d’un dossier contenant les fichiers.

### <a name="register-a-model-from-a-local-file"></a>Inscrire un modèle à partir d’un fichier local

```azurecli-interactive
az ml model register -n onnx_mnist -p mnist/model.onnx
```

Pour inclure plusieurs fichiers dans l’inscription du modèle, définissez `-p` avec le chemin d’un dossier contenant les fichiers.

Pour plus d’informations sur `az ml model register`, consultez la [documentation de référence](/cli/azure/ext/azure-cli-ml/ml/model).

# <a name="python"></a>[Python](#tab/python)

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Inscrire un modèle à partir d’une exécution de formation Azure ML

  Quand vous utilisez le kit SDK pour entraîner un modèle, vous pouvez recevoir un objet [Run](/python/api/azureml-core/azureml.core.run.run?preserve-view=true&view=azure-ml-py) ou [AutoMLRun](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun) en fonction de la façon dont vous avez entraîné le modèle. Chaque objet peut être utilisé pour inscrire un modèle créé par un exécution d’expérimentation.

  + Inscrire un modèle à partir d’un objet `azureml.core.Run` :
 
    ```python
    model = run.register_model(model_name='sklearn_mnist',
                               tags={'area': 'mnist'},
                               model_path='outputs/sklearn_mnist_model.pkl')
    print(model.name, model.id, model.version, sep='\t')
    ```

    Le paramètre `model_path` fait référence à l’emplacement cloud du modèle. Dans cet exemple, le chemin d’un fichier unique est utilisé. Pour inclure plusieurs fichiers dans l’inscription du modèle, définissez `model_path` avec le chemin d’un dossier contenant les fichiers. Pour plus d’informations, consultez la documentation sur [Run.register_model](/python/api/azureml-core/azureml.core.run.run?preserve-view=true&view=azure-ml-py#&preserve-view=trueregister-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-).

  + Inscrire un modèle à partir d’un objet `azureml.train.automl.run.AutoMLRun` :

    ```python
        description = 'My AutoML Model'
        model = run.register_model(description = description,
                                   tags={'area': 'mnist'})

        print(run.model_id)
    ```

    Dans cet exemple, les paramètres `metric` et `iteration` n’étant pas spécifiés, l’itération avec la meilleure métrique principale est inscrite. La valeur `model_id` retournée par l’exécution est utilisée à la place d’un nom de modèle.

    Pour plus d’informations, consultez la documentation sur [AutoMLRun.register_model](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun#register-model-model-name-none--description-none--tags-none--iteration-none--metric-none-).

### <a name="register-a-model-from-a-local-file"></a>Inscrire un modèle à partir d’un fichier local

Vous pouvez inscrire un modèle en indiquant son chemin local. Vous pouvez fournir le chemin d’un dossier ou d’un seul fichier. Vous pouvez utiliser cette méthode pour inscrire les modèles qui ont été entraînés avec Azure Machine Learning et téléchargés. Vous pouvez également utiliser cette méthode pour inscrire des modèles entraînés en dehors d’Azure Machine Learning.

[!INCLUDE [trusted models](../../includes/machine-learning-service-trusted-model.md)]

+ **Utilisation du kit SDK et d’ONNX**

    ```python
    import os
    import urllib.request
    from azureml.core.model import Model
    # Download model
    onnx_model_url = "https://www.cntk.ai/OnnxModels/mnist/opset_7/mnist.tar.gz"
    urllib.request.urlretrieve(onnx_model_url, filename="mnist.tar.gz")
    os.system('tar xvzf mnist.tar.gz')
    # Register model
    model = Model.register(workspace = ws,
                            model_path ="mnist/model.onnx",
                            model_name = "onnx_mnist",
                            tags = {"onnx": "demo"},
                            description = "MNIST image classification CNN from ONNX Model Zoo",)
    ```

  Pour inclure plusieurs fichiers dans l’inscription du modèle, définissez `model_path` avec le chemin d’un dossier contenant les fichiers.

Pour plus d’informations, consultez la documentation sur la [classe Model](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py).

Pour plus d’informations sur l’utilisation de modèles formés en dehors d’Azure Machine Learning, consultez [Déployer un modèle existant](how-to-deploy-existing-model.md).

---

## <a name="define-an-entry-script"></a>Définir un script d’entrée

[!INCLUDE [write entry script](../../includes/machine-learning-entry-script.md)]


## <a name="define-an-inference-configuration"></a>Définir une configuration d’inférence


Une configuration de l’inférence décrit comment configurer le service web contenant votre modèle. Elle est utilisée ultérieurement, quand vous déployez le modèle.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Voici une configuration d’inférence minimale possible :

```json
{
    "entryScript": "score.py",
    "sourceDirectory": "./working_dir"
}
```

Elle spécifie que le déploiement utilisera le fichier `score.py` dans le répertoire `./working_dir` pour traiter les demandes entrantes.

Pour une discussion plus approfondie sur les configurations d’inférence, [consultez cet article](./reference-azure-machine-learning-cli.md#inference-configuration-schema). 

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant illustre les opérations suivantes :

1. Chargement d’un [environnement organisé](resource-curated-environments.md) à partir de l’espace de travail
1. Clonage de l’environnement
1. Spécification de `scikit-learn` comme dépendance
1. Utilisation de l’environnement pour créer une InferenceConfig

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig


env = Environment.get(workspace, "AzureML-Minimal").clone(env_name)

for pip_package in ["scikit-learn"]:
    env.python.conda_dependencies.add_pip_package(pip_package)

inference_config = InferenceConfig(entry_script='path-to-score.py',
                                    environment=env)
```

Pour plus d’informations sur les environnements , consultez [Créer et gérer des environnements pour la formation et le déploiement](how-to-use-environments.md).

Pour plus d’informations sur la configuration des inférences, consultez la documentation sur la classe [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig?preserve-view=true&view=azure-ml-py).

---

> [!TIP] 
> Pour plus d’informations sur l’utilisation d’une image Docker personnalisée avec une configuration d’inférence, consultez le [guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md).

## <a name="choose-a-compute-target"></a>Choisir une cible de calcul

[!INCLUDE [aml-compute-target-deploy](../../includes/aml-compute-target-deploy.md)]

## <a name="define-a-deployment-configuration"></a>Définir une configuration de déploiement

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Les options disponibles pour une configuration de déploiement varient en fonction de la cible de calcul que vous choisissez.

[!INCLUDE [aml-local-deploy-config](../../includes/machine-learning-service-local-deploy-config.md)]

Pour plus d’informations, consultez [cette référence](./reference-azure-machine-learning-cli.md#deployment-configuration-schema).

# <a name="python"></a>[Python](#tab/python)

Avant de déployer votre modèle, vous devez définir la configuration de déploiement. *La configuration dr déploiement est propre à la cible de calcul qui va héberger le service web.* . Par exemple, quand vous déployez un modèle localement, vous devez spécifier le port sur lequel le service accepte les requêtes. La configuration de déploiement ne fait pas partie de votre script d’entrée. Elle est utilisée pour définir les caractéristiques de la cible de calcul qui hébergera le modèle et le script d’entrée.

Vous devrez peut-être également créer la ressource de calcul si, par exemple, aucune instance AKS (Azure Kubernetes Service) n’est encore associée à votre espace de travail.

Le tableau suivant donne un exemple de configuration de déploiement créée pour chaque cible de calcul :

| Cible de calcul | Exemple de configuration de déploiement |
| ----- | ----- |
| Local | `deployment_config = LocalWebservice.deploy_configuration(port=8890)` |
| Azure Container Instances | `deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |
| Azure Kubernetes Service | `deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |

Les classes des services web Local, Azure Container Instances et AKS peuvent être importées à partir de `azureml.core.webservice` :

```python
from azureml.core.webservice import AciWebservice, AksWebservice, LocalWebservice
```

---

## <a name="deploy-your-model"></a>Déployer votre modèle

Vous êtes maintenant prêt à déployer votre modèle. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

### <a name="using-a-registered-model"></a>Utilisation d’un modèle inscrit

Si vous avez inscrit votre modèle dans votre espace de travail Azure Machine Learning, remplacez « mymodel:1 » par le nom de votre modèle et son numéro de version.

```azurecli-interactive
az ml model deploy -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json
```

### <a name="using-a-local-model"></a>Utilisation d’un modèle local

Si vous préférez ne pas inscrire votre modèle, vous pouvez transmettre le paramètre « sourceDirectory » dans votre fichier inferenceconfig.json pour spécifier un répertoire local à partir duquel servir votre modèle.

```azurecli-interactive
az ml model deploy --ic inferenceconfig.json --dc deploymentconfig.json
```

# <a name="python"></a>[Python](#tab/python)

L’exemple ci-dessous montre un déploiement local. La syntaxe varie en fonction de la cible de calcul que vous avez choisie à l’étape précédente.

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Pour plus d’informations, consultez la documentation sur [LocalWebservice](/python/api/azureml-core/azureml.core.webservice.local.localwebservice?preserve-view=true&view=azure-ml-py), [Model.deploy()](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) et [Webservice](/python/api/azureml-core/azureml.core.webservice.webservice?preserve-view=true&view=azure-ml-py).

---

### <a name="understanding-service-state"></a>Fonctionnement de l’état du service

Pendant le déploiement du modèle, vous pouvez voir le changement de l’état du service lors de son déploiement complet.

Le tableau ci-après décrit les différents états de service :

| État du service web | Description | État final ?
| ----- | ----- | ----- |
| Transition | Le service est en cours de déploiement. | Non |
| Unhealthy | Le service a été déployé, mais est actuellement inaccessible.  | Non |
| Non planifiable | Le service ne peut pas être déployé pour l’instant en raison d’un manque de ressources. | Non |
| Échec | Le déploiement du service a échoué en raison d’une erreur ou d’un plantage. | Oui |
| Healthy | Le service est sain et le point de terminaison est disponible. | Oui |


### <a name="batch-inference"></a><a id="azuremlcompute"></a> Inférence par lots
Les cibles de calcul Azure Machine Learning sont créées et managées par Azure Machine Learning. Elles peuvent être utilisées pour la prédiction par lots à partir de pipelines Azure Machine Learning.

Pour obtenir une présentation détaillée de l’inférence par lots avec la capacité de calcul Azure Machine Learning, consultez le [guide pratique pour exécuter des prédictions par lots](tutorial-pipeline-batch-scoring-classification.md).

### <a name="iot-edge-inference"></a><a id="iotedge"></a> Inférence IoT Edge
La prise en charge du déploiement en périphérie est en préversion. Pour plus d’informations, consultez [Déployer Azure Machine Learning en tant que module IoT Edge](../iot-edge/tutorial-deploy-machine-learning.md).

## <a name="delete-resources"></a>Supprimer des ressources

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

Pour supprimer un service web déployé, utilisez `az ml service <name of webservice>`.

Pour supprimer de votre espace de travail un modèle inscrit, utilisez `az ml model delete <model id>`

Apprenez-en davantage sur [la suppression d’un service web](/cli/azure/ext/azure-cli-ml/ml/service#ext-azure-cli-ml-az-ml-service-delete) et [la suppression d’un modèle](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-delete).

# <a name="python"></a>[Python](#tab/python)

Pour supprimer un service web déployé, utilisez `service.delete()`.
Pour supprimer un modèle inscrit, utilisez `model.delete()`.

Pour plus d’informations, consultez la documentation sur [WebService.delete()](/python/api/azureml-core/azureml.core.webservice%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truedelete--) et [Model.delete()](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=truedelete--).

---

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre des problèmes d’échec de déploiement](how-to-troubleshoot-deployment.md)
* [Déployer dans Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Créer des applications clientes pour utiliser des services web](how-to-consume-web-service.md)
* [Mettre à jour un service web](how-to-deploy-update-web-service.md)
* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)
* [Créer des alertes d’événement et des déclencheurs pour les déploiements de modèle](how-to-use-event-grid.md)