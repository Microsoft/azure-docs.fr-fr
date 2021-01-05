---
title: Surveiller les unités de requête normalisée par seconde pour un conteneur ou un compte Azure Cosmos
description: Découvrez comment surveiller l’utilisation des unités de requête normalisée d’une opération dans Azure Cosmos DB. Les propriétaires d’un compte Azure Cosmos DB peuvent déterminer les opérations qui consomment le plus d’unités de requête.
ms.service: cosmos-db
ms.topic: how-to
author: kanshiG
ms.author: govindk
ms.date: 06/25/2020
ms.openlocfilehash: dc47f2f7a0f1586b197d14015fe2167293c806c6
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93099332"
---
# <a name="how-to-monitor-normalized-rus-for-an-azure-cosmos-container-or-an-account"></a>Comment surveiller des unités de requête normalisée par seconde pour un conteneur ou un compte Azure Cosmos
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Monitor pour Azure Cosmos DB fournit une vue de métriques pour surveiller votre compte et créer des tableaux de bord. Cette fonctionnalité ne vous oblige pas à activer ou à configurer quoi que ce soit explicitement, car ces métriques Azure Cosmos DB sont collectées par défaut.

La métrique **Consommation d’unités de requête normalisée** permet de déterminer dans quelle mesure les plages de clés de partition sont bien saturées par rapport au trafic. Azure Cosmos DB distribue le débit de manière égale entre toutes les plages de clés de partition. Cette mesure fournit une vue par seconde de l’utilisation maximale du débit pour la plage de clés de partition. Utilisez cette métrique pour calculer l’utilisation d’unités de requête dans une plage de clés de partition pour un conteneur donné. En utilisant cette métrique, si vous constatez un pourcentage élevé d’utilisation d’unités de requête entre toutes les plages de clés de partition, vous devez augmenter le débit pour répondre aux besoins de votre charge de travail. Exemple : l’utilisation normalisée est définie comme la quantité maximale d’utilisation de RU/s parmi toutes les plages de clés de partitions. Par exemple, supposez que votre débit maximal est de 20 000 RU/s et que vous disposez de deux plages de clés de partitions, P_1 et P_2, chacune autorisant une mise à l’échelle à 10 000 RU/s. Durant une seconde donnée, si P_1 a utilisé 6 000 RU/s et P_2 8 000 RU/s, l’utilisation normalisée est MAX(6 000 RU / 10 000 RU, 8 000 RU / 10 000 RU) = 0,8.

## <a name="what-to-expect-and-do-when-normalized-rus-is-higher"></a>Ce qu’il faut attendre et faire quand les unités de requête normalisée par seconde sont plus élevés

Lorsque la consommation normalisée de RU/s atteint 100 % pour une plage de clés de partition donnée et qu’un client fait encore des demandes dans cette fenêtre de temps de 1 seconde pour cette plage de clés de partition spécifique, celui-ci reçoit une erreur de débit limité. Le client doit respecter le temps d’attente suggéré et retenter la demande. Le kit de développement logiciel (SDK) facilite la gestion de cette situation en renouvelant les temps préconfigurés en attendant de manière appropriée.  Il n’est pas nécessaire d’afficher l’erreur de limitation du débit, car le RU normalisé a atteint 100 %. En effet, le RU normalisé est une valeur unique qui représente l’utilisation maximale de toutes les plages de clés de partition ; une plage de clés de partition peut être occupée, mais les autres plages de clés de partition peuvent traiter les demandes sans problème. Par exemple, une opération unique, telle qu’une procédure stockée qui consomme l’ensemble des RU/s sur une plage de clés de partition, entraîne un pic dans la consommation de RU/s normalisée. Dans ce cas, il n’y aura pas d’erreurs de limitation de débit immédiates si le taux de demandes est faible ou si des demandes sont effectuées sur d’autres partitions sur des plages de clés de partition différentes. 

Les métriques Azure Monitor vous permettent de rechercher des opérations selon leur code d’état de l’API SQL via la métrique **Total Requests** (Nombre total de requêtes). Plus tard, vous pouvez filtrer ces requêtes par selon le code d’état 429 et les diviser selon le critère **Operation Type** (Type d’opération).  

Pour rechercher les requêtes à débit limité, la méthode recommandée consiste à obtenir ces informations par le biais des journaux de diagnostic.

S’il existe un pic continu de 100 % de consommation de RU/s normalisé ou proche de 100 % parmi plusieurs plages de clés de partition, il est recommandé d’augmenter le débit. Vous pouvez déterminer quelles opérations sont lourdes, ainsi que leurs pics d’utilisation en utilisant les métriques et les journaux de diagnostic d’analyse d’Azure.

En résumé, la mesure **Consommation d’unités de requête normalisée** permet de déterminer la plage de clés de partition qui est plus chaude en termes d’utilisation. Ainsi, vous obtenez l’inclinaison du débit vers une plage de clés de partition. Vous pouvez ensuite suivre le journal **PartitionKeyRUConsumption** dans Azure Monitor pour obtenir des informations sur les clés de partition logique qui sont très utilisées. Cela pointera vers la modification dans le choix de la clé de partition ou la modification de la logique d’application. Pour résoudre la limitation du débit, distribuez la charge de données entre plusieurs partitions ou augmentez simplement le débit, car cela est véritablement nécessaire. 



## <a name="view-the-normalized-request-unit-consumption-metric"></a>Afficher la métrique de consommation d’unités de requête normalisée

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez **Surveillance** dans la barre de navigation gauche, puis sélectionnez **Métriques**.

   :::image type="content" source="./media/monitor-normalized-request-units/monitor-metrics-blade.png" alt-text="Volet Métriques dans Azure Monitor":::

3. À partir du volet **Métriques** > **Sélectionner une ressource** > choisissez l’ **abonnement** exigé, puis **Groupe de ressources**. Pour le **type de ressource** , sélectionnez **Comptes Azure Cosmos DB** , choisissez une de vos comptes Azure Cosmos existants, puis sélectionnez **Appliquer**.

   :::image type="content" source="./media/monitor-normalized-request-units/select-cosmos-db-account.png" alt-text="Choisir un compte Azure Cosmos pour afficher les métriques":::

4. Ensuite, vous pouvez sélectionner une métrique dans la liste des métriques disponibles. Vous pouvez sélectionner des métriques propres aux unités de requête, au stockage, à la latence, à la disponibilité, à Cassandra, etc. Pour découvrir de plus près toutes les métriques disponibles dans cette liste, consultez l’article [Métriques par catégorie](monitor-cosmos-db-reference.md). Dans cet exemple, nous allons sélectionner la métrique **Normalized RU Consumption** (Consommation d’unités de requête normalisée) et la valeur d’agrégation **Max**.

   En plus de ces détails, vous pouvez également sélectionner l’ **intervalle de temps** et la **granularité temporelle** des métriques. Au maximum, vous pouvez voir les métriques des 30 derniers jours.  Une fois que vous avez appliqué le filtre, un graphique s’affiche.

   :::image type="content" source="./media/monitor-normalized-request-units/normalized-request-unit-usage-metric.png" alt-text="Choisir une métrique à partir du portail Azure":::

### <a name="filters-for-normalized-request-unit-consumption"></a>Filtres pour la consommation d’unités de requête normalisée

Vous pouvez également filtrer les métriques et le graphique affiché par une valeur **CollectionName** , **DatabaseName** , **PartitionKeyRangeID** , et **Region** spécifique. Pour filtrer les métriques, sélectionnez **Add filter** (Ajouter un filtre), puis choisissez la propriété requise, par exemple **CollectionName** , ainsi que la valeur correspondante qui vous intéresse. Le graphique affiche ensuite les unités de consommation d’unités de requête normalisée consommées pour le conteneur pour la période sélectionnée.  

Vous pouvez regrouper des métriques à l’aide de l’option **Appliquer la division**.  

La métrique de consommation de l’unité de requête normalisée pour chaque conteneur s’affiche comme indiqué dans l’image suivante :

:::image type="content" source="./media/monitor-normalized-request-units/normalized-request-unit-usage-filters.png" alt-text="Appliquer des filtres à la métrique de consommation d’unités de requête normalisée":::

## <a name="next-steps"></a>Étapes suivantes

* Superviser les données d’Azure Cosmos DB en utilisant des [paramètres de diagnostic](cosmosdb-monitor-resource-logs.md) dans Azure.
* [Guide pratique pour auditer les opérations de plan de contrôle Azure Cosmos DB](audit-control-plane-logs.md)
