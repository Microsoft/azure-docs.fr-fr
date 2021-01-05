---
title: Approvisionner le débit sur des ressources de l’API Azure Cosmos DB pour MongoDB
description: Découvrez comment approvisionner le débit des conteneurs et des bases de données et la mise à l’échelle automatique sur des ressources de l’API Azure Cosmos DB pour MongoDB. Vous utiliserez le portail Azure, CLI, PowerShell et d’autres Kits de développement logiciel (SDK).
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 25eaa4a66fb4a73f976edbd30e6f82015ce84f6f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93086123"
---
# <a name="provision-database-container-or-autoscale-throughput-on-azure-cosmos-db-api-for-mongodb-resources"></a>Approvisionner le débit des conteneurs et des bases de données et la mise à l’échelle automatique sur des ressources de l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Cet article explique comment approvisionner le débit dans l’API Azure Cosmos DB pour MongoDB. Vous pouvez approvisionner un débit standard (manuel) ou avec mise à l’échelle automatique sur un conteneur ou sur une base de données et le partager entre les conteneurs de la base de données. Vous pouvez approvisionner le débit à l’aide du portail Azure, d’Azure CLI ou des Kits de développement logiciel (SDK) Azure Cosmos DB.

Si vous utilisez une autre API, consultez les articles relatifs à l’[API SQL](how-to-provision-container-throughput.md), à l’[API Cassandra](how-to-provision-throughput-cassandra.md) et à l’[API Gremlin](how-to-provision-throughput-gremlin.md) pour approvisionner le débit.

## <a name="azure-portal"></a><a id="portal-mongodb"></a> Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) ou sélectionnez un compte Azure Cosmos existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouvelle collection**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez une base de données ou si vous utilisez une base de données existante. Sélectionnez l’option **Approvisionner le débit d’une base de données** si vous souhaitez approvisionner le débit au niveau de la base de données.
   * Entrez un ID de collection.
   * Entrez une valeur de clé de partition (par exemple `/ItemID`).
   * Entrez un débit que vous voulez provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-provision-throughput-mongodb/provision-database-throughput-portal-mongodb-api.png" alt-text="Capture d’écran d’Explorateur de données, lors de la création d’une nouvelle collection avec un débit au niveau de la base de données":::

> [!Note]
> Si vous provisionnez le débit sur un conteneur dans un compte Azure Cosmos configuré avec l’API Azure Cosmos DB pour MongoDB, utilisez `/myShardKey` pour le chemin de clé de partition.

## <a name="net-sdk"></a><a id="dotnet-mongodb"></a> Kit de développement logiciel (SDK) .NET

```csharp
// refer to MongoDB .NET Driver
// https://docs.mongodb.com/drivers/csharp

// Create a new Client
String mongoConnectionString = "mongodb://DBAccountName:Password@DBAccountName.documents.azure.com:10255/?ssl=true&replicaSet=globaldb";
mongoUrl = new MongoUrl(mongoConnectionString);
mongoClientSettings = MongoClientSettings.FromUrl(mongoUrl);
mongoClient = new MongoClient(mongoClientSettings);

// Change the database name
mongoDatabase = mongoClient.GetDatabase("testdb");

// Change the collection name, throughput value then update via MongoDB extension commands
// https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb-custom-commands#update-collection

var result = mongoDatabase.RunCommand<BsonDocument>(@"{customAction: ""UpdateCollection"", collection: ""testcollection"", offerThroughput: 400}");
```

## <a name="azure-resource-manager"></a>Azure Resource Manager

Les modèles Azure Resource Manager peuvent être utilisés pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Consultez [Modèles Azure Resource Manager pour Azure Cosmos DB](templates-samples-mongodb.md) afin de voir des exemples.

## <a name="azure-cli"></a>Azure CLI

L’interface Azure CLI peut être utilisée pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Pour voir des exemples, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples-mongodb.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell peut être utilisé pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Pour obtenir des exemples, consultez [Exemples Azure PowerShell pour Azure Cosmos DB](powershell-samples-mongodb.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur le provisionnement du débit dans Azure Cosmos DB :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)