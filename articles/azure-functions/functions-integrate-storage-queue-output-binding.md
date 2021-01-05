---
title: Ajouter des messages au stockage de files d’attente Azure, à l’aide de Functions
description: Utilisez Azure Functions pour créer une fonction sans serveur appelée par une requête HTTP et qui crée un message dans une file d’attente de Stockage Azure.
ms.assetid: 0b609bc0-c264-4092-8e3e-0784dcc23b5d
ms.topic: how-to
ms.date: 04/24/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 9c635b01bcd04bd03191fca2590b0189bad0f544
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90982019"
---
# <a name="add-messages-to-an-azure-storage-queue-using-functions"></a>Ajouter des messages au stockage de files d’attente Azure, à l’aide de Functions

Dans Azure Functions, les liaisons d’entrée et de sortie fournissent une méthode déclarative pour rendre disponibles les données des services externes pour votre code. Dans ce démarrage rapide, vous utilisez une liaison de sortie pour créer un message dans une file d’attente lorsqu’une fonction est déclenchée par une requête HTTP. Vous utilisez le conteneur de stockage Azure pour afficher les messages de file d’attente créés par votre fonction.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide de démarrage rapide :

- Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- Suivez les instructions de [Créer votre première fonction à l’aide du Portail Azure](functions-create-first-azure-function.md) en ignorant l’étape **Supprimer des ressources**. Ce démarrage rapide crée l’application de fonction, ainsi que la fonction que vous utilisez ici.

## <a name="add-an-output-binding"></a><a name="add-binding"></a>Ajoutez une liaison de sortie

Dans cette section, l’interface utilisateur du portail vous permet d’ajouter une liaison de sortie de stockage de file d’attente à la fonction que vous avez créée précédemment. Cette liaison permet d’écrire un minimum de code pour créer un message dans une file d’attente. Vous n’êtes pas obligé d’écrire du code pour des tâches telles que l’ouverture d’une connexion de stockage, la création d’une file d’attente ou l’obtention d’une référence à une file d’attente. La liaison de sortie de file d’attente et le runtime Azure Functions se chargent de ces tâches.

1. Dans le portail Azure, ouvrez la page d’application de fonction correspondant à l’application de fonction que vous avez créée dans [Créer votre première fonction à l’aide du Portail Azure](functions-create-first-azure-function.md). Pour ouvrir la page, recherchez et sélectionnez **Function App**. Puis, sélectionnez votre application de fonction.

1. Sélectionnez votre application de fonction, puis sélectionnez la fonction que vous avez créée précédemment dans ce guide de démarrage rapide.

1. Sélectionnez **Integration** (Intégration), puis **+ Add output** (+ Ajouter une sortie).

   :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-create-output-binding.png" alt-text="Création d’une liaison de sortie pour votre fonction." border="true":::

1. Sélectionnez le type de liaison **Azure Queue Storage output** (Sortie de Stockage File d’attente Azure), puis ajoutez les paramètres indiqués dans le tableau qui suit cette capture d’écran : 

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-create-output-binding-details.png" alt-text="Création d’une liaison de sortie pour votre fonction." border="true":::
    
    | Paramètre      |  Valeur suggérée   | Description                              |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nom de message de paramètre** | outputQueueItem | Le nom du paramètre de liaison de sortie. | 
    | **Nom de la file d’attente**   | outqueue  | Le nom de la file d’attente à connecter à votre compte de stockage. |
    | **Connexion au compte de stockage** | AzureWebJobsStorage | Vous pouvez utiliser la connexion de compte de stockage qui est déjà utilisée par votre application de fonction, ou créez-en une.  |

1. Sélectionnez **OK** pour ajouter la liaison.

Maintenant que vous avez défini une liaison de sortie, vous devez mettre à jour le code afin d’utiliser la liaison pour ajouter des messages à une file d’attente.  

## <a name="add-code-that-uses-the-output-binding"></a>Ajouter le code qui utilise la liaison de sortie

Dans cette section, vous ajoutez le code qui écrit un message dans la file d’attente de sortie. Le message contient la valeur qui est transmise au déclencheur HTTP dans la chaîne de requête. Par exemple, si la chaîne de requête inclut `name=Azure`, le message de la file d'attente sera *Nom transmis à la fonction : Azure*.

1. Dans votre fonction, sélectionnez **Code + Test** (Coder et tester) pour afficher le code de fonction dans l’éditeur.

1. Mettez à jour le code de fonction en fonction du langage de la fonction :

    # <a name="c"></a>[C\#](#tab/csharp)

    Ajoutez un paramètre **outputQueueItem** à la signature de méthode, comme indiqué dans l’exemple suivant.

    ```cs
    public static async Task<IActionResult> Run(HttpRequest req,
        ICollector<string> outputQueueItem, ILogger log)
    {
        ...
    }
    ```

    Dans le corps de la fonction juste avant l’instruction `return`, ajoutez le code qui utilise le paramètre pour créer un message de file d’attente.

    ```cs
    outputQueueItem.Add("Name passed to the function: " + name);
    ```

    # <a name="javascript"></a>[JavaScript](#tab/nodejs)

    Ajoutez le code qui utilise la liaison de sortie sur l’objet `context.bindings` pour créer un message de file d’attente. Ajoutez ce code avant l’instruction `context.done`.

    ```javascript
    context.bindings.outputQueueItem = "Name passed to the function: " + 
                (req.query.name || req.body.name);
    ```

    ---

1. Sélectionnez **Enregistrer** pour enregistrer les modifications.

## <a name="test-the-function"></a>Tester la fonction

1. Après avoir enregistré les modifications de code, sélectionnez **Test** (Tester).
1. Vérifiez que votre test correspond à l’image ci-dessous, puis sélectionnez **Run** (Exécuter). 

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/functions-test-run-function.png" alt-text="Création d’une liaison de sortie pour votre fonction." border="true":::

    Notez que **Corps de la demande** contient la valeur `name`*Azure*. Cette valeur s’affiche dans le message de file d’attente qui est créé lorsque la fonction est appelée.
    
    Plutôt que de sélectionner **Exécuter** ici, vous pouvez appeler la fonction en entrant une URL dans un navigateur et en spécifiant la valeur `name` dans la chaîne de requête. La méthode du navigateur est affichée dans le [démarrage rapide précédent](functions-create-first-azure-function.md#test-the-function).

1. Vérifiez les journaux d’activité pour vous assurer que la fonction a réussi. 

Une nouvelle file d’attente nommée **outqueue** est créée dans votre compte de stockage, par le runtime des fonctions, lorsque la liaison de sortie est utilisée pour la première fois. Vous allez utiliser le compte de stockage pour vérifier que la file d’attente et un message dans celle-ci ont bien été créés.

### <a name="find-the-storage-account-connected-to-azurewebjobsstorage"></a>Rechercher le compte de stockage connecté à AzureWebJobsStorage


1. Accédez à votre application de fonction, puis sélectionnez **Configuration**.

1. Sous **Application settings** (Paramètres de l’application), sélectionnez **AzureWebJobsStorage**.

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-find-storage-account.png" alt-text="Création d’une liaison de sortie pour votre fonction." border="true":::

1. Localisez et notez le nom du compte.

    :::image type="content" source="./media/functions-integrate-storage-queue-output-binding/function-storage-account-name.png" alt-text="Création d’une liaison de sortie pour votre fonction." border="true":::

### <a name="examine-the-output-queue"></a>Analyser la file d’attente de sortie

1. Dans le groupe de ressources de votre application de fonction, sélectionnez le compte de stockage que vous utilisez pour ce démarrage rapide.

1. Sous **Queue service**, (Service de File d’attente), sélectionnez **Queues** (Files d’attente), puis la file d’attente nommée **outqueue**. 

   La file d’attente contient le message que la liaison de sortie de file d’attente a créé lors de l’exécution de la fonction déclenchée via HTTP. Si vous avez appelé la fonction avec la valeur `name` par défaut d'*Azure*, le message de la file d'attente est *Nom transmis à la fonction : Azure*.

1. Exécutez de nouveau la fonction, et vous verrez un nouveau message s’afficher dans la file d’attente.  

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [Clean up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez ajouté une liaison de sortie à une fonction existante. Pour en savoir plus sur la liaison vers le stockage de files d’attente, consultez la page [Liaisons de file d’attente de stockage Azure Functions](functions-bindings-storage-queue.md).

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps-2.md)]
