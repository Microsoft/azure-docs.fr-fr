---
title: Utiliser la syntaxe de requête Lucene complète
titleSuffix: Azure Cognitive Search
description: Syntaxe de requête Lucene pour la recherche approximative, la recherche de proximité, l’amélioration de termes, la recherche d’expressions régulières et la recherche par caractères génériques dans le service Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
tags: Lucene query analyzer syntax
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/05/2020
ms.openlocfilehash: ae4dd8b82e40b46da52a1b1f396569fda1dfea2b
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94694624"
---
# <a name="use-the-full-lucene-search-syntax-advanced-queries-in-azure-cognitive-search"></a>Utiliser la syntaxe de recherche Lucene « complète » (requêtes avancées dans Recherche cognitive Azure)

Lors de la construction de requêtes pour Recherche cognitive Azure, vous pouvez remplacer l’[analyseur de requêtes simple](query-simple-syntax.md) par défaut par l’[analyseur de requêtes Lucene dans Recherche cognitive Azure](query-lucene-syntax.md), plus vaste, afin de formuler des définitions de requêtes spécialisées et avancées. 

L’analyseur Lucene prend en charge des constructions de requêtes complexes, telles que des requêtes portant sur des champs, la recherche approximative, la recherche par caractères génériques infixes et suffixes, la recherche de proximité, la promotion de termes et la recherche d’expression régulière. Cette fonctionnalité plus puissante nécessite des capacités de traitement supplémentaires et peut donc entraîner des temps d’exécution un peu plus longs. Dans cet article, vous pouvez parcourir des exemples décrivant les opérations de requête disponibles lors de l’utilisation de la syntaxe complète.

> [!Note]
> Une grande partie des constructions de requêtes spécialisées activées par le biais de la syntaxe de requête Lucene complète ne sont pas soumises à une [analyse du texte](search-lucene-query-architecture.md#stage-2-lexical-analysis), ce qui peut être surprenant si vous prévoyez une recherche de radical ou une lemmatisation. L’analyse lexicale est effectuée uniquement sur des termes complets (requête sur un terme ou une expression). Les types de requête avec des termes incomplets (requête de préfixe, de caractère générique, d’expression régulière, partielle) sont ajoutés directement à l’arborescence de requête, en ignorant la phase d’analyse. La seule transformation effectuée sur les termes de requête partiels est l’utilisation de minuscules. 
>

## <a name="formulate-requests-in-postman"></a>Formuler des requêtes dans Postman

Les exemples suivants utilisent un index de recherche NYC Jobs composé de postes à pourvoir sur la base d’un jeu de données fourni par l’initiative [City of New York OpenData](https://opendata.cityofnewyork.us/). Ces données ne doivent pas être considérées comme étant à jour ou complètes. L’index se trouve sur un service de bac à sable fourni par Microsoft, ce qui signifie que vous n’avez pas besoin d’abonnement Azure ni de Recherche cognitive Azure pour essayer ces requêtes.

En revanche, vous avez besoin de Postman ou d’un outil équivalent pour émettre la requête HTTP sur GET. Pour plus d’informations, consultez l’article indiquant comment [explorer avec les clients REST](search-get-started-rest.md).

### <a name="set-the-request-header"></a>Définir l’en-tête de requête

1. Dans l’en-tête de requête, affectez la valeur `application/json` à **Content-Type**.

2. Ajoutez un **api-key** et affectez-lui cette chaîne : `252044BE3886FE4A8E3BAA4F595114BB`. Il s’agit d’une clé de requête pour le service de recherche de bac à sable qui héberge l’index NYC Jobs.

Une fois que vous avez spécifié l’en-tête de requête, vous pouvez le réutiliser pour toutes les requêtes dans cet article, en remplaçant uniquement la chaîne **search=** . 

  :::image type="content" source="media/search-query-lucene-examples/postman-header.png" alt-text="En-tête de demande Postman Définir les paramètres" border="false":::

### <a name="set-the-request-url"></a>Définir l’URL de requête

La requête est une commande GET accompagnée d’une URL contenant le point de terminaison de Recherche cognitive Azure et la chaîne de recherche.

  :::image type="content" source="media/search-query-lucene-examples/postman-basic-url-request-elements.png" alt-text="En-tête de demande Postman GET" border="false":::

L’URL est composée des éléments suivants :

+ **`https://azs-playground.search.windows.net/`** est un service de recherche de bac à sable tenu à jour par l’équipe de développement de Recherche cognitive Azure. 
+ **`indexes/nycjobs/`** est l’index NYC Jobs dans la collection d’index de ce service. Le nom du service et l’index sont tous deux obligatoires dans la requête.
+ **`docs`** est la collection de documents contenant tout le contenu disponible pour la recherche. La clé d’API de requête fournie dans l’en-tête de requête fonctionne uniquement sur les opérations de lecture ciblant la collection de documents.
+ **`api-version=2020-06-30`** définit la version de l’API, qui est un paramètre requis dans chaque requête.
+ **`search=*`** est la chaîne de requête qui, dans la requête initiale, est nulle, ce qui retourne les 50 premiers résultats (par défaut).

## <a name="send-your-first-query"></a>Envoyer votre première requête

En guise d’étape de vérification, collez la requête suivante dans GET et cliquez sur **Envoyer**. Les résultats sont retournés sous forme de documents JSON détaillés. Des documents entiers sont retournés, ce qui vous permet de voir tous les champs et toutes les valeurs.

Collez cette URL dans un client REST comme étape de validation et pour afficher la structure du document.

  ```http
  https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search=*
  ```

La chaîne de requête **`search=*`** , est une recherche non spécifiée équivalente à une recherche nulle ou vide. C’est la recherche la plus simple disponible.

Si vous le souhaitez, vous pouvez ajouter **`$count=true`** à l’URL pour retourner le nombre de documents correspondant aux critères de recherche. Une chaîne de recherche vide correspond à tous les documents figurant dans l’index (environ 2800 dans le cas de NYC Jobs).

## <a name="how-to-invoke-full-lucene-parsing"></a>Appel de l’analyse Lucene complète

Ajoutez **queryType=full** pour appeler la syntaxe de requête complète et substituer la syntaxe de requête simple par défaut. 

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&search=*
```

Tous les exemples de cet article spécifient le paramètre de requête **queryType=full**, ce qui indique la syntaxe complète est traitée par l’analyseur de requêtes Lucene. 

## <a name="example-1-query-scoped-to-a-list-of-fields"></a>Exemple 1 : Requête portée sur une liste de champs

Ce premier exemple n’est pas propre à Lucene, mais nous permet d’introduire le premier concept de requête fondamental : l’étendue d’un champ. Cet exemple limite l’exécution de la requête et la réponse à quelques champs spécifiques. Lors de l’utilisation de l’outil Postman ou Explorateur de recherche, il est important de connaître la structure d’une réponse JSON accessible en lecture. 

Par souci de concision, la requête cible uniquement le champ *business_title* et spécifie que seuls les titres de fonctions sont retournés. Le paramètre **searchFields** restreint l’exécution de la requête au champ business_title, et le paramètre **select** spécifie les champs inclus dans la réponse.

### <a name="search-expression"></a>Expression de recherche

```http
&search=*&searchFields=business_title&$select=business_title
```

Voici la même requête avec plusieurs champs dans une liste de valeurs séparées par des virgules.

```http
search=*&searchFields=business_title, posting_type&$select=business_title, posting_type
```

Les espaces après les virgules sont facultatifs.

> [!Tip]
> Lorsque vous utilisez l'API REST à partir du code de votre application, n'oubliez pas les paramètres d'encodage d'URL comme `$select` et `searchFields`.

### <a name="full-url"></a>URL complète

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&search=*&searchFields=business_title&$select=business_title
```

La réponse pour cette requête doit ressembler à la capture d’écran suivante.

  ![Exemple de réponse Postman avec scores](media/search-query-lucene-examples/postman-sample-results.png)

Vous avez peut-être remarqué le score de recherche dans la réponse. Des scores uniformes de 1 sont obtenus en l’absence de classement, soit parce que la recherche n’était pas une recherche en texte intégral, soit parce qu’aucun critère n’a été appliqué. Pour la recherche de valeur Null sans aucun critère, les lignes sont renvoyées dans un ordre arbitraire. Si vous incluez des critères de recherche réels, vous constaterez que les scores de recherche deviendront des valeurs significatives.

## <a name="example-2-fielded-search"></a>Exemple 2 : Recherche par champ

La syntaxe Lucene complète permet de restreindre des expressions de recherche individuelles à un champ spécifique. Cet exemple recherche les titres de fonctions contenant le terme « senior » mais pas « junior ».

### <a name="search-expression"></a>Expression de recherche

```http
$select=business_title&search=business_title:(senior NOT junior)
```

Voici la même requête avec plusieurs champs.

```http
$select=business_title, posting_type&search=business_title:(senior NOT junior) AND posting_type:external
```

### <a name="full-url"></a>URL complète

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&$select=business_title&search=business_title:(senior NOT junior)
```

  :::image type="content" source="media/search-query-lucene-examples/intrafieldfilter.png" alt-text="Expression de recherche de réponse de l’exemple Postman" border="false":::

Vous pouvez définir une opération de recherche par champ avec la syntaxe **fieldName:searchExpression**, où l'expression de recherche peut être un mot ou une phrase, ou une expression plus complexe entre parenthèses, éventuellement avec des opérateurs booléens. Voici quelques exemples :

- `business_title:(senior NOT junior)`
- `state:("New York" OR "New Jersey")`
- `business_title:(senior NOT junior) AND posting_type:external`

Veillez à placer les chaînes entre guillemets si vous souhaitez que les deux chaînes soient évaluées comme une seule entité, comme ici où deux emplacements distincts sont recherchés dans le champ `state`. Vérifiez également que l’opérateur est en majuscules, comme c’est le cas ici avec NOT et AND.

Le champ spécifié dans **fieldName:searchExpression** doit être un champ pouvant faire l’objet d’une recherche. Pour plus d’informations sur l’utilisation des attributs d’index dans les définitions de champs, consultez [Créer un index (API REST Recherche cognitive Azure)](/rest/api/searchservice/create-index).

> [!NOTE]
> Dans l’exemple ci-dessus, nous n’avons pas eu besoin d’utiliser le paramètre `searchFields` car chaque partie de la requête comporte un nom de champ explicitement spécifié. Cependant, vous pouvez toujours utiliser le paramètre `searchFields` si vous voulez exécuter une requête où certaines parties sont limitées à un champ spécifique, et le reste peut s’appliquer à plusieurs champs. Par exemple, la requête `search=business_title:(senior NOT junior) AND external&searchFields=posting_type` ne correspondrait à `senior NOT junior` qu'au niveau du champ `business_title`, alors qu'elle correspondrait au champ « externe » avec le champ `posting_type`. Le nom du champ fourni dans **fieldName:searchExpression** a toujours priorité sur le paramètre `searchFields`, c'est pourquoi dans cet exemple, nous n'avons pas besoin d'inclure `business_title` dans le paramètre `searchFields`.

## <a name="example-3-fuzzy-search"></a>Exemple 3 : Recherche partielle

La syntaxe Lucene complète prend également en charge la recherche approximative, avec une mise en correspondance des termes qui ont une construction similaire. Pour effectuer une recherche partielle, ajoutez le signe tilde `~` à la fin d’un mot avec un paramètre facultatif, une valeur comprise entre 0 et 2, qui spécifie la distance de modification. Par exemple, `blue~` ou `blue~1` retournent blue, blues et glue.

### <a name="search-expression"></a>Expression de recherche

```http
searchFields=business_title&$select=business_title&search=business_title:asosiate~
```

Les phrases ne sont pas prises en charge directement, mais vous pouvez spécifier une correspondance partielle pour différents éléments d’une phrase.

```http
searchFields=business_title&$select=business_title&search=business_title:asosiate~ AND comm~ 
```


### <a name="full-url"></a>URL complète

Cette requête recherche les postes à pourvoir contenant le terme « associate » (délibérément mal orthographié) :

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:asosiate~
```
  ![Réponse de recherche partielle](media/search-query-lucene-examples/fuzzysearch.png)


> [!Note]
> Les requêtes approximatives ne sont pas [analysées](search-lucene-query-architecture.md#stage-2-lexical-analysis). Les types de requête avec des termes incomplets (requête de préfixe, de caractère générique, d’expression régulière, partielle) sont ajoutés directement à l’arborescence de requête, en ignorant la phase d’analyse. La seule transformation effectuée sur les termes de requête incomplets est l’utilisation de minuscules.
>

## <a name="example-4-proximity-search"></a>Exemple 4 : Recherche de proximité
Les recherches de proximité servent à rechercher des termes qui sont proches les uns des autres dans un document. Insérez un signe tilde « ~ » à la fin d’une expression, suivi du nombre de mots qui créent la limite de proximité. Par exemple, "hotel airport"~5 recherche les termes hotel et airport distants de cinq mots ou moins dans un document.

### <a name="search-expression"></a>Expression de recherche

```http
searchFields=business_title&$select=business_title&search=business_title:%22senior%20analyst%22~1
```

### <a name="full-url"></a>URL complète

Dans cette requête, on recherche les postes contenant le terme « senior analyst » où les deux mots sont séparés au plus par un mot :

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:%22senior%20analyst%22~1
```
  :::image type="content" source="media/search-query-lucene-examples/proximity-before.png" alt-text="Requête de proximité" border="false":::

Réessayez, mais en supprimant les mots entre « senior analyst ». Notez que huit documents sont retournés pour cette requête, par opposition à 10 pour la requête précédente.

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:%22senior%20analyst%22~0
```

## <a name="example-5-term-boosting"></a>Exemple 5 : Promotion de termes
La promotion de termes signifie que vous pouvez accorder à un document un rang plus élevé s’il contient le terme promu, par rapport aux documents qui ne contiennent pas ce terme. Pour promouvoir un terme, utilisez le signe « ^ » avec un facteur de promotion (un nombre) à la fin du terme recherché. 

### <a name="full-urls"></a>URL complètes

Dans cette requête « avant », on recherche les postes à pourvoir contenant le terme *computer analyst* et on constate l’absence de résultat avec les deux mots *computer* et *analyst*. Par contre, les postes *computer* figurent en haut des résultats.

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:computer%20analyst
```
  :::image type="content" source="media/search-query-lucene-examples/termboostingbefore.png" alt-text="Promotion de termes avant" border="false":::

Dans la requête « après », on répète la recherche, cette fois en promouvant les résultats contenant le terme *analyst* par rapport au terme *computer* si les deux mots ensemble n’existent pas. 

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:computer%20analyst%5e2
```
Une version plus lisible de la requête ci-dessus est `search=business_title:computer analyst^2`. Pour obtenir une requête exploitable, `^2` est encodé sous la forme `%5E2`, ce qui est plus difficile à voir.

  :::image type="content" source="media/search-query-lucene-examples/termboostingafter.png" alt-text="Promotion de termes après" border="false":::

Il ne faut pas confondre la promotion de termes avec les profils de score, qui promeuvent certains champs plutôt que des termes spécifiques. L’exemple suivant permet d’illustrer les différences entre les deux.

Prenez un profil de score qui promeut les correspondances figurant dans un certain champ, par exemple **genre** dans l’exemple musicstoreindex. En utilisant la promotion de termes, vous pouvez promouvoir encore plus certains termes de recherche. Par exemple, "rock^2 electronic" promeut les documents qui contiennent les termes de recherche dans le champ **genre** , par rapport aux autres champs de recherche de l’index. Par ailleurs, les documents qui contiennent le terme de recherche « rock » bénéficient d’un classement plus élevé que ceux qui contiennent le terme de recherche « electronic » en raison de la valeur de promotion du terme (2).

Lors de la définition du niveau de facteur, plus le facteur de promotion est élevé, plus le terme est pertinent par rapport aux autres termes de recherche. Par défaut, le facteur de promotion est égal à 1. Ce facteur doit être positif, mais il peut être inférieur à 1 (par exemple 0,2).


## <a name="example-6-regex"></a>Exemple 6 : Expression régulière

Une recherche d’expression régulière trouve une correspondance en fonction du contenu placé entre des barres obliques « / », comme le décrit la [classe RegExp](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/util/automaton/RegExp.html).

### <a name="search-expression"></a>Expression de recherche

```http
searchFields=business_title&$select=business_title&search=business_title:/(Sen|Jun)ior/
```

### <a name="full-url"></a>URL complète

Dans cette requête, recherchez les postes à pourvoir contenant le terme « Senior » ou « Junior » : `search=business_title:/(Sen|Jun)ior/`.

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:/(Sen|Jun)ior/
```

  :::image type="content" source="media/search-query-lucene-examples/regex.png" alt-text="Requête par expression régulière" border="false":::

> [!Note]
> Les requêtes Regex ne sont pas [analysées](./search-lucene-query-architecture.md#stage-2-lexical-analysis). La seule transformation effectuée sur les termes de requête incomplets est l’utilisation de minuscules.
>

## <a name="example-7-wildcard-search"></a>Exemple 7 : Recherche par caractères génériques
Vous pouvez utiliser la syntaxe généralement reconnue pour effectuer des recherches avec plusieurs caractères génériques (\*) ou un caractère générique unique (?). Notez que l’Analyseur de requêtes Lucene prend en charge l’utilisation de ces symboles avec un terme unique, et non une expression.

### <a name="search-expression"></a>Expression de recherche

```http
searchFields=business_title&$select=business_title&search=business_title:prog*
```

### <a name="full-url"></a>URL complète

Dans cette requête, on recherche les postes à pourvoir qui contiennent le préfixe « prog », par exemple ceux contenant les termes « programmation » et « programmeur ». Vous ne pouvez pas utiliser un signe * ou ? comme premier caractère d’une recherche.

```GET
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&queryType=full&$count=true&searchFields=business_title&$select=business_title&search=business_title:prog*
```
  :::image type="content" source="media/search-query-lucene-examples/wildcard.png" alt-text="Requête par caractère générique" border="false":::

> [!Note]
> Les requêtes par caractères génériques ne sont pas [analysées](./search-lucene-query-architecture.md#stage-2-lexical-analysis). La seule transformation effectuée sur les termes de requête incomplets est l’utilisation de minuscules.
>

## <a name="next-steps"></a>Étapes suivantes
Essayez de spécifier l’Analyseur de requêtes Lucene dans votre code. Les liens suivants expliquent comment configurer des requêtes de recherche pour .NET et l’API REST. Ces liens utilisent la syntaxe simple par défaut. Vous devrez donc appliquer ce que vous avez appris dans cet article pour spécifier le **queryType**.

* [Interroger un index à l’aide du SDK .NET](./search-get-started-dotnet.md)
* [Interroger un index à l’aide de l’API REST](./search-get-started-powershell.md)

Vous trouverez des informations de référence supplémentaires sur la syntaxe et sur l’architecture de requête, ainsi que des exemples, en cliquant sur les liens suivants :

+ [Exemples de requête de syntaxe simple](search-query-simple-examples.md)
+ [Fonctionnement de la recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md)
+ [Syntaxe de requête simple](/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [Syntaxe de requête complète Lucene](/rest/api/searchservice/lucene-query-syntax-in-azure-search)