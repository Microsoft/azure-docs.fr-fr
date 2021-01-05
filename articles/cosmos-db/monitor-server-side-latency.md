---
title: Comment surveiller la latence côté serveur pour les opérations dans Azure Cosmos DB
description: Découvrez comment surveiller la latence du serveur pour les opérations dans un compte ou conteneur Azure Cosmos DB. Les propriétaires d’un compte Azure Cosmos DB peuvent comprendre les problèmes de latence côté serveur avec vos comptes Azure Cosmos.
ms.service: cosmos-db
ms.topic: how-to
author: kanshiG
ms.author: govindk
ms.date: 04/07/2020
ms.openlocfilehash: ec6a9db63504958640137fcd0fcfc904eb01afa5
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93074727"
---
# <a name="how-to-monitor-the-server-side-latency-for-operations-in-an-azure-cosmos-db-container-or-account"></a>Comment surveiller la latence côté serveur pour les opérations dans un conteneur ou compte Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Monitor pour Azure Cosmos DB fournit une vue de métriques pour surveiller votre compte et créer des tableaux de bord. Cette fonctionnalité ne vous oblige pas à activer ou à configurer quoi que ce soit explicitement, car ces métriques Azure Cosmos DB sont collectées par défaut. La métrique de latence côté serveur est utilisée pour afficher la latence côté serveur d’une opération. Azure Cosmos DB fournit un contrat de niveau de service de moins de 10 ms pour les opérations de lecture/écriture de point avec connectivité directe. Pour les opérations de lecture et d’écriture de point, les contrats de niveau de service sont calculés comme détaillé dans le [document SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/).

Vous pouvez surveiller la latence côté serveur si vous constatez une latence anormalement élevée pour l’opération point, par exemple :

* Une opération GET ou SET avec clé de partition et ID en mode de connectivité directe
* Une opération de lecture ou d'écriture, ou
* Une requête

Vous pouvez consulter le journal de diagnostic pour connaître la taille des données retournées. Si vous constatez une latence élevée soutenue pour les opérations de requête, vous devez parcourir le journal de diagnostic à la recherche d’un [débit (ou RU/seconde)](cosmosdb-monitor-resource-logs.md#diagnostic-queries) supérieur. La latence côté serveur indique le temps passé sur l’infrastructure principale avant que les données ne soient retournées au client. Il est important d’examiner cette mesure pour éliminer les éventuels problèmes de latence du serveur principal.

## <a name="view-the-server-side-latency-metric"></a>Voir les métriques de latence côté serveur

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Surveillance** dans la barre de navigation gauche, puis sélectionnez **Métriques**.

   :::image type="content" source="./media/monitor-server-side-latency/monitor-metrics-blade.png" alt-text="Volet Métriques dans Azure Monitor":::

1. À partir du volet **Métriques** > **Sélectionner une ressource** > choisissez l’ **abonnement** exigé, puis **Groupe de ressources**. Pour le **type de ressource** , sélectionnez **Comptes Azure Cosmos DB** , choisissez une de vos comptes Azure Cosmos existants, puis sélectionnez **Appliquer**.
   
   :::image type="content" source="./media/monitor-server-side-latency/select-cosmos-db-account.png" alt-text="Choisir le compte Azure Cosmos DB pour voir les métriques":::

1. Sélectionnez ensuite la métrique **Latence côté serveur** dans la liste des métriques disponibles. Pour découvrir de plus près toutes les métriques disponibles dans cette liste, consultez l’article [Métriques par catégorie](monitor-cosmos-db-reference.md). Dans cet exemple, nous allons sélectionner **Latence côté serveur** et **Moy** comme valeur d’agrégation. En plus de ces détails, vous pouvez également sélectionner l’ **intervalle de temps** et la **granularité temporelle** des métriques. Au maximum, vous pouvez voir les métriques des 30 derniers jours.  Une fois que vous avez appliqué le filtre, un graphique s’affiche. Vous pouvez voir la latence côté serveur par minute pour la période sélectionnée.  

   :::image type="content" source="./media/monitor-server-side-latency/server-side-latency-metric.png" alt-text="Choisir la métrique Latence côté serveur à partir du portail Azure":::

## <a name="filters-for-server-side-latency"></a>Filtres pour la latence côté serveur

Vous pouvez également filtrer les métriques et afficher les graphiques par valeur spécifique **CollectionName** , **ConnectionMode** , **DatabaseName** , **OperationType** , **Region** et **PublicAPIType**. 

Pour filtrer les métriques, sélectionnez **Ajouter un filtre** et choisissez la propriété nécessaire comme **PublicAPIType** , puis sélectionnez une valeur comme **sql**. Ajoutez un autre filtre pour **OperationType**. Le graphique affiche ensuite la latence côté serveur pour les différentes opérations au cours de la période sélectionnée. Les opérations exécutées par procédure stockée ne sont pas journalisées si bien qu’elles ne sont pas disponibles sous la métrique OperationType.

Les métriques **Latence côté serveur** pour chaque opération s’affichent comme indiqué dans l’image suivante :

:::image type="content" source="./media/monitor-server-side-latency/server-side-latency-filters.png" alt-text="Filtres pour les métriques de latence côté serveur":::

Vous pouvez également regrouper des métriques à l’aide de l’option **Appliquer la division**.  

## <a name="next-steps"></a>Étapes suivantes

* Superviser les données d’Azure Cosmos DB en utilisant des [paramètres de diagnostic](cosmosdb-monitor-resource-logs.md) dans Azure.
* [Guide pratique pour auditer les opérations de plan de contrôle Azure Cosmos DB](audit-control-plane-logs.md)
