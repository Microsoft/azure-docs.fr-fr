---
title: Utiliser une condition dans des modèles
description: Découvrez comment déployer des ressources Azure en fonction des conditions. Montre comment déployer une nouvelle ressource ou utiliser une ressource existante.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: f044863be7d0bfaaad57d3974a1d2856b27927ea
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589132"
---
# <a name="tutorial-use-condition-in-arm-templates"></a>Tutoriel : Utiliser une condition dans des modèles ARM

Découvrez comment déployer des ressources Azure en fonction des conditions d’un modèle Azure Resource Manager (modèle ARM).

Dans le didacticiel [Définir l’ordre de déploiement des ressources](./template-tutorial-create-templates-with-dependent-resources.md), vous créez une machine virtuelle, un réseau virtuel et d’autres ressources dépendantes, y compris un compte de stockage. Plutôt que de créer un compte de stockage à chaque fois, vous laissez le choix aux utilisateurs de créer un compte de stockage ou d’utiliser un compte de stockage existant. Vous définissez pour cela un paramètre supplémentaire. Si la valeur du paramètre est **new**, un compte de stockage est créé. Sinon, un compte de stockage existant avec le nom fourni est utilisé.

![Diagramme de condition d’utilisation d’un modèle Resource Manager](./media/template-tutorial-use-conditions/resource-manager-template-use-condition-diagram.png)

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Ouvrir un modèle de démarrage rapide
> * Modifier le modèle
> * Déployer le modèle
> * Nettoyer les ressources

Ce tutoriel traite uniquement d’un scénario de base d’utilisation de conditions. Pour plus d'informations, consultez les pages suivantes :

* [Structure de fichiers de modèle : Condition](conditional-resource-deployment.md).
* [Déployer une ressource de manière conditionnelle dans un modèle ARM](/azure/architecture/building-blocks/extending-templates/conditional-deploy).
* [Fonction de modèle : If](./template-functions-logical.md#if).
* [Fonctions de comparaison pour les modèles ARM](./template-functions-comparison.md)

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* Visual Studio Code avec l’extension Outils Resource Manager. Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Pour une sécurité optimale, utilisez un mot de passe généré pour le compte administrateur de la machine virtuelle. Voici un exemple pour générer un mot de passe :

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault a été conçu pour protéger les clés et autres secrets de chiffrement. Pour plus d’informations, consultez [Didacticiel : Intégrer Azure Key Vault à un déploiement de modèle ARM](./template-tutorial-use-key-vault.md). Nous vous recommandons également de mettre à jour votre mot de passe tous les trois mois.

## <a name="open-a-quickstart-template"></a>Ouvrir un modèle de démarrage rapide

Le référentiel Modèles de démarrage rapide Azure contient les modèles ARM. Au lieu de créer un modèle à partir de zéro, vous pouvez chercher un exemple de modèle et le personnaliser. Le modèle utilisé dans ce didacticiel se nomme [Déployer une machine virtuelle Windows simple](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

1. À partir de Visual Studio Code, sélectionnez **Fichier** > **Ouvrir un fichier**.
1. Collez l’URL suivante dans **Nom de fichier** :

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Sélectionnez **Ouvrir** pour ouvrir le fichier.
1. Il existe six ressources définies par le modèle :

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

    Il est utile de vérifier les informations de référence sur les modèles avant de personnaliser un modèle.

1. Sélectionnez **Fichier** > **Enregistrer sous** pour enregistrer une copie du fichier sur votre ordinateur local avec le nom _azuredeploy.json_.

## <a name="modify-the-template"></a>Modifier le modèle

Apportez deux modifications au modèle existant :

* Ajoutez un paramètre de nom de compte de stockage. Les utilisateurs peuvent spécifier un nouveau nom de compte de stockage ou un nom de compte de stockage existant.
* Ajoutez un nouveau paramètre appelé `newOrExisting`. Le déploiement utilise ce paramètre pour déterminer s’il faut créer un compte de stockage ou utiliser un compte de stockage existant.

Voici la procédure pour apporter les modifications :

1. Ouvrez _azuredeploy.json_ dans Visual Studio Code.
1. Remplacez les trois `variables('storageAccountName')` par `parameters('storageAccountName')` dans le modèle entier.
1. Supprimez la définition de variable suivante :

    ![Capture d’écran mettant en évidence les définitions de variables que vous devez supprimer.](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-remove-storageaccountname.png)

1. Ajoutez les deux paramètres suivants au début de la section des paramètres :

    ```json
    "storageAccountName": {
      "type": "string"
    },
    "newOrExisting": {
      "type": "string",
      "allowedValues": [
        "new",
        "existing"
      ]
    },
    ```

    Appuyez sur Alt+Maj+F pour mettre en forme le modèle dans Visual Studio Code.

    La définition de paramètres mise à jour ressemble à :

    ![Condition d’utilisation de Resource Manager](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template-parameters.png)

1. Ajoutez la ligne suivante au début de la définition du compte de stockage.

    ```json
    "condition": "[equals(parameters('newOrExisting'),'new')]",
    ```

    La condition vérifie la valeur du paramètre `newOrExisting`. Si la valeur du paramètre est **nouveau**, le déploiement crée le compte de stockage.

    La définition du compte de stockage mise à jour ressemble à :

    ![Capture d’écran montrant la définition du compte de stockage mise à jour.](./media/template-tutorial-use-conditions/resource-manager-tutorial-use-condition-template.png)
1. Mettez à jour la propriété `storageUri` de la définition de ressource de machine virtuelle avec la valeur suivante :

    ```json
    "storageUri": "[concat('https://', parameters('storageAccountName'), '.blob.core.windows.net')]"
    ```

    Cette modification est nécessaire lorsque vous utilisez un compte de stockage existant sous un autre groupe de ressources.

1. Enregistrez les modifications.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Se connecter à [Azure Cloud Shell](https://shell.azure.com)

1. Choisissez votre environnement préféré en sélectionnant **PowerShell** ou **Bash** (pour CLI) en haut à gauche. Il est nécessaire de redémarrer l’interpréteur de commandes lors d’un tel changement.

    ![Fichier de chargement du Cloud Shell du portail Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Sélectionnez **Charger/Télécharger des fichiers**, puis **Charger**. Consultez la capture d’écran précédente. Sélectionnez le fichier que vous avez enregistré dans la section précédente. Après avoir chargé le fichier, vous pouvez utiliser la commande `ls` et la commande `cat` pour vérifier que le chargement a été correctement effectué.

1. Exécutez le script PowerShell suivant pour déployer le modèle.

    > [!IMPORTANT]
    > Le nom du compte de stockage doit être unique dans Azure. Le nom ne doit contenir que des lettres minuscules ou des chiffres. Il ne doit pas compter plus de 24 caractères. Le nom du compte de stockage est le nom du projet suivi du suffixe **store**. Vérifiez que le nom du projet et le nom du compte de stockage généré respectent les critères de nommage des comptes de stockage.

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name and resource names"
    $newOrExisting = Read-Host -Prompt "Create new or use existing (Enter new or existing)"
    $location = Read-Host -Prompt "Enter the Azure location (i.e. centralus)"
    $vmAdmin = Read-Host -Prompt "Enter the admin username"
    $vmPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
    $dnsLabelPrefix = Read-Host -Prompt "Enter the DNS Label prefix"

    $resourceGroupName = "${projectName}rg"
    $storageAccountName = "${projectName}store"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -adminUsername $vmAdmin `
        -adminPassword $vmPassword `
        -dnsLabelPrefix $dnsLabelPrefix `
        -storageAccountName $storageAccountName `
        -newOrExisting $newOrExisting `
        -TemplateFile "$HOME/azuredeploy.json"

    Write-Host "Press [ENTER] to continue ..."
    ```

    > [!NOTE]
    > Le déploiement échoue si `newOrExisting` est **nouveau**, mais le compte de stockage avec le nom de compte de stockage spécifié déjà existe.

Essayez d’effectuer un autre déploiement avec `newOrExisting` défini sur **existing** et spécifiez un compte de stockage existant. Pour créer un compte de stockage au préalable, consultez [Créer un compte de stockage](../../storage/common/storage-account-create.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources. Pour supprimer le groupe de ressources, sélectionnez **Essayer** afin d’ouvrir Cloud Shell. Pour coller le script PowerShell, cliquez sur le volet de l’interpréteur de commandes, puis sélectionnez **Coller**.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name you used in the last procedure"
$resourceGroupName = "${projectName}rg"

Remove-AzResourceGroup -Name $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez développé un modèle qui permet aux utilisateurs de choisir de créer un compte de stockage ou d’utiliser un compte de stockage existant. Pour savoir comment récupérer des secrets dans Azure Key Vault et utiliser les secrets comme mots de passe dans le déploiement de modèle, consultez :

> [!div class="nextstepaction"]
> [Intégrer Key Vault dans le déploiement de modèle](./template-tutorial-use-key-vault.md)
