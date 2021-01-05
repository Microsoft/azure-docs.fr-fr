---
title: Informations de référence sur les opérateurs logiques OData
titleSuffix: Azure Cognitive Search
description: Documentation de référence et de la syntaxe pour l’utilisation des opérateurs logiques OData (and, or et not), dans les requêtes Recherche cognitive Azure.
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
ms.openlocfilehash: 27d5427d34de591f9cfeab2310d79a2fde217624
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88917871"
---
# <a name="odata-logical-operators-in-azure-cognitive-search---and-or-not"></a>Opérateurs logiques OData dans Recherche cognitive Azure - `and`, `or`, `not`

Les [expressions de filtre OData](query-odata-filter-orderby-syntax.md) dans Recherche cognitive Azure sont des expressions booléennes qui s’évaluent à `true` ou `false`. Vous pouvez écrire un filtre complexe en rédigeant une série de [filtres plus simples](search-query-odata-comparison-operators.md) et en les composant des opérateurs logiques de l’[algèbre booléen](https://en.wikipedia.org/wiki/Boolean_algebra) :

- `and`: Un opérateur binaire qui s’évalue à `true` si ses sous-expressions droites et gauches s’évaluent à `true`.
- `or`: Un opérateur binaire qui s’évalue à `true` si l’une de ses sous-expressions droite ou gauche s’évaluent à `true`.
- `not`: Un opérateur unaire qui s’évalue à `true` si sa sous-expression s’évalue à `false`, et vice versa.

Ces opérateurs, ainsi que les [opérateurs de collection`any` et `all`](search-query-odata-collection-operators.md), vous permettent de construire des filtres qui peuvent exprimer des critères de recherche complexes.

## <a name="syntax"></a>Syntaxe

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire d’une expression OData utilisant les opérateurs logiques.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
logical_expression ::=
    boolean_expression ('and' | 'or') boolean_expression
    | 'not' boolean_expression
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#logical_expression)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Il s’agit de deux formes d’expressions logiques : binaire (`and`/`or`), où il y a deux sous-expressions, et unaire (`not`), où il n’y en a qu’une. Les sous-expressions peuvent être des expressions booléennes de toutes sortes :

- Champs ou variables de plage de type `Edm.Boolean`
- Fonctions qui retournent des valeurs de type `Edm.Boolean`, telles que `geo.intersects` ou `search.ismatch`
- [Expressions de comparaison](search-query-odata-comparison-operators.md), telles que `rating gt 4`
- [Expressions de collection](search-query-odata-collection-operators.md), telles que `Rooms/any(room: room/Type eq 'Deluxe Room')`
- Littéraux booléens `true` ou `false`.
- Autres expressions logiques construites à l’aide de `and`, `or` et `not`.

> [!IMPORTANT]
> Il y a certaines situations où tous les types de sous-expressions peuvent être utilisés avec `and`/`or`, particulièrement dans des expressions lambda. Consultez [Opérateurs de collection OData dans Recherche cognitive Azure](search-query-odata-collection-operators.md#limitations) pour en savoir plus.

### <a name="logical-operators-and-null"></a>Opérateurs logiques et `null`

La plupart des expressions booléennes, telles que les fonctions et les comparaisons, ne peuvent pas produire de valeurs `null`, et les opérateurs logiques ne peuvent pas être appliqués au littéral `null` directement (par exemple, `x and null` n’est pas autorisé). Toutefois, les champs booléens peuvent être `null`, vous devez donc être conscient du comportement des opérateurs `and`, `or` et `not` en présence de null. Ceci est résumé dans le tableau suivant, où `b` est un champ de type `Edm.Boolean` :

| Expression | Résultat lorsque `b` est `null` |
| --- | --- |
| `b` | `false` |
| `not b` | `true` |
| `b eq true` | `false` |
| `b eq false` | `false` |
| `b eq null` | `true` |
| `b ne true` | `true` |
| `b ne false` | `true` |
| `b ne null` | `false` |
| `b and true` | `false` |
| `b and false` | `false` |
| `b or true` | `true` |
| `b or false` | `false` |

Lorsqu’un champ booléen `b` apparaît tout seul dans une expression de filtre, il se comporte comme s’il avait été écrit `b eq true`, donc si `b` est `null`, l’expression s’évalue à `false`. De même, `not b` se comporte comme `not (b eq true)`, donc elle s’évalue à `true`. Ainsi, les champs `null` se comportent de la même façon que `false`. Ceci est cohérent avec la manière dont ils se comportent lorsqu’ils sont combinés à d’autres expressions à l’aide de `and` et `or`, comme illustré dans le tableau ci-dessus. Malgré ça, une comparaison directe à `false` (`b eq false`) s’évalue toujours à `false`. En d’autres termes, `null` n’est pas égal à `false`, bien qu’il se comporte comme tel dans les expressions booléennes.

## <a name="examples"></a>Exemples

Correspondance des documents dans lesquels le champ `rating` est compris entre 3 et 5 (inclusif) :

```odata-filter-expr
    rating ge 3 and rating le 5
```

Correspondance des documents dans lesquels tous les éléments du champ `ratings` sont inférieurs à 3 ou supérieurs à 5 :

```odata-filter-expr
    ratings/all(r: r lt 3 or r gt 5)
```

Correspondance des documents dans lesquels le champ `location` est compris dans le polygone donné, et que le document ne contient pas le terme « public ».

```odata-filter-expr
    geo.intersects(location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))') and not search.ismatch('public')
```

Correspondance des documents pour les hôtels à Vancouver, au Canada, qui comprennent une chambre de luxe avec un prix de base inférieur à 160 :

```odata-filter-expr
    Address/City eq 'Vancouver' and Address/Country eq 'Canada' and Rooms/any(room: room/Type eq 'Deluxe Room' and room/BaseRate lt 160)
```

## <a name="next-steps"></a>Étapes suivantes  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)