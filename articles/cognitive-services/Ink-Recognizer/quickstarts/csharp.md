---
title: 'Démarrage rapide : Reconnaître l’encre numérique avec l’API REST Ink Recognizer et C#'
titleSuffix: Azure Cognitive Services
description: Ce guide de démarrage rapide montre comment utiliser l’API Ink Recognizer et C# pour commencer la reconnaissance des traits d’encre numérique.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 13a2d887b17ff319fb3a0f2bb0d5d0ff04629088
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369101"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-c"></a>Démarrage rapide : Reconnaître l’encre numérique avec l’API REST Ink Recognizer et C#

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

Utilisez ce guide de démarrage rapide pour commencer à envoyer des traits d’encre numérique à l’API Ink Recognizer. Cette application C# envoie une requête d’API contenant des données de trait d’encre au format JSON, puis obtient la réponse.

Bien que cette application soit écrite en C#, l’API est un service web RESTful compatible avec la plupart des langages de programmation.

En règle générale, vous appelez l’API à partir d’une application d’écriture manuscrite numérique. Ce guide de démarrage rapide envoie des données de trait d’encre pour l’exemple d’écriture manuscrite suivant à partir d’un fichier JSON.

![une image de texte manuscrit](../media/handwriting-sample.jpg)

Le code source de ce guide de démarrage rapide est disponible sur [GitHub](https://go.microsoft.com/fwlink/?linkid=2089502).

## <a name="prerequisites"></a>Prérequis

- N’importe quelle édition de [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/).
- [Newtonsoft.Json](https://www.newtonsoft.com/json)
    - Pour installer Newtonsoft.Json en tant que package NuGet dans Visual Studio :
        1. Cliquez avec le bouton droit sur le **Gestionnaire de solution**.
        2. Cliquez sur **Gérer les packages NuGet...**
        3. Recherchez `Newtonsoft.Json` et installez le package.
- Si vous utilisez Linux/MacOS, cette application peut être exécutée à l’aide de [Mono](https://www.mono-project.com/).

- Vous trouverez l’exemple de données de trait d’encre utilisé dans ce guide de démarrage rapide sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Vision/InkRecognition/quickstart/example-ink-strokes.json).

### <a name="create-an-ink-recognizer-resource"></a>Créer une ressource Ink Recognizer

[!INCLUDE [creating-an-ink-recognizer-resource](../includes/setup-instructions.md)]

## <a name="create-a-new-application"></a>Créer une application

1. Dans Visual Studio, créez une solution de console et ajoutez les packages suivants. 
    
    [!code-csharp[import statements](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=imports)]

2. Créez des variables pour votre point de terminaison et votre clé d’abonnement, ainsi que l’exemple de fichier JSON. Le point de terminaison sera combiné ultérieurement avec `inkRecognitionUrl` pour accéder à l’API. 

    [!code-csharp[endpoint file and key variables](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=vars)]

## <a name="create-a-function-to-send-requests"></a>Créer une fonction pour envoyer des requêtes

1. Créez une fonction asynchrone appelée `Request` qui prend les variables créées plus haut.

2. Définissez le protocole de sécurité et les informations d’en-tête du client à l’aide d’un objet `HttpClient`. Veillez à ajouter votre clé d’abonnement à l’en-tête `Ocp-Apim-Subscription-Key`. Créez ensuite un objet `StringContent` pour la requête.
 
3. Ensuite, envoyez la requête avec `PutAsync()`. Si la requête réussit, retournez la réponse.  
    
    [!code-csharp[request example method](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=request)]

## <a name="send-an-ink-recognition-request"></a>Envoyer une requête de reconnaissance de l’écriture manuscrite

1. Créez une fonction appelée `recognizeInk()`. Construisez la requête et envoyez-la en appelant la fonction `Request()` avec votre point de terminaison, la clé d’abonnement, l’URL de l’API et les données du trait d’encre numérique.

2. Désérialisez l’objet JSON et écrivez-le dans la console. 
    
    [!code-csharp[request to recognize ink data](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=recognize)]

## <a name="load-your-digital-ink-data"></a>Charger vos données d’encre numérique

Créez une fonction appelée `LoadJson()` pour charger le fichier JSON de données d’encre. Utilisez un `StreamReader` et un `JsonTextReader` pour créer un `JObject` et retournez-le.

[!code-csharp[load the JSON file](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=loadJson)]

## <a name="send-the-api-request"></a>Envoyer la requête d’API

1. Dans la méthode Main de votre application, chargez vos données JSON avec la fonction créée plus haut. 

2. Appelez la fonction `recognizeInk()` créée plus haut. Utilisez `System.Console.ReadKey()` pour maintenir la fenêtre de console ouverte après l’exécution de l’application.
    
    [!code-csharp[file main method](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=main)]


## <a name="run-the-application-and-view-the-response"></a>Exécuter l’application et afficher la réponse

Exécutez l'application. Une réponse correcte est retournée au format JSON. Vous pouvez également trouver la réponse JSON sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Vision/InkRecognition/quickstart/example-response.json).


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Référence d’API REST](/rest/api/cognitiveservices/inkrecognizer/inkrecognizer)


Pour observer le fonctionnement de l’API Ink Recognizer dans une application d’écriture manuscrite numérique, examinez les exemples d’applications suivants sur GitHub :
* [C# et UWP (plateforme Windows universelle)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# et WPF (Windows Presentation Foundation)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Application de navigateur web JavaScript](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Application mobile Android et Java](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Application mobile Swift et iOS](https://go.microsoft.com/fwlink/?linkid=2089805)