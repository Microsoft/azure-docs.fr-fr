---
title: Créer une fonction JavaScript à partir de la ligne de commande – Azure Functions
description: Apprenez à créer une fonction JavaScript à partir de la ligne de commande, puis à publier le projet Node.js local sur un hébergement serverless dans Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4a178cb0e69a6e2b666247bcb8e76ab2c7e272e1
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94635668"
---
# <a name="quickstart-create-a-javascript-function-in-azure-from-the-command-line"></a>Démarrage rapide : Créer une fonction JavaScript dans Azure à partir de la ligne de commande


[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

Dans cet article, vous allez utiliser des outils en ligne de commande pour créer une fonction JavaScript qui répond aux requêtes HTTP. Après avoir testé le code localement, vous le déployez dans l’environnement serverless d’Azure Functions. 

Le fait de suivre ce guide de démarrage rapide entraîne une faible dépense de quelques cents USD tout au plus dans votre compte Azure.

Si vous souhaitez consulter une version de cet article adaptée à Visual Studio Code, [cliquez ici](create-first-function-vs-code-node.md).

## <a name="configure-your-local-environment"></a>Configurer votre environnement local

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](./functions-run-local.md#v2) version 3.x.

+ Un des outils suivants pour créer des ressources Azure :

    + [Azure CLI version 2.4 ou ultérieure](/cli/azure/install-azure-cli).

    + [Azure PowerShell](/powershell/azure/install-az-ps) version 5.0 ou ultérieure.

+ [Node.js](https://nodejs.org/) version 12. La version 10 de Node.js est également prise en charge.

### <a name="prerequisite-check"></a>Vérification du prérequis

Vérifiez vos prérequis, selon que vous utilisez Azure CLI ou Azure PowerShell pour la création de ressources Azure :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ Dans une fenêtre de terminal ou de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `az --version` pour vérifier que vous disposez d’Azure CLI version 2.4 ou ultérieure.

+ Exécutez `az login` pour vous connecter à Azure et vérifier que l’abonnement est actif.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ Dans une fenêtre de terminal ou de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `(Get-Module -ListAvailable Az).Version` et vérifiez la version 5.0 ou ultérieure. 

+ Exécutez `Connect-AzAccount` pour vous connecter à Azure et vérifier que l’abonnement est actif.

---

## <a name="create-a-local-function-project"></a>Créer un projet de fonction local

Dans Azure Functions, un projet de fonction est un conteneur pour une ou plusieurs fonctions qui répondent chacune à un déclencheur spécifique. Toutes les fonctions d’un projet partagent les mêmes configurations locale et d’hébergement. Dans cette section, vous créez un projet de fonction qui contient une seule fonction.

1. Exécutez la commande `func init`, de la façon suivante, pour créer un projet Functions dans un dossier nommé *LocalFunctionProj* avec le runtime spécifié :  

    ```console
    func init LocalFunctionProj --javascript
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
    
    `func new` crée un sous-dossier correspondant au nom de la fonction qui contient le fichier de code approprié au langage choisi pour le projet, et un fichier config nommé *function.json*.
    
### <a name="optional-examine-the-file-contents"></a>(Facultatif) Examiner le contenu du fichier

Si vous le souhaitez, vous pouvez passer à [Exécuter la fonction localement](#run-the-function-locally) et examiner le contenu du fichier plus tard.

#### <a name="indexjs"></a>index.js

*index.js* exporte une fonction qui est déclenchée selon la configuration définie dans *function.json*.

:::code language="javascript" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/index.js":::

Pour un déclencheur HTTP, la fonction reçoit les données de requête dans la variable `req`, tel que cela est défini dans *function.json*. L’objet retourné, défini comme `$return` dans *function.json*, est la réponse. Pour en savoir plus, voir [Déclencheurs et liaisons HTTP Azure Functions](./functions-bindings-http-webhook.md?tabs=javascript).

#### <a name="functionjson"></a>function.json

*function.json* est un fichier config qui définit le `bindings` d’entrée et de sortie de la fonction, notamment le type du déclencheur. 

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/function.json":::

Chaque liaison nécessite une direction, un type et un nom unique. Le déclencheur HTTP comporte une liaison d’entrée de type [`httpTrigger`](functions-bindings-http-webhook-trigger.md) et une liaison de sortie de type [`http`](functions-bindings-http-webhook-output.md).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Créer l’application de fonction dans Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime node --runtime-version 12 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    
    La commande [az functionapp create](/cli/azure/functionapp#az-functionapp-create) crée l’application de fonction dans Azure. Si vous utilisez Node.js 10, remplacez aussi `--runtime-version` par `10`.
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime node -RuntimeVersion 12 -FunctionsVersion 3 -Location 'West Europe'
    ```
    
    L’applet de commande [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) crée l’application de fonction dans Azure. Si vous utilisez Node.js 10, remplacez `-RuntimeVersion` par `10`.
    
    ---
    
    Dans l’exemple précédent, remplacez `<STORAGE_NAME>` par le nom du compte que vous avez utilisé à l’étape précédente, puis remplacez `<APP_NAME>` par le nom global unique qui vous convient. `<APP_NAME>` représente également le domaine DNS par défaut pour l’application de fonction. 
    
    Cette commande crée une application de fonction qui s’exécute dans votre runtime de langage spécifié dans le [Plan Consommation Azure Functions](functions-scale.md#consumption-plan), qui est gratuit pour l’utilisation faite ici. La commande provisionne également une instance d’Azure Application Insights associée dans le même groupe de ressources, avec laquelle vous pouvez superviser votre application de fonction et visualiser les journaux. Pour plus d’informations, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md). L’instance n’entraîne aucun coût tant que vous ne l’activez pas.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-javascript)