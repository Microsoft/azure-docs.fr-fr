---
title: Guide pratique pour effectuer localement une exécution et un déploiement
titleSuffix: Azure Machine Learning
description: Cet article vous explique comment utiliser votre ordinateur local comme cible pour l’entraînement, le débogage ou le déploiement de modèles créés dans Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 11/20/2020
ms.topic: conceptual
ms.custom: how-to, deploy
ms.openlocfilehash: 71f393897dff266f1b0922a19eefd70cffea133d
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96600351"
---
# <a name="deploy-models-trained-with-azure-machine-learning-on-your-local-machines"></a>Déployer des modèles entraînés avec Azure Machine Learning sur vos ordinateurs locaux 

Cet article vous explique comment utiliser votre ordinateur local comme cible pour l’entraînement ou le déploiement de modèles créés dans Azure Machine Learning. Grâce à sa flexibilité, Azure Machine Learning peut fonctionner avec la plupart des frameworks de Machine Learning Python. Les solutions Machine Learning ont généralement des dépendances complexes qui peuvent être difficiles à dupliquer. Cet article vous montre comment trouver un compromis entre le contrôle total et la facilité d’utilisation.

Voici certains scénarios de déploiement local :

* Itération rapide de données, de scripts et de modèles tôt dans un projet
* Débogage et résolution des problèmes au cours de phases ultérieures
* Déploiement final sur le matériel géré par l’utilisateur

## <a name="prerequisites"></a>Prérequis

- Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).
- Un modèle et un environnement. Si vous n’avez pas de modèle entraîné, vous pouvez utiliser les fichiers de modèle et de dépendance fournis dans [ce tutoriel](tutorial-train-models-with-aml.md).
- Le [kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).
- Un gestionnaire Conda, comme Anaconda ou Miniconda, si vous voulez mettre en miroir les dépendances de package d’Azure Machine Learning
- Docker, si vous souhaitez utiliser une version conteneurisée de l’environnement Azure Machine Learning

## <a name="prepare-your-local-machine"></a>Préparer votre ordinateur local

La méthode la plus fiable pour exécuter localement un modèle Azure Machine Learning consiste à utiliser une image Docker. Une image Docker fournit une expérience isolée et conteneurisée qui duplique l’environnement d’exécution Azure, à l’exception des problèmes matériels. Pour plus d’informations sur l’installation et la configuration de Docker pour les scénarios de développement, consultez [Vue d’ensemble du développement à distance Docker sur Windows](/windows/dev-environment/docker/overview).

Il est possible d’attacher un débogueur à un processus qui s’exécute dans Docker (voir [Attachement à un conteneur en cours d’exécution](https://code.visualstudio.com/docs/remote/attach-container)). Toutefois, vous pouvez également déboguer et itérer votre code Python sans utiliser Docker. Dans ce scénario, il est important que votre ordinateur local utilise les mêmes bibliothèques que celles utilisées quand vous exécutez votre expérience dans Azure Machine Learning. Pour gérer les dépendances Python, Azure utilise [Conda](https://docs.conda.io/). Vous pouvez recréer l’environnement à l’aide d’autres gestionnaires de package, mais l’installation et la configuration de Conda sur votre ordinateur local constituent le moyen le plus simple d’effectuer la synchronisation. 

## <a name="prepare-your-entry-script"></a>Préparer votre script d’entrée

Même si vous utilisez Docker pour gérer le modèle et les dépendances, le script de scoring Python doit être local. Le script doit avoir deux méthodes :

- Une méthode `init()` qui n’accepte aucun argument et ne retourne rien 
- Une méthode `run()` qui prend une chaîne au format JSON et retourne un objet sérialisable JSON.

L’argument de la méthode `run()` se présente sous la forme : 

```json
{
    "data": <model-specific-data-structure>
}
```

L’objet que vous retournez à partir de la méthode `run()` doit implémenter `toJSON() -> string`.

L’exemple suivant montre comment charger un modèle scikit-learn inscrit et le noter avec des données NumPy. Cet exemple est basé sur le modèle et les dépendances de [ce tutoriel](tutorial-train-models-with-aml.md) :

```python
import json
import numpy as np
import os
import pickle
import joblib

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It's the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION).
    # For multiple models, it points to the folder containing all deployed models (./azureml-models).
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # Make prediction.
    y_hat = model.predict(data)
    # You can return any data type as long as it's JSON-serializable.
    return y_hat.tolist()
```

Pour obtenir des exemples plus avancés, notamment la génération automatique de schémas Swagger, et savoir comment noter des données binaires (par exemple, des images), consultez [Création de scripts d’entrée avancés](how-to-deploy-advanced-entry-script.md). 

## <a name="deploy-as-a-local-web-service-by-using-docker"></a>Déployer en tant que service web local à l’aide de Docker

Le moyen le plus simple de répliquer l’environnement utilisé par Azure Machine Learning consiste à déployer un service web à l’aide de Docker. Avec Docker en cours d’exécution sur votre ordinateur local, vous allez effectuer les opérations suivantes :

1. Vous connecter à l’espace de travail Azure Machine Learning dans lequel votre modèle est inscrit.
1. Créer un objet `Model` qui représente le modèle.
1. Créer un objet `Environment` qui contient les dépendances et définit l’environnement logiciel dans lequel votre code s’exécutera.
1. Créer un objet `InferenceConfig` qui associe le script d’entrée et l’`Environment`.
1. Créer un objet `DeploymentConfiguration` de la sous-classe `LocalWebserviceDeploymentConfiguration`.
1. Utiliser `Model.deploy()` pour créer un objet `Webservice`. Cette méthode télécharge l’image Docker et l’associe aux objets `Model`, `InferenceConfig` et `DeploymentConfiguration`.
1. Activez le `Webservice` à l’aide de `Webservice.wait_for_deployment()`.

Le code suivant illustre ces étapes :

```python
from azureml.core.webservice import Webservice
from azure.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model = Model(ws, 'sklearn_mnist')


myenv = Environment.get(workspace=ws, name="tutorial-env", version="1")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

deployment_config = LocalWebservice.deploy_configuration(port=6789)

local_service = Model.deploy(workspace=ws, 
                       name='sklearn-mnist-local', 
                       models=[model], 
                       inference_config=inference_config, 
                       deployment_config = deployment_config)

local_service.wait_for_deployment(show_output=True)
print(f"Scoring URI is : {local_service.scoring_uri}")
```

L’appel à `Model.deploy()` peut prendre quelques minutes. Une fois le déploiement initial du service web effectué, il est plus efficace d’utiliser la méthode `update()` plutôt que de partir de rien. Consultez [Mettre à jour un service web déployé](how-to-deploy-update-web-service.md).

### <a name="test-your-local-deployment"></a>Tester votre déploiement local

Quand vous exécutez le script de déploiement précédent, il génère l’URI dans lequel vous pouvez publier (POST) des données pour le scoring (par exemple, `http://localhost:6789/score`). L’exemple suivant montre un script qui effectue la notation d’exemples de données avec le modèle `"sklearn-mnist-local"` déployé localement. Le modèle, s’il est correctement entraîné, déduit que `normalized_pixel_values` doit être interprété comme un « 2 ». 

```python
import requests

normalized_pixel_values = "[\
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 0.5, 0.7, 1.0, 1.0, 0.6, 0.4, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.9, 0.1, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 1.0, 1.0, 1.0, 0.8, 0.6, 0.7, 1.0, 1.0, 0.5, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.2, 1.0, 1.0, 0.8, 0.1, 0.0, 0.0, 0.0, 0.8, 1.0, 0.5, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.3, 1.0, 0.8, 0.1, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, 0.3, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.1, 0.0, 0.0, 0.0, 0.0, 0.8, 1.0, 1.0, 0.3, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, 0.8, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.3, 1.0, 1.0, 0.9, 0.2, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, 0.6, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 1.0, 1.0, 0.6, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.9, 1.0, 0.9, 0.1, \
0.1, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.8, 1.0, 1.0, 0.6, \
0.6, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.3, 1.0, 1.0, 0.7, \
0.7, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.8, 1.0, 1.0, \
1.0, 0.6, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, \
1.0, 0.7, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 1.0, \
1.0, 1.0, 0.1, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, \
1.0, 1.0, 1.0, 0.2, 0.1, 0.1, 0.1, 0.1, 0.0, 0.0, 0.0, 0.1, 0.1, 0.1, 0.6, 0.6, 0.6, 0.6, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.7, 0.6, 0.7, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.7, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.7, 0.5, 0.5, 0.2, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.5, 0.5, 0.5, 0.5, 0.7, 1.0, 1.0, 1.0, 0.6, 0.5, 0.5, 0.2, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]"

input_data = "{\"data\": [" + normalized_pixel_values + "]}"

headers = {'Content-Type': 'application/json'}

scoring_uri = "http://localhost:6789/score"
resp = requests.post(scoring_uri, input_data, headers=headers)

print("Should be predicted as '2'")
print("prediction:", resp.text)
```

## <a name="download-and-run-your-model-directly"></a>Télécharger et exécuter votre modèle directement

L’utilisation de Docker pour déployer votre modèle en tant que service web est l’option la plus courante. Vous pouvez toutefois exécuter votre code directement à l’aide de scripts Python locaux. Vous aurez besoin de deux composants importants : 

- Le modèle lui-même
- Les dépendances sur lesquelles repose le modèle 

Vous pouvez télécharger le modèle comme suit :  

- À partir du portail, en choisissant l’onglet **Modèles**, en sélectionnant le modèle souhaité, puis dans la page **Détails**, en choisissant **Télécharger**.
- À partir de la ligne de commande, à l’aide de `az ml model download` (voir [Téléchargement de modèle](/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest#ext_azure_cli_ml_az_ml_model_download&preserve-view=false)).
- À l’aide de la méthode `Model.download()` du SDK Python (voir [Classe de modèle](/python/api/azureml-core/azureml.core.model.model?view=azure-ml-py#download-target-dir------exist-ok-false--exists-ok-none-&preserve-view=false)).

Un modèle Azure est constitué d’un ou de plusieurs objets Python sérialisés, empaquetés sous forme de fichier pickle Python (extension .pkl). Le contenu du fichier pickle dépend de la bibliothèque ou technique Machine Learning utilisée pour entraîner le modèle. Par exemple, si vous utilisez le modèle utilisé dans le tutoriel, vous pouvez charger le modèle avec :

```python
import pickle

with open('sklearn_mnist_model.pkl', 'rb') as f : 
    logistic_model = pickle.load(f, encoding='latin1')
```

Les dépendances sont toujours difficiles à mettre en œuvre, surtout avec le Machine Learning, où il peut souvent y avoir un web vertigineux d’exigences spécifiques liées à la version. Vous pouvez recréer un environnement Azure Machine Learning sur votre ordinateur local en tant qu’environnement Conda complet ou en tant qu’image Docker à l’aide de la méthode `build_local()` de la classe `Environment` : 

```python
ws = Workspace.from_config()
myenv = Environment.get(workspace=ws, name="tutorial-env", version="1")
myenv.build_local(workspace=ws, useDocker=False) #Creates conda environment.
```

Si vous définissez l’argument `build_local()` `useDocker` sur `True`, la fonction crée une image Docker plutôt qu’un environnement Conda. Si vous souhaitez davantage de contrôle, vous pouvez utiliser la méthode `save_to_directory()` de `Environment`, qui écrit des fichiers de définition conda_dependencies.yml et azureml_environment.json que vous pouvez affiner et utiliser comme base pour l’extension. 

La classe `Environment` comprend un certain nombre d’autres méthodes pour synchroniser les environnements sur votre matériel de calcul, votre espace de travail Azure et les images Docker. Pour en savoir plus, consultez [Classe d’environnement](/python/api/azureml-core/azureml.core.environment(class)).

Une fois que vous avez téléchargé le modèle et résolu ses dépendances, il n’existe aucune restriction définie par Azure sur la façon dont vous effectuez le scoring, dont vous ajustez le modèle, dont vous utilisez l’apprentissage de transfert, etc. 

## <a name="upload-a-retrained-model-to-azure-machine-learning"></a>Charger un modèle réentraîné sur Azure Machine Learning

Si vous disposez d’un modèle entraîné ou réentraîné localement, vous pouvez l’inscrire auprès d’Azure. Une fois l’inscription du modèle effectuée, vous pouvez poursuivre son réglage à l’aide du calcul Azure ou le déployer à l’aide de fonctionnalités Azure, comme le service [Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md) ou le serveur [Triton Inference Server (préversion)](how-to-deploy-with-triton.md).

Pour être utilisé avec le SDK Python d’Azure Machine Learning, un modèle doit être stocké en tant qu’objet Python sérialisé au format pickle (fichier .pkl). Il doit également implémenter une méthode `predict(data)` qui retourne un objet sérialisable JSON. Par exemple, vous pouvez stocker un modèle de diabète scikit-learn entraîné localement avec : 

```python
import joblib

from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge

dataset_x, dataset_y = load_diabetes(return_X_y=True)

sk_model = Ridge().fit(dataset_x, dataset_y)

joblib.dump(sk_model, "sklearn_regression_model.pkl")
```

Pour rendre le modèle disponible dans Azure, vous pouvez utiliser la méthode `register()` de la classe `Model` :

```python
from azureml.core.model import Model

model = Model.register(model_path="sklearn_regression_model.pkl",
                       model_name="sklearn_regression_model",
                       tags={'area': "diabetes", 'type': "regression"},
                       description="Ridge regression model to predict diabetes",
                       workspace=ws)
```

Vous pouvez ensuite rechercher votre modèle nouvellement inscrit sous l’onglet **Modèle** d’Azure Machine Learning :

:::image type="content" source="media/how-to-deploy-local/registered-model.png" alt-text="Capture d’écran de l’onglet Modèle d’Azure Machine Learning, montrant le modèle chargé.":::

Pour plus d’informations sur le chargement et la mise à jour des modèles et des environnements, consultez la rubrique indiquant comment [inscrire un modèle et effectuer un déploiement localement avec des utilisations avancées](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/deploy-to-local/register-model-deploy-local-advanced.ipynb).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les environnements de gestion, consultez [Créer et utiliser des environnements logiciels dans Azure Machine Learning](how-to-use-environments.md).
- Pour en savoir plus sur l’accès aux données à partir de votre magasin de données, consultez [Se connecter à des services de stockage sur Azure](how-to-access-data.md).
