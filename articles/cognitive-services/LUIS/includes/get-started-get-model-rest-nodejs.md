---
title: Obtenir un modèle avec un appel REST en Node.js
titleSuffix: Azure Cognitive Services
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 06/03/2020
ms.custom: devx-track-js
ms.openlocfilehash: fd25ef6ff44f1d62d553db518f7389da0e5d5019
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91534581"
---
[Documentation de référence](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c45) | [Exemple](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/LUIS/node-model-with-rest/model.js)

## <a name="prerequisites"></a>Prérequis

* Langage de programmation [Node.js](https://nodejs.org/)
* [Visual Studio Code](https://code.visualstudio.com/)

## <a name="example-utterances-json-file"></a>Exemples d’énoncés de fichier JSON

[!INCLUDE [Quickstart explanation of example utterance JSON file](get-started-get-model-json-example-utterances.md)]

## <a name="create-the-nodejs-project"></a>Créer le projet Node.js

1. Créez un dossier pour stocker votre projet Node.js, par exemple `node-model-with-rest`.

1. Ouvrez une nouvelle invite de commandes, accédez au dossier que vous avez créé et exécutez la commande suivante :

    ```console
    npm init
    ```

    Appuyez sur Entrée à chaque invite pour accepter les paramètres par défaut.

1. Installez le module request-promise en entrant la commande suivante :

    ```console
    npm install --save request
    npm install --save request-promise
    npm install --save querystring
    ```

## <a name="change-model-programmatically"></a>Changer le modèle programmatiquement

1. Créez un nouveau fichier appelé `model.js`. Ajoutez le code suivant :

    [!code-javascript[Code snippet](~/cognitive-services-quickstart-code/javascript/LUIS/node-model-with-rest/model.js)]

1. Remplacez les valeurs commençant par `YOUR-` par vos propres valeurs.

    |Information|Objectif|
    |--|--|
    |`YOUR-APP-ID`| Votre ID d’application LUIS. |
    |`YOUR-AUTHORING-KEY`|Votre clé de création (32 caractères).|
    |`YOUR-AUTHORING-ENDPOINT`| L’URL de votre point de terminaison de création. Par exemple : `https://replace-with-your-resource-name.api.cognitive.microsoft.com/`. Vous avez défini le nom de votre ressource au moment de sa création.|

    Les ressources et les clés affectées sont visibles dans le portail LUIS, dans la section Gérer de la page **Ressources Azure**. L’ID d’application est indiqué dans la section Gérer de la page **Paramètres de l’application**.

1. À l’invite de commandes, entrez la commande suivante pour exécuter le projet :

    ```console
    node model.js
    ```

1. Passez en revue la réponse de création :

    ```json
    addUtterance:
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
    train POST:
    {
      "statusId": 9,
      "status": "Queued"
    }
    train GET:
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
    done
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous aurez fini de suivre ce guide de démarrage rapide, supprimez le dossier de projet du système de fichiers.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Meilleures pratiques pour une application](../luis-concept-best-practices.md)