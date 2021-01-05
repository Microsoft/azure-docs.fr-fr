---
title: Créer des jeux de données Azure Machine Learning pour accéder aux données
titleSuffix: Azure Machine Learning
description: Découvrez comment créer des jeux de données Azure Machine Learning pour accéder à vos données pour des exécutions d’expérience de machine learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1, data4ml
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.openlocfilehash: 28e70a5d5a6ac4cd51f5ed3fc85afd47a5af68d8
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033270"
---
# <a name="create-azure-machine-learning-datasets"></a>Créer des jeux de données Azure Machine Learning



Dans cet article, vous découvrez comment créer des jeux de données Azure Machine Learning pour accéder aux données de vos expériences locales ou à distance avec le kit de développement logiciel (SDK) Python Azure Machine Learning. Pour comprendre où figurent les jeux de données dans le flux de travail global d’accès aux données d’Azure Machine Learning, consultez l’article [Sécuriser l’accès aux données](concept-data.md#data-workflow).

En créant un jeu de données, vous créez une référence à l’emplacement de la source de données, ainsi qu’une copie de ses métadonnées. Étant donné que les données restent à leur emplacement existant, vous n’exposez aucun coût de stockage supplémentaire et ne risquez pas l’intégrité de vos sources de données. Les jeux de données sont également évalués tardivement, ce qui contribue aux vitesses de performances de flux de travail. Vous pouvez créer des jeux de données à partir de magasins de données, d’URL publiques et de [Azure Open Datasets](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md).

Pour une expérience à faible niveau de codage, [créez des jeux de données Azure Machine Learning avec Azure Machine Learning Studio.](how-to-connect-data-ui.md#create-datasets).

Avec les jeux de données Azure Machine Learning, vous pouvez :

* Conserver une seule copie des données dans votre stockage, référencée par des jeux de données.

* Accédez simplement aux données lors de l’entraînement du modèle sans vous soucier des chaînes de connexion ou des chemins des données.[En savoir plus sur l’apprentissage avec des jeux de données](how-to-train-with-datasets.md).

* Partager des données et collaborer avec d’autres utilisateurs.

## <a name="prerequisites"></a>Prérequis

Pour créer et utiliser des jeux de données, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’en avez pas, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

* Un [espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* Le [SDK Azure Machine Learning pour Python installé](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py), qui inclut le paquet azureml-datasets.

    * Créez une [instance de calcul Azure Machine Learning](how-to-create-manage-compute-instance.md), qui est un environnement de développement entièrement configuré et géré qui comprend des blocs-notes intégrés et le kit de développement logiciel (SDK) déjà installé.

    **OR**

    * Travaillez sur votre propre bloc-notes Jupyter et installez vous-même le kit de développement logiciel (SDK) à l’aide de [ces instructions](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).

> [!NOTE]
> Certaines classes de jeu de données ont des dépendances avec le package [azureml-dataprep](/python/api/azureml-dataprep/?preserve-view=true&view=azure-ml-py), qui n’est compatible qu’avec Python 64 bits. Pour les utilisateurs Linux, ces classes sont uniquement prises en charge dans les distributions suivantes :  Red Hat Enterprise Linux (7, 8), Ubuntu (14.04, 16.04, 18.04), Fedora (27, 28), Debian (8, 9) et CentOS (7). Si vous utilisez des distributions non pris en charge, suivez [ce guide](/dotnet/core/install/linux) afin d'installer .NET Core 2.1 pour continuer. 

## <a name="compute-size-guidance"></a>Conseils liés à la taille de calcul

Quand vous créez un jeu de données, passez en revue la puissance de traitement de calcul et la taille de vos données en mémoire. La taille de vos données dans le stockage n’est pas la même que la taille des données dans un dataframe. Par exemple, les données des fichiers CSV peuvent décupler de volume dans un dataframe ; ainsi, un fichier CSV de 1 Go peut occuper 10 Go dans un dataframe. 

Si vos données sont compressées, elles peuvent s’étendre davantage ; 20 Go de données relativement éparses stockées au format de compression Parquet peuvent s’étendre jusqu’à environ 800 Go en mémoire. Étant donné que les fichiers Parquet stockent les données dans un format en colonnes, si vous avez uniquement besoin de la moitié des colonnes, il vous suffit de charger environ 400 Go en mémoire.

[Apprenez-en davantage sur l’optimisation du traitement des données dans Azure Machine Learning](concept-optimize-data-processing.md).

## <a name="dataset-types"></a>Types de jeux de données

Il existe deux types de jeux de données, selon la façon dont les utilisateurs les consomment lors de l’entraînement : FileDatasets et TabularDatasets. Les deux types peuvent être utilisés dans des flux de travail d’apprentissage Azure Machine Learning impliquant des estimateurs, AutoML, hyperDrive et des pipelines. 

### <a name="filedataset"></a>FileDataset

Un [FileDataset](/python/api/azureml-core/azureml.data.file_dataset.filedataset?preserve-view=true&view=azure-ml-py) fait référence à des fichiers uniques ou multiples dans vos magasins de données ou vos URL publiques. Si vos données sont déjà nettoyées et prêtes à l’emploi dans des expériences d’apprentissage, vous pouvez [télécharger ou monter](how-to-train-with-datasets.md#mount-vs-download) les fichiers dans votre calcul en tant qu’objet FileDataset. 

Nous vous recommandons d’utiliser FileDatasets pour vos flux de travail d’apprentissage automatique, car les fichiers sources peuvent être dans n’importe quel format, ce qui permet un éventail plus large de scénarios d’apprentissage automatique, dont l’apprentissage profond.

Créez un FileDataset avec le [kit de développement logiciel (SDK) Python](#create-a-filedataset) ou [Azure Machine Learning Studio](how-to-connect-data-ui.md#create-datasets).
### <a name="tabulardataset"></a>TabularDataset

Un [TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py) représente les données sous forme de tableau en analysant le fichier ou la liste de fichiers fournis. Cela vous permet de matérialiser les données dans une tramedonnées pandas ou Spark afin de pouvoir travailler avec des bibliothèques de formation et de préparation des données familières sans avoir à quitter votre notebook. Vous pouvez créer un objet `TabularDataset` à partir de fichiers .csv, .tsv, .parquet et .jsonl, et à partir de [résultats de requête SQL](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory?preserve-view=true&view=azure-ml-py#&preserve-view=truefrom-sql-query-query--validate-true--set-column-types-none--query-timeout-30-).

Avec TabularDatasets, vous pouvez spécifier un horodatage à partir d’une colonne dans les données ou à partir de là où les données du modèle de chemin sont stockées pour activer une caractéristique de série chronologique. Cette spécification permet un filtrage chronologique facile et efficace. Pour un exemple, consultez [Démonstration de l’API pour une série chronologique tabulaire avec des données météo NOAA](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/timeseries-datasets/tabular-timeseries-dataset-filtering.ipynb).

Créez un TabularDataset avec le [Kit de développement logiciel (SDK) Python](#create-a-tabulardataset) ou [Azure Machine Learning Studio](how-to-connect-data-ui.md#create-datasets).

>[!NOTE]
> Les flux de travail AutoML générés via Azure Machine Learning Studio prennent actuellement en charge uniquement les TabularDatasets. 

## <a name="access-datasets-in-a-virtual-network"></a>Accéder à des jeux de données dans un réseau virtuel

Si votre espace de travail se trouve dans un réseau virtuel, vous devez configurer le jeu de données pour ignorer la validation. Pour plus d’informations sur l’utilisation de magasins de données et de jeux de données dans un réseau virtuel, consultez [Sécuriser un espace de travail et les ressources associées](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

<a name="datasets-sdk"></a>

## <a name="create-datasets"></a>Créez les jeux de données

Pour que les données soient accessibles par Azure Machine Learning, les jeux de données doivent être créés à partir de chemins dans des [magasins de données Azure](how-to-access-data.md) ou des URL web publiques. 

Pour créer des jeux de données à partir d’un [magasin de données Azure](how-to-access-data.md) à l’aide du Kit de développement logiciel (SDK) Python :

1. Vérifiez que vous disposez d’un accès `contributor` ou `owner` au magasin de données Azure inscrit.

2. Créez le jeu de données en référençant des chemins d’accès dans le magasin de données. Vous pouvez créer un jeu de données à partir de plusieurs chemins d’accès dans plusieurs magasins de données. Il n’existe aucune limite inconditionnelle quant au nombre de fichiers ou à la taille de données à partir desquels vous pouvez créer un jeu de données. 

> [!NOTE]
> Pour chaque chemin d’accès aux données, quelques requêtes sont envoyées au service de stockage pour vérifier s’il pointe vers un fichier ou un dossier. Cette surcharge peut entraîner une dégradation des performances ou une défaillance. Un jeu de données référençant un dossier contenant 1 000 fichiers est considéré comme référençant un chemin d’accès de données. Nous vous recommandons de créer un jeu de données référençant moins de 100 chemins d’accès dans des magasins de données pour obtenir des performances optimales.

### <a name="create-a-filedataset"></a>Créer un FileDataset

Utilisez la méthode [`from_files()`](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory?preserve-view=true&view=azure-ml-py#&preserve-view=truefrom-files-path--validate-true-) sur la classe `FileDatasetFactory` pour charger des fichiers de n’importe quel format et créer un FileDataset non inscrit. 

Si votre stockage se trouve derrière un réseau virtuel ou un pare-feu, définissez le paramètre `validate=False` dans votre méthode `from_files()`. Cela permet de contourner l’étape de validation initiale et vous permet de créer votre jeu de données à partir de ces fichiers sécurisés. Apprenez-en davantage sur l’utilisation de [magasins de données et de jeux de données dans un réseau virtuel](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

```Python
# create a FileDataset pointing to files in 'animals' folder and its subfolders recursively
datastore_paths = [(datastore, 'animals')]
animal_ds = Dataset.File.from_files(path=datastore_paths)

# create a FileDataset from image and label files behind public web urls
web_paths = ['https://azureopendatastorage.blob.core.windows.net/mnist/train-images-idx3-ubyte.gz',
             'https://azureopendatastorage.blob.core.windows.net/mnist/train-labels-idx1-ubyte.gz']
mnist_ds = Dataset.File.from_files(path=web_paths)
```
Pour réutiliser et partager des jeux de données dans des expériences au sein de votre espace de travail, [inscrivez votre jeu de données](#register-datasets). 

> [!TIP] 
> Chargez des fichiers à partir d’un répertoire local et créez un FileDataset dans une méthode unique avec la méthode de préversion publique [upload_directory()](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory?preserve-view=true&view=azure-ml-py#upload-directory-src-dir--target--pattern-none--overwrite-false--show-progress-true-). Cette méthode est une fonctionnalité d’évaluation [expérimentale](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#stable-vs-experimental) en préversion qui peut évoluer à tout moment. 
> 
>  Elle permet de charger des données dans votre stockage sous-jacent, ce qui entraîne la facturation de coûts de stockage. 
### <a name="create-a-tabulardataset"></a>Créer un TabularDataset

Utilisez la méthode [`from_delimited_files()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory) sur la classe `TabularDatasetFactory` pour lire des fichiers au format .csv ou .tsv, puis créez un TabularDataset non inscrit. Si vous lisez à partir de plusieurs fichiers, les résultats sont agrégés dans une même représentation tabulaire. 

Si votre stockage se trouve derrière un réseau virtuel ou un pare-feu, définissez le paramètre `validate=False` dans votre méthode `from_delimited_files()`. Cela permet de contourner l’étape de validation initiale et vous permet de créer votre jeu de données à partir de ces fichiers sécurisés. Apprenez-en davantage sur l’utilisation de [magasins de données et de jeux de données dans un réseau virtuel](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

Le code suivant obtient l’espace de travail existant et le magasin de données souhaité par nom. Ensuite, il transmet les emplacements du magasin de fichiers et des fichiers au paramètre `path` pour créer un nouvel objet TabularDataset, `weather_ds`.

```Python
from azureml.core import Workspace, Datastore, Dataset

datastore_name = 'your datastore name'

# get existing workspace
workspace = Workspace.from_config()
    
# retrieve an existing datastore in the workspace by name
datastore = Datastore.get(workspace, datastore_name)

# create a TabularDataset from 3 file paths in datastore
datastore_paths = [(datastore, 'weather/2018/11.csv'),
                   (datastore, 'weather/2018/12.csv'),
                   (datastore, 'weather/2019/*.csv')]

weather_ds = Dataset.Tabular.from_delimited_files(path=datastore_paths)
```

Par défaut, quand vous créez un TabularDataset, les types de données des colonnes sont inférés automatiquement. Si les types inférés ne sont pas ceux que vous attendez, vous pouvez les spécifier en utilisant le code suivant. Le paramètre `infer_column_type` s’applique uniquement aux jeux de données créés à partir de fichiers délimités. [Apprenez-en davantage sur les types de données pris en charge](/python/api/azureml-core/azureml.data.dataset_factory.datatype?preserve-view=true&view=azure-ml-py).


```Python
from azureml.core import Dataset
from azureml.data.dataset_factory import DataType

# create a TabularDataset from a delimited file behind a public web url and convert column "Survived" to boolean
web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path, set_column_types={'Survived': DataType.to_bool()})

# preview the first 3 rows of titanic_ds
titanic_ds.take(3).to_pandas_dataframe()
```

|(Index)|PassengerId|Survived|Pclass|Nom|Sex|Age|SibSp|Parch|Ticket|Fare|Cabin|Embarked
-|-----------|--------|------|----|---|---|-----|-----|------|----|-----|--------|
0|1|False|3|Braund, Mr. Owen Harris|male|22.0|1|0|A/5 21171|7.2500||S
1|2|True|1|Cumings, Mrs. John Bradley (Florence Briggs Th...|female|38.0|1|0|PC 17599|71.2833|C85|C
2|3|True|3|Heikkinen, Miss. Laina|female|26,0|0|0|STON/O2. 3101282|7.9250||S

Pour réutiliser et partager des jeux de données dans des expériences de votre espace de travail, [inscrivez votre jeu de données](#register-datasets).

## <a name="create-a-dataset-from-pandas-dataframe"></a>Créer un jeu de données à partir d’une tramedonnées pandas

Pour créer un TabularDataset à partir d’une tramedonnées pandas en mémoire, écrivez les données dans un fichier local (par exemple, un fichier csv), puis créez votre jeu de données à partir de ce fichier. Le code suivant illustre ce workflow.

```python
# azureml-core of version 1.0.72 or higher is required
# azureml-dataprep[pandas] of version 1.1.34 or higher is required

from azureml.core import Workspace, Dataset
local_path = 'data/prepared.csv'
dataframe.to_csv(local_path)

# upload the local file to a datastore on the cloud

subscription_id = 'xxxxxxxxxxxxxxxxxxxxx'
resource_group = 'xxxxxx'
workspace_name = 'xxxxxxxxxxxxxxxx'

workspace = Workspace(subscription_id, resource_group, workspace_name)

# get the datastore to upload prepared data
datastore = workspace.get_default_datastore()

# upload the local file from src_dir to the target_path in datastore
datastore.upload(src_dir='data', target_path='data')

# create a dataset referencing the cloud location
dataset = Dataset.Tabular.from_delimited_files(path = [(datastore, ('data/prepared.csv'))])
```

> [!TIP]
> Créez et inscrivez un TabularDataset à partir d’un dataframe Spark ou Pandas en mémoire avec une méthode unique avec les méthodes de préversion publique [`register_spark_dataframe()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory?preserve-view=true&view=azure-ml-py#methods) et [`register_pandas_dataframe()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory?preserve-view=true&view=azure-ml-py#methods). Ces méthodes d’inscription sont des fonctionnalités d’évaluation [expérimentales](/python/api/overview/azure/ml/?preserve-view=true&view=azure-ml-py#stable-vs-experimental) susceptibles d’évoluer à tout moment. 
> 
>  Elles chargent les données dans votre stockage sous-jacent et entraînent la facturation de coûts de stockage. 

## <a name="register-datasets"></a>Inscrire des jeux de données

Pour terminer le processus de création, inscrivez vos jeux de données dans un espace de travail. Utilisez la méthode [`register()`](/python/api/azureml-core/azureml.data.abstract_dataset.abstractdataset?preserve-view=true&view=azure-ml-py#&preserve-view=trueregister-workspace--name--description-none--tags-none--create-new-version-false-) pour inscrire des jeux de données auprès de votre espace de travail afin de pouvoir les partager avec d’autres personnes et les réutiliser dans des expériences dans votre espace de travail :

```Python
titanic_ds = titanic_ds.register(workspace=workspace,
                                 name='titanic_ds',
                                 description='titanic training data')
```

## <a name="create-datasets-using-azure-resource-manager"></a>Créer des jeux de données à l'aide d'Azure Resource Manager

Les modèles disponibles à l'adresse [https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-dataset-create-*](https://github.com/Azure/azure-quickstart-templates/tree/master/) peuvent être utilisés pour créer des jeux de données.

Pour plus d'informations sur l'utilisation de ces modèles, consultez [Utiliser un modèle Azure Resource Manager afin de créer un espace de travail pour Azure Machine Learning](how-to-create-workspace-template.md).


## <a name="create-datasets-with-azure-open-datasets"></a>Créer des jeu de données avec Azure Open Datasets

Les jeux de données [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/) sont des jeux de données publics organisés que vous pouvez utiliser pour ajouter des fonctionnalités spécifiques à des scénarios à des solutions de Machine Learning afin d'obtenir des modèles plus précis. Les jeux de données englobent des données du domaine public portant sur la météo, le recensement, les jours fériés, la sécurité publique et la localisation et vous aident à former des modèles Machine Learning et à enrichir des solutions prédictives. Open Datasets se trouve dans le cloud dans Microsoft Azure et est inclus dans le SDK et dans le studio.

Découvrez comment créer des [jeux de données Azure Machine Learning à partir de Azure Open Datasets](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md). 

## <a name="train-with-datasets"></a>Entraîner avec des jeux de données

Utilisez vos jeux de données dans vos expériences d’apprentissage automatique pour la formation de modèles ML. [En savoir plus sur l’apprentissage avec des jeux de données](how-to-train-with-datasets.md)

## <a name="version-datasets"></a>Jeux de données avec version

Vous pouvez inscrire un nouveau jeu de données sous le même nom en créant une nouvelle version. La version d’un jeu de données est un moyen de marquer l’état de vos données, afin de pouvoir appliquer une version spécifique du jeu de données pour une expérimentation ou une reproduction ultérieure. En savoir plus sur les [versions des jeux de données](how-to-version-track-datasets.md).
```Python
# create a TabularDataset from Titanic training data
web_paths = ['https://dprepdata.blob.core.windows.net/demo/Titanic.csv',
             'https://dprepdata.blob.core.windows.net/demo/Titanic2.csv']
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_paths)

# create a new version of titanic_ds
titanic_ds = titanic_ds.register(workspace = workspace,
                                 name = 'titanic_ds',
                                 description = 'new titanic training data',
                                 create_new_version = True)
```

## <a name="next-steps"></a>Étapes suivantes

* Découvrir [comment former un modèle avec des jeux de données](how-to-train-with-datasets.md).
* Utilisez le Machine Learning automatisé pour [vous entraîner avec les TabularDatasets](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb).
* Pour obtenir plus d’exemples d’entraînement de jeux de données, consultez les [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).
