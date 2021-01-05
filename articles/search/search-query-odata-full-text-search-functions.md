---
title: Informations de référence sur la fonction de recherche en texte intégral OData
titleSuffix: Azure Cognitive Search
description: Fonctions de recherche en texte intégral OData, search.ismatch et search.ismatchscoring dans les requêtes de Recherche cognitive Azure.
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
ms.openlocfilehash: 78f9e4d8fa80fdf74bdb5cd79f4489d12696fcc2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935787"
---
# <a name="odata-full-text-search-functions-in-azure-cognitive-search---searchismatch-and-searchismatchscoring"></a>Fonctions de recherche en texte intégral OData dans Recherche cognitive Azure- `search.ismatch` et `search.ismatchscoring`

Recherche cognitive Azure prend en charge la recherche en texte intégral dans le contexte d’[expressions de filtre OData](query-odata-filter-orderby-syntax.md) via les fonctions `search.ismatch` et `search.ismatchscoring`. Ces fonctions permettent de combiner la recherche en texte intégral avec le filtrage booléen strict qui est impossible avec l’utilisation du paramètre `search` de niveau supérieur de l’[API de recherche](/rest/api/searchservice/search-documents).

> [!NOTE]
> Les fonctions `search.ismatch` et `search.ismatchscoring` sont uniquement prises en charge dans les filtres de l’[API de recherche](/rest/api/searchservice/search-documents). Elles ne sont pas prises en charge dans les API [Suggest](/rest/api/searchservice/suggestions) ou [Autocomplete](/rest/api/searchservice/autocomplete).

## <a name="syntax"></a>Syntaxe

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire des fonctions `search.ismatch` et `search.ismatchscoring` :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
search_is_match_call ::=
    'search.ismatch'('scoring')?'(' search_is_match_parameters ')'

search_is_match_parameters ::=
    string_literal(',' string_literal(',' query_type ',' search_mode)?)?

query_type ::= "'full'" | "'simple'"

search_mode ::= "'any'" | "'all'"
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#search_is_match_call)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

### <a name="searchismatch"></a>search.ismatch

La fonction `search.ismatch` évalue la requête de recherche en texte intégral comme une partie d’une expression de filtre. Les documents qui correspondent à la requête de recherche sont retournés dans le jeu de résultats. Les surcharges suivantes de cette fonction sont disponibles :

- `search.ismatch(search)`
- `search.ismatch(search, searchFields)`
- `search.ismatch(search, searchFields, queryType, searchMode)`

Les paramètres sont définis dans le tableau suivant :

| Nom du paramètre | Type | Description |
| --- | --- | --- |
| `search` | `Edm.String` | La requête de recherche (dans la syntaxe de requête Lucene [simple](query-simple-syntax.md) ou [complète](query-lucene-syntax.md)). |
| `searchFields` | `Edm.String` | Liste séparée par des virgules de champs de recherche où effectuer des recherches ; par défaut, tous les champs avec possibilité de recherche dans l’index. Lorsque vous utilisez la [recherche par champ](query-lucene-syntax.md#bkmk_fields) dans le paramètre `search`, les spécificateurs de champ de la requête Lucene remplacent tous les champs spécifiés dans ce paramètre. |
| `queryType` | `Edm.String` | `'simple'` ou `'full'` ; par défaut `'simple'`. Spécifie le langage de requête utilisé dans le paramètre `search`. |
| `searchMode` | `Edm.String` | `'any'` ou `'all'` ; par défaut `'any'`. Indique si tout ou partie des termes de recherche du paramètre `search`doit correspondre pour que le document soit considéré comme une correspondance. Lorsque vous utilisez les [opérateurs booléens Lucene](query-lucene-syntax.md#bkmk_boolean) dans le paramètre `search`, ils ont priorité sur ce paramètre. |

Tous les paramètres ci-dessus sont équivalents aux [paramètres de la requête de recherche de l’API de recherche](/rest/api/searchservice/search-documents) correspondante.

La fonction `search.ismatch` renvoie une valeur de type `Edm.Boolean`, qui vous permet de la composer avec d’autres sous-expressions de filtre à l’aide des [opérateurs logiques](search-query-odata-logical-operators.md) booléens.

> [!NOTE]
> Recherche cognitive Azure ne prend pas en charge l’utilisation de `search.ismatch` ou `search.ismatchscoring` à l’intérieur des expressions lambda. Cela signifie qu’il n’est pas possible d’écrire des filtres sur les collections d’objets qui peuvent mettre en corrélation les correspondances de recherche en texte intégral avec des correspondances de filtre strict sur le même objet. Pour plus d’informations sur cette restriction et pour obtenir des exemples, consultez [Résolution des problèmes de filtres de collection dans Recherche cognitive Azure](search-query-troubleshoot-collection-filters.md). Pour en savoir plus sur la raison de l’existence de cette restriction, consultez [Présentation des filtres de collection dans Recherche cognitive Azure](search-query-understand-collection-filters.md).


### <a name="searchismatchscoring"></a>Search.ismatchscoring

La fonction `search.ismatchscoring`, comme la fonction `search.ismatch`, renvoie `true` pour les documents qui correspondent à la requête de recherche en texte intégral transmise comme paramètre. La différence est que le score de pertinence des documents correspondant à la requête `search.ismatchscoring` contribue au score global du document, tandis que dans le cas de `search.ismatch`, le score du document n’est pas modifié. Les surcharges suivantes de cette fonction sont disponibles avec des paramètres identiques à ceux de `search.ismatch` :

- `search.ismatchscoring(search)`
- `search.ismatchscoring(search, searchFields)`
- `search.ismatchscoring(search, searchFields, queryType, searchMode)`

Cela signifie que les deux fonctions `search.ismatch` et `search.ismatchscoring` peuvent être utilisées dans la même expression de filtre.

## <a name="examples"></a>Exemples

Rechercher les documents avec le mot « waterfront ». Cette requête de filtre est identique à une [demande de recherche](/rest/api/searchservice/search-documents) avec `search=waterfront`.

```odata-filter-expr
    search.ismatchscoring('waterfront')
```

Rechercher les documents avec le mot « hostel » et une évaluation supérieure ou égale à 4, ou les documents avec le mot « motel » et une évaluation égale à 5. Notez que cette demande ne peut pas être exprimée sans la fonction `search.ismatchscoring`.

```odata-filter-expr
    search.ismatchscoring('hostel') and Rating ge 4 or search.ismatchscoring('motel') and Rating eq 5
```

Rechercher les documents sans le mot « luxury ».

```odata-filter-expr
    not search.ismatch('luxury')
```

Rechercher les documents avec la phrase « ocean view » ou une évaluation égale à 5. La requête `search.ismatchscoring` sera exécutée seulement sur les champs `HotelName` et `Rooms/Description`.

Les documents qui correspondent uniquement à la deuxième clause de la disjonction sont également renvoyés (hôtels avec une `Rating` égale à 5). L’explication est que ces documents ne correspondent à aucune des parties avec score de l’expression : ils sont renvoyés avec un score égal à zéro.

```odata-filter-expr
    search.ismatchscoring('"ocean view"', 'Rooms/Description,HotelName') or Rating eq 5
```

Rechercher les documents où les termes « hotel » et « airport » sont distants de 5 mots ou moins dans la description de l’hôtel, et où fumer n’est pas autorisé dans au moins certaines pièces. Cette requête utilise le [langage de requête Lucene complet](query-lucene-syntax.md).

```odata-filter-expr
    search.ismatch('"hotel airport"~5', 'Description', 'full', 'any') and Rooms/any(room: not room/SmokingAllowed)
```

## <a name="next-steps"></a>Étapes suivantes  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)