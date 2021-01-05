---
title: Glossaire Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Idées et concepts clés pour le service Metrics Advisor
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: mbullwin
ms.openlocfilehash: 6c461983053a145dfda58b9e3d26b39db0c339e5
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92893418"
---
# <a name="metrics-advisor-glossary-of-common-vocabulary-and-concepts"></a>Glossaire Metrics Advisor du vocabulaire et des concepts courants

Ce document explique les termes techniques utilisés dans Metrics Advisor. Utilisez cet article pour en savoir plus sur les concepts et objets courants que vous pouvez rencontrer lors de l’utilisation du service.

## <a name="data-feed"></a>Flux de données

> [!NOTE]
> Plusieurs mesures peuvent partager la même source de données ainsi que le même flux de données.

Un flux de données est ce que Metrics Advisor ingère de votre source de données, par exemple Cosmos DB ou SQL Server. Un flux de données contient des lignes de :
* horodatage
* zéro ou plusieurs dimensions
* une ou plusieurs mesures. 

## <a name="metric"></a>Métrique

Une métrique est une mesure quantifiable utilisée pour analyser et évaluer l’état d’un processus d’entreprise spécifique. Il peut s’agir d’une combinaison de plusieurs valeurs de séries chronologiques divisées en dimensions. Par exemple, une métrique d’ *intégrité du Web* peut contenir des dimensions pour le *nombre d’utilisateurs* et le *marché en-US* .

## <a name="dimension"></a>Dimension

Une dimension est une ou plusieurs valeurs catégoriques. Une combinaison de ces valeurs identifie une série chronologique particulière à une seule dimension, par exemple : pays, langue, abonné, etc.

## <a name="multi-dimensional-metric"></a>Métrique multidimensionnelle

Qu’est-ce qu’une métrique multidimentionnelle ? Utilisons deux exemples. 

**Chiffre d’affaires d’une entreprise**

Supposons que vous disposez de données relatives au chiffre d’affaires de votre entreprise. Vos données de série chronologique peuvent ressembler à ceci :

| Timestamp | Category | Marché | Chiffre d’affaires |
| ----------|----------|--------|----- |
| 1-6-2020 | Aliment | US | 1 000 |
| 1-6-2020 | Habillement | US | 2000 |
| 2-6-2020 | Aliment | Royaume-Uni | 800 | 
| ...      | ...  |... | ... |

Dans cet exemple, *Catégorie* et *Marché* sont des dimensions. *Chiffre d’affaires* est l’indicateur de performance clé (KPI) qui peut être découpé en différentes catégories et/ou marchés et peut également être agrégé. Par exemple, le chiffre d’affaires de la *nourriture* pour tous les marchés.
 
**Nombre d’erreurs pour une application complexe**

Supposons que vous avez des données pour le nombre d’erreurs journalisées dans une application. Vos données de série chronologique peuvent ressembler à ceci :

| Timestamp | Composant d'application | Region | Erreurs |
| ----------|----------|--------|----- |
| 1-6-2020 | Base de données des employés | Europe occidentale | 9000 |
| 1-6-2020 | File d’attente des messages | USA Est | 1 000 |
| 2-6-2020 | File d’attente des messages | USA Est | 8000| 
| ...      | ...           | ...     |  ...|

Dans cet exemple, *Composant d’application* et *Région* sont des dimensions. *Nombre d’erreurs* est l’indicateur de performance clé (KPI) qui peut être découpé en différentes catégories et/ou marchés et peut également être agrégé. Par exemple, le nombre d’erreurs de *file d’attente de messages* dans toutes les régions.

## <a name="measure"></a>Measure

Une mesure est un terme fondamental ou spécifique à une unité et une valeur quantifiable de la métrique.

## <a name="time-series"></a>Série chronologique

Une série chronologique est une série de points de données indexés (ou répertoriés ou représentés sous forme de graphique) dans l’ordre chronologique. Le plus souvent, une série chronologique est une séquence prise à des points successifs et espacés régulièrement dans le temps. Il s’agit d’une séquence de données en temps discret.

Dans Metrics Advisor, les valeurs d’une métrique sur une combinaison de dimensions spécifique sont appelées une série.

## <a name="granularity"></a>Granularité

La granularité indique la fréquence à laquelle les points de données sont générés au niveau de la source de données. Par exemple, tous les jours, toutes les heures.

## <a name="start-time"></a>Heure de début

L’heure de début est l’heure à laquelle Metrics Advisor doit commencer à ingérer des données de votre source de données. Votre source de données doit contenir des données à l’heure de début spécifiée.

## <a name="confidence-boundaries"></a>Limites de confiance

> [!NOTE]
> Les limites de confiance ne sont pas la seule mesure utilisée pour détecter des anomalies. Les points de données en dehors de cette limite peuvent être marqués comme normaux par le modèle de détection. 

Dans Metrics Advisor, les limites de confiance représentent la sensibilité de l’algorithme utilisé et sont utilisées pour filtrer les anomalies trop sensibles. Sur le portail Web, les limites de confiance apparaissent sous la forme d’une bande bleue transparente. Tous les points de la bande sont traités comme des points normaux.

Metrics Advisor fournit des outils pour ajuster la sensibilité des algorithmes utilisés. Consultez [Procédure : Configurez des métriques et affinez la configuration de détection ](how-tos/configure-metrics.md) pour plus d’informations.

![Limites de confiance](media/confidence-bounds.png)


## <a name="hook"></a>Hook

Metrics Advisor vous permet de créer des alertes en temps réel et de vous s’y abonner. Ces alertes sont envoyées via Internet, [à l’aide d’un hook](how-tos/alerts.md).

## <a name="anomaly-incident"></a>Incident d’anomalie

Après l’application d’une configuration de détection aux mesures, des incidents sont générés chaque fois qu’une série au sein de celle-ci présente une anomalie. Dans les jeux de données volumineux, cela peut s’avérer écrasant. Metrics Advisor regroupe alors une série d’anomalies au sein d’une mesure en un incident. Le service évalue également la gravité et fournit des outils pour [procéder au diagnostic de l’incident](how-tos/diagnose-incident.md).

### <a name="incident-tree"></a>Arborescence de l’incident

Dans Metrics Advisor, vous pouvez appliquer la détection d’anomalies sur les métriques. Metrics Advisor surveille alors automatiquement toutes les séries chronologiques de toutes les combinaisons de dimensions. En cas de détection d’anomalies, Metrics Advisor agrège les anomalies en incidents.
À l’issue d’un incident, Metrics Advisor fournira une arborescence d’incidents avec une hiérarchie d’anomalies de contribution et identifiera ceux qui ont le plus grand impact. Chaque incident a une anomalie de cause racine, qui est le nœud supérieur de l’arborescence.

### <a name="anomaly-grouping"></a>Regroupement d’anomalies

Metrics Advisor offre la possibilité de rechercher des séries chronologiques associées à des modèles similaires. Il peut également fournir des insights plus approfondis sur l’impact des autres dimensions et mettre en corrélation les anomalies.

### <a name="time-series-comparison"></a>Comparaison des séries chronologiques

Vous pouvez choisir plusieurs séries chronologiques pour comparer des tendances dans une visualisation unique. Vous disposez ainsi d’une méthode claire et précise pour afficher et comparer les séries associées.

## <a name="detection-configuration"></a>Configuration de détection

>[!Note]
>Les configurations de détection ne sont appliquées qu’au sein d’une mesure individuelle.

Sur le Portail web de Metrics Advisor, une configuration de détection (telle que la sensibilité, la répétition automatique et la direction) est indiquée dans le volet gauche lors de l’affichage d’une métrique. Les paramètres peuvent être réglés et appliqués à toutes les séries au sein de cette mesure.

Une configuration de détection est requise pour chaque série chronologique et détermine si un point dans la série chronologique est une anomalie. Metrics Advisor va configurer une configuration par défaut pour l’ensemble de la mesure lors de la première intégration des données. 

Vous pouvez également affiner la configuration en appliquant des paramètres de réglage sur un groupe de séries ou sur un groupe spécifique. Une seule configuration est appliquée à une série chronologique :
* Les configurations appliquées à une série spécifique remplacent les configurations d’un groupe
* Les configurations d’un groupe remplacent les configurations appliquées à l’ensemble de la mesure.

Metrics Advisor fournit plusieurs [méthodes de détection](how-tos/configure-metrics.md#anomaly-detection-methods) et vous pouvez les combiner à l’aide d’opérateurs logiques.

### <a name="smart-detection"></a>Détection intelligente

Détection d’anomalies à l’aide de plusieurs algorithmes de Machine Learning.

**Sensibilité**  : Valeur numérique pour ajuster la tolérance de la détection d’anomalie. Visuellement, plus la valeur est élevée, plus les limites supérieure et inférieure de la série chronologique sont étroites.

### <a name="hard-threshold"></a>Seuil définitif

Les valeurs en dehors des limites supérieure ou inférieure sont des anomalies.

**Min**  : Limite inférieure

**Max**  : Limite supérieure

### <a name="change-threshold"></a>Seuil de modification

Utilisez la valeur du point précédent pour déterminer si ce point est une anomalie.

**Pourcentage de modification**  : Comparé au point précédent, le point actuel est une anomalie si le pourcentage de la modification est supérieur à ce paramètre.

**Modification des points**  : Nombre de points à examiner.

### <a name="common-parameters"></a>Paramètres communs

**Direction**  : Un point est une anomalie uniquement lorsque l’écart se produit dans la direction *haut* , *bas* ou *les deux* .

**Anomalie non valide tant que**  : Un point de données n’est qu’une anomalie si un pourcentage spécifié de points précédents est également une anomalie.

## <a name="alert-settings"></a>Paramètres d’alerte

Les paramètres d’alerte déterminent les anomalies devant déclencher une alerte. Vous pouvez définir plusieurs alertes avec des paramètres différents. Par exemple, vous pouvez créer une alerte pour les anomalies ayant un impact plus faible et une autre pour des alertes de plus grande importance.

Vous pouvez également créer une alerte sur les mesures. Par exemple, une alerte qui n’est déclenchée que si deux mesures spécifiées comportent des anomalies.

### <a name="alert-scope"></a>Étendue de l’alerte

L’étendue de l’alerte fait référence à la portée de l’alerte. Quatre options s’offrent à vous :

**Anomalies de toutes les séries**  : Des alertes sont déclenchées pour les anomalies dans toutes les séries de la mesure.

**Anomalies dans le groupe de séries**  : Les alertes sont déclenchées uniquement pour les anomalies de dimensions spécifiques du groupe de séries. Le nombre de dimensions spécifiées doit être inférieur au nombre total de dimensions.

**Anomalies dans les séries de favoris**  : Les alertes sont déclenchées uniquement pour les anomalies ajoutées en tant que favoris. Vous pouvez choisir un groupe de séries en tant que favoris pour chaque configuration de détection.

**Anomalies dans les N premières séries**  : Les alertes sont déclenchées uniquement pour les anomalies dans les N premières séries. Vous pouvez définir des paramètres pour spécifier le nombre de timestamps à prendre en compte, ainsi que le nombre d’anomalies qu’ils doivent avoir pour envoyer l’alerte.

### <a name="severity"></a>Gravité

La gravité est un degré utilisé par Metrics Advisor pour décrire la gravité de l’incident, y compris *Élevé* , *Moyen* et *Faible* .

À l’heure actuelle, Metrics Advisor utilise les facteurs suivants pour mesurer la gravité de l’alerte :
1. La proportion de valeur et de quantité d’anomalies dans la mesure.
1. La confiance des anomalies.
1. Vos paramètres favoris contribuent également à la gravité.

### <a name="auto-snooze"></a>Répétition automatique

Certaines anomalies sont des problèmes temporaires, en particulier pour les mesures de granularité réduite. Vous pouvez *répéter* une alerte pour un nombre spécifique de points de temps. Si des anomalies se trouvent dans le nombre de points spécifié, aucune alerte n’est déclenchée. Le comportement de la répétition automatique peut être défini sur un niveau de métrique ou de série.

Le comportement de la répétition peut être défini sur un niveau de métrique ou de série.

## <a name="data-feed-settings"></a>Paramètres de flux de données

### <a name="ingestion-time-offset"></a>Décalage du temps d’ingestion

Par défaut, les données sont ingérées en fonction de la granularité (par exemple *tous les jours* ). En utilisant un entier positif, vous pouvez retarder l’ingestion des données par la valeur spécifiée. En utilisant un nombre négatif, vous pouvez avancer l’ingestion de la valeur spécifiée.

### <a name="max-ingestion-per-minute"></a>Ingestion maximale par minute

Définissez ce paramètre si votre source de données prend en charge une concurrence limitée. Sinon, conservez le paramètre par défaut.

### <a name="stop-retry-after"></a>Arrêter les nouvelles tentatives après

En cas d’échec de l’ingestion des données, Metrics Advisor réessaiera automatiquement après un certain laps de temps. Le début de la période correspond à l’heure d’occurrence de la première ingestion des données. La durée de la période de la nouvelle tentative est définie en fonction de la granularité. Si vous utilisez la valeur par défaut (`-1`), la période de nouvelle tentative sera déterminée en fonction de la granularité :

| Granularité       | Arrêter les nouvelles tentatives après           |
| :------------ | :--------------- |
| Tous les jours, Personnalisé (>= 1 jour), Toutes les semaines, Tous les mois, Tous les ans     | 7 jours          |
| Toutes les heures, Personnalisé (< 1 jour)       | 72 heures |

### <a name="min-retry-interval"></a>Intervalle de nouvelle tentative minimal

Vous pouvez spécifier l’intervalle minimal lorsque vous tentez de nouveau d’extraire des données de la source. Si vous utilisez la valeur par défaut (`-1`), l’intervalle entre les nouvelles tentatives sera déterminé en fonction de la granularité :

| Granularité       | Intervalle minimal de nouvelle tentative           |
| :------------ | :--------------- |
| Tous les jours, Personnalisé (>= 1 jour), Toutes les semaines, Tous les mois     | 30 minutes          |
| Toutes les heures, Personnalisé (< 1 jour)      | 10 minutes |
| Annuelle | 1 jour          |

### <a name="grace-period"></a>Période de grâce

> [!Note]
> La période de grâce commence à l’heure d’ingestion normale, plus le décalage de temps d’ingestion spécifié.
    
Une période de grâce est un laps de temps pendant lequel Metrics Advisor continue de récupérer (fetch) des données de la source de données, mais ne déclenche aucune alerte. Si aucune donnée n’a été ingérée après la période de grâce, une alerte *Flux de données non disponible* est déclenchée.

### <a name="snooze-alerts-in"></a>Répéter les alertes dans

Lorsque cette option a la valeur zéro, chaque timestamp à l’état *Non disponible* déclenche une alerte. Quand la valeur est différente de zéro, le nombre spécifié des alertes *Non disponible* sont répétées si aucune donnée n’a été extraite.

## <a name="data-feed-permissions"></a>Autorisations de flux de données

Il existe deux rôles pour gérer les autorisations de flux de données : *Administrateur* et *Viewer* . 

* Un *Administrateur* a le contrôle total du flux de données et des mesures qu’il contient. Ils peuvent activer, suspendre, supprimer le flux de données et effectuer des mises à jour vers des flux et des configurations. Un *Administrateur* est généralement le propriétaire des mesures.

* Un *Viewer* peut afficher le flux de données ou les mesures, mais n’est pas en mesure d’apporter des modifications. 

## <a name="next-steps"></a>Étapes suivantes
- [Présentation de Metrics Advisor](overview.md)
- [Utiliser le Portail web](quickstarts/web-portal.md)