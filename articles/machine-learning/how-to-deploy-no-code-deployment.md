---
title: Déploiement sans code (préversion)
titleSuffix: Azure Machine Learning
description: Aucun déploiement de code ne vous permet de déployer un modèle en tant que service web sans devoir créer manuellement un script d’entrée.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
ms.custom: deploy
ms.reviewer: larryfr
ms.openlocfilehash: a17126695aa5138d1df7fd17cfaa2f5f75ad1004
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93324917"
---
# <a name="preview-no-code-model-deployment"></a>(Préversion) Modèle de déploiement sans code

Le modèle de déploiement sans code est actuellement en version préliminaire et prend en charge les infrastructures d’apprentissage automatique suivants :

## <a name="tensorflow-savedmodel-format"></a>Format TensorFlow SavedModel
Les modèles TensorFlow doivent être inscrits au **format SavedModel** pour fonctionner avec un modèle de déploiement sans code.

Pour plus d’informations sur la création d’un SavedModel, consultez [ce lien](https://www.tensorflow.org/guide/saved_model).

Nous prenons en charge toute version de TensorFlow qui est indiquée sous « Tags » (balises) sur le [DockerHub TensorFlow](https://registry.hub.docker.com/r/tensorflow/serving/tags).

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='flowers',                        # Name of the registered model in your workspace.
                       model_path='./flowers_model',                # Local Tensorflow SavedModel folder to upload and register as a model.
                       model_framework=Model.Framework.TENSORFLOW,  # Framework used to create the model.
                       model_framework_version='1.14.0',            # Version of Tensorflow used to create the model.
                       description='Flowers model')

service_name = 'tensorflow-flower-service'
service = Model.deploy(ws, service_name, [model])
```

## <a name="onnx-models"></a>Modèles ONNX

L’inscription et le déploiement du modèle ONNX sont pris en charge pour tout graphique d’inférence ONNX. Les étapes de prétraitement et de post-traitement ne sont pas prises en charge actuellement.

Voici un exemple d’inscription et de déploiement d’un modèle MNIST ONNX :

```python
from azureml.core import Model

model = Model.register(workspace=ws,
                       model_name='mnist-sample',                  # Name of the registered model in your workspace.
                       model_path='mnist-model.onnx',              # Local ONNX model to upload and register as a model.
                       model_framework=Model.Framework.ONNX ,      # Framework used to create the model.
                       model_framework_version='1.3',              # Version of ONNX used to create the model.
                       description='Onnx MNIST model')

service_name = 'onnx-mnist-service'
service = Model.deploy(ws, service_name, [model])
```

Pour noter un modèle, consultez [Utiliser un modèle Machine Learning déployé en tant que service web](./how-to-consume-web-service.md). De nombreux projets ONNX utilisent des fichiers protobuf pour stocker de manière compacte les données de formation et de validation, ce qui peut rendre difficile de savoir quel est le format de données attendu par le service. En tant que développeur de modèles, vous devez documenter les éléments suivants pour vos développeurs :

* Format d’entrée (JSON ou binaire)
* Type et forme des données d’entrée (par exemple, un tableau de floats de forme [100,100,3])
* Informations relatives au domaine (par exemple, pour une image, l’espace de couleurs, l’ordre des composants et la normalisation des valeurs)

Si vous utilisez Pytorch, l’article [Exporting models from PyTorch to ONNX](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb) (Exportation de modèles de PyTorch vers ONNX) contient des informations sur la conversion et les limitations. 

## <a name="scikit-learn-models"></a>Modèles Scikit-learn

Aucun modèle de déploiement de code n’est pris en charge pour tous les types de modèles Scikit-learn intégrés.

Voici un exemple d’inscription et de déploiement d’un modèle sklearn sans code supplémentaire :

```python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = Model.register(workspace=ws,
                       model_name='my-sklearn-model',                # Name of the registered model in your workspace.
                       model_path='./sklearn_regression_model.pkl',  # Local file to upload and register as a model.
                       model_framework=Model.Framework.SCIKITLEARN,  # Framework used to create the model.
                       model_framework_version='0.19.1',             # Version of scikit-learn used to create the model.
                       resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5),
                       description='Ridge regression model to predict diabetes progression.',
                       tags={'area': 'diabetes', 'type': 'regression'})
                       
service_name = 'my-sklearn-service'
service = Model.deploy(ws, service_name, [model])
```

> [!NOTE]
> Les modèles qui prennent en charge predict_proba utilisent cette méthode par défaut. Pour remplacer cette valeur pour utiliser predict, vous pouvez modifier le corps POST comme indiqué ci-dessous :

```python
import json


input_payload = json.dumps({
    'data': [
        [ 0.03807591,  0.05068012,  0.06169621, 0.02187235, -0.0442235,
         -0.03482076, -0.04340085, -0.00259226, 0.01990842, -0.01764613]
    ],
    'method': 'predict'  # If you have a classification model, the default behavior is to run 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

> [!NOTE]
> Ces dépendances sont incluses dans le conteneur d’inférence scikit-learn prédéfini :

```yaml
    - dill
    - azureml-defaults
    - inference-schema[numpy-support]
    - scikit-learn
    - numpy
    - joblib
    - pandas
    - scipy
    - sklearn_pandas
```
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