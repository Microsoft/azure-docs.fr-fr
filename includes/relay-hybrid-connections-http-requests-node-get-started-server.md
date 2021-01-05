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
ms.openlocfilehash: fdd4e26839661ab9765b7d496e7f60c3686ba637
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177300"
---
### <a name="create-a-nodejs-application"></a>Création d’une application Node.js

Créez un fichier JavaScript appelé `listener.js`.

### <a name="add-the-relay-npm-package"></a>Ajout du package NPM de relais

Exécutez `npm install hyco-https` à partir d’une invite de commandes de nœud dans votre dossier de projet.

### <a name="write-some-code-to-handle-requests"></a>Écrire du code pour gérer les requêtes

1. Ajoutez la constante suivante au début du fichier `listener.js`.

    ```js
    const https = require('hyco-https');
    ```
2. Ajoutez les constantes suivantes au fichier `listener.js` pour les détails de la connexion hybride. Remplacez les espaces réservés entre crochets par les valeurs obtenues lors de la création de la connexion hybride.

   1. `const ns` - L’espace de noms du relais. Veillez à utiliser le nom de l’espace de noms complet, par exemple `{namespace}.servicebus.windows.net`.
   2. `const path` - Le nom de la connexion hybride.
   3. `const keyrule` - Le nom de la clé SAS.
   4. `const key` - La valeur de la clé SAS.

3. Ajoutez le code suivant au fichier `listener.js`. :

    Vous remarquez que le code n’est pas tellement différent d’un simple exemple de serveur HTTP que l’on peut trouver dans les didacticiels Node.js pour débutant, avec l’exception de l’utilisation de `createRelayedServer` de la fonction `createServer` typique.

    ```js
    var uri = https.createRelayListenUri(ns, path);
    var server = https.createRelayedServer(
        {
            server : uri,
            token : () => https.createRelayToken(uri, keyrule, key)
        },
        (req, res) => {
            console.log('request accepted: ' + req.method + ' on ' + req.url);
            res.setHeader('Content-Type', 'text/html');
            res.end('<html><head><title>Hey!</title></head><body>Relayed Node.js Server!</body></html>');
        });

    server.listen( (err) => {
            if (err) {
              return console.log('something bad happened', err)
            }
            console.log(`server is listening on ${port}`)
          });

    server.on('error', (err) => {
        console.log('error: ' + err);
    });
    ```
    Voici à quoi doit ressembler votre fichier listener.js :
   
    ```js
    const https = require('hyco-https');
   
    const ns = "{RelayNamespace}";
    const path = "{HybridConnectionName}";
    const keyrule = "{SASKeyName}";
    const key = "{SASKeyValue}";
   
    var uri = https.createRelayListenUri(ns, path);
    var server = https.createRelayedServer(
        {
            server : uri,
            token : () => https.createRelayToken(uri, keyrule, key)
        },
        (req, res) => {
            console.log('request accepted: ' + req.method + ' on ' + req.url);
            res.setHeader('Content-Type', 'text/html');
            res.end('<html><head><title>Hey!</title></head><body>Relayed Node.js Server!</body></html>');
        });

    server.listen( (err) => {
            if (err) {
              return console.log('something bad happened', err)
            }
            console.log(`server is listening on ${port}`)
          });

    server.on('error', (err) => {
        console.log('error: ' + err);
    });
    ```

