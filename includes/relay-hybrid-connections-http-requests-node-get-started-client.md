---
title: Fichier Include
description: Fichier Include
services: service-bus-relay
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 05/02/2018
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: 3c18efa7eb520b765c9bb3c2aff00104f971f5a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177301"
---
### <a name="create-a-nodejs-application"></a>Création d’une application Node.js

Si vous avez désactivé l’option « Nécessite l’autorisation client » lors de la création du Relai, vous pouvez envoyer des requêtes à l’URL de Connexions hybrides avec n’importe quel navigateur. Pour accéder à des points de terminaison protégés, vous devez créer et mettre un jeton dans l’en-tête `ServiceBusAuthorization`, montré ici.

Pour démarrer, créer un fichier JavaScript nommé `sender.js`.

### <a name="add-the-relay-npm-package"></a>Ajout du package NPM de relais

Exécutez `npm install hyco-https` à partir d’une invite de commandes de nœud dans votre dossier de projet. Ce package importe aussi le package normal `https`. Du côté du client, la différence clé est que le package fournit des fonctions pour construire des URI et des jetons de relai.

### <a name="write-some-code-to-send-messages"></a>Écriture de code pour envoyer des messages

1. Ajoutez le `constants` suivant au début du fichier `sender.js`.
   
    ```js
    const https = require('hyco-https');
    ```

2. Ajoutez les constantes suivantes au fichier `sender.js` pour les détails de la connexion hybride. Remplacez les espaces réservés entre crochets par les valeurs obtenues lors de la création de la connexion hybride.
   
   1. `const ns` - L’espace de noms du relais. Veillez à utiliser le nom de l’espace de noms complet, par exemple `{namespace}.servicebus.windows.net`.
   2. `const path` - Le nom de la connexion hybride.
   3. `const keyrule` - Le nom de la clé SAS.
   4. `const key` - La valeur de la clé SAS.

3. Ajoutez le code suivant au fichier `sender.js`. Vous remarquez que le code n’est pas tellement différent de l’utilisation normale du client HTTPS Node.js. Il ajoute juste l’en-tête d’autorisation.
   
    ```js
   https.get({
        hostname : ns,
        path : (!path || path.length == 0 || path[0] !== '/'?'/':'') + path,
        port : 443,
        headers : {
            'ServiceBusAuthorization' : 
                https.createRelayToken(https.createRelayHttpsUri(ns, path), keyrule, key)
        }
    }, (res) => {
        let error;
        if (res.statusCode !== 200) {
            console.error('Request Failed.\n Status Code: ${statusCode}');
            res.resume();
        } 
        else {
            res.setEncoding('utf8');
            res.on('data', (chunk) => {
                console.log(`BODY: ${chunk}`);
            });
            res.on('end', () => {
                console.log('No more data in response.');
            });
        };
    }).on('error', (e) => {
        console.error(`Got error: ${e.message}`);
    });
    ```
    Voici à quoi doit ressembler votre fichier sender.js :
   
    ```js
    const https = require('hyco-https');
       
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    https.get({
        hostname : ns,
        path : (!path || path.length == 0 || path[0] !== '/'?'/':'') + path,
        port : 443,
        headers : {
            'ServiceBusAuthorization' : 
                https.createRelayToken(https.createRelayHttpsUri(ns, path), keyrule, key)
        }
    }, (res) => {
        let error;
        if (res.statusCode !== 200) {
            console.error('Request Failed.\n Status Code: ${statusCode}');
            res.resume();
        } 
        else {
            res.setEncoding('utf8');
            res.on('data', (chunk) => {
                console.log(`BODY: ${chunk}`);
            });
            res.on('end', () => {
                console.log('No more data in response.');
            });
        };
    }).on('error', (e) => {
        console.error(`Got error: ${e.message}`);
    });
    ```

