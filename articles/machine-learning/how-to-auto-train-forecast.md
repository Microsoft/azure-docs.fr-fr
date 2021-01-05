---
title: Entraîner automatiquement un modèle de prévision de série chronologique
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser Azure Machine Learning pour entraîner un modèle de régression de prévisions de série chronologique en utilisant le Machine Learning automatisé.
services: machine-learning
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1, automl
ms.date: 08/20/2020
ms.openlocfilehash: 47cc67b408ff7fa50a244fffa8d41e640df0ecf3
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796429"
---
# <a name="auto-train-a-time-series-forecast-model"></a>Entraîner automatiquement un modèle de prévision de série chronologique


Dans cet article, vous allez apprendre à configurer et à effectuer l'apprentissage d’un modèle de régression de prévisions de séries chronologiques à l’aide du Machine Learning automatisé dans le [Kit de développement logiciel (SDK) Python Azure Machine Learning](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py). 

Pour cela, vous devez : 

> [!div class="checklist"]
> * Préparer les données pour la modélisation de série chronologique.
> * Configurer des paramètres spécifiques de série chronologique dans un objet [`AutoMLConfig`](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig).
> * Exécuter des prédictions avec les données de série chronologique.

Pour une expérience avec peu de code, consultez le [tutoriel : Prévoir la demande à l’aide du Machine Learning automatisé](tutorial-automated-ml-forecast.md) pour un exemple de prévision de série chronologique utilisant le Machine Learning automatisé dans [Azure Machine Learning Studio](https://ml.azure.com/).

Contrairement aux méthodes classiques de séries chronologiques, dans Machine Learning automatisé, les valeurs des séries chronologiques sont ajoutées à un tableau croisé dynamique pour devenir des dimensions supplémentaires pour le régresseur, avec d’autres prédicteurs. Cette approche intègre plusieurs variables contextuelles et les relations qu’elles entretiennent au cours de l’apprentissage. Dans la mesure où plusieurs facteurs peuvent influencer une prévision, cette méthode s’aligne bien sur les scénarios de prévision du monde réel. Par exemple, lors de la prévision des ventes, les interactions entre les tendances historiques, les taux de change et les prix déterminent conjointement le résultat des ventes. 

## <a name="prerequisites"></a>Prérequis

Pour cet article, vous avez besoin des éléments suivants : 

* Un espace de travail Azure Machine Learning. Pour créer l’espace de travail, voir [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Cet article suppose une connaissance de base en matière de configuration d’une expérience de Machine Learning automatisé. Suivez le [didacticiel](tutorial-auto-train-models.md) ou les [procédures](how-to-configure-auto-train.md) pour connaître les principaux modèles de conception des expériences de Machine Learning automatisé.

## <a name="preparing-data"></a>Préparation des données

La principale différence entre un type de tâche de régression de prévisions et un type de tâche de régression au sein d’AutoML consiste à intégrer une fonctionnalité dans vos données qui représente une série chronologique valide. Une série chronologique normale a une fréquence cohérente et bien définie et a une valeur à chaque point de l’exemple dans un intervalle de temps continu. 

Examinons la capture instantanée suivante d’un fichier `sample.csv`.
Ce jeu de données provient de données de ventes quotidiennes d'une entreprise qui possède deux magasins différents, A et B. 

En outre, il existe des fonctionnalités pour

 *  `week_of_year` : permet au modèle de détecter un caractère saisonnier hebdomadaire.
* `day_datetime` : représente une série chronologique propre avec une fréquence quotidienne.
* `sales_quantity` : colonne cible pour l’exécution des prédictions. 

```output
day_datetime,store,sales_quantity,week_of_year
9/3/2018,A,2000,36
9/3/2018,B,600,36
9/4/2018,A,2300,36
9/4/2018,B,550,36
9/5/2018,A,2100,36
9/5/2018,B,650,36
9/6/2018,A,2400,36
9/6/2018,B,700,36
9/7/2018,A,2450,36
9/7/2018,B,650,36
```


Lisez les données dans une trame de données Pandas, puis utilisez la fonction `to_datetime` pour vous assurer que la série chronologique est un type `datetime`.

```python
import pandas as pd
data = pd.read_csv("sample.csv")
data["day_datetime"] = pd.to_datetime(data["day_datetime"])
```

Dans ce cas, les données sont déjà triées dans l’ordre croissant par le champ d’heure `day_datetime`. Toutefois, lorsque vous configurez une expérience, vérifiez que la colonne d’heure souhaitée est triée dans l’ordre croissant pour générer une série chronologique valide. 

Le code suivant, 
* suppose que les données contiennent 1 000 enregistrements et effectue un fractionnement déterministe dans les données pour créer des jeux de données de formation et de test. 
* Identifie la colonne d’étiquette comme `sales_quantity`.
* Sépare le champ d’étiquette de `test_data` pour former l’ensemble `test_target`.

```python
train_data = data.iloc[:950]
test_data = data.iloc[-50:]

label =  "sales_quantity"
 
test_labels = test_data.pop(label).values
```

> [!IMPORTANT]
> Lors de l’entraînement d’un modèle pour la prévision de valeurs futures, assurez-vous que toutes les fonctionnalités utilisées pour l’entraînement peuvent être utilisées lors de l’exécution de prédictions à l’horizon prévu. <br> <br>Par exemple, lorsque vous créez une prévision de la demande, l’intégration d’une fonctionnalité pour le prix actuel du stock peut augmenter considérablement la précision de l’apprentissage. Toutefois, si vous prévoyez à un horizon lointain, vous ne pourrez peut-être pas prévoir avec précision les futures valeurs d’actions correspondant aux points de séries chronologiques futures et la précision du modèle pourrait en pâtir.

<a name="config"></a>

## <a name="training-and-validation-data"></a>Données de formation et de validation

Vous pouvez spécifier des jeux de données de formation et de validation distincts directement dans l’objet `AutoMLConfig`.   En savoir plus sur [AutoMLConfig](#configure-experiment).

Pour les prévisions de série chronologique, seule la **validation croisée à origine dynamique** est utilisée pour la validation par défaut. Transmettez les données d’entraînement et de validation en même temps et définissez le nombre de replis de validation croisée avec le paramètre `n_cross_validations` dans votre `AutoMLConfig`. Elle divise la série en données d’apprentissage et données de validation à l’aide d’un point d’origine. Les échantillons de validation croisée sont générés par glissement de l’origine temporelle. Cette stratégie permet de préserver l’intégrité des données des séries chronologiques et d’éliminer le risque de fuite de données

![Validation croisée à origine dynamique](./media/how-to-auto-train-forecast/ROCV.svg)

Vous pouvez également fournir vos propres données de validation. Pour en savoir plus, consultez [Configurer les fractionnements de données et la validation croisée dans AutoML](how-to-configure-cross-validation-data-splits.md#provide-validation-data).


```python
automl_config = AutoMLConfig(task='forecasting',
                             n_cross_validations=3,
                             ...
                             **time_series_settings)
```

En savoir plus sur la façon dont AutoML applique la validation croisée afin [d’empêcher les modèles de surajustement](concept-manage-ml-pitfalls.md#prevent-over-fitting).

## <a name="configure-experiment"></a>Configurer une expérience

L’objet [`AutoMLConfig`](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?preserve-view=true&view=azure-ml-py) définit les paramètres et les données nécessaires pour une tâche de Machine Learning automatisé. La configuration d’un modèle de prévisions est semblable à celle d’un modèle de régression standard, mais certains modèles et certaines options de configuration et étapes de caractérisation existent spécifiquement pour les données de séries chronologiques. 

### <a name="supported-models"></a>Modèles pris en charge
Le Machine Learning automatisé essaie automatiquement différents modèles et algorithmes dans le cadre du processus de création et de paramétrage du modèle. En tant qu’utilisateur, vous n’avez pas besoin de spécifier l’algorithme. Pour les expériences de prévision, les modèles natifs de série chronologique et de Deep Learning font partie du système de recommandation. Le tableau suivant récapitule ce sous-ensemble de modèles. 

>[!Tip]
> Les modèles de régression traditionnels sont également testés dans le cadre du système de recommandation pour les expériences de prévision. Pour obtenir la liste complète des modèles, consultez le [tableau des modèles pris en charge](how-to-configure-auto-train.md#supported-models). 

Modèles| Description | Avantages
----|----|---
Prophet (préversion)|Prophet fonctionne mieux avec les séries chronologiques ayant des effets saisonniers forts et plusieurs saisons de données historiques. Pour tirer parti de ce modèle, installez-le en local à l’aide de `pip install fbprophet`. | Précis, rapide et robuste aux valeurs hors norme, aux données manquantes et aux évolutions profondes des séries chronologiques.
Auto-ARIMA (préversion)|La moyenne mobile intégrée auto-régressive (ARIMA) est idéale pour les données fixes, c’est-à-dire les données dont les propriétés statistiques, comme la moyenne et la variance, sont constantes sur la totalité du jeu. Par exemple, quand on lance une pièce de monnaie, la probabilité d'obtenir face est de 50 %, que ce soit aujourd'hui, demain ou l'année prochaine.| Idéal pour les séries univariées, car les valeurs passées sont utilisées pour prédire les valeurs futures.
ForecastTCN (préversion)| ForecastTCN est un modèle de réseau neuronal conçu pour traiter les tâches de prévision les plus exigeantes, capturant les tendances non linéaires locales et globales dans les données, ainsi que les relations entre les séries chronologiques.|Capable de mobiliser des tendances complexes dans les données et de s’adapter facilement aux jeux de données les plus volumineux.

### <a name="configuration-settings"></a>Paramètres de configuration

Comme pour un problème de régression, vous définissez les paramètres d’entraînement standard comme type de tâche, le nombre d’itérations, les données d’apprentissage et le nombre de validations croisées. Pour les tâches de prévision, il existe des paramètres supplémentaires qui doivent être définis et qui affectent l’expérience. 

Ces paramètres supplémentaires sont résumés dans le tableau suivant. Consultez la [documentation de référence de la classe ForecastingParameter](/python/api/azureml-automl-core/azureml.automl.core.forecasting_parameters.forecastingparameters?preserve-view=true&view=azure-ml-py) pour obtenir des modèles de conception de la syntaxe.

| Nom du&nbsp;paramètre | Description | Obligatoire |
|-------|-------|-------|
|`time_column_name`|Permet de spécifier la colonne DateHeure dans les données d’entrée utilisées pour la génération de la série chronologique et la déduction de sa fréquence.|✓|
|`forecast_horizon`|Définit le nombre de périodes à venir que vous souhaitez prévoir. L’horizon est exprimé en unités de fréquence de série chronologique. Les unités sont basées sur l’intervalle de temps de vos données d’apprentissage (par exemple mensuelles ou hebdomadaires) que l’analyste doit prévoir.|✓|
|`enable_dnn`|[Activez les DNN de prévisions]().||
|`time_series_id_column_names`|La ou les noms de colonne utilisés pour identifier de manière unique la série chronologique dans des données qui ont plusieurs lignes avec le même horodatage. Si les identificateurs de série chronologique ne sont pas définis, le jeu de données est supposé être une série chronologique. Pour en savoir plus sur les séries chronologiques uniques, consultez [energy_demand_notebook](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand).||
|`freq`| Fréquence du jeu de données de série chronologique. Ce paramètre représente la fréquence attendue des événements, par exemple tous les jours, toutes les semaines, tous les ans, etc. Elle doit constituer un [alias de décalage Pandas](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#dateoffset-objects).||
|`target_lags`|Nombre de lignes selon lequel décaler les valeurs cibles en fonction de la fréquence des données. Le décalage est représenté sous la forme d’une liste ou d’un entier unique. Un décalage est nécessaire en l’absence de correspondance ou de corrélation par défaut des relations entre les variables indépendantes et la variable dépendante. ||
|`feature_lags`| Les fonctionnalités à décaler seront automatiquement déterminées par Machine Learning automatisé lorsque `target_lags` sont définis et `feature_lags` est défini sur `auto`. L’activation des décalages de fonctionnalités peut contribuer à améliorer l’exactitude. Les décalages de fonctionnalités sont désactivés par défaut. ||
|`target_rolling_window_size`|*n* périodes historiques à utiliser pour générer des valeurs prédites, < = taille du jeu d’apprentissage. En cas d’omission, *n* est la taille du jeu d’apprentissage complet. Spécifiez ce paramètre si vous souhaitez prendre en compte seulement une partie des données historiques pour l’entraînement du modèle. En savoir plus sur l’[agrégation de fenêtres dynamiques cibles](#target-rolling-window-aggregation).||
|`short_series_handling_config`| Permet une gestion courte des séries chronologiques pour éviter toute défaillance due à des données insuffisantes au cours de l’apprentissage. Le paramètre de gestion des séries courtes est défini sur `auto` par défaut. Découvrez-en plus sur la [gestion des séries courtes](#short-series-handling).|


Le code suivant, 
* Tire parti de la classe [`ForecastingParameters`](/python/api/azureml-automl-core/azureml.automl.core.forecasting_parameters.forecastingparameters?preserve-view=true&view=azure-ml-py) pour définir les paramètres de prévisions associés à l'apprentissage de l'expérience.
* Définit le `time_column_name` sur le champ `day_datetime` dans le jeu de données. 
* Définit le paramètre `time_series_id_column_names` sur `"store"`. Cela permet de s’assurer que **deux groupes de séries chronologiques distincts** sont créés pour les données ; une pour les banques A et B.
* Définit le `forecast_horizon` sur 50 afin de prédire pour l’ensemble du jeu de tests. 
* Définit une fenêtre de prévision sur 10 périodes avec `target_rolling_window_size`
* Spécifie un décalage unique sur les valeurs cibles pour deux périodes futures avec le paramètre `target_lags`. 
* Définit `target_lags` sur le paramètre « Automatique » recommandé, qui détectera automatiquement cette valeur pour vous.

```python
from azureml.automl.core.forecasting_parameters import ForecastingParameters

forecasting_parameters = ForecastingParameters(time_column_name='day_datetime', 
                                               forecast_horizon=50,
                                               time_series_id_column_names=["store"],
                                               freq='W',
                                               target_lags='auto',
                                               target_rolling_window_size=10)
                                              
```

Ces `forecasting_parameters` sont ensuite transmises à votre objet `AutoMLConfig` standard, ainsi que le type de tâche `forecasting`, la métrique principale, les critères de sortie et les données de formation. 

```python
from azureml.core.workspace import Workspace
from azureml.core.experiment import Experiment
from azureml.train.automl import AutoMLConfig
import logging

automl_config = AutoMLConfig(task='forecasting',
                             primary_metric='normalized_root_mean_squared_error',
                             experiment_timeout_minutes=15,
                             enable_early_stopping=True,
                             training_data=train_data,
                             label_column_name=label,
                             n_cross_validations=5,
                             enable_ensembling=False,
                             verbosity=logging.INFO,
                             **forecasting_parameters)
```

### <a name="featurization-steps"></a>Étapes de caractérisation

Dans chaque expérience de Machine Learning automatisée, des techniques de mise à l’échelle automatique et de normalisation sont appliquées par défaut à vos données. Ces techniques sont des types de **caractérisation** qui aident *certains* algorithmes sensibles aux caractéristiques à des échelles différentes. En savoir plus sur les étapes de caractérisation par défaut dans [Caractérisation dans AutoML](how-to-configure-auto-features.md#automatic-featurization)

Toutefois, les étapes suivantes sont effectuées uniquement pour les types de tâches `forecasting` :

* Détecter un exemple de fréquence de série chronologique (par exemple horaire, quotidien, hebdomadaire) et créer de nouveaux enregistrements pour les points temporels absents pour rendre la série continue.
* Imputer les valeurs manquantes dans la cible (via un préremplissage) et les colonnes de fonctionnalités (via les valeurs de colonne médiane)
* Créer des fonctionnalités basées sur des identificateurs de série chronologique pour activer des effets fixes sur différentes séries
* Créer des caractéristiques temporelles pour faciliter l’apprentissage de modèles saisonniers
* Encoder des variables catégorielles avec des quantités numériques

Pour obtenir un résumé des fonctionnalités qui sont créées à la suite de ces étapes, consultez [Transparence de la caractérisation](how-to-configure-auto-features.md#featurization-transparency)

> [!NOTE]
> Les étapes de caractérisation du Machine Learning automatisé (normalisation des fonctionnalités, gestion des données manquantes, conversion de texte en valeurs numériques, etc.) font partie du modèle sous-jacent. Lorsque vous utilisez le modèle pour des prédictions, les étapes de caractérisation qui sont appliquées pendant la formation sont appliquées automatiquement à vos données d’entrée.

#### <a name="customize-featurization"></a>Personnaliser la caractérisation

Vous pouvez également personnaliser vos paramètres de caractérisation pour vous assurer que les données et les caractéristiques utilisées pour effectuer l'apprentissage de votre modèle ML génèrent des prédictions pertinentes. 

Les personnalisations prises en charge pour des tâches `forecasting` sont les suivantes :

|Personnalisation|Définition|
|--|--|
|**Mise à jour de l’objectif de la colonne**|Remplacer le type de caractéristique détecté automatiquement pour la colonne spécifiée.|
|**Mise à jour des paramètres du transformateur** |Mettre à jour les paramètres du transformateur spécifié. Prend actuellement en charge *imputer* (fill_value et median).|
|**Suppression de colonnes** |Indique les colonnes à supprimer de la caractérisation.|

Pour personnaliser les caractérisations avec le kit de développement logiciel (SDK), spécifiez `"featurization": FeaturizationConfig` dans votre objet `AutoMLConfig`. En savoir plus sur les [caractérisations personnalisées](how-to-configure-auto-features.md#customize-featurization).

```python
featurization_config = FeaturizationConfig()

# `logQuantity` is a leaky feature, so we remove it.
featurization_config.drop_columns = ['logQuantitity']

# Force the CPWVOL5 feature to be of numeric type.
featurization_config.add_column_purpose('CPWVOL5', 'Numeric')

# Fill missing values in the target column, Quantity, with zeroes.
featurization_config.add_transformer_params('Imputer', ['Quantity'], {"strategy": "constant", "fill_value": 0})

# Fill mising values in the `INCOME` column with median value.
featurization_config.add_transformer_params('Imputer', ['INCOME'], {"strategy": "median"})
```

Si vous utilisez le studio Azure Machine Learning pour votre expérience, consultez l’article sur la [personnalisation de la caractérisation dans le studio](how-to-use-automated-ml-for-ml-models.md#customize-featurization).

## <a name="optional-configurations"></a>Configurations facultatives

Des configurations facultatives supplémentaires sont disponibles pour les tâches de prévisions, telles que l’activation du Deep Learning et la spécification d’une agrégation de fenêtres dynamiques cibles. 

### <a name="enable-deep-learning"></a>Activez le Deep Learning

> [!NOTE]
> Le support DNN pour les prévisions dans le Machine Learning automatisé est en **préversion** et n’est pas pris en charge pour les exécutions locales.

Vous pouvez également tirer parti du Deep Learning avec des réseaux neuronaux profonds, DNN, pour améliorer les scores de votre modèle. Le Deep Learning du ML automatisé permet de prévoir des données de série chronologique univariées et multivariées.

Les modèles Deep Learning ont trois capacités intrinsèques :
1. Ils peuvent apprendre de mappages arbitraires des entrées aux sorties.
1. Ils prennent en charge plusieurs entrées et sorties.
1. Ils peuvent extraire automatiquement des modèles dans les données d’entrée qui s’étendent sur des séquences longues. 

Pour activer le Deep Learning, définissez la `enable_dnn=True` dans l’objet `AutoMLConfig`.

```python
automl_config = AutoMLConfig(task='forecasting',
                             enable_dnn=True,
                             ...
                             **forecasting_parameters)
```
> [!Warning]
> Lorsque vous activez DNN pour les expériences créées avec le Kit de développement logiciel (SDK), les [meilleures explications des modèles](how-to-machine-learning-interpretability-automl.md) sont désactivées.

Pour activer DNN pour une expérience AutoML créée dans le studio Azure Machine Learning, consultez les [paramètres de type de tâche dans la procédure Studio](how-to-use-automated-ml-for-ml-models.md#create-and-run-experiment).

Pour obtenir un exemple de code détaillé utilisant des DNN, consultez [Beverage Production Forecasting notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-beer-remote/auto-ml-forecasting-beer-remote.ipynb).

### <a name="target-rolling-window-aggregation"></a>Agrégation de fenêtres dynamiques cibles
La dernière valeur de la cible constitue souvent la meilleure information dont un prédicteur peut disposer.  Les agrégations de fenêtres dynamiques cibles permettent d’ajouter une agrégation dynamique de valeurs de données comme caractéristiques. Le fait de générer ces nouvelles caractéristiques et de les utiliser comme données contextuelles supplémentaires contribue à la précision du modèle d’apprentissage.

Par exemple, imaginons que vous souhaitiez prédire la demande d’énergie. Nous pouvons ajouter une caractéristique de fenêtre dynamique de trois jours pour tenir compte des modifications thermiques des espaces chauffés. Dans cet exemple, créez cette fenêtre en définissant `target_rolling_window_size= 3` dans le constructeur `AutoMLConfig`. 

Le tableau montre l’ingénierie de caractéristiques obtenue qui se produit lors de l’application de l’agrégation de fenêtres. Les colonnes pour **minimum, maximum** et **somme** sont générées sur une fenêtre glissante de trois en fonction des paramètres définis. Chaque ligne comporte une nouvelle caractéristique calculée. Dans le cas du timestamp du 8 septembre 2017 à 4 h, les valeurs maximum, minimum et somme sont calculées suivant les **valeurs de la demande** du 8 septembre 2017 entre 1 h et 3 h. Cette fenêtre de trois se déplace de façon à remplir les données des lignes restantes.

![Fenêtre dynamique cible](./media/how-to-auto-train-forecast/target-roll.svg)

Consultez un exemple de code Python tirant parti de la [caractéristique d’agrégation de fenêtres dynamiques cibles](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb).

### <a name="short-series-handling"></a>Gestion des séries courtes

Le ML automatisé considère une série chronologique comme une **série courte** si le nombre de points de données est insuffisant pour mener les phases d'apprentissage et de validation du développement du modèle. Le nombre de points de données varie d'une expérience à l'autre et dépend du paramètre max_horizon, du nombre de divisions pour la validation croisée et de la longueur de la recherche arrière du modèle, c'est-à-dire le maximum d'historique nécessaire pour construire les caractéristiques de la série chronologique. Pour connaître le calcul exact, consultez la [documentation de référence de short_series_handling_configuration](/python/api/azureml-automl-core/azureml.automl.core.forecasting_parameters.forecastingparameters?preserve-view=true&view=azure-ml-py#short-series-handling-configuration).

Le ML automatisé permet par défaut une gestion des séries courtes avec le paramètre `short_series_handling_configuration` de l'objet `ForecastingParameters`. 

Pour activer la gestion des séries courtes, le paramètre `freq` doit également être défini. Pour définir une fréquence horaire, nous allons définir `freq='H'`. Consultez les options de chaîne de fréquence [ici](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#dateoffset-objects). Pour modifier le comportement par défaut, `short_series_handling_configuration = 'auto'`, mettez à jour le paramètre `short_series_handling_configuration` dans votre objet `ForecastingParameter`.  

```python
from azureml.automl.core.forecasting_parameters import ForecastingParameters

forecast_parameters = ForecastingParameters(time_column_name='day_datetime', 
                                            forecast_horizon=50,
                                            short_series_handling_configuration='auto',
                                            freq = 'H',
                                            target_lags='auto')
```
Le tableau suivant récapitule les paramètres disponibles pour `short_series_handling_config`.
 
|Paramètre|Description
|---|---
|`auto`| Le comportement par défaut en matière de gestion des séries courtes est le suivant : <li> *Si toutes les séries sont courtes*, remplissez les données. <br> <li> *Si toutes les séries ne sont pas courtes*, supprimez-les. 
|`pad`| Si `short_series_handling_config = pad`, le ML automatisé ajoute des valeurs aléatoires à chaque série courte trouvée. Vous trouverez ci-dessous la liste des types de colonnes et de leur contenu : <li>Colonnes d'objets avec « n'est pas un nombre » (NAN) <li> Colonnes numériques avec 0 <li> Colonnes booléennes/logiques avec False <li> La colonne cible est remplie avec des valeurs aléatoires, avec une moyenne de zéro et un écart type de 1. 
|`drop`| Si `short_series_handling_config = drop`, le ML automatisé supprime la série courte ; celle-ci ne sera donc pas utilisée pour l'apprentissage ou la prédiction. Les prédictions de ces séries renverront NAN.
|`None`| Aucune série n'est remplie ou supprimée

>[!WARNING]
>Le remplissage peut avoir un impact sur la précision du modèle qui en résulte, car nous introduisons des données artificielles dans le seul but de terminer l'apprentissage sans rencontrer d'échecs. <br> <br> Si la plupart des séries sont courtes, vous pouvez également constater un impact sur les résultats de l'explicabilité.

## <a name="run-the-experiment"></a>Exécuter l’expérience 

Une fois votre objet `AutoMLConfig` prêt, vous pouvez soumettre l’expérience. Une fois le modèle terminé, récupérez la meilleure itération d’exécution.

```python
ws = Workspace.from_config()
experiment = Experiment(ws, "Tutorial-automl-forecasting")
local_run = experiment.submit(automl_config, show_output=True)
best_run, fitted_model = local_run.get_output()
```
 
## <a name="forecasting-with-best-model"></a>Prévision avec le meilleur modèle

Utilisez la meilleure itération de modèle pour prévoir les valeurs du jeu de données de test.

La fonction `forecast()` permet de spécifier quand les prédictions doivent démarrer, contrairement à la `predict()`, qui est généralement utilisée pour les tâches de classification et de régression.

Dans l’exemple suivant, vous commencez par remplacer toutes les valeurs dans `y_pred` par `NaN`. L’origine de la prévision sera à la fin des données de formation dans ce cas. Toutefois, si vous avez remplacé uniquement le second semestre de `y_pred` par `NaN`, la fonction laisserait les valeurs numériques du premier semestre inchangées, mais prévoirait les valeurs `NaN` au second semestre. La fonction retourne à la fois les valeurs prédites et les fonctionnalités alignées.

Vous pouvez également utiliser le paramètre `forecast_destination` dans la fonction `forecast()` pour prévoir les valeurs jusqu’à une date spécifiée.

```python
label_query = test_labels.copy().astype(np.float)
label_query.fill(np.nan)
label_fcst, data_trans = fitted_pipeline.forecast(
    test_data, label_query, forecast_destination=pd.Timestamp(2019, 1, 8))
```

Calculez l’erreur quadratique moyenne (REQM) entre les valeurs `actual_labels` réelles et les valeurs prédites dans `predict_labels`.

```python
from sklearn.metrics import mean_squared_error
from math import sqrt

rmse = sqrt(mean_squared_error(actual_labels, predict_labels))
rmse
```

Maintenant que la précision du modèle global a été déterminée, l’étape suivante la plus réaliste consiste à utiliser le modèle pour prévoir des valeurs futures inconnues. 

Fournissez un jeu de données au même format que le jeu de test `test_data`, mais avec des dates/heures futures, et le leu de prédiction résultant correspond aux valeurs prédites pour chaque étape de la série chronologique. Supposons que les derniers enregistrements de la série chronologique dans le jeu de données aient été datés du 31/12/2018. Pour prévoir la demande pour le jour suivant (ou d’autant de périodes pour lesquelles vous avez besoin d’effectuer des prévisions, < = `forecast_horizon`), créez un seul enregistrement de série chronologique pour chaque magasin pour le 01/01/2019.

```output
day_datetime,store,week_of_year
01/01/2019,A,1
01/01/2019,A,1
```

Répétez les étapes nécessaires pour charger ces données futures dans une trame de données, puis exécutez `best_run.predict(test_data)` pour prédire les valeurs futures.

> [!NOTE]
> Les valeurs ne peuvent pas être prédites pour un nombre de périodes supérieur à `forecast_horizon`. Le modèle doit être ré-entraîné à un horizon plus lointain pour prédire les valeurs futures au-delà de l’horizon actuel.


## <a name="example-notebooks"></a>Exemples de notebooks
Consultez les [exemples de notebooks de prévision](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning) pour obtenir des exemples de code détaillés de la configuration de prévision avancée, notamment :

* [Détection et personnalisation de congé](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/auto-ml-forecasting-bike-share.ipynb)
* [Validation croisée d’origine](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb)
* [Décalages configurables](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/auto-ml-forecasting-bike-share.ipynb)
* [Caractéristiques des agrégations des périodes mobiles](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb)
* [DNN](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-beer-remote/auto-ml-forecasting-beer-remote.ipynb)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus d’informations sur [comment et où déployer un modèle](how-to-deploy-and-where.md).
* Découvrez l’[interprétabilité : explications des modèles en machine learning automatisé (préversion)](how-to-machine-learning-interpretability-automl.md). 
* Découvrez comment effectuer l’apprentissage de plusieurs modèles avec AutoML dans l’article [Many Models Solution Accelerator](https://aka.ms/many-models) (Accélérateur de solution de nombreux modèles).
* Suivez le [tutoriel](tutorial-auto-train-models.md) pour voir un exemple de bout en bout de création d’expériences avec le machine learning automatisé.
