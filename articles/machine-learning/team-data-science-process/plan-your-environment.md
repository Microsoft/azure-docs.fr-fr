---
title: Identifier les scénarios et planifier le processus d’analyse – Processus TDSP (Team Data Science Process) | Azure Machine Learning
description: Identifier les scénarios et planifier le traitement des données d’analyse avancée en répondant à une série de questions pertinentes.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: ffe1a4b9b32223cc57c6a6399fdc074a9d51d714
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322399"
---
# <a name="how-to-identify-scenarios-and-plan-for-advanced-analytics-data-processing"></a>Comment identifier les scénarios et planifier le traitement des données d’analyse avancée

Quelles sont les ressources requises pour créer un environnement pouvant effectuer un traitement analytique avancé sur un jeu de données ? Cet article propose une série de questions qui peuvent aider à identifier des tâches et des ressources pertinentes pour votre scénario.

Pour en savoir plus sur l’ordre des étapes principales pour l’analyse prédictive, consultez [Qu’est-ce que le processus TDSP (Team Data Science Process) ?](overview.md). Chaque étape nécessite des ressources spécifiques pour les tâches correspondant à votre scénario.

Répondez à des questions clés dans les domaines suivants pour identifier votre scénario :

* logistique des données
* caractéristiques des données
* qualité du jeu de données
* outils et langages favoris

## <a name="logistic-questions-data-locations-and-movement"></a>Questions logistiques : emplacements et déplacement des données

Les questions logistiques couvrent les éléments suivants :

* emplacement de la source de données
* destination cible dans Azure
* configuration requise pour le déplacement des données, notamment la planification, la quantité et les ressources impliquées

Vous pouvez avoir à déplacer les données plusieurs fois pendant le processus d’analyse. Un scénario courant consiste à déplacer les données locales dans un système de stockage sur Azure, puis dans Machine Learning Studio.

### <a name="what-is-your-data-source"></a>Quelle est votre source de données ?

Vos données sont-elles locales ou dans le cloud ? Les emplacements suivants sont possibles :

* une adresse HTTP publique
* un emplacement de fichier local ou réseau
* une base de données SQL Server
* un conteneur Stockage Azure

### <a name="what-is-the-azure-destination"></a>Qu’est-ce que la destination Azure ?

Où vos données doivent-elles se trouver pour le traitement ou la modélisation ? 

* Stockage Blob Azure
* Bases de données SQL Azure
* SQL Server sur une machine virtuelle Azure
* HDInsight (Hadoop sur Azure) ou tables Hive
* Azure Machine Learning
* Disques durs virtuels Azure montables

### <a name="how-are-you-going-to-move-the-data"></a>Comment allez-vous déplacer les données ?

Pour les procédures et les ressources d’ingestion ou de chargement des données dans différents environnements de stockage et de traitement, consultez :

* [Charger des données dans des environnements de stockage à des fins d’analyse](ingest-data.md)
* [Importer vos données d’entraînement dans Azure Machine Learning Studio (classique) à partir de différentes sources de données](../classic/import-data.md)

### <a name="does-the-data-need-to-be-moved-on-a-regular-schedule-or-modified-during-migration"></a>Les données doivent-elles être déplacées à intervalles réguliers ou modifiées pendant la migration ?

Envisagez d’utiliser Azure Data Factory (ADF) lorsque les données doivent être migrées en permanence. ADF peut être dans les cas suivants :

* scénario hybride impliquant à la fois des ressources locales et de cloud
* scénario où les données sont traitées, modifiées ou changées par la logique métier au cours de la migration

Pour plus d’informations, référez-vous à [Déplacement de données à partir d’une base de données SQL Server vers SQL Azure avec Azure Data Factory](move-sql-azure-adf.md).

### <a name="how-much-of-the-data-is-to-be-moved-to-azure"></a>Quelle est la quantité de données à déplacer vers Azure ?

Des jeux de données volumineux peuvent dépasser la capacité de stockage de certains environnements. Pour obtenir un exemple, consultez la discussion sur les limites de taille pour Machine Learning Studio (classique) dans la section suivante. Dans ces cas, vous pouvez utiliser un échantillon de données pendant l’analyse. Pour plus d’informations sur l’échantillonnage d’un jeu de données dans différents environnements Azure, référez-vous à [Échantillonner des données dans le processus TDSP (Team Data Science Process)](sample-data.md).

## <a name="data-characteristics-questions-type-format-and-size"></a>Questions sur les caractéristiques des données : type, format et taille

Ces questions sont essentielles pour la planification de vos environnements de stockage et de traitement. Elles vous aideront à choisir le scénario approprié pour votre type de données et à comprendre les restrictions.

### <a name="what-are-the-data-types"></a>Quels sont les types de données ?

* Numérique
* Par catégorie
* Chaînes
* Binary

### <a name="how-is-your-data-formatted"></a>Quel est le format de vos données ?

* Fichiers plats séparés par des virgules (CSV) ou séparées par une tabulation (TSV)
* Compressés ou non
* Objets blob Azure
* Tables Hadoop Hive
* Tables SQL Server

### <a name="how-large-is-your-data"></a>Quel volume vos données représentent-elles ?

* Faible : moins de 2 Go
* Moyenne : plus de 2 Go et moins de 10 Go
* Grand : supérieur à 10 Go

Prenons, par exemple, l’environnement Azure Machine Learning Studio (classique) :

* Pour obtenir la liste des formats et des types de données pris en charge par Azure Machine Learning Studio, référez-vous à la section [Formats et types de données pris en charge](../classic/import-data.md#supported-data-formats-and-data-types) .
* Pour plus d’informations sur les limitations des autres services Azure utilisés dans le processus d’analyse, référez-vous à [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="data-quality-questions-exploration-and-pre-processing"></a>Questions sur la qualité des données : exploration et prétraitement

### <a name="what-do-you-know-about-your-data"></a>Que savez-vous sur vos données ?

Comprendre les caractéristiques de base de vos données :

* Modèles ou tendances reconnaissables
* Valeurs hors norme qu’elles contiennent
* Nombre de valeurs manquantes

Cette étape est importante pour vous aider :

* à déterminer la quantité de traitement préliminaire nécessaire ;
* à formuler des hypothèses qui suggèrent les fonctionnalités ou le type d’analyse les plus appropriés ;
* à élaborer des plans de collecte de données supplémentaires.

Des techniques utiles d’inspection des données incluent le calcul statistique descriptif et des tracés de visualisation. Pour plus d’informations sur l’exploration d’un jeu de données dans différents environnements Azure, référez-vous à [Explorer les données du processus TDSP (Team Data Science Process)](explore-data.md).

### <a name="does-the-data-require-preprocessing-or-cleaning"></a>Les données nécessitent-elles un prétraitement ou un nettoyage ?

Vous devrez peut-être prétraiter et nettoyer vos données avant de pouvoir utiliser efficacement le jeu de données pour le Machine Learning. Les données brutes sont souvent bruyantes et peu fiables. Il est possible qu’il manque des valeurs. Leur utilisation pour la modélisation peut générer des résultats trompeurs. Pour obtenir une description, référez-vous à [Tâches de préparation des données pour l’apprentissage automatique amélioré](prepare-data.md).

## <a name="tools-and-languages-questions"></a>Questions sur les outils et les langues

Il existe de nombreuses options pour les langages, les environnements de développement et les outils. Soyez attentifs à vos besoins et à vos préférences.

### <a name="what-languages-do-you-prefer-to-use-for-analysis"></a>Quels langages préférez-vous utiliser pour l’analyse ?

* R
* Python
* SQL

### <a name="what-tools-should-you-use-for-data-analysis"></a>Quels outils devez-vous utiliser pour l’analyse des données ?

* [Microsoft Azure PowerShell](/powershell/azure/) : un langage de script utilisé pour administrer vos ressources Azure dans un langage de script
* [Azure Machine Learning Studio](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
* [Revolution Analytics](https://www.microsoft.com/sql-server/machinelearningserver)
* [RStudio](https://www.rstudio.com)
* [Python Tools for Visual Studio](/visualstudio/python/)
* [Anaconda](https://www.anaconda.com/)
* [Blocs-notes Jupyter](https://jupyter.org/)
* [Microsoft Power BI](https://powerbi.microsoft.com)

## <a name="identify-your-advanced-analytics-scenario"></a>Identification de votre scénario d’analyse avancée

Après avoir répondu aux questions de la section précédente, vous êtes prêt à déterminer le scénario qui vous convient le mieux. Les exemples de scénarios sont présentés dans la rubrique [Scénarios d’analyses avancées dans Azure Machine Learning](plan-sample-scenarios.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Qu’est-ce que le processus TDSP (Team Data Science Process) ?](overview.md)