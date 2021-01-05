---
title: Créer un script d’entrée pour les scénarios avancés
titleSuffix: Azure Machine Learning entry script authoring
description: Découvrez comment écrire des scripts d’entrée Azure Machine Learning à des fins de pré-traitement et de post-traitement au cours du déploiement.
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 09/17/2020
ms.author: gopalv
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: 5b05891500ae5fd66e5ec2381066ccd1d26aa7ec
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94578057"
---
# <a name="advanced-entry-script-authoring"></a>Création de scripts d’entrée avancés

Cet article explique comment écrire des scripts d’entrée pour des cas d’usage spécialisés.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous disposez déjà d’un modèle Machine Learning formé que vous envisagez de déployer avec Azure Machine Learning. Pour en savoir plus sur le déploiement de modèles, consultez [ce didacticiel](how-to-deploy-and-where.md).

## <a name="automatically-generate-a-swagger-schema"></a>Générer automatiquement un schéma Swagger

Si vous voulez générer automatiquement un schéma pour votre service web, spécifiez un exemple d’entrée et/ou de sortie dans le constructeur pour l’un des objets de type définis. Le type et l’exemple sont utilisés pour créer automatiquement le schéma. Azure Machine Learning crée ensuite une spécification (Swagger) [OpenAPI](https://swagger.io/docs/specification/about/) pour le service web pendant le déploiement. 

> [!WARNING]
> Vous ne devez pas utiliser de données sensibles ou privées comme exemples d’entrée ou de sortie. La page Swagger pour l’inférence hébergée par AML expose les exemples de données. 

Les types suivants sont pris en charge :

* `pandas`
* `numpy`
* `pyspark`
* Objet Python standard

Pour utiliser la génération de schéma, incluez le package `inference-schema` open source version 1.1.0 ou ultérieure dans votre fichier de dépendances. Pour plus d’informations sur ce package, consultez [https://github.com/Azure/InferenceSchema](https://github.com/Azure/InferenceSchema). Pour générer une consommation de service web automatisé swagger conforme, la fonction run() du script de scoring doit avoir la forme d’API suivante :
* Un premier paramètre de type « StandardPythonParameterType », nommé Inputs, imbriqué et contenant PandasDataframeParameterTypes.
* Deuxième paramètre facultatif de type « StandardPythonParameterType », nommé GlobalParameter, qui n’est pas imbriqué.
* Retourner un dictionnaire de type « StandardPythonParameterType », qui peut être imbriqué et contenant PandasDataFrameParameterTypes.
Définissez les exemples de formats d’entrée et de sortie dans les variables `input_sample` et `output_sample`, qui représentent les formats de requête et de réponse pour le service web. Utilisez ces exemples dans les éléments décoratifs des fonctions d’entrée et de sortie sur la fonction `run()`. L’exemple scikit-learn suivant utilise la génération de schéma.



```python
#Example: scikit-learn and Swagger
import json
import numpy as np
import os
from sklearn.externals import joblib
from sklearn.linear_model import Ridge

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType


def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment. Join this path with the filename of the model file.
    # It holds the path to the directory that contains the deployed model (./azureml-models/$MODEL_NAME/$VERSION).
    # If there are multiple models, this value is the path to the directory containing all deployed models (./azureml-models).
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')

    # If your model were stored in the same directory as your score.py, you could also use the following:
    # model_path = os.path.abspath(os.path.join(os.path.dirname(__file_), 'sklearn_mnist_model.pkl')

    # Deserialize the model file back into a sklearn model
    model = joblib.load(model_path)


input_sample = np.array([[10, 9, 8, 7, 6, 5, 4, 3, 2, 1]])
output_sample = np.array([3726.995])


@input_schema('data', NumpyParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

## <a name="power-bi-compatible-endpoint"></a>Point de terminaison compatible Power BI 

L’exemple suivant montre comment définir la forme d’API conformément à l’instruction ci-dessus. Cette méthode est prise en charge pour l’utilisation du service web déployé à partir de Power BI. ([Découvrez-en plus sur l’utilisation du service web à partir de Power BI](/power-bi/service-machine-learning-integration).)

```python
import json
import pickle
import numpy as np
import pandas as pd
import azureml.train.automl
from sklearn.externals import joblib
from sklearn.linear_model import Ridge

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.standard_py_parameter_type import StandardPythonParameterType
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.pandas_parameter_type import PandasParameterType


def init():
    global model
    # Replace filename if needed.
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_regression_model.pkl')
    # Deserialize the model file back into a sklearn model.
    model = joblib.load(model_path)

# providing 3 sample inputs for schema generation
numpy_sample_input = NumpyParameterType(np.array([[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]],dtype='float64'))
pandas_sample_input = PandasParameterType(pd.DataFrame({'name': ['Sarah', 'John'], 'age': [25, 26]}))
standard_sample_input = StandardPythonParameterType(0.0)

# This is a nested input sample, any item wrapped by `ParameterType` will be described by schema
sample_input = StandardPythonParameterType({'input1': numpy_sample_input, 
                                            'input2': pandas_sample_input, 
                                            'input3': standard_sample_input})

sample_global_parameters = StandardPythonParameterType(1.0) #this is optional
sample_output = StandardPythonParameterType([1.0, 1.0])

@input_schema('inputs', sample_input)
@input_schema('global_parameters', sample_global_parameters) #this is optional
@output_schema(sample_output)
def run(inputs, global_parameters):
    try:
        data = inputs['input1']
        # data will be convert to target format
        assert isinstance(data, np.ndarray)
        result = model.predict(data)
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```


## <a name="binary-ie-image-data"></a><a id="binary-data"></a> Données binaires (image)

Si votre modèle accepte les données binaires, comme une image, vous devez modifier le fichier `score.py` utilisé pour votre déploiement afin d’accepter les demandes HTTP brutes. Pour accepter des données brutes, utilisez la classe `AMLRequest` dans votre script d’entrée et ajoutez l’ élément décoratif `@rawhttp` à la fonction `run()`.

Voici l’exemple d’un `score.py` qui accepte des données binaires :

```python
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse


def init():
    print("This is init()")


@rawhttp
def run(request):
    print("This is run()")
    print("Request: [{0}]".format(request))
    if request.method == 'GET':
        # For this example, just return the URL for GETs.
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        reqBody = request.get_data(False)
        # For a real-world solution, you would load the data from reqBody
        # and send it to the model. Then return the response.

        # For demonstration purposes, this example just returns the posted data as the response.
        return AMLResponse(reqBody, 200)
    else:
        return AMLResponse("bad request", 500)
```

> [!IMPORTANT]
> La classe `AMLRequest` se trouve dans l’espace de noms `azureml.contrib`. Les entités dans cet espace de noms changent fréquemment car nous cherchons à améliorer le service. Tout ce qui est compris dans cet espace de noms doit être considéré comme une préversion et n’est pas entièrement pris en charge par Microsoft.
>
> Si vous avez besoin d’effectuer un test dans votre environnement de développement local, vous pouvez installer les composants à l’aide de la commande suivante :
>
> ```shell
> pip install azureml-contrib-services
> ```

La classe `AMLRequest` vous permet d’accéder uniquement aux données publiées brutes dans score.py. Il n’y a aucun composant côté client. À partir d’un client, vous publiez des données normalement. Par exemple, le code Python suivant lit un fichier image et publie les données :

```python
import requests
# Load image data
data = open('example.jpg', 'rb').read()
# Post raw data to scoring URI
res = requests.post(url='<scoring-uri>', data=data, headers={'Content-Type': 'application/octet-stream'})
```

<a id="cors"></a>

## <a name="cross-origin-resource-sharing-cors"></a>Partage des ressources cross-origin (CORS)

Le partage des ressources Cross-Origin permet de demander des ressources dans une page web à partir d’un autre domaine. CORS fonctionne par le biais d’en-têtes HTTP envoyés avec la demande du client et retournés avec la réponse du service. Pour plus d’informations sur CORS et les en-têtes valides, consultez [Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) sur Wikipédia.

Pour configurer que votre modèle de déploiement prend en charge le partage des ressources Cross-Origin, utilisez la classe `AMLResponse` dans votre script d’entrée. Cette classe vous permet de définir les en-têtes de l’objet de réponse.

L’exemple suivant définit l’en-tête `Access-Control-Allow-Origin` de la réponse à partir du script d’entrée :

```python
from azureml.contrib.services.aml_request import AMLRequest, rawhttp
from azureml.contrib.services.aml_response import AMLResponse

def init():
    print("This is init()")

@rawhttp
def run(request):
    print("This is run()")
    print("Request: [{0}]".format(request))
    if request.method == 'GET':
        # For this example, just return the URL for GETs.
        respBody = str.encode(request.full_path)
        return AMLResponse(respBody, 200)
    elif request.method == 'POST':
        reqBody = request.get_data(False)
        # For a real-world solution, you would load the data from reqBody
        # and send it to the model. Then return the response.

        # For demonstration purposes, this example
        # adds a header and returns the request body.
        resp = AMLResponse(reqBody, 200)
        resp.headers['Access-Control-Allow-Origin'] = "http://www.example.com"
        return resp
    else:
        return AMLResponse("bad request", 500)
```

> [!IMPORTANT]
> La classe `AMLResponse` se trouve dans l’espace de noms `azureml.contrib`. Les entités dans cet espace de noms changent fréquemment car nous cherchons à améliorer le service. Tout ce qui est compris dans cet espace de noms doit être considéré comme une préversion et n’est pas entièrement pris en charge par Microsoft.
>
> Si vous avez besoin d’effectuer un test dans votre environnement de développement local, vous pouvez installer les composants à l’aide de la commande suivante :
>
> ```shell
> pip install azureml-contrib-services
> ```


> [!WARNING]
> Azure Machine Learning achemine uniquement les requêtes POST et GET vers les conteneurs exécutant le service de scoring. Cela peut provoquer des erreurs dues à des navigateurs qui utilisent des requêtes OPTIONS pour des requêtes CORS préalables.
> 


## <a name="load-registered-models"></a>Charger les modèles inscrits

Il existe deux façons de localiser les modèles dans votre script d’entrée :
* `AZUREML_MODEL_DIR`: variable d’environnement contenant le chemin de l’emplacement du modèle.
* `Model.get_model_path`: API qui retourne le chemin du fichier du modèle d’après le nom de modèle inscrit.

#### <a name="azureml_model_dir"></a>AZUREML_MODEL_DIR

AZUREML_MODEL_DIR est une variable d’environnement créée au moment du déploiement du service. Vous pouvez utiliser cette variable d’environnement pour rechercher l’emplacement de chaque modèle déployé.

Le tableau suivant indique la valeur de la variable AZUREML_MODEL_DIR en fonction du nombre de modèles déployés :

| Déploiement | Valeur de la variable d’environnement |
| ----- | ----- |
| Modèle unique | Chemin du dossier contenant le modèle. |
| Modèles multiples | Chemin du dossier contenant tous les modèles. Les modèles sont localisés par nom et version dans ce dossier (`$MODEL_NAME/$VERSION`) |

Pendant l'inscription et le déploiement des modèles, ceux-ci sont placés à l'emplacement AZUREML_MODEL_DIR et ils conservent leur nom de fichier d'origine.

Pour obtenir le chemin d'accès à un fichier modèle dans votre script d'entrée, combinez la variable d'environnement avec le chemin du fichier que vous recherchez.

**Exemple avec un modèle unique**
```python
# Example when the model is a file
model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_regression_model.pkl')

# Example when the model is a folder containing a file
file_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'my_model_folder', 'sklearn_regression_model.pkl')
```

**Exemple avec des modèles multiples**

Dans ce scénario, deux modèles sont inscrits auprès de l’espace de travail :

* `my_first_model`: contient un fichier (`my_first_model.pkl`) et il n’existe qu’une seule version (`1`).
* `my_second_model`: contient un fichier (`my_second_model.pkl`) et il existe deux versions : `1` et `2`.

Quand le service est déployé, les deux modèles sont fournis dans l’opération de déploiement :

```python
first_model = Model(ws, name="my_first_model", version=1)
second_model = Model(ws, name="my_second_model", version=2)
service = Model.deploy(ws, "myservice", [first_model, second_model], inference_config, deployment_config)
```

Dans l’image Docker qui héberge le service, la variable d’environnement `AZUREML_MODEL_DIR` contient le répertoire où se trouvent les modèles.
Dans ce répertoire, chacun des modèles se trouve à un emplacement de répertoire `MODEL_NAME/VERSION`. Où `MODEL_NAME` est le nom du modèle inscrit et `VERSION` est la version du modèle. Les fichiers qui composent le modèle inscrit sont stockés dans ces répertoires.

Dans cet exemple, les chemins seraient `$AZUREML_MODEL_DIR/my_first_model/1/my_first_model.pkl` et `$AZUREML_MODEL_DIR/my_second_model/2/my_second_model.pkl`.


```python
# Example when the model is a file, and the deployment contains multiple models
first_model_name = 'my_first_model'
first_model_version = '1'
first_model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), first_model_name, first_model_version, 'my_first_model.pkl')
second_model_name = 'my_second_model'
second_model_version = '2'
second_model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), second_model_name, second_model_version, 'my_second_model.pkl')
```

### <a name="get_model_path"></a>get_model_path

Quand vous inscrivez un modèle, vous fournissez un nom de modèle qui est utilisé pour gérer le modèle dans le Registre. Vous utilisez ce nom avec la méthode [Model.get_model_path()](/python/api/azureml-core/azureml.core.model.model?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-model-path-model-name--version-none---workspace-none-) pour récupérer le chemin du ou des fichiers de modèle sur le système de fichiers local. Si vous inscrivez un dossier ou une collection de fichiers, cette API retourne le chemin du répertoire contenant ces fichiers.

Quand vous inscrivez un modèle, vous lui donnez un nom. Le nom correspond à l’endroit où le modèle est placé, localement ou durant le déploiement du service.

## <a name="framework-specific-examples"></a>Exemples spécifiques de l’infrastructure

Vous trouverez des exemples de scripts supplémentaires pour des cas d’utilisation spécifiques de l’apprentissage automatique ci-dessous :

* [PyTorch](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch)
* [TensorFlow](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow)
* [Keras](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras/train-hyperparameter-tune-deploy-with-keras.yml)
* [AutoML](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features)
* [ONNX](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx/)

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