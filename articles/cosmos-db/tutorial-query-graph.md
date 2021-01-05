---
title: Comment interroger des données de graphique dans Azure Cosmos DB ?
description: Découvrez comment interroger des données de graphes depuis Azure Cosmos DB en utilisant des requêtes Gremlin
author: christopheranderson
ms.author: chrande
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: tutorial
ms.date: 12/03/2018
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 545ffd303d2039a3c54088220c1fa74e742c750f
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360766"
---
# <a name="tutorial-query-azure-cosmos-db-gremlin-api-by-using-gremlin"></a>Didacticiel : interroger Azure Cosmos DB à l’aide de l’API Gremlin et de Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

L’[API Gremlin](graph-introduction.md) d’Azure Cosmos DB prend en charge les requêtes [Gremlin](https://github.com/tinkerpop/gremlin/wiki). Cet article fournit des exemples de documents et de requêtes pour vous aider à démarrer. L’article relatif à la [prise en charge de Gremlin](gremlin-support.md) comporte des informations de référence détaillées sur Gremlin.

Cet article décrit les tâches suivantes : 

> [!div class="checklist"]
> * Interrogation des données avec Gremlin

## <a name="prerequisites"></a>Conditions préalables requises

Pour le bon fonctionnement de ces requêtes, vous devez disposer d’un compte Azure Cosmos DB et de données de graphique dans le conteneur. Cela n’est pas le cas ? Lancez le [démarrage rapide de 5 minutes](create-graph-dotnet.md) ou le [tutoriel destiné aux développeurs](tutorial-query-graph.md) pour créer un compte et alimenter votre base de données. Vous pouvez exécuter les requêtes suivantes avec la [console Gremlin](https://tinkerpop.apache.org/docs/current/reference/#gremlin-console) ou avec votre pilote Gremlin favori.

## <a name="count-vertices-in-the-graph"></a>Compter le nombre de vertex du graphique

L’extrait de code suivant montre comment compter le nombre de vertex du graphique :

```
g.V().count()
```

## <a name="filters"></a>Filtres

Vous pouvez appliquer des filtres à l’aide des étapes `has` et `hasLabel` de Gremlin et les associer à l’aide de `and`, `or`, et `not` pour créer des filtres plus complexes. Azure Cosmos DB propose une indexation sans schéma de toutes les propriétés au sein de vos vertex et degrés pour les requêtes rapides :

```
g.V().hasLabel('person').has('age', gt(40))
```

## <a name="projection"></a>Projection

Vous pouvez projeter certaines propriétés dans les résultats de requête à l’aide de l’étape `values` :

```
g.V().hasLabel('person').values('firstName')
```

## <a name="find-related-edges-and-vertices"></a>Rechercher des vertex et des bords associés

Jusqu’ici, nous avons seulement abordé les opérateurs de requête fonctionnant dans n’importe quelle base de données. Les graphiques sont rapides et efficaces pour les opérations de traversée lorsque vous avez besoin d’accéder aux vertex et bords associés. Recherchons à présent tous les amis de Thomas. Nous suivons pour cela l’étape `outE` de Gremlin pour rechercher toutes les bords externes de Thomas afin de se diriger vers les vertex internes de ces bords en suivant l’étape `inV` de Gremlin :

```cs
g.V('thomas').outE('knows').inV().hasLabel('person')
```

La requête suivante effectue deux sauts pour rechercher tous les « amis des amis de Thomas », en appelant `outE` et `inV` deux fois. 

```cs
g.V('thomas').outE('knows').inV().hasLabel('person').outE('knows').inV().hasLabel('person')
```

Vous pouvez créer des requêtes plus complexes et implémenter une logique de traversée de graphique puissante à l’aide de Gremlin, y compris en mixant des expressions de filtre, en exécutant des boucles à l’aide de l’étape `loop` et en mettant en œuvre la navigation conditionnelle à l’aide de l’étape `choose`. En savoir plus sur ce que la [prise en charge de Gremlin](gremlin-support.md) vous permet de faire !

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez :

> [!div class="checklist"]
> * Effectuer des interrogations à l’aide de Graph 

Vous pouvez maintenant passer à la section Concepts pour plus d’informations sur Cosmos DB.

> [!div class="nextstepaction"]
> [Diffusion mondiale](distribute-data-globally.md) 

