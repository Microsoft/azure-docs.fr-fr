---
title: Créer une application Node.js Express avec l’API d’Azure Cosmos DB pour MongoDB (Partie 2)
description: Il s’agit de la partie 2 de cette série de didacticiels sur la création d’une application MongoDB avec Angular et Node sur Azure Cosmos DB à l’aide des mêmes API que celles utilisées pour MongoDB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18
ms.reviewer: sngun
ms.openlocfilehash: e3af713d6e120e66a0cde5116f4556bb8caedf55
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097845"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---create-a-nodejs-express-app"></a>Créer une application Angular avec l’API d’Azure Cosmos DB pour MongoDB - Créer une application Express Node.js
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Ce tutoriel en plusieurs parties montre comment créer une application écrite en Node.js avec Express et Angular, puis comment la connecter à votre [compte Cosmos configuré avec l’API de Cosmos DB pour MongoDB](mongodb-introduction.md).

La partie 2 de ce didacticiel est basée sur l’[introduction](tutorial-develop-mongodb-nodejs.md) et aborde les tâches suivantes :

> [!div class="checklist"]
> * Installation de l’interface CLI Angular et TypeScript
> * Création d’un projet à l’aide d’Angular
> * Génération de l’application à l’aide de l’infrastructure Express
> * Test de l’application dans Postman

## <a name="video-walkthrough"></a>Vidéo de procédure pas à pas

> [!VIDEO https://www.youtube.com/embed/lIwJIYcGSUg]

## <a name="prerequisites"></a>Prérequis

Avant de commencer cette partie du didacticiel, assurez-vous d’avoir bien visionné la [vidéo d’introduction](tutorial-develop-mongodb-nodejs.md).

Ce didacticiel requiert les éléments suivants : 
* [Node.js](https://nodejs.org/) version 8.4.0 ou supérieure.
* [Postman](https://www.getpostman.com/)
* [Visual Studio Code](https://code.visualstudio.com/) ou l’éditeur de code de votre choix.

> [!TIP]
> Ce didacticiel vous guide à travers les étapes pour générer l’application pas à pas. Si vous souhaitez télécharger le projet terminé, vous pouvez obtenir l’application complète à partir du [référentiel cosmosdb-angular](https://github.com/Azure-Samples/angular-cosmosdb) sur GitHub.

## <a name="install-the-angular-cli-and-typescript"></a>Installation de l’interface CLI Angular et TypeScript

1. Ouvrez une fenêtre d’invite de commandes pour Windows ou une fenêtre Terminal pour Mac et installez l’interface CLI Angular.

    ```bash
    npm install -g @angular/cli
    ```

2. Installez TypeScript en entrant la commande suivante dans l’invite de commandes. 

    ```bash
    npm install -g typescript
    ```

## <a name="use-the-angular-cli-to-create-a-new-project"></a>Utiliser l’interface CLI Angular pour créer un nouveau projet

1. Depuis l’invite de commandes, accédez au dossier où vous souhaitez créer votre projet, puis exécutez la commande suivante. Cette commande crée un dossier et un projet nommé angular-cosmosdb, puis installe les composants Angular requis pour une nouvelle application. Elle utilise le programme d’installation minimale (--minimal) et spécifie que le projet utilise Sass (une syntaxe de type CSS avec l’indicateur --style scss).

    ```bash
    ng new angular-cosmosdb --minimal --style scss
    ```

2. Une fois la commande terminée, remplacez les répertoires dans le dossier src/client.

    ```bash
    cd angular-cosmosdb
    ```

3. Ouvrez ensuite le dossier dans Visual Studio Code.

    ```bash
    code .
    ```

## <a name="build-the-app-using-the-express-framework"></a>Générer l’application à l’aide de l’infrastructure Express

1. Dans Visual Studio Code, dans le volet **Explorateur** , cliquez avec le bouton droit sur le dossier **src** , cliquez sur **Nouveau dossier** et nommez-le nouveau dossier *serveur*.

2. Dans le volet **Explorateur** , cliquez avec le bouton droit sur le dossier **serveur** , cliquez sur **Nouveau fichier** et nommez-le *index.js*.

3. Revenez à l’invite de commandes et utilisez la commande suivante pour installer l’analyseur de corps. Cela aide notre application à analyser les données JSON qui sont transmises aux API.

    ```bash
    npm i express body-parser --save
    ```

4. Dans Visual Studio Code, copiez le code suivant dans le fichier index.js. Ce code :
    * Référence Express
    * Tire l’analyseur de corps pour lire les données JSON dans le corps des requêtes
    * Utilise une fonctionnalité intégrée appelée le tracé
    * Définit des variables racines pour trouver plus facilement l’emplacement du code
    * Configure un port
    * Améliore Express
    * Indique à l’application comment utiliser l’intergiciel permettant de distribuer le serveur
    * Distribue tout ce qui se trouve dans le dossier dist pour constituer le contenu statique
    * Distribue l’application et index.html pour toutes les requêtes GET introuvables sur le serveur (pour obtenir des liens profonds)
    * Démarre le serveur avec app.listen
    * Utilise une fonction arrow pour se connecter au port actif
    
   ```node
   const express = require('express');
   const bodyParser = require('body-parser');
   const path = require('path');
   const routes = require('./routes');

   const root = './';
   const port = process.env.PORT || '3000';
   const app = express();

   app.use(bodyParser.json());
   app.use(bodyParser.urlencoded({ extended: false }));
   app.use(express.static(path.join(root, 'dist/angular-cosmosdb')));
   app.use('/api', routes);
   app.get('*', (req, res) => {
     res.sendFile('dist/angular-cosmosdb/index.html', {root});
   });

   app.listen(port, () => console.log(`API running on localhost:${port}`));
   ```

5. Dans Visual Studio Code, dans le volet **Explorateur** , cliquez avec le bouton droit sur le dossier **serveur** , puis cliquez sur **Nouveau dossier**. Nommez le nouveau fichier *routes.js*. 

6. Copiez le code suivant dans **routes.js**. Ce code :
   * Référence le routeur Express
   * Obtient les héros
   * Renvoie l’objet JSON pour un héros défini

   ```node
   const express = require('express');
   const router = express.Router();

   router.get('/heroes', (req, res) => {
    res.send(200, [
       {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    ])
   });

   module.exports=router;
   ```

7. Enregistre tous vos fichiers modifiés. 

8. Dans Visual Studio Code, cliquez sur le bouton **Déboguer** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/debug-button.png":::, puis cliquez sur le bouton représentant un engrenage :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/gear-button.png":::. Le fichier launch.json s’ouvre dans Visual Studio Code. 

8. À la ligne 11 du fichier launch.json, remplacez `"${workspaceFolder}\\server"` par `"program": "${workspaceRoot}/src/server/index.js"` et enregistrez le fichier.

9. Cliquez sur le bouton **Démarrer le débogage** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/start-debugging-button.png"::: pour exécuter l’application.

    L’application doit s’exécuter sans erreurs.

## <a name="use-postman-to-test-the-app"></a>Utiliser Postman pour tester l’application

1. Ouvrez Postman et placez `http://localhost:3000/api/heroes` dans la zone GET. 

2. Cliquez sur le bouton **Envoyer** et récupérez la réponse json à partir de l’application. 

    Cette réponse indique que l’application est active et s’exécute en local. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part2/azure-cosmos-db-postman.png" alt-text="Postman montrant la demande et la réponse":::


## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du didacticiel, vous avez :

> [!div class="checklist"]
> * Créé un projet Node.js à l’aide de l’interface CLI Angular
> * Testé l’application à l’aide de Postman

Vous pouvez maintenant passer à la partie suivante du didacticiel afin de générer une interface utilisateur.

> [!div class="nextstepaction"]
> [Générer l’interface utilisateur avec Angular](tutorial-develop-mongodb-nodejs-part3.md)
