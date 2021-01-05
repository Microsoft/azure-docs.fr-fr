---
title: Ajouter une hiérarchie de catégorie de navigation à facettes
titleSuffix: Azure Cognitive Search
description: Ajoutez la navigation à facettes pour le filtrage autonome dans les applications de recherche qui s’intègrent à Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 05be5295ae5f8c73c916a21bba7dbc98ab0c5e87
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89002790"
---
# <a name="how-to-implement-faceted-navigation-in-azure-cognitive-search"></a>Implémentation de la navigation par facettes dans la Recherche cognitive Azure

La navigation à facettes est un mécanisme de filtrage qui fournit une navigation autonome d'extraction dans les applications de recherche. Le terme « navigation à facettes » peut vous sembler peu familier, mais vous l’avez très certainement déjà utilisé. Comme l'indique l'exemple ci-dessous, la navigation à facettes correspond tout simplement aux catégories utilisées pour filtrer les résultats.

 ![Démonstration du portail de recrutement de la Recherche cognitive Azure](media/search-faceted-navigation/azure-search-faceting-example.png "Démonstration du portail de recrutement de la Recherche cognitive Azure")

La navigation à facettes constitue un autre point d’entrée pour la recherche. Elle offre une alternative pratique à la saisie manuelle d’expressions de recherche complexes. Les facettes peuvent vous aider à trouver ce que vous recherchez, tout en vous assurant d’obtenir au moins un résultat. En tant que développeur, les facettes vous permettent d’exposer les critères de recherche les plus utiles pour naviguer dans votre index de recherche. Dans les applications de vente au détail en ligne, la navigation à facettes repose souvent sur les marques, les catégories (chaussures pour enfants), la taille, le prix, la popularité et les évaluations. 

L’implémentation de la navigation par facettes varie en fonction des technologies de recherche. Dans la Recherche cognitive Azure, la navigation par facettes est créée au moment de la requête, à l’aide de champs précédemment attribués dans votre schéma.

-   Dans les requêtes créées par votre application, une requête doit envoyer des *paramètres de requête de facette* afin de recevoir les valeurs de filtre de facette disponibles pour ce jeu de résultats du document.

-   Pour réduire réellement le jeu de résultats du document, l’application doit également appliquer une expression `$filter`.

Dans le développement de votre application, l’écriture du code qui construit les requêtes constitue la majeure partie du travail. La plupart des comportements de l’application que vous attendez de la navigation par facettes sont fournis par le service, y compris la prise en charge intégrée de la définition des plages et la récupération des décomptes pour les résultats de facettes. Le service comprend également des valeurs par défaut qui vous aident à éviter les structures de navigation difficiles à gérer. 

## <a name="sample-code-and-demo"></a>Exemple de code et démonstration
Cet article prend l’exemple d’un portail de recrutement. L’exemple est implémenté en tant qu’application ASP.NET MVC.

- Consultez et testez la [démonstration en ligne du portail de recrutement de la Recherche cognitive Azure](https://aka.ms/azjobsdemo).

- Téléchargez le code à partir du [référentiel Azure-Samples sur GitHub](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs).

## <a name="get-started"></a>Bien démarrer
Si vous êtes novice en recherche et développement, considérez que la navigation par facettes affiche les possibilités de recherche autonome. Il s'agit d'un type d'expérience de recherche détaillée, en fonction de filtres prédéfinis, utilisés pour limiter rapidement les résultats de la recherche à l'aide d'actions de type pointer et cliquer. 

### <a name="interaction-model"></a>Modèle d’interaction

L'expérience de recherche pour la navigation à facettes est itérative. Donc, essayons de la comprendre comme séquence de requêtes qui se déroulent en réponse aux actions de l'utilisateur.

Le point de départ est une page d'application qui offre une navigation à facettes, généralement placée sur la périphérie. La navigation à facettes est souvent présentée sous forme d'arborescence avec des cases à cocher pour chaque valeur ou du texte interactif. 

1. Une requête envoyée à la Recherche cognitive Azure spécifie la structure de la navigation par facettes par le biais d’un ou plusieurs paramètres de requête de facette. Par exemple, la requête peut inclure `facet=Rating`, éventuellement avec une option `:values` ou `:sort` pour affiner la présentation.
2. La couche de présentation renvoie une page de recherche qui fournit une navigation à facettes, à l'aide des facettes spécifiées dans la requête.
3. Face à une structure de navigation à facettes qui inclut le paramètre Évaluation, vous cliquez sur « 4 » pour indiquer que seuls les produits dotés d’une évaluation minimale de 4 doivent être affichés. 
4. En réponse, l'application envoie une requête qui inclut `$filter=Rating ge 4` 
5. La couche de présentation met à jour la page en affichant un jeu de résultats réduit, contenant uniquement les éléments qui répondent aux nouveaux critères (dans ce cas, les produits avec une évaluation de 4 et supérieure).

Une facette est un paramètre de requête, mais ne la confondez pas avec l'entrée de requête. Elle n'est jamais utilisée comme critère de sélection dans une requête. Considérez plutôt les paramètres de requête de facette comme des entrées de la structure de navigation qui est renvoyée dans la réponse. Pour chaque paramètre de requête de facette que vous fournissez, la Recherche cognitive Azure évalue le nombre de documents dans les résultats partiels pour chaque valeur de la facette.

Notez le `$filter` à l'étape 4. Ce filtre constitue un aspect important de la navigation à facettes. Bien que les facettes et les filtres soient indépendants dans l’API, vous avez besoin des deux pour fournir l’expérience attendue. 

### <a name="app-design-pattern"></a>Modèle de conception de l’application

Dans le code d’application, le modèle consiste à utiliser les paramètres de requête de facette pour renvoyer la structure de navigation à facettes, ainsi que les résultats de la facette, mais aussi une expression $filter.  L’expression de filtre gère l’événement clic sur la valeur de facette. Considérez l'expression `$filter` comme le code derrière la réduction réelle des résultats de la recherche renvoyés à la couche de présentation. Dans le cas d'une facette Couleurs, le fait de cliquer sur la couleur Rouge est implémenté par le biais d'une expression `$filter` qui sélectionne uniquement les éléments qui ont une couleur rouge. 

### <a name="query-basics"></a>Principes de base des requêtes

Dans la Recherche cognitive Azure, une requête est spécifiée par le biais d’un ou de plusieurs paramètres de requête (consultez [Rechercher des documents](/rest/api/searchservice/Search-Documents) pour obtenir une description de chacun d’eux). Aucun des paramètres de requête n'est requis, mais vous devez en avoir au moins un pour qu'une requête soit valide.

La précision, interprétée comme la possibilité de filtrer les résultats non pertinents, s’effectue par le biais d’une ou de ces deux expressions :

-   **search=**  
    La valeur de ce paramètre constitue l’expression de recherche. Il peut s'agir d'une portion de texte unique ou d'une expression de recherche complexe qui comprend plusieurs termes et opérateurs. Sur le serveur, une expression de recherche est utilisée pour la recherche en texte intégral. Elle interroge les champs pouvant faire l'objet d'une recherche dans l'index pour la correspondance des termes et renvoie les résultats classés. Si vous définissez `search` sur null, l'exécution de la requête est effectuée sur la totalité de l'index (c'est-à-dire, `search=*`). Dans ce cas, d’autres éléments de la requête, comme `$filter` ou un profil de score, sont les principaux facteurs qui influencent les documents renvoyés (`($filter`) ainsi que leur ordre (`scoringProfile` ou `$orderby`).

-   **$filter =**  
    Un filtre est un mécanisme puissant pour limiter la taille des résultats de la recherche basés sur les valeurs des attributs de document spécifiques. Un `$filter` est évalué en premier, suivi de la logique de facettes qui génère les valeurs disponibles et les décomptes correspondants pour chaque valeur

Les expressions de recherche complexes diminuent les performances de la requête. Si possible, utilisez des expressions de filtre bien construites pour accroître la précision et améliorer les performances de la requête.

Pour mieux comprendre comment un filtre ajoute plus de précision, comparez une expression de recherche complexe à une expression qui contient une expression de filtre :

-   `GET /indexes/hotel/docs?search=lodging budget +Seattle –motel +parking`
-   `GET /indexes/hotel/docs?search=lodging&$filter=City eq 'Seattle' and Parking and Type ne 'motel'`

Les deux requêtes sont valides, mais la seconde est préférable si vous cherchez des établissements autres que des « motels » avec stationnement à Seattle.
-   La première requête repose sur ces mots spécifiques étant mentionnés ou non mentionnés dans des champs de type chaîne comme Nom, Description et tout autre champ contenant des données pouvant faire l'objet d'une recherche.
-   La deuxième requête recherche des correspondances précises sur des données structurées et est susceptible d'être beaucoup plus précise.

Dans les applications qui incluent la navigation à facettes, veillez à ce que chaque action de l’utilisateur sur une structure de navigation à facettes soit accompagnée d’une réduction du champ des résultats de recherche. Pour affiner les résultats, utilisez une expression de filtre.

<a name="howtobuildit"></a>

## <a name="build-a-faceted-navigation-app"></a>Créer une application de navigation à facettes
Vous implémentez la navigation par facettes avec la Recherche cognitive Azure dans le code d’application qui vous permet de créer la requête de recherche. La navigation à facettes repose sur les éléments de schéma que vous avez définis précédemment.

L'attribut d'index `Facetable [true|false]` , prédéfini sur votre index de recherche, est défini sur des champs sélectionnés pour activer ou désactiver leur utilisation dans une structure de navigation à facettes. Sans `"Facetable" = true`, un champ ne peut pas être utilisé dans la navigation à facettes.

La couche de présentation dans votre code fournit l'expérience utilisateur. Elle doit répertorier les éléments constitutifs de la navigation à facettes, comme l'étiquette, les valeurs, les cases à cocher et le décompte. L’API REST de Recherche cognitive Azure est indépendante de la plateforme. Vous pouvez donc utiliser la langue et la plateforme que vous souhaitez. L'important est d'inclure des éléments d'interface utilisateur qui prennent en charge l'actualisation incrémentielle, avec l'état de l'interface utilisateur mis à jour lorsque chaque facette supplémentaire est sélectionnée. 

Au moment de la requête, votre code d'application crée une requête qui inclut `facet=[string]`, un paramètre de requête qui fournit le champ sur lequel baser la facette. Une requête peut avoir plusieurs facettes, comme `&facet=color&facet=category&facet=rating`, chaque facette étant séparée par un caractère d'esperluette (&).

Le code d'application doit également construire une expression `$filter` pour gérer les événements de clic dans la navigation à facettes. Un `$filter` réduit les résultats de la recherche, à l'aide de la valeur de facette comme critère de filtre.

La Recherche cognitive Azure retourne les résultats de la recherche en fonction du ou des mots que vous avez saisis ainsi que les mises à jour de la structure de la navigation par facettes. Dans la Recherche cognitive Azure, la navigation par facettes est une construction à niveau unique, avec des valeurs de facettes et des décomptes des résultats trouvés pour chaque.

Dans les sections suivantes, nous allons étudier comment générer chaque partie.

<a name="buildindex"></a>

## <a name="build-the-index"></a>Création de l'index
Les facettes sont activées sur une base de champ par champ dans l'index, au moyen de cet attribut d'index : `"Facetable": true`.  
Tous les types de champs pouvant être utilisés dans la navigation à facettes sont `Facetable` par défaut. Ces types de champs incluent `Edm.String`, `Edm.DateTimeOffset` et tous les types de champs numériques (globalement, tous les types de champs peuvent être utilisés comme facettes, sauf `Edm.GeographyPoint` qui ne peut pas être utilisé dans la navigation à facettes). 

Lorsque vous créez un index, nous vous recommandons, pour la navigation à facettes, de désactiver explicitement les facettes pour les champs qui ne doivent jamais être utilisés comme facettes.  En particulier, les champs de chaîne pour les valeurs singleton, comme un ID ou un nom de produit, doivent être définis sur `"Facetable": false` pour empêcher leur utilisation accidentelle (et inefficace) dans la navigation à facettes. Le fait de désactiver les facettes inutiles permet de conserver une taille d’index réduite, ce qui améliore généralement les performances.

Vous trouverez ci-dessous une partie du schéma de l’exemple d’application de démonstration du portail de recrutement, sans certains attributs pour en réduire la taille :

```json
{
  ...
  "name": "nycjobs",
  "fields": [
    { "name": "id",                 "type": "Edm.String",              "searchable": false, "filterable": false, ... "facetable": false, ... },
    { "name": "job_id",             "type": "Edm.String",              "searchable": false, "filterable": false, ... "facetable": false, ... },
    { "name": "agency",              "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "posting_type",        "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "num_of_positions",    "type": "Edm.Int32",              "searchable": false, "filterable": true, ...  "facetable": true, ...  },
    { "name": "business_title",      "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "civil_service_title", "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "title_code_no",       "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "level",               "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "salary_range_from",   "type": "Edm.Int32",              "searchable": false, "filterable": true, ...  "facetable": true, ...  },
    { "name": "salary_range_to",     "type": "Edm.Int32",              "searchable": false, "filterable": true, ...  "facetable": true, ...  },
    { "name": "salary_frequency",    "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { "name": "work_location",       "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
…
    { "name": "geo_location",        "type": "Edm.GeographyPoint",     "searchable": false, "filterable": true, ...  "facetable": false, ... },
    { "name": "tags",                "type": "Collection(Edm.String)", "searchable": true,  "filterable": true, ...  "facetable": true, ...  }
  ],
…
}
```

Conformément à l’exemple de schéma, l’option `Facetable` est désactivée pour les champs de chaîne qui ne doivent pas être utilisés comme facettes, par exemple les valeurs d’ID. Le fait de désactiver les facettes inutiles permet de conserver une taille d’index réduite, ce qui améliore généralement les performances.

> [!TIP]
> Nous vous recommandons d'inclure l'ensemble des attributs d'index pour chaque champ. Bien que l'option `Facetable` soit activée par défaut pour presque tous les champs, le fait de configurer volontairement chaque attribut peut vous aider à réfléchir aux implications de chaque décision au sein du schéma. 

<a name="checkdata"></a>

## <a name="check-the-data"></a>Vérifier les données
La qualité de vos données a un effet direct sur la matérialisation attendue de la structure de la navigation à facettes. Elle affecte également la facilité de création des filtres afin de limiter le jeu de résultats.

Si vous souhaitez trier selon la marque ou le prix, chaque document doit contenir des valeurs *BrandName* et *ProductPrice* valides, cohérentes et productives en tant qu’option de filtre.

Voici quelques rappels des points à appliquer :

* Pour chaque champ que vous souhaitez utiliser comme facette, demandez-vous s'il contient des valeurs qui conviennent en tant que filtres dans une recherche autonome. Les valeurs doivent être courtes, descriptives et suffisamment distinctives pour offrir un choix clair entre les différentes options.
* Fautes d'orthographe ou valeurs presque correspondantes. Si vous créez une facette Couleur et que les valeurs de champ incluent Orange et Ornage (faute d'orthographe), une facette basée sur le champ Couleur renverrait les deux options.
* Le texte à casse mixte peut également causer des dégâts dans la navigation à facettes, où orange et Orange s'afficheraient comme deux valeurs différentes. 
* Les versions au singulier et au pluriel de la même valeur peuvent entraîner une facette distincte pour chacune.

Comme vous pouvez l'imaginer, la rigueur en termes de préparation des données est un aspect essentiel d'une navigation à facettes efficace.

<a name="presentationlayer"></a>

## <a name="build-the-ui"></a>Créer l’interface utilisateur
Le fait de travailler à partir de la couche de présentation peut vous aider à découvrir des exigences que vous auriez pu manquer dans le cas contraire et de comprendre les capacités essentielles à l'expérience de recherche.

En termes de navigation à facettes, votre page Web ou d'application affiche la structure de navigation à facettes, détecte une entrée utilisateur sur la page et insère les éléments modifiés. 

Pour les applications Web, la méthode AJAX est en général utilisée dans la couche de présentation, car elle vous permet d'actualiser les modifications incrémentielles. Vous pouvez également utiliser ASP.NET MVC ou toute autre plateforme de visualisation qui peut se connecter à un service Recherche cognitive Azure par le biais de HTTP. L’exemple d’application référencé dans cet article (**Démonstration du portail de recrutement de la Recherche cognitive Azure**) est une application ASP.NET MVC.

Dans l’exemple fourni, la navigation à facettes est intégrée dans la page de résultats. L’exemple suivant, extrait du fichier `index.cshtml` de l’exemple d’application, montre la structure HTML statique correspondant à l’affichage de la navigation à facettes sur la page des résultats de la recherche. La liste de facettes est créée ou recréée dynamiquement lorsque vous envoyez un terme de recherche, ou lorsque vous activez ou désactivez une facette.

```html
<div class="widget sidebar-widget jobs-filter-widget">
  <h5 class="widget-title">Filter Results</h5>
    <p id="filterReset"></p>
    <div class="widget-content">

      <h6 id="businessTitleFacetTitle">Business Title</h6>
      <ul class="filter-list" id="business_title_facets">
      </ul>

      <h6>Location</h6>
      <ul class="filter-list" id="posting_type_facets">
      </ul>

      <h6>Posting Type</h6>
      <ul class="filter-list" id="posting_type_facets"></ul>

      <h6>Minimum Salary</h6>
      <ul class="filter-list" id="salary_range_facets">
      </ul>

  </div>
</div>
```

L’extrait de code suivant, tiré de la page `index.cshtml`, crée dynamiquement le code HTML permettant d’afficher la première facette, Fonction. Des fonctions similaires génèrent dynamiquement le code HTML pour les autres facettes. Chaque facette est associée à une étiquette et à un nombre indiquant combien d’éléments ont été trouvés pour ce résultat de facette.

```js
function UpdateBusinessTitleFacets(data) {
  var facetResultsHTML = '';
  for (var i = 0; i < data.length; i++) {
    facetResultsHTML += '<li><a href="javascript:void(0)" onclick="ChooseBusinessTitleFacet(\'' + data[i].Value + '\');">' + data[i].Value + ' (' + data[i].Count + ')</span></a></li>';
  }

  $("#business_title_facets").html(facetResultsHTML);
}
```

> [!TIP]
> Lorsque vous concevez la page de résultats, pensez à ajouter un mécanisme de suppression des facettes. Si vous ajoutez des cases à cocher, il est facile de savoir comment effacer les filtres. Pour les autres dispositions, vous devrez peut-être utiliser un modèle de navigation ou une autre approche créative. Ainsi, dans l’exemple d’application du portail de recrutement, vous pouvez cliquer sur `[X]` situé en regard d’une facette pour effacer cette dernière.

<a name="buildquery"></a>

## <a name="build-the-query"></a>Création de la requête
Le code que vous écrivez pour la création de requêtes doit spécifier toutes les parties d'une requête valide, y compris les expressions de recherche, les facettes, les filtres, les profils de score ; tout ce qui sert à formuler une requête. Dans cette section, nous allons explorer l’emplacement où les facettes s’intègrent dans une requête, ainsi que la façon dont les filtres sont utilisés avec des facettes pour fournir un jeu de résultats réduit.

Notez que les facettes font partie intégrante de cet exemple d'application. L’expérience de recherche dans la démonstration du portail de recrutement est conçue autour de filtres et de la navigation à facettes. La position de la navigation à facettes sur la page démontre son importance. 

Pour commencer, prenons un exemple. L’exemple suivant, extrait du fichier `JobsSearch.cs`, construit une requête qui crée une navigation à facettes basée sur la Fonction, le Lieu, le Type de publication et le Salaire minimal. 

```cs
SearchParameters sp = new SearchParameters()
{
  ...
  // Add facets
  Facets = new List<String>() { "business_title", "posting_type", "level", "salary_range_from,interval:50000" },
};
```

Un paramètre de requête à facettes est défini sur un champ et, selon le type de données, peut être davantage paramétré par une liste délimitée par des virgules qui inclut `count:<integer>`, `sort:<>`, `interval:<integer>` et `values:<list>`. Une liste de valeurs est prise en charge pour les données numériques lors de la définition de plages. Consultez [Rechercher des documents (API de Recherche cognitive Azure)](/rest/api/searchservice/Search-Documents) pour obtenir des détails sur l’utilisation.

En plus des facettes, la requête formulée par votre application doit également créer des filtres pour limiter le jeu de documents candidats basés sur une sélection de valeur de facette. Pour un magasin de vélos, la navigation à facettes fournit des indications aux questions du type *Quels sont les couleurs, fabricants et types de vélos disponibles ?* . Le filtrage permet de répondre à des questions du type *Quels sont les vélos de type VTT et de couleur rouge dans cette gamme de prix ?* . Lorsque vous cliquez sur « Rouge » pour indiquer que seuls les produits de couleur rouge doivent s’afficher, la requête suivante envoyée par l’application inclut `$filter=Color eq 'Red'`.

L’extrait de code suivant tiré de la page `JobsSearch.cs` ajoute la Fonction sélectionnée au filtre si vous sélectionnez une valeur de la facette Fonction.

```cs
if (businessTitleFacet != "")
  filter = "business_title eq '" + businessTitleFacet + "'";
```

<a name="tips"></a> 

## <a name="tips-and-best-practices"></a>Conseils et meilleures pratiques

### <a name="indexing-tips"></a>Conseils d’indexation
**Améliorer l’efficacité des index si vous n’utilisez pas de zone de recherche**

Si votre application utilise exclusivement la navigation à facettes (autrement dit, aucune zone de recherche), vous pouvez marquer le champ en tant que `searchable=false`, `facetable=true` pour produire un index plus compact. En outre, l'indexation se produit uniquement sur les valeurs de facettes entières, sans césure de mots ou indexation des composants d'une valeur à plusieurs mots.

**Spécifier les champs qui peuvent servir de facettes**

N'oubliez pas que le schéma de l'index détermine quels champs sont disponibles pour être utilisés comme facettes. En supposant qu'un champ puisse être utilisé comme facette, la requête spécifie quels champs utiliser comme facettes. Le champ que vous utilisez comme facette fournit les valeurs qui apparaissent sous l'étiquette. 

Les valeurs qui s'affichent sous chaque étiquette sont récupérées à partir de l'index. Par exemple, si le champ de facette est *Couleur*, les valeurs disponibles pour le filtrage supplémentaire sont les valeurs de ce champ : Rouge, Noir, etc.

Pour les valeurs de type Numérique et DateHeure uniquement, vous pouvez définir explicitement des valeurs sur le champ de facette (par exemple, `facet=Rating,values:1|2|3|4|5`). Une liste de valeurs est autorisée pour ces types de champs afin de simplifier la séparation des résultats de la facette en plages contiguës (plages basées sur des valeurs numériques ou des périodes de temps). 

**Par défaut, vous ne pouvez avoir qu’un seul niveau de navigation par facettes** 

Comme mentionné, il n'existe aucune prise en charge directe de l'imbrication des facettes dans une hiérarchie. Par défaut, la navigation par facettes dans la Recherche cognitive Azure ne prend en charge qu’un seul niveau de filtres. Toutefois, des solutions de contournement existent. Vous pouvez encoder une structure hiérarchique de facette dans une `Collection(Edm.String)` avec un point d'entrée par hiérarchie. L’implémentation de cette solution de contournement n’est pas abordée dans cet article. 

### <a name="querying-tips"></a>Conseils d’interrogation
**Validation des champs**

Si vous générez dynamiquement la liste de facettes en fonction d’une entrée utilisateur non fiable, vérifiez la validité des noms des champs à facettes. Vous pouvez également contourner les noms lors de la génération d’URL à l’aide de `Uri.EscapeDataString()` dans .NET, ou son équivalent dans la plateforme de votre choix.

### <a name="filtering-tips"></a>Conseils de filtrage
**Augmenter la précision de la recherche avec des filtres**

Utilisation de filtres. Si vous utilisez uniquement les expressions de recherche, la recherche de radical peut entraîner le renvoi d'un document qui ne contient pas la valeur de facette précise dans aucun de ses champs.

**Augmenter les performances de recherche avec des filtres**

Les filtres réduisent le jeu de documents candidats pour la recherche et les excluent du classement. Pour les grands jeux de documents, l’utilisation d’une exploration à facettes sélective offre souvent de meilleures performances.
  
**Filtrer uniquement les champs à facettes**

Dans l’exploration à facettes, vous voulez en général inclure uniquement les documents contenant la valeur de facette dans un champ spécifique (facette), et pas n’importe où dans tous les champs de recherche. L'ajout d'un filtre renforce le champ cible en indiquant au service de rechercher uniquement dans le champ à facette pour trouver une valeur correspondante.

**Tronquer les résultats de facettes avec d’autres filtres**

Les résultats de la facette sont des documents trouvés dans les résultats de la recherche qui correspondent à un terme de la facette. Dans l’exemple suivant, dans les résultats de la recherche pour *cloud computing*, 254 éléments ont également la *spécification interne* comme type de contenu. Les éléments ne sont pas nécessairement mutuellement exclusifs. Si un élément répond aux critères des deux filtres, il est compté dans chacun d'eux. La duplication est possible lors de l’utilisation des facettes sur les champs `Collection(Edm.String)`, souvent utilisés pour implémenter le balisage de document.

```output
Search term: "cloud computing"
Content type
   Internal specification (254)
   Video (10)
```

En général, si vous trouvez que les résultats de la facette sont toujours trop volumineux, nous vous recommandons d’ajouter des filtres afin d’apporter aux utilisateurs de votre application plus d’options pour affiner la recherche.

### <a name="tips-about-result-count"></a>Conseils sur le décompte des résultats

**Limiter le nombre d’éléments dans la navigation par facettes**

Pour chaque champ à facettes dans l'arborescence de navigation, il existe une limite par défaut de 10 valeurs. Cette valeur par défaut est judicieuse pour les structures de navigation, car elle permet de conserver une taille gérable pour la liste des valeurs. Vous pouvez remplacer la valeur par défaut en affectant une valeur à compter.

* `&facet=city,count:5` spécifie que seules les cinq premières villes trouvées dans les résultats en tête du classement sont renvoyées en tant que résultat de la facette. Imaginez un exemple de requête affichant le terme de recherche « aéroport » et 32 correspondances. Si la requête spécifie `&facet=city,count:5`, seules les cinq premières villes uniques avec le plus de documents dans les résultats de la recherche sont incluses dans les résultats de la facette.

Notez la différence entre les résultats de la recherche et les résultats de la facette. Les résultats de la recherche sont tous les documents qui correspondent à la requête. Les résultats de la facette sont les correspondances pour chaque valeur de facette. Dans l’exemple, les résultats de la recherche incluent des noms de villes qui ne se trouvent pas dans la liste de classification de la facette (5, dans notre exemple). Les résultats filtrés par le biais de la navigation à facettes deviennent visibles lorsque vous effacez les facettes ou choisissez d’autres facettes en plus de Ville. 

> [!NOTE]
> Traiter de `count` lorsqu'il existe plus d'un type peut prêter à confusion. Le tableau suivant offre un bref résumé de l’utilisation du terme dans l’API de Recherche cognitive Azure, un exemple de code et la documentation. 

* `@colorFacet.count`<br/>
  Dans le code de présentation, un paramètre de décompte doit s’afficher sur la facette. Il est utilisé pour afficher le nombre de résultats de la facette. Dans les résultats de la facette, le décompte indique le nombre de documents qui correspondent au terme ou à la plage de la facette.
* `&facet=City,count:12`<br/>
  Dans une requête de facette, vous pouvez définir le décompte sur une valeur.  La valeur par défaut est 10, mais vous pouvez définir une valeur supérieure ou inférieure. Le paramètre `count:12` renvoie les 12 premières correspondances dans les résultats de la facette selon le décompte de documents.
* "`@odata.count`"<br/>
  Dans la réponse de la requête, cette valeur indique le nombre d'éléments correspondants dans les résultats de la recherche. En moyenne, il est supérieur à la somme de tous les résultats de la facette combinés, en raison de la présence d'éléments qui correspondent au terme de la recherche, mais sans correspondance avec la valeur de la facette.

**Obtenir les décomptes dans les résultats de facettes**

Lorsque vous ajoutez un filtre à une requête à facettes, il se peut que vous souhaitiez conserver l’instruction de facette (par exemple, `facet=Rating&$filter=Rating ge 4`). Techniquement, le paramètre facette=Évaluation n'est pas nécessaire, mais le fait de le conserver renvoie les décomptes des valeurs de facettes pour les évaluations de 4 et supérieures. Par exemple, si vous cliquez sur « 4 » et si la requête inclut un filtre pour une valeur supérieure ou égale à « 4 », des décomptes sont renvoyés pour chaque évaluation égale ou supérieure à 4.  

**Vérifier que les nombres de facettes sont exacts**

Dans certaines circonstances, il est possible que les décomptes de facettes ne correspondent pas aux jeux de résultats (consultez [Navigation par facettes dans la Recherche cognitive Azure (page de questions Microsoft Q&A)](/answers/topics/azure-cognitive-search.html)).

Les décomptes de facettes peuvent être erronés en raison de l'architecture de partitionnement. Chaque index de recherche a plusieurs partitions et chacune d’elles indique les N premières facettes par décompte de document, qui est ensuite combiné en un résultat unique. Si certaines partitions ont beaucoup de valeurs correspondantes, tandis que d’autres en ont moins, il est possible que certaines valeurs de facettes soient manquantes ou sous-comptabilisées dans les résultats.

Ce comportement peut changer à tout moment mais si vous rencontrez ce problème aujourd’hui, vous pouvez le contourner en gonflant artificiellement le décompte :\<number> sur un nombre élevé pour appliquer la déclaration complète à partir de chaque partition. Si la valeur de décompte : est supérieure ou égale au nombre de valeurs uniques dans le champ, vous êtes sûr d'obtenir des résultats précis. Toutefois, lorsque les décomptes de documents sont élevés, les performances baissent, alors utilisez cette option judicieusement.

### <a name="user-interface-tips"></a>Conseils sur l’interface utilisateur
**Ajouter des étiquettes pour chaque champ dans la navigation à facettes**

Les étiquettes sont généralement définies dans le code HTML ou le formulaire (`index.cshtml` dans l’exemple d’application). Il n’existe aucune API dans la Recherche cognitive Azure pour les étiquettes de navigation par facettes ou tout autre type de métadonnées.

<a name="rangefacets"></a>

## <a name="filter-based-on-a-range"></a>Filtrer sur une plage de valeurs
L’utilisation de facettes sur des plages de valeurs est une condition d’application de recherche courante. Les plages sont prises en charge pour les données numériques et les valeurs DateHeure. Vous pouvez en savoir plus sur chaque approche en consultant [Rechercher des documents (API de Recherche cognitive Azure)](/rest/api/searchservice/Search-Documents).

La Recherche cognitive Azure simplifie la création de plage en fournissant deux approches pour calculer une plage. Pour les deux approches, la Recherche cognitive Azure crée les plages appropriées avec les entrées que vous avez fournies. Par exemple, si vous spécifiez des valeurs de plage de 10|20|30, Recherche Azure crée automatiquement les plages 0-10, 10-20, 20-30. Votre application peut éventuellement supprimer les intervalles vides. 

**Approche 1 : Utiliser le paramètre d’intervalle**  
Pour définir les facettes de prix par incréments de 10 $, vous devez spécifier :`&facet=price,interval:10`

**Approche 2 : Utiliser une liste de valeurs**  
Pour les données numériques, vous pouvez utiliser une liste de valeurs.  Prenez en compte la plage de facette pour un champ `listPrice`, indiquée comme suit :

  ![Exemple de liste de valeurs](media/search-faceted-navigation/Facet-5-Prices.PNG "Exemple de liste de valeurs")

Pour spécifier une plage de facette comme celle de la capture d’écran précédente, utilisez une liste de valeurs :

> `facet=listPrice,values:10|25|100|500|1000|2500`

Chaque plage est créée avec 0 comme point de départ, une valeur de la liste comme point de terminaison, puis la plage précédente en moins pour créer des intervalles discrets. La Recherche cognitive Azure effectue cette opération dans le cadre de la navigation par facettes. Vous n'avez pas à écrire du code pour structurer chaque intervalle.

### <a name="build-a-filter-for-a-range"></a>Créer un filtre pour une plage
Pour filtrer les documents en fonction d’une plage que vous avez sélectionnée, vous pouvez utiliser les opérateurs de filtre `"ge"` et `"lt"` dans une expression en deux parties qui définit les points de terminaison de la plage. Par exemple, si vous choisissez la plage 10-25 pour un champ `listPrice`, le filtre sera `$filter=listPrice ge 10 and listPrice lt 25`. Dans l’exemple de code, l’expression de filtre utilise les paramètres **priceFrom** et **priceTo** pour définir les points de terminaison. 

  ![Requête pour une plage de valeurs](media/search-faceted-navigation/Facet-6-buildfilter.PNG "Requête pour une plage de valeurs")

<a name="geofacets"></a> 

## <a name="filter-based-on-distance"></a>Filtrer en fonction de la distance
Il est courant de voir des filtres qui vous aident à choisir un magasin, un restaurant ou une destination en fonction de sa proximité à votre emplacement actuel. Ce type de filtre peut ressembler à la navigation à facettes, mais c’est tout simplement un filtre. Nous le mentionnons ici pour ceux d'entre vous qui recherchent spécifiquement des conseils d'implémentation pour ce problème de conception particulier.

Il existe deux fonctions géospatiales dans la Recherche cognitive Azure, **geo.distance** et **geo.intersects**.

* La fonction **geo.distance** renvoie la distance en kilomètres entre deux points. L’un est un champ et l’autre est une constante considérée comme une partie du filtre. 
* La fonction **geo.intersects** renvoie true si un point donné se trouve dans un polygone donné. Le point est un champ et le polygone est spécifié sous forme de liste constante de coordonnées en tant que partie du filtre.

Vous trouverez des exemples de filtres dans [Syntaxe d’expression OData (Recherche cognitive Azure)](query-odata-filter-orderby-syntax.md).

<a name="tryitout"></a>

## <a name="try-the-demo"></a>Essayer la démonstration
La démonstration du portail de recrutement de la Recherche cognitive Azure contient les exemples référencés dans cet article.

-   Consultez et testez la [démonstration en ligne du portail de recrutement de la Recherche cognitive Azure](https://aka.ms/azjobsdemo).

-   Téléchargez le code à partir du [référentiel Azure-Samples sur GitHub](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs).

Lorsque vous utilisez les résultats de la recherche, observez les modifications de construction de la requête dans l'URL. Cette application ajoute des facettes à l'URI lors de la sélection de chaque.

1. Pour utiliser la fonctionnalité de mappage de l’application de démonstration, obtenez une clé Bing Maps auprès du [Centre de développement Bing Maps](https://www.bingmapsportal.com/). Collez-la sur la clé existante dans la page `index.cshtml`. Le paramètre `BingApiKey` du fichier `Web.config` n’est pas utilisé. 

2. Exécutez l'application. Suivez la visite guidée facultative ou fermez la boîte de dialogue.
   
3. Saisissez un terme de recherche, comme « analyste », puis cliquez sur l’icône de recherche. La requête s'exécute rapidement.
   
   Une structure de navigation à facettes est également renvoyée avec les résultats de la recherche. Dans la page de résultats de la recherche, la structure de la navigation à facettes inclut des décomptes pour chaque résultat de facette. Aucune facette n’est sélectionnée : tous les résultats correspondants sont donc renvoyés.
   
   ![Résultats de recherche avant la sélection des facettes](media/search-faceted-navigation/faceted-search-before-facets.png "Résultats de recherche avant la sélection des facettes")

4. Cliquez sur une Fonction, un Lieu ou un Salaire minimal. Les facettes sont égales à « null » pour la recherche initiale, mais lorsqu’elles prennent des valeurs, les éléments qui ne correspondent plus sont supprimés des résultats de la recherche.
   
   ![Résultats de recherche après la sélection des facettes](media/search-faceted-navigation/faceted-search-after-facets.png "Résultats de recherche après la sélection des facettes")

5. Pour effacer la requête à facettes afin de tester des comportements de requête différents, cliquez sur `[X]` en regard des facettes sélectionnées dans le but d’effacer ces dernières.
   
<a name="nextstep"></a>

## <a name="learn-more"></a>En savoir plus
Regardez la [vidéo de présentation approfondie de la Recherche cognitive Azure](https://channel9.msdn.com/Events/TechEd/Europe/2014/DBI-B410). À 45:25 sur la vidéo, vous trouverez une démonstration sur la façon d'implémenter des facettes.

Pour plus d'informations sur les principes de conception pour la navigation à facettes, nous vous recommandons les liens suivants :

* [Modèles de conception : Navigation à facettes](https://alistapart.com/article/design-patterns-faceted-navigation)
* [Problèmes de serveur frontal lors de l’implémentation de la recherche à facettes - Partie 1](https://articles.uie.com/faceted_search2/)