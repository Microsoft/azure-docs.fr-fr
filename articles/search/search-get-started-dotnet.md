---
title: 'Démarrage rapide : Créer un index de recherche en .NET'
titleSuffix: Azure Cognitive Search
description: Ce guide de démarrage rapide C# explique comment créer un index, charger des données et exécuter des requêtes à l’aide de la bibliothèque de client Azure.Search.Documents.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 11/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 448f2b3e42e98d78652a005f5d1c11f55acdebb3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021182"
---
# <a name="quickstart-create-a-search-index-using-the-azuresearchdocuments-client-library"></a>Démarrage rapide : Créer un index de recherche à l’aide de la bibliothèque de client Azure.Search.Documents

Utilisez la nouvelle [bibliothèque de client Azure.Search.Documents (version 11)](/dotnet/api/overview/azure/search.documents-readme) pour créer une application console .NET Core en C#, qui crée, charge et interroge un index de recherche.

Vous pouvez [télécharger le code source](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v11) pour commencer avec un projet terminé ou suivre les étapes décrites dans cet article pour créer votre propre projet.

> [!NOTE]
> Vous recherchez une version antérieure ? Consultez [Créer un index de recherche à l’aide de Microsoft.Azure.Search v10](search-get-started-dotnet-v10.md) à la place.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous devez disposer des outils et services suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

+ Service Recherche cognitive Azure. [Créez un service](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Vous pouvez utiliser un service gratuit pour ce guide de démarrage rapide. 

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/), toute édition. L’exemple de code a été testé sur l’édition Community gratuite de Visual Studio 2019.

Lors de la configuration de votre projet, vous téléchargerez le [package NuGet Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/).

Le Kit SDK Azure pour .NET est conforme à [.NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support), ce qui signifie .NET Framework 4.6.1 et .NET Core 2.0 comme configuration requise minimale.

## <a name="set-up-your-project"></a>Configuration de votre projet

Rassemblez les informations de connexion du service, puis démarrez Visual Studio pour créer un projet d’application console qui peut s’exécuter sur .NET Core.

<a name="get-service-info"></a>

### <a name="copy-a-key-and-endpoint"></a>Copier une clé et un point de terminaison

Les appels au service nécessitent un point de terminaison d’URL et une clé d’accès pour chaque requête. Dans un premier temps, recherchez la clé et l’URL de l’API à ajouter à votre projet. Vous allez spécifier les deux valeurs lors de la création du client dans une étape ultérieure.

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

2. Dans **Paramètres** > **Clés**, procurez-vous une clé d’administration pour disposer de droits complets sur le service, indispensables pour créer ou supprimer des objets. Il existe deux clés, primaire et secondaire, interchangeables. Vous pouvez utiliser celle de votre choix.

   ![Obtenir un point de terminaison et une clé d’accès HTTP](media/search-get-started-rest/get-url-key.png "Obtenir un point de terminaison et une clé d’accès HTTP")

Toutes les demandes nécessitent une clé API sur chaque demande envoyée à votre service. L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

### <a name="install-the-nuget-package"></a>Installez le package NuGet

Une fois le projet créé, ajoutez la bibliothèque de client. Le [package Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) consiste en une bibliothèque de client qui fournit toutes les API utilisées pour utiliser un service de recherche dans .NET.

1. Démarrez Visual Studio et créez une application console .NET Core.

1. Dans **Outils** > **Gestionnaire de package NuGet**, sélectionnez **Gérer les packages NuGet pour la solution...** . 

1. Cliquez sur **Parcourir**.

1. Recherchez `Azure.Search.Documents` et sélectionnez la version 11.0 ou ultérieure.

1. Cliquez sur **Installer** à droite pour ajouter l’assembly à vos projet et solution.

### <a name="create-a-search-client"></a>Créer un client de recherche

1. Dans **Program.cs**, remplacez l’espace de noms par `AzureSearch.SDK.Quickstart.v11`, puis ajoutez les directives `using` suivantes.

   ```csharp
   using Azure;
   using Azure.Search.Documents;
   using Azure.Search.Documents.Indexes;
   using Azure.Search.Documents.Indexes.Models;
   using Azure.Search.Documents.Models;
   ```

1. Créez deux clients : [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) crée l’index et [SearchClient](/dotnet/api/azure.search.documents.searchclient) charge et interroge un index existant. Tous deux ont besoin du point de terminaison de service et d’une clé API d’administration pour l’authentification avec des droits de création/suppression.

   ```csharp
   static void Main(string[] args)
   {
       string serviceName = "<YOUR-SERVICE-NAME>";
       string indexName = "hotels-quickstart";
       string apiKey = "<YOUR-ADMIN-API-KEY>";

        // Create a SearchIndexClient to send create/delete index commands
        Uri serviceEndpoint = new Uri($"https://{serviceName}.search.windows.net/");
        AzureKeyCredential credential = new AzureKeyCredential(apiKey);
        SearchIndexClient adminClient = new SearchIndexClient(serviceEndpoint, credential);

        // Create a SearchClient to load and query documents
        SearchClient srchclient = new SearchClient(serviceEndpoint, indexName, credential);
    ```

## <a name="1---create-an-index"></a>1 – Créer un index

Ce guide de démarrage rapide crée un index Hotels que vous allez charger avec des données sur des hôtels et sur lequel vous allez effectuer des requêtes. Dans cette étape, définissez les champs de l’index. Chaque définition de champ comprend un nom, un type de données et des attributs qui déterminent la façon dont le champ est utilisé.

Dans cet exemple, des méthodes synchrones de la bibliothèque Azure.Search.Documents sont utilisées à des fins de simplicité et de lisibilité. En revanche, dans des scénarios de production, vous devez utiliser des méthodes asynchrones pour maintenir la scalabilité et la réactivité de votre application. Par exemple, vous pouvez utiliser [CreateIndexAsync](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindexasync) au lieu de [CreateIndex.](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindex)

1. Ajoutez une définition de classe vide à votre projet : **Hotel.cs**

1. Copiez le code suivant dans **Hotel.cs** pour définir la structure d’un document d’hôtel. Les attributs du champ déterminent la façon dont il est utilisé dans une application. Par exemple, l’attribut `IsFilterable` doit être assigné à chaque champ qui prend en charge une expression de filtre.

    ```csharp
    using System;
    using System.Text.Json.Serialization;
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;

    namespace AzureSearch.Quickstart
    {
        public partial class Hotel
        {
            [SimpleField(IsKey = true, IsFilterable = true)]
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

            [SearchableField]
            public Address Address { get; set; }
        }
    }
    ```

   Dans la bibliothèque de client Azure.Search.Documents, vous pouvez utiliser [SearchableField](/dotnet/api/azure.search.documents.indexes.models.searchablefield) et [SimpleField](/dotnet/api/azure.search.documents.indexes.models.simplefield) pour simplifier les définitions de champs. Les deux sont des dérivés d’un [SearchField](/dotnet/api/azure.search.documents.indexes.models.searchfield) et peuvent potentiellement simplifier votre code :

   + `SimpleField` peut être n’importe quel type de données, ne peut jamais faire l’objet d’une recherche (il est ignoré pour les requêtes de recherche de texte intégral) et peut être récupéré (il n’est pas masqué). Les autres attributs sont désactivés par défaut, mais peuvent être activés. Vous pouvez utiliser un `SimpleField` pour les ID de document ou les champs utilisés seulement dans des filtres, des facettes ou des profils de scoring. Dans ce cas, veillez à appliquer tous les attributs nécessaires pour le scénario, comme `IsKey = true` pour un ID de document. Pour plus d’informations, consultez [SimpleFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SimpleFieldAttribute.cs) dans le code source.

   + `SearchableField` doit être une chaîne, et peut toujours faire l’objet d’une recherche et d’une récupération. Les autres attributs sont désactivés par défaut, mais peuvent être activés. Comme ce type de champ peut faire l’objet d’une recherche, il prend en charge les synonymes et l’ensemble complet des propriétés de l’analyseur. Pour plus d’informations, consultez [SearchableFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SearchableFieldAttribute.cs) dans le code source.

   Que vous utilisiez l’API `SearchField` de base ou un des modèles d’assistance, vous devez activer explicitement les attributs de filtre, de facette et de tri. Par exemple, les attributs [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable), [IsSortable](/dotnet/api/azure.search.documents.indexes.models.searchfield.issortable) et [IsFacetable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfacetable) doivent être explicitement attribués, comme illustré dans l’exemple ci-dessus. 

1. Ajoutez une deuxième définition de classe vide à votre projet : **Address.cs**.  Copiez le code suivant dans la classe.

   ```csharp
   using Azure.Search.Documents.Indexes;

    namespace AzureSearch.Quickstart
    {
        public partial class Address
        {
            [SearchableField(IsFilterable = true)]
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

1. Créez deux autres classes : **Hotel.Methods.cs** et **Address.Methods.cs** pour les substitutions ToString(). Ces classes servent à afficher les résultats de la recherche dans la sortie de la console.  Le contenu de ces classes n’est pas fourni dans cet article, mais vous pouvez copier le code à partir de [fichiers dans GitHub](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v11/AzureSearchQuickstart-v11).

1. Dans **Program.cs**, créez un objet [SearchIndex](/dotnet/api/azure.search.documents.indexes.models.searchindex), puis appelez la méthode [CreateIndex](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindex) pour exprimer l’index dans votre service de recherche. L’index comprend également un [SearchSuggester](/dotnet/api/azure.search.documents.indexes.models.searchsuggester) pour activer l’autocomplétion sur les champs spécifiés.

   ```csharp
    // Create hotels-quickstart index
    private static void CreateIndex(string indexName, SearchIndexClient adminClient)
    {
        FieldBuilder fieldBuilder = new FieldBuilder();
        var searchFields = fieldBuilder.Build(typeof(Hotel));

        var definition = new SearchIndex(indexName, searchFields);

        var suggester = new SearchSuggester("sg", new[] { "HotelName", "Category", "Address/City", "Address/StateProvince" });
        definition.Suggesters.Add(suggester);

        adminClient.CreateOrUpdateIndex(definition);
    }
   ```

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 – Charger des documents

Le service Recherche cognitive Azure recherche dans le contenu stocké dans le service. Au cours de cette étape, vous allez charger des documents JSON conformes à l’index de l’hôtel que vous venez de créer.

Dans Recherche cognitive Azure, les documents de recherche sont des structures de données qui sont à la fois des entrées pour l’indexation et des sorties de requêtes. Selon une source de données externe, les entrées de documents peuvent être des lignes dans une base de données, des objets blob dans le Stockage Blob ou des documents JSON sur le disque. Dans cet exemple, nous prenons un raccourci et incorporons des documents JSON pour quatre hôtels dans le code lui-même. 

Lors du chargement de documents, vous devez utiliser un objet [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1). Un objet `IndexDocumentsBatch` contient une collection d’[Actions](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1.actions), chacune contenant un document et une propriété qui indiquent à Recherche cognitive Azure l’action à effectuer ([upload, merge, delete et mergeOrUpload](search-what-is-data-import.md#indexing-actions)).

1. Dans **Program.cs**, créez un tableau des documents et des actions d’index, puis passez le tableau à `IndexDocumentsBatch`. Les documents ci-dessous sont conformes à l’index hotels-quickstart, tel que défini par la classe hotel.

    ```csharp
    // Upload documents in a single Upload request.
    private static void UploadDocuments(SearchClient searchClient)
    {
        IndexDocumentsBatch<Hotel> batch = IndexDocumentsBatch.Create(
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "1",
                    HotelName = "Secret Point Motel",
                    Description = "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
                    DescriptionFr = "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "air conditioning", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(1970, 1, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.6,
                    Address = new Address()
                    {
                        StreetAddress = "677 5th Ave",
                        City = "New York",
                        StateProvince = "NY",
                        PostalCode = "10022",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "2",
                    HotelName = "Twin Dome Motel",
                    Description = "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "free wifi", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(1979, 2, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.60,
                    Address = new Address()
                    {
                        StreetAddress = "140 University Town Center Dr",
                        City = "Sarasota",
                        StateProvince = "FL",
                        PostalCode = "34243",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "3",
                    HotelName = "Triple Landscape Hotel",
                    Description = "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Resort and Spa",
                    Tags = new[] { "air conditioning", "bar", "continental breakfast" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(2015, 9, 20, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4.80,
                    Address = new Address()
                    {
                        StreetAddress = "3393 Peachtree Rd",
                        City = "Atlanta",
                        StateProvince = "GA",
                        PostalCode = "30326",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "4",
                    HotelName = "Sublime Cliff Hotel",
                    Description = "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
                    DescriptionFr = "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
                    Category = "Boutique",
                    Tags = new[] { "concierge", "view", "24-hour front desk service" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(1960, 2, 06, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4.60,
                    Address = new Address()
                    {
                        StreetAddress = "7400 San Pedro Ave",
                        City = "San Antonio",
                        StateProvince = "TX",
                        PostalCode = "78216",
                        Country = "USA"
                    }
                })
            );

        try
        {
            IndexDocumentsResult result = searchClient.IndexDocuments(batch);
        }
        catch (Exception)
        {
            // If for some reason any documents are dropped during indexing, you can compensate by delaying and
            // retrying. This simple demo just logs the failed document keys and continues.
            Console.WriteLine("Failed to index some of the documents: {0}");
        }
    }
    ```

    Une fois que vous avez initialisé l’objet [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1), vous pouvez l’envoyer à l’index en appelant [IndexDocuments](/dotnet/api/azure.search.documents.searchclient.indexdocuments) sur votre objet [SearchClient](/dotnet/api/azure.search.documents.searchclient).

1. Ajoutez les lignes suivantes à Main(). Le chargement de documents s’effectue à l’aide de SearchClient, mais l’opération nécessite également des droits d’administrateur sur le service, généralement associé à SearchIndexClient. L’une des manières de configurer cette opération consiste à récupérer SearchClient par le biais de SearchIndexClient (adminClient dans cet exemple).

   ```csharp
    SearchClient ingesterClient = adminClient.GetSearchClient(indexName);

    // Load documents
    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(ingesterClient);
   ```

1. Étant donné qu’il s’agit d’une application console qui exécute toutes les commandes de manière séquentielle, ajoutez un délai d’attente de 2 secondes entre l’indexation et les requêtes.

    ```csharp
    // Wait 2 seconds for indexing to complete before starting queries (for demo and console-app purposes only)
    Console.WriteLine("Waiting for indexing...\n");
    System.Threading.Thread.Sleep(2000);
    ```

    Le retard de 2 secondes compense l’indexation, qui est asynchrone, afin que tous les documents puissent être indexés avant l’exécution des requêtes. Le codage dans un retard n’est nécessaire que dans les démonstrations, les tests et les exemples d’applications.

## <a name="3---search-an-index"></a>3 – Rechercher dans un index

Vous pouvez obtenir les résultats de la requête dès que le premier document est indexé, mais les tests réels de votre index doivent attendre que tous les documents soient indexés.

Cette section ajoute deux éléments de fonctionnalité : logique de requête et résultats. Pour les requêtes, utilisez la méthode [Search](/dotnet/api/azure.search.documents.searchclient.search). Cette méthode prend le texte de recherche (la chaîne de requête) ainsi que d’autres [options](/dotnet/api/azure.search.documents.searchoptions).

La classe [SearchResults](/dotnet/api/azure.search.documents.models.searchresults-1) représente les résultats.

1. Dans **Program.cs**, créez une méthode **WriteDocuments** qui affiche les résultats de la recherche sur la console.

    ```csharp
    // Write search results to console
    private static void WriteDocuments(SearchResults<Hotel> searchResults)
    {
        foreach (SearchResult<Hotel> result in searchResults.GetResults())
        {
            Console.WriteLine(result.Document);
        }

        Console.WriteLine();
    }
    ```

1. Créez une méthode **RunQueries** pour exécuter des requêtes et retourner des résultats. Les résultats sont des objets Hotel. Cet exemple montre la signature de la méthode et la première requête. Cette requête montre le paramètre Select qui vous permet de composer le résultat à l’aide de champs sélectionnés à partir du document.

    ```csharp
    // Run queries, use WriteDocuments to print output
    private static void RunQueries(SearchClient srchclient)
    {
        SearchOptions options;
        SearchResults<Hotel> response;

        Console.WriteLine("Query #1: Search on empty term '*' to return all documents, showing a subset of fields...\n");

        options = new SearchOptions()
        {
            IncludeTotalCount = true,
            Filter = "",
            OrderBy = { "" }
        };

        options.Select.Add("HotelId");
        options.Select.Add("HotelName");
        options.Select.Add("Address/City");

        response = srchclient.Search<Hotel>("*", options);
        WriteDocuments(response);
        ```

1. In the second query, search on a term, add a filter that selects documents where Rating is greater than 4, and then sort by Rating in descending order. Filter is a boolean expression that is evaluated over [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable) fields in an index. Filter queries either include or exclude values. As such, there is no relevance score associated with a filter query. 

    ```csharp
    Console.WriteLine("Query #2: Search on 'hotels', filter on 'Rating gt 4', sort by Rating in descending order...\n");

    options = new SearchOptions()
    {
        Filter = "Rating gt 4",
        OrderBy = { "Rating desc" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Rating");

    response = srchclient.Search<Hotel>("hotels", options);
    WriteDocuments(response);
    ```

1. La troisième requête illustre l’utilisation de searchFields, qui sert à limiter l’étendue d’une opération de recherche en texte intégral à des champs spécifiques.

    ```csharp
    Console.WriteLine("Query #3: Limit search to specific fields (pool in Tags field)...\n");

    options = new SearchOptions()
    {
        SearchFields = { "Tags" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Tags");

    response = srchclient.Search<Hotel>("pool", options);
    WriteDocuments(response);
    ```

1. La quatrième requête montre des facettes, qui peuvent être utilisées pour créer une structure de navigation à facettes. 

   ```csharp
    Console.WriteLine("Query #4: Facet on 'Category'...\n");

    options = new SearchOptions()
    {
        Filter = ""
    };

    options.Facets.Add("Category");

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Category");

    response = srchclient.Search<Hotel>("*", options);
    WriteDocuments(response);
   ```

1. La cinquième requête retourne un document spécifique. Une recherche de document est une réponse classique à l’événement OnClick dans un jeu de résultats.

   ```csharp
    Console.WriteLine("Query #5: Look up a specific document...\n");

    Response<Hotel> lookupResponse;
    lookupResponse = srchclient.GetDocument<Hotel>("3");

    Console.WriteLine(lookupResponse.Value.HotelId);
   ```

1. La dernière requête affiche la syntaxe pour l’autocomplétion ; elle simule une entrée utilisateur partielle « sa » qui est résolue en deux correspondances possibles dans les sourceFields associés au suggesteur que vous avez défini dans l’index.

   ```csharp
    Console.WriteLine("Query #6: Call Autocomplete on HotelName that starts with 'sa'...\n");

    var autoresponse = srchclient.Autocomplete("sa", "sg");
    WriteDocuments(autoresponse);
   ```

1. Ajoutez **RunQueries** à Main().

    ```csharp
    // Call the RunQueries method to invoke a series of queries
    Console.WriteLine("Starting queries...\n");
    RunQueries(srchclient);

    // End the program
    Console.WriteLine("{0}", "Complete. Press any key to end this program...\n");
    Console.ReadKey();
    ```

Les requêtes précédentes illustrent plusieurs [manières d’établir des correspondances entre des termes dans une requête](search-query-overview.md#types-of-queries) : recherche en texte intégral, filtres et autocomplétion.

La recherche en texte intégral et les filtres sont exécutés à l’aide de la méthode [SearchClient.Search](/dotnet/api/azure.search.documents.searchclient.search). Une requête de recherche peut être transmise dans la chaîne `searchText`, tandis qu’une expression de filtre peut être transmise dans la propriété [Filter](/dotnet/api/azure.search.documents.searchoptions.filter) de la classe [SearchOptions](/dotnet/api/azure.search.documents.searchoptions). Pour filtrer sans effectuer de recherche, transmettez simplement `"*"` pour le paramètre `searchText` de la méthode [Search](/dotnet/api/azure.search.documents.searchclient.search). Pour effectuer une recherche sans filtrage, ne définissez pas la propriété `Filter` et ne transmettez aucune instance `SearchOptions`.

## <a name="run-the-program"></a>Exécuter le programme

Appuyez sur F5 pour regénérer l’application et exécuter le programme dans son intégralité. 

La sortie comprend des messages de la méthode [Console.WriteLine](/dotnet/api/system.console.writeline), avec en plus des informations et des résultats de requête.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide C#, vous avez effectué un ensemble de tâches pour créer un index, le charger avec des documents et exécuter des requêtes. À différents stades, nous avons pris des raccourcis afin de simplifier le code pour une meilleure lisibilité et compréhension. Si vous êtes familiarisé avec les concepts de base, nous vous recommandons l’article suivant pour explorer d’autres approches et concepts afin d’approfondir vos connaissances. 

> [!div class="nextstepaction"]
> [Guide pratique pour développer dans .NET](search-howto-dotnet-sdk.md)

Vous souhaitez optimiser et réduire vos coûts de cloud ?

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts avec Cost Management](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)