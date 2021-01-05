---
title: Superviser et déboguer à l’aide de métriques dans Azure Cosmos DB
description: Utilisez les métriques d’Azure Cosmos DB pour déboguer les problèmes courants et surveiller la base de données.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 07/22/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 243f6f26be592e2db82d8f46df3de9aafcd2078b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996982"
---
# <a name="monitor-and-debug-with-metrics-in-azure-cosmos-db"></a>Superviser et déboguer à l’aide de métriques dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB fournit des métriques concernant le débit, le stockage, la cohérence, la disponibilité et la latence. Le portail Azure fournit une vue agrégée de ces métriques. Vous pouvez également consulter des métriques Azure Cosmos DB à partir de l’API Azure Monitor. Les valeurs de dimension des métriques, comme le nom du conteneur, ne respectent pas la casse. Vous devez donc utiliser une comparaison ne respectant pas la casse lorsque vous comparez les chaînes sur ces valeurs de dimension. Pour savoir comment consulter des métriques à partir d’Azure Monitor, consultez l’article [Obtenir des métriques d’Azure Monitor](./monitor-cosmos-db.md).

Cet article explique des cas d’utilisation courants et montre comment utiliser les métriques d’Azure Cosmos DB pour analyser et déboguer ces problèmes. Les métriques sont collectées toutes les cinq minutes et sont conservées pendant sept jours.

## <a name="view-metrics-from-azure-portal"></a>Voir les métriques depuis le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/)

1. Ouvrez le volet **Métriques**. Par défaut, le volet des métriques montre les métriques de stockage, d’index et d’unités de requête pour toutes les bases de données dans votre compte Azure Cosmos. Vous pouvez filtrer ces métriques par base de données, conteneur ou région. Vous pouvez également filtrer les métriques en fonction d’une précision temporelle spécifique. Vous trouverez plus de détails sur les métriques de débit, de stockage, de disponibilité, de latence et de cohérence sous les onglets correspondants. 

   :::image type="content" source="./media/use-metrics/performance-metrics.png" alt-text="Métriques de performances Cosmos DB dans le Portail Azure":::

Les métriques suivantes sont disponibles dans le volet **Métriques** : 

* **Métriques de débit** : cette métrique indique le nombre de demandes consommées ou ayant échoué (code de réponse 429) en raison du dépassement de la capacité de débit ou de stockage provisionnée pour le conteneur.

* **Métriques de stockage** - Cette métrique indique la taille utilisée par les données et les index.

* **Métriques de disponibilité** - Cette métrique montre le pourcentage de demandes réussies par rapport au nombre total de demandes par heure. Le taux de réussite est défini par les contrats SLA d’Azure Cosmos DB.

* **Métriques de latence** - Cette métrique montre la latence de lecture et d’écriture observée par Azure Cosmos DB dans la région où votre compte fonctionne. Vous pouvez visualiser la latence entre les régions pour un compte géorépliqué. Cette métrique ne représente pas la latence des demandes de bout en bout.

* **Métriques de cohérence** - Cette métrique montre le degré de cohérence final du modèle que vous choisissez. Pour les comptes multirégions, cette métrique indique également la latence de réplication entre les régions que vous avez sélectionnées.

* **Métriques système** - Cette métrique montre le nombre de demandes de métadonnées prises en charge par la partition principale. Elle permet également d’identifier les demandes limitées.

Les sections suivantes décrivent des scénarios courants où vous pouvez utiliser les métriques Azure Cosmos DB. 

## <a name="understand-how-many-requests-are-succeeding-or-causing-errors"></a>Connaître le nombre de requêtes ayant abouti ou ayant provoqué une erreur

Pour commencer, accédez au [portail Azure](https://portal.azure.com), puis accédez au panneau **Métriques**. Dans le panneau, recherchez le graphique **Nombre de requêtes ayant dépassé la capacité par tranche de 1 minute. Ce graphique montre, minute par minute, le nombre total de requêtes, segmentées par code d’état. Pour plus d’informations sur les codes d’état HTTP, consultez [Codes d’état HTTP pour Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

Le code d’état d’erreur le plus courant est 429 (limitation du débit). Cette erreur signifie que les requêtes envoyées à Azure Cosmos DB sont supérieures au débit provisionné. Dans ce cas, la solution la plus courante consiste à [effectuer une montée en puissance des unités de requête](./set-throughput.md) pour une collection donnée.

:::image type="content" source="media/use-metrics/metrics-12.png" alt-text="Nombre de requêtes par minute":::

## <a name="determine-the-throughput-distribution-across-partitions"></a>Déterminer la distribution du débit entre les partitions

Il est essentiel d’avoir une bonne cardinalité des clés de partition pour vos applications évolutives. Pour déterminer la distribution du débit au sein d’un conteneur partitionné, accédez au **panneau Métriques** dans le [portail Azure](https://portal.azure.com). Sous l’onglet **Débit**, la répartition du débit est affichée dans le graphique **Nombre maximal de RU/seconde consommées par chaque partition physique**. Le graphique suivant montre un exemple de mauvaise distribution des données mise en évidence par l’asymétrie de la partition située à l’extrême gauche.

:::image type="content" source="media/use-metrics/metrics-17.png" alt-text="Partition unique fortement utilisée":::

Une distribution inégale du débit peut aboutir à une *forte utilisation* de certaines partitions. Dans ce cas, une limitation des requêtes peut se produire et nécessiter un repartitionnement. Pour plus d’informations sur le partitionnement dans Azure Cosmos DB, consultez [Partitionner et mettre à l’échelle dans Azure Cosmos DB](./partitioning-overview.md).

## <a name="determine-the-storage-distribution-across-partitions"></a>Déterminer la distribution du stockage entre les partitions

Il est essentiel d’avoir une bonne cardinalité de partition pour vos applications évolutives. Pour déterminer la distribution du stockage au sein d’un conteneur partitionné, accédez au panneau Métriques dans le [portail Azure](https://portal.azure.com). Dans l’onglet Stockage, la répartition du stockage est indiquée dans le graphe Stockage de données + d’index consommé par les premières clés de partition. Le graphe suivant montre une mauvaise distribution du stockage des données, mise en évidence par l’asymétrie de la partition située à l’extrême gauche.

:::image type="content" source="media/use-metrics/metrics-07.png" alt-text="Exemple de mauvaise distribution des données":::

Vous pouvez connaître la clé de partition à l’origine du déséquilibre de la distribution en cliquant sur la partition du graphique.

:::image type="content" source="media/use-metrics/metrics-05.png" alt-text="Clé de partition provoquant un déséquilibre de la distribution":::

Une fois que vous avez identifié la clé de partition qui est à l’origine du déséquilibre, il est possible que vous deviez repartitionner votre conteneur avec une clé de partition mieux distribuée. Pour plus d’informations sur le partitionnement dans Azure Cosmos DB, consultez [Partitionner et mettre à l’échelle dans Azure Cosmos DB](./partitioning-overview.md).

## <a name="compare-data-size-against-index-size"></a>Comparer la taille des données et la taille de l’index

Dans Azure Cosmos DB, la consommation totale du stockage correspond à la somme de la taille des données et de la taille de l’index. En règle générale, la taille de l’index correspond à une fraction de la taille des données. Dans le panneau Métriques du [portail Azure](https://portal.azure.com), l’onglet Stockage présente la répartition de la consommation du stockage entre les données et l’index.

```csharp
// Measure the document size usage (which includes the index size)  
ResourceResponse<DocumentCollection> collectionInfo = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));
 Console.WriteLine("Document size quota: {0}, usage: {1}", collectionInfo.DocumentQuota, collectionInfo.DocumentUsage);
```

Si vous souhaitez conserver de l’espace d’index, vous pouvez ajuster la [stratégie d’indexation](index-policy.md).

## <a name="debug-why-queries-are-running-slow"></a>Résoudre les problèmes liés à l’exécution lente des requêtes

Dans les SDK de l’API SQL, Azure Cosmos DB fournit des statistiques relatives à l’exécution des requêtes.

```csharp
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
 UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName),
 "SELECT * FROM c WHERE c.city = 'Seattle'",
 new FeedOptions
 {
 PopulateQueryMetrics = true,
 MaxItemCount = -1,
 MaxDegreeOfParallelism = -1,
 EnableCrossPartitionQuery = true
 }).AsDocumentQuery();
FeedResponse<dynamic> result = await query.ExecuteNextAsync();

// Returns metrics by partition key range Id
IReadOnlyDictionary<string, QueryMetrics> metrics = result.QueryMetrics;
```

*QueryMetrics* fournit des informations détaillées sur la durée d’exécution de chaque composant de la requête. Les analyses constituent la cause racine la plus courante de la longueur d’exécution des requêtes, ce qui signifie que la requête n’a pas pu exploiter les index. Ce problème peut être résolu avec une meilleure condition de filtre.

## <a name="next-steps"></a>Étapes suivantes

Vous venez de découvrir comment superviser et déboguer les problèmes à l’aide des métriques fournies dans le portail Azure. Vous souhaiterez peut-être en savoir plus sur l’amélioration des performances de la base de données en lisant les articles suivants :

* Pour savoir comment consulter des métriques à partir d’Azure Monitor, consultez l’article [Obtenir des métriques d’Azure Monitor](./monitor-cosmos-db.md). 
* [Test des performances et de la mise à l’échelle avec Azure Cosmos DB](performance-testing.md)
* [Conseils sur les performances pour Azure Cosmos DB](performance-tips.md)