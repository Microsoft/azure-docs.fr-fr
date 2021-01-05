---
title: Entraîner avec azureml-datasets
titleSuffix: Azure Machine Learning
description: Découvrez mettre rendre vos données à la disposition de votre calcul local ou distant pour l’entraînement du modèle ML avec des jeux de données Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: b6ec9d7035194efc471fc06befad9822c8684a5d
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94685577"
---
# <a name="train-with-datasets-in-azure-machine-learning"></a>Entraîner avec des jeux de données dans Azure Machine Learning


Cet article va vous permettre d’apprendre à utiliser des jeux de données [Azure Machine Learning](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py) dans vos expériences de formation.  Vous pouvez utiliser des jeux de données dans votre cible de calcul locale ou distante sans vous soucier des chaînes de connexion ou des chemins de données.

Les jeux de données Azure Machine Learning fournissent une intégration transparente avec les fonctionnalités de formation d’Azure Machine Learning telles que [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py), [HyperDrive](/python/api/azureml-train-core/azureml.train.hyperdrive?preserve-view=true&view=azure-ml-py) et les [pipelines Azure Machine Learning](how-to-create-your-first-pipeline.md).

## <a name="prerequisites"></a>Prérequis

Pour créer des jeux de données et effectuer un entraînement avec eux, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.

* Un [espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python installé](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py) (1.13.0 ou version ultérieure), qui inclut le package azureml-datasets.

> [!Note]
> Certaines classes de jeu de données ont des dépendances avec le package [azureml-dataprep](/python/api/azureml-dataprep/?preserve-view=true&view=azure-ml-py). Pour les utilisateurs Linux, ces classes sont uniquement prises en charge dans les distributions suivantes :  Red Hat Enterprise Linux, Ubuntu, Fedora et CentOS.

## <a name="use-datasets-directly-in-training-scripts"></a>Utiliser des jeux de données directement dans les script d’entraînement

Si vous disposez de données structurées qui ne sont pas encore inscrites en tant que jeu de données, créez un TabularDataset et utilisez-le directement dans votre script de formation pour votre expérience locale ou distante.

Dans cet exemple, vous créez un [TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py) non inscrit et le spécifiez comme argument de script dans l’objet ScriptRunConfig pour la formation. Si vous souhaitez réutiliser ce TabularDataset avec d’autres expériences dans votre espace de travail, consultez [Comment inscrire des jeux de données dans votre espace de travail](how-to-create-register-datasets.md#register-datasets).

### <a name="create-a-tabulardataset"></a>Créer un TabularDataset

Le code suivant crée un TabularDataset non inscrit à partir d’une URL web.  

```Python
from azureml.core.dataset import Dataset

web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path)
```

Les objets TabularDataset permettent de charger les données de votre TabularDataset dans un DataFrame pandas ou Spark afin que vous puissiez travailler avec des bibliothèques de formation et de préparation des données familières sans avoir à quitter votre notebook.

### <a name="access-dataset-in-training-script"></a>Accéder au jeu de données dans le script de formation

Le code suivant configure un argument de script `--input-data` que vous spécifierez lors de la configuration de votre exécution de formation (voir la section suivante). Lorsque le jeu de données tabulaires est transmis comme valeur d’argument, Azure ML le résout en ID du jeu de données, que vous pouvez ensuite utiliser pour accéder au jeu de données dans votre script de formation (sans avoir à coder en dur le nom ou l’ID du jeu de données dans votre script). Il utilise ensuite la méthode [`to_pandas_dataframe()`](/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) pour charger ce jeu de données dans un dataframe pandas afin d’approfondir l’exploration et la préparation des données avant la formation.

> [!Note]
> Si votre source de données originale contient NaN, des chaînes vides ou des valeurs vides, lorsque vous utilisez `to_pandas_dataframe()`, ces valeurs sont remplacées par une valeur *Null*.

Si vous avez besoin de charger les données préparées dans un nouveau jeu de données à partir d’un dataframe pandas en mémoire, écrivez les données dans un fichier local (par exemple un fichier Parquet), puis créez un jeu de données à partir de ce fichier. Vous pouvez également créer des jeux de données à partir de chemins ou de fichiers locaux dans des magasins de données. Pour en savoir plus sur la création des jeux de données, consultez [cette page](how-to-create-register-datasets.md).

```Python
%%writefile $script_folder/train_titanic.py

import argparse
from azureml.core import Dataset, Run

parser = argparse.ArgumentParser()
parser.add_argument("--input-data", type=str)
args = parser.parse_args()

run = Run.get_context()
ws = run.experiment.workspace

# get the input dataset by ID
dataset = Dataset.get_by_id(ws, id=args.input_data)

# load the TabularDataset to pandas DataFrame
df = dataset.to_pandas_dataframe()
```

### <a name="configure-the-training-run"></a>Configurer l’exécution de l’apprentissage
Un objet [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrun?preserve-view=true&view=azure-ml-py) est utilisé pour configurer et soumettre l’exécution de la formation.

Ce code crée un objet ScriptRunConfig, `src`, qui spécifie :

* Un répertoire de script pour vos scripts. Tous les fichiers dans ce répertoire sont chargés dans les nœuds de cluster pour l’exécution.
* Le script d’entraînement, *train_titanic.py*.
* Le jeu de données d’entrée pour la formation, `titanic_ds`, comme argument de script. Azure ML le résoudra en ID correspondant au jeu de données lorsqu’il sera transmis à votre script.
* La cible de calcul pour l’exécution.
* L’environnement pour l’exécution.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=script_folder,
                      script='train_titanic.py',
                      # pass dataset as an input with friendly name 'titanic'
                      arguments=['--input-data', titanic_ds.as_named_input('titanic')],
                      compute_target=compute_target,
                      environment=myenv)
                             
# Submit the run configuration for your training run
run = experiment.submit(src)
run.wait_for_completion(show_output=True)                             
```

## <a name="mount-files-to-remote-compute-targets"></a>Monter des fichiers sur des cibles de calcul distantes

Si vous avez des données non structurées, créez un [FileDataset](/python/api/azureml-core/azureml.data.filedataset?preserve-view=true&view=azure-ml-py) et montez ou téléchargez vos fichiers de données pour les mettre à la disposition de votre cible de calcul à distance pour la formation. Découvrez quand utiliser [le montage ou le téléchargement](#mount-vs-download) pour vos expériences de formation à distance. 

L’exemple suivant crée un FileDataset et monte le jeu de données sur la cible de calcul en le transmettant comme argument au script de formation. 

> [!Note]
> Si vous utilisez une image de base Docker personnalisée, vous devez installer fuse via `apt-get install -y fuse` en tant que dépendance pour que le montage du jeu de données fonctionne. Découvrez la procédure de [création d’une image de build personnalisée](how-to-deploy-custom-docker-image.md#build-a-custom-base-image).

### <a name="create-a-filedataset"></a>Créer un FileDataset

L’exemple suivant crée un FileDataset non inscrit à partir d’URL web. Pour en savoir plus sur la création des jeux de données à partir d’autres sources, consultez [cette page](how-to-create-register-datasets.md).

```Python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
mnist_ds = Dataset.File.from_files(path = web_paths)
```

### <a name="configure-the-training-run"></a>Configurer l’exécution de l’apprentissage
Nous vous recommandons de transmettre le jeu de données comme argument lors du montage via le paramètre `arguments` du constructeur `ScriptRunConfig`. En procédant ainsi, vous obtiendrez le chemin de données (point de montage) dans votre script de formation via des arguments. Ainsi, vous serez en mesure d’utiliser le même script de formation pour le débogage local et la formation à distance sur toute plateforme cloud.

L’exemple suivant crée un ScriptRunConfig qui passe dans le FileDataset via `arguments`. Une fois que vous avez envoyé l’exécution, les fichiers de données référencés par le jeu de données `mnist_ds` sont montés sur la cible de calcul.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=script_folder,
                      script='train_mnist.py',
                      # the dataset will be mounted on the remote compute and the mounted path passed as an argument to the script
                      arguments=['--data-folder', mnist_ds.as_mount(), '--regularization', 0.5],
                      compute_target=compute_target,
                      environment=myenv)

# Submit the run configuration for your training run
run = experiment.submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="retrieve-the-data-in-your-training-script"></a>Récupérer les données dans votre script d’entraînement

Le code suivant montre comment récupérer les données dans votre script.

```Python
%%writefile $script_folder/train_mnist.py

import argparse
import os
import numpy as np
import glob

from utils import load_data

# retrieve the 2 arguments configured through `arguments` in the ScriptRunConfig
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# get the file paths on the compute
X_train_path = glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0]
X_test_path = glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0]
y_train_path = glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0]
y_test = glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0]

# load train and test set into numpy arrays
X_train = load_data(X_train_path, False) / 255.0
X_test = load_data(X_test_path, False) / 255.0
y_train = load_data(y_train_path, True).reshape(-1)
y_test = load_data(y_test, True).reshape(-1)
```

## <a name="mount-vs-download"></a>Différences entre montage et téléchargement

Le montage ou le téléchargement de fichiers de tout format sont pris en charge pour des jeux de données créés à partir des stockages suivants : Stockage Blob Azure, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database et Azure Database pour PostgreSQL. 

Quand vous **montez** un jeu de données, vous attachez les fichiers référencés par le jeu de données à un répertoire (point de montage) et le rendez disponible sur la cible de calcul. Le montage est pris en charge pour les calculs basés sur Linux, y compris la capacité de calcul Azure Machine Learning, les machines virtuelles et HDInsight. 

Lorsque vous **téléchargez** un jeu de données, tous les fichiers référencés par le jeu de données sont téléchargés sur la cible de calcul. Le téléchargement est pris en charge pour tous les types de calcul. 

Si votre script traite tous les fichiers référencés par le jeu de données et que votre disque de calcul peut contenir le jeu de données complet, le téléchargement est recommandé pour éviter la charge de traitement inhérente à la diffusion en continu des données à partir des services de stockage. Si la taille de vos données dépasse la taille du disque de calcul, le téléchargement n’est pas possible. Pour ce scénario, nous recommandons un montage, car seuls les fichiers de données utilisés par votre script sont chargés au moment du traitement.

Le code suivant monte `dataset` dans le répertoire Temp dans le chemin `mounted_path`

```python
import tempfile
mounted_path = tempfile.mkdtemp()

# mount dataset onto the mounted_path of a Linux-based compute
mount_context = dataset.mount(mounted_path)

mount_context.start()

import os
print(os.listdir(mounted_path))
print (mounted_path)
```

## <a name="directly-access-datasets-in-your-script"></a>Accéder directement aux jeux de données dans votre script

Les jeux de données inscrits sont accessibles localement et à distance sur des clusters de calcul comme la capacité de calcul Azure Machine Learning. Pour accéder à votre jeu de données inscrit dans plusieurs expériences, utilisez le code suivant afin d’accéder à votre espace de travail et à votre jeu de données inscrit en utilisant son nom. Par défaut, la méthode [`get_by_name()`](/python/api/azureml-core/azureml.core.dataset.dataset?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-by-name-workspace--name--version--latest--) sur la classe `Dataset` retourne la dernière version du jeu de données inscrit auprès de l’espace de travail.

```Python
%%writefile $script_folder/train.py

from azureml.core import Dataset, Run

run = Run.get_context()
workspace = run.experiment.workspace

dataset_name = 'titanic_ds'

# Get a dataset by name
titanic_ds = Dataset.get_by_name(workspace=workspace, name=dataset_name)

# Load a TabularDataset into pandas DataFrame
df = titanic_ds.to_pandas_dataframe()
```

## <a name="accessing-source-code-during-training"></a>Accès au code source pendant l’entraînement

Le stockage Blob Azure offre des vitesses de débit supérieures à celles du partage de fichiers Azure et s’adapte à un grand nombre de travaux démarrés en parallèle. C’est pourquoi nous vous recommandons de configurer vos exécutions pour utiliser le stockage Blob pour le transfert des fichiers de code source.

L’exemple de code suivant spécifie dans la configuration d’exécution le magasin de données d’objets blob à utiliser pour les transferts de code source.

```python 
# workspaceblobstore is the default blob storage
src.run_config.source_directory_data_store = "workspaceblobstore" 
```

## <a name="notebook-examples"></a>Exemples de notebooks

+ Les [notebooks de jeux de données](https://aka.ms/dataset-tutorial) illustrent et développent les concepts abordés dans cet article.
+ Découvrez comment [paramétrer des jeux de données dans vos pipelines ML](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-showcasing-dataset-and-pipelineparameter.ipynb).

## <a name="next-steps"></a>Étapes suivantes

* [Entraîner automatiquement des modèles Machine Learning](how-to-auto-train-remote.md) avec des TabularDatasets.

* [Entraîner des modèles de classification d’image](https://aka.ms/filedataset-samplenotebook) avec des FileDatasets.

* [Effectuez l’apprentissage des jeux de données à l'aide de pipelines](how-to-create-your-first-pipeline.md).
