---
title: Créer un conteneur dans l’API Cassandra d’Azure Cosmos DB
description: Découvrez comment créer un conteneur dans l’API Cassandra d’Azure Cosmos DB à l’aide du portail Azure et des Kits de développement logiciel (SDK) .Net, Java, Python, Node.js ou autres.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 01030a563c15eee7786058c2eae30d23803dfe42
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101644"
---
# <a name="create-a-container-in-azure-cosmos-db-cassandra-api"></a>Créer un conteneur dans l’API Cassandra d’Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article explique les différentes façons de créer un conteneur dans l’API Cassandra d’Azure Cosmos DB. Il montre comment créer un conteneur à l’aide de Portail Azure, d’Azure CLI, de PowerShell ou des Kits de développement logiciel (SDK) pris en charge. Cet article montre comment créer un conteneur, spécifier la clé de partition et provisionner le débit.

Cet article explique les différentes façons de créer un conteneur dans l’API Cassandra d’Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles relatifs à l’[API pour MongoDB](how-to-create-container-mongodb.md), à l’[API Gremlin](how-to-create-container-gremlin.md), à l’[API Table](how-to-create-container-table.md) et à l’[API SQL](how-to-create-container.md) pour créer le conteneur.

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

## <a name="create-using-azure-portal"></a><a id="portal-cassandra"></a>Créer en utilisant le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-cassandra-dotnet.md#create-a-database-account) ou sélectionnez un compte existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouvelle table**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez un espace de clés ou si vous utilisez une base de données existante.
   * Saisissez le nom de la table.
   * Entrez les propriétés et spécifiez une clé primaire.
   * Entrez un débit à provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-cassandra.png" alt-text="Capture d’écran de l’API Cassandra, boîte de dialogue Ajouter une table":::

> [!NOTE]
> Pour l’API Cassandra, la clé primaire est utilisée comme clé de partition.

## <a name="create-using-net-sdk"></a><a id="dotnet-cassandra"></a>Créer à l’aide du SDK .NET

```csharp
// Create a Cassandra table with a partition/primary key and provision 1000 RU/s throughput.
session.Execute(CREATE TABLE myKeySpace.myTable(
    user_id int PRIMARY KEY,
    firstName text,
    lastName text) WITH cosmosdb_provisioned_throughput=1000);
```

Si vous rencontrez une exception de délai d’expiration lors de la création d’une collection, effectuez une opération de lecture pour valider si la collection a été créée avec succès. L’opération de lecture lève une exception jusqu’à ce que l’opération de création de la collection aboutisse. Pour obtenir la liste des codes d’état pris en charge par l’opération de création, consultez l’article [Codes d’état HTTP pour Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Créer à l’aide d’Azure CLI

[Créez une table Cassandra à l’aide d’Azure CLI](./scripts/cli/cassandra/create.md). Pour obtenir la liste de tous les exemples propres à l’interface Azure CLI dans toutes les API Azure Cosmos DB, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Créer à l’aide de PowerShell

[Créez une table Cassandra à l’aide de PowerShell](./scripts/powershell/cassandra/create.md). Pour obtenir la liste de tous les exemples PowerShell pour toutes les API Azure Cosmos DB, consultez [Exemples PowerShell](powershell-samples.md)

## <a name="next-steps"></a>Étapes suivantes

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Unités de requête dans Azure Cosmos DB](request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Utiliser un compte Azure Cosmos](./account-databases-containers-items.md)