---
title: 'Tutoriel : Créer une application web Node.js avec le SDK JavaScript Azure Cosmos DB pour gérer les données de l’API SQL'
description: Ce didacticiel Node.js explique comment utiliser Microsoft Azure Cosmos DB pour stocker des données et y accéder à partir d’une application web Express Node.js hébergée sur la fonctionnalité Web Apps de Microsoft Azure App Service.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: sngun
ms.custom: devx-track-js
ms.openlocfilehash: 49cf54bda985f7d97b2db6a3ada7859aee829cff
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359538"
---
# <a name="tutorial-build-a-nodejs-web-app-using-the-javascript-sdk-to-manage-a-sql-api-account-in-azure-cosmos-db"></a>Tutoriel : Créer une application web Node.js avec le Kit de développement logiciel (SDK) JavaScript pour gérer un compte d’API SQL dans Azure Cosmos DB 
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.JS](sql-api-nodejs-application.md)
> * [Python](./create-sql-api-python.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

En tant que développeur, vous pouvez avoir des applications qui utilisent des données de documents NoSQL. Vous pouvez utiliser un compte d’API SQL dans Azure Cosmos DB pour stocker et accéder à ces données de documents. Ce didacticiel Node.js vous explique comment stocker des données et y accéder à partir d’un compte d’API SQL dans Azure Cosmos DB à l’aide d’une application Express Node.js hébergée sur la fonctionnalité Web Apps de Microsoft Azure App Service. Dans ce didacticiel, vous allez créer une application basée sur le web (application de tâches), qui vous permet de créer, récupérer et terminer des tâches. Ces dernières sont stockées en tant que documents JSON dans AzureCosmos DB. 

Ce didacticiel montre comment créer un compte d’API SQL dans Azure Cosmos DB à l’aide du portail Azure. Vous générerez et exécuterez ensuite une application web basée sur le Kit de développement logiciel (SDK) Node.js pour créer une base de données et un conteneur, et pour ajouter des éléments au conteneur. Ce tutoriel utilise le SDK JavaScript version 3.0.

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Création d’un compte Azure Cosmos DB
> * Création d’une application Node.js
> * Connecter une application à Azure Cosmos DB
> * Exécuter et déployer l’application dans Azure

## <a name="prerequisites"></a><a name="_Toc395783176"></a>Configuration requise

Avant de suivre les instructions de cet article, veillez à disposer des ressources suivantes :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js][Node.js] version 6.10 ou ultérieure.
* [Générateur Express](https://www.expressjs.com/starter/generator.html) (installation possible via `npm install express-generator -g`)
* Installez [Git][Git] sur votre station de travail locale.

## <a name="create-an-azure-cosmos-db-account"></a><a name="_Toc395637761"></a>Créer un compte Azure Cosmos DB
Commençons par créer un compte Azure Cosmos DB. Si vous possédez déjà un compte ou si vous utilisez l’émulateur Azure Cosmos DB pour ce didacticiel, vous pouvez passer à l’[étape 2 : Créer une application Node.js](#_Toc395783178).

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [cosmos-db-keys](../../includes/cosmos-db-keys.md)]

## <a name="create-a-new-nodejs-application"></a><a name="_Toc395783178"></a>Créer une application Node.js
Voyons maintenant comment créer un projet Node.js « Hello World » de base à l’aide de l’infrastructure Express.

1. Ouvrez votre terminal préféré, par exemple l’invite de commande Node.js.

1. Accédez au répertoire dans lequel vous souhaitez stocker la nouvelle application.

1. Utilisez le générateur Express pour générer une nouvelle application appelée **todo**.

   ```bash
   express todo
   ```

1. Ouvrez le répertoire **todo** et installez les dépendances.

   ```bash
   cd todo
   npm install
   ```

1. Exécutez la nouvelle application.

   ```bash
   npm start
   ```

1. Vous pouvez afficher votre nouvelle application en accédant à l’adresse `http://localhost:3000` dans votre navigateur.
   
   :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-express.png" alt-text="Découvrir Node.js - Capture d’écran de l’application Hello World dans une fenêtre de navigateur":::

   Arrêtez l’application en appuyant sur CTRL + C dans la fenêtre de terminal, puis sélectionnez **y** pour arrêter le programme de traitement par lots.

## <a name="install-the-required-modules"></a><a name="_Toc395783179"></a>Installer les modules requis

Le fichier **package.json** est l'un des fichiers créés à la racine du projet. Il contient une liste de modules supplémentaires qui sont nécessaires pour les applications Node.js. Lorsque vous déployez cette application sur Azure, ce fichier permet de déterminer quels modules doivent être installés sur Azure pour la prise en charge de votre application. Installez deux autres packages pour ce tutoriel.

1. Installez le module **\@azure/cosmos** via npm. 

   ```bash
   npm install @azure/cosmos
   ```

## <a name="connect-the-nodejs-application-to-azure-cosmos-db"></a><a name="_Toc395783180"></a>Connecter l’application Node.js à Azure Cosmos DB
Maintenant que vous avez terminé l’installation et la configuration initiales, ensuite vous écrirez du code qui est requis par l’application de tâches pour communiquer avec Azure Cosmos DB.

### <a name="create-the-model"></a>Créer le modèle
1. À la racine du répertoire de votre projet, créez un répertoire nommé **models**.  

2. Dans le répertoire **models**, créez un fichier nommé **taskDao.js**. Ce fichier contient le code nécessaire pour créer la base de données et le conteneur. Il définit également des méthodes pour lire, mettre à jour, créer et rechercher des tâches dans Azure Cosmos DB. 

3. Copiez le code suivant dans le fichier **taskDao.js** :

   ```javascript
    // @ts-check
    const CosmosClient = require('@azure/cosmos').CosmosClient
    const debug = require('debug')('todo:taskDao')

    // For simplicity we'll set a constant partition key
    const partitionKey = undefined
    class TaskDao {
      /**
       * Manages reading, adding, and updating Tasks in Cosmos DB
       * @param {CosmosClient} cosmosClient
       * @param {string} databaseId
       * @param {string} containerId
       */
      constructor(cosmosClient, databaseId, containerId) {
        this.client = cosmosClient
        this.databaseId = databaseId
        this.collectionId = containerId

        this.database = null
        this.container = null
      }

      async init() {
        debug('Setting up the database...')
        const dbResponse = await this.client.databases.createIfNotExists({
          id: this.databaseId
        })
        this.database = dbResponse.database
        debug('Setting up the database...done!')
        debug('Setting up the container...')
        const coResponse = await this.database.containers.createIfNotExists({
          id: this.collectionId
        })
        this.container = coResponse.container
        debug('Setting up the container...done!')
      }

      async find(querySpec) {
        debug('Querying for items from the database')
        if (!this.container) {
          throw new Error('Collection is not initialized.')
        }
        const { resources } = await this.container.items.query(querySpec).fetchAll()
        return resources
      }

      async addItem(item) {
        debug('Adding an item to the database')
        item.date = Date.now()
        item.completed = false
        const { resource: doc } = await this.container.items.create(item)
        return doc
      }

      async updateItem(itemId) {
        debug('Update an item in the database')
        const doc = await this.getItem(itemId)
        doc.completed = true

        const { resource: replaced } = await this.container
          .item(itemId, partitionKey)
          .replace(doc)
        return replaced
      }

      async getItem(itemId) {
        debug('Getting an item from the database')
        const { resource } = await this.container.item(itemId, partitionKey).read()
        return resource
      }
    }

    module.exports = TaskDao
   ```
4. Enregistrez et fermez le fichier **taskDao.js** .  

### <a name="create-the-controller"></a>Création du contrôleur

1. Dans le répertoire **routes** de votre projet, créez un fichier nommé **tasklist.js**.  

2. Ajoutez le code suivant dans **tasklist.js**. Ce code charge CosmosClient et les modules asynchrones, qui sont utilisés par **tasklist.js**. Il définit également la classe **TaskList** à qui est transmise en tant qu’instance de l’objet **TaskDao** défini précédemment :
   
   ```javascript
    const TaskDao = require("../models/TaskDao");
    
    class TaskList {
      /**
       * Handles the various APIs for displaying and managing tasks
       * @param {TaskDao} taskDao
       */
      constructor(taskDao) {
        this.taskDao = taskDao;
      }
      async showTasks(req, res) {
        const querySpec = {
          query: "SELECT * FROM root r WHERE r.completed=@completed",
          parameters: [
            {
              name: "@completed",
              value: false
            }
          ]
        };

        const items = await this.taskDao.find(querySpec);
        res.render("index", {
          title: "My ToDo List ",
          tasks: items
        });
      }

      async addTask(req, res) {
        const item = req.body;

        await this.taskDao.addItem(item);
        res.redirect("/");
      }

      async completeTask(req, res) {
        const completedTasks = Object.keys(req.body);
        const tasks = [];

        completedTasks.forEach(task => {
          tasks.push(this.taskDao.updateItem(task));
        });

        await Promise.all(tasks);

        res.redirect("/");
      }
    }

    module.exports = TaskList;
   ```

3. Enregistrez et fermez le fichier **tasklist.js** .

### <a name="add-configjs"></a>Ajout de config.js

1. À la racine du répertoire de projet, créez un fichier nommé **config.js**. 

2. Ajoutez le code suivant au fichier **config.js**. Ce code définit les paramètres de configuration et les valeurs nécessaires à notre application.
   
   ```javascript
   const config = {};

   config.host = process.env.HOST || "[the endpoint URI of your Azure Cosmos DB account]";
   config.authKey =
     process.env.AUTH_KEY || "[the PRIMARY KEY value of your Azure Cosmos DB account";
   config.databaseId = "ToDoList";
   config.containerId = "Items";

   if (config.host.includes("https://localhost:")) {
     console.log("Local environment detected");
     console.log("WARNING: Disabled checking of self-signed certs. Do not have this code in production.");
     process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
     console.log(`Go to http://localhost:${process.env.PORT || '3000'} to try the sample.`);
   }

   module.exports = config;
   ```

3. Dans le fichier **config.js**, mettez à jour les valeurs des paramètres HOST et AUTH_KEY en indiquant les valeurs trouvées dans la page Clés de votre compte Azure Cosmos DB, dans le [portail Azure](https://portal.azure.com). 

4. Enregistrez et fermez le fichier **config.js** .

### <a name="modify-appjs"></a>Modification de app.js

1. Dans le répertoire du projet, ouvrez le fichier **app.js** . Ce fichier a été créé précédemment lors de la création de l'application web Express.  

2. Ajoutez le code suivant au fichier **app.js**. Ce code définit le fichier de configuration à utiliser et charge les valeurs dans des variables que vous utiliserez dans les sections suivantes. 
   
   ```javascript
    const CosmosClient = require('@azure/cosmos').CosmosClient
    const config = require('./config')
    const TaskList = require('./routes/tasklist')
    const TaskDao = require('./models/taskDao')

    const express = require('express')
    const path = require('path')
    const logger = require('morgan')
    const cookieParser = require('cookie-parser')
    const bodyParser = require('body-parser')

    const app = express()

    // view engine setup
    app.set('views', path.join(__dirname, 'views'))
    app.set('view engine', 'jade')

    // uncomment after placing your favicon in /public
    //app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
    app.use(logger('dev'))
    app.use(bodyParser.json())
    app.use(bodyParser.urlencoded({ extended: false }))
    app.use(cookieParser())
    app.use(express.static(path.join(__dirname, 'public')))

    //Todo App:
    const cosmosClient = new CosmosClient({
      endpoint: config.host,
      key: config.authKey
    })
    const taskDao = new TaskDao(cosmosClient, config.databaseId, config.containerId)
    const taskList = new TaskList(taskDao)
    taskDao
      .init(err => {
        console.error(err)
      })
      .catch(err => {
        console.error(err)
        console.error(
          'Shutting down because there was an error settinig up the database.'
        )
        process.exit(1)
      })

    app.get('/', (req, res, next) => taskList.showTasks(req, res).catch(next))
    app.post('/addtask', (req, res, next) => taskList.addTask(req, res).catch(next))
    app.post('/completetask', (req, res, next) =>
      taskList.completeTask(req, res).catch(next)
    )
    app.set('view engine', 'jade')

    // catch 404 and forward to error handler
    app.use(function(req, res, next) {
      const err = new Error('Not Found')
      err.status = 404
      next(err)
    })

    // error handler
    app.use(function(err, req, res, next) {
      // set locals, only providing error in development
      res.locals.message = err.message
      res.locals.error = req.app.get('env') === 'development' ? err : {}

      // render the error page
      res.status(err.status || 500)
      res.render('error')
    })

    module.exports = app
   ```

3. Enfin, enregistrez et fermez le fichier **app.js**.

## <a name="build-a-user-interface"></a><a name="_Toc395783181"></a>Créer une interface utilisateur

Nous allons maintenant générer l’interface utilisateur afin qu’un utilisateur puisse interagir avec l’application. L'application Express que nous avons créée dans les sections précédentes utilise **Jade** comme moteur de vue.

1. Le fichier **layout.jade** du répertoire **views** sert de modèle global aux autres fichiers **.jade**. Dans cette étape, vous allez le modifier pour utiliser Twitter Bootstrap, qui est un kit de ressources utilisé pour concevoir des sites web.  

2. Ouvrez le fichier **layout.jade** trouvé dans le dossier **views** et remplacez le contenu par le code suivant :

   ```html
   doctype html
   html
     head
       title= title
       link(rel='stylesheet', href='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/css/bootstrap.min.css')
       link(rel='stylesheet', href='/stylesheets/style.css')
     body
       nav.navbar.navbar-inverse.navbar-fixed-top
         div.navbar-header
           a.navbar-brand(href='#') My Tasks
       block content
       script(src='//ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.2.min.js')
       script(src='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/bootstrap.min.js')
   ```

    Ce code demande au moteur **Jade** de générer un rendu HTML pour notre application et crée un **bloc** intitulé **content** dans lequel nous pouvons fournir la mise en page de nos pages de contenu. Enregistrez et fermez le fichier **layout.jade**.

3. Ouvrez maintenant le fichier **index.jade**, la vue qui sera utilisée par l'application, et remplacez le contenu du fichier par le code suivant :

   ```html
   extends layout
   block content
        h1 #{title}
        br
        
        form(action="/completetask", method="post")
         table.table.table-striped.table-bordered
            tr
              td Name
              td Category
              td Date
              td Complete
            if (typeof tasks === "undefined")
              tr
                td
            else
              each task in tasks
                tr
                  td #{task.name}
                  td #{task.category}
                  - var date  = new Date(task.date);
                  - var day   = date.getDate();
                  - var month = date.getMonth() + 1;
                  - var year  = date.getFullYear();
                  td #{month + "/" + day + "/" + year}
                  td
                   if(task.completed) 
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
                   else
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
          button.btn.btn-primary(type="submit") Update tasks
        hr
        form.well(action="/addtask", method="post")
          label Item Name:
          input(name="name", type="textbox")
          label Item Category:
          input(name="category", type="textbox")
          br
          button.btn(type="submit") Add item
   ```

Ce code étend la mise en page et fournit du contenu pour l’espace réservé **content** que nous avons vu plus haut dans le fichier **layout.jade**. Dans cette mise en page, nous avons créé deux formulaires HTML.

Le premier formulaire contient un tableau pour vos données et un bouton qui permet de mettre à jour des éléments en appelant la méthode **/completeTask** du contrôleur.
    
Le deuxième formulaire contient deux champs d'entrée et un bouton qui permet de créer un élément en appelant la méthode **/addtask** du contrôleur. C’est tout ce dont nous avons besoin pour faire fonctionner l’application.

## <a name="run-your-application-locally"></a><a name="_Toc395783181"></a>Exécuter votre application localement

Maintenant que vous avez créé l’application, vous pouvez l’exécuter localement en procédant comme suit :  

1. Pour tester l’application sur votre machine locale, exécutez la commande `npm start` dans le terminal pour démarrer votre application, puis actualisez la page `http://localhost:3000` dans votre navigateur. La page devrait maintenant ressembler à ce qui est présenté dans la capture d’écran suivante :
   
    :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-localhost.png" alt-text="Capture d’écran de l’application My Todo List dans une fenêtre de navigateur":::

    > [!TIP]
    > Si vous recevez une erreur liée à la mise en retrait dans le fichier layout.jade ou index.jade, assurez-vous que les deux premières lignes des deux fichiers sont justifiées à gauche et ne présentent aucun espace. Supprimez tout espace éventuel dans les deux premières lignes, enregistrez les deux fichiers, puis actualisez la fenêtre du navigateur. 

2. Utilisez les champs Élément, Nom d’élément et Catégorie pour saisir une nouvelle tâche, puis sélectionnez **Ajouter des éléments**. Cela permet de créer un document présentant ces propriétés dans Azure Cosmos DB. 

3. La page doit se mettre à jour et afficher le nouvel élément créé dans la liste des tâches.
   
    :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-added-task.png" alt-text="Capture d’écran de l’application avec un nouvel élément dans la liste des tâches":::

4. Pour terminer une tâche, cochez la case dans la colonne Complete, puis sélectionnez **Mettre à jour les tâches**. Cela met à jour le document que vous avez déjà créé et le supprime de la vue.

5. Pour arrêter l’application, appuyez sur CTRL + C dans la fenêtre de terminal, puis cliquez sur **Y** pour arrêter le traitement par lots.

## <a name="deploy-your-application-to-web-apps"></a><a name="_Toc395783182"></a>Déployer votre application sur Web Apps

Lorsque votre application a bien été exécutée localement, vous pouvez la déployer sur Azure en procédant comme suit :

1. Si vous ne l’avez pas encore fait, activez un référentiel git pour votre application Web Apps.

2. Ajoutez votre application Web Apps en tant que git distant.
   
   ```bash
   git remote add azure https://username@your-azure-website.scm.azurewebsites.net:443/your-azure-website.git
   ```

3. Déployez l’application en l’envoyant à l’appareil distant.
   
   ```bash
   git push azure main
   ```

4. Dans quelques secondes, votre application web sera publiée et lancée dans un navigateur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n’avez plus besoin de ces ressources, vous pouvez supprimer le groupe de ressources, le compte Azure Cosmos DB et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources que vous avez utilisé pour le compte Azure Cosmos DB, sélectionnez **Supprimer**, puis confirmez le nom du groupe de ressources à supprimer.

## <a name="next-steps"></a><a name="_Toc395637775"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer des applications mobiles avec Xamarin et Azure Cosmos DB](mobile-apps-with-xamarin.md)


[Node.js]: https://nodejs.org/
[Git]: https://git-scm.com/
[GitHub]: https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-todo-app