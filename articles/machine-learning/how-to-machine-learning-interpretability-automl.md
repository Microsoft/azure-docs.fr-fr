---
title: Explicabilité dans ML automatisé (version préliminaire)
titleSuffix: Azure Machine Learning
description: Découvrez comment obtenir des explications sur la façon dont votre modèle de Machine Learning automatisé détermine l’importance d’une caractéristique et effectue des prédictions lors de l’utilisation du Kit de développement logiciel (SDK) Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, automl
ms.author: mithigpe
author: minthigpen
ms.date: 07/09/2020
ms.openlocfilehash: cf1eb1c72cc93fcb72862b15f3884969915c24dd
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360647"
---
# <a name="interpretability-model-explanations-in-automated-machine-learning-preview"></a>Interprétabilité : explications des modèles en machine learning automatisé (version préliminaire)



Cet article explique comment obtenir des explications pour le Machine Learning (ML) automatisé dans Azure Machine Learning. Le ML automatisé vous aide à comprendre l’importance des caractéristiques traitées. 

Toutes les versions du Kit de développement logiciel (SDK) après la version 1.0.85 définissent `model_explainability=True` par défaut. Dans le Kit de développement logiciel (SDK) version 1.0.85 et ses versions antérieures, les utilisateurs doivent définir `model_explainability=True` dans l’objet `AutoMLConfig` afin d’utiliser l’interprétabilité du modèle. 

Dans cet article, vous apprendrez comment :

- Effectuer une interprétabilité pendant la formation pour le meilleur modèle ou n’importe quel modèle.
- Activer les visualisations pour vous aider à voir les modèles dans les données et les explications.
- Implémenter une interprétabilité pendant l’inférence ou le scoring.

## <a name="prerequisites"></a>Prérequis

- Fonctionnalités d’interprétabilité. Exécutez `pip install azureml-interpret` pour obtenir le package nécessaire.
- Des connaissances en matière de génération d’expériences de ML automatisé. Pour plus d’informations sur la façon d’utiliser le Kit de développement logiciel (SDK) Azure Machine Learning, suivez ce [tutoriel sur les modèles de régression](tutorial-auto-train-models.md) ou consultez la rubrique [Configurer des expériences de ML automatisé](how-to-configure-auto-train.md).

## <a name="interpretability-during-training-for-the-best-model"></a>Interprétabilité pendant l’entraînement pour le meilleur modèle

Récupérez l’explication à partir de `best_run`, qui inclut des explications pour les caractéristiques traitées.

> [!Warning]
> L’interprétation, la meilleure explication du modèle, n’est pas disponible pour les expériences de prévision de ML automatique, qui recommandent les algorithmes suivants comme meilleur modèle : 
> * ForecastTCN
> * Moyenne 
> * Naive
> * Moyenne saisonnière 
> * Naive saisonnière

### <a name="download-engineered-feature-importance-from-artifact-store"></a>Télécharger l’importance des caractéristiques d’ingénierie à partir de la boutique d’artefacts

Vous pouvez utiliser `ExplanationClient` pour télécharger des explications de caractéristiques traitées à partir du magasin d’artefacts de `best_run`. 

```python
from azureml.interpret import ExplanationClient

client = ExplanationClient.from_run(best_run)
engineered_explanations = client.download_model_explanation(raw=False)
print(engineered_explanations.get_feature_importance_dict())
```

## <a name="interpretability-during-training-for-any-model"></a>Interprétabilité pendant l’entraînement de n’importe quel modèle 

Lorsque vous calculez des explications de modèle et que vous les visualisez, vous n’êtes pas limité à une explication de modèle existante pour un modèle de ML automatisé. Vous pouvez également obtenir une explication de votre modèle avec des données de test différentes. Les étapes de cette section vous montrent comment calculer et visualiser l’importance des caractéristiques traitées en fonction de vos données de test.

### <a name="retrieve-any-other-automl-model-from-training"></a>Récupérer tout autre modèle AutoML à partir de l’entraînement

```python
automl_run, fitted_model = local_run.get_output(metric='accuracy')
```

### <a name="set-up-the-model-explanations"></a>Configurer les explications de modèle

Utilisez `automl_setup_model_explanations` pour obtenir des explications relatives aux caractéristiques traitées. Le `fitted_model` peut générer les éléments suivants :

- Données proposées à partir d’exemples formés ou test
- Listes de noms de caractéristiques traitées
- Classes accessibles dans votre colonne étiquetée dans des scénarios de classification

Le `automl_explainer_setup_obj` contient toutes les structures de la liste ci-dessus.

```python
from azureml.train.automl.runtime.automl_explain_utilities import automl_setup_model_explanations

automl_explainer_setup_obj = automl_setup_model_explanations(fitted_model, X=X_train, 
                                                             X_test=X_test, y=y_train, 
                                                             task='classification')
```

### <a name="initialize-the-mimic-explainer-for-feature-importance"></a>Initialiser l’explicatif d’imitation pour l’importance des caractéristiques

Afin de générer une explication pour les modèles AutoML, utilisez la classe `MimicWrapper`. Vous pouvez initialiser le MimicWrapper avec les paramètres suivants :

- L’objet de configuration explicatif
- Votre espace de travail
- Modèle de substitution pour expliquer le modèle de ML automatisé `fitted_model`

MimicWrapper prend également l’objet `automl_run` où les explications de caractéristiques traitées seront chargées.

```python
from azureml.interpret import MimicWrapper

# Initialize the Mimic Explainer
explainer = MimicWrapper(ws, automl_explainer_setup_obj.automl_estimator,
                         explainable_model=automl_explainer_setup_obj.surrogate_model, 
                         init_dataset=automl_explainer_setup_obj.X_transform, run=automl_run,
                         features=automl_explainer_setup_obj.engineered_feature_names, 
                         feature_maps=[automl_explainer_setup_obj.feature_map],
                         classes=automl_explainer_setup_obj.classes,
                         explainer_kwargs=automl_explainer_setup_obj.surrogate_model_params)
```

### <a name="use-mimicexplainer-for-computing-and-visualizing-engineered-feature-importance"></a>Utiliser MimicExplainer pour le calcul et la visualisation de l’importance des caractéristiques traitées

Vous pouvez appeler a méthode `explain()` dans MimicWrapper avec les exemples de tests transformés pour obtenir l’importance des caractéristiques traitées générées. Vous pouvez également utiliser `ExplanationDashboard` pour consulter la visualisation du tableau de bord des valeurs de l’importance des caractéristiques traitées générées par des caractériseurs de ML automatisé.

```python
engineered_explanations = explainer.explain(['local', 'global'], eval_dataset=automl_explainer_setup_obj.X_test_transform)
print(engineered_explanations.get_feature_importance_dict())
```

## <a name="interpretability-during-inference"></a>Interprétabilité pendant l’inférence

Dans cette section, vous allez apprendre à rendre un modèle de ML automatisé opérationnel avec l’explicatif qui a été utilisé pour calculer les explications dans la section précédente.

### <a name="register-the-model-and-the-scoring-explainer"></a>Inscrire le modèle et l’explicatif de scoring

Utilisez `TreeScoringExplainer` pour créer l’explicatif de scoring, qui calculera les valeurs d’importance des caractéristiques traitées au moment de l’inférence. Vous initialisez l’explicatif de scoring avec la valeur `feature_map` qui a été calculée précédemment. 

Enregistrez l’explicatif de scoring, puis inscrivez le modèle et l’explicatif de scoring auprès du service Gestion des modèles ML. Exécutez le code ci-dessous :

```python
from azureml.interpret.scoring.scoring_explainer import TreeScoringExplainer, save

# Initialize the ScoringExplainer
scoring_explainer = TreeScoringExplainer(explainer.explainer, feature_maps=[automl_explainer_setup_obj.feature_map])

# Pickle scoring explainer locally
save(scoring_explainer, exist_ok=True)

# Register trained automl model present in the 'outputs' folder in the artifacts
original_model = automl_run.register_model(model_name='automl_model', 
                                           model_path='outputs/model.pkl')

# Register scoring explainer
automl_run.upload_file('scoring_explainer.pkl', 'scoring_explainer.pkl')
scoring_explainer_model = automl_run.register_model(model_name='scoring_explainer', model_path='scoring_explainer.pkl')
```

### <a name="create-the-conda-dependencies-for-setting-up-the-service"></a>Créer les dépendances Conda pour la configuration du service

Ensuite, créez les dépendances d’environnement nécessaires dans le conteneur du modèle déployé. Notez que vous devez faire figurer azureml-defaults avec une version supérieure ou égale à 1.0.45 comme dépendance pip, car elle contient les fonctionnalités nécessaires pour héberger le modèle en tant que service web.

```python
from azureml.core.conda_dependencies import CondaDependencies

azureml_pip_packages = [
    'azureml-interpret', 'azureml-train-automl', 'azureml-defaults'
]

myenv = CondaDependencies.create(conda_packages=['scikit-learn', 'pandas', 'numpy', 'py-xgboost<=0.80'],
                                 pip_packages=azureml_pip_packages,
                                 pin_sdk_version=True)

with open("myenv.yml","w") as f:
    f.write(myenv.serialize_to_string())

with open("myenv.yml","r") as f:
    print(f.read())

```

### <a name="deploy-the-service"></a>Déployer le service

Déployez le service à l’aide du fichier Conda et du fichier de scoring des étapes précédentes.

```python
from azureml.core.webservice import Webservice
from azureml.core.webservice import AciWebservice
from azureml.core.model import Model, InferenceConfig
from azureml.core.environment import Environment

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1,
                                               memory_gb=1,
                                               tags={"data": "Bank Marketing",  
                                                     "method" : "local_explanation"},
                                               description='Get local explanations for Bank marketing test data')
myenv = Environment.from_conda_specification(name="myenv", file_path="myenv.yml")
inference_config = InferenceConfig(entry_script="score_local_explain.py", environment=myenv)

# Use configs and models generated above
service = Model.deploy(ws,
                       'model-scoring',
                       [scoring_explainer_model, original_model],
                       inference_config,
                       aciconfig)
service.wait_for_deployment(show_output=True)
```

### <a name="inference-with-test-data"></a>Inférer à l’aide de données de test

Inférez à l’aide de données de test pour voir la valeur prédite du modèle de ML automatisé. Affichez l’importance des caractéristiques traitées pour la valeur prédite.

```python
if service.state == 'Healthy':
    # Serialize the first row of the test data into json
    X_test_json = X_test[:1].to_json(orient='records')
    print(X_test_json)
    # Call the service to get the predictions and the engineered explanations
    output = service.run(X_test_json)
    # Print the predicted value
    print(output['predictions'])
    # Print the engineered feature importances for the predicted value
    print(output['engineered_local_importance_values'])
```

### <a name="visualize-to-discover-patterns-in-data-and-explanations-at-training-time"></a>Visualiser pour découvrir des modèles dans les données et les explications au moment de la formation

Vous pouvez visualiser le graphique d’importance d’une caractéristique dans votre espace de travail dans [Azure Machine Learning Studio](https://ml.azure.com). Une fois votre exécution de ML automatisée terminée, sélectionnez **Afficher les détails du modèle** pour afficher une exécution spécifique. Sélectionnez l’onglet **Explications** pour voir le tableau de bord de visualisation des explications.

[![Architecture d’interprétabilité de Machine Learning](./media/how-to-machine-learning-interpretability-automl/automl-explainability.png)](./media/how-to-machine-learning-interpretability-automl/automl-explainability.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la façon dont vous pouvez activer les explications des modèles et l’importance d’une caractéristique dans des domaines du Kit de développement logiciel (SDK) Azure Machine Learning autres que le Machine Learning automatisé, consultez l’[article concept sur l’interprétabilité](how-to-machine-learning-interpretability.md).
