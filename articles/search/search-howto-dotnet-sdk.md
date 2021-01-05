---
title: Utiliser Azure.Search.Documents (v11) dans .NET
titleSuffix: Azure Cognitive Search
description: Découvrez comment créer et gérer des objets de recherche dans une application .NET à l’aide de C# et de la bibliothèque de client Azure.Search.Documents (v11). Les extraits de code montrent comment se connecter au service et créer des index et des requêtes.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.devlang: dotnet
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 3ceead297ea726e256d806c08c22810b39296793
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94917169"
---
# <a name="how-to-use-azuresearchdocuments-in-a-c-net-application"></a>Utilisation d’Azure.Search.Documents dans une application .NET C#

Cet article explique comment créer et gérer des objets de recherche en utilisant C# et la bibliothèque de client [**Azure.Search.Documents**](/dotnet/api/overview/azure/search) (version 11).

## <a name="about-version-11"></a>À propos de la version 11

Le Kit SDK Azure pour .NET ajoute une nouvelle bibliothèque de client [**Azure.Search.Documents**](/dotnet/api/overview/azure/search) à partir de l’équipe du Kit SDK Azure dont les fonctionnalités sont équivalentes aux bibliothèques de client [Microsoft.Azure.Search](/dotnet/api/overview/azure/search/client10), mais qui utilise des approches et des conventions communes lorsque c’est possible. Certains exemples incluent l’authentification de clé [`AzureKeyCredential`](/dotnet/api/azure.azurekeycredential) et la sérialisation [System.Text.Json.Serialization](/dotnet/api/system.text.json.serialization) pour JSON.

Comme avec les versions précédentes, vous pouvez utiliser cette bibliothèque pour :

+ Créer et gérer des index de recherche, des sources de données, des indexeurs, des ensembles de compétences et des mappages de synonymes
+ Charger et gérer des documents de recherche dans un index
+ Exécuter des requêtes, tout cela sans avoir à gérer les détails de HTTP et JSON

La bibliothèque est distribuée sous forme de [package NuGet Azure.Search.Document](https://www.nuget.org/packages/Azure.Search.Documents/) unique, qui comprend toutes les API utilisées pour l’accès par programme à un service de recherche.

La bibliothèque cliente définit des classes comme `SearchIndex`, `SearchField` et `SearchDocument`, ainsi que des opérations telles que `SearchIndexClient.CreateIndex` et `SearchClient.Search` sur les classes `SearchIndexClient` et `SearchClient`. Ces classes sont organisées dans les espaces de noms suivants :

+ [`Azure.Search.Documents`](/dotnet/api/azure.search.documents)
+ [`Azure.Search.Documents.Indexes`](/dotnet/api/azure.search.documents.indexes)
+ [`Azure.Search.Documents.Indexes.Models`](/dotnet/api/azure.search.documents.indexes.models)
+ [`Azure.Search.Documents.Models`](/dotnet/api/azure.search.documents.models)

Azure.Search.Documents (version 11) cible la version [`2020-06-30` de l’API REST Recherche cognitive Azure](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/search/data-plane/Azure.Search/preview/2020-06-30). 

La bibliothèque de client ne fournit pas d’[opérations de management des services](/rest/api/searchmanagement/) telles que la création et la mise à l’échelle des services de recherche ainsi que la gestion des clés API. Si vous avez besoin de gérer vos ressources de recherche à partir d’une application .NET, utilisez la bibliothèque [Microsoft.Azure.Management.Search](/dotnet/api/overview/azure/search/management) du Kit SDK Azure pour .NET.

## <a name="upgrade-to-v11"></a>Mettre à niveau vers la version 11

Si vous utilisez une version antérieure du SDK .NET et que vous souhaitez mettre à niveau vers la version généralement disponible, consultez [Mettre à niveau vers la version 11 du Kit de développement logiciel (SDK) .NET Recherche cognitive Azure](search-dotnet-sdk-migration-version-11.md)

## <a name="sdk-requirements"></a>Configuration requise du kit de développement logiciel (SDK)

+ Visual Studio 2019 ou version ultérieure.

+ Votre propre service Recherche cognitive Azure. Pour utiliser le SDK, vous devez connaître le nom de votre service et une ou plusieurs clés API. [Créez un service dans le portail](search-create-service-portal.md) si vous n’en avez pas.

+ Téléchargez le [package Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents) via **Outils** > **Gestionnaire des packages NuGet** > **Gérer les packages NuGet pour la solution** dans Visual Studio. Recherchez le package nommé `Azure.Search.Documents`.

Le Kit SDK Azure pour .NET est conforme à [.NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support), ce qui signifie .NET Framework 4.6.1 et .NET Core 2.0 comme configuration requise minimale.

## <a name="example-application"></a>Exemple d’application

Cet article « enseigne par l’exemple » en s’appuyant sur l’exemple de code [DotNetHowTo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo) sur GitHub pour illustrer les concepts fondamentaux de Recherche cognitive Azure, à savoir la création, le chargement et l’interrogation d’un index de recherche.

Pour le reste de l’article, supposons un nouvel index nommé « hotels », comportant quelques documents, avec plusieurs requêtes qui correspondent à des résultats.

Voici le programme principal, qui décrit le flux global :

```csharp
// This sample shows how to delete, create, upload documents and query an index
static void Main(string[] args)
{
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    SearchIndexClient indexClient = CreateSearchIndexClient(configuration);

    string indexName = configuration["SearchIndexName"];

    Console.WriteLine("{0}", "Deleting index...\n");
    DeleteIndexIfExists(indexName, indexClient);

    Console.WriteLine("{0}", "Creating index...\n");
    CreateIndex(indexName, indexClient);

    SearchClient searchClient = indexClient.GetSearchClient(indexName);

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(searchClient);

    SearchClient indexClientForQueries = CreateSearchClientForQueries(indexName, configuration);

    Console.WriteLine("{0}", "Run queries...\n");
    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

Voici une capture d’écran partielle de la sortie, en supposant que vous exécutez cette application avec un nom de service et des clés API valides :

:::image type="content" source="media/search-howto-dotnet-sdk/console-output.png" alt-text="Sortie Console.WriteLine de l’exemple de programme":::

### <a name="client-types"></a>Types de clients

La bibliothèque de client utilise trois types de client pour diverses opérations : [`SearchIndexClient`](/dotnet/api/azure.search.documents.indexes.searchindexclient) pour créer, mettre à jour ou supprimer des index, [`SearchClient`](/dotnet/api/azure.search.documents.searchclient) pour charger ou interroger un index, et [`SearchIndexerClient`](/dotnet/api/azure.search.documents.indexes.searchindexerclient) pour utiliser des indexeurs et des ensembles de compétences. Cet article est consacré aux deux premières. 

Au minimum, tous les clients exigent le nom ou le point de terminaison du service ainsi qu’une clé API. Ces informations sont généralement fournies dans un fichier de configuration, semblable au fichier `appsettings.json` de l’[exemple d’application DotNetHowTo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo). Pour lire à partir du fichier de configuration, ajoutez `using Microsoft.Extensions.Configuration;` à votre programme.

L’instruction suivante crée le client d’index utilisé pour créer, mettre à jour ou supprimer des index. Elle utilise un point de terminaison de recherche et une clé API d’administrateur.

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
    string searchServiceEndPoint = configuration["SearchServiceEndPoint"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(new Uri(searchServiceEndPoint), new AzureKeyCredential(adminApiKey));
    return indexClient;
}
```

L’instruction suivant crée le client de recherche utilisé pour charger des documents ou exécuter des requêtes. `SearchClient` nécessite un index. Vous aurez besoin d’une clé API d’administrateur pour charger des documents, mais vous pouvez utiliser une clé API de requête pour exécuter des requêtes. 

```csharp
string indexName = configuration["SearchIndexName"];

private static SearchClient CreateSearchClientForQueries(string indexName, IConfigurationRoot configuration)
{
    string searchServiceEndPoint = configuration["SearchServiceEndPoint"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchClient searchClient = new SearchClient(new Uri(searchServiceEndPoint), indexName, new AzureKeyCredential(queryApiKey));
    return searchClient;
}
```

> [!NOTE]
> Si vous fournissez une clé non valide pour l’opération d’importation (par exemple, une clé de requête là où une clé d’administration était demandée), `SearchClient` génère une `CloudException` avec le message d’erreur « Forbidden » la première fois que vous invoquez une méthode d’opération dessus. Si cette situation se produit, vérifiez la clé API.
>

### <a name="deleting-the-index"></a>Suppression de l’index

Dans les premières étapes du développement, vous souhaiterez peut-être inclure une instruction [`DeleteIndex`](/dotnet/api/azure.search.documents.indexes.searchindexclient.deleteindex) pour supprimer un index de transition afin de pouvoir le recréer avec une définition mise à jour. L’exemple de code pour Recherche cognitive Azure comporte souvent une étape de suppression qui vous permet de réexécuter l’exemple.

La ligne suivante appelle `DeleteIndexIfExists` :

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteIndexIfExists(indexName, indexClient);
```

Cette méthode utilise l’élément `SearchIndexClient` donné pour vérifier si l’index existe et, dans l’affirmative, elle le supprime :

```csharp
private static void DeleteIndexIfExists(string indexName, SearchIndexClient indexClient)
{
    try
    {
        if (indexClient.GetIndex(indexName) != null)
        {
            indexClient.DeleteIndex(indexName);
        }
    }
    catch (RequestFailedException e) when (e.Status == 404)
    {
        // Throw an exception if the index name isn't found
        Console.WriteLine("The index doesn't exist. No deletion occurred.");
```

> [!NOTE]
> L’exemple de code de cet article utilise des méthodes synchrones pour des raisons de simplicité, mais nous vous recommandons d’utiliser des méthodes asynchrones dans vos applications pour qu’elles restent évolutives et réactives. Par exemple, dans la méthode ci-dessus, vous pouvez utiliser [`DeleteIndexAsync`](/dotnet/api/azure.search.documents.indexes.searchindexclient.deleteindexasync) au lieu de [`DeleteIndex`](/dotnet/api/azure.search.documents.indexes.searchindexclient.deleteindex).
>

## <a name="create-an-index"></a>Création d'un index

Vous pouvez utiliser [`SearchIndexClient`](/dotnet/api/azure.search.documents.indexes.searchindexclient) pour créer un index. 

La méthode ci-dessous crée un objet [`SearchIndex`](/dotnet/api/azure.search.documents.indexes.models.searchindex) avec une liste d’objets [`SearchField`](/dotnet/api/azure.search.documents.indexes.models.searchfield) qui définit le schéma du nouvel index. Chaque champ a un nom, un type de données et plusieurs attributs qui définissent son comportement de recherche. 

Les champs peuvent être définis à partir d’une classe de modèle à l’aide de [`FieldBuilder`](/dotnet/api/azure.search.documents.indexes.fieldbuilder). La classe `FieldBuilder` utilise la réflexion pour créer une liste d’objets `SearchField` pour l’index en examinant les attributs et propriétés publics de la classe de modèle `Hotel` donnée. Nous examinerons la classe `Hotel` plus en détail un peu plus tard.

```csharp
private static void CreateIndex(string indexName, SearchIndexClient indexClient)
{
    FieldBuilder fieldBuilder = new FieldBuilder();
    var searchFields = fieldBuilder.Build(typeof(Hotel));

    var definition = new SearchIndex(indexName, searchFields);

    indexClient.CreateOrUpdateIndex(definition);
}
```

En plus des champs, vous pouvez ajouter des profils de notation, des générateurs de suggestions ou des options CORS à l’index (ces paramètres sont omis de l’exemple par souci de concision). Vous trouverez plus d’informations sur l’objet SearchIndex et ses composants dans la liste des propriétés [`SearchIndex`](/dotnet/api/azure.search.documents.indexes.models.searchindex) ainsi que dans la page de [référence sur l’API REST](/rest/api/searchservice/).

> [!NOTE]
> Vous pouvez toujours créer directement la liste des objets `Field` au lieu d’utiliser la fonctionnalité `FieldBuilder`, si nécessaire. Par exemple, vous ne souhaitez pas utiliser une classe de modèle, ou vous pouvez être amené à recourir à une classe de modèle existante, que vous ne souhaitez pas modifier en ajoutant des attributs.
>

### <a name="call-createindex-in-main"></a>Appeler CreateIndex dans Main()

`Main` crée un index « hotels » en appelant la méthode ci-dessus :

```csharp
Console.WriteLine("{0}", "Creating index...\n");
CreateIndex(indexName, indexClient);
```

## <a name="use-a-model-class-for-data-representation"></a>Utiliser une classe de modèle pour la représentation des données

L’exemple DotNetHowTo utilise des classes de modèle pour les structures de données [Hotel](https://github.com/Azure-Samples/search-dotnet-getting-started/blob/master/DotNetHowTo/DotNetHowTo/Hotel.cs), [Address](https://github.com/Azure-Samples/search-dotnet-getting-started/blob/master/DotNetHowTo/DotNetHowTo/Address.cs) et [Room](https://github.com/Azure-Samples/search-dotnet-getting-started/blob/master/DotNetHowTo/DotNetHowTo/Room.cs). `Hotel` fait référence à `Address`, un type complexe à un niveau (un champ à plusieurs parties) et `Room` (une collection de champs à plusieurs parties).

Vous pouvez utiliser ces types pour créer et charger l’index ainsi que pour structurer la réponse à partir d’une requête :

```csharp
// Use-case: <Hotel> in a field definition
FieldBuilder fieldBuilder = new FieldBuilder();
var searchFields = fieldBuilder.Build(typeof(Hotel));

// Use-case: <Hotel> in a response
private static void WriteDocuments(SearchResults<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.GetResults())
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

Une autre approche consiste à ajouter directement des champs à un index. L'exemple suivant ne montre que quelques champs.

   ```csharp
    SearchIndex index = new SearchIndex(indexName)
    {
        Fields =
            {
                new SimpleField("hotelId", SearchFieldDataType.String) { IsKey = true, IsFilterable = true, IsSortable = true },
                new SearchableField("hotelName") { IsFilterable = true, IsSortable = true },
                new SearchableField("hotelCategory") { IsFilterable = true, IsSortable = true },
                new SimpleField("baseRate", SearchFieldDataType.Int32) { IsFilterable = true, IsSortable = true },
                new SimpleField("lastRenovationDate", SearchFieldDataType.DateTimeOffset) { IsFilterable = true, IsSortable = true }
            }
    };
   ```

### <a name="field-definitions"></a>Définitions de champs

Votre modèle de données dans .NET et le schéma d’index qui lui correspond doivent prendre en charge l’expérience de recherche que vous souhaitez offrir à vos utilisateurs finaux. Chaque objet de niveau supérieur dans .NET, comme la recherche de document dans un index de recherche, correspond à un résultat de recherche qui sera présenté dans votre interface utilisateur. Par exemple, dans une application de recherche d’hôtel, vos utilisateurs peuvent rechercher par nom d’hôtel, caractéristiques d’hôtel, ou caractéristiques de chambres spécifiques. 

Dans chaque classe, un champ est défini avec un type de données et des attributs qui déterminent la façon dont il est utilisé. Le nom de chaque propriété publique de chaque classe correspond à un champ portant le même nom dans la définition d’index. 

Observez l’extrait de code suivant, qui extrait plusieurs définitions de champs à partir de la classe Hotel. Notez que Address et Rooms sont des types C# ayant leurs propres définitions de classe (consultez l’exemple de code pour les afficher). Il s’agit de types complexes. Pour plus d’informations, consultez [How to model complex types](search-howto-complex-data-types.md) (Modélisation des types complexes).

```csharp
public partial class Hotel
{
    [SimpleField(IsKey = true, IsFilterable = true)]
    public string HotelId { get; set; }

    [SearchableField(IsSortable = true)]
    public string HotelName { get; set; }

    [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnLucene)]
    public string Description { get; set; }

    [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
    public string Category { get; set; }

    [JsonIgnore]
    public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;

    [SearchableField]
    public Address Address { get; set; }

    public Room[] Rooms { get; set; }
```

#### <a name="choosing-a-field-class"></a>Choix d’une classe de champ

Lors de la définition de champs, vous pouvez utiliser la classe [`SearchField`](/dotnet/api/azure.search.documents.indexes.models.searchfield) de base ou bien des modèles d’assistance dérivés qui servent de « modèles » avec des propriétés préconfigurées.

Un seul champ de votre index doit servir de clé de document (`IsKey = true`). Il doit s’agir d’une chaîne qui identifie chaque document de manière unique. Il est également nécessaire d’avoir `IsHidden = true`, ce qui signifie qu’il ne peut pas être visible dans les résultats de recherche.

| Type de champ | Description et utilisation |
|------------|-----------------------|
| [`SearchField`](/dotnet/api/azure.search.documents.indexes.models.searchfield) | Classe de base, avec la plupart des propriétés définies sur null, à l’exception de `Name`, qui est obligatoire, et de `AnalyzerName` dont la valeur par défaut est Lucene standard. |
| [`SimpleField`](/dotnet/api/azure.search.documents.indexes.models.simplefield) | Modèle d’assistance. Peut être n’importe quel type de données, ne peut jamais faire l’objet d’une recherche (il est ignoré pour les requêtes de recherche de texte intégral) et peut être récupéré (il n’est pas masqué). Les autres attributs sont désactivés par défaut, mais peuvent être activés. Vous pouvez utiliser un `SimpleField` pour les ID de document ou les champs utilisés seulement dans des filtres, des facettes ou des profils de scoring. Dans ce cas, veillez à appliquer tous les attributs nécessaires pour le scénario, comme `IsKey = true` pour un ID de document. Pour plus d’informations, consultez [SimpleFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SimpleFieldAttribute.cs) dans le code source. |
| [`SearchableField`](/dotnet/api/azure.search.documents.indexes.models.searchablefield) | Modèle d’assistance. Doit être une chaîne, et peut toujours faire l’objet d’une recherche et d’une récupération. Les autres attributs sont désactivés par défaut, mais peuvent être activés. Comme ce type de champ peut faire l’objet d’une recherche, il prend en charge les synonymes et l’ensemble complet des propriétés de l’analyseur. Pour plus d’informations, consultez [SearchableFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SearchableFieldAttribute.cs) dans le code source. |

Que vous utilisiez l’API `SearchField` de base ou un des modèles d’assistance, vous devez activer explicitement les attributs de filtre, de facette et de tri. Par exemple, les attributs [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable), [IsSortable](/dotnet/api/azure.search.documents.indexes.models.searchfield.issortable) et [IsFacetable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfacetable) doivent être explicitement attribués, comme illustré dans l’exemple ci-dessus.

#### <a name="adding-field-attributes"></a>Ajout d’attributs de champ

Notez que chaque champ comporte des attributs tels que `IsFilterable`, `IsSortable`, `IsKey` et `AnalyzerName`. Ces attributs sont mappés directement aux [attributs de champ correspondants dans un index de Recherche cognitive Azure](/rest/api/searchservice/create-index). La classe `FieldBuilder` utilise ces propriétés pour construire des définitions de champ pour l’index.

#### <a name="field-type-mapping"></a>Mappage de types de champ

Les types .NET des propriétés correspondent à leurs types de champ équivalents dans la définition d’index. Par exemple, la propriété de chaîne `Category` correspond au champ `category`, qui est de type `Edm.String`. Il existe des mappages de type similaire entre `bool?`, `Edm.Boolean`, `DateTimeOffset?` et `Edm.DateTimeOffset`, etc. 

Avez-vous remarqué la propriété `SmokingAllowed` ?

```csharp
[JsonIgnore]
public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;
```

L’attribut `JsonIgnore` de cette propriété indique au `FieldBuilder` de ne pas la sérialiser sur l’index en tant que champ.  C’est un excellent moyen de créer des propriétés calculées côté client que vous pouvez utiliser comme aides dans votre application.  Dans ce cas, la propriété `SmokingAllowed` indique s’il est permis de fumer dans l’une des `Room` de la collection `Rooms`. Si la valeur est false partout, cela signifie qu’il est interdit de fumer dans tout l’hôtel.

## <a name="load-an-index"></a>Charger un index

La prochaine étape dans `Main` remplit l’index nouvellement créé « hotels ». Ce remplissage d’index s’effectue dans la méthode suivante : (Une partir du code a été remplacée par « ... » pour l’illustration. Pour le code complet de remplissage des données, voir l’exemple complet de la solution.)

```csharp
private static void UploadDocuments(SearchClient searchClient)
{
    IndexDocumentsBatch<Hotel> batch = IndexDocumentsBatch.Create(
        IndexDocumentsAction.Upload(
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
            }),
        IndexDocumentsAction.Upload(
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
            }),
        IndexDocumentsAction.Upload(
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

    try
    {
        IndexDocumentsResult result = searchClient.IndexDocuments(batch);
    }
    catch (Exception)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine("Failed to index some of the documents: {0}");
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
```

Cette méthode présente quatre parties. La première crée un tableau de 3 objets `Hotel`, comprenant chacun 3 objets `Room`, qui servent de données en entrée à charger dans l’index. Ces données sont codées en dur pour plus de simplicité. Dans votre application réelle, les données seront probablement issues d’une source de données externe, comme une base de données SQL.

La deuxième partie crée un [`IndexDocumentsBatch`](/dotnet/api/azure.search.documents.models.indexdocumentsbatch) contenant les documents. Vous spécifiez l’opération que vous souhaitez appliquer au lot au moment de sa création, dans ce cas en appelant [`IndexDocumentsAction.Upload`](/dotnet/api/azure.search.documents.models.indexdocumentsaction.upload). Le lot est ensuite chargé dans l’index Recherche cognitive Azure par la méthode [`IndexDocuments`](/dotnet/api/azure.search.documents.searchclient.indexdocuments).

> [!NOTE]
> Dans cet exemple, nous allons simplement charger les documents. Si vous souhaitez fusionner les modifications dans les documents existants ou supprimer des documents, vous pouvez créer des lots en appelant `IndexDocumentsAction.Merge`, `IndexDocumentsAction.MergeOrUpload` ou `IndexDocumentsAction.Delete` à la place. Vous pouvez aussi combiner plusieurs opérations dans un lot unique en appelant `IndexBatch.New`, qui accepte une collection d’objets `IndexDocumentsAction`, dont chacun indique à Recherche cognitive Azure d’effectuer une opération spécifique sur un document. Vous pouvez créer chaque `IndexDocumentsAction` avec sa propre opération en appelant la méthode correspondante comme `IndexDocumentsAction.Merge`, `IndexAction.Upload`, et ainsi de suite.
> 

La troisième partie de cette méthode est un bloc catch qui gère un cas d'erreur important pour l'indexation. Si votre service de recherche ne parvient pas à indexer certains documents du lot, `IndexDocuments` génère un `IndexBatchException`. Cette exception peut se produire si vous indexez des documents lorsque votre service est surchargé. **Nous vous recommandons vivement de prendre en charge explicitement ce cas de figure dans votre code.** Vous pouvez retarder puis relancer l'indexation des documents qui ont échoué, ouvrir une session et continuer comme dans l’exemple, ou faire autre chose selon la cohérence des données requise par votre application.

Enfin, la méthode `UploadDocuments` retarde son exécution de deux secondes. L’indexation s’exécutant en mode asynchrone dans votre service de recherche, l’exemple d’application doit attendre quelque temps afin de s’assurer que les documents sont disponibles pour la recherche. Ce genre de retard n’est nécessaire que dans les démonstrations, les tests et les exemples d'applications.

### <a name="call-uploaddocuments-in-main"></a>Appeler UploadDocuments dans Main()

L’extrait de code suivant configure une instance de [`SearchClient`](/dotnet/api/azure.search.documents.searchclient) à l’aide de la méthode [`GetSearchClient`](/dotnet/api/azure.search.documents.indexes.searchindexclient.getsearchclient) de indexClient. indexClient utilise une clé API d’administrateur sur ses requêtes, ce qui est nécessaire pour charger ou actualiser des documents.

Une autre approche consiste à appeler `SearchClient` directement, en transmettant une clé API d’administrateur sur `AzureKeyCredential`.

```csharp
SearchClient searchClient = indexClient.GetSearchClient(indexName);

Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(searchClient);
```

## <a name="run-queries"></a>Exécuter des requêtes

Premièrement, configurez un `SearchClient` qui lit le point de terminaison de recherche et la clé API de requête à partir de **appsettings.json** :

```csharp
private static SearchClient CreateSearchClientForQueries(string indexName, IConfigurationRoot configuration)
{
    string searchServiceEndPoint = configuration["SearchServiceEndPoint"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchClient searchClient = new SearchClient(new Uri(searchServiceEndPoint), indexName, new AzureKeyCredential(queryApiKey));
    return searchClient;
}
```

Deuxièmement, définissez une méthode qui envoie une demande de requête. 

Chaque fois qu’elle exécute une requête, la méthode crée un objet [`SearchOptions`](/dotnet/api/azure.search.documents.searchoptions). Cet objet permet de spécifier des options supplémentaires pour la requête, comme le tri, le filtrage, la pagination et la génération de facettes. Dans cette méthode, nous définissons les propriétés `Filter`, `Select` et `OrderBy` pour différentes requêtes. Pour plus d’informations sur la syntaxe des requêtes de recherche, consultez [Syntaxe de requête simple](/rest/api/searchservice/Simple-query-syntax-in-Azure-Search).

L'étape suivante consiste à exécuter la requête de recherche. La recherche est exécutée à l’aide de la méthode `SearchClient.Search`. Pour chaque requête, transmettez le texte de recherche à utiliser en tant que chaîne (ou à défaut l’élément `"*"`) ainsi que les options de recherche créées précédemment. Nous spécifions également `Hotel` comme paramètre de type pour `SearchClient.Search`, qui demande au SDK de désérialiser les documents figurant dans les résultats de recherche, en objets de type `Hotel`.

```csharp
private static void RunQueries(SearchClient searchClient)
{
    SearchOptions options;
    SearchResults<Hotel> results;

    Console.WriteLine("Query 1: Search for 'motel'. Return only the HotelName in results:\n");

    options = new SearchOptions();
    options.Select.Add("HotelName");

    results = searchClient.Search<Hotel>("motel", options);

    WriteDocuments(results);

    Console.Write("Query 2: Apply a filter to find hotels with rooms cheaper than $100 per night, ");
    Console.WriteLine("returning the HotelId and Description:\n");

    options = new SearchOptions()
    {
        Filter = "Rooms/any(r: r/BaseRate lt 100)"
    };
    options.Select.Add("HotelId");
    options.Select.Add("Description");

    results = searchClient.Search<Hotel>("*", options);

    WriteDocuments(results);

    Console.Write("Query 3: Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    options =
        new SearchOptions()
        {
            Size = 2
        };
    options.OrderBy.Add("LastRenovationDate desc");
    options.Select.Add("HotelName");
    options.Select.Add("LastRenovationDate");

    results = searchClient.Search<Hotel>("*", options);

    WriteDocuments(results);

    Console.WriteLine("Query 4: Search the HotelName field for the term 'hotel':\n");

    options = new SearchOptions();
    options.SearchFields.Add("HotelName");

    //Adding details to select, because "Location" is not supported yet when deserialize search result to "Hotel"
    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Description");
    options.Select.Add("Category");
    options.Select.Add("Tags");
    options.Select.Add("ParkingIncluded");
    options.Select.Add("LastRenovationDate");
    options.Select.Add("Rating");
    options.Select.Add("Address");
    options.Select.Add("Rooms");

    results = searchClient.Search<Hotel>("hotel", options);

    WriteDocuments(results);
}
```

Troisièmement, définissez une méthode qui écrit la réponse, en imprimant chaque document sur la console :

```csharp
private static void WriteDocuments(SearchResults<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.GetResults())
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

### <a name="call-runqueries-in-main"></a>Appeler RunQueries dans Main()

```csharp
SearchClient indexClientForQueries = CreateSearchClientForQueries(indexName, configuration);

Console.WriteLine("{0}", "Running queries...\n");
RunQueries(indexClientForQueries);
```

### <a name="explore-query-constructs"></a>Explorer les constructions de requête

Examinons de plus près chacune des requêtes exécutées. Voici le code permettant d’exécuter la première requête :

```csharp
options = new SearchOptions();
options.Select.Add("HotelName");

results = searchClient.Search<Hotel>("motel", options);

WriteDocuments(results);
```

Dans ce cas, nous recherchons dans l’index entier le mot « motel », dans tout champ pouvant faire l’objet d’une recherche, et ne voulons récupérer que les noms d’hôtel, comme spécifié par l’option `Select`. Voici les résultats :

```output
Name: Secret Point Motel

Name: Twin Dome Motel
```

Dans la deuxième requête, utilisez un filtre pour sélectionner une chambre dont le tarif est inférieur à 100 $. Renvoyez uniquement l’ID de l’hôtel et la description dans les résultats :

```csharp
options = new SearchOptions()
{
    Filter = "Rooms/any(r: r/BaseRate lt 100)"
};
options.Select.Add("HotelId");
options.Select.Add("Description");

results = searchClient.Search<Hotel>("*", options);
```

La requête ci-dessus utilise une expression `$filter` OData (`Rooms/any(r: r/BaseRate lt 100)`) pour filtrer les documents dans l’index. Cet exemple utilise l’[opérateur any](./search-query-odata-collection-operators.md) pour appliquer le « BaseRate lt 100 » à chaque élément de la collection Rooms (Chambres). Pour plus d’informations, voir [Syntaxe de filtre OData](./query-odata-filter-orderby-syntax.md).

Dans la troisième requête, recherchez les deux hôtels ayant été le plus récemment rénovés, et affichez le nom et la date de la dernière rénovation de ces derniers. Voici le code : 

```csharp
options =
    new SearchOptions()
    {
        Size = 2
    };
options.OrderBy.Add("LastRenovationDate desc");
options.Select.Add("HotelName");
options.Select.Add("LastRenovationDate");

results = searchClient.Search<Hotel>("*", options);

WriteDocuments(results);
```

Dans la dernière requête, recherchez tous les noms d’hôtel contenant le mot « hotel » :

```csharp
options.Select.Add("HotelId");
options.Select.Add("HotelName");
options.Select.Add("Description");
options.Select.Add("Category");
options.Select.Add("Tags");
options.Select.Add("ParkingIncluded");
options.Select.Add("LastRenovationDate");
options.Select.Add("Rating");
options.Select.Add("Address");
options.Select.Add("Rooms");

results = searchClient.Search<Hotel>("hotel", options);

WriteDocuments(results);
```

Cette section conclut la présentation du SDK .NET, mais ne vous arrêtez pas en si bon chemin. La section suivante propose des ressources supplémentaires pour en savoir plus sur la programmation avec Recherche cognitive Azure.

## <a name="next-steps"></a>Étapes suivantes

+ Parcourez la documentation de référence de l’API pour [Azure.Search.Documents](/dotnet/api/azure.search.documents) et [API REST](/rest/api/searchservice/)

+ Parcourez d’autres exemples de code basés sur Azure.Search.Documents dans [azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples) et [search-getting-started-dotnet](https://github.com/Azure-Samples/search-dotnet-getting-started)

+ Consultez les [conventions d’affectation de noms](/rest/api/searchservice/Naming-rules) pour apprendre les règles de dénomination des différents objets.

+ Consultez les [types de données pris en charge](/rest/api/searchservice/Supported-data-types)