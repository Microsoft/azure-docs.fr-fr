---
title: Recherche partielle
titleSuffix: Azure Cognitive Search
description: Implémentez une expérience de recherche « vouliez-vous dire » pour corriger automatiquement un terme mal orthographié ou une faute de frappe.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/08/2020
ms.openlocfilehash: ac2690a5f18bb58c29b433f4a07e52096bbd268b
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701583"
---
# <a name="fuzzy-search-to-correct-misspellings-and-typos"></a>Recherche approximative pour corriger les fautes d’orthographe et de frappe

Recherche cognitive Azure prend en charge la recherche approximative, type de requête qui pallie les fautes d’orthographe et les termes mal orthographiés dans la chaîne d’entrée. Pour ce faire, le service analyse les termes ayant une composition similaire. L’extension de la recherche pour couvrir les correspondances proches a pour effet de corriger automatiquement une faute de frappe quand l’écart n’est qu’un petit nombre de caractères mal placés. 

## <a name="what-is-fuzzy-search"></a>Qu’est-ce que la recherche approximative ?

Il s’agit d’un exercice d’expansion qui produit une correspondance avec les termes ayant une composition similaire. Quand une recherche approximative est spécifiée, le moteur génère un graphe (basé sur la [théorie de l’automate fini déterministe](https://en.wikipedia.org/wiki/Deterministic_finite_automaton)) des termes composés de la même façon, pour l’ensemble des termes de la requête. Par exemple, si votre requête inclut les trois termes « university of washington », un graphe est créé pour chaque terme de la requête `search=university~ of~ washington~` (comme il n’y a pas de suppression des mots vides dans la recherche approximative, « of » obtient un graphe).

Le graphe se compose d’un maximum de 50 expansions, ou permutations, de chaque terme, capturant à la fois les variantes correctes et incorrectes dans le processus. Le moteur retourne ensuite les correspondances les plus pertinentes dans la réponse. 

Pour un terme comme « university », le graphe peut comporter « unversty, universty, university, universe, inverse ». Tous les documents qui correspondent à ces termes du graphe sont inclus dans les résultats. Contrairement à d’autres requêtes qui analysent le texte pour gérer les différentes formes du même mot (« mice » et « mouse »), les comparaisons dans une requête approximative sont effectuées telles quelles sans aucune analyse linguistique du texte. Les termes « universe » et « inverse », qui sont sémantiquement différents, constituent des correspondances, car les différences syntaxiques sont minimes.

Une correspondance est établie si les différences sont limitées à, au plus, deux modifications, une modification étant un caractère inséré, supprimé, substitué ou transposé. L’algorithme de correction de chaîne qui spécifie la valeur différentielle est la métrique [Distance de Damerau-Levenshtein](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance), décrite comme étant le nombre minimum d’opérations (insertions, suppressions, substitutions ou transpositions de deux caractères adjacents) nécessaires pour transformer une chaîne de caractères en une autre. 

Dans Recherche cognitive Azure :

+ La requête approximative s’applique aux termes entiers, mais vous pouvez prendre en charge des expressions par le biais de constructions ET. Par exemple, « Unviersté~ de~ « Wshington~ » correspondrait à « Université de Washington ».

+ La distance par défaut d’une modification est 2. La valeur `~0` signifie qu’il n’y a pas d’expansion (seul le terme exact est considéré comme une correspondance), mais vous pouvez spécifier `~1` pour un degré de différence ou une modification. 

+ Une requête approximative peut étendre un terme jusqu’à 50 permutations supplémentaires. Cette limite n’est pas configurable, mais vous pouvez réduire efficacement le nombre d’expansions en diminuant la distance de modification à 1.

+ Les réponses se composent de documents contenant une correspondance pertinente (jusqu’à 50).

Collectivement, les graphes sont soumis en tant que critères de correspondance par rapport aux jetons de l’index. Comme vous pouvez l’imaginer, la recherche approximative est fondamentalement plus lente que les autres formes de requête. La taille et la complexité de votre index peuvent déterminer si les avantages sont suffisants pour compenser la latence de la réponse.

> [!NOTE]
> La recherche approximative ayant tendance à être lente, il peut être utile d’examiner d’autres approches, telles que l’indexation N-gramme, avec sa progression de séquences de caractères courtes (séquences de deux et trois caractères pour les jetons bigrammes et trigrammes). En fonction de votre langue et de la surface d’exposition de la requête, l’approche N-gramme peut offrir de meilleures performances. Sachez toutefois que l’indexation N-gramme est très gourmande en stockage et génère des index beaucoup plus volumineux.
>
> Une autre solution, que vous pouvez envisager si vous souhaitez gérer uniquement les cas les plus flagrants, est une [carte de synonymes](search-synonyms.md). Par exemple, vous pouvez établir une correspondance entre « search » et « serach, serch, sarch » ou entre « retrieve » et « retreive ».

## <a name="indexing-for-fuzzy-search"></a>Indexation pour la recherche approximative

Les analyseurs ne sont pas utilisés lors du traitement des requêtes pour créer un graphe d’expansions, mais cela ne signifie pas qu’ils doivent être ignorés dans les scénarios de recherche approximative. Après tout, les analyseurs sont utilisés pendant l’indexation pour créer les jetons vis-à-vis desquels la correspondance est effectuée, que la requête soit une forme libre, une recherche filtrée ou une recherche approximative avec un graphe comme entrée. 

En général, quand vous affectez des analyseurs en fonction du champ, la décision d’affiner la chaîne d’analyse est basée sur le cas d’usage principal (un filtre ou une recherche en texte intégral) plutôt que sur des formes de requête spécialisées telles qu’une recherche approximative. Ainsi n’existe-t-il pas de recommandation d’analyseur spécifique pour la recherche approximative. 

Toutefois, si les requêtes de test ne produisent pas les correspondances attendues, vous pouvez essayer de faire varier l’analyseur d’indexation, en le définissant sur un [analyseur linguistique](index-add-language-analyzers.md), afin de voir si vous obtenez de meilleurs résultats. Certaines langues, en particulier celles qui ont des mutations de voyelle, peuvent tirer parti des formes fléchies et irrégulières générées par les processeurs de langage naturel Microsoft. Dans certains cas, l’utilisation de l’analyseur linguistique approprié peut s’avérer déterminante pour qu’un terme soit représenté sous forme de jetons d’une manière compatible avec la valeur fournie par l’utilisateur.

## <a name="how-to-use-fuzzy-search"></a>Comment utiliser la recherche approximative

Les requêtes approximatives sont construites à l’aide de la syntaxe de requête Lucene complète, par le biais de l’appel de l’[analyseur de requêtes Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html).

1. Définissez l’analyseur Lucene complet sur la requête (`queryType=full`).

1. Si vous le souhaitez, étendez la requête à des champs spécifiques, à l’aide de ce paramètre (`searchFields=<field1,field2>`). 

1. Ajoutez l’opérateur tilde (`~`) à la fin du terme entier (`search=<string>~`).

   Incluez un paramètre facultatif, un nombre compris entre 0 et 2 (valeur par défaut), si vous souhaitez spécifier la distance de modification (`~1`). Par exemple, « blue~ » ou « blue~1 » retournent « blue », « blues » et « glue ».

Dans Recherche cognitive Azure, en plus du terme et de la distance (maximum 2), il n’y a aucun paramètre supplémentaire à définir sur la requête.

> [!NOTE]
> Pendant leur traitement, les requêtes approximatives ne sont pas soumises à une [analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis). L’entrée de la requête est ajoutée directement à l’arborescence de requête et développée pour créer un graphe de termes. La seule transformation effectuée est la mise en minuscules.

## <a name="testing-fuzzy-search"></a>Test de la recherche approximative

Pour les tests simples, nous vous recommandons d’utiliser l’[Explorateur de recherche](search-explorer.md) ou [Postman](search-get-started-rest.md) afin d’effectuer une itération au sein d’une expression de requête. Les deux outils sont interactifs, ce qui signifie que vous pouvez rapidement parcourir plusieurs variantes d’un terme et évaluer les réponses retournées.

Quand les résultats sont ambigus, la [mise en surbrillance des correspondances](search-pagination-page-layout.md#hit-highlighting) peut vous aider à identifier la correspondance dans la réponse. 

> [!Note]
> L’utilisation de la mise en surbrillance des correspondances pour identifier les correspondances approximatives a des limites et fonctionne uniquement pour la recherche approximative de base. Si votre index a des profils de scoring, ou si vous enrichissez la requête avec une syntaxe supplémentaire, la mise en surbrillance des correspondances risque d’échouer pour identifier la correspondance. 

### <a name="example-1-fuzzy-search-with-the-exact-term"></a>Exemple 1 : Recherche approximative avec le terme exact

Supposons qu’un champ `"Description"` d’un document de recherche contient la chaîne suivante : `"Test queries with special characters, plus strings for MSFT, SQL and Java."`

Commencez par une recherche approximative sur « special » et ajoutez la mise en surbrillance des correspondances au champ Description :

```console
search=special~&highlight=Description
```

Dans la réponse, étant donné que vous avez ajouté la mise en surbrillance des correspondances, la mise en forme est appliquée à « special » en tant que terme correspondant.

```output
"@search.highlights": {
    "Description": [
        "Test queries with <em>special</em> characters, plus strings for MSFT, SQL and Java."
    ]
```

Renouvelez la demande en retirant quelques lettres (« pe ») de « special » afin qu’il soit mal orthographié :

```console
search=scial~&highlight=Description
```

Jusqu’à présent, aucune modification n’est apportée à la réponse. À l’aide de la distance par défaut de 2 degrés, la suppression de deux caractères (« pe ») de « special » permet toujours une correspondance correcte sur ce terme.

```output
"@search.highlights": {
    "Description": [
        "Test queries with <em>special</em> characters, plus strings for MSFT, SQL and Java."
    ]
```

Pour effectuer une demande supplémentaire, modifiez davantage le terme de recherche en retirant un dernier caractère, soit un total de trois suppressions (permettant de passer de « special » à « scal ») :

```console
search=scal~&highlight=Description
```

Notez que la même réponse est retournée, mais à présent la correspondance approximative ne porte pas sur « special », mais sur « SQL ».

```output
        "@search.score": 0.4232868,
        "@search.highlights": {
            "Description": [
                "Mix of special characters, plus strings for MSFT, <em>SQL</em>, 2019, Linux, Java."
            ]
```

L’intérêt de cet exemple développé est d’illustrer la clarté que la mise en surbrillance des correspondances peut apporter à des résultats ambigus. Dans tous les cas, le même document est retourné. Si vous aviez utilisé des ID de document pour vérifier une correspondance, vous auriez peut-être manqué le passage de « special » à « SQL ».

## <a name="see-also"></a>Voir aussi

+ [Fonctionnement de la recherche en texte intégral dans Recherche cognitive Azure (architecture d’analyse de requête)](search-lucene-query-architecture.md)
+ [Navigateur de recherche](search-explorer.md)
+ [Guide pratique pour interroger dans .NET](./search-get-started-dotnet.md)
+ [Guide pratique pour interroger dans REST](./search-get-started-powershell.md)