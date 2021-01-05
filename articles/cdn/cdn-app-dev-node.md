---
title: Prise en main du kit de développement logiciel Azure CDN pour Node.js | Microsoft Docs
description: Découvrez comment créer une simple application console Node.js qui montre comment automatiser la création et la gestion de profils et de points de terminaisons Azure CDN.
services: cdn
documentationcenter: nodejs
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: c4bb6a61-de3d-4f0c-9dca-202554c43dfa
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.custom: devx-track-js
ms.openlocfilehash: f5d5c7a6e1f6993b19f38db2ae846b213a1d553e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993363"
---
# <a name="get-started-with-azure-cdn-development"></a>Prise en main du développement Azure CDN
> [!div class="op_single_selector"]
> * [Node.JS](cdn-app-dev-node.md)
> * [.NET](cdn-app-dev-net.md)
> 
> 

Vous pouvez utiliser le [kit de développement logiciel Azure CDN pour Node.js](https://www.npmjs.com/package/azure-arm-cdn) pour automatiser la création et la gestion des points de terminaison et profils CDN.  Ce didacticiel présente la création d’une application console Node.js simple, qui exécute plusieurs des opérations disponibles.  Il n’a pas vocation à décrire en détail tous les aspects du kit de développement logiciel Azure CDN pour Node.js.

Pour suivre ce tutoriel, vous devez déjà avoir installé et configuré [Node.js](https://www.nodejs.org)**4.x.x** ou ultérieur.  Vous pouvez utiliser n’importe quel éditeur de texte pour créer votre application Node.js.  Pour écrire ce didacticiel, j’ai utilisé [Visual Studio Code](https://code.visualstudio.com).  

> [!TIP]
> Le [projet achevé de ce didacticiel](https://code.msdn.microsoft.com/Azure-CDN-SDK-for-Nodejs-c712bc74) est disponible en téléchargement sur MSDN.
> 
> 

[!INCLUDE [cdn-app-dev-prep](../../includes/cdn-app-dev-prep.md)]

## <a name="create-your-project-and-add-npm-dependencies"></a>Créer votre projet et ajouter des dépendances NPM
Maintenant que nous avons créé un groupe de ressources pour nos profils CDN et autorisé l’application Azure AD à gérer les points de terminaison et profils CDN au sein de ce groupe, nous pouvons créer notre application.

Créez un dossier pour stocker votre application.  À partir d’une console contenant les outils Node.js dans votre chemin d’accès actuel, définissez votre emplacement actuel vers ce nouveau dossier et initialisez votre projet en exécutant la commande suivante :

```console
npm init
```

Vous obtenez alors une série de questions qui vous permettront d’initialiser votre projet.  Comme **point d’entrée**, ce didacticiel utilise *app.js*.  Vous pouvez voir mes autres choix dans l’exemple suivant.

![sortie init NPM](./media/cdn-app-dev-node/cdn-npm-init.png)

Notre projet est maintenant initialisé avec un fichier *packages.json* .  Notre projet va utiliser certaines bibliothèques Azure contenues dans des packages NPM.  Nous allons utiliser le Runtime client Azure pour Node.js (ms-rest-azure) et la bibliothèque cliente Azure CDN pour Node.js (azure-arm-cd).  Ajoutons-les au projet sous forme de dépendances.

```console
npm install --save ms-rest-azure
npm install --save azure-arm-cdn
```

Une fois les packages installés, le fichier *package.json* doit se présenter comme dans cet exemple (les numéros de version peuvent varier) :

``` json
{
  "name": "cdn_node",
  "version": "1.0.0",
  "description": "Azure CDN Node.js tutorial project",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Cam Soper",
  "license": "MIT",
  "dependencies": {
    "azure-arm-cdn": "^0.2.1",
    "ms-rest-azure": "^1.14.4"
  }
}
```

Enfin, à l’aide de votre éditeur de texte, créez un fichier texte vide et enregistrez-le à la racine de votre dossier de projet en tant que *app.js*.  Nous pouvons maintenant commencer à écrire du code.

## <a name="requires-constants-authentication-and-structure"></a>Paramètres « require », constantes, authentification et structure
Ouvrez *app.js* dans votre éditeur pour commencer à écrire la structure de base de notre programme.

1. Ajoutez les paramètres « require » pour nos packages NPM au début du code, comme suit :
   
    ``` javascript
    var msRestAzure = require('ms-rest-azure');
    var cdnManagementClient = require('azure-arm-cdn');
    ```
2. Nous devons définir certaines constantes que nos méthodes utiliseront.  Ajoutez le code suivant.  Veillez à remplacer les espaces réservés, notamment les **&lt;éléments entre chevrons&gt;** , par vos propres valeurs, si nécessaire.
   
    ``` javascript
    //Tenant app constants
    const clientId = "<YOUR CLIENT ID>";
    const clientSecret = "<YOUR CLIENT AUTHENTICATION KEY>"; //Only for service principals
    const tenantId = "<YOUR TENANT ID>";
   
    //Application constants
    const subscriptionId = "<YOUR SUBSCRIPTION ID>";
    const resourceGroupName = "CdnConsoleTutorial";
    const resourceLocation = "<YOUR PREFERRED AZURE LOCATION, SUCH AS Central US>";
    ```
3. Nous allons maintenant instancier le client de gestion CDN et lui donner nos informations d’identification.
   
    ``` javascript
    var credentials = new msRestAzure.ApplicationTokenCredentials(clientId, tenantId, clientSecret);
    var cdnClient = new cdnManagementClient(credentials, subscriptionId);
    ```
   
    Si vous utilisez une authentification d’utilisateurs individuels, ces deux lignes aura un aspect légèrement différent.
   
   > [!IMPORTANT]
   > N’utilisez ce code que si vous privilégiez l’authentification d’utilisateurs individuels au principal du service.  Veillez à protéger vos informations d’identification d’utilisateurs individuels et à ne pas les divulguer.
   > 
   > 
   
    ``` javascript
    var credentials = new msRestAzure.UserTokenCredentials(clientId, 
        tenantId, '<username>', '<password>', '<redirect URI>');
    var cdnClient = new cdnManagementClient(credentials, subscriptionId);
    ```
   
    Veillez à remplacer les éléments entre **&lt;chevrons&gt;** par les informations appropriées.  Pour `<redirect URI>`, utilisez l’URI de redirection que vous avez entré lorsque vous avez inscrit l’application dans Azure AD.
4. Notre application console Node.js va prendre quelques paramètres de ligne de commande.  Vérifions qu’au moins un des paramètres a été transmis.
   
   ```javascript
   //Collect command-line parameters
   var parms = process.argv.slice(2);
   
   //Do we have parameters?
   if(parms == null || parms.length == 0)
   {
       console.log("Not enough parameters!");
       console.log("Valid commands are list, delete, create, and purge.");
       process.exit(1);
   }
   ```
5. Cela nous amène à la partie principale de notre programme, où nous créons des branches vers d’autres fonctions dépendant des paramètres qui ont été transmis.
   
    ```javascript
    switch(parms[0].toLowerCase())
    {
        case "list":
            cdnList();
            break;
   
        case "create":
            cdnCreate();
            break;
   
        case "delete":
            cdnDelete();
            break;
   
        case "purge":
            cdnPurge();
            break;
   
        default:
            console.log("Valid commands are list, delete, create, and purge.");
            process.exit(1);
    }
    ```
6. À plusieurs endroits de notre programme, nous devons nous assurer que le nombre de paramètres approprié a été transmis et afficher une aide si ces paramètres nous semblent incorrects.  Nous allons créer des fonctions pour effectuer cette vérification.
   
   ```javascript
   function requireParms(parmCount) {
       if(parms.length < parmCount) {
           usageHelp(parms[0].toLowerCase());
           process.exit(1);
       }
   }
   
   function usageHelp(cmd) {
       console.log("Usage for " + cmd + ":");
       switch(cmd)
       {
           case "list":
               console.log("list profiles");
               console.log("list endpoints <profile name>");
               break;
   
           case "create":
               console.log("create profile <profile name>");
               console.log("create endpoint <profile name> <endpoint name> <origin hostname>");
               break;
   
           case "delete":
               console.log("delete profile <profile name>");
               console.log("delete endpoint <profile name> <endpoint name>");
               break;
   
           case "purge":
               console.log("purge <profile name> <endpoint name> <path>");
               break;
   
           default:
               console.log("Invalid command.");
       }
   }
   ```
7. Les fonctions que nous allons utiliser sur le client de gestion CDN sont asynchrones, ce qui signifie qu’elles ont besoin d’une méthode pour relancer un appel à la fin de leur exécution.  Nous allons en créer une qui permet d’afficher la sortie à partir du client de gestion CDN (le cas échéant) et de fermer le programme correctement.
   
    ```javascript
    function callback(err, result, request, response) {
        if (err) {
            console.log(err);
            process.exit(1);
        } else {
            console.log((result == null) ? "Done!" : result);
            process.exit(0);
        }
    }
    ```

Maintenant que nous avons écrit la structure de base de notre programme, nous devons créer les fonctions appelées en fonction de nos paramètres.

## <a name="list-cdn-profiles-and-endpoints"></a>Répertorier les profils CDN et points de terminaison
Commençons par répertorier les profils et les points de terminaison existants.  Les commentaires de mon code indiquent la syntaxe appropriée pour nous permettre de savoir quel paramètre va à quel emplacement.

```javascript
// list profiles
// list endpoints <profile name>
function cdnList(){
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        case "profiles":
            console.log("Listing profiles...");
            cdnClient.profiles.listByResourceGroup(resourceGroupName, callback);
            break;

        case "endpoints":
            requireParms(3);
            console.log("Listing endpoints...");
            cdnClient.endpoints.listByProfile(parms[2], resourceGroupName, callback);
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}
```

## <a name="create-cdn-profiles-and-endpoints"></a>Créer des profils CDN et des points de terminaison
Nous allons ensuite écrire des fonctions permettant de créer des profils et des points de terminaison.

```javascript
function cdnCreate() {
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        case "profile":
            cdnCreateProfile();
            break;

        case "endpoint":
            cdnCreateEndpoint();
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}

// create profile <profile name>
function cdnCreateProfile() {
    requireParms(3);
    console.log("Creating profile...");
    var standardCreateParameters = {
        location: resourceLocation,
        sku: {
            name: 'Standard_Verizon'
        }
    };

    cdnClient.profiles.create(parms[2], standardCreateParameters, resourceGroupName, callback);
}

// create endpoint <profile name> <endpoint name> <origin hostname>        
function cdnCreateEndpoint() {
    requireParms(5);
    console.log("Creating endpoint...");
    var endpointProperties = {
        location: resourceLocation,
        origins: [{
            name: parms[4],
            hostName: parms[4]
        }]
    };

    cdnClient.endpoints.create(parms[3], endpointProperties, parms[2], resourceGroupName, callback);
}
```

## <a name="purge-an-endpoint"></a>Vider un point de terminaison
En supposant que le point de terminaison a été créé, nous allons certainement chercher à vider le contenu du programme dans notre point de terminaison.

```javascript
// purge <profile name> <endpoint name> <path>
function cdnPurge() {
    requireParms(4);
    console.log("Purging endpoint...");
    var purgeContentPaths = [ parms[3] ];
    cdnClient.endpoints.purgeContent(parms[2], parms[1], resourceGroupName, purgeContentPaths, callback);
}
```

## <a name="delete-cdn-profiles-and-endpoints"></a>Supprimer des profils CDN et des points de terminaison
La dernière fonction que nous allons ajouter permet de supprimer les points de terminaison et les profils.

```javascript
function cdnDelete() {
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        // delete profile <profile name>
        case "profile":
            requireParms(3);
            console.log("Deleting profile...");
            cdnClient.profiles.deleteIfExists(parms[2], resourceGroupName, callback);
            break;

        // delete endpoint <profile name> <endpoint name>
        case "endpoint":
            requireParms(4);
            console.log("Deleting endpoint...");
            cdnClient.endpoints.deleteIfExists(parms[3], parms[2], resourceGroupName, callback);
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}
```

## <a name="running-the-program"></a>Exécution du programme
Nous pouvons à présent exécuter notre programme Node.js à l’aide de notre débogueur favori ou sur la console.

> [!TIP]
> Si vous utilisez Visual Studio Code comme débogueur, vous devez configurer votre environnement pour transmettre les paramètres de ligne de commande.  Visual Studio Code utilise le fichier **lanuch.json** à cette fin.  Recherchez une propriété nommée **args** et ajoutez un tableau de valeurs de chaîne pour vos paramètres, pour obtenir un résultat semblable à ce qui suit : `"args": ["list", "profiles"]`.
> 
> 

Commençons par répertorier nos profils.

![Liste des profils](./media/cdn-app-dev-node/cdn-list-profiles.png)

Nous obtenons un tableau vide.  Cela est tout à fait normal puisque nous n’avons aucun profil dans notre groupe de ressources.  Créons donc maintenant un profil.

![Création d’un profil](./media/cdn-app-dev-node/cdn-create-profile.png)

À présent, ajoutons un point de terminaison.

![Création d’un point de terminaison](./media/cdn-app-dev-node/cdn-create-endpoint.png)

Pour finir, nous allons supprimer notre profil.

![Suppression d’un profil](./media/cdn-app-dev-node/cdn-delete-profile.png)

## <a name="next-steps"></a>Étapes suivantes
Pour voir le projet achevé obtenu à partir de cette procédure pas à pas, [téléchargez l’exemple](https://code.msdn.microsoft.com/Azure-CDN-SDK-for-Nodejs-c712bc74).

Pour voir la référence du kit de développement logiciel Azure CDN pour Node.js, consultez cette [référence](https://azure.github.io/azure-sdk-for-node/azure-arm-cdn/latest/).

Pour rechercher une documentation supplémentaire sur le kit de développement logiciel Azure pour Node.js, consultez la [référence complète](https://azure.github.io/azure-sdk-for-node/).

Gérez vos ressources CDN avec [PowerShell](cdn-manage-powershell.md).

