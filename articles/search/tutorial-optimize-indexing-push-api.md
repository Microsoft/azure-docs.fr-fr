---
title: 'Tutoriel C# : optimisation de l’indexation avec l’API Push'
titleSuffix: Azure Cognitive Search
description: Découvrez comment indexer efficacement des données à l’aide de l’API Push de Recherche cognitive Azure. Ce didacticiel et cet exemple de code utilisent le langage C#.
manager: liamca
author: dereklegenzoff
ms.author: delegenz
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 10/12/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: d22ff5c863617a3feb2a08d4b1889d0a7c10cd3a
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94693395"
---
# <a name="tutorial-optimize-indexing-with-the-push-api"></a>Tutoriel : Optimiser l’indexation avec l’API Push

Recherche cognitive Azure prend en charge [deux approches de base](search-what-is-data-import.md) pour importer des données dans un index de recherche : *envoyer (push)* les données dans l’index par programme ou pointer un [indexeur Recherche cognitive Azure](search-indexer-overview.md) vers une source de données prise en charge pour *extraire (pull)* les données.

Ce tutoriel explique comment indexer efficacement des données à l’aide du [modèle Push](search-what-is-data-import.md#pushing-data-to-an-index) en utilisant le traitement par lot des demandes et en utilisant une stratégie de nouvelle tentative d’interruption exponentielle. Vous pouvez [Télécharger et exécuter l’exemple d’application](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing). Cet article explique les aspects clés de l’application et les facteurs à prendre en compte lors de l’indexation des données.

Ce tutoriel utilise C# et le [SDK .NET](/dotnet/api/overview/azure/search) pour effectuer les tâches suivantes :

> [!div class="checklist"]
> * Création d'un index
> * Tester différentes tailles de lot pour déterminer la taille la plus efficace
> * Indexer les lots de façon asynchrone
> * Utiliser plusieurs threads pour augmenter les vitesses d’indexation
> * Utiliser une stratégie de nouvelle tentative de backoff exponentiel pour réessayer des documents ayant échoué

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Les services et outils suivants sont indispensables dans ce tutoriel.

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/), toute édition. L’exemple de code et les instructions ont été testés dans l’édition Communauté gratuite.

+ [Créez un service Recherche cognitive Azure](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) dans votre abonnement actuel.

<a name="get-service-info"></a>

## <a name="download-files"></a>Télécharger les fichiers

Le code source pour ce tutoriel se trouve dans le dossier [optimzize-data-indexing/v11](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing/v11) du dépôt GitHub [Azure-Samples/azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples).

## <a name="key-considerations"></a>Considérations relatives aux clés

Lors de l’envoi (push) de données dans un index, plusieurs considérations importantes affectent les vitesses d’indexation. Pour plus d’informations sur ces facteurs, consultez l’article [Indexer de grands jeux de données](search-howto-large-index.md).

Les six facteurs clés à prendre en compte sont les suivants :

+ **Niveau de service et nombre de partitions/réplicas** : L’ajout de partitions et l’augmentation de votre niveau augmentent les vitesses d’indexation.
+ **Schéma d’index** : L’ajout de champs et l’ajout de propriétés supplémentaires à des champs (comme *searchable*, *facetable* ou *filterable*) réduisent les vitesses d’indexation.
+ **Taille de lot** : La taille de lot optimale varie en fonction du schéma et du jeu de données de votre index.
+ **Nombre de threads/workers** : Un seul thread ne tirera pas pleinement parti des vitesses d’indexation
+ **Stratégie de nouvelle tentative** : Une stratégie de nouvelle tentative d’interruption exponentielle doit être utilisée pour optimiser l’indexation.
+ **Vitesse de transfert des données du réseau** : La vitesse de transfert des données peut être un facteur limitatif. Indexez les données à partir de votre environnement Azure pour augmenter la vitesse de transfert des données.


## <a name="1---create-azure-cognitive-search-service"></a>1 - Créer un service Recherche cognitive Azure

Pour suivre ce didacticiel, vous avez besoin d’un service Recherche cognitive Azure, que vous pouvez [créer dans le portail](search-create-service-portal.md). Nous vous recommandons d’utiliser le même niveau que celui que vous prévoyez d’utiliser en production pour pouvoir tester et optimiser avec précision les vitesses d’indexation.

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Obtenir une clé API d’administration et une URL pour Recherche cognitive Azure

Les appels d’API nécessitent l’URL du service et une clé d’accès. Un service de recherche est créé avec les deux. Ainsi, si vous avez ajouté la Recherche cognitive Azure à votre abonnement, effectuez ce qui suit pour obtenir les informations nécessaires :

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

1. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

   ![Obtenir un point de terminaison et une clé d’accès HTTP](media/search-get-started-rest/get-url-key.png "Obtenir un point de terminaison et une clé d’accès HTTP")

## <a name="2---set-up-your-environment"></a>2 - Configurer votre environnement

1. Démarrez Visual Studio et ouvrez **OptimizeDataIndexing.sln**.
1. Dans l’Explorateur de solutions, ouvrez **appsettings.json** pour fournir les informations de connexion.

```json
{
  "SearchServiceUri": "https://{service-name}.search.windows.net",
  "SearchServiceAdminApiKey": "",
  "SearchIndexName": "optimize-indexing"
}
```

## <a name="3---explore-the-code"></a>3 - Explorer le code

Une fois que vous avez mis à jour *appSettings.json*, l’exemple de programme dans **OptimizeDataIndexing.sln** doit être prêt pour la génération et l’exécution.

Ce code est dérivé du [Démarrage rapide de C#](search-get-started-dotnet.md). Vous trouverez des informations plus détaillées sur les principes de base de l’utilisation du kit de développement logiciel (SDK) .NET dans cet article.

Cette application console C#/.NET simple effectue les tâches suivantes :

+ Elle crée un index basé sur la structure de données de la classe Hotel C# (qui référence également la classe Address).
+ Teste différentes tailles de lot pour déterminer la taille la plus efficace
+ Indexe les données de façon asynchrone
    + Utilisation de plusieurs threads pour augmenter les vitesses d’indexation
    + Utilisation d’une stratégie de nouvelle tentative d’interruption exponentielle pour réessayer les éléments ayant échoué

 Avant d’exécuter le programme, prenez une minute pour étudier le code et les définitions d’index de cet exemple. Le code qui convient se trouve dans plusieurs fichiers :

  + **Hotel.cs** et **Address.cs** contiennent le schéma qui définit l’index
  + **DataGenerator.cs** contient une classe simple pour faciliter la création de grandes quantités de données d’hôtel
  + **ExponentialBackoff.cs** contient du code pour optimiser le processus d’indexation, comme décrit ci-dessous
  + **Program.cs** contient des fonctions qui créent et suppriment l’index Recherche cognitive Azure, indexent des lots de données et testent différentes tailles de lot

### <a name="creating-the-index"></a>Création de l’index

Cet exemple de programme utilise le SDK .NET pour définir et créer un index de Recherche cognitive Azure. Il tire parti de la classe `FieldBuilder` pour générer une structure d’index à partir d’une classe de modèle de données C#.

Le modèle de données est défini par la classe Hotel, qui contient également des références à la classe Address. La classe FieldBuilder explore plusieurs définitions de classe pour générer une structure de données complexes pour l’index. Des étiquettes de métadonnées sont utilisées pour définir les attributs de chaque champ, par exemple s’il peut faire l’objet d’une recherche ou d’un tri.

Les extraits de code suivants tirés du fichier **Hotel.cs** montrent comment spécifier un champ unique et une référence à une autre classe de modèle de données.

```csharp
. . .
[SearchableField(IsSortable = true)]
public string HotelName { get; set; }
. . .
public Address Address { get; set; }
. . .
```

Dans le fichier **Program.cs**, l’index est défini avec un nom et une collection de champs générés par la méthode `FieldBuilder.Build(typeof(Hotel))`, puis créé comme suit :

```csharp
private static async Task CreateIndexAsync(string indexName, SearchIndexClient indexClient)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address class is referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    FieldBuilder builder = new FieldBuilder();
    var definition = new SearchIndex(indexName, builder.Build(typeof(Hotel)));

    await indexClient.CreateIndexAsync(definition);
}
```

### <a name="generating-data"></a>Génération des données

Une classe simple est implémentée dans le fichier **DataGenerator.cs** pour générer des données à des fins de test. Le seul but de cette classe est de faciliter la génération d’un grand nombre de documents avec un ID unique pour l’indexation.

Pour obtenir une liste de 100 000 hôtels avec des ID uniques, vous devez exécuter les lignes de code suivantes :

```csharp
long numDocuments = 100000;
DataGenerator dg = new DataGenerator();
List<Hotel> hotels = dg.GetHotels(numDocuments, "large");
```

Deux tailles d’hôtels sont disponibles pour le test dans cet exemple : **small** et **large**.

Le schéma de votre index peut avoir un impact significatif sur les vitesses d’indexation. En raison de cet impact, il est logique de convertir cette classe afin de générer des données correspondant à votre schéma d’index prévu après l’exécution de ce didacticiel.

## <a name="4---test-batch-sizes"></a>4 - Tester les tailles de lot

Recherche cognitive Azure prend en charge les API suivantes pour charger un ou plusieurs documents dans un index :

+ [Ajout, mise à jour ou suppression de documents (API REST)](/rest/api/searchservice/AddUpdate-or-Delete-Documents)
+ [Classe IndexDocumentsAction](/dotnet/api/azure.search.documents.models.indexdocumentsaction) ou [classe IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch)

L’indexation de documents par lots améliore considérablement les performances d’indexation. Ces lots peuvent comporter jusqu’à 1000 documents, ou jusqu’à 16 Mo par lot.

La détermination de la taille de lot optimale pour vos données est un composant clé de l’optimisation des vitesses d’indexation. Les deux principaux facteurs qui influencent la taille de lot optimale sont les suivants :

+ Le schéma de votre index
+ La taille de vos données

Étant donné que la taille de lot optimale dépend de votre index et de vos données, la meilleure approche consiste à tester différentes tailles de lot pour déterminer ce qui produit les vitesses d’indexation les plus rapides pour votre scénario.

La fonction suivante illustre une approche simple pour tester les tailles de lot.

```csharp
public static async Task TestBatchSizesAsync(SearchClient searchClient, int min = 100, int max = 1000, int step = 100, int numTries = 3)
{
    DataGenerator dg = new DataGenerator();

    Console.WriteLine("Batch Size \t Size in MB \t MB / Doc \t Time (ms) \t MB / Second");
    for (int numDocs = min; numDocs <= max; numDocs += step)
    {
        List<TimeSpan> durations = new List<TimeSpan>();
        double sizeInMb = 0.0;
        for (int x = 0; x < numTries; x++)
        {
            List<Hotel> hotels = dg.GetHotels(numDocs, "large");

            DateTime startTime = DateTime.Now;
            await UploadDocumentsAsync(searchClient, hotels).ConfigureAwait(false);
            DateTime endTime = DateTime.Now;
            durations.Add(endTime - startTime);

            sizeInMb = EstimateObjectSize(hotels);
        }

        var avgDuration = durations.Average(timeSpan => timeSpan.TotalMilliseconds);
        var avgDurationInSeconds = avgDuration / 1000;
        var mbPerSecond = sizeInMb / avgDurationInSeconds;

        Console.WriteLine("{0} \t\t {1} \t\t {2} \t\t {3} \t {4}", numDocs, Math.Round(sizeInMb, 3), Math.Round(sizeInMb / numDocs, 3), Math.Round(avgDuration, 3), Math.Round(mbPerSecond, 3));

        // Pausing 2 seconds to let the search service catch its breath
        Thread.Sleep(2000);
    }

    Console.WriteLine();
}
```

Étant donné que tous les documents ne sont pas de la même taille (même si c’est le cas dans cet exemple), nous estimons la taille des données que nous envoyons au service de recherche. Pour cela, nous allons utiliser la fonction ci-dessous, qui convertit d’abord l’objet en JSON, puis détermine sa taille en octets. Cette technique nous permet de déterminer les tailles de lot les plus efficaces en termes de vitesses d’indexation en Mo/s.

```csharp
// Returns size of object in MB
public static double EstimateObjectSize(object data)
{
    // converting object to byte[] to determine the size of the data
    BinaryFormatter bf = new BinaryFormatter();
    MemoryStream ms = new MemoryStream();
    byte[] Array;

    // converting data to json for more accurate sizing
    var json = JsonSerializer.Serialize(data);
    bf.Serialize(ms, json);
    Array = ms.ToArray();

    // converting from bytes to megabytes
    double sizeInMb = (double)Array.Length / 1000000;

    return sizeInMb;
}
```

La fonction requiert un `SearchClient`, ainsi que le nombre de tentatives que vous souhaitez tester pour chaque taille de lot. Comme il peut y avoir une certaine variabilité dans les temps d’indexation pour chaque lot, nous essayons chaque lot trois fois par défaut afin que les résultats soient plus significatifs.

```csharp
await TestBatchSizesAsync(searchClient, numTries: 3);
```

Lorsque vous exécutez la fonction, vous devez voir une sortie semblable à celle-ci dans votre console :

   ![Sortie de la fonction de taille de lot de test](media/tutorial-optimize-data-indexing/test-batch-sizes.png "Sortie de la fonction de taille de lot de test")

Identifiez la taille de lot la plus efficace, puis utilisez cette taille de lot à l’étape suivante du didacticiel. Vous pouvez voir un plateau de Mo/s sur différentes tailles de lot.

## <a name="5---index-data"></a>5 - Indexer des données

Maintenant que nous avons identifié la taille de lot que nous envisageons d’utiliser, l’étape suivante consiste à commencer à indexer les données. Pour indexer les données efficacement, cet exemple :

+ utilise plusieurs threads/workers.
+ Implémente une stratégie de nouvelle tentative d’interruption exponentielle.

### <a name="use-multiple-threadsworkers"></a>Utiliser plusieurs threads/workers

Pour tirer pleinement parti des vitesses d’indexation de Recherche cognitive Azure, vous devrez probablement utiliser plusieurs threads pour envoyer simultanément des demandes d’indexation par lot au service.  

Plusieurs des considérations clés mentionnées ci-dessus ont un impact sur le nombre optimal de threads. Vous pouvez modifier cet exemple et effectuer des tests avec différents nombres de threads pour déterminer le nombre de threads optimal pour votre scénario. Toutefois, tant que plusieurs threads s’exécutent simultanément, vous devriez être en mesure de bénéficier de la majeure partie des gains d’efficacité.

Au fur et à mesure que les requêtes atteignent le service de recherche, vous pouvez rencontrer des [codes d’état HTTP](/rest/api/searchservice/http-status-codes) indiquant que la demande n’a pas abouti. Pendant l’indexation, deux codes d’état HTTP courants sont :

+ **503 Service indisponible** : Cette erreur signifie que le système est surchargé et que votre requête ne peut pas être traitée pour le moment.
+ **207 Multi-état** : Cette erreur signifie que certains documents ont réussi, mais qu’au moins un a échoué.

### <a name="implement-an-exponential-backoff-retry-strategy"></a>Implémenter une stratégie de nouvelle tentative d’interruption exponentielle

En cas d’échec, les requêtes doivent être retentées à l’aide d’une [stratégie de nouvelle tentative d’interruption exponentielle](/dotnet/architecture/microservices/implement-resilient-applications/implement-retries-exponential-backoff).

Le kit de développement logiciel (SDK) .NET de Recherche cognitive Azure retente automatiquement lors des erreurs 503 et autres requêtes ayant échoué, mais vous devez implémenter votre propre logique pour réessayer en cas de code 207. Des outils open source tels que [Polly](https://github.com/App-vNext/Polly) peuvent également être utilisés pour mettre en œuvre une stratégie de nouvelle tentative.

Dans cet exemple, nous implémentons notre propre stratégie de nouvelle tentative d’interruption exponentielle. Pour implémenter cette stratégie, nous commençons par définir certaines variables, dont `maxRetryAttempts` et la valeur `delay` initiale pour une requête ayant échoué :

```csharp
// Create batch of documents for indexing
var batch = IndexDocumentsBatch.Upload(hotels);

// Create an object to hold the result
IndexDocumentsResult result = null;

// Define parameters for exponential backoff
int attempts = 0;
TimeSpan delay = delay = TimeSpan.FromSeconds(2);
int maxRetryAttempts = 5;
```

Les résultats de l’opération d’indexation sont stockés dans la variable `IndexDocumentResult result`. Cette variable est importante, car elle vous permet de vérifier si des documents du lot ont échoué, comme illustré ci-dessous. En cas de défaillance partielle, un nouveau lot est créé en fonction de l’ID des documents ayant échoué.

Les exceptions `RequestFailedException` doivent également être interceptées, car elles indiquent que la demande a complètement échoué et doit également retentée.

```csharp
// Implement exponential backoff
do
{
    try
    {
        attempts++;
        result = await searchClient.IndexDocumentsAsync(batch).ConfigureAwait(false);

        var failedDocuments = result.Results.Where(r => r.Succeeded != true).ToList();

        // handle partial failure
        if (failedDocuments.Count > 0)
        {
            if (attempts == maxRetryAttempts)
            {
                Console.WriteLine("[MAX RETRIES HIT] - Giving up on the batch starting at {0}", id);
                break;
            }
            else
            {
                Console.WriteLine("[Batch starting at doc {0} had partial failure]", id);
                Console.WriteLine("[Retrying {0} failed documents] \n", failedDocuments.Count);

                // creating a batch of failed documents to retry
                var failedDocumentKeys = failedDocuments.Select(doc => doc.Key).ToList();
                hotels = hotels.Where(h => failedDocumentKeys.Contains(h.HotelId)).ToList();
                batch = IndexDocumentsBatch.Upload(hotels);

                Task.Delay(delay).Wait();
                delay = delay * 2;
                continue;
            }
        }

        return result;
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("[Batch starting at doc {0} failed]", id);
        Console.WriteLine("[Retrying entire batch] \n");

        if (attempts == maxRetryAttempts)
        {
            Console.WriteLine("[MAX RETRIES HIT] - Giving up on the batch starting at {0}", id);
            break;
        }

        Task.Delay(delay).Wait();
        delay = delay * 2;
    }
} while (true);
```

À partir de là, nous encapsulons le code d’interruption exponentielle dans une fonction afin qu’il puisse être facilement appelé.

Une autre fonction est ensuite créée pour gérer les threads actifs. Par souci de simplicité, cette fonction n’est pas incluse ici, mais se trouve dans [ExponentialBackoff.cs](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/optimize-data-indexing/v11/OptimizeDataIndexing/ExponentialBackoff.cs). La fonction peut être appelée à l’aide de la commande suivante, où `hotels` correspond aux données que nous voulons télécharger, `1000` à la taille du lot et `8` au nombre de threads simultanés :

```csharp
await ExponentialBackoff.IndexData(indexClient, hotels, 1000, 8);
```

Lorsque vous exécutez la fonction, vous devez voir une sortie semblable à celle ci-dessous :

![Sortie de la fonction de données d’index](media/tutorial-optimize-data-indexing/index-data-start.png "Sortie de la fonction de données d’index")

En cas d’échec d’un lot de documents, une erreur s’affiche pour indiquer qu’un échec s’est produit et que le lot est retenté :

```
[Batch starting at doc 6000 had partial failure]
[Retrying 560 failed documents]
```

Une fois l’exécution de la fonction terminée, vous pouvez vérifier que tous les documents ont été ajoutés à l’index.

## <a name="6---explore-index"></a>6 - Explorer l’index

Une fois le programme exécuté, vous pouvez explorer l’index de recherche rempli, par programme ou en utilisant l’[**Explorateur de recherche**](search-explorer.md) dans le portail.

### <a name="programatically"></a>Par programme

Il existe deux options principales pour vérifier le nombre de documents dans un index : l’[API Nombre de documents](/rest/api/searchservice/count-documents) et l’[API Obtention de statistiques d'index](/rest/api/searchservice/get-index-statistics). Les deux peuvent nécessiter du temps supplémentaire pour la mise à jour. Par conséquent, ne vous inquiétez pas si le nombre de documents renvoyés est inférieur à celui prévu initialement.

#### <a name="count-documents"></a>Nombre de documents

L'opération Count Documents récupère le nombre de documents dans un index de recherche :

```csharp
long indexDocCount = await searchClient.GetDocumentCountAsync();
```

#### <a name="get-index-statistics"></a>Obtention de statistiques d'index

L'opération Get Index Statistics retourne un nombre de documents pour l'index actuel, ainsi que l'utilisation du stockage. La mise à jour des statistiques d’index prendra plus de temps que celle du nombre de documents.

```csharp
var indexStats = await indexClient.GetIndexStatisticsAsync(indexName);
```

### <a name="azure-portal"></a>Portail Azure

Dans le portail Azure, ouvrez la page **Vue d’ensemble** du service de recherche, puis recherchez l’index **optimize-indexing** dans la liste **Index**.

  ![Liste des index de Recherche cognitive Azure](media/tutorial-optimize-data-indexing/portal-output.png "Liste des index de Recherche cognitive Azure")

Le *Nombre de documents* et la *Taille de stockage* sont basés sur l’[API Obtention de statistiques d'index](/rest/api/searchservice/get-index-statistics) et peuvent nécessiter plusieurs minutes pour se mettre à jour.

## <a name="reset-and-rerun"></a>Réinitialiser et réexécuter

Dans les premières étapes expérimentales du développement, l’approche la plus pratique pour les itérations de conception consiste à supprimer les objets de Recherche cognitive Azure et à autoriser votre code à les reconstruire. Les noms des ressources sont uniques. La suppression d’un objet vous permet de le recréer en utilisant le même nom.

L’exemple de code pour ce tutoriel recherche les index existants et les supprime pour vous permettre de réexécuter votre code.

Vous pouvez également utiliser le portail pour supprimer des index.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est judicieux à la fin d’un projet de supprimer les ressources dont vous n’avez plus besoin. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez fait connaissance avec le concept d’ingestion efficace des données, examinons de plus près l’architecture des requêtes Lucene et le fonctionnement de la recherche en texte intégral dans Recherche cognitive Azure.

> [!div class="nextstepaction"]
> [Fonctionnement de la recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md)