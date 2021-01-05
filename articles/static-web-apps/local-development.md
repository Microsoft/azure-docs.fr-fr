---
title: Configurer le développement local pour Azure Static Web Apps
description: Apprendre à définir votre environnement de développement local pour Azure Static Web Apps
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/08/2020
ms.author: buhollan
ms.custom: devx-track-js
ms.openlocfilehash: 4d6dae8a4f4ed83af3103e95e711bacdb62cf522
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91326165"
---
# <a name="set-up-local-development-for-azure-static-web-apps-preview"></a>Configurer le développement local pour Azure Static Web Apps en préversion

Une instance Azure Static Web Apps est constituée de deux types d’applications différents. La première est une application web pour votre contenu statique. Les applications web sont souvent créées avec des infrastructures et des bibliothèques frontales ou avec des générateurs de site statiques. Le deuxième aspect est l’API, qui est une application Azure Functions qui fournit un environnement de développement back-end complet.

En cas d’exécution dans le cloud, Azure Static Web Apps mappe en toute transparence les requêtes à l’itinéraire `api`entre l’application web et l’application Azure Functions sans nécessiter de configuration CORS. Localement, vous devez configurer votre application pour imiter ce comportement.

Cet article présente les bonnes pratiques recommandées pour le développement local, notamment les concepts suivants :

- Configurer l’application web pour le contenu statique
- Configuration de l’application Azure Functions pour l’API de votre application
- Débogage et exécution de l’application
- Bonne pratiques pour la structure des fichiers et des dossiers de votre application

## <a name="prerequisites"></a>Prérequis

- [Visual Studio Code](https://code.visualstudio.com/)
- [Extension Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) pour Visual Studio Code
- [Extension Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) pour Visual Studio Code
  - Nécessaire uniquement si vous n’utilisez pas d’infrastructure JavaScript front-end ou l’interface CLI d’un générateur de site statique

## <a name="run-projects-locally"></a>Exécuter des projets localement

L’exécution locale d’une application Azure Static Web App implique trois processus, selon que votre projet contient ou non une API.

- Exécution d’un serveur web local
- Exécution de l’API
- Connexion du projet web à l’API

En fonction du mode de création d’un site web, un serveur web local peut ou non être requis pour exécuter l’application dans le navigateur. Lorsque vous utilisez des infrastructures JavaScript front-end et des générateurs de site statiques, cette fonctionnalité est intégrée à leurs interfaces de ligne de commande (CLI) respectives. Les liens suivants pointent vers la référence CLI et vous proposent une sélection d’infrastructures, de bibliothèques et de générateurs.

### <a name="javascript-frameworks-and-libraries"></a>Bibliothèques et infrastructures JavaScript

- [CLI Angular](https://angular.io/cli)
- [CLI Vue](https://cli.vuejs.org/guide/creating-a-project.html)
- [CLI React](https://create-react-app.dev/)

### <a name="static-site-generators"></a>Générateurs de sites statiques

- [CLI Gatsby](https://www.gatsbyjs.org/docs/gatsby-cli/)
- [Hugo](https://gohugo.io/getting-started/quick-start/)
- [Jekyll](https://jekyllrb.com/docs/usage/)

Si vous utilisez un outil CLI pour traiter votre site, vous pouvez vous rendre directement à la section [Exécution de l’API](#run-api-locally).

### <a name="running-a-local-web-server-with-live-server"></a>Exécution d’un serveur web local avec Live Server

L’extension Live Server pour Visual Studio Code fournit un serveur web de développement local qui sert du contenu statique.

#### <a name="create-a-repository"></a>Créer un référentiel

1. Vérifiez que vous êtes connecté à GitHub et accédez à [https://github.com/staticwebdev/vanilla-api/generate](https://github.com/staticwebdev/vanilla-api/generate) pour créer un projet GitHub nommé **vanilla-api**, en utilisant ce modèle.

    :::image type="content" source="media/local-development/vanilla-api.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

1. Ouvrez Visual Studio Code.

1. Appuyez sur **F1** pour ouvrir la palette de commandes.

1. Tapez **clone** dans la zone de recherche, puis sélectionnez **Git: Clone**.

    :::image type="content" source="media/local-development/command-palette-git-clone.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

1. Entrez la valeur suivante dans **URL du dépôt**.

   ```http
   git@github.com:<YOUR_GITHUB_ACCOUNT>/vanilla-api.git
   ```

1. Sélectionnez un emplacement de dossier pour le nouveau projet.

1. Lorsque vous êtes invité à ouvrir le référentiel cloné, sélectionnez **Ouvrir**.

    :::image type="content" source="media/local-development/open-new-window.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

Visual Studio Code ouvre le projet cloné dans l’éditeur.

### <a name="run-the-website-locally-with-live-server"></a>Exécuter le site web localement avec Live Server

1. Appuyez sur **F1** pour ouvrir la palette de commandes.

1. Tapez **Live Server** dans la zone de recherche et sélectionnez **Live Server : Ouvrir avec Live Server**

    Un onglet de navigateur s’ouvre pour afficher l’application.

    :::image type="content" source="media/local-development/vanilla-api-site.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

    Cette application envoie une requête HTTP au point de terminaison `api/message`. Pour le moment, cette requête échoue car la partie API de cette application doit être démarrée.

### <a name="run-api-locally"></a>Exécuter l’API localement

Les API Azure Static Web Apps sont propulsées par Azure Functions. Pour plus d’informations sur l’ajout d’une API à un projet Azure Static Web Apps, consultez [Ajouter une API dans Azure Static Web Apps avec Azure Functions](add-api.md).

Dans le cadre du processus de création d’API, une configuration de lancement est créée pour Visual Studio Code. Cette configuration se trouve dans le dossier _.vscode_. Ce dossier contient tous les paramètres requis pour la génération et l’exécution locales de l’API.

1. Dans Visual Studio Code, appuyez sur **F5** pour démarrer l’API.

1. La nouvelle instance de terminal qui s’ouvre affiche la sortie du processus de build de l’API.

    :::image type="content" source="media/local-development/terminal-api-debug.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

   La barre d’état de Visual Studio Code est maintenant orange. Cette couleur indique que l’API est en cours d’exécution et que le débogueur est joint.

1. Ensuite, appuyez sur **Ctrl/Cmd** et cliquez sur l’URL dans le terminal pour ouvrir une fenêtre de navigateur qui appelle l’API.

    :::image type="content" source="media/local-development/hello-from-api-endpoint.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

### <a name="debugging-the-api"></a>Débogage de l’API

1. Ouvrez le fichier _api/GetMessage/index.js_ dans Visual Studio Code.

1. Cliquez à la ligne 2 dans la marge de gauche pour définir un point d’arrêt. Le point rouge qui apparaît indique que le point d’arrêt est défini.

    :::image type="content" source="media/local-development/breakpoint-set.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

1. Dans le navigateur, actualisez la page en cours d’exécution dans <http://127.0.0.1:7071/api/message>.

1. Le point d’arrêt est atteint dans Visual Studio Code et l’exécution du programme est suspendue.

   :::image type="content" source="media/local-development/breakpoint-hit.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

   Une [expérience de débogage complète est disponible dans Visual Studio Code](https://code.visualstudio.com/Docs/editor/debugging) pour votre API.

1. Appuyez sur le bouton **Continuer** dans la barre de débogage pour poursuivre l’exécution.

    :::image type="content" source="media/local-development/continue-button.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

### <a name="calling-the-api-from-the-application"></a>Appel de l’API à partir de l’application

Une fois déployé, Azure Static Web Apps mappe automatiquement ces requêtes avec les points de terminaison dans le dossier _api_. Ce mappage garantit que les requêtes de l’application à l’API se présentent comme dans l’exemple suivant.

```javascript
let response = await fetch("/api/message");
```

Selon que votre application est générée avec une interface CLI de l’infrastructure JavaScript, il existe deux façons de configurer le chemin d’accès à l’itinéraire `api` lors de l’exécution locale de votre application.

- Fichiers de configuration de l’environnement (recommandés pour les bibliothèques et infrastructures JavaScript)
- Proxy local

### <a name="environment-configuration-files"></a>Fichiers de configuration de l’environnement

Si vous compilez votre application avec des infrastructures front-end possédant une interface CLI, vous devez utiliser des fichiers de configuration d’environnement. Chaque infrastructure ou bibliothèque a une façon différente de gérer ces fichiers de configuration d’environnement. Il est courant de disposer d’un fichier de configuration pour le développement utilisé lorsque votre application s’exécute localement, et d’un fichier de configuration pour la production qui est utilisé lorsque votre application s’exécute en production. Votre interface CLI pour l’infrastructure JavaScript ou le générateur de site statique va automatiquement utiliser le fichier de développement dans le cas d’une exécution locale, et le fichier de production lorsque votre application est générée par Azure Static Web Apps.

Dans le fichier de configuration de développement, vous pouvez spécifier le chemin d’accès à l’API, qui pointe vers l’emplacement local `http:127.0.0.1:7071` où l’API s’exécute localement pour votre site.

```
API=http:127.0.0.1:7071/api
```

Dans le fichier de configuration de production, spécifiez le chemin d’accès à l’API en tant que `api`. De cette façon, votre application appellera l’API via « yoursite.com/api » lors de l’exécution en production.

```
API=api
```

Ces valeurs de configuration peuvent être référencées en tant que variables d’environnement Node dans le code JavaScript de l’application web.

```js
let response = await fetch(`${process.env.API}/message`);
```

Lorsque l’interface de ligne de commande est utilisée pour exécuter votre site en mode de développement ou pour générer le site en production, la valeur `process.env.API` est remplacée par la valeur du fichier de configuration approprié.

Pour plus d’informations sur la configuration des fichiers d’environnement pour les infrastructures et les bibliothèques JavaScript front-end, consultez les articles suivants :

- [Variables d’environnement Angular](https://angular.io/guide/build#configuring-application-environments)
- [React - Ajout de variables d’environnement personnalisées](https://create-react-app.dev/docs/adding-custom-environment-variables/)
- [Vue - Modes et variables d’environnement](https://cli.vuejs.org/guide/mode-and-env.html)

[!INCLUDE [static-web-apps-local-proxy](../../includes/static-web-apps-local-proxy.md)]

##### <a name="restart-live-server"></a>Redémarrer Live Server

1. Dans Visual Studio Code, appuyez sur **F1** pour ouvrir la palette de commandes.

1. Tapez **Live Server** et sélectionnez **Live Server : Arrêter Live Server**.

    :::image type="content" source="media/local-development/stop-live-server.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

1. Appuyez sur **F1** pour ouvrir la palette de commandes.

1. Tapez **Live Server** et sélectionnez **Live Server : Ouvrir avec Live Server**.

1. Actualisez l’application en cours d’exécution à l’adresse `http://locahost:3000`. Le navigateur affiche maintenant le message retourné par l’API.

    :::image type="content" source="media/local-development/hello-from-api.png" alt-text="Nouvelle fenêtre de référentiel GitHub":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer les paramètres d’application](application-settings.md)
