---
title: Ordre OData par référence
titleSuffix: Azure Cognitive Search
description: Documentation de référence de la syntaxe et du langage pour l’utilisation de order-by dans les requêtes Recherche cognitive Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/05/2020
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
ms.openlocfilehash: 83ab2c6b97435ace0d2bc508cbf522600391b60b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88926828"
---
# <a name="odata-orderby-syntax-in-azure-cognitive-search"></a>Syntaxe OData $orderby dans Recherche cognitive Azure

 Vous pouvez utiliser le [paramètre OData **$orderby**](query-odata-filter-orderby-syntax.md) pour appliquer un ordre de tri personnalisé pour les résultats de recherche dans Recherche cognitive Azure. Cet article décrit la syntaxe de **$orderby** en détail. Pour plus d’informations sur l’utilisation de **$orderby** lors de la présentation des résultats de recherche, consultez [Guide pratique pour utiliser les résultats de recherche dans Recherche cognitive Azure](search-pagination-page-layout.md).

## <a name="syntax"></a>Syntaxe

Le paramètre **$orderby** accepte une liste séparée par des virgules de 32  **clauses orderby**. La syntaxe d’une clause orderby est décrite par l’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
order_by_clause ::= (field_path | sortable_function) ('asc' | 'desc')?

sortable_function ::= geo_distance_call | 'search.score()'
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#order_by_clause)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Chaque clause possède des critères de tri, éventuellement suivis d’un ordre de tri (`asc` pour l’ordre croissant ou `desc` pour l’ordre décroissant). Si vous ne spécifiez pas d’ordre, la valeur par défaut est croissant. Si le champ contient des valeurs Null, celles-ci s’affichent en premier si le tri est `asc` et en dernier si le tri est `desc`.

Le critère de tri peut être le trajet d’un champ `sortable`, ou un appel à pour les fonctions [`geo.distance`](search-query-odata-geo-spatial-functions.md) ou [`search.score`](search-query-odata-search-score-function.md).

Si plusieurs documents ont les mêmes critères de tri et que la fonction `search.score` n’est pas utilisée (par exemple si vous triez sur un champ numérique `Rating` et que trois documents ont une évaluation de 4), les regroupements sont effectués par score des documents en ordre décroissant. Quand les scores des documents sont identiques (par exemple quand aucune requête de recherche en texte intégral n’est spécifiée dans la demande), l’ordre relatif des documents regroupés est indéterminé.

Vous pouvez spécifier plusieurs critères de tri. L’ordre des expressions détermine l’ordre de tri final. Par exemple, pour trier par ordre décroissant sur le score, suivi de l’évaluation, la syntaxe est `$orderby=search.score() desc,Rating desc`.

La syntaxe pour `geo.distance` dans **$orderby** est la même que dans **$filter**. Lors de l’utilisation de `geo.distance` dans **$orderby**, le champ auquel elle s’applique doit être de type `Edm.GeographyPoint` et être aussi `sortable`.

La syntaxe pour `search.score` dans **$orderby** est `search.score()`. La fonction `search.score` ne prend aucun paramètre.

## <a name="examples"></a>Exemples

Tirer les hôtels par ordre croissant sur le tarif de base :

```odata-filter-expr
    $orderby=BaseRate asc
```

Trier les hôtels par ordre décroissant sur l’évaluation, puis par ordre croissant sur le tarif de base (rappelez-vous que l’ordre croissant est la valeur par défaut) :

```odata-filter-expr
    $orderby=Rating desc,BaseRate
```

Trier les hôtels par ordre décroissant sur l’évaluation, puis par ordre croissant sur la distance à partir des coordonnées spécifiées :

```odata-filter-expr
    $orderby=Rating desc,geo.distance(Location, geography'POINT(-122.131577 47.678581)') asc
```

Trier les hôtels par ordre décroissant sur search.score et l’évaluation, puis par ordre croissant sur la distance à partir des coordonnées spécifiées. Entre deux d’hôtels avec des scores ou une évaluation identique, le plus proche est listé en premier :

```odata-filter-expr
    $orderby=search.score() desc,Rating desc,geo.distance(Location, geography'POINT(-122.131577 47.678581)') asc
```

## <a name="next-steps"></a>Étapes suivantes  

- [Guide pratique pour utiliser les résultats de recherche dans Recherche cognitive Azure](search-pagination-page-layout.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour la Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)