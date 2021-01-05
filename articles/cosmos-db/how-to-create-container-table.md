---
title: Créer un conteneur dans l’API Table Azure Cosmos DB
description: Découvrez comment créer un conteneur dans l’API Table Azure Cosmos DB avec le portail Azure et les Kits de développement logiciel (SDK) .Net, Java, Python, Node.js ou autres.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: fde3145e7bd7f4e53ae7a0c44f02e066c28ec785
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93101576"
---
# <a name="create-a-container-in-azure-cosmos-db-table-api"></a>Créer un conteneur dans l’API Table Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Cet article explique les différentes façons de créer un conteneur dans l’API Table Azure Cosmos DB. Il montre comment créer un conteneur à l’aide de Portail Azure, d’Azure CLI, de PowerShell ou des Kits de développement logiciel (SDK) pris en charge. Cet article montre comment créer un conteneur, spécifier la clé de partition et provisionner le débit.

Cet article explique les différentes façons de créer un conteneur dans l’API Table Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles relatifs à l’[API pour MongoDB](how-to-create-container-mongodb.md), à l’[API Cassandra](how-to-create-container-cassandra.md), à l’[API Gremlin](how-to-create-container-gremlin.md) et à l’[API SQL](how-to-create-container.md) pour créer le conteneur.

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

## <a name="create-using-azure-portal"></a><a id="portal-table"></a>Créer en utilisant le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-table-dotnet.md#create-a-database-account) ou sélectionnez un compte existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouvelle table**. Fournissez ensuite les détails suivants :

   * Entrez un ID de table.
   * Entrez un débit à provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-table.png" alt-text="Capture d’écran de l’API Table, boîte de dialogue Ajouter une table":::

> [!Note]
> Pour l’API Table, la clé de partition est spécifiée chaque fois que vous ajoutez une nouvelle ligne.

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Créer à l’aide d’Azure CLI

[Créer une table API Table à l’aide d’Azure CLI](./scripts/cli/table/create.md). Pour obtenir la liste de tous les exemples propres à l’interface Azure CLI dans toutes les API Azure Cosmos DB, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Créer à l’aide de PowerShell

[Créer une table API Table à l’aide de PowerShell](./scripts/powershell/table/create.md). Pour obtenir la liste de tous les exemples PowerShell pour toutes les API Azure Cosmos DB, consultez [Exemples PowerShell](powershell-samples.md)

## <a name="next-steps"></a>Étapes suivantes

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Unités de requête dans Azure Cosmos DB](request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Utiliser un compte Azure Cosmos](./account-databases-containers-items.md)