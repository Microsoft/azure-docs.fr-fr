---
title: Présentation des filtres de collection OData
titleSuffix: Azure Cognitive Search
description: Découvrez les mécanismes de fonctionnement des filtres de collection OData dans les requêtes Azure Cognitive Search, y compris les limitations et les comportements propres aux collections.
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
ms.openlocfilehash: 6af0f2b5221a737687578e939c14cecf3be14509
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88932914"
---
# <a name="understanding-odata-collection-filters-in-azure-cognitive-search"></a>Présentation des filtres de collection OData dans Recherche cognitive Azure

Pour appliquer un [filtre](query-odata-filter-orderby-syntax.md) sur des champs de collection dans la Recherche cognitive Azure, vous pouvez utiliser les [opérateurs `any` et `all`](search-query-odata-collection-operators.md) avec des **expressions lambda**. Les expressions lambda sont des expressions booléennes qui font référence à une **variable de portée**. Les opérateurs `any` et `all` sont analogues à une boucle `for` dans la plupart des langages de programmation, avec la variable de portée jouant le rôle de variable de boucle et l’expression lambda en tant que corps de la boucle. La variable de portée prend la valeur « actuelle » de la collection pendant l’itération de la boucle.

Du moins, c’est ainsi qu’elle fonctionne sur le plan conceptuel. En réalité, Recherche cognitive Azure implémente les filtres d’une manière très différente du fonctionnement des boucles `for`. Dans l’idéal, cette différence est invisible pour vous, mais elle ne l’est pas dans certaines situations. Le résultat final est qu’il existe des règles que vous devez suivre lors de l’écriture d’expressions lambda.

Cet article explique pourquoi les règles applicables aux filtres de collection existent en explorant la façon dont Recherche cognitive Azure Search exécute ces filtres. Si vous écrivez des filtres avancés avec des expressions lambda complexes, cet article peut vous être utile pour améliorer votre compréhension des possibilités offertes par les filtres et pourquoi.

Pour plus d’informations sur ce que sont les règles applicables aux filtres de collection, notamment des exemples, consultez [Résolution des problèmes liés aux filtres de collection OData dans Recherche cognitive Azure](search-query-troubleshoot-collection-filters.md).

## <a name="why-collection-filters-are-limited"></a>Raisons de la limitation des filtres de collection

Il existe trois raisons sous-jacentes pour lesquelles toutes les fonctionnalités de filtre ne sont pas prises en charge pour tous les types de collections :

1. Seuls certains opérateurs sont pris en charge pour certains types de données. Par exemple, cela n’a aucun sens de comparer les valeurs booléennes `true` et `false` à l’aide de `lt`, `gt`, etc.
1. Recherche cognitive Azure ne prend pas en charge la **recherche corrélée** sur des champs de type `Collection(Edm.ComplexType)`.
1. Recherche cognitive Azure utilise des index inversés pour exécuter des filtres sur tous les types de données, dont les collections.

La première raison est simplement une conséquence de la façon dont le langage OData et le système de type EDM sont définis. Les deux dernières sont expliquées de façon plus détaillée dans le reste de cet article.

## <a name="correlated-versus-uncorrelated-search"></a>Recherche corrélée et recherche non corrélée

Quand vous appliquez plusieurs critères de filtre sur une collection d’objets complexes, les critères sont **corrélés** car ils s’appliquent à *chaque objet de la collection*. Par exemple, le filtre suivant retourne les hôtels ayant au moins une chambre de luxe (deluxe) dont le tarif est inférieur à 100 :

```odata-filter-expr
    Rooms/any(room: room/Type eq 'Deluxe Room' and room/BaseRate lt 100)
```

Si le filtrage a été *sans corrélation*, le filtre ci-dessus peut retourner les hôtels où une chambre est de luxe (deluxe) et une autre chambre dispose d’un tarif de base inférieur à 100. Cela n’aurait pas de sens, car les deux clauses de l’expression lambda s’appliquent à la même variable de portée, à savoir `room`. C’est pourquoi ces filtres sont mis en corrélation.

Toutefois, pour la recherche en texte intégral, il n’existe aucun moyen de faire référence à une variable de portée spécifique. Si vous utilisez une recherche par champ pour émettre une [requête complète Lucene](query-lucene-syntax.md) comme celle-ci :

```odata-filter-expr
    Rooms/Type:deluxe AND Rooms/Description:"city view"
```

vous pouvez obtenir les hôtels où une chambre est de luxe (deluxe) et où une autre chambre mentionne « city view » (vue sur la ville) dans la description. Par exemple, le document ci-dessous avec comme `Id` la valeur `1` correspondrait à la requête :

```json
{
  "value": [
    {
      "Id": "1",
      "Rooms": [
        { "Type": "deluxe", "Description": "Large garden view suite" },
        { "Type": "standard", "Description": "Standard city view room" }
      ]
    },
    {
      "Id": "2",
      "Rooms": [
        { "Type": "deluxe", "Description": "Courtyard motel room" }
      ]
    }
  ]
}
```

La raison est que `Rooms/Type` fait référence à tous les termes analysés du champ `Rooms/Type` dans la totalité du document, et il en est de même pour `Rooms/Description`, comme indiqué dans les tableaux ci-dessous.

Façon dont `Rooms/Type` est stocké pour la recherche en texte intégral :

| Terme dans `Rooms/Type` | ID de document |
| --- | --- |
| deluxe | 1, 2 |
| standard | 1 |

Façon dont `Rooms/Description` est stocké pour la recherche en texte intégral :

| Terme dans `Rooms/Description` | ID de document |
| --- | --- |
| courtyard | 2 |
| city | 1 |
| garden | 1 |
| large | 1 |
| motel | 2 |
| room | 1, 2 |
| standard | 1 |
| suite | 1 |
| vue | 1 |

Par conséquent, contrairement au filtre ci-dessus, qui dit : « Trouver une correspondance avec les documents où une chambre a un `Type` égal à « Deluxe Room » (Chambre de luxe) et où **cette même chambre** a un `BaseRate` (Tarif de base) inférieur à 100 », la requête de recherche indique « Trouver une correspondance avec les documents où `Rooms/Type` (Chambres/Type) contient le terme « deluxe » (de luxe) et où `Rooms/Description` (Chambres/Description) contient l’expression « city view » (vue sur la ville). Il n’existe aucun concept de chambres individuelles dont les champs peuvent être mis en corrélation dans ce dernier cas.

> [!NOTE]
> Si vous voulez voir la prise en charge de la recherche corrélée ajoutée à Recherche cognitive Azure, votez pour [cet élément User Voice](https://feedback.azure.com/forums/263029-azure-search/suggestions/37735060-support-correlated-search-on-complex-collections).

## <a name="inverted-indexes-and-collections"></a>Index inversés et collections

Vous avez peut-être remarqué qu’il existe beaucoup moins de restrictions concernant les expressions lambda sur les collections complexes que pour les collections simples comme `Collection(Edm.Int32)`, `Collection(Edm.GeographyPoint)`, etc. En fait, Recherche cognitive Azure stocke les collections complexes en tant que collections réelles de sous-documents, tandis que les collections simples ne sont pas du tout stockées en tant que collections.

Par exemple, considérez un champ de collection de chaînes filtrable comme `seasons` dans un index destiné à un détaillant en ligne. Certains documents chargés dans cet index peuvent se présenter comme suit :

```json
{
  "value": [
    {
      "id": "1",
      "name": "Hiking boots",
      "seasons": ["spring", "summer", "fall"]
    },
    {
      "id": "2",
      "name": "Rain jacket",
      "seasons": ["spring", "fall", "winter"]
    },
    {
      "id": "3",
      "name": "Parka",
      "seasons": ["winter"]
    }
  ]
}
```

Les valeurs du champ `seasons` sont stockées dans une structure appelée **index inversé**, qui ressemble à ceci :

| Terme | ID de document |
| --- | --- |
| spring | 1, 2 |
| summer | 1 |
| fall | 1, 2 |
| winter | 2, 3 |

Cette structure de données est conçue pour répondre très rapidement à une question : Dans quels documents un terme donné apparaît-il ? Réponse à cette question équivaut davantage à un contrôle d’égalité simple qu’à une boucle sur une collection. En fait, c’est la raison pour laquelle, pour les collections de chaînes, Recherche cognitive Azure autorise uniquement `eq` comme opérateur de comparaison dans une expression lambda pour `any`.

En partant de l’égalité, nous allons ensuite voir comment il est possible de combiner plusieurs contrôles d’égalité sur la même variable de plage avec `or`. Cela fonctionne grâce à l’algèbre et à [la propriété distributive des quantificateurs](https://en.wikipedia.org/wiki/Existential_quantification#Negation). Cette expression :

```odata-filter-expr
    seasons/any(s: s eq 'winter' or s eq 'fall')
```

équivaut à :

```odata-filter-expr
    seasons/any(s: s eq 'winter') or seasons/any(s: s eq 'fall')
```

et chacune des deux sous-expressions `any` peut être exécutée efficacement à l’aide de l’index inversé. De plus, grâce à [la loi de la négation des quantificateurs](https://en.wikipedia.org/wiki/Existential_quantification#Negation), cette expression :

```odata-filter-expr
    seasons/all(s: s ne 'winter' and s ne 'fall')
```

équivaut à :

```odata-filter-expr
    not seasons/any(s: s eq 'winter' or s eq 'fall')
```

C’est pourquoi il est possible d’utiliser `all` avec `ne` et `and`.

> [!NOTE]
> Même si ce document ne contient pas de présentation détaillée, ces mêmes principes s’étendent également aux [tests de distance et d’intersection pour les collections de points de données géospatiales](search-query-odata-geo-spatial-functions.md). C’est pourquoi, dans `any` :
>
> - `geo.intersects` ne peut pas être inversé
> - `geo.distance` doit être comparé à l’aide de `lt` ou de `le`
> - Les expressions doivent être combinées avec `or`, et non `and`
>
> Les règles inverses s’appliquent pour `all`.

Un plus large choix d’expressions sont autorisées lors du filtrage sur des collections de types de données qui prennent en charge les opérateurs `lt`, `gt`, `le` et `ge`, comme `Collection(Edm.Int32)` par exemple. Plus précisément, vous pouvez utiliser `and` ainsi que `or` dans `any`, à condition que les expressions de comparaison sous-jacentes soient combinées en **comparaisons de plages** à l’aide de `and`, qui sont ensuite combinées à l’aide de `or`. Cette structure d’expressions booléennes est appelée [forme disjonctive normale (FND)](https://en.wikipedia.org/wiki/Disjunctive_normal_form), également appelée « ORs de ANDs ». À l’inverse, les expressions lambda pour `all` pour ces types de données doivent être en [forme normale conjonctive (FNC)](https://en.wikipedia.org/wiki/Conjunctive_normal_form), également appelée « ANDs de ORs ». Recherche cognitive Azure permet de telles comparaisons de plages car elle peut les exécuter efficacement à l’aide d’index inversés, tout comme elle peut effectuer une recherche de termes rapide pour les chaînes.

En résumé, voici les règles générales concernant ce qui est autorisé dans une expression lambda :

- Dans `any`, les *contrôles positifs* sont toujours autorisés, comme l’égalité, les comparaisons de plages, `geo.intersects` ou `geo.distance` comparé à `lt` ou `le` (considérez la « proximité » comme une égalité quand il s’agit d’un contrôle de distance).
- Dans `any`, `or` est toujours autorisé. Vous pouvez utiliser `and` uniquement pour les types de données qui peuvent exprimer des contrôles de plage, et uniquement si vous utilisez des ORs de ANDs (FND).
- Dans `all`, les règles sont inversées : seuls les *contrôles négatifs* sont autorisés. Vous pouvez toujours utiliser `and`, et vous pouvez utiliser `or` uniquement pour les contrôles de plage exprimés sous forme de ANDs de ORs (FNC).

Dans la pratique, voici les types de filtres que vous êtes le plus susceptible d’utiliser. Il est cependant toujours utile de comprendre les limites de ce qui est possible.

Pour obtenir des exemples spécifiques des types de filtres qui sont autorisés et de ceux qui ne le sont pas, consultez [Comment écrire des filtres de collection valides](search-query-troubleshoot-collection-filters.md#bkmk_examples).

## <a name="next-steps"></a>Étapes suivantes  

- [Résolution de problèmes liés aux filtres de collection OData dans Recherche cognitive Azure](search-query-troubleshoot-collection-filters.md)
- [Filtres dans la Recherche cognitive Azure](search-filters.md)
- [Vue d’ensemble du langage d’expression OData pour Recherche cognitive Azure](query-odata-filter-orderby-syntax.md)
- [Informations de référence sur la syntaxe d’expression OData pour Recherche cognitive Azure](search-query-odata-syntax-reference.md)
- [Rechercher des documents &#40;API REST de la recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents)