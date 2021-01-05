---
title: Prise en charge d’Azure Cosmos DB Gremlin et compatibilité avec les fonctionnalités TinkerPop
description: Découvrez le langage Gremlin d’Apache TinkerPop. Découvrez les fonctionnalités et les étapes disponibles dans Azure Cosmos DB et les différences de compatibilité avec le moteur de graphes TinkerPop.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: overview
ms.date: 11/11/2020
ms.author: sngun
ms.openlocfilehash: a149f0b331a77462aa53b948fedf25dd1331969e
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "94683622"
---
# <a name="azure-cosmos-db-gremlin-graph-support-and-compatibility-with-tinkerpop-features"></a>Prise en charge des graphes Azure Cosmos DB Gremlin et compatibilité avec les fonctionnalités TinkerPop
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Azure Cosmos DB prend en charge le langage de traversées de graphes [Apache Tinkerpop](https://tinkerpop.apache.org), connu sous le nom de [Gremlin](https://tinkerpop.apache.org/docs/3.3.2/reference/#graph-traversal-steps). Vous pouvez utiliser le langage Gremlin pour créer des entités de graphes (sommets et arêtes), modifier les propriétés au sein de ces entités, exécuter des requêtes et traversées et supprimer des entités.

Le moteur Azure Cosmos DB Graph suit de près la spécification des étapes de traversée [Apache TinkerPop](https://tinkerpop.apache.org/docs/current/reference/#graph-traversal-steps), mais avec quelques différences au niveau de l’implémentation, spécifiques à Azure Cosmos DB. Dans cet article, nous fournissons une procédure pas à pas pour Gremlin, et nous énumérons les fonctionnalités Gremlin qui sont prises en charge par l’API Gremlin.

## <a name="compatible-client-libraries"></a>Bibliothèques clientes compatibles

Le tableau suivant présente des pilotes Gremlin courants que vous pouvez utiliser sur Azure Cosmos DB :

| Téléchargement | Source | Mise en route | Version du connecteur prise en charge |
| --- | --- | --- | --- |
| [.NET](https://tinkerpop.apache.org/docs/3.4.6/reference/#gremlin-DotNet) | [Gremlin.NET sur GitHub](https://github.com/apache/tinkerpop/tree/master/gremlin-dotnet) | [Créer un graphe avec .NET](create-graph-dotnet.md) | 3.4.6 |
| [Java](https://mvnrepository.com/artifact/com.tinkerpop.gremlin/gremlin-java) | [Gremlin JavaDoc](https://tinkerpop.apache.org/javadocs/current/full/) | [Créer un graphe avec Java](create-graph-java.md) | 3.2.0+ |
| [Node.JS](https://www.npmjs.com/package/gremlin) | [Gremlin-JavaScript sur GitHub](https://github.com/apache/tinkerpop/tree/master/gremlin-javascript) | [Créer un graphe avec Node.js](create-graph-nodejs.md) | 3.3.4+ |
| [Python](https://tinkerpop.apache.org/docs/3.3.1/reference/#gremlin-python) | [Gremlin-Python sur GitHub](https://github.com/apache/tinkerpop/tree/master/gremlin-python) | [Créer un graphe avec Python](create-graph-python.md) | 3.2.7 |
| [PHP](https://packagist.org/packages/brightzone/gremlin-php) | [Gremlin-PHP sur GitHub](https://github.com/PommeVerte/gremlin-php) | [Créer un graphe avec PHP](create-graph-php.md) | 3.1.0 |
| [Go Lang](https://github.com/supplyon/gremcos/) | [Go Lang](https://github.com/supplyon/gremcos/) | | Cette bibliothèque a été créée par des contributeurs externes. L’équipe Azure Cosmos DB ne fournit donc aucun support ni maintenance pour celle-ci. |
| [Console Gremlin](https://tinkerpop.apache.org/downloads.html) | [Documents TinkerPop](https://tinkerpop.apache.org/docs/current/reference/#gremlin-console) |  [Créer un graphe à l’aide de la console Gremlin](create-graph-gremlin-console.md) | 3.2.0 + |

## <a name="supported-graph-objects"></a>Objets graphiques pris en charge

TinkerPop est une norme qui couvre un large éventail de technologies de graphes. Par conséquent, elle dispose de la terminologie standard utilisée pour décrire les fonctionnalités offertes par un fournisseur de graphes. Azure Cosmos DB fournit une base de données de graphes permanente, concurrentielle et accessible en écriture qui peut être partitionnée entre plusieurs serveurs ou clusters. 

Le tableau suivant répertorie les fonctionnalités TinkerPop implémentées par Azure Cosmos DB : 

| Category | Implémentation d’Azure Cosmos DB |  Notes | 
| --- | --- | --- |
| Fonctionnalités de graphe | Fournit la persistance et ConcurrentAccess. Conçu pour prendre en charge les transactions | Les méthodes de l’ordinateur peuvent être implémentées via le connecteur Spark. |
| Fonctionnalités variables | Prend en charge les valeurs booléennes, entières, byte, doubles, flottantes, longues et de chaîne | Prend en charge les types primitifs, et est compatible avec des types complexes via un modèle de données |
| Fonctionnalités de sommet | Prend en charge RemoveVertices, MetaProperties, AddVertices, MultiProperties, StringIds, UserSuppliedIds, AddProperty, RemoveProperty  | Prend en charge la création, la modification et la suppression de sommet |
| Fonctionnalités de propriétés de sommet | StringIds, UserSuppliedIds, AddProperty, RemoveProperty, BooleanValues, ByteValues, DoubleValues, FloatValues, IntegerValues, LongValues, StringValues | Prend en charge la création, la modification et la suppression des propriétés de sommet |
| Fonctionnalités d’arête | AddEdges, RemoveEdges, StringIds, UserSuppliedIds, AddProperty, RemoveProperty | Prend en charge la création, la modification et la suppression d’arêtes |
| Fonctionnalités de propriétés d’arête | Properties, BooleanValues, ByteValues, DoubleValues, FloatValues, IntegerValues, LongValues, StringValues | Prend en charge la création, la modification et la suppression d’arêtes |

## <a name="gremlin-wire-format"></a>Format de communication Gremlin

Azure Cosmos DB utilise le format JSON quand les résultats des opérations Gremlin sont retournés. Azure Cosmos DB prend actuellement en charge le format JSON. Par exemple, l’extrait de code suivant montre une représentation JSON d’un sommet *retourné au client* à partir d’Azure Cosmos DB :

```json
  {
    "id": "a7111ba7-0ea1-43c9-b6b2-efc5e3aea4c0",
    "label": "person",
    "type": "vertex",
    "outE": {
      "knows": [
        {
          "id": "3ee53a60-c561-4c5e-9a9f-9c7924bc9aef",
          "inV": "04779300-1c8e-489d-9493-50fd1325a658"
        },
        {
          "id": "21984248-ee9e-43a8-a7f6-30642bc14609",
          "inV": "a8e3e741-2ef7-4c01-b7c8-199f8e43e3bc"
        }
      ]
    },
    "properties": {
      "firstName": [
        {
          "value": "Thomas"
        }
      ],
      "lastName": [
        {
          "value": "Andersen"
        }
      ],
      "age": [
        {
          "value": 45
        }
      ]
    }
  }
```

Les propriétés utilisées par le format JSON pour les sommets sont décrites ci-dessous :

| Propriété | Description | 
| --- | --- | --- |
| `id` | ID du sommet. Doit être unique (en association avec la valeur de `_partition`, si applicable). Si aucune valeur n’est fournie, un GUID est fourni automatiquement. | 
| `label` | Intitulé du sommet. Cette propriété est utilisée pour décrire le type d’entité. |
| `type` | Utilisé pour distinguer les sommets des documents non-graphes |
| `properties` | Sac de propriétés définies par l’utilisateur associé au sommet. Chaque propriété peut avoir plusieurs valeurs. |
| `_partition` | Clé de partition du sommet. Utilisée pour le [partitionnement de graphe](graph-partitioning.md). |
| `outE` | Cette propriété contient une liste des arêtes externes d’un sommet. Permet de stocker les informations de contiguïté avec des sommets pour une exécution rapide des traversées. Les arêtes sont regroupées en fonction de leurs intitulés. |

Et l’arête contient les informations suivantes pour faciliter la navigation vers d’autres parties du graphe.

| Propriété | Description |
| --- | --- |
| `id` | ID de l’arête. Doit être unique (en association avec la valeur de `_partition`, si applicable). |
| `label` | Intitulé de l’arête. Cette propriété est facultative, elle est utilisée pour décrire le type de relation. |
| `inV` | Cette propriété contient la liste des sommets d’une arête. Permet de stocker les informations de contiguïté avec l’arête pour une exécution rapide des traversées. Les sommets sont regroupés en fonction de leurs intitulés. |
| `properties` | Sac de propriétés définies par l’utilisateur associé à l’arête. Chaque propriété peut avoir plusieurs valeurs. |

Chaque propriété peut stocker plusieurs valeurs dans un tableau. 

| Propriété | Description |
| --- | --- |
| `value` | Valeur de la propriété

## <a name="gremlin-steps"></a>Étapes de Gremlin

Nous allons maintenant examiner les étapes Gremlin prises en charge par Azure Cosmos DB. Pour des références complètes sur Gremlin, consultez [Référence TinkerPop](https://tinkerpop.apache.org/docs/3.3.2/reference).

| étape | Description | Documentation TinkerPop 3.2 |
| --- | --- | --- |
| `addE` | Ajoute une arête reliant deux sommets | [étape addE](https://tinkerpop.apache.org/docs/3.3.2/reference/#addedge-step) |
| `addV` | Ajoute un sommet au graphe | [étape addV](https://tinkerpop.apache.org/docs/3.3.2/reference/#addvertex-step) |
| `and` | Fait en sorte que toutes les traversées retournent une valeur | [étape and](https://tinkerpop.apache.org/docs/3.3.2/reference/#and-step) |
| `as` | Un modulateur d’étape pour attribuer une variable à la sortie d’une étape | [étape as](https://tinkerpop.apache.org/docs/3.3.2/reference/#as-step) |
| `by` | Un modulateur d’étape utilisé avec `group` et `order` | [étape by](https://tinkerpop.apache.org/docs/3.3.2/reference/#by-step) |
| `coalesce` | Renvoie la première traversée qui renvoie un résultat | [étape coalesce](https://tinkerpop.apache.org/docs/3.3.2/reference/#coalesce-step) |
| `constant` | Renvoie une valeur constante. Utilisé avec `coalesce`| [étape constant](https://tinkerpop.apache.org/docs/3.3.2/reference/#constant-step) |
| `count` | Renvoie le nombre de traversées | [étape count](https://tinkerpop.apache.org/docs/3.3.2/reference/#count-step) |
| `dedup` | Renvoie les valeurs en supprimant les doublons | [étape dedup](https://tinkerpop.apache.org/docs/3.3.2/reference/#dedup-step) |
| `drop` | Supprime les valeurs (sommet/arête) | [étape drop](https://tinkerpop.apache.org/docs/3.3.2/reference/#drop-step) |
| `executionProfile` | Crée une description de toutes les opérations générées par l’étape Gremlin exécutée | [Étape executionProfile](graph-execution-profile.md) |
| `fold` | Agit comme une barrière qui calcule l’agrégation des résultats| [étape fold](https://tinkerpop.apache.org/docs/3.3.2/reference/#fold-step) |
| `group` | Regroupe les valeurs basées sur les labels spécifiés| [étape group](https://tinkerpop.apache.org/docs/3.3.2/reference/#group-step) |
| `has` | Permet de filtrer les propriétés, les sommets et les arêtes. Prend en charge `hasLabel`, `hasId`, `hasNot`, et les variantes `has`. | [étape has](https://tinkerpop.apache.org/docs/3.3.2/reference/#has-step) |
| `inject` | Injecter des valeurs dans un flux de données| [étape inject](https://tinkerpop.apache.org/docs/3.3.2/reference/#inject-step) |
| `is` | Permet d’exécuter un filtre à l’aide d’une expression booléenne | [étape is](https://tinkerpop.apache.org/docs/3.3.2/reference/#is-step) |
| `limit` | Permet de limiter le nombre d’éléments dans la traversée| [étape limit](https://tinkerpop.apache.org/docs/3.3.2/reference/#limit-step) |
| `local` | Local encapsule une section d’une traversée, similaire à une sous-requête | [étape local](https://tinkerpop.apache.org/docs/3.3.2/reference/#local-step) |
| `not` | Permet de produire la négation d’un filtre | [étape not](https://tinkerpop.apache.org/docs/3.3.2/reference/#not-step) |
| `optional` | Renvoie le résultat de la traversée spécifiée si elle produit un résultat, sinon renvoie l’élément appelant | [étape optional](https://tinkerpop.apache.org/docs/3.3.2/reference/#optional-step) |
| `or` | Garantit qu’au moins l’une des traversées renvoie une valeur | [étape or](https://tinkerpop.apache.org/docs/3.3.2/reference/#or-step) |
| `order` | Renvoie les résultats dans l’ordre de tri spécifié | [étape order](https://tinkerpop.apache.org/docs/3.3.2/reference/#order-step) |
| `path` | Renvoie le chemin d’accès complet de la traversée | [étape path](https://tinkerpop.apache.org/docs/3.3.2/reference/#path-step) |
| `project` | Projette les propriétés sous forme de carte | [étape project](https://tinkerpop.apache.org/docs/3.3.2/reference/#project-step) |
| `properties` | Renvoie les propriétés pour les labels spécifiés | [étape properties](https://tinkerpop.apache.org/docs/3.3.2/reference/#_properties_step) |
| `range` | Filtre la plage de valeurs spécifiée| [étape range](https://tinkerpop.apache.org/docs/3.3.2/reference/#range-step) |
| `repeat` | Répète l’étape le nombre de fois spécifié. Permet d’effectuer des boucles | [répétez l’étape](https://tinkerpop.apache.org/docs/3.3.2/reference/#repeat-step) |
| `sample` | Permet d’échantillonner les résultats à partir de la traversée | [étape sample](https://tinkerpop.apache.org/docs/3.3.2/reference/#sample-step) |
| `select` | Permet de projeter les résultats à partir de la traversée |  [étape select](https://tinkerpop.apache.org/docs/3.3.2/reference/#select-step) |
| `store` | Pour les agrégations non bloquantes de la traversée | [étape store](https://tinkerpop.apache.org/docs/3.3.2/reference/#store-step) |
| `TextP.startingWith(string)` | Fonction de filtrage de chaîne. Cette fonction est utilisée comme prédicat de l'étape `has()` pour faire correspondre une propriété avec le début d'une chaîne donnée | [Prédicats TextP](https://tinkerpop.apache.org/docs/3.4.0/reference/#a-note-on-predicates) |
| `TextP.endingWith(string)` |  Fonction de filtrage de chaîne. Cette fonction est utilisée comme prédicat de l'étape `has()` pour faire correspondre une propriété avec la fin d'une chaîne donnée | [Prédicats TextP](https://tinkerpop.apache.org/docs/3.4.0/reference/#a-note-on-predicates) |
| `TextP.containing(string)` | Fonction de filtrage de chaîne. Cette fonction est utilisée comme prédicat de l'étape `has()` pour faire correspondre une propriété avec le contenu d'une chaîne donnée | [Prédicats TextP](https://tinkerpop.apache.org/docs/3.4.0/reference/#a-note-on-predicates) |
| `TextP.notStartingWith(string)` | Fonction de filtrage de chaîne. Cette fonction est utilisée comme prédicat de l'étape `has()` pour faire correspondre une propriété qui ne commence pas par une chaîne donnée | [Prédicats TextP](https://tinkerpop.apache.org/docs/3.4.0/reference/#a-note-on-predicates) |
| `TextP.notEndingWith(string)` | Fonction de filtrage de chaîne. Cette fonction est utilisée comme prédicat de l'étape `has()` pour faire correspondre une propriété qui ne finit pas par une chaîne donnée | [Prédicats TextP](https://tinkerpop.apache.org/docs/3.4.0/reference/#a-note-on-predicates) |
| `TextP.notContaining(string)` | Fonction de filtrage de chaîne. Cette fonction est utilisée comme prédicat de l'étape `has()` pour faire correspondre une propriété qui ne contient pas une chaîne donnée | [Prédicats TextP](https://tinkerpop.apache.org/docs/3.4.0/reference/#a-note-on-predicates) |
| `tree` | Chemins d’agrégation à partir d’un sommet dans une arborescence | [étape tree](https://tinkerpop.apache.org/docs/3.3.2/reference/#tree-step) |
| `unfold` | Dérouler un itérateur comme une étape| [étape unfold](https://tinkerpop.apache.org/docs/3.3.2/reference/#unfold-step) |
| `union` | Fusionner les résultats à partir de plusieurs traversées| [étape union](https://tinkerpop.apache.org/docs/3.3.2/reference/#union-step) |
| `V` | Inclut les étapes nécessaires pour les traversées entre les sommets et les arêtes `V`, `E`, `out`, `in`, `both`, `outE`, `inE`, `bothE`, `outV`, `inV`, `bothV` et `otherV` pour | [étapes sommet](https://tinkerpop.apache.org/docs/3.3.2/reference/#vertex-steps) |
| `where` | Permet de filtrer les résultats à partir de la traversée. Prend en charge les opérateurs `eq`, `neq`, `lt`, `lte`, `gt`, `gte` et `between`  | [étape where](https://tinkerpop.apache.org/docs/3.3.2/reference/#where-step) |

Le moteur optimisé pour l’écriture fourni par Azure Cosmos DB prend en charge l’indexation automatique de toutes les propriétés au sein des sommets et des arêtes par défaut. Par conséquent, les requêtes avec des filtres, les requêtes de plage, le tri ou les agrégations sur toutes les propriétés sont traités à partir de l’index et exécutés efficacement. Pour plus d’informations sur la façon dont fonctionne l’indexation dans Azure Cosmos DB, consultez notre article sur [l’indexation indépendante du schéma](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf).

## <a name="behavior-differences"></a>Différences de comportement

* Le moteur de graphe Azure Cosmos DB fonctionne en ***largeur d’abord** _ sur la traversée tandis que TinkerPop Gremlin fonctionne en profondeur d’abord. Ce comportement permet d'obtenir de meilleures performances dans un système horizontalement évolutif comme Cosmos DB.

## <a name="unsupported-features"></a>Fonctionnalités non prises en charge

_ * **[Gremlin Bytecode](https://tinkerpop.apache.org/docs/current/tutorials/gremlin-language-variants/)** _ est une spécification pour les traversées de graphe indépendante du langage de programmation. Cosmos DB Graph ne la prend pas encore en charge. Utilisez `GremlinClient.SubmitAsync()` et passez la traversée en tant que chaîne de texte.

_ La cardinalité des ensembles * **`property(set, 'xyz', 1)`** _ n’est actuellement pas prise en charge. Utilisez `property(list, 'xyz', 1)` à la place. Pour plus d’informations, consultez [Vertex properties with TinkerPop](http://tinkerpop.apache.org/docs/current/reference/#vertex-properties) (Propriétés de vertex avec TinkerPop).

_ L’***étape `match()`** _ n’est actuellement pas disponible. Cette étape fournit des fonctions d’interrogation déclarative.

_ Les ***objets en tant que propriétés** _ sur les sommets ou arêtes ne sont pas pris en charge. Les propriétés peuvent uniquement être des types primitifs ou des tableaux.

_ Le ***tri par propriétés de tableau** _ `order().by(<array property>)` n’est pas pris en charge. Seul est pris en charge le tri par types primitifs.

_ Les ***types JSON non primitifs** _ ne sont pas pris en charge. Utilisez les types `string`, `number` ou `true`/`false`. Les valeurs `null` ne sont pas prises en charge. 

_ Le sérialiseur ***GraphSONv3** n’est actuellement pas pris en charge. Utilisez les classes de sérialiseur, de lecteur et d’enregistreur `GraphSONv2` dans la configuration de la connexion. Les résultats renvoyés par l'API Azure Cosmos DB Gremlin ne sont pas au format GraphSON. 

_ Les **fonctions et expressions lambda** ne sont actuellement pas prises en charge. Cela comprend les fonctions `.map{<expression>}`, `.by{<expression>}` et `.filter{<expression>}`. Pour plus d’informations et pour découvrir comment les réécrire à l’aide d’étapes Gremlin, consultez [A Note on Lambdas](http://tinkerpop.apache.org/docs/current/reference/#a-note-on-lambdas) (Remarque sur les lambdas).

* Les ***transactions** _ ne sont pas prises en charge en raison de la nature distribuée du système.  Configurez un modèle d’accès concurrentiel optimiste approprié sur le compte Gremlin pour « lire vos propres écrits », et utilisez l’accès concurrentiel optimiste pour résoudre les conflits d’écritures.

## <a name="known-limitations"></a>Limitations connues

_ **Étapes de l’utilisation de l’index pour les requêtes Gremlin avec `.V()` à mi-parcours** : Actuellement, seul le premier appel `.V()` d’une traversée utilisera l’index pour résoudre les filtres ou prédicats qui lui sont associés. Les appels suivants ne consultent pas l’index, ce qui peut augmenter la latence et le coût de la requête.
    
    Assuming default indexing, a typical read Gremlin query that starts with the `.V()` step would use parameters in its attached filtering steps, such as `.has()` or `.where()` to optimize the cost and performance of the query. For example:

    ```java
    g.V().has('category', 'A')
    ```

    However, when more than one `.V()` step is included in the Gremlin query, the resolution of the data for the query might not be optimal. Take the following query as an example:

    ```java
    g.V().has('category', 'A').as('a').V().has('category', 'B').as('b').select('a', 'b')
    ```

    This query will return two groups of vertices based on their property called `category`. In this case, only the first call, `g.V().has('category', 'A')` will make use of the index to resolve the vertices based on the values of their properties.

    A workaround for this query is to use subtraversal steps such as `.map()` and `union()`. This is exemplified below:

    ```java
    // Query workaround using .map()
    g.V().has('category', 'A').as('a').map(__.V().has('category', 'B')).as('b').select('a','b')

    // Query workaround using .union()
    g.V().has('category', 'A').fold().union(unfold(), __.V().has('category', 'B'))
    ```

    You can review the performance of the queries by using the [Gremlin `executionProfile()` step](graph-execution-profile.md).

## <a name="next-steps"></a>Étapes suivantes

* Commencez par créer une application de graphe [à l’aide de nos kits SDK](create-graph-dotnet.md) 
* Découvrez plus en détail la [prise en charge des graphiques](graph-introduction.md) dans Azure Cosmos DB.
