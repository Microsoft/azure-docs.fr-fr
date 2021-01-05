---
title: Qu’est-ce qu’Azure Synapse Analytics ?
description: Vue d’ensemble d’Azure Synapse Analytics
services: synapse-analytics
author: saveenr
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 10/28/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.openlocfilehash: c315dae7e5f02f112dfdfbec02e1ebaaa5e48a9f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445774"
---
# <a name="what-is-azure-synapse-analytics"></a>Qu’est-ce qu’Azure Synapse Analytics ?

En entreprise, l’analytique doit pouvoir traiter des quantités massives de données, qu’elles soient brutes, épurées ou très organisées. Les entreprises sont alors généralement obligées de combiner les technologies du Big Data et de l’entreposage de données dans des pipelines de données complexes qui fonctionnent sur des données de magasins relationnels et de lacs de données. Ces types de solutions sont difficiles à créer, à gérer et à sécuriser. Leur complexité retarde la livraison des insights dont les entreprises ont besoin.

**Azure Synapse** est un service d’analytique intégré qui accélère la génération d’insights dans les entrepôts de données et les systèmes de Big Data. Azure Synapse réunit ce qu’il se fait de mieux dans les technologies **SQL** utilisées dans l’entreposage des données d’entreprise, les **technologies Spark** utilisées pour le Big Data et les **pipelines** pour l’intégration des données et l’ETL/ELT. **Synapse Studio** offre une expérience unifiée pour la gestion, la supervision, le développement et la sécurité. Synapse s’intègre en profondeur à d’autres services Azure, comme **PowerBI**, **CosmosDB** et **AzureML**.

## <a name="key-features--benefits"></a>Fonctionnalités et avantages clés

### <a name="industry-leading-sql"></a>SQL de pointe

* **Synapse SQL** est un système de requêtes distribuées qui permet aux entreprises d’implémenter des scénarios d’entreposage et de virtualisation de données à l’aide d’expériences T-SQL standard et connues. Il étend également les capacités de SQL pour prendre en charge les scénarios de streaming et de Machine Learning.

* Synapse SQL offre à la fois des modèles de ressources **serverless** et **dédiés** pour fournir des options de consommation et de facturation adaptées à vos besoins. Pour bénéficier de performances et de coûts prévisibles, créez des pools SQL dédiés afin de réserver de la puissance de traitement aux données stockées dans les tables SQL. Pour des charges de travail non planifiées ou en rafale, utilisez le point de terminaison SQL serverless toujours disponible.
* Utilisez les fonctionnalités de **streaming** intégrées pour placer des données de sources de données cloud dans des tables SQL.
* Intégrez l’IA à SQL en utilisant des modèles **Machine Learning** pour effectuer le scoring des données avec la [fonction T-SQL PREDICT](https://docs.microsoft.com/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest).

### <a name="industry-standard-apache-spark"></a>Apache Spark standard

**Apache Spark pour Azure Synapse** offre une intégration profonde et fluide avec Apache Spark, le moteur de Big Data open source le plus répandu dans les domaines de la préparation des données, de l’engineering données, des processus ETL (extraction, transformation et chargement) et de l’apprentissage automatique.

* Modèles ML avec algorithmes SparkML et intégration d’AzureML pour Apache Spark 2.4 avec prise en charge intégrée de Linux Foundation Delta Lake.
* Modèle de ressource simplifié qui vous évite d’avoir à vous soucier de la gestion des clusters.
* Démarrage rapide Spark et mise à l’échelle automatique agressive.
* Prise en charge intégrée de .NET pour Spark, ce qui vous permet de mettre à profit votre expertise en C# et votre code .NET existant au sein d’une application Spark.

### <a name="interop-of-sql-and-apache-spark-on-your-data-lake"></a>Interop de SQL et Apache Spark sur votre lac de données

Azure Synapse supprime les barrières technologiques traditionnelles qui empêchent d’utiliser conjointement SQL et Spark. Vous pouvez mélanger les fonctions de façon fluide selon vos besoins et votre expertise.

* Un système de métadonnées compatible Hive partagé permet aux tables définies sur des fichiers dans le lac de données d’être consommées de manière homogène par Spark ou Hive.
* SQL et Spark peuvent explorer et analyser directement des fichiers Parquet, CSV, TSV et JSON stockés dans le lac de données.
* Chargement et déchargement scalables rapides pour les données passant entre les bases de données SQL et Spark

### <a name="built-in-data-integration-via-pipelines"></a>Intégration des données intégrée via des pipelines

Azure Synapse fournit par défaut le même moteur d’intégration de données et les mêmes expériences qu’Azure Data Factory, ce qui vous permet de créer des pipelines ETL à grande échelle sans quitter Azure Synapse Analytics.

* Ingérer des données depuis plus de 90 sources de données
* ETL sans code avec les activités de flux de données
* Orchestrer des notebooks, des travaux Spark, des procédures stockées, des scripts SQL, etc.

### <a name="unified-management-monitoring-and-security"></a>Gestion, supervision et sécurité unifiées

Azure Synapse offre aux entreprises une méthode unique pour gérer les ressources d’analytique, superviser l’utilisation et l’activité, et appliquer la sécurité.

* Affecter des utilisateurs à un rôle pour simplifier l’accès aux ressources d’unique
* Contrôle d’accès fin sur les données et le code
* Tableau de bord unique pour superviser les ressources, l’utilisation et les utilisateurs dans SQL et Spark

### <a name="synapse-studio"></a>Synapse Studio

**Synapse Studio** est une expérience native dans le web qui regroupe dans un seul et même endroit tout ce dont les ingénieurs de données ont besoin pour effectuer les tâches nécessaires à la création d’une solution complète.

* Créer une solution d’analytique de bout en bout en un seul endroit : ingestion, exploration, préparation, orchestration, visualisation
* Productivité de pointe pour les ingénieurs de données écrivant du code SQL ou Spark : création, débogage et optimisation des performances
* Intégrer aux processus CI/CD d’entreprise

## <a name="engage-with-the-synapse-engineering-team"></a>Collaborer avec l’équipe d’ingénierie de Synapse

- [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-synapse) : posez vos questions sur le développement.
- [Page de questions Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-synapse-analytics.html) : posez vos questions techniques.

## <a name="next-steps"></a>Étapes suivantes

* [Bien démarrer avec Azure Synapse Analytics](get-started.md)
* [Créer un espace de travail](quickstart-create-workspace.md)
* [Utiliser un pool SQL serverless](quickstart-sql-on-demand.md)
