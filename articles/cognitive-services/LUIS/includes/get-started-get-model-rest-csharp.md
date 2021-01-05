---
title: Obtenir un modèle avec un appel REST en C#
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 06/03/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: aaf7658796c50bb30305b5fda0527141cb820289
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91534579"
---
[Documentation de référence](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [Exemple](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/LanguageUnderstanding/csharp-model-with-rest/Program.cs)

## <a name="prerequisites"></a>Prérequis

* [.NET Core 3.1](https://dotnet.microsoft.com/download)
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>Exemples d’énoncés de fichier JSON

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-pizza-app"></a>Créer une application de pizza

[!INCLUDE [Create pizza app](get-started-get-model-create-pizza-app.md)]

## <a name="change-model-programmatically"></a>Changer le modèle programmatiquement

1. Créez une application console ciblant le langage C#, avec un projet et le nom de dossier `csharp-model-with-rest`.

    ```console
    dotnet new console -lang C# -n csharp-model-with-rest
    ```

1. Accédez au répertoire `csharp-model-with-rest` que vous avez créé et installez les dépendances nécessaires avec les commandes suivantes :

    ```console
    cd csharp-model-with-rest
    dotnet add package System.Net.Http
    dotnet add package JsonFormatterPlus
    ```

1. Remplacez le contenu de Program.cs par le code suivant :

    [!code-csharp[Code snippet](~/cognitive-services-quickstart-code/dotnet/LanguageUnderstanding/csharp-model-with-rest/Program.cs)]

1. Remplacez les valeurs commençant par `YOUR-` par vos propres valeurs.

    |Information|Objectif|
    |--|--|
    |`YOUR-APP-ID`| Votre ID d’application LUIS. |
    |`YOUR-AUTHORING-KEY`|Votre clé de création (32 caractères).|
    |`YOUR-AUTHORING-ENDPOINT`| L’URL de votre point de terminaison de création. Par exemple : `https://replace-with-your-resource-name.api.cognitive.microsoft.com/`. Vous avez défini le nom de votre ressource au moment de sa création.|

    Les ressources et les clés affectées sont visibles dans le portail LUIS, dans la section Gérer de la page **Ressources Azure**. L’ID d’application est indiqué dans la section Gérer de la page **Paramètres de l’application**.

1. Générez l'application console.

    ```console
    dotnet build
    ```

1. Exécutez l’application console.

    ```console
    dotnet run
    ```

1. Passez en revue la réponse de création :

    ```console
    Added utterances.
    [
        {
            "value": {
                "ExampleId": 1137150691,
                "UtteranceText": "order a pizza"
            },
            "hasError": false
        },
        {
            "value": {
                "ExampleId": 1137150692,
                "UtteranceText": "order a large pepperoni pizza"
            },
            "hasError": false
        },
        {
            "value": {
                "ExampleId": 1137150693,
                "UtteranceText": "i want two large pepperoni pizzas on thin crust"
            },
            "hasError": false
        }
    ]
    Sent training request.
    {
        "statusId": 9,
        "status": "Queued"
    }
    Requested training status.
    [
        {
            "modelId": "edb46abf-0000-41ab-beb2-a41a0fe1630f",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "a5030be2-616c-4648-bf2f-380fa9417d37",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "3f2b1f31-a3c3-4fbd-8182-e9d9dbc120b9",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "e4b6704b-1636-474c-9459-fe9ccbeba51c",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "031d3777-2a00-4a7a-9323-9a3280a30000",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        },
        {
            "modelId": "9250e7a1-06eb-4413-9432-ae132ed32583",
            "details": {
                "statusId": 9,
                "status": "Queued",
                "exampleCount": 0
            }
        }
    ]
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous aurez fini de suivre ce guide de démarrage rapide, supprimez le dossier de projet du système de fichiers.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Meilleures pratiques pour une application](../luis-concept-best-practices.md)
