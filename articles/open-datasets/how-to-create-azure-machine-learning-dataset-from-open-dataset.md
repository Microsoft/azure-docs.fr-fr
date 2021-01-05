---
title: Créer des jeu de données avec Azure Open Datasets
titleSuffix: Azure Open Datasets
description: Découvrez comment créer des jeux de données Azure Machine Learning à partir de Azure Open Datasets.
ms.service: open-datasets
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.date: 08/05/2020
ms.custom: how-to, tracking-python
ms.openlocfilehash: 6b9357c0fcf414c2575ca6966e8e5a3716015058
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654913"
---
# <a name="create-azure-machine-learning-datasets-from-azure-open-datasets"></a>Créer des jeux de données Azure Machine Learning à partir de Azure Open Datasets

Dans cet article, vous allez apprendre à intégrer des données d’enrichissement organisées dans vos expériences d’apprentissage automatique locales ou distantes grâce à des jeux de données [Azure Machine Learning](../machine-learning/overview-what-is-azure-ml.md) et [Azure Open Datasets](./index.yml). 

En créant un [jeu de données Azure Machine Learning](../machine-learning/how-to-create-register-datasets.md), vous créez une référence à l’emplacement de la source de données, ainsi qu’une copie de ses métadonnées. Étant donné que les jeux de données sont évalués tardivement et que les données restent à leur emplacement existant :
* Vous n’engagez aucun coût de stockage supplémentaire.
* Vous ne risquez pas de modifier involontairement vos sources de données d’origine. 
* Vous améliorez les performances des workflows de ML.

Pour comprendre où figurent les jeux de données dans le flux de travail global d’accès aux données d’Azure Machine Learning, consultez l’article [Sécuriser l’accès aux données](../machine-learning/concept-data.md#data-workflow).

Les jeux de données Azure Open Datasets sont des jeux de données publics organisés que vous pouvez utiliser pour ajouter des fonctionnalités spécifiques à des scénarios afin d’enrichir vos solutions prédictives et d’améliorer leur précision. Reportez-vous au [catalogue Open Datasets](https://azure.microsoft.com/en-in/services/open-datasets/catalog/) pour obtenir des données du domaine public qui peuvent vous aider à effectuer l’apprentissage de modèles Machine Learning, par exemple :

* [météo](https://azure.microsoft.com/services/open-datasets/catalog/noaa-integrated-surface-data/)
* [recensement](https://azure.microsoft.com/services/open-datasets/catalog/us-decennial-census-zip/)
* [jours fériés](https://azure.microsoft.com/services/open-datasets/catalog/public-holidays/)
* [sécurité publique](https://azure.microsoft.com/services/open-datasets/catalog/chicago-safety-data/)
* location

Les jeux de données ouverts se trouvent dans le Cloud sur Microsoft Azure et sont inclus à la fois dans le [kit de développement logiciel (SDK) Python Azure Machine Learning](#create-datasets-with-the-sdk) et dans le [studio Azure Machine Learning](#create-datasets-with-the-studio).


## <a name="prerequisites"></a>Prérequis

Pour cet article, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’en avez pas, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

* Un [espace de travail Azure Machine Learning](../machine-learning/how-to-manage-workspace.md).

* Le [Kit de développement logiciel (SDK) Azure Machine Learning pour Python installé](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py), qui inclut le paquet `azureml-datasets`.

    * Créez une [instance de calcul Azure Machine Learning](../machine-learning/how-to-create-manage-compute-instance.md), qui est un environnement de développement entièrement configuré et géré qui comprend des blocs-notes intégrés et le kit de développement logiciel (SDK) déjà installé.

    **OR**

    * Travaillez sur votre propre environnement Python et installez vous-même le Kit de développement logiciel (SDK) à l’aide de [ces instructions](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).

> [!NOTE]
> Certaines classes de jeu de données ont des dépendances avec le package [azureml-dataprep](/python/api/azureml-dataprep/), qui n’est compatible qu’avec Python 64 bits. Pour les utilisateurs Linux, ces classes sont uniquement prises en charge dans les distributions suivantes :  Red Hat Enterprise Linux (7, 8), Ubuntu (14.04, 16.04, 18.04), Fedora (27, 28), Debian (8, 9) et CentOS (7).

## <a name="create-datasets-with-the-sdk"></a>Créer des jeux de données avec le kit de développement logiciel (SDK)

Pour créer des jeux de données Azure Machine Learning à l’aide des classes Azure Open Datasets dans le Kit de développement logiciel (SDK) Python, assurez-vous d’avoir installé le package avec `pip install azureml-opendatasets`. Chaque jeu de données discret est représenté par sa propre classe dans le SDK, et certaines classes sont disponibles en tant que [`TabularDataset` ou `FileDataset`](../machine-learning/how-to-create-register-datasets.md#dataset-types) Azure Machine Learning, ou les deux. Pour obtenir la liste complète des classes `opendatasets`, consultez la [documentation de référence](/python/api/azureml-opendatasets/azureml.opendatasets?preserve-view=true&view=azure-ml-py).

Vous pouvez récupérer certaines classes `opendatasets` en tant que `TabularDataset` ou `FileDataset`, ce qui vous permet de manipuler et/ou de télécharger les fichiers directement. D’autres classes peuvent obtenir un jeu de données **seulement** à l’aide des fonctions `get_tabular_dataset()` ou `get_file_dataset()` de la classe `Dataset` dans le SDK Python.

Le code suivant montre que la classe `opendatasets` MNIST peut retourner `TabularDataset` ou `FileDataset`. 


```python
from azureml.core import Dataset
from azureml.opendatasets import MNIST

# MNIST class can return either TabularDataset or FileDataset
tabular_dataset = MNIST.get_tabular_dataset()
file_dataset = MNIST.get_file_dataset()
```

Dans cet exemple, la classe `opendatasets` Diabète n’est disponible qu’en tant que `TabularDataset`, d’où l’utilisation de `get_tabular_dataset()`.

```python

from azureml.opendatasets import Diabetes
from azureml.core import Dataset

# Diabetes class can return ONLY TabularDataset and must be called from the static function
diabetes_tabular = Diabetes.get_tabular_dataset()
```
## <a name="register-datasets"></a>Inscrire des jeux de données

Inscrivez un jeu de données Azure Machine Learning auprès de votre espace de travail afin de pouvoir les partager avec d’autres personnes et les réutiliser dans des expériences dans votre espace de travail. Lorsque vous inscrivez un jeu de données Azure Machine Learning créé à partir d’Open Datasets, aucune donnée n’est immédiatement téléchargée, mais les données seront accessibles ultérieurement à la demande (pour la formation, par exemple) à partir d’un emplacement de stockage central.

Pour inscrire vos jeux de données auprès d’un espace de travail, utilisez la méthode [`register()`](/python/api/azureml-core/azureml.data.abstract_dataset.abstractdataset?view=azure-ml-py#register-workspace--name--description-none--tags-none--create-new-version-false-&preserve-view=true ). 
```Python
titanic_ds = titanic_ds.register(workspace=workspace,
                                 name='titanic_ds',
                                 description='titanic training data')
```

## <a name="create-datasets-with-the-studio"></a>Créer des jeux de données avec le studio

Vous pouvez également créer des jeux de données Azure Machine Learning à partir d’Azure Open Datasets à l’aide d’[Azure Machine Learning Studio](https://ml.azure.com), une interface web centralisée qui comprend des outils d’apprentissage automatique permettant de mettre en œuvre des scénarios de science des données pour des utilisateurs de science des données de tous niveaux de compétence.

> [!Note]
> Les jeux de données créés via Azure Machine Learning Studio sont automatiquement inscrits auprès de l’espace de travail.

1. Dans votre espace de travail, sélectionnez l’onglet **Jeux de données** sous **Ressources**. Dans le menu déroulant **Créer un jeu de données**, sélectionnez **À partir d’Open Datasets**.

    ![Ouvrir un jeu de données avec l’interface utilisateur](./media/how-to-create-dataset-from-open-dataset/open-datasets-1.png)

1. Choisissez un jeu de données en sélectionnant sa vignette. (Vous avez la possibilité de filtrer en utilisant la barre de recherche.) Sélectionnez **Suivant**.

    ![Choisir un jeu de données](./media/how-to-create-dataset-from-open-dataset/open-datasets-2.png)

1. Choisissez ensuite un nom sous lequel inscrire le jeu de données, puis filtrez éventuellement les données en utilisant les filtres disponibles. Dans cet exemple, pour le jeu de données des **jours fériés**, vous filtrez la période sur une année et le code pays sur États-Unis seulement. Pour plus d’informations sur les données, telles que les descriptions de champ et les plages de dates, consultez le [catalogue Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog). Sélectionnez **Create** (Créer).

    ![Définir les paramètres du jeu de données et créer le jeu de données](./media/how-to-create-dataset-from-open-dataset/open-datasets-3.png)

    Le jeu de données est maintenant disponible dans votre espace de travail sous **Jeux de données**. Vous pouvez l’utiliser de la même façon que les autres jeux de données que vous avez créés.


## <a name="access-datasets-for-your-experiments"></a>Accédez aux jeux de données pour vos expériences

Utilisez vos jeux de données dans vos expériences d’apprentissage automatique pour la formation de modèles ML. [Découvrez-en plus sur l’entraînement avec des jeux de données](../machine-learning/how-to-train-with-datasets.md).

## <a name="example-notebooks"></a>Exemples de notebooks

Pour obtenir des exemples et des démonstrations des fonctionnalités d’Open Datasets, consultez ces [exemples de notebooks](samples.md).

## <a name="next-steps"></a>Étapes suivantes

* [Effectuer l’apprentissage de votre premier modèle ML](../machine-learning/tutorial-1st-experiment-sdk-train.md).

* [Effectuer l’apprentissage avec des jeux de données](../machine-learning/how-to-train-with-datasets.md).

* [Créer un jeu de données Azure Machine Learning](../machine-learning/how-to-create-register-datasets.md).
