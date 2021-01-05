---
title: 'Démarrage rapide : Créer une application web Node.js'
description: Déployez votre premier programme Hello World Node.js dans Azure App Service en quelques minutes.
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 08/01/2020
ms.custom: mvc, devcenter, seodec18
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
ms.openlocfilehash: b42abfda863ed38eb3c29004630f905edef075c4
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359100"
---
# <a name="create-a-nodejs-web-app-in-azure"></a>Créer une application web Node.js dans Azure

::: zone pivot="platform-windows"  

Commencez avec Azure App Service en créant une application Node.js/Express localement à l’aide de Visual Studio Code et en la déployant dans le cloud. Comme vous utilisez un niveau App Service gratuit, vous pouvez suivre ce guide de démarrage rapide gratuitement.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension).
- <a href="https://git-scm.com/" target="_blank">Installez Git</a>
- [Node.js et npm](https://nodejs.org). Exécutez la commande `node --version` pour vérifier que Node.js est installé.
- [Visual Studio Code](https://code.visualstudio.com/).
- [Extension Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) pour Visual Studio Code.

## <a name="clone-and-run-a-local-nodejs-application"></a>Cloner et exécuter une application Node.js locale

1. Sur votre ordinateur local, ouvrez un terminal et clonez l’exemple de référentiel :

    ```bash
    git clone https://github.com/Azure-Samples/nodejs-docs-hello-world
    ```

1. Accédez au nouveau dossier de l’application :

    ```bash
    cd nodejs-docs-hello-world
    ```

1. Démarrez l’application pour la tester localement :

    ```bash
    npm start
    ```
    
1. Ouvrez votre navigateur et accédez à `http://localhost:1337`. Le navigateur doit afficher le message « Hello World! ».

1. Appuyez sur **Ctrl**+**C** dans le terminal pour arrêter le serveur.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=create-app)

## <a name="deploy-the-app-to-azure"></a>Déploiement de l’application dans Azure

Dans cette section, vous déployez votre application Node.js sur Azure à l’aide de VS Code et de l’extension Azure App Service.

1. Dans le terminal, vérifiez que vous vous trouvez dans le dossier *nodejs-docs-hello-world* , puis démarrez Visual Studio Code à l’aide de la commande suivante :

    ```bash
    code .
    ```

1. Dans la barre d’activité de VS Code, sélectionnez le logo Azure pour afficher l’explorateur **AZURE APP SERVICE**. Sélectionnez **Se connecter à Azure...** et suivez les instructions. (Consultez la section [Résolution des problèmes de connexion à Azure](#troubleshooting-azure-sign-in) plus bas si vous rencontrez des erreurs.) Quand vous êtes connecté, l’explorateur doit afficher le nom de votre abonnement Azure.

    ![Connexion à Azure](media/quickstart-nodejs/sign-in.png)

1. Dans l’explorateur **AZURE APP SERVICE** de VS Code, sélectionnez l’icône représentant une flèche bleue pointant vers le haut pour déployer votre application sur Azure. (Vous pouvez également appeler la même commande à partir de la **palette de commandes** ( **Ctrl**+**Maj**+**P** ) en tapant « déployer sur l’application web » et en choisissant **Azure App Service : Déployer sur l’application web** ).

    :::image type="content" source="media/quickstart-nodejs/deploy.png" alt-text="Capture d’écran d’Azure App Service dans VS Code montrant l’icône de flèche bleue sélectionnée.":::
        
1. Choisissez le dossier *nodejs-docs-hello-world*.

1. Choisissez une option de création basée sur le système d’exploitation sur lequel vous souhaitez effectuer le déploiement :

    - Linux : Sélectionnez **Créer une application web**.
    - Windows : Sélectionnez **Créer une application web... Avancé**

1. Saisissez un nom unique pour votre application web et appuyez sur **Entrée**. Le nom doit être unique sur l’ensemble d’Azure et comporter uniquement des caractères alphanumériques (« A-Z », « a-z » et « 0-9 ») et des traits d’union (« - »).

1. Si vous ciblez Linux, sélectionnez une version de Node.js quand vous y êtes invité. Une version **LTS** est recommandée.

1. Si vous ciblez Windows, suivez les invites supplémentaires :
    1. Sélectionnez **Créer un groupe de ressources** et entrez un nom pour le groupe de ressources, par exemple `AppServiceQS-rg`.
    1. Sélectionnez **Windows** comme système d’exploitation.
    1. Sélectionnez **Créer un plan App Service** , entrez un nom pour le plan (par exemple, `AppServiceQS-plan`), puis sélectionnez le niveau tarifaire **F1 Gratuit**.
    1. À l’invite Application Insights, choisissez **Ignorer pour le moment**.
    1. Choisissez une région près de chez vous ou à proximité des ressources auxquelles vous souhaitez accéder.

1. Quand vous avez répondu à toutes les invites, VS Code affiche les ressources Azure qui sont créées pour votre application dans sa fenêtre de notification.

    Si vous effectuez un déploiement dans Linux, sélectionnez **Oui** quand vous êtes invité à mettre à jour votre configuration pour exécuter `npm install` sur le serveur Linux cible.

    ![Invite concernant la mise à jour de la configuration sur le serveur Linux cible](media/quickstart-nodejs/server-build.png)

1. Sélectionnez **Oui** quand le message suivant s’affiche : **Toujours déployer l’espace de travail "nodejs-docs-hello-world" sur (nom de l’application)"** . Quand vous sélectionnez **Oui** , vous indiquez à VS Code de cibler automatiquement la même application web App Service pour les déploiements suivants.

1. Si vous effectuez le déploiement sur Linux, à l’issue du déploiement, sélectionnez **Parcourir le site Web** dans l’invite pour voir l’application web que vous venez de déployer. Le navigateur doit afficher le message « Hello World! ».

1. Si vous effectuez le déploiement sur Windows, vous devez d’abord définir le numéro de version Node.js pour l’application web :

    1. Dans VS Code, développez le nœud de votre nouveau service d’application, cliquez avec le bouton droit sur **Paramètres de l’application** , puis sélectionnez **Ajouter un nouveau paramètre...**  :

        ![Commande d’ajout de paramètre d’application](media/quickstart-nodejs/add-setting.png)

    1. Entrez `WEBSITE_NODE_DEFAULT_VERSION` pour la clé du paramètre.
    1. Entrez `10.15.2` pour la valeur du paramètre.
    1. Cliquez avec le bouton droit sur le nœud du service d’application et sélectionnez **Redémarrer**.

        ![Commande de redémarrage du service d’application](media/quickstart-nodejs/restart.png)

    1. Cliquez de nouveau sur le nœud du service d’application avec le bouton droit et sélectionnez **Parcourir le site Web**.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)

### <a name="troubleshooting-azure-sign-in"></a>Résolution des problèmes de connexion à Azure

Si vous rencontrez l’erreur **« Cannot find subscription with name [subscription ID] »** (L’abonnement avec le nom [ID d’abonnement] est introuvable) quand vous vous connectez à Azure, cela peut être dû au fait que vous vous trouvez derrière un proxy qui vous empêche d’atteindre l’API Azure. Configurez les variables d’environnement `HTTP_PROXY` et `HTTPS_PROXY` avec vos informations de proxy dans votre terminal à l’aide de `export`.

```bash
export HTTPS_PROXY=https://username:password@proxy:8080
export HTTP_PROXY=http://username:password@proxy:8080
```

Si la définition des variables d’environnement ne corrige pas le problème, contactez-nous en sélectionnant le bouton **J’ai rencontré un problème** plus haut.

### <a name="update-the-app"></a>Mettre à jour l’application

Pour déployer les modifications apportées à cette application, vous pouvez effectuer les modifications dans VS Code, enregistrer vos fichiers, puis appliquer la procédure suivie précédemment en choisissant l’application existante au lieu d’en créer une nouvelle.

## <a name="viewing-logs"></a>Consultation des journaux

Vous pouvez voir la sortie de journal (appels à `console.log`) à partir de l’application directement dans la fenêtre de sortie de VS Code.

1. Dans l’explorateur **AZURE APP SERVICE** , cliquez avec le bouton droit sur le nœud de l’application, puis choisissez **Commencer le streaming des journaux**.

    ![Commencer le streaming des journaux](media/quickstart-nodejs/view-logs.png)

1. Lorsque vous y êtes invité, choisissez d’activer la journalisation et de redémarrer l’application. Une fois l’application redémarrée, la fenêtre de résultats de VS Code s’ouvre avec une connexion au flux de journaux. 

    :::image type="content" source="media/quickstart-nodejs/enable-restart.png" alt-text="Capture d’écran de l’invite de Visual Studio Code pour activer la journalisation et redémarrer l’application avec le bouton Oui sélectionné.":::

1. Après quelques secondes, la fenêtre de sortie affiche un message indiquant que vous êtes connecté au service de streaming de journaux. Vous pouvez générer plus d’activités de sortie en actualisant la page dans le navigateur.

    <pre>
    Connecting to log stream...
    2020-03-04T19:29:44  Welcome, you are now connected to log-streaming service. The default timeout is 2 hours.
    Change the timeout with the App Setting SCM_LOGSTREAM_TIMEOUT (in seconds).
    </pre>

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=tailing-logs)

## <a name="next-steps"></a>Étapes suivantes

Félicitations, vous avez terminé ce démarrage rapide !

> [!div class="nextstepaction"]
> [Tutoriel : Application Node.js avec MongoDB](tutorial-nodejs-mongodb-app.md)

> [!div class="nextstepaction"]
> [Configurer une application Node.js](configure-language-nodejs.md)

Consultez les autres extensions Azure.

* [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Outils Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Outils Azure CLI](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Outils Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

Vous pouvez également tous les obtenir en installant le pack d’extension [Pack de nœuds pour Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).
::: zone-end

::: zone pivot="platform-linux"  
## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas de compte Azure, [inscrivez-vous dès maintenant](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension) pour obtenir un compte gratuit avec 200 $ de crédits Azure, ce qui vous permettra d’essayer toutes les combinaisons de services.

Vous devez avoir installé [Visual Studio Code](https://code.visualstudio.com/) ainsi que [Node.js et npm](https://nodejs.org/en/download), le gestionnaire de package Node.js.

Vous devez également installer l’[extension Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice), que vous pouvez utiliser pour créer, gérer et déployer des Web Apps Linux sur la plateforme PaaS (Platform as a Service) Azure.

### <a name="sign-in"></a>Se connecter

Une fois l’extension installée, connectez-vous à votre compte Azure. Dans la barre d’activité, sélectionnez le logo Azure pour afficher l’Explorateur **AZURE APP SERVICE**. Sélectionnez **Se connecter à Azure...** et suivez les instructions.

![se connecter à Azure](./media/quickstart-nodejs/sign-in.png)

### <a name="troubleshooting"></a>Dépannage

Si vous rencontrez l’erreur **« Cannot find subscription with name [subscription ID] »** (L’abonnement avec le nom [ID d’abonnement] est introuvable), cela peut être dû au fait que vous vous trouvez derrière un proxy qui vous empêche d’atteindre l’API Azure. Configurez les variables d’environnement `HTTP_PROXY` et `HTTPS_PROXY` avec vos informations de proxy dans votre terminal à l’aide de `export`.

```sh
export HTTPS_PROXY=https://username:password@proxy:8080
export HTTP_PROXY=http://username:password@proxy:8080
```

Si la définition des variables d’environnement ne corrige pas le problème, contactez-nous en sélectionnant le bouton **J’ai rencontré un problème** ci-dessous.

### <a name="prerequisite-check"></a>Vérification du prérequis

Avant de continuer, assurez-vous que tous les composants requis ont bien été installés et configurés.

Dans VS Code, vous devriez voir votre adresse e-mail Azure dans la barre d’État et votre abonnement dans l’Explorateur **AZURE APP SERVICE**.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=getting-started)

## <a name="create-your-nodejs-application"></a>Créer votre application Node.js

Créez ensuite une application Node.js pouvant être déployée dans le Cloud. Ce démarrage rapide utilise un générateur d’applications pour déployer rapidement l’application depuis un terminal.

> [!TIP]
> Si vous avez déjà terminé le [tutoriel Node.js](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial), vous pouvez passer directement à la section [Déployer sur Azure](#deploy-to-azure).

### <a name="scaffold-a-new-application-with-the-express-generator"></a>Structurer une nouvelle application avec le générateur Express

[Express](https://www.expressjs.com) est un framework couramment utilisé pour la création et l’exécution des applications Node.js. Vous pouvez créer/définir la structure d’une nouvelle application Express à l'aide de l’outil [Générateur Express](https://expressjs.com/en/starter/generator.html). Le générateur Express est fourni en tant que module npm et peut être exécuté directement (sans installation) à l’aide de l’outil en ligne de commande npm `npx`.

```bash
npx express-generator myExpressApp --view pug --git
```

Les paramètres `--view pug --git` indiquent au générateur qu’il doit utiliser le moteur de modèle [pug](https://pugjs.org/api/getting-started.html) (auparavant appelé `jade`) et créer un fichier `.gitignore`.

Pour installer toutes les dépendances de l’application, accédez au nouveau dossier et exécutez `npm install`.

```bash
cd myExpressApp
npm install
```

### <a name="run-the-application"></a>Exécution de l'application

Assurez-vous ensuite que l’application s’exécute. À partir du terminal, démarrez l’application à l’aide de la commande `npm start` pour démarrer le serveur.

```bash
npm start
```

À présent, ouvrez votre navigateur et accédez à `http://localhost:3000`. Voici ce que vous devriez obtenir :

![Exécution de l’application Express](./media/quickstart-nodejs/express.png)

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=create-app)

## <a name="deploy-to-azure"></a>Déployer dans Azure

Dans cette section, vous déployez votre application Node.js à l’aide de VS Code et de l’extension Azure App Service. Ce démarrage rapide utilise le modèle de déploiement le plus basique dans lequel votre application est compressée et déployée sur une application Web Azure sur Linux.

### <a name="deploy-using-azure-app-service"></a>Déployer à l’aide d’Azure App Service

Tout d’abord, ouvrez le dossier de votre application dans VS Code.

```bash
code .
```

Dans l’explorateur **AZURE APP SERVICE** , sélectionnez l’icône représentant une flèche bleue pointant vers le haut pour déployer votre application sur Azure.

:::image type="content" source="./media/quickstart-nodejs/deploy.png" alt-text="Capture d’écran du menu Azure App Service dans Visual Studio Code avec la flèche bleue Déployer sélectionnée.":::

> [!TIP]
> Vous pouvez également effectuer le déploiement à partir de la **palette de commandes** (Ctrl + Maj + P) en tapant « déployer sur l'application web » et en exécutant l’ **Azure App Service : Commande Déployer sur l'application web**.

1. Choisissez le répertoire actuellement ouvert, `myExpressApp`.

1. Choisissez **Créer une application web** pour déployer sur App Service sur Linux par défaut.

1. Saisissez un nom unique pour votre application Web et appuyez sur Entrée. Les caractères valides dans un nom d’application sont les suivants : « a-z », « 0-9 » et « - ».

1. Choisissez votre **version Node.js** (LTS est recommandé).

    Le canal de notification affiche les ressources Azure qui sont créées pour votre application.

1. Sélectionnez **Oui** lorsque vous êtes invité à mettre à jour votre configuration pour exécuter `npm install` sur le serveur cible. Votre application est alors déployée.

    :::image type="content" source="./media/quickstart-nodejs/server-build.png" alt-text="Capture d’écran de l’invite de mise à jour de votre configuration sur le serveur cible avec le bouton Oui sélectionné.":::

1. Lorsque le déploiement démarre, vous êtes invité à mettre à jour votre espace de travail afin que les déploiements ultérieurs ciblent automatiquement la même application web App Service. Choisissez **Oui** pour vous assurer que vos modifications sont déployées sur la bonne application.

    :::image type="content" source="./media/quickstart-nodejs/save-configuration.png" alt-text="Capture d’écran de l’invite de mise à jour de votre espace de travail avec le bouton Oui sélectionné.":::

> [!TIP]
> Assurez-vous que votre application utilise le port d’écoute fourni par la variable d’environnement PORT `process.env.PORT`.

### <a name="browse-the-app-in-azure"></a>Accéder à l’application dans Azure

Une fois le déploiement terminé, sélectionnez **Parcourir le site web** dans l’invite pour afficher l’application web que vous venez de déployer.

### <a name="troubleshooting"></a>Dépannage

Si vous voyez s’afficher le message d’erreur **« Vous n’êtes pas autorisé à afficher ce répertoire ou cette page. »** , c’est que l’application n’a probablement pas réussi à démarrer correctement. Accédez à la section suivante et affichez la sortie du journal pour rechercher et corriger l’erreur. Si vous n’êtes pas en mesure de la corriger, contactez-nous en sélectionnant le bouton **J’ai rencontré un problème** ci-dessous. Nous sommes là pour vous aider !

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)

### <a name="update-the-app"></a>Mettre à jour l’application

Vous pouvez déployer les modifications apportées à cette application en utilisant le même procédure et en choisissant l’application existante au lieu d’en créer une nouvelle.

## <a name="viewing-logs"></a>Consultation des journaux

Dans cette section, vous allez apprendre à visionner les journaux à partir de l’application App Service en cours d’exécution. Tous les appels à `console.log` dans l’application sont affichés dans la fenêtre de résultats de Visual Studio Code.

Recherchez l’application dans l'explorateur **AZURE APP SERVICE** , cliquez avec le bouton droit de la souris sur l’application, puis choisissez **Afficher les journaux d’activité de diffusion en continu**.

La fenêtre de résultats de VS Code s’ouvre avec une connexion au flux de journaux.

![Afficher les journaux d’activité de diffusion en continu](./media/quickstart-nodejs/view-logs.png)

:::image type="content" source="./media/quickstart-nodejs/enable-restart.png" alt-text="Capture d’écran de l’invite de VS Code pour permettre la journalisation des fichiers et le redémarrage de l’application web avec le bouton Oui sélectionné.":::

Après quelques secondes, vous verrez s’afficher un message indiquant que vous êtes connecté au service de diffusion en continu de journaux. Actualisez la page plusieurs fois pour voir davantage d’activité.

<pre>
2019-09-20 20:37:39.574 INFO  - Initiating warmup request to container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node
2019-09-20 20:37:55.011 INFO  - Waiting for response to warmup request for container msdocs-vscode-node_2_00ac292a. Elapsed time = 15.4373071 sec
2019-09-20 20:38:08.233 INFO  - Container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node initialized successfully and is ready to serve requests.
2019-09-20T20:38:21  Startup Request, url: /Default.cshtml, method: GET, type: request, pid: 61,1,7, SCM_SKIP_SSL_VALIDATION: 0, SCM_BIN_PATH: /opt/Kudu/bin, ScmType: None
</pre>

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=tailing-logs)

## <a name="next-steps"></a>Étapes suivantes

Félicitations, vous avez terminé ce démarrage rapide !

Consultez ensuite les autres extensions Azure.

* [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Outils Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Outils Azure CLI](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Outils Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

Vous pouvez également tous les obtenir en installant le pack d’extension [Pack de nœuds pour Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).


::: zone-end
