---
title: 'Tutoriel : Former votre premier modèle Machine Learning — Python'
titleSuffix: Azure Machine Learning
description: Dans cette troisième partie de notre série consacrée à la prise en main d’Azure Machine Learning, vous découvrez comment entraîner un modèle Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/15/2020
ms.custom: devx-track-python
ms.openlocfilehash: df511e79b73256833ec54c5906bb6acbc852bc46
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739618"
---
# <a name="tutorial-train-your-first-machine-learning-model-part-3-of-4"></a>Tutoriel : Entraîner votre premier modèle Machine Learning (partie 3 sur 4)

Ce tutoriel vous montre comment former un modèle Machine Learning dans Azure Machine Learning.

Ce tutoriel constitue la *partie 3 d’une série de quatre tutoriels* où vous découvrez les principes fondamentaux d’Azure Machine Learning et comment effectuer des tâches de machine learning basées sur des travaux dans Azure. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 1 : Configurer votre ordinateur local](tutorial-1st-experiment-sdk-setup-local.md) et [Partie 2 : Exécuter « Hello world! »](tutorial-1st-experiment-hello-world.md) de la série.

À présent, vous allez soumettre un script qui forme un modèle Machine Learning. Cet exemple vous permettra de comprendre comment Azure Machine Learning facilite la cohérence entre les procédures de débogage local et d’exécutions distantes.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créez un script d’entraînement.
> * Utiliser Conda pour définir un environnement Azure Machine Learning.
> * Créer un script de contrôle.
> * Comprendre les classes Azure Machine Learning (`Environment`, `Run`, `Metrics`).
> * Soumettre et exécuter votre script de formation.
> * Afficher le résultat de votre code dans le cloud.
> * Journaliser des métriques dans Azure Machine Learning.
> * Afficher vos métriques dans le cloud.

## <a name="prerequisites"></a>Prérequis

* Avoir effectué la [partie 2](tutorial-1st-experiment-hello-world.md) de la série.
* Avoir des connaissances préliminaires du langage Python et des workflows Machine Learning.
* Environnement de développement local, comme Visual Studio Code, Jupyter et PyCharm.
* Python (version 3.5 à 3.7).

## <a name="create-training-scripts"></a>Créer des scripts de formation

Tout d’abord, vous définissez l’architecture du réseau neuronal dans un fichier `model.py`. Tout votre code d’entraînement sera placé dans le sous-répertoire `src`, y compris `model.py`.

Le code suivant est tiré de [cet exemple d’introduction](https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html) de PyTorch. Notez que les concepts de Azure Machine Learning s’appliquent à tout code Machine Learning, pas seulement à PyTorch.

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/src/model.py":::

Ensuite, vous définissez le script de formation. Ce script télécharge le jeu de données CIFAR10 en utilisant les API PyTorch `torchvision.dataset`, configure le réseau défini dans `model.py` et l’entraîne pour deux périodes en utilisant un algorithme de gradient stochastique (ou SGD « Stochastic Gradient Descent ») et du calcul de la perte d’entropie croisée.

Créez un script `train.py` dans le sous-répertoire `src` :

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/src/train.py":::

Vous disposez maintenant de la structure de répertoires suivante :

```txt
tutorial
└──.azureml
|  └──config.json
└──src
|  └──hello.py
|  └──model.py
|  └──train.py
└──01-create-workspace.py
└──02-create-compute.py
└──03-run-hello.py
```

> [!div class="nextstepaction"]
> [J’ai créé les scripts d’entraînement](?success=create-scripts#environment) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=create-scripts)

## <a name="create-a-python-environment"></a><a name="environment"></a> Créer un environnement Python

À des fins de démonstration, nous allons utiliser un environnement Conda. (Les étapes pour un environnement virtuel pip sont pratiquement identiques.)

Créez un fichier nommé `pytorch-env.yml` dans le répertoire masqué `.azureml` :

:::code language="yml" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/environments/pytorch-env.yml":::

Cet environnement comporte toutes les dépendances requises par votre modèle et votre script de formation. Notez qu’il n’existe aucune dépendance du SDK Azure Machine Learning pour Python.

> [!div class="nextstepaction"]
> [J’ai créé le fichier d’environnement](?success=create-env-file#test-local) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=create-env-file)

## <a name="test-locally"></a><a name="test-local"></a> Tester localement

Utilisez le code suivant pour tester les exécutions de votre script localement dans cet environnement :

```bash
conda env create -f .azureml/pytorch-env.yml    # create conda environment
conda activate pytorch-env                      # activate conda environment
python src/train.py                             # train model
```

Après l’exécution de ce script, vous voyez toutes les données téléchargées dans un répertoire nommé `tutorial/data`.

> [!div class="nextstepaction"]
> [J’ai créé le fichier d’environnement](?success=test-local#create-local) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=test-local)

## <a name="create-the-control-script"></a><a name="create-local"></a> Créer le script de contrôle

La différence entre le script de contrôle suivant et celui que vous avez utilisé pour soumettre « Hello World ! » est que vous ajoutez quelques lignes supplémentaires pour définir l’environnement.

Créez un fichier Python dans le répertoire `tutorial` nommé `04-run-pytorch.py` :

```python
# 04-run-pytorch.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig

if __name__ == "__main__":
    ws = Workspace.from_config()
    experiment = Experiment(workspace=ws, name='day1-experiment-train')
    config = ScriptRunConfig(source_directory='./src',
                             script='train.py',
                             compute_target='cpu-cluster')

    # set up pytorch environment
    env = Environment.from_conda_specification(
        name='pytorch-env',
        file_path='./.azureml/pytorch-env.yml'
    )
    config.run_config.environment = env

    run = experiment.submit(config)

    aml_url = run.get_portal_url()
    print(aml_url)
```    
    
### <a name="understand-the-code-changes"></a>Comprendre les modifications du code

:::row:::
   :::column span="":::
      `env = ...`
   :::column-end:::
   :::column span="2":::
      Azure Machine Learning fournit le concept d’[environnement](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py) pour représenter un environnement Python reproductible et versionné afin d’exécuter des expériences. La création d’un environnement à partir d’un environnement Conda ou PIP local est simple.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config.run_config.environment = env`
   :::column-end:::
   :::column span="2":::
      Ajoute l’environnement à [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py).
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [J’ai créé le script de contrôle](?success=control-script#submit) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=control-script)


## <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit"></a> Soumettre l’exécution à Azure Machine Learning

Si vous êtes passé à des environnements locaux, veillez à revenir à un environnement où le SDK Azure Machine Learning pour Python est installé.

Ensuite, exécutez :

```bash
python 04-run-pytorch.py
```

>[!NOTE] 
> La première fois que vous exécutez ce script, Azure Machine Learning va générer une nouvelle image Docker à partir de votre environnement PyTorch. La totalité de l’exécution peut prendre 5 à 10 minutes. 
>
> Vous pouvez visualiser les journaux de la génération Docker dans Azure Machine Learning Studio. Suivez le lien vers le studio, sélectionnez l’onglet **Sorties + journaux**, puis sélectionnez `20_image_build_log.txt`.
>
> Cette image sera réutilisée dans les prochaines exécutions, ce qui les rendra plus rapides.

Une fois votre image générée, sélectionnez `70_driver_log.txt` pour visualiser la sortie de votre script d’entraînement.

```txt
Downloading https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz
...
Files already downloaded and verified
epoch=1, batch= 2000: loss 2.19
epoch=1, batch= 4000: loss 1.82
epoch=1, batch= 6000: loss 1.66
epoch=1, batch= 8000: loss 1.58
epoch=1, batch=10000: loss 1.52
epoch=1, batch=12000: loss 1.47
epoch=2, batch= 2000: loss 1.39
epoch=2, batch= 4000: loss 1.38
epoch=2, batch= 6000: loss 1.37
epoch=2, batch= 8000: loss 1.33
epoch=2, batch=10000: loss 1.31
epoch=2, batch=12000: loss 1.27
Finished Training
```

> [!WARNING]
> Si vous voyez une erreur `Your total snapshot size exceeds the limit`, cela signifie que le répertoire `data` se trouve dans la valeur de `source_directory` utilisée dans `ScriptRunConfig`.
>
> Déplacez `data` en dehors de `src`.

Les environnements peuvent être inscrits auprès d’un espace de travail avec `env.register(ws)`. Ils peuvent ensuite être facilement partagés, réutilisés et versionnés. Les environnements permettent de reproduire facilement les résultats précédents et de collaborer avec votre équipe.

Azure Machine Learning conserve aussi une collection d’environnements organisés. Ces environnements couvrent les scénarios de Machine Learning courants et sont sauvegardés par les images Docker mises en cache. Les images Docker mises en cache accélèrent la première exécution à distance.

En bref, l’utilisation d’environnements inscrits peut vous faire gagner du temps ! Pour plus d’informations, consultez [Comment utiliser les environnements](./how-to-use-environments.md).

> [!div class="nextstepaction"]
> [J’ai soumis l’exécution](?success=test-w-environment#log) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=test-w-environment)

## <a name="log-training-metrics"></a><a name="log"></a> Journaliser les métriques d’entraînement

Maintenant que vous avez un modèle de formation dans Azure Machine Learning, commencez le suivi de certaines mesures de performances.

Le script de formation en cours imprime les métriques sur le terminal. Azure Machine Learning fournit un mécanisme de journalisation des métriques avec davantage de fonctionnalités. En ajoutant quelques lignes de code, vous avez la possibilité de visualiser les métriques dans Studio et de comparer les métriques entre plusieurs exécutions.

### <a name="modify-trainpy-to-include-logging"></a>Modifier `train.py` pour inclure la journalisation

Modifiez votre script `train.py` pour y inclure deux lignes de code supplémentaires :

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/code/pytorch-cifar10-train-with-logging/train.py":::


#### <a name="understand-the-additional-two-lines-of-code"></a>Comprendre les deux lignes de code supplémentaires

Dans `train.py`, vous accédez à l’objet d’exécution _depuis_ le script d’entraînement lui-même en utilisant la méthode `Run.get_context()` et vous l’utilisez pour consigner des métriques :

```python
# in train.py
run = Run.get_context()

...

run.log('loss', loss)
```

Les métriques de Azure Machine Learning sont :

- Organisées par expérience et par exécution ; il est donc facile de suivre et de comparer des métriques.
- Équipées d’une interface utilisateur qui vous permet de visualiser les performances de formation dans Studio.
- Conçues pour être mises à l’échelle, vous conservez ces avantages même lorsque vous exécutez des centaines d’expériences.

> [!div class="nextstepaction"]
> [J’ai modifié train.py ](?success=modify-train#log) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=modify-train)

### <a name="update-the-conda-environment-file"></a>Mettre à jour le fichier d’environnement Conda

Le script `train.py` prenait uniquement une nouvelle dépendance sur `azureml.core`. Mettre à jour `pytorch-env.yml` pour refléter cette modification :

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/configuration/pytorch-aml-env.yml":::

> [!div class="nextstepaction"]
> [J’ai mis à jour le fichier d’environnement](?success=update-environment#submit-again) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=update-environment)

### <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit-again"></a> Soumettre l’exécution à Azure Machine Learning
Soumettre ce script une fois de plus :

```bash
python 04-run-pytorch.py
```

Cette fois, quand vous accédez au studio, accédez à l’onglet **Métriques**, où vous pouvez maintenant voir les mises à jour en direct sur la perte du modèle d’entraînement !

:::image type="content" source="media/tutorial-1st-experiment-sdk-train/logging-metrics.png" alt-text="Graphe des pertes de l’entraînement dans l’onglet Métriques.":::

> [!div class="nextstepaction"]
> [J’ai resoumis l’exécution](?success=resubmit-with-logging#next-steps) [J’ai rencontré un problème](https://www.research.net/r/7CTJQQN?issue=resubmit-with-logging)

## <a name="next-steps"></a>Étapes suivantes

Dans cette session, vous avez effectué une mise à niveau à partir d’un script « Hello World » de base vers un script de formation plus réaliste qui nécessitait l’exécution d’un environnement Python spécifique. Vous avez vu comment adopter un environnement Conda local dans le cloud avec des environnements Azure Machine Learning. Enfin, vous avez vu comment l’ajout de quelques lignes de code vous permet de journaliser des métriques dans Azure Machine Learning.

Il existe d’autres façons de créer des environnements Azure Machine Learning, y compris [à partir d’un fichier pip requirements.txt](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py#from-pip-requirements-name--file-path-) ou [à partir d’un environnement Conda existant](/python/api/azureml-core/azureml.core.environment.environment?preserve-view=true&view=azure-ml-py#from-existing-conda-environment-name--conda-environment-name-).

Dans la prochaine session, vous verrez comment utiliser des données dans Azure Machine Learning en chargeant le jeu de données CIFAR10 sur Azure.

> [!div class="nextstepaction"]
> [Tutoriel : Apporter vos propres données](tutorial-1st-experiment-bring-data.md)

>[!NOTE] 
> Si vous voulez arrêter la série de tutoriels ici sans passer à l’étape suivante, n’oubliez pas de [nettoyer vos ressources](tutorial-1st-experiment-bring-data.md#clean-up-resources).
