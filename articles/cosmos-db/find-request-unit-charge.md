---
title: Rechercher les frais d’unités de requête (RU) pour une requête SQL dans Azure Cosmos DB
description: Découvrez comment trouver les frais d’unités de requête pour les requêtes SQL exécutées sur un conteneur Azure Cosmos. Vous pouvez utiliser le portail Azure et les langages .NET, Java, Python et Node.js pour rechercher les frais de RU.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 9d0694a76bca832887d30601711894b953fe22e1
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93078439"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-sql-api"></a>Rechercher les frais des unités de requête pour les opérations exécutées dans l’API SQL d’Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB prend en charge de nombreuses API, notamment SQL, MongoDB, Cassandra, Gremlin et Table. Chaque API possède son propre ensemble d’opérations de base de données, qui vont de simples opérations ponctuelles de lecture et d'écriture à des requêtes complexes. Chaque opération de base de données consomme des ressources système en fonction de sa complexité.

Le coût de toutes les opérations de base de données, normalisé par Azure Cosmos DB, est exprimé en unités de requête (RU). Les RU correspondent en quelque sorte à une devise de performances, faisant abstraction des ressources système (UC, IOPS, mémoire, etc.) requises pour effectuer les opérations de base de données prises en charge par Azure Cosmos DB. Quelle que soit l’API utilisée pour interagir avec le conteneur Azure Cosmos, les coûts sont toujours mesurés en unités de requête, Que l’opération de base de données soit une opération d’écriture, de lecture de point ou de requête, les coûts sont toujours mesurés en unités de requête. Pour plus d’informations, consultez l’article [Unités de requête et considérations](request-units.md).

Cet article présente les différentes façons de rechercher la consommation d’[unités de requête](request-units.md) pour toute opération exécutée sur un conteneur dans l’API SQL d’Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles relatifs à l’[API pour MongoDB](find-request-unit-charge-mongodb.md), à l’[API Cassandra](find-request-unit-charge-cassandra.md), à l’[API Gremlin](find-request-unit-charge-gremlin.md) et à l’[API Table](find-request-unit-charge-table.md) pour trouver les frais de RU/s.

Il n’est actuellement possible de mesurer cette consommation que sur le portail Azure ou en examinant la réponse renvoyée par Azure Cosmos DB via l’un des kits de développement logiciel (SDK). Si vous utilisez l’API SQL, vous disposez de plusieurs options pour trouver la consommation d’unités de requête pour une opération sur un conteneur Azure Cosmos.

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-sql-api-dotnet.md#create-account) et remplissez-le avec des données, ou sélectionnez un compte Azure Cosmos existant qui contient déjà des données.

1. Accédez au volet **Explorateur de données** , puis sélectionnez le conteneur sur lequel vous voulez travailler.

1. Sélectionnez **Nouvelle requête SQL**.

1. Entrez une requête valide, puis sélectionnez **Exécuter la requête**.

1. Sélectionnez **Statistiques des requêtes** pour afficher les frais réels de la demande que vous avez exécutée.

:::image type="content" source="./media/find-request-unit-charge/portal-sql-query.png" alt-text="Capture d’écran des frais de demande de requête SQL sur le portail Azure":::

## <a name="use-the-net-sdk"></a>Utiliser le kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

Les objets retournés à partir du [Kit de développement logiciel (SDK) .NET v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) exposent une propriété `RequestCharge` :

```csharp
ResourceResponse<Document> fetchDocumentResponse = await client.ReadDocumentAsync(
    UriFactory.CreateDocumentUri("database", "container", "itemId"),
    new RequestOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    });
var requestCharge = fetchDocumentResponse.RequestCharge;

StoredProcedureResponse<string> storedProcedureCallResponse = await client.ExecuteStoredProcedureAsync<string>(
    UriFactory.CreateStoredProcedureUri("database", "container", "storedProcedureId"),
    new RequestOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    });
requestCharge = storedProcedureCallResponse.RequestCharge;

IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri("database", "container"),
    "SELECT * FROM c",
    new FeedOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    }).AsDocumentQuery();
while (query.HasMoreResults)
{
    FeedResponse<dynamic> queryResponse = await query.ExecuteNextAsync<dynamic>();
    requestCharge = queryResponse.RequestCharge;
}
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

Les objets retournés à partir du [SDK .NET v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) exposent une propriété `RequestCharge` :

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/CustomDocsSampleCode.cs?name=GetRequestCharge)]

Pour plus d’informations, consultez [Démarrage rapide : Générer une application web .NET à l’aide d’un compte d’API SQL dans Azure Cosmos DB](create-sql-api-dotnet.md).

---

## <a name="use-the-java-sdk"></a>Utiliser le SDK Java

Les objets retournés à partir du [Kit de développement logiciel Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) exposent une méthode `getRequestCharge()` :

```java
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPartitionKey(new PartitionKey("partitionKey"));

Observable<ResourceResponse<Document>> readDocumentResponse = client.readDocument(String.format("/dbs/%s/colls/%s/docs/%s", "database", "container", "itemId"), requestOptions);
readDocumentResponse.subscribe(result -> {
    double requestCharge = result.getRequestCharge();
});

Observable<StoredProcedureResponse> storedProcedureResponse = client.executeStoredProcedure(String.format("/dbs/%s/colls/%s/sprocs/%s", "database", "container", "storedProcedureId"), requestOptions, null);
storedProcedureResponse.subscribe(result -> {
    double requestCharge = result.getRequestCharge();
});

FeedOptions feedOptions = new FeedOptions();
feedOptions.setPartitionKey(new PartitionKey("partitionKey"));

Observable<FeedResponse<Document>> feedResponse = client
    .queryDocuments(String.format("/dbs/%s/colls/%s", "database", "container"), "SELECT * FROM c", feedOptions);
feedResponse.forEach(result -> {
    double requestCharge = result.getRequestCharge();
});
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application Java à l’aide d’un compte d’API SQL Azure Cosmos DB](create-sql-api-java.md).

## <a name="use-the-nodejs-sdk"></a>Utiliser le SDK Node.js

Les objets retournés à partir du [Kit de développement logiciel (SDK) Node.js](https://www.npmjs.com/package/@azure/cosmos) exposent un sous-objet `headers` qui mappe tous les en-têtes retournés par l’API HTTP sous-jacente. Les frais de demande sont disponibles sous la clé `x-ms-request-charge` :

```javascript
const item = await client
    .database('database')
    .container('container')
    .item('itemId', 'partitionKey')
    .read();
var requestCharge = item.headers['x-ms-request-charge'];

const storedProcedureResult = await client
    .database('database')
    .container('container')
    .storedProcedure('storedProcedureId')
    .execute({
        partitionKey: 'partitionKey'
    });
requestCharge = storedProcedureResult.headers['x-ms-request-charge'];

const query = client.database('database')
    .container('container')
    .items
    .query('SELECT * FROM c', {
        partitionKey: 'partitionKey'
    });
while (query.hasMoreResults()) {
    var result = await query.executeNext();
    requestCharge = result.headers['x-ms-request-charge'];
}
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application Node.js à l’aide d’un compte d’API SQL Azure Cosmos DB](create-sql-api-nodejs.md). 

## <a name="use-the-python-sdk"></a>Utiliser le SDK Python

L’objet `CosmosClient` retourné à partir du [Kit de développement logiciel (SDK) Python](https://pypi.org/project/azure-cosmos/) expose un dictionnaire `last_response_headers` qui mappe tous les en-têtes retournés par l’API HTTP sous-jacente pour la dernière opération exécutée. Les frais de demande sont disponibles sous la clé `x-ms-request-charge` :

```python
response = client.ReadItem(
    'dbs/database/colls/container/docs/itemId', {'partitionKey': 'partitionKey'})
request_charge = client.last_response_headers['x-ms-request-charge']

response = client.ExecuteStoredProcedure(
    'dbs/database/colls/container/sprocs/storedProcedureId', None, {'partitionKey': 'partitionKey'})
request_charge = client.last_response_headers['x-ms-request-charge']
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application Python à l’aide d’un compte d’API SQL Azure Cosmos DB](create-sql-api-python.md). 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’optimisation de votre consommation d’unités de requête, consultez les articles suivants :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)
* [Optimiser le coût du débit approvisionné dans Azure Cosmos DB](optimize-cost-throughput.md)
* [Optimiser le coût de requête dans Azure Cosmos DB](./optimize-cost-reads-writes.md)
* [Mettre à l’échelle le débit provisionné au niveau global](./request-units.md)
* [Provisionner le débit sur les conteneurs et les bases de données](set-throughput.md)
* [Approvisionner le débit pour un conteneur](how-to-provision-container-throughput.md)
* [Superviser et déboguer à l’aide de métriques dans Azure Cosmos DB](use-metrics.md)