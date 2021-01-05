---
title: Optimiser les coûts de stockage dans Azure Cosmos DB
description: Cet article explique comment gérer les coûts de stockage pour les données stockées dans Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/21/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 92bd3ff925080def4b2f074d07e662dfdbdbee01
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080848"
---
# <a name="optimize-storage-cost-in-azure-cosmos-db"></a>Optimiser les coûts de stockage dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB offre un stockage et un débit illimités. Contrairement au débit, que vous devez approvisionner/configurer sur vos bases de données ou conteneurs Azure Cosmos, le stockage est facturé en fonction de la consommation. Vous êtes facturé uniquement pour le stockage logique que vous consommez et vous n’êtes pas obligé de réserver de stockage à l’avance. Le stockage se met automatiquement à l’échelle en fonction des données que vous ajoutez ou supprimez dans un conteneur Azure Cosmos.

## <a name="storage-cost"></a>Coût de stockage

Le stockage est facturé en Go. Le stockage SSD local est utilisé par vos données et pour l’indexation. Le stockage total utilisé est égal au stockage requis par les données et les index utilisés dans toutes les régions où vous utilisez Azure Cosmos DB. Si vous répliquez un compte Azure Cosmos dans trois régions, vous paierez le coût de stockage total dans chacune de ces trois régions. Pour estimer vos besoins de stockage, accédez à l’outil [Capacity Planner](https://www.documentdb.com/capacityplanner). Le coût du stockage dans Azure Cosmos DB est de 0,25 $ Go/mois, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/) pour les dernières mises à jour. Vous pouvez définir des alertes pour déterminer le stockage utilisé par votre conteneur Azure Cosmos. Pour savoir comment surveiller votre stockage, consultez l’article [Surveiller Azure Cosmos DB](./monitor-cosmos-db.md).

## <a name="optimize-cost-with-item-size"></a>Optimiser les coûts avec la taille de l’élément

Dans Azure Cosmos DB, l’élément doit avoir une taille de 2 Mo tout au plus pour des performances optimales et des économies. Si vous avez besoin qu’un élément stocke plus de 2 Mo de données, envisagez de revoir la conception du schéma de l’élément. Dans les rares cas où vous ne pouvez pas redéfinir le schéma, vous pouvez fractionner l’élément en sous-éléments et les lier logiquement avec un identificateur (ID) commun. Toutes les fonctionnalités d’Azure Cosmos DB fonctionnent de façon cohérente via ancrage à cet identificateur logique.

## <a name="optimize-cost-with-indexing"></a>Optimiser les coûts avec l’indexation

Par défaut, les données sont automatiquement indexées, ce qui peut augmenter le stockage total consommé. Toutefois, vous pouvez appliquer des stratégies d’index personnalisées pour réduire cette surcharge. L’indexation automatique qui n’a pas été réglée via la stratégie représente environ 10 à 20 % de la taille de l’élément. En supprimant ou en personnalisant les stratégies d’index, vous ne payez pas de frais supplémentaires pour les écritures et aucune capacité de débit supplémentaire n’est requise. Consultez [Indexation dans Azure Cosmos DB](index-policy.md) pour configurer des stratégies d’indexation personnalisées. Si vous avez travaillé avec des bases de données relationnelles avant, vous pourriez penser que « tout indexer » équivaut à doubler la capacité de stockage voire plus. Toutefois, dans Azure Cosmos DB, dans le cas médian, cela représente une augmentation beaucoup plus faible. Dans Azure Cosmos DB, la surcharge de stockage que représente l’index est généralement faible (10-20 %) même avec l’indexation automatique, car il est conçu pour un encombrement de stockage faible. En gérant la stratégie d’indexation, vous pouvez contrôler le compromis entre les performances de requête et l’encombrement des index de manière plus précise.

## <a name="optimize-cost-with-time-to-live-and-change-feed"></a>Optimiser les coûts avec la durée de vie et le flux de modification

Une fois que vous n’avez plus besoin des données, vous pouvez les supprimer de votre compte Azure Cosmos avec la [durée de vie](time-to-live.md), le [flux de modification](change-feed.md) ou vous pouvez migrer les anciennes données vers un autre magasin de données comme le Stockage Blob Azure ou Azure Data Warehouse. Avec la « durée de vie » (TTL, Time to Live), Azure Cosmos DB permet de supprimer automatiquement des éléments d’un conteneur après une période déterminée. La durée de vie par défaut peut être définie au niveau du conteneur et être substituée par élément. Une fois la durée de vie définie au niveau d’un conteneur ou d’un élément, Azure Cosmos DB supprime automatiquement les éléments correspondants au terme de la période écoulée depuis la dernière modification. Avec le flux de modification, vous pouvez migrer des données vers un autre conteneur dans Azure Cosmos DB ou vers un magasin de données externe. La migration ne provoque aucun temps d’arrêt et lorsqu’elle est terminée, vous pouvez supprimer les données ou configurer la durée de vie pour qu’elle supprime le conteneur Azure Cosmos source.

## <a name="optimize-cost-with-rich-media-data-types"></a>Optimiser les coûts avec les types de données multimédias enrichies 

Si vous souhaitez stocker des types de contenus multimédias enrichis, par exemple, des vidéos, des images, etc., plusieurs options sont disponibles dans Azure Cosmos DB. Une option consiste à stocker ces types de contenus multimédias enrichis en tant qu’éléments Azure Cosmos. Il existe une limite de 2 Mo par élément que vous pouvez contourner avec le chaînage de l’élément de données dans plusieurs sous-éléments. Sinon, vous pouvez les stocker dans le Stockage Blob Azure et utiliser les métadonnées pour les référencer dans vos éléments Azure Cosmos. Cette approche représente des avantages et des inconvénients. La première approche vous permet d’obtenir les meilleures performances en termes de latence, de SLA de débit ainsi que des fonctionnalités de distribution mondiale clé en main pour les types de données multimédias enrichis en plus de vos éléments Azure Cosmos habituels. Mais le support associé à cette approche coûte plus cher. En stockant le contenu multimédia dans le Stockage Blob Azure, vous pourrez réduire vos coûts globaux. Si la latence est un facteur essentiel, vous pouvez utiliser le stockage Premium pour les fichiers multimédias enrichis qui sont référencés dans les éléments Azure Cosmos. Cette fonction s’intègre en mode natif avec CDN pour fournir des images à partir du serveur de périphérie à moindre coût pour contourner la restriction de la zone géographique. L’inconvénient avec ce scénario, c’est que vous devez gérer deux services : Azure Cosmos DB et le Stockage Blob Azure, ce qui peut augmenter les coûts d’exploitation. 

## <a name="check-storage-consumed"></a>Vérifier le stockage utilisé

Pour vérifier la consommation du stockage d’un conteneur Azure Cosmos, vous pouvez exécuter une requête HEAD ou GET sur le conteneur et inspecter les en-têtes `x-ms-request-quota` et `x-ms-request-usage`. Sinon, lorsque vous utilisez le SDK .NET, vous pouvez utiliser les propriétés [DocumentSizeQuota](/previous-versions/azure/dn850325(v%3Dazure.100)) et [DocumentSizeUsage](/previous-versions/azure/dn850324(v=azure.100)) propriétés pour connaître la quantité de stockage utilisée.

## <a name="using-sdk"></a>Utiliser le kit de développement logiciel (SDK)

```csharp
// Measure the item size usage (which includes the index size)
ResourceResponse<DocumentCollection> collectionInfo = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));   

Console.WriteLine("Item size quota: {0}, usage: {1}", collectionInfo.DocumentQuota, collectionInfo.DocumentUsage);
```

## <a name="next-steps"></a>Étapes suivantes

Pour continuer à développer vos connaissances sur l’optimisation des coûts dans Azure Cosmos DB, consultez les articles suivants :

* En savoir plus sur l’[optimisation pour le développement et le test](optimize-dev-test.md)
* En savoir plus sur les [factures Azure Cosmos DB](understand-your-bill.md)
* En savoir plus sur l’[optimisation du coût du débit](optimize-cost-throughput.md)
* En savoir plus sur l’[optimisation du coût des lectures et écritures](optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des requêtes](./optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des comptes Azure Cosmos sur plusieurs régions](optimize-cost-regions.md)
