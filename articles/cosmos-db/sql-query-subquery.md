---
title: Sous-requêtes SQL pour Azure Cosmos DB
description: Découvrez les sous-requêtes SQL et leurs cas d’utilisation courants et les différents types de sous-requêtes dans Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: tisande
ms.openlocfilehash: f5f209229d17a2587258d21ee90e7560e629d082
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340853"
---
# <a name="sql-subquery-examples-for-azure-cosmos-db"></a>Exemples de sous-requêtes SQL pour Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Une sous-requête est une requête imbriquée dans une autre requête. Une sous-requête est également appelée « requête interne » ou « sélection interne ». L’instruction qui contient une sous-requête est généralement appelée « requête externe ».

Cet article décrit les sous-requêtes SQL et leurs cas d’utilisation courants dans Azure Cosmos DB. Tous les exemples de requêtes dans ce document peuvent être exécutés sur un jeu de données d’alimentation qui est préchargé sur le [Playground de test de requêtes Azure Cosmos DB](https://www.documentdb.com/sql/demo).

## <a name="types-of-subqueries"></a>Types des sous-requêtes

Il existe deux principaux types de sous-requêtes :

* **Corrélée**  : sous-requête qui référence des valeurs de la requête externe. La sous-requête est évaluée une fois pour chaque ligne que traite la requête externe.
* **Non corrélée**  : sous-requête indépendante de la requête externe. Elle peut être exécutée indépendamment de la requête externe.

> [!NOTE]
> Azure Cosmos DB prend en charge uniquement les sous-requêtes corrélées.

Les sous-requêtes peuvent également être classées en fonction du nombre de lignes et colonnes qu’elles retournent. Il existe trois types :
* **Table** : retourne plusieurs lignes et plusieurs colonnes.
* **Multivaleur**  : retourne plusieurs lignes et une seule colonne.
* **Scalaire**  : retourne une seule ligne et une seule colonne.

Les requêtes SQL dans Azure Cosmos DB retournent toujours une seule colonne (une valeur simple ou un document complexe). Ainsi, seules les sous-requêtes scalaires et multivaleurs sont applicables dans Azure Cosmos DB. Vous pouvez utiliser une sous-requête multivaleur uniquement dans la clause FROM sous forme d’expression relationnelle. Vous pouvez utiliser une sous-requête scalaire comme expression scalaire dans la clause SELECT ou WHERE ou comme expression relationnelle dans la clause FROM.

## <a name="multi-value-subqueries"></a>Sous-requêtes multivaleurs

Les sous-requêtes multivaleurs retournent un ensemble de documents et sont toujours utilisées dans la clause FROM. Elles sont utilisées pour :

* Optimiser les expressions JOIN. 
* Évaluer les expressions coûteuses une seule fois et les référencer plusieurs fois.

## <a name="optimize-join-expressions"></a>Optimiser les expressions JOIN

Les sous-requêtes multivaleurs peuvent optimiser des expressions JOIN en envoyant les prédicats après chaque expression de sélection multiple, plutôt qu’après toutes les jointures croisées dans la clause WHERE.

Considérez la requête suivante :

```sql
SELECT Count(1) AS Count
FROM c
JOIN t IN c.tags
JOIN n IN c.nutrients
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND (n.nutritionValue > 0 
AND n.nutritionValue < 10) AND s.amount > 1
```

Pour cette requête, l’index correspond à n’importe quel document qui a une étiquette portant le nom « infant formula ». Cela correspond à un élément nutritif dont la valeur est comprise entre 0 et 10 et à un élément de portion dont la quantité est supérieure à 1. Dans ce cas, l’expression JOIN effectue le produit croisé de tous les éléments des tableaux tags, nutrients et servings pour chaque document correspondant avant l’application de tout filtre. 

Ensuite, la clause WHERE applique le prédicat de filtre sur chaque tuple <c, t, n, s>. Par exemple, si un document correspondant comporte 10 éléments dans chacun des trois tableaux, l’opération aboutit à 1 x 10 x 10 x 10 (autrement dit, 1 000) tuples. L’utilisation de sous-requêtes ici peut aider à filtrer des éléments de tableaux joints avant d’effectuer une jointure avec l’expression suivante.

Cette requête est équivalente à la précédente, mais utilise des sous-requêtes :

```sql
SELECT Count(1) AS Count
FROM c
JOIN (SELECT VALUE t FROM t IN c.tags WHERE t.name = 'infant formula')
JOIN (SELECT VALUE n FROM n IN c.nutrients WHERE n.nutritionValue > 0 AND n.nutritionValue < 10)
JOIN (SELECT VALUE s FROM s IN c.servings WHERE s.amount > 1)
```

Supposons qu’un seul élément du tableau tags correspond au filtre, et qu’il existe cinq éléments pour les tableaux nutrients et servings. Les expressions JOIN aboutissent à 1 x 1 x 5 x 5 = 25 éléments, au lieu de 1 000 éléments dans la première requête.

## <a name="evaluate-once-and-reference-many-times"></a>Évaluer une seule fois et référencer plusieurs fois

Les sous-requêtes peuvent aider à optimiser les requêtes ayant des expressions coûteuses telles que des fonctions définies par l’utilisateur, des chaînes complexes ou des expressions arithmétiques. Vous pouvez utiliser une sous-requête avec une expression JOIN pour évaluer l’expression une seule fois, mais la référencer plusieurs fois.

La requête suivante exécute la fonction définie par l’utilisateur `GetMaxNutritionValue` deux fois :

```sql
SELECT c.id, udf.GetMaxNutritionValue(c.nutrients) AS MaxNutritionValue
FROM c
WHERE udf.GetMaxNutritionValue(c.nutrients) > 100
```

Voici une requête équivalente qui exécute la fonction définie par l’utilisateur une seule fois :

```sql
SELECT TOP 1000 c.id, MaxNutritionValue
FROM c
JOIN (SELECT VALUE udf.GetMaxNutritionValue(c.nutrients)) MaxNutritionValue
WHERE MaxNutritionValue > 100
``` 

> [!NOTE] 
> Gardez à l’esprit que les expressions JOIN effectuent des produits croisés. Si l’expression de la fonction définie par l’utilisateur peut avoir une valeur non définie, vous devez vous assurer que l’expression JOIN produit toujours une seule ligne en retournant un objet à partir de la sous-requête plutôt que la valeur directement.
>

Voici un exemple similaire qui retourne un objet, plutôt qu’une valeur :

```sql
SELECT TOP 1000 c.id, m.MaxNutritionValue
FROM c
JOIN (SELECT udf.GetMaxNutritionValue(c.nutrients) AS MaxNutritionValue) m
WHERE m.MaxNutritionValue > 100
```

L’approche n’est pas limitée aux fonctions définies par l’utilisateur. Elle s’applique à toute expression potentiellement coûteuse. Par exemple, vous pouvez adopter la même approche avec la fonction mathématique `avg` :

```sql
SELECT TOP 1000 c.id, AvgNutritionValue
FROM c
JOIN (SELECT VALUE avg(n.nutritionValue) FROM n IN c.nutrients) AvgNutritionValue
WHERE AvgNutritionValue > 80
```

## <a name="mimic-join-with-external-reference-data"></a>Imiter la jointure avec des données de référence externes

Souvent, vous pouvez être amené à référencer des données statiques qui changent rarement, telles que les unités de mesure ou les codes de pays. Il est préférable de ne pas dupliquer ces données pour chaque document. Le fait d’éviter cette duplication permet d’économiser l’espace de stockage et d’améliorer les performances des écritures en réduisant la taille des documents. Vous pouvez utiliser une sous-requête pour imiter la sémantique d’une jointure interne avec une collection de données de référence.

Par exemple, considérez ce jeu de données de référence :

| **Unité** | **Nom**            | **Multiplicateur** | **Unité de base** |
| -------- | ------------------- | -------------- | ------------- |
| ng       | Nanogram            | 1,00E-09       | Gramme          |
| µg       | Microgram           | 1,00E-06       | Gramme          |
| mg       | Milligram           | 1,00E-03       | Gramme          |
| g        | Gramme                | 1,00E+00       | Gramme          |
| kg       | Kilogram            | 1,00E+03       | Gramme          |
| Mg       | Megagram            | 1,00E+06       | Gramme          |
| Gg       | Gigagram            | 1,00E+09       | Gramme          |
| nJ       | Nanojoule           | 1,00E-09       | Joule         |
| µJ       | Microjoule          | 1,00E-06       | Joule         |
| mJ       | Millijoule          | 1,00E-03       | Joule         |
| J        | Joule               | 1,00E+00       | Joule         |
| kJ       | Kilojoule           | 1,00E+03       | Joule         |
| MJ       | Megajoule           | 1,00E+06       | Joule         |
| GJ       | Gigajoule           | 1,00E+09       | Joule         |
| cal      | Calorie             | 1,00E+00       | calorie       |
| kcal     | Calorie             | 1,00E+03       | calorie       |
| IU       | International units |                |               |


La requête suivante imite une jointure avec ces données afin que le nom de l’unité soit ajouté à la sortie :

```sql
SELECT TOP 10 n.id, n.description, n.nutritionValue, n.units, r.name
FROM food
JOIN n IN food.nutrients
JOIN r IN (
    SELECT VALUE [
        {unit: 'ng', name: 'nanogram', multiplier: 0.000000001, baseUnit: 'gram'},
        {unit: 'µg', name: 'microgram', multiplier: 0.000001, baseUnit: 'gram'},
        {unit: 'mg', name: 'milligram', multiplier: 0.001, baseUnit: 'gram'},
        {unit: 'g', name: 'gram', multiplier: 1, baseUnit: 'gram'},
        {unit: 'kg', name: 'kilogram', multiplier: 1000, baseUnit: 'gram'},
        {unit: 'Mg', name: 'megagram', multiplier: 1000000, baseUnit: 'gram'},
        {unit: 'Gg', name: 'gigagram', multiplier: 1000000000, baseUnit: 'gram'},
        {unit: 'nJ', name: 'nanojoule', multiplier: 0.000000001, baseUnit: 'joule'},
        {unit: 'µJ', name: 'microjoule', multiplier: 0.000001, baseUnit: 'joule'},
        {unit: 'mJ', name: 'millijoule', multiplier: 0.001, baseUnit: 'joule'},
        {unit: 'J', name: 'joule', multiplier: 1, baseUnit: 'joule'},
        {unit: 'kJ', name: 'kilojoule', multiplier: 1000, baseUnit: 'joule'},
        {unit: 'MJ', name: 'megajoule', multiplier: 1000000, baseUnit: 'joule'},
        {unit: 'GJ', name: 'gigajoule', multiplier: 1000000000, baseUnit: 'joule'},
        {unit: 'cal', name: 'calorie', multiplier: 1, baseUnit: 'calorie'},
        {unit: 'kcal', name: 'Calorie', multiplier: 1000, baseUnit: 'calorie'},
        {unit: 'IU', name: 'International units'}
    ]
)
WHERE n.units = r.unit
```

## <a name="scalar-subqueries"></a>Sous-requêtes scalaires

Une expression de sous-requête scalaire est une sous-requête qui aboutit à une valeur unique. La valeur de l’expression de sous-requête scalaire est la valeur de la projection (clause SELECT) de la sous-requête.  Vous pouvez utiliser une expression de sous-requête scalaire dans de nombreux endroits où une expression scalaire est valide. Par exemple, vous pouvez utiliser une sous-requête scalaire dans toute expression au sein des clauses SELECT et WHERE.

Toutefois, l’utilisation d’une sous-requête scalaire n’est pas systématiquement un gage d’optimisation. Par exemple, le fait de passer une sous-requête scalaire en tant qu’argument à des fonctions définies par l’utilisateur ou le système n’offre aucun avantage en termes de latence ou de consommation d’unités de ressource.

Les sous-requêtes scalaires peuvent être classées comme suit :
* Sous-requêtes scalaires à expression simple
* Sous-requêtes scalaires d’agrégation

## <a name="simple-expression-scalar-subqueries"></a>Sous-requêtes scalaires à expression simple

Une sous-requête scalaire à expression simple est une sous-requête corrélée qui a une clause SELECT dépourvue d’expressions d’agrégation. Ces sous-requêtes n’apportent aucun avantage en termes d’optimisation, car le compilateur les convertit en une expression simple plus grande. Il n’existe aucun contexte corrélé entre les requêtes internes et externes.

Voici quelques exemples :

**Exemple 1**

```sql
SELECT 1 AS a, 2 AS b
```

Vous pouvez réécrire cette requête, à l’aide d’une sous-requête scalaire à expression simple, comme suit :

```sql
SELECT (SELECT VALUE 1) AS a, (SELECT VALUE 2) AS b
```

Les deux requêtes génèrent cette sortie :

```json
[
  { "a": 1, "b": 2 }
]
```

**Exemple 2**

```sql
SELECT TOP 5 Concat('id_', f.id) AS id
FROM food f
```

Vous pouvez réécrire cette requête, à l’aide d’une sous-requête scalaire à expression simple, comme suit :

```sql
SELECT TOP 5 (SELECT VALUE Concat('id_', f.id)) AS id
FROM food f
```

Sortie de la requête :

```json
[
  { "id": "id_03226" },
  { "id": "id_03227" },
  { "id": "id_03228" },
  { "id": "id_03229" },
  { "id": "id_03230" }
]
```

**Exemple 3**

```sql
SELECT TOP 5 f.id, Contains(f.description, 'fruit') = true ? f.description : undefined
FROM food f
```

Vous pouvez réécrire cette requête, à l’aide d’une sous-requête scalaire à expression simple, comme suit :

```sql
SELECT TOP 10 f.id, (SELECT f.description WHERE Contains(f.description, 'fruit')).description
FROM food f
```

Sortie de la requête :

```json
[
  { "id": "03230" },
  { "id": "03238", "description":"Babyfood, dessert, tropical fruit, junior" },
  { "id": "03229" },
  { "id": "03226", "description":"Babyfood, dessert, fruit pudding, orange, strained" },
  { "id": "03227" }
]
```

### <a name="aggregate-scalar-subqueries"></a>Sous-requêtes scalaires d’agrégation

Une sous-requête scalaire d’agrégation est une sous-requête qui a une fonction d’agrégation dans sa projection ou filtre qui aboutit à une valeur unique.

**Exemple 1 :**

Voici une sous-requête dont la projection contient une expression de fonction d’agrégation unique :

```sql
SELECT TOP 5 
    f.id, 
    (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg'
) AS count_mg
FROM food f
```

Sortie de la requête :

```json
[
  { "id": "03230", "count_mg": 13 },
  { "id": "03238", "count_mg": 14 },
  { "id": "03229", "count_mg": 13 },
  { "id": "03226", "count_mg": 15 },
  { "id": "03227", "count_mg": 19 }
]
```

**Exemple 2**

Voici une sous-requête avec plusieurs expressions de fonction d’agrégation :

```sql
SELECT TOP 5 f.id, (
    SELECT Count(1) AS count, Sum(n.nutritionValue) AS sum 
    FROM n IN f.nutrients 
    WHERE n.units = 'mg'
) AS unit_mg
FROM food f
```

Sortie de la requête :

```json
[
  { "id": "03230","unit_mg": { "count": 13,"sum": 147.072 } },
  { "id": "03238","unit_mg": { "count": 14,"sum": 107.385 } },
  { "id": "03229","unit_mg": { "count": 13,"sum": 141.579 } },
  { "id": "03226","unit_mg": { "count": 15,"sum": 183.91399999999996 } },
  { "id": "03227","unit_mg": { "count": 19,"sum": 94.788999999999987 } }
]
```

**Exemple 3**

Voici une requête avec une sous-requête d’agrégation dans la projection et le filtre :

```sql
SELECT TOP 5 
    f.id, 
    (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg') AS count_mg
FROM food f
WHERE (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg') > 20
```

Sortie de la requête :

```json
[
  { "id": "03235", "count_mg": 27 },
  { "id": "03246", "count_mg": 21 },
  { "id": "03267", "count_mg": 21 },
  { "id": "03269", "count_mg": 21 },
  { "id": "03274", "count_mg": 21 }
]
```

Une solution plus optimale pour écrire cette requête consiste à effectuer une jointure sur la sous-requête et à référencer l’alias de celle-ci dans les clauses SELECT et WHERE. Cette requête est plus efficace, car vous ne devez exécuter la sous-requête qu’au sein de l’instruction de jointure et non dans la projection et le filtre.

```sql
SELECT TOP 5 f.id, count_mg
FROM food f
JOIN (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg') AS count_mg
WHERE count_mg > 20
```

## <a name="exists-expression"></a>Expression EXISTS

Azure Cosmos DB prend en charge les expressions EXISTS. Il s’agit d’une sous-requête scalaire d’agrégation intégrée à l’API SQL Azure Cosmos DB. EXISTS est une expression booléenne qui prend une expression de sous-requête et retourne true si la sous-requête retourne des lignes. Sinon, elle retourne false.

Étant donné que l’API SQL Azure Cosmos DB considère indifféremment les expressions booléennes et toutes les autres expressions scalaires, vous pouvez utiliser EXISTS dans les clauses SELECT et WHERE. Cela diffère de T-SQL, où une expression booléenne (par exemple, EXISTS, BETWEEN et IN) est limitée au filtre.

Si la sous-requête EXISTS retourne une valeur unique qui n’est pas définie, EXISTS a la valeur false. Par exemple, considérez la requête suivante qui a la valeur false :
```sql
SELECT EXISTS (SELECT VALUE undefined)
```   


Si le mot clé VALUE dans la sous-requête précédente est omis, la requête a la valeur true :
```sql
SELECT EXISTS (SELECT undefined) 
```

La sous-requête inclut dans un objet la liste des valeurs de la liste sélectionnée. Si la liste sélectionnée n’a aucune valeur, la sous-requête retourne la valeur unique « {} ». Cette valeur étant définie, EXISTS a la valeur true.

### <a name="example-rewriting-array_contains-and-join-as-exists"></a>Exemple : réécriture d’ARRAY_CONTAINS et JOIN avec EXISTS

Un cas d’usage courant d’ARRAY_CONTAINS consiste à filtrer un document en fonction de l’existence d’un élément dans un tableau. Dans ce cas, nous vérifions si le tableau tags contient un élément nommé « orange ».

```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE ARRAY_CONTAINS(f.tags, {name: 'orange'})
```

Vous pouvez réécrire la même requête de manière à utiliser EXISTS :

```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE EXISTS(SELECT VALUE t FROM t IN f.tags WHERE t.name = 'orange')
```

De plus, ARRAY_CONTAINS peut uniquement vérifier si une valeur est égale à un élément dans un tableau. Si vous avez besoin de filtres plus complexes sur des propriétés de tableau, utilisez JOIN.

Considérez la requête suivante qui filtre en fonction des propriétés units et `nutritionValue` dans le tableau : 

```sql
SELECT VALUE c.description
FROM c
JOIN n IN c.nutrients
WHERE n.units= "mg" AND n.nutritionValue > 0
```

Pour chacun des documents dans la collection, un produit croisé est effectué avec ses éléments de tableau. Cette opération JOIN permet d’effectuer un filtrage sur les propriétés dans le tableau. Toutefois, la consommation de RU de cette requête est importante. Par exemple, si 1 000 documents contiennent 100 éléments dans chaque tableau, l’opération aboutit à 1 000 x 100 (autrement dit, 100 000) tuples.

L’utilisation d’EXISTS peut aider à éviter ce produit croisé couteux :

```sql
SELECT VALUE c.description
FROM c
WHERE EXISTS(
    SELECT VALUE n
    FROM n IN c.nutrients
    WHERE n.units = "mg" AND n.nutritionValue > 0
)
```

Dans ce cas, vous effectuez le filtrage sur les éléments de tableau dans la sous-requête EXISTS. Si un élément de tableau correspond au filtre, vous le projetez et EXISTS a la valeur true.

Vous pouvez également attribuer un alias à EXISTS et le référencer dans la projection :

```sql
SELECT TOP 1 c.description, EXISTS(
    SELECT VALUE n
    FROM n IN c.nutrients
    WHERE n.units = "mg" AND n.nutritionValue > 0) as a
FROM c
```

Sortie de la requête :

```json
[
    {
        "description": "Babyfood, dessert, fruit pudding, orange, strained",
        "a": true
    }
]
```

## <a name="array-expression"></a>Expression ARRAY

Vous pouvez utiliser l’expression ARRAY pour projeter les résultats d’une requête sous forme de tableau. Vous ne pouvez utiliser cette expression que dans la clause SELECT de la requête.

```sql
SELECT TOP 1   f.id, ARRAY(SELECT VALUE t.name FROM t in f.tags) AS tagNames
FROM  food f
```

Sortie de la requête :

```json
[
    {
        "id": "03238",
        "tagNames": [
            "babyfood",
            "dessert",
            "tropical fruit",
            "junior"
        ]
    }
]
```

Comme dans le cas des autres sous-requêtes, vous pouvez utiliser des filtres avec l’expression ARRAY.

```sql
SELECT TOP 1 c.id, ARRAY(SELECT VALUE t FROM t in c.tags WHERE t.name != 'infant formula') AS tagNames
FROM c
```

Sortie de la requête :

```json
[
    {
        "id": "03226",
        "tagNames": [
            {
                "name": "babyfood"
            },
            {
                "name": "dessert"
            },
            {
                "name": "fruit pudding"
            },
            {
                "name": "orange"
            },
            {
                "name": "strained"
            }
        ]
    }
]
```

Les expressions de tableau peuvent également apparaître après la clause FROM dans les sous-requêtes.

```sql
SELECT TOP 1 c.id, ARRAY(SELECT VALUE t.name FROM t in c.tags) as tagNames
FROM c
JOIN n IN (SELECT VALUE ARRAY(SELECT t FROM t in c.tags WHERE t.name != 'infant formula'))
```

Sortie de la requête :

```json
[
    {
        "id": "03238",
        "tagNames": [
            "babyfood",
            "dessert",
            "tropical fruit",
            "junior"
        ]
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Modéliser des données de document](modeling-data.md)
