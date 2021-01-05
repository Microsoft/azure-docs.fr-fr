---
title: Interrogation des données – Azure Time Series Insights Gen2 | Microsoft Docs
description: Concepts de l’interrogation de données et vue d’ensemble de l’API REST dans Azure Time Series Insights Gen2.
author: shreyasharmamsft
ms.author: shresha
manager: dpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/01/2020
ms.custom: seodec18
ms.openlocfilehash: 5682cf9856ad8969d930d72f53e888b03ff6ae75
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020859"
---
# <a name="querying-data-from-azure-time-series-insights-gen2"></a>Interrogation des données à partir d’Azure Time Series Insights Gen2

Azure Time Series Insights Gen2 permet d’interroger les données sur les événements et les métadonnées stockées dans l’environnement via des API de surface publiques. Ces API sont également utilisées par l’[explorateur Azure Time Series Insights TSI](./concepts-ux-panels.md).

Trois catégories principales d’API sont disponibles dans Azure Time Series Insights Gen2 :

* **API d’environnement** : ces API activent les requêtes sur l’environnement Azure Time Series Insights Gen2 proprement dit. Elles peuvent être utilisées pour recueillir la liste des environnements auxquels l’appelant a accès et les métadonnées de l’environnement.
* **API modèle-requête de série chronologique (TSM-Q)** : opérations CRUD (création, lecture, mise à jour et suppression) sur les métadonnées stockées dans le modèle Time Series de l’environnement. Celles-ci peuvent être utilisées pour accéder aux instances, types et hiérarchies, et les modifier.
* **API de requête de série chronologique (TSQ)** : permet la récupération des données de télémétrie ou d’événements au fur et à mesure de leur enregistrement à partir du fournisseur source et permet des calculs performants et des agrégations sur les données à l’aide de fonctions scalaires et d’agrégation avancées.

Azure Time Series Insights Gen2 utilise un langage d’expression enrichi basé sur des chaînes, [Time Series Expression (TSX)](/rest/api/time-series-insights/reference-time-series-expression-syntax), pour exprimer des calculs en [variables de série chronologique](./concepts-variables.md).

## <a name="azure-time-series-insights-gen2-apis-overview"></a>Présentation des API d’Azure Time Series Insights Gen2

Les API principales suivantes sont prises en charge.

[![Vue d'ensemble de la requête de série chronologique](media/v2-update-tsq/tsq.png)](media/v2-update-tsq/tsq.png#lightbox)

## <a name="environment-apis"></a>API d’environnement

* [API Obtenir les environnements](/rest/api/time-series-insights/management(gen1/gen2)/accesspolicies/listbyenvironment) : retourne la liste des environnements auxquels l’appelant est autorisé à accéder.
* [API Obtenir la disponibilité des environnements](/rest/api/time-series-insights/dataaccessgen2/query/getavailability) : retourne la distribution du nombre d’événements au cours du timestamp `$ts` de l’événement. Cette API permet de déterminer s’il s’est produit des événements dans l’environnement en renvoyant le nombre d’événements scindés en intervalles de temps, le cas échéant.
* [API Obtenir le schéma des événements](/rest/api/time-series-insights/dataaccessgen2/query/geteventschema) : retourne les métadonnées du schéma des événements pour une période de recherche donnée. Cette API permet de récupérer toutes les métadonnées et propriétés disponibles dans le schéma sur une période définie.

## <a name="time-series-model-query-tsm-q-apis"></a>API modèle-requête de série chronologique (TSM-Q)

La plupart de ces API prennent en charge l’opération d’exécution par lot pour activer les opérations CRUD par lot sur plusieurs entités de modèle Time Series :

* [API Paramètres du modèle](/rest/api/time-series-insights/reference-model-apis) : active *OBTENIR* et *CORRIGER* sur le type par défaut et le nom du modèle de l’environnement.
* [API Types](/rest/api/time-series-insights/reference-model-apis#types-api) : active CRUD sur les types Time Series et les variables associées.
* [API Hiérarchies](/rest/api/time-series-insights/reference-model-apis#hierarchies-api) : active CRUD sur les hiérarchies Time Series et les chemins d’accès de champs associés.
* [API Instances](/rest/api/time-series-insights/reference-model-apis#instances-api) : active CRUD sur les instances Time Series et les champs d’instance associés. En outre, l’API Instances prend en charge les opérations suivantes :
  * [Rechercher](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/search) : récupère une liste partielle des résultats d’une recherche d’instances de série chronologique en fonction des attributs d’instance.
  * [Suggérer](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/suggest) : recherche et suggère une liste partielle des résultats d’une recherche d’instances de série chronologique en fonction des attributs d’instance.

## <a name="time-series-query-tsq-apis"></a>API de requête de série chronologique (TSQ)

Ces API sont disponibles dans les deux magasins (chaud et froid) de notre solution de stockage multicouche. Les paramètres URL de requête permettent de spécifier le [type de stockage](/rest/api/time-series-insights/dataaccessgen2/query/execute#uri-parameters) sur lequel la requête doit s’exécuter :

* [API Obtenir les événements](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents) : permet l’interrogation et l’extraction d’événements bruts et des timestamps d’événements associés tels qu’ils sont enregistrés dans Azure Time Series Insights Gen2 à partir du fournisseur de source. Cette API permet de récupérer des événements bruts pour un ID Time Series donné et une étendue de recherche. Cette API prend en charge la pagination pour récupérer le jeu de données de réponse complet pour l’entrée sélectionnée.

  > [!IMPORTANT]

  > * Dans le cadre des [modifications à venir des règles de mise à plat et d’échappement au format JSON](./ingestion-rules-update.md), les tableaux sont stockés sous le type **Dynamique**. Les propriétés de charge utile stockées sous ce type sont **accessibles UNIQUEMENT par le biais de l’API d’extraction d’événements**.

* [API Obtenir les séries](/rest/api/time-series-insights/dataaccessgen2/query/execute#getseries) : Active l’interrogation et la récupération des valeurs calculées et des timestamps d’événements associés en appliquant des calculs définis par des variables sur des événements bruts. Ces variables peuvent être définies dans le modèle Time Series ou fournies inline dans la requête. Cette API prend en charge la pagination pour récupérer le jeu de données de réponse complet pour l’entrée sélectionnée.

* [API Agréger les séries](/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries) : permet l’interrogation et la récupération des valeurs agrégées et des timestamps d’intervalle associés en appliquant des calculs définis par des variables sur des événements bruts. Ces variables peuvent être définies dans le modèle Time Series ou fournies inline dans la requête. Cette API prend en charge la pagination pour récupérer le jeu de données de réponse complet pour l’entrée sélectionnée.
  
  Pour une étendue de recherche et un intervalle spécifiés, cette API renvoie une réponse agrégée par intervalle et par variable pour un ID de série chronologique. Le nombre d’intervalles dans le jeu de données de réponse est calculé en comptant les cycles d’époque (le nombre de millisecondes qui se sont écoulées depuis l’époque UNIX - 1er janvier 1970) et en divisant les cycles par la taille de l’intervalle de temps spécifiée dans la requête.

  Les timestamps retournés dans le jeu de réponses sont des limites de l’intervalle gauche, et non des événements échantillonnés à partir de l’intervalle.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les différentes variables qui peuvent être définies dans le [modèle Time Series](./concepts-model-overview.md).
* Découvrez-en plus sur l’interrogation des données à partir de l’[Explorateur Azure Time Series Insights](./concepts-ux-panels.md).