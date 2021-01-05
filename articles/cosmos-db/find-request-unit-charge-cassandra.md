---
title: Rechercher les frais d’unités de requête (RU) pour une requête de l’API Cassandra dans Azure Cosmos DB
description: Découvrez comment trouver les frais d’unités de requête pour les requêtes Cassandra exécutées sur un conteneur Azure Cosmos. Vous pouvez utiliser le portail Azure et les pilotes .NET et Java pour rechercher les frais de RU.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 8f7db5dc48f7fec1bcf2bcd1291b67da7b4412c7
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93094810"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-cassandra-api"></a>Rechercher les frais des unités de requête pour les opérations exécutées dans l’API Cassandra d’Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Azure Cosmos DB prend en charge de nombreuses API, notamment SQL, MongoDB, Cassandra, Gremlin et Table. Chaque API possède son propre ensemble d’opérations de base de données, qui vont de simples opérations ponctuelles de lecture et d'écriture à des requêtes complexes. Chaque opération de base de données consomme des ressources système en fonction de sa complexité.

Le coût de toutes les opérations de base de données, normalisé par Azure Cosmos DB, est exprimé en unités de requête (RU). Les RU correspondent en quelque sorte à une devise de performances, faisant abstraction des ressources système (UC, IOPS, mémoire, etc.) requises pour effectuer les opérations de base de données prises en charge par Azure Cosmos DB. Quelle que soit l’API utilisée pour interagir avec le conteneur Azure Cosmos, les coûts sont toujours mesurés en unités de requête, Que l’opération de base de données soit une opération d’écriture, de lecture de point ou de requête, les coûts sont toujours mesurés en unités de requête. Pour plus d’informations, consultez l’article [Unités de requête et considérations](request-units.md).

Cet article présente les différentes façons de rechercher la consommation d’[unités de requête](request-units.md) pour toute opération exécutée sur un conteneur dans l’API Cassandra d’Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles relatifs à l’[API pour MongoDB](find-request-unit-charge-mongodb.md), à l’[API Cassandra](find-request-unit-charge.md), à l’[API Gremlin](find-request-unit-charge-gremlin.md) et à l’[API Table](find-request-unit-charge-table.md) pour trouver les frais de RU/s.

Quand vous effectuez des opérations sur l’API Cassandra Azure Cosmos DB, les frais d’unités de requête sont retournés dans la charge utile entrante sous la forme d’un champ nommé `RequestCharge`. Vous disposez de plusieurs options pour récupérer les frais d’unités de requête.

## <a name="use-the-net-sdk"></a>Utiliser le kit de développement logiciel (SDK) .NET

Lorsque vous utilisez le [Kit de développement logiciel (SDK) .NET](https://www.nuget.org/packages/CassandraCSharpDriver/), vous pouvez récupérer la charge utile entrante sous la propriété `Info` d’un objet `RowSet` :

```csharp
RowSet rowSet = session.Execute("SELECT table_name FROM system_schema.tables;");
double requestCharge = BitConverter.ToDouble(rowSet.Info.IncomingPayload["RequestCharge"].Reverse().ToArray(), 0);
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application Cassandra à l’aide du Kit de développement logiciel (SDK) .NET et d’Azure Cosmos DB](create-cassandra-dotnet.md).

## <a name="use-the-java-sdk"></a>Utiliser le SDK Java

Lorsque vous utilisez le [Kit de développement logiciel (SDK) Java](https://mvnrepository.com/artifact/com.datastax.cassandra/cassandra-driver-core), vous pouvez récupérer la charge utile entrante en appelant la méthode `getExecutionInfo()` sur un objet `ResultSet` :

```java
ResultSet resultSet = session.execute("SELECT table_name FROM system_schema.tables;");
Double requestCharge = resultSet.getExecutionInfo().getIncomingPayload().get("RequestCharge").getDouble();
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application Cassandra à l’aide du Kit de développement logiciel (SDK) Java et d’Azure Cosmos DB](create-cassandra-java.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’optimisation de votre consommation d’unités de requête, consultez les articles suivants :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)
* [Optimiser le coût du débit approvisionné dans Azure Cosmos DB](optimize-cost-throughput.md)
* [Optimiser le coût de requête dans Azure Cosmos DB](./optimize-cost-reads-writes.md)