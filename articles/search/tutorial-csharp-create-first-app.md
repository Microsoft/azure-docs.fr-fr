---
title: Tutoriel C# pour créer votre première application
titleSuffix: Azure Cognitive Search
description: Découvrez comment générer votre première application de recherche en C#, étape par étape. Le tutoriel fournit un lien de téléchargement vers une application opérationnelle sur GitHub et le processus complet pour générer l’application à partir de zéro.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 09/30/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ec98762ac5918437e8fdb8426b54b79b1fb5b222
ms.sourcegitcommit: 541bb46e38ce21829a056da880c1619954678586
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2020
ms.locfileid: "91939719"
---
# <a name="tutorial-create-your-first-search-app-using-the-net-sdk"></a>Tutoriel : Créer votre première application de recherche à l’aide du SDK .NET

Ce tutoriel vous montre comment créer une application web qui interroge un index de recherche et retourne les résultats à partir de ce dernier à l’aide de Recherche cognitive Azure et de Visual Studio.

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Configurer un environnement de développement
> * Modéliser des structures de données
> * Créer une page web pour collecter les entrées de requête et afficher les résultats
> * Définir une méthode de recherche
> * Test de l'application

Vous allez également découvrir à quel point il est simple d’effectuer un appel de recherche. Les instructions essentielles du code que vous allez développer sont encapsulées dans les quelques lignes suivantes.

```csharp
var options = new SearchOptions()
{
    // The Select option specifies fields for the result set
    options.Select.Add("HotelName");
    options.Select.Add("Description");
};

var searchResult = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);
model.resultList = searchResult.Value.GetResults().ToList();
```

Un seul appel interroge l’index et retourne les résultats.

:::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-pool.png" alt-text="Recherche de *pool*" border="true":::

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel utilise un exemple d’index hébergé existant afin que vous puissiez vous concentrer sur la création d’une page de recherche qui collecte une chaîne de requête pour la demande et retourne les résultats. L’index contient des données d’hôtels fictives. Une fois que vous avez une page de base, vous pourrez l’améliorer dans les leçons suivantes pour inclure une pagination, des facettes et une expérience d’auto-complétion.

Une version terminée du code de ce tutoriel se trouve dans le projet suivant :

* [1-basic-search-page (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11/1-basic-search-page)

Ce tutoriel a été mis à jour pour utiliser le package Azure.Search.Documents (version 11). Pour obtenir une version antérieure du SDK .NET, consultez [Exemple de code Microsoft.Azure.Search (version 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v10).

## <a name="prerequisites"></a>Prérequis

Étant donné que vous utilisez un exemple d’index de recherche public hébergé par Microsoft, vous n’avez pas besoin d’un service de recherche ou d’un compte Azure pour ce tutoriel.

* [Visual Studio](https://visualstudio.microsoft.com/)

* [Bibliothèque de client Recherche cognitive Azure (version 11)](https://www.nuget.org/packages/Azure.Search.Documents/)

### <a name="install-and-run-the-project-from-github"></a>Installer et exécuter le projet à partir de GitHub

Si vous souhaitez profiter sans tarder d’une application opérationnelle, suivez les étapes ci-dessous pour télécharger et exécuter le code terminé.

1. Recherchez l’exemple sur GitHub : [Créer votre première application](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/create-first-app/v11).

1. Dans le [dossier racine](https://github.com/Azure-Samples/azure-search-dotnet-samples), sélectionnez **Code**, puis **Clone** ou **Download ZIP** (Télécharger le fichier ZIP) pour créer votre copie locale privée du projet.

1. À l’aide de Visual Studio, accédez à la solution pour la page de recherche de base (« 1-basic-search-page ») et ouvrez-la, puis sélectionnez **Démarrer sans débogage** (ou appuyez sur F5) pour générer et exécuter le programme.

1. Il s’agit d’un index d’hôtels. Tapez donc des mots permettant de rechercher des hôtels (par exemple, « wifi », « view » (vue), « bar », « parking ») et examinez les résultats.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-wifi.png" alt-text="Recherche de *pool*" border="true":::

Nous espérons que ce projet se déroule normalement et que l’application web est opérationnelle. De nombreux composants essentiels pour des recherches plus sophistiquées étant inclus dans cette application, il est judicieux de la suivre et de la recréer étape par étape. Les sections suivantes couvrent ces étapes.

## <a name="set-up-a-development-environment"></a>Configurer un environnement de développement

Pour créer ce projet à partir de zéro et ainsi mieux connaître les concepts de Recherche cognitive Azure, commencez avec un projet Visual Studio.

1. Dans Visual Studio, sélectionnez **Nouveau** > **Projet**, puis **Application web ASP.NET Core**.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-project1.png" alt-text="Recherche de *pool*" border="true":::

1. Donnez un nom au projet, tel que « FirstSearchApp », puis définissez l’emplacement. Sélectionnez **Create** (Créer).

1. Choisissez le modèle de projet **Application web (Model-View-Controller)** .

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-project2.png" alt-text="Recherche de *pool*" border="true":::

1. Installez la bibliothèque de client. Dans **Outils** > **Gestionnaire de package NuGet** > **Gérer les packages NuGet pour la solution...** , sélectionnez **Parcourir**, puis recherchez « azure.search.documents ». Installez **Azure.Search.Documents** (version 11 ou ultérieure), en acceptant les contrats de licence et les dépendances.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png" alt-text="Recherche de *pool*" border="true":::

### <a name="initialize-azure-cognitive-search"></a>Initialiser Recherche cognitive Azure

Pour cet exemple, vous utilisez des données d’hôtels disponibles publiquement. Ces données sont une collection arbitraire de 50 noms d’hôtels fictifs et de descriptions, créée uniquement à des fins de démonstration. Pour accéder à ces données, spécifiez un nom et une clé API.

1. Ouvrez **appsettings.json** et remplacez les lignes par défaut par les nom et clé suivants. La clé d’API indiquée ici n’est pas un exemple de clé ; c’est *exactement* la clé dont vous avez besoin pour accéder aux données d’hôtels. Votre fichier doit maintenant ressembler à ceci.

    ```csharp
    {
        "SearchServiceName": "azs-playground",
        "SearchServiceQueryApiKey": "EA4510A6219E14888741FCFC19BFBB82"
    }
    ```

1. Dans l’Explorateur de solutions, sélectionnez le fichier, puis dans Propriétés, définissez le paramètre **Copier dans le répertoire de sortie** sur **Copier si plus récent**.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-copy-if-newer.png" alt-text="Recherche de *pool*" border="true":::

## <a name="model-data-structures"></a>Modéliser des structures de données

Les modèles (classes C#) permettent de communiquer des données entre le client (vue), le serveur (contrôleur) et également le cloud Azure à l’aide de l’architecture MVC (modèle, vue, contrôleur). En règle générale, ces modèles reflètent la structure des données sollicitées.

Au cours de cette étape, vous allez modéliser les structures de données de l’index de recherche ainsi que la chaîne de recherche utilisée dans les communications entre la vue et le contrôleur. Dans l’index des hôtels, chaque hôtel a plusieurs chambres et une adresse en plusieurs parties. De même, la représentation complète d’un hôtel est une structure de données hiérarchique et imbriquée. Vous aurez besoin de trois classes pour créer chaque composant.

L’ensemble de classes **Hotel**, **Address** et **Room** est connu sous le terme de [*type complexe*](search-howto-complex-data-types.md), une fonctionnalité importante de Recherche cognitive Azure. Les types complexes peuvent avoir de nombreux niveaux de classes et de sous-classes et permettent de représenter des structures de données beaucoup plus complexes qu’au moyen de *types simples* (classe contenant uniquement des membres primitifs).

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **Modèles** > **Ajouter** > **Nouvel élément**.

1. Sélectionnez **Classe** et nommez l’élément Hotel.cs. Remplacez tout le contenu de Hotel.cs par le code suivant. Notez les membres **Address** et **Room** de la classe ; ces champs étant eux-mêmes des classes, vous aurez également besoin de modèles pour eux.

    ```csharp
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;
    using Microsoft.Spatial;
    using System;
    using System.Text.Json.Serialization;

    namespace FirstAzureSearchApp.Models
    {
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
    }
    ```

1. Répétez la même procédure de création de modèle pour la classe **Address**, en nommant le fichier Address.cs. Remplacez le contenu par ce qui suit.

    ```csharp
    using Azure.Search.Documents.Indexes;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Address
        {
            [SearchableField]
            public string StreetAddress { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string City { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string StateProvince { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string PostalCode { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string Country { get; set; }
        }
    }
    ```

1. Là encore, suivez le même processus pour créer la classe **Room**, mais nommez le fichier Room.cs.

    ```csharp
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;
    using System.Text.Json.Serialization;

    namespace FirstAzureSearchApp.Models
    {
        public partial class Room
        {
            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnMicrosoft)]
            public string Description { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrMicrosoft)]
            [JsonPropertyName("Description_fr")]
            public string DescriptionFr { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string Type { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public double? BaseRate { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string BedOptions { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public int SleepsCount { get; set; }

            [SimpleField(IsFilterable = true, IsFacetable = true)]
            public bool? SmokingAllowed { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string[] Tags { get; set; }
        }
    }
    ```

1. Le dernier modèle que vous allez créer dans ce tutoriel est une classe nommée **SearchData** et représente l’entrée de l’utilisateur (**searchText**) et la sortie de la recherche (**resultList**). Le type de la sortie, **SearchResults&lt;Hotel&gt;** , est critique ; en effet, il correspond exactement aux résultats de la recherche et vous devez passer cette référence à la vue. Remplacez le modèle par défaut par le code suivant.

    ```csharp
    using Azure.Search.Documents.Models;

    namespace FirstAzureSearchApp.Models
    {
        public class SearchData
        {
            // The text to search for.
            public string searchText { get; set; }

            // The list of results.
            public SearchResults<Hotel> resultList;
        }
    }
    ```

## <a name="create-a-web-page"></a>Créer une page web

Les modèles de projet sont fournis avec un certain nombre de vues clientes situées dans le dossier **Vues**. Les vues exactes dépendent de la version de .NET Core que vous utilisez (la version 3.1 est utilisée dans cet exemple). Pour ce tutoriel, vous allez modifier **Index.cshtml** afin d’inclure les éléments d’une page de recherche.

Supprimez le contenu d’Index.cshtml dans son intégralité et regénérez le fichier dans les étapes suivantes.

1. Ce tutoriel utilise deux petites images dans la vue : un logo Azure et une icône en forme de loupe (azure-logo.png et search.png). Copiez les images depuis le projet GitHub dans le dossier **wwwroot/images** de votre projet.

1. La première ligne d’Index.cshtml doit référencer le modèle utilisé pour communiquer des données entre le client (vue) et le serveur (contrôleur), qui correspond au modèle **SearchData** créé plus haut. Ajoutez cette ligne au fichier Index.cshtml.

    ```csharp
    @model FirstAzureSearchApp.Models.SearchData
    ```

1. Comme il est courant d’entrer un titre pour la vue, les lignes suivantes doivent être :

    ```csharp
    @{
        ViewData["Title"] = "Home Page";
    }
    ```

1. Après le titre, entrez une référence à une feuille de style HTML, que vous allez créer sous peu.

    ```csharp
    <head>
        <link rel="stylesheet" href="~/css/hotels.css" />
    </head>
    ```

1. Le corps de la vue gère deux cas d’usage. Tout d’abord, il doit fournir une page vide dès la première utilisation, avant qu’un texte de recherche ne soit entré. Deuxièmement, il doit gérer les résultats, en plus de la zone de texte de recherche, pour les requêtes répétées.

   Pour gérer les deux cas, vous devez vérifier si le modèle fourni à la vue est Null. Un modèle Null correspond au premier cas d’usage (l’exécution initiale de l’application). Ajoutez le code suivant au fichier Index.cshtml et lisez les commentaires.

    ```csharp
    <body>
    <h1 class="sampleTitle">
        <img src="~/images/azure-logo.png" width="80" />
        Hotels Search
    </h1>

    @using (Html.BeginForm("Index", "Home", FormMethod.Post))
    {
        // Display the search text box, with the search icon to the right of it.
        <div class="searchBoxForm">
            @Html.TextBoxFor(m => m.searchText, new { @class = "searchBox" }) <input class="searchBoxSubmit" type="submit" value="">
        </div>

        @if (Model != null)
        {
            // Show the result count.
            <p class="sampleText">
                @Model.resultList.Count Results
            </p>

            @for (var i = 0; i < Model.resultList.Count; i++)
            {
                // Display the hotel name and description.
                @Html.TextAreaFor(m => m.resultList[i].Document.HotelName, new { @class = "box1" })
                @Html.TextArea($"desc{i}", Model.resultList[i].Document.Description, new { @class = "box2" })
            }
        }
    }
    </body>
    ```

1. Ajoutez la feuille de style. Dans Visual Studio, dans **Fichier**> **Nouveau** > **Fichier**, sélectionnez **Feuille de style** (avec **Général** sélectionné).

   Remplacez le code par défaut par ce qui suit. Nous n’allons pas nous attarder davantage sur ce fichier ; les styles correspondent à une syntaxe HTML standard.

    ```html
    textarea.box1 {
        width: 648px;
        height: 30px;
        border: none;
        background-color: azure;
        font-size: 14pt;
        color: blue;
        padding-left: 5px;
    }

    textarea.box2 {
        width: 648px;
        height: 100px;
        border: none;
        background-color: azure;
        font-size: 12pt;
        padding-left: 5px;
        margin-bottom: 24px;
    }

    .sampleTitle {
        font: 32px/normal 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 32px;
        text-align: left;
    }

    .sampleText {
        font: 16px/bold 'Segoe UI Light',Arial,Helvetica,Sans-Serif;
        margin: 20px 0;
        font-size: 14px;
        text-align: left;
        height: 30px;
    }

    .searchBoxForm {
        width: 648px;
        box-shadow: 0 0 0 1px rgba(0,0,0,.1), 0 2px 4px 0 rgba(0,0,0,.16);
        background-color: #fff;
        display: inline-block;
        border-collapse: collapse;
        border-spacing: 0;
        list-style: none;
        color: #666;
    }

    .searchBox {
        width: 568px;
        font-size: 16px;
        margin: 5px 0 1px 20px;
        padding: 0 10px 0 0;
        border: 0;
        max-height: 30px;
        outline: none;
        box-sizing: content-box;
        height: 35px;
        vertical-align: top;
    }

    .searchBoxSubmit {
        background-color: #fff;
        border-color: #fff;
        background-image: url(/images/search.png);
        background-repeat: no-repeat;
        height: 20px;
        width: 20px;
        text-indent: -99em;
        border-width: 0;
        border-style: solid;
        margin: 10px;
        outline: 0;
    }
    ```

1. Enregistrez le fichier de feuille de style sous le nom hotels.css, dans le dossier **wwwroot/css**, avec le fichier site.css par défaut.

Notre vue est à présent complète. À ce stade, les modèles et les vues sont terminés. Seul reste à définir le contrôleur, qui relie tout cela.

## <a name="define-methods"></a>Définir des méthodes

Au cours de cette étape, vous modifiez le contenu du **contrôleur Home**.

1. Ouvrez le fichier HomeController.cs et remplacez les instructions **using** par ce qui suit.

    ```csharp
    using Azure;
    using Azure.Search.Documents;
    using Azure.Search.Documents.Indexes;
    using FirstAzureSearchApp.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Configuration;
    using System;
    using System.Diagnostics;
    using System.Linq;
    using System.Threading.Tasks;
    ```

### <a name="add-index-methods"></a>Ajouter des méthodes Index

Dans une application MVC, la méthode **index()** est une méthode d’action par défaut pour tout contrôleur. Elle ouvre la page HTML d’index. La méthode par défaut, qui ne prend aucun paramètre, est utilisée dans ce tutoriel pour le cas d’usage de démarrage de l’application : afficher une page de recherche vide.

Dans cette section, nous étendons la méthode pour prendre en charge un deuxième cas d’usage : afficher la page quand un utilisateur a entré du texte de recherche. Pour prendre en charge ce cas, la méthode d’index est étendue afin de prendre un modèle comme paramètre.

1. Ajoutez la méthode suivante, après la méthode **Index()** par défaut.

    ```csharp
        [HttpPost]
        public async Task<ActionResult> Index(SearchData model)
        {
            try
            {
                // Ensure the search string is valid.
                if (model.searchText == null)
                {
                    model.searchText = "";
                }

                // Make the Azure Cognitive Search call.
                await RunQueryAsync(model);
            }

            catch
            {
                return View("Error", new ErrorViewModel { RequestId = "1" });
            }
            return View(model);
        }
    ```

    Notez la déclaration **async** de la méthode et l’appel **await** à **RunQueryAsync**. Ces mots clés ont pour rôle d’effectuer des appels asynchrones et d’éviter, par là même, le blocage de threads sur le serveur.

    Le bloc **catch** utilise le modèle d’erreur qui a été créé par défaut.

### <a name="note-the-error-handling-and-other-default-views-and-methods"></a>Noter la gestion des erreurs et les autres méthodes et vues par défaut

L’ensemble de vues créées par défaut diffère légèrement suivant la version de .NET Core que vous utilisez. Pour .NET Core 3.1, les vues par défaut sont Index, Privacy et Error. Vous pouvez afficher ces pages par défaut lors de l’exécution de l’application et examiner la façon dont elles sont gérées dans le contrôleur.

Vous testerez la vue Error plus loin dans ce tutoriel.

Dans l’exemple GitHub, les vues non utilisées et leurs actions associées sont supprimées.

### <a name="add-the-runqueryasync-method"></a>Ajouter la méthode RunQueryAsync

L’appel de Recherche cognitive Azure est encapsulé dans notre méthode **RunQueryAsync**.

1. Tout d’abord, ajoutez des variables statiques pour configurer le service Azure et un appel pour les lancer.

    ```csharp
        private static SearchClient _searchClient;
        private static SearchIndexClient _indexClient;
        private static IConfigurationBuilder _builder;
        private static IConfigurationRoot _configuration;

        private void InitSearch()
        {
            // Create a configuration using appsettings.json
            _builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
            _configuration = _builder.Build();

            // Read the values from appsettings.json
            string searchServiceUri = _configuration["SearchServiceUri"];
            string queryApiKey = _configuration["SearchServiceQueryApiKey"];

            // Create a service and index client.
            _indexClient = new SearchIndexClient(new Uri(searchServiceUri), new AzureKeyCredential(queryApiKey));
            _searchClient = _indexClient.GetSearchClient("hotels");
        }
    ```

2. Maintenant, ajoutez la méthode **RunQueryAsync** proprement dite.

    ```csharp
    private async Task<ActionResult> RunQueryAsync(SearchData model)
    {
        InitSearch();

        var options = new SearchOptions() { };

        // Enter Hotel property names into this list so only these values will be returned.
        // If Select is empty, all values will be returned, which can be inefficient.
        options.Select.Add("HotelName");
        options.Select.Add("Description");

        // For efficiency, the search call should be asynchronous, so use SearchAsync rather than Search.
        var searchResult = await _searchClient.SearchAsync<Hotel>(model.searchText, options).ConfigureAwait(false);
        model.resultList = searchResult.Value.GetResults().ToList();

        // Display the results.
        return View("Index", model);
    }
    ```

    Dans cette méthode, assurez-vous d’abord que notre configuration Azure est lancée, puis définissez certaines options de recherche. L’option **Select** spécifie les champs à retourner dans les résultats, correspondant aux noms des propriétés dans la classe **hotel**. Si vous omettez **Select**, tous les champs non masqués sont retournés, ce qui peut être inefficace si vous êtes uniquement intéressé par un sous-ensemble de tous les champs possibles.

    L’appel asynchrone de recherche formule la demande (modélisée en tant que **searchText**) et la réponse (modélisée en tant que **searchResult**). Si vous déboguez ce code, la classe **SearchResult** est un bon candidat à la définition d’un point d’arrêt si vous devez examiner le contenu de **model.resultList**. Vous devriez trouver cela intuitif, les résultats contenant pour l’essentiel les données demandées.

### <a name="test-the-app"></a>Test de l'application

À présent, nous allons vérifier si l’application s’exécute correctement.

1. Sélectionnez **Déboguer** > **Démarrer sans débogage** ou appuyez sur **F5**. Si l’application s’exécute comme prévu, vous devez obtenir la vue d’index initiale.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-index.png" alt-text="Recherche de *pool*" border="true":::

1. Entrez une chaîne de requête telle que « beach » (ou n’importe quel texte qui vous vient à l’esprit), puis cliquez sur l’icône de recherche pour envoyer la demande.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-beach.png" alt-text="Recherche de *pool*" border="true":::

1. Essayez d’entrer « five star ». Notez que cette requête ne retourne aucun résultat. Une recherche plus sophistiquée traiterait « five star » (cinq étoiles) comme un synonyme de « luxury » (luxe) et retournerait les résultats correspondants. La prise en charge des [synonymes](search-synonyms.md) est disponible dans Recherche cognitive Azure, mais n’est pas traitée dans cette série de tutoriels.

1. Essayez d’entrer « hot » comme texte de recherche. Vous n’obtenez _aucune_ entrée contenant le mot « hotel ». Notre recherche porte uniquement sur les mots entiers, même si quelques résultats sont retournés.

1. Essayez d’autres mots, tels que « pool », « sunshine » ou « view ». Vous verrez Recherche cognitive Azure fonctionner à son niveau le plus simple, mais néanmoins convaincant.

## <a name="test-edge-conditions-and-errors"></a>Tester des erreurs et conditions limites

Il est important de vérifier que nos fonctionnalités de gestion des erreurs fonctionnent comme prévu, même quand les choses fonctionnent parfaitement. 

1. Dans la méthode **Index**, après l’appel **try {** , entrez la ligne **Throw new Exception()** . Cette exception force une erreur quand vous effectuez une recherche sur du texte.

2. Exécutez l’application, entrez « bar » comme texte de recherche, puis cliquez sur l’icône de recherche. L’exception doit aboutir à la vue d’erreur.

     :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-error.png" alt-text="Recherche de *pool*" border="true":::

    > [!Important]
    > Il est considéré comme risqué du point de vue de la sécurité de retourner des numéros d’erreur interne dans les pages d’erreur. Si votre application est prévue pour une utilisation générale, effectuez quelques recherches concernant la sécurité et les bonnes pratiques relatives à ce qu’il faut retourner en cas d’erreur.

3. Supprimez **Throw new Exception()** quand vous jugez que la gestion des erreurs fonctionne correctement.

## <a name="takeaways"></a>Éléments importants à retenir

Retenez les points importants suivants de ce projet :

* Un appel de Recherche cognitive Azure est concis et il est facile d’interpréter les résultats.
* Les appels asynchrones ajoutent une petite dose de complexité au contrôleur, mais constituent la meilleure approche si vous avez l’intention de développer des applications de qualité.
* Cette application a effectué une recherche de texte simple, définie par ce qui est configuré dans **searchOptions**. Toutefois, cette classe peut être remplie avec de nombreux membres qui rendent une recherche plus sophistiquée. Il n’est pas nécessaire d’effectuer un travail supplémentaire considérable pour rendre cette application beaucoup plus puissante.

## <a name="next-steps"></a>Étapes suivantes

Pour améliorer l’expérience utilisateur, ajoutez des fonctionnalités, notamment la pagination (au moyen de numéros de page ou d’un défilement infini) ainsi que l’autocomplétion et les suggestions. Vous pouvez aussi envisager des options de recherche plus complexes (par exemple, des recherches géographiques permettant de trouver des hôtels dans un rayon spécifique autour d’un point donné et le tri des résultats de la recherche).

Ces prochaines étapes sont traitées dans les autres tutoriels. Commençons par la pagination.

> [!div class="nextstepaction"]
> [Tutoriel C# : Pagination des résultats de la recherche - Recherche cognitive Azure](tutorial-csharp-paging.md)