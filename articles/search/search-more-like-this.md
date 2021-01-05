---
title: Fonctionnalité de requête moreLikeThis (préversion)
titleSuffix: Azure Cognitive Search
description: Décrit la fonctionnalité moreLikeThis (préversion), qui est disponible dans les versions préliminaires de l’API REST de Recherche cognitive Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 1c2f8058a85bbc0643ed31a7dc910339d0f6d9dd
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94697048"
---
# <a name="morelikethis-preview-in-azure-cognitive-search"></a>moreLikeThis (préversion) dans Recherche cognitive Azure

> [!IMPORTANT] 
> Cette fonctionnalité est actuellement disponible en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). L’[API REST version 2020-06-30-Preview](search-api-preview.md) fournit cette fonctionnalité. Il n’y a actuellement pas de prise en charge du portail ou du SDK .NET.

`moreLikeThis=[key]` est un paramètre de requête dans [L’API Search Documents](/rest/api/searchservice/search-documents) (Recherche de documents) qui recherche des documents similaires au document spécifié par la clé de document. Lorsqu’une requête de recherche est formulée avec `moreLikeThis`, une demande est générée en utilisant les termes de recherche extraits du document donné et qui décrivent le mieux ce document. La demande générée est ensuite utilisée pour effectuer la requête de recherche. Par défaut, le contenu de tous les champs pouvant faire l’objet d’une recherche est pris en compte, à l’exception des champs restreints que vous avez spécifiés à l’aide du paramètre `searchFields`. Le paramètre `moreLikeThis` ne peut pas être utilisé avec le paramètre de recherche, `search=[string]`.

Par défaut, le contenu de tous les champs de niveau supérieur pouvant faire l’objet d’une recherche est pris en compte. Si vous souhaitez spécifier des champs spécifiques à la place, vous pouvez utiliser le paramètre `searchFields`. 

Vous ne pouvez pas utiliser `MoreLikeThis` sur des sous-champs pouvant faire l’objet d’une recherche dans un [type complexe](search-howto-complex-data-types.md).

## <a name="examples"></a>Exemples

Tous les exemples suivants utilisent l’exemple Hôtels tirés du [Guide de démarrage rapide : Créer un index de recherche dans le portail Azure](search-get-started-portal.md).

### <a name="simple-query"></a>Requête simple

La requête suivante recherche les documents dont les champs de description sont les plus proches du champ du document source, tel que spécifié par le paramètre `moreLikeThis`.

```
GET /indexes/hotels-sample-index/docs?moreLikeThis=29&searchFields=Description&api-version=2020-06-30-Preview
```

Dans cet exemple, la requête recherche des hôtels similaires à celui ayant l’`HotelId` 29.
Au lieu d’utiliser HTTP GET, vous pouvez également appeler `MoreLikeThis` avec HTTP POST :

```
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30-Preview
    {
      "moreLikeThis": "29",
      "searchFields": "Description"
    }
```

### <a name="apply-filters"></a>Appliquer des filtres

`MoreLikeThis` peut être combiné avec d’autres paramètres de requête courants comme `$filter`. Par exemple, la requête peut être limitée à des hôtels dont la catégorie est « budget » et pour lesquels l’évaluation est supérieure à 3,5 :

```
GET /indexes/hotels-sample-index/docs?moreLikeThis=20&searchFields=Description&$filter=(Category eq 'Budget' and Rating gt 3.5)&api-version=2020-06-30-Preview
```

### <a name="select-fields-and-limit-results"></a>Sélectionner des champs et limiter les résultats

Le sélecteur `$top` peut être utilisé pour limiter le nombre de résultats devant être retournés dans une requête `MoreLikeThis`. Les champs peuvent aussi être sélectionnés avec `$select`. Ici, les trois premiers hôtels sont sélectionnés avec leur ID, leur nom et leur évaluation : 

```
GET /indexes/hotels-sample-index/docs?moreLikeThis=20&searchFields=Description&$filter=(Category eq 'Budget' and Rating gt 3.5)&$top=3&$select=HotelId,HotelName,Rating&api-version=2020-06-30-Preview
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez utiliser n’importe quel outil de test web pour faire des essais avec cette fonctionnalité.  Nous vous recommandons d’utiliser Postman pour cet exercice.

> [!div class="nextstepaction"]
> [Explorer les API REST de Recherche cognitive Azure](search-get-started-rest.md)