---
title: Plateforme de données Azure Monitor | Microsoft Docs
description: La supervision des données collectées par Azure Monitor est divisée en plusieurs métriques légères et capables de prendre en charge des scénarios en temps quasi-réel ainsi que les journaux d’activité utilisés pour une analyse avancée.
documentationcenter: ''
author: bwren
manager: carmonm
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/26/2019
ms.author: bwren
ms.openlocfilehash: e87ddd243aa248b896a26e6389ac1a219579a06d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87325574"
---
# <a name="azure-monitor-data-platform"></a>Plateforme de données Azure Monitor

De nos jours, pour avoir une visibilité des environnements informatiques complexes exécutant des applications distribuées qui s’appuient sur des services cloud et locaux, il est nécessaire de collecter des données opérationnelles à partir de chaque niveau et composant du système distribué. Vous devez être en mesure d’obtenir des insights approfondis sur ces données et de les regrouper dans un point unique offrant différentes perspectives pour prendre en charge la multitude des parties prenantes de votre organisation.

[Azure Monitor](../overview.md) collecte et agrège les données à partir de diverses sources dans une plateforme de données commune où elles peuvent être utilisées à des fins d’analyse, de visualisation et d’alerte. Il procure une expérience cohérente à partir de données provenant de plusieurs sources, mettant à votre disposition des insights détaillés sur toutes vos ressources supervisées et même sur les données provenant d’autres services qui stockent leurs données dans Azure Monitor.


![Vue d’ensemble d’Azure Monitor](media/data-platform/overview.png)

## <a name="observability-data-in-azure-monitor"></a>Données de visibilité dans Azure Monitor
Les métriques, journaux et traces distribuées sont couramment considérés comme les trois piliers de la visibilité. Ce sont les différents types de données qu’un outil de supervision doit collecter et analyser pour fournir une visibilité suffisante d’un système supervisé. Il est possible d’obtenir la visibilité en mettant en corrélation les données de plusieurs piliers et en les agrégeant sur l’ensemble des ressources en cours de supervision. Étant donné qu’Azure Monitor stocke ensemble des données de sources multiples, les données peuvent être mises en corrélation et analysées à l’aide d’un ensemble d’outils commun. Il met également en corrélation les données de plusieurs abonnements et locataires Azure, en plus d’héberger des données d’autres services.

Les ressources Azure génèrent une quantité importante de données de supervision. Azure Monitor regroupe ces données ainsi que les données de supervision provenant d’autres sources dans une plateforme de métriques ou de journaux. Chacune est optimisée pour des scénarios de supervision particuliers et prend en charge différentes fonctionnalités dans Azure Monitor. Les fonctionnalités, telles que l’analyse des données, les visualisations ou la génération d’alertes, vous obligent à comprendre les différences pour pouvoir implémenter de manière aussi efficace et économique que possible le scénario applicable dans votre situation. Les insights dans Azure Monitor tels qu’[Application Insights](../app/app-insights-overview.md) ou [Azure Monitor pour machines virtuelles](../insights/vminsights-overview.md) mettent à votre disposition des outils d’analyse avec lesquels vous pouvez vous concentrer sur un scénario de supervision spécifique sans avoir à comprendre les différences entre les deux types de données. 


### <a name="metrics"></a>Mesures
Les [métriques](data-platform-metrics.md) sont des valeurs numériques décrivant un aspect d’un système à un moment précis dans le temps. Elles sont collectées à intervalles réguliers et sont identifiées avec un horodatage, un nom, une valeur et une ou plusieurs étiquettes de définition. Les métriques peuvent être agrégées à l’aide d’un éventail d’algorithmes, comparées à d’autres métriques et analysées à des fins de détection de tendances. 

Les métriques dans Azure Monitor sont stockées dans une base de données de série chronologique qui est optimisée pour l’analyse des données horodatées. Ainsi, les métriques sont particulièrement adaptées à la génération d’alertes et à la détection rapide de problèmes. Elles peuvent vous donner des informations sur le niveau de performance de votre système, mais elles doivent généralement être combinées avec les journaux pour identifier la cause racine des problèmes.

Des métriques sont disponibles pour l’analyse interactive dans le portail Azure avec [Azure Metrics Explorer](./metrics-getting-started.md). Vous pouvez les ajouter à un [tableau de bord Azure](../learn/tutorial-app-dashboards.md) à des fins de visualisation en combinaison avec d’autres données et les utiliser pour [générer des alertes](alerts-metric.md) en quasi-temps réel.

Pour en savoir plus sur les métriques Azure Monitor, notamment sur leurs sources de données, consultez [Métriques dans Azure Monitor](data-platform-metrics.md).

### <a name="logs"></a>Journaux d’activité
Les [journaux](data-platform-logs.md) sont les événements qui se sont produits au sein du système. Ils peuvent contenir différents types de données et être du texte structuré ou en forme libre avec un horodatage. Ils peuvent être créés par intermittence à mesure que les événements dans l’environnement génèrent des entrées de journal, et un système surchargé génère généralement un volume de journaux plus grand.

Dans Azure Monitor, les journaux sont stockés dans un espace de travail Log Analytics basé sur [Azure Data Explorer](/azure/data-explorer/), qui fournit un moteur d’analyse puissante et un [langage de requête riche](/azure/kusto/query/). En général, les journaux contiennent suffisamment d’informations pour fournir le contexte complet du problème identifié et sont utiles pour identifier la cause racine des problèmes.

> [!NOTE]
> Il est important de faire la distinction entre les journaux Azure Monitor et les sources des données de journal dans Azure. Par exemple, les événements de niveau abonnement dans Azure sont écrits dans un [journal d’activité](platform-logs-overview.md) que vous pouvez voir à partir du menu Azure Monitor. La plupart des ressources écrivent des informations opérationnelles dans un [journal de ressources](platform-logs-overview.md) que vous pouvez transférer à différents emplacements. Les journaux Azure Monitor constituent une plateforme de données de journal qui collecte des journaux d’activité et des journaux de ressources ainsi que d’autres données de supervision pour fournir une analyse approfondie de l’ensemble de vos ressources.


 Vous pouvez utiliser des [requêtes de journal](../log-query/log-query-overview.md) interactivement avec [Log Analytics](../log-query/log-query-overview.md) dans le portail Azure ou ajouter les résultats à un [tableau de bord Azure](../learn/tutorial-app-dashboards.md) à des fins de visualisation en combinaison avec d’autres données. Vous pouvez également créer des [alertes de journal](alerts-log.md) qui déclenchent une alerte basée sur les résultats d’une requête de planification.

Pour en savoir plus sur les journaux Azure Monitor, notamment sur leurs sources de données, consultez [Journaux dans Azure Monitor](data-platform-logs.md).

### <a name="distributed-traces"></a>Traces distribuées
Les traces sont des séries d’événements associés qui suivent une demande utilisateur par le biais d’un système distribué. Elles peuvent servir à déterminer le comportement du code d’application et les performances de différentes transactions. Bien que les journaux soient souvent créés par les composants individuels d’un système distribué, une trace mesure le fonctionnement et les performances de votre application sur l’ensemble des composants.

Le traçage distribué dans Azure Monitor est activé avec le [SDK Application Insights](../app/distributed-tracing.md), tandis que les données de trace sont stockées avec les autres données de journal d’application collectées par Application Insights. Elles sont ainsi disponibles pour les mêmes outils d’analyse que les autres données de journal, notamment les requêtes de journal, les tableaux de bord et les alertes.

Pour en savoir plus sur le traçage distribué, consultez [Présentation du traçage distribué](../app/distributed-tracing.md).


## <a name="compare-azure-monitor-metrics-and-logs"></a>Comparer les métriques et les journaux Azure Monitor

Le tableau suivant compare les métriques et les journaux dans Azure Monitor.

| Attribut  | Mesures | Journaux d’activité |
|:---|:---|:---|
| Avantages | Légères et capables de prendre en charge des scénarios en quasi-temps réel tels que la génération d’alerte. Idéales pour la détection rapide des problèmes. | Analysés avec un langage de requête riche. Idéaux pour une analyse approfondie et l’identification de la cause racine. |
| Données | Valeur numérique uniquement | Texte ou données numériques |
| Structure | Ensemble standard de propriétés, notamment l’heure de l’échantillonnage, la ressource en cours de supervision et une valeur numérique. Certaines métriques incluent plusieurs dimensions pour une définition plus précise. | Ensemble unique de propriétés en fonction du type de journal. |
| Collection | Collectées à intervalles réguliers. | Peuvent être collectés de façon sporadique à mesure que des événements déclenchent la création d’un enregistrement. |
| Affichage dans le portail Azure | Metrics Explorer | Log Analytics |
| Contenu des sources de données | Métriques de plateforme collectées à partir de ressources Azure.<br>Applications supervisées par Application Insights.<br>Personnalisation définie par l’API ou l’application. | Journaux des applications et des ressources.<br>Solutions de supervision.<br>Agents et extensions de machine virtuelle.<br>Demandes d’application et exceptions.<br>Azure Security Center.<br>API du collecteur de données. |

## <a name="collect-monitoring-data"></a>Collecter des données de supervision
Les différentes [sources de données pour Azure Monitor](data-sources.md) écrivent dans un espace de travail Log Analytics (journaux) et/ou dans la base de données de métriques Azure Monitor (métriques). Certaines sources écrivent directement dans ces magasins de données, tandis que d’autres peuvent écrire à un autre emplacement, tel que le stockage Azure, et nécessiter une configuration pour remplir les journaux ou métriques. 

Pour obtenir la liste des différentes sources de données qui remplissent chaque type, consultez [Métriques dans Azure Monitor](data-platform-metrics.md) et [Journaux dans Azure Monitor](data-platform-logs.md).


## <a name="stream-data-to-external-systems"></a>Transmettre des données en continu vers les systèmes externes
En plus d’utiliser les outils dans Azure pour analyser les données de supervision, vous aurez peut-être besoin de transférer ces données vers un outil externe, comme un produit SIEM (Security Information and Event Management). Ce transfert est le plus souvent effectué directement à partir des ressources supervisées au travers d’[Azure Event Hubs](../../event-hubs/index.yml). Certaines sources peuvent être configurées pour envoyer des données directement à un hub d’événements, tandis que vous pouvez utiliser un autre processus comme une application logique pour récupérer les données requises. Pour plus d’informations, consultez [Diffuser des données de supervision Azure vers un hub d’événements pour les utiliser dans un outil externe](stream-monitoring-data-event-hubs.md).



## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur les [métriques dans Azure Monitor](data-platform-metrics.md).
- Découvrez-en plus sur les [journaux dans Azure Monitor](data-platform-logs.md).
- Découvrez les [données de surveillance disponibles](data-sources.md) pour différentes ressources dans Azure.

