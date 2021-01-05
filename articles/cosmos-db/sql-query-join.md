---
title: Requêtes JOIN SQL pour Azure Cosmos DB
description: Découvrez comment joindre (JOIN) plusieurs tables dans Azure Cosmos DB pour interroger les données
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/17/2019
ms.author: mjbrown
ms.openlocfilehash: c3fad0c7eed2b42a9ac340a091f17b90e22b000a
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93333832"
---
# <a name="joins-in-azure-cosmos-db"></a>Jointures dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans une base de données relationnelle, les jointures entre les tables sont la conséquence logique de la conception de schémas normalisés. À l’opposé, l’API SQL utilise le modèle de données dénormalisée des éléments sans schéma, ce qui est l’équivalent logique d’une *jointure réflexive*.

Les jointures internes aboutissent à un produit croisé complet des ensembles participants à la jointure. Le résultat d’une jointure à N voies est un jeu de tuples à N éléments, où chaque valeur dans le tuple est associée à l’alias défini participant à la jointure et est accessible en référençant cet alias dans d’autres clauses.

## <a name="syntax"></a>Syntaxe

Le langage prend en charge la syntaxe `<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`. Cette requête retourne un jeu de tuples avec `N` valeurs. Les valeurs de chaque tuple sont produites par l’itération de tous les alias du conteneur sur leurs ensembles respectifs. 

Examinons la clause FROM suivante : `<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`  
  
 Laissez chaque source définir `input_alias1, input_alias2, …, input_aliasN`. La close FROM renvoie un ensemble de N-tuples (un tuple avec N valeurs). Les valeurs de chaque tuple sont produites par l’itération de tous les alias du conteneur sur leurs ensembles respectifs.  
  
**Exemple 1** - 2 sources  
  
- Laissez `<from_source1>` être étendu à un conteneur et représenter le jeu {A, B, C}.  
  
- Laissez `<from_source2>` être étendu à un document référençant input_alias1 et représenter les jeux :  
  
    {1, 2} pour `input_alias1 = A,`  
  
    {3} pour `input_alias1 = B,`  
  
    {4, 5} pour `input_alias1 = C,`  
  
- La clause FROM `<from_source1> JOIN <from_source2>` engendre les tuples suivants :  
  
    (`input_alias1, input_alias2`):  
  
    `(A, 1), (A, 2), (B, 3), (C, 4), (C, 5)`  
  
**Exemple 2** - 3 sources  
  
- Laissez `<from_source1>` être étendu à un conteneur et représenter le jeu {A, B, C}.  
  
- Laissez `<from_source2>` être étendu à un document référençant input_`input_alias1` et représenter les jeux :  
  
    {1, 2} pour `input_alias1 = A,`  
  
    {3} pour `input_alias1 = B,`  
  
    {4, 5} pour `input_alias1 = C,`  
  
- Laissez `<from_source3>` être étendu à un document référençant input_`input_alias2` et représenter les jeux :  
  
    {100, 200} pour `input_alias2 = 1,`  
  
    {300} pour `input_alias2 = 3,`  
  
- La clause FROM `<from_source1> JOIN <from_source2> JOIN <from_source3>` engendre les tuples suivants :  
  
    (input_alias1, input_alias2, input_alias3) :  
  
    (A, 1, 100), (A, 1, 200), (B, 3, 300)  
  
  > [!NOTE]
  > Absence de tuples pour les autres valeurs de `input_alias1`, `input_alias2`, pour lesquelles `<from_source3>` n’a retourné aucune valeur.  
  
**Exemple 3** - 3 sources  
  
- Laissez <from_source1> être étendu à un conteneur et représenter le jeu {A, B, C}.  
  
- Laissez `<from_source1>` être étendu à un conteneur et représenter le jeu {A, B, C}.  
  
- Laissez <from_source2> être étendu à un document référençant input_source1 et représenter les jeux :  
  
    {1, 2} pour `input_alias1 = A,`  
  
    {3} pour `input_alias1 = B,`  
  
    {4, 5} pour `input_alias1 = C,`  
  
- Laissez `<from_source3>` être étendu à `input_alias1` et représenter les jeux :  
  
    {100, 200} pour `input_alias2 = A,`  
  
    {300} pour `input_alias2 = C,`  
  
- La clause FROM `<from_source1> JOIN <from_source2> JOIN <from_source3>` engendre les tuples suivants :  
  
    (`input_alias1, input_alias2, input_alias3`):  
  
    (A, 1, 100), (A, 1, 200), (A, 2, 100), (A, 2, 200), (C, 4, 300), (C, 5, 300)  
  
  > [!NOTE]
  > Cela a abouti à un produit croisé entre `<from_source2>` et `<from_source3>`, car les deux sont étendus à la même `<from_source1>`.  Cela a about à 4 (2 x 2) tuples de valeur A, 0 tuple de valeur B (1 x 0) et 2 (2 x 1) tuples de valeur C.  
  
## <a name="examples"></a>Exemples

Les exemples suivants illustrent le fonctionnement de la clause JOIN. Avant d’exécuter ces exemples, téléchargez l’exemple de [données de la famille](sql-query-getting-started.md#upload-sample-data). Dans l’exemple suivant, le résultat est vide, car le produit croisé de chaque élément de la source et d’un ensemble vide est vide :

```sql
    SELECT f.id
    FROM Families f
    JOIN f.NonExistent
```

Le résultat est le suivant :

```json
    [{
    }]
```

Dans l’exemple suivant, la jointure est un produit croisé entre deux objets JSON, l’`id` de la racine de l’élément et la sous-racine `children`. Le fait que `children` soit un tableau n’est pas efficace dans la jointure, car l’opération porte sur une seule racine qui est le tableau `children`. Nous n’obtenons que deux résultats, car le produit croisé de chaque élément avec le tableau génère exactement un seul élément.

```sql
    SELECT f.id
    FROM Families f
    JOIN f.children
```

Les résultats sont :

```json
    [
      {
        "id": "AndersenFamily"
      },
      {
        "id": "WakefieldFamily"
      }
    ]
```

L'exemple suivant illustre une jointure plus conventionnelle :

```sql
    SELECT f.id
    FROM Families f
    JOIN c IN f.children
```

Les résultats sont :

```json
    [
      {
        "id": "AndersenFamily"
      },
      {
        "id": "WakefieldFamily"
      },
      {
        "id": "WakefieldFamily"
      }
    ]
```

La source FROM de la clause JOIN est un itérateur. Ainsi, le flux dans l’exemple précédent est :  

1. Développez chaque élément enfant `c` dans le tableau.
2. Appliquez un produit croisé avec la racine de l’élément `f` avec chaque élément enfant `c` aplati par la première étape.
3. Enfin, projetez seule la propriété `id` de l’objet racine `f`.

Le premier élément, `AndersenFamily`, contient un seul élément `children`. Le jeu de résultats contient donc un seul objet. Le deuxième élément, `WakefieldFamily`, contient deux `children`. Le produit croisé génère donc deux objets, un pour chaque élément `children`. Les champs racine de ces deux éléments sont identiques, comme on peut l’attendre d’un produit croisé.

La véritable utilité de la clause JOIN est de former des tuples à partir du produit croisé dans une forme qui serait autrement difficile à projeter. L’exemple ci-dessous filtre la combinaison d’un tuple permettant à l’utilisateur de choisir une condition respectée par l’ensemble des tuples.

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName
    FROM Families f
    JOIN c IN f.children
    JOIN p IN c.pets
```

Les résultats sont :

```json
    [
      {
        "familyName": "AndersenFamily",
        "childFirstName": "Henriette Thaulow",
        "petName": "Fluffy"
      },
      {
        "familyName": "WakefieldFamily",
        "childGivenName": "Jesse",
        "petName": "Goofy"
      }, 
      {
       "familyName": "WakefieldFamily",
       "childGivenName": "Jesse",
       "petName": "Shadow"
      }
    ]
```

L’extension suivante de l’exemple précédent effectue une double jointure. Vous pourriez voir le produit croisé comme le pseudo-code ci-dessous :

```
    for-each(Family f in Families)
    {
        for-each(Child c in f.children)
        {
            for-each(Pet p in c.pets)
            {
                return (Tuple(f.id AS familyName,
                  c.givenName AS childGivenName,
                  c.firstName AS childFirstName,
                  p.givenName AS petName));
            }
        }
    }
```

`AndersenFamily` ayant un enfant qui a un animal, le produit croisé retourne une ligne (1\*1\*1) à partir de cette famille. `WakefieldFamily` a deux enfants, dont un seul a des animaux, en l’occurrence deux animaux. Le produit croisé pour cette famille génère 1\*1\*2 = 2 lignes.

L’exemple suivant contient un filtre supplémentaire sur `pet`, qui exclut tous les tuples dont le nom d’animal de compagnie (« pet ») n’est pas `Shadow`. Vous pouvez développer des tuples à partir de tableaux, filtrer n’importe quel élément du tuple et projeter n’importe quelle combinaison d’éléments.

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName
    FROM Families f
    JOIN c IN f.children
    JOIN p IN c.pets
    WHERE p.givenName = "Shadow"
```

Les résultats sont :

```json
    [
      {
       "familyName": "WakefieldFamily",
       "childGivenName": "Jesse",
       "petName": "Shadow"
      }
    ]
```

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer](sql-query-getting-started.md)
- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmosdb-dotnet)
- [Subqueries](sql-query-subquery.md)