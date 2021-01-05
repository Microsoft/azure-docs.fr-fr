---
title: Déplacement de données dans des pipelines ML
titleSuffix: Azure Machine Learning
description: Découvrez la façon dont les pipelines Azure Machine Learning ingèrent les données et apprenez à gérer et à déplacer les données entre les étapes du pipeline.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 08/20/2020
ms.topic: conceptual
ms.custom: how-to, contperf-fy20q4, devx-track-python, data4ml
ms.openlocfilehash: 8a5663df590e0f617f8049f0201e6e508351c755
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027575"
---
# <a name="moving-data-into-and-between-ml-pipeline-steps-python"></a>Déplacement de données au sein d’un pipeline ML et vers un autre pipeline ML (Python)



Cet article fournit du code pour l’importation, la transformation et le déplacement de données entre les étapes d’un pipeline Azure Machine Learning. Pour obtenir une vue d’ensemble du fonctionnement des données dans Azure Machine Learning, consultez [Accéder aux données dans les services de stockage Azure](how-to-access-data.md). Pour plus d’informations sur les avantages et la structure des pipelines Azure Machine Learning, consultez [Présentation des pipelines Azure Machine Learning](concept-ml-pipelines.md).

Cette article vous explique comment :

- Utiliser des objets `Dataset` pour les données préexistantes
- Accéder aux données dans vos étapes
- Fractionner les données `Dataset` en sous-ensembles, tels que des sous-ensembles d’apprentissage et de validation
- Créer des objets `PipelineData` pour transférer des données à l’étape suivante du pipeline
- Utiliser des objets `PipelineData` comme entrée pour les étapes de pipeline
- Créer des objets `Dataset` à partir des `PipelineData` que vous souhaitez conserver

> [!TIP]
> Une expérience améliorée pour la transmission de données temporaires entre les étapes de pipeline et la conservation de vos données après des exécutions de pipeline est disponible dans les classes en préversion publique, [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py) et [`OutputTabularDatasetConfig`](/python/api/azureml-core/azureml.data.output_dataset_config.outputtabulardatasetconfig?preserve-view=true&view=azure-ml-py).  Ces classes sont des fonctionnalités d’évaluation [expérimentales](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#&preserve-view=truestable-vs-experimental) susceptibles d’évoluer à tout moment.


## <a name="prerequisites"></a>Prérequis

Vous devez disposer des éléments suivants :

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

- Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) ou l’accès à [Azure Machine Learning studio](https://ml.azure.com/).

- Un espace de travail Azure Machine Learning.
  
  [Créez un espace de travail Azure Machine Learning](how-to-manage-workspace.md) ou utilisez un espace de travail existant via le Kit de développement logiciel (SDK) Python. Importez les classes `Workspace` et `Datastore`, puis chargez vos informations d’abonnement à partir du fichier `config.json` à l’aide de la fonction `from_config()`. Cette fonction recherche le fichier JSON dans le répertoire actif par défaut, mais vous pouvez également spécifier un paramètre de chemin d’accès pour pointer vers le fichier à l’aide de `from_config(path="your/file/path")`.

   ```python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

- Des données préexistantes. Cet article présente brièvement l’utilisation d’un [conteneur d’objets Blob Azure](../storage/blobs/storage-blobs-overview.md).

- Facultatif : Un pipeline de Machine Learning existant, tel que celui décrit dans [Créer et exécuter des pipelines de Machine Learning avec le kit SDK Azure Machine Learning](how-to-create-your-first-pipeline.md).

## <a name="use-dataset-objects-for-pre-existing-data"></a>Utiliser des objets `Dataset` pour les données préexistantes 

Le meilleur moyen d’ingérer des données dans un pipeline consiste à utiliser un objet [Dataset](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py). Les objets `Dataset` représentent les données persistantes disponibles dans un espace de travail.

Il existe de nombreuses façons de créer et d’inscrire des objets `Dataset`. Les jeux de données tabulaires sont destinés à des données délimitées disponibles dans un ou plusieurs fichiers. Les jeux de données de fichier sont destinés à des données binaires (telles que des images) ou des données que vous allez analyser. La méthode la plus simple pour créer des objets `Dataset` consiste à utiliser des objets Blob existants dans le stockage de l’espace de travail ou à des URL publiques :

```python
datastore = Datastore.get(workspace, 'training_data')
iris_dataset = Dataset.Tabular.from_delimited_files(DataPath(datastore, 'iris.csv'))

cats_dogs_dataset = Dataset.File.from_files(
    paths='https://download.microsoft.com/download/3/E/1/3E1C3F21-ECDB-4869-8368-6DEBA77B919F/kagglecatsanddogs_3367a.zip',
    archive_options=ArchiveOptions(archive_type=ArchiveType.ZIP, entry_glob='**/*.jpg')
)
```

Pour plus d’options sur la création de jeux de données avec différentes options et à partir de différentes sources, leur inscription et leur examen dans l’interface utilisateur Azure Machine Learning, la compréhension de la manière dont la taille des données influence la capacité de calcul et la gestion des versions, consultez [Créer des jeux de données Azure Machine Learning](how-to-create-register-datasets.md). 

### <a name="pass-datasets-to-your-script"></a>Passer des jeux de données à votre script

Pour passer le chemin d’accès du jeu de données à votre script, utilisez la méthode `as_named_input()` de l’objet `Dataset`. Vous pouvez passer l’objet `DatasetConsumptionConfig` résultant à votre script en tant qu’argument ou, en utilisant l’argument `inputs` à votre script de pipeline, vous pouvez récupérer le jeu de données à l’aide de `Run.get_context().input_datasets[]`.

Une fois que vous avez créé une entrée nommée, vous pouvez choisir son mode d’accès : `as_mount()` ou `as_download()`. Si votre script traite tous les fichiers de votre jeu de données et que le disque de votre ressource de calcul est suffisamment grand pour le jeu de données, le mode d’accès téléchargement est le meilleur choix. Le mode d’accès téléchargement évite la surcharge liée à la diffusion en continu des données au moment de l’exécution. Si votre script accède à un sous-ensemble du jeu de données ou s’il est trop grand pour votre calcul, utilisez le mode d’accès montage. Pour plus d’informations, consultez [Montage/ Téléchargement](./how-to-train-with-datasets.md#mount-vs-download)

Pour passer un jeu de données à votre étape de pipeline :

1. Utilisez `TabularDataset.as_named_input()` ou `FileDataset.as_named_input()` (pas de ’s’ à la fin) pour créer un objet `DatasetConsumptionConfig`
1. Utilisez `as_mount()` ou `as_download()` pour définir le mode d’accès
1. Transmettez les jeux de données à vos étapes de pipeline à l’aide de l'argument `arguments` ou `inputs`

L’extrait de code suivant montre le modèle commun de combinaison de ces étapes dans le constructeur `PythonScriptStep` : 

```python

train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    inputs=[iris_dataset.as_named_input('iris').as_mount()]
)
```

> [!NOTE]
> Vous devez remplacer les valeurs de tous ces arguments (autrement dit, `"train_data"`, `"train.py"`, `cluster` et `iris_dataset`) par vos propres données. L’extrait de code ci-dessus montre simplement la forme de l’appel et ne fait pas partie d’un exemple Microsoft. 

Vous pouvez également utiliser des méthodes telles que `random_split()` et `take_sample()` pour créer plusieurs entrées ou réduire la quantité de données transmises à votre étape de pipeline :

```python
seed = 42 # PRNG seed
smaller_dataset = iris_dataset.take_sample(0.1, seed=seed) # 10%
train, test = smaller_dataset.random_split(percentage=0.8, seed=seed)

train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    inputs=[train.as_named_input('train').as_download(), test.as_named_input('test').as_download()]
)
```

### <a name="access-datasets-within-your-script"></a>Accéder aux jeux de données dans votre script

Les entrées nommées dans votre script d’étape de pipeline sont disponibles en tant que dictionnaire dans l’objet `Run`. Récupérez l’objet actif `Run` à l’aide de `Run.get_context()`, puis récupérez le dictionnaire d’entrées nommées à l’aide de `input_datasets`. Si vous avez passé l’objet `DatasetConsumptionConfig` à l’aide de l’argument `arguments` plutôt que de l’argument `inputs`, accédez aux données à l’aide du code `ArgParser`. Les deux techniques sont illustrées dans l’extrait de code suivant.

```python
# In pipeline definition script:
# Code for demonstration only: It would be very confusing to split datasets between `arguments` and `inputs`
train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    arguments=['--training-folder', train.as_named_input('train').as_download()]
    inputs=[test.as_named_input('test').as_download()]
)

# In pipeline script
parser = argparse.ArgumentParser()
parser.add_argument('--training-folder', type=str, dest='train_folder', help='training data folder mounting point')
args = parser.parse_args()
training_data_folder = args.train_folder

testing_data_folder = Run.get_context().input_datasets['test']
```

La valeur transmise est le chemin d’accès au(x) fichier(s) de jeu de données.

Il est également possible d’accéder directement à un `Dataset` inscrit. Étant donné que les jeux de données inscrits sont persistants et partagés dans un espace de travail, vous pouvez les récupérer directement :

```python
run = Run.get_context()
ws = run.experiment.workspace
ds = Dataset.get_by_name(workspace=ws, name='mnist_opendataset')
```

> [!NOTE]
> Les extraits de code précédents illustrent la forme des appels et ne font pas partie d’un exemple Microsoft. Vous devez remplacer les différents arguments par les valeurs de votre propre projet.

## <a name="use-pipelinedata-for-intermediate-data"></a>Utiliser `PipelineData` pour les données intermédiaires

Tandis que les objets `Dataset` représentent des données persistantes, les objets [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) sont utilisés pour les données temporaires qui sont générées à partir des étapes de pipeline. Étant donné que la durée de vie d’un objet `PipelineData` est plus longue qu’une seule étape de pipeline, vous devez les définir dans le script de définition du pipeline. Lorsque vous créez un objet `PipelineData`, vous devez fournir un nom et un magasin de données où les données seront stockées. Transmettez vos objets `PipelineData` à votre `PythonScriptStep` en utilisant _les deux_ arguments `arguments` et `outputs` :

```python

default_datastore = workspace.get_default_datastore()
dataprep_output = PipelineData("clean_data", datastore=default_datastore)

dataprep_step = PythonScriptStep(
    name="prep_data",
    script_name="dataprep.py",
    compute_target=cluster,
    arguments=["--output-path", dataprep_output]
    inputs=[Dataset.get_by_name(workspace, 'raw_data')],
    outputs=[dataprep_output]
)

```

Vous pouvez choisir de créer votre objet `PipelineData` à l’aide d’un mode d’accès qui fournit un téléchargement immédiat. Dans ce cas, lorsque vous créez votre `PipelineData`, définissez `upload_mode` sur `"upload"` et utilisez l’argument `output_path_on_compute` pour spécifier le chemin d’accès dans lequel vous allez écrire les données :

```python
PipelineData("clean_data", datastore=def_blob_store, output_mode="upload", output_path_on_compute="clean_data_output/")
```

> [!NOTE]
> Les extraits de code précédents illustrent la forme des appels et ne font pas partie d’un exemple Microsoft. Vous devez remplacer les différents arguments par les valeurs de votre propre projet.

> [!TIP]
> Une expérience améliorée pour transmettre des données intermédiaires entre les étapes de pipeline est disponible avec la classe en préversion publique, [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py). Pour obtenir un exemple de code qui utilise `OutputFileDatasetConfig`, consultez l’article expliquant comment [générer un pipeline ML à deux étapes](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/pipeline-with-datasets/pipeline-for-image-classification.ipynb).


### <a name="use-pipelinedata-as-outputs-of-a-training-step"></a>Utiliser `PipelineData` comme sortie d’une étape d’apprentissage
Dans l’étape `PythonScriptStep` de votre pipeline, vous pouvez récupérer les chemins de sortie disponibles à l’aide des arguments du programme. Si cette étape est la première et initialise les données de sortie, vous devez créer le répertoire au chemin d’accès spécifié. Vous pouvez ensuite écrire les fichiers que vous souhaitez inclure dans les `PipelineData`.

```python
parser = argparse.ArgumentParser()
parser.add_argument('--output_path', dest='output_path', required=True)
args = parser.parse_args()
# Make directory for file
os.makedirs(os.path.dirname(args.output_path), exist_ok=True)
with open(args.output_path, 'w') as f:
    f.write("Step 1's output")
```

Si vous avez créé votre `PipelineData` avec l’argument `is_directory` défini sur `True`, il suffit d’effectuer l’appel à `os.makedirs()`, puis d’écrire tous les fichiers souhaités dans le chemin d’accès. Pour plus d’informations, consultez la documentation de référence de [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py).


### <a name="read-pipelinedata-as-inputs-to-non-initial-steps"></a>Lire les `PipelineData` en tant qu’entrées pour les étapes non initiales

Une fois que l’étape de pipeline initiale a écrit des données dans le chemin d’accès `PipelineData` et qu’elle devient une sortie de cette étape initiale, elle peut être utilisée comme entrée d’une étape ultérieure :

```python
step1_output_data = PipelineData("processed_data", datastore=def_blob_store, output_mode="upload")
# get adls gen 2 datastore already registered with the workspace
datastore = workspace.datastores['my_adlsgen2']

step1 = PythonScriptStep(
    name="generate_data",
    script_name="step1.py",
    runconfig = aml_run_config,
    arguments = ["--output_path", step1_output_data],
    inputs=[],
    outputs=[step1_output_data]
)

step2 = PythonScriptStep(
    name="read_pipeline_data",
    script_name="step2.py",
    compute_target=compute,
    runconfig = aml_run_config,
    arguments = ["--pd", step1_output_data],
    inputs=[step1_output_data]
)
pipeline = Pipeline(workspace=ws, steps=[step1, step2])
```

La valeur d’une entrée `PipelineData` est le chemin d’accès à la sortie précédente. 

> [!NOTE]
> Les extraits de code précédents illustrent la forme des appels et ne font pas partie d’un exemple Microsoft. Vous devez remplacer les différents arguments par les valeurs de votre propre projet.

> [!TIP]
> Une expérience améliorée pour transmettre des données intermédiaires entre les étapes de pipeline est disponible avec la classe en préversion publique, [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py). Pour obtenir un exemple de code qui utilise `OutputFileDatasetConfig`, consultez l’article expliquant comment [générer un pipeline ML à deux étapes](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/pipeline-with-datasets/pipeline-for-image-classification.ipynb).

Si, comme indiqué précédemment, la première étape a écrit un seul fichier, sa consommation peut ressembler à ceci : 

```python
parser = argparse.ArgumentParser()
parser.add_argument('--pd', dest='pd', required=True)
args = parser.parse_args()
with open(args.pd) as f:
    print(f.read())
```

## <a name="convert-pipelinedata-objects-to-datasets"></a>Convertir les objets `PipelineData` en `Dataset`

Si vous souhaitez rendre votre objet `PipelineData` disponible plus longtemps que la durée d’une exécution, utilisez sa fonction `as_dataset()` pour le convertir en `Dataset`. Vous pouvez ensuite inscrire le `Dataset`, ce qui en fait un citoyen de première classe dans votre espace de travail. Étant donné que votre objet `PipelineData` aura un chemin d’accès différent à chaque exécution du pipeline, il est fortement recommandé de définir `create_new_version` sur `True` lors de l’inscription d’un `Dataset` créé à partir d’un objet `PipelineData`.

```python
step1_output_ds = step1_output_data.as_dataset()
step1_output_ds.register(name="processed_data", create_new_version=True)

```
> [!TIP]
> Une expérience améliorée pour la conservation de vos données intermédiaires en dehors de vos exécutions de pipeline est disponible avec la classe en préversion publique, [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py). Pour obtenir un exemple de code qui utilise `OutputFileDatasetConfig`, consultez l’article expliquant comment [générer un pipeline ML à deux étapes](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/pipeline-with-datasets/pipeline-for-image-classification.ipynb).

## <a name="next-steps"></a>Étapes suivantes

* [Créer un jeu de données Azure Machine Learning](how-to-create-register-datasets.md)
* [Créer et exécuter des pipelines de machine learning avec le kit SDK Azure Machine Learning](how-to-create-your-first-pipeline.md)