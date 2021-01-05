---
title: 'Tutoriel : Créer le modèle prédictif avec un notebook (partie 1 sur 2)'
titleSuffix: Azure Machine Learning
description: Découvrez comment créer et déployer un modèle Machine Learning en utilisant du code dans un notebook Jupyter, afin de pouvoir l’utiliser pour prédire des résultats dans Microsoft Power BI.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: samkemp
author: samuel100
ms.reviewer: sdgilley
ms.date: 12/11/2020
ms.openlocfilehash: f8209c0d26cf8c572d10666696231b0468cfcbc6
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97370729"
---
# <a name="tutorial-power-bi-integration---create-the-predictive-model-with-a-notebook-part-1-of-2"></a>Tutoriel : Intégration Power BI – Créer le modèle prédictif avec un notebook (partie 1 sur 2)

Dans la première partie de ce tutoriel, vous allez entraîner et déployer un modèle Machine Learning prédictif en utilisant du code dans un notebook Jupyter. Dans la partie 2, vous utiliserez le modèle pour prédire les résultats dans Microsoft Power BI.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un bloc-notes Jupyter Notebook
> * Créer une instance de calcul Azure Machine Learning
> * Entraîner un modèle de régression à l’aide de scikit-learn
> * Déployer le modèle sur un point de terminaison de scoring en temps réel

Il existe trois façons différentes de créer et de déployer le modèle que vous utiliserez dans Power BI.  Cet article couvre l’option A : Entraîner et déployer des modèles à l’aide de notebooks.  Cette option montre une expérience de création Code First à l’aide de notebooks Jupyter hébergés dans le studio Azure Machine Learning. 

Vous pouvez utiliser à la place :

* [Option B : Entraîner et déployer des modèles à l’aide du concepteur](tutorial-power-bi-designer-model.md) – expérience de création à faible code à l’aide du concepteur (interface utilisateur de glisser-déplacer).
* [Option C : Entraîner et déployer des modèles à l’aide du Machine Learning automatisé](tutorial-power-bi-automated-model.md) – expérience de création sans code qui automatise entièrement la préparation des données et l’entraînement du modèle.


## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure ([un essai gratuit est disponible](https://aka.ms/AMLFree)). 
- Un espace de travail Azure Machine Learning. Si vous ne disposez pas déjà d’un espace de travail, suivez le [guide pratique pour créer un espace de travail Azure Machine Learning](./how-to-manage-workspace.md#create-a-workspace).
- Avoir des connaissances préliminaires du langage Python et des workflows Machine Learning.

## <a name="create-a-notebook-and-compute"></a>Créer un notebook et calculer

Dans la page d’accueil du [studio Azure Machine Learning](https://ml.azure.com), sélectionnez **Créer nouveau** suivi de **Notebook** :

:::image type="content" source="media/tutorial-power-bi/create-new-notebook.png" alt-text="Capture d’écran montrant comment créer un notebook":::
 
Dans la boîte de dialogue **Créer un fichier** qui s’affiche, entrez :

1. Un nom de fichier pour votre notebook (par exemple, `my_model_notebook`)
1. Modifiez **Type de fichier** en spécifiant **Notebook**.

Sélectionnez **Create** (Créer). Ensuite, vous devez créer un calcul et l’attacher à votre notebook pour pouvoir exécuter des cellules de code. Pour ce faire, sélectionnez l’icône plus (+) en haut du notebook :

:::image type="content" source="media/tutorial-power-bi/create-compute.png" alt-text="Capture d’écran montrant comment créer une instance de calcul":::

Ensuite, dans la page **Créer une instance de calcul** :

1. Choisissez une taille de processeur de machine virtuelle – dans le cadre de ce tutoriel, une taille **Standard_D11_v2** (deux cœurs, 14 Go de RAM) conviendra.
1. Sélectionnez **Suivant**. 
1. Dans la page **Configurer les paramètres**, spécifiez un **nom de calcul** valide (les caractères valides sont les lettres majuscules et minuscules, les chiffres et le caractère -).
1. Sélectionnez **Create** (Créer).

Dans le notebook, vous pouvez remarquer que le cercle en regard de **Calculer** est devenu cyan, ce qui indique que l’instance de calcul est en cours de création :

:::image type="content" source="media/tutorial-power-bi/creating.png" alt-text="Capture d’écran montrant le calcul en cours de création":::

> [!NOTE]
> Le provisionnement du calcul peut prendre environ 2-4 minutes.

Une fois le calcul provisionné, vous pouvez utiliser le notebook pour exécuter des cellules de code. Par exemple, tapez dans la cellule :

```python
import numpy as np

np.sin(3)
```

Appuyez ensuite sur **Maj-Entrée** (ou sur **Ctrl-Entrée**, ou sélectionnez le bouton de lecture en regard de la cellule). Vous devez normalement voir la sortie suivante :

:::image type="content" source="media/tutorial-power-bi/simple-sin.png" alt-text="Capture d’écran montrant l’exécution de la cellule":::

Vous êtes maintenant prêt à créer un modèle Machine Learning.

## <a name="build-a-model-using-scikit-learn"></a>Générer un modèle à l’aide de scikit-learn

Dans ce tutoriel, vous utilisez le [dataset Diabetes](https://www4.stat.ncsu.edu/~boos/var.select/diabetes.html), qui est mis à votre disposition dans [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/). 


### <a name="import-data"></a>Importer des données

Pour importer vos données, copiez-collez le code ci-dessous dans une nouvelle **cellule de code** dans votre notebook :

```python
from azureml.opendatasets import Diabetes

diabetes = Diabetes.get_tabular_dataset()
X = diabetes.drop_columns("Y")
y = diabetes.keep_columns("Y")
X_df = X.to_pandas_dataframe()
y_df = y.to_pandas_dataframe()
X_df.info()
```

La trame de données Pandas `X_df` contient 10 variables d’entrée de mesures de référence (telles que l’âge, le sexe, l’indice de masse corporelle, la pression artérielle moyenne et six mesures d’analyse sanguine). La trame de données Pandas `y_df` est la variable cible contenant une mesure quantitative de l’évolution de la maladie un an après les mesures de référence. Il y a un total de 442 enregistrements.

### <a name="train-model"></a>Effectuer l'apprentissage du modèle

Créez une nouvelle **cellule de code** dans votre notebook et copiez-collez l’extrait de code ci-dessous, qui construit un modèle de régression Ridge et sérialise le modèle à l’aide du format Pickle de Python :

```python
import joblib
from sklearn.linear_model import Ridge

model = Ridge().fit(X,y)
joblib.dump(model, 'sklearn_regression_model.pkl')
```

### <a name="register-the-model"></a>Inscrire le modèle

Outre le contenu du fichier de modèle lui-même, votre modèle inscrit stocke également les métadonnées du modèle (la description du modèle, les étiquettes et les informations du framework), qui seront utiles lors de la gestion et du déploiement de modèles dans votre espace de travail. Les étiquettes, par exemple, vous permettent de catégoriser vos modèles et d’appliquer des filtres lorsque vous listez les modèles dans votre espace de travail. En outre, marquer ce modèle avec le framework scikit-learn simplifie son déploiement en tant que service web, comme nous le verrons plus tard.

Copiez-collez le code ci-dessous dans une nouvelle **cellule de code** dans votre notebook :

```python
import sklearn

from azureml.core import Workspace
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

ws = Workspace.from_config()

model = Model.register(workspace=ws,
                       model_name='my-sklearn-model',                # Name of the registered model in your workspace.
                       model_path='./sklearn_regression_model.pkl',  # Local file to upload and register as a model.
                       model_framework=Model.Framework.SCIKITLEARN,  # Framework used to create the model.
                       model_framework_version=sklearn.__version__,  # Version of scikit-learn used to create the model.
                       sample_input_dataset=X,
                       sample_output_dataset=y,
                       resource_configuration=ResourceConfiguration(cpu=2, memory_in_gb=4),
                       description='Ridge regression model to predict diabetes progression.',
                       tags={'area': 'diabetes', 'type': 'regression'})

print('Name:', model.name)
print('Version:', model.version)
```

Vous pouvez également afficher le modèle dans Azure Machine Learning Studio en accédant à **Points de terminaison** dans le menu de gauche :

:::image type="content" source="media/tutorial-power-bi/model.png" alt-text="Capture d’écran montrant le modèle":::

### <a name="define-the-scoring-script"></a>Définir le script de scoring

Lorsque vous déployez un modèle à intégrer dans Microsoft Power BI, vous devez définir un *script de scoring* Python et un environnement personnalisé. Le script de scoring contient deux fonctions :

- `init()` – cette fonction est exécutée une fois que le service a démarré. Cette fonction charge le modèle (notez que le modèle est automatiquement téléchargé à partir du registre de modèle) et le désérialise.
- `run(data)` – cette fonction est exécutée lorsqu’un appel est effectué au service avec des données d’entrée nécessitant un scoring. 

>[!NOTE]
> Nous utilisons des décorateurs Python pour définir le schéma des données d’entrée et de sortie, ce qui est important pour le fonctionnement de l’intégration de Microsoft Power BI.

Copiez-collez le code ci-dessous dans une nouvelle **cellule de code** dans votre notebook. L’extrait de code ci-dessous contient des commandes magiques de cellule qui écriront le code dans un fichier nommé score.py.

```python
%%writefile score.py

import json
import pickle
import numpy as np
import pandas as pd
import os
import joblib
from azureml.core.model import Model

from inference_schema.schema_decorators import input_schema, output_schema
from inference_schema.parameter_types.numpy_parameter_type import NumpyParameterType
from inference_schema.parameter_types.pandas_parameter_type import PandasParameterType


def init():
    global model
    # Replace filename if needed.
    path = os.getenv('AZUREML_MODEL_DIR') 
    model_path = os.path.join(path, 'sklearn_regression_model.pkl')
    # Deserialize the model file back into a sklearn model.
    model = joblib.load(model_path)


input_sample = pd.DataFrame(data=[{
    "AGE": 5,
    "SEX": 2,
    "BMI": 3.1,
    "BP": 3.1,
    "S1": 3.1,
    "S2": 3.1,
    "S3": 3.1,
    "S4": 3.1,
    "S5": 3.1,
    "S6": 3.1
}])

# This is an integer type sample. Use the data type that reflects the expected result.
output_sample = np.array([0])

# To indicate that we support a variable length of data input,
# set enforce_shape=False
@input_schema('data', PandasParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        print("input_data....")
        print(data.columns)
        print(type(data))
        result = model.predict(data)
        print("result.....")
        print(result)
    # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

### <a name="define-the-custom-environment"></a>Définir l’environnement personnalisé

Ensuite, nous devons définir l’environnement de scoring du modèle : nous devons définir dans cet environnement les packages Python requis par le script de scoring (score.py) défini ci-dessus, comme pandas, scikit-learn, etc.

Pour définir l’environnement, copiez-collez le code ci-dessous dans une nouvelle **cellule de code** dans votre notebook :

```python
from azureml.core.model import InferenceConfig
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies

environment = Environment('my-sklearn-environment')
environment.python.conda_dependencies = CondaDependencies.create(pip_packages=[
    'azureml-defaults',
    'inference-schema[numpy-support]',
    'joblib',
    'numpy',
    'pandas',
    'scikit-learn=={}'.format(sklearn.__version__)
])

inference_config = InferenceConfig(entry_script='./score.py',environment=environment)
```

### <a name="deploy-the-model"></a>Déployer le modèle

Pour déployer le modèle, copiez-collez le code ci-dessous dans une nouvelle **cellule de code** dans votre notebook :

```python
service_name = 'my-diabetes-model'

service = Model.deploy(ws, service_name, [model], inference_config, overwrite=True)
service.wait_for_deployment(show_output=True)
```

>[!NOTE]
> Le déploiement du service peut prendre 2-4 minutes.

Vous devez voir la sortie suivante si le service est correctement déployé :

```txt
Tips: You can try get_logs(): https://aka.ms/debugimage#dockerlog or local deployment: https://aka.ms/debugimage#debug-locally to debug if deployment takes longer than 10 minutes.
Running......................................................................................
Succeeded
ACI service creation operation finished, operation "Succeeded"
```

Vous pouvez également afficher le service dans Azure Machine Learning Studio en accédant à **Points de terminaison** dans le menu de gauche :

:::image type="content" source="media/tutorial-power-bi/endpoint.png" alt-text="Capture d’écran montrant le point de terminaison":::

Il est recommandé de tester le service web pour vous assurer qu’il fonctionne comme prévu. Revenez à votre notebook en sélectionnant **Notebooks** dans le menu de gauche d’Azure Machine Learning Studio. Copiez-collez le code ci-dessous dans une nouvelle **cellule de code** dans votre notebook pour tester le service :

```python
import json


input_payload = json.dumps({
    'data': X_df[0:2].values.tolist(),
    'method': 'predict'  # If you have a classification model, you can get probabilities by changing this to 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

La sortie doit ressembler à la structure json suivante : `{'predict': [[205.59], [68.84]]}`.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à générer et à déployer un modèle de manière à ce qu’il puisse être consommé par Microsoft Power BI. Dans la partie suivante, vous allez apprendre à consommer ce modèle à partir d’un rapport Power BI.

> [!div class="nextstepaction"]
> [Tutoriel : Consommer un modèle dans Power BI](/power-bi/connect-data/service-aml-integrate?context=azure/machine-learning/context/ml-context)
