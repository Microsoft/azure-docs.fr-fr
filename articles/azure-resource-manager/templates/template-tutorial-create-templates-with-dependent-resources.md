---
title: Modèle avec ressources dépendantes
description: Découvrez comment créer votre premier modèle Azure Resource Manager (modèle ARM) avec plusieurs ressources, et comment le déployer ensuite dans le portail Azure
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: d1e5848e568f42fb8a77c65c775962f27a5a03df
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588034"
---
# <a name="tutorial-create-arm-templates-with-dependent-resources"></a>Tutoriel : Créer des modèles ARM avec des ressources dépendantes

Découvrez comment créer un modèle Azure Resource Manager (modèle ARM) pour déployer plusieurs ressources et configurer l’ordre de déploiement. Après avoir créé le modèle, vous déployez le modèle à l’aide de Cloud Shell à partir du portail Azure.

Dans ce didacticiel, vous créez un compte de stockage, une machine virtuelle, un réseau virtuel et d’autres ressources dépendantes. Certaines ressources ne peuvent pas être déployées avant qu’une autre ressource n’existe. Par exemple, vous ne pouvez pas créer la machine virtuelle avant que son compte de stockage et son interface réseau n’existent. Vous définissez cette relation en rendant une seule ressource dépendante des autres ressources. Resource Manager évalue les dépendances entre les ressources et les déploie dans leur ordre dépendant. Quand les ressources ne dépendent pas les unes des autres, Resource Manager les déploie en parallèle. Pour plus d’informations, consultez [Définir l’ordre de déploiement des ressources dans les modèles ARM](./define-resource-dependency.md).

![Schéma de l’ordre de déploiement des ressources dépendant du modèle Resource Manager](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-dependent-resources-diagram.png)

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Ouvrir un modèle de démarrage rapide
> * Explorer le modèle
> * Déployer le modèle

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
2. Collez l’URL suivante dans **Nom de fichier** :

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

3. Sélectionnez **Ouvrir** pour ouvrir le fichier.
4. Sélectionnez **Fichier** > **Enregistrer sous** pour enregistrer une copie du fichier sur votre ordinateur local avec le nom _azuredeploy.json_.

## <a name="explore-the-template"></a>Explorer le modèle

Lorsque vous explorez le modèle dans cette section, essayez de répondre aux questions suivantes :

* Combien de ressources Azure sont-elles définies dans ce modèle ?
* L’une des ressources est un compte de stockage Azure. La définition est-elle semblable à celle utilisée dans le dernier didacticiel ?
* Trouvez-vous les références de modèle pour les ressources définies dans ce modèle ?
* Trouvez-vous les dépendances des ressources ?

1. À partir de Visual Studio Code, réduisez les éléments jusqu'à ce que vous voyiez uniquement les éléments de premier niveau et les éléments de second niveau à l’intérieur des `resources` :

    ![Modèles ARM Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code.png)

    Il existe six ressources définies par le modèle :

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

     Il est utile de vérifier les informations de référence sur les modèles avant de personnaliser un modèle.

1. Développez la première ressource. Il s’agit d’un compte de stockage. Comparez la définition de ressource à la [référence de modèle](/azure/templates/Microsoft.Storage/storageAccounts).

    ![Définition du compte de stockage des modèles ARM Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-storage-account-definition.png)

1. Développez la deuxième ressource. Le type de ressource, est `Microsoft.Network/publicIPAddresses`. Comparez la définition de ressource à la [référence de modèle](/azure/templates/microsoft.network/publicipaddresses).

    ![Définition de l’adresse IP publique des modèles ARM Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-public-ip-address-definition.png)

1. Développez la troisième ressource. Le type de ressource, est `Microsoft.Network/networkSecurityGroups`. Comparez la définition de ressource à la [référence de modèle](/azure/templates/microsoft.network/networksecuritygroups).

    ![Définition de groupe de sécurité réseau pour les modèles ARM Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-network-security-group-definition.png)

1. Développez la quatrième ressource. Le type de ressource, est `Microsoft.Network/virtualNetworks` :

    ![Élément dependsOn de réseau virtuel des modèles ARM Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-virtual-network-definition.png)

    L’élément `dependsOn` vous permet de définir une ressource comme une dépendance sur une ou plusieurs ressources. Cette ressource dépend d’une autre ressource :

    * `Microsoft.Network/networkSecurityGroups`

1. Développez la cinquième ressource. Le type de ressource, est `Microsoft.Network/networkInterfaces`. La ressource dépend de deux autres ressources :

    * `Microsoft.Network/publicIPAddresses`
    * `Microsoft.Network/virtualNetworks`

1. Développez la sixième ressource. Cette ressource est une machine virtuelle. Elle dépend de deux autres ressources :

    * `Microsoft.Storage/storageAccounts`
    * `Microsoft.Network/networkInterfaces`

Le diagramme suivant illustre les ressources et les informations de dépendance pour ce modèle :

![Diagramme de dépendance des modèles ARM Visual Studio Code](./media/template-tutorial-create-templates-with-dependent-resources/resource-manager-template-visual-studio-code-dependency-diagram.png)

En spécifiant les dépendances, Resource Manager déploie efficacement la solution. Il déploie le compte de stockage, l’adresse IP publique et le réseau virtuel en parallèle car ils n’ont aucune dépendance. Après que l’adresse IP publique et le réseau virtuel sont déployés, l’interface réseau est créée. Lorsque toutes les autres ressources sont déployées, Resource Manager déploie la machine virtuelle.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Se connecter à [Azure Cloud Shell](https://shell.azure.com)

1. Choisissez votre environnement préféré en sélectionnant **PowerShell** ou **Bash** (pour CLI) en haut à gauche.  Il est nécessaire de redémarrer l’interpréteur de commandes lors d’un tel changement.

    ![Fichier de chargement du Cloud Shell du portail Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Sélectionnez **Charger/Télécharger des fichiers**, puis **Charger**. Consultez la capture d’écran précédente. Sélectionnez le fichier que vous avez enregistré précédemment. Après avoir chargé le fichier, vous pouvez utiliser la commande `ls` et la commande `cat` pour vérifier que le chargement a été correctement effectué.

1. Exécutez le script PowerShell suivant pour déployer le modèle.

    # <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    echo "Enter the virtual machine admin username:" &&
    read adminUsername &&
    echo "Enter the DNS label prefix:" &&
    read dnsLabelPrefix &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location $location &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json" --parameters adminUsername=$adminUsername dnsLabelPrefix=$dnsLabelPrefix
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $adminUsername = Read-Host -Prompt "Enter the virtual machine admin username"
    $adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
    $dnsLabelPrefix = Read-Host -Prompt "Enter the DNS label prefix"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -adminUsername $adminUsername `
        -adminPassword $adminPassword `
        -dnsLabelPrefix $dnsLabelPrefix `
        -TemplateFile "$HOME/azuredeploy.json"

    Write-Host "Press [ENTER] to continue ..."
    ```

    ---

1. Établir une connexion RDP vers la machine virtuelle pour vérifier la machine virtuelle a été créée avec succès.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources. Vous verrez six ressources au total dans le groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez développé et déployé un modèle pour créer une machine virtuelle, un réseau virtuel et les ressources dépendantes. Pour découvrir comment utiliser des scripts de déploiement pour effectuer des opérations de pré/post-déploiement, consultez :

> [!div class="nextstepaction"]
> [Utiliser un script de déploiement](./template-tutorial-deployment-script.md)
