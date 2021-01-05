---
title: 'Démarrage rapide : Bibliothèque de client C# Recherche de vidéos Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/19/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 6d50a8e2c9d0263616b25e25958be6a6f0fb7fe1
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88929265"
---
Utilisez ce guide de démarrage rapide pour démarrer une recherche de vidéos avec la bibliothèque de client Recherche de vidéos Bing pour C#. Si l’outil Recherche de vidéos Bing dispose d’une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingVideoSearch) avec des fonctionnalités et annotations supplémentaires.

## <a name="prerequisites"></a>Prérequis

* N’importe quelle édition de [Visual Studio 2017 ou ultérieure](https://visualstudio.microsoft.com/downloads/).
* Le framework Json.NET, disponible [sous forme de package NuGet](https://www.nuget.org/packages/Newtonsoft.Json/).

Pour ajouter la bibliothèque de client Recherche de vidéos Bing à votre projet, sélectionnez **Gérer les packages NuGet** à partir de l’**Explorateur de solutions** dans Visual Studio. Ajoutez le package `Microsoft.Azure.CognitiveServices.Search.VideoSearch`.

L’installation du [[package NuGet du SDK Recherche de vidéos]](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.VideoSearch/1.2.0) installe également les dépendances suivantes :

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](~/includes/cognitive-services-bing-video-search-signup-requirements.md)]


## <a name="create-and-initialize-a-project"></a>Créer et initialiser un projet

1. Créez une solution console en C# dans Visual Studio. Ajoutez ensuite les éléments suivants dans le fichier de code principal.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Azure.CognitiveServices.Search.VideoSearch;
    using Microsoft.Azure.CognitiveServices.Search.VideoSearch.Models;
    ```

2. Instanciez le client en créant un objet `ApiKeyServiceClientCredentials` avec votre clé d’abonnement, puis en appelant le constructeur.

    ```csharp
    var client = new VideoSearchAPI(new ApiKeyServiceClientCredentials("YOUR-ACCESS-KEY"));
    ```

## <a name="send-a-search-request-and-process-the-results"></a>Envoyer une requête de recherche et traiter les résultats

1. Utilisez le client pour envoyer une requête de recherche. Utilisez « SwiftKey » pour la requête de recherche.

    ```csharp
    var videoResults = client.Videos.SearchAsync(query: "SwiftKey").Result;
    ```

2. Si des résultats sont retournés, obtenez le premier avec `videoResults.Value[0]`. Ensuite, affichez l’ID, le titre et l’URL de la vidéo.

    ```csharp
    if (videoResults.Value.Count > 0)
    {
        var firstVideoResult = videoResults.Value[0];

        Console.WriteLine($"\r\nVideo result count: {videoResults.Value.Count}");
        Console.WriteLine($"First video id: {firstVideoResult.VideoId}");
        Console.WriteLine($"First video name: {firstVideoResult.Name}");
        Console.WriteLine($"First video url: {firstVideoResult.ContentUrl}");
    }
    else
    {
        Console.WriteLine("Couldn't find video results!");
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-video-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi 

* [Qu’est-ce que l’API Recherche de vidéos Bing ?](../../overview.md)
* [Exemples du Kit de développement logiciel (SDK) .NET pour Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
