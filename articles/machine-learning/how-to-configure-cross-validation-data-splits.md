---
title: Configurer la validation croisée et les fractionnements de données dans les expériences de Machine Learning automatisé
titleSuffix: Azure Machine Learning
description: Guide pratique pour configurer la validation croisée et les fractionnements de jeux de données pour les expériences de Machine Learning automatisé
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, automl
ms.author: cesardl
author: CESARDELATORRE
ms.reviewer: nibaccam
ms.date: 06/16/2020
ms.openlocfilehash: c29c8ab31507c0ec904a7534e50ef6523e1aab96
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360103"
---
# <a name="configure-data-splits-and-cross-validation-in-automated-machine-learning"></a>Configurer les fractionnements de données et la validation croisée dans les opérations de Machine Learning automatisé

Dans cet article, vous allez découvrir les différentes options de configuration des fractionnements de données de formation/validation et de la validation croisée pour vos expériences Machine Learning automatisé et AutoML.

Dans Azure Machine Learning, quand vous utilisez AutoML pour générer plusieurs modèles de ML. Chaque exécution enfant doit valider le modèle associé en calculant les mesures de qualité pour ce modèle, telles que la précision ou la pondération AUC. Ces métriques sont calculées en comparant les prédictions effectuées avec chaque modèle avec des balises d’observations passées basées sur les données de validation. 

Les expériences AutoML effectuent la validation de modèle automatiquement. Les sections suivantes décrivent comment vous pouvez personnaliser davantage les paramètres de validation avec le [Kit de développement logiciel (SDK) Python pour Azure Machine Learning](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py). 

Pour une expérience avec peu, voire sans code, consultez la section [Créer vos expériences de machine learning automatisé dans Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md). 

> [!NOTE]
> Studio prend actuellement en charge les fractionnements de données de formation/validation et les options de validation croisée, mais il ne prend pas en charge la spécification de fichiers de données individuels pour votre jeu de validation. 

## <a name="prerequisites"></a>Prérequis

Pour cet article, vous avez besoin des éléments suivants :

* Un espace de travail Azure Machine Learning. Pour créer l’espace de travail, voir [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Une connaissance de base en matière de configuration d’une expérience de Machine Learning automatisé avec le Kit de développement logiciel (SDK) pour Azure Machine Learning. Suivez le [didacticiel](tutorial-auto-train-models.md) ou les [procédures](how-to-configure-auto-train.md) pour afficher les modèles de conception fondamentaux des expériences de Machine Learning automatisé.

* Une compréhension de la validation croisée et des fractionnements de données de formation et de validation en tant que concepts ML. Pour obtenir une explication plus poussée,

    * [À propos des jeux de formation, de validation et de test dans le Machine Learning](https://towardsdatascience.com/train-validation-and-test-sets-72cb40cba9e7)

    * [Comprendre l’utilisation de la validation croisée](https://towardsdatascience.com/understanding-cross-validation-419dbd47e9bd)

## <a name="default--data-splits-and-cross-validation"></a>Fractionnements de données et validation croisée par défaut

Utilisez l’objet [AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?preserve-view=true&view=azure-ml-py) pour définir vos paramètres d’expérimentation et de formation. Dans l’extrait de code suivant, Notez que seuls les paramètres requis sont définis, c’est-à-dire que les paramètres pour `n_cross_validation` ou `validation_ data` ne sont **pas** inclus.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'Class'
                            )
```

Si vous ne spécifiez pas explicitement un paramètre `validation_data` ou `n_cross_validation`, AutoML applique les techniques par défaut en fonction du nombre de lignes dans le jeu de données unique `training_data` fourni :

|Formation sur la taille des&nbsp;données&nbsp;| Technique de validation |
|---|-----|
|**Contient plus&nbsp;de&nbsp;20 000&nbsp;lignes**| Le fractionnement des données de formation/validation est appliqué. La valeur par défaut consiste à prendre 10 % du jeu de données d’apprentissage initial en tant que jeu de validation. Ce jeu de validation est ensuite utilisé pour le calcul des métriques.
|**Contient moins&nbsp;de&nbsp;20 000&nbsp;lignes**| L’approche de validation croisée est appliquée. Le nombre de plis par défaut dépend du nombre de lignes. <br> **Si le jeu de données est inférieur à 1 000 lignes** , 10 plis sont utilisés. <br> **S’il y a entre 1 000 et 20 000 lignes** , trois plis sont utilisés.

## <a name="provide-validation-data"></a>Fournir des données de validation

Dans ce cas, vous pouvez démarrer avec un seul fichier de données et le fractionner en jeux d’apprentissage et de validation, ou vous pouvez fournir un fichier de données distinct pour le jeu de validation. Dans les deux cas, le paramètre `validation_data` de votre objet `AutoMLConfig` assigne les données à utiliser comme jeu de validation. Ce paramètre n’accepte que les jeux de données sous la forme d’un [jeu données Azure Machine Learning](how-to-create-register-datasets.md) ou d’un cadre de données Pandas.   

L’exemple de code suivant définit explicitement la partie des données fournies dans `dataset` à utiliser pour l’apprentissage et la validation.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

training_data, validation_data = dataset.random_split(percentage=0.8, seed=1)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = training_data,
                             validation_data = validation_data,
                             label_column_name = 'Class'
                            )
```

## <a name="provide-validation-set-size"></a>Fournir la taille du jeu de données de validation

Dans ce cas, un seul jeu de données est fourni pour l’expérience. Autrement dit, le paramètre `validation_data` n’est **pas** spécifié, et le jeu de données fourni est assigné au paramètre `training_data`.  Dans votre objet `AutoMLConfig`, vous pouvez définir le paramètre `validation_size` pour qu’il contienne une partie des données d’apprentissage pour la validation. Cela signifie que le jeu de validation sera fractionné par AutoML à partir du `training_data` initial fourni. Cette valeur doit être comprise entre 0,0 et 1,0 non inclus (par exemple, 0,2 signifie que 20 % des données sont conservées pour les données de validation).

Prenez l'exemple du code suivant :

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             validation_size = 0.2,
                             label_column_name = 'Class'
                            )
```

## <a name="set-the-number-of-cross-validations"></a>Définir le nombre de validations croisées

Pour effectuer une validation croisée, incluez le paramètre `n_cross_validations` et affectez-lui une valeur. Ce paramètre définit le nombre de validations croisées à effectuer, en fonction du même nombre de plis.

Dans le code suivant, cinq plis pour la validation croisée sont définis. Par conséquent, il est question de cinq formations différentes. Chaque formation utilise 4/5 des données, et chaque validation utilise 1/5 des données avec un pli de données d'exclusion différent pour chaque formation.

Par conséquent, les mesures sont calculées avec la moyenne des 5 métriques de validation.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             n_cross_validations = 5
                             label_column_name = 'Class'
                            )
```

## <a name="specify-custom-cross-validation-data-folds"></a>Spécifier des plis de données de validation croisée personnalisés

Vous pouvez également fournir vos propres plis de données de validation croisée (CV). Cela est considéré comme un scénario plus avancé, car vous spécifiez les colonnes à fractionner et à utiliser pour la validation.  Incluez les colonnes de fractionnement CV personnalisées dans vos données d’apprentissage et spécifiez les colonnes en remplissant les noms de colonnes dans le paramètre `cv_split_column_names`. Chaque colonne représente une séparation de validation croisée et est remplie avec des valeurs entières 1 ou 0 où 1 indique que la ligne doit être utilisée pour la formation et 0 indique que la ligne doit être utilisée pour la validation.

L’extrait de code suivant contient des données de marketing bancaires avec deux colonnes de fractionnement CV « CV1 » et « CV2 ».

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_with_cv.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'y',
                             cv_split_column_names = ['cv1', 'cv2']
                            )
```

> [!NOTE]
> Pour utiliser `cv_split_column_names` avec `training_data` et `label_column_name`, veuillez mettre à niveau votre Kit de développement logiciel (SDK) Python pour Azure Machine Learning version 1.6.0 ou ultérieure. Pour les versions précédentes du Kit de développement logiciel (SDK), consultez l’utilisation de `cv_splits_indices`, mais notez qu’il est utilisé avec les entrées de jeu de données `X` et `y` uniquement. 

## <a name="next-steps"></a>Étapes suivantes

* [Empêcher les données déséquilibrées et le surajustement](concept-manage-ml-pitfalls.md).
* [Tutoriel : Utiliser le machine learning automatisé pour prédire le prix des courses de taxi : division des données](tutorial-auto-train-models.md#split-the-data-into-train-and-test-sets)
* Guide pratique pour [entraîner automatiquement un modèle de prévision de série chronologique](how-to-auto-train-forecast.md).