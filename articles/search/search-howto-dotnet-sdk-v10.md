---
title: Utiliser Microsoft.Azure.Search (v10) dans .NET
titleSuffix: Azure Cognitive Search
description: Découvrez comment créer et gérer des objets de recherche dans une application .NET en utilisant C# et Microsoft.Azure.Search (version 10) du Kit de développement logiciel (SDK) .NET. Les extraits de code montrent comment se connecter au service et créer des index et des requêtes.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.devlang: dotnet
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 11102d95bc2aba65e6bc3cba71805a67f195947b
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92681282"
---
# <a name="how-to-use-microsoftazuresearch-in-a-c-net-application"></a>Utilisation de Microsoft.Azure.Search dans une application .NET C#

Cet article explique comment créer et gérer des objets de recherche en utilisant C# et la bibliothèque de client héritée [**Microsoft.Azure.Search**](/dotnet/api/overview/azure/search/client10) (version 10) dans le Kit de développement logiciel (SDK) Azure pour .NET. 

La version 10 est la version la plus récente du package Microsoft.Azure.Search. À l’avenir, les nouvelles fonctionnalités seront déployées dans [**Azure.Search.Documents**](/dotnet/api/overview/azure/search.documents-readme) par l’équipe du Kit de développement logiciel (SDK) Azure.

> [!NOTE]
> Si vous avez des projets de développement en cours ou existants, vous pouvez continuer à utiliser la version 10. Pour les nouveaux projets, ou pour utiliser de nouvelles fonctionnalités, vous devez passer à la [nouvelle bibliothèque](/dotnet/api/overview/azure/search.documents-readme).

## <a name="about-version-10"></a>À propos de la version 10

Le SDK se compose de quelques bibliothèques clientes qui vous permettent de gérer vos index, sources de données, indexeurs et cartes de synonymes, ainsi que de charger et gérer des documents et d’exécuter des requêtes, sans avoir à gérer les détails de HTTP et de JSON. Ces bibliothèques clientes sont distribuées sous la forme de packages NuGet.

Le package NuGet principal est `Microsoft.Azure.Search`, méta-package qui inclut tous les autres packages en tant que dépendances. Utilisez ce package si vous débutez, ou si vous savez que votre application a besoin de toutes les fonctionnalités de Recherche cognitive Azure.

Les autres packages NuGet dans le SDK sont les suivants :
 
  - `Microsoft.Azure.Search.Data`: Utilisez ce package si vous développez une application .NET à l’aide de Recherche cognitive Azure et que vous devez uniquement interroger ou mettre à jour des documents dans vos index. Si vous devez également créer ou mettre à jour des index, des cartes de synonymes ou d’autres ressources de niveau service, utilisez le package `Microsoft.Azure.Search` à la place.
  - `Microsoft.Azure.Search.Service`: Utilisez ce package si vous développez un processus d’automatisation en .NET pour gérer les index Recherche cognitive Azure, cartes de synonymes, indexeurs, sources de données ou autres ressources de niveau service. Si vous devez uniquement interroger ou mettre à jour des documents dans vos index, utilisez le package `Microsoft.Azure.Search.Data` à la place. Si vous avez besoin de toutes les fonctionnalités de Recherche cognitive Azure, utilisez le package `Microsoft.Azure.Search` à la place.
  - `Microsoft.Azure.Search.Common`: Types courants requis par les bibliothèques .NET de Recherche cognitive Azure. Vous n’avez pas besoin d’utiliser ce package directement dans votre application. Il est uniquement destiné à être utilisé en tant que dépendance.

Les différentes bibliothèques clientes définissent des classes comme `Index`, `Field` et `Document`, ainsi que des opérations telles que `Indexes.Create` et `Documents.Search` sur les classes `SearchServiceClient` et `SearchIndexClient`. Ces classes sont organisées dans les espaces de noms suivants :

* [Microsoft.Azure.Search](/dotnet/api/microsoft.azure.search)
* [Microsoft.Azure.Search.Models.](/dotnet/api/microsoft.azure.search.models)

Si vous souhaitez formuler des commentaires pour une prochaine mise à jour du Kit de développement logiciel (SDK), consultez notre [page de commentaires](https://feedback.azure.com/forums/263029-azure-search/) ou créez un problème sur [GitHub](https://github.com/azure/azure-sdk-for-net/issues) en mentionnant « Recherche cognitive Azure » dans le titre.

Le SDK .NET cible la version [`2019-05-06` de l’API REST de Recherche cognitive Azure](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/search/data-plane/Microsoft.Azure.Search.Data/stable/2019-05-06). Cette version inclut la prise en charge des [types complexes](search-howto-complex-data-types.md), de l’[enrichissement par IA](cognitive-search-concept-intro.md), de la [saisie semi-automatique](/rest/api/searchservice/autocomplete) et du [mode d’analyse JsonLines](search-howto-index-json-blobs.md) lors de l’indexation d’objets blob Azure. 

Ce Kit de développement logiciel (SDK) ne prend pas en charge les [opérations de gestion](/rest/api/searchmanagement/) telles que la création et la mise à l’échelle des services de recherche, ainsi que la gestion des clés API. Si vous avez besoin de gérer vos ressources de recherche à partir d’une application .NET, vous pouvez utiliser le [Kit de développement logiciel (SDK) .NET de la Recherche cognitive Azure](/dotnet/api/overview/azure/search/management).

## <a name="upgrade-to-v10"></a>Mise à niveau vers la version 10
Si vous utilisez déjà une version antérieure du Kit de développement logiciel (SDK) .NET Recherche cognitive Azure et que vous souhaitez mettre à niveau vers la dernière version mise à la disposition générale, [cet article](search-dotnet-sdk-migration-version-9.md) vous explique comment procéder.

## <a name="sdk-requirements"></a>Configuration requise du kit de développement logiciel (SDK)
1. Visual Studio 2017 ou version ultérieure.
2. Votre propre service Recherche cognitive Azure. Pour utiliser le SDK, vous devez connaître le nom de votre service et une ou plusieurs clés API. [Créer un service dans le portail](search-create-service-portal.md) vous guidera à travers ces étapes.
3. Téléchargez le [package NuGet](https://www.nuget.org/packages/Microsoft.Azure.Search) du SDK .NET Recherche cognitive Azure en utilisant « Gérer les packages NuGet » dans Visual Studio. Recherchez simplement le nom de package `Microsoft.Azure.Search` sur NuGet.org (ou l’un des autres noms de package ci-dessus si vous avez uniquement besoin d’un sous-ensemble des fonctionnalités).

Le SDK .NET Recherche cognitive Azure prend en charge les applications qui ciblent .NET Framework versions 4.5.2 et supérieures, ainsi que .NET Core versions 2.0 et supérieures.

## <a name="core-scenarios"></a>Principaux scénarios
Vous devez faire plusieurs choses dans votre application de recherche. Dans ce didacticiel, nous aborderons ces principaux scénarios :

* Création d'un index
* Remplissage de l'index avec des documents
* Recherche de documents à l'aide de filtres et de la recherche en texte intégral

L’exemple de code suivant illustre chacun de ces scénarios. N'hésitez pas à utiliser les extraits de code dans votre propre application.

### <a name="overview"></a>Vue d’ensemble
L’application exemple que nous allons examiner crée un index nommé « hotels », le remplit avec des documents, puis exécute des requêtes de recherche. Voici le programme principal, décrivant le flux global :

```csharp
// This sample shows how to delete, create, upload documents and query an index
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

    ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(indexClient);

    ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

> [!NOTE]
> Vous trouverez le code source complet de l’exemple d’application utilisé dans cette procédure sur [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).
> 
>

Nous allons le détailler, étape par étape. Tout d’abord, nous devons créer un objet `SearchServiceClient`. Cet objet vous permet de gérer les index. Pour en construire un, vous devez indiquer le nom de votre service Recherche cognitive Azure, ainsi qu’une clé API d'administration. Vous pouvez entrer ces informations dans le fichier `appsettings.json` de [l’exemple d’application](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
    return serviceClient;
}
```

> [!NOTE]
> Si vous fournissez une clé incorrecte (par exemple, une clé de requête là où une clé d’administration était demandée), `SearchServiceClient` génère une `CloudException` avec le message d’erreur « Forbidden » la première fois que vous invoquez une méthode d'opération dessus, comme `Indexes.Create`. Si cette situation se produit, vérifiez la clé API.
> 
> 

Les quelques lignes suivantes appellent des méthodes pour créer un index nommé « hotels », en le supprimant s’il existe déjà. Nous étudierons ces méthodes un peu plus tard.

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteIndexIfExists(indexName, serviceClient);

Console.WriteLine("{0}", "Creating index...\n");
CreateIndex(indexName, serviceClient);
```

Ensuite, l'index doit être rempli. Pour remplir l’index, nous aurons besoin d’un `SearchIndexClient`. Il existe deux façons d'en obtenir un : en le créant ou en appelant `Indexes.GetClient` sur `SearchServiceClient`. Pour des raisons pratiques, nous allons opter pour la deuxième solution.

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);
```

> [!NOTE]
> Dans une application de recherche classique, le remplissage et la gestion des index peuvent être gérés par un composant séparé des requêtes de recherche. `Indexes.GetClient` est pratique pour remplir un index car il vous évite de devoir fournir un `SearchCredentials` supplémentaire. Pour ce faire, il transmet la clé d’administration que vous avez utilisée afin de créer le `SearchServiceClient` dans le nouveau `SearchIndexClient`. Toutefois, dans la partie de votre application qui exécute des requêtes, il vaut mieux créer le `SearchIndexClient` directement afin de transmettre une clé de requête qui vous permet uniquement de lire des données au lieu d’une clé d’administration. Cela est conforme au principe du moindre privilège et vous permet de mieux sécuriser votre application. Pour en savoir plus sur les clés d’administration et les clés de requête, cliquez [ici](/rest/api/searchservice/#authentication-and-authorization).
> 
> 

Maintenant que nous avons un `SearchIndexClient`, nous pouvons remplir l'index. Le remplissage d’index est exécuté par une autre méthode que nous décrirons ultérieurement.

```csharp
Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(indexClient);
```

Enfin, nous exécutons quelques requêtes de recherche et affichons les résultats. Cette fois-ci, nous utilisons un `SearchIndexClient` différent :

```csharp
ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(indexName, configuration);

RunQueries(indexClientForQueries);
```

Nous examinerons la méthode `RunQueries` plus en détail un peu plus tard. Voici le code permettant de créer le nouveau `SearchIndexClient`:

```csharp
private static SearchIndexClient CreateSearchIndexClient(string indexName, IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, indexName, new SearchCredentials(queryApiKey));
    return indexClient;
}
```

Cette fois, nous utilisons une clé de requête, car nous n’avons pas besoin d’obtenir un accès en écriture à l’index. Vous pouvez entrer ces informations dans le fichier `appsettings.json` de [l’exemple d’application](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

Si vous exécutez cette application avec un nom de service et une clé API valides, la sortie doit être similaire à l’exemple suivant : (Une partie de la sortie de la console a été remplacée par « ... » pour l’illustration.)

```output

Deleting index...

Creating index...

Uploading documents...

Waiting for documents to be indexed...

Search the entire index for the term 'motel' and return only the HotelName field:

Name: Secret Point Motel

Name: Twin Dome Motel


Apply a filter to the index to find hotels with a room cheaper than $100 per night, and return the hotelId and description:

HotelId: 1
Description: The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Times Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.

HotelId: 2
Description: The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.


Search the entire index, order by a specific field (lastRenovationDate) in descending order, take the top two results, and show only hotelName and lastRenovationDate:

Name: Triple Landscape Hotel
Last renovated on: 9/20/2015 12:00:00 AM +00:00

Name: Twin Dome Motel
Last renovated on: 2/18/1979 12:00:00 AM +00:00


Search the hotel names for the term 'hotel':

HotelId: 3
Name: Triple Landscape Hotel
...

Complete.  Press any key to end application... 
```

Le code source complet de l'application est fourni à la fin de cet article.

Ensuite, nous allons étudier chacune des méthodes appelées par `Main`.

### <a name="creating-an-index"></a>Création d'un index
Après la création d’un `SearchServiceClient`, `Main` supprime l’index « hotels » s’il existe déjà. Cette suppression est effectuée par la méthode suivante :

```csharp
private static void DeleteIndexIfExists(string indexName, SearchServiceClient serviceClient)
{
    if (serviceClient.Indexes.Exists(indexName))
    {
        serviceClient.Indexes.Delete(indexName);
    }
}
```

Cette méthode utilise `SearchServiceClient` pour vérifier si l'index existe et, dans l’affirmative, elle le supprime.

> [!NOTE]
> L'exemple de code dans cet article utilise les méthodes synchrones du SDK .NET Recherche cognitive Azure pour plus de simplicité. Nous vous recommandons d'utiliser les méthodes asynchrones dans vos propres applications pour les rendre évolutives et réactives. Par exemple, dans la méthode ci-dessus, vous pouvez utiliser `ExistsAsync` et `DeleteAsync` au lieu de `Exists` et `Delete`.
> 
> 

Ensuite, `Main` crée un index « hotels » en appelant cette méthode :

```csharp
private static void CreateIndex(string indexName, SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = indexName,
        Fields = FieldBuilder.BuildForType<Hotel>()
    };
    
    serviceClient.Indexes.Create(definition);
}
```

Cette méthode crée un objet `Index` avec une liste d’objets `Field` qui définit le schéma du nouvel index. Chaque champ a un nom, un type de données et plusieurs attributs qui définissent son comportement de recherche. La classe `FieldBuilder` utilise la réflexion pour créer une liste d’objets `Field` pour l’index en examinant les attributs et propriétés publics de la classe de modèle `Hotel` donnée. Nous examinerons la classe `Hotel` plus en détail un peu plus tard.

> [!NOTE]
> Vous pouvez toujours créer directement la liste des objets `Field` au lieu d’utiliser la fonctionnalité `FieldBuilder`, si nécessaire. Par exemple, vous ne souhaitez pas utiliser une classe de modèle, ou vous pouvez être amené à recourir à une classe de modèle existante, que vous ne souhaitez pas modifier en ajoutant des attributs.
>
> 

En plus des champs, vous pouvez ajouter des profils de notation, des générateurs de suggestions ou des options CORS à l’index (ces paramètres sont omis de l’exemple par souci de concision). Vous trouverez plus d’informations sur l’objet Index et ses composants dans la page de [référence sur le Kit de développement logiciel (SDK)](/dotnet/api/microsoft.azure.search.models.index), ainsi que dans la page de [référence sur l’API REST de la Recherche cognitive Azure](/rest/api/searchservice/).

### <a name="populating-the-index"></a>Remplissage de l'index
La prochaine étape dans `Main` remplit l’index nouvellement créé. Ce remplissage d’index s’effectue dans la méthode suivante : (Une partir du code a été remplacée par « ... » pour l’illustration.  Pour le code complet de remplissage des données, voir l’exemple complet de la solution.)

```csharp
private static void UploadDocuments(ISearchIndexClient indexClient)
{
    var hotels = new Hotel[]
    {
        new Hotel()
        {
            HotelId = "1",
            HotelName = "Secret Point Motel",
            ...
            Address = new Address()
            {
                StreetAddress = "677 5th Ave",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Budget Room, 1 Queen Bed (Cityside)",
                    ...
                },
                new Room()
                {
                    Description = "Budget Room, 1 King Bed (Mountain View)",
                    ...
                },
                new Room()
                {
                    Description = "Deluxe Room, 2 Double Beds (City View)",
                    ...
                }
            }
        },
        new Hotel()
        {
            HotelId = "2",
            HotelName = "Twin Dome Motel",
            ...
            {
                StreetAddress = "140 University Town Center Dr",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Suite, 2 Double Beds (Mountain View)",
                    ...
                },
                new Room()
                {
                    Description = "Standard Room, 1 Queen Bed (City View)",
                    ...
                },
                new Room()
                {
                    Description = "Budget Room, 1 King Bed (Waterfront View)",
                    ...
                }
            }
        },
        new Hotel()
        {
            HotelId = "3",
            HotelName = "Triple Landscape Hotel",
            ...
            Address = new Address()
            {
                StreetAddress = "3393 Peachtree Rd",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Standard Room, 2 Queen Beds (Amenities)",
                    ...
                },
                new Room ()
                {
                    Description = "Standard Room, 2 Double Beds (Waterfront View)",
                    ...
                },
                new Room()
                {
                    Description = "Deluxe Room, 2 Double Beds (Cityside)",
                    ...
                }
            }
        }
    };

    var batch = IndexBatch.Upload(hotels);

    try
    {
        indexClient.Documents.Index(batch);
    }
    catch (IndexBatchException e)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
}
```

Cette méthode présente quatre parties. La première crée un tableau de 3 objets `Hotel`, comprenant chacun 3 objets `Room`, qui servent de données en entrée à charger dans l’index. Ces données sont codées en dur pour plus de simplicité. Dans votre application, vos données seront probablement issues d'une source de données externe, comme une base de données SQL.

La deuxième partie crée un `IndexBatch` contenant les documents. Vous spécifiez l’opération que vous souhaitez appliquer au lot au moment de sa création, dans ce cas en appelant `IndexBatch.Upload`. Le lot est ensuite chargé dans l’index Recherche cognitive Azure par la méthode `Documents.Index`.

> [!NOTE]
> Dans cet exemple, nous allons simplement charger les documents. Si vous souhaitez fusionner les modifications dans les documents existants ou supprimer des documents, vous pouvez créer des lots en appelant `IndexBatch.Merge`, `IndexBatch.MergeOrUpload` ou `IndexBatch.Delete` à la place. Vous pouvez aussi combiner plusieurs opérations dans un lot unique en appelant `IndexBatch.New`, qui accepte une collection d’objets `IndexAction`, dont chacun indique à Recherche cognitive Azure d’effectuer une opération spécifique sur un document. Vous pouvez créer chaque `IndexAction` avec sa propre opération en appelant la méthode correspondante comme `IndexAction.Merge`, `IndexAction.Upload`, et ainsi de suite.
> 
> 

La troisième partie de cette méthode est un bloc catch qui gère un cas d'erreur important pour l'indexation. Si votre service Recherche cognitive Azure ne parvient pas à indexer certains documents du lot, `Documents.Index` génère un `IndexBatchException`. Cette exception peut se produire si vous indexez des documents lorsque votre service est surchargé. **Nous vous recommandons vivement de prendre en charge explicitement ce cas de figure dans votre code.** Vous pouvez retarder puis relancer l'indexation des documents qui ont échoué, ouvrir une session et continuer comme dans l’exemple, ou faire autre chose selon la cohérence des données requise par votre application.

> [!NOTE]
> Vous pouvez utiliser la méthode [`FindFailedActionsToRetry`](/dotnet/api/microsoft.azure.search.indexbatchexception.findfailedactionstoretry) pour construire un nouveau lot contenant seulement les actions qui ont échoué lors d’un précédent appel à `Index`. Vous trouverez [sur StackOverflow](https://stackoverflow.com/questions/40012885/azure-search-net-sdk-how-to-use-findfailedactionstoretry) un fil de discussion sur l’utilisation adéquate de cette méthode.
>
>

Enfin, la méthode `UploadDocuments` retarde son exécution de deux secondes. L’indexation s’exécutant en mode asynchrone dans votre service Recherche cognitive Azure, l’exemple d’application doit attendre quelque temps afin de s’assurer que les documents sont disponibles pour la recherche. Ce genre de retard n’est nécessaire que dans les démonstrations, les tests et les exemples d'applications.

<a name="how-dotnet-handles-documents"></a>

#### <a name="how-the-net-sdk-handles-documents"></a>Gestion des documents par le Kit de développement logiciel (SDK) .NET
Vous vous demandez peut-être comment le SDK .NET Recherche cognitive Azure peut charger des instances d’une classe définie par l'utilisateur, comme `Hotel`, dans l'index. Pour répondre à cette question, examinons la classe `Hotel` :

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;
using Newtonsoft.Json;

public partial class Hotel
{
    [System.ComponentModel.DataAnnotations.Key]
    [IsFilterable]
    public string HotelId { get; set; }

    [IsSearchable, IsSortable]
    public string HotelName { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.EnLucene)]
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

    // SmokingAllowed reflects whether any room in the hotel allows smoking.
    // The JsonIgnore attribute indicates that a field should not be created 
    // in the index for this property and it will only be used by code in the client.
    [JsonIgnore]
    public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;

    [IsFilterable, IsSortable, IsFacetable]
    public DateTimeOffset? LastRenovationDate { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public double? Rating { get; set; }

    public Address Address { get; set; }

    [IsFilterable, IsSortable]
    public GeographyPoint Location { get; set; }

    public Room[] Rooms { get; set; }
}
```

La première chose à remarquer est que le nom de chaque propriété publique dans la classe `Hotel` mappe à un champ portant le même nom dans la définition d’index. Si vous souhaitez que chaque champ commence par une lettre minuscule (« casse mixte »), vous pouvez demander au Kit de développement logiciel (SDK) de mapper les noms de propriété à une casse mixte automatiquement avec l’attribut `[SerializePropertyNamesAsCamelCase]` sur la classe. Ce scénario est courant dans les applications .NET qui effectuent des liaisons de données, où le schéma cible échappe au contrôle du développeur de l’application, sans devoir violer les directives d’affectation de noms en « casse Pascal » dans .NET.

> [!NOTE]
> Le SDK .NET Recherche cognitive Azure utilise la bibliothèque [NewtonSoft JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm) pour sérialiser et désérialiser vos objets de modèle personnalisés vers et à partir de JSON. Vous pouvez personnaliser cette sérialisation si nécessaire. Pour plus d’informations, voir [Sérialisation personnalisée avec JSON.NET](#JsonDotNet).
> 
> 

La deuxième chose à remarquer est que chaque propriété est assortie d’attributs tels que `IsFilterable`, `IsSearchable`, `Key` et `Analyzer`. Ces attributs sont mappés directement aux [attributs de champ correspondants dans un index de Recherche cognitive Azure](/rest/api/searchservice/create-index). La classe `FieldBuilder` utilise ces propriétés pour construire des définitions de champ pour l’index.

La troisième chose importante concernant la classe `Hotel` a trait aux types de données des propriétés publiques. Les types .NET de ces propriétés correspondent à leurs types de champ équivalents dans la définition de l'index. Par exemple, la propriété de chaîne `Category` correspond au champ `category`, qui est de type `Edm.String`. Il existe des mappages de type similaire entre `bool?`, `Edm.Boolean`, `DateTimeOffset?` et `Edm.DateTimeOffset`, etc. Les règles spécifiques pour le mappage de type sont documentées avec la méthode `Documents.Get` dans [l’article de référence sur le Kit de développement logiciel (SDK) .NET du service Recherche cognitive Azure](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.get). La classe `FieldBuilder` effectue ce mappage pour vous, mais il peut toutefois être utile de comprendre son fonctionnement, pour les situations éventuelles de résolution des problèmes de sérialisation.

Avez-vous remarqué la propriété `SmokingAllowed` ?

```csharp
[JsonIgnore]
public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;
```

L’attribut `JsonIgnore` de cette propriété indique au `FieldBuilder` de ne pas la sérialiser sur l’index en tant que champ.  C’est un excellent moyen de créer des propriétés calculées côté client que vous pouvez utiliser comme aides dans votre application.  Dans ce cas, la propriété `SmokingAllowed` indique s’il est permis de fumer dans l’une des `Room` de la collection `Rooms`.  Si la valeur est false partout, cela signifie qu’il est interdit de fumer dans tout l’hôtel.

Certaines propriétés, telles que `Address` et `Rooms` sont des instances de classes .NET.  Ces propriétés représentent des structures de données plus complexes et, par conséquent, nécessitent des champs avec un [type de données complexe](./search-howto-complex-data-types.md) dans l’index.

La propriété `Address` représente un ensemble de plusieurs valeurs dans la classe `Address` définie ci-dessous :

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Newtonsoft.Json;

namespace AzureSearch.SDKHowTo
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

Cette classe contient les valeurs standards utilisées pour décrire les adresses aux États-Unis ou au Canada. Vous pouvez utiliser des types comme celui-ci pour regrouper des champs logiques dans l’index.

La propriété `Rooms` représente une série d’objets `Room` :

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Newtonsoft.Json;

namespace AzureSearch.SDKHowTo
{
    public partial class Room
    {
        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.EnMicrosoft)]
        public string Description { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.FrMicrosoft)]
        [JsonProperty("Description_fr")]
        public string DescriptionFr { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string Type { get; set; }

        [IsFilterable, IsFacetable]
        public double? BaseRate { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string BedOptions { get; set; }

        [IsFilterable, IsFacetable]
        public int SleepsCount { get; set; }

        [IsFilterable, IsFacetable]
        public bool? SmokingAllowed { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string[] Tags { get; set; }
    }
}
```

Votre modèle de données dans .NET et le schéma d’index qui lui correspond doivent être conçus pour prendre en charge l’expérience de recherche que vous souhaitez offrir à vos utilisateurs finaux. Chaque objet de niveau supérieur dans .NET, c’est-à-dire chaque document dans l’index, correspond à un résultat de recherche qui sera présenté dans votre interface utilisateur. Par exemple, dans une application de recherche d’hôtel, vos utilisateurs peuvent rechercher par nom d’hôtel, caractéristiques d’hôtel, ou caractéristiques de chambres spécifiques. Nous examinerons quelques exemples de requête un peu plus tard.

Cette capacité à utiliser vos propres classes pour interagir avec des documents dans l’index fonctionne dans les deux sens. Vous pouvez également récupérer les résultats de la recherche et laisser le Kit de développement logiciel (SDK) les désérialiser automatiquement à un type de votre choix, comme nous le verrons dans la section suivante.

> [!NOTE]
> Le SDK .NET Recherche cognitive Azure prend également en charge les documents dynamiquement typés à l’aide de la classe `Document`, qui est un mappage de type clé/valeur entre des noms de champ et des valeurs de champ. Cela est utile dans les cas où vous ne connaissez pas le schéma de l’index lors de sa conception et où il serait peu pratique d’établir une liaison à des classes de modèles spécifiques. Toutes les méthodes du SDK qui gèrent les documents ont des surcharges qui fonctionnent avec la classe `Document` , ainsi que des surcharges fortement typées qui acceptent un paramètre de type générique. Seules ces dernières sont utilisées dans l'exemple de code de ce didacticiel. La classe [`Document`](/dotnet/api/microsoft.azure.search.models.document) hérite de `Dictionary<string, object>`.
> 
>

**Pourquoi utiliser des types de données Nullable**

Lorsque vous créez vos propres classes de modèles à mapper à un index Recherche cognitive Azure, nous vous recommandons de déclarer les propriétés des types de valeurs, par exemple `bool` et `int`, comme acceptant la valeur null (par exemple, `bool?` au lieu de `bool`). Si vous utilisez une propriété ne pouvant être définie sur null, vous devez **garantir** qu’aucun document de cet index ne contient de valeur null pour le champ correspondant. Ni le Kit de développement logiciel ni le service Recherche cognitive Azure ne vous aideront à appliquer cette recommandation.

Il ne s’agit pas d’une préoccupation hypothétique : imaginez un scénario dans lequel vous ajoutez un nouveau champ à un index existant qui est de type `Edm.Int32`. Après la mise à jour de la définition d’index, ce nouveau champ prendra la valeur null pour tous les documents (car tous les types peuvent avoir la valeur null dans Recherche cognitive Azure). Si vous utilisez ensuite une classe de modèle avec une propriété `int` ne pouvant être définie sur null pour ce champ, vous obtiendrez l’exception `JsonSerializationException` ci-dessous lorsque vous tenterez de récupérer des documents :

```output
Error converting value {null} to type 'System.Int32'. Path 'IntValue'.
```

Pour cette raison, nous vous recommandons d'utiliser des types pour lesquels la valeur null est autorisée en tant que meilleure pratique.

<a name="JsonDotNet"></a>

#### <a name="custom-serialization-with-jsonnet"></a>Sérialisation personnalisée avec JSON.NET
Le kit de développement logiciel utilise JSON.NET pour sérialiser et désérialiser les documents. Si nécessaire, vous pouvez personnaliser la sérialisation et la désérialisation en définissant votre propre `JsonConverter` ou `IContractResolver`. Pour plus d’informations, voir la [documentation sur JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm). Cela peut s’avérer utile lorsque vous souhaitez adapter une classe de modèle existante de votre application à utiliser avec Recherche cognitive Azure et d’autres scénarios plus avancés. Par exemple, avec la sérialisation personnalisée, vous pouvez :

* Incluez ou exclure certaines propriétés de votre classe de modèle dans le stockage en tant que champs de document.
* Mappez des noms de propriété dans le code et des noms de champ de votre index.
* Créez des attributs personnalisés qui peuvent être utilisés pour le mappage des propriétés aux champs du document.

Vous pouvez trouver des exemples d’implémentation de sérialisation personnalisée dans les tests d’unités du kit de développement logiciel .NET Recherche cognitive Azure sur GitHub. [Ce dossier](https://github.com/Azure/azure-sdk-for-net/tree/4f6f4e4c90200c1b0621c4cead302a91e89f2aba/sdk/search/Microsoft.Azure.Search/tests/Tests/Models) est un bon point de départ. Il contient des classes qui sont utilisées par les tests de sérialisation personnalisés.

### <a name="searching-for-documents-in-the-index"></a>Recherche de documents dans l'index
La dernière étape dans l’exemple d’application consiste à rechercher certains documents dans l’index :

```csharp
private static void RunQueries(ISearchIndexClient indexClient)
{
    SearchParameters parameters;
    DocumentSearchResult<Hotel> results;

    Console.WriteLine("Search the entire index for the term 'motel' and return only the HotelName field:\n");

    parameters =
        new SearchParameters()
        {
            Select = new[] { "HotelName" }
        };

    results = indexClient.Documents.Search<Hotel>("motel", parameters);

    WriteDocuments(results);

    Console.Write("Apply a filter to the index to find hotels with a room cheaper than $100 per night, ");
    Console.WriteLine("and return the hotelId and description:\n");

    parameters =
        new SearchParameters()
        {
            Filter = "Rooms/any(r: r/BaseRate lt 100)",
            Select = new[] { "HotelId", "Description" }
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.Write("Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    parameters =
        new SearchParameters()
        {
            OrderBy = new[] { "LastRenovationDate desc" },
            Select = new[] { "HotelName", "LastRenovationDate" },
            Top = 2
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.WriteLine("Search the entire index for the term 'hotel':\n");

    parameters = new SearchParameters();
    results = indexClient.Documents.Search<Hotel>("hotel", parameters);

    WriteDocuments(results);
}
```

Chaque fois qu’elle exécute une requête, cette méthode crée tout d’abord un objet `SearchParameters`. Cet objet permet de spécifier des options supplémentaires pour la requête, comme le tri, le filtrage, la pagination et la génération de facettes. Dans cette méthode, nous définissons les propriétés `Filter`, `Select`, `OrderBy`, et `Top` pour différentes requêtes. Toutes les propriétés `SearchParameters` sont documentées [ici](/dotnet/api/microsoft.azure.search.models.searchparameters).

L'étape suivante consiste à exécuter la requête de recherche. La recherche est exécutée à l’aide de la méthode `Documents.Search`. Pour chaque requête, nous transmettons le texte de recherche à utiliser en tant que chaîne (ou l’élément `"*"`, s’il n’en existe aucun), ainsi que les paramètres de recherche créés précédemment. Nous spécifions également `Hotel` comme paramètre de type pour `Documents.Search`, qui demande au SDK de désérialiser les documents figurant dans les résultats de recherche, en objets de type `Hotel`.

> [!NOTE]
> Pour plus d’informations sur la syntaxe des requêtes de recherche, cliquez [ici](/rest/api/searchservice/Simple-query-syntax-in-Azure-Search).
> 
> 

Enfin, après l’exécution de chaque requête, cette méthode parcourt toutes les correspondances dans les résultats de la recherche et imprime chaque document dans la console :

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

Examinons de plus près chacune des requêtes exécutées. Voici le code permettant d’exécuter la première requête :

```csharp
parameters =
    new SearchParameters()
    {
        Select = new[] { "HotelName" }
    };

results = indexClient.Documents.Search<Hotel>("motel", parameters);

WriteDocuments(results);
```

Dans ce cas, nous recherchons dans l’index entier le mot « motel », dans tout champ pouvant faire l’objet d’une recherche, et ne voulons récupérer que les noms d’hôtel, comme spécifié par le paramètre `Select`. Voici les résultats :

```output
Name: Secret Point Motel

Name: Twin Dome Motel
```

La requête suivante est un peu plus intéressante.  Nous souhaitons trouver tous les hôtels proposant une chambre à moins de 100 $ la nuitée, et obtenir uniquement l’ID et la description des hôtels :

```csharp
parameters =
    new SearchParameters()
    {
        Filter = "Rooms/any(r: r/BaseRate lt 100)",
        Select = new[] { "HotelId", "Description" }
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

Cette requête utilise une expression `$filter` OData (`Rooms/any(r: r/BaseRate lt 100)`) pour filtrer les documents dans l’index. Cet exemple utilise l’[opérateur any](./search-query-odata-collection-operators.md) pour appliquer le « BaseRate lt 100 » à chaque élément de la collection Rooms (Chambres). Pour plus d’informations sur la syntaxe OData prise en charge par Recherche cognitive Azure, cliquez [ici](./query-odata-filter-orderby-syntax.md).

Voici les résultats de la requête :

```output
HotelId: 1
Description: The hotel is ideally located on the main commercial artery of the city in the heart of New York...

HotelId: 2
Description: The hotel is situated in a nineteenth century plaza, which has been expanded and renovated to...
```

Ensuite, nous souhaitons rechercher les deux hôtels ayant été le plus récemment rénovés, et afficher le nom et la date de la dernière rénovation de ces derniers. Voici le code : 

```csharp
parameters =
    new SearchParameters()
    {
        OrderBy = new[] { "LastRenovationDate desc" },
        Select = new[] { "HotelName", "LastRenovationDate" },
        Top = 2
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

Dans ce cas, nous utilisons à nouveau la syntaxe OData pour spécifier le paramètre `OrderBy` en tant que `lastRenovationDate desc`. Nous définissons également le paramètre `Top` sur 2 pour vérifier que nous obtenons uniquement les deux premiers documents. Comme précédemment, nous définissons le paramètre `Select` pour spécifier les champs qui doivent être renvoyés.

Voici les résultats :

```output
Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00
```

Enfin, nous souhaitons trouver tous les noms d’hôtel contenant le mot « hotel » :

```csharp
parameters = new SearchParameters()
{
    SearchFields = new[] { "HotelName" }
};
results = indexClient.Documents.Search<Hotel>("hotel", parameters);

WriteDocuments(results);
```

Voici les résultats, qui incluent tous les champs, dans la mesure où nous n’avons pas spécifié la propriété `Select` :

```output
    HotelId: 3
    Name: Triple Landscape Hotel
    ...
```

Cette étape termine le didacticiel, mais ne vous arrêtez pas en si bon chemin. Les étapes suivantes fournissent des ressources supplémentaires pour en apprendre davantage sur Recherche cognitive Azure.

## <a name="next-steps"></a>Étapes suivantes
* Parcourez les références relatives au [Kit de développement logiciel (SDK) .NET](/dotnet/api/microsoft.azure.search) et à [l’API REST](/rest/api/searchservice/).
* Consultez les [conventions d’affectation de noms](/rest/api/searchservice/Naming-rules) pour apprendre les règles de dénomination des différents objets.
* Faites connaissance avec les [types de données pris en charge](/rest/api/searchservice/Supported-data-types) par Recherche cognitive Azure.