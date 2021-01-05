---
title: Métriques des requêtes SQL pour l’API SQL dans Azure Cosmos DB
description: Découvrez comment instrumenter et déboguer les performances de requêtes SQL des demandes Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/23/2019
ms.author: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: fedcdd55a465f5c09c331a0fa917811c349b15b1
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097224"
---
# <a name="tuning-query-performance-with-azure-cosmos-db"></a>Réglage des performances de requête avec Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB fournit une [API SQL pour interroger des données](./sql-query-getting-started.md), sans nécessiter de schéma ou d’index secondaires. Cet article fournit les informations suivantes à l’attention des développeurs :

* Détails techniques sur le fonctionnement de l’exécution de requêtes SQL dans Azure Cosmos DB
* Développements sur les en-têtes de demande et réponse des requêtes, et sur les options du kit SDK client
* Conseils et meilleures pratiques pour les performances de requêtes
* Exemples montrant comment utiliser les statistiques d’exécution de SQL pour déboguer les performances de requêtes

## <a name="about-sql-query-execution"></a>À propos de l’exécution de requêtes SQL

Dans Azure Cosmos DB, vous stockez les données dans des conteneurs pouvant se remplir à n’importe quel [débit de requête ou taille de stockage](partitioning-overview.md). Azure Cosmos DB adapte de façon continue les données entre les partitions physiques situées en arrière-plan, pour gérer la croissance des données ou l’augmentation de débit approvisionné. Vous pouvez émettre des requêtes SQL sur n’importe quel conteneur à l’aide de l’API REST ou de l’un des [kits SDK SQL](sql-api-sdk-dotnet.md) pris en charge.

Pour résumer brièvement le partitionnement : vous définissez une clé de partition, par exemple "city", qui détermine la façon dont les données sont fractionnées entre plusieurs partitions physiques. Les données appartenant à une clé de partition unique (par exemple, "city" == "Seattle") sont stockées dans une partition physique, bien qu’en général une seule partition physique ait plusieurs clés de partition. Lorsqu’une partition a atteint sa taille de stockage, le service divise de façon homogène la partition en deux nouvelles partitions, et répartit la clé de partition uniformément entre ces partitions. Étant donné que les partitions sont temporaires, les API utilisent une abstraction de « plage de clé de partition » représentant les plages des hachages de clé de partition. 

Lorsque vous exécutez une requête sur Azure Cosmos DB, le kit SDK exécute les étapes logiques suivantes :

* Analyse de la requête SQL pour déterminer le plan d’exécution de la requête. 
* Si la requête contient un filtre sur la clé de partition, tel que `SELECT * FROM c WHERE c.city = "Seattle"`, elle est routée vers une partition unique. Si la requête n’a pas de filtre sur la clé de partition, elle est alors exécutée dans toutes les partitions, et les résultats sont fusionnés côté client.
* La requête est exécutée dans chaque partition, en série ou en parallèle, selon la configuration du client. Dans chaque partition, la requête risque de faire un ou plusieurs allers-retours selon la complexité de la requête, la taille de page configurée et le débit approvisionné de la collection. Chaque exécution retourne le nombre d’[unités de requête](request-units.md) consommées par exécution de la requête et, éventuellement, des statistiques d’exécution de la requête. 
* Le kit SDK réalise une synthèse des résultats de la requête sur les partitions. Par exemple, si la requête implique une clause ORDER BY sur l’ensemble des partitions, les résultats de chacune des partitions sont fusionnés et triés pour retourner un résultat global trié. Si la requête est une agrégation telle que `COUNT`, les nombres provenant de chaque partition sont additionnés pour obtenir le nombre total.

Les kits SDK offrent diverses options pour l’exécution de requêtes. Par exemple, dans .NET ces options sont disponibles dans la classe `FeedOptions`. Le tableau suivant décrit ces options ainsi que leur impact sur la durée d’exécution de la requête. 

| Option | Description |
| ------ | ----------- |
| `EnableCrossPartitionQuery` | Doit être définie sur true pour une requête qui exige son exécution sur plusieurs partitions. Il s’agit d’un indicateur explicite pour vous permettre de faire des compromis de performance conscients au moment du développement. |
| `EnableScanInQuery` | Doit être définie sur true si vous avez refusé l’indexation, mais souhaitez quand même exécuter la requête via une analyse. Uniquement applicable si l’indexation pour le chemin du filtre demandé est désactivé. | 
| `MaxItemCount` | Le nombre maximal d’éléments à retourner par aller-retour au serveur. Avec la valeur -1, vous pouvez laisser le serveur gérer le nombre d’éléments. Sinon, vous pouvez réduire cette valeur et récupérer uniquement un petit nombre d’éléments par aller-retour. 
| `MaxBufferedItemCount` | C’est une option côté client qui permet de limiter la consommation de mémoire lors de l’exécution sur plusieurs partitions de ORDER BY. Une valeur plus élevée permet de réduire la latence de tri de partitions croisées. |
| `MaxDegreeOfParallelism` | Obtient ou définit le nombre d’opérations simultanées qui sont exécutées côté client, lors de l’exécution de la requête en parallèle, dans le service de base de données Azure Cosmos. Une valeur de propriété positive limite le nombre d’opérations simultanées à la valeur définie. Si elle est définie sur une valeur inférieure à 0, le système détermine automatiquement le nombre d’opérations simultanées à exécuter. |
| `PopulateQueryMetrics` | Permet une journalisation détaillée des statistiques du temps passé dans les différentes phases d’exécution de la requête, telles que le moment de la compilation, la durée de la boucle d’indexation et la durée de chargement des documents. À partir des statistiques de requête, vous pouvez partager le résultat avec le support Azure pour diagnostiquer les problèmes de performances de requêtes. |
| `RequestContinuation` | Vous pouvez reprendre l’exécution de la requête en passant le jeton de continuation opaque retourné par une requête. Le jeton de continuation encapsule tous les états requis pour l’exécution de la requête. |
| `ResponseContinuationTokenLimitInKb` | Vous pouvez limiter la taille maximale du jeton de continuation retourné par le serveur. Vous devrez peut-être définir cette option si votre hôte d’application a des limites de taille d’en-tête de réponse. La définition de cette propriété peut augmenter la durée globale et les unités de requête (RU) consommées pour la requête.  |

Ainsi, prenons un exemple de requête sur la clé de partition demandée sur une collection, avec `/city` en tant que clé de partition, et une configuration de débit de 100 000 RU/s. Vous demandez cette requête à l’aide de `CreateDocumentQuery<T>` dans .NET comme suit :

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PopulateQueryMetrics = true, 
        MaxItemCount = -1, 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();

FeedResponse<dynamic> result = await query.ExecuteNextAsync();
```

L’extrait de code du kit SDK illustré ci-dessus correspond à la demande de l’API REST suivante :

```
POST https://arramacquerymetrics-westus.documents.azure.com/dbs/db/colls/sample/docs HTTP/1.1
x-ms-continuation: 
x-ms-documentdb-isquery: True
x-ms-max-item-count: -1
x-ms-documentdb-query-enablecrosspartition: True
x-ms-documentdb-query-parallelizecrosspartitionquery: True
x-ms-documentdb-query-iscontinuationexpected: True
x-ms-documentdb-populatequerymetrics: True
x-ms-date: Tue, 27 Jun 2017 21:52:18 GMT
authorization: type%3dmaster%26ver%3d1.0%26sig%3drp1Hi83Y8aVV5V6LzZ6xhtQVXRAMz0WNMnUuvriUv%2b4%3d
x-ms-session-token: 7:8,6:2008,5:8,4:2008,3:8,2:2008,1:8,0:8,9:8,8:4008
Cache-Control: no-cache
x-ms-consistency-level: Session
User-Agent: documentdb-dotnet-sdk/1.14.1 Host/32-bit MicrosoftWindowsNT/6.2.9200.0
x-ms-version: 2017-02-22
Accept: application/json
Content-Type: application/query+json
Host: arramacquerymetrics-westus.documents.azure.com
Content-Length: 52
Expect: 100-continue

{"query":"SELECT * FROM c WHERE c.city = 'Seattle'"}
```

Chaque page d’exécution de requête correspond à une API REST `POST`, avec l’en-tête `Accept: application/query+json`, et la requête SQL dans le corps. Chaque requête effectue un ou plusieurs allers-retours au serveur avec le jeton `x-ms-continuation` répercuté entre le client et le serveur pour reprendre l’exécution. Les options de configuration dans FeedOptions sont passées au serveur sous la forme d’en-têtes de demande. Par exemple, `MaxItemCount` correspond à `x-ms-max-item-count`. 

La requête retourne la réponse suivante (tronquée pour une meilleure lisibilité) :

```
HTTP/1.1 200 Ok
Cache-Control: no-store, no-cache
Pragma: no-cache
Transfer-Encoding: chunked
Content-Type: application/json
Server: Microsoft-HTTPAPI/2.0
Strict-Transport-Security: max-age=31536000
x-ms-last-state-change-utc: Tue, 27 Jun 2017 21:01:57.561 GMT
x-ms-resource-quota: documentSize=10240;documentsSize=10485760;documentsCount=-1;collectionSize=10485760;
x-ms-resource-usage: documentSize=1;documentsSize=884;documentsCount=2000;collectionSize=1408;
x-ms-item-count: 2000
x-ms-schemaversion: 1.3
x-ms-alt-content-path: dbs/db/colls/sample
x-ms-content-path: +9kEANVq0wA=
x-ms-xp-role: 1
x-ms-documentdb-query-metrics: totalExecutionTimeInMs=33.67;queryCompileTimeInMs=0.06;queryLogicalPlanBuildTimeInMs=0.02;queryPhysicalPlanBuildTimeInMs=0.10;queryOptimizationTimeInMs=0.00;VMExecutionTimeInMs=32.56;indexLookupTimeInMs=0.36;documentLoadTimeInMs=9.58;systemFunctionExecuteTimeInMs=0.00;userFunctionExecuteTimeInMs=0.00;retrievedDocumentCount=2000;retrievedDocumentSize=1125600;outputDocumentCount=2000;writeOutputTimeInMs=18.10;indexUtilizationRatio=1.00
x-ms-request-charge: 604.42
x-ms-serviceversion: version=1.14.34.4
x-ms-activity-id: 0df8b5f6-83b9-4493-abda-cce6d0f91486
x-ms-session-token: 2:2008
x-ms-gatewayversion: version=1.14.33.2
Date: Tue, 27 Jun 2017 21:59:49 GMT
```

Les en-têtes de réponse de la clé retournés par la requête comprennent les éléments suivants :

| Option | Description |
| ------ | ----------- |
| `x-ms-item-count` | Le nombre d’éléments retournés dans la réponse. Cela dépend du `x-ms-max-item-count` fourni, du nombre d’éléments qui peuvent être contenus dans la taille maximale de la charge utile de la réponse, du débit approvisionné et de la durée d’exécution de la requête. |  
| `x-ms-continuation:` | Le jeton de continuation pour reprendre l’exécution de la requête, si des résultats supplémentaires sont disponibles. | 
| `x-ms-documentdb-query-metrics` | Les statistiques de la requête pour l’exécution. Il s’agit d’une chaîne délimitée qui contient des statistiques de temps passé dans les différentes phases d’exécution de la requête. Retournées si `x-ms-documentdb-populatequerymetrics` a la valeur `True`. | 
| `x-ms-request-charge` | Le nombre d’[unités de requête](request-units.md) consommées par la requête. | 

Pour plus d’informations sur les options et les en-têtes de demande d’API REST, consultez [Interrogation des ressources à l’aide de l’API REST](/rest/api/cosmos-db/querying-cosmosdb-resources-using-the-rest-api).

## <a name="best-practices-for-query-performance"></a>Meilleures pratiques pour les performances de requêtes
Les éléments suivants sont les facteurs les plus courants qui affectent les performances de requêtes Azure Cosmos DB. Nous approfondissons chacune de ces rubriques dans cet article.

| Factor | Conseil | 
| ------ | -----| 
| Débit approvisionné | Mesurez la RU (unité de requête) par requête et assurez-vous d’avoir le débit approvisionné nécessaire pour vos requêtes. | 
| Partitionnement et clés de partition | Préférez les requêtes avec la valeur de clé de partition dans la clause de filtre pour une latence faible. |
| Options de requête et de kit SDK | Suivez les meilleures pratiques du kit SDK, comme la connectivité directe, et paramétrez les options d’exécution de requêtes du côté client. |
| Latence du réseau | Prenez en compte la surcharge réseau dans la prise de mesures, et utilisez des API d’hébergement multiple pour lire à partir de la région la plus proche. |
| Stratégie d’indexation | Assurez-vous que vous disposez de la stratégie ou des chemins d’indexation nécessaires pour la requête. |
| Mesures d’exécution des requêtes | Analysez les mesures de l’exécution des requêtes pour identifier les réécritures potentielles de formes de données et de requêtes.  |

### <a name="provisioned-throughput"></a>Débit approvisionné
Cosmos DB vous permet de créer des conteneurs de données, chacun avec un débit réservé qui est exprimé en RU (unité de requête) par seconde. La lecture d’un document de 1 Ko correspond à 1 RU, et chaque opération (y compris les requêtes) est normalisée en un nombre fixe d’unités de requête en fonction de sa complexité. Par exemple, si vous avez 1000 RU/s configurées pour votre conteneur, et que vous avez une requête, telle que `SELECT * FROM c WHERE c.city = 'Seattle'` qui consomme 5 RU, vous pouvez alors effectuer (1000 RU/s) / (5 RU/requête) = 200 requêtes/s de ce genre de requête par seconde. 

Si vous envoyez plus de 200 requêtes par seconde, le service démarre la limitation du débit de requêtes entrantes au-dessus de 200/s. Les kits SDK gèrent automatiquement cette situation en effectuant une interruption ou une nouvelle tentative, et vous pouvez donc remarquer une latence plus élevée pour ces requêtes. L’augmentation du débit approvisionné jusqu’à la valeur nécessaire améliore le débit et la latence des requêtes. 

Pour en savoir plus sur les unités de requête, consultez [Unités de requête](request-units.md).

### <a name="partitioning-and-partition-keys"></a>Partitionnement et clés de partition
Avec Azure Cosmos DB, les requêtes s’effectuent généralement dans l’ordre suivant, de la requête la plus rapide ou la plus efficace à la plus lente ou la moins efficace. 

* GET sur une clé de partition unique et une clé d’élément
* Requête avec une clause de filtre sur une clé de partition unique
* Requête sans clause de filtre d’égalité ou de plage sur une propriété
* Requête sans filtre

Les requêtes qui doivent consulter toutes les partitions nécessitent une latence plus élevée et peuvent consommer des RU plus élevées. Étant donné que chaque partition dispose de l’indexation automatique sur toutes les propriétés, la requête peut être servie efficacement à partir de l’index dans ce cas. Vous pouvez effectuer des requêtes qui englobent les partitions plus rapidement en utilisant les options de parallélisme.

Pour en savoir plus sur le partitionnement et les clés de partition, consultez [Partitionnement dans Azure Cosmos DB](partitioning-overview.md).

### <a name="sdk-and-query-options"></a>Options de requête et de kit SDK
Consultez [Conseils sur les performances](performance-tips.md) et [Tests de performances](performance-testing.md) pour savoir comment obtenir les meilleures performances côté client à partir d’Azure Cosmos DB. Cela implique d’utiliser les kits SDK les plus récents, de procéder à des configurations spécifiques aux plateformes, comme la fréquence de garbage collection ou le nombre de connexions par défaut, et d’utiliser des options de connectivité légère, comme Direct/TCP. 


#### <a name="max-item-count"></a>Nombre maximum d’éléments
Pour les requêtes, la valeur de `MaxItemCount` peut avoir un impact significatif sur la durée totale de la requête. Chaque aller-retour vers le serveur ne renverra pas plus que le nombre d’éléments dans `MaxItemCount` (valeur par défaut : 100 éléments). En définissant une valeur plus élevée (-1 constitue la valeur maximale et recommandée), vous réduisez la durée totale de la requête en limitant le nombre d’allers-retours entre le serveur et le client, en particulier pour les requêtes avec des jeux de résultats volumineux.

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxItemCount = -1, 
    }).AsDocumentQuery();
```

#### <a name="max-degree-of-parallelism"></a>Degré maximal de parallélisme
Pour les requêtes, réglez la valeur `MaxDegreeOfParallelism` afin d’identifier les meilleures configurations pour votre application, en particulier si vous effectuez des requêtes sur plusieurs partitions (sans filtre sur la valeur de clé de partition). `MaxDegreeOfParallelism` définit le nombre maximal de tâches parallèles, c’est-à-dire le nombre maximal de partitions à visiter en parallèle. 

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();
```

Supposons que
* D = nombre maximal de tâches parallèles par défaut (= nombre total de processeurs sur l’ordinateur client)
* P = nombre maximal de tâches parallèles spécifié par l’utilisateur
* N = nombre de partitions devant être visitées pour répondre à une requête

Voici comment les requêtes parallèles se comporteraient selon différentes valeurs de P.
* (P == 0) = > Mode série
* (P == 1) => Une tâche maximum
* (P > 1) => Nombre minimum de tâches parallèles (P, N) 
* (P < 1) => Nombre minimum de tâches parallèles (N, D)

Pour obtenir les notes de publication des kits SDK et des renseignements complémentaires sur les classes et les méthodes implémentées, consultez [Kits SDK SQL](sql-api-sdk-dotnet.md)

### <a name="network-latency"></a>Latence du réseau
Consultez [Distribution mondiale d’Azure Cosmos DB](tutorial-global-distribution-sql-api.md) pour savoir comment configurer la distribution mondiale et vous connecter à la région la plus proche. La latence du réseau a un impact significatif sur les performances des requêtes lorsque vous devez effectuer plusieurs allers-retours ou récupérer un jeu de résultats volumineux de la requête. 

La section sur les mesures d’exécution de requête explique comment récupérer le délai d’exécution de requêtes du serveur (`totalExecutionTimeInMs`), afin que vous puissiez faire la différence entre le temps passé à exécuter les requêtes et celui passé à gérer le trafic réseau.

### <a name="indexing-policy"></a>Stratégie d’indexation
Consultez [Configuration de la stratégie d’indexation](index-policy.md) pour l’indexation de chemins, de types et de modes, ainsi que leur impact sur l’exécution des requêtes. Par défaut, la stratégie d’indexation utilise une indexation de hachage pour les chaînes, ce qui est efficace pour les requêtes d’égalité, mais pas pour les requêtes de plage ou les requêtes de tri sélectif. Si vous avez besoin de requêtes de plage pour des chaînes, nous recommandons d’indiquer le type d’index de plage pour toutes les chaînes. 

Par défaut, Azure Cosmos DB applique une indexation automatique à toutes les données. Pour bénéficier de scénarios d’insertion hautes performances, envisagez d’exclure les chemins d’accès, car cela permet de réduire le coût des unités de requête pour chaque opération d’insertion. 

## <a name="query-execution-metrics"></a>Mesures d’exécution des requêtes
Vous pouvez obtenir des mesures détaillées sur l’exécution des requêtes en passant l’en-tête facultatif `x-ms-documentdb-populatequerymetrics` (`FeedOptions.PopulateQueryMetrics` dans le kit SDK .NET). La valeur retournée dans `x-ms-documentdb-query-metrics` contient les paires clé-valeur suivantes qui servent à la résolution plus poussée des problèmes d’exécution de requêtes. 

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PopulateQueryMetrics = true, 
    }).AsDocumentQuery();

FeedResponse<dynamic> result = await query.ExecuteNextAsync();

// Returns metrics by partition key range Id
IReadOnlyDictionary<string, QueryMetrics> metrics = result.QueryMetrics;

```

| Métrique | Unité | Description | 
| ------ | -----| ----------- |
| `totalExecutionTimeInMs` | millisecondes | Durée d’exécution de la requête | 
| `queryCompileTimeInMs` | millisecondes | Durée de compilation de la requête  | 
| `queryLogicalPlanBuildTimeInMs` | millisecondes | Durée de construction du plan de requête logique | 
| `queryPhysicalPlanBuildTimeInMs` | millisecondes | Durée de construction du plan de requête physique | 
| `queryOptimizationTimeInMs` | millisecondes | Temps consacré à l’optimisation de la requête | 
| `VMExecutionTimeInMs` | millisecondes | Temps passé dans l’exécution de la requête | 
| `indexLookupTimeInMs` | millisecondes | Temps passé dans la couche de l’index physique | 
| `documentLoadTimeInMs` | millisecondes | Temps de chargement de documents  | 
| `systemFunctionExecuteTimeInMs` | millisecondes | Temps total consacré à l’exécution de fonctions (intégrées) système, en millisecondes  | 
| `userFunctionExecuteTimeInMs` | millisecondes | Temps total consacré à l’exécution de fonctions définies par l’utilisateur, en millisecondes | 
| `retrievedDocumentCount` | count | Nombre total de documents récupérés  | 
| `retrievedDocumentSize` | octets | Taille totale des documents récupérés, en octets  | 
| `outputDocumentCount` | count | Nombre de documents sortants | 
| `writeOutputTimeInMs` | millisecondes | Durée moyenne d’exécution de la requête, en millisecondes | 
| `indexUtilizationRatio` | ratio (< = 1) | Ratio du nombre de documents mis en correspondance avec le filtre par rapport au nombre de documents chargés  | 

Les kits SDK client peuvent effectuer plusieurs opérations de requête en interne pour servir la requête dans chaque partition. Le client effectue plusieurs appels par partition si l’ensemble des résultats dépassent `x-ms-max-item-count`, si la requête excède le débit approvisionné pour la partition, si la charge utile de la requête atteint la taille maximale par page, ou si la requête atteint la limite de délai d’attente allouée du système. Chaque exécution de requête partielle retourne un `x-ms-documentdb-query-metrics` pour cette page. 

Voici quelques exemples de requête, et la façon d’interpréter certains mesures retournées à partir de l’exécution des requêtes : 

| Requête | Exemples de mesures | Description | 
| ------ | -----| ----------- |
| `SELECT TOP 100 * FROM c` | `"RetrievedDocumentCount": 101` | Le nombre de documents récupérés est 100 + 1 pour correspondre à la clause TOP. Le temps de la requête est essentiellement consacré à `WriteOutputTime` et `DocumentLoadTime` puisqu’il s’agit d’une analyse. | 
| `SELECT TOP 500 * FROM c` | `"RetrievedDocumentCount": 501` | RetrievedDocumentCount est désormais plus élevé (500 + 1 pour correspondre à la clause TOP). | 
| `SELECT * FROM c WHERE c.N = 55` | `"IndexLookupTime": "00:00:00.0009500"` | Environ 0,9 ms est passé dans IndexLookupTime pour une recherche de clé, car c’est une recherche d’index sur `/N/?`. | 
| `SELECT * FROM c WHERE c.N > 55` | `"IndexLookupTime": "00:00:00.0017700"` | Légèrement plus de temps (1,7 ms) est passé dans IndexLookupTime sur une analyse de plage, car c’est une recherche d’index sur `/N/?`. | 
| `SELECT TOP 500 c.N FROM c` | `"IndexLookupTime": "00:00:00.0017700"` | Même temps passé sur `DocumentLoadTime` que sur les requêtes précédentes, mais `WriteOutputTime` est inférieur, car nous ne projetons qu’une seule propriété. | 
| `SELECT TOP 500 udf.toPercent(c.N) FROM c` | `"UserDefinedFunctionExecutionTime": "00:00:00.2136500"` | Environ un temps de 213 ms est consacré à `UserDefinedFunctionExecutionTime` exécutant l’UDF sur chaque valeur de `c.N`. |
| `SELECT TOP 500 c.Name FROM c WHERE STARTSWITH(c.Name, 'Den')` | `"IndexLookupTime": "00:00:00.0006400", "SystemFunctionExecutionTime": "00:00:00.0074100"` | Environ 0,6 ms est consacré à `IndexLookupTime` sur `/Name/?`. La majeure partie de la durée d’exécution de la requête (~ 7 ms) dans `SystemFunctionExecutionTime`. |
| `SELECT TOP 500 c.Name FROM c WHERE STARTSWITH(LOWER(c.Name), 'den')` | `"IndexLookupTime": "00:00:00", "RetrievedDocumentCount": 2491,  "OutputDocumentCount": 500` | La requête est exécutée en tant qu’analyse, car elle utilise `LOWER`, et 500 documents sur les 2491 récupérés sont retournés. |


## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur les opérateurs et les mots clés de requête SQL pris en charge, consultez [Requête SQL](sql-query-getting-started.md). 
* Pour obtenir plus d’informations sur les unités de requête, consultez [Unités de requête](request-units.md).
* Pour en savoir plus sur la stratégie d’indexation, consultez [Stratégie d’indexation](index-policy.md)