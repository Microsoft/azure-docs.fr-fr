---
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 11/09/2018
ms.author: clemensv
ms.openlocfilehash: 9d4f7faa18ee7fae158afb42b8c42287e61dd103
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177650"
---
### <a name="create-a-nodejs-application"></a>Création d’une application Node.js

Créez un fichier JavaScript appelé `sender.js`.

### <a name="add-the-relay-npm-package"></a>Ajout du package NPM de relais

Exécutez `npm install hyco-ws` à partir d’une invite de commandes de nœud dans votre dossier de projet.

### <a name="write-some-code-to-send-messages"></a>Écriture de code pour envoyer des messages

1. Ajoutez le `constants` suivant au début du fichier `sender.js`.
   
    ```js
    const WebSocket = require('hyco-ws');
    const readline = require('readline')
        .createInterface({
            input: process.stdin,
            output: process.stdout
        });;
    ```
2. Ajoutez les constantes suivantes au fichier `sender.js` pour les détails de la connexion hybride. Remplacez les espaces réservés entre crochets par les valeurs obtenues lors de la création de la connexion hybride.
   
   1. `const ns` - L’espace de noms du relais. Veillez à utiliser le nom de l’espace de noms complet, par exemple `{namespace}.servicebus.windows.net`.
   2. `const path` - Le nom de la connexion hybride.
   3. `const keyrule` - Le nom de la clé SAS.
   4. `const key` - La valeur de la clé SAS.

3. Ajoutez le code suivant au fichier `sender.js` :
   
    ```js
    WebSocket.relayedConnect(
        WebSocket.createRelaySendUri(ns, path),
        WebSocket.createRelayToken('http://'+ns, keyrule, key),
        function (wss) {
            readline.on('line', (input) => {
                wss.send(input, null);
            });
   
            console.log('Started client interval.');
            wss.on('close', function () {
                console.log('stopping client interval');
                process.exit();
            });
        }
    );
    ```
    Voici à quoi doit ressembler votre fichier sender.js :
   
    ```js
    const WebSocket = require('hyco-ws');
    const readline = require('readline')
        .createInterface({
            input: process.stdin,
            output: process.stdout
        });;
   
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    WebSocket.relayedConnect(
        WebSocket.createRelaySendUri(ns, path),
        WebSocket.createRelayToken('http://'+ns, keyrule, key),
        function (wss) {
            readline.on('line', (input) => {
                wss.send(input, null);
            });
   
            console.log('Started client interval.');
            wss.on('close', function () {
                console.log('stopping client interval');
                process.exit();
            });
        }
    );
    ```

