---
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: include
ms.date: 07/31/2020
ms.author: gopalv
ms.openlocfilehash: 95acb028867caf6f497dd99ad3082efcaab09c7b
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325281"
---
Le script d’entrée reçoit les données envoyées à un service web déployé, puis les passe au modèle. Ensuite, il prend la réponse retournée par le modèle et la retourne au client. *Le script est propre à votre modèle*. Il doit comprendre les données que le modèle attend et retourne.

Les deux choses que vous devez accomplir dans votre script d’entrée sont les suivantes :

1. Charger votre modèle (à l’aide d’une fonction appelée `init()`)
1. Exécuter votre modèle sur des données d’entrée (à l’aide d’une fonction appelée `run()`)

Examinons ces étapes en détail.

### <a name="writing-init"></a>Écriture de la fonction init () 

#### <a name="loading-a-registered-model"></a>Chargement d’un modèle inscrit

Vos modèles inscrits sont stockés sous un chemin d’accès désigné par une variable d’environnement appelée `AZUREML_MODEL_DIR`. Pour plus d’informations sur la structure exacte du répertoire, consultez [Localiser les fichiers de modèles dans votre script d’entrée](../articles/machine-learning/how-to-deploy-advanced-entry-script.md#load-registered-models).

#### <a name="loading-a-local-model"></a>Chargement d’un modèle local

Si vous avez choisi de ne pas inscrire votre modèle et que vous avez transmis votre modèle dans le cadre de votre répertoire source, vous pouvez le lire comme vous le feriez localement, en transmettant le chemin d’accès au modèle relatif à votre script de scoring. Par exemple, si vous aviez un répertoire structuré comme suit :

```bash

- source_dir
    - score.py
    - models
        - model1.onnx

```

vous pouvez charger vos modèles avec le code Python suivant :

```python
import os

model = open(os.path.join('.', 'models', 'model1.onnx'))
```

### <a name="writing-run"></a>Écriture de la fonction run()

`run()` est exécutée chaque fois que votre modèle reçoit une demande de scoring et s’attend à ce que le corps de la demande soit un document JSON avec la structure suivante :

```json
{
    "data": <model-specific-data-structure>
}

```

L’entrée de `run()` est une chaîne Python contenant tout ce qui suit la clé « data ».

L’exemple suivant montre comment charger un modèle scikit-Learn inscrit et le noter avec des données numpy :

```python
import json
import numpy as np
import os
from sklearn.externals import joblib


def init():
    global model
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(data):
    try:
        data = np.array(json.loads(data))
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Pour obtenir des exemples plus avancés, notamment la génération automatique de schémas Swagger et des données binaires (c’est-à-dire d’images), lisez [l’article sur la création de scripts d’entrée avancés](../articles/machine-learning/how-to-deploy-advanced-entry-script.md).