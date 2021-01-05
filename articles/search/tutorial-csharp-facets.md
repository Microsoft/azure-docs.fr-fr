---
title: Tutoriel C# sur l’utilisation de facettes pour faciliter la navigation
titleSuffix: Azure Cognitive Search
description: Continuez à partir de « Résultats de la pagination » pour ajouter la navigation à facettes. Découvrez dans quelle mesure les facettes permettent d’affiner facilement une recherche.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 10/01/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ab15af07c5f63d375d8fdb4fc38e0853e207a0be
ms.sourcegitcommit: 67e8e1caa8427c1d78f6426c70bf8339a8b4e01d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91667280"
---
# <a name="tutorial-add-faceted-navigation-using-the-net-sdk"></a>Tutoriel : Ajouter la navigation à facettes à l’aide du SDK .NET

Les facettes permettent une navigation autonome en fournissant un ensemble de liens pour filtrer les résultats. Dans ce tutoriel, une structure de navigation par facettes est placée sur le côté gauche de la page, avec des étiquettes et du texte cliquable pour affiner les résultats.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Définir des propriétés de modèle en tant que _IsFacetable_
> * Ajouter la navigation à facettes à votre application

## <a name="overview"></a>Vue d’ensemble

Les facettes sont basées sur les champs de votre index de recherche. Une demande de requête qui inclut facet=[chaîne] fournit le champ devant servir de facette. Il est courant d’inclure plusieurs facettes, comme `&facet=category&facet=amenities`, chaque facette étant séparée par un caractère d’esperluette (&). Pour implémenter une structure de navigation par facettes, vous devez spécifier des facettes et des filtres. Le filtre est utilisé sur un événement Click pour affiner les résultats. Par exemple, si vous cliquez sur « budget », les résultats sont filtrés en fonction de ce critère.

Ce tutoriel étend le projet de pagination créé dans le tutoriel [Ajouter la pagination aux résultats de recherche](tutorial-csharp-paging.md).

Une version terminée du code de ce tutoriel se trouve dans le projet suivant :

* [4-add-facet-navigation (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/4-add-facet-navigation)

## <a name="prerequisites"></a>Prérequis

* Solution [2a-add-paging (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/2a-add-paging). Ce projet peut être votre propre version reposant sur le tutoriel précédent ou une copie issue de GitHub.

Ce tutoriel a été mis à jour pour utiliser le package [Azure.Search.Documents (version 11)](https://www.nuget.org/packages/Azure.Search.Documents/). Pour obtenir une version antérieure du SDK .NET, consultez [Exemple de code Microsoft.Azure.Search (version 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="set-model-properties-as-isfacetable"></a>Définir les propriétés de modèle en tant que IsFacetable

Pour qu’une propriété de modèle puisse être trouvée dans une recherche à facettes, elle doit être étiquetée avec **IsFacetable**.

1. Examinez la classe **Hotel**. **Category** et **Tags**, par exemple, sont marqués comme **IsFacetable**, contrairement à **HotelName** et **Description**. 

    ```cs
    public partial class Hotel
    {
        [SimpleField(IsFilterable = true, IsKey = true)]
        public string HotelId { get; set; }

        [SearchableField(IsSortable = true)]
        public string HotelName { get; set; }

        [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnLucene)]
        public string Description { get; set; }

        [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrLucene)]
        [JsonPropertyName("Description_fr")]
        public string DescriptionFr { get; set; }

        [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public string Category { get; set; }

        [SearchableField(IsFilterable = true, IsFacetable = true)]
        public string[] Tags { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public bool? ParkingIncluded { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public DateTimeOffset? LastRenovationDate { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
        public double? Rating { get; set; }

        public Address Address { get; set; }

        [SimpleField(IsFilterable = true, IsSortable = true)]
        public GeographyPoint Location { get; set; }

        public Room[] Rooms { get; set; }
    }
    ```

1. Comme nous n’allons pas changer d’étiquettes dans le cadre de ce tutoriel, fermez le fichier hotel.cs sans le modifier.

    > [!Note]
    > Une recherche à facettes génère une erreur si un champ demandé dans la recherche n’est pas correctement étiqueté.

## <a name="add-facet-navigation-to-your-app"></a>Ajouter la navigation à facettes à votre application

Pour cet exemple, nous allons permettre à l’utilisateur de sélectionner une catégorie d’hôtels, ou une commodité, dans des listes de liens affichés à gauche des résultats. L’utilisateur commence par entrer du texte de recherche, puis affine progressivement les résultats de la recherche en sélectionnant une catégorie ou une commodité.

Il appartient au contrôleur de passer les listes de facettes à la vue. Pour conserver les sélections de l’utilisateur à mesure que la recherche progresse, nous utilisons un stockage temporaire comme mécanisme de conservation de l’état.

![Utilisation de la navigation à facettes pour affiner une recherche portant sur le terme « pool »](./media/tutorial-csharp-create-first-app/azure-search-facet-nav.png)

### <a name="add-filter-strings-to-the-searchdata-model"></a>Ajouter des chaînes de filtre au modèle SearchData

1. Ouvrez le fichier SearchData.cs et ajoutez des propriétés de chaîne à la classe **SearchData** destinées à contenir les chaînes de filtre des facettes.

    ```cs
    public string categoryFilter { get; set; }
    public string amenityFilter { get; set; }
    ```

### <a name="add-the-facet-action-method"></a>Ajouter la méthode d’action Facet

Le contrôleur Home a besoin d’une nouvelle action, **Facet**, et de mises à jour pour ses actions **Index** et **Page** existantes et pour la méthode **RunQueryAsync**.

<!-- 1. Open the home controller file, and add the **using** statement, to enable the **List&lt;string&gt;** construct.

    ```cs
    using System.Collections.Generic; 
    ```-->

1. Remplacez la méthode d’action **Index(SearchData model)** .

    ```cs
    public async Task<ActionResult> Index(SearchData model)
    {
        try
        {
            // Ensure the search string is valid.
            if (model.searchText == null)
            {
                model.searchText = "";
            }

            // Make the search call for the first page.
            await RunQueryAsync(model, 0, 0, "", "").ConfigureAwait(false);
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "1" });
        }

        return View(model);
    }
    ```

1. Remplacez la méthode d’action **PageAsync(SearchData model)** .

    ```cs
    public async Task<ActionResult> PageAsync(SearchData model)
    {
        try
        {
            int page;

            // Calculate the page that should be displayed.
            switch (model.paging)
            {
                case "prev":
                    page = (int)TempData["page"] - 1;
                    break;

                case "next":
                    page = (int)TempData["page"] + 1;
                    break;

                default:
                    page = int.Parse(model.paging);
                    break;
            }

            // Recover the leftMostPage.
            int leftMostPage = (int)TempData["leftMostPage"];

            // Recover the filters.
            string catFilter = TempData["categoryFilter"].ToString();
            string ameFilter = TempData["amenityFilter"].ToString();

            // Recover the search text.
            model.searchText = TempData["searchfor"].ToString();

            // Search for the new page.
            await RunQueryAsync(model, page, leftMostPage, catFilter, ameFilter);
        }

        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "2" });
        }
        return View("Index", model);
    }
    ```

1. Ajoutez un méthode d’action **FacetAsync(SearchData model)** à activer quand l’utilisateur clique sur un lien à facette. Le modèle contient un filtre de recherche de catégorie ou de commodité. Ajoutez-le après l’action **PageAsync**.

    ```cs
    public async Task<ActionResult> FacetAsync(SearchData model)
    {
        try
        {
            // Filters set by the model override those stored in temporary data.
            string catFilter;
            string ameFilter;
            if (model.categoryFilter != null)
            {
                catFilter = model.categoryFilter;
            } else
            {
                catFilter = TempData["categoryFilter"].ToString();
            }

            if (model.amenityFilter != null)
            {
                ameFilter = model.amenityFilter;
            } else
            {
                ameFilter = TempData["amenityFilter"].ToString();
            }

            // Recover the search text.
            model.searchText = TempData["searchfor"].ToString();

            // Initiate a new search.
            await RunQueryAsync(model, 0, 0, catFilter, ameFilter).ConfigureAwait(false);
        }
        catch
        {
            return View("Error", new ErrorViewModel { RequestId = "2" });
        }

        return View("Index", model);
    }
    ```

### <a name="set-up-the-search-filter"></a>Configurer le filtre de recherche

Quand un utilisateur sélectionne une facette, il peut, par exemple, cliquer sur la catégorie **Resort and Spa** ; dans ce cas, seuls des hôtels spécifiés comme appartenant à cette catégorie doivent être retournés dans les résultats. Pour affiner une recherche de cette façon, nous devons configurer un _filtre_.

1. Remplacez la méthode **RunQueryAsync** par le code suivant. En gros, elle prend une chaîne de filtre de catégories et une chaîne de filtre de commodités et définit le paramètre **Filter** de **SearchOptions**.

    ```cs
    private async Task<ActionResult> RunQueryAsync(SearchData model, int page, int leftMostPage, string catFilter, string ameFilter)
    {
        InitSearch();

        string facetFilter = "";

        if (catFilter.Length > 0 && ameFilter.Length > 0)
        {
            // Both facets apply.
            facetFilter = $"{catFilter} and {ameFilter}"; 
        } else
        {
            // One, or zero, facets apply.
            facetFilter = $"{catFilter}{ameFilter}";
        }

        var options = new SearchOptions
        {
            Filter = facetFilter,

            SearchMode = SearchMode.All,

            // Skip past results that have already been returned.
            Skip = page * GlobalVariables.ResultsPerPage,

            // Take only the next page worth of results.
            Size = GlobalVariables.ResultsPerPage,

            // Include the total number of results.
            IncludeTotalCount = true,
        };

        // Return information on the text, and number, of facets in the data.
        options.Facets.Add("Category,count:20");
        options.Facets.Add("Tags,count:20");

        // Enter Hotel property names into this list, so only these values will be returned.
        options.Select.Add("HotelName");
        options.Select.Add("Description");
        options.Select.Add("Category");
        options.Select.Add("Tags");

        // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
        model.resultList = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);

        // This variable communicates the total number of pages to the view.
        model.pageCount = ((int)model.resultList.TotalCount + GlobalVariables.ResultsPerPage - 1) / GlobalVariables.ResultsPerPage;

        // This variable communicates the page number being displayed to the view.
        model.currentPage = page;

        // Calculate the range of page numbers to display.
        if (page == 0)
        {
            leftMostPage = 0;
        }
        else if (page <= leftMostPage)
        {
            // Trigger a switch to a lower page range.
            leftMostPage = Math.Max(page - GlobalVariables.PageRangeDelta, 0);
        }
        else if (page >= leftMostPage + GlobalVariables.MaxPageRange - 1)
        {
            // Trigger a switch to a higher page range.
            leftMostPage = Math.Min(page - GlobalVariables.PageRangeDelta, model.pageCount - GlobalVariables.MaxPageRange);
        }
        model.leftMostPage = leftMostPage;

        // Calculate the number of page numbers to display.
        model.pageRange = Math.Min(model.pageCount - leftMostPage, GlobalVariables.MaxPageRange);

        // Ensure Temp data is stored for the next call.
        TempData["page"] = page;
        TempData["leftMostPage"] = model.leftMostPage;
        TempData["searchfor"] = model.searchText;
        TempData["categoryFilter"] = catFilter;
        TempData["amenityFilter"] = ameFilter;

        // Return the new view.
        return View("Index", model);
    }
    ```

    Notez que les propriétés **Category** et **Tags** sont ajoutées à la liste des éléments **Select** à retourner. Cet ajout n’est pas obligatoire pour que la navigation par facettes fonctionne, mais nous utilisons ces informations pour vérifier que les filtres fonctionnent correctement.

### <a name="add-lists-of-facet-links-to-the-view"></a>Ajouter des listes de liens à facette à la vue

La vue nécessite certaines modifications significatives. 

1. Commencez par ouvrir le fichier hotels.css (dans le dossier wwwroot/css) et ajoutez les classes suivantes.

    ```html
    .facetlist {
        list-style: none;
    }

    .facetchecks {
        width: 250px;
        display: normal;
        color: #666;
        margin: 10px;
        padding: 5px;
    }

    .facetheader {
        font-size: 10pt;
        font-weight: bold;
        color: darkgreen;
    }
    ```

1. Pour la vue, organisez la sortie sous la forme d’un tableau, afin d’aligner correctement les listes de facettes sur la gauche et les résultats sur la droite. Ouvrez le fichier index.cshtml. Remplacez tout le contenu des balises HTML &lt;body&gt; par le code suivant.

    ```html
    <body>
        @using (Html.BeginForm("Index", "Home", FormMethod.Post))
        {
            <table>
                <tr>
                    <td></td>
                    <td>
                        <h1 class="sampleTitle">
                            <img src="~/images/azure-logo.png" width="80" />
                            Hotels Search - Facet Navigation
                        </h1>
                    </td>
                </tr>

                <tr>
                    <td></td>
                    <td>
                        <!-- Display the search text box, with the search icon to the right of it.-->
                        <div class="searchBoxForm">
                            @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input value="" class="searchBoxSubmit" type="submit">
                        </div>
                    </td>
                </tr>

                <tr>
                    <td valign="top">
                        <div id="facetplace" class="facetchecks">

                            @if (Model != null && Model.resultList != null)
                            {
                                List<string> categories = Model.resultList.Facets["Category"].Select(x => x.Value.ToString()).ToList();

                                if (categories.Count > 0)
                                {
                                    <h5 class="facetheader">Category:</h5>
                                    <ul class="facetlist">
                                        @for (var c = 0; c < categories.Count; c++)
                                        {
                                            var facetLink = $"{categories[c]} ({Model.resultList.Facets["Category"][c].Count})";
                                            <li>
                                                @Html.ActionLink(facetLink, "FacetAsync", "Home", new { categoryFilter = $"Category eq '{categories[c]}'" }, null)
                                            </li>
                                        }
                                    </ul>
                                }

                                List<string> tags = Model.resultList.Facets["Tags"].Select(x => x.Value.ToString()).ToList();

                                if (tags.Count > 0)
                                {
                                    <h5 class="facetheader">Amenities:</h5>
                                    <ul class="facetlist">
                                        @for (var c = 0; c < tags.Count; c++)
                                        {
                                            var facetLink = $"{tags[c]} ({Model.resultList.Facets["Tags"][c].Count})";
                                            <li>
                                                @Html.ActionLink(facetLink, "FacetAsync", "Home", new { amenityFilter = $"Tags/any(t: t eq '{tags[c]}')" }, null)
                                            </li>
                                        }
                                    </ul>
                                }
                            }
                        </div>
                    </td>
                    <td valign="top">
                        <div id="resultsplace">
                            @if (Model != null && Model.resultList != null)
                            {
                                // Show the result count.
                                <p class="sampleText">
                                    @Model.resultList.TotalCount Results
                                </p>

                                var results = Model.resultList.GetResults().ToList();

                                @for (var i = 0; i < results.Count; i++)
                                {
                                    string amenities = string.Join(", ", results[i].Document.Tags);

                                    string fullDescription = results[i].Document.Description;
                                    fullDescription += $"\nCategory: {results[i].Document.Category}";
                                    fullDescription += $"\nAmenities: {amenities}";


                                    // Display the hotel name and description.
                                    @Html.TextAreaFor(m => results[i].Document.HotelName, new { @class = "box1" })
                                    @Html.TextArea($"desc{i}", fullDescription, new { @class = "box2" })
                                }
                            }
                        </div>
                    </td>
                </tr>

                <tr>
                    <td></td>
                    <td valign="top">
                        @if (Model != null && Model.pageCount > 1)
                        {
                            // If there is more than one page of results, show the paging buttons.
                            <table>
                                <tr>
                                    <td class="tdPage">
                                        @if (Model.currentPage > 0)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink("|<", "PageAsync", "Home", new { paging = "0" }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">|&lt;</p>
                                        }
                                    </td>

                                    <td class="tdPage">
                                        @if (Model.currentPage > 0)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink("<", "PageAsync", "Home", new { paging = "prev" }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">&lt;</p>
                                        }
                                    </td>

                                    @for (var pn = Model.leftMostPage; pn < Model.leftMostPage + Model.pageRange; pn++)
                                    {
                                        <td class="tdPage">
                                            @if (Model.currentPage == pn)
                                            {
                                                // Convert displayed page numbers to 1-based and not 0-based.
                                                <p class="pageSelected">@(pn + 1)</p>
                                            }
                                            else
                                            {
                                                <p class="pageButton">
                                                    @Html.ActionLink((pn + 1).ToString(), "PageAsync", "Home", new { paging = @pn }, null)
                                                </p>
                                            }
                                        </td>
                                    }

                                    <td class="tdPage">
                                        @if (Model.currentPage < Model.pageCount - 1)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink(">", "PageAsync", "Home", new { paging = "next" }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">&gt;</p>
                                        }
                                    </td>

                                    <td class="tdPage">
                                        @if (Model.currentPage < Model.pageCount - 1)
                                        {
                                            <p class="pageButton">
                                                @Html.ActionLink(">|", "PageAsync", "Home", new { paging = Model.pageCount - 1 }, null)
                                            </p>
                                        }
                                        else
                                        {
                                            <p class="pageButtonDisabled">&gt;|</p>
                                        }
                                    </td>
                                </tr>
                            </table>
                        }
                    </td>
                </tr>
            </table>
        }
    </body>
    ```

    Remarquez l’utilisation de l’appel **Html.ActionLink**. Cet appel communique les chaînes de filtre valides au contrôleur quand l’utilisateur clique sur un lien à facette. 

### <a name="run-and-test-the-app"></a>Exécuter et tester l’application

La navigation à facettes permet à l’utilisateur d’affiner les recherches d’un simple clic, comme l’illustre la séquence suivante.

1. Exécutez l’application, puis tapez « airport » comme texte de recherche. Vérifiez que la liste de facettes apparaît clairement à gauche. Ces facettes sont toutes celles qui s’appliquent aux hôtels dont le descriptif comporte le terme « airport », et chacune est accompagnée de son nombre d’occurrences.

    ![Utilisation de la navigation à facettes pour affiner une recherche portant sur le terme « airport »](./media/tutorial-csharp-create-first-app/azure-search-facet-airport.png)

1. Cliquez sur la catégorie **Resort and Spa**. Vérifiez que tous les résultats appartiennent à cette catégorie.

    ![Réduction de la recherche à « Resort and Spa »](./media/tutorial-csharp-create-first-app/azure-search-facet-airport-ras.png)

1. Cliquez sur la commodité **continental breakfast**. Vérifiez que tous les résultats appartiennent toujours à la catégorie « Resort and Spa » et correspondent à la commodité sélectionnée.

    ![Réduction de la recherche à « continental breakfast »](./media/tutorial-csharp-create-first-app/azure-search-facet-airport-ras-cb.png)

1. Essayez de sélectionner une autre catégorie, puis une commodité, et observez la façon dont les résultats sont affinés. Ensuite, faites l’opération inverse, en choisissant d’abord une commodité, puis une catégorie. Envoyez une recherche vide pour réinitialiser la page.

    >[!Note]
    > Quand une sélection est effectuée dans une liste de facettes (par exemple, une catégorie), elle remplace toute sélection antérieure dans la liste des catégories.

## <a name="takeaways"></a>Éléments importants à retenir

Retenez les points importants suivants de ce projet :

* Il est impératif de marquer chaque champ définissable en tant que facette avec la propriété **IsFacetable** afin de l’inclure dans la navigation par facettes.
* Les facettes sont combinées avec des filtres pour réduire les résultats.
* Les facettes sont cumulatives, chaque sélection s’ajoutant à la précédente pour affiner les résultats.

## <a name="next-steps"></a>Étapes suivantes

Dans le tutoriel suivant, nous examinons le classement des résultats. À ce stade, les résultats sont simplement triés dans l’ordre dans lequel ils sont stockés dans la base de données.

> [!div class="nextstepaction"]
> [Tutoriel C# : Classer les résultats - Recherche cognitive Azure](tutorial-csharp-orders.md)
