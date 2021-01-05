---
title: Suivi des expériences et déploiement de modèles
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez comment suivre et journaliser les expériences à partir de Data Science Virtual Machine avec Azure Machine Learning et/ou MLFlow.
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: samkemp
ms.author: samkemp
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 17418b0255182934045acc9174b34cff2aefff99
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93307452"
---
# <a name="track-experiments-and-deploy-models-in-azure-machine-learning"></a>Suivi des expériences et déployer des modèles dans Azure Machine Learning

Améliorez le processus de création de modèle avec le suivi de vos expériences et la supervision des métriques d'exécution. Dans cet article, découvrez comment ajouter du code de journalisation à votre script d’entraînement à l’aide de l’API [MLflow](https://mlflow.org/) et comment suivre l’expérience dans Azure Machine Learning.

Le diagramme suivant montre qu’avec MLflow Tracking, vous suivez les métriques d’exécution d’une expérience et vous stockez les artefacts du modèle dans votre espace de travail Azure Machine Learning.

![suivre les expériences](./media/how-to-track-experiments/mlflow-diagram-track.png)

## <a name="prerequisites"></a>Prérequis

* Vous allez devoir [provisionner un espace de travail Azure Machine Learning](../how-to-manage-workspace.md#create-a-workspace)

## <a name="create-a-new-notebook"></a>Créer une nouvelle instance Notebook

Azure Machine Learning et le Kit de développement logiciel (SDK) MLFlow sont préinstallés sur Data Science Virtual Machine et accessibles dans l’environnement Conda * *azureml_py36_\** _. Dans Jupyterlab, cliquez sur le lanceur, puis sélectionnez le noyau suivant :

![sélection du noyau](./media/how-to-track-experiments/experiment-tracking-1.png)

## <a name="set-up-the-workspace"></a>Configurer l’espace de travail

Accédez au [portail Azure](https://portal.azure.com) et sélectionnez l’espace de travail que vous avez provisionné pour satisfaire les prérequis. __Télécharger config.json__ s’affiche (voir ci-dessous). Téléchargez la configuration et vérifiez qu’elle est stockée dans votre répertoire de travail de DSVM.

![Obtenir le fichier de configuration](./media/how-to-track-experiments/experiment-tracking-2.png)

La configuration contient des informations comme le nom de l’espace de travail, l’abonnement, et ainsi de suite ; cela signifie que vous n’avez pas besoin de coder en dur ces paramètres.

## <a name="track-dsvm-runs"></a>Suivre les exécutions de DSVM

Ajoutez le code suivant à votre notebook (ou script) pour définir l’objet d’espace de travail AzureML.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

>[!NOTE]
L’URI de suivi est valide pendant au maximum une heure. Si vous redémarrez votre script après un certain temps d’inactivité, utilisez l’API get_mlflow_tracking_uri pour obtenir un nouvel URI.

### <a name="load-the-data"></a>Chargement des données

Cet exemple utilise le jeu de données « diabète », un petit jeu de données bien connu fourni avec scikit-learn. Cette cellule charge le jeu de données et le divise en jeux d’entraînement et de test aléatoires.

```python
from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
from sklearn.externals import joblib

X, y = load_diabetes(return_X_y = True)
columns = ['age', 'gender', 'bmi', 'bp', 's1', 's2', 's3', 's4', 's5', 's6']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)
data = {
    "train":{"X": X_train, "y": y_train},
    "test":{"X": X_test, "y": y_test}
}

print ("Data contains", len(data['train']['X']), "training samples and",len(data['test']['X']), "test samples")
```

### <a name="add-tracking"></a>Ajouter un suivi

Ajoutez le suivi d’expérimentation à l’aide du kit de développement logiciel (SDK) Azure Machine Learning et chargez un modèle persistant sur l’enregistrement d’exécution de l’expérimentation. Le code suivant ajoute des étiquettes et charge un fichier de modèle dans l’exécution de l’expérience. Le modèle est également inscrit dans le registre des modèles Azure Machine Learning.

```python
# Get an experiment object from Azure Machine Learning
from azureml.mlflow import register_model
experiment_name = 'experiment_with_mlflow'
mlflow.set_experiment(experiment_name)

with mlflow.start_run():
    # Log the algorithm parameter alpha to the run
    mlflow.log_param('alpha', 0.03)

    # Create, fit, and test the scikit-learn Ridge regression model
    regression_model = Ridge(alpha=0.03)
    regression_model.fit(data['train']['X'], data['train']['y'])
    preds = regression_model.predict(data['test']['X'])

    # Output the Mean Squared Error to the notebook and to the run
    print('Mean Squared Error is', mean_squared_error(data['test']['y'], preds))
    mlflow.log_metric('mse', mean_squared_error(data['test']['y'], preds))

    # Save the model
    model_file_name = 'model.pkl'
    joblib.dump(value = regression_model, filename = model_file_name)

    # upload the model file explicitly into artifacts
    mlflow.log_artifact(model_file_name)
    # register the model
    register_model(mlflow.active_run(), 'diabetes_model', 'model.pkl', model_framework="ScikitLearn")
```

### <a name="view-runs-in-azure-machine-learning"></a>Voir les exécutions dans Azure Machine Learning

Vous pouvez voir l’expérience s’exécuter dans [Azure Machine Learning Studio](https://ml.azure.com). Cliquez sur __Expériences__ dans le menu de gauche et sélectionnez « experiment_with_mlflow » (ou si vous avez décidé de nommer votre expérience différemment dans l’extrait de code ci-dessus, cliquez sur le nom utilisé) :

![sélectionner l’expérience](./media/how-to-track-experiments/mlflow-experiments.png)

L’erreur quadratique moyenne (MSE) journalisée doit s’afficher :

![MSE](./media/how-to-track-experiments/mlflow-experiments-2.png)

Si vous cliquez sur l’exécution, d’autres détails s’affichent dans les __sorties+journaux__ , tout comme le modèle pickle

## <a name="deploy-model-in-azure-machine-learning"></a>Déployer un modèle dans Azure Machine Learning

Dans cette section, nous expliquons comment déployer les modèles entraînés sur DSVM dans Azure Machine Learning.

### <a name="step-1-create-inference-compute"></a>Étape 1 : Créer une capacité de calcul d’inférence

Dans le menu de gauche d’ [AzureML Studio](https://ml.azure.com), cliquez sur __Capacité de calcul__ , puis sur l’onglet __Clusters d’inférence__. Ensuite, cliquez sur __+ Nouveau__ comme indiqué ci-dessous :

![Créer une capacité de calcul d’inférence](./media/how-to-track-experiments/mlflow-experiments-6.png)

Dans le volet __Nouveau cluster d’inférence__ , fournissez des détails pour :

_ Nom de la capacité de calcul
* Service Kubernetes – Sélectionnez Créer
* Sélectionner la région
* Sélectionner la taille de machine virtuelle (pour les besoins de ce tutoriel, la valeur par défaut Standard_D3_v2 est suffisante)
* Rôle du cluster – Sélectionnez __Dev-test__
* Le nombre de nœuds doit être égal à __1__
* Configuration réseau – De base

Cliquez ensuite sur __Créer__.

![détails de la capacité de calcul](./media/how-to-track-experiments/mlflow-experiments-7.png)

### <a name="step-2-deploy-no-code-inference-service"></a>Étape 2 : Déployer un service d’inférence sans code

Au moment d’inscrire le modèle dans notre code à l’aide de `register_model`, nous avons spécifié le framework sklearn. Azure Machine Learning prend en charge les déploiements sans code pour les frameworks suivants :

* scikit-learn
* Format TensorFlow SavedModel
* Format de modèle ONNX

Le déploiement sans code signifie que vous pouvez effectuer un déploiement directement à partir de l’artefact du modèle sans avoir à spécifier un script de scoring spécifique.

Pour déployer le modèle diabetes, accédez au menu de gauche d’ [Azure Machine Learning Studio](https://ml.azure.com), puis sélectionnez __Modèles__. Cliquez ensuite sur le modèle inscrit diabetes_model :

![Sélectionner le modèle](./media/how-to-track-experiments/mlflow-experiments-3.png)

Ensuite, cliquez sur le bouton __Déployer__ dans le volet d’informations du modèle :

![Déployer](./media/how-to-track-experiments/mlflow-experiments-4.png)

Nous allons déployer le modèle dans le cluster d’inférence (Azure Kubernetes Service) que nous avons créé à l’étape 1. Fournissez les détails ci-dessous en attribuant un nom au service et en indiquant le nom du cluster de calcul AKS (créé à l’étape 1). Nous vous recommandons aussi de faire passer la __Capacité de réserve de processeur__ de 0,1 à 1 et la __Capacité de réserve de mémoire__  de 0,5 à 1. Pour ce faire, cliquez sur __Avancé__ et fournissez les détails. Cliquez ensuite sur __Déployer__.

![détails du déploiement](./media/how-to-track-experiments/mlflow-experiments-5.png)

### <a name="step-3-consume"></a>Étape 3 : Utiliser

Une fois le modèle déployé, voici les informations qui doivent s’afficher (pour accéder à cette page, cliquez sur Points de terminaison dans le menu de gauche, puis sur le nom du service déployé) :

![Consommer le modèle](./media/how-to-track-experiments/mlflow-experiments-8.png)

Vous devriez voir que l’état du déploiement passe de __transition en cours__ à __sain__. Par ailleurs, cette section d’informations présente le point de terminaison REST et les URL Swagger qu’un développeur d’applications peut utiliser pour intégrer votre modèle ML à ses applications.

Vous pouvez tester le point de terminaison en utilisant [Postman](https://www.postman.com/), ou utiliser le SDK AzureML :

```python
from azureml.core import Webservice
import json

# if you called your service differently then change the name below
service = Webservice(ws, name="diabetes-service")

input_payload = json.dumps({
    'data': X_test[0:2].tolist(),
    'method': 'predict'  # If you have a classification model, you can get probabilities by changing this to 'predict_proba'.
})

output = service.run(input_payload)

print(output)
```

### <a name="step-4-clean-up"></a>Étape 4 : Nettoyer

Supprimez la capacité de calcul d’inférence que vous avez créée à l’étape 1 pour éviter d’avoir à continuer de supporter des frais de calcul. Dans le menu de gauche d’Azure Machine Learning Studio, cliquez sur Calcul > Clusters d’inférence > Sélectionnez le calcul > Supprimer.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [déploiement de modèles dans AzureML](../how-to-deploy-and-where.md)