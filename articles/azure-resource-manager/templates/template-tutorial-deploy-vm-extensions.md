---
title: Déployer des extensions de machine virtuelle avec un modèle
description: Découvrez comment déployer des extensions de machine virtuelle avec des modèles Azure Resource Manager (modèles ARM).
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9e04006a0908832c623230d89caa62b0985f32e4
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587942"
---
# <a name="tutorial-deploy-virtual-machine-extensions-with-arm-templates"></a>Tutoriel : Déployer des extensions de machine virtuelle avec des modèles ARM

Découvrez comment utiliser des [extensions de machines virtuelles Azure](../../virtual-machines/extensions/features-windows.md) pour exécuter des tâches de configuration et d’automatisation post-déploiement sur des machines virtuelles Azure. De nombreuses extensions de machine virtuelle différentes peuvent être utilisées avec les machines virtuelles Azure. Dans ce tutoriel, vous allez déployer une extension de script personnalisée à partir d’un modèle Azure Resource Manager (modèle ARM) pour exécuter un script PowerShell sur une machine virtuelle Windows. Le script installe le serveur Web sur la machine virtuelle.

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Préparer un script PowerShell
> * Ouvrir un modèle de démarrage rapide
> * Modifier le modèle
> * Déployer le modèle

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* Visual Studio Code avec l’extension Outils Resource Manager. Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).
* Pour une sécurité optimale, utilisez un mot de passe généré pour le compte administrateur de la machine virtuelle. Voici un exemple pour générer un mot de passe :

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault a été conçu pour protéger les clés et autres secrets de chiffrement. Pour plus d’informations, consultez [Didacticiel : Intégrer Azure Key Vault à votre déploiement de modèle ARM](./template-tutorial-use-key-vault.md). Nous vous recommandons également de mettre à jour votre mot de passe tous les trois mois.

## <a name="prepare-a-powershell-script"></a>Préparer un script PowerShell

Vous pouvez utiliser un script PowerShell inline ou un fichier de script. Ce tutoriel montre comment utiliser un fichier de script. Un script PowerShell avec le contenu suivant est partagé à partir de [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-vm-extension/installWebServer.ps1) :

```azurepowershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Si vous choisissez de publier le fichier sur votre propre emplacement, mettez à jour l’élément `fileUri` du modèle plus tard dans ce tutoriel.

## <a name="open-a-quickstart-template"></a>Ouvrir un modèle de démarrage rapide

Le dépôt Modèles de démarrage rapide Azure contient les modèles ARM. Au lieu de créer un modèle à partir de zéro, vous pouvez chercher un exemple de modèle et le personnaliser. Le modèle utilisé dans ce didacticiel se nomme [Déployer une machine virtuelle Windows simple](https://azure.microsoft.com/resources/templates/101-vm-simple-windows/).

1. Dans Visual Studio Code, sélectionnez **Fichier** > **Ouvrir un fichier**.
1. Dans la zone **Nom de fichier**, collez l’URL suivante :

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```

1. Pour ouvrir le fichier, sélectionnez **Ouvrir**.
    Le modèle définit cinq ressources :

   * [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts).
   * [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses).
   * [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups).
   * [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks).
   * [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces).
   * [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines).

     Il est préférable d’avoir des notions de base sur ce modèle avant de le personnaliser.

1. Enregistrez une copie du fichier sur votre ordinateur local sous le nom *azuredeploy.json* en sélectionnant **Fichier** > **Enregistrer sous**.

## <a name="edit-the-template"></a>Modifier le modèle

Ajoutez une ressource d’extension de machine virtuelle au modèle existant avec le contenu suivant :

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "apiVersion": "2019-12-01",
  "name": "[concat(variables('vmName'),'/', 'InstallWebServer')]",
  "location": "[parameters('location')]",
  "dependsOn": [
      "[concat('Microsoft.Compute/virtualMachines/',variables('vmName'))]"
  ],
  "properties": {
      "publisher": "Microsoft.Compute",
      "type": "CustomScriptExtension",
      "typeHandlerVersion": "1.7",
      "autoUpgradeMinorVersion":true,
      "settings": {
        "fileUris": [
          "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-vm-extension/installWebServer.ps1"
        ],
        "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File installWebServer.ps1"
      }
  }
}
```

Pour plus d’informations sur la définition de cette ressource, consultez les [informations de référence sur les extensions](/azure/templates/microsoft.compute/virtualmachines/extensions). Voici quelques éléments importants :

* `name` : étant donné que la ressource d’extension est une ressource enfant de l’objet de machine virtuelle, le nom doit être composé du préfixe du nom de la machine virtuelle. Consultez [Définition du nom et du type des ressources enfants](child-resource-name-type.md).
* `dependsOn` : créez la ressource d’extension après avoir créé la machine virtuelle.
* `fileUris` : il s’agit des emplacements où sont stockés les fichiers de script. Si vous choisissez de ne pas utiliser l’emplacement fourni, vous devez mettre à jour les valeurs.
* `commandToExecute` : cette commande appelle le script.

Pour utiliser un script inline, supprimez `fileUris` et mettez à jour `commandToExecute` comme ceci :

```powershell
powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools && powershell.exe remove-item 'C:\\inetpub\\wwwroot\\iisstart.htm' && powershell.exe Add-Content -Path 'C:\\inetpub\\wwwroot\\iisstart.htm' -Value $('Hello World from ' + $env:computername)
```

Ce script inline met également à jour le contenu de _iisstart.html_.

Vous devez également ouvrir le port HTTP pour pouvoir accéder au serveur web.

1. Recherchez `securityRules` dans le modèle.
1. Ajoutez la règle suivante à côté de **default-allow-3389**.

    ```json
    {
      "name": "AllowHTTPInBound",
      "properties": {
        "priority": 1010,
        "access": "Allow",
        "direction": "Inbound",
        "destinationPortRange": "80",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "*"
      }
    }
    ```

## <a name="deploy-the-template"></a>Déployer le modèle

Pour connaître la procédure de déploiement, consultez la section **Déployer le modèle** du [Tutoriel : Créer des modèles ARM avec des ressources dépendantes](./template-tutorial-create-templates-with-dependent-resources.md#deploy-the-template). Nous vous recommandons d’utiliser un mot de passe généré pour le compte administrateur de la machine virtuelle. Consultez la section [Prérequis](#prerequisites) de cet article.

À partir de Cloud Shell, exécutez la commande suivante pour récupérer l’adresse IP publique de la machine virtuelle :

```azurepowershell
(Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName).IpAddress
```

Collez l’adresse IP dans un navigateur web. La page d’accueil Internet Information Services (IIS) par défaut s’ouvre :

![Page d’accueil Internet Information Services (IIS)](./media/template-tutorial-deploy-vm-extensions/resource-manager-template-deploy-extensions-customer-script-web-server.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le menu de gauche du portail Azure, sélectionnez **Groupe de ressources**.
2. Dans la zone **Filtrer par nom**, entrez le nom du groupe de ressources.
3. Sélectionnez le nom du groupe de ressources.
    Six ressources sont affichées dans le groupe de ressources.
4. Dans le menu supérieur, sélectionnez **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez déployé une machine virtuelle et une extension de machine virtuelle. L’extension a installé le serveur Web IIS sur la machine virtuelle. Pour savoir comment utiliser l’extension Azure SQL Database pour importer un fichier BACPAC, consultez :

> [!div class="nextstepaction"]
> [Déployer des extensions SQL](./template-tutorial-deploy-sql-extensions-bacpac.md)
