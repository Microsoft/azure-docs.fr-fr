---
title: Approvisionner le débit sur des ressources de l’API Cassandra d’Azure Cosmos DB
description: Découvrez comment approvisionner le débit des conteneurs et des bases de données et la mise à l’échelle automatique sur des ressources de l’API Cassandra d’Azure Cosmos DB. Vous utiliserez le portail Azure, CLI, PowerShell et d’autres Kits de développement logiciel (SDK).
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 63b633ed67c03a006a154bc69a1aafb4cb4aa6d0
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93086276"
---
# <a name="provision-database-container-or-autoscale-throughput-on-azure-cosmos-db-cassandra-api-resources"></a>Approvisionner le débit des conteneurs et des bases de données et la mise à l’échelle automatique sur des ressources de l’API Cassandra d’Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cet article explique comment approvisionner le débit dans l’API Cassandra d’Azure Cosmos DB. Vous pouvez approvisionner un débit standard (manuel) ou avec mise à l’échelle automatique sur un conteneur ou sur une base de données et le partager entre les conteneurs de la base de données. Vous pouvez approvisionner le débit à l’aide du portail Azure, d’Azure CLI ou des Kits de développement logiciel (SDK) Azure Cosmos DB.

Si vous utilisez une autre API, consultez les articles relatifs à l’[API SQL](how-to-provision-container-throughput.md), à l’[API pour MongoDB](how-to-provision-throughput-mongodb.md) et à l’[API Gremlin](how-to-provision-throughput-gremlin.md) pour approvisionner le débit.

## <a name="azure-portal"></a><a id="portal-cassandra"></a> Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) ou sélectionnez un compte Azure Cosmos existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez **Nouvelle table**. Fournissez ensuite les détails suivants :

   * Indiquez si vous créez un espace de clés ou si vous utilisez un espace de clés existant. Sélectionnez l’option **Approvisionner le débit d’une base de données** si vous souhaitez approvisionner le débit au niveau de l’espace de clés.
   * Entrez l’ID de table dans la commande CQL.
   * Entrez une valeur de clé primaire (par exemple, `/userrID`).
   * Entrez un débit que vous voulez provisionner (par exemple, 1 000 unités de requête).
   * Sélectionnez **OK**.

    :::image type="content" source="./media/how-to-provision-throughput-cassandra/provision-table-throughput-portal-cassandra-api.png" alt-text="Capture d’écran d’Explorateur de données, lors de la création d’une nouvelle collection avec un débit au niveau de la base de données":::

> [!Note]
> Si vous provisionnez le débit sur un conteneur dans un compte Azure Cosmos configuré avec l’API Cassandra, utilisez `/myPrimaryKey` pour le chemin de clé de partition.

## <a name="net-sdk"></a><a id="dotnet-cassandra"></a> Kit de développement logiciel (SDK) .NET

### <a name="provision-throughput-for-a-cassandra-table"></a>Approvisionner le débit pour une table Cassandra

```csharp
// Create a Cassandra table with a partition (primary) key and provision throughput of 400 RU/s
session.Execute("CREATE TABLE myKeySpace.myTable(
    user_id int PRIMARY KEY,
    firstName text,
    lastName text) WITH cosmosdb_provisioned_throughput=400");

```
Des commandes similaires peuvent être émises avec n’importe quel pilote conforme à CQL.

### <a name="alter-or-change-throughput-for-a-cassandra-table"></a>Modifier le débit d’une table Cassandra

```csharp
// Altering the throughput too can be done through code by issuing following command
session.Execute("ALTER TABLE myKeySpace.myTable WITH cosmosdb_provisioned_throughput=5000");
```

Une commande similaire peut être exécutée avec n’importe quel pilote conforme à CQL.

```csharp
// Create a Cassandra keyspace and provision throughput of 400 RU/s
session.Execute("CREATE KEYSPACE IF NOT EXISTS myKeySpace WITH cosmosdb_provisioned_throughput=400");
```

## <a name="azure-resource-manager"></a>Azure Resource Manager

Les modèles Azure Resource Manager peuvent être utilisés pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Consultez [Modèles Azure Resource Manager pour Azure Cosmos DB](templates-samples-cassandra.md) afin de voir des exemples.

## <a name="azure-cli"></a>Azure CLI

L’interface Azure CLI peut être utilisée pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Pour voir des exemples, consultez [Exemples Azure CLI pour Azure Cosmos DB](cli-samples-cassandra.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell peut être utilisé pour provisionner le débit de mise à l’échelle automatique sur des ressources de base de données ou de niveau conteneur pour toutes les API Azure Cosmos DB. Pour obtenir des exemples, consultez [Exemples Azure PowerShell pour Azure Cosmos DB](powershell-samples-cassandra.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur le provisionnement du débit dans Azure Cosmos DB :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)