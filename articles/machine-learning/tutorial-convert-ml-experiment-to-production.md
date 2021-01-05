---
title: Convertir du code de notebook en scripts Python
titleSuffix: Azure Machine Learning
description: Transformez vos notebooks expérimentaux de Machine Learning en code prêt pour la production à l’aide du modèle de code MLOpsPython. Après quoi, testez, déployez et automatisez ce code.
author: bjcmit
ms.author: brysmith
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 04/30/2020
ms.custom: devx-track-python
ms.openlocfilehash: 2f7746f079e740493348731376d0a5a7b1a9e954
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93317848"
---
# <a name="tutorial-convert-ml-experiments-to-production-python-code"></a>Tutoriel : Convertir des expériences ML en code Python de production

Dans ce tutoriel, vous allez apprendre à convertir des notebooks Jupyter en scripts Python pour faciliter les tests et l’automatisation en utilisant le modèle de code MLOpsPython et Azure Machine Learning. En général, ce processus sert à extraire le code d’expérimentation/entraînement d’un notebook Jupyter et de le convertir en scripts Python. Ces scripts peuvent ensuite être utilisés pour les tests et l’automatisation CI/CD dans votre environnement de production. 

Un projet de Machine Learning nécessite des expérimentations au cours desquelles des hypothèses sont testées avec des outils Agile comme Jupyter Notebook sur des jeux de données réels. Quand le modèle est prêt pour la production, vous devez placer le code du modèle dans un dépôt de code de production. Dans certains cas, le code du modèle doit être converti en scripts Python qui sont placés dans le dépôt de code de production. Ce tutoriel décrit l’approche recommandée pour exporter du code d’expérimentation en scripts Python.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Nettoyer le code non essentiel
> * Refactoriser le code Jupyter Notebook en fonctions
> * Créer des scripts Python pour des tâches associées
> * Créer des tests unitaires

## <a name="prerequisites"></a>Prérequis

- Générez le [modèle MLOpsPython](https://github.com/microsoft/MLOpsPython/generate) et utilisez les notebooks `experimentation/Diabetes Ridge Regression Training.ipynb` et `experimentation/Diabetes Ridge Regression Scoring.ipynb`. Ces notebooks sont utilisés pour illustrer la conversion d’une expérimentation en production. Vous pouvez trouver ces notebooks à l’adresse [https://github.com/microsoft/MLOpsPython/tree/master/experimentation](https://github.com/microsoft/MLOpsPython/tree/master/experimentation).
- Installez `nbconvert`. Suivez uniquement les instructions de la section __Installing nbconvert__ (Installation de nbconvert) dans la page [Installation](https://nbconvert.readthedocs.io/en/latest/install.html).

## <a name="remove-all-nonessential-code"></a>Supprimer tout le code non essentiel

Certaines parties de code écrites durant l’expérimentation sont uniquement destinées à des fins exploratoires. La première étape de conversion de code expérimental en code de production consiste donc à supprimer le code non essentiel. Cette opération rend également le code plus facile à gérer. Dans cette section, vous allez supprimer du code du notebook `experimentation/Diabetes Ridge Regression Training.ipynb`. Les instructions qui impriment la forme de `X` et `y` et la cellule qui appelle `features.describe` sont uniquement destinées à l’exploration de données et peuvent être supprimées. Une fois le code non essentiel supprimé, `experimentation/Diabetes Ridge Regression Training.ipynb` doit ressembler au code suivant sans le markdown :

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import joblib
import pandas as pd

sample_data = load_diabetes()

df = pd.DataFrame(
    data=sample_data.data,
    columns=sample_data.feature_names)
df['Y'] = sample_data.target

X = df.drop('Y', axis=1).values
y = df['Y'].values

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=0)
data = {"train": {"X": X_train, "y": y_train},
        "test": {"X": X_test, "y": y_test}}

args = {
    "alpha": 0.5
}

reg_model = Ridge(**args)
reg_model.fit(data["train"]["X"], data["train"]["y"])

preds = reg_model.predict(data["test"]["X"])
mse = mean_squared_error(preds, y_test)
metrics = {"mse": mse}
print(metrics)

model_name = "sklearn_regression_model.pkl"
joblib.dump(value=reg, filename=model_name)
```

## <a name="refactor-code-into-functions"></a>Refactoriser le code en fonctions

Deuxièmement, le code Jupyter doit être refactorisé en fonctions. La refactorisation du code en fonctions facilite le test unitaire et rend le code plus facile à gérer. Dans cette section, vous allez refactoriser :

- Le notebook Diabetes Ridge Regression Training (`experimentation/Diabetes Ridge Regression Training.ipynb`)
- Le notebook Diabetes Ridge Regression Scoring (`experimentation/Diabetes Ridge Regression Scoring.ipynb`)

### <a name="refactor-diabetes-ridge-regression-training-notebook-into-functions"></a>Refactoriser le notebook Diabetes Ridge Regression Training en fonctions

Dans `experimentation/Diabetes Ridge Regression Training.ipynb`, effectuez les étapes suivantes :

1. Créez une fonction appelée `split_data` pour fractionner le dataframe en données de test et d’entraînement. La fonction doit utiliser le dataframe `df` comme paramètre et retourner un dictionnaire contenant les clés `train` et `test`.

    Déplacez le code sous le titre *Fractionner les données en jeux d’entraînement et de validation* dans la fonction `split_data` et modifiez-le pour retourner l’objet `data`.

1. Créez une fonction appelée `train_model` qui utilise les paramètres `data` et `args`, puis retourne un modèle entraîné.

    Déplacez le code sous le titre *Modèle d’entraînement sur le jeu d’entraînement* dans la fonction `train_model` et modifiez-le pour retourner l’objet `reg_model`. Supprimez le dictionnaire `args`, les valeurs proviendront du paramètre `args`.

1. Créez une fonction appelée `get_model_metrics` qui utilise les paramètres `reg_model` et `data`, évalue le modèle, puis retourne un dictionnaire de métriques pour le modèle entraîné.

    Déplacez le code sous le titre *Valider le modèle sur le jeu de validation* dans la fonction `get_model_metrics` et modifiez-le pour retourner l’objet `metrics`.

Les trois fonctions doivent se présenter comme suit :

```python
# Split the dataframe into test and train data
def split_data(df):
    X = df.drop('Y', axis=1).values
    y = df['Y'].values

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=0)
    data = {"train": {"X": X_train, "y": y_train},
            "test": {"X": X_test, "y": y_test}}
    return data


# Train the model, return the model
def train_model(data, args):
    reg_model = Ridge(**args)
    reg_model.fit(data["train"]["X"], data["train"]["y"])
    return reg_model


# Evaluate the metrics for the model
def get_model_metrics(reg_model, data):
    preds = reg_model.predict(data["test"]["X"])
    mse = mean_squared_error(preds, data["test"]["y"])
    metrics = {"mse": mse}
    return metrics
```

Toujours dans `experimentation/Diabetes Ridge Regression Training.ipynb`, effectuez les étapes suivantes :

1. Créez une fonction appelée `main`. Celle-ci ne prend aucun paramètre et ne retourne rien.
1. Déplacez le code sous le titre « Charger les données » dans la fonction `main`.
1. Ajoutez des appels pour les fonctions récemment écrites dans la fonction `main` :
    ```python
    # Split Data into Training and Validation Sets
    data = split_data(df)
    ```

    ```python
    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)
    ```

    ```python
    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)
    ```
1. Déplacez le code sous le titre « Enregistrer le modèle » dans la fonction `main`.

La fonction `main` doit ressembler au code suivant :

```python
def main():
    # Load Data
    sample_data = load_diabetes()

    df = pd.DataFrame(
        data=sample_data.data,
        columns=sample_data.feature_names)
    df['Y'] = sample_data.target

    # Split Data into Training and Validation Sets
    data = split_data(df)

    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)

    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)

    # Save Model
    model_name = "sklearn_regression_model.pkl"

    joblib.dump(value=reg, filename=model_name)
```

À ce stade, il ne doit rester dans le notebook aucun code qui ne se trouve pas dans une fonction, à l’exception des instructions import dans la première cellule.

Ajoutez une instruction qui appelle la fonction `main`.

```python
main()
```

Après refactorisation, `experimentation/Diabetes Ridge Regression Training.ipynb` doit ressembler au code suivant sans le markdown :

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import pandas as pd
import joblib


# Split the dataframe into test and train data
def split_data(df):
    X = df.drop('Y', axis=1).values
    y = df['Y'].values

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=0)
    data = {"train": {"X": X_train, "y": y_train},
            "test": {"X": X_test, "y": y_test}}
    return data


# Train the model, return the model
def train_model(data, args):
    reg_model = Ridge(**args)
    reg_model.fit(data["train"]["X"], data["train"]["y"])
    return reg_model


# Evaluate the metrics for the model
def get_model_metrics(reg_model, data):
    preds = reg_model.predict(data["test"]["X"])
    mse = mean_squared_error(preds, data["test"]["y"])
    metrics = {"mse": mse}
    return metrics


def main():
    # Load Data
    sample_data = load_diabetes()

    df = pd.DataFrame(
        data=sample_data.data,
        columns=sample_data.feature_names)
    df['Y'] = sample_data.target

    # Split Data into Training and Validation Sets
    data = split_data(df)

    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)

    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)

    # Save Model
    model_name = "sklearn_regression_model.pkl"

    joblib.dump(value=reg, filename=model_name)

main()
```

### <a name="refactor-diabetes-ridge-regression-scoring-notebook-into-functions"></a>Refactoriser le notebook Diabetes Ridge Regression Scoring en fonctions

Dans `experimentation/Diabetes Ridge Regression Scoring.ipynb`, effectuez les étapes suivantes :

1. Créez une fonction appelée `init`. Celle-ci ne prend aucun paramètre et ne retourne rien.
1. Copiez le code sous le titre « Load Model » dans la fonction `init`.

La fonction `init` doit ressembler au code suivant :

```python
def init():
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)
```

Une fois la fonction `init` créée, remplacez tout le code sous le titre « Load model » par un appel unique à `init`, comme ceci :

```python
init()
```

Dans `experimentation/Diabetes Ridge Regression Scoring.ipynb`, effectuez les étapes suivantes :

1. Créez une fonction appelée `run` qui prend les paramètres `raw_data` et `request_headers` et retourne un dictionnaire de résultats, comme ceci :

    ```python
    {"result": result.tolist()}
    ```

1. Copiez le code sous les titres « Prepare Data » et « Score Data » dans la fonction `run`.

    Le code de la fonction `run` doit ressembler au suivant (n’oubliez pas de supprimer les instructions qui définissent les variables `raw_data` et `request_headers`, qui seront utilisées par la suite quand la fonction `run` sera appelée) :

    ```python
    def run(raw_data, request_headers):
        data = json.loads(raw_data)["data"]
        data = numpy.array(data)
        result = model.predict(data)

        return {"result": result.tolist()}
    ```

Une fois la fonction `run` créée, remplacez tout le code sous les titres « Prepare Data » et « Score Data » par le suivant :

```python
raw_data = '{"data":[[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}'
request_header = {}
prediction = run(raw_data, request_header)
print("Test result: ", prediction)
```

Le code précédent définit les variables `raw_data` et `request_header`, appelle la fonction `run` avec `raw_data` et `request_header`, puis imprime les prédictions.

Après refactorisation, `experimentation/Diabetes Ridge Regression Scoring.ipynb` doit ressembler au code suivant sans le markdown :

```python
import json
import numpy
from azureml.core.model import Model
import joblib

def init():
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)

def run(raw_data, request_headers):
    data = json.loads(raw_data)["data"]
    data = numpy.array(data)
    result = model.predict(data)

    return {"result": result.tolist()}

init()
test_row = '{"data":[[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}'
request_header = {}
prediction = run(test_row, {})
print("Test result: ", prediction)
```

## <a name="combine-related-functions-in-python-files"></a>Combiner des fonctions associées dans des fichiers Python

Troisièmement, les fonctions associées doivent être fusionnées dans des fichiers Python pour faciliter la réutilisation du code. Dans cette section, vous allez créer des fichiers Python pour les notebooks suivants :

- Le notebook Diabetes Ridge Regression Training (`experimentation/Diabetes Ridge Regression Training.ipynb`)
- Le notebook Diabetes Ridge Regression Scoring (`experimentation/Diabetes Ridge Regression Scoring.ipynb`)

### <a name="create-python-file-for-the-diabetes-ridge-regression-training-notebook"></a>Créer un fichier Python pour le notebook Diabetes Ridge Regression Training

Convertissez votre notebook en script exécutable. Pour cela, exécutez dans une invite de commandes l’instruction suivante qui utilise le package `nbconvert` et le chemin de `experimentation/Diabetes Ridge Regression Training.ipynb` :

```
jupyter nbconvert -- to script "Diabetes Ridge Regression Training.ipynb" –output train
```

Une fois le notebook converti en `train.py`, supprimez tous les commentaires non souhaités. Remplacez l’appel à `main()` à la fin du fichier par un appel conditionnel semblable au code suivant :

```python
if __name__ == '__main__':
    main()
```

Votre fichier `train.py` doit ressembler au code suivant :

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
import pandas as pd
import joblib


# Split the dataframe into test and train data
def split_data(df):
    X = df.drop('Y', axis=1).values
    y = df['Y'].values

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=0)
    data = {"train": {"X": X_train, "y": y_train},
            "test": {"X": X_test, "y": y_test}}
    return data


# Train the model, return the model
def train_model(data, args):
    reg_model = Ridge(**args)
    reg_model.fit(data["train"]["X"], data["train"]["y"])
    return reg_model


# Evaluate the metrics for the model
def get_model_metrics(reg_model, data):
    preds = reg_model.predict(data["test"]["X"])
    mse = mean_squared_error(preds, data["test"]["y"])
    metrics = {"mse": mse}
    return metrics


def main():
    # Load Data
    sample_data = load_diabetes()

    df = pd.DataFrame(
        data=sample_data.data,
        columns=sample_data.feature_names)
    df['Y'] = sample_data.target

    # Split Data into Training and Validation Sets
    data = split_data(df)

    # Train Model on Training Set
    args = {
        "alpha": 0.5
    }
    reg = train_model(data, args)

    # Validate Model on Validation Set
    metrics = get_model_metrics(reg, data)

    # Save Model
    model_name = "sklearn_regression_model.pkl"

    joblib.dump(value=reg, filename=model_name)

if __name__ == '__main__':
    main()
```

`train.py` peut maintenant être appelé à partir d’un terminal en exécutant `python train.py`.
Les fonctions de `train.py` peuvent également être appelées à partir d’autres fichiers.

Le fichier `train_aml.py` situé dans le répertoire `diabetes_regression/training` du référentiel MLOpsPython appelle les fonctions définies dans `train.py` dans le contexte d’une exécution de l’expérience Azure Machine Learning. Les fonctions peuvent également être appelées dans des tests unitaires, abordés plus loin dans ce guide.

### <a name="create-python-file-for-the-diabetes-ridge-regression-scoring-notebook"></a>Créer un fichier Python pour le notebook Diabetes Ridge Regression Scoring

Convertissez votre notebook en script exécutable. Pour cela, exécutez dans une invite de commandes l’instruction suivante qui utilise le package `nbconvert` et le chemin de `experimentation/Diabetes Ridge Regression Scoring.ipynb` :

```
jupyter nbconvert -- to script "Diabetes Ridge Regression Scoring.ipynb" –output score
```

Une fois le notebook converti en `score.py`, supprimez tous les commentaires non souhaités. Votre fichier `score.py` doit ressembler au code suivant :

```python
import json
import numpy
from azureml.core.model import Model
import joblib

def init():
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)

def run(raw_data, request_headers):
    data = json.loads(raw_data)["data"]
    data = numpy.array(data)
    result = model.predict(data)

    return {"result": result.tolist()}

init()
test_row = '{"data":[[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}'
request_header = {}
prediction = run(test_row, request_header)
print("Test result: ", prediction)
```

La variable `model` doit être globale afin d’être visible dans tout le script. Ajoutez l’instruction suivante au début de la fonction `init` :

```python
global model
```

Une fois l’instruction précédente ajoutée, la fonction `init` doit ressembler au code suivant :

```python
def init():
    global model

    # load the model from file into a global object
    model_path = Model.get_model_path(
        model_name="sklearn_regression_model.pkl")
    model = joblib.load(model_path)
```

## <a name="create-unit-tests-for-each-python-file"></a>Créer des tests unitaires pour chaque fichier Python

Quatrièmement, créez des tests unitaires pour vos fonctions Python. Les tests unitaires protègent le code contre les régressions fonctionnelles et facilitent sa gestion. Dans cette section, vous allez créer des tests unitaires pour les fonctions de `train.py`.

`train.py` contient plusieurs fonctions, mais nous n’allons créer qu’un seul test unitaire pour la fonction `train_model` à l’aide du framework Pytest dans ce tutoriel. Pytest n’est pas le seul framework de test unitaire Python, mais c’est l’un des plus couramment utilisés. Pour plus d’informations, visitez [Pytest](https://pytest.org).

Un test unitaire contient généralement trois actions principales :

- Organiser l’objet : création et configuration des objets nécessaires
- Agir sur un objet
- Déclarer ce qui est attendu

Le test unitaire appellera `train_model` avec des données et des arguments codés en dur, et validera que `train_model` a agi comme prévu en utilisant le modèle entraîné obtenu pour effectuer une prédiction et en comparant cette prédiction à une valeur attendue.

```python
import numpy as np
from code.training.train import train_model


def test_train_model():
    # Arrange
    X_train = np.array([1, 2, 3, 4, 5, 6]).reshape(-1, 1)
    y_train = np.array([10, 9, 8, 8, 6, 5])
    data = {"train": {"X": X_train, "y": y_train}}

    # Act
    reg_model = train_model(data, {"alpha": 1.2})

    # Assert
    preds = reg_model.predict([[1], [2]])
    np.testing.assert_almost_equal(preds, [9.93939393939394, 9.03030303030303])
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment convertir une expérience en code de production, consultez les liens suivants pour obtenir plus d’informations et pour connaître les étapes suivantes :

+ [MLOpsPython](https://github.com/microsoft/MLOpsPython/blob/master/docs/custom_model.md) : générez un pipeline CI/CD pour entraîner, évaluer et déployer votre propre modèle à l’aide d’Azure Pipelines et d’Azure Machine Learning
+ [Superviser les exécutions et les métriques des expériences Azure Machine Learning](./how-to-track-experiments.md)
+ [Superviser et collecter des données à partir des points de terminaison de service web Machine Learning](./how-to-enable-app-insights.md)