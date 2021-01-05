---
title: Créer un conteneur dans API Azure Cosmos DB pour MongoDB
description: Découvrez comment créer un conteneur dans l’API Azure Cosmos DB pour MongoDB avec le portail Azure et les kits SDK .Net, Java, Node.js ou autres.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: a5669b15c041f663605a62ef8d02b206928d0c14
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101593"
---
# <a name="create-a-container-in-azure-cosmos-db-api-for-mongodb"></a>Créer un conteneur dans API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Cet article explique les différentes façons de créer un conteneur dans l’API Azure Cosmos DB pour MongoDB. Il montre comment créer un conteneur à l’aide de Portail Azure, Azure CLI, PowerShell ou des kits de développement logiciel (SDK) pris en charge. Cet article montre comment créer un conteneur, spécifier la clé de partition et provisionner le débit.

Cet article explique les différentes façons de créer un conteneur dans l’API Azure Cosmos DB pour MongoDB. Si vous utilisez une autre API, consultez les articles [API SQL](how-to-create-container.md), [API Cassandra](how-to-create-container-cassandra.md), [API Gremlin](how-to-create-container-gremlin.md) et [API Table](how-to-create-container-table.md) pour créer le conteneur.

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

## <a name="create-using-azure-portal"></a><a id="portal-mongodb"></a>Créer en utilisant le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) ou sélectionnez un compte existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouveau conteneur**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez une base de données ou si vous utilisez une base de données existante.
   * Entrez un ID de conteneur.
   * Entrez une clé de sharding.
   * Entrez un débit à provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-mongodb.png" alt-text="Capture d'écran de l'API Azure Cosmos DB pour MongoDB, boîte de dialogue Ajouter un conteneur":::

## <a name="create-using-net-sdk"></a><a id="dotnet-mongodb"></a>Créer à l’aide du SDK .NET

```csharp
// Create a collection with a partition key by using Mongo Shell:
db.runCommand( { shardCollection: "myDatabase.myCollection", key: { myShardKey: "hashed" } } )
```

> [!Note]
> Le protocole filaire MongoDB ne comprend pas le concept d’[unités de requête](request-units.md). Pour créer une collection avec un débit provisionné dessus, utilisez le portail Azure ou les SDK Cosmos DB pour l’API SQL.

Si vous rencontrez une exception de délai d’expiration lors de la création d’une collection, effectuez une opération de lecture pour valider si la collection a été créée avec succès. L’opération de lecture lève une exception jusqu’à ce que l’opération de création de la collection aboutisse. Pour obtenir la liste des codes d’état pris en charge par l’opération de création, consultez l’article [Codes d’état HTTP pour Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Créer à l’aide d’Azure CLI

[Créer une collection pour l'API Azure Cosmos DB pour MongoDB à l'aide d'Azure CLI](./scripts/cli/mongodb/create.md). Pour obtenir la liste de tous les exemples propres à l’interface Azure CLI dans toutes les API Azure Cosmos DB, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Créer à l’aide de PowerShell

[Créer une collection pour l’API Azure Cosmos DB pour MongoDB à l’aide de PowerShell](./scripts/powershell/mongodb/create.md). Pour obtenir la liste de tous les exemples PowerShell pour toutes les API Azure Cosmos DB, consultez [Exemples PowerShell](powershell-samples.md)

## <a name="create-a-container-using-azure-resource-manager-templates"></a>Créer un conteneur à l’aide de modèles Azure Resource Manager

[Créer une collection pour l’API Azure Cosmos DB pour MongoDB à l’aide d’un modèle Resource Manager](./manage-with-templates.md#azure-cosmos-account-with-standard-provisioned-throughput).

## <a name="next-steps"></a>Étapes suivantes

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Unités de requête dans Azure Cosmos DB](request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Utiliser un compte Azure Cosmos](./account-databases-containers-items.md)