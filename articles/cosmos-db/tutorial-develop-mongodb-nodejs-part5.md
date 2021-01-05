---
title: Connecter l’application Angular à l’API d’Azure Cosmos DB pour MongoDB à l’aide de Mongoose
description: Ce didacticiel vous explique comment générer une application Node.js en utilisant Angular et Express pour gérer les données stockées dans Cosmos DB. Dans cette partie, vous allez utiliser Mongoose pour établir la connexion à Azure Cosmos DB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-js
ms.reviewer: sngun
ms.openlocfilehash: a3097fa539f460ef5e8ffe73598fa5d55516717e
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097802"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---use-mongoose-to-connect-to-cosmos-db"></a>Créer une application Angular avec l’API Azure Cosmos DB pour MongoDB - Utiliser Mongoose pour établir la connexion à Cosmos DB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Ce didacticiel en plusieurs parties montre comment créer une application Node.js avec Express et Angular, puis comment la connecter à votre [compte Cosmos configuré avec l’API Cosmos DB pour MongoDB](mongodb-introduction.md). Cet article contient la partie 5 du didacticiel et s’appuie sur la [partie 4](tutorial-develop-mongodb-nodejs-part4.md).

Dans cette partie du didacticiel, vous allez :

> [!div class="checklist"]
> * Utiliser Mongoose pour établir la connexion à Cosmos DB.
> * Obtenir votre chaîne de connexion Cosmos DB.
> * Créer le modèle héros.
> * Créer le service héros pour obtenir des données héros.
> * Exécutez l’application localement.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Avant de commencer ce didacticiel, terminez la procédure de [l’étape 4](tutorial-develop-mongodb-nodejs-part4.md).

* Ce didacticiel nécessite que vous exécutiez l’interface Azure CLI localement. Vous devez avoir installé Azure CLI 2.0 ou ultérieur. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau Azure CLI, consultez [Installer Azure CLI 2.0](/cli/azure/install-azure-cli).

* Ce didacticiel vous guide à travers les étapes pour générer l’application pas à pas. Si vous souhaitez télécharger le projet terminé, vous pouvez obtenir l’application complète à partir du [référentiel cosmosdb-angular](https://github.com/Azure-Samples/angular-cosmosdb) sur GitHub.

## <a name="use-mongoose-to-connect"></a>Utiliser Mongoose pour établir la connexion

Mongoose est une bibliothèque ODM (modèles de données objets) pour MongoDB et Node.js. Vous pouvez utiliser Mongoose pour établir la connexion au compte Azure Cosmos DB. Utilisez la procédure suivante pour installer Mongoose et établir la connexion à Azure Cosmos DB :

1. Installez le module npm mongoose. Cette API est utilisée pour communiquer avec MongoDB.

    ```bash
    npm i mongoose --save
    ```

1. Dans le dossier **server** , créez un fichier nommé **mongo.js**. Vous allez ajouter les détails de connexion de votre compte Azure Cosmos DB à ce fichier.

1. Copiez le code suivant dans le fichier **mongo.js**. Ce code fournit les fonctionnalités suivantes :

   * Requiert Mongoose.
   * Remplace la promesse Mongo consistant à utiliser la promesse de base intégrée à ES6/ES2015 et versions ultérieures.
   * Appelle un fichier env afin de configurer certaines choses si vous êtes chargé du processus de site, de la production ou du développement. Vous allez créer ce fichier dans la section suivante.
   * Ajoute la chaîne de connexion MongoDB définie dans le fichier env.
   * Crée une fonction de connexion nommée Mongoose.

     ```javascript
     const mongoose = require('mongoose');
     /**
     * Set to Node.js native promises
     * Per https://mongoosejs.com/docs/promises.html
     */
     mongoose.Promise = global.Promise;

     const env = require('./env/environment');

     // eslint-disable-next-line max-len
     const mongoUri = `mongodb://${env.accountName}:${env.key}@${env.accountName}.documents.azure.com:${env.port}/${env.databaseName}?ssl=true`;

     function connect() {
     mongoose.set('debug', true);
     return mongoose.connect(mongoUri, { useMongoClient: true });
     }

     module.exports = {
     connect,
     mongoose
     };
     ```
    
1. Dans le volet Explorateur, sous **server** , créez un dossier nommé **environment**. Dans le dossier **environment** , créez un fichier nommé **environment.js**.

1. À partir du fichier mongo.js, nous devons inclure des valeurs des paramètres `dbName`, `key`et `cosmosPort`. Copiez le code suivant dans le fichier **environment.js** :

    ```javascript
    // TODO: replace if yours are different
    module.exports = {
      accountName: 'your-cosmosdb-account-name-goes-here',
      databaseName: 'admin', 
      key: 'your-key-goes-here',
      port: 10255
    };
    ```

## <a name="get-the-connection-string"></a>Obtenir la chaîne de connexion

Pour connecter votre application à Azure Cosmos DB, vous devez mettre à jour les paramètres de configuration de l’application. Suivez la procédure suivante pour mettre à jour les paramètres : 

1. Dans le portail Azure, obtenez le numéro de port, le nom du compte Azure Cosmos DB et les valeurs de clé primaire de votre compte Azure Cosmos DB.

1. Dans le fichier **environment.js** , remplacez la valeur de `port` par 10255. 

    ```javascript
    const port = 10255;
    ```

1. Dans le fichier **environment.js** , remplacez la valeur de `accountName` par le nom de compte Azure Cosmos DB que vous avez créé à [l’étape 4](tutorial-develop-mongodb-nodejs-part4.md) du didacticiel. 

1. Récupérez la clé primaire du compte Azure Cosmos DB en entrant la commande CLI suivante dans la fenêtre du terminal : 

    ```azurecli-interactive
    az cosmosdb list-keys --name <cosmosdb-name> -g myResourceGroup
    ```    
    
    \<cosmosdb-name> est le nom du compte Azure Cosmos DB que vous avez créé dans la [partie 4](tutorial-develop-mongodb-nodejs-part4.md) du tutoriel.

1. Copiez la clé primaire dans le fichier **environment.js** en tant que valeur `key`.

Votre application dispose maintenant de toutes les informations nécessaires pour se connecter à Azure Cosmos DB. 

## <a name="create-a-hero-model"></a>Créer un modèle héros

Vous devez ensuite définir le schéma des données à stocker dans Azure Cosmos DB en définissant un fichier de modèle. Pour créer un _modèle héros_ qui définit le schéma des données, procédez comme suit :

1. Dans le volet Explorateur, sous le dossier **server** , créez un fichier nommé **hero.model.js**.

1. Copiez le code suivant dans le fichier **hero.model.js**. Ce code fournit les fonctionnalités suivantes :

   * Requiert Mongoose.
   * Crée un nouveau schéma avec un ID, un nom et un message.
   * Crée un modèle à l’aide du schéma.
   * Exporte le modèle. 
   * Nomme la collection **Heroes** (au lieu de **Heros** , le nom par défaut de la collection selon les règles d’affectation de noms au pluriel de Mongoose).

   ```javascript
   const mongoose = require('mongoose');

   const Schema = mongoose.Schema;

   const heroSchema = new Schema(
     {
       id: { type: Number, required: true, unique: true },
       name: String,
       saying: String
     },
     {
       collection: 'Heroes'
     }
   );

   const Hero = mongoose.model('Hero', heroSchema);

   module.exports = Hero;
   ```

## <a name="create-a-hero-service"></a>Créer un service héros

Après avoir créé le modèle héros, vous devez définir un service pour lire les données et effectuer des opérations de liste, création, suppression et mise à jour. Procédez comme suit pour créer un _service héros_ qui interroge les données à partir d’Azure Cosmos DB :

1. Dans le volet Explorateur, sous le dossier **server** , créez un fichier nommé **hero.model.js**.

1. Copiez le code suivant dans le fichier **hero.service.js**. Ce code fournit les fonctionnalités suivantes :

   * Obtient le modèle que vous avez créé.
   * Se connecte à la base de données.
   * Crée une variable `docquery` qui utilise la méthode `hero.find` pour définir une requête qui renvoie tous les héros.
   * Exécute une requête avec la fonction `docquery.exec` avec la promesse d’obtenir une liste de tous les héros, où l’état de la réponse est 200. 
   * Renvoie le message d’erreur si l’état est 500.
   * Obtient le héros car nous utilisons des modules. 

   ```javascript
   const Hero = require('./hero.model');

   require('./mongo').connect();

   function getHeroes() {
     const docquery = Hero.find({});
     docquery
       .exec()
       .then(heroes => {
         res.status(200).json(heroes);
       })
       .catch(error => {
         res.status(500).send(error);
         return;
       });
   }

   module.exports = {
     getHeroes
   };
   ```

## <a name="configure-routes"></a>Configurer des itinéraires

Ensuite, vous devez définir des itinéraires pour gérer les URL des requêtes get, create, read et delete. Les méthodes de routage spécifient les fonctions de rappel (également appelées _fonctions gestionnaires_ ). Ces fonctions sont appelées lorsque l’application reçoit une requête au point de terminaison et via la méthode HTTP spécifiés. Procédez comme suit pour ajouter le service de héros et définir vos itinéraires :

1. Dans Visual Studio Code, dans le fichier **routes.js** , commentez la fonction `res.send` qui envoie des exemples de données héros. Ajoutez une ligne pour appeler la fonction `heroService.getHeroes` à la place.

    ```javascript
    router.get('/heroes', (req, res) => {
      heroService.getHeroes(req, res);
    //  res.send(200, [
    //      {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    //  ])
    });
    ```

1. Dans le fichier **routes.js** le service héros `require` est requis :

    ```javascript
    const heroService = require('./hero.service'); 
    ```

1. Dans le fichier **hero.service.js** , mettez à jour la fonction `getHeroes` pour obtenir les paramètres `req` et `res` comme suit :

    ```javascript
    function getHeroes(req, res) {
    ```

Prenons quelques instants pour examiner et parcourir le code précédent. Le fichier index.js apparaît en premier. Il configure le serveur de nœud. Notez qu’il configure et définit les itinéraires. Ensuite, le fichier routes.js communique avec le service héros pour lui demander d’obtenir vos fonctions, comme **getHeroes** , et de transmettre les requêtes et les réponses. Le fichier hero.service.js obtient le modèle et établit la connexion à Mongo. Ensuite, il exécute **getHeroes** lorsque nous l’appelons et retourne la réponse 200. 

## <a name="run-the-app"></a>Exécuter l’application

Ensuite, exécutez l’application en procédant comme suit :

1. Dans Visual Studio Code, enregistrez toutes les modifications. Sur la gauche, sélectionnez le bouton **Déboguer** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part5/debug-button.png":::, puis sélectionnez le bouton **Démarrer le débogage** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part5/start-debugging-button.png":::.

1. Ensuite, basculez vers le navigateur. Ouvrez les **outils de développement** et **l’onglet Réseau**. Accédez à `http://localhost:3000`, où vous pouvez voir l’application.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part5/azure-cosmos-db-heroes-app.png" alt-text="Nouveau compte Azure Cosmos DB dans le portail Azure":::

Pour le moment, aucun héros n’est stocké dans l’application. Dans la partie suivante de ce didacticiel, nous allons ajouter des fonctionnalités d’ajout, d’envoi et de suppression. Nous pouvons ensuite ajouter, mettre à jour et supprimer des héros de l’interface utilisateur en utilisant des connexions Mongoose à notre base de données Azure Cosmos. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n’avez plus besoin des ressources, vous pouvez supprimer le groupe de ressources, le compte Azure Cosmos DB et toutes les ressources associées. Procédez comme suit pour supprimer le groupe de ressources :

 1. Accédez au groupe de ressources dans lequel vous avez créé le compte Azure Cosmos DB.
 1. Sélectionnez **Supprimer le groupe de ressources**.
 1. Confirmez le nom du groupe de ressources à supprimer et sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Passez à la partie 6 du didacticiel pour ajouter les fonctions Publier, Placer et Supprimer à l’application :

> [!div class="nextstepaction"]
> [Partie 6 : Ajouter les fonctions Publier, Placer et Supprimer à l’application](tutorial-develop-mongodb-nodejs-part6.md)