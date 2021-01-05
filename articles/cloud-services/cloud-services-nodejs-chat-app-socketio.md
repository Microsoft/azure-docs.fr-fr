---
title: Application Node.js avec Socket.IO - Azure
description: Utilisez ce didacticiel pour apprendre à héberger une application de conversation basée sur Socket.IO sur Azure. Socket.IO permet une communication en temps réel entre un serveur node.js et des clients.
services: cloud-services
documentationcenter: nodejs
author: tgore03
ms.service: cloud-services
ms.devlang: nodejs
ms.topic: article
ms.date: 08/17/2017
ms.author: tagore
ms.custom: devx-track-js
ms.openlocfilehash: ef7325b53f7d6450acdff4664f3e338c31be9612
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077216"
---
# <a name="build-a-nodejs-chat-application-with-socketio-on-an-azure-cloud-service"></a>Création d’une application de conversation instantanée Node.js avec Socket.IO sur un service cloud Azure

Socket.IO permet une communication en temps réel entre votre serveur node.js et vos clients. Ce didacticiel présente l’hébergement d’une application de conversation instantanée Socket.IO dans Azure. Pour plus d’informations sur Socket.IO, consultez [socket.io](https://socket.io).

Voici une capture d’écran de l’application terminée :

![Fenêtre de navigateur affichant le service hébergé sur Azure][completed-app]

## <a name="prerequisites"></a>Conditions préalables requises
Assurez-vous que les versions et les produits suivants sont installés pour mener à bien l'exemple dans cet article :

* Installez [Visual Studio](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx)
* Installez [Node.js](https://nodejs.org/download/)
* Installez [Python version 2.7.10](https://www.python.org/)

## <a name="create-a-cloud-service-project"></a>Création d'un projet de service cloud
La procédure suivante permet de créer un projet de service cloud hébergeant l'application Socket.IO.

1. À partir du **menu Démarrer** ou de l’**écran d’accueil**, recherchez **Windows PowerShell**. Enfin, cliquez avec le bouton droit sur **Windows PowerShell** et sélectionnez **Exécuter en tant qu’administrateur**.

    ![Icône Azure PowerShell][powershell-menu]
2. Créez un répertoire appelé **c:\\node**.

    ```powershell
    PS C:\> md node
    ```

3. Remplacez les répertoires par le répertoire **c:\\node**

    ```powershell
    PS C:\> cd node
    ```

4. Entrez les commandes suivantes pour créer une solution nommée **chatapp** et un rôle de travail nommé **WorkerRole1** :

    ```powershell
    PS C:\node> New-AzureServiceProject chatapp
    PS C:\Node> Add-AzureNodeWorkerRole
    ```

    Vous obtenez la réponse suivante :

    ![Résultat des cmdlets new-azureservice et add-azurenodeworkerrole](./media/cloud-services-nodejs-chat-app-socketio/socketio-1.png)

## <a name="download-the-chat-example"></a>Téléchargement de l'exemple de conversation instantanée
Pour ce projet, nous allons utiliser un exemple de conversation instantanée obtenu depuis le [référentiel GitHub Socket.IO]. Procédez comme suit pour télécharger l'exemple et l'ajouter au projet créé précédemment.

1. Créez une copie locale du référentiel en utilisant le bouton **Clone** . Vous pouvez aussi télécharger le projet en utilisant le bouton **ZIP** .

   ![Une fenêtre de navigateur avec https://github.com/LearnBoost/socket.io/tree/master/examples/chat, avec l’icône de téléchargement ZIP mis en surbrillance](./media/cloud-services-nodejs-chat-app-socketio/socketio-22.png)
2. Accédez à la structure de répertoires du référentiel local, jusqu’à atteindre le répertoire **examples\\chat**. Copiez le contenu de ce répertoire dans le répertoire **C:\\node\\chatapp\\WorkerRole1** créé précédemment.

   ![Explorateur affichant le contenu du répertoire examples\\chat extrait de l’archive][chat-contents]

   Les éléments mis en surbrillance sur cette capture d’écran sont les fichiers copiés à partir du répertoire **examples\\chat**
3. Dans le répertoire **C:\\node\\chatapp\\WorkerRole1**, supprimez le fichier **server.js**, puis renommez le fichier **app.js** en **server.js**. Cela supprime le fichier **server.js** par défaut créé précédemment par l’applet de commande **Add-AzureNodeWorkerRole** et le remplace par le fichier d’application de l’exemple de conversation instantanée.

### <a name="modify-serverjs-and-install-modules"></a>Modification du fichier Server.js et installation des modules
Avant de tester l'application dans l'émulateur Azure, nous allons apporter quelques modifications mineures. Procédez comme suit avec le fichier server.js :

1. Ouvrez le fichier **server.js** dans Visual Studio ou un éditeur de texte.
2. Recherchez la section **Module dependencies** située au début du fichier server.js, puis remplacez la ligne contenant **sio = require('..//..//lib//socket.io')** par **sio = require('socket.io')**, comme indiqué ci-dessous :

    ```js
    var express = require('express')
      , stylus = require('stylus')
      , nib = require('nib')
    //, sio = require('..//..//lib//socket.io'); //Original
      , sio = require('socket.io');                //Updated
      var port = process.env.PORT || 3000;         //Updated
    ```

3. Pour que l’application écoute le bon port, ouvrez le fichier server.js dans le Bloc-notes (ou votre éditeur favori), puis modifiez la ligne suivante en remplaçant **3000** par **process.env.port**, comme indiqué ci-dessous :

    ```js
    //app.listen(3000, function () {            //Original
    app.listen(process.env.port, function () {  //Updated
      var addr = app.address();
      console.log('   app listening on http://' + addr.address + ':' + addr.port);
    });
    ```

Après avoir enregistré vos modifications dans le fichier **server.js**, procédez comme suit pour installer les modules requis, puis testez l'application dans l'émulateur Azure :

1. Utilisez **Azure PowerShell** pour remplacer les répertoires du répertoire **C:\\node\\chatapp\\WorkerRole1**, puis exécutez la commande suivante pour installer les modules requis par cette application :

    ```powershell
    PS C:\node\chatapp\WorkerRole1> npm install
    ```

   Ceci installe les modules répertoriés dans le fichier package.json. Une fois la commande terminée, un texte similaire à celui présenté ci-dessous doit s'afficher :

   ![Résultat de la commande npm install][The-output-of-the-npm-install-command]
2. Comme cet exemple fait partie du référentiel GitHub Socket.IO et qu'il est directement référencé par la bibliothèque Socket.IO par un chemin d'accès relatif, Socket.IO n'est pas référencé dans le fichier package.json. Vous devez donc l'installer en exécutant la commande suivante :

   ```powershell
   PS C:\node\chatapp\WorkerRole1> npm install socket.io --save
    ```

### <a name="test-and-deploy"></a>Test et déploiement
1. Démarrez l'émulateur en exécutant la commande suivante :

    ```powershell
    PS C:\node\chatapp\WorkerRole1> Start-AzureEmulator -Launch
    ```

   > [!NOTE]
   > Si vous rencontrez des problèmes pour le lancement de l’émulateur, par ex. : Start-AzureEmulator : une erreur inattendue est survenue.  Détails : une erreur inattendue a été rencontrée. L’objet de communication System.ServiceModel.Channels.ServiceChannel ne peut pas être utilisé pour la communication car son état est Faulted.
   >
   > Réinstallez AzureAuthoringTools v 2.7.1 et AzureComputeEmulator v 2.7 : assurez-vous que la version correspond.

2. Ouvrez un navigateur et accédez à **http://127.0.0.1** .
3. À l'ouverture de la fenêtre du navigateur, entrez un pseudonyme, puis appuyez sur Entrée.
   Vous pouvez ainsi publier des messages sous un pseudonyme spécifique. Pour tester la fonctionnalité multi-utilisateurs, ouvrez d'autres fenêtres du navigateur en utilisant la même URL et en entrant différents pseudonymes.

   ![Deux fenêtres du navigateur affichant des messages de conversation instantanée des utilisateurs User1 et User2](./media/cloud-services-nodejs-chat-app-socketio/socketio-8.png)
4. Une fois le test de l'application terminé, arrêtez l'émulateur en exécutant la commande suivante :

    ```powershell
    PS C:\node\chatapp\WorkerRole1> Stop-AzureEmulator
    ```

5. Pour déployer l'application dans Azure, utilisez la cmdlet **Publish-AzureServiceProject** . Par exemple :

    ```powershell
    PS C:\node\chatapp\WorkerRole1> Publish-AzureServiceProject -ServiceName mychatapp -Location "East US" -Launch
    ```

   > [!IMPORTANT]
   > Veillez à utiliser un nom unique, sans quoi le processus de publication échoue. Une fois le déploiement terminé, le navigateur s'ouvre et accède au service déployé.
   >
   > Si vous recevez une erreur indiquant que le nom d’abonnement fourni n’existe pas dans le profil de publication importé, vous devez télécharger et importer le profil de publication de votre abonnement avant de le déployer dans Azure. Consultez la section **Déploiement de l'application dans Azure** du didacticiel [Création et déploiement d'une application Node.js dans Azure Cloud Services](https://azure.microsoft.com/develop/nodejs/tutorials/getting-started/)
   >
   >

   ![Fenêtre de navigateur affichant le service hébergé sur Azure][completed-app]

   > [!NOTE]
   > Si vous recevez une erreur indiquant que le nom d’abonnement fourni n’existe pas dans le profil de publication importé, vous devez télécharger et importer le profil de publication de votre abonnement avant de le déployer dans Azure. Consultez la section **Déploiement de l'application dans Azure** du didacticiel [Création et déploiement d'une application Node.js dans Azure Cloud Services](https://azure.microsoft.com/develop/nodejs/tutorials/getting-started/)
   >
   >

Votre application s'exécute à présent dans Azure, et peut transmettre des messages de conversation instantanée entre différents clients en utilisant Socket.IO.

> [!NOTE]
> Par souci de simplification, cet exemple présente simplement une conversation instantanée entre des utilisateurs connectés à la même instance. Cela veut dire que si le service cloud crée deux instances de rôle de travail, un utilisateur pourra utiliser la conversation instantanée pour communiquer uniquement avec les autres utilisateurs connectés à la même instance de rôle de travail. Pour mettre à l'échelle l'application afin d'utiliser plusieurs instances de rôle, vous pouvez utiliser une technologie telle que Service Bus pour partager l'état de stockage Socket.IO entre plusieurs instances. Pour en savoir plus, consultez les exemples d’utilisation des files d’attente et rubriques de Service Bus dans le [référentiel GitHub Kit de développement logiciel (SDK) Azure pour Node.js](https://github.com/WindowsAzure/azure-sdk-for-node).
>
>

## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous a montré comment créer une application de conversation instantanée de base, hébergée dans un service cloud Azure. Pour plus d’informations sur l’hébergement de cette application sur un site web Azure, consultez la page [Création d’une application de conversation instantanée Node.js avec Socket.IO sur un site web Azure][chatwebsite].

Pour plus d'informations, consultez aussi le [Centre pour développeurs Node.js](/azure/developer/javascript/).

[chatwebsite]: ./cloud-services-nodejs-develop-deploy-app.md

[Azure SLA]: https://www.windowsazure.com/support/sla/
[Azure SDK for Node.js GitHub repository]: https://github.com/WindowsAzure/azure-sdk-for-node
[completed-app]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-10.png
[Azure SDK for Node.js]: https://www.windowsazure.com/develop/nodejs/
[Node.js Web Application]: https://www.windowsazure.com/develop/nodejs/tutorials/getting-started/
[référentiel GitHub Socket.IO]: https://github.com/LearnBoost/socket.io/tree/0.9.14
[Azure Considerations]: #windowsazureconsiderations
[Hosting the Chat Example in a Worker Role]: #hostingthechatexampleinawebrole
[Summary and Next Steps]: #summary
[powershell-menu]: ./media/cloud-services-nodejs-chat-app-socketio/azure-powershell-start.png

[chat example]: https://github.com/LearnBoost/socket.io/tree/master/examples/chat
[chat-example-view]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-22.png


[chat-contents]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-5.png
[The-output-of-the-npm-install-command]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-7.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-chat-app-socketio/socketio-9.png