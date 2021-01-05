---
title: Modèle de données de dépendance Azure Monitor Application Insights
description: Modèle de données Application Insights pour la télémétrie des dépendances
ms.topic: conceptual
ms.date: 04/17/2017
ms.reviewer: sergkanz
ms.openlocfilehash: 0f9fc96479569c3411024068ed614d422035ab17
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87315969"
---
# <a name="dependency-telemetry-application-insights-data-model"></a>Télémétrie des dépendances : modèle de données Application Insights

La télémétrie des dépendances (dans [Application Insights](./app-insights-overview.md)) représente une interaction du composant surveillé avec un composant distant tel que SQL ou un point de terminaison HTTP.

## <a name="name"></a>Name

Nom de la commande lancée par cet appel de dépendance. Valeur de faible cardinalité. Exemples : nom de procédure stockée et modèle de chemin d’accès d’URL.

## <a name="id"></a>id

Identificateur d’une instance d’appel de dépendance. Utilisé pour la corrélation avec l’élément de télémétrie de demande correspondant à cet appel de dépendance. Pour plus d’informations, consultez la page relative à la [corrélation](./correlation.md).

## <a name="data"></a>Données

Commande lancée par cet appel de dépendance. Exemples : instruction SQL et URL HTTP avec tous les paramètres de requête.

## <a name="type"></a>Type

Nom du type de dépendance. Valeur de faible cardinalité pour le regroupement logique de dépendances et l’interprétation d’autres champs comme commandName et resultCode. Exemples : SQL, table Azure et HTTP.

## <a name="target"></a>Cible

Site cible d’un appel de dépendance. Exemples : nom de serveur, adresse d’hôte. Pour plus d’informations, consultez la page relative à la [corrélation](./correlation.md).

## <a name="duration"></a>Duration

Durée de la requête au format : `DD.HH:MM:SS.MMMMMM`. Doit être inférieure à `1000` jours.

## <a name="result-code"></a>Code de résultat

Code de résultat d’un appel de dépendance. Exemples : code d’erreur SQL et code d’état HTTP.

## <a name="success"></a>Succès

Indication de la réussite ou non d’un appel.

## <a name="custom-properties"></a>Propriétés personnalisées

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>Mesures personnalisées

[!INCLUDE [application-insights-data-model-measurements](../../../includes/application-insights-data-model-measurements.md)]


## <a name="next-steps"></a>Étapes suivantes

- Configurez le suivi des dépendances pour [.NET](./asp-net-dependencies.md).
- Configurez le suivi des dépendances pour [Java](./java-agent.md).
- [Écrire des données de télémétrie des dépendances personnalisées](./api-custom-events-metrics.md#trackdependency)
- Pour connaître les types et les modèles de données Application Insights, consultez [Modèle de données](data-model.md).
- Découvrez quelles [plateformes](./platforms.md) sont prises en charge par Application Insights.

