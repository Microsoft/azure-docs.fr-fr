---
title: Types de requêtes et composition
titleSuffix: Azure Cognitive Search
description: Principes de base pour la création d’une requête de recherche dans Recherche cognitive Azure, avec utilisation de paramètres pour filtrer, sélectionner et trier les résultats.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: 362f46290bbe2008f9fb862a8711577050050192
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94693247"
---
# <a name="query-types-and-composition-in-azure-cognitive-search"></a>Types et composition de requête dans Recherche cognitive Azure

Dans Recherche cognitive Azure, une requête est une spécification complète d’une opération d’aller-retour. Dans la requête, il existe des paramètres qui fournissent des instructions d’exécution pour le moteur, ainsi que des paramètres qui forment la réponse en retour. Non spécifiée (`search=*`), sans critères de correspondance et qui utilise des paramètres nuls ou par défaut, une requête s’exécute sur tous les champs recherchables en tant qu’opération de recherche de texte intégral et retourne un jeu de résultats sans score dans un ordre arbitraire.

L’exemple suivant est une requête représentative construite dans l’[API REST](/rest/api/searchservice/search-documents). Cet exemple cible l’[index de démonstration des hôtels](search-get-started-portal.md) et comprend des paramètres communs afin que vous puissiez vous faire une idée de ce à quoi ressemble une requête.

```
{
    "queryType": "simple" 
    "search": "+New York +restaurant",
    "searchFields": "Description, Address/City, Tags",
    "select": "HotelId, HotelName, Description, Rating, Address/City, Tags",
    "top": "10",
    "count": "true",
    "orderby": "Rating desc"
}
```

+ **`queryType`** définit l’analyseur qui peut être l’[analyseur de requêtes simple par défaut](search-query-simple-examples.md) (optimal pour la recherche en texte intégral), ou l’[analyseur de requêtes complet Lucene](search-query-lucene-examples.md) utilisé pour les constructions de requêtes avancées, telles que les expressions régulières, la recherche de proximité, la recherche approximative et par caractères génériques et bien d’autres encore.

+ **`search`** fournit le critère de correspondance, généralement des termes ou expressions, qui est néanmoins souvent accompagné d’opérateurs booléens. Les termes autonomes uniques constituent des requêtes de *terme*. Les requêtes en plusieurs parties entre guillemets sont des requêtes *d’expression*. La recherche peut être non définie, comme dans **`search=*`** , mais sans critère de correspondance, le jeu de résultats est composé de documents sélectionnés de manière arbitraire.

+ **`searchFields`** limite l’exécution des requêtes à des champs spécifiques. Tout champ attribué en tant que champ avec *possibilité de recherche* dans le schéma d’index peut être utilisé avec ce paramètre.

Les réponses sont également mises en forme par les paramètres que vous incluez dans la requête :

+ **`select`** spécifie les champs à retourner dans la réponse. Seuls les champs marqués comme *récupérables* dans l’index peuvent être utilisés dans une instruction select.

+ **`top`** retourne le nombre de documents les mieux correspondants spécifié. Dans cet exemple, seuls 10 correspondances sont retournées. Vous pouvez utiliser Top et Skip (non affichés) pour paginer les résultats.

+ **`count`** indique le nombre total de documents dans l’index complet, ce qui peut être supérieur à ce qui est retourné. 

+ **`orderby`** est utilisé si vous souhaitez trier les résultats en fonction d’une valeur, telle qu’une évaluation ou un emplacement. Dans le cas contraire, la valeur par défaut consiste à utiliser le score de pertinence pour classer les résultats.

Dans Recherche cognitive Azure, la requête s’exécute toujours sur un seul index qui est authentifié à l’aide d’une clé d’API (api-key) fournie dans la requête. Dans l’API REST, les deux éléments sont fournis dans les en-têtes de la requête.

### <a name="how-to-run-this-query"></a>Comment exécuter cette requête

Avant d’écrire du code, vous pouvez utiliser les outils de requête pour apprendre la syntaxe et tester différents paramètres. L’approche la plus rapide est l’outil de portail intégré, l’[Explorateur de recherche](search-explorer.md).

Si vous avez suivi ce [démarrage rapide pour créer l’index de démonstration des hôtels](search-get-started-portal.md), vous pouvez coller cette chaîne de requête dans la barre de recherche de l’Explorateur pour exécuter votre première requête : `search=+"New York" +restaurant&searchFields=Description, Address/City, Tags&$select=HotelId, HotelName, Description, Rating, Address/City, Tags&$top=10&$orderby=Rating desc&$count=true`

## <a name="how-query-operations-are-enabled-by-the-index"></a>Comment les opérations de requête sont activées par l’index

La conception des index et la conception des requêtes sont étroitement liées dans Recherche cognitive Azure. Il est important de savoir que le *schéma d’index*, qui comporte des attributs pour chaque champ, détermine le type de requête que vous pouvez créer. 

Les attributs d’index d’un champ définissent les opérations autorisées, par exemple si un champ *peut faire l’objet de recherches* dans l’index, s’il peut être *affiché* dans les résultats, s’il peut être *trié* ou *filtré*, etc. Dans l’exemple de chaîne de requête, `"$orderby": "Rating"` fonctionne uniquement parce que le champ Évaluation est marqué comme *triable* dans le schéma d’index. 

![Définition d’index pour l’exemple des hôtels](./media/search-query-overview/hotel-sample-index-definition.png "Définition d’index pour l’exemple Hôtel")

La capture d’écran ci-dessus présente une liste partielle des attributs d’index pour l’exemple des hôtels. Vous pouvez consulter le schéma d’index complet dans le portail. Pour en savoir plus sur les attributs d’index, consultez [Création d’une API REST d’index](/rest/api/searchservice/create-index).

> [!Note]
> Certaines fonctionnalités de requête s’appliquent à l’ensemble de l’index plutôt qu’à des champs spécifiques. Ces fonctionnalités incluent : les [cartes de synonymes](search-synonyms.md), les [analyseurs personnalisés](index-add-custom-analyzers.md), les [constructions de générateur de suggestions (pour les requêtes de saisie semi-automatique et de suggestion automatique)](index-add-suggesters.md) et la [logique de notation pour le classement des résultats](index-add-scoring-profiles.md).

## <a name="elements-of-a-query-request"></a>Éléments d’une demande de requête

Les requêtes s’appliquent toujours à un index unique. Vous ne pouvez pas joindre des index ou créer des structures de données personnalisées ou temporaires en tant que requête cible. 

Les éléments obligatoires dans une demande de requête incluent les composants suivants :

+ Collection des points de terminaison de service et des documents d’index, exprimée sous la forme d’une URL contenant les composants fixes et définis par l’utilisateur : **`https://<your-service-name>.search.windows.net/indexes/<your-index-name>/docs`**
+ **`api-version`** (API REST uniquement) est requis car plusieurs versions de l’API sont disponibles en permanence. 
+ **`api-key`** , représenté par une clé d’API de requête ou administration, authentifie la requête auprès votre service.
+ **`queryType`** , simple ou full, qui peut être omis si vous utilisez la syntaxe simple par défaut intégrée.
+ **`search`** ou **`filter`** fournit les critères de recherche, qui peuvent être non spécifiés si vous souhaitez effectuer une recherche vide. Les deux types de requêtes sont décrits dans le cadre d’un analyseur simple, mais même les requêtes avancées requièrent le paramètre de recherche pour prendre en compte les expressions de requêtes complexes.

Tous les autres paramètres de recherche sont facultatifs. Pour obtenir la liste complète des attributs, consultez la section relative à la [création d’index (REST)](/rest/api/searchservice/create-index). Pour en savoir plus sur le rôle des paramètres lors du traitement des requêtes, consultez [Fonctionnement de la recherche en texte intégral dans Recherche cognitive Azure](search-lucene-query-architecture.md).

## <a name="choose-apis-and-tools"></a>Choisir les API et les outils

Le tableau suivant liste les API et les approches basées sur des outils pour envoyer des requêtes.

| Méthodologie | Description |
|-------------|-------------|
| [Navigateur de recherche (portail)](search-explorer.md) | Fournit une barre de recherche et des options pour les sélections d’index et de version d’API. Les résultats sont retournés sous forme de documents JSON. Il est recommandé pour l’exploration, le test et la validation. <br/>[En savoir plus.](search-get-started-portal.md#query-index) | 
| [Postman ou autres outils REST](search-get-started-rest.md) | Les outils de test web constituent un excellent choix pour formuler des appels REST. L’API REST prend en charge toutes les opérations possibles dans Recherche cognitive Azure. Dans cet article, découvrez comment configurer un en-tête et un corps de requête HTTP pour l’envoi de requêtes à Recherche cognitive Azure.  |
| [SearchClient (.NET)](/dotnet/api/azure.search.documents.searchclient) | Client que vous pouvez utiliser pour interroger un index Recherche cognitive Azure.  <br/>[En savoir plus.](search-howto-dotnet-sdk.md)  |
| [Rechercher des documents (API REST)](/rest/api/searchservice/search-documents) | Méthodes GET ou POST sur un index, avec paramètres de requête pour une entrée supplémentaire.  |

## <a name="choose-a-parser-simple--full"></a>Choisir un analyseur : simple | full

Le service Recherche cognitive Azure repose sur Apache Lucene et vous offre le choix entre deux analyseurs de requêtes pour la gestion des requêtes classiques et spécialisées. Les requêtes qui utilisent l’analyseur simple sont formulées à l’aide de la [syntaxe de requête simple](query-simple-syntax.md), qui est sélectionnée par défaut pour sa rapidité et son efficacité dans les requêtes de texte au format libre. Cette syntaxe prend en charge un certain nombre d’opérateurs de recherche courants, notamment les opérateurs AND, OR, NOT, les expressions, les suffixes et les opérateurs de priorité.

La [syntaxe de requête complète Lucene](query-Lucene-syntax.md#bkmk_syntax), activée lorsque vous ajoutez `queryType=full` à la requête, expose le langage de requête expressif et largement adopté développé dans le cadre [d’Apache Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html). La syntaxe complète étend la syntaxe simple. Toute requête que vous écrivez avec la syntaxe simple s’exécute dans l’analyseur Lucene complet. 

Les exemples suivants illustrent parfaitement la situation : la même requête, mais avec des paramètres queryType différents, génère des résultats différents. Dans la première requête, l’élément `^3` après `historic` est traité comme une partie du terme recherché. Pour cette requête, le résultat qui arrive en tête du classement est « Marquis Plaza & Suites », dont la description comprend le terme *ocean*.

```
queryType=simple&search=ocean historic^3&searchFields=Description, Tags&$select=HotelId, HotelName, Tags, Description&$count=true
```

La même requête exécutée à l’aide de l’analyseur Lucene complet interprète `^3` comme élément de champ prioritaire. Le changement d’analyseur modifie le classement. Ainsi, les résultats contenant le terme *historic* arrivent en tête.

```
queryType=full&search=ocean historic^3&searchFields=Description, Tags&$select=HotelId, HotelName, Tags, Description&$count=true
```

<a name="types-of-queries"></a>

## <a name="types-of-queries"></a>Types de requête

Recherche cognitive Azure prend en charge un large éventail de types de requêtes. 

| Type de requête | Usage | Exemples et informations complémentaires |
|------------|--------|-------------------------------|
| Recherche de texte de forme libre | Paramètre de recherche et analyseur au choix| Une recherche en texte intégral recherche un ou plusieurs termes dans tous les champs *pouvant faire l’objet d’une recherche* de votre index, et fonctionne à l’instar des moteurs de recherche Google ou Bing. L’exemple dans l’introduction est une recherche en texte intégral.<br/><br/>La recherche en texte intégral fait l’objet d’une analyse lexicale à l’aide de l’analyseur Lucene standard (par défaut) pour mettre tous les termes en minuscules et supprimer les mots exclus tels que « the » (la, le, les). Vous pouvez remplacer l’analyseur par défaut en choisissant un [analyseur non anglais](index-add-language-analyzers.md#language-analyzer-list) ou un [analyseur spécialisé indépendant des langages](index-add-custom-analyzers.md#AnalyzerTable) qui modifiera les paramètres d’analyse lexicale. Un analyseur [mot clé](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) traite par exemple tout le contenu d’un champ comme un jeton unique. Ceci est utile pour les données comme les codes postaux, les ID et certains noms de produit. | 
| Recherche filtrée | [Expression de filtre OData](query-odata-filter-orderby-syntax.md) et analyseur au choix | Les requêtes de filtre évaluent une expression booléenne dans tous les champs *filtrables* d’un index. Contrairement à une recherche, une requête de filtre établit une correspondance avec le contenu exact d’un champ, y compris la casse dans les champs de type chaîne. Une autre différence est que les requêtes de filtre sont exprimées dans la syntaxe OData. <br/>[Exemple d’expression de filtre](search-query-simple-examples.md#example-3-filter-queries) |
| Recherche basée sur la localisation | Champ de [type Edm.GeographyPoint](/rest/api/searchservice/supported-data-types), expression de filtre et analyseur au choix | Les coordonnées stockées dans un champ de type Edm.GeographyPoint sont utilisées pour les recherches de type « rechercher à proximité » ou basées sur une carte. <br/>[Exemple de recherche sur la localisation](search-query-simple-examples.md#example-5-geo-search)|
| Recherche de plage | expression de filtre et analyseur simple | Dans Recherche cognitive Azure, les requêtes de plage sont créées à l’aide du paramètre de filtre. <br/>[Exemple de filtre de plage](search-query-simple-examples.md#example-4-range-filters) | 
| [Recherche par champ](query-lucene-syntax.md#bkmk_fields) | Paramètre de recherche et analyseur complet | Crée une expression de requête composite ciblant un champ unique. <br/>[Exemple de recherche par champ](search-query-lucene-examples.md#example-2-fielded-search) |
| [recherche approximative](query-lucene-syntax.md#bkmk_fuzzy) | Paramètre de recherche et analyseur complet | Recherche les termes ayant une construction ou une orthographe similaire. <br/>[Exemple de recherche approximative](search-query-lucene-examples.md#example-3-fuzzy-search) |
| [recherche de proximité](query-lucene-syntax.md#bkmk_proximity) | Paramètre de recherche et analyseur complet | Recherche les termes proches les uns des autres dans un document. <br/>[Exemple de recherche de proximité](search-query-lucene-examples.md#example-4-proximity-search) |
| [promotion de termes](query-lucene-syntax.md#bkmk_termboost) | Paramètre de recherche et analyseur complet | Élève le rang d’un document qui contient le terme de promotion, par rapport aux documents qui ne contiennent pas ce terme. <br/>[Exemple de promotion de termes](search-query-lucene-examples.md#example-5-term-boosting) |
| [recherche d’expression régulière](query-lucene-syntax.md#bkmk_regex) | Paramètre de recherche et analyseur complet | Exécute la recherche à partir du contenu d’une expression régulière. <br/>[Exemple de recherche d’expression régulière](search-query-lucene-examples.md#example-6-regex) |
|  [recherche par préfixe ou caractères génériques](query-lucene-syntax.md#bkmk_wildcard) | Paramètre de recherche et analyseur complet | Exécute la recherche à partir d’un préfixe et d’un tilde (`~`) ou d’un caractère unique (`?`). <br/>[Exemple de recherche par caractères génériques](search-query-lucene-examples.md#example-7-wildcard-search) |

## <a name="manage-search-results"></a>Gérer les résultats de recherche 

Les résultats de la requête sont envoyés sous forme de documents JSON dans l’API REST, mais si vous utilisez des API .NET, la sérialisation est intégrée. Vous pouvez mettre en forme les résultats en définissant des paramètres sur la requête, et en sélectionnant des champs spécifiques pour la réponse.

Vous pouvez utiliser les paramètres de la requête pour structurer le jeu de résultats des manières suivantes :

+ Limitation ou traitement par lot du nombre de documents dans les résultats (50 par défaut)
+ Sélection des champs à inclure dans les résultats
+ Définition d’un ordre de tri
+ Ajout de la mise en surbrillance des correspondances pour attirer l’attention sur les termes correspondants dans le corps des résultats de recherche

### <a name="tips-for-unexpected-results"></a>Conseils en cas de résultats inattendus

Parfois, la substance et non la structure de résultats est inattendue. Quand les résultats de requête ne sont pas à la hauteur de vos attentes, vous pouvez essayer ces modifications de requête pour voir si les résultats s’améliorent :

+ Remplacez **`searchMode=any`** (valeur par défaut) par **`searchMode=all`** pour exiger des correspondances sur tous les critères plutôt que sur un seul d’entre eux. Cela s’applique particulièrement quand des opérateurs booléens sont inclus dans la requête.

+ Changez la technique de requête si l’analyse lexicale ou du texte est nécessaire, mais que le type de requête exclut tout traitement linguistique. Dans la recherche en texte intégral, l’analyse lexicale ou du texte corrige automatiquement les fautes d’orthographe, les formes singulier-pluriel des noms, et même les noms ou les verbes irréguliers. Pour certaines requêtes telles que la recherche approximative ou par caractères génériques, l’analyse lexicale ne fait pas partie du pipeline d’analyse de requête. Dans certains scénarios, des expressions régulières ont été utilisées pour contourner ce problème. 

### <a name="paging-results"></a>Résultats de pagination
Recherche cognitive Azure facilite l’implémentation de la pagination des résultats de recherche. À l’aide des paramètres **`top`** et **`skip`** , vous pouvez facilement émettre des requêtes de recherche pour recevoir le jeu de résultats complet dans des sous-ensembles gérables, ordonnés qui permettent de bonnes pratiques de recherche dans l’interface utilisateur. Lors de la réception de ces sous-ensembles de résultats plus petits, vous pouvez également recevoir le nombre de documents dans l’ensemble total des résultats de la recherche.

Pour plus d’informations sur la pagination des résultats de recherche, consultez l’article [Navigation dans les résultats de recherche de Recherche cognitive Azure](search-pagination-page-layout.md).

### <a name="ordering-results"></a>Classement des résultats
Lors de la réception des résultats d’une requête de recherche, vous pouvez demander que Recherche cognitive Azure produise les résultats classés par valeurs dans un champ spécifique. Par défaut, Recherche cognitive Azure classe les résultats en fonction du rang du résultat de la recherche de chaque document, qui est dérivé de la méthode [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf).

Si vous souhaitez que Recherche cognitive Azure retourne les résultats en les classant avec une valeur autre que le résultat de la recherche, vous pouvez utiliser le paramètre de recherche **`orderby`** . Vous pouvez spécifier la valeur du paramètre **`orderby`** pour inclure les noms de champ et les appels à la fonction [ **`geo.distance()`**](query-odata-filter-orderby-syntax.md) pour les valeurs géospatiales. Chaque expression peut être suivie par `asc` pour indiquer que les résultats sont demandés dans l’ordre croissant, et par **`desc`** pour indiquer que les résultats sont demandés dans l’ordre décroissant. Le classement par défaut est l’ordre croissant.


### <a name="hit-highlighting"></a>Mise en surbrillance des correspondances
Dans Recherche cognitive Azure, vous pouvez mettre facilement en évidence la partie exacte des résultats de recherche qui correspondent à la requête de recherche en utilisant les paramètres **`highlight`** , **`highlightPreTag`** et **`highlightPostTag`** . Vous pouvez spécifier les champs *utilisables dans une recherche* dont le texte correspondant à la requête doit être mis en évidence ainsi que les balises de chaîne exactes à ajouter au début et à la fin du texte correspondant retourné par le service Recherche cognitive Azure.

## <a name="see-also"></a>Voir aussi

+ [Fonctionnement de la recherche en texte intégral dans Recherche cognitive Azure (architecture d’analyse de requête)](search-lucene-query-architecture.md)
+ [Navigateur de recherche](search-explorer.md)
+ [Guide pratique pour interroger dans .NET](./search-get-started-dotnet.md)
+ [Guide pratique pour interroger dans REST](./search-get-started-powershell.md)