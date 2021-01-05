---
title: Filtrer les résultats de la recherche
titleSuffix: Azure Cognitive Search
description: Filtrez par identité de sécurité d’utilisateur, langue, emplacement géographique ou valeurs numériques pour réduire les résultats de recherche des requêtes dans la Recherche cognitive Azure, un service de recherche dans le cloud hébergé sur Microsoft Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 6d83e5c39f97db49e2cc9b77cc806cff0a1fa6de
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94355982"
---
# <a name="filters-in-azure-cognitive-search"></a>Filtres dans la Recherche cognitive Azure 

Un *filtre* établit des critères pour la sélection des documents sur lesquels doit porter une requête de Recherche cognitive Azure. Une recherche non filtrée inclut tous les documents figurant dans l’index. Un filtre détermine l’étendue d’une requête de recherche en limitant celle-ci à un sous-ensemble de documents. Par exemple, un filtre peut restreindre une recherche en texte intégral à des produits d’une marque ou d’une couleur spécifiques, commercialisés à des niveaux de prix supérieurs à un seuil défini.

Certaines expériences de recherche imposent des exigences de filtre dans le cadre de leur implémentation, mais vous pouvez utiliser des filtres à chaque fois que vous souhaitez limiter la recherche à l’aide de critères *basés sur des valeurs* (par exemple, en définissant l’étendue de la recherche sur le type de produit « livres » de la catégorie « non-fiction » publiés par « Simon & Schuster »).

Si, au lieu de cela, votre objectif est une recherche ciblée sur des *structures* de données spécifiques (par exemple, en définissant l’étendue de la recherche sur un champ contenant des avis de clients), il existe des méthodes alternatives décrites ci-dessous.

## <a name="when-to-use-a-filter"></a>Quand utiliser un filtre

Certains filtres sont fondamentaux pour diverses expériences de recherche. C’est notamment le cas des filtres de « recherche à proximité », de navigation par facettes et de sécurité, qui montrent uniquement les documents qu’un utilisateur est autorisé à afficher. Si vous implémentez l’une de ces expériences, un filtre est requis. C’est le filtre associé à la requête de recherche qui fournit les coordonnées de géolocalisation, la catégorie de facettes sélectionnée par l’utilisateur, ou l’ID de sécurité du demandeur.

Voici des exemples de scénarios :

1. Utilisez un filtre pour découper votre index sur la base de valeurs de données figurant dans l’index. À partir d’un schéma donné, avec une ville, un type de logement et des commodités, vous pouvez créer un filtre permettant de sélectionner explicitement les documents correspondant à vos critères (par exemple, Marseille, appartement, front de mer). 

   Une recherche en texte intégral portant sur les mêmes entrées produit souvent des résultats similaires, mais un filtre est plus précis car il requiert une correspondance exacte entre la condition de filtre et le contenu de votre index. 

2. Utilisez un filtre si l’expérience de recherche intègre une condition de filtre :

   * La [navigation par facettes](search-faceted-navigation.md) utilise un filtre pour re-transmettre la catégorie de facettes sélectionnée par l’utilisateur.
   * La recherche basée sur la géolocalisation utilise un filtre pour transmettre les coordonnées de la localisation actuelle dans des applications de « recherche à proximité ». 
   * Les filtres de sécurité transmettent des identificateurs de sécurité en tant que critères de filtre, où une correspondance dans l’index sert de proxy pour les droits d’accès au document.

3. Utilisez un filtre si vous souhaitez appliquer des critères de recherche sur un champ numérique. 

   Les champs numériques sont récupérables dans le document et peuvent apparaître dans des résultats de recherche, mais ils ne peuvent pas faire l’objet individuellement d’une recherche en texte intégral. Si vous avez besoin de critères de sélection basés sur des données numériques, utilisez un filtre.

### <a name="alternative-methods-for-reducing-scope"></a>Autres méthodes de réduction de l’étendue de recherche

Si vous souhaitez restreindre vos résultats de recherche, les filtres ne sont pas le seul choix possible. Les solutions alternatives suivantes peuvent être plus appropriées selon votre objectif :

 + Le paramètre de requête `searchFields` permet de focaliser une recherche sur des champs spécifiques. Par exemple, si votre index comprend des champs distincts pour les descriptions en anglais et en espagnol, le paramètre searchFields vous permet de cibler les champs à utiliser pour une recherche en texte intégral. 

+ Le paramètre `$select` permet de spécifier les champs à inclure dans un jeu de résultats, ce qui a pour effet de réduire la réponse avant l’envoi de celle-ci à l’application appelante. Ce paramètre n’affine pas la requête et ne réduit pas la collection de documents. En revanche, si votre objectif est d’obtenir une plus petite réponse, ce paramètre constitue une option à envisager. 

Pour plus d’informations sur ces deux paramètres, voir [Rechercher des documents > Demande > Paramètres de requête](/rest/api/searchservice/search-documents#query-parameters).


## <a name="how-filters-are-executed"></a>Comment les filtres sont-ils exécutés ?

Au moment de la requête, un analyseur de filtre accepte les critères en entrée, convertit l’expression en expressions booléennes atomiques sous la forme d’une arborescence de filtres qui est ensuite évaluée sur les champs filtrables dans un index.

Le filtrage se produit en même temps que la recherche. Il permet de qualifier les documents à inclure dans le traitement en aval pour la récupération de documents et le scoring de leur pertinence. En association avec une chaîne de recherche, le filtre réduit efficacement l’ensemble de rappels de l’opération de recherche suivante. Utilisé seul (par exemple, lorsque la chaîne de requête est vide, où `search=*`), le critère de filtre est la seule entrée. 

## <a name="defining-filters"></a>Définition des filtres
Les filtres sont des expressions OData articulées à l’aide d’un [sous-ensemble de la syntaxe OData V4 prise en charge dans la Recherche cognitive Azure](/rest/api/searchservice/odata-expression-syntax-for-azure-search). 

Vous pouvez spécifier un filtre pour chaque opération de **recherche**, mais le filtre lui-même peut inclure plusieurs champs, plusieurs critères et, si vous utilisez une fonction **ismatch**, plusieurs expressions de recherche en texte intégral. Dans une expression de filtre comportant plusieurs parties, vous pouvez spécifier des prédicats dans n’importe quel ordre (soumis aux règles de précédence de l’opérateur). Vous n’obtenez aucun gain sensible des performances si vous tentez de réorganiser les prédicats dans une séquence particulière.

L’une des limites inconditionnelles sur une expression de filtre est la limite de taille maximale de la demande. La demande entière, filtre inclus, peut être un maximum de 16 Mo pour la commande POST ou de 8 Ko pour la commande GET. Le nombre de clauses dans votre expression de filtre est également limité. Une règle empirique est que, si vous avez des centaines de clauses, vous risquez d’atteindre la limite. Nous vous recommandons de concevoir votre application de telle sorte qu’elle ne génère pas de filtres de taille illimitée.

Les exemples suivants illustrent des définitions de filtre prototypiques dans plusieurs API.

```http
# Option 1:  Use $filter for GET
GET https://[service name].search.windows.net/indexes/hotels/docs?api-version=2020-06-30&search=*&$filter=Rooms/any(room: room/BaseRate lt 150.0)&$select=HotelId, HotelName, Rooms/Description, Rooms/BaseRate

# Option 2: Use filter for POST and pass it in the request body
POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "Rooms/any(room: room/BaseRate lt 150.0)",
    "select": "HotelId, HotelName, Rooms/Description, Rooms/BaseRate"
}
```

```csharp
    parameters =
        new SearchParameters()
        {
            Filter = "Rooms/any(room: room/BaseRate lt 150.0)",
            Select = new[] { "HotelId", "HotelName", "Rooms/Description" ,"Rooms/BaseRate"}
        };

    var results = searchIndexClient.Documents.Search("*", parameters);
```

## <a name="filter-usage-patterns"></a>Filtrer les modèles d’utilisation

Les exemples suivants illustrent plusieurs modèles d’utilisation pour des scénarios de filtre. Pour d’autres idées, voir [Syntaxe d’expression OData > Exemples](./search-query-odata-filter.md#examples).

+ **$filter** autonome, sans chaîne de requête, utile lorsque l’expression de filtre est en mesure de qualifier complètement les documents d’intérêt. À défaut de chaîne de requête, il n’y a ni analyse lexicale ou linguistique, ni notation, ni classement. Notez que la chaîne de recherche comporte uniquement un astérisque, ce qui signifie « faire correspondre tous les documents ».

   ```
   search=*&$filter=Rooms/any(room: room/BaseRate ge 60 and room/BaseRate lt 300) and Address/City eq 'Honolulu'
   ```

+ Combinaison de chaîne de requête et de **$filter**, où le filtre crée le sous-ensemble, et la chaîne de requête fournit les entrées de condition de recherche en texte intégral sur le sous-ensemble filtré. L’ajout de termes (théâtres à distance de marche) introduit des scores de recherche dans les résultats, où les documents qui correspondent le mieux aux termes sont mieux classés. Utiliser un filtre avec une chaîne de requête constitue le modèle d’utilisation le plus courant.

   ```
  search=walking distance theaters&$filter=Rooms/any(room: room/BaseRate ge 60 and room/BaseRate lt 300) and Address/City eq 'Seattle'&$count=true
   ```

+ Requêtes composées, séparées par « OR » (ou), chacune avec ses propres critères de filtre (par exemple, « beagle » dans « chien » ou « siamois » dans « chat »). Les expressions combinées utilisant `or` sont évaluées individuellement et la correspondance des documents joints avec chaque expression est retournée avec la réponse. Ce modèle d’utilisation est obtenu via la fonction `search.ismatchscoring`. Vous pouvez également utiliser la version sans scoring, `search.ismatch`.

   ```
   # Match on hostels rated higher than 4 OR 5-star motels.
   $filter=search.ismatchscoring('hostel') and Rating ge 4 or search.ismatchscoring('motel') and Rating eq 5

   # Match on 'luxury' or 'high-end' in the description field OR on category exactly equal to 'Luxury'.
   $filter=search.ismatchscoring('luxury | high-end', 'Description') or Category eq 'Luxury'&$count=true
   ```

  Il est également possible de combiner la recherche en texte intégral via `search.ismatchscoring` avec des filtres utilisant `and` au lieu de `or`. Toutefois, cette opération équivaut à utiliser les paramètres `search` et `$filter` dans une demande de recherche sur le plan fonctionnel. Par exemple, les deux requêtes suivantes génèrent le même résultat :

  ```
  $filter=search.ismatchscoring('pool') and Rating ge 4

  search=pool&$filter=Rating ge 4
  ```

Pour obtenir des instructions complètes sur des cas d’usage spécifiques, consultez les articles suivants :

+ [Filtres de facette](search-filters-facets.md)
+ [Filtres de langage](search-filters-language.md)
+ [Filtrage de sécurité](search-security-trimming-for-azure-search.md) 

## <a name="field-requirements-for-filtering"></a>Conditions requises des champs pour le filtrage

Dans l’API REST, la propriété filterable (filtrable) est *activée* par défaut pour les champs simples. Les champs filtrables augmentent la taille de l’index. Veillez à définir `"filterable": false` pour les champs que vous ne prévoyez pas réellement d’utiliser dans un filtre. Pour plus d’informations sur les paramètres des définitions de champ, voir [Create Index](/rest/api/searchservice/create-index) (Créer un index).

Dans le Kit de développement logiciel (SDK) .NET, la propriété filterable (filtrable) est *désactivée* par défaut. Vous pouvez rendre un champ filtrable en définissant la [propriété IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable) de l’objet [SearchField](/dotnet/api/azure.search.documents.indexes.models.searchfield) correspondant sur `true`. Dans l’exemple ci-dessous, l’attribut est défini sur la propriété `BaseRate` d’une classe de modèle mappant vers la définition d’index.

```csharp
[IsFilterable, IsSortable, IsFacetable]
public double? BaseRate { get; set; }
```

### <a name="making-an-existing-field-filterable"></a>Rendre filtrable un champ existant

Vous ne pouvez pas modifier des champs existants pour les rendre filtrables. À la place, vous devez ajouter un nouveau champ ou régénérer l’index. Pour plus d’informations sur la regénération d’un index ou la façon de remplir à nouveau des champs, consultez [Comment regénérer un index de Recherche cognitive Azure](search-howto-reindex.md).

## <a name="text-filter-fundamentals"></a>Notions de base concernant les filtres de texte

Les filtres de texte comparent les champs de chaîne aux chaînes littérales que vous fournissez dans le filtre. Contrairement à la recherche en texte intégral, les filtres de texte n’appliquent aucune analyse lexicale ou césure de mots. Les comparaisons portent alors uniquement sur des correspondances exactes. Par exemple, supposons un champ *f* contenant les mots « Sunny day » (journée ensoleillée). `$filter=f eq 'sunny day'` sera une correspondance, mais pas `$filter=f eq 'Sunny'`. 

Les chaînes de texte respectent la casse. Il n’y a pas de conversion en minuscules des mots contenant des majuscules. Ainsi, la chaîne `$filter=f eq 'Sunny day'` ne permet pas de trouver « sunny day ».

### <a name="approaches-for-filtering-on-text"></a>Approches pour le filtrage de texte

| Approche | Description | Quand l’utiliser |
|----------|-------------|-------------|
| [`search.in`](search-query-odata-search-in-function.md) | Une fonction qui compare un champ à une liste délimitée de chaînes. | Cette fonction est recommandée pour les [filtres de sécurité](search-security-trimming-for-azure-search.md) et pour tous les filtres dans lesquels plusieurs valeurs de texte brut doivent être comparées à un champ de chaîne. La fonction **search.in** est conçue pour fonctionner rapidement. Elle est donc beaucoup plus rapide qu’une comparaison explicite du champ à chaque chaîne à l’aide de `eq` et `or`. | 
| [`search.ismatch`](search-query-odata-full-text-search-functions.md) | Fonction permettant de combiner des opérations de recherche en texte intégral avec des opérations de filtre strictement booléen dans une même expression de filtre. | Utilisez **search.ismatch** (ou son équivalent **search.ismatchscoring** pour le scoring) lorsque vous souhaitez utiliser plusieurs combinaisons de filtres et de recherches dans une seule demande. Vous pouvez également l’utiliser pour un filtre *contains* afin de filtrer sur une chaîne partielle figurant à l’intérieur d’une chaîne de plus grande taille. |
| [`$filter=field operator string`](search-query-odata-comparison-operators.md) | Expression définie par l’utilisateur composée de champs, d’opérateurs et de valeurs. | Utilisez-la lorsque vous souhaitez rechercher des correspondances exactes entre un champ de chaîne et une valeur de chaîne. |

## <a name="numeric-filter-fundamentals"></a>Notions de base concernant les filtres numériques

Les champs numériques ne sont pas de type `searchable` (il n’est pas possible d’y effectuer une recherche) dans le contexte d’une recherche en texte intégral. Seules des chaînes peuvent faire l’objet d’une recherche en texte intégral. Par exemple, le terme de recherche 99,99 ne permet pas d’obtenir la liste des articles dont le prix est 99,99. Au lieu de cela, vous obtenez la liste des documents dont des champs de chaîne contiennent le nombre 99. Par conséquent, si vous avez des données numériques, l’hypothèse est que vous allez les utiliser pour des filtres, dont des plages, des facettes, des groupes, etc. 

Les documents contenant des champs numériques (prix, taille, référence, ID, etc.) fournissent ces valeurs dans les résultats de recherche si ces champs sont marqués en tant que `retrievable`. Ce qu’il importe de souligner ici, c’est que la recherche en texte intégral proprement dite ne s’applique pas aux champs de type numérique.

## <a name="next-steps"></a>Étapes suivantes

Tout d’abord, essayez l’**Explorateur de recherche** dans le portail pour soumettre des requêtes avec des paramètres de **$filter**. L’[index d’exemple immobilier](search-get-started-portal.md) fournit des résultats intéressants pour les requêtes filtrées suivantes lorsque vous les collez dans la barre de recherche :

```
# Geo-filter returning documents within 5 kilometers of Redmond, Washington state
# Use $count=true to get a number of hits returned by the query
# Use $select to trim results, showing values for named fields only
# Use search=* for an empty query string. The filter is the sole input

search=*&$count=true&$select=description,city,postCode&$filter=geo.distance(location,geography'POINT(-122.121513 47.673988)') le 5

# Numeric filters use comparison like greater than (gt), less than (lt), not equal (ne)
# Include "and" to filter on multiple fields (baths and bed)
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=baths gt 3 and beds gt 4

# Text filters can also use comparison operators
# Wrap text in single or double quotes and use the correct case
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=city gt 'Seattle'
```

Pour utiliser d’autres exemples, voir [Syntaxe d’expression de filtre OData > Exemples](./search-query-odata-filter.md#examples).

## <a name="see-also"></a>Voir aussi

+ [Fonctionnement de la recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md)
+ [API REST de recherche de documents](/rest/api/searchservice/search-documents)
+ [Syntaxe de requête simple](/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [Syntaxe de requête Lucene](/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [Types de données pris en charge](/rest/api/searchservice/supported-data-types)