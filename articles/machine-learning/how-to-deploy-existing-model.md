---
title: Utiliser et déployer des modèles existants
titleSuffix: Azure Machine Learning
description: Découvrez comment intégrer des modèles ML entraînés en dehors d’Azure dans le cloud Azure avec Azure Machine Learning. Déployez ensuite le modèle en tant que service web hébergé ou module IoT.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 07/17/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy
ms.openlocfilehash: 46b8f153e65f436fa1062a0606e0fb0136d972a5
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97824597"
---
# <a name="deploy-your-existing-model-with-azure-machine-learning"></a>Déployer votre modèle existant avec Azure Machine Learning


Dans cet article, vous allez apprendre à inscrire et à déployer un modèle Machine Learning que vous avez entraîné en dehors d’Azure Machine Learning. Vous pouvez déployer en tant que service web ou sur un appareil IoT Edge.  Une fois le déploiement terminé, vous pouvez surveiller votre modèle et détecter toute dérive des données dans Azure Machine Learning. 

Pour plus d’informations sur les concepts et les termes utilisés dans cet article, consultez [Gérer, déployer et surveiller des modèles Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Prérequis

* [Un espace de travail Azure Machine Learning](how-to-manage-workspace.md)
  + Les exemples Python supposent que la variable `ws` est définie sur votre espace de travail Azure Machine Learning. Pour plus d’informations sur la procédure de connexion à l’espace de travail, reportez-vous à la [documentation sur le kit SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#&preserve-view=trueworkspace).
  
  + Les exemples de l’interface CLI utilisent les espaces réservés `myworkspace` et `myresourcegroup`, que vous devez remplacer par le nom de votre espace de travail et le groupe de ressources qui le contient.

* Le [kit SDK Python d’Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).  

* [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) et [Extension CLI Machine Learning](reference-azure-machine-learning-cli.md).

* Un modèle entraîné. Le modèle doit être conservé dans un ou plusieurs fichiers de votre environnement de développement. <br><br>Pour illustrer l’inscription d’un modèle entraîné, l’exemple de code fourni dans cet article utilise les modèles issus du [projet d’analyse des sentiments sur Twitter de Paolo Ripamonti](https://www.kaggle.com/paoloripamonti/twitter-sentiment-analysis).

## <a name="register-the-models"></a>Inscrire les modèles

L’inscription d’un modèle vous permet de stocker, de gérer la version et d’effectuer le suivi des métadonnées sur les modèles dans votre espace de travail. Dans les exemples Python et CLI suivants, le répertoire `models` contient les fichiers `model.h5`, `model.w2v`, `encoder.pkl` et `tokenizer.pkl`. Cet exemple charge les fichiers contenus dans le répertoire `models` comme une nouvelle inscription de modèle nommée `sentiment` :

```python
from azureml.core.model import Model
# Tip: When model_path is set to a directory, you can use the child_paths parameter to include
#      only some of the files from the directory
model = Model.register(model_path = "./models",
                       model_name = "sentiment",
                       description = "Sentiment analysis model trained outside Azure Machine Learning",
                       workspace = ws)
```

Pour plus d’informations, consultez les informations de référence sur [Model.register()](/python/api/azureml-core/azureml.core.model%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=trueregister-workspace--model-path--model-name--tags-none--properties-none--description-none--datasets-none--model-framework-none--model-framework-version-none--child-paths-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none-).

```azurecli
az ml model register -p ./models -n sentiment -w myworkspace -g myresourcegroup
```

> [!TIP]
> Vous pouvez également définir des objets de dictionnaire `tags` et `properties` dans le modèle inscrit. Ces valeurs peuvent être utilisées ultérieurement pour identifier un modèle spécifique. Par exemple, le framework utilisé, les paramètres d’entraînement, etc.

Pour plus d’informations, consultez les informations de référence sur [az ml model register](/cli/azure/ext/azure-cli-ml/ml/model?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-register).


Pour plus d’informations sur l’inscription de modèle en général, consultez [Gérer, déployer et superviser des modèles d’apprentissage automatique](concept-model-management-and-deployment.md).

## <a name="define-inference-configuration"></a>Définir la configuration de l’inférence

La configuration de l’inférence définit l’environnement utilisé pour exécuter le modèle déployé. La configuration de l’inférence référence les entités suivantes qui sont utilisés pour exécuter le modèle quand il est déployé :

* Un script d’entrée, nommé `score.py`, charge le modèle au démarrage du service déployé. Ce script est également responsable de la réception de données, de leur transmission au modèle et du renvoi d’une réponse.
* Un [environnement](how-to-use-environments.md) Azure Machine Learning. Un environnement définit les dépendances logicielles nécessaires pour exécuter le modèle et le script d’entrée.

L’exemple suivant montre comment utiliser le SDK pour créer un environnement, puis l’utiliser avec une configuration d’inférence :

```python
from azureml.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# Create the environment
myenv = Environment(name="myenv")
conda_dep = CondaDependencies()

# Define the packages needed by the model and scripts
conda_dep.add_conda_package("tensorflow")
conda_dep.add_conda_package("numpy")
conda_dep.add_conda_package("scikit-learn")
# You must list azureml-defaults as a pip dependency
conda_dep.add_pip_package("azureml-defaults")
conda_dep.add_pip_package("keras")
conda_dep.add_pip_package("gensim")

# Adds dependencies to PythonSection of myenv
myenv.python.conda_dependencies=conda_dep

inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
```

Pour plus d’informations, consultez les articles suivants :

+ [Comment utiliser des environnements](how-to-use-environments.md).
+ Informations de référence sur [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig?preserve-view=true&view=azure-ml-py).


L’interface CLI charge la configuration de l’inférence à partir d’un fichier YAML :

```yaml
{
   "entryScript": "score.py",
   "runtime": "python",
   "condaFile": "myenv.yml"
}
```

Avec l’interface CLI, l’environnement conda est défini dans le fichier `myenv.yml` référencé par la configuration d’inférence. Le YAML suivant constitue le contenu de ce fichier :

```yaml
name: inference_environment
dependencies:
- python=3.6.2
- tensorflow
- numpy
- scikit-learn
- pip:
    - azureml-defaults
    - keras
    - gensim
```

Pour plus d’informations sur la configuration de l’inférence, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

### <a name="entry-script-scorepy"></a>Script d’entrée (score.py)

Le script d’entrée a uniquement deux fonctions requises, `init()` et `run(data)`. Ces fonctions sont utilisées pour initialiser le service au démarrage et exécuter le modèle à l’aide des données de la demande transmises par un client. Le reste du script gère le chargement et l’exécution des modèles.

> [!IMPORTANT]
> Il n’existe pas de script d’entrée générique qui fonctionne pour tous les modèles. Il est toujours spécifique au modèle qui est utilisé. Il doit comprendre comment charger le modèle, le format de données attendu par le modèle et comment noter les données à l’aide du modèle.

Le code Python suivant est un exemple de script d’entrée (`score.py`) :

```python
import os
import pickle
import json
import time
from keras.models import load_model
from keras.preprocessing.sequence import pad_sequences
from gensim.models.word2vec import Word2Vec

# SENTIMENT
POSITIVE = "POSITIVE"
NEGATIVE = "NEGATIVE"
NEUTRAL = "NEUTRAL"
SENTIMENT_THRESHOLDS = (0.4, 0.7)
SEQUENCE_LENGTH = 300

# Called when the deployed service starts
def init():
    global model
    global tokenizer
    global encoder
    global w2v_model

    # Get the path where the deployed model can be found.
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), './models')
    # load models
    model = load_model(model_path + '/model.h5')
    w2v_model = Word2Vec.load(model_path + '/model.w2v')

    with open(model_path + '/tokenizer.pkl','rb') as handle:
        tokenizer = pickle.load(handle)

    with open(model_path + '/encoder.pkl','rb') as handle:
        encoder = pickle.load(handle)

# Handle requests to the service
def run(data):
    try:
        # Pick out the text property of the JSON request.
        # This expects a request in the form of {"text": "some text to score for sentiment"}
        data = json.loads(data)
        prediction = predict(data['text'])
        #Return prediction
        return prediction
    except Exception as e:
        error = str(e)
        return error

# Determine sentiment from score
def decode_sentiment(score, include_neutral=True):
    if include_neutral:
        label = NEUTRAL
        if score <= SENTIMENT_THRESHOLDS[0]:
            label = NEGATIVE
        elif score >= SENTIMENT_THRESHOLDS[1]:
            label = POSITIVE
        return label
    else:
        return NEGATIVE if score < 0.5 else POSITIVE

# Predict sentiment using the model
def predict(text, include_neutral=True):
    start_at = time.time()
    # Tokenize text
    x_test = pad_sequences(tokenizer.texts_to_sequences([text]), maxlen=SEQUENCE_LENGTH)
    # Predict
    score = model.predict([x_test])[0]
    # Decode sentiment
    label = decode_sentiment(score, include_neutral=include_neutral)

    return {"label": label, "score": float(score),
       "elapsed_time": time.time()-start_at}  
```

Pour plus d’informations sur les scripts d’entrée, consultez [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="define-deployment"></a>Définir le déploiement

Le package [Webservice](/python/api/azureml-core/azureml.core.webservice?preserve-view=true&view=azure-ml-py) contient les classes utilisées pour le déploiement. La classe que vous utilisez détermine où le modèle est déployé. Par exemple, pour déployer un modèle comme un service web sur Azure Kubernetes Service, utilisez [AksWebService.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.akswebservice?view=azure-ml-py&preserve-view=true#&preserve-view=truedeploy-configuration-autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--primary-key-none--secondary-key-none--tags-none--properties-none--description-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none--compute-target-name-none-) pour créer la configuration de déploiement.

Le code Python suivant définit une configuration de déploiement pour un déploiement local. Cette configuration déploie le modèle sous la forme d’un service web sur votre ordinateur local.

> [!IMPORTANT]
> Un déploiement local nécessite une installation opérationnelle de [Docker](https://www.docker.com/) sur votre ordinateur local :

```python
from azureml.core.webservice import LocalWebservice

deployment_config = LocalWebservice.deploy_configuration()
```

Pour plus d’informations, consultez les informations de référence sur [LocalWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.localwebservice?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-configuration-port-none-).

L’interface CLI charge la configuration de déploiement à partir d’un fichier YAML :

```YAML
{
    "computeType": "LOCAL"
}
```

Le déploiement sur une autre cible de calcul, par exemple Azure Kubernetes Service dans le cloud Azure, est aussi simple que la modification de la configuration de déploiement. Pour plus d’informations, consultez [Comment et où déployer des modèles ?](how-to-deploy-and-where.md).

## <a name="deploy-the-model"></a>Déployer le modèle

L’exemple suivant charge des informations sur le modèle inscrit nommé `sentiment`, puis le déploie en tant que service nommé `sentiment`. Pendant le déploiement, la configuration de l’inférence et la configuration de déploiement sont utilisées pour créer et configurer l’environnement de service :

```python
from azureml.core.model import Model

model = Model(ws, name='sentiment')
service = Model.deploy(ws, 'myservice', [model], inference_config, deployment_config)

service.wait_for_deployment(True)
print(service.state)
print("scoring URI: " + service.scoring_uri)
```

Pour plus d’informations, consultez les informations de référence sur [Model.deploy()](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=truedeploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-).

Pour déployer le modèle à partir de l’interface CLI, utilisez la commande suivante. Cette commande déploie la version 1 du modèle inscrit (`sentiment:1`) à l’aide des configurations de l’inférence et de déploiement stockées dans les fichiers `inferenceConfig.json` et `deploymentConfig.json` :

```azurecli
az ml model deploy -n myservice -m sentiment:1 --ic inferenceConfig.json --dc deploymentConfig.json
```

Pour plus d’informations, consultez les informations de référence sur [az ml model deploy](/cli/azure/ext/azure-cli-ml/ml/model?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-model-deploy).

Pour plus d’informations sur le déploiement, consultez [Comment et où déployer des modèles ?](how-to-deploy-and-where.md).

## <a name="request-response-consumption"></a>Consommation de requête-réponse

Après le déploiement, l’URI de scoring s’affiche. Cet URI peut être utilisé par les clients pour envoyer des demandes au service. L’exemple suivant est un client Python simple qui envoie des données au service et affiche la réponse :

```python
import requests
import json

scoring_uri = 'scoring uri for your service'
headers = {'Content-Type':'application/json'}

test_data = json.dumps({'text': 'Today is a great day!'})

response = requests.post(scoring_uri, data=test_data, headers=headers)
print(response.status_code)
print(response.elapsed)
print(response.json())
```

Pour plus d’informations sur le mode d’utilisation du service déployé, consultez [Créer un client](how-to-consume-web-service.md).

## <a name="next-steps"></a>Étapes suivantes

* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)
* [Guide pratique pour créer un client pour un modèle déployé](how-to-consume-web-service.md)