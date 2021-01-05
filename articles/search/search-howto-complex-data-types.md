---
title: Comment modéliser les types de données complexes
titleSuffix: Azure Cognitive Search
description: Les structures de données imbriquées ou hiérarchiques peuvent être modélisées dans un index Recherche cognitive Azure à l’aide de types de données ComplexType et Collections.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
tags: complex data types; compound data types; aggregate data types
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/27/2020
ms.openlocfilehash: b0b2dd9904682121c83b22b9029097e7ee57fb11
ms.sourcegitcommit: 6b16e7cc62b29968ad9f3a58f1ea5f0baa568f02
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2020
ms.locfileid: "96303763"
---
# <a name="how-to-model-complex-data-types-in-azure-cognitive-search"></a>Modélisation de types de données complexes dans Recherche cognitive Azure

Les jeux de données externes utilisés pour remplir un index Recherche cognitive Azure peuvent avoir différentes formes. Ils incluent parfois des sous-structures hiérarchiques ou imbriquées. Des exemples incluent les adresses multiples pour un même client, les couleurs et les tailles multiples pour une même référence, les auteurs multiples pour un même livre, etc. En termes de modélisation, ces structures peuvent être désignées sous le nom de types de données *complexes*, *composées*, *composites* or *agrégées*. Le terme utilisé par Recherche cognitive Azure pour ce concept est **type complexe**. Dans Recherche cognitive Azure, les types complexes sont modélisés à l’aide de **champs complexes**. Un champ complexe est un champ qui contient des enfants (sous-champs) qui peuvent être des données de n’importe quel type, notamment d’autres types complexes. Ceci fonctionne d’une manière similaire aux types de données structurées dans un langage de programmation.

Les champs complexes représentent un objet unique dans le document ou un tableau d’objets, selon le type de données. Les champs de type `Edm.ComplexType` représentent des objets uniques, alors que des champs de type `Collection(Edm.ComplexType)` représentent des tableaux d’objets.

Recherche cognitive Azure prend nativement en charge les types et les collections complexes. Ces types vous permettent de modéliser presque n’importe quelle structure JSON dans un index Recherche cognitive Azure. Dans les versions précédentes d’API de Recherche cognitive Azure, seuls les jeux de lignes aplaties ont pu être importés. Dans la version la plus récente, votre index peut mieux correspondre aux données sources. En d’autres termes, si vos données sources contiennent des types complexes, votre index peut également contenir des types complexes.

Pour commencer, nous vous recommandons le [jeu de données d’hôtels](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md), que vous pouvez charger dans l’Assistant **Importer des données** du portail Azure. L’Assistant détecte les types complexes dans la source et suggère un schéma d’index basé sur les structures détectées.

> [!Note]
> La prise en charge des types complexes a commencé à être généralement disponible dans `api-version=2019-05-06`. 
>
> Si votre solution de recherche est basée sur des solutions de contournement antérieures de jeux de données aplatis d’une collection, vous devez modifier votre index pour inclure des types complexes pris en charge dans la dernière version d’API. Pour plus d’informations sur la mise à niveau des versions d’API, consultez [Mettre à niveau vers la dernière version de l’API REST](search-api-migration.md) ou [Mettre à niveau vers la dernière version du kit de développement logiciel (SDK) .NET](search-dotnet-sdk-migration-version-9.md).

## <a name="example-of-a-complex-structure"></a>Exemple de structure complexe

Le document JSON suivant est composé de champs simples et de champs complexes. Les champs complexes, tels que `Address` et `Rooms`, ont des sous-champs. `Address` contient un seul jeu de valeurs pour ces sous-champs, puisqu’il s’agit d’un objet unique dans le document. En revanche, `Rooms` a plusieurs ensembles de valeurs pour ses sous-champs, soit un pour chaque objet dans la collection.


```json
{
  "HotelId": "1",
  "HotelName": "Secret Point Motel",
  "Description": "Ideally located on the main commercial artery of the city in the heart of New York.",
  "Tags": ["Free wifi", "on-site parking", "indoor pool", "continental breakfast"]
  "Address": {
    "StreetAddress": "677 5th Ave",
    "City": "New York",
    "StateProvince": "NY"
  },
  "Rooms": [
    {
      "Description": "Budget Room, 1 Queen Bed (Cityside)",
      "RoomNumber": 1105,
      "BaseRate": 96.99,
    },
    {
      "Description": "Deluxe Room, 2 Double Beds (City View)",
      "Type": "Deluxe Room",
      "BaseRate": 150.99,
    }
    . . .
  ]
}
```

## <a name="indexing-complex-types"></a>Indexation des types complexes

Pendant l’indexation, vous pouvez avoir un maximum de 3 000 éléments dans toutes les collections complexes d’un même document. Un élément d’une collection complexe est membre de cette collection. par conséquent, dans le cas des chambres (la seule collection complexe de l’exemple Hôtel), chaque espace est un élément. Dans l’exemple ci-dessus, si le « Secret Point Motel » contient 500 chambres, le document de l’hôtel aurait 500 éléments de chambre. Pour les collections complexes imbriquées, chaque élément imbriqué est également compté, en plus de l’élément externe (parent).

Cette limite s’applique uniquement aux collections complexes, et non aux types complexes (tels que l’adresse) ou aux collections de chaînes (telles que les balises).

## <a name="creating-complex-fields"></a>Création de champs complexes

Comme avec n’importe quelle définition d’index, vous pouvez utiliser le portail, l’[API REST](/rest/api/searchservice/create-index), ou le kit de développement logiciel [ (SDK) .NET](/dotnet/api/azure.search.documents.indexes.models.searchindex) pour créer un schéma incluant des types complexes. 

L’exemple suivant montre un schéma d’index JSON avec des champs simples, des collections et des types complexes. Notez qu’au sein d’un type complexe, chaque sous-champ a un type et peut avoir des attributs, comme c’est le cas pour les champs de niveau supérieur. Le schéma correspond à l’exemple de données ci-dessus. `Address` est un champ complexe qui n’est pas une collection (un hôtel a une adresse). `Rooms` est un champ de collection complexe (un hôtel a plusieurs chambres).

```json
{
  "name": "hotels",
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
    { "name": "Address", "type": "Edm.ComplexType",
      "fields": [
        { "name": "StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
        { "name": "City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
        { "name": "StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true }
      ]
    },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)",
      "fields": [
        { "name": "Description", "type": "Edm.String", "searchable": true, "analyzer": "en.lucene" },
        { "name": "Type", "type": "Edm.String", "searchable": true },
        { "name": "BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true }
      ]
    }
  ]
}
```

## <a name="updating-complex-fields"></a>Mise à jour de champs complexes

Toutes les [règles de réindexation](search-howto-reindex.md) qui s’appliquent à des champs s’appliquent en général toujours aux champs complexes. Dans le cadre du rappel de quelques règles principales, notons que, contrairement à la plupart des modifications, l’ajout d’un champ à un type de complexe ne nécessite pas une reconstruction de l’index.

### <a name="structural-updates-to-the-definition"></a>Mises à jour structurelles de la définition

Vous pouvez ajouter des sous-champs à un champ complexe à tout moment sans qu’une reconstruction d’index soit nécessaire. Par exemple, l’ajout de « ZipCode » à `Address` ou d’« Amenities » (infrastructures) à `Rooms` est autorisé, tout comme l’ajout d’un champ de niveau supérieur à un index. Les documents existants ont une valeur Null pour les nouveaux champs jusqu'à ce que vous remplissiez explicitement ces champs en mettant à jour de vos données.

Notez qu’au sein d’un type complexe, chaque sous-champ a un type et peut avoir des attributs, comme c’est le cas pour les champs de niveau supérieur

### <a name="data-updates"></a>Mises à jour des données

La mise à jour de documents existants dans un index avec l’action `upload` fonctionne de la même façon pour les champs complexes et simples : tous les champs sont remplacés. Toutefois, `merge` (ou `mergeOrUpload` lorsqu’il est appliqué à un document existant) ne fonctionne pas de la même façon dans tous les champs. Plus précisément, `merge` ne prend pas en charge la fusion d’éléments dans une collection. Cette limitation existe pour les collections de types primitifs et les collections complexes. Pour mettre à jour une collection, vous devez récupérer la valeur de la collection complète, apporter des modifications, puis inclure la nouvelle collection dans la requête de l’API d’index.

## <a name="searching-complex-fields"></a>Recherche de champs complexes

Les expressions de recherche de forme libre fonctionnent comme prévu avec des types complexes. Si n’importe quel champ de recherche ou sous-champ n’importe où dans un document correspond, le document proprement est une correspondance.

Les requêtes sont plus nuancées lorsque vous avez plusieurs termes et opérateurs, et certains termes ont des noms de champs spécifiés, comme cela est possible avec la [syntaxe Lucene](query-lucene-syntax.md). Par exemple, cette requête essaie de faire correspondre deux termes, « Portland » et « OR » à deux sous-champs du champ adresse :

> `search=Address/City:Portland AND Address/State:OR`

Des requêtes de ce type sont *sans corrélation* pour la recherche en texte intégral, contrairement aux filtres. Dans les filtres, les requêtes relatives aux sous-champs d’une collection complexe sont corrélés à l’aide de variables de portée dans [`any` ou `all`](search-query-odata-collection-operators.md). La requête Lucene ci-dessus retourne des documents contenant « Portland, Maine » et « Portland, Oregon », ainsi que d’autres villes d’Oregon. C’est dû au fait que chaque clause s’applique à toutes les valeurs de son champ dans le document entier. Il n’existe donc pas de concept de « sous-élément actuel ». Pour plus d’informations à ce sujet, consultez [Présentation de filtres de collection OData dans Recherche cognitive Azure](search-query-understand-collection-filters.md).

## <a name="selecting-complex-fields"></a>Sélection de champs complexes

Le paramètre `$select` permet de choisir quels champs retourner dans les résultats de la recherche. Pour utiliser ce paramètre afin de sélectionner des sous-champs spécifiques d’un champ complexe, incluez le champ parent et le sous-champ séparés par une barre oblique (`/`).

> `$select=HotelName, Address/City, Rooms/BaseRate`

Les champs doivent être marqués comme récupérables dans l’index si vous souhaitez les afficher dans les résultats de la recherche. Seuls les champs marqués comme récupérable peuvent être utilisés dans une instruction `$select`.

## <a name="filter-facet-and-sort-complex-fields"></a>Filtrer et trier des champs complexes et activer des facettes pour les champs complexes

La même [syntaxe de chemin OData](query-odata-filter-orderby-syntax.md) utilisée pour le filtrage et les recherches par champ peut également être utilisée pour activer des facettes, trier et sélectionner dans le cadre d’une requête de recherche. Pour les types complexes, des règles qui définissent quels sous-champs peuvent être marqués triables ou dotés de facettes activées s’appliquent. Pour plus d’informations sur ces règles, consultez la [référence Créer une API d’index](/rest/api/searchservice/create-index).

### <a name="faceting-sub-fields"></a>Activation de facettes pour les sous-champs

Des facettes peuvent être activées pour n’importe quel sous-champ, sauf s’il est de type `Edm.GeographyPoint` ou `Collection(Edm.GeographyPoint)`.

Le nombre de documents retournés dans les résultats des facettes activées est calculé pour le document parent (un hôtel), pas pour les sous-documents dans une collection complexe (des chambres). Par exemple, supposez qu'un hôtel a 20 suites. Étant donné ce paramètre de facette `facet=Rooms/Type`, le nombre de facettes sera de un pour l’hôtel, pas de 20 pour les chambres.

### <a name="sorting-complex-fields"></a>Tri de champs complexes

Les opérations de tri s’appliquent aux documents (hôtels) et non aux sous-documents (chambres). Lorsque vous avez une collection de type complexe comme des chambres, il est important de savoir que vous ne pouvez pas du tout trier les chambres. En fait, vous ne pouvez trier aucune collection.

Les opérations de tri fonctionnent lorsque les champs ont une valeur unique pour chaque document, que le champ soit un champ simple ou un sous-champ dans un type complexe. Par exemple, `Address/City` est autorisé à être trié, car il n’y a qu’une seule adresse par hôtel. `$orderby=Address/City` triera donc les hôtels par ville.

### <a name="filtering-on-complex-fields"></a>Filtrage sur des champs complexes

Vous pouvez faire référence à des sous-champs d’un champ complexe dans une expression de filtre. Utilisez simplement la même [syntaxe de chemin OData](query-odata-filter-orderby-syntax.md) qui est utilisé pour l’activation de facettes, le tri et la sélection de champs. Par exemple, le filtre suivant retourne tous les hôtels au Canada :

> `$filter=Address/Country eq 'Canada'`

Pour filtrer sur un champ de collection complexe, vous pouvez utiliser une **expression lambda**[avec les opérateurs](search-query-odata-collection-operators.md)`any` et `all`. Dans ce cas, la **variable de portée** de l’expression lambda est un objet avec des sous-champs. Vous pouvez consulter ces sous-champs avec la syntaxe de chemin OData standard. Par exemple, le filtre suivant retourne tous les hôtels avec au moins une chambre de luxe et toutes les chambres non-fumeurs :

> `$filter=Rooms/any(room: room/Type eq 'Deluxe Room') and Rooms/all(room: not room/SmokingAllowed)`

Comme avec les champs simples de niveau supérieur, les sous-champs simples de champs complexe ne peuvent être inclus dans des filtres que si leur attribut **filtrable** est défini sur `true` dans la définition d’index. Pour plus d’informations, consultez la [référence Créer une API d’index](/rest/api/searchservice/create-index).

## <a name="next-steps"></a>Étapes suivantes

Essayer le [jeu de données des hôtels](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/README.md) dans l’Assistant **Importer des données**. Vous avez besoin des informations de connexion de Cosmos DB fournies dans le fichier Lisez-moi pour accéder aux données.

Lorsque vous avez ces informations, votre première étape dans l’Assistant est de créer une nouvelle source de données Azure Cosmos DB. Plus loin dans l’Assistant, lorsque vous accédez à la page d’index cible, vous voyez un index avec des types complexes. Créez et chargez cet index, puis exécutez des requêtes pour comprendre la nouvelle structure.

> [!div class="nextstepaction"]
> [Démarrage rapide : Assistant du portail pour l’importation, l’indexation et les requêtes](search-get-started-portal.md)
