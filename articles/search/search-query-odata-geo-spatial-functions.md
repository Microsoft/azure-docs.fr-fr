---
title: Informations de référence sur la fonction géospatiale OData
titleSuffix: Azure Cognitive Search
description: Documentation de syntaxe et de référence pour l’utilisation de fonctions géospatiales OData dans les requêtes Recherche cognitive Azure geo.distance et geo.intersects.
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
ms.openlocfilehash: 376cece922ca424ec78011224852b1fa5499da16
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88934835"
---
# <a name="odata-geo-spatial-functions-in-azure-cognitive-search---geodistance-and-geointersects"></a>Fonctions géospatiales OData dans Recherche cognitive Azure - `geo.distance` et `geo.intersects`

Recherche cognitive Azure prend en charge les requêtes géospatiales dans les [expressions de filtre OData](query-odata-filter-orderby-syntax.md) via les fonctions `geo.distance` et `geo.intersects`. La fonction `geo.distance` renvoie la distance en kilomètres entre deux points, l’un étant un champ ou une variable de portée et l’autre une constante passée comme partie du filtre. La fonction `geo.intersects` renvoie `true` si un point donné se trouve dans un polygone donné, où le point est un champ ou une variable de portée et où le polygone est spécifié sous la forme d’une constante passée comme partie du filtre.

La fonction `geo.distance` peut également être utilisée dans le paramètre [ **$orderby**](search-query-odata-orderby.md) pour trier les résultats de recherche par distance à partir d’un point donné. La syntaxe pour `geo.distance` dans **$orderby** est la même que dans **$filter**. Lors de l’utilisation de `geo.distance` dans **$orderby**, le champ auquel elle s’applique doit être de type `Edm.GeographyPoint` et être aussi **triable**.

> [!NOTE]
> Si vous spécifiez `geo.distance` dans le paramètre **$orderby**, le champ que vous passez à la fonction doit contenir un seul point géographique. Autrement dit, il doit être de type `Edm.GeographyPoint` et pas de type `Collection(Edm.GeographyPoint)`. Le tri sur les champs de collection n’est pas possible dans Recherche cognitive Azure.

## <a name="syntax"></a>Syntaxe

L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire des fonctions `geo.distance` et `geo.intersects`, ainsi que les valeurs géospatiales sur lequel elles fonctionnent :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
geo_distance_call ::=
    'geo.distance(' variable ',' geo_point ')'
    | 'geo.distance(' geo_point ',' variable ')'

geo_point ::= "geography'POINT(" lon_lat ")'"

lon_lat ::= float_literal ' ' float_literal

geo_intersects_call ::=
    'geo.intersects(' variable ',' geo_polygon ')'

/* You need at least four points to form a polygon, where the first and
last points are the same. */
geo_polygon ::=
    "geography'POLYGON((" lon_lat ',' lon_lat ',' lon_lat ',' lon_lat_list "))'"

lon_lat_list ::= lon_lat(',' lon_lat)*
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#geo_distance_call)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

### <a name="geodistance"></a>geo.distance

La fonction `geo.distance` prend deux paramètres de type `Edm.GeographyPoint` et renvoie une valeur `Edm.Double` qui correspond à la distance entre eux en kilomètres. Ceci diffère des autres services qui prennent en charge des opérations géospatiales OData, qui renvoient généralement les distances en mètres.

L’un des paramètres de `geo.distance` doit être une constante de point géographique et l’autre un chemin d’accès à un champ (ou une variable de portée dans le cas d’un filtre effectuant une itération sur un champ de type `Collection(Edm.GeographyPoint)`). Peu importe l’ordre de ces paramètres.

La constante de point géographique est au format `geography'POINT(<longitude> <latitude>)'`, où la longitude et latitude sont des constantes numériques.

> [!NOTE]
> Quand vous utilisez `ge` dans un filtre, vous devez comparer la distance renvoyée par la fonction avec une constante en utilisant `geo.distance`, `lt`, `le` ou `gt`. Les opérateurs `eq` et `ne` ne sont pas pris en charge lors de la comparaison de distances. Voici par exemple une utilisation correcte de `geo.distance` : `$filter=geo.distance(location, geography'POINT(-122.131577 47.678581)') le 5`.

### <a name="geointersects"></a>geo.intersects

La fonction `geo.intersects` prend une variable de type `Edm.GeographyPoint` et une constante `Edm.GeographyPolygon` et renvoie un `Edm.Boolean` -- `true` si le point se trouve dans les limites du polygone, `false` dans le cas contraire.

Le polygone est une surface en deux dimensions stockée sous la forme d’une séquence de points définissant un cadre englobant (voir [les exemples](#examples) ci-dessous). Le polygone doit être fermé, ce qui signifie que le premier point et le dernier doivent être le même. [Les points d’un polygone doivent être dans le sens antihoraire](/rest/api/searchservice/supported-data-types#Anchor_1).

### <a name="geo-spatial-queries-and-polygons-spanning-the-180th-meridian"></a>Requêtes géospatiales et polygones couvrant le 180{1}e{2} méridien

Pour de nombreuses bibliothèques de requêtes géospatiales, la formulation d’une requête incluant le 180{1}e{2} méridien (près de la ligne de changement de date) est hors limites ou nécessite une solution de contournement, comme diviser le polygone en deux parties, une de chaque côté du méridien.

Dans Recherche cognitive Azure, les requêtes géospatiales incluant une longitude de 180 degrés fonctionnent normalement si la forme de la requête est rectangulaire et que vos coordonnées s’alignent sur une disposition en grille le long de la longitude et de la latitude (par exemple `geo.intersects(location, geography'POLYGON((179 65, 179 66, -179 66, -179 65, 179 65))'`). Sinon, pour les formes non rectangulaires ou non alignées, envisagez l’approche par division du polygone.  

### <a name="geo-spatial-functions-and-null"></a>Fonctions géospatiales et `null`

Comme tous les autres champs qui ne sont pas des collections dans Recherche cognitive Azure, les champs de type `Edm.GeographyPoint` peuvent contenir des valeurs `null`. Lorsque Recherche cognitive Azure évalue `geo.intersects` pour un champ qui est `null`, le résultat sera toujours `false`. Le comportement de `geo.distance` dans ce cas dépend du contexte :

- Dans les filtres, la valeur `geo.distance` d’un champ `null` donne le résultat `null`. Cela signifie que le document ne correspond pas, car `null` comparé à n’importe quelle valeur non null a pour résultat `false`.
- Lors du tri des résultats à l’aide de **$orderby**, la valeur `geo.distance` d’un champ `null` donne pour résultat la distance maximale possible. Les documents comportant un tel champ auront un niveau de tri inférieur par rapport aux autres lorsque le sens de tri `asc` est utilisé (par défaut) et supérieur à tous les autres utilisateurs lorsque le sens est `desc`.

## <a name="examples"></a>Exemples

### <a name="filter-examples"></a>Exemples de filtres

Rechercher tous les hôtels dans les 10 kilomètres d’un point de référence donnée (où l’emplacement est un champ de type `Edm.GeographyPoint`) :

```odata-filter-expr
    geo.distance(location, geography'POINT(-122.131577 47.678581)') le 10
```

Rechercher tous les hôtels dans une fenêtre d’affichage décrite sous forme de polygone (où l’emplacement est un champ de type `Edm.GeographyPoint`). Notez que le polygone est fermé (les définitions du premier et du dernier point doivent les mêmes) et [les points doivent être listés dans le sens antihoraire](/rest/api/searchservice/supported-data-types#Anchor_1).

```odata-filter-expr
    geo.intersects(location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))')
```

### <a name="order-by-examples"></a>Exemples de clause Order by

Trier les hôtels par ordre décroissant sur `rating`, puis par ordre croissant sur la distance à partir des coordonnées spécifiées :

```odata-filter-expr
    rating desc,geo.distance(location, geography'POINT(-122.131577 47.678581)') asc
```

Trier les hôtels par ordre décroissant sur `search.score` et `rating`, puis par ordre croissant sur la distance à partir des coordonnées spécifiées, de sorte qu’entre deux d’hôtels avec une évaluation identique, le plus proche soit listé en premier :

```odata-filter-expr
    search.score() desc,rating desc,geo.distance(location, geography'POINT(-122.131577 47.678581)') asc
```

## <a name="next-steps"></a>Étapes suivantes  

- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)