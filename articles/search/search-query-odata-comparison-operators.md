---
title: Informations de référence sur les opérateurs de comparaison OData
titleSuffix: Azure Cognitive Search
description: Documentation de référence et de la syntaxe pour l’utilisation des opérateurs de comparaison OData (eq, ne, gt, lt, ge et le) dans les requêtes Recherche cognitive Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: fc5803f96c30ea1df362676aa8c4104bb0b69db3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88934869"
---
# <a name="odata-comparison-operators-in-azure-cognitive-search---eq-ne-gt-lt-ge-and-le"></a>Opérateurs de comparaison OData dans Recherche cognitive Azure : `eq`, `ne`, `gt`, `lt`, `ge`, `le`

L’opération la plus basique dans une [expression de filtre OData](query-odata-filter-orderby-syntax.md) dans Recherche cognitive Azure consiste à comparer un champ à une valeur donnée. Deux types de comparaison sont possibles : la comparaison d’égalité et la comparaison de plages. Vous pouvez utiliser les opérateurs suivants pour comparer un champ à une valeur constante :

Opérateurs d’égalité :

- `eq`: teste si un champ est **égal à** une valeur constante
- `ne`: teste si un champ n’est **pas égal à** une valeur constante

Opérateurs de plage :

- `gt`: teste si un champ est **supérieur à** une valeur constante
- `lt`: teste si un champ est **inférieur à** une valeur constante
- `ge`: teste si un champ est **supérieur ou égal à** une valeur constante
- `le`: teste si un champ est **inférieur ou égal à** une valeur constante

Vous pouvez utiliser plusieurs opérateurs de plage en association avec les [opérateurs logiques](search-query-odata-logical-operators.md) afin de tester si un champ est compris dans une certaine plage de valeurs. Consultez les [exemples](#examples) disponibles plus loin dans cet article.

> [!NOTE]
> Si vous préférez, vous pouvez placer la valeur constante sur le côté gauche de l’opérateur et le nom du champ sur le côté droit. Pour les opérateurs de plage, la signification de la comparaison est inversée. Par exemple, si la valeur constante se trouve sur la gauche, `gt` teste si la valeur constante est supérieure à celle du champ. Vous pouvez également utiliser les opérateurs de comparaison pour comparer le résultat d’une fonction, par exemple `geo.distance`, à une valeur. Pour les fonctions booléennes telles que `search.ismatch`, vous n’êtes pas obligé de comparer le résultat à `true` ou `false`.

## <a name="syntax"></a>Syntaxe

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire d’une expression OData utilisant les opérateurs de comparaison.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
comparison_expression ::=
    variable_or_function comparison_operator constant |
    constant comparison_operator variable_or_function

variable_or_function ::= variable | function_call

comparison_operator ::= 'gt' | 'lt' | 'ge' | 'le' | 'eq' | 'ne'
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#comparison_expression)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Il existe deux formes d’expressions de comparaison. La seule différence entre eux est si la constante apparaît sur la gauche ou la droite de l’opérateur. L’expression de l’autre côté de l’opérateur doit être une **variable** ou un appel de fonction. Une variable peut être un nom de champ ou une variable de portée dans le cadre d’une [expression lambda](search-query-odata-collection-operators.md).

## <a name="data-types-for-comparisons"></a>Types de données pour les comparaisons

Les types de données des deux côtés d’un opérateur de comparaison doivent être compatibles. Par exemple, si le côté gauche est un champ de type `Edm.DateTimeOffset`, le côté droit doit être une valeur date-heure constante. Les types de données numériques sont plus flexibles. Vous pouvez comparer les variables et les fonctions de n’importe quel type numérique avec des constantes du type numérique de votre choix, avec cependant quelques limitations, comme décrit dans le tableau suivant.

| Type de variable ou de fonction | Type valeur constant | Limites |
| --- | --- | --- |
| `Edm.Double` | `Edm.Double` | La comparaison est soumise à des [règles particulières pour `NaN`](#special-case-nan) |
| `Edm.Double` | `Edm.Int64` | La constante est convertie en `Edm.Double`, ce qui entraîne une perte de précision pour les valeurs de grande ampleur |
| `Edm.Double` | `Edm.Int32` | n/a |
| `Edm.Int64` | `Edm.Double` | Les comparaisons avec `NaN`, `-INF`, ou `INF` ne sont pas autorisées |
| `Edm.Int64` | `Edm.Int64` | n/a |
| `Edm.Int64` | `Edm.Int32` | La constante est convertie en `Edm.Int64` avant la comparaison |
| `Edm.Int32` | `Edm.Double` | Les comparaisons avec `NaN`, `-INF`, ou `INF` ne sont pas autorisées |
| `Edm.Int32` | `Edm.Int64` | n/a |
| `Edm.Int32` | `Edm.Int32` | n/a |

Pour les comparaisons qui ne sont pas autorisées, notamment la comparaison d’un champ de type `Edm.Int64` à `NaN`, l’API REST Recherche cognitive Azure renvoie une erreur « HTTP 400 : demande incorrecte ».

> [!IMPORTANT]
> Même si les comparaisons de types numériques sont flexibles, nous vous recommandons vivement d’écrire des comparaisons dans des filtres afin que la valeur constante soit du même type de données que la variable ou la fonction à laquelle elle est comparée. Cela est particulièrement important lorsque des valeurs à virgule flottante et entières sont mélangées, car les conversions implicites peuvent entraîner une perte de précision.

<a name="special-case-nan"></a>

### <a name="special-cases-for-null-and-nan"></a>Cas particuliers pour `null` et `NaN`

Lorsque vous utilisez des opérateurs de comparaison, il est important de se rappeler que tous les champs qui ne sont pas des collections dans Recherche cognitive Azure peuvent potentiellement être `null`. Le tableau suivant répertorie tous les résultats possibles pour une expression de comparaison où chaque côté peut être `null` :

| Opérateur | Résultat lorsque seul le champ ou la variable est `null` | Résultat lorsque seule la constante est `null` | Résultat lorsque le champ/la variable et la constante sont `null` |
| --- | --- | --- | --- |
| `gt` | `false` | HTTP 400 : erreur de demande incorrecte | HTTP 400 : erreur de demande incorrecte |
| `lt` | `false` | HTTP 400 : erreur de demande incorrecte | HTTP 400 : erreur de demande incorrecte |
| `ge` | `false` | HTTP 400 : erreur de demande incorrecte | HTTP 400 : erreur de demande incorrecte |
| `le` | `false` | HTTP 400 : erreur de demande incorrecte | HTTP 400 : erreur de demande incorrecte |
| `eq` | `false` | `false` | `true` |
| `ne` | `true` | `true` | `false` |

En résumé, `null` est uniquement égal à lui-même et n’est pas inférieur ou supérieur à toute autre valeur.

Si votre index comporte des champs de type `Edm.Double` et que vous chargez des valeurs `NaN` sur ces champs, vous devez en tenir compte lors de l’écriture des filtres. Recherche cognitive Azure implémente la norme IEEE 754 pour la gestion des valeurs `NaN`. Les comparaisons avec ces valeurs produisent des résultats non évidents, comme indiqué dans le tableau suivant.

| Opérateur | Résultat lorsqu’au moins un opérande est `NaN` |
| --- | --- |
| `gt` | `false` |
| `lt` | `false` |
| `ge` | `false` |
| `le` | `false` |
| `eq` | `false` |
| `ne` | `true` |

En résumé, `NaN` n’est pas égal à une autre valeur, y compris lui-même.

### <a name="comparing-geo-spatial-data"></a>Comparaison de données géospatiales

Vous ne pouvez pas comparer directement un champ de type `Edm.GeographyPoint` avec une valeur constante, mais vous pouvez utiliser la fonction `geo.distance`. Cette fonction retourne une valeur de type `Edm.Double`, ce qui vous permet de la comparer avec une valeur numérique constante et ainsi de filtrer en fonction de la distance qui la sépare des coordonnées géospatiales constantes. Consultez les [exemples](#examples) ci-dessous.

### <a name="comparing-string-data"></a>Comparaison de données de chaîne

Les chaînes peuvent être comparées dans des filtres, en cas de correspondance exacte, à l’aide des opérateurs `eq` et `ne`. Ces comparaisons sont sensibles à la casse.

## <a name="examples"></a>Exemples

Correspondance des documents dans lesquels le champ `Rating` est compris entre 3 et 5 (inclusif) :

```text
Rating ge 3 and Rating le 5
```

Correspondance des documents dans lesquels le champ `Location` est à moins de 2 kilomètres de la latitude et la longitude données :

```text
geo.distance(Location, geography'POINT(-122.031577 47.578581)') lt 2.0
```

Correspondance des documents dans lesquels le champ `LastRenovationDate` est supérieur ou égal au 1er janvier 2015, 00:00 UTC :

```text
LastRenovationDate ge 2015-01-01T00:00:00.000Z
```

Correspondance des documents dans lesquels le champ `Details/Sku` n’est pas `null` :

```text
Details/Sku ne null
```

Correspondance des documents pour les hôtels comprenant au moins une chambre de type « Chambre de luxe » et où la chaîne du champ `Rooms/Type` correspond exactement au filtre :

```text
Rooms/any(room: room/Type eq 'Deluxe Room')
```

## <a name="next-steps"></a>Étapes suivantes  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)