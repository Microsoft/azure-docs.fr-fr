---
title: Référence de la fonction search.in OData
titleSuffix: Azure Cognitive Search
description: Documentation de syntaxe et de référence pour l’utilisation de la fonction search.in dans les requêtes Recherche cognitive Azure.
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
ms.openlocfilehash: 9ad6f89392846564631b70f0acfb5658a050be80
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88922818"
---
# <a name="odata-searchin-function-in-azure-cognitive-search"></a>Fonction `search.in` OData dans Recherche cognitive Azure

Un scénario courant dans [Expressions de filtre OData](query-odata-filter-orderby-syntax.md) consiste à vérifier si un champ unique dans chaque document est égal à l’une des nombreuses valeurs possibles. Par exemple, voici comment certaines applications implémentent le [filtrage de sécurité](search-security-trimming-for-azure-search.md) : en comparant un champ contenant un ou plusieurs ID de principal avec une liste d’ID de principal représentant l’utilisateur qui émet la requête. Une façon d’écrire une requête telle que celle-ci consiste à utiliser les opérateurs [`eq`](search-query-odata-comparison-operators.md) et [`or`](search-query-odata-logical-operators.md) :

```odata-filter-expr
    group_ids/any(g: g eq '123' or g eq '456' or g eq '789')
```

Toutefois, il existe une manière plus courte de l’écrire, à l’aide de la fonction `search.in` :

```odata-filter-expr
    group_ids/any(g: search.in(g, '123, 456, 789'))
```

> [!IMPORTANT]
> En plus d’être plus courte et plus facile à lire, l’utilisation de `search.in` offre également des [avantages en termes de performances](#bkmk_performance) et permet d’éviter certaines [limites de taille de filtres](search-query-odata-filter.md#bkmk_limits) lorsqu’il existe des centaines voire des milliers de valeurs à inclure dans le filtre. Pour cette raison, nous recommandons fortement d’utiliser `search.in` au lieu d’une disjonction plus complexe d’expressions d’égalité.

> [!NOTE]
> La version 4.01 du protocole OData standard a récemment introduit l’[`in`opérateur](https://docs.oasis-open.org/odata/odata/v4.01/cs01/part2-url-conventions/odata-v4.01-cs01-part2-url-conventions.html#_Toc505773230), qui a un comportement similaire à la fonction `search.in` dans Recherche cognitive Azure. Toutefois, Recherche cognitive Azure ne prend pas en charge cet opérateur, vous devez donc utiliser la fonction `search.in` à la place.

## <a name="syntax"></a>Syntaxe

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire de la fonction `search.in` :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
search_in_call ::=
    'search.in(' variable ',' string_literal(',' string_literal)? ')'
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#search_in_call)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

La fonction `search.in` teste si un champ de type chaîne donné ou une variable de portée est égal à une des valeurs d’une liste donnée. L’égalité entre la variable et chaque valeur de la liste est déterminée de manière à respecter la casse, de la même façon que pour l’opérateur `eq`. Ainsi, une expression comme `search.in(myfield, 'a, b, c')` équivaut à `myfield eq 'a' or myfield eq 'b' or myfield eq 'c'`, sauf que `search.in` aboutira à de bien meilleures performances.

Il existe deux surcharges de la fonction `search.in` :

- `search.in(variable, valueList)`
- `search.in(variable, valueList, delimiters)`

Les paramètres sont définis dans le tableau suivant :

| Nom du paramètre | Type | Description |
| --- | --- | --- |
| `variable` | `Edm.String` | Une référence du champ de type chaîne (ou une variable de portée sur un champ de collection de chaînes dans le cas où `search.in` est utilisé à l’intérieur d’une expression `any` ou `all`). |
| `valueList` | `Edm.String` | Une chaîne contenant une liste délimitée de valeurs à mettre en correspondance le paramètre `variable`. Si le paramètre `delimiters` n’est pas spécifié, les séparateurs par défaut sont l’espace et la virgule. |
| `delimiters` | `Edm.String` | Une chaîne dans laquelle chaque caractère est traité comme un séparateur lors de l’analyse du paramètre `valueList`. La valeur par défaut de ce paramètre est `' ,'`, ce qui signifie que toutes les valeurs avec des espaces et/ou des virgules entre elles seront séparées. Si vous devez utiliser des séparateurs autres que des espaces et des virgules dans le cas où vos valeurs incluent ces caractères, vous pouvez spécifier d’autres séparateurs, tels que `'|'` dans ce paramètre. |

<a name="bkmk_performance"></a>

### <a name="performance-of-searchin"></a>Performances de `search.in`

Si vous utilisez `search.in`, vous pouvez vous attendre à des temps de réponse inférieurs à la seconde quand le deuxième paramètre contient une liste de plusieurs centaines ou plusieurs milliers de valeurs. Il n’existe pas de limite explicite quant au nombre d’éléments que vous pouvez passer à `search.in`, même si vous êtes néanmoins limité par la taille maximale de la demande. Cependant, la latence augmente en même temps que le nombre de valeurs.

## <a name="examples"></a>Exemples

Recherchez tous les hôtels avec un nom égal à « Sea View motel » ou « Budget hotel ». Les expressions contiennent des espaces, qui sont des séparateurs par défaut. Vous pouvez spécifier un autre séparateur entre guillemets simples comme troisième paramètre de chaîne :  

```odata-filter-expr
    search.in(HotelName, 'Sea View motel,Budget hotel', ',')
```

Recherchez tous les hôtels avec un nom égal à « Sea View motel » ou « Budget hotel » séparés par « | » :

```odata-filter-expr
    search.in(HotelName, 'Sea View motel|Budget hotel', '|')
```

Recherchez tous les hôtels avec des chambres comportant les balises « wifi » ou « baignoire » :

```odata-filter-expr
    Rooms/any(room: room/Tags/any(tag: search.in(tag, 'wifi, tub')))
```

Recherchez une correspondance entre des expressions tirées d’une collection, telles que « heated towel racks » ou « hairdryer included » dans les balises.

```odata-filter-expr
    Rooms/any(room: room/Tags/any(tag: search.in(tag, 'heated towel racks,hairdryer included', ','))
```

Recherchez tous les hôtels sans les balises « motel » ou « cabin » :

```odata-filter-expr
    Tags/all(tag: not search.in(tag, 'motel, cabin'))
```

## <a name="next-steps"></a>Étapes suivantes  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)