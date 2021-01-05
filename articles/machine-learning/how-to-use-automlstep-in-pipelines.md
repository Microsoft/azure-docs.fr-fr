---
title: Utiliser le ML automatisé dans des pipelines ML
titleSuffix: Azure Machine Learning
description: AutoMLStep vous permet d’utiliser le Machine Learning automatisé dans vos pipelines.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
manager: cgronlun
ms.date: 08/26/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, automl
ms.openlocfilehash: 4cbe43f224ddf349db6b182feb3a717bb2bfd32e
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93358828"
---
# <a name="use-automated-ml-in-an-azure-machine-learning-pipeline-in-python"></a>Utiliser le ML automatisé dans un pipeline Azure Machine Learning dans Python


La fonctionnalité de ML automatisé d’Azure Machine Learning vous permet de découvrir des modèles très performants sans devoir réimplémenter chaque approche possible. En les associant à des pipelines Azure Machine Learning, vous pouvez créer des flux de travail déployables, capables de découvrir rapidement l’algorithme le plus approprié pour vos données. Cet article montre comment associer efficacement une étape de préparation des données à une étape de ML automatisé. Le ML automatisé peut découvrir rapidement l’algorithme qui fonctionne le mieux pour vos données, tout en vous mettant sur la voie des opérations de Machine Learning et de l’implémentation du cycle de vie du modèle avec des pipelines.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.

* Un espace de travail Azure Machine Learning. Consultez [Créer un espace de travail Microsoft Azure Machine Learning](how-to-manage-workspace.md).  

* Une connaissances des équipements et du Kit de développement logiciel (SDK) destinés au [Machine Learning automatisé](concept-automated-ml.md) et aux [pipelines de Machine Learning](concept-ml-pipelines.md) d’Azure.

## <a name="review-automated-mls-central-classes"></a>Examiner les classes centrales du ML automatisé

Le ML automatisé dans un pipeline sont représentés par un objet `AutoMLStep`. La classe `AutoMLStep` est une sous-classe de la classe `PipelineStep`. Un graphique d’objets `PipelineStep` définit un `Pipeline`.

Il existe plusieurs sous-classes de la classe `PipelineStep`. En plus de l’objet `AutoMLStep`, cet article présente un objet `PythonScriptStep` pour la préparation des données et un autre pour l’inscription du modèle.

La méthode privilégiée pour déplacer initialement des données _vers_ un pipeline ML consiste à utiliser des objets `Dataset`. Pour déplacer des données _entre_ étapes, la méthode privilégiée consiste à utiliser des objets `PipelineData`. Pour être utilisé avec un objet `AutoMLStep`, l’objet `PipelineData` doit être transformé en objet `PipelineOutputTabularDataset`. Pour plus d’informations, consultez [Données d’entrée et de sortie des pipelines ML](how-to-move-data-in-out-of-pipelines.md).


> [!TIP]
> Une expérience améliorée pour transmettre des données temporaires entre les étapes de pipeline est disponible avec les classes en préversion publique, [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py) et [`OutputTabularDatasetConfig`](/python/api/azureml-core/azureml.data.output_dataset_config.outputtabulardatasetconfig?preserve-view=true&view=azure-ml-py).  Ces classes sont des fonctionnalités d’évaluation [expérimentales](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#&preserve-view=truestable-vs-experimental) susceptibles d’évoluer à tout moment.

L’objet `AutoMLStep` est configuré via un objet `AutoMLConfig`. `AutoMLConfig` est une classe flexible, comme indiqué dans [Configurer des expériences ML automatisés dans Python](./how-to-configure-auto-train.md#configure-your-experiment-settings). 

Un `Pipeline` s’exécute dans une `Experiment`. Le pipeline `Run` possède, pour chaque étape, une `StepRun` enfant. Les sorties de la `StepRun` enfant de ML automatisé sont les métriques d’apprentissage et le modèle le plus performant.

Pour concrétiser les choses, cet article crée un pipeline simple pour une tâche de classification. La tâche prédit la survie du Titanic. Nous n’allons pas aborder pas les données ou la tâche proprement dites, mais uniquement leur transmission.

## <a name="get-started"></a>Bien démarrer

### <a name="retrieve-initial-dataset"></a>Récupérer le jeu de données initial

Souvent, un workflow ML démarre avec des données de base préexistantes. Il s’agit d’un bon scénario pour un jeu de données inscrit. Les jeux de données sont visibles dans l’espace de travail, prennent en charge le contrôle de version et peuvent être explorés de manière interactive. Il existe de nombreuses façons de créer et de remplir un jeu de données, comme indiqué dans [Créer des jeux de données Azure Machine Learning](how-to-create-register-datasets.md). Étant donné que nous allons utiliser le Kit de développement logiciel (SDK) Python pour créer notre pipeline, utilisez le Kit de développement logiciel (SDK) pour télécharger les données de base et les inscrire sous le nom « titanic_ds ».

```python
from azureml.core import Workspace, Dataset

ws = Workspace.from_config()
if not 'titanic_ds' in ws.datasets.keys() :
    # create a TabularDataset from Titanic training data
    web_paths = ['https://dprepdata.blob.core.windows.net/demo/Titanic.csv',
                 'https://dprepdata.blob.core.windows.net/demo/Titanic2.csv']
    titanic_ds = Dataset.Tabular.from_delimited_files(path=web_paths)

    titanic_ds.register(workspace = ws,
                                     name = 'titanic_ds',
                                     description = 'Titanic baseline data',
                                     create_new_version = True)

titanic_ds = Dataset.get_by_name(ws, 'titanic_ds')
```

Le code commence par se connecter à l’espace de travail Azure Machine Learning défini dans le fichier **config.json** (pour une explication, consultez [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace). S’il n’existe pas de jeu de données nommé `'titanic_ds'` inscrit, le code en crée un. Le code télécharge les données CSV à partir du Web, les utilise pour instancier un `TabularDataset`, puis inscrit le jeu de données auprès de l’espace de travail. Enfin, la fonction `Dataset.get_by_name()` affecte le `Dataset` au jeu de données `titanic_ds`. 

### <a name="configure-your-storage-and-compute-target"></a>Configurer votre stockage et votre cible de calcul

Les ressources supplémentaires dont le pipeline a besoin sont un espace de stockage et, en général, des ressources de calcul Azure Machine Learning. 

```python
from azureml.core import Datastore
from azureml.core.compute import AmlCompute, ComputeTarget

datastore = ws.get_default_datastore()

compute_name = 'cpu-cluster'
if not compute_name in ws.compute_targets :
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                min_nodes=0,
                                                                max_nodes=1)
    compute_target = ComputeTarget.create(ws, compute_name, provisioning_config)

    compute_target.wait_for_completion(
        show_output=True, min_node_count=None, timeout_in_minutes=20)

    # Show the result
    print(compute_target.get_status().serialize())

compute_target = ws.compute_targets[compute_name]
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

Les données intermédiaires entre l’étape de préparation des données et l’étape de ML automatisé peuvent être stockées dans le magasin de données par défaut de l’espace de travail. Il nous suffit donc d’appeler la commande `get_default_datastore()` sur l’objet `Workspace`. 

Après cela, le code vérifie si la cible de calcul AML `'cpu-cluster'` existe déjà. Si ce n’est pas le cas, nous spécifions que nous souhaitons une petite cible de calcul basé sur l’UC. Si vous envisagez d’utiliser des fonctionnalités de Deep Learning du ML automatisé (par exemple, la caractérisation de texte avec prise en charge du DNN), vous devez choisir un dispositif de calcul avec prise en charge de GPU fort, comme décrit dans [Tailles de machine virtuelle à GPU optimisé](../virtual-machines/sizes-gpu.md). 

Le code se bloque jusqu’à ce que la cible soit approvisionnée, puis imprime des détails sur la cible de calcul fraîchement créée. Enfin, la cible de calcul nommée est récupérée à partir de l’espace de travail et attribuée à `compute_target`. 

### <a name="configure-the-training-run"></a>Configurer l’exécution de l’apprentissage

L’étape suivante consiste à s’assurer que l’exécution de l’apprentissage à distance englobe toutes les dépendances qu’exigent les étapes d’apprentissage. Les dépendances et le contexte du runtime sont définis en créant et en configurant un objet `RunConfiguration`. 

```python
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core import Environment 

aml_run_config = RunConfiguration()
# Use just-specified compute target ("cpu-cluster")
aml_run_config.target = compute_target

USE_CURATED_ENV = True
if USE_CURATED_ENV :
    curated_environment = Environment.get(workspace=ws, name="AzureML-Tutorial")
    aml_run_config.environment = curated_environment
else:
    aml_run_config.environment.python.user_managed_dependencies = False
    
    # Add some packages relied on by data prep step
    aml_run_config.environment.python.conda_dependencies = CondaDependencies.create(
        conda_packages=['pandas','scikit-learn'], 
        pip_packages=['azureml-sdk[automl]', 'azureml-dataprep[fuse,pandas]'], 
        pin_sdk_version=False)
```

Le code ci-dessus présente deux options pour la gestion des dépendances. Comme présenté, avec `USE_CURATED_ENV = True`, la configuration est basée sur un environnement organisé. Les environnements organisés sont préparés avec des bibliothèques interdépendantes communes, et peuvent être sensiblement plus rapides à mettre en ligne. Les environnements organisés comportent des images Docker prédéfinies dans le [Registre de conteneurs Microsoft](https://hub.docker.com/publishers/microsoftowner). Le chemin emprunté si vous remplacez `USE_CURATED_ENV` par `False` affiche le modèle pour la définition explicite de vos dépendances. Dans ce scénario, une nouvelle image Docker personnalisée sera créée et inscrite dans un Azure Container Registry au sein de votre groupe de ressources (consultez [Présentation des registres de conteneurs Docker privés dans Azure](../container-registry/container-registry-intro.md)). La création et l’inscription de cette image peuvent prendre du temps. 

## <a name="prepare-data-for-automated-machine-learning"></a>Préparer les données pour le Machine Learning automatisé

### <a name="write-the-data-preparation-code"></a>Écrire le code de préparation des données

Le jeu de données de base Titanic se compose d’un mélange de chiffres et de texte, où certaines valeurs sont manquantes. Pour le préparer au Machine Learning automatisé, l’étape du pipeline de préparation des données doit effectuer les opérations suivantes :

- Renseigner les valeurs manquantes avec des données aléatoires ou une catégorie reflétant leur caractère « inconnu ».
- Transformer les données catégorielles en entiers.
- Supprimer les colonnes que nous n’avons pas l’intention d’utiliser.
- Fractionner les données en jeux d’apprentissage et de test.
- Écrire les données transformées dans l’un ou l’autre
    - des chemins de sortie `PipelineData`.

```python
%%writefile dataprep.py
from azureml.core import Run

import pandas as pd 
import numpy as np 
import pyarrow as pa
import pyarrow.parquet as pq
import argparse

RANDOM_SEED=42

def prepare_age(df):
    # Fill in missing Age values from distribution of present Age values 
    mean = df["Age"].mean()
    std = df["Age"].std()
    is_null = df["Age"].isnull().sum()
    # compute enough (== is_null().sum()) random numbers between the mean, std
    rand_age = np.random.randint(mean - std, mean + std, size = is_null)
    # fill NaN values in Age column with random values generated
    age_slice = df["Age"].copy()
    age_slice[np.isnan(age_slice)] = rand_age
    df["Age"] = age_slice
    df["Age"] = df["Age"].astype(int)
    
    # Quantize age into 5 classes
    df['Age_Group'] = pd.qcut(df['Age'],5, labels=False)
    df.drop(['Age'], axis=1, inplace=True)
    return df

def prepare_fare(df):
    df['Fare'].fillna(0, inplace=True)
    df['Fare_Group'] = pd.qcut(df['Fare'],5,labels=False)
    df.drop(['Fare'], axis=1, inplace=True)
    return df 

def prepare_genders(df):
    genders = {"male": 0, "female": 1, "unknown": 2}
    df['Sex'] = df['Sex'].map(genders)
    df['Sex'].fillna(2, inplace=True)
    df['Sex'] = df['Sex'].astype(int)
    return df

def prepare_embarked(df):
    df['Embarked'].replace('', 'U', inplace=True)
    df['Embarked'].fillna('U', inplace=True)
    ports = {"S": 0, "C": 1, "Q": 2, "U": 3}
    df['Embarked'] = df['Embarked'].map(ports)
    return df
    
parser = argparse.ArgumentParser()
parser.add_argument('--output_path', dest='output_path', required=True)
args = parser.parse_args()
    
titanic_ds = Run.get_context().input_datasets['titanic_ds']
df = titanic_ds.to_pandas_dataframe().drop(['PassengerId', 'Name', 'Ticket', 'Cabin'], axis=1)
df = prepare_embarked(prepare_genders(prepare_fare(prepare_age(df))))

os.makedirs(os.path.dirname(args.output_path), exist_ok=True)
pq.write_table(pa.Table.from_pandas(df), args.output_path)

print(f"Wrote test to {args.output_path} and train to {args.output_path}")
```

L’extrait de code ci-dessus est un exemple complet, mais minimal, de préparation pour les données du Titanic. L’extrait de code commence par une « commande magique » Jupyter pour envoyer le code à un fichier. Si vous n’utilisez pas de bloc-notes Jupyter, supprimez cette ligne et créez le fichier manuellement.

Les différentes fonctions de la commande `prepare_` dans l’extrait de code ci-dessus modifient la colonne appropriée dans le jeu de données d’entrée. Ces fonctions opèrent sur les données une fois celles-ci converties en un objet `DataFrame` Pandas. Dans chaque cas, les données manquantes sont renseignées à l’aide de données aléatoires représentatives ou de données catégorielles reflétant leur caractère « inconnu ». Les données catégorielles basées sur du texte sont mappées à des entiers. Les colonnes superflues sont remplacées ou supprimées. 

Une fois que le code a défini les fonctions de préparation des données, il analyse l’argument d’entrée qui est le chemin d’accès de l’emplacement dans lequel nous voulons écrire nos données (ces valeurs seront déterminées par des `PipelineData` qui seront abordés à l’étape suivante). Le code récupère le `Dataset` `'titanic_cs'` inscrit, le convertit en un objet `DataFrame` Pandas et appelle les diverses fonctions de préparation des données. 

Étant donné que le `output_path` est complet, la fonction `os.makedirs()` est utilisée pour préparer la structure du répertoire. À ce stade, vous pourriez utiliser la fonction `DataFrame.to_csv()` pour écrire les données de sortie, mais des fichiers Parquet sont plus efficaces. Cette efficacité est probablement sans grande importance pour un jeu de données d’aussi petite taille, mais l’utilisation des fonctions `from_pandas()` et `write_table()` du package **PyArrow** ne nécessite que quelques frappes de plus que celle de la fonction `to_csv()`.

Les fichiers Parquet sont pris en charge en mode natif par l’étape de ML automatisé décrite ci-dessous. Leur utilisation ne nécessite donc aucun traitement particulier. 

### <a name="write-the-data-preparation-pipeline-step-pythonscriptstep"></a>Écrire l’étape du pipeline de préparation des données (`PythonScriptStep`)

Le code de préparation des données décrit ci-dessus doit être associé à un objet `PythonScripStep` à utiliser avec un pipeline. Le chemin d’accès de l’emplacement dans lequel est écrite la sortie de préparation des données Parquet est généré par un objet `PipelineData`. Les ressources préparées précédemment, telles que la `ComputeTarget`, la `RunConfig` et le `'titanic_ds' Dataset` sont utilisées pour élaborer la spécification.

Utilisateurs de PipelineData
```python
from azureml.pipeline.core import PipelineData

from azureml.pipeline.steps import PythonScriptStep
prepped_data_path = PipelineData("titanic_train", datastore).as_dataset()

dataprep_step = PythonScriptStep(
    name="dataprep", 
    script_name="dataprep.py", 
    compute_target=compute_target, 
    runconfig=aml_run_config,
    arguments=["--output_path", prepped_data_path],
    inputs=[titanic_ds.as_named_input("titanic_ds")],
    outputs=[prepped_data_path],
    allow_reuse=True
)
```
L’objet `prepped_data_path` est du type `PipelineOutputFileDataset`. Notez qu’il est spécifié dans les arguments `arguments` et `outputs`. En examinant l’étape précédente, vous verrez que, dans le code de préparation des données, la valeur de l’argument `'--output_path'` est le chemin d’accès de l’emplacement dans lequel le fichier Parquet a été écrit. 

> [!TIP]
> Une expérience améliorée pour transmettre des données intermédiaires entre les étapes de pipeline est disponible avec la classe en préversion publique, [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py). Pour obtenir un exemple de code qui utilise la classe `OutputFileDatasetConfig`, consultez l’article expliquant comment [générer un pipeline ML à deux étapes](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/pipeline-with-datasets/pipeline-for-image-classification.ipynb).

## <a name="train-with-automlstep"></a>Effectuer l’apprentissage avec AutoMLStep

La configuration d’une étape de pipeline ML automatisé est effectuée avec la classe `AutoMLConfig`. Cette classe flexible est décrite dans [Configurer des expériences de ML automatisé en Python](./how-to-configure-auto-train.md). L’entrée et la sortie des données sont les seuls aspects de la configuration qui nécessitent une attention particulière dans un pipeline ML. L’entrée et la sortie pour la classe `AutoMLConfig` dans les pipelines sont décrites en détail ci-dessous. Au-delà des données, un avantage des pipelines ML est la possibilité d’utiliser des cibles de calcul différentes pour les différentes étapes. Vous pouvez choisir d’utiliser une `ComputeTarget` plus puissante uniquement pour le processus de ML automatisé. Cette opération est aussi simple que l’affectation d’une `RunConfiguration` plus puissante au paramètre `run_configuration` de l’objet `AutoMLConfig`.

### <a name="send-data-to-automlstep"></a>Envoyer des données à `AutoMLStep`

Dans un pipeline ML, les données d’entrée doivent être un objet `Dataset`. La méthode la plus performante consiste à fournir les données d’entrée sous la forme d’objets `PipelineOutputTabularDataset`. Vous créez un objet de ce type avec l’une des commandes `parse_parquet_files()` ou `parse_delimited_files()` sur un `PipelineOutputFileDataset`, tel que l’objet `prepped_data_path`.

```python
# type(prepped_data_path) == PipelineOutputFileDataset
# type(prepped_data) == PipelineOutputTabularDataset
prepped_data = prepped_data_path.parse_parquet_files(file_extension=None)
```

L’extrait de code ci-dessus crée un `PipelineOutputTabularDataset` hautement performant à partir de la sortie `PipelineOutputFileDataset` de l’étape de préparation des données.

> [!TIP]
> La classe en préversion publique [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py) contient la méthode [read_delimited_files()](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py#&preserve-view=trueread-delimited-files-include-path-false--separator------header--promoteheadersbehavior-all-files-have-same-headers--3---partition-format-none--path-glob-none--set-column-types-none-) qui convertit un `OutputFileDatasetConfig` en [`OutputTabularDatasetConfig`](/python/api/azureml-core/azureml.data.output_dataset_config.outputtabulardatasetconfig?preserve-view=true&view=azure-ml-py) à des fins d’utilisation dans les exécutions AutoML.

Une autre option consiste à utiliser des objets `Dataset` inscrits dans l’espace de travail :

```python
prepped_data = Dataset.get_by_name(ws, 'Data_prepared')
```

Comparaison des deux techniques :

| Technique | Avantages et inconvénients | 
|-|-|
|`PipelineOutputTabularDataset`| Performances accrues | 
|| Itinéraire naturel à partir des `PipelineData` | 
|| Les données ne sont pas conservées après l’exécution du pipeline |
|| [Bloc-notes présentant la technique`PipelineOutputTabularDataset`](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb) |
| `Dataset` inscrit | Moins bonnes performances |
| | Peut être généré de nombreuses façons | 
| | Les données sont conservées et visibles dans l’espace de travail |
| | [Bloc-notes présentant la technique `Dataset` inscrite](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/continuous-retraining/auto-ml-continuous-retraining.ipynb)


### <a name="specify-automated-ml-outputs"></a>Spécifier les sorties du ML automatisé

Les sorties de l’étape `AutoMLStep` sont les scores de métrique finaux du modèle d’exécution plus performant et ce modèle lui-même. Pour utiliser ces sorties dans d’autres étapes du pipeline, préparez les objets `PipelineData` à les recevoir.

```python

from azureml.pipeline.core import TrainingOutput

metrics_data = PipelineData(name='metrics_data',
                            datastore=datastore,
                            pipeline_output_name='metrics_output',
                            training_output=TrainingOutput(type='Metrics'))

model_data = PipelineData(name='best_model_data',
                          datastore=datastore,
                          pipeline_output_name='model_output',
                          training_output=TrainingOutput(type='Model'))
```

L’extrait de code ci-dessus crée les deux objets `PipelineData` pour les métriques et la sortie du modèle. Chaque objet est nommé, affecté au magasin de données par défaut récupéré précédemment et associé au `type` et à la `TrainingOutput` spécifiques de l’étape `AutoMLStep`. Étant donné que nous attribuons un `pipeline_output_name` à ces objets `PipelineData`, leurs valeurs sont accessibles non seulement à partir de l’étape de pipeline particulière, mais à partir du pipeline dans son ensemble, comme expliqué ci-dessous dans la section « Examiner les résultats du pipeline ». 

### <a name="configure-and-create-the-automated-ml-pipeline-step"></a>Configurer et créer l’étape de pipeline ML automatisé

Une fois les entrées et les sorties définies, il est temps de créer le constructeur `AutoMLConfig` et l’étape `AutoMLStep`. Les détails de la configuration dépendent de votre tâche, comme décrit dans [Configurer des expériences de ML automatisé en Python](./how-to-configure-auto-train.md). Pour la tâche de classification de survie du Titanic, l’extrait de code suivant illustre une configuration simple.

```python
from azureml.train.automl import AutoMLConfig
from azureml.pipeline.steps import AutoMLStep

# Change iterations to a reasonable number (50) to get better accuracy
automl_settings = {
    "iteration_timeout_minutes" : 10,
    "iterations" : 2,
    "experiment_timeout_hours" : 0.25,
    "primary_metric" : 'AUC_weighted'
}

automl_config = AutoMLConfig(task = 'classification',
                             path = '.',
                             debug_log = 'automated_ml_errors.log',
                             compute_target = compute_target,
                             run_configuration = aml_run_config,
                             featurization = 'auto',
                             training_data = prepped_data,
                             label_column_name = 'Survived',
                             **automl_settings)

train_step = AutoMLStep(name='AutoML_Classification',
    automl_config=automl_config,
    outputs=[metrics_data,model_data],
    enable_default_model_output=False,
    enable_default_metrics_output=False,
    allow_reuse=True)
```
L’extrait de code montre un idiome couramment utilisé avec le constructeur `AutoMLConfig`. Les arguments relativement fluides (de type hyperparamètres) sont spécifiés dans un dictionnaire distinct, tandis que les valeurs moins sujettes à modification sont spécifiées directement dans le constructeur `AutoMLConfig`. Dans ce cas, les `automl_settings` spécifient une exécution brève qui prend fin après 2 itérations ou au maximum 15 minutes.

Le dictionnaire `automl_settings` est transmis au constructeur `AutoMLConfig` sous la forme d’arguments kwargs. Les autres paramètres ne sont pas complexes :

- La valeur de `task` est définie sur `classification` pour cet exemple. Les autres valeurs valides sont `regression` et `forecasting`.
- Les paramètres `path` et `debug_log` décrivent respectivement le chemin d’accès au projet et un fichier local dans lequel les informations de débogage seront consignées. 
- La cible de calcul `compute_target` est la valeur `compute_target` précédemment définie qui, dans cet exemple, est un ordinateur basé sur une UC peu onéreux. Si vous utilisez des fonctionnalités de Deep Learning du ML automatisé, vous pouvez modifier la cible de calcul pour qu’elle soit basée sur une GPU.
- `featurization` est défini sur `auto`. Pour plus d’informations, consultez la section [Caractérisation de données](./how-to-configure-auto-train.md#data-featurization) du document de configuration de ML automatisé. 
- `label_column_name` indique la colonne dont la prédiction nous intéresse 
- Les `training_data` sont définies sur les objets `PipelineOutputTabularDataset` constitués des sorties de l’étape de préparation des données 

L’étape `AutoMLStep` elle-même prend la `AutoMLConfig` et a, en sortie, les objets `PipelineData` créés pour contenir les métriques et les données du modèle. 

>[!Important]
> Vous devez définir `enable_default_model_output` et `enable_default_metrics_output` sur `True` uniquement si vous utilisez `AutoMLStepRun`.

Dans cet exemple, le processus de ML automatisé effectue des validations croisées sur les `training_data`. Vous pouvez contrôler le nombre de validations croisées avec l’argument `n_cross_validations`. Si vous avez déjà fractionné vos données d’apprentissage dans le cadre de vos étapes de préparation des données, vous pouvez définir `validation_data` sur son propre `Dataset`.

`X` est parfois utilisé pour les fonctionnalités de données, et `y` pour les étiquettes de données. Cette technique est déconseillée et vous devriez utiliser `training_data` pour l’entrée. 

## <a name="register-the-model-generated-by-automated-ml"></a>Inscrire le modèle généré par ML automatisé 

La dernière étape d’un pipeline ML simple consiste à inscrire le modèle créé. L’ajout du modèle au registre de modèles de l’espace de travail a pour effet que le modèle devient disponible sur le portail et que sa version peut être contrôlée. Pour inscrire le modèle, écrivez une autre `PythonScriptStep` qui prend la sortie de `model_data` de l’étape `AutoMLStep`.

### <a name="write-the-code-to-register-the-model"></a>Écrire le code pour inscrire le modèle

Un modèle est inscrit dans un `Workspace`. Vous êtes probablement familiarisé avec l’utilisation de la fonction `Workspace.from_config()` pour vous connecter à votre espace de travail sur votre ordinateur local, mais il existe une autre façon d’obtenir l’espace de travail à partir d’un pipeline ML en cours d’exécution. La commande `Run.get_context()` récupère l’objet `Run` actif. Cet objet `run` donne accès à de nombreux objets importants, dont le `Workspace` utilisé ici.

```python
%%writefile register_model.py
from azureml.core.model import Model, Dataset
from azureml.core.run import Run, _OfflineRun
from azureml.core import Workspace
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--model_name", required=True)
parser.add_argument("--model_path", required=True)
args = parser.parse_args()

print(f"model_name : {args.model_name}")
print(f"model_path: {args.model_path}")

run = Run.get_context()
ws = Workspace.from_config() if type(run) == _OfflineRun else run.experiment.workspace

model = Model.register(workspace=ws,
                       model_path=args.model_path,
                       model_name=args.model_name)

print("Registered version {0} of model {1}".format(model.version, model.name))
```

### <a name="write-the-pythonscriptstep-code"></a>Écrire le code PythonScriptStep

L’étape `PythonScriptStep` d’inscription du modèle utilise un `PipelineParameter` pour l’un de ses arguments. Les paramètres de pipeline sont des arguments pour des pipelines, qui peuvent être facilement définis au moment de l’envoi de l’exécution. Une fois déclarés, ils sont transmis en tant qu’arguments normaux. 

```python

from azureml.pipeline.core.graph import PipelineParameter

# The model name with which to register the trained model in the workspace.
model_name = PipelineParameter("model_name", default_value="TitanicSurvivalInitial")

register_step = PythonScriptStep(script_name="register_model.py",
                                       name="register_model",
                                       allow_reuse=False,
                                       arguments=["--model_name", model_name, "--model_path", model_data],
                                       inputs=[model_data],
                                       compute_target=compute_target,
                                       runconfig=aml_run_config)
```

## <a name="create-and-run-your-automated-ml-pipeline"></a>Créer et exécuter votre pipeline ML automatisé

La création et l’exécution d’un pipeline contenant une étape `AutoMLStep` ne présentent aucune différence par rapport à un pipeline normal. 

```python
from azureml.pipeline.core import Pipeline
from azureml.core import Experiment

pipeline = Pipeline(ws, [dataprep_step, train_step, register_step])

experiment = Experiment(workspace=ws, name='titanic_automl')

run = experiment.submit(pipeline, show_output=True)
run.wait_for_completion()
```

Le code ci-dessus combine les étapes de préparation des données, de ML automatisé et d’inscription de modèle dans un objet `Pipeline`. Il crée ensuite un objet `Experiment`. Le constructeur `Experiment` récupère l’expérience nommée si elle existe, ou la crée si nécessaire. Il envoie le `Pipeline` à l’`Experiment`, créant ainsi un objet `Run` qui exécutera le pipeline de manière asynchrone. La fonction `wait_for_completion()` effectue un blocage jusqu’à ce que l’exécution soit terminée.

### <a name="examine-pipeline-results"></a>Examiner les résultats du pipeline 

Une fois le cycle `run` accompli, vous pouvez récupérer les objets `PipelineData` auxquels un `pipeline_output_name` a été attribué. Vous pouvez télécharger les résultats et les charger en vue d’un traitement ultérieur.  

```python
metrics_output_port = run.get_pipeline_output('metrics_output')
model_output_port = run.get_pipeline_output('model_output')

metrics_output_port.download('.', show_progress=True)
model_output_port.download('.', show_progress=True)
```

Les fichiers téléchargés sont écrits dans le sous-répertoire `azureml/{run.id}/`. Le fichier de métriques au format JSON peut être converti en tramedonnées (dataframe) Pandas pour examen.

Pour le traitement local, il se peut que vous deviez installer des packages appropriés, tels que Pandas, Pickle, le Kit de développement logiciel (SDK) AzureML, etc. Pour cet exemple, il est probable que le meilleur modèle trouvé par le ML automatisé dépend de XGBoost.

```bash
!pip install xgboost==0.90
```

```python
import pandas as pd
import json

metrics_filename = metrics_output._path_on_datastore
# metrics_filename = path to downloaded file
with open(metrics_filename) as f:
   metrics_output_result = f.read()
   
deserialized_metrics_output = json.loads(metrics_output_result)
df = pd.DataFrame(deserialized_metrics_output)
df
```

L’extrait de code ci-dessus montre le fichier de métriques chargé à partir de son emplacement vers le magasin de données Azure. Vous pouvez également le charger à partir du fichier téléchargé, comme indiqué dans le commentaire. Une fois que vous l’avez désérialisé et converti en DataFrame Pandas, vous pouvez voir des métriques détaillées pour chacune des itérations de l’étape de ML automatisé.

Vous pouvez désérialiser le fichier du modèle dans un objet `Model` que vous pouvez utiliser à des fins d’inférence, d’analyse plus approfondie des métriques, etc. 

```python
import pickle

model_filename = model_output._path_on_datastore
# model_filename = path to downloaded file

with open(model_filename, "rb" ) as f:
    best_model = pickle.load(f)

# ... inferencing code not shown ...
```

Pour plus d’informations sur le chargement et l’utilisation des modèles existants, consultez [Utiliser un modèle existant avec Azure Machine Learning](how-to-deploy-existing-model.md).

### <a name="download-the-results-of-an-automated-ml-run"></a>Télécharger les résultats d’un cycle d’exécution de ML automatisé

Si vous avez suivi la procédure décrite dans l’article, vous disposez d’un objet `run` instancié. Autrement, vous pouvez aussi récupérer des objets `Run` tout prêts à partir du `Workspace` à l’aide d’un objet `Experiment`.

L’espace de travail contient un enregistrement complet de l’ensemble de vos expériences et exécution. Vous pouvez soit utiliser le portail pour rechercher et télécharger les sorties des expériences, soit utiliser du code. Pour accéder aux enregistrements d’une exécution historique, utilisez Azure Machine Learning pour rechercher l’ID de l’exécution qui vous intéresse. Cet ID vous permet de choisir le cycle `run` au moyen du `Workspace` et de l’`Experiment`.

```python
# Retrieved from Azure Machine Learning web UI
run_id = 'aaaaaaaa-bbbb-cccc-dddd-0123456789AB'
experiment = ws.experiments['titanic_automl']
run = next(run for run in ex.get_runs() if run.id == run_id)
```

Vous devez remplacer les chaînes dans le code ci-dessus par les éléments spécifiques de votre exécution historique. L’extrait de code ci-dessus suppose que vous avez affecté `ws` au `Workspace` approprié avec la `from_config()` normale. L’expérience qui vous intéresse est directement récupérée, puis le code trouve le cycle `Run` qui vous intéresse en cherchant une correspondance avec la valeur de `run.id`.

Une fois que vous avez un objet `Run`, vous pouvez télécharger les métriques et le modèle. 

```python
automl_run = next(r for r in run.get_children() if r.name == 'AutoML_Classification')
outputs = automl_run.get_outputs()
metrics = outputs['default_metrics_AutoML_Classification']
model = outputs['default_model_AutoML_Classification']

metrics.get_port_data_reference().download('.')
model.get_port_data_reference().download('.')
```

Chaque objet `Run` contient des objets `StepRun` qui contiennent des informations sur l’exécution de l’étape de pipeline spécifique. Le cycle `run` est recherché dans l’objet `StepRun` pour l’étape `AutoMLStep`. Les métriques et le modèle sont récupérés à l’aide de leurs noms par défaut, qui sont disponibles même si vous ne transmettez pas d’objets `PipelineData` au paramètre `outputs` de l’étape `AutoMLStep`. 

Enfin, les métriques et le modèle réels sont téléchargés sur votre ordinateur local, comme expliqué dans la section « Examiner les résultats du pipeline » ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

- Exécutez ce bloc-notes Jupyter montrant un [exemple complet de ML automatisé dans un pipeline](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb), qui utilise une régression pour prédire des tarifs de courses en taxis
- [Créer des expériences de ML automatisé sans écrire de code](how-to-use-automated-ml-for-ml-models.md)
- Explorez un vaste éventail de [bloc-notes Jupyter illustrant le ML automatisé](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/automated-machine-learning)
- Découvrez l’intégration de votre pipeline dans [MLOps de bout en bout](./concept-model-management-and-deployment.md#automate-the-ml-lifecycle) ou examinez le [référentiel GitHub MLOps](https://github.com/Microsoft/MLOpspython)