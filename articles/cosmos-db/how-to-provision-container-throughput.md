---
title: Approvisionner le débit d’un conteneur dans l’API SQL Azure Cosmos DB
description: Apprenez à provisionner le débit au niveau du conteneur dans l’API SQL Azure Cosmos DB avec le portail Azure, l’interface CLI, PowerShell et différents kits SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/14/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 4caf43cb972b44dd1482b9e6e467e41cae294708
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100097"
---
# <a name="provision-standard-manual-throughput-on-an-azure-cosmos-container---sql-api"></a>Approvisionner le débit standard (manuel) sur un conteneur Azure Cosmos - API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment approvisionner le débit standard (manuel) sur un conteneur dans l’API SQL Azure Cosmos DB. Vous pouvez provisionner le débit sur un seul conteneur, ou [provisionner le débit sur une base de données](how-to-provision-database-throughput.md) et le partager entre les conteneurs de la base de données. Vous pouvez provisionner le débit sur un conteneur à l’aide du portail Azure, d’Azure CLI ou des SDK Azure Cosmos DB.

Si vous utilisez une autre API, consultez les articles [API pour MongoDB](how-to-provision-throughput-mongodb.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [API Gremlin](how-to-provision-throughput-gremlin.md) pour approvisionner le débit.

## <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-sql-api-dotnet.md#create-account) ou sélectionnez un compte Azure Cosmos existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouveau conteneur**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez une base de données ou si vous utilisez une base de données existante.
   * Entrez un ID de conteneur.
   * Entrez une valeur de clé de partition (par exemple `/ItemID`).
   * Entrez un débit que vous voulez provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-provision-container-throughput/provision-container-throughput-portal-sql-api.png" alt-text="Capture d’écran de Data Explorer, avec l’option Nouvelle collection mise en évidence":::

## <a name="azure-cli-or-powershell"></a>Azure CLI ou PowerShell

Pour créer un conteneur avec un débit dédié, voir

* [Créer un conteneur à l’aide d’Azure CLI](manage-with-cli.md#create-a-container)
* [Créer un conteneur à l’aide de PowerShell](manage-with-powershell.md#create-container)

## <a name="net-sdk"></a>Kit de développement logiciel (SDK) .NET

> [!Note]
> Utilisez les SDK Cosmos pour l’API SQL afin de provisionner le débit de toutes les API Cosmos DB, à l’exception des API Cassandra et MongoDB.

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

```csharp
// Create a container with a partition key and provision throughput of 400 RU/s
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "myContainerName";
myCollection.PartitionKey.Paths.Add("/myPartitionKey");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    myCollection,
    new RequestOptions { OfferThroughput = 400 });
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/ContainerDocsSampleCode.cs?name=ContainerCreateWithThroughput)]

---

## <a name="javascript-sdk"></a>Kit de développement logiciel (SDK) JavaScript

```javascript
// Create a new Client
const client = new CosmosClient({ endpoint, key });

// Create a database
const { database } = await client.databases.createIfNotExists({ id: "databaseId" });

// Create a container with the specified throughput
const { resource } = await database.containers.createIfNotExists({
id: "containerId",
throughput: 1000
});

// To update an existing container or databases throughput, you need to user the offers API
// Get all the offers
const { resources: offers } = await client.offers.readAll().fetchAll();

// Find the offer associated with your container or the database
const offer = offers.find((_offer) => _offer.offerResourceId === resource._rid);

// Change the throughput value
offer.content.offerThroughput = 2000;

// Replace the offer.
await client.offer(offer.id).replace(offer);
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur le provisionnement du débit dans Azure Cosmos DB :

* [Comment approvisionner le débit standard (manuel) sur une base de données](how-to-provision-database-throughput.md)
* [Comment approvisionner le débit avec mise à l’échelle automatique sur une base de données](how-to-provision-autoscale-throughput.md)
* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)
