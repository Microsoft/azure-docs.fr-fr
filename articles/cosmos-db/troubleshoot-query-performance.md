---
title: Résoudre des problèmes de requête lors de l’utilisation d’Azure Cosmos DB
description: Apprenez à identifier, diagnostiquer et résoudre les problèmes de requête SQL Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: troubleshooting
ms.date: 10/12/2020
ms.author: tisande
ms.subservice: cosmosdb-sql
ms.reviewer: sngun
ms.openlocfilehash: 42f01b140a44d7aa6d75dece9a4398fd7b41bf5a
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905109"
---
# <a name="troubleshoot-query-issues-when-using-azure-cosmos-db"></a>Résoudre des problèmes de requête lors de l’utilisation d’Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article décrit une approche générale recommandée pour le dépannage des requêtes dans Azure Cosmos DB. Bien que vous ne devriez pas considérer les étapes décrites dans cet article comme une protection complète contre les problèmes de requête potentiels, nous avons inclus ici les conseils sur les performances les plus courants. Vous devez utiliser cet article comme point de départ pour le dépannage des requêtes lentes ou coûteuses dans l’API Core (SQL) d’Azure Cosmos DB. Vous pouvez également utiliser des [journaux de diagnostic](cosmosdb-monitor-resource-logs.md) pour identifier les requêtes qui sont lentes ou qui consomment un débit significatif. Si vous utilisez l’API d’Azure Cosmos DB pour MongoDB, vous devez utiliser le [guide de résolution des problèmes de requête relatif à l’API d’Azure Cosmos DB pour MongoDB](mongodb-troubleshoot-query.md).

Les optimisations de requête dans Azure Cosmos DB sont classées de manière générale comme suit :

- celles qui réduisent les frais en unités de requête (RU) de la requête
- Optimisations qui réduisent simplement la latence

Si vous réduisez les frais en RU d’une requête, vous diminuez généralement aussi la latence.

Cet article fournit des exemples que vous pouvez recréer à l’aide du [jeu de données nutrition](https://github.com/CosmosDB/labs/blob/master/dotnet/setup/NutritionData.json).

## <a name="common-sdk-issues"></a>Problèmes courants du Kit de développement logiciel (SDK)

Avant de lire ce guide, il est utile de prendre en compte les problèmes courants liés au Kit de développement logiciel (SDK) et qui ne sont pas liés au moteur de requête.

- Suivez les [conseils relatifs aux performances du kit SDK](performance-tips.md).
    - [Guide de résolution des problèmes du kit SDK .NET](troubleshoot-dot-net-sdk.md)
    - [Guide de résolution des problèmes du kit SDK Java](troubleshoot-java-sdk-v4-sql.md)
- Le Kit de développement logiciel (SDK) permet de définir un paramètre `MaxItemCount` pour vos requêtes, mais vous ne pouvez pas spécifier un nombre minimal d’éléments.
    - Le code doit gérer toute taille de page comprise entre zéro et le `MaxItemCount`.
- Parfois, les requêtes peuvent avoir des pages vides, même si des résultats se trouvent sur une page ultérieure. Les raisons peuvent être les suivantes :
    - Le Kit de développement logiciel (SDK) peut effectuer plusieurs appels réseau.
    - La requête peut prendre beaucoup de temps pour récupérer les documents.
- Toutes les requêtes ont un jeton de continuation qui permet à la requête de continuer. Veillez à vider complètement la requête. En savoir plus sur la [gestion de plusieurs pages de résultats](sql-query-pagination.md#handling-multiple-pages-of-results)

## <a name="get-query-metrics"></a>Obtenir les métriques de requête

Lors de l’optimisation d’une requête dans Azure Cosmos DB, la première étape consiste toujours à [obtenir les métriques de requête](profile-sql-api-query.md) pour votre requête. Ces métriques sont également disponibles par le biais du portail Azure. Une fois que vous avez exécuté votre requête dans l’Explorateur de données, les métriques de requête apparaissent en regard de l’onglet **Résultats** :

:::image type="content" source="./media/troubleshoot-query-performance/obtain-query-metrics.png" alt-text="Obtention de métriques de requête" lightbox="./media/troubleshoot-query-performance/obtain-query-metrics.png":::

Après avoir obtenu les métriques de requête, comparez le **nombre de documents récupérés** au **nombre de documents de sortie** pour votre requête. Utilisez cette comparaison pour identifier les sections pertinentes à vérifier dans cet article.

Le **nombre de documents récupérés** correspond au nombre de documents que le moteur de requête a dû charger. Le **nombre de documents de sortie** correspond au nombre de documents qui ont été nécessaires pour les résultats de la requête. Si le **nombre de documents récupérés** est beaucoup plus élevé que le **nombre de documents de sortie**, il y a au moins une partie de votre requête qui n’a pas pu utiliser d’index et qui a dû effectuer une analyse.

Reportez-vous aux sections suivantes pour comprendre les optimisations de requête pertinentes pour votre scénario.

### <a name="querys-ru-charge-is-too-high"></a>Les frais en RU de la requête sont trop élevés

#### <a name="retrieved-document-count-is-significantly-higher-than-output-document-count"></a>Le nombre de documents récupérés est beaucoup plus élevé que le nombre de documents de sortie

- [Incluez les chemins nécessaires dans la stratégie d’indexation.](#include-necessary-paths-in-the-indexing-policy)

- [Comprenez quelles fonctions système utilisent l’index.](#understand-which-system-functions-use-the-index)

- [Identifiez les requêtes d’agrégation qui utilisent l’index.](#understand-which-aggregate-queries-use-the-index)

- [Optimisez les requêtes qui ont à la fois un filtre et une clause ORDER BY.](#optimize-queries-that-have-both-a-filter-and-an-order-by-clause)

- [Optimisez les expressions JOIN à l’aide d’une sous-requête.](#optimize-join-expressions-by-using-a-subquery)

<br>

#### <a name="retrieved-document-count-is-approximately-equal-to-output-document-count"></a>Le nombre de documents récupérés est à peu près égal au nombre de documents de sortie

- [Réduisez les requêtes entre les partitions.](#minimize-cross-partition-queries)

- [Optimisez les requêtes qui ont des filtres sur plusieurs propriétés.](#optimize-queries-that-have-filters-on-multiple-properties)

- [Optimisez les requêtes qui ont à la fois un filtre et une clause ORDER BY.](#optimize-queries-that-have-both-a-filter-and-an-order-by-clause)

<br>

### <a name="querys-ru-charge-is-acceptable-but-latency-is-still-too-high"></a>Les frais en RU de la requête sont acceptables, mais la latence est encore trop élevée

- [Améliorez la proximité.](#improve-proximity)

- [Augmentez le débit provisionné.](#increase-provisioned-throughput)

- [Augmentez MaxConcurrency.](#increase-maxconcurrency)

- [Augmentez MaxBufferedItemCount.](#increase-maxbuffereditemcount)

## <a name="queries-where-retrieved-document-count-exceeds-output-document-count"></a>Requêtes où le nombre de documents récupérés dépasse le nombre de documents de sortie

 Le **nombre de documents récupérés** correspond au nombre de documents que le moteur de requête a dû charger. Le **nombre de documents de sortie** correspond au nombre de documents retournés par la requête. Si le **nombre de documents récupérés** est beaucoup plus élevé que le **nombre de documents de sortie**, il y a au moins une partie de votre requête qui n’a pas pu utiliser d’index et qui a dû effectuer une analyse.

Voici un exemple de requête d’analyse qui n’a pas été entièrement servie par l’index :

Requête :

 ```sql
SELECT VALUE c.description
FROM c
WHERE UPPER(c.description) = "BABYFOOD, DESSERT, FRUIT DESSERT, WITHOUT ASCORBIC ACID, JUNIOR"
 ```

Métriques de requête :

```
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

Le **nombre de documents récupérés** (60 951) est beaucoup plus élevé que le **nombre de documents de sortie** (7), indiquant que cette requête a entraîné une analyse de document. Dans ce cas, la fonction système [UPPER ()](sql-query-upper.md) n’utilise pas d’index.

### <a name="include-necessary-paths-in-the-indexing-policy"></a>Inclure les chemins nécessaires dans la stratégie d’indexation

Votre stratégie d’indexation doit couvrir toutes les propriétés incluses dans les clauses `WHERE`, les clauses `ORDER BY`, `JOIN` et la plupart des fonctions système. Les chemins souhaités spécifiés dans la stratégie d’index doivent correspondre aux propriétés dans les documents JSON.

> [!NOTE]
> Les propriétés de la stratégie d’indexation Azure Cosmos DB sont sensibles à la casse

Si vous exécutez la requête simple suivante sur le jeu de données [nutrition](https://github.com/CosmosDB/labs/blob/master/dotnet/setup/NutritionData.json), vous constaterez que les frais en RU sont plus faibles quand la propriété de la clause `WHERE` est indexée :

#### <a name="original"></a>Original

Requête :

```sql
SELECT *
FROM c
WHERE c.description = "Malabar spinach, cooked"
```

Stratégie d’indexation :

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/description/*"
        }
    ]
}
```

**Frais en RU :** 409.51 RU

#### <a name="optimized"></a>Optimisée

Stratégie d’indexation mise à jour :

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": []
}
```

**Frais en RU :** 2.98 RU

Vous pouvez ajouter des propriétés à la stratégie d’indexation à tout moment, sans impact sur la disponibilité en lecture ou en écriture. Vous pouvez [suivre la progression de la transformation d’index](./how-to-manage-indexing-policy.md#dotnet-sdk).

### <a name="understand-which-system-functions-use-the-index"></a>Comprendre quelles fonctions système utilisent l’index

La plupart des fonctions système utilisent des index. Voici une liste de certaines fonctions de chaîne courantes qui utilisent des index :

- STARTSWITH(str_expr1, str_expr2, bool_expr)  
- CONTAINS(str_expr, str_expr, bool_expr)
- LEFT(str_expr, num_expr) = str_expr
- SUBSTRING(str_expr, num_expr, num_expr) = str_expr, mais seulement si la première num_expr est 0

Voici quelques fonctions système courantes qui n’utilisent pas l’index et doivent charger chaque document :

| **Fonction système**                     | **Idées pour l’optimisation**             |
| --------------------------------------- |------------------------------------------------------------ |
| UPPER/LOWER                             | Au lieu d’utiliser la fonction système pour normaliser les données pour les comparaisons, normalisez la casse lors de l’insertion. Une requête telle que ```SELECT * FROM c WHERE UPPER(c.name) = 'BOB'``` devient ```SELECT * FROM c WHERE c.name = 'BOB'```. |
| Fonctions mathématiques (non-agrégations) | Si vous devez calculer fréquemment une valeur dans votre requête, stockez cette valeur en tant que propriété dans votre document JSON. |

------

Si une fonction système utilise des index et qu’elle a toujours des frais en RU (unités de requête) élevés, vous pouvez essayer d’ajouter `ORDER BY` à la requête. Dans certains cas, l’ajout de `ORDER BY` peut améliorer l’utilisation d’index de fonction système, en particulier si la requête est à exécution longue ou s’étend sur plusieurs pages.

Prenons l’exemple de la requête ci-dessous avec `CONTAINS`. `CONTAINS` doit utiliser un index, mais supposons qu’après avoir ajouté l’index approprié, vous observez toujours des frais en RU (unités de requête) très élevés lors de l’exécution de la requête ci-dessous :

Requête d’origine :

```sql
SELECT *
FROM c
WHERE CONTAINS(c.town, "Sea")
```

Requête mise à jour avec `ORDER BY` :

```sql
SELECT *
FROM c
WHERE CONTAINS(c.town, "Sea")
ORDER BY c.town
```

### <a name="understand-which-aggregate-queries-use-the-index"></a>Identifier les requêtes d’agrégation qui utilisent l’index

Dans la plupart des cas, les fonctions système d’agrégation dans Azure Cosmos DB utilisent l’index. Toutefois, en fonction des filtres ou des clauses supplémentaires dans une requête d’agrégation, le moteur d’interrogation peut être amené à charger un grand nombre de documents. En règle générale, le moteur d’interrogation applique d’abord les filtres d’égalité et de plage. Après l’application de ces filtres, le moteur d’interrogation peut évaluer des filtres supplémentaires et recourir au chargement des documents restants pour calculer l’agrégat, le cas échéant.

Par exemple, pour ces deux exemples de requêtes, la requête avec les filtres de fonction système d’égalité et `CONTAINS` est généralement plus efficace qu’une requête avec simplement un filtre de fonction système `CONTAINS`. Cela est dû au fait que le filtre d’égalité est appliqué en premier et utilise l’index avant le chargement des documents pour le filtre `CONTAINS`, plus onéreux.

Requête avec le filtre `CONTAINS` seulement (frais RU plus élevés) :

```sql
SELECT COUNT(1)
FROM c
WHERE CONTAINS(c.description, "spinach")
```

Requête avec le filtre d’égalité et le filtre `CONTAINS` (frais RU réduits) :

```sql
SELECT AVG(c._ts)
FROM c
WHERE c.foodGroup = "Sausages and Luncheon Meats" AND CONTAINS(c.description, "spinach")
```

Voici des exemples supplémentaires de requêtes d’agrégation qui n’utiliseront pas pleinement l’index :

#### <a name="queries-with-system-functions-that-dont-use-the-index"></a>Requêtes avec des fonctions système qui n’utilisent pas l’index

Vous devez consulter la [page de la fonction système](sql-query-system-functions.md) correspondante pour voir si elle utilise l’index.

```sql
SELECT MAX(c._ts)
FROM c
WHERE CONTAINS(c.description, "spinach")
```

#### <a name="aggregate-queries-with-user-defined-functionsudfs"></a>Requêtes d’agrégation avec des fonctions définies par l’utilisateur (UDF)

```sql
SELECT AVG(c._ts)
FROM c
WHERE udf.MyUDF("Sausages and Luncheon Meats")
```

#### <a name="queries-with-group-by"></a>Requêtes avec GROUP BY

Les frais RU de requêtes avec `GROUP BY` augmenteront à mesure que la cardinalité des propriétés de la clause `GROUP BY` augmente. Dans la requête ci-dessous, par exemple, les frais RU de la requête augmentent à mesure que le nombre de descriptions uniques augmente.

Les frais RU d’une fonction d’agrégation avec une clause `GROUP BY` seront supérieurs à ceux d’une fonction d’agrégation seule. Dans cet exemple, le moteur d’interrogation doit charger chaque document qui correspond au filtre `c.foodGroup = "Sausages and Luncheon Meats"`, de sorte que les frais RU devraient être élevés.

```sql
SELECT COUNT(1)
FROM c
WHERE c.foodGroup = "Sausages and Luncheon Meats"
GROUP BY c.description
```

Si vous prévoyez d’exécuter fréquemment les mêmes requêtes d’agrégation, il peut être plus efficace de créer un affichage matérialisé en temps réel avec le [flux de modification Azure Cosmos DB](change-feed.md) que d’exécuter des requêtes individuelles.

### <a name="optimize-queries-that-have-both-a-filter-and-an-order-by-clause"></a>Optimiser les requêtes qui ont à la fois un filtre et une clause ORDER BY

Les requêtes avec un filtre et une clause `ORDER BY` utilisent normalement un index de plage, mais elles sont plus efficaces si elles peuvent être servies à partir d’un index composite. En plus de modifier la stratégie d’indexation, vous devez ajouter toutes les propriétés de l’index composite à la clause `ORDER BY`. Cette modification de la requête permet de s’assurer qu’elle utilise l’index composite.  Vous pouvez observer l’impact en exécutant une requête sur le jeu de données [nutrition](https://github.com/CosmosDB/labs/blob/master/dotnet/setup/NutritionData.json) :

#### <a name="original"></a>Original

Requête :

```sql
SELECT *
FROM c
WHERE c.foodGroup = "Soups, Sauces, and Gravies"
ORDER BY c._ts ASC
```

Stratégie d’indexation :

```json
{

        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[]
}
```

**Frais en RU :** 44.28 RU

#### <a name="optimized"></a>Optimisée

Requête mise à jour (comprend les deux propriétés dans la clause `ORDER BY`) :

```sql
SELECT *
FROM c
WHERE c.foodGroup = "Soups, Sauces, and Gravies"
ORDER BY c.foodGroup, c._ts ASC
```

Stratégie d’indexation mise à jour :

```json
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/foodGroup",
                    "order":"ascending"
        },
                {  
                    "path":"/_ts",
                    "order":"ascending"
                }
            ]
        ]
    }

```

**Frais en RU :** 8.86 RU

### <a name="optimize-join-expressions-by-using-a-subquery"></a>Optimiser les expressions JOIN à l’aide d’une sous-requête

Les sous-requêtes multivaleurs peuvent optimiser des expressions `JOIN` en envoyant les prédicats après chaque expression de sélection multiple, plutôt qu’après toutes les jointures croisées dans la clause `WHERE`.

Prenons par exemple la requête suivante :

```sql
SELECT Count(1) AS Count
FROM c
JOIN t IN c.tags
JOIN n IN c.nutrients
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND (n.nutritionValue > 0
AND n.nutritionValue < 10) AND s.amount > 1
```

**Frais en RU :** 167.62 RU

Pour cette requête, l’index correspond à n’importe quel document qui a une étiquette portant le nom `infant formula`, avec `nutritionValue` supérieure à 0, et `amount` supérieure à 1. Dans ce cas, l’expression `JOIN` effectue le produit croisé de tous les éléments des tableaux tags, nutrients et servings pour chaque document correspondant avant l’application de tout filtre. Ensuite, la clause `WHERE` applique le prédicat de filtre sur chaque tuple `<c, t, n, s>`.

Par exemple, si un document correspondant comporte 10 éléments dans chacun des trois tableaux, l’opération aboutit à 1 x 10 x 10 x 10 (autrement dit, 1 000) tuples. L’utilisation de sous-requêtes ici peut aider à filtrer des éléments de tableaux joints avant d’effectuer une jointure avec l’expression suivante.

Cette requête est équivalente à la précédente, mais utilise des sous-requêtes :

```sql
SELECT Count(1) AS Count
FROM c
JOIN (SELECT VALUE t FROM t IN c.tags WHERE t.name = 'infant formula')
JOIN (SELECT VALUE n FROM n IN c.nutrients WHERE n.nutritionValue > 0 AND n.nutritionValue < 10)
JOIN (SELECT VALUE s FROM s IN c.servings WHERE s.amount > 1)
```

**Frais en RU :** 22.17 RU

Supposons qu’un seul élément du tableau tags correspond au filtre, et qu’il existe cinq éléments pour les tableaux nutrients et servings. Les expressions `JOIN` aboutissent à 1 x 1 x 5 x 5 = 25 éléments, au lieu de 1000 éléments dans la première requête.

## <a name="queries-where-retrieved-document-count-is-equal-to-output-document-count"></a>Requêtes où le nombre de documents récupérés est égal au nombre de documents de sortie

Si le **nombre de documents récupérés** est à peu près égal au **nombre de documents de sortie**, le moteur de requête n’a pas eu à analyser de nombreux documents inutiles. Pour de nombreuses requêtes, telles que celles qui utilisent le mot clé `TOP`, le **nombre de documents récupérés** peut dépasser le **nombre de documents de sortie** de 1. Vous n’avez pas besoin de vous en préoccuper.

### <a name="minimize-cross-partition-queries"></a>Réduire les requêtes entre les partitions

Azure Cosmos DB utilise le [partitionnement](partitioning-overview.md) pour mettre à l’échelle des conteneurs en fonction de l’augmentation des besoins en unités de requête et en stockage des données. Chaque partition physique a un index distinct et indépendant. Si votre requête a un filtre d’égalité qui correspond à la clé de partition de votre conteneur, vous devrez uniquement vérifier l’index de la partition pertinente. Cette optimisation réduit le nombre total de RU requis par la requête.

Si vous disposez d’un grand nombre d’unités de requête provisionnées (plus de 30 000) ou d’une grande quantité de données stockées (plus de 100 Go environ), vous disposez probablement d’un conteneur suffisamment grand pour constater une réduction significative des frais en RU liés aux requêtes.

Par exemple, si vous créez un conteneur avec la clé de partition foodGroup, les requêtes suivantes ne devront vérifier qu’une seule partition physique :

```sql
SELECT *
FROM c
WHERE c.foodGroup = "Soups, Sauces, and Gravies" and c.description = "Mushroom, oyster, raw"
```

Les requêtes qui ont un filtre `IN` avec la clé de partition ne vérifient que les partitions physiques pertinentes et n’effectuent pas de distribution ramifiée (fan-out) :

```sql
SELECT *
FROM c
WHERE c.foodGroup IN("Soups, Sauces, and Gravies", "Vegetables and Vegetable Products") and c.description = "Mushroom, oyster, raw"
```

Les requêtes qui ont des filtres de plage sur la clé de partition ou qui n’ont aucun filtre sur la clé de partition devront vérifier l’index de chaque partition physique pour obtenir les résultats :

```sql
SELECT *
FROM c
WHERE c.description = "Mushroom, oyster, raw"
```

```sql
SELECT *
FROM c
WHERE c.foodGroup > "Soups, Sauces, and Gravies" and c.description = "Mushroom, oyster, raw"
```

### <a name="optimize-queries-that-have-filters-on-multiple-properties"></a>Optimiser les requêtes qui ont des filtres sur plusieurs propriétés

Les requêtes avec des filtres sur plusieurs propriétés utilisent normalement un index de plage, mais elles sont plus efficaces si elles peuvent être servies à partir d’un index composite. Pour les petites quantités de données, cette optimisation n’aura pas d’impact significatif. Toutefois, elle peut s’avérer utile pour les grandes quantités de données. Vous pouvez uniquement optimiser, au plus, un filtre de non-égalité par index composite. Si votre requête comporte plusieurs filtres de non-égalité, choisissez l’un d’entre eux qui utilisera l’index composite. Le reste va continuer à utiliser des index de plage. Le filtre de non-égalité doit être défini en dernier dans l’index composite. [En savoir plus sur les index composites](index-policy.md#composite-indexes).

Voici quelques exemples de requêtes qui peuvent être optimisées avec un index composite :

```sql
SELECT *
FROM c
WHERE c.foodGroup = "Vegetables and Vegetable Products" AND c._ts = 1575503264
```

```sql
SELECT *
FROM c
WHERE c.foodGroup = "Vegetables and Vegetable Products" AND c._ts > 1575503264
```

Voici l’index composite approprié :

```json
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/foodGroup",
                    "order":"ascending"
                },
                {  
                    "path":"/_ts",
                    "order":"ascending"
                }
            ]
        ]
}
```

## <a name="optimizations-that-reduce-query-latency"></a>Optimisations qui réduisent la latence des requêtes

Dans de nombreux cas, les frais en RU peuvent être acceptables, même si la latence des requêtes est encore trop élevée. Les sections suivantes présentent différents conseils pour réduire la latence des requêtes. Si vous exécutez la même requête plusieurs fois sur le même jeu de données, elle aura généralement les mêmes frais de RU à chaque fois. Mais la latence des requêtes peut varier d’une exécution de requête à une autre.

### <a name="improve-proximity"></a>Améliorer la proximité

Les requêtes exécutées à partir d’une région différente de celle du compte Azure Cosmos DB auront une latence plus élevée que si elles étaient exécutées dans la même région. Par exemple, si vous exécutez du code sur votre ordinateur de bureau, vous devez vous attendre à une latence supérieure de quelques dizaines ou centaines de millisecondes (ou plus) à la même requête provenant d’une machine virtuelle située dans la même région Azure qu’Azure Cosmos DB. Il est facile de [distribuer les données globalement dans Azure Cosmos DB](distribute-data-globally.md) pour vous assurer de pouvoir rapprocher vos données de votre application.

### <a name="increase-provisioned-throughput"></a>Augmenter le débit provisionné

Dans Azure Cosmos DB, votre débit provisionné est mesuré en unités de requête (RU). Supposez que vous avez une requête qui consomme 5 RU de débit. Par exemple, si vous provisionnez 1 000 RU, vous pouvez exécuter cette requête 200 fois par seconde. Si vous avez essayé d’exécuter la requête alors que le débit disponible est insuffisant, Azure Cosmos DB retourne une erreur HTTP 429. Les kits SDK actuels de l’API Core (SQL) réessaient automatiquement d’exécuter cette requête après un bref délai. Les requêtes limitées prennent plus de temps ; par conséquent, l’augmentation du débit provisionné peut améliorer la latence des requêtes. Vous pouvez observer le [nombre total de requêtes limitées](use-metrics.md#understand-how-many-requests-are-succeeding-or-causing-errors) dans le panneau **Métriques** du portail Azure.

### <a name="increase-maxconcurrency"></a>Augmenter MaxConcurrency

Les requêtes parallèles interrogent plusieurs partitions en parallèle. Les données d’une collection partitionnée individuelle sont toutefois extraites en série dans le cadre de la requête. Par conséquent, si vous réglez de MaxConcurrency sur le nombre de partitions, cela vous donnera les meilleures chances de résultats pour la requête, sous réserve que toutes les autres conditions système restent inchangées. Si vous ne connaissez pas le nombre de partitions, vous pouvez définir MaxConcurrency (ou MaxDegreesOfParallelism dans les versions antérieures du SDK) sur un nombre élevé. Le système choisit la valeur minimale (nombre de partitions, entrée fournie par l’utilisateur) comme degré maximal de parallélisme.

### <a name="increase-maxbuffereditemcount"></a>Augmenter MaxBufferedItemCount

Les requêtes sont conçues pour pré-extraire les résultats pendant que le lot de résultats actuel est en cours de traitement par le client. La pré-récupération permet d’améliorer la latence globale d’une requête. La définition de setMaxBufferedItemCount limite le nombre de résultats pré-extraits. Pour que la requête tire le maximum de bénéfices de la pré-extraction, affectez à cette valeur le nombre de résultats attendus (ou un nombre plus élevé). Si vous définissez cette valeur sur-1, le système détermine automatiquement le nombre d’éléments à mettre en mémoire tampon.

## <a name="next-steps"></a>Étapes suivantes
Reportez-vous aux articles suivants pour des informations sur la mesure des unités de requête par requête et pour obtenir des statistiques d’exécution afin d’optimiser vos requêtes, entre autres :

* [Obtenir des métriques sur l’exécution des requêtes SQL à l’aide du SDK .NET](profile-sql-api-query.md)
* [Réglage des performances de requête avec Azure Cosmos DB](./sql-api-query-metrics.md)
* [Conseils en matière de performances pour le Kit de développement logiciel (SDK) .NET](performance-tips.md)
* [Conseils en matière de performances pour le Kit de développement logiciel (SDK) Java v4](performance-tips-java-sdk-v4-sql.md)