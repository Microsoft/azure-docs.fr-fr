---
title: 'Démarrage rapide : Bibliothèque de client C# Recherche d’images Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/21/2020
ms.author: aahi
ms.openlocfilehash: e051eca990ae0aa0b5a79c208a594e1b2332bcb2
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97612639"
---
Utilisez ce guide de démarrage rapide pour effectuer votre première recherche d’images à l’aide de la bibliothèque de client Recherche d’images Bing. 

Cette bibliothèque est un wrapper de l’API REST et contient les mêmes fonctionnalités. 

Vous allez créer une application C# simple qui envoie une requête de recherche d’image, analyse la réponse JSON et affiche l’URL de la première image retournée.

Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingImageSearch) avec une gestion des erreurs supplémentaire et des annotations.

## <a name="prerequisites"></a>Prérequis

* Si vous utilisez Windows, n’importe quelle édition de [Visual Studio 2017 ou version ultérieure](https://visualstudio.microsoft.com/vs/whatsnew/)
* Si vous utilisez macOS ou Linux, [VS Code](https://code.visualstudio.com) avec [.NET Core installé](https://dotnet.microsoft.com/learn/dotnet/hello-world-tutorial/install)
* [Un abonnement Azure gratuit](https://azure.microsoft.com/free/dotnet)

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](~/includes/cognitive-services-bing-image-search-signup-requirements.md)]

Consultez également [Tarification Cognitive Services - API Recherche Bing](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/).

## <a name="create-a-console-project"></a>Créer un projet de console

Tout d’abord, créez une application console C#.

# <a name="visual-studio"></a>[Visual Studio](#tab/visualstudio)

1. Créez une solution de console nommée *BingImageSearch* dans Visual Studio.
    
1. Ajoutez le [package NuGet de recherche d’images cognitive](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.ImageSearch) :
    1. Cliquez avec le bouton droit sur l’**Explorateur de solutions**.
    1. Sélectionnez **Gérer les packages NuGet**.
    1. Recherchez et sélectionnez *Microsoft.Azure.CognitiveServices.Search.ImageSearch*, puis installez le package.
    
# <a name="vs-code"></a>[Code Visual Studio](#tab/vscode)

1. Ouvrez la fenêtre de terminal dans VS Code.
1. Créez un projet de console nommé *BingImageSearch* en entrant le code suivant dans la fenêtre de terminal :
    
    ```bash
    dotnet new console -n BingImageSearch
    ```
1. Ouvrez le dossier *BingImageSearch* dans VS Code.
1. Ajoutez le [package NuGet de recherche d’images cognitive](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.ImageSearch) en entrant le code suivant dans la fenêtre de terminal :

    ```bash
    dotnet add package Microsoft.Azure.CognitiveServices.Search.ImageSearch
    ```

---

## <a name="initialize-the-application"></a>Initialiser l’application


1. Remplacez toutes les instructions `using` dans *Program.cs* par le code suivant :

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.Azure.CognitiveServices.Search.ImageSearch;
    using Microsoft.Azure.CognitiveServices.Search.ImageSearch.Models;
    ```

1. Dans la méthode `Main` de votre projet, créez des variables pour votre clé d’abonnement valide, les résultats d’image devant être retournés par Bing et un terme de recherche. Ensuite, instanciez le client de recherche d’image à l’aide de la clé.

    ```csharp
    static async Task Main(string[] args)
    {
        //IMPORTANT: replace this variable with your Cognitive Services subscription key
        string subscriptionKey = "ENTER YOUR KEY HERE";
        //stores the image results returned by Bing
        Images imageResults = null;
        // the image search term to be used in the query
        string searchTerm = "canadian rockies";
        
        //initialize the client
        //NOTE: If you're using version 1.2.0 or below for the Bing Image Search client library, 
        // use ImageSearchAPI() instead of ImageSearchClient() to initialize your search client.
        
        var client = new ImageSearchClient(new ApiKeyServiceClientCredentials(subscriptionKey));
    }
    ```
    
## <a name="send-a-search-query-using-the-client"></a>Envoyer une requête de recherche avec le client
    
Toujours dans la méthode `Main`, utilisez le client pour effectuer une recherche avec un texte de requête :
    
```csharp
// make the search request to the Bing Image API, and get the results"
imageResults = await client.Images.SearchAsync(query: searchTerm).Result; //search query
```

## <a name="parse-and-view-the-first-image-result"></a>Analyser et afficher le premier résultat d’image

Analysez les résultats d’image retournés par la réponse. 

Si la réponse contient des résultats de recherche, stockez le premier résultat et affichez certains de ses détails.

```csharp
if (imageResults != null)
{
    //display the details for the first image result.
    var firstImageResult = imageResults.Value.First();
    Console.WriteLine($"\nTotal number of returned images: {imageResults.Value.Count}\n");
    Console.WriteLine($"Copy the following URLs to view these images on your browser.\n");
    Console.WriteLine($"URL to the first image:\n\n {firstImageResult.ContentUrl}\n");
    Console.WriteLine($"Thumbnail URL for the first image:\n\n {firstImageResult.ThumbnailUrl}");
    Console.WriteLine("Press any key to exit ...");
    Console.ReadKey();
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel d’une application monopage Recherche d’images Bing](../../tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que la Recherche d’images Bing ?](../../overview.md)  
* [Essayez une démonstration interactive en ligne](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [Exemples .NET pour le SDK Azure Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
* [Documentation Azure Cognitive Services](../../../index.yml)
* [Informations de référence sur l’API Recherche d’images Bing](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)