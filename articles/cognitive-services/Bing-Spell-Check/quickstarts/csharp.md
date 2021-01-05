---
title: 'Démarrage rapide : Vérifier l’orthographe avec l’API REST et C# - Vérification orthographique Bing'
titleSuffix: Azure Cognitive Services
description: Commencez à utiliser l’API REST Vérification orthographique Bing et C# pour vérifier l’orthographe et la grammaire.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: quickstart
ms.date: 05/21/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: ace7a0ccaba533c9e72961536159d32af5ab8d98
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352726"
---
# <a name="quickstart-check-spelling-with-the-bing-spell-check-rest-api-and-c"></a>Démarrage rapide : Vérifier l’orthographe avec l’API REST Vérification orthographique Bing et C#

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Utilisez ce guide de démarrage rapide pour effectuer votre premier appel à l’API REST Vérification orthographique Bing. Cette simple application C# envoie une demande à l’API et retourne une liste de suggestions de corrections. 

Bien que cette application soit écrite en C#, l’API est un service web RESTful compatible avec la plupart des langages de programmation. Le code source de cette application est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Search/BingAutosuggestv7.cs).

## <a name="prerequisites"></a>Prérequis

* N’importe quelle édition de [Visual Studio 2017 ou ultérieure](https://www.visualstudio.com/downloads/).
* Le package NuGet Newtonsoft.Json. 
     
   Pour installer ce package dans Visual Studio :

     1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le fichier Solution.
     1. Sélectionnez **Gérer les packages NuGet pour la solution**.
     1. Recherchez *Newtonsoft.Json* et installez le package.

* Si vous utilisez Linux/MacOS, vous pouvez exécuter cette application en utilisant [Mono](https://www.mono-project.com/).

[!INCLUDE [cognitive-services-bing-spell-check-signup-requirements](../../../../includes/cognitive-services-bing-spell-check-signup-requirements.md)]

## <a name="create-and-initialize-a-project"></a>Créer et initialiser un projet

1. Créez une solution de console nommée SpellCheckSample dans Visual Studio. Ajoutez ensuite les espaces de noms suivants dans le fichier de code principal :
    
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Newtonsoft.Json;
    ```

2. Créez des variables pour le point de terminaison d’API, votre clé d’abonnement et le texte dont l’orthographe doit être vérifiée. Vous pouvez utiliser le point de terminaison global dans le code suivant, ou le point de terminaison de [sous-domaine personnalisé](../../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

    ```csharp
    namespace SpellCheckSample
    {
        class Program
        {
            static string host = "https://api.cognitive.microsoft.com";
            static string path = "/bing/v7.0/spellcheck?";
            static string key = "<ENTER-KEY-HERE>";
            //text to be spell-checked
            static string text = "Hollo, wrld!";
        }
    }
    ```

3. Créez une chaîne pour vos paramètres de recherche : 

   1. Attribuez le code de votre marché au paramètre `mkt` à l’aide de l’opérateur `=`. Le code du marché correspond au code du pays ou de la région depuis lesquels vous effectuez la demande. 

   1. Ajoutez le paramètre `mode` avec l’opérateur `&`, puis attribuez le mode de vérification orthographique. Le mode peut être soit `proof` (détecte la plupart des erreurs d’orthographe et de grammaire) ou `spell` (détecte la plupart des erreurs d’orthographe mais pas autant d’erreurs de grammaire).
    
    ```csharp
    static string params_ = "mkt=en-US&mode=proof";
    ```

## <a name="create-and-send-a-spell-check-request"></a>Créer et envoyer une demande de vérification orthographique

1. Créez une fonction asynchrone appelée `SpellCheck()` pour envoyer une demande à l’API. Créez un `HttpClient` et ajoutez votre clé d’abonnement à l’en-tête `Ocp-Apim-Subscription-Key`. Dans la fonction, suivez les étapes suivantes.

    ```csharp
    async static void SpellCheck()
    {
        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

        HttpResponseMessage response = null;
        // add the rest of the code snippets here (except for main())...
    }
    ```

2. Créez l’URI pour votre requête en ajoutant votre hôte, votre chemin d’accès et vos paramètres.
    
    ```csharp
    string uri = host + path + params_;
    ```

3. Créez une liste avec un objet `KeyValuePair` contenant votre texte que vous utilisez pour créer un objet `FormUrlEncodedContent`. Définissez les informations d’en-tête et utilisez `PostAsync()` pour envoyer la demande.

    ```csharp
    var values = new Dictionary<string, string>();
    values.Add("text", text);
    var content = new FormUrlEncodedContent(values);
    content.Headers.ContentType = new MediaTypeHeaderValue("application/x-www-form-urlencoded");
    response = await client.PostAsync(uri, new FormUrlEncodedContent(values));
    ```

## <a name="get-and-print-the-api-response"></a>Obtenir et imprimer la réponse de l’API

### <a name="get-the-client-id-header"></a>Obtenir l’en-tête ID client

Si la réponse contient un en-tête `X-MSEdge-ClientID`, obtenez la valeur et imprimez-la.

``` csharp
string client_id;
if (response.Headers.TryGetValues("X-MSEdge-ClientID", out IEnumerable<string> header_values))
{
    client_id = header_values.First();
    Console.WriteLine("Client ID: " + client_id);
}
```

### <a name="get-the-response"></a>Obtenir la réponse

Obtenez la réponse de l’API. Désérialisez l’objet JSON et imprimez-le dans la console.

```csharp
string contentString = await response.Content.ReadAsStringAsync();

dynamic jsonObj = JsonConvert.DeserializeObject(contentString);
Console.WriteLine(jsonObj);
```

## <a name="call-the-spell-check-function"></a>Appeler la fonction de vérification orthographique

Dans la fonction `Main()` de votre projet, appelez `SpellCheck()`.

```csharp
static void Main(string[] args)
{
    SpellCheck();
    Console.ReadLine();
}
```

## <a name="run-the-application"></a>Exécution de l'application

Créez et exécutez votre projet. Si vous utilisez Visual Studio, appuyez sur **F5** pour déboguer le fichier.

## <a name="example-json-response"></a>Exemple de réponse JSON

Une réponse correcte est retournée au format JSON, comme dans l’exemple suivant : 

```json
{
   "_type": "SpellCheck",
   "flaggedTokens": [
      {
         "offset": 0,
         "token": "Hollo",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "Hello",
               "score": 0.9115257530801
            },
            {
               "suggestion": "Hollow",
               "score": 0.858039839213461
            },
            {
               "suggestion": "Hallo",
               "score": 0.597385084464481
            }
         ]
      },
      {
         "offset": 7,
         "token": "wrld",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "world",
               "score": 0.9115257530801
            }
         ]
      }
   ]
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../tutorials/spellcheck.md)

- [Qu’est-ce que l’API Vérification orthographique Bing ?](../overview.md)
- [Informations de référence sur l’API Vérification orthographique Bing v7](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference)