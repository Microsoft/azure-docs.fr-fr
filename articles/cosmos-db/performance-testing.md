---
title: Test des performances et de la mise à l’échelle avec Azure Cosmos DB
description: Découvrez comment effectuer un test des performances et de la mise à l’échelle avec Azure Cosmos DB. Vous pouvez ensuite évaluer les fonctionnalités d’Azure Cosmos DB dans des scénarios d’applications hautes performances.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/23/2019
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: cb0dd8eb879c5511da3c79a3dee6a715fb1275e4
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358641"
---
# <a name="performance-and-scale-testing-with-azure-cosmos-db"></a>Test des performances et de la mise à l’échelle avec Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le test des performances et de la mise à l’échelle est une étape essentielle du développement d’applications. Pour de nombreuses applications, la couche Base de données a un impact significatif sur les performances globales et la scalabilité. Il s’agit donc d’une composante essentielle des tests de performances. [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) a été spécialement conçu pour offrir une mise à l’échelle élastique et des performances prévisibles. Ces capacités en font une solution de choix pour les applications qui ont besoin d’une couche Base de données hautement performante. 

Cet article fournit des informations de référence pour les développeurs qui implémentent des suites de test de performances pour leurs charges de travail Azure Cosmos DB. Il permet aussi d’évaluer Azure Cosmos DB pour des scénarios d’applications à hautes performances. Il se concentre principalement sur les tests de performances isolés de la base de données, mais inclut également les meilleures pratiques relatives aux applications de production.

Après avoir lu cet article, vous serez en mesure de répondre aux questions suivantes : 

* Où trouver un exemple d’application cliente .NET pour tester les performances d’Azure Cosmos DB ? 
* Comment faire pour atteindre de hauts niveaux de débit avec Azure Cosmos DB à partir de mon application cliente ?

Pour commencer avec le code, téléchargez le projet à partir de l’[exemple de test des performances Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/documentdb-benchmark). 

> [!NOTE]
> L’objectif de cette application est de montrer comment obtenir les meilleures performances d’Azure Cosmos DB avec un petit nombre d’ordinateurs clients. L’objectif de l’exemple n’est pas d’atteindre la capacité de débit de pointe d’Azure Cosmos DB (qui peut évoluer sans aucune limite).

Si vous recherchez des options de configuration côté client pour améliorer les performances d’Azure Cosmos DB, consultez page [Conseils sur les performances d’Azure Cosmos DB](performance-tips.md).

## <a name="run-the-performance-testing-application"></a>Exécution de l’application de test des performances
Le moyen le plus rapide de commencer est de compiler et exécuter l’exemple .NET, comme décrit dans la procédure ci-dessous. Vous pouvez aussi examiner le code source et implémenter des configurations similaires dans vos propres applications clientes.

**Étape 1 :** téléchargez le projet à partir de l’[exemple de test de performances Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/documentdb-benchmark) ou dupliquez (fork) le dépôt GitHub.

**Étape 2 :** modifiez les paramètres pour EndpointUrl, AuthorizationKey, CollectionThroughput et DocumentTemplate (facultatif) dans App.config.

> [!NOTE]
> Avant d’approvisionner des collections avec un débit élevé, reportez-vous à la [page des tarifs](https://azure.microsoft.com/pricing/details/cosmos-db/) pour estimer les coûts par collection. Azure Cosmos DB facture le stockage et le débit de façon indépendante sur une base horaire. Vous pouvez faire des économies en soustrayant ou diminuant le débit de vos conteneurs Azure Cosmos après les tests.
> 
> 

**Étape 3 :** compilez et exécutez l’application console à partir de la ligne de commande. Un résultat similaire à ce qui suit s’affiche normalement :

```bash
C:\Users\cosmosdb\Desktop\Benchmark>DocumentDBBenchmark.exe
Summary:
---------------------------------------------------------------------
Endpoint: https://arramacquerymetrics.documents.azure.com:443/
Collection : db.data at 100000 request units per second
Document Template*: Player.json
Degree of parallelism*: -1
---------------------------------------------------------------------
DocumentDBBenchmark starting...
Found collection data with 100000 RU/s
Starting Inserts with 100 tasks
Inserted 4503 docs @ 4491 writes/s, 47070 RU/s (122B max monthly 1KB reads)
Inserted 17910 docs @ 8862 writes/s, 92878 RU/s (241B max monthly 1KB reads)
Inserted 32339 docs @ 10531 writes/s, 110366 RU/s (286B max monthly 1KB reads)
Inserted 47848 docs @ 11675 writes/s, 122357 RU/s (317B max monthly 1KB reads)
Inserted 58857 docs @ 11545 writes/s, 120992 RU/s (314B max monthly 1KB reads)
Inserted 69547 docs @ 11378 writes/s, 119237 RU/s (309B max monthly 1KB reads)
Inserted 80687 docs @ 11345 writes/s, 118896 RU/s (308B max monthly 1KB reads)
Inserted 91455 docs @ 11272 writes/s, 118131 RU/s (306B max monthly 1KB reads)
Inserted 102129 docs @ 11208 writes/s, 117461 RU/s (304B max monthly 1KB reads)
Inserted 112444 docs @ 11120 writes/s, 116538 RU/s (302B max monthly 1KB reads)
Inserted 122927 docs @ 11063 writes/s, 115936 RU/s (301B max monthly 1KB reads)
Inserted 133157 docs @ 10993 writes/s, 115208 RU/s (299B max monthly 1KB reads)
Inserted 144078 docs @ 10988 writes/s, 115159 RU/s (298B max monthly 1KB reads)
Inserted 155415 docs @ 11013 writes/s, 115415 RU/s (299B max monthly 1KB reads)
Inserted 166126 docs @ 10992 writes/s, 115198 RU/s (299B max monthly 1KB reads)
Inserted 173051 docs @ 10739 writes/s, 112544 RU/s (292B max monthly 1KB reads)
Inserted 180169 docs @ 10527 writes/s, 110324 RU/s (286B max monthly 1KB reads)
Inserted 192469 docs @ 10616 writes/s, 111256 RU/s (288B max monthly 1KB reads)
Inserted 199107 docs @ 10406 writes/s, 109054 RU/s (283B max monthly 1KB reads)
Inserted 200000 docs @ 9930 writes/s, 104065 RU/s (270B max monthly 1KB reads)

Summary:
---------------------------------------------------------------------
Inserted 200000 docs @ 9928 writes/s, 104063 RU/s (270B max monthly 1KB reads)
---------------------------------------------------------------------
DocumentDBBenchmark completed successfully.
Press any key to exit...
```

**Étape 4 (si nécessaire) :** le débit signalé (RU/s) à partir de l’outil doit être identique ou supérieur au débit provisionné de la collection ou d’un ensemble de collections. Dans le cas contraire, le fait d’augmenter la valeur de DegreeOfParallelism par petits incréments peut vous aider à atteindre la limite. Si le débit de votre application cliente se stabilise, démarrez plusieurs instances de l’application sur d’autres ordinateurs clients. Si vous avez besoin d’aide pour cette étape, remplissez un ticket de support à partir du [portail Azure](https://portal.azure.com).

Une fois l’application exécutée, vous pouvez tester des [stratégies d’indexation](index-policy.md) et des [niveaux de cohérence](consistency-levels.md) différents pour comprendre leur impact sur le débit et la latence. Vous pouvez également examiner le code source et implémenter des configurations similaires dans vos propres suites de test ou applications de production.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons vu comment effectuer des tests de performances et de mise à l’échelle avec Azure Cosmos DB en utilisant une application console .NET. Pour plus d’informations, consultez les articles suivants :

* [Exemple de test des performances Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/documentdb-benchmark)
* [Options de configuration côté client permettant d’améliorer les performances d’Azure Cosmos DB](performance-tips.md)
* [Guide de partitionnement et de mise à l’échelle dans Azure Cosmos DB](partitioning-overview.md)


