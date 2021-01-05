---
title: Créer un conteneur dans l’API SQL Azure Cosmos DB
description: Découvrez comment créer un conteneur dans l’API SQL Azure Cosmos DB avec le portail Azure et les kits SDK .Net, Java, Python, Node.js ou autres.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 302c5d6e8e523a11b8773f10bb6089e3bea09bdd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006846"
---
# <a name="create-a-container-in-azure-cosmos-db-sql-api"></a>Créer un conteneur dans l’API SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique les différentes façons de créer un conteneur dans l’API SQL Azure Cosmos DB. Il montre comment créer un conteneur à l’aide du portail Azure, d’Azure CLI, de PowerShell ou des kits de développement logiciel pris en charge. Cet article montre comment créer un conteneur, spécifier la clé de partition et provisionner le débit.

Cet article explique les différentes façons de créer un conteneur dans l’API SQL Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles [API pour MongoDB](how-to-create-container-mongodb.md), [API Cassandra](how-to-create-container-cassandra.md), [API Gremlin](how-to-create-container-gremlin.md) et [API Table](how-to-create-container-table.md) pour créer le conteneur.

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

## <a name="create-a-container-using-azure-portal"></a><a id="portal-sql"></a>Créer un conteneur à l’aide du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-sql-api-dotnet.md#create-account) ou sélectionnez un compte existant.

1. Ouvrez le volet **Explorateur de données**, puis sélectionnez **Nouveau conteneur**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez une base de données ou si vous utilisez une base de données existante.
   * Entrez un ID de conteneur.
   * Entrez une clé de partition.
   * Entrez un débit à provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-sql.png" alt-text="Capture d’écran du volet Explorateur de données, avec l’option Nouveau conteneur mise en surbrillance":::

## <a name="create-a-container-using-azure-cli"></a><a id="cli-sql"></a>Créer un conteneur à l’aide d’Azure CLI

[Créer un conteneur à l'aide d'Azure CLI](manage-with-cli.md#create-a-container). Pour obtenir la liste de tous les exemples propres à l’interface Azure CLI dans toutes les API Azure Cosmos DB, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples.md).

## <a name="create-a-container-using-powershell"></a>Créer un conteneur à l’aide de PowerShell

[Créer un conteneur à l’aide de PowerShell](manage-with-powershell.md#create-container). Pour obtenir la liste de tous les exemples PowerShell pour toutes les API Azure Cosmos DB, consultez [Exemples PowerShell](powershell-samples.md)

## <a name="create-a-container-using-net-sdk"></a><a id="dotnet-sql"></a>Créer un conteneur à l’aide du kit de développement logiciel (SDK) .NET

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

## <a name="next-steps"></a>Étapes suivantes

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Unités de requête dans Azure Cosmos DB](request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Utiliser un compte Azure Cosmos](./account-databases-containers-items.md)