---
title: Rechercher les frais d’unités de requête (RU) pour une les requêtes d’une API Table dans Azure Cosmos DB
description: Découvrez comment trouver les frais d’unités de requête pour les requêtes de l’API Table exécutées sur un conteneur Azure Cosmos. Vous pouvez utiliser le portail Azure et les langages .NET, Java, Python et Node.js pour rechercher les frais de RU.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 1d7a12e436fd3bc1700dc4a1d76dc2b80d861144
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93078456"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-table-api"></a>Rechercher les frais des unités de requête pour les opérations exécutées dans l’API Table d’Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Azure Cosmos DB prend en charge de nombreuses API, notamment SQL, MongoDB, Cassandra, Gremlin et Table. Chaque API possède son propre ensemble d’opérations de base de données, qui vont de simples opérations ponctuelles de lecture et d'écriture à des requêtes complexes. Chaque opération de base de données consomme des ressources système en fonction de sa complexité.

Le coût de toutes les opérations de base de données, normalisé par Azure Cosmos DB, est exprimé en unités de requête (RU). Les RU correspondent en quelque sorte à une devise de performances, faisant abstraction des ressources système (UC, IOPS, mémoire, etc.) requises pour effectuer les opérations de base de données prises en charge par Azure Cosmos DB. Quelle que soit l’API utilisée pour interagir avec le conteneur Azure Cosmos, les coûts sont toujours mesurés en unités de requête, Que l’opération de base de données soit une opération d’écriture, de lecture de point ou de requête, les coûts sont toujours mesurés en unités de requête. Pour plus d’informations, consultez l’article [Unités de requête et considérations](request-units.md).

Cet article présente les différentes façons de rechercher la consommation d’[unités de requête](request-units.md) pour toute opération exécutée sur un conteneur dans l’API Table Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles [API pour MongoDB](find-request-unit-charge-mongodb.md), [API Cassandra](find-request-unit-charge-cassandra.md), [API Gremlin](find-request-unit-charge-gremlin.md) et [API SQL](find-request-unit-charge.md) pour trouver les frais de RU/s.

## <a name="use-the-net-sdk"></a>Utiliser le kit de développement logiciel (SDK) .NET

Actuellement, le seul Kit de développement logiciel (SDK) qui retourne les frais d’unités de requête pour les opérations de table est le [Kit de développement .NET Standard](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table). L’objet `TableResult` expose une propriété `RequestCharge` qui est renseignée par le SDK quand vous l’utilisez sur l’API Table Azure Cosmos DB :

```csharp
CloudTable tableReference = client.GetTableReference("table");
TableResult tableResult = tableReference.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
if (tableResult.RequestCharge.HasValue) // would be false when using Azure Storage Tables
{
    double requestCharge = tableResult.RequestCharge.Value;
}
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application d’API Table avec le Kit de développement logiciel (SDK) .NET et Azure Cosmos DB](create-table-dotnet.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’optimisation de votre consommation d’unités de requête, consultez les articles suivants :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)
* [Optimiser le coût du débit approvisionné dans Azure Cosmos DB](optimize-cost-throughput.md)
* [Optimiser le coût de requête dans Azure Cosmos DB](./optimize-cost-reads-writes.md)