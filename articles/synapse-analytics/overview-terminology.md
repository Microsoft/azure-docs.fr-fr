---
title: Terminologie – Azure Synapse Analytics
description: Guide de référence pour l’utilisateur d’Azure Synapse Analytics
services: synapse-analytics
author: saveenr
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 11/18/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.openlocfilehash: 7f0ca83c194aae6f4a8d04d70c2d4f3746ad2ca5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446709"
---
# <a name="azure-synapse-analytics-terminology"></a>Terminologie Azure Synapse Analytics

Ce document présente les concepts de base d’Azure Synapse Analytics.

## <a name="basics"></a>Informations de base

Un **espace de travail Synapse** est une limite de collaboration sécurisable pour l’analytique d’entreprise basée sur le cloud dans Azure. Un espace de travail est déployé dans une région spécifique et est associé à un compte ADLS Gen2 et à un système de fichiers (pour le stockage des données temporaires). Un espace de travail se trouve sous un groupe de ressources.

Un espace de travail vous permet d’effectuer des analyses avec SQL et Apache Spark. Les ressources disponibles pour les analyses SQL et Spark sont organisées en **pools** SQL et Spark. 

## <a name="linked-services"></a>Services liés

Un espace de travail peut contenir un nombre quelconque de **services liés**, essentiellement des chaînes de connexion qui définissent les informations de connexion nécessaires à l’espace de travail pour se connecter à des ressources externes.

## <a name="synapse-sql"></a>SQL Synapse

**Synapse SQL** permet d’effectuer des analyses basées sur T-SQL dans un espace de travail Synapse. Synapse SQL dispose de deux modèles de consommation : dédié et serverless.  Pour le modèle dédié, utilisez des **pools SQL** dédiés. Un espace de travail peut avoir un nombre quelconque de ces pools. Pour utiliser le modèle serverless, utilisez les **pools SQL serverless**. Chaque espace de travail possède un de ces pools.

À l’intérieur de Synapse Studio, vous pouvez travailler avec des pools SQL en créant et en exécutant des **scripts SQL**.

## <a name="apache-spark-for-synapse"></a>Apache Spark pour Synapse

Pour utiliser les analyses Spark, créez et utilisez des **pools Apache Spark serverless** dans votre espace de travail Synapse. Lorsque vous commencez à utiliser un pool Spark, les espaces de travail créent une **session Spark** afin de gérer les ressources associées à cette session. 

Il existe deux façons d’utiliser Spark dans Synapse :
* **Notebooks Spark** pour l’ingénierie et la science des données avec Scala, PySpark, C# et SparkSQL
* **Définitions de travaux Spark** pour l’exécution de travaux Spark par lots à l’aide de fichiers jar.

## <a name="pipelines"></a>Pipelines

Les pipelines représente la manière dont Azure Synapse assure l’intégration de données, ce qui vous permet de déplacer des données entre les services et d’orchestrer des activités.

* **Pipeline** – Regroupement logique d’activités qui effectuent une tâche ensemble.
* **Activités** – Définition des actions à effectuer sur des données au sein d’un pipeline, telles que la copie de données, l’exécution d’un notebook ou d’un script SQL.
* **Flux de données** – Type spécifique d’activité qui fournit une expérience sans code permettant d’effectuer une transformation de données qui utilise Synapse Spark en arrière-plan.
* **Déclencheur** – Exécute un pipeline. Il peut être exécuté manuellement ou automatiquement (planification, fenêtre bascule ou événement).
* **Jeu de données d’intégration** – Vue de données nommée qui pointe ou référence simplement les données à utiliser dans une activité en guise d’entrée et de sortie. Elle appartient à un service lié.

## <a name="next-steps"></a>Étapes suivantes

* [Bien démarrer avec Azure Synapse Analytics](get-started.md)
* [Créer un espace de travail](quickstart-create-workspace.md)
* [Utiliser un pool SQL serverless](quickstart-sql-on-demand.md)

