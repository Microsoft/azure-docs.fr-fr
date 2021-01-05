---
title: Créer un conteneur dans Azure Cosmos DB API Gremlin
description: Découvrez comment créer un conteneur dans Azure Cosmos DB API Gremlin à l’aide de Portail Azure, .NET et d’autres kits de développement logiciel (SDK).
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: f7e9de1f23ec46af08fe96b5db3170fac9a7eb2e
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101627"
---
# <a name="create-a-container-in-azure-cosmos-db-gremlin-api"></a>Créer un conteneur dans Azure Cosmos DB API Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article explique les différentes façons de créer un conteneur dans Azure Cosmos DB API Gremlin. Il montre comment créer un conteneur à l’aide de Portail Azure, Azure CLI, PowerShell ou des kits de développement logiciel (SDK) pris en charge. Cet article montre comment créer un conteneur, spécifier la clé de partition et provisionner le débit.

Cet article explique les différentes façons de créer un conteneur dans Azure Cosmos DB API Gremlin. Si vous utilisez une autre API, consultez les articles [API pour MongoDB](how-to-create-container-mongodb.md), [API Cassandra](how-to-create-container-cassandra.md), [API de table](how-to-create-container-table.md)et [API SQL](how-to-create-container.md)pour créer le conteneur.

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

## <a name="create-using-azure-portal"></a><a id="portal-gremlin"></a>Créer en utilisant le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-graph-dotnet.md#create-a-database-account) ou sélectionnez un compte existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouveau graphe**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez une base de données, ou si vous utilisez une base de données existante.
   * Entrez un ID de graphe.
   * Sélectionnez la capacité de stockage **Illimitée**.
   * Entrez une clé de partition pour les vertex.
   * Entrez un débit à provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-gremlin.png" alt-text="Capture d’écran de l’API Gremlin, boîte de dialogue Ajouter un graphique":::

## <a name="create-using-net-sdk"></a><a id="dotnet-sql-graph"></a>Créer à l’aide du SDK .NET

Si vous rencontrez une exception de délai d’expiration lors de la création d’une collection, effectuez une opération de lecture pour valider si la collection a été créée avec succès. L’opération de lecture lève une exception jusqu’à ce que l’opération de création de la collection aboutisse. Pour obtenir la liste des codes d’état pris en charge par l’opération de création, consultez l’article [Codes d’état HTTP pour Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

```csharp
// Create a container with a partition key and provision 1000 RU/s throughput.
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "myContainerName";
myCollection.PartitionKey.Paths.Add("/myPartitionKey");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    myCollection,
    new RequestOptions { OfferThroughput = 1000 });
```

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Créer à l’aide d’Azure CLI

[Créer un graphique Gremlin à l'aide d'Azure CLI](./scripts/cli/gremlin/create.md). Pour obtenir la liste de tous les exemples propres à l’interface Azure CLI dans toutes les API Azure Cosmos DB, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Créer à l’aide de PowerShell

[Créer un graphe Gremlin à l’aide de PowerShell](./scripts/powershell/gremlin/create.md). Pour obtenir la liste de tous les exemples PowerShell pour toutes les API Azure Cosmos DB, consultez [Exemples PowerShell](powershell-samples.md)

## <a name="next-steps"></a>Étapes suivantes

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Unités de requête dans Azure Cosmos DB](request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Utiliser un compte Azure Cosmos](./account-databases-containers-items.md)