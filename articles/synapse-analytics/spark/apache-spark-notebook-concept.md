---
title: Vue d’ensemble des notebooks Azure Synapse Analytics
description: Cet article fournit une vue d’ensemble des capacités offertes par les notebooks Azure Synapse Analytics.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/18/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: bad71cfeb8f15f463bb9659be704374325d1f50a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461998"
---
# <a name="azure-synapse-analytics-notebooks"></a>Notebooks Azure Synapse Analytics

Un notebook Synapse Studio est une interface web permettant de créer des fichiers contenant du code, des visualisations et du texte descriptif en direct. Les blocs-notes constituent un bon endroit où valider des idées et effectuer des expérimentations rapides pour extraire des insights de vos données. 

Un bloc-notes Azure Synapse Studio permet d’effectuer les opérations suivantes :

* Commencer à travailler sans le moindre effort de configuration.
* Sécuriser les données avec des fonctionnalités de sécurité d’entreprise intégrées.
* Analyser des données dans des formats bruts (CSV, txt, JSON, etc.), des formats de fichiers traités (Parquet, Delta Lake, ORC, etc.) et des fichiers de données tabulaires SQL sur Spark et SQL.
* Être productif grâce à des fonctionnalités de création améliorées et à la visualisation de données intégrée.

Cette section contient des articles sur la combinaison de langages, la création de visualisations de données, le paramétrage de notebooks, la création de pipelines et bien plus encore. Elle contient également des références et des tutoriels sur la façon dont vous pouvez vous familiariser avec le développement de notebook.

## <a name="create-manage-and-use-notebooks"></a>Créer, gérer et utiliser des notebooks
Vous pouvez gérer les notebooks à l’aide de l’interface utilisateur de Synapse Studio. 

Pour en savoir plus sur la façon dont vous pouvez créer et gérer des notebooks, consultez les articles suivants :
  - Gérer des notebooks
    - [Créer des notebooks](./spark/../apache-spark-development-using-notebooks.md#create-a-notebook)
    - [Développer des notebooks](./spark/../apache-spark-development-using-notebooks.md#develop-notebooks)
    - [Importer des données dans un bloc-notes](./spark/../apache-spark-development-using-notebooks.md#bring-data-to-a-notebook)
    - [Utiliser plusieurs langages à l’aide de commandes magic et de tables temporaires](./spark/../apache-spark-development-using-notebooks.md#integrate-a-notebook)
    - [Utiliser des commandes magic de cellule](./spark/../apache-spark-development-using-notebooks.md#magic-commands)
  - Développement
    - [Configurer les paramètres de session Spark](./spark/../apache-spark-development-using-notebooks.md#spark-session-config)
    - [Utiliser des utilitaires Microsoft Spark](./spark/../microsoft-spark-utilities.md)
    - [Visualiser les données à l’aide de notebooks et de bibliothèques](./spark/../apache-spark-data-visualization.md)
    - [Intégrer un notebooks dans des pipelines](./spark/../apache-spark-development-using-notebooks.md#integrate-a-notebook)


## <a name="next-steps"></a>Étapes suivantes
Les notebooks sont également largement utilisés pour la préparation et la visualisation de données, l’apprentissage automatique et d’autres scénarios en lien avec le Big Data. Pour en savoir plus sur l’utilisation des notebooks pour votre analyse de données et les scénarios de Big Data, consultez les tutoriels suivants :
  - [Créer un notebook](./spark/../../quickstart-apache-spark-notebook.md)
  - [Créer des visualisations à l’aide de notebooks Synapse Studio](./spark/../apache-spark-data-visualization-tutorial.md)
  - [Créer des modèles Machine Learning avec Apache Spark MLlib](./spark/../apache-spark-machine-learning-mllib-notebook.md)
  - [Créer des modèles Machine Learning avec Azure AutoML](./spark/../apache-spark-azure-machine-learning-tutorial.md)