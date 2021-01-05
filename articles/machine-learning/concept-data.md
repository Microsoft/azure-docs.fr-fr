---
title: Sécuriser l’accès aux données dans le cloud
titleSuffix: Azure Machine Learning
description: Apprenez à vous connecter en toute sécurité à votre stockage de données sur Azure à l’aide de magasins de données et jeux de données Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 08/31/2020
ms.custom: devx-track-python, data4ml
ms.openlocfilehash: 6d8c04e48a3d0009a152830a4ee332cd706c4b2c
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360171"
---
# <a name="secure-data-access-in-azure-machine-learning"></a>Sécuriser l’accès aux données dans Azure Machine Learning

Azure Machine Learning facilite la connexion à vos données dans le cloud.  Il fournit une couche d’abstraction sur le service de stockage sous-jacent, ce qui vous permet d’accéder de manière sécurisée à vos données et de les utiliser sans avoir à écrire du code propre à votre type de stockage. Azure Machine Learning propose également les fonctionnalités de données suivantes :

*    Interopérabilité avec les DataFrames Pandas et Spark
*    Gestion de versions et suivi de la traçabilité des données
*    Étiquetage des données 
*    Supervision de la dérive des données
    
## <a name="data-workflow"></a>Workflow de données

Quand vous êtes prêt à utiliser les données dans votre solution de stockage cloud, nous vous recommandons de recourir au workflow de livraison de données suivant. Ce workflow suppose que vous disposez d’un [compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal) et de données dans un service de stockage cloud au sein d’Azure. 

1. Créez un [magasin de données Azure Machine Learning](#datastores) pour stocker les informations de connexion à votre stockage Azure.

2. À partir de ce magasin de données, créez un [jeu de données Azure Machine Learning](#datasets) pour pointer vers un ou plusieurs fichiers spécifiques dans votre stockage sous-jacent. 

3. Pour utiliser ce jeu de données dans votre expérience machine learning, vous pouvez
    1. Le monter sur la cible de calcul de votre expérience pour l’entraînement du modèle.

        **OR** 

    1. Le consommer directement dans des solutions Azure Machine Learning, telles que des exécutions d’expériences de machine learning automatisé, des pipelines de Machine Learning ou le [Concepteur Azure Machine Learning](concept-designer.md).

4. Créez des [moniteurs de jeux de données](#data-drift) pour le jeu de données de sortie de votre modèle afin de détecter la dérive de données. 

5. Si une dérive de données est détectée, mettez à jour votre jeu de données d’entrée et renouvelez l’entraînement de votre modèle en conséquence.

Le diagramme suivant fournit une démonstration visuelle de ce workflow recommandé.

![Diagramme représentant le service Stockage Azure, avec transmission vers un magasin de données, puis vers un jeu de données. La transmission s'effectue ensuite du jeu de données vers l'apprentissage du modèle, puis vers la dérive des données, avant de revenir au jeu de données.](./media/concept-data/data-concept-diagram.svg)

## <a name="datastores"></a>Magasins de données

Les magasins de données Azure Machine Learning conservent les informations de connexion à votre stockage Azure de manière sécurisée. Vous n’avez donc pas à les coder dans vos scripts. [Inscrivez et créez un magasin de données](how-to-access-data.md) pour vous connecter facilement à votre compte de stockage et accéder aux données de votre service de stockage Azure sous-jacent. 

Services de stockage cloud pris en charge dans Azure qui peuvent être enregistrés en tant que magasins de données :

+ Conteneur d’objets blob Azure
+ Partage de fichiers Azure
+ Azure Data Lake
+ Azure Data Lake Gen2
+ Azure SQL Database
+ Azure Database pour PostgreSQL
+ Système de fichiers Databricks
+ Azure Database pour MySQL

## <a name="datasets"></a>Groupes de données

Les jeux de données Azure Machine Learning ne sont pas des copies de vos données. En créant un jeu de données, vous créez une référence aux données dans son service de stockage, ainsi qu'une copie de ses métadonnées. 

Étant donné que les jeux de données sont évalués tardivement et que les données restent à leur emplacement existant :

* Vous n’engagez aucun coût de stockage supplémentaire.
* Vous ne risquez pas de modifier involontairement vos sources de données d’origine.
* Vous améliorez les performances des workflows de ML.

Pour interagir avec vos données dans le stockage, [créez un jeu de données](how-to-create-register-datasets.md) afin d’empaqueter vos données dans un objet consommable pour les tâches de machine learning. Inscrivez le jeu de données dans votre espace de travail pour le partager et le réutiliser dans différentes expériences sans avoir à gérer la complexité liée à l’ingestion des données.

Les jeux de données peuvent être créés à partir de fichiers locaux, d’URL publiques, d’[Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/) ou de services de stockage Azure par le biais de magasins de données. 

Il existe 2 types de datasets : 

+ Un [FileDataset](/python/api/azureml-core/azureml.data.file_dataset.filedataset?preserve-view=true&view=azure-ml-py) fait référence à des fichiers uniques ou multiples dans vos magasins de données ou vos URL publiques. Si vos données sont déjà nettoyées et prêtes à l’emploi dans des expériences de formation, vous pouvez [télécharger ou monter des fichiers](how-to-train-with-datasets.md#mount-files-to-remote-compute-targets) référencés par FileDatasets dans votre cible de calcul.

+ Un [TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py) représente les données sous forme de tableau en analysant le fichier ou la liste de fichiers fournis. Vous pouvez charger un TabularDataset dans une trame de données Pandas ou Spark afin d’effectuer une autre manipulation et un nettoyage. Pour obtenir la liste complète des formats de données à partir desquels vous pouvez créer des TabularDatasets, consultez la [classe TabularDatasetFactory](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory).

Vous trouverez des fonctionnalités supplémentaires des jeux de données dans la documentation suivante :

+ [Effectuez le suivi et la gestion des versions](how-to-version-track-datasets.md) de traçabilité des jeux de données.
+ [Supervisez votre jeu de données](how-to-monitor-datasets.md) pour faciliter la détection de la dérive de données.    

## <a name="work-with-your-data"></a>Travailler avec vos données

Avec les jeux de données, vous pouvez effectuer un certain nombre de tâches de Machine Learning grâce à une intégration fluide avec les fonctionnalités Azure Machine Learning. 

+ Créez un [projet d’étiquetage de données](#label).
+ Entraînez des modèles Machine Learning :
     + [expériences de ML automatisé](how-to-use-automated-ml-for-ml-models.md)
     + le [concepteur](tutorial-designer-automobile-price-train-score.md#import-data)
     + [notebooks](how-to-train-with-datasets.md)
     + [Pipelines Azure Machine Learning](how-to-create-your-first-pipeline.md)
+ Accédez aux jeux de données pour le scoring avec l’[inférence par lot](./tutorial-pipeline-batch-scoring-classification.md) dans des [pipelines machine learning](how-to-create-your-first-pipeline.md).
+ Configurez un moniteur de jeu de données pour la détection de la [dérive de données](#drift).

<a name="label"></a>

## <a name="data-labeling"></a>Étiquetage des données

L’étiquetage des grandes quantités de données a souvent été un casse-tête dans les projets de Machine Learning. Ceux avec un composant de vision par ordinateur, tels que la classification d’images ou la détection d’objets, nécessitent généralement des milliers d’images et d’étiquettes correspondantes.

Azure Machine Learning vous fournit un emplacement central pour créer, gérer et superviser les projets d’étiquetage. Les projets d’étiquetage aident à coordonner les données, les étiquettes et les membres de l’équipe, ce qui vous permet de gérer plus efficacement les tâches d’étiquetage. Les tâches actuellement prises en charge sont la classification d’images, à plusieurs étiquettes ou multiclasse, et l’identification des objets à l’aide de zones délimitées.

Créez un [projet d’étiquetage des données](how-to-create-labeling-projects.md) et générez un jeu de données à utiliser dans des expériences Machine Learning.

<a name="drift"></a>

## <a name="data-drift"></a>Dérive de données

Dans le contexte de l’apprentissage automatique, la dérive de données est la modification des données d’entrée du modèle qui entraîne une dégradation des performances du modèle. C’est l’une des principales raisons pour lesquelles la précision du modèle se dégrade au fil du temps, de sorte que la surveillance de la dérive des données permet de détecter les problèmes de performance du modèle.

Pour en savoir plus sur la détection et l’alerte en cas de dérive de données dans un jeu de données, consultez l’article [Créer un moniteur de jeu de données](how-to-monitor-datasets.md).

## <a name="next-steps"></a>Étapes suivantes 

+ Créez un jeu de données dans Azure Machine Learning Studio ou avec le SDK Python [en suivant ces étapes](how-to-create-register-datasets.md).
+ Essayez des exemples d’entraînement de jeux de données avec nos [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).