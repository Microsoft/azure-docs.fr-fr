---
title: Filtres de facette pour la navigation de recherche dans les applications
titleSuffix: Azure Cognitive Search
description: Effectuez un filtrage selon des critères par identité de sécurité d’utilisateur, emplacement géographique ou valeurs numériques pour réduire les résultats de recherche des requêtes effectuées dans la Recherche cognitive Azure, un service de recherche dans le cloud hébergé sur Microsoft Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/05/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 9360fc000e01e1c52561cbaa3e2f2968e67e2fa2
ms.sourcegitcommit: a07a01afc9bffa0582519b57aa4967d27adcf91a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91740868"
---
# <a name="how-to-build-a-facet-filter-in-azure-cognitive-search"></a>Comment créer un filtre de facette dans la Recherche cognitive Azure 

La navigation par facettes est utilisée pour le filtrage autonome sur les résultats d’une requête dans une application de recherche, dans le cadre de laquelle votre application propose des contrôles d’interface utilisateur permettant de limiter une recherche à des groupes de documents (par exemple, des catégories ou des marques) ; la Recherche cognitive Azure fournit la structure de données sur laquelle reposera l’opération. Dans cet article, nous passerons rapidement en revue les étapes de base de la procédure de création d’une structure de navigation par facettes, sur laquelle sera basée l’expérience de recherche que vous souhaitez proposer. 

> [!div class="checklist"]
> * Choisissez des champs pour le filtrage et la création de facettes
> * Définissez des attributs sur le champ
> * Créez l’index et chargez les données
> * Ajoutez des filtres de facettes à une requête
> * Gérez les résultats

Les facettes sont dynamiques et renvoyées sur une requête. Les réponses associées à une recherche incluent les catégories de facettes utilisées pour parcourir les résultats. Si vous n’êtes pas familiarisé avec les facettes, l’exemple suivant illustre une structure de navigation par facettes.

:::image type="complex" source="media/search-filters-facets/facet-nav.png" alt-text="Résultats de recherche filtrés":::
« Image représentant une boîte de dialogue de recherche avec les résultats de recherche filtrés et regroupés par intitulé de poste. Une flèche indique que les résultats sont des facettes qui s’affichent dans une structure de navigation à facettes ».:::image-end:::

Vous découvrez ce type de navigation et souhaitez en savoir plus ? Consultez [Implémentation de la navigation par facettes dans la Recherche cognitive Azure](search-faceted-navigation.md).

## <a name="choose-fields"></a>Choisir des champs

Vous pouvez calculer des facettes sur la base de champs à une seule valeur, ou de collections. Les champs les plus efficaces dans une navigation par facettes présentent une cardinalité faible (un petit nombre de valeurs distinctes, qui se répètent tout au long d’un document dans votre corpus de recherche, par exemple une liste de couleurs, de pays/régions ou de noms de marques). 

La création de facettes est activée champ par champ lorsque vous générez l’index, si vous définissez l’attribut `facetable` sur `true`. Vous devez généralement définir également l’attribut `filterable` sur `true` pour ces champs, afin que votre application de recherche puisse filtrer sur ces champs en fonction des facettes sélectionnées par l’utilisateur final. 

Lors de la création d’un index à l’aide de l’API REST, tout [type de champ](/rest/api/searchservice/supported-data-types) pouvant éventuellement être utilisé dans la navigation à facettes est marqué comme `facetable` par défaut :

+ `Edm.String`
+ `Edm.DateTimeOffset`
+ `Edm.Boolean`
+ Types de champs numériques : `Edm.Int32`, `Edm.Int64`, `Edm.Double`
+ Collections des types ci-dessus (par exemple, `Collection(Edm.String)` ou `Collection(Edm.Double)`)

Vous ne pouvez pas utiliser les champs `Edm.GeographyPoint` ou `Collection(Edm.GeographyPoint)` dans la navigation par facettes. Les facettes fonctionnent mieux sur les champs présentant une faible cardinalité. En raison de la résolution des coordonnées géographiques, il est rare que les deux jeux de coordonnées soient égaux dans un jeu de données. Par conséquent, elles ne sont pas prises en charge pour les coordonnées géographiques. Vous aurez besoin d’un champ de ville ou de région pour les facettes créées par lieu.

## <a name="set-attributes"></a>Définir des attributs

Les attributs d’index qui contrôlent l’utilisation d’un champ sont ajoutés aux définitions de champs individuels dans l’index. Dans l’exemple suivant, les champs présentant une faible cardinalité, utiles pour les facettes, incluent une `category` (hôtel, motel, auberge de jeunesse...), des `tags` et des `rating`. Les attributs `filterable` et `facetable` de ces champs sont définis explicitement dans l’exemple suivant à titre d’illustration. 

> [!Tip]
> Pour optimiser le stockage et les performances, une meilleure pratique consiste à désactiver la création de facettes pour les champs qui ne doivent pas être utilisés en tant que facettes. En particulier, les champs de chaîne pour les valeurs uniques, comme un ID ou un nom de produit, doivent être définis sur `"facetable": false` pour empêcher leur utilisation accidentelle (et inefficace) dans la navigation à facettes.


```json
{
  "name": "hotels",  
  "fields": [
    { "name": "hotelId", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false },
    { "name": "baseRate", "type": "Edm.Double" },
    { "name": "description", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false },
    { "name": "description_fr", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene" },
    { "name": "hotelName", "type": "Edm.String", "facetable": false },
    { "name": "category", "type": "Edm.String", "filterable": true, "facetable": true },
    { "name": "tags", "type": "Collection(Edm.String)", "filterable": true, "facetable": true },
    { "name": "parkingIncluded", "type": "Edm.Boolean",  "filterable": true, "facetable": true, "sortable": false },
    { "name": "smokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true, "sortable": false },
    { "name": "lastRenovationDate", "type": "Edm.DateTimeOffset" },
    { "name": "rating", "type": "Edm.Int32", "filterable": true, "facetable": true },
    { "name": "location", "type": "Edm.GeographyPoint" }
  ]
}
```

> [!Note]
> Cette définition d’index est copiée à partir de la section relative à la [création d’un index de Recherche cognitive Azure à l’aide de l’API REST](./search-get-started-powershell.md). Il est identique, à l’exception de légères différences dans les définitions de champ. Les attributs `filterable` et `facetable` sont ajoutés de manière explicite sur les champs `category`, `tags`, `parkingIncluded`, `smokingAllowed`, et `rating`. Dans la pratique, `filterable` et `facetable` seraient activés par défaut sur ces champs lorsque vous utilisez l’API REST. Lorsque vous utilisez le Kit de développement logiciel (SDK) .NET, ces attributs doivent être activés explicitement.

## <a name="build-and-load-an-index"></a>Créer et charger un index

La [création et le remplissage de l’index](./search-get-started-dotnet.md#1---create-an-index) sont une étape intermédiaire (et peut-être évidente) avant la formulation d’une requête. Nous signalons cette étape ici par souci d’exhaustivité. Une manière de déterminer si l’index est disponible consiste à vérifier la liste des index dans le [portail](https://portal.azure.com).

## <a name="add-facet-filters-to-a-query"></a>Ajoutez des filtres de facettes à une requête

Dans le code d’application, construisez une requête spécifiant toutes les parties d’une requête valide, y compris les expressions de recherche, les facettes, les filtres, les profils de score, à savoir tout ce qui sert à formuler une requête. L’exemple suivant génère une requête qui crée une navigation par facettes selon le type d’hébergement, d’évaluation et d’autres équipements.

```csharp
var sp = new SearchParameters()
{
    ...
    // Add facets
    Facets = new[] { "category", "rating", "parkingIncluded", "smokingAllowed" }.ToList()
};
```

### <a name="return-filtered-results-on-click-events"></a>Renvoyer des résultats filtrés sur les événements clic

Lorsque l’utilisateur final clique sur une valeur de facette, le gestionnaire de l’événement click doit utiliser une expression de filtre pour comprendre l’intention de l’utilisateur. Si vous disposez d’une facette`category`, un clic sur la catégorie `$filter` est implémenté avec une expression qui sélectionne les hébergements de ce type. Lorsqu’un utilisateur clique sur « motels » pour indiquer que seul ce type d’hébergement doit être affiché, la requête suivante que l’application envoie inclut la chaîne `$filter=category eq 'motel'`.

L’extrait de code suivant ajoute la catégorie au filtre si l’utilisateur sélectionne une valeur à partir de la facette « Category ».

```csharp
if (!String.IsNullOrEmpty(categoryFacet))
    filter = $"category eq '{categoryFacet}'";
```

Si l’utilisateur clique sur une valeur de facette pour un champ de la collection comme `tags`, par exemple le « pool » de valeurs, l’application doit utiliser la syntaxe de filtre suivante : `$filter=tags/any(t: t eq 'pool')`

## <a name="tips-and-workarounds"></a>Conseils et solutions de contournement

### <a name="initialize-a-page-with-facets-in-place"></a>Initialiser une page sur laquelle des facettes sont définies

Si vous souhaitez initialiser une page sur laquelle des facettes sont définies, vous pouvez envoyer une requête dans le cadre de l’initialisation de la page, afin d’amorcer la page avec une structure de facette initiale.

### <a name="preserve-a-facet-navigation-structure-asynchronously-of-filtered-results"></a>Conserver une structure de navigation par facettes de manière asynchrone par rapport aux résultats filtrés

L’une des difficultés liées à la navigation par facettes dans la Recherche cognitive Azure est le fait que les facettes existent uniquement pour les résultats actuels. Dans la pratique, il est courant de conserver un ensemble statique de facettes, afin que l’utilisateur puisse naviguer dans l’ordre inverse, en suivant les étapes déjà parcourues pour explorer les autres chemins d’accès via le contenu de la recherche. 

Il s’agit d’un cas d’usage courant, mais non fourni de manière prête à l’emploi par la structure de navigation par facettes. Les développeurs qui veulent utiliser des facettes statiques contournent cette limitation en émettant deux requêtes filtrées, l’une dont la portée se limite aux résultats et l’autre, qui permet de créer une liste statique de facettes pour la navigation.

## <a name="see-also"></a>Voir aussi

+ [Filtres dans la Recherche cognitive Azure](search-filters.md)
+ [Création d’une API REST d’index](/rest/api/searchservice/create-index)
+ [API REST de recherche de documents](/rest/api/searchservice/search-documents)