---
title: Questions fréquentes sur l’API Gremlin dans Azure Cosmos DB
description: Obtenez des réponses aux questions fréquentes sur l’API Gremlin dans Azure Cosmos DB
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: conceptual
ms.date: 04/28/2020
ms.author: sngun
ms.openlocfilehash: 0374886482b43eefe6df5e107fd4da371633099d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359776"
---
# <a name="frequently-asked-questions-about-the-gremlin-api-in-azure-cosmos-db"></a>Questions fréquentes sur l’API Gremlin dans Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article apporte des réponses à certaines questions fréquentes sur l’API Gremlin dans Azure Cosmos DB.

## <a name="how-to-evaluate-the-efficiency-of-gremlin-queries"></a>Comment évaluer l’efficacité des requêtes Gremlin

L’étape de l’aperçu **executionProfile()** peut être utilisée pour fournir une analyse du plan d'exécution de la requête. Cette étape doit être ajoutée à la fin de n’importe quelle requête Gremlin comme illustré dans l’exemple suivant :

**Exemple de requête**

```
g.V('mary').out('knows').executionProfile()
```

**Exemple de sortie**

```json
[
  {
    "gremlin": "g.V('mary').out('knows').executionProfile()",
    "totalTime": 8,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 3,
        "annotations": {
          "percentTime": 37.5
        },
        "counts": {
          "resultCount": 1
        }
      },
      {
        "name": "GetEdges",
        "time": 5,
        "annotations": {
          "percentTime": 62.5
        },
        "counts": {
          "resultCount": 0
        },
        "storeOps": [
          {
            "count": 0,
            "size": 0,
            "time": 0.6
          }
        ]
      },
      {
        "name": "GetNeighborVertices",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      }
    ]
  }
]
```

La sortie du profil ci-dessus indique combien de temps est consacré à l’obtention des objets de vertex, des objets d’arête, ainsi que la taille du jeu de données de travail. Cela concerne les mesures de coût standard pour les requêtes Azure Cosmos DB.

## <a name="other-frequently-asked-questions"></a>Autres questions fréquentes

### <a name="how-are-rus-charged-when-running-queries-on-a-graph-database"></a>Comment les RU/s sont-elles facturées lors de l’exécution des requêtes sur une base de données de graphique ?

Tous les objets, sommets et arêtes des graphes sont affichés sous forme de documents JSON dans le back-end. Dans la mesure où une seule requête Gremlin peut modifier un ou plusieurs objets de graphe à la fois, le coût associé est directement lié aux objets ou arêtes traités par la requête. Azure Cosmos DB utilise le même processus pour toutes les autres API. Pour plus d’informations, voir [Unités de requête dans Azure Cosmos DB](request-units.md).

Les frais de RU sont basés sur le jeu de données de travail de la traversée, et non sur le jeu de résultats. Par exemple, si une requête vise à obtenir un seul vertex en résultat mais doit traverser plusieurs autres objets sur le chemin, le coût sera basé sur tous les objets de graphe utilisés pour calculer cet unique vertex de résultat.

### <a name="whats-the-maximum-scale-that-a-graph-database-can-have-in-azure-cosmos-db-gremlin-api"></a>Quelle est la mise à l’échelle maximale qu’une base de données de graphe peut présenter dans l’API Gremlin Azure Cosmos DB ?

Azure Cosmos DB utilise le [partitionnement horizontal](partitioning-overview.md) pour répondre automatiquement aux exigences d’augmentation de stockage et de débit. Le débit et la capacité de stockage maximum d’une charge de travail sont déterminés par le nombre de partitions associées à un conteneur donné. Toutefois, un conteneur d’API Gremlin comporte un ensemble spécifique d’instructions pour garantir une expérience de performances à l’échelle correcte. Pour plus d’informations sur le partitionnement de données et les meilleurs pratiques, voir l’article sur le [partitionnement dans Azure Cosmos DB](partitioning-overview.md).

### <a name="for-cnet-development-should-i-use-the-microsoftazuregraphs-package-or-gremlinnet"></a>Pour le développement en C#/.NET, dois-je utiliser le package Microsoft.Azure.Graphs ou Gremlin.NET ?

L’API Gremlin d’Azure Cosmos DB se sert des pilotes open source comme connecteurs principaux pour le service. Nous recommandons alors d’utiliser les [pilotes qui sont pris en charge par Apache Tinkerpop](https://tinkerpop.apache.org/).

### <a name="how-can-i-protect-against-injection-attacks-using-gremlin-drivers"></a>Comment puis-je me protéger contre les attaques par injection à l’aide de pilotes Gremlin ?

La majorité des pilotes Apache Tinkerpop Gremlin natifs offrent la possibilité de fournir un dictionnaire de paramètres pour l’exécution des requêtes. Voici un exemple montrant comment le faire sous [Gremlin.Net](https://tinkerpop.apache.org/docs/3.2.7/reference/#gremlin-DotNet) et [Gremlin-Javascript](https://github.com/Azure-Samples/azure-cosmos-db-graph-nodejs-getting-started/blob/main/app.js).

### <a name="why-am-i-getting-the-gremlin-query-compilation-error-unable-to-find-any-method-error"></a>Pourquoi est-ce que je reçois le message d’erreur « Erreur de compilation de la requête Gremlin : Impossible de trouver une méthode » ?

L’API Gremlin d’Azure Cosmos DB implémente un sous-ensemble des fonctionnalités définies dans la surface d’exposition de Gremlin. Pour voir les étapes prises en charge ou pour plus d’informations, consultez l’article [Prise en charge de Gremlin](gremlin-support.md).

La meilleure solution de contournement consiste à récrire les étapes Gremlin nécessaires avec la fonctionnalité prise en charge puisque toutes les étapes Gremlin essentielles sont prises en charge par Azure Cosmos DB.

### <a name="why-am-i-getting-the-websocketexception-the-server-returned-status-code-200-when-status-code-101-was-expected-error"></a>Pourquoi est-ce que je reçois le message d’erreur « WebSocketException : Le serveur a retourné le code d’état "200" alors que le code d’état attendu était "101" » ?

Cette erreur survient probablement lorsqu’un point de terminaison incorrect est utilisé. Le point de terminaison qui génère cette erreur a le modèle suivant :

`https:// YOUR_DATABASE_ACCOUNT.documents.azure.com:443/`

C’est le point de terminaison de documents pour votre base de données de graphique.  Le bon point de terminaison à utiliser est le point de terminaison Gremlin qui a le format suivant :

`https://YOUR_DATABASE_ACCOUNT.gremlin.cosmosdb.azure.com:443/`

### <a name="why-am-i-getting-the-requestrateistoolarge-error"></a>Pourquoi est-ce que je reçois le message d’erreur « RequestRateIsTooLarge » ?

Cette erreur signifie que les unités de requête par seconde allouées ne sont pas suffisantes pour traiter la requête. Cette erreur se produit généralement lorsque vous exécutez une requête qui obtient tous les vertex :

```
// Query example:
g.V()
```

Cette requête va tenter de récupérer tous les vertex du graphique. Par conséquent, le coût de cette requête sera au moins égal au nombre de vertex en termes d’unités de requête. Le paramètre RU/s doit être ajusté pour répondre à cette requête.

### <a name="why-do-my-gremlin-driver-connections-get-dropped-eventually"></a>Pourquoi mes connexions de pilote Gremlin finissent par être supprimées ?

Une connexion Gremlin est établie via une connexion WebSocket. Bien que les connexions WebSocket ne disposent pas d’une durée de vie spécifique, l’API Gremlin Azure Cosmos DB terminera les connexions inactives après 30 minutes d’inactivité.

### <a name="why-cant-i-use-fluent-api-calls-in-the-native-gremlin-drivers"></a>Pourquoi ne puis-je pas utiliser d’appels d’API Fluent dans les pilotes Gremlin natifs ?

Les appels d’API Fluent ne sont pas encore pris en charge par l’API Gremlin d’Azure Cosmos DB. Les appels d’API Fluent nécessitent une fonctionnalité de mise en forme interne, appelée la « prise en charge bytecode », qui n’est actuellement pas prise en charge par l’API Gremlin d’Azure Cosmos DB. Pour la même raison, le dernier pilote Gremlin-JavaScript n’est pas pris en charge actuellement.

## <a name="next-steps"></a>Étapes suivantes

* [Prise en charge du protocole de communication Gremlin Azure Cosmos DB](gremlin-support.md)
* Créer, interroger et parcourir une base de données de graphe Azure Cosmos DB à l’aide de la [console Gremlin](create-graph-gremlin-console.md)
