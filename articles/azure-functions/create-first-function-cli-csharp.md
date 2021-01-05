---
title: Créer une fonction C# à partir de la ligne de commande – Azure Functions
description: Apprenez à créer une fonction à partir de la ligne de commande, puis à publier le projet local sur un hébergement serverless dans Azure Functions.
ms.date: 10/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-csharp
- devx-track-azurecli
ms.openlocfilehash: b87408669dbb9268d6e3cd47b410b888ef1a19c2
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94637147"
---
# <a name="quickstart-create-a-c-function-in-azure-from-the-command-line"></a>Démarrage rapide : Créer une fonction C# dans Azure à partir de la ligne de commande

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

Dans cet article, vous allez utiliser des outils en ligne de commande pour créer une fonction basée sur une bibliothèque de classes C# qui répond aux requêtes HTTP. Après avoir testé le code localement, vous le déployez dans l’environnement serverless d’Azure Functions.

Le fait de suivre ce guide de démarrage rapide entraîne une faible dépense de quelques cents USD tout au plus dans votre compte Azure.

Si vous souhaitez consulter une version de cet article adaptée à Visual Studio Code, [cliquez ici](create-first-function-vs-code-csharp.md).

## <a name="configure-your-local-environment"></a>Configurer votre environnement local

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Kit SDK .NET Core 3.1](https://www.microsoft.com/net/download)

+ [Azure Functions Core Tools](functions-run-local.md#v2) version 3.x.

+ Un des outils suivants pour créer des ressources Azure :

    + [Azure CLI](/cli/azure/install-azure-cli) version 2.4 ou ultérieure.

    + [Azure PowerShell](/powershell/azure/install-az-ps) version 5.0 ou ultérieure.

### <a name="prerequisite-check"></a>Vérification du prérequis

Vérifiez vos prérequis, selon que vous utilisez Azure CLI ou Azure PowerShell pour la création de ressources Azure :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ Dans une fenêtre de terminal ou une fenêtre de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `az --version` pour vérifier que vous disposez d’Azure CLI version 2.4 ou ultérieure.

+ Exécutez `az login` pour vous connecter à Azure et vérifier que l’abonnement est actif.

+ Exécutez `dotnet --list-sdks` pour vérifier que le SDK .NET Core version 3.1.x est installé.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ Dans une fenêtre de terminal ou une fenêtre de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `(Get-Module -ListAvailable Az).Version` et vérifiez la version 5.0 ou ultérieure. 

+ Exécutez `Connect-AzAccount` pour vous connecter à Azure et vérifier que l’abonnement est actif.

+ Exécutez `dotnet --list-sdks` pour vérifier que le SDK .NET Core version 3.1.x est installé.

---

## <a name="create-a-local-function-project"></a>Créer un projet de fonction local

Dans Azure Functions, un projet de fonction est un conteneur pour une ou plusieurs fonctions qui répondent chacune à un déclencheur spécifique. Toutes les fonctions d’un projet partagent les mêmes configurations locale et d’hébergement. Dans cette section, vous créez un projet de fonction qui contient une seule fonction.

1. Exécutez la commande `func init`, de la façon suivante, pour créer un projet Functions dans un dossier nommé *LocalFunctionProj* avec le runtime spécifié :  

    ```csharp
    func init LocalFunctionProj --dotnet
    ```

1. Accédez au dossier du projet :

    ```console
    cd LocalFunctionProj
    ```

    Ce dossier contient divers fichiers pour le projet, notamment des fichiers config nommés [local.settings.json](functions-run-local.md#local-settings-file) et [host.json](functions-host-json.md). Dans la mesure où *local.settings.json* peut contenir des secrets téléchargés à partir d’Azure, le fichier est exclu du contrôle de code source par défaut dans le fichier *.gitignore*.

1. Ajoutez une fonction à votre projet à l’aide de la commande suivante, où l’argument `--name` est le nom unique de votre fonction (HttpExample) et où l’argument `--template` spécifie le déclencheur de la fonction (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ``` 

    `func new` crée un fichier de code HttpExample.cs.

### <a name="optional-examine-the-file-contents"></a>(Facultatif) Examiner le contenu du fichier

Si vous le souhaitez, vous pouvez passer à [Exécuter la fonction localement](#run-the-function-locally) et examiner le contenu du fichier plus tard.

#### <a name="httpexamplecs"></a>HttpExample.cs

*HttpExample.cs* contient une méthode `Run` qui reçoit des données de requête dans la variable `req` de type [HttpRequest](/dotnet/api/microsoft.aspnetcore.http.httprequest), décorée avec **HttpTriggerAttribute** qui définit le comportement du déclencheur.

:::code language="csharp" source="~/functions-docs-csharp/http-trigger-template/HttpExample.cs":::

L’objet retourné est un [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) qui retourne un message de réponse sous la forme d’un [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200) ou d’un [BadRequestObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestobjectresult) (400). Pour en savoir plus, voir [Déclencheurs et liaisons HTTP Azure Functions](./functions-bindings-http-webhook.md?tabs=csharp).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Créer l’application de fonction dans Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime dotnet --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    La commande [az functionapp create](/cli/azure/functionapp#az_functionapp_create) crée l’application de fonction dans Azure. 
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime dotnet -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    L’applet de commande [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) crée l’application de fonction dans Azure. 
    
    ---
    
    Dans l’exemple précédent, remplacez `<STORAGE_NAME>` par le nom du compte que vous avez utilisé à l’étape précédente, puis remplacez `<APP_NAME>` par le nom global unique qui vous convient. `<APP_NAME>` représente également le domaine DNS par défaut pour l’application de fonction. 
    
    Cette commande crée une application de fonction qui s’exécute dans votre runtime de langage spécifié dans le [Plan Consommation Azure Functions](functions-scale.md#consumption-plan), qui est gratuit pour l’utilisation faite ici. La commande provisionne également une instance d’Azure Application Insights associée dans le même groupe de ressources, avec laquelle vous pouvez superviser votre application de fonction et visualiser les journaux. Pour plus d’informations, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md). L’instance n’entraîne aucun coût tant que vous ne l’activez pas.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure]

[Se connecter à une file d’attente Stockage Azure]: functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp
