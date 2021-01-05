---
title: Démarrage rapide C# hérité
titleSuffix: Azure Cognitive Search
description: Ce démarrage rapide C# utilise la version 10 de la bibliothèque de client (Microsoft.Azure.Search) pour créer, charger et interroger un index de recherche.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 10/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 8dc2eb898c12e374bc503c5a05f00eb20667443b
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701838"
---
# <a name="quickstart-create-a-search-index-using-the-legacy-microsoftazuresearch-v10-client-library"></a>Démarrage rapide : Créer un index de recherche à l’aide de la bibliothèque de client héritée Microsoft.Azure.Search v10

Cet article a trait au démarrage rapide C# pour la bibliothèque de client héritée [**Microsoft.Azure.Search**](/dotnet/api/overview/azure/search/client10) (version 10), désormais remplacée par la bibliothèque de client [**Azure.Search.Documents**](/dotnet/api/overview/azure/search.documents-readme) (version 11).

> [!NOTE]
> Si vous avez des projets de développement existants ou en préparation, vous pouvez continuer à utiliser la version 10. Toutefois, pour de nouveaux projets, ou pour utiliser de nouvelles fonctionnalités, vous devez passer à la [nouvelle bibliothèque](/dotnet/api/overview/azure/search.documents-readme).

## <a name="about-this-quickstart"></a>À propos de ce démarrage rapide

Générez une application console .NET Core en C#, qui crée, charge et interroge un index Recherche cognitive Azure à l’aide de Visual Studio et des [bibliothèques de client Microsoft.Azure.Search](/dotnet/api/overview/azure/search/client10). 

Cet article explique comment créer l’application. Vous pouvez aussi [télécharger et exécuter l’application complète](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v10).

> [!NOTE]
> Le code de démonstration présent dans cet article utilise les méthodes synchrones du Kit de développement logiciel (SDK) .NET Recherche cognitive Azure version 10 par simplicité. Toutefois, pour les scénarios de production, nous recommandons d’utiliser les méthodes asynchrones dans vos propres applications pour les rendre scalables et réactives. Par exemple, vous pouvez utiliser `CreateAsync` et `DeleteAsync` au lieu de `Create` et `Delete`.

## <a name="prerequisites"></a>Prérequis

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

+ Service Recherche cognitive Azure. [Créez un service](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) dans votre abonnement actuel. Vous pouvez utiliser un service gratuit pour ce guide de démarrage rapide. 

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/), toute édition. L’exemple de code et les instructions ont été testés dans l’édition Communauté gratuite.

<a name="get-service-info"></a>

## <a name="get-a-key-and-url"></a>Obtenir une clé et une URL

Les appels au service nécessitent un point de terminaison d’URL et une clé d’accès pour chaque requête. Un service de recherche est créé avec les deux. Ainsi, si vous avez ajouté la Recherche cognitive Azure à votre abonnement, effectuez ce qui suit pour obtenir les informations nécessaires :

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

2. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

   Obtenez aussi la clé de requête. Il est recommandé d’émettre des demandes de requête avec un accès en lecture seule.

![Obtenir un point de terminaison et une clé d’accès HTTP](media/search-get-started-rest/get-url-key.png "Obtenir un point de terminaison et une clé d’accès HTTP")

Toutes les demandes nécessitent une clé API sur chaque demande envoyée à votre service. L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="set-up-your-environment"></a>Configurer votre environnement

Commencez par ouvrir Visual Studio et créer un nouveau projet d’application console pouvant s’exécuter sur .NET Core.

### <a name="install-nuget-packages"></a>Installer les packages NuGet

Le [package Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/) est constitué de quelques bibliothèques de client distribuées comme packages NuGet.

Pour ce projet, utilisez la version 10 du package NuGet `Microsoft.Azure.Search` ainsi que le dernier package NuGet `Microsoft.Extensions.Configuration.Json`.

1. Dans **Outils** > **Gestionnaire de package NuGet**, sélectionnez **Gérer les packages NuGet pour la solution...** . 

1. Cliquez sur **Parcourir**.

1. Recherchez `Microsoft.Azure.Search` et sélectionnez la version 10.

1. Cliquez sur **Installer** à droite pour ajouter l’assembly à vos projet et solution.

1. Répétez l’opération pour `Microsoft.Extensions.Configuration.Json`, en sélectionnant la version 2.2.0 ou ultérieure.


### <a name="add-azure-cognitive-search-service-information"></a>Ajouter des informations relatives au service Recherche cognitive Azure

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis sélectionnez **Ajouter** > **Nouvel élément...** . 

1. Dans Ajouter un nouvel élément, recherchez « JSON » pour retourner une liste liée à JSON de types d’éléments.

1. Choisissez **Fichier JSON**, nommez le fichier « appsettings.json », puis cliquez sur **Ajouter**. 

1. Ajoutez le fichier à votre répertoire de sortie. Cliquez avec le bouton droit sur appsettings.json et sélectionnez **Propriétés**. Dans **Copier dans le répertoire de sortie**, sélectionnez **Copier si plus récent**.

1. Copiez le code JSON ci-dessous dans votre nouveau fichier JSON. 

    ```json
    {
      "SearchServiceName": "<YOUR-SEARCH-SERVICE-NAME>",
      "SearchServiceAdminApiKey": "<YOUR-ADMIN-API-KEY>",
      "SearchIndexName": "hotels-quickstart"
    }
    ```

1. Remplacez le nom du service de recherche (YOUR-SEARCH-SERVICE-NAME) et la clé API d’administration (YOUR-ADMIN-API-KEY) par des valeurs valides. Si le point de terminaison de votre service est `https://mydemo.search.windows.net`, le nom du service serait « `mydemo` ».

### <a name="add-class-method-files-to-your-project"></a>Ajouter les fichiers de classe « .Method » à votre projet

Cette étape est requise pour produire une sortie significative dans la console. Lors de l’impression des résultats dans la fenêtre de console, des champs individuels à partir de l’objet Hotel doivent être retournés sous forme de chaînes. Cette étape permet d’implémenter [ToString()](/dotnet/api/system.object.tostring) pour effectuer cette tâche, en copiant le code nécessaire dans deux nouveaux fichiers.

1. Ajoutez deux définitions de classe vides à votre projet : Address.Methods.cs, Hotel.Methods.cs

1. Dans Address.Methods.cs, remplacez le contenu par défaut par le code suivant, [lignes 1 à 25](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/quickstart/v10/AzureSearchQuickstart/Address.Methods.cs#L1-L25).

1. Dans Hotel.Methods.cs, copiez les [lignes 1 à 68](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/quickstart/v10/AzureSearchQuickstart/Hotel.Methods.cs#L1-L68).

## <a name="1---create-index"></a>1 - Créer un index

L’index des hôtels se compose de champs simples et complexes, où un champ simple est « HotelName » ou « Description », et les champs complexes sont une adresse avec des sous-champs, ou une collection de chambres. Lorsqu’un index inclut des types complexes, isolez les définitions de champ complexes dans des classes distinctes.

1. Ajoutez deux définitions de classe vides à votre projet : Address.cs, Hotel.cs

1. Dans Address.cs, remplacez le contenu par défaut par le code suivant :

    ```csharp
    using System;
    using Microsoft.Azure.Search;
    using Microsoft.Azure.Search.Models;
    using Newtonsoft.Json;

    namespace AzureSearchQuickstart
    {
        public partial class Address
        {
            [IsSearchable]
            public string StreetAddress { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string City { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string StateProvince { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string PostalCode { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string Country { get; set; }
        }
    }
    ```

1. Dans Hotel.cs, la classe définit la structure globale de l’index, notamment des références à la classe d’adresses.

    ```csharp
    namespace AzureSearchQuickstart
    {
        using System;
        using Microsoft.Azure.Search;
        using Microsoft.Azure.Search.Models;
        using Newtonsoft.Json;

        public partial class Hotel
        {
            [System.ComponentModel.DataAnnotations.Key]
            [IsFilterable]
            public string HotelId { get; set; }

            [IsSearchable, IsSortable]
            public string HotelName { get; set; }

            [IsSearchable]
            [Analyzer(AnalyzerName.AsString.EnMicrosoft)]
            public string Description { get; set; }

            [IsSearchable]
            [Analyzer(AnalyzerName.AsString.FrLucene)]
            [JsonProperty("Description_fr")]
            public string DescriptionFr { get; set; }

            [IsSearchable, IsFilterable, IsSortable, IsFacetable]
            public string Category { get; set; }

            [IsSearchable, IsFilterable, IsFacetable]
            public string[] Tags { get; set; }

            [IsFilterable, IsSortable, IsFacetable]
            public bool? ParkingIncluded { get; set; }

            [IsFilterable, IsSortable, IsFacetable]
            public DateTimeOffset? LastRenovationDate { get; set; }

            [IsFilterable, IsSortable, IsFacetable]
            public double? Rating { get; set; }

            public Address Address { get; set; }
        }
    }
    ```

    Les attributs du champ déterminent la façon dont il est utilisé dans une application. Par exemple, l’attribut `IsSearchable` doit être affecté à tous les champs qui doivent être inclus dans une recherche en texte intégral. 
    
    > [!NOTE]
    > Dans le kit de développement logiciel (SDK) .NET, les champs doivent être explicitement attribués en tant que [`IsSearchable`](/dotnet/api/microsoft.azure.search.models.field.issearchable), [`IsFilterable`](/dotnet/api/microsoft.azure.search.models.field.isfilterable), [`IsSortable`](/dotnet/api/microsoft.azure.search.models.field.issortable) et [`IsFacetable`](/dotnet/api/microsoft.azure.search.models.field.isfacetable). Ce comportement contraste avec l’API REST qui active implicitement l’attribution en fonction du type de données (par exemple, les champs de chaîne simple peuvent automatiquement faire l’objet d’une recherche).

    Dans votre index, un seul champ de type `string` doit être désigné en tant que champ de *clé* qui identifie de façon unique chaque document. Dans ce schéma, la clé est `HotelId`.

    Dans cet index, les champs de description utilisent la propriété [`analyzer`](/dotnet/api/microsoft.azure.search.models.field.analyzer) facultative, spécifiée lorsque vous souhaitez remplacer l’analyseur Lucene standard par défaut. Le champ `description_fr` utilise l’analyseur Lucene en français ([FrLucene](/dotnet/api/microsoft.azure.search.models.analyzername.frlucene)), car il stocke le texte en français. Le champ `description` utilise l’analyseur de langue Microsoft facultatif ([EnMicrosoft](/dotnet/api/microsoft.azure.search.models.analyzername.enmicrosoft)).

1. Dans Program.cs, créez une instance de la classe [`SearchServiceClient`](/dotnet/api/microsoft.azure.search.searchserviceclient) pour vous connecter au service, à l’aide de valeurs qui sont stockées dans le fichier de configuration de l’application (appsettings.json). 

   `SearchServiceClient` a une propriété [`Indexes`](/dotnet/api/microsoft.azure.search.searchserviceclient.indexes), qui fournit toutes les méthodes dont vous avez besoin pour créer, lister, mettre à jour ou supprimer des index Recherche cognitive Azure. 

    ```csharp
    using System;
    using System.Linq;
    using System.Threading;
    using Microsoft.Azure.Search;
    using Microsoft.Azure.Search.Models;
    using Microsoft.Extensions.Configuration;

    namespace AzureSearchQuickstart
    {
        class Program {
            // Demonstrates index delete, create, load, and query
            // Commented-out code is uncommented in later steps
            static void Main(string[] args)
            {
                IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
                IConfigurationRoot configuration = builder.Build();

                SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

                string indexName = configuration["SearchIndexName"];

                Console.WriteLine("{0}", "Deleting index...\n");
                DeleteIndexIfExists(indexName, serviceClient);

                Console.WriteLine("{0}", "Creating index...\n");
                CreateIndex(indexName, serviceClient);

                // Uncomment next 3 lines in "2 - Load documents"
                // ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);
                // Console.WriteLine("{0}", "Uploading documents...\n");
                // UploadDocuments(indexClient);

                // Uncomment next 2 lines in "3 - Search an index"
                // Console.WriteLine("{0}", "Searching index...\n");
                // RunQueries(indexClient);

                Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
                Console.ReadKey();
            }

            // Create the search service client
            private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
            {
                string searchServiceName = configuration["SearchServiceName"];
                string adminApiKey = configuration["SearchServiceAdminApiKey"];

                SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
                return serviceClient;
            }

            // Delete an existing index to reuse its name
            private static void DeleteIndexIfExists(string indexName, SearchServiceClient serviceClient)
            {
                if (serviceClient.Indexes.Exists(indexName))
                {
                    serviceClient.Indexes.Delete(indexName);
                }
            }

            // Create an index whose fields correspond to the properties of the Hotel class.
            // The Address property of Hotel will be modeled as a complex field.
            // The properties of the Address class in turn correspond to sub-fields of the Address complex field.
            // The fields of the index are defined by calling the FieldBuilder.BuildForType() method.
            private static void CreateIndex(string indexName, SearchServiceClient serviceClient)
            {
                var definition = new Microsoft.Azure.Search.Models.Index()
                {
                    Name = indexName,
                    Fields = FieldBuilder.BuildForType<Hotel>()
                };

                serviceClient.Indexes.Create(definition);
            }
        }
    }    
    ```

    Si possible, partagez une seule instance de `SearchServiceClient` dans votre application pour éviter l’ouverture d’un trop grand nombre de connexions. Les méthodes de classe sont thread-safe pour permettre ce partage.

   La classe dispose de plusieurs constructeurs. Celle que vous cherchez utilise le nom de votre service de recherche et un objet `SearchCredentials` en tant paramètres. `SearchCredentials` encapsule votre clé API.

    Dans la définition d’index, le moyen le plus simple de créer les objets `Field` consiste à appeler la méthode `FieldBuilder.BuildForType` en transmettant une classe de modèle pour le paramètre de type. Une classe de modèle comporte des propriétés qui sont mappées sur les champs de votre index. Ce mappage vous permet de lier des documents à partir de votre index de recherche à des instances de votre classe de modèle.

    > [!NOTE]
    > Si vous n’envisagez pas d’utiliser une classe de modèle, vous pouvez toujours définir votre index en créant les objets `Field` directement. Vous pouvez fournir le nom du champ au constructeur, ainsi que le type de données (ou un analyseur pour les champs de chaîne). Vous pouvez également définir d’autres propriétés comme `IsSearchable`, `IsFilterable`, etc.
    >

1. Appuyez sur F5 pour générer l’application et créer l’index. 

    Si le projet est correctement généré, une fenêtre de console s’ouvre, en affichant des messages d’état à l’écran pour la suppression et la création de l’index. 

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 – Charger des documents

Dans la Recherche cognitive Azure, les documents sont des structures de données qui sont à la fois des entrées pour l’indexation et des sorties de requêtes. Selon une source de données externe, les entrées de documents peuvent être des lignes dans une base de données, des objets blob dans le Stockage Blob ou des documents JSON sur le disque. Dans cet exemple, nous prenons un raccourci et incorporons des documents JSON pour quatre hôtels dans le code lui-même. 

Lors du chargement de documents, vous devez utiliser un objet [`IndexBatch`](/dotnet/api/microsoft.azure.search.models.indexbatch). `IndexBatch` contient une collection d’objets [`IndexAction`](/dotnet/api/microsoft.azure.search.models.indexaction), chacun contenant un document et une propriété qui indiquent à la Recherche cognitive Azure l’action à effectuer ([upload, merge, delete et mergeOrUpload](search-what-is-data-import.md#indexing-actions)).

1. Dans Program.cs, créez un tableau des documents et des actions d’index, puis transmettez le tableau à `IndexBatch`. Les documents ci-dessous sont conformes à l’index hotels-quickstart, tel que défini par les classes hotel et address.

    ```csharp
    // Upload documents as a batch
    private static void UploadDocuments(ISearchIndexClient indexClient)
    {
        var actions = new IndexAction<Hotel>[]
        {
            IndexAction.Upload(
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
                }
            ),
            IndexAction.Upload(
                new Hotel()
                {
                    HotelId = "2",
                    HotelName = "Twin Dome Motel",
                    Description = "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "free wifi", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate =  new DateTimeOffset(1979, 2, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.60,
                    Address = new Address()
                    {
                        StreetAddress = "140 University Town Center Dr",
                        City = "Sarasota",
                        StateProvince = "FL",
                        PostalCode = "34243",
                        Country = "USA"
                    }
                }
            ),
            IndexAction.Upload(
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
                }
            ),
            IndexAction.Upload(
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
                    Rating = 4.6,
                    Address = new Address()
                    {
                        StreetAddress = "7400 San Pedro Ave",
                        City = "San Antonio",
                        StateProvince = "TX",
                        PostalCode = "78216",
                        Country = "USA"
                    }
                }
            ),
        };

        var batch = IndexBatch.New(actions);

        try
        {
            indexClient.Documents.Index(batch);
        }
        catch (IndexBatchException e)
        {
            // When a service is under load, indexing might fail for some documents in the batch. 
            // Depending on your application, you can compensate by delaying and retrying. 
            // For this simple demo, we just log the failed document keys and continue.
            Console.WriteLine(
                "Failed to index some of the documents: {0}",
                String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
        }

        // Wait 2 seconds before starting queries 
        Console.WriteLine("Waiting for indexing...\n");
        Thread.Sleep(2000);
    }
    ```

    Une fois que vous avez initialisé l’objet `IndexBatch`, vous pouvez l’envoyer à l’index en appelant [`Documents.Index`](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.index) sur votre objet [`SearchIndexClient`](/dotnet/api/microsoft.azure.search.searchindexclient). `Documents` est une propriété de `SearchIndexClient` qui fournit des méthodes pour l’ajout, la modification, la suppression ou l’interrogation de documents dans votre index.

    L’élément `try`/`catch` entourant l’appel à la méthode `Index` intercepte les échecs d’indexation, qui peuvent se produire si votre service est surchargé. Dans le code de production, vous pouvez retarder, puis relancer l’indexation des documents qui ont échoué, ouvrir une session et continuer comme dans l’exemple, ou gérer la situation d’une autre manière qui répond à la cohérence des données requise par votre application.

    Le retard de 2 secondes compense l’indexation, qui est asynchrone, afin que tous les documents puissent être indexés avant l’exécution des requêtes. Le codage dans un retard n’est nécessaire que dans les démonstrations, les tests et les exemples d’applications.

1. Dans Program.cs, dans main, supprimez les commentaires des lignes pour « 2 - Charger des documents ». 

    ```csharp
    // Uncomment next 3 lines in "2 - Load documents"
    ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);
    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(indexClient);
    ```
1. Appuyez sur F5 pour regénérer l’application. 

    Si le projet est correctement généré, une fenêtre de console s’ouvre, en affichant des messages d’état, cette fois sur le chargement des documents. Dans le portail Azure, dans la page **Vue d’ensemble** du service de recherche, l’index hotels-quickstart doit maintenant afficher 4 documents.

Pour plus d’informations sur le traitement des documents, consultez [« Comment le SDK .NET traite les documents »](search-howto-dotnet-sdk-v10.md#how-dotnet-handles-documents).

## <a name="3---search-an-index"></a>3 – Rechercher dans un index

Vous pouvez obtenir les résultats de la requête dès que le premier document est indexé, mais les tests réels de votre index doivent attendre que tous les documents soient indexés. 

Cette section ajoute deux éléments de fonctionnalité : logique de requête et résultats. Pour les requêtes, utilisez la méthode [`Search`](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.search). Cette méthode accepte le texte recherché ainsi que d’autres [paramètres](/dotnet/api/microsoft.azure.search.models.searchparameters). 

La classe [`DocumentsSearchResult`](/dotnet/api/microsoft.azure.search.models.documentsearchresult-1) représente les résultats.


1. Dans Program.cs, créez une méthode WriteDocuments qui imprime les résultats de la recherche dans la console.

    ```csharp
    private static void WriteDocuments(DocumentSearchResult<Hotel> searchResults)
    {
        foreach (SearchResult<Hotel> result in searchResults.Results)
        {
            Console.WriteLine(result.Document);
        }

        Console.WriteLine();
    }
    ```

1. Créez une méthode RunQueries pour exécuter des requêtes et retourner des résultats. Les résultats sont des objets Hotel. Vous pouvez utiliser le paramètre select pour présenter les champs individuels. Si un champ n’est pas inclus dans le paramètre select, sa propriété Hotel correspondante aura la valeur Null.

    ```csharp
    private static void RunQueries(ISearchIndexClient indexClient)
    {
        SearchParameters parameters;
        DocumentSearchResult<Hotel> results;

        // Query 1 
        Console.WriteLine("Query 1: Search for term 'Atlanta' with no result trimming");
        parameters = new SearchParameters();
        results = indexClient.Documents.Search<Hotel>("Atlanta", parameters);
        WriteDocuments(results);

        // Query 2
        Console.WriteLine("Query 2: Search on the term 'Atlanta', with trimming");
        Console.WriteLine("Returning only these fields: HotelName, Tags, Address:\n");
        parameters =
            new SearchParameters()
            {
                Select = new[] { "HotelName", "Tags", "Address" },
            };
        results = indexClient.Documents.Search<Hotel>("Atlanta", parameters);
        WriteDocuments(results);

        // Query 3
        Console.WriteLine("Query 3: Search for the terms 'restaurant' and 'wifi'");
        Console.WriteLine("Return only these fields: HotelName, Description, and Tags:\n");
        parameters =
            new SearchParameters()
            {
                Select = new[] { "HotelName", "Description", "Tags" }
            };
        results = indexClient.Documents.Search<Hotel>("restaurant, wifi", parameters);
        WriteDocuments(results);

        // Query 4 -filtered query
        Console.WriteLine("Query 4: Filter on ratings greater than 4");
        Console.WriteLine("Returning only these fields: HotelName, Rating:\n");
        parameters =
            new SearchParameters()
            {
                Filter = "Rating gt 4",
                Select = new[] { "HotelName", "Rating" }
            };
        results = indexClient.Documents.Search<Hotel>("*", parameters);
        WriteDocuments(results);

        // Query 5 - top 2 results
        Console.WriteLine("Query 5: Search on term 'boutique'");
        Console.WriteLine("Sort by rating in descending order, taking the top two results");
        Console.WriteLine("Returning only these fields: HotelId, HotelName, Category, Rating:\n");
        parameters =
            new SearchParameters()
            {
                OrderBy = new[] { "Rating desc" },
                Select = new[] { "HotelId", "HotelName", "Category", "Rating" },
                Top = 2
            };
        results = indexClient.Documents.Search<Hotel>("boutique", parameters);
        WriteDocuments(results);
    }
    ```

    Il existe deux [façons de mettre en correspondance des termes dans une requête](search-query-overview.md#types-of-queries) : la recherche en texte intégral et les filtres. Une requête de recherche en texte intégral recherche un ou plusieurs termes dans les champs `IsSearchable` de votre index. Un filtre est une expression booléenne évaluée sur les champs `IsFilterable` dans un index. Vous pouvez utiliser la recherche en texte intégral et les filtres conjointement ou séparément.

    Les filtres et les recherches sont exécutés à l’aide de la méthode `Documents.Search` . Une requête de recherche peut être transmise dans le paramètre `searchText`, tandis qu’une expression de filtre peut être transmise dans la propriété `Filter` de la classe `SearchParameters`. Pour filtrer sans effectuer de recherche, transmettez simplement `"*"` pour le paramètre `searchText`. Pour effectuer une recherche sans appliquer de filtre, ne définissez pas la propriété `Filter` et ne transmettez pas une instance `SearchParameters`.

1. Dans Program.cs, dans main, supprimez les commentaires des lignes pour « 3 - Rechercher dans un index ». 

    ```csharp
    // Uncomment next 2 lines in "3 - Search an index"
    Console.WriteLine("{0}", "Searching documents...\n");
    RunQueries(indexClient);
    ```
1. La solution est maintenant terminée. Appuyez sur F5 pour regénérer l’application et exécuter le programme dans son intégralité. 

    La sortie contient les mêmes messages qu’auparavant, avec l’ajout des informations et des résultats de la requête.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide C#, vous avez effectué une série de tâches pour créer un index, le charger avec des documents et exécuter des requêtes. À différents stades, nous avons pris des raccourcis afin de simplifier le code pour une meilleure lisibilité et compréhension. Si vous êtes familiarisé avec les concepts de base, nous vous recommandons l’article suivant pour explorer d’autres approches et concepts afin d’approfondir vos connaissances. 

L’exemple de code et l’index sont des versions développées de celui-ci. L’exemple suivant ajoute une collection Rooms, utilise différentes classes et actions, et examine de plus près le fonctionnement du traitement.

> [!div class="nextstepaction"]
> [Guide pratique pour développer dans .NET](search-howto-dotnet-sdk.md)

Vous souhaitez optimiser et réduire vos coûts de cloud ?

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts avec Cost Management](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)