---
title: Caractérisation dans les expériences AutoML
titleSuffix: Azure Machine Learning
description: Découvrez les paramètres de caractérisation proposés par les offres Azure Machine Learning et la manière dont l’ingénierie de caractéristiques est prise en charge dans les expériences de ML automatisé.
author: nibaccam
ms.author: nibaccam
ms.reviewer: nibaccam
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, automl
ms.date: 05/28/2020
ms.openlocfilehash: 658db1604895515525e5a4826a43c0b21d9698b1
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359627"
---
# <a name="featurization-in-automated-machine-learning"></a>Caractérisation dans le Machine Learning automatisé



Dans ce guide, vous allez apprendre ce qui suit :

- Quels paramètres de caractérisation sont proposés par le Azure Machine Learning.
- Comment personnaliser ces fonctionnalités pour vos [expériences de machine learning automatisé](concept-automated-ml.md).

*L’ingénierie des caractéristiques* est le processus qui consiste à utiliser la connaissance du domaine des données pour créer des fonctionnalités qui aident les algorithmes de ML à améliorer leur apprentissage. Dans Azure Machine Learning, des techniques de mise à l’échelle et de normalisation des données sont appliquées pour faciliter l’ingénierie de caractéristiques. Généralement, ces techniques et l’ingénierie des caractéristiques sont appelées *caractérisation* dans les expériences de Machine Learning automatisé *(AutoML)* .

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous savez déjà comment configurer une expérience AutoML. Pour plus d'informations sur la configuration, consultez les articles suivants :

- Pour une première expérience basée sur le code : [Configurer des expériences de ML automatisé à l’aide du Kit de développement logiciel (SDK) Azure Machine Learning pour Python](how-to-configure-auto-train.md).
- Pour une expérience avec peu ou pas de code  : [Créer, examiner et déployer des modèles de machine learning automatisé avec Azure Machine Learning](how-to-use-automated-ml-for-ml-models.md).

## <a name="configure-featurization"></a>Configuration de la caractérisation

Dans chaque expérience de Machine Learning automatisée, des [techniques de mise à l’échelle automatique et de normalisation](#featurization) sont appliquées par défaut à vos données. Ces techniques sont des types de caractérisation qui aident *certains* algorithmes sensibles aux caractéristiques à des échelles différentes. Toutefois, vous pouvez également activer des caractérisations supplémentaires, telles que l’ *imputation des valeurs manquantes* , *l’encodage* et les *transformations*.

> [!NOTE]
> Les étapes de caractérisation du Machine Learning automatisé (normalisation des fonctionnalités, gestion des données manquantes, conversion de texte en valeurs numériques) font partie du modèle sous-jacent. Lorsque vous utilisez le modèle pour des prédictions, les étapes de caractérisation qui sont appliquées pendant la formation sont appliquées automatiquement à vos données d’entrée.

Pour les expériences configurées avec le Kit de développement logiciel (SDK) Python, vous pouvez activer ou désactiver le paramètre de caractérisation et spécifier les étapes de caractérisation qui doivent être utilisées pour votre expérience. Si vous utilisez le studio Azure Machine Learning, consultez [les étapes pour activer la caractérisation](how-to-use-automated-ml-for-ml-models.md#customize-featurization).

Le tableau suivant présente les paramètres acceptés pour `featurization` dans la [classe AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig) :

|Configuration de la caractérisation | Description|
------------- | ------------- |
|`"featurization": 'auto'`| Indique que, dans le cadre du prétraitement, des [étapes de garde-fous des données et de caractérisation](#featurization) doivent être effectuées automatiquement. Il s’agit du paramètre par défaut.|
|`"featurization": 'off'`| Indique que des étapes de caractérisation ne doivent pas être effectuées automatiquement.|
|`"featurization":`&nbsp;`'FeaturizationConfig'`| Indique que des étapes de caractérisation personnalisées doivent être utilisées. [Découvrez comment personnaliser la caractérisation](#customize-featurization).|

<a name="featurization"></a>

## <a name="automatic-featurization"></a>Caractérisation automatique

Le tableau suivant récapitule les techniques appliquées automatiquement à vos données. Ces techniques sont appliquées pour les expériences configurées à l’aide du kit de développement logiciel (SDK) ou du studio. Pour désactiver ce comportement, définissez `"featurization": 'off'` dans votre objet `AutoMLConfig`.

> [!NOTE]
> Si vous envisagez d’exporter vos modèles créés par ML automatisé vers un [modèle ONNX](concept-onnx.md), seules les options de caractérisation marquées d’un astérisque (*) sont prises en charge dans le format ONNX. Apprenez-en davantage sur la [conversion de modèles au format ONNX](concept-automated-ml.md#use-with-onnx).

|Étapes&nbsp;de caractérisation| Description |
| ------------- | ------------- |
|**Supprimer les caractéristiques de cardinalité élevée ou d’absence de variance** _ |Supprime ces fonctionnalités des ensembles de formation et de validation. S’applique aux fonctionnalités dont toutes les valeurs sont manquantes, avec la même valeur sur toutes les lignes, ou avec une cardinalité élevée (par exemple, les codes de hachage, les ID, ou les GUID).|
|_*Imputer des valeurs manquantes**_ |Pour les fonctionnalités numériques, remplacement par la moyenne des valeurs dans la colonne.<br/><br/>Pour les fonctionnalités catégorielles, remplacement par la valeur la plus fréquente.|
|_*Générer des caractéristiques supplémentaires**_ |Pour les caractéristiques de type date/heure : Année, Mois, Jour, Jour de la semaine, Jour de l’année, Trimestre, Semaine de l’année, Heure, Minute, Seconde.<br><br> _Pour les tâches de prévision,* ces fonctionnalités DateHeure supplémentaires sont créées : Année ISO, Semestre - semestre, Mois civil comme chaîne, Semaine, Jour de la semaine comme chaîne, Jour du trimestre, Jour de l’année, AM/PM (0 si l’heure se situe avant midi (12 pm), sinon 1), AM/PM comme chaîne, Heure de la journée (base de 12 heures)<br/><br/>Pour les caractéristiques de type texte : Fréquence des termes basée sur les unigrammes, les bigrammes et les trigrammes. En savoir plus sur [comment cela se passe avec BERT.](#bert-integration)|
|**Transformer et encoder** _|Les fonctionnalités numériques avec très peu de valeurs uniques sont transformées en fonctionnalités catégorielles.<br/><br/>L’encodage one-hot est utilisé pour les fonctionnalités catégoriques de faible cardinalité. L’encodage one-hot-hash est utilisé pour les fonctionnalités catégoriques de haute cardinalité.|
|_ *Incorporations de mots**|Caractériseur de texte convertissant les vecteurs de jetons de texte en vecteurs de phrase à l’aide d’un modèle déjà formé. Le vecteur d’incorporation de chaque mot d’un document est agrégé avec le reste pour produire un vecteur de fonctionnalité de document.|
|**Encodages cibles**|Pour les fonctionnalités catégorielles, cette étape mappe chaque catégorie avec une valeur cible moyenne pour les problèmes de régression, et à la probabilité de chaque classe pour les problèmes de classification. Une pondération basée sur la fréquence et une validation croisée par échantillons (« k-fold ») sont appliquées pour réduire l’ajustement du mappage et le bruit provoqué par les catégories de données éparses.|
|**Encodage de texte cible**|Pour l'entrée de texte, un modèle linéaire empilé avec « bag-of-words » est utilisé afin de générer la probabilité de chaque classe.|
|**WoE (Poids de la preuve)**|Calcule la valeur WoE en tant que mesure de corrélation des colonnes catégorielles vers la colonne cible. Le WoE est calculé en tant qu'enregistrement du ratio de probabilités à l'intérieur et à l'extérieur de la classe. Cette étape produit une colonne de fonctionnalités numériques par classe et évite d'avoir à imputer les valeurs manquantes et le traitement de valeur hors norme.|
|**Distance de cluster**|Effectue l’apprentissage d’un modèle de clustering k-moyennes sur toutes les colonnes numériques. Génère de nouvelles fonctionnalités *k* , une nouvelle fonctionnalité numérique par cluster, contenant la distance de chaque échantillon par rapport au centroïde de chaque cluster.|

## <a name="data-guardrails"></a>Garde-fous des données

*Les Garde-fous des données* vous aident à identifier les problèmes potentiels liés à vos données (par exemple, valeurs manquantes ou [déséquilibre de classe](concept-manage-ml-pitfalls.md#identify-models-with-imbalanced-data)). Ils vous aident également à prendre des mesures correctives pour améliorer les résultats.

Les Garde-fous des données sont appliqués :

- **Pour les expériences du Kit de développement logiciel (SDK)** : Lorsque les paramètres `"featurization": 'auto'` ou `validation=auto` sont spécifiés dans votre objet `AutoMLConfig`.
- **Pour les expériences de studio** : Quand l’option Caractérisation automatique est activée.

Vous pouvez consulter les Garde-fous des données pour votre expérience :

- En définissant `show_output=True` lorsque vous soumettez une expérience à l’aide du kit de développement logiciel (SDK).

- Dans le studio, sous l’onglet **Garde-fous des données** de votre exécution de ML automatisé.

### <a name="data-guardrail-states"></a>États des Garde-fous des données

Les garde-fous des données affichent l’un des trois états suivants :

|State| Description |
|----|---- |
|**Passed**| Aucun problème de données n’a été détecté et aucune action n’est requise de votre part. |
|**Done**| Des modifications ont été appliquées à vos données. Nous vous encourageons à passer en revue les actions correctives prises par le ML automatisé pour vérifier que les modifications s’alignent sur les résultats attendus. |
|**Alerted**| Un problème de données a été détecté mais n’a pas pu être résolu. Nous vous encourageons à examiner et à résoudre le problème.|

### <a name="supported-data-guardrails"></a>Garde-fous des données pris en charge

Le tableau suivant décrit les garde-fous des données pris en charge ainsi que les états associés que vous pouvez rencontrer lors l’envoi de votre expérience :

Garde-fou|Statut|Condition&nbsp;pour&nbsp;le déclencheur
---|---|---
**Imputation des valeurs de caractéristique manquantes** |Passed <br><br><br> Terminé| Aucune valeur de caractéristique manquante n’a été détectée dans vos données d’entraînement. Découvrez plus d’informations sur l’[imputation d’une valeur manquante](./how-to-use-automated-ml-for-ml-models.md#customize-featurization). <br><br> Des valeurs de caractéristique manquantes ont été détectées dans vos données de formation et imputées.
**Gestion des caractéristiques à cardinalité élevée** |Passed <br><br><br> Terminé| Aucune caractéristique à cardinalité élevée n’a été détectée après analyse de vos entrées. <br><br> Des caractéristiques à cardinalité élevée ont été détectées dans vos entrées et ont été gérées.
**Gestion du fractionnement de la validation** |Terminé| La configuration de la validation a été définie sur `'auto'` et les données de formation contenaient *moins de 20 000 lignes*. <br> Chaque itération du modèle entraîné a été validée à l’aide de la validation croisée. Découvrez-en plus sur les [données de validation](./how-to-configure-auto-train.md#training-validation-and-test-data). <br><br> La configuration de la validation a été définie sur `'auto'` et les données de formation contenaient *plus de 20 000 lignes*. <br> Les données d’entrée ont été divisées en un jeu de données d’entraînement et un jeu de données de validation pour la validation du modèle.
**Détection de l’équilibrage des classes** |Passed <br><br><br><br>Alerté <br><br><br>Terminé | Vos entrées ont été analysées et toutes les classes sont équilibrées dans vos données d’entraînement. Un jeu de données est considéré comme équilibré si chaque classe a une bonne représentation dans le jeu de données, telle que mesurée par le nombre et le ratio des échantillons. <br><br> Des classes déséquilibrées ont été détectées dans vos entrées. Pour corriger le biais du modèle, résolvez le problème d’équilibrage. Découvrez-en plus sur les [données déséquilibrées](./concept-manage-ml-pitfalls.md#identify-models-with-imbalanced-data).<br><br> Des classes déséquilibrées ont été détectées dans vos entrées et la logique de balayage a déterminé d’appliquer l’équilibrage.
**Détection des problèmes de mémoire** |Passed <br><br><br><br> Terminé |<br> Aucun problème potentiel d’insuffisance de mémoire n’a été détecté après analyse des valeurs {horizon, décalage, fenêtre dynamique} sélectionnées. Découvrez-en plus sur les [configurations de prévision](./how-to-auto-train-forecast.md#configuration-settings) de série chronologique. <br><br><br>Les valeurs {horizon, décalage, fenêtre dynamique} sélectionnées ont été analysées et peuvent entraîner une insuffisance de mémoire dans votre expérience. Les configurations de la fenêtre dynamique ou de décalage ont été désactivées.
**Détection de la fréquence** |Passed <br><br><br><br> Terminé |<br> La série chronologique a été analysée et tous les points de données sont alignés sur la fréquence détectée. <br> <br> La série chronologique a été analysée et les points de données qui ne sont pas alignés sur la fréquence détectée ont été détectés. Ces points de données ont été supprimés du jeu de données. Découvrez-en plus sur la [préparation des données pour la prévision de série chronologique](./how-to-auto-train-forecast.md#preparing-data).

## <a name="customize-featurization"></a>Personnaliser la caractérisation

Vous pouvez personnaliser vos paramètres de caractérisation pour vous assurer que les données et les caractéristiques utilisées pour l’apprentissage de votre modèle ML génèrent des prédictions pertinentes.

Pour personnaliser les caractérisations, spécifiez `"featurization": FeaturizationConfig` dans votre objet `AutoMLConfig`. Si vous utilisez le studio Azure Machine Learning pour votre expérience, consultez le [guide pratique](how-to-use-automated-ml-for-ml-models.md#customize-featurization). Pour personnaliser la caractérisation des types de tâches de prévision, reportez-vous à [la manière d’effectuer les prévisions](how-to-auto-train-forecast.md#customize-featurization).

Les personnalisations prises en charge sont notamment les suivantes :

|Personnalisation|Définition|
|--|--|
|**Mise à jour de l’objectif de la colonne**|Remplacer le type de caractéristique détecté automatiquement pour la colonne spécifiée.|
|**Mise à jour des paramètres du transformateur** |Mettre à jour les paramètres du transformateur spécifié. Prend actuellement en charge *Imputer* (moyen, le plus fréquent et médian) et *HashOneHotEncoder*.|
|**Suppression de colonnes** |Indique les colonnes à supprimer de la caractérisation.|
|**Transformateurs de blocs**| Indique les transformateurs de blocs à utiliser dans le processus de caractérisation.|

Créez l’objet `FeaturizationConfig` à l’aide d’appels d’API :

```python
featurization_config = FeaturizationConfig()
featurization_config.blocked_transformers = ['LabelEncoder']
featurization_config.drop_columns = ['aspiration', 'stroke']
featurization_config.add_column_purpose('engine-size', 'Numeric')
featurization_config.add_column_purpose('body-style', 'CategoricalHash')
#default strategy mean, add transformer param for for 3 columns
featurization_config.add_transformer_params('Imputer', ['engine-size'], {"strategy": "median"})
featurization_config.add_transformer_params('Imputer', ['city-mpg'], {"strategy": "median"})
featurization_config.add_transformer_params('Imputer', ['bore'], {"strategy": "most_frequent"})
featurization_config.add_transformer_params('HashOneHotEncoder', [], {"number_of_bits": 3})
```

## <a name="featurization-transparency"></a>Transparence de la caractérisation

La caractérisation est appliquée automatiquement à chaque modèle AutoML.  La caractérisation comprend une ingénierie de caractéristiques automatisée (quand `"featurization": 'auto'`) et une fonction de mise à l’échelle et de normalisation, qui ont un impact sur l’algorithme sélectionné et ses valeurs d’hyperparamètres. AutoML prend en charge différentes méthodes pour garantir la visibilité de ce qui a été appliqué à votre modèle.

Prenons l’exemple suivant :

+ Il existe quatre fonctionnalités d’entrée : A (numérique), B (numérique), C (numérique), D (DateTime).
+ La fonctionnalité numérique C est supprimée, car il s’agit d’une colonne d’ID avec toutes les valeurs uniques.
+ Il manque des valeurs dans les caractéristiques numériques A et B : une moyenne leur est donc affectée.
+ La fonctionnalité DateTime D a 11 fonctionnalités d’ingénierie différentes.

Pour accéder à ces informations, utilisez la sortie `fitted_model` de votre exécution d’expérience de ML automatisé.

```python
automl_config = AutoMLConfig(…)
automl_run = experiment.submit(automl_config …)
best_run, fitted_model = automl_run.get_output()
```
### <a name="automated-feature-engineering"></a>Ingénierie des fonctionnalités automatisées 
`get_engineered_feature_names()` retourne une liste de noms de fonctionnalités d’ingénierie.

  >[!Note]
  >Utilisez 'timeseriestransformer' pour task='forecasting', sinon utilisez 'datatransformer' pour la tâche 'regression' ou 'classification'.

  ```python
  fitted_model.named_steps['timeseriestransformer']. get_engineered_feature_names ()
  ```

Cette liste inclut tous les noms des fonctionnalités d’ingénierie. 

  ```
  ['A', 'B', 'A_WASNULL', 'B_WASNULL', 'year', 'half', 'quarter', 'month', 'day', 'hour', 'am_pm', 'hour12', 'wday', 'qday', 'week']
  ```

`get_featurization_summary()` obtient un résumé de caractérisation de toutes les fonctionnalités d’entrée.

  ```python
  fitted_model.named_steps['timeseriestransformer'].get_featurization_summary()
  ```

Output

  ```
  [{'RawFeatureName': 'A',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'B',
    'TypeDetected': 'Numeric',
    'Dropped': 'No',
    'EngineeredFeatureCount': 2,
    'Tranformations': ['MeanImputer', 'ImputationMarker']},
   {'RawFeatureName': 'C',
    'TypeDetected': 'Numeric',
    'Dropped': 'Yes',
    'EngineeredFeatureCount': 0,
    'Tranformations': []},
   {'RawFeatureName': 'D',
    'TypeDetected': 'DateTime',
    'Dropped': 'No',
    'EngineeredFeatureCount': 11,
    'Tranformations': ['DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime','DateTime']}]
  ```

   |Output|Définition|
   |----|--------|
   |RawFeatureName|Fonctionnalité/colonne d’entrée du jeu de données fournis.|
   |TypeDetected|Type de données détecté de la fonctionnalité d’entrée.|
   |Supprimé|Indique si la fonctionnalité d’entrée a été supprimée ou utilisée.|
   |EngineeringFeatureCount|Nombre de fonctionnalités générées par le biais de transformations d’ingénierie de la fonctionnalité automatisée.|
   |Transformations|Liste des transformations appliquées à des fonctionnalités d’entrée pour générer des fonctionnalités d’ingénierie.|

### <a name="scaling-and-normalization"></a>Mise à l’échelle et normalisation

Pour comprendre la mise à l’échelle/ma normalisation et l’algorithme sélectionné avec ses valeurs d’hyperparamètres, utilisez `fitted_model.steps`. 

L’exemple de sortie suivant est l’exécution de `fitted_model.steps` pour une série choisie :

```
[('RobustScaler', 
  RobustScaler(copy=True, 
  quantile_range=[10, 90], 
  with_centering=True, 
  with_scaling=True)), 

  ('LogisticRegression', 
  LogisticRegression(C=0.18420699693267145, class_weight='balanced', 
  dual=False, 
  fit_intercept=True, 
  intercept_scaling=1, 
  max_iter=100, 
  multi_class='multinomial', 
  n_jobs=1, penalty='l2', 
  random_state=None, 
  solver='newton-cg', 
  tol=0.0001, 
  verbose=0, 
  warm_start=False))
```

Pour obtenir plus de détails, utilisez cette fonction d’assistance : 

```python
from pprint import pprint

def print_model(model, prefix=""):
    for step in model.steps:
        print(prefix + step[0])
        if hasattr(step[1], 'estimators') and hasattr(step[1], 'weights'):
            pprint({'estimators': list(
                e[0] for e in step[1].estimators), 'weights': step[1].weights})
            print()
            for estimator in step[1].estimators:
                print_model(estimator[1], estimator[0] + ' - ')
        else:
            pprint(step[1].get_params())
            print()

print_model(model)
```

Cette fonction d’assistance retourne la sortie suivante pour une exécution particulière à l’aide de `LogisticRegression with RobustScalar` en tant qu’algorithme spécifique.

```
RobustScaler
{'copy': True,
'quantile_range': [10, 90],
'with_centering': True,
'with_scaling': True}

LogisticRegression
{'C': 0.18420699693267145,
'class_weight': 'balanced',
'dual': False,
'fit_intercept': True,
'intercept_scaling': 1,
'max_iter': 100,
'multi_class': 'multinomial',
'n_jobs': 1,
'penalty': 'l2',
'random_state': None,
'solver': 'newton-cg',
'tol': 0.0001,
'verbose': 0,
'warm_start': False}
```

### <a name="predict-class-probability"></a>Prédire une probabilité de classe

Les modèles produits à l’aide de Machine Learning automatisé ont tous des objets wrapper qui reflètent la fonctionnalité de leur classe d’origine open source. La plupart des objets wrapper de modèle de classification retournés par le Machine Learning automatisé implémentent la fonction `predict_proba()` qui accepte un échantillon de données de matrice avec pointillés ou de type tableau de vos fonctionnalités (valeurs X) et retourne un tableau à n dimensions de chaque échantillon et sa probabilité de classe respective.

En supposant que vous avez récupéré le meilleur modèle exécuté et ajusté en utilisant les appels ci-dessus, vous pouvez appeler `predict_proba()` directement à partir du modèle ajusté, en fournissant un exemple de `X_test` au format approprié en fonction du type de modèle.

```python
best_run, fitted_model = automl_run.get_output()
class_prob = fitted_model.predict_proba(X_test)
```

Si le modèle sous-jacent ne prend pas en charge la fonction `predict_proba()` ou si le format est incorrect, une exception spécifique de la classe de modèle est levée. Pour obtenir des exemples de la manière dont cette fonction est implémentée pour différents types de modèles, consultez les documents de référence [RandomForestClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html#sklearn.ensemble.RandomForestClassifier.predict_proba) et [XGBoost](https://xgboost.readthedocs.io/en/latest/python/python_api.html).

## <a name="bert-integration"></a>Intégration de BERT

[BERT](https://techcommunity.microsoft.com/t5/azure-ai/how-bert-is-integrated-into-azure-automated-machine-learning/ba-p/1194657) est utilisé dans la couche de caractérisation de ML automatisé. Dans cette couche, si une colonne contient du texte libre ou d’autres types de données tels que des horodateurs ou des nombres simples, nous les caractérisons en conséquence.

Pour BERT, le modèle est affiné et formé à l’aide des étiquettes fournies par l’utilisateur. À partir de là, les incorporations de documents sont générées en tant que fonctionnalités en aux côtés d’autres, comme les fonctionnalités de timestamp, de jour de la semaine. 


### <a name="bert-steps"></a>Étapes BERT

Pour appeler BERT, vous devez définir `enable_dnn: True` dans automl_settings et utiliser un calcul GPU (par exemple, `vm_size = "STANDARD_NC6"` ou un GPU supérieur). Si un calcul de l’UC est utilisé, au lieu de BERT, AutoML active le caractériseur BiLSTM DNN.

AutoML effectue les étapes suivantes pour BERT. 

1. **Prétraitement et création de jetons pour toutes les colonnes de texte**. Par exemple, le transformateur « StringCast » se trouve dans le résumé de caractérisation du modèle final. Vous trouverez un exemple de la façon de générer le résumé de caractérisation du modèle dans [ce notebook](https://towardsdatascience.com/automated-text-classification-using-machine-learning-3df4f4f9570b).

2. **Concaténez toutes les colonnes de texte en une seule colonne de texte** , d’où le `StringConcatTransformer` dans le modèle final. 

    Notre implémentation de BERT limite la longueur totale du texte d’un exemple de formation à 128 jetons. Cela signifie qu’en cas de concaténation, toutes les colonnes de texte doivent idéalement avoir une longueur maximale de 128 jetons. S’il y a plusieurs colonnes, chaque colonne doit être élaguée de telle sorte que cette condition soit remplie. Sinon, pour les colonnes concaténées de longueur > 128 jetons, la couche du générateur de jetons de BERT va tronquer cette entrée à 128 jetons.

3. **Dans le cadre du balayage de fonctionnalité, AutoML compare BERT avec la ligne de base (conteneur de fonctionnalités de mots) sur un échantillon de données.** Cette comparaison détermine si BERT donne des améliorations de précision. Si BERT s’exécute mieux que la ligne de base, AutoML utilise BERT pour la caractérisation de texte pour l’ensemble des données. Dans ce cas, vous verrez le `PretrainedTextDNNTransformer` dans le modèle final.

BERT s’exécute généralement plus longtemps que les autres caractériseurs. Pour de meilleures performances, nous vous conseillons d’utiliser « STANDARD_NC24r » ou « STANDARD_NC24rs_V3 » pour leurs capacités RDMA. 

AutoML distribuera l’entraînement BERT sur plusieurs nœuds s’ils sont disponibles (avec un maximum de huit nœuds). Pour ce faire, vous pouvez définir le paramètre `max_concurrent_iterations` sur une valeur supérieure à 1 dans votre objet `AutoMLConfig`. 
### <a name="supported-languages"></a>Langues prises en charge

AutoML prend actuellement en charge environ 100 langues et choisit le modèle BERT approprié en fonction de la langue du jeu de données. Pour les données en allemand, nous utilisons le modèle BERT allemand. Pour l’anglais, nous utilisons le modèle BERT anglais. Pour toutes les autres langues, nous utilisons le modèle BERT multilingue.

Dans le code suivant, le modèle BERT allemand est déclenché, étant donné que la langue du jeu de données est définie sur `deu` le code de langue à trois lettres pour l’allemand conformément à la [classification ISO](https://iso639-3.sil.org/code/deu) :

```python
from azureml.automl.core.featurization import FeaturizationConfig

featurization_config = FeaturizationConfig(dataset_language='deu')

automl_settings = {
    "experiment_timeout_minutes": 120,
    "primary_metric": 'accuracy', 
# All other settings you want to use 
    "featurization": featurization_config,
    
  "enable_dnn": True, # This enables BERT DNN featurizer
    "enable_voting_ensemble": False,
    "enable_stack_ensemble": False
}
```

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment configurer vos expériences de ML automatisé :

    * Pour une première expérience basée sur le code : [Configurer des expériences de ML automatisé à l’aide du Kit de développement logiciel (SDK) Azure Machine Learning](how-to-configure-auto-train.md).
    * Pour une expérience avec peu ou pas de code  : [Créez vos expériences de Machine Learning automatisé dans le studio Azure Machine Learning](how-to-use-automated-ml-for-ml-models.md).

* Découvrez plus d’informations sur [comment et où déployer un modèle](how-to-deploy-and-where.md).

* Découvrez plus d’informations sur [comment entraîner un modèle de régression à l’aide du machine learning automatisé](tutorial-auto-train-models.md) ou [comment entraîner avec le machine learning automatisé sur une ressource distante](how-to-auto-train-remote.md).
