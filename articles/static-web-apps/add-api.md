---
title: Ajouter une API dans Azure Static Web Apps avec Azure Functions
description: Prenez en main Azure Static Web Apps en ajoutant une API serverless à votre application web statique à l’aide d’Azure Functions.
services: static-web-apps
author: manekinekko
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/29/2020
ms.author: wachegha
ms.custom: devx-track-js
ms.openlocfilehash: 53be2ee9b6b3ab46f081747aec58b9c99fc54f0d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349954"
---
# <a name="add-an-api-to-azure-static-web-apps-preview-with-azure-functions"></a>Ajouter une API dans Azure Static Web Apps (en préversion) avec Azure Functions

Vous pouvez ajouter des API serverless à Azure Static Web Apps via l’intégration à Azure Functions. Cet article explique comment ajouter et déployer une API sur un site Azure Static Web Apps.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif.
  - Si vous ne possédez pas de compte, vous pouvez [créer un compte gratuit](https://azure.microsoft.com/free).
- [Visual Studio Code](https://code.visualstudio.com/)
- [Extension Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) pour Visual Studio Code
- Extension [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) de Visual Studio Code.
- [Node.js](https://nodejs.org/download/) pour exécuter l’application API localement

## <a name="create-a-git-repository"></a>Créer un dépôt Git

Les étapes suivantes montrent comment créer un référentiel et cloner les fichiers sur votre ordinateur.

1. Vérifiez que vous êtes connecté à GitHub et accédez à [https://github.com/staticwebdev/vanilla-basic/generate](https://github.com/login?return_to=/staticwebdev/vanilla-basic/generate) pour créer un dépôt.
1. Dans la zone _Repository name_ (Nom du référentiel), entrez **my-vanilla-api**.
1. Cliquez sur **Create repository from template** (Créer un référentiel à partir du modèle).

   :::image type="content" source="media/add-api/create-repository.png" alt-text="Créer un référentiel à partir de vanilla-basic":::

Une fois votre projet créé, copiez l’URL dans votre navigateur pour le nouveau dépôt. Vous utilisez cette URL dans Visual Studio Code pour cloner le dépôt Git.

1. Appuyez sur **F1** pour ouvrir la commande dans la palette de commandes
1. Collez l’URL dans l’invite _Git: Clone_, puis appuyez sur **Entrée**.

   :::image type="content" source="media/add-api/vscode-git-0.png" alt-text="Cloner un projet GitHub à l’aide de Visual Studio Code":::

    Suivez les invites pour sélectionner un emplacement de dépôt auquel cloner le projet.

## <a name="create-the-api"></a>Création de l’API

Ensuite, vous créez un projet Azure Functions en tant qu’API de l’application. 

1. Dans le projet _my-vanilla-api_, créez un sous-dossier nommé **api**.
1. Appuyez sur **F1** pour ouvrir la palette de commandes
1. Entrez **Azure Functions : Create New Project...** (Créer un projet...)
1. Appuyez sur **Entrée**
1. Choisissez **Parcourir**
1. Sélectionner le dossier **api** comme répertoire de votre espace de travail de projet
1. Choisissez **Select** (Sélectionner)

   :::image type="content" source="media/add-api/create-azure-functions-vscode-1.png" alt-text="Capture d’écran montrant le dossier API et le bouton Sélectionner.":::

1. Quand vous y êtes invité, indiquez les informations suivantes :

    - _Sélectionnez un langage_ : Choisir **JavaScript**
    - _Sélectionner un modèle pour la première fonction de votre projet_ : Choisir **HTTP trigger** (Déclencheur HTTP)
    - _Fournir un nom de fonction_ : Entrez **GetMessage**.
    - _Niveau d’autorisation_ : Choisissez l’option **Anonymous** (Anonyme), qui permet à quiconque d’appeler le point de terminaison de votre fonction.
        - Pour en savoir plus sur les niveaux d’autorisation, consultez [Clés d’autorisation](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys).

Visual Studio Code génère un projet Azure Functions avec un fonction déclenchée HTTP.

Votre application a maintenant une structure de projet similaire à celle de l’exemple suivant.

```files
├── api
│   ├── GetMessage
│   │   ├── function.json
│   │   ├── index.js
│   │   └── sample.dat
│   ├── host.json
│   ├── local.settings.json
│   ├── package.json
│   └── proxies.json
├── index.html
├── readme.md
└── styles.css
```

Ensuite, vous allez modifier la fonction `GetMessage` pour retourner un message au front-end.

1. Mettez à jour la fonction `GetMessage` sous _api/GetMessage/index.js_ avec le code suivant.

    ```javascript
    module.exports = async function (context, req) {
      context.res = {
        body: {
          text: "Hello from the API"
        }
      };
    };
    ```

1. Mettez à jour la configuration de `GetMessage` sous `api/GetMessage/function.json` avec les paramètres suivants.

    ```json
    {
      "bindings": [
        {
          "authLevel": "anonymous",
          "type": "httpTrigger",
          "direction": "in",
          "name": "req",
          "methods": [
            "get"
          ],
          "route": "message"
        },
        {
          "type": "http",
          "direction": "out",
          "name": "res"
        }
      ]
    }
    ```

Avec les paramètres ci-dessus, le point de terminaison de l’API est :

- Déclenché avec une requête HTTP adressée à la fonction
- Disponible pour toutes les requêtes, quel que soit l’état de l’authentification
- Exposé via l’itinéraire _/api/message_

## <a name="run-the-api-locally"></a>Exécuter l’API localement

Visual Studio Code s’intègre à [Azure Functions Core Tools](../azure-functions/functions-run-local.md) pour vous permettre d’exécuter ce projet sur votre ordinateur de développement local avant toute publication sur Azure.

> [!TIP]
> Avant de poursuivre, vérifiez que vous avez toutes les ressources listées dans la section [Prérequis](#prerequisites) installées.

1. Exécutez la fonction en appuyant sur **F5** pour démarrer l’application Functions.

1. Si Azure Functions Core Tools n’est pas encore installé, sélectionnez **Installer** à l’invite.

    Core Tools montre la sortie de l’application en cours d’exécution dans le panneau _Terminal_. Dans la sortie, vous pouvez voir le point de terminaison de l’URL de votre fonction déclenchée par HTTP en train de s’exécuter localement.

    :::image type="content" source="media/add-api/create-azure-functions-vscode-2.png" alt-text="Capture d’écran montrant l’onglet TERMINAL dans lequel vous pouvez voir l’URL.":::

1. Avec Core Tools en cours d’exécution, accédez à l’URL suivante pour vérifier que l’API s’exécute correctement : `http://localhost:7071/api/message`.

   La réponse figurant dans le navigateur doit ressembler à l’exemple suivant :

   :::image type="content" source="media/add-api/create-azure-functions-vscode-3.png" alt-text="Capture d’écran montrant une réponse texte dans un navigateur.":::

1. Pour arrêter la session de débogage, appuyez sur **Maj+F5**.

### <a name="call-the-api-from-the-application"></a>Appeler l’API à partir de l’application

Lors du déploiement sur Azure, les demandes envoyées à l’API sont automatiquement routées vers l’application Functions pour les demandes envoyées à la route `api`. Si vous travaillez en local, vous devez configurer les paramètres de l’application pour qu’ils proxysent les demandes adressées à l’API locale.

[!INCLUDE [static-web-apps-local-proxy](../../includes/static-web-apps-local-proxy.md)]

#### <a name="update-html-files-to-access-the-api"></a>Mettre à jour les fichiers HTML pour accéder à l’API

1. Ensuite, mettez à jour le contenu du fichier _index.html_ à l’aide du code suivant pour extraire le texte de la fonction API et l’afficher à l’écran :

   ```html
   <!DOCTYPE html>
   <html lang="en">

   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <link rel="stylesheet" href="styles.css">
     <title>Vanilla JavaScript App</title>
   </head>

   <body>
     <main>
       <h1>Vanilla JavaScript App</h1>
       <p>Loading content from the API: <b id="name">...</b></p>
     </main>

     <script>
       (async function() {
         let { text } = await( await fetch(`/api/message`)).json();
         document.querySelector('#name').textContent = text;
       }())
     </script>
   </body>

   </html>
   ```

1. Appuyez sur **F5** pour démarrer le projet d’API.

1. Appuyez sur **F1** et choisissez **Live Server : Ouvrir avec Live Server**.

    Vous devez maintenant voir le message de l’API dans la page web.

   :::image type="content" source="media/add-api/create-azure-functions-vscode-4.png" alt-text="Capture d’écran montrant le message API dans un navigateur.":::

   > [!NOTE]
   > Vous pouvez utiliser d’autres serveurs ou proxys HTTP pour servir le fichier `index.html`. L’accès au fichier `index.html` n’est pas possible à l’aide de `file:///`.

1. Appuyez sur **Maj+F5** pour arrêter le projet d’API.

### <a name="commit-and-push-your-changes-to-github"></a>Valider et envoyer vos modifications à GitHub

À l’aide de Visual Studio Code, validez et envoyez vos modifications dans le référentiel git distant.

1. Appuyez sur **F1** pour ouvrir la palette de commandes
1. Taper **Git: Commit All**
1. Ajoutez un message de validation et appuyez sur **Entrée**.
1. Appuyez sur **F1**.
1. Tapez **Git: push** et appuyez sur **Entrée**.

## <a name="create-a-static-web-app"></a>Créer une application web statique

1. Accéder au [portail Azure](https://portal.azure.com)
1. Cliquez sur **Create a Resource** (Créer une ressource).
1. Recherchez **Static Web Apps**.
1. Cliquez sur **Static Web Apps (préversion)** .
1. Cliquez sur **Créer**

Ensuite, ajoutez les paramètres propres à l’application.

1. Sélectionnez votre _abonnement Azure_.
1. Sélectionner ou créer un _groupe de ressources_
1. Nommez l’application **my-vanilla-api**.
1. Sélectionner la _région_ la plus proche de vous
1. Sélectionner **SKU** _gratuite_
1. Cliquer sur le bouton **Sign-in with GitHub** (Se connecter à GitHub) et authentifiez-vous auprès de GitHub
1. Sélectionner votre _Organisation_ préférée
1. Sélectionner **my-vanilla-api** dans la liste déroulante _Repository_ (Référentiel)
1. Sélectionner **master** maître dans la liste déroulante _Branch_ (Branche)
1. Sélectionner l’infrastructure de votre choix pour la configuration de build dans la liste déroulante _Présélections de build_

 > Ces champs reflètent la structure du projet par défaut du type d'application. Modifiez les valeurs en fonction de votre application.

Ensuite, ajoutez les détails de build suivants.

1. Entrez **/** dans _App location_ (Emplacement de l’application).
1. Entrez **api** dans _App location_.
1. Désactivez la valeur par défaut de l’_emplacement de l’artefact d’application_ en laissant la zone vide.
1. Cliquez sur **Vérifier + créer**.
1. Cliquez sur le bouton **Créer**

    Une fois que vous avez cliqué sur le bouton _Créer_, Azure effectue deux opérations. Tout d’abord, les services cloud sous-jacents sont créés pour prendre en charge l’application. Ensuite, un processus en arrière-plan commence à créer et à déployer l’application.

1. Cliquez sur le bouton **Accéder à la ressource** pour accéder à la page _Vue d’ensemble_ de l’application web.

    Lorsque l’application est générée en arrière-plan, vous pouvez cliquer sur la bannière qui contient un lien pour afficher l’état de la build.

    :::image type="content" source="media/add-api/github-action-flag.png" alt-text="Workflow GitHub":::

1. Une fois le déploiement terminé, vous pouvez accéder à l’application web en cliquant sur le lien _URL_ situé dans la page _Vue d’ensemble_.

    :::image type="content" source="media/add-api/static-app-url-from-portal.png" alt-text="Accéder à l’URL de l’application statique à partir du Portail Azure":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne souhaitez pas conserver cette application pour une utilisation ultérieure, vous pouvez utiliser la procédure suivante pour supprimer l’application Azure Static Web App et ses ressources associées.

1. Accéder au [portail Azure](https://portal.azure.com)
1. Dans la barre de recherche supérieure, taper **Groupes de ressources**
1. Cliquer sur **Groupes de ressources**
1. Sélectionner **myResourceGroup**
1. Dans la page _myResourceGroup_, assurez-vous que les ressources répertoriées sont bien celles que vous souhaitez supprimer.
1. Sélectionnez **Supprimer**.
1. Taper **myResourceGroup** dans la zone de texte
1. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer les paramètres d’application](./application-settings.md)
