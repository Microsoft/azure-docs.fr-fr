---
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 11/09/2018
ms.author: clemensv
ms.openlocfilehash: 531866ece1c4acacb926c9e9c6598158c1aa077f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177653"
---
### <a name="create-a-nodejs-application"></a>Création d’une application Node.js

Créez un fichier JavaScript appelé `listener.js`.

### <a name="add-the-relay-npm-package"></a>Ajout du package NPM de relais

Exécutez `npm install hyco-ws` à partir d’une invite de commandes de nœud dans votre dossier de projet.

### <a name="write-some-code-to-receive-messages"></a>Écriture de code pour recevoir des messages

1. Ajoutez la constante suivante au début du fichier `listener.js`.
   
    ```js
    const WebSocket = require('hyco-ws');
    ```
2. Ajoutez les constantes suivantes au fichier `listener.js` pour les détails de la connexion hybride. Remplacez les espaces réservés entre crochets par les valeurs obtenues lors de la création de la connexion hybride.
   
   1. `const ns` - L’espace de noms du relais. Veillez à utiliser le nom de l’espace de noms complet, par exemple `{namespace}.servicebus.windows.net`.
   2. `const path` - Le nom de la connexion hybride.
   3. `const keyrule` - Le nom de la clé SAS.
   4. `const key` - La valeur de la clé SAS.

3. Ajoutez le code suivant au fichier `listener.js` :
   
    ```js
    var wss = WebSocket.createRelayedServer(
    {
        server : WebSocket.createRelayListenUri(ns, path),
        token: WebSocket.createRelayToken('http://' + ns, keyrule, key)
    }, 
    function (ws) {
        console.log('connection accepted');
        ws.onmessage = function (event) {
            console.log(event.data);
        };
        ws.on('close', function () {
            console.log('connection closed');
        });       
    });
   
    console.log('listening');
   
    wss.on('error', function(err) {
        console.log('error' + err);
    });
    ```
    Voici à quoi doit ressembler votre fichier listener.js :
   
    ```js
    const WebSocket = require('hyco-ws');
   
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    var wss = WebSocket.createRelayedServer(
        {
            server : WebSocket.createRelayListenUri(ns, path),
            token: WebSocket.createRelayToken('http://' + ns, keyrule, key)
        }, 
        function (ws) {
            console.log('connection accepted');
            ws.onmessage = function (event) {
                console.log(event.data);
            };
            ws.on('close', function () {
                console.log('connection closed');
            });       
    });
   
    console.log('listening');
   
    wss.on('error', function(err) {
        console.log('error' + err);
    });
    ```

