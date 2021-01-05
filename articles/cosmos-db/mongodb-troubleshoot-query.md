---
title: Résoudre les problèmes de requête lors de l’utilisation de l’API Azure Cosmos DB pour MongoDB
description: Découvrez comment identifier, diagnostiquer et dépanner les API d’Azure Cosmos DB pour les problèmes de requête MongoDB.
author: timsander1
ms.service: cosmos-db
ms.topic: troubleshooting
ms.subservice: cosmosdb-mongo
ms.date: 10/12/2020
ms.author: tisande
ms.reviewer: sngun
ms.openlocfilehash: 88ef081c75a64b5cb7517ba6994834b3a64a0e6f
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340887"
---
# <a name="troubleshoot-query-issues-when-using-the-azure-cosmos-db-api-for-mongodb"></a>Résoudre les problèmes de requête lors de l’utilisation de l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Cet article décrit une approche générale recommandée pour le dépannage des requêtes dans Azure Cosmos DB. Bien que vous ne devriez pas considérer les étapes décrites dans cet article comme une protection complète contre les problèmes de requête potentiels, nous avons inclus ici les conseils sur les performances les plus courants. Vous devez utiliser cet article comme point de départ pour le dépannage des requêtes lentes ou coûteuses dans l’API de Azure Cosmos DB pour MongoDB. Si vous utilisez l’API Azure Cosmos DB Core (SQL), consultez l’article correspondant dans le Guide de dépannage des requêtes d’API [SQL](troubleshoot-query-performance.md).

Les optimisations de requête dans Azure Cosmos DB sont classées de manière générale comme suit :

- celles qui réduisent les frais en unités de requête (RU) de la requête
- Optimisations qui réduisent simplement la latence

Si vous réduisez les frais en RU d’une requête, vous diminuez généralement aussi la latence.

Cet article fournit des exemples que vous pouvez recréer à l’aide du [jeu de données nutrition](https://github.com/CosmosDB/labs/blob/master/dotnet/setup/NutritionData.json).

> [!NOTE] 
> Cet article suppose que vous utilisez la version 3,6 de Azure Cosmos DB API « s » pour MongoDB. Certaines requêtes qui fonctionnent mal dans la version 3,2 présentent des améliorations significatives dans la version 3,6. Effectuez une mise à niveau vers la version 3,6 en soumettant une [demande de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="use-explain-command-to-get-metrics"></a>Utilisez $explain commande pour récupérer les mesures

Lorsque vous optimisez une requête dans Azure Cosmos DB, la première étape consiste toujours à [obtenir les frais de facturation RU](find-request-unit-charge-mongodb.md) pour votre requête. En règle générale, vous devez explorer les moyens de réduire le coût du RU pour les requêtes dont les frais sont supérieurs à 50. 

Outre l’obtention des frais de l’RU, vous devez utiliser la commande `$explain` pour obtenir les mesures d’utilisation des requêtes et des index. Voici un exemple qui exécute une requête et utilise la commande `$explain` pour afficher les mesures d’utilisation des requêtes et des index :

**commande $explain :**

```
db.coll.find({foodGroup: "Baby Foods"}).explain({"executionStatistics": true })
```

**Output:**

```json
{
    "stages" : [ 
        {
            "stage" : "$query",
            "timeInclusiveMS" : 905.2888,
            "timeExclusiveMS" : 905.2888,
            "in" : 362,
            "out" : 362,
            "details" : {
                "database" : "db-test",
                "collection" : "collection-test",
                "query" : {
                    "foodGroup" : {
                        "$eq" : "Baby Foods"
                    }
                },
                "pathsIndexed" : [],
                "pathsNotIndexed" : [ 
                    "foodGroup"
                ],
                "shardInformation" : [ 
                    {
                        "activityId" : "e68e6bdd-5e89-4ec5-b053-3dbbc2428140",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 788.5867,
                        "preemptions" : 1,
                        "outputDocumentCount" : 362,
                        "retrievedDocumentCount" : 8618
                    }
                ],
                "queryMetrics" : {
                    "retrievedDocumentCount" : 8618,
                    "retrievedDocumentSizeBytes" : 104963042,
                    "outputDocumentCount" : 362,
                    "outputDocumentSizeBytes" : 2553535,
                    "indexHitRatio" : 0.0016802042237178,
                    "totalQueryExecutionTimeMS" : 777.72,
                    "queryPreparationTimes" : {
                        "queryCompilationTimeMS" : 0.19,
                        "logicalPlanBuildTimeMS" : 0.14,
                        "physicalPlanBuildTimeMS" : 0.09,
                        "queryOptimizationTimeMS" : 0.03
                    },
                    "indexLookupTimeMS" : 0,
                    "documentLoadTimeMS" : 687.22,
                    "vmExecutionTimeMS" : 774.09,
                    "runtimeExecutionTimes" : {
                        "queryEngineExecutionTimeMS" : 37.45,
                        "systemFunctionExecutionTimeMS" : 10.82,
                        "userDefinedFunctionExecutionTimeMS" : 0
                    },
                    "documentWriteTimeMS" : 49.42
                }
            }
        }
    ],
    "estimatedDelayFromRateLimitingInMilliseconds" : 0.0,
    "continuation" : {
        "hasMore" : false
    },
    "ok" : 1.0
}
```

Le résultat de la commande `$explain` est long et contient des informations détaillées sur l’exécution des requêtes. En général, toutefois, il existe quelques sections où vous devez vous concentrer sur l’optimisation des performances des requêtes :

| Métrique | Description | 
| ------ | ----------- |
| `timeInclusiveMS` | Latence des requêtes du backend |
| `pathsIndexed` | Affiche les index utilisés par la requête | 
| `pathsNotIndexed` | Affiche les index que la requête aurait pu utiliser, le cas échéant | 
| `shardInformation` | Résumé des performances des requêtes pour une [partition physique](./partitioning-overview.md#physical-partitions)particulière | 
| `retrievedDocumentCount` | Nombre de documents chargés par le moteur de requête | 
| `outputDocumentCount` | Nombre de documents renvoyés dans les résultats de la requête | 
| `estimatedDelayFromRateLimitingInMilliseconds` | Latence de requête supplémentaire estimée due à la limitation du débit | 

Après avoir obtenu les métriques de requête, comparez le `retrievedDocumentCount` au `outputDocumentCount` pour votre requête. Utilisez cette comparaison pour identifier les sections pertinentes à vérifier dans cet article. `retrievedDocumentCount` est le nombre de documents que le moteur de requête doit charger. Le `outputDocumentCount` correspond au nombre de documents qui ont été nécessaires pour les résultats de la requête. Si le `retrievedDocumentCount`  est beaucoup plus élevé que le `outputDocumentCount`, il y a au moins une partie de votre requête qui n’a pas pu utiliser un index et est nécessaire pour effectuer une analyse.

Reportez-vous aux sections suivantes pour comprendre les optimisations de requête pertinentes pour votre scénario.

### <a name="querys-ru-charge-is-too-high"></a>Les frais en RU de la requête sont trop élevés

#### <a name="retrieved-document-count-is-significantly-higher-than-output-document-count"></a>Le nombre de documents récupérés est beaucoup plus élevé que le nombre de documents de sortie

- [Incluez les index nécessaires.](#include-necessary-indexes)

- [Identifiez les opérations d’agrégation qui utilisent l’index.](#understand-which-aggregation-operations-use-the-index)

#### <a name="retrieved-document-count-is-approximately-equal-to-output-document-count"></a>Le nombre de documents récupérés est à peu près égal au nombre de documents de sortie

- [Réduisez les requêtes entre les partitions.](#minimize-cross-partition-queries)

### <a name="querys-ru-charge-is-acceptable-but-latency-is-still-too-high"></a>Les frais en RU de la requête sont acceptables, mais la latence est encore trop élevée

- [Améliorez la proximité.](#improve-proximity)

- [Augmentez le débit provisionné.](#increase-provisioned-throughput)

## <a name="queries-where-retrieved-document-count-exceeds-output-document-count"></a>Requêtes où le nombre de documents récupérés dépasse le nombre de documents de sortie

 Le `retrievedDocumentCount` correspond au nombre de documents que le moteur de requête a dû charger. Le `outputDocumentCount` correspond au nombre de documents retournés par la requête. Si le `retrievedDocumentCount` est beaucoup plus élevé que le `outputDocumentCount`, il y a au moins une partie de votre requête qui n’a pas pu utiliser d’index et qui a dû effectuer une analyse.

Voici un exemple de requête d’analyse qui n’a pas été entièrement servie par l’index :

**commande $explain :**

```
db.coll.find(
  {
    $and : [
            { "foodGroup" : "Cereal Grains and Pasta"}, 
            { "description" : "Oat bran, cooked"}
        ]
  }
).explain({"executionStatistics": true })
```

**Output:**

```json
{
    "stages" : [ 
        {
            "stage" : "$query",
            "timeInclusiveMS" : 436.5716,
            "timeExclusiveMS" : 436.5716,
            "in" : 1,
            "out" : 1,
            "details" : {
                "database" : "db-test",
                "collection" : "indexing-test",
                "query" : {
                    "$and" : [ 
                        {
                            "foodGroup" : {
                                "$eq" : "Cereal Grains and Pasta"
                            }
                        }, 
                        {
                            "description" : {
                                "$eq" : "Oat bran, cooked"
                            }
                        }
                    ]
                },
                "pathsIndexed" : [],
                "pathsNotIndexed" : [ 
                    "foodGroup", 
                    "description"
                ],
                "shardInformation" : [ 
                    {
                        "activityId" : "13a5977e-a10a-4329-b68e-87e4f0081cac",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 435.4867,
                        "preemptions" : 1,
                        "outputDocumentCount" : 1,
                        "retrievedDocumentCount" : 8618
                    }
                ],
                "queryMetrics" : {
                    "retrievedDocumentCount" : 8618,
                    "retrievedDocumentSizeBytes" : 104963042,
                    "outputDocumentCount" : 1,
                    "outputDocumentSizeBytes" : 6064,
                    "indexHitRatio" : 0.0,
                    "totalQueryExecutionTimeMS" : 433.64,
                    "queryPreparationTimes" : {
                        "queryCompilationTimeMS" : 0.12,
                        "logicalPlanBuildTimeMS" : 0.09,
                        "physicalPlanBuildTimeMS" : 0.1,
                        "queryOptimizationTimeMS" : 0.02
                    },
                    "indexLookupTimeMS" : 0,
                    "documentLoadTimeMS" : 387.44,
                    "vmExecutionTimeMS" : 432.93,
                    "runtimeExecutionTimes" : {
                        "queryEngineExecutionTimeMS" : 45.36,
                        "systemFunctionExecutionTimeMS" : 16.86,
                        "userDefinedFunctionExecutionTimeMS" : 0
                    },
                    "documentWriteTimeMS" : 0.13
                }
            }
        }
    ],
    "estimatedDelayFromRateLimitingInMilliseconds" : 0.0,
    "continuation" : {
        "hasMore" : false
    },
    "ok" : 1.0
}
```

La `retrievedDocumentCount` (8618) est beaucoup plus élevée que la `outputDocumentCount` (1), ce qui implique que cette requête nécessite une analyse de document. 

### <a name="include-necessary-indexes"></a>Incluez les index nécessaires

Vous devez vérifier le tableau de `pathsNotIndexed` et ajouter ces index. Dans cet exemple, les chemins d’accès `foodGroup` et `description` doivent être indexés.

```json
"pathsNotIndexed" : [ 
                    "foodGroup", 
                    "description"
                ]
```

L’indexation des meilleures pratiques dans l’API d’Azure Cosmos DB pour MongoDB est différente de MongoDB. Dans l’API d’Azure Cosmos DB pour MongoDB, les index composés sont utilisés uniquement dans les requêtes qui doivent trier efficacement selon plusieurs propriétés. Si vous avez des requêtes avec des filtres sur plusieurs propriétés, vous devez créer des index de champ unique pour chacune de ces propriétés. Les prédicats de requête peuvent utiliser plusieurs index de champ unique.

[Les index génériques](mongodb-indexing.md#wildcard-indexes) peuvent simplifier l’indexation. Contrairement à MongoDB, les index génériques peuvent prendre en charge plusieurs champs dans les prédicats de requête. Il n’y aura pas de différence dans les performances de requête si vous utilisez un seul index générique au lieu de créer un index distinct pour chaque propriété. L’ajout d’un index générique pour toutes les propriétés est le moyen le plus simple d’optimiser toutes vos requêtes.

Vous pouvez ajouter de nouveaux index à tout moment, sans effet sur la disponibilité en écriture ou en lecture. Vous pouvez [suivre la progression de la transformation d’index](./how-to-manage-indexing-policy.md#dotnet-sdk).

### <a name="understand-which-aggregation-operations-use-the-index"></a>Comprendre les opérations d’agrégation qui utilisent l’index

Dans la plupart des cas, les opérations d’agrégation dans l’API d’Azure Cosmos DB pour MongoDB utilisent partiellement des index. En règle générale, le moteur de requête applique d’abord les filtres d’égalité et de plage et utilise des index. Après l’application de ces filtres, le moteur d’interrogation peut évaluer des filtres supplémentaires et recourir au chargement des documents restants pour calculer l’agrégat, le cas échéant. 

Voici un exemple :

```
db.coll.aggregate( [
   { $match: { foodGroup: 'Fruits and Fruit Juices' } },
   {
     $group: {
        _id: "$foodGroup",
        total: { $max: "$version" }
     }
   }
] )
```

Dans ce cas, les index peuvent optimiser l’étape `$match` . L’ajout d’un index pour `foodGroup` améliore considérablement les performances des requêtes. Comme dans MongoDB, vous devez rapidement placer `$match` dans le pipeline d’agrégation pour maximiser l’utilisation des index.

Dans l’API d’Azure Cosmos DB pour MongoDB, les index ne sont pas utilisés pour l’agrégation réelle, qui dans ce cas est `$max`. L’ajout d’un index sur `version` n’améliore pas les performances des requêtes.

## <a name="queries-where-retrieved-document-count-is-equal-to-output-document-count"></a>Requêtes où le nombre de documents récupérés est égal au nombre de documents de sortie

Si le `retrievedDocumentCount` est à peu près égal au `outputDocumentCount`, le moteur de requête n’a pas eu à analyser de nombreux documents inutiles.

### <a name="minimize-cross-partition-queries"></a>Réduire les requêtes entre les partitions

Azure Cosmos DB utilise le [partitionnement](partitioning-overview.md) pour mettre à l’échelle des conteneurs en fonction de l’augmentation des besoins en unités de requête et en stockage des données. Chaque partition physique a un index distinct et indépendant. Si votre requête a un filtre d’égalité qui correspond à la clé de partition de votre conteneur, vous devrez uniquement vérifier l’index de la partition pertinente. Cette optimisation réduit le nombre total de RU requis par la requête. [En savoir plus sur les différences entre les requêtes dans la partition et les requêtes sur plusieurs partitions](how-to-query-container.md).

Si vous disposez d’un grand nombre d’unités de requête provisionnées (plus de 30 000) ou d’une grande quantité de données stockées (plus de 100 Go environ), vous disposez probablement d’un conteneur suffisamment grand pour constater une réduction significative des frais en RU liés aux requêtes. 

Vous pouvez consulter le tableau `shardInformation` pour comprendre les mesures de la requête pour chaque partition physique individuelle. Le nombre de valeurs de `shardKeyRangeId` uniques est le nombre de partitions physiques dans lesquelles la requête devait être exécutée. Dans cet exemple, la requête a été exécutée sur quatre partitions physiques. Il est important de comprendre que l’exécution est totalement indépendante de l’utilisation de l’index. En d’autres termes, les requêtes entre les partitions peuvent toujours utiliser des index.

```json
  "shardInformation" : [ 
                    {
                        "activityId" : "42f670a8-a201-4c58-8023-363ac18d9e18",
                        "shardKeyRangeId" : "5",
                        "durationMS" : 24.3859,
                        "preemptions" : 1,
                        "outputDocumentCount" : 463,
                        "retrievedDocumentCount" : 463
                    }, 
                    {
                        "activityId" : "a8bf762a-37b9-4c07-8ed4-ae49961373c0",
                        "shardKeyRangeId" : "2",
                        "durationMS" : 35.8328,
                        "preemptions" : 1,
                        "outputDocumentCount" : 905,
                        "retrievedDocumentCount" : 905
                    }, 
                    {
                        "activityId" : "3754e36b-4258-49a6-8d4d-010555628395",
                        "shardKeyRangeId" : "1",
                        "durationMS" : 67.3969,
                        "preemptions" : 1,
                        "outputDocumentCount" : 1479,
                        "retrievedDocumentCount" : 1479
                    }, 
                    {
                        "activityId" : "a69a44ee-db97-4fe9-b489-3791f3d52878",
                        "shardKeyRangeId" : "0",
                        "durationMS" : 185.1523,
                        "preemptions" : 1,
                        "outputDocumentCount" : 867,
                        "retrievedDocumentCount" : 867
                    }
                ]
```

## <a name="optimizations-that-reduce-query-latency"></a>Optimisations qui réduisent la latence des requêtes

Dans de nombreux cas, les frais en RU peuvent être acceptables, même si la latence des requêtes est encore trop élevée. Les sections suivantes présentent différents conseils pour réduire la latence des requêtes. Si vous exécutez la même requête plusieurs fois sur le même jeu de données, elle aura généralement les mêmes frais de RU à chaque fois. Mais la latence des requêtes peut varier d’une exécution de requête à une autre.

### <a name="improve-proximity"></a>Améliorer la proximité

Les requêtes exécutées à partir d’une région différente de celle du compte Azure Cosmos DB auront une latence plus élevée que si elles étaient exécutées dans la même région. Par exemple, si vous exécutez du code sur votre ordinateur de bureau, vous devez vous attendre à une latence supérieure de quelques dizaines ou centaines de millisecondes (ou plus) à la même requête provenant d’une machine virtuelle située dans la même région Azure qu’Azure Cosmos DB. Il est facile de [distribuer les données globalement dans Azure Cosmos DB](tutorial-global-distribution-mongodb.md) pour vous assurer de pouvoir rapprocher vos données de votre application.

### <a name="increase-provisioned-throughput"></a>Augmenter le débit provisionné

Dans Azure Cosmos DB, votre débit provisionné est mesuré en unités de requête (RU). Supposez que vous avez une requête qui consomme 5 RU de débit. Par exemple, si vous provisionnez 1 000 RU, vous pouvez exécuter cette requête 200 fois par seconde. Si vous avez essayé d’exécuter la requête alors que le débit disponible est insuffisant, Azure Cosmos DB limite le nombre des demandes. L’API de Azure Cosmos DB pour MongoDB réessaiera automatiquement cette requête après avoir attendu un peu de temps. Les requêtes limitées prennent plus de temps ; par conséquent, l’augmentation du débit provisionné peut améliorer la latence des requêtes.

La valeur `estimatedDelayFromRateLimitingInMilliseconds` donne une idée des avantages de la latence potentielle si vous augmentez le débit.

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes de performances des requêtes (API SQL)](troubleshoot-query-performance.md)
* [Gérer l’indexation dans l’API pour MongoDB d’Azure Cosmos DB](mongodb-indexing.md)