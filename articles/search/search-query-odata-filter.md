---
title: Informations de référence sur le filtre OData
titleSuffix: Azure Cognitive Search
description: Informations de référence sur le langage OData et syntaxe complète utilisée pour créer des expressions de filtre dans les requêtes Recherche cognitive Azure.
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
ms.openlocfilehash: 0f33b5a28d7c83be7e546c3f61bc517047c51312
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88934852"
---
# <a name="odata-filter-syntax-in-azure-cognitive-search"></a>Syntaxe OData $filter dans Recherche cognitive Azure

Recherche cognitive Azure utilise des [expressions de filtre OData](query-odata-filter-orderby-syntax.md) pour appliquer des critères supplémentaires à une requête de recherche en plus des termes de recherche en texte intégral. Cet article décrit la syntaxe des filtres en détail. Pour obtenir des informations plus générales sur ce que sont les filtres et comment les utiliser pour des scénarios de requête spécifiques, consultez [Filtres dans Recherche cognitive Azure](search-filters.md).

## <a name="syntax"></a>Syntaxe

Un filtre dans le langage OData est une expression booléenne, qui à son tour peut prendre un de différents types d’expression, comme indiqué par l’EBNF ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) suivant :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
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
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#boolean_expression)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Les types d’expressions booléennes incluent les suivants :

- Expressions de filtre de collection utilisant `any` ou `all`. Celles-ci appliquent des critères de filtre aux champs de collection. Pour plus d’informations, consultez [Opérateurs de collection OData dans Recherche cognitive Azure](search-query-odata-collection-operators.md).
- Expressions logiques qui combinent d’autres expressions booléennes à l’aide des opérateurs `and`, `or` et `not`. Pour plus d’informations, consultez [Opérateurs logiques OData dans Recherche cognitive Azure](search-query-odata-logical-operators.md).
- Expressions de comparaison, qui comparent des champs ou variables de plage à des valeurs constantes à l’aide des opérateurs `eq`, `ne`, `gt`, `lt`, `ge` et `le`. Pour plus d’informations, consultez [Opérateurs de comparaison OData dans Recherche cognitive Azure](search-query-odata-comparison-operators.md). Les expressions de comparaison sont également utilisées pour comparer des distances entre des coordonnées géospatiales à l’aide de la fonction `geo.distance`. Pour plus d’informations, consultez [Fonctions géospatiales OData dans Recherche cognitive Azure](search-query-odata-geo-spatial-functions.md).
- Littéraux booléens `true` et `false`. Ces constantes peuvent parfois être utiles lors de la génération de filtre par programmation, mais ne sont pas systématiquement utilisées dans la pratique.
- Appels de fonctions booléennes, notamment :
  - `geo.intersects`, qui teste si un point donné se trouve dans un polygone donné. Pour plus d’informations, consultez [Fonctions géospatiales OData dans Recherche cognitive Azure](search-query-odata-geo-spatial-functions.md).
  - `search.in`, qui compare un champ ou une variable de plage avec chaque valeur d’une liste de valeurs. Pour plus d’informations, consultez [Fonction OData `search.in` dans Recherche cognitive Azure](search-query-odata-search-in-function.md).
  - `search.ismatch` et `search.ismatchscoring`, qui exécutent des opérations de recherche en texte intégral dans un contexte de filtre. Pour plus d’informations, consultez [Fonctions de recherche en texte intégral OData dans Recherche cognitive Azure](search-query-odata-full-text-search-functions.md).
- Chemins d’accès de champ ou variables de plage de type `Edm.Boolean`. Par exemple, si votre index comporte un champ booléen appelé `IsEnabled` et que vous souhaitez retourner tous les documents où ce champ est `true`, votre expression de filtre peut être simplement le nom `IsEnabled`.
- Expressions booléennes entre parenthèses. L’utilisation de parenthèses peut vous aider à déterminer explicitement l’ordre des opérations dans un filtre. Pour plus d’informations sur la priorité par défaut des opérateurs OData, consultez la section suivante.

### <a name="operator-precedence-in-filters"></a>Précédence des opérateurs dans les filtres

Si vous écrivez une expression de filtre sans parenthèses autour de ses sous-expressions, Recherche cognitive Azure l’évalue en fonction d’un ensemble de règles de priorité d’opérateur. Ces règles sont basées sur les opérateurs utilisés pour combiner des sous-expressions. Le tableau suivant répertorie les groupes d’opérateurs dans l’ordre de la priorité la plus élevée à la plus faible :

| Groupe | Opérateur(s) |
| --- | --- |
| Opérateurs logiques | `not` |
| Opérateurs de comparaison | `eq`, `ne`, `gt`, `lt`, `ge`, `le` |
| Opérateurs logiques | `and` |
| Opérateurs logiques | `or` |

Un opérateur qui est plus élevé dans le tableau ci-dessus sera « lié plus étroitement » à ses opérandes que les autres opérateurs. Par exemple, `and` a une priorité plus élevée que `or`, et les opérateurs de comparaison ont une priorité plus élevée que ces deux-ci. Par conséquent, les deux expressions suivantes sont équivalentes :

```odata-filter-expr
    Rating gt 0 and Rating lt 3 or Rating gt 7 and Rating lt 10
    ((Rating gt 0) and (Rating lt 3)) or ((Rating gt 7) and (Rating lt 10))
```

L’opérateur `not` a la priorité la plus élevée de tous, supérieure encore aux opérateurs de comparaison. C’est pourquoi, si vous essayez d’écrire un filtre tel que le suivant :

```odata-filter-expr
    not Rating gt 5
```

Vous obtenez ce message d’erreur :

```text
    Invalid expression: A unary operator with an incompatible type was detected. Found operand type 'Edm.Int32' for operator kind 'Not'.
```

Cette erreur se produit car l’opérateur est associé au champ `Rating` uniquement, qui est de type `Edm.Int32`, et non à l’expression de comparaison entière. La solution consiste à placer l’opérande de `not` entre parenthèses :

```odata-filter-expr
    not (Rating gt 5)
```

<a name="bkmk_limits"></a>

### <a name="filter-size-limitations"></a>Limitations de taille des filtres

Il existe des limites de taille et de complexité des expressions de filtre que vous pouvez envoyer à Recherche cognitive Azure. Les limites sont en gros basées sur le nombre de clauses de votre expression de filtre. Une directive appropriée est que, si vous avez des centaines de clauses, vous risquez de dépasser la limite. Nous vous recommandons de concevoir votre application de telle sorte qu’elle ne génère pas de filtres de taille illimitée.

> [!TIP]
> L’utilisation de [la fonction `search.in`](search-query-odata-search-in-function.md) au lieu de disjonctions longues de comparaisons d’égalité peut permettre d’éviter la limite de clause de filtre, dans la mesure où un appel de fonction est comptabilisé comme une seule clause.

## <a name="examples"></a>Exemples

Rechercher tous les hôtels avec au moins une chambre, un tarif de base inférieur à 200 $ et une note d’évaluation égale ou supérieure à 4 :

```odata-filter-expr
    $filter=Rooms/any(room: room/BaseRate lt 200.0) and Rating ge 4
```

Rechercher tous les hôtels autres que « Sea View Motel » qui ont été rénovés depuis 2010 :

```odata-filter-expr
    $filter=HotelName ne 'Sea View Motel' and LastRenovationDate ge 2010-01-01T00:00:00Z
```

Rechercher tous les hôtels qui ont été rénovés en 2010 ou ultérieurement. Le littéral DateHeure inclut des informations de fuseau horaire pour l’heure standard du Pacifique :  

```odata-filter-expr
    $filter=LastRenovationDate ge 2010-01-01T00:00:00-08:00
```

Rechercher tous les hôtels avec un parking inclus et où toutes les chambres sont non-fumeur :

```odata-filter-expr
    $filter=ParkingIncluded and Rooms/all(room: not room/SmokingAllowed)
```

 \- OR -  

```odata-filter-expr
    $filter=ParkingIncluded eq true and Rooms/all(room: room/SmokingAllowed eq false)
```

Rechercher tous les hôtels de catégorie « Luxury » ou incluant un parking et ayant une évaluation de 5 :  

```odata-filter-expr
    $filter=(Category eq 'Luxury' or ParkingIncluded eq true) and Rating eq 5
```

Rechercher tous les hôtels avec la balise « wifi » dans au moins une chambre (où chaque chambre a des balises stockées dans un champ `Collection(Edm.String)`) :  

```odata-filter-expr
    $filter=Rooms/any(room: room/Tags/any(tag: tag eq 'wifi'))
```

Rechercher tous les hôtels avec n’importe quelles chambres :  

```odata-filter-expr
    $filter=Rooms/any()
```

Rechercher tous les hôtels qui n’ont pas de chambres :

```odata-filter-expr
    $filter=not Rooms/any()
```

Rechercher tous les hôtels dans les 10 kilomètres d’un point de référence donnée (où `Location` est un champ de type `Edm.GeographyPoint`) :

```odata-filter-expr
    $filter=geo.distance(Location, geography'POINT(-122.131577 47.678581)') le 10
```

Rechercher tous les hôtels dans une fenêtre d’affichage décrite sous forme de polygone (où `Location` est un champ de type Edm.GeographyPoint). Le polygone doit être fermé, ce qui signifie que le premier point et le dernier doivent être le même. En outre, [les points doivent être répertoriés dans le sens inverse des aiguilles d’une montre](/rest/api/searchservice/supported-data-types#Anchor_1).

```odata-filter-expr
    $filter=geo.intersects(Location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))')
```

Rechercher tous les hôtels où le champ « Description » est nul. Le champ sera nul s’il n’a jamais été défini ou s’il a été défini explicitement sur la valeur Null :  

```odata-filter-expr
    $filter=Description eq null
```

Recherchez tous les hôtels avec un nom égal à « Sea View motel » ou « Budget hotel ». Ces expressions contiennent des espaces, et l’espace est un séparateur par défaut. Vous pouvez spécifier un autre séparateur entre guillemets simples comme troisième paramètre de chaîne :  

```odata-filter-expr
    $filter=search.in(HotelName, 'Sea View motel,Budget hotel', ',')
```

Recherchez tous les hôtels avec un nom égal à « Sea View motel » ou « Budget hotel » séparés par « | » :  

```odata-filter-expr
    $filter=search.in(HotelName, 'Sea View motel|Budget hotel', '|')
```

Recherchez tous les hôtels où toutes les chambres comportent la balise « wifi » ou « baignoire » :

```odata-filter-expr
    $filter=Rooms/any(room: room/Tags/any(tag: search.in(tag, 'wifi, tub'))
```

Recherchez une correspondance entre des expressions tirées d’une collection, telles que « heated towel racks » ou « hairdryer included » dans les balises.

```odata-filter-expr
    $filter=Rooms/any(room: room/Tags/any(tag: search.in(tag, 'heated towel racks,hairdryer included', ','))
```

Rechercher les documents avec le mot « waterfront ». Cette requête de filtre est identique à une [demande de recherche](/rest/api/searchservice/search-documents) avec `search=waterfront`.

```odata-filter-expr
    $filter=search.ismatchscoring('waterfront')
```

Rechercher les documents avec le mot « hostel » et une évaluation supérieure ou égale à 4, ou les documents avec le mot « motel » et une évaluation égale à 5. Cette requête n’a pas pu être exprimée sans la fonction `search.ismatchscoring` dans la mesure où elle combine la recherche en texte intégral et des opérations de filtre à l’aide de `or`.

```odata-filter-expr
    $filter=search.ismatchscoring('hostel') and rating ge 4 or search.ismatchscoring('motel') and rating eq 5
```

Rechercher les documents sans le mot « luxury ».

```odata-filter-expr
    $filter=not search.ismatch('luxury')
```

Rechercher les documents avec la phrase « ocean view » ou une évaluation égale à 5. La requête `search.ismatchscoring` sera exécutée seulement sur les champs `HotelName` et `Description`. Les documents qui correspondent uniquement à la deuxième clause de la disjonction sont également renvoyés (hôtels avec une `Rating` égale à 5). Ces documents sont renvoyés avec un score égal à zéro pour montrer clairement qu’ils ne correspondent à aucune des parties avec score de l’expression.

```odata-filter-expr
    $filter=search.ismatchscoring('"ocean view"', 'Description,HotelName') or Rating eq 5
```

Recherchez les hôtels où les termes « hôtel » et « aéroport » sont séparés par plus de cinq mots dans la description, et où toutes les chambres sont non-fumeur. Cette requête utilise le [langage de requête Lucene complet](query-lucene-syntax.md).

```odata-filter-expr
    $filter=search.ismatch('"hotel airport"~5', 'Description', 'full', 'any') and not Rooms/any(room: room/SmokingAllowed)
```

## <a name="next-steps"></a>Étapes suivantes  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)