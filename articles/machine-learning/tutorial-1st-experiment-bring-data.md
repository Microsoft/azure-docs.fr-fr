---
title: 'Tutoriel : Utiliser vos propres données'
titleSuffix: Azure Machine Learning
description: La partie 4 de la série de tutoriels sur la prise en main d’Azure Machine Learning vous explique comment utiliser vos propres données dans le cadre d’un entraînement à distance.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/15/2020
ms.custom: tracking-python
ms.openlocfilehash: de89f9d87b010dc3710e7d82f4d846de12303905
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739431"
---
# <a name="tutorial-use-your-own-data-part-4-of-4"></a>Tutoriel : Utiliser vos propres données (partie 4 sur 4)

Ce tutoriel vous montre comment télécharger et utiliser vos propres données pour former des modèles Machine Learning dans Azure Machine Learning.

Ce tutoriel constitue la *quatrième partie d’une série de quatre tutoriels* conçus pour vous apprendre les principes fondamentaux d’Azure Machine Learning et à effectuer des tâches de machine learning basées sur des travaux dans Azure. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 1 : Configurer un script « Hello World »](tutorial-1st-experiment-sdk-setup-local.md), [Partie 2 : Exécuter « Hello World »](tutorial-1st-experiment-hello-world.md) et [Partie 3 : Effectuer l'apprentissage d'un modèle](tutorial-1st-experiment-sdk-train.md).

Dans le troisième tutoriel de notre série, [Partie 3 : Entraîner un modèle](tutorial-1st-experiment-sdk-train.md), des données ont été téléchargées via la méthode `torchvision.datasets.CIFAR10` intégrée dans l’API PyTorch. Cependant, dans de nombreux cas, vous allez utiliser vos propres données pour réaliser un entraînement à distance. Cet article montre le workflow que vous pouvez suivre pour travailler avec vos propres données dans Azure Machine Learning.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Configurer un script d’entraînement pour utiliser des données dans un répertoire local.
> * Tester le script d’entraînement en local.
> * Charger des données sur Azure.
> * Créer un script de contrôle.
> * Comprendre les nouveaux concepts d’Azure Machine Learning (passage de paramètres, jeux de données, magasins de données).
> * Soumettre et exécuter votre script de formation.
> * Afficher le résultat de votre code dans le cloud.

## <a name="prerequisites"></a>Prérequis

* Avoir effectué la [partie 3](tutorial-1st-experiment-sdk-train.md) de la série.
* Avoir des connaissances préliminaires du langage Python et des workflows Machine Learning.
* Environnement de développement local, comme Visual Studio Code, Jupyter et PyCharm.
* Python (version 3.5 à 3.7).

## <a name="adjust-the-training-script"></a>Ajuster le script de formation

À présent, votre script d’entraînement (tutorial/src/train.py) s’exécute dans Azure Machine Learning et peut superviser les performances du modèle. Nous allons paramétrer le script d’entraînement en introduisant des arguments. L’utilisation d’arguments vous permet de comparer facilement différents hyperparamètres.

Actuellement, notre script d’entraînement est configuré pour télécharger le jeu de données CIFAR10 à chaque exécution. Le code Python ci-dessous a été ajusté pour lire les données d’un répertoire.

>[!NOTE] 
> L’utilisation de `argparse` paramètre le script.

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/code/pytorch-cifar10-your-data/train.py":::

### <a name="understanding-the-code-changes"></a>Compréhension des modifications du code

Le code de `train.py` a utilisé la bibliothèque `argparse` pour configurer `data_path`, `learning_rate` et `momentum`.

```python
# .... other code
parser = argparse.ArgumentParser()
parser.add_argument('--data_path', type=str, help='Path to the training data')
parser.add_argument('--learning_rate', type=float, default=0.001, help='Learning rate for SGD')
parser.add_argument('--momentum', type=float, default=0.9, help='Momentum for SGD')
args = parser.parse_args()
# ... other code
```

En outre, le script `train.py` a été adapté pour mettre à jour l’optimiseur afin d’utiliser les paramètres définis par l’utilisateur :

```python
optimizer = optim.SGD(
    net.parameters(),
    lr=args.learning_rate,     # get learning rate from command-line argument
    momentum=args.momentum,    # get momentum from command-line argument
)
```
> [!div class="nextstepaction"]
> [J’ai modifié le script d’entraînement](?success=adjust-training-script#test-locally) [J’ai rencontré un problème](https://www.research.net/r/7C6W7BQ?issue=adjust-training-script)

## <a name="test-the-script-locally"></a><a name="test-locally"></a> Tester le script localement

Votre script accepte à présent _data path_ (chemin d’accès aux données) en tant qu’argument. Pour commencer, effectuez un test en local. Ajoutez à la structure de répertoires de votre tutoriel un dossier appelé `data`. La structure de votre répertoire doit ressembler à ce qui suit :

```txt
tutorial
└──.azureml
|  └──config.json
|  └──pytorch-env.yml
└──data
└──src
|  └──hello.py
|  └──model.py
|  └──train.py
└──01-create-workspace.py
└──02-create-compute.py
└──03-run-hello.py
└──04-run-pytorch.py
```

Si vous n’avez pas exécuté `train.py` localement dans le tutoriel précédent, vous n’avez pas le répertoire `data/`. Dans ce cas, exécutez la méthode `torchvision.datasets.CIFAR10` en local avec `download=True` dans votre script `train.py`.

Pour exécuter en local le script de formation modifié, appelez :

```bash
python src/train.py --data_path ./data --learning_rate 0.003 --momentum 0.92
```

Vous ne devez pas télécharger le jeu de données CIFAR10 en passant un chemin d’accès local aux données. Vous pouvez également faire des expériences avec différentes valeurs pour les hyperparamètres _learning rate_ et _momentum_ sans avoir à les coder en dur dans le script d’entraînement.

> [!div class="nextstepaction"]
> [J’ai testé le script localement](?success=test-locally#upload) [J’ai rencontré un problème](https://www.research.net/r/7C6W7BQ?issue=test-locally)

## <a name="upload-the-data-to-azure"></a><a name="upload"></a> Charger les données dans Azure

Pour exécuter ce script dans Azure Machine Learning, vous devez rendre vos données d’entraînement disponibles dans Azure. Votre espace de travail Azure Machine Learning est fourni avec un magasin de données _par défaut_. Il s’agit d’un compte Stockage Blob Azure où vous pouvez stocker vos données d’entraînement.

>[!NOTE] 
> Azure Machine Learning vous permet de connecter d’autres magasins de données cloud qui stockent vos données. Pour plus d’informations, consultez la [documentation des magasins de données](./concept-data.md).  

Créez un script de contrôle Python appelé `05-upload-data.py` dans le répertoire `tutorial` :

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/05-upload-data.py":::

La valeur de `target_path` spécifie le chemin du magasin de données où les données CIFAR10 seront téléchargées.

>[!TIP] 
> Vous utilisez Azure Machine Learning pour charger les données, mais vous pouvez utiliser l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) pour charger des fichiers ad hoc. Si vous avez besoin d’un outil ETL, vous pouvez utiliser [Azure Data Factory](../data-factory/introduction.md) pour ingérer vos données dans Azure.

Exécutez le fichier Python pour charger les données. (Le chargement doit normalement être rapide, moins de 60 secondes.)

```bash
python 05-upload-data.py
```

Vous devez normalement voir la sortie standard suivante :

```txt
Uploading ./data\cifar-10-batches-py\data_batch_2
Uploaded ./data\cifar-10-batches-py\data_batch_2, 4 files out of an estimated total of 9
.
.
Uploading ./data\cifar-10-batches-py\data_batch_5
Uploaded ./data\cifar-10-batches-py\data_batch_5, 9 files out of an estimated total of 9
Uploaded 9 files
```

> [!div class="nextstepaction"]
> [J’ai chargé les données](?success=upload-data#control-script) [J’ai rencontré un problème](https://www.research.net/r/7C6W7BQ?issue=upload-data)

## <a name="create-a-control-script"></a><a name="control-script"></a> Créer un script de contrôle

Comme vous l’avez fait précédemment, créez un script de contrôle Python appelé `06-run-pytorch-data.py` :

```python
# 06-run-pytorch-data.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig
from azureml.core import Dataset

if __name__ == "__main__":
    ws = Workspace.from_config()
    datastore = ws.get_default_datastore()
    dataset = Dataset.File.from_files(path=(datastore, 'datasets/cifar10'))

    experiment = Experiment(workspace=ws, name='day1-experiment-data')

    config = ScriptRunConfig(
        source_directory='./src',
        script='train.py',
        compute_target='cpu-cluster',
        arguments=[
            '--data_path', dataset.as_named_input('input').as_mount(),
            '--learning_rate', 0.003,
            '--momentum', 0.92],
    )
    # set up pytorch environment
    env = Environment.from_conda_specification(
        name='pytorch-env',
        file_path='./.azureml/pytorch-env.yml'
    )
    config.run_config.environment = env

    run = experiment.submit(config)
    aml_url = run.get_portal_url()
    print("Submitted to compute cluster. Click link below")
    print("")
    print(aml_url)
```

### <a name="understand-the-code-changes"></a>Comprendre les modifications du code

Le script de contrôle est similaire à celui de la [partie 3 de notre série](tutorial-1st-experiment-sdk-train.md), avec ces nouvelles lignes de code :

:::row:::
   :::column span="":::
      `dataset = Dataset.File.from_files( ... )`
   :::column-end:::
   :::column span="2":::
      Un [jeu de données](/python/api/azureml-core/azureml.core.dataset.dataset?preserve-view=true&view=azure-ml-py) est utilisé pour référencer les données que vous avez chargées dans Stockage Blob Azure. Les jeux de données sont une couche d’abstraction au-dessus de vos données, conçue pour améliorer la fiabilité.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config = ScriptRunConfig(...)`
   :::column-end:::
   :::column span="2":::
      [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) est modifié pour inclure une liste d’arguments qui seront passés dans `train.py`. L’argument `dataset.as_named_input('input').as_mount()` signifie que le répertoire spécifié sera _monté_ sur la cible de calcul.
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [J’ai créé le script de contrôle](?success=control-script#submit-to-cloud) [J’ai rencontré un problème](https://www.research.net/r/7C6W7BQ?issue=control-script)

## <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit-to-cloud"></a> Soumettre l’exécution à Azure Machine Learning

À présent, soumettez à nouveau l’exécution pour utiliser la nouvelle configuration :

```bash
python 06-run-pytorch-data.py
```

Ce code va produire une URL vers l’expérience dans Azure Machine Learning Studio. Si vous accédez à ce lien, vous pourrez voir votre code en cours d’exécution.

> [!div class="nextstepaction"]
> [J’ai resoumis l’exécution](?success=submit-to-cloud#inspect-log) [J’ai rencontré un problème](https://www.research.net/r/7C6W7BQ?issue=submit-to-cloud)

### <a name="inspect-the-log-file"></a><a name="inspect-log"></a> Examiner le fichier journal

Dans le studio, accédez à l’exécution de l’expérience (en sélectionnant la sortie de l’URL précédente), puis à **Sorties + journaux**. Sélectionnez le fichier `70_driver_log.txt`. Vous devez normalement voir la sortie suivante :

```txt
Processing 'input'.
Processing dataset FileDataset
{
  "source": [
    "('workspaceblobstore', 'datasets/cifar10')"
  ],
  "definition": [
    "GetDatastoreFiles"
  ],
  "registration": {
    "id": "XXXXX",
    "name": null,
    "version": null,
    "workspace": "Workspace.create(name='XXXX', subscription_id='XXXX', resource_group='X')"
  }
}
Mounting input to /tmp/tmp9kituvp3.
Mounted input to /tmp/tmp9kituvp3 as folder.
Exit __enter__ of DatasetContextManager
Entering Run History Context Manager.
Current directory:  /mnt/batch/tasks/shared/LS_root/jobs/dsvm-aml/azureml/tutorial-session-3_1600171983_763c5381/mounts/workspaceblobstore/azureml/tutorial-session-3_1600171983_763c5381
Preparing to call script [ train.py ] with arguments: ['--data_path', '$input', '--learning_rate', '0.003', '--momentum', '0.92']
After variable expansion, calling script [ train.py ] with arguments: ['--data_path', '/tmp/tmp9kituvp3', '--learning_rate', '0.003', '--momentum', '0.92']

Script type = None
===== DATA =====
DATA PATH: /tmp/tmp9kituvp3
LIST FILES IN DATA PATH...
['cifar-10-batches-py', 'cifar-10-python.tar.gz']
```

Remarques :

- Azure Machine Learning a monté automatiquement pour vous Stockage Blob sur le cluster de calcul.
- ``dataset.as_named_input('input').as_mount()`` utilisé dans le script de contrôle donne après résolution le point de montage.

> [!div class="nextstepaction"]
> [J’ai examiné le fichier journal](?success=inspect-log#clean-up-resources) [J’ai rencontré un problème](https://www.research.net/r/7C6W7BQ?issue=inspect-log)

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

Vous pouvez également conserver le groupe de ressources mais supprimer un espace de travail unique. Affichez les propriétés de l’espace de travail, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, nous avons vu comment charger des données sur Azure en utilisant `Datastore`. Le magasin de données a servi de stockage cloud pour votre espace de travail, ce qui vous offre un emplacement permanent et flexible pour conserver vos données.

Vous avez appris comment modifier votre script de formation pour accepter un chemin d’accès aux données via la ligne de commande. En utilisant `Dataset` vous avez pu monter un répertoire sur l’exécution distante. 

Maintenant que vous avez un modèle, découvrez :

* Comment [déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).
