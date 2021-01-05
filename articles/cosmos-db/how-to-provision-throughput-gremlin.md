---
title: Approvisionner le débit sur des ressources de l’API Gremlin d’Azure Cosmos DB
description: Découvrez comment approvisionner le débit des conteneurs, des bases de données et de la mise à l’échelle automatique sur des ressources de l’API Gremlin d’Azure Cosmos DB. Vous utiliserez le portail Azure, CLI, PowerShell et d’autres kits de développement logiciel (SDK).
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 3c2af7f33135a8c6621db233451231ffa89c2d64
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93086157"
---
# <a name="provision-database-container-or-autoscale-throughput-on-azure-cosmos-db-gremlin-api-resources"></a>Approvisionner le débit des conteneurs, des bases de données et la mise à l’échelle automatique sur des ressources de l’API Gremlin d’Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article explique comment provisionner le débit dans l’API Gremlin d’Azure Cosmos DB. Vous pouvez approvisionner un débit standard (manuel) ou avec mise à l’échelle automatique sur un conteneur ou sur une base de données et le partager entre les conteneurs de la base de données. Vous pouvez provisionner le débit à l’aide du portail Azure, d’Azure CLI ou des SDK Azure Cosmos DB.

Si vous utilisez une autre API, consultez les articles [API SQL](how-to-provision-container-throughput.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [API pour MongoDB](how-to-provision-throughput-mongodb.md) pour approvisionner le débit.

## <a name="azure-portal"></a><a id="portal-gremlin"></a> Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) ou sélectionnez un compte Azure Cosmos existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouveau graphe**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez une base de données ou si vous utilisez une base de données existante. Sélectionnez l’option **Approvisionner le débit d’une base de données** si vous souhaitez approvisionner le débit au niveau de la base de données.
   * Entrez un ID de graphe.
   * Entrez une valeur de clé de partition (par exemple `/ItemID`).
   * Entrez un débit que vous voulez provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-provision-throughput-gremlin/provision-database-throughput-portal-gremlin-api.png" alt-text="Capture d’écran de l’Explorateur de données, lors de la création d’un nouveau graphique avec un débit au niveau de base de données":::

## <a name="net-sdk"></a>Kit de développement logiciel (SDK) .NET

> [!Note]
> Utilisez les SDK Cosmos pour l’API SQL afin de provisionner le débit de toutes les API Azure Cosmos DB, à l’exception des API Cassandra et MongoDB.

### <a name="provision-container-level-throughput"></a>Approvisionner le débit au niveau d’un conteneur

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

### <a name="provision-database-level-throughput"></a>Approvisionner le débit au niveau d’une base de données

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

```csharp
//set the throughput for the database
RequestOptions options = new RequestOptions
{
    OfferThroughput = 500
};

//create the database
await client.CreateDatabaseIfNotExistsAsync(
    new Database {Id = databaseName},  
    options);
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/DatabaseDocsSampleCode.cs?name=DatabaseCreateWithThroughput)]

---

## <a name="azure-resource-manager"></a>Azure Resource Manager

Les modèles Azure Resource Manager peuvent être utilisés pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Consultez [Modèles Azure Resource Manager pour Azure Cosmos DB](templates-samples-gremlin.md) afin de voir des exemples.

## <a name="azure-cli"></a>Azure CLI

L’interface Azure CLI peut être utilisée pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Pour voir des exemples, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples-gremlin.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell peut être utilisé pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Pour obtenir des exemples, consultez [Exemples Azure PowerShell pour Azure Cosmos DB](powershell-samples-gremlin.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur le provisionnement du débit dans Azure Cosmos DB :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)