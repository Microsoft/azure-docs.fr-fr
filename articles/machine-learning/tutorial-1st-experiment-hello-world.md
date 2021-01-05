---
title: 'Tutoriel : Exécuter un script Python « Hello World! »'
titleSuffix: Azure Machine Learning
description: La partie 2 de notre série de tutoriels sur la prise en main d’Azure Machine Learning explique comment soumettre un script Python « Hello World » simple dans le cloud.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/15/2020
ms.custom: devx-track-python
ms.openlocfilehash: 971bac8a0b0951d4e07e139aea6c465a9159b8db
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96570958"
---
# <a name="tutorial-run-a-hello-world-python-script-part-2-of-4"></a>Tutoriel : Exécuter un script Python « Hello World! » (partie 2 sur 4)

Dans ce tutoriel, vous allez découvrir comment utiliser le SDK Azure Machine Learning pour Python pour soumettre et exécuter un script Python « Hello World! ».

Ce tutoriel constitue la *partie 2 d’une série de quatre tutoriels* où vous découvrez les principes fondamentaux d’Azure Machine Learning et comment effectuer des tâches de machine learning basées sur des travaux dans Azure. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 1 : Configurer votre machine locale pour Azure Machine Learning](tutorial-1st-experiment-sdk-setup-local.md).

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
> * Créer et exécuter un script Python « Hello World! » localement.
> * Créer un script de contrôle Python pour soumettre « Hello World » à Azure Machine Learning.
> * Comprendre les concepts d’Azure Machine Learning en matière de script de contrôle.
> * Soumettre et exécuter le script « Hello world! ».
> * Afficher le résultat de votre code dans le cloud.

## <a name="prerequisites"></a>Prérequis

- Avoir effectué la [partie 1](tutorial-1st-experiment-sdk-setup-local.md) si vous n’avez pas déjà un espace de travail Azure Machine Learning.
- Avoir des connaissances préliminaires du langage Python et des workflows Machine Learning.
- Environnement de développement local, comme Visual Studio Code, Jupyter et PyCharm.
- Python (version 3.5 à 3.7).

## <a name="create-and-run-a-python-script-locally"></a>Créer et exécuter un script Python en local

Créez un sous-répertoire appelé `src` sous le répertoire `tutorial` pour stocker le code que vous souhaitez exécuter sur un cluster de calcul Azure Machine Learning. Dans le sous-répertoire `src`, créez le script Python `hello.py` :

```python
# src/hello.py
print("Hello world!")
```

La structure du répertoire de votre projet doit maintenant ressembler à ce qui suit :

```Bash
tutorial
└──.azureml
|  └──config.json
└──src
|  └──hello.py
└──01-create-workspace.py
└──02-create-compute.py
```


### <a name="test-your-script-locally"></a><a name="test"></a>Valider votre script en local

Vous pouvez exécuter votre code localement en utilisant votre IDE favori ou un terminal. L’exécution locale du code présente l’avantage de pouvoir déboguer le code en mode interactif.

```bash
cd <path/to/tutorial>
python ./src/hello.py
```

> [!div class="nextstepaction"]
> [J’ai exécuté le script localement](?success=run-local#control-script) [J’ai rencontré un problème](https://www.research.net/r/7C2NTH7?issue=run-local)

## <a name="create-a-control-script"></a><a name="control-script"></a> Créer un script de contrôle

Un *script de contrôle* vous permet d’exécuter votre script `hello.py` dans le cloud. Vous utilisez le script de contrôle pour contrôler comment et où votre code de machine learning est exécuté.  

Dans le répertoire de votre tutoriel, créez un nouveau fichier Python appelé `03-run-hello.py` et copiez-collez le code suivant dans ce fichier :

```python
# tutorial/03-run-hello.py
from azureml.core import Workspace, Experiment, Environment, ScriptRunConfig

ws = Workspace.from_config()
experiment = Experiment(workspace=ws, name='day1-experiment-hello')

config = ScriptRunConfig(source_directory='./src', script='hello.py', compute_target='cpu-cluster')

run = experiment.submit(config)
aml_url = run.get_portal_url()
print(aml_url)
```



### <a name="understand-the-code"></a>Comprendre le code

Voici une description du fonctionnement du script de contrôle :

:::row:::
   :::column span="":::
      `ws = Workspace.from_config()`
   :::column-end:::
   :::column span="2":::
      [L’espace de travail](/python/api/azureml-core/azureml.core.workspace.workspace?preserve-view=true&view=azure-ml-py) se connecte à votre espace de travail Azure Machine Learning, afin que vous puissiez communiquer avec vos ressources Azure Machine Learning.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `experiment =  Experiment( ... )`
   :::column-end:::
   :::column span="2":::
      [L’expérience](/python/api/azureml-core/azureml.core.experiment.experiment?preserve-view=true&view=azure-ml-py) offre un moyen simple d’organiser plusieurs exécutions sous un nom unique. Plus tard, vous pourrez voir comment les expériences facilitent la comparaison des mesures entre des dizaines d’exécutions.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config = ScriptRunConfig( ... )` 
   :::column-end:::
   :::column span="2":::
      [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) encapsule votre code `hello.py` et le transmet à votre espace de travail. Comme son nom l’indique, vous pouvez utiliser cette classe pour _configurer_ la manière dont vous souhaitez que votre _script_ _s’exécute_ dans Azure Machine Learning. Il spécifie également la cible de calcul sur laquelle le script s’exécutera. Dans ce code, la cible est le cluster de calcul que vous avez créé dans le [tutoriel de configuration](tutorial-1st-experiment-sdk-setup-local.md).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `run = experiment.submit(config)`
   :::column-end:::
   :::column span="2":::
       Soumet votre script. Cette soumission est appelée une [exécution](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py). Un Run encapsule une seule exécution de votre code. Utilisez une exécution pour surveiller la progression du script, capturer la sortie, analyser les résultats, visualiser les métriques, etc.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `aml_url = run.get_portal_url()` 
   :::column-end:::
   :::column span="2":::
        L’objet `run` fournit un handle sur l’exécution de votre code. Surveillez la progression depuis Azure Machine Learning Studio avec l’URL affichée par le script Python.  
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [J’ai créé le script de contrôle](?success=create-control-script#submit) [J’ai rencontré un problème](https://www.research.net/r/7C2NTH7?issue=create-control-script)

## <a name="submit-and-run-your-code-in-the-cloud"></a><a name="submit"></a> Soumettre et exécuter votre code dans le cloud

Exécutez votre script de contrôle, qui exécute à son tout `hello.py` sur le cluster de calcul que vous avez créé dans le [tutoriel de configuration](tutorial-1st-experiment-sdk-setup-local.md).

Comptez 5 à 10 minutes pour la première exécution. Ceci est dû aux événements suivants :

* Une image Docker est créée dans le cloud
* Le cluster de calcul repasse de 0 à 1 nœud
* L’image Docker est téléchargée dans le calcul. 

Les exécutions suivantes sont plus rapides (environ 15 secondes) car l’image Docker est mise en cache dans le calcul. Pour le vérifier, vous pouvez renvoyer le code ci-dessous une fois la première exécution terminée.

```bash
python 03-run-hello.py
```

> [!TIP]
> Si l’exécution de ce code génère une erreur indiquant que vous n’avez pas accès à l’abonnement, consultez [Se connecter à un espace de travail](how-to-manage-workspace.md?tab=python#connect-multi-tenant) pour obtenir plus d’informations sur les options d’authentification.

> [!div class="nextstepaction"]
> [J’ai soumis le code dans le cloud](?success=submit-to-cloud#monitor) [J’ai rencontré un problème](https://www.research.net/r/7C2NTH7?issue=submit-to-cloud)

## <a name="monitor-your-code-in-the-cloud-by-using-the-studio"></a><a name="monitor"></a>Surveiller votre code dans le cloud en utilisant Studio

La sortie va contenir un lien vers Studio qui se présente comme ceci : `https://ml.azure.com/experiments/hello-world/runs/<run-id>?wsid=/subscriptions/<subscription-id>/resourcegroups/<resource-group>/workspaces/<workspace-name>`.

Suivez le lien et accédez à l’onglet **Sorties + journaux**. Vous pouvez y voir un fichier `70_driver_log.txt` qui se présente comme ceci :

```txt
 1: [2020-08-04T22:15:44.407305] Entering context manager injector.
 2: [context_manager_injector.py] Command line Options: Namespace(inject=['ProjectPythonPath:context_managers.ProjectPythonPath', 'RunHistory:context_managers.RunHistory', 'TrackUserError:context_managers.TrackUserError', 'UserExceptions:context_managers.UserExceptions'], invocation=['hello.py'])
 3: Starting the daemon thread to refresh tokens in background for process with pid = 31263
 4: Entering Run History Context Manager.
 5: Preparing to call script [ hello.py ] with arguments: []
 6: After variable expansion, calling script [ hello.py ] with arguments: []
 7:
 8: Hello world!
 9: Starting the daemon thread to refresh tokens in background for process with pid = 31263
10:
11:
12: The experiment completed successfully. Finalizing run...
13: Logging experiment finalizing status in history service.
14: [2020-08-04T22:15:46.541334] TimeoutHandler __init__
15: [2020-08-04T22:15:46.541396] TimeoutHandler __enter__
16: Cleaning up all outstanding Run operations, waiting 300.0 seconds
17: 1 items cleaning up...
18: Cleanup took 0.1812913417816162 seconds
19: [2020-08-04T22:15:47.040203] TimeoutHandler __exit__
```

À la ligne 8, vous voyez apparaître la sortie du script « Hello World ! » .

Le fichier `70_driver_log.txt` contient la sortie standard d’une exécution. Ce fichier peut être utile lors du débogage des exécutions distantes dans le cloud.

> [!div class="nextstepaction"]
> [J’ai vu le journal dans le studio](?success=monitor-in-studio#next-steps) [J’ai rencontré un problème](https://www.research.net/r/7C2NTH7?issue=monitor-in-studio)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez pris un script « Hello World » simple et vous l’avez exécuté sur Azure. Vous avez vu comment vous connecter à votre espace de travail Azure Machine Learning, comment créer une expérience et comment soumettre votre code `hello.py` dans le cloud.

Dans le tutoriel suivant, vous allez développer ces apprentissages en exécutant un nom plus intéressant que `print("Hello world!")`.

> [!div class="nextstepaction"]
> [Tutoriel : entraîner un modèle](tutorial-1st-experiment-sdk-train.md)

>[!NOTE] 
> Si vous voulez arrêter la série de tutoriels ici sans passer à l’étape suivante, n’oubliez pas de [nettoyer vos ressources](tutorial-1st-experiment-bring-data.md#clean-up-resources).