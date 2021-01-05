---
title: Ajouter la saisie semi-automatique dans une zone de recherche
titleSuffix: Azure Cognitive Search
description: Activez les actions de recherche en cours de frappe dans la Recherche cognitive Azure en créant des suggesteurs et en formulant des requêtes qui complètent automatiquement une zone de recherche avec des termes ou des expressions complets. Vous pouvez également retourner des correspondances suggérées.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/24/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 25c87971455ed3c5f59c92748794720d61e599e3
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96339606"
---
# <a name="add-autocomplete-and-suggestions-to-client-apps-using-azure-cognitive-search"></a>Ajouter l’autocomplétion et les suggestions aux applications clientes à l’aide de Recherche cognitive Azure

La recherche en cours de frappe est une technique courante pour améliorer la productivité des requêtes initiées par l’utilisateur. Dans Recherche cognitive Azure, cette expérience est prise en charge via l’*autocomplétion*, qui termine un terme ou une expression en fonction d’une entrée partielle (en complétant « micro » par « microsoft » par exemple). Les *suggestions* constituent une autre expérience utilisateur : il s’agit d’une liste succincte de documents correspondants (retournant des titres de livres avec un ID pour pouvoir lier vers une page d’informations sur ce livre). L’autocomplétion et les suggestions sont basées sur une correspondance dans l’index. Le service ne propose pas de requêtes qui ne retournent aucun résultat.

Pour implémenter ces expériences dans Recherche cognitive Azure, vous avez besoin des éléments suivants :

+ Une définition *suggesteur* qui est incorporée dans le schéma d’index.
+ Une *requête* spécifiant l’API [Autocomplétion](/rest/api/searchservice/autocomplete) ou [Suggestions](/rest/api/searchservice/suggestions) sur la demande.
+ Un *contrôle d’interface utilisateur* pour gérer les interactions de type recherche en cours de frappe dans votre application cliente. Nous vous recommandons d’utiliser une bibliothèque JavaScript existante à cet effet.

Dans Recherche cognitive Azure, les requêtes avec autocomplétion et les résultats suggérés sont récupérés à partir de l’index de recherche, à partir des champs sélectionnés que vous avez enregistrés avec un suggesteur. Un suggesteur fait partie de l’index. Il spécifie les champs qui fournissent du contenu qui complètent une requête et/ou suggère un résultat. Quand l’index est créé et chargé, une structure de données de suggesteur est créée en interne pour stocker les préfixes utilisés pour la correspondance sur des requêtes partielles. Pour les suggestions, le choix de champs appropriés qui sont uniques, ou au moins non répétitifs, est essentiel à l’expérience. Pour plus d’informations, consultez [Créer un suggesteur](index-add-suggesters.md).

Le reste de cet article se concentre sur les requêtes et le code client. Il utilise JavaScript et C# pour illustrer des points clés. Les exemples d’API REST sont utilisés pour présenter chaque opération de façon concise. Pour obtenir des liens vers des exemples de code de bout en bout, consultez [Étapes suivantes](#next-steps).

## <a name="set-up-a-request"></a>Configurer une demande

Les éléments d’une demande incluent l’une des API de recherche en cours de frappe, une requête partielle et un suggesteur. Le script suivant illustre les composants d’une demande, à l’aide de l’API REST d’autocomplétion.

```http
POST /indexes/myxboxgames/docs/autocomplete?search&api-version=2020-06-30
{
  "search": "minecraf",
  "suggesterName": "sg"
}
```

Le **suggesterName** vous donne les champs de suggesteur utilisés pour compléter les termes ou les suggestions. Pour les suggestions en particulier, la liste de champs doit être composée de ceux qui offrent des choix clairs parmi les résultats de correspondance. Sur un site qui vend des jeux informatiques, le champ peut être le titre du jeu.

Le paramètre **search** fournit la requête partielle, où les caractères sont transmis à la demande de requête via le contrôle Autocomplete dans jQuery. Dans l’exemple ci-dessus, « minecraf » est une illustration statique de ce que le contrôle peut transmettre.

Les API n’imposent pas d’exigences de longueur minimale sur la requête partielle qui peut inclure un seul caractère. Toutefois, le contrôle Autocomplete de jQuery fournit une longueur minimale. Un minimum de deux ou trois caractères est normal.

Les correspondances se trouvent au début d’un terme n’importe où dans la chaîne d’entrée. Prenons comme exemple « The Quick Brown Fox » : l’autocomplétion et les suggestions correspondent à des versions partielles de « The », « Quick », « Brown » ou « Fox », mais pas à des termes partiels tels que « rown » ou « ox ». En outre, chaque correspondance définit l’étendue des expansions en aval. Une requête partielle « Quick br » est une correspondance de « Quick Brown » ou « Quick Bread », mais ni « Brown » ni « Bread » ne sont des correspondances, sauf si « Quick » les précède.

### <a name="apis-for-search-as-you-type"></a>API pour la recherche en cours de frappe

Cliquez sur les liens suivants pour accéder aux pages de référence des SDK REST et .NET :

+ [REST API Suggestions](/rest/api/searchservice/suggestions) 
+ [API REST Autocomplétion](/rest/api/searchservice/autocomplete) 
+ [Méthode SuggestAsync](/dotnet/api/azure.search.documents.searchclient.suggestasync)
+ [Méthode AutocompleteAsync](/dotnet/api/azure.search.documents.searchclient.autocompleteasync)

## <a name="structure-a-response"></a>Structurer une réponse

Les réponses de l’autocomplétion et des suggestions sont celles que vous pouvez attendre pour le modèle : L’[autocomplétion](/rest/api/searchservice/autocomplete#response) retourne une liste de termes et les [suggestions](/rest/api/searchservice/suggestions#response) retournent des termes plus un ID de document pour vous permettre de récupérer le document (utilisez l’API [Document de recherche](/rest/api/searchservice/lookup-document) pour récupérer le document spécifique pour une page d’informations).

Les réponses sont mises en forme par les paramètres que vous incluez dans la demande. Pour l’autocomplétion, définissez [**autocompleteMode**](/rest/api/searchservice/autocomplete#autocomplete-modes) pour déterminer si la complétion du texte se produit sur un ou deux termes. Pour les suggestions, le champ que vous choisissez détermine le contenu de la réponse.

Vous devez affiner la réponse pour éviter les doublons ou les résultats non liés. Pour contrôler les résultats, incluez davantage de paramètres dans la demande. Les paramètres suivants s’appliquent à la fois à l’autocomplétion et aux suggestions. Ils sont toutefois peut-être plus utiles pour les suggestions, en particulier lorsqu’un suggesteur inclut plusieurs champs.

| Paramètre | Usage |
|-----------|-------|
| **$select** | Si vous avez plusieurs champs source (**sourceFields**) dans un suggesteur, utilisez **$select** pour choisir le champ qui contribue aux valeurs (`$select=GameTitle`). |
| **searchFields** | Limitez la requête à des champs spécifiques. |
| **$filter** | Appliquez des critères de correspondance sur le jeu de résultats (`$filter=Category eq 'ActionAdventure'`). |
| **$top** | Limitez les résultats à un nombre spécifique (`$top=5`).|

## <a name="add-user-interaction-code"></a>Ajouter un code d’interaction utilisateur

Le remplissage automatique d’un terme de requête ou l’affichage d’une liste de liens correspondants nécessite un code d’interaction utilisateur, en général JavaScript, qui peut consommer des demandes provenant de sources externes, telles que les requêtes d’autocomplétion ou de suggestion sur un index Recherche cognitive Azure.

Bien que vous puissiez écrire ce code en mode natif, il est beaucoup plus facile d’utiliser les fonctions d’une bibliothèque JavaScript existante. Cet article en présente deux : l’une pour les suggestions et l’autre pour l’autocomplétion. 

+ [Le widget d’autocomplétion (jQuery UI)](https://jqueryui.com/autocomplete/) est utilisé dans l’exemple de suggestion. Vous pouvez créer une zone de recherche, puis la référencer dans une fonction JavaScript qui utilise le widget d’autocomplétion. Les propriétés du widget définissent la source (fonction d’autocomplétion ou de suggestions), la longueur minimale des caractères d’entrée avant que l’action soit effectuée et le positionnement.

+ Le [plug-in d’autocomplétion fourni par XDSoft](https://xdsoft.net/jqplugins/autocomplete/) est utilisé dans l’exemple d’autocomplétion.

Nous utilisons ces bibliothèques pour créer la zone de recherche prenant en charge les suggestions et l’autocomplétion. Les entrées collectées dans la zone de recherche sont associées à des suggestions et des actions d’autocomplétion.

## <a name="suggestions"></a>Suggestions

Cette section vous guide tout au long de l’implémentation des résultats suggérés, en commençant par la définition de la zone de recherche. Elle montre également un script qui appelle la première bibliothèque d’autocomplétion JavaScript référencée dans cet article.

### <a name="create-a-search-box"></a>Créer une zone de recherche

En prenant l’exemple de la [bibliothèque d’autocomplétion jQuery UI](https://jqueryui.com/autocomplete/) et d’un projet MVC en C#, vous pouvez définir la zone de recherche à l’aide de JavaScript dans le fichier **Index.cshtml**. La bibliothèque ajoute l’interaction de recherche en cours de frappe en effectuant des appels asynchrones au contrôleur MVC afin de récupérer les suggestions.

Dans **Index.cshtml** sous le dossier \Views\Home, une ligne pour créer une zone de recherche peut se présenter comme suit :

```html
<input class="searchBox" type="text" id="searchbox1" placeholder="search">
```

Cet exemple est une simple zone de texte avec une classe pour le style, un ID qui doit être référencé par JavaScript et un texte d’espace réservé.  

Dans le même fichier, incorporez le code JavaScript qui fait référence à la zone de recherche. La fonction suivante appelle l’API Suggest, qui demande les documents correspondants suggérés en fonction des entrées de terme partielles :

```javascript
$(function () {
    $("#searchbox1").autocomplete({
        source: "/home/suggest?highlights=false&fuzzy=false&",
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

`source` indique à la fonction d’autocomplétion de jQuery UI où récupérer la liste des éléments à afficher sous la zone de recherche. Dans la mesure où ce projet est un projet MVC, il appelle la fonction **Suggest** dans le fichier **HomeController.cs** qui contient la logique permettant de retourner les suggestions de requête. Cette fonction transmet également quelques paramètres pour contrôler la mise en surbrillance, les correspondances approximatives et les termes. L’API JavaScript d’autocomplétion ajoute le paramètre de terme.

`minLength: 3` garantit que les recommandations ne sont affichées que lorsque plus de trois caractères sont saisis dans la zone de recherche.

### <a name="enable-fuzzy-matching"></a>Activer la correspondance approximative

La recherche approximative vous permet d’obtenir des résultats selon des correspondances proches, même si l’utilisateur a mal épelé un mot dans la zone de recherche. La distance d’édition est 1, ce qui signifie qu’il peut y avoir un écart maximal de 1 caractère entre l’entrée de l’utilisateur et une correspondance. 

```javascript
source: "/home/suggest?highlights=false&fuzzy=true&",
```

### <a name="enable-highlighting"></a>Activer la mise en surbrillance

La mise en surbrillance applique le style de police aux caractères du résultat qui correspondent à l’entrée de l’utilisateur. Par exemple, si l’entrée partielle est « micro », le résultat est le suivant : **micro** soft, **micro** scope, et ainsi de suite. La mise en surbrillance est basée sur les paramètres HighlightPreTag et HighlightPostTag, définis en ligne avec la fonction Suggestion.

```javascript
source: "/home/suggest?highlights=true&fuzzy=true&",
```

### <a name="suggest-function"></a>Fonction Suggest

Si vous utilisez C# et une application MVC, le **fichier HomeController.cs** sous le répertoire Controllers est l’endroit où vous pouvez créer une classe pour les résultats suggérés. Dans .NET, une fonction Suggest est basée sur la [méthode SuggestAsync](/dotnet/api/azure.search.documents.searchclient.suggestasync). Pour plus d’informations sur le SDK .NET, consultez [Guide pratique pour utiliser la Recherche cognitive Azure à partir d’une application .NET](search-howto-dotnet-sdk.md).

La méthode `InitSearch` crée un client d’index HTTP authentifié dans le service Recherche cognitive Azure. Les propriétés de la classe [SuggestOptions](/dotnet/api/azure.search.documents.suggestoptions) déterminent les champs recherchés et retournés dans les résultats, le nombre de correspondances et l’utilisation de la correspondance approximative. 

Pour l’autocomplétion, la correspondance approximative est limitée à une distance d’une modification (un caractère omis ou mal placé). Notez que la correspondance approximative dans les requêtes avec autocomplétion peut parfois produire des résultats inattendus en fonction de la taille de l’index et de la façon dont il est partitionnée. Pour plus d’informations, consultez les [concepts de partition et de partitionnement](search-capacity-planning.md#concepts-search-units-replicas-partitions-shards).

```csharp
public async Task<ActionResult> SuggestAsync(bool highlights, bool fuzzy, string term)
{
    InitSearch();

    var options = new SuggestOptions()
    {
        UseFuzzyMatching = fuzzy,
        Size = 8,
    };

    if (highlights)
    {
        options.HighlightPreTag = "<b>";
        options.HighlightPostTag = "</b>";
    }

    // Only one suggester can be specified per index.
    // The suggester for the Hotels index enables autocomplete/suggestions on the HotelName field only.
    // During indexing, HotelNames are indexed in patterns that support autocomplete and suggested results.
    var suggestResult = await _searchClient.SuggestAsync<Hotel>(term, "sg", options).ConfigureAwait(false);

    // Convert the suggest query results to a list that can be displayed in the client.
    List<string> suggestions = suggestResult.Value.Results.Select(x => x.Text).ToList();

    // Return the list of suggestions.
    return new JsonResult(suggestions);
}
```

La fonction SuggestAsync prend deux paramètres qui déterminent si les meilleurs résultats sont renvoyés ou si la correspondance approximative est utilisée en plus de la recherche de terme. Vous pouvez inclure jusqu’à huit correspondances dans les résultats suggérés. La méthode crée un [objet SuggestOptions](/dotnet/api/azure.search.documents.suggestoptions), qui est ensuite passé à l’API de suggestion. Le résultat est ensuite converti en JSON pour être visible par le client.

## <a name="autocomplete"></a>Autocomplétion

Jusqu’à présent, le code de l’expérience utilisateur de recherche a été centré sur les suggestions. Le bloc de code suivant montre l’autocomplétion, en utilisant la fonction d’autocomplétion XDSoft jQuery UI, en passant une requête d’autocomplétion pour Recherche cognitive Azure. Comme avec les suggestions, dans une application C#, le code qui prend en charge l’interaction utilisateur se trouve dans **index.cshtml**.

```javascript
$(function () {
    // using modified jQuery Autocomplete plugin v1.2.8 https://xdsoft.net/jqplugins/autocomplete/
    // $.autocomplete -> $.autocompleteInline
    $("#searchbox1").autocompleteInline({
        appendMethod: "replace",
        source: [
            function (text, add) {
                if (!text) {
                    return;
                }

                $.getJSON("/home/autocomplete?term=" + text, function (data) {
                    if (data && data.length > 0) {
                        currentSuggestion2 = data[0];
                        add(data);
                    }
                });
            }
        ]
    });

    // complete on TAB and clear on ESC
    $("#searchbox1").keydown(function (evt) {
        if (evt.keyCode === 9 /* TAB */ && currentSuggestion2) {
            $("#searchbox1").val(currentSuggestion2);
            return false;
        } else if (evt.keyCode === 27 /* ESC */) {
            currentSuggestion2 = "";
            $("#searchbox1").val("");
        }
    });
});
```

### <a name="autocomplete-function"></a>Fonction d’autocomplétion

La fonction Autocomplétion est basée sur la [méthode AutocompleteAsync](/dotnet/api/azure.search.documents.searchclient.autocompleteasync). Comme avec les suggestions, ce bloc de code se trouve dans le fichier **HomeController.cs**.

```csharp
public async Task<ActionResult> AutoCompleteAsync(string term)
{
    InitSearch();

    // Setup the autocomplete parameters.
    var ap = new AutocompleteOptions()
    {
        Mode = AutocompleteMode.OneTermWithContext,
        Size = 6
    };
    var autocompleteResult = await _searchClient.AutocompleteAsync(term, "sg", ap).ConfigureAwait(false);

    // Convert the autocompleteResult results to a list that can be displayed in the client.
    List<string> autocomplete = autocompleteResult.Value.Results.Select(x => x.Text).ToList();

    return new JsonResult(autocomplete);
}
```

La fonction d’autocomplétion prend l’entrée du terme de recherche. La méthode crée un [objet AutoCompleteParameters](/rest/api/searchservice/autocomplete). Le résultat est ensuite converti en JSON pour être visible par le client.

## <a name="next-steps"></a>Étapes suivantes

Suivez ces liens pour obtenir des instructions de bout en bout ou du code illustrant les deux expériences de recherche en cours de frappe. Les deux exemples de code incluent des implémentations hybrides des suggestions et de l’autocomplétion.

+ [Tutoriel : Créer votre première application en C# (leçon 3)](tutorial-csharp-type-ahead-and-suggestions.md)
+ [Exemple de code C# : azure-search-dotnet-samples/create-first-app/3-add-typeahead/](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10/3-add-typeahead)