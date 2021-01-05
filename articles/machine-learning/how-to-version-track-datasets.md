---
title: Contrôle de version des jeux de données
titleSuffix: Azure Machine Learning
description: Découvrez comment optimiser la version de vos jeux de données et comment le contrôle de version fonctionne avec les pipelines Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sihhu
author: MayMSFT
ms.reviewer: nibaccam
ms.date: 03/09/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: 8c9beaca0fb3ee7881559ffcc955f171bc2ddd7b
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886445"
---
# <a name="version-and-track-datasets-in-experiments"></a>Gérer les versions et suivre des jeux de données dans les expériences

Dans cet article, vous apprendrez à gérer les versions et à suivre les jeux de données Azure Machine Learning pour la reproductibilité. Un contrôle de version d’un jeu de données est un moyen de marquer l’état de vos données, afin de pouvoir appliquer une version spécifique du jeu de données pour des expérimentations ultérieures.

Scénarios de contrôle de version classiques :

* Quand de nouvelles données sont disponibles pour un nouvel entraînement
* Quand vous appliquez différentes approches de préparation des données ou d’ingénierie des caractéristiques

## <a name="prerequisites"></a>Prérequis

Pour ce didacticiel, vous avez besoin des éléments suivants :

- [SDK Azure Machine Learning pour Python installé](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py). Ce kit de développement logiciel (SDK) comprend le package [azureml-datasets](/python/api/azureml-core/azureml.core.dataset?preserve-view=true&view=azure-ml-py).
    
- Un [espace de travail Azure Machine Learning](concept-workspace.md). Récupérez un espace de travail existant en exécutant le code suivant, ou bien [créez un nouvel espace de travail](how-to-manage-workspace.md).

    ```Python
    import azureml.core
    from azureml.core import Workspace
    
    ws = Workspace.from_config()
    ```
- Un [jeu de données Azure Machine Learning](how-to-create-register-datasets.md).

<a name="register"></a>

## <a name="register-and-retrieve-dataset-versions"></a>Inscrire et récupérer des versions de jeux de données

En inscrivant un jeu de données, vous pouvez gérer sa version, le réutiliser et le partager entre les expériences et avec des collègues. Vous pouvez inscrire plusieurs jeux de données sous le même nom et récupérer une version spécifique à partir de son nom et de son numéro de version.

### <a name="register-a-dataset-version"></a>Inscrire une version de jeu de données

Le code suivant inscrit une nouvelle version du jeu de données `titanic_ds` en affectant au paramètre `create_new_version` la valeur `True`. S’il n’existe aucun jeu de données `titanic_ds` enregistré avec l’espace de travail, le code crée un nouveau jeu de données portant le nom `titanic_ds` et définit sa version sur 1.

```Python
titanic_ds = titanic_ds.register(workspace = workspace,
                                 name = 'titanic_ds',
                                 description = 'titanic training data',
                                 create_new_version = True)
```

### <a name="retrieve-a-dataset-by-name"></a>Récupérer un jeu de données par nom

Par défaut, la méthode [get_by_name()](/python/api/azureml-core/azureml.core.dataset.dataset?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-by-name-workspace--name--version--latest--) sur la classe `Dataset` retourne la dernière version du jeu de données inscrit auprès de l’espace de travail. 

Le code suivant obtient la version 1 du jeu de données `titanic_ds`.

```Python
from azureml.core import Dataset
# Get a dataset by name and version number
titanic_ds = Dataset.get_by_name(workspace = workspace,
                                 name = 'titanic_ds', 
                                 version = 1)
```

<a name="best-practice"></a>

## <a name="versioning-best-practice"></a>Meilleures pratiques relatives au contrôle de version

Lorsque vous créez une version d’un jeu de données, vous ne créez *pas* une copie supplémentaire des données avec l’espace de travail. Les jeux de données étant des références aux données de votre service de stockage, vous disposez d’une seule source de vérité, gérée par votre service de stockage.

>[!IMPORTANT]
> Si les données référencées par votre jeu de données sont remplacées ou supprimées, l’appel d’une version spécifique du jeu de données ne permet *pas* d’annuler la modification.

Lorsque vous chargez des données à partir d’un jeu données, le contenu des données actuelles référencées par le jeu de données est toujours chargé. Si vous souhaitez vous assurer que chaque version du jeu de données est reproductible, nous vous recommandons de ne pas modifier le contenu des données référencées par la version du jeu de données. Lorsque de nouvelles données arrivent, enregistrez les nouveaux fichiers de données dans un dossier de données distinct, puis créez une nouvelle version du jeu de données pour inclure les données de ce nouveau dossier.

L’image et l’exemple de code suivants montrent la méthode recommandée pour structurer vos dossiers de données et créer des versions de jeu de données faisant référence à ces dossiers :

![Structure de dossiers](./media/how-to-version-track-datasets/folder-image.png)

```Python
from azureml.core import Dataset

# get the default datastore of the workspace
datastore = workspace.get_default_datastore()

# create & register weather_ds version 1 pointing to all files in the folder of week 27
datastore_path1 = [(datastore, 'Weather/week 27')]
dataset1 = Dataset.File.from_files(path=datastore_path1)
dataset1.register(workspace = workspace,
                  name = 'weather_ds',
                  description = 'weather data in week 27',
                  create_new_version = True)

# create & register weather_ds version 2 pointing to all files in the folder of week 27 and 28
datastore_path2 = [(datastore, 'Weather/week 27'), (datastore, 'Weather/week 28')]
dataset2 = Dataset.File.from_files(path = datastore_path2)
dataset2.register(workspace = workspace,
                  name = 'weather_ds',
                  description = 'weather data in week 27, 28',
                  create_new_version = True)

```

<a name="pipeline"></a>

## <a name="version-an-ml-pipeline-output-dataset"></a>Gérer la version d’un jeu de données de sortie de pipeline ML

Vous pouvez utiliser un jeu de données comme entrée et sortie de chaque étape de [pipeline ML](concept-ml-pipelines.md). Lorsque vous réexécutez des pipelines, la sortie de chaque étape de pipeline est inscrite en tant que nouvelle version du jeu de données.

Les pipelines ML renseignent la sortie de chaque étape dans un nouveau dossier chaque fois que le pipeline est réexécuté. Ce comportement permet de reproduire les jeux de données de sortie avec version. En savoir plus sur les [jeux de données dans les pipelines](how-to-create-your-first-pipeline.md#steps).

```Python
from azureml.core import Dataset
from azureml.pipeline.steps import PythonScriptStep
from azureml.pipeline.core import Pipeline, PipelineData
from azureml.core. runconfig import CondaDependencies, RunConfiguration

# get input dataset 
input_ds = Dataset.get_by_name(workspace, 'weather_ds')

# register pipeline output as dataset
output_ds = PipelineData('prepared_weather_ds', datastore=datastore).as_dataset()
output_ds = output_ds.register(name='prepared_weather_ds', create_new_version=True)

conda = CondaDependencies.create(
    pip_packages=['azureml-defaults', 'azureml-dataprep[fuse,pandas]'], 
    pin_sdk_version=False)

run_config = RunConfiguration()
run_config.environment.docker.enabled = True
run_config.environment.python.conda_dependencies = conda

# configure pipeline step to use dataset as the input and output
prep_step = PythonScriptStep(script_name="prepare.py",
                             inputs=[input_ds.as_named_input('weather_ds')],
                             outputs=[output_ds],
                             runconfig=run_config,
                             compute_target=compute_target,
                             source_directory=project_folder)
```

<a name="track"></a>

## <a name="track-data-in-your-experiments"></a>Suivre les données dans vos expériences

Azure Machine Learning effectue le suivi de vos données dans votre expérience en tant que jeux de données d’entrée et de sortie.  

Voici les scénarios dans lesquels vos données sont suivies en tant que **jeu de données d’entrée**. 

* En tant qu’`DatasetConsumptionConfig`objet par le biais du paramètre `inputs` ou `arguments` de votre objet `ScriptRunConfig` lors de l’envoi d’une exécution d’expérimentation. 

* Lorsque des méthodes, telles que get_by_name() ou get_by_id(), sont appelées dans votre script. Pour ce scénario, le nom attribué au jeu de données lorsque vous l’avez enregistré dans l’espace de travail est le nom affiché. 

Voici les scénarios dans lesquels vos données sont suivies en tant que **jeu de données de sortie**.  

* Transmettez un objet `OutputFileDatasetConfig` par le biais du paramètre `outputs` ou `arguments` lors de l’envoi d’une exécution d’expérimentation. Les objets `OutputFileDatasetConfig` peuvent également être utilisés pour rendre les données persistantes entre les étapes du pipeline. Consultez [Déplacer des données entre des étapes du pipeline ML](how-to-move-data-in-out-of-pipelines.md).
    > [!TIP]
    > [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig?preserve-view=true&view=azure-ml-py) est une classe de préversion publique qui contient des fonctionnalités d'évaluation [expérimentales](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#&preserve-view=truestable-vs-experimental) susceptibles d’être modifiées à tout moment.

* Inscrivez un jeu de données dans votre script. Pour ce scénario, le nom attribué au jeu de données lorsque vous l’avez enregistré dans l’espace de travail est le nom affiché. Dans l'exemple suivant, `training_ds` est le nom qui s’afficherait.

    ```Python
   training_ds = unregistered_ds.register(workspace = workspace,
                                     name = 'training_ds',
                                     description = 'training data'
                                     )
    ```

* Envoyez l’exécution enfant avec un jeu de données non inscrit dans le script. Cela aboutit à un jeu de données anonyme enregistré.

### <a name="trace-datasets-in-experiment-runs"></a>Suivre des jeux de données dans les exécutions d’expérimentation

Pour chaque expérience Machine Learning, vous pouvez facilement suivre les jeux de données utilisés comme entrée avec l’objet `Run` de l’expérience.

Le code suivant utilise la méthode [`get_details()`](/python/api/azureml-core/azureml.core.run.run?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-details--) pour suivre les jeux de données d’entrée qui ont été utilisés lors de l’exécution de l’expérimentation :

```Python
# get input datasets
inputs = run.get_details()['inputDatasets']
input_dataset = inputs[0]['dataset']

# list the files referenced by input_dataset
input_dataset.to_path()
```

Vous pouvez également trouver `input_datasets` à partir d’expériences à l’aide d’[Azure Machine Learning Studio](). 

L’illustration suivante montre où trouver le jeu de données d’entrée d’une expérience sur Azure Machine Learning Studio. Pour cet exemple, accédez à votre volet **Expériences** et ouvrez l’onglet **Propriétés** pour une exécution spécifique de votre expérience, `keras-mnist`.

![Jeux de données d'entrée](./media/how-to-version-track-datasets/input-datasets.png)

Utilisez le code suivant pour inscrire des modèles avec des jeux de données :

```Python
model = run.register_model(model_name='keras-mlp-mnist',
                           model_path=model_path,
                           datasets =[('training data',train_dataset)])
```

Après l’inscription, vous pouvez voir la liste des modèles inscrits auprès du jeu de données à l’aide de Python ou en accédant au [studio](https://ml.azure.com/).

L’affichage suivant est issu du volet **Jeux de données** sous **Ressources**. Sélectionnez le jeu de données, puis l’onglet **Modèles** pour obtenir la liste des modèles qui sont inscrits avec le jeu de données. 

![Jeux de données d'entrée](./media/how-to-version-track-datasets/dataset-models.png)

## <a name="next-steps"></a>Étapes suivantes

* [Entraîner avec des jeux de données](how-to-train-with-datasets.md)
* [Plus d’exemples de notebooks de jeu de données](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/)