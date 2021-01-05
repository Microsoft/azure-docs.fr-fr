---
title: 'Démarrage rapide : bibliothèque de client Recherche Web Bing avec C#'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/19/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: d391586ade9e9a58344f9b1666802a453770152a
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94386566"
---
La bibliothèque de client de l’API Recherche Web Bing simplifie l’intégration de l’API dans votre application C#. Dans ce démarrage rapide, vous allez apprendre à instancier un client, envoyer une requête et imprimer la réponse.

Vous voulez voir le code tout de suite ? Des exemples de [bibliothèques de client Recherche Bing pour .NET](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7) sont disponibles sur GitHub.

## <a name="prerequisites"></a>Conditions préalables requises
Voici quelques points dont vous aurez besoin avant d’exécuter ce démarrage rapide :

* [Visual Studio](https://visualstudio.microsoft.com/downloads/) ou
* [Visual Studio Code 2017](https://code.visualstudio.com/download)
  * [C# pour Visual Studio Code](https://visualstudio.microsoft.com/downloads/)
  * [Gestionnaire de package NuGet](https://github.com/jmrog/vscode-nuget-package-manager)
* [Kit de développement logiciel (SDK) .NET Core](https://www.microsoft.com/net/download)

[!INCLUDE [bing-web-search-quickstart-signup](~/includes/bing-web-search-quickstart-signup.md)]

## <a name="create-a-project-and-install-dependencies"></a>Créer un projet et installer des dépendances

> [!TIP]
> Récupérer le dernier code en tant que solution Visual Studio à partir de [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/).

La première étape consiste à créer un nouveau projet de console. Si vous avez besoin d’aide pour configurer un projet de console, consultez [Hello World--votre premier programme (Guide de programmation C#)](/dotnet/csharp/programming-guide/inside-a-program/hello-world-your-first-program). Pour utiliser le kit de développement logiciel de l’API Recherche Web Bing dans votre application, vous devrez installer `Microsoft.Azure.CognitiveServices.Search.WebSearch` à l’aide du gestionnaire de package NuGet.

Le [package du kit de développement Web de l’API Recherche Web Bing](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.WebSearch/1.2.0) installe également :

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

## <a name="declare-dependencies"></a>Déclarer des dépendances

Ouvrez votre projet dans Visual Studio ou Visual Studio Code et importez ces dépendances :

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.CognitiveServices.Search.WebSearch;
using Microsoft.Azure.CognitiveServices.Search.WebSearch.Models;
using System.Linq;
using System.Threading.Tasks;
```

## <a name="create-project-scaffolding"></a>Créer la génération de modèles automatique du projet

Lorsque vous avez créé votre nouveau projet de console, il est nécessaire de créer un espace de noms et une classe pour votre application. Votre programme doit avoir l’aspect de cet exemple :

```csharp
namespace WebSearchSDK
{
    class YOUR_PROGRAM
    {

        // The code in the following sections goes here.

    }
}
```

Dans les sections suivantes, nous allons créer notre exemple d’application au sein de cette classe.

## <a name="construct-a-request"></a>Construire une requête

Ce code construit la requête de la recherche.

```csharp
public static async Task WebResults(WebSearchClient client)
{
    try
    {
        var webData = await client.Web.SearchAsync(query: "Yosemite National Park");
        Console.WriteLine("Searching for \"Yosemite National Park\"");

        // Code for handling responses is provided in the next section...

    }
    catch (Exception ex)
    {
        Console.WriteLine("Encountered exception. " + ex.Message);
    }
}
```

## <a name="handle-the-response"></a>Gérer la réponse

Ensuite, nous allons ajouter du code pour analyser la réponse et imprimer les résultats. Le `Name` et `Url` pour la première page web, image, vidéo et le premier bulletin d’informations sont imprimés s’ils sont présents dans l’objet de la réponse.

```csharp
if (webData?.WebPages?.Value?.Count > 0)
{
    // find the first web page
    var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

    if (firstWebPagesResult != null)
    {
        Console.WriteLine("Webpage Results # {0}", webData.WebPages.Value.Count);
        Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
        Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
    }
    else
    {
        Console.WriteLine("Didn't find any web pages...");
    }
}
else
{
    Console.WriteLine("Didn't find any web pages...");
}

/*
 * Images
 * If the search response contains images, the first result's name
 * and url are printed.
 */
if (webData?.Images?.Value?.Count > 0)
{
    // find the first image result
    var firstImageResult = webData.Images.Value.FirstOrDefault();

    if (firstImageResult != null)
    {
        Console.WriteLine("Image Results # {0}", webData.Images.Value.Count);
        Console.WriteLine("First Image result name: {0} ", firstImageResult.Name);
        Console.WriteLine("First Image result URL: {0} ", firstImageResult.ContentUrl);
    }
    else
    {
        Console.WriteLine("Didn't find any images...");
    }
}
else
{
    Console.WriteLine("Didn't find any images...");
}

/*
 * News
 * If the search response contains news articles, the first result's name
 * and url are printed.
 */
if (webData?.News?.Value?.Count > 0)
{
    // find the first news result
    var firstNewsResult = webData.News.Value.FirstOrDefault();

    if (firstNewsResult != null)
    {
        Console.WriteLine("\r\nNews Results # {0}", webData.News.Value.Count);
        Console.WriteLine("First news result name: {0} ", firstNewsResult.Name);
        Console.WriteLine("First news result URL: {0} ", firstNewsResult.Url);
    }
    else
    {
        Console.WriteLine("Didn't find any news articles...");
    }
}
else
{
    Console.WriteLine("Didn't find any news articles...");
}

/*
 * Videos
 * If the search response contains videos, the first result's name
 * and url are printed.
 */
if (webData?.Videos?.Value?.Count > 0)
{
    // find the first video result
    var firstVideoResult = webData.Videos.Value.FirstOrDefault();

    if (firstVideoResult != null)
    {
        Console.WriteLine("\r\nVideo Results # {0}", webData.Videos.Value.Count);
        Console.WriteLine("First Video result name: {0} ", firstVideoResult.Name);
        Console.WriteLine("First Video result URL: {0} ", firstVideoResult.ContentUrl);
    }
    else
    {
        Console.WriteLine("Didn't find any videos...");
    }
}
else
{
    Console.WriteLine("Didn't find any videos...");
}
```

## <a name="declare-the-main-method"></a>Déclarer la méthode principale

Dans cette application, la méthode principale inclut le code qui instancie le client, valide le `subscriptionKey`et appelle `WebResults`. Assurez-vous que vous entrez une clé d’abonnement valide pour votre compte Azure avant de continuer.

```csharp
static async Task Main(string[] args)
{
    var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

    await WebResults(client);

    Console.WriteLine("Press any key to exit...");
    Console.ReadKey();
}
```

## <a name="run-the-application"></a>Exécution de l'application

Exécutons maintenant l’application !

```console
dotnet run
```

## <a name="define-functions-and-filter-results"></a>Définir des fonctions et filtrer des résultats

Maintenant que vous avez effectué votre premier appel à l’API Recherche Web Bing, examinons quelques fonctions qui mettent en évidence les fonctionnalités du kit de développement logiciel pour affiner des requêtes et filtrer les résultats. Chaque fonction peut être ajoutée à votre programme C# créée dans la section précédente.

### <a name="limit-the-number-of-results-returned-by-bing"></a>Limiter le nombre de résultats retournés par Bing

Cet exemple utilise les paramètres `count` et `offset` pour limiter le nombre de résultats retournés par la recherche « Meilleurs restaurants à Seattle ». Le `Name` et `Url` pour le premier résultat sont imprimés.

1. Ajoutez ce code à votre projet de console :

    ```csharp
    public static async Task WebResultsWithCountAndOffset(WebSearchClient client)
    {
        try
        {
            var webData = await client.Web.SearchAsync(query: "Best restaurants in Seattle", offset: 10, count: 20);
            Console.WriteLine("\r\nSearching for \" Best restaurants in Seattle \"");

            if (webData?.WebPages?.Value?.Count > 0)
            {
                var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

                if (firstWebPagesResult != null)
                {
                    Console.WriteLine("Web Results #{0}", webData.WebPages.Value.Count);
                    Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
                    Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
                }
                else
                {
                    Console.WriteLine("Couldn't find first web result!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any Web data..");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```

2. Ajoutez `WebResultsWithCountAndOffset` à `main` :

    ```csharp
    static async Task Main(string[] args)
    {
        var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        await WebResults(client);
        // Search with count and offset...
        await WebResultsWithCountAndOffset(client);  

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```

3. Exécutez l'application.

### <a name="filter-for-news"></a>Filtrer les actualités

Cet exemple utilise le paramètre `response_filter` pour filtrer les résultats de la recherche. Les résultats de la recherche retournés sont limités aux articles d’actualité pour « Microsoft ». Le `Name` et `Url` pour le premier résultat sont imprimés.

1. Ajoutez ce code à votre projet de console :

    ```csharp
    public static async Task WebSearchWithResponseFilter(WebSearchClient client)
    {
        try
        {
            IList<string> responseFilterstrings = new List<string>() { "news" };
            var webData = await client.Web.SearchAsync(query: "Microsoft", responseFilter: responseFilterstrings);
            Console.WriteLine("\r\nSearching for \" Microsoft \" with response filter \"news\"");

            if (webData?.News?.Value?.Count > 0)
            {
                var firstNewsResult = webData.News.Value.FirstOrDefault();

                if (firstNewsResult != null)
                {
                    Console.WriteLine("News Results #{0}", webData.News.Value.Count);
                    Console.WriteLine("First news result name: {0} ", firstNewsResult.Name);
                    Console.WriteLine("First news result URL: {0} ", firstNewsResult.Url);
                }
                else
                {
                    Console.WriteLine("Couldn't find first News results!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any News data..");
            }

        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```

2. Ajoutez `WebResultsWithCountAndOffset` à `main` :

    ```csharp
    static Task Main(string[] args)
    {
        var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        await WebResults(client);
        // Search with count and offset...
        await WebResultsWithCountAndOffset(client);  
        // Search with news filter...
        await WebSearchWithResponseFilter(client);

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```

3. Exécutez l'application.

### <a name="use-safe-search-answer-count-and-the-promote-filter"></a>Utiliser la recherche sécurisée, le comptage des réponses et le filtre Promouvoir

Cet exemple utilise les paramètres `answer_count`, `promote` et `safe_search` pour filtrer des résultats de recherche pour « Vidéos de musique ». Le `Name` et `ContentUrl` pour le premier résultat sont affichés.

1. Ajoutez ce code à votre projet de console :

    ```csharp
    public static async Task WebSearchWithAnswerCountPromoteAndSafeSearch(WebSearchClient client)
    {
        try
        {
            IList<string> promoteAnswertypeStrings = new List<string>() { "videos" };
            var webData = await client.Web.SearchAsync(query: "Music Videos", answerCount: 2, promote: promoteAnswertypeStrings, safeSearch: SafeSearch.Strict);
            Console.WriteLine("\r\nSearching for \"Music Videos\"");

            if (webData?.Videos?.Value?.Count > 0)
            {
                var firstVideosResult = webData.Videos.Value.FirstOrDefault();

                if (firstVideosResult != null)
                {
                    Console.WriteLine("Video Results # {0}", webData.Videos.Value.Count);
                    Console.WriteLine("First Video result name: {0} ", firstVideosResult.Name);
                    Console.WriteLine("First Video result URL: {0} ", firstVideosResult.ContentUrl);
                }
                else
                {
                    Console.WriteLine("Couldn't find videos results!");
                }
            }
            else
            {
                Console.WriteLine("Didn't see any data..");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Encountered exception. " + ex.Message);
        }
    }
    ```

2. Ajoutez `WebResultsWithCountAndOffset` à `main` :

    ```csharp
    static async Task Main(string[] args)
    {
        var client = new WebSearchClient(new ApiKeyServiceClientCredentials("YOUR_SUBSCRIPTION_KEY"));

        await WebResults(client);
        // Search with count and offset...
        await WebResultsWithCountAndOffset(client);  
        // Search with news filter...
        await WebSearchWithResponseFilter(client);
        // Search with answer count, promote, and safe search
        await WebSearchWithAnswerCountPromoteAndSafeSearch(client);

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
    ```

3. Exécutez l'application.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous avez terminé ce projet, veillez à supprimer votre clé d’abonnement dans le code de l’application.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Exemples du SDK Node.js Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/)
