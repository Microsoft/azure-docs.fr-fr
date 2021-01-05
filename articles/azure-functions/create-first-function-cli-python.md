---
title: Créer une fonction Python à partir de la ligne de commande – Azure Functions
description: Apprenez à créer une fonction Python à partir de la ligne de commande, puis à publier le projet local sur un hébergement serverless dans Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-python
- devx-track-azurecli
ms.openlocfilehash: 5f8bf6fb6f464bcb74d34e2d99fac16c24b12615
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96175710"
---
# <a name="quickstart-create-a-python-function-in-azure-from-the-command-line"></a>Démarrage rapide : Créer une fonction Python dans Azure à partir de la ligne de commande

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

Dans cet article, vous utilisez des outils en ligne de commande pour créer une fonction Python qui répond à des requêtes HTTP. Après avoir testé le code localement, vous le déployez dans l’environnement serverless d’Azure Functions.

Le fait de suivre ce guide de démarrage rapide entraîne une faible dépense de quelques cents USD tout au plus dans votre compte Azure.

Si vous souhaitez consulter une version de cet article adaptée à Visual Studio Code, [cliquez ici](create-first-function-vs-code-python.md).

## <a name="configure-your-local-environment"></a>Configurer votre environnement local

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2) version 3.x.
  
+ Un des outils suivants pour créer des ressources Azure :

    + [Azure CLI](/cli/azure/install-azure-cli) version 2.4 ou ultérieure.

    + [Azure PowerShell](/powershell/azure/install-az-ps) version 5.0 ou ultérieure.

+ [Python 3.8 (64 bits)](https://www.python.org/downloads/release/python-382/), [Python 3.7 (64 bits)](https://www.python.org/downloads/release/python-375/), [Python 3.6 (64 bits)](https://www.python.org/downloads/release/python-368/), qui sont tous pris en charge par la version 3.x. d’Azure Functions.

### <a name="prerequisite-check"></a>Vérification du prérequis

Vérifiez vos prérequis, selon que vous utilisez Azure CLI ou Azure PowerShell pour la création de ressources Azure :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

+ Dans une fenêtre de terminal ou de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `az --version` pour vérifier que vous disposez d’Azure CLI version 2.4 ou ultérieure.

+ Exécutez `az login` pour vous connecter à Azure et vérifier que l’abonnement est actif.

+ Exécutez `python --version` (Linux/macOS) ou `py --version` (Windows) pour vérifier que vous disposez de Python version 3.8.x, 3.7.x ou 3.6.x.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

+ Dans une fenêtre de terminal ou de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `(Get-Module -ListAvailable Az).Version` et vérifiez la version 5.0 ou ultérieure. 

+ Exécutez `Connect-AzAccount` pour vous connecter à Azure et vérifier que l’abonnement est actif.

+ Exécutez `python --version` (Linux/macOS) ou `py --version` (Windows) pour vérifier que vous disposez de Python version 3.8.x, 3.7.x ou 3.6.x.

---

## <a name="create-and-activate-a-virtual-environment"></a><a name="create-venv"></a>Créer et activer un environnement virtuel

Dans le dossier approprié, exécutez les commandes suivantes pour créer et activer un environnement virtuel nommé `.venv`. Veillez à utiliser Python 3.8, 3.7 ou 3.6, qui sont pris en charge par Azure Functions.

# <a name="bash"></a>[bash](#tab/bash)

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

Si Python n’a pas installé le package venv sur votre distribution Linux, exécutez la commande suivante :

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
py -m venv .venv
```

```powershell
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -m venv .venv
```

```cmd
.venv\scripts\activate
```

---

Vous devez exécuter toutes les commandes suivantes dans cet environnement virtuel activé. 

## <a name="create-a-local-function-project"></a>Créer un projet de fonction local

Dans Azure Functions, un projet de fonction est un conteneur pour une ou plusieurs fonctions qui répondent chacune à un déclencheur spécifique. Toutes les fonctions d’un projet partagent les mêmes configurations locale et d’hébergement. Dans cette section, vous créez un projet de fonction qui contient une seule fonction.

1. Exécutez la commande `func init`, de la façon suivante, pour créer un projet Functions dans un dossier nommé *LocalFunctionProj* avec le runtime spécifié :  

    ```console
    func init LocalFunctionProj --python
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

#### <a name="__init__py"></a>\_\_init\_\_.py

*\_\_init\_\_.py* contient une fonction Python `main()`, qui se déclenche selon la configuration définie dans *function.json*.

:::code language="python" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/__init__.py":::

Pour un déclencheur HTTP, la fonction reçoit les données de requête dans la variable `req`, tel que cela est défini dans *function.json*. `req` est une instance de la [classe azure.functions.HttpRequest](/python/api/azure-functions/azure.functions.httprequest). L’objet retourné, défini comme `$return` dans *function.json*, est une instance de la [classe azure.functions.HttpResponse](/python/api/azure-functions/azure.functions.httpresponse). Pour en savoir plus, voir [Déclencheurs et liaisons HTTP Azure Functions](./functions-bindings-http-webhook.md?tabs=python).

#### <a name="functionjson"></a>function.json

*function.json* est un fichier config qui définit le `bindings` d’entrée et de sortie de la fonction, notamment le type du déclencheur.

Vous pouvez changer `scriptFile` pour appeler un autre fichier Python, si vous le souhaitez.

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::

Chaque liaison nécessite une direction, un type et un nom unique. Le déclencheur HTTP comporte une liaison d’entrée de type [`httpTrigger`](functions-bindings-http-webhook-trigger.md) et une liaison de sortie de type [`http`](functions-bindings-http-webhook-output.md).

[!INCLUDE [functions-run-function-test-local-cli](../../includes/functions-run-function-test-local-cli.md)]

## <a name="create-supporting-azure-resources-for-your-function"></a>Créer des ressources Azure de prise en charge pour votre fonction

Avant de déployer le code de votre fonction dans Azure, vous devez créer trois ressources :

- Un groupe de ressources, qui est un conteneur logique pour les ressources associées.
- Un compte de stockage, qui conserve l’état et d’autres informations spécifiques à vos projets.
- Une application de fonction, qui fournit l’environnement d’exécution de votre code de fonction. Une application de fonction est mappée à votre projet de fonction local. Elle vous permet de regrouper les fonctions en tant qu’unité logique pour faciliter la gestion, le déploiement et le partage des ressources.

Utilisez les commandes suivantes pour créer ces éléments. Azure CLI et PowerShell sont pris en charge.

1. Si vous ne l’avez pas déjà fait, connectez-vous à Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    ```azurecli
    az login
    ```

    La commande [az login](/cli/azure/reference-index#az-login) vous connecte à votre compte Azure.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell) 
    ```azurepowershell
    Connect-AzAccount
    ```

    L’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) vous connecte à votre compte Azure.

    ---

1. Créez un groupe de ressources nommé `AzureFunctionsQuickstart-rg` dans la région `westeurope`. 

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az group create --name AzureFunctionsQuickstart-rg --location westeurope
    ```
 
    La commande [az group create](/cli/azure/group#az-group-create) crée un groupe de ressources. Vous créez généralement votre groupe de ressources et vos ressources dans une région près de chez vous, en utilisant une région disponible retournée par la commande `az account list-locations`.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzResourceGroup -Name AzureFunctionsQuickstart-rg -Location westeurope
    ```

    La commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) crée un groupe de ressources. Vous créez généralement votre groupe de ressources et vos ressources dans une région près de chez vous, en utilisant une région disponible retournée par l’applet de commande [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

    ---

    > [!NOTE]
    > Vous ne pouvez pas héberger des applications Linux et Windows dans le même groupe de ressources. Si vous disposez d’un groupe de ressources existant nommé `AzureFunctionsQuickstart-rg` avec une application de fonction Windows ou une application web, vous devez utiliser un autre groupe de ressources.

1. Créez un compte de stockage universel dans votre groupe de ressources et votre région :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

    ```azurecli
    az storage account create --name <STORAGE_NAME> --location westeurope --resource-group AzureFunctionsQuickstart-rg --sku Standard_LRS
    ```

    La commande [az storage account create](/cli/azure/storage/account#az-storage-account-create) crée le compte de stockage. 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

    ```azurepowershell
    New-AzStorageAccount -ResourceGroupName AzureFunctionsQuickstart-rg -Name <STORAGE_NAME> -SkuName Standard_LRS -Location westeurope
    ```

    L’applet de commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) crée le compte de stockage.

    ---

    Dans l’exemple précédent, remplacez `<STORAGE_NAME>` par un nom qui vous convient et qui est unique dans Stockage Azure. Les noms doivent contenir entre 3 et 24 caractères, et comporter uniquement des lettres minuscules. `Standard_LRS` spécifie un compte universel, qui est [pris en charge par Functions](storage-considerations.md#storage-account-requirements).
    
    Le compte de stockage n’engendre que quelques centimes (USD) de dépense dans ce guide de démarrage rapide.

1. Créer l’application de fonction dans Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
        
    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location westeurope --runtime python --runtime-version 3.8 --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME> --os-type linux
    ```
    
    La commande [az functionapp create](/cli/azure/functionapp#az_functionapp_create) crée l’application de fonction dans Azure. Si vous utilisez Python 3.7 ou 3.6, remplacez `--runtime-version` par `3.7` ou `3.6`, respectivement.
    
    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -FunctionsVersion 3 -RuntimeVersion 3.8 -Runtime python -Location 'West Europe'
    ```
    
    L’applet de commande [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) crée l’application de fonction dans Azure. Si vous utilisez Python 3.7 ou 3.6, remplacez `-RuntimeVersion` par `3.7` ou `3.6`, respectivement.

    ---
    
    Dans l’exemple précédent, remplacez `<STORAGE_NAME>` par le nom du compte que vous avez utilisé à l’étape précédente, puis remplacez `<APP_NAME>` par le nom global unique qui vous convient.  `<APP_NAME>` représente également le domaine DNS par défaut pour l’application de fonction. 
    
    Cette commande crée une application de fonction qui s’exécute dans votre runtime de langage spécifié dans le [Plan Consommation Azure Functions](functions-scale.md#consumption-plan), qui est gratuit pour l’utilisation faite ici. La commande provisionne également une instance d’Azure Application Insights associée dans le même groupe de ressources, avec laquelle vous pouvez superviser votre application de fonction et visualiser les journaux. Pour plus d’informations, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md). L’instance n’entraîne aucun coût tant que vous ne l’activez pas.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

Exécutez la commande suivante pour afficher en temps quasi-réel les [journaux de streaming](functions-run-local.md#enable-streaming-logs) dans Application Insights du portail Azure :

```console
func azure functionapp logstream <APP_NAME> --browser
```

Dans une fenêtre de terminal distincte ou dans le navigateur, appelez à nouveau la fonction distante. Un journal détaillé de l’exécution de la fonction dans Azure est affiché dans le terminal. 

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-python)

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/python-functions-qs-survey)