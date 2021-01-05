---
title: Utiliser le profil d’exécution pour évaluer des requêtes dans l’API Gremlin Azure Cosmos DB
description: Découvrez comment dépanner et améliorer vos requêtes Gremlin à l’aide de l’étape de profil d’exécution.
services: cosmos-db
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 03/27/2019
ms.author: chrande
ms.openlocfilehash: 18cefb1dd80368a8ccdad9f6f3ffc30881a8a889
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93087483"
---
# <a name="how-to-use-the-execution-profile-step-to-evaluate-your-gremlin-queries"></a>Guide pratique pour utiliser l’étape de profil d’exécution pour évaluer vos requêtes Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article fournit une vue d’ensemble de l’utilisation de l’étape de profil d’exécution pour les bases de données de graphe d’API Gremlin Azure Cosmos DB. Cette étape fournit des informations pertinentes sur la résolution des problèmes et l’optimisation des requêtes. Elle est compatible avec n’importe quelle requête Gremlin pouvant être exécutée sur un compte d’API Gremlin Cosmos DB.

Pour utiliser cette étape, ajoutez simplement l’appel de fonction `executionProfile()` à la fin de votre requête Gremlin. **Votre requête Gremlin sera exécutée** et le résultat de l’opération retournera un objet de réponse JSON avec le profil d’exécution de requête.

Par exemple :

```java
    // Basic traversal
    g.V('mary').out()

    // Basic traversal with execution profile call
    g.V('mary').out().executionProfile()
```

Après l’appel de l’étape `executionProfile()`, la réponse est un objet JSON qui comprend l’étape Gremlin exécutée, la durée totale nécessaire et un tableau des opérateurs de runtime Cosmos DB générés par l’instruction.

> [!NOTE]
> Cette implémentation pour le profil d’exécution n’est pas définie dans la spécification Apache Tinkerpop. Elle est propre à l’implémentation de l’API Gremlin Azure Cosmos DB.


## <a name="response-example"></a>Exemple de réponse

Voici un exemple annoté de sortie retournée :

> [!NOTE]
> Cet exemple est annoté avec des commentaires qui expliquent la structure générale de la réponse. Une réponse executionProfile réelle ne contiendra aucun commentaire.

```json
[
  {
    // The Gremlin statement that was executed.
    "gremlin": "g.V('mary').out().executionProfile()",

    // Amount of time in milliseconds that the entire operation took.
    "totalTime": 28,

    // An array containing metrics for each of the steps that were executed. 
    // Each Gremlin step will translate to one or more of these steps.
    // This list is sorted in order of execution.
    "metrics": [
      {
        // This operation obtains a set of Vertex objects.
        // The metrics include: time, percentTime of total execution time, resultCount, 
        // fanoutFactor, count, size (in bytes) and time.
        "name": "GetVertices",
        "time": 24,
        "annotations": {
          "percentTime": 85.71
        },
        "counts": {
          "resultCount": 2
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 2,
            "size": 696,
            "time": 0.4
          }
        ]
      },
      {
        // This operation obtains a set of Edge objects. 
        // Depending on the query, these might be directly adjacent to a set of vertices, 
        // or separate, in the case of an E() query.
        //
        // The metrics include: time, percentTime of total execution time, resultCount, 
        // fanoutFactor, count, size (in bytes) and time.
        "name": "GetEdges",
        "time": 4,
        "annotations": {
          "percentTime": 14.29
        },
        "counts": {
          "resultCount": 1
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 1,
            "size": 419,
            "time": 0.67
          }
        ]
      },
      {
        // This operation obtains the vertices that a set of edges point at.
        // The metrics include: time, percentTime of total execution time and resultCount.
        "name": "GetNeighborVertices",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 1
        }
      },
      {
        // This operation represents the serialization and preparation for a result from 
        // the preceding graph operations. The metrics include: time, percentTime of total 
        // execution time and resultCount.
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 1
        }
      }
    ]
  }
]
```

> [!NOTE]
> L’étape executionProfile exécute la requête Gremlin. Cela comprend les étapes `addV` ou `addE`, qui entraînent la création et la validation des modifications spécifiées dans la requête. Par conséquent, les unités de requête générées par la requête Gremlin seront également facturées.

## <a name="execution-profile-response-objects"></a>Objets de réponse de profil d’exécution

La réponse d’une fonction executionProfile() génère une hiérarchie d’objets JSON avec la structure suivante :
  - **Objet d’opération Gremlin**  : représente l’intégralité de l’opération Gremlin qui a été exécutée. Contient les propriétés suivantes.
    - `gremlin`: instruction Gremlin explicite qui a été exécutée.
    - `totalTime`: durée, en millisecondes, nécessaire à l’exécution de l’étape. 
    - `metrics`: tableau qui contient tous les opérateurs de runtime Cosmos DB qui ont été exécutés pour répondre à la requête. Cette liste est triée par ordre d’exécution.
    
  - **Opérateurs de runtime Cosmos DB**  : représente chacun des composants de l’opération Gremlin entière. Cette liste est triée par ordre d’exécution. Chaque objet contient les propriétés suivantes :
    - `name`: nom de l’opérateur. Il s’agit du type d’étape qui a été évaluée et exécutée. Pour en savoir plus, consultez le tableau ci-dessous.
    - `time`: durée, en millisecondes, nécessaire à l’exécution d’un opérateur donné.
    - `annotations`: contient des informations supplémentaires propres à l’opérateur exécuté.
    - `annotations.percentTime`: pourcentage de la durée totale consacré à l’exécution de l’opérateur en question.
    - `counts`: nombre d’objets qui ont été retournés à partir de la couche de stockage par cet opérateur. Ce nombre est contenu dans la valeur scalaire `counts.resultCount`.
    - `storeOps`: représente une opération de stockage qui peut couvrir une ou plusieurs partitions.
    - `storeOps.fanoutFactor`: représente le nombre de partitions sollicitées par cette opération de stockage spécifique.
    - `storeOps.count`: représente le nombre de résultats retournés par cette opération de stockage.
    - `storeOps.size`: représente la taille en octets du résultat d’une opération de stockage donnée.

Opérateur de runtime Gremlin Cosmos DB|Description
---|---
`GetVertices`| Cette étape obtient un ensemble d’objets à prédicat à partir de la couche de persistance. 
`GetEdges`| Cette étape obtient les bords adjacents à un ensemble de vertex. Cette étape peut entraîner une ou plusieurs opérations de stockage.
`GetNeighborVertices`| Cette étape obtient les vertex connectés à un ensemble de bords. Les bords contiennent les clés de partitions et les ID de leurs vertex source et cible.
`Coalesce`| Cette étape prend en compte l’évaluation de deux opérations chaque fois que l’étape Gremlin `coalesce()` est exécutée.
`CartesianProductOperator`| Cette étape calcule un produit cartésien de deux jeux de données. Elle est généralement exécutée chaque fois que les prédicats `to()` ou `from()` sont utilisés.
`ConstantSourceOperator`| Cette étape calcule une expression pour produire une valeur constante comme résultat.
`ProjectOperator`| Cette étape prépare et sérialise une réponse à l’aide du résultat des opérations précédentes.
`ProjectAggregation`| Cette étape prépare et sérialise une réponse pour une opération d’agrégation.

> [!NOTE]
> Cette liste continuera à être mise à jour au fur et à mesure de l’ajout de nouveaux opérateurs.

## <a name="examples-on-how-to-analyze-an-execution-profile-response"></a>Exemples d’analyse d’une réponse de profil d’exécution

Voici quelques exemples d’optimisations courantes qui peuvent être détectées à l’aide de la réponse de profil d’exécution :
  - Requête de distribution ramifiée aveugle.
  - Requête non filtrée.

### <a name="blind-fan-out-query-patterns"></a>Modèles de requête de distribution ramifiée aveugle

Considérez la réponse de profil d’exécution suivante provenant d’un **graphe partitionné**  :

```json
[
  {
    "gremlin": "g.V('tt0093640').executionProfile()",
    "totalTime": 46,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 46,
        "annotations": {
          "percentTime": 100
        },
        "counts": {
          "resultCount": 1
        },
        "storeOps": [
          {
            "fanoutFactor": 5,
            "count": 1,
            "size": 589,
            "time": 75.61
          }
        ]
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 1
        }
      }
    ]
  }
]
```

Nous pouvons en tirer les conclusions suivantes :
- La requête est une recherche d’ID unique, puisque l’instruction Gremlin suit le modèle `g.V('id')`.
- À en juger par la métrique `time`, la latence de cette requête semble être élevée, car elle est [supérieure à 10 ms pour une seule opération de lecture de point](./introduction.md#guaranteed-speed-at-any-scale).
- Si nous examinons l’objet `storeOps`, nous constatons que `fanoutFactor` est égal à `5`, ce qui signifie que [cinq partitions](./partitioning-overview.md) ont été sollicitées par cette opération.

En conclusion de cette analyse, nous pouvons dire que la première requête accède à plus de partitions que nécessaire. Ce problème peut être résolu en spécifiant la clé de partitionnement dans la requête en tant que prédicat. Cela permettra de réduire la latence et le coût par requête. Explorez plus en détail le [partitionnement de graphe](graph-partitioning.md). `g.V('tt0093640').has('partitionKey', 't1001')` serait une requête plus optimale.

### <a name="unfiltered-query-patterns"></a>Modèles de requête non filtrée

Comparez les deux réponses de profil d’exécution suivantes. Pour rester simple, ces exemples utilisent un seul graphe partitionné.

Cette première requête récupère tous les sommets avec l’étiquette `tweet`, puis obtient leurs sommets voisins :

```json
[
  {
    "gremlin": "g.V().hasLabel('tweet').out().executionProfile()",
    "totalTime": 42,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 31,
        "annotations": {
          "percentTime": 73.81
        },
        "counts": {
          "resultCount": 30
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 13,
            "size": 6819,
            "time": 1.02
          }
        ]
      },
      {
        "name": "GetEdges",
        "time": 6,
        "annotations": {
          "percentTime": 14.29
        },
        "counts": {
          "resultCount": 18
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 20,
            "size": 7950,
            "time": 1.98
          }
        ]
      },
      {
        "name": "GetNeighborVertices",
        "time": 5,
        "annotations": {
          "percentTime": 11.9
        },
        "counts": {
          "resultCount": 20
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 4,
            "size": 1070,
            "time": 1.19
          }
        ]
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 20
        }
      }
    ]
  }
]
```

Notez le profil de la même requête, mais désormais avec un filtre supplémentaire, `has('lang', 'en')`, avant d’explorer les sommets adjacents :

```json
[
  {
    "gremlin": "g.V().hasLabel('tweet').has('lang', 'en').out().executionProfile()",
    "totalTime": 14,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 14,
        "annotations": {
          "percentTime": 58.33
        },
        "counts": {
          "resultCount": 11
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 11,
            "size": 4807,
            "time": 1.27
          }
        ]
      },
      {
        "name": "GetEdges",
        "time": 5,
        "annotations": {
          "percentTime": 20.83
        },
        "counts": {
          "resultCount": 18
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 18,
            "size": 7159,
            "time": 1.7
          }
        ]
      },
      {
        "name": "GetNeighborVertices",
        "time": 5,
        "annotations": {
          "percentTime": 20.83
        },
        "counts": {
          "resultCount": 18
        },
        "storeOps": [
          {
            "fanoutFactor": 1,
            "count": 4,
            "size": 1070,
            "time": 1.01
          }
        ]
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 18
        }
      }
    ]
  }
]
```

Ces deux requêtes atteignent le même résultat, mais la première nécessite davantage d’unités de requête car elle doit itérer un plus grand jeu de données initial avant d’interroger les éléments adjacents. Nous pouvons voir des indicateurs de ce comportement quand nous comparons les paramètres suivants des deux réponses :
- La valeur `metrics[0].time` est plus élevée dans la première réponse, ce qui indique que cette étape a duré plus longtemps.
- La valeur `metrics[0].counts.resultsCount` est également plus élevée dans la première réponse, ce qui indique que le jeu de données de travail initial était plus volumineux.

## <a name="next-steps"></a>Étapes suivantes
* Apprenez-en davantage sur les [fonctionnalités Gremlin prises en charge](gremlin-support.md) dans Azure Cosmos DB. 
* Apprenez-en davantage sur l’[API Gremlin dans Azure Cosmos DB](graph-introduction.md).