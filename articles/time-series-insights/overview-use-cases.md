---
title: Cas d’usage de Gen2 – Azure Time Series Insights Gen2 | Microsoft Docs
description: Découvrez des cas d’usage d’Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 12/16/2020
ms.custom: seodec18
ms.openlocfilehash: 7d97958c5fd1274495da88c064b63e59e354f691
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606961"
---
# <a name="azure-time-series-insights-gen2-use-cases"></a>Cas d’usage d’Azure Time Series Insights Gen2

Cet article récapitule plusieurs cas d’usage courants d’Azure Time Series Insights Gen2. Les recommandations de cet article constituent le point de départ pour développer vos applications et solutions avec Azure Time Series Insights Gen2.

Plus précisément, cet article répond aux questions suivantes :

* Quels sont les cas d’usage courants d’Azure Time Series Insights Gen2 ?
* Quels sont les avantages de l’utilisation d’Azure Time Series Insights Gen2 dans le cadre de l’[exploration des données et de la détection d’anomalies visuelles](#data-exploration-and-visual-anomaly-detection) ?
* Quels sont les avantages de l’utilisation d’Azure Time Series Insights Gen2 dans le cadre d’une [analyse opérationnelle et de l’efficacité des processus](#operational-analysis-and-driving-process-efficiency) ?
* Quels sont les avantages de l’utilisation d’Azure Time Series Insights Gen2 dans le cadre [d’analyses avancées](#advanced-analytics) ?

Ces scénarios sont décrits dans les sections suivantes.

## <a name="introduction"></a>Introduction

Azure Time Series Insights Gen2 est une offre PaaS de bout en bout. Elle est utilisée pour collecter, traiter, stocker, analyser et interroger des données à l’échelle IoT optimisées pour une série chronologique hautement contextualisée. Elle est idéale pour l’exploration de données ad hoc et l’analyse opérationnelle. Il s’agit d’une offre de service personnalisée et extensible de manière unique qui répond aux besoins étendus des déploiements IoT sectoriels.

## <a name="data-exploration-and-visual-anomaly-detection"></a>Exploration des données et détection d’anomalies visuelles

Explorez et analysez instantanément des milliards d’événements pour repérer les anomalies et découvrir les tendances cachées de vos données. Azure Time Series Insights Gen2 offre des performances en temps quasi réel pour vos charges de travail d’analyse IoT et DevOps.

[![Explorateur de données](media/v2-update-use-cases/data-explorer.png)](media/v2-update-use-cases/data-explorer.png#lightbox)

La plupart des clients s’accordent pour dire que l’une des fonctionnalités les plus intéressantes d’Azure Time Series Insights Gen2 est la rapidité avec laquelle les informations sont fournies :

* Azure Time Series Insights Gen2 ne requiert aucune préparation de données initiale.
* Très rapide, il vous connecte à des milliards d’événements dans votre environnement Azure IoT Hub ou d’instances Azure Event Hubs en quelques minutes.
* Une fois connecté, vous pouvez visualiser et analyser des milliards d’événements pour repérer les anomalies et découvrir les tendances cachées de vos données.

Azure Time Series Insights Gen2 est intuitif et simple à utiliser. Vous pouvez interagir avec vos données sans avoir à écrire une seule ligne de code. En outre, il n’est pas nécessaire d’apprendre un nouveau langage, bien qu’Azure Time Series Insights Gen2 fournisse un langage de requête texte précis pour les utilisateurs avancés qui maîtrisent SQL. Quant aux novices, ils peuvent utiliser la plateforme via un mode d’exploration simplifié.

Les clients tirent parti de la vitesse de diagnostic des problèmes liés aux ressources. Ils peuvent effectuer des analyses d’opérations de développement pour identifier la cause racine d’un bogue dans une solution IoT. Ils peuvent également identifier les zones à marquer pour investigation dans le cadre de leurs initiatives en science des données.

Il existe trois méthodes principales pour interagir avec les données stockées dans Azure Time Series Insights Gen2 :

* La méthode la plus simple pour commencer consiste à utiliser l’explorateur de la préversion d’Azure Time Series Insights Gen2. Vous pouvez l’utiliser pour visualiser rapidement toutes vos données IoT au même endroit. Il fournit des outils tels qu’une carte thermique pour vous aider à détecter les anomalies dans vos données. Il propose également une vue en perspective. Vous pouvez l’utiliser pour comparer jusqu’à quatre vues provenant d’un ou plusieurs environnements Azure Time Series Insights Gen2 dans un tableau de bord unique. Le tableau de bord vous donne une vue d’ensemble des données de série chronologique de tous vos emplacements. Apprenez-en davantage sur l’[Explorateur Azure Time Series Insights Gen2](./concepts-ux-panels.md). Pour planifier votre environnement, consultez [Azure Time Series Insights Gen2 planning](./how-to-plan-your-environment.md) (Planification d’Azure Time Series Insights Gen2).

* La deuxième méthode consiste à utiliser le kit SDK JavaScript pour incorporer rapidement de puissants graphiques et graphes dans votre application web. Avec seulement quelques lignes de code, vous pouvez créer des requêtes puissantes. Utilisez-les pour remplir les graphiques en courbes, les graphiques à secteurs, les graphiques à barres, les cartes thermiques, les grilles de données et bien plus encore. Tous ces éléments sont prêts à l’emploi dans le kit de développement logiciel (SDK). Le kit de développement logiciel (SDK) extrait également les API de requête Azure Time Series Insights Gen2. Vous pouvez les utiliser pour créer des prédicats de type SQL afin d’interroger les données que vous souhaitez afficher dans un tableau de bord. Pour les solutions hybrides de couche présentation, Azure Time Series Insights Gen2 propose des URL paramétrables. Elles fournissent des points de connexion transparents avec l’explorateur de la préversion d’Azure Time Series Insights Gen2 pour des analyses détaillées des données.

  * Pour en savoir plus sur le SDK JavaScript, consultez la documentation de la [bibliothèque de client JS](https://github.com/microsoft/tsiclient/blob/master/docs/API.md) et de [l’Exemple de client](https://github.com/Microsoft/tsiclient).

  * Pour en savoir plus sur le partage des URL et la nouvelle interface utilisateur, consultez [Visualiser les données dans l’explorateur Azure Time Series Insights Gen2](./concepts-ux-panels.md).

* La troisième méthode consiste à utiliser les puissantes API pour interroger les données stockées dans Azure Time Series Insights Gen2. Azure Time Series Insights Gen2 utilise des opérateurs temporels tels que `from`, `to`, `first` et `last`. Il dispose de transformations et d'agrégations telles que `average`, `sum`, `min`, `max`, `time-weighted average`, `time-weighted sum`, etc. Il permet également l’utilisation du filtrage, des opérateurs arithmétiques et booléens, des fonctions scalaires, etc. Tous ces opérateurs permettent aux applications en aval d’identifier rapidement des tendances et des modèles intéressants dans vos données. Utilisez-les pour remplir les visualisations personnalisées afin de détecter les anomalies.

## <a name="operational-analysis-and-driving-process-efficiency"></a>Analyse opérationnelle et efficacité du processus de pilotage

Utilisez Azure Time Series Insights Gen2 pour surveiller l’intégrité, l’utilisation et les performances des équipements à grande échelle et mesurer l’efficacité opérationnelle. Il permet de gérer des charges de travail IoT diverses et imprévisibles sans nuire aux performances d’ingestion ou de requête.

[![Capture d’écran montrant les appareils IoT et les données d’application, le traitement de flux, l’efficacité opérationnelle, l’intelligence/les insights et l’analytique avancée dans Azure Time Series Insights Gen2.](media/v2-update-use-cases/overview.png)](media/v2-update-use-cases/overview.png#lightbox)

La diffusion et le traitement en continu de données provenant des processus opérationnels peuvent être avantageux pour n’importe quelle activité s’ils sont associés à la technologie ou à la solution appropriée. Souvent, ces solutions sont une combinaison de plusieurs systèmes. Elles permettent l’exploration et l’analyse des données qui évoluent en permanence, en particulier en ce qui concerne l’IoT, et partagent un modèle commun.

Ces modèles reposent souvent sur des plateformes compatibles avec l’IoT qui ingèrent des milliards d’événements provenant d’appareils et de capteurs qui regroupent différents paramètres régionaux. Ces systèmes traitent et analysent les données en continu pour générer des informations et des actions pertinentes en temps réel. Les données sont généralement archivées en stockage chaud ou froid pour des analyses par lots en temps quasi réel.

Les données collectées sont soumises à une série de traitements visant à les nettoyer et à les contextualiser dans le cadre de scénarios de requêtes et d’analyses en aval. Azure propose des services enrichis qui peuvent être appliqués à des scénarios IoT tels que la maintenance et la création de ressources. Ces services incluent Azure Time Series Insights Gen2, IoT Hub, Event Hubs, Azure Stream Analytics, Azure Functions, Azure Logic Apps, Azure Databricks, Azure Machine Learning et Power BI.

L’architecture de la solution peut être obtenue de la manière suivante :

* Ingérez des données via IoT Hub ou Event Hubs pour une sécurité, un débit et une latence de qualité optimale.
* Effectuez le traitement des données et les calculs. Synthétisez les données ingérées via des services tels que Stream Analytics, Logic Apps et Azure Functions. Le service que vous utilisez varie selon les besoins spécifiques en matière de traitement de données.
* Les signaux calculés à partir du pipeline de traitement sont envoyés à Azure Time Series Insights Gen2 pour stockage et analyse.

Azure Time Series Insights Gen2 permet d’explorer les données en temps quasi réel et fournit des analyses basées sur les ressources par rapport aux données historiques. Selon les besoins de votre entreprise, les travaux MapReduce et Hive peuvent être exécutés sur les données stockées dans Azure Time Series Insights Gen2 en connectant Azure Time Series Insights Gen2 à Azure HDInsight. Les données stockées dans Azure Time Series Insights Gen2 sont disponibles pour Power BI et d’autres applications client par le biais des API de requête de surface publique Azure Time Series Insights Gen2. Ces données peuvent être utilisées pour les scénarios d’activité détaillée et d’intelligence opérationnelle.

## <a name="advanced-analytics"></a>Analytique avancée

Bénéficiez d’une intégration avec des services analytiques avancés tels que Machine Learning et Azure Databricks. Azure Time Series Insights Gen2 saisit des données brutes provenant de millions d’appareils. Il ajoute des données contextuelles qui peuvent être utilisées en toute transparence par une suite de services Azure Analytics.

[![Analyses](media/v2-update-use-cases/advanced-analytics.png)](media/v2-update-use-cases/advanced-analytics.png#lightbox)

Les analyses avancées et l’apprentissage machine consomment et traitent de grands volumes de données. Ces données sont utilisées pour prendre des décisions reposant sur les données et pour réaliser des analyses prédictives. Dans le cadre de l’IoT, les algorithmes d’analyse avancée s’enrichissent à partir de données collectées via des millions d’appareils. Ces appareils transmettent des données plusieurs fois par seconde. Les données collectées à partir d’appareils IoT sont brutes. Elles ne disposent pas d’informations contextuelles telles que l’emplacement de l’appareil et l’unité de lecture du capteur. Par conséquent, ces données brutes sont difficiles à exploiter directement dans le cadre d’analyses avancées.

Pour combler les lacunes des données IoT pour effectuer des analyses avancées, Azure Time Series Insights Gen2 a recours à deux méthodes simples et économiques :

* Tout d’abord, Azure Time Series Insights Gen2 collecte les données de télémétrie brutes provenant de millions d’appareils à l’aide de IoT Hub. Il enrichit ces données avec des informations contextuelles, puis les transforme en format PARQUET. Ce format peut facilement être intégré à d’autres services d’analyse avancée, tels que Machine Learning, Azure Databricks et d’autres applications tierces.

    Azure Time Series Insights Gen2 peut servir de source fiable pour toutes les données au sein d’une organisation. Il crée un référentiel central pour les charges de travail en aval à utiliser. Étant donné qu’Azure Time Series Insights Gen2 est un service de stockage en temps quasi réel, les modèles d’analyse avancée peuvent s’enrichir en permanence à partir des données de télémétrie IoT entrantes. Par conséquent, les modèles peuvent effectuer des prévisions plus précises.

* En outre, la sortie des modèles Machine Learning et de prédiction peut être injectée dans Azure Time Series Insights Gen2 pour visualiser et stocker leurs résultats. Cette procédure permet aux organisations d’optimiser et d’ajuster leurs modèles. Azure Time Series Insights Gen2 vous permet de facilement visualiser les données de télémétrie sur le même plan que les résultats du modèle qui a bénéficié de l’apprentissage. De cette façon, il permet aux équipes de science des données d’identifier les anomalies et les modèles.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur l’[Explorateur Azure Time Series Insights Gen2](./concepts-ux-panels.md).
* Lisez les [Meilleures pratiques pour Azure Time Series Insights Gen2](./how-to-plan-your-environment.md) pour planifier votre environnement.
* Lisez la documentation de [l’exemple de client](https://github.com/Microsoft/tsiclient).
