---
title: 'Démarrage rapide : Bibliothèque de client JavaScript Recherche de vidéos Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/19/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: fa1ecb24649d9355e6104bff8add5430fdd64c3c
ms.sourcegitcommit: 5dbea4631b46d9dde345f14a9b601d980df84897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91377470"
---
Utilisez ce guide de démarrage rapide pour démarrer une recherche de vidéos avec la bibliothèque de client Recherche de vidéos Bing pour JavaScript. Si l’outil Recherche de vidéos Bing dispose d’une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/blob/master/Samples/videoSearch.js). Il contient d’autres fonctionnalités et annotations.

## <a name="prerequisites"></a>Prérequis

* La dernière version de [Node.js](https://nodejs.org/en/download/).
* Le [SDK Recherche de vidéos Bing pour JavaScript](https://www.npmjs.com/package/@azure/cognitiveservices-videosearch)
     *  Pour l’installer, exécutez `npm install @azure/cognitiveservices-videosearch`
* La classe `CognitiveServicesCredentials` du package `@azure/ms-rest-azure-js` pour authentifier le client.
     * Pour l’installer, exécutez `npm install @azure/ms-rest-azure-js`

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](~/includes/cognitive-services-bing-video-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

1. Créez un fichier JavaScript dans votre éditeur ou IDE favori, puis ajoutez une instruction `require()` pour la bibliothèque de client Recherche de vidéos Bing et le module `CognitiveServicesCredentials`. Créez une variable pour votre clé d’abonnement. 
    
    ```javascript
    const CognitiveServicesCredentials = require('@azure/ms-rest-azure-js').CognitiveServicesCredentials;
    const VideoSearchAPIClient = require('@azure/cognitiveservices-videosearch');
    ```

2. Créez une instance de `CognitiveServicesCredentials` avec votre clé. Ensuite, utilisez-la pour créer une instance du client de recherche de vidéos.

    ```javascript
    let credentials = new CognitiveServicesCredentials('YOUR-ACCESS-KEY');
    let client = new VideoSearchAPIClient(credentials);
    ```

## <a name="send-the-search-request"></a>Envoyer la requête de recherche

1. Utilisez `client.videosOperations.search()` pour envoyer une requête de recherche à l’API Recherche de vidéos Bing. Quand les résultats de la recherche sont retournés, utilisez `.then()` pour journaliser le résultat.
    
    ```javascript
    client.videosOperations.search('Interstellar Trailer').then((result) => {
        console.log(result.value);
    }).catch((err) => {
        throw err;
    });
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-video-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi 

* [Qu’est-ce que l’API Recherche de vidéos Bing ?](../../overview.md)
* [Exemples du Kit de développement logiciel (SDK) .NET pour Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)