---
title: Informations de référence sur la syntaxe d’expression OData
titleSuffix: Azure Cognitive Search
description: Caractéristiques de la grammaire et de la syntaxe formelles pour les expressions OData dans les requêtes de Recherche cognitive Azure.
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
ms.openlocfilehash: 83e9ae4aa68bc0c819d02a0cc6c39758549811cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88928851"
---
# <a name="odata-expression-syntax-reference-for-azure-cognitive-search"></a>Informations de référence sur la syntaxe d’expression OData pour la Recherche cognitive Azure

La Recherche cognitive Azure utilise les [expressions OData](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html) comme paramètres dans l’ensemble de l’API. En règle générale, les expressions OData sont utilisées pour les paramètres `$orderby` et `$filter`. Ces expressions peuvent être complexes et contenir plusieurs clauses, fonctions et opérateurs. Toutefois, les expressions OData telles que les chemins de propriété sont utilisées dans de nombreuses parties de l’API REST de Recherche cognitive Azure, y compris quand elles sont simples. Par exemple, les expressions de chemin d’accès sont utilisées pour faire référence à des sous-champs de champs complexes dans l’ensemble de l’API. C’est notamment le cas lors du référencement de sous-champs dans un [suggesteur](index-add-suggesters.md), une [fonction de scoring](index-add-scoring-profiles.md), le paramètre `$select` ainsi que dans les [recherches par champ dans les requêtes Lucene](query-lucene-syntax.md).

Cet article décrit toutes ces formes d’expressions OData qui utilisent une grammaire formelle. Il existe également un [diagramme interactif](#syntax-diagram) permettant de faciliter l’exploration visuelle de la grammaire.

## <a name="formal-grammar"></a>Grammaire formelle

Nous pouvons décrire le sous-ensemble du langage OData pris en charge par la Recherche cognitive Azure à l’aide d’une grammaire EBNF ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form), ou Forme de Backus-Naur étendue). Les règles sont triées dans l’ordre « décroissant », en commençant par les expressions plus complexes qui sont ensuite décomposées en expressions plus primitives. Les règles de grammaire qui correspondent aux paramètres spécifiques de l’API REST de Recherche cognitive Azure sont situées dans la partie supérieure :

- [`$filter`](search-query-odata-filter.md) : défini par la règle `filter_expression`.
- [`$orderby`](search-query-odata-orderby.md) : défini par la règle `order_by_expression`.
- [`$select`](search-query-odata-select.md) : défini par la règle `select_expression`.
- Chemins d’accès des champs : définis par la règle `field_path`. Les chemins d’accès des champs sont utilisés dans l’ensemble de l’API. Ils peuvent faire référence aux champs de niveau supérieur d’un index ou à des sous-champs présentant un ou plusieurs [champs complexes](search-howto-complex-data-types.md) comme ancêtres.

Après l’EBNF, vous trouverez un [diagramme de syntaxe](https://en.wikipedia.org/wiki/Syntax_diagram) dans lequel vous pouvez naviguer afin d’explorer la grammaire et les relations entre ses règles de manière interactive.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
/* Top-level rules */

filter_expression ::= boolean_expression

order_by_expression ::= order_by_clause(',' order_by_clause)*

select_expression ::= '*' | field_path(',' field_path)*

field_path ::= identifier('/'identifier)*


/* Shared base rules */

identifier ::= [a-zA-Z_][a-zA-Z_0-9]*


/* Rules for $orderby */

order_by_clause ::= (field_path | sortable_function) ('asc' | 'desc')?

sortable_function ::= geo_distance_call | 'search.score()'


/* Rules for $filter */

boolean_expression ::=
    collection_filter_expression
    | logical_expression
    | comparison_expression
    | boolean_literal
    | boolean_function_call
    | '(' boolean_expression ')'
    | variable

/* This can be a range variable in the case of a lambda, or a field path. */
variable ::= identifier | field_path

collection_filter_expression ::=
    field_path'/all(' lambda_expression ')'
    | field_path'/any(' lambda_expression ')'
    | field_path'/any()'

lambda_expression ::= identifier ':' boolean_expression

logical_expression ::=
    boolean_expression ('and' | 'or') boolean_expression
    | 'not' boolean_expression

comparison_expression ::= 
    variable_or_function comparison_operator constant | 
    constant comparison_operator variable_or_function

variable_or_function ::= variable | function_call

comparison_operator ::= 'gt' | 'lt' | 'ge' | 'le' | 'eq' | 'ne'


/* Rules for constants and literals */

constant ::=
    string_literal
    | date_time_offset_literal
    | integer_literal
    | float_literal
    | boolean_literal
    | 'null'

string_literal ::= "'"([^'] | "''")*"'"

date_time_offset_literal ::= date_part'T'time_part time_zone

date_part ::= year'-'month'-'day

time_part ::= hour':'minute(':'second('.'fractional_seconds)?)?

zero_to_fifty_nine ::= [0-5]digit

digit ::= [0-9]

year ::= digit digit digit digit

month ::= '0'[1-9] | '1'[0-2]

day ::= '0'[1-9] | [1-2]digit | '3'[0-1]

hour ::= [0-1]digit | '2'[0-3]

minute ::= zero_to_fifty_nine

second ::= zero_to_fifty_nine

fractional_seconds ::= integer_literal

time_zone ::= 'Z' | sign hour':'minute

sign ::= '+' | '-'

/* In practice integer literals are limited in length to the precision of
the corresponding EDM data type. */
integer_literal ::= digit+

float_literal ::=
    sign? whole_part fractional_part? exponent?
    | 'NaN'
    | '-INF'
    | 'INF'

whole_part ::= integer_literal

fractional_part ::= '.'integer_literal

exponent ::= 'e' sign? integer_literal

boolean_literal ::= 'true' | 'false'


/* Rules for functions */

function_call ::=
    geo_distance_call |
    boolean_function_call

geo_distance_call ::=
    'geo.distance(' variable ',' geo_point ')'
    | 'geo.distance(' geo_point ',' variable ')'

geo_point ::= "geography'POINT(" lon_lat ")'"

lon_lat ::= float_literal ' ' float_literal

boolean_function_call ::=
    geo_intersects_call |
    search_in_call |
    search_is_match_call

geo_intersects_call ::=
    'geo.intersects(' variable ',' geo_polygon ')'

/* You need at least four points to form a polygon, where the first and
last points are the same. */
geo_polygon ::=
    "geography'POLYGON((" lon_lat ',' lon_lat ',' lon_lat ',' lon_lat_list "))'"

lon_lat_list ::= lon_lat(',' lon_lat)*

search_in_call ::=
    'search.in(' variable ',' string_literal(',' string_literal)? ')'

/* Note that it is illegal to call search.ismatch or search.ismatchscoring
from inside a lambda expression. */
search_is_match_call ::=
    'search.ismatch'('scoring')?'(' search_is_match_parameters ')'

search_is_match_parameters ::=
    string_literal(',' string_literal(',' query_type ',' search_mode)?)?

query_type ::= "'full'" | "'simple'"

search_mode ::= "'any'" | "'all'"
```

## <a name="syntax-diagram"></a>Diagramme de syntaxe

Pour explorer visuellement la grammaire du langage OData pris en charge par la Recherche cognitive Azure, vous pouvez vous aider du diagramme de syntaxe interactif :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/)

## <a name="see-also"></a>Voir aussi  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)
- [Syntaxe de requête Lucene](query-lucene-syntax.md)
- [Syntaxe de requête simple dans la Recherche cognitive Azure](query-simple-syntax.md)