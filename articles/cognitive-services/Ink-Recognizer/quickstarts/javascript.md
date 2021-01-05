---
title: 'Démarrage rapide : Reconnaître l’encre numérique avec l’API REST Ink Recognizer et Node.js'
titleSuffix: Azure Cognitive Services
description: Utilisez l’API Ink Recognizer et JavaScript pour commencer la reconnaissance des traits d’encre numérique dans ce guide de démarrage rapide.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: ca1c18fd1ea8507c8b6a58b5f786b9686b1f3a34
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369072"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-javascript"></a>Démarrage rapide : Reconnaître l’encre numérique avec l’API REST Ink Recognizer et JavaScript

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

Utilisez ce guide de démarrage rapide pour commencer à appliquer l’API Ink Recognizer aux traits d’encre numérique. Cette application JavaScript envoie une requête d’API contenant des données de trait d’encre au format JSON, puis affiche la réponse.

Alors que cette application est écrite en JavaScript et s’exécute dans votre navigateur web, l’API est un service web RESTful compatible avec la plupart des langages de programmation.

En règle générale, vous appelez l’API à partir d’une application d’écriture manuscrite numérique. Ce guide de démarrage rapide envoie des données de trait d’encre pour l’exemple d’écriture manuscrite suivant à partir d’un fichier JSON.

![une image de texte manuscrit](../media/handwriting-sample.jpg)

Le code source de ce guide de démarrage rapide est disponible sur [GitHub](https://go.microsoft.com/fwlink/?linkid=2089905).

## <a name="prerequisites"></a>Prérequis

- Un navigateur web
- Vous trouverez l’exemple de données de trait d’encre utilisé dans ce guide de démarrage rapide sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/javascript/InkRecognition/quickstart/example-ink-strokes.json).

### <a name="create-an-ink-recognizer-resource"></a>Créer une ressource Ink Recognizer

[!INCLUDE [creating an ink recognizer resource](../includes/setup-instructions.md)]

## <a name="create-a-new-application"></a>Créer une application

1. Créez un fichier `.html` dans l’éditeur ou l’IDE de votre choix. Puis ajoutez du HTML de base pour le code que nous ajouterons plus tard.
    
    ```html
    <!DOCTYPE html>
    <html>
    
        <head>
            <script type="text/javascript">
            </script>
        </head>
        
        <body>
        </body>
    
    </html>
    ```

2. Dans la balise `<body>`, ajoutez le code suivant :
    1. Deux zones de texte pour l’affichage de la requête JSON et la réponse.
    2. Un bouton pour appeler la fonction `recognizeInk()` qui est créée ultérieurement.
    
    ```HTML
    <!-- <body>-->
        <h2>Send a request to the Ink Recognition API</h2>
        <p>Request:</p>
        <textarea id="request" style="width:800px;height:300px"></textarea>
        <p>Response:</p>
        <textarea id="response" style="width:800px;height:300px"></textarea>
        <br>
        <button type="button" onclick="recognizeInk()">Recognize Ink</button>
    <!--</body>-->
    ```

## <a name="load-the-example-json-data"></a>Charger l’exemple de données JSON

1. Dans la balise `<script>`, créez une variable pour sampleJson. Puis créez une fonction JavaScript nommée `openFile()` qui ouvre un Explorateur de fichiers pour vous permettre de sélectionner votre fichier JSON. Lorsque le bouton `Recognize ink` est activé, il appelle cette fonction et commence la lecture du fichier.
2. Utilisez la fonction `onload()` d’un objet `FileReader` pour traiter le fichier de façon asynchrone. 
    1. Remplacer les caractères `\n` ou `\r` dans le fichier par une chaîne vide. 
    2. Utilisez `JSON.parse()` pour convertir le texte en JSON valide
    3. Mettez à jour la zone de texte `request` dans l’application. Utilisez `JSON.stringify()` pour formater la chaîne JSON. 
    
    ```javascript
    var sampleJson = "";
    function openFile(event) {
        var input = event.target;
    
        var reader = new FileReader();
        reader.onload = function(){
            sampleJson = reader.result.replace(/(\\r\\n|\\n|\\r)/gm, "");
            sampleJson = JSON.parse(sampleJson);
            document.getElementById('request').innerHTML = JSON.stringify(sampleJson, null, 2);
        };
        reader.readAsText(input.files[0]);
    };
    ```

## <a name="send-a-request-to-the-ink-recognizer-api"></a>Envoyer une requête à l’API Ink Recognizer

1. Au sein de la balise `<script>`, créez une fonction nommée `recognizeInk()`. Cette fonction appellera ultérieurement l’API et mettra à jour la page avec la réponse. Ajoutez le code obtenu lors des étapes précédentes à cette fonction. 
        
    ```javascript
    function recognizeInk() {
    // add the code from the below steps here 
    }
    ```

    1. Créez des variables pour votre URL de point de terminaison, clé d’abonnement et exemple JSON. Créez ensuite un objet `XMLHttpRequest` pour envoyer la requête d’API. 
        
        ```javascript
        // Replace the below URL with the correct one for your subscription. 
        // Your endpoint can be found in the Azure portal. For example: "https://<your-custom-subdomain>.cognitiveservices.azure.com";
        var SERVER_ADDRESS = process.env["INK_RECOGNITION_ENDPOINT"];
        var ENDPOINT_URL = SERVER_ADDRESS + "/inkrecognizer/v1.0-preview/recognize";
        var SUBSCRIPTION_KEY = process.env["INK_RECOGNITION_SUBSCRIPTION_KEY"];
        var xhttp = new XMLHttpRequest();
        ```
    2. Créez la fonction return pour l’objet `XMLHttpRequest`. Cette fonction analysera la réponse de l’API à partir d’une requête réussie et l’affichera dans l’application. 
            
        ```javascript
        function returnFunction(xhttp) {
            var response = JSON.parse(xhttp.responseText);
            console.log("Response: %s ", response);
            document.getElementById('response').innerHTML = JSON.stringify(response, null, 2);
        }
        ```
    3. Créez la fonction error pour l’objet request. Cette fonction journalise l’erreur dans la console. 
            
        ```javascript
        function errorFunction() {
            console.log("Error: %s, Detail: %s", xhttp.status, xhttp.responseText);
        }
        ```

    4. Créez une fonction pour la propriété `onreadystatechange` de l’objet de la requête. Lors du changement de l’état de préparation de l’objet de la requête, les fonctions return et error précédentes seront appliquées.
            
        ```javascript
        xhttp.onreadystatechange = function () {
            if (this.readyState === 4) {
                if (this.status === 200) {
                    returnFunction(xhttp);
                } else {
                    errorFunction(xhttp);
                }
            }
        };
        ```
    
    5. Envoyez la requête d’API. Ajoutez votre clé d’abonnement à l’en-tête `Ocp-Apim-Subscription-Key`, puis définissez le `content-type` sur `application/json`.
    
        ```javascript
        xhttp.open("PUT", ENDPOINT_URL, true);
        xhttp.setRequestHeader("Ocp-Apim-Subscription-Key", SUBSCRIPTION_KEY);
        xhttp.setRequestHeader("content-type", "application/json");
        xhttp.send(JSON.stringify(sampleJson));
        };
        ```

## <a name="run-the-application-and-view-the-response"></a>Exécuter l’application et afficher la réponse

Cette application peut être exécutée au sein de votre navigateur web. Une réponse correcte est retournée au format JSON. Vous pouvez également trouver la réponse JSON sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/javascript/InkRecognition/quickstart/example-response.json):

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Référence d’API REST](/rest/api/cognitiveservices/inkrecognizer/inkrecognizer)

Pour observer le fonctionnement de l’API Ink Recognizer dans une application d’écriture manuscrite numérique, examinez les exemples d’applications suivants sur GitHub :
* [C# et UWP (plateforme Windows universelle)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# et WPF (Windows Presentation Foundation)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Application de navigateur web JavaScript](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Application mobile Android et Java](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Application mobile Swift et iOS](https://go.microsoft.com/fwlink/?linkid=2089805)