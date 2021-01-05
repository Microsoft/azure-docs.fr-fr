---
title: Modélisation des données de graphe pour l’API Gremlin d’Azure Cosmos DB
description: Découvrez comment modéliser une base de données de graphe avec l’API Gremlin d’Azure Cosmos DB. Cet article explique à quel moment utiliser une base de données de graphe et précise les bonnes pratiques de modélisation des entités et des relations.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 12/02/2019
ms.author: chrande
ms.openlocfilehash: d99e2e2ffd63b050e7373c98084fed3fb14727bf
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93357043"
---
# <a name="graph-data-modeling-for-azure-cosmos-db-gremlin-api"></a>Modélisation des données de graphe pour l’API Gremlin d’Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Le document suivant est conçu pour fournir des recommandations de modélisation des données de graphe. Cette étape est essentielle afin de garantir la scalabilité et les performances d’un système de base de données de graphe à mesure que les données évoluent. Un modèle de données efficace est particulièrement important avec des graphes à grande échelle.

## <a name="requirements"></a>Spécifications

Le processus décrit dans ce guide est basé sur les hypothèses suivantes :
 * Les **entités** dans l’espace du problème sont identifiées. Ces entités sont destinées à être consommés _atomiquement_ pour chaque demande. En d’autres termes, le système de base de données n’est pas conçu pour récupérer les données d’une entité unique dans plusieurs demandes de requête.
 * Vous comprenez les **exigences en lecture et en écriture** pour le système de base de données. Ces exigences vont guider les optimisations nécessaires pour le modèle de données de graphe.
 * Les principes du [standard des graphes de propriétés Apache Tinkerpop](https://tinkerpop.apache.org/docs/current/reference/#graph-computing) sont bien compris.

## <a name="when-do-i-need-a-graph-database"></a>Quand ai-je besoin d’une base de données de graphe ?

Une solution de base de données de graphe peut être appliquée de façon optimale si les entités et les relations dans un domaine de données ont une des caractéristiques suivantes : 

* Les entités sont **fortement connectées** via des relations descriptives. L’avantage dans ce scénario est le fait que les relations sont conservées dans le stockage.
* Il existe des **relations cycliques** ou des **entités auto-référencées**. Ce modèle représente souvent un certain défi lors de l’utilisation de bases de données relationnelles ou de documents.
* Il existe des **relations évoluant dynamiquement** entre les entités. Ce modèle s’applique en particulier à des données structurées en arborescence ou de façon hiérarchique avec de nombreux niveaux.
* Il existe des **relations plusieurs-à-plusieurs** entre les entités.
* Il existe des **exigences d’écriture et de lecture sur les entités et les relations**. 

Si les critères ci-dessus sont remplis, il est probable qu’une approche de base de données de graphe fournira des avantages pour la **complexité des requêtes** , la **scalabilité du modèle de données** et les **performances des requêtes**.

L’étape suivante consiste à déterminer si le graphe va être utilisé à des fins analytiques ou transactionnelles. Si le graphe est destiné à être utilisé pour des calculs intensifs et des charges de travail importantes de traitement des données, il peut être utile d’explorer le [connecteur Spark Cosmos DB](./spark-connector.md) et l’utilisation de la [bibliothèque GraphX](https://spark.apache.org/graphx/). 

## <a name="how-to-use-graph-objects"></a>Comment utiliser des objets de graphe

Le [standard des graphes de propriétés Apache Tinkerpop](https://tinkerpop.apache.org/docs/current/reference/#graph-computing) définit deux types d’objets : les **sommets** et les **arêtes**. 

Voici les bonnes pratiques pour les propriétés dans les objets de graphe :

| Object | Propriété | Type | Notes |
| --- | --- | --- |  --- |
| Sommet | id | String | Appliqué de façon unique par partition. Si une valeur n’est pas fournie lors de l’insertion, un GUID généré automatiquement sera stocké. |
| Sommet | label | String | Cette propriété est utilisée pour définir le type d’entité que représente le sommet. Si une valeur n’est pas fournie, la valeur par défaut « vertex » (sommet) sera utilisée. |
| Sommet | properties | Chaîne, booléen, numérique | Une liste de propriétés distinctes stockées sous forme de paires clé-valeur dans chaque sommet. |
| Sommet | clé de partition | Chaîne, booléen, numérique | Cette propriété définit où le sommet et ses arêtes sortantes seront stockés. Pour plus d’informations, consultez [partitionnement de graphe](graph-partitioning.md). |
| Edge | id | String | Appliqué de façon unique par partition. Généré automatiquement par défaut. Les arêtes n’ont généralement pas besoin d’être récupérées de façon univoque via un ID. |
| Edge | label | String | Cette propriété est utilisée pour définir le type de relation entre deux sommets. |
| Edge | properties | Chaîne, booléen, numérique | Une liste de propriétés distinctes stockées sous forme de paires clé-valeur dans chaque arête. |

> [!NOTE]
> Les arêtes ne nécessitent pas une valeur de clé de partition, car leur valeur est affectée automatiquement en fonction de leur sommet source. Pour plus d’informations, consultez l’article [Partitionnement de graphe](graph-partitioning.md).

## <a name="entity-and-relationship-modeling-guidelines"></a>Instructions pour la modélisation des entités et des relations

Voici un ensemble de recommandations sur l’approche de la modélisation des données pour une base de données de graphe de l’API Gremlin d’Azure Cosmos DB. Ces instructions supposent qu’il existe une définition d’un domaine de données et des requêtes pour celui-ci.

> [!NOTE]
> Les étapes décrites ci-dessous sont présentées en tant que recommandations. Le modèle final doit être évalué et testé avant qu’il soit considéré comme prêt pour la production. De plus, les recommandations ci-dessous sont spécifiques à l’implémentation de l’API Gremlin d’Azure Cosmos DB. 

### <a name="modeling-vertices-and-properties"></a>Modélisation des sommets et des propriétés 

La première étape pour un modèle de données de graphe consiste à mapper chaque entité identifiée à un **objet sommet**. Un mappage un-à-un de toutes les entités aux sommets doit être une étape initiale et est sujet à modification.

Un piège courant est de mapper les propriétés d’une même entité en tant que sommet distinct. Prenons l’exemple ci-dessous, où la même entité est représentée de deux manières différentes :

* **Propriétés basées sur les sommets**  : Dans cette approche, l’entité utilise trois sommets distincts et deux arêtes pour décrire ses propriétés. Bien que cette approche puisse réduire la redondance, elle accroît la complexité du modèle. Un accroissement de la complexité du modèle peut entraîner une augmentation des temps de latence, de la complexité des requêtes et du temps de calcul. Ce modèle peut également présenter des problèmes liés au partitionnement.

:::image type="content" source="./media/graph-modeling/graph-modeling-1.png" alt-text="Modèle d’entité avec des sommets pour les propriétés." border="false":::

* **Sommet avec des propriétés incorporées**  : Cette approche tire parti de la liste de paires clé-valeur pour représenter toutes les propriétés de l’entité à l’intérieur d’un sommet. Cette approche aboutit à une complexité réduite du modèle, ce qui permet des requêtes plus simples et des traversées moins consommatrices de ressources.

:::image type="content" source="./media/graph-modeling/graph-modeling-2.png" alt-text="Diagramme montrant le sommet Luis du diagramme précédent avec ID, étiquette et propriétés." border="false":::

> [!NOTE]
> Les exemples ci-dessus montrent un modèle de graphe simplifié seulement pour comparer les deux méthodes de division des propriétés de l’entité.

Le modèle de **sommet avec propriétés incorporées** fournit généralement une approche plus performante et plus évolutive. L’approche par défaut pour un nouveau modèle de données de graphe doit se faire autour de ce modèle.

Il existe cependant des scénarios où faire référence à une propriété peut offrir des avantages. Par exemple : si la propriété référencée est fréquemment mise à jour. Utiliser un sommet distinct pour représenter une propriété qui est constamment modifiée va minimiser la quantité d’opérations d’écriture que la mise à jour va nécessiter.

### <a name="relationship-modeling-with-edge-directions"></a>Modélisation des relations avec les sens des arêtes

Une fois les sommets modélisés, les arêtes peuvent être ajoutées pour indiquer les relations entre ceux-ci. Le premier aspect qui doit être évalué est le **sens de la relation**. 

Les objets arête ont un sens par défaut qui est suivi par une traversée lors de l’utilisation de la fonction `out()` ou `outE()`. L’utilisation de cette direction naturelle aboutit à un fonctionnement efficace, car tous les sommets sont stockés avec leurs arêtes sortantes. 

Cependant, traverser dans le sens opposé d’une arête, en utilisant la fonction `in()`, aboutit toujours à une requête multipartition. Explorez plus en détail le [partitionnement de graphe](graph-partitioning.md). S’il est nécessaire de traverser constamment en utilisant la fonction `in()`, il est recommandé d’ajouter des arêtes dans les deux sens.

Vous pouvez déterminer le sens de l’arête en utilisant les prédicats `.to()` ou `.from()` pour l’étape `.addE()` de Gremlin. Vous pouvez aussi utiliser la [bibliothèque BulkExecutor pour l’API Gremlin](bulk-executor-graph-dotnet.md).

> [!NOTE]
> Les objets arête ont un sens par défaut.

### <a name="relationship-labeling"></a>Étiquetage des relations

L’utilisation d’étiquettes de relation descriptives peut améliorer l’efficacité des opérations de résolution des arêtes. Ce modèle peut être appliqué de ces différentes façons :
* Utilisez des termes non génériques pour étiqueter une relation.
* Associez l’étiquette du sommet source à l’étiquette du sommet cible avec le nom de la relation.

:::image type="content" source="./media/graph-modeling/graph-modeling-3.png" alt-text="Exemples d’étiquetage des relations." border="false":::

Plus l’étiquette utilisée par la traversée pour filtrer les arêtes sera spécifique, meilleur sera le résultat. Cette décision peut également avoir un impact significatif sur le coût des requêtes. Vous pouvez évaluer le coût des requêtes à tout moment [en utilisant l’étape executionProfile](graph-execution-profile.md).


## <a name="next-steps"></a>Étapes suivantes : 
* Consultez la liste des [étapes Gremlin](gremlin-support.md) prises en charge.
* Découvrez plus d’informations sur le [partitionnement des bases de données de graphe](graph-partitioning.md) pour gérer les graphes à grande échelle.
* Évaluez vos requêtes Gremlin en utilisant l’[étape de profil d’exécution](graph-execution-profile.md).