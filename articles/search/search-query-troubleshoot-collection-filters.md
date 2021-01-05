---
title: Résolution des problèmes de filtres de collection OData
titleSuffix: Azure Cognitive Search
description: Découvrez des approches permettant de résoudre des erreurs liées aux filtres de collection OData dans les requêtes de Recherche cognitive Azure.
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
ms.openlocfilehash: 3050f701c11773207aa6054d4d08d908d87b2ce7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88932064"
---
# <a name="troubleshooting-odata-collection-filters-in-azure-cognitive-search"></a>Résolution de problèmes liés aux filtres de collection OData dans la Recherche cognitive Azure

Pour appliquer un [filtre](query-odata-filter-orderby-syntax.md) sur des champs de collection dans la Recherche cognitive Azure, vous pouvez utiliser les [opérateurs `any` et `all`](search-query-odata-collection-operators.md) avec des **expressions lambda**. Une expression lambda est un sous-filtre qui est appliqué à chaque élément d’une collection.

Toutes les fonctionnalités des expressions de filtre ne sont pas disponibles dans une expression lambda. Les fonctionnalités diffèrent selon le type de données du champ de la collection que vous souhaitez filtrer. Cela peut entraîner une erreur si vous essayez d’utiliser une fonctionnalité dans une expression lambda qui n’est pas prise en charge dans ce contexte. Si vous rencontrez de telles erreurs lors de la tentative d’écriture d’un filtre complexe sur des champs de collection, cet article vous aidera à résoudre le problème.

## <a name="common-collection-filter-errors"></a>Erreurs de filtre de collection courantes

Le tableau suivant répertorie les erreurs que vous pouvez rencontrer lorsque vous tentez d’exécuter un filtre de collection. Ces erreurs se produisent lorsque vous utilisez une fonctionnalité d’expressions de filtre qui n’est pas prise en charge dans une expression lambda. Chaque erreur donne des conseils sur la façon dont vous pouvez réécrire votre filtre afin d’éviter l’erreur. Le tableau contient également un lien vers la section appropriée de cet article, qui fournit plus d’informations sur la façon d’éviter cette erreur.

| Message d’erreur | Situation | Pour plus d'informations, consultez la rubrique |
| --- | --- | --- |
| La fonction « ismatch » n’a aucun paramètre lié à la variable de portée « s ». Seules les références de champ liées sont prises en charge dans les expressions lambda (« any » ou « all »). Veuillez modifier votre filtre afin que la fonction « ismatch » soit en dehors de l’expression lambda et réessayez. | Utilisation de `search.ismatch` ou `search.ismatchscoring` dans une expression lambda | [Règles de filtrage des collections complexes](#bkmk_complex) |
| Expression lambda non valide. Un test d’égalité ou d’inégalité où le contraire était attendu a été trouvé dans une expression lambda qui itère sur un champ de type collection (Edm.String). Pour « any », utilisez des expressions de forme « x eq y » ou « search.in(...) ». Pour « all », utilisez des expressions de forme « x ne y », « not (x eq y) » ou « not search.in(...) ». | Filtrage sur un champ de type `Collection(Edm.String)` | [Règles de filtrage des collections de chaînes](#bkmk_strings) |
| Expression lambda non valide. Une forme non prise en charge d’une expression booléenne complexe a été trouvé. Pour « any », utilisez des expressions « ORs de ANDs », également appelées forme normale disjonctive. Par exemple : « (a et b) ou (c et d) », où a, b, c et d sont des sous-expressions de comparaison ou d’égalité. Pour « all », utilisez des expressions « ANDs de ORs », également appelées forme normale conjonctive. Par exemple : « (a ou b) ou (c ou d) », où a, b, c et d sont des sous-expressions de comparaison ou d’inégalité. Exemples d’expressions de comparaison : « x gt 5 », « x le 2 ». Exemple d’expression d’égalité : « x eq 5 ». Exemple d’expression d’inégalité : « x ne 5 ». | Filtrage sur des champs de type `Collection(Edm.DateTimeOffset)`, `Collection(Edm.Double)`, `Collection(Edm.Int32)` ou `Collection(Edm.Int64)` | [Règles de filtrage des collections comparables](#bkmk_comparables) |
| Expression lambda non valide. Une utilisation non prise en charge de geo.distance() ou de geo.intersects() a été trouvée dans une expression lambda qui itère sur un champ de type collection(Edm.GeographyPoint). Pour « any », vérifiez que vous comparez geo.distance() à l’aide des opérateurs « lt » ou « le » et qu’aucune utilisation de geo.intersects() n’est négative. Pour « all », vérifiez que vous comparez geo.distance() à l’aide des opérateurs « gt » ou « ge » et que toute utilisation de geo.intersects() est négative. | Filtrage sur un champ de type `Collection(Edm.GeographyPoint)` | [Règles de filtrage des collections GeographyPoint](#bkmk_geopoints) |
| Expression lambda non valide. Les expressions booléennes complexes ne sont pas prises en charge dans les expressions lambda qui itèrent sur des champs de type Collection(Edm.GeographyPoint). Pour « any », reliez les sous-expressions avec « ou » ; « et » n’est pas pris en charge. Pour « all », reliez les sous-expressions avec « et » ; « ou » n’est pas pris en charge. | Filtrage sur des champs de type `Collection(Edm.String)` ou `Collection(Edm.GeographyPoint)` | [Règles de filtrage des collections de chaînes](#bkmk_strings) <br/><br/> [Règles de filtrage des collections GeographyPoint](#bkmk_geopoints) |
| Expression lambda non valide. Trouver un opérateur de comparaison (« lt », « le », « gt » ou « ge »). Seuls des opérateurs d’égalité sont autorisés dans les expressions lambda qui itèrent sur les champs de type collection (Edm.String). Pour « any », utilisez des expressions de forme « x eq y ». Pour « all », utilisez des expressions de forme « x ne y » ou « not (x eq y) ». | Filtrage sur un champ de type `Collection(Edm.String)` | [Règles de filtrage des collections de chaînes](#bkmk_strings) |

<a name="bkmk_examples"></a>

## <a name="how-to-write-valid-collection-filters"></a>Comment écrire des filtres de collection valides

Les règles d’écriture de filtres de collection valides sont différentes pour chaque type de données. Les sections suivantes décrivent les règles en donnant des exemples de fonctionnalités de filtres qui sont prises en charge ou ne le sont pas :

- [Règles de filtrage des collections de chaînes](#bkmk_strings)
- [Règles de filtrage des collections booléennes](#bkmk_bools)
- [Règles de filtrage des collections GeographyPoint](#bkmk_geopoints)
- [Règles de filtrage des collections comparables](#bkmk_comparables)
- [Règles de filtrage des collections complexes](#bkmk_complex)

<a name="bkmk_strings"></a>

## <a name="rules-for-filtering-string-collections"></a>Règles de filtrage des collections de chaînes

Dans les expressions lambda pour les collections de chaînes, les seuls opérateurs de comparaison pouvant être utilisés sont `eq` et `ne`.

> [!NOTE]
> La Recherche cognitive Azure ne prend pas en charge les opérateurs `lt`/`le`/`gt`/`ge` pour les chaînes, qu’ils soient inclus ou non dans une expression lambda.

Le corps d’un `any` peut uniquement tester l’égalité, tandis que le corps d’un `all` peut uniquement tester l’inégalité.

Il est également possible de combiner plusieurs expressions via `or` dans le corps d’un `any` et via `and` dans le corps d’un `all`. Étant donné que la fonction `search.in` est équivalente à la combinaison de vérifications d’égalité avec `or`, elle est également autorisée dans le corps d’un `any`. À l’inverse, `not search.in` est autorisé dans le corps d’un `all`.

Par exemple, ces expressions sont autorisées :

- `tags/any(t: t eq 'books')`
- `tags/any(t: search.in(t, 'books, games, toys'))`
- `tags/all(t: t ne 'books')`
- `tags/all(t: not (t eq 'books'))`
- `tags/all(t: not search.in(t, 'books, games, toys'))`
- `tags/any(t: t eq 'books' or t eq 'games')`
- `tags/all(t: t ne 'books' and not (t eq 'games'))`

alors que ces expressions ne le sont pas :

- `tags/any(t: t ne 'books')`
- `tags/any(t: not search.in(t, 'books, games, toys'))`
- `tags/all(t: t eq 'books')`
- `tags/all(t: search.in(t, 'books, games, toys'))`
- `tags/any(t: t eq 'books' and t ne 'games')`
- `tags/all(t: t ne 'books' or not (t eq 'games'))`

<a name="bkmk_bools"></a>

## <a name="rules-for-filtering-boolean-collections"></a>Règles de filtrage des collections booléennes

Le type `Edm.Boolean` prend uniquement en charge les opérateurs `eq` et `ne`. Par conséquent, permettre la combinaison de clauses qui vérifient la même variable de portée avec `and`/`or` n’a pas de sens, car cela entraîne toujours des tautologies ou des contradictions.

Voici quelques exemples de filtres sur les collections booléennes qui sont autorisés :

- `flags/any(f: f)`
- `flags/all(f: f)`
- `flags/any(f: f eq true)`
- `flags/any(f: f ne true)`
- `flags/all(f: not f)`
- `flags/all(f: not (f eq true))`

Contrairement aux collections de chaînes, les collections booléennes n’ont aucune limite sur laquelle l’opérateur peut être utilisé dans le type d’expression lambda. Les deux `eq` et `ne` peuvent être utilisés dans le corps de `any` ou de `all`.

Les expressions comme celles qui suivent ne sont pas autorisées pour les collections booléennes :

- `flags/any(f: f or not f)`
- `flags/any(f: f or f)`
- `flags/all(f: f and not f)`
- `flags/all(f: f and f eq true)`

<a name="bkmk_geopoints"></a>

## <a name="rules-for-filtering-geographypoint-collections"></a>Règles de filtrage des collections GeographyPoint

Les valeurs de type `Edm.GeographyPoint` dans une collection ne peuvent pas être comparées directement entre les unes aux autres. Au lieu de cela, elles doivent être utilisées en tant que paramètres des fonctions `geo.distance` et `geo.intersects`. La fonction `geo.distance` doit à son tour être comparée à une valeur de distance à l’aide d’un des opérateurs de comparaison `lt`, `le`, `gt` ou `ge`. Ces règles s’appliquent également aux champs Edm.GeographyPoint autres que collection.

Comme les collections de chaînes, les collections `Edm.GeographyPoint` ont des règles relatives à la façon dont les fonctions géospatiales peuvent être utilisées et combinées dans les différents types d’expressions lambda :

- Les opérateurs de comparaison que vous pouvez utiliser avec la fonction `geo.distance` varient selon le type d’expression lambda. Pour `any`, vous pouvez utiliser uniquement `lt` ou `le`. Pour `all`, vous pouvez utiliser uniquement `gt` ou `ge`. Vous pouvez inverser les expressions qui impliquent `geo.distance`, mais vous devrez modifier l’opérateur de comparaison (`geo.distance(...) lt x` devient `not (geo.distance(...) ge x)` et `geo.distance(...) le x` devient `not (geo.distance(...) gt x)`).
- Dans le corps d’un `all`, la fonction `geo.intersects` doit être négative. Dans le corps d’un `any`, la fonction `geo.intersects` doit être négative.
- Dans le corps d’un `any`, des expressions géospatiales peuvent être combinées à l’aide de `or`. Dans le corps d’un `all`, ces expressions peuvent être combinées à l’aide de `and`.

Les limitations ci-dessus existent pour des raisons similaires à la limitation d’égalités/d’inégalités sur les collections de chaînes. Consultez [Présentation des filtres de collection OData dans la Recherche cognitive Azure](search-query-understand-collection-filters.md) pour en savoir plus sur ces raisons.

Voici quelques exemples de filtres autorisés sur `Edm.GeographyPoint` :

- `locations/any(l: geo.distance(l, geography'POINT(-122 49)') lt 10)`
- `locations/any(l: not (geo.distance(l, geography'POINT(-122 49)') ge 10) or geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/all(l: geo.distance(l, geography'POINT(-122 49)') ge 10 and not geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`

Les expressions comme celles qui suivent ne sont pas autorisées pour les collections `Edm.GeographyPoint` :

- `locations/any(l: l eq geography'POINT(-122 49)')`
- `locations/any(l: not geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/all(l: geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/any(l: geo.distance(l, geography'POINT(-122 49)') gt 10)`
- `locations/all(l: geo.distance(l, geography'POINT(-122 49)') lt 10)`
- `locations/any(l: geo.distance(l, geography'POINT(-122 49)') lt 10 and geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`
- `locations/all(l: geo.distance(l, geography'POINT(-122 49)') le 10 or not geo.intersects(l, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'))`

<a name="bkmk_comparables"></a>

## <a name="rules-for-filtering-comparable-collections"></a>Règles de filtrage des collections comparables

Cette section s’applique à tous les types de données suivants :

- `Collection(Edm.DateTimeOffset)`
- `Collection(Edm.Double)`
- `Collection(Edm.Int32)`
- `Collection(Edm.Int64)`

Les types tels que `Edm.Int32` et `Edm.DateTimeOffset` prennent en charge les six des opérateurs de comparaison : `eq`, `ne`, `lt`, `le`, `gt` et `ge`. Les expressions lambda sur des collections de ces types peuvent contenir des expressions simples utilisant l’un de ces opérateurs. Cela s’applique à `any` et à `all`. Les filtres suivants sont par exemple autorisés :

- `ratings/any(r: r ne 5)`
- `dates/any(d: d gt 2017-08-24T00:00:00Z)`
- `not margins/all(m: m eq 3.5)`

Toutefois, il existe des limitations sur la façon dont ces expressions de comparaison peuvent être combinées dans des expressions plus complexes à l’intérieur d’une expression lambda :

- Règles pour `any` :
  - Des expressions d’inégalité simples ne peuvent pas être combinées de façon utile avec d’autres expressions. Par exemple, cette expression est autorisée :
    - `ratings/any(r: r ne 5)`

    mais cette expression ne l’est pas :
    - `ratings/any(r: r ne 5 and r gt 2)`

    et, bien que cette expression soit autorisée, elle n’est pas utile, car les conditions se chevauchent :
    - `ratings/any(r: r ne 5 or r gt 7)`
  - Des expressions de comparaison simples impliquant `eq`, `lt`, `le`, `gt` ou `ge` peuvent être combinées avec `and`/`or`. Par exemple :
    - `ratings/any(r: r gt 2 and r le 5)`
    - `ratings/any(r: r le 5 or r gt 7)`
  - Les expressions de comparaison combinées avec `and` (conjonctions) peuvent également l’être à l’aide de `or`. Dans une logique booléenne, cette forme est appelée « [Forme normale disjonctive](https://en.wikipedia.org/wiki/Disjunctive_normal_form) » (FND). Par exemple :
    - `ratings/any(r: (r gt 2 and r le 5) or (r gt 7 and r lt 10))`
- Règles pour `all` :
  - Des expressions d’égalité simples ne peuvent pas être combinées de façon utile avec d’autres expressions. Par exemple, cette expression est autorisée :
    - `ratings/all(r: r eq 5)`

    mais cette expression ne l’est pas :
    - `ratings/all(r: r eq 5 or r le 2)`

    et, bien que cette expression soit autorisée, elle n’est pas utile, car les conditions se chevauchent :
    - `ratings/all(r: r eq 5 and r le 7)`
  - Des expressions de comparaison simples impliquant `ne`, `lt`, `le`, `gt` ou `ge` peuvent être combinées avec `and`/`or`. Par exemple :
    - `ratings/all(r: r gt 2 and r le 5)`
    - `ratings/all(r: r le 5 or r gt 7)`
  - Les expressions de comparaison combinées avec `or` (disjonctions) peuvent également l’être à l’aide de `and`. Dans une logique booléenne, cette forme est appelée « [Forme normale conjonctive](https://en.wikipedia.org/wiki/Conjunctive_normal_form) » (FNC). Par exemple :
    - `ratings/all(r: (r le 2 or gt 5) and (r lt 7 or r ge 10))`

<a name="bkmk_complex"></a>

## <a name="rules-for-filtering-complex-collections"></a>Règles de filtrage des collections complexes

Les expressions lambda sur des collections complexes prennent en charge une syntaxe beaucoup plus flexible que les expressions lambda sur des collections de types primitifs. Dans une telle construction lambda, vous pouvez utiliser n’importe quelle construction de filtre que vous pouvez également utiliser ailleurs, à deux exceptions près seulement.

Tout d’abord, les fonctions `search.ismatch` et `search.ismatchscoring` ne sont pas prises en charge dans les expressions lambda. Pour plus d’informations, consultez [Présentation des filtres de collection OData dans la Recherche cognitive Azure](search-query-understand-collection-filters.md).

Ensuite, il n’est pas permis de faire référence à des champs qui ne sont pas *liés* à la variable de portée (appelée  *variables libres*). Voyons par exemple les deux expressions de filtres OData équivalentes suivantes :

1. `stores/any(s: s/amenities/any(a: a eq 'parking')) and details/margin gt 0.5`
1. `stores/any(s: s/amenities/any(a: a eq 'parking' and details/margin gt 0.5))`

La première expression sera autorisée, alors que la seconde forme sera rejetée, car `details/margin` n’est pas lié à la variable de portée `s`.

Cette règle s’applique également aux expressions qui ont des variables liées dans une portée externe. Ces variables sont libres par rapport à la portée dans laquelle elles apparaissent. Par exemple, la première expression est autorisée, tandis que la seconde expression, équivalente, ne l’est pas, car `s/name` est libre en par rapport à la portée de la variable de portée `a` :

1. `stores/any(s: s/amenities/any(a: a eq 'parking') and s/name ne 'Flagship')`
1. `stores/any(s: s/amenities/any(a: a eq 'parking' and s/name ne 'Flagship'))`

Cette limitation ne devrait pas être un problème dans la pratique, car il est toujours possible de construire des filtres de telle façon que les expressions lambda contiennent uniquement des variables liées.

## <a name="cheat-sheet-for-collection-filter-rules"></a>Aide-mémoire pour les règles de filtre de collection

Le tableau suivant récapitule les règles de construction de filtres valides pour chaque type de données de collection.

[!INCLUDE [Limitations on OData lambda expressions in Azure Cognitive Search](../../includes/search-query-odata-lambda-limitations.md)]

Pour obtenir des exemples montrant comment construire des filtres valides pour chaque cas, consultez [Comment écrire des filtres de collection valides](#bkmk_examples).

Si vous écrivez souvent des filtres et qu’il vous serait plus utile de comprendre les principes fondamentaux des règles plutôt que simplement les mémoriser, consultez [Présentation des filtres de collection OData dans la Recherche cognitive Azure](search-query-understand-collection-filters.md).

## <a name="next-steps"></a>Étapes suivantes  

- [Présentation des filtres de collection OData dans la Recherche cognitive Azure](search-query-understand-collection-filters.md)
- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)