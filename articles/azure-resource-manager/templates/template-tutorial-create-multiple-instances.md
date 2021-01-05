---
title: Créer plusieurs instances de ressource
description: Découvrez comment créer un modèle Azure Resource Manager (modèle ARM) pour déployer plusieurs instances de ressource Azure.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: fc1401959adb97f8c4caf6d413a212d9f3b62801
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588112"
---
# <a name="tutorial-create-multiple-resource-instances-with-arm-templates"></a>Tutoriel : Créer plusieurs instances de ressources grâce à des modèles ARM

Découvrez comment effectuer une itération dans votre modèle Azure Resource Manager (modèle ARM) pour créer plusieurs instances d’une ressource Azure. Dans ce tutoriel, vous modifiez un modèle pour créer trois instances de compte de stockage.

![Azure Resource Manager crée plusieurs instances de diagramme](./media/template-tutorial-create-multiple-instances/resource-manager-template-create-multiple-instances-diagram.png)

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Ouvrir un modèle de démarrage rapide
> * Modifier le modèle
> * Déployer le modèle

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* Visual Studio Code avec l’extension Outils Resource Manager. Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="open-a-quickstart-template"></a>Ouvrir un modèle de démarrage rapide

[Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/) est un référentiel de modèles ARM. Au lieu de créer un modèle à partir de zéro, vous pouvez chercher un exemple de modèle et le personnaliser. Le modèle utilisé dans ce démarrage rapide se nomme [Créer un compte de stockage standard](https://azure.microsoft.com/resources/templates/101-storage-account-create/). Le modèle définit une ressource de compte de stockage Azure.

1. À partir de Visual Studio Code, sélectionnez **Fichier** > **Ouvrir un fichier**.
1. Collez l’URL suivante dans **Nom de fichier** :

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```

1. Sélectionnez **Ouvrir** pour ouvrir le fichier.
1. Une ressource `Microsoft.Storage/storageAccounts` est définie dans le modèle. Comparer le modèle à la [référence du modèle](/azure/templates/Microsoft.Storage/storageAccounts). Il est préférable de comprendre quelques notions basiques du modèle avant de le personnaliser.
1. Sélectionnez **Fichier** > **Enregistrer sous** pour enregistrer le fichier sous le nom _azuredeploy.json_ sur votre ordinateur local.

## <a name="edit-the-template"></a>Modifier le modèle

Le modèle existant crée un unique compte de stockage. Vous personnalisez le modèle pour créer trois comptes de stockage.

Dans Visual Studio Code, effectuez les quatre modifications suivantes :

![Azure Resource Manager crée plusieurs instances](./media/template-tutorial-create-multiple-instances/resource-manager-template-create-multiple-instances.png)

1. Ajoutez un élément `copy` à la définition de ressource du compte de stockage. Dans l’élément `copy`, vous indiquez le nombre d’itérations et une variable pour cette boucle. La valeur de décompte doit être un entier positif et ne pas dépasser 800.
2. La fonction `copyIndex()` renvoie l’itération actuelle dans la boucle. Vous utilisez l’index en tant que préfixe du nom. `copyIndex()` est basé sur zéro. Pour décaler la valeur d’index, vous pouvez passer une valeur dans la fonction `copyIndex()`. Par exemple : `copyIndex(1)`.
3. Supprimez l’élément `variables`, car il n’est plus utilisé.
4. Supprimez l’élément `outputs`. Ce n’est plus nécessaire.

Le modèle complet ressemble à ceci :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      },
      "kind": "StorageV2",
      "copy": {
        "name": "storagecopy",
        "count": 3
      },
      "properties": {}
    }
  ]
}
```

Pour plus d’informations sur la création de plusieurs instances, consultez [Itération de ressource dans les modèles ARM](./copy-resources.md).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Se connecter à [Azure Cloud Shell](https://shell.azure.com)

1. Choisissez votre environnement préféré en sélectionnant **PowerShell** ou **Bash** (pour CLI) en haut à gauche. Il est nécessaire de redémarrer l’interpréteur de commandes lors d’un tel changement.

    ![Fichier de chargement du Cloud Shell du portail Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Sélectionnez **Charger/Télécharger des fichiers**, puis **Charger**. Consultez la capture d’écran précédente. Sélectionnez le fichier que vous avez enregistré dans la section précédente. Après avoir chargé le fichier, vous pouvez utiliser la commande `ls` et la commande `cat` pour vérifier que le chargement a été correctement effectué.

1. Dans le Cloud Shell, exécutez les commandes suivantes. Sélectionnez l’onglet pour afficher le code PowerShell ou CLI.

    # <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json"
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json"
    ```

    ---

Après un déploiement de modèle réussi, vous pouvez afficher les trois comptes de stockage créés dans le groupe de ressources spécifié. Comparez les noms de compte de stockage avec la définition de nom dans le modèle.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
echo "Enter a project name that is used to generate resource group name:" &&
read projectName &&
resourceGroupName="${projectName}rg" &&
az storage account list --resource-group $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
$resourceGroupName = "${projectName}rg"

Get-AzStorageAccount -ResourceGroupName $resourceGroupName
Write-Host "Press [ENTER] to continue ..."
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.  Vous devez voir au total trois ressources dans le groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à créer plusieurs instances de compte de stockage. Dans le tutoriel suivant, vous développez un modèle disposant de plusieurs ressources et types de ressources. Certaines ressources comportent des ressources dépendantes.

> [!div class="nextstepaction"]
> [Créer des ressources dépendantes](./template-tutorial-create-templates-with-dependent-resources.md)
