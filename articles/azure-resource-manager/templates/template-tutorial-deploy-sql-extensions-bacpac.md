---
title: Importer des fichiers SQL BACPAC avec des modèles
description: Découvrez comment utiliser les extensions Azure SQL Database pour importer des fichiers SQL BACPAC avec des modèles Azure Resource Manager (modèles ARM).
author: mumian
ms.date: 12/09/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 2d50903f464c03157ee393787af6ddfdad975aed
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588027"
---
# <a name="tutorial-import-sql-bacpac-files-with-arm-templates"></a>Tutoriel : Importer des fichiers SQL BACPAC avec des modèles ARM

Découvrez comment utiliser les extensions Azure SQL Database pour importer un fichier [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) avec des modèles ARM (Azure Resource Manager). Les artefacts de déploiement correspondent à tous les fichiers, en plus des principaux fichiers modèles requis pour effectuer un déploiement. Le fichier BACPAC est un artefact.

Dans ce tutoriel, vous allez créer un modèle pour déployer un [serveur SQL logique](../../azure-sql/database/logical-servers.md) et une base de données unique, et importer un fichier BACPAC. Pour plus d’informations sur la façon de déployer des extensions de machine virtuelle Azure à l’aide de modèles ARM, consultez [Tutoriel : Déployer des extensions de machine virtuelle avec des modèles ARM](./template-tutorial-deploy-vm-extensions.md).

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
>
> * Préparer un fichier BACPAC
> * Ouvrir un modèle de démarrage rapide
> * Modifier le modèle
> * Déployez le modèle.
> * Vérifier le déploiement

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* Visual Studio Code avec l’extension Outils Resource Manager Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md).
* Pour une sécurité optimale, utilisez un mot de passe généré pour le compte administrateur de serveur. Voici un exemple que vous pouvez utiliser pour générer un mot de passe :

    ```console
    openssl rand -base64 32
    ```

    Azure Key Vault a été conçu pour protéger les clés et autres secrets de chiffrement. Pour plus d’informations, consultez [Didacticiel : Intégrer Azure Key Vault à un déploiement de modèle ARM](./template-tutorial-use-key-vault.md). Nous vous recommandons également de mettre à jour votre mot de passe tous les trois mois.

## <a name="prepare-a-bacpac-file"></a>Préparer un fichier BACPAC

Un fichier BACPAC est partagé dans [GitHub](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-sql-extension/SQLDatabaseExtension.bacpac). Pour créer le vôtre, consultez [Exporter une base de données d’Azure SQL Database vers un fichier BACPAC](../../azure-sql/database/database-export.md). Si vous choisissez de publier le fichier sur votre propre emplacement, vous devez mettre à jour le modèle plus tard dans ce didacticiel.

Le fichier BACPAC doit être stocké dans un compte de stockage Azure avant de pouvoir être importé à l’aide d’un modèle ARM. Le script PowerShell suivant prépare le fichier BACPAC en suivant les étapes ci-dessous :

* Télécharger le fichier BACPAC.
* Création d’un compte Azure Storage.
* Créer un conteneur d’objets blob dans un compte de stockage
* Télécharger le fichier BACPAC dans le conteneur.
* Afficher la clé du compte de stockage et l’URL de l’objet blob

1. Sélectionnez **Essayer** pour ouvrir l’interpréteur de commandes. Ensuite, collez le script PowerShell suivant dans la fenêtre de l’interpréteur de commandes.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate Azure resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"

    $resourceGroupName = "${projectName}rg"
    $storageAccountName = "${projectName}store"
    $containerName = "bacpacfiles"
    $bacpacFileName = "SQLDatabaseExtension.bacpac"
    $bacpacUrl = "https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-sql-extension/SQLDatabaseExtension.bacpac"

    # Download the bacpac file
    Invoke-WebRequest -Uri $bacpacUrl -OutFile "$HOME/$bacpacFileName"

    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
                                           -Name $storageAccountName `
                                           -SkuName Standard_LRS `
                                           -Location $location
    $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName `
                                                  -Name $storageAccountName).Value[0]

    # Create a container
    New-AzStorageContainer -Name $containerName -Context $storageAccount.Context

    # Upload the BACPAC file to the container
    Set-AzStorageBlobContent -File $HOME/$bacpacFileName `
                             -Container $containerName `
                             -Blob $bacpacFileName `
                             -Context $storageAccount.Context

    Write-Host "The storage account key is $storageAccountKey"
    Write-Host "The BACPAC file URL is https://$storageAccountName.blob.core.windows.net/$containerName/$bacpacFileName"
    Write-Host "Press [ENTER] to continue ..."
    ```

1. Notez la clé du compte de stockage et l’URL du fichier BACPAC. Vous avez besoin de ces valeurs lorsque vous déployez le modèle.

## <a name="open-a-quickstart-template"></a>Ouvrir un modèle de démarrage rapide

Le modèle utilisé dans ce tutoriel est stocké dans [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-sql-extension/azuredeploy.json).

1. À partir de Visual Studio Code, sélectionnez **Fichier** > **Ouvrir un fichier**.
1. Collez l’URL suivante dans **Nom de fichier** :

    ```url
    https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-sql-extension/azuredeploy.json
    ```

1. Sélectionnez **Ouvrir** pour ouvrir le fichier.

    Deux ressources sont définies dans le modèle :

   * `Microsoft.Sql/servers`. Consultez la [référence de modèle](/azure/templates/microsoft.sql/servers).
   * `Microsoft.SQL.servers/databases`. Consultez la [référence de modèle](/azure/templates/microsoft.sql/servers/databases).

        Il est préférable d’avoir des notions de base sur ce modèle avant de le personnaliser.
1. Sélectionnez **Fichier** > **Enregistrer sous** pour enregistrer une copie du fichier sur votre ordinateur local avec le nom *azuredeploy.json*.

## <a name="edit-the-template"></a>Modifier le modèle

1. Ajoutez deux paramètres supplémentaires à la fin de la section des `parameters` pour définir la clé de compte de stockage et l’URL BACPAC.

    ```json
        "storageAccountKey": {
          "type":"string",
          "metadata":{
            "description": "Specifies the key of the storage account where the BACPAC file is stored."
          }
        },
        "bacpacUrl": {
          "type":"string",
          "metadata":{
            "description": "Specifies the URL of the BACPAC file."
          }
        }
    ```

    Ajoutez une virgule après l’accolade fermante de la propriété `adminPassword` (`}`). Pour mettre en forme le fichier JSON à partir de Visual Studio Code, sélectionnez Maj+Alt+F.

    Pour obtenir ces deux valeurs, consultez [Préparer un fichier BACPAC](#prepare-a-bacpac-file).

1. Ajoutez deux ressources supplémentaires au modèle.

    * Pour permettre à l’extension SQL Database d’importer des fichiers BACPAC, vous devez autoriser le trafic à partir des services Azure. Ajoutez la définition de règle de pare-feu suivante sous la définition du serveur :

        ```json
        "resources": [
          {
            "type": "firewallrules",
            "apiVersion": "2015-05-01-preview",
            "name": "AllowAllAzureIps",
            "location": "[parameters('location')]",
            "dependsOn": [
              "[parameters('databaseServerName')]"
            ],
            "properties": {
              "startIpAddress": "0.0.0.0",
              "endIpAddress": "0.0.0.0"
            }
          }
        ]
        ```

        Le modèle ressemble à ceci :

        ![Modèle avec définition de règle de pare-feu](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-firewall.png)

    * Ajoutez une ressource d’extension SQL Database à la définition de base de données avec le code JSON suivant :

        ```json
        "resources": [
          {
            "type": "extensions",
            "apiVersion": "2014-04-01",
            "name": "Import",
            "dependsOn": [
              "[resourceId('Microsoft.Sql/servers/databases', parameters('databaseServerName'), parameters('databaseName'))]"
            ],
            "properties": {
              "storageKeyType": "StorageAccessKey",
              "storageKey": "[parameters('storageAccountKey')]",
              "storageUri": "[parameters('bacpacUrl')]",
              "administratorLogin": "[parameters('adminUser')]",
              "administratorLoginPassword": "[parameters('adminPassword')]",
              "operationMode": "Import"
            }
          }
        ]
        ```

        Le modèle ressemble à ceci :

        ![Modèle avec extension SQL Database](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac.png)

        Pour comprendre la définition de ressource, consultez la [référence de l’extension SQL Database](/azure/templates/microsoft.sql/servers/databases/extensions). Voici quelques éléments importants :

        * `dependsOn` : la ressource d’extension doit être créée une fois que la base de données a été créée.
        * `storageKeyType` : spécifiez le type de la clé de stockage à utiliser. La valeur peut être `StorageAccessKey` ou `SharedAccessKey`. Utilisez `StorageAccessKey` dans ce tutoriel.
        * `storageKey` : Spécifiez la clé pour le compte de stockage où est stocké le fichier BACPAC. Si le type de clé de stockage est `SharedAccessKey`, il doit être précédé de « ? ».
        * `storageUri` : spécifiez l’URL du fichier BACPAC stocké dans un compte de stockage.
        * `administratorLoginPassword` : mot de passe de l'administrateur SQL. Utilisez un mot de passe généré. Consultez les [Conditions préalables](#prerequisites).

Le modèle complet ressemble à ceci :

[!code-json[](~/resourcemanager-templates/tutorial-sql-extension/azuredeploy2.json?range=1-106&highlight=38-49,62-76,86-103)]

## <a name="deploy-the-template"></a>Déployer le modèle

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Reportez-vous à la section [Déployer le modèle](./template-tutorial-create-templates-with-dependent-resources.md#deploy-the-template) pour connaître la procédure de déploiement. Utilisez plutôt le script de déploiement PowerShell suivant :

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name that is used earlier"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$adminUsername = Read-Host -Prompt "Enter the SQL admin username"
$adminPassword = Read-Host -Prompt "Enter the admin password" -AsSecureString
$storageAccountKey = Read-Host -Prompt "Enter the storage account key"
$bacpacUrl = Read-Host -Prompt "Enter the URL of the BACPAC file"
$resourceGroupName = "${projectName}rg"

#New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -adminUser $adminUsername `
    -adminPassword $adminPassword `
    -TemplateFile "$HOME/azuredeploy.json" `
    -storageAccountKey $storageAccountKey `
    -bacpacUrl $bacpacUrl

Write-Host "Press [ENTER] to continue ..."
```

Envisagez d’utiliser le même nom de projet que celui que vous avez utilisé lors de la préparation du fichier BACPAC, afin que toutes les ressources soient stockées dans le même groupe de ressources. De cette façon, il est plus facile de gérer les tâches liées aux ressources, telles que leur nettoyage. Si vous utilisez le même nom de projet, vous pouvez soit supprimer la commande `New-AzResourceGroup` du script, soit répondre oui (o) ou non (n) quand vous êtes invité à mettre à jour le groupe de ressources existant.

Utilisez un mot de passe généré. Consultez les [Conditions préalables](#prerequisites).

## <a name="verify-the-deployment"></a>Vérifier le déploiement

Pour accéder au serveur à partir de votre ordinateur client, vous devez ajouter une règle de pare-feu supplémentaire. Pour plus d’informations, consultez [Créer et gérer des règles de pare-feu IP](../../azure-sql/database/firewall-configure.md#create-and-manage-ip-firewall-rules).

Dans le portail Azure, sélectionnez la base de données dans le groupe de ressources qui vient d’être déployé. Sélectionnez l’**éditeur de requêtes (préversion)** , puis entrez les informations d’identification d’administrateur. Vous verrez deux tables importées dans la base de données.

![Éditeur de requêtes (préversion)](./media/template-tutorial-deploy-sql-extensions-bacpac/resource-manager-tutorial-deploy-sql-extensions-bacpac-query-editor.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
1. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
1. Sélectionnez le nom du groupe de ressources. Vous verrez six ressources au total dans le groupe de ressources.
1. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déployé un serveur et une base de données, et vous avez importé un fichier BACPAC. Pour savoir comment résoudre les problèmes de déploiement de modèle, consultez :

> [!div class="nextstepaction"]
> [Résoudre les problèmes des déploiements de modèles ARM](./template-tutorial-troubleshoot.md)
