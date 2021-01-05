---
title: Ingestion des données avec Azure Data Factory
titleSuffix: Azure Machine Learning
description: Découvrez les options disponibles pour construire un pipeline d’ingestion des données avec Azure Data Factory et les avantages de chacune.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: iefedore
author: eedorenko
manager: davete
ms.reviewer: larryfr
ms.date: 03/01/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, data4ml
ms.openlocfilehash: 73850d8022618dd6544d19564e425288aff09771
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360630"
---
# <a name="data-ingestion-with-azure-data-factory"></a>Ingestion des données avec Azure Data Factory

Dans cet article, vous allez découvrir les options disponibles pour la création d’un pipeline d’ingestion de données avec Azure Data Factory (ADF). Ce pipeline est utilisé pour ingérer des données à utiliser avec Azure Machine Learning. Azure Data Factory vous permet d’extraire, de transformer et de charger (ETL) facilement des données. Une fois les données transformées et chargées dans le stockage, elles peuvent être utilisées pour effectuer l’apprentissage de vos modèles Machine Learning.

Une transformation simple des données peut être effectuée avec les activités et les instruments ADF natifs, tels que le [flux de données](../data-factory/control-flow-execute-data-flow-activity.md). Lorsqu’il s’agit de scénarios plus compliqués, les données peuvent être traitées avec du code personnalisé. Par exemple, le code Python ou R.

Il existe plusieurs techniques courantes d’utilisation d’Azure Data Factory pour transformer des données lors de l’ingestion. Chaque technique a ses avantages et ses inconvénients qui déterminent s’ils sont adaptés à un cas d’usage spécifique :

| Technique | Avantages | Inconvénients |
| ----- | ----- | ----- |
| ADF + Azure Functions | Faible latence, calcul serverless</br>Fonctions avec état</br>Fonctions réutilisables | Convient uniquement au traitement de courte durée |
| ADF + composant personnalisé | Calcul parallèle à grande échelle</br>Convient aux algorithmes lourds | Enveloppement du code dans un fichier exécutable</br>Complexité de la gestion des dépendances et des E/S |
| ADF + notebook Azure Databricks | Apache Spark</br>Environnement Python natif | Peut être coûteuse</br>La création initiale de clusters prend du temps et ajoute de la latence

## <a name="adf-with-azure-functions"></a>ADF avec Azure Functions

![Le diagramme illustre un pipeline Azure Data Factory, avec une fonction Azure et l’exécution du pipeline de ML, ainsi qu’un pipeline Azure Machine Learning, avec un modèle d’apprentissage et la façon dont ils interagissent avec les données brutes et les données préparées.](media/how-to-data-ingest-adf/adf-function.png)

Azure Functions vous permet d’exécuter de petits morceaux de code (« fonctions ») sans vous préoccuper de l’infrastructure de l’application. Dans cette option, les données sont traitées avec du code Python personnalisé enveloppé dans une fonction Azure. 

La fonction est appelée à l’aide de l’[activité ADF Azure Functions](../data-factory/control-flow-azure-function-activity.md). Cette approche est une bonne option pour les transformations légères de données. 

* Avantages :
    * Les données sont traitées sur un calcul serverless avec une latence relativement faible
    * Le pipeline ADF peut appeler une [fonction Azure durable](../azure-functions/durable/durable-functions-overview.md) qui peut implémenter un workflow sophistiqué de transformation des données 
    * Les détails de la transformation de données sont extraits par la fonction Azure qui peut être réutilisée et appelée à partir d’autres emplacements
* Inconvénients :
    * Les fonctions Azure Functions doivent être créées avant d’être utilisées avec ADF
    * Azure Functions n’est utile que pour le traitement de données de courte durée

## <a name="adf-with-custom-component-activity"></a>ADF avec activité de composant personnalisé

![Le diagramme illustre un pipeline Azure Data Factory, avec un composant personnalisé et l’exécution du pipeline de ML, ainsi qu’un pipeline Azure Machine Learning, avec un modèle d’apprentissage et la façon dont ils interagissent avec les données brutes et les données préparées.](media/how-to-data-ingest-adf/adf-customcomponent.png)

Dans cette option, les données sont traitées avec du code Python personnalisé enveloppé dans un fichier exécutable. Il est appelé à l’aide d’une [activité de composant personnalisé ADF](../data-factory/transform-data-using-dotnet-custom-activity.md). Cette approche est plus adaptée aux données volumineuses que la technique précédente.

* Avantages :
    * Les données sont traitées sur le pool [Azure Batch](../batch/batch-technical-overview.md), qui permet un calcul parallèle à grande échelle et à haute performance
    * Peut être utilisée pour exécuter des algorithmes lourds et traiter de grandes quantités de données
* Inconvénients :
    * Le pool Azure Batch doit être créé avant d’être utilisé avec ADF
    * Repose sur l’ingénierie liée à l’enveloppement du code Python dans un fichier exécutable. Complexité de la gestion des dépendances et des paramètres entrée/sortie

## <a name="adf-with-azure-databricks-python-notebook"></a>ADF avec notebook Python Azure Databricks

![Le diagramme illustre un pipeline Azure Data Factory, avec Azure Databricks Python et l’exécution du pipeline de ML, ainsi qu’un pipeline Azure Machine Learning, avec un modèle d’apprentissage et la façon dont ils interagissent avec les données brutes et les données préparées.](media/how-to-data-ingest-adf/adf-databricks.png)

[Azure Databricks](https://azure.microsoft.com/services/databricks/) est une plateforme d’analytique basée sur Apache Spark dans le cloud Microsoft.

Dans cette technique, la transformation des données est effectuée par un [notebook Python](../data-factory/transform-data-using-databricks-notebook.md) exécuté sur un cluster Azure Databricks. C’est probablement l’approche la plus courante qui tire parti de toute la puissance d’un service Azure Databricks. Elle est conçue pour le traitement des données distribuées à grande échelle.

* Avantages :
    * Les données sont transformées sur le service Azure de traitement des données le plus puissant, qui est assisté par l’environnement Apache Spark.
    * Prise en charge native de Python avec les infrastructures et bibliothèques de science des données, notamment TensorFlow, PyTorch etscikit-learn.
    * Il n’est pas nécessaire d’envelopper le code Python dans des fonctions ou des modules exécutables. Le code fonctionne comme tel quel.
* Inconvénients :
    * L’infrastructure Azure Databricks doit être créée avant d’être utilisée avec ADF
    * Peut coûter cher en fonction de la configuration Azure Databricks
    * La rotation des clusters de calcul à partir du mode « froid » prend un certain temps, ce qui confère une latence élevée à la solution 
    

## <a name="consuming-data-in-azure-machine-learning-pipelines"></a>Utilisation de données dans des pipelines Azure Machine Learning

![Le diagramme illustre un pipeline Azure Data Factory et un pipeline Azure Machine Learning, et la façon dont ils interagissent avec les données brutes et les données préparées. Le pipeline Data Factory alimente les données dans la base de données préparée, qui alimente un magasin de données, qui alimente les jeux de données de l’espace de travail Machine Learning.](media/how-to-data-ingest-adf/aml-dataset.png)

Les données transformées du pipeline ADF sont enregistrées dans le stockage de données (comme Azure Blob). Azure Machine Learning peut accéder à ces données à l’aide de [magasins de données](./how-to-access-data.md#create-and-register-datastores) et de [jeux de données](./how-to-create-register-datasets.md).

Chaque fois que le pipeline ADF s’exécute, les données sont enregistrées à un autre emplacement du stockage. Pour transmettre l’emplacement à Azure Machine Learning, le pipeline ADF appelle un pipeline Azure Machine Learning. Lors de l’appel du pipeline ML, l’emplacement des données et l’ID d’exécution sont envoyés en tant que paramètres. Le pipeline ML peut ensuite créer un magasin de données ou un jeu de données à l’aide de l’emplacement des données. 

> [!TIP]
> Les jeux de données [prennent en charge le contrôle de version](./how-to-version-track-datasets.md), de sorte que le pipeline ML peut inscrire une nouvelle version du jeu de données qui pointe vers les données les plus récentes à partir du pipeline ADF.

Une fois que les données sont accessibles par le biais d’un magasin de données ou d’un jeu de données, vous pouvez les utiliser pour effectuer l’apprentissage d’un modèle ML. Le processus d’apprentissage peut faire partie du même pipeline ML appelé à partir d’ADF. Il peut également s’agir d’un processus distinct, tel que l’expérimentation dans un notebook Jupyter.

Étant donné que les jeux de données prennent en charge le contrôle de version et que chaque exécution à partir du pipeline crée une nouvelle version, il est facile de comprendre quelle version des données a été utilisée pour effectuer l’apprentissage d’un modèle.

## <a name="next-steps"></a>Étapes suivantes

* [Exécuter un notebook Databricks dans Azure Data Factory](../data-factory/transform-data-using-databricks-notebook.md)
* [Accéder aux données dans les services de stockage Azure](./how-to-access-data.md#create-and-register-datastores)
* [Effectuer l’apprentissage de modèles dans Azure Machine Learning](./how-to-train-with-datasets.md)
* [DevOps pour un pipeline d’ingestion des données](./how-to-cicd-data-ingestion.md)