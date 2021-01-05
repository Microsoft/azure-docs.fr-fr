---
title: Exemples de notebooks Jupyter utilisant des données NOAA
titleSuffix: Azure Open Datasets
description: Utilisez les exemples de notebooks Jupyter pour Azure Open Datasets afin de savoir comment charger des jeux de données ouverts et les utiliser pour enrichir les données de démonstration. Les techniques utilisent Spark et Pandas à des fins de traitement des données.
ms.service: open-datasets
ms.topic: sample
author: cjgronlund
ms.author: cgronlun
ms.date: 05/06/2020
ms.openlocfilehash: eddfcc36c6440ce155d7b9d81031db449cfa8d2b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492435"
---
# <a name="example-jupyter-notebooks-show-how-to-enrich-data-with-open-datasets"></a>Exemples de notebooks Jupyter montrant comment enrichir les données avec Open Datasets 
Les exemples de notebooks Jupyter pour Azure Open Datasets montrent comment charger des jeux de données ouverts et les utiliser pour enrichir les données de démonstration. Les techniques utilisent Apache Spark et Pandas à des fins de traitement des données.

>[!IMPORTANT]
>Dans un environnement autre que Spark, Azure Open Datasets permet de télécharger un seul mois de données à la fois, avec certaines classes, afin d’éviter l'erreur MemoryError liée aux jeux de données volumineux.

## <a name="load-noaa-integrated-surface-database-isd-data"></a>Charger des données ISD (Integrated Surface Database) NOAA 
|Notebook        | Description                                    |
|----------------|------------------------------------------------|
|[Charger un mois récent de données météorologiques dans un dataframe Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-access/02-weather-to-pandas-dataframe.ipynb) | Découvrez comment charger des données météorologiques historiques dans votre dataframe Pandas favori. |
|[Charger un mois récent de données météorologiques dans un dataframe Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-access/01-weather-to-spark-dataframe.ipynb) | Découvrez comment charger des données météorologiques historiques dans votre dataframe Spark favori.  |

## <a name="join-demo-data-with-noaa-isd-data"></a>Joindre des données de démonstration avec des données ISD NOAA 
|Notebook        | Description                                    |
|----------------|------------------------------------------------|
|[Joindre des données de démonstration avec des données météorologiques - Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/02-weather-join-in-pandas.ipynb) | Joignez un jeu de données de démonstration d'un mois d'emplacements de capteur avec relevés météorologiques dans un dataframe Pandas.  |
|[Joindre des données de démonstration avec des données météorologiques - Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/01-weather-join-in-spark.ipynb) | Joignez un jeu de données de démonstration d'emplacements de capteur avec relevés météorologiques dans un dataframe Spark. |

## <a name="join-nyc-taxi-data-with-noaa-isd-data"></a>Joindre des données de taxi de New York avec des données ISD NOAA 
|Notebook        | Description                                    |
|----------------|------------------------------------------------|
|[Données de trajet des taxis enrichies avec des données météorologiques - Pandas](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/04-nyc-taxi-join-weather-in-pandas.ipynb) | Chargez les données de trajet des taxis de New York (sur un mois) et enrichissez-les avec des données météorologiques dans un dataframe Pandas. Cet exemple remplace la méthode `get_pandas_limit` et équilibre les performances de chargement des données avec la quantité de données.|
|[Données de trajet des taxis enrichies avec des données météorologiques - Spark](https://github.com/Azure/OpenDatasetsNotebooks/blob/master/tutorials/data-join/03-nyc-taxi-join-weather-in-spark.ipynb) | Chargez les données de trajet des taxis de New York et enrichissez-les avec des données météorologiques dans un dataframe Spark.  |

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Modélisation de régression avec Machine Learning automatisé et jeu de données ouvert](../machine-learning/tutorial-auto-train-models.md?context=azure%2fopen-datasets%2fcontext%2fopen-datasets-context)
* [Kit de développement logiciel (SDK) Python pour Open Datasets](/python/api/azureml-opendatasets/azureml.opendatasets)
* [Catalogue Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/catalog/)
* [Créer un jeu de données Azure Machine Learning à partir d’Azure Open Datasets](how-to-create-azure-machine-learning-dataset-from-open-dataset.md)
