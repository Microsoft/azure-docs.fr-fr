---
title: Utiliser la syntaxe de requête Lucene simple
titleSuffix: Azure Cognitive Search
description: Apprenez, à l’aide d’exemples, à exécuter des requêtes basées sur la syntaxe simple pour effectuer des recherches en texte intégral, filtrées, géographiques ou à facettes sur un index de Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/12/2020
ms.openlocfilehash: ff9495e37a499b5502d8f8ced79b69608fa9552a
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401744"
---
# <a name="use-the-simple-search-syntax-in-azure-cognitive-search"></a>Utiliser la syntaxe de recherche « simple » dans Recherche cognitive Azure

Dans Recherche cognitive Azure, la [syntaxe de requête simple](query-simple-syntax.md) appelle l’analyseur de requêtes par défaut pour une recherche en texte intégral. Cet analyseur rapide gère des scénarios courants, notamment la recherche en texte intégral, la recherche filtrée et à facettes, ainsi que la recherche de préfixe. Cet article utilise des exemples pour illustrer l’utilisation de la syntaxe simple dans une demande [Rechercher des documents (API REST)](/rest/api/searchservice/search-documents).

> [!NOTE]
> Une autre syntaxe de requête est la syntaxe [Lucene complète](query-lucene-syntax.md), qui prend en charge des structures de requête plus complexes, telles qu’une recherche approximative et par caractères génériques. Pour plus d’informations et d’exemples, consultez [Utiliser la syntaxe Lucene complète](search-query-lucene-examples.md).

## <a name="nyc-jobs-examples"></a>Exemples d’emplois à New York

Les exemples suivants tirent parti de l’[index de recherche NYC Jobs](https://azjobsdemo.azurewebsites.net/) composé de postes à pourvoir sur la base d’un jeu de données fourni par l’[initiative City of New York OpenData](https://nycopendata.socrata.com/). Ces données ne doivent pas être considérées comme étant à jour ou complètes. L’index se trouve sur un service de bac à sable fourni par Microsoft, ce qui signifie que vous n’avez pas besoin d’abonnement Azure ni de Recherche cognitive Azure pour essayer ces requêtes.

En revanche, vous avez besoin de Postman ou d’un outil équivalent pour émettre la requête HTTP sur GET ou POST. Si vous ne connaissez pas bien ces outils, consultez [Démarrage rapide : Explorer l’API REST de Recherche cognitive Azure](search-get-started-rest.md).

## <a name="set-up-the-request"></a>Configurer la requête

1. Les en-têtes de requête doivent comporter les valeurs suivantes :

   | Clé | Valeur |
   |-----|-------|
   | Content-Type | `application/json`|
   | api-key  | `252044BE3886FE4A8E3BAA4F595114BB` </br> (il s’agit de la clé API de requête réelle pour le service de recherche de bac à sable qui héberge l’index NYC Jobs) |

1. Définissez le verbe sur **`GET`** .

1. Définissez l’URL sur **`https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&search=*&$count=true`** . 

   + La collection de documents dans l’index comprend tout le contenu disponible pour la recherche. La clé API de requête fournie dans l’en-tête de requête fonctionne uniquement pour les opérations de lecture ciblant la collection de documents.

   + **`$count=true`** retourne le nombre des documents correspondant aux critères de recherche. Une chaîne de recherche vide comptabilisera tous les documents figurant dans l’index (environ 2558 dans le cas de NYC Jobs).

   + **`search=*`** est une requête non spécifiée équivalente à une recherche nulle ou vide. Elle n’est pas particulièrement utile, mais c’est la recherche la plus simple que vous puissiez effectuer, et elle affiche tous les champs récupérables dans l’index, avec toutes les valeurs.

1. En guise d’étape de vérification, collez la requête suivante dans GET et cliquez sur **Envoyer**. Les résultats sont retournés sous forme de documents JSON détaillés.

   ```http
   https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search=*&queryType=full
   ```

### <a name="how-to-invoke-simple-query-parsing"></a>Appel de l’analyse de requête simple

Pour les requêtes interactives, vous n’avez rien à spécifier : l’analyse simple est l’analyse par défaut. Dans le code, si vous avez précédemment appelé **`queryType=full`** , vous pouvez réinitialiser la valeur par défaut avec **`queryType=simple`** .

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
{
    "queryType": "simple"
}
```

## <a name="example-1-full-text-search-on-specific-fields"></a>Exemple 1 : Recherche en texte intégral sur des champs spécifiques

Ce premier exemple n’est pas propre à un analyseur, mais nous permet d’introduire le premier concept de requête fondamental : la contenance. Cet exemple limite l’exécution de la requête et la réponse à quelques champs spécifiques. Lors de l’utilisation de l’outil Postman ou Explorateur de recherche, il est important de connaître la structure d’une réponse JSON accessible en lecture. 

Cette requête cible uniquement *business_title* dans **`searchFields`** , en spécifiant grâce au paramètre **`select`** le même champ dans la réponse.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
{
    "count": true,
    "queryType": "simple",
    "search": "*",
    "searchFields": "business_title",
    "select": "business_title"
}
```

La réponse pour cette requête doit ressembler à la capture d’écran suivante.

  :::image type="content" source="media/search-query-lucene-examples/postman-sample-results.png" alt-text="Exemple de réponse Postman" border="false":::

Vous avez peut-être remarqué le score de recherche dans la réponse. Des scores uniformes de **1** sont obtenus en l’absence de classement, soit parce que la recherche n’était pas une recherche en texte intégral, soit parce qu’aucun critère n’a été fourni. Pour une recherche vide, les lignes reviennent dans un ordre arbitraire. Si vous incluez des critères réels, vous constaterez que les scores de recherche deviendront des valeurs significatives.

## <a name="example-2-look-up-by-id"></a>Exemple 2 : Recherche par ID

Lorsque vous retournez des résultats de recherche dans une requête, une étape logique suivante consiste à fournir une page de détails qui comprend davantage de champs du document. Cet exemple montre comment retourner un document unique en utilisant une [opération de recherche](/rest/api/searchservice/lookup-document) pour transmettre l’ID du document.

Tous les documents ont un identificateur unique. Pour tester la syntaxe d’une requête de recherche, retournez tout d’abord une liste d’ID de document afin d’en trouver un à utiliser. Pour NYC Jobs, les identificateurs sont stockés dans le champ `id`.

```http
GET /indexes/nycjobs/docs?api-version=2020-06-30&search=*&$select=id&$count=true
```

Ensuite, récupérez un document de la collection basé sur `id` « 9E1E3AF9-0660-4E00-AF51-9B654925A2D5 », qui est apparu en premier dans la réponse précédente. La requête suivante retourne tous les champs récupérables pour le document entier.

```http
GET /indexes/nycjobs/docs/9E1E3AF9-0660-4E00-AF51-9B654925A2D5?api-version=2020-06-30
```

## <a name="example-3-filter-queries"></a>Exemple 3 : Requêtes de filtre

La [syntaxe de filtre](./search-query-odata-filter.md) est une expression OData que vous pouvez utiliser de façon autonome ou avec **`search`** . Un filtre autonome, dépourvu de paramètre de recherche, est utile quand l’expression de filtre est en mesure de qualifier complètement les documents d’intérêt. En l’absence d’une chaîne de requête, il n’y a ni analyse lexicale ou linguistique, ni scoring (tous les scores sont égaux à 1), ni classement. Vous pouvez remarquer que la chaîne de recherche est vide.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "",
      "filter": "salary_frequency eq 'Annual' and salary_range_from gt 90000",
      "select": "job_id, business_title, agency, salary_range_from"
    }
```

Utilisés conjointement, le filtre est d’abord appliqué à la totalité de l’index et la recherche est effectuée sur les résultats du filtre. Les filtres peuvent donc être utiles pour améliorer les performances des requêtes, puisqu’ils limitent le nombre de documents que devra traiter la requête de recherche.

  :::image type="content" source="media/search-query-simple-examples/filtered-query.png" alt-text="Réponse d’une requête de filtre" border="false":::

Une autre méthode efficace pour combiner un filtre et une recherche consiste à utiliser **`search.ismatch*()`** dans une expression de filtre, dans laquelle vous pouvez utiliser une requête de recherche dans le filtre. Cette expression de filtre utilise un caractère générique sur *plan* afin de sélectionner les documents dont le champ business_title contient les termes plan, planificateur, planification, et ainsi de suite.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "",
      "filter": "search.ismatch('plan*', 'business_title', 'full', 'any')",
      "select": "job_id, business_title, agency, salary_range_from"
    }
```

Pour plus d’informations sur la fonction, consultez la [description de la fonction search.ismatch dans les exemples de filtre](./search-query-odata-full-text-search-functions.md#examples).

## <a name="example-4-range-filters"></a>Exemple 4 : Filtres de plage

Le filtrage de plage est pris en charge par le biais des expressions **`$filter`** pour n’importe quel type de données. Les exemples ci-après effectuent des recherches sur des champs numériques et de chaîne. 

Les types de données sont importants dans les filtres de plage et fonctionnent mieux lorsque les données numériques se trouvent dans des champs numériques, et les données de chaîne dans des champs de chaîne. L’utilisation de données numériques dans les champs de chaîne n’est pas adaptée aux plages, car les chaînes numériques ne sont pas comparables dans la Recherche cognitive Azure.

La requête suivante est une plage numérique :

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "",
      "filter": "num_of_positions ge 5 and num_of_positions lt 10",
      "select": "job_id, business_title, num_of_positions, agency",
      "orderby": "agency"
    }
```
La réponse pour cette requête doit ressembler à la capture d’écran suivante.

  :::image type="content" source="media/search-query-simple-examples/rangefilternumeric.png" alt-text="Filtre de plage pour les plages numériques" border="false":::

Dans cette requête, la plage se trouve sur un champ de chaîne (business_title) :

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "",
      "filter": "business_title ge 'A*' and business_title lt 'C*'",
      "select": "job_id, business_title, agency",
      "orderby": "business_title"
    }
```

La réponse pour cette requête doit ressembler à la capture d’écran suivante.

  :::image type="content" source="media/search-query-simple-examples/rangefiltertext.png" alt-text="Filtre de plage pour les plages de texte" border="false":::

> [!NOTE]
> L’utilisation de facettes sur des plages de valeurs est une condition d’application de recherche courante. Pour plus d’informations et d’exemples, consultez [Comment créer un filtre de facette](search-filters-facets.md).

## <a name="example-5-geo-search"></a>Exemple 5 : Recherche basée sur la localisation

L’exemple d’index inclut un champ geo_location avec des coordonnées de latitude et de longitude. Cet exemple utilise la [fonction geo.distance](search-query-odata-geo-spatial-functions.md#examples) qui applique un filtre sur les documents situés à une distance arbitraire (en kilomètres) d’un point de départ que vous spécifiez. Vous pouvez ajuster la dernière valeur de la requête (4) pour réduire ou étendre la surface de la requête.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "",
      "filter": "geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4",
      "select": "business_title, work_location"
    }
```

Pour améliorer la lisibilité des résultats de la recherche, ces derniers sont tronqués afin d’inclure un poste et le lieu de travail. Les coordonnées de départ ont été obtenues à partir d’un document aléatoire dans l’index (dans le cas présent, pour le lieu de travail Staten Island).

  :::image type="content" source="media/search-query-simple-examples/geo-search.png" alt-text="Carte de Staten Island" border="false":::

## <a name="example-6-search-precision"></a>Exemple 6 : Précision de la recherche

Les requêtes de termes portent sur des termes uniques, pouvant être nombreux, qui sont évalués de manière indépendante. Les requêtes d’expressions sont placées entre guillemets et évaluées comme une chaîne textuelle. La précision de la correspondance est contrôlée par des opérateurs et searchMode.

Exemple 1 : `search=fire` correspond à 140 résultats, où toutes les correspondances contiennent le mot fire quelque part dans le document.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "fire"
    }
```

Exemple 2 : `search=fire department` retourne 2002 résultats. Des correspondances sont retournées pour les documents contenant fire ou department.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "fire department"
    }
```

Exemple 3 : `search="fire department"` retourne 77 résultats. La mise de la chaîne entre guillemets a pour effet de créer une recherche d’expression composée des deux termes, et des correspondances sont trouvées sur des termes tokenisés dans l’index, consistant en la combinaison des termes. Cela explique pourquoi une recherche comme `search=+fire +department` n’est pas équivalente. Les deux termes sont obligatoires, mais sont analysés de manière indépendante. 

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
    "count": true,
    "search": "\"fire department\""
    }
```

> [!Note]
> Comme une requête d’expression est spécifiée à l’aide de guillemets, cet exemple ajoute un caractère d’échappement (`\`) pour conserver la syntaxe.

## <a name="example-7-booleans-with-searchmode"></a>Exemple 7 : Opérateurs booléens avec searchMode

La syntaxe simple prend en charge les opérateurs booléens sous la forme de caractères (`+, -, |`). Le paramètre searchMode indique le compromis entre précision et rappel, **`searchMode=any`** favorisant le rappel (la mise en correspondance sur n’importe quel critère qualifie un document pour le jeu de résultats), et **`searchMode=all`** favorisant la précision (tous les critères doivent être mis en correspondance). 

La valeur par défaut est **`searchMode=any`** , ce qui peut être déroutant si vous placez plusieurs opérateurs dans une requête et obtenez des résultats moins affinés que prévu. Cela est particulièrement vrai avec NOT, où les résultats incluent tous les documents « ne contenant pas » un terme spécifique.

Avec le searchMode par défaut (any), 2 800 documents sont retournés : ceux contenant l’expression « fire department », plus tous les documents ne contenant pas l’expression « Metrotech Center ».

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "\"fire department\"-\"Metrotech Center\"",
      "searchMode": "any"
    }
```

La réponse pour cette requête doit ressembler à la capture d’écran suivante.

  :::image type="content" source="media/search-query-simple-examples/searchmodeany.png" alt-text="Mode de recherche any" border="false":::

Affecter la valeur **`searchMode=all`** applique un effet cumulé aux critères et retourne un jeu de résultats plus petit (21 documents) constitué de documents contenant l’expression entière « fire department », moins les emplois à l’adresse du Metrotech Center.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "\"fire department\"-\"Metrotech Center\"",
      "searchMode": "all"
    }
```

  :::image type="content" source="media/search-query-simple-examples/searchmodeall.png" alt-text="Mode de recherche all" border="false":::

## <a name="example-8-structuring-results"></a>Exemple 8 : Structuration des résultats

Plusieurs paramètres contrôlent quels champs figurent dans les résultats de la recherche, le nombre de documents retournés dans chaque lot, ainsi que l’ordre de tri. Cet exemple reprend quelques-uns des exemples précédents, mais limite les résultats à des champs spécifiques à l’aide de l’instruction **`$select`** et de critères de recherche textuelle, retournant 82 correspondances.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "\"fire department\"",
      "searchMode": "any",
      "select": "job_id,agency,business_title,civil_service_title,work_location,job_description"
    }
```

En ajoutant à l’exemple précédent, vous pouvez trier par titre. Ce tri fonctionne car civil_service_title est *triable* dans l’index.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "\"fire department\"",
      "searchMode": "any",
      "select": "job_id,agency,business_title,civil_service_title,work_location,job_description",
      "orderby": "civil_service_title"
    }
```

La pagination des résultats est implémentée à l’aide du paramètre **`$top`** , retournant dans ce cas les cinq premiers documents :

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "\"fire department\"",
      "searchMode": "any",
      "select": "job_id,agency,business_title,civil_service_title,work_location,job_description",
      "orderby": "civil_service_title",
      "top": "5"
    }
```

Pour obtenir les cinq suivants, ignorez le premier lot :

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "count": true,
      "search": "\"fire department\"",
      "searchMode": "any",
      "select": "job_id,agency,business_title,civil_service_title,work_location,job_description",
      "orderby": "civil_service_title",
      "top": "5",
      "skip": "5"
    }
```

## <a name="next-steps"></a>Étapes suivantes

Essayez de spécifier des requêtes dans le code. Les liens suivants expliquent comment configurer des requêtes de recherche à l’aide des kits de développement logiciel (SDK) Azure.

+ [Interroger un index à l’aide du SDK .NET](search-get-started-dotnet.md)
+ [Interroger un index à l’aide du SDK Python](search-get-started-python.md)
+ [Interroger un index à l’aide du SDK JavaScript](search-get-started-javascript.md)

Vous trouverez des informations de référence supplémentaires sur la syntaxe et sur l’architecture de requête, ainsi que des exemples, en cliquant sur les liens suivants :

+ [Exemples de syntaxe de requête Lucene pour créer des requêtes avancées](search-query-lucene-examples.md)
+ [Fonctionnement de la recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md)
+ [Syntaxe de requête simple](query-simple-syntax.md)
+ [Syntaxe de requête complète Lucene](query-lucene-syntax.md)
+ [Syntaxe du filtre](search-query-odata-filter.md)