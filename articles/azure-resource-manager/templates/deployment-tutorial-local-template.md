---
title: 'Tutoriel : Déployer un modèle Azure Resource Manager local'
description: Découvrez comment déployer un modèle ARM (Azure Resource Manager) à partir de votre ordinateur local
ms.date: 05/20/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 640d314711e34119dac5e1c5bf9fa245685b6f38
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368134"
---
# <a name="tutorial-deploy-a-local-arm-template"></a>Tutoriel : Déployer un modèle ARM local

Découvrez comment déployer un modèle ARM (Azure Resource Manager) à partir de votre ordinateur local Il faut compter environ **8 minutes** pour effectuer ce tutoriel.

Ce tutoriel est le premier d’une série. Au fur et à mesure que vous progressez dans la série, vous modularisez le modèle en créant un modèle lié, stockez le modèle lié dans un compte de stockage et le sécurisez à l’aide du jeton SAS, et apprenez à créer un pipeline DevOps pour déployer des modèles. Cette série porte sur le déploiement de modèles. Si vous souhaitez apprendre le développement de modèles, consultez les [tutoriels pour les débutants](./template-tutorial-create-first-template.md).

## <a name="get-tools"></a>Obtenir les outils

Commençons par nous assurer que vous disposez des outils indispensables pour déployer des modèles.

### <a name="command-line-deployment"></a>Déploiement en ligne de commande

Vous avez besoin d’Azure PowerShell ou d’Azure CLI pour déployer le modèle. Pour les instructions d’installation, consultez :

- [Installation d'Azure PowerShell](/powershell/azure/install-az-ps)
- [Installer Azure CLI sur Windows](/cli/azure/install-azure-cli-windows)
- [Installer Azure CLI sur Linux](/cli/azure/install-azure-cli-linux)
- [Installer Azure CLI sur macOS](/cli/azure/install-azure-cli-macos)

Après avoir installé Azure PowerShell ou Azure CLI, prenez soin de vous connecter une première fois. Pour obtenir de l’aide, consultez [Se connecter - PowerShell](/powershell/azure/install-az-ps#sign-in) ou [Se connecter - Azure CLI](/cli/azure/get-started-with-azure-cli#sign-in).

### <a name="editor-optional"></a>Éditeur (facultatif)

Les modèles sont des fichiers JSON. Pour examiner/modifier des modèles, vous avez besoin d’un bon éditeur JSON. Nous conseillons Visual Studio Code avec l’extension Outils Resource Manager. Si vous devez installer ces outils, consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="review-template"></a>Vérifier le modèle

Le modèle déploie un compte de stockage, un plan App Service et une application web. Si vous êtes intéressé par la création du modèle, vous pouvez suivre le [tutoriel sur les modèles de démarrage rapide](template-tutorial-quickstart-template.md). Toutefois, cela n’est pas obligatoire pour effectuer ce tutoriel.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

> [!IMPORTANT]
> Ce nom doit comprendre entre 3 et 24 caractères, uniquement des lettres en minuscules et des nombres. Le nom doit être unique. Dans le modèle, le nom du compte de stockage est le nom du projet auquel **store** est ajouté, et le nom du projet doit comprendre entre 3 et 11 caractères. Le nom du projet doit donc respecter les exigences du nom du compte de stockage et comporter moins de 11 caractères.

Enregistrez une copie du modèle sur votre ordinateur local avec l’extension _.json_, par exemple _azuredeploy.json_. Vous allez déployer ce modèle ultérieurement dans le tutoriel.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Pour commencer à utiliser Azure PowerShell/Azure CLI afin de déployer un modèle, connectez-vous avec vos informations d’identification Azure.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az login
```

---

Si vous avez plusieurs abonnements Azure, sélectionnez celui que vous souhaitez utiliser. Remplacez `[SubscriptionID/SubscriptionName]` et les crochets `[]` par vos informations d’abonnement :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az account set --subscription [SubscriptionID/SubscriptionName]
```

---

## <a name="create-resource-group"></a>Créer un groupe de ressources

Lorsque vous déployez un modèle, vous indiquez un groupe de ressources pour contenir les ressources. Avant d’exécuter la commande de déploiement, créez le groupe de ressources avec Azure CLI ou Azure PowerShell. Sélectionnez les onglets dans la section de code suivante pour choisir entre Azure PowerShell et Azure CLI. Les exemples CLI de cet article sont écrits pour l’interpréteur de commandes Bash.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource and resource group names"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroup `
  -Name $resourceGroupName `
  -Location "Central US"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource and resource group names:"
read projectName
resourceGroupName="${projectName}rg"

az group create \
  --name $resourceGroupName \
  --location "Central US"
```

---

## <a name="deploy-template"></a>Déployer un modèle

Utilisez l’une des options de déploiement, ou les deux, pour déployer le modèle.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-AzResourceGroupDeployment `
  -Name DeployLocalTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -projectName $projectName `
  -verbose
```

Pour en savoir plus sur le déploiement de modèles avec Azure PowerShell, consultez [Déployer des ressources à l’aide de modèles ARM et d’Azure PowerShell](./deploy-powershell.md).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
echo "Enter the same project name:"
read projectName
echo "Enter the template file path and file name:"
read templateFile
resourceGroupName="${projectName}rg"

az deployment group create \
  --name DeployLocalTemplate \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters projectName=$projectName \
  --verbose
```

Pour en savoir plus sur le déploiement de modèles avec Azure CLI, consultez [Déployer des ressources à l’aide de modèles ARM et d’Azure CLI](./deploy-cli.md).

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à déployer un modèle local. Dans le tutoriel suivant, vous allez séparer le modèle en un modèle principal et un modèle lié, et apprendre à stocker et à sécuriser le modèle lié.

> [!div class="nextstepaction"]
> [Déployer un modèle lié](./deployment-tutorial-linked-template.md)
