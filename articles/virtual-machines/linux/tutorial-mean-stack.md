---
title: 'Didacticiel : déployer une pile MEAN sur une machine virtuelle Linux dans Azure | Microsoft Docs'
description: Avec ce didacticiel, vous allez apprendre à créer une pile MongoDB, Express, AngularJS et Node.js (MEAN) sur une machine virtuelle Linux dans Azure.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/08/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 8b472c5f1175f006e4c7a6574135460540bf2283
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/11/2018
ms.locfileid: "38720650"
---
# <a name="tutorial-create-a-mongodb-express-angularjs-and-nodejs-mean-stack-on-a-linux-virtual-machine-in-azure"></a>Didacticiel : créer une pile MongoDB, Express, AngularJS et Node.js (MEAN) sur une machine virtuelle Linux dans Azure

Ce didacticiel montre comment implémenter une pile MongoDB, Express, AngularJS et Node.js (MEAN) sur une machine virtuelle Linux dans Azure. La pile MEAN que vous créez permet l’ajout, la suppression et le référencement de livres dans une base de données. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle Linux
> * Installer Node.js
> * Installer MongoDB et configurer le serveur
> * Installer Express et définir les itinéraires au serveur
> * Accéder aux itinéraires avec AngularJS
> * Exécution de l’application

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez procéder à une installation ou une mise à niveau, consultez [Installation d’Azure CLI 2.0]( /cli/azure/install-azure-cli).


## <a name="create-a-linux-vm"></a>Créer une machine virtuelle Linux

Créez un groupe de ressources avec la commande [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create), puis créez une machine virtuelle Linux avec la commande [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant utilise Azure CLI pour créer un groupe de ressources nommé *myResourceGroupMEAN* à l’emplacement *eastus*. Une machine virtuelle nommée *myVM* est créée à l’aide de clés SSH si elles n’existent pas déjà à un emplacement de clé par défaut. Pour utiliser un ensemble spécifique de clés, utilisez l’option --ssh-key-value.

```azurecli-interactive
az group create --name myResourceGroupMEAN --location eastus
az vm create \
    --resource-group myResourceGroupMEAN \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password 'Azure12345678!' \
    --generate-ssh-keys
az vm open-port --port 3300 --resource-group myResourceGroupMEAN --name myVM
```

Lorsque la machine virtuelle est créée, l’interface de ligne de commande Azure affiche des informations similaires à l’exemple suivant : 

```azurecli-interactive
{
  "fqdns": "",
  "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroupMEAN/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.72.77.9",
  "resourceGroup": "myResourceGroupMEAN"
}
```
Notez la valeur de `publicIpAddress`. Cette adresse est utilisée pour accéder à la machine virtuelle.

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle. Veillez à utiliser l’adresse IP publique correcte. Dans l’exemple ci-dessus, notre adresse IP était 13.72.77.9.

```bash
ssh azureuser@13.72.77.9
```

## <a name="install-nodejs"></a>Installer Node.js

[Node.js](https://nodejs.org/fr/) est un runtime JavaScript reposant sur le moteur JavaScript V8 de Chrome. Node.js est utilisé dans ce didacticiel pour configurer les itinéraires Express et les contrôleurs AngularJS.

Sur la machine virtuelle, installez Node.js à l’aide de l’interpréteur de commandes Bash que vous avez ouvert avec SSH.

```bash
sudo apt-get install -y nodejs
```

## <a name="install-mongodb-and-set-up-the-server"></a>Installer MongoDB et configurer le serveur
[MongoDB](http://www.mongodb.com) stocke les données dans des documents flexibles, similaires à JSON. Les champs d’une base de données peuvent varier d’un document à l’autre, et la structure des données peut changer au fil du temps. Pour notre exemple d’application, nous ajoutons à MongoDB des enregistrements de livre qui contiennent le nom du livre, le numéro isbn, l’auteur et le nombre de pages. 

1. Sur la machine virtuelle, définissez la clé MongoDB à l’aide de l’interpréteur de commandes Bash que vous avez ouvert avec SSH.

    ```bash
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
    echo "deb [ arch=amd64 ] http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
    ```

2. Mettez à jour le Gestionnaire de package avec la clé.
  
    ```bash
    sudo apt-get update
    ```

3. Installez MongoDB.

    ```bash
    sudo apt-get install -y mongodb
    ```

4. Démarrez le serveur.

    ```bash
    sudo service mongodb start
    ```

5. Nous devons également installer le package [body-parser](https://www.npmjs.com/package/body-parser-json) afin de pouvoir traiter les requêtes passées JSON sur le serveur.

    Installez le Gestionnaire de package npm.

    ```bash
    sudo apt-get install npm
    ```

    Installez le package body-parser.
    
    ```bash
    sudo npm install body-parser
    ```

6. Créez un dossier du nom de *Books* et ajoutez-y un fichier nommé *server.js* qui contient la configuration du serveur web.

    ```node.js
    var express = require('express');
    var bodyParser = require('body-parser');
    var app = express();
    app.use(express.static(__dirname + '/public'));
    app.use(bodyParser.json());
    require('./apps/routes')(app);
    app.set('port', 3300);
    app.listen(app.get('port'), function() {
        console.log('Server up: http://localhost:' + app.get('port'));
    });
    ```

## <a name="install-express-and-set-up-routes-to-the-server"></a>Installer Express et définir les itinéraires au serveur

[Express](https://expressjs.com) est un framework d’applications web Node.js flexible et minimal qui fournit des fonctionnalités aux applications web et mobiles. Express est utilisé dans ce didacticiel pour transmettre des informations de livre en provenance ou à destination de notre base de données MongoDB. [Mongoose](http://mongoosejs.com) offre une solution simple, reposant sur un schéma, pour modéliser les données de vos applications. Dans ce didacticiel, Mongoose sert à fournir un schéma de livre pour la base de données.

1. Installez Express et Mongoose.

    ```bash
    sudo npm install express mongoose
    ```

2. Dans le dossier *Books*, créez un dossier appelé *apps* et ajoutez un fichier nommé *routes.js* avec les itinéraires Express définis.

    ```node.js
    var Book = require('./models/book');
    module.exports = function(app) {
      app.get('/book', function(req, res) {
        Book.find({}, function(err, result) {
          if ( err ) throw err;
          res.json(result);
        });
      }); 
      app.post('/book', function(req, res) {
        var book = new Book( {
          name:req.body.name,
          isbn:req.body.isbn,
          author:req.body.author,
          pages:req.body.pages
        });
        book.save(function(err, result) {
          if ( err ) throw err;
          res.json( {
            message:"Successfully added book",
            book:result
          });
        });
      });
      app.delete("/book/:isbn", function(req, res) {
        Book.findOneAndRemove(req.query, function(err, result) {
          if ( err ) throw err;
          res.json( {
            message: "Successfully deleted the book",
            book: result
          });
        });
      });
      var path = require('path');
      app.get('*', function(req, res) {
        res.sendfile(path.join(__dirname + '/public', 'index.html'));
      });
    };
    ```

3. Dans le dossier *apps*, créez un dossier appelé *models* et ajoutez un fichier nommé *book.js* avec la configuration de modèle de livre définie.  

    ```node.js
    var mongoose = require('mongoose');
    var dbHost = 'mongodb://localhost:27017/test';
    mongoose.connect(dbHost);
    mongoose.connection;
    mongoose.set('debug', true);
    var bookSchema = mongoose.Schema( {
      name: String,
      isbn: {type: String, index: true},
      author: String,
      pages: Number
    });
    var Book = mongoose.model('Book', bookSchema);
    module.exports = mongoose.model('Book', bookSchema); 
    ```

## <a name="access-the-routes-with-angularjs"></a>Accéder aux itinéraires avec AngularJS

[AngularJS](https://angularjs.org) fournit un framework web permettant de créer des vues dynamiques dans vos applications web. Dans ce didacticiel, nous utilisons AngularJS pour connecter notre page web à Express et effectuer des actions sur notre base de données de livres.

1. Changez le répertoire de sauvegarde pour *Books* (`cd ../..`), puis créez un dossier du nom de *public* et ajoutez un fichier nommé *script.js* avec la configuration de contrôleur définie.

    ```node.js
    var app = angular.module('myApp', []);
    app.controller('myCtrl', function($scope, $http) {
      $http( {
        method: 'GET',
        url: '/book'
      }).then(function successCallback(response) {
        $scope.books = response.data;
      }, function errorCallback(response) {
        console.log('Error: ' + response);
      });
      $scope.del_book = function(book) {
        $http( {
          method: 'DELETE',
          url: '/book/:isbn',
          params: {'isbn': book.isbn}
        }).then(function successCallback(response) {
          console.log(response);
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
      };
      $scope.add_book = function() {
        var body = '{ "name": "' + $scope.Name + 
        '", "isbn": "' + $scope.Isbn +
        '", "author": "' + $scope.Author + 
        '", "pages": "' + $scope.Pages + '" }';
        $http({
          method: 'POST',
          url: '/book',
          data: body
        }).then(function successCallback(response) {
          console.log(response);
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
      };
    });
    ```
    
2. Dans le dossier *public*, créez un fichier du nom de *index.html* avec la page web définie.

    ```html
    <!doctype html>
    <html ng-app="myApp" ng-controller="myCtrl">
      <head>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
        <script src="script.js"></script>
      </head>
      <body>
        <div>
          <table>
            <tr>
              <td>Name:</td> 
              <td><input type="text" ng-model="Name"></td>
            </tr>
            <tr>
              <td>Isbn:</td>
              <td><input type="text" ng-model="Isbn"></td>
            </tr>
            <tr>
              <td>Author:</td> 
              <td><input type="text" ng-model="Author"></td>
            </tr>
            <tr>
              <td>Pages:</td>
              <td><input type="number" ng-model="Pages"></td>
            </tr>
          </table>
          <button ng-click="add_book()">Add</button>
        </div>
        <hr>
        <div>
          <table>
            <tr>
              <th>Name</th>
              <th>Isbn</th>
              <th>Author</th>
              <th>Pages</th>
            </tr>
            <tr ng-repeat="book in books">
              <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
              <td>{{book.name}}</td>
              <td>{{book.isbn}}</td>
              <td>{{book.author}}</td>
              <td>{{book.pages}}</td>
            </tr>
          </table>
        </div>
      </body>
    </html>
    ```

##  <a name="run-the-application"></a>Exécution de l’application

1. Changez le répertoire de sauvegarde pour *Books* (`cd ..`) et démarrez le serveur en exécutant la commande suivante :

    ```bash
    nodejs server.js
    ```

2. Ouvrez un navigateur web à l’adresse que vous avez enregistrée pour la machine virtuelle. Par exemple : *http://13.72.77.9:3300*. Un résultat similaire à cette page doit s’afficher :

    ![Enregistrement de livre](media/tutorial-mean/meanstack-init.png)

3. Renseignez les zones de texte et cliquez sur **Ajouter**. Par exemple : 

    ![Ajout d’un enregistrement de livre](media/tutorial-mean/meanstack-add.png)

4. Après avoir actualisé la page, vous devez afficher un résultat ressemblant à cette page :

    ![Liste des enregistrements de livre](media/tutorial-mean/meanstack-list.png)

5. Vous pouvez cliquer sur **Supprimer** et supprimer l’enregistrement de livre à partir de la base de données.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une application web qui effectue le suivi d’enregistrements de livre à l’aide d’une pile MEAN sur une machine virtuelle Linux. Vous avez appris à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle Linux
> * Installer Node.js
> * Installer MongoDB et configurer le serveur
> * Installer Express et définir les itinéraires au serveur
> * Accéder aux itinéraires avec AngularJS
> * Exécution de l’application

Passez au didacticiel suivant pour savoir comment mieux protéger les serveurs SSL à l’aide des certificats SSL.

> [!div class="nextstepaction"]
> [Sécuriser un serveur web avec SSL](tutorial-secure-web-server.md)
