---
title: Déployer des modèles avec le service Cloud Shell
description: Utilisez Azure Resource Manager et le service Cloud Shell pour déployer des ressources sur Azure. Les ressources sont définies dans un modèle Azure Resource Manager.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: d0b519955cb877f5fb9640cc0cf95a898e1743ef
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92681260"
---
# <a name="deploy-arm-templates-from-cloud-shell"></a>Déployer des modèles Azure Resource Manager à partir du service Cloud Shell

Vous pouvez utiliser le service [Cloud Shell](../../cloud-shell/overview.md) pour déployer un modèle Azure Resource Manager (modèle ARM). Vous pouvez déployer un modèle ARM stocké à distance ou sur le compte de stockage local pour le service Cloud Shell.

Vous pouvez effectuer le déploiement sur n’importe quelle étendue. Cet article présente un déploiement sur un groupe de ressources.

## <a name="deploy-remote-template"></a>Déployer un modèle distant

Pour déployer un modèle externe, indiquez précisément l’URI du modèle comme vous le feriez pour tout autre déploiement externe. Le modèle externe peut se trouver dans un dépôt GitHub ou un compte de stockage externe.

1. Ouvrez l’invite du service Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/open-cloud-shell.png" alt-text="Ouvrir Cloud Shell":::

1. Pour déployer le modèle, exécutez les commandes suivantes :

   # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "Central US"
   az deployment group create \
     --name ExampleDeployment \
     --resource-group ExampleGroup \
     --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="deploy-local-template"></a>Déployer un modèle local

Pour déployer un modèle local, vous devez commencer par charger votre modèle dans le compte de stockage connecté à votre session Cloud Shell.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez votre groupe de ressources Cloud Shell. Le modèle de nom est `cloud-shell-storage-<region>`.

   ![Sélection du groupe de ressources](./media/deploy-cloud-shell/select-cloud-shell-resource-group.png)

1. Sélectionnez le compte de stockage de votre Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/cloud-shell-storage.png" alt-text="Sélectionner un compte de stockage":::

1. Sélectionnez **Partages de fichiers**.

   :::image type="content" source="./media/deploy-cloud-shell/files-shares.png" alt-text="Sélectionner des partages de fichiers":::

1. Sélectionnez le partage de fichiers par défaut pour le service Cloud Shell. Le format de nom du partage de fichiers est `cs-<user>-<domain>-com-<uniqueGuid>`.

   :::image type="content" source="./media/deploy-cloud-shell/select-file-share.png" alt-text="Partage de fichiers par défaut":::

1. Ajoutez un répertoire pour stocker vos modèles. Sélectionnez ce répertoire.

   :::image type="content" source="./media/deploy-cloud-shell/add-directory.png" alt-text="Ajouter un répertoire":::

1. Sélectionnez **Télécharger**.

   :::image type="content" source="./media/deploy-cloud-shell/upload-template.png" alt-text="Charger un modèle":::

1. Recherchez et chargez votre modèle.

   :::image type="content" source="./media/deploy-cloud-shell/select-template.png" alt-text="Sélectionner un modèle":::

1. Ouvrez l’invite du service Cloud Shell.

   :::image type="content" source="./media/deploy-cloud-shell/open-cloud-shell.png" alt-text="Ouvrir Cloud Shell":::

1. Accédez au répertoire **clouddrive**. Accédez au répertoire que vous avez ajouté pour stocker les modèles.

1. Pour déployer le modèle, exécutez les commandes suivantes :

   # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

   ```azurecli-interactive
   az group create --name ExampleGroup --location "South Central US"
   az deployment group create \
     --resource-group ExampleGroup \
     --template-file azuredeploy.json \
     --parameters storageAccountType=Standard_GRS
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```azurepowershell-interactive
   New-AzResourceGroup -Name ExampleGroup -Location "Central US"
   New-AzResourceGroupDeployment `
     -DeploymentName ExampleDeployment `
     -ResourceGroupName ExampleGroup `
     -TemplateFile azuredeploy.json `
     -storageAccountType Standard_GRS
   ```

   ---

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les commandes de déploiement, consultez [Déployer des ressources à l’aide de modèles ARM et l’interface CLI Azure](deploy-cli.md) et [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](deploy-powershell.md).
- Pour afficher un aperçu des modifications avant de déployer un modèle, consultez [Opération de simulation de déploiement de modèle ARM](template-deploy-what-if.md).
