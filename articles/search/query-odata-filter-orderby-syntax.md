---
title: Vue d’ensemble du langage OData
titleSuffix: Azure Cognitive Search
description: Vue d’ensemble du langage OData pour les requêtes de Recherche cognitive Azure filter, select et order by.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/10/2020
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
ms.openlocfilehash: d04311fce81d147a0830918aee1d4a2a9c0808d4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88923396"
---
# <a name="odata-language-overview-for-filter-orderby-and-select-in-azure-cognitive-search"></a>Vue d’ensemble du langage OData pour `$filter`, `$orderby` et `$select` dans la Recherche cognitive Azure

La Recherche cognitive Azure prend en charge un sous-ensemble de la syntaxe des expressions OData pour les expressions **$filter**, **$orderby** et **$select**. Les expressions de filtre sont évaluées lors de l’analyse de la requête, limitant la recherche à des champs spécifiques ou ajoutant des critères de correspondance utilisés lors du parcours des index. Les expressions order by sont appliquées dans une étape de post-traitement sur un jeu de résultats pour trier les documents renvoyés. Les expressions select déterminent les champs du document inclus dans le jeu de résultats. La syntaxe de ces expressions se distingue de la syntaxe de recherche [simple](query-simple-syntax.md) ou [complète](query-lucene-syntax.md) utilisée dans le paramètre **search** et ce, malgré la présence d'un certain chevauchement pour référencer les champs.

Cet article fournit une vue d'ensemble du langage OData utilisé dans les expressions filter, order by et select. Ce langage est présenté de bas en haut, en commençant par les éléments de base. La syntaxe de niveau supérieur pour chaque paramètre est décrite dans un article distinct :

- [Syntaxe $filter](search-query-odata-filter.md)
- [Syntaxe $orderby](search-query-odata-orderby.md)
- [Syntaxe $select](search-query-odata-select.md)

Les expressions OData peuvent être aussi simples que complexes, mais toutes des éléments communs. Dans la Recherche cognitive Azure, les éléments de base d’une expression OData sont les suivants :

- Les **chemins de champs**, qui font référence à des champs spécifiques de votre index.
- Les **constantes**, qui sont les valeurs littérales d’un certain type de données.

> [!NOTE]
> Dans la Recherche cognitive Azure, la terminologie employée diffère à certains égards du langage [OData standard](https://www.odata.org/documentation/). Ce que nous appelons **champ** dans la Recherche cognitive Azure est appelé **propriété** dans OData. De même, **chemin de champ** est appelé **chemin de propriété**. Un **index** contenant des **documents** dans la Recherche cognitive Azure est plus généralement désigné comme un **jeu d’entités** contenant des **entités** dans OData. La terminologie de la Recherche cognitive Azure est utilisée tout au long de cet article.

## <a name="field-paths"></a>Chemins de champs

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire des chemins des champs.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
field_path ::= identifier('/'identifier)*

identifier ::= [a-zA-Z_][a-zA-Z_0-9]*
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#field_path)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Un chemin de champ comprend un ou plusieurs **identificateurs** séparés par des barres obliques. Chaque identificateur correspond à une séquence de caractères qui doit commencer par une lettre ASCII ou un trait de soulignement, et contenir uniquement des lettres ASCII, des chiffres ou des traits de soulignement. Il peut s'agir de lettres majuscules ou minuscules.

Un identificateur peut faire référence à un nom de champ ou à une **variable de portée** dans le contexte d’une [expression de collection](search-query-odata-collection-operators.md) (`any` ou `all`) au sein d'un filtre. Une variable de portée s'apparente à une variable de boucle qui représente l’élément actuel de la collection. Pour les collections complexes, cette variable représente un objet, ce qui explique pourquoi vous pouvez utiliser des chemins de champs pour faire référence aux sous-champs de la variable. Cela s'apparente à la notation par points de nombreux langages de programmation.

Des exemples de chemins de champs sont présentés dans le tableau suivant :

| Chemin de champ | Description |
| --- | --- |
| `HotelName` | Fait référence à un champ d'index de niveau supérieur |
| `Address/City` | Fait référence au sous-champ `City` d'un champ d'index complexe ; `Address` est de type `Edm.ComplexType` dans cet exemple |
| `Rooms/Type` | Fait référence au sous-champ `Type` d'un champ de collection complexe ; `Rooms` est de type `Collection(Edm.ComplexType)` dans cet exemple |
| `Stores/Address/Country` | Fait référence au sous-champ `Country` du sous-champ `Address` d'un champ de collection complexe ; `Stores` est de type `Collection(Edm.ComplexType)` et `Address` de type `Edm.ComplexType` dans cet exemple |
| `room/Type` | Fait référence au sous-champ `Type` de la variable de portée `room`, par exemple, dans l’expression filter `Rooms/any(room: room/Type eq 'deluxe')` |
| `store/Address/Country` | Fait référence au sous-champ `Country` du sous-champ `Address` de la variable de portée `store`, par exemple, dans l’expression filter `Stores/any(store: store/Address/Country eq 'Canada')` |

La signification d’un chemin de champ diffère selon le contexte. Dans les filtres, un chemin de champ fait référence à la valeur d’une *seule instance* de champ dans le document actif. Dans d’autres contextes, tels que **$orderby**, **$select**, ou dans la [recherche par champ de la syntaxe Lucene complète](query-lucene-syntax.md#bkmk_fields), un chemin de champ fait référence au champ à proprement parler. Cette différence n'est pas sans conséquence sur la manière d'utiliser les chemins de champs dans les filtres.

Examinez le chemin de champ `Address/City`. Dans un filtre, il fait référence à une seule ville pour le document actif, comme « San Francisco ». En revanche, `Rooms/Type` fait référence au sous-champ `Type` pour de nombreuses chambres (par exemple, « standard » pour la première chambre, « deluxe » pour la deuxième chambre, etc.). `Rooms/Type` ne faisant pas référence à une *instance unique* du sous-champ `Type`, il ne peut pas être utilisé directement dans un filtre. Au lieu de cela, pour filtrer sur un type de chambre, vous devez utiliser une [expression lambda](search-query-odata-collection-operators.md) avec une variable de portée, telle que :

```odata
Rooms/any(room: room/Type eq 'deluxe')
```

Dans cet exemple, la variable de portée `room` s’affiche dans le chemin de champ `room/Type`. Ainsi, `room/Type` fait référence au type de chambre actuel dans le document actif. Il s’agit d’une instance unique du sous-champ `Type`, et il peut donc être utilisé directement dans le filtre.

### <a name="using-field-paths"></a>Utilisation des chemins de champs

Les chemins de champs sont utilisés dans de nombreux paramètres des [API REST de Recherche cognitive Azure](/rest/api/searchservice/). Le tableau ci-dessous répertorie tous les emplacements où ils peuvent être utilisés, ainsi que les restrictions qui s'y rapportent :

| API | Nom du paramètre | Restrictions |
| --- | --- | --- |
| [Créer](/rest/api/searchservice/create-index) ou [Mettre à jour](/rest/api/searchservice/update-index) l'index | `suggesters/sourceFields` | None |
| [Créer](/rest/api/searchservice/create-index) ou [Mettre à jour](/rest/api/searchservice/update-index) l'index | `scoringProfiles/text/weights` | Peut uniquement faire référence à des champs **pouvant faire l'objet d'une recherche** |
| [Créer](/rest/api/searchservice/create-index) ou [Mettre à jour](/rest/api/searchservice/update-index) l'index | `scoringProfiles/functions/fieldName` | Peuvent uniquement faire référence à des champs **filtrables** |
| [action](/rest/api/searchservice/search-documents) | `search` quand `queryType` est `full` | Peut uniquement faire référence à des champs **pouvant faire l'objet d'une recherche** |
| [action](/rest/api/searchservice/search-documents) | `facet` | Peut uniquement faire référence à des champs **à choix multiples** |
| [action](/rest/api/searchservice/search-documents) | `highlight` | Peut uniquement faire référence à des champs **pouvant faire l'objet d'une recherche** |
| [action](/rest/api/searchservice/search-documents) | `searchFields` | Peut uniquement faire référence à des champs **pouvant faire l'objet d'une recherche** |
| [Suggest](/rest/api/searchservice/suggestions) et [Autocomplete](/rest/api/searchservice/autocomplete) | `searchFields` | Peuvent uniquement faire référence à des champs appartenant à un [suggesteur](index-add-suggesters.md) |
| [Search](/rest/api/searchservice/search-documents), [Suggest](/rest/api/searchservice/suggestions) et [Autocomplete](/rest/api/searchservice/autocomplete) | `$filter` | Peuvent uniquement faire référence à des champs **filtrables** |
| [Search](/rest/api/searchservice/search-documents) et [Suggest](/rest/api/searchservice/suggestions) | `$orderby` | Peuvent uniquement faire référence à des champs **triables** |
| [Search](/rest/api/searchservice/search-documents), [Suggest](/rest/api/searchservice/suggestions) et [Lookup](/rest/api/searchservice/lookup-document) | `$select` | Peuvent uniquement faire référence à des champs **récupérables** |

## <a name="constants"></a>Constantes

Dans OData, les constantes correspondent aux valeurs littérales d'un type [Entity Data Model](/dotnet/framework/data/adonet/entity-data-model) (EDM) donné. Pour obtenir la liste des types pris en charge dans la Recherche cognitive Azure, consultez [Types de données pris en charge](/rest/api/searchservice/supported-data-types). Les constantes des types de collection ne sont pas prises en charge.

Le tableau suivant présente des exemples de constantes pour chaque type de données pris en charge par la Recherche cognitive Azure :

| Type de données | Exemples de constantes |
| --- | --- |
| `Edm.Boolean` | `true`, `false` |
| `Edm.DateTimeOffset` | `2019-05-06T12:30:05.451Z` |
| `Edm.Double` | `3.14159`, `-1.2e7`, `NaN`, `INF`, `-INF` |
| `Edm.GeographyPoint` | `geography'POINT(-122.131577 47.678581)'` |
| `Edm.GeographyPolygon` | `geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))'` |
| `Edm.Int32` | `123`, `-456` |
| `Edm.Int64` | `283032927235` |
| `Edm.String` | `'hello'` |

### <a name="escaping-special-characters-in-string-constants"></a>Échappement des caractères spéciaux dans les constantes de chaîne

Les constantes de chaîne dans OData sont délimitées par des guillemets simples. Si vous devez construire une requête avec une constante de chaîne susceptible de contenir des guillemets simples, vous pouvez placer les guillemets incorporés dans une séquence d’échappement en les doublant.

Par exemple, une expression avec une apostrophe non mise en forme comme « voiture d'Alice » est représentée dans OData en tant que constante de chaîne `'Alice''s car'`.

> [!IMPORTANT]
> Quand vous construisez des filtres programmatiquement, pensez à échapper les constantes de chaîne qui proviennent d’une entrée utilisateur. Vous pouvez ainsi limiter les risques d’[attaques par injection](https://wikipedia.org/wiki/SQL_injection), en particulier lors de l’utilisation de filtres pour implémenter un [filtrage de sécurité](search-security-trimming-for-azure-search.md).

### <a name="constants-syntax"></a>Syntaxe des constantes

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire de la plupart des constantes présentées dans le tableau ci-dessus. La grammaire correspondant aux types géospatiaux est disponible dans [Fonctions géospatiales OData dans la Recherche cognitive Azure](search-query-odata-geo-spatial-functions.md).

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
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
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#constant)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

## <a name="building-expressions-from-field-paths-and-constants"></a>Création d’expressions à partir de chemins de champs et de constantes

Les chemins de champs et les constantes constituent les éléments de base d'une expression OData, mais elles n'en restent pas moins des expressions complètes. Dans la Recherche cognitive Azure, le paramètre **$select** n’est en fait rien d’autre qu’une liste de chemins de champs séparés par des virgules, et **$orderby** n’est pas plus compliqué que **$select**. Si votre index contient un champ de type `Edm.Boolean`, vous pouvez même écrire un filtre qui n’est rien d’autre que le chemin de ce champ. Les constantes `true` et `false` sont également des filtres valides.

Cela étant, la plupart du temps vous aurez besoin d'expressions plus complexes faisant référence à plusieurs champs et constantes. Ces expressions sont générées de différentes manières selon le paramètre.

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire des paramètres **$filter**, **$orderby** et **$select**. Ils sont conçus à partir d'expressions plus simples faisant référence à des chemins de champs et constantes :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
filter_expression ::= boolean_expression

order_by_expression ::= order_by_clause(',' order_by_clause)*

select_expression ::= '*' | field_path(',' field_path)*
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#filter_expression)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Les paramètres **$orderby** et **$select** correspondent à des listes d'expressions plus simples séparées par des virgules. Le paramètre **$filter** est une expression conditionnelle composée de plus simples sous-expressions. Ces sous-expressions sont combinées à l’aide d'opérateurs logiques tels que [`and`, `or` et `not`](search-query-odata-logical-operators.md), d'opérateurs de comparaison tels que [`eq`, `lt`, `gt`, et ainsi de suite](search-query-odata-comparison-operators.md), et d'opérateurs de collection tels que [`any` et `all`](search-query-odata-collection-operators.md).

Les paramètres **$filter**, **$orderby** et **$select** sont abordés plus en détail dans les articles suivants :

- [Syntaxe OData $filter dans la Recherche cognitive Azure](search-query-odata-filter.md)
- [Syntaxe OData $orderby dans la Recherche cognitive Azure](search-query-odata-orderby.md)
- [Syntaxe OData $select dans la Recherche cognitive Azure](search-query-odata-select.md)

## <a name="see-also"></a>Voir aussi  

- [Navigation par facettes dans la Recherche cognitive Azure](search-faceted-navigation.md)
- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)
- [Syntaxe de requête Lucene](query-lucene-syntax.md)
- [Syntaxe de requête simple dans la Recherche cognitive Azure](query-simple-syntax.md)