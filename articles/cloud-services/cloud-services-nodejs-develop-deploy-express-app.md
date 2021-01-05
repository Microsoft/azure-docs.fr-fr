---
title: Création et déploiement d’une application Node.js Express dans Azure Cloud Services
titleSuffix: Azure Cloud Services
description: Utilisez ce didacticiel pour créer une application à l’aide du module Express, qui fournit une infrastructure MVC pour la création d’applications web Node.js.
services: cloud-services
documentationcenter: nodejs
author: tgore03
ms.service: cloud-services
ms.devlang: nodejs
ms.topic: article
ms.date: 08/17/2017
ms.author: tagore
ms.custom: devx-track-js
ms.openlocfilehash: 7ecaccf36988e94d0cb1114bd04c5d571d4fe86d
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92071861"
---
# <a name="build-and-deploy-a-nodejs-web-application-using-express-on-an-azure-cloud-services"></a>Création et déploiement d’une application web Node.js avec Express sur Azure Cloud Services

Node.js inclut un ensemble minimal de fonctionnalités dans le runtime principal.
Les développeurs utilisent souvent des modules tiers pour fournir des fonctionnalités supplémentaires lors du développement d'une application Node.js. Dans ce didacticiel, vous allez créer une application en utilisant le module [Express](https://github.com/expressjs/express), qui fournit un framework MVC pour la création d’applications web Node.js.

Voici une capture d’écran de l’application terminée :

![Navigateur Web affichant Bienvenue sur Express dans Azure](./media/cloud-services-nodejs-develop-deploy-express-app/node36.png)

## <a name="create-a-cloud-service-project"></a>Création d'un projet de service cloud
[!INCLUDE [install-dev-tools](../../includes/install-dev-tools.md)]

Procédez comme suit pour créer un projet de service cloud nommé « expressapp » :

1. À partir du **menu Démarrer** ou de l’**écran d’accueil**, recherchez **Windows PowerShell**. Enfin, cliquez avec le bouton droit sur **Windows PowerShell** et sélectionnez **Exécuter en tant qu’administrateur**.

    ![Icône Azure PowerShell](./media/cloud-services-nodejs-develop-deploy-express-app/azure-powershell-start.png)
2. Remplacez les répertoires du répertoire **c:\\node**, puis entrez les commandes suivantes pour créer une solution nommée **expressapp** et un rôle web nommé **WebRole1** :

   ```powershell
   PS C:\node> New-AzureServiceProject expressapp
   PS C:\Node\expressapp> Add-AzureNodeWebRole
   PS C:\Node\expressapp> Set-AzureServiceProjectRole WebRole1 Node 0.10.21
   ```

   > [!NOTE]
   > Par défaut, **Add-AzureNodeWebRole** utilise une ancienne version de Node.js. L'instruction **Set-AzureServiceProjectRole** ci-dessus indique à Azure d'utiliser la version v0.10.21 de Node.  Notez que les paramètres respectent la casse.  Vous pouvez vérifier que la version correcte de Node.js a été sélectionnée en vérifiant la propriété **moteurs** dans **WebRole1\package.json**.
>
>

## <a name="install-express"></a>Installation d'Express
1. Installez le générateur Express en exécutant la commande suivante :

    ```powershell
    PS C:\node\expressapp> npm install express-generator -g
    ```

    Le résultat de la commande npm doit ressembler à l'exemple ci-dessous :

    ![Windows PowerShell affichant le résultat de la commande npm install express.](./media/cloud-services-nodejs-develop-deploy-express-app/express-g.png)
2. Remplacez les répertoires du répertoire **WebRole1** et utilisez la commande express pour créer une application :

    ```powershell
    PS C:\node\expressapp\WebRole1> express
    ```

    Vous êtes invité à remplacer votre application précédente. Tapez **y** ou **yes** pour continuer. Express génère le fichier app.js et une structure de dossiers pour la création de votre application.

    ![Résultat de la commande express](./media/cloud-services-nodejs-develop-deploy-express-app/node23.png)
3. Pour installer les dépendances supplémentaires définies dans le fichier package.json, entrez la commande suivante :

    ```powershell
    PS C:\node\expressapp\WebRole1> npm install
    ```

   ![Résultat de la commande npm install](./media/cloud-services-nodejs-develop-deploy-express-app/node26.png)
4. Utilisez la commande suivante pour copier le fichier **bin/www** vers **server.js**. De cette façon, le service cloud peut trouver le point d'entrée pour cette application.

    ```powershell
    PS C:\node\expressapp\WebRole1> copy bin/www server.js
    ```

   Une fois cette commande terminée, un fichier **server.js** doit se trouver dans le répertoire WebRole1.
5. Modifiez **server.js** pour supprimer l’un des caractères « . » de la ligne suivante.

    ```js
    var app = require('../app');
    ```

   Une fois cette modification apportée, la ligne doit apparaître comme suit.

    ```js
    var app = require('./app');
    ```

   Cette modification est nécessaire, car nous avons déplacé le fichier (anciennement **bin/www**) vers le même répertoire que le fichier d'application requis. Une fois cette modification effectuée, enregistrez le fichier **server.js** .
6. Utilisez la commande suivante pour exécuter l'application dans l'émulateur Azure :

    ```powershell
    PS C:\node\expressapp\WebRole1> Start-AzureEmulator -launch
    ```

    ![Page Web contenant Bienvenue sur Express.](./media/cloud-services-nodejs-develop-deploy-express-app/node28.png)

## <a name="modifying-the-view"></a>Modification de la vue
À présent, modifiez la vue pour afficher le message « Bienvenue sur Express dans Azure ».

1. Entrez la commande suivante pour ouvrir le fichier index.jade :

    ```powershell
    PS C:\node\expressapp\WebRole1> notepad views/index.jade
    ```

   ![Contenu du fichier index.jade.](./media/cloud-services-nodejs-develop-deploy-express-app/getting-started-19.png)

   Jade est le moteur de vue par défaut des applications Express. Pour plus d’informations sur le moteur de vue Jade, consultez [http://jade-lang.com][http://jade-lang.com].
2. Modifiez la dernière ligne du texte en ajoutant **dans Azure**.

   ![Dernières lignes du fichier index.jade : p Bienvenue sur \#{title} dans Azure](./media/cloud-services-nodejs-develop-deploy-express-app/node31.png)
3. Enregistrez le fichier et quittez le Bloc-notes.
4. Actualisez votre navigateur pour afficher vos modifications.

   ![Navigateur Web affichant la page Bienvenue sur Express dans Azure](./media/cloud-services-nodejs-develop-deploy-express-app/node32.png)

Après avoir testé l'application, utilisez la cmdlet **Stop-AzureEmulator** pour arrêter l'émulateur.

## <a name="publishing-the-application-to-azure"></a>Publication de l'application dans Azure
Dans la fenêtre Azure PowerShell, utilisez la cmdlet **Publish-AzureServiceProject** pour déployer l'application dans un service cloud.

```powershell
PS C:\node\expressapp\WebRole1> Publish-AzureServiceProject -ServiceName myexpressapp -Location "East US" -Launch
```

Une fois le déploiement terminé, votre navigateur s'ouvre et affiche la page web.

![Navigateur web affichant la page Express. L'URL indique que la page est maintenant hébergée sur Azure.](./media/cloud-services-nodejs-develop-deploy-express-app/node36.png)

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez le [Centre pour développeurs Node.js](/azure/developer/javascript/).

[Node.js Web Application]: https://www.windowsazure.com/develop/nodejs/tutorials/getting-started/
[Express]: https://expressjs.com/
[http://jade-lang.com]: http://jade-lang.com