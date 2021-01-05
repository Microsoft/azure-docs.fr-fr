---
title: 'Démarrage rapide : Utiliser la bibliothèque cliente Recherche personnalisée Bing pour C#'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 02/27/2020
ms.author: aahi
ms.openlocfilehash: 82ceb5b45a2c2bfe754995d3c1b39e46181830ae
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97844790"
---
Commencez à utiliser la bibliothèque cliente Recherche personnalisée Bing pour C#. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base. L’API Recherche personnalisée Bing vous permet de créer des expériences de recherche exemptes de publicité, adaptées aux sujets qui vous intéressent. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingCustomWebSearch).

Utilisez la bibliothèque cliente Recherche personnalisée Bing pour C# pour :
* Rechercher des résultats de recherche sur le web à partir de votre instance Recherche personnalisée Bing.

[Documentation de référence](/dotnet/api/overview/azure/cognitiveservices/bing-custom-search-readme?view=azure-dotnet) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Search.BingCustomSearch) | [Package (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0) | [Exemples](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples)


## <a name="prerequisites"></a>Prérequis

- Une instance Recherche personnalisée Bing. Consultez [Démarrage rapide : Créer votre première instance Recherche personnalisée Bing](../../quick-start.md) pour plus amples informations.
- Microsoft [.NET Core](https://www.microsoft.com/net/download/core)
- N’importe quelle édition de [Visual Studio 2017 ou ultérieure](https://www.visualstudio.com/downloads/)
- Si vous utilisez Linux/MacOS, cette application peut être exécutée à l’aide de [Mono](https://www.mono-project.com/).
- Le package NuGet de [Recherche personnalisée Bing](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0). 
    - Dans l’**Explorateur de solutions** de Visual Studio, cliquez avec le bouton droit sur votre projet et sélectionnez **Gérer les packages NuGet** dans le menu. Installez le package `Microsoft.Azure.CognitiveServices.Search.CustomSearch`. L’installation du package NuGet Custom Search installe aussi les assemblys suivants :
        - Microsoft.Rest.ClientRuntime
        - Microsoft.Rest.ClientRuntime.Azure
        - Newtonsoft.Json

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](~/includes/cognitive-services-bing-custom-search-signup-requirements.md)]


## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

1. Créez une application console C# dans Visual Studio. Ajoutez ensuite les packages suivants à votre projet.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.Azure.CognitiveServices.Search.CustomSearch;
    ```

2. Dans la méthode principale de votre application, instanciez le client de recherche avec votre clé API.

    ```csharp
    var client = new CustomSearchAPI(new ApiKeyServiceClientCredentials("YOUR-SUBSCRIPTION-KEY"));
    ```

## <a name="send-the-search-request-and-receive-a-response"></a>Envoyer la requête de recherche et recevoir une réponse
    
1. Envoyez une requête de recherche à l’aide de la méthode `SearchAsync()` de votre client et enregistrez la réponse. Veillez à remplacer votre `YOUR-CUSTOM-CONFIG-ID` par l’ID de configuration de votre instance (vous trouvez l’ID dans le [portail Recherche personnalisée Bing](https://www.customsearch.ai/)). Cet exemple recherche « Xbox ».

    ```csharp
    // This will look up a single query (Xbox).
    var webData = client.CustomInstance.SearchAsync(query: "Xbox", customConfig: Int32.Parse("YOUR-CUSTOM-CONFIG-ID")).Result;
    ```

2. La méthode `SearchAsync()` retourne un objet `WebData`. Utilisez l’objet pour itérer au sein des `WebPages` qui ont été trouvés. Ce code recherche le premier résultat de page web et affiche le `Name` et l’`URL` de la page web.

    ```csharp
    if (webData?.WebPages?.Value?.Count > 0)
    {
        // find the first web page
        var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

        if (firstWebPagesResult != null)
        {
            Console.WriteLine("Number of webpage results {0}", webData.WebPages.Value.Count);
            Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
            Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
        }
        else
        {
            Console.WriteLine("Couldn't find web results!");
        }
    }
    else
    {
        Console.WriteLine("Didn't see any Web data..");
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web Recherche personnalisée](../../tutorials/custom-search-web-page.md)