---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 02/19/2020
ms.author: glenga
ms.openlocfilehash: a5c113849296275432acf1f5603377a1909a2c04
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96842391"
---
## <a name="run-the-function-locally"></a>Exécuter la fonction localement

Azure Functions Core Tools s’intègre à Visual Studio Code pour vous permettre d’exécuter et de déboguer un projet Azure Functions localement. Pour plus d’informations sur le débogage dans Visual Studio Code, consultez [Déboguer localement des fonctions PowerShell Azure](../articles/azure-functions/functions-debug-powershell-local.md). 
1. Pour appeler votre fonction, appuyez sur <kbd>F5</kbd> pour démarrer le projet d’application de fonction. La sortie de Core Tools est affichée dans le panneau **Terminal**. Si vous rencontrez des problèmes d’exécution sur Windows, assurez-vous que le terminal par défaut pour Visual Studio Code n’est pas défini sur **WSL Bash**.

1. Dans le panneau **Terminal**, copiez le point de terminaison de l’URL de votre fonction déclenchée via HTTP.

    ![Sortie Azure locale](./media/functions-run-function-test-local-vs-code-ps/functions-vscode-f5.png)

1. Ajoutez la chaîne de requête `?name=<yourname>` à cette URL, puis utilisez `Invoke-RestMethod` dans une deuxième invite de commandes PowerShell pour exécuter la requête, comme suit :

    ```powershell
    PS > Invoke-RestMethod -Method Get -Uri http://localhost:7071/api/HttpTrigger?name=PowerShell
    Hello PowerShell
    ```

    Vous pouvez également exécuter la requête GET depuis un navigateur avec l’URL suivante :

    `http://localhost:7071/api/HttpExample?name=PowerShell`

    Quand vous appelez le point de terminaison HttpTrigger sans passer de paramètre `name` comme paramètre de requête ou dans le corps, la fonction retourne une erreur `BadRequest`. Quand vous examinez le code dans run.ps1, vous voyez que cette erreur est voulue.

1. Les informations relatives à la requête s’affichent dans le panneau **Terminal**.

    ![Exécution de fonctions dans le panneau Terminal](./media/functions-run-function-test-local-vs-code-ps/function-execution-terminal.png)

1. Quand vous avez terminé, appuyez sur **Ctrl+C** pour arrêter Core Tools.

Après avoir vérifié que la fonction s’exécute correctement sur votre ordinateur local, il est temps de publier le projet sur Azure.
