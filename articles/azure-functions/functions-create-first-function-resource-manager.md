---
title: Créer votre première fonction à l’aide de modèles Azure Resource Manager
description: Créez et déployez sur Azure une simple fonction serverless déclenchée par HTTP à l’aide d’un modèle Azure Resource Manager (modèle ARM).
ms.date: 3/5/2020
ms.topic: quickstart
ms.service: azure-functions
ms.custom: subject-armqs
ms.openlocfilehash: 1e623405faa89ff41eccdaa57578bc8ac94cd78c
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422822"
---
# <a name="quickstart-create-and-deploy-azure-functions-resources-from-an-arm-template"></a>Démarrage rapide : Créer et déployer des ressources Azure Functions à partir d’un modèle ARM

Dans cet article, vous allez utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une fonction qui répond à des requêtes HTTP. 

Le fait de suivre ce guide de démarrage rapide entraîne une petite dépense de quelques cents USD tout au plus dans votre compte Azure. 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-function-app-create-dynamic%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

### <a name="azure-account"></a>Compte Azure 

Avant de commencer, vous devez disposer d’un compte Azure doté d’un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

### <a name="create-a-local-functions-project"></a>Créer un projet de fonctions local

Cet article nécessite un projet de code de fonctions local à exécuter sur des ressources Azure que vous créez. Si vous ne créez pas d’abord un projet à publier, vous ne pourrez pas effectuer la section relative au déploiement de cet article. 

Choisissez l’un des onglets suivants, suivez le lien, puis effectuez la section permettant de créer une application de fonction dans le langage de votre choix :

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Créez votre projet Functions local dans le langage de votre choix avec Visual Studio Code :  

+ [C#](create-first-function-vs-code-csharp.md)
+ [Java](create-first-function-vs-code-java.md)
+ [JavaScript](create-first-function-vs-code-node.md)
+ [PowerShell](create-first-function-vs-code-powershell.md)
+ [Python](create-first-function-vs-code-python.md)
+ [TypeScript](create-first-function-vs-code-typescript.md)

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[Créer votre projet de fonctions local dans Visual Studio](functions-create-your-first-function-visual-studio.md#create-a-function-app-project)

# <a name="command-line"></a>[Ligne de commande](#tab/command-line)

Créez votre projet Functions local dans le langage de votre choix à partir de la ligne de commande :

+ [C#](create-first-function-cli-csharp.md)
+ [Java](create-first-function-cli-java.md)
+ [JavaScript](create-first-function-cli-node.md)
+ [PowerShell](create-first-function-cli-powershell.md)
+ [Python](create-first-function-cli-python.md)
+ [TypeScript](create-first-function-cli-typescript.md)

---

Une fois que vous avez créé votre projet localement, vous créez les ressources nécessaires à l’exécution de votre nouvelle fonction dans Azure. 

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-function-app-create-dynamic/).

:::code language="json" source="~/quickstart-templates/101-function-app-create-dynamic/azuredeploy.json":::

Les quatre ressources Azure suivantes sont créées par ce modèle :

+ [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageaccounts) : création d’un compte de stockage Azure, exigé par Functions.
+ [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms) : création d’un plan d’hébergement Consommation serverless pour l’application de fonction.
+ [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites) : création d’une application de fonction.
+ [**Microsoft.Insights/Components**](/azure/templates/microsoft.insights/components) : création d’une instance Application Insights à des fins de supervision.

## <a name="deploy-the-template"></a>Déployer le modèle

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli-interactive
read -p "Enter a resource group name that is used for generating resource names:" resourceGroupName &&
read -p "Enter the location (like 'eastus' or 'northeurope'):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```
# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter a resource group name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (like 'eastus' or 'northeurope')"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```
---

## <a name="validate-the-deployment"></a>Valider le déploiement

Ensuite, vous validez l’application de fonction qui héberge les ressources que vous avez créées en publiant votre projet sur Azure et en appelant le point de terminaison HTTP de la fonction.

### <a name="publish-the-function-project-to-azure"></a>Publier le projet de fonction sur Azure

Les étapes suivantes vous permettent de publier votre projet sur les nouvelles ressources Azure :

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE [functions-republish-vscode](../../includes/functions-republish-vscode.md)]

Dans la sortie, copiez l’URL du déclencheur HTTP. Vous l’utilisez pour tester votre fonction en cours d’exécution dans Azure. 

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**.

1. Dans **Choisir une cible de publication**, choisissez **Plan de consommation Azure Functions** en cliquant sur **Sélectionner**, puis sélectionnez **Créer un profil**.

    :::image type="content" source="media/functions-create-first-function-arm/choose-publish-target-visual-studio.png" alt-text="Choisir une cible de publication existante":::

1. Choisissez votre **abonnement**, développez le groupe de ressources, sélectionnez votre application de fonction, puis sélectionnez **OK**.

1. Une fois la publication terminée, copiez l’**URL du site**.

    :::image type="content" source="media/functions-create-first-function-arm/publish-summary-site-url.png" alt-text="Copier l’URL du site à partir du résumé de la publication":::

1. Ajoutez le chemin `/api/<FUNCTION_NAME>?name=Functions`, où `<FUNCTION_NAME>` correspond au nom de votre fonction. L’URL qui appelle la fonction à déclencheur HTTP est au format suivant :

    `http://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?name=Functions`

Vous utilisez cette URL pour tester votre fonction de déclencheur HTTP en cours d’exécution dans Azure.

# <a name="command-line"></a>[Ligne de commande](#tab/command-line)

Pour publier votre code local dans une application de fonction sur Azure, utilisez la commande `publish` :

```cmd
func azure functionapp publish <FUNCTION_APP_NAME>
```

Dans cet exemple, remplacez `<FUNCTION_APP_NAME>` par le nom de votre application de fonction. Vous devrez peut-être vous reconnecter en utilisant `az login`. 

Dans la sortie, copiez l’URL du déclencheur HTTP. Vous l’utilisez pour tester votre fonction en cours d’exécution dans Azure.

---

### <a name="invoke-the-function-on-azure"></a>Appeler la fonction sur Azure

Collez l’URL que vous avez copiée pour la requête HTTP dans la barre d’adresse de votre navigateur, vérifiez que la chaîne de requête `name` sous la forme `?name=Functions` a été ajoutée à la fin de cette URL, puis exécutez la requête. 

La réponse doit ressembler à ceci :

<pre>Hello Functions!</pre>

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous passez à l’étape suivante, qui consiste à ajouter une liaison de sortie de file d’attente Stockage Azure, gardez toutes vos ressources en place, car vous allez tirer parti de ce que vous avez déjà fait.

Sinon, utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources qu’il contient afin d’éviter les coûts supplémentaires.

```azurecli
az group delete --name <RESOURCE_GROUP_NAME>
```

Remplacez `<RESOURCE_GROUP_NAME>` par le nom de votre groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez publié votre première fonction, découvrez-en plus en y ajoutant une liaison de sortie.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure](functions-add-output-binding-storage-queue-vs-code.md)

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure](functions-add-output-binding-storage-queue-vs.md)

# <a name="command-line"></a>[Ligne de commande](#tab/command-line)

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure](functions-add-output-binding-storage-queue-cli.md)

---
