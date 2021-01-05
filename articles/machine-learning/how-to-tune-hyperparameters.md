---
title: Optimiser les hyperparamètres pour votre modèle
titleSuffix: Azure Machine Learning
description: Ajustez avec efficacité les hyperparamètres de vos modèles Deep Learning et Machine Learning avec Azure Machine Learning.
ms.author: swatig
author: swatig007
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 03/30/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q1
ms.openlocfilehash: a1a0b89dee5a6d57e6f317c5f6c8b61ffeda3c33
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029071"
---
# <a name="tune-hyperparameters-for-your-model-with-azure-machine-learning"></a>Optimiser les hyperparamètres de votre modèle avec Azure Machine Learning


Automatisez une optimisation efficace des hyperparamètres à l’aide du [package HyperDrive](/python/api/azureml-train-core/azureml.train.hyperdrive?preserve-view=true&view=azure-ml-py) Azure Machine Learning. Découvrez comment effectuer les étapes nécessaires pour régler les hyperparamètres à l’aide du SDK [Azure Machine Learning](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py) :

1. Définir l’espace de recherche de paramètres
1. Spécifier une métrique principale à optimiser  
1. Spécifier la stratégie d’arrêt anticipé pour les exécutions à faibles performances
1. Allouer des ressources
1. Lancer une expérience avec la configuration définie
1. Visualiser les exécutions d’entraînement
1. Sélectionner la meilleure configuration pour votre modèle

## <a name="what-are-hyperparameters"></a>Présentation des hyperparamètres

Les **hyperparamètres** sont des paramètres réglables qui vous permettent de contrôler le processus d’entraînement du modèle. Par exemple, avec des réseaux neuronaux, vous déterminez le nombre de couches masquées et le nombre de nœuds dans chaque couche. Les performances du modèle dépendent fortement des hyperparamètres.

 L’**optimisation des hyperparamètres** est le processus de recherche de la configuration des hyperparamètres qui produit les meilleures performances. Le processus est généralement manuel et gourmand en ressources informatiques.

Azure Machine Learning vous permet d’automatiser l’optimisation des hyperparamètres et d’exécuter des expérimentations parallèles pour optimiser efficacement les hyperparamètres.


## <a name="define-the-search-space"></a>Définir l’espace de recherche

Optimisez les hyperparamètres en explorant la plage de valeurs définie pour chaque hyperparamètre.

Les hyperparamètres peuvent être discrets ou continus et ont une distribution des valeurs décrite par une [expression de paramètre](/python/api/azureml-train-core/azureml.train.hyperdrive.parameter_expressions?preserve-view=true&view=azure-ml-py).

### <a name="discrete-hyperparameters"></a>Hyperparamètres discrets 

Les hyperparamètres discrets sont spécifiés en tant que `choice` parmi des valeurs discrètes. Voici à quoi `choice` peut correspondre :

* une ou plusieurs valeurs séparées par une virgule
* un objet `range`
* un objet `list` arbitraire


```Python
    {
        "batch_size": choice(16, 32, 64, 128)
        "number_of_hidden_layers": choice(range(1,5))
    }
```

Dans ce cas, `batch_size` prend l’une des valeurs [16, 32, 64, 128] et `number_of_hidden_layers` prend l’une des valeurs [1, 2, 3, 4].

Les hyperparamètres discrets avancés suivants peuvent également être spécifiés en utilisant une distribution.

* `quniform(low, high, q)` : retourne une valeur comme round(uniform(low, high) / q) * q
* `qloguniform(low, high, q)` : retourne une valeur comme round(exp(uniform(low, high)) / q) * q
* `qnormal(mu, sigma, q)` : retourne une valeur comme round(normal(mu, sigma) / q) * q
* `qlognormal(mu, sigma, q)` : retourne une valeur comme round(exp(normal(mu, sigma)) / q) * q

### <a name="continuous-hyperparameters"></a>Hyperparamètres continus 

Les hyperparamètres continus sont spécifiés comme une distribution sur une plage continue de valeurs :

* `uniform(low, high)` : retourne une valeur uniformément distribuée entre low (valeur basse) et high (valeur haute)
* `loguniform(low, high)` : retourne une valeur calculée en fonction de exp(uniform(low, high)), de sorte que le logarithme de la valeur de retour est uniformément distribué
* `normal(mu, sigma)` : retourne une valeur réelle qui est normalement distribuée avec la moyenne mu et l’écart type sigma
* `lognormal(mu, sigma)` : retourne une valeur calculée en fonction de exp(normal(mu, sigma)), de sorte que le logarithme de la valeur de retour est normalement distribué

Voici un exemple de définition d’espace de paramètres :

```Python
    {    
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1)
    }
```

Ce code définit un espace de recherche avec deux paramètres : `learning_rate` et `keep_probability`. `learning_rate` présente une distribution normale avec une valeur moyenne de 10 et un écart type de 3. `keep_probability` présente une distribution uniforme avec une valeur minimale de 0,05 et une valeur maximale de 0,1.

### <a name="sampling-the-hyperparameter-space"></a>Échantillonnage de l’espace des hyperparamètres

Spécifiez la méthode d’échantillonnage des paramètres à utiliser dans l’espace des hyperparamètres. Azure Machine Learning prend en charge les méthodes suivantes :

* Échantillonnage aléatoire
* Échantillonnage par grille
* Échantillonnage bayésien

#### <a name="random-sampling"></a>Échantillonnage aléatoire

L’[échantillonnage aléatoire](/python/api/azureml-train-core/azureml.train.hyperdrive.randomparametersampling?preserve-view=true&view=azure-ml-py) prend en charge les hyperparamètres discrets et continus. Il prend en charge l’arrêt précoce des exécutions à faibles performances. Certains utilisateurs effectuent une recherche initiale avec l’échantillonnage aléatoire, puis affinent l’espace de recherche pour améliorer les résultats.

Dans l’échantillonnage aléatoire, les valeurs des hyperparamètres sont sélectionnées de façon aléatoire à partir de l’espace de recherche défini. 

```Python
from azureml.train.hyperdrive import RandomParameterSampling
from azureml.train.hyperdrive import normal, uniform, choice
param_sampling = RandomParameterSampling( {
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```

#### <a name="grid-sampling"></a>Échantillonnage par grille

L’[échantillonnage de grille](/python/api/azureml-train-core/azureml.train.hyperdrive.gridparametersampling?preserve-view=true&view=azure-ml-py) prend en charge les hyperparamètres discrets. Utilisez l’échantillonnage de grille si vous avez un budget pour effectuer une recherche exhaustive sur l’espace de recherche. Prend en charge l’arrêt anticipé des exécutions à faibles performances.

Effectue une recherche de grille simple sur toutes les valeurs possibles. L’échantillonnage de grille ne peut être utilisé qu’avec des hyperparamètres `choice`. Par exemple, l’espace suivant compte six échantillons :

```Python
from azureml.train.hyperdrive import GridParameterSampling
from azureml.train.hyperdrive import choice
param_sampling = GridParameterSampling( {
        "num_hidden_layers": choice(1, 2, 3),
        "batch_size": choice(16, 32)
    }
)
```

#### <a name="bayesian-sampling"></a>Échantillonnage bayésien

L’[échantillonnage bayésien](/python/api/azureml-train-core/azureml.train.hyperdrive.bayesianparametersampling?preserve-view=true&view=azure-ml-py) est basé sur l’algorithme d’optimisation bayésienne. Il choisit des exemples en fonction de la façon dont les exemples précédents ont été exécutés, afin que les nouveaux exemples améliorent la métrique principale.

L’échantillonnage bayésien est recommandé si vous disposez d’un budget suffisant pour explorer l’espace hyperparamétrique. Pour de meilleurs résultats, nous recommandons un nombre maximal d’exécutions supérieur ou égal à 20 fois le nombre d’hyperparamètres configurés. 

Le nombre d’exécutions simultanées a un impact sur l’efficacité du processus d’optimisation. Un nombre inférieur d’exécutions simultanées peut déboucher sur une meilleure convergence d’échantillonnage, car le plus faible degré de parallélisme accroît le nombre d’exécutions qui bénéficient des exécutions ayant abouti précédemment.

L’échantillonnage bayésien prend en charge seulement les distributions `choice`, `uniform` et `quniform` sur l’espace de recherche.

```Python
from azureml.train.hyperdrive import BayesianParameterSampling
from azureml.train.hyperdrive import uniform, choice
param_sampling = BayesianParameterSampling( {
        "learning_rate": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```



## <a name="specify-primary-metric"></a><a name="specify-primary-metric-to-optimize"></a> Spécifier la métrique principale

Spécifiez la [métrique principale](/python/api/azureml-train-core/azureml.train.hyperdrive.primarymetricgoal?preserve-view=true&view=azure-ml-py) que l’optimisation des hyperparamètres doit optimiser. Chaque exécution d’entraînement est évaluée par rapport à la métrique principale. La stratégie d’arrêt anticipé utilise la métrique principale pour identifier les exécutions aux faibles performances.

Spécifiez les attributs suivants pour votre métrique principale :

* `primary_metric_name` : le nom de la métrique principale doit correspondre exactement au nom de la métrique journalisée par le script d’entraînement
* `primary_metric_goal`: il peut s’agir de `PrimaryMetricGoal.MAXIMIZE` ou de `PrimaryMetricGoal.MINIMIZE`. Il détermine si la métrique principale est maximisée ou minimisée lors de l’évaluation des exécutions. 

```Python
primary_metric_name="accuracy",
primary_metric_goal=PrimaryMetricGoal.MAXIMIZE
```

Cet exemple maximise la « précision ».

### <a name="log-metrics-for-hyperparameter-tuning"></a><a name="log-metrics-for-hyperparameter-tuning"></a>Journaliser des métriques pour l’optimisation des hyperparamètres

Le script d’entraînement pour votre modèle **doit** journaliser la métrique principale lors de l’entraînement du modèle, afin que HyperDrive puisse y accéder pour l’optimisation des hyperparamètres.

Journalisez la métrique principale dans votre script d’entraînement avec l’extrait de code suivant :

```Python
from azureml.core.run import Run
run_logger = Run.get_context()
run_logger.log("accuracy", float(val_accuracy))
```

Le script d’entraînement calcule la valeur de `val_accuracy` et la journalise comme valeur de précision de la métrique principale. Chaque fois que la métrique est journalisée, elle est reçue par le service d’optimisation des hyperparamètres. C’est à vous de déterminer la fréquence des rapports.

Pour plus d’informations sur les valeurs de journalisation dans les exécutions d’entraînement du modèle, consultez [Activer la journalisation dans les exécutions d’entraînement Azure ML](how-to-track-experiments.md).

## <a name="specify-early-termination-policy"></a><a name="early-termination"></a> Spécifier une stratégie d’arrêt anticipé

Une stratégie d’arrêt anticipé permet d’arrêter automatiquement les exécutions peu performantes. Un arrêt anticipé améliore l’efficacité du calcul.

Vous pouvez configurer les paramètres suivants pour contrôler à quel moment une stratégie s’applique :

* `evaluation_interval` : fréquence d’application de la stratégie. Chaque journalisation de la métrique principale par le script d’entraînement compte pour un intervalle. Une valeur de 1 pour `evaluation_interval` applique la stratégie chaque fois que le script d’entraînement signale la métrique principale. Une valeur de 2 pour `evaluation_interval` applique la stratégie à chaque fois. La valeur par défaut de `evaluation_interval` est 1 si ce paramètre n’est pas spécifié.
* `delay_evaluation` : retarde la première évaluation de la stratégie pour un nombre d’intervalles spécifié. Il s’agit d’un paramètre facultatif qui évite un arrêt prématuré des exécutions d’entraînement en permettant à toutes les configurations de s’exécuter pour un nombre minimal d’intervalles. S’il est spécifié, la stratégie s’applique à chaque multiple de evaluation_interval qui est supérieur ou égal à delay_evaluation.

Azure Machine Learning prend en charge les stratégies d’arrêt anticipé suivantes :
* [Stratégie Bandit](#bandit-policy)
* [Stratégie d’arrêt médiane](#median-stopping-policy)
* [Stratégie de sélection de troncation](#truncation-selection-policy)
* [Stratégie sans arrêt](#no-termination-policy-default)


### <a name="bandit-policy"></a>Stratégie Bandit

La [stratégie Bandit](/python/api/azureml-train-core/azureml.train.hyperdrive.banditpolicy?preserve-view=true&view=azure-ml-py#&preserve-view=truedefinition) est une stratégie d’arrêt basée sur le facteur de marge/marge totale et l’intervalle d’évaluation. Cette stratégie arrête de façon anticipée les exécutions quand la métrique principale se trouve en dehors du facteur de marge/marge totale spécifié par rapport à l’exécution la plus performante.

> [!NOTE]
> L’échantillonnage bayésien ne prend pas en charge l’arrêt anticipé. Lorsque vous utilisez l’échantillonnage bayésien, définissez `early_termination_policy = None`.

Spécifiez les paramètres de configuration suivants :

* `slack_factor` ou `slack_amount` : marge autorisée par rapport à l’exécution d’entraînement la plus performante. `slack_factor` spécifie la marge autorisée sous forme de ratio. `slack_amount` spécifie la marge autorisée en valeur absolue, au lieu d’un ratio.

    Par exemple, considérez une stratégie Bandit appliquée avec un intervalle de 10. Supposez que l’exécution la plus performante à l’intervalle 10 a signalé une métrique principale de 0,8 avec l’objectif de maximiser la métrique principale. Si la stratégie spécifie avec un `slack_factor` de 0,2, les exécutions d’entraînement, dont la meilleure mesure à l’intervalle 10 est inférieure à 0,66 (0,8/(1+`slack_factor`)) seront arrêtées.
* `evaluation_interval` : (facultatif) fréquence d’application de la stratégie
* `delay_evaluation` : (facultatif) retarde la première évaluation de la stratégie pour un nombre d’intervalles spécifié


```Python
from azureml.train.hyperdrive import BanditPolicy
early_termination_policy = BanditPolicy(slack_factor = 0.1, evaluation_interval=1, delay_evaluation=5)
```

Dans cet exemple, la stratégie d’arrêt anticipé est appliquée à chaque intervalle quand des métriques sont signalées, en commençant à l’intervalle d’évaluation 5. Toute exécution dont la meilleure métrique est inférieure à (1/(1+0,1) ou 91 % de la meilleure exécution sera arrêtée.

### <a name="median-stopping-policy"></a>Stratégie d’arrêt médiane

La [stratégie d’arrêt médiane](/python/api/azureml-train-core/azureml.train.hyperdrive.medianstoppingpolicy?preserve-view=true&view=azure-ml-py) est une stratégie d’arrêt anticipé basée sur les moyennes mobiles des métriques principales rapportées par les exécutions. Cette stratégie calcule les moyennes mobiles pour toutes les exécutions d’entraînement et arrête les exécutions dont les valeurs de la métrique principale sont moins bonnes que la médiane des moyennes.

Cette stratégie prend les paramètres de configuration suivants :
* `evaluation_interval` : fréquence d’application de la stratégie (paramètre facultatif).
* `delay_evaluation` : retarde la première évaluation de la stratégie pour un nombre d’intervalles spécifié (paramètre facultatif).


```Python
from azureml.train.hyperdrive import MedianStoppingPolicy
early_termination_policy = MedianStoppingPolicy(evaluation_interval=1, delay_evaluation=5)
```

Dans cet exemple, la stratégie d’arrêt anticipé est appliquée à chaque intervalle, en commençant à l’intervalle d’évaluation 5. Une exécution est interrompue à l’intervalle 5 si sa meilleure métrique principale est moins bonne que la médiane des moyennes mobiles sur les intervalles 1 à 5 pour toutes les exécutions d’entraînement.

### <a name="truncation-selection-policy"></a>Stratégie de sélection de troncation

La [stratégie de sélection de troncation](/python/api/azureml-train-core/azureml.train.hyperdrive.truncationselectionpolicy?preserve-view=true&view=azure-ml-py) annule un pourcentage des exécutions les moins performantes à chaque intervalle d’évaluation. Les exécutions sont comparées à l’aide de la métrique principale. 

Cette stratégie prend les paramètres de configuration suivants :

* `truncation_percentage` : le pourcentage des exécutions avec les performances les moins bonnes à arrêter à chaque intervalle d’évaluation. Nombre entier compris entre 1 et 99.
* `evaluation_interval` : (facultatif) fréquence d’application de la stratégie
* `delay_evaluation` : (facultatif) retarde la première évaluation de la stratégie pour un nombre d’intervalles spécifié


```Python
from azureml.train.hyperdrive import TruncationSelectionPolicy
early_termination_policy = TruncationSelectionPolicy(evaluation_interval=1, truncation_percentage=20, delay_evaluation=5)
```

Dans cet exemple, la stratégie d’arrêt anticipé est appliquée à chaque intervalle, en commençant à l’intervalle d’évaluation 5. Une exécution est arrêtée à l’intervalle 5 si sa performance à cet intervalle se trouve dans les 20 % des performances les moins bonnes de toutes les exécutions à l’intervalle 5.

### <a name="no-termination-policy-default"></a>Aucune stratégie de fin (par défaut)

Si aucune stratégie n’est spécifiée, le service d’optimisation des hyperparamètres permet à toutes les exécutions d’entraînement d’arriver à leur terme.

```Python
policy=None
```

### <a name="picking-an-early-termination-policy"></a>Sélection d’une stratégie d’arrêt anticipé

* Si vous cherchez une stratégie classique qui permet de réaliser des économies, sans arrêter les tâches prometteuses, vous pouvez utiliser une stratégie d’arrêt à la médiane avec un `evaluation_interval` de 1 et un `delay_evaluation` de 5. Il s’agit de valeurs prudentes, qui peuvent fournir approximativement 25 à 35 % d’économies sans perte sur la métrique principale (d’après nos évaluations).
* Pour des économies plus importantes, utilisez une stratégie Bandit avec une plus petite marge autorisée ou une stratégie de sélection de troncation avec un pourcentage de troncation plus élevé.

## <a name="allocate-resources"></a>Allouer des ressources

Contrôlez le budget de vos ressources en spécifiant le nombre maximal d’exécutions d’entraînement.

* `max_total_runs` : nombre maximal d’exécutions d’entraînement. Doit être un entier compris entre 1 et 1000.
* `max_duration_minutes` : (facultatif) durée maximale en minutes de l’expérience d’optimisation des hyperparamètres. S’exécute après l’annulation de cette durée.

>[!NOTE] 
>Si `max_total_runs` et `max_duration_minutes` sont tous deux spécifiés, l’expérience d’optimisation des hyperparamètres s’arrête dès que le premier de ces deux seuils est atteint.

Par ailleurs, spécifiez le nombre maximal d’exécutions d’entraînement qui peuvent s’exécuter simultanément pendant votre recherche d’optimisation des hyperparamètres.

* `max_concurrent_runs` : (facultatif) nombre maximal d’exécutions qui peuvent s’exécuter simultanément. En l’absence de spécification, toutes les exécutions s’exécutent en parallèle. En cas de spécification, doit être un entier compris entre 1 et 100.

>[!NOTE] 
>Le nombre d’exécutions simultanées est limité par les ressources disponibles dans la cible de calcul spécifiée. Vérifiez que la cible de calcul dispose des ressources nécessaires à l’accès concurrentiel souhaité.

```Python
max_total_runs=20,
max_concurrent_runs=4
```

Ce code configure l’expérience d’optimisation des hyperparamètres pour utiliser un maximum de 20 exécutions au total, en exécutant 4 configurations à la fois.

## <a name="configure-experiment"></a>Configurer une expérience

Pour [configurer votre optimisation des hyperparamètres](/python/api/azureml-train-core/azureml.train.hyperdrive.hyperdriverunconfig?preserve-view=true&view=azure-ml-py), fournissez les informations suivantes :
* Espace de recherche des hyperparamètres défini
* Votre stratégie d’arrêt anticipé
* Métrique principale
* Paramètres d’allocation des ressources
* ScriptRunConfig `src`

ScriptRunConfig est le script d’entraînement qui s’exécute avec les hyperparamètres échantillonnés. Il définit les ressources par travail (un ou plusieurs nœuds) et la cible de calcul à utiliser.

> [!NOTE]
>La cible de calcul spécifiée dans `src` doit avoir suffisamment de ressources pour répondre à votre niveau d’accès concurrentiel. Pour plus d’informations sur ScriptRunConfig, consultez [Configurer des exécutions d’entraînement](how-to-set-up-training-targets.md).

Configurez votre expérience d’optimisation des hyperparamètres :

```Python
from azureml.train.hyperdrive import HyperDriveConfig
hd_config = HyperDriveConfig(run_config=src,
                             hyperparameter_sampling=param_sampling,
                             policy=early_termination_policy,
                             primary_metric_name="accuracy",
                             primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                             max_total_runs=100,
                             max_concurrent_runs=4)
```

## <a name="submit-experiment"></a>Soumettre l’expérience

Une fois que vous avez défini votre configuration d’optimisation des hyperparamètres, [soumettez une expérience](/python/api/azureml-core/azureml.core.experiment%28class%29?preserve-view=true&view=azure-ml-py#&preserve-view=truesubmit-config--tags-none----kwargs-) :

```Python
from azureml.core.experiment import Experiment
experiment = Experiment(workspace, experiment_name)
hyperdrive_run = experiment.submit(hd_config)
```

## <a name="warm-start-your-hyperparameter-tuning-experiment-optional"></a>Démarrage à chaud de votre expérience de paramétrage d’hyperparamètres (facultatif)

La recherche des meilleures valeurs d’hyperparamètres pour votre modèle peut être un processus itératif. Vous pouvez réutiliser les connaissances des cinq précédentes exécutions pour accélérer l’optimisation des hyperparamètres.


Le démarrage à chaud est géré différemment en fonction de la méthode d’échantillonnage :
- **Échantillonnage bayésien** : Les essais de l’exécution précédente sont utilisés comme connaissance préalable pour choisir de nouveaux exemples et pour améliorer la métrique principale.
- **Échantillonnage aléatoire** ou **échantillonnage de grille** :  L’arrêt anticipé utilise les connaissances des précédentes exécutions pour déterminer les performances d’exécution médiocres. 

Spécifiez la liste des exécutions parentes que vous souhaitez démarrer à chaud.

```Python
from azureml.train.hyperdrive import HyperDriveRun

warmstart_parent_1 = HyperDriveRun(experiment, "warmstart_parent_run_ID_1")
warmstart_parent_2 = HyperDriveRun(experiment, "warmstart_parent_run_ID_2")
warmstart_parents_to_resume_from = [warmstart_parent_1, warmstart_parent_2]
```

Si une expérience d’optimisation des hyperparamètres est annulée, vous pouvez reprendre les exécutions d’entraînement à partir du dernier point de contrôle. Toutefois, votre script d’entraînement doit gérer la logique de point de contrôle.

L’exécution d’entraînement doit utiliser la même configuration d’hyperparamètre et monter les dossiers Outputs. Le script d’entraînement doit accepter l’argument `resume-from`, qui contient les fichiers de point de contrôle ou de modèle à partir desquels reprendre l’exécution de l’entraînement. Vous pouvez reprendre des exécutions de formation individuelles à l’aide de l’extrait de code suivant :

```Python
from azureml.core.run import Run

resume_child_run_1 = Run(experiment, "resume_child_run_ID_1")
resume_child_run_2 = Run(experiment, "resume_child_run_ID_2")
child_runs_to_resume = [resume_child_run_1, resume_child_run_2]
```

Vous pouvez configurer votre expérience de paramétrage d’hyperparamètres pour démarrer à chaud à partir d’une expérience précédente ou reprendre des exécutions individuelles à l’aide des paramètres facultatifs `resume_from` et `resume_child_runs` dans la configuration :

```Python
from azureml.train.hyperdrive import HyperDriveConfig

hd_config = HyperDriveConfig(run_config=src,
                             hyperparameter_sampling=param_sampling,
                             policy=early_termination_policy,
                             resume_from=warmstart_parents_to_resume_from,
                             resume_child_runs=child_runs_to_resume,
                             primary_metric_name="accuracy",
                             primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                             max_total_runs=100,
                             max_concurrent_runs=4)
```

## <a name="visualize-experiment"></a>Visualiser l’expérience

Utilisez le [widget Notebook](/python/api/azureml-widgets/azureml.widgets.rundetails?preserve-view=true&view=azure-ml-py) pour visualiser la progression de vos exécutions d’entraînement. L’extrait de code suivant vous permet de visualiser toutes vos exécutions d’optimisation des hyperparamètres dans un notebook Jupyter :

```Python
from azureml.widgets import RunDetails
RunDetails(hyperdrive_run).show()
```

Ce code présente un tableau avec des détails sur les exécutions d’entraînement pour chacune des configurations d’hyperparamètres.

![Tableau des optimisations des hyperparamètres](./media/how-to-tune-hyperparameters/HyperparameterTuningTable.png)

Vous pouvez également visualiser les performances de chacune des exécutions au fil de l’entraînement. 

![Tracé de l’optimisation des hyperparamètres](./media/how-to-tune-hyperparameters/HyperparameterTuningPlot.png)

Vous pouvez identifier visuellement la corrélation entre les performances et les valeurs des hyperparamètres individuels avec un tracé de coordonnées parallèles. 

[![Coordonnées parallèles de l'optimisation des hyperparamètres](./media/how-to-tune-hyperparameters/HyperparameterTuningParallelCoordinates.png)](media/how-to-tune-hyperparameters/hyperparameter-tuning-parallel-coordinates-expanded.png)

Vous pouvez aussi visualiser toutes vos exécutions d’optimisation des hyperparamètres dans le portail web Azure. Pour plus d’informations sur la visualisation d’une expérience sur le portail, consultez le [guide pratique pour suivre les expériences](how-to-monitor-view-training-logs.md#view-the-experiment-in-the-web-portal).

## <a name="find-the-best-model"></a>Trouver le meilleur modèle

Une fois que toutes les exécutions d’optimisation des hyperparamètres ont abouti, identifiez la configuration la plus performante et les valeurs des hyperparamètres :

```Python
best_run = hyperdrive_run.get_best_run_by_primary_metric()
best_run_metrics = best_run.get_metrics()
parameter_values = best_run.get_details()['runDefinition']['Arguments']

print('Best Run Id: ', best_run.id)
print('\n Accuracy:', best_run_metrics['accuracy'])
print('\n learning rate:',parameter_values[3])
print('\n keep probability:',parameter_values[5])
print('\n batch size:',parameter_values[7])
```

## <a name="sample-notebook"></a>Exemple de notebook
Reportez-vous aux notebooks train-hyperparameter-* dans ce dossier :
* [how-to-use-azureml/ml-frameworks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Étapes suivantes
* [Faire le suivi d’une expérience](how-to-track-experiments.md)
* [Déployer un modèle entraîné](how-to-deploy-and-where.md)