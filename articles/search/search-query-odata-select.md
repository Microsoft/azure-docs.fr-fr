---
title: Référence Select OData
titleSuffix: Azure Cognitive Search
description: Référence de syntaxe et de langage pour la sélection explicite de champs à retourner dans les résultats de recherche des requêtes Recherche cognitive Azure.
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
ms.openlocfilehash: 54b6ae227fc4b3b951717799660543c02874dda0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88919656"
---
# <a name="odata-select-syntax-in-azure-cognitive-search"></a>Syntaxe OData $select dans la Recherche cognitive Azure

 Vous pouvez utiliser le paramètre [ **$select** OData](query-odata-filter-orderby-syntax.md) pour choisir les champs à inclure dans les résultats de recherche à partir de la Recherche cognitive Azure. Cet article décrit la syntaxe de **$select** en détail. Pour plus d’informations sur l’utilisation de **$select** lors de la présentation des résultats de recherche, consultez [Guide pratique pour utiliser les résultats de la recherche dans la Recherche cognitive Azure](search-pagination-page-layout.md).

## <a name="syntax"></a>Syntaxe

Le paramètre **$select** détermine quels champs de chaque document sont renvoyés dans le jeu de résultats de la requête. L’extension EBNF suivante ([Extended Backus-Naur Form](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) définit la grammaire du paramètre **$select** :

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
select_expression ::= '*' | field_path(',' field_path)*

field_path ::= identifier('/'identifier)*
```

Un diagramme de syntaxe interactif est également disponible :

> [!div class="nextstepaction"]
> [Diagramme de syntaxe OData pour la Recherche cognitive Azure](https://azuresearch.github.io/odata-syntax-diagram/#select_expression)

> [!NOTE]
> Consultez [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md) pour l’extension EBNF complète.

Le paramètre **$select** se présente sous deux formes :

1. Une seule étoile (`*`), indiquant que tous les champs récupérables doivent être renvoyés, ou
1. Une liste de chemins d’accès aux champs séparés par des virgules identifiant les champs à renvoyer.

Lorsque vous utilisez la deuxième forme, vous pouvez uniquement spécifier les champs récupérables dans la liste.

Si vous répertoriez un champ complexe sans spécifier explicitement ses sous-champs, tous les sous-champs récupérables figureront dans le jeu de résultats de la requête. Par exemple, supposons que votre index comporte un champ `Address` avec les sous-champs `Street`, `City`, et `Country` qui sont tous récupérables. Si vous spécifiez `Address` dans **$select**, les résultats de la requête incluront les trois sous-champs.

## <a name="examples"></a>Exemples

Incluez champs `HotelId`, `HotelName`, et `Rating` de niveau supérieur dans les résultats, ainsi que le sous-champ `City` de `Address` :

```odata-filter-expr
    $select=HotelId, HotelName, Rating, Address/City
```

Le résultat peut ressembler à cet exemple :

```json
{
  "HotelId": "1",
  "HotelName": "Secret Point Motel",
  "Rating": 4,
  "Address": {
    "City": "New York"
  }
}
```

Incluez le champ `HotelName` de niveau supérieur dans les résultats, ainsi que tous les sous-champs de `Address` et les sous-champs `Type` et `BaseRate` de chaque objet de la collection `Rooms` :

```odata-filter-expr
    $select=HotelName, Address, Rooms/Type, Rooms/BaseRate
```

Le résultat peut ressembler à cet exemple :

```json
{
  "HotelName": "Secret Point Motel",
  "Rating": 4,
  "Address": {
    "StreetAddress": "677 5th Ave",
    "City": "New York",
    "StateProvince": "NY",
    "Country": "USA",
    "PostalCode": "10022"
  },
  "Rooms": [
    {
      "Type": "Budget Room",
      "BaseRate": 9.69
    },
    {
      "Type": "Budget Room",
      "BaseRate": 8.09
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes  

- [Guide pratique pour utiliser les résultats de la recherche dans la Recherche cognitive Azure](search-pagination-page-layout.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)