---
title: Superviser l’utilisation du débit d’une opération dans Azure Cosmos DB
description: Découvrez comment surveiller le débit ou l’utilisation des unités de requête d’une opération dans Azure Cosmos DB. Les propriétaires d’un compte Azure Cosmos DB peuvent déterminer les opérations qui nécessitent le plus d’unités de requête.
ms.service: cosmos-db
ms.topic: how-to
author: kanshiG
ms.author: govindk
ms.date: 04/09/2020
ms.openlocfilehash: d6f0f30d7437b6f8ecc1d915eb3d3195f2504fec
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93098244"
---
# <a name="how-to-monitor-throughput-or-request-unit-usage-of-an-operation-in-azure-cosmos-db"></a>Comment surveiller le débit ou l’utilisation des unités de requête d’une opération dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Monitor pour Azure Cosmos DB fournit une vue de métriques pour surveiller votre compte et créer des tableaux de bord. Cette fonctionnalité ne vous oblige pas à activer ou à configurer quoi que ce soit explicitement, car ces métriques Azure Cosmos DB sont collectées par défaut. La métrique **Unités de requête totales** est utilisée pour obtenir l’utilisation des unités de requête pour différents types d’opérations. Vous pouvez analyser ultérieurement les opérations qui ont utilisé la majeure partie du débit. Par défaut, les données de débit sont agrégées à un intervalle d’une minute. Vous pouvez cependant changer l’unité d’agrégation en modifiant l’option de granularité temporelle.

Vous pouvez analyser les données d’utilisation des unités de requête de deux façons :

* Dans l’intervalle de temps donné, déterminer les opérations qui consomment le plus d’unités de requête.
* Déterminer quelles opérations dominent généralement dans votre charge de travail en consommant le plus d’unités de requête.
Cette analyse vous permet de vous concentrer sur des opérations comme l’insertion, l’upsert et la recherche dans leur indexation. Vous pouvez déterminer si vous êtes sur ou sous des champs spécifiques de l’indexation, et modifier la [stratégie d’indexation](index-policy.md#include-exclude-paths) pour inclure ou exclure les chemins.

Si vous remarquez que certaines requêtes consomment davantage d’unités de requête, vous pouvez effectuer des actions comme :

* Reconsidérer si vous demandez la quantité de données appropriée.
* Modifier la requête pour utiliser l’index avec une clause de filtre.
* Effectuer moins d’appels de fonction définie par l’utilisateur consommant beaucoup de ressources.
* Définir des clés de partition pour minimiser le déploiement des requêtes dans différentes partitions.
* Vous pouvez également utiliser les métriques des requêtes retournées dans la réponse des appels et les détails du journal de diagnostic, et vous reporter à l’article sur l’[optimisation des performances des requêtes](sql-api-query-metrics.md) pour plus d’informations sur l’exécution des requêtes.
* Vous pouvez commencer en partant de la somme, puis examiner l’utilisation moyenne en utilisant la dimension appropriée.

## <a name="view-the-total-request-unit-usage-metric"></a>Visualiser la métrique d’utilisation totale des unités de requête

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Surveillance** dans la barre de navigation gauche, puis sélectionnez **Métriques**.

   :::image type="content" source="./media/monitor-request-unit-usage/monitor-metrics-blade.png" alt-text="Volet Métriques dans Azure Monitor":::

1. À partir du volet **Métriques** > **Sélectionner une ressource** > choisissez l’ **abonnement** exigé, puis **Groupe de ressources**. Pour le **type de ressource** , sélectionnez **Comptes Azure Cosmos DB** , choisissez une de vos comptes Azure Cosmos existants, puis sélectionnez **Appliquer**.

   :::image type="content" source="./media/monitor-request-unit-usage/select-cosmos-db-account.png" alt-text="Choisir le compte Azure Cosmos DB pour visualiser les métriques":::

1. Sélectionnez ensuite la métrique **Unités de requête totales** dans la liste des métriques disponibles. Pour découvrir de plus près toutes les métriques disponibles dans cette liste, consultez l’article [Métriques par catégorie](monitor-cosmos-db-reference.md). Dans cet exemple, nous allons sélectionner **Unités de requête totales** et **Moyenne** comme valeur d’agrégation. En plus de ces détails, vous pouvez également sélectionner l’ **intervalle de temps** et la **granularité temporelle** des métriques. Au maximum, vous pouvez voir les métriques des 30 derniers jours.  Une fois que vous avez appliqué le filtre, un graphique s’affiche. Vous pouvez voir le nombre moyen d’unités de requête consommées par minute pendant la période sélectionnée.  

   :::image type="content" source="./media/monitor-request-unit-usage/request-unit-usage-metric.png" alt-text="Choisir une métrique à partir du portail Azure":::

## <a name="filters-for-request-unit-usage"></a>Filtres l’utilisation des unités de requête

Vous pouvez aussi filtrer les métriques et obtenir des graphiques affichés en fonction d’une valeur spécifique de **CollectionName** , **DatabaseName** , **OperationType** , **Region** , **Status** et **StatusCode**. Les options **Ajouter un filtre** et **Appliquer le fractionnement** vous permettent de filtrer l’utilisation des unités de requête et de regrouper les métriques.

Pour obtenir l’utilisation des unités de requête de chaque opération par total (somme) ou par moyenne, sélectionnez **Appliquer le fractionnement** , puis choisissez **Type d’opération** et la valeur du filtre comme illustré dans l’image suivante :

   :::image type="content" source="./media/monitor-request-unit-usage/request-unit-usage-operations.png" alt-text="Unités de requête Cosmos DB pour des opérations dans Azure Monitor":::

Si vous voulez voir l’utilisation des unités de requête par collection, sélectionnez **Appliquer le fractionnement** et choisissez le nom de la collection comme filtre. Vous voyez alors un graphique comme celui-ci avec un choix de collections dans le tableau de bord. Vous pouvez ensuite sélectionner un nom de collection spécifique pour voir plus de détails :

   :::image type="content" source="./media/monitor-request-unit-usage/request-unit-usage-collection.png" alt-text="Unités de requête Cosmos DB pour toutes les opérations de la collection dans Azure Monitor":::

## <a name="next-steps"></a>Étapes suivantes

* Superviser les données d’Azure Cosmos DB en utilisant des [paramètres de diagnostic](cosmosdb-monitor-resource-logs.md) dans Azure.
* [Guide pratique pour auditer les opérations de plan de contrôle Azure Cosmos DB](audit-control-plane-logs.md)
