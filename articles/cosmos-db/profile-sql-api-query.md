---
title: Obtenir des métriques de performances et d’exécution de requête SQL
description: Apprenez à récupérer des métriques d’exécution de requête SQL, ainsi qu’à profiler les performances de requête SQL de demandes Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/17/2019
ms.author: girobins
ms.custom: devx-track-csharp
ms.openlocfilehash: 66aefea441d78303ccd611d9df10eea985d61e7c
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097394"
---
# <a name="get-sql-query-execution-metrics-and-analyze-query-performance-using-net-sdk"></a>Obtenir des métriques d’exécution de requête SQL et analyser les performances de requête à l’aide du Kit de développement logiciel (SDK) .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment profiler les performances de requête SQL sur Azure Cosmos DB. Ce profilage exécutable à l’aide d’un objet `QueryMetrics` récupéré à partir du Kit de développement logiciel (SDK) .NET est détaillé ici. [QueryMetrics](/dotnet/api/microsoft.azure.documents.querymetrics) est un objet fortement typé contenant des informations sur l’exécution de la requête principale. Ces mesures sont décrites plus en détail dans l’article [Réglage des performances des requêtes](./sql-api-query-metrics.md).

## <a name="set-the-feedoptions-parameter"></a>Définir le paramètre FeedOptions

Toutes les surcharges pour [DocumentClient.CreateDocumentQuery](/dotnet/api/microsoft.azure.documents.client.documentclient.createdocumentquery) prennent un paramètre facultatif [FeedOptions](/dotnet/api/microsoft.azure.documents.client.feedoptions). Cette option permet de régler et paramétrer l’exécution de la requête. 

Pour collecter les métriques d’exécution de requête SQL, vous devez définir le paramètre [PopulateQueryMetrics](/dotnet/api/microsoft.azure.documents.client.feedoptions.populatequerymetrics#P:Microsoft.Azure.Documents.Client.FeedOptions.PopulateQueryMetrics) dans l’objet [FeedOptions](/dotnet/api/microsoft.azure.documents.client.feedoptions) sur `true`. La définition de `PopulateQueryMetrics` sur true a pour effet que le paramètre `FeedResponse` contient les valeurs `QueryMetrics` pertinentes. 

## <a name="get-query-metrics-with-asdocumentquery"></a>Obtenir des métriques de requête avec AsDocumentQuery()
L’exemple de code suivant montre comment récupérer des métriques lors de l’utilisation de la méthode [AsDocumentQuery()](/dotnet/api/microsoft.azure.documents.linq.documentqueryable.asdocumentquery) :

```csharp
// Initialize this DocumentClient and Collection
DocumentClient documentClient = null;
DocumentCollection collection = null;

// Setting PopulateQueryMetrics to true in the FeedOptions
FeedOptions feedOptions = new FeedOptions
{
    PopulateQueryMetrics = true
};

string query = "SELECT TOP 5 * FROM c";
IDocumentQuery<dynamic> documentQuery = documentClient.CreateDocumentQuery(Collection.SelfLink, query, feedOptions).AsDocumentQuery();

while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    
    // This dictionary maps the partitionId to the QueryMetrics of that query
    IReadOnlyDictionary<string, QueryMetrics> partitionIdToQueryMetrics = feedResponse.QueryMetrics;
    
    // At this point you have QueryMetrics which you can serialize using .ToString()
    foreach (KeyValuePair<string, QueryMetrics> kvp in partitionIdToQueryMetrics)
    {
        string partitionId = kvp.Key;
        QueryMetrics queryMetrics = kvp.Value;
        
        // Do whatever logging you need
        DoSomeLoggingOfQueryMetrics(query, partitionId, queryMetrics);
    }
}
```
## <a name="aggregating-querymetrics"></a>Agrégation de QueryMetrics

Notez que la section précédente contenait plusieurs appels à la méthode [ExecuteNextAsync](/dotnet/api/microsoft.azure.documents.linq.idocumentquery-1.executenextasync). Chaque appel a retourné un objet `FeedResponse` contenant un dictionnaire de `QueryMetrics`, un pour chaque continuation de la requête. L’exemple suivant montre comment agréger ces `QueryMetrics` à l’aide de LINQ :

```csharp
List<QueryMetrics> queryMetricsList = new List<QueryMetrics>();

while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    
    // This dictionary maps the partitionId to the QueryMetrics of that query
    IReadOnlyDictionary<string, QueryMetrics> partitionIdToQueryMetrics = feedResponse.QueryMetrics;
    queryMetricsList.AddRange(partitionIdToQueryMetrics.Values);
}

// Aggregate the QueryMetrics using the + operator overload of the QueryMetrics class.
QueryMetrics aggregatedQueryMetrics = queryMetricsList.Aggregate((curr, acc) => curr + acc);
Console.WriteLine(aggregatedQueryMetrics);
```

## <a name="grouping-query-metrics-by-partition-id"></a>Regroupement des métriques de requête par ID de partition

Vous pouvez regrouper les `QueryMetrics` par ID de partition. Le regroupement par ID de partition vous permet de voir si une partition spécifique occasionne des problèmes de performances par rapport à d’autres. L’exemple suivant montre comment regrouper les `QueryMetrics` avec LINQ :

```csharp
List<KeyValuePair<string, QueryMetrics>> partitionedQueryMetrics = new List<KeyValuePair<string, QueryMetrics>>();
while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    
    // This dictionary is maps the partitionId to the QueryMetrics of that query
    IReadOnlyDictionary<string, QueryMetrics> partitionIdToQueryMetrics = feedResponse.QueryMetrics;
    partitionedQueryMetrics.AddRange(partitionIdToQueryMetrics.ToList());
}

// Now we are able to group the query metrics by partitionId
IEnumerable<IGrouping<string, KeyValuePair<string, QueryMetrics>>> groupedByQueryMetrics = partitionedQueryMetrics
    .GroupBy(kvp => kvp.Key);

// If we wanted to we could even aggregate the groupedby QueryMetrics
foreach(IGrouping<string, KeyValuePair<string, QueryMetrics>> grouping in groupedByQueryMetrics)
{
    string partitionId = grouping.Key;
    QueryMetrics aggregatedQueryMetricsForPartition = grouping
        .Select(kvp => kvp.Value)
        .Aggregate((curr, acc) => curr + acc);
    DoSomeLoggingOfQueryMetrics(query, partitionId, aggregatedQueryMetricsForPartition);
}
```

## <a name="linq-on-documentquery"></a>LINQ sur DocumentQuery

Vous pouvez également obtenir l’objet `FeedResponse` à partir d’une requête LINQ à l’aide de la méthode `AsDocumentQuery()` :

```csharp
IDocumentQuery<Document> linqQuery = client.CreateDocumentQuery(collection.SelfLink, feedOptions)
    .Take(1)
    .Where(document => document.Id == "42")
    .OrderBy(document => document.Timestamp)
    .AsDocumentQuery();
FeedResponse<Document> feedResponse = await linqQuery.ExecuteNextAsync<Document>();
IReadOnlyDictionary<string, QueryMetrics> queryMetrics = feedResponse.QueryMetrics;
```

## <a name="expensive-queries"></a>Requêtes coûteuses

Vous pouvez capturer les unités de requête consommées par chaque requête afin d’examiner les requêtes coûteuses, c’est-à-dire celles qui consomment un débit élevé. Vous pouvez obtenir le coût de requête à l’aide de la propriété [RequestCharge](/dotnet/api/microsoft.azure.documents.client.feedresponse-1.requestcharge) dans `FeedResponse`. Pour en savoir plus sur la façon d’obtenir le coût de requête à l’aide du portail Azure et de différents kits de développement logiciel (SDK), voir l’article [Rechercher les frais d’unités de requête](find-request-unit-charge.md).

```csharp
string query = "SELECT * FROM c";
IDocumentQuery<dynamic> documentQuery = documentClient.CreateDocumentQuery(Collection.SelfLink, query, feedOptions).AsDocumentQuery();

while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    double requestCharge = feedResponse.RequestCharge
    
    // Log the RequestCharge how ever you want.
    DoSomeLogging(requestCharge);
}
```

## <a name="get-the-query-execution-time"></a>Obtenir le temps d’exécution de requête

Lors du calcul du temps requis pour exécuter une requête côté client, veillez à n’inclure que le temps de l’appel de la méthode `ExecuteNextAsync` et non d’autres parties de votre base de code. Seuls ces appels vous aident à calculer la durée d’exécution de la requête, comme illustré dans l’exemple suivant :

```csharp
string query = "SELECT * FROM c";
IDocumentQuery<dynamic> documentQuery = documentClient.CreateDocumentQuery(Collection.SelfLink, query, feedOptions).AsDocumentQuery();
Stopwatch queryExecutionTimeEndToEndTotal = new Stopwatch();
while (documentQuery.HasMoreResults)
{
    // Execute one continuation of the query
    queryExecutionTimeEndToEndTotal.Start();
    FeedResponse<dynamic> feedResponse = await documentQuery.ExecuteNextAsync();
    queryExecutionTimeEndToEndTotal.Stop();
}

// Log the elapsed time
DoSomeLogging(queryExecutionTimeEndToEndTotal.Elapsed);
```

## <a name="scan-queries-commonly-slow-and-expensive"></a>Requêtes d’analyse (couramment lentes et coûteuses)

Une requête d’analyse est une requête qui n’a pas été servie par l’index, de sorte que de nombreux documents sont chargés avant le renvoi du jeu de résultats.

Vous trouverez ci-dessous un exemple d’analyse :

```sql
SELECT VALUE c.description 
FROM   c 
WHERE UPPER(c.description) = "BABYFOOD, DESSERT, FRUIT DESSERT, WITHOUT ASCORBIC ACID, JUNIOR"
```

Le filtre de cette requête utilise la fonction système UPPER qui n’est pas servie à partir de l’index. L’exécution de cette requête sur une collection volumineuse a généré les métriques de requête suivantes pour la première continuation :

```
QueryMetrics

Retrieved Document Count                 :          60,951
Retrieved Document Size                  :     399,998,938 bytes
Output Document Count                    :               7
Output Document Size                     :             510 bytes
Index Utilization                        :            0.00 %
Total Query Execution Time               :        4,500.34 milliseconds
  Query Preparation Times
    Query Compilation Time               :            0.09 milliseconds
    Logical Plan Build Time              :            0.05 milliseconds
    Physical Plan Build Time             :            0.04 milliseconds
    Query Optimization Time              :            0.01 milliseconds
  Index Lookup Time                      :            0.01 milliseconds
  Document Load Time                     :        4,177.66 milliseconds
  Runtime Execution Times
    Query Engine Times                   :          322.16 milliseconds
    System Function Execution Time       :           85.74 milliseconds
    User-defined Function Execution Time :            0.00 milliseconds
  Document Write Time                    :            0.01 milliseconds
Client Side Metrics
  Retry Count                            :               0
  Request Charge                         :        4,059.95 RUs
```

Notez les valeurs suivantes dans la sortie des métriques de requête :

```
Retrieved Document Count                 :          60,951
Retrieved Document Size                  :     399,998,938 bytes
```

Cette requête a chargé 60 951 documents totalisant 399 998 938 d’octets. Le chargement d’un tel nombre d’octets a un coût élevé en terme d’unités de requête. L’exécution de la requête prend également beaucoup de temps, comme le montre clairement la propriété de durée totale :

```
Total Query Execution Time               :        4,500.34 milliseconds
```

Cela signifie que l’exécution de la requête a pris 4,5 secondes (et il ne s’agissait là que d’une seule continuation).

Pour optimiser cet exemple de requête, évitez d’utiliser la fonction UPPER dans le filtre. Au lieu de cela, lorsque de la création ou de la mise à jour de documents, les valeurs `c.description` doivent être insérées en caractères majuscules. La requête devient alors : 

```sql
SELECT VALUE c.description 
FROM   c 
WHERE c.description = "BABYFOOD, DESSERT, FRUIT DESSERT, WITHOUT ASCORBIC ACID, JUNIOR"
```

Cette requête peut désormais être servie à partir de l’index.

Pour en savoir plus sur le réglage des performances des requêtes, voir l’article [Régler les performances des requêtes](./sql-api-query-metrics.md).

## <a name="references"></a><a id="References"></a>Références

- [Spécification SQL Azure Cosmos DB](./sql-query-getting-started.md)
- [ANSI SQL 2011](https://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=53681)
- [JSON](https://json.org/)
- [LINQ](/previous-versions/dotnet/articles/bb308959(v=msdn.10)) 

## <a name="next-steps"></a>Étapes suivantes

- [Régler les performances des requêtes](sql-api-query-metrics.md)
- [Vue d’ensemble de l’indexation](index-overview.md)
- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)