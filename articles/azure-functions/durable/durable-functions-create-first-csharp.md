---
title: Créer votre première fonction durable dans Azure à l’aide de C#
description: Créez et publiez une fonction durable Azure à l’aide de Visual Studio ou de Visual Studio Code.
author: jeffhollan
ms.topic: quickstart
ms.date: 03/18/2020
ms.author: azfuncdf
zone_pivot_groups: code-editors-set-one
ms.openlocfilehash: d5b6ecc12cee983cee0772da8b6f8f26a3b5a8ae
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008174"
---
# <a name="create-your-first-durable-function-in-c"></a>Créer votre première fonction durable dans C\#

*Durable Functions* est une extension d’[Azure Functions](../functions-overview.md) qui vous permet d’écrire des fonctions avec état dans un environnement serverless. L’extension gère l’état, les points de contrôle et les redémarrages à votre place.

::: zone pivot="code-editor-vscode"

Cet article explique comment utiliser Visual Studio Code pour créer et tester en local une fonction durable « Hello world ».  Cette fonction orchestre et chaîne des appels à d’autres fonctions. Vous allez ensuite publier le code de la fonction dans Azure. Ces outils sont disponibles dans le cadre de l’[extension Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) de VS Code.

![La capture d’écran montre une fenêtre Visual Studio Code avec une fonction durable.](./media/durable-functions-create-first-csharp/functions-vscode-complete.png)

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

* Installez [Visual Studio Code](https://code.visualstudio.com/download).

* Installez les extensions VS Code suivantes :
    - [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
    - [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

* Vérifiez que vous disposez de la dernière version [d’Azure Functions Core Tools](../functions-run-local.md).

* Durable Functions nécessite un compte de stockage Azure. Vous avez besoin d’un abonnement Azure.

* Assurez-vous d’avoir la version 3.1 ou ultérieure du [kit SDK .NET Core](https://dotnet.microsoft.com/download) installée.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="create-your-local-project"></a><a name="create-an-azure-functions-project"></a>Créer votre projet local 

Dans cette section, vous utilisez Visual Studio Code pour créer un projet Azure Functions local. 

1. Dans Visual Studio Code, appuyez sur F1 (ou Ctrl/Cmd+Maj+P) pour ouvrir la palette de commandes. Dans la palette de commandes, recherchez et sélectionnez `Azure Functions: Create New Project...`.

    ![Créer un projet Function](media/durable-functions-create-first-csharp/functions-vscode-create-project.png)

1. Désignez un emplacement de dossier vide pour votre projet et choisissez **Sélectionner**.

1. Suivez les invites et fournissez les informations suivantes :

    | Prompt | Valeur | Description |
    | ------ | ----- | ----------- |
    | Sélectionner un langage pour votre projet d’application de fonction | C# | Créez un projet de fonctions C# local. |
    | Sélectionner une version | Azure Functions v3 | Vous voyez cette option uniquement quand les outils Core Tools ne sont pas déjà installés. Dans le cas présent, les outils Core Tools sont installés la première fois que vous exécutez l’application. |
    | Sélectionner un modèle pour la première fonction de votre projet | Ignorer pour le moment | |
    | Sélectionner la façon dont vous souhaitez ouvrir votre projet | Ouvrir dans la fenêtre active | Ouvre à nouveau VS Code dans le dossier que vous avez sélectionné. |

Visual Studio Code installe la solution Azure Functions Core Tools, si nécessaire. Il crée également un projet d’application de fonction dans un dossier. Ce projet contient les fichiers config [host.json](../functions-host-json.md) et [local.settings.json](../functions-run-local.md#local-settings-file).

## <a name="add-functions-to-the-app"></a>Ajouter des fonctions à l’application

Les étapes suivantes utilisent un modèle pour créer le code de fonction durable dans votre projet.

1. Dans la palette de commandes, recherchez et sélectionnez `Azure Functions: Create Function...`.

1. Suivez les invites et fournissez les informations suivantes :

    | Prompt | Valeur | Description |
    | ------ | ----- | ----------- |
    | Sélectionner un modèle pour votre fonction | DurableFunctionsOrchestration | Créer une orchestration Durable Functions |
    | Fournir un nom de fonction | HelloOrchestration | Nom de la classe dans laquelle les fonctions sont créées |
    | Fournir un espace de noms | Company.Function | Espace de noms de la classe générée |

1. Lorsque VS Code vous invite à sélectionner un compte de stockage, choisissez **Sélectionner un compte de stockage**. En suivant les invites, fournissez les informations suivantes pour créer un compte de stockage dans Azure.

    | Prompt | Valeur | Description |
    | ------ | ----- | ----------- |
    | Sélectionner un abonnement | *nom de votre abonnement* | Sélectionner votre abonnement Azure |
    | Sélectionner un compte de stockage | Création d’un nouveau compte de stockage |  |
    | Entrer le nom du nouveau compte de stockage | *nom unique* | Nom du compte de stockage à créer |
    | Sélectionner un groupe de ressources | *nom unique* | Nom du groupe de ressources à créer |
    | Sélectionner un emplacement | *region* | Sélectionner une région proche de vous |

Une classe contenant les nouvelles fonctions est ajoutée au projet. VS Code ajoute également la chaîne de connexion du compte de stockage à *local.settings.json*, et une référence au package NuGet [`Microsoft.Azure.WebJobs.Extensions.DurableTask`](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask) dans le fichier de projet *.csproj*.

Ouvrez le nouveau fichier *HelloOrchestration.cs* pour afficher le contenu. Cette fonction durable est un exemple de chaînage de fonctions simple avec les méthodes suivantes :  

| Méthode | FunctionName | Description |
| -----  | ------------ | ----------- |
| **`RunOrchestrator`** | `HelloOrchestration` | Gère l’orchestration durable. Dans ce cas, l’orchestration démarre, crée une liste et ajoute le résultat des trois appels de fonctions à la liste.  À la fin des trois appels de fonctions, elle retourne la liste. |
| **`SayHello`** | `HelloOrchestration_Hello` | La fonction retourne « hello ». Cette fonction contient la logique métier actuellement orchestrée. |
| **`HttpStart`** | `HelloOrchestration_HttpStart` | [Fonction déclenchée par HTTP](../functions-bindings-http-webhook.md) qui démarre une instance de l’orchestration et retourne une réponse d’état de vérification. |

Maintenant que vous avez créé un projet de fonction et une fonction durable, vous pouvez tester la fonction sur votre ordinateur local.

## <a name="test-the-function-locally"></a>Tester la fonction en local

Azure Functions Core Tools vous permet d’exécuter un projet Azure Functions sur votre ordinateur de développement local. Vous êtes invité à installer ces outils la première fois que vous démarrez une fonction dans Visual Studio Code.

1. Pour tester votre fonction, définissez un point d’arrêt dans le code de la fonction d’activité `SayHello` et appuyez sur F5 pour démarrer le projet d’application de fonction. La sortie de Core Tools est affichée dans le panneau **Terminal**.

    > [!NOTE]
    > Pour plus d’informations sur le débogage, consultez [Diagnostics Durable Functions](durable-functions-diagnostics.md#debugging).

1. Dans le panneau **Terminal**, copiez le point de terminaison de l’URL de votre fonction déclenchée via HTTP.

    ![Sortie Azure locale](media/durable-functions-create-first-csharp/functions-vscode-f5.png)

1. À l’aide d’un outil tel que [Postman](https://www.getpostman.com/) ou [cURL](https://curl.haxx.se/), envoyez une requête HTTP POST au point de terminaison de l’URL.

   La réponse est le résultat initial de la fonction HTTP, qui nous indique que l’orchestration durable a bien été démarrée. Il ne s’agit pas encore du résultat final de l’orchestration. La réponse contient plusieurs URL utiles. Pour le moment, demandons l’état de l’orchestration.

1. Copiez la valeur de l’URL pour `statusQueryGetUri`, collez-la dans la barre d’adresse du navigateur, puis exécutez la requête. Vous pouvez également continuer à utiliser Postman pour envoyer la requête GET.

   La requête interroge l’instance d’orchestration pour obtenir l’état. Une fois l’exécution de l’instance terminée, vous devez recevoir une réponse comprenant les sorties ou résultats de la fonction durable. Voici comment il se présente : 

    ```json
    {
        "name": "HelloOrchestration",
        "instanceId": "9a528a9e926f4b46b7d3deaa134b7e8a",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2020-03-18T21:54:49Z",
        "lastUpdatedTime": "2020-03-18T21:54:54Z"
    }
    ```

1. Pour arrêter le débogage, appuyez sur **Maj + F5** dans VS Code.

Après avoir vérifié que la fonction s’exécute correctement sur votre ordinateur local, il est temps de publier le projet sur Azure.

[!INCLUDE [functions-create-function-app-vs-code](../../../includes/functions-sign-in-vs-code.md)]

[!INCLUDE [functions-publish-project-vscode](../../../includes/functions-publish-project-vscode.md)]

## <a name="test-your-function-in-azure"></a>Tester votre fonction dans Azure

1. Copiez l’URL du déclencheur HTTP à partir du panneau **Sortie**. L’URL qui appelle la fonction déclenchée via HTTP doit être au format suivant :

    `https://<functionappname>.azurewebsites.net/api/HelloOrchestration_HttpStart`

1. Collez cette nouvelle URL de requête HTTP dans la barre d’adresse de votre navigateur. Vous devez obtenir la même réponse d’état que lorsque vous avez utilisé l’application publiée.

## <a name="next-steps"></a>Étapes suivantes

Vous avez utilisé Visual Studio Code pour créer et publier une application de fonction durable C#.

> [!div class="nextstepaction"]
> [Découvrez maintenant les modèles courants de fonctions durables](durable-functions-overview.md#application-patterns)

::: zone-end

::: zone pivot="code-editor-visualstudio"

Cet article explique comment utiliser Visual Studio 2019 pour créer et tester en local une fonction durable « Hello world ».  Cette fonction orchestre et chaîne des appels à d’autres fonctions. Vous allez ensuite publier le code de la fonction dans Azure. Ces outils sont disponibles dans le cadre de la charge de travail de développement Azure dans Visual Studio 2019.

![La capture d’écran montre une fenêtre Visual Studio 2019 avec une fonction durable.](./media/durable-functions-create-first-csharp/functions-vs-complete.png)

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

* Installez [Visual Studio 2019](https://visualstudio.microsoft.com/vs/). Vérifiez que la charge de travail de **développement Azure** est également installée. Visual Studio 2017 prend également en charge le développement de fonctions durables, mais l’interface utilisateur et les étapes diffèrent.

* Vérifiez que l’[émulateur de stockage Azure](../../storage/common/storage-use-emulator.md) est installé et démarré.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-function-app-project"></a>Créer un projet d’application de fonction

Le modèle Azure Functions crée un projet qui peut être publié dans une application de fonction dans Azure. Une application de fonctions vous permet de regrouper des fonctions en une unité logique pour faciliter la gestion, le déploiement et le partage des ressources.

1. Dans Visual Studio, sélectionnez **Nouveau** > **Projet** dans le menu **Fichier**.

1. Dans la boîte de dialogue **Créer un projet**, recherchez `functions`, choisissez le modèle **Azure Functions**, puis sélectionnez **Suivant**. 

    ![Boîte de dialogue Nouveau projet pour créer une fonction dans Visual Studio](./media/durable-functions-create-first-csharp/functions-vs-new-project.png)

1. Entrez un **Nom du projet**, puis sélectionnez **OK**. Le nom de projet devant être valide en tant qu’espace de noms C#, n’utilisez ni traits d’union, ni traits de soulignement, ni aucun autre caractère non alphanumérique.

1. Dans **Créer une application Azure Functions**, utilisez les paramètres spécifiés dans le tableau sous l’image.

    ![Boîte de dialogue Créer une application Azure Functions dans Visual Studio](./media/durable-functions-create-first-csharp/functions-vs-new-function.png)

    | Paramètre      | Valeur suggérée  | Description                      |
    | ------------ |  ------- |----------------------------------------- |
    | **Version** | Azure Functions 3.0 <br />(.NET Core) | Crée un projet de fonction qui utilise le Runtime Azure Functions version 3.0 prenant en charge .NET Core 3.1. Pour plus d’informations, consultez [Guide pratique pour cibler la version du runtime Azure Functions](../functions-versions.md).   |
    | **Modèle** | Vide | Crée une application de fonction vide. |
    | **Compte de stockage**  | Émulateur de stockage | Un compte de stockage est nécessaire pour la gestion des états de fonction durable. |

4. Sélectionnez **Créer** pour créer un projet de fonction vide. Ce projet contient les fichiers de configuration de base nécessaires à l’exécution de vos fonctions.

## <a name="add-functions-to-the-app"></a>Ajouter des fonctions à l’application

Les étapes suivantes utilisent un modèle pour créer le code de fonction durable dans votre projet.

1. Cliquez avec le bouton droit sur le projet dans Visual Studio et sélectionnez **Ajouter** > **Nouvelle fonction Azure**.

    ![Ajouter une nouvelle fonction](./media/durable-functions-create-first-csharp/functions-vs-add-function.png)

1. Vérifiez qu’**Azure Function** est sélectionné dans le menu Ajouter, tapez un nom pour votre fichier C#, puis sélectionnez **Ajouter**.

1. Sélectionnez le modèle **Orchestration Durable Functions**, puis sélectionnez **OK**.

    ![Sélectionner un modèle durable](./media/durable-functions-create-first-csharp/functions-vs-select-template.png)

Une nouvelle fonction durable est ajoutée à l’application.  Ouvrez le nouveau fichier .cs pour afficher le contenu. Cette fonction durable est un exemple de chaînage de fonctions simple avec les méthodes suivantes :  

| Méthode | FunctionName | Description |
| -----  | ------------ | ----------- |
| **`RunOrchestrator`** | `<file-name>` | Gère l’orchestration durable. Dans ce cas, l’orchestration démarre, crée une liste et ajoute le résultat des trois appels de fonctions à la liste.  À la fin des trois appels de fonctions, elle retourne la liste. |
| **`SayHello`** | `<file-name>_Hello` | La fonction retourne « hello ». Cette fonction contient la logique métier actuellement orchestrée. |
| **`HttpStart`** | `<file-name>_HttpStart` | [Fonction déclenchée par HTTP](../functions-bindings-http-webhook.md) qui démarre une instance de l’orchestration et retourne une réponse d’état de vérification. |

Maintenant que vous avez créé un projet de fonction et une fonction durable, vous pouvez tester la fonction sur votre ordinateur local.

## <a name="test-the-function-locally"></a>Tester la fonction en local

Azure Functions Core Tools vous permet d’exécuter un projet Azure Functions sur votre ordinateur de développement local. Vous êtes invité à installer ces outils la première fois que vous démarrez une fonction dans Visual Studio.

1. Pour tester votre fonction, appuyez sur F5. Si vous y êtes invité, acceptez la requête dans Visual Studio pour télécharger et installer Azure Functions Core (CLI) Tools. Vous devrez peut-être activer une exception de pare-feu afin de permettre aux outils de prendre en charge les requêtes HTTP.

2. Copiez l’URL de votre fonction à partir de la sortie runtime Azure Functions.

    ![Azure runtime local](./media/durable-functions-create-first-csharp/functions-vs-debugging.png)

3. Collez l’URL de la requête HTTP dans la barre d’adresse de votre navigateur et exécutez la requête. La capture d’écran suivante du navigateur montre la requête renvoyée par la fonction suite à la demande locale GET :

    ![La capture d’écran montre une fenêtre de navigateur avec statusQueryGetUri appelé.](./media/durable-functions-create-first-csharp/functions-vs-status.png)

    La réponse est le résultat initial de la fonction HTTP, qui nous indique que l’orchestration durable a bien été démarrée.  Il ne s’agit pas encore du résultat final de l’orchestration.  La réponse contient plusieurs URL utiles.  Pour le moment, demandons l’état de l’orchestration.

4. Copiez la valeur de l’URL pour `statusQueryGetUri` et collez-la dans la barre d’adresse du navigateur, puis exécutez la requête.

    La requête interroge l’instance d’orchestration pour obtenir l’état. Vous obtenez une réponse éventuelle qui ressemble à l’exemple ci-dessous.  Cette sortie nous indique que l’instance est terminée. Elle contient les sorties ou résultats de la fonction durable.

    ```json
    {
        "instanceId": "d495cb0ac10d4e13b22729c37e335190",
        "runtimeStatus": "Completed",
        "input": null,
        "customStatus": null,
        "output": [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ],
        "createdTime": "2019-11-02T07:07:40Z",
        "lastUpdatedTime": "2019-11-02T07:07:52Z"
    }
    ```

5. Pour arrêter le débogage, appuyez sur **MAJ + F5**.

Après avoir vérifié que la fonction s’exécute correctement sur votre ordinateur local, il est temps de publier le projet sur Azure.

## <a name="publish-the-project-to-azure"></a>Publication du projet sur Azure

Vous devez disposer d’une application de fonction dans votre abonnement Azure avant de pouvoir publier votre projet. Vous pouvez créer une application de fonction directement à partir de Visual Studio.

[!INCLUDE [Publish the project to Azure](../../../includes/functions-vstools-publish.md)]

## <a name="test-your-function-in-azure"></a>Tester votre fonction dans Azure

1. Copiez l’URL de base de l’application de fonction à partir de la page de profil de publication. Remplacez la partie `localhost:port` de l’URL que vous avez utilisée lors du test en local de la fonction par la nouvelle URL de base.

    L’URL qui appelle le déclencheur HTTP de la fonction durable doit être au format suivant :

    `https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>_HttpStart`

2. Collez cette nouvelle URL de requête HTTP dans la barre d’adresse de votre navigateur. Vous devez obtenir la même réponse d’état que lorsque vous avez utilisé l’application publiée.

## <a name="next-steps"></a>Étapes suivantes

Vous avez utilisé Visual Studio pour créer et publier une application de fonction durable C#.

> [!div class="nextstepaction"]
> [Découvrez maintenant les modèles courants de fonctions durables](durable-functions-overview.md#application-patterns)

::: zone-end
